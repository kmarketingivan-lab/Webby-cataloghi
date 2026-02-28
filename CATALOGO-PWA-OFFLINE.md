# CATALOGO-PWA-OFFLINE

§5. INSTALL PROMPT
beforeinstallprompt event
typescript
// lib/pwa/install-prompt.ts
export type BeforeInstallPromptEvent = Event & {
  prompt: () => Promise<void>;
  userChoice: Promise<{ outcome: 'accepted' | 'dismissed' }>;
};

declare global {
  interface WindowEventMap {
    beforeinstallprompt: BeforeInstallPromptEvent;
  }
}

export class InstallPromptManager {
  private deferredPrompt: BeforeInstallPromptEvent | null = null;
  private listeners: Array<(available: boolean) => void> = [];

  constructor() {
    if (typeof window === 'undefined') return;
    
    window.addEventListener('beforeinstallprompt', this.handleBeforeInstallPrompt);
    window.addEventListener('appinstalled', this.handleAppInstalled);
  }

  private handleBeforeInstallPrompt = (e: BeforeInstallPromptEvent) => {
    e.preventDefault();
    this.deferredPrompt = e;
    this.notifyListeners(true);
    
    // Log install availability
    this.logAnalytics('install_available');
  };

  private handleAppInstalled = () => {
    this.deferredPrompt = null;
    this.notifyListeners(false);
    this.logAnalytics('installed');
    
    // Clean up
    window.removeEventListener('beforeinstallprompt', this.handleBeforeInstallPrompt);
    window.removeEventListener('appinstalled', this.handleAppInstalled);
  };

  public async showPrompt(): Promise<boolean> {
    if (!this.deferredPrompt) {
      return false;
    }

    try {
      this.deferredPrompt.prompt();
      const choice = await this.deferredPrompt.userChoice;
      
      this.logAnalytics('install_prompt_shown', {
        outcome: choice.outcome
      });

      this.deferredPrompt = null;
      this.notifyListeners(false);
      
      return choice.outcome === 'accepted';
    } catch (error) {
      console.error('Error showing install prompt:', error);
      this.logAnalytics('install_error', { error: error.message });
      return false;
    }
  }

  public isInstallable(): boolean {
    return !!this.deferredPrompt;
  }

  public addListener(listener: (available: boolean) => void) {
    this.listeners.push(listener);
  }

  public removeListener(listener: (available: boolean) => void) {
    this.listeners = this.listeners.filter(l => l !== listener);
  }

  private notifyListeners(available: boolean) {
    this.listeners.forEach(listener => listener(available));
  }

  private logAnalytics(event: string, params?: Record<string, any>) {
    // Integrate with your analytics provider
    if (typeof window !== 'undefined' && (window as any).gtag) {
      (window as any).gtag('event', event, params);
    }
  }
}

// Singleton instance
export const installPromptManager = new InstallPromptManager();
Custom install button
typescript
// components/pwa/install-button.tsx
'use client';

import { useState, useEffect } from 'react';
import { installPromptManager } from '@/lib/pwa/install-prompt';

interface InstallButtonProps {
  className?: string;
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
}

export default function InstallButton({ 
  className = '', 
  variant = 'primary',
  size = 'md'
}: InstallButtonProps) {
  const [isInstallable, setIsInstallable] = useState(false);
  const [isInstalled, setIsInstalled] = useState(false);
  const [isIOS, setIsIOS] = useState(false);

  useEffect(() => {
    const handleInstallabilityChange = (available: boolean) => {
      setIsInstallable(available);
    };

    installPromptManager.addListener(handleInstallabilityChange);
    setIsInstallable(installPromptManager.isInstallable());
    
    // Detect iOS
    const userAgent = window.navigator.userAgent.toLowerCase();
    setIsIOS(/iphone|ipad|ipod/.test(userAgent));
    
    // Detect if already installed
    const isStandalone = window.matchMedia('(display-mode: standalone)').matches;
    setIsInstalled(isStandalone);

    return () => {
      installPromptManager.removeListener(handleInstallabilityChange);
    };
  }, []);

  const handleInstall = async () => {
    const installed = await installPromptManager.showPrompt();
    if (installed) {
      setIsInstalled(true);
    }
  };

  const sizeClasses = {
    sm: 'px-3 py-1 text-sm',
    md: 'px-4 py-2 text-base',
    lg: 'px-6 py-3 text-lg'
  };

  const variantClasses = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    outline: 'border border-blue-600 text-blue-600 hover:bg-blue-50'
  };

  if (isInstalled) {
    return (
      <button
        className={`${sizeClasses[size]} ${variantClasses[variant]} rounded-lg font-medium transition-colors opacity-50 cursor-default ${className}`}
        disabled
      >
        ✓ App Installed
      </button>
    );
  }

  if (isIOS) {
    return (
      <div className={`inline-block ${className}`}>
        <div className="text-sm text-gray-600 mb-1">
          Tap <span className="font-bold">Share</span> then{' '}
          <span className="font-bold">Add to Home Screen</span>
        </div>
      </div>
    );
  }

  if (!isInstallable) {
    return null;
  }

  return (
    <button
      onClick={handleInstall}
      className={`${sizeClasses[size]} ${variantClasses[variant]} rounded-lg font-medium transition-colors ${className}`}
    >
      Install App
    </button>
  );
}
Install banner component
typescript
// components/pwa/install-banner.tsx
'use client';

import { useState, useEffect } from 'react';
import { XMarkIcon, ArrowDownTrayIcon } from '@heroicons/react/24/outline';
import { installPromptManager } from '@/lib/pwa/install-prompt';

interface InstallBannerProps {
  delay?: number; // Delay in milliseconds before showing
  storageKey?: string;
  maxDismissals?: number;
}

export default function InstallBanner({
  delay = 3000,
  storageKey = 'pwa-install-banner-dismissals',
  maxDismissals = 3
}: InstallBannerProps) {
  const [isVisible, setIsVisible] = useState(false);
  const [isInstallable, setIsInstallable] = useState(false);
  const [isIOS, setIsIOS] = useState(false);

  useEffect(() => {
    const handleInstallabilityChange = (available: boolean) => {
      setIsInstallable(available);
    };

    installPromptManager.addListener(handleInstallabilityChange);
    setIsInstallable(installPromptManager.isInstallable());

    // Check dismissals
    const dismissals = parseInt(localStorage.getItem(storageKey) || '0');
    if (dismissals >= maxDismissals) {
      return;
    }

    // Detect iOS
    const userAgent = window.navigator.userAgent.toLowerCase();
    setIsIOS(/iphone|ipad|ipod/.test(userAgent));

    // Check if already installed
    const isStandalone = window.matchMedia('(display-mode: standalone)').matches;
    if (isStandalone) {
      return;
    }

    // Show after delay
    const timer = setTimeout(() => {
      if (isInstallable || isIOS) {
        setIsVisible(true);
      }
    }, delay);

    return () => {
      clearTimeout(timer);
      installPromptManager.removeListener(handleInstallabilityChange);
    };
  }, [delay, storageKey, maxDismissals, isInstallable]);

  const handleDismiss = () => {
    setIsVisible(false);
    const current = parseInt(localStorage.getItem(storageKey) || '0');
    localStorage.setItem(storageKey, (current + 1).toString());
  };

  const handleInstall = async () => {
    if (isIOS) return; // iOS shows instructions only
    
    const installed = await installPromptManager.showPrompt();
    if (installed) {
      setIsVisible(false);
    }
  };

  if (!isVisible) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 z-50 bg-gradient-to-r from-blue-500 to-purple-600 text-white p-4 shadow-lg">
      <div className="container mx-auto flex items-center justify-between">
        <div className="flex items-center space-x-4">
          <div className="hidden sm:block">
            <div className="w-12 h-12 bg-white/20 rounded-lg flex items-center justify-center">
              <ArrowDownTrayIcon className="w-6 h-6" />
            </div>
          </div>
          
          <div>
            <h3 className="font-bold text-lg">
              {isIOS ? 'Install Our App' : 'Install App for Better Experience'}
            </h3>
            <p className="text-sm opacity-90">
              {isIOS
                ? 'Add to your home screen for quick access'
                : 'Install our app to use it offline and get faster access'}
            </p>
            
            {isIOS && (
              <div className="mt-2 text-sm">
                Tap <span className="font-bold">Share</span> →{' '}
                <span className="font-bold">Add to Home Screen</span> →
                <span className="font-bold"> Add</span>
              </div>
            )}
          </div>
        </div>

        <div className="flex items-center space-x-3">
          {!isIOS && (
            <button
              onClick={handleInstall}
              className="bg-white text-blue-600 px-4 py-2 rounded-lg font-semibold hover:bg-gray-100 transition-colors"
            >
              Install Now
            </button>
          )}
          
          <button
            onClick={handleDismiss}
            className="p-2 hover:bg-white/20 rounded-full transition-colors"
            aria-label="Dismiss"
          >
            <XMarkIcon className="w-5 h-5" />
          </button>
        </div>
      </div>
    </div>
  );
}
iOS install instructions
typescript
// components/pwa/ios-install-guide.tsx
'use client';

import { useState, useEffect } from 'react';
import { DevicePhoneMobileIcon, ShareIcon, SquaresPlusIcon, CheckIcon } from '@heroicons/react/24/outline';

export default function IOSInstallGuide() {
  const [isVisible, setIsVisible] = useState(false);
  const [isIOS, setIsIOS] = useState(false);
  const [isStandalone, setIsStandalone] = useState(false);

  useEffect(() => {
    const userAgent = window.navigator.userAgent.toLowerCase();
    const ios = /iphone|ipad|ipod/.test(userAgent);
    setIsIOS(ios);

    const standalone = window.matchMedia('(display-mode: standalone)').matches;
    setIsStandalone(standalone);

    // Check if guide was shown before
    const shown = localStorage.getItem('ios-install-guide-shown');
    if (ios && !standalone && !shown) {
      setIsVisible(true);
    }
  }, []);

  const handleClose = () => {
    setIsVisible(false);
    localStorage.setItem('ios-install-guide-shown', 'true');
  };

  const handleDontShowAgain = () => {
    setIsVisible(false);
    localStorage.setItem('ios-install-guide-shown', 'true');
    // Set for 30 days
    const expiry = new Date();
    expiry.setDate(expiry.getDate() + 30);
    localStorage.setItem('ios-install-guide-expiry', expiry.toISOString());
  };

  if (!isVisible) return null;

  const steps = [
    {
      icon: ShareIcon,
      title: 'Tap Share Button',
      description: 'Open Safari menu and tap the share icon'
    },
    {
      icon: SquaresPlusIcon,
      title: 'Add to Home Screen',
      description: 'Scroll down and tap "Add to Home Screen"'
    },
    {
      icon: CheckIcon,
      title: 'Confirm Installation',
      description: 'Tap "Add" in the top right corner'
    }
  ];

  return (
    <div className="fixed inset-0 z-50 bg-black/50 flex items-center justify-center p-4">
      <div className="bg-white rounded-2xl max-w-md w-full overflow-hidden">
        <div className="p-6">
          <div className="flex items-center justify-between mb-6">
            <div className="flex items-center space-x-3">
              <div className="w-10 h-10 bg-blue-100 rounded-lg flex items-center justify-center">
                <DevicePhoneMobileIcon className="w-6 h-6 text-blue-600" />
              </div>
              <h2 className="text-xl font-bold text-gray-900">
                Install on iOS
              </h2>
            </div>
          </div>

          <div className="space-y-6">
            {steps.map((step, index) => (
              <div key={index} className="flex items-start space-x-4">
                <div className="flex-shrink-0">
                  <div className="w-12 h-12 bg-blue-50 rounded-lg flex items-center justify-center">
                    <step.icon className="w-6 h-6 text-blue-600" />
                  </div>
                </div>
                <div>
                  <h3 className="font-semibold text-gray-900">
                    Step {index + 1}: {step.title}
                  </h3>
                  <p className="text-gray-600 mt-1">
                    {step.description}
                  </p>
                </div>
              </div>
            ))}
          </div>

          <div className="mt-8 p-4 bg-blue-50 rounded-lg">
            <p className="text-sm text-blue-800">
              <strong>Tip:</strong> After installation, the app will work offline 
              and launch instantly like a native app.
            </p>
          </div>
        </div>

        <div className="border-t px-6 py-4 bg-gray-50">
          <div className="flex justify-between">
            <button
              onClick={handleDontShowAgain}
              className="px-4 py-2 text-gray-600 hover:text-gray-800 font-medium"
            >
              Don't show again
            </button>
            <button
              onClick={handleClose}
              className="px-4 py-2 bg-blue-600 text-white rounded-lg font-medium hover:bg-blue-700 transition-colors"
            >
              Got it!
            </button>
          </div>
        </div>
      </div>
    </div>
  );
}
Install analytics
typescript
// lib/analytics/pwa-analytics.ts
export interface InstallEventParams {
  platform?: string;
  userAgent?: string;
  referrer?: string;
  screenSize?: string;
  language?: string;
}

export class PWAAnalytics {
  private static instance: PWAAnalytics;

  private constructor() {
    this.setupEventListeners();
  }

  public static getInstance(): PWAAnalytics {
    if (!PWAAnalytics.instance) {
      PWAAnalytics.instance = new PWAAnalytics();
    }
    return PWAAnalytics.instance;
  }

  private setupEventListeners(): void {
    if (typeof window === 'undefined') return;

    // Track beforeinstallprompt
    window.addEventListener('beforeinstallprompt', (e) => {
      this.trackEvent('pwa_install_available', {
        platform: this.getPlatform(),
        userAgent: navigator.userAgent.substring(0, 100)
      });
    });

    // Track app installed
    window.addEventListener('appinstalled', () => {
      this.trackEvent('pwa_installed', {
        platform: this.getPlatform(),
        displayMode: this.getDisplayMode(),
        timestamp: new Date().toISOString()
      });
    });

    // Track display mode changes
    const mediaQuery = window.matchMedia('(display-mode: standalone)');
    mediaQuery.addEventListener('change', (e) => {
      this.trackEvent('pwa_display_mode_changed', {
        mode: e.matches ? 'standalone' : 'browser'
      });
    });

    // Track engagement metrics
    this.trackEngagement();
  }

  private getPlatform(): string {
    const ua = navigator.userAgent;
    if (/android/i.test(ua)) return 'android';
    if (/iPad|iPhone|iPod/.test(ua)) return 'ios';
    if (/Windows/.test(ua)) return 'windows';
    if (/Mac/.test(ua)) return 'mac';
    if (/Linux/.test(ua)) return 'linux';
    return 'unknown';
  }

  private getDisplayMode(): string {
    if (typeof window === 'undefined') return 'browser';
    
    const isStandalone = window.matchMedia('(display-mode: standalone)').matches;
    const isFullscreen = window.matchMedia('(display-mode: fullscreen)').matches;
    const isMinimalUI = window.matchMedia('(display-mode: minimal-ui)').matches;

    if (isStandalone) return 'standalone';
    if (isFullscreen) return 'fullscreen';
    if (isMinimalUI) return 'minimal-ui';
    return 'browser';
  }

  public trackInstallPromptShown(params?: InstallEventParams): void {
    this.trackEvent('pwa_install_prompt_shown', {
      ...params,
      platform: this.getPlatform(),
      displayMode: this.getDisplayMode()
    });
  }

  public trackInstallChoice(accepted: boolean, params?: InstallEventParams): void {
    this.trackEvent('pwa_install_choice', {
      ...params,
      accepted,
      platform: this.getPlatform(),
      displayMode: this.getDisplayMode()
    });
  }

  public trackInstallError(error: string, params?: InstallEventParams): void {
    this.trackEvent('pwa_install_error', {
      ...params,
      error,
      platform: this.getPlatform(),
      displayMode: this.getDisplayMode()
    });
  }

  private trackEvent(eventName: string, params: Record<string, any> = {}): void {
    // Integrate with your analytics provider
    const eventData = {
      event: eventName,
      ...params,
      timestamp: new Date().toISOString(),
      page: window.location.pathname,
      pwa: true
    };

    // Google Analytics 4
    if ((window as any).gtag) {
      (window as any).gtag('event', eventName, params);
    }

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.log('[PWA Analytics]', eventName, eventData);
    }

    // Send to your backend
    this.sendToBackend(eventData);
  }

  private async sendToBackend(data: Record<string, any>): Promise<void> {
    try {
      await fetch('/api/analytics/pwa-events', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
        keepalive: true // Ensure send on page unload
      });
    } catch (error) {
      console.error('Failed to send analytics:', error);
    }
  }

  private trackEngagement(): void {
    let startTime = Date.now();
    let hasSentEngagement = false;

    // Track session start
    this.trackEvent('pwa_session_start', {
      displayMode: this.getDisplayMode(),
      platform: this.getPlatform()
    });

    // Track engagement after 10 seconds
    const engagementTimer = setTimeout(() => {
      if (!hasSentEngagement) {
        this.trackEvent('pwa_engaged', {
          duration: 10,
          displayMode: this.getDisplayMode()
        });
        hasSentEngagement = true;
      }
    }, 10000);

    // Track beforeunload
    window.addEventListener('beforeunload', () => {
      const duration = Math.floor((Date.now() - startTime) / 1000);
      this.trackEvent('pwa_session_end', {
        duration,
        displayMode: this.getDisplayMode()
      });
      clearTimeout(engagementTimer);
    });
  }
}

// Hook for React components
export function usePWAAnalytics() {
  useEffect(() => {
    const analytics = PWAAnalytics.getInstance();
    
    // Track page view in PWA
    const displayMode = window.matchMedia('(display-mode: standalone)').matches ? 'standalone' : 'browser';
    analytics.trackEvent('pwa_page_view', {
      page: window.location.pathname,
      displayMode
    });

    // Track PWA-specific interactions
    const handleClick = (e: MouseEvent) => {
      const target = e.target as HTMLElement;
      if (target.closest('[data-pwa-action]')) {
        const action = target.closest('[data-pwa-action]')?.getAttribute('data-pwa-action');
        analytics.trackEvent(`pwa_action_${action}`, {
          element: target.tagName,
          text: target.textContent?.substring(0, 50)
        });
      }
    };

    document.addEventListener('click', handleClick);
    return () => document.removeEventListener('click', handleClick);
  }, []);
}
Post-install experience
typescript
// lib/pwa/post-install.ts
export class PostInstallExperience {
  private static instance: PostInstallExperience;
  private isFirstRun = false;
  private isStandalone = false;

  private constructor() {
    this.initialize();
  }

  public static getInstance(): PostInstallExperience {
    if (!PostInstallExperience.instance) {
      PostInstallExperience.instance = new PostInstallExperience();
    }
    return PostInstallExperience.instance;
  }

  private initialize(): void {
    if (typeof window === 'undefined') return;

    // Check if running in standalone mode
    this.isStandalone = window.matchMedia('(display-mode: standalone)').matches;

    // Check if first run after install
    const installed = localStorage.getItem('pwa-installed');
    if (this.isStandalone && !installed) {
      this.isFirstRun = true;
      localStorage.setItem('pwa-installed', 'true');
      this.onFirstRun();
    }

    // Listen for app installed event
    window.addEventListener('appinstalled', this.handleAppInstalled);
    
    // Listen for display mode changes
    window.matchMedia('(display-mode: standalone)').addEventListener('change', (e) => {
      this.isStandalone = e.matches;
      this.onDisplayModeChange(e.matches);
    });
  }

  private handleAppInstalled = (): void => {
    this.isFirstRun = true;
    this.isStandalone = true;
    localStorage.setItem('pwa-installed', 'true');
    this.onFirstRun();
  };

  private onFirstRun(): void {
    // Show welcome message
    this.showWelcomeMessage();
    
    // Track installation
    this.trackEvent('first_run_after_install');
    
    // Set up first run preferences
    this.setupFirstRunPreferences();
  }

  private onDisplayModeChange(isStandalone: boolean): void {
    if (isStandalone) {
      this.trackEvent('entered_standalone_mode');
      this.optimizeForStandalone();
    } else {
      this.trackEvent('exited_standalone_mode');
    }
  }

  private showWelcomeMessage(): void {
    // Check if we should show welcome message
    const welcomeShown = localStorage.getItem('pwa-welcome-shown');
    if (welcomeShown) return;

    // Create and show welcome modal
    const modal = document.createElement('div');
    modal.innerHTML = `
      <div style="position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0,0,0,0.5); z-index: 9999; display: flex; align-items: center; justify-content: center;">
        <div style="background: white; border-radius: 16px; padding: 24px; max-width: 400px; margin: 16px;">
          <h2 style="font-size: 1.5rem; font-weight: bold; margin-bottom: 16px;">Welcome to Our App! 🎉</h2>
          <p style="margin-bottom: 20px; color: #666;">
            Thank you for installing our app. Here's what you can do:
          </p>
          <ul style="margin-bottom: 24px; color: #666;">
            <li style="margin-bottom: 8px;">✓ Use offline</li>
            <li style="margin-bottom: 8px;">✓ Receive push notifications</li>
            <li style="margin-bottom: 8px;">✓ Faster loading times</li>
            <li style="margin-bottom: 8px;">✓ Home screen access</li>
          </ul>
          <button id="welcome-close" style="background: #3b82f6; color: white; border: none; padding: 12px 24px; border-radius: 8px; font-weight: 600; cursor: pointer; width: 100%;">
            Get Started
          </button>
        </div>
      </div>
    `;

    document.body.appendChild(modal);

    modal.querySelector('#welcome-close')?.addEventListener('click', () => {
      document.body.removeChild(modal);
      localStorage.setItem('pwa-welcome-shown', 'true');
    });
  }

  private setupFirstRunPreferences(): void {
    // Enable features by default on first run
    const preferences = {
      offlineMode: true,
      pushNotifications: true,
      backgroundSync: true,
      highResolutionImages: navigator.connection?.saveData ? false : true
    };

    localStorage.setItem('pwa-preferences', JSON.stringify(preferences));
  }

  private optimizeForStandalone(): void {
    // Adjust UI for standalone mode
    document.documentElement.style.setProperty('--standalone-padding', 'env(safe-area-inset-top)');
    
    // Enable standalone-only features
    if ('getInstalledRelatedApps' in navigator) {
      this.checkForRelatedApps();
    }
  }

  private async checkForRelatedApps(): Promise<void> {
    try {
      const relatedApps = await (navigator as any).getInstalledRelatedApps();
      if (relatedApps && relatedApps.length > 0) {
        this.trackEvent('related_apps_installed', {
          count: relatedApps.length
        });
      }
    } catch (error) {
      // Feature not supported
    }
  }

  private trackEvent(event: string, params?: Record<string, any>): void {
    // Integrate with analytics
    console.log('[Post-Install]', event, params);
  }

  // Public API
  public isFirstRunAfterInstall(): boolean {
    return this.isFirstRun;
  }

  public isInStandaloneMode(): boolean {
    return this.isStandalone;
  }

  public getPreferences(): any {
    const prefs = localStorage.getItem('pwa-preferences');
    return prefs ? JSON.parse(prefs) : null;
  }

  public updatePreference(key: string, value: any): void {
    const prefs = this.getPreferences() || {};
    prefs[key] = value;
    localStorage.setItem('pwa-preferences', JSON.stringify(prefs));
  }
}

// React hook for post-install features
export function usePostInstall() {
  const [isStandalone, setIsStandalone] = useState(false);
  const [isFirstRun, setIsFirstRun] = useState(false);
  const [preferences, setPreferences] = useState<any>(null);

  useEffect(() => {
    const postInstall = PostInstallExperience.getInstance();
    
    setIsStandalone(postInstall.isInStandaloneMode());
    setIsFirstRun(postInstall.isFirstRunAfterInstall());
    setPreferences(postInstall.getPreferences());

    // Update when display mode changes
    const mediaQuery = window.matchMedia('(display-mode: standalone)');
    const handleChange = (e: MediaQueryListEvent) => {
      setIsStandalone(e.matches);
    };
    
    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  const updatePreference = useCallback((key: string, value: any) => {
    const postInstall = PostInstallExperience.getInstance();
    postInstall.updatePreference(key, value);
    setPreferences(postInstall.getPreferences());
  }, []);

  return {
    isStandalone,
    isFirstRun,
    preferences,
    updatePreference
  };
}
§6. PUSH NOTIFICATIONS
VAPID keys generation
typescript
// scripts/generate-vapid-keys.ts
import webpush from 'web-push';
import fs from 'fs';
import path from 'path';

export function generateVAPIDKeys(): { publicKey: string; privateKey: string } {
  const vapidKeys = webpush.generateVAPIDKeys();
  
  return {
    publicKey: vapidKeys.publicKey,
    privateKey: vapidKeys.privateKey
  };
}

export function saveVAPIDKeysToEnv(keys: { publicKey: string; privateKey: string }): void {
  const envPath = path.join(process.cwd(), '.env.local');
  let envContent = '';
  
  if (fs.existsSync(envPath)) {
    envContent = fs.readFileSync(envPath, 'utf8');
  }
  
  // Remove existing VAPID keys
  envContent = envContent.replace(/NEXT_PUBLIC_VAPID_PUBLIC_KEY=.*\n/g, '');
  envContent = envContent.replace(/VAPID_PRIVATE_KEY=.*\n/g, '');
  
  // Add new keys
  envContent += `NEXT_PUBLIC_VAPID_PUBLIC_KEY=${keys.publicKey}\n`;
  envContent += `VAPID_PRIVATE_KEY=${keys.privateKey}\n`;
  
  fs.writeFileSync(envPath, envContent);
  console.log('VAPID keys generated and saved to .env.local');
}

export function saveVAPIDKeysToFile(keys: { publicKey: string; privateKey: string }): void {
  const outputPath = path.join(process.cwd(), 'vapid-keys.json');
  fs.writeFileSync(outputPath, JSON.stringify(keys, null, 2));
  console.log(`VAPID keys saved to ${outputPath}`);
}

// CLI execution
if (require.main === module) {
  const keys = generateVAPIDKeys();
  console.log('Generated VAPID Keys:');
  console.log('Public Key:', keys.publicKey);
  console.log('Private Key:', keys.privateKey);
  
  saveVAPIDKeysToEnv(keys);
  saveVAPIDKeysToFile(keys);
}

// Type definitions for web-push
declare module 'web-push' {
  export function generateVAPIDKeys(): { publicKey: string; privateKey: string };
  export function setVapidDetails(
    subject: string,
    publicKey: string,
    privateKey: string
  ): void;
}
typescript
// lib/push/vapid-config.ts
import webpush from 'web-push';

export class VAPIDConfig {
  private static instance: VAPIDConfig;
  
  public readonly publicKey: string;
  private readonly privateKey: string;
  public readonly subject: string;

  private constructor() {
    this.publicKey = process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY || '';
    this.privateKey = process.env.VAPID_PRIVATE_KEY || '';
    this.subject = process.env.VAPID_SUBJECT || `mailto:${process.env.NEXT_PUBLIC_SUPPORT_EMAIL || 'admin@example.com'}`;

    if (!this.publicKey || !this.privateKey) {
      throw new Error('VAPID keys are not configured. Run npm run generate-vapid-keys');
    }

    this.initializeWebPush();
  }

  public static getInstance(): VAPIDConfig {
    if (!VAPIDConfig.instance) {
      VAPIDConfig.instance = new VAPIDConfig();
    }
    return VAPIDConfig.instance;
  }

  private initializeWebPush(): void {
    webpush.setVapidDetails(
      this.subject,
      this.publicKey,
      this.privateKey
    );
  }

  public getConfig() {
    return {
      publicKey: this.publicKey,
      subject: this.subject
    };
  }

  public getWebPush() {
    return webpush;
  }
}
Subscription management
typescript
// lib/push/subscription-manager.ts
export interface PushSubscriptionData {
  endpoint: string;
  keys: {
    p256dh: string;
    auth: string;
  };
}

export interface SubscriptionMetadata {
  userId?: string;
  userAgent: string;
  platform: string;
  language: string;
  createdAt: Date;
  lastUsed: Date;
  notificationPreferences: {
    enabled: boolean;
    categories: string[];
    quietHours?: {
      start: string;
      end: string;
    };
  };
}

export class SubscriptionManager {
  private db: IDBDatabase | null = null;
  private readonly dbName = 'PushSubscriptionsDB';
  private readonly storeName = 'subscriptions';

  constructor() {
    this.initializeDB();
  }

  private async initializeDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };

      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        
        // Create object store
        const store = db.createObjectStore(this.storeName, { keyPath: 'endpoint' });
        
        // Create indexes
        store.createIndex('userId', 'userId', { unique: false });
        store.createIndex('createdAt', 'createdAt', { unique: false });
        store.createIndex('platform', 'platform', { unique: false });
        
        console.log('Push subscription database initialized');
      };
    });
  }

  public async subscribe(
    subscription: PushSubscription,
    metadata: Omit<SubscriptionMetadata, 'createdAt' | 'lastUsed'>
  ): Promise<void> {
    await this.waitForDB();

    const subscriptionData: PushSubscriptionData = {
      endpoint: subscription.endpoint,
      keys: {
        p256dh: this.arrayBufferToBase64(subscription.getKey('p256dh')!),
        auth: this.arrayBufferToBase64(subscription.getKey('auth')!)
      }
    };

    const fullMetadata: SubscriptionMetadata & { subscription: PushSubscriptionData } = {
      ...metadata,
      createdAt: new Date(),
      lastUsed: new Date(),
      subscription: subscriptionData
    };

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      
      const request = store.put(fullMetadata);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        // Also send to backend
        this.sendToBackend(subscriptionData, metadata);
        resolve();
      };
    });
  }

  public async unsubscribe(endpoint: string): Promise<void> {
    await this.waitForDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readwrite');
      const store = transaction.objectStore(this.storeName);
      
      const request = store.delete(endpoint);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        // Notify backend
        this.notifyBackendUnsubscribe(endpoint);
        resolve();
      };
    });
  }

  public async getSubscription(endpoint: string): Promise<(SubscriptionMetadata & { subscription: PushSubscriptionData }) | null> {
    await this.waitForDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      
      const request = store.get(endpoint);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result || null);
    });
  }

  public async getAllSubscriptions(): Promise<(SubscriptionMetadata & { subscription: PushSubscriptionData })[]> {
    await this.waitForDB();

    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction([this.storeName], 'readonly');
      const store = transaction.objectStore(this.storeName);
      
      const request = store.getAll();

      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve(request.result || []);
    });
  }

  public async updateLastUsed(endpoint: string): Promise<void> {
    await this.waitForDB();

    const subscription = await this.getSubscription(endpoint

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-26-PWA-OFFLINE
Prompt ID: 26 / 48
Parte: 2
Exported: 2026-02-06T11:37:03.571Z
Characters: 1521
════════════════════════════════════════════════════════════

payload.type);
  }
}

async function handleBackgroundSync(data: any) {
  // Perform background sync
  if ('sync' in self.registration) {
    try {
      await (self.registration as any).sync.register('background-sync');
    } catch (error) {
      console.error('Background sync failed:', error);
    }
  }
}

async function handleBackgroundUpdate(data: any) {
  // Check for updates
  try {
    const cache = await caches.open('app-shell');
    const response = await fetch('/api/version');
    const version = await response.text();
    
    const cachedVersion = await cache.match('/version.txt');
    
    if (!cachedVersion || (await cachedVersion.text()) !== version) {
      // Update available
      self.clients.matchAll().then((clients) => {
        clients.forEach((client) => {
          client.postMessage({
            type: 'UPDATE_AVAILABLE',
            version: version
          });
        });
      });
    }
  } catch (error) {
    console.error('Background update check failed:', error);
  }
}

async function handleBackgroundFetch(data: any) {
  // Fetch data in background
  if (data.urls && Array.isArray(data.urls)) {
    const cache = await caches.open('background-data');
    
    for (const url of data.urls) {
      try {
        const response = await fetch(url);
        if (response.ok) {
          await cache.put(url, response.clone());
          
          // Notify clients
          self.clients.matchAll().then((clients) => {
            clients.forEach((client) => {
             

## § ADVANCED PATTERNS: PWA OFFLINE

### Server Actions con Validazione

```typescript
// app/actions/pwa-offline.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### PWA OFFLINE - Utility Helper #689

```typescript
// lib/utils/pwa-offline-helper-689.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #813

```typescript
// lib/utils/pwa-offline-helper-813.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #235

```typescript
// lib/utils/pwa-offline-helper-235.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #398

```typescript
// lib/utils/pwa-offline-helper-398.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #440

```typescript
// lib/utils/pwa-offline-helper-440.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #650

```typescript
// lib/utils/pwa-offline-helper-650.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #642

```typescript
// lib/utils/pwa-offline-helper-642.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #602

```typescript
// lib/utils/pwa-offline-helper-602.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #556

```typescript
// lib/utils/pwa-offline-helper-556.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #768

```typescript
// lib/utils/pwa-offline-helper-768.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #449

```typescript
// lib/utils/pwa-offline-helper-449.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #281

```typescript
// lib/utils/pwa-offline-helper-281.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #687

```typescript
// lib/utils/pwa-offline-helper-687.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #921

```typescript
// lib/utils/pwa-offline-helper-921.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #979

```typescript
// lib/utils/pwa-offline-helper-979.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PWA OFFLINE - Utility Helper #818

```typescript
// lib/utils/pwa-offline-helper-818.ts
import { z } from "zod";

interface PWAOFFLINEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class PWAOFFLINEProcessor<TInput, TOutput> {
  private config: PWAOFFLINEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PWAOFFLINEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<PWAOFFLINEConfig> {
    return Object.freeze({ ...this.con