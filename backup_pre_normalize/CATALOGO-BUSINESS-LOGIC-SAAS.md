# CATALOGO-BUSINESS-LOGIC-SAAS

CATALOGO-BUSINESS-LOGIC-SAAS ESPANSO
Next.js 14 + Prisma + Stripe
§ MULTI-TENANCY PATTERNS
Schema-based isolation
typescript
// lib/tenancy/schema-isolation.ts
import { PrismaClient } from '@prisma/client';

export class TenantAwarePrismaClient extends PrismaClient {
  private tenantId: string | null = null;

  setTenantId(tenantId: string) {
    this.tenantId = tenantId;
  }

  getTenantId(): string | null {
    return this.tenantId;
  }

  $extends({
    query: {
      $allModels: {
        async findMany({ args, query }) {
          if (this.tenantId && args?.where) {
            args.where = { ...args.where, tenantId: this.tenantId };
          }
          return query(args);
        },
        async findUnique({ args, query }) {
          if (this.tenantId && args?.where) {
            args.where = { ...args.where, tenantId: this.tenantId };
          }
          return query(args);
        },
        async create({ args, query }) {
          if (this.tenantId && args?.data) {
            args.data = { ...args.data, tenantId: this.tenantId };
          }
          return query(args);
        },
        async update({ args, query }) {
          if (this.tenantId && args?.where) {
            args.where = { ...args.where, tenantId: this.tenantId };
          }
          return query(args);
        },
        async delete({ args, query }) {
          if (this.tenantId && args?.where) {
            args.where = { ...args.where, tenantId: this.tenantId };
          }
          return query(args);
        },
      },
    },
  });
}

// Singleton instance
export const prisma = new TenantAwarePrismaClient();
Row-level security
typescript
// lib/tenancy/row-level-security.ts
export interface TenantContext {
  tenantId: string;
  userId: string;
  role: 'owner' | 'admin' | 'member' | 'viewer';
}

export class RowLevelSecurity {
  static async enforceTenantAccess(
    model: any,
    tenantId: string,
    action: 'read' | 'write' | 'delete'
  ): Promise<boolean> {
    // Check tenant subscription status
    const tenant = await prisma.tenant.findUnique({
      where: { id: tenantId },
      include: { subscription: true },
    });

    if (!tenant || tenant.status === 'SUSPENDED') {
      throw new Error('Tenant access denied');
    }

    // Apply role-based access control
    const userRole = await this.getUserRole(tenantId);
    const permissions = this.getRolePermissions(userRole);

    return permissions.includes(action);
  }

  private static async getUserRole(tenantId: string): Promise<string> {
    // Implementation depends on your auth system
    return 'member';
  }

  private static getRolePermissions(role: string): string[] {
    const permissions: Record<string, string[]> = {
      owner: ['read', 'write', 'delete'],
      admin: ['read', 'write'],
      member: ['read'],
      viewer: ['read'],
    };
    return permissions[role] || [];
  }
}
Tenant context middleware
typescript
// middleware/tenant-context.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/tenancy/schema-isolation';

export async function tenantContextMiddleware(
  request: NextRequest,
  response: NextResponse
) {
  const tenantId = request.headers.get('x-tenant-id') || 
                   request.cookies.get('tenant-id')?.value;

  if (!tenantId) {
    // Try to infer tenant from subdomain
    const host = request.headers.get('host');
    const subdomain = host?.split('.')[0];
    
    if (subdomain && subdomain !== 'www' && subdomain !== 'app') {
      const tenant = await prisma.tenant.findUnique({
        where: { subdomain },
        select: { id: true },
      });
      
      if (tenant) {
        prisma.setTenantId(tenant.id);
        response.cookies.set('tenant-id', tenant.id);
        return response;
      }
    }
    
    return NextResponse.json(
      { error: 'Tenant context required' },
      { status: 400 }
    );
  }

  prisma.setTenantId(tenantId);
  return response;
}

// app/api/[...path]/route.ts usage
export async function middleware(request: NextRequest) {
  const response = NextResponse.next();
  return await tenantContextMiddleware(request, response);
}
Cross-tenant queries prevention
typescript
// lib/tenancy/query-guard.ts
export class QueryGuard {
  static validateTenantScope(
    query: any,
    allowedTenantId: string
  ): { valid: boolean; error?: string } {
    // Deep check for tenantId in where clauses
    const containsTenantId = (obj: any): boolean => {
      if (!obj || typeof obj !== 'object') return false;
      
      if (obj.tenantId !== undefined) {
        return obj.tenantId === allowedTenantId;
      }
      
      if (obj.OR) {
        return obj.OR.every((condition: any) => containsTenantId(condition));
      }
      
      if (obj.AND) {
        return obj.AND.every((condition: any) => containsTenantId(condition));
      }
      
      return Object.values(obj).some((value) => containsTenantId(value));
    };

    if (!containsTenantId(query.where)) {
      return {
        valid: false,
        error: 'Query must be scoped to a single tenant',
      };
    }

    return { valid: true };
  }

  static sanitizeResult(results: any[], tenantId: string): any[] {
    return results.filter((result) => {
      if (result.tenantId && result.tenantId !== tenantId) {
        console.warn(`Cross-tenant data access attempted for tenant: ${tenantId}`);
        return false;
      }
      return true;
    });
  }
}
Tenant-specific customization
typescript
// lib/tenancy/customization.ts
export interface TenantConfig {
  id: string;
  theme: {
    primaryColor: string;
    logoUrl: string;
    customCss?: string;
  };
  features: Record<string, boolean>;
  limits: Record<string, number>;
  metadata: Record<string, any>;
}

export class TenantCustomizationService {
  private configCache = new Map<string, TenantConfig>();

  async getConfig(tenantId: string): Promise<TenantConfig> {
    if (this.configCache.has(tenantId)) {
      return this.configCache.get(tenantId)!;
    }

    const config = await prisma.tenantConfig.findUnique({
      where: { tenantId },
      include: {
        theme: true,
        featureFlags: true,
        customLimits: true,
      },
    });

    const defaultConfig: TenantConfig = {
      id: tenantId,
      theme: {
        primaryColor: '#3b82f6',
        logoUrl: '/default-logo.png',
      },
      features: {
        analytics: true,
        apiAccess: false,
        customDomains: false,
      },
      limits: {
        users: 10,
        storage: 1024, // MB
        apiCalls: 1000,
      },
      metadata: {},
    };

    const mergedConfig = config
      ? this.mergeConfigs(defaultConfig, config)
      : defaultConfig;

    this.configCache.set(tenantId, mergedConfig);
    return mergedConfig;
  }

  private mergeConfigs(defaultConfig: TenantConfig, dbConfig: any): TenantConfig {
    return {
      ...defaultConfig,
      theme: {
        ...defaultConfig.theme,
        ...dbConfig.theme,
      },
      features: {
        ...defaultConfig.features,
        ...Object.fromEntries(
          dbConfig.featureFlags.map((ff: any) => [ff.feature, ff.enabled])
        ),
      },
      limits: {
        ...defaultConfig.limits,
        ...Object.fromEntries(
          dbConfig.customLimits.map((cl: any) => [cl.limitType, cl.value])
        ),
      },
      metadata: dbConfig.metadata || {},
    };
  }

  async updateConfig(
    tenantId: string,
    updates: Partial<TenantConfig>
  ): Promise<TenantConfig> {
    // Update database
    await prisma.$transaction([
      prisma.tenantConfig.upsert({
        where: { tenantId },
        update: {
          theme: updates.theme
            ? { update: updates.theme }
            : undefined,
          featureFlags: updates.features
            ? {
                deleteMany: {},
                create: Object.entries(updates.features).map(
                  ([feature, enabled]) => ({ feature, enabled })
                ),
              }
            : undefined,
          customLimits: updates.limits
            ? {
                deleteMany: {},
                create: Object.entries(updates.limits).map(
                  ([limitType, value]) => ({ limitType, value })
                ),
              }
            : undefined,
          metadata: updates.metadata,
        },
        create: {
          tenantId,
          theme: { create: updates.theme || {} },
          featureFlags: updates.features
            ? {
                create: Object.entries(updates.features).map(
                  ([feature, enabled]) => ({ feature, enabled })
                ),
              }
            : undefined,
          customLimits: updates.limits
            ? {
                create: Object.entries(updates.limits).map(
                  ([limitType, value]) => ({ limitType, value })
                ),
              }
            : undefined,
          metadata: updates.metadata || {},
        },
      }),
    ]);

    // Clear cache
    this.configCache.delete(tenantId);

    return this.getConfig(tenantId);
  }
}
§ SEAT-BASED BILLING
User seats management
typescript
// lib/billing/seats/seat-manager.ts
export interface SeatAllocation {
  totalSeats: number;
  usedSeats: number;
  availableSeats: number;
  pendingInvitations: number;
  overageSeats?: number;
}

export class SeatManager {
  async getAllocation(tenantId: string): Promise<SeatAllocation> {
    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
      include: {
        plan: true,
        _count: {
          select: {
            activeUsers: true,
            pendingInvitations: {
              where: { expiresAt: { gt: new Date() } },
            },
          },
        },
      },
    });

    if (!subscription) {
      throw new Error('No active subscription found');
    }

    const totalSeats = subscription.plan.seats;
    const usedSeats = subscription._count.activeUsers;
    const pendingInvitations = subscription._count.pendingInvitations;
    const overageSeats = Math.max(0, usedSeats - totalSeats);

    return {
      totalSeats,
      usedSeats,
      availableSeats: Math.max(0, totalSeats - usedSeats),
      pendingInvitations,
      overageSeats,
    };
  }

  async canAssignSeat(tenantId: string): Promise<{
    allowed: boolean;
    requiresUpgrade?: boolean;
    overageFee?: number;
  }> {
    const allocation = await this.getAllocation(tenantId);
    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
      include: { plan: true },
    });

    if (allocation.availableSeats > 0) {
      return { allowed: true };
    }

    // Check if plan allows overage
    if (subscription?.plan.allowOverage) {
      const overageFee = subscription.plan.overagePricePerSeat;
      return {
        allowed: true,
        requiresUpgrade: false,
        overageFee,
      };
    }

    return {
      allowed: false,
      requiresUpgrade: true,
    };
  }
}
Seat assignment/revocation
typescript
// lib/billing/seats/seat-assignment.ts
export class SeatAssignmentService {
  async assignSeat(
    tenantId: string,
    userId: string,
    options?: {
      sendInvitation?: boolean;
      role?: string;
      expiresAt?: Date;
    }
  ): Promise<{ success: boolean; message: string }> {
    // Check seat availability
    const seatCheck = await seatManager.canAssignSeat(tenantId);
    
    if (!seatCheck.allowed) {
      if (seatCheck.requiresUpgrade) {
        return {
          success: false,
          message: 'No available seats. Please upgrade your plan.',
        };
      }
    }

    // Check if user already has a seat
    const existingAssignment = await prisma.userSeat.findUnique({
      where: {
        tenantId_userId: {
          tenantId,
          userId,
        },
      },
    });

    if (existingAssignment) {
      if (existingAssignment.status === 'ACTIVE') {
        return {
          success: false,
          message: 'User already has an active seat',
        };
      }
      
      // Reactivate existing seat
      await prisma.userSeat.update({
        where: { id: existingAssignment.id },
        data: {
          status: 'ACTIVE',
          assignedAt: new Date(),
          revokedAt: null,
        },
      });
      
      return {
        success: true,
        message: 'Seat reactivated successfully',
      };
    }

    // Create new seat assignment
    await prisma.userSeat.create({
      data: {
        tenantId,
        userId,
        status: 'ACTIVE',
        role: options?.role || 'MEMBER',
        assignedAt: new Date(),
        ...(options?.expiresAt && { expiresAt: options.expiresAt }),
      },
    });

    // Handle overage billing if applicable
    if (seatCheck.overageFee) {
      await this.handleOverage(tenantId, seatCheck.overageFee);
    }

    // Send invitation if requested
    if (options?.sendInvitation) {
      await this.sendSeatInvitation(tenantId, userId);
    }

    return {
      success: true,
      message: 'Seat assigned successfully',
    };
  }

  async revokeSeat(
    tenantId: string,
    userId: string,
    reason?: string
  ): Promise<void> {
    await prisma.userSeat.update({
      where: {
        tenantId_userId: {
          tenantId,
          userId,
        },
      },
      data: {
        status: 'REVOKED',
        revokedAt: new Date(),
        revocationReason: reason,
      },
    });

    // Log seat revocation for analytics
    await prisma.seatEvent.create({
      data: {
        tenantId,
        userId,
        eventType: 'REVOKED',
        metadata: { reason },
      },
    });
  }

  private async handleOverage(tenantId: string, overageFee: number): Promise<void> {
    // Create overage charge in Stripe
    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
      select: { stripeSubscriptionId: true },
    });

    if (subscription?.stripeSubscriptionId) {
      await stripe.subscriptionItems.createUsageRecord(
        subscription.stripeSubscriptionId,
        {
          quantity: 1,
          timestamp: Math.floor(Date.now() / 1000),
          action: 'increment',
        }
      );
    }

    // Record overage in database
    await prisma.overageCharge.create({
      data: {
        tenantId,
        chargeAmount: overageFee,
        chargeType: 'SEAT_OVERAGE',
        status: 'PENDING',
      },
    });
  }

  private async sendSeatInvitation(
    tenantId: string,
    userId: string
  ): Promise<void> {
    const invitation = await prisma.seatInvitation.create({
      data: {
        tenantId,
        userId,
        token: crypto.randomUUID(),
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000), // 7 days
      },
    });

    // Send email
    await emailService.sendSeatInvitation({
      to: userId, // Assuming userId is email
      tenantId,
      invitationToken: invitation.token,
    });
  }
}
Overage handling
typescript
// lib/billing/seats/overage-handler.ts
export class OverageHandler {
  async calculateOverage(tenantId: string): Promise<{
    seatOverage: number;
    additionalCharges: number;
    nextBillingCycleEstimate: number;
  }> {
    const allocation = await seatManager.getAllocation(tenantId);
    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
      include: { plan: true },
    });

    const seatOverage = allocation.overageSeats || 0;
    const overagePricePerSeat = subscription?.plan.overagePricePerSeat || 0;
    const additionalCharges = seatOverage * overagePricePerSeat;

    // Calculate usage trends
    const usageTrend = await this.calculateUsageTrend(tenantId);
    const projectedOverage = Math.max(0, usageTrend.projectedSeats - allocation.totalSeats);
    const nextBillingCycleEstimate = projectedOverage * overagePricePerSeat;

    return {
      seatOverage,
      additionalCharges,
      nextBillingCycleEstimate,
    };
  }

  private async calculateUsageTrend(tenantId: string): Promise<{
    projectedSeats: number;
    growthRate: number;
  }> {
    const last30Days = await prisma.userSeat.findMany({
      where: {
        tenantId,
        assignedAt: {
          gte: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000),
        },
      },
      orderBy: { assignedAt: 'asc' },
    });

    if (last30Days.length < 2) {
      return { projectedSeats: last30Days.length, growthRate: 0 };
    }

    // Simple linear regression
    const dates = last30Days.map((seat, index) => index);
    const seats = last30Days.map((_, index) => index + 1);

    const n = dates.length;
    const sumX = dates.reduce((a, b) => a + b, 0);
    const sumY = seats.reduce((a, b) => a + b, 0);
    const sumXY = dates.reduce((sum, x, i) => sum + x * seats[i], 0);
    const sumX2 = dates.reduce((sum, x) => sum + x * x, 0);

    const slope = (n * sumXY - sumX * sumY) / (n * sumX2 - sumX * sumX);
    const growthRate = slope * 30; // Project to 30 days

    const projectedSeats = Math.max(
      0,
      seats[seats.length - 1] + growthRate
    );

    return { projectedSeats, growthRate };
  }

  async sendOverageAlerts(tenantId: string): Promise<void> {
    const overage = await this.calculateOverage(tenantId);
    const threshold = 0.8; // 80% usage

    const allocation = await seatManager.getAllocation(tenantId);
    const usagePercentage = allocation.usedSeats / allocation.totalSeats;

    if (usagePercentage >= threshold) {
      // Send warning email
      await emailService.sendOverageWarning({
        tenantId,
        usagePercentage: Math.round(usagePercentage * 100),
        overageSeats: overage.seatOverage,
        additionalCharges: overage.additionalCharges,
      });

      // Create notification
      await prisma.notification.create({
        data: {
          tenantId,
          type: 'OVERAGE_WARNING',
          title: 'High Seat Usage',
          message: `You're using ${Math.round(usagePercentage * 100)}% of your available seats.`,
          metadata: overage,
        },
      });
    }

    if (overage.seatOverage > 0) {
      // Send overage notification
      await emailService.sendOverageNotification({
        tenantId,
        overageSeats: overage.seatOverage,
        additionalCharges: overage.additionalCharges,
      });
    }
  }
}
Seat usage analytics
typescript
// lib/billing/seats/seat-analytics.ts
export interface SeatAnalytics {
  dailyActiveSeats: Array<{ date: string; count: number }>;
  seatUtilizationRate: number;
  averageSeatDuration: number; // in days
  churnRate: number;
  invitationConversionRate: number;
  roleDistribution: Record<string, number>;
}

export class SeatAnalyticsService {
  async getAnalytics(
    tenantId: string,
    timeframe: '7d' | '30d' | '90d' | '1y' = '30d'
  ): Promise<SeatAnalytics> {
    const days = this.getDaysFromTimeframe(timeframe);
    const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);

    const [
      dailyActiveSeats,
      totalSeats,
      seatAssignments,
      invitations,
      roleDistribution,
    ] = await Promise.all([
      this.getDailyActiveSeats(tenantId, startDate),
      this.getTotalSeats(tenantId),
      this.getSeatAssignments(tenantId, startDate),
      this.getInvitationStats(tenantId, startDate),
      this.getRoleDistribution(tenantId),
    ]);

    const seatUtilizationRate = this.calculateUtilizationRate(dailyActiveSeats, totalSeats);
    const averageSeatDuration = this.calculateAverageDuration(seatAssignments);
    const churnRate = this.calculateChurnRate(seatAssignments);
    const invitationConversionRate = this.calculateConversionRate(invitations);

    return {
      dailyActiveSeats,
      seatUtilizationRate,
      averageSeatDuration,
      churnRate,
      invitationConversionRate,
      roleDistribution,
    };
  }

  private async getDailyActiveSeats(
    tenantId: string,
    startDate: Date
  ): Promise<Array<{ date: string; count: number }>> {
    const results = await prisma.$queryRaw<{ date: string; count: number }[]>`
      SELECT 
        DATE(us.assigned_at) as date,
        COUNT(DISTINCT us.user_id) as count
      FROM user_seats us
      WHERE us.tenant_id = ${tenantId}
        AND us.assigned_at >= ${startDate}
        AND (us.revoked_at IS NULL OR us.revoked_at >= ${startDate})
      GROUP BY DATE(us.assigned_at)
      ORDER BY date ASC
    `;

    return results;
  }

  private calculateUtilizationRate(
    dailyActiveSeats: Array<{ date: string; count: number }>,
    totalSeats: number
  ): number {
    if (dailyActiveSeats.length === 0 || totalSeats === 0) return 0;
    
    const totalActiveSeats = dailyActiveSeats.reduce(
      (sum, day) => sum + day.count,
      0
    );
    const totalPossibleSeats = totalSeats * dailyActiveSeats.length;
    
    return totalActiveSeats / totalPossibleSeats;
  }

  private calculateAverageDuration(
    assignments: Array<{ assignedAt: Date; revokedAt: Date | null }>
  ): number {
    if (assignments.length === 0) return 0;
    
    const totalDuration = assignments.reduce((sum, assignment) => {
      const endDate = assignment.revokedAt || new Date();
      const duration = endDate.getTime() - assignment.assignedAt.getTime();
      return sum + duration;
    }, 0);
    
    return totalDuration / assignments.length / (1000 * 60 * 60 * 24); // Convert to days
  }

  private calculateChurnRate(
    assignments: Array<{ revokedAt: Date | null }>
  ): number {
    const totalAssignments = assignments.length;
    const revokedAssignments = assignments.filter(a => a.revokedAt !== null).length;
    
    return totalAssignments > 0 ? revokedAssignments / totalAssignments : 0;
  }

  private calculateConversionRate(
    invitations: Array<{ accepted: boolean }>
  ): number {
    const totalInvitations = invitations.length;
    const acceptedInvitations = invitations.filter(i => i.accepted).length;
    
    return totalInvitations > 0 ? acceptedInvitations / totalInvitations : 0;
  }

  private async getRoleDistribution(tenantId: string): Promise<Record<string, number>> {
    const results = await prisma.userSeat.groupBy({
      by: ['role'],
      where: {
        tenantId,
        status: 'ACTIVE',
      },
      _count: {
        userId: true,
      },
    });

    return results.reduce((acc, { role, _count }) => {
      acc[role] = _count.userId;
      return acc;
    }, {} as Record<string, number>);
  }

  private getDaysFromTimeframe(timeframe: string): number {
    const daysMap: Record<string, number> = {
      '7d': 7,
      '30d': 30,
      '90d': 90,
      '1y': 365,
    };
    return daysMap[timeframe] || 30;
  }
}
Pending invitations count
typescript
// lib/billing/seats/invitation-manager.ts
export class InvitationManager {
  async getPendingInvitations(tenantId: string): Promise<{
    count: number;
    invitations: Array<{
      id: string;
      email: string;
      role: string;
      expiresAt: Date;
      createdAt: Date;
    }>;
  }> {
    const invitations = await prisma.seatInvitation.findMany({
      where: {
        tenantId,
        status: 'PENDING',
        expiresAt: { gt: new Date() },
      },
      include: {
        user: {
          select: {
            email: true,
            name: true,
          },
        },
      },
      orderBy: { createdAt: 'desc' },
    });

    return {
      count: invitations.length,
      invitations: invitations.map((inv) => ({
        id: inv.id,
        email: inv.user.email,
        role: inv.role,
        expiresAt: inv.expiresAt,
        createdAt: inv.createdAt,
      })),
    };
  }

  async resendInvitation(
    invitationId: string,
    tenantId: string
  ): Promise<void> {
    const invitation = await prisma.seatInvitation.findUnique({
      where: { id: invitationId, tenantId },
    });

    if (!invitation) {
      throw new Error('Invitation not found');
    }

    // Update expiration
    await prisma.seatInvitation.update({
      where: { id: invitationId },
      data: {
        expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
        resentAt: new Date(),
        resendCount: { increment: 1 },
      },
    });

    // Resend email
    await emailService.sendSeatInvitation({
      to: invitation.userId,
      tenantId,
      invitationToken: invitation.token,
      isResend: true,
    });
  }

  async cancelInvitation(
    invitationId: string,
    tenantId: string,
    reason?: string
  ): Promise<void> {
    await prisma.seatInvitation.update({
      where: {
        id: invitationId,
        tenantId,
        status: 'PENDING',
      },
      data: {
        status: 'CANCELLED',
        cancelledAt: new Date(),
        cancellationReason: reason,
      },
    });

    // Free up the seat if it was counted
    await this.updateSeatCount(tenantId);
  }

  async acceptInvitation(
    token: string,
    userId: string
  ): Promise<{ success: boolean; tenantId?: string; error?: string }> {
    const invitation = await prisma.seatInvitation.findUnique({
      where: { token },
      include: { tenant: true },
    });

    if (!invitation) {
      return { success: false, error: 'Invalid invitation token' };
    }

    if (invitation.expiresAt < new Date()) {
      return { success: false, error: 'Invitation has expired' };
    }

    if (invitation.status !== 'PENDING') {
      return { success: false, error: 'Invitation is no longer valid' };
    }

    // Check seat availability
    const seatCheck = await seatManager.canAssignSeat(invitation.tenantId);
    if (!seatCheck.allowed) {
      return {
        success: false,
        error: 'No available seats. Please contact the workspace admin.',
      };
    }

    // Assign seat
    await prisma.$transaction(async (tx) => {
      // Update invitation
      await tx.seatInvitation.update({
        where: { id: invitation.id },
        data: {
          status: 'ACCEPTED',
          acceptedAt: new Date(),
        },
      });

      // Create seat assignment
      await tx.userSeat.create({
        data: {
          tenantId: invitation.tenantId,
          userId,
          status: 'ACTIVE',
          role: invitation.role,
          assignedAt: new Date(),
        },
      });

      // Handle overage if applicable
      if (seatCheck.overageFee) {
        await this.handleOverage(invitation.tenantId, seatCheck.overageFee);
      }
    });

    return {
      success: true,
      tenantId: invitation.tenantId,
    };
  }

  private async updateSeatCount(tenantId: string): Promise<void> {
    // Recalculate seat usage
    const allocation = await seatManager.getAllocation(tenantId);
    
    // Update subscription metadata
    await prisma.subscription.update({
      where: { tenantId },
      data: {
        metadata: {
          ...(await prisma.subscription.findUnique({ where: { tenantId } }))
            ?.metadata,
          seatCount: allocation.usedSeats,
          lastSeatRecount: new Date(),
        },
      },
    });
  }
}
§ USAGE-BASED BILLING
Metered billing setup
typescript
// lib/billing/usage/metered-billing.ts
export interface MeteredFeature {
  id: string;
  name: string;
  unit: string;
  aggregation: 'sum' | 'max' | 'last' | 'count';
  pricePerUnit: number;
  includedUnits: number;
  tiers?: Array<{
    from: number;
    to?: number;
    pricePerUnit: number;
  }>;
}

export class MeteredBillingService {
  async setupMeteredBilling(
    tenantId: string,
    features: MeteredFeature[]
  ): Promise<void> {
    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
      include: { plan: true },
    });

    if (!subscription) {
      throw new Error('No subscription found');
    }

    // Create metered items in Stripe
    const stripeSubscription = await stripe.subscriptions.retrieve(
      subscription.stripeSubscriptionId!
    );

    for (const feature of features) {
      // Check if item already exists
      const existingItem = stripeSubscription.items.data.find(
        (item) => item.price.metadata?.featureId === feature.id
      );

      if (!existingItem) {
        // Create new price
        const price = await stripe.prices.create({
          currency: 'usd',
          product: subscription.plan.stripeProductId!,
          recurring: {
            interval: 'month',
            usage_type: 'metered',
          },
          unit_amount: Math.round(feature.pricePerUnit * 100), // Convert to cents
          metadata: {
            featureId: feature.id,
            type: 'metered',
          },
        });

        // Add to subscription
        await stripe.subscriptionItems.create({
          subscription: subscription.stripeSubscriptionId!,
          price: price.id,
          metadata: {
            featureId: feature.id,
            includedUnits: feature.includedUnits.toString(),
          },
        });
      }

      // Store in database
      await prisma.meteredFeature.upsert({
        where: {
          tenantId_featureId: {
            tenantId,
            featureId: feature.id,
          },
        },
        update: {
          name: feature.name,
          unit: feature.unit,
          aggregation: feature.aggregation,
          pricePerUnit: feature.pricePerUnit,
          includedUnits: feature.includedUnits,
          tiers: feature.tiers,
        },
        create: {
          tenantId,
          featureId: feature.id,
          name: feature.name,
          unit: feature.unit,
          aggregation: feature.aggregation,
          pricePerUnit: feature.pricePerUnit,
          includedUnits: feature.includedUnits,
          tiers: feature.tiers,
        },
      });
    }
  }

  async recordUsage(
    tenantId: string,
    featureId: string,
    quantity: number,
    timestamp?: Date,
    metadata?: Record<string, any>
  ): Promise<void> {
    const feature = await prisma.meteredFeature.findUnique({
      where: {
        tenantId_featureId: {
          tenantId,
          featureId,
        },
      },
    });

    if (!feature) {
      throw new Error(`Feature ${featureId} not configured for metered billing`);
    }

    const subscription = await prisma.subscription.findUnique({
      where: { tenantId },
    });

    if (!subscription?.stripeSubscriptionId) {
      throw new Error('No active Stripe subscription found');
    }

    // Find the subscription item for this feature
    const stripeSubscription = await stripe.subscriptions.retrieve(
      subscription.stripeSubscriptionId
    );

    const subscriptionItem = stripeSubscription.items.data.find(
      (item) => item.price.metadata?.featureId === featureId
    );

    if (!subscriptionItem) {
      throw new Error(`Subscription item not found for feature ${featureId}`);
    }

    // Record usage in Stripe
    await stripe.subscriptionItems.createUsageRecord(
      subscriptionItem.id,
      {
        quantity,
        timestamp: Math.floor((timestamp || new Date()).getTime() / 1000),
        action: 'increment',
      }
    );

    // Store usage locally for analytics
    await prisma.usageRecord.create({
      data: {
        tenantId,
        featureId,
        quantity,
        recordedAt: timestamp || new Date(),
        metadata: metadata || {},
        subscriptionItemId: subscriptionItem.id,
      },
    });

    // Check for threshold alerts
    await this.checkThresholds(tenantId, featureId, quantity);
  }

  private async checkThresholds(
    tenantId: string,
    featureId: string,
    currentUsage: number
  ): Promise<void> {
    const feature = await prisma.meteredFeature.findUnique({
      where: {
        tenantId_featureId: {
          tenantId,
          featureId,
        },
      },
    });

    if (!feature?.includedUnits) return;

    const usagePercentage = (currentUsage / feature.includedUnits) * 100;
    const thresholds = [80, 90, 95, 100];

    for (const threshold of thresholds) {
      if (usagePercentage >= threshold && usagePercentage < threshold + 5) {
        // Check if alert already sent recently
        const recentAlert = await prisma.usageAlert.findFirst({
          where: {
            tenantId,
            featureId,
            threshold,
            createdAt: {
              gte: new Date(Date.now() - 24 * 60 * 60 * 1000), // Last 24 hours
            },
          },
        });

        if (!recentAlert) {
          // Send alert
          await this.sendThresholdAlert(
            tenantId,
            featureId,
            threshold,
            currentUsage,
            feature.includedUnits
          );
        }
        break;
      }
    }
  }

  private async sendThresholdAlert(
    tenantId: string,
    featureId: string,
    threshold: number,
    currentUsage: number,
    includedUnits: number
  ): Promise<void> {
    // Store alert
    await prisma.usageAlert.create({
      data: {
        tenantId,
        featureId,
        threshold,
        currentUsage,
        includedUnits,
        alertType: threshold === 100 ? 'OVER_LIMIT' : 'WARNING',
      },
    });

    // Send notification
    await emailService.sendUsageAlert({
      tenantId,
      featureId,
      threshold,
      currentUsage,
      includedUnits,
      usagePercentage: Math.round((currentUsage / includedUnits) * 100),
    });
  }
}
Usage tracking
typescript
// lib/billing/usage/usage-tracker.ts
export class UsageTracker {
  private buffer = new Map<string, Array<{
    feature