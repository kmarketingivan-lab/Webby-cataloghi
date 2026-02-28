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
   


---

## NEXT.JS 14 ROUTE HANDLERS — COMPLETE PATTERNS

### Panoramica
Pattern completi per Route Handlers in Next.js 14 App Router con validazione Zod, error handling, middleware-like patterns e type-safe responses.

### Implementazione Completa

```typescript
// lib/api/response.ts
import { NextResponse } from "next/server";
import { ZodError } from "zod";

// ============================================================
// STANDARD API RESPONSE FORMAT — RFC 7807 compatible
// ============================================================
interface ApiSuccessResponse<T> {
  success: true;
  data: T;
  meta?: {
    page?: number;
    perPage?: number;
    total?: number;
    totalPages?: number;
  };
}

interface ApiErrorResponse {
  success: false;
  error: {
    type: string;
    title: string;
    status: number;
    detail: string;
    instance?: string;
    errors?: Array<{
      field: string;
      message: string;
      code: string;
    }>;
  };
}

type ApiResponse<T> = ApiSuccessResponse<T> | ApiErrorResponse;

export function apiSuccess<T>(data: T, meta?: ApiSuccessResponse<T>["meta"]): NextResponse {
  const body: ApiSuccessResponse<T> = { success: true, data };
  if (meta) body.meta = meta;
  return NextResponse.json(body, { status: 200 });
}

export function apiCreated<T>(data: T): NextResponse {
  return NextResponse.json(
    { success: true, data } satisfies ApiSuccessResponse<T>,
    { status: 201 }
  );
}

export function apiNoContent(): NextResponse {
  return new NextResponse(null, { status: 204 });
}

export function apiError(
  status: number,
  title: string,
  detail: string,
  type = "about:blank"
): NextResponse {
  const body: ApiErrorResponse = {
    success: false,
    error: { type, title, status, detail },
  };
  return NextResponse.json(body, {
    status,
    headers: { "Content-Type": "application/problem+json" },
  });
}

export function apiValidationError(error: ZodError): NextResponse {
  const body: ApiErrorResponse = {
    success: false,
    error: {
      type: "validation_error",
      title: "Validation Error",
      status: 422,
      detail: "The request body contains invalid fields.",
      errors: error.issues.map((issue) => ({
        field: issue.path.join("."),
        message: issue.message,
        code: issue.code,
      })),
    },
  };
  return NextResponse.json(body, {
    status: 422,
    headers: { "Content-Type": "application/problem+json" },
  });
}

export function apiNotFound(resource: string, id: string): NextResponse {
  return apiError(
    404,
    "Not Found",
    `${resource} with id '${id}' was not found.`,
    "not_found"
  );
}

export function apiUnauthorized(): NextResponse {
  return apiError(
    401,
    "Unauthorized",
    "Authentication is required to access this resource.",
    "unauthorized"
  );
}

export function apiForbidden(): NextResponse {
  return apiError(
    403,
    "Forbidden",
    "You do not have permission to perform this action.",
    "forbidden"
  );
}

export function apiConflict(detail: string): NextResponse {
  return apiError(409, "Conflict", detail, "conflict");
}
```

```typescript
// lib/api/route-handler.ts
import { NextRequest, NextResponse } from "next/server";
import { ZodSchema, z } from "zod";
import { apiError, apiValidationError } from "./response";
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";

// ============================================================
// TYPE-SAFE ROUTE HANDLER WRAPPER
// ============================================================
interface RouteContext {
  params: Record<string, string>;
}

interface HandlerConfig<TBody = unknown, TQuery = unknown> {
  auth?: boolean;
  roles?: string[];
  bodySchema?: ZodSchema<TBody>;
  querySchema?: ZodSchema<TQuery>;
}

interface HandlerContext<TBody = unknown, TQuery = unknown> {
  req: NextRequest;
  params: Record<string, string>;
  body: TBody;
  query: TQuery;
  user: {
    id: string;
    email: string;
    name: string;
    role: string;
  } | null;
}

export function createHandler<TBody = unknown, TQuery = unknown>(
  config: HandlerConfig<TBody, TQuery>,
  handler: (ctx: HandlerContext<TBody, TQuery>) => Promise<NextResponse>
) {
  return async (req: NextRequest, context: RouteContext) => {
    try {
      // Authentication check
      let user = null;
      if (config.auth) {
        const session = await getServerSession(authOptions);
        if (!session?.user) {
          return apiError(401, "Unauthorized", "Authentication required.");
        }
        user = session.user as HandlerContext["user"];

        // Role check
        if (config.roles && user && !config.roles.includes(user.role)) {
          return apiError(403, "Forbidden", "Insufficient permissions.");
        }
      }

      // Body validation
      let body = {} as TBody;
      if (config.bodySchema) {
        const rawBody = await req.json().catch(() => null);
        if (rawBody === null) {
          return apiError(400, "Bad Request", "Invalid JSON body.");
        }
        const result = config.bodySchema.safeParse(rawBody);
        if (!result.success) {
          return apiValidationError(result.error);
        }
        body = result.data;
      }

      // Query validation
      let query = {} as TQuery;
      if (config.querySchema) {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        const result = config.querySchema.safeParse(searchParams);
        if (!result.success) {
          return apiValidationError(result.error);
        }
        query = result.data;
      }

      return await handler({
        req,
        params: context.params,
        body,
        query,
        user,
      });
    } catch (error) {
      console.error("[API Error]", error);
      if (error instanceof z.ZodError) {
        return apiValidationError(error);
      }
      return apiError(
        500,
        "Internal Server Error",
        "An unexpected error occurred."
      );
    }
  };
}
```

### Varianti e Configurazioni

```typescript
// app/api/products/route.ts
import { NextRequest } from "next/server";
import { z } from "zod";
import { prisma } from "@/lib/prisma";
import { createHandler } from "@/lib/api/route-handler";
import { apiSuccess, apiCreated } from "@/lib/api/response";

// ============================================================
// QUERY SCHEMA — pagination, filtering, sorting
// ============================================================
const listQuerySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  perPage: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.enum(["name", "price", "createdAt", "updatedAt"]).default("createdAt"),
  order: z.enum(["asc", "desc"]).default("desc"),
  search: z.string().optional(),
  category: z.string().optional(),
  minPrice: z.coerce.number().nonnegative().optional(),
  maxPrice: z.coerce.number().positive().optional(),
  status: z.enum(["ACTIVE", "DRAFT", "ARCHIVED"]).optional(),
});

// ============================================================
// CREATE SCHEMA
// ============================================================
const createProductSchema = z.object({
  name: z.string().min(1).max(255),
  slug: z.string().min(1).max(255).regex(/^[a-z0-9-]+$/),
  description: z.string().min(10).max(5000),
  price: z.number().positive(),
  compareAtPrice: z.number().positive().optional(),
  categoryId: z.string().uuid(),
  tags: z.array(z.string()).max(10).default([]),
  images: z.array(z.object({
    url: z.string().url(),
    alt: z.string().max(255).default(""),
    position: z.number().int().nonnegative(),
  })).min(1).max(20),
  variants: z.array(z.object({
    name: z.string(),
    sku: z.string(),
    price: z.number().positive(),
    stock: z.number().int().nonnegative(),
    attributes: z.record(z.string()),
  })).optional(),
  metadata: z.record(z.unknown()).optional(),
});

// ============================================================
// GET /api/products — List with pagination, filtering, sorting
// ============================================================
export const GET = createHandler(
  {
    querySchema: listQuerySchema,
  },
  async ({ query }) => {
    const { page, perPage, sort, order, search, category, minPrice, maxPrice, status } = query;
    const skip = (page - 1) * perPage;

    const where = {
      ...(search && {
        OR: [
          { name: { contains: search, mode: "insensitive" as const } },
          { description: { contains: search, mode: "insensitive" as const } },
        ],
      }),
      ...(category && { category: { slug: category } }),
      ...(minPrice !== undefined && { price: { gte: minPrice } }),
      ...(maxPrice !== undefined && { price: { ...( minPrice !== undefined ? { gte: minPrice } : {}), lte: maxPrice } }),
      ...(status && { status }),
    };

    const [products, total] = await Promise.all([
      prisma.product.findMany({
        where,
        orderBy: { [sort]: order },
        skip,
        take: perPage,
        include: {
          category: { select: { id: true, name: true, slug: true } },
          images: { orderBy: { position: "asc" }, take: 1 },
          _count: { select: { reviews: true } },
        },
      }),
      prisma.product.count({ where }),
    ]);

    return apiSuccess(products, {
      page,
      perPage,
      total,
      totalPages: Math.ceil(total / perPage),
    });
  }
);

// ============================================================
// POST /api/products — Create product
// ============================================================
export const POST = createHandler(
  {
    auth: true,
    roles: ["ADMIN", "EDITOR"],
    bodySchema: createProductSchema,
  },
  async ({ body, user }) => {
    const product = await prisma.product.create({
      data: {
        ...body,
        createdById: user!.id,
        images: { create: body.images },
        variants: body.variants ? { create: body.variants } : undefined,
        tags: {
          connectOrCreate: body.tags.map((tag) => ({
            where: { name: tag },
            create: { name: tag, slug: tag.toLowerCase().replace(/\s+/g, "-") },
          })),
        },
      },
      include: {
        category: true,
        images: true,
        variants: true,
        tags: true,
      },
    });

    return apiCreated(product);
  }
);
```

```typescript
// app/api/products/[id]/route.ts
import { z } from "zod";
import { prisma } from "@/lib/prisma";
import { createHandler } from "@/lib/api/route-handler";
import { apiSuccess, apiNoContent, apiNotFound, apiConflict } from "@/lib/api/response";

const updateProductSchema = z.object({
  name: z.string().min(1).max(255).optional(),
  description: z.string().min(10).max(5000).optional(),
  price: z.number().positive().optional(),
  status: z.enum(["ACTIVE", "DRAFT", "ARCHIVED"]).optional(),
  categoryId: z.string().uuid().optional(),
}).refine((data) => Object.keys(data).length > 0, {
  message: "At least one field must be provided for update",
});

// ============================================================
// GET /api/products/[id]
// ============================================================
export const GET = createHandler({}, async ({ params }) => {
  const product = await prisma.product.findUnique({
    where: { id: params.id },
    include: {
      category: true,
      images: { orderBy: { position: "asc" } },
      variants: true,
      tags: true,
      reviews: {
        take: 5,
        orderBy: { createdAt: "desc" },
        include: { user: { select: { name: true, image: true } } },
      },
      _count: { select: { reviews: true } },
    },
  });

  if (!product) {
    return apiNotFound("Product", params.id);
  }

  return apiSuccess(product);
});

// ============================================================
// PATCH /api/products/[id]
// ============================================================
export const PATCH = createHandler(
  {
    auth: true,
    roles: ["ADMIN", "EDITOR"],
    bodySchema: updateProductSchema,
  },
  async ({ params, body }) => {
    const existing = await prisma.product.findUnique({
      where: { id: params.id },
    });

    if (!existing) {
      return apiNotFound("Product", params.id);
    }

    const product = await prisma.product.update({
      where: { id: params.id },
      data: {
        ...body,
        updatedAt: new Date(),
      },
      include: { category: true, images: true },
    });

    return apiSuccess(product);
  }
);

// ============================================================
// DELETE /api/products/[id]
// ============================================================
export const DELETE = createHandler(
  { auth: true, roles: ["ADMIN"] },
  async ({ params }) => {
    const existing = await prisma.product.findUnique({
      where: { id: params.id },
      include: { _count: { select: { orderItems: true } } },
    });

    if (!existing) {
      return apiNotFound("Product", params.id);
    }

    if (existing._count.orderItems > 0) {
      return apiConflict(
        "Cannot delete product with existing orders. Archive it instead."
      );
    }

    await prisma.product.delete({ where: { id: params.id } });
    return apiNoContent();
  }
);
```

### Edge Cases e Error Handling

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";

// ============================================================
// RATE LIMITING — Token Bucket Algorithm
// ============================================================
interface RateLimitConfig {
  maxTokens: number;
  refillRate: number;
  refillInterval: number;
}

const buckets = new Map<string, { tokens: number; lastRefill: number }>();

export function rateLimit(
  key: string,
  config: RateLimitConfig = { maxTokens: 60, refillRate: 10, refillInterval: 60000 }
): { allowed: boolean; remaining: number; resetAt: number } {
  const now = Date.now();
  let bucket = buckets.get(key);

  if (!bucket) {
    bucket = { tokens: config.maxTokens, lastRefill: now };
    buckets.set(key, bucket);
  }

  const elapsed = now - bucket.lastRefill;
  const refills = Math.floor(elapsed / config.refillInterval);

  if (refills > 0) {
    bucket.tokens = Math.min(
      config.maxTokens,
      bucket.tokens + refills * config.refillRate
    );
    bucket.lastRefill = now;
  }

  if (bucket.tokens > 0) {
    bucket.tokens--;
    return {
      allowed: true,
      remaining: bucket.tokens,
      resetAt: bucket.lastRefill + config.refillInterval,
    };
  }

  return {
    allowed: false,
    remaining: 0,
    resetAt: bucket.lastRefill + config.refillInterval,
  };
}

// ============================================================
// CORS CONFIGURATION
// ============================================================
const ALLOWED_ORIGINS = [
  "https://myapp.com",
  "https://admin.myapp.com",
  ...(process.env.NODE_ENV === "development" ? ["http://localhost:3000"] : []),
];

export function corsHeaders(req: NextRequest): HeadersInit {
  const origin = req.headers.get("origin") ?? "";
  const isAllowed = ALLOWED_ORIGINS.includes(origin);

  return {
    "Access-Control-Allow-Origin": isAllowed ? origin : "",
    "Access-Control-Allow-Methods": "GET, POST, PUT, PATCH, DELETE, OPTIONS",
    "Access-Control-Allow-Headers": "Content-Type, Authorization, X-Request-Id",
    "Access-Control-Max-Age": "86400",
    "Access-Control-Allow-Credentials": "true",
  };
}

export function handleCORS(req: NextRequest): NextResponse | null {
  if (req.method === "OPTIONS") {
    return new NextResponse(null, {
      status: 204,
      headers: corsHeaders(req),
    });
  }
  return null;
}

// ============================================================
// REQUEST ID TRACKING
// ============================================================
export function getRequestId(req: NextRequest): string {
  return req.headers.get("x-request-id") ?? crypto.randomUUID();
}

// ============================================================
// API VERSIONING via headers or path
// ============================================================
export function getApiVersion(req: NextRequest): string {
  const headerVersion = req.headers.get("api-version");
  if (headerVersion) return headerVersion;

  const pathMatch = req.nextUrl.pathname.match(/\/api\/v(\d+)\//);
  if (pathMatch) return pathMatch[1];

  return "1";
}
```

### Errori Comuni da Evitare
- **Missing Content-Type**: Sempre impostare `application/json` o `application/problem+json` per errori
- **N+1 queries in list endpoint**: Usa `include` Prisma per eager loading
- **Rate limit in-memory su serverless**: Usa Redis o Upstash per rate limiting persistente
- **CORS wildcard in produzione**: Mai usare `*` come origin — elenca domini specifici

### Checklist di Verifica
- [ ] Tutti gli endpoint restituiscono formato consistente `{success, data/error}`
- [ ] Le validazioni usano Zod con messaggi chiari
- [ ] Rate limiting è configurato per endpoint critici
- [ ] CORS è configurato per domini specifici
- [ ] I route handler gestiscono tutti gli HTTP status codes appropriati
- [ ] Le query Prisma usano `include` per evitare N+1

---

## PAGINATION-PATTERNS

### Panoramica
Pattern completi per cursor-based e offset-based pagination con supporto per filtering, sorting, e API response standardizzate.

### Implementazione Completa

```typescript
// lib/api/pagination.ts
import { z } from "zod";

// ============================================================
// OFFSET-BASED PAGINATION
// ============================================================
export const offsetPaginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  perPage: z.coerce.number().int().min(1).max(100).default(20),
});

export type OffsetPagination = z.infer<typeof offsetPaginationSchema>;

export function getOffsetParams(pagination: OffsetPagination) {
  return {
    skip: (pagination.page - 1) * pagination.perPage,
    take: pagination.perPage,
  };
}

export function buildOffsetMeta(
  pagination: OffsetPagination,
  total: number,
  baseUrl: string
) {
  const totalPages = Math.ceil(total / pagination.perPage);
  const hasNext = pagination.page < totalPages;
  const hasPrev = pagination.page > 1;

  return {
    page: pagination.page,
    perPage: pagination.perPage,
    total,
    totalPages,
    hasNext,
    hasPrev,
    links: {
      self: `${baseUrl}?page=${pagination.page}&perPage=${pagination.perPage}`,
      first: `${baseUrl}?page=1&perPage=${pagination.perPage}`,
      last: `${baseUrl}?page=${totalPages}&perPage=${pagination.perPage}`,
      ...(hasNext && { next: `${baseUrl}?page=${pagination.page + 1}&perPage=${pagination.perPage}` }),
      ...(hasPrev && { prev: `${baseUrl}?page=${pagination.page - 1}&perPage=${pagination.perPage}` }),
    },
  };
}

// ============================================================
// CURSOR-BASED PAGINATION
// ============================================================
export const cursorPaginationSchema = z.object({
  cursor: z.string().optional(),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  direction: z.enum(["forward", "backward"]).default("forward"),
});

export type CursorPagination = z.infer<typeof cursorPaginationSchema>;

export function getCursorParams(pagination: CursorPagination) {
  return {
    take: pagination.limit + 1,
    ...(pagination.cursor && {
      cursor: { id: pagination.cursor },
      skip: 1,
    }),
  };
}

export function buildCursorMeta<T extends { id: string }>(
  items: T[],
  pagination: CursorPagination
) {
  const hasMore = items.length > pagination.limit;
  const data = hasMore ? items.slice(0, pagination.limit) : items;
  const startCursor = data[0]?.id ?? null;
  const endCursor = data[data.length - 1]?.id ?? null;

  return {
    data,
    pageInfo: {
      hasNextPage: hasMore,
      hasPreviousPage: !!pagination.cursor,
      startCursor,
      endCursor,
    },
  };
}
```

```typescript
// app/api/posts/route.ts — cursor-based pagination example
import { prisma } from "@/lib/prisma";
import { createHandler } from "@/lib/api/route-handler";
import { cursorPaginationSchema, getCursorParams, buildCursorMeta } from "@/lib/api/pagination";
import { NextResponse } from "next/server";
import { z } from "zod";

const querySchema = cursorPaginationSchema.extend({
  authorId: z.string().uuid().optional(),
  tag: z.string().optional(),
  status: z.enum(["PUBLISHED", "DRAFT"]).default("PUBLISHED"),
});

export const GET = createHandler(
  { querySchema },
  async ({ query }) => {
    const { cursor, limit, direction, authorId, tag, status } = query;

    const where = {
      status,
      ...(authorId && { authorId }),
      ...(tag && { tags: { some: { slug: tag } } }),
    };

    const cursorParams = getCursorParams({ cursor, limit, direction });

    const posts = await prisma.post.findMany({
      where,
      ...cursorParams,
      orderBy: { createdAt: "desc" },
      include: {
        author: { select: { id: true, name: true, image: true } },
        tags: { select: { id: true, name: true, slug: true } },
        _count: { select: { comments: true, likes: true } },
      },
    });

    const { data, pageInfo } = buildCursorMeta(posts, { cursor, limit, direction });

    return NextResponse.json({
      success: true,
      data,
      pageInfo,
    });
  }
);
```

### Edge Cases e Error Handling

```typescript
// lib/api/filter-builder.ts
import { Prisma } from "@prisma/client";
import { z } from "zod";

// ============================================================
// ADVANCED FILTER BUILDER — supports operators
// ============================================================
const filterOperatorSchema = z.object({
  field: z.string(),
  operator: z.enum(["eq", "neq", "gt", "gte", "lt", "lte", "contains", "startsWith", "in", "notIn"]),
  value: z.union([z.string(), z.number(), z.boolean(), z.array(z.string())]),
});

export type FilterOperator = z.infer<typeof filterOperatorSchema>;

export function buildPrismaFilter(filters: FilterOperator[]): Record<string, unknown> {
  const where: Record<string, unknown> = {};

  for (const filter of filters) {
    const { field, operator, value } = filter;

    switch (operator) {
      case "eq":
        where[field] = value;
        break;
      case "neq":
        where[field] = { not: value };
        break;
      case "gt":
        where[field] = { gt: value };
        break;
      case "gte":
        where[field] = { gte: value };
        break;
      case "lt":
        where[field] = { lt: value };
        break;
      case "lte":
        where[field] = { lte: value };
        break;
      case "contains":
        where[field] = { contains: value as string, mode: "insensitive" };
        break;
      case "startsWith":
        where[field] = { startsWith: value as string, mode: "insensitive" };
        break;
      case "in":
        where[field] = { in: value as string[] };
        break;
      case "notIn":
        where[field] = { notIn: value as string[] };
        break;
    }
  }

  return where;
}

// ============================================================
// SORT BUILDER — validates allowed sort fields
// ============================================================
export function buildPrismaSort(
  sort: string,
  order: "asc" | "desc",
  allowedFields: string[]
): Record<string, "asc" | "desc"> | undefined {
  if (!allowedFields.includes(sort)) return undefined;
  return { [sort]: order };
}

// ============================================================
// SEARCH QUERY — full-text with multiple fields
// ============================================================
export function buildSearchFilter(
  search: string | undefined,
  fields: string[]
): Record<string, unknown> | undefined {
  if (!search || search.trim().length === 0) return undefined;

  return {
    OR: fields.map((field) => ({
      [field]: { contains: search.trim(), mode: "insensitive" },
    })),
  };
}
```

### Errori Comuni da Evitare
- **Offset pagination su dataset grandi**: Cursor-based è molto più performante per milioni di record
- **Missing total count**: Sempre includere il count totale per offset pagination (necessario per UI)
- **Cursor non unico**: Il cursor deve basarsi su un campo univoco (es. `id` o `createdAt` + `id`)
- **Sorting con cursor inconsistente**: L'ordine di sorting deve matchare la direzione del cursor

### Checklist di Verifica
- [ ] La pagination default ha un perPage ragionevole (20-50)
- [ ] Il max perPage è limitato (100) per prevenire abuse
- [ ] I link HATEOAS (next, prev, first, last) sono inclusi nella response
- [ ] La cursor pagination gestisce correttamente il primo e ultimo batch
- [ ] Le query supportano filtering + sorting + pagination combinati

---

## ZOD-VALIDATION-PATTERNS

### Panoramica
Pattern avanzati di validazione con Zod per API Next.js 14: schema riutilizzabili, transform, discriminated unions, e integrazione con form e API routes.

### Implementazione Completa

```typescript
// lib/validations/shared.ts
import { z } from "zod";

// ============================================================
// REUSABLE FIELD VALIDATORS
// ============================================================
export const email = z
  .string()
  .email("Invalid email address")
  .min(5)
  .max(255)
  .toLowerCase()
  .trim();

export const password = z
  .string()
  .min(8, "Password must be at least 8 characters")
  .max(128)
  .regex(/[A-Z]/, "Must contain at least one uppercase letter")
  .regex(/[a-z]/, "Must contain at least one lowercase letter")
  .regex(/[0-9]/, "Must contain at least one number")
  .regex(/[^A-Za-z0-9]/, "Must contain at least one special character");

export const slug = z
  .string()
  .min(1)
  .max(255)
  .regex(/^[a-z0-9]+(?:-[a-z0-9]+)*$/, "Invalid slug format");

export const uuid = z.string().uuid("Invalid UUID format");

export const positiveInt = z.coerce.number().int().positive();

export const isoDate = z.coerce.date();

export const url = z.string().url("Invalid URL").max(2048);

export const phone = z
  .string()
  .regex(
    /^\+?[1-9]\d{1,14}$/,
    "Invalid phone number. Use E.164 format (e.g., +1234567890)"
  );

export const hexColor = z
  .string()
  .regex(/^#[0-9A-Fa-f]{6}$/, "Invalid hex color");

// ============================================================
// COMMON OBJECT SCHEMAS
// ============================================================
export const addressSchema = z.object({
  line1: z.string().min(1).max(255),
  line2: z.string().max(255).optional(),
  city: z.string().min(1).max(100),
  state: z.string().min(1).max(100),
  postalCode: z.string().min(1).max(20),
  country: z.string().length(2, "Use ISO 3166-1 alpha-2 country code"),
});

export const moneySchema = z.object({
  amount: z.number().nonnegative(),
  currency: z.string().length(3, "Use ISO 4217 currency code").toUpperCase(),
});

export const dateRangeSchema = z
  .object({
    from: isoDate,
    to: isoDate,
  })
  .refine((data) => data.to > data.from, {
    message: "End date must be after start date",
    path: ["to"],
  });

export const paginationSchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  perPage: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.string().optional(),
  order: z.enum(["asc", "desc"]).default("desc"),
});

// ============================================================
// DISCRIMINATED UNION — notification types
// ============================================================
export const notificationSchema = z.discriminatedUnion("type", [
  z.object({
    type: z.literal("email"),
    to: email,
    subject: z.string().min(1).max(255),
    body: z.string().min(1).max(50000),
    cc: z.array(email).max(10).optional(),
  }),
  z.object({
    type: z.literal("sms"),
    to: phone,
    body: z.string().min(1).max(160),
  }),
  z.object({
    type: z.literal("push"),
    userId: uuid,
    title: z.string().min(1).max(100),
    body: z.string().min(1).max(500),
    data: z.record(z.string()).optional(),
  }),
]);

export type Notification = z.infer<typeof notificationSchema>;

// ============================================================
// TRANSFORM PATTERNS
// ============================================================
export const createUserSchema = z.object({
  name: z.string().min(1).max(255).trim(),
  email,
  password,
}).transform((data) => ({
  ...data,
  displayName: data.name,
  normalizedEmail: data.email.toLowerCase(),
  createdAt: new Date(),
}));

export type CreateUserInput = z.input<typeof createUserSchema>;
export type CreateUserOutput = z.output<typeof createUserSchema>;
```

```typescript
// lib/validations/api-helpers.ts
import { z, ZodSchema, ZodError } from "zod";
import { NextRequest, NextResponse } from "next/server";

// ============================================================
// VALIDATE REQUEST HELPER — for use without createHandler
// ============================================================
export async function validateBody<T>(
  req: NextRequest,
  schema: ZodSchema<T>
): Promise<
  | { success: true; data: T }
  | { success: false; error: NextResponse }
> {
  try {
    const raw = await req.json();
    const data = schema.parse(raw);
    return { success: true, data };
  } catch (error) {
    if (error instanceof ZodError) {
      return {
        success: false,
        error: NextResponse.json(
          {
            success: false,
            error: {
              type: "validation_error",
              title: "Validation Error",
              status: 422,
              detail: "Request body validation failed",
              errors: error.issues.map((i) => ({
                field: i.path.join("."),
                message: i.message,
                code: i.code,
              })),
            },
          },
          { status: 422 }
        ),
      };
    }
    return {
      success: false,
      error: NextResponse.json(
        { success: false, error: { status: 400, title: "Bad Request", detail: "Invalid JSON" } },
        { status: 400 }
      ),
    };
  }
}

export function validateQuery<T>(
  req: NextRequest,
  schema: ZodSchema<T>
): { success: true; data: T } | { success: false; error: NextResponse } {
  const params = Object.fromEntries(req.nextUrl.searchParams);
  const result = schema.safeParse(params);

  if (!result.success) {
    return {
      success: false,
      error: NextResponse.json(
        {
          success: false,
          error: {
            type: "validation_error",
            title: "Invalid Query Parameters",
            status: 400,
            errors: result.error.issues.map((i) => ({
              field: i.path.join("."),
              message: i.message,
              code: i.code,
            })),
          },
        },
        { status: 400 }
      ),
    };
  }

  return { success: true, data: result.data };
}

// ============================================================
// OPENAPI SCHEMA GENERATION from Zod
// ============================================================
export function zodToOpenAPI(schema: ZodSchema, description?: string): object {
  if (schema instanceof z.ZodString) {
    return { type: "string", description };
  }
  if (schema instanceof z.ZodNumber) {
    return { type: "number", description };
  }
  if (schema instanceof z.ZodBoolean) {
    return { type: "boolean", description };
  }
  if (schema instanceof z.ZodArray) {
    return {
      type: "array",
      items: zodToOpenAPI(schema.element),
      description,
    };
  }
  if (schema instanceof z.ZodObject) {
    const shape = schema.shape;
    const properties: Record<string, object> = {};
    const required: string[] = [];

    for (const [key, value] of Object.entries(shape)) {
      properties[key] = zodToOpenAPI(value as ZodSchema);
      if (!(value instanceof z.ZodOptional)) {
        required.push(key);
      }
    }

    return {
      type: "object",
      properties,
      required: required.length > 0 ? required : undefined,
      description,
    };
  }
  return { description };
}
```

### Errori Comuni da Evitare
- **Zod non tree-shaken**: Importa solo `z` e i tipi necessari per ridurre il bundle size
- **Schema duplicati**: Riutilizza gli schema base (email, uuid) invece di ridefinirli ovunque
- **Transform su input schema**: Ricorda che `z.input<>` e `z.output<>` differiscono quando usi `.transform()`
- **Coerce per query params**: Usa sempre `z.coerce.number()` per query parameters (sono stringhe)

### Checklist di Verifica
- [ ] Tutti gli input utente sono validati con Zod
- [ ] I messaggi di errore sono user-friendly (non tecnici)
- [ ] Gli schema sono condivisi tra frontend e API
- [ ] I tipi TypeScript sono generati automaticamente dagli schema
- [ ] Le query parameters usano `z.coerce` per la conversione da string
- [ ] I discriminated union gestiscono correttamente i diversi payload type



---

## API-RATE-LIMITING-ADVANCED

### Panoramica
Implementazione avanzata di rate limiting con sliding window, tier-based limits, Redis backend e middleware pattern per Next.js 14.

### Implementazione Completa

```typescript
// lib/api/rate-limiter.ts
import { Redis } from "@upstash/redis";
import { Ratelimit } from "@upstash/ratelimit";
import { NextRequest, NextResponse } from "next/server";
import { headers } from "next/headers";

// ============================================================
// UPSTASH REDIS RATE LIMITER
// ============================================================
const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// Different rate limiters for different tiers
const rateLimiters = {
  free: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(20, "1 m"),
    prefix: "rl:free",
    analytics: true,
  }),
  pro: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(100, "1 m"),
    prefix: "rl:pro",
    analytics: true,
  }),
  enterprise: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(1000, "1 m"),
    prefix: "rl:enterprise",
    analytics: true,
  }),
  auth: new Ratelimit({
    redis,
    limiter: Ratelimit.fixedWindow(5, "15 m"),
    prefix: "rl:auth",
    analytics: true,
  }),
  upload: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, "1 h"),
    prefix: "rl:upload",
    analytics: true,
  }),
};

type RateLimitTier = keyof typeof rateLimiters;

// ============================================================
// GET CLIENT IDENTIFIER
// ============================================================
function getClientIdentifier(req: NextRequest): string {
  const forwarded = req.headers.get("x-forwarded-for");
  const realIp = req.headers.get("x-real-ip");
  return forwarded?.split(",")[0]?.trim() ?? realIp ?? "anonymous";
}

// ============================================================
// RATE LIMIT MIDDLEWARE
// ============================================================
export async function withRateLimit(
  req: NextRequest,
  tier: RateLimitTier = "free",
  identifier?: string
): Promise<NextResponse | null> {
  const id = identifier ?? getClientIdentifier(req);
  const limiter = rateLimiters[tier];

  const { success, limit, remaining, reset } = await limiter.limit(id);

  const rateLimitHeaders = {
    "X-RateLimit-Limit": limit.toString(),
    "X-RateLimit-Remaining": remaining.toString(),
    "X-RateLimit-Reset": reset.toString(),
    "Retry-After": success ? "" : Math.ceil((reset - Date.now()) / 1000).toString(),
  };

  if (!success) {
    return NextResponse.json(
      {
        success: false,
        error: {
          type: "rate_limit_exceeded",
          title: "Too Many Requests",
          status: 429,
          detail: `Rate limit exceeded. Try again in ${Math.ceil((reset - Date.now()) / 1000)} seconds.`,
        },
      },
      { status: 429, headers: rateLimitHeaders }
    );
  }

  return null;
}

// ============================================================
// IN-MEMORY SLIDING WINDOW — for serverless without Redis
// ============================================================
interface SlidingWindowEntry {
  timestamp: number;
  count: number;
}

class InMemorySlidingWindow {
  private windows = new Map<string, SlidingWindowEntry[]>();
  private maxRequests: number;
  private windowMs: number;

  constructor(maxRequests: number, windowMs: number) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }

  check(key: string): { allowed: boolean; remaining: number; resetAt: number } {
    const now = Date.now();
    const windowStart = now - this.windowMs;

    let entries = this.windows.get(key) ?? [];
    entries = entries.filter((e) => e.timestamp > windowStart);

    const totalCount = entries.reduce((sum, e) => sum + e.count, 0);

    if (totalCount >= this.maxRequests) {
      const oldestInWindow = entries[0]?.timestamp ?? now;
      return {
        allowed: false,
        remaining: 0,
        resetAt: oldestInWindow + this.windowMs,
      };
    }

    entries.push({ timestamp: now, count: 1 });
    this.windows.set(key, entries);

    return {
      allowed: true,
      remaining: this.maxRequests - totalCount - 1,
      resetAt: now + this.windowMs,
    };
  }

  cleanup(): void {
    const now = Date.now();
    for (const [key, entries] of this.windows) {
      const filtered = entries.filter((e) => e.timestamp > now - this.windowMs);
      if (filtered.length === 0) {
        this.windows.delete(key);
      } else {
        this.windows.set(key, filtered);
      }
    }
  }
}

// Singleton con cleanup periodico
export const apiLimiter = new InMemorySlidingWindow(60, 60_000);
export const authLimiter = new InMemorySlidingWindow(5, 15 * 60_000);

// Cleanup ogni 5 minuti
if (typeof setInterval !== "undefined") {
  setInterval(() => {
    apiLimiter.cleanup();
    authLimiter.cleanup();
  }, 5 * 60_000);
}
```

### Varianti e Configurazioni

```typescript
// middleware.ts — Next.js middleware con rate limiting
import { NextRequest, NextResponse } from "next/server";
import { withRateLimit } from "@/lib/api/rate-limiter";

export async function middleware(req: NextRequest) {
  const path = req.nextUrl.pathname;

  // Rate limit API routes
  if (path.startsWith("/api/")) {
    // Determine tier from API key or session
    const apiKey = req.headers.get("x-api-key");
    let tier: "free" | "pro" | "enterprise" = "free";
    let identifier: string | undefined;

    if (apiKey) {
      // Look up API key tier (simplified — in production, cache this)
      const keyPrefix = apiKey.substring(0, 8);
      if (keyPrefix.startsWith("ent_")) tier = "enterprise";
      else if (keyPrefix.startsWith("pro_")) tier = "pro";
      identifier = apiKey;
    }

    // Stricter limits for auth endpoints
    if (path.startsWith("/api/auth/")) {
      const rateLimited = await withRateLimit(req, "auth");
      if (rateLimited) return rateLimited;
    }

    // Upload endpoints have separate limits
    if (path.startsWith("/api/upload")) {
      const rateLimited = await withRateLimit(req, "upload", identifier);
      if (rateLimited) return rateLimited;
    }

    // General API rate limit
    const rateLimited = await withRateLimit(req, tier, identifier);
    if (rateLimited) return rateLimited;
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/api/:path*"],
};
```

### Edge Cases e Error Handling

```typescript
// lib/api/rate-limit-headers.ts
import { NextResponse } from "next/server";

// ============================================================
// ADD RATE LIMIT HEADERS TO ANY RESPONSE
// ============================================================
export function addRateLimitHeaders(
  response: NextResponse,
  info: { limit: number; remaining: number; reset: number }
): NextResponse {
  response.headers.set("X-RateLimit-Limit", info.limit.toString());
  response.headers.set("X-RateLimit-Remaining", info.remaining.toString());
  response.headers.set("X-RateLimit-Reset", new Date(info.reset).toISOString());

  if (info.remaining <= 0) {
    const retryAfter = Math.ceil((info.reset - Date.now()) / 1000);
    response.headers.set("Retry-After", retryAfter.toString());
  }

  return response;
}

// ============================================================
// API KEY VALIDATION AND TIER DETECTION
// ============================================================
interface ApiKeyInfo {
  id: string;
  tier: "free" | "pro" | "enterprise";
  ownerId: string;
  isActive: boolean;
  rateLimit: number;
  expiresAt: Date | null;
}

export async function validateApiKey(
  key: string,
  prisma: any
): Promise<ApiKeyInfo | null> {
  const apiKey = await prisma.apiKey.findUnique({
    where: { key },
    include: {
      owner: { select: { id: true, plan: true } },
    },
  });

  if (!apiKey || !apiKey.isActive) return null;
  if (apiKey.expiresAt && apiKey.expiresAt < new Date()) return null;

  // Update last used timestamp
  await prisma.apiKey.update({
    where: { id: apiKey.id },
    data: { lastUsedAt: new Date() },
  });

  const tierLimits = { free: 20, pro: 100, enterprise: 1000 };

  return {
    id: apiKey.id,
    tier: apiKey.owner.plan as ApiKeyInfo["tier"],
    ownerId: apiKey.owner.id,
    isActive: true,
    rateLimit: tierLimits[apiKey.owner.plan as keyof typeof tierLimits] ?? 20,
    expiresAt: apiKey.expiresAt,
  };
}
```

### Errori Comuni da Evitare
- **Rate limit in-memory su serverless**: Le funzioni serverless non condividono stato — usa Redis (Upstash)
- **Missing Retry-After header**: Sempre includere per 429 responses (richiesto da molti client)
- **IP detection errata**: Dietro proxy/CDN, usa `x-forwarded-for` o `x-real-ip`, non `req.ip`
- **Rate limit per endpoint, non per utente**: Combina entrambi per protezione completa

### Checklist di Verifica
- [ ] Ogni tier ha limiti appropriati (free < pro < enterprise)
- [ ] Le risposte 429 includono `Retry-After` header
- [ ] L'autenticazione ha rate limit più restrittivi
- [ ] Le API keys sono validabili e hanno scadenza
- [ ] Il cleanup periodico previene memory leak in-memory
- [ ] I rate limit headers sono presenti su tutte le risposte API

---

## API-VERSIONING-STRATEGIES

### Panoramica
Strategie complete per versionare le API REST in Next.js 14: path-based, header-based, query param-based, con deprecation workflow e backward compatibility.

### Implementazione Completa

```typescript
// lib/api/versioning.ts
import { NextRequest, NextResponse } from "next/server";

// ============================================================
// API VERSION DETECTION
// ============================================================
export type ApiVersion = "v1" | "v2" | "v3";

const CURRENT_VERSION: ApiVersion = "v2";
const SUPPORTED_VERSIONS: ApiVersion[] = ["v1", "v2"];
const DEPRECATED_VERSIONS: ApiVersion[] = ["v1"];
const SUNSET_DATES: Record<string, string> = {
  v1: "2025-06-01",
};

export function detectApiVersion(req: NextRequest): ApiVersion {
  // 1. Check path: /api/v2/products
  const pathMatch = req.nextUrl.pathname.match(/\/api\/(v\d+)\//);
  if (pathMatch && SUPPORTED_VERSIONS.includes(pathMatch[1] as ApiVersion)) {
    return pathMatch[1] as ApiVersion;
  }

  // 2. Check Accept header: Accept: application/vnd.myapp.v2+json
  const accept = req.headers.get("accept") ?? "";
  const acceptMatch = accept.match(/application\/vnd\.myapp\.(v\d+)\+json/);
  if (acceptMatch && SUPPORTED_VERSIONS.includes(acceptMatch[1] as ApiVersion)) {
    return acceptMatch[1] as ApiVersion;
  }

  // 3. Check custom header: X-API-Version: v2
  const headerVersion = req.headers.get("x-api-version");
  if (headerVersion && SUPPORTED_VERSIONS.includes(headerVersion as ApiVersion)) {
    return headerVersion as ApiVersion;
  }

  // 4. Check query param: ?api_version=v2
  const queryVersion = req.nextUrl.searchParams.get("api_version");
  if (queryVersion && SUPPORTED_VERSIONS.includes(queryVersion as ApiVersion)) {
    return queryVersion as ApiVersion;
  }

  return CURRENT_VERSION;
}

// ============================================================
// VERSION-AWARE RESPONSE HEADERS
// ============================================================
export function addVersionHeaders(
  response: NextResponse,
  version: ApiVersion
): NextResponse {
  response.headers.set("X-API-Version", version);

  if (DEPRECATED_VERSIONS.includes(version)) {
    response.headers.set("Deprecation", "true");
    response.headers.set(
      "Link",
      `</api/${CURRENT_VERSION}>; rel="successor-version"`
    );

    const sunsetDate = SUNSET_DATES[version];
    if (sunsetDate) {
      response.headers.set("Sunset", new Date(sunsetDate).toUTCString());
    }
  }

  return response;
}

// ============================================================
// VERSIONED ROUTE HANDLER
// ============================================================
type VersionedHandler = (
  req: NextRequest,
  context: { params: Record<string, string> }
) => Promise<NextResponse>;

interface VersionedHandlers {
  v1?: VersionedHandler;
  v2?: VersionedHandler;
  v3?: VersionedHandler;
}

export function versionedHandler(handlers: VersionedHandlers): VersionedHandler {
  return async (req, context) => {
    const version = detectApiVersion(req);

    const handler = handlers[version];
    if (!handler) {
      return NextResponse.json(
        {
          success: false,
          error: {
            type: "unsupported_version",
            title: "Unsupported API Version",
            status: 400,
            detail: `API version '${version}' is not supported. Supported versions: ${SUPPORTED_VERSIONS.join(", ")}`,
          },
        },
        { status: 400 }
      );
    }

    const response = await handler(req, context);
    return addVersionHeaders(response, version);
  };
}
```

### Varianti e Configurazioni

```typescript
// app/api/v2/products/route.ts — versioned endpoint example
import { NextRequest } from "next/server";
import { prisma } from "@/lib/prisma";
import { versionedHandler } from "@/lib/api/versioning";
import { apiSuccess } from "@/lib/api/response";

// ============================================================
// V1 RESPONSE FORMAT (legacy, simpler)
// ============================================================
async function getProductsV1(req: NextRequest) {
  const products = await prisma.product.findMany({
    select: {
      id: true,
      name: true,
      price: true,
      description: true,
      createdAt: true,
    },
    take: 50,
  });

  // V1: flat response, no meta
  return apiSuccess(products);
}

// ============================================================
// V2 RESPONSE FORMAT (current, richer)
// ============================================================
async function getProductsV2(req: NextRequest) {
  const page = parseInt(req.nextUrl.searchParams.get("page") ?? "1");
  const perPage = parseInt(req.nextUrl.searchParams.get("perPage") ?? "20");

  const [products, total] = await Promise.all([
    prisma.product.findMany({
      skip: (page - 1) * perPage,
      take: perPage,
      include: {
        category: { select: { id: true, name: true, slug: true } },
        images: { take: 1, orderBy: { position: "asc" } },
        _count: { select: { reviews: true, variants: true } },
      },
    }),
    prisma.product.count(),
  ]);

  // V2: includes relations, pagination meta, hypermedia links
  const enrichedProducts = products.map((p) => ({
    ...p,
    _links: {
      self: `/api/v2/products/${p.id}`,
      category: `/api/v2/categories/${p.categoryId}`,
      reviews: `/api/v2/products/${p.id}/reviews`,
    },
  }));

  return apiSuccess(enrichedProducts, {
    page,
    perPage,
    total,
    totalPages: Math.ceil(total / perPage),
  });
}

export const GET = versionedHandler({
  v1: getProductsV1,
  v2: getProductsV2,
});

// ============================================================
// RESPONSE TRANSFORMER — adapt v2 shape back to v1
// ============================================================
interface ProductV1 {
  id: string;
  name: string;
  price: number;
  description: string;
}

interface ProductV2 {
  id: string;
  name: string;
  price: number;
  description: string;
  category: { id: string; name: string };
  images: { url: string }[];
  _count: { reviews: number };
  _links: Record<string, string>;
}

export function transformV2toV1(product: ProductV2): ProductV1 {
  return {
    id: product.id,
    name: product.name,
    price: product.price,
    description: product.description,
  };
}

// ============================================================
// DEPRECATION NOTICE — log usage of deprecated endpoints
// ============================================================
export async function logDeprecationUsage(
  version: string,
  endpoint: string,
  clientId: string
) {
  console.warn(
    `[DEPRECATION] Client ${clientId} using deprecated ${version} endpoint: ${endpoint}`
  );

  // In production, log to analytics
  // await analytics.track('deprecated_api_usage', { version, endpoint, clientId });
}
```

### Edge Cases e Error Handling

```typescript
// lib/api/openapi-spec.ts
import { z } from "zod";

// ============================================================
// OPENAPI 3.0 SPEC GENERATOR
// ============================================================
interface OpenAPIPath {
  summary: string;
  description?: string;
  tags?: string[];
  parameters?: object[];
  requestBody?: object;
  responses: Record<string, object>;
}

interface OpenAPISpec {
  openapi: string;
  info: {
    title: string;
    version: string;
    description: string;
    contact?: { email: string };
  };
  servers: Array<{ url: string; description: string }>;
  paths: Record<string, Record<string, OpenAPIPath>>;
  components: {
    schemas: Record<string, object>;
    securitySchemes: Record<string, object>;
  };
}

export function generateOpenAPISpec(): OpenAPISpec {
  return {
    openapi: "3.0.3",
    info: {
      title: "My App API",
      version: "2.0.0",
      description: "REST API for My App - Next.js 14",
      contact: { email: "api@myapp.com" },
    },
    servers: [
      { url: "https://api.myapp.com", description: "Production" },
      { url: "https://staging-api.myapp.com", description: "Staging" },
      { url: "http://localhost:3000", description: "Development" },
    ],
    paths: {
      "/api/v2/products": {
        get: {
          summary: "List products",
          tags: ["Products"],
          parameters: [
            {
              name: "page",
              in: "query",
              schema: { type: "integer", default: 1, minimum: 1 },
            },
            {
              name: "perPage",
              in: "query",
              schema: { type: "integer", default: 20, minimum: 1, maximum: 100 },
            },
            {
              name: "search",
              in: "query",
              schema: { type: "string" },
              description: "Full-text search across name and description",
            },
            {
              name: "category",
              in: "query",
              schema: { type: "string" },
              description: "Filter by category slug",
            },
          ],
          responses: {
            "200": {
              description: "Successful response",
              content: {
                "application/json": {
                  schema: {
                    type: "object",
                    properties: {
                      success: { type: "boolean", example: true },
                      data: {
                        type: "array",
                        items: { $ref: "#/components/schemas/Product" },
                      },
                      meta: { $ref: "#/components/schemas/PaginationMeta" },
                    },
                  },
                },
              },
            },
            "429": {
              description: "Rate limit exceeded",
              content: {
                "application/problem+json": {
                  schema: { $ref: "#/components/schemas/Error" },
                },
              },
            },
          },
        },
        post: {
          summary: "Create product",
          tags: ["Products"],
          requestBody: {
            required: true,
            content: {
              "application/json": {
                schema: { $ref: "#/components/schemas/CreateProduct" },
              },
            },
          },
          responses: {
            "201": {
              description: "Product created",
              content: {
                "application/json": {
                  schema: {
                    type: "object",
                    properties: {
                      success: { type: "boolean" },
                      data: { $ref: "#/components/schemas/Product" },
                    },
                  },
                },
              },
            },
            "422": {
              description: "Validation error",
              content: {
                "application/problem+json": {
                  schema: { $ref: "#/components/schemas/ValidationError" },
                },
              },
            },
          },
        },
      },
    },
    components: {
      schemas: {
        Product: {
          type: "object",
          properties: {
            id: { type: "string", format: "uuid" },
            name: { type: "string" },
            slug: { type: "string" },
            price: { type: "number" },
            description: { type: "string" },
            status: { type: "string", enum: ["ACTIVE", "DRAFT", "ARCHIVED"] },
            category: { $ref: "#/components/schemas/CategorySummary" },
            createdAt: { type: "string", format: "date-time" },
            updatedAt: { type: "string", format: "date-time" },
          },
        },
        CategorySummary: {
          type: "object",
          properties: {
            id: { type: "string", format: "uuid" },
            name: { type: "string" },
            slug: { type: "string" },
          },
        },
        PaginationMeta: {
          type: "object",
          properties: {
            page: { type: "integer" },
            perPage: { type: "integer" },
            total: { type: "integer" },
            totalPages: { type: "integer" },
          },
        },
        Error: {
          type: "object",
          properties: {
            success: { type: "boolean", example: false },
            error: {
              type: "object",
              properties: {
                type: { type: "string" },
                title: { type: "string" },
                status: { type: "integer" },
                detail: { type: "string" },
              },
            },
          },
        },
        ValidationError: {
          type: "object",
          properties: {
            success: { type: "boolean", example: false },
            error: {
              type: "object",
              properties: {
                type: { type: "string", example: "validation_error" },
                title: { type: "string" },
                status: { type: "integer", example: 422 },
                errors: {
                  type: "array",
                  items: {
                    type: "object",
                    properties: {
                      field: { type: "string" },
                      message: { type: "string" },
                      code: { type: "string" },
                    },
                  },
                },
              },
            },
          },
        },
        CreateProduct: {
          type: "object",
          required: ["name", "slug", "description", "price", "categoryId"],
          properties: {
            name: { type: "string", minLength: 1, maxLength: 255 },
            slug: { type: "string", pattern: "^[a-z0-9-]+$" },
            description: { type: "string", minLength: 10 },
            price: { type: "number", exclusiveMinimum: 0 },
            categoryId: { type: "string", format: "uuid" },
            tags: { type: "array", items: { type: "string" } },
          },
        },
      },
      securitySchemes: {
        bearerAuth: {
          type: "http",
          scheme: "bearer",
          bearerFormat: "JWT",
        },
        apiKey: {
          type: "apiKey",
          in: "header",
          name: "X-API-Key",
        },
      },
    },
  };
}

// ============================================================
// OPENAPI ROUTE — serve spec as JSON
// ============================================================
// app/api/docs/openapi.json/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const spec = generateOpenAPISpec();
  return NextResponse.json(spec, {
    headers: {
      "Content-Type": "application/json",
      "Cache-Control": "public, max-age=3600",
    },
  });
}
```

### Errori Comuni da Evitare
- **Versioning misto**: Scegli UNA strategia (path-based è la più chiara) e usa quella consistently
- **Breaking change senza nuova versione**: Sempre creare v(N+1) per breaking changes
- **Deprecation senza notice**: Usa header `Deprecation` e `Sunset` per comunicare le timeline
- **OpenAPI out of sync**: Genera la spec dal codice (Zod schemas) per mantenerla aggiornata

### Checklist di Verifica
- [ ] Tutte le risposte includono `X-API-Version` header
- [ ] Le versioni deprecate includono `Deprecation` e `Sunset` headers
- [ ] La spec OpenAPI è servita su `/api/docs/openapi.json`
- [ ] I transformer tra versioni sono testati
- [ ] Il changelog delle API è documentato per ogni versione
- [ ] Le nuove versioni non rompono i client esistenti

---

## API-CACHING-STRATEGIES

### Panoramica
Pattern completi di caching per API Next.js 14: HTTP caching headers, stale-while-revalidate, ETag, cache invalidation, e integration con Redis.

### Implementazione Completa

```typescript
// lib/api/cache.ts
import { NextRequest, NextResponse } from "next/server";
import { Redis } from "@upstash/redis";
import crypto from "crypto";

// ============================================================
// HTTP CACHE HEADERS HELPER
// ============================================================
interface CacheConfig {
  maxAge?: number;
  sMaxAge?: number;
  staleWhileRevalidate?: number;
  private?: boolean;
  noStore?: boolean;
  mustRevalidate?: boolean;
}

export function setCacheHeaders(
  response: NextResponse,
  config: CacheConfig
): NextResponse {
  if (config.noStore) {
    response.headers.set(
      "Cache-Control",
      "no-store, no-cache, must-revalidate"
    );
    return response;
  }

  const parts: string[] = [];

  if (config.private) parts.push("private");
  else parts.push("public");

  if (config.maxAge !== undefined) parts.push(`max-age=${config.maxAge}`);
  if (config.sMaxAge !== undefined) parts.push(`s-maxage=${config.sMaxAge}`);
  if (config.staleWhileRevalidate !== undefined) {
    parts.push(`stale-while-revalidate=${config.staleWhileRevalidate}`);
  }
  if (config.mustRevalidate) parts.push("must-revalidate");

  response.headers.set("Cache-Control", parts.join(", "));
  return response;
}

// ============================================================
// ETAG SUPPORT
// ============================================================
export function generateETag(data: unknown): string {
  const hash = crypto
    .createHash("md5")
    .update(JSON.stringify(data))
    .digest("hex");
  return `"${hash}"`;
}

export function handleConditionalRequest(
  req: NextRequest,
  data: unknown,
  response: NextResponse
): NextResponse {
  const etag = generateETag(data);
  response.headers.set("ETag", etag);

  const ifNoneMatch = req.headers.get("if-none-match");
  if (ifNoneMatch === etag) {
    return new NextResponse(null, {
      status: 304,
      headers: { ETag: etag },
    });
  }

  return response;
}

// ============================================================
// REDIS CACHE LAYER
// ============================================================
const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

interface CacheOptions {
  ttl: number;
  tags?: string[];
}

export async function cacheGet<T>(key: string): Promise<T | null> {
  const cached = await redis.get(key);
  if (!cached) return null;
  return cached as T;
}

export async function cacheSet<T>(
  key: string,
  data: T,
  options: CacheOptions
): Promise<void> {
  await redis.set(key, JSON.stringify(data), { ex: options.ttl });

  // Store cache tags for invalidation
  if (options.tags) {
    for (const tag of options.tags) {
      await redis.sadd(`cache:tag:${tag}`, key);
      await redis.expire(`cache:tag:${tag}`, options.ttl + 60);
    }
  }
}

export async function cacheInvalidateByTag(tag: string): Promise<void> {
  const keys = await redis.smembers(`cache:tag:${tag}`);
  if (keys.length > 0) {
    await redis.del(...keys, `cache:tag:${tag}`);
  }
}

export async function cacheInvalidate(key: string): Promise<void> {
  await redis.del(key);
}

// ============================================================
// CACHED HANDLER WRAPPER
// ============================================================
export function withCache<T>(
  keyFn: (req: NextRequest) => string,
  options: CacheOptions,
  handler: (req: NextRequest) => Promise<T>
) {
  return async (req: NextRequest): Promise<NextResponse> => {
    if (req.method !== "GET") {
      const result = await handler(req);
      return NextResponse.json(result);
    }

    const cacheKey = keyFn(req);
    const cached = await cacheGet<T>(cacheKey);

    if (cached) {
      const response = NextResponse.json(cached);
      response.headers.set("X-Cache", "HIT");
      setCacheHeaders(response, {
        sMaxAge: options.ttl,
        staleWhileRevalidate: options.ttl * 2,
      });
      return handleConditionalRequest(req, cached, response);
    }

    const result = await handler(req);
    await cacheSet(cacheKey, result, options);

    const response = NextResponse.json(result);
    response.headers.set("X-Cache", "MISS");
    setCacheHeaders(response, {
      sMaxAge: options.ttl,
      staleWhileRevalidate: options.ttl * 2,
    });
    return handleConditionalRequest(req, result, response);
  };
}
```

### Varianti e Configurazioni

```typescript
// app/api/categories/route.ts — cached endpoint example
import { NextRequest } from "next/server";
import { prisma } from "@/lib/prisma";
import { withCache, cacheInvalidateByTag } from "@/lib/api/cache";
import { apiSuccess, apiCreated } from "@/lib/api/response";

// ============================================================
// GET /api/categories — cached for 5 minutes
// ============================================================
export const GET = withCache(
  (req) => {
    const params = req.nextUrl.searchParams.toString();
    return `categories:list:${params}`;
  },
  { ttl: 300, tags: ["categories"] },
  async (req) => {
    const categories = await prisma.category.findMany({
      include: { _count: { select: { products: true } } },
      orderBy: { name: "asc" },
    });

    return { success: true, data: categories };
  }
);

// ============================================================
// POST /api/categories — invalidate cache on create
// ============================================================
export async function POST(req: NextRequest) {
  const body = await req.json();

  const category = await prisma.category.create({
    data: body,
  });

  // Invalidate all category caches
  await cacheInvalidateByTag("categories");

  return apiCreated(category);
}
```

### Edge Cases e Error Handling

```typescript
// lib/api/cache-warming.ts

// ============================================================
// CACHE WARMING — pre-populate cache on deploy
// ============================================================
export async function warmCache(): Promise<void> {
  const endpoints = [
    "/api/categories",
    "/api/products?page=1&perPage=20",
    "/api/settings",
  ];

  const baseUrl = process.env.NEXT_PUBLIC_APP_URL ?? "http://localhost:3000";

  await Promise.allSettled(
    endpoints.map(async (endpoint) => {
      const response = await fetch(`${baseUrl}${endpoint}`, {
        headers: { "X-Cache-Warm": "true" },
      });
      console.log(`Cache warm: ${endpoint} - ${response.status}`);
    })
  );
}

// ============================================================
// STALE CONTENT HANDLER — serve stale on error
// ============================================================
export async function staleOnError<T>(
  cacheKey: string,
  fetcher: () => Promise<T>,
  ttl: number
): Promise<{ data: T; stale: boolean }> {
  try {
    const fresh = await fetcher();
    await cacheSet(cacheKey, fresh, { ttl });
    return { data: fresh, stale: false };
  } catch (error) {
    const stale = await cacheGet<T>(cacheKey);
    if (stale) {
      console.warn(`Serving stale cache for ${cacheKey}:`, error);
      return { data: stale, stale: true };
    }
    throw error;
  }
}
```

### Errori Comuni da Evitare
- **Cache su dati personalizzati**: Non cachare risposte che contengono dati specifici dell'utente con cache pubblica
- **Missing cache invalidation**: Ogni write (POST/PUT/PATCH/DELETE) deve invalidare le cache relative
- **ETag su risposte paginate**: L'ETag deve includere i parametri di paginazione nel calcolo
- **Cache stampede**: Usa locking o probabilistic early expiration per prevenire thundering herd

### Checklist di Verifica
- [ ] I GET endpoints hanno `Cache-Control` headers appropriati
- [ ] Le risposte includono `ETag` per conditional requests
- [ ] La cache è invalidata su ogni operazione di write
- [ ] I dati sensibili/personalizzati usano `private` cache
- [ ] Il cache warming è configurato per gli endpoint critici
- [ ] Lo stale-while-revalidate è usato per dati che tollerano stale content



---

## API-WEBHOOKS-OUTBOUND

### Panoramica
Pattern per implementare outbound webhooks: registrazione endpoint, delivery con retry, firma delle request, event queue, e dashboard di monitoraggio.

### Implementazione Completa

```typescript
// lib/api/webhooks/outbound.ts
import crypto from "crypto";
import { prisma } from "@/lib/prisma";

// ============================================================
// WEBHOOK EVENT TYPES
// ============================================================
export const WEBHOOK_EVENTS = [
  "order.created",
  "order.updated",
  "order.completed",
  "order.cancelled",
  "payment.succeeded",
  "payment.failed",
  "payment.refunded",
  "product.created",
  "product.updated",
  "product.deleted",
  "customer.created",
  "customer.updated",
  "subscription.created",
  "subscription.cancelled",
  "subscription.renewed",
] as const;

export type WebhookEvent = (typeof WEBHOOK_EVENTS)[number];

// ============================================================
// WEBHOOK SIGNATURE
// ============================================================
export function signWebhookPayload(
  payload: string,
  secret: string,
  timestamp: number
): string {
  const signedPayload = `${timestamp}.${payload}`;
  return crypto
    .createHmac("sha256", secret)
    .update(signedPayload)
    .digest("hex");
}

export function generateWebhookSecret(): string {
  return `whsec_${crypto.randomBytes(32).toString("hex")}`;
}

// ============================================================
// WEBHOOK DELIVERY
// ============================================================
interface WebhookDeliveryResult {
  success: boolean;
  statusCode: number | null;
  responseBody: string | null;
  duration: number;
  error: string | null;
}

export async function deliverWebhook(
  url: string,
  event: WebhookEvent,
  payload: Record<string, unknown>,
  secret: string
): Promise<WebhookDeliveryResult> {
  const timestamp = Math.floor(Date.now() / 1000);
  const body = JSON.stringify({
    id: crypto.randomUUID(),
    event,
    created_at: new Date().toISOString(),
    data: payload,
  });

  const signature = signWebhookPayload(body, secret, timestamp);
  const startTime = Date.now();

  try {
    const controller = new AbortController();
    const timeout = setTimeout(() => controller.abort(), 30_000);

    const response = await fetch(url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "X-Webhook-Id": crypto.randomUUID(),
        "X-Webhook-Timestamp": timestamp.toString(),
        "X-Webhook-Signature": `v1=${signature}`,
        "User-Agent": "MyApp-Webhooks/1.0",
      },
      body,
      signal: controller.signal,
    });

    clearTimeout(timeout);

    const responseBody = await response.text().catch(() => null);
    const duration = Date.now() - startTime;

    return {
      success: response.ok,
      statusCode: response.status,
      responseBody: responseBody?.substring(0, 1000) ?? null,
      duration,
      error: response.ok ? null : `HTTP ${response.status}`,
    };
  } catch (error) {
    const duration = Date.now() - startTime;
    return {
      success: false,
      statusCode: null,
      responseBody: null,
      duration,
      error: error instanceof Error ? error.message : "Unknown error",
    };
  }
}

// ============================================================
// WEBHOOK DISPATCH WITH RETRY
// ============================================================
const RETRY_DELAYS = [0, 60, 300, 1800, 7200, 21600]; // seconds

export async function dispatchWebhook(
  event: WebhookEvent,
  payload: Record<string, unknown>
): Promise<void> {
  const subscriptions = await prisma.webhookSubscription.findMany({
    where: {
      isActive: true,
      events: { has: event },
    },
  });

  for (const sub of subscriptions) {
    await prisma.webhookDelivery.create({
      data: {
        subscriptionId: sub.id,
        event,
        payload: payload as any,
        status: "PENDING",
        attemptCount: 0,
        maxAttempts: RETRY_DELAYS.length,
        nextAttemptAt: new Date(),
      },
    });
  }
}

// ============================================================
// WEBHOOK WORKER — process delivery queue
// ============================================================
export async function processWebhookQueue(): Promise<number> {
  const pendingDeliveries = await prisma.webhookDelivery.findMany({
    where: {
      status: { in: ["PENDING", "RETRYING"] },
      nextAttemptAt: { lte: new Date() },
    },
    include: { subscription: true },
    take: 50,
    orderBy: { nextAttemptAt: "asc" },
  });

  let processed = 0;

  for (const delivery of pendingDeliveries) {
    const result = await deliverWebhook(
      delivery.subscription.url,
      delivery.event as WebhookEvent,
      delivery.payload as Record<string, unknown>,
      delivery.subscription.secret
    );

    if (result.success) {
      await prisma.webhookDelivery.update({
        where: { id: delivery.id },
        data: {
          status: "DELIVERED",
          attemptCount: delivery.attemptCount + 1,
          lastAttemptAt: new Date(),
          statusCode: result.statusCode,
          responseBody: result.responseBody,
          duration: result.duration,
        },
      });
    } else {
      const nextAttempt = delivery.attemptCount + 1;
      const isLastAttempt = nextAttempt >= delivery.maxAttempts;

      await prisma.webhookDelivery.update({
        where: { id: delivery.id },
        data: {
          status: isLastAttempt ? "FAILED" : "RETRYING",
          attemptCount: nextAttempt,
          lastAttemptAt: new Date(),
          statusCode: result.statusCode,
          responseBody: result.responseBody,
          duration: result.duration,
          error: result.error,
          nextAttemptAt: isLastAttempt
            ? null
            : new Date(Date.now() + RETRY_DELAYS[nextAttempt] * 1000),
        },
      });

      if (isLastAttempt) {
        // Auto-disable subscription after too many failures
        const recentFailures = await prisma.webhookDelivery.count({
          where: {
            subscriptionId: delivery.subscriptionId,
            status: "FAILED",
            createdAt: { gte: new Date(Date.now() - 24 * 60 * 60 * 1000) },
          },
        });

        if (recentFailures >= 10) {
          await prisma.webhookSubscription.update({
            where: { id: delivery.subscriptionId },
            data: { isActive: false, disabledReason: "Too many failures" },
          });
        }
      }
    }

    processed++;
  }

  return processed;
}
```

### Varianti e Configurazioni

```typescript
// app/api/webhooks/subscriptions/route.ts
import { NextRequest } from "next/server";
import { z } from "zod";
import { prisma } from "@/lib/prisma";
import { createHandler } from "@/lib/api/route-handler";
import { apiSuccess, apiCreated } from "@/lib/api/response";
import { generateWebhookSecret, WEBHOOK_EVENTS } from "@/lib/api/webhooks/outbound";

const createSubscriptionSchema = z.object({
  url: z.string().url().startsWith("https://", "Webhook URL must use HTTPS"),
  events: z.array(z.enum(WEBHOOK_EVENTS)).min(1),
  description: z.string().max(255).optional(),
});

// ============================================================
// POST /api/webhooks/subscriptions — register webhook
// ============================================================
export const POST = createHandler(
  {
    auth: true,
    roles: ["ADMIN"],
    bodySchema: createSubscriptionSchema,
  },
  async ({ body, user }) => {
    const secret = generateWebhookSecret();

    const subscription = await prisma.webhookSubscription.create({
      data: {
        url: body.url,
        events: body.events,
        description: body.description,
        secret,
        ownerId: user!.id,
        isActive: true,
      },
    });

    return apiCreated({
      ...subscription,
      secret, // Show secret only on creation
    });
  }
);

// ============================================================
// GET /api/webhooks/subscriptions — list webhooks
// ============================================================
export const GET = createHandler(
  { auth: true, roles: ["ADMIN"] },
  async ({ user }) => {
    const subscriptions = await prisma.webhookSubscription.findMany({
      where: { ownerId: user!.id },
      select: {
        id: true,
        url: true,
        events: true,
        description: true,
        isActive: true,
        disabledReason: true,
        createdAt: true,
        _count: { select: { deliveries: true } },
      },
      orderBy: { createdAt: "desc" },
    });

    return apiSuccess(subscriptions);
  }
);
```

### Edge Cases e Error Handling

```typescript
// app/api/webhooks/test/route.ts
import { NextRequest } from "next/server";
import { z } from "zod";
import { createHandler } from "@/lib/api/route-handler";
import { apiSuccess, apiNotFound } from "@/lib/api/response";
import { deliverWebhook } from "@/lib/api/webhooks/outbound";
import { prisma } from "@/lib/prisma";

const testSchema = z.object({
  subscriptionId: z.string().uuid(),
  event: z.string().default("test.ping"),
});

// ============================================================
// POST /api/webhooks/test — send test webhook
// ============================================================
export const POST = createHandler(
  { auth: true, roles: ["ADMIN"], bodySchema: testSchema },
  async ({ body }) => {
    const sub = await prisma.webhookSubscription.findUnique({
      where: { id: body.subscriptionId },
    });

    if (!sub) return apiNotFound("WebhookSubscription", body.subscriptionId);

    const result = await deliverWebhook(
      sub.url,
      body.event as any,
      { message: "This is a test webhook delivery", timestamp: new Date().toISOString() },
      sub.secret
    );

    return apiSuccess({
      delivered: result.success,
      statusCode: result.statusCode,
      duration: result.duration,
      error: result.error,
    });
  }
);
```

### Errori Comuni da Evitare
- **No HTTPS enforcement**: Accetta solo URL HTTPS per sicurezza
- **Missing timeout**: Imposta un timeout di 30s per ogni delivery per evitare blocking
- **Secret esposto nei log**: Non loggare mai il webhook secret
- **Retry senza backoff**: Usa exponential backoff (60s, 5m, 30m, 2h, 6h)

### Checklist di Verifica
- [ ] I payload sono firmati con HMAC-SHA256
- [ ] L'URL webhook accetta solo HTTPS
- [ ] Il retry usa exponential backoff
- [ ] Le subscription vengono disabilitate dopo troppi fallimenti
- [ ] Il secret viene mostrato solo alla creazione
- [ ] La queue ha un worker che processa le delivery in batch
- [ ] I test webhook sono disponibili per debugging
