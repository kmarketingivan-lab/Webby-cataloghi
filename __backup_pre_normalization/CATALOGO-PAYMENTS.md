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