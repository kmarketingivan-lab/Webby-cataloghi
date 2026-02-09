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