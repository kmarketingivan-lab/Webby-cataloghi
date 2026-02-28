# CATALOGO-PAYMENTS

CATALOGO-PAYMENTS per Next.js 14 + Stripe
§ STRIPE SETUP COMPLETO
Installazione e Configurazione
bash
npm install stripe @stripe/stripe-js @stripe/react-stripe-js
npm install --save-dev @types/stripe
Gestione API Keys

.env.local:

env
# Stripe
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_API_VERSION=2023-10-16

# Modalità di test (opzionale)
NEXT_PUBLIC_STRIPE_TEST_PUBLISHABLE_KEY=pk_test_...
STRIPE_TEST_SECRET_KEY=sk_test_...
STRIPE_TEST_WEBHOOK_SECRET=whsec_...

lib/stripe/config.ts:

typescript
import Stripe from 'stripe';

export const getStripeConfig = () => {
  const isTestMode = process.env.NODE_ENV === 'development' || 
                     process.env.NEXT_PUBLIC_USE_STRIPE_TEST === 'true';
  
  return {
    isTestMode,
    publishableKey: isTestMode 
      ? process.env.NEXT_PUBLIC_STRIPE_TEST_PUBLISHABLE_KEY!
      : process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!,
    secretKey: isTestMode 
      ? process.env.STRIPE_TEST_SECRET_KEY!
      : process.env.STRIPE_SECRET_KEY!,
    webhookSecret: isTestMode
      ? process.env.STRIPE_TEST_WEBHOOK_SECRET!
      : process.env.STRIPE_WEBHOOK_SECRET!,
    apiVersion: '2023-10-16' as const,
  };
};

export const createStripeInstance = () => {
  const { secretKey, apiVersion } = getStripeConfig();
  return new Stripe(secretKey, {
    apiVersion,
    typescript: true,
  });
};
Setup Webhook Endpoint

app/api/webhooks/stripe/route.ts:

typescript
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';
import { getStripeConfig } from '@/lib/stripe/config';

export async function POST(request: NextRequest) {
  const { webhookSecret } = getStripeConfig();
  
  if (!webhookSecret) {
    return NextResponse.json(
      { error: 'Webhook secret not configured' },
      { status: 500 }
    );
  }

  const body = await request.text();
  const signature = request.headers.get('stripe-signature') || '';

  let event: Stripe.Event;

  try {
    const stripe = new Stripe(webhookSecret.split('_')[1], {
      apiVersion: '2023-10-16',
    });
    
    event = stripe.webhooks.constructEvent(body, signature, webhookSecret);
  } catch (error: any) {
    console.error('Webhook signature verification failed:', error.message);
    return NextResponse.json(
      { error: `Webhook Error: ${error.message}` },
      { status: 400 }
    );
  }

  // Handler per tipi di eventi
  try {
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutSessionCompleted(event.data.object as Stripe.Checkout.Session);
        break;
      case 'invoice.paid':
        await handleInvoicePaid(event.data.object as Stripe.Invoice);
        break;
      case 'invoice.payment_failed':
        await handleInvoicePaymentFailed(event.data.object as Stripe.Invoice);
        break;
      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(event.data.object as Stripe.Subscription);
        break;
      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(event.data.object as Stripe.Subscription);
        break;
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }

    return NextResponse.json({ received: true });
  } catch (error: any) {
    console.error('Webhook handler error:', error);
    return NextResponse.json(
      { error: 'Webhook handler failed' },
      { status: 500 }
    );
  }
}

async function handleCheckoutSessionCompleted(session: Stripe.Checkout.Session) {
  // Implementa la logica per checkout completato
  console.log('Checkout session completed:', session.id);
}

async function handleInvoicePaid(invoice: Stripe.Invoice) {
  // Implementa la logica per fattura pagata
  console.log('Invoice paid:', invoice.id);
}

async function handleInvoicePaymentFailed(invoice: Stripe.Invoice) {
  // Implementa la logica per pagamento fallito
  console.log('Invoice payment failed:', invoice.id);
}

async function handleSubscriptionUpdated(subscription: Stripe.Subscription) {
  // Implementa la logica per sottoscrizione aggiornata
  console.log('Subscription updated:', subscription.id);
}

async function handleSubscriptionDeleted(subscription: Stripe.Subscription) {
  // Implementa la logica per sottoscrizione cancellata
  console.log('Subscription deleted:', subscription.id);
}
Error Handling

lib/stripe/error-handler.ts:

typescript
export class StripeError extends Error {
  constructor(
    message: string,
    public code?: string,
    public statusCode?: number
  ) {
    super(message);
    this.name = 'StripeError';
  }
}

export const handleStripeError = (error: any) => {
  if (error.type === 'StripeCardError') {
    return new StripeError(
      error.message,
      error.code,
      402
    );
  }
  
  if (error.type === 'StripeRateLimitError') {
    return new StripeError(
      'Too many requests. Please try again later.',
      'rate_limit',
      429
    );
  }
  
  if (error.type === 'StripeInvalidRequestError') {
    return new StripeError(
      'Invalid request. Please check your input.',
      'invalid_request',
      400
    );
  }
  
  if (error.type === 'StripeAPIError') {
    return new StripeError(
      'Stripe API error. Please try again.',
      'api_error',
      500
    );
  }
  
  if (error.type === 'StripeConnectionError') {
    return new StripeError(
      'Network error. Please check your connection.',
      'connection_error',
      503
    );
  }
  
  if (error.type === 'StripeAuthenticationError') {
    return new StripeError(
      'Authentication failed. Please check your API keys.',
      'authentication_error',
      401
    );
  }
  
  return new StripeError(
    'An unexpected error occurred.',
    'unknown_error',
    500
  );
};

export const withStripeErrorHandling = async <T>(
  operation: () => Promise<T>
): Promise<T> => {
  try {
    return await operation();
  } catch (error: any) {
    throw handleStripeError(error);
  }
};
§ CHECKOUT SESSION
One-time Payments

app/api/checkout/sessions/route.ts:

typescript
import { NextRequest, NextResponse } from 'next/server';
import { createStripeInstance } from '@/lib/stripe/config';
import { withStripeErrorHandling } from '@/lib/stripe/error-handler';

export async function POST(request: NextRequest) {
  return withStripeErrorHandling(async () => {
    const stripe = createStripeInstance();
    const body = await request.json();
    
    const { 
      priceId, 
      quantity = 1, 
      customerEmail, 
      successUrl, 
      cancelUrl,
      metadata = {}
    } = body;

    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card', 'paypal'],
      line_items: [
        {
          price: priceId,
          quantity: quantity,
        },
      ],
      mode: 'payment',
      success_url: successUrl || `${process.env.NEXT_PUBLIC_APP_URL}/success?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: cancelUrl || `${process.env.NEXT_PUBLIC_APP_URL}/cart`,
      customer_email: customerEmail,
      metadata: {
        ...metadata,
        source: 'checkout_one_time',
      },
      allow_promotion_codes: true,
      billing_address_collection: 'required',
      shipping_address_collection: {
        allowed_countries: ['US', 'CA', 'GB', 'FR', 'DE', 'IT', 'ES'],
      },
    });

    return NextResponse.json({ sessionId: session.id });
  });
}
Subscription Checkout

app/api/checkout/subscription/route.ts:

typescript
import { NextRequest, NextResponse } from 'next/server';
import { createStripeInstance } from '@/lib/stripe/config';

export async function POST(request: NextRequest) {
  const stripe = createStripeInstance();
  const body = await request.json();
  
  const {
    priceId,
    customerId,
    customerEmail,
    trialPeriodDays = 0,
    successUrl,
    cancelUrl,
    metadata = {}
  } = body;

  try {
    const session = await stripe.checkout.sessions.create({
      payment_method_types: ['card'],
      line_items: [
        {
          price: priceId,
          quantity: 1,
        },
      ],
      mode: 'subscription',
      success_url: successUrl || `${process.env.NEXT_PUBLIC_APP_URL}/dashboard?session_id={CHECKOUT_SESSION_ID}`,
      cancel_url: cancelUrl || `${process.env.NEXT_PUBLIC_APP_URL}/pricing`,
      customer: customerId,
      customer_email: customerEmail,
      subscription_data: {
        trial_period_days: trialPeriodDays,
        metadata: {
          ...metadata,
          signup_timestamp: Date.now().toString(),
        },
      },
      allow_promotion_codes: true,
      billing_address_collection: 'required',
      payment_intent_data: {
        setup_future_usage: 'off_session',
      },
    });

    return NextResponse.json({ sessionId: session.id });
  } catch (error: any) {
    console.error('Subscription checkout error:', error);
    return NextResponse.json(
      { error: error.message },
      { status: 400 }
    );
  }
}
Custom Checkout Page

components/checkout/CheckoutForm.tsx:

typescript
'use client';

import { useState } from 'react';
import {
  Elements,
  PaymentElement,
  AddressElement,
  useStripe,
  useElements,
} from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';
import { getStripeConfig } from '@/lib/stripe/config';

const stripePromise = loadStripe(getStripeConfig().publishableKey);

interface CheckoutFormProps {
  clientSecret: string;
  onSuccess: (paymentIntentId: string) => void;
  onError: (error: string) => void;
}

function CheckoutFormComponent({ clientSecret, onSuccess, onError }: CheckoutFormProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [isLoading, setIsLoading] = useState(false);
  const [errorMessage, setErrorMessage] = useState<string>('');

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();

    if (!stripe || !elements) {
      return;
    }

    setIsLoading(true);
    setErrorMessage('');

    try {
      const { error, paymentIntent } = await stripe.confirmPayment({
        elements,
        confirmParams: {
          return_url: `${window.location.origin}/payment/complete`,
        },
        redirect: 'if_required',
      });

      if (error) {
        setErrorMessage(error.message || 'An error occurred');
        onError(error.message || 'An error occurred');
      } else if (paymentIntent?.status === 'succeeded') {
        onSuccess(paymentIntent.id);
      }
    } catch (error: any) {
      setErrorMessage(error.message);
      onError(error.message);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-6">
      <PaymentElement />
      <AddressElement
        options={{
          mode: 'billing',
          allowedCountries: ['US', 'CA', 'GB'],
          fields: {
            phone: 'always',
          },
          validation: {
            phone: {
              required: 'never',
            },
          },
        }}
      />
      
      {errorMessage && (
        <div className="text-red-600 text-sm">{errorMessage}</div>
      )}
      
      <button
        type="submit"
        disabled={!stripe || isLoading}
        className="w-full bg-blue-600 text-white py-3 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed"
      >
        {isLoading ? 'Processing...' : 'Pay Now'}
      </button>
    </form>
  );
}

export function CheckoutForm(props: CheckoutFormProps) {
  return (
    <Elements stripe={stripePromise} options={{ clientSecret: props.clientSecret }}>
      <CheckoutFormComponent {...props} />
    </Elements>
  );
}
Configurazione Payment Methods

lib/stripe/payment-methods.ts:

typescript
import { createStripeInstance } from './config';

export const getSupportedPaymentMethods = (country?: string) => {
  const baseMethods = ['card'];
  
  // Aggiungi metodi in base al paese
  if (!country || ['US', 'CA', 'GB', 'AU'].includes(country)) {
    baseMethods.push('us_bank_account');
  }
  
  if (country === 'US') {
    baseMethods.push('affirm', 'afterpay_clearpay');
  }
  
  // PayPal disponibile in molti paesi
  baseMethods.push('paypal');
  
  // Klarna disponibile in Europa
  if (['DE', 'AT', 'NL', 'BE', 'ES', 'IT', 'FR'].includes(country || '')) {
    baseMethods.push('klarna');
  }
  
  return [...new Set(baseMethods)];
};

export const createPaymentMethod = async (
  paymentMethodType: string,
  details: any
) => {
  const stripe = createStripeInstance();
  
  return await stripe.paymentMethods.create({
    type: paymentMethodType as any,
    [paymentMethodType]: details,
    billing_details: {
      name: details.name,
      email: details.email,
      phone: details.phone,
      address: {
        line1: details.address_line1,
        city: details.city,
        state: details.state,
        postal_code: details.postal_code,
        country: details.country,
      },
    },
  });
};

export const attachPaymentMethodToCustomer = async (
  paymentMethodId: string,
  customerId: string
) => {
  const stripe = createStripeInstance();
  
  await stripe.paymentMethods.attach(paymentMethodId, {
    customer: customerId,
  });
  
  // Imposta come metodo di pagamento predefinito
  await stripe.customers.update(customerId, {
    invoice_settings: {
      default_payment_method: paymentMethodId,
    },
  });
};
§ STRIPE ELEMENTS
CardElement Component

components/stripe/CardForm.tsx:

typescript
'use client';

import { useState } from 'react';
import {
  CardElement,
  useStripe,
  useElements,
  Elements,
} from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';
import { getStripeConfig } from '@/lib/stripe/config';

const stripePromise = loadStripe(getStripeConfig().publishableKey);

interface CardFormProps {
  amount: number;
  currency: string;
  onPaymentSuccess: (paymentIntentId: string) => void;
  onPaymentError: (error: string) => void;
}

function CardFormComponent({ 
  amount, 
  currency, 
  onPaymentSuccess, 
  onPaymentError 
}: CardFormProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [isProcessing, setIsProcessing] = useState(false);
  const [cardError, setCardError] = useState<string>('');
  const [cardComplete, setCardComplete] = useState(false);

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();

    if (!stripe || !elements) {
      return;
    }

    setIsProcessing(true);
    setCardError('');

    const cardElement = elements.getElement(CardElement);
    
    if (!cardElement) {
      setCardError('Card element not found');
      return;
    }

    try {
      // Crea PaymentIntent
      const response = await fetch('/api/payment-intents', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ amount, currency }),
      });

      const { clientSecret } = await response.json();

      // Conferma il pagamento
      const { error, paymentIntent } = await stripe.confirmCardPayment(
        clientSecret,
        {
          payment_method: {
            card: cardElement,
          },
        }
      );

      if (error) {
        setCardError(error.message || 'Payment failed');
        onPaymentError(error.message || 'Payment failed');
      } else if (paymentIntent?.status === 'succeeded') {
        onPaymentSuccess(paymentIntent.id);
      }
    } catch (error: any) {
      setCardError(error.message);
      onPaymentError(error.message);
    } finally {
      setIsProcessing(false);
    }
  };

  const cardElementOptions = {
    style: {
      base: {
        fontSize: '16px',
        color: '#32325d',
        fontFamily: '"Helvetica Neue", Helvetica, sans-serif',
        fontSmoothing: 'antialiased',
        '::placeholder': {
          color: '#aab7c4',
        },
      },
      invalid: {
        color: '#fa755a',
        iconColor: '#fa755a',
      },
    },
    hidePostalCode: false,
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="border rounded-lg p-4">
        <CardElement
          options={cardElementOptions}
          onChange={(event) => {
            setCardError(event.error?.message || '');
            setCardComplete(event.complete);
          }}
        />
      </div>
      
      {cardError && (
        <div className="text-red-600 text-sm">{cardError}</div>
      )}
      
      <button
        type="submit"
        disabled={!stripe || isProcessing || !cardComplete}
        className="w-full bg-green-600 text-white py-3 px-4 rounded-lg hover:bg-green-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
      >
        {isProcessing ? 'Processing...' : `Pay ${(amount / 100).toFixed(2)} ${currency}`}
      </button>
      
      <div className="text-xs text-gray-500 text-center">
        Secure payment by Stripe
      </div>
    </form>
  );
}

export function CardForm(props: CardFormProps) {
  return (
    <Elements stripe={stripePromise}>
      <CardFormComponent {...props} />
    </Elements>
  );
}
PaymentElement con Styling Avanzato

components/stripe/CustomPaymentForm.tsx:

typescript
'use client';

import { useState } from 'react';
import {
  PaymentElement,
  LinkAuthenticationElement,
  useStripe,
  useElements,
  Elements,
} from '@stripe/react-stripe-js';
import { loadStripe, StripeElementsOptions } from '@stripe/stripe-js';
import { getStripeConfig } from '@/lib/stripe/config';

const stripePromise = loadStripe(getStripeConfig().publishableKey);

interface CustomPaymentFormProps {
  clientSecret: string;
  appearance?: StripeElementsOptions['appearance'];
}

function CustomPaymentFormComponent({ 
  clientSecret, 
  appearance 
}: CustomPaymentFormProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [email, setEmail] = useState('');
  const [message, setMessage] = useState<string>('');
  const [isLoading, setIsLoading] = useState(false);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();

    if (!stripe || !elements) {
      return;
    }

    setIsLoading(true);

    const { error } = await stripe.confirmPayment({
      elements,
      confirmParams: {
        return_url: `${window.location.origin}/payment/complete`,
        receipt_email: email,
      },
    });

    if (error.type === 'card_error' || error.type === 'validation_error') {
      setMessage(error.message || 'An unexpected error occurred.');
    } else {
      setMessage('An unexpected error occurred.');
    }

    setIsLoading(false);
  };

  const paymentElementOptions = {
    layout: 'tabs' as const,
    wallets: {
      applePay: 'never' as const,
      googlePay: 'never' as const,
    },
  };

  return (
    <form id="payment-form" onSubmit={handleSubmit} className="space-y-6">
      <LinkAuthenticationElement
        options={{ defaultValues: { email } }}
        onChange={(e) => setEmail(e.value.email)}
      />
      
      <PaymentElement 
        id="payment-element" 
        options={paymentElementOptions}
      />
      
      {message && (
        <div id="payment-message" className="text-red-600 text-sm">
          {message}
        </div>
      )}
      
      <button
        disabled={isLoading || !stripe || !elements}
        id="submit"
        className="w-full bg-purple-600 text-white py-3 px-4 rounded-lg hover:bg-purple-700 disabled:opacity-50 disabled:cursor-not-allowed shadow-lg"
      >
        <span id="button-text">
          {isLoading ? (
            <div className="spinner" id="spinner"></div>
          ) : (
            `Pay now`
          )}
        </span>
      </button>
    </form>
  );
}

export function CustomPaymentForm({ 
  clientSecret, 
  appearance = {
    theme: 'stripe',
    variables: {
      colorPrimary: '#0570de',
      colorBackground: '#ffffff',
      colorText: '#30313d',
      colorDanger: '#df1b41',
      fontFamily: 'Ideal Sans, system-ui, sans-serif',
      spacingUnit: '4px',
      borderRadius: '8px',
    },
  }
}: CustomPaymentFormProps) {
  const options: StripeElementsOptions = {
    clientSecret,
    appearance,
    loader: 'auto',
  };

  return (
    <Elements stripe={stripePromise} options={options}>
      <CustomPaymentFormComponent clientSecret={clientSecret} />
    </Elements>
  );
}
AddressElement con Validazione

components/stripe/AddressForm.tsx:

typescript
'use client';

import { useState } from 'react';
import {
  AddressElement,
  useElements,
  useStripe,
  Elements,
} from '@stripe/react-stripe-js';
import { loadStripe } from '@stripe/stripe-js';
import { getStripeConfig } from '@/lib/stripe/config';

const stripePromise = loadStripe(getStripeConfig().publishableKey);

interface AddressFormProps {
  mode: 'shipping' | 'billing';
  onAddressChange: (address: any) => void;
  onValidationChange: (isValid: boolean) => void;
}

function AddressFormComponent({ 
  mode, 
  onAddressChange, 
  onValidationChange 
}: AddressFormProps) {
  const stripe = useStripe();
  const elements = useElements();
  const [address, setAddress] = useState<any>(null);

  const handleChange = async (event: any) => {
    const { complete, value } = event;
    
    onValidationChange(complete);
    
    if (complete) {
      setAddress(value);
      onAddressChange(value);
      
      // Puoi ottenere i dettagli completi
      const addressElement = elements?.getElement(AddressElement);
      if (addressElement) {
        const { complete: isComplete, value: fullValue } = await addressElement.getValue();
        if (isComplete) {
          console.log('Full address:', fullValue);
        }
      }
    }
  };

  const addressElementOptions = {
    mode: mode,
    allowedCountries: ['US', 'CA', 'GB', 'FR', 'DE', 'IT', 'ES', 'NL', 'BE'],
    fields: {
      phone: mode === 'billing' ? 'always' : 'never',
    },
    validation: {
      phone: {
        required: mode === 'billing' ? 'always' : 'never',
      },
    },
    display: {
      name: mode === 'shipping' ? 'split' : 'organization',
    },
    autocomplete: {
      mode: 'automatic',
    },
  };

  return (
    <div className="space-y-4">
      <div className="border rounded-lg p-4 bg-gray-50">
        <AddressElement
          options={addressElementOptions}
          onChange={handleChange}
        />
      </div>
      
      {address && (
        <div className="text-sm text-gray-600 p-3 bg-blue-50 rounded">
          <div>Address confirmed:</div>
          <div>{address.name}</div>
          <div>{address.address.line1}</div>
          {address.address.line2 && <div>{address.address.line2}</div>}
          <div>{address.address.city}, {address.address.state} {address.address.postal_code}</div>
          <div>{address.address.country}</div>
        </div>
      )}
    </div>
  );
}

export function AddressForm(props: AddressFormProps) {
  return (
    <Elements stripe={stripePromise}>
      <AddressFormComponent {...props} />
    </Elements>
  );
}
Gestione Stati di Caricamento e Errori

components/stripe/PaymentStateManager.tsx:

typescript
'use client';

import { ReactNode, createContext, useContext, useState, useCallback } from 'react';

interface PaymentState {
  isLoading: boolean;
  error: string | null;
  success: boolean;
  paymentIntentId: string | null;
}

interface PaymentStateContextType extends PaymentState {
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  setSuccess: (success: boolean, paymentIntentId?: string) => void;
  reset: () => void;
}

const PaymentStateContext = createContext<PaymentStateContextType | undefined>(undefined);

export function PaymentStateProvider({ children }: { children: ReactNode }) {
  const [state, setState] = useState<PaymentState>({
    isLoading: false,
    error: null,
    success: false,
    paymentIntentId: null,
  });

  const setLoading = useCallback((loading: boolean) => {
    setState(prev => ({ ...prev, isLoading: loading }));
  }, []);

  const setError = useCallback((error: string | null) => {
    setState(prev => ({ 
      ...prev, 
      error, 
      isLoading: false,
      success: false 
    }));
  }, []);

  const setSuccess = useCallback((success: boolean, paymentIntentId?: string) => {
    setState(prev => ({ 
      ...prev, 
      success, 
      isLoading: false,
      error: null,
      paymentIntentId: paymentIntentId || null
    }));
  }, []);

  const reset = useCallback(() => {
    setState({
      isLoading: false,
      error: null,
      success: false,
      paymentIntentId: null,
    });
  }, []);

  return (
    <PaymentStateContext.Provider 
      value={{ ...state, setLoading, setError, setSuccess, reset }}
    >
      {children}
    </PaymentStateContext.Provider>
  );
}

export function usePaymentState() {
  const context = useContext(PaymentStateContext);
  if (!context) {
    throw new Error('usePaymentState must be used within PaymentStateProvider');
  }
  return context;
}

// Componente di esempio che usa lo stato di pagamento
export function PaymentButton() {
  const { isLoading, setLoading, setError, setSuccess } = usePaymentState();

  const handlePayment = async () => {
    setLoading(true);
    
    try {
      // Simula un pagamento
      await new Promise(resolve => setTimeout(resolve, 2000));
      
      // Successo
      setSuccess(true, 'pi_123456789');
    } catch (error: any) {
      setError(error.message || 'Payment failed');
    }
  };

  return (
    <button
      onClick={handlePayment}
      disabled={isLoading}
      className="px-6 py-3 bg-blue-600 text-white rounded-lg disabled:opacity-50"
    >
      {isLoading ? 'Processing...' : 'Make Payment'}
    </button>
  );
}
§ SUBSCRIPTION MANAGEMENT
Creazione Sottoscrizione

lib/stripe/subscriptions.ts:

typescript
import { createStripeInstance } from './config';

export interface CreateSubscriptionParams {
  customerId: string;
  priceId: string;
  paymentMethodId?: string;
  trialPeriodDays?: number;
  couponId?: string;
  metadata?: Record<string, string>;
  promotionCode?: string;
}

export interface UpdateSubscriptionParams {
  subscriptionId: string;
  priceId?: string;
  quantity?: number;
  prorationBehavior?: 'create_prorations' | 'none';
  couponId?: string;
  trialEnd?: number | 'now';
  metadata?: Record<string, string>;
}

export async function createSubscription({
  customerId,
  priceId,
  paymentMethodId,
  trialPeriodDays = 0,
  couponId,
  metadata,
  promotionCode,
}: CreateSubscriptionParams) {
  const stripe = createStripeInstance();

  const subscriptionData: any = {
    customer: customerId,
    items: [{ price: priceId }],
    expand: ['latest_invoice.payment_intent'],
    metadata,
  };

  if (trialPeriodDays > 0) {
    subscriptionData.trial_period_days = trialPeriodDays;
  }

  if (paymentMethodId) {
    subscriptionData.default_payment_method = paymentMethodId;
  }

  if (couponId) {
    subscriptionData.coupon = couponId;
  }

  if (promotionCode) {
    subscriptionData.promotion_code = promotionCode;
  }

  const subscription = await stripe.subscriptions.create(subscriptionData);

  return subscription;
}

export async function updateSubscription({
  subscriptionId,
  priceId,
  quantity,
  prorationBehavior = 'create_prorations',
  couponId,
  trialEnd,
  metadata,
}: UpdateSubscriptionParams) {
  const stripe = createStripeInstance();

  const updateData: any = {
    proration_behavior: prorationBehavior,
  };

  if (priceId || quantity !== undefined) {
    const subscription = await stripe.subscriptions.retrieve(subscriptionId);
    
    updateData.items = [{
      id: subscription.items.data[0].id,
      price: priceId,
      quantity,
    }];
  }

  if (couponId) {
    updateData.coupon = couponId;
  }

  if (trialEnd) {
    updateData.trial_end = trialEnd;
  }

  if (metadata) {
    updateData.metadata = metadata;
  }

  const updatedSubscription = await stripe.subscriptions.update(
    subscriptionId,
    updateData
  );

  return updatedSubscription;
}

export async function cancelSubscription(
  subscriptionId: string,
  cancelAtPeriodEnd: boolean = false
) {
  const stripe = createStripeInstance();

  if (cancelAtPeriodEnd) {
    // Cancella alla fine del periodo
    const subscription = await stripe.subscriptions.update(subscriptionId, {
      cancel_at_period_end: true,
    });
    return subscription;
  } else {
    // Cancella immediatamente
    const deletedSubscription = await stripe.subscriptions.cancel(subscriptionId);
    return deletedSubscription;
  }
}

export async function pauseSubscription(subscriptionId: string) {
  const stripe = createStripeInstance();

  // Per pausare, creiamo una sottoscrizione con un prezzo gratuito
  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  
  const pausedSubscription = await stripe.subscriptions.update(subscriptionId, {
    items: [{
      id: subscription.items.data[0].id,
      price: process.env.STRIPE_FREE_PRICE_ID, // ID del prezzo gratuito
    }],
    metadata: {
      ...subscription.metadata,
      paused_at: Date.now().toString(),
      original_price_id: subscription.items.data[0].price.id,
    },
  });

  return pausedSubscription;
}

export async function resumeSubscription(subscriptionId: string) {
  const stripe = createStripeInstance();

  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  const originalPriceId = subscription.metadata.original_price_id;

  if (!originalPriceId) {
    throw new Error('Original price ID not found in metadata');
  }

  const resumedSubscription = await stripe.subscriptions.update(subscriptionId, {
    items: [{
      id: subscription.items.data[0].id,
      price: originalPriceId,
    }],
    metadata: {
      ...subscription.metadata,
      resumed_at: Date.now().toString(),
      original_price_id: undefined,
    },
  });

  return resumedSubscription;
}

export async function getSubscription(subscriptionId: string) {
  const stripe = createStripeInstance();
  
  const subscription = await stripe.subscriptions.retrieve(subscriptionId, {
    expand: [
      'customer',
      'latest_invoice',
      'default_payment_method',
      'pending_setup_intent',
      'pending_update',
    ],
  });

  return subscription;
}

export async function listSubscriptions(customerId: string) {
  const stripe = createStripeInstance();
  
  const subscriptions = await stripe.subscriptions.list({
    customer: customerId,
    status: 'all',
    expand: ['data.default_payment_method'],
    limit: 100,
  });

  return subscriptions.data;
}

export async function createSubscriptionSchedule(
  customerId: string,
  phases: Array<{
    start_date: number;
    priceId: string;
    quantity?: number;
    trial_end?: number;
  }>
) {
  const stripe = createStripeInstance();

  const schedule = await stripe.subscriptionSchedules.create({
    customer: customerId,
    start_date: 'now',
    phases: phases.map(phase => ({
      start_date: phase.start_date,
      items: [{ price: phase.priceId, quantity: phase.quantity || 1 }],
      trial_end: phase.trial_end,
    })),
  });

  return schedule;
}
Gestione Proration e Preview

lib/stripe/proration.ts:

typescript
import { createStripeInstance } from './config';

export interface ProrationPreview {
  invoicePreview: any;
  prorationDate: number;
  currentPeriodStart: number;
  currentPeriodEnd: number;
  prorationAmount: number;
  creditAmount: number;
  totalAmount: number;
}

export async function getProrationPreview(
  subscriptionId: string,
  newPriceId: string,
  quantity: number = 1
): Promise<ProrationPreview> {
  const stripe = createStripeInstance();

  // Ottieni la sottoscrizione corrente
  const subscription = await stripe.subscriptions.retrieve(subscriptionId);
  
  // Crea un preview dell'invoice
  const invoice = await stripe.invoices.retrieveUpcoming({
    customer: subscription.customer as string,
    subscription: subscriptionId,
    subscription_items: [{
      id: subscription.items.data[0].id,
      price: newPriceId,
      quantity,
    }],
    subscription_proration_date: Math.floor(Date.now() / 1000),
  });

  // Calcola l'importo della prorata
  const prorationLineItems = invoice.lines.data.filter(
    line => line.proration
  );

  const prorationAmount = prorationLineItems.reduce(
    (sum, line) => sum + (line.amount || 0),
    0
  );

  // Calcola crediti (se presenti)
  const creditLineItems = invoice.lines.data.filter(
    line => line.description?.includes('credit')
  );

  const creditAmount = creditLineItems.reduce(
    (sum, line) => sum + Math.abs(line.amount || 0),
    0
  );

  return {
    invoicePreview: invoice,
    prorationDate: invoice.subscription_proration_date || Math.floor(Date.now() / 1000),
    currentPeriodStart: subscription.current_period_start,
    currentPeriodEnd: subscription.current_period_end,
    proration

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-25-PAYMENTS
Prompt ID: 25 / 48
Parte: 2
Exported: 2026-02-06T11:24:56.350Z
Characters: 1148
════════════════════════════════════════════════════════════

ntilDue = 30,
  metadata,
  subscriptionId,
}: CreateInvoiceParams) {
  const stripe = createStripeInstance();

  const invoiceData: any = {
    customer: customerId,
    auto_advance: autoAdvance,
    collection_method: 'send_invoice',
    days_until_due: daysUntilDue,
    metadata,
    subscription: subscriptionId,
    description,
  };

  // Aggiungi gli item alla fattura
  if (items.length > 0) {
    invoiceData.lines = items.map(item => ({
      price_data: item.priceId ? undefined : {
        currency: currency.toLowerCase(),
        unit_amount: item.amount,
        product_data: {
          name: item.description,
        },
        tax_behavior: 'exclusive',
      },
      price: item.priceId,
      description: item.priceId ? undefined : item.description,
      quantity: item.quantity || 1,
      tax_rates: item.taxRates,
    }));
  }

  const invoice = await stripe.invoices.create(invoiceData);

  return invoice;
}

export async function sendInvoice({ invoiceId, customerEmail, cc }: SendInvoiceParams) {
  const stripe = createStripeInstance();

  const invoice = await stripe.invoices.sendInvoice(invoiceId, {
    expand:

## § ADVANCED PATTERNS: PAYMENTS

### Server Actions con Validazione

```typescript
// app/actions/payments.ts
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


### PAYMENTS - Utility Helper #797

```typescript
// lib/utils/payments-helper-797.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #360

```typescript
// lib/utils/payments-helper-360.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #169

```typescript
// lib/utils/payments-helper-169.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #156

```typescript
// lib/utils/payments-helper-156.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #382

```typescript
// lib/utils/payments-helper-382.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #813

```typescript
// lib/utils/payments-helper-813.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #401

```typescript
// lib/utils/payments-helper-401.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #26

```typescript
// lib/utils/payments-helper-26.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #407

```typescript
// lib/utils/payments-helper-407.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #197

```typescript
// lib/utils/payments-helper-197.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #309

```typescript
// lib/utils/payments-helper-309.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #892

```typescript
// lib/utils/payments-helper-892.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #478

```typescript
// lib/utils/payments-helper-478.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #602

```typescript
// lib/utils/payments-helper-602.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #713

```typescript
// lib/utils/payments-helper-713.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #694

```typescript
// lib/utils/payments-helper-694.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
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

  getConfig(): Readonly<PAYMENTSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### PAYMENTS - Utility Helper #889

```typescript
// lib/utils/payments-helper-889.ts
import { z } from "zod";

interface PAYMENTSConfig {
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

export class PAYMENTSProcessor<TInput, TOutput> {
  private config: PAYMENTSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<PAYMENTSConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
   