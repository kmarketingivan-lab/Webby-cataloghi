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
