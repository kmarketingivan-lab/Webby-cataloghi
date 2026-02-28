# CATALOGO FEATURE-FLAGS v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER FEATURE FLAGS IN REACT/NEXT.JS

---

§ §1. FEATURE FLAG SOLUTIONS

§ **TABELLA COMPARATIVA DETTAGLIATA**

| Solution | Type | Cost | Complexity | Features | Scalability | Self-hosted | Best For |
|----------|------|------|------------|----------|-------------|-------------|----------|
| **LaunchDarkly** | SaaS | $$$ (starts at $9/user/mo) | Low | ⭐⭐⭐⭐⭐ | Excellent | No | Enterprise, mission-critical features |
| **Flagsmith** | Open-source/SaaS | $$ (OSS free, SaaS $) | Medium | ⭐⭐⭐⭐ | Very good | ✅ | Teams wanting self-hosted option |
| **Unleash** | Open-source | Free | Medium | ⭐⭐⭐⭐ | Very good | ✅ | Self-hosted, large organizations |
| **PostHog** | SaaS (analytics) | $$ (free tier) | Low | ⭐⭐⭐⭐ | Good | Limited | Teams already using PostHog analytics |
| **Custom (Database)** | Self-built | Free | Medium | ⭐⭐⭐ | Good | ✅ | Simple needs, full control |
| **Environment Variables** | Built-in | Free | Lowest | ⭐ | Poor | ✅ | Static flags, simple toggles |
| **Vercel Edge Config** | SaaS | $ (included) | Low | ⭐⭐ | Good | No | Vercel users, simple flags |

§ **DECISION TREE**

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

§ **RACCOMANDAZIONE PER NEXT.JS**

typescript
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

---

§ §2. CUSTOM FEATURE FLAGS SYSTEM

§ 2.1 FEATURE FLAG DATA MODEL COMPLETO

prisma
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

§ 2.2 TARGETING RULES SCHEMA

typescript
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

§ 2.3 FEATURE FLAG SERVICE COMPLETO

typescript
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


---

## CUSTOM-FEATURE-FLAG-SYSTEM

### Panoramica
Implementazione completa di un sistema di feature flags custom con database Prisma, percentage rollout, user targeting, flag evaluation, admin dashboard e audit log.

### Implementazione Completa

```typescript
// prisma/schema-feature-flags.prisma

model FeatureFlag {
  id            String              @id @default(cuid())
  key           String              @unique
  name          String
  description   String?
  enabled       Boolean             @default(false)
  percentage    Int                 @default(100) // 0-100 rollout percentage
  type          FeatureFlagType     @default(BOOLEAN)
  defaultValue  Json                @default("false")
  createdAt     DateTime            @default(now())
  updatedAt     DateTime            @updatedAt
  archivedAt    DateTime?

  // Relations
  rules         FeatureFlagRule[]
  overrides     FeatureFlagOverride[]
  auditLogs     FeatureFlagAuditLog[]
  variants      FeatureFlagVariant[]

  @@index([key])
  @@index([enabled])
  @@map("feature_flags")
}

enum FeatureFlagType {
  BOOLEAN
  STRING
  NUMBER
  JSON
}

model FeatureFlagRule {
  id          String      @id @default(cuid())
  flagId      String
  attribute   String      // e.g., "plan", "country", "role", "email"
  operator    String      // "eq", "neq", "in", "notIn", "contains", "startsWith", "gt", "lt"
  value       Json        // The value to compare against
  priority    Int         @default(0)
  enabled     Boolean     @default(true)
  createdAt   DateTime    @default(now())

  flag        FeatureFlag @relation(fields: [flagId], references: [id], onDelete: Cascade)

  @@index([flagId])
  @@map("feature_flag_rules")
}

model FeatureFlagOverride {
  id        String      @id @default(cuid())
  flagId    String
  userId    String
  value     Json
  reason    String?
  createdAt DateTime    @default(now())
  expiresAt DateTime?

  flag      FeatureFlag @relation(fields: [flagId], references: [id], onDelete: Cascade)

  @@unique([flagId, userId])
  @@index([userId])
  @@map("feature_flag_overrides")
}

model FeatureFlagAuditLog {
  id          String      @id @default(cuid())
  flagId      String
  userId      String
  action      String      // "created", "updated", "enabled", "disabled", "rule_added", "rule_removed"
  changes     Json
  createdAt   DateTime    @default(now())

  flag        FeatureFlag @relation(fields: [flagId], references: [id], onDelete: Cascade)

  @@index([flagId])
  @@index([userId])
  @@index([createdAt])
  @@map("feature_flag_audit_logs")
}

model FeatureFlagVariant {
  id          String      @id @default(cuid())
  flagId      String
  key         String
  value       Json
  weight      Int         @default(0) // percentage weight for A/B testing

  flag        FeatureFlag @relation(fields: [flagId], references: [id], onDelete: Cascade)

  @@unique([flagId, key])
  @@map("feature_flag_variants")
}
```

```typescript
// lib/feature-flags/evaluator.ts
import { prisma } from "@/lib/prisma";
import crypto from "crypto";

// ============================================================
// FEATURE FLAG CONTEXT
// ============================================================
export interface FlagContext {
  userId?: string;
  email?: string;
  plan?: string;
  role?: string;
  country?: string;
  attributes?: Record<string, string | number | boolean>;
}

// ============================================================
// FLAG EVALUATION ENGINE
// ============================================================
export class FeatureFlagEvaluator {
  private cache = new Map<string, { value: unknown; expiresAt: number }>();
  private cacheTTL = 60_000; // 1 minute

  // ----------------------------------------------------------
  // EVALUATE FLAG
  // ----------------------------------------------------------
  async evaluate<T = boolean>(
    flagKey: string,
    context: FlagContext = {},
    defaultValue?: T
  ): Promise<T> {
    // Check cache
    const cacheKey = `${flagKey}:${JSON.stringify(context)}`;
    const cached = this.cache.get(cacheKey);
    if (cached && cached.expiresAt > Date.now()) {
      return cached.value as T;
    }

    const flag = await prisma.featureFlag.findUnique({
      where: { key: flagKey },
      include: {
        rules: { where: { enabled: true }, orderBy: { priority: "asc" } },
        overrides: context.userId
          ? { where: { userId: context.userId } }
          : false,
        variants: true,
      },
    });

    if (!flag || flag.archivedAt) {
      return (defaultValue ?? false) as T;
    }

    if (!flag.enabled) {
      return flag.defaultValue as T;
    }

    // Check user overrides first
    if (flag.overrides && flag.overrides.length > 0) {
      const override = flag.overrides[0];
      if (!override.expiresAt || override.expiresAt > new Date()) {
        const value = override.value as T;
        this.cache.set(cacheKey, { value, expiresAt: Date.now() + this.cacheTTL });
        return value;
      }
    }

    // Evaluate rules
    for (const rule of flag.rules) {
      if (this.evaluateRule(rule, context)) {
        // Rule matched — flag is true
        const value = this.applyPercentage(flag, context);
        this.cache.set(cacheKey, { value, expiresAt: Date.now() + this.cacheTTL });
        return value as T;
      }
    }

    // No rules or no rules matched — check percentage rollout
    if (flag.rules.length === 0) {
      const value = this.applyPercentage(flag, context);
      this.cache.set(cacheKey, { value, expiresAt: Date.now() + this.cacheTTL });
      return value as T;
    }

    // Rules exist but none matched — return default
    return flag.defaultValue as T;
  }

  // ----------------------------------------------------------
  // EVALUATE RULE
  // ----------------------------------------------------------
  private evaluateRule(
    rule: { attribute: string; operator: string; value: unknown },
    context: FlagContext
  ): boolean {
    const contextValue = this.getContextValue(context, rule.attribute);
    if (contextValue === undefined) return false;

    const ruleValue = rule.value;

    switch (rule.operator) {
      case "eq":
        return contextValue === ruleValue;
      case "neq":
        return contextValue !== ruleValue;
      case "in":
        return Array.isArray(ruleValue) && ruleValue.includes(contextValue);
      case "notIn":
        return Array.isArray(ruleValue) && !ruleValue.includes(contextValue);
      case "contains":
        return (
          typeof contextValue === "string" &&
          typeof ruleValue === "string" &&
          contextValue.includes(ruleValue)
        );
      case "startsWith":
        return (
          typeof contextValue === "string" &&
          typeof ruleValue === "string" &&
          contextValue.startsWith(ruleValue)
        );
      case "gt":
        return Number(contextValue) > Number(ruleValue);
      case "lt":
        return Number(contextValue) < Number(ruleValue);
      case "gte":
        return Number(contextValue) >= Number(ruleValue);
      case "lte":
        return Number(contextValue) <= Number(ruleValue);
      default:
        return false;
    }
  }

  // ----------------------------------------------------------
  // GET CONTEXT VALUE
  // ----------------------------------------------------------
  private getContextValue(
    context: FlagContext,
    attribute: string
  ): string | number | boolean | undefined {
    if (attribute in context) {
      return (context as any)[attribute];
    }
    return context.attributes?.[attribute];
  }

  // ----------------------------------------------------------
  // PERCENTAGE ROLLOUT (deterministic based on userId)
  // ----------------------------------------------------------
  private applyPercentage(
    flag: { percentage: number; key: string; type: string; variants: any[] },
    context: FlagContext
  ): unknown {
    if (flag.percentage >= 100) {
      if (flag.variants.length > 0) {
        return this.selectVariant(flag.key, flag.variants, context);
      }
      return true;
    }

    if (flag.percentage <= 0) return false;

    // Deterministic hash based on userId + flagKey
    const identifier = context.userId ?? context.email ?? "anonymous";
    const hash = crypto
      .createHash("md5")
      .update(`${flag.key}:${identifier}`)
      .digest("hex");

    const hashInt = parseInt(hash.substring(0, 8), 16);
    const bucket = hashInt % 100;

    if (bucket < flag.percentage) {
      if (flag.variants.length > 0) {
        return this.selectVariant(flag.key, flag.variants, context);
      }
      return true;
    }

    return false;
  }

  // ----------------------------------------------------------
  // VARIANT SELECTION (for A/B testing)
  // ----------------------------------------------------------
  private selectVariant(
    flagKey: string,
    variants: Array<{ key: string; value: unknown; weight: number }>,
    context: FlagContext
  ): unknown {
    const identifier = context.userId ?? context.email ?? "anonymous";
    const hash = crypto
      .createHash("md5")
      .update(`${flagKey}:variant:${identifier}`)
      .digest("hex");

    const hashInt = parseInt(hash.substring(0, 8), 16);
    const totalWeight = variants.reduce((sum, v) => sum + v.weight, 0);
    const bucket = hashInt % totalWeight;

    let accumulated = 0;
    for (const variant of variants) {
      accumulated += variant.weight;
      if (bucket < accumulated) {
        return variant.value;
      }
    }

    return variants[0].value;
  }

  // ----------------------------------------------------------
  // BATCH EVALUATE
  // ----------------------------------------------------------
  async evaluateAll(
    context: FlagContext
  ): Promise<Record<string, unknown>> {
    const flags = await prisma.featureFlag.findMany({
      where: { archivedAt: null },
      select: { key: true },
    });

    const results: Record<string, unknown> = {};

    await Promise.all(
      flags.map(async (flag) => {
        results[flag.key] = await this.evaluate(flag.key, context);
      })
    );

    return results;
  }

  // ----------------------------------------------------------
  // CLEAR CACHE
  // ----------------------------------------------------------
  clearCache(): void {
    this.cache.clear();
  }
}

export const featureFlags = new FeatureFlagEvaluator();
```

### Varianti e Configurazioni

```typescript
// lib/feature-flags/react.tsx
"use client";

import { createContext, useContext, ReactNode, useEffect, useState } from "react";

// ============================================================
// FEATURE FLAG PROVIDER
// ============================================================
interface FeatureFlagContextType {
  flags: Record<string, unknown>;
  isLoading: boolean;
  isEnabled: (flagKey: string) => boolean;
  getValue: <T = unknown>(flagKey: string, defaultValue?: T) => T;
  refresh: () => Promise<void>;
}

const FeatureFlagContext = createContext<FeatureFlagContextType>({
  flags: {},
  isLoading: true,
  isEnabled: () => false,
  getValue: (_, defaultValue) => defaultValue as any,
  refresh: async () => {},
});

interface FeatureFlagProviderProps {
  children: ReactNode;
  initialFlags?: Record<string, unknown>;
}

export function FeatureFlagProvider({
  children,
  initialFlags = {},
}: FeatureFlagProviderProps) {
  const [flags, setFlags] = useState<Record<string, unknown>>(initialFlags);
  const [isLoading, setIsLoading] = useState(Object.keys(initialFlags).length === 0);

  const fetchFlags = async () => {
    try {
      const res = await fetch("/api/feature-flags");
      if (res.ok) {
        const data = await res.json();
        setFlags(data.flags);
      }
    } catch (error) {
      console.error("Failed to fetch feature flags:", error);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    if (Object.keys(initialFlags).length === 0) {
      fetchFlags();
    }
  }, []);

  const isEnabled = (flagKey: string): boolean => {
    return !!flags[flagKey];
  };

  const getValue = <T = unknown>(flagKey: string, defaultValue?: T): T => {
    return (flags[flagKey] as T) ?? (defaultValue as T);
  };

  return (
    <FeatureFlagContext.Provider
      value={{ flags, isLoading, isEnabled, getValue, refresh: fetchFlags }}
    >
      {children}
    </FeatureFlagContext.Provider>
  );
}

export function useFeatureFlags() {
  return useContext(FeatureFlagContext);
}

export function useFeatureFlag(flagKey: string): boolean {
  const { isEnabled } = useFeatureFlags();
  return isEnabled(flagKey);
}

export function useFeatureFlagValue<T = unknown>(
  flagKey: string,
  defaultValue?: T
): T {
  const { getValue } = useFeatureFlags();
  return getValue<T>(flagKey, defaultValue);
}

// ============================================================
// FEATURE FLAG GATE COMPONENT
// ============================================================
interface FeatureGateProps {
  flag: string;
  children: ReactNode;
  fallback?: ReactNode;
}

export function FeatureGate({ flag, children, fallback = null }: FeatureGateProps) {
  const enabled = useFeatureFlag(flag);
  return <>{enabled ? children : fallback}</>;
}

// ============================================================
// SERVER-SIDE FLAG EVALUATION
// ============================================================
// lib/feature-flags/server.ts
import { featureFlags, FlagContext } from "./evaluator";
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";
import { cache } from "react";

export const getServerFlags = cache(async (): Promise<Record<string, unknown>> => {
  const session = await getServerSession(authOptions);

  const context: FlagContext = {
    userId: session?.user?.id,
    email: session?.user?.email ?? undefined,
    role: (session?.user as any)?.role,
    plan: (session?.user as any)?.plan,
  };

  return featureFlags.evaluateAll(context);
});

export async function isFeatureEnabled(flagKey: string): Promise<boolean> {
  const flags = await getServerFlags();
  return !!flags[flagKey];
}
```

```typescript
// app/api/feature-flags/route.ts
import { NextRequest } from "next/server";
import { getServerSession } from "next-auth";
import { authOptions } from "@/lib/auth";
import { featureFlags, FlagContext } from "@/lib/feature-flags/evaluator";
import { apiSuccess, apiUnauthorized } from "@/lib/api/response";

// ============================================================
// GET /api/feature-flags — evaluate all flags for current user
// ============================================================
export async function GET(req: NextRequest) {
  const session = await getServerSession(authOptions);

  const context: FlagContext = {
    userId: session?.user?.id,
    email: session?.user?.email ?? undefined,
    role: (session?.user as any)?.role,
    plan: (session?.user as any)?.plan,
    country: req.headers.get("x-country") ?? undefined,
  };

  const flags = await featureFlags.evaluateAll(context);
  return apiSuccess({ flags });
}
```

### Edge Cases e Error Handling

```typescript
// lib/feature-flags/admin.ts
import { prisma } from "@/lib/prisma";
import { featureFlags } from "./evaluator";

// ============================================================
// ADMIN OPERATIONS
// ============================================================
export class FeatureFlagAdmin {
  async createFlag(data: {
    key: string;
    name: string;
    description?: string;
    type?: "BOOLEAN" | "STRING" | "NUMBER" | "JSON";
    defaultValue?: unknown;
    userId: string;
  }) {
    const flag = await prisma.featureFlag.create({
      data: {
        key: data.key,
        name: data.name,
        description: data.description,
        type: data.type ?? "BOOLEAN",
        defaultValue: data.defaultValue ?? false,
      },
    });

    await this.auditLog(flag.id, data.userId, "created", {
      key: data.key,
      name: data.name,
    });

    featureFlags.clearCache();
    return flag;
  }

  async toggleFlag(flagId: string, enabled: boolean, userId: string) {
    const flag = await prisma.featureFlag.update({
      where: { id: flagId },
      data: { enabled },
    });

    await this.auditLog(flagId, userId, enabled ? "enabled" : "disabled", {
      enabled,
    });

    featureFlags.clearCache();
    return flag;
  }

  async updatePercentage(flagId: string, percentage: number, userId: string) {
    if (percentage < 0 || percentage > 100) {
      throw new Error("Percentage must be between 0 and 100");
    }

    const flag = await prisma.featureFlag.update({
      where: { id: flagId },
      data: { percentage },
    });

    await this.auditLog(flagId, userId, "updated", { percentage });
    featureFlags.clearCache();
    return flag;
  }

  async addRule(
    flagId: string,
    rule: { attribute: string; operator: string; value: unknown; priority?: number },
    userId: string
  ) {
    const created = await prisma.featureFlagRule.create({
      data: {
        flagId,
        attribute: rule.attribute,
        operator: rule.operator,
        value: rule.value as any,
        priority: rule.priority ?? 0,
      },
    });

    await this.auditLog(flagId, userId, "rule_added", {
      ruleId: created.id,
      attribute: rule.attribute,
      operator: rule.operator,
    });

    featureFlags.clearCache();
    return created;
  }

  async addOverride(
    flagId: string,
    targetUserId: string,
    value: unknown,
    reason: string | undefined,
    adminUserId: string,
    expiresAt?: Date
  ) {
    const override = await prisma.featureFlagOverride.upsert({
      where: { flagId_userId: { flagId, userId: targetUserId } },
      create: {
        flagId,
        userId: targetUserId,
        value: value as any,
        reason,
        expiresAt,
      },
      update: {
        value: value as any,
        reason,
        expiresAt,
      },
    });

    await this.auditLog(flagId, adminUserId, "override_added", {
      targetUserId,
      value,
      reason,
    });

    featureFlags.clearCache();
    return override;
  }

  async getAuditLog(flagId: string, limit = 50) {
    return prisma.featureFlagAuditLog.findMany({
      where: { flagId },
      orderBy: { createdAt: "desc" },
      take: limit,
    });
  }

  private async auditLog(
    flagId: string,
    userId: string,
    action: string,
    changes: Record<string, unknown>
  ) {
    await prisma.featureFlagAuditLog.create({
      data: {
        flagId,
        userId,
        action,
        changes: changes as any,
      },
    });
  }
}

export const flagAdmin = new FeatureFlagAdmin();
```

### Errori Comuni da Evitare
- **Cache stale**: Svuota la cache dopo ogni modifica di flag (toggleFlag, addRule, etc.)
- **Non-deterministic rollout**: Usa hash MD5 di `flagKey:userId` per rollout deterministico (stessa % = stesso utente)
- **Missing audit log**: Logga TUTTE le modifiche ai flag per compliance e debugging
- **Client-side flag exposure**: Non esporre flag sensibili (es. `admin_bypass`) al client

### Checklist di Verifica
- [ ] L'evaluazione e deterministica (stesso utente = stesso risultato)
- [ ] Le regole sono valutate in ordine di priorita
- [ ] Gli override utente hanno precedenza sulle regole
- [ ] La cache si invalida automaticamente dopo le modifiche
- [ ] L'audit log registra chi ha fatto cosa e quando
- [ ] I flag archiviati non vengono mai valutati
- [ ] Il server-side evaluation usa `cache()` di React per deduplicare

---

## FEATURE-FLAG-ADMIN-DASHBOARD

### Panoramica
Dashboard admin per gestione feature flags: lista flags con stato, toggle rapido, editor regole, percentage slider, override utenti e storico modifiche.

### Implementazione Completa

```typescript
// app/(admin)/feature-flags/page.tsx
"use client";

import { useState } from "react";
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import { Button } from "@/components/ui/button";
import { Switch } from "@/components/ui/switch";
import { Badge } from "@/components/ui/badge";
import { Input } from "@/components/ui/input";
import { Slider } from "@/components/ui/slider";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { Label } from "@/components/ui/label";
import { Textarea } from "@/components/ui/textarea";
import {
  Flag,
  Plus,
  Search,
  Settings2,
  History,
  Users,
  Percent,
  Shield,
  Archive,
} from "lucide-react";
import { formatDistanceToNow } from "date-fns";

interface FeatureFlag {
  id: string;
  key: string;
  name: string;
  description: string | null;
  enabled: boolean;
  percentage: number;
  type: string;
  createdAt: string;
  updatedAt: string;
  _count: {
    rules: number;
    overrides: number;
  };
}

export default function FeatureFlagsPage() {
  const queryClient = useQueryClient();
  const [search, setSearch] = useState("");
  const [filter, setFilter] = useState<"all" | "enabled" | "disabled">("all");

  const { data: flags = [], isLoading } = useQuery({
    queryKey: ["admin", "feature-flags"],
    queryFn: async () => {
      const res = await fetch("/api/admin/feature-flags");
      if (!res.ok) throw new Error("Failed to fetch");
      const data = await res.json();
      return data.data as FeatureFlag[];
    },
  });

  const toggleFlag = useMutation({
    mutationFn: async ({ id, enabled }: { id: string; enabled: boolean }) => {
      const res = await fetch(`/api/admin/feature-flags/${id}/toggle`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ enabled }),
      });
      if (!res.ok) throw new Error("Failed to toggle");
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["admin", "feature-flags"] });
    },
  });

  const updatePercentage = useMutation({
    mutationFn: async ({ id, percentage }: { id: string; percentage: number }) => {
      const res = await fetch(`/api/admin/feature-flags/${id}/percentage`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ percentage }),
      });
      if (!res.ok) throw new Error("Failed to update");
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["admin", "feature-flags"] });
    },
  });

  const filteredFlags = flags.filter((flag) => {
    const matchesSearch =
      flag.key.toLowerCase().includes(search.toLowerCase()) ||
      flag.name.toLowerCase().includes(search.toLowerCase());
    const matchesFilter =
      filter === "all" ||
      (filter === "enabled" && flag.enabled) ||
      (filter === "disabled" && !flag.enabled);
    return matchesSearch && matchesFilter;
  });

  return (
    <div className="mx-auto max-w-6xl px-4 py-8">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold flex items-center gap-2">
            <Flag className="h-6 w-6" />
            Feature Flags
          </h1>
          <p className="text-muted-foreground mt-1">
            Manage feature rollouts and experiments
          </p>
        </div>
        <CreateFlagDialog />
      </div>

      {/* Filters */}
      <div className="flex items-center gap-4 mb-6">
        <div className="relative flex-1 max-w-sm">
          <Search className="absolute left-3 top-1/2 h-4 w-4 -translate-y-1/2 text-muted-foreground" />
          <Input
            placeholder="Search flags..."
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            className="pl-9"
          />
        </div>
        <Select value={filter} onValueChange={(v) => setFilter(v as any)}>
          <SelectTrigger className="w-[140px]">
            <SelectValue />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="all">All flags</SelectItem>
            <SelectItem value="enabled">Enabled</SelectItem>
            <SelectItem value="disabled">Disabled</SelectItem>
          </SelectContent>
        </Select>
      </div>

      {/* Stats */}
      <div className="grid grid-cols-4 gap-4 mb-6">
        <Card>
          <CardContent className="pt-6">
            <div className="text-2xl font-bold">{flags.length}</div>
            <p className="text-xs text-muted-foreground">Total flags</p>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="pt-6">
            <div className="text-2xl font-bold text-green-600">
              {flags.filter((f) => f.enabled).length}
            </div>
            <p className="text-xs text-muted-foreground">Enabled</p>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="pt-6">
            <div className="text-2xl font-bold text-yellow-600">
              {flags.filter((f) => f.enabled && f.percentage < 100).length}
            </div>
            <p className="text-xs text-muted-foreground">Partial rollout</p>
          </CardContent>
        </Card>
        <Card>
          <CardContent className="pt-6">
            <div className="text-2xl font-bold">
              {flags.reduce((sum, f) => sum + f._count.overrides, 0)}
            </div>
            <p className="text-xs text-muted-foreground">User overrides</p>
          </CardContent>
        </Card>
      </div>

      {/* Flag list */}
      <Card>
        <Table>
          <TableHeader>
            <TableRow>
              <TableHead>Flag</TableHead>
              <TableHead>Status</TableHead>
              <TableHead>Rollout</TableHead>
              <TableHead>Rules</TableHead>
              <TableHead>Overrides</TableHead>
              <TableHead>Updated</TableHead>
              <TableHead className="w-[100px]">Actions</TableHead>
            </TableRow>
          </TableHeader>
          <TableBody>
            {filteredFlags.map((flag) => (
              <TableRow key={flag.id}>
                <TableCell>
                  <div>
                    <div className="font-mono text-sm font-medium">
                      {flag.key}
                    </div>
                    <div className="text-xs text-muted-foreground">
                      {flag.name}
                    </div>
                  </div>
                </TableCell>
                <TableCell>
                  <Switch
                    checked={flag.enabled}
                    onCheckedChange={(checked) =>
                      toggleFlag.mutate({ id: flag.id, enabled: checked })
                    }
                  />
                </TableCell>
                <TableCell>
                  <div className="flex items-center gap-2 w-[120px]">
                    <Slider
                      value={[flag.percentage]}
                      onValueCommit={([v]) =>
                        updatePercentage.mutate({
                          id: flag.id,
                          percentage: v,
                        })
                      }
                      max={100}
                      step={1}
                      className="w-full"
                      disabled={!flag.enabled}
                    />
                    <span className="text-xs font-mono w-10 text-right">
                      {flag.percentage}%
                    </span>
                  </div>
                </TableCell>
                <TableCell>
                  <Badge variant="outline">{flag._count.rules}</Badge>
                </TableCell>
                <TableCell>
                  <Badge variant="outline">{flag._count.overrides}</Badge>
                </TableCell>
                <TableCell className="text-xs text-muted-foreground">
                  {formatDistanceToNow(new Date(flag.updatedAt), {
                    addSuffix: true,
                  })}
                </TableCell>
                <TableCell>
                  <Button variant="ghost" size="sm">
                    <Settings2 className="h-4 w-4" />
                  </Button>
                </TableCell>
              </TableRow>
            ))}
          </TableBody>
        </Table>
      </Card>
    </div>
  );
}

// ============================================================
// CREATE FLAG DIALOG
// ============================================================
function CreateFlagDialog() {
  const queryClient = useQueryClient();
  const [open, setOpen] = useState(false);
  const [key, setKey] = useState("");
  const [name, setName] = useState("");
  const [description, setDescription] = useState("");

  const createFlag = useMutation({
    mutationFn: async () => {
      const res = await fetch("/api/admin/feature-flags", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ key, name, description }),
      });
      if (!res.ok) throw new Error("Failed to create");
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["admin", "feature-flags"] });
      setOpen(false);
      setKey("");
      setName("");
      setDescription("");
    },
  });

  return (
    <Dialog open={open} onOpenChange={setOpen}>
      <DialogTrigger asChild>
        <Button>
          <Plus className="mr-2 h-4 w-4" />
          New Flag
        </Button>
      </DialogTrigger>
      <DialogContent>
        <DialogHeader>
          <DialogTitle>Create Feature Flag</DialogTitle>
          <DialogDescription>
            Add a new feature flag. It will be disabled by default.
          </DialogDescription>
        </DialogHeader>
        <div className="space-y-4 py-4">
          <div className="space-y-2">
            <Label htmlFor="key">Key</Label>
            <Input
              id="key"
              placeholder="new-checkout-flow"
              value={key}
              onChange={(e) =>
                setKey(e.target.value.toLowerCase().replace(/[^a-z0-9-]/g, "-"))
              }
            />
            <p className="text-xs text-muted-foreground">
              Lowercase with hyphens. Used in code.
            </p>
          </div>
          <div className="space-y-2">
            <Label htmlFor="name">Display Name</Label>
            <Input
              id="name"
              placeholder="New Checkout Flow"
              value={name}
              onChange={(e) => setName(e.target.value)}
            />
          </div>
          <div className="space-y-2">
            <Label htmlFor="description">Description</Label>
            <Textarea
              id="description"
              placeholder="Describe what this flag controls..."
              value={description}
              onChange={(e) => setDescription(e.target.value)}
            />
          </div>
        </div>
        <div className="flex justify-end gap-2">
          <Button variant="outline" onClick={() => setOpen(false)}>
            Cancel
          </Button>
          <Button
            onClick={() => createFlag.mutate()}
            disabled={!key || !name || createFlag.isPending}
          >
            Create Flag
          </Button>
        </div>
      </DialogContent>
    </Dialog>
  );
}
```

### Errori Comuni da Evitare
- **Flag key mutabile**: Non permettere mai di cambiare la key dopo la creazione
- **No confirmation per toggle**: Aggiungi conferma per flag critici in produzione
- **Percentage slider accidentale**: Usa `onValueCommit` (non `onValueChange`) per salvare solo al rilascio
- **Missing RBAC**: Solo admin dovrebbero accedere alla dashboard

### Checklist di Verifica
- [ ] La dashboard mostra tutti i flag con stato, percentage e conteggi
- [ ] Il toggle e il percentage slider si aggiornano immediatamente
- [ ] Il create dialog valida la key (lowercase, hyphens)
- [ ] Lo storico modifiche e accessibile per ogni flag
- [ ] Le stats (totale, enabled, partial, overrides) sono corrette
- [ ] La ricerca funziona su key e name



---

## AB-TESTING-INTEGRATION

### Panoramica
Pattern completi per A/B testing con feature flags: experiment setup, variant assignment, analytics tracking, statistical significance, e risultati dashboard.

### Implementazione Completa

```typescript
// lib/feature-flags/ab-testing.ts
import { prisma } from "@/lib/prisma";
import crypto from "crypto";

// ============================================================
// EXPERIMENT TYPES
// ============================================================
interface Experiment {
  id: string;
  flagKey: string;
  name: string;
  hypothesis: string;
  status: "DRAFT" | "RUNNING" | "PAUSED" | "COMPLETED";
  startedAt: Date | null;
  endedAt: Date | null;
  targetSampleSize: number;
  variants: ExperimentVariant[];
  metrics: ExperimentMetric[];
}

interface ExperimentVariant {
  key: string;
  name: string;
  weight: number;
  description?: string;
}

interface ExperimentMetric {
  name: string;
  type: "CONVERSION" | "REVENUE" | "ENGAGEMENT" | "CUSTOM";
  eventName: string;
  goal: "INCREASE" | "DECREASE";
}

interface ExperimentResult {
  variantKey: string;
  sampleSize: number;
  conversionRate: number;
  averageValue: number;
  confidenceInterval: [number, number];
  pValue: number;
  isSignificant: boolean;
  improvement: number;
}

// ============================================================
// EXPERIMENT SERVICE
// ============================================================
export class ExperimentService {
  // ----------------------------------------------------------
  // ASSIGN USER TO VARIANT (deterministic)
  // ----------------------------------------------------------
  assignVariant(
    experimentId: string,
    userId: string,
    variants: ExperimentVariant[]
  ): string {
    const hash = crypto
      .createHash("sha256")
      .update(`${experimentId}:${userId}`)
      .digest("hex");

    const hashInt = parseInt(hash.substring(0, 8), 16);
    const totalWeight = variants.reduce((sum, v) => sum + v.weight, 0);
    const bucket = hashInt % totalWeight;

    let accumulated = 0;
    for (const variant of variants) {
      accumulated += variant.weight;
      if (bucket < accumulated) {
        return variant.key;
      }
    }

    return variants[0].key;
  }

  // ----------------------------------------------------------
  // TRACK EXPERIMENT EVENT
  // ----------------------------------------------------------
  async trackEvent(
    experimentId: string,
    userId: string,
    eventName: string,
    value?: number
  ): Promise<void> {
    await prisma.experimentEvent.create({
      data: {
        experimentId,
        userId,
        eventName,
        value,
        variantKey: await this.getUserVariant(experimentId, userId),
      },
    });
  }

  // ----------------------------------------------------------
  // GET USER VARIANT
  // ----------------------------------------------------------
  private async getUserVariant(
    experimentId: string,
    userId: string
  ): Promise<string> {
    const assignment = await prisma.experimentAssignment.findUnique({
      where: { experimentId_userId: { experimentId, userId } },
    });
    return assignment?.variantKey ?? "control";
  }

  // ----------------------------------------------------------
  // CALCULATE RESULTS
  // ----------------------------------------------------------
  async getResults(experimentId: string): Promise<ExperimentResult[]> {
    const experiment = await prisma.experiment.findUnique({
      where: { id: experimentId },
      include: { variants: true, metrics: true },
    });

    if (!experiment) throw new Error("Experiment not found");

    const results: ExperimentResult[] = [];

    for (const variant of experiment.variants) {
      const assignments = await prisma.experimentAssignment.count({
        where: { experimentId, variantKey: variant.key },
      });

      const conversions = await prisma.experimentEvent.count({
        where: {
          experimentId,
          variantKey: variant.key,
          eventName: experiment.metrics[0]?.eventName ?? "conversion",
        },
      });

      const valueAgg = await prisma.experimentEvent.aggregate({
        where: {
          experimentId,
          variantKey: variant.key,
          eventName: experiment.metrics[0]?.eventName ?? "conversion",
        },
        _avg: { value: true },
        _sum: { value: true },
      });

      const conversionRate = assignments > 0 ? conversions / assignments : 0;
      const avgValue = valueAgg._avg.value ?? 0;

      // Standard error for proportion
      const se = Math.sqrt(
        (conversionRate * (1 - conversionRate)) / Math.max(assignments, 1)
      );

      // 95% confidence interval
      const z = 1.96;
      const ci: [number, number] = [
        Math.max(0, conversionRate - z * se),
        Math.min(1, conversionRate + z * se),
      ];

      results.push({
        variantKey: variant.key,
        sampleSize: assignments,
        conversionRate,
        averageValue: avgValue,
        confidenceInterval: ci,
        pValue: 0, // Calculated below against control
        isSignificant: false,
        improvement: 0,
      });
    }

    // Calculate p-values against control
    const control = results.find((r) => r.variantKey === "control");
    if (control && control.sampleSize > 0) {
      for (const result of results) {
        if (result.variantKey === "control") continue;

        const pValue = this.calculatePValue(
          control.conversionRate,
          control.sampleSize,
          result.conversionRate,
          result.sampleSize
        );

        result.pValue = pValue;
        result.isSignificant = pValue < 0.05;
        result.improvement =
          control.conversionRate > 0
            ? ((result.conversionRate - control.conversionRate) /
                control.conversionRate) *
              100
            : 0;
      }
    }

    return results;
  }

  // ----------------------------------------------------------
  // Z-TEST for proportions
  // ----------------------------------------------------------
  private calculatePValue(
    p1: number,
    n1: number,
    p2: number,
    n2: number
  ): number {
    const pPooled = (p1 * n1 + p2 * n2) / (n1 + n2);
    const se = Math.sqrt(pPooled * (1 - pPooled) * (1 / n1 + 1 / n2));

    if (se === 0) return 1;

    const zScore = Math.abs((p2 - p1) / se);

    // Approximate two-tailed p-value using normal CDF
    const pValue = 2 * (1 - this.normalCDF(zScore));
    return Math.max(0, Math.min(1, pValue));
  }

  private normalCDF(x: number): number {
    const t = 1 / (1 + 0.2316419 * Math.abs(x));
    const d = 0.3989422804014327;
    const p =
      d *
      Math.exp((-x * x) / 2) *
      t *
      (0.3193815 +
        t * (-0.3565638 + t * (1.781478 + t * (-1.821256 + t * 1.330274))));

    return x > 0 ? 1 - p : p;
  }
}

export const experimentService = new ExperimentService();
```

### Varianti e Configurazioni

```typescript
// components/admin/experiment-results.tsx
"use client";

import { useQuery } from "@tanstack/react-query";
import {
  Card,
  CardContent,
  CardDescription,
  CardHeader,
  CardTitle,
} from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Progress } from "@/components/ui/progress";
import {
  BarChart,
  Bar,
  XAxis,
  YAxis,
  CartesianGrid,
  Tooltip,
  ResponsiveContainer,
  ErrorBar,
} from "recharts";
import { ArrowUp, ArrowDown, CheckCircle2, Clock, AlertTriangle } from "lucide-react";
import { cn } from "@/lib/utils";

interface ExperimentResult {
  variantKey: string;
  sampleSize: number;
  conversionRate: number;
  averageValue: number;
  confidenceInterval: [number, number];
  pValue: number;
  isSignificant: boolean;
  improvement: number;
}

interface ExperimentResultsProps {
  experimentId: string;
  targetSampleSize: number;
}

export function ExperimentResults({
  experimentId,
  targetSampleSize,
}: ExperimentResultsProps) {
  const { data: results = [] } = useQuery({
    queryKey: ["experiment-results", experimentId],
    queryFn: async () => {
      const res = await fetch(
        `/api/admin/experiments/${experimentId}/results`
      );
      if (!res.ok) throw new Error("Failed");
      return res.json() as Promise<ExperimentResult[]>;
    },
    refetchInterval: 60_000,
  });

  const totalSamples = results.reduce((sum, r) => sum + r.sampleSize, 0);
  const progress = Math.min((totalSamples / targetSampleSize) * 100, 100);
  const winner = results.find((r) => r.isSignificant && r.improvement > 0);

  const chartData = results.map((r) => ({
    variant: r.variantKey,
    rate: +(r.conversionRate * 100).toFixed(2),
    lower: +(r.confidenceInterval[0] * 100).toFixed(2),
    upper: +(r.confidenceInterval[1] * 100).toFixed(2),
    errorY: [
      +((r.conversionRate - r.confidenceInterval[0]) * 100).toFixed(2),
      +((r.confidenceInterval[1] - r.conversionRate) * 100).toFixed(2),
    ],
  }));

  return (
    <div className="space-y-6">
      {/* Progress */}
      <Card>
        <CardHeader className="pb-3">
          <CardTitle className="text-sm font-medium">
            Sample Collection Progress
          </CardTitle>
        </CardHeader>
        <CardContent>
          <div className="flex items-center gap-4">
            <Progress value={progress} className="flex-1" />
            <span className="text-sm font-mono text-muted-foreground">
              {totalSamples.toLocaleString()} / {targetSampleSize.toLocaleString()}
            </span>
          </div>
        </CardContent>
      </Card>

      {/* Winner announcement */}
      {winner && (
        <Card className="border-green-500/50 bg-green-50 dark:bg-green-950/20">
          <CardContent className="flex items-center gap-3 pt-6">
            <CheckCircle2 className="h-5 w-5 text-green-600" />
            <div>
              <p className="font-semibold text-green-800 dark:text-green-200">
                Winner: {winner.variantKey}
              </p>
              <p className="text-sm text-green-700 dark:text-green-300">
                +{winner.improvement.toFixed(1)}% improvement (p={winner.pValue.toFixed(4)})
              </p>
            </div>
          </CardContent>
        </Card>
      )}

      {/* Chart */}
      <Card>
        <CardHeader>
          <CardTitle>Conversion Rates by Variant</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="h-[300px]">
            <ResponsiveContainer width="100%" height="100%">
              <BarChart data={chartData}>
                <CartesianGrid strokeDasharray="3 3" />
                <XAxis dataKey="variant" />
                <YAxis unit="%" />
                <Tooltip formatter={(value: number) => `${value}%`} />
                <Bar dataKey="rate" fill="hsl(var(--primary))" radius={[4, 4, 0, 0]}>
                  <ErrorBar dataKey="errorY" width={4} stroke="hsl(var(--foreground))" />
                </Bar>
              </BarChart>
            </ResponsiveContainer>
          </div>
        </CardContent>
      </Card>

      {/* Results table */}
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {results.map((result) => (
          <Card key={result.variantKey}>
            <CardHeader className="pb-2">
              <div className="flex items-center justify-between">
                <CardTitle className="text-base">
                  {result.variantKey}
                </CardTitle>
                {result.variantKey !== "control" && (
                  <Badge
                    variant={result.isSignificant ? "default" : "secondary"}
                  >
                    {result.isSignificant ? (
                      <>
                        <CheckCircle2 className="mr-1 h-3 w-3" />
                        Significant
                      </>
                    ) : (
                      <>
                        <Clock className="mr-1 h-3 w-3" />
                        Collecting
                      </>
                    )}
                  </Badge>
                )}
              </div>
            </CardHeader>
            <CardContent className="space-y-2">
              <div className="flex justify-between text-sm">
                <span className="text-muted-foreground">Sample size</span>
                <span className="font-mono">{result.sampleSize.toLocaleString()}</span>
              </div>
              <div className="flex justify-between text-sm">
                <span className="text-muted-foreground">Conversion rate</span>
                <span className="font-mono font-semibold">
                  {(result.conversionRate * 100).toFixed(2)}%
                </span>
              </div>
              <div className="flex justify-between text-sm">
                <span className="text-muted-foreground">95% CI</span>
                <span className="font-mono text-xs">
                  [{(result.confidenceInterval[0] * 100).toFixed(2)}%,{" "}
                  {(result.confidenceInterval[1] * 100).toFixed(2)}%]
                </span>
              </div>
              {result.variantKey !== "control" && (
                <>
                  <div className="flex justify-between text-sm">
                    <span className="text-muted-foreground">p-value</span>
                    <span className="font-mono">{result.pValue.toFixed(4)}</span>
                  </div>
                  <div className="flex justify-between text-sm">
                    <span className="text-muted-foreground">Improvement</span>
                    <span
                      className={cn(
                        "font-mono font-semibold flex items-center gap-1",
                        result.improvement > 0 ? "text-green-600" : "text-red-600"
                      )}
                    >
                      {result.improvement > 0 ? (
                        <ArrowUp className="h-3 w-3" />
                      ) : (
                        <ArrowDown className="h-3 w-3" />
                      )}
                      {Math.abs(result.improvement).toFixed(1)}%
                    </span>
                  </div>
                </>
              )}
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

### Edge Cases e Error Handling

```typescript
// lib/feature-flags/launchdarkly.ts
import * as LaunchDarkly from "@launchdarkly/node-server-sdk";

// ============================================================
// LAUNCHDARKLY INTEGRATION
// ============================================================
let ldClient: LaunchDarkly.LDClient | null = null;

export async function getLDClient(): Promise<LaunchDarkly.LDClient> {
  if (ldClient) return ldClient;

  const sdkKey = process.env.LAUNCHDARKLY_SDK_KEY;
  if (!sdkKey) throw new Error("LAUNCHDARKLY_SDK_KEY not set");

  ldClient = LaunchDarkly.init(sdkKey, {
    logger: LaunchDarkly.basicLogger({ level: "warn" }),
  });

  await ldClient.waitForInitialization({ timeout: 5 });
  return ldClient;
}

export async function evaluateLDFlag<T>(
  flagKey: string,
  user: {
    id: string;
    email?: string;
    name?: string;
    plan?: string;
    country?: string;
    custom?: Record<string, string | number | boolean>;
  },
  defaultValue: T
): Promise<T> {
  try {
    const client = await getLDClient();

    const ldContext: LaunchDarkly.LDContext = {
      kind: "user",
      key: user.id,
      email: user.email,
      name: user.name,
      plan: user.plan,
      country: user.country,
      ...user.custom,
    };

    const value = await client.variation(flagKey, ldContext, defaultValue);
    return value as T;
  } catch (error) {
    console.error(`Failed to evaluate LD flag ${flagKey}:`, error);
    return defaultValue;
  }
}

// ============================================================
// VERCEL EDGE CONFIG INTEGRATION
// ============================================================
import { createClient } from "@vercel/edge-config";

const edgeConfig = createClient(process.env.EDGE_CONFIG);

export async function evaluateEdgeFlag(flagKey: string): Promise<boolean> {
  try {
    const value = await edgeConfig.get<boolean>(flagKey);
    return value ?? false;
  } catch {
    return false;
  }
}

export async function getAllEdgeFlags(): Promise<Record<string, unknown>> {
  try {
    const items = await edgeConfig.getAll();
    return items ?? {};
  } catch {
    return {};
  }
}
```

### Errori Comuni da Evitare
- **Sample size insufficiente**: Non dichiarare un vincitore prima di raggiungere significativita statistica (p < 0.05)
- **Peeking problem**: Non controllare i risultati troppo spesso — definisci il sample size target prima
- **Non-deterministic assignment**: L'assegnazione deve essere deterministica (hash dell'userId) per consistenza
- **Missing LaunchDarkly cleanup**: Chiudi il client LD durante lo shutdown dell'applicazione

### Checklist di Verifica
- [ ] L'assegnazione variante e deterministica (stesso utente = stessa variante)
- [ ] Il p-value viene calcolato con Z-test per proporzioni
- [ ] I risultati mostrano confidence intervals
- [ ] Il progress tracker indica quanti campioni mancano
- [ ] Il vincitore viene dichiarato solo con p < 0.05
- [ ] L'integrazione LaunchDarkly ha fallback a default values
- [ ] L'Edge Config di Vercel ha error handling per connessione fallita



---

## FEATURE-FLAG-TESTING-PATTERNS

### Panoramica
Pattern per testare feature flags: mock del provider, test con diverse configurazioni, test di rollout percentage, test dei componenti gated e test E2E.

### Implementazione Completa

```typescript
// __tests__/feature-flags/evaluator.test.ts
import { describe, it, expect, beforeEach, vi } from "vitest";
import { FeatureFlagEvaluator, FlagContext } from "@/lib/feature-flags/evaluator";

// ============================================================
// MOCK PRISMA
// ============================================================
vi.mock("@/lib/prisma", () => ({
  prisma: {
    featureFlag: {
      findUnique: vi.fn(),
      findMany: vi.fn(),
    },
  },
}));

import { prisma } from "@/lib/prisma";

describe("FeatureFlagEvaluator", () => {
  let evaluator: FeatureFlagEvaluator;

  beforeEach(() => {
    evaluator = new FeatureFlagEvaluator();
    evaluator.clearCache();
    vi.clearAllMocks();
  });

  // ----------------------------------------------------------
  // BASIC EVALUATION
  // ----------------------------------------------------------
  it("returns default value for non-existent flag", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue(null);

    const result = await evaluator.evaluate("non-existent", {}, false);
    expect(result).toBe(false);
  });

  it("returns default value for archived flag", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "archived-flag",
      enabled: true,
      archivedAt: new Date(),
      defaultValue: true,
      percentage: 100,
      type: "BOOLEAN",
      rules: [],
      overrides: [],
      variants: [],
    } as any);

    const result = await evaluator.evaluate("archived-flag", {});
    expect(result).toBe(false);
  });

  it("returns default value when flag is disabled", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "disabled-flag",
      enabled: false,
      archivedAt: null,
      defaultValue: false,
      percentage: 100,
      type: "BOOLEAN",
      rules: [],
      overrides: [],
      variants: [],
    } as any);

    const result = await evaluator.evaluate("disabled-flag", {});
    expect(result).toBe(false);
  });

  // ----------------------------------------------------------
  // RULE EVALUATION
  // ----------------------------------------------------------
  it("matches rule with eq operator", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "pro-feature",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 100,
      type: "BOOLEAN",
      rules: [
        { attribute: "plan", operator: "eq", value: "pro", enabled: true, priority: 0 },
      ],
      overrides: [],
      variants: [],
    } as any);

    const proUser: FlagContext = { userId: "u1", plan: "pro" };
    const freeUser: FlagContext = { userId: "u2", plan: "free" };

    expect(await evaluator.evaluate("pro-feature", proUser)).toBe(true);

    evaluator.clearCache();
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "pro-feature",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 100,
      type: "BOOLEAN",
      rules: [
        { attribute: "plan", operator: "eq", value: "pro", enabled: true, priority: 0 },
      ],
      overrides: [],
      variants: [],
    } as any);

    expect(await evaluator.evaluate("pro-feature", freeUser)).toBe(false);
  });

  it("matches rule with in operator", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "beta-feature",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 100,
      type: "BOOLEAN",
      rules: [
        {
          attribute: "country",
          operator: "in",
          value: ["US", "UK", "DE"],
          enabled: true,
          priority: 0,
        },
      ],
      overrides: [],
      variants: [],
    } as any);

    const usUser: FlagContext = { userId: "u1", country: "US" };
    expect(await evaluator.evaluate("beta-feature", usUser)).toBe(true);
  });

  // ----------------------------------------------------------
  // OVERRIDE
  // ----------------------------------------------------------
  it("user override takes precedence over rules", async () => {
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue({
      id: "1",
      key: "test-flag",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 0,
      type: "BOOLEAN",
      rules: [],
      overrides: [
        { userId: "u1", value: true, expiresAt: null },
      ],
      variants: [],
    } as any);

    const ctx: FlagContext = { userId: "u1" };
    expect(await evaluator.evaluate("test-flag", ctx)).toBe(true);
  });

  // ----------------------------------------------------------
  // PERCENTAGE ROLLOUT
  // ----------------------------------------------------------
  it("deterministic percentage rollout", async () => {
    const flag = {
      id: "1",
      key: "rollout-flag",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 50,
      type: "BOOLEAN",
      rules: [],
      overrides: [],
      variants: [],
    };

    // Same user should always get same result
    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue(flag as any);
    const result1 = await evaluator.evaluate("rollout-flag", { userId: "user-abc" });
    evaluator.clearCache();

    vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue(flag as any);
    const result2 = await evaluator.evaluate("rollout-flag", { userId: "user-abc" });

    expect(result1).toBe(result2);
  });

  it("roughly matches target percentage across many users", async () => {
    const flag = {
      id: "1",
      key: "half-rollout",
      enabled: true,
      archivedAt: null,
      defaultValue: false,
      percentage: 50,
      type: "BOOLEAN",
      rules: [],
      overrides: [],
      variants: [],
    };

    let enabledCount = 0;
    const totalUsers = 1000;

    for (let i = 0; i < totalUsers; i++) {
      evaluator.clearCache();
      vi.mocked(prisma.featureFlag.findUnique).mockResolvedValue(flag as any);
      const result = await evaluator.evaluate("half-rollout", {
        userId: `user-${i}`,
      });
      if (result) enabledCount++;
    }

    // Should be roughly 50% (within 10% margin)
    const ratio = enabledCount / totalUsers;
    expect(ratio).toBeGreaterThan(0.4);
    expect(ratio).toBeLessThan(0.6);
  });
});
```

### Varianti e Configurazioni

```typescript
// __tests__/feature-flags/components.test.tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import { FeatureGate, FeatureFlagProvider } from "@/lib/feature-flags/react";

// ============================================================
// TEST FEATURE GATE COMPONENT
// ============================================================
describe("FeatureGate", () => {
  it("renders children when flag is enabled", () => {
    render(
      <FeatureFlagProvider initialFlags={{ "new-ui": true }}>
        <FeatureGate flag="new-ui">
          <div data-testid="new-ui">New UI</div>
        </FeatureGate>
      </FeatureFlagProvider>
    );

    expect(screen.getByTestId("new-ui")).toBeInTheDocument();
  });

  it("renders fallback when flag is disabled", () => {
    render(
      <FeatureFlagProvider initialFlags={{ "new-ui": false }}>
        <FeatureGate flag="new-ui" fallback={<div data-testid="old-ui">Old UI</div>}>
          <div data-testid="new-ui">New UI</div>
        </FeatureGate>
      </FeatureFlagProvider>
    );

    expect(screen.queryByTestId("new-ui")).not.toBeInTheDocument();
    expect(screen.getByTestId("old-ui")).toBeInTheDocument();
  });

  it("renders fallback for missing flag", () => {
    render(
      <FeatureFlagProvider initialFlags={{}}>
        <FeatureGate flag="unknown-flag" fallback={<div data-testid="fallback">Fallback</div>}>
          <div data-testid="gated">Gated Content</div>
        </FeatureGate>
      </FeatureFlagProvider>
    );

    expect(screen.queryByTestId("gated")).not.toBeInTheDocument();
    expect(screen.getByTestId("fallback")).toBeInTheDocument();
  });
});

// ============================================================
// TEST HELPER — create test flag provider
// ============================================================
export function TestFlagProvider({
  flags,
  children,
}: {
  flags: Record<string, unknown>;
  children: React.ReactNode;
}) {
  return (
    <FeatureFlagProvider initialFlags={flags}>{children}</FeatureFlagProvider>
  );
}

// Usage in other tests:
// render(
//   <TestFlagProvider flags={{ "dark-mode": true, "beta-feature": false }}>
//     <MyComponent />
//   </TestFlagProvider>
// );
```

### Edge Cases e Error Handling

```typescript
// lib/feature-flags/middleware-flags.ts
import { NextRequest, NextResponse } from "next/server";

// ============================================================
// MIDDLEWARE-LEVEL FLAG CHECK — for A/B testing entire pages
// ============================================================
export async function handleFlaggedRoutes(req: NextRequest): Promise<NextResponse | null> {
  const path = req.nextUrl.pathname;

  // Check for A/B test routes
  const abTestRoutes: Record<string, { flagKey: string; variantPath: string }> = {
    "/checkout": {
      flagKey: "new-checkout",
      variantPath: "/checkout-v2",
    },
    "/pricing": {
      flagKey: "new-pricing-page",
      variantPath: "/pricing-v2",
    },
  };

  const abTest = abTestRoutes[path];
  if (!abTest) return null;

  // Get user ID from cookie or generate one
  let userId = req.cookies.get("ab-user-id")?.value;
  if (!userId) {
    userId = crypto.randomUUID();
  }

  // Deterministic assignment
  const hash = await crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(`${abTest.flagKey}:${userId}`)
  );
  const hashArray = new Uint8Array(hash);
  const bucket = hashArray[0] % 100;

  // 50% rollout
  if (bucket < 50) {
    const response = NextResponse.rewrite(new URL(abTest.variantPath, req.url));
    response.cookies.set("ab-user-id", userId, {
      httpOnly: true,
      maxAge: 60 * 60 * 24 * 30,
      sameSite: "lax",
    });
    return response;
  }

  const response = NextResponse.next();
  response.cookies.set("ab-user-id", userId, {
    httpOnly: true,
    maxAge: 60 * 60 * 24 * 30,
    sameSite: "lax",
  });
  return response;
}
```

### Errori Comuni da Evitare
- **Test non deterministici**: Usa userId fissi nei test per risultati predicibili
- **Missing TestProvider**: Wrappa sempre i componenti con `TestFlagProvider` nei test
- **Middleware flag evaluation**: Non fare query DB nel middleware — usa Edge Config o cookie-based assignment
- **Flaky percentage test**: Testa con almeno 1000 utenti e margine del 10%

### Checklist di Verifica
- [ ] I test coprono: flag non esistente, disabilitato, archiviato
- [ ] I test delle regole coprono tutti gli operatori (eq, in, gt, etc.)
- [ ] Gli override hanno precedenza sulle regole nei test
- [ ] Il rollout percentage e deterministico (stessi input = stesso output)
- [ ] Il FeatureGate renderizza correttamente children e fallback
- [ ] Il TestFlagProvider semplifica il setup dei test
- [ ] Il middleware A/B test usa cookie per persistenza
