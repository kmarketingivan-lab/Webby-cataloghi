# 📊 CATALOGO ANALYTICS & MONITORING v1.0

## Catalogo Completo per Applicazioni Next.js Enterprise

**Versione:** 1.0.0  
**Data:** Gennaio 2026  
**Applicabilità:** Next.js 14+, React 18+, Node.js 20+  
**Focus:** Analytics, Monitoring, Observability, Logging

---

## 📑 Indice

1. [Introduzione all'Analytics & Monitoring](#1-introduzione-allanalytics--monitoring)
2. [Web Analytics - Google Analytics 4](#2-web-analytics---google-analytics-4)
3. [Product Analytics - PostHog](#3-product-analytics---posthog)
4. [Core Web Vitals & Performance Monitoring](#4-core-web-vitals--performance-monitoring)
5. [Error Tracking - Sentry](#5-error-tracking---sentry)
6. [Application Performance Monitoring (APM)](#6-application-performance-monitoring-apm)
7. [Logging & Structured Logs](#7-logging--structured-logs)
8. [Real User Monitoring (RUM)](#8-real-user-monitoring-rum)
9. [Observability - OpenTelemetry](#9-observability---opentelemetry)
10. [Metrics & Dashboards - Prometheus & Grafana](#10-metrics--dashboards---prometheus--grafana)
11. [Alerting & Incident Management](#11-alerting--incident-management)
12. [Session Replay & Heatmaps](#12-session-replay--heatmaps)
13. [A/B Testing & Feature Flags](#13-ab-testing--feature-flags)
14. [Privacy & GDPR Compliance](#14-privacy--gdpr-compliance)
15. [Costi e Confronto Strumenti](#15-costi-e-confronto-strumenti)
16. [Appendice: Quick Reference](#appendice-quick-reference)

---

## 1. Introduzione all'Analytics & Monitoring

### 1.1 Perché Analytics & Monitoring

L'analytics e il monitoring sono fondamentali per comprendere il comportamento degli utenti, identificare problemi tecnici e prendere decisioni basate sui dati. In un'applicazione Next.js moderna, una strategia completa di observability include diversi livelli.

```
┌─────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY STACK                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   METRICS   │  │    LOGS     │  │   TRACES    │            │
│  │             │  │             │  │             │            │
│  │ • Counters  │  │ • Events    │  │ • Requests  │            │
│  │ • Gauges    │  │ • Errors    │  │ • Spans     │            │
│  │ • Histograms│  │ • Context   │  │ • Context   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│         │                │                │                    │
│         └────────────────┼────────────────┘                    │
│                          │                                     │
│                          ▼                                     │
│  ┌─────────────────────────────────────────────────────────┐  │
│  │              UNIFIED OBSERVABILITY PLATFORM              │  │
│  │   (Grafana, Datadog, New Relic, OpenTelemetry)          │  │
│  └─────────────────────────────────────────────────────────┘  │
│                          │                                     │
│                          ▼                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │  Dashboards  │  │   Alerts     │  │   Reports    │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 I Tre Pilastri dell'Observability

| Pilastro | Descrizione | Strumenti Tipici |
|----------|-------------|------------------|
| **Metrics** | Dati numerici aggregati nel tempo | Prometheus, StatsD, CloudWatch |
| **Logs** | Eventi discreti con contesto | Pino, Winston, ELK Stack |
| **Traces** | Percorso delle richieste attraverso i servizi | OpenTelemetry, Jaeger, Zipkin |

### 1.3 Tipi di Analytics

```
┌─────────────────────────────────────────────────────────────────┐
│                      ANALYTICS ECOSYSTEM                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────┐    ┌───────────────────┐                │
│  │   WEB ANALYTICS   │    │ PRODUCT ANALYTICS │                │
│  ├───────────────────┤    ├───────────────────┤                │
│  │ • Page views      │    │ • User behavior   │                │
│  │ • Traffic sources │    │ • Feature usage   │                │
│  │ • Bounce rate     │    │ • Funnels         │                │
│  │ • Session duration│    │ • Retention       │                │
│  └───────────────────┘    └───────────────────┘                │
│           │                        │                            │
│           └────────────┬───────────┘                            │
│                        │                                        │
│  ┌───────────────────┐ │ ┌───────────────────┐                 │
│  │ ERROR TRACKING    │ │ │ PERFORMANCE       │                 │
│  ├───────────────────┤ │ ├───────────────────┤                 │
│  │ • Exceptions      │ │ │ • Core Web Vitals │                 │
│  │ • Stack traces    │ │ │ • Response times  │                 │
│  │ • User impact     │ │ │ • Resource usage  │                 │
│  │ • Alerts          │ │ │ • Bottlenecks     │                 │
│  └───────────────────┘ │ └───────────────────┘                 │
│                        │                                        │
│                        ▼                                        │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              BUSINESS INTELLIGENCE                       │   │
│  │   Conversion rates, Revenue, Customer LTV, Churn        │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 1.4 Analytics Stack Raccomandato per Next.js

```typescript
// Recommended Analytics Stack for Next.js 2025
const analyticsStack = {
  // Web Analytics - Traffic & Engagement
  webAnalytics: {
    primary: 'Google Analytics 4',
    alternative: 'Plausible',  // Privacy-focused
    implementation: '@next/third-parties'
  },
  
  // Product Analytics - User Behavior
  productAnalytics: {
    primary: 'PostHog',
    alternative: 'Mixpanel',
    features: ['funnels', 'cohorts', 'feature-flags']
  },
  
  // Error Tracking
  errorTracking: {
    primary: 'Sentry',
    features: ['session-replay', 'performance', 'profiling']
  },
  
  // Performance Monitoring
  performance: {
    rum: 'Web Vitals Library',
    apm: 'OpenTelemetry',
    dashboards: 'Grafana'
  },
  
  // Logging
  logging: {
    library: 'Pino',
    aggregation: 'Loki',
    visualization: 'Grafana'
  },
  
  // Observability Platform
  observability: {
    tracing: 'OpenTelemetry',
    metrics: 'Prometheus',
    visualization: 'Grafana Cloud'
  }
};
```

---

## 2. Web Analytics - Google Analytics 4

### 2.1 Introduzione a GA4

Google Analytics 4 (GA4) è la piattaforma standard per web analytics dal 2023. Utilizza un modello basato su eventi invece di sessioni, offrendo maggiore flessibilità nel tracciamento.

**Caratteristiche principali:**
- **Event-based model**: Tutto è un evento, incluse le pageview
- **Key Events** (ex Conversions): Azioni importanti da tracciare
- **Enhanced Measurement**: Tracciamento automatico di scroll, click, video
- **Cross-platform**: Web e App in un unico property

### 2.2 Setup GA4 in Next.js con @next/third-parties

```bash
# Installazione
pnpm add @next/third-parties
```

```typescript
// app/layout.tsx
import { GoogleAnalytics } from '@next/third-parties/google';

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="it">
      <body>{children}</body>
      {/* GA4 dopo il body per non bloccare il rendering */}
      <GoogleAnalytics gaId={process.env.NEXT_PUBLIC_GA_ID!} />
    </html>
  );
}
```

### 2.3 Configurazione Completa con Consent Management

```typescript
// lib/analytics/google-analytics.ts
'use client';

import { sendGAEvent } from '@next/third-parties/google';

// Configuration
export const GA_MEASUREMENT_ID = process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID;

// Check if analytics should be enabled
export const isAnalyticsEnabled = (): boolean => {
  return (
    typeof window !== 'undefined' &&
    process.env.NODE_ENV === 'production' &&
    Boolean(GA_MEASUREMENT_ID) &&
    GA_MEASUREMENT_ID !== 'G-XXXXXXXXXX' &&
    hasUserConsent()
  );
};

// Check user consent
export const hasUserConsent = (): boolean => {
  if (typeof window === 'undefined') return false;
  const consent = localStorage.getItem('analytics_consent');
  return consent === 'granted';
};

// Set user consent
export const setAnalyticsConsent = (granted: boolean) => {
  localStorage.setItem('analytics_consent', granted ? 'granted' : 'denied');
  
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('consent', 'update', {
      analytics_storage: granted ? 'granted' : 'denied',
    });
  }
};

// Custom event types
export interface GAEventParams {
  action: string;
  category?: string;
  label?: string;
  value?: number;
  [key: string]: unknown;
}

// Send custom event
export const trackEvent = (params: GAEventParams) => {
  if (!isAnalyticsEnabled()) return;
  
  sendGAEvent('event', params.action, {
    event_category: params.category,
    event_label: params.label,
    value: params.value,
    ...params,
  });
};

// Track page view (for SPA navigation)
export const trackPageView = (url: string, title?: string) => {
  if (!isAnalyticsEnabled()) return;
  
  sendGAEvent('event', 'page_view', {
    page_location: url,
    page_title: title || document.title,
  });
};

// E-commerce events
export const trackPurchase = (transaction: {
  transaction_id: string;
  value: number;
  currency: string;
  items: Array<{
    item_id: string;
    item_name: string;
    price: number;
    quantity: number;
  }>;
}) => {
  if (!isAnalyticsEnabled()) return;
  
  sendGAEvent('event', 'purchase', transaction);
};

// User identification
export const setUserId = (userId: string) => {
  if (!isAnalyticsEnabled()) return;
  
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('config', GA_MEASUREMENT_ID!, {
      user_id: userId,
    });
  }
};

// User properties
export const setUserProperties = (properties: Record<string, unknown>) => {
  if (!isAnalyticsEnabled()) return;
  
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('set', 'user_properties', properties);
  }
};
```

### 2.4 Components per Analytics

```typescript
// components/analytics/GoogleAnalyticsProvider.tsx
'use client';

import { usePathname, useSearchParams } from 'next/navigation';
import { useEffect, Suspense } from 'react';
import { trackPageView, isAnalyticsEnabled } from '@/lib/analytics/google-analytics';

function PageViewTracker() {
  const pathname = usePathname();
  const searchParams = useSearchParams();

  useEffect(() => {
    if (!isAnalyticsEnabled()) return;
    
    const url = pathname + (searchParams?.toString() ? `?${searchParams}` : '');
    trackPageView(window.location.origin + url);
  }, [pathname, searchParams]);

  return null;
}

export function GoogleAnalyticsProvider({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <>
      <Suspense fallback={null}>
        <PageViewTracker />
      </Suspense>
      {children}
    </>
  );
}
```

```typescript
// components/analytics/ConsentBanner.tsx
'use client';

import { useState, useEffect } from 'react';
import { setAnalyticsConsent, hasUserConsent } from '@/lib/analytics/google-analytics';

export function ConsentBanner() {
  const [showBanner, setShowBanner] = useState(false);

  useEffect(() => {
    // Check if consent was already given
    const consent = localStorage.getItem('analytics_consent');
    if (consent === null) {
      setShowBanner(true);
    }
  }, []);

  const handleAccept = () => {
    setAnalyticsConsent(true);
    setShowBanner(false);
  };

  const handleDecline = () => {
    setAnalyticsConsent(false);
    setShowBanner(false);
  };

  if (!showBanner) return null;

  return (
    <div className="fixed bottom-0 left-0 right-0 bg-gray-900 text-white p-4 z-50">
      <div className="max-w-7xl mx-auto flex flex-col sm:flex-row items-center justify-between gap-4">
        <p className="text-sm">
          Utilizziamo cookie analitici per migliorare la tua esperienza.
          Leggi la nostra{' '}
          <a href="/privacy" className="underline">
            Privacy Policy
          </a>
        </p>
        <div className="flex gap-2">
          <button
            onClick={handleDecline}
            className="px-4 py-2 text-sm bg-gray-700 rounded hover:bg-gray-600"
          >
            Rifiuta
          </button>
          <button
            onClick={handleAccept}
            className="px-4 py-2 text-sm bg-blue-600 rounded hover:bg-blue-500"
          >
            Accetta
          </button>
        </div>
      </div>
    </div>
  );
}
```

### 2.5 Eventi GA4 Raccomandati

```typescript
// lib/analytics/events.ts

// Recommended events per categoria
export const GA4Events = {
  // E-commerce
  ecommerce: {
    viewItem: (item: { item_id: string; item_name: string; price: number }) => ({
      action: 'view_item',
      currency: 'EUR',
      value: item.price,
      items: [item],
    }),
    addToCart: (item: { item_id: string; item_name: string; price: number; quantity: number }) => ({
      action: 'add_to_cart',
      currency: 'EUR',
      value: item.price * item.quantity,
      items: [item],
    }),
    beginCheckout: (items: any[], value: number) => ({
      action: 'begin_checkout',
      currency: 'EUR',
      value,
      items,
    }),
    purchase: (transaction: {
      transaction_id: string;
      value: number;
      items: any[];
    }) => ({
      action: 'purchase',
      ...transaction,
      currency: 'EUR',
    }),
  },
  
  // Lead Generation
  leadGen: {
    generateLead: (form: string, value?: number) => ({
      action: 'generate_lead',
      event_category: 'engagement',
      event_label: form,
      value,
    }),
    formSubmit: (formName: string) => ({
      action: 'form_submit',
      event_category: 'forms',
      event_label: formName,
    }),
    signup: (method: string) => ({
      action: 'sign_up',
      method,
    }),
  },
  
  // Engagement
  engagement: {
    search: (searchTerm: string) => ({
      action: 'search',
      search_term: searchTerm,
    }),
    share: (method: string, contentType: string, itemId: string) => ({
      action: 'share',
      method,
      content_type: contentType,
      item_id: itemId,
    }),
    fileDownload: (fileName: string, fileType: string) => ({
      action: 'file_download',
      event_category: 'engagement',
      event_label: fileName,
      file_extension: fileType,
    }),
  },
  
  // User Actions
  userActions: {
    login: (method: string) => ({
      action: 'login',
      method,
    }),
    selectContent: (contentType: string, contentId: string) => ({
      action: 'select_content',
      content_type: contentType,
      content_id: contentId,
    }),
  },
};

// Usage example
// trackEvent(GA4Events.ecommerce.addToCart({
//   item_id: 'SKU_123',
//   item_name: 'Product Name',
//   price: 29.99,
//   quantity: 1
// }));
```

### 2.6 Web Vitals to GA4

```typescript
// lib/analytics/web-vitals.ts
'use client';

import { useReportWebVitals } from 'next/web-vitals';
import { sendGAEvent } from '@next/third-parties/google';

export function WebVitalsReporter() {
  useReportWebVitals((metric) => {
    const { name, value, id, rating } = metric;
    
    // Send to GA4
    sendGAEvent('event', name, {
      // Google Analytics expects integers
      value: Math.round(name === 'CLS' ? value * 1000 : value),
      event_category: 'Web Vitals',
      event_label: id,
      // Non-interaction to avoid affecting bounce rate
      non_interaction: true,
      // Custom dimensions
      metric_rating: rating,
      metric_value: value,
    });
    
    // Also log in development
    if (process.env.NODE_ENV === 'development') {
      console.log(`[Web Vitals] ${name}:`, {
        value: value.toFixed(2),
        rating,
        id,
      });
    }
  });
  
  return null;
}

// In layout.tsx:
// <WebVitalsReporter />
```

### 2.7 Debug View e Testing

```typescript
// lib/analytics/debug.ts

// Enable GA4 Debug Mode
export const enableDebugMode = () => {
  if (typeof window !== 'undefined' && window.gtag) {
    // Set debug mode
    window.gtag('config', process.env.NEXT_PUBLIC_GA_MEASUREMENT_ID!, {
      debug_mode: true,
    });
  }
};

// Debug logger for development
export const logEvent = (eventName: string, params: Record<string, unknown>) => {
  if (process.env.NODE_ENV === 'development') {
    console.group(`📊 GA4 Event: ${eventName}`);
    console.log('Parameters:', params);
    console.log('Timestamp:', new Date().toISOString());
    console.groupEnd();
  }
};

// Validate event parameters
export const validateEventParams = (params: Record<string, unknown>): boolean => {
  const errors: string[] = [];
  
  // Check reserved parameter names
  const reservedParams = ['firebase_', 'google_', 'ga_'];
  Object.keys(params).forEach(key => {
    if (reservedParams.some(prefix => key.startsWith(prefix))) {
      errors.push(`Parameter "${key}" uses a reserved prefix`);
    }
  });
  
  // Check parameter value length
  Object.entries(params).forEach(([key, value]) => {
    if (typeof value === 'string' && value.length > 100) {
      errors.push(`Parameter "${key}" exceeds 100 characters`);
    }
  });
  
  if (errors.length > 0) {
    console.warn('GA4 Event Validation Errors:', errors);
    return false;
  }
  
  return true;
};
```

---

## 3. Product Analytics - PostHog

### 3.1 Introduzione a PostHog

PostHog è una piattaforma open-source all-in-one per product analytics che include: analytics, session recording, feature flags, A/B testing, surveys e altro.

**Vantaggi:**
- Open-source e self-hostable
- Generosa free tier (1M eventi/mese)
- Feature flags integrati
- Session replay incluso
- Nessun vendor lock-in

### 3.2 Setup PostHog in Next.js

```bash
# Installazione
pnpm add posthog-js @posthog/react
```

```typescript
// lib/analytics/posthog.ts
'use client';

import posthog from 'posthog-js';
import { PostHogProvider as PHProvider } from 'posthog-js/react';
import { usePathname, useSearchParams } from 'next/navigation';
import { useEffect, Suspense } from 'react';

// Initialize PostHog
if (typeof window !== 'undefined') {
  posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY!, {
    api_host: process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://app.posthog.com',
    // Recommended defaults for 2025
    defaults: '2025-11-30',
    // Or manually configure:
    capture_pageview: false, // We'll capture manually for SPA
    capture_pageleave: true,
    person_profiles: 'identified_only',
    // Privacy settings
    persistence: 'localStorage',
    // Performance
    autocapture: true,
    // Session recording
    session_recording: {
      maskAllInputs: true,
      maskTextSelector: '[data-ph-mask]',
    },
    // Feature flags
    bootstrap: {
      featureFlags: {},
    },
  });
}

export { posthog };
```

```typescript
// components/providers/PostHogProvider.tsx
'use client';

import { posthog } from '@/lib/analytics/posthog';
import { PostHogProvider as PHProvider, usePostHog } from 'posthog-js/react';
import { usePathname, useSearchParams } from 'next/navigation';
import { useEffect, Suspense } from 'react';

function PostHogPageView() {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const posthog = usePostHog();

  useEffect(() => {
    if (pathname && posthog) {
      let url = window.origin + pathname;
      if (searchParams?.toString()) {
        url += '?' + searchParams.toString();
      }
      
      posthog.capture('$pageview', {
        $current_url: url,
      });
    }
  }, [pathname, searchParams, posthog]);

  return null;
}

export function PostHogProvider({ children }: { children: React.ReactNode }) {
  return (
    <PHProvider client={posthog}>
      <Suspense fallback={null}>
        <PostHogPageView />
      </Suspense>
      {children}
    </PHProvider>
  );
}
```

### 3.3 User Identification

```typescript
// lib/analytics/posthog-identify.ts
'use client';

import { posthog } from './posthog';

interface UserProperties {
  email?: string;
  name?: string;
  plan?: 'free' | 'pro' | 'enterprise';
  company?: string;
  createdAt?: string;
  [key: string]: unknown;
}

// Identify user after login
export const identifyUser = (userId: string, properties?: UserProperties) => {
  posthog.identify(userId, properties);
};

// Update user properties
export const setUserProperties = (properties: UserProperties) => {
  posthog.people.set(properties);
};

// Set properties once (won't overwrite)
export const setUserPropertiesOnce = (properties: UserProperties) => {
  posthog.people.set_once(properties);
};

// Increment numeric properties
export const incrementUserProperty = (property: string, value: number = 1) => {
  posthog.people.increment(property, value);
};

// Reset (on logout)
export const resetUser = () => {
  posthog.reset();
};

// Group identification (for B2B)
export const setGroup = (groupType: string, groupKey: string, properties?: Record<string, unknown>) => {
  posthog.group(groupType, groupKey, properties);
};

// Example usage in auth callback
// identifyUser(user.id, {
//   email: user.email,
//   name: user.name,
//   plan: user.subscription?.plan,
//   company: user.organization?.name,
//   createdAt: user.createdAt,
// });
```

### 3.4 Event Tracking

```typescript
// lib/analytics/posthog-events.ts
'use client';

import { posthog } from './posthog';

// Generic event capture
export const capture = (
  eventName: string,
  properties?: Record<string, unknown>
) => {
  posthog.capture(eventName, properties);
};

// Structured events per feature
export const ProductEvents = {
  // Onboarding
  onboarding: {
    started: () => capture('onboarding_started'),
    stepCompleted: (step: number, stepName: string) => 
      capture('onboarding_step_completed', { step, step_name: stepName }),
    completed: (duration: number) => 
      capture('onboarding_completed', { duration_seconds: duration }),
    skipped: (atStep: number) => 
      capture('onboarding_skipped', { skipped_at_step: atStep }),
  },
  
  // Feature usage
  feature: {
    used: (featureName: string, metadata?: Record<string, unknown>) =>
      capture('feature_used', { feature: featureName, ...metadata }),
    enabled: (featureName: string) =>
      capture('feature_enabled', { feature: featureName }),
    disabled: (featureName: string) =>
      capture('feature_disabled', { feature: featureName }),
  },
  
  // Subscription
  subscription: {
    viewed: (plan: string) => 
      capture('subscription_viewed', { plan }),
    started: (plan: string, price: number) =>
      capture('subscription_started', { plan, price }),
    upgraded: (fromPlan: string, toPlan: string) =>
      capture('subscription_upgraded', { from_plan: fromPlan, to_plan: toPlan }),
    downgraded: (fromPlan: string, toPlan: string) =>
      capture('subscription_downgraded', { from_plan: fromPlan, to_plan: toPlan }),
    cancelled: (reason?: string) =>
      capture('subscription_cancelled', { reason }),
  },
  
  // Content interaction
  content: {
    viewed: (contentType: string, contentId: string, title?: string) =>
      capture('content_viewed', { content_type: contentType, content_id: contentId, title }),
    shared: (contentType: string, contentId: string, method: string) =>
      capture('content_shared', { content_type: contentType, content_id: contentId, share_method: method }),
    downloaded: (contentType: string, contentId: string) =>
      capture('content_downloaded', { content_type: contentType, content_id: contentId }),
  },
  
  // Errors (complement to Sentry)
  error: {
    encountered: (errorType: string, errorMessage: string, context?: Record<string, unknown>) =>
      capture('error_encountered', { error_type: errorType, error_message: errorMessage, ...context }),
    reported: (errorId: string) =>
      capture('error_reported', { error_id: errorId }),
  },
};

// Time tracking for events
export class EventTimer {
  private startTime: number;
  private eventName: string;
  
  constructor(eventName: string) {
    this.eventName = eventName;
    this.startTime = Date.now();
  }
  
  complete(properties?: Record<string, unknown>) {
    const duration = Date.now() - this.startTime;
    capture(this.eventName, {
      ...properties,
      duration_ms: duration,
    });
  }
}

// Usage:
// const timer = new EventTimer('document_edited');
// ... user edits document ...
// timer.complete({ document_id: '123' });
```

### 3.5 Feature Flags

```typescript
// lib/analytics/feature-flags.ts
'use client';

import { posthog } from './posthog';
import { useFeatureFlagEnabled, useFeatureFlagVariantKey } from 'posthog-js/react';

// Hook for boolean flags
export function useFeatureFlag(flagKey: string): boolean {
  return useFeatureFlagEnabled(flagKey) ?? false;
}

// Hook for multivariate flags
export function useFeatureVariant(flagKey: string): string | undefined {
  return useFeatureFlagVariantKey(flagKey) ?? undefined;
}

// Check flag imperatively
export const isFeatureEnabled = (flagKey: string): boolean => {
  return posthog.isFeatureEnabled(flagKey) ?? false;
};

// Get variant value
export const getFeatureVariant = (flagKey: string): string | boolean | undefined => {
  return posthog.getFeatureFlag(flagKey);
};

// Get flag payload (additional data)
export const getFeatureFlagPayload = (flagKey: string): unknown => {
  return posthog.getFeatureFlagPayload(flagKey);
};

// Reload flags (useful after user identification)
export const reloadFeatureFlags = async (): Promise<void> => {
  await posthog.reloadFeatureFlags();
};

// Override flags for testing
export const overrideFeatureFlag = (flagKey: string, value: string | boolean) => {
  posthog.featureFlags.override({ [flagKey]: value });
};

// Clear overrides
export const clearFeatureFlagOverrides = () => {
  posthog.featureFlags.override(false);
};
```

```typescript
// components/FeatureGate.tsx
'use client';

import { useFeatureFlag, useFeatureVariant } from '@/lib/analytics/feature-flags';
import { ReactNode } from 'react';

interface FeatureGateProps {
  flag: string;
  children: ReactNode;
  fallback?: ReactNode;
}

export function FeatureGate({ flag, children, fallback = null }: FeatureGateProps) {
  const isEnabled = useFeatureFlag(flag);
  
  return isEnabled ? <>{children}</> : <>{fallback}</>;
}

// Usage:
// <FeatureGate flag="new-checkout-flow">
//   <NewCheckout />
// </FeatureGate>

interface VariantGateProps {
  flag: string;
  variants: Record<string, ReactNode>;
  fallback?: ReactNode;
}

export function VariantGate({ flag, variants, fallback = null }: VariantGateProps) {
  const variant = useFeatureVariant(flag);
  
  if (!variant || !variants[variant]) {
    return <>{fallback}</>;
  }
  
  return <>{variants[variant]}</>;
}

// Usage:
// <VariantGate
//   flag="pricing-experiment"
//   variants={{
//     control: <OriginalPricing />,
//     variant_a: <NewPricingA />,
//     variant_b: <NewPricingB />,
//   }}
// />
```

### 3.6 Server-Side PostHog

```typescript
// lib/analytics/posthog-server.ts
import { PostHog } from 'posthog-node';

// Server-side PostHog client
const posthogClient = new PostHog(process.env.POSTHOG_API_KEY!, {
  host: process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://app.posthog.com',
  flushAt: 1, // Flush immediately in serverless
  flushInterval: 0,
});

// Capture server-side event
export const captureServerEvent = async (
  distinctId: string,
  event: string,
  properties?: Record<string, unknown>
) => {
  posthogClient.capture({
    distinctId,
    event,
    properties,
  });
  
  // Flush in serverless environment
  await posthogClient.flush();
};

// Get feature flag server-side
export const getServerFeatureFlag = async (
  distinctId: string,
  flagKey: string,
  options?: {
    groups?: Record<string, string>;
    personProperties?: Record<string, unknown>;
  }
): Promise<boolean | string | undefined> => {
  return await posthogClient.getFeatureFlag(flagKey, distinctId, options);
};

// Get all feature flags for user
export const getAllServerFeatureFlags = async (
  distinctId: string,
  options?: {
    groups?: Record<string, string>;
    personProperties?: Record<string, unknown>;
  }
) => {
  return await posthogClient.getAllFlags(distinctId, options);
};

// Shutdown (for graceful shutdown)
export const shutdownPostHog = async () => {
  await posthogClient.shutdown();
};
```

```typescript
// app/api/track/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { captureServerEvent } from '@/lib/analytics/posthog-server';

export async function POST(request: NextRequest) {
  try {
    const { userId, event, properties } = await request.json();
    
    await captureServerEvent(userId, event, {
      ...properties,
      $ip: request.headers.get('x-forwarded-for') || request.ip,
    });
    
    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Error tracking event:', error);
    return NextResponse.json(
      { error: 'Failed to track event' },
      { status: 500 }
    );
  }
}
```



---

## 4. Core Web Vitals & Performance Monitoring

### 4.1 Introduzione ai Core Web Vitals

I Core Web Vitals sono metriche ufficiali di Google che misurano l'esperienza utente reale. Dal 2024, le tre metriche principali sono:

```
┌─────────────────────────────────────────────────────────────────┐
│                    CORE WEB VITALS 2025                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                    LCP (Loading)                         │   │
│  │              Largest Contentful Paint                    │   │
│  │                                                          │   │
│  │    Good          Needs Improvement      Poor             │   │
│  │  ≤ 2.5s              2.5s - 4s         > 4s             │   │
│  │   🟢                    🟡              🔴              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   INP (Interactivity)                    │   │
│  │             Interaction to Next Paint                    │   │
│  │         (Replaced FID in March 2024)                     │   │
│  │                                                          │   │
│  │    Good          Needs Improvement      Poor             │   │
│  │  ≤ 200ms            200ms - 500ms      > 500ms          │   │
│  │   🟢                    🟡              🔴              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                   CLS (Visual Stability)                 │   │
│  │             Cumulative Layout Shift                      │   │
│  │                                                          │   │
│  │    Good          Needs Improvement      Poor             │   │
│  │  ≤ 0.1               0.1 - 0.25        > 0.25           │   │
│  │   🟢                    🟡              🔴              │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│  📊 Il 75° percentile delle visite deve raggiungere "Good"     │
│     per essere considerato "Passing" da Google                 │
└─────────────────────────────────────────────────────────────────┘
```

### 4.2 Implementazione Web Vitals in Next.js

```bash
# Installazione web-vitals library
pnpm add web-vitals
```

```typescript
// lib/performance/web-vitals.ts
'use client';

import { onCLS, onINP, onLCP, onFCP, onTTFB, Metric } from 'web-vitals';

export type WebVitalsMetric = {
  name: 'CLS' | 'INP' | 'LCP' | 'FCP' | 'TTFB';
  value: number;
  rating: 'good' | 'needs-improvement' | 'poor';
  id: string;
  delta: number;
  navigationType: string;
  attribution?: unknown;
};

// Thresholds per metric
const thresholds = {
  LCP: { good: 2500, poor: 4000 },
  INP: { good: 200, poor: 500 },
  CLS: { good: 0.1, poor: 0.25 },
  FCP: { good: 1800, poor: 3000 },
  TTFB: { good: 800, poor: 1800 },
};

// Rating calculator
const getRating = (name: string, value: number): 'good' | 'needs-improvement' | 'poor' => {
  const threshold = thresholds[name as keyof typeof thresholds];
  if (!threshold) return 'good';
  
  if (value <= threshold.good) return 'good';
  if (value <= threshold.poor) return 'needs-improvement';
  return 'poor';
};

// Report handler type
type ReportHandler = (metric: WebVitalsMetric) => void;

// Initialize web vitals reporting
export const initWebVitals = (onReport: ReportHandler) => {
  const handleMetric = (metric: Metric) => {
    onReport({
      name: metric.name as WebVitalsMetric['name'],
      value: metric.value,
      rating: metric.rating,
      id: metric.id,
      delta: metric.delta,
      navigationType: metric.navigationType || 'navigate',
      attribution: (metric as any).attribution,
    });
  };

  onCLS(handleMetric);
  onINP(handleMetric);
  onLCP(handleMetric);
  onFCP(handleMetric);
  onTTFB(handleMetric);
};

// Send metrics to analytics endpoint
export const reportWebVitals = async (metric: WebVitalsMetric) => {
  // Send to custom endpoint
  if (process.env.NEXT_PUBLIC_VITALS_ENDPOINT) {
    try {
      await fetch(process.env.NEXT_PUBLIC_VITALS_ENDPOINT, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...metric,
          timestamp: Date.now(),
          url: window.location.href,
          userAgent: navigator.userAgent,
          connection: (navigator as any).connection?.effectiveType,
        }),
        keepalive: true, // Ensure request completes even if page unloads
      });
    } catch (error) {
      console.error('Failed to report web vitals:', error);
    }
  }
  
  // Log in development
  if (process.env.NODE_ENV === 'development') {
    const emoji = metric.rating === 'good' ? '🟢' : metric.rating === 'needs-improvement' ? '🟡' : '🔴';
    console.log(`${emoji} ${metric.name}:`, metric.value.toFixed(2), `(${metric.rating})`);
  }
};
```

### 4.3 Web Vitals Component

```typescript
// components/performance/WebVitalsReporter.tsx
'use client';

import { useEffect } from 'react';
import { initWebVitals, reportWebVitals, WebVitalsMetric } from '@/lib/performance/web-vitals';
import { trackEvent } from '@/lib/analytics/google-analytics';
import { capture } from '@/lib/analytics/posthog-events';

interface WebVitalsReporterProps {
  // Optional: send to custom destinations
  destinations?: {
    googleAnalytics?: boolean;
    posthog?: boolean;
    custom?: (metric: WebVitalsMetric) => void;
  };
}

export function WebVitalsReporter({ 
  destinations = { googleAnalytics: true, posthog: true } 
}: WebVitalsReporterProps) {
  useEffect(() => {
    initWebVitals((metric) => {
      // Send to our endpoint
      reportWebVitals(metric);
      
      // Send to Google Analytics
      if (destinations.googleAnalytics) {
        trackEvent({
          action: metric.name,
          category: 'Web Vitals',
          label: metric.id,
          value: Math.round(metric.name === 'CLS' ? metric.value * 1000 : metric.value),
          metric_rating: metric.rating,
        });
      }
      
      // Send to PostHog
      if (destinations.posthog) {
        capture('web_vital_measured', {
          metric_name: metric.name,
          metric_value: metric.value,
          metric_rating: metric.rating,
          metric_id: metric.id,
        });
      }
      
      // Custom handler
      if (destinations.custom) {
        destinations.custom(metric);
      }
    });
  }, [destinations]);
  
  return null;
}
```

### 4.4 API Endpoint per Web Vitals

```typescript
// app/api/vitals/route.ts
import { NextRequest, NextResponse } from 'next/server';

interface VitalsPayload {
  name: string;
  value: number;
  rating: string;
  id: string;
  delta: number;
  navigationType: string;
  timestamp: number;
  url: string;
  userAgent: string;
  connection?: string;
}

// Store metrics (in production, use a time-series database)
const metricsBuffer: VitalsPayload[] = [];
const BUFFER_SIZE = 100;

export async function POST(request: NextRequest) {
  try {
    const metric = await request.json() as VitalsPayload;
    
    // Add to buffer
    metricsBuffer.push(metric);
    
    // Flush when buffer is full
    if (metricsBuffer.length >= BUFFER_SIZE) {
      await flushMetrics();
    }
    
    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Error processing vitals:', error);
    return NextResponse.json(
      { error: 'Failed to process vitals' },
      { status: 500 }
    );
  }
}

async function flushMetrics() {
  const metrics = metricsBuffer.splice(0, metricsBuffer.length);
  
  // In production, send to your observability platform
  // Example: Send to Prometheus Pushgateway, Grafana Cloud, etc.
  console.log(`Flushing ${metrics.length} web vitals metrics`);
  
  // Aggregate metrics
  const aggregated = aggregateMetrics(metrics);
  console.log('Aggregated metrics:', aggregated);
}

function aggregateMetrics(metrics: VitalsPayload[]) {
  const byName: Record<string, number[]> = {};
  
  metrics.forEach(m => {
    if (!byName[m.name]) byName[m.name] = [];
    byName[m.name].push(m.value);
  });
  
  return Object.entries(byName).map(([name, values]) => ({
    name,
    count: values.length,
    p50: percentile(values, 50),
    p75: percentile(values, 75),
    p90: percentile(values, 90),
    p99: percentile(values, 99),
  }));
}

function percentile(values: number[], p: number): number {
  const sorted = values.slice().sort((a, b) => a - b);
  const index = Math.ceil((p / 100) * sorted.length) - 1;
  return sorted[Math.max(0, index)];
}
```

### 4.5 Performance Budget

```typescript
// lib/performance/budget.ts

export interface PerformanceBudget {
  metrics: {
    LCP: number;
    INP: number;
    CLS: number;
    FCP: number;
    TTFB: number;
  };
  resources: {
    totalSize: number; // KB
    jsSize: number;    // KB
    cssSize: number;   // KB
    imageSize: number; // KB
    fontSize: number;  // KB
  };
  requests: {
    total: number;
    js: number;
    css: number;
    images: number;
  };
}

// Recommended budgets for Next.js app
export const recommendedBudget: PerformanceBudget = {
  metrics: {
    LCP: 2500,    // ms
    INP: 200,     // ms
    CLS: 0.1,     // score
    FCP: 1800,    // ms
    TTFB: 600,    // ms
  },
  resources: {
    totalSize: 500,   // KB
    jsSize: 200,      // KB
    cssSize: 50,      // KB
    imageSize: 200,   // KB
    fontSize: 50,     // KB
  },
  requests: {
    total: 50,
    js: 10,
    css: 3,
    images: 30,
  },
};

// Budget checker
export function checkBudget(
  actual: Partial<PerformanceBudget>,
  budget: PerformanceBudget = recommendedBudget
): {
  passed: boolean;
  violations: string[];
  warnings: string[];
} {
  const violations: string[] = [];
  const warnings: string[] = [];
  
  // Check metrics
  if (actual.metrics) {
    Object.entries(actual.metrics).forEach(([key, value]) => {
      const budgetValue = budget.metrics[key as keyof typeof budget.metrics];
      if (value > budgetValue * 1.5) {
        violations.push(`${key}: ${value} exceeds budget ${budgetValue} by more than 50%`);
      } else if (value > budgetValue) {
        warnings.push(`${key}: ${value} exceeds budget ${budgetValue}`);
      }
    });
  }
  
  // Check resources
  if (actual.resources) {
    Object.entries(actual.resources).forEach(([key, value]) => {
      const budgetValue = budget.resources[key as keyof typeof budget.resources];
      if (value > budgetValue * 1.5) {
        violations.push(`${key}: ${value}KB exceeds budget ${budgetValue}KB by more than 50%`);
      } else if (value > budgetValue) {
        warnings.push(`${key}: ${value}KB exceeds budget ${budgetValue}KB`);
      }
    });
  }
  
  return {
    passed: violations.length === 0,
    violations,
    warnings,
  };
}
```

### 4.6 Performance Monitoring Script (CI/CD)

```typescript
// scripts/check-performance.ts
import { chromium } from 'playwright';
import { checkBudget, recommendedBudget } from '../lib/performance/budget';

async function runPerformanceAudit(url: string) {
  const browser = await chromium.launch();
  const context = await browser.newContext();
  const page = await context.newPage();
  
  // Enable Performance API
  await page.addInitScript(() => {
    window.performanceMetrics = {};
    
    new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'largest-contentful-paint') {
          (window as any).performanceMetrics.LCP = entry.startTime;
        }
      }
    }).observe({ entryTypes: ['largest-contentful-paint'] });
    
    new PerformanceObserver((list) => {
      let cls = 0;
      for (const entry of list.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          cls += (entry as any).value;
        }
      }
      (window as any).performanceMetrics.CLS = cls;
    }).observe({ entryTypes: ['layout-shift'] });
  });
  
  // Navigate and wait for load
  const startTime = Date.now();
  await page.goto(url, { waitUntil: 'networkidle' });
  
  // Collect metrics
  const metrics = await page.evaluate(() => {
    const nav = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    const paint = performance.getEntriesByName('first-contentful-paint')[0];
    
    return {
      TTFB: nav.responseStart - nav.requestStart,
      FCP: paint ? paint.startTime : 0,
      LCP: (window as any).performanceMetrics?.LCP || 0,
      CLS: (window as any).performanceMetrics?.CLS || 0,
      loadTime: Date.now() - nav.fetchStart,
    };
  });
  
  // Check bundle sizes
  const resources = await page.evaluate(() => {
    const entries = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
    const sizes = {
      jsSize: 0,
      cssSize: 0,
      imageSize: 0,
      fontSize: 0,
      totalSize: 0,
    };
    
    entries.forEach(entry => {
      const size = entry.transferSize / 1024; // Convert to KB
      sizes.totalSize += size;
      
      if (entry.name.includes('.js')) sizes.jsSize += size;
      else if (entry.name.includes('.css')) sizes.cssSize += size;
      else if (entry.name.match(/\.(jpg|jpeg|png|gif|webp|avif|svg)/)) sizes.imageSize += size;
      else if (entry.name.match(/\.(woff|woff2|ttf|eot)/)) sizes.fontSize += size;
    });
    
    return sizes;
  });
  
  await browser.close();
  
  // Check against budget
  const result = checkBudget(
    { metrics: metrics as any, resources },
    recommendedBudget
  );
  
  console.log('\n📊 Performance Audit Results');
  console.log('============================\n');
  
  console.log('Metrics:');
  Object.entries(metrics).forEach(([key, value]) => {
    const unit = key === 'CLS' ? '' : 'ms';
    console.log(`  ${key}: ${typeof value === 'number' ? value.toFixed(2) : value}${unit}`);
  });
  
  console.log('\nResources:');
  Object.entries(resources).forEach(([key, value]) => {
    console.log(`  ${key}: ${value.toFixed(2)}KB`);
  });
  
  if (result.violations.length > 0) {
    console.log('\n❌ Violations:');
    result.violations.forEach(v => console.log(`  - ${v}`));
  }
  
  if (result.warnings.length > 0) {
    console.log('\n⚠️ Warnings:');
    result.warnings.forEach(w => console.log(`  - ${w}`));
  }
  
  if (result.passed) {
    console.log('\n✅ Performance budget: PASSED');
  } else {
    console.log('\n❌ Performance budget: FAILED');
    process.exit(1);
  }
}

// Run audit
const url = process.argv[2] || 'http://localhost:3000';
runPerformanceAudit(url).catch(console.error);
```

---

## 5. Error Tracking - Sentry

### 5.1 Introduzione a Sentry

Sentry è la piattaforma leader per error tracking e performance monitoring. Per Next.js offre funzionalità avanzate come Session Replay, Profiling e Tracing.

### 5.2 Setup Sentry in Next.js

```bash
# Installazione automatica con wizard
npx @sentry/wizard@latest -i nextjs

# Oppure installazione manuale
pnpm add @sentry/nextjs
```

### 5.3 Configurazione File

```typescript
// sentry.client.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  
  // Environment-aware settings
  enabled: process.env.NODE_ENV === 'production',
  environment: process.env.NODE_ENV,
  
  // Release tracking
  release: process.env.NEXT_PUBLIC_APP_VERSION || 'development',
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.2 : 1.0,
  
  // Profiling
  profilesSampleRate: 0.1,
  
  // Session Replay
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  
  // Debugging
  debug: process.env.NODE_ENV === 'development',
  
  // Integrations
  integrations: [
    Sentry.replayIntegration({
      // Privacy settings
      maskAllText: true,
      maskAllInputs: true,
      blockAllMedia: false,
    }),
    Sentry.browserTracingIntegration(),
    Sentry.feedbackIntegration({
      colorScheme: 'system',
      showBranding: false,
    }),
  ],
  
  // Filter errors
  ignoreErrors: [
    // Common browser errors to ignore
    'ResizeObserver loop limit exceeded',
    'ResizeObserver loop completed with undelivered notifications',
    'Non-Error exception captured',
    'Non-Error promise rejection captured',
    // Network errors
    'Failed to fetch',
    'NetworkError',
    'Load failed',
    // User-triggered
    'AbortError',
  ],
  
  // Before send hook
  beforeSend(event, hint) {
    // Filter out development errors
    if (process.env.NODE_ENV === 'development') {
      console.log('[Sentry] Would send:', event);
      return null;
    }
    
    // Sanitize sensitive data
    if (event.request?.headers) {
      delete event.request.headers['Authorization'];
      delete event.request.headers['Cookie'];
    }
    
    return event;
  },
  
  // Before breadcrumb
  beforeBreadcrumb(breadcrumb) {
    // Filter sensitive breadcrumbs
    if (breadcrumb.category === 'xhr' || breadcrumb.category === 'fetch') {
      const url = breadcrumb.data?.url as string;
      if (url?.includes('/api/auth')) {
        return null; // Don't track auth requests
      }
    }
    return breadcrumb;
  },
});

// Capture router transitions for Next.js App Router
export const onRouterTransitionStart = Sentry.captureRouterTransitionStart;
```

```typescript
// sentry.server.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  
  enabled: process.env.NODE_ENV === 'production',
  environment: process.env.NODE_ENV,
  release: process.env.NEXT_PUBLIC_APP_VERSION,
  
  // Performance
  tracesSampleRate: 0.2,
  profilesSampleRate: 0.1,
  
  // Server-specific settings
  integrations: [
    // Database query tracing
    Sentry.prismaIntegration(),
    // HTTP tracing
    Sentry.httpIntegration(),
  ],
  
  // Filtering
  ignoreErrors: [
    'ECONNREFUSED',
    'ECONNRESET',
    'ETIMEDOUT',
  ],
});
```

```typescript
// sentry.edge.config.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  
  enabled: process.env.NODE_ENV === 'production',
  environment: process.env.NODE_ENV,
  release: process.env.NEXT_PUBLIC_APP_VERSION,
  
  tracesSampleRate: 0.2,
});
```

### 5.4 Instrumentation File

```typescript
// instrumentation.ts
import * as Sentry from '@sentry/nextjs';

export async function register() {
  if (process.env.NEXT_RUNTIME === 'nodejs') {
    await import('./sentry.server.config');
  }

  if (process.env.NEXT_RUNTIME === 'edge') {
    await import('./sentry.edge.config');
  }
}

export const onRequestError = Sentry.captureRequestError;
```

### 5.5 Global Error Component

```typescript
// app/global-error.tsx
'use client';

import * as Sentry from '@sentry/nextjs';
import { useEffect } from 'react';

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    Sentry.captureException(error);
  }, [error]);

  return (
    <html>
      <body>
        <div className="min-h-screen flex items-center justify-center bg-gray-100">
          <div className="max-w-md w-full bg-white rounded-lg shadow-lg p-8 text-center">
            <h1 className="text-2xl font-bold text-gray-900 mb-4">
              Qualcosa è andato storto
            </h1>
            <p className="text-gray-600 mb-6">
              Si è verificato un errore imprevisto. Il nostro team è stato notificato.
            </p>
            {error.digest && (
              <p className="text-sm text-gray-400 mb-4">
                Error ID: {error.digest}
              </p>
            )}
            <button
              onClick={reset}
              className="px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
            >
              Riprova
            </button>
          </div>
        </div>
      </body>
    </html>
  );
}
```

### 5.6 Error Boundary Component

```typescript
// components/ErrorBoundary.tsx
'use client';

import * as Sentry from '@sentry/nextjs';
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error?: Error;
  eventId?: string;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    const eventId = Sentry.captureException(error, {
      extra: {
        componentStack: errorInfo.componentStack,
      },
    });
    
    this.setState({ eventId });
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <div className="p-6 bg-red-50 border border-red-200 rounded-lg">
          <h2 className="text-lg font-semibold text-red-800 mb-2">
            Si è verificato un errore
          </h2>
          <p className="text-red-600 mb-4">
            {this.state.error?.message}
          </p>
          <button
            onClick={() => {
              if (this.state.eventId) {
                Sentry.showReportDialog({ eventId: this.state.eventId });
              }
            }}
            className="px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700"
          >
            Segnala problema
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 5.7 Custom Error Tracking Utilities

```typescript
// lib/error-tracking/sentry-utils.ts
import * as Sentry from '@sentry/nextjs';

// Error context types
interface ErrorContext {
  userId?: string;
  feature?: string;
  component?: string;
  action?: string;
  metadata?: Record<string, unknown>;
}

// Capture error with context
export const captureError = (
  error: Error | string,
  context?: ErrorContext
): string => {
  const eventId = Sentry.captureException(
    typeof error === 'string' ? new Error(error) : error,
    {
      tags: {
        feature: context?.feature,
        component: context?.component,
        action: context?.action,
      },
      extra: context?.metadata,
      user: context?.userId ? { id: context.userId } : undefined,
    }
  );
  
  // Log in development
  if (process.env.NODE_ENV === 'development') {
    console.error('[Error Tracked]', error, context);
  }
  
  return eventId;
};

// Capture message (non-error event)
export const captureMessage = (
  message: string,
  level: Sentry.SeverityLevel = 'info',
  context?: ErrorContext
): string => {
  return Sentry.captureMessage(message, {
    level,
    tags: {
      feature: context?.feature,
      component: context?.component,
    },
    extra: context?.metadata,
  });
};

// Set user context
export const setUser = (user: {
  id: string;
  email?: string;
  username?: string;
  [key: string]: unknown;
} | null) => {
  Sentry.setUser(user);
};

// Add breadcrumb
export const addBreadcrumb = (
  message: string,
  category: string,
  data?: Record<string, unknown>,
  level: Sentry.SeverityLevel = 'info'
) => {
  Sentry.addBreadcrumb({
    message,
    category,
    data,
    level,
    timestamp: Date.now() / 1000,
  });
};

// Set tags for context
export const setTags = (tags: Record<string, string>) => {
  Object.entries(tags).forEach(([key, value]) => {
    Sentry.setTag(key, value);
  });
};

// Wrap async function with error tracking
export const withErrorTracking = <T extends (...args: any[]) => Promise<any>>(
  fn: T,
  context?: ErrorContext
): T => {
  return (async (...args: Parameters<T>) => {
    const transaction = Sentry.startInactiveSpan({
      name: context?.action || fn.name || 'anonymous',
      op: context?.feature || 'function',
    });
    
    try {
      const result = await fn(...args);
      transaction?.end();
      return result;
    } catch (error) {
      captureError(error as Error, context);
      transaction?.end();
      throw error;
    }
  }) as T;
};

// Sentry logger (sends logs to Sentry)
export const sentryLogger = {
  info: (message: string, data?: Record<string, unknown>) => {
    Sentry.logger?.info(message, data);
  },
  warn: (message: string, data?: Record<string, unknown>) => {
    Sentry.logger?.warn(message, data);
  },
  error: (message: string, data?: Record<string, unknown>) => {
    Sentry.logger?.error(message, data);
  },
};

// Performance measurement
export const measurePerformance = async <T>(
  name: string,
  operation: () => Promise<T>,
  tags?: Record<string, string>
): Promise<T> => {
  const span = Sentry.startInactiveSpan({
    name,
    op: 'function',
    attributes: tags,
  });
  
  try {
    const result = await operation();
    span?.end();
    return result;
  } catch (error) {
    span?.end();
    throw error;
  }
};
```

### 5.8 Next.js Configuration per Sentry

```javascript
// next.config.js
const { withSentryConfig } = require('@sentry/nextjs');

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Your existing config
};

const sentryWebpackPluginOptions = {
  // Sentry organization and project
  org: process.env.SENTRY_ORG,
  project: process.env.SENTRY_PROJECT,
  
  // Auth token for uploading source maps
  authToken: process.env.SENTRY_AUTH_TOKEN,
  
  // Only upload source maps in production
  silent: process.env.NODE_ENV !== 'production',
  
  // Upload source maps for better stack traces
  widenClientFileUpload: true,
  
  // Hide source maps from users
  hideSourceMaps: true,
  
  // Disable logger injection to reduce bundle size
  disableLogger: true,
  
  // Automatically tree-shake Sentry
  automaticVercelMonitors: true,
};

module.exports = withSentryConfig(nextConfig, sentryWebpackPluginOptions);
```



---

## 6. Application Performance Monitoring (APM)

### 6.1 Introduzione all'APM

L'Application Performance Monitoring (APM) fornisce visibilità end-to-end sulle performance dell'applicazione, tracciando richieste attraverso tutti i servizi.

```
┌─────────────────────────────────────────────────────────────────┐
│                       APM OVERVIEW                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client Request                                                │
│        │                                                        │
│        ▼                                                        │
│   ┌─────────────────────────────────────────────────────────┐  │
│   │                  TRACE ID: abc123                        │  │
│   │                                                          │  │
│   │   ┌─────────┐    ┌─────────┐    ┌─────────┐            │  │
│   │   │ Next.js │───▶│   API   │───▶│Database │            │  │
│   │   │ Server  │    │ Route   │    │  Query  │            │  │
│   │   └─────────┘    └─────────┘    └─────────┘            │  │
│   │      50ms          120ms           80ms                  │  │
│   │                                                          │  │
│   │   Total Request Time: 250ms                             │  │
│   └─────────────────────────────────────────────────────────┘  │
│                                                                 │
│   📊 Metrics Collected:                                        │
│   • Response time per endpoint                                 │
│   • Error rates                                                │
│   • Throughput (requests/sec)                                  │
│   • Database query performance                                 │
│   • External service latency                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 6.2 APM con Sentry Performance

```typescript
// lib/apm/sentry-apm.ts
import * as Sentry from '@sentry/nextjs';

// Create custom transaction
export const startTransaction = (
  name: string,
  op: string,
  data?: Record<string, unknown>
) => {
  return Sentry.startSpan({
    name,
    op,
    attributes: data,
  }, (span) => span);
};

// Measure database query
export const measureDatabaseQuery = async <T>(
  queryName: string,
  query: () => Promise<T>,
  metadata?: Record<string, unknown>
): Promise<T> => {
  return Sentry.startSpan(
    {
      name: queryName,
      op: 'db.query',
      attributes: {
        'db.system': 'postgresql',
        ...metadata,
      },
    },
    async (span) => {
      const startTime = Date.now();
      try {
        const result = await query();
        const duration = Date.now() - startTime;
        
        // Track slow queries
        if (duration > 1000) {
          Sentry.addBreadcrumb({
            category: 'db',
            message: `Slow query: ${queryName} took ${duration}ms`,
            level: 'warning',
          });
        }
        
        return result;
      } catch (error) {
        span?.setStatus({ code: 2, message: 'Error' });
        throw error;
      }
    }
  );
};

// Measure external API call
export const measureExternalCall = async <T>(
  serviceName: string,
  url: string,
  method: string,
  call: () => Promise<T>
): Promise<T> => {
  return Sentry.startSpan(
    {
      name: `${method} ${serviceName}`,
      op: 'http.client',
      attributes: {
        'http.url': url,
        'http.method': method,
        'peer.service': serviceName,
      },
    },
    async (span) => {
      try {
        return await call();
      } catch (error) {
        span?.setStatus({ code: 2, message: 'Error' });
        throw error;
      }
    }
  );
};

// Wrapper for API routes
export const withAPM = <T extends (...args: any[]) => Promise<any>>(
  handler: T,
  routeName: string
): T => {
  return (async (...args: Parameters<T>) => {
    return Sentry.startSpan(
      {
        name: routeName,
        op: 'http.server',
      },
      async () => {
        return handler(...args);
      }
    );
  }) as T;
};
```

### 6.3 Custom Metrics

```typescript
// lib/apm/metrics.ts
import * as Sentry from '@sentry/nextjs';

// Counter metric
export const incrementCounter = (
  name: string,
  value: number = 1,
  tags?: Record<string, string>
) => {
  Sentry.metrics.increment(name, value, { tags });
};

// Gauge metric
export const setGauge = (
  name: string,
  value: number,
  tags?: Record<string, string>
) => {
  Sentry.metrics.gauge(name, value, { tags });
};

// Distribution metric (for percentiles)
export const distribution = (
  name: string,
  value: number,
  tags?: Record<string, string>
) => {
  Sentry.metrics.distribution(name, value, { tags });
};

// Timing metric
export const timing = (
  name: string,
  callback: () => Promise<void> | void,
  tags?: Record<string, string>
) => {
  return Sentry.metrics.timing(name, callback, { tags });
};

// Example usage in API route
// app/api/orders/route.ts
/*
export async function POST(request: Request) {
  incrementCounter('orders.created', 1, { source: 'web' });
  
  const order = await timing(
    'orders.processing_time',
    async () => await processOrder(request),
    { type: 'standard' }
  );
  
  setGauge('orders.queue_size', await getQueueSize());
  distribution('orders.value', order.total, { currency: 'EUR' });
  
  return Response.json(order);
}
*/
```

---

## 7. Logging & Structured Logs

### 7.1 Introduzione al Logging

Il logging strutturato è fondamentale per debugging, monitoring e auditing. In Next.js, Pino è la scelta raccomandata per le sue performance superiori.

### 7.2 Setup Pino Logger

```bash
# Installazione
pnpm add pino pino-pretty
```

```typescript
// lib/logger/index.ts
import pino, { Logger, LoggerOptions } from 'pino';

// Log levels
export type LogLevel = 'trace' | 'debug' | 'info' | 'warn' | 'error' | 'fatal';

// Logger configuration
const config: LoggerOptions = {
  level: process.env.LOG_LEVEL || (process.env.NODE_ENV === 'production' ? 'info' : 'debug'),
  
  // Redact sensitive fields
  redact: {
    paths: [
      'password',
      'token',
      'accessToken',
      'refreshToken',
      'authorization',
      'cookie',
      'req.headers.authorization',
      'req.headers.cookie',
      'res.headers["set-cookie"]',
    ],
    censor: '[REDACTED]',
  },
  
  // Base fields included in every log
  base: {
    env: process.env.NODE_ENV,
    version: process.env.npm_package_version,
    service: 'next-app',
  },
  
  // Timestamp format
  timestamp: pino.stdTimeFunctions.isoTime,
  
  // Format for development
  ...(process.env.NODE_ENV !== 'production' && {
    transport: {
      target: 'pino-pretty',
      options: {
        colorize: true,
        translateTime: 'HH:MM:ss',
        ignore: 'pid,hostname',
      },
    },
  }),
  
  // Custom serializers
  serializers: {
    err: pino.stdSerializers.err,
    error: pino.stdSerializers.err,
    req: (req) => ({
      method: req.method,
      url: req.url,
      path: req.path,
      query: req.query,
      params: req.params,
      headers: {
        'user-agent': req.headers?.['user-agent'],
        'content-type': req.headers?.['content-type'],
      },
    }),
    res: (res) => ({
      statusCode: res.statusCode,
    }),
  },
};

// Create logger instance
export const logger = pino(config);

// Create child logger with context
export const createLogger = (context: Record<string, unknown>): Logger => {
  return logger.child(context);
};

// Module-specific loggers
export const apiLogger = createLogger({ module: 'api' });
export const dbLogger = createLogger({ module: 'database' });
export const authLogger = createLogger({ module: 'auth' });
export const jobLogger = createLogger({ module: 'jobs' });
```

### 7.3 Request Logger Middleware

```typescript
// lib/logger/request-logger.ts
import { NextRequest, NextResponse } from 'next/server';
import { logger, createLogger } from './index';
import { nanoid } from 'nanoid';

// Generate request ID
const generateRequestId = () => nanoid(12);

// Request context type
export interface RequestContext {
  requestId: string;
  method: string;
  path: string;
  startTime: number;
}

// Log request
export const logRequest = (
  req: NextRequest,
  context: RequestContext
) => {
  const reqLogger = createLogger({
    requestId: context.requestId,
    method: context.method,
    path: context.path,
  });
  
  reqLogger.info({
    type: 'request',
    headers: {
      'user-agent': req.headers.get('user-agent'),
      'content-type': req.headers.get('content-type'),
      'x-forwarded-for': req.headers.get('x-forwarded-for'),
    },
  }, `Incoming ${context.method} ${context.path}`);
  
  return reqLogger;
};

// Log response
export const logResponse = (
  reqLogger: ReturnType<typeof createLogger>,
  context: RequestContext,
  statusCode: number
) => {
  const duration = Date.now() - context.startTime;
  
  const logLevel = statusCode >= 500 ? 'error' : 
                   statusCode >= 400 ? 'warn' : 'info';
  
  reqLogger[logLevel]({
    type: 'response',
    statusCode,
    duration,
  }, `${context.method} ${context.path} ${statusCode} - ${duration}ms`);
};

// Middleware factory for API routes
export const withRequestLogging = <T>(
  handler: (req: NextRequest, context: RequestContext) => Promise<T>
) => {
  return async (req: NextRequest): Promise<T> => {
    const context: RequestContext = {
      requestId: generateRequestId(),
      method: req.method,
      path: new URL(req.url).pathname,
      startTime: Date.now(),
    };
    
    const reqLogger = logRequest(req, context);
    
    try {
      const result = await handler(req, context);
      
      if (result instanceof NextResponse) {
        logResponse(reqLogger, context, result.status);
      }
      
      return result;
    } catch (error) {
      reqLogger.error({
        type: 'error',
        error: error instanceof Error ? {
          message: error.message,
          stack: error.stack,
          name: error.name,
        } : error,
      }, 'Request failed');
      
      throw error;
    }
  };
};
```

### 7.4 Structured Log Events

```typescript
// lib/logger/events.ts
import { logger, createLogger } from './index';

// Business event logger
export const logBusinessEvent = (
  eventName: string,
  data: Record<string, unknown>,
  userId?: string
) => {
  const eventLogger = createLogger({
    eventType: 'business',
    eventName,
    userId,
  });
  
  eventLogger.info(data, `Business event: ${eventName}`);
};

// Audit logger
export const logAuditEvent = (
  action: string,
  resource: string,
  resourceId: string,
  userId: string,
  changes?: Record<string, unknown>
) => {
  const auditLogger = createLogger({
    eventType: 'audit',
    action,
    resource,
    resourceId,
    userId,
  });
  
  auditLogger.info({ changes }, `Audit: ${action} ${resource}/${resourceId}`);
};

// Security logger
export const logSecurityEvent = (
  eventType: 'login' | 'logout' | 'failed_login' | 'permission_denied' | 'suspicious_activity',
  details: Record<string, unknown>,
  severity: 'info' | 'warn' | 'error' = 'info'
) => {
  const securityLogger = createLogger({
    eventType: 'security',
    securityEvent: eventType,
  });
  
  securityLogger[severity](details, `Security: ${eventType}`);
};

// Performance logger
export const logPerformance = (
  operation: string,
  duration: number,
  metadata?: Record<string, unknown>
) => {
  const perfLogger = createLogger({
    eventType: 'performance',
    operation,
  });
  
  const level = duration > 5000 ? 'warn' : 'info';
  
  perfLogger[level]({
    duration,
    ...metadata,
  }, `Performance: ${operation} took ${duration}ms`);
};

// Usage examples:
/*
// Business event
logBusinessEvent('order_placed', {
  orderId: 'ORD-123',
  total: 99.99,
  itemCount: 3,
}, 'user-456');

// Audit event
logAuditEvent('update', 'user', 'user-123', 'admin-1', {
  before: { role: 'user' },
  after: { role: 'admin' },
});

// Security event
logSecurityEvent('failed_login', {
  email: 'user@example.com',
  ip: '192.168.1.1',
  attempts: 3,
}, 'warn');

// Performance event
logPerformance('database_query', 1523, {
  query: 'SELECT * FROM orders',
  resultCount: 100,
});
*/
```

### 7.5 Log Aggregation Setup

```typescript
// lib/logger/transports.ts
import pino from 'pino';

// Create multi-transport logger for production
export const createProductionLogger = () => {
  return pino({
    level: process.env.LOG_LEVEL || 'info',
    
    // Multiple destinations
    transport: {
      targets: [
        // Console output (JSON in production)
        {
          target: 'pino/file',
          options: { destination: 1 }, // stdout
          level: 'info',
        },
        // File output for errors
        {
          target: 'pino/file',
          options: {
            destination: './logs/error.log',
            mkdir: true,
          },
          level: 'error',
        },
        // HTTP transport to log aggregator
        {
          target: 'pino-http-send',
          options: {
            url: process.env.LOG_AGGREGATOR_URL,
            headers: {
              Authorization: `Bearer ${process.env.LOG_AGGREGATOR_TOKEN}`,
            },
            batchSize: 100,
            interval: 5000,
          },
          level: 'info',
        },
      ],
    },
  });
};

// Loki transport configuration
export const lokiTransport = {
  target: 'pino-loki',
  options: {
    host: process.env.LOKI_HOST || 'http://localhost:3100',
    batching: true,
    interval: 5,
    labels: {
      app: 'next-app',
      env: process.env.NODE_ENV,
    },
  },
};
```

### 7.6 Log Rotation

```typescript
// lib/logger/rotation.ts
import pino from 'pino';

// Logger with file rotation
export const createRotatingLogger = () => {
  return pino({
    level: 'info',
    transport: {
      target: 'pino-roll',
      options: {
        file: './logs/app',
        frequency: 'daily', // Rotate daily
        size: '10MB',       // Or when file reaches 10MB
        mkdir: true,
        extension: '.log',
        dateFormat: 'yyyy-MM-dd',
        limit: {
          count: 7, // Keep 7 days of logs
        },
      },
    },
  });
};
```

---

## 8. Real User Monitoring (RUM)

### 8.1 Introduzione al RUM

Real User Monitoring cattura le metriche di performance direttamente dai browser degli utenti reali, fornendo dati più accurati rispetto ai test sintetici.

### 8.2 Custom RUM Implementation

```typescript
// lib/rum/index.ts
'use client';

interface RUMData {
  // Navigation timing
  dns: number;
  tcp: number;
  ssl: number;
  ttfb: number;
  download: number;
  domInteractive: number;
  domComplete: number;
  loadTime: number;
  
  // Resource timing
  resourceCount: number;
  resourceSize: number;
  jsSize: number;
  cssSize: number;
  imageSize: number;
  
  // Page info
  url: string;
  referrer: string;
  
  // User info
  userAgent: string;
  connection?: string;
  deviceMemory?: number;
  hardwareConcurrency?: number;
}

// Collect RUM data
export const collectRUMData = (): RUMData | null => {
  if (typeof window === 'undefined') return null;
  
  const nav = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
  if (!nav) return null;
  
  const resources = performance.getEntriesByType('resource') as PerformanceResourceTiming[];
  
  // Calculate resource sizes
  let jsSize = 0, cssSize = 0, imageSize = 0, totalSize = 0;
  resources.forEach(r => {
    const size = r.transferSize || 0;
    totalSize += size;
    if (r.name.includes('.js')) jsSize += size;
    else if (r.name.includes('.css')) cssSize += size;
    else if (r.name.match(/\.(jpg|jpeg|png|gif|webp|svg)/)) imageSize += size;
  });
  
  return {
    // Navigation timing
    dns: nav.domainLookupEnd - nav.domainLookupStart,
    tcp: nav.connectEnd - nav.connectStart,
    ssl: nav.secureConnectionStart > 0 ? nav.connectEnd - nav.secureConnectionStart : 0,
    ttfb: nav.responseStart - nav.requestStart,
    download: nav.responseEnd - nav.responseStart,
    domInteractive: nav.domInteractive - nav.fetchStart,
    domComplete: nav.domComplete - nav.fetchStart,
    loadTime: nav.loadEventEnd - nav.fetchStart,
    
    // Resources
    resourceCount: resources.length,
    resourceSize: totalSize,
    jsSize,
    cssSize,
    imageSize,
    
    // Page info
    url: window.location.href,
    referrer: document.referrer,
    
    // User info
    userAgent: navigator.userAgent,
    connection: (navigator as any).connection?.effectiveType,
    deviceMemory: (navigator as any).deviceMemory,
    hardwareConcurrency: navigator.hardwareConcurrency,
  };
};

// Send RUM data
export const sendRUMData = async (data: RUMData) => {
  const endpoint = process.env.NEXT_PUBLIC_RUM_ENDPOINT;
  if (!endpoint) return;
  
  try {
    await fetch(endpoint, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        ...data,
        timestamp: Date.now(),
        sessionId: getSessionId(),
      }),
      keepalive: true,
    });
  } catch (error) {
    console.error('Failed to send RUM data:', error);
  }
};

// Session ID management
const getSessionId = (): string => {
  const key = 'rum_session_id';
  let sessionId = sessionStorage.getItem(key);
  
  if (!sessionId) {
    sessionId = `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    sessionStorage.setItem(key, sessionId);
  }
  
  return sessionId;
};
```

### 8.3 RUM Component

```typescript
// components/performance/RUMCollector.tsx
'use client';

import { useEffect } from 'react';
import { collectRUMData, sendRUMData } from '@/lib/rum';

export function RUMCollector() {
  useEffect(() => {
    // Wait for page to fully load
    if (document.readyState === 'complete') {
      setTimeout(collectAndSend, 0);
    } else {
      window.addEventListener('load', () => {
        setTimeout(collectAndSend, 0);
      });
    }
    
    function collectAndSend() {
      const data = collectRUMData();
      if (data) {
        sendRUMData(data);
      }
    }
  }, []);
  
  return null;
}
```

---

## 9. Observability - OpenTelemetry

### 9.1 Introduzione a OpenTelemetry

OpenTelemetry (OTel) è lo standard CNCF per la raccolta di telemetria (traces, metrics, logs). Fornisce un'interfaccia vendor-neutral per instrumentare le applicazioni.

```
┌─────────────────────────────────────────────────────────────────┐
│                    OPENTELEMETRY ARCHITECTURE                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐        │
│  │   Next.js   │    │   API       │    │  Database   │        │
│  │    App      │    │  Service    │    │   (Prisma)  │        │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘        │
│         │                  │                   │                │
│         └────────┬─────────┴───────────────────┘                │
│                  │                                              │
│                  ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              OpenTelemetry SDK                           │   │
│  │   • Auto-instrumentation                                 │   │
│  │   • Context propagation                                  │   │
│  │   • Sampling                                             │   │
│  └─────────────────────────────────────────────────────────┘   │
│                  │                                              │
│                  ▼                                              │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              OpenTelemetry Collector                     │   │
│  │   • Receives telemetry                                   │   │
│  │   • Processes/transforms                                 │   │
│  │   • Exports to backends                                  │   │
│  └─────────────────────────────────────────────────────────┘   │
│                  │                                              │
│         ┌───────┴───────┬───────────────┐                      │
│         ▼               ▼               ▼                      │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐                 │
│  │  Jaeger  │    │Prometheus│    │  Grafana │                 │
│  │ (Traces) │    │ (Metrics)│    │  (Viz)   │                 │
│  └──────────┘    └──────────┘    └──────────┘                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### 9.2 Setup OpenTelemetry in Next.js

```bash
# Installazione
pnpm add @opentelemetry/api \
  @opentelemetry/sdk-node \
  @opentelemetry/auto-instrumentations-node \
  @opentelemetry/exporter-trace-otlp-http \
  @opentelemetry/exporter-metrics-otlp-http \
  @opentelemetry/resources \
  @opentelemetry/semantic-conventions
```

```typescript
// instrumentation.node.ts
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { OTLPMetricExporter } from '@opentelemetry/exporter-metrics-otlp-http';
import { PeriodicExportingMetricReader } from '@opentelemetry/sdk-metrics';
import { Resource } from '@opentelemetry/resources';
import { SEMRESATTRS_SERVICE_NAME, SEMRESATTRS_SERVICE_VERSION, SEMRESATTRS_DEPLOYMENT_ENVIRONMENT } from '@opentelemetry/semantic-conventions';

// Service resource
const resource = new Resource({
  [SEMRESATTRS_SERVICE_NAME]: process.env.OTEL_SERVICE_NAME || 'next-app',
  [SEMRESATTRS_SERVICE_VERSION]: process.env.npm_package_version || '1.0.0',
  [SEMRESATTRS_DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV || 'development',
});

// Trace exporter
const traceExporter = new OTLPTraceExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT 
    ? `${process.env.OTEL_EXPORTER_OTLP_ENDPOINT}/v1/traces`
    : 'http://localhost:4318/v1/traces',
  headers: process.env.OTEL_EXPORTER_OTLP_HEADERS 
    ? JSON.parse(process.env.OTEL_EXPORTER_OTLP_HEADERS)
    : {},
});

// Metric exporter
const metricExporter = new OTLPMetricExporter({
  url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT
    ? `${process.env.OTEL_EXPORTER_OTLP_ENDPOINT}/v1/metrics`
    : 'http://localhost:4318/v1/metrics',
});

// Initialize SDK
const sdk = new NodeSDK({
  resource,
  traceExporter,
  metricReader: new PeriodicExportingMetricReader({
    exporter: metricExporter,
    exportIntervalMillis: 15000, // Export every 15 seconds
  }),
  instrumentations: [
    getNodeAutoInstrumentations({
      // Disable fs instrumentation (too noisy)
      '@opentelemetry/instrumentation-fs': {
        enabled: false,
      },
      // Configure HTTP instrumentation
      '@opentelemetry/instrumentation-http': {
        ignoreIncomingRequestHook: (request) => {
          // Ignore health checks and static assets
          const path = request.url || '';
          return path.includes('/health') || 
                 path.includes('/_next/') || 
                 path.includes('/favicon');
        },
      },
    }),
  ],
});

// Start SDK
sdk.start();

// Graceful shutdown
process.on('SIGTERM', () => {
  sdk.shutdown()
    .then(() => console.log('OpenTelemetry SDK shut down'))
    .catch((error) => console.error('Error shutting down SDK', error))
    .finally(() => process.exit(0));
});

export { sdk };
```

### 9.3 Custom Spans and Metrics

```typescript
// lib/otel/tracing.ts
import { trace, SpanStatusCode, context, propagation } from '@opentelemetry/api';
import { metrics } from '@opentelemetry/api';

// Get tracer
const tracer = trace.getTracer('next-app', '1.0.0');

// Get meter
const meter = metrics.getMeter('next-app', '1.0.0');

// Create counters
const requestCounter = meter.createCounter('http_requests_total', {
  description: 'Total HTTP requests',
});

const errorCounter = meter.createCounter('errors_total', {
  description: 'Total errors',
});

// Create histograms
const requestDuration = meter.createHistogram('http_request_duration_ms', {
  description: 'HTTP request duration in milliseconds',
});

// Create custom span
export const createSpan = async <T>(
  name: string,
  operation: () => Promise<T>,
  attributes?: Record<string, string | number | boolean>
): Promise<T> => {
  return tracer.startActiveSpan(name, async (span) => {
    try {
      if (attributes) {
        span.setAttributes(attributes);
      }
      
      const result = await operation();
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (error) {
      span.setStatus({
        code: SpanStatusCode.ERROR,
        message: error instanceof Error ? error.message : 'Unknown error',
      });
      
      if (error instanceof Error) {
        span.recordException(error);
      }
      
      throw error;
    } finally {
      span.end();
    }
  });
};

// Trace database operation
export const traceDbOperation = async <T>(
  operation: string,
  query: () => Promise<T>,
  metadata?: Record<string, string>
): Promise<T> => {
  return createSpan(
    `db.${operation}`,
    query,
    {
      'db.system': 'postgresql',
      'db.operation': operation,
      ...metadata,
    }
  );
};

// Trace external HTTP call
export const traceHttpCall = async <T>(
  url: string,
  method: string,
  call: () => Promise<T>
): Promise<T> => {
  return createSpan(
    `HTTP ${method}`,
    call,
    {
      'http.url': url,
      'http.method': method,
    }
  );
};

// Record metrics
export const recordMetrics = {
  requestStart: () => {
    requestCounter.add(1);
    return Date.now();
  },
  requestEnd: (startTime: number, route: string, statusCode: number) => {
    requestDuration.record(Date.now() - startTime, {
      route,
      status_code: statusCode.toString(),
    });
  },
  error: (type: string) => {
    errorCounter.add(1, { type });
  },
};

// Context propagation helpers
export const extractTraceContext = (headers: Headers): context.Context => {
  const carrier: Record<string, string> = {};
  headers.forEach((value, key) => {
    carrier[key] = value;
  });
  return propagation.extract(context.active(), carrier);
};

export const injectTraceContext = (headers: Headers): void => {
  const carrier: Record<string, string> = {};
  propagation.inject(context.active(), carrier);
  Object.entries(carrier).forEach(([key, value]) => {
    headers.set(key, value);
  });
};
```

### 9.4 OpenTelemetry Collector Configuration

```yaml
# otel-collector-config.yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318

processors:
  batch:
    timeout: 1s
    send_batch_size: 1024
  
  memory_limiter:
    check_interval: 1s
    limit_mib: 1000
    spike_limit_mib: 200
  
  resource:
    attributes:
      - key: environment
        value: production
        action: insert

exporters:
  # Jaeger for traces
  jaeger:
    endpoint: jaeger:14250
    tls:
      insecure: true
  
  # Prometheus for metrics
  prometheus:
    endpoint: "0.0.0.0:8889"
    namespace: nextapp
  
  # Loki for logs
  loki:
    endpoint: http://loki:3100/loki/api/v1/push
    labels:
      attributes:
        service: service_name
        level: level
  
  # Debug (development only)
  logging:
    loglevel: debug

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [jaeger]
    
    metrics:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [prometheus]
    
    logs:
      receivers: [otlp]
      processors: [memory_limiter, batch]
      exporters: [loki]
```



---

## 10. Metrics & Dashboards - Prometheus & Grafana

### 10.1 Introduzione a Prometheus

Prometheus è il sistema di monitoring standard per metriche time-series, integrato nativamente con Kubernetes e ampiamente supportato.

### 10.2 Custom Metrics Endpoint

```typescript
// app/api/metrics/route.ts
import { NextResponse } from 'next/server';
import { register, Counter, Histogram, Gauge } from 'prom-client';

// Initialize default metrics
import { collectDefaultMetrics } from 'prom-client';
collectDefaultMetrics({ prefix: 'nextjs_' });

// Custom metrics
export const httpRequestsTotal = new Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

export const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.05, 0.1, 0.3, 0.5, 1, 2, 5],
});

export const activeConnections = new Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

export const errorRate = new Counter({
  name: 'errors_total',
  help: 'Total errors',
  labelNames: ['type', 'route'],
});

// Business metrics
export const ordersTotal = new Counter({
  name: 'orders_total',
  help: 'Total orders placed',
  labelNames: ['status', 'payment_method'],
});

export const orderValue = new Histogram({
  name: 'order_value_euros',
  help: 'Order value in euros',
  buckets: [10, 25, 50, 100, 250, 500, 1000],
});

export const activeUsers = new Gauge({
  name: 'active_users',
  help: 'Number of active users',
});

// Expose metrics endpoint
export async function GET() {
  try {
    const metrics = await register.metrics();
    return new NextResponse(metrics, {
      headers: {
        'Content-Type': register.contentType,
      },
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to collect metrics' },
      { status: 500 }
    );
  }
}
```

### 10.3 Metrics Middleware

```typescript
// lib/metrics/middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { httpRequestsTotal, httpRequestDuration, errorRate } from '@/app/api/metrics/route';

export function withMetrics<T extends NextResponse>(
  handler: (req: NextRequest) => Promise<T>,
  route: string
) {
  return async (req: NextRequest): Promise<T> => {
    const startTime = Date.now();
    const method = req.method;
    
    try {
      const response = await handler(req);
      const statusCode = response.status.toString();
      const duration = (Date.now() - startTime) / 1000;
      
      // Record metrics
      httpRequestsTotal.inc({ method, route, status_code: statusCode });
      httpRequestDuration.observe({ method, route, status_code: statusCode }, duration);
      
      return response;
    } catch (error) {
      const duration = (Date.now() - startTime) / 1000;
      
      httpRequestsTotal.inc({ method, route, status_code: '500' });
      httpRequestDuration.observe({ method, route, status_code: '500' }, duration);
      errorRate.inc({ type: 'handler_error', route });
      
      throw error;
    }
  };
}

// Usage in API route:
// export const GET = withMetrics(async (req) => { ... }, '/api/users');
```

### 10.4 Grafana Dashboard JSON

```json
{
  "dashboard": {
    "title": "Next.js Application Dashboard",
    "uid": "nextjs-app",
    "panels": [
      {
        "title": "Request Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m]))",
            "legendFormat": "req/s"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "reqps"
          }
        },
        "gridPos": { "h": 4, "w": 6, "x": 0, "y": 0 }
      },
      {
        "title": "Error Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status_code=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
            "legendFormat": "%"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "steps": [
                { "value": 0, "color": "green" },
                { "value": 1, "color": "yellow" },
                { "value": 5, "color": "red" }
              ]
            }
          }
        },
        "gridPos": { "h": 4, "w": 6, "x": 6, "y": 0 }
      },
      {
        "title": "Response Time (p95)",
        "type": "stat",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
            "legendFormat": "p95"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "s",
            "thresholds": {
              "steps": [
                { "value": 0, "color": "green" },
                { "value": 0.5, "color": "yellow" },
                { "value": 1, "color": "red" }
              ]
            }
          }
        },
        "gridPos": { "h": 4, "w": 6, "x": 12, "y": 0 }
      },
      {
        "title": "Request Rate by Route",
        "type": "timeseries",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (route)",
            "legendFormat": "{{route}}"
          }
        ],
        "gridPos": { "h": 8, "w": 12, "x": 0, "y": 4 }
      },
      {
        "title": "Response Time Distribution",
        "type": "heatmap",
        "targets": [
          {
            "expr": "sum(rate(http_request_duration_seconds_bucket[5m])) by (le)",
            "format": "heatmap",
            "legendFormat": "{{le}}"
          }
        ],
        "gridPos": { "h": 8, "w": 12, "x": 12, "y": 4 }
      },
      {
        "title": "Core Web Vitals",
        "type": "timeseries",
        "targets": [
          {
            "expr": "avg(web_vital_value{metric_name=\"LCP\"})",
            "legendFormat": "LCP"
          },
          {
            "expr": "avg(web_vital_value{metric_name=\"INP\"})",
            "legendFormat": "INP"
          },
          {
            "expr": "avg(web_vital_value{metric_name=\"CLS\"}) * 1000",
            "legendFormat": "CLS (x1000)"
          }
        ],
        "gridPos": { "h": 8, "w": 24, "x": 0, "y": 12 }
      }
    ]
  }
}
```

### 10.5 Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: 'nextjs-app'
    static_configs:
      - targets: ['nextjs-app:3000']
    metrics_path: '/api/metrics'
    scrape_interval: 10s
    
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
    
  - job_name: 'postgres-exporter'
    static_configs:
      - targets: ['postgres-exporter:9187']
```

---

## 11. Alerting & Incident Management

### 11.1 Alert Rules per Prometheus

```yaml
# alert-rules.yml
groups:
  - name: nextjs-app
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | printf \"%.2f\" }}% (threshold: 5%)"
          runbook_url: "https://runbooks.example.com/high-error-rate"
      
      # Slow response times
      - alert: SlowResponseTime
        expr: |
          histogram_quantile(0.95, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le, route)
          ) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Slow response times on {{ $labels.route }}"
          description: "P95 response time is {{ $value | printf \"%.2f\" }}s"
      
      # High memory usage
      - alert: HighMemoryUsage
        expr: |
          process_resident_memory_bytes / 1024 / 1024 > 512
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value | printf \"%.0f\" }}MB"
      
      # Application down
      - alert: ApplicationDown
        expr: up{job="nextjs-app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Application is down"
          description: "The Next.js application has been down for more than 1 minute"
      
      # Core Web Vitals degradation
      - alert: PoorLCP
        expr: avg(web_vital_value{metric_name="LCP"}) > 4000
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "LCP is degraded"
          description: "Average LCP is {{ $value | printf \"%.0f\" }}ms (target: <2500ms)"
      
      - alert: PoorINP
        expr: avg(web_vital_value{metric_name="INP"}) > 500
        for: 15m
        labels:
          severity: warning
        annotations:
          summary: "INP is degraded"
          description: "Average INP is {{ $value | printf \"%.0f\" }}ms (target: <200ms)"

  - name: business-metrics
    rules:
      - alert: LowOrderVolume
        expr: |
          sum(increase(orders_total[1h])) < 10
        for: 1h
        labels:
          severity: warning
        annotations:
          summary: "Low order volume"
          description: "Only {{ $value | printf \"%.0f\" }} orders in the last hour"
```

### 11.2 Alertmanager Configuration

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default-receiver'
  routes:
    - match:
        severity: critical
      receiver: 'critical-receiver'
      continue: true
    - match:
        severity: warning
      receiver: 'warning-receiver'

receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'team@example.com'
        send_resolved: true
  
  - name: 'critical-receiver'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts-critical'
        title: '🚨 {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true
    pagerduty_configs:
      - service_key: '${PAGERDUTY_SERVICE_KEY}'
        send_resolved: true
  
  - name: 'warning-receiver'
    slack_configs:
      - api_url: '${SLACK_WEBHOOK_URL}'
        channel: '#alerts-warning'
        title: '⚠️ {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
        send_resolved: true

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname']
```

### 11.3 Slack Integration per Alerts

```typescript
// lib/alerting/slack.ts
interface SlackAlert {
  channel: string;
  title: string;
  message: string;
  severity: 'info' | 'warning' | 'critical';
  fields?: Array<{ title: string; value: string; short?: boolean }>;
}

const severityColors = {
  info: '#36a64f',
  warning: '#ffcc00',
  critical: '#ff0000',
};

export const sendSlackAlert = async (alert: SlackAlert): Promise<void> => {
  const webhookUrl = process.env.SLACK_WEBHOOK_URL;
  if (!webhookUrl) return;
  
  const payload = {
    channel: alert.channel,
    attachments: [
      {
        color: severityColors[alert.severity],
        title: alert.title,
        text: alert.message,
        fields: alert.fields,
        footer: `${process.env.APP_NAME || 'Next.js App'} | ${new Date().toISOString()}`,
        ts: Math.floor(Date.now() / 1000),
      },
    ],
  };
  
  await fetch(webhookUrl, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload),
  });
};

// Usage:
// await sendSlackAlert({
//   channel: '#alerts',
//   title: 'High Error Rate Detected',
//   message: 'Error rate has exceeded 5% for the past 5 minutes',
//   severity: 'critical',
//   fields: [
//     { title: 'Current Rate', value: '7.2%', short: true },
//     { title: 'Threshold', value: '5%', short: true },
//   ],
// });
```

---

## 12. Session Replay & Heatmaps

### 12.1 Session Replay con Sentry

Session Replay è già incluso nella configurazione Sentry (sezione 5). Configurazione aggiuntiva:

```typescript
// lib/replay/sentry-replay.ts
import * as Sentry from '@sentry/nextjs';

// Configure replay privacy
export const replayPrivacyConfig = {
  // Mask all text by default
  maskAllText: true,
  
  // Mask all inputs
  maskAllInputs: true,
  
  // Block media
  blockAllMedia: false,
  
  // Custom selectors to mask
  maskTextSelector: '.sensitive-data, [data-sensitive], .pii',
  
  // Custom selectors to block
  blockSelector: '.do-not-record, [data-no-replay]',
  
  // Unblock specific elements
  unmaskTextSelector: '.public-text',
};

// Initialize replay with custom config
export const initReplay = () => {
  Sentry.replayIntegration({
    ...replayPrivacyConfig,
    // Network request bodies/headers to capture
    networkDetailAllowUrls: [
      '/api/public/',
    ],
    // Network requests to ignore
    networkRequestHeaders: ['X-Request-Id'],
    networkResponseHeaders: ['X-Request-Id'],
  });
};
```

### 12.2 Heatmaps con PostHog

```typescript
// lib/heatmaps/posthog-heatmap.ts
'use client';

import { posthog } from '@/lib/analytics/posthog';

// PostHog includes built-in heatmaps
// Enable autocapture for click tracking
export const enableHeatmaps = () => {
  posthog.opt_in_capturing();
  
  // Ensure toolbar is enabled for viewing heatmaps
  if (process.env.NODE_ENV === 'development') {
    posthog.loadToolbar({
      // Your PostHog toolbar token
    });
  }
};

// Manual click tracking for custom elements
export const trackElementClick = (
  elementName: string,
  elementType: string,
  metadata?: Record<string, unknown>
) => {
  posthog.capture('$autocapture', {
    $event_type: 'click',
    $element_name: elementName,
    $element_type: elementType,
    ...metadata,
  });
};
```

### 12.3 Custom Session Recording

```typescript
// lib/replay/custom-recorder.ts
'use client';

interface SessionEvent {
  type: 'click' | 'scroll' | 'input' | 'navigation' | 'error';
  timestamp: number;
  data: Record<string, unknown>;
}

class SessionRecorder {
  private events: SessionEvent[] = [];
  private sessionId: string;
  private isRecording: boolean = false;
  
  constructor() {
    this.sessionId = this.generateSessionId();
  }
  
  private generateSessionId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  start() {
    if (this.isRecording) return;
    this.isRecording = true;
    
    // Track clicks
    document.addEventListener('click', this.handleClick);
    
    // Track scroll
    document.addEventListener('scroll', this.handleScroll, { passive: true });
    
    // Track errors
    window.addEventListener('error', this.handleError);
    
    console.log(`[SessionRecorder] Started recording session: ${this.sessionId}`);
  }
  
  stop() {
    if (!this.isRecording) return;
    this.isRecording = false;
    
    document.removeEventListener('click', this.handleClick);
    document.removeEventListener('scroll', this.handleScroll);
    window.removeEventListener('error', this.handleError);
    
    console.log(`[SessionRecorder] Stopped recording. Events: ${this.events.length}`);
  }
  
  private handleClick = (e: MouseEvent) => {
    const target = e.target as HTMLElement;
    this.events.push({
      type: 'click',
      timestamp: Date.now(),
      data: {
        x: e.clientX,
        y: e.clientY,
        target: target.tagName,
        className: target.className,
        id: target.id,
        text: target.textContent?.slice(0, 50),
      },
    });
  };
  
  private handleScroll = () => {
    this.events.push({
      type: 'scroll',
      timestamp: Date.now(),
      data: {
        scrollX: window.scrollX,
        scrollY: window.scrollY,
        scrollHeight: document.body.scrollHeight,
        viewportHeight: window.innerHeight,
      },
    });
  };
  
  private handleError = (e: ErrorEvent) => {
    this.events.push({
      type: 'error',
      timestamp: Date.now(),
      data: {
        message: e.message,
        filename: e.filename,
        lineno: e.lineno,
        colno: e.colno,
      },
    });
  };
  
  getEvents(): SessionEvent[] {
    return [...this.events];
  }
  
  async flush() {
    if (this.events.length === 0) return;
    
    const eventsToSend = this.events.splice(0, this.events.length);
    
    try {
      await fetch('/api/sessions', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          sessionId: this.sessionId,
          events: eventsToSend,
        }),
        keepalive: true,
      });
    } catch (error) {
      console.error('[SessionRecorder] Failed to flush events:', error);
      // Re-add events on failure
      this.events.unshift(...eventsToSend);
    }
  }
}

export const sessionRecorder = new SessionRecorder();
```

---

## 13. A/B Testing & Feature Flags

### 13.1 A/B Testing con PostHog

```typescript
// lib/experiments/posthog-experiments.ts
'use client';

import { posthog } from '@/lib/analytics/posthog';
import { usePostHog } from 'posthog-js/react';
import { useEffect, useState } from 'react';

// Get experiment variant
export const getExperimentVariant = (experimentKey: string): string | undefined => {
  return posthog.getFeatureFlag(experimentKey) as string | undefined;
};

// Hook for experiments
export function useExperiment(experimentKey: string): {
  variant: string | undefined;
  isLoading: boolean;
} {
  const posthog = usePostHog();
  const [variant, setVariant] = useState<string | undefined>(undefined);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    // Wait for feature flags to load
    posthog.onFeatureFlags(() => {
      setVariant(posthog.getFeatureFlag(experimentKey) as string);
      setIsLoading(false);
    });
  }, [experimentKey, posthog]);
  
  return { variant, isLoading };
}

// Track experiment exposure
export const trackExperimentExposure = (
  experimentKey: string,
  variant: string
) => {
  posthog.capture('$experiment_exposure', {
    $experiment_key: experimentKey,
    $experiment_variant: variant,
  });
};

// Track conversion
export const trackExperimentConversion = (
  experimentKey: string,
  conversionEvent: string,
  metadata?: Record<string, unknown>
) => {
  posthog.capture(conversionEvent, {
    $experiment_key: experimentKey,
    ...metadata,
  });
};
```

### 13.2 Experiment Component

```typescript
// components/experiments/Experiment.tsx
'use client';

import { useExperiment, trackExperimentExposure } from '@/lib/experiments/posthog-experiments';
import { useEffect, ReactNode } from 'react';

interface ExperimentProps {
  name: string;
  variants: Record<string, ReactNode>;
  fallback?: ReactNode;
  trackExposure?: boolean;
}

export function Experiment({
  name,
  variants,
  fallback = null,
  trackExposure = true,
}: ExperimentProps) {
  const { variant, isLoading } = useExperiment(name);
  
  useEffect(() => {
    if (!isLoading && variant && trackExposure) {
      trackExperimentExposure(name, variant);
    }
  }, [name, variant, isLoading, trackExposure]);
  
  if (isLoading) {
    return fallback;
  }
  
  if (!variant || !variants[variant]) {
    return variants['control'] || fallback;
  }
  
  return <>{variants[variant]}</>;
}

// Usage:
// <Experiment
//   name="pricing-page-experiment"
//   variants={{
//     control: <OriginalPricing />,
//     variant_a: <NewPricingA />,
//     variant_b: <NewPricingB />,
//   }}
//   fallback={<OriginalPricing />}
// />
```

### 13.3 Server-Side Feature Flags

```typescript
// lib/experiments/server-flags.ts
import { PostHog } from 'posthog-node';
import { cookies } from 'next/headers';

const posthog = new PostHog(process.env.POSTHOG_API_KEY!, {
  host: process.env.NEXT_PUBLIC_POSTHOG_HOST,
});

// Get user ID from cookies or generate one
async function getDistinctId(): Promise<string> {
  const cookieStore = await cookies();
  let distinctId = cookieStore.get('ph_distinct_id')?.value;
  
  if (!distinctId) {
    distinctId = `anon-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    // Note: Can't set cookies in server components directly
  }
  
  return distinctId;
}

// Server-side feature flag check
export async function getFeatureFlag(
  flagKey: string,
  userId?: string
): Promise<boolean | string | undefined> {
  const distinctId = userId || await getDistinctId();
  
  const flag = await posthog.getFeatureFlag(flagKey, distinctId);
  
  return flag;
}

// Get all flags for user
export async function getAllFeatureFlags(userId?: string) {
  const distinctId = userId || await getDistinctId();
  
  return await posthog.getAllFlags(distinctId);
}

// Shutdown client
export async function shutdownPostHog() {
  await posthog.shutdown();
}
```

---

## 14. Privacy & GDPR Compliance

### 14.1 Privacy-First Analytics

```typescript
// lib/privacy/consent-manager.ts
'use client';

export type ConsentCategory = 'necessary' | 'analytics' | 'marketing' | 'preferences';

export interface ConsentState {
  necessary: boolean;  // Always true
  analytics: boolean;
  marketing: boolean;
  preferences: boolean;
  timestamp: string;
  version: string;
}

const CONSENT_KEY = 'privacy_consent';
const CONSENT_VERSION = '1.0';

// Get current consent state
export const getConsentState = (): ConsentState | null => {
  if (typeof window === 'undefined') return null;
  
  const stored = localStorage.getItem(CONSENT_KEY);
  if (!stored) return null;
  
  try {
    const consent = JSON.parse(stored) as ConsentState;
    // Check if consent version is current
    if (consent.version !== CONSENT_VERSION) {
      return null; // Re-ask for consent if version changed
    }
    return consent;
  } catch {
    return null;
  }
};

// Save consent state
export const setConsentState = (consent: Omit<ConsentState, 'timestamp' | 'version'>) => {
  const state: ConsentState = {
    ...consent,
    necessary: true, // Always required
    timestamp: new Date().toISOString(),
    version: CONSENT_VERSION,
  };
  
  localStorage.setItem(CONSENT_KEY, JSON.stringify(state));
  
  // Apply consent to analytics tools
  applyConsent(state);
  
  return state;
};

// Check if specific consent is given
export const hasConsent = (category: ConsentCategory): boolean => {
  const state = getConsentState();
  if (!state) return false;
  return state[category];
};

// Apply consent to all analytics tools
const applyConsent = (consent: ConsentState) => {
  // Google Analytics
  if (typeof window !== 'undefined' && window.gtag) {
    window.gtag('consent', 'update', {
      analytics_storage: consent.analytics ? 'granted' : 'denied',
      ad_storage: consent.marketing ? 'granted' : 'denied',
      functionality_storage: consent.preferences ? 'granted' : 'denied',
      personalization_storage: consent.preferences ? 'granted' : 'denied',
    });
  }
  
  // PostHog
  if (consent.analytics) {
    import('@/lib/analytics/posthog').then(({ posthog }) => {
      posthog.opt_in_capturing();
    });
  } else {
    import('@/lib/analytics/posthog').then(({ posthog }) => {
      posthog.opt_out_capturing();
    });
  }
  
  // Sentry
  if (!consent.analytics) {
    import('@sentry/nextjs').then((Sentry) => {
      Sentry.close();
    });
  }
};

// Accept all
export const acceptAllConsent = () => {
  return setConsentState({
    necessary: true,
    analytics: true,
    marketing: true,
    preferences: true,
  });
};

// Reject all (except necessary)
export const rejectAllConsent = () => {
  return setConsentState({
    necessary: true,
    analytics: false,
    marketing: false,
    preferences: false,
  });
};
```

### 14.2 Cookie Banner Component

```typescript
// components/privacy/CookieBanner.tsx
'use client';

import { useState, useEffect } from 'react';
import {
  getConsentState,
  setConsentState,
  acceptAllConsent,
  rejectAllConsent,
  ConsentState,
} from '@/lib/privacy/consent-manager';

export function CookieBanner() {
  const [showBanner, setShowBanner] = useState(false);
  const [showSettings, setShowSettings] = useState(false);
  const [consent, setConsent] = useState<Partial<ConsentState>>({
    necessary: true,
    analytics: false,
    marketing: false,
    preferences: false,
  });
  
  useEffect(() => {
    const existingConsent = getConsentState();
    if (!existingConsent) {
      setShowBanner(true);
    }
  }, []);
  
  const handleAcceptAll = () => {
    acceptAllConsent();
    setShowBanner(false);
  };
  
  const handleRejectAll = () => {
    rejectAllConsent();
    setShowBanner(false);
  };
  
  const handleSaveSettings = () => {
    setConsentState(consent as Omit<ConsentState, 'timestamp' | 'version'>);
    setShowBanner(false);
    setShowSettings(false);
  };
  
  if (!showBanner) return null;
  
  return (
    <div className="fixed bottom-0 left-0 right-0 bg-white border-t shadow-lg z-50 p-4">
      <div className="max-w-7xl mx-auto">
        {!showSettings ? (
          <div className="flex flex-col md:flex-row items-center justify-between gap-4">
            <div className="flex-1">
              <h3 className="font-semibold mb-1">🍪 Utilizziamo i cookie</h3>
              <p className="text-sm text-gray-600">
                Utilizziamo cookie per migliorare la tua esperienza, analizzare il traffico
                e personalizzare i contenuti.{' '}
                <a href="/privacy" className="underline">
                  Leggi la Privacy Policy
                </a>
              </p>
            </div>
            <div className="flex gap-2">
              <button
                onClick={() => setShowSettings(true)}
                className="px-4 py-2 text-sm border rounded hover:bg-gray-50"
              >
                Impostazioni
              </button>
              <button
                onClick={handleRejectAll}
                className="px-4 py-2 text-sm border rounded hover:bg-gray-50"
              >
                Rifiuta tutti
              </button>
              <button
                onClick={handleAcceptAll}
                className="px-4 py-2 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
              >
                Accetta tutti
              </button>
            </div>
          </div>
        ) : (
          <div className="space-y-4">
            <h3 className="font-semibold">Gestisci preferenze cookie</h3>
            
            <div className="space-y-3">
              <label className="flex items-center gap-3">
                <input
                  type="checkbox"
                  checked={true}
                  disabled
                  className="w-4 h-4"
                />
                <div>
                  <span className="font-medium">Necessari</span>
                  <p className="text-sm text-gray-600">
                    Essenziali per il funzionamento del sito
                  </p>
                </div>
              </label>
              
              <label className="flex items-center gap-3">
                <input
                  type="checkbox"
                  checked={consent.analytics}
                  onChange={(e) => setConsent({ ...consent, analytics: e.target.checked })}
                  className="w-4 h-4"
                />
                <div>
                  <span className="font-medium">Analytics</span>
                  <p className="text-sm text-gray-600">
                    Ci aiutano a capire come utilizzi il sito
                  </p>
                </div>
              </label>
              
              <label className="flex items-center gap-3">
                <input
                  type="checkbox"
                  checked={consent.marketing}
                  onChange={(e) => setConsent({ ...consent, marketing: e.target.checked })}
                  className="w-4 h-4"
                />
                <div>
                  <span className="font-medium">Marketing</span>
                  <p className="text-sm text-gray-600">
                    Per mostrarti annunci pertinenti
                  </p>
                </div>
              </label>
              
              <label className="flex items-center gap-3">
                <input
                  type="checkbox"
                  checked={consent.preferences}
                  onChange={(e) => setConsent({ ...consent, preferences: e.target.checked })}
                  className="w-4 h-4"
                />
                <div>
                  <span className="font-medium">Preferenze</span>
                  <p className="text-sm text-gray-600">
                    Ricordano le tue preferenze per visite future
                  </p>
                </div>
              </label>
            </div>
            
            <div className="flex gap-2 pt-2">
              <button
                onClick={() => setShowSettings(false)}
                className="px-4 py-2 text-sm border rounded hover:bg-gray-50"
              >
                Indietro
              </button>
              <button
                onClick={handleSaveSettings}
                className="px-4 py-2 text-sm bg-blue-600 text-white rounded hover:bg-blue-700"
              >
                Salva preferenze
              </button>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

### 14.3 Data Subject Request Handler

```typescript
// lib/privacy/dsr-handler.ts
import { logger } from '@/lib/logger';

export type DSRType = 'access' | 'delete' | 'export' | 'rectify' | 'restrict';

interface DSRRequest {
  type: DSRType;
  userId: string;
  email: string;
  details?: string;
  requestedAt: Date;
}

// Handle Data Subject Request
export const handleDSR = async (request: DSRRequest): Promise<void> => {
  logger.info({
    event: 'dsr_received',
    type: request.type,
    userId: request.userId,
  }, `DSR request received: ${request.type}`);
  
  switch (request.type) {
    case 'access':
      await handleAccessRequest(request);
      break;
    case 'delete':
      await handleDeleteRequest(request);
      break;
    case 'export':
      await handleExportRequest(request);
      break;
    case 'rectify':
      await handleRectifyRequest(request);
      break;
    case 'restrict':
      await handleRestrictRequest(request);
      break;
  }
};

async function handleAccessRequest(request: DSRRequest) {
  // Collect all user data from various sources
  // - Database records
  // - Analytics data
  // - Log entries
  // - Third-party services
  
  logger.info({ userId: request.userId }, 'Processing access request');
}

async function handleDeleteRequest(request: DSRRequest) {
  // Delete user data from:
  // 1. Database
  // 2. PostHog (via API)
  // 3. Sentry (via API)
  // 4. GA4 (via API)
  // 5. Logs
  
  logger.info({ userId: request.userId }, 'Processing delete request');
  
  // Example: Delete from PostHog
  await fetch('https://app.posthog.com/api/person/' + request.userId + '/', {
    method: 'DELETE',
    headers: {
      Authorization: `Bearer ${process.env.POSTHOG_PERSONAL_API_KEY}`,
    },
  });
}

async function handleExportRequest(request: DSRRequest) {
  // Export all user data in machine-readable format
  logger.info({ userId: request.userId }, 'Processing export request');
}

async function handleRectifyRequest(request: DSRRequest) {
  // Allow user to correct their data
  logger.info({ userId: request.userId }, 'Processing rectify request');
}

async function handleRestrictRequest(request: DSRRequest) {
  // Restrict processing of user data
  logger.info({ userId: request.userId }, 'Processing restrict request');
}
```



---

## 15. Costi e Confronto Strumenti

### 15.1 Tabella Comparativa Tools

```
┌──────────────────────────────────────────────────────────────────────────────────────────┐
│                           CONFRONTO STRUMENTI ANALYTICS & MONITORING                     │
├──────────────────────────────────────────────────────────────────────────────────────────┤
│ Categoria        │ Strumento       │ Free Tier          │ Pro Pricing     │ Note        │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ WEB ANALYTICS    │ Google          │ Illimitato         │ GA360: $150k+   │ Standard    │
│                  │ Analytics 4     │                    │ /anno           │ per tutti   │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Plausible       │ 30 giorni trial    │ €9/mese (10k pv)│ Privacy-    │
│                  │                 │                    │                 │ focused     │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Fathom          │ -                  │ $14/mese        │ Privacy     │
│                  │                 │                    │ (100k pv)       │             │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ PRODUCT          │ PostHog         │ 1M eventi/mese     │ Pay-per-use     │ Open-source │
│ ANALYTICS        │                 │ + session replay   │ dopo free tier  │ self-host   │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Mixpanel        │ 20M eventi/mese    │ $28/mese        │ Potente     │
│                  │                 │                    │ (100M eventi)   │ ma costoso  │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Amplitude       │ 10M eventi/mese    │ Custom pricing  │ Enterprise  │
│                  │                 │                    │                 │             │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ ERROR            │ Sentry          │ 5k errori/mese     │ $26/mese        │ Industry    │
│ TRACKING         │                 │ + session replay   │ (50k errori)    │ standard    │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Bugsnag         │ 7.5k eventi/mese   │ $59/mese        │ Mobile      │
│                  │                 │                    │                 │ focused     │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Rollbar         │ 5k eventi/mese     │ $19/mese        │ Buon free   │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ APM &            │ Datadog         │ 14 giorni trial    │ $15/host/mese   │ Full-stack  │
│ OBSERVABILITY    │                 │                    │ + usage         │ ma costoso  │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ New Relic       │ 100GB/mese         │ Pay-per-GB      │ Generoso    │
│                  │                 │                    │ dopo free       │ free tier   │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Grafana Cloud   │ 10k series/14d     │ $8/utente/mese  │ Open-source │
│                  │                 │ + logs + traces    │                 │ friendly    │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ SESSION          │ Sentry Replay   │ Incluso in Sentry  │ Incluso         │ Integrato   │
│ REPLAY           │                 │                    │                 │             │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ PostHog Replay  │ 5k sessioni/mese   │ Pay-per-use     │ Integrato   │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ FullStory       │ 1k sessioni/mese   │ Custom pricing  │ Enterprise  │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Hotjar          │ 35 sessioni/giorno │ €32/mese        │ Popolare    │
├──────────────────┼─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │                 │                    │                 │             │
│ LOG              │ Better Stack    │ 1GB/mese           │ $0.25/GB        │ Good UX     │
│ MANAGEMENT       │                 │                    │                 │             │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Loki + Grafana  │ Self-host: Free    │ Grafana Cloud   │ Open-source │
│                  │                 │                    │ $8/utente       │             │
│                  ├─────────────────┼────────────────────┼─────────────────┼─────────────┤
│                  │ Papertrail      │ 100MB/mese         │ $7/mese (1GB)   │ Semplice    │
└──────────────────┴─────────────────┴────────────────────┴─────────────────┴─────────────┘
```

### 15.2 Stack Raccomandati per Budget

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           STACK PER BUDGET                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  💰 BUDGET: €0 - Startup/Side Project                                       │
│  ═══════════════════════════════════════════                                │
│  • GA4 (free) - Web analytics                                               │
│  • PostHog free tier - Product analytics + session replay                   │
│  • Sentry free tier - Error tracking                                        │
│  • Pino + stdout - Logging                                                  │
│  • Vercel Analytics (free) - Basic performance                              │
│                                                                             │
│  Costo totale: €0/mese                                                      │
│  Limiti: 1M eventi PostHog, 5k errori Sentry                               │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  💰 BUDGET: €50-100/mese - Small Business                                   │
│  ═══════════════════════════════════════════                                │
│  • GA4 (free) - Web analytics                                               │
│  • PostHog Growth - Product analytics + replay                              │
│  • Sentry Team - Error tracking + performance                               │
│  • Better Stack - Log management                                            │
│  • Uptime monitoring (UptimeRobot)                                          │
│                                                                             │
│  Costo stimato: ~€70/mese                                                   │
│  Copre: ~500k users, full observability                                     │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  💰 BUDGET: €200-500/mese - Scale-up                                        │
│  ═══════════════════════════════════════════                                │
│  • GA4 + BigQuery Export - Advanced analytics                               │
│  • PostHog Enterprise - Full product suite                                  │
│  • Sentry Business - Full APM + profiling                                   │
│  • Grafana Cloud Pro - Metrics + logs + traces                              │
│  • OpenTelemetry - Instrumentation                                          │
│                                                                             │
│  Costo stimato: ~€350/mese                                                  │
│  Copre: ~2M users, enterprise features                                      │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  💰 BUDGET: €1000+/mese - Enterprise                                        │
│  ═══════════════════════════════════════════                                │
│  • GA360 o Snowplow - Enterprise analytics                                  │
│  • Amplitude/Mixpanel - Advanced product analytics                          │
│  • Datadog - Full-stack observability                                       │
│  • Custom dashboards + alerting                                             │
│  • 24/7 monitoring + incident management                                    │
│                                                                             │
│  Costo: Custom pricing                                                      │
│  Copre: Unlimited scale, SLAs, support                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 15.3 ROI dell'Analytics & Monitoring

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     ROI DELL'ANALYTICS & MONITORING                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📈 BENEFICI MISURABILI                                                     │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Metrica                    │ Prima  │ Dopo   │ Miglioramento        │   │
│  ├────────────────────────────┼────────┼────────┼──────────────────────┤   │
│  │ MTTR (Mean Time to Repair) │ 4h     │ 30min  │ -87%                 │   │
│  │ Conversion Rate            │ 2.5%   │ 3.5%   │ +40%                 │   │
│  │ Page Load Time             │ 4.2s   │ 1.8s   │ -57%                 │   │
│  │ Error Rate                 │ 5%     │ 0.5%   │ -90%                 │   │
│  │ User Satisfaction (NPS)    │ 30     │ 55     │ +83%                 │   │
│  │ Churn Rate                 │ 8%     │ 4%     │ -50%                 │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  💰 CALCOLO ROI ESEMPIO (E-commerce 100k visite/mese)                       │
│                                                                             │
│  Costo analytics stack: €100/mese = €1,200/anno                             │
│                                                                             │
│  Benefici:                                                                  │
│  • Conversion +40%: 2.5% → 3.5% su €50k revenue = +€20k/anno               │
│  • Meno downtime: 4h → 0.5h/mese = risparmio €5k/anno                      │
│  • Meno bug in prod: risparmio dev time = €10k/anno                        │
│                                                                             │
│  Totale benefici: €35k/anno                                                 │
│  ROI: (35k - 1.2k) / 1.2k = 2,817%                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Appendice: Quick Reference

### A.1 Checklist Analytics Setup

```markdown
## Analytics & Monitoring Checklist

### Setup Iniziale
- [ ] Google Analytics 4 configurato
- [ ] Consent management implementato
- [ ] PostHog o product analytics tool integrato
- [ ] Sentry configurato per error tracking
- [ ] Web Vitals monitoring attivo
- [ ] Logging strutturato con Pino

### Eventi da Tracciare
- [ ] Page views
- [ ] User authentication (login, logout, signup)
- [ ] Core conversions (purchase, lead, signup)
- [ ] Feature usage
- [ ] Errors e exceptions
- [ ] Performance metrics

### Privacy & Compliance
- [ ] Cookie banner implementato
- [ ] Privacy policy aggiornata
- [ ] Data retention configurata
- [ ] DSR (Data Subject Request) handler
- [ ] IP anonymization attiva

### Alerting
- [ ] Error rate alerts
- [ ] Performance degradation alerts
- [ ] Availability monitoring
- [ ] Business metrics alerts

### Documentation
- [ ] Event tracking plan documentato
- [ ] Dashboard condivise con team
- [ ] Runbook per incidenti
- [ ] Metriche KPI definite
```

### A.2 Event Naming Conventions

```typescript
// Event naming conventions
const eventNamingConventions = {
  // Use snake_case
  correct: 'user_signed_up',
  incorrect: 'userSignedUp',
  
  // Start with entity
  pattern: '{entity}_{action}_{modifier?}',
  
  // Examples
  examples: {
    user: [
      'user_signed_up',
      'user_logged_in',
      'user_profile_updated',
      'user_subscription_started',
    ],
    order: [
      'order_created',
      'order_completed',
      'order_cancelled',
      'order_refunded',
    ],
    product: [
      'product_viewed',
      'product_added_to_cart',
      'product_removed_from_cart',
      'product_searched',
    ],
    page: [
      'page_viewed',
      'page_scrolled',
      'page_time_spent',
    ],
    feature: [
      'feature_used',
      'feature_enabled',
      'feature_disabled',
    ],
  },
  
  // Reserved prefixes (don't use)
  reserved: ['$', 'mp_', 'ga_', 'fb_'],
  
  // Max length
  maxLength: 40,
};
```

### A.3 Comandi Utili

```bash
# Google Analytics Debug
# Aggiungi ?debug=1 all'URL o usa GA Debug Extension

# PostHog Debug Mode
# Nel browser console:
posthog.debug()

# Sentry Debug
# In sentry.client.config.ts: debug: true

# Check Web Vitals nel browser
# Chrome DevTools → Performance → Record
# Oppure: Lighthouse audit

# Prometheus queries utili
# Request rate:
rate(http_requests_total[5m])

# Error rate:
sum(rate(http_requests_total{status_code=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# P95 latency:
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# Test Pino logging
node -e "const pino = require('pino')(); pino.info({test: true}, 'Hello')"

# View Pino logs with pretty print
node app.js | pino-pretty

# Check metrics endpoint
curl http://localhost:3000/api/metrics

# OpenTelemetry debug
export OTEL_LOG_LEVEL=debug
```

### A.4 Troubleshooting Guide

```markdown
## Troubleshooting Comune

### GA4 non traccia eventi
1. Verifica che il Measurement ID sia corretto
2. Controlla che il consent sia stato dato
3. Usa GA4 DebugView per verificare
4. Controlla la console per errori JavaScript

### PostHog non mostra dati
1. Verifica API key e host
2. Controlla che opt_in_capturing() sia chiamato
3. Verifica che capture_pageview non sia false
4. Controlla Network tab per richieste a PostHog

### Sentry non cattura errori
1. Verifica DSN
2. Controlla che enabled: true in production
3. Verifica beforeSend non filtri tutto
4. Testa con Sentry.captureMessage('test')

### Web Vitals non misurati
1. Assicurati che la pagina sia completamente caricata
2. Interagisci con la pagina per INP
3. Usa Chrome DevTools Performance panel
4. Verifica che web-vitals sia importato correttamente

### Metrics endpoint non risponde
1. Verifica che prom-client sia installato
2. Controlla che il registry sia inizializzato
3. Verifica che l'endpoint sia raggiungibile
4. Controlla logs per errori

### Logs non visibili
1. Verifica LOG_LEVEL environment variable
2. Controlla che pino-pretty sia installato (dev)
3. Verifica che stdout non sia reindirizzato
4. Controlla transport configuration
```

### A.5 Metriche KPI Raccomandate

```typescript
// KPI metrics to track
const kpiMetrics = {
  // Technical KPIs
  technical: {
    availability: {
      name: 'Uptime',
      target: '99.9%',
      formula: '(total_time - downtime) / total_time * 100',
    },
    errorRate: {
      name: 'Error Rate',
      target: '<1%',
      formula: 'errors / total_requests * 100',
    },
    p95Latency: {
      name: 'P95 Response Time',
      target: '<500ms',
      formula: 'percentile(response_time, 95)',
    },
    lcp: {
      name: 'LCP (p75)',
      target: '<2.5s',
      formula: 'percentile(lcp_value, 75)',
    },
    inp: {
      name: 'INP (p75)',
      target: '<200ms',
      formula: 'percentile(inp_value, 75)',
    },
    cls: {
      name: 'CLS (p75)',
      target: '<0.1',
      formula: 'percentile(cls_value, 75)',
    },
  },
  
  // Business KPIs
  business: {
    conversionRate: {
      name: 'Conversion Rate',
      target: 'Industry dependent',
      formula: 'conversions / visitors * 100',
    },
    bounceRate: {
      name: 'Bounce Rate',
      target: '<50%',
      formula: 'single_page_sessions / total_sessions * 100',
    },
    avgSessionDuration: {
      name: 'Avg Session Duration',
      target: '>2min',
      formula: 'sum(session_duration) / count(sessions)',
    },
    dau: {
      name: 'Daily Active Users',
      target: 'Growth',
      formula: 'unique_users_per_day',
    },
    retention: {
      name: 'Day 7 Retention',
      target: '>20%',
      formula: 'returning_users_day7 / new_users_day0 * 100',
    },
  },
  
  // Product KPIs
  product: {
    featureAdoption: {
      name: 'Feature Adoption',
      target: '>30%',
      formula: 'users_using_feature / total_users * 100',
    },
    timeToValue: {
      name: 'Time to First Value',
      target: '<5min',
      formula: 'time_from_signup_to_first_action',
    },
    nps: {
      name: 'Net Promoter Score',
      target: '>50',
      formula: 'promoters% - detractors%',
    },
  },
};
```

### A.6 Risorse Utili

```markdown
## Documentazione Ufficiale
- [Google Analytics 4](https://developers.google.com/analytics)
- [PostHog Docs](https://posthog.com/docs)
- [Sentry for Next.js](https://docs.sentry.io/platforms/javascript/guides/nextjs/)
- [OpenTelemetry JS](https://opentelemetry.io/docs/instrumentation/js/)
- [Prometheus](https://prometheus.io/docs/)
- [Grafana](https://grafana.com/docs/)
- [Pino](https://getpino.io/)
- [Web Vitals](https://web.dev/articles/vitals)

## Next.js Specifico
- [Next.js Analytics Guide](https://nextjs.org/docs/pages/guides/analytics)
- [Next.js Third Party Libraries](https://nextjs.org/docs/app/guides/third-party-libraries)
- [Vercel Analytics](https://vercel.com/docs/analytics)

## Best Practices
- [Google Tag Manager Best Practices](https://developers.google.com/tag-platform/tag-manager/best-practices)
- [Observability Engineering (O'Reilly)](https://www.oreilly.com/library/view/observability-engineering/9781492076438/)
- [SRE Book - Monitoring](https://sre.google/sre-book/monitoring-distributed-systems/)

## Community
- [Analytics Subreddit](https://reddit.com/r/analytics)
- [PostHog Community](https://posthog.com/community)
- [Sentry Discord](https://discord.com/invite/sentry)
```

---

## Changelog

| Versione | Data | Modifiche |
|----------|------|-----------|
| 1.0.0 | Gennaio 2026 | Release iniziale |

---

**Fine del Catalogo Analytics & Monitoring v1.0**

*Questo catalogo è parte della serie di cataloghi per lo sviluppo di applicazioni Next.js enterprise. Per domande o contributi, aprire una issue nel repository.*
