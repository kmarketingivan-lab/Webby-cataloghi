# CATALOGO-ERROR-HANDLING-ADVANCED

Catalogo Error Handling Avanzato per Next.js 14 - Espansione
§ RESULT PATTERN (neverthrow)
Ok/Err Types
typescript
import { ok, err, Result, ResultAsync } from 'neverthrow';

// Definizione di tipi di errore
type DatabaseError = { type: 'DATABASE_ERROR'; message: string; code: number };
type ValidationError = { type: 'VALIDATION_ERROR'; field: string; details: string };
type NetworkError = { type: 'NETWORK_ERROR'; status: number };

type AppError = DatabaseError | ValidationError | NetworkError;

// Funzioni che restituiscono Result
const validateUser = (data: any): Result<{ id: string; email: string }, ValidationError> => {
  if (!data.email.includes('@')) {
    return err({ 
      type: 'VALIDATION_ERROR', 
      field: 'email', 
      details: 'Invalid email format' 
    });
  }
  return ok({ id: '123', email: data.email });
};

const fetchUserFromDb = (id: string): ResultAsync<{ name: string }, DatabaseError> => {
  return ResultAsync.fromPromise(
    prisma.user.findUnique({ where: { id } }),
    (error): DatabaseError => ({
      type: 'DATABASE_ERROR',
      message: error instanceof Error ? error.message : 'Unknown database error',
      code: 500
    })
  );
};
Railway Oriented Programming
typescript
// Composizione di operazioni con railway pattern
class UserService {
  async createUser(
    input: UserInput
  ): Promise<Result<User, AppError>> {
    // Railway: ogni operazione si collega alla successiva
    return validateUser(input)
      .asyncAndThen(validated => this.checkEmailUniqueness(validated.email))
      .asyncAndThen(() => this.hashPassword(input.password))
      .asyncAndThen(hashedPassword => 
        this.saveUser({ ...input, password: hashedPassword })
      )
      .map(user => this.sanitizeUser(user))
      .mapErr(error => this.transformError(error));
  }

  private async saveUser(
    data: any
  ): Promise<Result<User, DatabaseError>> {
    try {
      const user = await prisma.user.create({ data });
      return ok(user);
    } catch (error) {
      return err({
        type: 'DATABASE_ERROR',
        message: 'Failed to save user',
        code: 500
      });
    }
  }
}

// Helper per railway programming
const railway = {
  combine: <T, E>(results: Result<T, E>[]): Result<T[], E> => {
    const values: T[] = [];
    for (const result of results) {
      if (result.isErr()) return err(result.error);
      values.push(result.value);
    }
    return ok(values);
  },

  sequential: async <T, E>(
    tasks: (() => Promise<Result<T, E>>)[]
  ): Promise<Result<T[], E>> => {
    const results: T[] = [];
    for (const task of tasks) {
      const result = await task();
      if (result.isErr()) return err(result.error);
      results.push(result.value);
    }
    return ok(results);
  }
};
Chaining Operations
typescript
// Chain con trasformazione
const processOrder = (orderId: string): ResultAsync<Order, AppError> => {
  return fetchOrder(orderId)
    .andThen(order => validateOrder(order))
    .andThen(order => checkInventory(order.items))
    .andThen(order => calculateTotal(order))
    .map(order => applyDiscounts(order))
    .map(order => generateInvoice(order))
    .mapErr(error => enrichError(error, { orderId }));
};

// Chain asincrono con early return
const checkoutFlow = async (
  userId: string, 
  cartId: string
): Promise<Result<CheckoutResult, AppError>> => {
  const result = await getUser(userId)
    .andThen(user => getCart(user.id, cartId))
    .asyncAndThen(cart => validateCart(cart))
    .asyncAndThen(cart => processPayment(cart))
    .asyncAndThen(payment => createOrder(payment, userId))
    .andThen(order => sendConfirmation(order));

  return result.match(
    success => ok({ order: success, status: 'COMPLETED' }),
    error => {
      // Logica di compensazione in caso di errore
      return rollbackCheckout(userId, cartId)
        .map(() => ({ status: 'FAILED', error }))
        .mapErr(rollbackError => ({
          type: 'ROLLBACK_ERROR',
          message: `Checkout failed and rollback also failed: ${rollbackError}`
        }));
    }
  );
};
Error Transformation
typescript
class ErrorTransformer {
  static toUserFriendly(error: AppError): UserFriendlyError {
    switch (error.type) {
      case 'DATABASE_ERROR':
        return {
          title: 'Service Unavailable',
          message: 'Please try again later',
          severity: 'high',
          retryable: true
        };
      case 'VALIDATION_ERROR':
        return {
          title: 'Invalid Input',
          message: `Please check the ${error.field} field`,
          severity: 'low',
          retryable: false
        };
      case 'NETWORK_ERROR':
        return {
          title: 'Connection Error',
          message: 'Please check your internet connection',
          severity: 'medium',
          retryable: true
        };
      default:
        return this.unknownError();
    }
  }

  static toLogFormat(error: AppError): LogEntry {
    const baseLog = {
      timestamp: new Date().toISOString(),
      errorType: error.type,
      context: {}
    };

    switch (error.type) {
      case 'DATABASE_ERROR':
        return {
          ...baseLog,
          severity: 'ERROR',
          details: { code: error.code, message: error.message },
          stack: new Error().stack
        };
      case 'VALIDATION_ERROR':
        return {
          ...baseLog,
          severity: 'WARN',
          details: { field: error.field, details: error.details }
        };
      default:
        return { ...baseLog, severity: 'ERROR', details: error };
    }
  }
}

// Middleware per trasformazione errori in Next.js
export const withErrorHandling = <T extends any[], R>(
  handler: (...args: T) => Promise<Result<R, AppError>>
) => {
  return async (...args: T): Promise<NextResponse> => {
    const result = await handler(...args);
    
    return result.match(
      (data) => NextResponse.json({ success: true, data }),
      (error) => {
        const userError = ErrorTransformer.toUserFriendly(error);
        const logEntry = ErrorTransformer.toLogFormat(error);
        
        // Log per monitoraggio
        logger.error(logEntry);
        
        return NextResponse.json(
          { success: false, error: userError },
          { status: this.getStatusCode(error) }
        );
      }
    );
  };
};
§ TYPED ERRORS
Custom Error Classes Hierarchy
typescript
// Base Error Class
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
  abstract readonly isOperational: boolean;
  
  readonly timestamp: Date;
  readonly context?: Record<string, any>;

  constructor(
    message: string,
    options: {
      cause?: Error;
      context?: Record<string, any>;
    } = {}
  ) {
    super(message, { cause: options.cause });
    this.name = this.constructor.name;
    this.timestamp = new Date();
    this.context = options.context;
    Error.captureStackTrace?.(this, this.constructor);
  }

  toJSON() {
    return {
      name: this.name,
      code: this.code,
      message: this.message,
      statusCode: this.statusCode,
      timestamp: this.timestamp,
      context: this.context,
      stack: this.stack,
      cause: this.cause
    };
  }
}

// Categoria: Validation Errors
class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;
  readonly isOperational = true;

  constructor(
    public readonly field: string,
    message: string,
    public readonly validationType?: string
  ) {
    super(`Validation failed for field ${field}: ${message}`);
  }
}

class RequiredFieldError extends ValidationError {
  readonly code = 'REQUIRED_FIELD';
  
  constructor(field: string) {
    super(field, 'This field is required', 'required');
  }
}

class InvalidFormatError extends ValidationError {
  readonly code = 'INVALID_FORMAT';
  
  constructor(field: string, expectedFormat: string) {
    super(field, `Must match format: ${expectedFormat}`, 'format');
  }
}

// Categoria: Business Logic Errors
class BusinessRuleError extends AppError {
  readonly code = 'BUSINESS_RULE_ERROR';
  readonly statusCode = 409;
  readonly isOperational = true;

  constructor(
    public readonly rule: string,
    message: string
  ) {
    super(`Business rule violation: ${rule} - ${message}`);
  }
}

class InsufficientFundsError extends BusinessRuleError {
  readonly code = 'INSUFFICIENT_FUNDS';
  
  constructor(required: number, available: number) {
    super(
      'MINIMUM_BALANCE',
      `Required: ${required}, Available: ${available}`
    );
  }
}

// Categoria: Infrastructure Errors
class DatabaseError extends AppError {
  readonly code = 'DATABASE_ERROR';
  readonly statusCode = 503;
  readonly isOperational = true;

  constructor(
    public readonly operation: string,
    message: string,
    options?: { cause?: Error; context?: Record<string, any> }
  ) {
    super(`Database operation failed: ${operation} - ${message}`, options);
  }
}

class UniqueConstraintError extends DatabaseError {
  readonly code = 'UNIQUE_CONSTRAINT';
  
  constructor(table: string, field: string, value: string) {
    super(
      'INSERT/UPDATE',
      `Duplicate value '${value}' for field '${field}' in table '${table}'`
    );
  }
}
Discriminated Union Errors
typescript
// Definizione di union type discriminato
type ApiError = 
  | { type: 'VALIDATION'; field: string; details: string; code: 'INVALID_FORMAT' | 'REQUIRED' }
  | { type: 'AUTHENTICATION'; reason: 'INVALID_TOKEN' | 'EXPIRED' | 'MISSING'; scope: string }
  | { type: 'AUTHORIZATION'; permission: string; resource: string; userId: string }
  | { type: 'RATE_LIMIT'; limit: number; resetAt: Date; endpoint: string }
  | { type: 'EXTERNAL_SERVICE'; service: string; status: number; response: any }
  | { type: 'INTERNAL'; errorId: string; timestamp: Date };

// Type guard per narrowing
const isValidationError = (error: ApiError): error is Extract<ApiError, { type: 'VALIDATION' }> => {
  return error.type === 'VALIDATION';
};

const isAuthenticationError = (error: ApiError): error is Extract<ApiError, { type: 'AUTHENTICATION' }> => {
  return error.type === 'AUTHENTICATION';
};

// Handler con discriminated union
class ErrorHandler {
  handle(error: ApiError): HandleResult {
    // Il type narrowing avviene automaticamente nello switch
    switch (error.type) {
      case 'VALIDATION':
        return this.handleValidationError(error);
        
      case 'AUTHENTICATION':
        return this.handleAuthenticationError(error);
        
      case 'AUTHORIZATION':
        return this.handleAuthorizationError(error);
        
      case 'RATE_LIMIT':
        return this.handleRateLimitError(error);
        
      case 'EXTERNAL_SERVICE':
        return this.handleExternalServiceError(error);
        
      case 'INTERNAL':
        return this.handleInternalError(error);
        
      default:
        // TypeScript garantisce exhaustive checking
        const _exhaustiveCheck: never = error;
        return _exhaustiveCheck;
    }
  }

  private handleValidationError(error: Extract<ApiError, { type: 'VALIDATION' }>): HandleResult {
    // TypeScript sa che error ha 'field' e 'details'
    return {
      statusCode: 400,
      body: {
        error: {
          code: error.code,
          field: error.field,
          message: `Validation failed: ${error.details}`
        }
      }
    };
  }

  private handleAuthenticationError(error: Extract<ApiError, { type: 'AUTHENTICATION' }>): HandleResult {
    return {
      statusCode: 401,
      body: {
        error: {
          reason: error.reason,
          scope: error.scope,
          message: `Authentication failed: ${error.reason}`
        }
      }
    };
  }
}
Error Type Narrowing
typescript
// Pattern matching avanzato per errori
class ErrorNarrower {
  // User-Defined Type Guards
  static isDatabaseError(error: unknown): error is DatabaseError {
    return error instanceof DatabaseError;
  }

  static isNetworkError(error: unknown): error is { status: number; url: string } {
    return (
      typeof error === 'object' &&
      error !== null &&
      'status' in error &&
      'url' in error
    );
  }

  static isValidationError(error: unknown): error is ValidationError {
    return error instanceof ValidationError;
  }

  // Narrowing con pattern matching funzionale
  static matchError<T>(
    error: unknown,
    patterns: {
      DatabaseError: (err: DatabaseError) => T;
      NetworkError: (err: { status: number; url: string }) => T;
      ValidationError: (err: ValidationError) => T;
      Default: (err: unknown) => T;
    }
  ): T {
    if (this.isDatabaseError(error)) {
      return patterns.DatabaseError(error);
    }
    
    if (this.isNetworkError(error)) {
      return patterns.NetworkError(error);
    }
    
    if (this.isValidationError(error)) {
      return patterns.ValidationError(error);
    }
    
    return patterns.Default(error);
  }

  // Narrowing per errori di API
  static narrowApiError(response: any): ApiError {
    if (response?.error?.type) {
      switch (response.error.type) {
        case 'validation':
          return {
            type: 'VALIDATION',
            field: response.error.field,
            details: response.error.details,
            code: response.error.code
          } as const;
          
        case 'authentication':
          return {
            type: 'AUTHENTICATION',
            reason: response.error.reason,
            scope: response.error.scope
          } as const;
          
        default:
          return {
            type: 'INTERNAL',
            errorId: crypto.randomUUID(),
            timestamp: new Date()
          } as const;
      }
    }
    
    // Fallback per errori sconosciuti
    return {
      type: 'INTERNAL',
      errorId: crypto.randomUUID(),
      timestamp: new Date()
    } as const;
  }
}

// Hook React per narrowing errori
export const useErrorHandler = () => {
  const handleError = useCallback((error: unknown) => {
    return ErrorNarrower.matchError(error, {
      DatabaseError: (err) => {
        // Logica specifica per DatabaseError
        logger.error('Database error', err);
        return toast.error('Database error occurred');
      },
      NetworkError: (err) => {
        // Logica specifica per NetworkError
        logger.warn('Network error', err);
        return toast.error('Network connection failed');
      },
      ValidationError: (err) => {
        // Logica specifica per ValidationError
        logger.info('Validation error', err);
        return toast.error(`Validation failed: ${err.field}`);
      },
      Default: (err) => {
        // Logica per errori sconosciuti
        logger.error('Unknown error', err);
        return toast.error('An unexpected error occurred');
      }
    });
  }, []);

  return { handleError };
};
Exhaustive Error Handling
typescript
// Utility per exhaustive checking
export const assertNever = (x: never): never => {
  throw new Error(`Unexpected object: ${x}`);
};

// Pattern per exhaustive handling in React components
type ComponentError = 
  | { type: 'LOADING_FAILED'; resource: string; retry: () => void }
  | { type: 'RENDER_ERROR'; component: string; fallback?: React.ReactNode }
  | { type: 'PROP_VALIDATION'; prop: string; expected: string }
  | { type: 'STATE_CORRUPTION'; module: string; recovery: () => void };

const ErrorDisplay: React.FC<{ error: ComponentError }> = ({ error }) => {
  switch (error.type) {
    case 'LOADING_FAILED':
      return (
        <div className="error-loading">
          <p>Failed to load {error.resource}</p>
          <button onClick={error.retry}>Retry</button>
        </div>
      );
      
    case 'RENDER_ERROR':
      return (
        <div className="error-render">
          <p>Component {error.component} failed to render</p>
          {error.fallback || <DefaultFallback />}
        </div>
      );
      
    case 'PROP_VALIDATION':
      return (
        <div className="error-prop">
          <p>Invalid prop {error.prop}. Expected: {error.expected}</p>
        </div>
      );
      
    case 'STATE_CORRUPTION':
      return (
        <div className="error-state">
          <p>State corruption detected in {error.module}</p>
          <button onClick={error.recovery}>Recover State</button>
        </div>
      );
      
    default:
      // TypeScript garantisce che tutti i casi siano gestiti
      return assertNever(error);
  }
};

// Exhaustive handling per API routes
export const exhaustiveApiHandler = async (
  req: NextRequest,
  handler: () => Promise<NextResponse>
): Promise<NextResponse> => {
  try {
    return await handler();
  } catch (error) {
    const classifiedError = classifyError(error);
    
    switch (classifiedError.category) {
      case 'VALIDATION':
        return NextResponse.json(
          { error: 'Validation failed', details: classifiedError.details },
          { status: 400 }
        );
        
      case 'AUTHENTICATION':
        return NextResponse.json(
          { error: 'Authentication required' },
          { status: 401 }
        );
        
      case 'AUTHORIZATION':
        return NextResponse.json(
          { error: 'Insufficient permissions' },
          { status: 403 }
        );
        
      case 'NOT_FOUND':
        return NextResponse.json(
          { error: 'Resource not found' },
          { status: 404 }
        );
        
      case 'CONFLICT':
        return NextResponse.json(
          { error: 'Resource conflict' },
          { status: 409 }
        );
        
      case 'RATE_LIMIT':
        return NextResponse.json(
          { error: 'Too many requests' },
          { status: 429 }
        );
        
      case 'SERVER_ERROR':
        // Log error interno
        logger.error('Server error', classifiedError);
        return NextResponse.json(
          { error: 'Internal server error', errorId: classifiedError.id },
          { status: 500 }
        );
        
      default:
        // Questo caso non dovrebbe mai verificarsi
        const _exhaustiveCheck: never = classifiedError;
        logger.fatal('Unhandled error category', _exhaustiveCheck);
        return NextResponse.json(
          { error: 'Unknown error' },
          { status: 500 }
        );
    }
  }
};

// Classificatore errori per exhaustive handling
const classifyError = (error: unknown): ClassifiedError => {
  if (error instanceof ValidationError) {
    return {
      category: 'VALIDATION',
      details: error.details,
      id: crypto.randomUUID()
    };
  }
  
  if (error instanceof AuthenticationError) {
    return {
      category: 'AUTHENTICATION',
      details: error.reason,
      id: crypto.randomUUID()
    };
  }
  
  // ... altre classificazioni
  
  // Default: errore server
  return {
    category: 'SERVER_ERROR',
    details: String(error),
    id: crypto.randomUUID()
  };
};
§ ERROR MONITORING DASHBOARD
Error Trends
typescript
// Dashboard per trend degli errori
interface ErrorTrendsDashboard {
  getDailyTrends(dateRange: DateRange): Promise<ErrorTrend[]>;
  getHourlyBreakdown(date: Date): Promise<HourlyErrorCount[]>;
  getErrorRateOverTime(interval: TimeInterval): Promise<ErrorRatePoint[]>;
  compareTrends(periodA: DateRange, periodB: DateRange): Promise<TrendComparison>;
}

class ErrorTrendsService implements ErrorTrendsDashboard {
  constructor(private readonly analyticsDb: AnalyticsDatabase) {}

  async getDailyTrends(dateRange: DateRange): Promise<ErrorTrend[]> {
    const trends = await this.analyticsDb.query(`
      SELECT 
        DATE(timestamp) as date,
        error_type,
        COUNT(*) as error_count,
        COUNT(DISTINCT user_id) as affected_users,
        AVG(resolution_time) as avg_resolution_time
      FROM error_events
      WHERE timestamp BETWEEN $1 AND $2
      GROUP BY DATE(timestamp), error_type
      ORDER BY date DESC, error_count DESC
    `, [dateRange.start, dateRange.end]);

    return trends.map(trend => ({
      date: trend.date,
      errorType: trend.error_type,
      count: parseInt(trend.error_count),
      affectedUsers: parseInt(trend.affected_users),
      avgResolutionTime: trend.avg_resolution_time,
      trendDirection: this.calculateTrendDirection(trend)
    }));
  }

  async getErrorRateOverTime(
    interval: TimeInterval
  ): Promise<ErrorRatePoint[]> {
    const intervalSql = this.getIntervalSql(interval);
    
    const rates = await this.analyticsDb.query(`
      WITH error_counts AS (
        SELECT 
          ${intervalSql} as time_bucket,
          COUNT(*) as error_count
        FROM error_events
        WHERE timestamp > NOW() - INTERVAL '7 days'
        GROUP BY time_bucket
      ),
      request_counts AS (
        SELECT 
          ${intervalSql} as time_bucket,
          COUNT(*) as request_count
        FROM request_logs
        WHERE timestamp > NOW() - INTERVAL '7 days'
        GROUP BY time_bucket
      )
      SELECT 
        COALESCE(e.time_bucket, r.time_bucket) as time_bucket,
        COALESCE(e.error_count, 0) as error_count,
        COALESCE(r.request_count, 0) as request_count,
        CASE 
          WHEN COALESCE(r.request_count, 0) = 0 THEN 0
          ELSE (COALESCE(e.error_count, 0) * 100.0 / r.request_count)
        END as error_rate
      FROM error_counts e
      FULL OUTER JOIN request_counts r ON e.time_bucket = r.time_bucket
      ORDER BY time_bucket
    `);

    return rates.map(rate => ({
      timestamp: rate.time_bucket,
      errorCount: rate.error_count,
      requestCount: rate.request_count,
      errorRate: parseFloat(rate.error_rate),
      severity: this.calculateSeverity(rate.error_rate)
    }));
  }

  private calculateTrendDirection(trend: any): 'increasing' | 'decreasing' | 'stable' {
    // Logica per determinare la direzione del trend
    const previousCount = this.getPreviousPeriodCount(trend);
    const percentageChange = ((trend.error_count - previousCount) / previousCount) * 100;
    
    if (Math.abs(percentageChange) < 5) return 'stable';
    return percentageChange > 0 ? 'increasing' : 'decreasing';
  }
}

// React Component per Error Trends
export const ErrorTrendsDashboard: React.FC = () => {
  const [trends, setTrends] = useState<ErrorTrend[]>([]);
  const [selectedPeriod, setSelectedPeriod] = useState<DateRange>({
    start: subDays(new Date(), 7),
    end: new Date()
  });

  useEffect(() => {
    const loadTrends = async () => {
      const data = await errorTrendsService.getDailyTrends(selectedPeriod);
      setTrends(data);
    };
    loadTrends();
  }, [selectedPeriod]);

  return (
    <DashboardCard title="Error Trends">
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Trend Chart */}
        <div className="col-span-2">
          <ErrorTrendChart trends={trends} />
        </div>
        
        {/* Top Errors */}
        <div>
          <h3 className="text-lg font-semibold mb-4">Top Errors</h3>
          <ErrorList errors={trends.slice(0, 5)} />
        </div>
        
        {/* Trend Summary */}
        <div>
          <h3 className="text-lg font-semibold mb-4">Trend Summary</h3>
          <TrendSummary trends={trends} />
        </div>
      </div>
    </DashboardCard>
  );
};
Error Grouping
typescript
// Servizio per grouping degli errori
interface ErrorGroup {
  id: string;
  fingerprint: string;
  title: string;
  firstSeen: Date;
  lastSeen: Date;
  count: number;
  affectedUsers: number;
  status: 'active' | 'resolved' | 'ignored';
  assignedTo?: string;
  tags: string[];
}

class ErrorGrouper {
  // Genera fingerprint per grouping basato su stack trace e contesto
  generateFingerprint(error: ErrorEvent): string {
    const components = [
      this.normalizeStackTrace(error.stack),
      error.errorType,
      this.extractKeyContext(error.context),
      error.sourceFile?.split('/').pop()
    ].filter(Boolean);

    return crypto
      .createHash('sha256')
      .update(components.join('::'))
      .digest('hex')
      .substring(0, 16);
  }

  // Algoritmo di grouping intelligente
  async groupSimilarErrors(
    errors: ErrorEvent[],
    similarityThreshold = 0.8
  ): Promise<ErrorGroup[]> {
    const groups: Map<string, ErrorGroup> = new Map();

    for (const error of errors) {
      const fingerprint = this.generateFingerprint(error);
      
      if (groups.has(fingerprint)) {
        const group = groups.get(fingerprint)!;
        group.count++;
        group.lastSeen = new Date(Math.max(group.lastSeen.getTime(), error.timestamp.getTime()));
        group.affectedUsers = await this.calculateAffectedUsers(group.id);
      } else {
        groups.set(fingerprint, {
          id: fingerprint,
          fingerprint,
          title: this.generateGroupTitle(error),
          firstSeen: error.timestamp,
          lastSeen: error.timestamp,
          count: 1,
          affectedUsers: 1,
          status: 'active',
          tags: this.extractTags(error),
          assignedTo: this.assignToTeam(error)
        });
      }
    }

    return Array.from(groups.values());
  }

  // Merge di gruppi simili
  async mergeSimilarGroups(
    groups: ErrorGroup[],
    similarityThreshold = 0.7
  ): Promise<ErrorGroup[]> {
    const mergedGroups: ErrorGroup[] = [];
    const processed = new Set<string>();

    for (let i = 0; i < groups.length; i++) {
      if (processed.has(groups[i].id)) continue;

      const primaryGroup = { ...groups[i] };
      processed.add(primaryGroup.id);

      for (let j = i + 1; j < groups.length; j++) {
        if (processed.has(groups[j].id)) continue;

        const similarity = this.calculateGroupSimilarity(
          primaryGroup,
          groups[j]
        );

        if (similarity >= similarityThreshold) {
          // Merge dei gruppi
          primaryGroup.count += groups[j].count;
          primaryGroup.affectedUsers += groups[j].affectedUsers;
          primaryGroup.firstSeen = new Date(
            Math.min(primaryGroup.firstSeen.getTime(), groups[j].firstSeen.getTime())
          );
          primaryGroup.lastSeen = new Date(
            Math.max(primaryGroup.lastSeen.getTime(), groups[j].lastSeen.getTime())
          );
          primaryGroup.tags = [...new Set([...primaryGroup.tags, ...groups[j].tags])];
          
          processed.add(groups[j].id);
        }
      }

      mergedGroups.push(primaryGroup);
    }

    return mergedGroups;
  }

  private calculateGroupSimilarity(
    groupA: ErrorGroup,
    groupB: ErrorGroup
  ): number {
    const factors = [
      // Similarità dello stack trace
      this.similarityScore(groupA.title, groupB.title, 0.4),
      
      // Similarità del contesto
      this.contextSimilarity(groupA.tags, groupB.tags, 0.3),
      
      // Similarità temporale
      this.temporalSimilarity(groupA.firstSeen, groupB.firstSeen, 0.2),
      
      // Similarità degli utenti affetti
      this.userSimilarity(groupA.affectedUsers, groupB.affectedUsers, 0.1)
    ];

    return factors.reduce((sum, score) => sum + score, 0);
  }
}

// Component React per Error Grouping
export const ErrorGroupsDashboard: React.FC = () => {
  const [groups, setGroups] = useState<ErrorGroup[]>([]);
  const [selectedGroup, setSelectedGroup] = useState<string | null>(null);

  const handleMergeGroups = async (groupIds: string[]) => {
    const groupsToMerge = groups.filter(g => groupIds.includes(g.id));
    const merged = await errorGrouper.mergeSimilarGroups(groupsToMerge);
    
    // Aggiorna lo stato
    setGroups(prev => [
      ...prev.filter(g => !groupIds.includes(g.id)),
      ...merged
    ]);
  };

  return (
    <div className="space-y-6">
      <div className="flex justify-between items-center">
        <h2 className="text-2xl font-bold">Error Groups</h2>
        <GroupingControls onMerge={handleMergeGroups} />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Lista Gruppi */}
        <div className="lg:col-span-2">
          <ErrorGroupsTable
            groups={groups}
            selectedGroup={selectedGroup}
            onSelectGroup={setSelectedGroup}
          />
        </div>

        {/* Dettaglio Gruppo */}
        <div>
          {selectedGroup ? (
            <ErrorGroupDetail
              group={groups.find(g => g.id === selectedGroup)!}
              onUpdate={(updates) => {
                setGroups(prev => prev.map(g => 
                  g.id === selectedGroup ? { ...g, ...updates } : g
                ));
              }}
            />
          ) : (
            <div className="p-6 border rounded-lg">
              <p className="text-gray-500">Select a group to view details</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};
Affected Users Count
typescript
// Servizio per tracciamento utenti affetti
interface AffectedUser {
  userId: string;
  sessionId: string;
  firstErrorTime: Date;
  lastErrorTime: Date;
  errorCount: number;
  errorTypes: string[];
  deviceInfo: DeviceInfo;
  recoveryAttempts: number;
  status: 'active' | 'recovered' | 'escalated';
}

class AffectedUsersTracker {
  constructor(
    private readonly redis: RedisClient,
    private readonly db: Database
  ) {}

  // Traccia un utente affetto da errore
  async trackAffectedUser(
    error: ErrorEvent,
    userInfo: UserInfo
  ): Promise<void> {
    const key = `affected_user:${error.fingerprint}:${userInfo.userId}`;
    
    const existing = await this.redis.get(key);
    if (existing) {
      const data = JSON.parse(existing);
      data.errorCount += 1;
      data.lastErrorTime = new Date().toISOString();
      data.errorTypes = [...new Set([...data.errorTypes, error.errorType])];
      
      await this.redis.setex(key, 86400, JSON.stringify(data)); // 24h TTL
      
      // Check per escalation
      if (data.errorCount >= 5) {
        await this.escalateUser(data);
      }
    } else {
      const affectedUser: AffectedUser = {
        userId: userInfo.userId,
        sessionId: userInfo.sessionId,
        firstErrorTime: new Date(),
        lastErrorTime: new Date(),
        errorCount: 1,
        errorTypes: [error.errorType],
        deviceInfo: userInfo.deviceInfo,
        recoveryAttempts: 0,
        status: 'active'
      };
      
      await this.redis.setex(key, 86400, JSON.stringify(affectedUser));
      await this.db.affectedUsers.create({ data: affectedUser });
    }
  }

  // Ottieni statistiche utenti affetti
  async getAffectedUsersStats(
    errorFingerprint?: string,
    timeRange?: DateRange
  ): Promise<AffectedUsersStats> {
    const query = this.db.affectedUsers.groupBy({
      by: ['status', 'errorTypes'],
      where: {
        ...(errorFingerprint && { errorTypes: { has: errorFingerprint } }),
        ...(timeRange && {
          firstErrorTime: {
            gte: timeRange.start,
            lte: timeRange.end
          }
        })
      },
      _count: {
        userId: true
      },
      _avg: {
        errorCount: true
      }
    });

    const stats = await query;
    
    return {
      totalAffected: stats.reduce((sum, s) => sum + s._count.userId, 0),
      byStatus: stats.reduce((acc, s) => ({
        ...acc,
        [s.status]: s._count.userId
      }), {}),
      avgErrorsPerUser: stats[0]?._avg.errorCount || 0,
      recoveryRate: await this.calculateRecoveryRate(stats)
    };
  }

  // Identifica pattern di utenti affetti
  async identifyUserPatterns(): Promise<UserPattern[]> {
    const patterns = await this.db.$queryRaw<UserPattern[]>`
      SELECT 
        device_info->>'os' as os,
        device_info->>'browser' as browser,
        ARRAY_AGG(DISTINCT error_type) as common_errors,
        COUNT(DISTINCT user_id) as user_count,
        AVG(error_count) as avg_errors
      FROM affected_users
      WHERE first_error_time > NOW() - INTERVAL '7 days'
      GROUP BY os, browser
      HAVING COUNT(DISTINCT user_id) > 5
      ORDER BY user_count DESC
    `;

    return patterns.map(pattern => ({
      ...pattern,
      severity: this.calculatePatternSeverity(pattern)
    }));
  }

  // Notifica utenti affetti quando l'errore è risolto
  async notifyRecoveredUsers(
    errorFingerprint: string,
    resolutionDetails: string
  ): Promise<void> {
    const affectedUsers = await this.db.affectedUsers.findMany({
      where: {
        errorTypes: { has: errorFingerprint },
        status: 'active'
      },
      select: { userId: true, email: true }
    });

    for (const user of affectedUsers) {
      await this.sendRecoveryNotification(user.email, {
        errorFingerprint,
        resolutionDetails,
        timestamp: new Date()
      });
      
      await this.db.affectedUsers.update({
        where: { userId: user.userId },
        data: { status: 'recovered' }
      });
    }
  }
}

// Component React per visualizzazione utenti affetti
export const AffectedUsersDashboard: React.FC<{ errorGroupId?: string }> = ({ errorGroupId }) => {
  const [stats, setStats] = useState<AffectedUsersStats | null>(null);
  const [users, setUsers] = useState<AffectedUser[]>([]);
  const [patterns, setPatterns] = useState<UserPattern[]>([]);

  useEffect(() => {
    const loadData = async () => {
      const [statsData, usersData, patternsData] = await Promise.all([
        affectedUsersTracker.getAffectedUsersStats(errorGroupId),
        affectedUsersTracker

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-02-ERROR-HANDLING-ADVANCED
Prompt ID: 2 / 19
Parte: 2
Exported: 2026-02-06T16:28:06.979Z
Characters: 602
════════════════════════════════════════════════════════════

     }
      }
    } catch (localError) {
      console.warn('Local storage recovery failed:', localError);
    }

    return null;
  }

  // Hook React per auto-salvataggio
  export const useAutoSave = <T,>(
    initialData: T,
    options: UseAutoSaveOptions<T>
  ) => {
    const [data, setData] = useState<T>(initialData);
    const [isSaving, setIsSaving] = useState(false);
    const [lastSaved, setLastSaved] = useState<Date | null>(null);
    const [draftAvailable, setDraftAvailable] = useState(false);

    const autoSaveManagerRef = useRef<AutoSaveManager<T> | null>(null);

    useEffect(()

## § ADVANCED PATTERNS: ERROR HANDLING ADVANCED

### Server Actions con Validazione

```typescript
// app/actions/error-handling-advanced.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### ERROR HANDLING ADVANCED - Utility Helper #590

```typescript
// lib/utils/error-handling-advanced-helper-590.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #893

```typescript
// lib/utils/error-handling-advanced-helper-893.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #209

```typescript
// lib/utils/error-handling-advanced-helper-209.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #925

```typescript
// lib/utils/error-handling-advanced-helper-925.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #949

```typescript
// lib/utils/error-handling-advanced-helper-949.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #574

```typescript
// lib/utils/error-handling-advanced-helper-574.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #92

```typescript
// lib/utils/error-handling-advanced-helper-92.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #33

```typescript
// lib/utils/error-handling-advanced-helper-33.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #613

```typescript
// lib/utils/error-handling-advanced-helper-613.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #153

```typescript
// lib/utils/error-handling-advanced-helper-153.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #335

```typescript
// lib/utils/error-handling-advanced-helper-335.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #360

```typescript
// lib/utils/error-handling-advanced-helper-360.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #486

```typescript
// lib/utils/error-handling-advanced-helper-486.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #925

```typescript
// lib/utils/error-handling-advanced-helper-925.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #727

```typescript
// lib/utils/error-handling-advanced-helper-727.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<ERRORHANDLINGADVANCEDConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### ERROR HANDLING ADVANCED - Utility Helper #53

```typescript
// lib/utils/error-handling-advanced-helper-53.ts
import { z } from "zod";

interface ERRORHANDLINGADVANCEDConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class ERRORHANDLINGADVANCEDProcessor<TInput, TOutput> {
  private config: ERRORHANDLINGADVANCEDConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<ERRORHANDLINGADVANCEDConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {