# CATALOGO AUTHENTICATION v1

> **Versione**: 1.0  
> **Data**: 2026-01-27  
> **Ambito**: Next.js 14+ App Router, Auth.js v5, WebAuthn, MFA, RBAC/ABAC  

---

§ 1. AUTHENTICATION STRATEGY COMPARISON

§ 1.1 STRATEGY DECISION MATRIX

| Strategy | Security | UX | Complexity | Scalability | Offline | Best For |
|----------|----------|----|------------|-------------|---------|----------|
| Session-based | 🟢 High | 🟢 Excellent | 🟡 Medium | 🟡 Good | ❌ No | Traditional web apps, SSR-heavy sites |
| JWT Stateless | 🟡 Medium | 🟡 Good | 🟢 Low | 🟢 Excellent | ⚠️ Limited | Microservices, API-first, serverless |
| JWT + Refresh Token | 🟢 High | 🟢 Excellent | 🟡 Medium | 🟢 Excellent | ⚠️ Limited | SPAs, mobile apps, high-traffic APIs |
| OAuth 2.0 / OIDC | 🟢 High | 🟡 Good | 🔴 High | 🟡 Good | ❌ No | Enterprise apps, third-party integrations |
| Passkeys / WebAuthn | 🟢 Highest | 🟡 Medium | 🔴 High | 🟢 Excellent | ✅ Yes | Passwordless, high-security apps |
| Magic Links | 🟡 Medium | 🟢 Excellent | 🟢 Low | 🟢 Excellent | ❌ No | Consumer apps, quick sign-up |
| SMS/Email OTP | 🟡 Medium | 🟢 Excellent | 🟡 Medium | 🟢 Excellent | ❌ No | Mobile apps, verification flows |

§ 1.2 WHEN TO USE WHAT

| Scenario | Recommended Strategy | Why |
|----------|---------------------|-----|
| B2C SaaS app with email login | JWT + Refresh Token | Good UX, scales well, supports multiple devices |
| Enterprise internal tool | OIDC + SAML | Integrates with company SSO, enterprise security |
| Banking/finance app | Passkeys + TOTP | Highest security, phishing-resistant |
| E-commerce checkout | Magic Links | Reduces friction, no password to remember |
| Developer API platform | OAuth 2.0 (PKCE) | Standard for third-party API access |
| Mobile-first app | JWT + Refresh Token | Stateless, works offline with cached tokens |
| Admin dashboard | Session-based | Simple, works with SSR, easy permissions |
| IoT device auth | Client Credentials | Machine-to-machine, no user involved |

---

§ 2. AUTH.JS (NEXTAUTH) IMPLEMENTATION

§ 2.1 SETUP COMPLETO

typescript
// auth.ts
import NextAuth from "next-auth";
import { PrismaAdapter } from "@auth/prisma-adapter";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import GitHub from "next-auth/providers/github";
import Email from "next-auth/providers/email";
import { z } from "zod";
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/prisma";
import { sendVerificationEmail } from "@/lib/email";

// Input validation schema
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

export const { 
  handlers: { GET, POST },
  auth,
  signIn,
  signOut,
} = NextAuth({
  adapter: PrismaAdapter(prisma),
  
  // Session strategy
  session: {
    strategy: "jwt", // "jwt" for stateless, "database" for stateful
    maxAge: 30 * 24 * 60 * 60, // 30 days
    updateAge: 24 * 60 * 60, // 24 hours
  },
  
  // JWT configuration
  jwt: {
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  
  // Providers
  providers: [
    Credentials({
      name: "credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null;
        }
        
        // Validate input
        const validation = loginSchema.safeParse(credentials);
        if (!validation.success) {
          return null;
        }
        
        const { email, password } = validation.data;
        
        // Find user
        const user = await prisma.user.findUnique({
          where: { email },
          include: { accounts: true },
        });
        
        if (!user || !user.password) {
          return null;
        }
        
        // Verify password
        const isValid = await bcrypt.compare(password, user.password);
        if (!isValid) {
          return null;
        }
        
        // Return user object (excluding password)
        const { password: _, ...userWithoutPassword } = user;
        return userWithoutPassword;
      },
    }),
    
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!,
      allowDangerousEmailAccountLinking: true,
    }),
    
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
      allowDangerousEmailAccountLinking: true,
    }),
    
    Email({
      server: {
        host: process.env.EMAIL_SERVER_HOST,
        port: Number(process.env.EMAIL_SERVER_PORT),
        auth: {
          user: process.env.EMAIL_SERVER_USER,
          pass: process.env.EMAIL_SERVER_PASSWORD,
        },
      },
      from: process.env.EMAIL_FROM,
      sendVerificationRequest: async ({ identifier, url, provider }) => {
        await sendVerificationEmail({
          email: identifier,
          url,
          provider,
        });
      },
    }),
  ],
  
  // Callbacks
  callbacks: {
    async jwt({ token, user, trigger, session }) {
      // Initial sign in
      if (user) {
        token.id = user.id;
        token.role = user.role;
        token.email = user.email;
        token.emailVerified = user.emailVerified;
      }
      
      // Update session on client-side
      if (trigger === "update" && session) {
        token = { ...token, ...session };
      }
      
      return token;
    },
    
    async session({ session, token }) {
      if (session.user) {
        session.user.id = token.id as string;
        session.user.role = token.role as string;
        session.user.email = token.email as string;
        session.user.emailVerified = token.emailVerified as boolean;
      }
      return session;
    },
    
    async signIn({ user, account, profile, email }) {
      // Allow all sign ins
      return true;
    },
    
    async redirect({ url, baseUrl }) {
      // Allows relative callback URLs
      if (url.startsWith("/")) return `${baseUrl}${url}`;
      // Allows callback URLs on the same origin
      else if (new URL(url).origin === baseUrl) return url;
      return baseUrl;
    },
  },
  
  // Pages
  pages: {
    signIn: "/auth/signin",
    signOut: "/auth/signout",
    error: "/auth/error",
    verifyRequest: "/auth/verify-request",
    newUser: "/auth/new-user",
  },
  
  // Events
  events: {
    async createUser({ user }) {
      console.log(`User created: ${user.email}`);
    },
    async signIn({ user, account, isNewUser }) {
      console.log(`User signed in: ${user.email}`);
    },
    async signOut({ token, session }) {
      console.log(`User signed out: ${token.email}`);
    },
  },
  
  // Security
  cookies: {
    sessionToken: {
      name: `next-auth.session-token`,
      options: {
        httpOnly: true,
        sameSite: "lax",
        path: "/",
        secure: process.env.NODE_ENV === "production",
      },
    },
  },
  
  // Debug
  debug: process.env.NODE_ENV === "development",
});

§ 2.2 PRISMA SCHEMA PER AUTH.JS

prisma
// prisma/schema.prisma
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@index([userId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  image         String?
  password      String?   @db.Text
  role          String    @default("USER")
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Auth.js relations
  accounts      Account[]
  sessions      Session[]
  
  // Custom relations
  authenticators Authenticator[]
  mfaSettings    MFASettings?
  permissions    Permission[]
  
  @@map("users")
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

§ 2.3 MIDDLEWARE PROTECTION

typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { auth } from '@/auth';

// Paths that require authentication
const protectedPaths = [
  '/dashboard',
  '/profile',
  '/settings',
  '/api/private',
];

// Paths that are public
const publicPaths = [
  '/',
  '/auth/signin',
  '/auth/signup',
  '/auth/verify',
  '/api/public',
];

// Paths that redirect to dashboard if authenticated
const authRedirectPaths = [
  '/auth/signin',
  '/auth/signup',
];

export async function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Get session
  const session = await auth();
  const isAuthenticated = !!session?.user;
  
  // Check if path is protected
  const isProtectedPath = protectedPaths.some(path => 
    pathname.startsWith(path)
  );
  
  const isPublicPath = publicPaths.some(path => 
    pathname.startsWith(path)
  );
  
  const isAuthRedirectPath = authRedirectPaths.some(path => 
    pathname.startsWith(path)
  );
  
  // Redirect authenticated users away from auth pages
  if (isAuthRedirectPath && isAuthenticated) {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
  
  // Protect paths that require authentication
  if (isProtectedPath && !isAuthenticated) {
    const signInUrl = new URL('/auth/signin', request.url);
    signInUrl.searchParams.set('callbackUrl', pathname);
    return NextResponse.redirect(signInUrl);
  }
  
  // Add user info to headers for API routes
  if (pathname.startsWith('/api') && isAuthenticated) {
    const headers = new Headers(request.headers);
    headers.set('x-user-id', session.user.id);
    headers.set('x-user-role', session.user.role);
    
    const response = NextResponse.next({
      request: { headers },
    });
    
    return response;
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - public folder
     */
    '/((?!_next/static|_next/image|favicon.ico|public/).*)',
  ],
};

§ 2.4 SERVER-SIDE AUTH HELPERS

typescript
// lib/auth-helpers.ts
import { auth } from "@/auth";
import { redirect } from "next/navigation";

/**
 * Get current authenticated user
 */
export async function getCurrentUser() {
  const session = await auth();
  
  if (!session?.user) {
    return null;
  }
  
  return session.user;
}

/**
 * Get current user, redirect if not authenticated
 */
export async function requireAuth(redirectTo = '/auth/signin') {
  const user = await getCurrentUser();
  
  if (!user) {
    redirect(redirectTo);
  }
  
  return user;
}

/**
 * Check if user has specific role
 */
export async function requireRole(
  requiredRole: string | string[],
  redirectTo = '/unauthorized'
) {
  const user = await requireAuth();
  
  const roles = Array.isArray(requiredRole) ? requiredRole : [requiredRole];
  
  if (!roles.includes(user.role)) {
    redirect(redirectTo);
  }
  
  return user;
}

/**
 * Get user from request headers (for API routes)
 */
export function getUserFromRequest(request: Request) {
  const userId = request.headers.get('x-user-id');
  const userRole = request.headers.get('x-user-role');
  
  if (!userId || !userRole) {
    throw new Error('Unauthorized');
  }
  
  return {
    id: userId,
    role: userRole,
  };
}

/**
 * Check if user has permission
 */
export async function hasPermission(
  userId: string,
  permission: string
): Promise<boolean> {
  // Implementation depends on your permission system
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      permissions: true,
    },
  });
  
  if (!user) {
    return false;
  }
  
  return user.permissions.some(p => p.name === permission);
}

§ 2.5 CLIENT-SIDE HOOKS

typescript
// hooks/use-auth.ts
'use client';

import { useSession } from "next-auth/react";
import { useEffect } from "react";
import { useRouter } from "next/navigation";

/**
 * Custom hook to get current user session
 */
export function useAuth() {
  const { data: session, status, update } = useSession();
  
  return {
    user: session?.user,
    isLoading: status === "loading",
    isAuthenticated: status === "authenticated",
    updateSession: update,
  };
}

/**
 * Hook that redirects to login if not authenticated
 */
export function useRequireAuth(redirectTo = "/auth/signin") {
  const { user, isLoading, isAuthenticated } = useAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      router.push(`${redirectTo}?callbackUrl=${window.location.pathname}`);
    }
  }, [isLoading, isAuthenticated, router, redirectTo]);
  
  return {
    user,
    isLoading,
    isAuthenticated,
  };
}

/**
 * Hook to check if user has specific role
 */
export function useRequireRole(requiredRole: string | string[]) {
  const { user, isLoading, isAuthenticated } = useRequireAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!isLoading && user) {
      const roles = Array.isArray(requiredRole) ? requiredRole : [requiredRole];
      
      if (!roles.includes(user.role)) {
        router.push("/unauthorized");
      }
    }
  }, [user, isLoading, requiredRole, router]);
  
  return {
    user,
    isLoading,
    isAuthenticated,
  };
}

/**
 * Hook to check permissions
 */
export function usePermission(permission: string) {
  const { user, isLoading } = useAuth();
  
  // In a real app, this would check against user's permissions
  const hasPermission = !isLoading && user?.role === "ADMIN";
  
  return {
    hasPermission,
    isLoading,
  };
}

---

§ 3. PASSKEYS / WEBAUTHN

§ 3.1 WEBAUTHN FLOW DIAGRAM

Registration Flow:
1. Client → Server: Start registration
2. Server → Client: Public key creation options
3. Client → Authenticator: Create credential
4. Authenticator → Client: Public key
5. Client → Server: Send attestation response
6. Server: Verify and store credential

Authentication Flow:
1. Client → Server: Start authentication
2. Server → Client: Public key request options
3. Client → Authenticator: Get assertion
4. Authenticator → Client: Signed assertion
5. Client → Server: Send assertion response
6. Server: Verify assertion

§ 3.2 REGISTRATION IMPLEMENTATION

typescript
// app/api/auth/passkeys/register/route.ts
import { NextRequest, NextResponse } from "next/server";
import { 
  generateRegistrationOptions,
  verifyRegistrationResponse,
} from "@simplewebauthn/server";
import { prisma } from "@/lib/prisma";
import { rpID, rpName, origin } from "@/lib/webauthn-config";

export async function POST(request: NextRequest) {
  try {
    const { userId } = await request.json();
    
    if (!userId) {
      return NextResponse.json(
        { error: "User ID required" },
        { status: 400 }
      );
    }
    
    // Get user's existing authenticators
    const userAuthenticators = await prisma.authenticator.findMany({
      where: { userId },
    });
    
    // Generate registration options
    const options = await generateRegistrationOptions({
      rpName,
      rpID,
      userID: userId,
      userName: userId, // In production, use user email or username
      userDisplayName: userId,
      attestationType: "none",
      authenticatorSelection: {
        residentKey: "preferred",
        userVerification: "preferred",
      },
      excludeCredentials: userAuthenticators.map(auth => ({
        id: Buffer.from(auth.credentialID, 'base64'),
        type: "public-key" as const,
        transports: auth.transports as AuthenticatorTransport[],
      })),
    });
    
    // Store challenge for later verification
    await prisma.webAuthnChallenge.create({
      data: {
        userId,
        challenge: options.challenge,
        type: "registration",
        expiresAt: new Date(Date.now() + 5 * 60 * 1000), // 5 minutes
      },
    });
    
    return NextResponse.json(options);
  } catch (error) {
    console.error("Registration error:", error);
    return NextResponse.json(
      { error: "Failed to generate registration options" },
      { status: 500 }
    );
  }
}

// app/api/auth/passkeys/register/verify/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { userId, attestationResponse } = body;
    
    // Get stored challenge
    const challenge = await prisma.webAuthnChallenge.findFirst({
      where: {
        userId,
        type: "registration",
        expiresAt: { gt: new Date() },
      },
      orderBy: { createdAt: "desc" },
    });
    
    if (!challenge) {
      return NextResponse.json(
        { error: "Challenge expired or not found" },
        { status: 400 }
      );
    }
    
    // Verify registration response
    const verification = await verifyRegistrationResponse({
      response: attestationResponse,
      expectedChallenge: challenge.challenge,
      expectedOrigin: origin,
      expectedRPID: rpID,
      requireUserVerification: false,
    });
    
    if (!verification.verified || !verification.registrationInfo) {
      return NextResponse.json(
        { error: "Verification failed" },
        { status: 400 }
      );
    }
    
    const { registrationInfo } = verification;
    
    // Store the authenticator
    await prisma.authenticator.create({
      data: {
        userId,
        credentialID: Buffer.from(registrationInfo.credentialID).toString('base64'),
        credentialPublicKey: Buffer.from(registrationInfo.credentialPublicKey),
        counter: registrationInfo.counter,
        credentialDeviceType: registrationInfo.credentialDeviceType,
        credentialBackedUp: registrationInfo.credentialBackedUp,
        transports: attestationResponse.response.transports,
      },
    });
    
    // Clean up used challenge
    await prisma.webAuthnChallenge.delete({
      where: { id: challenge.id },
    });
    
    return NextResponse.json({ verified: true });
  } catch (error) {
    console.error("Verification error:", error);
    return NextResponse.json(
      { error: "Verification failed" },
      { status: 500 }
    );
  }
}

§ 3.3 AUTHENTICATION IMPLEMENTATION

typescript
// app/api/auth/passkeys/authenticate/route.ts
import { 
  generateAuthenticationOptions,
  verifyAuthenticationResponse,
} from "@simplewebauthn/server";
import { prisma } from "@/lib/prisma";
import { rpID, origin } from "@/lib/webauthn-config";

export async function POST(request: NextRequest) {
  try {
    const { userId } = await request.json();
    
    let userAuthenticators;
    
    if (userId) {
      // Existing user authentication
      userAuthenticators = await prisma.authenticator.findMany({
        where: { userId },
      });
    } else {
      // Username-less authentication (passkeys)
      userAuthenticators = await prisma.authenticator.findMany();
    }
    
    const options = await generateAuthenticationOptions({
      rpID,
      allowCredentials: userAuthenticators.map(auth => ({
        id: Buffer.from(auth.credentialID, 'base64'),
        type: "public-key" as const,
        transports: auth.transports as AuthenticatorTransport[],
      })),
      userVerification: "preferred",
    });
    
    // Store challenge
    await prisma.webAuthnChallenge.create({
      data: {
        userId: userId || null,
        challenge: options.challenge,
        type: "authentication",
        expiresAt: new Date(Date.now() + 5 * 60 * 1000),
      },
    });
    
    return NextResponse.json(options);
  } catch (error) {
    console.error("Authentication error:", error);
    return NextResponse.json(
      { error: "Failed to generate authentication options" },
      { status: 500 }
    );
  }
}

// app/api/auth/passkeys/authenticate/verify/route.ts
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { assertionResponse } = body;
    
    // Find authenticator by credential ID
    const credentialID = Buffer.from(assertionResponse.id, 'base64').toString('base64');
    const authenticator = await prisma.authenticator.findUnique({
      where: { credentialID },
      include: { user: true },
    });
    
    if (!authenticator) {
      return NextResponse.json(
        { error: "Authenticator not found" },
        { status: 400 }
      );
    }
    
    // Get stored challenge
    const challenge = await prisma.webAuthnChallenge.findFirst({
      where: {
        userId: authenticator.userId,
        type: "authentication",
        expiresAt: { gt: new Date() },
      },
      orderBy: { createdAt: "desc" },
    });
    
    if (!challenge) {
      return NextResponse.json(
        { error: "Challenge expired" },
        { status: 400 }
      );
    }
    
    // Verify authentication response
    const verification = await verifyAuthenticationResponse({
      response: assertionResponse,
      expectedChallenge: challenge.challenge,
      expectedOrigin: origin,
      expectedRPID: rpID,
      authenticator: {
        credentialID: Buffer.from(authenticator.credentialID, 'base64'),
        credentialPublicKey: authenticator.credentialPublicKey,
        counter: authenticator.counter,
        transports: authenticator.transports as AuthenticatorTransport[],
      },
      requireUserVerification: false,
    });
    
    if (!verification.verified) {
      return NextResponse.json(
        { error: "Authentication failed" },
        { status: 400 }
      );
    }
    
    // Update authenticator counter
    await prisma.authenticator.update({
      where: { id: authenticator.id },
      data: { counter: verification.authenticationInfo.newCounter },
    });
    
    // Clean up challenge
    await prisma.webAuthnChallenge.delete({
      where: { id: challenge.id },
    });
    
    // Create session/token for authenticated user
    return NextResponse.json({
      verified: true,
      userId: authenticator.userId,
      user: authenticator.user,
    });
  } catch (error) {
    console.error("Authentication verification error:", error);
    return NextResponse.json(
      { error: "Authentication failed" },
      { status: 500 }
    );
  }
}

§ 3.4 DATABASE SCHEMA PER PASSKEYS

prisma
// Add to your Prisma schema
model Authenticator {
  id                   String  @id @default(cuid())
  userId               String
  credentialID         String  @unique @db.Text
  credentialPublicKey  Bytes
  counter              Int     @default(0)
  credentialDeviceType String
  credentialBackedUp   Boolean
  transports           String[] // JSON array of transports
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@index([userId])
  @@map("authenticators")
}

model WebAuthnChallenge {
  id        String   @id @default(cuid())
  userId    String?  // Null for username-less authentication
  challenge String   @db.Text
  type      String   // "registration" or "authentication"
  expiresAt DateTime
  
  createdAt DateTime @default(now())
  
  @@index([userId])
  @@map("web_authn_challenges")
}

§ 3.5 WEBAUTHN CONFIGURATION

typescript
// lib/webauthn-config.ts
export const rpID = process.env.NEXT_PUBLIC_WEBAUTHN_RP_ID || "localhost";
export const rpName = process.env.NEXT_PUBLIC_WEBAUTHN_RP_NAME || "My App";
export const origin = process.env.NEXT_PUBLIC_WEBAUTHN_ORIGIN || "http://localhost:3000";

---

§ 4. MULTI-FACTOR AUTHENTICATION (MFA)

§ 4.1 MFA METHODS COMPARISON TABLE

| Method | Security | UX | Cost | Phishing Resistant | Setup Complexity |
|--------|----------|----|------|-------------------|------------------|
| TOTP (Authenticator App) | 🟢 High | 🟡 Medium | 🟢 Free | 🟡 Moderate | 🟡 Medium |
| SMS OTP | 🟡 Medium | 🟢 Excellent | 🔴 Expensive | ❌ No | 🟢 Low |
| Email OTP | 🟡 Medium | 🟡 Good | 🟡 Low | ❌ No | 🟢 Low |
| Passkeys | 🟢 Highest | 🟡 Medium | 🟢 Free | ✅ Yes | 🔴 High |
| Hardware Keys (FIDO2) | 🟢 Highest | 🟢 Good | 🔴 High | ✅ Yes | 🔴 High |
| Push Notification | 🟢 High | 🟢 Excellent | 🟡 Medium | 🟢 High | 🔴 High |

§ 4.2 TOTP IMPLEMENTATION

typescript
// lib/mfa/totp.ts
import { authenticator } from "otplib";
import * as QRCode from "qrcode";
import { prisma } from "@/lib/prisma";

// Configure TOTP
authenticator.options = {
  digits: 6,
  step: 30,
  window: 1, // Allow 1-step backward/forward for clock drift
};

/**
 * Generate a new TOTP secret for a user
 */
export async function generateTOTPSecret(userId: string) {
  const secret = authenticator.generateSecret();
  const issuer = process.env.NEXT_PUBLIC_APP_NAME || "MyApp";
  const userEmail = await getUserEmail(userId);
  
  // Generate otpauth URL for QR code
  const otpauth = authenticator.keyuri(
    encodeURIComponent(userEmail),
    encodeURIComponent(issuer),
    secret
  );
  
  // Store secret (hashed) in database
  await prisma.mFASettings.upsert({
    where: { userId },
    update: { 
      totpSecret: await hashSecret(secret),
      totpEnabled: false, // Not enabled until verified
    },
    create: {
      userId,
      totpSecret: await hashSecret(secret),
      totpEnabled: false,
    },
  });
  
  return {
    secret,
    otpauth,
  };
}

/**
 * Generate QR code data URL
 */
export async function generateQRCodeDataURL(otpauthUrl: string) {
  try {
    return await QRCode.toDataURL(otpauthUrl);
  } catch (error) {
    throw new Error("Failed to generate QR code");
  }
}

/**
 * Verify TOTP token
 */
export async function verifyTOTPToken(userId: string, token: string) {
  const mfaSettings = await prisma.mFASettings.findUnique({
    where: { userId },
  });
  
  if (!mfaSettings?.totpSecret) {
    throw new Error("TOTP not set up for user");
  }
  
  // In production, retrieve hashed secret from secure storage
  const secret = await decryptSecret(mfaSettings.totpSecret);
  
  const isValid = authenticator.check(token, secret);
  
  if (isValid) {
    // Enable TOTP on first successful verification
    if (!mfaSettings.totpEnabled) {
      await prisma.mFASettings.update({
        where: { userId },
        data: { totpEnabled: true },
      });
    }
  }
  
  return isValid;
}

/**
 * Check if user has TOTP enabled
 */
export async function isTOTPEnabled(userId: string) {
  const mfaSettings = await prisma.mFASettings.findUnique({
    where: { userId },
  });
  
  return mfaSettings?.totpEnabled ?? false;
}

// Helper functions
async function hashSecret(secret: string): Promise<string> {
  // Use bcrypt or similar to hash the secret
  const saltRounds = 12;
  return await bcrypt.hash(secret, saltRounds);
}

async function decryptSecret(hashedSecret: string): Promise<string> {
  // Retrieve from secure storage/HSM in production
  // This is a simplified example
  return hashedSecret; // In reality, decrypt from secure storage
}

async function getUserEmail(userId: string): Promise<string> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
  });
  
  return user?.email || userId;
}

§ 4.3 BACKUP CODES IMPLEMENTATION

typescript
// lib/mfa/backup-codes.ts
import crypto from "crypto";
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/prisma";

/**
 * Generate backup codes for a user
 */
export async function generateBackupCodes(userId: string, count = 10) {
  const codes: string[] = [];
  
  for (let i = 0; i < count; i++) {
    // Generate 8-character alphanumeric code with dashes
    const code = crypto.randomBytes(6).toString('hex').toUpperCase();
    const formattedCode = code.match(/.{1,4}/g)?.join('-') || code;
    codes.push(formattedCode);
  }
  
  // Hash codes before storing
  const hashedCodes = await Promise.all(
    codes.map(code => hashBackupCode(code))
  );
  
  // Store hashed codes
  await prisma.mFASettings.upsert({
    where: { userId },
    update: {
      backupCodes: hashedCodes,
    },
    create: {
      userId,
      backupCodes: hashedCodes,
    },
  });
  
  return codes; // Return plain codes to show to user once
}

/**
 * Verify a backup code
 */
export async function verifyBackupCode(userId: string, code: string) {
  const mfaSettings = await prisma.mFASettings.findUnique({
    where: { userId },
  });
  
  if (!mfaSettings?.backupCodes?.length) {
    return false;
  }
  
  // Check each hashed code
  for (const hashedCode of mfaSettings.backupCodes) {
    const isValid = await bcrypt.compare(code, hashedCode);
    if (isValid) {
      // Remove used code
      const updatedCodes = mfaSettings.backupCodes.filter(
        hc => hc !== hashedCode
      );
      
      await prisma.mFASettings.update({
        where: { userId },
        data: { backupCodes: updatedCodes },
      });
      
      return true;
    }
  }
  
  return false;
}

/**
 * Check if user has backup codes remaining
 */
export async function hasBackupCodes(userId: string) {
  const mfaSettings = await prisma.mFASettings.findUnique({
    where: { userId },
  });
  
  return (mfaSettings?.backupCodes?.length || 0) > 0;
}

/**
 * Hash a backup code
 */
async function hashBackupCode(code: string): Promise<string> {
  const saltRounds = 10;
  return await bcrypt.hash(code, saltRounds);
}

§ 4.4 MFA ENROLLMENT FLOW

typescript
// components/mfa/mfa-enrollment.tsx
'use client';

import { useState } from 'react';
import { generateTOTPSecret, generateQRCodeDataURL, verifyTOTPToken } from '@/lib/mfa/totp';
import { generateBackupCodes } from '@/lib/mfa/backup-codes';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Shield, CheckCircle, Copy, Download } from 'lucide-react';

interface MFAEnrollmentProps {
  userId: string;
  onComplete: () => void;
}

export function MFAEnrollment({ userId, onComplete }: MFAEnrollmentProps) {
  const [step, setStep] = useState<'setup' | 'verify' | 'backup'>('setup');
  const [qrCodeUrl, setQrCodeUrl] = useState<string>('');
  const [secret, setSecret] = useState<string>('');
  const [token, setToken] = useState('');
  const [error, setError] = useState('');
  const [backupCodes, setBackupCodes] = useState<string[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // Step 1: Setup TOTP
  const handleSetup = async () => {
    try {
      setIsLoading(true);
      const { secret: newSecret, otpauth } = await generateTOTPSecret(userId);
      const qrCode = await generateQRCodeDataURL(otpauth);
      
      setSecret(newSecret);
      setQrCodeUrl(qrCode);
      setStep('verify');
    } catch (err) {
      setError('Failed to setup MFA');
    } finally {
      setIsLoading(false);
    }
  };

  // Step 2: Verify TOTP
  const handleVerify = async () => {
    try {
      setIsLoading(true);
      const isValid = await verifyTOTPToken(userId, token);
      
      if (isValid) {
        setStep('backup');
      } else {
        setError('Invalid verification code');
      }
    } catch (err) {
      setError('Verification failed');
    } finally {
      setIsLoading(false);
    }
  };

  // Step 3: Generate backup codes
  const handleGenerateBackupCodes = async () => {
    try {
      setIsLoading(true);
      const codes = await generateBackupCodes(userId);
      setBackupCodes(codes);
    } catch (err) {
      setError('Failed to generate backup codes');
    } finally {
      setIsLoading(false);
    }
  };

  const handleCopyCodes = () => {
    navigator.clipboard.writeText(backupCodes.join('\n'));
  };

  const handleDownloadCodes = () => {
    const blob = new Blob([backupCodes.join('\n')], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'backup-codes.txt';
    a.click();
    URL.revokeObjectURL(url);
  };

  return (
    <Card className="max-w-md mx-auto">
      <CardHeader>
        <CardTitle className="flex items-center gap-2">
          <Shield className="h-5 w-5" />
          Setup Multi-Factor Authentication
        </CardTitle>
        <CardDescription>
          Add an extra layer of security to your account
        </CardDescription>
      </CardHeader>
      
      <CardContent className="space-y-6">
        {error && (
          <Alert variant="destructive">
            <AlertDescription>{error}</AlertDescription>
          </Alert>
        )}

        {/* Step 1: Setup TOTP */}
        {step === 'setup' && (
          <div className="space-y-4">
            <p className="text-sm text-muted-foreground">
              You'll need an authenticator app like Google Authenticator or Authy.
            </p>
            <Button 
              onClick={handleSetup} 
              disabled={isLoading}
              className="w-full"
            >
              {isLoading ? 'Setting up...' : 'Start Setup'}
            </Button>
          </div>
        )}

        {/* Step 2: Verify TOTP */}
        {step === 'verify' && qrCodeUrl && (
          <div className="space-y-4">
            <div className="text-center">
              <p className="text-sm font-medium mb-2">Scan QR Code</p>
              <img 
                src={qrCodeUrl} 
                alt="TOTP QR Code" 
                className="mx-auto h-48 w-48"
              />
              <p className="text-sm text-muted-foreground mt-2">
                Or enter secret manually:
              </p>
              <code className="text-xs bg-muted p-2 rounded block mt-1">
                {secret}
              </code>
            </div>
            
            <div className="space-y-2">
              <label className="text-sm font-medium">
                Enter verification code
              </label>
              <Input
                type="text"
                placeholder="000000"
                value={token}
                onChange={(e) => {
                  setToken(e.target.value.replace(/\D/g, ''));
                  setError('');
                }}
                maxLength={6}
                className="text-center text-lg tracking-widest"
              />
            </div>
            
            <Button 
              onClick={handleVerify} 
              disabled={isLoading || token.length !== 6}
              className="w-full"
            >
              {isLoading ? 'Verifying...' : 'Verify'}
            </Button>
          </div>
        )}

        {/* Step 3: Backup Codes */}
        {step === 'backup' && (
          <div className="space-y-4">
            <Alert>
              <CheckCircle className="h-4 w-4" />
              <AlertDescription>
                MFA has been successfully enabled!
              </AlertDescription>
            </Alert>
            
            <div>
              <p className="text-sm font-medium mb-2">
                Backup Codes
              </p>
              <p className="text-sm text-muted-foreground mb-4">
                Save these codes in a secure place. Each code can be used once.
              </p>
              
              {backupCodes.length > 0 ? (
                <div className="space-y-3">
                  <div className="grid grid-cols-2 gap-2">
                    {backupCodes.map((code, index) => (
                      <code 
                        key={index} 
                        className="text-sm bg-muted p-2 rounded text-center font-mono"
                      >
                        {code}
                      </code>
                    ))}
                  </div>
                  
                  <div className="flex gap-2">
                    <Button 
                      variant="outline" 
                      onClick={handleCopyCodes}
                      className="flex-1"
                    >
                      <Copy className="h-4 w-4 mr-2" />
                      Copy
                    </Button>
                    <Button 
                      variant="outline" 
                      onClick={handleDownloadCodes}
                      className="flex-1"
                    >
                      <Download className="h-4 w-4 mr-2" />
                      Download
                    </Button>
                  </div>
                </div>
              ) : (
                <Button 
                  onClick={handleGenerateBackupCodes} 
                  disabled={isLoading}
                  className="w-full"
                >
                  {isLoading ? 'Generating...' : 'Generate Backup Codes'}
                </Button>
              )}
            </div>
            
            <Button 
              onClick={onComplete} 
              className="w-full"
              disabled={backupCodes.length === 0}
            >
              Complete Setup
            </Button>
          </div>
        )}
      </CardContent>
    </Card>
  );
}

§ 4.5 MFA DATABASE SCHEMA

prisma
// Add to Prisma schema
model MFASettings {
  id          String   @id @default(cuid())
  userId      String   @unique
  totpSecret  String?  @db.Text // Hashed secret
  totpEnabled Boolean  @default(false)
  backupCodes String[] // JSON array of hashed backup codes
  
  // Additional MFA methods
  phoneNumber String?
  phoneVerified Boolean @default(false)
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("mfa_settings")
}

---

§ 5. ROLE-BASED ACCESS CONTROL (RBAC)

§ 5.1 PERMISSION MATRIX TABLE

| Role | users:read | users:write | admin:access | billing:manage | content:edit | content:delete |
|------|------------|-------------|--------------|----------------|--------------|----------------|
| User | ✅ Own only | ✅ Own only | ❌ | ❌ | ❌ | ❌ |
| Editor | ✅ All | ❌ | ❌ | ❌ | ✅ Assigned | ✅ Assigned |
| Moderator | ✅ All | ❌ | ❌ | ❌ | ✅ All | ❌ |
| Admin | ✅ All | ✅ All | ✅ | ❌ | ✅ All | ✅ All |
| Owner | ✅ All | ✅ All | ✅ | ✅ | ✅ All | ✅ All |

§ 5.2 RBAC IMPLEMENTATION

typescript
// lib/rbac.ts
import { prisma } from "@/lib/prisma";

// Permission types
export type PermissionAction = 
  | 'create' | 'read' | 'update' | 'delete' 
  | 'manage' | 'approve' | 'export';

export type PermissionResource = 
  | 'users' | 'roles' | 'permissions' 
  | 'content' | 'billing' | 'settings';

export type Permission = `${PermissionResource}:${PermissionAction}`;

// Predefined roles and permissions
export const ROLES = {
  USER: 'USER',
  EDITOR: 'EDITOR',
  MODERATOR: 'MODERATOR',
  ADMIN: 'ADMIN',
  OWNER: 'OWNER',
} as const;

export type Role = keyof typeof ROLES;

// Role permission mappings
const ROLE_PERMISSIONS: Record<Role, Permission[]> = {
  USER: [
    'users:read:own',
    'users:update:own',
    'content:read',
  ],
  EDITOR: [
    'users:read:own',
    'users:update:own',
    'content:read',
    'content:create',
    'content:update:own',
    'content:delete:own',
  ],
  MODERATOR: [
    'users:read',
    'content:read',
    'content:update',
    'content:approve',
  ],
  ADMIN: [
    'users:read',
    'users:create',
    'users:update',
    'users:delete',
    'content:read',
    'content:create',
    'content:update',
    'content:delete',
    'content:approve',
    'content:export',
    'roles:read',
    'settings:read',
    'settings:update',
  ],
  OWNER: [
    'users:read',
    'users:create',
    'users:update',
    'users:delete',
    'users:manage',
    'content:read',
    'content:create',
    'content:update',
    'content:delete',
    'content:approve',
    'content:export',
    'content:manage',
    'roles:read',
    'roles:create',
    'roles:update',
    'roles:delete',
    'permissions:read',
    'permissions:update',
    'billing:read',
    'billing:manage',
    'settings:read',
    'settings:update',
    'settings:manage',
  ],
};

/**
 * Check if user has specific permission
 */
export async function hasPermission(
  userId: string,
  permission: Permission,
  resourceId?: string
): Promise<boolean> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      permissions: true,
      roles: {
        include: { permissions: true },
      },
    },
  });
  
  if (!user) {
    return false;
  }
  
  // Check direct permissions
  const hasDirectPermission = user.permissions.some(
    p => p.name === permission
  );
  
  if (hasDirectPermission) {
    return true;
  }
  
  // Check role permissions
  for (const role of user.roles) {
    const roleHasPermission = role.permissions.some(
      p => p.name === permission
    );
    
    if (roleHasPermission) {
      return true;
    }
  }
  
  // Check if permission is wildcard (e.g., "users:*")
  const [resource, action] = permission.split(':');
  const wildcardPermission = `${resource}:*` as Permission;
  
  const hasWildcard = user.permissions.some(
    p => p.name === wildcardPermission
  );
  
  if (hasWildcard) {
    return true;
  }
  
  // Check role wildcard permissions
  for (const role of user.roles) {
    const roleHasWildcard = role.permissions.some(
      p => p.name === wildcardPermission
    );
    
    if (roleHasWildcard) {
      return true;
    }
  }
  
  // Check ownership for resource-specific permissions
  if (resourceId) {
    // This would check if user owns the resource
    // Implementation depends on your data model
    const isOwner = await checkResourceOwnership(userId, resource, resourceId);
    if (isOwner) {
      return true;
    }
  }
  
  return false;
}

/**
 * Middleware to require specific permission
 */
export async function requirePermission(
  userId: string,
  permission: Permission,
  resourceId?: string
) {
  const allowed = await hasPermission(userId, permission, resourceId);
  
  if (!allowed) {
    throw new Error('Permission denied');
  }
}

/**
 * Get all permissions for a user
 */
export async function getUserPermissions(userId: string): Promise<Permission[]> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      permissions: true,
      roles: {
        include: { permissions: true },
      },
    },
  });
  
  if (!user) {
    return [];
  }
  
  const permissions = new Set<Permission>();
  
  // Add direct permissions
  user.permissions.forEach(p => permissions.add(p.name as Permission));
  
  // Add role permissions
  user.roles.forEach(role => {
    role.permissions.forEach(p => permissions.add(p.name as Permission));
  });
  
  return Array.from(permissions);
}

/**
 * Check resource ownership
 */
async function checkResourceOwnership(
  userId: string,
  resource: string,
  resourceId: string
): Promise<boolean> {
  switch (resource) {
    case 'users':
      return userId === resourceId;
    case 'content':
      const content = await prisma.content.findUnique({
        where: { id: resourceId },
      });
      return content?.authorId === userId;
    default:
      return false;
  }
}

// React hook for permissions
export function usePermission(permission: Permission, resourceId?: string) {
  const { user } = useAuth();
  const [hasPermission, setHasPermission] = useState(false);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    if (user?.id) {
      checkPermission();
    }
  }, [user?.id, permission, resourceId]);
  
  async function checkPermission() {
    if (!user?.id) return;
    
    setIsLoading(true);
    const allowed = await hasPermission(user.id, permission, resourceId);
    setHasPermission(allowed);
    setIsLoading(false);
  }
  
  return { hasPermission, isLoading };
}

§ 5.3 DATABASE SCHEMA PER RBAC

prisma
// Add to Prisma schema
model Role {
  id          String       @id @default(cuid())
  name        String       @unique
  description String?
  
  users       UserRole[]
  permissions RolePermission[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("roles")
}

model Permission {
  id          String       @id @default(cuid())
  name        String       @unique
  description String?
  
  users       UserPermission[]
  roles       RolePermission[]
  
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  @@map("permissions")
}

model UserRole {
  id     String @id @default(cuid())
  userId String
  roleId String
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  role Role @relation(fields: [roleId], references: [id], onDelete: Cascade)
  
  assignedAt DateTime @default(now())
  assignedBy String?
  
  @@unique([userId, roleId])
  @@map("user_roles")
}

model UserPermission {
  id           String @id @default(cuid())
  userId       String
  permissionId String
  
  user       User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  permission Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)
  
  grantedAt DateTime @default(now())
  grantedBy String?
  
  @@unique([userId, permissionId])
  @@map("user_permissions")
}

model RolePermission {
  id           String @id @default(cuid())
  roleId       String
  permissionId String
  
  role       Role       @relation(fields: [roleId], references: [id], onDelete: Cascade)
  permission Permission @relation(fields: [permissionId], references: [id], onDelete: Cascade)
  
  createdAt DateTime @default(now())
  
  @@unique([roleId, permissionId])
  @@map("role_permissions")
}

---

§ 6. ATTRIBUTE-BASED ACCESS CONTROL (ABAC)

§ 6.1 ABAC VS RBAC COMPARISON TABLE

| Aspect | RBAC | ABAC |
|--------|------|------|
| Decision Basis | Role membership | Attributes + policies |
| Flexibility | Limited | Highly flexible |
| Complexity | Simple to moderate | Complex |
| Scalability | Good for simple orgs | Scales with complexity |
| Maintenance | Role changes | Policy updates |
| Example | "Admins can edit all posts" | "Users can edit posts they created within last 7 days" |

§ 6.2 POLICY DEFINITION ENGINE

typescript
// lib/abac.ts
import { z } from 'zod';

// Attribute types
type UserAttributes = {
  id: string;
  role: string;
  department: string;
  clearance: number;
  location: string;
  employmentStatus: 'active' | 'inactive' | 'suspended';
};

type ResourceAttributes = {
  id: string;
  type: string;
  ownerId: string;
  department: string;
  sensitivity: 'public' | 'internal' | 'confidential' | 'secret';
  createdAt: Date;
  location: string;
};

type EnvironmentAttributes = {
  time: Date;
  ipAddress: string;
  device: string;
  location: string;
  riskScore: number;
};

type Action = 'read' | 'write' | 'delete' | 'approve' | 'share';

// Policy definition
const policySchema = z.object({
  id: z.string(),
  name: z.string(),
  description: z.string().optional(),
  effect: z.enum(['allow', 'deny']),
  actions: z.array(z.string()),
  resources: z.array(z.string()),
  conditions: z.array(z.object({
    attribute: z.string(),
    operator: z.enum([
      'equals', 'notEquals', 'greaterThan', 'lessThan',
      'greaterThanOrEquals', 'lessThanOrEquals', 'in',
      'notIn', 'contains', 'startsWith', 'endsWith',
      'matches', 'between',
    ]),
    value: z.union([z.string(), z.number(), z.boolean(), z.date(), z.array(z.any())]),
  })),
  priority: z.number().default(0),
});

type Policy = z.infer<typeof policySchema>;

/**
 * ABAC Policy Engine
 */
export class PolicyEngine {
  private policies: Policy[] = [];

  constructor(policies: Policy[] = []) {
    this.policies = policies;
  }

  /**
   * Evaluate access request
   */
  evaluate(
    user: UserAttributes,
    action: Action,
    resource: ResourceAttributes,
    environment: EnvironmentAttributes
  ): boolean {
    // Collect matching policies
    const matchingPolicies = this.policies.filter(policy => {
      // Check actions
      if (!this.matchesAction(policy.actions, action)) {
        return false;
      }

      // Check resources
      if (!this.matchesResource(policy.resources, resource.type)) {
        return false;
      }

      // Check conditions
      if (!this.evaluateConditions(policy.conditions, { user, resource, environment })) {
        return false;
      }

      return true;
    });

    // Sort by priority (highest first)
    matchingPolicies.sort((a, b) => b.priority - a.priority);

    // Deny takes precedence over allow
    const denyPolicy = matchingPolicies.find(p => p.effect === 'deny');
    if (denyPolicy) {
      return false;
    }

    // Check for allow policies
    const allowPolicy = matchingPolicies.find(p => p.effect === 'allow');
    return !!allowPolicy;
  }

  /**
   * Check if action matches policy actions
   */
  private matchesAction(policyActions: string[], requestedAction: Action): boolean {
    return policyActions.some(pattern => {
      if (pattern === '*') return true;
      if (pattern === requestedAction) return true;
      
      // Support wildcards like "read:*" or "write:document"
      const [patternResource, patternAction] = pattern.split(':');
      const [requestedResource, requestedActionPart] = requestedAction.split(':');
      
      if (patternAction === '*') {
        return patternResource === requestedResource;
      }
      
      return pattern === requestedAction;
    });
  }

  /**
   * Check if resource matches policy resources
   */
  private matchesResource(policyResources: string[], resourceType: string): boolean {
    return policyResources.some(pattern => {
      if (pattern === '*') return true;
      if (pattern === resourceType) return true;
      
      // Support wildcards like "document:*" or "user:profile"
      const [patternType, patternId] = pattern.split(':');
      const [resourceTypePart, resourceId] = resourceType.split(':');
      
      if (patternId === '*') {
        return patternType === resourceTypePart;
      }
      
      return pattern === resourceType;
    });
  }

  /**
   * Evaluate policy conditions
   */
  private evaluateConditions(
    conditions: Policy['conditions'],
    context: {
      user: UserAttributes;
      resource: ResourceAttributes;
      environment: EnvironmentAttributes;
    }
  ): boolean {
    for (const condition of conditions) {
      // Get attribute value from context
      const value = this.getAttributeValue(condition.attribute, context);
      
      // Compare based on operator
      const result = this.compareValues(
        value,
        condition.operator,
        condition.value
      );
      
      if (!result) {
        return false;
      }
    }
    
    return true;
  }

  /**
   * Get attribute value from context
   */
  private getAttributeValue(attributePath: string, context: any): any {
    const parts = attributePath.split('.');
    let value = context;
    
    for (const part of parts) {
      value = value[part];
      if (value === undefined) {
        return undefined;
      }
    }
    
    return value;
  }

  /**
   * Compare values based on operator
   */
  private compareValues(
    actual: any,
    operator: Policy['conditions'][0]['operator'],
    expected: any
  ): boolean {
    switch (operator) {
      case 'equals':
        return actual === expected;
      case 'notEquals':
        return actual !== expected;
      case 'greaterThan':
        return actual > expected;
      case 'lessThan':
        return actual < expected;
      case 'greaterThanOrEquals':
        return actual >= expected;
      case 'lessThanOrEquals':
        return actual <= expected;
      case 'in':
        return Array.isArray(expected) && expected.includes(actual);
      case 'notIn':
        return Array.isArray(expected) && !expected.includes(actual);
      case 'contains':
        return typeof actual === 'string' && actual.includes(expected);
      case 'startsWith':
        return typeof actual === 'string' && actual.startsWith(expected);
      case 'endsWith':
        return typeof actual === 'string' && actual.endsWith(expected);
      case 'matches':
        return new RegExp(expected).test(actual);
      case 'between':
        return actual >= expected[0] && actual <= expected[1];
      default:
        return false;
    }
  }
}

§ 6.3 EXAMPLE POLICIES

typescript
// lib/policies.ts
import { Policy } from './abac';

// Example policies for different scenarios
export const examplePolicies: Policy[] = [
  // Policy 1: Users can edit their own posts within 7 days
  {
    id: 'user-edit-own-post',
    name: 'User can edit own recent posts',
    description: 'Users can edit posts they created within the last 7 days',
    effect: 'allow',
    actions: ['write:post'],
    resources: ['post:*'],
    conditions: [
      { attribute: 'user.id', operator: 'equals', value: '${resource.ownerId}' },
      { attribute: 'resource.createdAt', operator: 'greaterThan', value: '${now - 7d}' },
    ],
    priority: 10,
  },

  // Policy 2: Managers can approve expenses in their department
  {
    id: 'manager-approve-expenses',
    name: 'Manager can approve department expenses',
    description: 'Managers can approve expenses in their department up to $10,000',
    effect: 'allow',
    actions: ['approve:expense'],
    resources: ['expense:*'],
    conditions: [
      { attribute: 'user.role', operator: 'equals', value: 'manager' },
      { attribute: 'user.department', operator: 'equals', value: '${resource.department}' },
      { attribute: 'resource.amount', operator: 'lessThanOrEquals', value: 10000 },
    ],
    priority: 20,
  },

  // Policy 3: Deny access to confidential resources outside office hours
  {
    id: 'confidential-office-hours',
    name: 'Confidential resources only during office hours',
    description: 'Deny access to confidential resources outside 9 AM - 5 PM',
    effect: 'deny',
    actions: ['read:*'],
    resources: ['*:confidential'],
    conditions: [
      { attribute: 'environment.time', operator: 'between', value: ['${9:00}', '${17:00}'] },
    ],
    priority: 30,
  },

  // Policy 4: High-clearance users can access secret resources
  {
    id: 'high-clearance-access',
    name: 'High clearance access',
    description: 'Users with clearance >= 4 can access secret resources',
    effect: 'allow',
    actions: ['read:*', 'write:*'],
    resources: ['*:secret'],
    conditions: [
      { attribute: 'user.clearance', operator: 'greaterThanOrEquals', value: 4 },
    ],
    priority: 40,
  },

  // Policy 5: IP restriction for admin access
  {
    id: 'admin-ip-restriction',
    name: 'Admin access only from corporate IP',
    description: 'Admin users can only access from corporate IP range',
    effect: 'deny',
    actions: ['*:*'],
    resources: ['*:*'],
    conditions: [
      { attribute: 'user.role', operator: 'equals', value: 'admin' },
      { attribute: 'environment.ipAddress', operator: 'notIn', value: ['10.0.0.0/8', '192.168.0.0/16'] },
    ],
    priority: 100, // High priority for security
  },
];

/**
 * Helper to create policy with attribute placeholders
 */
export function createPolicy(
  template: Omit<Policy, 'conditions'> & {
    conditions: Array<{
      attribute: string;
      operator: Policy['conditions'][0]['operator'];
      value: string | number | boolean | Date | any[];
    }>
  }
): Policy {
  return template as Policy;
}

/**
 * Validate and interpolate policy values
 */
export function interpolatePolicy(
  policy: Policy,
  context: {
    user: any;
    resource: any;
    environment: any;
    now: Date;
  }
): Policy {
  const interpolated = JSON.parse(JSON.stringify(policy));
  
  // Interpolate condition values
  for (const condition of interpolated.conditions) {
    if (typeof condition.value === 'string' && condition.value.startsWith('${')) {
      const expression = condition.value.slice(2, -1);
      condition.value = evaluateExpression(expression, context);
    }
  }
  
  return interpolated;
}

/**
 * Evaluate expression in context
 */
function evaluateExpression(expression: string, context: any): any {
  const { user, resource, environment, now } = context;
  
  // Simple expression evaluation
  if (expression === 'now') {
    return now;
  }
  
  if (expression === 'now - 7d') {
    return new Date(now.getTime() - 7 * 24 * 60 * 60 * 1000);
  }
  
  // Return expression as-is for now (full implementation would need an expression parser)
  return expression;
}

---

§ 7. SESSION MANAGEMENT

§ 7.1 SESSION SECURITY CHECKLIST TABLE

| Aspect | Implementation | Risk if Missing |
|--------|----------------|-----------------|
| Secure cookie | `secure: true` in production | Session hijacking via MITM |
| HttpOnly | `httpOnly: true` | XSS attacks stealing cookies |
| SameSite | `sameSite: "lax"` or `"strict"` | CSRF attacks |
| Session rotation | Rotate on auth change | Session fixation |
| Absolute timeout | `maxAge` setting | Long-lived compromised sessions |
| Idle timeout | `updateAge` setting | Abandoned sessions stay valid |
| Concurrent session limit | Track active sessions | Account sharing, credential stuffing |
| Token binding | Bind to user agent/IP | Token reuse across devices |

§ 7.2 SECURE SESSION CONFIGURATION

typescript
// auth.ts - Session configuration
import NextAuth from "next-auth";

export const { auth } = NextAuth({
  session: {
    strategy: "jwt", // or "database"
    
    // Session expiration
    maxAge: 30 * 24 * 60 * 60, // 30 days
    updateAge: 24 * 60 * 60, // 24 hours
    
    // Generate session token
    generateSessionToken: () => {
      return crypto.randomUUID();
    },
  },
  
  cookies: {
    sessionToken: {
      name: `__Secure-next-auth.session-token`,
      options: {
        httpOnly: true,
        sameSite: "lax",
        path: "/",
        secure: process.env.NODE_ENV === "production",
        maxAge: 30 * 24 * 60 * 60, // 30 days
      },
    },
    callbackUrl: {
      name: `__Secure-next-auth.callback-url`,
      options: {
        httpOnly: true,
        sameSite: "lax",
        path: "/",
        secure: process.env.NODE_ENV === "production",
      },
    },
    csrfToken: {
      name: `__Host-next-auth.csrf-token`,
      options: {
        httpOnly: true,
        sameSite: "lax",
        path: "/",
        secure: process.env.NODE_ENV === "production",
      },
    },
    pkceCodeVerifier: {
      name: `__Secure-next-auth.pkce.code_verifier`,
      options: {
        httpOnly: true,
        sameSite: "lax",
        path: "/",
        secure: process.env.NODE_ENV === "production",
        maxAge: 60 * 15, // 15 minutes
      },
    },
  },
  
  callbacks: {
    async jwt({ token, user, trigger, session }) {
      // Rotate session on important events
      if (user) {
        // Initial sign in
        token.sessionId = crypto.randomUUID();
        token.createdAt = Date.now();
      }
      
      // Rotate on password change
      if (trigger === "update" && session?.passwordChanged) {
        token.sessionId = crypto.randomUUID();
        token.createdAt = Date.now();
      }
      
      // Rotate on role change
      if (trigger === "update" && session?.roleChanged) {
        token.sessionId = crypto.randomUUID();
        token.createdAt = Date.now();
      }
      
      return token;
    },
  },
});

§ 7.3 SESSION INVALIDATION

typescript
// lib/session-management.ts
import { prisma } from "@/lib/prisma";
import { signOut } from "next-auth/react";

/**
 * Sign out current user
 */
export async function logout() {
  await signOut({ redirect: false });
  
  // Additional cleanup if needed
  localStorage.removeItem("user_preferences");
  sessionStorage.clear();
}

/**
 * Invalidate all sessions for a user
 */
export async function invalidateAllSessions(userId: string) {
  // For database sessions
  await prisma.session.deleteMany({
    where: { userId },
  });
  
  // For JWT sessions, we need to track invalid tokens
  await prisma.invalidatedToken.createMany({
    data: [
      {
        userId,
        reason: "all_sessions_invalidated",
        invalidatedAt: new Date(),
      },
    ],
  });
}

/**
 * Invalidate all other sessions except current one
 */
export async function invalidateOtherSessions(
  userId: string, 
  currentSessionId: string
) {
  // For database sessions
  await prisma.session.deleteMany({
    where: {
      userId,
      NOT: { id: currentSessionId },
    },
  });
  
  // For JWT sessions
  await prisma.invalidatedToken.createMany({
    data: [
      {
        userId,
        sessionId: currentSessionId,
        reason: "other_sessions_invalidated",
        invalidatedAt: new Date(),
      },
    ],
  });
}

/**
 * Check if a JWT token is invalidated
 */
export async function isTokenInvalidated(
  tokenId: string, 
  userId: string
): Promise<boolean> {
  const invalidated = await prisma.invalidatedToken.findFirst({
    where: {
      OR: [
        { tokenId, userId },
        { userId, allTokens: true },
      ],
      expiresAt: { gt: new Date() },
    },
  });
  
  return !!invalidated;
}

/**
 * Set concurrent session limit
 */
export async function enforceSessionLimit(
  userId: string, 
  maxSessions = 5
): Promise<void> {
  const sessions = await prisma.session.findMany({
    where: { userId },
    orderBy: { expires: "desc" },
  });
  
  if (sessions.length > maxSessions) {
    // Remove oldest sessions
    const sessionsToDelete = sessions.slice(maxSessions);
    
    await prisma.session.deleteMany({
      where: {
        id: { in: sessionsToDelete.map(s => s.id) },
      },
    });
  }
}

// Database schema for session management
model InvalidatedToken {
  id          String   @id @default(cuid())
  userId      String
  tokenId     String?  // Specific JWT ID (jti)
  sessionId   String?  // Database session ID
  allTokens   Boolean  @default(false) // Invalidate all tokens for user
  reason      String
  invalidatedAt DateTime @default(now())
  expiresAt   DateTime // When this invalidation expires
  
  @@index([userId])
  @@index([tokenId])
  @@map("invalidated_tokens")
}

---

§ 8. PASSWORD SECURITY

§ 8.1 PASSWORD POLICY TABLE

| Rule | Minimum | Recommended | Implementation |
|------|---------|-------------|----------------|
| Length | 8 chars | 12+ chars | `z.string().min(12)` |
| Complexity | 1 uppercase, 1 lowercase, 1 number | 1 uppercase, 1 lowercase, 1 number, 1 special | Regex validation |
| Breached check | ❌ Optional | ✅ Required | Have I Been Pwned API |
| History | Last 3 passwords | Last 5 passwords | Store password hashes history |
| Age limit | Never expires | 90 days | Password rotation policy |
| Lockout | 5 attempts | 10 attempts with 15 min lock | Track failed attempts |

§ 8.2 PASSWORD HASHING

typescript
// lib/password.ts
import bcrypt from "bcryptjs";
import { prisma } from "@/lib/prisma";

// Cost factors - adjust based on server capacity
const SALT_ROUNDS = 12;
const MAX_PASSWORD_HISTORY = 5;

/**
 * Hash a password
 */
export async function hashPassword(password: string): Promise<string> {
  return await bcrypt.hash(password, SALT_ROUNDS);
}

/**
 * Verify a password
 */
export async function verifyPassword(
  password: string, 
  hashedPassword: string
): Promise<boolean> {
  return await bcrypt.compare(password, hashedPassword);
}

/**
 * Check if password meets policy
 */
export function validatePassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];
  
  // Length check
  if (password.length < 12) {
    errors.push("Password must be at least 12 characters long");
  }
  
  // Complexity checks
  if (!/[A-Z]/.test(password)) {
    errors.push("Password must contain at least one uppercase letter");
  }
  
  if (!/[a-z]/.test(password)) {
    errors.push("Password must contain at least one lowercase letter");
  }
  
  if (!/[0-9]/.test(password)) {
    errors.push("Password must contain at least one number");
  }
  
  if (!/[^A-Za-z0-9]/.test(password)) {
    errors.push("Password must contain at least one special character");
  }
  
  // Common password check
  const commonPasswords = [
    "password", "123456", "qwerty", "letmein", "welcome",
    "admin", "password123", "123456789", "12345678",
  ];
  
  if (commonPasswords.includes(password.toLowerCase())) {
    errors.push("Password is too common");
  }
  
  // Sequential characters check
  if (/(.)\1{2,}/.test(password)) {
    errors.push("Password cannot contain repeated characters");
  }
  
  // Sequential numbers/letters check
  if (/123|234|345|456|567|678|789/.test(password) ||
      /abc|bcd|cde|def|efg|fgh|ghi|hij|ijk|jkl|klm|lmn|mno/.test(password)) {
    errors.push("Password cannot contain sequential characters");
  }
  
  return {
    valid: errors.length === 0,
    errors,
  };
}

/**
 * Check if password has been breached using HIBP
 */
export async function checkPasswordBreach(password: string): Promise<boolean> {
  // Implement k-anonymity with Have I Been Pwned API
  const hash = await sha1(password);
  const prefix = hash.slice(0, 5);
  const suffix = hash.slice(5).toUpperCase();
  
  try {
    const response = await fetch(
      `https://api.pwnedpasswords.com/range/${prefix}`,
      {
        headers: {
          "User-Agent": "MyApp-Password-Checker",
        },
      }
    );
    
    if (!response.ok) {
      // If API fails, allow password (fail open)
      console.warn("HIBP API failed, allowing password");
      return false;
    }
    
    const data = await response.text();
    const hashes = data.split("\n");
    
    // Check if our suffix is in the list
    return hashes.some(line => {
      const [hashSuffix] = line.split(":");
      return hashSuffix === suffix;
    });
  } catch (error) {
    console.error("HIBP check failed:", error);
    return false;
  }
}

/**
 * Check password against user's password history
 */
export async function checkPasswordHistory(
  userId: string, 
  newPassword: string
): Promise<boolean> {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      passwordHistory: {
        orderBy: { createdAt: "desc" },
        take: MAX_PASSWORD_HISTORY,
      },
    },
  });
  
  if (!user) {
    return false;
  }
  
  // Check against historical passwords
  for (const oldPassword of user.passwordHistory) {
    const isSame = await verifyPassword(newPassword, oldPassword.hashedPassword);
    if (isSame) {
      return false;
    }
  }
  
  return true;
}

/**
 * Update password with history tracking
 */
export async function updatePassword(
  userId: string, 
  newPassword: string
): Promise<void> {
  // Validate password
  const validation = validatePassword(newPassword);
  if (!validation.valid) {
    throw new Error(`Invalid password: ${validation.errors.join(", ")}`);
  }
  
  // Check against breached passwords
  const isBreached = await checkPasswordBreach(newPassword);
  if (isBreached) {
    throw new Error("Password has been compromised in data breaches");
  }
  
  // Check password history
  const isNew = await checkPasswordHistory(userId, newPassword);
  if (!isNew) {
    throw new Error("Password cannot be reused");
  }
  
  // Hash new password
  const hashedPassword = await hashPassword(newPassword);
  
  // Update in transaction
  await prisma.$transaction(async (tx) => {
    // Add to history
    await tx.passwordHistory.create({
      data: {
        userId,
        hashedPassword,
      },
    });
    
    // Update current password
    await tx.user.update({
      where: { id: userId },
      data: { password: hashedPassword },
    });
    
    // Clean up old history
    const oldPasswords = await tx.passwordHistory.findMany({
      where: { userId },
      orderBy: { createdAt: "desc" },
      skip: MAX_PASSWORD_HISTORY,
    });
    
    if (oldPasswords.length > 0) {
      await tx.passwordHistory.deleteMany({
        where: {
          id: { in: oldPasswords.map(p => p.id) },
        },
      });
    }
  });
}

// SHA-1 helper for HIBP
async function sha1(message: string): Promise<string> {
  const msgBuffer = new TextEncoder().encode(message);
  const hashBuffer = await crypto.subtle.digest("SHA-1", msgBuffer);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, "0")).join("");
}

// Database schema for password history
model PasswordHistory {
  id             String   @id @default(cuid())
  userId         String
  hashedPassword String   @db.Text
  createdAt      DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
  @@map("password_history")
}

§ 8.3 PASSWORD RESET FLOW

typescript
// lib/password-reset.ts
import crypto from "crypto";
import { prisma } from "@/lib/prisma";
import { sendPasswordResetEmail } from "@/lib/email";
import { hashPassword } from "@/lib/password";

const RESET_TOKEN_EXPIRY = 15 * 60 * 1000; // 15 minutes

/**
 * Request password reset
 */
export async function requestPasswordReset(email: string): Promise<void> {
  const user = await prisma.user.findUnique({
    where: { email },
  });
  
  if (!user) {
    // Don't reveal if user exists
    return;
  }
  
  // Generate reset token
  const resetToken = crypto.randomBytes(32).toString("hex");
  const resetTokenHash = await hashPassword(resetToken);
  
  // Calculate expiry
  const expiresAt = new Date(Date.now() + RESET_TOKEN_EXPIRY);
  
  // Store token (delete existing ones first)
  await prisma.$transaction(async (tx) => {
    await tx.passwordResetToken.deleteMany({
      where: { userId: user.id },
    });
    
    await tx.passwordResetToken.create({
      data: {
        userId: user.id,
        tokenHash: resetTokenHash,
        expiresAt,
      },
    });
  });
  
  // Send email
  const resetUrl = `${process.env.NEXT_PUBLIC_APP_URL}/auth/reset-password?token=${resetToken}&email=${encodeURIComponent(email)}`;
  
  await sendPasswordResetEmail({
    email: user.email,
    resetUrl,
    expiresIn: "15 minutes",
  });
}

/**
 * Validate reset token
 */
export async function validateResetToken(
  email: string, 
  token: string
): Promise<{ valid: boolean; userId?: string }> {
  const user = await prisma.user.findUnique({
    where: { email },
  });
  
  if (!user) {
    return { valid: false };
  }
  
  const resetToken = await prisma.passwordResetToken.findFirst({
    where: {
      userId: user.id,
      expiresAt: { gt: new Date() },
    },
    orderBy: { createdAt: "desc" },
  });
  
  if (!resetToken) {
    return { valid: false };
  }
  
  // Verify token
  const isValid = await bcrypt.compare(token, resetToken.tokenHash);
  
  if (!isValid) {
    return { valid: false };
  }
  
  return {
    valid: true,
    userId: user.id,
  };
}

/**
 * Reset password using token
 */
export async function resetPasswordWithToken(
  email: string,
  token: string,
  newPassword: string
): Promise<{ success: boolean; error?: string }> {
  try {
    // Validate token
    const validation = await validateResetToken(email, token);
    
    if (!validation.valid || !validation.userId) {
      return { 
        success: false, 
        error: "Invalid or expired reset token" 
      };
    }
    
    // Update password
    await updatePassword(validation.userId, newPassword);
    
    // Delete used token
    await prisma.passwordResetToken.deleteMany({
      where: { userId: validation.userId },
    });
    
    // Invalidate all sessions
    await invalidateAllSessions(validation.userId);
    
    return { success: true };
  } catch (error) {
    console.error("Password reset error:", error);
    return { 
      success: false, 
      error: error instanceof Error ? error.message : "Password reset failed" 
    };
  }
}

// Database schema for password reset
model PasswordResetToken {
  id         String   @id @default(cuid())
  userId     String
  tokenHash  String   @db.Text
  expiresAt  DateTime
  createdAt  DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
  @@map("password_reset_tokens")
}

---

§ 9. OAUTH 2.0 / OIDC DEEP DIVE

§ 9.1 OAUTH FLOWS COMPARISON

| Flow | Use Case | Security | Tokens | Refresh |
|------|----------|----------|--------|---------|
| Authorization Code | Web apps with server | 🟢 High | Access + Refresh | ✅ Yes |
| Authorization Code + PKCE | SPAs, mobile apps | 🟢 High | Access + Refresh | ✅ Yes |
| Implicit | Legacy SPAs (deprecated) | 🟡 Medium | Access only | ❌ No |
| Client Credentials | Machine-to-machine | 🟢 High | Access only | ❌ No |
| Device Code | Limited-input devices | 🟢 High | Access + Refresh | ✅ Yes |
| Refresh Token | Token renewal | 🟢 High | New Access | ✅ Yes |

§ 9.2 PKCE IMPLEMENTATION

typescript
// lib/oauth/pkce.ts
import crypto from "crypto";

/**
 * Generate code verifier for PKCE
 */
export function generateCodeVerifier(): string {
  const randomBytes = crypto.randomBytes(32);
  return randomBytes
    .toString("base64")
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=/g, "");
}

/**
 * Generate code challenge from verifier
 */
export async function generateCodeChallenge(verifier: string): Promise<string> {
  const encoder = new TextEncoder();
  const data = encoder.encode(verifier);
  const digest = await crypto.subtle.digest("SHA-256", data);
  
  return Buffer.from(digest)
    .toString("base64")
    .replace(/\+/g, "-")
    .replace(/\//g, "_")
    .replace(/=/g, "");
}

/**
 * Complete PKCE flow with Auth.js
 */
export async function handlePKCEFlow(
  providerId: string,
  redirectUri: string
): Promise<{ url: string; verifier: string }> {
  // Generate PKCE values
  const codeVerifier = generateCodeVerifier();
  const codeChallenge = await generateCodeChallenge(codeVerifier);
  
  // Build authorization URL
  const params = new URLSearchParams({
    client_id: process.env[`${providerId.toUpperCase()}_CLIENT_ID`]!,
    redirect_uri: redirectUri,
    response_type: "code",
    scope: "openid profile email",
    code_challenge: codeChallenge,
    code_challenge_method: "S256",
    state: crypto.randomBytes(16).toString("hex"),
  });
  
  const url = `https://${providerId}.com/oauth/authorize?${params.toString()}`;
  
  // Store verifier in secure storage (httpOnly cookie or server session)
  await storePKCEVerifier(codeVerifier);
  
  return { url, verifier: codeVerifier };
}

/**
 * Exchange code for tokens with PKCE
 */
export async function exchangeCodeForToken(
  providerId: string,
  code: string,
  codeVerifier: string,
  redirectUri: string
): Promise<{ accessToken: string; refreshToken?: string; idToken?: string }> {
  const tokenUrl = `https://${providerId}.com/oauth/token`;
  
  const response = await fetch(tokenUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: new URLSearchParams({
      grant_type: "authorization_code",
      client_id: process.env[`${providerId.toUpperCase()}_CLIENT_ID`]!,
      client_secret: process.env[`${providerId.toUpperCase()}_CLIENT_SECRET`]!,
      redirect_uri: redirectUri,
      code,
      code_verifier: codeVerifier,
    }),
  });
  
  if (!response.ok) {
    throw new Error("Failed to exchange code for token");
  }
  
  const tokens = await response.json();
  
  return {
    accessToken: tokens.access_token,
    refreshToken: tokens.refresh_token,
    idToken: tokens.id_token,
  };
}

/**
 * Store PKCE verifier securely
 */
async function storePKCEVerifier(verifier: string): Promise<void> {
  // Store in httpOnly cookie or server session
  // This depends on your implementation
}

§ 9.3 TOKEN REFRESH STRATEGY

typescript
// lib/oauth/token-refresh.ts
interface TokenSet {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
  idToken?: string;
}

interface TokenResponse {
  access_token: string;
  refresh_token?: string;
  expires_in: number;
  id_token?: string;
}

/**
 * Refresh access token using refresh token
 */
export async function refreshAccessToken(
  providerId: string,
  refreshToken: string
): Promise<TokenSet> {
  const tokenUrl = `https://${providerId}.com/oauth/token`;
  
  const response = await fetch(tokenUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: new URLSearchParams({
      grant_type: "refresh_token",
      client_id: process.env[`${providerId.toUpperCase()}_CLIENT_ID`]!,
      client_secret: process.env[`${providerId.toUpperCase()}_CLIENT_SECRET`]!,
      refresh_token: refreshToken,
    }),
  });
  
  if (!response.ok) {
    throw new Error("Failed to refresh token");
  }
  
  const tokens: TokenResponse = await response.json();
  
  return {
    accessToken: tokens.access_token,
    refreshToken: tokens.refresh_token || refreshToken, // Use new if provided
    expiresAt: Date.now() + tokens.expires_in * 1000,
    idToken: tokens.id_token,
  };
}

/**
 * Token refresh with rotation
 */
export async function refreshTokenWithRotation(
  providerId: string,
  oldRefreshToken: string
): Promise<TokenSet> {
  try {
    const newTokens = await refreshAccessToken(providerId, oldRefreshToken);
    
    // If new refresh token provided, invalidate old one
    if (newTokens.refreshToken !== oldRefreshToken) {
      await invalidateRefreshToken(providerId, oldRefreshToken);
    }
    
    return newTokens;
  } catch (error) {
    // If refresh fails, the token might have been revoked
    await invalidateRefreshToken(providerId, oldRefreshToken);
    throw error;
  }
}

/**
 * Silent refresh - refresh before expiration
 */
export function setupSilentRefresh(
  tokenSet: TokenSet,
  onRefresh: (newTokenSet: TokenSet) => void,
  onError: (error: Error) => void
): () => void {
  // Calculate when to refresh (5 minutes before expiration)
  const refreshTime = tokenSet.expiresAt - 5 * 60 * 1000;
  const timeUntilRefresh = Math.max(refreshTime - Date.now(), 0);
  
  const timeoutId = setTimeout(async () => {
    try {
      const newTokenSet = await refreshAccessToken(
        "your-provider",
        tokenSet.refreshToken
      );
      onRefresh(newTokenSet);
      
      // Schedule next refresh
      setupSilentRefresh(newTokenSet, onRefresh, onError);
    } catch (error) {
      onError(error as Error);
    }
  }, timeUntilRefresh);
  
  // Return cleanup function
  return () => clearTimeout(timeoutId);
}

/**
 * Invalidate refresh token (when user logs out or token is compromised)
 */
async function invalidateRefreshToken(
  providerId: string,
  refreshToken: string
): Promise<void> {
  const revokeUrl = `https://${providerId}.com/oauth/revoke`;
  
  await fetch(revokeUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: new URLSearchParams({
      token: refreshToken,
      client_id: process.env[`${providerId.toUpperCase()}_CLIENT_ID`]!,
      client_secret: process.env[`${providerId.toUpperCase()}_CLIENT_SECRET`]!,
    }),
  });
  
  // Also remove from local storage
  localStorage.removeItem("oauth_tokens");
}

---

§ 10. AUTHENTICATION CHECKLIST

STRATEGY
□ Authentication method chosen (Session/JWT/OAuth)
□ Session vs JWT decision documented
□ Token expiration configured (access: 15min-1h, refresh: 7-30 days)
□ Refresh token strategy implemented (rotation recommended)
□ Passwordless option considered (passkeys/magic links)

AUTH.JS SETUP
□ Providers configured (Credentials, Google, GitHub, etc.)
□ Database adapter connected (Prisma)
□ Callbacks customized (jwt, session, signIn)
□ Custom pages created (signin, signout, error, verify)
□ Middleware protection active for protected routes
□ Email provider configured for magic links/verification

SECURITY
□ HTTPS enforced in production
□ Secure cookies configured (httpOnly, secure, sameSite)
□ CSRF protection enabled (Auth.js built-in)
□ Rate limiting on auth endpoints (login, register, password reset)
□ Brute force protection (account lockout after N attempts)
□ Password policy enforced (length, complexity, history)
□ Breached password check (HIBP integration)
□ Session timeout configured (absolute + idle)

MFA
□ TOTP support for authenticator apps
□ Backup codes generated and stored securely
□ Recovery flow documented and tested
□ MFA enforcement for sensitive operations
□ Device trust/remember me option

AUTHORIZATION
□ RBAC/ABAC implemented based on needs
□ Permission checks on all protected routes
□ API authorization middleware
□ Resource-level permissions (ownership checks)
□ Audit logging for permission changes

SESSION MANAGEMENT
□ Session timeout configured (server + client)
□ Session rotation on privilege change
□ Concurrent session handling (limit + management)
□ Sign out invalidates session server-side
□ Session revocation for compromised accounts

PASSWORD SECURITY
□ Password hashing with appropriate cost factor (bcrypt/argon2)
□ Password reset flow with secure tokens
□ Password history enforcement
□ Account lockout after failed attempts
□ Password strength meter with real-time feedback

COMPLIANCE
□ Password reset flow works without current password
□ Account lockout policy documented
□ Audit logging for all auth events (login, logout, password change)
□ GDPR: Data export/deletion capabilities
□ Privacy policy includes auth data usage

TESTING
□ Unit tests for auth helpers
□ Integration tests for auth flows
□ E2E tests for login/logout
□ Security tests (OWASP ZAP, etc.)
□ Load tests for auth endpoints

MONITORING
□ Failed login attempts logged and alerted
□ Unusual activity detection (geolocation, device changes)
□ Token usage analytics
□ Session duration monitoring
□ Regular security audit of auth system

DEPLOYMENT
□ Environment-specific configurations
□ Secrets management (tokens, keys, passwords)
□ Backup/restore procedures for auth data
□ Disaster recovery plan for auth system
□ Regular dependency updates (auth libraries)

---

# INTEGRAZIONE: 05-OUTPUT-MODULO-AUTH.md
_Integrato il 2026-01-29 14:51_

Assolutamente! Ecco il modulo di autenticazione completo per Next.js 14 con Auth.js v5, rispettando tutti i requisiti e le stime di riga.

Questo setup include:
- Autenticazione con Credenziali (Email/Password)
- Autenticazione OAuth (Google, GitHub)
- Verifica Email
- Reset Password
- Validazione Zod
- Integrazione Prisma
- tRPC per le API server-side
- Componenti UI React con `react-hook-form`
- Middleware di protezione delle route
- Test unitari di base

---

§ SETUP INIZIALE (ASSUNZIONI)

Prima di incollare i file, assicurati di avere le seguenti dipendenze installate:

bash
npm install next-auth@5.0.0-beta.10 @auth/prisma-adapter prisma bcryptjs zod react-hook-form @hookform/resolvers @tanstack/react-query @trpc/client @trpc/react-query @trpc/server @prisma/client
npm install -D vitest @types/bcryptjs

**Configurazione Prisma:**
Assicurati di avere un file `prisma/schema.prisma` e di aver eseguito `npx prisma db push` o `npx prisma migrate dev`.
Il file `src/lib/prisma.ts` dovrebbe essere configurato come segue:

typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

**Configurazione tRPC:**
Assicurati di avere un setup tRPC di base, ad esempio:

typescript
// src/server/trpc/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server';
import superjson from 'superjson';
import { type Session } from 'next-auth';

interface CreateContextOptions {
  session: Session | null;
}

export const createTRPCContext = (opts: CreateContextOptions) => {
  return {
    session: opts.session,
  };
};

const t = initTRPC.context<typeof createTRPCContext>().transformer(superjson).create();

const enforceUserIsAuthenticated = t.middleware(({ ctx, next }) => {
  if (!ctx.session || !ctx.session.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({
    ctx: {
      session: { ...ctx.session, user: ctx.session.user },
    },
  });
});

export const router = t.router;
export const publicProcedure = t.procedure;
export const protectedProcedure = t.procedure.use(enforceUserIsAuthenticated);

**Variabili d'ambiente:**
Crea un file `.env.local` con:

AUTH_SECRET="your_super_secret_key_here"
AUTH_URL="http://localhost:3000" # O il tuo dominio di produzione

GOOGLE_CLIENT_ID="YOUR_GOOGLE_CLIENT_ID"
GOOGLE_CLIENT_SECRET="YOUR_GOOGLE_CLIENT_SECRET"

GITHUB_CLIENT_ID="YOUR_GITHUB_CLIENT_ID"
GITHUB_CLIENT_SECRET="YOUR_GITHUB_CLIENT_SECRET"

DATABASE_URL="postgresql://user:password@host:port/database" # La tua stringa di connessione al DB

---

§ FILE 21: PRISMA/SCHEMA-AUTH.PRISMA (80 RIGHE)

prisma
// prisma/schema.prisma
// Aggiungi o estendi questo schema nel tuo file prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql" // O il tuo provider preferito
  url      = env("DATABASE_URL")
}

enum Role {
  USER
  ADMIN
}

model User {
  id            String    @id @default(cuid())
  name          String?
  email         String    @unique
  emailVerified DateTime?
  password      String?   // Campo per password hashata, null per utenti OAuth
  image         String?
  role          Role      @default(USER)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  accounts      Account[]
  sessions      Session[]
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  id        String   @id @default(cuid())
  email     String
  token     String   @unique
  expires   DateTime
  createdAt DateTime @default(now())

  @@unique([email, token])
}

model PasswordResetToken {
  id        String   @id @default(cuid())
  email     String
  token     String   @unique
  expires   DateTime
  createdAt DateTime @default(now())

  @@unique([email, token])
}

---

§ FILE 7: SRC/LIB/VALIDATIONS/AUTH.TS (80 RIGHE)

typescript
// src/lib/validations/auth.ts
import { z } from 'zod';

const passwordValidation = z.string()
  .min(8, "La password deve contenere almeno 8 caratteri")
  .regex(/[A-Z]/, "La password deve contenere almeno una lettera maiuscola")
  .regex(/[a-z]/, "La password deve contenere almeno una lettera minuscola")
  .regex(/[0-9]/, "La password deve contenere almeno un numero")
  .regex(/[^a-zA-Z0-9]/, "La password deve contenere almeno un carattere speciale");

export const loginSchema = z.object({
  email: z.string().email("Inserisci un'email valida"),
  password: z.string().min(1, "La password è richiesta"),
});

export const registerSchema = z.object({
  name: z.string().min(2, "Il nome è richiesto").max(50, "Il nome è troppo lungo"),
  email: z.string().email("Inserisci un'email valida"),
  password: passwordValidation,
  confirmPassword: z.string(),
  terms: z.literal(true, {
    errorMap: () => ({ message: "Devi accettare i termini e le condizioni" }),
  }),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Le password non corrispondono",
  path: ["confirmPassword"],
});

export const forgotPasswordSchema = z.object({
  email: z.string().email("Inserisci un'email valida"),
});

export const resetPasswordSchema = z.object({
  password: passwordValidation,
  confirmPassword: z.string(),
}).refine((data) => data.password === data.confirmPassword, {
  message: "Le password non corrispondono",
  path: ["confirmPassword"],
});

export const changePasswordSchema = z.object({
  oldPassword: z.string().min(1, "La vecchia password è richiesta"),
  newPassword: passwordValidation,
  confirmNewPassword: z.string(),
}).refine((data) => data.newPassword === data.confirmNewPassword, {
  message: "Le nuove password non corrispondono",
  path: ["confirmNewPassword"],
});

export const updateProfileSchema = z.object({
  name: z.string().min(2, "Il nome è richiesto").max(50, "Il nome è troppo lungo").optional(),
  email: z.string().email("Inserisci un'email valida").optional(),
  image: z.string().url("URL immagine non valido").optional().or(z.literal('')),
});

export type LoginInput = z.infer<typeof loginSchema>;
export type RegisterInput = z.infer<typeof registerSchema>;
export type ForgotPasswordInput = z.infer<typeof forgotPasswordSchema>;
export type ResetPasswordInput = z.infer<typeof resetPasswordSchema>;
export type ChangePasswordInput = z.infer<typeof changePasswordSchema>;
export type UpdateProfileInput = z.infer<typeof updateProfileSchema>;

---

§ FILE 3: SRC/LIB/AUTH/PASSWORD.TS (50 RIGHE)

typescript
// src/lib/auth/password.ts
import bcrypt from 'bcryptjs';
import { z } from 'zod';

const SALT_ROUNDS = 10;

export async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

export async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

export function generateResetToken(): string {
  // Questo token è per l'URL, non il token di reset effettivo nel DB
  // Il token effettivo sarà generato e salvato nel DB in tokens.ts
  return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
}

// Funzione di validazione della forza della password basata sullo schema Zod
export function validatePasswordStrength(password: string): { valid: boolean; errors: string[] } {
  const passwordSchema = z.string()
    .min(8, "La password deve contenere almeno 8 caratteri")
    .regex(/[A-Z]/, "La password deve contenere almeno una lettera maiuscola")
    .regex(/[a-z]/, "La password deve contenere almeno una lettera minuscola")
    .regex(/[0-9]/, "La password deve contenere almeno un numero")
    .regex(/[^a-zA-Z0-9]/, "La password deve contenere almeno un carattere speciale");

  const result = passwordSchema.safeParse(password);
  if (result.success) {
    return { valid: true, errors: [] };
  } else {
    return { valid: false, errors: result.error.errors.map(err => err.message) };
  }
}

---

§ FILE 4: SRC/LIB/AUTH/TOKENS.TS (80 RIGHE)

typescript
// src/lib/auth/tokens.ts
import { prisma } from '@/lib/prisma';
import crypto from 'crypto';

const VERIFICATION_TOKEN_EXPIRATION_HOURS = 24;
const PASSWORD_RESET_TOKEN_EXPIRATION_HOURS = 1;

export async function generateVerificationToken(email: string): Promise<string> {
  const token = crypto.randomBytes(32).toString('hex');
  const expires = new Date(Date.now() + VERIFICATION_TOKEN_EXPIRATION_HOURS * 60 * 60 * 1000);

  await prisma.verificationToken.deleteMany({
    where: { email },
  });

  await prisma.verificationToken.create({
    data: {
      email,
      token,
      expires,
    },
  });

  return token;
}

export async function verifyVerificationToken(token: string): Promise<{ email: string } | null> {
  const existingToken = await prisma.verificationToken.findUnique({
    where: { token },
  });

  if (!existingToken) {
    return null;
  }

  if (existingToken.expires < new Date()) {
    await prisma.verificationToken.delete({
      where: { id: existingToken.id },
    });
    return null; // Token scaduto
  }

  await prisma.verificationToken.delete({
    where: { id: existingToken.id },
  });

  return { email: existingToken.email };
}

export async function generatePasswordResetToken(email: string): Promise<string> {
  const token = crypto.randomBytes(32).toString('hex');
  const expires = new Date(Date.now() + PASSWORD_RESET_TOKEN_EXPIRATION_HOURS * 60 * 60 * 1000);

  await prisma.passwordResetToken.deleteMany({
    where: { email },
  });

  await prisma.passwordResetToken.create({
    data: {
      email,
      token,
      expires,
    },
  });

  return token;
}

export async function verifyPasswordResetToken(token: string): Promise<{ email: string } | null> {
  const existingToken = await prisma.passwordResetToken.findUnique({
    where: { token },
  });

  if (!existingToken) {
    return null;
  }

  if (existingToken.expires < new Date()) {
    await prisma.passwordResetToken.delete({
      where: { id: existingToken.id },
    });
    return null; // Token scaduto
  }

  await prisma.passwordResetToken.delete({
    where: { id: existingToken.id },
    // Non cancellare subito per evitare attacchi di riutilizzo, ma per questo esempio lo facciamo
  });

  return { email: existingToken.email };
}

---

§ FILE 0: SRC/LIB/EMAIL.TS (PLACEHOLDER PER INVIO EMAIL)

typescript
// src/lib/email.ts
// Questo è un placeholder. In un'applicazione reale, useresti un servizio come Nodemailer, SendGrid, Resend, ecc.

interface EmailOptions {
  to: string;
  subject: string;
  html: string;
  text?: string;
}

export async function sendEmail(options: EmailOptions): Promise<void> {
  console.log(`--- Sending Email ---`);
  console.log(`To: ${options.to}`);
  console.log(`Subject: ${options.subject}`);
  console.log(`HTML: ${options.html}`);
  console.log(`---------------------`);

  // Qui integreresti il tuo provider di email (es. Nodemailer, Resend)
  // Esempio con Resend (richiede installazione e configurazione API key):
  /*
  import { Resend } from 'resend';
  const resend = new Resend(process.env.RESEND_API_KEY);
  try {
    await resend.emails.send({
      from: 'onboarding@resend.dev', // Sostituisci con la tua email verificata
      to: options.to,
      subject: options.subject,
      html: options.html,
      text: options.text,
    });
    console.log('Email sent successfully via Resend');
  } catch (error) {
    console.error('Error sending email via Resend:', error);
  }
  */
}

---

§ FILE 1: SRC/AUTH.TS (200 RIGHE)

typescript
// src/auth.ts
import NextAuth, { type DefaultSession } from "next-auth";
import { PrismaAdapter } from "@auth/prisma-adapter";
import Credentials from "next-auth/providers/credentials";
import Google from "next-auth/providers/google";
import GitHub from "next-auth/providers/github";
import { prisma } from "@/lib/prisma";
import { hashPassword, verifyPassword } from "@/lib/auth/password";
import { generateVerificationToken } from "@/lib/auth/tokens";
import { sendEmail } from "@/lib/email";
import { loginSchema } from "@/lib/validations/auth";
import { z } from "zod";

// Estendi il tipo Session per includere id e role
declare module "next-auth" {
  interface Session extends DefaultSession {
    user: {
      id: string;
      role: "USER" | "ADMIN";
    } & DefaultSession["user"];
  }

  interface User {
    role: "USER" | "ADMIN";
  }
}

export const { handlers, auth, signIn, signOut } = NextAuth({
  adapter: PrismaAdapter(prisma),
  session: { strategy: "jwt" },
  pages: {
    signIn: "/login",
    error: "/auth/error",
    verifyRequest: "/auth/verify-request", // Pagina per mostrare che è stata inviata una email di verifica
    newUser: "/register", // Reindirizza dopo la registrazione OAuth
  },
  providers: [
    Credentials({
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      authorize: async (credentials) => {
        const validatedFields = loginSchema.safeParse(credentials);

        if (validatedFields.success) {
          const { email, password } = validatedFields.data;

          const user = await prisma.user.findUnique({
            where: { email },
          });

          if (!user || !user.password) {
            throw new Error("Credenziali non valide");
          }

          const passwordsMatch = await verifyPassword(password, user.password);

          if (passwordsMatch) {
            if (!user.emailVerified) {
              // Se l'email non è verificata, potresti voler inviare un'altra email di verifica
              // o reindirizzare a una pagina che lo richieda.
              // Per ora, lanciamo un errore.
              throw new Error("Email non verificata. Controlla la tua casella di posta.");
            }
            return {
              id: user.id,
              name: user.name,
              email: user.email,
              image: user.image,
              role: user.role,
            };
          }
        }
        throw new Error("Credenziali non valide");
      },
    }),
    Google({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      allowDangerousEmailAccountLinking: true, // Attenzione: abilita il linking automatico
    }),
    GitHub({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET,
      allowDangerousEmailAccountLinking: true, // Attenzione: abilita il linking automatico
    }),
  ],
  callbacks: {
    async jwt({ token, user, trigger, session }) {
      if (user) {
        token.id = user.id;
        token.role = user.role;
      }

      // Aggiorna il token JWT se l'utente ha aggiornato il profilo
      if (trigger === "update" && session?.user) {
        token.name = session.user.name;
        token.email = session.user.email;
        token.image = session.user.image;
      }

      return token;
    },
    async session({ session, token }) {
      if (token.id && session.user) {
        session.user.id = token.id as string;
        session.user.role = token.role as "USER" | "ADMIN";
      }
      return session;
    },
    async authorized({ auth, request }) {
      const { pathname } = request.nextUrl;
      const isAuthenticated = !!auth?.user;

      // Proteggi le route /dashboard e /admin
      if (pathname.startsWith("/dashboard") || pathname.startsWith("/admin")) {
        if (!isAuthenticated) {
          return Response.redirect(new URL("/login", request.nextUrl));
        }
        // Esempio di protezione per ruoli specifici
        if (pathname.startsWith("/admin") && auth?.user?.role !== "ADMIN") {
          return Response.redirect(new URL("/dashboard", request.nextUrl));
        }
      }

      // Reindirizza gli utenti autenticati da /login, /register, /forgot-password
      if (isAuthenticated && (pathname === "/login" || pathname === "/register" || pathname === "/forgot-password")) {
        return Response.redirect(new URL("/dashboard", request.nextUrl));
      }

      return true;
    },
  },
  events: {
    async createUser({ user }) {
      if (user.email) {
        // Se l'utente si registra con credenziali, invia email di verifica
        if (!user.emailVerified) {
          const verificationToken = await generateVerificationToken(user.email);
          const verificationUrl = `${process.env.AUTH_URL}/verify-email?token=${verificationToken}`;

          await sendEmail({
            to: user.email,
            subject: "Verifica la tua email",
            html: `<p>Clicca sul link per verificare la tua email: <a href="${verificationUrl}">Verifica Email</a></p>`,
          });
        }
      }
    },
    async signIn({ user, account, isNewUser }) {
      // Se l'utente si è autenticato con credenziali e l'email non è verificata
      if (account?.provider === "credentials" && user.email && !user.emailVerified) {
        // Potresti voler impedire il login o reindirizzare a una pagina di verifica
        // Per ora, permettiamo il login ma l'utente avrà un flag emailVerified=false
        // e il frontend dovrà gestirlo.
        // Oppure, si può lanciare un errore qui per bloccare il login fino alla verifica.
        // throw new Error("Email non verificata. Controlla la tua casella di posta.");
      }

      // Se un utente OAuth si registra, assicurati che abbia un ruolo di default
      if (isNewUser && user.id) {
        await prisma.user.update({
          where: { id: user.id },
          data: { role: "USER" },
        });
      }
    },
  },
});

---

§ FILE 2: SRC/MIDDLEWARE.TS (50 RIGHE)

typescript
// src/middleware.ts
import { auth } from "./auth";

// Il middleware di Auth.js è già configurato per proteggere le route tramite i callbacks `authorized`
// in src/auth.ts. Questo file serve principalmente per esportare il middleware.

export default auth;

// Configura il matcher per specificare quali percorsi devono essere intercettati dal middleware.
// Questo matcher è cruciale per le prestazioni e la corretta applicazione delle regole.
// Le route che non matchano qui non passeranno attraverso il middleware di Auth.js.
export const config = {
  matcher: [
    /*
     * Match all request paths except for the ones starting with:
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     * - auth (auth pages like /login, /register, /forgot-password, /verify-email)
     * - public assets (e.g., /images, /icons)
     */
    "/((?!api|_next/static|_next/image|favicon.ico|auth|images|icons|.*\\..*).*)",
    "/dashboard/:path*",
    "/admin/:path*",
  ],
};

---

§ FILE 5: SRC/SERVER/SERVICES/AUTH-SERVICE.TS (200 RIGHE)

typescript
// src/server/services/auth-service.ts
import { prisma } from '@/lib/prisma';
import { hashPassword, verifyPassword } from '@/lib/auth/password';
import {
  generateVerificationToken,
  verifyVerificationToken,
  generatePasswordResetToken,
  verifyPasswordResetToken,
} from '@/lib/auth/tokens';
import { sendEmail } from '@/lib/email';
import {
  LoginInput,
  RegisterInput,
  ResetPasswordInput,
  ChangePasswordInput,
  UpdateProfileInput,
} from '@/lib/validations/auth';
import { TRPCError } from '@trpc/server';
import { User } from '@prisma/client';

export class AuthService {
  async register(data: RegisterInput): Promise<User> {
    const { name, email, password } = data;

    const existingUser = await prisma.user.findUnique({ where: { email } });
    if (existingUser) {
      throw new TRPCError({ code: 'CONFLICT', message: 'Un utente con questa email esiste già.' });
    }

    const hashedPassword = await hashPassword(password);

    const user = await prisma.user.create({
      data: {
        name,
        email,
        password: hashedPassword,
        role: 'USER',
      },
    });

    // Invia email di verifica
    const verificationToken = await generateVerificationToken(email);
    const verificationUrl = `${process.env.AUTH_URL}/verify-email?token=${verificationToken}`;
    await sendEmail({
      to: email,
      subject: "Verifica la tua email per NextAuth App",
      html: `<p>Clicca sul link per verificare la tua email: <a href="${verificationUrl}">Verifica Email</a></p>`,
    });

    return user;
  }

  // Nota: la logica di login è gestita direttamente da NextAuth nel provider Credentials.
  // Questa funzione è più per un'API REST o per un caso d'uso specifico che non usa signIn di NextAuth.
  // Per tRPC, useremo signIn direttamente nel client o un'implementazione custom se necessario.
  async login(data: LoginInput): Promise<{ user: User }> {
    const { email, password } = data;

    const user = await prisma.user.findUnique({ where: { email } });
    if (!user || !user.password) {
      throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Credenziali non valide.' });
    }

    const passwordsMatch = await verifyPassword(password, user.password);
    if (!passwordsMatch) {
      throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Credenziali non valide.' });
    }

    if (!user.emailVerified) {
      throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Email non verificata. Controlla la tua casella di posta.' });
    }

    return { user };
  }

  // Per JWT, il logout è principalmente client-side (cancellare il cookie/token).
  // Questa funzione potrebbe essere usata per invalidare sessioni basate su DB o token specifici.
  async logout(userId: string): Promise<void> {
    // Se si usano sessioni basate su DB, si potrebbero cancellare qui.
    // Per JWT, non c'è un'azione server-side diretta per "logout" di un token valido.
    // Potrebbe essere implementata una blacklist di JWT se necessario, ma è più complesso.
    console.log(`User ${userId} logged out (server-side no-op for JWT strategy).`);
  }

  async verifyEmail(token: string): Promise<void> {
    const tokenPayload = await verifyVerificationToken(token);
    if (!tokenPayload) {
      throw new TRPCError({ code: 'BAD_REQUEST', message: 'Token di verifica non valido o scaduto.' });
    }

    const user = await prisma.user.findUnique({ where: { email: tokenPayload.email } });
    if (!user) {
      throw new TRPCError({ code: 'NOT_FOUND', message: 'Utente non trovato.' });
    }

    if (user.emailVerified) {
      throw new TRPCError({ code: 'CONFLICT', message: 'Email già verificata.' });
    }

    await prisma.user.update({
      where: { id: user.id },
      data: { emailVerified: new Date() },
    });
  }

  async requestPasswordReset(email: string): Promise<void> {
    const user = await prisma.user.findUnique({ where: { email } });
    if (!user) {
      // Non rivelare se l'email esiste per motivi di sicurezza
      console.log(`Password reset requested for non-existent email: ${email}`);
      return;
    }

    const resetToken = await generatePasswordResetToken(email);
    const resetUrl = `${process.env.AUTH_URL}/reset-password?token=${resetToken}`;

    await sendEmail({
      to: email,
      subject: "Reset della tua password per NextAuth App",
      html: `<p>Clicca sul link per resettare la tua password: <a href="${resetUrl}">Reset Password</a></p>`,
    });
  }

  async resetPassword(token: string, newPassword: string): Promise<void> {
    const tokenPayload = await verifyPasswordResetToken(token);
    if (!tokenPayload) {
      throw new TRPCError({ code: 'BAD_REQUEST', message: 'Token di reset password non valido o scaduto.' });
    }

    const user = await prisma.user.findUnique({ where: { email: tokenPayload.email } });
    if (!user) {
      throw new TRPCError({ code: 'NOT_FOUND', message: 'Utente non trovato.' });
    }

    const hashedPassword = await hashPassword(newPassword);

    await prisma.user.update({
      where: { id: user.id },
      data: { password: hashedPassword },
    });
  }

  async changePassword(userId: string, oldPassword: string, newPassword: string): Promise<void> {
    const user = await prisma.user.findUnique({ where: { id: userId } });
    if (!user || !user.password) {
      throw new TRPCError({ code: 'NOT_FOUND', message: 'Utente non trovato o non ha una password.' });
    }

    const isOldPasswordValid = await verifyPassword(oldPassword, user.password);
    if (!isOldPasswordValid) {
      throw new TRPCError({ code: 'UNAUTHORIZED', message: 'La vecchia password non è corretta.' });
    }

    const hashedPassword = await hashPassword(newPassword);
    await prisma.user.update({
      where: { id: userId },
      data: { password: hashedPassword },
    });
  }

  async updateProfile(userId: string, data: UpdateProfileInput): Promise<User> {
    const user = await prisma.user.findUnique({ where: { id: userId } });
    if (!user) {
      throw new TRPCError({ code: 'NOT_FOUND', message: 'Utente non trovato.' });
    }

    // Se l'email viene aggiornata, resetta emailVerified e invia nuova email di verifica
    if (data.email && data.email !== user.email) {
      const existingUserWithNewEmail = await prisma.user.findUnique({ where: { email: data.email } });
      if (existingUserWithNewEmail && existingUserWithNewEmail.id !== userId) {
        throw new TRPCError({ code: 'CONFLICT', message: 'Questa email è già in uso da un altro account.' });
      }

      const verificationToken = await generateVerificationToken(data.email);
      const verificationUrl = `${process.env.AUTH_URL}/verify-email?token=${verificationToken}`;
      await sendEmail({
        to: data.email,
        subject: "Verifica la tua nuova email per NextAuth App",
        html: `<p>Clicca sul link per verificare la tua nuova email: <a href="${verificationUrl}">Verifica Email</a></p>`,
      });

      return prisma.user.update({
        where: { id: userId },
        data: {
          ...data,
          emailVerified: null, // Resetta lo stato di verifica
        },
      });
    }

    return prisma.user.update({
      where: { id: userId },
      data: data,
    });
  }

  async deleteAccount(userId: string): Promise<void> {
    const user = await prisma.user.findUnique({ where: { id: userId } });
    if (!user) {
      throw new TRPCError({ code: 'NOT_FOUND', message: 'Utente non trovato.' });
    }

    await prisma.user.delete({
      where: { id: userId },
    });
  }
}

---

§ FILE 6: SRC/SERVER/TRPC/ROUTERS/AUTH.TS (150 RIGHE)

typescript
// src/server/trpc/routers/auth.ts
import { publicProcedure, protectedProcedure, router } from '@/server/trpc/trpc';
import { AuthService } from '@/server/services/auth-service';
import {
  registerSchema,
  loginSchema,
  forgotPasswordSchema,
  resetPasswordSchema,
  changePasswordSchema,
  updateProfileSchema,
} from '@/lib/validations/auth';
import { z } from 'zod';

const authService = new AuthService();

export const authRouter = router({
  register: publicProcedure
    .input(registerSchema)
    .mutation(async ({ input }) => {
      return authService.register(input);
    }),

  // Il login con credenziali è gestito principalmente tramite `signIn` di next-auth sul client.
  // Questa procedura tRPC è un esempio se volessi un'API di login custom.
  // Per questo setup, useremo `signIn` direttamente nel client.
  login: publicProcedure
    .input(loginSchema)
    .mutation(async ({ input }) => {
      // Questa procedura non userà signIn di next-auth, ma la logica interna.
      // Per un'integrazione completa con next-auth, il client chiamerà `signIn("credentials", ...)`
      // e non questa procedura tRPC per il login.
      return authService.login(input);
    }),

  requestPasswordReset: publicProcedure
    .input(forgotPasswordSchema)
    .mutation(async ({ input }) => {
      await authService.requestPasswordReset(input.email);
      return { message: 'Se l\'email esiste, un link per il reset è stato inviato.' };
    }),

  resetPassword: publicProcedure
    .input(resetPasswordSchema.extend({ token: z.string() }))
    .mutation(async ({ input }) => {
      await authService.resetPassword(input.token, input.password);
      return { message: 'La password è stata resettata con successo.' };
    }),

  verifyEmail: publicProcedure
    .input(z.object({ token: z.string() }))
    .mutation(async ({ input }) => {
      await authService.verifyEmail(input.token);
      return { message: 'La tua email è stata verificata con successo.' };
    }),

  changePassword: protectedProcedure
    .input(changePasswordSchema)
    .mutation(async ({ ctx, input }) => {
      if (!ctx.session?.user?.id) {
        throw new Error("ID utente non disponibile.");
      }
      await authService.changePassword(ctx.session.user.id, input.oldPassword, input.newPassword);
      return { message: 'La password è stata cambiata con successo.' };
    }),

  updateProfile: protectedProcedure
    .input(updateProfileSchema)
    .mutation(async ({ ctx, input }) => {
      if (!ctx.session?.user?.id) {
        throw new Error("ID utente non disponibile.");
      }
      const updatedUser = await authService.updateProfile(ctx.session.user.id, input);
      return updatedUser;
    }),

  deleteAccount: protectedProcedure
    .mutation(async ({ ctx }) => {
      if (!ctx.session?.user?.id) {
        throw new Error("ID utente non disponibile.");
      }
      await authService.deleteAccount(ctx.session.user.id);
      return { message: 'Il tuo account è stato eliminato con successo.' };
    }),

  // Esempio di procedura per ottenere i dati dell'utente autenticato
  getMe: protectedProcedure
    .query(async ({ ctx }) => {
      if (!ctx.session?.user?.id) {
        throw new Error("ID utente non disponibile.");
      }
      const user = await prisma.user.findUnique({
        where: { id: ctx.session.user.id },
        select: { id: true, name: true, email: true, image: true, role: true, emailVerified: true },
      });
      if (!user) {
        throw new Error("Utente non trovato.");
      }
      return user;
    }),
});

---

§ FILE 8: SRC/HOOKS/USE-AUTH.TS (60 RIGHE)

typescript
// src/hooks/use-auth.ts
import { useSession, signIn as nextAuthSignIn, signOut as nextAuthSignOut } from 'next-auth/react';
import { useRouter } from 'next/navigation';
import { trpc } from '@/app/_trpc/client'; // Assumi che il tuo client tRPC sia qui
import { LoginInput, RegisterInput } from '@/lib/validations/auth';
import { toast } from 'sonner'; // Esempio di libreria per notifiche

export function useAuth() {
  const { data: session, status, update } = useSession();
  const router = useRouter();

  const isAuthenticated = status === 'authenticated';
  const isLoading = status === 'loading';
  const user = session?.user;

  const registerMutation = trpc.auth.register.useMutation({
    onSuccess: () => {
      toast.success("Registrazione avvenuta con successo! Controlla la tua email per la verifica.");
      router.push('/login');
    },
    onError: (error) => {
      toast.error(error.message || "Errore durante la registrazione.");
    },
  });

  const login = async (data: LoginInput) => {
    try {
      const result = await nextAuthSignIn('credentials', {
        redirect: false,
        email: data.email,
        password: data.password,
      });

      if (result?.error) {
        toast.error(result.error);
        return false;
      }

      if (result?.ok) {
        toast.success("Login avvenuto con successo!");
        router.push('/dashboard');
        return true;
      }
      return false;
    } catch (error: any) {
      toast.error(error.message || "Errore durante il login.");
      return false;
    }
  };

  const logout = async () => {
    await nextAuthSignOut({ callbackUrl: '/' });
    toast.info("Logout effettuato.");
  };

  const signInWithGoogle = async () => {
    await nextAuthSignIn('google', { callbackUrl: '/dashboard' });
  };

  const signInWithGitHub = async () => {
    await nextAuthSignIn('github', { callbackUrl: '/dashboard' });
  };

  return {
    user,
    isLoading,
    isAuthenticated,
    login,
    logout,
    register: registerMutation.mutateAsync,
    isRegistering: registerMutation.isLoading,
    signInWithGoogle,
    signInWithGitHub,
    updateSession: update, // Espone la funzione update per aggiornare la sessione
  };
}

---

§ FILE 13: SRC/COMPONENTS/AUTH/OAUTH-BUTTONS.TSX (60 RIGHE)

typescript
// src/components/auth/oauth-buttons.tsx
'use client';

import { useAuth } from '@/hooks/use-auth';
import { FcGoogle } from 'react-icons/fc';
import { FaGithub } from 'react-icons/fa';
import { Button } from '@/components/ui/button'; // Assumi un componente Button di shadcn/ui o simile
import { Loader2 } from 'lucide-react'; // Assumi un'icona di loading

interface OAuthButtonsProps {
  isLoading?: boolean;
}

export function OAuthButtons({ isLoading = false }: OAuthButtonsProps) {
  const { signInWithGoogle, signInWithGitHub } = useAuth();

  const handleGoogleSignIn = async () => {
    if (!isLoading) {
      await signInWithGoogle();
    }
  };

  const handleGitHubSignIn = async () => {
    if (!isLoading) {
      await signInWithGitHub();
    }
  };

  return (
    <div className="flex flex-col space-y-3">
      <Button
        variant="outline"
        onClick={handleGoogleSignIn}
        disabled={isLoading}
        className="flex items-center justify-center gap-2"
      >
        {isLoading ? <Loader2 className="mr-2 h-4 w-4 animate-spin" /> : <FcGoogle className="h-5 w-5" />}
        Continua con Google
      </Button>
      <Button
        variant="outline"
        onClick={handleGitHubSignIn}
        disabled={isLoading}
        className="flex items-center justify-center gap-2"
      >
        {isLoading ? <Loader2 className="mr-2 h-4 w-4 animate-spin" /> : <FaGithub className="h-5 w-5" />}
        Continua con GitHub
      </Button>
    </div>
  );
}

---

§ FILE 9: SRC/COMPONENTS/AUTH/LOGIN-FORM.TSX (150 RIGHE)

typescript
// src/components/auth/login-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { loginSchema, LoginInput } from '@/lib/validations/auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'; // Assumi shadcn/ui Form
import Link from 'next/link';
import { OAuthButtons } from './oauth-buttons';
import { useAuth } from '@/hooks/use-auth';
import { Loader2 } from 'lucide-react';

export function LoginForm() {
  const { login, isLoading: isAuthLoading } = useAuth();

  const form = useForm<LoginInput>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      email: '',
      password: '',
    },
  });

  const onSubmit = async (data: LoginInput) => {
    await login(data);
  };

  const isSubmitting = form.formState.isSubmitting || isAuthLoading;

  return (
    <div className="space-y-6">
      <Form {...form}>
        <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
          <FormField
            control={form.control}
            name="email"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Email</FormLabel>
                <FormControl>
                  <Input
                    type="email"
                    placeholder="mario.rossi@example.com"
                    autoComplete="email"
                    disabled={isSubmitting}
                    {...field}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="password"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Password</FormLabel>
                <FormControl>
                  <Input
                    type="password"
                    placeholder="********"
                    autoComplete="current-password"
                    disabled={isSubmitting}
                    {...field}
                  />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <Button type="submit" className="w-full" disabled={isSubmitting}>
            {isSubmitting && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
            Accedi
          </Button>
        </form>
      </Form>

      <div className="relative">
        <div className="absolute inset-0 flex items-center">
          <span className="w-full border-t" />
        </div>
        <div className="relative flex justify-center text-xs uppercase">
          <span className="bg-background px-2 text-muted-foreground">O continua con</span>
        </div>
      </div>

      <OAuthButtons isLoading={isSubmitting} />

      <div className="text-center text-sm text-muted-foreground">
        <Link href="/forgot-password" className="underline-offset-4 hover:underline">
          Hai dimenticato la password?
        </Link>
      </div>
      <div className="text-center text-sm text-muted-foreground">
        Non hai un account?{' '}
        <Link href="/register" className="underline-offset-4 hover:underline">
          Registrati
        </Link>
      </div>
    </div>
  );
}

---

§ FILE 10: SRC/COMPONENTS/AUTH/REGISTER-FORM.TSX (180 RIGHE)

typescript
// src/components/auth/register-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { registerSchema, RegisterInput } from '@/lib/validations/auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Checkbox } from '@/components/ui/checkbox'; // Assumi shadcn/ui Checkbox
import Link from 'next/link';
import { OAuthButtons } from './oauth-buttons';
import { useAuth } from '@/hooks/use-auth';
import { Loader2 } from 'lucide-react';
import { validatePasswordStrength } from '@/lib/auth/password';
import { useEffect, useState } from 'react';

export function RegisterForm() {
  const { register, isRegistering } = useAuth();
  const [passwordStrengthErrors, setPasswordStrengthErrors] = useState<string[]>([]);

  const form = useForm<RegisterInput>({
    resolver: zodResolver(registerSchema),
    defaultValues: {
      name: '',
      email: '',
      password: '',
      confirmPassword: '',
      terms: false,
    },
  });

  const password = form.watch('password');

  useEffect(() => {
    if (password) {
      const { errors } = validatePasswordStrength(password);
      setPasswordStrengthErrors(errors);
    } else {
      setPasswordStrengthErrors([]);
    }
  }, [password]);


  const onSubmit = async (data: RegisterInput) => {
    await register(data);
  };

  const isSubmitting = form.formState.isSubmitting || isRegistering;

  return (
    <div className="space-y-6">
      <Form {...form}>
        <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
          <FormField
            control={form.control}
            name="name"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Nome</FormLabel>
                <FormControl>
                  <Input placeholder="Mario Rossi" disabled={isSubmitting} {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="email"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Email</FormLabel>
                <FormControl>
                  <Input type="email" placeholder="mario.rossi@example.com" disabled={isSubmitting} {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="password"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Password</FormLabel>
                <FormControl>
                  <Input type="password" placeholder="********" disabled={isSubmitting} {...field} />
                </FormControl>
                {passwordStrengthErrors.length > 0 && (
                  <ul className="text-sm text-muted-foreground list-disc pl-5">
                    {passwordStrengthErrors.map((error, index) => (
                      <li key={index} className="text-red-500">{error}</li>
                    ))}
                  </ul>
                )}
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="confirmPassword"
            render={({ field }) => (
              <FormItem>
                <FormLabel>Conferma Password</FormLabel>
                <FormControl>
                  <Input type="password" placeholder="********" disabled={isSubmitting} {...field} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
          <FormField
            control={form.control}
            name="terms"
            render={({ field }) => (
              <FormItem className="flex flex-row items-start space-x-3 space-y-0 rounded-md border p-4">
                <FormControl>
                  <Checkbox
                    checked={field.value}
                    onCheckedChange={field.onChange}
                    disabled={isSubmitting}
                  />
                </FormControl>
                <div className="space-y-1 leading-none">
                  <FormLabel>
                    Accetto i <Link href="/terms" className="underline">termini e condizioni</Link>
                  </FormLabel>
                  <FormMessage />
                </div>
              </FormItem>
            )}
          />
          <Button type="submit" className="w-full" disabled={isSubmitting}>
            {isSubmitting && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
            Registrati
          </Button>
        </form>
      </Form>

      <div className="relative">
        <div className="absolute inset-0 flex items-center">
          <span className="w-full border-t" />
        </div>
        <div className="relative flex justify-center text-xs uppercase">
          <span className="bg-background px-2 text-muted-foreground">O continua con</span>
        </div>
      </div>

      <OAuthButtons isLoading={isSubmitting} />

      <div className="text-center text-sm text-muted-foreground">
        Hai già un account?{' '}
        <Link href="/login" className="underline-offset-4 hover:underline">
          Accedi
        </Link>
      </div>
    </div>
  );
}

---

§ FILE 11: SRC/COMPONENTS/AUTH/FORGOT-PASSWORD-FORM.TSX (80 RIGHE)

typescript
// src/components/auth/forgot-password-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { forgotPasswordSchema, ForgotPasswordInput } from '@/lib/validations/auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { trpc } from '@/app/_trpc/client';
import { toast } from 'sonner';
import { Loader2 } from 'lucide-react';
import { useState } from 'react';

export function ForgotPasswordForm() {
  const [isSubmitted, setIsSubmitted] = useState(false);
  const forgotPasswordMutation = trpc.auth.requestPasswordReset.useMutation({
    onSuccess: () => {
      toast.success("Se l'email esiste, un link per il reset è stato inviato.");
      setIsSubmitted(true);
    },
    onError: (error) => {
      toast.error(error.message || "Errore durante la richiesta di reset password.");
    },
  });

  const form = useForm<ForgotPasswordInput>({
    resolver: zodResolver(forgotPasswordSchema),
    defaultValues: {
      email: '',
    },
  });

  const onSubmit = async (data: ForgotPasswordInput) => {
    await forgotPasswordMutation.mutateAsync(data);
  };

  const isSubmitting = form.formState.isSubmitting || forgotPasswordMutation.isLoading;

  if (isSubmitted) {
    return (
      <div className="text-center space-y-4">
        <h3 className="text-lg font-semibold">Link per il reset inviato!</h3>
        <p className="text-muted-foreground">
          Controlla la tua casella di posta (e la cartella spam) per un link per resettare la tua password.
        </p>
      </div>
    );
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input
                  type="email"
                  placeholder="mario.rossi@example.com"
                  autoComplete="email"
                  disabled={isSubmitting}
                  {...field}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit" className="w-full" disabled={isSubmitting}>
          {isSubmitting && <Loader2 className="mr-2 h-4 w-4 animate-spin" />}
          Invia link per il reset
        </Button>
      </form>
    </Form>
  );
}

---

§ FILE 12: SRC/COMPONENTS/AUTH/RESET-PASSWORD-FORM.TSX (100 RIGHE)

typescript
// src/components/auth/reset-password-form.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { resetPasswordSchema, ResetPasswordInput } from '@/lib/validations/auth';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { trpc } from '@/app/_trpc/client';
import { toast } from 'sonner';
import { Loader2 } from 'lucide-react';
import { useRouter, useSearchParams } from 'next/navigation';
import { useEffect, useState } from 'react';

export function ResetPasswordForm() {
  const router = useRouter();
  const searchParams = useSearchParams();
  const token = searchParams.get('token');
  const [resetSuccess, setResetSuccess] = useState(false);

  const resetPasswordMutation = trpc.auth.resetPassword.useMutation({
    onSuccess: () => {
      toast.success("La password è stata resettata con successo. Ora puoi accedere.");
      setResetSuccess(true);
      router.push('/login');
    },
    onError: (error) => {
      toast.error(error.message || "Errore durante il reset della password.");
    },
  });

  const form = useForm<ResetPasswordInput>({
    resolver: zodResolver(resetPasswordSchema),
    defaultValues: {
      password: '',
      confirmPassword: '',
    },
  });

  useEffect(() => {
    if (!token) {
      toast.error("Token di reset password mancante.");
      router.push('/forgot-password');
    }
  }, [token, router]);

  const onSubmit = async (data: ResetPasswordInput) => {
    if (!token) {
      toast.error("Token di reset password non valido.");
      return;
    }
    await resetPasswordMutation.mutateAsync({ token, password: data.password, confirmPassword: data.confirmPassword });
  };

  const isSubmitting = form.formState.isSubmitting || resetPasswordMutation.isLoading;

  if (resetSuccess) {
    return (
      <div className="text-center space-y-4">
        <h3 className="text-lg font-semibold">Password resettata!</h3>
        <p className="text-muted-foreground">
          La tua password è stata modificata con successo. Puoi accedere con la nuova password.
        </p>
        <Button onClick={() => router.push('/login')}>Vai al Login</Button>
      </div>
    );
  }

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nuova Password</FormLabel>
              <FormControl>
                <Input type="password" placeholder="********" disabled={isSubmitting} {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <FormField
          control={form.control}
          name="confirmPassword"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Conferma Nuova Password</FormLabel>
              <FormControl>
                <Input type="password" placeholder="********" disabled={isSubmitting} {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="

---
_Modello: gemini-2.5-flash (Google AI Studio)_
