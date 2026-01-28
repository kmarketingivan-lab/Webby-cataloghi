# CATALOGO REST API DESIGN v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: URL design, HTTP methods, Status codes, Response formats, Pagination, Versioning, Authentication

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
