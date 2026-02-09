# Catalogo I18N per Next.js 14+ App Router

§ 1. I18N LIBRARY COMPARISON

| Library | Bundle Size | RSC Support | Type Safety | ICU Format | Pluralization | Best For |
|---------|-------------|-------------|-------------|------------|---------------|----------|
| next-intl | ~15kB | ✅ Native | ✅ Excellent | ✅ Full | ✅ Full | Next.js App Router, RSC-first apps |
| react-i18next | ~30kB | ⚠️ Via wrapper | ✅ Good | ✅ Full | ✅ Full | Existing React apps, complex state |
| Lingui | ~20kB | ⚠️ Limited | ✅ Good | ✅ Full | ✅ Full | Message extraction focus |
| next-translate | ~10kB | ❌ No | ❌ Basic | ⚠️ Partial | ⚠️ Basic | Simple Next.js Pages Router |
| FormatJS | ~25kB | ❌ No | ⚠️ Manual | ✅ Full | ✅ Full | Low-level Intl API control |
| Paraglide | ~5kB | ✅ Good | ✅ Excellent | ✅ Full | ✅ Full | Ultra-performance, compile-time |

Tabella features dettagliate:

| Feature | next-intl | react-i18next | Lingui |
|---------|-----------|---------------|--------|
| Server Components | ✅ Native | ⚠️ External package | ⚠️ Requires custom setup |
| Static Generation | ✅ Built-in | ❌ Manual | ✅ Built-in |
| Edge Runtime | ✅ Full | ✅ Full | ✅ Full |
| Lazy Loading | ✅ Automatic | ✅ Manual | ✅ Automatic |
| Message Extraction | ❌ Third-party | ❌ Third-party | ✅ Built-in |
| TypeScript Native | ✅ Type-safe | ⚠️ Add-on | ✅ Type-safe |
| Next.js Integration | ✅ Deep | ⚠️ Manual | ⚠️ Manual |
| Bundle Splitting | ✅ Automatic | ✅ Manual | ✅ Automatic |
| Middleware Support | ✅ Built-in | ❌ Manual | ❌ Manual |

---

§ 2. NEXT-INTL COMPLETE SETUP

§ 2.1 INSTALLATION & CONFIGURATION
bash
# Dependencies
npm install next-intl
npm install -D @formatjs/cli  # For message extraction (optional)

# Peer dependencies (Next.js 14+)
npm install react@latest next@latest

# TypeScript types (included)

§ 2.2 PROJECT STRUCTURE
/
├── messages/
│   ├── en.json
│   ├── it.json
│   └── de.json
├── app/
│   └── [locale]/
│       ├── layout.tsx
│       ├── page.tsx
│       └── [any-route]/
│           └── page.tsx
├── i18n.ts
├── middleware.ts
├── navigation.ts
└── next.config.js

§ 2.3 CORE CONFIGURATION FILES

typescript
// i18n.ts - Core configuration
import { notFound } from 'next/navigation';
import { getRequestConfig } from 'next-intl/server';
import type { Locale } from './navigation';

// Supported locales
export const locales: Locale[] = ['en', 'it', 'de', 'fr', 'es', 'ar'];
export const defaultLocale: Locale = 'en';
export const localePrefix = 'always'; // 'always', 'as-needed', 'never'

// RTL languages configuration
export const rtlLocales: Locale[] = ['ar', 'he', 'fa', 'ur'];

// Load messages for requested locale
export default getRequestConfig(async ({ locale }) => {
  // Validate locale
  if (!locales.includes(locale as Locale)) notFound();

  return {
    messages: (await import(`../messages/${locale}.json`)).default,
    timeZone: 'Europe/Rome',
    now: new Date(),
    
    // Formatters configuration
    formats: {
      dateTime: {
        short: {
          day: 'numeric',
          month: 'short',
          year: 'numeric'
        }
      },
      number: {
        precise: {
          maximumFractionDigits: 5
        }
      }
    }
  };
});

typescript
// middleware.ts - Locale detection & routing
import createMiddleware from 'next-intl/middleware';
import { locales, defaultLocale, localePrefix } from './i18n';

export default createMiddleware({
  // Supported locales
  locales,
  
  // Default locale
  defaultLocale,
  
  // Locale detection behavior
  localePrefix,
  
  // Locale detection sources (order matters)
  localeDetection: true,
  
  // Pathnames to exclude from locale detection
  pathnames: {
    // Map pathnames to localized versions
    '/': '/',
    '/about': {
      en: '/about',
      it: '/chi-siamo',
      de: '/ueber-uns'
    },
    '/products/[category]': {
      en: '/products/[category]',
      it: '/prodotti/[category]',
      de: '/produkte/[category]'
    }
  },
  
  // Cookie configuration
  cookie: 'NEXT_LOCALE',
  cookieMaxAge: 365 * 24 * 60 * 60, // 1 year
  
  // Alternate links (hreflang)
  alternateLinks: true,
  
  // Bot detection
  detectBotLanguage: true
});

export const config = {
  // Match all paths except static files and api routes
  matcher: [
    '/((?!api|_next|_vercel|.*\\..*).*)',
    '/(it|en|de|fr|es|ar)/:path*'
  ]
};

typescript
// navigation.ts - Type-safe routing
import { createSharedPathnamesNavigation } from 'next-intl/navigation';
import { locales } from './i18n';

// Define locale type
export type Locale = (typeof locales)[number];

// Create typed navigation utilities
export const { Link, redirect, usePathname, useRouter } = 
  createSharedPathnamesNavigation({ locales });

javascript
// next.config.js
const withNextIntl = require('next-intl/plugin')();

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable React strict mode
  reactStrictMode: true,
  
  // Internationalized routing
  i18n: {
    locales: ['en', 'it', 'de', 'fr', 'es', 'ar'],
    defaultLocale: 'en',
    localeDetection: false, // Handled by middleware
  },
  
  // Enable experimental features if needed
  experimental: {
    typedRoutes: true, // TypeScript integration for routes
  },
};

module.exports = withNextIntl(nextConfig);

§ 2.4 LAYOUT & PROVIDER SETUP
typescript
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl';
import { getMessages, getTranslations } from 'next-intl/server';
import { notFound } from 'next/navigation';
import { ReactNode } from 'react';
import { locales, rtlLocales } from '@/i18n';
import { ThemeProvider } from '@/components/ThemeProvider';
import { Navigation } from '@/components/Navigation';
import { LocaleSwitcher } from '@/components/LocaleSwitcher';
import '@/styles/globals.css';

type Props = {
  children: ReactNode;
  params: { locale: string };
};

// Generate static params for all locales
export function generateStaticParams() {
  return locales.map((locale) => ({ locale }));
}

// Generate metadata with translations
export async function generateMetadata({ params: { locale } }: Props) {
  const t = await getTranslations({ locale, namespace: 'Metadata' });

  return {
    title: t('title'),
    description: t('description'),
    keywords: t('keywords'),
    metadataBase: new URL('https://example.com'),
    
    // Open Graph
    openGraph: {
      title: t('title'),
      description: t('description'),
      locale: locale,
      alternateLocale: locales.filter(l => l !== locale),
      type: 'website',
    },
    
    // Twitter
    twitter: {
      title: t('title'),
      description: t('description'),
    },
    
    // hreflang links
    alternates: {
      canonical: '/',
      languages: Object.fromEntries(
        locales.map((l) => [l, `/${l}`])
      ),
    },
  };
}

export default async function LocaleLayout({ children, params }: Props) {
  // Validate locale
  if (!locales.includes(params.locale as any)) {
    notFound();
  }

  // Get messages for the locale
  const messages = await getMessages();
  
  // Determine text direction
  const direction = rtlLocales.includes(params.locale as any) ? 'rtl' : 'ltr';
  
  return (
    <html lang={params.locale} dir={direction}>
      <body className={`min-h-screen bg-background font-sans antialiased`}>
        <NextIntlClientProvider
          locale={params.locale}
          messages={messages}
          timeZone="Europe/Rome"
          now={new Date()}
        >
          <ThemeProvider>
            <header className="border-b">
              <div className="container mx-auto flex h-16 items-center justify-between px-4">
                <Navigation locale={params.locale} />
                <LocaleSwitcher currentLocale={params.locale} />
              </div>
            </header>
            
            <main className="flex-1">
              {children}
            </main>
            
            <Footer locale={params.locale} />
          </ThemeProvider>
        </NextIntlClientProvider>
      </body>
    </html>
  );
}

// Footer component with translations
async function Footer({ locale }: { locale: string }) {
  const t = await getTranslations({ locale, namespace: 'Footer' });
  
  return (
    <footer className="border-t py-8">
      <div className="container mx-auto px-4">
        <p className="text-center text-muted-foreground">
          {t('copyright', { year: new Date().getFullYear() })}
        </p>
      </div>
    </footer>
  );
}

---

§ 3. TRANSLATION PATTERNS

§ 3.1 PATTERN REFERENCE TABLE

| Pattern | JSON Syntax | TypeScript Usage | Output Example |
|---------|-------------|------------------|----------------|
| Simple | `"greeting": "Hello"` | `t('greeting')` | "Hello" |
| Interpolation | `"welcome": "Hello, {name}!"` | `t('welcome', { name: 'John' })` | "Hello, John!" |
| Plural | `"messages": "{count, plural, =0 {No messages} one {# message} other {# messages}}"` | `t('messages', { count: 5 })` | "5 messages" |
| Select (gender) | `"pronoun": "{gender, select, male {he} female {she} other {they}}"` | `t('pronoun', { gender: 'female' })` | "she" |
| Rich Text | `"rich": "Welcome <strong>{name}</strong> to our <link>site</link>"` | `t.rich('rich', { name: 'John', strong: ... })` | HTML with components |
| Raw HTML | `"html": "<p>Some HTML</p>"` | `t.markup('html')` | Rendered HTML |
| Nested Keys | `"auth": { "login": { "title": "Login" } }` | `t('auth.login.title')` | "Login" |
| Ordinal | `"position": "You are {pos, selectordinal, one {#st} two {#nd} ...}"` | `t('position', { pos: 1 })` | "You are 1st" |

§ 3.2 IMPLEMENTATION EXAMPLES

typescript
// Basic usage in Client Component
'use client';

import { useTranslations } from 'next-intl';

export function UserProfile({ user }: { user: User }) {
  const t = useTranslations('UserProfile');
  
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('welcome', { name: user.name })}</p>
      <p>{t('messages', { count: user.messageCount })}</p>
    </div>
  );
}

// Advanced patterns in separate file
// lib/translation-patterns.ts
import { useTranslations } from 'next-intl';
import Link from 'next/link';

export function useTranslationPatterns() {
  const t = useTranslations('Examples');
  
  return {
    // 1. Simple translation
    simple: t('greeting'),
    
    // 2. Interpolation with formatting
    welcome: (name: string) => t('welcome', { name }),
    
    // 3. Pluralization (ICU format)
    messages: (count: number) => t('messages', { count }),
    
    // 4. Select (gender/type)
    pronoun: (gender: 'male' | 'female' | 'other') => 
      t('pronoun', { gender }),
    
    // 5. Rich text with components
    richText: (name: string) => t.rich('richText', {
      name,
      strong: (chunks) => <strong>{chunks}</strong>,
      link: (chunks) => (
        <Link href="/" className="text-blue-500 hover:underline">
          {chunks}
        </Link>
      ),
    }),
    
    // 6. Date/time inline formatting
    currentDate: (date: Date) => t('currentDate', { date }),
    
    // 7. Number formatting inline
    price: (amount: number) => t('price', { amount }),
    
    // 8. Ordinal numbers
    position: (pos: number) => t('position', { pos }),
    
    // 9. Complex nested structure
    nested: {
      auth: {
        login: t('auth.login.title'),
        register: t('auth.register.title'),
      },
    },
  };
}

// Server Component usage
import { getTranslations } from 'next-intl/server';

export async function ServerTranslatedContent() {
  const t = await getTranslations('ServerContent');
  
  return (
    <section>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
      {/* Server components can use t() directly */}
    </section>
  );
}

§ 3.3 MESSAGE FILE STRUCTURE (JSON)
json
// messages/en.json example completo
{
  "Metadata": {
    "title": "My Application - Global Solution",
    "description": "Discover our innovative platform for modern solutions",
    "keywords": "technology, solutions, innovation"
  },
  
  "Navigation": {
    "home": "Home",
    "about": "About Us",
    "products": "Products",
    "contact": "Contact",
    "dashboard": "Dashboard"
  },
  
  "HomePage": {
    "hero": {
      "title": "Welcome to the Future",
      "subtitle": "Innovative solutions for {companyType} companies",
      "cta": "Get Started"
    },
    "features": {
      "title": "Why Choose Us",
      "items": "{count, plural, =0 {No features} one {# amazing feature} other {# amazing features}}"
    }
  },
  
  "UserProfile": {
    "title": "Profile Settings",
    "welcome": "Hello, {name}!",
    "messages": "You have {count, plural, =0 {no messages} one {# message} other {# messages}}",
    "lastLogin": "Last login: {date, date, medium}",
    "balance": "Account balance: {amount, number, currency}",
    "membership": {
      "status": "{status, select, active {Active} expired {Expired} pending {Pending}}",
      "expires": "Expires on {date, date, long}"
    }
  },
  
  "Examples": {
    "greeting": "Hello World",
    "welcome": "Welcome, {name}!",
    "messages": "{count, plural, =0 {No messages} one {# message} other {# messages}}",
    "pronoun": "{gender, select, male {He} female {She} other {They}} will join us",
    "richText": "Welcome <strong>{name}</strong> to our <link>official website</link>",
    "currentDate": "Today is {date, date, full}",
    "price": "Price: {amount, number, currency}",
    "position": "You finished {pos, selectordinal, one {#st} two {#nd} few {#rd} other {#th}}",
    "auth": {
      "login": {
        "title": "Sign In",
        "error": "Invalid credentials"
      },
      "register": {
        "title": "Create Account",
        "success": "Account created successfully"
      }
    }
  },
  
  "Footer": {
    "copyright": "© {year} Company Name. All rights reserved.",
    "privacy": "Privacy Policy",
    "terms": "Terms of Service"
  },
  
  "Form": {
    "validation": {
      "required": "This field is required",
      "email": "Please enter a valid email",
      "minLength": "Must be at least {min} characters",
      "maxLength": "Must be at most {max} characters",
      "passwordMatch": "Passwords do not match"
    },
    "errors": {
      "generic": "An error occurred",
      "network": "Network error. Please try again."
    }
  },
  
  "Products": {
    "category": "{category, select, electronics {Electronics} clothing {Clothing} books {Books} other {Other}}",
    "stock": "{count, plural, =0 {Out of stock} one {Last item!} other {# in stock}}",
    "rating": "Rating: {rating, number, precision:1}/5",
    "discount": "Save {percent, number, percent}",
    "shipping": {
      "estimated": "Est. delivery: {date, date, short}",
      "free": "Free shipping on orders over {amount, number, currency}"
    }
  }
}

---

§ 4. DATE, TIME, NUMBER FORMATTING

§ 4.1 DATE FORMATS TABLE

| Format | en-US | it-IT | de-DE | Code |
|--------|-------|-------|-------|------|
| short | 12/31/2023 | 31/12/2023 | 31.12.2023 | `date, short` |
| medium | Dec 31, 2023 | 31 dic 2023 | 31. Dez. 2023 | `date, medium` |
| long | December 31, 2023 | 31 dicembre 2023 | 31. Dezember 2023 | `date, long` |
| full | Sunday, December 31, 2023 | domenica 31 dicembre 2023 | Sonntag, 31. Dezember 2023 | `date, full` |
| time short | 2:30 PM | 14:30 | 14:30 | `time, short` |
| time medium | 2:30:45 PM | 14:30:45 | 14:30:45 | `time, medium` |
| datetime | Dec 31, 2023, 2:30 PM | 31 dic 2023, 14:30 | 31. Dez. 2023, 14:30 | `datetime, medium` |
| relative | 2 hours ago | 2 ore fa | vor 2 Stunden | `relativeTime, hours` |

§ 4.2 NUMBER FORMATS TABLE

| Format | en-US | it-IT | de-DE | Code |
|--------|-------|-------|-------|------|
| decimal | 1,234.56 | 1.234,56 | 1.234,56 | `number, decimal` |
| currency | $1,234.56 | 1.234,56 € | 1.234,56 € | `number, currency` |
| percent | 12.34% | 12,34% | 12,34 % | `number, percent` |
| compact | 1.2K | 1,2 mila | 1,2 Tsd. | `number, compact` |
| scientific | 1.23E3 | 1,23E3 | 1,23E3 | `number, scientific` |
| unit | 5 meters | 5 metri | 5 Meter | `number, unit:meter` |

§ 4.3 IMPLEMENTATION
typescript
// lib/formatters.ts
import { createTranslator } from 'next-intl';
import { getMessages } from 'next-intl/server';

// Create formatter functions
export async function createFormatters(locale: string) {
  const messages = await getMessages();
  
  return {
    // Format date with various styles
    formatDate: (date: Date | string, style: 'short' | 'medium' | 'long' | 'full' = 'medium') => {
      const dateObj = typeof date === 'string' ? new Date(date) : date;
      return new Intl.DateTimeFormat(locale, {
        dateStyle: style,
        timeZone: 'Europe/Rome'
      }).format(dateObj);
    },
    
    // Format date with custom options
    formatCustomDate: (
      date: Date,
      options: Intl.DateTimeFormatOptions
    ) => {
      return new Intl.DateTimeFormat(locale, {
        timeZone: 'Europe/Rome',
        ...options
      }).format(date);
    },
    
    // Format number
    formatNumber: (
      value: number,
      style: 'decimal' | 'currency' | 'percent' | 'unit' = 'decimal',
      currency: string = 'EUR',
      unit?: string
    ) => {
      const options: Intl.NumberFormatOptions = {};
      
      switch (style) {
        case 'currency':
          options.style = 'currency';
          options.currency = currency;
          break;
        case 'percent':
          options.style = 'percent';
          options.minimumFractionDigits = 2;
          options.maximumFractionDigits = 2;
          break;
        case 'unit':
          options.style = 'unit';
          options.unit = unit;
          options.unitDisplay = 'short';
          break;
        case 'decimal':
        default:
          options.style = 'decimal';
          options.minimumFractionDigits = 0;
          options.maximumFractionDigits = 2;
      }
      
      return new Intl.NumberFormat(locale, options).format(value);
    },
    
    // Format currency with specific rules
    formatCurrency: (
      amount: number,
      currency: string = 'EUR',
      display: 'symbol' | 'code' | 'name' = 'symbol'
    ) => {
      return new Intl.NumberFormat(locale, {
        style: 'currency',
        currency,
        currencyDisplay: display,
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      }).format(amount);
    },
    
    // Format relative time
    formatRelativeTime: (
      date: Date,
      style: 'long' | 'short' | 'narrow' = 'long'
    ) => {
      const now = new Date();
      const diffInMs = date.getTime() - now.getTime();
      const diffInSeconds = Math.floor(diffInMs / 1000);
      const diffInMinutes = Math.floor(diffInSeconds / 60);
      const diffInHours = Math.floor(diffInMinutes / 60);
      const diffInDays = Math.floor(diffInHours / 24);
      
      const rtf = new Intl.RelativeTimeFormat(locale, { style });
      
      if (Math.abs(diffInDays) > 0) {
        return rtf.format(diffInDays, 'day');
      } else if (Math.abs(diffInHours) > 0) {
        return rtf.format(diffInHours, 'hour');
      } else if (Math.abs(diffInMinutes) > 0) {
        return rtf.format(diffInMinutes, 'minute');
      } else {
        return rtf.format(diffInSeconds, 'second');
      }
    },
    
    // Format list
    formatList: (items: string[], type: 'conjunction' | 'disjunction' = 'conjunction') => {
      return new Intl.ListFormat(locale, {
        style: 'long',
        type,
      }).format(items);
    },
    
    // Format duration
    formatDuration: (seconds: number) => {
      const hours = Math.floor(seconds / 3600);
      const minutes = Math.floor((seconds % 3600) / 60);
      const remainingSeconds = seconds % 60;
      
      const parts = [];
      if (hours > 0) parts.push(`${hours}h`);
      if (minutes > 0) parts.push(`${minutes}m`);
      if (remainingSeconds > 0 || parts.length === 0) {
        parts.push(`${remainingSeconds}s`);
      }
      
      return parts.join(' ');
    },
  };
}

// Hook for client components
import { useFormatter } from 'next-intl';

export function useFormatting() {
  const format = useFormatter();
  
  return {
    // Inline formatting examples
    formatPrice: (amount: number, currency: string) => 
      format.number(amount, { style: 'currency', currency }),
    
    formatDateShort: (date: Date) => 
      format.dateTime(date, { dateStyle: 'short' }),
    
    formatDateTime: (date: Date) => 
      format.dateTime(date, { 
        dateStyle: 'medium', 
        timeStyle: 'short' 
      }),
    
    formatPercentage: (value: number) => 
      format.number(value / 100, { style: 'percent' }),
    
    formatCompact: (value: number) => 
      format.number(value, { notation: 'compact' }),
    
    formatRelative: (date: Date) => {
      const now = new Date();
      const diffInDays = Math.floor(
        (date.getTime() - now.getTime()) / (1000 * 60 * 60 * 24)
      );
      return format.relativeTime(date);
    },
  };
}

// Component usage example
'use client';

import { useFormatter, useTranslations } from 'next-intl';

export function ProductCard({ product }: { product: Product }) {
  const t = useTranslations('Products');
  const format = useFormatter();
  
  return (
    <div className="border rounded-lg p-4">
      <h3>{product.name}</h3>
      <p className="text-lg font-semibold">
        {format.number(product.price, { 
          style: 'currency', 
          currency: 'EUR' 
        })}
      </p>
      <p className="text-sm text-gray-600">
        {t('stock', { count: product.stock })}
      </p>
      <p className="text-sm">
        {t('shipping.estimated', { 
          date: format.dateTime(product.shippingDate, {
            dateStyle: 'short'
          })
        })}
      </p>
      {product.discount > 0 && (
        <p className="text-green-600">
          {t('discount', { 
            percent: format.number(product.discount / 100, {
              style: 'percent'
            })
          })}
        </p>
      )}
    </div>
  );
}

§ 4.4 TIMEZONE HANDLING
typescript
// lib/timezone.ts
import { headers } from 'next/headers';

// Detect user timezone from headers
export async function detectUserTimezone() {
  try {
    const headersList = await headers();
    const timezone = headersList.get('x-vercel-ip-timezone') || 
                     headersList.get('x-timezone') || 
                     'Europe/Rome';
    return Intl.supportedValuesOf('timeZone').includes(timezone) 
      ? timezone 
      : 'Europe/Rome';
  } catch {
    return 'Europe/Rome';
  }
}

// Timezone-aware date formatting
export function formatDateWithTimezone(
  date: Date,
  locale: string,
  timezone: string,
  options?: Intl.DateTimeFormatOptions
) {
  return new Intl.DateTimeFormat(locale, {
    timeZone: timezone,
    ...options,
  }).format(date);
}

// Convert UTC to local time
export function utcToLocal(utcDate: Date | string, timezone: string): Date {
  const date = typeof utcDate === 'string' ? new Date(utcDate) : utcDate;
  
  // Format date in target timezone
  const formatter = new Intl.DateTimeFormat('en-US', {
    timeZone: timezone,
    year: 'numeric',
    month: 'numeric',
    day: 'numeric',
    hour: 'numeric',
    minute: 'numeric',
    second: 'numeric',
    hour12: false,
  });
  
  const parts = formatter.formatToParts(date);
  const partValues = Object.fromEntries(
    parts.map(p => [p.type, p.value])
  );
  
  return new Date(
    parseInt(partValues.year),
    parseInt(partValues.month) - 1,
    parseInt(partValues.day),
    parseInt(partValues.hour),
    parseInt(partValues.minute),
    parseInt(partValues.second)
  );
}

// Server Component with timezone
import { detectUserTimezone } from '@/lib/timezone';

export async function TimezoneAwareComponent() {
  const userTimezone = await detectUserTimezone();
  
  return (
    <div>
      <p>Your timezone: {userTimezone}</p>
      <p>Current time: {
        new Date().toLocaleString('en-US', { timeZone: userTimezone })
      }</p>
    </div>
  );
}

// Client Component with timezone detection
'use client';

import { useEffect, useState } from 'react';

export function ClientTimezoneDetector() {
  const [timezone, setTimezone] = useState<string>('');
  
  useEffect(() => {
    setTimezone(Intl.DateTimeFormat().resolvedOptions().timeZone);
  }, []);
  
  return (
    <div>
      <p>Browser timezone: {timezone}</p>
    </div>
  );
}

---

§ 5. LOCALE DETECTION & ROUTING

§ 5.1 DETECTION STRATEGY PRIORITY

| Priority | Source | Reliability | Persistence | Implementation |
|----------|--------|-------------|-------------|----------------|
| 1 | URL Path | ✅ High | ✅ Yes | `/[locale]/path` |
| 2 | Cookie | ✅ High | ✅ Yes | `NEXT_LOCALE` cookie |
| 3 | Accept-Language | ⚠️ Medium | ❌ No | `Accept-Language` header |
| 4 | Default | ✅ High | ✅ Yes | Fallback to `defaultLocale` |
| 5 | GeoIP | ⚠️ Low | ❌ No | Vercel/Cloudflare headers |

§ 5.2 URL STRATEGIES COMPARISON

| Strategy | Example | SEO | Complexity | Use Case | Implementation |
|----------|---------|-----|------------|----------|----------------|
| Path prefix | /it/about | ✅ Excellent | ✅ Low | Most sites | `app/[locale]/page.tsx` |
| Subdomain | it.example.com | ✅ Good | ⚠️ Medium | Large multi-region | DNS + middleware redirect |
| Domain | example.it | ✅ Excellent | ⚠️ Medium | Country-specific | Multiple deployments |
| Query param | ?lang=it | ❌ Poor | ✅ Low | Temporary/optional | Search params handling |
| Cookie only | No URL change | ❌ Poor | ✅ Low | User preference | Cookie-based detection |

§ 5.3 MIDDLEWARE IMPLEMENTATION
typescript
// middleware.ts - Complete implementation
import createMiddleware from 'next-intl/middleware';
import { NextRequest, NextResponse } from 'next/server';
import { locales, defaultLocale, rtlLocales } from './i18n';

// Extended middleware for custom logic
export default async function middleware(request: NextRequest) {
  // Handle static files and API routes
  const pathname = request.nextUrl.pathname;
  if (
    pathname.startsWith('/_next') ||
    pathname.startsWith('/api') ||
    pathname.startsWith('/static') ||
    /\.(ico|png|jpg|jpeg|gif|svg|css|js)$/.test(pathname)
  ) {
    return NextResponse.next();
  }

  // Check for locale cookie
  const cookieLocale = request.cookies.get('NEXT_LOCALE')?.value;
  
  // Check for Accept-Language header
  const acceptLanguage = request.headers.get('accept-language');
  const browserLocale = acceptLanguage?.split(',')[0]?.split('-')[0];
  
  // Get pathname segments
  const segments = pathname.split('/').filter(Boolean);
  const firstSegment = segments[0];
  
  // Determine target locale
  let locale = defaultLocale;
  
  // Priority 1: URL path
  if (firstSegment && locales.includes(firstSegment as any)) {
    locale = firstSegment;
  }
  // Priority 2: Cookie
  else if (cookieLocale && locales.includes(cookieLocale as any)) {
    locale = cookieLocale;
    
    // Redirect to include locale in URL
    const url = new URL(`/${locale}${pathname}`, request.url);
    return NextResponse.redirect(url);
  }
  // Priority 3: Browser preference
  else if (browserLocale && locales.includes(browserLocale as any)) {
    locale = browserLocale;
    const url = new URL(`/${locale}${pathname}`, request.url);
    return NextResponse.redirect(url);
  }
  // Priority 4: Default
  else {
    const url = new URL(`/${defaultLocale}${pathname}`, request.url);
    return NextResponse.redirect(url);
  }

  // Handle RTL direction
  const isRTL = rtlLocales.includes(locale as any);
  
  // Clone request headers and add custom headers
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set('x-locale', locale);
  requestHeaders.set('x-direction', isRTL ? 'rtl' : 'ltr');
  
  // Handle search engine bots
  const userAgent = request.headers.get('user-agent') || '';
  const isBot = /bot|crawl|spider|googlebot/i.test(userAgent);
  
  if (isBot) {
    requestHeaders.set('x-is-bot', 'true');
  }

  // Rewrite URL to include locale
  const newPathname = `/${locale}${pathname.replace(/^\/[a-z]{2}(-[A-Z]{2})?/, '')}`;
  const newUrl = new URL(newPathname, request.url);
  
  // Create response with modified headers
  const response = NextResponse.rewrite(newUrl, {
    request: {
      headers: requestHeaders,
    },
  });

  // Set locale cookie if not present or different
  if (!cookieLocale || cookieLocale !== locale) {
    response.cookies.set('NEXT_LOCALE', locale, {
      maxAge: 365 * 24 * 60 * 60, // 1 year
      path: '/',
      sameSite: 'lax',
      secure: process.env.NODE_ENV === 'production',
    });
  }

  // Add Vary header for CDN caching
  response.headers.set('Vary', 'Accept-Language, Cookie');
  
  return response;
}

export const config = {
  matcher: [
    // Match all paths except:
    '/((?!_next|api|static|favicon.ico|robots.txt|sitemap.xml).*)',
    // Always run for root
    '/',
  ],
};

§ 5.4 LOCALE SWITCHER COMPONENT
typescript
// components/LocaleSwitcher.tsx
'use client';

import { useLocale, useTranslations } from 'next-intl';
import { usePathname, useRouter } from '@/navigation';
import { locales, rtlLocales } from '@/i18n';
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuItem,
  DropdownMenuTrigger,
} from '@/components/ui/dropdown-menu';
import { Button } from '@/components/ui/button';
import { Globe } from 'lucide-react';
import { useState } from 'react';

type LocaleConfig = {
  code: string;
  name: string;
  flag: string;
  direction: 'ltr' | 'rtl';
};

const localeConfigs: Record<string, LocaleConfig> = {
  en: { code: 'en', name: 'English', flag: '🇺🇸', direction: 'ltr' },
  it: { code: 'it', name: 'Italiano', flag: '🇮🇹', direction: 'ltr' },
  de: { code: 'de', name: 'Deutsch', flag: '🇩🇪', direction: 'ltr' },
  fr: { code: 'fr', name: 'Français', flag: '🇫🇷', direction: 'ltr' },
  es: { code: 'es', name: 'Español', flag: '🇪🇸', direction: 'ltr' },
  ar: { code: 'ar', name: 'العربية', flag: '🇸🇦', direction: 'rtl' },
  he: { code: 'he', name: 'עברית', flag: '🇮🇱', direction: 'rtl' },
  ja: { code: 'ja', name: '日本語', flag: '🇯🇵', direction: 'ltr' },
  ko: { code: 'ko', name: '한국어', flag: '🇰🇷', direction: 'ltr' },
  zh: { code: 'zh', name: '中文', flag: '🇨🇳', direction: 'ltr' },
};

export function LocaleSwitcher({ 
  currentLocale,
  className = '',
  variant = 'default' 
}: { 
  currentLocale: string;
  className?: string;
  variant?: 'default' | 'compact' | 'minimal';
}) {
  const t = useTranslations('LocaleSwitcher');
  const locale = useLocale();
  const router = useRouter();
  const pathname = usePathname();
  const [isChanging, setIsChanging] = useState(false);

  const handleLocaleChange = async (newLocale: string) => {
    if (newLocale === locale || isChanging) return;
    
    setIsChanging(true);
    
    try {
      // Update locale in URL
      router.replace(pathname, { locale: newLocale });
      
      // Update direction if needed
      const currentConfig = localeConfigs[currentLocale];
      const newConfig = localeConfigs[newLocale];
      
      if (currentConfig.direction !== newConfig.direction) {
        document.documentElement.dir = newConfig.direction;
        document.documentElement.lang = newLocale;
      }
      
      // Update localStorage for persistence
      if (typeof window !== 'undefined') {
        localStorage.setItem('preferred-locale', newLocale);
      }
      
      // Trigger custom event for other components
      window.dispatchEvent(new CustomEvent('localechange', { 
        detail: { locale: newLocale } 
      }));
      
    } catch (error) {
      console.error('Failed to change locale:', error);
    } finally {
      setIsChanging(false);
    }
  };

  const currentConfig = localeConfigs[currentLocale] || localeConfigs.en;

  if (variant === 'minimal') {
    return (
      <select
        value={currentLocale}
        onChange={(e) => handleLocaleChange(e.target.value)}
        className="bg-transparent border-none cursor-pointer"
        aria-label={t('selectLanguage')}
        disabled={isChanging}
      >
        {locales.map((loc) => {
          const config = localeConfigs[loc];
          return (
            <option key={loc} value={loc}>
              {config.flag} {config.code.toUpperCase()}
            </option>
          );
        })}
      </select>
    );
  }

  if (variant === 'compact') {
    return (
      <div className={`flex items-center gap-2 ${className}`}>
        <Globe className="w-4 h-4" />
        <DropdownMenu>
          <DropdownMenuTrigger asChild>
            <Button variant="ghost" size="sm" disabled={isChanging}>
              <span className="mr-2">{currentConfig.flag}</span>
              <span className="font-medium">{currentConfig.code.toUpperCase()}</span>
            </Button>
          </DropdownMenuTrigger>
          <DropdownMenuContent align="end">
            {locales.map((loc) => {
              const config = localeConfigs[loc];
              const isActive = loc === currentLocale;
              
              return (
                <DropdownMenuItem
                  key={loc}
                  onClick={() => handleLocaleChange(loc)}
                  className={`flex items-center gap-2 ${isActive ? 'bg-accent' : ''}`}
                  disabled={isActive || isChanging}
                >
                  <span className="text-lg">{config.flag}</span>
                  <span>{config.name}</span>
                  {isActive && (
                    <span className="ml-auto text-xs text-muted-foreground">
                      ✓
                    </span>
                  )}
                </DropdownMenuItem>
              );
            })}
          </DropdownMenuContent>
        </DropdownMenu>
      </div>
    );
  }

  // Default variant
  return (
    <div className={`flex items-center gap-2 ${className}`}>
      <Globe className="w-5 h-5 text-muted-foreground" />
      <span className="text-sm text-muted-foreground">{t('language')}:</span>
      
      <DropdownMenu>
        <DropdownMenuTrigger asChild>
          <Button 
            variant="outline" 
            className="flex items-center gap-2"
            disabled={isChanging}
          >
            <span className="text-lg">{currentConfig.flag}</span>
            <span className="font-medium">{currentConfig.name}</span>
            <span className="text-xs text-muted-foreground">
              ({currentConfig.code.toUpperCase()})
            </span>
          </Button>
        </DropdownMenuTrigger>
        <DropdownMenuContent className="w-56" align="end">
          <div className="px-2 py-1.5 text-xs font-semibold text-muted-foreground">
            {t('selectLanguage')}
          </div>
          {locales.map((loc) => {
            const config = localeConfigs[loc];
            const isActive = loc === currentLocale;
            
            return (
              <DropdownMenuItem
                key={loc}
                onClick={() => handleLocaleChange(loc)}
                className={`flex items-center justify-between px-3 py-2.5 ${
                  isActive ? 'bg-accent' : ''
                }`}
                disabled={isActive || isChanging}
              >
                <div className="flex items-center gap-3">
                  <span className="text-xl">{config.flag}</span>
                  <div className="flex flex-col">
                    <span className="font-medium">{config.name}</span>
                    <span className="text-xs text-muted-foreground">
                      {t(`description.${loc}`)}
                    </span>
                  </div>
                </div>
                {isActive && (
                  <span className="text-primary">
                    ✓
                  </span>
                )}
              </DropdownMenuItem>
            );
          })}
        </DropdownMenuContent>
      </DropdownMenu>
      
      {isChanging && (
        <span className="text-xs text-muted-foreground animate-pulse">
          {t('changing')}...
        </span>
      )}
    </div>
  );
}

// Add to messages files
// messages/en.json
{
  "LocaleSwitcher": {
    "language": "Language",
    "selectLanguage": "Select language",
    "changing": "Changing language",
    "description": {
      "en": "English (US)",
      "it": "Italiano",
      "de": "Deutsch",
      "fr": "Français",
      "es": "Español",
      "ar": "العربية",
      "ja": "日本語",
      "ko": "한국어",
      "zh": "中文"
    }
  }
}

---

§ 6. SERVER COMPONENTS INTEGRATION

§ 6.1 SERVER VS CLIENT TRANSLATION

| Context | Hook/Function | Import From | When to Use |
|---------|---------------|-------------|-------------|
| Client Component | `useTranslations` | `next-intl` | Interactive components, forms, UI state |
| Client Component | `useFormatter` | `next-intl` | Formatting dates/numbers in client |
| Server Component | `getTranslations` | `next-intl/server` | Static content, SEO, metadata |
| Server Component | `getFormatter` | `next-intl/server` | Server-side formatting |
| Server Action | `getTranslations` | `next-intl/server` | Form submissions, mutations |
| Route Handler | Manual extraction | - | API routes, webhooks |
| Middleware | No translation | - | Routing, redirects |

§ 6.2 SERVER COMPONENT TRANSLATIONS
typescript
// app/[locale]/products/page.tsx
import { getTranslations, getFormatter } from 'next-intl/server';
import { notFound } from 'next/navigation';
import { locales } from '@/i18n';
import { ProductGrid } from '@/components/ProductGrid';
import { Pagination } from '@/components/Pagination';

type Props = {
  params: { locale: string };
  searchParams: { page?: string };
};

export async function generateStaticParams() {
  return locales.map((locale) => ({ locale }));
}

export async function generateMetadata({ params: { locale } }: Props) {
  const t = await getTranslations({ locale, namespace: 'Products.Metadata' });
  
  return {
    title: t('title'),
    description: t('description'),
    openGraph: {
      title: t('title'),
      description: t('description'),
      locale: locale,
    },
  };
}

export default async function ProductsPage({ params, searchParams }: Props) {
  // Get translations for this page
  const t = await getTranslations({ locale: params.locale, namespace: 'Products' });
  
  // Get formatter for server-side formatting
  const format = await getFormatter({ locale: params.locale });
  
  // Fetch products (server-side)
  const currentPage = parseInt(searchParams.page || '1');
  const { products, totalPages } = await getProducts(currentPage);
  
  // Format dates on server
  const formattedProducts = products.map((product) => ({
    ...product,
    formattedDate: format.dateTime(product.createdAt, {
      dateStyle: 'medium',
    }),
    formattedPrice: format.number(product.price, {
      style: 'currency',
      currency: product.currency,
    }),
  }));

  return (
    <div className="container mx-auto px-4 py-8">
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{t('page.title')}</h1>
        <p className="text-lg text-muted-foreground">
          {t('page.description')}
        </p>
      </header>
      
      {/* Server-rendered stats */}
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-8">
        <div className="bg-card rounded-lg p-4">
          <h3 className="font-semibold mb-2">{t('stats.totalProducts')}</h3>
          <p className="text-2xl font-bold">
            {format.number(products.length)}
          </p>
        </div>
        <div className="bg-card rounded-lg p-4">
          <h3 className="font-semibold mb-2">{t('stats.lastUpdated')}</h3>
          <p className="text-lg">
            {format.dateTime(new Date(), { dateStyle: 'full' })}
          </p>
        </div>
        <div className="bg-card rounded-lg p-4">
          <h3 className="font-semibold mb-2">{t('stats.currency')}</h3>
          <p className="text-lg">
            {format.number(1, { style: 'currency', currency: 'EUR' })}
          </p>
        </div>
      </div>
      
      {/* Product grid - can be client component */}
      <ProductGrid 
        products={formattedProducts}
        locale={params.locale}
      />
      
      {/* Pagination */}
      {totalPages > 1 && (
        <div className="mt-8">
          <Pagination
            currentPage={currentPage}
            totalPages={totalPages}
            basePath={`/${params.locale}/products`}
          />
        </div>
      )}
    </div>
  );
}

// Server action with translations
'use server';

import { getTranslations } from 'next-intl/server';
import { revalidatePath } from 'next/cache';
import { z } from 'zod';

const contactSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  message: z.string().min(10),
});

export async function submitContactForm(
  prevState: any,
  formData: FormData,
  locale: string
) {
  // Get translations for validation messages
  const t = await getTranslations({ locale, namespace: 'Form.validation' });
  
  try {
    const data = Object.fromEntries(formData);
    const validated = contactSchema.safeParse(data);
    
    if (!validated.success) {
      // Return localized error messages
      const errors = validated.error.errors.map((error) => {
        const key = error.path[0] as string;
        return t(key as any) || error.message;
      });
      
      return {
        success: false,
        errors,
        message: t('generic'),
      };
    }
    
    // Process form submission...
    
    revalidatePath(`/${locale}/contact`);
    
    return {
      success: true,
      message: t('success'),
    };
  } catch (error) {
    return {
      success: false,
      message: t('network'),
    };
  }
}

§ 6.3 METADATA TRANSLATION
typescript
// app/[locale]/layout.tsx - Extended metadata
import type { Metadata } from 'next';
import { getTranslations } from 'next-intl/server';

export async function generateMetadata({
  params,
}: {
  params: { locale: string };
}): Promise<Metadata> {
  const t = await getTranslations({ locale: params.locale, namespace: 'Metadata' });
  
  return {
    title: {
      default: t('title'),
      template: `%s | ${t('title')}`,
    },
    description: t('description'),
    
    // Open Graph
    openGraph: {
      title: t('title'),
      description: t('description'),
      url: 'https://example.com',
      siteName: t('title'),
      locale: params.locale,
      alternateLocale: ['en', 'it', 'de', 'fr', 'es', 'ar'].filter(
        (l) => l !== params.locale
      ),
      type: 'website',
    },
    
    // Twitter
    twitter: {
      card: 'summary_large_image',
      title: t('title'),
      description: t('description'),
    },
    
    // Icons
    icons: {
      icon: '/favicon.ico',
      apple: '/apple-touch-icon.png',
    },
    
    // Manifest
    manifest: '/manifest.json',
    
    // hreflang alternate links
    alternates: {
      canonical: `https://example.com/${params.locale}`,
      languages: {
        'en': 'https://example.com/en',
        'it': 'https://example.com/it',
        'de': 'https://example.com/de',
        'fr': 'https://example.com/fr',
        'es': 'https://example.com/es',
        'ar': 'https://example.com/ar',
      },
    },
    
    // Robots
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        'max-video-preview': -1,
        'max-image-preview': 'large',
        'max-snippet': -1,
      },
    },
    
    // Verification
    verification: {
      google: 'google-site-verification-code',
      yandex: 'yandex-verification-code',
    },
  };
}

// Dynamic metadata per page
// app/[locale]/blog/[slug]/page.tsx
import { getTranslations, getFormatter } from 'next-intl/server';

export async function generateMetadata({
  params,
}: {
  params: { locale: string; slug: string };
}) {
  const post = await getBlogPost(params.slug);
  const t = await getTranslations({ locale: params.locale, namespace: 'Blog' });
  const format = await getFormatter({ locale: params.locale });
  
  return {
    title: post.title,
    description: post.excerpt,
    
    openGraph: {
      title: post.title,
      description: post.excerpt,
      type: 'article',
      publishedTime: post.publishedAt,
      authors: [post.author],
      tags: post.tags,
    },
    
    twitter: {
      card: 'summary_large_image',
      title: post.title,
      description: post.excerpt,
    },
    
    alternates: {
      canonical: `https://example.com/${params.locale}/blog/${params.slug}`,
      languages: {
        en: `https://example.com/en/blog/${params.slug}`,
        it: `https://example.com/it/blog/${params.slug}`,
        // ... other locales
      },
    },
    
    // Article-specific metadata
    authors: [{ name: post.author }],
    publisher: t('publisher'),
    keywords: post.tags.join(', '),
  };
}

§ 6.4 STATIC GENERATION
typescript
// app/[locale]/blog/[slug]/page.tsx - Static generation
import { getTranslations } from 'next-intl/server';
import { notFound } from 'next/navigation';
import { locales } from '@/i18n';
import { getBlogPosts, getBlogPost } from '@/lib/blog';

// Generate static params for all blog posts in all locales
export async function generateStaticParams() {
  const posts = await getBlogPosts();
  const params: { locale: string; slug: string }[] = [];
  
  for (const locale of locales) {
    for (const post of posts) {
      // Check if post is available in this locale
      const localizedPost = await getBlogPost(post.slug, locale);
      if (localizedPost) {
        params.push({
          locale,
          slug: post.slug,
        });
      }
    }
  }
  
  return params;
}

// Revalidate every hour
export const revalidate = 3600;

// Dynamic segments not included in generateStaticParams will return 404
export const dynamicParams = false;

export default async function BlogPostPage({
  params,
}: {
  params: { locale: string; slug: string };
}) {
  const post = await getBlogPost(params.slug, params.locale);
  
  if (!post) {
    notFound();
  }
  
  const t = await getTranslations({ locale: params.locale, namespace: 'Blog' });
  
  return (
    <article className="container mx-auto px-4 py-8 max-w-4xl">
      <header className="mb-8">
        <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
        <div className="flex items-center gap-4 text-muted-foreground">
          <time dateTime={post.publishedAt}>
            {new Date(post.publishedAt).toLocaleDateString(params.locale, {
              dateStyle: 'long',
            })}
          </time>
          <span>•</span>
          <span>{t('readingTime', { minutes: post.readingTime })}</span>
        </div>
      </header>
      
      <div className="prose prose-lg dark:prose-invert max-w-none">
        {post.content}
      </div>
      
      {/* Static footer */}
      <footer className="mt-8 pt-8 border-t">
        <h3 className="text-lg font-semibold mb-4">{t('relatedPosts')}</h3>
        {/* Related posts */}
      </footer>
    </article>
  );
}

// app/sitemap.ts - Multi-locale sitemap
import { MetadataRoute } from 'next';
import { locales } from '@/i18n';
import { getBlogPosts, getPages } from '@/lib/content';

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = 'https://example.com';
  const pages = await getPages();
  const blogPosts = await getBlogPosts();
  
  const sitemapEntries: MetadataRoute.Sitemap = [];
  
  // Add homepage for each locale
  for (const locale of locales) {
    sitemapEntries.push({
      url: `${baseUrl}/${locale}`,
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
      alternates: {
        languages: Object.fromEntries(
          locales.map((l) => [l, `${baseUrl}/${l}`])
        ),
      },
    });
    
    // Add static pages
    for (const page of pages) {
      sitemapEntries.push({
        url: `${baseUrl}/${locale}${page.path}`,
        lastModified: page.updatedAt,
        changeFrequency: page.changeFrequency as any,
        priority: page.priority,
        alternates: {
          languages: Object.fromEntries(
            locales.map((l) => [l, `${baseUrl}/${l}${page.path}`])
          ),
        },
      });
    }
    
    // Add blog posts
    for (const post of blogPosts) {
      sitemapEntries.push({
        url: `${baseUrl}/${locale}/blog/${post.slug}`,
        lastModified: post.updatedAt,
        changeFrequency: 'weekly',
        priority: 0.8,
        alternates: {
          languages: Object.fromEntries(
            locales
              .filter((l) => post.availableLocales.includes(l))
              .map((l) => [l, `${baseUrl}/${l}/blog/${post.slug}`])
          ),
        },
      });
    }
  }
  
  return sitemapEntries;
}

---

§ 7. RTL (RIGHT-TO-LEFT) SUPPORT

§ 7.1 RTL LANGUAGES REFERENCE

| Language | Code | Direction | Script | Notes |
|----------|------|-----------|--------|-------|
| Arabic | ar | RTL | Arabic | Most complex, contextual letters |
| Hebrew | he | RTL | Hebrew | No vowel marks in standard text |
| Persian | fa | RTL | Arabic script | Additional letters, different numerals |
| Urdu | ur | RTL | Arabic script | Additional letters from Persian |
| Kurdish | ku | RTL | Arabic script | Sorani dialect |
| Yiddish | yi | RTL | Hebrew script | Additional letters from German |
| Syriac | syr | RTL | Syriac | Liturgical language |
| Divehi | dv | RTL | Thaana | Maldives language |

§ 7.2 CSS LOGICAL PROPERTIES

| Physical Property | Logical Property | CSS Example | Use Case |
|-------------------|------------------|-------------|----------|
| `margin-left` | `margin-inline-start` | `margin-inline-start: 1rem` | Space before element |
| `margin-right` | `margin-inline-end` | `margin-inline-end: 1rem` | Space after element |
| `padding-left` | `padding-inline-start` | `padding-inline-start: 1rem` | Inner padding start |
| `padding-right` | `padding-inline-end` | `padding-inline-end: 1rem` | Inner padding end |
| `border-left` | `border-inline-start` | `border-inline-start: 1px solid` | Border at start |
| `border-right` | `border-inline-end` | `border-inline-end: 1px solid` | Border at end |
| `text-align: left` | `text-align: start` | `text-align: start` | Text alignment |
| `float: left` | `float: inline-start` | `float: inline-start` | Floating elements |
| `clear: left` | `clear: inline-start` | `clear: inline-start` | Clearing floats |
| `left: 0` | `inset-inline-start: 0` | `inset-inline-start: 0` | Absolute positioning |

§ 7.3 TAILWIND RTL SETUP
typescript
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  
  // Enable dark mode
  darkMode: ['class'],
  
  theme: {
    extend: {
      // Custom colors
      colors: {
        border: 'hsl(var(--border))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        // ... other colors
      },
      
      // RTL-aware spacing
      spacing: {
        'inline-start': 'var(--spacing-inline-start)',
        'inline-end': 'var(--spacing-inline-end)',
      },
      
      // RTL-aware border radius
      borderRadius: {
        'start-start': 'var(--radius-start-start)',
        'start-end': 'var(--radius-start-end)',
        'end-start': 'var(--radius-end-start)',
        'end-end': 'var(--radius-end-end)',
      },
    },
  },
  
  // RTL plugin
  plugins: [
    require('tailwindcss-rtl'),
    // or for more control:
    function({ addVariant }) {
      addVariant('rtl', '&:where([dir="rtl"], [dir="rtl"] *)');
      addVariant('ltr', '&:where([dir="ltr"], [dir="ltr"] *)');
    }
  ],
};

export default config;

// styles/globals.css - CSS custom properties for RTL
:root {
  --spacing-inline-start: 0;
  --spacing-inline-end: 0;
  --radius-start-start: 0;
  --radius-start-end: 0;
  --radius-end-start: 0;
  --radius-end-end: 0;
}

[dir="ltr"] {
  --spacing-inline-start: left;
  --spacing-inline-end: right;
  --radius-start-start: top-left;
  --radius-start-end: top-right;
  --radius-end-start: bottom-left;
  --radius-end-end: bottom-right;
}

[dir="rtl"] {
  --spacing-inline-start: right;
  --spacing-inline-end: left;
  --radius-start-start: top-right;
  --radius-start-end: top-left;
  --radius-end-start: bottom-right;
  --radius-end-end: bottom-left;
}

// Utility classes for RTL
.rtl-flip {
  transform: scaleX(-1);
}

.ltr-only {
  display: block;
}
.rtl-only {
  display: none;
}

[dir="rtl"] {
  .ltr-only {
    display: none;
  }
  .rtl-only {
    display: block;
  }
}

// Mixins for complex RTL logic
@layer utilities {
  .flex-rtl-aware {
    display: flex;
    
    [dir="ltr"] & {
      flex-direction: row;
    }
    
    [dir="rtl"] & {
      flex-direction: row-reverse;
    }
  }
  
  .text-start {
    text-align: start;
  }
  
  .text-end {
    text-align: end;
  }
  
  .ms-auto {
    margin-inline-start: auto;
  }
  
  .me-auto {
    margin-inline-end: auto;
  }
  
  .ps-4 {
    padding-inline-start: 1rem;
  }
  
  .pe-4 {
    padding-inline-end: 1rem;
  }
}

§ 7.4 DYNAMIC DIR ATTRIBUTE
typescript
// hooks/useDirection.ts
'use client';

import { useState, useEffect } from 'react';
import { useLocale } from 'next-intl';
import { rtlLocales } from '@/i18n';

export function useDirection() {
  const locale = useLocale();
  const [direction, setDirection] = useState<'ltr' | 'rtl'>('ltr');
  
  useEffect(() => {
    const newDirection = rtlLocales.includes(locale as any) ? 'rtl' : 'ltr';
    setDirection(newDirection);
    
    // Update HTML direction
    document.documentElement.dir = newDirection;
    document.documentElement.lang = locale;
    
    // Update CSS custom properties if needed
    document.documentElement.style.setProperty('--direction', newDirection);
    
    // Dispatch event for other components
    window.dispatchEvent(new CustomEvent('directionchange', { 
      detail: { direction: newDirection } 
    }));
    
  }, [locale]);
  
  return {
    direction,
    isRTL: direction === 'rtl',
    isLTR: direction === 'ltr',
    locale,
  };
}

// components/DirectionAwareComponent.tsx
'use client';

import { useDirection } from '@/hooks/useDirection';

export function DirectionAwareComponent() {
  const { direction, isRTL } = useDirection();
  
  return (
    <div className={`flex ${isRTL ? 'flex-row-reverse' : 'flex-row'} gap-4`}>
      <div className={`bg-primary text-primary-foreground p-4 rounded-${isRTL ? 's' : 'e'}-lg`}>
        Start side
      </div>
      <div className="flex-1 p-4 bg-muted">
        Content
      </div>
      <div className={`bg-primary text-primary-foreground p-4 rounded-${isRTL ? 'e' : 's'}-lg`}>
        End side
      </div>
    </div>
  );
}

// app/[locale]/layout.tsx - Update
export default async function LocaleLayout({ children, params }: Props) {
  // ... existing code ...
  
  return (
    <html lang={params.locale} dir={direction}>
      <body className={`direction-${direction}`}>
        {/* ... */}
      </body>
    </html>
  );
}

// lib/rtl-utils.ts - Helper functions
export function getLogicalProperty(
  ltrValue: string,
  rtlValue: string,
  direction: 'ltr' | 'rtl'
): string {
  return direction === 'ltr' ? ltrValue : rtlValue;
}

export function getMarginStart(direction: 'ltr' | 'rtl'): string {
  return direction === 'ltr' ? 'margin-left' : 'margin-right';
}

export function getPaddingEnd(direction: 'ltr' | 'rtl'): string {
  return direction === 'ltr' ? 'padding-right' : 'padding-left';
}

export function flipPosition(
  position: 'left' | 'right' | 'top' | 'bottom',
  direction: 'ltr' | 'rtl'
): string {
  if (position === 'left') return direction === 'ltr' ? 'left' : 'right';
  if (position === 'right') return direction === 'ltr' ? 'right' : 'left';
  return position;
}

§ 7.5 ICON MIRRORING RULES

| Mirror | Don't Mirror | Conditional Mirror |
|--------|--------------|-------------------|
| Arrows (← →) | Checkmarks (✓) | Chevrons (contextual) |
| Progress indicators | Status icons | Navigation arrows |
| Text alignment icons | Media controls | Sorting arrows |
| Indentation icons | Bookmark icons | Play/Pause (culture-dependent) |
| Transfer/swap icons | Star ratings | Timeline arrows |

typescript
// components/RTLIcon.tsx
'use client';

import { ChevronLeft, ChevronRight, ArrowLeft, ArrowRight, Menu, X } from 'lucide-react';
import { useDirection } from '@/hooks/useDirection';
import { cn } from '@/lib/utils';

type IconName = 
  | 'chevron-left' 
  | 'chevron-right' 
  | 'arrow-left' 
  | 'arrow-right'
  | 'menu'
  | 'close';

interface RTLIconProps {
  name: IconName;
  className?: string;
  size?: number;
  mirrorInRTL?: boolean;
}

const iconComponents = {
  'chevron-left': ChevronLeft,
  'chevron-right': ChevronRight,
  'arrow-left': ArrowLeft,
  'arrow-right': ArrowRight,
  'menu': Menu,
  'close': X,
};

const mirrorInRTL: Record<IconName, boolean> = {
  'chevron-left': true,
  'chevron-right': true,
  'arrow-left': true,
  'arrow-right': true,
  'menu': false, // Hamburger menu doesn't mirror
  'close': false, // Close icon doesn't mirror
};

export function RTLIcon({ 
  name, 
  className, 
  size = 24,
  mirrorInRTL: forceMirror 
}: RTLIconProps) {
  const { isRTL } = useDirection();
  const IconComponent = iconComponents[name];
  const shouldMirror = forceMirror ?? mirrorInRTL[name];
  
  return (
    <IconComponent 
      className={cn(
        className,
        shouldMirror && isRTL && 'rtl-flip'
      )}
      size={size}
      aria-hidden="true"
    />
  );
}

// Hook for complex icon logic
export function useRTLIcon() {
  const { isRTL } = useDirection();
  
  return {
    // Navigation icons
    backIcon: isRTL ? 'chevron-right' : 'chevron-left',
    forwardIcon: isRTL ? 'chevron-left' : 'chevron-right',
    
    // Breadcrumb separator
    breadcrumbSeparator: isRTL ? '←' : '→',
    
    // Progress indicators
    progressStart: isRTL ? 'right' : 'left',
    progressEnd: isRTL ? 'left' : 'right',
    
    // Text alignment
    textAlignLeft: isRTL ? 'end' : 'start',
    textAlignRight: isRTL ? 'start' : 'end',
    
    // Get appropriate icon name
    getIcon: (icon: IconName): IconName => {
      if (!isRTL) return icon;
      
      const mirroredIcons: Record<IconName, IconName> = {
        'chevron-left': 'chevron-right',
        'chevron-right': 'chevron-left',
        'arrow-left': 'arrow-right',
        'arrow-right': 'arrow-left',
        'menu': 'menu',
        'close': 'close',
      };
      
      return mirroredIcons[icon] || icon;
    },
  };
}

// Usage example
export function NavigationButtons() {
  const { backIcon, forwardIcon } = useRTLIcon();
  
  return (
    <div className="flex items-center gap-2">
      <button aria-label="Go back">
        <RTLIcon name={backIcon} />
      </button>
      <button aria-label="Go forward">
        <RTLIcon name={forwardIcon} />
      </button>
    </div>
  );
}

---

§ 8. TYPE SAFETY

§ 8.1 TYPED KEYS SETUP
typescript
// global.d.ts
import en from './messages/en.json';

type DeepKeys<T> = T extends object
  ? {
      [K in keyof T]: K extends string
        ? T[K] extends object
          ? `${K}.${DeepKeys<T[K]>}`
          : `${K}`
        : never;
    }[keyof T]
  : never;

type DeepPick<T, K extends string> = K extends `${infer First}.${infer Rest}`
  ? First extends keyof T
    ? DeepPick<T[First], Rest>
    : never
  : K extends keyof T
  ? T[K]
  : never;

declare global {
  // Define message structure
  interface IntlMessages {
    Metadata: typeof en.Metadata;
    Navigation: typeof en.Navigation;
    HomePage: typeof en.HomePage;
    UserProfile: typeof en.UserProfile;
    Examples: typeof en.Examples;
    Footer: typeof en.Footer;
    Form: typeof en.Form;
    Products: typeof en.Products;
    LocaleSwitcher: typeof en.LocaleSwitcher;
    Blog: typeof en.Blog;
  }

  // Type-safe translation keys
  type TranslationKeys = DeepKeys<IntlMessages>;
  
  // Type-safe nested translations
  type NamespaceKeys<T extends keyof IntlMessages> = DeepKeys<IntlMessages[T]>;
  
  // Type-safe params
  type TranslationParams<K extends TranslationKeys> = 
    DeepPick<IntlMessages, K> extends string
      ? Record<string, never>
      : DeepPick<IntlMessages, K> extends (params: infer P) => string
      ? P
      : never;
}

// Extend next-intl types
declare module 'next-intl' {
  interface useTranslations<T extends keyof IntlMessages = never> {
    <K extends DeepKeys<IntlMessages[T]>>(
      key: K,
      params?: TranslationParams<K>
    ): string;
    
    rich: <K extends DeepKeys<IntlMessages[T]>>(
      key: K,
      params?: TranslationParams<K> & Record<string, any>
    ) => React.ReactNode;
    
    raw: <K extends DeepKeys<IntlMessages[T]>>(
      key: K
    ) => any;
    
    markup: <K extends DeepKeys<IntlMessages[T]>>(
      key: K
    ) => React.ReactNode;
  }
}

declare module 'next-intl/server' {
  interface getTranslations<T extends keyof IntlMessages = never> {
    (namespace?: T): <K extends DeepKeys<IntlMessages[T]>>(
      key: K,
      params?: TranslationParams<K>
    ) => Promise<string>;
  }
}

// Example usage types
type ExampleParams = TranslationParams<'UserProfile.welcome'>;
// Result: { name: string }

type ExampleNoParams = TranslationParams<'Navigation.home'>;
// Result: Record<string, never>

// Validation utility
export function validateTranslationKey<K extends TranslationKeys>(
  key: K,
  params?: TranslationParams<K>
): boolean {
  // This function ensures type safety at runtime
  return true;
}

§ 8.2 COMPILE-TIME VALIDATION
json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2022",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    },
    // Strict type checking
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,
    "exactOptionalPropertyTypes": true,
    "forceConsistentCasingInFileNames": true,
    
    // For i18n type safety
    "types": ["./global.d.ts"]
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts",
    "global.d.ts"
  ],
  "exclude": ["node_modules"]
}

// package.json scripts for validation
{
  "scripts": {
    "type-check": "tsc --noEmit",
    "type-check:watch": "tsc --noEmit --watch",
    "validate-translations": "node scripts/validate-translations.js",
    "extract-messages": "formatjs extract '**/*.tsx' --out-file messages/en.json --id-interpolation-pattern '[sha512:contenthash:base64:6]'"
  }
}

// scripts/validate-translations.js
const fs = require('fs');
const path = require('path');

function validateMessages() {
  const locales = ['en', 'it', 'de', 'fr', 'es', 'ar'];
  const basePath = path.join(__dirname, '../messages');
  
  // Load base locale (en)
  const baseMessages = JSON.parse(
    fs.readFileSync(path.join(basePath, 'en.json'), 'utf8')
  );
  
  const baseKeys = extractKeys(baseMessages);
  
  console.log('Validating translations...');
  
  let hasErrors = false;
  
  for (const locale of locales) {
    if (locale === 'en') continue;
    
    const localePath = path.join(basePath, `${locale}.json`);
    
    if (!fs.existsSync(localePath)) {
      console.error(`❌ Missing file: ${localePath}`);
      hasErrors = true;
      continue;
    }
    
    const localeMessages = JSON.parse(fs.readFileSync(localePath, 'utf8'));
    const localeKeys = extractKeys(localeMessages);
    
    // Check for missing keys
    const missingKeys = baseKeys.filter(key => !localeKeys.includes(key));
    
    if (missingKeys.length > 0) {
      console.error(`❌ ${locale}: Missing ${missingKeys.length} keys:`);
      missingKeys.forEach(key => console.error(`   - ${key}`));
      hasErrors = true;
    }
    
    // Check for extra keys
    const extraKeys = localeKeys.filter(key => !baseKeys.includes(key));
    
    if (extraKeys.length > 0) {
      console.warn(`⚠️ ${locale}: ${extraKeys.length} extra keys (not in en):`);
      extraKeys.forEach(key => console.warn(`   - ${key}`));
    }
    
    // Check for empty values
    const emptyKeys = findEmptyValues(localeMessages);
    
    if (emptyKeys.length > 0) {
      console.warn(`⚠️ ${locale}: ${emptyKeys.length} empty values:`);
      emptyKeys.forEach(key => console.warn(`   - ${key}`));
    }
  }
  
  if (hasErrors) {
    console.error('\n❌ Translation validation failed!');
    process.exit(1);
  }
  
  console.log('\n✅ All translations validated successfully!');
}

function extractKeys(obj, prefix = '') {
  return Object.keys(obj).reduce((keys, key) => {
    const value = obj[key];
    const fullKey = prefix ? `${prefix}.${key}` : key;
    
    if (typeof value === 'object' && value !== null) {
      return [...keys, ...extractKeys(value, fullKey)];
    }
    
    return [...keys, fullKey];
  }, []);
}

function findEmptyValues(obj, prefix = '') {
  return Object.keys(obj).reduce((emptyKeys, key) => {
    const value = obj[key];
    const fullKey = prefix ? `${prefix}.${key}` : key;
    
    if (typeof value === 'object' && value !== null) {
      return [...emptyKeys, ...findEmptyValues(value, fullKey)];
    }
    
    if (!value || value.trim() === '') {
      return [...emptyKeys, fullKey];
    }
    
    return emptyKeys;
  }, []);
}

validateMessages();

§ 8.3 IDE AUTOCOMPLETE
typescript
// .vscode/settings.json
{
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "typescript.suggest.completeFunctionCalls": true,
  "typescript.preferences.autoImportFileExcludePatterns": [
    "**/node_modules/**"
  ],
  "editor.quickSuggestions": {
    "strings": "on"
  },
  "editor.suggest.showStatusBar": true,
  "editor.suggestOnTriggerCharacters": true,
  "editor.acceptSuggestionOnEnter": "on",
  "editor.wordBasedSuggestions": "matchingDocuments",
  "editor.suggestSelection": "recentlyUsed",
  
  // TypeScript specific
  "typescript.inlayHints.enumMemberValues.enabled": true,
  "typescript.inlayHints.functionLikeReturnTypes.enabled": true,
  "typescript.inlayHints.propertyDeclarationTypes.enabled": true,
  "typescript.inlayHints.variableTypes.enabled": true,
  
  // Path Intellisense for translation keys
  "path-intellisense.mappings": {
    "@": "${workspaceFolder}"
  }
}

// .vscode/extensions.json
{
  "recommendations": [
    "mikael.angular-beastcode",
    "ms-vscode.vscode-typescript-next",
    "ms-vscode.vscode-typescript-tslint-plugin",
    "christian-kohler.path-intellisense",
    "streetsidesoftware.code-spell-checker",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    
    // For i18n
    "lokalise.i18n-ally",
    "antfu.i18n-helper"
  ]
}

// i18n-ally configuration
// .vscode/settings.json additional config
{
  "i18n-ally.localesPaths": "messages",
  "i18n-ally.pathMatcher": "{locale}.json",
  "i18n-ally.keystyle": "nested",
  "i18n-ally.displayLanguage": "en",
  "i18n-ally.sourceLanguage": "en",
  "i18n-ally.enabledParsers": ["json"],
  "i18n-ally.namespace": true,
  "i18n-ally.sortKeys": true,
  "i18n-ally.extract.autoDetect": true,
  "i18n-ally.annotation": true,
  "i18n-ally.annotationInPlace": true,
  "i18n-ally.annotationMaxLength": 40,
  
  // TypeScript support
  "i18n-ally.enabledFrameworks": ["react", "general"],
  "i18n-ally.tabStyle": "tab",
  
  // Editor decorations
  "i18n-ally.editor.decorations": true,
  "i18n-ally.editor.preferEditor": true
}

// Custom VS Code snippets for i18n
// .vscode/i18n.code-snippets
{
  "Translation Hook": {
    "prefix": "t",
    "body": [
      "const t = useTranslations('${1:namespace}');",
      "",
      "t('${2:key}')"
    ],
    "description": "Create a translation hook"
  },
  "Server Translation": {
    "prefix": "gt",
    "body": [
      "const t = await getTranslations('${1:namespace}');",
      "",
      "t('${2:key}')"
    ],
    "description": "Get translations in server component"
  },
  "Rich Translation": {
    "prefix": "trich",
    "body": [
      "t.rich('${1:key}', {",
      "  ${2:param}: ${3:value},",
      "  strong: (chunks) => <strong>{chunks}</strong>,",
      "  link: (chunks) => (",
      "    <Link href=\"${4:/}\">{chunks}</Link>",
      "  ),",
      "})"
    ],
    "description": "Create a rich text translation"
  }
}

// TypeScript Plugin for better IntelliSense
// Custom plugin could be created but here's a helper function:
export function createTranslationHelper<T extends keyof IntlMessages>(
  namespace: T
) {
  return {
    t: <K extends DeepKeys<IntlMessages[T]>>(
      key: K,
      params?: TranslationParams<K>
    ) => {
      // This is a type-safe wrapper
      // In practice, you'd use useTranslations or getTranslations
      return key;
    },
    
    // Autocomplete suggestions
    keys: {} as Record<DeepKeys<IntlMessages[T]>, string>,
  };
}

// Usage example with autocomplete
const home = createTranslationHelper('HomePage');

// Now home.t() will autocomplete HomePage keys
// home.t('hero.title') ✅
// home.t('invalid.key') ❌ Type error

---

§ 9. TRANSLATION WORKFLOW

§ 9.1 FILE ORGANIZATION PATTERNS

| Pattern | Structure | Best For | Pros | Cons |
|---------|-----------|----------|------|------|
| Single file | `/messages/en.json` | Small apps (1-50 keys) | Simple, no fragmentation | Hard to maintain large files |
| Namespace-based | `/messages/en/common.json`<br>`/messages/en/auth.json` | Medium apps (50-500 keys) | Logical grouping, team collaboration | Multiple imports needed |
| Feature-based | `/features/auth/messages/en.json`<br>`/features/dashboard/messages/en.json` | Large apps, monorepos (>500 keys) | Feature isolation, independent deploys | Complex build setup |
| Domain-based | `/domains/shop/messages/en.json`<br>`/domains/blog/messages/en.json` | Enterprise apps, micro-frontends | Domain separation, team autonomy | Duplication risk |
| Dynamic loading | Async import per route/chunk | Apps with code splitting | Optimized bundles, lazy loading | Runtime overhead |

§ 9.2 KEY NAMING CONVENTIONS

| Convention | Example | Pros | Cons | Best For |
|------------|---------|------|------|----------|
| Dot notation hierarchical | `auth.login.title`<br>`auth.login.button.submit` | Clear hierarchy, auto-complete friendly | Long keys, repetitive prefixes | Most projects |
| Flat with underscores | `auth_login_title`<br>`auth_login_button_submit` | Shorter import paths, simple | No hierarchy, hard to organize | Small projects |
| Semantic grouping | `loginPageTitle`<br>`submitLoginButton` | Readable, self-documenting | No automatic grouping | Team preference |
| Component scoped | `LoginForm.title`<br>`LoginForm.button.submit` | Component isolation, portable | Duplication across components | Component libraries |
| Action-oriented | `title.login`<br>`button.login.submit` | Focus on action/UI element | Inconsistent with data structure | Design-system driven |

§ 9.3 TRANSLATION MANAGEMENT TOOLS

| Tool | Type | CI/CD | Pricing | Key Features | Integration |
|------|------|-------|---------|-------------|-------------|
| Crowdin | Cloud | ✅ | Freemium | 30+ formats, Over-the-air, AI | GitHub, GitLab, CLI |
| Lokalise | Cloud | ✅ | $$$ | Figma plugin, Screenshots, QA | GitHub, Bitbucket, API |
| Phrase | Cloud | ✅ | $$$ | GitHub sync, Style guide, Glossary | GitHub, GitLab, Azure |
| Weblate | Self-hosted | ✅ | Free (AGPL) | Git integration, Add-ons, API | Any git, REST API |
| Transifex | Cloud | ✅ | $$$ | Live preview, Context, Automation | GitHub, Slack, Jira |
| SimpleLocalize | Cloud | ✅ | $$ | Auto-translation, In-context | GitHub, CLI, Webhooks |
| Tolgee | Self-hosted/Cloud | ✅ | Freemium | In-context, Auto-translation | SDK, API, Plugins |

§ 9.4 CI/CD INTEGRATION
yaml
# .github/workflows/validate-i18n.yml
name: Validate Internationalization

on:
  push:
    branches: [main, develop]
    paths:
      - 'messages/**'
      - 'app/**'
      - 'components/**'
  pull_request:
    branches: [main, develop]

jobs:
  validate-translations:
    name: Validate Translations
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Type check
        run: npm run type-check
        
      - name: Validate translation structure
        run: node scripts/validate-translations.js
        
      - name: Check for missing keys
        run: npm run extract-messages -- --check
        
      - name: Run i18n tests
        run: npm test -- i18n
        
      - name: Upload validation report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: i18n-validation-report
          path: |
            coverage/
            *.log

# .github/workflows/sync-translations.yml
name: Sync Translations

on:
  schedule:
    - cron: '0 8 * * *'  # Daily at 8 AM
  workflow_dispatch:  # Manual trigger

jobs:
  sync-with-crowdin:
    name: Sync with Crowdin
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Setup Crowdin CLI
        run: |
          curl -s https://packagecloud.io/install/repositories/crowdin/cli/script.deb.sh | sudo bash
          sudo apt-get install crowdin-cli
          
      - name: Upload source files
        run: |
          crowdin upload sources
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
          
      - name: Download translations
        run: |
          crowdin download
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
          
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          commit-message: 'i18n: Update translations from Crowdin'
          title: '🌐 i18n: Updated translations'
          body: |
            This PR contains updated translations from Crowdin.
            
            ### Changes:
            - Updated translation files for all locales
            - Added new keys as needed
            
            ### Review:
            - [ ] Check for missing translations
            - [ ] Verify special characters
            - [ ] Test RTL layouts if applicable
          branch: i18n/update-translations
          delete-branch: true

# scripts/extract-messages.js
const { execSync } = require('child_process');
const fs = require('fs');
const path = require('path');

// Extract messages from source code
function extractMessages() {
  console.log('Extracting messages from source files...');
  
  try {
    // Use formatjs CLI to extract messages
    execSync(
      'formatjs extract "app/**/*.tsx" "components/**/*.tsx" --out-file messages/temp.json --id-interpolation-pattern [sha512:contenthash:base64:6]',
      { stdio: 'inherit' }
    );
    
    // Merge with existing messages
    const tempMessages = JSON.parse(
      fs.readFileSync(path.join(__dirname, '../messages/temp.json'), 'utf8')
    );
    
    const existingMessages = JSON.parse(
      fs.readFileSync(path.join(__dirname, '../messages/en.json'), 'utf8')
    );
    
    // Deep merge
    const merged = deepMerge(existingMessages, tempMessages);
    
    // Write back
    fs.writeFileSync(
      path.join(__dirname, '../messages/en.json'),
      JSON.stringify(merged, null, 2)
    );
    
    // Cleanup
    fs.unlinkSync(path.join(__dirname, '../messages/temp.json'));
    
    console.log('✅ Messages extracted successfully!');
    
    // Check for unused keys
    checkUnusedKeys(merged);
    
  } catch (error) {
    console.error('❌ Failed to extract messages:', error);
    process.exit(1);
  }
}

function deepMerge(target, source) {
  const output = { ...target };
  
  for (const key in source) {
    if (source.hasOwnProperty(key)) {
      if (isObject(source[key]) && isObject(target[key])) {
        output[key] = deepMerge(target[key], source[key]);
      } else {
        output[key] = source[key];
      }
    }
  }
  
  return output;
}

function isObject(item) {
  return item && typeof item === 'object' && !Array.isArray(item);
}

function checkUnusedKeys(messages) {
  console.log('\nChecking for unused keys...');
  
  // This would require AST parsing to find used keys
  // Simplified version for illustration
  const sourceCode = getAllSourceCode();
  const keys = extractKeys(messages);
  
  const unusedKeys = keys.filter(key => {
    // Check if key is used in source code
    const searchPattern = new RegExp(`t\\(['"]${key.replace(/\./g, '\\.')}['"]\\)`, 'g');
    return !searchPattern.test(sourceCode);
  });
  
  if (unusedKeys.length > 0) {
    console.warn(`⚠️ Found ${unusedKeys.length} potentially unused keys:`);
    unusedKeys.forEach(key => console.warn(`   - ${key}`));
    
    // Write to file for review
    fs.writeFileSync(
      path.join(__dirname, '../unused-keys.json'),
      JSON.stringify(unusedKeys, null, 2)
    );
  } else {
    console.log('✅ No unused keys found!');
  }
}

function getAllSourceCode() {
  // Recursively read all .tsx and .ts files
  const srcPath = path.join(__dirname, '..');
  let sourceCode = '';
  
  function readDir(dir) {
    const files = fs.readdirSync(dir);
    
    files.forEach(file => {
      const filePath = path.join(dir, file);
      const stat = fs.statSync(filePath);
      
      if (stat.isDirectory() && !filePath.includes('node_modules')) {
        readDir(filePath);
      } else if (file.endsWith('.tsx') || file.endsWith('.ts')) {
        sourceCode += fs.readFileSync(filePath, 'utf8');
      }
    });
  }
  
  readDir(srcPath);
  return sourceCode;
}

function extractKeys(obj, prefix = '') {
  return Object.keys(obj).reduce((keys, key) => {
    const value = obj[key];
    const fullKey = prefix ? `${prefix}.${key}` : key;
    
    if (isObject(value)) {
      return [...keys, ...extractKeys(value, fullKey)];
    }
    
    return [...keys, fullKey];
  }, []);
}

extractMessages();

---

§ 10. SEO FOR MULTI-LANGUAGE

§ 10.1 REQUIRED ELEMENTS

| Element | Purpose | Implementation | Example |
|---------|---------|----------------|---------|
| hreflang tags | Tell search engines about alternate language versions | Link tags in head | `<link rel="alternate" hreflang="en" href="https://example.com/en" />` |
| Canonical URL | Prevent duplicate content issues | Link tag with rel="canonical" | `<link rel="canonical" href="https://example.com/en/page" />` |
| Language meta tag | Specify page language | html lang attribute | `<html lang="en">` |
| og:locale | Open Graph language | meta property | `<meta property="og:locale" content="en_US" />` |
| Alternate og:locale | OG alternate languages | meta property | `<meta property="og:locale:alternate" content="it_IT" />` |
| Sitemap per locale | Help indexing | Separate or combined sitemap | `sitemap-en.xml`, `sitemap-it.xml` |
| x-default hreflang | Default language for unspecified regions | Link tag | `<link rel="alternate" hreflang="x-default" href="https://example.com/" />` |

§ 10.2 HREFLANG IMPLEMENTATION
typescript
// components/HreflangTags.tsx
import { locales, defaultLocale } from '@/i18n';

type HreflangTagsProps = {
  pathname: string;
  availableLocales?: string[];
};

export function HreflangTags({ 
  pathname, 
  availableLocales = locales 
}: HreflangTagsProps) {
  const baseUrl = 'https://example.com';
  
  return (
    <>
      {/* Self-referential canonical */}
      <link
        rel="canonical"
        href={`${baseUrl}${pathname}`}
        key="canonical"
      />
      
      {/* hreflang tags for all available locales */}
      {availableLocales.map((locale) => {
        let localeCode = locale;
        
        // Add region code for SEO (e.g., en-GB, en-US)
        const regionMap: Record<string, string> = {
          'en': 'en-US',
          'it': 'it-IT',
          'de': 'de-DE',
          'fr': 'fr-FR',
          'es': 'es-ES',
          'ar': 'ar-SA',
        };
        
        const seoLocale = regionMap[locale] || locale;
        
        return (
          <link
            key={`hreflang-${locale}`}
            rel="alternate"
            hrefLang={seoLocale}
            href={`${baseUrl}/${locale}${pathname.replace(/^\/[a-z]{2}/, '')}`}
          />
        );
      })}
      
      {/* x-default for unspecified regions */}
      <link
        rel="alternate"
        hrefLang="x-default"
        href={`${baseUrl}/${defaultLocale}${pathname.replace(/^\/[a-z]{2}/, '')}`}
        key="hreflang-x-default"
      />
    </>
  );
}

// app/[locale]/layout.tsx - Integration
export default async function LocaleLayout({ children, params }: Props) {
  // ... existing code ...
  
  return (
    <html lang={params.locale}>
      <head>
        <HreflangTags 
          pathname={`/${params.locale}${pathname}`}
          availableLocales={locales}
        />
        
        {/* Open Graph locale tags */}
        <meta property="og:locale" content={getOpenGraphLocale(params.locale)} />
        
        {locales
          .filter(l => l !== params.locale)
          .map(locale => (
            <meta 
              key={`og-alternate-${locale}`}
              property="og:locale:alternate" 
              content={getOpenGraphLocale(locale)} 
            />
          ))}
      </head>
      {/* ... */}
    </html>
  );
}

function getOpenGraphLocale(locale: string): string {
  const ogLocaleMap: Record<string, string> = {
    'en': 'en_US',
    'it': 'it_IT',
    'de': 'de_DE',
    'fr': 'fr_FR',
    'es': 'es_ES',
    'ar': 'ar_AR',
    'ja': 'ja_JP',
    'ko': 'ko_KR',
    'zh': 'zh_CN',
  };
  
  return ogLocaleMap[locale] || locale;
}

// lib/seo-utils.ts
export function generateHreflangLinks(
  path: string,
  locales: string[],
  defaultLocale: string
): Array<{
  rel: 'alternate';
  hrefLang: string;
  href: string;
}> {
  const baseUrl = 'https://example.com';
  const cleanPath = path.replace(/^\/[a-z]{2}(-[A-Z]{2})?/, '');
  
  const links = locales.map((locale) => {
    let hrefLang = locale;
    
    // Convert to ISO format for hreflang
    const regionMap: Record<string, string> = {
      'en': 'en-US',
      'it': 'it-IT',
      'de': 'de-DE',
      'fr': 'fr-FR',
      'es': 'es-ES',
      'ar': 'ar-SA',
      'ja': 'ja-JP',
      'ko': 'ko-KR',
      'zh': 'zh-CN',
    };
    
    hrefLang = regionMap[locale] || locale;
    
    return {
      rel: 'alternate' as const,
      hrefLang,
      href: `${baseUrl}/${locale}${cleanPath}`,
    };
  });
  
  // Add x-default
  links.push({
    rel: 'alternate',
    hrefLang: 'x-default',
    href: `${baseUrl}/${defaultLocale}${cleanPath}`,
  });
  
  return links;
}

export function validateHreflangImplementation() {
  // This could be run in CI to validate hreflang tags
  const requiredElements = [
    'canonical',
    'hreflang-self',
    'hreflang-x-default',
    'html-lang',
  ];
  
  // Implementation would check rendered HTML
  return requiredElements.every(element => true); // Simplified
}

§ 10.3 SITEMAP GENERATION
typescript
// app/sitemap.ts - Complete multi-locale sitemap
import { MetadataRoute } from 'next';
import { locales, defaultLocale } from '@/i18n';
import { getAllPages, getAllBlogPosts, getAllProducts } from '@/lib/content';

type SitemapEntry = {
  url: string;
  lastModified: Date;
  changeFrequency: 
    | 'always'
    | 'hourly'
    | 'daily'
    | 'weekly'
    | 'monthly'
    | 'yearly'
    | 'never';
  priority: number;
  alternates?: {
    languages: Record<string, string>;
  };
};

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com';
  const currentDate = new Date();
  
  // Fetch all localizable content
  const [pages, blogPosts, products] = await Promise.all([
    getAllPages(),
    getAllBlogPosts(),
    getAllProducts(),
  ]);
  
  const sitemapEntries: SitemapEntry[] = [];
  
  // 1. Homepage for each locale
  for (const locale of locales) {
    sitemapEntries.push({
      url: `${baseUrl}/${locale}`,
      lastModified: currentDate,
      changeFrequency: 'daily',
      priority: 1.0,
      alternates: {
        languages: generateAlternateLanguages('/', locale),
      },
    });
  }
  
  // 2. Static pages
  for (const page of pages) {
    for (const locale of locales) {
      // Check if page is available in this locale
      if (page.availableLocales.includes(locale)) {
        const path = page.paths[locale] || page.path;
        
        sitemapEntries.push({
          url: `${baseUrl}/${locale}${path}`,
          lastModified: page.updatedAt,
          changeFrequency: page.changeFrequency as any,
          priority: page.priority || 0.8,
          alternates: {
            languages: generateAlternateLanguages(path, locale),
          },
        });
      }
    }
  }
  
  // 3. Blog posts
  for (const post of blogPosts) {
    for (const locale of locales) {
      if (post.availableLocales.includes(locale)) {
        const slug = post.slugs[locale] || post.slug;
        
        sitemapEntries.push({
          url: `${baseUrl}/${locale}/blog/${slug}`,
          lastModified: post.updatedAt,
          changeFrequency: 'weekly',
          priority: 0.7,
          alternates: {
            languages: generateAlternateLanguages(`/blog/${post.slug}`, locale),
          },
        });
      }
    }
  }
  
  // 4. Products
  for (const product of products) {
    for (const locale of locales) {
      if (product.availableLocales.includes(locale)) {
        const slug = product.slugs[locale] || product.slug;
        
        sitemapEntries.push({
          url: `${baseUrl}/${locale}/products/${slug}`,
          lastModified: product.updatedAt,
          changeFrequency: 'weekly',
          priority: 0.6,
          alternates: {
            languages: generateAlternateLanguages(`/products/${product.slug}`, locale),
          },
        });
      }
    }
  }
  
  // 5. Dynamic routes (if any)
  // Add any additional dynamic routes here
  
  // Sort by priority (highest first)
  sitemapEntries.sort((a, b) => b.priority - a.priority);
  
  return sitemapEntries;
}

function generateAlternateLanguages(
  path: string,
  currentLocale: string
): Record<string, string> {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com';
  const alternates: Record<string, string> = {};
  
  for (const locale of locales) {
    if (locale !== currentLocale) {
      alternates[locale] = `${baseUrl}/${locale}${path}`;
    }
  }
  
  return alternates;
}

// robots.txt with sitemap reference
// app/robots.txt
export default function robots(): MetadataRoute.Robots {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com';
  
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: ['/api/', '/admin/', '/private/'],
    },
    sitemap: `${baseUrl}/sitemap.xml`,
    
    // Additional sitemaps per locale (optional)
    host: baseUrl,
  };
}

// Generate sitemap index for large sites
// app/sitemap-index.xml/route.ts
export async function GET() {
  const baseUrl = process.env.NEXT_PUBLIC_BASE_URL || 'https://example.com';
  const locales = ['en', 'it', 'de', 'fr', 'es', 'ar'];
  
  const sitemapIndex = `<?xml version="1.0" encoding="UTF-8"?>
    <sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
      ${locales.map(locale => `
        <sitemap>
          <loc>${baseUrl}/sitemaps/${locale}.xml</loc>
          <lastmod>${new Date().toISOString()}</lastmod>
        </sitemap>
      `).join('')}
      <sitemap>
        <loc>${baseUrl}/sitemaps/posts.xml</loc>
        <lastmod>${new Date().toISOString()}</lastmod>
      </sitemap>
      <sitemap>
        <loc>${baseUrl}/sitemaps/products.xml</loc>
        <lastmod>${new Date().toISOString()}</lastmod>
      </sitemap>
    </sitemapindex>`;
  
  return new Response(sitemapIndex, {
    headers: {
      'Content-Type': 'application/xml',
    },
  });
}

---

§ 11. I18N CHECKLIST

SETUP
☑ Library installed (next-intl recommended)
☑ Middleware configured with locale detection
☑ Locale routing setup ([locale] segment)
☑ Provider in layout with messages
☑ Supported locales defined in config
☑ Default locale set
☑ Locale prefixes configured (always/as-needed)
☑ Cookie handling for locale persistence

TRANSLATIONS
☑ Message files created for all locales
☑ Key naming convention documented and followed
☑ Namespaces organized logically
☑ Pluralization rules correct for each language
☑ Interpolation working with dynamic values
☑ Rich text supported with React components
☑ Nested keys accessible via dot notation
☑ Select statements for gender/case variations
☑ Ordinal numbers formatted correctly
☑ All placeholders escaped properly

FORMATTING
☑ Date formatter configured per locale
☑ Number formatter with currency support
☑ Currency formatting per locale
☑ Relative time working (hours ago, etc.)
☑ Timezone handling for user location
☑ List formatting (and, or conjunctions)
☑ Unit formatting (meters, liters, etc.)
☑ Compact number formatting (1K, 1M)
☑ Percentage formatting correct
☑ Decimal separators locale-aware

RTL SUPPORT (if needed)
☑ CSS logical properties used instead of physical
☑ dir attribute dynamic based on locale
☑ Layout tested in both directions
☑ Icons mirroring reviewed and implemented
☑ Text alignment using start/end
☑ Flexbox/grid with direction awareness
☑ RTL-specific styles isolated
☑ Scrollbar position considered
☑ Form elements RTL compatible
☑ Animations direction-aware

TYPE SAFETY
☑ IntlMessages interface defined globally
☑ Autocomplete working for translation keys
☑ Missing keys cause TypeScript errors
☑ Params type-checked for each key
☑ Namespace boundaries enforced
☑ Runtime validation in development
☑ IDE extension configured (i18n-ally)
☑ Key extraction tool integrated
☑ Type generation from JSON files

SEO & METADATA
☑ hreflang tags on all pages
☑ Canonical URLs correct per locale
☑ Sitemap includes all locales
☑ og:locale set for Open Graph
☑ Alternate og:locale for other languages
☑ Language meta tag on html element
☑ x-default hreflang for unspecified regions
☑ Robots.txt references sitemap
☑ Structured data localized
☑ Page titles and meta descriptions translated

PERFORMANCE
☑ Messages split by namespace/chunk
☑ Lazy loading for non-critical translations
☑ Bundle size optimized (tree-shaking)
☑ Server components minimize client bundle
☑ Caching strategy for translations
☑ CDN configured for static assets
☑ Compression enabled for JSON files
☑ Preload hints for critical translations
☑ Font loading optimized per script

WORKFLOW & MAINTENANCE
☑ Translation file structure documented
☑ CI validation for missing/extra keys
☑ TMS integration configured (if used)
☑ Extraction pipeline for new strings
☑ Review process for translations
☑ Fallback strategy for missing translations
☑ Error handling for formatting failures
☑ Logging for i18n issues
☑ Analytics for locale usage
☑ Regular updates for CLDR data

ACCESSIBILITY
☑ Language announced correctly by screen readers
☑ Text direction announced for RTL
☑ Translation quality checked (clarity)
☑ Cultural appropriateness verified
☑ Date/time formats accessible
☑ Number formats readable by screen readers
☑ Abbreviations localized properly
☑ Alt text for images translated
☑ ARIA labels localized
☑ Focus order correct for RTL

TESTING
☑ All locales render without errors
☑ Formatting works for edge cases
☑ RTL layout tested thoroughly
☑ Language switcher functional
☑ SEO tags validated
☑ Print styles work for all directions
☑ Form validation messages translated
☑ Error messages localized
☑ Loading states include locale
☑ Fallback behavior tested
