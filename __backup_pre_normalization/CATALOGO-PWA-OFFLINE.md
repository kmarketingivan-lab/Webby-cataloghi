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
             