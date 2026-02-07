# CATALOGO-REST-API-DESIGN-v2

Catálogo Completo: REST API Design per Next.js 14
§ REST FUNDAMENTALS
HTTP Methods Semantics
typescript
/**
 * HTTP Methods e loro semantica RESTful
 */
export const HTTP_METHODS = {
  GET: {
    method: 'GET',
    description: 'Recupera una rappresentazione della risorsa',
    idempotent: true,
    safe: true,
    requestBody: false,
    responseBody: true
  },
  POST: {
    method: 'POST',
    description: 'Crea una nuova risorsa o esegue un\'azione',
    idempotent: false,
    safe: false,
    requestBody: true,
    responseBody: true
  },
  PUT: {
    method: 'PUT',
    description: 'Sostituisce completamente una risorsa',
    idempotent: true,
    safe: false,
    requestBody: true,
    responseBody: true
  },
  PATCH: {
    method: 'PATCH',
    description: 'Modifica parzialmente una risorsa',
    idempotent: false,
    safe: false,
    requestBody: true,
    responseBody: true
  },
  DELETE: {
    method: 'DELETE',
    description: 'Rimuove una risorsa',
    idempotent: true,
    safe: false,
    requestBody: false,
    responseBody: true
  },
  HEAD: {
    method: 'HEAD',
    description: 'Recupera solo gli header della risposta',
    idempotent: true,
    safe: true,
    requestBody: false,
    responseBody: false
  },
  OPTIONS: {
    method: 'OPTIONS',
    description: 'Descrive le opzioni di comunicazione',
    idempotent: true,
    safe: true,
    requestBody: false,
    responseBody: true
  }
} as const;
Status Codes Reference
typescript
/**
 * Codici di stato HTTP organizzati per categoria
 */
export const STATUS_CODES = {
  // 2xx Success
  OK: 200,
  CREATED: 201,
  ACCEPTED: 202,
  NO_CONTENT: 204,
  RESET_CONTENT: 205,
  PARTIAL_CONTENT: 206,

  // 3xx Redirection
  MOVED_PERMANENTLY: 301,
  FOUND: 302,
  SEE_OTHER: 303,
  NOT_MODIFIED: 304,
  TEMPORARY_REDIRECT: 307,
  PERMANENT_REDIRECT: 308,

  // 4xx Client Errors
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  METHOD_NOT_ALLOWED: 405,
  NOT_ACCEPTABLE: 406,
  CONFLICT: 409,
  GONE: 410,
  UNPROCESSABLE_ENTITY: 422,
  TOO_MANY_REQUESTS: 429,

  // 5xx Server Errors
  INTERNAL_SERVER_ERROR: 500,
  NOT_IMPLEMENTED: 501,
  BAD_GATEWAY: 502,
  SERVICE_UNAVAILABLE: 503,
  GATEWAY_TIMEOUT: 504
} as const;

/**
 * Mappa codici di stato a messaggi descrittivi
 */
export const STATUS_MESSAGES: Record<number, string> = {
  200: 'OK',
  201: 'Created',
  202: 'Accepted',
  204: 'No Content',
  400: 'Bad Request',
  401: 'Unauthorized',
  403: 'Forbidden',
  404: 'Not Found',
  405: 'Method Not Allowed',
  409: 'Conflict',
  422: 'Unprocessable Entity',
  429: 'Too Many Requests',
  500: 'Internal Server Error',
  503: 'Service Unavailable'
};
Idempotency
typescript
/**
 * Implementazione idempotenza con chiavi idempotenti
 */
export class IdempotencyManager {
  private static readonly store = new Map<string, { response: any; timestamp: number }>();
  private static readonly TTL = 24 * 60 * 60 * 1000; // 24 ore

  static async executeWithIdempotency<T>(
    key: string,
    operation: () => Promise<T>
  ): Promise<T> {
    this.cleanupExpired();
    
    const cached = this.store.get(key);
    if (cached) {
      return cached.response;
    }
    
    const result = await operation();
    this.store.set(key, {
      response: result,
      timestamp: Date.now()
    });
    
    return result;
  }

  static generateKey(
    method: string,
    path: string,
    userId: string,
    bodyHash?: string
  ): string {
    return `${method}:${path}:${userId}:${bodyHash || ''}`;
  }

  private static cleanupExpired(): void {
    const now = Date.now();
    for (const [key, value] of this.store.entries()) {
      if (now - value.timestamp > this.TTL) {
        this.store.delete(key);
      }
    }
  }
}

// Middleware per gestire header Idempotency-Key
export const idempotencyMiddleware = async (
  request: Request,
  next: () => Promise<Response>
) => {
  const idempotencyKey = request.headers.get('Idempotency-Key');
  
  if (!idempotencyKey) {
    return next();
  }
  
  const userId = request.headers.get('X-User-ID') || 'anonymous';
  const method = request.method;
  const path = new URL(request.url).pathname;
  
  // Hash del body per chiave più precisa
  const bodyText = await request.clone().text();
  const bodyHash = bodyText ? await crypto.subtle.digest('SHA-256', new TextEncoder().encode(bodyText)).then(hash => Array.from(new Uint8Array(hash)).map(b => b.toString(16).padStart(2, '0')).join('')) : undefined;
  
  const key = IdempotencyManager.generateKey(method, path, userId, bodyHash);
  
  return IdempotencyManager.executeWithIdempotency(key, next);
};
HATEOAS Basics
typescript
/**
 * Interfaccia per link HATEOAS
 */
export interface HALink {
  href: string;
  rel: string;
  method?: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  title?: string;
  type?: string;
  templated?: boolean;
}

/**
 * Wrapper per risorse con link HATEOAS
 */
export interface HATEOASResource<T> {
  _links: {
    self: HALink;
    [key: string]: HALink | HALink[];
  };
  _embedded?: {
    [key: string]: HATEOASResource<any> | HATEOASResource<any>[];
  };
  data: T;
}

/**
 * Builder per risorse HATEOAS
 */
export class HATEOASBuilder<T> {
  private resource: Partial<HATEOASResource<T>> = {
    _links: {} as any
  };

  constructor(private baseUrl: string) {}

  withData(data: T): this {
    this.resource.data = data;
    return this;
  }

  withSelfLink(path: string, method: string = 'GET'): this {
    this.resource._links!.self = {
      href: `${this.baseUrl}${path}`,
      rel: 'self',
      method: method as any
    };
    return this;
  }

  withLink(rel: string, path: string, method?: string, templated?: boolean): this {
    this.resource._links![rel] = {
      href: `${this.baseUrl}${path}`,
      rel,
      method: method as any,
      templated
    };
    return this;
  }

  withCollectionLink(rel: string, path: string): this {
    this.resource._links![rel] = {
      href: `${this.baseUrl}${path}`,
      rel: 'collection',
      method: 'GET'
    };
    return this;
  }

  build(): HATEOASResource<T> {
    if (!this.resource._links!.self) {
      throw new Error('Self link is required');
    }
    return this.resource as HATEOASResource<T>;
  }
}

// Esempio di utilizzo
export const createUserResource = (user: any, baseUrl: string): HATEOASResource<any> => {
  return new HATEOASBuilder(baseUrl)
    .withData(user)
    .withSelfLink(`/users/${user.id}`)
    .withLink('update', `/users/${user.id}`, 'PUT')
    .withLink('delete', `/users/${user.id}`, 'DELETE')
    .withCollectionLink('users', '/users')
    .build();
};
Content Negotiation
typescript
/**
 * Supporto per content negotiation
 */
export const SUPPORTED_CONTENT_TYPES = {
  JSON: 'application/json',
  JSON_API: 'application/vnd.api+json',
  HAL_JSON: 'application/hal+json',
  FORM_URLENCODED: 'application/x-www-form-urlencoded',
  MULTIPART_FORM: 'multipart/form-data',
  XML: 'application/xml',
  TEXT: 'text/plain'
} as const;

export class ContentNegotiator {
  static getBestMatch(
    acceptHeader: string | null,
    availableTypes: string[]
  ): string | null {
    if (!acceptHeader) {
      return availableTypes[0] || null;
    }

    const accepted = this.parseAcceptHeader(acceptHeader);
    
    for (const { type, q } of accepted) {
      if (availableTypes.includes(type)) {
        return type;
      }
      
      // Cerca pattern (es. application/*)
      if (type.includes('/*')) {
        const mimeType = type.replace('/*', '');
        const match = availableTypes.find(t => t.startsWith(mimeType));
        if (match) return match;
      }
    }
    
    return null;
  }

  private static parseAcceptHeader(header: string): Array<{ type: string; q: number }> {
    return header.split(',')
      .map(part => {
        const [type, ...params] = part.trim().split(';');
        const q = params.find(p => p.startsWith('q='));
        return {
          type,
          q: q ? parseFloat(q.split('=')[1]) : 1.0
        };
      })
      .sort((a, b) => b.q - a.q);
  }

  static isContentTypeSupported(contentType: string | null): boolean {
    if (!contentType) return false;
    const type = contentType.split(';')[0].trim();
    return Object.values(SUPPORTED_CONTENT_TYPES).includes(type as any);
  }
}

// Middleware per content negotiation
export const contentNegotiationMiddleware = async (
  request: Request,
  next: (contentType: string) => Promise<Response>
) => {
  const acceptHeader = request.headers.get('Accept');
  const availableTypes = Object.values(SUPPORTED_CONTENT_TYPES);
  
  const bestType = ContentNegotiator.getBestMatch(acceptHeader, availableTypes);
  
  if (!bestType) {
    return new Response(
      JSON.stringify({
        error: 'Not Acceptable',
        message: 'None of the requested content types are supported'
      }),
      {
        status: 406,
        headers: {
          'Content-Type': 'application/json',
          'Accept': availableTypes.join(', ')
        }
      }
    );
  }
  
  return next(bestType);
};
§ NEXT.JS ROUTE HANDLERS
GET, POST, PUT, PATCH, DELETE Handlers
typescript
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

/**
 * Template per Route Handlers in Next.js 14
 */
export async function GET(
  request: NextRequest,
  context: { params: Promise<Record<string, string>> }
) {
  try {
    const params = await context.params;
    const searchParams = request.nextUrl.searchParams;
    
    // Esempio: GET /api/users?page=1&limit=10
    const page = parseInt(searchParams.get('page') || '1');
    const limit = parseInt(searchParams.get('limit') || '10');
    
    // Logica business
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
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: 'Internal Server Error',
        message: error instanceof Error ? error.message : 'Unknown error'
      },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    // Validazione con Zod
    const userSchema = z.object({
      name: z.string().min(2),
      email: z.string().email(),
      age: z.number().min(18).optional()
    });
    
    const validatedData = userSchema.parse(body);
    
    // Creazione risorsa
    const newUser = await createUser(validatedData);
    
    return NextResponse.json(
      {
        success: true,
        data: newUser,
        message: 'User created successfully'
      },
      { status: 201 }
    );
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          success: false,
          error: 'Validation Error',
          details: error.errors
        },
        { status: 422 }
      );
    }
    
    return NextResponse.json(
      {
        success: false,
        error: 'Internal Server Error'
      },
      { status: 500 }
    );
  }
}

export async function PUT(
  request: NextRequest,
  context: { params: Promise<Record<string, string>> }
) {
  try {
    const params = await context.params;
    const body = await request.json();
    
    // Sostituzione completa
    const updatedResource = await replaceResource(params.id, body);
    
    return NextResponse.json({
      success: true,
      data: updatedResource,
      message: 'Resource replaced successfully'
    });
  } catch (error) {
    // Gestione errori
  }
}

export async function PATCH(
  request: NextRequest,
  context: { params: Promise<Record<string, string>> }
) {
  try {
    const params = await context.params;
    const body = await request.json();
    
    // Aggiornamento parziale
    const patchedResource = await updatePartial(params.id, body);
    
    return NextResponse.json({
      success: true,
      data: patchedResource,
      message: 'Resource updated successfully'
    });
  } catch (error) {
    // Gestione errori
  }
}

export async function DELETE(
  request: NextRequest,
  context: { params: Promise<Record<string, string>> }
) {
  try {
    const params = await context.params;
    
    await deleteResource(params.id);
    
    return new NextResponse(null, { status: 204 });
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: 'Not Found',
        message: 'Resource not found'
      },
      { status: 404 }
    );
  }
}

// Funzioni helper di esempio
async function getUsers(params: { page: number; limit: number }) {
  return [];
}

async function createUser(data: any) {
  return { id: '1', ...data };
}

async function replaceResource(id: string, data: any) {
  return { id, ...data };
}

async function updatePartial(id: string, data: any) {
  return { id, ...data };
}

async function deleteResource(id: string) {
  // Logica eliminazione
}
Request Parsing (params, query, body)
typescript
import { NextRequest } from 'next/server';
import { z } from 'zod';

/**
 * Utility per parsing richieste
 */
export class RequestParser {
  static async parseBody<T>(request: NextRequest, schema: z.ZodSchema<T>): Promise<T> {
    const contentType = request.headers.get('content-type');
    
    if (contentType?.includes('application/json')) {
      const body = await request.json();
      return schema.parse(body);
    }
    
    if (contentType?.includes('application/x-www-form-urlencoded')) {
      const formData = await request.formData();
      const body = Object.fromEntries(formData.entries());
      return schema.parse(body);
    }
    
    if (contentType?.includes('multipart/form-data')) {
      const formData = await request.formData();
      const body: Record<string, any> = {};
      
      for (const [key, value] of formData.entries()) {
        if (value instanceof File) {
          body[key] = {
            name: value.name,
            type: value.type,
            size: value.size
          };
        } else {
          body[key] = value;
        }
      }
      
      return schema.parse(body);
    }
    
    throw new Error('Unsupported content type');
  }

  static parseQuery(request: NextRequest, schema: z.ZodSchema<any>) {
    const searchParams = request.nextUrl.searchParams;
    const query: Record<string, string | string[]> = {};
    
    for (const [key, value] of searchParams.entries()) {
      if (query[key]) {
        if (Array.isArray(query[key])) {
          (query[key] as string[]).push(value);
        } else {
          query[key] = [query[key] as string, value];
        }
      } else {
        query[key] = value;
      }
    }
    
    return schema.parse(query);
  }

  static parseParams(params: Record<string, string>, schema: z.ZodSchema<any>) {
    return schema.parse(params);
  }

  static getPaginationParams(request: NextRequest) {
    const searchParams = request.nextUrl.searchParams;
    const page = Math.max(1, parseInt(searchParams.get('page') || '1'));
    const limit = Math.min(100, Math.max(1, parseInt(searchParams.get('limit') || '20')));
    const offset = (page - 1) * limit;
    
    return { page, limit, offset };
  }
}

/**
 * Schema per query parameters comuni
 */
export const paginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.string().optional(),
  order: z.enum(['asc', 'desc']).default('desc')
});

export const filterSchema = z.object({
  search: z.string().optional(),
  status: z.string().optional(),
  from: z.string().datetime().optional(),
  to: z.string().datetime().optional()
});

// Esempio di utilizzo in Route Handler
export async function GET(request: NextRequest) {
  try {
    const query = RequestParser.parseQuery(request, paginationSchema.merge(filterSchema));
    const pagination = RequestParser.getPaginationParams(request);
    
    return NextResponse.json({
      query,
      pagination
    });
  } catch (error) {
    // Gestione errori
  }
}
Response Formatting
typescript
import { NextResponse } from 'next/server';

/**
 * Formato standard per risposte API
 */
export interface ApiResponse<T = any> {
  success: boolean;
  data?: T;
  error?: string;
  message?: string;
  meta?: {
    timestamp: string;
    version: string;
    [key: string]: any;
  };
  pagination?: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
  links?: {
    self: string;
    next?: string;
    prev?: string;
    first?: string;
    last?: string;
  };
}

/**
 * Builder per risposte API consistenti
 */
export class ApiResponseBuilder {
  private response: Partial<ApiResponse> = {
    success: true,
    meta: {
      timestamp: new Date().toISOString(),
      version: '1.0'
    }
  };

  static success<T>(data: T, message?: string): ApiResponse<T> {
    return {
      success: true,
      data,
      message,
      meta: {
        timestamp: new Date().toISOString(),
        version: '1.0'
      }
    };
  }

  static error(error: string, message?: string, details?: any): ApiResponse {
    return {
      success: false,
      error,
      message,
      data: details,
      meta: {
        timestamp: new Date().toISOString(),
        version: '1.0'
      }
    };
  }

  static paginated<T>(
    data: T[],
    page: number,
    limit: number,
    total: number,
    baseUrl: string
  ): ApiResponse<T[]> {
    const totalPages = Math.ceil(total / limit);
    
    return {
      success: true,
      data,
      pagination: {
        page,
        limit,
        total,
        totalPages,
        hasNext: page < totalPages,
        hasPrev: page > 1
      },
      links: {
        self: `${baseUrl}?page=${page}&limit=${limit}`,
        first: `${baseUrl}?page=1&limit=${limit}`,
        last: `${baseUrl}?page=${totalPages}&limit=${limit}`,
        next: page < totalPages ? `${baseUrl}?page=${page + 1}&limit=${limit}` : undefined,
        prev: page > 1 ? `${baseUrl}?page=${page - 1}&limit=${limit}` : undefined
      },
      meta: {
        timestamp: new Date().toISOString(),
        version: '1.0'
      }
    };
  }

  withData<T>(data: T): this {
    this.response.data = data;
    return this;
  }

  withMessage(message: string): this {
    this.response.message = message;
    return this;
  }

  withPagination(
    page: number,
    limit: number,
    total: number,
    baseUrl: string
  ): this {
    const totalPages = Math.ceil(total / limit);
    
    this.response.pagination = {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1
    };
    
    this.response.links = {
      self: `${baseUrl}?page=${page}&limit=${limit}`,
      first: `${baseUrl}?page=1&limit=${limit}`,
      last: totalPages > 0 ? `${baseUrl}?page=${totalPages}&limit=${limit}` : `${baseUrl}?page=1&limit=${limit}`,
      next: page < totalPages ? `${baseUrl}?page=${page + 1}&limit=${limit}` : undefined,
      prev: page > 1 ? `${baseUrl}?page=${page - 1}&limit=${limit}` : undefined
    };
    
    return this;
  }

  withMeta(key: string, value: any): this {
    if (!this.response.meta) {
      this.response.meta = {};
    }
    this.response.meta[key] = value;
    return this;
  }

  build(): ApiResponse {
    return this.response as ApiResponse;
  }
}

/**
 * Wrapper per NextResponse con formato standard
 */
export class ApiResponseFormatter {
  static json<T>(
    data: ApiResponse<T>,
    status: number = 200,
    headers?: Record<string, string>
  ): NextResponse {
    const response = NextResponse.json(data, { status });
    
    // Headers standard
    response.headers.set('Content-Type', 'application/json');
    response.headers.set('X-API-Version', '1.0');
    response.headers.set('X-Response-Time', Date.now().toString());
    
    // Headers personalizzati
    if (headers) {
      Object.entries(headers).forEach(([key, value]) => {
        response.headers.set(key, value);
      });
    }
    
    return response;
  }

  static success<T>(data: T, status: number = 200, message?: string) {
    return this.json(ApiResponseBuilder.success(data, message), status);
  }

  static error(
    error: string,
    status: number = 500,
    message?: string,
    details?: any
  ) {
    return this.json(
      ApiResponseBuilder.error(error, message, details),
      status
    );
  }

  static paginated<T>(
    data: T[],
    page: number,
    limit: number,
    total: number,
    request: Request
  ) {
    const url = new URL(request.url);
    const baseUrl = `${url.origin}${url.pathname}`;
    
    return this.json(
      ApiResponseBuilder.paginated(data, page, limit, total, baseUrl),
      200
    );
  }
}
Headers Handling
typescript
import { NextRequest, NextResponse } from 'next/server';

/**
 * Gestione avanzata degli header
 */
export class HeaderManager {
  // Header standardizzati
  static readonly STANDARD_HEADERS = {
    CORS: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, PATCH, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization, X-API-Key, Idempotency-Key',
      'Access-Control-Max-Age': '86400'
    },
    SECURITY: {
      'X-Content-Type-Options': 'nosniff',
      'X-Frame-Options': 'DENY',
      'X-XSS-Protection': '1; mode=block',
      'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
      'Referrer-Policy': 'strict-origin-when-cross-origin'
    },
    CACHING: {
      'Cache-Control': 'no-store, no-cache, must-revalidate',
      'Pragma': 'no-cache',
      'Expires': '0'
    },
    API_INFO: {
      'X-API-Version': '1.0',
      'X-API-Name': 'My API',
      'X-Powered-By': 'Next.js 14'
    }
  };

  static applyStandardHeaders(response: NextResponse): NextResponse {
    const headers = {
      ...this.STANDARD_HEADERS.CORS,
      ...this.STANDARD_HEADERS.SECURITY,
      ...this.STANDARD_HEADERS.API_INFO
    };

    Object.entries(headers).forEach(([key, value]) => {
      response.headers.set(key, value);
    });

    return response;
  }

  static setCacheHeaders(
    response: NextResponse,
    cacheControl: string,
    etag?: string,
    lastModified?: string
  ): NextResponse {
    response.headers.set('Cache-Control', cacheControl);
    
    if (etag) {
      response.headers.set('ETag', etag);
    }
    
    if (lastModified) {
      response.headers.set('Last-Modified', lastModified);
    }
    
    return response;
  }

  static setRateLimitHeaders(
    response: NextResponse,
    limit: number,
    remaining: number,
    reset: number
  ): NextResponse {
    response.headers.set('X-RateLimit-Limit', limit.toString());
    response.headers.set('X-RateLimit-Remaining', remaining.toString());
    response.headers.set('X-RateLimit-Reset', reset.toString());
    return response;
  }

  static parseCustomHeaders(request: NextRequest): {
    apiKey?: string;
    authorization?: string;
    correlationId?: string;
    idempotencyKey?: string;
  } {
    return {
      apiKey: request.headers.get('X-API-Key') || undefined,
      authorization: request.headers.get('Authorization') || undefined,
      correlationId: request.headers.get('X-Correlation-ID') || undefined,
      idempotencyKey: request.headers.get('Idempotency-Key') || undefined
    };
  }

  static createConditionalHeaders(
    request: NextRequest,
    etag: string,
    lastModified: string
  ): { status: number; headers: Headers } | null {
    const ifNoneMatch = request.headers.get('If-None-Match');
    const ifModifiedSince = request.headers.get('If-Modified-Since');
    
    if (ifNoneMatch && ifNoneMatch === etag) {
      return {
        status: 304,
        headers: new Headers({
          'ETag': etag,
          'Cache-Control': 'public, max-age=3600'
        })
      };
    }
    
    if (ifModifiedSince && new Date(ifModifiedSince) >= new Date(lastModified)) {
      return {
        status: 304,
        headers: new Headers({
          'Last-Modified': lastModified,
          'Cache-Control': 'public, max-age=3600'
        })
      };
    }
    
    return null;
  }
}

/**
 * Middleware per gestione header
 */
export const headerMiddleware = async (
  request: NextRequest,
  next: () => Promise<NextResponse>
) => {
  // Pre-processing: log headers
  console.log('Request Headers:', Object.fromEntries(request.headers.entries()));
  
  // Gestione CORS preflight
  if (request.method === 'OPTIONS') {
    const response = new NextResponse(null, { status: 204 });
    HeaderManager.applyStandardHeaders(response);
    return response;
  }
  
  // Esegui la richiesta
  const response = await next();
  
  // Post-processing: applica header standard
  HeaderManager.applyStandardHeaders(response);
  
  // Aggiungi correlation ID se presente
  const correlationId = request.headers.get('X-Correlation-ID');
  if (correlationId) {
    response.headers.set('X-Correlation-ID', correlationId);
  }
  
  // Aggiungi tempo di risposta
  const startTime = parseInt(request.headers.get('X-Request-Start') || Date.now().toString());
  const responseTime = Date.now() - startTime;
  response.headers.set('X-Response-Time', responseTime.toString());
  
  return response;
};
Streaming Responses
typescript
import { NextRequest } from 'next/server';

/**
 * Utility per streaming responses
 */
export class StreamResponse {
  static async jsonStream<T>(
    dataStream: AsyncIterable<T>,
    request?: NextRequest
  ): Promise<Response> {
    const encoder = new TextEncoder();
    const stream = new ReadableStream({
      async start(controller) {
        try {
          controller.enqueue(encoder.encode('{"data":['));
          
          let first = true;
          for await (const item of dataStream) {
            if (!first) {
              controller.enqueue(encoder.encode(','));
            }
            controller.enqueue(encoder.encode(JSON.stringify(item)));
            first = false;
          }
          
          controller.enqueue(encoder.encode(']}'));
          controller.close();
        } catch (error) {
          controller.error(error);
        }
      }
    });
    
    return new Response(stream, {
      headers: {
        'Content-Type': 'application/json',
        'Transfer-Encoding': 'chunked',
        'X-Content-Type-Options': 'nosniff'
      }
    });
  }

  static async ndjsonStream<T>(
    dataStream: AsyncIterable<T>
  ): Promise<Response> {
    const encoder = new TextEncoder();
    const stream = new ReadableStream({
      async start(controller) {
        try {
          for await (const item of dataStream) {
            controller.enqueue(encoder.encode(JSON.stringify(item) + '\n'));
          }
          controller.close();
        } catch (error) {
          controller.error(error);
        }
      }
    });
    
    return new Response(stream, {
      headers: {
        'Content-Type': 'application/x-ndjson',
        'Transfer-Encoding': 'chunked'
      }
    });
  }

  static async sseStream(
    eventStream: AsyncIterable<{ event?: string; data: any; id?: string; retry?: number }>
  ): Promise<Response> {
    const encoder = new TextEncoder();
    const stream = new ReadableStream({
      async start(controller) {
        try {
          for await (const message of eventStream) {
            let lines = [];
            
            if (message.event) {
              lines.push(`event: ${message.event}`);
            }
            
            if (message.id) {
              lines.push(`id: ${message.id}`);
            }
            
            if (message.retry) {
              lines.push(`retry: ${message.retry}`);
            }
            
            lines.push(`data: ${JSON.stringify(message.data)}`);
            controller.enqueue(encoder.encode(lines.join('\n') + '\n\n'));
          }
          controller.close();
        } catch (error) {
          controller.error(error);
        }
      }
    });
    
    return new Response(stream, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive'
      }
    });
  }
}

/**
 * Esempio di generatore di dati streaming
 */
async function* generateLargeDataSet(count: number) {
  for (let i = 0; i < count; i++) {
    yield {
      id: i,
      name: `Item ${i}`,
      timestamp: new Date().toISOString()
    };
    
    // Simula qualche elaborazione
    await new Promise(resolve => setTimeout(resolve, 10));
  }
}

/**
 * Route Handler con streaming
 */
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const count = parseInt(searchParams.get('count') || '1000');
  const format = searchParams.get('format') || 'json';
  
  const dataStream = generateLargeDataSet(count);
  
  switch (format) {
    case 'ndjson':
      return StreamResponse.ndjsonStream(dataStream);
    case 'sse':
      return StreamResponse.sseStream(dataStream);
    default:
      return StreamResponse.jsonStream(dataStream, request);
  }
}
§ URL DESIGN
Resource Naming Conventions
typescript
/**
 * Convenzioni per naming delle risorse RESTful
 */
export class ResourceNaming {
  // Regole per naming delle risorse
  static readonly RULES = {
    // Use nouns, not verbs
    // Bad: /getUsers, /createOrder
    // Good: /users, /orders
    
    // Use plural nouns for collections
    // /users, /orders, /products
    
    // Use singular nouns for specific resources
    // /users/{id}, /orders/{id}
    
    // Use hyphens for multi-word resources
    // /user-roles, /shipping-addresses
    
    // Use lowercase letters
    // /users, not /Users
    
    // Avoid file extensions
    // /api/users, not /api/users.json
  };

  /**
   * Genera URL per risorse secondo le convenzioni
   */
  static generateResourceUrl(
    resourceName: string,
    baseUrl: string = '/api',
    id?: string,
    subResource?: string
  ): string {
    // Normalizza il nome della risorsa
    const normalized = this.normalizeResourceName(resourceName);
    
    // Costruisci il percorso base
    let path = `${baseUrl}/${normalized}`;
    
    // Aggiungi ID se specificato
    if (id) {
      path += `/${id}`;
    }
    
    // Aggiungi sotto-risorsa se specificata
    if (subResource) {
      path += `/${this.normalizeResourceName(subResource)}`;
    }
    
    return path;
  }

  /**
   * Normalizza il nome della risorsa
   */
  private static normalizeResourceName(name: string): string {
    return name
      .toLowerCase()
      .replace(/\s+/g, '-')
      .replace(/[^a-z0-9-]/g, '')
      .replace(/-+/g, '-')
      .replace(/^-|-$/g, '');
  }

  /**
   * Estrae informazioni dalla URL
   */
  static parseResourceUrl(
    url: string,
    baseUrl: string = '/api'
  ): {
    resource: string;
    id?: string;
    subResource?: string;
    action?: string;
  } {
    const path = url.replace(baseUrl, '').replace(/^\/|\/$/g, '');
    const parts = path.split('/').filter(Boolean);
    
    const result: any = {};
    
    if (parts.length >= 1) {
      result.resource = parts[0];
   