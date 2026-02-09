# CATALOGO-TRPC-API-v1

> **Dominio**: API
> **Stack**: Next.js, React, TypeScript, Prisma, tRPC, Zod, Vitest
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [src/server/trpc/trpc.ts](#1-src-server-trpc-trpc-ts) | `src/server/trpc/trpc.ts` |
| 2 | [src/server/trpc/routers/_app.ts](#2-src-server-trpc-routers-_app-ts) | `src/server/trpc/routers/_app.ts` |
| 3 | [src/server/trpc/client.ts](#3-src-server-trpc-client-ts) | `src/server/trpc/client.ts` |
| 4 | [src/app/api/trpc/[trpc]/route.ts](#4-src-app-api-trpc-trpc-route-ts) | `src/app/api/trpc/[trpc]/route.ts` |
| 5 | [src/lib/api/errors.ts](#5-src-lib-api-errors-ts) | `src/lib/api/errors.ts` |
| 6 | [src/lib/api/response.ts](#6-src-lib-api-response-ts) | `src/lib/api/response.ts` |
| 7 | [src/lib/api/rate-limit.ts](#7-src-lib-api-rate-limit-ts) | `src/lib/api/rate-limit.ts` |
| 8 | [src/lib/api/validation.ts](#8-src-lib-api-validation-ts) | `src/lib/api/validation.ts` |
| 9 | [src/lib/api/fetch.ts](#9-src-lib-api-fetch-ts) | `src/lib/api/fetch.ts` |
| 10 | [src/hooks/use-api.ts](#10-src-hooks-use-api-ts) | `src/hooks/use-api.ts` |
| 11 | [src/server/middleware/index.ts](#11-src-server-middleware-index-ts) | `src/server/middleware/index.ts` |
| 12 | [tests/api.test.ts](#12-tests-api-test-ts) | `tests/api.test.ts` |

---

## 1. src/server/trpc/trpc.ts

```typescript
import { initTRPC, TRPCError } from "@trpc/server";
import { type CreateNextContextOptions } from "@trpc/server/adapters/next";
import superjson from "superjson";
import { ZodError } from "zod";
import { auth } from "@/auth"; // Presuppone l'esistenza di un modulo di autenticazione (es. NextAuth.js)
import { prisma } from "@/lib/prisma"; // Presuppone l'esistenza di un'istanza di Prisma Client
import {
  ApiError,
  AuthenticationError,
  AuthorizationError,
  handleApiError,
  RateLimitError,
  toTRPCError,
  ValidationError,
} from "@/lib/api/errors";
import { checkRateLimit } from "@/lib/api/rate-limit";
import { type Session } from "next-auth"; // Importa il tipo Session da next-auth

// Interfaccia per il contesto tRPC
interface CreateContextOptions {
  session: Session | null;
}

/**
 * Funzione per creare il contesto per le procedure tRPC.
 * Viene eseguita per ogni richiesta API.
 */
export const createTRPCContext = async (opts: CreateNextContextOptions) => {
  const { req } = opts;

  // Ottieni la sessione utente. Assicurati che il tuo `auth` restituisca una Promise<Session | null>.
  const session = await auth();

  // Log della sessione per debug (opzionale)
  // console.log("tRPC Context Session:", session?.user?.email);

  return {
    session,
    prisma, // Inietta l'istanza di Prisma Client nel contesto
    req, // Inietta l'oggetto request per accesso diretto se necessario (es. headers)
  };
};

/**
 * Inizializza tRPC con il contesto e il trasformatore.
 * Il trasformatore `superjson` permette di serializzare tipi complessi come Date, Map, Set.
 */
const t = initTRPC.context<typeof createTRPCContext>().create({
  transformer: superjson,
  /**
   * Formatta gli errori per renderli più leggibili e gestibili lato client.
   * Converte gli errori Zod e le nostre `ApiError` custom in `TRPCError`.
   */
  errorFormatter({ shape, error }) {
    // Gestione degli errori di validazione Zod
    if (error.cause instanceof ZodError) {
      return {
        ...shape,
        data: {
          ...shape.data,
          zodError: error.cause.flatten(), // Dettagli specifici dell'errore Zod
          code: "BAD_REQUEST", // Codice HTTP 400
          message: "Errore di validazione dei dati.",
        },
      };
    }

    // Gestione delle nostre `ApiError` custom
    if (error.cause instanceof ApiError) {
      const trpcError = toTRPCError(error.cause);
      return {
        ...shape,
        code: trpcError.code,
        message: trpcError.message,
        data: {
          ...shape.data,
          code: trpcError.code,
          httpStatus: trpcError.data?.httpStatus,
          details: error.cause.details,
        },
      };
    }

    // Per tutti gli altri errori, usa il formato di default di tRPC
    return {
      ...shape,
      data: {
        ...shape.data,
        code: shape.code,
        message: error.message,
      },
    };
  },
});

/**
 * Esporta il router tRPC per combinare le procedure.
 */
export const createTRPCRouter = t.router;

/**
 * Procedura pubblica: accessibile a tutti, autenticati o meno.
 */
export const publicProcedure = t.procedure;

/**
 * Middleware per l'autenticazione: assicura che l'utente sia loggato.
 */
const enforceUserIsAuthed = t.middleware(({ ctx, next }) => {
  if (!ctx.session || !ctx.session.user) {
    throw new TRPCError({
      code: "UNAUTHORIZED",
      message: "Non autenticato. Effettua il login per accedere.",
      cause: new AuthenticationError("Non autenticato."),
    });
  }
  return next({
    ctx: {
      // Inietta l'utente autenticato nel contesto per le procedure successive
      session: { ...ctx.session, user: ctx.session.user },
    },
  });
});

/**
 * Procedura protetta: richiede che l'utente sia autenticato.
 */
export const protectedProcedure = t.procedure.use(enforceUserIsAuthed);

/**
 * Middleware per l'autorizzazione: assicura che l'utente abbia il ruolo di 'ADMIN'.
 * Richiede che `enforceUserIsAuthed` sia già stato eseguito.
 */
const enforceUserIsAdmin = t.middleware(({ ctx, next }) => {
  // `ctx.session.user` è garantito esistere grazie a `enforceUserIsAuthed`
  if (!ctx.session?.user || ctx.session.user.role !== "ADMIN") {
    throw new TRPCError({
      code: "FORBIDDEN",
      message: "Non autorizzato. Accesso riservato agli amministratori.",
      cause: new AuthorizationError("Accesso negato. Ruolo insufficiente."),
    });
  }
  return next({
    ctx: {
      // Inietta l'utente admin nel contesto
      session: { ...ctx.session, user: ctx.session.user },
    },
  });
});

/**
 * Procedura admin: richiede che l'utente sia autenticato e abbia il ruolo 'ADMIN'.
 */
export const adminProcedure = t.procedure.use(enforceUserIsAuthed).use(enforceUserIsAdmin);

/**
 * Middleware per il rate limiting.
 * Applica un limite di richieste basato sull'IP del client.
 */
const rateLimitMiddleware = t.middleware(async ({ ctx, next, path }) => {
  const ip = ctx.req.headers["x-forwarded-for"] || ctx.req.socket.remoteAddress || "127.0.0.1";
  const identifier = `trpc_rate_limit_${ip}_${path}`; // Identificatore unico per il rate limiting

  const { success, remaining } = await checkRateLimit(identifier);

  if (!success) {
    throw new TRPCError({
      code: "TOO_MANY_REQUESTS",
      message: "Troppe richieste. Riprova più tardi.",
      cause: new RateLimitError("Rate limit exceeded."),
    });
  }

  // Puoi aggiungere headers di rate limiting alla risposta se necessario
  // ctx.res.setHeader("X-RateLimit-Remaining", remaining.toString());
  // ctx.res.setHeader("X-RateLimit-Limit", "10"); // Esempio, dovrebbe essere dinamico

  return next();
});

/**
 * Procedura con rate limiting: applica il limite di richieste.
 * Può essere combinata con altre procedure (es. `publicProcedure.use(rateLimitMiddleware)`).
 */
export const rateLimitedProcedure = t.procedure.use(rateLimitMiddleware);

/**
 * Middleware per il logging delle richieste.
 * Utile per il debug e il monitoraggio.
 */
const loggingMiddleware = t.middleware(async ({ path, type, next }) => {
  const start = Date.now();
  const result = await next();
  const durationMs = Date.now() - start;
  console.log(`[tRPC] ${type} ${path} - ${durationMs}ms - ${result.ok ? "OK" : "ERROR"}`);
  return result;
});

/**
 * Procedura con logging: logga ogni richiesta.
 */
export const loggedProcedure = t.procedure.use(loggingMiddleware);

// Puoi combinare i middleware come preferisci, ad esempio:
// export const protectedAndLoggedProcedure = protectedProcedure.use(loggingMiddleware);
```

---

## 2. src/server/trpc/routers/_app.ts

```typescript
import { createTRPCRouter } from "@/server/trpc/trpc";
import { authRouter } from "./auth"; // Presuppone l'esistenza di un router auth
import { userRouter } from "./user"; // Presuppone l'esistenza di un router user
import { productRouter } from "./product"; // Esempio di un altro router
import { orderRouter } from "./order"; // Esempio di un altro router

/**
 * Router di esempio per l'autenticazione.
 * Normalmente conterrebbe procedure per login, registrazione, ecc.
 */
export const authRouter = createTRPCRouter({
  // Esempio di procedura pubblica per il login
  login: publicProcedure
    .input(z.object({ email: z.string().email(), password: z.string().min(6) }))
    .mutation(async ({ input }) => {
      // Logica di login...
      console.log("Login attempt for:", input.email);
      return { message: "Login successful (mock)" };
    }),
  // Esempio di procedura protetta per ottenere la sessione
  getSession: protectedProcedure.query(async ({ ctx }) => {
    return ctx.session;
  }),
});

/**
 * Router di esempio per gli utenti.
 * Normalmente conterrebbe procedure per CRUD utenti.
 */
export const userRouter = createTRPCRouter({
  // Esempio di procedura protetta per ottenere l'utente corrente
  me: protectedProcedure.query(async ({ ctx }) => {
    // Qui potresti recuperare i dettagli completi dell'utente dal DB usando ctx.prisma
    return ctx.session.user;
  }),
  // Esempio di procedura admin per listare tutti gli utenti
  listUsers: adminProcedure.query(async ({ ctx }) => {
    const users = await ctx.prisma.user.findMany({
      select: { id: true, name: true, email: true, role: true, createdAt: true },
    });
    return users;
  }),
});

/**
 * Router di esempio per i prodotti.
 */
export const productRouter = createTRPCRouter({
  // Esempio di procedura pubblica per ottenere un prodotto per ID
  getById: publicProcedure
    .input(z.object({ id: z.string().cuid() }))
    .query(async ({ input, ctx }) => {
      const product = await ctx.prisma.product.findUnique({ where: { id: input.id } });
      if (!product) {
        throw new TRPCError({ code: "NOT_FOUND", message: "Prodotto non trovato." });
      }
      return product;
    }),
  // Esempio di procedura protetta per creare un nuovo prodotto
  create: protectedProcedure
    .input(z.object({ name: z.string().min(3), price: z.number().positive() }))
    .mutation(async ({ input, ctx }) => {
      const newProduct = await ctx.prisma.product.create({ data: input });
      return newProduct;
    }),
});

/**
 * Router di esempio per gli ordini.
 */
export const orderRouter = createTRPCRouter({
  // Esempio di procedura protetta per ottenere gli ordini dell'utente corrente
  myOrders: protectedProcedure.query(async ({ ctx }) => {
    const orders = await ctx.prisma.order.findMany({
      where: { userId: ctx.session.user.id },
      include: { items: true },
    });
    return orders;
  }),
});

// Importa Zod per i router di esempio
import { z } from "zod";
import { TRPCError } from "@trpc/server";

/**
 * Router radice che combina tutti i sub-routers.
 * Questo è il punto di ingresso principale per la tua API tRPC.
 */
export const appRouter = createTRPCRouter({
  auth: authRouter,
  user: userRouter,
  product: productRouter,
  order: orderRouter,
  // ... altri routers che potresti aggiungere
});

// Esporta il tipo del router radice per l'uso lato client
export type AppRouter = typeof appRouter;
```

---

## 3. src/server/trpc/client.ts

```typescript
// Client-side tRPC setup
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { httpBatchLink, loggerLink } from "@trpc/client";
import { createTRPCReact } from "@trpc/react-query";
import React, { useState } from "react";
import superjson from "superjson";
import { type AppRouter } from "./routers/_app";

/**
 * Inizializza il client tRPC React.
 * Questo hook ti permette di interagire con la tua API tRPC da componenti React.
 */
export const trpc = createTRPCReact<AppRouter>();

/**
 * Componente provider per tRPC e React Query.
 * Avvolge la tua applicazione per fornire accesso al client tRPC e alla cache di React Query.
 */
export function TRPCProvider({ children }: { children: React.ReactNode }) {
  // Stato per mantenere un'unica istanza di QueryClient e tRPC client per tutta la durata dell'app
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            // Configurazione di default per le query (es. tempo di validità, retry)
            staleTime: 5 * 60 * 1000, // 5 minuti
            refetchOnWindowFocus: false, // Non refetchare automaticamente al focus della finestra
            retry: 1, // Riprova una volta in caso di errore
          },
        },
      })
  );

  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        // Link per il logging delle richieste tRPC (solo in sviluppo)
        loggerLink({
          enabled: (opts) =>
            process.env.NODE_ENV === "development" ||
            (opts.direction === "down" && opts.result instanceof Error),
        }),
        // Link per il batching HTTP delle richieste tRPC
        httpBatchLink({
          url: getBaseUrl() + "/api/trpc", // URL del tuo endpoint tRPC
          // Puoi aggiungere headers qui, ad esempio per l'autenticazione
          // headers() {
          //   const token = localStorage.getItem("authToken");
          //   return token ? { Authorization: `Bearer ${token}` } : {};
          // },
        }),
      ],
      transformer: superjson, // Usa superjson per la serializzazione/deserializzazione
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
    </trpc.Provider>
  );
}

/**
 * Funzione helper per ottenere l'URL base dell'API.
 * Gestisce ambienti di sviluppo e produzione.
 */
function getBaseUrl() {
  if (typeof window !== "undefined") return ""; // Browser
  if (process.env.VERCEL_URL) return `https://${process.env.VERCEL_URL}`; // Vercel
  // Fallback per sviluppo locale
  return `http://localhost:${process.env.PORT ?? 3000}`;
}
```

---

## 4. src/app/api/trpc/[trpc]/route.ts

```typescript
// Next.js API route handler per tRPC (App Router)
import { fetchRequestHandler } from "@trpc/server/adapters/fetch";
import { appRouter } from "@/server/trpc/routers/_app";
import { createTRPCContext } from "@/server/trpc/trpc";

/**
 * Gestore principale per tutte le richieste tRPC.
 * Utilizza `fetchRequestHandler` per adattare le richieste Fetch API a tRPC.
 */
const handler = (req: Request) =>
  fetchRequestHandler({
    endpoint: "/api/trpc", // L'endpoint tRPC che hai configurato
    req, // L'oggetto Request di Next.js
    router: appRouter, // Il tuo router tRPC radice
    createContext: createTRPCContext, // La funzione per creare il contesto tRPC
    onError:
      process.env.NODE_ENV === "development"
        ? ({ path, error }) => {
            console.error(`❌ tRPC failed on ${path ?? "<no-path>"}: ${error.message}`);
          }
        : undefined,
  });

// Esporta il gestore per i metodi GET e POST.
// tRPC utilizza principalmente POST per le mutazioni e GET per le query.
export { handler as GET, handler as POST };
```

---

## 5. src/lib/api/errors.ts

```typescript
import { TRPCError } from "@trpc/server";
import { ZodError } from "zod";

/**
 * Classe base per gli errori API personalizzati.
 * Permette di standardizzare la gestione degli errori in tutta l'applicazione.
 */
export class ApiError extends Error {
  constructor(
    message: string,
    public code: string, // Codice interno dell'errore (es. 'VALIDATION_FAILED')
    public statusCode: number, // Codice di stato HTTP (es. 400, 401)
    public details?: unknown // Dettagli aggiuntivi sull'errore (es. errori di validazione Zod)
  ) {
    super(message);
    this.name = "ApiError";
    // Mantiene lo stack trace corretto in V8
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ApiError);
    }
  }
}

/**
 * Errore di validazione dei dati (HTTP 400 Bad Request).
 * Utilizzato quando l'input non rispetta lo schema atteso.
 */
export class ValidationError extends ApiError {
  constructor(message: string = "Errore di validazione dei dati.", details?: unknown) {
    super(message, "VALIDATION_FAILED", 400, details);
    this.name = "ValidationError";
  }
}

/**
 * Errore di autenticazione (HTTP 401 Unauthorized).
 * Utilizzato quando l'utente non è autenticato.
 */
export class AuthenticationError extends ApiError {
  constructor(message: string = "Non autenticato. Effettua il login per accedere.") {
    super(message, "UNAUTHENTICATED", 401);
    this.name = "AuthenticationError";
  }
}

/**
 * Errore di autorizzazione (HTTP 403 Forbidden).
 * Utilizzato quando l'utente è autenticato ma non ha i permessi per accedere.
 */
export class AuthorizationError extends ApiError {
  constructor(message: string = "Non autorizzato. Accesso negato.") {
    super(message, "FORBIDDEN", 403);
    this.name = "AuthorizationError";
  }
}

/**
 * Errore risorsa non trovata (HTTP 404 Not Found).
 * Utilizzato quando la risorsa richiesta non esiste.
 */
export class NotFoundError extends ApiError {
  constructor(message: string = "Risorsa non trovata.") {
    super(message, "NOT_FOUND", 404);
    this.name = "NotFoundError";
  }
}

/**
 * Errore di conflitto (HTTP 409 Conflict).
 * Utilizzato quando una richiesta entra in conflitto con lo stato corrente del server (es. risorsa duplicata).
 */
export class ConflictError extends ApiError {
  constructor(message: string = "Conflitto. La risorsa esiste già o non può essere modificata.") {
    super(message, "CONFLICT", 409);
    this.name = "ConflictError";
  }
}

/**
 * Errore di rate limiting (HTTP 429 Too Many Requests).
 * Utilizzato quando l'utente ha superato il limite di richieste consentite.
 */
export class RateLimitError extends ApiError {
  constructor(message: string = "Troppe richieste. Riprova più tardi.") {
    super(message, "TOO_MANY_REQUESTS", 429);
    this.name = "RateLimitError";
  }
}

/**
 * Errore generico del server (HTTP 500 Internal Server Error).
 * Utilizzato per errori inaspettati.
 */
export class InternalServerError extends ApiError {
  constructor(message: string = "Errore interno del server. Riprova più tardi.") {
    super(message, "INTERNAL_SERVER_ERROR", 500);
    this.name = "InternalServerError";
  }
}

/**
 * Funzione per gestire errori sconosciuti e convertirli in `ApiError`.
 * Utile per catturare errori generici e dar loro un formato standard.
 */
export function handleApiError(error: unknown): ApiError {
  if (error instanceof ApiError) {
    return error;
  }
  if (error instanceof ZodError) {
    return new ValidationError("Errore di validazione dei dati.", error.flatten());
  }
  if (error instanceof TRPCError) {
    // Se è già un TRPCError, prova a mappare il suo codice a un ApiError
    switch (error.code) {
      case "UNAUTHORIZED":
        return new AuthenticationError(error.message);
      case "FORBIDDEN":
        return new AuthorizationError(error.message);
      case "NOT_FOUND":
        return new NotFoundError(error.message);
      case "BAD_REQUEST":
        return new ValidationError(error.message, error.cause);
      case "CONFLICT":
        return new ConflictError(error.message);
      case "TOO_MANY_REQUESTS":
        return new RateLimitError(error.message);
      default:
        return new InternalServerError(error.message || "Errore sconosciuto dal server.");
    }
  }
  if (error instanceof Error) {
    console.error("Errore non gestito:", error);
    return new InternalServerError(error.message || "Si è verificato un errore inaspettato.");
  }
  console.error("Errore di tipo sconosciuto:", error);
  return new InternalServerError("Si è verificato un errore sconosciuto.");
}

/**
 * Converte una `ApiError` personalizzata in una `TRPCError`.
 * Questo è fondamentale per la gestione degli errori all'interno del framework tRPC.
 */
export function toTRPCError(error: ApiError): TRPCError {
  let trpcCode: TRPCError["code"];

  switch (error.statusCode) {
    case 400:
      trpcCode = "BAD_REQUEST";
      break;
    case 401:
      trpcCode = "UNAUTHORIZED";
      break;
    case 403:
      trpcCode = "FORBIDDEN";
      break;
    case 404:
      trpcCode = "NOT_FOUND";
      break;
    case 409:
      trpcCode = "CONFLICT";
      break;
    case 429:
      trpcCode = "TOO_MANY_REQUESTS";
      break;
    case 500:
      trpcCode = "INTERNAL_SERVER_ERROR";
      break;
    default:
      trpcCode = "INTERNAL_SERVER_ERROR";
  }

  return new TRPCError({
    code: trpcCode,
    message: error.message,
    cause: error, // Mantiene l'errore originale come causa
    // Puoi aggiungere dati extra per il client, come lo statusCode HTTP
    // Questo sarà disponibile in `error.data.httpStatus` lato client
    // e `error.data.code` sarà il codice tRPC (es. 'BAD_REQUEST')
    data: {
      httpStatus: error.statusCode,
      code: error.code, // Il nostro codice interno (es. 'VALIDATION_FAILED')
      details: error.details,
    },
  });
}
```

---

## 6. src/lib/api/response.ts

```typescript
import { ApiError, InternalServerError } from "./errors";

/**
 * Interfaccia per i metadati di paginazione.
 */
export interface PaginationMeta {
  totalItems: number;
  itemCount: number;
  itemsPerPage: number;
  totalPages: number;
  currentPage: number;
}

/**
 * Interfaccia standardizzata per tutte le risposte API.
 * Fornisce un formato coerente per successi ed errori.
 */
export interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string; // Codice dell'errore (es. 'VALIDATION_FAILED')
    message: string; // Messaggio leggibile dall'utente
    statusCode: number; // Codice di stato HTTP
    details?: unknown; // Dettagli aggiuntivi (es. errori di validazione)
  };
  meta?: {
    pagination?: PaginationMeta;
    timestamp: string; // Timestamp della risposta
    requestId?: string; // ID univoco della richiesta per il tracing (opzionale)
    [key: string]: unknown; // Proprietà meta aggiuntive
  };
}

/**
 * Genera una risposta API di successo.
 * @param data I dati da restituire.
 * @param meta Metadati aggiuntivi (es. paginazione).
 * @returns Un oggetto ApiResponse con `success: true`.
 */
export function successResponse<T>(data: T, meta?: object): ApiResponse<T> {
  return {
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      // requestId: generateRequestId(), // Implementa una funzione per generare un ID univoco
      ...meta,
    },
  };
}

/**
 * Genera una risposta API di errore.
 * @param error L'oggetto ApiError da cui generare la risposta.
 * @returns Un oggetto ApiResponse con `success: false` e i dettagli dell'errore.
 */
export function errorResponse(error: ApiError): ApiResponse<never> {
  // Assicurati che l'errore sia un'istanza di ApiError, altrimenti usa InternalServerError
  const apiError = error instanceof ApiError ? error : new InternalServerError(error.message);

  return {
    success: false,
    error: {
      code: apiError.code,
      message: apiError.message,
      statusCode: apiError.statusCode,
      details: apiError.details,
    },
    meta: {
      timestamp: new Date().toISOString(),
      // requestId: generateRequestId(),
    },
  };
}

/**
 * Genera una risposta API di successo con metadati di paginazione.
 * @param data L'array di elementi paginati.
 * @param pagination I metadati di paginazione.
 * @param additionalMeta Metadati aggiuntivi da includere.
 * @returns Un oggetto ApiResponse con `success: true` e i dati paginati.
 */
export function paginatedResponse<T>(
  data: T[],
  pagination: PaginationMeta,
  additionalMeta?: object
): ApiResponse<T[]> {
  return {
    success: true,
    data,
    meta: {
      timestamp: new Date().toISOString(),
      pagination,
      ...additionalMeta,
    },
  };
}

// Funzione helper per generare un ID di richiesta (esempio)
// function generateRequestId(): string {
//   return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
// }
```

---

## 7. src/lib/api/rate-limit.ts

```typescript
// Rate limiting utilities
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
import { TRPCError } from "@trpc/server";
import { RateLimitError } from "./errors";

// Inizializza il client Redis per Upstash.
// Assicurati che le variabili d'ambiente UPSTASH_REDIS_REST_URL e UPSTASH_REDIS_REST_TOKEN siano configurate.
export const redis = Redis.fromEnv();

/**
 * Istanza di Ratelimit configurata con Redis.
 * Limita 10 richieste ogni 10 secondi per identificatore.
 * `analytics: true` abilita la raccolta di metriche su Upstash.
 */
export const ratelimit = new Ratelimit({
  redis: redis,
  limiter: Ratelimit.slidingWindow(10, "10 s"), // 10 richieste ogni 10 secondi
  analytics: true,
  /**
   * La funzione `ephemeralCache` è utile per ridurre il numero di chiamate a Redis
   * per identificatori che superano rapidamente il limite.
   * In questo caso, usiamo una cache in-memory per 1 secondo.
   */
  ephemeralCache: new Map(),
});

/**
 * Controlla il limite di richieste per un dato identificatore.
 * @param identifier L'identificatore unico per il rate limiting (es. IP, ID utente).
 * @returns Un oggetto con `success` (true se la richiesta è consentita) e `remaining` (richieste rimanenti).
 */
export async function checkRateLimit(
  identifier: string
): Promise<{ success: boolean; remaining: number }> {
  const { success, limit, reset, remaining } = await ratelimit.limit(identifier);

  // Puoi loggare i dettagli del rate limit per debug
  // console.log(`Rate Limit for ${identifier}: Success=${success}, Remaining=${remaining}/${limit}, Reset=${new Date(reset * 1000).toISOString()}`);

  return { success, remaining };
}

/**
 * Funzione helper per creare un middleware di rate limiting per tRPC.
 * Questo è un esempio di come potresti incapsulare la logica di rate limiting.
 *
 * @param limit Il numero massimo di richieste consentite.
 * @param window La finestra di tempo (es. "10s", "1m", "1h").
 * @returns Un middleware tRPC che applica il rate limiting.
 */
export function createTRPCRateLimitMiddleware(limit: number, window: string) {
  const customRatelimit = new Ratelimit({
    redis: redis,
    limiter: Ratelimit.slidingWindow(limit, window),
    analytics: true,
    ephemeralCache: new Map(),
  });

  return t.middleware(async ({ ctx, next, path }) => {
    const ip = ctx.req.headers["x-forwarded-for"] || ctx.req.socket.remoteAddress || "127.0.0.1";
    const identifier = `trpc_custom_rate_limit_${ip}_${path}`;

    const { success, remaining } = await customRatelimit.limit(identifier);

    if (!success) {
      throw new TRPCError({
        code: "TOO_MANY_REQUESTS",
        message: `Troppe richieste. Riprova tra ${Math.ceil((customRatelimit.get and customRatelimit.get(identifier)?.reset || 0) - Date.now() / 1000)} secondi.`,
        cause: new RateLimitError("Rate limit exceeded."),
      });
    }

    return next();
  });
}

// Importa t per il middleware (necessario per createTRPCRateLimitMiddleware)
import { t } from "@/server/trpc/trpc"; // Assicurati che `t` sia esportato da trpc.ts
```

---

## 8. src/lib/api/validation.ts

```typescript
// Validation utilities
import { z, ZodError } from "zod";
import { ValidationError } from "./errors";
import { FieldValues, UseFormProps, useForm } from "react-hook-form";
import { zodResolver as rhfZodResolver } from "@hookform/resolvers/zod";

/**
 * Schema per un ID CUID (Collision-resistant Unique Identifier).
 * Utile per identificatori generati lato server.
 */
export const idSchema = z.string().cuid("ID non valido.");

/**
 * Schema per un indirizzo email.
 */
export const emailSchema = z.string().email("Indirizzo email non valido.");

/**
 * Schema per una password robusta.
 * Richiede almeno 8 caratteri, una maiuscola, una minuscola, un numero e un carattere speciale.
 */
export const passwordSchema = z
  .string()
  .min(8, "La password deve contenere almeno 8 caratteri.")
  .regex(/[A-Z]/, "La password deve contenere almeno una lettera maiuscola.")
  .regex(/[a-z]/, "La password deve contenere almeno una lettera minuscola.")
  .regex(/[0-9]/, "La password deve contenere almeno un numero.")
  .regex(/[^a-zA-Z0-9]/, "La password deve contenere almeno un carattere speciale.");

/**
 * Schema per uno slug URL-friendly.
 * Permette solo lettere minuscole, numeri e trattini.
 */
export const slugSchema = z
  .string()
  .min(1, "Lo slug non può essere vuoto.")
  .regex(/^[a-z0-9-]+$/, "Lo slug può contenere solo lettere minuscole, numeri e trattini.")
  .transform((val) => val.toLowerCase()); // Assicura che lo slug sia sempre minuscolo

/**
 * Schema per la paginazione standard.
 * Include `page`, `pageSize`, `sortBy` e `sortOrder`.
 */
export const paginationSchema = z.object({
  page: z.coerce // Tenta di convertire a numero
    .number({ invalid_type_error: "Il numero di pagina deve essere un numero." })
    .int("Il numero di pagina deve essere un intero.")
    .positive("Il numero di pagina deve essere positivo.")
    .default(1),
  pageSize: z.coerce
    .number({ invalid_type_error: "La dimensione della pagina deve essere un numero." })
    .int("La dimensione della pagina deve essere un intero.")
    .min(1, "La dimensione della pagina deve essere almeno 1.")
    .max(100, "La dimensione della pagina non può superare 100.")
    .default(10),
  sortBy: z.string().optional(),
  sortOrder: z.enum(["asc", "desc"], { invalid_type_error: "L'ordine di ordinamento deve essere 'asc' o 'desc'." }).default("desc"),
});

/**
 * Funzione helper per validare i dati contro uno schema Zod.
 * Lancia un `ValidationError` se la validazione fallisce.
 * @param schema Lo schema Zod da usare per la validazione.
 * @param data I dati da validare.
 * @returns I dati validati e tipizzati.
 */
export function validate<T>(schema: z.ZodSchema<T>, data: unknown): T {
  try {
    return schema.parse(data);
  } catch (error) {
    if (error instanceof ZodError) {
      throw new ValidationError("Errore di validazione dei dati.", error.flatten());
    }
    throw new ValidationError("Errore di validazione sconosciuto.", error);
  }
}

/**
 * Wrapper per `zodResolver` di `@hookform/resolvers/zod`.
 * Fornisce un'integrazione semplificata con `react-hook-form` e Zod.
 * @param schema Lo schema Zod da usare per la validazione del form.
 * @returns Un resolver compatibile con `react-hook-form`.
 */
export function zodResolver<T extends z.ZodSchema<any>>(schema: T) {
  return rhfZodResolver(schema);
}

/**
 * Hook personalizzato per l'integrazione di form con Zod e React Hook Form.
 * @param schema Lo schema Zod per il form.
 * @param options Opzioni per `useForm`.
 * @returns L'oggetto restituito da `useForm`.
 */
export function useZodForm<T extends z.ZodSchema<any>>(
  schema: T,
  options?: UseFormProps<z.infer<T>>
) {
  return useForm<z.infer<T>>({
    ...options,
    resolver: zodResolver(schema),
  });
}
```

---

## 9. src/lib/api/fetch.ts

```typescript
import { ApiError, InternalServerError } from "./errors";

/**
 * Opzioni estese per la funzione `apiFetch`.
 * Permette di aggiungere parametri URL e un timeout.
 */
export interface FetchOptions extends RequestInit {
  params?: Record<string, string | number | boolean | undefined>;
  timeout?: number; // Timeout in millisecondi
  baseUrl?: string; // URL base per la richiesta
}

/**
 * Wrapper type-safe attorno all'API Fetch nativa.
 * Gestisce la costruzione dell'URL, il timeout, la gestione degli errori e la deserializzazione JSON.
 * @param url L'URL della risorsa da recuperare.
 * @param options Opzioni per la richiesta Fetch.
 * @returns Una Promise che si risolve con i dati tipizzati della risposta.
 * @throws ApiError in caso di errore HTTP o di rete.
 */
export async function apiFetch<T>(url: string, options?: FetchOptions): Promise<T> {
  const { params, timeout, baseUrl = "" /* process.env.NEXT_PUBLIC_API_URL */, ...fetchInit } =
    options || {};

  // Costruisci l'URL con i parametri di query
  const urlObj = new URL(url, baseUrl);
  if (params) {
    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined && value !== null) {
        urlObj.searchParams.append(key, String(value));
      }
    });
  }

  const controller = new AbortController();
  const timeoutId = timeout ? setTimeout(() => controller.abort(), timeout) : null;

  try {
    const response = await fetch(urlObj.toString(), {
      ...fetchInit,
      signal: controller.signal,
      headers: {
        "Content-Type": "application/json",
        ...fetchInit.headers,
      },
    });

    if (timeoutId) clearTimeout(timeoutId);

    // Gestione degli errori HTTP
    if (!response.ok) {
      let errorData: any;
      try {
        errorData = await response.json();
      } catch (jsonError) {
        // Se la risposta non è JSON, usa il testo dello stato
        errorData = { message: response.statusText || `Errore HTTP ${response.status}` };
      }

      // Prova a mappare l'errore a una ApiError custom
      throw new ApiError(
        errorData.message || `Errore API: ${response.status}`,
        errorData.code || "API_ERROR",
        response.status,
        errorData.details
      );
    }

    // Se la risposta è 204 No Content, restituisci undefined o un oggetto vuoto
    if (response.status === 204) {
      return undefined as T;
    }

    // Deserializza la risposta JSON
    return (await response.json()) as T;
  } catch (error) {
    if (timeoutId) clearTimeout(timeoutId);
    if (error instanceof DOMException && error.name === "AbortError") {
      throw new ApiError("La richiesta è scaduta.", "REQUEST_TIMEOUT", 408);
    }
    // Rilancia ApiError già gestiti, altrimenti avvolgi in InternalServerError
    if (error instanceof ApiError) {
      throw error;
    }
    console.error("Errore in apiFetch:", error);
    throw new InternalServerError("Si è verificato un errore di rete o un errore inatteso.");
  }
}

/**
 * Oggetto `api` che fornisce metodi specializzati per i verbi HTTP comuni.
 * Semplifica l'esecuzione di richieste GET, POST, PUT, PATCH, DELETE.
 */
export const api = {
  get: <T>(url: string, options?: FetchOptions) =>
    apiFetch<T>(url, { ...options, method: "GET" }),

  post: <T>(url: string, data: unknown, options?: FetchOptions) =>
    apiFetch<T>(url, { ...options, method: "POST", body: JSON.stringify(data) }),

  put: <T>(url: string, data: unknown, options?: FetchOptions) =>
    apiFetch<T>(url, { ...options, method: "PUT", body: JSON.stringify(data) }),

  patch: <T>(url: string, data: unknown, options?: FetchOptions) =>
    apiFetch<T>(url, { ...options, method: "PATCH", body: JSON.stringify(data) }),

  delete: <T>(url: string, options?: FetchOptions) =>
    apiFetch<T>(url, { ...options, method: "DELETE" }),
};
```

---

## 10. src/hooks/use-api.ts

```typescript
import {
  useQuery,
  useMutation,
  useInfiniteQuery,
  type UseQueryOptions,
  type UseQueryResult,
  type UseMutationOptions,
  type UseMutationResult,
  type UseInfiniteQueryOptions,
  type UseInfiniteQueryResult,
  QueryKey,
} from "@tanstack/react-query";
import { type PaginationMeta } from "@/lib/api/response";

/**
 * Interfaccia per i risultati paginati.
 * Utile per le query infinite.
 */
export interface PaginatedResult<T> {
  data: T[];
  meta: {
    pagination: PaginationMeta;
    [key: string]: unknown;
  };
}

/**
 * Hook React per eseguire query API con `@tanstack/react-query`.
 * Fornisce un wrapper tipizzato e standardizzato per `useQuery`.
 *
 * @param key La chiave della query per la cache di React Query.
 * @param fetcher La funzione che esegue la chiamata API e restituisce i dati.
 * @param options Opzioni aggiuntive per `useQuery`.
 * @returns Il risultato della query.
 */
export function useApiQuery<TData, TError = Error>(
  key: QueryKey,
  fetcher: () => Promise<TData>,
  options?: Omit<UseQueryOptions<TData, TError, TData, QueryKey>, "queryKey" | "queryFn">
): UseQueryResult<TData, TError> {
  return useQuery<TData, TError, TData, QueryKey>({
    queryKey: key,
    queryFn: fetcher,
    ...options,
  });
}

/**
 * Hook React per eseguire mutazioni API con `@tanstack/react-query`.
 * Fornisce un wrapper tipizzato e standardizzato per `useMutation`.
 *
 * @param mutationFn La funzione che esegue la mutazione API.
 * @param options Opzioni aggiuntive per `useMutation`.
 * @returns Il risultato della mutazione.
 */
export function useApiMutation<TData, TError = Error, TVariables = void, TContext = unknown>(
  mutationFn: (variables: TVariables) => Promise<TData>,
  options?: Omit<UseMutationOptions<TData, TError, TVariables, TContext>, "mutationFn">
): UseMutationResult<TData, TError, TVariables, TContext> {
  return useMutation<TData, TError, TVariables, TContext>({
    mutationFn,
    ...options,
  });
}

/**
 * Hook React per eseguire query API infinite (paginate) con `@tanstack/react-query`.
 * Utile per implementare "scroll infinito" o paginazione "carica altro".
 *
 * @param key La chiave della query per la cache di React Query.
 * @param fetcher La funzione che esegue la chiamata API per una pagina specifica.
 * @param options Opzioni aggiuntive per `useInfiniteQuery`.
 * @returns Il risultato della query infinita.
 */
export function useInfiniteApiQuery<TData, TError = Error, TPageParam = string>(
  key: QueryKey,
  fetcher: (pageParam?: TPageParam) => Promise<PaginatedResult<TData>>,
  options?: Omit<
    UseInfiniteQueryOptions<PaginatedResult<TData>, TError, PaginatedResult<TData>, TData[], QueryKey, TPageParam>,
    "queryKey" | "queryFn" | "getNextPageParam"
  > & {
    getNextPageParam: (lastPage: PaginatedResult<TData>, allPages: PaginatedResult<TData>[]) => TPageParam | undefined;
  }
): UseInfiniteQueryResult<PaginatedResult<TData>, TError> {
  return useInfiniteQuery<PaginatedResult<TData>, TError, PaginatedResult<TData>, TData[], QueryKey, TPageParam>({
    queryKey: key,
    queryFn: ({ pageParam }) => fetcher(pageParam),
    ...options,
  });
}
```

---

## 11. src/server/middleware/index.ts

```typescript
import { ZodSchema, ZodError } from "zod";
import { ValidationError, ApiError, InternalServerError } from "@/lib/api/errors";
import { errorResponse } from "@/lib/api/response";
import { checkRateLimit } from "@/lib/api/rate-limit";

// Definizione di un tipo generico per il middleware (simile a Express)
// Questo tipo di middleware è pensato per route API generiche (es. REST), non tRPC-specifiche.
export type Middleware<Req = any, Res = any> = (
  req: Req,
  res: Res,
  next: (err?: unknown) => void
) => void | Promise<void>;

/**
 * Compositore di middleware.
 * Permette di combinare più funzioni middleware in una singola funzione.
 * @param middlewares Un array di funzioni middleware.
 * @returns Una singola funzione middleware composta.
 */
export function compose<Req, Res>(...middlewares: Middleware<Req, Res>[]): Middleware<Req, Res> {
  return (req, res, next) => {
    let index = 0;

    const dispatch = async (i: number, err?: unknown) => {
      if (err) return next(err);
      if (i >= middlewares.length) return next();

      const middleware = middlewares[i];
      try {
        await middleware(req, res, (e) => dispatch(i + 1, e));
      } catch (error) {
        next(error);
      }
    };

    dispatch(index);
  };
}

/**
 * Middleware per l'autenticazione.
 * Verifica la presenza di un token di autenticazione (es. JWT) nell'header.
 * Questo è un esempio semplificato.
 */
export const withAuth: Middleware = async (req, res, next) => {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json(errorResponse(new ApiError("Token di autenticazione mancante o non valido.", "UNAUTHENTICATED", 401)));
  }

  const token = authHeader.split(" ")[1];
  // Qui dovresti validare il token (es. JWT.verify)
  // Per semplicità, simuliamo una validazione
  if (token === "valid-token") {
    // req.user = { id: "user-123", role: "USER" }; // Aggiungi l'utente al request object
    next();
  } else {
    return res.status(401).json(errorResponse(new ApiError("Token di autenticazione non valido.", "UNAUTHENTICATED", 401)));
  }
};

/**
 * Middleware per il rate limiting.
 * Utilizza `checkRateLimit` per limitare le richieste per IP.
 */
export const withRateLimit: Middleware = async (req, res, next) => {
  const ip = req.headers["x-forwarded-for"] || req.socket.remoteAddress || "127.0.0.1";
  const identifier = `api_rate_limit_${ip}`;

  const { success, remaining } = await checkRateLimit(identifier);

  if (!success) {
    res.setHeader("X-RateLimit-Limit", "10"); // Esempio
    res.setHeader("X-RateLimit-Remaining", "0");
    return res.status(429).json(errorResponse(new ApiError("Troppe richieste. Riprova più tardi.", "TOO_MANY_REQUESTS", 429)));
  }

  res.setHeader("X-RateLimit-Remaining", remaining.toString());
  next();
};

/**
 * Middleware per il logging delle richieste.
 * Logga dettagli sulla richiesta e sulla risposta.
 */
export const withLogging: Middleware = async (req, res, next) => {
  const start = Date.now();
  const originalEnd = res.end;
  const originalWrite = res.write;
  const chunks: Buffer[] = [];

  // Intercetta la risposta per loggarla
  res.write = (...restArgs: any[]) => {
    chunks.push(Buffer.from(restArgs[0]));
    return originalWrite.apply(res, restArgs);
  };

  res.end = (...restArgs: any[]) => {
    if (restArgs[0]) {
      chunks.push(Buffer.from(restArgs[0]));
    }
    const body = Buffer.concat(chunks).toString("utf8");
    const durationMs = Date.now() - start;
    console.log(`[API] ${req.method} ${req.url} - Status: ${res.statusCode} - ${durationMs}ms - Response: ${body.substring(0, 100)}...`);
    originalEnd.apply(res, restArgs);
  };

  console.log(`[API] ${req.method} ${req.url} - Incoming request.`);
  next();
};

/**
 * Middleware per la validazione dei dati di input tramite Zod.
 * @param schema Lo schema Zod da usare per validare `req.body` o `req.query`.
 * @param target Il target della validazione ('body' o 'query').
 * @returns Una funzione middleware.
 */
export const withValidation = (schema: ZodSchema, target: "body" | "query" = "body"): Middleware => {
  return (req, res, next) => {
    try {
      if (target === "body") {
        req.body = schema.parse(req.body);
      } else {
        req.query = schema.parse(req.query);
      }
      next();
    } catch (error) {
      if (error instanceof ZodError) {
        return res.status(400).json(errorResponse(new ValidationError("Errore di validazione dei dati.", error.flatten())));
      }
      next(error); // Passa altri errori al gestore errori
    }
  };
};

/**
 * Middleware per la gestione centralizzata degli errori.
 * Cattura gli errori passati a `next(err)` e li formatta in una risposta standard.
 */
export const withErrorHandler: Middleware = (err, req, res, next) => {
  if (res.headersSent) {
    return next(err); // Se le header sono già state inviate, lascia che il gestore errori di default di Express gestisca
  }

  const apiError = err instanceof ApiError ? err : new InternalServerError(err?.message || "Errore interno del server.");

  console.error(`[API Error] ${req.method} ${req.url}:`, apiError);

  res.status(apiError.statusCode).json(errorResponse(apiError));
};
```

---

## 12. tests/api.test.ts

```typescript
import { describe, it, expect, vi, beforeEach, afterEach } from "vitest";
import {
  ApiError,
  ValidationError,
  AuthenticationError,
  NotFoundError,
  handleApiError,
  toTRPCError,
  RateLimitError,
} from "@/lib/api/errors";
import { successResponse, errorResponse, paginatedResponse } from "@/lib/api/response";
import { checkRateLimit, ratelimit, redis } from "@/lib/api/rate-limit";
import { idSchema, emailSchema, validate, paginationSchema } from "@/lib/api/validation";
import { apiFetch, api } from "@/lib/api/fetch";
import { compose, withAuth, withRateLimit, withValidation, withErrorHandler, Middleware } from "@/server/middleware";
import { z, ZodError } from "zod";
import { TRPCError } from "@trpc/server";

// Mock per il client Redis di Upstash per i test di rate limiting
vi.mock("@upstash/redis", () => ({
  Redis: {
    fromEnv: vi.fn(() => ({
      get: vi.fn(),
      set: vi.fn(),
      eval: vi.fn(() => Promise.resolve([1, 1000])), // Simula successo e reset
    })),
  },
}));

// Mock per la funzione fetch globale
const mockFetch = vi.fn();
beforeEach(() => {
  global.fetch = mockFetch;
});
afterEach(() => {
  vi.restoreAllMocks();
});

describe("src/lib/api/errors.ts", () => {
  it("should create custom ApiError instances correctly", () => {
    const error = new ValidationError("Invalid input", { field: "name" });
    expect(error).toBeInstanceOf(ApiError);
    expect(error.message).toBe("Invalid input");
    expect(error.code).toBe("VALIDATION_FAILED");
    expect(error.statusCode).toBe(400);
    expect(error.details).toEqual({ field: "name" });
  });

  it("handleApiError should convert ZodError to ValidationError", () => {
    const zodError = new ZodError([{ path: ["email"], message: "Invalid email", code: "invalid_string" }]);
    const apiError = handleApiError(zodError);
    expect(apiError).toBeInstanceOf(ValidationError);
    expect(apiError.message).toBe("Errore di validazione dei dati.");
    expect(apiError.details).toBeDefined();
  });

  it("handleApiError should convert generic Error to InternalServerError", () => {
    const genericError = new Error("Something went wrong");
    const apiError = handleApiError(genericError);
    expect(apiError.statusCode).toBe(500);
    expect(apiError.code).toBe("INTERNAL_SERVER_ERROR");
  });

  it("toTRPCError should convert ApiError to TRPCError", () => {
    const apiError = new AuthenticationError("Not logged in");
    const trpcError = toTRPCError(apiError);
    expect(trpcError).toBeInstanceOf(TRPCError);
    expect(trpcError.code).toBe("UNAUTHORIZED");
    expect(trpcError.message).toBe("Not logged in");
    expect(trpcError.data?.httpStatus).toBe(401);
    expect(trpcError.data?.code).toBe("UNAUTHENTICATED");
  });
});

describe("src/lib/api/response.ts", () => {
  it("should create a success response", () => {
    const data = { id: "1", name: "Test" };
    const response = successResponse(data);
    expect(response.success).toBe(true);
    expect(response.data).toEqual(data);
    expect(response.meta).toBeDefined();
    expect(response.meta?.timestamp).toBeDefined();
  });

  it("should create an error response", () => {
    const error = new NotFoundError("Item not found");
    const response = errorResponse(error);
    expect(response.success).toBe(false);
    expect(response.error).toBeDefined();
    expect(response.error?.code).toBe("NOT_FOUND");
    expect(response.error?.message).toBe("Item not found");
    expect(response.error?.statusCode).toBe(404);
  });

  it("should create a paginated response", () => {
    const data = [{ id: "1" }, { id: "2" }];
    const pagination = {
      totalItems: 10,
      itemCount: 2,
      itemsPerPage: 2,
      totalPages: 5,
      currentPage: 1,
    };
    const response = paginatedResponse(data, pagination);
    expect(response.success).toBe(true);
    expect(response.data).toEqual(data);
    expect(response.meta?.pagination).toEqual(pagination);
  });
});

describe("src/lib/api/rate-limit.ts", () => {
  it("should check rate limit successfully", async () => {
    const identifier = "test_ip_123";
    const { success, remaining } = await checkRateLimit(identifier);
    expect(success).toBe(true);
    expect(remaining).toBe(1); // Mocked to return 1
    expect(ratelimit.redis.eval).toHaveBeenCalled();
  });

  it("should handle rate limit exceeded (mocked)", async () => {
    // Mock eval per simulare il superamento del limite
    (ratelimit.redis.eval as vi.Mock).mockResolvedValueOnce([0, 1000]); // 0 remaining, reset in 1000ms

    const identifier = "test_ip_exceeded";
    const { success, remaining } = await checkRateLimit(identifier);
    expect(success).toBe(false);
    expect(remaining).toBe(0);
  });
});

describe("src/lib/api/validation.ts", () => {
  it("should validate idSchema correctly", () => {
    const validId = "clq0000000000000000000000";
    expect(idSchema.parse(validId)).toBe(validId);
    expect(() => idSchema.parse("invalid-id")).toThrow(ZodError);
  });

  it("should validate emailSchema correctly", () => {
    const validEmail = "test@example.com";
    expect(emailSchema.parse(validEmail)).toBe(validEmail);
    expect(() => emailSchema.parse("invalid-email")).toThrow(ZodError);
  });

  it("validate function should throw ValidationError for invalid data", () => {
    const schema = z.object({ name: z.string().min(3) });
    expect(() => validate(schema, { name: "ab" })).toThrow(ValidationError);
  });

  it("validate function should return parsed data for valid input", () => {
    const schema = z.object({ name: z.string().min(3) });
    const data = { name: "abc" };
    expect(validate(schema, data)).toEqual(data);
  });

  it("should validate paginationSchema correctly", () => {
    const validPagination = { page: 2, pageSize: 20, sortBy: "name", sortOrder: "asc" };
    expect(paginationSchema.parse(validPagination)).toEqual(validPagination);

    const defaultPagination = paginationSchema.parse({});
    expect(defaultPagination).toEqual({ page: 1, pageSize: 10, sortOrder: "desc" });

    expect(() => paginationSchema.parse({ page: -1 })).toThrow(ZodError);
  });
});

describe("src/lib/api/fetch.ts", () => {
  it("apiFetch should make a GET request and return data", async () => {
    const mockData = { message: "Success" };
    mockFetch.mockResolvedValueOnce({
      ok: true,
      status: 200,
      json: () => Promise.resolve(mockData),
    });

    const result = await apiFetch("/test");
    expect(result).toEqual(mockData);
    expect(mockFetch).toHaveBeenCalledWith("/test", expect.objectContaining({ method: "GET" }));
  });

  it("apiFetch should throw ApiError for non-ok responses", async () => {
    mockFetch.mockResolvedValueOnce({
      ok: false,
      status: 404,
      statusText: "Not Found",
      json: () => Promise.resolve({ message: "Resource not found", code: "NOT_FOUND" }),
    });

    await expect(apiFetch("/non-existent")).rejects.toBeInstanceOf(ApiError);
    await expect(apiFetch("/non-existent")).rejects.toHaveProperty("statusCode", 404);
  });

  it("api.post should make a POST request with body", async () => {
    const mockData = { id: "1", name: "New Item" };
    const payload = { name: "New Item" };
    mockFetch.mockResolvedValueOnce({
      ok: true,
      status: 201,
      json: () => Promise.resolve(mockData),
    });

    const result = await api.post("/items", payload);
    expect(result).toEqual(mockData);
    expect(mockFetch).toHaveBeenCalledWith(
      "/items",
      expect.objectContaining({
        method: "POST",
        body: JSON.stringify(payload),
        headers: { "Content-Type": "application/json" },
      })
    );
  });

  it("apiFetch should handle timeout", async () => {
    mockFetch.mockImplementationOnce(
      () =>
        new Promise((resolve) => {
          setTimeout(() => resolve({ ok: true, status: 200, json: () => Promise.resolve({}) }), 100);
        })
    );

    await expect(apiFetch("/timeout", { timeout: 50 })).rejects.toBeInstanceOf(ApiError);
    await expect(apiFetch("/timeout", { timeout: 50 })).rejects.toHaveProperty("code", "REQUEST_TIMEOUT");
  });
});

describe("src/server/middleware/index.ts", () => {
  const mockReq: any = {
    headers: {},
    body: {},
    query: {},
    socket: { remoteAddress: "127.0.0.1" },
    method: "GET",
    url: "/test",
  };
  const mockRes: any = {
    status: vi.fn().mockReturnThis(),
    json: vi.fn().mockReturnThis(),
    setHeader: vi.fn().mockReturnThis(),
    end: vi.fn(),
    write: vi.fn(),
    headersSent: false,
  };
  const mockNext = vi.fn();

  beforeEach(() => {
    vi.clearAllMocks();
    mockReq.headers = {};
    mockReq.body = {};
    mockReq.query = {};
    mockRes.status.mockReturnThis();
    mockRes.json.mockReturnThis();
    mockRes.setHeader.mockReturnThis();
    mockRes.end.mockClear();
    mockRes.write.mockClear();
    mockNext.mockClear();
  });

  it("compose should execute middlewares in order", async () => {
    const mw1: Middleware = (req, res, next) => {
      req.order = ["mw1"];
      next();
    };
    const mw2: Middleware = (req, res, next) => {
      req.order.push("mw2");
      next();
    };
    const composed = compose(mw1, mw2);

    await composed(mockReq, mockRes, mockNext);
    expect(mockReq.order).toEqual(["mw1", "mw2"]);
    expect(mockNext).toHaveBeenCalledTimes(1);
  });

  it("withAuth should allow valid token", async () => {
    mockReq.headers.authorization = "Bearer valid-token";
    await withAuth(mockReq, mockRes, mockNext);
    expect(mockNext).toHaveBeenCalledTimes(1);
    expect(mockRes.status).not.toHaveBeenCalled();
  });

  it("withAuth should deny invalid token", async () => {
    mockReq.headers.authorization = "Bearer invalid-token";
    await withAuth(mockReq, mockRes, mockNext);
    expect(mockRes.status).toHaveBeenCalledWith(401);
    expect(mockRes.json).toHaveBeenCalledWith(expect.objectContaining({ success: false }));
    expect(mockNext).not.toHaveBeenCalled();
  });

  it("withRateLimit should allow request if limit not exceeded", async () => {
    (ratelimit.redis.eval as vi.Mock).mockResolvedValueOnce([1, 1000]); // Success
    await withRateLimit(mockReq, mockRes, mockNext);
    expect(mockNext).toHaveBeenCalledTimes(1);
    expect(mockRes.setHeader).toHaveBeenCalledWith("X-RateLimit-Remaining", "1");
  });

  it("withRateLimit should deny request if limit exceeded", async () => {
    (ratelimit.redis.eval as vi.Mock).mockResolvedValueOnce([0, 1000]); // Exceeded
    await withRateLimit(mockReq, mockRes, mockNext);
    expect(mockRes.status).toHaveBeenCalledWith(429);
    expect(mockRes.json).toHaveBeenCalledWith(expect.objectContaining({ success: false }));
    expect(mockNext).not.toHaveBeenCalled();
  });

  it("withValidation should validate req.body and call next", async () => {
    const schema = z.object({ name: z.string().min(3) });
    mockReq.body = { name: "Valid Name" };
    const validator = withValidation(schema, "body");
    await validator(mockReq, mockRes, mockNext);
    expect(mockReq.body).toEqual({ name: "Valid Name" });
    expect(mockNext).toHaveBeenCalledTimes(1);
  });

  it("withValidation should return 400 for invalid req.body", async () => {
    const schema = z.object({ name: z.string().min(3) });
    mockReq.body = { name: "ab" };
    const validator = withValidation(schema, "body");
    await validator(mockReq, mockRes, mockNext);
    expect(mockRes.status).toHaveBeenCalledWith(400);
    expect(mockRes.json).toHaveBeenCalledWith(expect.objectContaining({ success: false }));
    expect(mockNext).not.toHaveBeenCalled();
  });

  it("withErrorHandler should catch ApiError and send formatted response", async () => {
    const error = new NotFoundError("Resource not found");
    await withErrorHandler(error, mockReq, mockRes, mockNext);
    expect(mockRes.status).toHaveBeenCalledWith(404);
    expect(mockRes.json).toHaveBeenCalledWith(expect.objectContaining({ success: false, error: { code: "NOT_FOUND" } }));
    expect(mockNext).not.toHaveBeenCalled();
  });

  it("withErrorHandler should catch generic Error and send 500", async () => {
    const error = new Error("Something unexpected");
    await withErrorHandler(error, mockReq, mockRes, mockNext);
    expect(mockRes.status).toHaveBeenCalledWith(500);
    expect(mockRes.json).toHaveBeenCalledWith(expect.objectContaining({ success: false, error: { code: "INTERNAL_SERVER_ERROR" } }));
    expect(mockNext).not.toHaveBeenCalled();
  });
});
```

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 19873_