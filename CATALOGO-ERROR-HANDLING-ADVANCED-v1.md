# CATALOGO-ERROR-HANDLING-ADVANCED-v1

> **Dominio**: Error Handling
> **Stack**: Next.js, React, TypeScript, shadcn/ui, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [src/lib/errors/base.ts](#1-src-lib-errors-base-ts) | `src/lib/errors/base.ts` |
| 2 | [src/lib/errors/codes.ts](#2-src-lib-errors-codes-ts) | `src/lib/errors/codes.ts` |
| 3 | [src/lib/errors/handler.ts](#3-src-lib-errors-handler-ts) | `src/lib/errors/handler.ts` |
| 4 | [src/lib/errors/logger.ts](#4-src-lib-errors-logger-ts) | `src/lib/errors/logger.ts` |
| 5 | [src/components/error-boundary.tsx](#5-src-components-error-boundary-tsx) | `src/components/error-boundary.tsx` |
| 6 | [src/app/error.tsx](#6-src-app-error-tsx) | `src/app/error.tsx` |
| 7 | [src/app/not-found.tsx](#7-src-app-not-found-tsx) | `src/app/not-found.tsx` |
| 8 | [src/app/global-error.tsx](#8-src-app-global-error-tsx) | `src/app/global-error.tsx` |
| 9 | [src/components/ui/error-alert.tsx](#9-src-components-ui-error-alert-tsx) | `src/components/ui/error-alert.tsx` |
| 10 | [src/hooks/use-error-handler.ts](#10-src-hooks-use-error-handler-ts) | `src/hooks/use-error-handler.ts` |
| 11 | [tests/error-handling.test.ts](#11-tests-error-handling-test-ts) | `tests/error-handling.test.ts` |

---

## 1. src/lib/errors/base.ts

```typescript
// src/lib/errors/base.ts
/**
 * @fileoverview Defines base and specific application error classes.
 * These custom error classes provide structured error information,
 * including a unique code, HTTP status, and operational status,
 * making error handling more predictable and robust.
 */

/**
 * Base class for all application-specific errors.
 * Extends the native Error class to add structured properties
 * like a unique error code, HTTP status code, and an operational flag.
 */
export class AppError extends Error {
  /**
   * Constructs an AppError instance.
   * @param message - A human-readable error message.
   * @param code - A unique, machine-readable error code (e.g., "VALIDATION_ERROR").
   * @param statusCode - The HTTP status code associated with this error (default: 500).
   * @param isOperational - A flag indicating if the error is operational (expected, handled)
   *                        or programmatic (unexpected, bug). Operational errors can be
   *                        safely exposed to the client with appropriate messages.
   * @param details - Optional additional details about the error, useful for debugging.
   */
  constructor(
    message: string,
    public code: string,
    public statusCode: number = 500,
    public isOperational: boolean = true,
    public details?: unknown
  ) {
    super(message);
    // Set the prototype explicitly to ensure `instanceof` works correctly
    Object.setPrototypeOf(this, AppError.prototype);
    this.name = this.constructor.name; // Set name to the class name (e.g., "AppError")

    // Capture stack trace, excluding the constructor call from the stack.
    // This makes stack traces cleaner by pointing to where the error was created.
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }

  /**
   * Returns a JSON representation of the error, useful for API responses.
   * Excludes sensitive or internal properties like `stack` in production environments.
   * @returns An object containing serializable error properties.
   */
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode,
      details: this.details,
      // Stack trace is typically omitted in production for security/verbosity,
      // but can be included for development or specific logging.
      // stack: process.env.NODE_ENV === 'development' ? this.stack : undefined,
    };
  }
}

/**
 * Error class for validation failures (e.g., invalid input, missing fields).
 * Corresponds to HTTP 400 Bad Request.
 */
export class ValidationError extends AppError {
  constructor(message: string, code: string = "INVALID_INPUT", details?: unknown) {
    super(message, code, 400, true, details);
    Object.setPrototypeOf(this, ValidationError.prototype);
  }
}

/**
 * Error class for authentication failures (e.g., invalid credentials, missing token).
 * Corresponds to HTTP 401 Unauthorized.
 */
export class AuthenticationError extends AppError {
  constructor(message: string, code: string = "UNAUTHENTICATED", details?: unknown) {
    super(message, code, 401, true, details);
    Object.setPrototypeOf(this, AuthenticationError.prototype);
  }
}

/**
 * Error class for authorization failures (e.g., insufficient permissions).
 * Corresponds to HTTP 403 Forbidden.
 */
export class AuthorizationError extends AppError {
  constructor(message: string, code: string = "UNAUTHORIZED", details?: unknown) {
    super(message, code, 403, true, details);
    Object.setPrototypeOf(this, AuthorizationError.prototype);
  }
}

/**
 * Error class for resources not found.
 * Corresponds to HTTP 404 Not Found.
 */
export class NotFoundError extends AppError {
  constructor(message: string, code: string = "NOT_FOUND", details?: unknown) {
    super(message, code, 404, true, details);
    Object.setPrototypeOf(this, NotFoundError.prototype);
  }
}

/**
 * Error class for conflicts (e.g., resource already exists, optimistic locking failure).
 * Corresponds to HTTP 409 Conflict.
 */
export class ConflictError extends AppError {
  constructor(message: string, code: string = "CONFLICT", details?: unknown) {
    super(message, code, 409, true, details);
    Object.setPrototypeOf(this, ConflictError.prototype);
  }
}

/**
 * Error class for rate limiting exceeded.
 * Corresponds to HTTP 429 Too Many Requests.
 */
export class RateLimitError extends AppError {
  constructor(message: string, code: string = "RATE_LIMIT_EXCEEDED", details?: unknown) {
    super(message, code, 429, true, details);
    Object.setPrototypeOf(this, RateLimitError.prototype);
  }
}

/**
 * Error class for issues with external services (e.g., API timeouts, invalid responses).
 * Corresponds to HTTP 502 Bad Gateway or 503 Service Unavailable.
 */
export class ExternalServiceError extends AppError {
  constructor(message: string, code: string = "EXTERNAL_SERVICE_ERROR", statusCode: number = 502, details?: unknown) {
    super(message, code, statusCode, false, details); // Often non-operational, as it indicates an external dependency issue
    Object.setPrototypeOf(this, ExternalServiceError.prototype);
  }
}

/**
 * Error class for database-related issues (e.g., connection errors, query failures).
 * Corresponds to HTTP 500 Internal Server Error.
 */
export class DatabaseError extends AppError {
  constructor(message: string, code: string = "DATABASE_ERROR", details?: unknown) {
    super(message, code, 500, false, details); // Database errors are typically programmatic/non-operational
    Object.setPrototypeOf(this, DatabaseError.prototype);
  }
}

/**
 * Error class for when the service is temporarily unavailable.
 * Corresponds to HTTP 503 Service Unavailable.
 */
export class ServiceUnavailableError extends AppError {
  constructor(message: string, code: string = "SERVICE_UNAVAILABLE", details?: unknown) {
    super(message, code, 503, false, details);
    Object.setPrototypeOf(this, ServiceUnavailableError.prototype);
  }
}

/**
 * Generic internal server error.
 * Corresponds to HTTP 500 Internal Server Error.
 */
export class InternalServerError extends AppError {
  constructor(message: string = "An unexpected internal server error occurred.", code: string = "INTERNAL_ERROR", details?: unknown) {
    super(message, code, 500, false, details);
    Object.setPrototypeOf(this, InternalServerError.prototype);
  }
}
```

---

## 2. src/lib/errors/codes.ts

```typescript
// src/lib/errors/codes.ts
/**
 * @fileoverview Defines a comprehensive set of error codes,
 * their corresponding HTTP status codes, and user-friendly messages.
 * This centralizes error metadata for consistent error responses and logging.
 */

/**
 * Enum for standardized, machine-readable error codes across the application.
 * These codes help in programmatically identifying and handling specific error scenarios.
 */
export enum ErrorCode {
  // --- General Errors ---
  UNKNOWN_ERROR = "UNKNOWN_ERROR",
  INTERNAL_ERROR = "INTERNAL_ERROR",
  SERVICE_UNAVAILABLE = "SERVICE_UNAVAILABLE",
  TIMEOUT_ERROR = "TIMEOUT_ERROR",
  NETWORK_ERROR = "NETWORK_ERROR",

  // --- Validation Errors (400 Bad Request) ---
  VALIDATION_ERROR = "VALIDATION_ERROR", // Generic validation failure
  INVALID_INPUT = "INVALID_INPUT",
  MISSING_FIELD = "MISSING_FIELD",
  INVALID_FORMAT = "INVALID_FORMAT",
  VALUE_TOO_SHORT = "VALUE_TOO_SHORT",
  VALUE_TOO_LONG = "VALUE_TOO_LONG",
  INVALID_ENUM_VALUE = "INVALID_ENUM_VALUE",
  INVALID_DATE = "INVALID_DATE",
  INVALID_EMAIL = "INVALID_EMAIL",
  INVALID_PASSWORD = "INVALID_PASSWORD",

  // --- Authentication Errors (401 Unauthorized) ---
  UNAUTHENTICATED = "UNAUTHENTICATED",
  INVALID_CREDENTIALS = "INVALID_CREDENTIALS",
  TOKEN_EXPIRED = "TOKEN_EXPIRED",
  INVALID_TOKEN = "INVALID_TOKEN",
  MISSING_TOKEN = "MISSING_TOKEN",
  ACCOUNT_LOCKED = "ACCOUNT_LOCKED",
  ACCOUNT_DISABLED = "ACCOUNT_DISABLED",

  // --- Authorization Errors (403 Forbidden) ---
  UNAUTHORIZED = "UNAUTHORIZED",
  FORBIDDEN = "FORBIDDEN",
  INSUFFICIENT_PERMISSIONS = "INSUFFICIENT_PERMISSIONS",
  ACCESS_DENIED = "ACCESS_DENIED",

  // --- Resource Errors (404 Not Found, 409 Conflict) ---
  NOT_FOUND = "NOT_FOUND",
  RESOURCE_NOT_FOUND = "RESOURCE_NOT_FOUND",
  USER_NOT_FOUND = "USER_NOT_FOUND",
  PRODUCT_NOT_FOUND = "PRODUCT_NOT_FOUND",
  ORDER_NOT_FOUND = "ORDER_NOT_FOUND",

  ALREADY_EXISTS = "ALREADY_EXISTS",
  CONFLICT = "CONFLICT",
  DUPLICATE_ENTRY = "DUPLICATE_ENTRY",
  EMAIL_ALREADY_REGISTERED = "EMAIL_ALREADY_REGISTERED",
  USERNAME_ALREADY_TAKEN = "USERNAME_ALREADY_TAKEN",

  // --- Rate Limiting Errors (429 Too Many Requests) ---
  RATE_LIMIT_EXCEEDED = "RATE_LIMIT_EXCEEDED",

  // --- External Service Errors (5xx) ---
  EXTERNAL_SERVICE_ERROR = "EXTERNAL_SERVICE_ERROR",
  THIRD_PARTY_API_FAILURE = "THIRD_PARTY_API_FAILURE",
  PAYMENT_SERVICE_ERROR = "PAYMENT_SERVICE_ERROR",
  EMAIL_SERVICE_ERROR = "EMAIL_SERVICE_ERROR",

  // --- Database Errors (5xx) ---
  DATABASE_ERROR = "DATABASE_ERROR",
  DB_CONNECTION_FAILED = "DB_CONNECTION_FAILED",
  DB_QUERY_FAILED = "DB_QUERY_FAILED",
  DB_UNIQUE_CONSTRAINT_VIOLATION = "DB_UNIQUE_CONSTRAINT_VIOLATION",
  DB_RECORD_NOT_FOUND = "DB_RECORD_NOT_FOUND", // Specific for DB not found, can map to 404
}

/**
 * Mapping from ErrorCode to standard HTTP status codes.
 * This ensures consistent HTTP responses for specific error types.
 */
export const errorCodeToStatus: Record<ErrorCode, number> = {
  // General
  [ErrorCode.UNKNOWN_ERROR]: 500,
  [ErrorCode.INTERNAL_ERROR]: 500,
  [ErrorCode.SERVICE_UNAVAILABLE]: 503,
  [ErrorCode.TIMEOUT_ERROR]: 504, // Gateway Timeout
  [ErrorCode.NETWORK_ERROR]: 503, // Service Unavailable or 502 Bad Gateway

  // Validation (400)
  [ErrorCode.VALIDATION_ERROR]: 400,
  [ErrorCode.INVALID_INPUT]: 400,
  [ErrorCode.MISSING_FIELD]: 400,
  [ErrorCode.INVALID_FORMAT]: 400,
  [ErrorCode.VALUE_TOO_SHORT]: 400,
  [ErrorCode.VALUE_TOO_LONG]: 400,
  [ErrorCode.INVALID_ENUM_VALUE]: 400,
  [ErrorCode.INVALID_DATE]: 400,
  [ErrorCode.INVALID_EMAIL]: 400,
  [ErrorCode.INVALID_PASSWORD]: 400,

  // Authentication (401)
  [ErrorCode.UNAUTHENTICATED]: 401,
  [ErrorCode.INVALID_CREDENTIALS]: 401,
  [ErrorCode.TOKEN_EXPIRED]: 401,
  [ErrorCode.INVALID_TOKEN]: 401,
  [ErrorCode.MISSING_TOKEN]: 401,
  [ErrorCode.ACCOUNT_LOCKED]: 401,
  [ErrorCode.ACCOUNT_DISABLED]: 401,

  // Authorization (403)
  [ErrorCode.UNAUTHORIZED]: 403,
  [ErrorCode.FORBIDDEN]: 403,
  [ErrorCode.INSUFFICIENT_PERMISSIONS]: 403,
  [ErrorCode.ACCESS_DENIED]: 403,

  // Resource (404, 409)
  [ErrorCode.NOT_FOUND]: 404,
  [ErrorCode.RESOURCE_NOT_FOUND]: 404,
  [ErrorCode.USER_NOT_FOUND]: 404,
  [ErrorCode.PRODUCT_NOT_FOUND]: 404,
  [ErrorCode.ORDER_NOT_FOUND]: 404,

  [ErrorCode.ALREADY_EXISTS]: 409,
  [ErrorCode.CONFLICT]: 409,
  [ErrorCode.DUPLICATE_ENTRY]: 409,
  [ErrorCode.EMAIL_ALREADY_REGISTERED]: 409,
  [ErrorCode.USERNAME_ALREADY_TAKEN]: 409,

  // Rate Limiting (429)
  [ErrorCode.RATE_LIMIT_EXCEEDED]: 429,

  // External Service (5xx)
  [ErrorCode.EXTERNAL_SERVICE_ERROR]: 502, // Bad Gateway
  [ErrorCode.THIRD_PARTY_API_FAILURE]: 502,
  [ErrorCode.PAYMENT_SERVICE_ERROR]: 503, // Service Unavailable
  [ErrorCode.EMAIL_SERVICE_ERROR]: 503,

  // Database (5xx)
  [ErrorCode.DATABASE_ERROR]: 500,
  [ErrorCode.DB_CONNECTION_FAILED]: 500,
  [ErrorCode.DB_QUERY_FAILED]: 500,
  [ErrorCode.DB_UNIQUE_CONSTRAINT_VIOLATION]: 409, // Specific DB error mapped to conflict
  [ErrorCode.DB_RECORD_NOT_FOUND]: 404, // Specific DB error mapped to not found
};

/**
 * User-friendly error messages associated with each ErrorCode.
 * These messages can be displayed directly to the client or used as a base for localization.
 */
export const errorMessages: Record<ErrorCode, string> = {
  // General
  [ErrorCode.UNKNOWN_ERROR]: "An unexpected error occurred. Please try again later.",
  [ErrorCode.INTERNAL_ERROR]: "An internal server error occurred. We are working to fix it.",
  [ErrorCode.SERVICE_UNAVAILABLE]: "The service is temporarily unavailable. Please try again shortly.",
  [ErrorCode.TIMEOUT_ERROR]: "The request timed out. Please try again.",
  [ErrorCode.NETWORK_ERROR]: "A network error occurred. Please check your internet connection.",

  // Validation
  [ErrorCode.VALIDATION_ERROR]: "One or more input fields are invalid.",
  [ErrorCode.INVALID_INPUT]: "The provided input is invalid.",
  [ErrorCode.MISSING_FIELD]: "A required field is missing.",
  [ErrorCode.INVALID_FORMAT]: "The input is not in the correct format.",
  [ErrorCode.VALUE_TOO_SHORT]: "The value provided is too short.",
  [ErrorCode.VALUE_TOO_LONG]: "The value provided is too long.",
  [ErrorCode.INVALID_ENUM_VALUE]: "The provided value is not a valid option.",
  [ErrorCode.INVALID_DATE]: "The date provided is invalid.",
  [ErrorCode.INVALID_EMAIL]: "Please enter a valid email address.",
  [ErrorCode.INVALID_PASSWORD]: "Password must be at least 8 characters long and include a number and a special character.",

  // Authentication
  [ErrorCode.UNAUTHENTICATED]: "You are not authenticated. Please log in.",
  [ErrorCode.INVALID_CREDENTIALS]: "Invalid email or password.",
  [ErrorCode.TOKEN_EXPIRED]: "Your session has expired. Please log in again.",
  [ErrorCode.INVALID_TOKEN]: "The authentication token is invalid.",
  [ErrorCode.MISSING_TOKEN]: "Authentication token is missing.",
  [ErrorCode.ACCOUNT_LOCKED]: "Your account has been locked due to too many failed login attempts.",
  [ErrorCode.ACCOUNT_DISABLED]: "Your account has been disabled. Please contact support.",

  // Authorization
  [ErrorCode.UNAUTHORIZED]: "You are not authorized to perform this action.",
  [ErrorCode.FORBIDDEN]: "Access to this resource is forbidden.",
  [ErrorCode.INSUFFICIENT_PERMISSIONS]: "You do not have the necessary permissions.",
  [ErrorCode.ACCESS_DENIED]: "Access denied.",

  // Resource
  [ErrorCode.NOT_FOUND]: "The requested resource could not be found.",
  [ErrorCode.RESOURCE_NOT_FOUND]: "The requested resource could not be found.",
  [ErrorCode.USER_NOT_FOUND]: "The specified user could not be found.",
  [ErrorCode.PRODUCT_NOT_FOUND]: "The specified product could not be found.",
  [ErrorCode.ORDER_NOT_FOUND]: "The specified order could not be found.",

  [ErrorCode.ALREADY_EXISTS]: "The resource you are trying to create already exists.",
  [ErrorCode.CONFLICT]: "A conflict occurred with the current state of the resource.",
  [ErrorCode.DUPLICATE_ENTRY]: "A duplicate entry was found.",
  [ErrorCode.EMAIL_ALREADY_REGISTERED]: "This email address is already registered.",
  [ErrorCode.USERNAME_ALREADY_TAKEN]: "This username is already taken.",

  // Rate Limiting
  [ErrorCode.RATE_LIMIT_EXCEEDED]: "You have exceeded the rate limit. Please try again later.",

  // External Service
  [ErrorCode.EXTERNAL_SERVICE_ERROR]: "An external service encountered an error.",
  [ErrorCode.THIRD_PARTY_API_FAILURE]: "A third-party API failed to respond correctly.",
  [ErrorCode.PAYMENT_SERVICE_ERROR]: "The payment service is currently unavailable.",
  [ErrorCode.EMAIL_SERVICE_ERROR]: "The email service encountered an error.",

  // Database
  [ErrorCode.DATABASE_ERROR]: "A database error occurred.",
  [ErrorCode.DB_CONNECTION_FAILED]: "Could not connect to the database.",
  [ErrorCode.DB_QUERY_FAILED]: "A database query failed.",
  [ErrorCode.DB_UNIQUE_CONSTRAINT_VIOLATION]: "A record with this unique identifier already exists.",
  [ErrorCode.DB_RECORD_NOT_FOUND]: "The requested database record could not be found.",
};
```

---

## 3. src/lib/errors/handler.ts

```typescript
// src/lib/errors/handler.ts
/**
 * @fileoverview Provides a central error handling utility to normalize
 * various types of errors (e.g., native, database, validation) into
 * standardized AppError instances. This ensures consistent error responses
 * and simplifies error processing throughout the application.
 */

import {
  AppError,
  ValidationError,
  NotFoundError,
  ConflictError,
  AuthenticationError,
  AuthorizationError,
  ExternalServiceError,
  DatabaseError,
  InternalServerError,
} from "./base";
import { ErrorCode, errorCodeToStatus, errorMessages } from "./codes";

// --- External Library Type Imports (mocked for demonstration if not installed) ---
// For Prisma:
// npm install @prisma/client
// import { Prisma } from '@prisma/client';
// For Zod:
// npm install zod
// import { ZodError, ZodIssue } from 'zod';
// For tRPC:
// npm install @trpc/server
// import { TRPCError } from '@trpc/server';

// Mock types if actual libraries are not installed for this example
declare module "@prisma/client/runtime/library" {
  export class PrismaClientKnownRequestError extends Error {
    code: string;
    meta?: Record<string, unknown>;
    clientVersion: string;
    constructor(message: string, options: { code: string; clientVersion: string; meta?: Record<string, unknown> });
  }
}
import { PrismaClientKnownRequestError } from "@prisma/client/runtime/library";

interface ZodIssue {
  code: string;
  expected?: string;
  received?: string;
  path: (string | number)[];
  message: string;
  unionErrors?: ZodError[];
}
export class ZodError extends Error {
  issues: ZodIssue[];
  constructor(issues: ZodIssue[]);
  format(): Record<string, unknown>;
}

export class TRPCError extends Error {
  code: string; // e.g., 'BAD_REQUEST', 'UNAUTHORIZED', 'FORBIDDEN', 'NOT_FOUND', 'INTERNAL_SERVER_ERROR'
  message: string;
  cause?: unknown;
  constructor(opts: { message?: string; code: string; cause?: unknown });
}
// --- End External Library Type Imports ---

/**
 * Global error handler function.
 * It takes an unknown error and attempts to normalize it into an AppError.
 * This function acts as the central point for processing all errors
 * before they are logged or sent as responses.
 * @param error The error to handle, which can be of any type.
 * @returns An AppError instance, ensuring a consistent error structure.
 */
export function handleError(error: unknown): AppError {
  // 1. If the error is already an AppError, return it directly.
  if (error instanceof AppError) {
    return error;
  }

  // 2. Handle specific known error types from external libraries.
  // Prisma errors (database errors)
  if (error instanceof PrismaClientKnownRequestError) {
    return handlePrismaError(error);
  }

  // Zod validation errors
  if (error instanceof ZodError) {
    return handleZodError(error);
  }

  // tRPC errors
  if (error instanceof TRPCError) {
    return handleTRPCError(error);
  }

  // 3. Handle generic native Error instances.
  if (error instanceof Error) {
    // Check for common network/timeout errors
    if ((error as NodeJS.ErrnoException).code === 'ECONNREFUSED' || error.message.includes('connect ECONNREFUSED')) {
      return new ExternalServiceError(
        errorMessages[ErrorCode.NETWORK_ERROR],
        ErrorCode.NETWORK_ERROR,
        errorCodeToStatus[ErrorCode.NETWORK_ERROR],
        { originalMessage: error.message }
      );
    }
    if (error.name === 'AbortError' || error.message.includes('timeout')) {
      return new ExternalServiceError(
        errorMessages[ErrorCode.TIMEOUT_ERROR],
        ErrorCode.TIMEOUT_ERROR,
        errorCodeToStatus[ErrorCode.TIMEOUT_ERROR],
        { originalMessage: error.message }
      );
    }

    // For any other generic Error, wrap it as an InternalServerError.
    // These are typically unexpected programmatic errors.
    return new InternalServerError(
      errorMessages[ErrorCode.INTERNAL_ERROR],
      ErrorCode.INTERNAL_ERROR,
      { originalMessage: error.message, stack: error.stack }
    );
  }

  // 4. Handle completely unknown error types (e.g., strings, numbers, objects).
  // These are highly unexpected and indicate a programmatic issue.
  return new InternalServerError(
    errorMessages[ErrorCode.UNKNOWN_ERROR],
    ErrorCode.UNKNOWN_ERROR,
    { originalError: error }
  );
}

/**
 * Handles Prisma-specific errors and converts them into AppError instances.
 * @param error The PrismaClientKnownRequestError instance.
 * @returns An AppError subclass or AppError.
 */
function handlePrismaError(error: PrismaClientKnownRequestError): AppError {
  switch (error.code) {
    case "P2002": // Unique constraint violation
      const target = (error.meta?.target as string[])?.join(", ") || "field(s)";
      return new ConflictError(
        `Duplicate entry: A record with this ${target} already exists.`,
        ErrorCode.DB_UNIQUE_CONSTRAINT_VIOLATION,
        { target, originalError: error.message }
      );
    case "P2025": // Record not found
      const modelName = (error.meta?.modelName as string) || "record";
      return new NotFoundError(
        `The requested ${modelName} could not be found.`,
        ErrorCode.DB_RECORD_NOT_FOUND,
        { originalError: error.message }
      );
    case "P2003": // Foreign key constraint failed
      return new ConflictError(
        "Foreign key constraint failed. Related resource does not exist or cannot be deleted.",
        ErrorCode.DATABASE_ERROR,
        { originalError: error.message }
      );
    // Add more Prisma error codes as needed
    default:
      // For other Prisma errors, treat as a generic database error
      return new DatabaseError(
        `A database operation failed: ${error.message}`,
        ErrorCode.DATABASE_ERROR,
        { prismaCode: error.code, originalError: error.message }
      );
  }
}

/**
 * Handles Zod validation errors and converts them into a ValidationError.
 * Extracts detailed validation issues for client-side feedback.
 * @param error The ZodError instance.
 * @returns A ValidationError instance.
 */
function handleZodError(error: ZodError): ValidationError {
  const details = error.issues.map((issue: ZodIssue) => ({
    path: issue.path.join("."),
    message: issue.message,
    code: issue.code,
  }));
  return new ValidationError(
    errorMessages[ErrorCode.VALIDATION_ERROR],
    ErrorCode.VALIDATION_ERROR,
    details
  );
}

/**
 * Handles tRPC errors and converts them into appropriate AppError instances.
 * Maps tRPC error codes to standard HTTP status codes and AppError types.
 * @param error The TRPCError instance.
 * @returns An AppError subclass or AppError.
 */
function handleTRPCError(error: TRPCError): AppError {
  const message = error.message || errorMessages[ErrorCode.INTERNAL_ERROR];
  const details = { originalTRPCCode: error.code, originalMessage: error.message, cause: error.cause };

  switch (error.code) {
    case "BAD_REQUEST":
      return new ValidationError(message, ErrorCode.INVALID_INPUT, details);
    case "UNAUTHORIZED":
      return new AuthenticationError(message, ErrorCode.UNAUTHENTICATED, details);
    case "FORBIDDEN":
      return new AuthorizationError(message, ErrorCode.UNAUTHORIZED, details);
    case "NOT_FOUND":
      return new NotFoundError(message, ErrorCode.NOT_FOUND, details);
    case "CONFLICT":
      return new ConflictError(message, ErrorCode.CONFLICT, details);
    case "TOO_MANY_REQUESTS":
      return new RateLimitError(message, ErrorCode.RATE_LIMIT_EXCEEDED, details);
    case "INTERNAL_SERVER_ERROR":
      return new InternalServerError(message, ErrorCode.INTERNAL_ERROR, details);
    case "TIMEOUT":
      return new ExternalServiceError(message, ErrorCode.TIMEOUT_ERROR, 504, details);
    case "PARSE_ERROR":
    case "CLIENT_CLOSED_REQUEST":
    case "METHOD_NOT_SUPPORTED":
    case "PRECONDITION_FAILED":
    case "PAYLOAD_TOO_LARGE":
    case "UNPROCESSABLE_CONTENT":
    case "SERVER_ERROR": // Generic server error from tRPC
    default:
      // Map tRPC code to a generic internal server error or a more specific one if possible
      const statusCode = errorCodeToStatus[ErrorCode.INTERNAL_ERROR];
      return new InternalServerError(
        message,
        ErrorCode.INTERNAL_ERROR,
        details
      );
  }
}
```

---

## 4. src/lib/errors/logger.ts

```typescript
// src/lib/errors/logger.ts
/**
 * @fileoverview Provides centralized logging functions for errors, warnings, and info messages.
 * It integrates with AppError for structured logging and includes hooks for external
 * error reporting services like Sentry.
 */

import { AppError, InternalServerError } from "./base";
import { ErrorCode } from "./codes";

/**
 * Interface defining the structure of an error log entry.
 * This helps standardize the data captured for each error.
 */
export interface ErrorLog {
  timestamp: string;
  level: "error" | "warn" | "info";
  error?: {
    name: string;
    message: string;
    code?: string;
    statusCode?: number;
    isOperational?: boolean;
    stack?: string;
    details?: unknown;
  };
  message?: string; // For warn/info logs
  context?: {
    userId?: string;
    path?: string;
    method?: string;
    ip?: string;
    userAgent?: string;
    requestId?: string; // Unique ID for a request, useful for tracing
    [key: string]: unknown; // Allow arbitrary context properties
  };
  details?: unknown; // For warn/info logs
}

/**
 * Logs an error to the console and potentially to external services.
 * This function is overloaded to accept either an AppError or a generic Error.
 * @param error The error object to log (AppError or native Error).
 * @param context Optional contextual information related to the error (e.g., user ID, request path).
 */
export function logError(error: AppError | Error, context?: ErrorLog["context"]): void {
  let appError: AppError;

  if (error instanceof AppError) {
    appError = error;
  } else {
    // Wrap generic errors as InternalServerError for consistent logging
    appError = new InternalServerError(
      error.message || "An unknown error occurred.",
      ErrorCode.INTERNAL_ERROR,
      { originalError: error.message, stack: error.stack }
    );
    // Mark as non-operational as it's an unexpected generic error
    appError.isOperational = false;
  }

  const logEntry: ErrorLog = {
    timestamp: new Date().toISOString(),
    level: "error",
    error: {
      name: appError.name,
      message: appError.message,
      code: appError.code,
      statusCode: appError.statusCode,
      isOperational: appError.isOperational,
      stack: appError.stack, // Include stack trace for errors
      details: appError.details,
    },
    context: context,
  };

  // Log to console (or a more sophisticated logger like Winston/Pino)
  // In production, you might filter stack traces or sensitive details.
  console.error("[ERROR]", JSON.stringify(logEntry, null, 2));

  // Report non-operational errors to external services
  if (!appError.isOperational) {
    reportToSentry(appError, context);
    // reportToLogRocket(appError, context); // Example for another service
  }
}

/**
 * Logs a warning message.
 * @param message The warning message.
 * @param details Optional additional details for the warning.
 * @param context Optional contextual information.
 */
export function logWarning(message: string, details?: unknown, context?: ErrorLog["context"]): void {
  const logEntry: ErrorLog = {
    timestamp: new Date().toISOString(),
    level: "warn",
    message: message,
    details: details,
    context: context,
  };
  console.warn("[WARN]", JSON.stringify(logEntry, null, 2));
}

/**
 * Logs an informational message.
 * @param message The informational message.
 * @param details Optional additional details for the info log.
 * @param context Optional contextual information.
 */
export function logInfo(message: string, details?: unknown, context?: ErrorLog["context"]): void {
  const logEntry: ErrorLog = {
    timestamp: new Date().toISOString(),
    level: "info",
    message: message,
    details: details,
    context: context,
  };
  console.info("[INFO]", JSON.stringify(logEntry, null, 2));
}

/**
 * Reports an error to Sentry (or a similar external error tracking service).
 * This is a placeholder function; in a real application, you would integrate
 * with the Sentry SDK (e.g., `Sentry.captureException`).
 * @param error The AppError instance to report.
 * @param context Optional additional context to send to Sentry.
 */
export function reportToSentry(error: AppError, context?: object): void {
  if (process.env.NODE_ENV === "production" && process.env.NEXT_PUBLIC_SENTRY_DSN) {
    // Example Sentry integration (requires @sentry/nextjs or similar SDK)
    // import * as Sentry from '@sentry/nextjs';
    // Sentry.withScope((scope) => {
    //   scope.setTag("error_code", error.code);
    //   scope.setExtra("statusCode", error.statusCode);
    //   scope.setExtra("isOperational", error.isOperational);
    //   if (error.details) scope.setExtra("details", error.details);
    //   if (context) scope.setContext("app_context", context);
    //   Sentry.captureException(error);
    // });
    console.warn("[SENTRY] Reporting error to Sentry (mocked):", error.name, error.message, context);
  } else {
    console.warn("[SENTRY] Sentry reporting is disabled or DSN not configured.");
  }
}

// Example for another service (e.g., LogRocket)
// export function reportToLogRocket(error: AppError, context?: object): void {
//   if (process.env.NODE_ENV === "production" && typeof window !== "undefined" && window.LogRocket) {
//     window.LogRocket.captureException(error, {
//       extra: {
//         code: error.code,
//         statusCode: error.statusCode,
//         isOperational: error.isOperational,
//         details: error.details,
//         ...context,
//       },
//     });
//   }
// }
```

---

## 5. src/components/error-boundary.tsx

```typescript
// src/components/error-boundary.tsx
"use client";

/**
 * @fileoverview Provides React Error Boundary components and hooks
 * for gracefully handling runtime errors in client-side React trees.
 * This prevents entire applications from crashing due to unexpected errors
 * in a component's render, lifecycle methods, or event handlers.
 */

import { Component, type ReactNode, useState, useCallback, useEffect } from "react";
import { Button } from "@/components/ui/button"; // Assuming shadcn/ui button
import { AlertTriangle, RefreshCw } from "lucide-react"; // Assuming lucide-react for icons
import { logError } from "@/lib/errors/logger"; // Import our error logger
import { AppError, InternalServerError } from "@/lib/errors/base";
import { ErrorCode } from "@/lib/errors/codes";

interface Props {
  /** The children components that the ErrorBoundary will protect. */
  children: ReactNode;
  /** Optional fallback UI to render when an error occurs. */
  fallback?: ReactNode;
  /** Callback function invoked when an error is caught. */
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
  /** A key that, when changed, will reset the error boundary's state. Useful for retrying. */
  resetKeys?: unknown[];
}

interface State {
  hasError: boolean;
  error?: Error;
}

/**
 * A React Error Boundary component.
 * Catches JavaScript errors anywhere in its child component tree, logs them,
 * and displays a fallback UI instead of crashing the entire application.
 *
 * Usage:
 * <ErrorBoundary fallback={<p>Something went wrong!</p>}>
 *   <MyProblematicComponent />
 * </ErrorBoundary>
 */
export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: undefined };
  }

  /**
   * Static method to update state when an error is thrown.
   * This method is called after an error has been thrown by a descendant component.
   * It receives the error that was thrown as a parameter.
   * @param error The error that was thrown.
   * @returns An object to update the component's state.
   */
  static getDerivedStateFromError(error: Error): State {
    // Update state so the next render will show the fallback UI.
    return { hasError: true, error };
  }

  /**
   * This method is called after an error has been thrown by a descendant component.
   * It receives two parameters: the error and an object with `componentStack` information.
   * This is the place to log error information.
   * @param error The error that was thrown.
   * @param errorInfo An object containing `componentStack` information.
   */
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // You can also log the error to an error reporting service
    logError(error, { componentStack: errorInfo.componentStack });

    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  /**
   * Resets the error boundary's state, allowing it to re-render its children.
   * This is typically called when a "Try Again" button is clicked.
   */
  handleRetry = () => {
    this.setState({ hasError: false, error: undefined });
  };

  /**
   * Resets the error boundary when `resetKeys` prop changes.
   * This is useful for scenarios where you want to retry rendering
   * the children based on external state changes (e.g., route change).
   */
  componentDidUpdate(prevProps: Props) {
    if (this.state.hasError && this.props.resetKeys && prevProps.resetKeys !== this.props.resetKeys) {
      // Check if any of the resetKeys have actually changed
      const changed = this.props.resetKeys.some((key, index) => key !== prevProps.resetKeys?.[index]);
      if (changed) {
        this.setState({ hasError: false, error: undefined });
      }
    }
  }

  render() {
    if (this.state.hasError) {
      // You can render any custom fallback UI
      if (this.props.fallback) {
        return this.props.fallback;
      }

      // Default fallback UI
      const errorToDisplay = this.state.error instanceof AppError
        ? this.state.error
        : new InternalServerError(this.state.error?.message || "An unexpected error occurred.", ErrorCode.INTERNAL_ERROR, { originalError: this.state.error?.message, stack: this.state.error?.stack });

      return (
        <div className="flex flex-col items-center justify-center min-h-screen bg-gray-50 text-gray-800 p-6">
          <AlertTriangle className="w-16 h-16 text-red-500 mb-6" />
          <h1 className="text-3xl font-bold mb-4">Oops! Something went wrong.</h1>
          <p className="text-lg text-gray-600 mb-6">
            {errorToDisplay.isOperational ? errorToDisplay.message : "An unexpected error occurred. We're working on a fix."}
          </p>
          {process.env.NODE_ENV === 'development' && errorToDisplay.stack && (
            <details className="mb-6 p-4 bg-gray-100 rounded-md w-full max-w-lg text-sm text-gray-700">
              <summary className="font-semibold cursor-pointer">Error Details (Dev Only)</summary>
              <pre className="whitespace-pre-wrap break-words mt-2">{errorToDisplay.stack}</pre>
            </details>
          )}
          <Button onClick={this.handleRetry} className="flex items-center gap-2">
            <RefreshCw className="w-4 h-4" /> Try again
          </Button>
        </div>
      );
    }

    return this.props.children;
  }
}

/**
 * A custom React hook to manage error state and provide a way to trigger
 * an error boundary from a functional component.
 * This hook is useful when you want to catch errors that occur outside of
 * render methods (e.g., in event handlers or async functions) and propagate
 * them to the nearest ErrorBoundary.
 *
 * Usage:
 * const { error, resetError, triggerError } = useErrorBoundary();
 *
 * if (error) return <p>Caught error: {error.message}</p>;
 *
 * const handleClick = () => {
 *   try {
 *     // ... some logic that might throw
 *   } catch (e) {
 *     triggerError(e); // Propagate error to ErrorBoundary
 *   }
 * };
 */
export function useErrorBoundary() {
  const [error, setError] = useState<Error | null>(null);

  // Effect to reset error when component unmounts or if a reset key changes
  useEffect(() => {
    return () => setError(null);
  }, []);

  const triggerError = useCallback((err: unknown) => {
    if (err instanceof Error) {
      setError(err);
    } else {
      setError(new Error(String(err))); // Convert non-Error types to Error
    }
  }, []);

  const resetError = useCallback(() => {
    setError(null);
  }, []);

  return { error, triggerError, resetError };
}

/**
 * A Higher-Order Component (HOC) to wrap a functional or class component
 * with an ErrorBoundary.
 *
 * Usage:
 * export default withErrorBoundary(MyComponent, <p>Component failed!</p>);
 */
export function withErrorBoundary<P extends object>(
  WrappedComponent: React.ComponentType<P>,
  fallback?: ReactNode,
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void,
  resetKeys?: unknown[]
) {
  const ComponentWithName = WrappedComponent.displayName || WrappedComponent.name || 'Component';

  const HOC = (props: P) => (
    <ErrorBoundary fallback={fallback} onError={onError} resetKeys={resetKeys}>
      <WrappedComponent {...props} />
    </ErrorBoundary>
  );

  HOC.displayName = `withErrorBoundary(${ComponentWithName})`;
  return HOC;
}
```

---

## 6. src/app/error.tsx

```typescript
// src/app/error.tsx
"use client";

/**
 * @fileoverview This is the global error page for the Next.js App Router.
 * It catches errors in a route segment and provides a user-friendly fallback UI.
 * This component is a Client Component.
 *
 * Learn more: https://nextjs.org/docs/app/building-your-application/routing/error-handling
 */

import { useEffect } from "react";
import { Button } from "@/components/ui/button"; // Assuming shadcn/ui button
import { AlertTriangle } from "lucide-react"; // Assuming lucide-react for icons
import { logError } from "@/lib/errors/logger"; // Import our error logger
import { handleError } from "@/lib/errors/handler"; // Import our error handler
import { AppError, InternalServerError } from "@/lib/errors/base";
import { ErrorCode } from "@/lib/errors/codes";

/**
 * The default error page component for Next.js App Router.
 * @param {object} props - The component props.
 * @param {Error & { digest?: string }} props.error - The error that was caught.
 * @param {() => void} props.reset - A function to attempt to recover from the error by re-rendering the segment.
 */
export default function ErrorPage({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  // Use our centralized error handler to normalize the error
  const appError: AppError = handleError(error);

  // Log the error to our logging service
  useEffect(() => {
    // Ensure the error is logged only once per error instance
    logError(appError, {
      location: "Next.js App Router Error Page",
      digest: error.digest, // Next.js specific error digest
    });
  }, [appError, error.digest]); // Depend on appError and digest to re-log if error changes

  // Determine the message to display to the user
  const displayMessage = appError.isOperational
    ? appError.message
    : "An unexpected error occurred. We're working to fix this.";

  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gray-50 text-gray-800 p-6 text-center">
      <AlertTriangle className="w-20 h-20 text-red-500 mb-8 animate-pulse" />
      <h1 className="text-4xl font-extrabold mb-4 text-gray-900">
        {appError.statusCode === 404 ? "Page Not Found" : "Something went wrong!"}
      </h1>
      <p className="text-xl text-gray-700 mb-8 max-w-md">
        {displayMessage}
      </p>
      <div className="flex flex-col sm:flex-row gap-4">
        <Button
          onClick={
            // Attempt to recover by trying to re-render the segment
            () => reset()
          }
          className="px-6 py-3 text-lg"
        >
          Try again
        </Button>
        <Button
          variant="outline"
          onClick={() => (window.location.href = "/")} // Navigate to home page
          className="px-6 py-3 text-lg"
        >
          Go home
        </Button>
      </div>
      {process.env.NODE_ENV === 'development' && (
        <details className="mt-10 p-4 bg-gray-100 rounded-lg w-full max-w-lg text-sm text-gray-700 text-left">
          <summary className="font-semibold cursor-pointer">Error Details (Development Only)</summary>
          <pre className="whitespace-pre-wrap break-words mt-2">
            <code>
              Name: {appError.name}<br />
              Code: {appError.code}<br />
              Status: {appError.statusCode}<br />
              Operational: {appError.isOperational ? 'Yes' : 'No'}<br />
              Original Message: {error.message}<br />
              Digest: {error.digest || 'N/A'}<br />
              Stack: {appError.stack}
            </code>
          </pre>
        </details>
      )}
    </div>
  );
}
```

---

## 7. src/app/not-found.tsx

```typescript
// src/app/not-found.tsx
/**
 * @fileoverview This is the custom 404 Not Found page for the Next.js App Router.
 * It is rendered when a route is not found.
 *
 * Learn more: https://nextjs.org/docs/app/api-reference/file-conventions/not-found
 */

import Link from "next/link";
import { FileQuestion } from "lucide-react"; // Assuming lucide-react for icons
import { Button } from "@/components/ui/button"; // Assuming shadcn/ui button

/**
 * The default 404 Not Found page component for Next.js App Router.
 */
export default function NotFoundPage() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen bg-gray-50 text-gray-800 p-6 text-center">
      <FileQuestion className="w-20 h-20 text-blue-500 mb-8 animate-bounce" />
      <h1 className="text-4xl font-extrabold mb-4 text-gray-900">404 - Page Not Found</h1>
      <p className="text-xl text-gray-700 mb-8 max-w-md">
        Oops! The page you're looking for doesn't exist or has been moved.
      </p>
      <Button asChild className="px-6 py-3 text-lg">
        <Link href="/">
          Go home
        </Link>
      </Button>
    </div>
  );
}
```

---

## 8. src/app/global-error.tsx

```typescript
// src/app/global-error.tsx
"use client";

/**
 * @fileoverview This is the root error boundary for the Next.js App Router.
 * It catches errors in the root layout and provides a minimal fallback UI.
 * This component must be a Client Component and must render `<html>` and `<body>` tags.
 *
 * Errors in `error.tsx` are caught by this `global-error.tsx`.
 *
 * Learn more: https://nextjs.org/docs/app/building-your-application/routing/error-handling#handling-errors-in-root-layouts
 */

import { useEffect } from "react";
import { Button } from "@/components/ui/button"; // Assuming shadcn/ui button
import { logError } from "@/lib/errors/logger"; // Import our error logger
import { handleError } from "@/lib/errors/handler"; // Import our error handler
import { AppError, InternalServerError } from "@/lib/errors/base";
import { ErrorCode } from "@/lib/errors/codes";

/**
 * The global error page component for Next.js App Router.
 * This catches errors that occur in the root layout or in `error.tsx` itself.
 * @param {object} props - The component props.
 * @param {Error & { digest?: string }} props.error - The error that was caught.
 * @param {() => void} props.reset - A function to attempt to recover from the error by re-rendering the segment.
 */
export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  // Use our centralized error handler to normalize the error
  const appError: AppError = handleError(error);

  // Log the error to our logging service
  useEffect(() => {
    logError(appError, {
      location: "Next.js App Router Global Error Page",
      digest: error.digest,
    });
  }, [appError, error.digest]);

  // Determine the message to display to the user
  const displayMessage = appError.isOperational
    ? appError.message
    : "An unexpected system error occurred. Please try again later.";

  return (
    <html>
      <body>
        <div className="flex flex-col items-center justify-center min-h-screen bg-red-50 text-red-800 p-6 text-center">
          <h1 className="text-4xl font-extrabold mb-4">Critical System Error!</h1>
          <p className="text-xl text-red-700 mb-8 max-w-md">
            {displayMessage}
          </p>
          <Button
            onClick={() => reset()} // Attempt to recover by re-rendering the root layout
            className="px-6 py-3 text-lg bg-red-600 hover:bg-red-700 text-white"
          >
            Reload Application
          </Button>
          {process.env.NODE_ENV === 'development' && (
            <details className="mt-10 p-4 bg-red-100 rounded-lg w-full max-w-lg text-sm text-red-700 text-left">
              <summary className="font-semibold cursor-pointer">Error Details (Development Only)</summary>
              <pre className="whitespace-pre-wrap break-words mt-2">
                <code>
                  Name: {appError.name}<br />
                  Code: {appError.code}<br />
                  Status: {appError.statusCode}<br />
                  Operational: {appError.isOperational ? 'Yes' : 'No'}<br />
                  Original Message: {error.message}<br />
                  Digest: {error.digest || 'N/A'}<br />
                  Stack: {appError.stack}
                </code>
              </pre>
            </details>
          )}
        </div>
      </body>
    </html>
  );
}
```

---

## 9. src/components/ui/error-alert.tsx

```typescript
// src/components/ui/error-alert.tsx
"use client";

/**
 * @fileoverview Reusable UI component for displaying error messages in an alert box.
 * It can display generic Error objects or structured AppError objects.
 * This component is designed to be dismissible and can optionally include a retry action.
 */

import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert"; // Assuming shadcn/ui alert components
import { Button } from "@/components/ui/button"; // Assuming shadcn/ui button
import { XCircle, RefreshCw } from "lucide-react"; // Assuming lucide-react for icons
import { AppError } from "@/lib/errors/base";
import { ErrorCode, errorMessages } from "@/lib/errors/codes";

interface ErrorAlertProps {
  /** The error object to display. Can be a native Error or an AppError. If null, the alert is not shown. */
  error: Error | AppError | null;
  /** Optional callback function to dismiss the alert. If provided, a dismiss button will be shown. */
  onDismiss?: () => void;
  /** Optional callback function to retry an action. If provided, a retry button will be shown. */
  onRetry?: () => void;
  /** Optional title for the alert. Defaults to "Error" or the AppError's name. */
  title?: string;
  /** Optional custom message to override the error's message. */
  message?: string;
}

/**
 * A reusable component to display error messages in a styled alert box.
 * It intelligently extracts information from AppError instances for richer display.
 */
export function ErrorAlert({ error, onDismiss, onRetry, title, message }: ErrorAlertProps) {
  if (!error) {
    return null;
  }

  let errorTitle: string;
  let errorMessage: string;
  let errorCode: string | undefined;

  if (error instanceof AppError) {
    errorTitle = title || error.name;
    errorMessage = message || (error.isOperational ? error.message : errorMessages[ErrorCode.INTERNAL_ERROR]);
    errorCode = error.code;
  } else {
    // For generic Error objects, provide a default title and message
    errorTitle = title || "Error";
    errorMessage = message || error.message || errorMessages[ErrorCode.UNKNOWN_ERROR];
    errorCode = undefined; // No specific code for generic errors
  }

  return (
    <Alert variant="destructive" className="flex items-start space-x-4 p-4">
      <XCircle className="h-5 w-5 mt-0.5 flex-shrink-0" />
      <div className="flex-grow">
        <AlertTitle className="text-lg font-semibold flex items-center gap-2">
          {errorTitle}
          {errorCode && <span className="text-sm font-normal text-red-200">({errorCode})</span>}
        </AlertTitle>
        <AlertDescription className="mt-1 text-red-100">
          {errorMessage}
        </AlertDescription>
        {(onRetry || onDismiss) && (
          <div className="flex gap-2 mt-4">
            {onRetry && (
              <Button onClick={onRetry} variant="secondary" size="sm" className="flex items-center gap-1">
                <RefreshCw className="h-4 w-4" /> Retry
              </Button>
            )}
            {onDismiss && (
              <Button onClick={onDismiss} variant="outline" size="sm">
                Dismiss
              </Button>
            )}
          </div>
        )}
      </div>
    </Alert>
  );
}
```

---

## 10. src/hooks/use-error-handler.ts

```typescript
// src/hooks/use-error-handler.ts
"use client";

/**
 * @fileoverview Provides a React hook for managing and handling errors within functional components.
 * This hook centralizes error state, processes raw errors into AppError instances,
 * and integrates with the application's logging mechanism.
 */

import { useState, useCallback } from "react";
import { AppError } from "@/lib/errors/base";
import { handleError as globalHandleError } from "@/lib/errors/handler"; // Renamed to avoid conflict
import { logError } from "@/lib/errors/logger";

/**
 * A custom React hook for handling errors in functional components.
 * It provides state to store the last encountered error, a function to process
 * and set errors, and a function to clear the error state.
 *
 * Usage:
 * const { error, handleError, clearError } = useErrorHandler();
 *
 * if (error) {
 *   return <ErrorAlert error={error} onDismiss={clearError} />;
 * }
 *
 * const fetchData = async () => {
 *   try {
 *     // ... fetch data
 *   } catch (err) {
 *     handleError(err); // Process and store the error
 *   }
 * };
 */
export function useErrorHandler() {
  // State to hold the AppError instance
  const [error, setError] = useState<AppError | null>(null);

  /**
   * Processes an unknown error, converts it into an AppError,
   * sets it in the component's state, and logs it.
   * This function is memoized using useCallback.
   * @param err The raw error (can be any type).
   * @param context Optional context to pass to the logger.
   */
  const handleError = useCallback((err: unknown, context?: object) => {
    // Use the global error handler to normalize the error into an AppError
    const appError = globalHandleError(err);
    setError(appError); // Set the error in the component's state
    logError(appError, context); // Log the error using the application's logger
  }, []); // No dependencies, so it's created once

  /**
   * Clears the current error state, effectively dismissing the error.
   * This function is memoized using useCallback.
   */
  const clearError = useCallback(() => setError(null), []); // No dependencies

  return { error, handleError, clearError };
}
```

---

## 11. tests/error-handling.test.ts

```typescript
// tests/error-handling.test.ts
/**
 * @fileoverview Comprehensive test suite for the error handling module.
 * This file contains unit tests for custom error classes, error code mappings,
 * the central error handler, and the error logger.
 */

import {
  AppError,
  ValidationError,
  AuthenticationError,
  NotFoundError,
  ConflictError,
  RateLimitError,
  ExternalServiceError,
  DatabaseError,
  InternalServerError,
} from "../src/lib/errors/base";
import { ErrorCode, errorCodeToStatus, errorMessages } from "../src/lib/errors/codes";
import { handleError } from "../src/lib/errors/handler";
import { logError, logWarning, logInfo, reportToSentry } from "../src/lib/errors/logger";
// Mock external types for testing handler.ts
import { PrismaClientKnownRequestError } from "@prisma/client/runtime/library";
import { ZodError, ZodIssue } from "zod"; // Assuming zod is installed for testing
import { TRPCError } from "@trpc/server"; // Assuming @trpc/server is installed for testing

// Mock console methods to spy on logs
const consoleErrorSpy = jest.spyOn(console, "error").mockImplementation(() => {});
const consoleWarnSpy = jest.spyOn(console, "warn").mockImplementation(() => {});
const consoleInfoSpy = jest.spyOn(console, "info").mockImplementation(() => {});

// Mock Sentry reporting
jest.mock("../src/lib/errors/logger", () => ({
  ...jest.requireActual("../src/lib/errors/logger"),
  reportToSentry: jest.fn(),
}));

describe("Error Handling Module", () => {
  beforeEach(() => {
    jest.clearAllMocks();
  });

  // --- AppError and Subclasses Tests ---
  describe("AppError and Subclasses", () => {
    it("AppError should be an instance of Error and AppError", () => {
      const error = new AppError("Test message", "TEST_CODE", 400);
      expect(error).toBeInstanceOf(Error);
      expect(error).toBeInstanceOf(AppError);
      expect(error.name).toBe("AppError");
      expect(error.message).toBe("Test message");
      expect(error.code).toBe("TEST_CODE");
      expect(error.statusCode).toBe(400);
      expect(error.isOperational).toBe(true);
    });

    it("AppError toJSON should return correct structure", () => {
      const error = new AppError("Test message", "TEST_CODE", 400, true, { extra: "data" });
      const json = error.toJSON();
      expect(json).toEqual({
        name: "AppError",
        message: "Test message",
        code: "TEST_CODE",
        statusCode: 400,
        details: { extra: "data" },
      });
      expect(json).not.toHaveProperty("stack"); // By default, stack is omitted
    });

    it("ValidationError should have correct defaults", () => {
      const error = new ValidationError("Invalid input", "INVALID_INPUT");
      expect(error).toBeInstanceOf(ValidationError);
      expect(error.statusCode).toBe(400);
      expect(error.code).toBe("INVALID_INPUT");
    });

    it("NotFoundError should have correct defaults", () => {
      const error = new NotFoundError("Resource not found");
      expect(error).toBeInstanceOf(NotFoundError);
      expect(error.statusCode).toBe(404);
      expect(error.code).toBe("NOT_FOUND");
    });

    it("InternalServerError should be non-operational by default", () => {
      const error = new InternalServerError();
      expect(error).toBeInstanceOf(InternalServerError);
      expect(error.statusCode).toBe(500);
      expect(error.code).toBe("INTERNAL_ERROR");
      expect(error.isOperational).toBe(false);
    });
  });

  // --- ErrorCode and Mappings Tests ---
  describe("ErrorCode and Mappings", () => {
    it("ErrorCode enum should contain expected values", () => {
      expect(ErrorCode.VALIDATION_ERROR).toBe("VALIDATION_ERROR");
      expect(ErrorCode.NOT_FOUND).toBe("NOT_FOUND");
      expect(ErrorCode.INTERNAL_ERROR).toBe("INTERNAL_ERROR");
    });

    it("errorCodeToStatus should map codes to correct HTTP statuses", () => {
      expect(errorCodeToStatus[ErrorCode.INVALID_INPUT]).toBe(400);
      expect(errorCodeToStatus[ErrorCode.UNAUTHENTICATED]).toBe(401);
      expect(errorCodeToStatus[ErrorCode.FORBIDDEN]).toBe(403);
      expect(errorCodeToStatus[ErrorCode.RESOURCE_NOT_FOUND]).toBe(404);
      expect(errorCodeToStatus[ErrorCode.DUPLICATE_ENTRY]).toBe(409);
      expect(errorCodeToStatus[ErrorCode.RATE_LIMIT_EXCEEDED]).toBe(429);
      expect(errorCodeToStatus[ErrorCode.INTERNAL_ERROR]).toBe(500);
      expect(errorCodeToStatus[ErrorCode.EXTERNAL_SERVICE_ERROR]).toBe(502);
    });

    it("errorMessages should provide user-friendly messages", () => {
      expect(errorMessages[ErrorCode.INVALID_INPUT]).toBe("The provided input is invalid.");
      expect(errorMessages[ErrorCode.NOT_FOUND]).toBe("The requested resource could not be found.");
      expect(errorMessages[ErrorCode.INTERNAL_ERROR]).toBe("An internal server error occurred. We are working to fix it.");
    });
  });

  // --- handleError Tests ---
  describe("handleError", () => {
    it("should return AppError if already an AppError", () => {
      const originalError = new AuthenticationError("Invalid token");
      const handledError = handleError(originalError);
      expect(handledError).toBe(originalError);
    });

    it("should convert generic Error to InternalServerError", () => {
      const genericError = new Error("Something unexpected happened");
      const handledError = handleError(genericError);
      expect(handledError).toBeInstanceOf(InternalServerError);
      expect(handledError.message).toBe(errorMessages[ErrorCode.INTERNAL_ERROR]);
      expect(handledError.code).toBe(ErrorCode.INTERNAL_ERROR);
      expect(handledError.statusCode).toBe(500);
      expect(handledError.isOperational).toBe(false);
      expect(handledError.details).toHaveProperty("originalMessage", genericError.message);
    });

    it("should convert unknown types to InternalServerError", () => {
      const unknownError = "a string error";
      const handledError = handleError(unknownError);
      expect(handledError).toBeInstanceOf(InternalServerError);
      expect(handledError.message).toBe(errorMessages[ErrorCode.UNKNOWN_ERROR]);
      expect(handledError.code).toBe(ErrorCode.UNKNOWN_ERROR);
      expect(handledError.statusCode).toBe(500);
      expect(handledError.isOperational).toBe(false);
      expect(handledError.details).toHaveProperty("originalError", unknownError);
    });

    it("should handle PrismaClientKnownRequestError (P2002 - unique constraint)", () => {
      const prismaError = new PrismaClientKnownRequestError(
        "Unique constraint failed on the fields: (`email`)",
        { code: "P2002", clientVersion: "2.x", meta: { target: ["email"] } }
      );
      const handledError = handleError(prismaError);
      expect(handledError).toBeInstanceOf(ConflictError);
      expect(handledError.code).toBe(ErrorCode.DB_UNIQUE_CONSTRAINT_VIOLATION);
      expect(handledError.statusCode).toBe(409);
      expect(handledError.message).toContain("email already exists");
      expect(handledError.details).toHaveProperty("target", "email");
    });

    it("should handle PrismaClientKnownRequestError (P2025 - record not found)", () => {
      const prismaError = new PrismaClientKnownRequestError(
        "An operation failed because it depends on one or more records that were required but not found. Record to update not found.",
        { code: "P2025", clientVersion: "2.x", meta: { modelName: "User" } }
      );
      const handledError = handleError(prismaError);
      expect(handledError).toBeInstanceOf(NotFoundError);
      expect(handledError.code).toBe(ErrorCode.DB_RECORD_NOT_FOUND);
      expect(handledError.statusCode).toBe(404);
      expect(handledError.message).toContain("User could not be found");
    });

    it("should handle ZodError", () => {
      const zodError = new ZodError([
        { code: "invalid_type", expected: "string", received: "number", path: ["name"], message: "Expected string, received number" },
        { code: "too_small", minimum: 5, type: "string", inclusive: true, exact: false, path: ["password"], message: "String must contain at least 5 character(s)" },
      ]);
      const handledError = handleError(zodError);
      expect(handledError).toBeInstanceOf(ValidationError);
      expect(handledError.code).toBe(ErrorCode.VALIDATION_ERROR);
      expect(handledError.statusCode).toBe(400);
      expect(handledError.message).toBe(errorMessages[ErrorCode.VALIDATION_ERROR]);
      expect(handledError.details).toEqual([
        { path: "name", message: "Expected string, received number", code: "invalid_type" },
        { path: "password", message: "String must contain at least 5 character(s)", code: "too_small" },
      ]);
    });

    it("should handle TRPCError (BAD_REQUEST)", () => {
      const trpcError = new TRPCError({ code: "BAD_REQUEST", message: "Invalid input for procedure" });
      const handledError = handleError(trpcError);
      expect(handledError).toBeInstanceOf(ValidationError);
      expect(handledError.code).toBe(ErrorCode.INVALID_INPUT);
      expect(handledError.statusCode).toBe(400);
      expect(handledError.message).toBe("Invalid input for procedure");
      expect(handledError.details).toHaveProperty("originalTRPCCode", "BAD_REQUEST");
    });

    it("should handle TRPCError (UNAUTHORIZED)", () => {
      const trpcError = new TRPCError({ code: "UNAUTHORIZED", message: "Authentication failed" });
      const handledError = handleError(trpcError);
      expect(handledError).toBeInstanceOf(AuthenticationError);
      expect(handledError.code).toBe(ErrorCode.UNAUTHENTICATED);
      expect(handledError.statusCode).toBe(401);
      expect(handledError.message).toBe("Authentication failed");
    });

    it("should handle TRPCError (INTERNAL_SERVER_ERROR)", () => {
      const trpcError = new TRPCError({ code: "INTERNAL_SERVER_ERROR", message: "Server crashed" });
      const handledError = handleError(trpcError);
      expect(handledError).toBeInstanceOf(InternalServerError);
      expect(handledError.code).toBe(ErrorCode.INTERNAL_ERROR);
      expect(handledError.statusCode).toBe(500);
      expect(handledError.message).toBe("Server crashed");
    });

    it("should handle network connection refused error", () => {
      const networkError = new Error("connect ECONNREFUSED 127.0.0.1:8080");
      (networkError as NodeJS.ErrnoException).code = 'ECONNREFUSED';
      const handledError = handleError(networkError);
      expect(handledError).toBeInstanceOf(ExternalServiceError);
      expect(handledError.code).toBe(ErrorCode.NETWORK_ERROR);
      expect(handledError.statusCode).toBe(errorCodeToStatus[ErrorCode.NETWORK_ERROR]);
      expect(handledError.message).toBe(errorMessages[ErrorCode.NETWORK_ERROR]);
      expect(handledError.isOperational).toBe(false);
    });

    it("should handle timeout error", () => {
      const timeoutError = new Error("The operation timed out.");
      timeoutError.name = 'AbortError'; // Common for fetch timeouts
      const handledError = handleError(timeoutError);
      expect(handledError).toBeInstanceOf(ExternalServiceError);
      expect(handledError.code).toBe(ErrorCode.TIMEOUT_ERROR);
      expect(handledError.statusCode).toBe(errorCodeToStatus[ErrorCode.TIMEOUT_ERROR]);
      expect(handledError.message).toBe(errorMessages[ErrorCode.TIMEOUT_ERROR]);
      expect(handledError.isOperational).toBe(false);
    });
  });

  // --- Logger Tests ---
  describe("Logger", () => {
    it("logError should log AppError to console.error and not report operational errors", () => {
      const error = new ValidationError("Invalid field", "INVALID_FIELD");
      logError(error, { userId: "123" });
      expect(consoleErrorSpy).toHaveBeenCalledTimes(1);
      const loggedOutput = JSON.parse(consoleErrorSpy.mock.calls[0][1]);
      expect(loggedOutput.level).toBe("error");
      expect(loggedOutput.error.code).toBe("INVALID_FIELD");
      expect(loggedOutput.context.userId).toBe("123");
      expect(reportToSentry).not.toHaveBeenCalled(); // Operational error
    });

    it("logError should log generic Error to console.error and report non-operational errors", () => {
      const genericError = new Error("Critical bug!");
      logError(genericError, { path: "/api/data" });
      expect(consoleErrorSpy).toHaveBeenCalledTimes(1);
      const loggedOutput = JSON.parse(consoleErrorSpy.mock.calls[0][1]);
      expect(loggedOutput.level).toBe("error");
      expect(loggedOutput.error.code).toBe(ErrorCode.INTERNAL_ERROR); // Wrapped as InternalServerError
      expect(loggedOutput.error.isOperational).toBe(false);
      expect(loggedOutput.context.path).toBe("/api/data");
      expect(reportToSentry).toHaveBeenCalledTimes(1); // Non-operational error
      expect(reportToSentry).toHaveBeenCalledWith(expect.any(InternalServerError), { path: "/api/data" });
    });

    it("logWarning should log to console.warn", () => {
      logWarning("Something might be wrong", { threshold: 0.8 });
      expect(consoleWarnSpy).toHaveBeenCalledTimes(1);
      const loggedOutput = JSON.parse(consoleWarnSpy.mock.calls[0][1]);
      expect(loggedOutput.level).toBe("warn");
      expect(loggedOutput.message).toBe("Something might be wrong");
      expect(loggedOutput.details).toEqual({ threshold: 0.8 });
    });

    it("logInfo should log to console.info", () => {
      logInfo("User logged in", { userId: "456" });
      expect(consoleInfoSpy).toHaveBeenCalledTimes(1);
      const loggedOutput = JSON.parse(consoleInfoSpy.mock.calls[0][1]);
      expect(loggedOutput.level).toBe("info");
      expect(loggedOutput.message).toBe("User logged in");
      expect(loggedOutput.context.userId).toBe("456");
    });

    it("reportToSentry should be called for non-operational AppErrors in production", () => {
      process.env.NODE_ENV = "production";
      process.env.NEXT_PUBLIC_SENTRY_DSN = "http://example.com"; // Simulate DSN being set
      const error = new DatabaseError("DB connection failed");
      logError(error);
      expect(reportToSentry).toHaveBeenCalledTimes(1);
      expect(reportToSentry).toHaveBeenCalledWith(error, undefined);
      delete process.env.NEXT_PUBLIC_SENTRY_DSN;
      process.env.NODE_ENV = "test"; // Reset for other tests
    });

    it("reportToSentry should not be called for operational AppErrors", () => {
      const error = new NotFoundError("User not found"); // Operational
      logError(error);
      expect(reportToSentry).not.toHaveBeenCalled();
    });
  });
});
```

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 22272_