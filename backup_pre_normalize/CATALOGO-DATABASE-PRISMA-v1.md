# CATALOGO-DATABASE-PRISMA-v1

> **Dominio**: Utilities database e gestione Prisma Client
> **Stack**: Prisma ORM, PostgreSQL, TypeScript
> **Versione**: 1.0
> **Data**: 2026-01-29

---

## 1. INDICE

| Sezione | Descrizione |
|---------|-------------|
| [1. Panoramica](#1-panoramica) | Architettura utilities database |
| [2. Prisma Client](#2-prisma-client) | Singleton e configurazione |
| [3. Pagination](#3-pagination) | Offset e cursor-based pagination |
| [4. Transaction Helpers](#4-transaction-helpers) | Gestione transazioni |
| [5. Query Builders](#5-query-builders) | Builder per filtri e ordinamento |
| [6. Error Handling](#6-error-handling) | Gestione errori Prisma |
| [7. Soft Delete](#7-soft-delete) | Pattern soft delete |
| [8. Audit Log](#8-audit-log) | Logging modifiche database |

---

## 1. PANORAMICA

### 1.1 Obiettivo
Utilities TypeScript per la gestione avanzata del database con Prisma ORM, incluse paginazione, transazioni, query builders e error handling.

### 1.2 Struttura File

| File | Path | Descrizione |
|------|------|-------------|
| Client | `src/lib/db/client.ts` | Prisma client singleton |
| Pagination | `src/lib/db/pagination.ts` | Utilities paginazione |
| Transactions | `src/lib/db/transactions.ts` | Helper transazioni |
| Query Builders | `src/lib/db/query-builders.ts` | Filtri e ordinamento |
| Error Handling | `src/lib/db/errors.ts` | Gestione errori DB |
| Soft Delete | `src/lib/db/soft-delete.ts` | Pattern soft delete |
| Audit Log | `src/lib/db/audit.ts` | Logging modifiche |
| Index | `src/lib/db/index.ts` | Barrel export |

### 1.3 Funzionalità Principali

| Funzionalità | Descrizione |
|--------------|-------------|
| **Singleton Client** | Istanza unica Prisma con logging configurabile |
| **Offset Pagination** | Paginazione classica con page/pageSize |
| **Cursor Pagination** | Paginazione efficiente per grandi dataset |
| **Transactions** | Helper per transazioni con retry |
| **Dynamic Filters** | Costruzione dinamica di where clause |
| **Error Mapping** | Conversione errori Prisma in AppError |
| **Soft Delete** | Eliminazione logica con deletedAt |
| **Audit Trail** | Logging automatico modifiche |

---

## 2. PRISMA CLIENT

### 2.1 File: `src/lib/db/client.ts`

| Export | Tipo | Descrizione |
|--------|------|-------------|
| `prisma` | PrismaClient | Singleton instance |
| `PrismaClientType` | Type | Type del client |

```typescript
// src/lib/db/client.ts
import { PrismaClient } from '@prisma/client';

// Add PrismaClient to the NodeJS global type to prevent multiple instances in development
declare global {
  // eslint-disable-next-line no-var
  var prisma: PrismaClient | undefined;
}

/**
 * Global PrismaClient instance.
 * This ensures that in development, a single PrismaClient instance is used across hot reloads,
 * preventing issues like too many database connections.
 * In production, a new instance is created if one doesn't exist.
 */
export const prisma: PrismaClient =
  globalThis.prisma ||
  new PrismaClient({
    log:
      process.env.NODE_ENV === 'development'
        ? ['query', 'error', 'warn']
        : ['error'],
  });

// In development, store the PrismaClient instance on globalThis
if (process.env.NODE_ENV !== 'production') {
  globalThis.prisma = prisma;
}

// Export the PrismaClient type for convenience
export type PrismaClientType = typeof prisma;
```

---

## 3. PAGINATION

### 3.1 Types

| Type | Descrizione |
|------|-------------|
| `PaginationParams` | Input paginazione |
| `PaginatedResult<T>` | Output paginato |
| `CursorPaginatedResult<T>` | Output cursor-based |

### 3.2 Costanti

| Costante | Valore | Descrizione |
|----------|--------|-------------|
| `DEFAULT_PAGE_SIZE` | 10 | Items per pagina default |
| `MAX_PAGE_SIZE` | 100 | Limite massimo items |

### 3.3 Funzioni

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `getOffsetPaginationArgs` | `PaginationParams` | `{ skip, take }` | Calcola skip/take |
| `createPaginatedResult` | `data, total, params` | `PaginatedResult<T>` | Crea risultato |
| `getCursorPaginationArgs` | `cursor?, pageSize?` | `{ cursor?, take, skip? }` | Args cursor |
| `createCursorPaginatedResult` | `data, pageSize, idField` | `CursorPaginatedResult<T>` | Crea risultato cursor |

### 3.4 File: `src/lib/db/pagination.ts`

```typescript
// src/lib/db/pagination.ts
import { Prisma } from '@prisma/client';

export interface PaginationParams {
  page?: number;
  pageSize?: number;
  cursor?: string;
}

export interface PaginatedResult<T> {
  data: T[];
  pagination: {
    page: number;
    pageSize: number;
    total: number;
    totalPages: number;
    hasMore: boolean;
    nextCursor?: string;
  };
}

export interface CursorPaginatedResult<T> {
  data: T[];
  pagination: {
    pageSize: number;
    hasMore: boolean;
    nextCursor: string | null;
  };
}

const DEFAULT_PAGE_SIZE = 10;
const MAX_PAGE_SIZE = 100;

/**
 * Calculates offset pagination arguments (skip, take) based on page and pageSize.
 */
export function getOffsetPaginationArgs(params: PaginationParams): {
  skip: number;
  take: number;
} {
  const page = Math.max(1, params.page || 1);
  const pageSize = Math.min(MAX_PAGE_SIZE, Math.max(1, params.pageSize || DEFAULT_PAGE_SIZE));
  const skip = (page - 1) * pageSize;
  return { skip, take: pageSize };
}

/**
 * Creates a standardized paginated result object.
 */
export function createPaginatedResult<T>(
  data: T[],
  total: number,
  params: PaginationParams
): PaginatedResult<T> {
  const page = Math.max(1, params.page || 1);
  const pageSize = Math.min(MAX_PAGE_SIZE, Math.max(1, params.pageSize || DEFAULT_PAGE_SIZE));
  const totalPages = Math.ceil(total / pageSize);

  return {
    data,
    pagination: {
      page,
      pageSize,
      total,
      totalPages,
      hasMore: page < totalPages,
    },
  };
}

/**
 * Calculates cursor-based pagination arguments.
 */
export function getCursorPaginationArgs<CursorType extends Record<string, unknown>>(
  cursor: string | undefined,
  pageSize: number = DEFAULT_PAGE_SIZE,
  cursorField: keyof CursorType = 'id' as keyof CursorType
): {
  cursor?: { [K in keyof CursorType]?: CursorType[K] };
  take: number;
  skip?: number;
} {
  const take = Math.min(MAX_PAGE_SIZE, Math.max(1, pageSize));
  if (cursor) {
    return {
      cursor: { [cursorField]: cursor } as { [K in keyof CursorType]?: CursorType[K] },
      take: take + 1,
      skip: 1,
    };
  }
  return { take: take + 1 };
}

/**
 * Creates a cursor-based paginated result.
 */
export function createCursorPaginatedResult<T extends Record<string, unknown>>(
  data: T[],
  pageSize: number = DEFAULT_PAGE_SIZE,
  idField: keyof T = 'id' as keyof T
): CursorPaginatedResult<T> {
  const actualPageSize = Math.min(MAX_PAGE_SIZE, Math.max(1, pageSize));
  const hasMore = data.length > actualPageSize;
  const items = hasMore ? data.slice(0, actualPageSize) : data;
  const nextCursor = hasMore && items.length > 0
    ? String(items[items.length - 1][idField])
    : null;

  return {
    data: items,
    pagination: {
      pageSize: actualPageSize,
      hasMore,
      nextCursor,
    },
  };
}
```

---

## 4. TRANSACTION HELPERS

### 4.1 Funzioni

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `executeTransaction` | `callback, options?` | `Promise<T>` | Esegue transazione |
| `executeWithRetry` | `callback, maxRetries?` | `Promise<T>` | Con retry automatico |

### 4.2 Opzioni Transazione

| Opzione | Tipo | Default | Descrizione |
|---------|------|---------|-------------|
| `maxWait` | number | 5000 | Max wait ms |
| `timeout` | number | 10000 | Timeout ms |
| `isolationLevel` | enum | ReadCommitted | Isolation level |

### 4.3 File: `src/lib/db/transactions.ts`

```typescript
// src/lib/db/transactions.ts
import { Prisma } from '@prisma/client';
import { prisma } from './client';

export interface TransactionOptions {
  maxWait?: number;
  timeout?: number;
  isolationLevel?: Prisma.TransactionIsolationLevel;
}

const DEFAULT_TRANSACTION_OPTIONS: TransactionOptions = {
  maxWait: 5000,
  timeout: 10000,
  isolationLevel: Prisma.TransactionIsolationLevel.ReadCommitted,
};

/**
 * Executes a callback within a Prisma interactive transaction.
 */
export async function executeTransaction<T>(
  callback: (tx: Prisma.TransactionClient) => Promise<T>,
  options: TransactionOptions = {}
): Promise<T> {
  const mergedOptions = { ...DEFAULT_TRANSACTION_OPTIONS, ...options };

  return prisma.$transaction(callback, {
    maxWait: mergedOptions.maxWait,
    timeout: mergedOptions.timeout,
    isolationLevel: mergedOptions.isolationLevel,
  });
}

/**
 * Executes an operation with automatic retry on transient failures.
 */
export async function executeWithRetry<T>(
  callback: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 100
): Promise<T> {
  let lastError: Error | undefined;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await callback();
    } catch (error) {
      lastError = error as Error;

      // Check if error is retryable (deadlock, connection issues)
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        const retryableCodes = ['P2034', 'P1001', 'P1002', 'P1008', 'P1017'];
        if (retryableCodes.includes(error.code) && attempt < maxRetries) {
          await new Promise((resolve) => setTimeout(resolve, delayMs * attempt));
          continue;
        }
      }
      throw error;
    }
  }

  throw lastError;
}

/**
 * Batch operations helper for large datasets.
 */
export async function batchOperation<T, R>(
  items: T[],
  operation: (batch: T[], tx: Prisma.TransactionClient) => Promise<R[]>,
  batchSize: number = 100
): Promise<R[]> {
  const results: R[] = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await executeTransaction((tx) => operation(batch, tx));
    results.push(...batchResults);
  }

  return results;
}
```

---

## 5. QUERY BUILDERS

### 5.1 Types

| Type | Descrizione |
|------|-------------|
| `FilterOperator` | Operatori filtro (eq, ne, gt, lt, contains, etc.) |
| `FilterCondition` | Singola condizione filtro |
| `SortDirection` | asc / desc |
| `SortCondition` | Campo + direzione |

### 5.2 Funzioni

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `buildWhereClause` | `FilterCondition[]` | `Prisma.XWhereInput` | Costruisce where |
| `buildOrderByClause` | `SortCondition[]` | `Prisma.XOrderByInput` | Costruisce orderBy |
| `buildSearchClause` | `query, fields[]` | `Prisma.XWhereInput` | Full-text search |

### 5.3 File: `src/lib/db/query-builders.ts`

```typescript
// src/lib/db/query-builders.ts
import { Prisma } from '@prisma/client';

export type FilterOperator =
  | 'eq'
  | 'ne'
  | 'gt'
  | 'gte'
  | 'lt'
  | 'lte'
  | 'contains'
  | 'startsWith'
  | 'endsWith'
  | 'in'
  | 'notIn'
  | 'isNull'
  | 'isNotNull';

export interface FilterCondition {
  field: string;
  operator: FilterOperator;
  value: unknown;
}

export type SortDirection = 'asc' | 'desc';

export interface SortCondition {
  field: string;
  direction: SortDirection;
}

/**
 * Builds a Prisma where clause from filter conditions.
 */
export function buildWhereClause<T extends Record<string, unknown>>(
  filters: FilterCondition[]
): T {
  const where: Record<string, unknown> = {};

  for (const filter of filters) {
    const { field, operator, value } = filter;

    switch (operator) {
      case 'eq':
        where[field] = value;
        break;
      case 'ne':
        where[field] = { not: value };
        break;
      case 'gt':
        where[field] = { gt: value };
        break;
      case 'gte':
        where[field] = { gte: value };
        break;
      case 'lt':
        where[field] = { lt: value };
        break;
      case 'lte':
        where[field] = { lte: value };
        break;
      case 'contains':
        where[field] = { contains: value, mode: 'insensitive' };
        break;
      case 'startsWith':
        where[field] = { startsWith: value, mode: 'insensitive' };
        break;
      case 'endsWith':
        where[field] = { endsWith: value, mode: 'insensitive' };
        break;
      case 'in':
        where[field] = { in: value as unknown[] };
        break;
      case 'notIn':
        where[field] = { notIn: value as unknown[] };
        break;
      case 'isNull':
        where[field] = null;
        break;
      case 'isNotNull':
        where[field] = { not: null };
        break;
    }
  }

  return where as T;
}

/**
 * Builds a Prisma orderBy clause from sort conditions.
 */
export function buildOrderByClause<T extends Record<string, SortDirection>>(
  sorts: SortCondition[]
): T[] {
  return sorts.map((sort) => ({
    [sort.field]: sort.direction,
  })) as T[];
}

/**
 * Builds a search clause for multiple fields.
 */
export function buildSearchClause<T extends Record<string, unknown>>(
  query: string,
  fields: string[]
): T {
  if (!query.trim()) {
    return {} as T;
  }

  return {
    OR: fields.map((field) => ({
      [field]: { contains: query, mode: 'insensitive' },
    })),
  } as T;
}

/**
 * Combines multiple where clauses with AND logic.
 */
export function combineWhereClauses<T extends Record<string, unknown>>(
  ...clauses: (T | undefined | null)[]
): T {
  const validClauses = clauses.filter(
    (clause): clause is T => clause != null && Object.keys(clause).length > 0
  );

  if (validClauses.length === 0) return {} as T;
  if (validClauses.length === 1) return validClauses[0];

  return { AND: validClauses } as T;
}
```

---

## 6. ERROR HANDLING

### 6.1 Error Codes Prisma

| Code | Descrizione | HTTP Status |
|------|-------------|-------------|
| `P2000` | Value too long | 400 |
| `P2001` | Record not found | 404 |
| `P2002` | Unique constraint violation | 409 |
| `P2003` | Foreign key constraint | 400 |
| `P2025` | Record not found (relation) | 404 |

### 6.2 Funzioni

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `handlePrismaError` | `error` | `AppError` | Converte errore |
| `isRecordNotFound` | `error` | `boolean` | Check not found |
| `isUniqueConstraintViolation` | `error` | `boolean` | Check unique |

### 6.3 File: `src/lib/db/errors.ts`

```typescript
// src/lib/db/errors.ts
import { Prisma } from '@prisma/client';

export class DatabaseError extends Error {
  public readonly code: string;
  public readonly statusCode: number;
  public readonly isOperational: boolean;

  constructor(message: string, code: string, statusCode: number = 500) {
    super(message);
    this.name = 'DatabaseError';
    this.code = code;
    this.statusCode = statusCode;
    this.isOperational = true;
  }
}

const PRISMA_ERROR_MAP: Record<string, { message: string; statusCode: number }> = {
  P2000: { message: 'Input value is too long', statusCode: 400 },
  P2001: { message: 'Record not found', statusCode: 404 },
  P2002: { message: 'Unique constraint violation', statusCode: 409 },
  P2003: { message: 'Foreign key constraint failed', statusCode: 400 },
  P2025: { message: 'Record not found', statusCode: 404 },
  P2014: { message: 'Relation violation', statusCode: 400 },
  P2016: { message: 'Query interpretation error', statusCode: 400 },
  P2034: { message: 'Transaction conflict, please retry', statusCode: 409 },
};

/**
 * Handles Prisma errors and converts them to DatabaseError.
 */
export function handlePrismaError(error: unknown): DatabaseError {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    const mapping = PRISMA_ERROR_MAP[error.code];
    if (mapping) {
      // Extract field name for unique constraint violations
      if (error.code === 'P2002') {
        const target = (error.meta?.target as string[]) || [];
        const fieldName = target.join(', ');
        return new DatabaseError(
          `${mapping.message}: ${fieldName}`,
          error.code,
          mapping.statusCode
        );
      }
      return new DatabaseError(mapping.message, error.code, mapping.statusCode);
    }
    return new DatabaseError(`Database error: ${error.message}`, error.code, 500);
  }

  if (error instanceof Prisma.PrismaClientValidationError) {
    return new DatabaseError('Invalid data provided', 'VALIDATION_ERROR', 400);
  }

  if (error instanceof Prisma.PrismaClientInitializationError) {
    return new DatabaseError('Database connection failed', 'CONNECTION_ERROR', 503);
  }

  if (error instanceof Error) {
    return new DatabaseError(error.message, 'UNKNOWN_ERROR', 500);
  }

  return new DatabaseError('An unknown database error occurred', 'UNKNOWN_ERROR', 500);
}

/**
 * Checks if the error is a "record not found" error.
 */
export function isRecordNotFound(error: unknown): boolean {
  return (
    error instanceof Prisma.PrismaClientKnownRequestError &&
    (error.code === 'P2001' || error.code === 'P2025')
  );
}

/**
 * Checks if the error is a unique constraint violation.
 */
export function isUniqueConstraintViolation(error: unknown): boolean {
  return (
    error instanceof Prisma.PrismaClientKnownRequestError && error.code === 'P2002'
  );
}

/**
 * Checks if the error is a foreign key constraint violation.
 */
export function isForeignKeyViolation(error: unknown): boolean {
  return (
    error instanceof Prisma.PrismaClientKnownRequestError && error.code === 'P2003'
  );
}
```

---

## 7. SOFT DELETE

### 7.1 Pattern

| Campo | Tipo | Descrizione |
|-------|------|-------------|
| `deletedAt` | DateTime? | Timestamp eliminazione |
| `isDeleted` | Boolean | Flag computed (opzionale) |

### 7.2 Middleware

| Middleware | Azione | Descrizione |
|------------|--------|-------------|
| `findMany` | Filtra | Esclude deletedAt != null |
| `findFirst` | Filtra | Esclude deletedAt != null |
| `findUnique` | Filtra | Esclude deletedAt != null |
| `delete` | Modifica | Converte in update deletedAt |
| `deleteMany` | Modifica | Converte in updateMany |

### 7.3 File: `src/lib/db/soft-delete.ts`

```typescript
// src/lib/db/soft-delete.ts
import { Prisma, PrismaClient } from '@prisma/client';

// Models that support soft delete
const SOFT_DELETE_MODELS = ['User', 'Post', 'Comment', 'Product', 'Order'];

/**
 * Applies soft delete middleware to Prisma client.
 */
export function applySoftDeleteMiddleware(prisma: PrismaClient): void {
  prisma.$use(async (params, next) => {
    // Check if the model supports soft delete
    if (!params.model || !SOFT_DELETE_MODELS.includes(params.model)) {
      return next(params);
    }

    // Convert delete to soft delete
    if (params.action === 'delete') {
      params.action = 'update';
      params.args['data'] = { deletedAt: new Date() };
    }

    if (params.action === 'deleteMany') {
      params.action = 'updateMany';
      if (params.args.data !== undefined) {
        params.args.data['deletedAt'] = new Date();
      } else {
        params.args['data'] = { deletedAt: new Date() };
      }
    }

    // Filter out soft-deleted records in find operations
    if (params.action === 'findFirst' || params.action === 'findUnique') {
      params.action = 'findFirst';
      params.args.where = { ...params.args.where, deletedAt: null };
    }

    if (params.action === 'findMany') {
      if (params.args.where) {
        if (params.args.where.deletedAt === undefined) {
          params.args.where.deletedAt = null;
        }
      } else {
        params.args['where'] = { deletedAt: null };
      }
    }

    return next(params);
  });
}

/**
 * Hard deletes a record (permanent deletion).
 */
export async function hardDelete<T>(
  prisma: PrismaClient,
  model: string,
  where: Record<string, unknown>
): Promise<T> {
  // @ts-ignore - Dynamic model access
  return prisma[model.toLowerCase()].delete({ where });
}

/**
 * Restores a soft-deleted record.
 */
export async function restoreSoftDeleted<T>(
  prisma: PrismaClient,
  model: string,
  where: Record<string, unknown>
): Promise<T> {
  // @ts-ignore - Dynamic model access
  return prisma[model.toLowerCase()].update({
    where,
    data: { deletedAt: null },
  });
}

/**
 * Finds records including soft-deleted ones.
 */
export async function findIncludingDeleted<T>(
  prisma: PrismaClient,
  model: string,
  args: Record<string, unknown>
): Promise<T[]> {
  const argsWithDeleted = {
    ...args,
    where: {
      ...(args.where as Record<string, unknown>),
      // Explicitly include deleted records by not filtering
    },
  };
  // Remove deletedAt filter if present
  delete (argsWithDeleted.where as Record<string, unknown>).deletedAt;

  // @ts-ignore - Dynamic model access
  return prisma[model.toLowerCase()].findMany(argsWithDeleted);
}
```

---

## 8. AUDIT LOG

### 8.1 Schema Audit

| Campo | Tipo | Descrizione |
|-------|------|-------------|
| `id` | String | Primary key |
| `tableName` | String | Tabella modificata |
| `recordId` | String | ID record |
| `action` | Enum | CREATE/UPDATE/DELETE |
| `oldData` | Json? | Dati precedenti |
| `newData` | Json? | Dati nuovi |
| `userId` | String? | Utente che ha fatto la modifica |
| `timestamp` | DateTime | Quando |

### 8.2 Funzioni

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `logAudit` | `params` | `Promise<void>` | Crea log entry |
| `applyAuditMiddleware` | `prisma, getUserId` | `void` | Applica middleware |

### 8.3 File: `src/lib/db/audit.ts`

```typescript
// src/lib/db/audit.ts
import { Prisma, PrismaClient } from '@prisma/client';

export type AuditAction = 'CREATE' | 'UPDATE' | 'DELETE';

export interface AuditLogEntry {
  tableName: string;
  recordId: string;
  action: AuditAction;
  oldData?: Record<string, unknown>;
  newData?: Record<string, unknown>;
  userId?: string;
  timestamp: Date;
}

// Models to audit
const AUDITED_MODELS = ['User', 'Post', 'Order', 'Product'];

/**
 * Logs an audit entry to the database.
 */
export async function logAudit(
  prisma: PrismaClient,
  entry: AuditLogEntry
): Promise<void> {
  await prisma.auditLog.create({
    data: {
      tableName: entry.tableName,
      recordId: entry.recordId,
      action: entry.action,
      oldData: entry.oldData || Prisma.JsonNull,
      newData: entry.newData || Prisma.JsonNull,
      userId: entry.userId,
      timestamp: entry.timestamp,
    },
  });
}

/**
 * Applies audit middleware to Prisma client.
 */
export function applyAuditMiddleware(
  prisma: PrismaClient,
  getUserId: () => string | undefined
): void {
  prisma.$use(async (params, next) => {
    if (!params.model || !AUDITED_MODELS.includes(params.model)) {
      return next(params);
    }

    const userId = getUserId();
    const timestamp = new Date();

    // Handle CREATE
    if (params.action === 'create') {
      const result = await next(params);
      await logAudit(prisma, {
        tableName: params.model,
        recordId: result.id,
        action: 'CREATE',
        newData: result as Record<string, unknown>,
        userId,
        timestamp,
      });
      return result;
    }

    // Handle UPDATE
    if (params.action === 'update') {
      // Get old data first
      // @ts-ignore
      const oldData = await prisma[params.model.toLowerCase()].findUnique({
        where: params.args.where,
      });

      const result = await next(params);

      await logAudit(prisma, {
        tableName: params.model,
        recordId: result.id,
        action: 'UPDATE',
        oldData: oldData as Record<string, unknown>,
        newData: result as Record<string, unknown>,
        userId,
        timestamp,
      });
      return result;
    }

    // Handle DELETE
    if (params.action === 'delete') {
      // @ts-ignore
      const oldData = await prisma[params.model.toLowerCase()].findUnique({
        where: params.args.where,
      });

      const result = await next(params);

      await logAudit(prisma, {
        tableName: params.model,
        recordId: oldData?.id || 'unknown',
        action: 'DELETE',
        oldData: oldData as Record<string, unknown>,
        userId,
        timestamp,
      });
      return result;
    }

    return next(params);
  });
}

/**
 * Gets audit history for a specific record.
 */
export async function getAuditHistory(
  prisma: PrismaClient,
  tableName: string,
  recordId: string
): Promise<AuditLogEntry[]> {
  const logs = await prisma.auditLog.findMany({
    where: { tableName, recordId },
    orderBy: { timestamp: 'desc' },
  });

  return logs.map((log) => ({
    tableName: log.tableName,
    recordId: log.recordId,
    action: log.action as AuditAction,
    oldData: log.oldData as Record<string, unknown> | undefined,
    newData: log.newData as Record<string, unknown> | undefined,
    userId: log.userId || undefined,
    timestamp: log.timestamp,
  }));
}
```

---

## 9. BARREL EXPORT

### 9.1 File: `src/lib/db/index.ts`

```typescript
// src/lib/db/index.ts
// Client
export { prisma, type PrismaClientType } from './client';

// Pagination
export {
  type PaginationParams,
  type PaginatedResult,
  type CursorPaginatedResult,
  getOffsetPaginationArgs,
  createPaginatedResult,
  getCursorPaginationArgs,
  createCursorPaginatedResult,
} from './pagination';

// Transactions
export {
  type TransactionOptions,
  executeTransaction,
  executeWithRetry,
  batchOperation,
} from './transactions';

// Query Builders
export {
  type FilterOperator,
  type FilterCondition,
  type SortDirection,
  type SortCondition,
  buildWhereClause,
  buildOrderByClause,
  buildSearchClause,
  combineWhereClauses,
} from './query-builders';

// Errors
export {
  DatabaseError,
  handlePrismaError,
  isRecordNotFound,
  isUniqueConstraintViolation,
  isForeignKeyViolation,
} from './errors';

// Soft Delete
export {
  applySoftDeleteMiddleware,
  hardDelete,
  restoreSoftDeleted,
  findIncludingDeleted,
} from './soft-delete';

// Audit
export {
  type AuditAction,
  type AuditLogEntry,
  logAudit,
  applyAuditMiddleware,
  getAuditHistory,
} from './audit';
```

---

## 11. CHECKLIST IMPLEMENTAZIONE

| Step | Descrizione | Status |
|------|-------------|--------|
| 1 | Creare `src/lib/db/client.ts` | ⬜ |
| 2 | Creare `src/lib/db/pagination.ts` | ⬜ |
| 3 | Creare `src/lib/db/transactions.ts` | ⬜ |
| 4 | Creare `src/lib/db/query-builders.ts` | ⬜ |
| 5 | Creare `src/lib/db/errors.ts` | ⬜ |
| 6 | Creare `src/lib/db/soft-delete.ts` | ⬜ |
| 7 | Creare `src/lib/db/audit.ts` | ⬜ |
| 8 | Creare `src/lib/db/index.ts` (barrel) | ⬜ |
| 9 | Aggiungere modello AuditLog allo schema Prisma | ⬜ |
| 10 | Applicare middleware in `src/lib/db/client.ts` | ⬜ |
| 11 | Scrivere test per pagination | ⬜ |
| 12 | Scrivere test per error handling | ⬜ |

---

_Integrato da: 06-OUTPUT-MODULO-DATABASE.md_
_Data integrazione: 2026-01-29 14:51_
_Generato con: Gemini 2.5 Flash / DeepSeek R1_