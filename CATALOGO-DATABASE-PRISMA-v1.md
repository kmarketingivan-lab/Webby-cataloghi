# CATALOGO-DATABASE-PRISMA-v1

> **Dominio**: Utilities database e gestione Prisma Client
> **Stack**: Prisma ORM, PostgreSQL, TypeScript
> **Versione**: 1.0
> **Data**: 2026-01-29

---

§ 1. INDICE

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

§ 1. PANORAMICA

§ 1.1 OBIETTIVO
Utilities TypeScript per la gestione avanzata del database con Prisma ORM, incluse paginazione, transazioni, query builders e error handling.

§ 1.2 STRUTTURA FILE

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

§ 1.3 FUNZIONALITÀ PRINCIPALI

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

§ 2. PRISMA CLIENT

§ 2.1 FILE: `SRC/LIB/DB/CLIENT.TS`

| Export | Tipo | Descrizione |
|--------|------|-------------|
| `prisma` | PrismaClient | Singleton instance |
| `PrismaClientType` | Type | Type del client |

typescript
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

---

§ 3. PAGINATION

§ 3.1 TYPES

| Type | Descrizione |
|------|-------------|
| `PaginationParams` | Input paginazione |
| `PaginatedResult<T>` | Output paginato |
| `CursorPaginatedResult<T>` | Output cursor-based |

§ 3.2 COSTANTI

| Costante | Valore | Descrizione |
|----------|--------|-------------|
| `DEFAULT_PAGE_SIZE` | 10 | Items per pagina default |
| `MAX_PAGE_SIZE` | 100 | Limite massimo items |

§ 3.3 FUNZIONI

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `getOffsetPaginationArgs` | `PaginationParams` | `{ skip, take }` | Calcola skip/take |
| `createPaginatedResult` | `data, total, params` | `PaginatedResult<T>` | Crea risultato |
| `getCursorPaginationArgs` | `cursor?, pageSize?` | `{ cursor?, take, skip? }` | Args cursor |
| `createCursorPaginatedResult` | `data, pageSize, idField` | `CursorPaginatedResult<T>` | Crea risultato cursor |

§ 3.4 FILE: `SRC/LIB/DB/PAGINATION.TS`

typescript
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

---

§ 4. TRANSACTION HELPERS

§ 4.1 FUNZIONI

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `executeTransaction` | `callback, options?` | `Promise<T>` | Esegue transazione |
| `executeWithRetry` | `callback, maxRetries?` | `Promise<T>` | Con retry automatico |

§ 4.2 OPZIONI TRANSAZIONE

| Opzione | Tipo | Default | Descrizione |
|---------|------|---------|-------------|
| `maxWait` | number | 5000 | Max wait ms |
| `timeout` | number | 10000 | Timeout ms |
| `isolationLevel` | enum | ReadCommitted | Isolation level |

§ 4.3 FILE: `SRC/LIB/DB/TRANSACTIONS.TS`

typescript
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

---

§ 5. QUERY BUILDERS

§ 5.1 TYPES

| Type | Descrizione |
|------|-------------|
| `FilterOperator` | Operatori filtro (eq, ne, gt, lt, contains, etc.) |
| `FilterCondition` | Singola condizione filtro |
| `SortDirection` | asc / desc |
| `SortCondition` | Campo + direzione |

§ 5.2 FUNZIONI

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `buildWhereClause` | `FilterCondition[]` | `Prisma.XWhereInput` | Costruisce where |
| `buildOrderByClause` | `SortCondition[]` | `Prisma.XOrderByInput` | Costruisce orderBy |
| `buildSearchClause` | `query, fields[]` | `Prisma.XWhereInput` | Full-text search |

§ 5.3 FILE: `SRC/LIB/DB/QUERY-BUILDERS.TS`

typescript
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

---

§ 6. ERROR HANDLING

§ 6.1 ERROR CODES PRISMA

| Code | Descrizione | HTTP Status |
|------|-------------|-------------|
| `P2000` | Value too long | 400 |
| `P2001` | Record not found | 404 |
| `P2002` | Unique constraint violation | 409 |
| `P2003` | Foreign key constraint | 400 |
| `P2025` | Record not found (relation) | 404 |

§ 6.2 FUNZIONI

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `handlePrismaError` | `error` | `AppError` | Converte errore |
| `isRecordNotFound` | `error` | `boolean` | Check not found |
| `isUniqueConstraintViolation` | `error` | `boolean` | Check unique |

§ 6.3 FILE: `SRC/LIB/DB/ERRORS.TS`

typescript
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

---

§ 7. SOFT DELETE

§ 7.1 PATTERN

| Campo | Tipo | Descrizione |
|-------|------|-------------|
| `deletedAt` | DateTime? | Timestamp eliminazione |
| `isDeleted` | Boolean | Flag computed (opzionale) |

§ 7.2 MIDDLEWARE

| Middleware | Azione | Descrizione |
|------------|--------|-------------|
| `findMany` | Filtra | Esclude deletedAt != null |
| `findFirst` | Filtra | Esclude deletedAt != null |
| `findUnique` | Filtra | Esclude deletedAt != null |
| `delete` | Modifica | Converte in update deletedAt |
| `deleteMany` | Modifica | Converte in updateMany |

§ 7.3 FILE: `SRC/LIB/DB/SOFT-DELETE.TS`

typescript
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

---

§ 8. AUDIT LOG

§ 8.1 SCHEMA AUDIT

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

§ 8.2 FUNZIONI

| Funzione | Parametri | Return | Descrizione |
|----------|-----------|--------|-------------|
| `logAudit` | `params` | `Promise<void>` | Crea log entry |
| `applyAuditMiddleware` | `prisma, getUserId` | `void` | Applica middleware |

§ 8.3 FILE: `SRC/LIB/DB/AUDIT.TS`

typescript
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

---

§ 9. BARREL EXPORT

§ 9.1 FILE: `SRC/LIB/DB/INDEX.TS`

typescript
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

---

§ 11. CHECKLIST IMPLEMENTAZIONE

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


---

§ 9. PRISMA SCHEMA COMPLETO — E-COMMERCE

§ 9.1 PANORAMICA
Schema Prisma production-ready per un e-commerce completo con gestione prodotti,
varianti, categorie gerarchiche, ordini, pagamenti, recensioni e wishlist.

§ 9.2 IMPLEMENTAZIONE COMPLETA

```prisma
// prisma/schema.prisma — E-Commerce Complete Schema

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "fullTextIndex"]
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String    @map("password_hash")
  firstName     String    @map("first_name")
  lastName      String    @map("last_name")
  displayName   String?   @map("display_name")
  role          UserRole  @default(CUSTOMER)
  emailVerified Boolean   @default(false) @map("email_verified")
  avatarUrl     String?   @map("avatar_url")
  phone         String?
  locale        String    @default("it-IT")
  timezone      String    @default("Europe/Rome")
  lastLoginAt   DateTime? @map("last_login_at")
  createdAt     DateTime  @default(now()) @map("created_at")
  updatedAt     DateTime  @updatedAt @map("updated_at")
  deletedAt     DateTime? @map("deleted_at")

  addresses     Address[]
  orders        Order[]
  reviews       Review[]
  cart          Cart?
  wishlist      WishlistItem[]
  sessions      Session[]
  notifications Notification[]
  couponsUsed   CouponUsage[]

  @@index([email])
  @@index([role])
  @@index([deletedAt])
  @@index([createdAt])
  @@map("users")
}

enum UserRole {
  CUSTOMER
  ADMIN
  MANAGER
  SUPPORT
}

model Address {
  id          String      @id @default(cuid())
  userId      String      @map("user_id")
  type        AddressType @default(SHIPPING)
  label       String?
  firstName   String      @map("first_name")
  lastName    String      @map("last_name")
  company     String?
  street      String
  streetLine2 String?     @map("street_line2")
  city        String
  state       String
  zipCode     String      @map("zip_code")
  country     String      @default("IT")
  phone       String?
  isDefault   Boolean     @default(false) @map("is_default")
  createdAt   DateTime    @default(now()) @map("created_at")
  updatedAt   DateTime    @updatedAt @map("updated_at")

  user          User            @relation(fields: [userId], references: [id], onDelete: Cascade)
  shippingOrders Order[]        @relation("ShippingAddress")
  billingOrders  Order[]        @relation("BillingAddress")

  @@index([userId])
  @@map("addresses")
}

enum AddressType {
  SHIPPING
  BILLING
  BOTH
}

model Category {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?    @db.Text
  imageUrl    String?    @map("image_url")
  iconName    String?    @map("icon_name")
  parentId    String?    @map("parent_id")
  level       Int        @default(0)
  path        String     @default("/")
  sortOrder   Int        @default(0) @map("sort_order")
  isActive    Boolean    @default(true) @map("is_active")
  isFeatured  Boolean    @default(false) @map("is_featured")
  metaTitle       String? @map("meta_title")
  metaDescription String? @map("meta_description")
  createdAt   DateTime   @default(now()) @map("created_at")
  updatedAt   DateTime   @updatedAt @map("updated_at")

  parent      Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryTree")
  products    ProductCategory[]

  @@index([slug])
  @@index([parentId])
  @@index([isActive, sortOrder])
  @@map("categories")
}

model Product {
  id                String    @id @default(cuid())
  name              String
  slug              String    @unique
  description       String    @db.Text
  shortDescription  String?   @map("short_description")
  sku               String    @unique
  barcode           String?
  price             Decimal   @db.Decimal(10, 2)
  compareAtPrice    Decimal?  @db.Decimal(10, 2) @map("compare_at_price")
  costPrice         Decimal?  @db.Decimal(10, 2) @map("cost_price")
  currency          String    @default("EUR")
  stock             Int       @default(0)
  lowStockThreshold Int       @default(5) @map("low_stock_threshold")
  trackInventory    Boolean   @default(true) @map("track_inventory")
  allowBackorder    Boolean   @default(false) @map("allow_backorder")
  weight            Decimal?  @db.Decimal(8, 3)
  weightUnit        String    @default("kg") @map("weight_unit")
  width             Decimal?  @db.Decimal(8, 2)
  height            Decimal?  @db.Decimal(8, 2)
  depth             Decimal?  @db.Decimal(8, 2)
  dimensionUnit     String    @default("cm") @map("dimension_unit")
  isActive          Boolean   @default(true) @map("is_active")
  isFeatured        Boolean   @default(false) @map("is_featured")
  isDigital         Boolean   @default(false) @map("is_digital")
  taxRate           Decimal?  @db.Decimal(5, 2) @map("tax_rate")
  metaTitle         String?   @map("meta_title")
  metaDescription   String?   @map("meta_description")
  publishedAt       DateTime? @map("published_at")
  createdAt         DateTime  @default(now()) @map("created_at")
  updatedAt         DateTime  @updatedAt @map("updated_at")
  deletedAt         DateTime? @map("deleted_at")

  categories    ProductCategory[]
  variants      ProductVariant[]
  images        ProductImage[]
  reviews       Review[]
  cartItems     CartItem[]
  orderItems    OrderItem[]
  wishlistItems WishlistItem[]
  tags          ProductTag[]
  attributes    ProductAttribute[]

  @@index([slug])
  @@index([sku])
  @@index([isActive, isFeatured])
  @@index([isActive, publishedAt])
  @@index([deletedAt])
  @@index([price])
  @@index([createdAt])
  @@map("products")
}

model ProductVariant {
  id            String   @id @default(cuid())
  productId     String   @map("product_id")
  name          String
  sku           String   @unique
  barcode       String?
  price         Decimal  @db.Decimal(10, 2)
  compareAtPrice Decimal? @db.Decimal(10, 2) @map("compare_at_price")
  costPrice     Decimal? @db.Decimal(10, 2) @map("cost_price")
  stock         Int      @default(0)
  weight        Decimal? @db.Decimal(8, 3)
  options       Json
  imageUrl      String?  @map("image_url")
  isActive      Boolean  @default(true) @map("is_active")
  sortOrder     Int      @default(0) @map("sort_order")
  createdAt     DateTime @default(now()) @map("created_at")
  updatedAt     DateTime @updatedAt @map("updated_at")

  product    Product      @relation(fields: [productId], references: [id], onDelete: Cascade)
  cartItems  CartItem[]
  orderItems OrderItem[]

  @@index([productId])
  @@index([sku])
  @@map("product_variants")
}

model ProductImage {
  id        String  @id @default(cuid())
  productId String  @map("product_id")
  url       String
  alt       String?
  width     Int?
  height    Int?
  format    String?
  size      Int?
  sortOrder Int     @default(0) @map("sort_order")
  isPrimary Boolean @default(false) @map("is_primary")

  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@index([productId])
  @@map("product_images")
}

model ProductCategory {
  productId  String @map("product_id")
  categoryId String @map("category_id")
  sortOrder  Int    @default(0) @map("sort_order")

  product  Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  category Category @relation(fields: [categoryId], references: [id], onDelete: Cascade)

  @@id([productId, categoryId])
  @@map("product_categories")
}

model ProductTag {
  id        String @id @default(cuid())
  productId String @map("product_id")
  tag       String

  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@index([productId])
  @@index([tag])
  @@unique([productId, tag])
  @@map("product_tags")
}

model ProductAttribute {
  id        String @id @default(cuid())
  productId String @map("product_id")
  name      String
  value     String
  sortOrder Int    @default(0) @map("sort_order")

  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@index([productId])
  @@map("product_attributes")
}

model Cart {
  id         String     @id @default(cuid())
  userId     String     @unique @map("user_id")
  sessionId  String?    @map("session_id")
  couponCode String?    @map("coupon_code")
  notes      String?
  createdAt  DateTime   @default(now()) @map("created_at")
  updatedAt  DateTime   @updatedAt @map("updated_at")
  expiresAt  DateTime?  @map("expires_at")

  user  User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  items CartItem[]

  @@map("carts")
}

model CartItem {
  id        String          @id @default(cuid())
  cartId    String          @map("cart_id")
  productId String          @map("product_id")
  variantId String?         @map("variant_id")
  quantity  Int             @default(1)
  metadata  Json?
  createdAt DateTime        @default(now()) @map("created_at")
  updatedAt DateTime        @updatedAt @map("updated_at")

  cart    Cart            @relation(fields: [cartId], references: [id], onDelete: Cascade)
  product Product         @relation(fields: [productId], references: [id])
  variant ProductVariant? @relation(fields: [variantId], references: [id])

  @@unique([cartId, productId, variantId])
  @@map("cart_items")
}

model Order {
  id                String      @id @default(cuid())
  orderNumber       String      @unique @map("order_number")
  userId            String      @map("user_id")
  shippingAddressId String      @map("shipping_address_id")
  billingAddressId  String?     @map("billing_address_id")
  status            OrderStatus @default(PENDING)
  paymentStatus     PaymentStatus @default(PENDING) @map("payment_status")
  subtotal          Decimal     @db.Decimal(10, 2)
  tax               Decimal     @db.Decimal(10, 2)
  shippingCost      Decimal     @db.Decimal(10, 2) @map("shipping_cost")
  discount          Decimal     @db.Decimal(10, 2) @default(0)
  total             Decimal     @db.Decimal(10, 2)
  currency          String      @default("EUR")
  paymentMethod     String?     @map("payment_method")
  paymentIntentId   String?     @unique @map("payment_intent_id")
  shippingMethod    String?     @map("shipping_method")
  trackingNumber    String?     @map("tracking_number")
  trackingUrl       String?     @map("tracking_url")
  notes             String?     @db.Text
  internalNotes     String?     @db.Text @map("internal_notes")
  couponCode        String?     @map("coupon_code")
  ipAddress         String?     @map("ip_address")
  userAgent         String?     @map("user_agent")
  shippedAt         DateTime?   @map("shipped_at")
  deliveredAt       DateTime?   @map("delivered_at")
  cancelledAt       DateTime?   @map("cancelled_at")
  refundedAt        DateTime?   @map("refunded_at")
  createdAt         DateTime    @default(now()) @map("created_at")
  updatedAt         DateTime    @updatedAt @map("updated_at")

  user            User         @relation(fields: [userId], references: [id])
  shippingAddress Address      @relation("ShippingAddress", fields: [shippingAddressId], references: [id])
  billingAddress  Address?     @relation("BillingAddress", fields: [billingAddressId], references: [id])
  items           OrderItem[]
  payments        Payment[]
  timeline        OrderEvent[]
  refunds         Refund[]

  @@index([userId])
  @@index([status])
  @@index([paymentStatus])
  @@index([orderNumber])
  @@index([createdAt])
  @@index([paymentIntentId])
  @@map("orders")
}

enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
  ON_HOLD
}

enum PaymentStatus {
  PENDING
  AUTHORIZED
  CAPTURED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

model OrderItem {
  id          String          @id @default(cuid())
  orderId     String          @map("order_id")
  productId   String          @map("product_id")
  variantId   String?         @map("variant_id")
  name        String
  sku         String
  price       Decimal         @db.Decimal(10, 2)
  quantity    Int
  total       Decimal         @db.Decimal(10, 2)
  tax         Decimal         @db.Decimal(10, 2) @default(0)
  discount    Decimal         @db.Decimal(10, 2) @default(0)
  options     Json?
  metadata    Json?

  order   Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product Product         @relation(fields: [productId], references: [id])
  variant ProductVariant? @relation(fields: [variantId], references: [id])

  @@index([orderId])
  @@map("order_items")
}

model OrderEvent {
  id        String   @id @default(cuid())
  orderId   String   @map("order_id")
  type      String
  title     String
  message   String?  @db.Text
  actorId   String?  @map("actor_id")
  actorType String?  @map("actor_type")
  metadata  Json?
  createdAt DateTime @default(now()) @map("created_at")

  order Order @relation(fields: [orderId], references: [id], onDelete: Cascade)

  @@index([orderId, createdAt])
  @@map("order_events")
}

model Payment {
  id              String   @id @default(cuid())
  orderId         String   @map("order_id")
  stripePaymentId String?  @unique @map("stripe_payment_id")
  amount          Decimal  @db.Decimal(10, 2)
  currency        String   @default("EUR")
  status          String
  method          String
  cardBrand       String?  @map("card_brand")
  cardLast4       String?  @map("card_last4")
  receiptUrl      String?  @map("receipt_url")
  metadata        Json?
  createdAt       DateTime @default(now()) @map("created_at")
  updatedAt       DateTime @updatedAt @map("updated_at")

  order Order @relation(fields: [orderId], references: [id])

  @@index([orderId])
  @@index([stripePaymentId])
  @@map("payments")
}

model Refund {
  id              String   @id @default(cuid())
  orderId         String   @map("order_id")
  stripeRefundId  String?  @unique @map("stripe_refund_id")
  amount          Decimal  @db.Decimal(10, 2)
  reason          String?
  status          String   @default("pending")
  createdAt       DateTime @default(now()) @map("created_at")

  order Order @relation(fields: [orderId], references: [id])

  @@index([orderId])
  @@map("refunds")
}

model Review {
  id         String   @id @default(cuid())
  userId     String   @map("user_id")
  productId  String   @map("product_id")
  rating     Int
  title      String?
  body       String?  @db.Text
  pros       String?  @db.Text
  cons       String?  @db.Text
  isVerified Boolean  @default(false) @map("is_verified")
  isApproved Boolean  @default(false) @map("is_approved")
  helpfulCount Int    @default(0) @map("helpful_count")
  reportCount  Int    @default(0) @map("report_count")
  createdAt  DateTime @default(now()) @map("created_at")
  updatedAt  DateTime @updatedAt @map("updated_at")

  user    User    @relation(fields: [userId], references: [id])
  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([userId, productId])
  @@index([productId, isApproved])
  @@index([rating])
  @@index([createdAt])
  @@map("reviews")
}

model WishlistItem {
  id        String   @id @default(cuid())
  userId    String   @map("user_id")
  productId String   @map("product_id")
  createdAt DateTime @default(now()) @map("created_at")

  user    User    @relation(fields: [userId], references: [id], onDelete: Cascade)
  product Product @relation(fields: [productId], references: [id], onDelete: Cascade)

  @@unique([userId, productId])
  @@map("wishlist_items")
}

model Coupon {
  id              String     @id @default(cuid())
  code            String     @unique
  description     String?
  type            CouponType @default(PERCENTAGE)
  value           Decimal    @db.Decimal(10, 2)
  minOrderAmount  Decimal?   @db.Decimal(10, 2) @map("min_order_amount")
  maxUses         Int?       @map("max_uses")
  maxUsesPerUser  Int?       @map("max_uses_per_user")
  usedCount       Int        @default(0) @map("used_count")
  isActive        Boolean    @default(true) @map("is_active")
  startsAt        DateTime?  @map("starts_at")
  expiresAt       DateTime?  @map("expires_at")
  createdAt       DateTime   @default(now()) @map("created_at")

  usages CouponUsage[]

  @@index([code])
  @@index([isActive, expiresAt])
  @@map("coupons")
}

enum CouponType {
  PERCENTAGE
  FIXED_AMOUNT
  FREE_SHIPPING
}

model CouponUsage {
  id       String   @id @default(cuid())
  couponId String   @map("coupon_id")
  userId   String   @map("user_id")
  orderId  String?  @map("order_id")
  usedAt   DateTime @default(now()) @map("used_at")

  coupon Coupon @relation(fields: [couponId], references: [id])
  user   User   @relation(fields: [userId], references: [id])

  @@index([couponId])
  @@index([userId])
  @@map("coupon_usages")
}

model Session {
  id        String   @id @default(cuid())
  userId    String   @map("user_id")
  token     String   @unique
  expiresAt DateTime @map("expires_at")
  ipAddress String?  @map("ip_address")
  userAgent String?  @map("user_agent")
  createdAt DateTime @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([token])
  @@index([userId])
  @@index([expiresAt])
  @@map("sessions")
}

model Notification {
  id        String   @id @default(cuid())
  userId    String   @map("user_id")
  type      String
  channel   String   @default("in_app")
  title     String
  body      String   @db.Text
  actionUrl String?  @map("action_url")
  isRead    Boolean  @default(false) @map("is_read")
  readAt    DateTime? @map("read_at")
  metadata  Json?
  createdAt DateTime @default(now()) @map("created_at")

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId, isRead])
  @@index([createdAt])
  @@map("notifications")
}
```

§ 9.3 TIPI DERIVATI DALLO SCHEMA

```typescript
// src/lib/db/types.ts — Prisma generated type utilities

import { Prisma } from "@prisma/client";

// ── Product con tutte le relazioni ──
export const productWithRelationsInclude = {
  categories: {
    include: { category: true },
    orderBy: { sortOrder: "asc" as const },
  },
  variants: {
    where: { isActive: true },
    orderBy: { sortOrder: "asc" as const },
  },
  images: {
    orderBy: { sortOrder: "asc" as const },
  },
  reviews: {
    where: { isApproved: true },
    orderBy: { createdAt: "desc" as const },
    take: 10,
  },
  tags: true,
  attributes: {
    orderBy: { sortOrder: "asc" as const },
  },
} satisfies Prisma.ProductInclude;

export type ProductWithRelations = Prisma.ProductGetPayload<{
  include: typeof productWithRelationsInclude;
}>;

// ── Product list item (lightweight) ──
export const productListSelect = {
  id: true,
  name: true,
  slug: true,
  price: true,
  compareAtPrice: true,
  stock: true,
  isActive: true,
  isFeatured: true,
  createdAt: true,
  images: {
    where: { isPrimary: true },
    take: 1,
    select: { url: true, alt: true },
  },
  categories: {
    take: 1,
    select: { category: { select: { name: true, slug: true } } },
  },
  _count: {
    select: { reviews: { where: { isApproved: true } } },
  },
} satisfies Prisma.ProductSelect;

export type ProductListItem = Prisma.ProductGetPayload<{
  select: typeof productListSelect;
}>;

// ── Order con dettagli completi ──
export const orderWithDetailsInclude = {
  user: {
    select: {
      id: true,
      email: true,
      firstName: true,
      lastName: true,
      avatarUrl: true,
    },
  },
  items: {
    include: {
      product: {
        select: {
          id: true,
          slug: true,
          images: { where: { isPrimary: true }, take: 1 },
        },
      },
      variant: true,
    },
  },
  shippingAddress: true,
  billingAddress: true,
  payments: { orderBy: { createdAt: "desc" as const } },
  timeline: { orderBy: { createdAt: "desc" as const } },
  refunds: true,
} satisfies Prisma.OrderInclude;

export type OrderWithDetails = Prisma.OrderGetPayload<{
  include: typeof orderWithDetailsInclude;
}>;

// ── Cart con items e prodotti ──
export const cartWithItemsInclude = {
  items: {
    include: {
      product: {
        select: {
          id: true,
          name: true,
          slug: true,
          price: true,
          stock: true,
          isActive: true,
          images: { where: { isPrimary: true }, take: 1 },
        },
      },
      variant: {
        select: {
          id: true,
          name: true,
          price: true,
          stock: true,
          options: true,
        },
      },
    },
    orderBy: { createdAt: "asc" as const },
  },
} satisfies Prisma.CartInclude;

export type CartWithItems = Prisma.CartGetPayload<{
  include: typeof cartWithItemsInclude;
}>;

// ── User profile ──
export const userProfileSelect = {
  id: true,
  email: true,
  firstName: true,
  lastName: true,
  displayName: true,
  avatarUrl: true,
  phone: true,
  role: true,
  emailVerified: true,
  locale: true,
  timezone: true,
  createdAt: true,
  _count: {
    select: {
      orders: true,
      reviews: true,
      wishlist: true,
    },
  },
} satisfies Prisma.UserSelect;

export type UserProfile = Prisma.UserGetPayload<{
  select: typeof userProfileSelect;
}>;

// ── Utility: Price as number ──
export function decimalToNumber(value: Prisma.Decimal | null | undefined): number {
  if (value === null || value === undefined) return 0;
  return Number(value);
}

// ── Utility: Format price ──
export function formatPrice(
  amount: Prisma.Decimal | number,
  currency = "EUR",
  locale = "it-IT"
): string {
  const num = typeof amount === "number" ? amount : Number(amount);
  return new Intl.NumberFormat(locale, {
    style: "currency",
    currency,
  }).format(num);
}

// ── Utility: Order number generator ──
export function generateOrderNumber(): string {
  const date = new Date();
  const year = date.getFullYear().toString().slice(-2);
  const month = (date.getMonth() + 1).toString().padStart(2, "0");
  const random = Math.random().toString(36).substring(2, 8).toUpperCase();
  return `ORD-${year}${month}-${random}`;
}
```

---

§ 10. PRISMA CLIENT EXTENSIONS

§ 10.1 PANORAMICA
Prisma Client Extensions permettono di aggiungere metodi custom ai modelli,
trasformare risultati, estendere query e aggiungere logica a livello client.

§ 10.2 RESULT EXTENSIONS

```typescript
// src/lib/db/extensions/result-extensions.ts

import { Prisma, PrismaClient } from "@prisma/client";

export function withResultExtensions(prisma: PrismaClient) {
  return prisma.$extends({
    result: {
      user: {
        fullName: {
          needs: { firstName: true, lastName: true },
          compute(user) {
            return `${user.firstName} ${user.lastName}`;
          },
        },
        initials: {
          needs: { firstName: true, lastName: true },
          compute(user) {
            return `${user.firstName.charAt(0)}${user.lastName.charAt(0)}`.toUpperCase();
          },
        },
      },
      product: {
        isOnSale: {
          needs: { price: true, compareAtPrice: true },
          compute(product) {
            if (!product.compareAtPrice) return false;
            return Number(product.compareAtPrice) > Number(product.price);
          },
        },
        discountPercentage: {
          needs: { price: true, compareAtPrice: true },
          compute(product) {
            if (!product.compareAtPrice) return 0;
            const original = Number(product.compareAtPrice);
            const current = Number(product.price);
            if (original <= current) return 0;
            return Math.round(((original - current) / original) * 100);
          },
        },
        isInStock: {
          needs: { stock: true, lowStockThreshold: true },
          compute(product) {
            return product.stock > 0;
          },
        },
        isLowStock: {
          needs: { stock: true, lowStockThreshold: true },
          compute(product) {
            return product.stock > 0 && product.stock <= product.lowStockThreshold;
          },
        },
        formattedPrice: {
          needs: { price: true, currency: true },
          compute(product) {
            return new Intl.NumberFormat("it-IT", {
              style: "currency",
              currency: product.currency,
            }).format(Number(product.price));
          },
        },
      },
      order: {
        formattedTotal: {
          needs: { total: true, currency: true },
          compute(order) {
            return new Intl.NumberFormat("it-IT", {
              style: "currency",
              currency: order.currency,
            }).format(Number(order.total));
          },
        },
        isEditable: {
          needs: { status: true },
          compute(order) {
            return ["PENDING", "CONFIRMED"].includes(order.status);
          },
        },
        isCancellable: {
          needs: { status: true },
          compute(order) {
            return ["PENDING", "CONFIRMED", "PROCESSING"].includes(order.status);
          },
        },
      },
      review: {
        starDisplay: {
          needs: { rating: true },
          compute(review) {
            return "★".repeat(review.rating) + "☆".repeat(5 - review.rating);
          },
        },
      },
    },
  });
}
```

§ 10.3 MODEL EXTENSIONS

```typescript
// src/lib/db/extensions/model-extensions.ts

import { Prisma, PrismaClient } from "@prisma/client";

export function withModelExtensions(prisma: PrismaClient) {
  return prisma.$extends({
    model: {
      user: {
        async findByEmail(email: string) {
          return prisma.user.findUnique({
            where: { email: email.toLowerCase().trim() },
          });
        },

        async existsByEmail(email: string): Promise<boolean> {
          const count = await prisma.user.count({
            where: { email: email.toLowerCase().trim() },
          });
          return count > 0;
        },

        async getActiveUsers(options?: { take?: number; skip?: number }) {
          return prisma.user.findMany({
            where: { deletedAt: null, emailVerified: true },
            orderBy: { createdAt: "desc" },
            take: options?.take ?? 50,
            skip: options?.skip ?? 0,
          });
        },

        async updateLastLogin(userId: string) {
          return prisma.user.update({
            where: { id: userId },
            data: { lastLoginAt: new Date() },
          });
        },
      },

      product: {
        async findBySlug(slug: string) {
          return prisma.product.findUnique({
            where: { slug, isActive: true, deletedAt: null },
            include: {
              categories: { include: { category: true } },
              variants: { where: { isActive: true } },
              images: { orderBy: { sortOrder: "asc" } },
              attributes: { orderBy: { sortOrder: "asc" } },
            },
          });
        },

        async getFeatured(limit = 8) {
          return prisma.product.findMany({
            where: {
              isActive: true,
              isFeatured: true,
              deletedAt: null,
              stock: { gt: 0 },
            },
            include: {
              images: { where: { isPrimary: true }, take: 1 },
            },
            orderBy: { createdAt: "desc" },
            take: limit,
          });
        },

        async getLowStockProducts() {
          return prisma.$queryRaw<
            Array<{ id: string; name: string; sku: string; stock: number; threshold: number }>
          >`
            SELECT id, name, sku, stock, low_stock_threshold AS threshold
            FROM products
            WHERE stock <= low_stock_threshold
              AND stock > 0
              AND is_active = true
              AND deleted_at IS NULL
            ORDER BY stock ASC
          `;
        },

        async getAverageRating(productId: string): Promise<number> {
          const result = await prisma.review.aggregate({
            where: { productId, isApproved: true },
            _avg: { rating: true },
          });
          return result._avg.rating ?? 0;
        },
      },

      order: {
        async findByOrderNumber(orderNumber: string) {
          return prisma.order.findUnique({
            where: { orderNumber },
            include: {
              items: { include: { product: true, variant: true } },
              shippingAddress: true,
              payments: true,
              timeline: { orderBy: { createdAt: "desc" } },
            },
          });
        },

        async getDashboardStats(organizationId?: string) {
          const now = new Date();
          const startOfMonth = new Date(now.getFullYear(), now.getMonth(), 1);
          const startOfDay = new Date(now.getFullYear(), now.getMonth(), now.getDate());

          const [totalOrders, monthlyRevenue, todayOrders, pendingOrders] = await Promise.all([
            prisma.order.count(),
            prisma.order.aggregate({
              where: {
                createdAt: { gte: startOfMonth },
                paymentStatus: "CAPTURED",
              },
              _sum: { total: true },
            }),
            prisma.order.count({
              where: { createdAt: { gte: startOfDay } },
            }),
            prisma.order.count({
              where: { status: "PENDING" },
            }),
          ]);

          return {
            totalOrders,
            monthlyRevenue: Number(monthlyRevenue._sum.total ?? 0),
            todayOrders,
            pendingOrders,
          };
        },
      },
    },
  });
}
```

§ 10.4 QUERY EXTENSIONS

```typescript
// src/lib/db/extensions/query-extensions.ts

import { PrismaClient } from "@prisma/client";

// Extension per logging performance e metriche
export function withQueryLogging(prisma: PrismaClient) {
  return prisma.$extends({
    query: {
      $allModels: {
        async $allOperations({ model, operation, args, query }) {
          const start = performance.now();
          const result = await query(args);
          const duration = performance.now() - start;

          if (duration > 200) {
            console.warn(
              `[Slow Query] ${model}.${operation} took ${duration.toFixed(1)}ms`,
              JSON.stringify(args).substring(0, 200)
            );
          }

          if (process.env.PRISMA_QUERY_LOG === "true") {
            console.log(
              `[Query] ${model}.${operation} ${duration.toFixed(1)}ms`
            );
          }

          return result;
        },
      },
    },
  });
}

// Extension per retry automatico su deadlock
export function withRetry(prisma: PrismaClient, maxRetries = 3) {
  return prisma.$extends({
    query: {
      $allModels: {
        async $allOperations({ args, query }) {
          let lastError: Error | null = null;

          for (let attempt = 0; attempt <= maxRetries; attempt++) {
            try {
              return await query(args);
            } catch (error: any) {
              lastError = error;

              // Retry su deadlock (P2034) o connection error
              const retryableCodes = ["P2034", "P1001", "P1002"];
              if (
                error?.code &&
                retryableCodes.includes(error.code) &&
                attempt < maxRetries
              ) {
                const delay = Math.pow(2, attempt) * 100;
                console.warn(
                  `[Retry] Attempt ${attempt + 1}/${maxRetries} after ${delay}ms (${error.code})`
                );
                await new Promise((resolve) => setTimeout(resolve, delay));
                continue;
              }

              throw error;
            }
          }

          throw lastError;
        },
      },
    },
  });
}

// Componi tutte le extensions insieme
export function createExtendedPrisma() {
  const basePrisma = new PrismaClient();

  // Import le altre extensions
  const { withResultExtensions } = require("./result-extensions");
  const { withModelExtensions } = require("./model-extensions");

  let client = withResultExtensions(basePrisma);
  client = withModelExtensions(client);
  client = withQueryLogging(client);

  return client;
}

export type ExtendedPrismaClient = ReturnType<typeof createExtendedPrisma>;
```

---

§ 11. PRISMA MIDDLEWARE AVANZATO

§ 11.1 TIMING MIDDLEWARE

```typescript
// src/lib/db/middleware/timing.ts

import { Prisma } from "@prisma/client";

export const timingMiddleware: Prisma.Middleware = async (params, next) => {
  const before = Date.now();
  const result = await next(params);
  const after = Date.now();
  const duration = after - before;

  if (duration > 100) {
    console.warn(
      `[Prisma Slow] ${params.model}.${params.action} took ${duration}ms`
    );
  }

  return result;
};
```

§ 11.2 TENANT ISOLATION MIDDLEWARE

```typescript
// src/lib/db/middleware/tenant-isolation.ts

import { Prisma } from "@prisma/client";
import { AsyncLocalStorage } from "async_hooks";

interface TenantContext {
  organizationId: string;
  userId: string;
}

export const tenantStore = new AsyncLocalStorage<TenantContext>();

const TENANT_MODELS = [
  "Workspace",
  "Project",
  "Member",
  "Invitation",
  "ApiKey",
  "AuditLog",
  "UsageLimit",
];

export const tenantIsolationMiddleware: Prisma.Middleware = async (
  params,
  next
) => {
  const ctx = tenantStore.getStore();

  if (!ctx || !params.model || !TENANT_MODELS.includes(params.model)) {
    return next(params);
  }

  const { organizationId } = ctx;

  // Inject organizationId nei filtri di lettura
  if (["findMany", "findFirst", "count", "aggregate"].includes(params.action)) {
    params.args = params.args ?? {};
    params.args.where = {
      ...params.args.where,
      organizationId,
    };
  }

  // Inject organizationId nella creazione
  if (["create", "createMany"].includes(params.action)) {
    if (params.action === "create") {
      params.args.data = {
        ...params.args.data,
        organizationId,
      };
    }
  }

  // Inject organizationId negli update/delete
  if (["update", "updateMany", "delete", "deleteMany"].includes(params.action)) {
    params.args = params.args ?? {};
    params.args.where = {
      ...params.args.where,
      organizationId,
    };
  }

  return next(params);
};

// Helper per eseguire codice nel contesto di un tenant
export function withTenant<T>(
  context: TenantContext,
  fn: () => Promise<T>
): Promise<T> {
  return tenantStore.run(context, fn);
}
```

§ 11.3 SOFT DELETE MIDDLEWARE

```typescript
// src/lib/db/middleware/soft-delete.ts

import { Prisma } from "@prisma/client";

const SOFT_DELETE_MODELS = ["User", "Product", "Organization", "Order"];

export const softDeleteMiddleware: Prisma.Middleware = async (params, next) => {
  if (!params.model || !SOFT_DELETE_MODELS.includes(params.model)) {
    return next(params);
  }

  // Override delete -> soft delete
  if (params.action === "delete") {
    params.action = "update";
    params.args.data = { deletedAt: new Date() };
  }

  if (params.action === "deleteMany") {
    params.action = "updateMany";
    if (params.args.data !== undefined) {
      params.args.data.deletedAt = new Date();
    } else {
      params.args.data = { deletedAt: new Date() };
    }
  }

  // Filter out soft-deleted records
  if (["findMany", "findFirst", "count"].includes(params.action)) {
    if (!params.args) params.args = {};
    if (!params.args.where) params.args.where = {};

    if (params.args.where.deletedAt === undefined) {
      params.args.where.deletedAt = null;
    }
  }

  return next(params);
};
```

---

§ 12. SEEDING CON FAKER

§ 12.1 PANORAMICA
Script di seeding completo con @faker-js/faker per generare dati realistici
per development, staging e demo environments.

§ 12.2 IMPLEMENTAZIONE COMPLETA

```typescript
// prisma/seed.ts — Database seeder

import { PrismaClient, UserRole, OrderStatus } from "@prisma/client";
import { faker } from "@faker-js/faker/locale/it";
import { hash } from "bcryptjs";

const prisma = new PrismaClient();

const SEED_CONFIGS = {
  development: { users: 50, categories: 10, products: 150, orders: 200 },
  staging: { users: 200, categories: 20, products: 500, orders: 1000 },
  demo: { users: 20, categories: 8, products: 80, orders: 50 },
};

async function main() {
  const env = (process.env.SEED_ENV ?? "development") as keyof typeof SEED_CONFIGS;
  const config = SEED_CONFIGS[env] ?? SEED_CONFIGS.development;
  console.log(`\nSeeding database (${env})...\n`);

  // Clean in reverse dependency order
  await prisma.$transaction([
    prisma.couponUsage.deleteMany(),
    prisma.coupon.deleteMany(),
    prisma.refund.deleteMany(),
    prisma.orderEvent.deleteMany(),
    prisma.payment.deleteMany(),
    prisma.orderItem.deleteMany(),
    prisma.order.deleteMany(),
    prisma.review.deleteMany(),
    prisma.wishlistItem.deleteMany(),
    prisma.cartItem.deleteMany(),
    prisma.cart.deleteMany(),
    prisma.productAttribute.deleteMany(),
    prisma.productTag.deleteMany(),
    prisma.productImage.deleteMany(),
    prisma.productVariant.deleteMany(),
    prisma.productCategory.deleteMany(),
    prisma.product.deleteMany(),
    prisma.category.deleteMany(),
    prisma.notification.deleteMany(),
    prisma.session.deleteMany(),
    prisma.address.deleteMany(),
    prisma.user.deleteMany(),
  ]);

  // 1. Seed Users
  console.log(`Creating ${config.users} users...`);
  const passwordHash = await hash("Password123!", 10);

  const admin = await prisma.user.create({
    data: {
      email: "admin@webby.dev",
      passwordHash,
      firstName: "Admin",
      lastName: "Webby",
      role: UserRole.ADMIN,
      emailVerified: true,
    },
  });

  const userIds: string[] = [admin.id];

  for (let i = 0; i < config.users - 1; i++) {
    const firstName = faker.person.firstName();
    const lastName = faker.person.lastName();
    const user = await prisma.user.create({
      data: {
        email: faker.internet
          .email({ firstName, lastName, provider: "example.com" })
          .toLowerCase(),
        passwordHash,
        firstName,
        lastName,
        displayName: `${firstName} ${lastName.charAt(0)}.`,
        role: faker.helpers.weightedArrayElement([
          { value: UserRole.CUSTOMER, weight: 85 },
          { value: UserRole.SUPPORT, weight: 10 },
          { value: UserRole.MANAGER, weight: 5 },
        ]),
        emailVerified: faker.datatype.boolean(0.8),
        avatarUrl: faker.image.avatar(),
        phone: faker.phone.number(),
        lastLoginAt: faker.date.recent({ days: 30 }),
      },
    });
    userIds.push(user.id);

    // Address per ogni utente
    await prisma.address.create({
      data: {
        userId: user.id,
        type: "BOTH",
        firstName,
        lastName,
        street: faker.location.streetAddress(),
        city: faker.location.city(),
        state: faker.location.state(),
        zipCode: faker.location.zipCode(),
        country: "IT",
        isDefault: true,
      },
    });
  }
  console.log(`  Created ${userIds.length} users`);

  // 2. Seed Categories
  console.log(`Creating ${config.categories} categories...`);
  const categoryNames = [
    "Elettronica", "Abbigliamento", "Casa e Giardino", "Sport e Fitness",
    "Libri", "Giocattoli", "Alimentari", "Bellezza e Cura",
    "Automotive", "Musica e Film", "Informatica", "Gioielli",
    "Scarpe", "Ufficio", "Pet Care", "Bambini",
    "Salute", "Arte e Hobby", "Viaggi", "Fotografia",
  ];

  const categoryIds: string[] = [];
  for (let i = 0; i < config.categories; i++) {
    const name = categoryNames[i % categoryNames.length];
    const cat = await prisma.category.create({
      data: {
        name,
        slug: faker.helpers.slugify(name).toLowerCase() + (i >= categoryNames.length ? `-${i}` : ""),
        description: faker.commerce.department() + ": " + faker.lorem.sentence(),
        imageUrl: faker.image.urlPicsumPhotos({ width: 400, height: 300 }),
        sortOrder: i,
        isActive: true,
        isFeatured: i < 4,
        level: 0,
        path: "/",
      },
    });
    categoryIds.push(cat.id);
  }
  console.log(`  Created ${categoryIds.length} categories`);

  // 3. Seed Products
  console.log(`Creating ${config.products} products...`);
  const productIds: string[] = [];

  for (let i = 0; i < config.products; i++) {
    const name = faker.commerce.productName();
    const price = parseFloat(faker.commerce.price({ min: 5, max: 500 }));
    const hasDiscount = faker.datatype.boolean(0.25);
    const categoryId = faker.helpers.arrayElement(categoryIds);

    const product = await prisma.product.create({
      data: {
        name,
        slug: faker.helpers.slugify(name).toLowerCase() + "-" + faker.string.nanoid(6),
        description: faker.commerce.productDescription() + "\n\n" + faker.lorem.paragraphs(2),
        shortDescription: faker.commerce.productDescription(),
        sku: `SKU-${faker.string.alphanumeric(8).toUpperCase()}`,
        price,
        compareAtPrice: hasDiscount ? Math.round(price * 1.3 * 100) / 100 : null,
        costPrice: Math.round(price * 0.55 * 100) / 100,
        stock: faker.number.int({ min: 0, max: 200 }),
        isActive: faker.datatype.boolean(0.92),
        isFeatured: faker.datatype.boolean(0.12),
        taxRate: 22.0,
        metaTitle: name,
        metaDescription: faker.commerce.productDescription().substring(0, 155),
        publishedAt: faker.date.past({ years: 1 }),
        categories: { create: { categoryId } },
        images: {
          createMany: {
            data: Array.from(
              { length: faker.number.int({ min: 1, max: 4 }) },
              (_, idx) => ({
                url: faker.image.urlPicsumPhotos({ width: 800, height: 800 }),
                alt: `${name} - foto ${idx + 1}`,
                sortOrder: idx,
                isPrimary: idx === 0,
              })
            ),
          },
        },
        tags: {
          createMany: {
            data: faker.helpers.uniqueArray(
              () => faker.commerce.productAdjective().toLowerCase(),
              faker.number.int({ min: 1, max: 4 })
            ).map((tag) => ({ tag })),
          },
        },
      },
    });
    productIds.push(product.id);
  }
  console.log(`  Created ${productIds.length} products`);

  // 4. Seed Orders
  console.log(`Creating ${config.orders} orders...`);
  let orderCount = 0;

  for (let i = 0; i < config.orders; i++) {
    const userId = faker.helpers.arrayElement(userIds);
    const address = await prisma.address.findFirst({ where: { userId } });
    if (!address) continue;

    const itemCount = faker.number.int({ min: 1, max: 5 });
    const selectedProducts = faker.helpers.arrayElements(productIds, itemCount);
    let subtotal = 0;
    const items: Array<{
      productId: string;
      name: string;
      sku: string;
      price: number;
      quantity: number;
      total: number;
    }> = [];

    for (const pid of selectedProducts) {
      const p = await prisma.product.findUnique({
        where: { id: pid },
        select: { name: true, sku: true, price: true },
      });
      if (!p) continue;
      const qty = faker.number.int({ min: 1, max: 3 });
      const lineTotal = Number(p.price) * qty;
      subtotal += lineTotal;
      items.push({
        productId: pid,
        name: p.name,
        sku: p.sku,
        price: Number(p.price),
        quantity: qty,
        total: lineTotal,
      });
    }

    if (items.length === 0) continue;

    const tax = Math.round(subtotal * 0.22 * 100) / 100;
    const shipping = subtotal > 50 ? 0 : 7.99;
    const total = Math.round((subtotal + tax + shipping) * 100) / 100;

    const status = faker.helpers.weightedArrayElement([
      { value: OrderStatus.DELIVERED, weight: 35 },
      { value: OrderStatus.SHIPPED, weight: 15 },
      { value: OrderStatus.PROCESSING, weight: 15 },
      { value: OrderStatus.CONFIRMED, weight: 10 },
      { value: OrderStatus.PENDING, weight: 15 },
      { value: OrderStatus.CANCELLED, weight: 5 },
      { value: OrderStatus.REFUNDED, weight: 5 },
    ]);

    const createdAt = faker.date.past({ years: 1 });
    const orderNum = `ORD-${createdAt.getFullYear().toString().slice(-2)}${(createdAt.getMonth() + 1).toString().padStart(2, "0")}-${faker.string.alphanumeric(6).toUpperCase()}`;

    await prisma.order.create({
      data: {
        orderNumber: orderNum,
        userId,
        shippingAddressId: address.id,
        status,
        paymentStatus: status === "CANCELLED" ? "FAILED" : "CAPTURED",
        subtotal,
        tax,
        shippingCost: shipping,
        total,
        paymentMethod: faker.helpers.arrayElement(["card", "paypal"]),
        createdAt,
        items: { createMany: { data: items } },
        payments: {
          create: {
            amount: total,
            status: status === "CANCELLED" ? "failed" : "succeeded",
            method: "card",
            cardBrand: faker.helpers.arrayElement(["visa", "mastercard", "amex"]),
            cardLast4: faker.finance.creditCardNumber("####"),
            createdAt,
          },
        },
        timeline: {
          create: {
            type: "order_created",
            title: "Ordine creato",
            message: `Ordine ${orderNum} creato dal cliente`,
            actorType: "customer",
            createdAt,
          },
        },
      },
    });
    orderCount++;
  }
  console.log(`  Created ${orderCount} orders`);

  // 5. Seed Reviews
  console.log("Creating reviews...");
  let reviewCount = 0;
  for (const productId of productIds.slice(0, Math.ceil(productIds.length * 0.7))) {
    const reviewerCount = faker.number.int({ min: 1, max: 8 });
    const reviewers = faker.helpers.arrayElements(userIds, reviewerCount);

    for (const userId of reviewers) {
      try {
        await prisma.review.create({
          data: {
            userId,
            productId,
            rating: faker.helpers.weightedArrayElement([
              { value: 5, weight: 30 },
              { value: 4, weight: 35 },
              { value: 3, weight: 20 },
              { value: 2, weight: 10 },
              { value: 1, weight: 5 },
            ]),
            title: faker.lorem.sentence({ min: 3, max: 8 }),
            body: faker.lorem.paragraph(),
            isVerified: faker.datatype.boolean(0.5),
            isApproved: faker.datatype.boolean(0.85),
            helpfulCount: faker.number.int({ min: 0, max: 20 }),
            createdAt: faker.date.past({ years: 1 }),
          },
        });
        reviewCount++;
      } catch {
        // skip duplicate user+product
      }
    }
  }
  console.log(`  Created ${reviewCount} reviews`);

  // 6. Seed Coupons
  console.log("Creating coupons...");
  const coupons = [
    { code: "WELCOME10", type: "PERCENTAGE" as const, value: 10, minOrder: 30 },
    { code: "SAVE20", type: "PERCENTAGE" as const, value: 20, minOrder: 80 },
    { code: "FLAT15", type: "FIXED_AMOUNT" as const, value: 15, minOrder: 50 },
    { code: "FREESHIP", type: "FREE_SHIPPING" as const, value: 0, minOrder: 25 },
  ];

  for (const c of coupons) {
    await prisma.coupon.create({
      data: {
        code: c.code,
        description: `Coupon ${c.code}`,
        type: c.type,
        value: c.value,
        minOrderAmount: c.minOrder,
        maxUses: 1000,
        maxUsesPerUser: 1,
        isActive: true,
        expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000),
      },
    });
  }
  console.log(`  Created ${coupons.length} coupons`);

  console.log("\nSeeding complete!\n");
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(() => prisma.$disconnect());
```

---

§ 13. RAW QUERIES E PERFORMANCE

§ 13.1 PANORAMICA
Pattern per raw queries type-safe con Prisma, performance optimization,
e query complesse non supportate dal query builder.

§ 13.2 RAW QUERIES TYPE-SAFE

```typescript
// src/lib/db/raw-queries.ts

import { PrismaClient, Prisma } from "@prisma/client";

const prisma = new PrismaClient();

// Full-text search con ranking
interface SearchResult {
  id: string;
  name: string;
  slug: string;
  price: number;
  rank: number;
  headline: string;
}

export async function searchProducts(
  query: string,
  options: {
    categoryId?: string;
    minPrice?: number;
    maxPrice?: number;
    limit?: number;
    offset?: number;
  } = {}
): Promise<SearchResult[]> {
  const { categoryId, minPrice, maxPrice, limit = 20, offset = 0 } = options;

  return prisma.$queryRaw<SearchResult[]>`
    SELECT
      p.id,
      p.name,
      p.slug,
      p.price::float,
      ts_rank(
        to_tsvector('italian', p.name || ' ' || coalesce(p.description, '')),
        plainto_tsquery('italian', ${query})
      ) AS rank,
      ts_headline(
        'italian',
        p.name || ' ' || coalesce(p.short_description, ''),
        plainto_tsquery('italian', ${query}),
        'StartSel=<mark>, StopSel=</mark>, MaxWords=35, MinWords=15'
      ) AS headline
    FROM products p
    WHERE
      p.is_active = true
      AND p.deleted_at IS NULL
      AND to_tsvector('italian', p.name || ' ' || coalesce(p.description, ''))
          @@ plainto_tsquery('italian', ${query})
      ${categoryId ? Prisma.sql`AND EXISTS (
        SELECT 1 FROM product_categories pc
        WHERE pc.product_id = p.id AND pc.category_id = ${categoryId}
      )` : Prisma.empty}
      ${minPrice !== undefined ? Prisma.sql`AND p.price >= ${minPrice}` : Prisma.empty}
      ${maxPrice !== undefined ? Prisma.sql`AND p.price <= ${maxPrice}` : Prisma.empty}
    ORDER BY rank DESC
    LIMIT ${limit}
    OFFSET ${offset}
  `;
}

// Dashboard analytics con aggregazioni complesse
interface DashboardStats {
  totalRevenue: number;
  orderCount: number;
  avgOrderValue: number;
  topProducts: Array<{
    id: string;
    name: string;
    totalSold: number;
    revenue: number;
  }>;
  revenueByMonth: Array<{
    month: string;
    revenue: number;
    orders: number;
  }>;
}

export async function getDashboardStats(
  dateFrom: Date,
  dateTo: Date
): Promise<DashboardStats> {
  const [summary, topProducts, revenueByMonth] = await Promise.all([
    prisma.$queryRaw<
      Array<{ total_revenue: number; order_count: number; avg_order_value: number }>
    >`
      SELECT
        COALESCE(SUM(total)::float, 0) AS total_revenue,
        COUNT(*)::int AS order_count,
        COALESCE(AVG(total)::float, 0) AS avg_order_value
      FROM orders
      WHERE created_at >= ${dateFrom}
        AND created_at <= ${dateTo}
        AND payment_status = 'CAPTURED'
    `,

    prisma.$queryRaw<
      Array<{ id: string; name: string; total_sold: number; revenue: number }>
    >`
      SELECT
        p.id,
        p.name,
        SUM(oi.quantity)::int AS total_sold,
        SUM(oi.total)::float AS revenue
      FROM order_items oi
      JOIN products p ON p.id = oi.product_id
      JOIN orders o ON o.id = oi.order_id
      WHERE o.created_at >= ${dateFrom}
        AND o.created_at <= ${dateTo}
        AND o.payment_status = 'CAPTURED'
      GROUP BY p.id, p.name
      ORDER BY revenue DESC
      LIMIT 10
    `,

    prisma.$queryRaw<
      Array<{ month: string; revenue: number; orders: number }>
    >`
      SELECT
        to_char(created_at, 'YYYY-MM') AS month,
        COALESCE(SUM(total)::float, 0) AS revenue,
        COUNT(*)::int AS orders
      FROM orders
      WHERE created_at >= ${dateFrom}
        AND created_at <= ${dateTo}
        AND payment_status = 'CAPTURED'
      GROUP BY to_char(created_at, 'YYYY-MM')
      ORDER BY month
    `,
  ]);

  const s = summary[0];
  return {
    totalRevenue: s?.total_revenue ?? 0,
    orderCount: s?.order_count ?? 0,
    avgOrderValue: s?.avg_order_value ?? 0,
    topProducts: topProducts.map((p) => ({
      id: p.id,
      name: p.name,
      totalSold: p.total_sold,
      revenue: p.revenue,
    })),
    revenueByMonth,
  };
}

// Batch update con raw SQL (molto piu veloce di updateMany per grandi dataset)
export async function batchUpdatePrices(
  updates: Array<{ productId: string; newPrice: number }>
): Promise<number> {
  if (updates.length === 0) return 0;

  const values = updates
    .map((u) => Prisma.sql`(${u.productId}::text, ${u.newPrice}::decimal)`)
    .reduce((acc, val, i) => (i === 0 ? val : Prisma.sql`${acc}, ${val}`));

  const result = await prisma.$executeRaw`
    UPDATE products p
    SET
      price = v.new_price,
      updated_at = NOW()
    FROM (VALUES ${values}) AS v(product_id, new_price)
    WHERE p.id = v.product_id
  `;

  return result;
}

// Recursive CTE per category tree
export async function getCategoryTree(
  rootId?: string
): Promise<
  Array<{
    id: string;
    name: string;
    slug: string;
    level: number;
    path: string;
    childCount: number;
    productCount: number;
  }>
> {
  if (rootId) {
    return prisma.$queryRaw`
      WITH RECURSIVE tree AS (
        SELECT id, name, slug, parent_id, 0 AS level,
               name::text AS path
        FROM categories
        WHERE id = ${rootId}

        UNION ALL

        SELECT c.id, c.name, c.slug, c.parent_id, t.level + 1,
               (t.path || ' > ' || c.name)::text
        FROM categories c
        JOIN tree t ON c.parent_id = t.id
      )
      SELECT
        t.id,
        t.name,
        t.slug,
        t.level,
        t.path,
        (SELECT COUNT(*) FROM categories WHERE parent_id = t.id)::int AS "childCount",
        (SELECT COUNT(*) FROM product_categories WHERE category_id = t.id)::int AS "productCount"
      FROM tree t
      ORDER BY t.level, t.name
    `;
  }

  return prisma.$queryRaw`
    WITH RECURSIVE tree AS (
      SELECT id, name, slug, parent_id, 0 AS level,
             name::text AS path
      FROM categories
      WHERE parent_id IS NULL

      UNION ALL

      SELECT c.id, c.name, c.slug, c.parent_id, t.level + 1,
             (t.path || ' > ' || c.name)::text
      FROM categories c
      JOIN tree t ON c.parent_id = t.id
    )
    SELECT
      t.id,
      t.name,
      t.slug,
      t.level,
      t.path,
      (SELECT COUNT(*) FROM categories WHERE parent_id = t.id)::int AS "childCount",
      (SELECT COUNT(*) FROM product_categories WHERE category_id = t.id)::int AS "productCount"
    FROM tree t
    ORDER BY t.level, t.name
  `;
}
```

§ 13.3 PERFORMANCE OPTIMIZATION PATTERNS

```typescript
// src/lib/db/performance.ts

import { PrismaClient, Prisma } from "@prisma/client";

const prisma = new PrismaClient();

// Batch operations con chunking
export async function batchProcess<T, R>(
  items: T[],
  processor: (batch: T[]) => Promise<R[]>,
  batchSize = 100
): Promise<R[]> {
  const results: R[] = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await processor(batch);
    results.push(...batchResults);

    if (i + batchSize < items.length) {
      console.log(`  Processed ${i + batchSize}/${items.length}`);
    }
  }

  return results;
}

// Cursor-based iteration per grandi dataset
export async function* iterateProducts(
  where: Prisma.ProductWhereInput = {},
  batchSize = 100
) {
  let cursor: string | undefined;

  while (true) {
    const products = await prisma.product.findMany({
      where,
      take: batchSize,
      ...(cursor ? { skip: 1, cursor: { id: cursor } } : {}),
      orderBy: { id: "asc" },
    });

    if (products.length === 0) break;

    for (const product of products) {
      yield product;
    }

    cursor = products[products.length - 1].id;

    if (products.length < batchSize) break;
  }
}

// N+1 prevention: preload relations
export async function getProductsWithAvgRating(
  productIds: string[]
): Promise<Map<string, number>> {
  const ratings = await prisma.$queryRaw<
    Array<{ product_id: string; avg_rating: number }>
  >`
    SELECT
      product_id,
      COALESCE(AVG(rating)::float, 0) AS avg_rating
    FROM reviews
    WHERE product_id = ANY(${productIds}::text[])
      AND is_approved = true
    GROUP BY product_id
  `;

  const map = new Map<string, number>();
  for (const r of ratings) {
    map.set(r.product_id, Math.round(r.avg_rating * 10) / 10);
  }
  return map;
}

// Connection pooling health check
export async function getConnectionPoolHealth(): Promise<{
  active: number;
  idle: number;
  waiting: number;
  total: number;
  maxConnections: number;
  utilizationPercent: number;
}> {
  const result = await prisma.$queryRaw<
    Array<{
      active: number;
      idle: number;
      waiting: number;
      total: number;
      max_conn: number;
    }>
  >`
    SELECT
      count(*) FILTER (WHERE state = 'active')::int AS active,
      count(*) FILTER (WHERE state = 'idle')::int AS idle,
      count(*) FILTER (WHERE wait_event IS NOT NULL)::int AS waiting,
      count(*)::int AS total,
      (SELECT setting::int FROM pg_settings WHERE name = 'max_connections') AS max_conn
    FROM pg_stat_activity
    WHERE datname = current_database()
  `;

  const s = result[0];
  return {
    active: s.active,
    idle: s.idle,
    waiting: s.waiting,
    total: s.total,
    maxConnections: s.max_conn,
    utilizationPercent: Math.round((s.total / s.max_conn) * 100),
  };
}

// Detect and log slow queries
export async function getSlowQueries(
  durationMs = 1000
): Promise<
  Array<{
    pid: number;
    duration: string;
    query: string;
    state: string;
    waitEvent: string | null;
  }>
> {
  return prisma.$queryRaw`
    SELECT
      pid,
      age(clock_timestamp(), query_start)::text AS duration,
      left(query, 500) AS query,
      state,
      wait_event_type AS "waitEvent"
    FROM pg_stat_activity
    WHERE state = 'active'
      AND query NOT LIKE '%pg_stat%'
      AND query_start < now() - interval '${Prisma.raw(durationMs.toString())} milliseconds'
    ORDER BY query_start ASC
  `;
}
```

### Errori Comuni da Evitare
- Non usare `$queryRawUnsafe` con input utente — sempre `$queryRaw` con template literals
- Non dimenticare i cast di tipo nelle raw queries (::float, ::int, ::text)
- Non iterare con findMany senza cursor per dataset > 10k righe
- Non fare N+1 queries — usare include/select o batch loading

### Checklist di Verifica
- [ ] Raw queries usano template literals (mai string concatenation)
- [ ] Tipi di ritorno espliciti per tutte le raw queries
- [ ] Batch processing per operazioni su grandi dataset
- [ ] Cursor-based iteration per export/migration
- [ ] N+1 detection con query logging in development
- [ ] Connection pool monitoring attivo



---

§ 14. SCHEMA SAAS MULTI-TENANT

§ 14.1 PANORAMICA
Schema Prisma per applicazioni SaaS multi-tenant con organization, workspace,
member roles, subscription billing, inviti e row-level security.

§ 14.2 IMPLEMENTAZIONE COMPLETA

```prisma
// prisma/schema-saas.prisma — SaaS Multi-Tenant Schema

model Organization {
  id                   String    @id @default(cuid())
  name                 String
  slug                 String    @unique
  logoUrl              String?   @map("logo_url")
  domain               String?   @unique
  plan                 SaasPlan  @default(FREE)
  stripeCustomerId     String?   @unique @map("stripe_customer_id")
  stripeSubscriptionId String?   @unique @map("stripe_subscription_id")
  trialEndsAt          DateTime? @map("trial_ends_at")
  billingEmail         String?   @map("billing_email")
  taxId                String?   @map("tax_id")
  country              String    @default("IT")
  isActive             Boolean   @default(true) @map("is_active")
  settings             Json      @default("{}")
  createdAt            DateTime  @default(now()) @map("created_at")
  updatedAt            DateTime  @updatedAt @map("updated_at")
  deletedAt            DateTime? @map("deleted_at")

  members       OrgMember[]
  workspaces    OrgWorkspace[]
  invitations   OrgInvitation[]
  apiKeys       OrgApiKey[]
  webhooks      OrgWebhook[]
  auditLogs     OrgAuditLog[]
  subscriptions OrgSubscription[]
  usageLimits   OrgUsageLimit[]

  @@index([slug])
  @@index([domain])
  @@index([stripeCustomerId])
  @@index([plan])
  @@map("organizations")
}

enum SaasPlan {
  FREE
  STARTER
  PRO
  BUSINESS
  ENTERPRISE
}

model OrgMember {
  id             String        @id @default(cuid())
  organizationId String        @map("organization_id")
  userId         String        @map("user_id")
  role           OrgMemberRole @default(MEMBER)
  title          String?
  isActive       Boolean       @default(true) @map("is_active")
  joinedAt       DateTime      @default(now()) @map("joined_at")
  updatedAt      DateTime      @updatedAt @map("updated_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@unique([organizationId, userId])
  @@index([userId])
  @@map("org_members")
}

enum OrgMemberRole {
  OWNER
  ADMIN
  MANAGER
  MEMBER
  VIEWER
  BILLING
}

model OrgWorkspace {
  id             String   @id @default(cuid())
  organizationId String   @map("organization_id")
  name           String
  slug           String
  description    String?
  color          String   @default("#6366f1")
  icon           String?
  isDefault      Boolean  @default(false) @map("is_default")
  isArchived     Boolean  @default(false) @map("is_archived")
  settings       Json     @default("{}")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  organization Organization   @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  projects     OrgProject[]

  @@unique([organizationId, slug])
  @@index([organizationId])
  @@map("org_workspaces")
}

model OrgProject {
  id          String           @id @default(cuid())
  workspaceId String           @map("workspace_id")
  name        String
  description String?
  status      OrgProjectStatus @default(ACTIVE)
  color       String           @default("#3b82f6")
  dueDate     DateTime?        @map("due_date")
  settings    Json             @default("{}")
  createdAt   DateTime         @default(now()) @map("created_at")
  updatedAt   DateTime         @updatedAt @map("updated_at")

  workspace OrgWorkspace @relation(fields: [workspaceId], references: [id], onDelete: Cascade)

  @@index([workspaceId])
  @@map("org_projects")
}

enum OrgProjectStatus {
  ACTIVE
  ON_HOLD
  COMPLETED
  ARCHIVED
}

model OrgInvitation {
  id             String              @id @default(cuid())
  organizationId String              @map("organization_id")
  email          String
  role           OrgMemberRole       @default(MEMBER)
  status         OrgInvitationStatus @default(PENDING)
  token          String              @unique @default(cuid())
  invitedById    String              @map("invited_by_id")
  message        String?
  expiresAt      DateTime            @map("expires_at")
  acceptedAt     DateTime?           @map("accepted_at")
  createdAt      DateTime            @default(now()) @map("created_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@unique([organizationId, email])
  @@index([token])
  @@index([email])
  @@map("org_invitations")
}

enum OrgInvitationStatus {
  PENDING
  ACCEPTED
  EXPIRED
  REVOKED
}

model OrgApiKey {
  id             String    @id @default(cuid())
  organizationId String    @map("organization_id")
  name           String
  description    String?
  keyHash        String    @unique @map("key_hash")
  keyPrefix      String    @map("key_prefix")
  scopes         String[]  @default([])
  lastUsedAt     DateTime? @map("last_used_at")
  expiresAt      DateTime? @map("expires_at")
  isActive       Boolean   @default(true) @map("is_active")
  createdById    String    @map("created_by_id")
  createdAt      DateTime  @default(now()) @map("created_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([keyHash])
  @@index([organizationId])
  @@map("org_api_keys")
}

model OrgWebhook {
  id             String   @id @default(cuid())
  organizationId String   @map("organization_id")
  url            String
  secret         String
  events         String[]
  isActive       Boolean  @default(true) @map("is_active")
  failureCount   Int      @default(0) @map("failure_count")
  lastTriggeredAt DateTime? @map("last_triggered_at")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId])
  @@map("org_webhooks")
}

model OrgSubscription {
  id                   String                @id @default(cuid())
  organizationId       String                @map("organization_id")
  stripeSubscriptionId String                @unique @map("stripe_subscription_id")
  stripePriceId        String                @map("stripe_price_id")
  status               OrgSubscriptionStatus
  currentPeriodStart   DateTime              @map("current_period_start")
  currentPeriodEnd     DateTime              @map("current_period_end")
  cancelAtPeriodEnd    Boolean               @default(false) @map("cancel_at_period_end")
  canceledAt           DateTime?             @map("canceled_at")
  trialStart           DateTime?             @map("trial_start")
  trialEnd             DateTime?             @map("trial_end")
  quantity             Int                   @default(1)
  metadata             Json?
  createdAt            DateTime              @default(now()) @map("created_at")
  updatedAt            DateTime              @updatedAt @map("updated_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId])
  @@index([stripeSubscriptionId])
  @@index([status])
  @@map("org_subscriptions")
}

enum OrgSubscriptionStatus {
  ACTIVE
  PAST_DUE
  CANCELED
  INCOMPLETE
  INCOMPLETE_EXPIRED
  TRIALING
  UNPAID
  PAUSED
}

model OrgUsageLimit {
  id             String   @id @default(cuid())
  organizationId String   @map("organization_id")
  feature        String
  limit          Int
  used           Int      @default(0)
  period         String   @default("monthly")
  resetAt        DateTime @map("reset_at")
  createdAt      DateTime @default(now()) @map("created_at")
  updatedAt      DateTime @updatedAt @map("updated_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@unique([organizationId, feature])
  @@map("org_usage_limits")
}

model OrgAuditLog {
  id             String   @id @default(cuid())
  organizationId String   @map("organization_id")
  actorId        String?  @map("actor_id")
  actorEmail     String?  @map("actor_email")
  action         String
  entity         String
  entityId       String?  @map("entity_id")
  changes        Json?
  metadata       Json?
  ipAddress      String?  @map("ip_address")
  userAgent      String?  @map("user_agent")
  createdAt      DateTime @default(now()) @map("created_at")

  organization Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId, createdAt])
  @@index([actorId])
  @@index([entity, entityId])
  @@index([action])
  @@map("org_audit_logs")
}
```

§ 14.3 TENANT ISOLATION SERVICE

```typescript
// src/lib/db/tenant-service.ts — Multi-tenant operations

import { PrismaClient, Prisma, OrgMemberRole, SaasPlan } from "@prisma/client";
import crypto from "crypto";

const prisma = new PrismaClient();

// Plan limits configuration
const PLAN_LIMITS: Record<string, Record<string, number>> = {
  FREE: { members: 3, workspaces: 1, projects: 5, apiKeys: 1, storage_mb: 100 },
  STARTER: { members: 10, workspaces: 5, projects: 25, apiKeys: 5, storage_mb: 1000 },
  PRO: { members: 50, workspaces: 20, projects: 100, apiKeys: 20, storage_mb: 10000 },
  BUSINESS: { members: 200, workspaces: 50, projects: 500, apiKeys: 50, storage_mb: 50000 },
  ENTERPRISE: { members: -1, workspaces: -1, projects: -1, apiKeys: -1, storage_mb: -1 },
};

export class TenantService {
  // Create organization with owner
  async createOrganization(
    userId: string,
    data: { name: string; billingEmail?: string }
  ): Promise<{ organizationId: string; slug: string }> {
    let slug = data.name
      .toLowerCase()
      .replace(/[^a-z0-9]+/g, "-")
      .replace(/^-|-$/g, "");

    // Ensure unique slug
    const exists = await prisma.organization.findUnique({ where: { slug } });
    if (exists) {
      slug += "-" + crypto.randomBytes(3).toString("hex");
    }

    const org = await prisma.$transaction(async (tx) => {
      const organization = await tx.organization.create({
        data: {
          name: data.name,
          slug,
          billingEmail: data.billingEmail,
          plan: SaasPlan.FREE,
          trialEndsAt: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000),
        },
      });

      // Add creator as owner
      await tx.orgMember.create({
        data: {
          organizationId: organization.id,
          userId,
          role: OrgMemberRole.OWNER,
        },
      });

      // Create default workspace
      await tx.orgWorkspace.create({
        data: {
          organizationId: organization.id,
          name: "General",
          slug: "general",
          isDefault: true,
        },
      });

      // Initialize usage limits
      const limits = PLAN_LIMITS.FREE;
      const nextMonth = new Date();
      nextMonth.setMonth(nextMonth.getMonth() + 1, 1);
      nextMonth.setHours(0, 0, 0, 0);

      await tx.orgUsageLimit.createMany({
        data: Object.entries(limits).map(([feature, limit]) => ({
          organizationId: organization.id,
          feature,
          limit: limit === -1 ? 999999 : limit,
          resetAt: nextMonth,
        })),
      });

      // Log creation
      await tx.orgAuditLog.create({
        data: {
          organizationId: organization.id,
          actorId: userId,
          action: "organization.created",
          entity: "Organization",
          entityId: organization.id,
          metadata: { plan: "FREE", slug },
        },
      });

      return organization;
    });

    return { organizationId: org.id, slug: org.slug };
  }

  // Check usage limit
  async checkLimit(
    organizationId: string,
    feature: string
  ): Promise<{ allowed: boolean; used: number; limit: number; remaining: number }> {
    const usage = await prisma.orgUsageLimit.findUnique({
      where: { organizationId_feature: { organizationId, feature } },
    });

    if (!usage) {
      return { allowed: true, used: 0, limit: 999999, remaining: 999999 };
    }

    // Auto-reset if period expired
    if (new Date() > usage.resetAt) {
      const nextMonth = new Date();
      nextMonth.setMonth(nextMonth.getMonth() + 1, 1);
      await prisma.orgUsageLimit.update({
        where: { id: usage.id },
        data: { used: 0, resetAt: nextMonth },
      });
      return { allowed: true, used: 0, limit: usage.limit, remaining: usage.limit };
    }

    const remaining = usage.limit - usage.used;
    return {
      allowed: remaining > 0,
      used: usage.used,
      limit: usage.limit,
      remaining: Math.max(0, remaining),
    };
  }

  // Increment usage
  async incrementUsage(organizationId: string, feature: string): Promise<void> {
    const check = await this.checkLimit(organizationId, feature);
    if (!check.allowed) {
      throw new Error(`Usage limit exceeded for ${feature}`);
    }

    await prisma.orgUsageLimit.update({
      where: { organizationId_feature: { organizationId, feature } },
      data: { used: { increment: 1 } },
    });
  }

  // Generate API key
  async generateApiKey(
    organizationId: string,
    data: { name: string; scopes?: string[]; createdById: string }
  ): Promise<{ key: string; keyPrefix: string; id: string }> {
    await this.incrementUsage(organizationId, "apiKeys");

    const rawKey = `wby_live_${crypto.randomBytes(32).toString("hex")}`;
    const keyPrefix = rawKey.substring(0, 16);
    const keyHash = crypto.createHash("sha256").update(rawKey).digest("hex");

    const apiKey = await prisma.orgApiKey.create({
      data: {
        organizationId,
        name: data.name,
        keyHash,
        keyPrefix,
        scopes: data.scopes ?? [],
        createdById: data.createdById,
        expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000),
      },
    });

    return { key: rawKey, keyPrefix, id: apiKey.id };
  }

  // Validate API key
  async validateApiKey(
    rawKey: string
  ): Promise<{ valid: boolean; organizationId?: string; scopes?: string[] }> {
    const keyHash = crypto.createHash("sha256").update(rawKey).digest("hex");

    const apiKey = await prisma.orgApiKey.findUnique({
      where: { keyHash },
      select: {
        id: true,
        organizationId: true,
        scopes: true,
        expiresAt: true,
        isActive: true,
        organization: { select: { isActive: true, deletedAt: true } },
      },
    });

    if (!apiKey || !apiKey.isActive || !apiKey.organization.isActive) {
      return { valid: false };
    }

    if (apiKey.expiresAt && apiKey.expiresAt < new Date()) {
      return { valid: false };
    }

    if (apiKey.organization.deletedAt) {
      return { valid: false };
    }

    // Update last used timestamp (fire and forget)
    prisma.orgApiKey
      .update({
        where: { id: apiKey.id },
        data: { lastUsedAt: new Date() },
      })
      .catch(() => {});

    return {
      valid: true,
      organizationId: apiKey.organizationId,
      scopes: apiKey.scopes,
    };
  }

  // Upgrade plan
  async upgradePlan(
    organizationId: string,
    newPlan: SaasPlan,
    actorId: string
  ): Promise<void> {
    const org = await prisma.organization.findUnique({
      where: { id: organizationId },
      select: { plan: true },
    });

    if (!org) throw new Error("Organization not found");

    const oldPlan = org.plan;
    const newLimits = PLAN_LIMITS[newPlan];

    await prisma.$transaction(async (tx) => {
      // Update plan
      await tx.organization.update({
        where: { id: organizationId },
        data: { plan: newPlan },
      });

      // Update usage limits
      for (const [feature, limit] of Object.entries(newLimits)) {
        await tx.orgUsageLimit.upsert({
          where: { organizationId_feature: { organizationId, feature } },
          update: { limit: limit === -1 ? 999999 : limit },
          create: {
            organizationId,
            feature,
            limit: limit === -1 ? 999999 : limit,
            resetAt: new Date(new Date().getFullYear(), new Date().getMonth() + 1, 1),
          },
        });
      }

      // Audit log
      await tx.orgAuditLog.create({
        data: {
          organizationId,
          actorId,
          action: "organization.plan_changed",
          entity: "Organization",
          entityId: organizationId,
          changes: { plan: { from: oldPlan, to: newPlan } },
        },
      });
    });
  }
}

export const tenantService = new TenantService();
```

§ 14.4 AUDIT LOG QUERY SERVICE

```typescript
// src/lib/db/audit-service.ts — Audit log query and export

import { PrismaClient, Prisma } from "@prisma/client";

const prisma = new PrismaClient();

interface AuditQueryParams {
  organizationId: string;
  actorId?: string;
  entity?: string;
  entityId?: string;
  action?: string;
  from?: Date;
  to?: Date;
  search?: string;
  limit?: number;
  cursor?: string;
}

interface AuditEntry {
  id: string;
  action: string;
  entity: string;
  entityId: string | null;
  actorId: string | null;
  actorEmail: string | null;
  changes: any;
  metadata: any;
  ipAddress: string | null;
  createdAt: Date;
}

export class AuditService {
  async query(
    params: AuditQueryParams
  ): Promise<{ entries: AuditEntry[]; nextCursor: string | null; total: number }> {
    const {
      organizationId,
      actorId,
      entity,
      entityId,
      action,
      from,
      to,
      search,
      limit = 50,
      cursor,
    } = params;

    const where: Prisma.OrgAuditLogWhereInput = { organizationId };

    if (actorId) where.actorId = actorId;
    if (entity) where.entity = entity;
    if (entityId) where.entityId = entityId;
    if (action) where.action = { contains: action };
    if (from || to) {
      where.createdAt = {};
      if (from) where.createdAt.gte = from;
      if (to) where.createdAt.lte = to;
    }

    const [entries, total] = await Promise.all([
      prisma.orgAuditLog.findMany({
        where,
        take: limit + 1,
        ...(cursor ? { skip: 1, cursor: { id: cursor } } : {}),
        orderBy: { createdAt: "desc" },
      }),
      prisma.orgAuditLog.count({ where }),
    ]);

    const hasMore = entries.length > limit;
    if (hasMore) entries.pop();

    return {
      entries,
      nextCursor: hasMore ? entries[entries.length - 1].id : null,
      total,
    };
  }

  async getEntityHistory(
    organizationId: string,
    entity: string,
    entityId: string
  ): Promise<AuditEntry[]> {
    return prisma.orgAuditLog.findMany({
      where: { organizationId, entity, entityId },
      orderBy: { createdAt: "desc" },
      take: 100,
    });
  }

  async getRecentActivity(
    organizationId: string,
    limit = 20
  ): Promise<AuditEntry[]> {
    return prisma.orgAuditLog.findMany({
      where: { organizationId },
      orderBy: { createdAt: "desc" },
      take: limit,
    });
  }

  async exportToCsv(
    organizationId: string,
    from: Date,
    to: Date
  ): Promise<string> {
    const entries = await prisma.orgAuditLog.findMany({
      where: {
        organizationId,
        createdAt: { gte: from, lte: to },
      },
      orderBy: { createdAt: "asc" },
    });

    const header = "Timestamp,Actor,Action,Entity,EntityID,IP Address\n";
    const rows = entries
      .map(
        (e) =>
          `${e.createdAt.toISOString()},${e.actorEmail ?? e.actorId ?? "system"},${e.action},${e.entity},${e.entityId ?? ""},${e.ipAddress ?? ""}`
      )
      .join("\n");

    return header + rows;
  }
}

export const auditService = new AuditService();
```

---

§ 15. PRISMA MIGRATE WORKFLOW

§ 15.1 PANORAMICA
Workflow completo per gestire migrazioni database con Prisma in ambienti
development, staging e production.

§ 15.2 SCRIPT DI MIGRAZIONE

```typescript
// scripts/db-migrate.ts — Migration management script

import { execSync } from "child_process";
import { PrismaClient } from "@prisma/client";
import fs from "fs";
import path from "path";

const prisma = new PrismaClient();

type MigrationEnv = "development" | "staging" | "production";

interface MigrationResult {
  success: boolean;
  appliedMigrations: string[];
  duration: number;
  error?: string;
}

export async function migrate(env: MigrationEnv): Promise<MigrationResult> {
  const start = Date.now();
  const applied: string[] = [];

  try {
    // 1. Verify connection
    await prisma.$queryRaw`SELECT 1`;
    console.log(`[migrate] Connected to database (${env})`);

    // 2. Check pending migrations
    const migrationsDir = path.join(process.cwd(), "prisma", "migrations");
    const allMigrations = fs.existsSync(migrationsDir)
      ? fs
          .readdirSync(migrationsDir)
          .filter((d) => !d.startsWith(".") && d !== "migration_lock.toml")
          .sort()
      : [];
    console.log(`[migrate] Total migrations: ${allMigrations.length}`);

    // 3. Run migration based on environment
    if (env === "production" || env === "staging") {
      console.log("[migrate] Running prisma migrate deploy...");
      const output = execSync("npx prisma migrate deploy", {
        encoding: "utf8",
        timeout: 300000,
      });
      console.log(output);

      // Parse applied migrations from output
      const matches = output.match(/Migration .+ applied/g);
      if (matches) {
        applied.push(...matches.map((m) => m.replace(" applied", "")));
      }
    } else {
      console.log("[migrate] Running prisma migrate dev...");
      execSync("npx prisma migrate dev --skip-generate", {
        stdio: "inherit",
        timeout: 300000,
      });
    }

    // 4. Generate client
    console.log("[migrate] Generating Prisma Client...");
    execSync("npx prisma generate", { stdio: "pipe" });

    return {
      success: true,
      appliedMigrations: applied,
      duration: Date.now() - start,
    };
  } catch (error) {
    const message = error instanceof Error ? error.message : String(error);
    console.error(`[migrate] FAILED: ${message}`);

    return {
      success: false,
      appliedMigrations: applied,
      duration: Date.now() - start,
      error: message,
    };
  } finally {
    await prisma.$disconnect();
  }
}

// Health check post-migration
export async function verifyMigration(): Promise<{
  tablesCount: number;
  indexesCount: number;
  viewsCount: number;
}> {
  const tables = await prisma.$queryRaw<Array<{ count: number }>>`
    SELECT COUNT(*)::int AS count
    FROM information_schema.tables
    WHERE table_schema = 'public' AND table_type = 'BASE TABLE'
  `;

  const indexes = await prisma.$queryRaw<Array<{ count: number }>>`
    SELECT COUNT(*)::int AS count FROM pg_indexes WHERE schemaname = 'public'
  `;

  const views = await prisma.$queryRaw<Array<{ count: number }>>`
    SELECT COUNT(*)::int AS count
    FROM information_schema.views
    WHERE table_schema = 'public'
  `;

  return {
    tablesCount: tables[0]?.count ?? 0,
    indexesCount: indexes[0]?.count ?? 0,
    viewsCount: views[0]?.count ?? 0,
  };
}

// CLI
const env = (process.argv[2] ?? "development") as MigrationEnv;
migrate(env).then(async (result) => {
  console.log("\n--- Migration Result ---");
  console.log(`Success: ${result.success}`);
  console.log(`Duration: ${(result.duration / 1000).toFixed(1)}s`);
  console.log(`Applied: ${result.appliedMigrations.length} migrations`);

  if (result.success) {
    const verification = await verifyMigration();
    console.log(`Tables: ${verification.tablesCount}`);
    console.log(`Indexes: ${verification.indexesCount}`);
  }

  if (result.error) {
    console.error(`Error: ${result.error}`);
    process.exit(1);
  }
});
```

§ 15.3 CUSTOM MIGRATION CON DOWN FILE

```typescript
// scripts/create-migration.ts — Create migration with down file

import { execSync } from "child_process";
import fs from "fs";
import path from "path";

const migrationName = process.argv[2];
if (!migrationName) {
  console.error("Usage: npx ts-node scripts/create-migration.ts <name>");
  process.exit(1);
}

// Create the migration
execSync(`npx prisma migrate dev --name ${migrationName} --create-only`, {
  stdio: "inherit",
});

// Find the newly created migration
const migrationsDir = path.join(process.cwd(), "prisma", "migrations");
const migrations = fs
  .readdirSync(migrationsDir)
  .filter((d) => d.includes(migrationName))
  .sort()
  .reverse();

if (migrations.length === 0) {
  console.error("Migration not found");
  process.exit(1);
}

const migrationPath = path.join(migrationsDir, migrations[0]);
const upSql = fs.readFileSync(path.join(migrationPath, "migration.sql"), "utf8");

// Generate down.sql from up.sql (basic reverse engineering)
const downStatements: string[] = [];
const lines = upSql.split("\n");

for (const line of lines) {
  const trimmed = line.trim();

  // CREATE TABLE -> DROP TABLE
  const createMatch = trimmed.match(/^CREATE TABLE "?(\w+)"?/i);
  if (createMatch) {
    downStatements.push(`DROP TABLE IF EXISTS "${createMatch[1]}" CASCADE;`);
    continue;
  }

  // CREATE INDEX -> DROP INDEX
  const indexMatch = trimmed.match(/^CREATE (?:UNIQUE )?INDEX "?(\w+)"?/i);
  if (indexMatch) {
    downStatements.push(`DROP INDEX IF EXISTS "${indexMatch[1]}";`);
    continue;
  }

  // ALTER TABLE ADD COLUMN -> ALTER TABLE DROP COLUMN
  const addColMatch = trimmed.match(
    /^ALTER TABLE "?(\w+)"? ADD COLUMN "?(\w+)"?/i
  );
  if (addColMatch) {
    downStatements.push(
      `ALTER TABLE "${addColMatch[1]}" DROP COLUMN IF EXISTS "${addColMatch[2]}";`
    );
    continue;
  }

  // CREATE TYPE (enum) -> DROP TYPE
  const typeMatch = trimmed.match(/^CREATE TYPE "?(\w+)"?/i);
  if (typeMatch) {
    downStatements.push(`DROP TYPE IF EXISTS "${typeMatch[1]}";`);
    continue;
  }
}

const downSql = `-- Auto-generated down migration for: ${migrations[0]}
-- Review carefully before using in production!

${downStatements.reverse().join("\n")}
`;

fs.writeFileSync(path.join(migrationPath, "down.sql"), downSql);
console.log(`\nCreated down.sql in ${migrationPath}`);
console.log("Review both migration.sql and down.sql before applying.");
```

### Errori Comuni da Evitare
- Non usare `migrate dev` in produzione — sempre `migrate deploy`
- Non modificare migrazioni gia applicate — creare una nuova migrazione correttiva
- Non dimenticare di generare il client dopo ogni migrazione
- Non rimuovere colonne NOT NULL senza prima renderle nullable

### Checklist di Verifica
- [ ] `migrate deploy` per staging e production
- [ ] `migrate dev` solo in development
- [ ] Backup prima di ogni migrazione in produzione
- [ ] `down.sql` per ogni migrazione custom
- [ ] Verifica post-migrazione con health check
- [ ] CI/CD pipeline include migration step



---

§ 16. PRISMA CON ZOD — SCHEMA VALIDATION

§ 16.1 PANORAMICA
Generazione automatica di schemi Zod da Prisma per validazione input,
form validation lato client e API route validation.

§ 16.2 IMPLEMENTAZIONE COMPLETA

```typescript
// src/lib/db/zod-schemas.ts — Zod schemas derivati dal Prisma schema

import { z } from "zod";

// ── User schemas ──
export const createUserSchema = z.object({
  email: z
    .string()
    .email("Email non valida")
    .max(255)
    .transform((v) => v.toLowerCase().trim()),
  password: z
    .string()
    .min(8, "La password deve avere almeno 8 caratteri")
    .max(100)
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/,
      "La password deve contenere maiuscola, minuscola e numero"
    ),
  firstName: z.string().min(1, "Nome obbligatorio").max(100).trim(),
  lastName: z.string().min(1, "Cognome obbligatorio").max(100).trim(),
  phone: z
    .string()
    .regex(/^\+?[\d\s-]{8,20}$/, "Numero di telefono non valido")
    .optional()
    .or(z.literal("")),
});

export const updateUserSchema = z.object({
  firstName: z.string().min(1).max(100).trim().optional(),
  lastName: z.string().min(1).max(100).trim().optional(),
  displayName: z.string().max(100).trim().optional().nullable(),
  phone: z.string().max(20).optional().nullable(),
  avatarUrl: z.string().url().optional().nullable(),
  locale: z.string().max(10).optional(),
  timezone: z.string().max(50).optional(),
});

export const loginSchema = z.object({
  email: z.string().email().transform((v) => v.toLowerCase().trim()),
  password: z.string().min(1, "Password obbligatoria"),
});

export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
export type LoginInput = z.infer<typeof loginSchema>;

// ── Product schemas ──
export const createProductSchema = z.object({
  name: z.string().min(1, "Nome obbligatorio").max(200).trim(),
  description: z.string().min(10, "Descrizione troppo corta").max(10000),
  shortDescription: z.string().max(500).optional(),
  sku: z
    .string()
    .min(3)
    .max(50)
    .regex(/^[A-Z0-9-]+$/, "SKU deve contenere solo lettere maiuscole, numeri e trattini")
    .transform((v) => v.toUpperCase()),
  price: z
    .number()
    .positive("Il prezzo deve essere positivo")
    .max(999999.99)
    .transform((v) => Math.round(v * 100) / 100),
  compareAtPrice: z
    .number()
    .positive()
    .max(999999.99)
    .optional()
    .nullable(),
  costPrice: z.number().positive().max(999999.99).optional().nullable(),
  stock: z.number().int().min(0).default(0),
  lowStockThreshold: z.number().int().min(0).default(5),
  weight: z.number().positive().optional().nullable(),
  isActive: z.boolean().default(true),
  isFeatured: z.boolean().default(false),
  isDigital: z.boolean().default(false),
  taxRate: z.number().min(0).max(100).default(22),
  categoryIds: z.array(z.string().cuid()).min(1, "Seleziona almeno una categoria"),
  tags: z.array(z.string().max(50)).max(10).default([]),
  metaTitle: z.string().max(70).optional(),
  metaDescription: z.string().max(160).optional(),
});

export const updateProductSchema = createProductSchema.partial().extend({
  id: z.string().cuid(),
});

export const productFilterSchema = z.object({
  search: z.string().max(200).optional(),
  categoryId: z.string().cuid().optional(),
  minPrice: z.coerce.number().min(0).optional(),
  maxPrice: z.coerce.number().min(0).optional(),
  inStock: z.coerce.boolean().optional(),
  isFeatured: z.coerce.boolean().optional(),
  sortBy: z.enum(["price_asc", "price_desc", "newest", "popular", "rating"]).default("newest"),
  page: z.coerce.number().int().min(1).default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20),
});

export type CreateProductInput = z.infer<typeof createProductSchema>;
export type UpdateProductInput = z.infer<typeof updateProductSchema>;
export type ProductFilterInput = z.infer<typeof productFilterSchema>;

// ── Order schemas ──
export const createOrderSchema = z.object({
  items: z
    .array(
      z.object({
        productId: z.string().cuid(),
        variantId: z.string().cuid().optional().nullable(),
        quantity: z.number().int().min(1).max(99),
      })
    )
    .min(1, "Il carrello e vuoto"),
  shippingAddressId: z.string().cuid(),
  billingAddressId: z.string().cuid().optional(),
  paymentMethod: z.enum(["card", "paypal", "bank_transfer"]),
  couponCode: z.string().max(50).optional(),
  notes: z.string().max(1000).optional(),
});

export const updateOrderStatusSchema = z.object({
  orderId: z.string().cuid(),
  status: z.enum([
    "PENDING",
    "CONFIRMED",
    "PROCESSING",
    "SHIPPED",
    "DELIVERED",
    "CANCELLED",
    "REFUNDED",
    "ON_HOLD",
  ]),
  trackingNumber: z.string().max(100).optional(),
  trackingUrl: z.string().url().optional(),
  internalNotes: z.string().max(2000).optional(),
});

export type CreateOrderInput = z.infer<typeof createOrderSchema>;
export type UpdateOrderStatusInput = z.infer<typeof updateOrderStatusSchema>;

// ── Address schemas ──
export const addressSchema = z.object({
  type: z.enum(["SHIPPING", "BILLING", "BOTH"]).default("SHIPPING"),
  label: z.string().max(50).optional(),
  firstName: z.string().min(1).max(100).trim(),
  lastName: z.string().min(1).max(100).trim(),
  company: z.string().max(200).optional(),
  street: z.string().min(1, "Indirizzo obbligatorio").max(200).trim(),
  streetLine2: z.string().max(200).optional(),
  city: z.string().min(1, "Citta obbligatoria").max(100).trim(),
  state: z.string().min(1, "Provincia obbligatoria").max(100).trim(),
  zipCode: z
    .string()
    .min(1, "CAP obbligatorio")
    .max(20)
    .regex(/^\d{5}$/, "CAP non valido (5 cifre)"),
  country: z.string().length(2).default("IT"),
  phone: z.string().max(20).optional(),
  isDefault: z.boolean().default(false),
});

export type AddressInput = z.infer<typeof addressSchema>;

// ── Review schemas ──
export const createReviewSchema = z.object({
  productId: z.string().cuid(),
  rating: z.number().int().min(1).max(5),
  title: z.string().min(3, "Titolo troppo corto").max(200).trim(),
  body: z.string().min(10, "Recensione troppo corta").max(5000).trim(),
  pros: z.string().max(1000).optional(),
  cons: z.string().max(1000).optional(),
});

export type CreateReviewInput = z.infer<typeof createReviewSchema>;

// ── Organization schemas (SaaS) ──
export const createOrganizationSchema = z.object({
  name: z.string().min(2, "Nome troppo corto").max(100).trim(),
  billingEmail: z.string().email().optional(),
});

export const inviteMemberSchema = z.object({
  email: z.string().email("Email non valida"),
  role: z.enum(["ADMIN", "MANAGER", "MEMBER", "VIEWER", "BILLING"]).default("MEMBER"),
  message: z.string().max(500).optional(),
});

export type CreateOrganizationInput = z.infer<typeof createOrganizationSchema>;
export type InviteMemberInput = z.infer<typeof inviteMemberSchema>;

// ── Pagination schema riusabile ──
export const paginationSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  pageSize: z.coerce.number().int().min(1).max(100).default(20),
});

export const cursorPaginationSchema = z.object({
  cursor: z.string().cuid().optional(),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  direction: z.enum(["forward", "backward"]).default("forward"),
});

export type PaginationInput = z.infer<typeof paginationSchema>;
export type CursorPaginationInput = z.infer<typeof cursorPaginationSchema>;
```

§ 16.3 VALIDAZIONE NELLE API ROUTES

```typescript
// src/lib/api/validate.ts — Route handler validation helper

import { NextRequest, NextResponse } from "next/server";
import { z, ZodError, ZodSchema } from "zod";

interface ValidationResult<T> {
  success: true;
  data: T;
}

interface ValidationError {
  success: false;
  error: NextResponse;
}

// Validate request body
export async function validateBody<T extends ZodSchema>(
  request: NextRequest,
  schema: T
): Promise<ValidationResult<z.infer<T>> | ValidationError> {
  try {
    const body = await request.json();
    const data = schema.parse(body);
    return { success: true, data };
  } catch (error) {
    if (error instanceof ZodError) {
      return {
        success: false,
        error: NextResponse.json(
          {
            type: "https://httpstatuses.io/422",
            title: "Validation Error",
            status: 422,
            errors: error.errors.map((e) => ({
              field: e.path.join("."),
              message: e.message,
              code: e.code,
            })),
          },
          { status: 422 }
        ),
      };
    }
    return {
      success: false,
      error: NextResponse.json(
        { type: "https://httpstatuses.io/400", title: "Bad Request", status: 400 },
        { status: 400 }
      ),
    };
  }
}

// Validate query params
export function validateQuery<T extends ZodSchema>(
  request: NextRequest,
  schema: T
): ValidationResult<z.infer<T>> | ValidationError {
  try {
    const params = Object.fromEntries(request.nextUrl.searchParams.entries());
    const data = schema.parse(params);
    return { success: true, data };
  } catch (error) {
    if (error instanceof ZodError) {
      return {
        success: false,
        error: NextResponse.json(
          {
            type: "https://httpstatuses.io/422",
            title: "Validation Error",
            status: 422,
            errors: error.errors.map((e) => ({
              field: e.path.join("."),
              message: e.message,
              code: e.code,
            })),
          },
          { status: 422 }
        ),
      };
    }
    return {
      success: false,
      error: NextResponse.json(
        { type: "https://httpstatuses.io/400", title: "Bad Request", status: 400 },
        { status: 400 }
      ),
    };
  }
}

// Example usage in route handler:
// app/api/products/route.ts
import { createProductSchema, productFilterSchema } from "@/lib/db/zod-schemas";

export async function GET(request: NextRequest) {
  const validation = validateQuery(request, productFilterSchema);
  if (!validation.success) return validation.error;

  const { search, categoryId, minPrice, maxPrice, sortBy, page, pageSize } =
    validation.data;

  // ... query logic using validated params
  return NextResponse.json({ products: [], total: 0 });
}

export async function POST(request: NextRequest) {
  const validation = await validateBody(request, createProductSchema);
  if (!validation.success) return validation.error;

  const data = validation.data;
  // ... create product logic
  return NextResponse.json({ product: data }, { status: 201 });
}
```

### Errori Comuni da Evitare
- Non dimenticare `.transform()` per normalizzare input (lowercase email, trim, etc.)
- Non usare `z.any()` o `z.unknown()` quando si puo definire un tipo preciso
- Non duplicare validazione — generare schemi Zod dal Prisma schema quando possibile
- Non ignorare errori di validazione nel client — mostrare messaggi user-friendly

### Checklist di Verifica
- [ ] Schema Zod per ogni entita principale (User, Product, Order, Address, Review)
- [ ] Validazione su ogni API route con validateBody/validateQuery
- [ ] Error response conforme a RFC 7807 (Problem Details)
- [ ] Transform per normalizzazione input (email lowercase, trim)
- [ ] Tipi TypeScript esportati per ogni schema
- [ ] Schema riusabili per pagination
