# CATALOGO-BOILERPLATE-STRUTTURA-v1

> **Dominio**: Struttura file e codice base Next.js 14
> **Stack**: Next.js 14 App Router, TypeScript, Prisma, tRPC
> **Versione**: 1.0
> **Data**: 2026-01-29

---

§ 1. INDICE

| Sezione | Descrizione |
|---------|-------------|
| [1. Panoramica](#1-panoramica) | Architettura e convenzioni |
| [2. App Layer](#2-app-layer) | Layout, pagine, loading, error |
| [3. Lib Layer](#3-lib-layer) | Utils, constants, prisma, env |
| [4. Components Layer](#4-components-layer) | Providers e UI base |
| [5. Types Layer](#5-types-layer) | Type definitions |
| [6. Hooks Layer](#6-hooks-layer) | Custom React hooks |
| [7. Server Layer](#7-server-layer) | tRPC setup |
| [8. Database Layer](#8-database-layer) | Schema Prisma |

---

§ 1. PANORAMICA

§ 1.1 OBIETTIVO
Struttura completa di file per un progetto Next.js 14 con App Router, pronta per lo sviluppo.

§ 1.2 ARCHITETTURA DIRECTORY

src/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root layout
│   ├── page.tsx            # Homepage
│   ├── loading.tsx         # Global loading
│   ├── error.tsx           # Error boundary
│   ├── not-found.tsx       # 404 page
│   ├── globals.css         # Global styles
│   └── api/                # API routes
│       └── trpc/[trpc]/    # tRPC handler
├── components/             # React components
│   ├── providers.tsx       # Context providers
│   └── ui/                 # UI primitives
├── lib/                    # Utilities
│   ├── utils.ts            # Helper functions
│   ├── constants.ts        # App constants
│   ├── prisma.ts           # Prisma client
│   └── env.ts              # Env validation
├── hooks/                  # Custom hooks
├── types/                  # TypeScript types
├── server/                 # Server-side code
│   └── trpc/               # tRPC routers
└── prisma/
    └── schema.prisma       # Database schema

§ 1.3 CONVENZIONI NAMING

| Tipo | Convenzione | Esempio |
|------|-------------|---------|
| Componenti | PascalCase | `Button.tsx` |
| Hooks | camelCase con `use` | `useDebounce.ts` |
| Utils | camelCase | `formatDate.ts` |
| Types | PascalCase | `ApiResponse.ts` |
| Constants | UPPER_SNAKE | `MAX_PAGE_SIZE` |
| Files CSS | kebab-case | `globals.css` |

---

§ 2. APP LAYER

§ 2.1 MAPPA FILE APP ROUTER

| File | Path | Descrizione |
|------|------|-------------|
| Root Layout | `src/app/layout.tsx` | HTML wrapper, providers |
| Homepage | `src/app/page.tsx` | Landing page |
| Loading | `src/app/loading.tsx` | Suspense fallback |
| Error | `src/app/error.tsx` | Error boundary |
| Not Found | `src/app/not-found.tsx` | 404 page |
| Globals | `src/app/globals.css` | Tailwind base |
| tRPC Route | `src/app/api/trpc/[trpc]/route.ts` | API handler |

§ 2.2 ROOT LAYOUT

**File**: `src/app/layout.tsx`

| Responsabilità | Dettaglio |
|----------------|-----------|
| HTML structure | `<html>`, `<head>`, `<body>` |
| Font loading | Google Fonts (Inter) |
| Metadata | SEO base |
| Providers | Context wrapper |
| Analytics | Tracking component |
| Toast | Notification container |

typescript
import type { ReactNode } from 'react';
import { Inter } from 'next/font/google';
import { Metadata } from 'next';
import { ToastContainer } from 'react-toastify';
import { Providers } from '../components/providers';
import { Analytics } from '../components/analytics';

const inter = Inter({ subsets: ['latin'] });

interface LayoutProps {
  children: ReactNode;
}

export default function RootLayout({ children }: LayoutProps) {
  return (
    <html lang="it" className={inter.className}>
      <head>
        <Metadata />
        <link rel="icon" href="/favicon.ico" />
      </head>
      <body>
        <Providers>
          {children}
          <ToastContainer />
          <Analytics />
        </Providers>
      </body>
    </html>
  );
}

§ 2.3 HOMEPAGE

**File**: `src/app/page.tsx`

typescript
import type { ReactNode } from 'react';
import { useState, useEffect } from 'react';
import { useSession } from 'next-auth/react';
import { useRouter } from 'next/router';

export default function HomePage() {
  const { data: session } = useSession();
  const router = useRouter();

  if (session) {
    router.push('/dashboard');
  }

  return (
    <div className="flex justify-center items-center h-screen">
      <h1 className="text-3xl font-bold">Benvenuto!</h1>
    </div>
  );
}

§ 2.4 LOADING STATE

**File**: `src/app/loading.tsx`

typescript
import type { ReactNode } from 'react';

export default function Loading() {
  return (
    <div className="flex justify-center items-center h-screen">
      <div className="spinner-border animate-spin inline-block w-8 h-8 border-4 rounded-full text-gray-200" />
    </div>
  );
}

§ 2.5 ERROR BOUNDARY

**File**: `src/app/error.tsx`

| Metodo | Scopo |
|--------|-------|
| `getDerivedStateFromError` | Cattura errore |
| `componentDidCatch` | Log errore |
| `render` | UI fallback |

typescript
import type { ReactNode, ErrorInfo } from 'react';
import { useState } from 'react';

interface ErrorBoundaryProps {
  children: ReactNode;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

export default class ErrorBoundary extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Errore:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="flex justify-center items-center h-screen">
          <h1 className="text-3xl font-bold">Errore!</h1>
          <p>{this.state.error?.message}</p>
          <button onClick={() => this.setState({ hasError: false, error: null })}>Riprova</button>
        </div>
      );
    }

    return this.props.children;
  }
}

§ 2.6 NOT FOUND PAGE

**File**: `src/app/not-found.tsx`

typescript
import type { ReactNode } from 'react';

export default function NotFoundPage() {
  return (
    <div className="flex justify-center items-center h-screen">
      <h1 className="text-3xl font-bold">Pagina non trovata!</h1>
    </div>
  );
}

§ 2.7 GLOBAL STYLES

**File**: `src/app/globals.css`

| Layer | Contenuto |
|-------|-----------|
| `@layer base` | Stili base globali |
| `@layer components` | Classi componenti |
| `@layer utilities` | Utility custom |
| `:root` | CSS variables |

css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  @apply bg-white text-gray-900;
}

@layer components {
  .btn {
    @apply py-2 px-4 rounded;
  }
}

@layer utilities {
  .container {
    @apply max-w-md mx-auto;
  }
}

:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
}

body {
  font-family: 'Inter', sans-serif;
}

---

§ 3. LIB LAYER

§ 3.1 MAPPA FILE LIB

| File | Path | Descrizione |
|------|------|-------------|
| Utils | `src/lib/utils.ts` | Helper functions |
| Constants | `src/lib/constants.ts` | App constants |
| Prisma | `src/lib/prisma.ts` | DB client |
| Env | `src/lib/env.ts` | Env validation |

§ 3.2 UTILS

**File**: `src/lib/utils.ts`

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `cn` | `...classes` | `string` | Merge class names |
| `formatDate` | `date` | `string` | Format date IT |
| `formatCurrency` | `value` | `string` | Format EUR |
| `formatNumber` | `value` | `string` | Format number IT |
| `slugify` | `str` | `string` | Create URL slug |
| `truncate` | `str, length` | `string` | Truncate text |
| `debounce` | `fn, delay` | `Function` | Debounce function |
| `sleep` | `ms` | `Promise` | Async delay |
| `generateId` | - | `string` | UUID generator |
| `isValidEmail` | `email` | `boolean` | Email validation |
| `parseJSON` | `json` | `any \| null` | Safe JSON parse |

typescript
export function cn(...classes: string[]) {
  return classes.filter(Boolean).join(' ');
}

export function formatDate(date: Date) {
  return date.toLocaleDateString('it-IT');
}

export function formatCurrency(value: number) {
  return value.toLocaleString('it-IT', { style: 'currency', currency: 'EUR' });
}

export function formatNumber(value: number) {
  return value.toLocaleString('it-IT');
}

export function slugify(str: string) {
  return str.toLowerCase().replace(/ /g, '-');
}

export function truncate(str: string, length: number) {
  return str.length > length ? str.substring(0, length) + '...' : str;
}

export function debounce(fn: Function, delay: number) {
  let timeout: NodeJS.Timeout;
  return (...args: any[]) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn(...args), delay);
  };
}

export function sleep(ms: number) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

export function generateId() {
  return crypto.randomUUID();
}

export function isValidEmail(email: string) {
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  return emailRegex.test(email);
}

export function parseJSON(json: string) {
  try {
    return JSON.parse(json);
  } catch (error) {
    return null;
  }
}

§ 3.3 CONSTANTS

**File**: `src/lib/constants.ts`

| Costante | Tipo | Descrizione |
|----------|------|-------------|
| `APP_NAME` | string | Nome applicazione |
| `APP_DESCRIPTION` | string | Descrizione app |
| `APP_URL` | string | URL base |
| `PAGINATION` | object | Config paginazione |
| `ROUTES` | object | Route principali |

typescript
export const APP_NAME = 'Platform';
export const APP_DESCRIPTION = 'Descrizione dell\'applicazione';
export const APP_URL = process.env.NEXT_PUBLIC_APP_URL;

export const PAGINATION = {
  DEFAULT_PAGE_SIZE: 10,
  MAX_PAGE_SIZE: 100,
} as const;

export const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  REGISTER: '/register',
  DASHBOARD: '/dashboard',
} as const;

§ 3.4 PRISMA CLIENT

**File**: `src/lib/prisma.ts`

typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

prisma.$use(async (params, next) => {
  const result = await next(params);
  console.log('Query:', params, result);
  return result;
});

export default prisma;

§ 3.5 ENV VALIDATION

**File**: `src/lib/env.ts`

| Variabile | Schema | Required |
|-----------|--------|----------|
| `DATABASE_URL` | `z.string().url()` | Yes |
| `NEXTAUTH_URL` | `z.string().url()` | Yes |
| `NEXTAUTH_SECRET` | `z.string().min(32)` | Yes |

typescript
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);

---

§ 4. COMPONENTS LAYER

§ 4.1 MAPPA COMPONENTI

| File | Path | Descrizione |
|------|------|-------------|
| Providers | `src/components/providers.tsx` | Context wrapper |
| UI Index | `src/components/ui/index.ts` | Barrel export |
| Button | `src/components/ui/button.tsx` | Button component |
| Input | `src/components/ui/input.tsx` | Input component |
| Card | `src/components/ui/card.tsx` | Card components |
| Toast | `src/components/ui/toast.tsx` | Toast system |

§ 4.2 PROVIDERS

**File**: `src/components/providers.tsx`

| Provider | Package | Scopo |
|----------|---------|-------|
| `QueryClientProvider` | react-query | Data fetching |
| `SessionProvider` | next-auth | Auth state |
| `ThemeProvider` | next-themes | Dark mode |
| `ToastProvider` | react-toastify | Notifications |

typescript
import { QueryClientProvider } from 'react-query';
import { SessionProvider } from 'next-auth/react';
import { ThemeProvider } from 'next-themes';
import { ToastProvider } from 'react-toastify';

const queryClient = new QueryClient();

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <QueryClientProvider client={queryClient}>
      <SessionProvider>
        <ThemeProvider>
          <ToastProvider>{children}</ToastProvider>
        </ThemeProvider>
      </SessionProvider>
    </QueryClientProvider>
  );
}

§ 4.3 BUTTON COMPONENT

**File**: `src/components/ui/button.tsx`

| Prop | Tipo | Default | Descrizione |
|------|------|---------|-------------|
| `variant` | enum | `default` | Stile visivo |
| `size` | enum | `default` | Dimensione |
| `loading` | boolean | false | Loading state |
| `disabled` | boolean | false | Disabled state |
| `asChild` | boolean | false | Render as child |

| Variant | Stile |
|---------|-------|
| `default` | Primary background |
| `destructive` | Red/danger |
| `outline` | Border only |
| `secondary` | Secondary background |
| `ghost` | Transparent |
| `link` | Text link |

typescript
import type { ReactNode } from 'react';

interface ButtonProps {
  children: ReactNode;
  variant: 'default' | 'destructive' | 'outline' | 'secondary' | 'ghost' | 'link';
  size: 'default' | 'sm' | 'lg' | 'icon';
  loading: boolean;
  disabled: boolean;
  asChild: boolean;
}

export function Button({ children, variant, size, loading, disabled, asChild }: ButtonProps) {
  const className = cn(
    'btn',
    variant === 'default' && 'bg-primary text-white',
    variant === 'destructive' && 'bg-red-500 text-white',
    variant === 'outline' && 'bg-transparent border border-primary text-primary',
    variant === 'secondary' && 'bg-secondary text-white',
    variant === 'ghost' && 'bg-transparent text-primary',
    variant === 'link' && 'text-primary',
    size === 'sm' && 'py-1 px-2',
    size === 'lg' && 'py-3 px-4',
    size === 'icon' && 'p-2',
    loading && 'opacity-50 cursor-not-allowed',
    disabled && 'opacity-50 cursor-not-allowed'
  );

  if (asChild) {
    return <div className={className}>{children}</div>;
  }

  return (
    <button className={className} disabled={disabled || loading}>
      {children}
    </button>
  );
}

§ 4.4 INPUT COMPONENT

**File**: `src/components/ui/input.tsx`

typescript
import type { ReactNode } from 'react';
import { forwardRef } from 'react';

interface InputProps {
  children: ReactNode;
  variant: 'default' | 'error';
  icon: ReactNode;
}

export const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ children, variant, icon, ...props }, ref) => {
    const className = cn(
      'input',
      variant === 'default' && 'border border-gray-300',
      variant === 'error' && 'border border-red-500'
    );

    return (
      <div className="flex items-center">
        {icon && <div className="mr-2">{icon}</div>}
        <input ref={ref} className={className} {...props} />
      </div>
    );
  }
);

§ 4.5 CARD COMPONENTS

**File**: `src/components/ui/card.tsx`

| Componente | Descrizione |
|------------|-------------|
| `Card` | Container principale |
| `CardHeader` | Header area |
| `CardTitle` | Titolo card |
| `CardDescription` | Sottotitolo |
| `CardContent` | Contenuto principale |
| `CardFooter` | Footer area |

typescript
import type { ReactNode } from 'react';

interface CardProps {
  children: ReactNode;
}

export function Card({ children }: CardProps) {
  return <div className="card">{children}</div>;
}

export function CardHeader({ children }: { children: ReactNode }) {
  return <div className="card-header">{children}</div>;
}

export function CardTitle({ children }: { children: ReactNode }) {
  return <h2 className="card-title">{children}</h2>;
}

export function CardDescription({ children }: { children: ReactNode }) {
  return <p className="card-description">{children}</p>;
}

export function CardContent({ children }: { children: ReactNode }) {
  return <div className="card-content">{children}</div>;
}

export function CardFooter({ children }: { children: ReactNode }) {
  return <div className="card-footer">{children}</div>;
}

§ 4.6 UI BARREL EXPORT

**File**: `src/components/ui/index.ts`

typescript
export * from './button';
export * from './input';
export * from './card';
export * from './toast';

---

§ 5. TYPES LAYER

§ 5.1 TYPES PRINCIPALI

**File**: `src/types/index.ts`

| Type | Descrizione |
|------|-------------|
| `ApiResponse<T>` | Response wrapper API |
| `PaginatedResponse<T>` | Response paginata |
| `ApiError` | Error structure |
| `AsyncFunction<T>` | Async function type |

typescript
export interface ApiResponse<T> {
  data: T;
  error: null | string;
}

export interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    totalPages: number;
    totalItems: number;
  };
}

export interface ApiError {
  message: string;
  code: number;
}

export type AsyncFunction<T> = () => Promise<T>;

---

§ 6. HOOKS LAYER

§ 6.1 MAPPA HOOKS

| Hook | File | Descrizione |
|------|------|-------------|
| `useDebounce` | `use-debounce.ts` | Debounce values |
| `useLocalStorage` | `use-local-storage.ts` | Persist to localStorage |
| `useMediaQuery` | `use-media-query.ts` | Responsive queries |

§ 6.2 USEDEBOUNCE

**File**: `src/hooks/use-debounce.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `value` | `T` | Valore da debounceare |
| `delay` | `number` | Delay in ms |
| **Return** | `T` | Valore debouncato |

typescript
import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timeout = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timeout);
    };
  }, [value, delay]);

  return debouncedValue;
}

§ 6.3 USELOCALSTORAGE

**File**: `src/hooks/use-local-storage.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `key` | `string` | Chiave storage |
| `initialValue` | `T` | Valore default |
| **Return** | `[T, SetState<T>]` | Value e setter |

typescript
import { useState, useEffect } from 'react';

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    const storedValue = localStorage.getItem(key);
    return storedValue ? JSON.parse(storedValue) : initialValue;
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [value, key]);

  return [value, setValue] as const;
}

§ 6.4 USEMEDIAQUERY

**File**: `src/hooks/use-media-query.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `query` | `string` | Media query CSS |
| **Return** | `boolean` | Match result |

typescript
import { useState, useEffect } from 'react';

export function useMediaQuery(query: string) {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    setMatches(mediaQuery.matches);

    const listener = () => {
      setMatches(mediaQuery.matches);
    };

    mediaQuery.addEventListener('change', listener);

    return () => {
      mediaQuery.removeEventListener('change', listener);
    };
  }, [query]);

  return matches;
}

---

§ 7. SERVER LAYER

§ 7.1 TRPC SETUP

| File | Path | Descrizione |
|------|------|-------------|
| Main Router | `src/server/trpc/trpc.ts` | tRPC config |
| App Router | `src/server/trpc/routers/_app.ts` | Root router |
| API Route | `src/app/api/trpc/[trpc]/route.ts` | Next.js handler |

§ 7.2 TRPC MAIN

**File**: `src/server/trpc/trpc.ts`

typescript
import * as trpc from '@trpc/server';
import * as trpcNext from '@trpc/server/adapters/next';
import { z } from 'zod';

export const appRouter = trpc.router()
  .query('hello', {
    input: z.string(),
    async resolve({ input }) {
      return `Hello, ${input}!`;
    },
  })
  .mutation('createUser', {
    input: z.object({
      name: z.string(),
      email: z.string().email(),
    }),
    async resolve({ input }) {
      return { id: 1, name: input.name, email: input.email };
    },
  });

export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});

§ 7.3 APP ROUTER

**File**: `src/server/trpc/routers/_app.ts`

typescript
import * as trpc from '@trpc/server';
import { z } from 'zod';

export const appRouter = trpc.router()
  .query('getUser', {
    input: z.string(),
    async resolve({ input }) {
      return { id: 1, name: 'John Doe', email: 'john.doe@example.com' };
    },
  });

§ 7.4 API ROUTE HANDLER

**File**: `src/app/api/trpc/[trpc]/route.ts`

typescript
import * as trpcNext from '@trpc/server/adapters/next';
import { appRouter } from '../../../server/trpc/trpc';

export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});

---

§ 8. DATABASE LAYER

§ 8.1 SCHEMA PRISMA

**File**: `prisma/schema.prisma`

| Modello | Campi Principali | Relazioni |
|---------|------------------|-----------|
| `User` | id, name, email, password | Session[], Account[] |
| `Session` | id, userId, expires | User |
| `Account` | id, userId, provider, accessToken | User |
| `VerificationToken` | id, identifier, token, expires | - |

prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

model User {
  id       String   @id @default(cuid())
  name     String
  email    String   @unique
  password String
  sessions Session[]
}

model Session {
  id       String   @id @default(cuid())
  userId   String
  user     User     @relation(fields: [userId], references: [id])
  expires  DateTime
}

model Account {
  id         String @id @default(cuid())
  userId     String
  user       User   @relation(fields: [userId], references: [id])
  type       String
  provider   String
  idProvider String
  accessToken String
}

model VerificationToken {
  id         String   @id @default(cuid())
  identifier String
  token      String   @unique
  expires    DateTime
}

---

§ 10. CHECKLIST SETUP

| Step | File | Status |
|------|------|--------|
| 1 | Creare `src/app/layout.tsx` | ⬜ |
| 2 | Creare `src/app/page.tsx` | ⬜ |
| 3 | Creare `src/app/loading.tsx` | ⬜ |
| 4 | Creare `src/app/error.tsx` | ⬜ |
| 5 | Creare `src/app/not-found.tsx` | ⬜ |
| 6 | Creare `src/app/globals.css` | ⬜ |
| 7 | Creare `src/lib/utils.ts` | ⬜ |
| 8 | Creare `src/lib/constants.ts` | ⬜ |
| 9 | Creare `src/lib/prisma.ts` | ⬜ |
| 10 | Creare `src/lib/env.ts` | ⬜ |
| 11 | Creare `src/components/providers.tsx` | ⬜ |
| 12 | Creare componenti UI base | ⬜ |
| 13 | Creare `src/types/index.ts` | ⬜ |
| 14 | Creare hooks custom | ⬜ |
| 15 | Setup tRPC | ⬜ |
| 16 | Creare `prisma/schema.prisma` | ⬜ |
| 17 | Eseguire `npx prisma generate` | ⬜ |

---

_Integrato da: 04-OUTPUT-BOILERPLATE-STRUTTURA.md_
_Data integrazione: 2026-01-29 14:51_
_Generato con: Gemini 2.5 Flash / DeepSeek R1_

CATALOGO-BOILERPLATE-STRUTTURA-v1
1. Struttura Directory Standard
1.1 Landing Page
text
Copia
Scarica
landing-page/
├── .next/
├── public/
│   ├── fonts/
│   ├── images/
│   │   ├── hero/
│   │   └── logos/
│   └── favicon.ico
├── src/
│   ├── app/
│   │   ├── (marketing)/          # Gruppo di route non affette da layout
│   │   │   ├── page.tsx
│   │   │   ├── layout.tsx
│   │   │   └── pricing/
│   │   │       └── page.tsx
│   │   ├── (legal)/             # Termini, privacy
│   │   │   ├── terms/
│   │   │   └── privacy/
│   │   ├── api/
│   │   │   └── contact/
│   │   │       └── route.ts
│   │   ├── layout.tsx          # Root layout
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                 # Componenti riutilizzabili
│   │   │   ├── button.tsx
│   │   │   └── card.tsx
│   │   ├── sections/           # Sezioni della landing
│   │   │   ├── HeroSection.tsx
│   │   │   ├── FeaturesSection.tsx
│   │   │   └── CTASection.tsx
│   │   └── shared/
│   │       ├── Header.tsx
│   │       └── Footer.tsx
│   ├── hooks/
│   │   └── useScrollPosition.ts
│   ├── lib/
│   │   └── utils.ts
│   └── types/
│       └── index.ts
├── .env.local
├── next.config.ts
├── package.json
└── tsconfig.json

File contenuto iniziale - src/app/(marketing)/layout.tsx:

tsx
Copia
Scarica
import { ReactNode } from 'react';
import { Header } from '@/components/shared/Header';
import { Footer } from '@/components/shared/Footer';

export default function MarketingLayout({
  children,
}: {
  children: ReactNode;
}) {
  return (
    <div className="min-h-screen flex flex-col">
      <Header />
      <main className="flex-1">
        {children}
      </main>
      <Footer />
    </div>
  );
}

File contenuto iniziale - src/components/sections/HeroSection.tsx:

tsx
Copia
Scarica
import { Button } from '@/components/ui/button';
import { ArrowRight } from 'lucide-react';

interface HeroSectionProps {
  title: string;
  subtitle: string;
  ctaText: string;
  ctaLink: string;
}

export function HeroSection({
  title = "Build your next idea faster",
  subtitle = "A complete starter kit for your Next.js projects with TypeScript, Tailwind, and best practices included.",
  ctaText = "Get Started",
  ctaLink = "/signup"
}: HeroSectionProps) {
  return (
    <section className="relative overflow-hidden py-20 lg:py-32">
      <div className="container mx-auto px-4 text-center">
        <h1 className="text-4xl md:text-6xl lg:text-7xl font-bold tracking-tight mb-6">
          {title}
        </h1>
        <p className="text-xl text-gray-600 dark:text-gray-300 mb-10 max-w-3xl mx-auto">
          {subtitle}
        </p>
        <Button size="lg" className="gap-2">
          {ctaText}
          <ArrowRight className="h-4 w-4" />
        </Button>
      </div>
    </section>
  );
}
1.2 SaaS Platform
text
Copia
Scarica
saas-platform/
├── src/
│   ├── app/
│   │   ├── (auth)/             # Routes autenticazione
│   │   │   ├── login/
│   │   │   ├── register/
│   │   │   └── forgot-password/
│   │   ├── (dashboard)/        # Area protetta
│   │   │   ├── layout.tsx      # Layout con sidebar
│   │   │   ├── page.tsx        # Dashboard home
│   │   │   ├── analytics/
│   │   │   ├── settings/
│   │   │   └── billing/
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   ├── webhooks/
│   │   │   └── trpc/          # tRPC router
│   │   ├── admin/             # Admin area
│   │   │   └── layout.tsx     # Admin layout separato
│   │   └── layout.tsx
│   ├── components/
│   │   ├── dashboard/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Navbar.tsx
│   │   │   └── StatsCards.tsx
│   │   ├── forms/            # Form complessi
│   │   │   ├── OnboardingForm.tsx
│   │   │   └── BillingForm.tsx
│   │   └── providers/        # Context providers
│   │       ├── ThemeProvider.tsx
│   │       └── AuthProvider.tsx
│   ├── lib/
│   │   ├── db/
│   │   │   └── prisma.ts
│   │   ├── auth/
│   │   │   └── auth.ts       # NextAuth o alternative
│   │   ├── api/             # Client API
│   │   │   └── client.ts
│   │   └── utils/
│   │       ├── validators.ts # Zod schemas
│   │       └── constants.ts
│   ├── hooks/
│   │   ├── useSubscription.ts
│   │   └── useUser.ts
│   ├── types/
│   │   └── database.ts      # Tipi generati da Prisma
│   └── styles/
│       └── globals.css
├── prisma/
│   ├── schema.prisma
│   └── seeds/
├── tests/
│   ├── e2e/
│   └── unit/
└── middleware.ts

File contenuto iniziale - src/app/(dashboard)/layout.tsx:

tsx
Copia
Scarica
import { ReactNode } from 'react';
import { DashboardSidebar } from '@/components/dashboard/Sidebar';
import { DashboardNavbar } from '@/components/dashboard/Navbar';
import { auth } from '@/lib/auth/auth';

export default async function DashboardLayout({
  children,
}: {
  children: ReactNode;
}) {
  const session = await auth();
  
  if (!session) {
    redirect('/login');
  }

  return (
    <div className="min-h-screen bg-gray-50 dark:bg-gray-900">
      <DashboardSidebar />
      <div className="lg:pl-64">
        <DashboardNavbar user={session.user} />
        <main className="py-6">
          <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
            {children}
          </div>
        </main>
      </div>
    </div>
  );
}
1.3 E-commerce
text
Copia
Scarica
ecommerce/
├── src/
│   ├── app/
│   │   ├── (store)/          # Frontend store
│   │   │   ├── products/
│   │   │   │   ├── [id]/
│   │   │   │   └── category/
│   │   │   ├── cart/
│   │   │   ├── checkout/
│   │   │   └── order/
│   │   ├── account/          # Customer account
│   │   ├── api/
│   │   │   ├── cart/
│   │   │   ├── checkout/
│   │   │   └── stripe/
│   │   └── layout.tsx
│   ├── components/
│   │   ├── store/
│   │   │   ├── ProductCard.tsx
│   │   │   ├── ProductGrid.tsx
│   │   │   └── CartDrawer.tsx
│   │   ├── checkout/
│   │   │   ├── CheckoutForm.tsx
│   │   │   └── OrderSummary.tsx
│   │   └── layout/
│   │       ├── StoreHeader.tsx
│   │       └── CategoryNav.tsx
│   ├── lib/
│   │   ├── stripe/
│   │   ├── inventory/
│   │   └── cart/
│   │       └── store.ts     # Cart state management
│   ├── hooks/
│   │   └── useCart.ts
│   └── data/
│       └── products.ts     # Mock data o CMS integration
1.4 Blog/CMS
text
Copia
Scarica
blog-cms/
├── src/
│   ├── app/
│   │   ├── blog/
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx
│   │   │   ├── category/
│   │   │   │   └── [category]/
│   │   │   └── tag/
│   │   │       └── [tag]/
│   │   ├── admin/          # Editor interface
│   │   │   └── posts/
│   │   └── layout.tsx
│   ├── components/
│   │   ├── blog/
│   │   │   ├── PostCard.tsx
│   │   │   ├── TableOfContents.tsx
│   │   │   └── AuthorBio.tsx
│   │   ├── cms/
│   │   │   ├── Editor/
│   │   │   └── MediaUpload.tsx
│   │   └── shared/
│   │       └── Search.tsx
│   ├── lib/
│   │   ├── markdown/
│   │   │   └── processor.ts
│   │   ├── content/        # Contentlayer o simili
│   │   └── redis/          # Caching
│   └── content/
│       ├── posts/
│       └── authors/
1.5 Admin Dashboard
text
Copia
Scarica
admin-dashboard/
├── src/
│   ├── app/
│   │   ├── admin/
│   │   │   ├── layout.tsx      # Admin layout con auth
│   │   │   ├── dashboard/
│   │   │   ├── users/
│   │   │   ├── content/
│   │   │   ├── settings/
│   │   │   └── analytics/
│   │   └── api/
│   │       └── admin/
│   ├── components/
│   │   ├── admin/
│   │   │   ├── DataTable/
│   │   │   ├── Charts/
│   │   │   └── Modals/
│   │   └── ui/
│   │       └── table.tsx      # Shadcn/ui table
│   ├── lib/
│   │   ├── admin/
│   │   │   └── permissions.ts
│   │   └── api/
│   │       └── admin-client.ts
│   └── hooks/
│       └── useAdminQuery.ts
2. next.config.ts Patterns
2.1 Configurazione Base per SaaS
typescript
Copia
Scarica
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'avatars.githubusercontent.com',
        pathname: '**',
      },
      {
        protocol: 'https',
        hostname: 'lh3.googleusercontent.com',
        pathname: '**',
      },
      {
        protocol: 'https',
        hostname: '*.cloudinary.com',
        pathname: '**',
      },
    ],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
  experimental: {
    serverActions: {
      bodySizeLimit: '2mb',
    },
    optimizeCss: true,
    scrollRestoration: true,
  },
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-DNS-Prefetch-Control',
            value: 'on'
          },
          {
            key: 'X-XSS-Protection',
            value: '1; mode=block'
          },
          {
            key: 'X-Frame-Options',
            value: 'SAMEORIGIN'
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff'
          },
          {
            key: 'Referrer-Policy',
            value: 'origin-when-cross-origin'
          },
          {
            key: 'Permissions-Policy',
            value: 'camera=(), microphone=(), geolocation=()'
          }
        ],
      },
    ];
  },
  async redirects() {
    return [
      {
        source: '/home',
        destination: '/',
        permanent: true,
      },
      {
        source: '/admin',
        destination: '/admin/dashboard',
        permanent: false,
      },
    ];
  },
  async rewrites() {
    return [
      {
        source: '/api/v1/:path*',
        destination: '/api/:path*',
      },
    ];
  },
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
        net: false,
        tls: false,
      };
    }
    
    config.module.rules.push({
      test: /\.svg$/,
      use: ['@svgr/webpack'],
    });
    
    return config;
  },
};

export default nextConfig;
2.2 Configurazione E-commerce con Bundle Analyzer
typescript
Copia
Scarica
import type { NextConfig } from "next";
import withBundleAnalyzer from '@next/bundle-analyzer';

const bundleAnalyzer = withBundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
});

const nextConfig: NextConfig = {
  reactStrictMode: true,
  images: {
    domains: [
      'cdn.shopify.com',
      'images.unsplash.com',
      'localhost',
    ],
    formats: ['image/avif', 'image/webp'],
  },
  experimental: {
    optimizePackageImports: [
      'lucide-react',
      'date-fns',
      '@radix-ui/react-icons',
    ],
  },
  compiler: {
    removeConsole: process.env.NODE_ENV === 'production' ? {
      exclude: ['error', 'warn'],
    } : false,
  },
  async rewrites() {
    return [
      {
        source: '/sitemap.xml',
        destination: '/api/sitemap',
      },
      {
        source: '/robots.txt',
        destination: '/api/robots',
      },
    ];
  },
  async headers() {
    return [
      {
        source: '/checkout/:path*',
        headers: [
          { key: 'Cache-Control', value: 'no-store, max-age=0' },
        ],
      },
      {
        source: '/api/webhooks/:path*',
        headers: [
          { key: 'Cache-Control', value: 'no-store, max-age=0' },
        ],
      },
    ];
  },
  modularizeImports: {
    'lucide-react': {
      transform: 'lucide-react/dist/esm/icons/{{member}}',
    },
  },
};

export default bundleAnalyzer(nextConfig);
2.3 Configurazione Blog/CMS con MDX
typescript
Copia
Scarica
import type { NextConfig } from "next";
import createMDX from '@next/mdx';
import remarkGfm from 'remark-gfm';
import rehypePrism from 'rehype-prism-plus';

const withMDX = createMDX({
  options: {
    remarkPlugins: [remarkGfm],
    rehypePlugins: [rehypePrism],
  },
});

const nextConfig: NextConfig = {
  pageExtensions: ['ts', 'tsx', 'md', 'mdx'],
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.unsplash.com',
      },
      {
        protocol: 'https',
        hostname: '**.githubusercontent.com',
      },
    ],
  },
  async redirects() {
    return [
      {
        source: '/blog/page/1',
        destination: '/blog',
        permanent: true,
      },
    ];
  },
  async rewrites() {
    return [
      {
        source: '/rss.xml',
        destination: '/api/rss',
      },
      {
        source: '/feed.xml',
        destination: '/api/rss',
      },
    ];
  },
  webpack: (config, options) => {
    config.module.rules.push({
      test: /\.mdx?$/,
      use: [
        options.defaultLoaders.babel,
        {
          loader: '@mdx-js/loader',
          options: {
            providerImportSource: '@mdx-js/react',
          },
        },
      ],
    });
    return config;
  },
};

export default withMDX(nextConfig);
3. TypeScript Configuration
3.1 tsconfig.json ottimale per App Router
json
Copia
Scarica
{
  "compilerOptions": {
    "target": "es5",
    "lib": ["dom", "dom.iterable", "es6"],
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
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@lib/*": ["./src/lib/*"],
      "@hooks/*": ["./src/hooks/*"],
      "@types/*": ["./src/types/*"],
      "@styles/*": ["./src/styles/*"],
      "@public/*": ["./public/*"]
    },
    "forceConsistentCasingInFileNames": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
3.2 Custom Type Definitions (types/next.d.ts)
typescript
Copia
Scarica
import type { NextPage } from 'next';
import type { AppProps } from 'next/app';
import type { ReactElement, ReactNode } from 'react';

// Estendere i tipi di Next.js
declare module 'next' {
  export type NextPageWithLayout<P = {}, IP = P> = NextPage<P, IP> & {
    getLayout?: (page: ReactElement) => ReactNode;
  };
}

declare module 'next/app' {
  type AppPropsWithLayout = AppProps & {
    Component: NextPageWithLayout;
  };
}

// Estensioni globali
declare global {
  interface Window {
    gtag: (...args: any[]) => void;
    dataLayer: Record<string, any>[];
  }
  
  // Per variabili d'ambiente tipizzate
  namespace NodeJS {
    interface ProcessEnv {
      NODE_ENV: 'development' | 'production' | 'test';
      NEXT_PUBLIC_APP_URL: string;
      NEXT_PUBLIC_GOOGLE_ANALYTICS_ID?: string;
      DATABASE_URL: string;
      NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: string;
      STRIPE_SECRET_KEY: string;
    }
  }
}

// Utility types comuni
export type Nullable<T> = T | null;
export type Optional<T> = T | undefined;
export type ValueOf<T> = T[keyof T];
export type DeepPartial<T> = T extends object ? {
  [P in keyof T]?: DeepPartial<T[P]>;
} : T;

// Tipi per API responses
export type ApiResponse<T = any> = {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
};

export type PaginatedResponse<T> = {
  items: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
  hasNext: boolean;
  hasPrev: boolean;
};
3.3 Database Types (types/database.ts)
typescript
Copia
Scarica
import { Prisma } from '@prisma/client';

// Tipi per entità principali
export type UserWithProfile = Prisma.UserGetPayload<{
  include: {
    profile: true;
    accounts: true;
    sessions: true;
  };
}>;

export type ProductWithCategory = Prisma.ProductGetPayload<{
  include: {
    category: true;
    images: true;
    reviews: true;
  };
}>;

export type OrderWithDetails = Prisma.OrderGetPayload<{
  include: {
    items: {
      include: {
        product: true;
      };
    };
    user: {
      select: {
        id: true;
        email: true;
        name: true;
      };
    };
    shippingAddress: true;
    billingAddress: true;
  };
}>;

// Tipi per form
export type UserFormData = {
  email: string;
  name: string;
  image?: string;
  role?: 'USER' | 'ADMIN';
};

export type ProductFormData = {
  name: string;
  description: string;
  price: number;
  categoryId: string;
  stock: number;
  images: string[];
  featured: boolean;
  published: boolean;
};

// Tipi per filtro
export type ProductFilter = {
  category?: string;
  minPrice?: number;
  maxPrice?: number;
  featured?: boolean;
  sortBy?: 'price_asc' | 'price_desc' | 'newest' | 'popular';
  search?: string;
};

// Tipi per paginazione
export type PaginationParams = {
  page?: number;
  limit?: number;
  sortBy?: string;
  sortOrder?: 'asc' | 'desc';
};
4. Tailwind Configuration
4.1 Configurazione Completa (tailwind.config.ts)
typescript
Copia
Scarica
import type { Config } from 'tailwindcss';
import { fontFamily } from 'tailwindcss/defaultTheme';

const config: Config = {
  darkMode: ['class'],
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/features/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
        brand: {
          50: '#f0f9ff',
          100: '#e0f2fe',
          200: '#bae6fd',
          300: '#7dd3fc',
          400: '#38bdf8',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
          800: '#075985',
          900: '#0c4a6e',
          950: '#082f49',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      fontFamily: {
        sans: ['var(--font-sans)', ...fontFamily.sans],
        heading: ['var(--font-heading)', ...fontFamily.sans],
        mono: ['var(--font-mono)', ...fontFamily.mono],
      },
      fontSize: {
        '2xs': ['0.625rem', '0.75rem'],
        '3xs': ['0.5rem', '0.625rem'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
        '128': '32rem',
        '144': '36rem',
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
        'fade-in': 'fade-in 0.5s ease-out',
        'fade-out': 'fade-out 0.5s ease-out',
        'slide-in-right': 'slide-in-right 0.3s ease-out',
        'slide-out-right': 'slide-out-right 0.3s ease-out',
        'pulse-slow': 'pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite',
        'spin-slow': 'spin 2s linear infinite',
        'bounce-slow': 'bounce 2s infinite',
      },
      keyframes: {
        'accordion-down': {
          from: { height: '0' },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: '0' },
        },
        'fade-in': {
          '0%': { opacity: '0', transform: 'translateY(10px)' },
          '100%': { opacity: '1', transform: 'translateY(0)' },
        },
        'fade-out': {
          '0%': { opacity: '1', transform: 'translateY(0)' },
          '100%': { opacity: '0', transform: 'translateY(10px)' },
        },
        'slide-in-right': {
          '0%': { transform: 'translateX(100%)' },
          '100%': { transform: 'translateX(0)' },
        },
        'slide-out-right': {
          '0%': { transform: 'translateX(0)' },
          '100%': { transform: 'translateX(100%)' },
        },
      },
      screens: {
        'xs': '475px',
        '3xl': '1600px',
        '4xl': '1920px',
      },
      boxShadow: {
        'soft': '0 2px 15px -3px rgba(0, 0, 0, 0.07), 0 10px 20px -2px rgba(0, 0, 0, 0.04)',
        'hard': '0 10px 40px rgba(0, 0, 0, 0.1)',
        'inner-lg': 'inset 0 2px 4px 0 rgba(0, 0, 0, 0.06)',
        'glow': '0 0 20px rgba(59, 130, 246, 0.5)',
      },
      backdropBlur: {
        xs: '2px',
      },
      backgroundImage: {
        'gradient-radial': 'radial-gradient(var(--tw-gradient-stops))',
        'gradient-conic': 'conic-gradient(from 180deg at 50% 50%, var(--tw-gradient-stops))',
        'gradient-shimmer': 'linear-gradient(90deg, transparent 0%, rgba(255,255,255,0.1) 50%, transparent 100%)',
      },
    },
  },
  plugins: [
    require('tailwindcss-animate'),
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
    require('@tailwindcss/aspect-ratio'),
  ],
};

export default config;
4.2 CSS Variables Setup (app/globals.css)
css
Copia
Scarica
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 221.2 83.2% 53.3%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 217.2 91.2% 59.8%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 224.3 76.3% 48%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
    font-feature-settings: "rlig" 1, "calt" 1;
  }
}

@layer components {
  .container-custom {
    @apply container mx-auto px-4 sm:px-6 lg:px-8;
  }
  
  .btn-primary {
    @apply inline-flex items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-primary text-primary-foreground hover:bg-primary/90 h-10 px-4 py-2;
  }
  
  .btn-secondary {
    @apply inline-flex items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 bg-secondary text-secondary-foreground hover:bg-secondary/80 h-10 px-4 py-2;
  }
  
  .card {
    @apply rounded-lg border bg-card text-card-foreground shadow-sm;
  }
  
  .card-header {
    @apply flex flex-col space-y-1.5 p-6;
  }
  
  .card-title {
    @apply text-2xl font-semibold leading-none tracking-tight;
  }
  
  .card-description {
    @apply text-sm text-muted-foreground;
  }
  
  .card-content {
    @apply p-6 pt-0;
  }
  
  .card-footer {
    @apply flex items-center p-6 pt-0;
  }
}

@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
  
  .hide-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
  }
  
  .hide-scrollbar::-webkit-scrollbar {
    display: none;
  }
  
  .gradient-text {
    @apply bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent;
  }
  
  .glass-effect {
    @apply backdrop-blur-md bg-white/10 dark:bg-black/10 border border-white/20 dark:border-black/20;
  }
}
5. ESLint & Prettier
5.1 ESLint Configuration (.eslintrc.json)
json
Copia
Scarica
{
  "extends": [
    "next/core-web-vitals",
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
    "plugin:jsx-a11y/recommended",
    "prettier"
  ],
  "plugins": [
    "@typescript-eslint",