# CATALOGO PAYMENTS v1

> **Versione**: 1.0  
> **Data**: 2026-01-27  
> **Ambito**: Stripe payments, Subscriptions, SaaS billing, E-commerce  

---

## 1. PAYMENT PROVIDER COMPARISON

| Provider | Regions | Fees | Payout Speed | Subscriptions | Marketplace | Best For |
|----------|---------|------|--------------|---------------|-------------|----------|
| Stripe | 🟢 Global | 2.9% + $0.30 | 2-7 days | ✅ Excellent | ✅ Connect | Most use cases |
| Paddle | 🟢 Global | 5% + $0.50 | 7-14 days | ✅ Good | ❌ No | SaaS (Merchant of Record) |
| Lemon Squeezy | 🟢 Global | 5% + $0.50 | 7-14 days | ✅ Good | ❌ No | Digital products (MoR) |
| PayPal | 🟢 Global | 2.9% + $0.30 | 1-2 days | ⚠️ Limited | ⚠️ Limited | Consumer checkout |
| Square | 🟡 US/UK/CA/AU | 2.6% + $0.10 | 1-2 days | ⚠️ Limited | ❌ No | In-person + online |
| Braintree | 🟢 Global | 2.9% + $0.30 | 2-4 days | ✅ Good | ❌ No | Enterprise PayPal |
| Adyen | 🟢 Global | €0.10 + % | 2-3 days | ✅ Excellent | ✅ Excellent | Enterprise global |

### Merchant of Record (MoR) Comparison

| Feature | Stripe | Paddle | Lemon Squeezy |
|---------|--------|--------|---------------|
| You handle taxes | ✅ Yes | ❌ No (MoR) | ❌ No (MoR) |
| VAT/GST compliance | 🔴 Manual | 🟢 Automatic | 🟢 Automatic |
| Invoicing | 🟡 Manual | 🟢 Automatic | 🟢 Automatic |
| Chargebacks | 🔴 Your risk | 🟡 Shared | 🟡 Shared |
| Payment methods | 🟢 135+ | 🟡 Limited | 🟡 Limited |
| Complexity | 🟡 Higher | 🟢 Lower | 🟢 Lower |
| Cost transparency | 🟢 Clear | 🔴 Opaque | 🔴 Opaque |
| Best for | All use cases | SaaS products | Digital downloads |

**Decision guide:**  
- **Stripe:** Full control, global, multiple products  
- **Paddle/Lemon Squeezy:** Simplicity, tax handling, digital/SaaS  
- **PayPal:** Consumer checkout supplement  
- **Adyen:** Enterprise, high volume, global payments  

---

## 2. STRIPE SETUP

### 2.1 Installation & Configuration

```typescript
// lib/stripe/client.ts
import Stripe from 'stripe';
import { z } from 'zod';

// Environment validation
const envSchema = z.object({
  STRIPE_SECRET_KEY: z.string().min(1),
  STRIPE_WEBHOOK_SECRET: z.string().min(1),
  STRIPE_PUBLISHABLE_KEY: z.string().min(1),
  NODE_ENV: z.enum(['development', 'production', 'test']),
});

const env = envSchema.parse(process.env);

// Stripe version - pin to specific version for stability
const STRIPE_API_VERSION = '2024-11-20.acacia';

// Initialize Stripe instance with type safety
export const stripe = new Stripe(env.STRIPE_SECRET_KEY, {
  apiVersion: STRIPE_API_VERSION,
  appInfo: {
    name: 'MyApp',
    version: '1.0.0',
    url: 'https://myapp.com',
  },
  typescript: true,
});

// Stripe error handling wrapper
export class StripeError extends Error {
  constructor(
    message: string,
    public originalError: unknown,
    public code?: string,
    public statusCode?: number
  ) {
    super(message);
    this.name = 'StripeError';
  }
}

// Safe Stripe call wrapper with error handling
export async function safeStripeCall<T>(
  operation: () => Promise<T>,
  context: string
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    console.error(`Stripe error in ${context}:`, error);
    
    // Handle Stripe specific errors
    if (error instanceof Stripe.errors.StripeError) {
      throw new StripeError(
        `Stripe ${context} failed: ${error.message}`,
        error,
        error.code,
        error.statusCode
      );
    }
    
    // Handle generic errors
    throw new StripeError(
      `Stripe ${context} failed`,
      error
    );
  }
}

// Webhook signature verification
export function verifyWebhookSignature(
  payload: string | Buffer,
  signature: string,
  secret: string
): Stripe.Event {
  try {
    return stripe.webhooks.constructEvent(
      payload,
      signature,
      secret
    );
  } catch (error) {
    console.error('Webhook signature verification failed:', error);
    throw new StripeError(
      'Webhook signature verification failed',
      error
    );
  }
}

// Utility: Get publishable key
export function getPublishableKey(): string {
  return env.STRIPE_PUBLISHABLE_KEY;
}

// Utility: Check if in test mode
export function isTestMode(): boolean {
  return env.STRIPE_SECRET_KEY.startsWith('sk_test_');
}
```

### 2.2 Environment Variables

```bash
# .env.local
# Development/Test
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

# Production
# STRIPE_SECRET_KEY=sk_live_...
# STRIPE_WEBHOOK_SECRET=whsec_...
# STRIPE_PUBLISHABLE_KEY=pk_live_...
# NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
```

### 2.3 Stripe Client Components

```typescript
// lib/stripe/client-components.ts
'use client';

import { loadStripe, Stripe } from '@stripe/stripe-js';
import { Elements } from '@stripe/react-stripe-js';

let stripePromise: Promise<Stripe | null>;

export function getStripe(): Promise<Stripe | null> {
  if (!stripePromise) {
    stripePromise = loadStripe(
      process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
    );
  }
  return stripePromise;
}

// Stripe Elements provider wrapper
export function StripeProvider({
  children,
  options = {},
}: {
  children: React.ReactNode;
  options?: any;
}) {
  return (
    <Elements stripe={getStripe()} options={options}>
      {children}
    </Elements>
  );
}

// Hook to get Stripe instance
export function useStripe() {
  const { stripe } = useElements();
  return stripe;
}

// Hook to get Elements instance
export function useStripeElements() {
  const { elements } = useElements();
  return elements;
}

// Custom hook for Stripe.js loading state
export function useStripeJs() {
  const [stripeJs, setStripeJs] = useState<Stripe | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    getStripe()
      .then(stripe => {
        setStripeJs(stripe);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, []);

  return { stripeJs, loading, error };
}
```

---

## 3. ONE-TIME PAYMENTS

### 3.1 Checkout Session Flow

```
1. User clicks "Buy Now" / "Checkout"
2. Client → POST /api/checkout
3. Server → stripe.checkout.sessions.create()
4. Server returns → { sessionId: 'cs_test_...' }
5. Client → redirectToCheckout({ sessionId })
6. Stripe Checkout UI → Payment flow
7. Payment succeeds → Webhook → checkout.session.completed
8. Webhook handler → Fulfill order
9. User redirected → /success?session_id={CHECKOUT_SESSION_ID}
10. Success page → Retrieve session → Show confirmation
```

### 3.2 Server-side Checkout Session

```typescript
// app/api/checkout/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { stripe, safeStripeCall } from '@/lib/stripe/client';
import { getCurrentUser } from '@/lib/auth-helpers';

// Input validation schema
const checkoutSchema = z.object({
  priceId: z.string().min(1),
  quantity: z.number().int().positive().default(1),
  successUrl: z.string().url().optional(),
  cancelUrl: z.string().url().optional(),
  clientReferenceId: z.string().optional(),
  metadata: z.record(z.string()).optional(),
  promotionCode: z.string().optional(),
});

export async function POST(request: NextRequest) {
  try {
    // Authentication check
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }

    // Parse and validate input
    const body = await request.json();
    const validation = checkoutSchema.safeParse(body);
    
    if (!validation.success) {
      return NextResponse.json(
        { 
          error: 'Invalid input',
          details: validation.error.flatten().fieldErrors 
        },
        { status: 422 }
      );
    }

    const { 
      priceId, 
      quantity, 
      successUrl, 
      cancelUrl,
      clientReferenceId,
      metadata,
      promotionCode 
    } = validation.data;

    // Get or create Stripe customer
    let customerId = user.stripeCustomerId;
    
    if (!customerId) {
      const customer = await safeStripeCall(
        () => stripe.customers.create({
          email: user.email,
          name: user.name,
          metadata: {
            userId: user.id,
            source: 'checkout',
          },
        }),
        'customer.create'
      );
      
      customerId = customer.id;
      
      // Update user in database with Stripe customer ID
      // await updateUserStripeCustomerId(user.id, customer.id);
    }

    // Create checkout session
    const session = await safeStripeCall(
      () => stripe.checkout.sessions.create({
        customer: customerId,
        payment_method_types: ['card'],
        line_items: [
          {
            price: priceId,
            quantity: quantity,
            adjustable_quantity: {
              enabled: true,
              minimum: 1,
              maximum: 10,
            },
          },
        ],
        mode: 'payment',
        success_url: successUrl || 
          `${process.env.NEXT_PUBLIC_APP_URL}/checkout/success?session_id={CHECKOUT_SESSION_ID}`,
        cancel_url: cancelUrl || 
          `${process.env.NEXT_PUBLIC_APP_URL}/checkout`,
        allow_promotion_codes: !!promotionCode,
        ...(promotionCode && { discounts: [{ promotion_code: promotionCode }] }),
        metadata: {
          userId: user.id,
          orderType: 'one_time',
          ...metadata,
        },
        client_reference_id: clientReferenceId || user.id,
        billing_address_collection: 'required',
        shipping_address_collection: {
          allowed_countries: ['US', 'CA', 'GB', 'AU', 'DE', 'FR', 'IT', 'ES'],
        },
        automatic_tax: {
          enabled: true,
        },
        custom_text: {
          submit: {
            message: 'You will be charged immediately. Your order will be processed within 24 hours.',
          },
        },
        expires_at: Math.floor(Date.now() / 1000) + 30 * 60, // 30 minutes
      }),
      'checkout.session.create'
    );

    return NextResponse.json({ 
      sessionId: session.id,
      url: session.url,
    });
  } catch (error) {
    console.error('Checkout creation error:', error);
    
    if (error instanceof Error) {
      return NextResponse.json(
        { error: error.message },
        { status: 500 }
      );
    }
    
    return NextResponse.json(
      { error: 'Failed to create checkout session' },
      { status: 500 }
    );
  }
}

// GET endpoint to retrieve session (for success page)
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const sessionId = searchParams.get('session_id');
    
    if (!sessionId) {
      return NextResponse.json(
        { error: 'session_id is required' },
        { status: 400 }
      );
    }
    
    const session = await safeStripeCall(
      () => stripe.checkout.sessions.retrieve(sessionId, {
        expand: ['line_items', 'customer'],
      }),
      'checkout.session.retrieve'
    );
    
    return NextResponse.json({ session });
  } catch (error) {
    console.error('Session retrieval error:', error);
    return NextResponse.json(
      { error: 'Failed to retrieve session' },
      { status: 500 }
    );
  }
}
```

### 3.3 Client-side Checkout Button

```typescript
// components/CheckoutButton.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Button } from '@/components/ui/button';
import { loadStripe } from '@stripe/stripe-js';
import { toast } from 'sonner';
import { Loader2, ShoppingCart } from 'lucide-react';

interface CheckoutButtonProps {
  priceId: string;
  quantity?: number;
  className?: string;
  variant?: 'default' | 'outline' | 'secondary' | 'ghost' | 'link';
  size?: 'default' | 'sm' | 'lg' | 'icon';
  children?: React.ReactNode;
  metadata?: Record<string, string>;
  promotionCode?: string;
}

export function CheckoutButton({
  priceId,
  quantity = 1,
  className,
  variant = 'default',
  size = 'default',
  children,
  metadata,
  promotionCode,
}: CheckoutButtonProps) {
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const handleCheckout = async () => {
    setLoading(true);
    
    try {
      // Create checkout session
      const response = await fetch('/api/checkout', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          priceId,
          quantity,
          metadata,
          promotionCode,
        }),
      });

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Failed to create checkout session');
      }

      // Redirect to Stripe Checkout
      const stripe = await loadStripe(
        process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
      );
      
      if (!stripe) {
        throw new Error('Stripe failed to load');
      }

      const { error } = await stripe.redirectToCheckout({
        sessionId: data.sessionId,
      });

      if (error) {
        throw error;
      }
    } catch (error) {
      console.error('Checkout error:', error);
      toast.error(
        error instanceof Error 
          ? error.message 
          : 'Failed to start checkout'
      );
    } finally {
      setLoading(false);
    }
  };

  return (
    <Button
      onClick={handleCheckout}
      disabled={loading}
      className={className}
      variant={variant}
      size={size}
    >
      {loading ? (
        <>
          <Loader2 className="mr-2 h-4 w-4 animate-spin" />
          Processing...
        </>
      ) : (
        <>
          <ShoppingCart className="mr-2 h-4 w-4" />
          {children || 'Buy Now'}
        </>
      )}
    </Button>
  );
}

// Example usage:
// <CheckoutButton 
//   priceId="price_abc123" 
//   metadata={{ productId: "prod_xyz" }}
// >
//   Purchase License
// </CheckoutButton>
```

### 3.4 Checkout Configuration Table

| Setting | Purpose | Example | Required |
|---------|---------|---------|----------|
| `mode` | Payment type | `'payment'` / `'subscription'` | ✅ |
| `line_items` | Products to purchase | `[{ price: 'price_123', quantity: 1 }]` | ✅ |
| `success_url` | Post-payment redirect | `/success?session_id={CHECKOUT_SESSION_ID}` | ✅ |
| `cancel_url` | Cancel redirect | `/checkout` | ✅ |
| `customer` | Customer ID | `'cus_abc123'` | ⚠️ |
| `customer_email` | Pre-fill email | `'user@example.com'` | ⚠️ |
| `metadata` | Custom data | `{ orderId: '123', userId: '456' }` | ❌ |
| `client_reference_id` | Internal reference | `'order_789'` | ❌ |
| `allow_promotion_codes` | Coupons | `true` | ❌ |
| `billing_address_collection` | Address collection | `'required'` / `'auto'` | ❌ |
| `shipping_address_collection` | Shipping countries | `{ allowed_countries: ['US', 'CA'] }` | ❌ |
| `automatic_tax` | Tax calculation | `{ enabled: true }` | ❌ |
| `custom_text` | Custom messaging | `{ submit: { message: '...' } }` | ❌ |
| `expires_at` | Session expiry | UNIX timestamp | ❌ |

### 3.5 Embedded Checkout (Elements)

```typescript
// components/EmbeddedCheckout.tsx
'use client';

import { useState, FormEvent } from 'react';
import {
  PaymentElement,
  useStripe,
  useElements,
} from '@stripe/react-stripe-js';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Loader2 } from 'lucide-react';

interface EmbeddedCheckoutProps {
  clientSecret: string;
  onSuccess?: () => void;
  onError?: (error: Error) => void;
}

export function EmbeddedCheckout({ 
  clientSecret, 
  onSuccess, 
  onError 
}: EmbeddedCheckoutProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string>();

  const handleSubmit = async (e: FormEvent) => {
    e.preventDefault();
    
    if (!stripe || !elements) {
      return;
    }

    setLoading(true);
    setError(undefined);

    try {
      const { error: submitError } = await stripe.confirmPayment({
        elements,
        confirmParams: {
          return_url: `${window.location.origin}/payment/complete`,
        },
        redirect: 'if_required',
      });

      if (submitError) {
        setError(submitError.message);
        onError?.(new Error(submitError.message));
      } else {
        // Payment succeeded
        onSuccess?.();
      }
    } catch (err) {
      const errorMessage = err instanceof Error ? err.message : 'Payment failed';
      setError(errorMessage);
      onError?.(new Error(errorMessage));
    } finally {
      setLoading(false);
    }
  };

  return (
    <Card className="max-w-md mx-auto">
      <CardHeader>
        <CardTitle>Complete Payment</CardTitle>
      </CardHeader>
      <form onSubmit={handleSubmit}>
        <CardContent className="space-y-4">
          <PaymentElement />
          
          {error && (
            <div className="p-3 bg-red-50 border border-red-200 rounded-md">
              <p className="text-sm text-red-600">{error}</p>
            </div>
          )}
        </CardContent>
        <CardFooter>
          <Button 
            type="submit" 
            disabled={!stripe || loading}
            className="w-full"
          >
            {loading ? (
              <>
                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                Processing...
              </>
            ) : (
              'Pay Now'
            )}
          </Button>
        </CardFooter>
      </form>
    </Card>
  );
}
```

---

## 4. SUBSCRIPTIONS

### 4.1 Subscription Models Table

| Model | Billing | Use Case | Stripe Implementation | Complexity |
|-------|---------|----------|----------------------|------------|
| Flat rate | Fixed monthly | SaaS basic plan | Single `price` with `recurring` | 🟢 Low |
| Tiered | Usage-based tiers | API calls, storage | Tiered `price` | 🟡 Medium |
| Per-seat | Per user | Team/workspace | Quantity-based `price` | 🟡 Medium |
| Metered | Pay-as-you-go | Compute, bandwidth | Metered `price` + usage records | 🔴 High |
| Hybrid | Base + usage | Enterprise SaaS | Multiple `prices` | 🔴 High |

### 4.2 Price Configuration

```typescript
// lib/stripe/prices.ts
import { stripe, safeStripeCall } from './client';

export interface CreatePriceParams {
  productId: string;
  unitAmount: number; // in cents
  currency: string;
  interval?: 'day' | 'week' | 'month' | 'year';
  intervalCount?: number;
  nickname?: string;
  tiers?: Array<{
    upTo: number | 'inf';
    unitAmount: number;
  }>;
  tiersMode?: 'graduated' | 'volume';
  billingScheme?: 'per_unit' | 'tiered';
  usageType?: 'licensed' | 'metered';
}

export async function createPrice(params: CreatePriceParams) {
  return safeStripeCall(
    () => stripe.prices.create({
      product: params.productId,
      unit_amount: params.unitAmount,
      currency: params.currency,
      ...(params.interval && {
        recurring: {
          interval: params.interval,
          interval_count: params.intervalCount,
          usage_type: params.usageType || 'licensed',
        },
      }),
      ...(params.nickname && { nickname: params.nickname }),
      ...(params.tiers && { 
        tiers: params.tiers,
        tiers_mode: params.tiersMode || 'graduated',
      }),
      billing_scheme: params.billingScheme || 'per_unit',
      metadata: {
        createdBy: 'api',
        timestamp: Date.now().toString(),
      },
    }),
    'price.create'
  );
}

export async function getActivePrices(productId?: string) {
  return safeStripeCall(
    () => stripe.prices.list({
      active: true,
      ...(productId && { product: productId }),
      limit: 100,
    }),
    'price.list'
  );
}
```

### 4.3 Subscription Lifecycle Management

```typescript
// lib/stripe/subscriptions.ts
import { stripe, safeStripeCall } from './client';

export interface CreateSubscriptionParams {
  customerId: string;
  priceId: string;
  quantity?: number;
  trialDays?: number;
  metadata?: Record<string, string>;
  paymentBehavior?: 'default_incomplete' | 'allow_incomplete' | 'error_if_incomplete';
  promotionCode?: string;
}

export async function createSubscription(params: CreateSubscriptionParams) {
  return safeStripeCall(
    () => stripe.subscriptions.create({
      customer: params.customerId,
      items: [
        {
          price: params.priceId,
          quantity: params.quantity,
        },
      ],
      ...(params.trialDays && {
        trial_period_days: params.trialDays,
      }),
      metadata: params.metadata,
      payment_behavior: params.paymentBehavior || 'default_incomplete',
      ...(params.promotionCode && {
        promotion_code: params.promotionCode,
      }),
      expand: ['latest_invoice.payment_intent'],
    }),
    'subscription.create'
  );
}

export async function getSubscription(subscriptionId: string) {
  return safeStripeCall(
    () => stripe.subscriptions.retrieve(subscriptionId, {
      expand: ['customer', 'latest_invoice'],
    }),
    'subscription.retrieve'
  );
}

export async function updateSubscription(
  subscriptionId: string,
  updates: {
    priceId?: string;
    quantity?: number;
    trialEnd?: 'now' | number;
    prorationBehavior?: 'create_prorations' | 'none';
    metadata?: Record<string, string>;
  }
) {
  const subscription = await getSubscription(subscriptionId);
  
  const updateParams: Stripe.SubscriptionUpdateParams = {
    ...(updates.metadata && { metadata: updates.metadata }),
  };
  
  if (updates.priceId) {
    const currentItem = subscription.items.data[0];
    updateParams.items = [
      {
        id: currentItem.id,
        price: updates.priceId,
        quantity: updates.quantity,
      },
    ];
  } else if (updates.quantity !== undefined) {
    const currentItem = subscription.items.data[0];
    updateParams.items = [
      {
        id: currentItem.id,
        quantity: updates.quantity,
      },
    ];
  }
  
  if (updates.trialEnd) {
    updateParams.trial_end = updates.trialEnd;
  }
  
  if (updates.prorationBehavior) {
    updateParams.proration_behavior = updates.prorationBehavior;
  }
  
  return safeStripeCall(
    () => stripe.subscriptions.update(subscriptionId, updateParams),
    'subscription.update'
  );
}

export async function cancelSubscription(
  subscriptionId: string,
  cancelAtPeriodEnd = true
) {
  return safeStripeCall(
    () => stripe.subscriptions.update(subscriptionId, {
      cancel_at_period_end: cancelAtPeriodEnd,
    }),
    'subscription.cancel'
  );
}

export async function reactivateSubscription(subscriptionId: string) {
  return safeStripeCall(
    () => stripe.subscriptions.update(subscriptionId, {
      cancel_at_period_end: false,
    }),
    'subscription.reactivate'
  );
}

export async function listSubscriptions(
  customerId?: string,
  status?: Stripe.SubscriptionListParams.Status
) {
  return safeStripeCall(
    () => stripe.subscriptions.list({
      ...(customerId && { customer: customerId }),
      ...(status && { status }),
      limit: 100,
      expand: ['data.customer'],
    }),
    'subscription.list'
  );
}
```

### 4.4 Subscription Lifecycle Events

| Event | Webhook | Business Logic | Priority |
|-------|---------|----------------|----------|
| Subscription created | `customer.subscription.created` | Provision access, send welcome | 🔴 Critical |
| Subscription updated | `customer.subscription.updated` | Update access, notify plan change | 🟠 High |
| Subscription deleted | `customer.subscription.deleted` | Revoke access, cleanup | 🔴 Critical |
| Trial ending | `customer.subscription.trial_will_end` | Send reminder email | 🟡 Medium |
| Payment failed | `invoice.payment_failed` | Notify user, retry logic | 🟠 High |
| Payment succeeded | `invoice.payment_failed` | Update billing status, extend access | 🟢 Normal |
| Subscription scheduled to cancel | `customer.subscription.updated` | Send cancellation confirmation | 🟡 Medium |

### 4.5 Customer Portal

```typescript
// app/api/customer-portal/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe, safeStripeCall } from '@/lib/stripe/client';
import { getCurrentUser } from '@/lib/auth-helpers';

export async function POST(request: NextRequest) {
  try {
    const user = await getCurrentUser();
    if (!user) {
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }

    if (!user.stripeCustomerId) {
      return NextResponse.json(
        { error: 'No Stripe customer found' },
        { status: 400 }
      );
    }

    // Create portal session
    const portalSession = await safeStripeCall(
      () => stripe.billingPortal.sessions.create({
        customer: user.stripeCustomerId!,
        return_url: `${process.env.NEXT_PUBLIC_APP_URL}/dashboard/billing`,
        configuration: process.env.STRIPE_PORTAL_CONFIGURATION_ID,
        flow_data: {
          type: 'subscription_update_confirm',
          subscription_update_confirm: {
            subscription: user.subscriptions?.[0]?.stripeSubscriptionId,
            items: [
              {
                id: user.subscriptions?.[0]?.id,
                price: process.env.STRIPE_PRICE_ID_PRO,
                quantity: 1,
              },
            ],
          },
        },
      }),
      'billingPortal.session.create'
    );

    return NextResponse.json({ url: portalSession.url });
  } catch (error) {
    console.error('Portal session error:', error);
    return NextResponse.json(
      { error: 'Failed to create portal session' },
      { status: 500 }
    );
  }
}
```

---

## 5. WEBHOOKS

### 5.1 Critical Webhook Events Table

| Event | Priority | Action Required | Idempotent Key |
|-------|----------|-----------------|----------------|
| `checkout.session.completed` | 🔴 Critical | Fulfill order / Provision access | ✅ |
| `customer.subscription.created` | 🔴 Critical | Provision access, send welcome | ✅ |
| `customer.subscription.updated` | 🟠 High | Update access, notify changes | ✅ |
| `customer.subscription.deleted` | 🔴 Critical | Revoke access, cleanup | ✅ |
| `invoice.paid` | 🟢 Normal | Update billing status | ✅ |
| `invoice.payment_failed` | 🟠 High | Notify user, retry logic | ✅ |
| `invoice.payment_action_required` | 🟡 Medium | Notify user to complete payment | ✅ |
| `customer.created` | 🟢 Normal | Sync customer data | ✅ |
| `charge.succeeded` | 🟢 Normal | Log successful charge | ✅ |
| `charge.refunded` | 🟠 High | Handle refund, update status | ✅ |
| `charge.disputed` | 🔴 Critical | Handle chargeback, gather evidence | ✅ |
| `payment_intent.succeeded` | 🟢 Normal | Update payment status | ✅ |
| `payment_intent.payment_failed` | 🟠 High | Notify user of payment failure | ✅ |

### 5.2 Webhook Handler

```typescript
// app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { stripe, verifyWebhookSignature } from '@/lib/stripe/client';
import { prisma } from '@/lib/prisma';

// Handle individual webhook events
async function handleWebhookEvent(event: Stripe.Event) {
  console.log(`Processing webhook: ${event.type} [${event.id}]`);

  switch (event.type) {
    case 'checkout.session.completed':
      await handleCheckoutSessionCompleted(event.data.object as Stripe.Checkout.Session);
      break;
    
    case 'customer.subscription.created':
      await handleSubscriptionCreated(event.data.object as Stripe.Subscription);
      break;
    
    case 'customer.subscription.updated':
      await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
      break;
    
    case 'customer.subscription.deleted':
      await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
      break;
    
    case 'invoice.paid':
      await handleInvoicePaid(event.data.object as Stripe.Invoice);
      break;
    
    case 'invoice.payment_failed':
      await handleInvoicePaymentFailed(event.data.object as Stripe.Invoice);
      break;
    
    case 'charge.refunded':
      await handleChargeRefunded(event.data.object as Stripe.Charge);
      break;
    
    case 'charge.dispute.created':
      await handleDisputeCreated(event.data.object as Stripe.Dispute);
      break;
    
    default:
      console.log(`Unhandled event type: ${event.type}`);
  }
}

// Checkout session completed
async function handleCheckoutSessionCompleted(session: Stripe.Checkout.Session) {
  const { id, customer, metadata, mode } = session;
  
  console.log(`Checkout session ${id} completed for customer ${customer}`);
  
  // Check idempotency
  const existing = await prisma.webhookEvent.findUnique({
    where: { eventId: id },
  });
  
  if (existing) {
    console.log(`Event ${id} already processed`);
    return;
  }
  
  if (mode === 'payment') {
    // Handle one-time payment
    await prisma.order.create({
      data: {
        id: `order_${Date.now()}`,
        stripeSessionId: id,
        stripeCustomerId: customer as string,
        userId: metadata?.userId,
        amount: session.amount_total || 0,
        currency: session.currency || 'usd',
        status: 'paid',
        metadata: metadata,
      },
    });
    
    // Send order confirmation email
    // await sendOrderConfirmationEmail(session.customer_details?.email, metadata);
  } else if (mode === 'subscription') {
    // Handle subscription creation
    const subscription = await stripe.subscriptions.retrieve(
      session.subscription as string
    );
    
    await prisma.subscription.create({
      data: {
        id: subscription.id,
        userId: metadata?.userId,
        stripeCustomerId: customer as string,
        stripePriceId: subscription.items.data[0].price.id,
        status: subscription.status,
        currentPeriodStart: new Date(subscription.current_period_start * 1000),
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
        cancelAtPeriodEnd: subscription.cancel_at_period_end || false,
        metadata: metadata,
      },
    });
    
    // Provision access
    // await provisionUserAccess(metadata?.userId, subscription);
  }
  
  // Record processed event
  await prisma.webhookEvent.create({
    data: {
      eventId: id,
      type: session.object,
      data: session,
    },
  });
}

// Subscription created
async function handleSubscriptionCreated(subscription: Stripe.Subscription) {
  const { id, customer, metadata } = subscription;
  
  console.log(`Subscription ${id} created for customer ${customer}`);
  
  await prisma.subscription.upsert({
    where: { id },
    update: {
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      cancelAtPeriodEnd: subscription.cancel_at_period_end || false,
    },
    create: {
      id,
      userId: metadata?.userId,
      stripeCustomerId: customer as string,
      stripePriceId: subscription.items.data[0].price.id,
      status: subscription.status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      cancelAtPeriodEnd: subscription.cancel_at_period_end || false,
      metadata: metadata,
    },
  });
  
  // Send welcome email for new subscription
  // await sendSubscriptionWelcomeEmail(customer, subscription);
}

// Subscription updated
async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  const { id, status, metadata } = subscription;
  
  console.log(`Subscription ${id} updated to status: ${status}`);
  
  await prisma.subscription.update({
    where: { id },
    data: {
      status,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      cancelAtPeriodEnd: subscription.cancel_at_period_end || false,
    },
  });
  
  // Handle plan changes
  if (metadata?.planChanged) {
    // await handlePlanChange(subscription);
  }
}

// Subscription deleted
async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  const { id, metadata } = subscription;
  
  console.log(`Subscription ${id} deleted`);
  
  await prisma.subscription.update({
    where: