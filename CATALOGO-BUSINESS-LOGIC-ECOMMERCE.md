# CATALOGO-BUSINESS-LOGIC-ECOMMERCE

CATALOGO-BUSINESS-LOGIC-ECOMMERCE per Next.js 14 + Prisma
Espansione delle Sezioni Aggiuntive
§ PRICING ENGINE
Schema Prisma
prisma
// schema.prisma
model ProductPrice {
  id               String   @id @default(cuid())
  productId        String
  product          Product  @relation(fields: [productId], references: [id])
  basePrice        Decimal  @db.Decimal(10, 2)
  salePrice        Decimal? @db.Decimal(10, 2)
  memberPrice      Decimal? @db.Decimal(10, 2)
  currency         String   @default("EUR")
  taxRateId        String?
  taxRate          TaxRate? @relation(fields: [taxRateId], references: [id])
  bulkPricingRules BulkPricingRule[]
  priceRules       PriceRule[]
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt

  @@index([productId])
}

model PriceRule {
  id              String   @id @default(cuid())
  name            String
  description     String?
  type            RuleType @default(FIXED_AMOUNT)
  value           Decimal  @db.Decimal(10, 2)
  priority        Int      @default(0)
  isActive        Boolean  @default(true)
  startDate       DateTime?
  endDate         DateTime?
  minQuantity     Int?
  maxQuantity     Int?
  customerGroupId String?
  customerGroup   CustomerGroup? @relation(fields: [customerGroupId], references: [id])
  productPriceId  String?
  productPrice    ProductPrice? @relation(fields: [productPriceId], references: [id])
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  @@index([customerGroupId])
  @@index([productPriceId])
}

model BulkPricingRule {
  id             String   @id @default(cuid())
  productPriceId String
  productPrice   ProductPrice @relation(fields: [productPriceId], references: [id])
  minQuantity    Int
  maxQuantity    Int?
  discountType   DiscountType @default(PERCENTAGE)
  discountValue  Decimal  @db.Decimal(10, 2)
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  @@unique([productPriceId, minQuantity])
}

model TaxRate {
  id            String   @id @default(cuid())
  countryCode   String
  stateCode     String?
  postalCode    String?
  rate          Decimal  @db.Decimal(5, 4) // 0.2000 per 20%
  isDefault     Boolean  @default(false)
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  @@unique([countryCode, stateCode, postalCode])
}

model CurrencyRate {
  id           String   @id @default(cuid())
  fromCurrency String
  toCurrency   String
  rate         Decimal  @db.Decimal(10, 6)
  effectiveFrom DateTime @default(now())
  effectiveTo   DateTime?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  @@index([fromCurrency, toCurrency, effectiveFrom])
}

enum RuleType {
  PERCENTAGE
  FIXED_AMOUNT
  FIXED_PRICE
}

enum DiscountType {
  PERCENTAGE
  FIXED_AMOUNT
}
Pricing Service
typescript
// app/lib/pricing/pricing-engine.ts
import { Decimal } from '@prisma/client/runtime/library';

export interface PricingContext {
  userId?: string;
  customerGroupId?: string;
  quantity: number;
  region?: {
    countryCode: string;
    stateCode?: string;
    postalCode?: string;
  };
  currency: string;
  applyTax: boolean;
}

export interface CalculatedPrice {
  basePrice: Decimal;
  finalPrice: Decimal;
  discountAmount: Decimal;
  taxAmount: Decimal;
  taxRate: Decimal;
  currency: string;
  appliedRules: Array<{
    name: string;
    type: string;
    value: Decimal;
  }>;
}

export class PricingEngine {
  private async getProductPrice(productId: string): Promise<ProductPrice> {
    return await prisma.productPrice.findFirstOrThrow({
      where: { productId },
      include: {
        bulkPricingRules: true,
        priceRules: {
          where: {
            isActive: true,
            OR: [
              { startDate: null, endDate: null },
              {
                startDate: { lte: new Date() },
                endDate: { gte: new Date() }
              }
            ]
          },
          orderBy: { priority: 'desc' }
        },
        taxRate: true
      }
    });
  }

  private async getCurrencyRate(from: string, to: string): Promise<Decimal> {
    const rate = await prisma.currencyRate.findFirst({
      where: {
        fromCurrency: from,
        toCurrency: to,
        effectiveFrom: { lte: new Date() },
        OR: [
          { effectiveTo: null },
          { effectiveTo: { gte: new Date() } }
        ]
      },
      orderBy: { effectiveFrom: 'desc' }
    });
    
    return rate?.rate || new Decimal(1);
  }

  private async getTaxRate(context: PricingContext): Promise<TaxRate> {
    if (!context.region) {
      return await prisma.taxRate.findFirstOrThrow({
        where: { isDefault: true }
      });
    }

    const { countryCode, stateCode, postalCode } = context.region;
    
    const taxRate = await prisma.taxRate.findFirst({
      where: {
        countryCode,
        stateCode: stateCode || null,
        postalCode: postalCode || null
      }
    });

    if (!taxRate) {
      return await prisma.taxRate.findFirstOrThrow({
        where: { countryCode, stateCode: null, postalCode: null }
      });
    }

    return taxRate;
  }

  private applyBulkPricing(
    basePrice: Decimal,
    quantity: number,
    rules: BulkPricingRule[]
  ): Decimal {
    const applicableRule = rules
      .sort((a, b) => b.minQuantity - a.minQuantity)
      .find(rule => quantity >= rule.minQuantity && 
        (!rule.maxQuantity || quantity <= rule.maxQuantity));

    if (!applicableRule) return basePrice;

    switch (applicableRule.discountType) {
      case 'PERCENTAGE':
        return basePrice.mul(
          new Decimal(1).minus(applicableRule.discountValue.div(100))
        );
      case 'FIXED_AMOUNT':
        return Decimal.max(
          basePrice.minus(applicableRule.discountValue),
          new Decimal(0)
        );
      default:
        return basePrice;
    }
  }

  private applyPriceRules(
    price: Decimal,
    rules: PriceRule[],
    context: PricingContext
  ): { finalPrice: Decimal; appliedRules: Array<any> } {
    let finalPrice = price;
    const appliedRules: Array<any> = [];

    for (const rule of rules) {
      // Controlla limiti quantità
      if (rule.minQuantity && context.quantity < rule.minQuantity) continue;
      if (rule.maxQuantity && context.quantity > rule.maxQuantity) continue;

      // Controlla gruppo cliente
      if (rule.customerGroupId && 
          rule.customerGroupId !== context.customerGroupId) continue;

      switch (rule.type) {
        case 'PERCENTAGE':
          finalPrice = finalPrice.mul(
            new Decimal(1).minus(rule.value.div(100))
          );
          break;
        case 'FIXED_AMOUNT':
          finalPrice = Decimal.max(
            finalPrice.minus(rule.value),
            new Decimal(0)
          );
          break;
        case 'FIXED_PRICE':
          finalPrice = rule.value;
          break;
      }

      appliedRules.push({
        name: rule.name,
        type: rule.type,
        value: rule.value
      });
    }

    return { finalPrice, appliedRules };
  }

  public async calculatePrice(
    productId: string,
    context: PricingContext
  ): Promise<CalculatedPrice> {
    // Ottieni prezzo base
    const productPrice = await this.getProductPrice(productId);
    
    // Determina prezzo iniziale (member > sale > base)
    let basePrice = productPrice.basePrice;
    if (context.customerGroupId && productPrice.memberPrice) {
      basePrice = productPrice.memberPrice;
    } else if (productPrice.salePrice) {
      basePrice = productPrice.salePrice;
    }

    // Applica sconti quantità
    let quantityAdjustedPrice = this.applyBulkPricing(
      basePrice,
      context.quantity,
      productPrice.bulkPricingRules
    );

    // Applica regole di prezzo
    const { finalPrice: ruleAdjustedPrice, appliedRules } = 
      this.applyPriceRules(
        quantityAdjustedPrice,
        productPrice.priceRules,
        context
      );

    // Converti valuta se necessario
    let finalPrice = ruleAdjustedPrice;
    if (productPrice.currency !== context.currency) {
      const rate = await this.getCurrencyRate(
        productPrice.currency,
        context.currency
      );
      finalPrice = finalPrice.mul(rate);
    }

    // Calcola tasse
    let taxAmount = new Decimal(0);
    let taxRate = new Decimal(0);
    
    if (context.applyTax) {
      const tax = await this.getTaxRate(context);
      taxRate = tax.rate;
      taxAmount = finalPrice.mul(taxRate);
    }

    return {
      basePrice: basePrice,
      finalPrice: finalPrice.plus(taxAmount),
      discountAmount: basePrice.minus(finalPrice),
      taxAmount,
      taxRate,
      currency: context.currency,
      appliedRules
    };
  }

  public async validateMinimumOrderValue(
    cartTotal: Decimal,
    region: string
  ): Promise<{ isValid: boolean; minimum: Decimal; difference: Decimal }> {
    const config = await prisma.orderConfig.findFirst({
      where: { region }
    });

    const minimum = config?.minimumOrderValue || new Decimal(0);
    const isValid = cartTotal.greaterThanOrEqualTo(minimum);
    const difference = Decimal.max(minimum.minus(cartTotal), new Decimal(0));

    return { isValid, minimum, difference };
  }
}
Tax Calculation Service
typescript
// app/lib/pricing/tax-service.ts
export class TaxService {
  public async calculateTaxes(
    items: Array<{
      productId: string;
      price: Decimal;
      quantity: number;
    }>,
    shippingAddress: {
      countryCode: string;
      stateCode?: string;
      postalCode?: string;
    }
  ): Promise<{
    taxableAmount: Decimal;
    taxAmount: Decimal;
    breakdown: Array<{
      rate: Decimal;
      amount: Decimal;
      taxableAmount: Decimal;
    }>;
  }> {
    const taxRates = new Map<string, TaxRate>();
    const itemsByTaxRate = new Map<string, Array<{ price: Decimal; quantity: number }>>();

    // Raggruppa items per tax rate
    for (const item of items) {
      const productPrice = await prisma.productPrice.findFirst({
        where: { productId: item.productId },
        include: { taxRate: true }
      });

      let taxRate: TaxRate;
      if (productPrice?.taxRate) {
        taxRate = productPrice.taxRate;
      } else {
        taxRate = await prisma.taxRate.findFirstOrThrow({
          where: {
            countryCode: shippingAddress.countryCode,
            stateCode: shippingAddress.stateCode || null,
            postalCode: shippingAddress.postalCode || null,
            OR: [
              { postalCode: null }
            ]
          },
          orderBy: [
            { postalCode: 'desc' }, // Più specifico prima
            { stateCode: 'desc' }
          ]
        });
      }

      const taxKey = taxRate.id;
      taxRates.set(taxKey, taxRate);

      if (!itemsByTaxRate.has(taxKey)) {
        itemsByTaxRate.set(taxKey, []);
      }
      itemsByTaxRate.get(taxKey)!.push({
        price: item.price,
        quantity: item.quantity
      });
    }

    // Calcola tasse per ogni aliquota
    const breakdown: Array<{
      rate: Decimal;
      amount: Decimal;
      taxableAmount: Decimal;
    }> = [];

    let totalTaxableAmount = new Decimal(0);
    let totalTaxAmount = new Decimal(0);

    for (const [taxKey, items] of itemsByTaxRate.entries()) {
      const taxRate = taxRates.get(taxKey)!;
      const taxableAmount = items.reduce(
        (sum, item) => sum.plus(item.price.mul(item.quantity)),
        new Decimal(0)
      );
      const taxAmount = taxableAmount.mul(taxRate.rate);

      breakdown.push({
        rate: taxRate.rate,
        amount: taxAmount,
        taxableAmount
      });

      totalTaxableAmount = totalTaxableAmount.plus(taxableAmount);
      totalTaxAmount = totalTaxAmount.plus(taxAmount);
    }

    return {
      taxableAmount: totalTaxableAmount,
      taxAmount: totalTaxAmount,
      breakdown
    };
  }

  public async isTaxExempt(
    userId: string,
    countryCode: string
  ): Promise<boolean> {
    const user = await prisma.user.findUnique({
      where: { id: userId },
      include: { taxExemptions: true }
    });

    if (!user) return false;

    return user.taxExemptions.some(exemption =>
      exemption.countryCode === countryCode ||
      exemption.countryCode === '*'
    );
  }
}
§ INVENTORY MANAGEMENT
Schema Prisma
prisma
// schema.prisma
model ProductInventory {
  id               String   @id @default(cuid())
  productId        String   @unique
  product          Product  @relation(fields: [productId], references: [id])
  sku              String   @unique
  quantity         Int      @default(0)
  reserved         Int      @default(0)
  lowStockThreshold Int     @default(10)
  reorderPoint     Int      @default(20)
  reorderQuantity  Int      @default(50)
  allowBackorder   Boolean  @default(false)
  warehouseId      String?
  warehouse        Warehouse? @relation(fields: [warehouseId], references: [id])
  lastRestocked    DateTime?
  createdAt        DateTime @default(now())
  updatedAt        DateTime @updatedAt

  @@index([sku])
  @@index([warehouseId])
}

model Warehouse {
  id          String   @id @default(cuid())
  code        String   @unique
  name        String
  address     Json     // { street, city, state, postalCode, country }
  isActive    Boolean  @default(true)
  isDefault   Boolean  @default(false)
  capacities  Json?    // { maxItems: number, categories: string[] }
  inventories ProductInventory[]
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

model StockMovement {
  id          String      @id @default(cuid())
  inventoryId String
  inventory   ProductInventory @relation(fields: [inventoryId], references: [id])
  type        StockMovementType
  quantity    Int
  referenceId String?     // Order ID, Return ID, etc.
  referenceType String?   // 'order', 'return', 'adjustment'
  description String?
  previousQty Int
  newQty      Int
  userId      String?
  user        User?      @relation(fields: [userId], references: [id])
  createdAt   DateTime   @default(now())

  @@index([inventoryId])
  @@index([referenceId, referenceType])
  @@index([createdAt])
}

model StockReservation {
  id           String   @id @default(cuid())
  inventoryId  String
  inventory    ProductInventory @relation(fields: [inventoryId], references: [id])
  orderId      String?
  order        Order?   @relation(fields: [orderId], references: [id])
  cartId       String?
  quantity     Int
  expiresAt    DateTime // Quando la prenotazione scade
  status       ReservationStatus @default(ACTIVE)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  @@index([inventoryId])
  @@index([orderId])
  @@index([cartId])
  @@index([expiresAt])
  @@index([status])
}

model LowStockAlert {
  id          String   @id @default(cuid())
  inventoryId String
  inventory   ProductInventory @relation(fields: [inventoryId], references: [id])
  threshold   Int
  currentQty  Int
  notifiedAt  DateTime
  resolvedAt  DateTime?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([inventoryId])
  @@index([resolvedAt])
}

enum StockMovementType {
  INBOUND
  OUTBOUND
  ADJUSTMENT
  RESERVATION
  RELEASE
}

enum ReservationStatus {
  ACTIVE
  CONFIRMED
  EXPIRED
  CANCELLED
  RELEASED
}
Inventory Service
typescript
// app/lib/inventory/inventory-service.ts
import { EventEmitter } from 'events';

export class InventoryService extends EventEmitter {
  private readonly RESERVATION_TIMEOUT = 30 * 60 * 1000; // 30 minuti

  public async checkStock(
    productId: string,
    quantity: number,
    warehouseId?: string
  ): Promise<{
    available: boolean;
    availableQty: number;
    needsBackorder: boolean;
    warehouseId?: string;
  }> {
    const whereClause: any = { productId };
    if (warehouseId) {
      whereClause.warehouseId = warehouseId;
    } else {
      whereClause.warehouse = { isDefault: true };
    }

    const inventory = await prisma.productInventory.findFirst({
      where: whereClause,
      include: {
        warehouse: true
      }
    });

    if (!inventory) {
      throw new Error(`Inventory not found for product ${productId}`);
    }

    const availableQty = inventory.quantity - inventory.reserved;
    const available = availableQty >= quantity;
    const needsBackorder = !available && inventory.allowBackorder;

    return {
      available,
      availableQty,
      needsBackorder,
      warehouseId: inventory.warehouseId || undefined
    };
  }

  public async reserveStock(
    items: Array<{
      productId: string;
      quantity: number;
      warehouseId?: string;
    }>,
    cartId: string,
    expiresInMs: number = this.RESERVATION_TIMEOUT
  ): Promise<StockReservation[]> {
    const reservations: StockReservation[] = [];

    for (const item of items) {
      const inventory = await prisma.productInventory.findFirst({
        where: {
          productId: item.productId,
          warehouseId: item.warehouseId || undefined
        }
      });

      if (!inventory) {
        throw new Error(`Inventory not found for product ${item.productId}`);
      }

      const available = inventory.quantity - inventory.reserved;
      if (available < item.quantity && !inventory.allowBackorder) {
        throw new Error(`Insufficient stock for product ${item.productId}`);
      }

      // Crea prenotazione
      const reservation = await prisma.stockReservation.create({
        data: {
          inventoryId: inventory.id,
          cartId,
          quantity: item.quantity,
          expiresAt: new Date(Date.now() + expiresInMs),
          status: 'ACTIVE'
        }
      });

      // Aggiorna quantità riservata
      await prisma.productInventory.update({
        where: { id: inventory.id },
        data: {
          reserved: { increment: item.quantity }
        }
      });

      // Registra movimento
      await prisma.stockMovement.create({
        data: {
          inventoryId: inventory.id,
          type: 'RESERVATION',
          quantity: item.quantity,
          previousQty: inventory.reserved,
          newQty: inventory.reserved + item.quantity,
          description: `Reserved for cart ${cartId}`
        }
      });

      reservations.push(reservation);
    }

    return reservations;
  }

  public async confirmReservation(
    reservationId: string,
    orderId: string
  ): Promise<void> {
    const reservation = await prisma.stockReservation.findUniqueOrThrow({
      where: { id: reservationId },
      include: { inventory: true }
    });

    if (reservation.status !== 'ACTIVE') {
      throw new Error(`Reservation ${reservationId} is not active`);
    }

    if (reservation.expiresAt < new Date()) {
      await this.releaseReservation(reservationId);
      throw new Error(`Reservation ${reservationId} has expired`);
    }

    // Conferma prenotazione
    await prisma.$transaction([
      prisma.stockReservation.update({
        where: { id: reservationId },
        data: {
          orderId,
          status: 'CONFIRMED'
        }
      }),
      prisma.productInventory.update({
        where: { id: reservation.inventoryId },
        data: {
          quantity: { decrement: reservation.quantity },
          reserved: { decrement: reservation.quantity }
        }
      }),
      prisma.stockMovement.create({
        data: {
          inventoryId: reservation.inventoryId,
          type: 'OUTBOUND',
          quantity: reservation.quantity,
          referenceId: orderId,
          referenceType: 'order',
          previousQty: reservation.inventory.quantity,
          newQty: reservation.inventory.quantity - reservation.quantity,
          description: `Confirmed for order ${orderId}`
        }
      })
    ]);

    // Controlla stock basso
    await this.checkLowStock(reservation.inventoryId);
  }

  public async releaseReservation(reservationId: string): Promise<void> {
    const reservation = await prisma.stockReservation.findUniqueOrThrow({
      where: { id: reservationId },
      include: { inventory: true }
    });

    await prisma.$transaction([
      prisma.stockReservation.update({
        where: { id: reservationId },
        data: { status: 'RELEASED' }
      }),
      prisma.productInventory.update({
        where: { id: reservation.inventoryId },
        data: { reserved: { decrement: reservation.quantity } }
      }),
      prisma.stockMovement.create({
        data: {
          inventoryId: reservation.inventoryId,
          type: 'RELEASE',
          quantity: reservation.quantity,
          previousQty: reservation.inventory.reserved,
          newQty: reservation.inventory.reserved - reservation.quantity,
          description: `Released reservation ${reservationId}`
        }
      })
    ]);
  }

  public async cleanupExpiredReservations(): Promise<number> {
    const expiredReservations = await prisma.stockReservation.findMany({
      where: {
        status: 'ACTIVE',
        expiresAt: { lt: new Date() }
      },
      include: { inventory: true }
    });

    let releasedCount = 0;
    for (const reservation of expiredReservations) {
      try {
        await this.releaseReservation(reservation.id);
        releasedCount++;
      } catch (error) {
        console.error(`Failed to release reservation ${reservation.id}:`, error);
      }
    }

    return releasedCount;
  }

  private async checkLowStock(inventoryId: string): Promise<void> {
    const inventory = await prisma.productInventory.findUniqueOrThrow({
      where: { id: inventoryId }
    });

    const available = inventory.quantity - inventory.reserved;
    
    if (available <= inventory.lowStockThreshold) {
      const existingAlert = await prisma.lowStockAlert.findFirst({
        where: {
          inventoryId,
          resolvedAt: null
        }
      });

      if (!existingAlert) {
        const alert = await prisma.lowStockAlert.create({
          data: {
            inventoryId,
            threshold: inventory.lowStockThreshold,
            currentQty: available,
            notifiedAt: new Date()
          }
        });

        // Emetti evento per notifica
        this.emit('lowStock', {
          productId: inventory.productId,
          sku: inventory.sku,
          available,
          threshold: inventory.lowStockThreshold,
          alertId: alert.id
        });
      }
    }
  }

  public async adjustStock(
    productId: string,
    quantity: number,
    type: 'INBOUND' | 'OUTBOUND' | 'ADJUSTMENT',
    reason: string,
    reference?: { id: string; type: string }
  ): Promise<void> {
    const inventory = await prisma.productInventory.findFirstOrThrow({
      where: { productId }
    });

    const newQuantity = type === 'OUTBOUND' 
      ? inventory.quantity - quantity
      : inventory.quantity + quantity;

    await prisma.$transaction([
      prisma.productInventory.update({
        where: { id: inventory.id },
        data: { quantity: newQuantity }
      }),
      prisma.stockMovement.create({
        data: {
          inventoryId: inventory.id,
          type,
          quantity,
          referenceId: reference?.id,
          referenceType: reference?.type,
          previousQty: inventory.quantity,
          newQty: newQuantity,
          description: reason
        }
      })
    ]);

    await this.checkLowStock(inventory.id);
  }

  public async getMultiWarehouseAvailability(
    productId: string,
    quantity: number
  ): Promise<Array<{
    warehouseId: string;
    warehouseCode: string;
    available: number;
    canFulfill: boolean;
    shippingEstimate: number; // giorni
  }>> {
    const inventories = await prisma.productInventory.findMany({
      where: { productId },
      include: { warehouse: true }
    });

    const results = [];
    for (const inv of inventories) {
      const available = inv.quantity - inv.reserved;
      const canFulfill = available >= quantity || inv.allowBackorder;
      
      results.push({
        warehouseId: inv.warehouseId!,
        warehouseCode: inv.warehouse!.code,
        available,
        canFulfill,
        shippingEstimate: this.calculateShippingEstimate(inv.warehouse!)
      });
    }

    return results.sort((a, b) => a.shippingEstimate - b.shippingEstimate);
  }

  private calculateShippingEstimate(warehouse: Warehouse): number {
    // Logica per calcolare stima spedizione in base a zona
    const address = warehouse.address as any;
    // Implementazione semplificata
    return address.country === 'IT' ? 2 : 5;
  }

  public async handleBackorder(
    productId: string,
    quantity: number,
    orderId: string
  ): Promise<{
    backorderId: string;
    estimatedDate: Date;
  }> {
    const inventory = await prisma.productInventory.findFirstOrThrow({
      where: { productId, allowBackorder: true }
    });

    const backorder = await prisma.backorder.create({
      data: {
        productId,
        orderId,
        quantity,
        status: 'PENDING',
        estimatedDate: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 giorni
      }
    });

    this.emit('backorderCreated', {
      backorderId: backorder.id,
      productId,
      quantity,
      orderId,
      estimatedDate: backorder.estimatedDate
    });

    return {
      backorderId: backorder.id,
      estimatedDate: backorder.estimatedDate
    };
  }
}
Scheduler per Cleanup Reservations
typescript
// app/lib/inventory/scheduler.ts
import { CronJob } from 'cron';

export class InventoryScheduler {
  private inventoryService: InventoryService;

  constructor(inventoryService: InventoryService) {
    this.inventoryService = inventoryService;
  }

  public start(): void {
    // Ogni 5 minuti, pulisci le prenotazioni scadute
    const cleanupJob = new CronJob('*/5 * * * *', async () => {
      try {
        const released = await this.inventoryService.cleanupExpiredReservations();
        if (released > 0) {
          console.log(`Released ${released} expired reservations`);
        }
      } catch (error) {
        console.error('Failed to cleanup expired reservations:', error);
      }
    });

    // Ogni giorno a mezzanotte, controlla stock basso per tutti i prodotti
    const lowStockCheckJob = new CronJob('0 0 * * *', async () => {
      try {
        const lowStockItems = await prisma.productInventory.findMany({
          where: {
            quantity: {
              lte: prisma.productInventory.fields.lowStockThreshold
            }
          },
          include: { product: true }
        });

        for (const item of lowStockItems) {
          this.inventoryService.emit('lowStock', {
            productId: item.productId,
            productName: item.product.name,
            available: item.quantity - item.reserved,
            threshold: item.lowStockThreshold
          });
        }
      } catch (error) {
        console.error('Failed to check low stock:', error);
      }
    });

    cleanupJob.start();
    lowStockCheckJob.start();
  }
}
§ PROMOTIONS ENGINE
Schema Prisma
prisma
// schema.prisma
model Promotion {
  id                String   @id @default(cuid())
  code              String?  @unique
  name              String
  description       String?
  type              PromotionType
  value             Decimal  @db.Decimal(10, 2)
  valueType         ValueType @default(PERCENTAGE)
  isActive          Boolean  @default(true)
  startDate         DateTime
  endDate           DateTime
  usageLimit        Int?     // Limite totale
  perUserLimit      Int?     // Limite per utente
  usedCount         Int      @default(0)
  minCartValue      Decimal? @db.Decimal(10, 2)
  minQuantity       Int?
  maxDiscountAmount Decimal? @db.Decimal(10, 2)
  isStackable       Boolean  @default(false)
  isExclusive       Boolean  @default(false)
  priority          Int      @default(0)
  
  // Restrizioni
  restrictedToUserIds String[] // JSON array di user IDs
  restrictedToEmailDomains String[] // JSON array di domini
  
  // Applicabilità
  applyToAllProducts Boolean @default(true)
  applyToCategories  String[] // JSON array di category IDs
  applyToProducts    String[] // JSON array di product IDs
  excludeCategories  String[] // JSON array di category IDs
  excludeProducts    String[] // JSON array di product IDs
  
  // Shipping
  freeShipping       Boolean @default(false)
  freeShippingMin    Decimal? @db.Decimal(10, 2)
  
  // BOGO
  bogoBuyProductId   String?
  bogoBuyQuantity    Int?
  bogoGetProductId   String?
  bogoGetQuantity    Int?
  bogoDiscountType   DiscountType? @default(PERCENTAGE)
  bogoDiscountValue  Decimal? @db.Decimal(10, 2)
  
  // Usage Tracking
  usageHistory       PromotionUsage[]
  createdAt          DateTime @default(now())
  updatedAt          DateTime @updatedAt

  @@index([code])
  @@index([startDate, endDate])
  @@index([isActive])
}

model PromotionUsage {
  id           String   @id @default(cuid())
  promotionId  String
  promotion    Promotion @relation(fields: [promotionId], references: [id])
  userId       String?
  user         User?    @relation(fields: [userId], references: [id])
  orderId      String?
  order        Order?   @relation(fields: [orderId], references: [id])
  cartId       String?
  discountAmount Decimal @db.Decimal(10, 2)
  usedAt       DateTime @default(now())
  
  @@index([promotionId])
  @@index([userId])
  @@index([orderId])
  @@index([cartId])
}

enum PromotionType {
  COUPON
  AUTOMATIC
  BOGO
  LOYALTY
  SEASONAL
}

enum ValueType {
  PERCENTAGE
  FIXED_AMOUNT
  FIXED_PRICE
  FREE_SHIPPING
}
Promotions Service
typescript
// app/lib/promotions/promotion-engine.ts
export interface CartContext {
  userId?: string;
  userEmail?: string;
  items: Array<{
    productId: string;
    categoryId: string;
    price: Decimal;
    quantity: number;
  }>;
  subtotal: Decimal;
  shippingCost: Decimal;
  shippingMethod?: string;
}

export interface AppliedPromotion {
  promotionId: string;
  code?: string;
  name: string;
  type: PromotionType;
  discountAmount: Decimal;
  freeShipping?: boolean;
  bogoItems?: Array<{
    buyProductId: string;
    getProductId: string;
    discount: Decimal;
  }>;
}

export class PromotionEngine {
  public async validateAndApplyPromotion(
    code: string,
    context: CartContext
  ): Promise<{
    valid: boolean;
    promotion?: AppliedPromotion;
    error?: string;
    warnings?: string[];
  }> {
    const promotion = await prisma.promotion.findUnique({
      where: { code },
      include: {
        usageHistory: {
          where: { userId: context.userId || undefined },
          orderBy: { usedAt: 'desc' }
        }
      }
    });

    if (!promotion) {
      return { valid: false, error: 'Promotion code not found' };
    }

    // Validazioni
    const validation = await this.validatePromotion(promotion, context);
    if (!validation.valid) {
      return { valid: false, error: validation.error };
    }

    // Applica promozione
    const appliedPromotion = await this.applyPromotion(promotion, context);

    return {
      valid: true,
      promotion: appliedPromotion,
      warnings: validation.warnings
    };
  }

  private async validatePromotion(
    promotion: Promotion,
    context: CartContext
  ): Promise<{
    valid: boolean;
    error?: string;
    warnings?: string[];
  }> {
    const warnings: string[] = [];

    // Data validity
    const now = new Date();
    if (promotion.startDate > now) {
      return { valid: false, error: 'Promotion not started yet' };
    }
    if (promotion.endDate < now) {
      return { valid: false, error: 'Promotion has expired' };
    }

    // Active status
    if (!promotion.isActive) {
      return { valid: false, error: 'Promotion is not active' };
    }

    // Usage limits
    if (promotion.usageLimit && promotion.usedCount >= promotion.usageLimit) {
      return { valid: false, error: 'Promotion usage limit reached' };
    }

    // Per user limit
    if (context.userId && promotion.perUserLimit) {
      const userUsageCount = promotion.usageHistory.filter(
        usage => usage.userId === context.userId
      ).length;
      if (userUsageCount >= promotion.perUserLimit) {
        return { valid: false, error: 'You have already used this promotion' };
      }
    }

    // User restrictions
    if (promotion.restrictedToUserIds.length > 0 && context.userId) {
      if (!promotion.restrictedToUserIds.includes(context.userId)) {
        return { valid: false, error: 'Promotion not available for your account' };
      }
    }

    // Email domain restrictions
    if (promotion.restrictedToEmailDomains.length > 0 && context.userEmail) {
      const domain = context.userEmail.split('@')[1];
      if (!promotion.restrictedToEmailDomains.includes(domain)) {
        return { valid: false, error: 'Promotion not available for your email domain' };
      }
    }

    // Minimum cart value
    if (promotion.minCartValue && context.subtotal.lessThan(promotion.minCartValue)) {
      const difference = promotion.minCartValue.minus(context.subtotal);
      return { 
        valid: false, 
        error: `Add

## § ADVANCED PATTERNS: BUSINESS LOGIC ECOMMERCE

### Server Actions con Validazione

```typescript
// app/actions/business-logic-ecommerce.ts
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


### BUSINESS LOGIC ECOMMERCE - Utility Helper #490

```typescript
// lib/utils/business-logic-ecommerce-helper-490.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #536

```typescript
// lib/utils/business-logic-ecommerce-helper-536.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #884

```typescript
// lib/utils/business-logic-ecommerce-helper-884.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #2

```typescript
// lib/utils/business-logic-ecommerce-helper-2.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #978

```typescript
// lib/utils/business-logic-ecommerce-helper-978.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #916

```typescript
// lib/utils/business-logic-ecommerce-helper-916.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #816

```typescript
// lib/utils/business-logic-ecommerce-helper-816.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #658

```typescript
// lib/utils/business-logic-ecommerce-helper-658.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #315

```typescript
// lib/utils/business-logic-ecommerce-helper-315.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #383

```typescript
// lib/utils/business-logic-ecommerce-helper-383.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #318

```typescript
// lib/utils/business-logic-ecommerce-helper-318.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #200

```typescript
// lib/utils/business-logic-ecommerce-helper-200.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #62

```typescript
// lib/utils/business-logic-ecommerce-helper-62.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #72

```typescript
// lib/utils/business-logic-ecommerce-helper-72.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #854

```typescript
// lib/utils/business-logic-ecommerce-helper-854.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #828

```typescript
// lib/utils/business-logic-ecommerce-helper-828.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<BUSINESSLOGICECOMMERCEConfig> = {}) {
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

  getConfig(): Readonly<BUSINESSLOGICECOMMERCEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### BUSINESS LOGIC ECOMMERCE - Utility Helper #571

```typescript
// lib/utils/business-logic-ecommerce-helper-571.ts
import { z } from "zod";

interface BUSINESSLOGICECOMMERCEConfig {
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

export class BUSINESSLOGICECOMMERCEProcessor<TInput, TOutput> {
  private config: BUSINESSLOGICECOMMERCEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => 