# CATALOGO-BOILERPLATE-STRUTTURA

Espansione CATALOGO-BOILERPLATE-STRUTTURA Next.js 14 App Router
§ FOLDER STRUCTURE COMPLETA
text
src/
├── app/                          # App Router
│   ├── (auth)/                   # Route group: Auth pages
│   │   ├── login/
│   │   │   ├── page.tsx
│   │   │   └── layout.tsx
│   │   └── register/
│   │       ├── page.tsx
│   │       └── layout.tsx
│   ├── (dashboard)/              # Route group: Dashboard area
│   │   ├── dashboard/
│   │   │   ├── analytics/
│   │   │   │   └── page.tsx
│   │   │   ├── settings/
│   │   │   │   └── page.tsx
│   │   │   └── layout.tsx
│   │   └── layout.tsx            # Dashboard layout wrapper
│   ├── (marketing)/              # Route group: Marketing pages
│   │   ├── page.tsx              # Homepage
│   │   ├── pricing/
│   │   │   └── page.tsx
│   │   ├── blog/
│   │   │   ├── [slug]/
│   │   │   │   └── page.tsx
│   │   │   └── page.tsx
│   │   └── layout.tsx
│   ├── api/                      # API Routes (vedi sezione dedicata)
│   ├── @modal/                   # Parallel route: Modal
│   │   └── default.tsx
│   ├── @sidebar/                 # Parallel route: Sidebar
│   │   └── default.tsx
│   ├── layout.tsx                # Root layout
│   └── template.tsx              # Root template
├── components/                   # Shared components
│   ├── ui/                       # Primitives/UI components (shadcn/ui style)
│   │   ├── button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   └── index.ts         # Barrel export
│   │   ├── dialog/
│   │   ├── input/
│   │   └── index.ts              # Barrel export per tutti gli UI
│   ├── forms/                    # Form-specific components
│   │   ├── FormField.tsx
│   │   ├── FormSelect.tsx
│   │   ├── FormDatePicker.tsx
│   │   └── index.ts
│   ├── layouts/                  # Layout components
│   │   ├── Header/
│   │   │   ├── Header.tsx
│   │   │   └── index.ts
│   │   ├── Footer/
│   │   └── Sidebar/
│   ├── _components/              # Private components (non esportati)
│   │   ├── InternalComponent.tsx
│   │   └── AnotherInternal/
│   └── features/                 # Feature-based components
│       ├── dashboard/
│       │   ├── StatsCard.tsx
│       │   ├── RecentActivity.tsx
│       │   └── index.ts
│       ├── blog/
│       │   ├── BlogCard.tsx
│       │   ├── BlogList.tsx
│       │   └── index.ts
│       └── auth/
│           ├── LoginForm.tsx
│           ├── RegisterForm.tsx
│           └── index.ts
├── hooks/                        # Custom hooks
│   ├── use-debounce.ts
│   ├── use-local-storage.ts
│   ├── use-media-query.ts
│   └── index.ts                  # Barrel export
├── lib/                          # Utilities & libraries (vedi sezione dedicata)
├── services/                     # API/services layer
│   ├── api-client.ts
│   ├── user-service.ts
│   ├── product-service.ts
│   └── index.ts
├── store/                        # State management (Zustand/Redux)
│   ├── slices/
│   │   ├── auth-slice.ts
│   │   └── ui-slice.ts
│   └── index.ts
├── types/                        # TypeScript definitions (vedi sezione dedicata)
├── utils/                        # Utility functions
│   ├── formatters.ts
│   ├── validators.ts
│   └── index.ts
├── styles/                       # Global/styles
│   ├── globals.css
│   ├── tokens.css
│   └── components/
│       ├── button.css
│       └── input.css
├── config/                       # Configuration files
│   ├── site-config.ts
│   ├── feature-flags.ts
│   └── env.ts
└── public/                       # Static assets
    ├── images/
    ├── fonts/
    └── icons/
Barrel Exports Pattern
typescript
// components/ui/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Dialog } from './Dialog';

// Import usage:
import { Button, Input } from '@/components/ui';
Colocazione File per Componenti
text
components/
└── feature-name/
    ├── ComponentName.tsx         # Componente principale
    ├── ComponentName.test.tsx    # Test (Jest/Vitest)
    ├── ComponentName.stories.tsx # Storybook
    ├── ComponentName.types.ts    # Tipi specifici del componente
    ├── ComponentName.module.css  # Stili modulari (se necessario)
    └── index.ts                  # Barrel export
§ ROUTE GROUPS PATTERNS
(auth) Group - Layout condiviso per autenticazione
typescript
// app/(auth)/layout.tsx
export default function AuthLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen bg-gray-50 flex flex-col justify-center py-12 sm:px-6 lg:px-8">
      <div className="sm:mx-auto sm:w-full sm:max-w-md">
        <Logo />
      </div>
      <div className="mt-8 sm:mx-auto sm:w-full sm:max-w-md">
        <div className="bg-white py-8 px-4 shadow sm:rounded-lg sm:px-10">
          {children}
        </div>
      </div>
    </div>
  );
}
(dashboard) Group - Area protetta con autenticazione
typescript
// app/(dashboard)/layout.tsx
import { auth } from '@/lib/auth';
import { redirect } from 'next/navigation';

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth();
  
  if (!session) {
    redirect('/login');
  }
  
  return (
    <div className="flex min-h-screen">
      <DashboardSidebar />
      <main className="flex-1 p-6">
        <DashboardHeader />
        <div className="mt-6">
          {children}
        </div>
      </main>
    </div>
  );
}
(marketing) Group - Layout per landing pages
typescript
// app/(marketing)/layout.tsx
import MarketingHeader from '@/components/layouts/MarketingHeader';
import MarketingFooter from '@/components/layouts/MarketingFooter';

export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="min-h-screen flex flex-col">
      <MarketingHeader />
      <main className="flex-1">
        {children}
      </main>
      <MarketingFooter />
    </div>
  );
}
Nested Layouts Strategy
typescript
// app/(dashboard)/dashboard/layout.tsx - Layout annidato
export default function DashboardSubLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="space-y-6">
      <div className="border-b pb-4">
        <h1 className="text-3xl font-bold">Dashboard</h1>
        <p className="text-gray-600">Manage your account and settings</p>
      </div>
      <div className="grid grid-cols-1 md:grid-cols-4 gap-6">
        <aside className="md:col-span-1">
          <DashboardNav />
        </aside>
        <div className="md:col-span-3">
          {children}
        </div>
      </div>
    </div>
  );
}
§ PARALLEL ROUTES
@modal Slot per modali route-based
typescript
// app/@modal/default.tsx
export default function DefaultModal() {
  return null; // Renderizza nulla quando non c'è modal attivo
}

// app/@modal/(.)photos/[id]/page.tsx - Intercepted route
import PhotoModal from '@/components/modals/PhotoModal';

export default function InterceptedPhotoModal({
  params,
}: {
  params: { id: string };
}) {
  return <PhotoModal photoId={params.id} />;
}

// app/layout.tsx - Integrazione slot
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        {modal}
        <div id="modal-root" />
      </body>
    </html>
  );
}
@sidebar Slot per sidebar condizionali
typescript
// app/@sidebar/default.tsx
import DefaultSidebar from '@/components/layouts/DefaultSidebar';

export default function DefaultSidebarSlot() {
  return <DefaultSidebar />;
}

// app/dashboard/@sidebar/page.tsx - Sidebar specifica per dashboard
import DashboardSidebar from '@/components/layouts/DashboardSidebar';

export default function DashboardSidebarSlot() {
  return <DashboardSidebar />;
}

// app/layout.tsx con slot sidebar
export default function RootLayout({
  children,
  sidebar,
}: {
  children: React.ReactNode;
  sidebar: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className="flex">
        {sidebar}
        <main className="flex-1">{children}</main>
      </body>
    </html>
  );
}
Loading e Error per Slot
typescript
// app/@modal/loading.tsx
export default function ModalLoading() {
  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-white"></div>
    </div>
  );
}

// app/@modal/error.tsx
'use client';

export default function ModalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  return (
    <div className="fixed inset-0 bg-black/50 flex items-center justify-center">
      <div className="bg-white p-6 rounded-lg">
        <h2 className="text-lg font-bold mb-2">Something went wrong!</h2>
        <p className="text-gray-600 mb-4">{error.message}</p>
        <button
          onClick={() => reset()}
          className="px-4 py-2 bg-blue-600 text-white rounded"
        >
          Try again
        </button>
      </div>
    </div>
  );
}
§ INTERCEPTING ROUTES
Pattern per Intercepting Routes
typescript
// app/photos/[id]/page.tsx - Route normale (full page)
import PhotoDetail from '@/components/photos/PhotoDetail';

export default function PhotoPage({ params }: { params: { id: string } }) {
  return (
    <div className="container mx-auto py-8">
      <PhotoDetail photoId={params.id} />
    </div>
  );
}

// app/@modal/(.)photos/[id]/page.tsx - Intercepted route (stesso livello)
import PhotoModal from '@/components/photos/PhotoModal';

export default function InterceptedPhotoPage({
  params,
}: {
  params: { id: string };
}) {
  return <PhotoModal photoId={params.id} />;
}

// app/feed/(..)photos/[id]/page.tsx - Intercepted da parent level
// app/feed/featured/(...)photos/[id]/page.tsx - Intercepted da root level
Photo Gallery Modal Pattern Completo
typescript
// components/photos/PhotoModal.tsx
'use client';

import { useRouter } from 'next/navigation';
import { useEffect, useRef } from 'react';

export default function PhotoModal({ photoId }: { photoId: string }) {
  const router = useRouter();
  const dialogRef = useRef<HTMLDialogElement>(null);

  useEffect(() => {
    if (!dialogRef.current?.open) {
      dialogRef.current?.showModal();
    }

    const handleEscape = (e: KeyboardEvent) => {
      if (e.key === 'Escape') {
        router.back();
      }
    };

    document.addEventListener('keydown', handleEscape);
    return () => document.removeEventListener('keydown', handleEscape);
  }, [router]);

  const handleClose = () => {
    router.back();
  };

  return (
    <dialog
      ref={dialogRef}
      className="fixed inset-0 z-50 bg-black/50 backdrop-blur-sm"
      onClick={handleClose}
    >
      <div
        className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2"
        onClick={(e) => e.stopPropagation()}
      >
        {/* Contenuto del modal */}
        <div className="bg-white rounded-lg p-6 max-w-2xl">
          <h2 className="text-2xl font-bold mb-4">Photo {photoId}</h2>
          <img
            src={`/api/photos/${photoId}`}
            alt={`Photo ${photoId}`}
            className="w-full h-auto rounded"
          />
          <button
            onClick={handleClose}
            className="mt-4 px-4 py-2 bg-gray-200 rounded"
          >
            Close
          </button>
        </div>
      </div>
    </dialog>
  );
}
§ API ROUTES ORGANIZATION
Route Handlers Structure
text
app/
├── api/
│   ├── v1/                       # Versioning API
│   │   ├── auth/
│   │   │   ├── login/
│   │   │   │   └── route.ts
│   │   │   ├── register/
│   │   │   │   └── route.ts
│   │   │   └── logout/
│   │   │       └── route.ts
│   │   ├── users/
│   │   │   ├── route.ts          # GET, POST
│   │   │   ├── [id]/
│   │   │   │   └── route.ts      # GET, PUT, DELETE
│   │   │   └── [id]/profile/
│   │   │       └── route.ts
│   │   ├── products/
│   │   │   └── route.ts
│   │   └── health/
│   │       └── route.ts
│   ├── middleware.ts             # Middleware per tutte le API
│   └── route.ts                  # API root (redirect a v1 o docs)
Middleware per API
typescript
// app/api/middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { rateLimiter } from '@/lib/rate-limiter';
import { validateApiKey } from '@/lib/auth/api-keys';

export async function apiMiddleware(request: NextRequest) {
  // Rate limiting
  const ip = request.ip ?? request.headers.get('x-forwarded-for') ?? '127.0.0.1';
  const isRateLimited = await rateLimiter.isLimited(ip);
  
  if (isRateLimited) {
    return NextResponse.json(
      { error: 'Too many requests' },
      { status: 429 }
    );
  }

  // API Key validation (per routes non pubbliche)
  if (!request.headers.get('x-api-key') && !request.nextUrl.pathname.includes('/public/')) {
    const apiKey = request.headers.get('x-api-key');
    if (!apiKey || !(await validateApiKey(apiKey))) {
      return NextResponse.json(
        { error: 'Invalid API key' },
        { status: 401 }
      );
    }
  }

  // CORS headers
  const response = NextResponse.next();
  response.headers.set('Access-Control-Allow-Origin', '*');
  response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization, x-api-key');

  return response;
}

// Utilizzo in route handlers
export const config = {
  matcher: '/api/:path*',
};
Route Handler Esempio con Versioning
typescript
// app/api/v1/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createUser, getUsers } from '@/lib/db/users';
import { userSchema } from '@/lib/validations/user';
import { withApiErrorHandler } from '@/lib/api/error-handler';

export const GET = withApiErrorHandler(async (request: NextRequest) => {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '10');
  
  const users = await getUsers({ page, limit });
  
  return NextResponse.json({
    success: true,
    data: users,
    pagination: {
      page,
      limit,
      total: users.length
    }
  });
});

export const POST = withApiErrorHandler(async (request: NextRequest) => {
  const body = await request.json();
  const validatedData = userSchema.parse(body);
  
  const user = await createUser(validatedData);
  
  return NextResponse.json(
    {
      success: true,
      data: user,
      message: 'User created successfully'
    },
    { status: 201 }
  );
});
Error Handling Centralizzato
typescript
// lib/api/error-handler.ts
import { NextRequest, NextResponse } from 'next/server';
import { ZodError } from 'zod';
import { AppError } from '@/lib/errors';

export function withApiErrorHandler(
  handler: (request: NextRequest) => Promise<NextResponse>
) {
  return async (request: NextRequest) => {
    try {
      return await handler(request);
    } catch (error) {
      console.error('API Error:', error);
      
      if (error instanceof ZodError) {
        return NextResponse.json(
          {
            success: false,
            error: 'Validation failed',
            details: error.errors
          },
          { status: 400 }
        );
      }
      
      if (error instanceof AppError) {
        return NextResponse.json(
          {
            success: false,
            error: error.message,
            code: error.code
          },
          { status: error.statusCode }
        );
      }
      
      // Errori sconosciuti
      return NextResponse.json(
        {
          success: false,
          error: 'Internal server error',
          requestId: request.headers.get('x-request-id')
        },
        { status: 500 }
      );
    }
  };
}

// lib/errors.ts - Errori personalizzati
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500
  ) {
    super(message);
    this.name = 'AppError';
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 'NOT_FOUND', 404);
  }
}

export class ValidationError extends AppError {
  constructor(details: any) {
    super('Validation failed', 'VALIDATION_ERROR', 400);
  }
}
§ SHARED TYPES ORGANIZATION
/types Folder Structure
text
types/
├── api/                          # API-related types
│   ├── requests/
│   │   ├── auth.ts
│   │   ├── user.ts
│   │   └── product.ts
│   ├── responses/
│   │   ├── auth.ts
│   │   ├── user.ts
│   │   └── common.ts
│   └── index.ts
├── domain/                       # Domain entities
│   ├── user.ts
│   ├── product.ts
│   ├── order.ts
│   └── index.ts
├── components/                   # Component props types
│   ├── ui.ts
│   ├── forms.ts
│   └── layout.ts
├── lib/                          # Library types
│   ├── auth.ts
│   └── database.ts
├── utils/                        # Utility types
│   ├── helpers.ts
│   └── guards.ts
├── env.d.ts                      # Environment variables
└── index.ts                      # Barrel exports
Discriminated Unions
typescript
// types/domain/user.ts
export type UserRole = 'admin' | 'user' | 'moderator';

export interface BaseUser {
  id: string;
  email: string;
  createdAt: Date;
}

export interface AdminUser extends BaseUser {
  role: 'admin';
  permissions: string[];
  managedTeams: string[];
}

export interface RegularUser extends BaseUser {
  role: 'user' | 'moderator';
  profile: UserProfile;
  settings: UserSettings;
}

export type User = AdminUser | RegularUser;

// Type guard
export function isAdminUser(user: User): user is AdminUser {
  return user.role === 'admin';
}

// Usage example
function handleUserAction(user: User) {
  if (isAdminUser(user)) {
    // TypeScript sa che user è AdminUser qui
    console.log(user.managedTeams);
  } else {
    // TypeScript sa che user è RegularUser qui
    console.log(user.profile);
  }
}
Zod Schemas Location
typescript
// lib/validations/
├── schemas/
│   ├── user.ts
│   ├── product.ts
│   ├── auth.ts
│   └── common.ts
├── transformers/                 # Schema transformers
│   ├── date-transformer.ts
│   └── password-transformer.ts
└── index.ts

// lib/validations/user.ts
import { z } from 'zod';
import { passwordSchema } from './common';

export const userSchema = z.object({
  id: z.string().uuid().optional(),
  email: z.string().email(),
  username: z.string().min(3).max(50),
  password: passwordSchema,
  role: z.enum(['admin', 'user', 'moderator']).default('user'),
  profile: z.object({
    firstName: z.string().min(1).max(50),
    lastName: z.string().min(1).max(50),
    avatar: z.string().url().optional(),
  }).optional(),
  settings: z.object({
    emailNotifications: z.boolean().default(true),
    theme: z.enum(['light', 'dark', 'system']).default('system'),
  }).optional(),
});

export type UserInput = z.input<typeof userSchema>;
export type UserOutput = z.output<typeof userSchema>;
export type User = z.infer<typeof userSchema>;

// API response types correlati
// types/api/responses/user.ts
import { User } from '@/types/domain/user';

export interface UserResponse {
  success: boolean;
  data: User;
  meta?: {
    requestId: string;
    timestamp: Date;
  };
}

export interface UsersListResponse {
  success: boolean;
  data: User[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    pages: number;
  };
}
§ LIB FOLDER PATTERNS
/lib/db - Database Utilities
typescript
// lib/db/
├── client.ts                     # DB client initialization
├── schema/                       # Database schemas (Drizzle, Prisma)
│   ├── users.ts
│   ├── products.ts
│   └── index.ts
├── queries/                      # Query functions
│   ├── user-queries.ts
│   ├── product-queries.ts
│   └── index.ts
├── migrations/                   # Migration scripts
└── index.ts

// lib/db/client.ts
import { drizzle } from 'drizzle-orm/vercel-postgres';
import { sql } from '@vercel/postgres';
import * as schema from './schema';

export const db = drizzle(sql, { schema });

// lib/db/queries/user-queries.ts
import { db } from '../client';
import { users } from '../schema/users';
import { eq } from 'drizzle-orm';

export async function getUserById(id: string) {
  const [user] = await db
    .select()
    .from(users)
    .where(eq(users.id, id))
    .limit(1);
  
  return user;
}

export async function createUser(data: typeof users.$inferInsert) {
  const [user] = await db
    .insert(users)
    .values(data)
    .returning();
  
  return user;
}
/lib/auth - Authentication Utilities
typescript
// lib/auth/
├── index.ts                      # Main auth exports
├── config.ts                     # Auth configuration
├── providers/                    # Auth providers
│   ├── credentials.ts
│   ├── google.ts
│   └── github.ts
├── session.ts                    # Session management
├── tokens.ts                     // Token utilities (JWT, etc.)
└── middleware.ts                 // Auth middleware

// lib/auth/index.ts
import { Auth.js } from '@auth/core';
import Google from '@auth/core/providers/google';
import Credentials from '@auth/core/providers/credentials';
import { DrizzleAdapter } from '@auth/drizzle-adapter';
import { db } from '@/lib/db';

export const authConfig = {
  adapter: DrizzleAdapter(db),
  providers: [
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
    }),
    Credentials({
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        // Implementazione custom
      },
    }),
  ],
  session: {
    strategy: 'jwt',
  },
  pages: {
    signIn: '/login',
    error: '/auth/error',
  },
  callbacks: {
    async session({ session, token }) {
      if (token.sub) {
        session.user.id = token.sub;
      }
      return session;
    },
  },
};
/lib/email - Email Utilities
typescript
// lib/email/
├── client.ts                     // Email client (Resend, SendGrid, etc.)
├── templates/                    // Email templates
│   ├── welcome.tsx
│   ├── reset-password.tsx
│   └── newsletter.tsx
├── queue.ts                      // Email queue (BullMQ, etc.)
└── index.ts

// lib/email/client.ts
import { Resend } from 'resend';

export const resend = new Resend(process.env.RESEND_API_KEY);

export async function sendWelcomeEmail(email: string, name: string) {
  const { data, error } = await resend.emails.send({
    from: 'Acme <onboarding@acme.com>',
    to: [email],
    subject: 'Welcome to Acme!',
    react: WelcomeEmail({ name }),
  });

  if (error) {
    throw new Error(`Failed to send email: ${error.message}`);
  }

  return data;
}

// lib/email/templates/welcome.tsx
import * as React from 'react';

interface WelcomeEmailProps {
  name: string;
}

export function WelcomeEmail({ name }: WelcomeEmailProps) {
  return (
    <div>
      <h1>Welcome, {name}!</h1>
      <p>Thanks for joining our platform.</p>
    </div>
  );
}
/lib/utils - Generic Utilities
typescript
// lib/utils/
├── index.ts                      // Barrel exports
├── formatters.ts                 // Data formatting
├── validators.ts                 // Validation helpers
├── strings.ts                    // String manipulation
├── arrays.ts                     // Array utilities
├── objects.ts                    // Object utilities
├── dates.ts                      // Date utilities
└── async.ts                      // Async utilities

// lib/utils/formatters.ts
export function formatCurrency(amount: number, currency = 'EUR'): string {
  return new Intl.NumberFormat('it-IT', {
    style: 'currency',
    currency,
  }).format(amount);
}

export function formatDate(date: Date | string, locale = 'it-IT'): string {
  const d = typeof date === 'string' ? new Date(date) : date;
  return new Intl.DateTimeFormat(locale, {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(d);
}

// lib/utils/async.ts
export function withTimeout<T>(
  promise: Promise<T>,
  timeout: number,
  errorMessage = 'Operation timed out'
): Promise<T> {
  return Promise.race([
    promise,
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error(errorMessage)), timeout)
    ),
  ]);
}

export async function retry<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries === 0) throw error;
    await new Promise(resolve => setTimeout(resolve, delay));
    return retry(fn, retries - 1, delay * 2);
  }
}
/lib/validations - Zod Schemas
typescript
// lib/validations/
├── index.ts                      // Barrel exports
├── schemas/                      // Main schemas
│   ├── user.ts
│   ├── product.ts
│   ├── auth.ts
│   └── common.ts
├── transformers/                 // Schema transformers
│   ├── date.ts
│   └── password.ts
└── utils.ts                      // Validation utilities

// lib/validations/common.ts
import { z } from 'zod';

export const emailSchema = z
  .string()
  .email('Invalid email address')
  .min(1, 'Email is required');

export const passwordSchema = z
  .string()
  .min(8, 'Password must be at least 8 characters')
  .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
  .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
  .regex(/\d/, 'Password must contain at least one number');

// lib/validations/transformers/date.ts
import { z } from 'zod';

export const dateTransformer = z
  .string()
  .or(z.date())
  .transform((val) => (typeof val === 'string' ? new Date(val) : val))
  .refine((val) => !isNaN(val.getTime()), {
    message: 'Invalid date format',
  });
§ COMPONENTS ORGANIZATION
/components/ui - Primitives (shadcn/ui style)
typescript
// components/ui/
├── button/
│   ├── Button.tsx
│   ├── Button.test.tsx
│   ├── Button.stories.tsx
│   └── index.ts
├── input/
│   ├── Input.tsx
│   ├── Input.test.tsx
│   └── index.ts
├── dialog/
│   ├── Dialog.tsx
│   ├── DialogContent.tsx
│   ├── DialogHeader.tsx
│   └── index.ts
├── dropdown-menu/
├── select/
├── table/
└── index.ts                     // Barrel export di tutti i componenti UI

// components/ui/Button.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';
import { ButtonProps } from '@/types/components/ui';

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant = 'default', size = 'default', ...props }, ref) => {
    return (
      <button
        className={cn(
          'inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50',
          {
            'bg-primary text-primary-foreground hover:bg-primary/90': variant === 'default',
            'bg-destructive text-destructive-foreground hover:bg-destructive/90': variant === 'destructive',
            'border border-input bg-background hover:bg-accent hover:text-accent-foreground': variant === 'outline',
          },
          {
            'h-10 px-4 py-2': size === 'default',
            'h-9 rounded-md px-3': size === 'sm',
            'h-11 rounded-md px-8': size === 'lg',
          },
          className
        )}
        ref={ref}
        {...props}
      />
    );
  }
);
Button.displayName = 'Button';

export { Button };
/components/forms - Form Components
typescript
// components/forms/
├── Form.tsx                      // Form wrapper con react-hook-form
├── FormField.tsx                 // Field con label, error, etc.
├── FormInput.tsx                 // Input field
├── FormSelect.tsx                // Select field
├── FormTextarea.tsx              // Textarea field
├── FormCheckbox.tsx              // Checkbox field
├── FormDatePicker.tsx            // Date picker field
├── FormRadioGroup.tsx            // Radio group field
└── index.ts

// components/forms/Form.tsx
'use client';

import * as React from 'react';
import { useForm, FormProvider } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

interface FormProps<T extends z.ZodType<any, any>> {
  schema: T;
  defaultValues?: z.infer<T>;
  onSubmit: (data: z.infer<T>) => void | Promise<void>;
  children: React.ReactNode;
  className?: string;
}

export function Form<T extends z.ZodType<any, any>>({
  schema,
  defaultValues,
  onSubmit,
  children,
  className,
}: FormProps<T>) {
  const methods = useForm<z.infer<T>>({
    resolver: zodResolver(schema),
    defaultValues,
  });

  return (
    <FormProvider {...methods}>
      <form
        onSubmit={methods.handleSubmit(onSubmit)}
        className={className}
      >
        {children}
      </form>
    </FormProvider>
  );
}
/components/layouts - Layout Components
typescript
// components/layouts/
├── Header/
│   ├── Header.tsx
│   ├── Nav.tsx
│   ├── UserMenu.tsx
│   └── index.ts
├── Footer/
│   ├── Footer.tsx
│   ├── SocialLinks.tsx
│   └── index.ts
├── Sidebar/
│   ├── Sidebar.tsx
│   ├── SidebarNav.tsx
│   └── index.ts
├── Container.tsx                 // Container responsive
├── Grid.tsx                      // Grid layout
├── Section.tsx                   // Section wrapper
└── index.ts

// components/layouts/Container.tsx
import * as React from 'react';
import { cn } from '@/lib/utils';

interface ContainerProps extends React.HTMLAttributes<HTMLDivElement> {
  size?: 'sm' | 'md' | 'lg' | 'xl' | 'full';
}

export function Container({
  className,
  size = 'lg',
  ...props
}: ContainerProps) {
  return (
    <div
      className