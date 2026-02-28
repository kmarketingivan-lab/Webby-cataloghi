# CATALOGO ERROR-HANDLING v1

§ §1. ERROR TYPES TAXONOMY

| Type | Example | Handling | User Message |
|------|---------|----------|--------------|
| Validation Error | Invalid email | Show field error | "Please enter valid email" |
| Authentication | Not logged in | Redirect to login | "Please sign in" |
| Authorization | No permission | Show forbidden | "You don't have access" |
| Not Found | 404 | Show 404 page | "Page not found" |
| Network Error | Fetch failed | Retry + fallback | "Connection error" |
| Server Error | 500 | Log + generic msg | "Something went wrong" |
| Rate Limit | 429 | Wait + retry | "Too many requests" |
| Business Logic | Out of stock | Show specific msg | "Item out of stock" |

§ §2. ERROR BOUNDARY SYSTEM

§ 2.1 GLOBAL ERROR BOUNDARY

typescript
// app/error.tsx (Next.js App Router)
'use client';

import { useEffect } from 'react';
import { captureException } from '@sentry/nextjs';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { AlertCircle, RefreshCw, Home } from 'lucide-react';

interface GlobalErrorProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function GlobalError({ error, reset }: GlobalErrorProps) {
  useEffect(() => {
    // Log error to Sentry
    captureException(error, {
      tags: { type: 'global_error' },
      extra: {
        digest: error.digest,
        timestamp: new Date().toISOString(),
      },
    });
    
    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('Global error caught:', error);
    }
  }, [error]);

  return (
    <html lang="en">
      <body className="min-h-screen bg-background">
        <main className="container mx-auto px-4 py-16 flex items-center justify-center">
          <Card className="w-full max-w-md">
            <CardHeader>
              <div className="flex items-center gap-3">
                <AlertCircle className="h-8 w-8 text-destructive" />
                <CardTitle className="text-2xl">Something went wrong</CardTitle>
              </div>
            </CardHeader>
            
            <CardContent className="space-y-4">
              <p className="text-muted-foreground">
                We apologize for the inconvenience. Our team has been notified and is working to fix the issue.
              </p>
              
              {process.env.NODE_ENV === 'development' && (
                <div className="mt-4 p-3 bg-muted rounded-lg overflow-auto">
                  <pre className="text-xs text-muted-foreground">
                    {error.message}
                    {error.digest && `\n\nDigest: ${error.digest}`}
                  </pre>
                </div>
              )}
              
              <div className="flex items-center gap-2 text-sm text-muted-foreground">
                <AlertCircle className="h-4 w-4" />
                <span>Error ID: {error.digest || 'N/A'}</span>
              </div>
            </CardContent>
            
            <CardFooter className="flex flex-col sm:flex-row gap-3">
              <Button
                onClick={reset}
                className="flex-1 gap-2"
                variant="default"
              >
                <RefreshCw className="h-4 w-4" />
                Try again
              </Button>
              
              <Button
                onClick={() => window.location.href = '/'}
                className="flex-1 gap-2"
                variant="outline"
              >
                <Home className="h-4 w-4" />
                Go home
              </Button>
            </CardFooter>
          </Card>
        </main>
      </body>
    </html>
  );
}

§ 2.2 SEGMENT ERROR BOUNDARIES

typescript
// app/dashboard/error.tsx (Example per segment)
'use client';

import { useEffect } from 'react';
import { captureException } from '@sentry/nextjs';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { AlertCircle, RefreshCw } from 'lucide-react';

interface DashboardErrorProps {
  error: Error & { digest?: string };
  reset: () => void;
}

export default function DashboardError({ error, reset }: DashboardErrorProps) {
  useEffect(() => {
    captureException(error, {
      tags: { type: 'dashboard_error' },
      extra: {
        segment: 'dashboard',
        digest: error.digest,
      },
    });
  }, [error]);

  return (
    <div className="flex-1 flex flex-col items-center justify-center p-8">
      <Card className="w-full max-w-md">
        <CardHeader>
          <div className="flex items-center gap-3">
            <AlertCircle className="h-8 w-8 text-amber-500" />
            <CardTitle className="text-2xl">Dashboard Error</CardTitle>
          </div>
        </CardHeader>
        
        <CardContent className="space-y-4">
          <p className="text-muted-foreground">
            There was a problem loading your dashboard data. This section may be temporarily unavailable.
          </p>
          
          <div className="bg-amber-50 dark:bg-amber-900/20 border border-amber-200 dark:border-amber-800 rounded-lg p-4">
            <p className="text-sm text-amber-800 dark:text-amber-200">
              <strong>Tip:</strong> Try refreshing the page. If the problem persists, contact support.
            </p>
          </div>
        </CardContent>
        
        <CardFooter>
          <Button
            onClick={reset}
            className="w-full gap-2"
            variant="default"
          >
            <RefreshCw className="h-4 w-4" />
            Reload Dashboard
          </Button>
        </CardFooter>
      </Card>
    </div>
  );
}

// app/api/[segment]/error.tsx pattern
// app/products/error.tsx
'use client';

import { Button } from '@/components/ui/button';
import { AlertTriangle } from 'lucide-react';

interface ProductsErrorProps {
  error: Error;
  reset: () => void;
}

export default function ProductsError({ error, reset }: ProductsErrorProps) {
  return (
    <div className="flex flex-col items-center justify-center p-8 text-center">
      <AlertTriangle className="h-16 w-16 text-amber-500 mb-4" />
      <h2 className="text-2xl font-bold mb-2">Unable to load products</h2>
      <p className="text-muted-foreground mb-6">
        We're having trouble loading the product catalog. Please try again.
      </p>
      <div className="flex gap-3">
        <Button onClick={reset} variant="default">
          Try again
        </Button>
        <Button variant="outline" onClick={() => window.location.reload()}>
          Refresh page
        </Button>
      </div>
    </div>
  );
}

§ 2.3 COMPONENT ERROR BOUNDARY

typescript
// components/ErrorBoundary.tsx
'use client';

import React, { Component, ErrorInfo, ReactNode } from 'react';
import { captureException } from '@sentry/nextjs';
import { cn } from '@/lib/utils';
import { AlertCircle, RefreshCw } from 'lucide-react';
import { Button } from '@/components/ui/button';

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
  className?: string;
  componentName?: string;
}

interface ErrorBoundaryState {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    // Log to Sentry with component context
    captureException(error, {
      tags: {
        component_boundary: true,
        component_name: this.props.componentName || 'Unknown',
      },
      extra: {
        componentStack: errorInfo.componentStack,
        timestamp: new Date().toISOString(),
      },
    });

    // Call custom error handler if provided
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }

    // Log to console in development
    if (process.env.NODE_ENV === 'development') {
      console.error('Component error:', error);
      console.error('Component stack:', errorInfo.componentStack);
    }
  }

  resetError = (): void => {
    this.setState({ hasError: false, error: undefined });
  };

  render(): ReactNode {
    if (this.state.hasError) {
      // Use custom fallback if provided
      if (this.props.fallback) {
        return this.props.fallback;
      }

      // Default fallback UI
      return (
        <div className={cn('p-6 border rounded-lg bg-card', this.props.className)}>
          <div className="flex flex-col items-center justify-center text-center space-y-4">
            <AlertCircle className="h-12 w-12 text-destructive" />
            
            <div className="space-y-2">
              <h3 className="text-lg font-semibold">Component Error</h3>
              <p className="text-sm text-muted-foreground">
                {this.props.componentName 
                  ? `The ${this.props.componentName} component encountered an error.`
                  : 'This component encountered an error.'
                }
              </p>
            </div>

            {process.env.NODE_ENV === 'development' && this.state.error && (
              <div className="w-full p-3 bg-muted rounded-lg overflow-auto">
                <pre className="text-xs text-muted-foreground">
                  {this.state.error.message}
                  {this.state.error.stack && `\n\n${this.state.error.stack}`}
                </pre>
              </div>
            )}

            <div className="flex gap-2">
              <Button
                size="sm"
                onClick={this.resetError}
                variant="default"
                className="gap-2"
              >
                <RefreshCw className="h-3 w-3" />
                Retry component
              </Button>
            </div>
          </div>
        </div>
      );
    }

    return this.props.children;
  }
}

// Higher-order component wrapper
export function withErrorBoundary<P extends object>(
  Component: React.ComponentType<P>,
  options?: Omit<ErrorBoundaryProps, 'children'>
): React.ComponentType<P> {
  const WrappedComponent = (props: P) => (
    <ErrorBoundary {...options}>
      <Component {...props} />
    </ErrorBoundary>
  );

  // Set display name for debugging
  WrappedComponent.displayName = `withErrorBoundary(${Component.displayName || Component.name})`;

  return WrappedComponent;
}

// Hook for functional components
export function useErrorBoundary() {
  const [error, setError] = React.useState<Error | null>(null);

  const handleError = React.useCallback((error: Error) => {
    setError(error);
    captureException(error, {
      tags: { hook_error: true },
    });
  }, []);

  const resetError = React.useCallback(() => {
    setError(null);
  }, []);

  return {
    error,
    handleError,
    resetError,
    ErrorBoundary: ({ children, fallback }: { children: ReactNode; fallback?: ReactNode }) => {
      if (error) {
        return fallback || (
          <div className="p-4 border border-destructive/20 bg-destructive/10 rounded-lg">
            <p className="text-sm text-destructive">
              Error: {error.message}
            </p>
            <Button
              size="sm"
              variant="outline"
              onClick={resetError}
              className="mt-2"
            >
              Retry
            </Button>
          </div>
        );
      }
      return <>{children}</>;
    },
  };
}

§ §3. API ERROR HANDLING

§ 3.1 CUSTOM ERROR CLASSES

typescript
// lib/errors/index.ts
export type ErrorCode = 
  | 'VALIDATION_ERROR'
  | 'AUTHENTICATION_ERROR'
  | 'AUTHORIZATION_ERROR'
  | 'NOT_FOUND_ERROR'
  | 'CONFLICT_ERROR'
  | 'RATE_LIMIT_ERROR'
  | 'EXTERNAL_SERVICE_ERROR'
  | 'INTERNAL_SERVER_ERROR'
  | 'BAD_REQUEST_ERROR';

export interface ErrorDetails {
  [key: string]: string[];
}

export class AppError extends Error {
  public readonly code: ErrorCode;
  public readonly statusCode: number;
  public readonly details?: ErrorDetails;
  public readonly timestamp: Date;
  public readonly requestId?: string;

  constructor(
    message: string,
    code: ErrorCode = 'INTERNAL_SERVER_ERROR',
    statusCode: number = 500,
    details?: ErrorDetails,
    requestId?: string
  ) {
    super(message);
    this.name = this.constructor.name;
    this.code = code;
    this.statusCode = statusCode;
    this.details = details;
    this.timestamp = new Date();
    this.requestId = requestId;

    // Capture stack trace
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON() {
    return {
      success: false,
      error: {
        code: this.code,
        message: this.message,
        details: this.details,
        requestId: this.requestId,
        timestamp: this.timestamp.toISOString(),
      },
    };
  }
}

export class ValidationError extends AppError {
  constructor(message: string, details?: ErrorDetails, requestId?: string) {
    super(message, 'VALIDATION_ERROR', 400, details, requestId);
  }
}

export class AuthenticationError extends AppError {
  constructor(message = 'Authentication required', requestId?: string) {
    super(message, 'AUTHENTICATION_ERROR', 401, undefined, requestId);
  }
}

export class AuthorizationError extends AppError {
  constructor(message = 'You do not have permission to access this resource', requestId?: string) {
    super(message, 'AUTHORIZATION_ERROR', 403, undefined, requestId);
  }
}

export class NotFoundError extends AppError {
  constructor(message = 'Resource not found', requestId?: string) {
    super(message, 'NOT_FOUND_ERROR', 404, undefined, requestId);
  }
}

export class ConflictError extends AppError {
  constructor(message = 'Resource conflict', details?: ErrorDetails, requestId?: string) {
    super(message, 'CONFLICT_ERROR', 409, details, requestId);
  }
}

export class RateLimitError extends AppError {
  constructor(message = 'Rate limit exceeded', retryAfter?: number, requestId?: string) {
    super(message, 'RATE_LIMIT_ERROR', 429, undefined, requestId);
    if (retryAfter) {
      this.statusCode = 429;
    }
  }
}

export class ExternalServiceError extends AppError {
  constructor(
    message = 'External service error',
    serviceName?: string,
    requestId?: string
  ) {
    super(
      message,
      'EXTERNAL_SERVICE_ERROR',
      502,
      serviceName ? { service: [serviceName] } : undefined,
      requestId
    );
  }
}

export class BadRequestError extends AppError {
  constructor(message = 'Bad request', details?: ErrorDetails, requestId?: string) {
    super(message, 'BAD_REQUEST_ERROR', 400, details, requestId);
  }
}

// Utility function to check error type
export function isAppError(error: unknown): error is AppError {
  return error instanceof AppError;
}

// Utility to create validation errors from Zod errors
import { ZodError } from 'zod';

export function fromZodError(zodError: ZodError): ValidationError {
  const details: ErrorDetails = {};
  
  zodError.errors.forEach((error) => {
    const path = error.path.join('.');
    if (!details[path]) {
      details[path] = [];
    }
    details[path].push(error.message);
  });

  return new ValidationError('Validation failed', details);
}

§ 3.2 API ROUTE ERROR HANDLER

typescript
// lib/api/error-handler.ts
import { NextResponse } from 'next/server';
import { captureException } from '@sentry/nextjs';
import { AppError, isAppError, ErrorCode } from '@/lib/errors';
import { ZodError } from 'zod';

export interface ErrorResponse {
  success: false;
  error: {
    code: ErrorCode;
    message: string;
    details?: Record<string, string[]>;
    requestId?: string;
    timestamp: string;
  };
}

export function handleApiError(error: unknown): NextResponse<ErrorResponse> {
  // Generate request ID for tracing
  const requestId = crypto.randomUUID();
  
  // Handle Zod validation errors
  if (error instanceof ZodError) {
    const details: Record<string, string[]> = {};
    error.errors.forEach((err) => {
      const path = err.path.join('.');
      if (!details[path]) details[path] = [];
      details[path].push(err.message);
    });
    
    const validationError: ErrorResponse = {
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Validation failed',
        details,
        requestId,
        timestamp: new Date().toISOString(),
      },
    };
    
    return NextResponse.json(validationError, { status: 400 });
  }
  
  // Handle custom AppError
  if (isAppError(error)) {
    // Log non-client errors to Sentry
    if (error.statusCode >= 500) {
      captureException(error, {
        tags: {
          error_code: error.code,
          status_code: error.statusCode.toString(),
          request_id: requestId,
        },
        extra: {
          details: error.details,
          timestamp: error.timestamp,
        },
      });
    }
    
    const response: ErrorResponse = {
      success: false,
      error: {
        code: error.code,
        message: error.message,
        details: error.details,
        requestId,
        timestamp: error.timestamp.toISOString(),
      },
    };
    
    return NextResponse.json(response, { status: error.statusCode });
  }
  
  // Handle generic errors
  const isDev = process.env.NODE_ENV === 'development';
  const message = isDev 
    ? error instanceof Error ? error.message : 'Unknown error'
    : 'Internal server error';
  
  // Log all unknown errors to Sentry
  captureException(error, {
    tags: {
      error_type: 'unhandled',
      request_id: requestId,
    },
    extra: {
      original_error: error,
    },
  });
  
  const response: ErrorResponse = {
    success: false,
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message,
      requestId,
      timestamp: new Date().toISOString(),
    },
  };
  
  // Don't leak internal details in production
  if (!isDev && error instanceof Error) {
    console.error(`Internal error [${requestId}]:`, error.message);
  } else {
    console.error(`Error [${requestId}]:`, error);
  }
  
  return NextResponse.json(response, { status: 500 });
}

// Middleware wrapper for API routes
import { NextRequest, NextResponse } from 'next/server';

export function withErrorHandler(
  handler: (req: NextRequest, ...args: any[]) => Promise<NextResponse>
) {
  return async (req: NextRequest, ...args: any[]) => {
    try {
      return await handler(req, ...args);
    } catch (error) {
      return handleApiError(error);
    }
  };
}

// API route utility functions
export function successResponse<T>(data: T, status = 200): NextResponse {
  return NextResponse.json({
    success: true,
    data,
    timestamp: new Date().toISOString(),
  }, { status });
}

export function errorResponse(
  message: string,
  code: ErrorCode,
  status: number,
  details?: Record<string, string[]>
): NextResponse<ErrorResponse> {
  return NextResponse.json({
    success: false,
    error: {
      code,
      message,
      details,
      timestamp: new Date().toISOString(),
    },
  }, { status });
}

§ 3.3 ERROR RESPONSE FORMAT

typescript
// Example usage in API routes
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { withErrorHandler, successResponse } from '@/lib/api/error-handler';
import { NotFoundError, ValidationError } from '@/lib/errors';
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2),
});

async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const data = createUserSchema.parse(body);
    
    // Business logic...
    if (body.email.includes('test')) {
      throw new ValidationError('Test emails are not allowed', {
        email: ['Test emails are not allowed'],
      });
    }
    
    const user = { id: '1', ...data };
    return successResponse(user, 201);
  } catch (error) {
    // Let the error handler deal with it
    throw error;
  }
}

export const POST = withErrorHandler(POST);

// Example error response:
// {
//   "success": false,
//   "error": {
//     "code": "VALIDATION_ERROR",
//     "message": "Validation failed",
//     "details": {
//       "email": ["Invalid email format"]
//     },
//     "requestId": "123e4567-e89b-12d3-a456-426614174000",
//     "timestamp": "2024-01-15T10:30:00.000Z"
//   }
// }

§ §4. CLIENT-SIDE ERROR HANDLING

§ 4.1 FETCH ERROR WRAPPER

typescript
// lib/fetch.ts
import { AppError, ValidationError, AuthenticationError, AuthorizationError, NotFoundError, RateLimitError } from '@/lib/errors';

export interface FetchOptions extends RequestInit {
  timeout?: number;
  retries?: number;
  retryDelay?: number;
  retryOn?: number[];
  onUploadProgress?: (progress: number) => void;
}

export interface FetchResponse<T = any> {
  data: T;
  response: Response;
  requestId?: string;
  timestamp: string;
}

export class FetchError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string,
    public details?: Record<string, string[]>,
    public requestId?: string
  ) {
    super(message);
    this.name = 'FetchError';
  }
}

async function fetchWithTimeout(
  input: RequestInfo,
  init: RequestInit = {},
  timeout = 10000
): Promise<Response> {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(input, {
      ...init,
      signal: controller.signal,
    });
    clearTimeout(timeoutId);
    return response;
  } catch (error) {
    clearTimeout(timeoutId);
    throw error;
  }
}

async function fetchWithRetry<T>(
  input: RequestInfo,
  options: FetchOptions = {},
  retryCount = 0
): Promise<FetchResponse<T>> {
  const {
    timeout = 10000,
    retries = 3,
    retryDelay = 1000,
    retryOn = [408, 429, 500, 502, 503, 504],
    ...fetchOptions
  } = options;

  try {
    const response = await fetchWithTimeout(input, fetchOptions, timeout);
    
    // Handle rate limiting
    if (response.status === 429) {
      const retryAfter = response.headers.get('Retry-After');
      const delay = retryAfter ? parseInt(retryAfter) * 1000 : retryDelay;
      throw new RateLimitError('Rate limit exceeded', delay);
    }
    
    const data = await response.json();
    
    if (!response.ok) {
      // Handle structured error responses
      if (data?.error) {
        const { code, message, details, requestId } = data.error;
        throw new FetchError(message, response.status, code, details, requestId);
      }
      
      // Handle non-structured errors
      throw new FetchError(
        data?.message || `HTTP error ${response.status}`,
        response.status
      );
    }
    
    return {
      data: data.data || data,
      response,
      requestId: data.error?.requestId,
      timestamp: data.timestamp,
    };
  } catch (error) {
    // Retry logic
    const shouldRetry = 
      retryCount < retries &&
      (error instanceof FetchError 
        ? retryOn.includes(error.status)
        : error instanceof TypeError || // Network errors
          error.name === 'AbortError' || // Timeout
          error instanceof RateLimitError);
    
    if (shouldRetry) {
      const delay = error instanceof RateLimitError 
        ? error.statusCode 
        : retryDelay * Math.pow(2, retryCount); // Exponential backoff
      
      await new Promise(resolve => setTimeout(resolve, delay));
      return fetchWithRetry(input, options, retryCount + 1);
    }
    
    // Re-throw with proper typing
    if (error instanceof FetchError || error instanceof AppError) {
      throw error;
    }
    
    // Wrap unknown errors
    if (error instanceof Error) {
      throw new FetchError(error.message, 0);
    }
    
    throw new FetchError('Unknown fetch error', 0);
  }
}

// Main fetch function
export async function apiFetch<T = any>(
  input: RequestInfo,
  options: FetchOptions = {}
): Promise<FetchResponse<T>> {
  const defaultHeaders: Record<string, string> = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  };
  
  const config: FetchOptions = {
    ...options,
    headers: {
      ...defaultHeaders,
      ...options.headers,
    },
  };
  
  return fetchWithRetry<T>(input, config);
}

// Convenience methods
export const api = {
  get<T>(url: string, options?: FetchOptions) {
    return apiFetch<T>(url, { ...options, method: 'GET' });
  },
  
  post<T>(url: string, data?: any, options?: FetchOptions) {
    return apiFetch<T>(url, {
      ...options,
      method: 'POST',
      body: data ? JSON.stringify(data) : undefined,
    });
  },
  
  put<T>(url: string, data?: any, options?: FetchOptions) {
    return apiFetch<T>(url, {
      ...options,
      method: 'PUT',
      body: data ? JSON.stringify(data) : undefined,
    });
  },
  
  patch<T>(url: string, data?: any, options?: FetchOptions) {
    return apiFetch<T>(url, {
      ...options,
      method: 'PATCH',
      body: data ? JSON.stringify(data) : undefined,
    });
  },
  
  delete<T>(url: string, options?: FetchOptions) {
    return apiFetch<T>(url, { ...options, method: 'DELETE' });
  },
};

// Hook for using fetch in React components
import { useState, useCallback, useRef } from 'react';

export interface UseFetchOptions<T> extends FetchOptions {
  onSuccess?: (data: T) => void;
  onError?: (error: FetchError) => void;
  enabled?: boolean;
}

export function useFetch<T = any>() {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<FetchError | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const abortControllerRef = useRef<AbortController | null>(null);

  const execute = useCallback(async (
    url: string,
    options: UseFetchOptions<T> = {}
  ): Promise<T | null> => {
    const { onSuccess, onError, enabled = true, ...fetchOptions } = options;
    
    if (!enabled) return null;
    
    // Cancel previous request
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
    }
    
    const controller = new AbortController();
    abortControllerRef.current = controller;
    
    setIsLoading(true);
    setError(null);
    
    try {
      const result = await apiFetch<T>(url, {
        ...fetchOptions,
        signal: controller.signal,
      });
      
      setData(result.data);
      onSuccess?.(result.data);
      return result.data;
    } catch (err) {
      if (err instanceof FetchError) {
        setError(err);
        onError?.(err);
      }
      return null;
    } finally {
      if (abortControllerRef.current === controller) {
        abortControllerRef.current = null;
      }
      setIsLoading(false);
    }
  }, []);

  const reset = useCallback(() => {
    setData(null);
    setError(null);
    setIsLoading(false);
    if (abortControllerRef.current) {
      abortControllerRef.current.abort();
      abortControllerRef.current = null;
    }
  }, []);

  return {
    data,
    error,
    isLoading,
    execute,
    reset,
  };
}

§ 4.2 REACT QUERY ERROR HANDLING

typescript
// lib/react-query/error-handling.ts
import { QueryClient, DefaultOptions } from '@tanstack/react-query';
import { FetchError } from '@/lib/fetch';
import { captureException } from '@sentry/nextjs';

// Global error handler for React Query
export function handleReactQueryError(error: unknown) {
  if (error instanceof FetchError) {
    // Don't log client errors (4xx) to Sentry
    if (error.status >= 500) {
      captureException(error, {
        tags: {
          react_query_error: true,
          error_status: error.status.toString(),
        },
        extra: {
          code: error.code,
          details: error.details,
          requestId: error.requestId,
        },
      });
    }
    
    // Handle specific error types
    switch (error.status) {
      case 401:
        // Redirect to login
        if (typeof window !== 'undefined') {
          window.location.href = `/auth/login?redirect=${encodeURIComponent(window.location.pathname)}`;
        }
        break;
      case 403:
        // Show forbidden message
        console.warn('Forbidden access:', error.message);
        break;
      case 404:
        // Handle not found
        console.warn('Resource not found:', error.message);
        break;
      case 429:
        // Rate limiting - could show a toast
        console.warn('Rate limited:', error.message);
        break;
    }
  } else if (error instanceof Error) {
    // Log unknown errors
    captureException(error, {
      tags: { react_query_error: true },
    });
  }
}

// Query client configuration
export function createQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: {
        retry: (failureCount, error) => {
          // Don't retry on client errors (4xx)
          if (error instanceof FetchError && error.status >= 400 && error.status < 500) {
            return false;
          }
          // Retry up to 3 times for server errors
          return failureCount < 3;
        },
        retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
        staleTime: 5 * 60 * 1000, // 5 minutes
        gcTime: 10 * 60 * 1000, // 10 minutes
        refetchOnWindowFocus: false,
        refetchOnReconnect: true,
        refetchOnMount: true,
        onError: handleReactQueryError,
      },
      mutations: {
        retry: false,
        onError: handleReactQueryError,
      },
    },
  });
}

// Hook for query error handling
import { useQuery, UseQueryOptions, useMutation, UseMutationOptions } from '@tanstack/react-query';
import { api, FetchResponse } from '@/lib/fetch';

export function useApiQuery<TData = any, TError = FetchError>(
  key: string[],
  url: string,
  options?: UseQueryOptions<FetchResponse<TData>, TError>
) {
  return useQuery<FetchResponse<TData>, TError>({
    queryKey: key,
    queryFn: () => api.get<TData>(url),
    ...options,
  });
}

export function useApiMutation<TData = any, TVariables = any, TError = FetchError>(
  method: 'POST' | 'PUT' | 'PATCH' | 'DELETE',
  url: string | ((variables: TVariables) => string),
  options?: UseMutationOptions<FetchResponse<TData>, TError, TVariables>
) {
  return useMutation<FetchResponse<TData>, TError, TVariables>({
    mutationFn: async (variables) => {
      const endpoint = typeof url === 'function' ? url(variables) : url;
      
      switch (method) {
        case 'POST':
          return api.post<TData>(endpoint, variables);
        case 'PUT':
          return api.put<TData>(endpoint, variables);
        case 'PATCH':
          return api.patch<TData>(endpoint, variables);
        case 'DELETE':
          return api.delete<TData>(endpoint);
        default:
          throw new Error(`Unsupported method: ${method}`);
      }
    },
    ...options,
  });
}

// Error boundary for React Query errors
import { useQueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from '@/components/ErrorBoundary';

export function QueryErrorBoundary({ children }: { children: React.ReactNode }) {
  const { reset } = useQueryErrorResetBoundary();

  return (
    <ErrorBoundary
      onError={(error) => {
        console.error('Query error caught:', error);
      }}
      reset={reset}
    >
      {children}
    </ErrorBoundary>
  );
}

§ 4.3 FORM ERROR HANDLING

typescript
// lib/form/error-handling.ts
import { FetchError } from '@/lib/fetch';
import { ZodError } from 'zod';
import { FieldError, FieldValues } from 'react-hook-form';

export interface FormError {
  field?: string;
  message: string;
  code?: string;
}

export function parseFormError(error: unknown): FormError[] {
  if (error instanceof FetchError && error.details) {
    // Convert FetchError details to FormError array
    const errors: FormError[] = [];
    Object.entries(error.details).forEach(([field, messages]) => {
      messages.forEach(message => {
        errors.push({
          field,
          message,
          code: error.code,
        });
      });
    });
    return errors;
  }
  
  if (error instanceof ZodError) {
    // Convert ZodError to FormError array
    return error.errors.map(zodError => ({
      field: zodError.path.join('.'),
      message: zodError.message,
      code: 'VALIDATION_ERROR',
    }));
  }
  
  if (error instanceof Error) {
    return [{
      message: error.message,
      code: error.name,
    }];
  }
  
  return [{
    message: 'An unknown error occurred',
    code: 'UNKNOWN_ERROR',
  }];
}

export function mapFormErrors<T extends FieldValues>(
  errors: FormError[]
): Record<string, FieldError> {
  const result: Record<string, FieldError> = {};
  
  errors.forEach(error => {
    if (error.field) {
      result[error.field] = {
        type: 'server',
        message: error.message,
      };
    }
  });
  
  return result;
}

// Hook for form error handling
import { useState, useCallback } from 'react';
import { useForm, UseFormReturn } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

export interface UseFormErrorHandlingProps<T extends FieldValues> {
  schema?: z.ZodType<T>;
  defaultValues?: Partial<T>;
  onServerError?: (errors: FormError[]) => void;
}

export function useFormErrorHandling<T extends FieldValues>({
  schema,
  defaultValues,
  onServerError,
}: UseFormErrorHandlingProps<T> = {}) {
  const [formError, setFormError] = useState<FormError | null>(null);
  const [fieldErrors, setFieldErrors] = useState<Record<string, string>>({});

  const formMethods = useForm<T>({
    resolver: schema ? zodResolver(schema) : undefined,
    defaultValues,
  });

  const handleServerError = useCallback((error: unknown) => {
    const errors = parseFormError(error);
    
    // Set form-level error if no field errors
    const fieldErrors = errors.filter(e => e.field);
    const formErrors = errors.filter(e => !e.field);
    
    if (fieldErrors.length > 0) {
      const mappedErrors: Record<string, string> = {};
      fieldErrors.forEach(error => {
        if (error.field) {
          mappedErrors[error.field] = error.message;
          formMethods.setError(error.field, {
            type: 'server',
            message: error.message,
          });
        }
      });
      setFieldErrors(mappedErrors);
    }
    
    if (formErrors.length > 0) {
      setFormError(formErrors[0]);
    }
    
    onServerError?.(errors);
  }, [formMethods, onServerError]);

  const resetErrors = useCallback(() => {
    setFormError(null);
    setFieldErrors({});
    formMethods.clearErrors();
  }, [formMethods]);

  return {
    ...formMethods,
    formError,
    fieldErrors,
    handleServerError,
    resetErrors,
  };
}

// Form error display component
// components/form/FormErrorDisplay.tsx
import { AlertCircle, XCircle } from 'lucide-react';
import { cn } from '@/lib/utils';

interface FormErrorDisplayProps {
  error?: {
    message?: string;
    code?: string;
  };
  className?: string;
  variant?: 'form' | 'field';
}

export function FormErrorDisplay({ error, className, variant = 'form' }: FormErrorDisplayProps) {
  if (!error?.message) return null;

  if (variant === 'field') {
    return (
      <div className={cn('flex items-center gap-1 mt-1 text-destructive text-sm', className)}>
        <XCircle className="h-3 w-3 flex-shrink-0" />
        <span>{error.message}</span>
      </div>
    );
  }

  return (
    <div className={cn(
      'flex items-start gap-2 p-3 rounded-lg border bg-destructive/10 border-destructive/20',
      className
    )}>
      <AlertCircle className="h-4 w-4 mt-0.5 flex-shrink-0 text-destructive" />
      <div className="flex-1">
        <p className="text-sm font-medium text-destructive">{error.message}</p>
        {error.code && (
          <p className="text-xs text-destructive/70 mt-1">Error code: {error.code}</p>
        )}
      </div>
    </div>
  );
}

§ §5. SENTRY INTEGRATION

§ 5.1 SETUP

typescript
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NEXT_PUBLIC_APP_ENV || "development",
  release: process.env.NEXT_PUBLIC_APP_VERSION || "1.0.0",
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === "production" ? 0.1 : 1.0,
  
  // Session replay
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
  
  // Filter out common errors
  ignoreErrors: [
    // Browser extensions
    "Extension context invalidated",
    "ResizeObserver loop limit exceeded",
    // Network errors
    "Failed to fetch",
    "NetworkError when attempting to fetch resource",
  ],
  
  beforeSend(event, hint) {
    const error = hint?.originalException;
    
    // Filter out development errors in production
    if (process.env.NODE_ENV !== "production") {
      return event;
    }
    
    // Don't send certain types of errors
    if (error instanceof Error) {
      // Don't send authentication errors
      if (error.message.includes("Authentication")) {
        return null;
      }
      
      // Don't send validation errors
      if (error.message.includes("Validation")) {
        return null;
      }
    }
    
    return event;
  },
  
  integrations: [
    Sentry.replayIntegration({
      maskAllText: true,
      blockAllMedia: true,
    }),
    Sentry.browserTracingIntegration(),
  ],
});

typescript
// sentry.server.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.APP_ENV || "development",
  release: process.env.APP_VERSION || "1.0.0",
  
  // Performance monitoring
  tracesSampleRate: process.env.NODE_ENV === "production" ? 0.1 : 1.0,
  
  // Filter out health check and monitoring requests
  beforeSendTransaction(event) {
    // Don't send transactions for health checks
    if (event.request?.url?.includes("/health")) {
      return null;
    }
    
    return event;
  },
});

typescript
// sentry.edge.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.APP_ENV || "development",
  release: process.env.APP_VERSION || "1.0.0",
  
  tracesSampleRate: 0.1,
});

§ 5.2 ERROR CAPTURING

typescript
// lib/sentry/client.ts
import * as Sentry from '@sentry/nextjs';
import { FetchError } from '@/lib/fetch';
import { AppError } from '@/lib/errors';

export function captureAppError(error: AppError | FetchError, context?: Record<string, any>) {
  Sentry.captureException(error, {
    level: 'error',
    tags: {
      error_code: error.code || error.status?.toString() || 'unknown',
      error_type: error.constructor.name,
    },
    extra: {
      ...context,
      message: error.message,
      details: 'details' in error ? error.details : undefined,
      requestId: 'requestId' in error ? error.requestId : undefined,
    },
  });
}

export function captureMessage(message: string, level: Sentry.SeverityLevel = 'info', context?: Record<string, any>) {
  Sentry.captureMessage(message, {
    level,
    extra: context,
  });
}

export function setUserContext(user: { id: string; email?: string; name?: string }) {
  Sentry.setUser({
    id: user.id,
    email: user.email,
    username: user.name,
  });
}

export function clearUserContext() {
  Sentry.setUser(null);
}

export function setTag(key: string, value: string) {
  Sentry.setTag(key, value);
}

export function addBreadcrumb(crumb: Sentry.Breadcrumb) {
  Sentry.addBreadcrumb(crumb);
}

export function withSentryContext<T extends (...args: any[]) => any>(
  fn: T,
  context: Record<string, any>
): T {
  return ((...args: Parameters<T>) => {
    Sentry.withScope((scope) => {
      Object.entries(context).forEach(([key, value]) => {
        scope.setExtra(key, value);
      });
      return fn(...args);
    });
  }) as T;
}

§ 5.3 CUSTOM CONTEXT

typescript
// lib/sentry/context.ts
import * as Sentry from '@sentry/nextjs';

export interface ErrorContext {
  userId?: string;
  organizationId?: string;
  featureFlags?: Record<string, boolean>;
  requestId?: string;
  url?: string;
  userAgent?: string;
  locale?: string;
  timezone?: string;
  screenResolution?: string;
}

export function enrichErrorContext(context: ErrorContext) {
  // Set user if available
  if (context.userId) {
    Sentry.setUser({
      id: context.userId,
      // You might want to fetch more user details here
    });
  }
  
  // Set tags
  if (context.organizationId) {
    Sentry.setTag('organization_id', context.organizationId);
  }
  
  if (context.locale) {
    Sentry.setTag('locale', context.locale);
  }
  
  if (context.timezone) {
    Sentry.setTag('timezone', context.timezone);
  }
  
  // Set extra context
  if (context.featureFlags) {
    Sentry.setExtra('feature_flags', context.featureFlags);
  }
  
  if (context.requestId) {
    Sentry.setExtra('request_id', context.requestId);
  }
  
  if (context.url) {
    Sentry.setExtra('url', context.url);
  }
  
  if (context.userAgent) {
    Sentry.setExtra('user_agent', context.userAgent);
  }
  
  if (context.screenResolution) {
    Sentry.setExtra('screen_resolution', context.screenResolution);
  }
}

// Hook for React components
import { useEffect } from 'react';
import { usePathname, useSearchParams } from 'next/navigation';

export function useSentryContext(context: Partial<ErrorContext> = {}) {
  const pathname = usePathname();
  const searchParams = useSearchParams();
  
  useEffect(() => {
    const url = `${pathname}${searchParams.toString() ? `?${searchParams.toString()}` : ''}`;
    
    enrichErrorContext({
      url,
      ...context,
    });
    
    // Clean up on unmount
    return () => {
      if (context.userId) {
        Sentry.setUser(null);
      }
    };
  }, [pathname, searchParams, context]);
}

// Middleware for adding context to API routes
import { NextRequest, NextResponse } from 'next/server';

export function withSentryContextMiddleware(
  handler: (req: NextRequest, context: ErrorContext) => Promise<NextResponse>
) {
  return async (req: NextRequest) => {
    const requestId = crypto.randomUUID();
    const url = req.url;
    const userAgent = req.headers.get('user-agent') || undefined;
    
    const context: ErrorContext = {
      requestId,
      url,
      userAgent,
      // Add more context from request if needed
    };
    
    try {
      // Enrich Sentry context
      enrichErrorContext(context);
      
      // Add request ID to response headers
      const response = await handler(req, context);
      response.headers.set('X-Request-ID', requestId);
      return response;
    } catch (error) {
      // Ensure error is captured with context
      Sentry.captureException(error);
      throw error;
    }
  };
}

§ §6. USER-FRIENDLY ERROR MESSAGES

§ 6.1 MESSAGE MAPPING

typescript
// lib/errors/messages.ts
import { ErrorCode } from '@/lib/errors';

export interface ErrorMessage {
  title: string;
  description: string;
  action?: string;
  icon?: string;
}

export const ERROR_MESSAGES: Record<ErrorCode, ErrorMessage> = {
  VALIDATION_ERROR: {
    title: 'Validation Error',
    description: 'Please check your input and try again.',
    action: 'Fix the errors above',
  },
  AUTHENTICATION_ERROR: {
    title: 'Authentication Required',
    description: 'Please sign in to access this resource.',
    action: 'Sign in',
  },
  AUTHORIZATION_ERROR: {
    title: 'Access Denied',
    description: 'You don\'t have permission to access this resource.',
    action: 'Request access',
  },
  NOT_FOUND_ERROR: {
    title: 'Not Found',
    description: 'The requested resource could not be found.',
    action: 'Go back',
  },
  CONFLICT_ERROR: {
    title: 'Conflict',
    description: 'The resource has been modified by another user.',
    action: 'Refresh and try again',
  },
  RATE_LIMIT_ERROR: {
    title: 'Too Many Requests',
    description: 'You\'ve made too many requests. Please try again later.',
    action: 'Try again in a few minutes',
  },
  EXTERNAL_SERVICE_ERROR: {
    title: 'Service Unavailable',
    description: 'An external service is temporarily unavailable.',
    action: 'Try again later',
  },
  INTERNAL_SERVER_ERROR: {
    title: 'Internal Server Error',
    description: 'Something went wrong on our end. Our team has been notified.',
    action: 'Try again',
  },
  BAD_REQUEST_ERROR: {
    title: 'Bad Request',
    description: 'The request could not be understood.',
    action: 'Check your request',
  },
};

// Function to get user-friendly error message
export function getUserErrorMessage(
  code: ErrorCode,
  defaultMessage?: string
): ErrorMessage {
  const message = ERROR_MESSAGES[code];
  
  if (!message) {
    return {
      title: 'Error',
      description: defaultMessage || 'An unexpected error occurred.',
      action: 'Try again',
    };
  }
  
  return message;
}

// Function to translate technical errors to user-friendly messages
export function translateError(error: unknown): ErrorMessage {
  if (typeof error === 'string') {
    return {
      title: 'Error',
      description: error,
      action: 'Try again',
    };
  }
  
  if (error instanceof Error) {
    // Check if it's an AppError
    if ('code' in error && typeof error.code === 'string') {
      const code = error.code as ErrorCode;
      return getUserErrorMessage(code, error.message);
    }
    
    // Generic error
    return {
      title: 'Error',
      description: error.message,
      action: 'Try again',
    };
  }
  
  // Unknown error
  return {
    title: 'Unexpected Error',
    description: 'An unexpected error occurred. Please try again.',
    action: 'Try again',
  };
}

§ 6.2 ERROR UI COMPONENTS

typescript
// components/errors/ErrorCard.tsx
import { AlertCircle, RefreshCw, Home, ArrowLeft } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { cn } from '@/lib/utils';
import { translateError } from '@/lib/errors/messages';

interface ErrorCardProps {
  error?: unknown;
  title?: string;
  description?: string;
  action?: string;
  onRetry?: () => void;
  onHome?: () => void;
  onBack?: () => void;
  className?: string;
  variant?: 'default' | 'destructive' | 'warning';
}

export function ErrorCard({
  error,
  title,
  description,
  action,
  onRetry,
  onHome,
  onBack,
  className,
  variant = 'default',
}: ErrorCardProps) {
  const translatedError = error ? translateError(error) : null;
  
  const displayTitle = title || translatedError?.title || 'Error';
  const displayDescription = description || translatedError?.description || 'An error occurred';
  const displayAction = action || translatedError?.action;
  
  const variantStyles = {
    default: 'border-border bg-card',
    destructive: 'border-destructive/20 bg-destructive/10',
    warning: 'border-amber-200 bg-amber-50 dark:border-amber-800 dark:bg-amber-900/20',
  };
  
  const iconColors = {
    default: 'text-muted-foreground',
    destructive: 'text-destructive',
    warning: 'text-amber-600 dark:text-amber-400',
  };
  
  return (
    <Card className={cn(variantStyles[variant], className)}>
      <CardHeader>
        <div className="flex items-center gap-3">
          <AlertCircle className={cn('h-6 w-6', iconColors[variant])} />
          <CardTitle className="text-lg">{displayTitle}</CardTitle>
        </div>
      </CardHeader>
      
      <CardContent>
        <p className="text-muted-foreground">{displayDescription}</p>
      </CardContent>
      
      {(onRetry || onHome || onBack) && (
        <CardFooter className="flex flex-wrap gap-2">
          {onRetry && (
            <Button
              onClick={onRetry}
              variant={variant === 'destructive' ? 'destructive' : 'default'}
              size="sm"
              className="gap-2"
            >
              <RefreshCw className="h-4 w-4" />
              {displayAction || 'Try again'}
            </Button>
          )}
          
          {onBack && (
            <Button
              onClick={onBack}
              variant="outline"
              size="sm"
              className="gap-2"
            >
              <ArrowLeft className="h-4 w-4" />
              Go back
            </Button>
          )}
          
          {onHome && (
            <Button
              onClick={onHome}
              variant="ghost"
              size="sm"
              className="gap-2"
            >
              <Home className="h-4 w-4" />
              Go home
            </Button>
          )}
        </CardFooter>
      )}
    </Card>
  );
}

typescript
// components/errors/ErrorToast.tsx
'use client';

import { useEffect, useState } from 'react';
import { X, AlertCircle, CheckCircle, Info, XCircle } from 'lucide-react';
import { cn } from '@/lib/utils';
import { translateError } from '@/lib/errors/messages';

interface ErrorToastProps {
  error: unknown;
  type?: 'error' | 'success' | 'info' | 'warning';
  duration?: number;
  onDismiss?: () => void;
}

export function ErrorToast({
  error,
  type = 'error',
  duration = 5000,
  onDismiss,
}: ErrorToastProps) {
  const [isVisible, setIsVisible] = useState(true);
  
  const translatedError = translateError(error);
  
  const typeConfig = {
    error: {
      icon: XCircle,
      bg: 'bg-destructive',
      border: 'border-destructive',
      text: 'text-destructive-foreground',
    },
    success: {
      icon: CheckCircle,
      bg: 'bg-green-500',
      border: 'border-green-500',
      text: 'text-green-foreground',
    },
    warning: {
      icon: AlertCircle,
      bg: 'bg-amber-500',
      border: 'border-amber-500',
      text: 'text-amber-foreground',
    },
    info: {
      icon: Info,
      bg: 'bg-blue-500',
      border: 'border-blue-500',
      text: 'text-blue-foreground',
    },
  };
  
  const config = typeConfig[type];
  const Icon = config.icon;
  
  useEffect(() => {
    if (duration > 0) {
      const timer = setTimeout(() => {
        setIsVisible(false);
        onDismiss?.();
      }, duration);
      
      return () => clearTimeout(timer);
    }
  }, [duration, onDismiss]);
  
  if (!isVisible) return null;
  
  return (
    <div className={cn(
      'fixed top-4 right-4 z-50 max-w-sm',
      'animate-in slide-in-from-right-5'
    )}>
      <div className={cn(
        'rounded-lg border shadow-lg p-4',
        config.bg,
        config.border,
        config.text
      )}>
        <div className="flex items-start gap-3">
          <Icon className="h-5 w-5 flex-shrink-0 mt-0.5" />
          <div className="flex-1">
            <h4 className="font-semibold">{translatedError.title}</h4>
            <p className="text-sm opacity-90 mt-1">{translatedError.description}</p>
          </div>
          <button
            onClick={() => {
              setIsVisible(false);
              onDismiss?.();
            }}
            className="ml-4 flex-shrink-0 rounded-md opacity-70 hover:opacity-100 focus:outline-none"
          >
            <X className="h-4 w-4" />
          </button>
        </div>
      </div>
    </div>
  );
}

// Toast provider for global error notifications
import { createContext, useContext, useCallback, ReactNode } from 'react';

interface Toast {
  id: string;
  error: unknown;
  type: 'error' | 'success' | 'info' | 'warning';
}

interface ToastContextType {
  showToast: (error: unknown, type?: Toast['type']) => void;
  dismissToast: (id: string) => void;
}

const ToastContext = createContext<ToastContextType | undefined>(undefined);

export function ToastProvider({ children }: { children: ReactNode }) {
  const [toasts, setToasts] = useState<Toast[]>([]);
  
  const showToast = useCallback((error: unknown, type: Toast['type'] = 'error') => {
    const id = crypto.randomUUID();
    setToasts(prev => [...prev, { id, error, type }]);
    
    // Auto-dismiss after 5 seconds
    setTimeout(() => {
      setToasts(prev => prev.filter(toast => toast.id !== id));
    }, 5000);
  }, []);
  
  const dismissToast = useCallback((id: string) => {
    setToasts(prev => prev.filter(toast => toast.id !== id));
  }, []);
  
  return (
    <ToastContext.Provider value={{ showToast, dismissToast }}>
      {children}
      <div className="fixed top-4 right-4 z-50 space-y-2">
        {toasts.map(toast => (
          <ErrorToast
            key={toast.id}
            error={toast.error}
            type={toast.type}
            onDismiss={() => dismissToast(toast.id)}
          />
        ))}
      </div>
    </ToastContext.Provider>
  );
}

export function useToast() {
  const context = useContext(ToastContext);
  if (!context) {
    throw new Error('useToast must be used within ToastProvider');
  }
  return context;
}

§ §7. NOT FOUND HANDLING

§ 7.1 NEXT.JS NOT-FOUND

typescript
// app/not-found.tsx
import Link from 'next/link';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { Home, Search, ArrowLeft } from 'lucide-react';

export default function NotFound() {
  return (
    <div className="min-h-screen flex items-center justify-center bg-background p-4">
      <Card className="w-full max-w-md">
        <CardHeader className="text-center">
          <div className="mx-auto mb-4 flex h-16 w-16 items-center justify-center rounded-full bg-muted">
            <Search className="h-8 w-8 text-muted-foreground" />
          </div>
          <CardTitle className="text-3xl">404 - Page Not Found</CardTitle>
        </CardHeader>
        
        <CardContent className="text-center space-y-4">
          <p className="text-muted-foreground">
            The page you're looking for doesn't exist or has been moved.
          </p>
          
          <div className="bg-muted rounded-lg p-4">
            <p className="text-sm">
              <strong>Possible reasons:</strong>
            </p>
            <ul className="text-sm text-muted-foreground mt-2 space-y-1 text-left">
              <li>• The URL might be mistyped</li>
              <li>• The page might have been deleted</li>
              <li>• You might not have access to this page</li>
            </ul>
          </div>
        </CardContent>
        
        <CardFooter className="flex flex-col sm:flex-row gap-3">
          <Button asChild className="flex-1 gap-2">
            <Link href="/">
              <Home className="h-4 w-4" />
              Go home
            </Link>
          </Button>
          
          <Button variant="outline" asChild className="flex-1 gap-2">
            <Link href="#" onClick={() => window.history.back()}>
              <ArrowLeft className="h-4 w-4" />
              Go back
            </Link>
          </Button>
        </CardFooter>
      </Card>
    </div>
  );
}

// Usage in components or server actions
import { notFound } from 'next/navigation';

// In a server component
export async function UserPage({ params }: { params: { userId: string } }) {
  const user = await getUser(params.userId);
  
  if (!user) {
    notFound();
  }
  
  return <UserProfile user={user} />;
}

// In a server action
export async function getUserAction(userId: string) {
  const user = await getUser(userId);
  
  if (!user) {
    throw new Error('User not found');
    // This will be caught by the error boundary
  }
  
  return user;
}

§ §8. OFFLINE ERROR HANDLING

typescript
// lib/offline/error-handling.ts
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(
    typeof navigator !== 'undefined' ? navigator.onLine : true
  );

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

export class OfflineError extends Error {
  constructor(message = 'You are offline. Please check your connection.') {
    super(message);
    this.name = 'OfflineError';
  }
}

export class RequestQueue {
  private queue: Array<{
    id: string;
    request: () => Promise<any>;
    resolve: (value: any) => void;
    reject: (error: any) => void;
    retries: number;
    maxRetries: number;
  }> = [];
  
  private isProcessing = false;
  private maxQueueSize = 100;

  async add<T>(
    request: () => Promise<T>,
    maxRetries = 3
  ): Promise<T> {
    return new Promise((resolve, reject) => {
      if (this.queue.length >= this.maxQueueSize) {
        reject(new Error('Request queue is full'));
        return;
      }

      const id = crypto.randomUUID();
      this.queue.push({
        id,
        request,
        resolve,
        reject,
        retries: 0,
        maxRetries,
      });

      this.processQueue();
    });
  }

  private async processQueue() {
    if (this.isProcessing || this.queue.length === 0) {
      return;
    }

    this.isProcessing = true;

    while (this.queue.length > 0) {
      const item = this.queue[0];
      
      try {
        const result = await item.request();
        item.resolve(result);
        this.queue.shift();
      } catch (error) {
        item.retries++;
        
        if (item.retries >= item.maxRetries) {
          item.reject(error);
          this.queue.shift();
        } else {
          // Move to end of queue for retry
          this.queue.push(this.queue.shift()!);
          await new Promise(resolve => setTimeout(resolve, 1000 * item.retries));
        }
      }
    }

    this.isProcessing = false;
  }

  clear() {
    this.queue.forEach(item => {
      item.reject(new Error('Queue cleared'));
    });
    this.queue = [];
  }

  get size() {
    return this.queue.length;
  }
}

// Hook for offline-aware fetching
import { useCallback, useRef } from 'react';
import { RequestQueue } from './request-queue';

export function useOfflineFetch() {
  const queueRef = useRef(new RequestQueue());
  const isOnline = useOnlineStatus();

  const fetchWithOfflineSupport = useCallback(
    async <T>(
      input: RequestInfo,
      init?: RequestInit,
      options?: {
        queueWhenOffline?: boolean;
        maxRetries?: number;
      }
    ): Promise<T> => {
      const { queueWhenOffline = true, maxRetries = 3 } = options || {};

      if (!isOnline && queueWhenOffline) {
        // Queue the request when offline
        return queueRef.current.add(
          () => fetch(input, init).then(res => res.json()),
          maxRetries
        );
      }

      if (!isOnline) {
        throw new OfflineError();
      }

      // Execute immediately when online
      const response = await fetch(input, init);
      if (!response.ok) {
        throw new Error(`HTTP error ${response.status}`);
      }
      return response.json();
    },
    [isOnline]
  );

  const retryQueuedRequests = useCallback(() => {
    // The queue will automatically process when we come back online
    console.log(`Retrying ${queueRef.current.size} queued requests`);
  }, []);

  return {
    fetch: fetchWithOfflineSupport,
    isOnline,
    queueSize: queueRef.current.size,
    retryQueuedRequests,
    clearQueue: () => queueRef.current.clear(),
  };
}

// Offline indicator component
// components/OfflineIndicator.tsx
'use client';

import { useEffect, useState } from 'react';
import { Wifi, WifiOff } from 'lucide-react';
import { cn } from '@/lib/utils';

export function OfflineIndicator() {
  const [isVisible, setIsVisible] = useState(false);
  const isOnline = useOnlineStatus();

  useEffect(() => {
    if (!isOnline) {
      setIsVisible(true);
    } else {
      // Hide with delay when coming back online
      const timer = setTimeout(() => setIsVisible(false), 3000);
      return () => clearTimeout(timer);
    }
  }, [isOnline]);

  if (!isVisible) return null;

  return (
    <div
      className={cn(
        'fixed bottom-4 left-4 right-4 sm:left-auto sm:right-4 sm:w-64',
        'p-4 rounded-lg shadow-lg z-50',
        'animate-in slide-in-from-bottom-5',
        isOnline
          ? 'bg-green-50 border border-green-200 text-green-800 dark:bg-green-900/20 dark:border-green-800 dark:text-green-200'
          : 'bg-amber-50 border border-amber-200 text-amber-800 dark:bg-amber-900/20 dark:border-amber-800 dark:text-amber-200'
      )}
    >
      <div className="flex items-center gap-3">
        {isOnline ? (
          <Wifi className="h-5 w-5" />
        ) : (
          <WifiOff className="h-5 w-5" />
        )}
        <div className="flex-1">
          <p className="font-medium">
            {isOnline ? 'You are back online' : 'You are offline'}
          </p>
          <p className="text-sm opacity-90 mt-1">
            {isOnline
              ? 'Your connection has been restored.'
              : 'Check your internet connection.'}
          </p>
        </div>
        <button
          onClick={() => setIsVisible(false)}
          className="text-sm font-medium hover:opacity-80"
        >
          Dismiss
        </button>
      </div>
    </div>
  );
}

§ §9. ERROR LOGGING & ALERTING

§ 9.1 LOGGING STRATEGY

| Environment | Log Level | Destination |
|-------------|-----------|-------------|
| Development | Debug | Console |
| Staging | Info | Console + Sentry |
| Production | Error | Sentry + Log service |

§ 9.2 ALERT THRESHOLDS

typescript
// lib/monitoring/alerts.ts
import * as Sentry from '@sentry/nextjs';

export interface AlertThreshold {
  errorCount: number;
  timeWindow: number; // minutes
  errorTypes?: string[];
  tags?: Record<string, string>;
}

export class AlertManager {
  private errorCounts = new Map<string, number>();
  private alertedIssues = new Set<string>();

  constructor(private thresholds: AlertThreshold[] = []) {
    // Set up periodic cleanup
    setInterval(() => this.cleanup(), 60000); // Clean up every minute
  }

  trackError(error: Error, tags?: Record<string, string>) {
    const errorKey = this.getErrorKey(error, tags);
    const count = (this.errorCounts.get(errorKey) || 0) + 1;
    this.errorCounts.set(errorKey, count);

    this.checkThresholds(error, tags, count);
  }

  private getErrorKey(error: Error, tags?: Record<string, string>): string {
    const parts = [error.name, error.message];
    if (tags) {
      parts.push(JSON.stringify(tags));
    }
    return parts.join('|');
  }

  private checkThresholds(error: Error, tags?: Record<string, string>, currentCount?: number) {
    const errorKey = this.getErrorKey(error, tags);
    const count = currentCount || this.errorCounts.get(errorKey) || 0;

    for (const threshold of this.thresholds) {
      // Check if error matches threshold criteria
      if (this.matchesThreshold(error, tags, threshold)) {
        if (count >= threshold.errorCount && !this.alertedIssues.has(errorKey)) {
          this.sendAlert(error, tags, count, threshold);
          this.alertedIssues.add(errorKey);
        }
      }
    }
  }

  private matchesThreshold(error: Error, tags: Record<string, string> | undefined, threshold: AlertThreshold): boolean {
    // Check error types
    if (threshold.errorTypes && !threshold.errorTypes.includes(error.name)) {
      return false;
    }

    // Check tags
    if (threshold.tags) {
      for (const [key, value] of Object.entries(threshold.tags)) {
        if (tags?.[key] !== value) {
          return false;
        }
      }
    }

    return true;
  }

  private sendAlert(error: Error, tags: Record<string, string> | undefined, count: number, threshold: AlertThreshold) {
    const alertMessage = `Alert: ${error.name} occurred ${count} times in ${threshold.timeWindow} minutes`;
    
    // Send to Sentry as a message
    Sentry.captureMessage(alertMessage, {
      level: 'warning',
      tags: {
        alert_type: 'threshold',
        ...tags,
      },
      extra: {
        error_name: error.name,
        error_message: error.message,
        error_count: count,
        time_window: `${threshold.timeWindow} minutes`,
        threshold: threshold.errorCount,
      },
    });

    // Could also send to other alerting services (Slack, PagerDuty, etc.)
    console.warn(`ALERT: ${alertMessage}`);
  }

  private cleanup() {
    // In a real implementation, you would track timestamps and remove old counts
    // For simplicity, we'll just clear the counts periodically
    this.errorCounts.clear();
    this.alertedIssues.clear();
  }
}

// Default thresholds
const DEFAULT_THRESHOLDS: AlertThreshold[] = [
  {
    errorCount: 10,
    timeWindow: 5,
    errorTypes: ['InternalServerError', 'ExternalServiceError'],
    tags: { environment: 'production' },
  },
  {
    errorCount: 50,
    timeWindow: 1,
    errorTypes: ['RateLimitError'],
  },
  {
    errorCount: 100,
    timeWindow: 1,
    errorTypes: ['ValidationError'],
  },
];

export const alertManager = new AlertManager(DEFAULT_THRESHOLDS);

// Hook for tracking component errors
export function useErrorAlerting() {
  const trackError = useCallback((error: Error, context?: Record<string, string>) => {
    // Track for alerting
    alertManager.trackError(error, context);
    
    // Also send to Sentry
    Sentry.captureException(error, {
      tags: context,
    });
  }, []);

  return { trackError };
}

§ §10. ERROR HANDLING CHECKLIST

BOUNDARIES
✅ Global error.tsx
✅ Per-route error boundaries
✅ Component ErrorBoundary

API
✅ Custom error classes
✅ Consistent error format
✅ Error handler middleware
✅ Proper status codes

CLIENT
✅ Fetch wrapper with error handling
✅ React Query error config
✅ Form error display
✅ Toast notifications

MONITORING
✅ Sentry setup
✅ User context
✅ Breadcrumbs
✅ Custom tags

UI
✅ Error page designs
✅ Inline error states
✅ Retry buttons
✅ Helpful messages

TESTING
□ Error boundary tests
□ API error tests
□ UI error state tests

---

## ERROR-MONITORING-AND-REPORTING

### Panoramica
Sistema completo di error monitoring: Sentry integration, error boundary con recovery, user feedback collection, error grouping e alerting.

### Implementazione Completa

```typescript
// lib/monitoring/error-reporter.ts
import * as Sentry from "@sentry/nextjs";

// ============================================================
// ERROR REPORTER SERVICE
// ============================================================
interface ErrorContext {
  userId?: string;
  sessionId?: string;
  route?: string;
  action?: string;
  metadata?: Record<string, unknown>;
}

export class ErrorReporter {
  private static instance: ErrorReporter;

  static getInstance(): ErrorReporter {
    if (!this.instance) {
      this.instance = new ErrorReporter();
    }
    return this.instance;
  }

  // ----------------------------------------------------------
  // CAPTURE ERROR
  // ----------------------------------------------------------
  captureError(error: Error, context?: ErrorContext): string {
    const eventId = Sentry.captureException(error, {
      tags: {
        route: context?.route,
        action: context?.action,
      },
      user: context?.userId
        ? { id: context.userId }
        : undefined,
      extra: {
        ...context?.metadata,
        sessionId: context?.sessionId,
      },
    });

    if (process.env.NODE_ENV === "development") {
      console.error("[ErrorReporter]", error.message, context);
    }

    return eventId;
  }

  // ----------------------------------------------------------
  // CAPTURE MESSAGE (non-error warnings)
  // ----------------------------------------------------------
  captureMessage(message: string, level: "info" | "warning" | "error" = "warning", context?: ErrorContext): void {
    Sentry.captureMessage(message, {
      level,
      tags: { route: context?.route, action: context?.action },
      extra: context?.metadata,
    });
  }

  // ----------------------------------------------------------
  // ADD BREADCRUMB (for error context trail)
  // ----------------------------------------------------------
  addBreadcrumb(
    category: string,
    message: string,
    data?: Record<string, unknown>,
    level: "info" | "warning" | "error" = "info"
  ): void {
    Sentry.addBreadcrumb({
      category,
      message,
      data,
      level,
      timestamp: Date.now() / 1000,
    });
  }

  // ----------------------------------------------------------
  // SET USER CONTEXT
  // ----------------------------------------------------------
  setUser(user: { id: string; email?: string; plan?: string } | null): void {
    if (user) {
      Sentry.setUser({ id: user.id, email: user.email, plan: user.plan });
    } else {
      Sentry.setUser(null);
    }
  }

  // ----------------------------------------------------------
  // PERFORMANCE MONITORING
  // ----------------------------------------------------------
  startTransaction(name: string, op: string): ReturnType<typeof Sentry.startSpan> {
    return Sentry.startSpan({ name, op }, (span) => span);
  }

  // ----------------------------------------------------------
  // USER FEEDBACK
  // ----------------------------------------------------------
  async submitUserFeedback(eventId: string, feedback: {
    name: string;
    email: string;
    comments: string;
  }): Promise<void> {
    Sentry.captureUserFeedback({
      event_id: eventId,
      name: feedback.name,
      email: feedback.email,
      comments: feedback.comments,
    });
  }
}

export const errorReporter = ErrorReporter.getInstance();
```

```typescript
// components/error/error-boundary-with-feedback.tsx
"use client";

import { Component, ReactNode } from "react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle, CardDescription } from "@/components/ui/card";
import { AlertTriangle, RefreshCw, Send, Bug } from "lucide-react";
import { errorReporter } from "@/lib/monitoring/error-reporter";

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
  eventId: string | null;
  showFeedback: boolean;
  feedbackSent: boolean;
  feedbackName: string;
  feedbackEmail: string;
  feedbackComments: string;
}

export class ErrorBoundaryWithFeedback extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      eventId: null,
      showFeedback: false,
      feedbackSent: false,
      feedbackName: "",
      feedbackEmail: "",
      feedbackComments: "",
    };
  }

  static getDerivedStateFromError(error: Error): Partial<State> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo): void {
    const eventId = errorReporter.captureError(error, {
      metadata: { componentStack: errorInfo.componentStack },
    });
    this.setState({ eventId });
    this.props.onError?.(error, errorInfo);
  }

  handleReset = () => {
    this.setState({
      hasError: false,
      error: null,
      eventId: null,
      showFeedback: false,
      feedbackSent: false,
    });
  };

  handleSubmitFeedback = async () => {
    if (!this.state.eventId) return;

    await errorReporter.submitUserFeedback(this.state.eventId, {
      name: this.state.feedbackName,
      email: this.state.feedbackEmail,
      comments: this.state.feedbackComments,
    });

    this.setState({ feedbackSent: true });
  };

  render() {
    if (!this.state.hasError) return this.props.children;

    if (this.props.fallback) return this.props.fallback;

    return (
      <div className="flex min-h-[400px] items-center justify-center p-8">
        <Card className="max-w-md w-full">
          <CardHeader>
            <div className="flex items-center gap-2">
              <AlertTriangle className="h-5 w-5 text-destructive" />
              <CardTitle className="text-lg">Something went wrong</CardTitle>
            </div>
            <CardDescription>
              We've been notified and are working on a fix.
              {this.state.eventId && (
                <span className="block mt-1 font-mono text-xs">
                  Error ID: {this.state.eventId}
                </span>
              )}
            </CardDescription>
          </CardHeader>
          <CardContent className="space-y-4">
            <div className="flex gap-2">
              <Button onClick={this.handleReset} className="flex-1">
                <RefreshCw className="mr-2 h-4 w-4" />
                Try Again
              </Button>
              <Button
                variant="outline"
                onClick={() => this.setState({ showFeedback: true })}
                className="flex-1"
              >
                <Bug className="mr-2 h-4 w-4" />
                Report
              </Button>
            </div>

            {this.state.showFeedback && !this.state.feedbackSent && (
              <div className="space-y-3 border-t pt-4">
                <p className="text-sm font-medium">Help us fix this issue</p>
                <Input
                  placeholder="Your name"
                  value={this.state.feedbackName}
                  onChange={(e) => this.setState({ feedbackName: e.target.value })}
                />
                <Input
                  placeholder="Your email"
                  type="email"
                  value={this.state.feedbackEmail}
                  onChange={(e) => this.setState({ feedbackEmail: e.target.value })}
                />
                <Textarea
                  placeholder="What were you doing when this happened?"
                  value={this.state.feedbackComments}
                  onChange={(e) => this.setState({ feedbackComments: e.target.value })}
                />
                <Button
                  size="sm"
                  onClick={this.handleSubmitFeedback}
                  disabled={!this.state.feedbackComments}
                >
                  <Send className="mr-2 h-4 w-4" />
                  Send Feedback
                </Button>
              </div>
            )}

            {this.state.feedbackSent && (
              <p className="text-sm text-green-600 text-center">
                Thank you for your feedback!
              </p>
            )}
          </CardContent>
        </Card>
      </div>
    );
  }
}
```

### Errori Comuni da Evitare
- **Sentry in development**: Disabilita Sentry in dev o usa un DSN separato
- **PII nei breadcrumb**: Non loggare password, token o dati sensibili
- **Error boundary senza recovery**: Fornisci sempre un pulsante "Try Again"
- **Missing source maps**: Carica le source maps su Sentry per stack trace leggibili

### Checklist di Verifica
- [ ] Sentry e configurato con DSN, environment e release
- [ ] L'error boundary cattura errori di rendering React
- [ ] Il feedback form raccoglie contesto dall'utente
- [ ] I breadcrumb forniscono trail di azioni pre-errore
- [ ] Le source maps sono caricate per deobfuscation
- [ ] L'error grouping in Sentry e corretto (no duplicati)


---

## ERROR-LOGGING-STRUCTURED

### Panoramica
Sistema di logging strutturato con livelli, contesto arricchito, log rotation e integrazione con servizi di monitoring esterni.

### Implementazione Completa

```typescript
// lib/logging/structured-logger.ts

type LogLevel = "debug" | "info" | "warn" | "error" | "fatal";

interface LogEntry {
  timestamp: string;
  level: LogLevel;
  message: string;
  context: Record<string, unknown>;
  error?: {
    name: string;
    message: string;
    stack?: string;
    code?: string;
  };
  request?: {
    method: string;
    url: string;
    userAgent?: string;
    ip?: string;
    requestId?: string;
  };
  duration?: number;
  userId?: string;
  traceId?: string;
  spanId?: string;
  service: string;
  environment: string;
}

interface LoggerConfig {
  service: string;
  environment: string;
  minLevel: LogLevel;
  transports: LogTransport[];
}

interface LogTransport {
  name: string;
  minLevel: LogLevel;
  write(entry: LogEntry): void | Promise<void>;
}

const LOG_LEVELS: Record<LogLevel, number> = {
  debug: 0,
  info: 1,
  warn: 2,
  error: 3,
  fatal: 4,
};

class ConsoleTransport implements LogTransport {
  name = "console";
  minLevel: LogLevel = "debug";

  write(entry: LogEntry): void {
    const color = {
      debug: "\x1b[36m",
      info: "\x1b[32m",
      warn: "\x1b[33m",
      error: "\x1b[31m",
      fatal: "\x1b[35m",
    }[entry.level];
    const reset = "\x1b[0m";
    const prefix = `${color}[${entry.level.toUpperCase()}]${reset}`;
    const msg = `${prefix} ${entry.timestamp} [${entry.service}] ${entry.message}`;

    if (entry.level === "error" || entry.level === "fatal") {
      console.error(msg, entry.error ?? "", entry.context);
    } else if (entry.level === "warn") {
      console.warn(msg, entry.context);
    } else {
      console.log(msg, Object.keys(entry.context).length > 0 ? entry.context : "");
    }
  }
}

class JsonFileTransport implements LogTransport {
  name = "json-file";
  minLevel: LogLevel = "info";
  private buffer: string[] = [];
  private flushInterval: ReturnType<typeof setInterval>;

  constructor(private filePath: string, flushIntervalMs = 5000) {
    this.flushInterval = setInterval(() => this.flush(), flushIntervalMs);
  }

  write(entry: LogEntry): void {
    this.buffer.push(JSON.stringify(entry));
    if (this.buffer.length >= 100) {
      this.flush();
    }
  }

  private flush(): void {
    if (this.buffer.length === 0) return;
    const batch = this.buffer.splice(0);
    const content = batch.join("\n") + "\n";
    // In Node.js environment, append to file
    if (typeof globalThis.process !== "undefined") {
      const fs = require("fs");
      fs.appendFileSync(this.filePath, content);
    }
  }

  destroy(): void {
    clearInterval(this.flushInterval);
    this.flush();
  }
}

export class StructuredLogger {
  private config: LoggerConfig;
  private defaultContext: Record<string, unknown> = {};

  constructor(config: LoggerConfig) {
    this.config = config;
  }

  setDefaultContext(ctx: Record<string, unknown>): void {
    this.defaultContext = { ...this.defaultContext, ...ctx };
  }

  child(context: Record<string, unknown>): StructuredLogger {
    const child = new StructuredLogger(this.config);
    child.defaultContext = { ...this.defaultContext, ...context };
    return child;
  }

  debug(message: string, context: Record<string, unknown> = {}): void {
    this.log("debug", message, context);
  }

  info(message: string, context: Record<string, unknown> = {}): void {
    this.log("info", message, context);
  }

  warn(message: string, context: Record<string, unknown> = {}): void {
    this.log("warn", message, context);
  }

  error(message: string, error?: Error, context: Record<string, unknown> = {}): void {
    this.log("error", message, context, error);
  }

  fatal(message: string, error?: Error, context: Record<string, unknown> = {}): void {
    this.log("fatal", message, context, error);
  }

  private log(level: LogLevel, message: string, context: Record<string, unknown>, error?: Error): void {
    if (LOG_LEVELS[level] < LOG_LEVELS[this.config.minLevel]) return;

    const entry: LogEntry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      context: { ...this.defaultContext, ...context },
      service: this.config.service,
      environment: this.config.environment,
    };

    if (error) {
      entry.error = {
        name: error.name,
        message: error.message,
        stack: error.stack,
        code: (error as any).code,
      };
    }

    for (const transport of this.config.transports) {
      if (LOG_LEVELS[level] >= LOG_LEVELS[transport.minLevel]) {
        try {
          transport.write(entry);
        } catch (err) {
          console.error(`Failed to write to transport ${transport.name}:`, err);
        }
      }
    }
  }

  // Request logging middleware
  requestLogger() {
    return (req: any, res: any, next: () => void) => {
      const start = Date.now();
      const requestId = req.headers["x-request-id"] ?? crypto.randomUUID();
      const child = this.child({ requestId });

      child.info(`${req.method} ${req.url}`, {
        method: req.method,
        url: req.url,
        userAgent: req.headers["user-agent"],
      });

      const originalEnd = res.end;
      res.end = (...args: any[]) => {
        const duration = Date.now() - start;
        child.info(`${req.method} ${req.url} ${res.statusCode}`, {
          statusCode: res.statusCode,
          duration,
        });
        originalEnd.apply(res, args);
      };

      next();
    };
  }
}

// Factory
export function createLogger(service: string): StructuredLogger {
  return new StructuredLogger({
    service,
    environment: process.env.NODE_ENV ?? "development",
    minLevel: (process.env.LOG_LEVEL as LogLevel) ?? "info",
    transports: [
      new ConsoleTransport(),
      ...(process.env.NODE_ENV === "production"
        ? [new JsonFileTransport(`/var/log/${service}.log`)]
        : []),
    ],
  });
}
```

### Checklist di Verifica
- [ ] Ogni log entry ha timestamp ISO 8601, service name e environment
- [ ] Gli errori includono stack trace e error code
- [ ] Il logger supporta child loggers con contesto ereditato
- [ ] I transport hanno livelli minimi indipendenti
- [ ] Il file transport usa buffering per performance
