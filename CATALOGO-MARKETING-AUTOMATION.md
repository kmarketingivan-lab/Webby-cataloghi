# CATALOGO-MARKETING-AUTOMATION

Next.js 14 Marketing Automation & Tracking Catalogo
§ TRACKING FUNDAMENTALS
First-party vs Third-party Tracking
typescript
// lib/tracking/types.ts
export enum TrackingType {
  FIRST_PARTY = 'first_party',
  THIRD_PARTY = 'third_party'
}

export interface TrackingConfig {
  type: TrackingType;
  endpoint: string;
  requiresConsent: boolean;
  dataRetentionDays: number;
  privacyPolicyUrl: string;
}

export const firstPartyConfig: TrackingConfig = {
  type: TrackingType.FIRST_PARTY,
  endpoint: '/api/collect',
  requiresConsent: true,
  dataRetentionDays: 365,
  privacyPolicyUrl: '/privacy'
};

export const thirdPartyConfig: TrackingConfig = {
  type: TrackingType.THIRD_PARTY,
  endpoint: 'https://analytics.google.com/g/collect',
  requiresConsent: true,
  dataRetentionDays: 730,
  privacyPolicyUrl: 'https://policies.google.com/privacy'
};
Cookie Consent Integration
typescript
// components/consent/CookieBanner.tsx
'use client';

import { useState, useEffect } from 'react';
import { createConsent, getConsent, updateConsent } from '@/lib/consent';

export default function CookieBanner() {
  const [showBanner, setShowBanner] = useState(false);
  const [preferences, setPreferences] = useState({
    necessary: true,
    analytics: false,
    marketing: false,
    personalization: false
  });

  useEffect(() => {
    const consent = getConsent();
    if (!consent) {
      setShowBanner(true);
    }
  }, []);

  const handleAcceptAll = () => {
    const allAccepted = {
      necessary: true,
      analytics: true,
      marketing: true,
      personalization: true,
      timestamp: new Date().toISOString()
    };
    createConsent(allAccepted);
    setShowBanner(false);
    window.location.reload();
  };

  const handleSavePreferences = () => {
    updateConsent({
      ...preferences,
      timestamp: new Date().toISOString()
    });
    setShowBanner(false);
    window.location.reload();
  };

  if (!showBanner) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t shadow-lg p-4 z-50">
      <div className="max-w-6xl mx-auto">
        <h3 className="text-lg font-semibold mb-2">Cookie Preferences</h3>
        <p className="text-sm text-gray-600 mb-4">
          We use cookies to improve your experience. Choose which cookies you allow.
        </p>
        
        <div className="space-y-3 mb-6">
          {Object.entries(preferences).map(([key, value]) => (
            <div key={key} className="flex items-center justify-between">
              <div>
                <span className="font-medium capitalize">{key}</span>
                <p className="text-sm text-gray-500">
                  {key === 'necessary' ? 'Required for site functionality' :
                   key === 'analytics' ? 'Helps us understand user behavior' :
                   'Used for personalized experiences'}
                </p>
              </div>
              <input
                type="checkbox"
                checked={value}
                disabled={key === 'necessary'}
                onChange={(e) => setPreferences(prev => ({
                  ...prev,
                  [key]: e.target.checked
                }))}
                className="h-5 w-5 rounded"
              />
            </div>
          ))}
        </div>

        <div className="flex gap-4">
          <button
            onClick={handleAcceptAll}
            className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
          >
            Accept All
          </button>
          <button
            onClick={handleSavePreferences}
            className="px-4 py-2 bg-gray-200 rounded hover:bg-gray-300"
          >
            Save Preferences
          </button>
          <button
            onClick={() => setShowBanner(false)}
            className="px-4 py-2 text-gray-600 hover:text-gray-800"
          >
            Reject Non-Essential
          </button>
        </div>
      </div>
    </div>
  );
}
Server-side Tracking
typescript
// app/api/collect/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { validateEvent, storeEvent } from '@/lib/analytics/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validation = validateEvent(body);
    
    if (!validation.valid) {
      return NextResponse.json(
        { error: validation.error },
        { status: 400 }
      );
    }

    // Store in database
    await storeEvent(body);
    
    // Forward to analytics providers if consented
    if (body.consent?.analytics) {
      // Forward to GA4
      await fetch('https://www.google-analytics.com/g/collect', {
        method: 'POST',
        body: new URLSearchParams({
          v: '2',
          tid: process.env.GA4_MEASUREMENT_ID!,
          cid: body.clientId,
          en: body.eventName,
          ...body.params
        })
      });
    }

    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Tracking error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// lib/analytics/server.ts
export async function trackServerEvent(event: ServerEvent) {
  const endpoint = process.env.NEXT_PUBLIC_TRACKING_ENDPOINT || '/api/collect';
  
  const payload = {
    ...event,
    userAgent: event.userAgent,
    ip: event.ip,
    timestamp: new Date().toISOString(),
    pageUrl: event.pageUrl,
    referrer: event.referrer,
    utm: event.utm || {}
  };

  // Store in your database
  await prisma?.analyticsEvent.create({
    data: {
      eventName: event.eventName,
      eventData: payload,
      userId: event.userId,
      sessionId: event.sessionId,
      pagePath: event.pageUrl,
      utmSource: event.utm?.source,
      utmMedium: event.utm?.medium,
      utmCampaign: event.utm?.campaign,
      createdAt: new Date()
    }
  });

  return payload;
}
UTM Parameters Handling
typescript
// lib/analytics/utm.ts
export interface UTMParams {
  source?: string;
  medium?: string;
  campaign?: string;
  term?: string;
  content?: string;
}

export function extractUTMParams(url: string): UTMParams {
  const parsedUrl = new URL(url, 'http://localhost');
  const params = parsedUrl.searchParams;
  
  return {
    source: params.get('utm_source') || undefined,
    medium: params.get('utm_medium') || undefined,
    campaign: params.get('utm_campaign') || undefined,
    term: params.get('utm_term') || undefined,
    content: params.get('utm_content') || undefined
  };
}

export function storeUTMParams(utm: UTMParams) {
  if (typeof window === 'undefined') return;
  
  // Store in cookies (30 days expiration)
  const expires = new Date();
  expires.setDate(expires.getDate() + 30);
  
  document.cookie = `utm_source=${utm.source || ''}; expires=${expires.toUTCString()}; path=/`;
  document.cookie = `utm_medium=${utm.medium || ''}; expires=${expires.toUTCString()}; path=/`;
  document.cookie = `utm_campaign=${utm.campaign || ''}; expires=${expires.toUTCString()}; path=/`;
  
  // Store in localStorage for persistence
  localStorage.setItem('utm_params', JSON.stringify({
    ...utm,
    firstTouch: new Date().toISOString()
  }));
}

export function getUTMParams(): UTMParams & { firstTouch?: string } {
  if (typeof window === 'undefined') return {};
  
  const stored = localStorage.getItem('utm_params');
  if (stored) {
    return JSON.parse(stored);
  }
  
  // Fallback to cookies
  return {
    source: getCookie('utm_source'),
    medium: getCookie('utm_medium'),
    campaign: getCookie('utm_campaign')
  };
}

function getCookie(name: string): string | undefined {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop()?.split(';').shift();
}
§ GOOGLE ANALYTICS 4
GA4 Setup per Next.js
typescript
// lib/analytics/ga4.ts
import { ConsentState } from '@/lib/consent';

declare global {
  interface Window {
    dataLayer: any[];
    gtag: (...args: any[]) => void;
  }
}

export class GA4Tracker {
  private measurementId: string;
  private enabled: boolean = false;

  constructor(measurementId: string) {
    this.measurementId = measurementId;
    this.initialize();
  }

  private initialize() {
    if (typeof window === 'undefined') return;

    // Load gtag script
    const script = document.createElement('script');
    script.src = `https://www.googletagmanager.com/gtag/js?id=${this.measurementId}`;
    script.async = true;
    document.head.appendChild(script);

    window.dataLayer = window.dataLayer || [];
    window.gtag = function() {
      window.dataLayer.push(arguments);
    };

    window.gtag('js', new Date());
    window.gtag('config', this.measurementId, {
      page_path: window.location.pathname,
      transport_type: 'beacon',
      allow_google_signals: false,
      allow_ad_personalization_signals: false
    });

    this.enabled = true;
  }

  public trackPageView(path: string, title?: string) {
    if (!this.enabled) return;
    
    window.gtag('event', 'page_view', {
      page_title: title || document.title,
      page_location: window.location.href,
      page_path: path
    });
  }

  public trackEvent(eventName: string, params?: Record<string, any>) {
    if (!this.enabled) return;
    
    window.gtag('event', eventName, {
      ...params,
      send_to: this.measurementId
    });
  }

  public setUserProperties(properties: Record<string, any>) {
    if (!this.enabled) return;
    
    window.gtag('set', 'user_properties', properties);
  }

  public setUserId(userId: string) {
    if (!this.enabled) return;
    
    window.gtag('set', 'user_id', userId);
  }

  public setConsent(consent: ConsentState) {
    if (!this.enabled) return;
    
    window.gtag('consent', 'update', {
      analytics_storage: consent.analytics ? 'granted' : 'denied',
      ad_storage: consent.marketing ? 'granted' : 'denied',
      ad_user_data: consent.marketing ? 'granted' : 'denied',
      ad_personalization: consent.personalization ? 'granted' : 'denied'
    });
  }
}

// Hook for React components
export function useGA4() {
  const [tracker, setTracker] = useState<GA4Tracker | null>(null);

  useEffect(() => {
    if (process.env.NEXT_PUBLIC_GA4_MEASUREMENT_ID) {
      const ga4 = new GA4Tracker(process.env.NEXT_PUBLIC_GA4_MEASUREMENT_ID);
      setTracker(ga4);
    }
  }, []);

  return tracker;
}
Custom Events
typescript
// lib/analytics/events.ts
export enum EventCategory {
  ENGAGEMENT = 'engagement',
  ECOMMERCE = 'ecommerce',
  USER = 'user',
  SYSTEM = 'system'
}

export interface BaseEvent {
  category: EventCategory;
  action: string;
  label?: string;
  value?: number;
  properties?: Record<string, any>;
}

export const EcommerceEvents = {
  VIEW_ITEM: 'view_item',
  ADD_TO_CART: 'add_to_cart',
  REMOVE_FROM_CART: 'remove_from_cart',
  VIEW_CART: 'view_cart',
  BEGIN_CHECKOUT: 'begin_checkout',
  ADD_SHIPPING_INFO: 'add_shipping_info',
  ADD_PAYMENT_INFO: 'add_payment_info',
  PURCHASE: 'purchase',
  REFUND: 'refund'
} as const;

export const UserEvents = {
  SIGN_UP: 'sign_up',
  LOGIN: 'login',
  LOGOUT: 'logout',
  UPDATE_PROFILE: 'update_profile',
  SUBSCRIBE: 'subscribe',
  UNSUBSCRIBE: 'unsubscribe'
} as const;

export function trackEcommerceEvent(
  eventName: string,
  items: CartItem[],
  value?: number,
  currency: string = 'EUR'
) {
  const ga4Tracker = window.gtag;
  
  if (ga4Tracker) {
    ga4Tracker('event', eventName, {
      currency,
      value,
      items: items.map(item => ({
        item_id: item.id,
        item_name: item.name,
        price: item.price,
        quantity: item.quantity,
        category: item.category,
        variant: item.variant
      }))
    });
  }
}

// Enhanced ecommerce implementation
export class EnhancedEcommerceTracker {
  private impressionProducts: ProductImpression[] = [];
  private currentCart: CartItem[] = [];

  public trackProductImpression(product: ProductImpression) {
    this.impressionProducts.push(product);
    
    window.gtag?.('event', 'view_item_list', {
      item_list_name: product.listName,
      items: [{
        item_id: product.id,
        item_name: product.name,
        item_category: product.category,
        price: product.price,
        index: this.impressionProducts.length - 1
      }]
    });
  }

  public trackProductClick(product: Product, listName?: string) {
    window.gtag?.('event', 'select_item', {
      item_list_name: listName,
      items: [{
        item_id: product.id,
        item_name: product.name,
        item_category: product.category,
        price: product.price
      }]
    });
  }

  public addToCart(item: CartItem) {
    this.currentCart.push(item);
    
    window.gtag?.('event', 'add_to_cart', {
      currency: 'EUR',
      value: item.price * item.quantity,
      items: [{
        item_id: item.id,
        item_name: item.name,
        item_category: item.category,
        price: item.price,
        quantity: item.quantity
      }]
    });
  }
}
User Properties
typescript
// lib/analytics/user.ts
export interface UserProperties {
  user_id?: string;
  email?: string;
  name?: string;
  role?: string;
  subscription_tier?: 'free' | 'premium' | 'enterprise';
  signup_date?: string;
  last_login_date?: string;
  total_orders?: number;
  total_spent?: number;
  account_age_days?: number;
  preferred_language?: string;
  device_category?: 'mobile' | 'desktop' | 'tablet';
  acquisition_channel?: string;
}

export class UserPropertyManager {
  private static instance: UserPropertyManager;
  private properties: UserProperties = {};

  private constructor() {}

  public static getInstance(): UserPropertyManager {
    if (!UserPropertyManager.instance) {
      UserPropertyManager.instance = new UserPropertyManager();
    }
    return UserPropertyManager.instance;
  }

  public setProperty(key: keyof UserProperties, value: any) {
    this.properties[key] = value;
    this.syncWithAnalytics();
  }

  public setProperties(properties: Partial<UserProperties>) {
    this.properties = { ...this.properties, ...properties };
    this.syncWithAnalytics();
  }

  public updateFromUser(user: any) {
    if (!user) return;

    const properties: Partial<UserProperties> = {
      user_id: user.id,
      email: user.email,
      name: user.name,
      role: user.role,
      subscription_tier: user.subscriptionTier,
      signup_date: user.createdAt,
      last_login_date: new Date().toISOString(),
      total_orders: user.orderCount,
      total_spent: user.totalSpent
    };

    // Calculate account age
    if (user.createdAt) {
      const signupDate = new Date(user.createdAt);
      const today = new Date();
      const diffTime = Math.abs(today.getTime() - signupDate.getTime());
      properties.account_age_days = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
    }

    this.setProperties(properties);
  }

  private syncWithAnalytics() {
    // Sync with GA4
    if (window.gtag) {
      window.gtag('set', 'user_properties', this.properties);
    }

    // Sync with Segment
    if (window.analytics) {
      window.analytics.identify(this.properties.user_id, this.properties);
    }

    // Store in localStorage
    localStorage.setItem('user_properties', JSON.stringify(this.properties));
  }

  public getProperties(): UserProperties {
    return { ...this.properties };
  }
}
§ SEGMENT/CDP
Segment Setup
typescript
// lib/segment/config.ts
export interface SegmentConfig {
  writeKey: string;
  cdnURL?: string;
  appName?: string;
  pageCategory?: string;
  defaultSettings?: {
    integrations: {
      [key: string]: boolean;
    };
  };
}

export class SegmentAnalytics {
  private static instance: SegmentAnalytics;
  private initialized = false;

  private constructor() {}

  public static getInstance(): SegmentAnalytics {
    if (!SegmentAnalytics.instance) {
      SegmentAnalytics.instance = new SegmentAnalytics();
    }
    return SegmentAnalytics.instance;
  }

  public initialize(config: SegmentConfig) {
    if (this.initialized || typeof window === 'undefined') return;

    // Load Segment script
    const script = document.createElement('script');
    script.src = config.cdnURL || 'https://cdn.segment.com/analytics.js/v1/' + config.writeKey + '/analytics.min.js';
    script.async = true;
    document.head.appendChild(script);

    script.onload = () => {
      if (window.analytics) {
        // Configure Segment
        window.analytics.load(config.writeKey);
        window.analytics.page(config.pageCategory);
        
        // Set default integrations
        if (config.defaultSettings) {
          window.analytics.configure(config.defaultSettings);
        }

        this.initialized = true;
      }
    };
  }

  public page(category?: string, name?: string, properties?: Record<string, any>) {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.page(category, name, {
      ...properties,
      url: window.location.href,
      path: window.location.pathname,
      referrer: document.referrer,
      timestamp: new Date().toISOString()
    });
  }

  public track(event: string, properties?: Record<string, any>) {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.track(event, {
      ...properties,
      timestamp: new Date().toISOString()
    });
  }

  public identify(userId: string, traits?: Record<string, any>) {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.identify(userId, traits);
  }

  public group(groupId: string, traits?: Record<string, any>) {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.group(groupId, traits);
  }

  public alias(userId: string, previousId?: string) {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.alias(userId, previousId);
  }

  public reset() {
    if (!this.initialized || !window.analytics) return;
    
    window.analytics.reset();
  }
}

// React Hook
export function useSegment() {
  const [segment, setSegment] = useState<SegmentAnalytics | null>(null);

  useEffect(() => {
    const segmentInstance = SegmentAnalytics.getInstance();
    
    if (process.env.NEXT_PUBLIC_SEGMENT_WRITE_KEY) {
      segmentInstance.initialize({
        writeKey: process.env.NEXT_PUBLIC_SEGMENT_WRITE_KEY,
        appName: 'My Next.js App',
        defaultSettings: {
          integrations: {
            'Google Analytics': true,
            'Mixpanel': true,
            'Amplitude': true,
            'Facebook Pixel': false, // Disabled by default for privacy
            'LinkedIn Insight Tag': false
          }
        }
      });
      
      setSegment(segmentInstance);
    }
  }, []);

  return segment;
}
Destination Management
typescript
// lib/segment/destinations.ts
export interface DestinationConfig {
  id: string;
  name: string;
  enabled: boolean;
  settings: Record<string, any>;
  mapping?: Record<string, string>;
}

export class DestinationManager {
  private destinations: Map<string, DestinationConfig> = new Map();

  constructor() {
    this.loadDestinations();
  }

  private loadDestinations() {
    const defaultDestinations: DestinationConfig[] = [
      {
        id: 'google-analytics',
        name: 'Google Analytics 4',
        enabled: true,
        settings: {
          measurementId: process.env.NEXT_PUBLIC_GA4_MEASUREMENT_ID,
          sendPageView: true,
          enhancedEcommerce: true
        },
        mapping: {
          'page': 'page_view',
          'Product Viewed': 'view_item',
          'Product Added': 'add_to_cart',
          'Order Completed': 'purchase'
        }
      },
      {
        id: 'mixpanel',
        name: 'Mixpanel',
        enabled: true,
        settings: {
          projectToken: process.env.NEXT_PUBLIC_MIXPANEL_TOKEN,
          people: true,
          trackPageView: false
        }
      },
      {
        id: 'amplitude',
        name: 'Amplitude',
        enabled: true,
        settings: {
          apiKey: process.env.NEXT_PUBLIC_AMPLITUDE_API_KEY,
          includeReferrer: true,
          includeUtm: true
        }
      }
    ];

    defaultDestinations.forEach(dest => {
      this.destinations.set(dest.id, dest);
    });
  }

  public getDestination(id: string): DestinationConfig | undefined {
    return this.destinations.get(id);
  }

  public getAllDestinations(): DestinationConfig[] {
    return Array.from(this.destinations.values());
  }

  public updateDestination(id: string, updates: Partial<DestinationConfig>) {
    const destination = this.destinations.get(id);
    if (destination) {
      this.destinations.set(id, { ...destination, ...updates });
      this.saveDestinations();
    }
  }

  public toggleDestination(id: string, enabled: boolean) {
    this.updateDestination(id, { enabled });
  }

  private saveDestinations() {
    if (typeof window !== 'undefined') {
      const destinationsArray = this.getAllDestinations();
      localStorage.setItem('segment_destinations', JSON.stringify(destinationsArray));
    }
  }

  public syncWithSegment() {
    if (window.analytics) {
      const destinations = this.getAllDestinations();
      const integrations: Record<string, boolean> = {};
      
      destinations.forEach(dest => {
        integrations[dest.name] = dest.enabled;
      });
      
      window.analytics.configure({
        integrations
      });
    }
  }

  public forwardEvent(eventType: string, payload: any) {
    const destinations = this.getAllDestinations().filter(d => d.enabled);
    
    destinations.forEach(destination => {
      this.sendToDestination(destination, eventType, payload);
    });
  }

  private async sendToDestination(
    destination: DestinationConfig,
    eventType: string,
    payload: any
  ) {
    switch (destination.id) {
      case 'google-analytics':
        await this.sendToGA4(destination, eventType, payload);
        break;
      case 'mixpanel':
        await this.sendToMixpanel(destination, eventType, payload);
        break;
      case 'amplitude':
        await this.sendToAmplitude(destination, eventType, payload);
        break;
    }
  }

  private async sendToGA4(destination: DestinationConfig, eventType: string, payload: any) {
    const eventName = destination.mapping?.[eventType] || eventType;
    
    if (window.gtag) {
      window.gtag('event', eventName, {
        ...payload,
        send_to: destination.settings.measurementId
      });
    }
  }
}
Data Quality
typescript
// lib/segment/data-quality.ts
export interface DataValidationRule {
  field: string;
  type: 'string' | 'number' | 'boolean' | 'email' | 'timestamp';
  required: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
  minValue?: number;
  maxValue?: number;
}

export class DataQualityManager {
  private validationRules: Map<string, DataValidationRule[]> = new Map();

  constructor() {
    this.setupDefaultRules();
  }

  private setupDefaultRules() {
    // Page event rules
    this.validationRules.set('page', [
      { field: 'name', type: 'string', required: true, minLength: 1, maxLength: 200 },
      { field: 'category', type: 'string', required: false, maxLength: 100 },
      { field: 'properties', type: 'object', required: true },
      { field: 'timestamp', type: 'timestamp', required: true }
    ]);

    // Track event rules
    this.validationRules.set('track', [
      { field: 'event', type: 'string', required: true, minLength: 1, maxLength: 100 },
      { field: 'properties', type: 'object', required: true },
      { field: 'userId', type: 'string', required: false },
      { field: 'anonymousId', type: 'string', required: false }
    ]);

    // Identify event rules
    this.validationRules.set('identify', [
      { field: 'userId', type: 'string', required: false },
      { field: 'traits', type: 'object', required: true },
      { field: 'email', type: 'email', required: false }
    ]);
  }

  public validateEvent(eventType: string, data: any): ValidationResult {
    const rules = this.validationRules.get(eventType);
    if (!rules) {
      return { valid: true, warnings: ['No validation rules found'] };
    }

    const errors: string[] = [];
    const warnings: string[] = [];

    for (const rule of rules) {
      const value = data[rule.field];
      
      // Check required fields
      if (rule.required && (value === undefined || value === null || value === '')) {
        errors.push(`${rule.field} is required`);
        continue;
      }

      if (value !== undefined && value !== null) {
        // Type validation
        switch (rule.type) {
          case 'string':
            if (typeof value !== 'string') {
              errors.push(`${rule.field} must be a string`);
            } else {
              if (rule.minLength && value.length < rule.minLength) {
                errors.push(`${rule.field} must be at least ${rule.minLength} characters`);
              }
              if (rule.maxLength && value.length > rule.maxLength) {
                errors.push(`${rule.field} must be at most ${rule.maxLength} characters`);
              }
              if (rule.pattern && !rule.pattern.test(value)) {
                errors.push(`${rule.field} must match pattern ${rule.pattern}`);
              }
            }
            break;

          case 'number':
            if (typeof value !== 'number') {
              errors.push(`${rule.field} must be a number`);
            } else {
              if (rule.minValue !== undefined && value < rule.minValue) {
                errors.push(`${rule.field} must be at least ${rule.minValue}`);
              }
              if (rule.maxValue !== undefined && value > rule.maxValue) {
                errors.push(`${rule.field} must be at most ${rule.maxValue}`);
              }
            }
            break;

          case 'email':
            if (typeof value !== 'string') {
              errors.push(`${rule.field} must be a string`);
            } else {
              const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
              if (!emailRegex.test(value)) {
                warnings.push(`${rule.field} appears to be an invalid email`);
              }
            }
            break;

          case 'timestamp':
            const date = new Date(value);
            if (isNaN(date.getTime())) {
              errors.push(`${rule.field} must be a valid timestamp`);
            }
            break;
        }
      }
    }

    // Check for PII data
    this.checkForPII(data, warnings);

    return {
      valid: errors.length === 0,
      errors,
      warnings
    };
  }

  private checkForPII(data: any, warnings: string[]) {
    const piiFields = [
      'email', 'phone', 'ssn', 'password', 'creditCard',
      'address', 'ip', 'location', 'birthDate'
    ];

    const checkObject = (obj: any, path: string = '') => {
      if (!obj || typeof obj !== 'object') return;

      for (const [key, value] of Object.entries(obj)) {
        const fullPath = path ? `${path}.${key}` : key;
        
        if (piiFields.some(pii => key.toLowerCase().includes(pii))) {
          warnings.push(`Potential PII detected in field: ${fullPath}`);
        }

        if (typeof value === 'object' && value !== null) {
          checkObject(value, fullPath);
        }
      }
    };

    checkObject(data);
  }

  public sanitizeEvent(data: any): any {
    const sanitized = { ...data };
    
    // Remove sensitive data
    const sensitiveFields = ['password', 'creditCard', 'ssn', 'accessToken'];
    
    sensitiveFields.forEach(field => {
      if (sanitized[field] !== undefined) {
        delete sanitized[field];
      }
      if (sanitized.properties && sanitized.properties[field]) {
        delete sanitized.properties[field];
      }
    });

    // Truncate long strings
    this.truncateStrings(sanitized, 1000);

    return sanitized;
  }

  private truncateStrings(obj: any, maxLength: number) {
    for (const [key, value] of Object.entries(obj)) {
      if (typeof value === 'string' && value.length > maxLength) {
        obj[key] = value.substring(0, maxLength) + '...';
      } else if (typeof value === 'object' && value !== null) {
        this.truncateStrings(value, maxLength);
      }
    }
  }
}

export interface ValidationResult {
  valid: boolean;
  errors?: string[];
  warnings?: string[];
}
§ HEATMAPS & SESSION RECORDING
Hotjar Integration
typescript
// lib/heatmaps/hotjar.ts
export class HotjarTracker {
  private siteId: number;
  private enabled: boolean = false;

  constructor(siteId: number) {
    this.siteId = siteId;
  }

  public initialize() {
    if (typeof window === 'undefined' || this.enabled) return;

    (function(h: any, o: any, t: any, j: any, a?: any, r?: any) {
      h.hj = h.hj || function() { (h.hj.q = h.hj.q || []).push(arguments); };
      h._hjSettings = { hjid: this.siteId, hjsv: 6 };
      a = o.getElementsByTagName('head')[0];
      r = o.createElement('script'); r.async = 1;
      r.src = t + h._hjSettings.hjid + j + h._hjSettings.hjsv;
      a.appendChild(r);
    })(window, document, 'https://static.hotjar.com/c/hotjar-', '.js?sv=');

    this.enabled = true;
  }

  public identify(userId: string, properties?: Record<string, any>) {
    if (!this.enabled || !window.hj) return;
    
    window.hj('identify', userId, properties);
  }

  public tagRecording(tag: string) {
    if (!this.enabled || !window.hj) return;
    
    window.hj('tagRecording', [tag]);
  }

  public virtualPageView(virtualPath: string) {
    if (!this.enabled || !window.hj) return;
    
    window.hj('vpv', virtualPath);
  }

  public stateChange(state: string) {
    if (!this.enabled || !window.hj) return;
    
    window.hj('stateChange', state);
  }

  public setConsent(granted: boolean) {
    if (!this.enabled || !window.hj) return;
    
    window.hj('consent', granted ? 'granted' : 'denied');
  }
}
Microsoft Clarity Integration
typescript
// lib/heatmaps/clarity.ts
export class ClarityTracker {
  private projectId: string;
  private enabled: boolean = false;

  constructor(projectId: string) {
    this.projectId = projectId;
  }

  public initialize() {
    if (typeof window === 'undefined' || this.enabled) return;

    (function(c: any, l: any, a: any, r: any, i: any, t: any, y: any) {
      c[a] = c[a] || function() { (c[a].q = c[a].q || []).push(arguments); };
      t = l.createElement(r); t.async = 1;
      t.src = "https://www.clarity.ms/tag/" + i;
      y = l.getElementsByTagName(r)[0]; y.parentNode.insertBefore(t, y);
    })(window, document, "clarity", "script", this.projectId);

    this.enabled = true;
  }

  public identify(userId: string, sessionId?: string) {
    if (!this.enabled || !window.clarity) return;
    
    window.clarity("identify", userId, sessionId);
  }

  public setTag(key: string, value: string) {
    if (!this.enabled || !window.clarity) return;
    
    window.clarity("set", key, value);
  }

  public upgrade(key: string) {
    if (!this.enabled || !window.clarity) return;
    
    window.clarity("upgrade", key);
  }

  public consent(granted: boolean) {
    if (!this.enabled || !window.clarity) return;
    
    window.clarity("consent", granted);
  }

  public event(name: string, data?: Record<string, any>) {
    if (!this.enabled || !window.clarity) return;
    
    window.clarity("event", name, data);
  }
}
Event Tagging for Heatmaps
typescript
// lib/heatmaps/events.ts
export interface HeatmapEvent {
  selector: string;
  eventType: 'click' | 'scroll' | 'hover' | 'input' | 'form_submit';
  metadata?: Record<string, any>;
  timestamp: Date;
}

export class HeatmapEventTracker {
  private events: HeatmapEvent[] = [];
  private maxEvents = 1000;
  private isRecording = false;

  constructor() {
    this.setupEventListeners();
  }

  private setupEventListeners() {
    if (typeof window === 'undefined') return;

    // Click events
    document.addEventListener('click', (e) => {
      if (!this.isRecording) return;

      const target = e.target as HTMLElement;
      const selector = this.generateSelector(target);

      this.trackEvent({
        selector,
        eventType: 'click',
        metadata: {
          text: target.textContent?.slice(0, 100),
          tagName: target.tagName,
          className: target.className,
          href: target.getAttribute?.('href'),
          position: {
            x: e.clientX,
            y: e.clientY
          }
        },
        timestamp: new Date()
      });
    });


## § ADVANCED PATTERNS: MARKETING AUTOMATION

### Server Actions con Validazione

```typescript
// app/actions/marketing-automation.ts
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


### MARKETING AUTOMATION - Utility Helper #938

```typescript
// lib/utils/marketing-automation-helper-938.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #660

```typescript
// lib/utils/marketing-automation-helper-660.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #904

```typescript
// lib/utils/marketing-automation-helper-904.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #132

```typescript
// lib/utils/marketing-automation-helper-132.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #185

```typescript
// lib/utils/marketing-automation-helper-185.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #30

```typescript
// lib/utils/marketing-automation-helper-30.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #880

```typescript
// lib/utils/marketing-automation-helper-880.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #596

```typescript
// lib/utils/marketing-automation-helper-596.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #849

```typescript
// lib/utils/marketing-automation-helper-849.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #838

```typescript
// lib/utils/marketing-automation-helper-838.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #970

```typescript
// lib/utils/marketing-automation-helper-970.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #438

```typescript
// lib/utils/marketing-automation-helper-438.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #969

```typescript
// lib/utils/marketing-automation-helper-969.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #677

```typescript
// lib/utils/marketing-automation-helper-677.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #5

```typescript
// lib/utils/marketing-automation-helper-5.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #489

```typescript
// lib/utils/marketing-automation-helper-489.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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

  getConfig(): Readonly<MARKETINGAUTOMATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETING AUTOMATION - Utility Helper #298

```typescript
// lib/utils/marketing-automation-helper-298.ts
import { z } from "zod";

interface MARKETINGAUTOMATIONConfig {
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

export class MARKETINGAUTOMATIONProcessor<TInput, TOutput> {
  private config: MARKETINGAUTOMATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETINGAUTOMATIONConfig> = {}) {
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
          result = await p