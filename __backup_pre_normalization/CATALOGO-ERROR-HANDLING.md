# CATALOGO-ERROR-HANDLING

Catalogo Error Handling per Next.js 14 App Router
§ ERROR BOUNDARIES COMPLETE
error.tsx per Route Segments
typescript
// app/dashboard/error.tsx
'use client'

import { useEffect } from 'react'
import { ErrorBoundaryProps } from '@/types/error'

export default function DashboardError({
  error,
  reset,
}: ErrorBoundaryProps) {
  useEffect(() => {
    // Log error to error reporting service
    console.error('Dashboard Error:', error)
  }, [error])

  return (
    <div className="dashboard-error-container">
      <h2>Dashboard Unavailable</h2>
      <p>We're having trouble loading your dashboard.</p>
      <button
        onClick={reset}
        className="retry-button"
      >
        Try Again
      </button>
      <div className="error-details">
        <details>
          <summary>Error Details</summary>
          <pre>{error.message}</pre>
        </details>
      </div>
    </div>
  )
}
global-error.tsx per Root
typescript
// app/global-error.tsx
'use client'

import { GlobalErrorProps } from '@/types/error'
import { logError } from '@/lib/error-logging'

export default function GlobalError({
  error,
  reset,
}: GlobalErrorProps) {
  useEffect(() => {
    // Critical error logging
    logError('GLOBAL_ERROR', error, {
      severity: 'critical',
      userAgent: navigator.userAgent,
      url: window.location.href,
    })
  }, [error])

  return (
    <html lang="it">
      <body>
        <main className="global-error">
          <div className="error-content">
            <h1>Something went wrong!</h1>
            <p>We apologize for the inconvenience.</p>
            
            <div className="error-actions">
              <button onClick={() => reset()}>
                Try Again
              </button>
              <button onClick={() => window.location.href = '/'}>
                Go to Homepage
              </button>
              <button onClick={() => window.location.reload()}>
                Refresh Page
              </button>
            </div>
            
            <div className="contact-support">
              <p>If the problem persists, please contact support.</p>
              <a href="mailto:support@example.com">
                support@example.com
              </a>
            </div>
          </div>
        </main>
      </body>
    </html>
  )
}
Reset Functionality
typescript
// lib/error-recovery.ts
export class ErrorRecovery {
  private static maxRetries = 3
  private static retryCounts = new Map<string, number>()

  static async resetWithRetry(
    resetFn: () => void,
    errorKey: string
  ): Promise<void> {
    const currentRetries = this.retryCounts.get(errorKey) || 0
    
    if (currentRetries >= this.maxRetries) {
      // Clear state and do hard reset
      this.retryCounts.delete(errorKey)
      window.location.reload()
      return
    }

    try {
      resetFn()
      this.retryCounts.set(errorKey, currentRetries + 1)
      
      // Wait to see if error persists
      await new Promise(resolve => setTimeout(resolve, 1000))
    } catch (retryError) {
      console.error('Reset failed:', retryError)
    }
  }

  static clearErrorState(errorKey: string): void {
    this.retryCounts.delete(errorKey)
    // Clear any related localStorage/sessionStorage
    const errorPattern = new RegExp(`^${errorKey}`)
    Object.keys(localStorage).forEach(key => {
      if (errorPattern.test(key)) {
        localStorage.removeItem(key)
      }
    })
  }
}

// Usage in error.tsx
'use client'
import { ErrorRecovery } from '@/lib/error-recovery'

export default function ComponentError({ error, reset }: ErrorBoundaryProps) {
  const handleReset = async () => {
    await ErrorRecovery.resetWithRetry(reset, 'component-error')
  }

  return (
    <button onClick={handleReset}>
      Reset Component
    </button>
  )
}
Error Recovery Patterns
typescript
// hooks/use-error-recovery.ts
import { useState, useCallback } from 'react'

interface ErrorRecoveryState<T> {
  error: Error | null
  data: T | null
  retryCount: number
  isRecovering: boolean
}

export function useErrorRecovery<T>(
  asyncFn: () => Promise<T>,
  maxRetries = 3
) {
  const [state, setState] = useState<ErrorRecoveryState<T>>({
    error: null,
    data: null,
    retryCount: 0,
    isRecovering: false,
  })

  const execute = useCallback(async () => {
    setState(prev => ({ ...prev, isRecovering: true }))
    
    for (let attempt = 0; attempt <= maxRetries; attempt++) {
      try {
        const result = await asyncFn()
        setState({
          error: null,
          data: result,
          retryCount: attempt,
          isRecovering: false,
        })
        return result
      } catch (err) {
        if (attempt === maxRetries) {
          setState(prev => ({
            ...prev,
            error: err as Error,
            retryCount: attempt,
            isRecovering: false,
          }))
          throw err
        }
        
        // Exponential backoff
        await new Promise(resolve => 
          setTimeout(resolve, 1000 * Math.pow(2, attempt))
        )
      }
    }
  }, [asyncFn, maxRetries])

  const reset = useCallback(() => {
    setState({
      error: null,
      data: null,
      retryCount: 0,
      isRecovering: false,
    })
  }, [])

  return { ...state, execute, reset }
}
Nested Error Boundaries
typescript
// components/nested-error-boundary.tsx
'use client'

import React, { Component, ErrorInfo, ReactNode } from 'react'
import { NestedErrorBoundaryProps, NestedErrorFallback } from '@/types/error'

export class NestedErrorBoundary extends Component<
  NestedErrorBoundaryProps,
  { hasError: boolean; error: Error | null }
> {
  constructor(props: NestedErrorBoundaryProps) {
    super(props)
    this.state = { hasError: false, error: null }
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Nested Error Boundary caught:', error, errorInfo)
    
    // Bubble up error if parent boundary should handle it
    if (this.props.bubbleUp && this.props.onBubbleUp) {
      this.props.onBubbleUp(error)
    }
  }

  handleReset = () => {
    this.setState({ hasError: false, error: null })
    this.props.onReset?.()
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ? (
        this.props.fallback({
          error: this.state.error!,
          reset: this.handleReset,
          boundaryId: this.props.boundaryId,
        })
      ) : (
        <DefaultNestedFallback
          error={this.state.error!}
          reset={this.handleReset}
        />
      )
    }

    return this.props.children
  }
}

// Parent component managing nested boundaries
export function ParentComponent() {
  const [errors, setErrors] = useState<Record<string, Error>>({})

  const handleBubbleUp = useCallback((boundaryId: string, error: Error) => {
    setErrors(prev => ({ ...prev, [boundaryId]: error }))
  }, [])

  return (
    <div>
      <NestedErrorBoundary
        boundaryId="sidebar"
        fallback={SidebarError}
        onBubbleUp={(error) => handleBubbleUp('sidebar', error)}
        bubbleUp={false}
      >
        <Sidebar />
      </NestedErrorBoundary>

      <NestedErrorBoundary
        boundaryId="main-content"
        fallback={MainContentError}
        onBubbleUp={(error) => handleBubbleUp('main', error)}
        bubbleUp={true}
      >
        <MainContent />
      </NestedErrorBoundary>

      {/* Global error summary */}
      {Object.keys(errors).length > 0 && (
        <GlobalErrorSummary errors={errors} />
      )}
    </div>
  )
}
§ API ERROR HANDLING
Standardized Error Response Format
typescript
// types/api-error.ts
export interface ApiErrorResponse {
  success: false
  error: {
    code: string
    message: string
    details?: Record<string, unknown>
    validationErrors?: ValidationError[]
    timestamp: string
    traceId?: string
    path?: string
  }
}

export interface ValidationError {
  field: string
  message: string
  code: string
}

export interface ApiSuccessResponse<T = unknown> {
  success: true
  data: T
  meta?: {
    page?: number
    limit?: number
    total?: number
    [key: string]: unknown
  }
}

export type ApiResponse<T = unknown> = 
  | ApiSuccessResponse<T>
  | ApiErrorResponse

// HTTP Status Code mapping
export enum HttpStatusCode {
  BAD_REQUEST = 400,
  UNAUTHORIZED = 401,
  FORBIDDEN = 403,
  NOT_FOUND = 404,
  CONFLICT = 409,
  UNPROCESSABLE_ENTITY = 422,
  TOO_MANY_REQUESTS = 429,
  INTERNAL_SERVER_ERROR = 500,
  BAD_GATEWAY = 502,
  SERVICE_UNAVAILABLE = 503,
  GATEWAY_TIMEOUT = 504,
}

// Error codes catalog
export enum ErrorCode {
  // Validation errors (1000-1999)
  VALIDATION_FAILED = 'VALIDATION_1000',
  REQUIRED_FIELD = 'VALIDATION_1001',
  INVALID_FORMAT = 'VALIDATION_1002',
  
  // Authentication errors (2000-2999)
  UNAUTHENTICATED = 'AUTH_2000',
  INVALID_CREDENTIALS = 'AUTH_2001',
  TOKEN_EXPIRED = 'AUTH_2002',
  INSUFFICIENT_PERMISSIONS = 'AUTH_2003',
  
  // Resource errors (3000-3999)
  NOT_FOUND = 'RESOURCE_3000',
  ALREADY_EXISTS = 'RESOURCE_3001',
  CONFLICT = 'RESOURCE_3002',
  
  // Business logic errors (4000-4999)
  BUSINESS_RULE_VIOLATION = 'BUSINESS_4000',
  INVALID_STATE = 'BUSINESS_4001',
  
  // System errors (5000-5999)
  INTERNAL_ERROR = 'SYSTEM_5000',
  SERVICE_UNAVAILABLE = 'SYSTEM_5001',
  DATABASE_ERROR = 'SYSTEM_5002',
  EXTERNAL_SERVICE_ERROR = 'SYSTEM_5003',
  
  // Rate limiting (6000-6999)
  RATE_LIMIT_EXCEEDED = 'RATE_6000',
}
Error Serialization
typescript
// lib/api-error.ts
export class ApiError extends Error {
  constructor(
    public code: ErrorCode,
    message: string,
    public statusCode: HttpStatusCode = HttpStatusCode.INTERNAL_SERVER_ERROR,
    public details?: Record<string, unknown>,
    public validationErrors?: ValidationError[]
  ) {
    super(message)
    this.name = 'ApiError'
    
    // Maintain proper stack trace
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ApiError)
    }
  }

  toResponse(path?: string): ApiErrorResponse {
    return {
      success: false,
      error: {
        code: this.code,
        message: this.message,
        details: this.details,
        validationErrors: this.validationErrors,
        timestamp: new Date().toISOString(),
        traceId: this.generateTraceId(),
        path,
      },
    }
  }

  private generateTraceId(): string {
    return `trace_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
  }

  // Factory methods for common errors
  static notFound(message: string, details?: Record<string, unknown>) {
    return new ApiError(
      ErrorCode.NOT_FOUND,
      message,
      HttpStatusCode.NOT_FOUND,
      details
    )
  }

  static validation(
    message: string,
    validationErrors: ValidationError[]
  ) {
    return new ApiError(
      ErrorCode.VALIDATION_FAILED,
      message,
      HttpStatusCode.UNPROCESSABLE_ENTITY,
      undefined,
      validationErrors
    )
  }

  static unauthorized(message = 'Authentication required') {
    return new ApiError(
      ErrorCode.UNAUTHENTICATED,
      message,
      HttpStatusCode.UNAUTHORIZED
    )
  }

  static forbidden(message = 'Insufficient permissions') {
    return new ApiError(
      ErrorCode.INSUFFICIENT_PERMISSIONS,
      message,
      HttpStatusCode.FORBIDDEN
    )
  }
}

// API Route handler wrapper
export async function withApiErrorHandler<T>(
  handler: () => Promise<T>,
  path?: string
): Promise<ApiResponse<T>> {
  try {
    const data = await handler()
    return {
      success: true,
      data,
    }
  } catch (error) {
    console.error('API Error:', error)
    
    if (error instanceof ApiError) {
      return error.toResponse(path)
    }
    
    // Handle unexpected errors
    const apiError = new ApiError(
      ErrorCode.INTERNAL_ERROR,
      'An unexpected error occurred',
      HttpStatusCode.INTERNAL_SERVER_ERROR,
      process.env.NODE_ENV === 'development' 
        ? { originalError: error instanceof Error ? error.message : String(error) }
        : undefined
    )
    
    return apiError.toResponse(path)
  }
}

// Usage in API routes
// app/api/users/[id]/route.ts
import { withApiErrorHandler, ApiError } from '@/lib/api-error'

export async function GET(
  request: Request,
  { params }: { params: { id: string } }
) {
  return withApiErrorHandler(async () => {
    const user = await getUserById(params.id)
    
    if (!user) {
      throw ApiError.notFound(`User with ID ${params.id} not found`)
    }
    
    if (!user.isActive) {
      throw ApiError.forbidden('User account is deactivated')
    }
    
    return user
  }, `/api/users/${params.id}`)
}
§ FORM ERRORS
Field-level and Form-level Error Handling
typescript
// types/form-error.ts
export interface FormFieldError {
  field: string
  message: string
  code: string
}

export interface FormError {
  message: string
  code: string
  fieldErrors?: FormFieldError[]
  timestamp: string
}

export interface FormState<T = unknown> {
  data?: T
  errors?: FormError
  isSubmitting: boolean
  isSuccess: boolean
  touched: Record<string, boolean>
}

// hooks/use-form-errors.ts
import { useState, useCallback } from 'react'

export function useFormErrors<T extends Record<string, any>>(
  initialData: T
) {
  const [state, setState] = useState<FormState<T>>({
    data: initialData,
    errors: undefined,
    isSubmitting: false,
    isSuccess: false,
    touched: {},
  })

  const setFieldError = useCallback((
    field: keyof T,
    message: string,
    code = 'VALIDATION_ERROR'
  ) => {
    setState(prev => ({
      ...prev,
      errors: {
        message: 'Form validation failed',
        code,
        fieldErrors: [
          ...(prev.errors?.fieldErrors || []).filter(e => e.field !== field),
          { field: field as string, message, code },
        ],
        timestamp: new Date().toISOString(),
      },
    }))
  }, [])

  const setFormError = useCallback((
    message: string,
    code = 'FORM_ERROR',
    fieldErrors?: FormFieldError[]
  ) => {
    setState(prev => ({
      ...prev,
      errors: {
        message,
        code,
        fieldErrors,
        timestamp: new Date().toISOString(),
      },
      isSubmitting: false,
    }))
  }, [])

  const clearErrors = useCallback(() => {
    setState(prev => ({
      ...prev,
      errors: undefined,
    }))
  }, [])

  const clearFieldError = useCallback((field: keyof T) => {
    setState(prev => ({
      ...prev,
      errors: prev.errors ? {
        ...prev.errors,
        fieldErrors: prev.errors.fieldErrors?.filter(e => e.field !== field),
      } : undefined,
    }))
  }, [])

  const setTouched = useCallback((field: keyof T, isTouched = true) => {
    setState(prev => ({
      ...prev,
      touched: {
        ...prev.touched,
        [field]: isTouched,
      },
    }))
  }, [])

  return {
    formState: state,
    setFieldError,
    setFormError,
    clearErrors,
    clearFieldError,
    setTouched,
    setState,
  }
}
Server Action Errors with useFormState
typescript
// actions/form-actions.ts
'use server'

import { ApiError } from '@/lib/api-error'
import { ZodError } from 'zod'
import { formSchema, FormData } from '@/schemas/form-schema'

export async function submitFormAction(
  prevState: FormState<FormData> | undefined,
  formData: FormData
): Promise<FormState<FormData>> {
  try {
    // Validate with Zod
    const validatedData = formSchema.parse(formData)
    
    // Business logic validation
    if (await isDuplicateEmail(validatedData.email)) {
      return {
        data: validatedData,
        errors: {
          message: 'Form submission failed',
          code: 'DUPLICATE_EMAIL',
          fieldErrors: [{
            field: 'email',
            message: 'This email is already registered',
            code: 'DUPLICATE',
          }],
          timestamp: new Date().toISOString(),
        },
        isSubmitting: false,
        isSuccess: false,
        touched: {},
      }
    }
    
    // Process form
    await saveFormData(validatedData)
    
    return {
      data: validatedData,
      errors: undefined,
      isSubmitting: false,
      isSuccess: true,
      touched: {},
    }
    
  } catch (error) {
    if (error instanceof ZodError) {
      const fieldErrors = error.errors.map(err => ({
        field: err.path.join('.'),
        message: err.message,
        code: err.code,
      }))
      
      return {
        data: formData,
        errors: {
          message: 'Validation failed',
          code: 'VALIDATION_ERROR',
          fieldErrors,
          timestamp: new Date().toISOString(),
        },
        isSubmitting: false,
        isSuccess: false,
        touched: {},
      }
    }
    
    if (error instanceof ApiError) {
      return {
        data: formData,
        errors: {
          message: error.message,
          code: error.code,
          timestamp: new Date().toISOString(),
        },
        isSubmitting: false,
        isSuccess: false,
        touched: {},
      }
    }
    
    // Unexpected error
    return {
      data: formData,
      errors: {
        message: 'An unexpected error occurred',
        code: 'INTERNAL_ERROR',
        timestamp: new Date().toISOString(),
      },
      isSubmitting: false,
      isSuccess: false,
      touched: {},
    }
  }
}

// Usage in component
// app/components/form-component.tsx
'use client'

import { useFormState } from 'react-dom'
import { submitFormAction } from '@/actions/form-actions'
import { FormErrorDisplay } from '@/components/form-error-display'

const initialState: FormState = {
  data: undefined,
  errors: undefined,
  isSubmitting: false,
  isSuccess: false,
  touched: {},
}

export function UserForm() {
  const [state, formAction] = useFormState(submitFormAction, initialState)

  return (
    <form action={formAction}>
      <div className="form-group">
        <label htmlFor="email">Email</label>
        <input
          type="email"
          id="email"
          name="email"
          aria-invalid={state.errors?.fieldErrors?.some(e => e.field === 'email')}
          aria-describedby={state.errors?.fieldErrors?.some(e => e.field === 'email') ? 'email-error' : undefined}
        />
        {state.errors?.fieldErrors?.map(error => 
          error.field === 'email' && (
            <div key="email-error" id="email-error" className="field-error">
              {error.message}
            </div>
          )
        )}
      </div>

      <FormErrorDisplay
        error={state.errors}
        showDetails={process.env.NODE_ENV === 'development'}
      />

      <button 
        type="submit"
        disabled={state.isSubmitting}
      >
        {state.isSubmitting ? 'Submitting...' : 'Submit'}
      </button>

      {state.isSuccess && (
        <div className="success-message">
          Form submitted successfully!
        </div>
      )}
    </form>
  )
}
Zod Error Formatting
typescript
// lib/zod-error-formatter.ts
import { ZodError } from 'zod'
import { FormFieldError } from '@/types/form-error'

export class ZodErrorFormatter {
  static format(error: ZodError): FormFieldError[] {
    return error.errors.map(err => ({
      field: this.formatPath(err.path),
      message: this.formatMessage(err),
      code: err.code,
    }))
  }

  private static formatPath(path: (string | number)[]): string {
    return path.join('.')
  }

  private static formatMessage(error: ZodError['errors'][0]): string {
    switch (error.code) {
      case 'too_small':
        return error.type === 'string' 
          ? `Must be at least ${error.minimum} characters`
          : `Must be at least ${error.minimum}`
      
      case 'too_big':
        return error.type === 'string'
          ? `Must be at most ${error.maximum} characters`
          : `Must be at most ${error.maximum}`
      
      case 'invalid_string':
        if (error.validation === 'email') return 'Invalid email address'
        if (error.validation === 'url') return 'Invalid URL'
        return 'Invalid format'
      
      case 'invalid_type':
        return `Expected ${error.expected}, received ${error.received}`
      
      default:
        return error.message
    }
  }

  static toUserFriendly(error: ZodError): string {
    const fieldErrors = this.format(error)
    
    if (fieldErrors.length === 1) {
      return fieldErrors[0].message
    }
    
    return `${fieldErrors.length} validation errors found`
  }
}

// Custom Zod schema with enhanced error messages
import { z } from 'zod'

export const userSchema = z.object({
  email: z.string()
    .email('Please enter a valid email address')
    .min(1, 'Email is required'),
  
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/\d/, 'Password must contain at least one number'),
  
  age: z.number()
    .min(18, 'Must be at least 18 years old')
    .max(120, 'Invalid age'),
}).superRefine((data, ctx) => {
  // Cross-field validation
  if (data.password.includes(data.email.split('@')[0])) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: 'Password cannot contain your email username',
      path: ['password'],
    })
  }
})
§ ASYNC ERROR HANDLING
Try/Catch Patterns for Async Operations
typescript
// lib/async-error-handling.ts
export async function safeAsync<T>(
  promise: Promise<T>,
  errorHandler?: (error: unknown) => void
): Promise<{ data: T | null; error: Error | null }> {
  try {
    const data = await promise
    return { data, error: null }
  } catch (error) {
    const err = error instanceof Error 
      ? error 
      : new Error(String(error))
    
    if (errorHandler) {
      errorHandler(err)
    } else {
      console.error('Async operation failed:', err)
    }
    
    return { data: null, error: err }
  }
}

// With retry logic
export async function retryableAsync<T>(
  operation: () => Promise<T>,
  options: {
    maxRetries?: number
    delayMs?: number
    shouldRetry?: (error: Error) => boolean
  } = {}
): Promise<T> {
  const {
    maxRetries = 3,
    delayMs = 1000,
    shouldRetry = () => true,
  } = options

  let lastError: Error

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation()
    } catch (error) {
      lastError = error instanceof Error ? error : new Error(String(error))
      
      if (attempt === maxRetries || !shouldRetry(lastError)) {
        break
      }
      
      // Exponential backoff with jitter
      const jitter = Math.random() * 200 // ±100ms
      const waitTime = delayMs * Math.pow(2, attempt) + jitter
      
      await new Promise(resolve => setTimeout(resolve, waitTime))
    }
  }

  throw lastError!
}

// Usage examples
export async function fetchUserData(userId: string) {
  return retryableAsync(
    () => fetch(`/api/users/${userId}`).then(res => {
      if (!res.ok) throw new Error(`HTTP ${res.status}`)
      return res.json()
    }),
    {
      maxRetries: 3,
      shouldRetry: (error) => {
        // Only retry on network errors or 5xx status
        const message = error.message
        return message.includes('Failed to fetch') || 
               message.match(/HTTP 5\d{2}/) !== null
      },
    }
  )
}
Promise Rejection Handling
typescript
// lib/promise-rejection-handler.ts
export class PromiseRejectionHandler {
  private static unhandledRejections = new Set<Promise<unknown>>()
  private static rejectionListeners: ((error: Error) => void)[] = []

  static track(promise: Promise<unknown>): Promise<unknown> {
    this.unhandledRejections.add(promise)
    
    promise
      .then(() => this.unhandledRejections.delete(promise))
      .catch((error) => {
        this.unhandledRejections.delete(promise)
        this.handleRejection(error)
      })
    
    return promise
  }

  static addListener(listener: (error: Error) => void): void {
    this.rejectionListeners.push(listener)
  }

  static removeListener(listener: (error: Error) => void): void {
    const index = this.rejectionListeners.indexOf(listener)
    if (index > -1) {
      this.rejectionListeners.splice(index, 1)
    }
  }

  private static handleRejection(error: Error): void {
    console.error('Unhandled Promise rejection:', error)
    
    // Notify listeners
    this.rejectionListeners.forEach(listener => {
      try {
        listener(error)
      } catch (listenerError) {
        console.error('Error in rejection listener:', listenerError)
      }
    })
  }

  static getUnhandledCount(): number {
    return this.unhandledRejections.size
  }
}

// Initialize in your app
if (typeof window !== 'undefined') {
  // Global unhandled rejection handler
  window.addEventListener('unhandledrejection', (event) => {
    event.preventDefault()
    PromiseRejectionHandler.handleRejection(event.reason)
  })
}
Async Boundary Components
typescript
// components/async-boundary.tsx
'use client'

import { ReactNode, Suspense } from 'react'
import { ErrorBoundary, FallbackProps } from 'react-error-boundary'

interface AsyncBoundaryProps {
  children: ReactNode
  loadingFallback: ReactNode
  errorFallback: ReactNode | ((props: FallbackProps) => ReactNode)
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void
  resetKeys?: any[]
}

export function AsyncBoundary({
  children,
  loadingFallback,
  errorFallback,
  onError,
  resetKeys,
}: AsyncBoundaryProps) {
  return (
    <ErrorBoundary
      fallback={errorFallback}
      onError={onError}
      resetKeys={resetKeys}
    >
      <Suspense fallback={loadingFallback}>
        {children}
      </Suspense>
    </ErrorBoundary>
  )
}

// Usage with data fetching
export function UserProfile({ userId }: { userId: string }) {
  return (
    <AsyncBoundary
      loadingFallback={<ProfileSkeleton />}
      errorFallback={(props) => (
        <ProfileError
          error={props.error}
          resetErrorBoundary={props.resetErrorBoundary}
          userId={userId}
        />
      )}
      resetKeys={[userId]}
    >
      <UserProfileContent userId={userId} />
    </AsyncBoundary>
  )
}

async function UserProfileContent({ userId }: { userId: string }) {
  const user = await fetchUser(userId)
  return <Profile user={user} />
}
Suspense + Error Boundary Combination
typescript
// components/suspense-with-error.tsx
'use client'

import { 
  Suspense, 
  Component, 
  ErrorInfo, 
  ReactNode,
  lazy 
} from 'react'

interface SuspenseWithErrorProps {
  children: ReactNode
  fallback: ReactNode
  errorComponent?: React.ComponentType<{ error: Error }>
  onError?: (error: Error) => void
}

interface State {
  hasError: boolean
  error?: Error
}

export class SuspenseWithError extends Component<
  SuspenseWithErrorProps, 
  State
> {
  constructor(props: SuspenseWithErrorProps) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error }
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo): void {
    console.error('Suspense error:', error, errorInfo)
    this.props.onError?.(error)
  }

  reset = (): void => {
    this.setState({ hasError: false, error: undefined })
  }

  render(): ReactNode {
    if (this.state.hasError && this.state.error) {
      if (this.props.errorComponent) {
        const ErrorComponent = this.props.errorComponent
        return <ErrorComponent error={this.state.error} />
      }
      
      return (
        <div className="suspense-error">
          <h3>Failed to load component</h3>
          <p>{this.state.error.message}</p>
          <button onClick={this.reset}>Retry</button>
        </div>
      )
    }

    return (
      <Suspense fallback={this.props.fallback}>
        {this.props.children}
      </Suspense>
    )
  }
}

// Usage with lazy loading
const LazyDashboard = lazy(() => import('@/components/dashboard'))

export function App() {
  return (
    <SuspenseWithError
      fallback={<DashboardLoader />}
      errorComponent={DashboardError}
      onError={(error) => {
        // Log to monitoring service
        logError('DASHBOARD_LOAD_ERROR', error)
      }}
    >
      <LazyDashboard />
    </SuspenseWithError>
  )
}
§ ERROR LOGGING
Sentry Integration
typescript
// lib/sentry/config.ts
import * as Sentry from '@sentry/nextjs'

export function initSentry() {
  if (process.env.NEXT_PUBLIC_SENTRY_DSN) {
    Sentry.init({
      dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
      environment: process.env.NODE_ENV,
      release: process.env.NEXT_PUBLIC_APP_VERSION,
      tracesSampleRate: parseFloat(process.env.SENTRY_TRACES_SAMPLE_RATE || '0.1'),
      profilesSampleRate: parseFloat(process.env.SENTRY_PROFILES_SAMPLE_RATE || '0.1'),
      
      integrations: [
        Sentry.browserTracingIntegration(),
        Sentry.replayIntegration({
          maskAllText: true,
          blockAllMedia: true,
        }),
      ],
      
      beforeSend(event, hint) {
        // Filter out specific errors
        if (hint?.originalException instanceof Error) {
          const error = hint.originalException
          
          // Ignore network errors for offline users
          if (error.message.includes('Failed to fetch') && !navigator.onLine) {
            return null
          }
          
          // Ignore specific error types
          if (error.name === 'ChunkLoadError') {
            return null
          }
        }
        
        // Add user context if available
        const user = getUserFromStorage()
        if (user) {
          event.user = {
            id: user.id,
            email: user.email,
            username: user.username,
          }
        }
        
        return event
      },
    })
  }
}

// Error boundary integration
export function withSentryErrorBoundary(
  Component: React.ComponentType,
  options?: {
    showDialog?: boolean
    fallback?: React.ComponentType<{ error: Error }>
  }
) {
  return function SentryErrorBoundaryWrapper(props: any) {
    return (
      <Sentry.ErrorBoundary
        showDialog={options?.showDialog}
        fallback={options?.fallback}
        onError={(error) => {
          Sentry.captureException(error, {
            tags: { component: Component.name },
            extra: { props },
          })
        }}
      >
        <Component {...props} />
      </Sentry.ErrorBoundary>
    )
  }
}

// Custom breadcrumbs
export function addBreadcrumb(
  message: string,
  category: string,
  data?: Record<string, any>,
  level: Sentry.SeverityLevel = 'info'
) {
  Sentry.addBreadcrumb({
    message,
    category,
    data,
    level,
  })
}

// API route monitoring
export async function withSentryApiHandler<T>(
  handler: (req: Request) => Promise<T>,
  context?: { params?: Record<string, string> }
) {
  return Sentry.withScope(async (scope) => {
    if (context?.params) {
      scope.setContext('params', context.params)
    }
    
    try {
      return await handler
    } catch (error) {
      Sentry.captureException(error)
      throw error
    }
  })
}
Error Grouping and Context
typescript
// lib/error-logging.ts
interface ErrorContext {
  userId?: string
  sessionId?: string
  route?: string
  component?: string
  userAgent?: string
  timestamp: string
  [key: string]: any
}

interface LoggedError extends Error {
  context?: ErrorContext
  fingerprint?: string[]
}

export class ErrorLogger {
  private static instance: ErrorLogger
  private context: Partial<ErrorContext> = {}
  private breadcrumbs: Array<{
    message: string
    timestamp: string
    data?: Record<string, any>
  }> = []

  static getInstance(): ErrorLogger {
    if (!ErrorLogger.instance) {
      ErrorLogger.instance = new ErrorLogger()
    }
    return ErrorLogger.instance
  }

  setContext(newContext: Partial<ErrorContext>): void {
    this.context = { ...this.context, ...newContext }
  }

  addBreadcrumb(message: string, data?: Record<string, any>): void {
    this.breadcrumbs.push({
      message,
      timestamp: new Date().toISOString(),
      data,
    })
    
    // Keep only last 100 breadcrumbs
   