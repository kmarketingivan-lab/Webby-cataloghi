# CATALOGO REST API DESIGN v2

> **Versione**: 2.0
> **Data**: 2026-01-27
> **Ambito**: URL design, HTTP methods, Status codes, Response formats, Pagination, Versioning, Authentication, tRPC, OpenAPI, Webhooks, Testing, Gateway patterns
> **Sezioni**: 1-20 (13 originali + 7 espansione)

---

## 1. URL DESIGN RULES

| Rule                      | Correct                                 | Wrong                               | Reason                                          | HTTP Verb |
| ------------------------- | --------------------------------------- | ----------------------------------- | ----------------------------------------------- | --------- |
| Resource naming plural    | `/users`                                | `/user`                             | REST convention: resources are plural           | GET, POST |
| Nested resource limit     | `/users/123/posts`                      | `/users/123/posts/456/comments/789` | Avoid deep nesting >2 levels                    | GET       |
| Path vs query             | `/users/123`                            | `/users?id=123`                     | Use path for resource id                        | GET       |
| Filtering                 | `/posts?status=published`               | `/posts/status/published`           | Query parameters better for filters             | GET       |
| Sorting                   | `/posts?sort=-createdAt`                | `/posts/sort/desc`                  | Query params more flexible                      | GET       |
| Pagination                | `/posts?page=2&pageSize=20`             | `/posts/2/20`                       | Explicit query params standard                  | GET       |
| Versioning in URL         | `/v1/users`                             | `/users?version=1`                  | URL versioning is clear                         | GET, POST |
| Versioning in header      | `Accept: application/vnd.myapi.v1+json` | `/users?v=1`                        | Header versioning allows backward compatibility | GET       |
| No verbs in URL           | `/users`                                | `/getUsers`                         | REST is resource-based, not action-based        | GET       |
| Consistent casing         | `/users`                                | `/Users`                            | URLs should be lowercase                        | GET       |
| Use hyphens               | `/user-profiles`                        | `/user_profiles`                    | Hyphens improve readability                     | GET       |
| Avoid file extensions     | `/users`                                | `/users.json`                       | MIME types should be in headers                 | GET       |
| Singular resource in POST | `/users`                                | `/users/create`                     | POST to collection, not action                  | POST      |
| Action via HTTP           | `POST /users/123/activate`              | `/users/123/activateUser`           | Use HTTP method semantics                       | POST      |
| Boolean filters           | `/posts?published=true`                 | `/posts/published/true`             | Query params for filtering                      | GET       |
| Range filters             | `/orders?amount_gt=100`                 | `/orders/minAmount/100`             | Query params flexible for ranges                | GET       |
| Optional segments         | `/users/{id}/posts?limit=10`            | `/users/{id}/posts/10`              | Query params are optional                       | GET       |
| Search query              | `/products?search=laptop`               | `/products/search/laptop`           | Query params standard                           | GET       |
| Resource identifiers      | `/users/123`                            | `/users/name/john`                  | IDs preferred over names                        | GET       |
| Relationships             | `/users/123/friends`                    | `/users/123?include=friends`        | Both valid, nested preferred for direct child   | GET       |
| Batch actions             | `/users/batch`                          | `/users/multiUpdate`                | Endpoint name generic for batch                 | POST      |

---

## 2. HTTP METHODS MATRIX

| Method  | Idempotent | Safe | Request Body | Response Body | Status Codes  | Use Case                  |
| ------- | ---------- | ---- | ------------ | ------------- | ------------- | ------------------------- |
| GET     | Yes        | Yes  | No           | JSON          | 200, 404      | Fetch resource            |
| POST    | No         | No   | Yes          | JSON          | 201, 400      | Create resource           |
| PUT     | Yes        | No   | Yes          | JSON          | 200, 204, 400 | Replace resource          |
| PATCH   | No         | No   | Partial JSON | JSON          | 200, 204, 400 | Update resource partially |
| DELETE  | Yes        | No   | No           | JSON          | 204, 404      | Delete resource           |
| OPTIONS | Yes        | Yes  | No           | Headers       | 200           | Preflight CORS            |
| HEAD    | Yes        | Yes  | No           | Headers only  | 200, 404      | Check existence           |

---

## 3. STATUS CODES REFERENCE

| Code | Name                   | When to Use         | Response Body | Retry |
| ---- | ---------------------- | ------------------- | ------------- | ----- |
| 200  | OK                     | Success GET         | `{data:...}`  | No    |
| 201  | Created                | Resource created    | `{data:...}`  | No    |
| 202  | Accepted               | Async processing    | `{jobId:...}` | Yes   |
| 204  | No Content             | Success w/o body    | None          | No    |
| 301  | Moved Permanently      | Redirect URL        | `Location`    | Yes   |
| 302  | Found                  | Temporary redirect  | `Location`    | Yes   |
| 304  | Not Modified           | Caching             | None          | No    |
| 400  | Bad Request            | Validation error    | `{error:...}` | No    |
| 401  | Unauthorized           | Auth missing        | `{error:...}` | No    |
| 403  | Forbidden              | Auth insufficient   | `{error:...}` | No    |
| 404  | Not Found              | Resource missing    | `{error:...}` | No    |
| 405  | Method Not Allowed     | Wrong verb          | `{error:...}` | No    |
| 406  | Not Acceptable         | MIME not supported  | `{error:...}` | No    |
| 409  | Conflict               | Duplicate resource  | `{error:...}` | No    |
| 410  | Gone                   | Resource deleted    | `{error:...}` | No    |
| 415  | Unsupported Media Type | Bad Content-Type    | `{error:...}` | No    |
| 422  | Unprocessable Entity   | Validation failed   | `{error:...}` | No    |
| 429  | Too Many Requests      | Rate limit exceeded | `{error:...}` | Yes   |
| 500  | Internal Server Error  | Unexpected          | `{error:...}` | Yes   |
| 502  | Bad Gateway            | Upstream error      | `{error:...}` | Yes   |
| 503  | Service Unavailable    | Maintenance         | `{error:...}` | Yes   |
| 504  | Gateway Timeout        | Upstream timeout    | `{error:...}` | Yes   |

---

## 4. RESPONSE FORMATS

### 4.1 Success Response Schema

```typescript
export interface SuccessResponse<T> {
  success: true;
  data: T;
  meta?: {
    pagination?: {
      page: number;
      pageSize: number;
      total: number;
      totalPages: number;
      hasNext: boolean;
      hasPrev: boolean;
    };
    [key: string]: any;
  };
}
```

### 4.2 Error Response Schema

```typescript
export interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: Record<string, string[]>;
  };
}
```

### 4.3 Pagination Response

```typescript
export interface PaginatedResponse<T> {
  success: true;
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}
```

### 4.4 API Response Type Union

```typescript
export type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

// Usage
function handleResponse<T>(res: ApiResponse<T>) {
  if (res.success) {
    return res.data;
  } else {
    throw new Error(res.error.message);
  }
}
```

---

## 5. PAGINATION PATTERNS

| Pattern    | Pros                | Cons                | Best For       | Implementation          |
| ---------- | ------------------- | ------------------- | -------------- | ----------------------- |
| Offset     | Simple, standard    | Large datasets slow | Small tables   | `?page=2&pageSize=20`   |
| Cursor     | Efficient, stable   | Complex client      | Feeds, streams | `?cursor=xyz&limit=20`  |
| Keyset     | Fast, no duplicates | Cannot jump pages   | Big tables     | `?afterId=100&limit=20` |
| Page Token | Encoded cursor      | Hard to debug       | Public APIs    | `?pageToken=abc123`     |

### 5.1 Offset Pagination Implementation

```typescript
// Query
const page = parseInt(req.query.page as string) || 1;
const pageSize = Math.min(parseInt(req.query.pageSize as string) || 20, 100);
const offset = (page - 1) * pageSize;

const [data, total] = await Promise.all([
  db.users.findMany({ skip: offset, take: pageSize }),
  db.users.count(),
]);

return {
  success: true,
  data,
  pagination: {
    page,
    pageSize,
    total,
    totalPages: Math.ceil(total / pageSize),
    hasNext: page * pageSize < total,
    hasPrev: page > 1,
  },
};
```

### 5.2 Cursor Pagination Implementation

```typescript
const cursor = req.query.cursor as string | undefined;
const limit = Math.min(parseInt(req.query.limit as string) || 20, 100);

const data = await db.users.findMany({
  take: limit + 1,
  cursor: cursor ? { id: cursor } : undefined,
  skip: cursor ? 1 : 0,
  orderBy: { id: 'asc' },
});

const hasNext = data.length > limit;
if (hasNext) data.pop();

return {
  success: true,
  data,
  pagination: {
    nextCursor: hasNext ? data[data.length - 1].id : null,
    hasNext,
  },
};
```

---

## 6. VERSIONING STRATEGIES

| Strategy | URL Example        | Header Example                        | Pros                  | Cons             | Recommendation |
| -------- | ------------------ | ------------------------------------- | --------------------- | ---------------- | -------------- |
| URL      | `/v1/users`        | —                                     | Easy, visible         | Harder to evolve | ✅ Recommended |
| Header   | `/users`           | `Accept: application/vnd.app.v1+json` | Transparent, flexible | Hard to discover | Optional       |
| Query    | `/users?version=1` | —                                     | Simple                | Bad convention   | ❌ Avoid       |

---

## 7. AUTHENTICATION PATTERNS

| Pattern      | Security | Stateless | Use Case        | Implementation                         |
| ------------ | -------- | --------- | --------------- | -------------------------------------- |
| Bearer Token | High     | ✅        | Mobile apps     | `Authorization: Bearer <token>`        |
| API Key      | Medium   | ✅        | Public APIs     | `x-api-key: <key>`                     |
| OAuth2       | High     | ✅        | User delegation | `Authorization: Bearer <access_token>` |
| JWT          | High     | ✅        | Microservices   | `Authorization: Bearer <jwt>`          |

### 7.1 JWT Middleware

```typescript
import jwt from 'jsonwebtoken';
import { Request, Response, NextFunction } from 'express';

export interface AuthRequest extends Request {
  userId?: string;
}

export function authMiddleware(req: AuthRequest, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({
      success: false,
      error: { code: 'UNAUTHORIZED', message: 'Missing token' },
    });
  }

  const token = authHeader.slice(7);
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as { userId: string };
    req.userId = payload.userId;
    next();
  } catch {
    return res.status(401).json({
      success: false,
      error: { code: 'INVALID_TOKEN', message: 'Invalid or expired token' },
    });
  }
}
```

---

## 8. RATE LIMITING

### 8.1 Standard Headers

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 50
X-RateLimit-Reset: 1675000000
Retry-After: 60
```

### 8.2 Redis Implementation

```typescript
import Redis from 'ioredis';
import { Request, Response, NextFunction } from 'express';

const redis = new Redis();

interface RateLimitOptions {
  windowMs: number;
  max: number;
}

export function rateLimit(options: RateLimitOptions) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const key = `rate:${req.ip}`;
    const count = await redis.incr(key);
    
    if (count === 1) {
      await redis.expire(key, Math.ceil(options.windowMs / 1000));
    }

    const ttl = await redis.ttl(key);
    
    res.setHeader('X-RateLimit-Limit', options.max);
    res.setHeader('X-RateLimit-Remaining', Math.max(0, options.max - count));
    res.setHeader('X-RateLimit-Reset', Date.now() + ttl * 1000);

    if (count > options.max) {
      res.setHeader('Retry-After', ttl);
      return res.status(429).json({
        success: false,
        error: { code: 'RATE_LIMIT', message: 'Too many requests' },
      });
    }

    next();
  };
}

// Usage: app.use(rateLimit({ windowMs: 60000, max: 100 }));
```

---

## 9. HATEOAS & LINKS

```typescript
interface Link {
  rel: string;
  href: string;
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
}

interface HypermediaResponse<T> {
  success: true;
  data: T;
  links: Link[];
}

// Example response
const userResponse: HypermediaResponse<User> = {
  success: true,
  data: { id: '123', name: 'John', email: 'john@example.com' },
  links: [
    { rel: 'self', href: '/users/123', method: 'GET' },
    { rel: 'update', href: '/users/123', method: 'PATCH' },
    { rel: 'delete', href: '/users/123', method: 'DELETE' },
    { rel: 'posts', href: '/users/123/posts', method: 'GET' },
  ],
};
```

---

## 10. API DOCUMENTATION (OpenAPI + Zod)

```typescript
import { z } from 'zod';
import { extendZodWithOpenApi, OpenAPIRegistry, OpenApiGeneratorV3 } from '@asteasolutions/zod-to-openapi';

extendZodWithOpenApi(z);

const registry = new OpenAPIRegistry();

// Define schemas
const UserSchema = z.object({
  id: z.string().openapi({ example: 'usr_123' }),
  name: z.string().min(2).max(100).openapi({ example: 'John Doe' }),
  email: z.string().email().openapi({ example: 'john@example.com' }),
}).openapi('User');

// Register endpoints
registry.registerPath({
  method: 'get',
  path: '/users/{id}',
  tags: ['Users'],
  request: {
    params: z.object({ id: z.string() }),
  },
  responses: {
    200: {
      description: 'User found',
      content: { 'application/json': { schema: UserSchema } },
    },
    404: { description: 'User not found' },
  },
});

// Generate OpenAPI document
const generator = new OpenApiGeneratorV3(registry.definitions);
const openApiDoc = generator.generateDocument({
  openapi: '3.1.0',
  info: { title: 'My API', version: '1.0.0' },
  servers: [{ url: 'https://api.example.com/v1' }],
});
```

---

## 11. EXPRESS CRUD EXAMPLE

```typescript
import express from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

// Schemas
const CreateUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
});

const UpdateUserSchema = CreateUserSchema.partial();

// Validation middleware
function validate<T>(schema: z.ZodSchema<T>) {
  return (req: express.Request, res: express.Response, next: express.NextFunction) => {
    const result = schema.safeParse(req.body);
    if (!result.success) {
      return res.status(422).json({
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input',
          details: result.error.flatten().fieldErrors,
        },
      });
    }
    req.body = result.data;
    next();
  };
}

// Routes
app.get('/users', async (req, res) => {
  const users = await db.users.findMany();
  res.json({ success: true, data: users });
});

app.get('/users/:id', async (req, res) => {
  const user = await db.users.findUnique({ where: { id: req.params.id } });
  if (!user) {
    return res.status(404).json({
      success: false,
      error: { code: 'NOT_FOUND', message: 'User not found' },
    });
  }
  res.json({ success: true, data: user });
});

app.post('/users', validate(CreateUserSchema), async (req, res) => {
  const user = await db.users.create({ data: req.body });
  res.status(201).json({ success: true, data: user });
});

app.patch('/users/:id', validate(UpdateUserSchema), async (req, res) => {
  const user = await db.users.update({
    where: { id: req.params.id },
    data: req.body,
  });
  res.json({ success: true, data: user });
});

app.delete('/users/:id', async (req, res) => {
  await db.users.delete({ where: { id: req.params.id } });
  res.status(204).send();
});

// Error handler
app.use((err: Error, req: express.Request, res: express.Response, next: express.NextFunction) => {
  console.error(err);
  res.status(500).json({
    success: false,
    error: { code: 'INTERNAL_ERROR', message: 'Something went wrong' },
  });
});

app.listen(3000);
```

---

## 12. NEXT.JS API ROUTES (App Router)

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
});

export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const page = parseInt(searchParams.get('page') || '1');
  const pageSize = Math.min(parseInt(searchParams.get('pageSize') || '20'), 100);

  const users = await db.users.findMany({
    skip: (page - 1) * pageSize,
    take: pageSize,
  });

  return NextResponse.json({ success: true, data: users });
}

export async function POST(req: NextRequest) {
  const body = await req.json();
  const result = CreateUserSchema.safeParse(body);

  if (!result.success) {
    return NextResponse.json(
      {
        success: false,
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input',
          details: result.error.flatten().fieldErrors,
        },
      },
      { status: 422 }
    );
  }

  const user = await db.users.create({ data: result.data });
  return NextResponse.json({ success: true, data: user }, { status: 201 });
}
```

```typescript
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  const user = await db.users.findUnique({ where: { id: params.id } });

  if (!user) {
    return NextResponse.json(
      { success: false, error: { code: 'NOT_FOUND', message: 'User not found' } },
      { status: 404 }
    );
  }

  return NextResponse.json({ success: true, data: user });
}

export async function DELETE(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  await db.users.delete({ where: { id: params.id } });
  return new NextResponse(null, { status: 204 });
}
```

---

## 13. API DESIGN CHECKLIST

| Category | Item |
| -------- | ---- |
| URL Design | ✅ Plural resource naming |
| URL Design | ✅ No verbs in URL |
| URL Design | ✅ Use hyphens, not underscores |
| URL Design | ✅ Lowercase URLs only |
| URL Design | ✅ No file extensions |
| URL Design | ✅ Path for IDs, query for filters |
| Versioning | ✅ Strategy defined (URL recommended) |
| HTTP Methods | ✅ Correct verbs for operations |
| Status Codes | ✅ Consistent status codes |
| Validation | ✅ Input validation on all endpoints |
| Response | ✅ Consistent response schema |
| Response | ✅ Success/Error differentiation |
| Pagination | ✅ Pagination implemented |
| Filtering | ✅ Filtering & sorting available |
| Security | ✅ Rate limiting applied |
| Security | ✅ Authentication enforced |
| Error Handling | ✅ Global error handler |
| Observability | ✅ Logging & metrics |
| Documentation | ✅ OpenAPI docs up-to-date |
| HATEOAS | ⚠️ Links added (optional) |


---

## 14. REST vs GRAPHQL vs tRPC COMPARISON

### 14.1 Technology Decision Matrix

| Aspect | REST | GraphQL | tRPC | gRPC |
|--------|------|---------|------|------|
| Protocol | HTTP/1.1+ | HTTP/1.1+ | HTTP/1.1+ | HTTP/2 |
| Data Format | JSON | JSON | JSON | Protobuf |
| Schema | OpenAPI (optional) | SDL (required) | TypeScript (inferred) | .proto (required) |
| Type Safety | ❌ Manual | ⚠️ Codegen needed | ✅ Native | ✅ Native |
| Overfetching | ❌ Common | ✅ Solved | ✅ Solved | ✅ Solved |
| Underfetching | ❌ Common | ✅ Solved | ⚠️ Depends | ⚠️ Depends |
| HTTP Caching | ✅ Native | ❌ Complex | ❌ Custom | ❌ None |
| File Upload | ✅ Native | ⚠️ Multipart spec | ✅ Native | ⚠️ Streaming |
| Real-time | ⚠️ SSE/WebSocket | ✅ Subscriptions | ✅ Subscriptions | ✅ Streaming |
| Learning Curve | 🟢 Low | 🟡 Medium | 🟢 Low | 🔴 High |
| Tooling | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Best For | Public APIs | Complex UIs | Full-stack TS | Microservices |

### 14.2 When to Use What

| Scenario | Recommended | Why |
|----------|-------------|-----|
| Public API for third parties | REST | Universal, cacheable, well-documented |
| Mobile app with limited bandwidth | GraphQL | Fetch exactly what's needed |
| Full-stack TypeScript monorepo | tRPC | End-to-end type safety, no codegen |
| Internal microservices | gRPC | High performance, strong contracts |
| Real-time dashboard | GraphQL | Subscriptions built-in |
| CRUD-heavy admin panel | REST or tRPC | Simple, predictable |
| Serverless functions | tRPC or REST | Simple deployment |
| Rapid prototyping | tRPC | Zero boilerplate |

---

## 15. tRPC IMPLEMENTATION

### 15.1 Server Setup (Next.js App Router)

```typescript
// server/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server';
import { ZodError } from 'zod';
import superjson from 'superjson';
import { getServerSession } from 'next-auth';

export interface Context {
  session: Awaited<ReturnType<typeof getServerSession>> | null;
  headers: Headers;
}

export const createTRPCContext = async (opts: { headers: Headers }): Promise<Context> => {
  const session = await getServerSession();
  return { session, headers: opts.headers };
};

const t = initTRPC.context<Context>().create({
  transformer: superjson,
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        zodError: error.cause instanceof ZodError ? error.cause.flatten() : null,
      },
    };
  },
});

export const router = t.router;
export const publicProcedure = t.procedure;
export const middleware = t.middleware;

// Auth middleware
const enforceAuth = middleware(async ({ ctx, next }) => {
  if (!ctx.session?.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({ ctx: { session: ctx.session, user: ctx.session.user } });
});

export const protectedProcedure = t.procedure.use(enforceAuth);
```

### 15.2 Router Definition

```typescript
// server/routers/user.ts
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';
import { prisma } from '@/lib/prisma';
import { TRPCError } from '@trpc/server';

export const userRouter = router({
  getById: publicProcedure
    .input(z.object({ id: z.string().cuid() }))
    .query(async ({ input }) => {
      const user = await prisma.user.findUnique({
        where: { id: input.id },
        select: { id: true, name: true, email: true, image: true },
      });
      if (!user) throw new TRPCError({ code: 'NOT_FOUND' });
      return user;
    }),

  me: protectedProcedure.query(async ({ ctx }) => {
    return prisma.user.findUnique({
      where: { id: ctx.user.id },
      include: { profile: true },
    });
  }),

  update: protectedProcedure
    .input(z.object({
      name: z.string().min(2).max(100).optional(),
      email: z.string().email().optional(),
    }))
    .mutation(async ({ ctx, input }) => {
      return prisma.user.update({
        where: { id: ctx.user.id },
        data: input,
      });
    }),

  list: publicProcedure
    .input(z.object({
      cursor: z.string().cuid().optional(),
      limit: z.number().min(1).max(100).default(20),
    }))
    .query(async ({ input }) => {
      const users = await prisma.user.findMany({
        take: input.limit + 1,
        cursor: input.cursor ? { id: input.cursor } : undefined,
        orderBy: { createdAt: 'desc' },
      });

      let nextCursor: string | undefined;
      if (users.length > input.limit) {
        nextCursor = users.pop()?.id;
      }
      return { items: users, nextCursor };
    }),
});
```

### 15.3 Client Setup

```typescript
// lib/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '@/server/routers/_app';

export const trpc = createTRPCReact<AppRouter>();

// providers/trpc-provider.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { httpBatchLink } from '@trpc/client';
import { useState } from 'react';
import superjson from 'superjson';
import { trpc } from '@/lib/trpc';

export function TRPCProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());
  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: '/api/trpc',
          transformer: superjson,
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
}
```


---

## 16. OPENAPI 3.1 SPECIFICATION

### 16.1 OpenAPI Template Structure

```yaml
# openapi.yaml
openapi: 3.1.0
info:
  title: My API
  version: 1.0.0
  description: Production-ready REST API

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: http://localhost:3000/api/v1
    description: Development

paths:
  /users:
    get:
      tags: [Users]
      summary: List all users
      operationId: listUsers
      security:
        - BearerAuth: []
      parameters:
        - $ref: '#/components/parameters/PageParam'
        - $ref: '#/components/parameters/LimitParam'
      responses:
        '200':
          description: Users list
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'

components:
  schemas:
    User:
      type: object
      required: [id, email, name]
      properties:
        id:
          type: string
          format: cuid
        email:
          type: string
          format: email
        name:
          type: string

    UserListResponse:
      type: object
      properties:
        success:
          type: boolean
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'

    Pagination:
      type: object
      properties:
        page:
          type: integer
        pageSize:
          type: integer
        total:
          type: integer
        hasNext:
          type: boolean

  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        default: 1

    LimitParam:
      name: limit
      in: query
      schema:
        type: integer
        default: 20
        maximum: 100

  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

---

## 17. WEBHOOK IMPLEMENTATION

### 17.1 Webhook Event Types

| Event | Payload | Retry Policy | Idempotency |
|-------|---------|--------------|-------------|
| user.created | Full user object | 3x exponential | webhook_id |
| user.updated | User with changed fields | 3x exponential | webhook_id |
| user.deleted | User ID only | 3x exponential | webhook_id |
| order.placed | Order with items | 5x exponential | order_id |
| order.paid | Order with payment | 5x exponential | payment_id |
| subscription.created | Subscription details | 3x exponential | subscription_id |
| subscription.cancelled | Subscription ID + reason | 3x exponential | webhook_id |

### 17.2 Webhook Service Implementation

```typescript
// lib/webhooks/webhook-service.ts
import crypto from 'crypto';

interface WebhookPayload {
  id: string;
  type: string;
  timestamp: string;
  data: Record<string, unknown>;
}

export class WebhookService {
  private static readonly MAX_RETRIES = 3;
  private static readonly RETRY_DELAYS = [1000, 5000, 30000];

  static createPayload(type: string, data: Record<string, unknown>): WebhookPayload {
    return {
      id: crypto.randomUUID(),
      type,
      timestamp: new Date().toISOString(),
      data,
    };
  }

  static sign(payload: string, secret: string): string {
    return crypto.createHmac('sha256', secret).update(payload, 'utf8').digest('hex');
  }

  static verify(payload: string, signature: string, secret: string): boolean {
    const expected = this.sign(payload, secret);
    try {
      return crypto.timingSafeEqual(
        Buffer.from(signature, 'hex'),
        Buffer.from(expected, 'hex')
      );
    } catch {
      return false;
    }
  }

  static async send(url: string, secret: string, payload: WebhookPayload) {
    const body = JSON.stringify(payload);
    const signature = this.sign(body, secret);

    for (let attempt = 0; attempt <= this.MAX_RETRIES; attempt++) {
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'X-Webhook-ID': payload.id,
            'X-Webhook-Signature': `sha256=${signature}`,
          },
          body,
          signal: AbortSignal.timeout(30000),
        });

        if (response.ok) return { success: true, statusCode: response.status };
        if (response.status >= 400 && response.status < 500 && response.status !== 429) {
          return { success: false, statusCode: response.status };
        }
        if (attempt < this.MAX_RETRIES) {
          await new Promise(r => setTimeout(r, this.RETRY_DELAYS[attempt]));
        }
      } catch (error) {
        if (attempt === this.MAX_RETRIES) {
          return { success: false, error: String(error) };
        }
        await new Promise(r => setTimeout(r, this.RETRY_DELAYS[attempt]));
      }
    }
    return { success: false, error: 'Max retries exceeded' };
  }
}
```


---

## 18. API TESTING

### 18.1 Testing Strategy Matrix

| Test Type | Tool | Scope | Run Frequency | Duration |
|-----------|------|-------|---------------|----------|
| Unit | Vitest/Jest | Handler logic | Every commit | <1s |
| Integration | Supertest | Full HTTP cycle | Every commit | 5-30s |
| Contract | Pact | API compatibility | Daily/PR | 1-5m |
| E2E | Playwright | Full flow | Daily | 5-15m |
| Load | k6/Artillery | Performance | Weekly/Release | 10-30m |
| Security | OWASP ZAP | Vulnerabilities | Weekly | 30m-2h |

### 18.2 API Route Unit Tests

```typescript
// __tests__/api/users.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { GET, POST } from '@/app/api/users/route';
import { NextRequest } from 'next/server';
import { prisma } from '@/lib/prisma';

vi.mock('@/lib/prisma', () => ({
  prisma: {
    user: {
      findMany: vi.fn(),
      create: vi.fn(),
    },
  },
}));

describe('GET /api/users', () => {
  beforeEach(() => vi.clearAllMocks());

  it('returns paginated users list', async () => {
    const mockUsers = [
      { id: '1', name: 'User 1', email: 'user1@test.com' },
      { id: '2', name: 'User 2', email: 'user2@test.com' },
    ];

    vi.mocked(prisma.user.findMany).mockResolvedValue(mockUsers);

    const request = new NextRequest('http://localhost/api/users?page=1&limit=20');
    const response = await GET(request);
    const data = await response.json();

    expect(response.status).toBe(200);
    expect(data.success).toBe(true);
    expect(data.data).toHaveLength(2);
  });
});

describe('POST /api/users', () => {
  it('creates user with valid data', async () => {
    const newUser = { id: 'new-id', email: 'new@test.com', name: 'New User' };
    vi.mocked(prisma.user.create).mockResolvedValue(newUser);

    const request = new NextRequest('http://localhost/api/users', {
      method: 'POST',
      body: JSON.stringify({ email: 'new@test.com', name: 'New User' }),
    });

    const response = await POST(request);
    expect(response.status).toBe(201);
  });

  it('returns 422 for invalid email', async () => {
    const request = new NextRequest('http://localhost/api/users', {
      method: 'POST',
      body: JSON.stringify({ email: 'invalid-email', name: 'Test' }),
    });

    const response = await POST(request);
    expect(response.status).toBe(422);
  });
});
```

---

## 19. API GATEWAY PATTERNS

### 19.1 Gateway Pattern Comparison

| Pattern | Use Case | Complexity | Latency | Best For |
|---------|----------|------------|---------|----------|
| API Gateway | Single entry point | Medium | +5-20ms | Microservices |
| BFF (Backend for Frontend) | Client-specific APIs | High | +10-30ms | Mobile + Web |
| GraphQL Federation | Unified graph | High | +15-50ms | Large organizations |
| Service Mesh | Inter-service | Very High | +2-5ms | Kubernetes |
| Edge Functions | Low latency | Low | +1-5ms | Global distribution |

### 19.2 Rate Limiting Implementation

```typescript
// middleware/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';
import { NextRequest, NextResponse } from 'next/server';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, '1 m'), // 100 req/min
  analytics: true,
});

export async function rateLimitMiddleware(req: NextRequest) {
  const ip = req.ip ?? req.headers.get('x-forwarded-for') ?? '127.0.0.1';
  const { success, limit, remaining, reset } = await ratelimit.limit(ip);

  if (!success) {
    return NextResponse.json(
      { error: 'Too many requests' },
      {
        status: 429,
        headers: {
          'X-RateLimit-Limit': limit.toString(),
          'X-RateLimit-Remaining': remaining.toString(),
          'X-RateLimit-Reset': reset.toString(),
          'Retry-After': Math.ceil((reset - Date.now()) / 1000).toString(),
        },
      }
    );
  }

  return null; // Continue to handler
}
```

---

## 20. REST API EXPANSION CHECKLIST

```
API ARCHITECTURE
□ REST vs tRPC decision documented
□ OpenAPI 3.1 spec complete
□ Versioning strategy defined
□ Error response schema standardized
□ Pagination implemented

tRPC (if applicable)
□ Server context configured
□ Router structure defined
□ Auth middleware implemented
□ Client provider setup
□ Type inference working

WEBHOOKS
□ Event types documented
□ Signature verification
□ Retry policy defined
□ Idempotency keys
□ Delivery logging

TESTING
□ Unit tests for handlers
□ Integration tests with HTTP
□ Contract tests (if multi-team)
□ Load testing baseline
□ Security scan scheduled

GATEWAY / INFRASTRUCTURE
□ Rate limiting configured
□ CORS policy set
□ Request logging enabled
□ Response caching (where applicable)
□ Health check endpoint

DOCUMENTATION
□ OpenAPI spec up-to-date
□ Examples for all endpoints
□ Error codes documented
□ Authentication flow explained
□ Changelog maintained
```
