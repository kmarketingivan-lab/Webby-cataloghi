# CATALOGO FEATURE-FLAGS v1
## Documentazione Tecnica Completa per Feature Flags in React/Next.js

---

## §1. FEATURE FLAG SOLUTIONS

### **Tabella Comparativa Dettagliata**

| Solution | Type | Cost | Complexity | Features | Scalability | Self-hosted | Best For |
|----------|------|------|------------|----------|-------------|-------------|----------|
| **LaunchDarkly** | SaaS | $$$ (starts at $9/user/mo) | Low | ⭐⭐⭐⭐⭐ | Excellent | No | Enterprise, mission-critical features |
| **Flagsmith** | Open-source/SaaS | $$ (OSS free, SaaS $) | Medium | ⭐⭐⭐⭐ | Very good | ✅ | Teams wanting self-hosted option |
| **Unleash** | Open-source | Free | Medium | ⭐⭐⭐⭐ | Very good | ✅ | Self-hosted, large organizations |
| **PostHog** | SaaS (analytics) | $$ (free tier) | Low | ⭐⭐⭐⭐ | Good | Limited | Teams already using PostHog analytics |
| **Custom (Database)** | Self-built | Free | Medium | ⭐⭐⭐ | Good | ✅ | Simple needs, full control |
| **Environment Variables** | Built-in | Free | Lowest | ⭐ | Poor | ✅ | Static flags, simple toggles |
| **Vercel Edge Config** | SaaS | $ (included) | Low | ⭐⭐ | Good | No | Vercel users, simple flags |

### **Decision Tree**

```
┌─ Budget? ──────────────────┐
│                            │
│ $$$ Enterprise budget      │ → LaunchDarkly
│ $$ Mid-range budget        │ → Flagsmith or PostHog
│ $ Free/OSS preference      │ → Unleash or Custom
│                            │
└────────────────────────────┘
          │
          ▼
┌─ Self-hosted needed? ──────┐
│                            │
│ Yes, on-premise           │ → Unleash or Flagsmith
│ No, cloud is fine         │ → LaunchDarkly or PostHog
│ Already on Vercel         │ → Vercel Edge Config
│                            │
└────────────────────────────┘
```

### **Raccomandazione per Next.js**

```typescript
const RECOMMENDED_APPROACH = {
  // Per progetti semplici:
  simple: {
    approach: 'Environment variables + custom logic',
    pros: 'Free, simple, no dependencies',
    cons: 'No runtime changes, requires deployment',
    useWhen: 'Few flags, static targeting, small team'
  },
  
  // Per progetti medi:
  medium: {
    approach: 'Custom database system',
    pros: 'Full control, self-hosted, free',
    cons: 'Maintenance overhead, build targeting rules',
    useWhen: 'Need runtime changes, moderate complexity'
  },
  
  // Per progetti complessi:
  complex: {
    approach: 'Unleash self-hosted',
    pros: 'Battle-tested, good SDKs, OSS',
    cons: 'Self-hosting overhead',
    useWhen: 'Complex targeting, A/B testing needed'
  },
  
  // Per progetti enterprise:
  enterprise: {
    approach: 'LaunchDarkly or Flagsmith SaaS',
    pros: 'Managed service, support, advanced features',
    cons: 'Cost, vendor lock-in',
    useWhen: 'Large team, mission-critical, need support'
  }
};
```

---

## §2. CUSTOM FEATURE FLAGS SYSTEM

### 2.1 Feature Flag Data Model Completo

```prisma
// schema.prisma - Feature Flags System
model FeatureFlag {
  id            String        @id @default(cuid())
  
  // Basic info
  key           String        @unique
  name          String
  description   String?
  
  // Status
  enabled       Boolean       @default(false)
  archived      Boolean       @default(false)
  
  // Targeting configuration
  targeting     Json?         // Targeting rules (see TargetingRules interface)
  percentage    Int?          @default(0) // 0-100
  
  // Environments
  environments  String[]      @default(["production"])
  defaultValue  Boolean       @default(false) // Fallback value
  
  // Metadata
  type          FeatureFlagType @default(BOOLEAN)
  tags          String[]
  createdBy     String
  updatedBy     String
  
  // Timestamps
  createdAt     DateTime      @default(now())
  updatedAt     DateTime      @updatedAt
  
  // Audit
  lastEvaluatedAt DateTime?
  evaluationCount Int         @default(0)
  
  // Relations
  overrides     FeatureFlagOverride[]
  auditLogs     FeatureFlagAuditLog[]
  
  // Indexes
  @@index([key])
  @@index([enabled])
  @@index([archived])
  @@index([environments])
  @@index([type])
  @@index([tags])
}

model FeatureFlagOverride {
  id          String   @id @default(cuid())
  
  // Target reference
  flagKey     String
  flag        FeatureFlag @relation(fields: [flagKey], references: [key], onDelete: Cascade)
  
  // Target identification
  targetType  TargetType // 'user', 'organization', 'session', 'ip'
  targetId    String
  targetName  String?    // For display purposes
  
  // Override value
  enabled     Boolean
  
  // Expiration
  expiresAt   DateTime?
  
  // Metadata
  reason      String?
  createdBy   String
  
  // Timestamps
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
  
  // Unique constraint
  @@unique([flagKey, targetType, targetId])
  
  // Indexes
  @@index([flagKey])
  @@index([targetType, targetId])
  @@index([expiresAt])
}

model FeatureFlagAuditLog {
  id          String         @id @default(cuid())
  flagKey     String
  flag        FeatureFlag    @relation(fields: [flagKey], references: [key])
  
  // Action
  action      AuditAction
  changes     Json?          // Before/after values
  metadata    Json?          // Additional context
  
  // User context
  userId      String?
  userEmail   String?
  userAgent   String?
  ipAddress   String?
  
  // Timestamp
  createdAt   DateTime       @default(now())
  
  // Indexes
  @@index([flagKey])
  @@index([action])
  @@index([userId])
  @@index([createdAt])
}

// Enums
enum FeatureFlagType {
  BOOLEAN      // Simple on/off
  PERCENTAGE   // Gradual rollout
  MULTIVARIATE // A/B testing with variants
  EXPERIMENT   // A/B test with analytics
}

enum TargetType {
  USER
  ORGANIZATION
  SESSION
  IP_ADDRESS
  EMAIL_DOMAIN
  CUSTOM_ATTRIBUTE
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  ENABLE
  DISABLE
  OVERRIDE_ADD
  OVERRIDE_REMOVE
  EVALUATION
}
```

### 2.2 Targeting Rules Schema

```typescript
// lib/feature-flags/types.ts
export interface TargetingRules {
  // Environment-based
  environment?: string[];
  
  // User targeting
  users?: {
    ids?: string[];
    emails?: string[];
    emailDomains?: string[];
  };
  
  // Organization targeting
  organizations?: {
    ids?: string[];
    names?: string[];
  };
  
  // Geography
  geography?: {
    countries?: string[];
    regions?: string[];
    cities?: string[];
  };
  
  // Device/browser
  device?: {
    types?: ('mobile' | 'tablet' | 'desktop')[];
    browsers?: string[];
    os?: string[];
  };
  
  // Custom attributes
  attributes?: Array<{
    key: string;
    operator: 'equals' | 'not_equals' | 'contains' | 'starts_with' | 'ends_with' | 'greater_than' | 'less_than';
    value: any;
  }>;
  
  // Date-based
  dateRange?: {
    start?: Date;
    end?: Date;
    timezone?: string;
  };
  
  // Percentage rollout
  percentage?: number; // 0-100
  
  // Compound rules
  rules?: Array<{
    operator: 'AND' | 'OR' | 'NOT';
    conditions: TargetingRules[];
  }>;
}

export interface EvaluationContext {
  // User context
  userId?: string;
  userEmail?: string;
  userName?: string;
  
  // Organization context
  organizationId?: string;
  organizationName?: string;
  
  // Session context
  sessionId?: string;
  
  // Device/geo context
  ipAddress?: string;
  userAgent?: string;
  country?: string;
  region?: string;
  city?: string;
  timezone?: string;
  
  // Custom attributes
  attributes?: Record<string, any>;
  
  // Environment
  environment?: string;
  
  // For percentage rollout
  stableIdentifier?: string; // For consistent assignment
}

export interface FeatureFlagConfig {
  key: string;
  name: string;
  description?: string;
  enabled: boolean;
  type: FeatureFlagType;
  targeting?: TargetingRules;
  percentage?: number;
  environments?: string[];
  defaultValue?: boolean;
  tags?: string[];
  variants?: Record<string, any>; // For multivariate flags
}

export interface EvaluationResult {
  enabled: boolean;
  flagKey: string;
  reason: string;
  variant?: string;
  value?: any;
  metadata: {
    evaluatedAt: Date;
    targetingMatched: boolean;
    percentageApplied: boolean;
    overrideApplied?: boolean;
  };
}
```

### 2.3 Feature Flag Service Completo

```typescript
// lib/services/feature-flag-service.ts
import { PrismaClient, FeatureFlag, FeatureFlagType } from '@prisma/client';
import { z } from 'zod';
import { 
  TargetingRules, 
  EvaluationContext, 
  FeatureFlagConfig,
  EvaluationResult 
} from '@/lib/feature-flags/types';
import { hashString } from '@/lib/utils/crypto';
import { redis } from '@/lib/redis';

const prisma = new PrismaClient();

// ============ VALIDATION SCHEMAS ============
const FeatureFlagCreateSchema = z.object({
  key: z.string()
    .min(1)
    .max(100)
    .regex(/^[a-z0-9-_]+$/, 'Key must contain only lowercase letters, numbers, hyphens, and underscores'),
  
  name: z.string().min(1).max(200),
  description: z.string().max(1000).optional(),
  
  type: z.nativeEnum(FeatureFlagType).default(FeatureFlagType.BOOLEAN),
  enabled: z.boolean().default(false),
  
  targeting: z.any().optional(), // Will validate with TargetingRulesSchema
  percentage: z.number().min(0).max(100).optional(),
  
  environments: z.array(z.string()).default(['production']),
  defaultValue: z.boolean().default(false),
  tags: z.array(z.string()).optional(),
  
  variants: z.record(z.any()).optional(),
});

const TargetingRulesSchema = z.object({
  environment: z.array(z.string()).optional(),
  users: z.object({
    ids: z.array(z.string()).optional(),
    emails: z.array(z.string().email()).optional(),
    emailDomains: z.array(z.string()).optional(),
  }).optional(),
  organizations: z.object({
    ids: z.array(z.string()).optional(),
    names: z.array(z.string()).optional(),
  }).optional(),
  geography: z.object({
    countries: z.array(z.string().length(2)).optional(),
    regions: z.array(z.string()).optional(),
    cities: z.array(z.string()).optional(),
  }).optional(),
  device: z.object({
    types: z.array(z.enum(['mobile', 'tablet', 'desktop'])).optional(),
    browsers: z.array(z.string()).optional(),
    os: z.array(z.string()).optional(),
  }).optional(),
  attributes: z.array(z.object({
    key: z.string(),
    operator: z.enum(['equals', 'not_equals', 'contains', 'starts_with', 'ends_with', 'greater_than', 'less_than']),
    value: z.any(),
  })).optional(),
  dateRange: z.object({
    start: z.date().optional(),
    end: z.date().optional(),
    timezone: z.string().optional(),
  }).optional(),
  percentage: z.number().min(0).max(100).optional(),
  rules: z.array(z.object({
    operator: z.enum(['AND', 'OR', 'NOT']),
    conditions: z.array(z.any()).min(1),
  })).optional(),
});

// ============ FEATURE FLAG SERVICE ============
export class FeatureFlagService {
  private readonly CACHE_TTL = 60; // 60 seconds
  private readonly CACHE_PREFIX = 'feature-flag';
  
  // ============ PUBLIC API ============
  
  /**
   * Check if a feature flag is enabled for a given context
   */
  async isEnabled(
    flagKey: string,
    context: EvaluationContext = {},
    environment: string = process.env.NODE_ENV || 'development'
  ): Promise<boolean> {
    const result = await this.evaluate(flagKey, context, environment);
    return result.enabled;
  }
  
  /**
   * Evaluate a feature flag with detailed result
   */
  async evaluate(
    flagKey: string,
    context: EvaluationContext = {},
    environment: string = process.env.NODE_ENV || 'development'
  ): Promise<EvaluationResult> {
    try {
      // Get flag from cache or database
      const flag = await this.getFlag(flagKey);
      
      if (!flag) {
        return this.createNotFoundResult(flagKey);
      }
      
      // Check if flag is archived
      if (flag.archived) {
        return this.createArchivedResult(flag);
      }
      
      // Check environment
      if (!flag.environments.includes(environment)) {
        return this.createEnvironmentMismatchResult(flag, environment);
      }
      
      // Check for override
      const override = await this.getOverride(flagKey, context);
      if (override) {
        return this.createOverrideResult(flag, override);
      }
      
      // Evaluate targeting rules
      let enabled = flag.enabled;
      let reason = 'global_enabled';
      
      if (!flag.enabled) {
        return this.createDisabledResult(flag);
      }
      
      // Apply targeting rules if present
      if (flag.targeting) {
        const targetingResult = this.evaluateTargeting(
          flag.targeting as TargetingRules,
          context,
          environment
        );
        
        if (targetingResult.matched) {
          enabled = true;
          reason = 'targeting_matched';
        } else {
          enabled = false;
          reason = 'targeting_not_matched';
        }
      }
      
      // Apply percentage rollout if flag is enabled
      if (enabled && flag.percentage && flag.percentage < 100) {
        const inPercentage = this.isInPercentage(
          flag.percentage,
          context,
          flagKey
        );
        
        if (inPercentage) {
          reason = 'percentage_matched';
        } else {
          enabled = false;
          reason = 'percentage_not_matched';
        }
      }
      
      // Update evaluation stats
      await this.recordEvaluation(flagKey);
      
      return {
        enabled,
        flagKey,
        reason,
        metadata: {
          evaluatedAt: new Date(),
          targetingMatched: reason === 'targeting_matched',
          percentageApplied: flag.percentage !== null && flag.percentage !== undefined,
        },
      };
      
    } catch (error) {
      console.error(`Feature flag evaluation error for ${flagKey}:`, error);
      
      // Fallback to default value or false
      const flag = await this.getFlag(flagKey);
      const defaultValue = flag?.defaultValue ?? false;
      
      return {
        enabled: defaultValue,
        flagKey,
        reason: 'error_fallback',
        metadata: {
          evaluatedAt: new Date(),
          targetingMatched: false,
          percentageApplied: false,
          error: error instanceof Error ? error.message : 'Unknown error',
        },
      };
    }
  }
  
  /**
   * Get all flags for a context (optimized for client-side)
   */
  async getAllFlags(
    context: EvaluationContext = {},
    environment: string = process.env.NODE_ENV || 'development'
  ): Promise<Record<string, boolean>> {
    const flags = await this.getAllActiveFlags();
    const results: Record<string, boolean> = {};
    
    // Evaluate each flag
    for (const flag of flags) {
      // Fast path: skip flags not in environment
      if (!flag.environments.includes(environment)) {
        results[flag.key] = flag.defaultValue || false;
        continue;
      }
      
      // Fast path: check override first
      const override = await this.getOverride(flag.key, context);
      if (override !== null) {
        results[flag.key] = override;
        continue;
      }
      
      // Fast path: globally disabled
      if (!flag.enabled) {
        results[flag.key] = false;
        continue;
      }
      
      // Full evaluation
      const result = await this.evaluate(flag.key, context, environment);
      results[flag.key] = result.enabled;
    }
    
    return results;
  }
  
  /**
   * Get variant for multivariate flag
   */
  async getVariant(
    flagKey: string,
    context: EvaluationContext = {},
    environment: string = 'production'
  ): Promise<string | null> {
    const flag = await this.getFlag(flagKey);
    
    if (!flag || flag.type !== FeatureFlagType.MULTIVARIATE) {
      return null;
    }
    
    // Check if flag is enabled for this context
    const isEnabled = await this.isEnabled(flagKey, context, environment);
    if (!isEnabled) {
      return null;
    }
    
    // Get stable identifier for consistent assignment
    const stableId = context.stableIdentifier || 
                     context.userId || 
                     context.sessionId || 
                     Math.random().toString();
    
    // Hash and assign variant
    const hash = hashString(stableId + flagKey);
    const variants = Object.keys(flag.variants || {});
    
    if (variants.length === 0) {
      return null;
    }
    
    // Use hash to deterministically select variant
    const index = Math.abs(hash) % variants.length;
    return variants[index];
  }
  
  // ============ ADMIN METHODS ============
  
  /**
   * Create a new feature flag
   */
  async createFlag(data: z.infer<typeof FeatureFlagCreateSchema>, createdBy: string): Promise<FeatureFlag> {
    try {
      // Validate input
      const validated = FeatureFlagCreateSchema.parse(data);
      
      // Validate targeting rules if provided
      if (validated.targeting) {
        TargetingRulesSchema.parse(validated.targeting);
      }
      
      // Create flag
      const flag = await prisma.featureFlag.create({
        data: {
          ...validated,
          createdBy,
          updatedBy: createdBy,
        },
      });
      
      // Clear cache
      await this.clearFlagCache(flag.key);
      
      // Audit log
      await prisma.featureFlagAuditLog.create({
        data: {
          flagKey: flag.key,
          action: 'CREATE',
          userId: createdBy,
          changes: { from: null, to: validated },
        },
      });
      
      return flag;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid feature flag data', error.errors);
      }
      throw error;
    }
  }
  
  /**
   * Update an existing feature flag
   */
  async updateFlag(
    key: string,
    data: Partial<z.infer<typeof FeatureFlagCreateSchema>>,
    updatedBy: string
  ): Promise<FeatureFlag> {
    try {
      // Get existing flag
      const existing = await prisma.featureFlag.findUnique({
        where: { key },
      });
      
      if (!existing) {
        throw new NotFoundError(`Feature flag ${key} not found`);
      }
      
      // Validate partial data
      const validated = FeatureFlagCreateSchema.partial().parse(data);
      
      // Validate targeting rules if provided
      if (validated.targeting) {
        TargetingRulesSchema.parse(validated.targeting);
      }
      
      // Update flag
      const flag = await prisma.featureFlag.update({
        where: { key },
        data: {
          ...validated,
          updatedBy,
        },
      });
      
      // Clear cache
      await this.clearFlagCache(key);
      
      // Audit log
      await prisma.featureFlagAuditLog.create({
        data: {
          flagKey: key,
          action: 'UPDATE',
          userId: updatedBy,
          changes: { from: existing, to: validated },
        },
      });
      
      return flag;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid feature flag data', error.errors);
      }
      throw error;
    }
  }
  
  /**
   * Set an override for a specific target
   */
  async setOverride(
    flagKey: string,
    targetType: string,
    targetId: string,
    enabled: boolean,
    options?: {
      expiresAt?: Date;
      reason?: string;
      createdBy?: string;
      targetName?: string;
    }
  ): Promise<void> {
    try {
      // Check if flag exists
      const flag = await prisma.featureFlag.findUnique({
        where: { key: flagKey },
      });
      
      if (!flag) {
        throw new NotFoundError(`Feature flag ${flagKey} not found`);
      }
      
      const { expiresAt, reason, createdBy = 'system', targetName } = options || {};
      
      // Create or update override
      await prisma.featureFlagOverride.upsert({
        where: {
          flagKey_targetType_targetId: {
            flagKey,
            targetType,
            targetId,
          },
        },
        create: {
          flagKey,
          targetType,
          targetId,
          targetName,
          enabled,
          expiresAt,
          reason,
          createdBy,
        },
        update: {
          enabled,
          expiresAt,
          reason,
          updatedAt: new Date(),
        },
      });
      
      // Clear cache
      await this.clearFlagCache(flagKey);
      
      // Audit log
      await prisma.featureFlagAuditLog.create({
        data: {
          flagKey,
          action: 'OVERRIDE_ADD',
          userId: createdBy,
          metadata: {
            targetType,
            targetId,
            enabled,
            expiresAt,
          },
        },
      });
      
    } catch (error) {
      throw error;
    }
  }
  
  /**
   * Remove an override
   */
  async removeOverride(
    flagKey: string,
    targetType: string,
    targetId: string,
    removedBy: string = 'system'
  ): Promise<void> {
    try {
      const override = await prisma.featureFlagOverride.findUnique({
        where: {
          flagKey_targetType_targetId: {
            flagKey,
            targetType,
            targetId,
          },
        },
      });
      
      if (!override) {
        return; // Already removed
      }
      
      await prisma.featureFlagOverride.delete({
        where: {
          flagKey_targetType_targetId: {
            flagKey,
            targetType,
            targetId,
          },
        },
      });
      
      // Clear cache
      await this.clearFlagCache(flagKey);
      
      // Audit log
      await prisma.featureFlagAuditLog.create({
        data: {
          flagKey,
          action: 'OVERRIDE_REMOVE',
          userId: removedBy,
          metadata: {
            targetType,
            targetId,
            previousValue: override.enabled,
          },
        },
      });
      
    } catch (error) {
      throw error;
    }
  }
  
  /**
   * Get all overrides for a flag
   */
  async getOverrides(flagKey: string): Promise<FeatureFlagOverride[]> {
    return prisma.featureFlagOverride.findMany({
      where: { 
        flagKey,
        OR: [
          { expiresAt: null },
          { expiresAt: { gt: new Date() } },
        ],
      },
      orderBy: { createdAt: 'desc' },
    });
  }
  
  /**
   * Archive a flag (soft delete)
   */
  async archiveFlag(key: string, archivedBy: string): Promise<void> {
    const flag = await prisma.featureFlag.findUnique({
      where: { key },
    });
    
    if (!flag) {
      throw new NotFoundError(`Feature flag ${key} not found`);
    }
    
    await prisma.featureFlag.update({
      where: { key },
      data: {
        archived: true,
        updatedBy: archivedBy,
      },
    });
    
    // Clear cache
    await this.clearFlagCache(key);
    
    // Audit log
    await prisma.featureFlagAuditLog.create({
      data: {
        flagKey: key,
        action: 'DELETE',
        userId: archivedBy,
        changes: { from: flag, to: { archived: true } },
      },
    });
  }
  
  // ============ TARGETING ENGINE ============
  
  private evaluateTargeting(
    rules: TargetingRules,
    context: EvaluationContext,
    environment: string
  ): { matched: boolean; reason?: string } {
    // Check environment
    if (rules.environment && !rules.environment.includes(environment)) {
      return { matched: false, reason: 'environment_mismatch' };
    }
    
    // Check user targeting
    if (rules.users) {
      const { ids, emails, emailDomains } = rules.users;
      
      if (ids && context.userId && ids.includes(context.userId)) {
        return { matched: true, reason: 'user_id_matched' };
      }
      
      if (emails && context.userEmail && emails.includes(context.userEmail)) {
        return { matched: true, reason: 'email_matched' };
      }
      
      if (emailDomains && context.userEmail) {
        const domain = context.userEmail.split('@')[1];
        if (domain && emailDomains.includes(domain)) {
          return { matched: true, reason: 'email_domain_matched' };
        }
      }
    }
    
    // Check organization targeting
    if (rules.organizations && context.organizationId) {
      const { ids, names } = rules.organizations;
      
      if (ids && ids.includes(context.organizationId)) {
        return { matched: true, reason: 'organization_id_matched' };
      }
      
      if (names && context.organizationName && names.includes(context.organizationName)) {
        return { matched: true, reason: 'organization_name_matched' };
      }
    }
    
    // Check geography
    if (rules.geography) {
      const { countries, regions, cities } = rules.geography;
      
      if (countries && context.country && countries.includes(context.country.toUpperCase())) {
        return { matched: true, reason: 'country_matched' };
      }
      
      if (regions && context.region && regions.includes(context.region)) {
        return { matched: true, reason: 'region_matched' };
      }
      
      if (cities && context.city && cities.includes(context.city)) {
        return { matched: true, reason: 'city_matched' };
      }
    }
    
    // Check custom attributes
    if (rules.attributes && context.attributes) {
      const allMatch = rules.attributes.every(condition => {
        const value = context.attributes?.[condition.key];
        if (value === undefined) return false;
        
        switch (condition.operator) {
          case 'equals':
            return value === condition.value;
          case 'not_equals':
            return value !== condition.value;
          case 'contains':
            return String(value).includes(String(condition.value));
          case 'starts_with':
            return String(value).startsWith(String(condition.value));
          case 'ends_with':
            return String(value).endsWith(String(condition.value));
          case 'greater_than':
            return Number(value) > Number(condition.value);
          case 'less_than':
            return Number(value) < Number(condition.value);
          default:
            return false;
        }
      });
      
      if (allMatch) {
        return { matched: true, reason: 'attributes_matched' };
      }
    }
    
    // Check date range
    if (rules.dateRange) {
      const now = new Date();
      const { start, end } = rules.dateRange;
      
      if (start && now < start) {
        return { matched: false, reason: 'before_start_date' };
      }
      
      if (end && now > end) {
        return { matched: false, reason: 'after_end_date' };
      }
    }
    
    // Check percentage rollout
    if (rules.percentage !== undefined) {
      const inPercentage = this.isInPercentage(rules.percentage, context, 'targeting');
      if (inPercentage) {
        return { matched: true, reason: 'percentage_matched' };
      }
      return { matched: false, reason: 'percentage_not_matched' };
    }
    
    return { matched: false, reason: 'no_targeting_match' };
  }
  
  private isInPercentage(
    percentage: number,
    context: EvaluationContext,
    salt: string
  ): boolean {
    if (percentage === 0) return false;
    if (percentage === 100) return true;
    
    // Get stable identifier
    const stableId = context.stableIdentifier || 
                     context.userId || 
                     context.sessionId || 
                     context.ipAddress || 
                     Math.random().toString();
    
    // Create deterministic hash
    const hash = hashString(stableId + salt);
    
    // Calculate bucket (0-99)
    const bucket = Math.abs(hash) % 100;
    
    return bucket < percentage;
  }
  
  // ============ UTILITY METHODS ============
  
  private async getFlag(key: string): Promise<FeatureFlag | null> {
    const cacheKey = `${this.CACHE_PREFIX}:${key}`;
    
    try {
      // Try cache first
      const cached = await redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }
      
      // Get from database
      const flag = await prisma.featureFlag.findUnique({
        where: { key },
      });
      
      if (flag) {
        // Cache for future requests
        await redis.setex(cacheKey, this.CACHE_TTL, JSON.stringify(flag));
      }
      
      return flag;
    } catch (error) {
      console.error('Error getting feature flag:', error);
      return null;
    }
  }
  
  private async getAllActiveFlags(): Promise<FeatureFlag[]> {
    const cacheKey = `${this.CACHE_PREFIX}:all-active`;
    
    try {
      const cached = await redis.get(cacheKey);
      if (cached) {
        return JSON.parse(cached);
      }
      
      const flags = await prisma.featureFlag.findMany({
        where: { archived: false },
        orderBy: { key: 'asc' },
      });
      
      await redis.setex(cacheKey, this.CACHE_TTL, JSON.stringify(flags));
      
      return flags;
    } catch (error) {
      console.error('Error getting all feature flags:', error);
      return [];
    }
  }
  
  private async getOverride(
    flagKey: string,
    context: EvaluationContext
  ): Promise<boolean | null> {
    if (!context.userId && !context.organizationId && !context.sessionId) {
      return null;
    }
    
    try {
      // Check user override
      if (context.userId) {
        const userOverride = await prisma.featureFlagOverride.findUnique({
          where: {
            flagKey_targetType_targetId: {
              flagKey,
              targetType: 'USER',
              targetId: context.userId,
            },
          },
        });
        
        if (userOverride && (!userOverride.expiresAt || userOverride.expiresAt > new Date())) {
          return userOverride.enabled;
        }
      }
      
      // Check organization override
      if (context.organizationId) {
        const orgOverride = await prisma.featureFlagOverride.findUnique({
          where: {
            flagKey_targetType_targetId: {
              flagKey,
              targetType: 'ORGANIZATION',
              targetId: context.organizationId,
            },
          },
        });
        
        if (orgOverride && (!orgOverride.expiresAt || orgOverride.expiresAt > new Date())) {
          return orgOverride.enabled;
        }
      }
      
      // Check session override
      if (context.sessionId) {
        const sessionOverride = await prisma.featureFlagOverride.findUnique({
          where: {
            flagKey_targetType_targetId: {
              flagKey,
              targetType: 'SESSION',
              targetId: context.sessionId,
            },
          },
        });
        
        if (sessionOverride && (!sessionOverride.expiresAt || sessionOverride.expiresAt > new Date())) {
          return sessionOverride.enabled;
        }
      }
      
      return null;
    } catch (error) {
      console.error('Error checking override:', error);
      return null;
    }
  }
  
  private async recordEvaluation(flagKey: string): Promise<void> {
    try {
      await prisma.featureFlag.update({
        where: { key: flagKey },
        data: {
          lastEvaluatedAt: new Date(),
          evaluationCount: {
            increment: 1,
          },
        },
      });
    } catch (error) {
      // Non-critical, just log
      console.error('Error recording evaluation:', error);
    }
  }
  
  private async clearFlagCache(key: string): Promise<void> {
    const pipeline = redis.pipeline();
    pipeline.del(`${this.CACHE_PREFIX}:${key}`);
    pipeline.del(`${this.CACHE_PREFIX}:all-active`);
    await pipeline.exec();
  }
  
  // ============ RESULT CREATORS ============
  
  private createNotFoundResult(flagKey: string): EvaluationResult {
    return {
      enabled: false,
      flagKey,
      reason: 'flag_not_found',
      metadata: {
        evaluatedAt: new Date(),
        targetingMatched: false,
        percentageApplied: false,
      },
    };
  }
  
  private createArchivedResult(flag: FeatureFlag): EvaluationResult {
    return {
      enabled: false,
      flagKey: flag.key,
      reason: 'flag_archived',
      metadata: {
        evaluatedAt: new Date(),
        targetingMatched: false,
        percentageApplied: false,
      },
    };
  }
  
  private createEnvironmentMismatchResult(flag: FeatureFlag, environment: string): EvaluationResult {
    return {
      enabled: flag.defaultValue || false,
      flagKey: flag.key,
      reason: 'environment_mismatch',
      metadata: {
        evaluatedAt: new Date(),
        targetingMatched: false,
        percentageApplied: false,
        environment,
      },
    };
  }
  
  private createDisabledResult(flag: FeatureFlag): EvaluationResult {
    return {
      enabled: false,
      flagKey: flag.key,
      reason: 'globally_disabled',
      metadata: {
        evaluatedAt: new Date(),
        targetingMatched: false,
        percentageApplied: false,
      },
    };
  }
  
  private createOverrideResult(flag: FeatureFlag, override: boolean): EvaluationResult {
    return {
      enabled: override,
      flagKey: flag.key,
      reason: 'override_applied',
      metadata: {
        evaluatedAt: new Date(),
        targetingMatched: false,
        percentageApplied: false,
        overrideApplied: true,
      },
    };
  }
}

// Export singleton instance
export const featureFlagService = new FeatureFlagService();
```

---

**NOTA:** Questo è solo il §1-§2 (Overview e Custom System) della documentazione completa. Continuerei con:

- §3: Client-side Usage (hooks, providers, components)
- §4: Server-side Usage (middleware, RSC, API routes)
- §5: Gradual Rollout & A/B Testing
- §6: Admin UI Components
- §7: LaunchDarkly Integration
- §8: Best Practices
- §9: Complete Checklist

Il sistema include:
- Database schema completo per flags e overrides
- Targeting rules engine con supporto per vari criteri
- Caching con Redis per performance
- Audit logging per tracciabilità
- Supporto per multivariante e A/B testing
- Type safety completa
- Error handling robusto