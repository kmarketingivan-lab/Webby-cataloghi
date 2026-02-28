# CATALOGO-FEATURE-FLAGS

CATALOGO-FEATURE-FLAGS per Next.js 14 + TypeScript
§ FEATURE FLAG PATTERNS
Boolean Flags
typescript
// types/feature-flags.ts
export interface BooleanFlag {
  type: 'boolean';
  key: string;
  defaultValue: boolean;
  description: string;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
  environments: {
    [environment: string]: {
      enabled: boolean;
      lastModified: Date;
    };
  };
}

// esempi di utilizzo
export const FEATURE_FLAGS = {
  NEW_CHECKOUT_FLOW: {
    type: 'boolean' as const,
    key: 'new-checkout-flow',
    defaultValue: false,
    description: 'Abilita il nuovo flusso di checkout',
  },
  DARK_MODE: {
    type: 'boolean' as const,
    key: 'dark-mode',
    defaultValue: true,
    description: 'Abilita la modalità scura',
  },
} as const;
Percentage Rollout
typescript
// types/feature-flags.ts
export interface PercentageFlag {
  type: 'percentage';
  key: string;
  percentage: number; // 0-100
  stickinessProperty: string; // 'userId', 'sessionId', 'ip'
  description: string;
  overrides?: {
    userIds?: string[];
    userGroups?: string[];
  };
}

// utils/percentage-rollout.ts
export class PercentageRollout {
  static isEnabled(
    flag: PercentageFlag, 
    identifier: string, 
    overrideCheck?: boolean
  ): boolean {
    if (overrideCheck) return true;
    
    // Hash l'identificatore per ottenere un valore consistente
    const hash = this.hashString(identifier);
    const normalized = hash % 100;
    
    return normalized < flag.percentage;
  }
  
  private static hashString(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) - hash) + str.charCodeAt(i);
      hash |= 0;
    }
    return Math.abs(hash);
  }
}

// implementazione
export const PERCENTAGE_FLAGS = {
  BETA_FEATURES: {
    type: 'percentage' as const,
    key: 'beta-features',
    percentage: 10,
    stickinessProperty: 'userId',
    description: 'Rollout graduale delle feature beta',
  },
};
User Segment Targeting
typescript
// types/feature-flags.ts
export type UserSegment = 
  | 'beta-testers'
  | 'premium-users'
  | 'new-users'
  | 'vip-users'
  | 'geo:us'
  | 'geo:eu';

export interface SegmentFlag {
  type: 'segment';
  key: string;
  segments: UserSegment[];
  description: string;
  rules: SegmentRule[];
}

export interface SegmentRule {
  property: string;
  operator: 'equals' | 'contains' | 'startsWith' | 'in' | 'greaterThan';
  value: any;
}

// utils/user-segments.ts
export interface UserContext {
  userId?: string;
  email?: string;
  groups: string[];
  country?: string;
  subscriptionTier?: 'free' | 'premium' | 'enterprise';
  signupDate?: Date;
  customAttributes?: Record<string, any>;
}

export class SegmentEvaluator {
  static isInSegment(user: UserContext, segment: UserSegment): boolean {
    const segmentHandlers: Record<string, (user: UserContext) => boolean> = {
      'beta-testers': (u) => u.groups.includes('beta-testers'),
      'premium-users': (u) => u.subscriptionTier === 'premium' || u.subscriptionTier === 'enterprise',
      'new-users': (u) => {
        if (!u.signupDate) return false;
        const daysSinceSignup = (Date.now() - u.signupDate.getTime()) / (1000 * 60 * 60 * 24);
        return daysSinceSignup < 30;
      },
      'geo:us': (u) => u.country === 'US',
      'geo:eu': (u) => ['IT', 'FR', 'DE', 'ES', 'UK'].includes(u.country || ''),
    };
    
    return segmentHandlers[segment]?.(user) || false;
  }
  
  static evaluateRules(user: UserContext, rules: SegmentRule[]): boolean {
    return rules.every(rule => {
      const userValue = user.customAttributes?.[rule.property] || user[rule.property as keyof UserContext];
      
      switch (rule.operator) {
        case 'equals':
          return userValue === rule.value;
        case 'contains':
          return typeof userValue === 'string' && userValue.includes(rule.value);
        case 'startsWith':
          return typeof userValue === 'string' && userValue.startsWith(rule.value);
        case 'in':
          return Array.isArray(rule.value) && rule.value.includes(userValue);
        case 'greaterThan':
          return typeof userValue === 'number' && userValue > rule.value;
        default:
          return false;
      }
    });
  }
}
A/B Testing Flags
typescript
// types/feature-flags.ts
export interface ABTestFlag {
  type: 'ab-test';
  key: string;
  variants: {
    name: string;
    weight: number; // 0-100, somma totale = 100
    payload?: any;
  }[];
  tracking: {
    eventName: string;
    goals: string[];
    successMetrics: string[];
  };
  description: string;
}

// utils/ab-testing.ts
export class ABTestAssigner {
  static assignVariant(
    flag: ABTestFlag, 
    identifier: string
  ): { variant: string; payload?: any } {
    const hash = this.hashString(identifier);
    const randomValue = hash % 100;
    
    let cumulativeWeight = 0;
    for (const variant of flag.variants) {
      cumulativeWeight += variant.weight;
      if (randomValue < cumulativeWeight) {
        return {
          variant: variant.name,
          payload: variant.payload,
        };
      }
    }
    
    // Fallback alla prima variante
    return {
      variant: flag.variants[0].name,
      payload: flag.variants[0].payload,
    };
  }
  
  static trackExposure(
    flagKey: string,
    variant: string,
    userContext: UserContext
  ) {
    // Invia evento ad analytics
    if (typeof window !== 'undefined' && (window as any).gtag) {
      (window as any).gtag('event', 'feature_flag_exposure', {
        flag_key: flagKey,
        variant,
        user_id: userContext.userId,
      });
    }
  }
  
  private static hashString(str: string): number {
    // Implementazione hash consistente
    let hash = 5381;
    for (let i = 0; i < str.length; i++) {
      hash = (hash * 33) ^ str.charCodeAt(i);
    }
    return hash >>> 0;
  }
}

// esempio flag A/B test
export const AB_TEST_FLAGS = {
  BUTTON_COLOR_TEST: {
    type: 'ab-test' as const,
    key: 'button-color-test',
    variants: [
      { name: 'control', weight: 33, payload: { color: 'blue' } },
      { name: 'variant_a', weight: 33, payload: { color: 'green' } },
      { name: 'variant_b', weight: 34, payload: { color: 'red' } },
    ],
    tracking: {
      eventName: 'button_click',
      goals: ['purchase_completed', 'signup_completed'],
      successMetrics: ['conversion_rate', 'click_through_rate'],
    },
    description: 'Test sul colore del bottone principale',
  },
};
Kill Switches
typescript
// types/feature-flags.ts
export interface KillSwitch {
  type: 'kill-switch';
  key: string;
  status: 'active' | 'inactive';
  affectedServices: string[];
  severity: 'low' | 'medium' | 'high' | 'critical';
  activationCondition: {
    type: 'error-rate' | 'latency' | 'manual';
    threshold?: number;
  };
  fallbackBehavior: 'disable-feature' | 'use-legacy' | 'show-maintenance';
  description: string;
}

// utils/kill-switches.ts
export class KillSwitchManager {
  private static switches: Map<string, KillSwitch> = new Map();
  private static metrics: Map<string, ServiceMetrics> = new Map();
  
  static registerSwitch(killSwitch: KillSwitch) {
    this.switches.set(killSwitch.key, killSwitch);
  }
  
  static updateMetrics(service: string, metrics: ServiceMetrics) {
    this.metrics.set(service, metrics);
    
    // Controlla se qualche kill switch deve essere attivato
    this.evaluateSwitches();
  }
  
  static isKillSwitchActive(key: string): boolean {
    const killSwitch = this.switches.get(key);
    if (!killSwitch || killSwitch.status !== 'active') return false;
    
    if (killSwitch.activationCondition.type === 'manual') {
      return true;
    }
    
    const serviceMetrics = this.metrics.get(killSwitch.affectedServices[0]);
    if (!serviceMetrics) return false;
    
    switch (killSwitch.activationCondition.type) {
      case 'error-rate':
        return serviceMetrics.errorRate >= (killSwitch.activationCondition.threshold || 5);
      case 'latency':
        return serviceMetrics.p95Latency >= (killSwitch.activationCondition.threshold || 1000);
      default:
        return false;
    }
  }
  
  static getFallbackBehavior(key: string) {
    return this.switches.get(key)?.fallbackBehavior || 'disable-feature';
  }
  
  private static evaluateSwitches() {
    for (const [key, killSwitch] of this.switches) {
      if (killSwitch.activationCondition.type !== 'manual') {
        const shouldActivate = this.isKillSwitchActive(key);
        // Qui potresti attivare/disattivare automaticamente
        // o notificare un operatore
      }
    }
  }
}

interface ServiceMetrics {
  errorRate: number;
  p95Latency: number;
  requestCount: number;
  timestamp: Date;
}

// esempio kill switch
export const KILL_SWITCHES = {
  PAYMENT_SERVICE: {
    type: 'kill-switch' as const,
    key: 'payment-service-kill-switch',
    status: 'active',
    affectedServices: ['payment-service', 'checkout-service'],
    severity: 'critical',
    activationCondition: {
      type: 'error-rate',
      threshold: 10, // 10% error rate
    },
    fallbackBehavior: 'use-legacy',
    description: 'Disabilita il nuovo servizio di pagamento in caso di errori',
  },
};
§ LAUNCHDARKLY INTEGRATION
SDK Setup per Next.js
typescript
// lib/launchdarkly/server.ts
import { LDClient, LDContext, init } from 'launchdarkly-node-server-sdk';

let client: LDClient | null = null;

export async function getServerSideLDClient(): Promise<LDClient> {
  if (client && client.initialized()) {
    return client;
  }
  
  const sdkKey = process.env.LAUNCHDARKLY_SDK_KEY;
  if (!sdkKey) {
    throw new Error('LAUNCHDARKLY_SDK_KEY non configurato');
  }
  
  client = init(sdkKey, {
    application: {
      id: 'nextjs-app',
      version: process.env.npm_package_version,
    },
    streamUri: process.env.LAUNCHDARKLY_STREAM_URI,
    eventsUri: process.env.LAUNCHDARKLY_EVENTS_URI,
    diagnosticOptOut: process.env.NODE_ENV === 'development',
  });
  
  await client.waitForInitialization();
  return client;
}

export function createLDContext(user?: {
  id?: string;
  email?: string;
  name?: string;
  custom?: Record<string, any>;
}): LDContext {
  const anonymousContext: LDContext = {
    kind: 'user',
    key: 'anonymous',
    anonymous: true,
  };
  
  if (!user?.id) {
    return anonymousContext;
  }
  
  return {
    kind: 'user',
    key: user.id,
    email: user.email,
    name: user.name,
    custom: user.custom,
    anonymous: false,
  };
}

// lib/launchdarkly/client.ts
import { LDClient } from 'launchdarkly-js-client-sdk';

let client: LDClient | null = null;

export async function initializeLDClient(
  userContext: LDContext
): Promise<LDClient> {
  if (client && client.initialized()) {
    return client;
  }
  
  const clientSideID = process.env.NEXT_PUBLIC_LAUNCHDARKLY_CLIENT_ID;
  if (!clientSideID) {
    throw new Error('NEXT_PUBLIC_LAUNCHDARKLY_CLIENT_ID non configurato');
  }
  
  const { initialize } = await import('launchdarkly-js-client-sdk');
  
  client = initialize(clientSideID, userContext, {
    bootstrap: 'localStorage',
    diagnosticOptOut: process.env.NODE_ENV === 'development',
    streaming: true,
  });
  
  return new Promise((resolve) => {
    client!.on('ready', () => resolve(client!));
  });
}

export function getLDClient(): LDClient | null {
  return client;
}
Server-Side Flags
typescript
// lib/launchdarkly/server-flags.ts
import { getServerSideLDClient, createLDContext } from './server';

export async function getServerSideFlag(
  key: string,
  defaultValue: boolean,
  user?: any
): Promise<boolean> {
  try {
    const client = await getServerSideLDClient();
    const context = createLDContext(user);
    
    return await client.variation(key, context, defaultValue);
  } catch (error) {
    console.error(`Errore nel recupero flag ${key}:`, error);
    return defaultValue;
  }
}

export async function getAllServerSideFlags(
  user?: any
): Promise<Record<string, boolean>> {
  try {
    const client = await getServerSideLDClient();
    const context = createLDContext(user);
    
    // Recupera tutte le flag della tua applicazione
    // Dovresti mantenere una lista delle flag conosciute
    const knownFlags = [
      'new-checkout-flow',
      'dark-mode-enabled',
      'beta-features',
      // ... altre flag
    ];
    
    const flags: Record<string, boolean> = {};
    
    for (const key of knownFlags) {
      flags[key] = await client.variation(key, context, false);
    }
    
    return flags;
  } catch (error) {
    console.error('Errore nel recupero flag:', error);
    return {};
  }
}

// app/api/checkout/route.ts - Esempio di utilizzo
import { NextRequest, NextResponse } from 'next/server';
import { getServerSideFlag } from '@/lib/launchdarkly/server-flags';

export async function POST(request: NextRequest) {
  try {
    const user = await getCurrentUser(request); // tua funzione di autenticazione
    
    // Controlla se il nuovo checkout è abilitato per questo utente
    const newCheckoutEnabled = await getServerSideFlag(
      'new-checkout-flow',
      false,
      user
    );
    
    if (newCheckoutEnabled) {
      // Usa il nuovo flusso
      return await newCheckoutFlow(request);
    } else {
      // Usa il vecchio flusso
      return await legacyCheckoutFlow(request);
    }
  } catch (error) {
    return NextResponse.json(
      { error: 'Checkout failed' },
      { status: 500 }
    );
  }
}
Client-Side Flags
typescript
// hooks/useLaunchDarkly.ts
import { useEffect, useState, useCallback } from 'react';
import { initializeLDClient, getLDClient } from '@/lib/launchdarkly/client';
import type { LDClient, LDContext } from 'launchdarkly-js-client-sdk';

export function useLaunchDarkly(userContext?: LDContext) {
  const [client, setClient] = useState<LDClient | null>(null);
  const [flags, setFlags] = useState<Record<string, any>>({});
  const [isInitialized, setIsInitialized] = useState(false);
  
  useEffect(() => {
    async function init() {
      const context = userContext || {
        kind: 'user',
        key: 'anonymous',
        anonymous: true,
      };
      
      const ldClient = await initializeLDClient(context);
      setClient(ldClient);
      setIsInitialized(true);
      
      // Imposta i valori iniziali delle flag
      const initialFlags = ldClient.allFlags();
      setFlags(initialFlags);
      
      // Ascolta cambiamenti in tempo reale
      ldClient.on('change', (changedFlags: Record<string, any>) => {
        setFlags((prevFlags) => ({
          ...prevFlags,
          ...changedFlags,
        }));
      });
    }
    
    if (!client) {
      init();
    }
    
    return () => {
      // Cleanup
      if (client) {
        client.close();
      }
    };
  }, [userContext]);
  
  const identify = useCallback(async (newContext: LDContext) => {
    if (client) {
      await client.identify(newContext);
    }
  }, [client]);
  
  const track = useCallback((eventName: string, data?: any) => {
    if (client) {
      client.track(eventName, data);
    }
  }, [client]);
  
  return {
    client,
    flags,
    isInitialized,
    identify,
    track,
  };
}

// hooks/useFeatureFlag.ts - Versione specifica per LaunchDarkly
import { useLaunchDarkly } from './useLaunchDarkly';
import { useCallback, useEffect, useState } from 'react';

export function useFeatureFlag(key: string, defaultValue: any = false) {
  const { client, flags, isInitialized } = useLaunchDarkly();
  const [value, setValue] = useState(defaultValue);
  
  useEffect(() => {
    if (isInitialized && client) {
      // Imposta il valore iniziale
      const flagValue = flags[key] !== undefined ? flags[key] : defaultValue;
      setValue(flagValue);
      
      // Ascolta cambiamenti per questa specifica flag
      const onChange = () => {
        const newValue = client.variation(key, defaultValue);
        setValue(newValue);
      };
      
      client.on(`change:${key}`, onChange);
      
      return () => {
        client.off(`change:${key}`, onChange);
      };
    }
  }, [key, defaultValue, client, flags, isInitialized]);
  
  const identify = useCallback((userContext: LDContext) => {
    if (client) {
      client.identify(userContext);
    }
  }, [client]);
  
  return {
    value,
    identify,
    isReady: isInitialized,
  };
}

// components/FeatureGate.tsx
'use client';

import { ReactNode } from 'react';
import { useFeatureFlag } from '@/hooks/useFeatureFlag';

interface FeatureGateProps {
  flag: string;
  defaultValue?: boolean;
  children: ReactNode;
  fallback?: ReactNode;
  loadingComponent?: ReactNode;
}

export function FeatureGate({
  flag,
  defaultValue = false,
  children,
  fallback = null,
  loadingComponent = null,
}: FeatureGateProps) {
  const { value, isReady } = useFeatureFlag(flag, defaultValue);
  
  if (!isReady) {
    return <>{loadingComponent}</>;
  }
  
  return <>{value ? children : fallback}</>;
}
Real-Time Updates
typescript
// lib/launchdarkly/realtime.ts
import { getLDClient } from './client';

export function subscribeToFlagChanges(
  flagKey: string,
  callback: (newValue: any, oldValue: any) => void
): () => void {
  const client = getLDClient();
  
  if (!client) {
    console.warn('LaunchDarkly client non inizializzato');
    return () => {};
  }
  
  const handler = () => {
    const newValue = client.variation(flagKey, false);
    const oldValue = client.variation(flagKey, false, { omitEvaluation: true });
    callback(newValue, oldValue);
  };
  
  client.on(`change:${flagKey}`, handler);
  
  // Restituisci funzione di cleanup
  return () => {
    client.off(`change:${flagKey}`, handler);
  };
}

export function subscribeToAllFlags(
  callback: (flags: Record<string, any>) => void
): () => void {
  const client = getLDClient();
  
  if (!client) {
    console.warn('LaunchDarkly client non inizializzato');
    return () => {};
  }
  
  const handler = (changedFlags: Record<string, any>) => {
    const allFlags = client.allFlags();
    callback(allFlags);
  };
  
  client.on('change', handler);
  
  return () => {
    client.off('change', handler);
  };
}

// hooks/useRealtimeFlags.ts
import { useEffect, useState } from 'react';
import { 
  subscribeToFlagChanges, 
  subscribeToAllFlags 
} from '@/lib/launchdarkly/realtime';

export function useRealtimeFlag<T = any>(
  flagKey: string,
  defaultValue: T
): T {
  const [value, setValue] = useState<T>(defaultValue);
  
  useEffect(() => {
    const unsubscribe = subscribeToFlagChanges(flagKey, (newValue) => {
      setValue(newValue);
    });
    
    return unsubscribe;
  }, [flagKey]);
  
  return value;
}

export function useRealtimeFlags() {
  const [flags, setFlags] = useState<Record<string, any>>({});
  
  useEffect(() => {
    const unsubscribe = subscribeToAllFlags((newFlags) => {
      setFlags(newFlags);
    });
    
    return unsubscribe;
  }, []);
  
  return flags;
}
Analytics Integration
typescript
// lib/launchdarkly/analytics.ts
import { getLDClient } from './client';

export interface GoalMetric {
  name: string;
  value: number;
  context?: Record<string, any>;
}

export class LaunchDarklyAnalytics {
  static trackGoal(goalName: string, metric: GoalMetric) {
    const client = getLDClient();
    
    if (!client) {
      console.warn('Impossibile tracciare goal: client non inizializzato');
      return;
    }
    
    // Invia metriche custom a LaunchDarkly
    client.track(goalName, metric.context, metric.value);
    
    // Integrazione con Google Analytics 4 (opzionale)
    if (typeof window !== 'undefined' && (window as any).gtag) {
      (window as any).gtag('event', goalName, {
        value: metric.value,
        ...metric.context,
      });
    }
    
    // Integrazione con Mixpanel (opzionale)
    if (typeof window !== 'undefined' && (window as any).mixpanel) {
      (window as any).mixpanel.track(goalName, {
        value: metric.value,
        ...metric.context,
      });
    }
  }
  
  static trackFeatureExposure(
    flagKey: string,
    variant: string,
    userContext: any
  ) {
    const client = getLDClient();
    
    if (!client) return;
    
    // LaunchDarkly traccia automaticamente le exposure
    // Questa funzione è per tracking aggiuntivo
    client.track(`feature_exposure_${flagKey}`, userContext, 1);
    
    // Custom analytics
    if (typeof window !== 'undefined' && (window as any).dataLayer) {
      (window as any).dataLayer.push({
        event: 'feature_exposure',
        flag_key: flagKey,
        variant: variant,
        user_id: userContext.userId,
        timestamp: new Date().toISOString(),
      });
    }
  }
  
  static trackABTestConversion(
    testKey: string,
    variant: string,
    conversionType: string,
    value?: number
  ) {
    const client = getLDClient();
    
    if (!client) return;
    
    client.track(`${testKey}_conversion`, { variant, conversionType }, value);
  }
}

// middleware/analytics.ts - Esempio di middleware per tracciare feature flag usage
import { NextRequest, NextResponse } from 'next/server';
import { getServerSideLDClient } from '@/lib/launchdarkly/server';

export async function analyticsMiddleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Estrai informazioni utente dal request
  const userContext = extractUserContext(request);
  
  // Traccia l'accesso alla pagina con le flag attive
  if (userContext.userId) {
    try {
      const client = await getServerSideLDClient();
      const flags = await client.allFlagsState(userContext);
      
      // Invia dati di analisi
      trackPageView(request.url, userContext, flags);
    } catch (error) {
      console.error('Errore nel tracciamento analytics:', error);
    }
  }
  
  return response;
}
§ POSTHOG FEATURE FLAGS
Setup per Next.js
typescript
// lib/posthog/client.ts
import posthog from 'posthog-js';
import { PostHogProvider } from 'posthog-js/react';

export function initPostHog() {
  if (typeof window !== 'undefined') {
    const apiKey = process.env.NEXT_PUBLIC_POSTHOG_KEY;
    const apiHost = process.env.NEXT_PUBLIC_POSTHOG_HOST || 'https://app.posthog.com';
    
    if (!apiKey) {
      console.warn('PostHog API key non configurata');
      return;
    }
    
    posthog.init(apiKey, {
      api_host: apiHost,
      person_profiles: 'always', // o 'identified_only'
      capture_pageview: false, // Disabilita auto-capture per Next.js
      capture_pageleave: true,
      disable_session_recording: process.env.NODE_ENV === 'development',
      loaded: (ph) => {
        if (process.env.NODE_ENV === 'development') {
          ph.debug();
        }
      },
    });
  }
}

export function PostHogClientProvider({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  return (
    <PostHogProvider client={posthog}>
      {children}
    </PostHogProvider>
  );
}

// app/providers.tsx
'use client';

import { PostHogClientProvider } from '@/lib/posthog/client';
import { useEffect } from 'react';

export function Providers({ children }: { children: React.ReactNode }) {
  useEffect(() => {
    // Inizializza PostHog solo lato client
    import('@/lib/posthog/client').then(({ initPostHog }) => {
      initPostHog();
    });
  }, []);
  
  return (
    <PostHogClientProvider>
      {children}
    </PostHogClientProvider>
  );
}

// lib/posthog/server.ts
import { PostHog } from 'posthog-node';

let posthogServer: PostHog | null = null;

export function getPostHogServer(): PostHog {
  if (posthogServer) {
    return posthogServer;
  }
  
  const apiKey = process.env.POSTHOG_API_KEY;
  const host = process.env.POSTHOG_HOST || 'https://app.posthog.com';
  
  if (!apiKey) {
    throw new Error('POSTHOG_API_KEY non configurato');
  }
  
  posthogServer = new PostHog(apiKey, {
    host,
    flushAt: 1, // Invia immediatamente
    flushInterval: 0,
  });
  
  return posthogServer;
}

export async function shutdownPostHog() {
  if (posthogServer) {
    await posthogServer.shutdown();
    posthogServer = null;
  }
}
Multivariate Flags
typescript
// types/posthog-flags.ts
export interface MultivariateFlag<T = any> {
  key: string;
  variants: Array<{
    name: string;
    rollout_percentage: number;
    payload?: T;
  }>;
  filters: {
    groups?: Array<{
      rollout_percentage: number;
      properties: Array<{
        key: string;
        value: any;
        operator?: string;
        type?: string;
      }>;
    }>;
  };
}

// hooks/usePostHogFlag.ts
import { usePostHog } from 'posthog-js/react';
import { useEffect, useState } from 'react';

export function usePostHogFlag<T = any>(
  flagKey: string,
  defaultValue?: T,
  options?: {
    personProperties?: Record<string, any>;
    groupProperties?: Record<string, any>;
    sendEvent?: boolean;
  }
): { value: T; loading: boolean } {
  const posthog = usePostHog();
  const [value, setValue] = useState<T>(defaultValue as T);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    if (!posthog) {
      setLoading(false);
      return;
    }
    
    const fetchFlag = async () => {
      try {
        const flagValue = await posthog.getFeatureFlag(
          flagKey,
          options?.personProperties || {},
          options?.groupProperties || {},
          options?.sendEvent
        );
        
        // Se il flag non esiste o è undefined, usa defaultValue
        setValue(flagValue !== undefined ? flagValue : defaultValue as T);
      } catch (error) {
        console.error(`Errore nel recupero flag ${flagKey}:`, error);
        setValue(defaultValue as T);
      } finally {
        setLoading(false);
      }
    };
    
    fetchFlag();
    
    // Sottoscrizione a cambiamenti in tempo reale
    const unsubscribe = posthog.onFeatureFlags(() => {
      fetchFlag();
    });
    
    return () => {
      unsubscribe();
    };
  }, [flagKey, posthog, defaultValue, options]);
  
  return { value, loading };
}

// hooks/useMultivariateFlag.ts
import { usePostHog } from 'posthog-js/react';

export function useMultivariateFlag<T = any>(
  flagKey: string,
  variants: Record<string, T>,
  defaultVariant: string
): { variant: string; payload: T; loading: boolean } {
  const posthog = usePostHog();
  const [result, setResult] = useState({
    variant: defaultVariant,
    payload: variants[defaultVariant],
    loading: true,
  });
  
  useEffect(() => {
    if (!posthog) {
      setResult(prev => ({ ...prev, loading: false }));
      return;
    }
    
    const evaluateFlag = async () => {
      try {
        const activeVariant = await posthog.getFeatureFlag(flagKey);
        
        if (activeVariant && activeVariant in variants) {
          setResult({
            variant: activeVariant,
            payload: variants[activeVariant],
            loading: false,
          });
        } else {
          setResult({
            variant: defaultVariant,
            payload: variants[defaultVariant],
            loading: false,
          });
        }
      } catch (error) {
        console.error(`Errore valutazione multivariate flag ${flagKey}:`, error);
        setResult({
          variant: defaultVariant,
          payload: variants[defaultVariant],
          loading: false,
        });
      }
    };
    
    evaluateFlag();
    
    // Ascolta cambiamenti
    const unsubscribe = posthog.onFeatureFlags(() => {
      evaluateFlag();
    });
    
    return unsubscribe;
  }, [flagKey, posthog, variants, defaultVariant]);
  
  return result;
}

// esempio di utilizzo
export const MULTIVARIATE_FLAGS = {
  PRICING_TABLE_TEST: {
    key: 'pricing-table-variant',
    variants: {
      control: { layout: 'vertical', highlight: 'pro' },
      variant_a: { layout: 'horizontal', highlight: 'enterprise' },
      variant_b: { layout: 'grid', highlight: 'basic' },
    },
    defaultVariant: 'control',
  },
};
Rollout Percentages
typescript
// utils/posthog-rollout.ts
export class PostHogRollout {
  static async getRolloutPercentage(
    flagKey: string,
    distinctId: string,
    personProperties?: Record<string, any>
  ): Promise<number> {
    // PostHog gestisce internamente le percentuali
    // Questa funzione è per debug e monitoring
    
    const posthog = await import('posthog-js');
    const flagPayload = await posthog.getFeatureFlagPayload(flagKey);
    
    if (!flagPayload) {
      return 0;
    }
    
    // Estrai informazioni sul rollout dalla configurazione del flag
    // Nota: questa è una semplificazione
    return this.calculateUserPercentage(distinctId, flagKey);
  }
  
  private static calculateUserPercentage(
    distinctId: string,
    flagKey: string
  ): number {
    // Hash deterministico per l'utente e la flag
    const hash = this.hashCode(`${distinctId}-${flagKey}`);
    return (hash % 10000) / 100; // Ritorna 0-99.99
  }
  
  private static hashCode(str: string): number {
    let hash = 0;
    for (let i = 0; i < str.length; i++) {
      const char = str.charCodeAt(i);
      hash = ((hash << 5) - hash) + char;
      hash = hash & hash; // Convert to 32bit integer
    }
    return Math.abs(hash);
  }
  
  static async isUserInRollout(
    flagKey: string,
    distinctId: string,
    rolloutPercentage: number
  ): Promise<boolean> {
    const percentage = await this.getRolloutPercentage(flagKey, distinctId);
    return percentage < rolloutPercentage;
  }
}

// hooks/useRolloutFlag.ts
import { usePostHog } from 'posthog-js/react';

interface RolloutFlagOptions {
  distinctId?: string;
  personProperties?: Record<string, any>;
  groups?: Record<string, any>;
  onlyEvaluateLocally?: boolean;
  sendEvent?: boolean;
}

export function useRolloutFlag(
  flagKey: string,
  rolloutPercentage: number,
  options?: RolloutFlagOptions
): { enabled: boolean; loading: boolean } {
  const posthog = usePostHog();
  const [enabled, setEnabled] = useState(false);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    if (!posthog) {
      setLoading(false);
      return;
    }
    
    const evaluateFlag = async () => {
      try {
        const distinctId = options?.distinctId || posthog.get_distinct_id();
        
        // Usa PostHog per valutare la flag
        const isEnabled = await posthog.isFeatureEnabled(
          flagKey,
          distinctId,
          {
            personProperties: options?.personProperties,
            groups: options?.groups,
            onlyEvaluateLocally: options?.onlyEvaluateLocally,
            sendEvent: options?.sendEvent,
          }
        );
        
        setEnabled(isEnabled || false);
      } catch (error) {
        console.error(`Errore valutazione rollout flag ${flagKey}:`, error);
        setEnabled(false);
      } finally {
        setLoading(false);
      }
    };
    
    evaluateFlag();
    
    // Re-evalua quando cambiano le proprietà
    const unsubscribe = posthog.onFeatureFlags(() => {
      evaluateFlag();
    });
    
    return unsubscribe;
  }, [flagKey, posthog, rolloutPercentage, options]);
  
  return { enabled, loading };
}
User Properties Targeting
typescript
// lib/posthog/targeting.ts
import posthog from 'posthog-js';

export interface UserTargetingRule {
  property: string;
  operator: 'equals' | 'not_equals' | 'contains' | 'not_contains' | 'regex';
  value: any;
  type?: 'person' | 'group' | 'event';
}

export class PostHogTargeting {
  static async evaluateUser(
    distinctId: string,

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-04-FEATURE-FLAGS
Prompt ID: 4 / 19
Parte: 2
Exported: 2026-02-06T16:48:43.219Z
Characters: 1181
════════════════════════════════════════════════════════════

' : 'POST';
      
      const response = await fetch(url, {
        method,
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(values),
      });
      
      if (response.ok) {
        onSuccess();
      }
    } catch (error) {
      console.error('Errore nel salvataggio flag:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <Modal
      title={editingFlag ? 'Modifica Flag' : 'Crea Nuova Flag'}
      visible={visible}
      onCancel={onCancel}
      width={800}
      footer={null}
      destroyOnClose
    >
      <Form
        form={form}
        layout="vertical"
        onFinish={handleSubmit}
        initialValues={{
          type: 'BOOLEAN',
          status: 'DRAFT',
          defaultValue: false,
          rolloutPercentage: 0,
          isExperiment: false,
        }}
      >
        <Form.Item
          name="key"
          label="Chiave"
          rules={[
            { required: true, message: 'Inserisci una chiave' },
            { pattern: /^[a-z0-9-]+$/, message: 'Solo lettere minuscole, numeri e trattini' },
          ]}
        >
          <Input placeholder="es. new-checkout-flow

## § ADVANCED PATTERNS: FEATURE FLAGS

### Server Actions con Validazione

```typescript
// app/actions/feature-flags.ts
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


### FEATURE FLAGS - Utility Helper #508

```typescript
// lib/utils/feature-flags-helper-508.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #740

```typescript
// lib/utils/feature-flags-helper-740.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #423

```typescript
// lib/utils/feature-flags-helper-423.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #233

```typescript
// lib/utils/feature-flags-helper-233.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #108

```typescript
// lib/utils/feature-flags-helper-108.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #79

```typescript
// lib/utils/feature-flags-helper-79.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #511

```typescript
// lib/utils/feature-flags-helper-511.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #454

```typescript
// lib/utils/feature-flags-helper-454.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #527

```typescript
// lib/utils/feature-flags-helper-527.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #679

```typescript
// lib/utils/feature-flags-helper-679.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #284

```typescript
// lib/utils/feature-flags-helper-284.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #16

```typescript
// lib/utils/feature-flags-helper-16.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #404

```typescript
// lib/utils/feature-flags-helper-404.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #923

```typescript
// lib/utils/feature-flags-helper-923.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #469

```typescript
// lib/utils/feature-flags-helper-469.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #201

```typescript
// lib/utils/feature-flags-helper-201.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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

  getConfig(): Readonly<FEATUREFLAGSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FEATURE FLAGS - Utility Helper #728

```typescript
// lib/utils/feature-flags-helper-728.ts
import { z } from "zod";

interface FEATUREFLAGSConfig {
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

export class FEATUREFLAGSProcessor<TInput, TOutput> {
  private config: FEATUREFLAGSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FEATUREFLAGSConfig> = {}) {
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
          timestamp: 