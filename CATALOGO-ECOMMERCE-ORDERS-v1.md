---
# Integrato da: 12-OUTPUT-ECOMMERCE-ORDERS-PAYMENTS.md
# Data integrazione: 2026-01-29 14:51
# Generato con: Gemini 2.5 Flash / DeepSeek R1
---

Ecco i moduli ORDERS e PAYMENTS completi, come richiesto, con integrazione Stripe, webhook handling, TypeScript strictness e le stime di righe per ciascun file.

---

§ FILE 1: PRISMA/SCHEMA-ECOMMERCE-ORDERS.PRISMA

prisma
// This file defines the Prisma schema for Orders and Payments in an e-commerce application.

// --- User Model (Placeholder - assuming it exists in another schema file or here) ---
// If User model is in a separate file, ensure it's imported or referenced correctly.
model User {
  id              String    @id @default(cuid())
  email           String    @unique
  name            String?
  orders          Order[]
  stripeCustomerId String?  @unique // Link to Stripe Customer
  paymentMethods  PaymentMethod[]
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
}

// --- Order Module ---

model Order {
  id                  String              @id @default(cuid())
  orderNumber         String              @unique // e.g., ORD-2024-XXXXX
  userId              String?
  user                User?               @relation(fields: [userId], references: [id])

  // Statuses
  status              OrderStatus         @default(PENDING)
  paymentStatus       PaymentStatus       @default(PENDING)
  fulfillmentStatus   FulfillmentStatus   @default(UNFULFILLED)

  // Items
  items               OrderItem[]

  // Addresses (snapshots at the time of order)
  shippingAddress     Json                // Snapshot of shipping address
  billingAddress      Json                // Snapshot of billing address

  // Totals
  subtotal            Decimal             @db.Decimal(10, 2) // Sum of item prices
  discount            Decimal             @default(0) @db.Decimal(10, 2) // Total discount applied
  shippingCost        Decimal             @default(0) @db.Decimal(10, 2) // Shipping charges
  tax                 Decimal             @default(0) @db.Decimal(10, 2) // Total tax
  total               Decimal             @db.Decimal(10, 2) // subtotal - discount + shipping + tax

  // Payment Information
  paymentMethodId     String?             // ID of the payment method used (e.g., Stripe PaymentMethod ID)
  paymentIntentId     String?             @unique // Stripe PaymentIntent ID for this order
  currency            String              @default("EUR") // Currency code, e.g., "EUR", "USD"

  // Shipping Information
  shippingMethod      String?             // e.g., "Standard Shipping", "Express"
  trackingNumber      String?
  trackingUrl         String?
  shippedAt           DateTime?
  deliveredAt         DateTime?

  // Coupon/Promotion
  couponCode          String?
  couponDiscount      Decimal?            @db.Decimal(10, 2) // Discount specifically from a coupon

  // Notes
  customerNote        String?             @db.Text
  internalNote        String?             @db.Text // For internal staff use

  // Timestamps
  createdAt           DateTime            @default(now())
  updatedAt           DateTime            @updatedAt
  cancelledAt         DateTime?
  completedAt         DateTime?           // When the order is fully delivered and closed

  // Relations
  payments            Payment[]           // Records of actual payment transactions
  refunds             Refund[]            // Records of refunds issued for this order
  timeline            OrderEvent[]        // Audit log of order status changes and events

  @@index([userId])
  @@index([status])
  @@index([orderNumber])
  @@index([createdAt])
  @@index([paymentStatus])
  @@index([fulfillmentStatus])
}

// Represents an item within an order.
model OrderItem {
  id          String    @id @default(cuid())
  orderId     String
  order       Order     @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId   String
  variantId   String?   // If products have variants (e.g., size, color)
  name        String
  sku         String?
  imageUrl    String?
  quantity    Int
  price       Decimal   @db.Decimal(10, 2) // Price per unit at the time of order
  subtotal    Decimal   @db.Decimal(10, 2) // quantity * price
  taxRate     Decimal   @default(0) @db.Decimal(5, 2) // e.g., 0.22 for 22%
  taxAmount   Decimal   @default(0) @db.Decimal(10, 2)

  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([orderId])
  @@index([productId])
}

// Audit log for order events and status changes.
model OrderEvent {
  id          String      @id @default(cuid())
  orderId     String
  order       Order       @relation(fields: [orderId], references: [id], onDelete: Cascade)
  type        OrderEventType // e.g., STATUS_CHANGE, PAYMENT_RECEIVED, REFUND_ISSUED, NOTE_ADDED
  description String      @db.Text
  details     Json?       // Optional JSON for additional context (e.g., oldStatus, newStatus, trackingNumber)
  userId      String?     // User who performed the action (e.g., admin)
  createdAt   DateTime    @default(now())

  @@index([orderId])
  @@index([createdAt])
}

// --- Payment Module ---

model Payment {
  id              String          @id @default(cuid())
  orderId         String
  order           Order           @relation(fields: [orderId], references: [id])
  transactionId   String          @unique // Gateway transaction ID (e.g., Stripe Charge ID)
  paymentIntentId String?         // Stripe PaymentIntent ID, if applicable
  method          String          // e.g., "Stripe Card", "PayPal", "Bank Transfer"
  amount          Decimal         @db.Decimal(10, 2)
  currency        String          @default("EUR")
  status          PaymentStatus   @default(PENDING) // PENDING, PAID, FAILED, REFUNDED
  gatewayResponse Json?           // Raw response from payment gateway
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  @@index([orderId])
  @@index([transactionId])
  @@index([paymentIntentId])
  @@index([status])
}

model Refund {
  id              String          @id @default(cuid())
  orderId         String
  order           Order           @relation(fields: [orderId], references: [id])
  paymentId       String?         // Optional: link to a specific payment if refunding part of it
  payment         Payment?        @relation(fields: [paymentId], references: [id])
  refundId        String          @unique // Gateway refund ID (e.g., Stripe Refund ID)
  amount          Decimal         @db.Decimal(10, 2)
  currency        String          @default("EUR")
  reason          String?         // e.g., "Customer cancelled", "Product returned"
  status          RefundStatus    @default(PENDING) // PENDING, SUCCEEDED, FAILED
  gatewayResponse Json?           // Raw response from payment gateway
  createdAt       DateTime        @default(now())
  updatedAt       DateTime        @updatedAt

  @@index([orderId])
  @@index([refundId])
  @@index([status])
}

// --- Payment Method (for saved cards/payment details) ---
model PaymentMethod {
  id              String    @id @default(cuid())
  userId          String
  user            User      @relation(fields: [userId], references: [id])
  stripePaymentMethodId String @unique // Stripe PaymentMethod ID
  type            String    // e.g., "card", "paypal"
  brand           String?   // e.g., "Visa", "Mastercard"
  last4           String?   // Last 4 digits of card
  expMonth        Int?
  expYear         Int?
  isDefault       Boolean   @default(false)
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt

  @@index([userId])
  @@index([stripePaymentMethodId])
}


// --- Enums ---

enum OrderStatus {
  PENDING         // Order created, awaiting payment/confirmation
  CONFIRMED       // Payment received, order confirmed
  PROCESSING      // Order being prepared for shipment
  SHIPPED         // Order has been shipped
  DELIVERED       // Order has been delivered to the customer
  CANCELLED       // Order cancelled by customer or admin
  REFUNDED        // Order fully refunded
  PARTIALLY_REFUNDED // Order partially refunded
}

enum PaymentStatus {
  PENDING         // Payment initiated, not yet confirmed
  PAID            // Payment successfully received
  FAILED          // Payment failed
  REFUNDED        // Payment fully refunded
  PARTIALLY_REFUNDED // Part of the payment has been refunded
  AUTHORIZED      // Payment authorized, but not yet captured (e.g., for pre-orders)
  CAPTURED        // Authorized payment has been captured
}

enum FulfillmentStatus {
  UNFULFILLED       // No items have been shipped
  PARTIALLY_FULFILLED // Some items have been shipped
  FULFILLED         // All items have been shipped
  RETURNED          // Items have been returned
  PARTIALLY_RETURNED // Some items have been returned
}

enum OrderEventType {
  ORDER_CREATED
  STATUS_CHANGE
  PAYMENT_RECEIVED
  PAYMENT_FAILED
  REFUND_ISSUED
  REFUND_FAILED
  SHIPPED
  DELIVERED
  NOTE_ADDED
  ITEM_UPDATED
  ADDRESS_UPDATED
  COUPON_APPLIED
  ORDER_CANCELLED
}

enum RefundStatus {
  PENDING
  SUCCEEDED
  FAILED
}

---

§ FILE 2: SRC/SERVER/SERVICES/ORDER-SERVICE.TS

typescript
import { PrismaClient, Order, OrderItem, Payment, Refund, OrderEvent, OrderStatus, PaymentStatus, FulfillmentStatus, OrderEventType, Prisma } from '@prisma/client';
import { Decimal } from '@prisma/client/runtime/library';
import { PaymentService } from './payment-service'; // Assuming PaymentService exists
import { z } from 'zod'; // For input validation types

// --- Input/Output Types ---
export type CreateOrderInput = z.infer<typeof import('../../lib/validations/order').createOrderInputSchema>;
export type UpdateOrderInput = z.infer<typeof import('../../lib/validations/order').updateOrderInputSchema>;
export type TrackingInfo = { trackingNumber: string; trackingUrl?: string; carrier?: string };
export type RefundInput = z.infer<typeof import('../../lib/validations/order').refundInputSchema>;
export type OrderEventInput = z.infer<typeof import('../../lib/validations/order').orderEventInputSchema>;
export type ListParams = { page?: number; pageSize?: number; orderBy?: string; sortOrder?: 'asc' | 'desc' };
export type OrderListParams = ListParams & {
  userId?: string;
  status?: OrderStatus;
  paymentStatus?: PaymentStatus;
  fulfillmentStatus?: FulfillmentStatus;
  search?: string; // Search by orderNumber, customer name, etc.
  startDate?: Date;
  endDate?: Date;
};

export type PaginatedResult<T> = {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
};

export type OrderStats = {
  totalOrders: number;
  totalRevenue: Decimal;
  pendingOrders: number;
  shippedOrders: number;
  deliveredOrders: number;
  cancelledOrders: number;
};

// --- Notification Types (for sendOrderNotification) ---
export enum NotificationType {
  ORDER_CREATED = 'ORDER_CREATED',
  ORDER_CONFIRMED = 'ORDER_CONFIRMED',
  ORDER_SHIPPED = 'ORDER_SHIPPED',
  ORDER_DELIVERED = 'ORDER_DELIVERED',
  ORDER_CANCELLED = 'ORDER_CANCELLED',
  ORDER_REFUNDED = 'ORDER_REFUNDED',
  PAYMENT_FAILED = 'PAYMENT_FAILED',
}

export class OrderService {
  private prisma: PrismaClient;
  private paymentService: PaymentService; // Dependency injection for PaymentService

  constructor(prisma: PrismaClient, paymentService: PaymentService) {
    this.prisma = prisma;
    this.paymentService = paymentService;
  }

  // --- CRUD Operations ---

  /**
   * Creates a new order.
   * @param data - Order creation data.
   * @returns The created order.
   */
  async create(data: CreateOrderInput): Promise<Order> {
    const { userId, items, shippingAddress, billingAddress, couponCode, couponDiscount, customerNote, ...orderData } = data;

    const orderNumber = await this.generateOrderNumber();
    const subtotal = items.reduce((sum, item) => sum + item.quantity * item.price, 0);
    const total = new Decimal(subtotal)
      .minus(orderData.discount || 0)
      .minus(couponDiscount || 0)
      .plus(orderData.shippingCost || 0)
      .plus(orderData.tax || 0);

    const newOrder = await this.prisma.order.create({
      data: {
        ...orderData,
        orderNumber,
        userId,
        subtotal: new Decimal(subtotal),
        total,
        shippingAddress: shippingAddress as Prisma.InputJsonValue,
        billingAddress: billingAddress as Prisma.InputJsonValue,
        couponCode,
        couponDiscount: couponDiscount ? new Decimal(couponDiscount) : undefined,
        customerNote,
        items: {
          createMany: {
            data: items.map(item => ({
              productId: item.productId,
              variantId: item.variantId,
              name: item.name,
              sku: item.sku,
              imageUrl: item.imageUrl,
              quantity: item.quantity,
              price: new Decimal(item.price),
              subtotal: new Decimal(item.quantity * item.price),
              taxRate: new Decimal(item.taxRate || 0),
              taxAmount: new Decimal(item.taxAmount || 0),
            })),
          },
        },
        timeline: {
          create: {
            type: OrderEventType.ORDER_CREATED,
            description: 'Order created successfully.',
            userId: userId, // Assuming the user creating the order is the customer
          },
        },
      },
      include: { items: true, user: true, timeline: true },
    });

    await this.sendOrderNotification(newOrder, NotificationType.ORDER_CREATED);
    return newOrder;
  }

  /**
   * Retrieves an order by its ID.
   * @param id - The order ID.
   * @returns The order or null if not found.
   */
  async getById(id: string): Promise<Order | null> {
    return this.prisma.order.findUnique({
      where: { id },
      include: { items: true, user: true, payments: true, refunds: true, timeline: true },
    });
  }

  /**
   * Retrieves an order by its order number.
   * @param orderNumber - The unique order number.
   * @returns The order or null if not found.
   */
  async getByOrderNumber(orderNumber: string): Promise<Order | null> {
    return this.prisma.order.findUnique({
      where: { orderNumber },
      include: { items: true, user: true, payments: true, refunds: true, timeline: true },
    });
  }

  /**
   * Retrieves a paginated list of orders for a specific user.
   * @param userId - The ID of the user.
   * @param params - Pagination and filtering parameters.
   * @returns A paginated result of orders.
   */
  async getUserOrders(userId: string, params?: ListParams): Promise<PaginatedResult<Order>> {
    const { page = 1, pageSize = 10, orderBy = 'createdAt', sortOrder = 'desc' } = params || {};

    const where: Prisma.OrderWhereInput = { userId };

    const [orders, total] = await this.prisma.$transaction([
      this.prisma.order.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { [orderBy]: sortOrder },
        include: { items: true },
      }),
      this.prisma.order.count({ where }),
    ]);

    return {
      data: orders,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  /**
   * Retrieves a paginated list of all orders (typically for admin).
   * @param params - Pagination and filtering parameters.
   * @returns A paginated result of orders.
   */
  async list(params?: OrderListParams): Promise<PaginatedResult<Order>> {
    const { page = 1, pageSize = 10, orderBy = 'createdAt', sortOrder = 'desc', userId, status, paymentStatus, fulfillmentStatus, search, startDate, endDate } = params || {};

    const where: Prisma.OrderWhereInput = {
      userId,
      status,
      paymentStatus,
      fulfillmentStatus,
      createdAt: {
        gte: startDate,
        lte: endDate,
      },
      OR: search ? [
        { orderNumber: { contains: search, mode: 'insensitive' } },
        { user: { name: { contains: search, mode: 'insensitive' } } },
        { user: { email: { contains: search, mode: 'insensitive' } } },
      ] : undefined,
    };

    const [orders, total] = await this.prisma.$transaction([
      this.prisma.order.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { [orderBy]: sortOrder },
        include: { items: true, user: true },
      }),
      this.prisma.order.count({ where }),
    ]);

    return {
      data: orders,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  // --- Status Updates ---

  /**
   * Updates the main status of an order and logs the event.
   * @param id - The order ID.
   * @param status - The new order status.
   * @param note - An optional note for the status change.
   * @param changedByUserId - The ID of the user performing the change (e.g., admin).
   * @returns The updated order.
   */
  async updateStatus(id: string, status: OrderStatus, note?: string, changedByUserId?: string): Promise<Order> {
    const order = await this.getById(id);
    if (!order) {
      throw new Error(`Order with ID ${id} not found.`);
    }

    const updatedOrder = await this.prisma.order.update({
      where: { id },
      data: {
        status,
        timeline: {
          create: {
            type: OrderEventType.STATUS_CHANGE,
            description: `Order status changed from ${order.status} to ${status}. ${note || ''}`.trim(),
            details: { oldStatus: order.status, newStatus: status, note },
            userId: changedByUserId,
          },
        },
        ...(status === OrderStatus.DELIVERED && { deliveredAt: new Date(), completedAt: new Date() }),
        ...(status === OrderStatus.CANCELLED && { cancelledAt: new Date() }),
        ...(status === OrderStatus.REFUNDED && { paymentStatus: PaymentStatus.REFUNDED }),
      },
      include: { items: true, user: true, timeline: true },
    });

    // Send specific notifications based on status
    let notificationType: NotificationType | undefined;
    switch (status) {
      case OrderStatus.CONFIRMED: notificationType = NotificationType.ORDER_CONFIRMED; break;
      case OrderStatus.SHIPPED: notificationType = NotificationType.ORDER_SHIPPED; break;
      case OrderStatus.DELIVERED: notificationType = NotificationType.ORDER_DELIVERED; break;
      case OrderStatus.CANCELLED: notificationType = NotificationType.ORDER_CANCELLED; break;
      case OrderStatus.REFUNDED: notificationType = NotificationType.ORDER_REFUNDED; break;
    }
    if (notificationType) {
      await this.sendOrderNotification(updatedOrder, notificationType);
    }

    return updatedOrder;
  }

  /**
   * Confirms an order, typically after successful payment.
   * @param id - The order ID.
   * @returns The confirmed order.
   */
  async confirm(id: string, changedByUserId?: string): Promise<Order> {
    return this.updateStatus(id, OrderStatus.CONFIRMED, 'Order confirmed after successful payment.', changedByUserId);
  }

  /**
   * Marks an order as processing.
   * @param id - The order ID.
   * @returns The updated order.
   */
  async markAsProcessing(id: string, changedByUserId?: string): Promise<Order> {
    return this.updateStatus(id, OrderStatus.PROCESSING, 'Order is now being processed for fulfillment.', changedByUserId);
  }

  /**
   * Marks an order as shipped and adds tracking information.
   * @param id - The order ID.
   * @param trackingInfo - Tracking number and URL.
   * @returns The updated order.
   */
  async ship(id: string, trackingInfo: TrackingInfo, changedByUserId?: string): Promise<Order> {
    const order = await this.getById(id);
    if (!order) {
      throw new Error(`Order with ID ${id} not found.`);
    }

    const updatedOrder = await this.prisma.order.update({
      where: { id },
      data: {
        status: OrderStatus.SHIPPED,
        fulfillmentStatus: FulfillmentStatus.FULFILLED, // Assuming full shipment
        trackingNumber: trackingInfo.trackingNumber,
        trackingUrl: trackingInfo.trackingUrl,
        shippedAt: new Date(),
        timeline: {
          create: {
            type: OrderEventType.SHIPPED,
            description: `Order shipped with tracking number: ${trackingInfo.trackingNumber}.`,
            details: trackingInfo,
            userId: changedByUserId,
          },
        },
      },
      include: { items: true, user: true, timeline: true },
    });

    await this.sendOrderNotification(updatedOrder, NotificationType.ORDER_SHIPPED);
    return updatedOrder;
  }

  /**
   * Marks an order as delivered.
   * @param id - The order ID.
   * @returns The updated order.
   */
  async markAsDelivered(id: string, changedByUserId?: string): Promise<Order> {
    return this.updateStatus(id, OrderStatus.DELIVERED, 'Order marked as delivered.', changedByUserId);
  }

  /**
   * Cancels an order.
   * @param id - The order ID.
   * @param reason - The reason for cancellation.
   * @returns The cancelled order.
   */
  async cancel(id: string, reason: string, changedByUserId?: string): Promise<Order> {
    const order = await this.getById(id);
    if (!order) {
      throw new Error(`Order with ID ${id} not found.`);
    }
    if (order.status === OrderStatus.DELIVERED || order.status === OrderStatus.SHIPPED) {
      throw new Error('Cannot cancel an order that has already been shipped or delivered.');
    }

    // If payment was made, initiate a refund
    if (order.paymentStatus === PaymentStatus.PAID && order.paymentIntentId) {
      try {
        await this.createRefund(id, {
          amount: order.total.toNumber(),
          reason: `Order cancellation: ${reason}`,
          requestedBy: changedByUserId || order.userId || 'system',
        });
        // The refund process will update the payment status
      } catch (error) {
        console.error(`Failed to initiate refund for order ${id} during cancellation:`, error);
        // Decide how to handle this: throw error, mark order as "Cancellation Failed", etc.
        throw new Error(`Order cancellation failed: Could not process refund. ${error instanceof Error ? error.message : String(error)}`);
      }
    }

    return this.updateStatus(id, OrderStatus.CANCELLED, `Order cancelled. Reason: ${reason}`, changedByUserId);
  }

  // --- Payment Handling ---

  /**
   * Updates the payment status of an order.
   * @param id - The order ID.
   * @param status - The new payment status.
   * @param transactionId - Optional: The payment gateway transaction ID.
   * @returns The updated order.
   */
  async updatePaymentStatus(id: string, status: PaymentStatus, transactionId?: string, changedByUserId?: string): Promise<Order> {
    const order = await this.getById(id);
    if (!order) {
      throw new Error(`Order with ID ${id} not found.`);
    }

    const updatedOrder = await this.prisma.order.update({
      where: { id },
      data: {
        paymentStatus: status,
        timeline: {
          create: {
            type: status === PaymentStatus.PAID ? OrderEventType.PAYMENT_RECEIVED : OrderEventType.PAYMENT_FAILED,
            description: `Payment status changed to ${status}. ${transactionId ? `Transaction ID: ${transactionId}` : ''}`,
            details: { oldPaymentStatus: order.paymentStatus, newPaymentStatus: status, transactionId },
            userId: changedByUserId,
          },
        },
        ...(status === PaymentStatus.PAID && { status: OrderStatus.CONFIRMED }), // Auto-confirm on successful payment
      },
      include: { items: true, user: true, timeline: true },
    });

    if (status === PaymentStatus.PAID) {
      await this.sendOrderNotification(updatedOrder, NotificationType.ORDER_CONFIRMED);
    } else if (status === PaymentStatus.FAILED) {
      await this.sendOrderNotification(updatedOrder, NotificationType.PAYMENT_FAILED);
    }

    return updatedOrder;
  }

  /**
   * Processes a successful payment for an order.
   * This method is typically called by a webhook handler after a payment intent succeeds.
   * @param id - The order ID.
   * @param paymentIntentId - The Stripe PaymentIntent ID.
   * @param transactionId - The Stripe Charge ID (or similar).
   * @param paymentMethodId - The Stripe PaymentMethod ID.
   * @param gatewayResponse - The raw response from the payment gateway.
   * @returns The updated order.
   */
  async processPayment(id: string, paymentIntentId: string, transactionId: string, paymentMethodId: string, gatewayResponse: Prisma.InputJsonValue): Promise<Order> {
    const order = await this.getById(id);
    if (!order) {
      throw new Error(`Order with ID ${id} not found.`);
    }
    if (order.paymentStatus === PaymentStatus.PAID) {
      console.warn(`Order ${id} already marked as PAID. Skipping duplicate payment processing.`);
      return order;
    }

    // Create a Payment record
    await this.prisma.payment.create({
      data: {
        orderId: id,
        transactionId: transactionId,
        paymentIntentId: paymentIntentId,
        method: 'Stripe Card', // Or dynamically determine
        amount: order.total,
        currency: order.currency,
        status: PaymentStatus.PAID,
        gatewayResponse,
      },
    });

    return this.prisma.order.update({
      where: { id },
      data: {
        paymentStatus: PaymentStatus.PAID,
        status: OrderStatus.CONFIRMED, // Automatically confirm order
        paymentIntentId: paymentIntentId,
        paymentMethodId: paymentMethodId,
        timeline: {
          create: {
            type: OrderEventType.PAYMENT_RECEIVED,
            description: `Payment of ${order.total} ${order.currency} received. PaymentIntent ID: ${paymentIntentId}, Transaction ID: ${transactionId}.`,
            details: { paymentIntentId, transactionId, amount: order.total, currency: order.currency },
          },
        },
      },
      include: { items: true, user: true, timeline: true },
    });
  }

  // --- Refunds ---

  /**
   * Initiates a refund for an order. This creates a Refund record and calls the payment service.
   * @param orderId - The ID of the order to refund.
   * @param data - Refund details.
   * @returns The created Refund record.
   */
  async createRefund(orderId: string, data: RefundInput): Promise<Refund> {
    const { amount, reason, requestedBy } = data;
    const order = await this.getById(orderId);
    if (!order) {
      throw new Error(`Order with ID ${orderId} not found.`);
    }
    if (!order.paymentIntentId) {
      throw new Error(`Order ${orderId} does not have a payment intent to refund.`);
    }
    if (order.paymentStatus !== PaymentStatus.PAID && order.paymentStatus !== PaymentStatus.PARTIALLY_REFUNDED) {
      throw new Error(`Order ${orderId} payment status is ${order.paymentStatus}, cannot issue refund.`);
    }

    const refundAmount = new Decimal(amount);
    if (refundAmount.greaterThan(order.total)) {
      throw new Error(`Refund amount ${refundAmount} exceeds order total ${order.total}.`);
    }

    // Create a pending refund record
    const newRefund = await this.prisma.refund.create({
      data: {
        orderId,
        amount: refundAmount,
        currency: order.currency,
        reason,
        status: RefundStatus.PENDING,
        // paymentId: // Optionally link to a specific payment if multiple exist
      },
    });

    // Call payment service to process the actual refund with the gateway
    try {
      const stripeRefund = await this.paymentService.createRefund(order.paymentIntentId, refundAmount.toNumber());

      // Update the refund record with gateway details
      const updatedRefund = await this.prisma.refund.update({
        where: { id: newRefund.id },
        data: {
          refundId: stripeRefund.id,
          status: RefundStatus.SUCCEEDED, // Stripe refund is typically immediate
          gatewayResponse: stripeRefund as Prisma.InputJsonValue,
        },
      });

      // Update order payment status and timeline
      const currentRefundedAmount = (await this.prisma.refund.aggregate({
        _sum: { amount: true },
        where: { orderId, status: RefundStatus.SUCCEEDED },
      }))._sum.amount || new Decimal(0);

      const newTotalRefunded = currentRefundedAmount.plus(refundAmount);
      let newPaymentStatus: PaymentStatus;
      let newOrderStatus: OrderStatus;

      if (newTotalRefunded.equals(order.total)) {
        newPaymentStatus = PaymentStatus.REFUNDED;
        newOrderStatus = OrderStatus.REFUNDED;
      } else {
        newPaymentStatus = PaymentStatus.PARTIALLY_REFUNDED;
        newOrderStatus = OrderStatus.PARTIALLY_REFUNDED;
      }

      await this.prisma.order.update({
        where: { id: orderId },
        data: {
          paymentStatus: newPaymentStatus,
          status: newOrderStatus,
          timeline: {
            create: {
              type: OrderEventType.REFUND_ISSUED,
              description: `Refund of ${refundAmount} ${order.currency} issued. Reason: ${reason}. Refund ID: ${stripeRefund.id}.`,
              details: { amount: refundAmount, reason, refundId: stripeRefund.id, paymentIntentId: order.paymentIntentId },
              userId: requestedBy,
            },
          },
        },
      });

      await this.sendOrderNotification(order, NotificationType.ORDER_REFUNDED); // Or PARTIALLY_REFUNDED
      return updatedRefund;
    } catch (error) {
      // Mark refund as failed if gateway call fails
      await this.prisma.refund.update({
        where: { id: newRefund.id },
        data: {
          status: RefundStatus.FAILED,
          gatewayResponse: { error: error instanceof Error ? error.message : String(error) } as Prisma.InputJsonValue,
        },
      });
      await this.addEvent(orderId, {
        type: OrderEventType.REFUND_FAILED,
        description: `Refund initiation failed for ${refundAmount} ${order.currency}. Error: ${error instanceof Error ? error.message : String(error)}`,
        userId: requestedBy,
      });
      throw new Error(`Failed to process refund via payment gateway: ${error instanceof Error ? error.message : String(error)}`);
    }
  }

  /**
   * Processes a refund (e.g., updates status after webhook confirmation).
   * Note: In Stripe, refund creation is typically synchronous, so this might be less used.
   * @param refundId - The ID of the refund record.
   * @param gatewayRefundId - The actual refund ID from the payment gateway.
   * @param status - The final status of the refund.
   * @param gatewayResponse - The raw response from the payment gateway.
   * @returns The updated Refund record.
   */
  async processRefund(refundId: string, gatewayRefundId: string, status: RefundStatus, gatewayResponse: Prisma.InputJsonValue): Promise<Refund> {
    const refund = await this.prisma.refund.findUnique({ where: { id: refundId } });
    if (!refund) {
      throw new Error(`Refund with ID ${refundId} not found.`);
    }

    const updatedRefund = await this.prisma.refund.update({
      where: { id: refundId },
      data: {
        refundId: gatewayRefundId,
        status,
        gatewayResponse,
      },
    });

    // Update order status based on refund
    if (status === RefundStatus.SUCCEEDED) {
      const order = await this.getById(refund.orderId);
      if (order) {
        const totalRefunded = (await this.prisma.refund.aggregate({
          _sum: { amount: true },
          where: { orderId: order.id, status: RefundStatus.SUCCEEDED },
        }))._sum.amount || new Decimal(0);

        let newPaymentStatus: PaymentStatus;
        let newOrderStatus: OrderStatus;

        if (totalRefunded.equals(order.total)) {
          newPaymentStatus = PaymentStatus.REFUNDED;
          newOrderStatus = OrderStatus.REFUNDED;
        } else {
          newPaymentStatus = PaymentStatus.PARTIALLY_REFUNDED;
          newOrderStatus = OrderStatus.PARTIALLY_REFUNDED;
        }

        await this.prisma.order.update({
          where: { id: order.id },
          data: {
            paymentStatus: newPaymentStatus,
            status: newOrderStatus,
            timeline: {
              create: {
                type: OrderEventType.REFUND_ISSUED,
                description: `Refund of ${refund.amount} ${refund.currency} processed successfully. Refund ID: ${gatewayRefundId}.`,
                details: { amount: refund.amount, refundId: gatewayRefundId },
              },
            },
          },
        });
      }
    } else if (status === RefundStatus.FAILED) {
      await this.addEvent(refund.orderId, {
        type: OrderEventType.REFUND_FAILED,
        description: `Refund ${gatewayRefundId} failed.`,
        details: { gatewayResponse },
      });
    }

    return updatedRefund;
  }

  // --- Timeline / Audit Log ---

  /**
   * Adds an event to an order's timeline.
   * @param orderId - The ID of the order.
   * @param event - The event data.
   * @returns The created OrderEvent.
   */
  async addEvent(orderId: string, event: OrderEventInput): Promise<OrderEvent> {
    return this.prisma.orderEvent.create({
      data: {
        orderId,
        type: event.type,
        description: event.description,
        details: event.details as Prisma.InputJsonValue,
        userId: event.userId,
      },
    });
  }

  /**
   * Retrieves the timeline events for an order.
   * @param orderId - The ID of the order.
   * @returns An array of OrderEvent objects, sorted by creation date.
   */
  async getTimeline(orderId: string): Promise<OrderEvent[]> {
    return this.prisma.orderEvent.findMany({
      where: { orderId },
      orderBy: { createdAt: 'asc' },
    });
  }

  // --- Helpers ---

  /**
   * Generates a unique order number in the format ORD-YYYY-XXXXX.
   * @returns A unique order number string.
   */
  async generateOrderNumber(): Promise<string> {
    const year = new Date().getFullYear();
    let counter = 0;
    let orderNumber: string;

    do {
      const lastOrder = await this.prisma.order.findFirst({
        where: { orderNumber: { startsWith: `ORD-${year}-` } },
        orderBy: { createdAt: 'desc' },
        select: { orderNumber: true },
      });

      if (lastOrder) {
        const lastCounter = parseInt(lastOrder.orderNumber.split('-')[2], 10);
        counter = lastCounter + 1;
      } else {
        counter = 1;
      }
      orderNumber = `ORD-${year}-${String(counter).padStart(5, '0')}`;
    } while (await this.prisma.order.findUnique({ where: { orderNumber } }));

    return orderNumber;
  }

  /**
   * Sends a notification (e.g., email) about an order event.
   * This is a placeholder and would typically integrate with an email/notification service.
   * @param order - The order object.
   * @param type - The type of notification to send.
   */
  private async sendOrderNotification(order: Order, type: NotificationType): Promise<void> {
    // In a real application, this would integrate with an email service (e.g., SendGrid, Nodemailer)
    // or a notification system (e.g., push notifications, SMS).
    console.log(`--- Sending Notification for Order ${order.orderNumber} (${type}) ---`);
    console.log(`To: ${order.user?.email || 'Guest'}`);
    console.log(`Subject: Your Order ${order.orderNumber} - ${type.replace(/_/g, ' ').toLowerCase()}`);
    console.log('Details:', order);
    console.log('----------------------------------------------------');
    // Example:
    // await emailService.sendEmail({
    //   to: order.user?.email || order.shippingAddress.email,
    //   subject: `Your Order ${order.orderNumber} - ${type}`,
    //   template: `order-${type.toLowerCase()}-template`,
    //   data: { order, user: order.user },
    // });
  }

  // --- Reports ---

  /**
   * Retrieves key statistics for orders within a given date range.
   * @param startDate - The start date for the report.
   * @param endDate - The end date for the report.
   * @returns Order statistics.
   */
  async getOrderStats(startDate: Date, endDate: Date): Promise<OrderStats> {
    const whereClause: Prisma.OrderWhereInput = {
      createdAt: {
        gte: startDate,
        lte: endDate,
      },
    };

    const [totalOrders, totalRevenueResult, pendingOrders, shippedOrders, deliveredOrders, cancelledOrders] = await this.prisma.$transaction([
      this.prisma.order.count({ where: whereClause }),
      this.prisma.order.aggregate({
        _sum: { total: true },
        where: { ...whereClause, paymentStatus: PaymentStatus.PAID },
      }),
      this.prisma.order.count({ where: { ...whereClause, status: OrderStatus.PENDING } }),
      this.prisma.order.count({ where: { ...whereClause, status: OrderStatus.SHIPPED } }),
      this.prisma.order.count({ where: { ...whereClause, status: OrderStatus.DELIVERED } }),
      this.prisma.order.count({ where: { ...whereClause, status: OrderStatus.CANCELLED } }),
    ]);

    const totalRevenue = totalRevenueResult._sum.total || new Decimal(0);

    return {
      totalOrders,
      totalRevenue,
      pendingOrders,
      shippedOrders,
      deliveredOrders,
      cancelledOrders,
    };
  }
}

---

§ FILE 3: SRC/SERVER/SERVICES/PAYMENT-SERVICE.TS

typescript
import Stripe from 'stripe';
import { PrismaClient, Order, User, PaymentStatus, OrderStatus, RefundStatus, Prisma } from '@prisma/client';
import { OrderService, NotificationType } from './order-service'; // Assuming OrderService exists
import { env } from '../../env.mjs'; // Assuming env variables are managed by next-env.d.ts or similar

export class PaymentService {
  private stripe: Stripe;
  private prisma: PrismaClient;
  private orderService: OrderService; // Dependency injection for OrderService

  constructor(prisma: PrismaClient, orderService: OrderService) {
    this.stripe = new Stripe(env.STRIPE_SECRET_KEY, {
      apiVersion: '2023-10-16', // Use a recent API version
      typescript: true,
    });
    this.prisma = prisma;
    this.orderService = orderService;
  }

  // --- Payment Intents ---

  /**
   * Creates a Stripe Payment Intent for a given order.
   * @param order - The order for which to create the payment intent.
   * @returns The Stripe PaymentIntent object.
   */
  async createPaymentIntent(order: Order): Promise<Stripe.PaymentIntent> {
    if (order.total.lessThanOrEqual(0)) {
      throw new Error('Order total must be greater than zero to create a payment intent.');
    }

    const customer = order.userId ? await this.getOrCreateCustomer(order.user!) : undefined;

    const paymentIntent = await this.stripe.paymentIntents.create({
      amount: order.total.mul(100).toNumber(), // Stripe expects amount in cents
      currency: order.currency.toLowerCase(),
      metadata: {
        orderId: order.id,
        orderNumber: order.orderNumber,
        userId: order.userId || 'guest',
      },
      customer: customer?.id,
      // You can add more options like payment_method_types, setup_future_usage, etc.
    });

    // Update order with paymentIntentId
    await this.prisma.order.update({
      where: { id: order.id },
      data: {
        paymentIntentId: paymentIntent.id,
        paymentStatus: PaymentStatus.PENDING,
      },
    });

    return paymentIntent;
  }

  /**
   * Confirms a Stripe Payment Intent. This is typically done client-side or via webhook.
   * @param paymentIntentId - The ID of the Payment Intent to confirm.
   * @returns The confirmed Stripe PaymentIntent object.
   */
  async confirmPaymentIntent(paymentIntentId: string): Promise<Stripe.PaymentIntent> {
    // Note: In most modern Stripe integrations, confirmation happens client-side
    // or automatically via webhooks. This method might be used for server-side confirmation
    // in specific flows.
    const paymentIntent = await this.stripe.paymentIntents.confirm(paymentIntentId);
    return paymentIntent;
  }

  /**
   * Cancels a Stripe Payment Intent.
   * @param paymentIntentId - The ID of the Payment Intent to cancel.
   * @returns The cancelled Stripe PaymentIntent object.
   */
  async cancelPaymentIntent(paymentIntentId: string): Promise<Stripe.PaymentIntent> {
    const paymentIntent = await this.stripe.paymentIntents.cancel(paymentIntentId);
    // Optionally update order status here
    return paymentIntent;
  }

  /**
   * Retrieves a Stripe Payment Intent.
   * @param paymentIntentId - The ID of the Payment Intent.
   * @returns The Stripe PaymentIntent object.
   */
  async retrievePaymentIntent(paymentIntentId: string): Promise<Stripe.PaymentIntent> {
    return this.stripe.paymentIntents.retrieve(paymentIntentId);
  }

  // --- Refunds ---

  /**
   * Creates a refund for a Stripe Payment Intent.
   * @param paymentIntentId - The ID of the Payment Intent to refund.
   * @param amount - The amount to refund in the smallest currency unit (e.g., cents). If not provided, full refund.
   * @returns The Stripe Refund object.
   */
  async createRefund(paymentIntentId: string, amount?: number): Promise<Stripe.Refund> {
    const params: Stripe.RefundCreateParams = {
      payment_intent: paymentIntentId,
      amount: amount ? Math.round(amount * 100) : undefined, // Convert to cents
    };
    const refund = await this.stripe.refunds.create(params);
    return refund;
  }

  // --- Payment Methods ---

  /**
   * Attaches a payment method to a Stripe customer.
   * @param customerId - The Stripe Customer ID.
   * @param paymentMethodId - The Stripe PaymentMethod ID.
   */
  async attachPaymentMethod(customerId: string, paymentMethodId: string): Promise<void> {
    await this.stripe.paymentMethods.attach(paymentMethodId, { customer: customerId });
  }

  /**
   * Lists saved payment methods for a Stripe customer.
   * @param customerId - The Stripe Customer ID.
   * @returns An array of Stripe PaymentMethod objects.
   */
  async listPaymentMethods(customerId: string): Promise<Stripe.PaymentMethod[]> {
    const paymentMethods = await this.stripe.customers.listPaymentMethods(customerId, { type: 'card' });
    return paymentMethods.data;
  }

  /**
   * Detaches a payment method from a Stripe customer.
   * @param paymentMethodId - The Stripe PaymentMethod ID to detach.
   */
  async detachPaymentMethod(paymentMethodId: string): Promise<void> {
    await this.stripe.paymentMethods.detach(paymentMethodId);
  }

  // --- Customers ---

  /**
   * Creates a new Stripe customer.
   * @param user - The user object from your database.
   * @returns The Stripe Customer object.
   */
  async createCustomer(user: User): Promise<Stripe.Customer> {
    const customer = await this.stripe.customers.create({
      email: user.email,
      name: user.name || undefined,
      metadata: {
        userId: user.id,
      },
    });
    await this.prisma.user.update({
      where: { id: user.id },
      data: { stripeCustomerId: customer.id },
    });
    return customer;
  }

  /**
   * Retrieves an existing Stripe customer or creates a new one if not found.
   * @param user - The user object from your database.
   * @returns The Stripe Customer object.
   */
  async getOrCreateCustomer(user: User): Promise<Stripe.Customer> {
    if (user.stripeCustomerId) {
      try {
        const customer = await this.stripe.customers.retrieve(user.stripeCustomerId);
        if (customer && !customer.deleted) {
          return customer as Stripe.Customer;
        }
      } catch (error) {
        console.warn(`Stripe customer ${user.stripeCustomerId} not found or invalid for user ${user.id}. Creating new customer.`);
        // Fall through to create new customer if retrieve fails or customer is deleted
      }
    }
    return this.createCustomer(user);
  }

  /**
   * Updates an existing Stripe customer.
   * @param customerId - The Stripe Customer ID.
   * @param data - Update parameters for the customer.
   * @returns The updated Stripe Customer object.
   */
  async updateCustomer(customerId: string, data: Stripe.CustomerUpdateParams): Promise<Stripe.Customer> {
    return this.stripe.customers.update(customerId, data);
  }

  // --- Webhooks ---

  /**
   * Handles incoming Stripe webhook events.
   * @param payload - The raw request body from Stripe.
   * @param signature - The 'stripe-signature' header.
   */
  async handleWebhook(payload: string, signature: string): Promise<void> {
    let event: Stripe.Event;

    try {
      event = this.stripe.webhooks.constructEvent(
        payload,
        signature,
        env.STRIPE_WEBHOOK_SECRET
      );
    } catch (err) {
      console.error(`⚠️ Webhook signature verification failed.`, err);
      throw new Error(`Webhook Error: ${err instanceof Error ? err.message : String(err)}`);
    }

    console.log(`✅ Stripe Webhook received: ${event.type}`);

    try {
      switch (event.type) {
        case 'payment_intent.succeeded':
          await this.handlePaymentIntentSucceeded(event.data.object as Stripe.PaymentIntent);
          break;
        case 'payment_intent.payment_failed':
          await this.handlePaymentIntentFailed(event.data.object as Stripe.PaymentIntent);
          break;
        case 'charge.refunded':
          await this.handleChargeRefunded(event.data.object as Stripe.Charge);
          break;
        case 'customer.created':
          // Handle customer creation if needed, though we usually create them proactively
          break;
        case 'payment_method.attached':
          await this.handlePaymentMethodAttached(event.data.object as Stripe.PaymentMethod);
          break;
        case 'payment_method.detached':
          await this.handlePaymentMethodDetached(event.data.object as Stripe.PaymentMethod);
          break;
        // Add more event types as needed (e.g., checkout.session.completed, invoice.paid)
        default:
          console.warn(`Unhandled event type ${event.type}`);
      }
    } catch (error) {
      console.error(`Error processing Stripe webhook event ${event.type}:`, error);
      // Depending on the error, you might want to re-throw or log for manual review
      throw error; // Re-throw to ensure webhook endpoint returns a non-200 status
    }
  }

  /**
   * Handles the `payment_intent.succeeded` webhook event.
   * Updates the corresponding order's payment status and confirms the order.
   * @param paymentIntent - The Stripe PaymentIntent object.
   */
  private async handlePaymentIntentSucceeded(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const orderId = paymentIntent.metadata?.orderId;
    if (!orderId) {
      console.error(`PaymentIntent ${paymentIntent.id} succeeded but missing orderId in metadata.`);
      return;
    }

    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    if (!order) {
      console.error(`Order with ID ${orderId} not found for PaymentIntent ${paymentIntent.id}.`);
      return;
    }

    if (order.paymentStatus === PaymentStatus.PAID) {
      console.log(`Order ${orderId} already marked as PAID. Skipping update for PaymentIntent ${paymentIntent.id}.`);
      return;
    }

    const charge = paymentIntent.latest_charge as Stripe.Charge | string | null;
    const transactionId = typeof charge === 'string' ? charge : charge?.id;
    const paymentMethodId = paymentIntent.payment_method as string | null;

    if (!transactionId || !paymentMethodId) {
      console.error(`PaymentIntent ${paymentIntent.id} succeeded but missing charge ID or payment method ID.`);
      return;
    }

    await this.orderService.processPayment(orderId, paymentIntent.id, transactionId, paymentMethodId, paymentIntent as Prisma.InputJsonValue);
    console.log(`Order ${orderId} payment successfully processed and confirmed.`);
  }

  /**
   * Handles the `payment_intent.payment_failed` webhook event.
   * Updates the corresponding order's payment status to FAILED.
   * @param paymentIntent - The Stripe PaymentIntent object.
   */
  private async handlePaymentIntentFailed(paymentIntent: Stripe.PaymentIntent): Promise<void> {
    const orderId = paymentIntent.metadata?.orderId;
    if (!orderId) {
      console.error(`PaymentIntent ${paymentIntent.id} failed but missing orderId in metadata.`);
      return;
    }

    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    if (!order) {
      console.error(`Order with ID ${orderId} not found for failed PaymentIntent ${paymentIntent.id}.`);
      return;
    }

    if (order.paymentStatus === PaymentStatus.FAILED) {
      console.log(`Order ${orderId} already marked as FAILED. Skipping update for PaymentIntent ${paymentIntent.id}.`);
      return;
    }

    await this.orderService.updatePaymentStatus(orderId, PaymentStatus.FAILED, paymentIntent.id);
    console.log(`Order ${orderId} payment failed for PaymentIntent ${paymentIntent.id}.`);
    await this.orderService.addEvent(orderId, {
      type: OrderEventType.PAYMENT_FAILED,
      description: `Payment failed for PaymentIntent ${paymentIntent.id}. Last error: ${paymentIntent.last_payment_error?.message || 'Unknown error'}.`,
      details: { paymentIntentId: paymentIntent.id, error: paymentIntent.last_payment_error },
    });
  }

  /**
   * Handles the `charge.refunded` webhook event.
   * Updates the corresponding order's payment status and creates/updates refund records.
   * @param charge - The Stripe Charge object.
   */
  private async handleChargeRefunded(charge: Stripe.Charge): Promise<void> {
    if (!charge.refunds || charge.refunds.data.length === 0) {
      console.warn(`Charge ${charge.id} was refunded but no refund data found.`);
      return;
    }

    const orderId = charge.metadata?.orderId;
    if (!orderId) {
      console.error(`Charge ${charge.id} refunded but missing orderId in metadata.`);
      return;
    }

    const order = await this.prisma.order.findUnique({ where: { id: orderId } });
    if (!order) {
      console.error(`Order with ID ${orderId} not found for refunded Charge ${charge.id}.`);
      return;
    }

    for (const stripeRefund of charge.refunds.data) {
      // Find or create our internal Refund record
      let internalRefund = await this.prisma.refund.findUnique({ where: { refundId: stripeRefund.id } });

      if (!internalRefund) {
        // This might happen if refund was initiated directly via Stripe dashboard
        // or if our createRefund method didn't complete its update step.
        internalRefund = await this.prisma.refund.create({
          data: {
            orderId: order.id,
            refundId: stripeRefund.id,
            amount: new Prisma.Decimal(stripeRefund.amount / 100), // Convert cents to currency unit
            currency: stripeRefund.currency.toUpperCase(),
            reason: stripeRefund.reason || 'unknown',
            status: RefundStatus.SUCCEEDED,
            gatewayResponse: stripeRefund as Prisma.InputJsonValue,
            // paymentId: // Try to link to a payment if possible
          },
        });
        console.log(`Created new internal refund record ${internalRefund.id} for Stripe refund ${stripeRefund.id}.`);
      } else if (internalRefund.status !== RefundStatus.SUCCEEDED) {
        // Update existing pending refund
        internalRefund = await this.prisma.refund.update({
          where: { id: internalRefund.id },
          data: {
            status: RefundStatus.SUCCEEDED,
            gatewayResponse: stripeRefund as Prisma.InputJsonValue,
          },
        });
        console.log(`Updated internal refund record ${internalRefund.id} to SUCCEEDED for Stripe refund ${stripeRefund.id}.`);
      } else {
        console.log(`Internal refund record ${internalRefund.id} already SUCCEEDED for Stripe refund ${stripeRefund.id}. Skipping update.`);
        continue; // Skip to next refund if already processed
      }

      // Update order status based on total refunds
      const totalRefundedAmount = (await this.prisma.refund.aggregate({
        _sum: { amount: true },
        where: { orderId: order.id, status: RefundStatus.SUCCEEDED },
      }))._sum.amount || new Prisma.Decimal(0);

      let newPaymentStatus: PaymentStatus;
      let newOrderStatus: OrderStatus;

      if (totalRefundedAmount.equals(order.total)) {
        newPaymentStatus = PaymentStatus.REFUNDED;
        newOrderStatus = OrderStatus.REFUNDED;
      } else if (totalRefundedAmount.greaterThan(0)) {
        newPaymentStatus = PaymentStatus.PARTIALLY_REFUNDED;
        newOrderStatus = OrderStatus.PARTIALLY_REFUNDED;
      } else {
        // Should not happen if a refund just occurred
        newPaymentStatus = order.paymentStatus;
        newOrderStatus = order.status;
      }

      await this.prisma.order.update({
        where: { id: order.id },
        data: {
          paymentStatus: newPaymentStatus,
          status: newOrderStatus,
          timeline: {
            create: {
              type: OrderEventType.REFUND_ISSUED,
              description: `Refund of ${internalRefund.amount} ${internalRefund.currency} processed by Stripe. Refund ID: ${stripeRefund.id}.`,
              details: { amount: internalRefund.amount, refundId: stripeRefund.id, chargeId: charge.id },
            },
          },
        },
      });
      await this.orderService.sendOrderNotification(order, NotificationType.ORDER_REFUNDED);
      console.log(`Order ${order.id} payment status updated to ${newPaymentStatus}.`);
    }
  }

  /**
   * Handles the `payment_method.attached` webhook event.
   * Saves the payment method details to the database for future use.
   * @param paymentMethod - The Stripe PaymentMethod object.
   */
  private async handlePaymentMethodAttached(paymentMethod: Stripe.PaymentMethod): Promise<void> {
    const customerId = paymentMethod.customer;
    if (!customerId || typeof customerId !== 'string') {
      console.error(`PaymentMethod ${paymentMethod.id} attached but missing customer ID.`);
      return;
    }

    const user = await this.prisma.user.findUnique({ where: { stripeCustomerId: customerId } });
    if (!user) {
      console.error(`User not found for Stripe Customer ID ${customerId}. Cannot save payment method.`);
      return;
    }

    // Check if it already exists to prevent duplicates
    const existingPm = await this.prisma.paymentMethod.findUnique({
      where: { stripePaymentMethodId: paymentMethod.id },
    });

    if (!existingPm) {
      await this.prisma.paymentMethod.create({
        data: {
          userId: user.id,
          stripePaymentMethodId: paymentMethod.id,
          type: paymentMethod.type,
          brand: paymentMethod.card?.brand,
          last4: paymentMethod.card?.last4,
          expMonth: paymentMethod.card?.exp_month,
          expYear: paymentMethod.card?.exp_year,
          isDefault: false, // Can implement logic to set as default if it's the first one
        },
      });
      console.log(`Payment method ${paymentMethod.id} saved for user ${user.id}.`);
    } else {
      console.log(`Payment method ${paymentMethod.id} already exists for user ${user.id}. Skipping.`);
    }
  }

  /**
   * Handles the `payment_method.detached` webhook event.
   * Removes the payment method details from the database.
   * @param paymentMethod - The Stripe PaymentMethod object.
   */
  private async handlePaymentMethodDetached(paymentMethod: Stripe.PaymentMethod): Promise<void> {
    try {
      await this.prisma.paymentMethod.delete({
        where: { stripePaymentMethodId: paymentMethod.id },
      });
      console.log(`Payment method ${paymentMethod.id} removed from database.`);
    } catch (error) {
      console.error(`Failed to delete payment method ${paymentMethod.id} from database:`, error);
    }
  }
}

---

§ FILE 4: SRC/APP/API/WEBHOOKS/STRIPE/ROUTE.TS

typescript
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { OrderService } from '../../../../server/services/order-service';
import { PaymentService } from '../../../../server/services/payment-service';
import { env } from '../../../../env.mjs'; // Assuming env variables are managed

// Initialize Prisma Client and Services
const prisma = new PrismaClient();
const orderService = new OrderService(prisma, {} as PaymentService); // Temporarily pass empty object, will be set correctly
const paymentService = new PaymentService(prisma, orderService);
// Now set the correct paymentService dependency in orderService
(orderService as any).paymentService = paymentService;


export async function POST(req: NextRequest) {
  const signature = req.headers.get('stripe-signature');

  if (!signature) {
    return new NextResponse('No stripe-signature header found', { status: 400 });
  }

  let event;
  try {
    const rawBody = await req.text(); // Get raw body for signature verification
    await paymentService.handleWebhook(rawBody, signature);
    return new NextResponse(JSON.stringify({ received: true }), { status: 200 });
  } catch (err) {
    console.error('Stripe webhook error:', err);
    return new NextResponse(`Webhook Error: ${err instanceof Error ? err.message : 'Unknown error'}`, { status: 400 });
  }
}

// Ensure Prisma client is disconnected on application shutdown
process.on('beforeExit', () => {
  prisma.$disconnect();
});

---

§ FILE 5: SRC/SERVER/TRPC/ROUTERS/ORDERS.TS

typescript
import { z } from 'zod';
import { protectedProcedure, publicProcedure, router } from '../trpc'; // Adjust path as needed
import { PrismaClient, Order, OrderEvent, Refund, OrderStatus, PaymentStatus } from '@prisma/client';
import { OrderService, PaginatedResult, OrderListParams, RefundInput, OrderEventInput, TrackingInfo } from '../../services/order-service';
import { PaymentService } from '../../services/payment-service';
import {
  createOrderInputSchema,
  orderListParamsSchema,
  updateOrderStatusSchema,
  cancelOrderSchema,
  refundInputSchema,
  orderEventInputSchema,
  trackingInfoSchema,
} from '../../../lib/validations/order'; // Adjust path as needed

// Initialize Prisma Client and Services
const prisma = new PrismaClient();
const orderService = new OrderService(prisma, {} as PaymentService); // Temporarily pass empty object
const paymentService = new PaymentService(prisma, orderService);
// Now set the correct paymentService dependency in orderService
(orderService as any).paymentService = paymentService;


export const ordersRouter = router({
  // --- Public Procedures (e.g., for guest checkout or order lookup by public number) ---

  /**
   * Get an order by its unique order number.
   * Can be public if order number is considered a "secret" or requires additional verification.
   */
  getByOrderNumber: publicProcedure
    .input(z.string())
    .query(async ({ input }) => {
      const order = await orderService.getByOrderNumber(input);
      if (!order) {
        throw new Error('Order not found.');
      }
      // Potentially filter sensitive data for public view
      return order;
    }),

  // --- Protected Procedures (requires user authentication) ---

  /**
   * Create a new order.
   * Assumes the user is authenticated and `ctx.session.user.id` is available.
   */
  create: protectedProcedure
    .input(createOrderInputSchema)
    .mutation(async ({ input, ctx }) => {
      // Ensure userId is set to the authenticated user's ID
      const orderData = { ...input, userId: ctx.session.user.id };
      const order = await orderService.create(orderData);
      return order;
    }),

  /**
   * Get a single order by ID.
   * Ensures the order belongs to the authenticated user or the user is an admin.
   */
  getById: protectedProcedure
    .input(z.string())
    .query(async ({ input, ctx }) => {
      const order = await orderService.getById(input);
      if (!order) {
        throw new Error('Order not found.');
      }
      // Basic authorization: user can only see their own orders
      if (order.userId !== ctx.session.user.id && !ctx.session.user.isAdmin) { // Assuming isAdmin role
        throw new Error('Unauthorized access to order.');
      }
      return order;
    }),

  /**
   * Get a paginated list of orders for the authenticated user.
   */
  getUserOrders: protectedProcedure
    .input(orderListParamsSchema.optional())
    .query(async ({ input, ctx }) => {
      const orders = await orderService.getUserOrders(ctx.session.user.id, input);
      return orders;
    }),

  /**
   * Create a Stripe Payment Intent for an order.
   * This is called from the client-side checkout process.
   */
  createPaymentIntent: protectedProcedure
    .input(z.object({ orderId: z.string() }))
    .mutation(async ({ input, ctx }) => {
      const order = await orderService.getById(input.orderId);
      if (!order || order.userId !== ctx.session.user.id) {
        throw new Error('Order not found or unauthorized.');
      }
      if (order.paymentStatus === PaymentStatus.PAID) {
        throw new Error('Order already paid.');
      }

      const paymentIntent = await paymentService.createPaymentIntent(order);
      return { clientSecret: paymentIntent.client_secret };
    }),

  /**
   * Confirm payment for an order after client-side Stripe confirmation.
   * This is typically a final server-side check/update.
   */
  confirmPayment: protectedProcedure
    .input(z.object({
      orderId: z.string(),
      paymentIntentId: z.string(),
    }))
    .mutation(async ({ input, ctx }) => {
      const order = await orderService.getById(input.orderId);
      if (!order || order.userId !== ctx.session.user.id) {
        throw new Error('Order not found or unauthorized.');
      }
      if (order.paymentIntentId !== input.paymentIntentId) {
        throw new Error('Payment Intent ID mismatch.');
      }

      // The webhook handler for payment_intent.succeeded should ideally handle the final status update.
      // This endpoint can serve as a fallback or for immediate client feedback.
      const stripePaymentIntent = await paymentService.retrievePaymentIntent(input.paymentIntentId);

      if (stripePaymentIntent.status === 'succeeded') {
        // Ensure order is updated if webhook was missed or delayed
        if (order.paymentStatus !== PaymentStatus.PAID) {
          const charge = stripePaymentIntent.latest_charge as Stripe.Charge | string | null;
          const transactionId = typeof charge === 'string' ? charge : charge?.id || 'unknown';
          const paymentMethodId = stripePaymentIntent.payment_method as string || 'unknown';
          return orderService.processPayment(input.orderId, input.paymentIntentId, transactionId, paymentMethodId, stripePaymentIntent as Prisma.InputJsonValue);
        }
        return order; // Already paid, return existing order
      } else if (stripePaymentIntent.status === 'requires_payment_method' || stripePaymentIntent.status === 'requires_action') {
        throw new Error('Payment requires further action or a new payment method.');
      } else {
        throw new Error(`Payment Intent status: ${stripePaymentIntent.status}. Payment not successful.`);
      }
    }),

  /**
   * Get the timeline/audit log for a specific order.
   */
  getTimeline: protectedProcedure
    .input(z.string())
    .query(async ({ input, ctx }) => {
      const order = await orderService.getById(input);
      if (!order || (order.userId !== ctx.session.user.id && !ctx.session.user.isAdmin)) {
        throw new Error('Order not found or unauthorized.');
      }
      return orderService.getTimeline(input);
    }),

  // --- Admin Procedures (requires admin role) ---

  /**
   * Get a paginated list of all orders (admin view).
   */
  list: protectedProcedure
    .input(orderListParamsSchema.optional())
    .query(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const orders = await orderService.list(input);
      return orders;
    }),

  /**
   * Update the status of an order (admin action).
   */
  updateStatus: protectedProcedure
    .input(updateOrderStatusSchema)
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const { id, status, note } = input;
      const updatedOrder = await orderService.updateStatus(id, status, note, ctx.session.user.id);
      return updatedOrder;
    }),

  /**
   * Mark an order as shipped (admin action).
   */
  shipOrder: protectedProcedure
    .input(z.object({
      id: z.string(),
      trackingInfo: trackingInfoSchema,
    }))
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const updatedOrder = await orderService.ship(input.id, input.trackingInfo, ctx.session.user.id);
      return updatedOrder;
    }),

  /**
   * Mark an order as delivered (admin action).
   */
  markAsDelivered: protectedProcedure
    .input(z.string())
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const updatedOrder = await orderService.markAsDelivered(input, ctx.session.user.id);
      return updatedOrder;
    }),

  /**
   * Cancel an order (admin action).
   */
  cancel: protectedProcedure
    .input(cancelOrderSchema)
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const { id, reason } = input;
      const cancelledOrder = await orderService.cancel(id, reason, ctx.session.user.id);
      return cancelledOrder;
    }),

  /**
   * Create a refund for an order (admin action).
   */
  createRefund: protectedProcedure
    .input(refundInputSchema)
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const refund = await orderService.createRefund(input.orderId, { ...input, requestedBy: ctx.session.user.id });
      return refund;
    }),

  /**
   * Add an arbitrary event to an order's timeline (admin action).
   */
  addEvent: protectedProcedure
    .input(orderEventInputSchema)
    .mutation(async ({ input, ctx }) => {
      if (!ctx.session.user.isAdmin) {
        throw new Error('Unauthorized: Admin access required.');
      }
      const event = await orderService.addEvent(input.orderId, { ...input, userId: ctx.session.user.id });
      return event;
    }),
});

// Export types for client-side usage
export type OrdersRouter = typeof ordersRouter;

---

§ FILE 6: SRC/LIB/VALIDATIONS/ORDER.TS

typescript
import { z } from 'zod';
import { OrderStatus, PaymentStatus, FulfillmentStatus, OrderEventType, RefundStatus } from '@prisma/client';

// --- Reusable Schemas ---

export const addressSchema = z.object({
  firstName: z.string().min(1, 'First name is required.'),
  lastName: z.string().min(1, 'Last name is required.'),
  company: z.string().optional(),
  address1: z.string().min(1, 'Address line 1 is required.'),
  address2: z.string().optional(),
  city: z.string().min(1, 'City is required.'),
  state: z.string().optional(), // Can be required for certain countries
  zip: z.string().min(1, 'Zip code is required.'),
  country: z.string().min(1, 'Country is required.'),
  phone: z.string().optional(),
  email: z.string().email('Invalid email address.').optional(), // Optional for address, but often required for order
});

export const orderItemSchema = z.object({
  productId: z.string().min(1, 'Product ID is required.'),
  variantId: z.string().optional(),
  name: z.string().min(1, 'Product name is required.'),
  sku: z.string().optional(),
  imageUrl: z.string().url('Invalid image URL.').optional(),
  quantity: z.number().int().min(1, 'Quantity must be at least 1.'),
  price: z.number().positive('Price must be positive.'),
  taxRate: z.number().min(0).max(1).optional().default(0), // e.g., 0.22 for 22%
  taxAmount: z.number().min(0).optional().default(0),
});

export const trackingInfoSchema = z.object({
  trackingNumber: z.string().min(1, 'Tracking number is required.'),
  trackingUrl: z.string().url('Invalid tracking URL.').optional(),
  carrier: z.string().optional(),
});

// --- Order Specific Schemas ---

export const createOrderInputSchema = z.object({
  userId: z.string().optional(), // Will be set by server for authenticated users
  items: z.array(orderItemSchema).min(1, 'Order must contain at least one item.'),
  shippingAddress: addressSchema,
  billingAddress: addressSchema,
  shippingCost: z.number().min(0).optional().default(0),
  discount: z.number().min(0).optional().default(0),
  tax: z.number().min(0).optional().default(0),
  currency: z.string().length(3, 'Currency must be a 3-letter code (e.g., EUR).').default('EUR'),
  paymentMethodId: z.string().optional(), // If a saved payment method is used
  shippingMethod: z.string().optional(),
  couponCode: z.string().optional(),
  couponDiscount: z.number().min(0).optional(),
  customerNote: z.string().max(1000).optional(),
});

export const updateOrderInputSchema = z.object({
  id: z.string().min(1, 'Order ID is required.'),
  // Add fields that can be updated, e.g., shippingMethod, customerNote
  shippingMethod: z.string().optional(),
  customerNote: z.string().max(1000).optional(),
  internalNote: z.string().max(1000).optional(),
  // For admin updates:
  shippingAddress: addressSchema.optional(),
  billingAddress: addressSchema.optional(),
});

export const updateOrderStatusSchema = z.object({
  id: z.string().min(1, 'Order ID is required.'),
  status: z.nativeEnum(OrderStatus),
  note: z.string().max(500).optional(),
});

export const cancelOrderSchema = z.object({
  id: z.string().min(1, 'Order ID is required.'),
  reason: z.string().min(5, 'Cancellation reason is required.'),
});

export const orderListParamsSchema = z.object({
  page: z.number().int().min(1).optional().default(1),
  pageSize: z.number().int().min(1).max(100).optional().default(10),
  orderBy: z.string().optional().default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).optional().default('desc'),
  userId: z.string().optional(),
  status: z.nativeEnum(OrderStatus).optional(),
  paymentStatus: z.nativeEnum(PaymentStatus).optional(),
  fulfillmentStatus: z.nativeEnum(FulfillmentStatus).optional(),
  search: z.string().optional(),
  startDate: z.coerce.date().optional(),
  endDate: z.coerce.date().optional(),
});

export const refundInputSchema = z.object({
  orderId: z.string().min(1, 'Order ID is required.'),
  amount: z.number().positive('Refund amount must be positive.'),
  reason: z.string().min(5, 'Refund reason is required.'),
  requestedBy: z.string().optional(), // User ID of who requested the refund
});

export const orderEventInputSchema = z.object({
  orderId: z.string().min(1, 'Order ID is required.'),
  type: z.nativeEnum(OrderEventType),
  description: z.string().min(1, 'Event description is required.'),
  details: z.record(z.any()).optional(), // JSON object for additional details
  userId: z.string().optional(), // User who performed the action
});

---

§ FILE 7: SRC/HOOKS/USE-ORDERS.TS

typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { trpc } from '../utils/trpc'; // Adjust path to your tRPC client setup
import { OrderListParams, CreateOrderInput, UpdateOrderInput, TrackingInfo, RefundInput, OrderEventInput } from '../server/services/order-service';
import { OrderStatus } from '@prisma/client';

// --- Query Hooks ---

/**
 * Fetches a paginated list of all orders (admin view).
 * @param params - Pagination and filtering parameters.
 */
export function useOrders(params?: OrderListParams) {
  return trpc.orders.list.useQuery(params, {
    keepPreviousData: true, // Keep old data while fetching new for better UX
  });
}

/**
 * Fetches a single order by its ID.
 * @param id - The ID of the order.
 */
export function useOrder(id: string) {
  return trpc.orders.getById.useQuery(id, {
    enabled: !!id, // Only run query if ID is available
  });
}

/**
 * Fetches a single order by its unique order number.
 * @param orderNumber - The unique order number.
 */
export function useOrderByNumber(orderNumber: string) {
  return trpc.orders.getByOrderNumber.useQuery(orderNumber, {
    enabled: !!orderNumber,
  });
}

/**
 * Fetches a paginated list of orders for the current authenticated user.
 * @param params - Pagination and filtering parameters.
 */
export function useUserOrders(params?: OrderListParams) {
  return trpc.orders.getUserOrders.useQuery(params, {
    keepPreviousData: true,
  });
}

/**
 * Fetches the timeline events for a specific order.
 * @param orderId - The ID of the order.
 */
export function useOrderTimeline(orderId: string) {
  return trpc.orders.getTimeline.useQuery(orderId, {
    enabled: !!orderId,
  });
}

// --- Mutation Hooks ---

/**
 * Hook for creating a new order.
 */
export function useCreateOrder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateOrderInput) => trpc.orders.create.mutate(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['orders.getUserOrders'] }); // Invalidate user's order list
      queryClient.invalidateQueries({ queryKey: ['orders.list'] }); // Invalidate admin's order list
    },
  });
}

/**
 * Hook for updating an order's status (admin action).
 */
export function useUpdateOrderStatus() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { id: string; status: OrderStatus; note?: string }) => trpc.orders.updateStatus.mutate(data),
    onSuccess: (updatedOrder) => {
      queryClient.invalidateQueries({ queryKey: ['orders.getById', updatedOrder.id] });
      queryClient.invalidateQueries({ queryKey: ['orders.list'] });
      queryClient.invalidateQueries({ queryKey: ['orders.getUserOrders', updatedOrder.userId] });
      queryClient.invalidateQueries({ queryKey: ['orders.getTimeline', updatedOrder.id] });
    },
  });
}

/**
 * Hook for cancelling an order (admin or user action, depending on permissions).
 */
export function useCancelOrder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { id: string; reason: string }) => trpc.orders.cancel.mutate(data),
    onSuccess: (cancelledOrder) => {
      queryClient.invalidateQueries({ queryKey: ['orders.getById', cancelledOrder.id] });
      queryClient.invalidateQueries({ queryKey: ['orders.list'] });
      queryClient.invalidateQueries({ queryKey: ['orders.getUserOrders', cancelledOrder.userId] });
      queryClient.invalidateQueries({ queryKey: ['orders.getTimeline', cancelledOrder.id] });
    },
  });
}

/**
 * Hook for marking an order as shipped (admin action).
 */
export function useShipOrder() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { id: string; trackingInfo: TrackingInfo }) => trpc.orders.shipOrder.mutate(data),
    onSuccess: (shippedOrder) => {
      queryClient.invalidateQueries({ queryKey: ['orders.getById', shippedOrder.id] });
      queryClient.invalidateQueries({ queryKey: ['orders.list'] });
      queryClient.invalidateQueries({ queryKey: ['orders.getTimeline', shippedOrder.id] });
    },
  });
}

/**
 * Hook for creating a refund for an order (admin action).
 */
export function useCreateRefund() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: RefundInput) => trpc.orders.createRefund.mutate(data),
    onSuccess: (refund) => {
      queryClient.invalidateQueries({ queryKey: ['orders.getById', refund.orderId] });
      queryClient.invalidateQueries({ queryKey: ['orders.list'] });
      queryClient.invalidateQueries({ queryKey: ['orders.getTimeline', refund.orderId] });
    },
  });
}

/**
 * Hook for creating a Stripe Payment Intent.
 */
export function useCreatePaymentIntent() {
  return useMutation({
    mutationFn: (data: { orderId: string }) => trpc.orders.createPaymentIntent.mutate(data),
  });
}

/**
 * Hook for confirming payment after client-side Stripe interaction.
 */
export function useConfirmPayment() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: { orderId: string; paymentIntentId: string }) => trpc.orders.confirmPayment.mutate(data),
    onSuccess: (confirmedOrder) => {
      queryClient.invalidateQueries({ queryKey: ['orders.getById', confirmedOrder.id] });
      queryClient.invalidateQueries({ queryKey: ['orders.list'] });
      queryClient.invalidateQueries({ queryKey: ['orders.getUserOrders', confirmedOrder.userId] });
      queryClient.invalidateQueries({ queryKey: ['orders.getTimeline', confirmedOrder.id] });
    },
  });
}

---

§ FILE 8: SRC/COMPONENTS/ORDERS/ORDER-LIST.TSX

typescript
'use client';

import React, { useState } from 'react';
import { useUserOrders, useOrders } from '../../hooks/use-orders'; // Adjust path
import { OrderStatus } from '@prisma/client';
import { OrderStatusBadge } from './order-status-badge'; // Adjust path
import Link from 'next/link';
import { format } from 'date-fns';
import { Button } from '../ui/button'; // Assuming a UI library like shadcn/ui
import { Input } from '../ui/input';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '../ui/select';
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '../ui/table';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '../ui/card';
import { PaginationComponent } from '../ui/pagination'; // Custom pagination component
import { useSession } from 'next-auth/react'; // Assuming NextAuth.js for session

interface OrderListProps {
  isAdminView?: boolean;
}

export function OrderList({ isAdminView = false }: OrderListProps) {
  const { data: session } = useSession();
  const [page, setPage] = useState(1);
  const [pageSize, setPageSize] = useState(10);
  const [statusFilter, setStatusFilter] = useState<OrderStatus | undefined>(undefined);
  const [searchQuery, setSearchQuery] = useState('');

  const params = {
    page,
    pageSize,
    status: statusFilter,
    search: searchQuery || undefined,
  };

  const { data, isLoading, isError, error } = isAdminView
    ? useOrders(params)
    : useUserOrders(params);

  if (isLoading) return <p>Loading orders...</p>;
  if (isError) return <p className="text-red-500">Error loading orders: {error?.message}</p>;
  if (!data || data.data.length === 0) return <p>No orders found.</p>;

  const orders = data.data;
  const totalPages = data.totalPages;

  return (
    <Card>
      <CardHeader>
        <CardTitle>{isAdminView ? 'All Orders' : 'Your Orders'}</CardTitle>
        <CardDescription>
          {isAdminView ? 'Manage all customer orders.' : 'View your recent order history.'}
        </CardDescription>
      </CardHeader>
      <CardContent>
        <div className="flex flex-col md:flex-row gap-4 mb-6">
          {isAdminView && (
            <Input
              placeholder="Search by order number or customer..."
              value={searchQuery}
              onChange={(e) => setSearchQuery(e.target.value)}
              className="max-w-sm"
            />
          )}
          <Select
            value={statusFilter || ''}
            onValueChange={(value: OrderStatus | '') => setStatusFilter(value === '' ? undefined : value)}
          >
            <SelectTrigger className="w-[180px]">
              <SelectValue placeholder="Filter by Status" />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="">All Statuses</SelectItem>
              {Object.values(OrderStatus).map((status) => (
                <SelectItem key={status} value={status}>
                  {status.replace(/_/g, ' ')}
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
        </div>

        <div className="overflow-x-auto">
          <Table>
            <TableHeader>
              <TableRow>
                <TableHead>Order #</TableHead>
                {isAdminView && <TableHead>Customer</TableHead>}
                <TableHead>Date</TableHead>
                <TableHead>Total</TableHead>
                <TableHead>Status</TableHead>
                <TableHead>Payment</TableHead>
                <TableHead className="text-right">Actions</TableHead>
              </TableRow>
            </TableHeader>
            <TableBody>
              {orders.map((order) => (
                <TableRow key={order.id}>
                  <TableCell className="font-medium">{order.orderNumber}</TableCell>
                  {isAdminView && <TableCell>{order.user?.name || order.user?.email || 'Guest'}</TableCell>}
                  <TableCell>{format(new Date(order.createdAt), 'MMM dd, yyyy HH:mm')}</TableCell>
                  <TableCell>{order.currency} {order.total.toFixed(2)}</TableCell>
                  <TableCell>
                    <OrderStatusBadge status={order.status} />
                  </TableCell>
                  <TableCell>
                    <OrderStatusBadge status={order.paymentStatus as unknown as OrderStatus} /> {/* Re-use badge for payment status */}
                  </TableCell>
                  <TableCell className="text-right">
                    <Link href={`/orders/${order.id}`}>
                      <Button variant="outline" size="sm">View Details</Button>
                    </Link>
                  </TableCell>
                </TableRow>
              ))}
            </TableBody>
          </Table>
        </div>

        <PaginationComponent
          currentPage={page}
          totalPages={totalPages}
          onPageChange={setPage}
          className="mt-6"
        />
      </CardContent>
    </Card>
  );
}

// Placeholder for a generic Pagination component
interface PaginationProps {
  currentPage: number;
  totalPages: number;
  onPageChange: (page: number) => void;
  className?: string;
}

const PaginationComponent: React.FC<PaginationProps> = ({ currentPage, totalPages, onPageChange, className }) => {
  const pages = Array.from({ length: totalPages }, (_, i) => i + 1);

  return (
    <div className={`flex justify-center items-center space-x-2 ${className}`}>
      <Button
        variant="outline"
        size="sm"
        onClick={() => onPageChange(currentPage - 1)}
        disabled={currentPage === 1}
      >
        Previous
      </Button>
      {pages.map((pageNumber) => (
        <Button
          key={pageNumber}
          variant={pageNumber === currentPage ? 'default' : 'outline'}
          size="sm"
          onClick={() => onPageChange(pageNumber)}
        >
          {pageNumber}
        </Button>
      ))}
      <Button
        variant="outline"
        size="sm"
        onClick={() => onPageChange(currentPage + 1)}
        disabled={currentPage === totalPages}
      >
        Next
      </Button>
    </div>
  );
};

---

§ FILE 9: SRC/COMPONENTS/ORDERS/ORDER-DETAIL.TSX

typescript
'use client';

import React from 'react';
import { useOrder, useCancelOrder, useUpdateOrderStatus, useCreateRefund, useShipOrder } from '../../hooks/use-orders'; // Adjust path
import { OrderStatusBadge } from './order-status-badge'; // Adjust path
import { OrderTimeline } from './order-timeline'; // Adjust path
import { format } from 'date-fns';
import { Order, OrderItem, OrderStatus, PaymentStatus, FulfillmentStatus } from '@prisma/client';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '../ui/card';
import { Separator } from '../ui/separator';
import { Button } from '../ui/button';
import { useToast } from '../ui/use-toast'; // Assuming shadcn/ui toast
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '../ui/alert-dialog';
import { Input } from '../ui/input';
import { Label } from '../ui/label';
import { Textarea } from '../ui/textarea';
import { useSession } from 'next-auth/react'; // For admin checks
import { z } from 'zod';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '../ui/form';

interface OrderDetailProps {
  orderId: string;
}

const addressDisplaySchema = z.object({
  firstName: z.string(),
  lastName: z.string(),
  company: z.string().optional(),
  address1: z.string(),
  address2: z.string().optional(),
  city: z.string(),
  state: z.string().optional(),
  zip: z.string(),
  country: z.string(),
  phone: z.string().optional(),
  email: z.string().email().optional(),
});

type AddressDisplayProps = z.infer<typeof addressDisplaySchema>;

const AddressDisplay: React.FC<{ address: AddressDisplayProps; title: string }> = ({ address, title }) => (
  <Card className="p-4">
    <h4 className="font-semibold mb-2">{title}</h4>
    <p>{address.firstName} {address.lastName}</p>
    {address.company && <p>{address.company}</p>}
    <p>{address.address1}</p>
    {address.address2 && <p>{address.address2}</p>}
    <p>{address.city}, {address.state} {address.zip}</p>
    <p>{address.country}</p>
    {address.phone && <p>Phone: {address.phone}</p>}
    {address.email && <p>Email: {address.email}</p>}
  </Card>
);

const OrderItemDisplay: React.FC<{ item: OrderItem }> = ({ item }) => (
  <div className="flex items-center gap-4 py-2 border-b last:border-b-0">
    {item.imageUrl && (
      <img src={item.imageUrl} alt={item.name} className="w-16 h-16 object-cover rounded" />
    )}
    <div className="flex-1">
      <p className="font-medium">{item.name}</p>
      {item.variantId && <p className="text-sm text-gray-500">Variant: {item.variantId}</p>}
      <p className="text-sm text-gray-500">SKU: {item.sku || 'N/A'}</p>
      <p className="text-sm text-gray-500">Qty: {item.quantity} x {item.price.toFixed(2)} {item.order.currency}</p>
    </div>
    <p className="font-semibold">{item.order.currency} {item.subtotal.toFixed(2)}</p>
  </div>
);

const RefundFormSchema = z.object({
  amount: z.number().positive('Amount must be positive.'),
  reason: z.string().min(5, 'Reason is required.'),
});

const ShipOrderFormSchema = z.object({
  trackingNumber: z.string().min(1, 'Tracking number is required.'),
  trackingUrl: z.string().url('Invalid URL').optional().or(z.literal('')),
  carrier: z.string().optional(),
});

export function OrderDetail({ orderId }: OrderDetailProps) {
  const { data: session } = useSession();
  const isAdmin = session?.user?.isAdmin; // Assuming isAdmin property on session user
  const { data: order, isLoading, isError, error } = useOrder(orderId);
  const { toast } = useToast();

  const cancelMutation = useCancelOrder();
  const updateStatusMutation = useUpdateOrderStatus();
  const createRefundMutation = useCreateRefund();
  const shipOrderMutation = useShipOrder();

  const refundForm = useForm<z.infer<typeof RefundFormSchema>>({
    resolver: zodResolver(RefundFormSchema),
    defaultValues: {
      amount: 0,
      reason: '',
    },
  });

  const shipOrderForm = useForm<z.infer<typeof ShipOrderFormSchema>>({
    resolver: zodResolver(ShipOrderFormSchema),
    defaultValues: {
      trackingNumber: '',
      trackingUrl: '',
      carrier: '',
    },
  });

  React.useEffect(() => {
    if (order) {
      refundForm.setValue('amount', order.total.toNumber());
    }
  }, [order, refundForm]);

  const handleCancelOrder = async (reason: string) => {
    try {
      await cancelMutation.mutateAsync({ id: orderId, reason });
      toast({ title: 'Order Cancelled', description: `Order ${order?.orderNumber} has been cancelled.` });
    } catch (err) {
      toast({ title: 'Error', description: `Failed to cancel order: ${err instanceof Error ? err.message : String(err)}`, variant: 'destructive' });
    }
  };

  const handleUpdateStatus = async (status: OrderStatus, note?: string) => {
    try {
      await updateStatusMutation.mutateAsync({ id: orderId, status, note });
      toast({ title: 'Status Updated', description: `Order status changed to ${status}.` });
    } catch (err) {
      toast({ title: 'Error', description: `Failed to update status: ${err instanceof Error ? err.message : String(err)}`, variant: 'destructive' });
    }
  };

  const handleCreateRefund = async (values: z.infer<typeof RefundFormSchema>) => {
    if (!order) return;
    try {
      await createRefundMutation.mutateAsync({ orderId: order.id, ...values });
      toast({ title: 'Refund Initiated', description: `Refund of ${order.currency} ${values.amount.toFixed(2)} initiated.` });
      refundForm.reset();
    } catch (err) {
      toast({ title: 'Error', description: `Failed to initiate refund: ${err instanceof Error ? err.message : String(err)}`, variant: 'destructive' });
    }
  };

  const handleShipOrder = async (values: z.infer<typeof ShipOrderFormSchema>) => {
    if (!order) return;
    try {
      await shipOrderMutation.mutateAsync({ id: order.id, trackingInfo: values });
      toast({ title: 'Order Shipped', description: `Order ${order.orderNumber} has been marked as shipped.` });
      shipOrderForm.reset();
    } catch (err) {
      toast({ title: 'Error', description: `Failed to ship order: ${err instanceof Error ? err.message : String(err)}`, variant: 'destructive' });
    }
  };


  if (isLoading) return <p>Loading order details...</p>;
  if (isError) return <p className="text-red-500">Error loading order: {error?.message}</p>;
  if (!order) return <p>Order not found.</p>;

  const shippingAddress = addressDisplaySchema.parse(order.shippingAddress);
  const billingAddress = addressDisplaySchema.parse(order.billingAddress);

  const canCancel = order.status !== OrderStatus.CANCELLED && order.status !== OrderStatus.DELIVERED && order.status !== OrderStatus.REFUNDED;
  const canShip = isAdmin && order.status === OrderStatus.PROCESSING && order.fulfillmentStatus === FulfillmentStatus.UNFULFILLED;
  const canDeliver = isAdmin && order.status === OrderStatus.SHIPPED;
  const canRefund = isAdmin && (order.paymentStatus === PaymentStatus.PAID || order.paymentStatus === PaymentStatus.PARTIALLY_REFUNDED);

  return (
    <div className="container mx-auto py-8">
      <Card className="mb-8">
        <CardHeader>
          <CardTitle className="flex items-center justify-between">
            Order #{order.orderNumber}
            <div className="flex gap-2">
              <OrderStatusBadge status={order.status} />
              <OrderStatusBadge status={order.paymentStatus as unknown as OrderStatus} /> {/* Re-use badge for payment status */}
            </div>
          </CardTitle>
          <CardDescription>
            Placed on {format(new Date(order.createdAt), 'MMM dd, yyyy HH:mm')} by {order.user?.name || order.user?.email || 'Guest'}
          </CardDescription>
        </CardHeader>
        <CardContent>
          <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <div>
              <h3 className="text-lg font-semibold mb-4">Order Summary</h3>
              <p><strong>Subtotal:</strong> {order.currency} {order.subtotal.toFixed(2)}</p>
              {order.discount.greaterThan(0) && <p><strong>Discount:</strong> -{order.currency} {order.discount.toFixed(2)}</p>}
              {order.couponCode && <p><strong>Coupon ({order.couponCode}):</strong> -{order.currency} {order.couponDiscount?.toFixed(2) || '0.00'}</p>}
              <p><strong>Shipping:</strong> {order.currency} {order.shippingCost.toFixed(2)}</p>
              <p><strong>Tax:</strong> {order.currency} {order.tax.toFixed(2)}</p>
              <Separator className="my-2" />
              <p className="text-xl font-bold"><strong>Total:</strong> {order.currency} {order.total.toFixed(2)}</p>
            </div>

            <div>
              <h3 className="text-lg font-semibold mb-4">Payment & Shipping</h3>
              <p><strong>Payment Method:</strong> {order.paymentMethodId ? 'Stripe Card' : 'N/A'}</p>
              <p><strong>Payment Intent ID:</strong> {order.paymentIntentId || 'N/A'}</p>
              <p><strong>Shipping Method:</strong> {order.shippingMethod || 'N/A'}</p>
              {order.trackingNumber && (
                <p>
                  <strong>Tracking:</strong>{' '}
                  {order.trackingUrl ? (
                    <a href={order.trackingUrl} target="_blank" rel="noopener noreferrer" className="text-blue-600 hover:underline">
                      {order.trackingNumber}
                    </a>
                  ) : (
                    order.trackingNumber
                  )}
                </p>
              )}
              {order.shippedAt && <p><strong>Shipped At:</strong> {format(new Date(order.shippedAt), 'MMM dd, yyyy HH:mm')}</p>}
              {order.deliveredAt && <p><strong>Delivered At:</strong> {format(new Date(order.deliveredAt), 'MMM dd, yyyy HH:mm')}</p>}
            </div>

            <div>
              <h3 className="text-lg font-semibold mb-4">Notes</h3>
              {order.customerNote && (
                <div className="mb-2">
                  <p className="font-medium">Customer Note:</p>
                  <p className="text-sm text-gray-700">{order.customerNote}</p>
                </div>
              )}
              {order.internalNote && isAdmin && (
                <div>
                  <p className="font-medium">Internal Note:</p>
                  <p className="text-sm text-gray-700">{order.internalNote}</p>
                </div>
              )}
              {!order.customerNote && !order.internalNote && <p className="text-gray-500">No notes for this order.</p>}
            </div>
          </div>

          <Separator className="my-6" />

          <h3 className="text-lg font-semibold mb-4">Items</h3>
          <div className="space-y-4">
            {order.items.map((item) => (
              <OrderItemDisplay key={item.id} item={item} />
            ))}
          </div>

          <Separator className="my-6" />

          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <AddressDisplay address={shippingAddress} title="Shipping Address" />
            <AddressDisplay address={billingAddress} title="Billing Address" />
          </div>

          <Separator className="my-6" />

          <h3 className="text-lg font-semibold mb-4">Order Actions</h3>
          <div className="flex flex-wrap gap-4 mb-6">
            {canCancel && (
              <AlertDialog>
                <AlertDialogTrigger asChild>
                  <Button variant="destructive" disabled={cancelMutation.isLoading}>
                    {cancelMutation.isLoading ? 'Cancelling...' : 'Cancel Order'}
                  </Button>
                </AlertDialogTrigger>
                <AlertDialogContent>
                  <AlertDialogHeader>
                    <AlertDialogTitle>Are you absolutely sure?</AlertDialogTitle>
                    <AlertDialogDescription>
                      This action cannot be undone. This will cancel the order and attempt to refund any payment.
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <div className="grid gap-2">
                    <Label htmlFor="cancel-reason">Reason for cancellation</Label>
                    <Textarea id="cancel-reason" placeholder="e.g., Customer requested, item out of stock" />
                  </div>
                  <AlertDialogFooter>
                    <AlertDialogCancel>Dismiss</AlertDialogCancel>
                    <AlertDialogAction onClick={() => handleCancelOrder((document.getElementById('cancel-reason') as HTMLTextAreaElement)?.value || 'No reason provided')}>
                      Confirm Cancel
                    </AlertDialogAction>
                  </AlertDialogFooter>
                </AlertDialogContent>
              </AlertDialog>
            )}

            {isAdmin && order.status === OrderStatus.CONFIRMED && (
              <Button onClick={() => handleUpdateStatus(OrderStatus.PROCESSING, 'Order moved to processing.')} disabled={updateStatusMutation.isLoading}>
                {updateStatusMutation.isLoading ? 'Processing...' : 'Mark as Processing'}
              </Button>
            )}

            {canShip && (
              <AlertDialog>
                <AlertDialogTrigger asChild>
                  <Button disabled={shipOrderMutation.isLoading}>
                    {shipOrderMutation.isLoading ? 'Shipping...' : 'Mark as Shipped'}
                  </Button>
                </AlertDialogTrigger>
                <AlertDialogContent>
                  <AlertDialogHeader>
                    <AlertDialogTitle>Ship Order</AlertDialogTitle>
                    <AlertDialogDescription>
                      Enter tracking information for this order.
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <Form {...shipOrderForm}>
                    <form onSubmit={shipOrderForm.handleSubmit(handleShipOrder)} className="space-y-4">
                      <FormField
                        control={shipOrderForm.control}
                        name="trackingNumber"
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Tracking Number</FormLabel>
                            <FormControl>
                              <Input {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                      <FormField
                        control={shipOrderForm.control}
                        name="trackingUrl"
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Tracking URL (Optional)</FormLabel>
                            <FormControl>
                              <Input {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                      <FormField
                        control={shipOrderForm.control}
                        name="carrier"
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Carrier (Optional)</FormLabel>
                            <FormControl>
                              <Input {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                      <AlertDialogFooter>
                        <AlertDialogCancel onClick={() => shipOrderForm.reset()}>Cancel</AlertDialogCancel>
                        <Button type="submit" disabled={shipOrderMutation.isLoading}>
                          {shipOrderMutation.isLoading ? 'Shipping...' : 'Confirm Ship'}
                        </Button>
                      </AlertDialogFooter>
                    </form>
                  </Form>
                </AlertDialogContent>
              </AlertDialog>
            )}

            {canDeliver && (
              <Button onClick={() => handleUpdateStatus(OrderStatus.DELIVERED, 'Order marked as delivered.')} disabled={updateStatusMutation.isLoading}>
                {updateStatusMutation.isLoading ? 'Delivering...' : 'Mark as Delivered'}
              </Button>
            )}

            {canRefund && (
              <AlertDialog>
                <AlertDialogTrigger asChild>
                  <Button variant="outline" disabled={createRefundMutation.isLoading}>
                    {createRefundMutation.isLoading ? 'Refunding...' : 'Initiate Refund'}
                  </Button>
                </AlertDialogTrigger>
                <AlertDialogContent>
                  <AlertDialogHeader>
                    <AlertDialogTitle>Initiate Refund</AlertDialogTitle>
                    <AlertDialogDescription>
                      Enter the amount to refund for this order. Max: {order.currency} {order.total.toFixed(2)}
                    </AlertDialogDescription>
                  </AlertDialogHeader>
                  <Form {...refundForm}>
                    <form onSubmit={refundForm.handleSubmit(handleCreateRefund)} className="space-y-4">
                      <FormField
                        control={refundForm.control}
                        name="amount"
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Refund Amount ({order.currency})</FormLabel>
                            <FormControl>
                              <Input type="number" step="0.01" max={order.total.toNumber()} {...field} onChange={e => field.onChange(parseFloat(e.target.value))} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                      <FormField
                        control={refundForm.control}
                        name="reason"
                        render={({ field }) => (
                          <FormItem>
                            <FormLabel>Reason for Refund</FormLabel>
                            <FormControl>
                              <Textarea placeholder="e.g., Customer return, damaged item" {...field} />
                            </FormControl>
                            <FormMessage />
                          </FormItem>
                        )}
                      />
                      <AlertDialogFooter>
                        <AlertDialogCancel onClick={() => refundForm.reset()}>Cancel</AlertDialogCancel>
                        <Button type="submit" disabled={createRefundMutation.isLoading}>
                          {createRefundMutation.isLoading ? 'Refunding...' : 'Confirm Refund'}
                        </Button>
                      </AlertDialogFooter>
                    </form>
                  </Form>
                </AlertDialogContent>
              </AlertDialog>
            )}
          </div>

          <Separator className="my-6" />

          <h3 className="text-lg font-semibold mb-4">Order Timeline</h3>
          <OrderTimeline orderId={order.id} />
        </CardContent>
      </Card>
    </div>
  );
}

---

§ FILE 10: SRC/COMPONENTS/ORDERS/ORDER-STATUS-BADGE.TSX

typescript
import React from 'react';
import { OrderStatus, PaymentStatus, FulfillmentStatus } from '@prisma/client';
import { Badge } from '../ui/badge'; // Assuming shadcn/ui Badge component

interface OrderStatusBadgeProps {
  status: OrderStatus | PaymentStatus | FulfillmentStatus;
  className?: string;
}

export function OrderStatusBadge({ status, className }: OrderStatusBadgeProps) {
  let variant: 'default' | 'secondary' | 'destructive' | 'outline' | 'success' | 'warning' | null = null;
  let text: string = status.replace(/_/g, ' ');

  switch (status) {
    // Order Statuses
    case OrderStatus.PENDING:
      variant = 'warning';
      break;
    case OrderStatus.CONFIRMED:
    case OrderStatus.PROCESSING:
      variant = 'default'; // Or a custom 'info' variant
      break;
    case OrderStatus.SHIPPED:
    case OrderStatus.DELIVERED:
      variant = 'success';
      break;
    case OrderStatus.CANCELLED:
    case OrderStatus.REFUNDED:
      variant = 'destructive';
      break;
    case OrderStatus.PARTIALLY_REFUNDED:
      variant = 'warning';
      break;

    // Payment Statuses
    case PaymentStatus.PENDING:
    case PaymentStatus.AUTHORIZED:
      variant = 'warning';
      text = 'Payment Pending';
      break;
    case PaymentStatus.PAID:
    case PaymentStatus.CAPTURED:
      variant = 'success';
      text = 'Payment Paid';
      break;
    case PaymentStatus.FAILED:
      variant = 'destructive';
      text = 'Payment Failed';
      break;
    case PaymentStatus.REFUNDED:
      variant = 'destructive';
      text = 'Payment Refunded';
      break;
    case PaymentStatus.PARTIALLY_REFUNDED:
      variant = 'warning';
      text = 'Partially Refunded';
      break;

    // Fulfillment Statuses
    case FulfillmentStatus.UNFULFILLED:
      variant = 'warning';
      text = 'Unfulfilled';
      break;
    case FulfillmentStatus.PARTIALLY_FULFILLED:
      variant = 'default';
      text = 'Partially Fulfilled';
      break;
    case FulfillmentStatus.FULFILLED:
      variant = 'success';
      text = 'Fulfilled';
      break;
    case FulfillmentStatus.RETURNED:
      variant = 'destructive';
      text = 'Returned';
      break;
    case FulfillmentStatus.PARTIALLY_RETURNED:
      variant = 'warning';
      text = 'Partially Returned';
      break;

    default:
      variant = 'secondary';
      break;
  }

  // Custom variants for shadcn/ui if needed, e.g., 'success' for green, 'warning' for orange
  // For now, mapping to existing variants or using default/secondary
  const getShadcnVariant = (v: typeof variant) => {
    switch (v) {
      case 'success': return 'default'; // You might need to extend shadcn/ui for custom colors
      case 'warning': return 'secondary'; // Use secondary for warning-like color
      default: return v || 'secondary';
    }
  };

  const colorClass = (v: typeof variant) => {
    switch (v) {
      case 'success': return 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200';
      case 'warning': return 'bg-orange-100 text-orange-800 dark:bg-orange-900 dark:text-orange-200';
      case 'destructive': return 'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-200';
      case 'default': return 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200';
      default: return 'bg-gray-100 text-gray-800 dark:bg-gray-900 dark:text-gray-200';
    }
  };


  return (
    <Badge className={`${colorClass(variant)} ${className}`}>
      {text}
    </Badge>
  );
}

---

§ FILE 11: SRC/COMPONENTS/ORDERS/ORDER-TIMELINE.TSX

typescript
'use client';

import React from 'react';
import { useOrderTimeline } from '../../hooks/use-orders'; // Adjust path
import { format } from 'date-fns';
import { OrderEventType } from '@prisma/client';
import { Card, CardContent } from '../ui/card'; // Assuming shadcn/ui Card

interface OrderTimelineProps {
  orderId: string;
}

export function OrderTimeline({ orderId }: OrderTimelineProps) {
  const { data: timeline, isLoading, isError, error } = useOrderTimeline(orderId);

  if (isLoading) return <p>Loading timeline...</p>;
  if (isError) return <p className="text-red-500">Error loading timeline: {error?.message}</p>;
  if (!timeline || timeline.length === 0) return <p>No events recorded for this order.</p>;

  return (
    <div className="relative pl-6 border-l-2 border-gray-200 dark:border-gray-700">
      {timeline.map((event, index) => (
        <div key={event.id} className="mb-8 last:mb-0">
          <div className="absolute -left-3 mt-1 flex h-6 w-6 items-center justify-center rounded-full bg-blue-500 text-white">
            {/* Icon based on event type could go here */}
            <span className="text-xs">{(timeline.length - index)}</span>
          </div>
          <Card className="ml-4 p-4">
            <CardContent className="p-0">
              <div className="flex justify-between items-start mb-2">
                <h4 className="font-semibold text-base">
                  {event.type.replace(/_/g, ' ').toLowerCase().split(' ').map(word => word.charAt(0).toUpperCase() + word.slice(1)).join(' ')}
                </h4>
                <span className="text-sm text-gray-500 dark:text-gray-400">
                  {format(new Date(event.createdAt), 'MMM dd, yyyy HH:mm')}
                </span>
              </div>
              <p className="text-sm text-gray-700 dark:text-gray-300">{event.description}</p>
              {event.details && Object.keys(event.details as object).length > 0 && (
                <div className="mt-2 text-xs text-gray-600 dark:text-gray-400">
                  <p className="font-medium">Details:</p>
                  <pre className="bg-gray-50 dark:bg-gray-800 p-2 rounded overflow-x-auto">
                    {JSON.stringify(event.details, null, 2)}
                  </pre>
                </div>
              )}
              {event.userId && (
                <p className="mt-2 text-xs text-gray-500 dark:text-gray-400">
                  Performed by User ID: {event.userId}
                </p>
              )}
            </CardContent>
          </Card>
        </div>
      ))}
    </div>
  );
}

---

§ FILE 12: SRC/COMPONENTS/PAYMENT/STRIPE-ELEMENTS.TSX

typescript
'use client';

import React, { useState, useEffect } from 'react';
import { Elements, useStripe, useElements, PaymentElement } from '@stripe/react-stripe-js';
import { loadStripe, StripeElementsOptions } from '@stripe/stripe-js';
import { Button } from '../ui/button'; // Assuming shadcn/ui Button
import { useToast } from '../ui/use-toast'; // Assuming shadcn/ui toast
import { trpc } from '../../utils/trpc'; // Adjust path to your tRPC client setup
import { useRouter } from 'next/navigation';

// Load Stripe.js outside of a component’s render to avoid recreating the Stripe object on every render.
const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY!);

interface CheckoutFormProps {
  orderId: string;
  clientSecret: string;
}

const CheckoutForm: React.FC<CheckoutFormProps> = ({ orderId, clientSecret }) => {
  const stripe = useStripe();
  const elements = useElements();
  const { toast } = useToast();
  const router = useRouter();

  const [isLoading, setIsLoading] = useState(false);

  const confirmPaymentMutation = trpc.orders.confirmPayment.useMutation();

  const handleSubmit = async (event: React.FormEvent) => {
    event.preventDefault();

    if (!stripe || !elements || !clientSecret) {
      // Stripe.js has not yet loaded.
      // Make sure to disable form submission until Stripe.js has loaded.
      return;
    }

    setIsLoading(true);

    const { error: submitError } = await elements.submit();
    if (submitError) {
      toast({
        title: 'Payment Error',
        description: submitError.message || 'An unexpected error occurred.',
        variant: 'destructive',
      });
      setIsLoading(false);
      return;
    }

    const { error: confirmError, paymentIntent } = await stripe.confirmPayment({
      elements,
      clientSecret,
      confirmParams: {
        return_url: `${window.location.origin}/orders/${orderId}?payment_success=true`, // Redirect URL after payment
      },
      redirect: 'if_required', // Handle redirect manually if needed
    });

    if (confirmError) {
      toast({
        title: 'Payment Confirmation Failed',
        description: confirmError.message || 'An error occurred during payment confirmation.',
        variant: 'destructive',
      });
      setIsLoading(false);
      return;
    }

    if (paymentIntent && paymentIntent.status === 'succeeded') {
      try {
        // Call your backend to confirm payment status and update order
        await confirmPaymentMutation.mutateAsync({
          orderId,
          paymentIntentId: paymentIntent.id,
        });
        toast({
          title: 'Payment Successful!',
          description: 'Your order has been placed.',
          variant: 'success', // Assuming you have a 'success' variant for toast
        });
        router.push(`/orders/${orderId}?payment_success=true`);
      } catch (backendError) {
        console.error('Backend confirmation failed:', backendError);
        toast({
          title: 'Payment Processed, but Backend Update Failed',
          description: 'Please contact support with your order number. Your payment was successful.',
          variant: 'destructive',
        });
        router.push(`/orders/${orderId}?payment_success=true&backend_error=true`);
      }
    } else if (paymentIntent && paymentIntent.status === 'requires_action') {
      // Handle 3D Secure or other actions
      toast({
        title: 'Payment Requires Action',
        description: 'Please complete the required action in the new window/tab.',
        variant: 'warning',
      });
      // Stripe.js will handle the redirect for 'requires_action' if return_url is set.
    } else

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 34011_