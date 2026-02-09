# CATALOGO NOTIFICATIONS v1

> **Versione**: 1.0  
> **Data**: 2026-01-27  
> **Ambito**: Multi-channel notifications, Email, Push, SMS, In-app  

---

## 1. NOTIFICATION CHANNELS COMPARISON

| Channel | Delivery Speed | Reliability | Cost | User Reach | Best For |
|---------|----------------|-------------|------|------------|----------|
| Email | 🟡 Medium (1-5 min) | 🟢 High (95%+) | 💰 Low ($0.10/1000) | 🟢 100% | Transactional, newsletters, receipts |
| Push (Web) | 🟢 Instant (<1s) | 🟡 Medium (80-90%) | 🟢 Free | 🟡 60-80% | Real-time alerts, engagement |
| Push (Mobile) | 🟢 Instant (<1s) | 🟢 High (95%+) | 🟢 Free | 🟢 90%+ | Mobile app notifications |
| In-App | 🟢 Instant (<100ms) | 🟢 High (100%) | 🟢 Free | 🟢 100% | App-specific updates, social |
| SMS | 🟢 Instant (<10s) | 🟢 High (98%+) | 🔴 High ($0.01-0.10) | 🟢 98% | Critical alerts, OTP |
| Slack/Discord | 🟢 Instant (<2s) | 🟢 High (95%+) | 🟢 Free | 🟡 40-60% | Team collaboration, monitoring |
| Webhook | 🟢 Instant (<100ms) | 🟢 High (99%+) | 🟢 Free | 🟡 50-70% | System integrations, automation |

### Multi-Channel Strategy Table

| Notification Type | Primary | Fallback | Urgency | Optimal Delivery |
|-------------------|---------|----------|---------|------------------|
| Welcome | Email + In-App | - | Low | Email (delayed), In-App (immediate) |
| Password Reset | Email | SMS | High | Email with SMS backup |
| New Message | Push + In-App | Email | Medium | Push for instant, In-App for persistence |
| Payment Failed | Email + SMS | - | Critical | SMS first, Email for details |
| Marketing | Email | - | Low | Email with tracking |
| Security Alert | Push + Email | SMS | Critical | Push for speed, Email for audit |
| Order Shipped | Email | In-App | Medium | Email with tracking details |
| Event Reminder | Push | Email | Medium | Push 1h before, Email 24h before |
| System Down | SMS + Push | Email | Critical | SMS for admins, Push for users |
| Social Mention | In-App | Email | Low | In-App real-time |

---

## 2. EMAIL PROVIDERS COMPARISON

| Provider | Free Tier | Deliverability | API Quality | Templates | Best For |
|----------|-----------|----------------|-------------|-----------|----------|
| Resend | 3k/month | 🟢 Excellent | 🟢 Excellent | 🟢 React Email | Startups, developers |
| SendGrid | 100/day | 🟢 Excellent | 🟢 Good | 🟡 UI editor | Enterprise, high volume |
| Postmark | 100/month | 🟢 Excellent | 🟢 Excellent | 🟡 Custom | Transactional only |
| AWS SES | 62k/month (EC2) | 🟢 Good | 🟡 Medium | 🔴 None | AWS shops, cost-conscious |
| Mailgun | 5k/month (3mo) | 🟡 Good | 🟡 Medium | 🟡 UI editor | High volume, flexibility |
| Brevo | 300/day | 🟡 Good | 🟡 Medium | 🟢 Drag & drop | Marketing + transactional |

---

## 3. TRANSACTIONAL EMAIL IMPLEMENTATION

### 3.1 Resend Setup (Recommended)

```typescript
// lib/email/resend.ts
import { Resend } from 'resend';
import { RateLimiterMemory } from 'rate-limiter-flexible';

const resend = new Resend(process.env.RESEND_API_KEY!);

// Rate limiter: 100 emails per user per hour
const rateLimiter = new RateLimiterMemory({
  points: 100,
  duration: 60 * 60, // 1 hour
});

interface EmailOptions {
  to: string | string[];
  subject: string;
  from?: string;
  replyTo?: string;
  tags?: { name: string; value: string }[];
  attachments?: {
    filename: string;
    content: Buffer | string;
    contentType?: string;
  }[];
  headers?: Record<string, string>;
}

export async function sendEmail(
  options: EmailOptions,
  reactComponent: React.ReactElement,
  userId?: string
): Promise<{ success: boolean; messageId?: string; error?: string }> {
  try {
    // Rate limiting check
    if (userId) {
      try {
        await rateLimiter.consume(`email:${userId}`);
      } catch (error) {
        throw new Error('Rate limit exceeded. Please try again later.');
      }
    }

    const response = await resend.emails.send({
      from: options.from || process.env.EMAIL_FROM!,
      to: Array.isArray(options.to) ? options.to : [options.to],
      replyTo: options.replyTo,
      subject: options.subject,
      react: reactComponent,
      tags: options.tags,
      attachments: options.attachments,
      headers: {
        'List-Unsubscribe': `<${process.env.APP_URL}/unsubscribe?email=${encodeURIComponent(
          Array.isArray(options.to) ? options.to[0] : options.to
        )}>`,
        ...options.headers,
      },
    });

    if (response.error) {
      console.error('Resend error:', response.error);
      return {
        success: false,
        error: response.error.message,
      };
    }

    return {
      success: true,
      messageId: response.data?.id,
    };
  } catch (error) {
    console.error('Email send error:', error);
    
    // Don't expose internal errors to user
    return {
      success: false,
      error: 'Failed to send email',
    };
  }
}

// Utility to send emails in background (non-blocking)
export function sendEmailAsync(
  options: EmailOptions,
  reactComponent: React.ReactElement,
  userId?: string
) {
  // In production, use a job queue
  if (process.env.NODE_ENV === 'production') {
    // Queue for background processing
    emailQueue.add('send-email', {
      options,
      reactComponent,
      userId,
    });
  } else {
    // In development, send immediately
    sendEmail(options, reactComponent, userId).catch(console.error);
  }
}
```

### 3.2 Email Templates System

```typescript
// lib/email/templates/base.tsx
import {
  Body,
  Container,
  Head,
  Heading,
  Hr,
  Html,
  Link,
  Preview,
  Section,
  Tailwind,
  Text,
} from '@react-email/components';

interface BaseEmailProps {
  children: React.ReactNode;
  preview?: string;
  title?: string;
  footerText?: string;
}

export function BaseEmail({
  children,
  preview = '',
  title = '',
  footerText = '',
}: BaseEmailProps) {
  const currentYear = new Date().getFullYear();
  const appName = process.env.NEXT_PUBLIC_APP_NAME || 'MyApp';
  const appUrl = process.env.NEXT_PUBLIC_APP_URL || 'https://app.example.com';

  return (
    <Html>
      <Head />
      <Preview>{preview}</Preview>
      <Tailwind>
        <Body className="bg-gray-50 font-sans">
          <Container className="mx-auto max-w-2xl p-4 bg-white">
            {/* Header */}
            <Section className="mb-6">
              <Heading as="h1" className="text-2xl font-bold text-gray-900">
                {appName}
              </Heading>
              {title && (
                <Text className="text-gray-600 mt-2">
                  {title}
                </Text>
              )}
            </Section>

            {/* Content */}
            {children}

            <Hr className="my-6" />

            {/* Footer */}
            <Section>
              <Text className="text-sm text-gray-500">
                {footerText || `© ${currentYear} ${appName}. All rights reserved.`}
              </Text>
              <Text className="text-xs text-gray-400 mt-2">
                <Link
                  href={`${appUrl}/preferences`}
                  className="text-blue-500 hover:underline"
                >
                  Manage your preferences
                </Link>
                {' • '}
                <Link
                  href={`${appUrl}/unsubscribe`}
                  className="text-blue-500 hover:underline"
                >
                  Unsubscribe
                </Link>
              </Text>
              <Text className="text-xs text-gray-400 mt-2">
                Sent with ❤️ from {appName}
              </Text>
            </Section>
          </Container>
        </Body>
      </Tailwind>
    </Html>
  );
}
```

### 3.3 Email Template Implementations

```typescript
// lib/email/templates/welcome.tsx
import { Button, Section, Text } from '@react-email/components';
import { BaseEmail } from './base';

interface WelcomeEmailProps {
  name: string;
  verifyUrl: string;
}

export function WelcomeEmail({ name, verifyUrl }: WelcomeEmailProps) {
  return (
    <BaseEmail
      title="Welcome to Our Platform!"
      preview="Get started with your new account"
    >
      <Section>
        <Text className="text-lg">
          Hi {name},
        </Text>
        
        <Text className="mt-4">
          Welcome to our platform! We're excited to have you on board.
        </Text>
        
        <Text className="mt-2">
          To get started, please verify your email address by clicking the button below:
        </Text>
        
        <Section className="text-center mt-6 mb-6">
          <Button
            href={verifyUrl}
            className="bg-blue-500 text-white px-6 py-3 rounded-lg font-semibold"
          >
            Verify Email Address
          </Button>
        </Section>
        
        <Text className="text-sm text-gray-500 mt-4">
          If the button doesn't work, copy and paste this link into your browser:
        </Text>
        <Text className="text-sm text-gray-500 break-all">
          {verifyUrl}
        </Text>
      </Section>
    </BaseEmail>
  );
}

// lib/email/templates/password-reset.tsx
import { Button, Section, Text } from '@react-email/components';
import { BaseEmail } from './base';

interface PasswordResetEmailProps {
  name: string;
  resetUrl: string;
  expiryMinutes: number;
}

export function PasswordResetEmail({ 
  name, 
  resetUrl, 
  expiryMinutes = 15 
}: PasswordResetEmailProps) {
  return (
    <BaseEmail
      title="Reset Your Password"
      preview="Click here to reset your password"
    >
      <Section>
        <Text className="text-lg">
          Hi {name},
        </Text>
        
        <Text className="mt-4">
          We received a request to reset your password. Click the button below to create a new password:
        </Text>
        
        <Section className="text-center mt-6 mb-6">
          <Button
            href={resetUrl}
            className="bg-red-500 text-white px-6 py-3 rounded-lg font-semibold"
          >
            Reset Password
          </Button>
        </Section>
        
        <Text className="text-sm text-gray-500">
          This link will expire in {expiryMinutes} minutes.
        </Text>
        
        <Text className="mt-4">
          If you didn't request a password reset, please ignore this email or contact support if you have concerns.
        </Text>
      </Section>
    </BaseEmail>
  );
}

// lib/email/templates/invoice.tsx
import { Section, Text } from '@react-email/components';
import { BaseEmail } from './base';

interface InvoiceEmailProps {
  name: string;
  invoiceNumber: string;
  amount: number;
  dueDate: string;
  invoiceUrl: string;
  items: Array<{
    description: string;
    quantity: number;
    unitPrice: number;
    total: number;
  }>;
}

export function InvoiceEmail({
  name,
  invoiceNumber,
  amount,
  dueDate,
  invoiceUrl,
  items,
}: InvoiceEmailProps) {
  const formattedAmount = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);

  return (
    <BaseEmail
      title={`Invoice #${invoiceNumber}`}
      preview={`Your invoice for ${formattedAmount} is ready`}
    >
      <Section>
        <Text className="text-lg">
          Hi {name},
        </Text>
        
        <Text className="mt-4">
          Your invoice #{invoiceNumber} for {formattedAmount} is now available.
        </Text>
        
        <Section className="bg-gray-50 p-4 rounded-lg my-6">
          <Text className="font-semibold">Invoice Summary</Text>
          <Text>Invoice #: {invoiceNumber}</Text>
          <Text>Amount Due: {formattedAmount}</Text>
          <Text>Due Date: {dueDate}</Text>
          
          {items.length > 0 && (
            <Section className="mt-4">
              <Text className="font-semibold mb-2">Items:</Text>
              {items.map((item, index) => (
                <Text key={index} className="text-sm">
                  • {item.description}: {item.quantity} × ${item.unitPrice.toFixed(2)} = ${item.total.toFixed(2)}
                </Text>
              ))}
            </Section>
          )}
        </Section>
        
        <Text>
          <a
            href={invoiceUrl}
            className="text-blue-500 hover:underline font-semibold"
          >
            View and Pay Invoice →
          </a>
        </Text>
      </Section>
    </BaseEmail>
  );
}
```

### 3.4 Email Queue (Background Jobs)

```typescript
// lib/email/queue.ts
import Queue from 'bull';
import Redis from 'ioredis';
import { sendEmail } from './resend';

const redisConfig = {
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  password: process.env.REDIS_PASSWORD,
  maxRetriesPerRequest: null,
};

const connection = new Redis(redisConfig);

// Create queue
export const emailQueue = new Queue('email', {
  connection,
  defaultJobOptions: {
    attempts: 3,
    backoff: {
      type: 'exponential',
      delay: 1000, // 1s, 2s, 4s, etc.
    },
    removeOnComplete: 100, // Keep last 100 completed jobs
    removeOnFail: 500, // Keep last 500 failed jobs
  },
  limiter: {
    max: 100, // Max jobs per duration
    duration: 1000, // Per second
  },
});

// Process jobs
emailQueue.process('send-email', async (job) => {
  const { options, reactComponent, userId } = job.data;
  
  console.log(`Processing email to: ${options.to}`);
  
  const result = await sendEmail(options, reactComponent, userId);
  
  if (!result.success) {
    throw new Error(result.error || 'Failed to send email');
  }
  
  return { messageId: result.messageId };
});

// Handle job events
emailQueue.on('completed', (job, result) => {
  console.log(`Email sent: ${job.id}, Message ID: ${result.messageId}`);
});

emailQueue.on('failed', (job, error) => {
  console.error(`Email failed: ${job?.id}`, error);
  
  // Alert if too many failures
  if (job?.attemptsMade >= 3) {
    console.error(`Email job ${job.id} failed after 3 attempts`);
    // Send alert to admin
  }
});

// Cleanup function
export async function cleanupEmailQueue() {
  // Remove old jobs
  await emailQueue.clean(60 * 60 * 1000, 'completed'); // 1 hour
  await emailQueue.clean(7 * 24 * 60 * 60 * 1000, 'failed'); // 1 week
  
  // Get queue stats
  const counts = await emailQueue.getJobCounts();
  console.log('Email queue stats:', counts);
}

// Initialize queue on startup
export async function initializeEmailQueue() {
  await cleanupEmailQueue();
  console.log('Email queue initialized');
}
```

---

## 4. WEB PUSH NOTIFICATIONS

### 4.1 Browser Support Table

| Browser | Push API | Service Worker | Permissions |
|---------|----------|----------------|-------------|
| Chrome | ✅ 42+ | ✅ 40+ | 🟢 Default allow |
| Firefox | ✅ 44+ | ✅ 44+ | 🟢 Default allow |
| Safari | ✅ 16.4+ | ✅ 11.1+ | 🔴 Requires user gesture |
| Edge | ✅ 17+ | ✅ 17+ | 🟢 Default allow |
| iOS Safari | ✅ 16.4+ | ⚠️ Limited | 🔴 Requires user gesture |

### 4.2 Web Push Setup

```typescript
// public/sw.js - Service Worker
self.addEventListener('push', (event) => {
  if (!event.data) return;

  const data = event.data.json();
  
  const options = {
    body: data.body,
    icon: data.icon || '/icon-192x192.png',
    badge: '/badge-72x72.png',
    image: data.image,
    data: data.data || {},
    actions: data.actions || [],
    requireInteraction: data.requireInteraction || false,
    tag: data.tag, // Group notifications
    silent: data.silent || false,
    vibrate: data.vibrate || [200, 100, 200],
    timestamp: data.timestamp || Date.now(),
  };

  event.waitUntil(
    self.registration.showNotification(data.title, options)
  );
});

self.addEventListener('notificationclick', (event) => {
  event.notification.close();

  const { data } = event.notification;
  
  if (event.action) {
    // Handle action button click
    console.log(`Action clicked: ${event.action}`);
    
    // Send analytics
    if (data.notificationId) {
      fetch(`/api/notifications/${data.notificationId}/click`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ action: event.action }),
      }).catch(console.error);
    }
    
    // Open URL if provided
    const actionUrl = data.actions?.[event.action]?.url;
    if (actionUrl) {
      event.waitUntil(
        clients.openWindow(actionUrl)
      );
    }
  } else if (data.url) {
    // Handle notification click
    event.waitUntil(
      clients.matchAll({ type: 'window' }).then((windowClients) => {
        // Check if window is already open
        for (const client of windowClients) {
          if (client.url === data.url && 'focus' in client) {
            return client.focus();
          }
        }
        
        // Open new window
        return clients.openWindow(data.url);
      })
    );
    
    // Track click
    if (data.notificationId) {
      fetch(`/api/notifications/${data.notificationId}/click`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
      }).catch(console.error);
    }
  }
});

// Background sync for offline
self.addEventListener('sync', (event) => {
  if (event.tag === 'sync-notifications') {
    event.waitUntil(syncNotifications());
  }
});

async function syncNotifications() {
  const requests = await indexedDB.getAll('pending-notifications');
  
  for (const request of requests) {
    try {
      await fetch(request.url, request.options);
      await indexedDB.delete('pending-notifications', request.id);
    } catch (error) {
      console.error('Failed to sync notification:', error);
    }
  }
}
```

```typescript
// lib/push/web-push.ts
import webPush from 'web-push';
import { prisma } from '@/lib/prisma';

// Generate VAPID keys (run once and store in env)
// const vapidKeys = webPush.generateVAPIDKeys();
// console.log('Public Key:', vapidKeys.publicKey);
// console.log('Private Key:', vapidKeys.privateKey);

// Initialize web-push
webPush.setVapidDetails(
  `mailto:${process.env.EMAIL_FROM}`,
  process.env.VAPID_PUBLIC_KEY!,
  process.env.VAPID_PRIVATE_KEY!
);

export interface PushSubscription {
  endpoint: string;
  keys: {
    p256dh: string;
    auth: string;
  };
}

export interface PushNotification {
  title: string;
  body: string;
  icon?: string;
  badge?: string;
  image?: string;
  data?: Record<string, any>;
  actions?: Array<{
    action: string;
    title: string;
    icon?: string;
  }>;
  tag?: string;
  requireInteraction?: boolean;
  silent?: boolean;
  vibrate?: number[];
}

/**
 * Subscribe user to push notifications
 */
export async function subscribeUser(
  userId: string,
  subscription: PushSubscription
) {
  // Store subscription
  await prisma.pushSubscription.upsert({
    where: {
      endpoint: subscription.endpoint,
    },
    update: {
      userId,
      p256dh: subscription.keys.p256dh,
      auth: subscription.keys.auth,
      expiresAt: null,
    },
    create: {
      userId,
      endpoint: subscription.endpoint,
      p256dh: subscription.keys.p256dh,
      auth: subscription.keys.auth,
    },
  });

  // Send welcome notification
  await sendPushNotification(userId, {
    title: 'Notifications Enabled',
    body: 'You will now receive push notifications from our app.',
    icon: '/icon-192x192.png',
    data: { type: 'welcome' },
  });

  console.log(`User ${userId} subscribed to push notifications`);
}

/**
 * Unsubscribe user from push notifications
 */
export async function unsubscribeUser(endpoint: string) {
  await prisma.pushSubscription.delete({
    where: { endpoint },
  });
  
  console.log(`Unsubscribed endpoint: ${endpoint}`);
}

/**
 * Send push notification to user
 */
export async function sendPushNotification(
  userId: string,
  notification: PushNotification
): Promise<{ sent: number; failed: number }> {
  const subscriptions = await prisma.pushSubscription.findMany({
    where: { userId },
  });

  if (subscriptions.length === 0) {
    return { sent: 0, failed: 0 };
  }

  const payload = JSON.stringify({
    title: notification.title,
    body: notification.body,
    icon: notification.icon,
    badge: notification.badge,
    image: notification.image,
    data: {
      ...notification.data,
      url: notification.data?.url || process.env.NEXT_PUBLIC_APP_URL,
      timestamp: Date.now(),
    },
    actions: notification.actions,
    tag: notification.tag,
    requireInteraction: notification.requireInteraction,
    silent: notification.silent,
    vibrate: notification.vibrate,
  });

  const results = await Promise.allSettled(
    subscriptions.map(async (subscription) => {
      try {
        await webPush.sendNotification(
          {
            endpoint: subscription.endpoint,
            keys: {
              p256dh: subscription.p256dh,
              auth: subscription.auth,
            },
          },
          payload,
          {
            TTL: 86400, // 24 hours
            urgency: 'high',
          }
        );
        
        // Update last sent
        await prisma.pushSubscription.update({
          where: { id: subscription.id },
          data: { lastSentAt: new Date() },
        });
        
        return { success: true, subscriptionId: subscription.id };
      } catch (error) {
        console.error('Push notification failed:', error);
        
        // If subscription is invalid, remove it
        if (error.statusCode === 410) {
          await prisma.pushSubscription.delete({
            where: { id: subscription.id },
          });
        }
        
        return { success: false, subscriptionId: subscription.id, error };
      }
    })
  );

  const sent = results.filter(r => r.status === 'fulfilled' && r.value.success).length;
  const failed = results.length - sent;

  return { sent, failed };
}

/**
 * Send push notification to multiple users
 */
export async function sendBulkPushNotification(
  userIds: string[],
  notification: PushNotification,
  batchSize = 100
): Promise<{ totalSent: number; totalFailed: number }> {
  let totalSent = 0;
  let totalFailed = 0;

  for (let i = 0; i < userIds.length; i += batchSize) {
    const batch = userIds.slice(i, i + batchSize);
    
    const results = await Promise.all(
      batch.map(userId => sendPushNotification(userId, notification))
    );

    totalSent += results.reduce((sum, r) => sum + r.sent, 0);
    totalFailed += results.reduce((sum, r) => sum + r.failed, 0);

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 1000));
  }

  return { totalSent, totalFailed };
}
```

### 4.3 Permission Request Flow

```typescript
// hooks/usePushNotifications.ts
import { useState, useEffect, useCallback } from 'react';
import { toast } from 'sonner';

export function usePushNotifications() {
  const [permission, setPermission] = useState<NotificationPermission>('default');
  const [isSupported, setIsSupported] = useState(false);
  const [isSubscribed, setIsSubscribed] = useState(false);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    // Check browser support
    setIsSupported(
      'serviceWorker' in navigator &&
      'PushManager' in window &&
      'Notification' in window
    );

    // Check current permission
    if ('Notification' in window) {
      setPermission(Notification.permission);
    }

    // Check if already subscribed
    checkSubscription();
  }, []);

  const checkSubscription = useCallback(async () => {
    if (!isSupported) return;

    const registration = await navigator.serviceWorker.ready;
    const subscription = await registration.pushManager.getSubscription();
    
    setIsSubscribed(!!subscription);
  }, [isSupported]);

  const requestPermission = useCallback(async (): Promise<NotificationPermission> => {
    if (!isSupported) {
      toast.error('Push notifications are not supported in your browser');
      return 'denied';
    }

    try {
      const newPermission = await Notification.requestPermission();
      setPermission(newPermission);
      
      if (newPermission === 'granted') {
        toast.success('Notifications enabled!');
      } else if (newPermission === 'denied') {
        toast.info('Notifications blocked. You can enable them in your browser settings.');
      }
      
      return newPermission;
    } catch (error) {
      console.error('Error requesting notification permission:', error);
      toast.error('Failed to request notification permission');
      return 'denied';
    }
  }, [isSupported]);

  const subscribe = useCallback(async (): Promise<boolean> => {
    if (!isSupported || permission !== 'granted') {
      const newPermission = await requestPermission();
      if (newPermission !== 'granted') return false;
    }

    setIsLoading(true);

    try {
      // Register service worker
      const registration = await navigator.serviceWorker.register('/sw.js');
      
      // Get existing subscription
      let subscription = await registration.pushManager.getSubscription();
      
      if (!subscription) {
        // Create new subscription
        const publicKey = process.env.NEXT_PUBLIC_VAPID_PUBLIC_KEY!;
        subscription = await registration.pushManager.subscribe({
          userVisibleOnly: true,
          applicationServerKey: urlBase64ToUint8Array(publicKey),
        });
      }

      // Send subscription to server
      const response = await fetch('/api/push/subscribe', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(subscription),
      });

      if (!response.ok) {
        throw new Error('Failed to subscribe');
      }

      setIsSubscribed(true);
      toast.success('Subscribed to push notifications!');
      return true;
    } catch (error) {
      console.error('Error subscribing to push notifications:', error);
      toast.error('Failed to subscribe to notifications');
      return false;
    } finally {
      setIsLoading(false);
    }
  }, [isSupported, permission, requestPermission]);

  const unsubscribe = useCallback(async (): Promise<boolean> => {
    if (!isSupported) return false;

    setIsLoading(true);

    try {
      const registration = await navigator.serviceWorker.ready;
      const subscription = await registration.pushManager.getSubscription();

      if (subscription) {
        await subscription.unsubscribe();
        
        // Notify server
        await fetch('/api/push/unsubscribe', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ endpoint: subscription.endpoint }),
        });

        setIsSubscribed(false);
        toast.success('Unsubscribed from push notifications');
        return true;
      }
      
      return false;
    } catch (error) {
      console.error('Error unsubscribing from push notifications:', error);
      toast.error('Failed to unsubscribe from notifications');
      return false;
    } finally {
      setIsLoading(false);
    }
  }, [isSupported]);

  return {
    isSupported,
    permission,
    isSubscribed,
    isLoading,
    requestPermission,
    subscribe,
    unsubscribe,
    checkSubscription,
  };
}

// Helper function
function urlBase64ToUint8Array(base64String: string): Uint8Array {
  const padding = '='.repeat((4 - (base64String.length % 4)) % 4);
  const base64 = (base64String + padding)
    .replace(/\-/g, '+')
    .replace(/_/g, '/');

  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);

  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }

  return outputArray;
}

// components/PushPermissionPrompt.tsx
'use client';

import { useState } from 'react';
import { Bell, BellOff, Check } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog';
import { usePushNotifications } from '@/hooks/usePushNotifications';

interface PushPermissionPromptProps {
  open: boolean;
  onOpenChange: (open: boolean) => void;
}

export function PushPermissionPrompt({
  open,
  onOpenChange,
}: PushPermissionPromptProps) {
  const {
    isSupported,
    isSubscribed,
    isLoading,
    subscribe,
    unsubscribe,
  } = usePushNotifications();

  const [step, setStep] = useState<'intro' | 'enabled'>('intro');

  const handleEnable = async () => {
    const success = await subscribe();
    if (success) {
      setStep('enabled');
    }
  };

  const handleDisable = async () => {
    await unsubscribe();
    setStep('intro');
  };

  if (!isSupported) {
    return (
      <Dialog open={open} onOpenChange={onOpenChange}>
        <DialogContent>
          <DialogHeader>
            <DialogTitle>Notifications Not Supported</DialogTitle>
            <DialogDescription>
              Your browser doesn't support push notifications. Please use a modern browser like Chrome, Firefox, or Edge.
            </DialogDescription>
          </DialogHeader>
          <DialogFooter>
            <Button onClick={() => onOpenChange(false)}>Close</Button>
          </DialogFooter>
        </DialogContent>
      </Dialog>
    );
  }

  return (
    <Dialog open={open} onOpenChange={onOpenChange}>
      <DialogContent>
        {step === 'intro' ? (
          <>
            <DialogHeader>
              <div className="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-full bg-blue-100">
                <Bell className="h-6 w-6 text-blue-600" />
              </div>
              <DialogTitle>Stay Updated</DialogTitle>
              <DialogDescription>
                Enable push notifications to receive important updates, messages, and alerts directly on your device.
              </DialogDescription>
            </DialogHeader>

            <div className="space-y-4 py-4">
              <div className="space-y-2">
                <h4 className="font-medium">What you'll get:</h4>
                <ul className="space-y-2 text-sm text-gray-600">
                  <li className="flex items-start">
                    <Check className="mr-2 h-4 w-4 text-green-500 mt-0.5" />
                    <span>Real-time message notifications</span>
                  </li>
                  <li className="flex items-start">
                    <Check className="mr-2 h-4 w-4 text-green-500 mt-0.5" />
                    <span>Important security alerts</span>
                  </li>
                  <li className="flex items-start">
                    <Check className="mr-2 h-4 w-4 text-green-500 mt-0.5" />
                    <span>Updates on your activity</span>
                  </li>
                </ul>
              </div>

              <div className="rounded-lg bg-gray-50 p-4">
                <p className="text-sm text-gray-600">
                  You can always manage notifications in your browser settings or from the app preferences.
                </p>
              </div>
            </div>

            <DialogFooter>
              <Button
                variant="outline"
                onClick={() => onOpenChange(false)}
              >
                Not Now
              </Button>
              <Button
                onClick={handleEnable}
                disabled={isLoading}
                className="bg-blue-500 hover:bg-blue-600"
              >
                {isLoading ? 'Enabling...' : 'Enable Notifications'}
              </Button>
            </DialogFooter>
          </>
        ) : (
          <>
            <DialogHeader>
              <div className="mx-auto mb-4 flex h-12 w-12 items-center justify-center rounded-full bg-green-100">
                <Bell className="h-6 w-6 text-green-600" />
              </div>
              <DialogTitle>Notifications Enabled!</DialogTitle>
              <DialogDescription>
                You'll now receive push notifications. You can manage your preferences anytime.
              </DialogDescription>
            </DialogHeader>

            <DialogFooter>
              <Button
                variant="outline"
                onClick={handleDisable}
                disabled={isLoading}
              >
                <BellOff className="mr-2 h-4 w-4" />
                Disable
              </Button>
              <Button onClick={() => onOpenChange(false)}>
                Done
              </Button>
            </DialogFooter>
          </>
        )}
      </DialogContent>
    </Dialog>
  );
}
```

---

## 5. IN-APP NOTIFICATIONS

### 5.1 Notification Types Table

| Type | Persistence | UI Location | Auto-dismiss | Interactive | Best For |
|------|-------------|-------------|--------------|-------------|----------|
| Toast | No (5-10s) | Bottom/Corner | Yes (5s) | Limited | Success/error messages |
| Banner | Session | Top | Optional | Yes | System messages |
| Bell/Dropdown | Yes (DB) | Header | No | Yes | User notifications |
| Modal | No | Center | No | Yes | Critical actions |
| Inline | Context | In content | No | Yes | Form feedback |

### 5.2 Toast System (Sonner)

```typescript
// lib/toast.ts
import { toast, ToastOptions } from 'sonner';

// Custom toast presets
export const toastPresets = {
  success: (message: string, options?: ToastOptions) => {
    return toast.success(message, {
      icon: '✅',
      duration: 5000,
      ...options,
    });
  },
  
  error: (message: string, options?: ToastOptions) => {
    return toast.error(message, {
      icon: '❌',
      duration: 8000,
      ...options,
    });
  },
  
  info: (message: string, options?: ToastOptions) => {
    return toast.info(message, {
      icon: 'ℹ️',
      duration: 4000,
      ...options,
    });
  },
  
  warning: (message: string, options?: ToastOptions) => {
    return toast.warning(message, {
      icon: '⚠️',
      duration: 6000,
      ...options,
    });
  },
  
  promise: <T>(
    promise: Promise<T>,
    options: {
      loading: string;
      success: string | ((data: T) => string);
      error: string | ((error: any) => string);
      finally?: () => void;
    }
  ) => {
    return toast.promise(promise, {
      loading: options.loading,
      success: options.success,
      error: options.error,
      finally: options.finally,
    });
  },
  
  action: (
    message: string,
    actions: {
      label: string;
      onClick: () => void;
      variant?: 'default' | 'destructive';
    }[]
  ) => {
    return toast(message, {
      action: {
        label: actions[0].label,
        onClick: actions[0].onClick,
      },
    });
  },
};

// Custom Toaster component for global styling
export function Toaster() {
  return (
    <ToastProvider>
      <ToasterComponent
        position="bottom-right"
        toastOptions={{
          className: 'group toast',
          duration: 5000,
          classNames: {
            toast: 'group toast group-[.toaster]:bg-background group-[.toaster]:text-foreground group-[.toaster]:border-border group-[.toaster]:shadow-lg',
            description: 'group-[.toast]:text-muted-foreground',
            actionButton: 'group-[.toast]:bg-primary group-[.toast]:text-primary-foreground',
            cancelButton: 'group-[.toast]:bg-muted group-[.toast]:text-muted-foreground',
          },
        }}
      />
    </ToastProvider>
  );
}

// Hook for custom toast actions
export function useToast() {
  const showToast = (type: keyof typeof toastPresets, message: string, options?: ToastOptions) => {
    return toastPresets[type](message, options);
  };

  const dismiss = (id?: string | number) => {
    toast.dismiss(id);
  };

  const dismissAll = () => {
    toast.dismiss();
  };

  return {
    success: (message: string, options?: ToastOptions) => toastPresets.success(message, options),
    error: (message: string, options?: ToastOptions) => toastPresets.error(message, options),
    info: (message: string, options?: ToastOptions) => toastPresets.info(message, options),
    warning: (message: string, options?: ToastOptions) => toastPresets.warning(message, options),
    promise: toastPresets.promise,
    action: toastPresets.action,
    dismiss,
    dismissAll,
    showToast,
  };
}
```

### 5.3 Notification Center

```typescript
// components/NotificationCenter.tsx
'use client';

import { useState, useEffect } from 'react';
import { Bell, Check, ChevronRight, Loader2 } from 'lucide-react';
import { Button } from '@/components/ui/button';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuGroup,
  DropdownMenuItem,
  DropdownMenuLabel,
  DropdownMenuSeparator,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { ScrollArea } from '@/components/ui/scroll-area';
import { cn } from '@/lib/utils';
import { useNotifications } from '@/hooks/useNotifications';

export function NotificationCenter() {
  const {
    notifications,
    unreadCount,
    isLoading,
    markAsRead,
    markAllAsRead,
    loadMore,
    hasMore,
  } = useNotifications();

  const [open, setOpen] = useState(false);

  // Format time ago
  const timeAgo = (date: Date) => {
    const seconds = Math.floor((new Date().getTime() - date.getTime()) / 1000);
    
    if (seconds < 60) return 'Just now';
    if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
    if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
    if (seconds < 604800) return `${Math.floor(seconds / 86400)}d ago`;
    return new Date(date).toLocaleDateString();
  };

  const handleNotificationClick = async (notificationId: string, url?: string) => {
    await markAsRead(notificationId);
    
    if (url) {
      window.location.href = url;
    }
    
    setOpen(false);
  };

  return (
    <DropdownMenu open={open} onOpenChange={setOpen}>
      <DropdownMenuTrigger asChild>
        <Button
          variant="ghost"
          size="icon"
          className="relative"
        >
          <Bell className="h-5 w-5" />
          {unreadCount > 0 && (
            <span className="absolute -top-1 -right-1 flex h-5 w-5 items-center justify-center rounded-full bg-red-500 text-xs font-medium text-white">
              {unreadCount > 9 ? '9+' : unreadCount}
            </span>
          )}
        </Button>
      </DropdownMenuTrigger>
      
      <DropdownMenuContent className="w-96" align="end">
        <DropdownMenuLabel className="flex items-center justify-between">
          <span>Notifications</span>
          {unreadCount > 0 && (
            <Button
              variant="ghost"
              size="sm"
              onClick={markAllAsRead}
              className="h-auto p-0 text-xs"
            >
              <Check className="mr-1 h-3 w-3" />
              Mark all read
            </Button>
          )}
        </DropdownMenuLabel>
        
        <DropdownMenuSeparator />
        
        <ScrollArea className="h-96">
          <DropdownMenuGroup>
            {isLoading && notifications.length === 0 ? (
              <div className="flex justify-center p-4">
                <Loader2 className="h-6 w-6 animate-spin text-gray-400" />
              </div>
            ) : notifications.length === 0 ? (
              <div className="p-4 text-center text-sm text-gray-500">
                No notifications yet
              </div>
            ) : (
              <>
                {notifications.map((notification) => (
                  <DropdownMenuItem
                    key={notification.id}
                    className={cn(
                      'flex flex-col items-start p-3 cursor-pointer',
                      !notification.readAt && 'bg-blue-50'
                    )}
                    onClick={() => handleNotificationClick(
                      notification.id,
                      notification.data?.url
                    )}
                  >
                    <div className="flex w-full items-start justify-between">
                      <div className="flex-1">
                        <p className="font-medium text-sm">
                          {notification.title}
                        </p>
                        <p className="text-sm text-gray-600 mt-1">
                          {notification.body}
                        </p>
                        <div className="flex items-center justify-between mt-2">
                          <span className="text-xs text-gray-400">
                            {timeAgo(new Date(notification.createdAt))}
                          </span>
                          {notification.data?.action && (
                            <span className="inline-flex items-center text-xs text-blue-500">
                              {notification.data.action}
                              <ChevronRight className="ml-1 h-3 w-3" />
                            </span>
                          )}
                        </div>
                      </div>
                      
                      {!notification.readAt && (
                        <div className="ml-2 h-2 w-2 rounded-full bg-blue-500" />
                      )}
                    </div>
                    
                    {notification.data?.metadata && (
                      <div className="mt-2 text-xs text-gray-500">
                        {notification.data.metadata}
                      </div>
                    )}
                  </DropdownMenuItem>
                ))}
                
                {hasMore && (
                  <div className="p-2 text-center">
                    <Button
                      variant="ghost"
                      size="sm"
                      onClick={loadMore}
                      disabled={isLoading}
                    >
                      {isLoading ? 'Loading...' : 'Load more'}
                    </Button>
                  </div>
                )}
              </>
            )}
          </DropdownMenuGroup>
        </ScrollArea>
        
        <DropdownMenuSeparator />
        
        <div className="p-2">
          <Button
            variant="ghost"
            className="w-full justify-center text-sm"
            asChild
          >
            <a href="/notifications">View all notifications</a>
          </Button>
        </div>
      </DropdownMenuContent>
    </DropdownMenu>
  );
}

// Hook for notifications
// hooks/useNotifications.ts
import { useState, useEffect, useCallback } from 'react';
import { useSocket } from '@/lib/socket';

interface Notification {
  id: string;
  userId: string;
  type: string;
  title: string;
  body: string;
  data?: Record<string, any>;
  readAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}

export function useNotifications() {
  const [notifications, setNotifications] = useState<Notification[]>([]);
  const [unreadCount, setUnreadCount] = useState(0);
  const [isLoading, setIsLoading] = useState(true);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);
  
  const { socket, isConnected } = useSocket();

  const fetchNotifications = useCallback(async (pageNum = 1) => {
    setIsLoading(true);
    
    try {
      const response = await fetch(`/api/notifications?page=${pageNum}&limit=20`);
      const data = await response.json();
      
      if (pageNum === 1) {
        setNotifications(data.notifications);
      } else {
        setNotifications(prev => [...prev, ...data.notifications]);
      }
      
      setUnreadCount(data.unreadCount);
      setHasMore(data.hasMore);
      setPage(pageNum);
    } catch (error) {
      console.error('Failed to fetch notifications:', error);
    } finally {
      setIsLoading(false);
    }
  }, []);

  const markAsRead = useCallback(async (notificationId: string) => {
    try {
      await fetch(`/api/notifications/${notificationId}/read`, {
        method: 'POST',
      });
      
      setNotifications(prev =>
        prev.map(n =>
          n.id === notificationId ? { ...n, readAt: new Date() } : n
        )
      );
      
      setUnreadCount(prev => Math.max(0, prev - 1));
    } catch (error) {
      console.error('Failed to mark as read:', error);
    }
  }, []);

  const markAllAsRead = useCallback(async () => {
    try {
      await fetch('/api/notifications/read-all', {
        method: 'POST',
      });
      
      setNotifications(prev =>
        prev.map(n => ({ ...n, readAt: new Date() }))
      );
      
      setUnreadCount(0);
    } catch (error) {
      console.error('Failed to mark all as read:', error);
    }
  }, []);

  const loadMore = useCallback(() => {
    fetchNotifications(page + 1);
  }, [fetchNotifications, page]);

  // Listen for new notifications via WebSocket
  useEffect(() => {
    if (!socket || !isConnected) return;

    const handleNewNotification = (notification: Notification) => {
      setNotifications(prev => [notification, ...prev]);
      setUnreadCount(prev => prev + 1);
    };

    socket.on('notification:new', handleNewNotification);

    return () => {
      socket.off('notification:new', handleNewNotification);
    };
  }, [socket, isConnected]);

  // Initial fetch
  useEffect(() => {
    fetchNotifications();
  }, [fetchNotifications]);

  return {
    notifications,
    unreadCount,
    isLoading,
    hasMore,
    markAsRead,
    markAllAsRead,
    loadMore,
    refetch: () => fetchNotifications(1),
  };
}
```

### 5.4 Database Schema for Notifications

```prisma
// Prisma schema
model Notification {
  id        String   @id @default(cuid())
  userId    String
  
  // Content
  type      String   // 'info', 'success', 'warning', 'error', 'message', 'system'
  title     String
  body      String   @db.Text
  data      Json?    // Additional metadata
  
  // Status
  readAt    DateTime?
  archivedAt DateTime?
  
  // Timestamps
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  // Relations
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Indexes
  @@index([userId])
  @@index([userId, readAt])
  @@index([userId, createdAt])
  @@map("notifications")
}

model PushSubscription {
  id        String   @id @default(cuid())
  userId    String
  endpoint  String   @unique
  p256dh    String
  auth      String
  expiresAt DateTime?
  lastSentAt DateTime?
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId])
  @@map("push_subscriptions")
}
```

### 5.5 Real-time Updates with WebSocket

```typescript
// lib/socket.ts
import { useEffect, useRef, useState } from 'react';
import { io, Socket } from 'socket.io-client';

let socketInstance: Socket | null = null;

export function useSocket() {
  const [isConnected, setIsConnected] = useState(false);
  const socketRef = useRef<Socket | null>(null);

  useEffect(() => {
    // Initialize socket only once
    if (!socketInstance) {
      socketInstance = io(process.env.NEXT_PUBLIC_WS_URL || '', {
        path: '/api/socket',
        transports: ['websocket', 'polling'],
        reconnection: true,
        reconnectionAttempts: 5,
        reconnectionDelay: 1000,
      });

      socketInstance.on('connect', () => {
        console.log('Socket connected');
        setIsConnected(true);
      });

      socketInstance.on('disconnect', () => {
        console.log('Socket disconnected');
        setIsConnected(false);
      });

      socketInstance.on('error', (error) => {
        console.error('Socket error:', error);
      });
    }

    socketRef.current = socketInstance;

    return () => {
      // Don't disconnect on unmount - keep connection alive
      // socketInstance?.disconnect();
    };
  }, []);

  const emit = (event: string, data?: any) => {
    socketRef.current?.emit(event, data);
  };

  const on = (event: string, callback: (...args: any[]) => void) => {
    socketRef.current?.on(event, callback);
  };

  const off = (event: string, callback?: (...args: any[]) => void) => {
    socketRef.current?.off(event, callback);
  };

  return {
    socket: socketRef.current,
    isConnected,
    emit,
    on,
    off,
  };
}

// API route for WebSocket
// app/api/socket/route.ts
import { Server as NetServer } from 'http';
import { Server as SocketIOServer } from 'socket.io';
import { NextApiRequest } from 'next';
import { NextRequest } from 'next/server';

const SocketHandler = (req: NextRequest) => {
  if (global.io) {
    console.log('Socket is already running');
  } else {
    console.log('Socket is initializing');
    
    const httpServer = (req as any).socket?.server;
    if (!httpServer) {
      return new Response('Server not found', { status: 500 });
    }
    
    const io = new SocketIOServer(httpServer, {
      path: '/api/socket',
      cors: {
        origin: process.env.NEXT_PUBLIC_APP_URL,
        methods: ['GET', 'POST'],
      },
    });
    
    io.on('connection', (socket) => {
      console.log('New client connected:', socket.id);
      
      // Join user room for private notifications
      socket.on('join', (userId: string) => {
        socket.join(`user:${userId}`);
        console.log(`Socket ${socket.id} joined user:${userId}`);
      });
      
      // Handle custom events
      socket.on('notification:send', (data) => {
        const { userId, notification } = data;
        io.to(`user:${userId}`).emit('notification:new', notification);
      });
      
      socket.on('disconnect', () => {
        console.log('Client disconnected:', socket.id);
      });
    });
    
    global.io = io;
  }
  
  return new Response('Socket initialized', { status: 200 });
};

export { SocketHandler as GET, SocketHandler as POST };
```

---

## 6. SMS NOTIFICATIONS

### 6.1 SMS Providers Comparison

| Provider | Free Tier | Coverage | Price/SMS | Two-way | Best For |
|----------|-----------|----------|-----------|---------|----------|
| Twilio | Trial $15 | 🟢 Global | $0.0075 | ✅ | Enterprise, reliability |
| AWS SNS | ❌ None | 🟢 Global | $0.00645 | ❌ | AWS stack, cost-effective |
| Vonage | €2 trial | 🟢 Global | Varies | ✅ | International, features |
| Plivo | ❌ None | 🟢 Global | $0.005 | ✅ | High volume, API-first |
| Textbelt | 1/day | 🟢 Global | Free | ❌ | Testing, low volume |

### 6.2 Twilio Implementation

```typescript
// lib/sms/twilio.ts
import twilio from 'twilio';
import { RateLimiterMemory } from 'rate-limiter-flexible';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

// Rate limiter: 5 SMS per user per hour
const smsRateLimiter = new RateLimiterMemory({
  points: 5,
  duration: 60 * 60, // 1 hour
});

interface SMSSendOptions {
  to: string; // E.164 format: +1234567890
  body: string;
  from?: string;
  mediaUrl?: string[];
  maxPrice?: number;
  validityPeriod?: number; // seconds
  statusCallback?: string;
}

interface SMSResult {
  success: boolean;
  messageId?: string;
  error?: string;
  price?: number;
  status?: string;
}

/**
 * Send SMS using Twilio
 */
export async function sendSMS(
  options: SMSSendOptions,
  userId?: string
): Promise<SMSResult> {
  try {
    // Rate limiting check
    if (userId) {
      try {
        await smsRateLimiter.consume(`sms:${userId}`);
      } catch (error) {
        throw new Error('SMS rate limit exceeded. Please try again later.');
      }
    }

    // Validate phone number
    if (!isValidE164(options.to)) {
      throw new Error('Invalid phone number format. Use E.164: +1234567890');
    }

    // Validate message length
    if (options.body.length > 1600) {
      throw new Error('SMS message too long (max 1600 chars)');
    }

    const message = await client.messages.create({
      body: options.body,
      to: options.to,
      from: options.from || process.env.TWILIO_PHONE_NUMBER!,
      mediaUrl: options.mediaUrl,
      maxPrice: options.maxPrice,
      validityPeriod: options.validityPeriod,
      statusCallback: options.statusCallback || `${process.env.APP_URL}/api/sms/callback`,
    });

    return {
      success: true,
      messageId: message.sid,
      price: parseFloat(message.price || '0'),
      status: message.status,
    };
  } catch (error: any) {
    console.error('SMS send error:', error);
    
    // Handle specific Twilio errors
    if (error.code === 21211) {
      return {
        success: false,
        error: 'Invalid phone number',
      };
    }
    
    if (error.code === 21610) {
      return {
        success: false,
        error: 'Phone number is not SMS-capable',
      };
    }
    
    return {
      success: false,
      error: error.message || 'Failed to send SMS',
    };
  }
}

/**
 * Send OTP via SMS
 */
export async function sendOTP(
  phoneNumber: string,
  code: string,
  expiryMinutes = 10,
  userId?: string
): Promise<SMSResult> {
  const message = `Your verification code: ${code}. Valid for ${expiryMinutes} minutes.`;
  
  return sendSMS({
    to: phoneNumber,
    body: message,
    from: process.env.TWILIO_VERIFY_SERVICE_SID ? undefined : process.env.TWILIO_PHONE_NUMBER,
  }, userId);
}

/**
 * Verify OTP using Twilio Verify service
 */
export async function verifyOTP(
  phoneNumber: string,
  code: string
): Promise<{ success: boolean; error?: string }> {
  try {
    const verificationCheck = await client.verify.v2
      .services(process.env.TWILIO_VERIFY_SERVICE_SID!)
      .verificationChecks.create({
        to: phoneNumber,
        code: code,
      });

    return {
      success: verificationCheck.status === 'approved',
    };
  } catch (error: any) {
    console.error('OTP verification error:', error);
    return {
      success: false,
      error: error.message || 'Verification failed',
    };
  }
}

/**
 * Send bulk SMS (with rate limiting)
 */
export async function sendBulkSMS(
  recipients: Array<{ phoneNumber: string; message?: string }>,
  template: string,
  batchSize = 10
): Promise<Array<{ phoneNumber: string; result: SMSResult }>> {
  const results: Array<{ phoneNumber: string; result: SMSResult }> = [];

  for (let i = 0; i < recipients.length; i += batchSize) {
    const batch = recipients.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(async ({ phoneNumber, message }) => {
        const body = message || template;
        const result = await sendSMS({
          to: phoneNumber,
          body,
        });
        
        return { phoneNumber, result };
      })
    );

    results.push(...batchResults);

    // Rate limiting between batches
    if (i + batchSize < recipients.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }

  return results;
}

// Helper function
function isValidE164(phoneNumber: string): boolean {
  const regex = /^\+[1-9]\d{1,14}$/;
  return regex.test(phoneNumber);
}

// Handle SMS delivery status callbacks
export async function handleSMSStatusCallback(
  messageSid: string,
  status: string
) {
  console.log(`SMS ${messageSid} status: ${status}`);
  
  // Update database with delivery status
  // This would typically update a notification record
}
```

---

## 7. NOTIFICATION PREFERENCES

### 7.1 Preference Matrix

| Notification Type | Email | Push | In-App | SMS | Default |
|-------------------|-------|------|--------|-----|---------|
| Security alerts | ✅ Required | Optional | ✅ Required | Optional | All ON |
| New messages | Default ON | Default ON | ✅ Required | OFF | Push + In-App |
| Marketing | Default OFF | OFF | OFF | OFF | All OFF |
| Product updates | Default ON | OFF | ✅ Required | OFF | Email + In-App |
| Payment reminders | Default ON | Optional | ✅ Required | Optional | Email + In-App |
| Social mentions | Default ON | Default ON | ✅ Required | OFF | Push + In-App |
| System maintenance | ✅ Required | ✅ Required | ✅ Required | ✅ Required | All ON |
| Newsletters | Default OFF | OFF | OFF | OFF | All OFF |

### 7.2 Preferences Schema

```prisma
model NotificationPreference {
  id        String   @id @default(cuid())
  userId    String   @unique
  
  // Channel preferences
  emailEnabled     Boolean @default(true)
  pushEnabled      Boolean @default(true)
  inAppEnabled     Boolean @default(true)
  smsEnabled       Boolean @default(false)
  
  // Type-specific preferences
  emailTypes       Json    @default("{}") // { security: true, marketing: false, ... }
  pushTypes        Json    @default("{}")
  inAppTypes       Json    @default("{}")
  smsTypes         Json    @default("{}")
  
  // Frequency settings
  quietHoursStart  String? // "22:00"
  quietHoursEnd    String? // "08:00"
  timezone         String  @default("UTC")
  
  // Global overrides
  doNotDisturb     Boolean @default(false)
  pauseUntil       DateTime?
  
  // Relationships
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Timestamps
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("notification_preferences")
}
```

### 7.3 Preferences UI Component

```typescript
// components/NotificationPreferences.tsx
'use client';

import { useState, useEffect } from 'react';
import { Bell, Mail, Smartphone, MessageSquare, Moon } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Switch } from '@/components/ui/switch';
import { Label } from '@/components/ui/label';
import { Separator } from '@/components/ui/separator';
import { toast } from 'sonner';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

interface NotificationType {
  id: string;
  label: string;
  description: string;
  defaultChannels: {
    email: boolean;
    push: boolean;
    inApp: boolean;
    sms: boolean;
  };
}

interface Preferences {
  emailEnabled: boolean;
  pushEnabled: boolean;
  inAppEnabled: boolean;
  smsEnabled: boolean;
  emailTypes: Record<string, boolean>;
  pushTypes: Record<string, boolean>;
  inAppTypes: Record<string, boolean>;
  smsTypes: Record<string, boolean>;
  quietHoursStart: string;
  quietHoursEnd: string;
  timezone: string;
  doNotDisturb: boolean;
}

const notificationTypes: NotificationType[] = [
  {
    id: 'security',
    label: 'Security Alerts',
    description: 'Important security notifications and account activity',
    defaultChannels: { email: true, push: true, inApp: true, sms: true },
  },
  {
    id: 'messages',
    label: 'New Messages',
    description: 'When you receive new messages or comments',
    defaultChannels: { email: true, push: true, inApp: true, sms: false },
  },
  {
    id: 'marketing',
    label: 'Marketing Emails',
    description: 'Product updates, promotions, and newsletters',
    defaultChannels: { email: false, push: false, inApp: false, sms: false },
  },
  {
    id: 'payment',
    label: 'Payment Reminders',
    description: 'Billing reminders and payment confirmations',
    defaultChannels: { email: true, push: true, inApp: true, sms: false },
  },
  {
    id: 'social',
    label: 'Social Mentions',
    description: 'When someone mentions or follows you',
    defaultChannels: { email: true, push: true, inApp: true, sms: false },
  },
];

const timezones = [
  'UTC',
  'America/New_York',
  'America/Chicago',
  'America/Denver',
  'America/Los_Angeles',
  'Europe/London',
  'Europe/Paris',
  'Asia/Tokyo',
  'Australia/Sydney',
];

export function NotificationPreferences() {
  const [preferences, setPreferences] = useState<Preferences>({
    emailEnabled: true,
    pushEnabled: true,
    inAppEnabled: true,
    smsEnabled: false,
    emailTypes: {},
    pushTypes: {},
    inAppTypes: {},
    smsTypes: {},
    quietHoursStart: '22:00',
    quietHoursEnd: '08:00',
    timezone: 'UTC',
    doNotDisturb: false,
  });
  const [isLoading, setIsLoading] = useState(true);
  const [isSaving, setIsSaving] = useState(false);

  useEffect(() => {
    loadPreferences();
  }, []);

  const loadPreferences = async () => {
    try {
      const response = await fetch('/api/notifications/preferences');
      const data = await response.json();
      setPreferences(data);
    } catch (error) {
      console.error('Failed to load preferences:', error);
      toast.error('Failed to load preferences');
    } finally {
      setIsLoading(false);
    }
  };

  const savePreferences = async () => {
    setIsSaving(true);
    
    try {
      const response = await fetch('/api/notifications/preferences', {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(preferences),
      });

      if (!response.ok) throw new Error('Save failed');
      
      toast.success('Preferences saved');
    } catch (error) {
      console.error('Failed to save preferences:', error);
      toast.error('Failed to save preferences');
    } finally {
      setIsSaving(false);
    }
  };

  const updateChannelPreference = (channel: keyof Preferences, value: boolean) => {
    setPreferences(prev => ({
      ...prev,
      [channel]: value,
    }));
  };

  const updateTypePreference = (
    channel: 'emailTypes' | 'pushTypes' | 'inAppTypes' | 'smsTypes',
    typeId: string,
    value: boolean
  ) => {
    setPreferences(prev => ({
      ...prev,
      [channel]: {
        ...prev[channel],
        [typeId]: value,
      },
    }));
  };

  const resetToDefaults = () => {
    setPreferences({
      emailEnabled: true,
      pushEnabled: true,
      inAppEnabled: true,
      smsEnabled: false,
      emailTypes: {},
      pushTypes: {},
      inAppTypes: {},
      smsTypes: {},
      quietHoursStart: '22:00',
      quietHoursEnd: '08:00',
      timezone: 'UTC',
      doNotDisturb: false,
    });
  };

  if (isLoading) {
    return (
      <Card>
        <CardContent className="flex justify-center p-8">
          <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500" />
        </CardContent>
      </Card>
    );
  }

  return (
    <div className="space-y-6">
      {/* Channel Preferences */}
      <Card>
        <CardHeader>
          <CardTitle>Notification Channels</CardTitle>
          <CardDescription>
            Choose how you want to receive notifications
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="flex items-center justify-between">
            <div className="space-y-0.5">
              <div className="flex items-center gap-2">
                <Mail className="h-4 w-4 text-gray-500" />
                <Label htmlFor="email-enabled">Email</Label>
              </div>
              <p className="text-sm text-gray-500">
                Receive notifications via email
              </p>
            </div>
            <Switch
              id="email-enabled"
              checked={preferences.emailEnabled}
              onCheckedChange={(checked) => updateChannelPreference('emailEnabled', checked)}
            />
          </div>
          
          <div className="flex items-center justify-between">
            <div className="space-y-0.5">
              <div className="flex items-center gap-2">
                <Bell className="h-4 w-4 text-gray-500" />
                <Label htmlFor="push-enabled">Push Notifications</Label>
              </div>
              <p className="text-sm text-gray-500">
                Receive notifications on your device
              </p>
            </div>
            <Switch
              id="push-enabled"
              checked={preferences.pushEnabled}
              onCheckedChange={(checked) => updateChannelPreference('pushEnabled', checked)}
            />
          </div>
          
          <div className="flex items-center justify-between">
            <div className="space-y-0.5">
              <div className="flex items-center gap-2">
                <Smartphone className="h-4 w-4 text-gray-500" />
                <Label htmlFor="inapp-enabled">In-App Notifications</Label>
              </div>
              <p className="text-sm text-gray-500">
                See notifications within the app
              </p>
            </div>
            <Switch
              id="inapp-enabled"
              checked={preferences.inAppEnabled}
              onCheckedChange={(checked) => updateChannelPreference('inAppEnabled', checked)}
            />
          </div>
          
          <div className="flex items-center justify-between">
            <div className="space-y-0.5">
              <div className="flex items-center gap-2">
                <MessageSquare className="h-4 w-4 text-gray-500" />
                <Label htmlFor="sms-enabled">SMS</Label>
              </div>
              <p className="text-sm text-gray-500">
                Receive notifications via text message
              </p>
            </div>
            <Switch
              id="sms-enabled"
              checked={preferences.smsEnabled}
              onCheckedChange={(checked) => updateChannelPreference('smsEnabled', checked)}
            />
          </div>
        </CardContent>
      </Card>

      {/* Notification Types */}
      <Card>
        <CardHeader>
          <CardTitle>Notification Types</CardTitle>
          <CardDescription>
            Customize which types of notifications you receive
          </CardDescription>
        </CardHeader>
        <CardContent>
          <div className="space-y-6">
            {notificationTypes.map((type) => (
              <div key={type.id}>
                <div className="flex items-start justify-between">
                  <div className="space-y-1">
                    <Label>{type.label}</Label>
                    <p className="text-sm text-gray-500">
                      {type.description}
                    </p>
                  </div>
                </div>
                
                <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mt-3">
                  {preferences.emailEnabled && (
                    <div className="flex items-center space-x-2">
                      <Switch
                        checked={preferences.emailTypes[type.id] ?? type.defaultChannels.email}
                        onCheckedChange={(checked) =>
                          updateTypePreference('emailTypes', type.id, checked)
                        }
                        id={`email-${type.id}`}
                      />
                      <Label htmlFor={`email-${type.id}`} className="text-sm">
                        Email
                      </Label>
                    </div>
                  )}
                  
                  {preferences.pushEnabled && (
                    <div className="flex items-center space-x-2">
                      <Switch
                        checked={preferences.pushTypes[type.id] ?? type.defaultChannels.push}
                        onCheckedChange={(checked) =>
                          updateTypePreference('pushTypes', type.id, checked)
                        }
                        id={`push-${type.id}`}
                      />
                      <Label htmlFor={`push-${type.id}`} className="text-sm">
                        Push
                      </Label>
                    </div>
                  )}
                  
                  {preferences.inAppEnabled && (
                    <div className="flex items-center space-x-2">
                      <Switch
                        checked={preferences.inAppTypes[type.id] ?? type.defaultChannels.inApp}
                        onCheckedChange={(checked) =>
                          updateTypePreference('inAppTypes', type.id, checked)
                        }
                        id={`inapp-${type.id}`}
                      />
                      <Label htmlFor={`inapp-${type.id}`} className="text-sm">
                        In-App
                      </Label>
                    </div>
                  )}
                  
                  {preferences.smsEnabled && (
                    <div className="flex items-center space-x-2">
                      <Switch
                        checked={preferences.smsTypes[type.id] ?? type.defaultChannels.sms}
                        onCheckedChange={(checked) =>
                          updateTypePreference('smsTypes', type.id, checked)
                        }
                        id={`sms-${type.id}`}
                      />
                      <Label htmlFor={`sms-${type.id}`} className="text-sm">
                        SMS
                      </Label>
                    </div>
                  )}
                </div>
                
                <Separator className="mt-4" />
              </div>
            ))}
          </div>
        </CardContent>
      </Card>

      {/* Quiet Hours */}
      <Card>
        <CardHeader>
          <CardTitle className="flex items-center gap-2">
            <Moon className="h-5 w-5" />
            Quiet Hours
          </CardTitle>
          <CardDescription>
            Pause notifications during specific times
          </CardDescription>
        </CardHeader>
        <CardContent className="space-y-4">
          <div className="flex items-center justify-between">
            <Label htmlFor="do-not-disturb">Do Not Disturb</Label>
            <Switch
              id="do-not-disturb"
              checked={preferences.doNotDisturb}
              onCheckedChange={(checked) =>
                setPreferences(prev => ({ ...prev, doNotDisturb: checked }))
              }
            />
          </div>
          
          {!preferences.doNotDisturb && (
            <>
              <div className="grid grid-cols-2 gap-4">
                <div className="space-y-2">
                  <Label htmlFor="quiet-hours-start">Start Time</Label>
                  <input
                    type="time"
                    id="quiet-hours-start"
                    value={preferences.quietHoursStart}
                    onChange={(e) =>
                      setPreferences(prev => ({
                        ...prev,
                        quietHoursStart: e.target.value,
                      }))
                    }
                    className="w-full p-2 border rounded-md"
                  />
                </div>
                
                <div className="space-y-2">
                  <Label htmlFor="quiet-hours-end">End Time</Label>
                  <input
                    type="time"
                    id="quiet-hours-end"
                    value={preferences.quietHoursEnd}
                    onChange={(e) =>
                      setPreferences(prev => ({
                        ...prev,
                        quietHoursEnd: e.target.value,
                      }))
                    }
                    className="w-full p-2 border rounded-md"
                  />
                </div>
              </div>
              
              <div className="space-y-2">
                <Label htmlFor="timezone">Timezone</Label>
                <Select
                  value={preferences.timezone}
                  onValueChange={(value) =>
                    setPreferences(prev => ({ ...prev, timezone: value }))
                  }
                >
                  <SelectTrigger>
                    <SelectValue placeholder="Select timezone" />
                  </SelectTrigger>
                  <SelectContent>
                    {timezones.map((tz) => (
                      <SelectItem key={tz} value={tz}>
                        {tz}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
              </div>
              
              <p className="text-sm text-gray-500">
                Notifications will be paused between {preferences.quietHoursStart} and {preferences.quietHoursEnd} ({preferences.timezone})
              </p>
            </>
          )}
        </CardContent>
      </Card>

      {/* Actions */}
      <div className="flex justify-between">
        <Button
          variant="outline"
          onClick={resetToDefaults}
        >
          Reset to Defaults
        </Button>
        <Button
          onClick={savePreferences}
          disabled={isSaving}
        >
          {isSaving ? 'Saving...' : 'Save Preferences'}
        </Button>
      </div>
    </div>
  );
}
```

---

## 8. NOTIFICATION SERVICE ARCHITECTURE

### 8.1 Architecture Diagram

```
User Action → Event Emitter → Notification Service
                                    ↓
                            [Middleware Layer]
                                    ↓
                    ┌───────────┼───────────┐
               Preferences    Throttling    Template
                  Check         Layer        Engine
                                    ↓
                            [Channel Router]
                                    ↓
                    ┌───────────┼───────────┐
                  Email        Push         SMS
                    ↓            ↓           ↓
                [Queue]      [Queue]     [Queue]
                    ↓            ↓           ↓
                Resend       Web Push     Twilio
                    ↓            ↓           ↓
              ┌─────┼─────┐     │           │
              │   Tracking  │     │           │
              │   Analytics │     │           │
              │   Logging   │     │           │
              └─────────────┘     └───────────┘
```

### 8.2 Unified Notification Service

```typescript
// lib/notifications/service.ts
import { sendEmail } from '@/lib/email/resend';
import { sendPushNotification } from '@/lib/push/web-push';
import { sendSMS } from '@/lib/sms/twilio';
import { prisma } from '@/lib/prisma';
import { WelcomeEmail } from '@/lib/email/templates/welcome';
import { PasswordResetEmail } from '@/lib/email/templates/password-reset';
import { renderAsync } from '@react-email/render';
import React from 'react';

export type NotificationType = 
  | 'welcome'
  | 'password_reset'
  | 'new_message'
  | 'payment_failed'
  | 'security_alert'
  | 'marketing'
  | 'system_update';

export interface NotificationData {
  userId: string;
  type: NotificationType;
  data: Record<string, any>;
  channels?: Array<'email' | 'push' | 'sms' | 'in_app'>;
  priority?: 'low' | 'medium' | 'high' | 'critical';
  delay?: number; // Delay in seconds
}

interface NotificationResult {
  success: boolean;
  notificationId: string;
  results: {
    email?: { success: boolean; messageId?: string; error?: string };
    push?: { success: boolean; sent: number; failed: number };
    sms?: { success: boolean; messageId?: string; error?: string };
    in_app?: { success: boolean; notificationId: string };
  };
}

export class NotificationService {
  private static instance: NotificationService;
  
  private constructor() {}
  
  public static getInstance(): NotificationService {
    if (!NotificationService.instance) {
      NotificationService.instance = new NotificationService();
    }
    return NotificationService.instance;
  }
  
  /**
   * Send notification to user
   */
  async sendNotification({
    userId,
    type,
    data,
    channels = ['email', 'push', 'in_app'],
    priority = 'medium',
    delay = 0,
  }: NotificationData): Promise<NotificationResult> {
    // Check user preferences
    const preferences = await this.getUserPreferences(userId);
    const allowedChannels = this.filterChannelsByPreferences(channels, preferences, type);
    
    if (allowedChannels.length === 0) {
      return {
        success: false,
        notificationId: '',
        results: {},
      };
    }
    
    // Check quiet hours
    if (this.isQuietHours(preferences)) {
      // Queue for later delivery
      return await this.queueNotification({
        userId,
        type,
        data,
        channels: allowedChannels,
        priority,
      });
    }
    
    // Create notification record
    const notification = await prisma.notification.create({
      data: {
        userId,
        type,
        title: this.getTemplate(type).title(data),
        body: this.getTemplate(type).body(data),
        data,
      },
    });
    
    const results: NotificationResult['results'] = {};
    
    // Send through each channel
    for (const channel of allowedChannels) {
      switch (channel) {
        case 'email':
          if (preferences.emailEnabled) {
            results.email = await this.sendEmailNotification(userId, type, data, notification.id);
          }
          break;
        
        case 'push':
          if (preferences.pushEnabled) {
            results.push = await this.sendPushNotification(userId, type, data);
          }
          break;
        
        case 'sms':
          if (preferences.smsEnabled) {
            results.sms = await this.sendSMSNotification(userId, type, data);
          }
          break;
        
        case 'in_app':
          if (preferences.inAppEnabled) {
            results.in_app = await this.createInAppNotification(notification);
          }
          break;
      }
    }
    
    // Update notification with delivery results
    await prisma.notification.update({
      where: { id: notification.id },
      data: {
        data: {
          ...data,
          deliveryResults: results,
        },
      },
    });
    
    return {
      success: Object.values(results).some(r => r?.success),
      notificationId: notification.id,
      results,
    };
  }
  
  /**
   * Send bulk notifications
   */
  async sendBulkNotifications(
    notifications: NotificationData[],
    batchSize = 100
  ): Promise<Array<NotificationResult>> {
    const results: NotificationResult[] = [];
    
    for (let i = 0; i < notifications.length; i += batchSize) {
      const batch = notifications.slice(i, i + batchSize);
      
      const batchResults = await Promise.all(
        batch.map(notification => this.sendNotification(notification))
      );
      
      results.push(...batchResults);
      
      // Rate limiting between batches
      if (i + batchSize < notifications.length) {
        await new Promise(resolve => setTimeout(resolve, 1000));
      }
    }
    
    return results;
  }
  
  /**
   * Get user preferences
   */
  private async getUserPreferences(userId: string) {
    const preferences = await prisma.notificationPreference.findUnique({
      where: { userId },
    });
    
    if (!preferences) {
      // Create default preferences
      return await prisma.notificationPreference.create({
        data: { userId },
      });
    }
    
    return preferences;
  }
  
  /**
   * Filter channels based on user preferences
   */
  private filterChannelsByPreferences(
    channels: NotificationData['channels'],
    preferences: any,
    type: NotificationType
  ): Array<'email' | 'push' | 'sms' | 'in_app'> {
    return (channels || []).filter(channel => {
      if (!preferences[`${channel}Enabled`]) return false;
      
      // Check type-specific preferences
      const typeKey = type.replace(/_/g, '');
      const typePrefs = preferences[`${channel}Types`] || {};
      
      return typePrefs[typeKey] !== false;
    }) as Array<'email' | 'push' | 'sms' | 'in_app'>;
  }
  
  /**
   * Check if current time is within quiet hours
   */
  private isQuietHours(preferences: any): boolean {
    if (preferences.doNotDisturb) return true;
    
    if (!preferences.quietHoursStart || !preferences.quietHoursEnd) {
      return false;
    }
    
    const now = new Date();
    const [startHour, startMinute] = preferences.quietHoursStart.split(':').map(Number);
    const [endHour, endMinute] = preferences.quietHoursEnd.split(':').map(Number);
    
    const startTime = new Date(now);
    startTime.setHours(startHour, startMinute, 0, 0);
    
    const endTime = new Date(now);
    endTime.setHours(endHour, endMinute, 0, 0);
    
    // Handle overnight quiet hours
    if (endTime < startTime) {
      return now >= startTime || now < endTime;
    }
    
    return now >= startTime && now < endTime;
  }
  
  /**
   * Queue notification for later delivery
   */
  private async queueNotification(data: NotificationData) {
    // Implementation depends on your job queue (BullMQ, Inngest, etc.)
    // This would add the notification to a delayed queue
    
    return {
      success: true,
      notificationId: 'queued',
      results: {},
    };
  }
  
  /**
   * Send email notification
   */
  private async sendEmailNotification(
    userId: string,
    type: NotificationType,
    data: Record<string, any>,
    notificationId: string
  ) {
    try {
      const user = await prisma.user.findUnique({
        where: { id: userId },
      });
      
      if (!user?.email) {
        return { success: false, error: 'User email not found' };
      }
      
      const template = this.getTemplate(type);
      const emailComponent = template.email(data);
      
      if (!emailComponent) {
        return { success: false, error: 'Email template not found' };
      }
      
      const emailHtml = await renderAsync(emailComponent);
      
      const result = await sendEmail({
        to: user.email,
        subject: template.title(data),
      }, emailComponent, userId);
      
      // Track email delivery
      if (result.success && result.messageId) {
        await prisma.notificationDelivery.create({
          data: {
            notificationId,
            channel: 'email',
            providerId: result.messageId,
            status: 'sent',
          },
        });
      }
      
      return result;
    } catch (error) {
      console.error('Email notification error:', error);
      return { success: false, error: String(error) };
    }
  }
  
  /**
   * Send push notification
   */
  private async sendPushNotification(
    userId: string,
    type: NotificationType,
    data: Record<string, any>
  ) {
    try {
      const template = this.getTemplate(type);
      
      const result = await sendPushNotification(userId, {
        title: template.title(data),
        body: template.body(data),
        data: {
          ...data,
          type,
          timestamp: Date.now(),
        },
      });
      
      return result;
    } catch (error) {
      console.error('Push notification error:', error);
      return { success: false, sent: 0, failed: 1 };
    }
  }
  
  /**
   * Send SMS notification
   */
  private async sendSMSNotification(
    userId: string,
    type: NotificationType,
    data: Record<string, any>
  ) {
    try {
      const user = await prisma.user.findUnique({
        where: { id: userId },
      });
      
      if (!user?.phone) {
        return { success: false, error: 'User phone not found' };
      }
      
      const template = this.getTemplate(type);
      const message = template.sms(data);
      
      if (!message) {
        return { success: false, error: 'SMS template not found' };
      }
      
      const result = await sendSMS({
        to: user.phone,
        body: message,
      }, userId);
      
      return result;
    } catch (error) {
      console.error('SMS notification error:', error);
      return { success: false, error: String(error) };
    }
  }
  
  /**
   * Create in-app notification
   */
  private async createInAppNotification(notification: any) {
    // Notification already created in database
    // Emit real-time event if WebSocket is connected
    
    try {
      // Emit via WebSocket
      if (global.io) {
        global.io.to(`user:${notification.userId}`).emit('notification:new', notification);
      }
      
      return { success: true, notificationId: notification.id };
    } catch (error) {
      console.error('In-app notification error:', error);
      return { success: false, error: String(error) };
    }
  }
  
  /**
   * Get template for notification type
   */
  private getTemplate(type: NotificationType) {
    const templates = {
      welcome: {
        title: (data: any) => `Welcome to ${data.appName || 'Our App'}!`,
        body: (data: any) => `Hi ${data.name}, welcome to our platform!`,
        email: (data: any) => <WelcomeEmail name={data.name} verifyUrl={data.verifyUrl} />,
        sms: (data: any) => `Welcome to ${data.appName}! Verify your email: ${data.verifyUrl}`,
      },
      password_reset: {
        title: () => 'Reset Your Password',
        body: () => 'Click the link to reset your password',
        email: (data: any) => (
          <PasswordResetEmail 
            name={data.name} 
            resetUrl={data.resetUrl} 
            expiryMinutes={data.expiryMinutes || 15}
          />
        ),
        sms: (data: any) => `Reset your password: ${data.resetUrl}. Expires in ${data.expiryMinutes || 15} minutes.`,
      },
      // Add more templates...
    };
    
    return templates[type] || templates.welcome;
  }
}

// Export singleton instance
export const notificationService = NotificationService.getInstance();
```

---

## 9. DELIVERY TRACKING & ANALYTICS

### 9.1 Tracking Implementation

```typescript
// lib/notifications/tracking.ts
import { prisma } from '@/lib/prisma';

export interface DeliveryStatus {
  notificationId: string;
  channel: 'email' | 'push' | 'sms' | 'in_app';
  providerId?: string; // External provider ID
  status: 'sent' | 'delivered' | 'opened' | 'clicked' | 'failed' | 'bounced';
  error?: string;
  metadata?: Record<string, any>;
}

export async function trackDelivery(status: DeliveryStatus) {
  try {
    await prisma.notificationDelivery.create({
      data: {
        notificationId: status.notificationId,
        channel: status.channel,
        providerId: status.providerId,
        status: status.status,
        error: status.error,
        metadata: status.metadata,
      },
    });
    
    // Update notification status
    if (status.status === 'opened' || status.status === 'clicked') {
      await prisma.notification.update({
        where: { id: status.notificationId },
        data: {
          data: {
            set: {
              lastInteraction: new Date().toISOString(),
              interactionType: status.status,
            },
          },
        },
      });
    }
  } catch (error) {
    console.error('Failed to track delivery:', error);
  }
}

// Email open tracking pixel
export function generateTrackingPixel(notificationId: string): string {
  const pixelUrl = `${process.env.APP_URL}/api/notifications/${notificationId}/track/open`;
  
  return `
    <img 
      src="${pixelUrl}" 
      width="1" 
      height="1" 
      style="display:none;width:1px;height:1px;" 
      alt=""
    />
  `;
}

// Click tracking wrapper
export function trackClick(notificationId: string, url: string): string {
  const trackingUrl = `${process.env.APP_URL}/api/notifications/${notificationId}/track/click`;
  
  return `${trackingUrl}?redirect=${encodeURIComponent(url)}`;
}

// API route for tracking
// app/api/notifications/[id]/track/[type]/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string; type: string } }
) {
  const { id: notificationId, type } = params;
  
  try {
    // Record the interaction
    await trackDelivery({
      notificationId,
      channel: 'email', // Assuming email for open/click tracking
      status: type as any, // 'opened' or 'clicked'
      metadata: {
        userAgent: request.headers.get('user-agent'),
        ip: request.headers.get('x-forwarded-for') || request.ip,
        timestamp: new Date().toISOString(),
      },
    });
    
    if (type === 'click') {
      // Redirect to original URL
      const redirectUrl = request.nextUrl.searchParams.get('redirect');
      if (redirectUrl) {
        return NextResponse.redirect(redirectUrl);
      }
    }
    
    // Return transparent pixel for open tracking
    if (type === 'open') {
      const pixel = Buffer.from(
        'R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7',
        'base64'
      );
      
      return new NextResponse(pixel, {
        headers: {
          'Content-Type': 'image/gif',
          'Cache-Control': 'no-cache, no-store, must-revalidate',
          'Pragma': 'no-cache',
          'Expires': '0',
        },
      });
    }
    
    return NextResponse.json({ success: true });
  } catch (error) {
    console.error('Tracking error:', error);
    return NextResponse.json(
      { error: 'Tracking failed' },
      { status: 500 }
    );
  }
}
```

### 9.2 Delivery Status Schema

```prisma
model NotificationDelivery {
  id             String   @id @default(cuid())
  notificationId String
  channel        String   // 'email', 'push', 'sms', 'in_app'
  providerId     String?  // External provider ID (Message ID, etc.)
  status         String   // 'sent', 'delivered', 'opened', 'clicked', 'failed', 'bounced'
  error          String?
  metadata       Json?
  
  notification Notification @relation(fields: [notificationId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  
  @@index([notificationId])
  @@index([channel])
  @@index([status])
  @@index([createdAt])
  @@map("notification_deliveries")
}
```

---

## 10. RATE LIMITING & THROTTLING

### 10.1 Rate Limits Implementation

```typescript
// lib/notifications/throttle.ts
import { RateLimiterMemory, RateLimiterQueue } from 'rate-limiter-flexible';
import { prisma } from '@/lib/prisma';

// Per-user rate limiters
const userLimiters = {
  email: new RateLimiterMemory({
    points: 10, // 10 emails per hour
    duration: 60 * 60,
  }),
  
  push: new RateLimiterMemory({
    points: 20, // 20 pushes per hour
    duration: 60 * 60,
  }),
  
  sms: new RateLimiterMemory({
    points: 5, // 5 SMS per hour
    duration: 60 * 60,
  }),
  
  in_app: new RateLimiterMemory({
    points: 50, // 50 in-app notifications per hour
    duration: 60 * 60,
  }),
};

// Global rate limiters
const globalLimiters = {
  email: new RateLimiterMemory({
    points: 1000, // 1000 emails per hour globally
    duration: 60 * 60,
  }),
  
  push: new RateLimiterMemory({
    points: 5000, // 5000 pushes per hour globally
    duration: 60 * 60,
  }),
  
  sms: new RateLimiterMemory({
    points: 500, // 500 SMS per hour globally
    duration: 60 * 60,
  }),
};

// Queues for rate-limited operations
const queues = {
  email: new RateLimiterQueue(userLimiters.email, {
    maxQueueSize: 1000,
  }),
  
  push: new RateLimiterQueue(userLimiters.push, {
    maxQueueSize: 5000,
  }),
  
  sms: new RateLimiterQueue(userLimiters.sms, {
    maxQueueSize: 100,
  }),
};

export async function checkRateLimit(
  channel: 'email' | 'push' | 'sms' | 'in_app',
  userId?: string
): Promise<{ allowed: boolean; retryAfter?: number }> {
  try {
    // Check global limits first
    await globalLimiters[channel]?.consume('global');
    
    // Check user limits if userId provided
    if (userId) {
      await userLimiters[channel].consume(`user:${userId}:${channel}`);
    }
    
    return { allowed: true };
  } catch (error: any) {
    if (error.msBeforeNext) {
      return {
        allowed: false,
        retryAfter: Math.ceil(error.msBeforeNext / 1000),
      };
    }
    
    return { allowed: false };
  }
}

export async function throttleNotification(
  channel: 'email' | 'push' | 'sms',
  userId: string,
  operation: () => Promise<any>
): Promise<any> {
  try {
    // Remove from queue when ready
    const removeFromQueue = await queues[channel].removeTokens(1);
    
    // Execute the operation
    return await operation();
  } catch (error: any) {
    if (error.msBeforeNext) {
      // Wait and retry
      await new Promise(resolve => setTimeout(resolve, error.msBeforeNext));
      return await operation();
    }
    
    throw error;
  }
}

// Track notification bursts
export async function trackBurst(
  channel: string,
  userId: string,
  count: number
) {
  const key = `burst:${userId}:${channel}:${Date.now()}`;
  
  // Store in Redis with expiry
  // This would use your Redis client
  // await redisClient.setex(key, 60, count.toString());
  
  // Alert if burst exceeds threshold
  if (count > 10) {
    console.warn(`High notification burst for user ${userId}: ${count} ${channel} notifications`);
    // Send alert to admin
  }
}
```

---

## 11. NOTIFICATIONS CHECKLIST

```
EMAIL
✅ Provider configured (Resend recommended)
✅ Domain verified (SPF, DKIM, DMARC)
✅ Templates created (React Email)
✅ Unsubscribe link in all emails
✅ Queue for background sending
✅ Bounce handling implemented
✅ Open/click tracking enabled
✅ Rate limiting per user
✅ Error handling and retries

PUSH (WEB)
✅ Service worker registered
✅ VAPID keys generated and stored
✅ Permission UX designed and tested
✅ Subscription stored in database
✅ Offline support considered
✅ Notification click handling
✅ Subscription cleanup (invalid endpoints)
✅ Cross-browser compatibility tested

IN-APP
✅ Toast library configured (Sonner)
✅ Notification center UI built
✅ Database schema optimized
✅ Real-time updates (WebSocket/SSE)
✅ Mark as read functionality
✅ Pagination for notification history
✅ Badge count updates in real-time
✅ Mobile-responsive design

SMS (if needed)
✅ Provider configured (Twilio recommended)
✅ Phone verification flow implemented
✅ Templates within character limits
✅ Opt-out handling (STOP, HELP keywords)
✅ Delivery status tracking
✅ International number support
✅ Cost monitoring and alerts

PREFERENCES
✅ Preference schema designed
✅ Default preferences set appropriately
✅ UI for user control implemented
✅ Preferences respected in service layer
✅ Quiet hours functionality
✅ Do Not Disturb mode
✅ Timezone support

ARCHITECTURE
✅ Unified notification service
✅ Channel routing logic
✅ Template engine with React Email
✅ Background job queue (BullMQ/Inngest)
✅ Error handling & automatic retries
✅ Event-driven architecture
✅ Service layer for business logic

TRACKING & ANALYTICS
✅ Delivery status tracked per channel
✅ Open/click tracking for email
✅ Provider webhooks configured
✅ Analytics dashboard (optional)
✅ Performance monitoring
✅ Delivery rate reporting
✅ Bounce rate monitoring

COMPLIANCE
✅ CAN-SPAM compliance (email headers)
✅ GDPR consent for marketing emails
✅ Unsubscribe mechanism in all channels
✅ Data retention policy implemented
✅ Privacy policy includes notification data
✅ Accessibility tested (screen readers)

PERFORMANCE
✅ Rate limiting implemented
✅ Queue processing optimized
✅ Database indexes created
✅ Caching strategy for preferences
✅ Batch operations for bulk sends
✅ Monitoring for queue backlogs

TESTING
✅ Unit tests for notification service
✅ Integration tests for email sending
✅ E2E tests for notification flows
✅ Load tests for high volume
✅ Browser tests for push permissions
✅ Mobile device testing

SECURITY
✅ Input validation for all templates
✅ No sensitive data in notifications
✅ Secure storage of API keys
✅ Rate limiting to prevent abuse
✅ Audit logging for notification sends
✅ Data encryption at rest

MAINTENANCE
✅ Regular dependency updates
✅ Monitoring for failed deliveries
✅ Alerting for system issues
✅ Backup of notification data
✅ Documentation for troubleshooting
✅ Regular review of spam reports
```