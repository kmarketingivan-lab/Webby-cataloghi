# CATALOGO-BOILERPLATE-STRUTTURA-v1

> **Dominio**: Struttura file e codice base Next.js 14
> **Stack**: Next.js 14 App Router, TypeScript, Prisma, tRPC
> **Versione**: 1.0
> **Data**: 2026-01-29

---

## INDICE

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

## 1. PANORAMICA

### 1.1 Obiettivo
Struttura completa di file per un progetto Next.js 14 con App Router, pronta per lo sviluppo.

### 1.2 Architettura Directory

```
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
```

### 1.3 Convenzioni Naming

| Tipo | Convenzione | Esempio |
|------|-------------|---------|
| Componenti | PascalCase | `Button.tsx` |
| Hooks | camelCase con `use` | `useDebounce.ts` |
| Utils | camelCase | `formatDate.ts` |
| Types | PascalCase | `ApiResponse.ts` |
| Constants | UPPER_SNAKE | `MAX_PAGE_SIZE` |
| Files CSS | kebab-case | `globals.css` |

---

## 2. APP LAYER

### 2.1 Mappa File App Router

| File | Path | Descrizione |
|------|------|-------------|
| Root Layout | `src/app/layout.tsx` | HTML wrapper, providers |
| Homepage | `src/app/page.tsx` | Landing page |
| Loading | `src/app/loading.tsx` | Suspense fallback |
| Error | `src/app/error.tsx` | Error boundary |
| Not Found | `src/app/not-found.tsx` | 404 page |
| Globals | `src/app/globals.css` | Tailwind base |
| tRPC Route | `src/app/api/trpc/[trpc]/route.ts` | API handler |

### 2.2 Root Layout

**File**: `src/app/layout.tsx`

| Responsabilità | Dettaglio |
|----------------|-----------|
| HTML structure | `<html>`, `<head>`, `<body>` |
| Font loading | Google Fonts (Inter) |
| Metadata | SEO base |
| Providers | Context wrapper |
| Analytics | Tracking component |
| Toast | Notification container |

```typescript
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
```

### 2.3 Homepage

**File**: `src/app/page.tsx`

```typescript
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
```

### 2.4 Loading State

**File**: `src/app/loading.tsx`

```typescript
import type { ReactNode } from 'react';

export default function Loading() {
  return (
    <div className="flex justify-center items-center h-screen">
      <div className="spinner-border animate-spin inline-block w-8 h-8 border-4 rounded-full text-gray-200" />
    </div>
  );
}
```

### 2.5 Error Boundary

**File**: `src/app/error.tsx`

| Metodo | Scopo |
|--------|-------|
| `getDerivedStateFromError` | Cattura errore |
| `componentDidCatch` | Log errore |
| `render` | UI fallback |

```typescript
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
```

### 2.6 Not Found Page

**File**: `src/app/not-found.tsx`

```typescript
import type { ReactNode } from 'react';

export default function NotFoundPage() {
  return (
    <div className="flex justify-center items-center h-screen">
      <h1 className="text-3xl font-bold">Pagina non trovata!</h1>
    </div>
  );
}
```

### 2.7 Global Styles

**File**: `src/app/globals.css`

| Layer | Contenuto |
|-------|-----------|
| `@layer base` | Stili base globali |
| `@layer components` | Classi componenti |
| `@layer utilities` | Utility custom |
| `:root` | CSS variables |

```css
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
```

---

## 3. LIB LAYER

### 3.1 Mappa File Lib

| File | Path | Descrizione |
|------|------|-------------|
| Utils | `src/lib/utils.ts` | Helper functions |
| Constants | `src/lib/constants.ts` | App constants |
| Prisma | `src/lib/prisma.ts` | DB client |
| Env | `src/lib/env.ts` | Env validation |

### 3.2 Utils

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

```typescript
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
```

### 3.3 Constants

**File**: `src/lib/constants.ts`

| Costante | Tipo | Descrizione |
|----------|------|-------------|
| `APP_NAME` | string | Nome applicazione |
| `APP_DESCRIPTION` | string | Descrizione app |
| `APP_URL` | string | URL base |
| `PAGINATION` | object | Config paginazione |
| `ROUTES` | object | Route principali |

```typescript
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
```

### 3.4 Prisma Client

**File**: `src/lib/prisma.ts`

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

prisma.$use(async (params, next) => {
  const result = await next(params);
  console.log('Query:', params, result);
  return result;
});

export default prisma;
```

### 3.5 Env Validation

**File**: `src/lib/env.ts`

| Variabile | Schema | Required |
|-----------|--------|----------|
| `DATABASE_URL` | `z.string().url()` | Yes |
| `NEXTAUTH_URL` | `z.string().url()` | Yes |
| `NEXTAUTH_SECRET` | `z.string().min(32)` | Yes |

```typescript
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
});

export const env = envSchema.parse(process.env);
```

---

## 4. COMPONENTS LAYER

### 4.1 Mappa Componenti

| File | Path | Descrizione |
|------|------|-------------|
| Providers | `src/components/providers.tsx` | Context wrapper |
| UI Index | `src/components/ui/index.ts` | Barrel export |
| Button | `src/components/ui/button.tsx` | Button component |
| Input | `src/components/ui/input.tsx` | Input component |
| Card | `src/components/ui/card.tsx` | Card components |
| Toast | `src/components/ui/toast.tsx` | Toast system |

### 4.2 Providers

**File**: `src/components/providers.tsx`

| Provider | Package | Scopo |
|----------|---------|-------|
| `QueryClientProvider` | react-query | Data fetching |
| `SessionProvider` | next-auth | Auth state |
| `ThemeProvider` | next-themes | Dark mode |
| `ToastProvider` | react-toastify | Notifications |

```typescript
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
```

### 4.3 Button Component

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

```typescript
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
```

### 4.4 Input Component

**File**: `src/components/ui/input.tsx`

```typescript
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
```

### 4.5 Card Components

**File**: `src/components/ui/card.tsx`

| Componente | Descrizione |
|------------|-------------|
| `Card` | Container principale |
| `CardHeader` | Header area |
| `CardTitle` | Titolo card |
| `CardDescription` | Sottotitolo |
| `CardContent` | Contenuto principale |
| `CardFooter` | Footer area |

```typescript
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
```

### 4.6 UI Barrel Export

**File**: `src/components/ui/index.ts`

```typescript
export * from './button';
export * from './input';
export * from './card';
export * from './toast';
```

---

## 5. TYPES LAYER

### 5.1 Types Principali

**File**: `src/types/index.ts`

| Type | Descrizione |
|------|-------------|
| `ApiResponse<T>` | Response wrapper API |
| `PaginatedResponse<T>` | Response paginata |
| `ApiError` | Error structure |
| `AsyncFunction<T>` | Async function type |

```typescript
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
```

---

## 6. HOOKS LAYER

### 6.1 Mappa Hooks

| Hook | File | Descrizione |
|------|------|-------------|
| `useDebounce` | `use-debounce.ts` | Debounce values |
| `useLocalStorage` | `use-local-storage.ts` | Persist to localStorage |
| `useMediaQuery` | `use-media-query.ts` | Responsive queries |

### 6.2 useDebounce

**File**: `src/hooks/use-debounce.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `value` | `T` | Valore da debounceare |
| `delay` | `number` | Delay in ms |
| **Return** | `T` | Valore debouncato |

```typescript
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
```

### 6.3 useLocalStorage

**File**: `src/hooks/use-local-storage.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `key` | `string` | Chiave storage |
| `initialValue` | `T` | Valore default |
| **Return** | `[T, SetState<T>]` | Value e setter |

```typescript
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
```

### 6.4 useMediaQuery

**File**: `src/hooks/use-media-query.ts`

| Parametro | Tipo | Descrizione |
|-----------|------|-------------|
| `query` | `string` | Media query CSS |
| **Return** | `boolean` | Match result |

```typescript
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
```

---

## 7. SERVER LAYER

### 7.1 tRPC Setup

| File | Path | Descrizione |
|------|------|-------------|
| Main Router | `src/server/trpc/trpc.ts` | tRPC config |
| App Router | `src/server/trpc/routers/_app.ts` | Root router |
| API Route | `src/app/api/trpc/[trpc]/route.ts` | Next.js handler |

### 7.2 tRPC Main

**File**: `src/server/trpc/trpc.ts`

```typescript
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
```

### 7.3 App Router

**File**: `src/server/trpc/routers/_app.ts`

```typescript
import * as trpc from '@trpc/server';
import { z } from 'zod';

export const appRouter = trpc.router()
  .query('getUser', {
    input: z.string(),
    async resolve({ input }) {
      return { id: 1, name: 'John Doe', email: 'john.doe@example.com' };
    },
  });
```

### 7.4 API Route Handler

**File**: `src/app/api/trpc/[trpc]/route.ts`

```typescript
import * as trpcNext from '@trpc/server/adapters/next';
import { appRouter } from '../../../server/trpc/trpc';

export default trpcNext.createNextApiHandler({
  router: appRouter,
  createContext: () => null,
});
```

---

## 8. DATABASE LAYER

### 8.1 Schema Prisma

**File**: `prisma/schema.prisma`

| Modello | Campi Principali | Relazioni |
|---------|------------------|-----------|
| `User` | id, name, email, password | Session[], Account[] |
| `Session` | id, userId, expires | User |
| `Account` | id, userId, provider, accessToken | User |
| `VerificationToken` | id, identifier, token, expires | - |

```prisma
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
```

---

## CHECKLIST SETUP

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
