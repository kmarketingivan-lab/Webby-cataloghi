# CATALOGO-TRPC-API

Catalogo TRPC-API per Next.js 14 App Router - Espansione Completa
§ TRPC V11 SETUP
Full App Router Setup
typescript
// app/api/trpc/[trpc]/route.ts
import { fetchRequestHandler } from '@trpc/server/adapters/fetch';
import { appRouter } from '@/server/api/root';
import { createTRPCContext } from '@/server/api/trpc';

const handler = async (req: Request) => {
  return fetchRequestHandler({
    endpoint: '/api/trpc',
    req,
    router: appRouter,
    createContext: createTRPCContext,
    onError:
      process.env.NODE_ENV === 'development'
        ? ({ path, error }) => {
            console.error(
              `❌ tRPC failed on ${path ?? '<no-path>'}: ${error.message}`
            );
          }
        : undefined,
    responseMeta({ ctx, paths, type, errors }) {
      // Cache API responses for 1 minute, revalidate every 10 seconds
      const ONE_MINUTE_IN_SECONDS = 60;
      if (ctx?.res && paths?.every(path => path.startsWith('posts.list'))) {
        return {
          headers: {
            'cache-control': `s-maxage=10, stale-while-revalidate=${ONE_MINUTE_IN_SECONDS}`,
          },
        };
      }
      return {};
    },
  });
};

export { handler as GET, handler as POST };
Server Client Setup
typescript
// lib/trpc/server-client.ts
import 'server-only';
import { createTRPCClient, httpBatchLink } from '@trpc/client';
import { appRouter } from '@/server/api/root';
import { cookies } from 'next/headers';

export const createServerClient = () => {
  return createTRPCClient<typeof appRouter>({
    links: [
      httpBatchLink({
        url: `${process.env.NEXT_PUBLIC_APP_URL}/api/trpc`,
        headers: async () => {
          const cookieStore = await cookies();
          const token = cookieStore.get('session-token')?.value;
          
          return {
            cookie: cookieStore.toString(),
            ...(token && { Authorization: `Bearer ${token}` }),
          };
        },
        fetch: (url, options) => {
          return fetch(url, {
            ...options,
            credentials: 'include',
          });
        },
      }),
    ],
  });
};

// Usage in Server Components
export async function ServerComponent() {
  const trpc = createServerClient();
  const user = await trpc.user.getCurrent.query();
  
  return <div>{user.name}</div>;
}
React Query Integration
typescript
// lib/trpc/react.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { loggerLink, unstable_httpBatchStreamLink } from '@trpc/client';
import { createTRPCReact } from '@trpc/react-query';
import { useState } from 'react';
import { AppRouter } from '@/server/api/root';
import superjson from 'superjson';

export const api = createTRPCReact<AppRouter>();

export function TRPCReactProvider(props: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 minute
            gcTime: 5 * 60 * 1000, // 5 minutes
            retry: (failureCount, error: any) => {
              if (error?.data?.code === 'NOT_FOUND') return false;
              if (error?.data?.code === 'UNAUTHORIZED') return false;
              return failureCount < 2;
            },
            refetchOnWindowFocus: false,
            refetchOnMount: true,
            refetchOnReconnect: true,
          },
          mutations: {
            retry: false,
            onError: (error: any) => {
              console.error('Mutation error:', error);
            },
          },
        },
      })
  );

  const [trpcClient] = useState(() =>
    api.createClient({
      links: [
        loggerLink({
          enabled: (opts) =>
            process.env.NODE_ENV === 'development' ||
            (opts.direction === 'down' && opts.result instanceof Error),
        }),
        unstable_httpBatchStreamLink({
          transformer: superjson,
          url: `${process.env.NEXT_PUBLIC_APP_URL}/api/trpc`,
          headers: async () => {
            const headers = new Headers();
            headers.set('x-trpc-source', 'nextjs-react');
            return headers;
          },
          fetch: async (url, options) => {
            return fetch(url, {
              ...options,
              credentials: 'include',
            });
          },
        }),
      ],
    })
  );

  return (
    <QueryClientProvider client={queryClient}>
      <api.Provider client={trpcClient} queryClient={queryClient}>
        {props.children}
      </api.Provider>
    </QueryClientProvider>
  );
}
Middleware Integration
typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';
import { createTRPCContext } from '@/server/api/trpc';

export async function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Add CORS headers for tRPC
  if (request.nextUrl.pathname.startsWith('/api/trpc')) {
    response.headers.set('Access-Control-Allow-Origin', '*');
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }
  
  // Rate limiting for API routes
  const ip = request.ip ?? '127.0.0.1';
  const rateLimitKey = `rate-limit:${ip}`;
  
  // You would implement your rate limiting logic here
  // For example, using Redis or similar
  
  return response;
}

export const config = {
  matcher: ['/api/trpc/:path*', '/api/:path*'],
};
Context Creation
typescript
// server/api/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server';
import superjson from 'superjson';
import { ZodError } from 'zod';
import { cookies, headers } from 'next/headers';
import { cache } from 'react';
import { auth } from '@/lib/auth';

export const createTRPCContext = cache(async (opts: { headers: Headers }) => {
  // Get session from auth library
  const session = await auth();
  
  // Parse cookies for additional data
  const cookieStore = await cookies();
  const userAgent = headers().get('user-agent') || '';
  
  // Rate limiting data
  const ip = headers().get('x-forwarded-for') || 'unknown';
  
  return {
    session,
    ip,
    userAgent,
    cookies: cookieStore,
    // Add database connection or other services
    db: {}, // Your database connection
    redis: {}, // Your Redis connection
    ...opts,
  };
});

export type Context = Awaited<ReturnType<typeof createTRPCContext>>;

const t = initTRPC.context<Context>().create({
  transformer: superjson,
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError:
          error.cause instanceof ZodError ? error.cause.flatten() : null,
      },
    };
  },
});

export const createTRPCRouter = t.router;
export const publicProcedure = t.procedure;
export const protectedProcedure = t.procedure.use(
  t.middleware(async ({ ctx, next }) => {
    if (!ctx.session?.user) {
      throw new TRPCError({ code: 'UNAUTHORIZED' });
    }
    return next({
      ctx: {
        ...ctx,
        session: { ...ctx.session, user: ctx.session.user },
      },
    });
  })
);
§ ROUTER ORGANIZATION
Router Splitting
typescript
// server/api/routers/_app.ts
import { createTRPCRouter } from '../trpc';

export const appRouter = createTRPCRouter({});

export type AppRouter = typeof appRouter;
typescript
// server/api/routers/user.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure, protectedProcedure } from '../trpc';

export const userRouter = createTRPCRouter({
  // Public procedures
  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ ctx, input }) => {
      // Implementation
      return { id: input.id, name: 'John Doe' };
    }),
  
  // Protected procedures
  updateProfile: protectedProcedure
    .input(z.object({
      name: z.string().min(2),
      email: z.string().email(),
    }))
    .mutation(async ({ ctx, input }) => {
      // Only authenticated users can access this
      const userId = ctx.session.user.id;
      // Update logic here
      return { success: true };
    }),
});
Nested Routers
typescript
// server/api/routers/post.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure, protectedProcedure } from '../trpc';

const postRouter = createTRPCRouter({
  list: publicProcedure
    .input(z.object({
      limit: z.number().min(1).max(100).default(20),
      cursor: z.string().optional(),
    }))
    .query(async ({ ctx, input }) => {
      // Implementation
      return { posts: [], nextCursor: null };
    }),
  
  get: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ ctx, input }) => {
      // Implementation
      return { id: input.id, title: 'Post Title', content: '...' };
    }),
});

const adminPostRouter = createTRPCRouter({
  create: protectedProcedure
    .input(z.object({
      title: z.string().min(1),
      content: z.string(),
      published: z.boolean().default(false),
    }))
    .mutation(async ({ ctx, input }) => {
      // Admin-only logic
      return { id: '123', ...input };
    }),
  
  delete: protectedProcedure
    .input(z.object({ id: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Admin-only delete logic
      return { success: true };
    }),
});

export const postRouter = createTRPCRouter({
  public: postRouter,
  admin: adminPostRouter,
});
Shared Procedures
typescript
// server/api/procedures/base.ts
import { z } from 'zod';
import { publicProcedure } from '../trpc';

// Reusable input validators
export const paginationSchema = z.object({
  page: z.number().min(1).default(1),
  pageSize: z.number().min(1).max(100).default(20),
  cursor: z.string().optional(),
});

export const searchSchema = z.object({
  query: z.string().min(1).max(100),
  filters: z.record(z.any()).optional(),
});

// Base procedures with common middleware
export const withPagination = publicProcedure.input(paginationSchema);
export const withSearch = publicProcedure.input(searchSchema);
typescript
// server/api/procedures/audited.ts
import { protectedProcedure } from '../trpc';

export const auditedProcedure = protectedProcedure
  .use(async ({ ctx, next }) => {
    const startTime = Date.now();
    const result = await next();
    const duration = Date.now() - startTime;
    
    // Log audit trail
    await ctx.db.auditLog.create({
      data: {
        userId: ctx.session.user.id,
        action: ctx.path,
        duration,
        ip: ctx.ip,
        userAgent: ctx.userAgent,
      },
    });
    
    return result;
  });
Input/Output Validation
typescript
// server/api/validations/user.ts
import { z } from 'zod';

export const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(2).max(100),
  role: z.enum(['USER', 'ADMIN', 'MODERATOR']),
  createdAt: z.date(),
  updatedAt: z.date(),
});

export const createUserSchema = userSchema.pick({
  email: true,
  name: true,
  role: true,
}).extend({
  password: z.string().min(8),
});

export const updateUserSchema = createUserSchema.partial();

// Response schemas
export const userResponseSchema = userSchema;
export const usersResponseSchema = z.array(userSchema);
export const paginatedUsersResponseSchema = z.object({
  users: usersResponseSchema,
  total: z.number(),
  page: z.number(),
  pageSize: z.number(),
});
typescript
// server/api/routers/user.router.ts
import { createTRPCRouter, protectedProcedure } from '../trpc';
import { 
  createUserSchema, 
  updateUserSchema, 
  paginatedUsersResponseSchema 
} from '../validations/user';

export const userRouter = createTRPCRouter({
  create: protectedProcedure
    .input(createUserSchema)
    .output(z.object({ id: z.string(), email: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Input is validated by Zod schema
      // Output is also validated
      return { id: '123', email: input.email };
    }),
  
  list: protectedProcedure
    .input(z.object({
      page: z.number().default(1),
      pageSize: z.number().default(20),
    }))
    .output(paginatedUsersResponseSchema)
    .query(async ({ ctx, input }) => {
      // Implementation
      return {
        users: [],
        total: 0,
        page: input.page,
        pageSize: input.pageSize,
      };
    }),
});
§ PROCEDURES AVANZATE
Queries with Advanced Features
typescript
// server/api/routers/advanced.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';
import { TRPCError } from '@trpc/server';
import { redis } from '@/lib/redis';

export const advancedRouter = createTRPCRouter({
  // Cached query with Redis
  getCachedData: publicProcedure
    .input(z.object({ key: z.string() }))
    .query(async ({ ctx, input }) => {
      const cacheKey = `data:${input.key}`;
      
      // Try cache first
      const cached = await redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }
      
      // Fetch from database
      const data = await ctx.db.data.findUnique({
        where: { key: input.key },
      });
      
      if (!data) {
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Data not found',
        });
      }
      
      // Cache for 5 minutes
      await redis.setex(cacheKey, 300, JSON.stringify(data));
      
      return data;
    }),
  
  // Infinite scroll query
  infinitePosts: publicProcedure
    .input(z.object({
      limit: z.number().min(1).max(50).default(10),
      cursor: z.string().nullish(),
    }))
    .query(async ({ ctx, input }) => {
      const { limit, cursor } = input;
      
      const items = await ctx.db.post.findMany({
        take: limit + 1, // Get one extra to know if there's more
        cursor: cursor ? { id: cursor } : undefined,
        orderBy: { createdAt: 'desc' },
      });
      
      let nextCursor: typeof cursor | undefined = undefined;
      if (items.length > limit) {
        const nextItem = items.pop();
        nextCursor = nextItem!.id;
      }
      
      return {
        items,
        nextCursor,
      };
    }),
  
  // Batched query
  getMultipleUsers: publicProcedure
    .input(z.object({ ids: z.array(z.string()) }))
    .query(async ({ ctx, input }) => {
      if (input.ids.length > 100) {
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'Maximum 100 IDs allowed',
        });
      }
      
      const users = await ctx.db.user.findMany({
        where: { id: { in: input.ids } },
      });
      
      // Return in same order as requested
      const userMap = new Map(users.map(u => [u.id, u]));
      return input.ids.map(id => userMap.get(id) || null);
    }),
});
Mutations with Transaction Support
typescript
// server/api/routers/transaction.router.ts
import { z } from 'zod';
import { createTRPCRouter, protectedProcedure } from '../trpc';
import { TRPCError } from '@trpc/server';

export const transactionRouter = createTRPCRouter({
  transferFunds: protectedProcedure
    .input(z.object({
      fromAccountId: z.string(),
      toAccountId: z.string(),
      amount: z.number().positive(),
      description: z.string().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      return await ctx.db.$transaction(async (tx) => {
        // 1. Lock and check source account
        const fromAccount = await tx.account.findUnique({
          where: { id: input.fromAccountId },
          select: { balance: true, userId: true },
        });
        
        if (!fromAccount) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Source account not found',
          });
        }
        
        if (fromAccount.userId !== ctx.session.user.id) {
          throw new TRPCError({
            code: 'FORBIDDEN',
            message: 'You do not own this account',
          });
        }
        
        if (fromAccount.balance < input.amount) {
          throw new TRPCError({
            code: 'BAD_REQUEST',
            message: 'Insufficient funds',
          });
        }
        
        // 2. Check destination account
        const toAccount = await tx.account.findUnique({
          where: { id: input.toAccountId },
        });
        
        if (!toAccount) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Destination account not found',
          });
        }
        
        // 3. Perform transfer
        await tx.account.update({
          where: { id: input.fromAccountId },
          data: { balance: { decrement: input.amount } },
        });
        
        await tx.account.update({
          where: { id: input.toAccountId },
          data: { balance: { increment: input.amount } },
        });
        
        // 4. Create transaction record
        const transaction = await tx.transaction.create({
          data: {
            fromAccountId: input.fromAccountId,
            toAccountId: input.toAccountId,
            amount: input.amount,
            description: input.description,
            status: 'COMPLETED',
          },
        });
        
        return {
          transactionId: transaction.id,
          newBalance: fromAccount.balance - input.amount,
          timestamp: new Date(),
        };
      });
    }),
  
  // Bulk mutation
  bulkUpdate: protectedProcedure
    .input(z.object({
      updates: z.array(z.object({
        id: z.string(),
        data: z.record(z.any()),
      })),
    }))
    .mutation(async ({ ctx, input }) => {
      const results = await Promise.all(
        input.updates.map(async (update) => {
          try {
            const result = await ctx.db.item.update({
              where: { id: update.id },
              data: update.data,
            });
            return { id: update.id, success: true, data: result };
          } catch (error) {
            return { id: update.id, success: false, error: (error as Error).message };
          }
        })
      );
      
      return {
        total: results.length,
        successful: results.filter(r => r.success).length,
        failed: results.filter(r => !r.success).length,
        results,
      };
    }),
});
Subscriptions (WebSocket)
typescript
// server/api/routers/subscription.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';
import { observable } from '@trpc/server/observable';
import { EventEmitter } from 'events';
import { TRPCError } from '@trpc/server';

// Create event emitter for real-time events
const ee = new EventEmitter();

export const subscriptionRouter = createTRPCRouter({
  // Simple subscription
  onNotification: publicProcedure
    .input(z.object({ userId: z.string() }))
    .subscription(({ input }) => {
      return observable<{ message: string; timestamp: Date }>((emit) => {
        const onNotification = (data: { userId: string; message: string }) => {
          if (data.userId === input.userId) {
            emit.next({
              message: data.message,
              timestamp: new Date(),
            });
          }
        };
        
        ee.on('notification', onNotification);
        
        return () => {
          ee.off('notification', onNotification);
        };
      });
    }),
  
  // Chat room subscription
  onChatMessage: publicProcedure
    .input(z.object({ roomId: z.string() }))
    .subscription(({ input }) => {
      return observable<{
        id: string;
        userId: string;
        message: string;
        timestamp: Date;
      }>((emit) => {
        const onMessage = (message: any) => {
          if (message.roomId === input.roomId) {
            emit.next(message);
          }
        };
        
        ee.on(`chat:${input.roomId}`, onMessage);
        
        return () => {
          ee.off(`chat:${input.roomId}`, onMessage);
        };
      });
    }),
  
  // Send chat message (mutation that triggers subscription)
  sendChatMessage: publicProcedure
    .input(z.object({
      roomId: z.string(),
      message: z.string().min(1).max(1000),
    }))
    .mutation(async ({ ctx, input }) => {
      const chatMessage = {
        id: Math.random().toString(36).substr(2, 9),
        userId: ctx.session?.user?.id || 'anonymous',
        roomId: input.roomId,
        message: input.message,
        timestamp: new Date(),
      };
      
      // Emit event to all subscribers
      ee.emit(`chat:${input.roomId}`, chatMessage);
      
      // Also store in database
      await ctx.db.chatMessage.create({
        data: {
          roomId: input.roomId,
          userId: chatMessage.userId,
          message: input.message,
        },
      });
      
      return { success: true, messageId: chatMessage.id };
    }),
  
  // Counter with subscription
  counter: publicProcedure
    .input(z.object({ interval: z.number().min(100).max(10000) }))
    .subscription(({ input }) => {
      return observable<number>((emit) => {
        let count = 0;
        const interval = setInterval(() => {
          count++;
          emit.next(count);
        }, input.interval);
        
        return () => {
          clearInterval(interval);
        };
      });
    }),
});
Batching Optimization
typescript
// server/api/routers/batch.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';
import DataLoader from 'dataloader';

export const batchRouter = createTRPCRouter({
  // Using DataLoader for batch loading
  getUsersBatch: publicProcedure
    .input(z.object({ ids: z.array(z.string()) }))
    .query(async ({ ctx, input }) => {
      // Create DataLoader instance
      const userLoader = new DataLoader(async (ids: readonly string[]) => {
        const users = await ctx.db.user.findMany({
          where: { id: { in: ids as string[] } },
        });
        
        const userMap = new Map(users.map(u => [u.id, u]));
        return ids.map(id => userMap.get(id) || null);
      });
      
      return Promise.all(input.ids.map(id => userLoader.load(id)));
    }),
  
  // Batch mutation with validation
  batchUpdate: publicProcedure
    .input(z.object({
      updates: z.array(z.object({
        id: z.string(),
        data: z.record(z.any()),
      })).max(50), // Limit batch size
    }))
    .mutation(async ({ ctx, input }) => {
      const results = [];
      
      for (const update of input.updates) {
        try {
          const result = await ctx.db.item.update({
            where: { id: update.id },
            data: update.data,
          });
          results.push({ id: update.id, success: true, data: result });
        } catch (error) {
          results.push({ 
            id: update.id, 
            success: false, 
            error: (error as Error).message 
          });
        }
      }
      
      return {
        batchId: `batch_${Date.now()}`,
        processed: results.length,
        successful: results.filter(r => r.success).length,
        results,
      };
    }),
});
Advanced Error Handling
typescript
// server/api/errors/custom-errors.ts
import { TRPCError } from '@trpc/server';

export class CustomTRPCError extends TRPCError {
  constructor(
    code: TRPCError['code'],
    message: string,
    public details?: Record<string, any>
  ) {
    super({ code, message });
  }
}

export class ValidationError extends CustomTRPCError {
  constructor(message: string, public fields: Record<string, string[]>) {
    super('BAD_REQUEST', message, { fields });
  }
}

export class BusinessError extends CustomTRPCError {
  constructor(message: string, public businessCode: string) {
    super('BAD_REQUEST', message, { businessCode });
  }
}

export class RateLimitError extends CustomTRPCError {
  constructor(message: string, public retryAfter: number) {
    super('TOO_MANY_REQUESTS', message, { retryAfter });
  }
}
typescript
// server/api/routers/error.router.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure } from '../trpc';
import { TRPCError } from '@trpc/server';
import { 
  ValidationError, 
  BusinessError, 
  RateLimitError 
} from '../errors/custom-errors';

export const errorRouter = createTRPCRouter({
  // Structured error example
  createWithValidation: publicProcedure
    .input(z.object({
      email: z.string().email(),
      age: z.number().min(18),
    }))
    .mutation(async ({ input }) => {
      if (input.age > 120) {
        throw new ValidationError('Invalid age', {
          age: ['Age must be less than 120'],
        });
      }
      
      // Business logic error
      if (input.email.includes('spam')) {
        throw new BusinessError('Suspicious email detected', 'EMAIL_SPAM');
      }
      
      return { success: true };
    }),
  
  // Error with retry information
  rateLimited: publicProcedure
    .input(z.object({ action: z.string() }))
    .mutation(async ({ ctx, input }) => {
      // Check rate limit
      const key = `rate:${ctx.ip}:${input.action}`;
      const attempts = await ctx.redis.incr(key);
      
      if (attempts === 1) {
        await ctx.redis.expire(key, 60); // 1 minute
      }
      
      if (attempts > 5) {
        throw new RateLimitError('Too many attempts', 60); // Retry after 60 seconds
      }
      
      return { attempts };
    }),
  
  // Error transformation
  getWithErrorHandling: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ ctx, input }) => {
      try {
        const data = await ctx.db.item.findUniqueOrThrow({
          where: { id: input.id },
        });
        return data;
      } catch (error) {
        // Transform database errors to tRPC errors
        if ((error as any).code === 'P2025') {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Item not found',
            cause: error,
          });
        }
        
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch item',
          cause: error,
        });
      }
    }),
});
§ MIDDLEWARE
Auth Middleware
typescript
// server/middleware/auth.ts
import { TRPCError } from '@trpc/server';
import { middleware } from '../trpc';

// Rate limiting middleware
export const rateLimitMiddleware = middleware(async ({ ctx, next }) => {
  const { redis, ip } = ctx;
  const key = `rate-limit:${ip}`;
  
  const current = await redis.get(key);
  if (current && parseInt(current) > 100) {
    throw new TRPCError({
      code: 'TOO_MANY_REQUESTS',
      message: 'Rate limit exceeded',
    });
  }
  
  await redis.incr(key);
  await redis.expire(key, 60); // Reset after 60 seconds
  
  return next();
});

// Role-based middleware
export const roleMiddleware = (allowedRoles: string[]) => 
  middleware(async ({ ctx, next }) => {
    if (!ctx.session?.user) {
      throw new TRPCError({ code: 'UNAUTHORIZED' });
    }
    
    if (!allowedRoles.includes(ctx.session.user.role)) {
      throw new TRPCError({ 
        code: 'FORBIDDEN',
        message: 'Insufficient permissions',
      });
    }
    
    return next({
      ctx: {
        ...ctx,
        user: ctx.session.user,
      },
    });
  });

// API key middleware
export const apiKeyMiddleware = middleware(async ({ ctx, next }) => {
  const apiKey = ctx.headers.get('x-api-key');
  
  if (!apiKey) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: 'API key required',
    });
  }
  
  const validKey = await ctx.db.apiKey.findUnique({
    where: { key: apiKey, active: true },
  });
  
  if (!validKey) {
    throw new TRPCError({
      code: 'UNAUTHORIZED',
      message: 'Invalid API key',
    });
  }
  
  // Update last used
  await ctx.db.apiKey.update({
    where: { id: validKey.id },
    data: { lastUsed: new Date() },
  });
  
  return next({
    ctx: {
      ...ctx,
      apiKey: validKey,
    },
  });
});

// Combine multiple middleware
export const authenticatedRateLimited = middleware(async ({ ctx, next }) => {
  // First check auth
  if (!ctx.session?.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  
  // Then check rate limit
  await rateLimitMiddleware({ ctx, next });
  
  return next({
    ctx: {
      ...ctx,
      user: ctx.session.user,
    },
  });
});
Logging Middleware
typescript
// server/middleware/logging.ts
import { middleware } from '../trpc';

export const loggingMiddleware = middleware(async ({ ctx, path, type, next }) => {
  const start = Date.now();
  const requestId = Math.random().toString(36).substring(7);
  
  // Log request
  console.log(`[${requestId}] ${type.toUpperCase()} ${path}`, {
    ip: ctx.ip,
    userAgent: ctx.userAgent,
    userId: ctx.session?.user?.id,
  });
  
  try {
    const result = await next();
    const duration = Date.now() - start;
    
    // Log successful response
    console.log(`[${requestId}] SUCCESS ${duration}ms`);
    
    // Audit log for sensitive operations
    if (path.includes('.delete') || path.includes('.update')) {
      await ctx.db.auditLog.create({
        data: {
          requestId,
          userId: ctx.session?.user?.id,
          action: path,
          method: type,
          duration,
          status: 'SUCCESS',
          ip: ctx.ip,
          userAgent: ctx.userAgent,
        },
      });
    }
    
    return result;
  } catch (error) {
    const duration = Date.now() - start;
    
    // Log error
    console.error(`[${requestId}] ERROR ${duration}ms`, error);
    
    // Audit log for errors
    await ctx.db.auditLog.create({
      data: {
        requestId,
        userId: ctx.session?.user?.id,
        action: path,
        method: type,
        duration,
        status: 'ERROR',
        error: (error as Error).message,
        ip: ctx.ip,
        userAgent: ctx.userAgent,
      },
    });
    
    throw error;
  }
});

// Performance monitoring middleware
export const performanceMiddleware = middleware(async ({ ctx, next, path }) => {
  const start = performance.now();
  
  // Add timeout
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new TRPCError({
        code: 'TIMEOUT',
        message: 'Request timeout',
      }));
    }, 10000); // 10 second timeout
  });
  
  try {
    const result = await Promise.race([next(), timeoutPromise]);
    const duration = performance.now() - start;
    
    // Log slow requests
    if (duration > 1000) { // More than 1 second
      console.warn(`Slow request: ${path} took ${duration.toFixed(2)}ms`);
    }
    
    // Add server timing header
    if (ctx.res) {
      ctx.res.headers.set('Server-Timing', `total;dur=${duration}`);
    }
    
    return result;
  } catch (error) {
    const duration = performance.now() - start;
    console.error(`Request failed after ${duration.toFixed(2)}ms:`, error);
    throw error;
  }
});
Context Augmentation
typescript
// server/middleware/context.ts
import { middleware } from '../trpc';

// Add request metadata to context
export const requestMetadataMiddleware = middleware(async ({ ctx, next }) => {
  const requestId = Math.random().toString(36).substring(7);
  const startTime = Date.now();
  
  return next({
    ctx: {
      ...ctx,
      requestMetadata: {
        id: requestId,
        startTime,
        ip: ctx.ip,
        userAgent: ctx.userAgent,
        country: ctx.headers.get('cf-ipcountry') || 'unknown',
      },
    },
  });
});

// Add caching context
export const cacheContextMiddleware = middleware(async ({ ctx, next }) => {
  return next({
    ctx: {
      ...ctx,
      cache: {
        get: async (key: string) => {
          return ctx.redis.get(key);
        },
        set: async (key: string, value: any, ttl?: number) => {
          if (ttl) {
            return ctx.redis.setex(key, ttl, JSON.stringify(value));
          }
          return ctx.redis.set(key, JSON.stringify(value));
        },
        delete: async (key: string) => {
          return ctx.redis.del(key);
        },
      },
    },
  });
});

// Add feature flags context
export const featureFlagsMiddleware = middleware(async ({ ctx, next }) => {
  const userId = ctx.session?.user?.id;
  
  // Get feature flags for user
  const flags = await ctx.db.featureFlag.findMany({
    where: {
      OR: [
        { userId },
        { userId: null }, // Global flags
      ],
    },
  });
  
  const featureFlags = flags.reduce((acc, flag) => {
    acc[flag.name] = flag.enabled;
    return acc;
  }, {} as Record<string, boolean>);
  
  return next({
    ctx: {
      ...ctx,
      featureFlags,
    },
  });
});
§ OPTIMISTIC UPDATES
useMutation with onMutate
typescript
// hooks/useOptimisticMutation.ts
import { useState } from 'react';
import { useQueryClient } from '@tanstack/react-query';
import { api } from '@/lib/trpc/react';
import type { RouterOutputs } from '@/lib/trpc';

export function useOptimisticMutation() {
  const queryClient = useQueryClient();
  const utils = api.useUtils();
  
  const addPost = api.post.create.useMutation

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-33-TRPC-API
Prompt ID: 33 / 48
Parte: 2
Exported: 2026-02-06T12:55:30.093Z
Characters: 1444
════════════════════════════════════════════════════════════

st fileRecord = await ctx.db.file.create({
        data: {
          userId: ctx.session.user.id,
          originalName: file.name,
          fileName: uniqueKey,
          filePath: publicUrl,
          mimeType: file.type,
          size: file.size,
          storageType: 'S3',
          metadata: {
            bucket,
            key: uniqueKey,
            region: process.env.AWS_REGION,
          },
          uploadStatus: 'COMPLETED',
        },
      });
      
      return {
        id: fileRecord.id,
        url: publicUrl,
        key: uniqueKey,
        bucket,
        size: file.size,
      };
    }),
});
Presigned URLs Pattern
typescript
// server/api/routers/presigned.router.ts
import { z } from 'zod';
import { createTRPCRouter, protectedProcedure } from '../trpc';
import { 
  S3Client, 
  PutObjectCommand, 
  GetObjectCommand,
  DeleteObjectCommand 
} from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3Client = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

export const presignedRouter = createTRPCRouter({
  // Generate presigned URL for upload
  generateUploadUrl: protectedProcedure
    .input(
      z.object({
        fileName: z.string(),
        fileType: z.string(),
        fileSize: z.number().max(100 * 1024

## § ADVANCED PATTERNS: TRPC API

### Server Actions con Validazione

```typescript
// app/actions/trpc-api.ts
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


### TRPC API - Utility Helper #293

```typescript
// lib/utils/trpc-api-helper-293.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #56

```typescript
// lib/utils/trpc-api-helper-56.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #388

```typescript
// lib/utils/trpc-api-helper-388.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #654

```typescript
// lib/utils/trpc-api-helper-654.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #878

```typescript
// lib/utils/trpc-api-helper-878.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #148

```typescript
// lib/utils/trpc-api-helper-148.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #240

```typescript
// lib/utils/trpc-api-helper-240.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #649

```typescript
// lib/utils/trpc-api-helper-649.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #640

```typescript
// lib/utils/trpc-api-helper-640.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #908

```typescript
// lib/utils/trpc-api-helper-908.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #922

```typescript
// lib/utils/trpc-api-helper-922.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #69

```typescript
// lib/utils/trpc-api-helper-69.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #309

```typescript
// lib/utils/trpc-api-helper-309.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #282

```typescript
// lib/utils/trpc-api-helper-282.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #837

```typescript
// lib/utils/trpc-api-helper-837.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #113

```typescript
// lib/utils/trpc-api-helper-113.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<TRPCAPIConfig> = {}) {
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

  getConfig(): Readonly<TRPCAPIConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### TRPC API - Utility Helper #515

```typescript
// lib/utils/trpc-api-helper-515.ts
import { z } from "zod";

interface TRPCAPIConfig {
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

export class TRPCAPIProcessor<TInput, TOutput> {
  private config: TRPCAPIConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(