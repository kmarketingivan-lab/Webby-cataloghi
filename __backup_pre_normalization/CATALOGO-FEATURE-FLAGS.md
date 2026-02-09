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