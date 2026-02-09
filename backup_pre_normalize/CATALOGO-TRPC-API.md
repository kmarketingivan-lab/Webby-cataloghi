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