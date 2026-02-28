# CATALOGO PAYMENTS v1

> **Versione**: 1.0  
> **Data**: 2026-01-27  
> **Ambito**: Stripe payments, Subscriptions, SaaS billing, E-commerce  

---

§ 1. PAYMENT PROVIDER COMPARISON

| Provider | Regions | Fees | Payout Speed | Subscriptions | Marketplace | Best For |
|----------|---------|------|--------------|---------------|-------------|----------|
| Stripe | 🟢 Global | 2.9% + $0.30 | 2-7 days | ✅ Excellent | ✅ Connect | Most use cases |
| Paddle | 🟢 Global | 5% + $0.50 | 7-14 days | ✅ Good | ❌ No | SaaS (Merchant of Record) |
| Lemon Squeezy | 🟢 Global | 5% + $0.50 | 7-14 days | ✅ Good | ❌ No | Digital products (MoR) |
| PayPal | 🟢 Global | 2.9% + $0.30 | 1-2 days | ⚠️ Limited | ⚠️ Limited | Consumer checkout |
| Square | 🟡 US/UK/CA/AU | 2.6% + $0.10 | 1-2 days | ⚠️ Limited | ❌ No | In-person + online |
| Braintree | 🟢 Global | 2.9% + $0.30 | 2-4 days | ✅ Good | ❌ No | Enterprise PayPal |
| Adyen | 🟢 Global | €0.10 + % | 2-3 days | ✅ Excellent | ✅ Excellent | Enterprise global |

§ MERCHANT OF RECORD (MOR) COMPARISON

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

§ 2. STRIPE SETUP

§ 2.1 INSTALLATION & CONFIGURATION

typescript
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

§ 2.2 ENVIRONMENT VARIABLES

bash
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

§ 2.3 STRIPE CLIENT COMPONENTS

typescript
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

---

§ 3. ONE-TIME PAYMENTS

§ 3.1 CHECKOUT SESSION FLOW

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

§ 3.2 SERVER-SIDE CHECKOUT SESSION

typescript
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

§ 3.3 CLIENT-SIDE CHECKOUT BUTTON

typescript
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

§ 3.4 CHECKOUT CONFIGURATION TABLE

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

§ 3.5 EMBEDDED CHECKOUT (ELEMENTS)

typescript
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

---

§ 4. SUBSCRIPTIONS

§ 4.1 SUBSCRIPTION MODELS TABLE

| Model | Billing | Use Case | Stripe Implementation | Complexity |
|-------|---------|----------|----------------------|------------|
| Flat rate | Fixed monthly | SaaS basic plan | Single `price` with `recurring` | 🟢 Low |
| Tiered | Usage-based tiers | API calls, storage | Tiered `price` | 🟡 Medium |
| Per-seat | Per user | Team/workspace | Quantity-based `price` | 🟡 Medium |
| Metered | Pay-as-you-go | Compute, bandwidth | Metered `price` + usage records | 🔴 High |
| Hybrid | Base + usage | Enterprise SaaS | Multiple `prices` | 🔴 High |

§ 4.2 PRICE CONFIGURATION

typescript
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

§ 4.3 SUBSCRIPTION LIFECYCLE MANAGEMENT

typescript
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

§ 4.4 SUBSCRIPTION LIFECYCLE EVENTS

| Event | Webhook | Business Logic | Priority |
|-------|---------|----------------|----------|
| Subscription created | `customer.subscription.created` | Provision access, send welcome | 🔴 Critical |
| Subscription updated | `customer.subscription.updated` | Update access, notify plan change | 🟠 High |
| Subscription deleted | `customer.subscription.deleted` | Revoke access, cleanup | 🔴 Critical |
| Trial ending | `customer.subscription.trial_will_end` | Send reminder email | 🟡 Medium |
| Payment failed | `invoice.payment_failed` | Notify user, retry logic | 🟠 High |
| Payment succeeded | `invoice.payment_failed` | Update billing status, extend access | 🟢 Normal |
| Subscription scheduled to cancel | `customer.subscription.updated` | Send cancellation confirmation | 🟡 Medium |

§ 4.5 CUSTOMER PORTAL

typescript
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

---

§ 5. WEBHOOKS

§ 5.1 CRITICAL WEBHOOK EVENTS TABLE

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

§ 5.2 WEBHOOK HANDLER

typescript
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


---

§ 10. STRIPE CHECKOUT — ONE-TIME PAYMENTS

§ 10.1 PANORAMICA
Implementazione completa di Stripe Checkout per pagamenti one-time con gestione
sessione, pagine di successo/cancellazione, e line items dinamici.

§ 10.2 IMPLEMENTAZIONE COMPLETA

```typescript
// app/api/checkout/route.ts — Create Checkout Session

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { z } from "zod";
import { prisma } from "@/lib/db/client";
import { getServerSession } from "@/lib/auth";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
  typescript: true,
});

const checkoutSchema = z.object({
  items: z.array(
    z.object({
      productId: z.string().cuid(),
      variantId: z.string().cuid().optional(),
      quantity: z.number().int().min(1).max(99),
    })
  ).min(1),
  couponCode: z.string().optional(),
  shippingAddressId: z.string().cuid().optional(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { items, couponCode, shippingAddressId } = checkoutSchema.parse(body);

    // Fetch products with prices from database
    const productIds = items.map((i) => i.productId);
    const products = await prisma.product.findMany({
      where: { id: { in: productIds }, isActive: true, deletedAt: null },
      include: {
        variants: true,
        images: { where: { isPrimary: true }, take: 1 },
      },
    });

    if (products.length !== productIds.length) {
      return NextResponse.json(
        { error: "One or more products not found or inactive" },
        { status: 400 }
      );
    }

    // Build line items
    const lineItems: Stripe.Checkout.SessionCreateParams.LineItem[] = [];

    for (const item of items) {
      const product = products.find((p) => p.id === item.productId);
      if (!product) continue;

      let price: number;
      let name: string;

      if (item.variantId) {
        const variant = product.variants.find((v) => v.id === item.variantId);
        if (!variant) {
          return NextResponse.json(
            { error: `Variant ${item.variantId} not found` },
            { status: 400 }
          );
        }
        price = Number(variant.price);
        name = `${product.name} - ${variant.name}`;
      } else {
        price = Number(product.price);
        name = product.name;
      }

      // Check stock
      const availableStock = item.variantId
        ? product.variants.find((v) => v.id === item.variantId)?.stock ?? 0
        : product.stock;

      if (availableStock < item.quantity) {
        return NextResponse.json(
          { error: `Insufficient stock for ${name}` },
          { status: 400 }
        );
      }

      lineItems.push({
        price_data: {
          currency: "eur",
          product_data: {
            name,
            description: product.shortDescription ?? undefined,
            images: product.images[0]?.url ? [product.images[0].url] : undefined,
            metadata: {
              productId: product.id,
              variantId: item.variantId ?? "",
              sku: product.sku,
            },
          },
          unit_amount: Math.round(price * 100),
        },
        quantity: item.quantity,
      });
    }

    // Get or create Stripe customer
    let stripeCustomerId: string;
    const user = await prisma.user.findUnique({
      where: { id: session.user.id },
      select: { email: true, firstName: true, lastName: true },
    });

    const existingCustomers = await stripe.customers.list({
      email: user!.email,
      limit: 1,
    });

    if (existingCustomers.data.length > 0) {
      stripeCustomerId = existingCustomers.data[0].id;
    } else {
      const customer = await stripe.customers.create({
        email: user!.email,
        name: `${user!.firstName} ${user!.lastName}`,
        metadata: { userId: session.user.id },
      });
      stripeCustomerId = customer.id;
    }

    // Create checkout session
    const checkoutSessionParams: Stripe.Checkout.SessionCreateParams = {
      mode: "payment",
      customer: stripeCustomerId,
      line_items: lineItems,
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/checkout/success?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/checkout/cancel`,
      metadata: {
        userId: session.user.id,
        itemsJson: JSON.stringify(items),
        shippingAddressId: shippingAddressId ?? "",
      },
      payment_intent_data: {
        metadata: { userId: session.user.id },
      },
      shipping_options: [
        {
          shipping_rate_data: {
            type: "fixed_amount",
            fixed_amount: { amount: 0, currency: "eur" },
            display_name: "Spedizione gratuita",
            delivery_estimate: {
              minimum: { unit: "business_day", value: 3 },
              maximum: { unit: "business_day", value: 5 },
            },
          },
        },
        {
          shipping_rate_data: {
            type: "fixed_amount",
            fixed_amount: { amount: 999, currency: "eur" },
            display_name: "Spedizione express",
            delivery_estimate: {
              minimum: { unit: "business_day", value: 1 },
              maximum: { unit: "business_day", value: 2 },
            },
          },
        },
      ],
      allow_promotion_codes: true,
      locale: "it",
      expires_at: Math.floor(Date.now() / 1000) + 30 * 60,
    };

    if (couponCode) {
      const coupons = await stripe.coupons.list({ limit: 100 });
      const matchingCoupon = coupons.data.find(
        (c) => c.name?.toUpperCase() === couponCode.toUpperCase()
      );
      if (matchingCoupon) {
        checkoutSessionParams.discounts = [{ coupon: matchingCoupon.id }];
        delete checkoutSessionParams.allow_promotion_codes;
      }
    }

    const checkoutSession = await stripe.checkout.sessions.create(
      checkoutSessionParams
    );

    return NextResponse.json({
      sessionId: checkoutSession.id,
      url: checkoutSession.url,
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Validation error", details: error.errors },
        { status: 422 }
      );
    }
    console.error("[Checkout] Error:", error);
    return NextResponse.json(
      { error: "Failed to create checkout session" },
      { status: 500 }
    );
  }
}
```

§ 10.3 SUCCESS PAGE

```typescript
// app/checkout/success/page.tsx — Checkout Success Page

import { redirect } from "next/navigation";
import Stripe from "stripe";
import { prisma } from "@/lib/db/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { CheckCircle2, Package, ArrowRight } from "lucide-react";
import Link from "next/link";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

interface Props {
  searchParams: { session_id?: string };
}

export default async function CheckoutSuccessPage({ searchParams }: Props) {
  const sessionId = searchParams.session_id;
  if (!sessionId) redirect("/");

  let session: Stripe.Checkout.Session;
  try {
    session = await stripe.checkout.sessions.retrieve(sessionId, {
      expand: ["line_items", "payment_intent", "customer"],
    });
  } catch {
    redirect("/");
  }

  if (session.payment_status !== "paid") {
    redirect("/checkout/cancel");
  }

  const order = await prisma.order.findFirst({
    where: { paymentIntentId: (session.payment_intent as Stripe.PaymentIntent)?.id },
    include: {
      items: true,
      shippingAddress: true,
    },
  });

  return (
    <div className="container max-w-2xl mx-auto py-16 px-4">
      <div className="text-center mb-8">
        <CheckCircle2 className="mx-auto h-16 w-16 text-green-500 mb-4" />
        <h1 className="text-3xl font-bold mb-2">Ordine confermato!</h1>
        <p className="text-muted-foreground">
          Grazie per il tuo acquisto. Riceverai una email di conferma a breve.
        </p>
      </div>

      <Card className="mb-6">
        <CardHeader>
          <div className="flex items-center justify-between">
            <CardTitle className="text-lg">Dettagli Ordine</CardTitle>
            {order && (
              <Badge variant="secondary">{order.orderNumber}</Badge>
            )}
          </div>
        </CardHeader>
        <CardContent className="space-y-4">
          {session.line_items?.data.map((item) => (
            <div
              key={item.id}
              className="flex items-center justify-between py-2 border-b last:border-0"
            >
              <div>
                <p className="font-medium">{item.description}</p>
                <p className="text-sm text-muted-foreground">
                  Quantita: {item.quantity}
                </p>
              </div>
              <p className="font-semibold">
                {new Intl.NumberFormat("it-IT", {
                  style: "currency",
                  currency: item.currency ?? "eur",
                }).format((item.amount_total ?? 0) / 100)}
              </p>
            </div>
          ))}

          <div className="pt-4 border-t space-y-2">
            <div className="flex justify-between text-sm">
              <span>Subtotale</span>
              <span>
                {new Intl.NumberFormat("it-IT", {
                  style: "currency",
                  currency: "eur",
                }).format((session.amount_subtotal ?? 0) / 100)}
              </span>
            </div>
            {(session.total_details?.amount_shipping ?? 0) > 0 && (
              <div className="flex justify-between text-sm">
                <span>Spedizione</span>
                <span>
                  {new Intl.NumberFormat("it-IT", {
                    style: "currency",
                    currency: "eur",
                  }).format(
                    (session.total_details?.amount_shipping ?? 0) / 100
                  )}
                </span>
              </div>
            )}
            {(session.total_details?.amount_discount ?? 0) > 0 && (
              <div className="flex justify-between text-sm text-green-600">
                <span>Sconto</span>
                <span>
                  -
                  {new Intl.NumberFormat("it-IT", {
                    style: "currency",
                    currency: "eur",
                  }).format(
                    (session.total_details?.amount_discount ?? 0) / 100
                  )}
                </span>
              </div>
            )}
            <div className="flex justify-between font-bold text-lg pt-2 border-t">
              <span>Totale</span>
              <span>
                {new Intl.NumberFormat("it-IT", {
                  style: "currency",
                  currency: "eur",
                }).format((session.amount_total ?? 0) / 100)}
              </span>
            </div>
          </div>
        </CardContent>
      </Card>

      {order?.shippingAddress && (
        <Card className="mb-6">
          <CardHeader>
            <CardTitle className="text-lg flex items-center gap-2">
              <Package className="h-5 w-5" />
              Indirizzo di spedizione
            </CardTitle>
          </CardHeader>
          <CardContent>
            <p>
              {order.shippingAddress.firstName} {order.shippingAddress.lastName}
            </p>
            <p>{order.shippingAddress.street}</p>
            <p>
              {order.shippingAddress.zipCode} {order.shippingAddress.city} (
              {order.shippingAddress.state})
            </p>
          </CardContent>
        </Card>
      )}

      <div className="flex gap-4 justify-center">
        <Button asChild>
          <Link href="/orders">
            I miei ordini <ArrowRight className="ml-2 h-4 w-4" />
          </Link>
        </Button>
        <Button variant="outline" asChild>
          <Link href="/products">Continua lo shopping</Link>
        </Button>
      </div>
    </div>
  );
}
```

---

§ 11. STRIPE SUBSCRIPTIONS — LIFECYCLE COMPLETO

§ 11.1 PANORAMICA
Gestione completa delle subscription Stripe: creazione piani, checkout subscription,
billing portal, upgrade/downgrade, trial, cancellation, pause/resume.

§ 11.2 PIANO E PRICING CONFIG

```typescript
// lib/stripe/plans.ts — Subscription plans configuration

export interface PlanConfig {
  id: string;
  name: string;
  description: string;
  monthlyPriceId: string;
  yearlyPriceId: string;
  monthlyPrice: number;
  yearlyPrice: number;
  features: string[];
  limits: Record<string, number>;
  recommended?: boolean;
  badge?: string;
}

export const PLANS: PlanConfig[] = [
  {
    id: "starter",
    name: "Starter",
    description: "Per freelancer e piccoli team",
    monthlyPriceId: process.env.STRIPE_STARTER_MONTHLY_PRICE_ID!,
    yearlyPriceId: process.env.STRIPE_STARTER_YEARLY_PRICE_ID!,
    monthlyPrice: 19,
    yearlyPrice: 190,
    features: [
      "Fino a 5 utenti",
      "5 GB storage",
      "Email support",
      "API access base",
    ],
    limits: { users: 5, storage_gb: 5, api_calls: 10000 },
  },
  {
    id: "pro",
    name: "Pro",
    description: "Per team in crescita",
    monthlyPriceId: process.env.STRIPE_PRO_MONTHLY_PRICE_ID!,
    yearlyPriceId: process.env.STRIPE_PRO_YEARLY_PRICE_ID!,
    monthlyPrice: 49,
    yearlyPrice: 490,
    features: [
      "Fino a 20 utenti",
      "50 GB storage",
      "Priority support",
      "API access completo",
      "Webhooks",
      "Custom domain",
    ],
    limits: { users: 20, storage_gb: 50, api_calls: 100000 },
    recommended: true,
    badge: "Piu popolare",
  },
  {
    id: "business",
    name: "Business",
    description: "Per aziende enterprise",
    monthlyPriceId: process.env.STRIPE_BUSINESS_MONTHLY_PRICE_ID!,
    yearlyPriceId: process.env.STRIPE_BUSINESS_YEARLY_PRICE_ID!,
    monthlyPrice: 99,
    yearlyPrice: 990,
    features: [
      "Utenti illimitati",
      "500 GB storage",
      "24/7 dedicated support",
      "API access illimitato",
      "SSO / SAML",
      "SLA 99.9%",
      "Audit log",
      "Custom integrations",
    ],
    limits: { users: 999999, storage_gb: 500, api_calls: 999999 },
  },
];

export function getPlanById(planId: string): PlanConfig | undefined {
  return PLANS.find((p) => p.id === planId);
}

export function getPlanByPriceId(priceId: string): PlanConfig | undefined {
  return PLANS.find(
    (p) => p.monthlyPriceId === priceId || p.yearlyPriceId === priceId
  );
}

export function isYearly(priceId: string): boolean {
  return PLANS.some((p) => p.yearlyPriceId === priceId);
}
```

§ 11.3 SUBSCRIPTION CHECKOUT

```typescript
// app/api/subscriptions/checkout/route.ts — Create Subscription Checkout

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";
import { getPlanById } from "@/lib/stripe/plans";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

const subscriptionCheckoutSchema = z.object({
  planId: z.string(),
  interval: z.enum(["monthly", "yearly"]),
  organizationId: z.string().cuid(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { planId, interval, organizationId } =
      subscriptionCheckoutSchema.parse(body);

    const plan = getPlanById(planId);
    if (!plan) {
      return NextResponse.json({ error: "Plan not found" }, { status: 404 });
    }

    const priceId =
      interval === "yearly" ? plan.yearlyPriceId : plan.monthlyPriceId;

    // Get or create Stripe customer
    const { prisma } = await import("@/lib/db/client");
    const org = await prisma.organization.findUnique({
      where: { id: organizationId },
      select: { stripeCustomerId: true, name: true, billingEmail: true },
    });

    if (!org) {
      return NextResponse.json(
        { error: "Organization not found" },
        { status: 404 }
      );
    }

    let customerId = org.stripeCustomerId;

    if (!customerId) {
      const customer = await stripe.customers.create({
        email: org.billingEmail ?? session.user.email,
        name: org.name,
        metadata: {
          organizationId,
          userId: session.user.id,
        },
      });
      customerId = customer.id;

      await prisma.organization.update({
        where: { id: organizationId },
        data: { stripeCustomerId: customerId },
      });
    }

    const checkoutSession = await stripe.checkout.sessions.create({
      mode: "subscription",
      customer: customerId,
      line_items: [{ price: priceId, quantity: 1 }],
      success_url: `${process.env.NEXT_PUBLIC_APP_URL}/settings/billing?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
      subscription_data: {
        trial_period_days: 14,
        metadata: {
          organizationId,
          planId,
          interval,
        },
      },
      allow_promotion_codes: true,
      billing_address_collection: "auto",
      tax_id_collection: { enabled: true },
      locale: "it",
    });

    return NextResponse.json({ url: checkoutSession.url });
  } catch (error) {
    console.error("[Subscription Checkout] Error:", error);
    return NextResponse.json(
      { error: "Failed to create checkout" },
      { status: 500 }
    );
  }
}
```

§ 11.4 BILLING PORTAL

```typescript
// app/api/subscriptions/portal/route.ts — Stripe Customer Portal

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { getServerSession } from "@/lib/auth";
import { prisma } from "@/lib/db/client";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const { organizationId } = await request.json();

    const org = await prisma.organization.findUnique({
      where: { id: organizationId },
      select: { stripeCustomerId: true },
    });

    if (!org?.stripeCustomerId) {
      return NextResponse.json(
        { error: "No billing account found" },
        { status: 404 }
      );
    }

    const portalSession = await stripe.billingPortal.sessions.create({
      customer: org.stripeCustomerId,
      return_url: `${process.env.NEXT_PUBLIC_APP_URL}/settings/billing`,
    });

    return NextResponse.json({ url: portalSession.url });
  } catch (error) {
    console.error("[Billing Portal] Error:", error);
    return NextResponse.json(
      { error: "Failed to create portal session" },
      { status: 500 }
    );
  }
}
```

§ 11.5 UPGRADE / DOWNGRADE

```typescript
// app/api/subscriptions/change-plan/route.ts — Change subscription plan

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";
import { prisma } from "@/lib/db/client";
import { getPlanById } from "@/lib/stripe/plans";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

const changePlanSchema = z.object({
  organizationId: z.string().cuid(),
  newPlanId: z.string(),
  interval: z.enum(["monthly", "yearly"]),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { organizationId, newPlanId, interval } = changePlanSchema.parse(body);

    const plan = getPlanById(newPlanId);
    if (!plan) {
      return NextResponse.json({ error: "Plan not found" }, { status: 404 });
    }

    const org = await prisma.organization.findUnique({
      where: { id: organizationId },
      select: { stripeSubscriptionId: true, plan: true },
    });

    if (!org?.stripeSubscriptionId) {
      return NextResponse.json(
        { error: "No active subscription" },
        { status: 404 }
      );
    }

    const subscription = await stripe.subscriptions.retrieve(
      org.stripeSubscriptionId
    );

    const priceId =
      interval === "yearly" ? plan.yearlyPriceId : plan.monthlyPriceId;

    // Determine proration behavior
    const currentPlan = getPlanById(org.plan.toLowerCase());
    const isUpgrade =
      (currentPlan?.monthlyPrice ?? 0) < plan.monthlyPrice;

    const updatedSubscription = await stripe.subscriptions.update(
      subscription.id,
      {
        items: [
          {
            id: subscription.items.data[0].id,
            price: priceId,
          },
        ],
        proration_behavior: isUpgrade
          ? "create_prorations"
          : "none",
        metadata: {
          organizationId,
          planId: newPlanId,
          interval,
          previousPlan: org.plan,
        },
      }
    );

    // Update organization plan in database
    await prisma.organization.update({
      where: { id: organizationId },
      data: {
        plan: newPlanId.toUpperCase() as any,
        stripeSubscriptionId: updatedSubscription.id,
      },
    });

    // Audit log
    await prisma.orgAuditLog.create({
      data: {
        organizationId,
        actorId: session.user.id,
        action: "subscription.plan_changed",
        entity: "Subscription",
        entityId: updatedSubscription.id,
        changes: {
          plan: { from: org.plan, to: newPlanId.toUpperCase() },
          interval: { to: interval },
          proration: isUpgrade ? "applied" : "none",
        },
      },
    });

    return NextResponse.json({
      subscription: {
        id: updatedSubscription.id,
        status: updatedSubscription.status,
        plan: newPlanId,
        interval,
        currentPeriodEnd: new Date(
          updatedSubscription.current_period_end * 1000
        ).toISOString(),
      },
    });
  } catch (error) {
    console.error("[Change Plan] Error:", error);
    return NextResponse.json(
      { error: "Failed to change plan" },
      { status: 500 }
    );
  }
}
```

---

§ 12. STRIPE WEBHOOK HANDLER

§ 12.1 PANORAMICA
Webhook handler completo con signature verification, event routing,
idempotency, error handling e retry-safe processing.

§ 12.2 IMPLEMENTAZIONE COMPLETA

```typescript
// app/api/webhooks/stripe/route.ts — Stripe Webhook Handler

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { headers } from "next/headers";
import { prisma } from "@/lib/db/client";
import { getPlanByPriceId } from "@/lib/stripe/plans";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

// Idempotency: track processed events
async function isEventProcessed(eventId: string): Promise<boolean> {
  const existing = await prisma.$queryRaw<Array<{ count: number }>>`
    SELECT count(*)::int FROM processed_webhook_events
    WHERE event_id = ${eventId}
  `;
  return (existing[0]?.count ?? 0) > 0;
}

async function markEventProcessed(eventId: string, eventType: string): Promise<void> {
  await prisma.$executeRaw`
    INSERT INTO processed_webhook_events (event_id, event_type, processed_at)
    VALUES (${eventId}, ${eventType}, NOW())
    ON CONFLICT (event_id) DO NOTHING
  `;
}

export async function POST(request: NextRequest) {
  const body = await request.text();
  const headersList = headers();
  const signature = headersList.get("stripe-signature");

  if (!signature) {
    return NextResponse.json(
      { error: "Missing stripe-signature header" },
      { status: 400 }
    );
  }

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (err) {
    console.error("[Webhook] Signature verification failed:", err);
    return NextResponse.json(
      { error: "Invalid signature" },
      { status: 400 }
    );
  }

  // Idempotency check
  if (await isEventProcessed(event.id)) {
    console.log(`[Webhook] Event ${event.id} already processed, skipping`);
    return NextResponse.json({ received: true });
  }

  console.log(`[Webhook] Processing event: ${event.type} (${event.id})`);

  try {
    switch (event.type) {
      // ── Checkout completed ──
      case "checkout.session.completed":
        await handleCheckoutCompleted(
          event.data.object as Stripe.Checkout.Session
        );
        break;

      // ── Subscription events ──
      case "customer.subscription.created":
        await handleSubscriptionCreated(
          event.data.object as Stripe.Subscription
        );
        break;

      case "customer.subscription.updated":
        await handleSubscriptionUpdated(
          event.data.object as Stripe.Subscription
        );
        break;

      case "customer.subscription.deleted":
        await handleSubscriptionDeleted(
          event.data.object as Stripe.Subscription
        );
        break;

      // ── Payment events ──
      case "payment_intent.succeeded":
        await handlePaymentSucceeded(
          event.data.object as Stripe.PaymentIntent
        );
        break;

      case "payment_intent.payment_failed":
        await handlePaymentFailed(
          event.data.object as Stripe.PaymentIntent
        );
        break;

      // ── Invoice events ──
      case "invoice.paid":
        await handleInvoicePaid(event.data.object as Stripe.Invoice);
        break;

      case "invoice.payment_failed":
        await handleInvoicePaymentFailed(
          event.data.object as Stripe.Invoice
        );
        break;

      // ── Refund events ──
      case "charge.refunded":
        await handleChargeRefunded(event.data.object as Stripe.Charge);
        break;

      default:
        console.log(`[Webhook] Unhandled event type: ${event.type}`);
    }

    await markEventProcessed(event.id, event.type);
    return NextResponse.json({ received: true });
  } catch (error) {
    console.error(`[Webhook] Error processing ${event.type}:`, error);
    return NextResponse.json(
      { error: "Webhook processing failed" },
      { status: 500 }
    );
  }
}

// ── Event Handlers ──

async function handleCheckoutCompleted(
  session: Stripe.Checkout.Session
): Promise<void> {
  if (session.mode === "payment") {
    // One-time payment: create order
    const userId = session.metadata?.userId;
    const itemsJson = session.metadata?.itemsJson;
    const shippingAddressId = session.metadata?.shippingAddressId;

    if (!userId || !itemsJson) return;

    const items = JSON.parse(itemsJson) as Array<{
      productId: string;
      variantId?: string;
      quantity: number;
    }>;

    // Build order items
    const orderItems = [];
    let subtotal = 0;

    for (const item of items) {
      const product = await prisma.product.findUnique({
        where: { id: item.productId },
        include: { variants: { where: { id: item.variantId ?? "" } } },
      });
      if (!product) continue;

      const variant = item.variantId ? product.variants[0] : null;
      const price = Number(variant?.price ?? product.price);
      const total = price * item.quantity;
      subtotal += total;

      orderItems.push({
        productId: product.id,
        variantId: item.variantId ?? null,
        name: variant ? `${product.name} - ${variant.name}` : product.name,
        sku: variant?.sku ?? product.sku,
        price,
        quantity: item.quantity,
        total,
      });

      // Decrement stock
      if (variant) {
        await prisma.productVariant.update({
          where: { id: variant.id },
          data: { stock: { decrement: item.quantity } },
        });
      } else {
        await prisma.product.update({
          where: { id: product.id },
          data: { stock: { decrement: item.quantity } },
        });
      }
    }

    const tax = Math.round(subtotal * 0.22 * 100) / 100;
    const shippingCost = (session.total_details?.amount_shipping ?? 0) / 100;
    const discount = (session.total_details?.amount_discount ?? 0) / 100;
    const total = (session.amount_total ?? 0) / 100;

    const date = new Date();
    const orderNumber = `ORD-${date.getFullYear().toString().slice(-2)}${(date.getMonth() + 1).toString().padStart(2, "0")}-${Math.random().toString(36).substring(2, 8).toUpperCase()}`;

    const defaultAddress = await prisma.address.findFirst({
      where: { userId, isDefault: true },
    });

    await prisma.order.create({
      data: {
        orderNumber,
        userId,
        shippingAddressId: shippingAddressId || defaultAddress?.id || "",
        status: "CONFIRMED",
        paymentStatus: "CAPTURED",
        subtotal,
        tax,
        shippingCost,
        discount,
        total,
        paymentMethod: "card",
        paymentIntentId:
          typeof session.payment_intent === "string"
            ? session.payment_intent
            : session.payment_intent?.id,
        items: { createMany: { data: orderItems } },
        payments: {
          create: {
            stripePaymentId:
              typeof session.payment_intent === "string"
                ? session.payment_intent
                : session.payment_intent?.id,
            amount: total,
            status: "succeeded",
            method: "card",
          },
        },
        timeline: {
          create: {
            type: "order_created",
            title: "Ordine creato",
            message: "Pagamento ricevuto e ordine confermato",
            actorType: "system",
          },
        },
      },
    });

    console.log(`[Webhook] Order ${orderNumber} created for user ${userId}`);
  }

  if (session.mode === "subscription") {
    console.log("[Webhook] Subscription checkout completed");
  }
}

async function handleSubscriptionCreated(
  subscription: Stripe.Subscription
): Promise<void> {
  const organizationId = subscription.metadata.organizationId;
  if (!organizationId) return;

  const priceId = subscription.items.data[0]?.price.id;
  const plan = getPlanByPriceId(priceId);

  await prisma.$transaction([
    prisma.organization.update({
      where: { id: organizationId },
      data: {
        stripeSubscriptionId: subscription.id,
        plan: (plan?.id.toUpperCase() ?? "STARTER") as any,
      },
    }),
    prisma.orgSubscription.create({
      data: {
        organizationId,
        stripeSubscriptionId: subscription.id,
        stripePriceId: priceId,
        status: subscription.status.toUpperCase() as any,
        currentPeriodStart: new Date(subscription.current_period_start * 1000),
        currentPeriodEnd: new Date(subscription.current_period_end * 1000),
        trialStart: subscription.trial_start
          ? new Date(subscription.trial_start * 1000)
          : null,
        trialEnd: subscription.trial_end
          ? new Date(subscription.trial_end * 1000)
          : null,
      },
    }),
  ]);
}

async function handleSubscriptionUpdated(
  subscription: Stripe.Subscription
): Promise<void> {
  const existing = await prisma.orgSubscription.findUnique({
    where: { stripeSubscriptionId: subscription.id },
  });
  if (!existing) return;

  const priceId = subscription.items.data[0]?.price.id;

  await prisma.orgSubscription.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      stripePriceId: priceId,
      status: subscription.status.toUpperCase() as any,
      currentPeriodStart: new Date(subscription.current_period_start * 1000),
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
      cancelAtPeriodEnd: subscription.cancel_at_period_end,
      canceledAt: subscription.canceled_at
        ? new Date(subscription.canceled_at * 1000)
        : null,
    },
  });
}

async function handleSubscriptionDeleted(
  subscription: Stripe.Subscription
): Promise<void> {
  const existing = await prisma.orgSubscription.findUnique({
    where: { stripeSubscriptionId: subscription.id },
    select: { organizationId: true },
  });
  if (!existing) return;

  await prisma.$transaction([
    prisma.orgSubscription.update({
      where: { stripeSubscriptionId: subscription.id },
      data: { status: "CANCELED", canceledAt: new Date() },
    }),
    prisma.organization.update({
      where: { id: existing.organizationId },
      data: { plan: "FREE", stripeSubscriptionId: null },
    }),
  ]);
}

async function handlePaymentSucceeded(
  paymentIntent: Stripe.PaymentIntent
): Promise<void> {
  const order = await prisma.order.findFirst({
    where: { paymentIntentId: paymentIntent.id },
  });
  if (!order) return;

  await prisma.order.update({
    where: { id: order.id },
    data: { paymentStatus: "CAPTURED" },
  });
}

async function handlePaymentFailed(
  paymentIntent: Stripe.PaymentIntent
): Promise<void> {
  const order = await prisma.order.findFirst({
    where: { paymentIntentId: paymentIntent.id },
  });
  if (!order) return;

  await prisma.$transaction([
    prisma.order.update({
      where: { id: order.id },
      data: { paymentStatus: "FAILED", status: "ON_HOLD" },
    }),
    prisma.orderEvent.create({
      data: {
        orderId: order.id,
        type: "payment_failed",
        title: "Pagamento fallito",
        message: paymentIntent.last_payment_error?.message ?? "Payment failed",
        actorType: "system",
      },
    }),
  ]);
}

async function handleInvoicePaid(invoice: Stripe.Invoice): Promise<void> {
  console.log(`[Webhook] Invoice ${invoice.id} paid: ${invoice.amount_paid}`);
}

async function handleInvoicePaymentFailed(
  invoice: Stripe.Invoice
): Promise<void> {
  console.log(`[Webhook] Invoice ${invoice.id} payment failed`);

  if (invoice.subscription) {
    const sub = await prisma.orgSubscription.findUnique({
      where: {
        stripeSubscriptionId:
          typeof invoice.subscription === "string"
            ? invoice.subscription
            : invoice.subscription.id,
      },
      select: { organizationId: true },
    });

    if (sub) {
      await prisma.orgAuditLog.create({
        data: {
          organizationId: sub.organizationId,
          action: "subscription.payment_failed",
          entity: "Subscription",
          entityId: typeof invoice.subscription === "string"
            ? invoice.subscription
            : invoice.subscription.id,
          metadata: {
            invoiceId: invoice.id,
            amountDue: invoice.amount_due,
            attemptCount: invoice.attempt_count,
          },
        },
      });
    }
  }
}

async function handleChargeRefunded(charge: Stripe.Charge): Promise<void> {
  const paymentIntentId =
    typeof charge.payment_intent === "string"
      ? charge.payment_intent
      : charge.payment_intent?.id;

  if (!paymentIntentId) return;

  const order = await prisma.order.findFirst({
    where: { paymentIntentId },
  });
  if (!order) return;

  const refundAmount = charge.amount_refunded / 100;
  const isFullRefund = charge.refunded;

  await prisma.$transaction([
    prisma.refund.create({
      data: {
        orderId: order.id,
        amount: refundAmount,
        reason: "customer_request",
        status: "succeeded",
      },
    }),
    prisma.order.update({
      where: { id: order.id },
      data: {
        status: isFullRefund ? "REFUNDED" : order.status,
        paymentStatus: isFullRefund ? "REFUNDED" : "PARTIALLY_REFUNDED",
        refundedAt: isFullRefund ? new Date() : undefined,
      },
    }),
    prisma.orderEvent.create({
      data: {
        orderId: order.id,
        type: isFullRefund ? "full_refund" : "partial_refund",
        title: isFullRefund ? "Rimborso completo" : "Rimborso parziale",
        message: `Rimborsato: EUR ${refundAmount.toFixed(2)}`,
        actorType: "system",
      },
    }),
  ]);
}
```

---

§ 13. STRIPE REFUNDS

§ 13.1 IMPLEMENTAZIONE COMPLETA

```typescript
// app/api/orders/[orderId]/refund/route.ts — Process Refund

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";
import { prisma } from "@/lib/db/client";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

const refundSchema = z.object({
  amount: z.number().positive().optional(),
  reason: z.enum([
    "duplicate",
    "fraudulent",
    "requested_by_customer",
    "product_not_received",
    "product_defective",
  ]),
  notes: z.string().max(1000).optional(),
  restoreStock: z.boolean().default(true),
});

export async function POST(
  request: NextRequest,
  { params }: { params: { orderId: string } }
) {
  try {
    const session = await getServerSession();
    if (!session?.user || session.user.role !== "ADMIN") {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { amount, reason, notes, restoreStock } = refundSchema.parse(body);

    const order = await prisma.order.findUnique({
      where: { id: params.orderId },
      include: {
        payments: { where: { status: "succeeded" }, take: 1 },
        items: true,
        refunds: true,
      },
    });

    if (!order) {
      return NextResponse.json({ error: "Order not found" }, { status: 404 });
    }

    if (!order.paymentIntentId) {
      return NextResponse.json(
        { error: "No payment intent found" },
        { status: 400 }
      );
    }

    // Calculate max refundable amount
    const totalRefunded = order.refunds.reduce(
      (sum, r) => sum + Number(r.amount),
      0
    );
    const maxRefundable = Number(order.total) - totalRefunded;

    const refundAmount = amount ?? maxRefundable;

    if (refundAmount > maxRefundable) {
      return NextResponse.json(
        {
          error: `Maximum refundable amount is EUR ${maxRefundable.toFixed(2)}`,
        },
        { status: 400 }
      );
    }

    // Process refund via Stripe
    const stripeRefund = await stripe.refunds.create({
      payment_intent: order.paymentIntentId,
      amount: Math.round(refundAmount * 100),
      reason:
        reason === "requested_by_customer" || reason === "product_not_received"
          ? "requested_by_customer"
          : reason === "duplicate"
            ? "duplicate"
            : "fraudulent",
      metadata: {
        orderId: order.id,
        orderNumber: order.orderNumber,
        reason,
        notes: notes ?? "",
        processedBy: session.user.id,
      },
    });

    // Record refund in database
    const isFullRefund = refundAmount >= maxRefundable;

    await prisma.$transaction(async (tx) => {
      await tx.refund.create({
        data: {
          orderId: order.id,
          stripeRefundId: stripeRefund.id,
          amount: refundAmount,
          reason,
          status: stripeRefund.status ?? "succeeded",
        },
      });

      await tx.order.update({
        where: { id: order.id },
        data: {
          status: isFullRefund ? "REFUNDED" : order.status,
          paymentStatus: isFullRefund ? "REFUNDED" : "PARTIALLY_REFUNDED",
          refundedAt: isFullRefund ? new Date() : undefined,
          internalNotes: notes
            ? `${order.internalNotes ?? ""}\n[Refund] ${notes}`
            : undefined,
        },
      });

      await tx.orderEvent.create({
        data: {
          orderId: order.id,
          type: isFullRefund ? "full_refund" : "partial_refund",
          title: isFullRefund ? "Rimborso completo" : "Rimborso parziale",
          message: `EUR ${refundAmount.toFixed(2)} rimborsato. Motivo: ${reason}`,
          actorId: session.user.id,
          actorType: "admin",
          metadata: {
            stripeRefundId: stripeRefund.id,
            amount: refundAmount,
            reason,
          },
        },
      });

      // Restore stock if requested
      if (restoreStock && isFullRefund) {
        for (const item of order.items) {
          if (item.variantId) {
            await tx.productVariant.update({
              where: { id: item.variantId },
              data: { stock: { increment: item.quantity } },
            });
          } else {
            await tx.product.update({
              where: { id: item.productId },
              data: { stock: { increment: item.quantity } },
            });
          }
        }
      }
    });

    return NextResponse.json({
      refund: {
        id: stripeRefund.id,
        amount: refundAmount,
        status: stripeRefund.status,
        isFullRefund,
      },
    });
  } catch (error) {
    if (error instanceof Stripe.errors.StripeError) {
      return NextResponse.json(
        { error: error.message },
        { status: error.statusCode ?? 500 }
      );
    }
    console.error("[Refund] Error:", error);
    return NextResponse.json(
      { error: "Failed to process refund" },
      { status: 500 }
    );
  }
}
```

---

§ 14. PAYMENT INTENT FLOW CON 3D SECURE

§ 14.1 PANORAMICA
Flusso PaymentIntent completo con SCA (Strong Customer Authentication),
gestione 3D Secure, conferma lato client e server.

§ 14.2 IMPLEMENTAZIONE SERVER-SIDE

```typescript
// app/api/payments/create-intent/route.ts — Create PaymentIntent

import { NextRequest, NextResponse } from "next/server";
import Stripe from "stripe";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

const createIntentSchema = z.object({
  amount: z.number().positive().max(999999),
  currency: z.string().length(3).default("eur"),
  orderId: z.string().cuid(),
  savePaymentMethod: z.boolean().default(false),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { amount, currency, orderId, savePaymentMethod } =
      createIntentSchema.parse(body);

    // Get or create Stripe customer
    const customers = await stripe.customers.list({
      email: session.user.email,
      limit: 1,
    });

    let customerId: string;
    if (customers.data.length > 0) {
      customerId = customers.data[0].id;
    } else {
      const customer = await stripe.customers.create({
        email: session.user.email,
        metadata: { userId: session.user.id },
      });
      customerId = customer.id;
    }

    const paymentIntent = await stripe.paymentIntents.create({
      amount: Math.round(amount * 100),
      currency,
      customer: customerId,
      metadata: {
        orderId,
        userId: session.user.id,
      },
      automatic_payment_methods: {
        enabled: true,
      },
      setup_future_usage: savePaymentMethod ? "off_session" : undefined,
      receipt_email: session.user.email,
      description: `Order payment for ${orderId}`,
    });

    return NextResponse.json({
      clientSecret: paymentIntent.client_secret,
      paymentIntentId: paymentIntent.id,
    });
  } catch (error) {
    console.error("[PaymentIntent] Error:", error);
    return NextResponse.json(
      { error: "Failed to create payment intent" },
      { status: 500 }
    );
  }
}
```

§ 14.3 IMPLEMENTAZIONE CLIENT-SIDE

```typescript
// components/checkout/PaymentForm.tsx — Stripe Elements Payment Form

"use client";

import { useState } from "react";
import {
  PaymentElement,
  useStripe,
  useElements,
  Elements,
} from "@stripe/react-stripe-js";
import { loadStripe } from "@stripe/stripe-js";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Alert, AlertDescription } from "@/components/ui/alert";
import { Loader2, Lock, AlertCircle } from "lucide-react";

const stripePromise = loadStripe(
  process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!
);

interface PaymentFormProps {
  clientSecret: string;
  orderId: string;
  amount: number;
  currency?: string;
}

function PaymentFormInner({ orderId, amount, currency = "EUR" }: Omit<PaymentFormProps, "clientSecret">) {
  const stripe = useStripe();
  const elements = useElements();
  const [error, setError] = useState<string | null>(null);
  const [processing, setProcessing] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!stripe || !elements) return;

    setProcessing(true);
    setError(null);

    const { error: submitError } = await elements.submit();
    if (submitError) {
      setError(submitError.message ?? "Errore nella validazione del form");
      setProcessing(false);
      return;
    }

    const { error: confirmError } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/checkout/success?order_id=${orderId}`,
      },
    });

    if (confirmError) {
      if (confirmError.type === "card_error" || confirmError.type === "validation_error") {
        setError(confirmError.message ?? "Pagamento rifiutato");
      } else {
        setError("Si e verificato un errore. Riprova.");
      }
      setProcessing(false);
    }
  };

  const formattedAmount = new Intl.NumberFormat("it-IT", {
    style: "currency",
    currency,
  }).format(amount);

  return (
    <form onSubmit={handleSubmit}>
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Lock className="h-5 w-5" />
            Pagamento sicuro
          </CardTitle>
        </CardHeader>
        <CardContent className="space-y-4">
          <PaymentElement
            options={{
              layout: "tabs",
              defaultValues: {
                billingDetails: {
                  address: { country: "IT" },
                },
              },
            }}
          />

          {error && (
            <Alert variant="destructive">
              <AlertCircle className="h-4 w-4" />
              <AlertDescription>{error}</AlertDescription>
            </Alert>
          )}
        </CardContent>
        <CardFooter>
          <Button
            type="submit"
            className="w-full"
            size="lg"
            disabled={!stripe || !elements || processing}
          >
            {processing ? (
              <>
                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                Elaborazione...
              </>
            ) : (
              `Paga ${formattedAmount}`
            )}
          </Button>
        </CardFooter>
      </Card>
    </form>
  );
}

export function PaymentForm({ clientSecret, ...props }: PaymentFormProps) {
  return (
    <Elements
      stripe={stripePromise}
      options={{
        clientSecret,
        appearance: {
          theme: "stripe",
          variables: {
            colorPrimary: "#6366f1",
            colorBackground: "#ffffff",
            colorText: "#1f2937",
            colorDanger: "#ef4444",
            fontFamily: "Inter, system-ui, sans-serif",
            spacingUnit: "4px",
            borderRadius: "8px",
          },
          rules: {
            ".Input": {
              border: "1px solid #e5e7eb",
              boxShadow: "0 1px 2px 0 rgb(0 0 0 / 0.05)",
            },
            ".Input:focus": {
              border: "1px solid #6366f1",
              boxShadow: "0 0 0 1px #6366f1",
            },
          },
        },
        locale: "it",
      }}
    >
      <PaymentFormInner {...props} />
    </Elements>
  );
}
```

---

§ 15. IDEMPOTENCY E RETRY LOGIC

§ 15.1 PANORAMICA
Implementazione idempotency keys per prevenire pagamenti duplicati,
retry logic con exponential backoff per failed payments, e dunning management.

§ 15.2 IMPLEMENTAZIONE COMPLETA

```typescript
// lib/stripe/idempotency.ts — Idempotency and retry utilities

import Stripe from "stripe";
import crypto from "crypto";
import { prisma } from "@/lib/db/client";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

// Generate deterministic idempotency key
export function generateIdempotencyKey(
  userId: string,
  orderId: string,
  action: string
): string {
  const data = `${userId}:${orderId}:${action}:${new Date().toISOString().slice(0, 10)}`;
  return crypto.createHash("sha256").update(data).digest("hex").substring(0, 32);
}

// Create payment with idempotency
export async function createPaymentSafe(params: {
  amount: number;
  currency: string;
  customerId: string;
  orderId: string;
  userId: string;
  paymentMethodId: string;
}): Promise<{
  success: boolean;
  paymentIntent?: Stripe.PaymentIntent;
  error?: string;
}> {
  const idempotencyKey = generateIdempotencyKey(
    params.userId,
    params.orderId,
    "payment"
  );

  try {
    const paymentIntent = await stripe.paymentIntents.create(
      {
        amount: Math.round(params.amount * 100),
        currency: params.currency,
        customer: params.customerId,
        payment_method: params.paymentMethodId,
        confirm: true,
        metadata: {
          orderId: params.orderId,
          userId: params.userId,
        },
        automatic_payment_methods: {
          enabled: true,
          allow_redirects: "never",
        },
      },
      { idempotencyKey }
    );

    return { success: true, paymentIntent };
  } catch (error) {
    if (error instanceof Stripe.errors.StripeError) {
      if (error.type === "StripeIdempotencyError") {
        console.warn("[Payment] Duplicate request detected");
        return { success: false, error: "Duplicate payment request" };
      }
      return { success: false, error: error.message };
    }
    return { success: false, error: "Unknown error" };
  }
}

// Retry failed payment with exponential backoff
export async function retryPayment(
  paymentIntentId: string,
  maxRetries = 3
): Promise<{ success: boolean; attempt: number; error?: string }> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const paymentIntent = await stripe.paymentIntents.confirm(
        paymentIntentId
      );

      if (paymentIntent.status === "succeeded") {
        return { success: true, attempt };
      }

      if (paymentIntent.status === "requires_action") {
        return {
          success: false,
          attempt,
          error: "Customer authentication required",
        };
      }
    } catch (error) {
      if (error instanceof Stripe.errors.StripeError) {
        if (error.code === "card_declined" || error.code === "expired_card") {
          return { success: false, attempt, error: error.message };
        }
      }

      if (attempt < maxRetries) {
        const delay = Math.pow(2, attempt) * 1000;
        console.log(
          `[Retry] Attempt ${attempt}/${maxRetries} failed, retrying in ${delay}ms`
        );
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  return { success: false, attempt: maxRetries, error: "Max retries exceeded" };
}

// Dunning management: schedule payment retries for failed subscriptions
export async function processDunning(): Promise<{
  processed: number;
  recovered: number;
  failed: number;
}> {
  let processed = 0;
  let recovered = 0;
  let failed = 0;

  // Find subscriptions with past_due status
  const pastDueSubs = await prisma.orgSubscription.findMany({
    where: { status: "PAST_DUE" },
    include: {
      organization: {
        select: { id: true, stripeCustomerId: true, billingEmail: true },
      },
    },
  });

  for (const sub of pastDueSubs) {
    processed++;

    try {
      // Get latest invoice
      const invoices = await stripe.invoices.list({
        subscription: sub.stripeSubscriptionId,
        status: "open",
        limit: 1,
      });

      if (invoices.data.length === 0) continue;

      const invoice = invoices.data[0];

      // Attempt to pay invoice
      if (invoice.attempt_count < 4) {
        await stripe.invoices.pay(invoice.id);
        recovered++;

        await prisma.orgAuditLog.create({
          data: {
            organizationId: sub.organizationId,
            action: "dunning.payment_recovered",
            entity: "Subscription",
            entityId: sub.stripeSubscriptionId,
            metadata: {
              invoiceId: invoice.id,
              attemptCount: invoice.attempt_count + 1,
            },
          },
        });
      } else {
        failed++;

        // Cancel subscription after too many failures
        await stripe.subscriptions.cancel(sub.stripeSubscriptionId);

        await prisma.orgAuditLog.create({
          data: {
            organizationId: sub.organizationId,
            action: "dunning.subscription_cancelled",
            entity: "Subscription",
            entityId: sub.stripeSubscriptionId,
            metadata: {
              reason: "max_retry_exceeded",
              totalAttempts: invoice.attempt_count,
            },
          },
        });
      }
    } catch (error) {
      console.error(
        `[Dunning] Failed for subscription ${sub.stripeSubscriptionId}:`,
        error
      );
      failed++;
    }
  }

  return { processed, recovered, failed };
}
```

---

§ 16. PRICING PAGE COMPONENT

§ 16.1 IMPLEMENTAZIONE COMPLETA

```typescript
// components/pricing/PricingPage.tsx — Complete Pricing Table

"use client";

import { useState, useTransition } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Switch } from "@/components/ui/switch";
import { Label } from "@/components/ui/label";
import { Check, Loader2, Sparkles } from "lucide-react";
import { cn } from "@/lib/utils";
import { useRouter } from "next/navigation";

interface Plan {
  id: string;
  name: string;
  description: string;
  monthlyPrice: number;
  yearlyPrice: number;
  features: string[];
  recommended?: boolean;
  badge?: string;
}

const PLANS: Plan[] = [
  {
    id: "starter",
    name: "Starter",
    description: "Per freelancer e piccoli team",
    monthlyPrice: 19,
    yearlyPrice: 190,
    features: [
      "Fino a 5 utenti",
      "5 GB storage",
      "Email support",
      "API access base",
      "1 workspace",
    ],
  },
  {
    id: "pro",
    name: "Pro",
    description: "Per team in crescita",
    monthlyPrice: 49,
    yearlyPrice: 490,
    features: [
      "Fino a 20 utenti",
      "50 GB storage",
      "Priority support",
      "API access completo",
      "20 workspace",
      "Webhooks",
      "Custom domain",
      "Analytics avanzate",
    ],
    recommended: true,
    badge: "Piu popolare",
  },
  {
    id: "business",
    name: "Business",
    description: "Per aziende enterprise",
    monthlyPrice: 99,
    yearlyPrice: 990,
    features: [
      "Utenti illimitati",
      "500 GB storage",
      "24/7 dedicated support",
      "API illimitato",
      "Workspace illimitati",
      "SSO / SAML",
      "SLA 99.9%",
      "Audit log completo",
      "Custom integrations",
      "Account manager dedicato",
    ],
  },
];

interface PricingPageProps {
  currentPlanId?: string;
  organizationId?: string;
}

export function PricingPage({ currentPlanId, organizationId }: PricingPageProps) {
  const [isYearly, setIsYearly] = useState(true);
  const [isPending, startTransition] = useTransition();
  const [loadingPlan, setLoadingPlan] = useState<string | null>(null);
  const router = useRouter();

  const handleSelectPlan = async (planId: string) => {
    setLoadingPlan(planId);

    try {
      const res = await fetch("/api/subscriptions/checkout", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          planId,
          interval: isYearly ? "yearly" : "monthly",
          organizationId,
        }),
      });

      const data = await res.json();

      if (data.url) {
        window.location.href = data.url;
      }
    } catch (error) {
      console.error("Checkout failed:", error);
    } finally {
      setLoadingPlan(null);
    }
  };

  const yearlySavings = (plan: Plan) => {
    const monthlyTotal = plan.monthlyPrice * 12;
    const savings = monthlyTotal - plan.yearlyPrice;
    const percent = Math.round((savings / monthlyTotal) * 100);
    return { savings, percent };
  };

  return (
    <div className="py-16 px-4">
      <div className="text-center max-w-3xl mx-auto mb-12">
        <h1 className="text-4xl font-bold tracking-tight mb-4">
          Prezzi semplici e trasparenti
        </h1>
        <p className="text-xl text-muted-foreground mb-8">
          Scegli il piano adatto al tuo team. Upgrade, downgrade o cancella in
          qualsiasi momento.
        </p>

        <div className="flex items-center justify-center gap-3">
          <Label htmlFor="billing-toggle" className={cn(!isYearly && "font-bold")}>
            Mensile
          </Label>
          <Switch
            id="billing-toggle"
            checked={isYearly}
            onCheckedChange={setIsYearly}
          />
          <Label htmlFor="billing-toggle" className={cn(isYearly && "font-bold")}>
            Annuale
          </Label>
          {isYearly && (
            <Badge variant="secondary" className="ml-2">
              Risparmia fino al 17%
            </Badge>
          )}
        </div>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-6xl mx-auto">
        {PLANS.map((plan) => {
          const price = isYearly ? plan.yearlyPrice : plan.monthlyPrice;
          const displayPrice = isYearly
            ? Math.round(plan.yearlyPrice / 12)
            : plan.monthlyPrice;
          const { savings, percent } = yearlySavings(plan);
          const isCurrent = currentPlanId === plan.id;

          return (
            <Card
              key={plan.id}
              className={cn(
                "relative flex flex-col",
                plan.recommended &&
                  "border-primary shadow-lg scale-105 z-10"
              )}
            >
              {plan.badge && (
                <div className="absolute -top-3 left-1/2 -translate-x-1/2">
                  <Badge className="px-3 py-1">
                    <Sparkles className="w-3 h-3 mr-1" />
                    {plan.badge}
                  </Badge>
                </div>
              )}

              <CardHeader className="text-center pb-2">
                <CardTitle className="text-2xl">{plan.name}</CardTitle>
                <CardDescription>{plan.description}</CardDescription>
              </CardHeader>

              <CardContent className="flex-1">
                <div className="text-center mb-6">
                  <div className="flex items-baseline justify-center gap-1">
                    <span className="text-4xl font-bold">
                      EUR {displayPrice}
                    </span>
                    <span className="text-muted-foreground">/mese</span>
                  </div>
                  {isYearly && (
                    <p className="text-sm text-green-600 mt-1">
                      EUR {plan.yearlyPrice}/anno (risparmi EUR {savings})
                    </p>
                  )}
                </div>

                <ul className="space-y-3">
                  {plan.features.map((feature) => (
                    <li key={feature} className="flex items-start gap-2">
                      <Check className="h-5 w-5 text-green-500 shrink-0 mt-0.5" />
                      <span className="text-sm">{feature}</span>
                    </li>
                  ))}
                </ul>
              </CardContent>

              <CardFooter>
                <Button
                  className="w-full"
                  size="lg"
                  variant={plan.recommended ? "default" : "outline"}
                  disabled={isCurrent || loadingPlan !== null}
                  onClick={() => handleSelectPlan(plan.id)}
                >
                  {loadingPlan === plan.id ? (
                    <Loader2 className="mr-2 h-4 w-4 animate-spin" />
                  ) : null}
                  {isCurrent ? "Piano attuale" : "Inizia ora"}
                </Button>
              </CardFooter>
            </Card>
          );
        })}
      </div>

      <p className="text-center text-sm text-muted-foreground mt-8">
        14 giorni di prova gratuita su tutti i piani. Nessuna carta di credito
        richiesta.
      </p>
    </div>
  );
}
```

### Errori Comuni da Evitare
- Non processare lo stesso webhook event due volte — implementare idempotency
- Non salvare mai il secret key di Stripe nel client-side code
- Non dimenticare di verificare la firma del webhook con `constructEvent()`
- Non creare PaymentIntent senza idempotency key per ordini
- Non dimenticare di gestire `requires_action` status per 3D Secure
- Non processare refund senza verificare il massimo rimborsabile

### Checklist di Verifica
- [ ] Webhook signature verification attiva
- [ ] Idempotency keys su ogni creazione PaymentIntent
- [ ] Event processing idempotente (tracked via DB)
- [ ] Tutti gli stati del PaymentIntent gestiti (succeeded, failed, requires_action)
- [ ] Subscription lifecycle completo (create, update, cancel, dunning)
- [ ] Refund parziale e totale implementati
- [ ] Pricing page con toggle monthly/yearly
- [ ] 3D Secure / SCA handling completo
- [ ] Stripe Elements configurato con tema custom
- [ ] Error handling Stripe-specific con messaggi user-friendly



---

§ 17. PAYPAL INTEGRATION

§ 17.1 PANORAMICA
Integrazione completa PayPal Checkout con Orders API v2, capture payments,
gestione webhook e componente React per il pulsante PayPal.

§ 17.2 SERVER-SIDE — CREATE ORDER

```typescript
// app/api/paypal/create-order/route.ts — PayPal Create Order

import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";
import { prisma } from "@/lib/db/client";

const PAYPAL_BASE_URL =
  process.env.NODE_ENV === "production"
    ? "https://api-m.paypal.com"
    : "https://api-m.sandbox.paypal.com";

async function getPayPalAccessToken(): Promise<string> {
  const clientId = process.env.PAYPAL_CLIENT_ID!;
  const clientSecret = process.env.PAYPAL_CLIENT_SECRET!;
  const auth = Buffer.from(`${clientId}:${clientSecret}`).toString("base64");

  const response = await fetch(`${PAYPAL_BASE_URL}/v1/oauth2/token`, {
    method: "POST",
    headers: {
      Authorization: `Basic ${auth}`,
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: "grant_type=client_credentials",
  });

  if (!response.ok) {
    throw new Error(`PayPal auth failed: ${response.status}`);
  }

  const data = await response.json();
  return data.access_token;
}

const createOrderSchema = z.object({
  orderId: z.string().cuid(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { orderId } = createOrderSchema.parse(body);

    const order = await prisma.order.findUnique({
      where: { id: orderId, userId: session.user.id },
      include: {
        items: true,
        shippingAddress: true,
      },
    });

    if (!order) {
      return NextResponse.json({ error: "Order not found" }, { status: 404 });
    }

    if (order.paymentStatus !== "PENDING") {
      return NextResponse.json(
        { error: "Order already paid" },
        { status: 400 }
      );
    }

    const accessToken = await getPayPalAccessToken();

    const paypalItems = order.items.map((item) => ({
      name: item.name.substring(0, 127),
      description: `SKU: ${item.sku}`,
      sku: item.sku,
      unit_amount: {
        currency_code: order.currency.toUpperCase(),
        value: Number(item.price).toFixed(2),
      },
      quantity: item.quantity.toString(),
      category: "PHYSICAL_GOODS" as const,
    }));

    const paypalOrderPayload = {
      intent: "CAPTURE",
      purchase_units: [
        {
          reference_id: order.id,
          description: `Ordine ${order.orderNumber}`,
          custom_id: order.id,
          invoice_id: order.orderNumber,
          amount: {
            currency_code: order.currency.toUpperCase(),
            value: Number(order.total).toFixed(2),
            breakdown: {
              item_total: {
                currency_code: order.currency.toUpperCase(),
                value: Number(order.subtotal).toFixed(2),
              },
              tax_total: {
                currency_code: order.currency.toUpperCase(),
                value: Number(order.tax).toFixed(2),
              },
              shipping: {
                currency_code: order.currency.toUpperCase(),
                value: Number(order.shippingCost).toFixed(2),
              },
              discount: {
                currency_code: order.currency.toUpperCase(),
                value: Number(order.discount).toFixed(2),
              },
            },
          },
          items: paypalItems,
          shipping: order.shippingAddress
            ? {
                name: {
                  full_name: `${order.shippingAddress.firstName} ${order.shippingAddress.lastName}`,
                },
                address: {
                  address_line_1: order.shippingAddress.street,
                  admin_area_2: order.shippingAddress.city,
                  admin_area_1: order.shippingAddress.state,
                  postal_code: order.shippingAddress.zipCode,
                  country_code: order.shippingAddress.country,
                },
              }
            : undefined,
        },
      ],
      application_context: {
        brand_name: "Webby Store",
        landing_page: "NO_PREFERENCE",
        user_action: "PAY_NOW",
        return_url: `${process.env.NEXT_PUBLIC_APP_URL}/checkout/paypal-return`,
        cancel_url: `${process.env.NEXT_PUBLIC_APP_URL}/checkout/cancel`,
        locale: "it-IT",
      },
    };

    const response = await fetch(`${PAYPAL_BASE_URL}/v2/checkout/orders`, {
      method: "POST",
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "Content-Type": "application/json",
        "PayPal-Request-Id": `order-${order.id}-${Date.now()}`,
      },
      body: JSON.stringify(paypalOrderPayload),
    });

    if (!response.ok) {
      const errorBody = await response.json();
      console.error("[PayPal] Create order failed:", errorBody);
      return NextResponse.json(
        { error: "Failed to create PayPal order" },
        { status: 500 }
      );
    }

    const paypalOrder = await response.json();

    // Store PayPal order ID in our order
    await prisma.order.update({
      where: { id: orderId },
      data: { paymentIntentId: paypalOrder.id, paymentMethod: "paypal" },
    });

    const approveUrl = paypalOrder.links.find(
      (link: any) => link.rel === "approve"
    )?.href;

    return NextResponse.json({
      paypalOrderId: paypalOrder.id,
      approveUrl,
    });
  } catch (error) {
    console.error("[PayPal] Error:", error);
    return NextResponse.json(
      { error: "Failed to create PayPal order" },
      { status: 500 }
    );
  }
}
```

§ 17.3 CAPTURE PAYMENT

```typescript
// app/api/paypal/capture/route.ts — PayPal Capture Payment

import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";
import { getServerSession } from "@/lib/auth";
import { prisma } from "@/lib/db/client";

const PAYPAL_BASE_URL =
  process.env.NODE_ENV === "production"
    ? "https://api-m.paypal.com"
    : "https://api-m.sandbox.paypal.com";

async function getPayPalAccessToken(): Promise<string> {
  const clientId = process.env.PAYPAL_CLIENT_ID!;
  const clientSecret = process.env.PAYPAL_CLIENT_SECRET!;
  const auth = Buffer.from(`${clientId}:${clientSecret}`).toString("base64");

  const response = await fetch(`${PAYPAL_BASE_URL}/v1/oauth2/token`, {
    method: "POST",
    headers: {
      Authorization: `Basic ${auth}`,
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: "grant_type=client_credentials",
  });

  const data = await response.json();
  return data.access_token;
}

const captureSchema = z.object({
  paypalOrderId: z.string(),
});

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession();
    if (!session?.user) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const body = await request.json();
    const { paypalOrderId } = captureSchema.parse(body);

    const accessToken = await getPayPalAccessToken();

    const captureResponse = await fetch(
      `${PAYPAL_BASE_URL}/v2/checkout/orders/${paypalOrderId}/capture`,
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${accessToken}`,
          "Content-Type": "application/json",
        },
      }
    );

    if (!captureResponse.ok) {
      const errorBody = await captureResponse.json();
      console.error("[PayPal] Capture failed:", errorBody);
      return NextResponse.json(
        { error: "Failed to capture payment" },
        { status: 500 }
      );
    }

    const captureData = await captureResponse.json();

    if (captureData.status === "COMPLETED") {
      const orderId = captureData.purchase_units[0]?.custom_id;
      if (!orderId) {
        return NextResponse.json(
          { error: "Order ID not found in PayPal response" },
          { status: 500 }
        );
      }

      const captureAmount =
        captureData.purchase_units[0]?.payments?.captures?.[0];

      await prisma.$transaction([
        prisma.order.update({
          where: { id: orderId },
          data: {
            status: "CONFIRMED",
            paymentStatus: "CAPTURED",
          },
        }),
        prisma.payment.create({
          data: {
            orderId,
            stripePaymentId: paypalOrderId,
            amount: parseFloat(captureAmount?.amount?.value ?? "0"),
            currency: captureAmount?.amount?.currency_code?.toLowerCase() ?? "eur",
            status: "succeeded",
            method: "paypal",
            metadata: {
              paypalOrderId,
              captureId: captureAmount?.id,
              payerEmail:
                captureData.payer?.email_address,
            },
          },
        }),
        prisma.orderEvent.create({
          data: {
            orderId,
            type: "payment_captured",
            title: "Pagamento PayPal ricevuto",
            message: `Pagamento di ${captureAmount?.amount?.value} ${captureAmount?.amount?.currency_code} ricevuto via PayPal`,
            actorType: "system",
          },
        }),
      ]);

      return NextResponse.json({
        status: "COMPLETED",
        orderId,
        captureId: captureAmount?.id,
      });
    }

    return NextResponse.json({
      status: captureData.status,
      details: captureData,
    });
  } catch (error) {
    console.error("[PayPal Capture] Error:", error);
    return NextResponse.json(
      { error: "Failed to capture payment" },
      { status: 500 }
    );
  }
}
```

§ 17.4 PAYPAL BUTTON COMPONENT

```typescript
// components/checkout/PayPalButton.tsx — PayPal Checkout Button

"use client";

import { useState, useEffect, useRef } from "react";
import { Button } from "@/components/ui/button";
import { Alert, AlertDescription } from "@/components/ui/alert";
import { Loader2, AlertCircle } from "lucide-react";
import { useRouter } from "next/navigation";

declare global {
  interface Window {
    paypal?: any;
  }
}

interface PayPalButtonProps {
  orderId: string;
  amount: number;
  currency?: string;
  onSuccess?: (details: any) => void;
  onError?: (error: any) => void;
}

export function PayPalButton({
  orderId,
  amount,
  currency = "EUR",
  onSuccess,
  onError,
}: PayPalButtonProps) {
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [sdkReady, setSdkReady] = useState(false);
  const paypalRef = useRef<HTMLDivElement>(null);
  const router = useRouter();

  // Load PayPal SDK
  useEffect(() => {
    const existingScript = document.getElementById("paypal-sdk");
    if (existingScript) {
      setSdkReady(true);
      setLoading(false);
      return;
    }

    const script = document.createElement("script");
    script.id = "paypal-sdk";
    script.src = `https://www.paypal.com/sdk/js?client-id=${process.env.NEXT_PUBLIC_PAYPAL_CLIENT_ID}&currency=${currency}&locale=it_IT`;
    script.async = true;

    script.onload = () => {
      setSdkReady(true);
      setLoading(false);
    };

    script.onerror = () => {
      setError("Impossibile caricare PayPal. Riprova piu tardi.");
      setLoading(false);
    };

    document.body.appendChild(script);

    return () => {
      const el = document.getElementById("paypal-sdk");
      if (el) el.remove();
    };
  }, [currency]);

  // Render PayPal buttons
  useEffect(() => {
    if (!sdkReady || !window.paypal || !paypalRef.current) return;

    // Clear previous buttons
    paypalRef.current.innerHTML = "";

    window.paypal
      .Buttons({
        style: {
          layout: "vertical",
          color: "gold",
          shape: "rect",
          label: "paypal",
          height: 48,
        },

        createOrder: async () => {
          try {
            const res = await fetch("/api/paypal/create-order", {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ orderId }),
            });

            const data = await res.json();
            if (data.error) throw new Error(data.error);
            return data.paypalOrderId;
          } catch (err: any) {
            setError(err.message ?? "Errore nella creazione dell'ordine PayPal");
            throw err;
          }
        },

        onApprove: async (data: any) => {
          try {
            const res = await fetch("/api/paypal/capture", {
              method: "POST",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({ paypalOrderId: data.orderID }),
            });

            const captureData = await res.json();

            if (captureData.status === "COMPLETED") {
              onSuccess?.(captureData);
              router.push(
                `/checkout/success?order_id=${captureData.orderId}`
              );
            } else {
              setError("Pagamento non completato. Riprova.");
              onError?.(captureData);
            }
          } catch (err: any) {
            setError(err.message ?? "Errore nella cattura del pagamento");
            onError?.(err);
          }
        },

        onCancel: () => {
          setError("Pagamento annullato.");
        },

        onError: (err: any) => {
          console.error("[PayPal Button] Error:", err);
          setError("Si e verificato un errore con PayPal. Riprova.");
          onError?.(err);
        },
      })
      .render(paypalRef.current);
  }, [sdkReady, orderId, onSuccess, onError, router]);

  if (loading) {
    return (
      <Button disabled className="w-full" size="lg">
        <Loader2 className="mr-2 h-4 w-4 animate-spin" />
        Caricamento PayPal...
      </Button>
    );
  }

  return (
    <div className="space-y-3">
      {error && (
        <Alert variant="destructive">
          <AlertCircle className="h-4 w-4" />
          <AlertDescription>{error}</AlertDescription>
        </Alert>
      )}
      <div ref={paypalRef} className="min-h-[48px]" />
    </div>
  );
}
```

---

§ 18. PAYPAL WEBHOOK HANDLER

§ 18.1 IMPLEMENTAZIONE COMPLETA

```typescript
// app/api/webhooks/paypal/route.ts — PayPal Webhook Handler

import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/db/client";
import crypto from "crypto";

const PAYPAL_BASE_URL =
  process.env.NODE_ENV === "production"
    ? "https://api-m.paypal.com"
    : "https://api-m.sandbox.paypal.com";

const PAYPAL_WEBHOOK_ID = process.env.PAYPAL_WEBHOOK_ID!;

// Verify PayPal webhook signature
async function verifyWebhookSignature(
  headers: Record<string, string>,
  body: string
): Promise<boolean> {
  const clientId = process.env.PAYPAL_CLIENT_ID!;
  const clientSecret = process.env.PAYPAL_CLIENT_SECRET!;
  const auth = Buffer.from(`${clientId}:${clientSecret}`).toString("base64");

  const tokenRes = await fetch(`${PAYPAL_BASE_URL}/v1/oauth2/token`, {
    method: "POST",
    headers: {
      Authorization: `Basic ${auth}`,
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: "grant_type=client_credentials",
  });
  const tokenData = await tokenRes.json();

  const verifyRes = await fetch(
    `${PAYPAL_BASE_URL}/v1/notifications/verify-webhook-signature`,
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${tokenData.access_token}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        auth_algo: headers["paypal-auth-algo"],
        cert_url: headers["paypal-cert-url"],
        transmission_id: headers["paypal-transmission-id"],
        transmission_sig: headers["paypal-transmission-sig"],
        transmission_time: headers["paypal-transmission-time"],
        webhook_id: PAYPAL_WEBHOOK_ID,
        webhook_event: JSON.parse(body),
      }),
    }
  );

  const verifyData = await verifyRes.json();
  return verifyData.verification_status === "SUCCESS";
}

export async function POST(request: NextRequest) {
  const body = await request.text();
  const headers: Record<string, string> = {};
  request.headers.forEach((value, key) => {
    headers[key] = value;
  });

  // Verify signature
  const isValid = await verifyWebhookSignature(headers, body);
  if (!isValid) {
    console.error("[PayPal Webhook] Invalid signature");
    return NextResponse.json({ error: "Invalid signature" }, { status: 401 });
  }

  const event = JSON.parse(body);
  console.log(`[PayPal Webhook] Event: ${event.event_type}`);

  try {
    switch (event.event_type) {
      case "PAYMENT.CAPTURE.COMPLETED": {
        const capture = event.resource;
        const customId = capture.custom_id;

        if (customId) {
          await prisma.$transaction([
            prisma.order.update({
              where: { id: customId },
              data: { paymentStatus: "CAPTURED", status: "CONFIRMED" },
            }),
            prisma.orderEvent.create({
              data: {
                orderId: customId,
                type: "paypal_capture_completed",
                title: "Pagamento PayPal confermato",
                message: `Cattura ${capture.id} completata: ${capture.amount.value} ${capture.amount.currency_code}`,
                actorType: "system",
              },
            }),
          ]);
        }
        break;
      }

      case "PAYMENT.CAPTURE.DENIED": {
        const capture = event.resource;
        const customId = capture.custom_id;

        if (customId) {
          await prisma.order.update({
            where: { id: customId },
            data: { paymentStatus: "FAILED", status: "ON_HOLD" },
          });
        }
        break;
      }

      case "PAYMENT.CAPTURE.REFUNDED": {
        const refund = event.resource;
        const captureId = refund.links?.find(
          (l: any) => l.rel === "up"
        )?.href;

        console.log(`[PayPal Webhook] Refund processed: ${refund.id}`);

        // Find order by PayPal capture and update
        const payment = await prisma.payment.findFirst({
          where: {
            method: "paypal",
            metadata: { path: ["captureId"], equals: captureId },
          },
        });

        if (payment) {
          await prisma.refund.create({
            data: {
              orderId: payment.orderId,
              amount: parseFloat(refund.amount.value),
              reason: "paypal_refund",
              status: "succeeded",
            },
          });
        }
        break;
      }

      case "CHECKOUT.ORDER.APPROVED": {
        console.log("[PayPal Webhook] Order approved, awaiting capture");
        break;
      }

      default:
        console.log(`[PayPal Webhook] Unhandled event: ${event.event_type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error) {
    console.error("[PayPal Webhook] Processing error:", error);
    return NextResponse.json(
      { error: "Processing failed" },
      { status: 500 }
    );
  }
}
```

---

§ 19. STRIPE INVOICING

§ 19.1 PANORAMICA
Generazione e gestione fatture Stripe per subscription e one-time payments,
con PDF download e invio automatico.

§ 19.2 IMPLEMENTAZIONE COMPLETA

```typescript
// lib/stripe/invoicing.ts — Invoice management

import Stripe from "stripe";
import { prisma } from "@/lib/db/client";

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: "2024-04-10",
});

interface InvoiceListResult {
  invoices: Array<{
    id: string;
    number: string | null;
    status: string | null;
    amountDue: number;
    amountPaid: number;
    currency: string;
    createdAt: Date;
    dueDate: Date | null;
    pdfUrl: string | null;
    hostedUrl: string | null;
    description: string | null;
  }>;
  hasMore: boolean;
}

export async function listInvoices(
  customerId: string,
  options: { limit?: number; startingAfter?: string } = {}
): Promise<InvoiceListResult> {
  const { limit = 10, startingAfter } = options;

  const invoices = await stripe.invoices.list({
    customer: customerId,
    limit: limit + 1,
    starting_after: startingAfter,
    expand: ["data.subscription"],
  });

  const hasMore = invoices.data.length > limit;
  const data = hasMore ? invoices.data.slice(0, limit) : invoices.data;

  return {
    invoices: data.map((inv) => ({
      id: inv.id,
      number: inv.number,
      status: inv.status,
      amountDue: inv.amount_due / 100,
      amountPaid: inv.amount_paid / 100,
      currency: inv.currency,
      createdAt: new Date(inv.created * 1000),
      dueDate: inv.due_date ? new Date(inv.due_date * 1000) : null,
      pdfUrl: inv.invoice_pdf,
      hostedUrl: inv.hosted_invoice_url,
      description: inv.description,
    })),
    hasMore,
  };
}

// Create one-off invoice for custom charges
export async function createCustomInvoice(params: {
  customerId: string;
  items: Array<{
    description: string;
    amount: number;
    quantity?: number;
  }>;
  dueDate?: Date;
  memo?: string;
  autoAdvance?: boolean;
}): Promise<{ invoiceId: string; invoiceUrl: string | null }> {
  const { customerId, items, dueDate, memo, autoAdvance = true } = params;

  const invoice = await stripe.invoices.create({
    customer: customerId,
    collection_method: "send_invoice",
    due_date: dueDate
      ? Math.floor(dueDate.getTime() / 1000)
      : Math.floor(Date.now() / 1000) + 30 * 24 * 60 * 60,
    description: memo,
    auto_advance: autoAdvance,
  });

  for (const item of items) {
    await stripe.invoiceItems.create({
      customer: customerId,
      invoice: invoice.id,
      description: item.description,
      unit_amount: Math.round(item.amount * 100),
      quantity: item.quantity ?? 1,
      currency: "eur",
    });
  }

  // Finalize and send
  const finalizedInvoice = await stripe.invoices.finalizeInvoice(invoice.id);

  if (autoAdvance) {
    await stripe.invoices.sendInvoice(invoice.id);
  }

  return {
    invoiceId: finalizedInvoice.id,
    invoiceUrl: finalizedInvoice.hosted_invoice_url,
  };
}

// Void an open invoice
export async function voidInvoice(
  invoiceId: string
): Promise<{ success: boolean }> {
  try {
    await stripe.invoices.voidInvoice(invoiceId);
    return { success: true };
  } catch (error) {
    console.error("[Invoice] Void failed:", error);
    return { success: false };
  }
}

// Get upcoming invoice preview
export async function getUpcomingInvoice(
  customerId: string,
  subscriptionId?: string
): Promise<{
  amountDue: number;
  currency: string;
  periodStart: Date;
  periodEnd: Date;
  lines: Array<{
    description: string | null;
    amount: number;
    quantity: number | null;
  }>;
} | null> {
  try {
    const invoice = await stripe.invoices.retrieveUpcoming({
      customer: customerId,
      subscription: subscriptionId,
    });

    return {
      amountDue: invoice.amount_due / 100,
      currency: invoice.currency,
      periodStart: new Date(invoice.period_start * 1000),
      periodEnd: new Date(invoice.period_end * 1000),
      lines: invoice.lines.data.map((line) => ({
        description: line.description,
        amount: line.amount / 100,
        quantity: line.quantity,
      })),
    };
  } catch {
    return null;
  }
}
```

§ 19.3 INVOICE LIST UI COMPONENT

```typescript
// components/billing/InvoiceList.tsx — Invoice History Component

"use client";

import { useState, useEffect } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Download, ExternalLink, FileText, Loader2 } from "lucide-react";

interface Invoice {
  id: string;
  number: string | null;
  status: string | null;
  amountDue: number;
  amountPaid: number;
  currency: string;
  createdAt: string;
  dueDate: string | null;
  pdfUrl: string | null;
  hostedUrl: string | null;
}

interface InvoiceListProps {
  organizationId: string;
}

export function InvoiceList({ organizationId }: InvoiceListProps) {
  const [invoices, setInvoices] = useState<Invoice[]>([]);
  const [loading, setLoading] = useState(true);
  const [hasMore, setHasMore] = useState(false);
  const [loadingMore, setLoadingMore] = useState(false);

  useEffect(() => {
    fetchInvoices();
  }, [organizationId]);

  async function fetchInvoices(startingAfter?: string) {
    const isLoadMore = !!startingAfter;
    if (isLoadMore) setLoadingMore(true);
    else setLoading(true);

    try {
      const params = new URLSearchParams({ organizationId });
      if (startingAfter) params.set("startingAfter", startingAfter);

      const res = await fetch(`/api/billing/invoices?${params}`);
      const data = await res.json();

      if (isLoadMore) {
        setInvoices((prev) => [...prev, ...data.invoices]);
      } else {
        setInvoices(data.invoices);
      }
      setHasMore(data.hasMore);
    } catch (error) {
      console.error("Failed to fetch invoices:", error);
    } finally {
      setLoading(false);
      setLoadingMore(false);
    }
  }

  const statusBadge = (status: string | null) => {
    switch (status) {
      case "paid":
        return <Badge variant="default">Pagata</Badge>;
      case "open":
        return <Badge variant="secondary">In attesa</Badge>;
      case "draft":
        return <Badge variant="outline">Bozza</Badge>;
      case "void":
        return <Badge variant="destructive">Annullata</Badge>;
      case "uncollectible":
        return <Badge variant="destructive">Non riscuotibile</Badge>;
      default:
        return <Badge variant="outline">{status ?? "N/A"}</Badge>;
    }
  };

  const formatAmount = (amount: number, currency: string) =>
    new Intl.NumberFormat("it-IT", {
      style: "currency",
      currency,
    }).format(amount);

  if (loading) {
    return (
      <Card>
        <CardContent className="flex items-center justify-center py-12">
          <Loader2 className="h-6 w-6 animate-spin text-muted-foreground" />
        </CardContent>
      </Card>
    );
  }

  if (invoices.length === 0) {
    return (
      <Card>
        <CardContent className="flex flex-col items-center justify-center py-12 text-center">
          <FileText className="h-12 w-12 text-muted-foreground mb-4" />
          <p className="text-lg font-medium">Nessuna fattura</p>
          <p className="text-sm text-muted-foreground">
            Le fatture appariranno qui dopo il primo pagamento.
          </p>
        </CardContent>
      </Card>
    );
  }

  return (
    <Card>
      <CardHeader>
        <CardTitle>Storico Fatture</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b text-left">
                <th className="p-3 font-medium">Fattura</th>
                <th className="p-3 font-medium">Data</th>
                <th className="p-3 font-medium">Importo</th>
                <th className="p-3 font-medium">Stato</th>
                <th className="p-3 font-medium text-right">Azioni</th>
              </tr>
            </thead>
            <tbody>
              {invoices.map((inv) => (
                <tr key={inv.id} className="border-b hover:bg-muted/50">
                  <td className="p-3 font-mono text-xs">
                    {inv.number ?? inv.id.slice(0, 12)}
                  </td>
                  <td className="p-3">
                    {new Date(inv.createdAt).toLocaleDateString("it-IT")}
                  </td>
                  <td className="p-3 font-semibold">
                    {formatAmount(inv.amountDue, inv.currency)}
                  </td>
                  <td className="p-3">{statusBadge(inv.status)}</td>
                  <td className="p-3 text-right space-x-2">
                    {inv.pdfUrl && (
                      <Button variant="ghost" size="sm" asChild>
                        <a href={inv.pdfUrl} target="_blank" rel="noopener">
                          <Download className="h-4 w-4" />
                        </a>
                      </Button>
                    )}
                    {inv.hostedUrl && (
                      <Button variant="ghost" size="sm" asChild>
                        <a href={inv.hostedUrl} target="_blank" rel="noopener">
                          <ExternalLink className="h-4 w-4" />
                        </a>
                      </Button>
                    )}
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>

        {hasMore && (
          <div className="flex justify-center mt-4">
            <Button
              variant="outline"
              onClick={() =>
                fetchInvoices(invoices[invoices.length - 1].id)
              }
              disabled={loadingMore}
            >
              {loadingMore ? (
                <Loader2 className="mr-2 h-4 w-4 animate-spin" />
              ) : null}
              Carica altre fatture
            </Button>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### Errori Comuni da Evitare
- Non creare fatture senza verificare che il customer esista in Stripe
- Non inviare fatture in draft — sempre finalizzare prima
- Non dimenticare di gestire il webhook `invoice.payment_failed` per dunning
- Non esporre PDF URLs senza autenticazione

### Checklist di Verifica
- [ ] Stripe Checkout one-time payments funzionante
- [ ] Subscription checkout con trial period
- [ ] Billing portal configurato
- [ ] Upgrade/downgrade con proration
- [ ] Webhook handler completo con idempotency
- [ ] Refund parziale e totale
- [ ] PayPal integration con capture
- [ ] Payment form con Stripe Elements e 3D Secure
- [ ] Idempotency keys su tutti i pagamenti
- [ ] Retry logic con exponential backoff
- [ ] Dunning management per failed subscriptions
- [ ] Pricing page con monthly/yearly toggle
- [ ] Invoice list con PDF download
- [ ] PayPal webhook handler con signature verification
