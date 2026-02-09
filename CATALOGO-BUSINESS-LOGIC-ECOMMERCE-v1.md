# CATALOGO-BUSCHERES-LOGIC-ECOMMERCE-v1
§ DOCUMENTAZIONE TECNICA COMPLETA PER PIATTAFORMA E-COMMERCE

---

§ §1. E-COMMERCE ARCHITECTURE OVERVIEW

§ 1.1 DIAGRAMMA ARCHITETTURA ASCII

┌─────────────────────────────────────────────────────────────────────────┐
│                           CLIENT (Next.js 14)                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐   │
│  │   App Router │  │  Components │  │    Hooks    │  │   Zustand    │   │
│  │   (Server)   │  │  (Client)   │  │  (React)    │  │   (State)    │   │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘   │
│         │                 │                 │                 │          │
│         └─────────────────────────────────────────────────────┘          │
│                              │ tRPC / Fetch                               │
└──────────────────────────────┼───────────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────────┐
│                         API LAYER (Next.js)                              │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    tRPC Router (/api/trpc)                       │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │   │
│  │  │ Products │  │   Cart   │  │ Checkout │  │  Orders  │        │   │
│  │  │  Router  │  │  Router  │  │  Router  │  │  Router  │        │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │   │
│  │                                                                 │   │
│  │                    API Routes (/api/*)                          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐        │   │
│  │  │ Stripe   │  │ Webhooks │  │  Upload  │  │  Health  │        │   │
│  │  │ Webhook  │  │          │  │          │  │  Check   │        │   │
│  │  └──────────┘  └──────────┘  └──────────┘  └──────────┘        │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                               │ Service Layer                            │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │  ProductService  CartService  CheckoutService  OrderService     │   │
│  │  DiscountService ShippingService TaxService InventoryService    │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                               │ Prisma Client                            │
└──────────────────────────────┬───────────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────────┐
│                         DATABASE LAYER                                   │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    PostgreSQL (Vercel/Neon)                      │   │
│  │  • Products, Variants, Categories                               │   │
│  │  • Inventory, Stock Reservations                                │   │
│  │  • Carts, Orders, Order Items                                   │   │
│  │  • Users, Addresses, Payments                                   │   │
│  │  • Reviews, Wishlists                                           │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                    Redis (Upstash)                               │   │
│  │  • Session storage                                              │   │
│  │  • Rate limiting                                                │   │
│  │  • Cache (products, categories)                                 │   │
│  │  • Real-time inventory                                          │   │
│  └──────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────┬───────────────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────────────┐
│                      EXTERNAL SERVICES                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │    Stripe   │  │   Shippo    │  │  TaxJar     │  │   Resend    │    │
│  │  Payments   │  │  Shipping   │  │   Taxes     │  │   Emails    │    │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘    │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                      │
│  │  Cloudinary │  │   Algolia   │  │   Sentry    │                      │
│  │    Images   │  │    Search   │  │  Monitoring │                      │
│  └─────────────┘  └─────────────┘  └─────────────┘                      │
└─────────────────────────────────────────────────────────────────────────┘

§ 1.2 TABELLA DECISIONALE: MONOLITH VS MICROSERVICES

| Scala | Architettura | Motivazione | Raccomandazione |
|-------|-------------|-------------|-----------------|
| < 100 ordini/giorno | **Monolith (Next.js Full-Stack)** | Sviluppo veloce, deployment semplice, debugging facile, costi bassi | ✅ **RACCOMANDATO** per startup e piccoli e-commerce |
| 100-1000 ordini/giorno | **Modular Monolith** | Separazione logica mantenendo deployment unico, scaling verticale | ✅ **OTTIMO** per crescita iniziale |
| 1000-10k ordini/giorno | **Service-Oriented Architecture** | Servizi separati per domini critici (payments, inventory, search) | ⚠️ **CONSIDERARE** quando team > 10 devs |
| > 10k ordini/giorno | **Microservices** | Scaling indipendente, fault isolation, deploy frequenti | ❌ **SOVRAINGEGNERIZZATO** per la maggior parte |

§ 1.3 TABELLA: BUILD VS BUY PER COMPONENTI

| Componente | Build | Buy | Raccomandazione |
|------------|-------|-----|-----------------|
| **Payment Processing** | Implementare PCI compliance, fraud detection, multi-currency | Stripe, PayPal, Adyen | ✅ **BUY** (Stripe) - Complessità e rischio altissimi |
| **Shipping & Fulfillment** | Integrare con API carrier singoli, gestire tracking | Shippo, Easyship, ShipStation | ✅ **BUY** (Shippo) - Multi-carrier, rates in real-time |
| **Tax Calculation** | Database tax rates, logica per stati/regioni | TaxJar, Avalara, Stripe Tax | ✅ **BUY** (TaxJar o Stripe Tax) - Aggiornamenti automatici |
| **Email Marketing** | SMTP server, template system, analytics | Klaviyo, Mailchimp, Resend | ✅ **BUY** (Resend per transazionali + Klaviyo per marketing) |
| **Product Search** | Full-text search PostgreSQL, ranking semplice | Algolia, Elasticsearch | ⚠️ **BUILD iniziale, BUY dopo** (Algolia quando scaling) |
| **Image Optimization** | Serverless functions, CDN personalizzato | Cloudinary, Imgix, Next.js Image | ✅ **BUY** (Cloudinary) - Formati moderni, ottimizzazione automatica |
| **Inventory Management** | Database schema, API per stock levels | TradeGecko, Linnworks | ✅ **BUILD** - Core business logic, bisogno di controllo completo |
| **Reviews & Ratings** | Database semplice, moderation UI | Yotpo, Trustpilot | ✅ **BUILD iniziale** - Poi valutare se bisogno di social proof |
| **Wishlist/Saved Items** | Database relazione user-product | ❌ | ✅ **BUILD** - Semplicissimo, costo basso |
| **Analytics & Reporting** | Query SQL custom, dashboard simple | Google Analytics, Mixpanel, Metabase | ⚠️ **HYBRID** (GA4 per web, BUILD per business metrics) |
| **CDN & Media Storage** | AWS S3 + CloudFront, gestione manuale | Vercel Blob, Cloudflare R2 | ✅ **BUY** (Vercel Blob) - Integrazione nativa con Next.js |

---

§ §2. PRODUCT CATALOG SYSTEM

§ 2.1 DATABASE SCHEMA COMPLETO (PRISMA)

prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ============ PRODUCT CATALOG ============
model Product {
  id             String         @id @default(cuid())
  sku            String         @unique
  name           String
  slug           String         @unique
  description    String?
  shortDescription String?
  
  // Pricing
  basePrice      Decimal        @db.Decimal(10, 2)
  compareAtPrice Decimal?       @db.Decimal(10, 2)
  costPrice      Decimal?       @db.Decimal(10, 2)
  
  // Categorization
  categoryId     String?
  category       Category?      @relation(fields: [categoryId], references: [id])
  brandId        String?
  brand          Brand?         @relation(fields: [brandId], references: [id])
  
  // Status & Visibility
  status         ProductStatus  @default(DRAFT)
  isFeatured     Boolean        @default(false)
  isAvailable    Boolean        @default(true)
  publishedAt    DateTime?
  
  // SEO & Metadata
  seoTitle       String?
  seoDescription String?
  keywords       String[]
  
  // Inventory
  manageInventory Boolean       @default(true)
  trackInventory  Boolean       @default(true)
  allowBackorder  Boolean       @default(false)
  lowStockThreshold Int         @default(5)
  
  // Shipping
  weight         Decimal?       @db.Decimal(10, 2) // in kg
  width          Decimal?       @db.Decimal(10, 2) // in cm
  height         Decimal?       @db.Decimal(10, 2) // in cm
  depth          Decimal?       @db.Decimal(10, 2) // in cm
  
  // Type
  type           ProductType    @default(SIMPLE)
  isDigital      Boolean        @default(false)
  isSubscription Boolean        @default(false)
  
  // Relations
  variants       ProductVariant[]
  options        ProductOption[]
  attributes     ProductAttribute[]
  images         ProductImage[]
  reviews        Review[]
  
  // Timestamps
  createdAt      DateTime       @default(now())
  updatedAt      DateTime       @updatedAt
  deletedAt      DateTime?
  
  @@index([slug])
  @@index([categoryId])
  @@index([brandId])
  @@index([sku])
  @@index([status])
}

enum ProductStatus {
  DRAFT
  ACTIVE
  ARCHIVED
  OUT_OF_STOCK
  DISCONTINUED
}

enum ProductType {
  SIMPLE
  VARIABLE
  BUNDLE
  DIGITAL
  SUBSCRIPTION
}

model ProductVariant {
  id            String         @id @default(cuid())
  productId     String
  product       Product        @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  // Identification
  sku           String         @unique
  name          String
  optionValues  Json           // { size: "M", color: "Red" }
  
  // Pricing
  price         Decimal        @db.Decimal(10, 2)
  compareAtPrice Decimal?      @db.Decimal(10, 2)
  costPrice     Decimal?       @db.Decimal(10, 2)
  
  // Inventory
  quantity      Int            @default(0)
  trackQuantity Boolean        @default(true)
  
  // Shipping
  weight        Decimal?       @db.Decimal(10, 2)
  width         Decimal?       @db.Decimal(10, 2)
  height        Decimal?       @db.Decimal(10, 2)
  depth         Decimal?       @db.Decimal(10, 2)
  
  // Status
  isAvailable   Boolean        @default(true)
  
  // Relations
  inventory     Inventory[]
  cartItems     CartItem[]
  orderItems    OrderItem[]
  
  // Timestamps
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  
  @@index([productId])
  @@index([sku])
  @@unique([productId, optionValues])
}

model ProductOption {
  id          String   @id @default(cuid())
  productId   String
  product     Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  name        String   // "Size", "Color", "Material"
  values      String[] // ["S", "M", "L", "XL"]
  position    Int      @default(0)
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([productId])
}

model ProductAttribute {
  id          String   @id @default(cuid())
  productId   String
  product     Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  name        String   // "Material", "Care Instructions", "Origin"
  value       String   // "100% Cotton", "Machine wash cold", "Made in Italy"
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([productId])
}

model ProductImage {
  id          String   @id @default(cuid())
  productId   String
  product     Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  url         String
  altText     String?
  position    Int      @default(0)
  
  // Cloudinary/Imgix specific
  publicId    String?
  width       Int?
  height      Int?
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([productId])
}

// ============ CATEGORIES & BRANDS ============
model Category {
  id            String     @id @default(cuid())
  name          String
  slug          String     @unique
  description   String?
  
  // Hierarchy
  parentId      String?
  parent        Category?  @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children      Category[] @relation("CategoryHierarchy")
  
  // SEO
  seoTitle      String?
  seoDescription String?
  
  // Media
  imageUrl      String?
  
  // Relations
  products      Product[]
  
  // Timestamps
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  
  @@index([slug])
  @@index([parentId])
}

model Brand {
  id            String     @id @default(cuid())
  name          String
  slug          String     @unique
  description   String?
  logoUrl       String?
  websiteUrl    String?
  
  // SEO
  seoTitle      String?
  seoDescription String?
  
  // Relations
  products      Product[]
  
  // Timestamps
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  
  @@index([slug])
}

// ============ INVENTORY MANAGEMENT ============
model Inventory {
  id            String     @id @default(cuid())
  variantId     String
  variant       ProductVariant @relation(fields: [variantId], references: [id], onDelete: Cascade)
  warehouseId   String?
  warehouse     Warehouse? @relation(fields: [warehouseId], references: [id])
  
  quantity      Int        @default(0)
  reserved      Int        @default(0)
  
  // Calculated
  available     Int        @default(0)
  
  // Timestamps
  lastUpdatedAt DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  
  @@index([variantId])
  @@index([warehouseId])
  @@unique([variantId, warehouseId])
}

model InventoryMovement {
  id            String     @id @default(cuid())
  variantId     String
  variant       ProductVariant @relation(fields: [variantId], references: [id])
  warehouseId   String?
  warehouse     Warehouse? @relation(fields: [warehouseId], references: [id])
  
  type          InventoryMovementType
  quantity      Int
  previousQty   Int
  newQty        Int
  
  // Reference
  referenceType String?    // "order", "adjustment", "transfer"
  referenceId   String?    // orderId, adjustmentId, transferId
  
  reason        String?
  notes         String?
  
  createdById   String?
  createdBy     User?      @relation(fields: [createdById], references: [id])
  
  createdAt     DateTime   @default(now())
  
  @@index([variantId])
  @@index([createdAt])
  @@index([referenceType, referenceId])
}

enum InventoryMovementType {
  STOCK_IN
  STOCK_OUT
  RESERVATION
  RELEASE
  ADJUSTMENT
  TRANSFER_IN
  TRANSFER_OUT
}

model Warehouse {
  id            String     @id @default(cuid())
  name          String
  code          String     @unique
  address       Json?      // Address object
  isActive      Boolean    @default(true)
  isDefault     Boolean    @default(false)
  
  // Contact
  contactName   String?
  contactEmail  String?
  contactPhone  String?
  
  // Relations
  inventory     Inventory[]
  
  // Timestamps
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
}

model StockReservation {
  id            String     @id @default(cuid())
  variantId     String
  variant       ProductVariant @relation(fields: [variantId], references: [id])
  warehouseId   String?
  warehouse     Warehouse? @relation(fields: [warehouseId], references: [id])
  
  quantity      Int
  expiresAt     DateTime   // Auto-release after expiration
  
  // Reference
  cartId        String?
  cart          Cart?      @relation(fields: [cartId], references: [id])
  orderId       String?
  order         Order?     @relation(fields: [orderId], references: [id])
  
  // Status
  status        StockReservationStatus @default(ACTIVE)
  
  createdAt     DateTime   @default(now())
  updatedAt     DateTime   @updatedAt
  
  @@index([variantId])
  @@index([expiresAt])
  @@index([status])
}

enum StockReservationStatus {
  ACTIVE
  CONFIRMED
  RELEASED
  EXPIRED
}

// ============ PRICE HISTORY ============
model PriceHistory {
  id            String     @id @default(cuid())
  productId     String?
  product       Product?   @relation(fields: [productId], references: [id])
  variantId     String?
  variant       ProductVariant? @relation(fields: [variantId], references: [id])
  
  oldPrice      Decimal?   @db.Decimal(10, 2)
  newPrice      Decimal    @db.Decimal(10, 2)
  changeType    PriceChangeType
  reason        String?
  
  changedById   String?
  changedBy     User?      @relation(fields: [changedById], references: [id])
  
  effectiveFrom DateTime   @default(now())
  effectiveTo   DateTime?
  
  createdAt     DateTime   @default(now())
  
  @@index([productId])
  @@index([variantId])
  @@index([effectiveFrom])
}

enum PriceChangeType {
  REGULAR_UPDATE
  DISCOUNT
  SEASONAL
  PROMOTION
  CLEARANCE
}

§ 2.2 PRODUCT TYPES COMPARISON TABLE

| Type | Use Case | Schema Approach | Example | Complexità |
|------|----------|----------------|---------|------------|
| **Simple** | Single product, no options | Solo `Product` con campi base | Libro, utensile da cucina | Bassa |
| **Variable** | Multiple options (size, color) | `Product` + `ProductVariant` + `ProductOption` | Maglietta con taglie/colori | Media |
| **Bundle** | Multiple products sold together | `Product` (type=BUNDLE) + `BundleItem` relation | Kit per barba (rasoio + schiuma) | Media-Alta |
| **Digital** | Downloadable files, no shipping | `Product` (isDigital=true) + `DigitalAsset` | E-book, PDF, software | Bassa |
| **Subscription** | Recurring payments | `Product` (isSubscription=true) + `SubscriptionPlan` | Box mensile, software SaaS | Alta |
| **Configurable** | Customizable (engraving, etc.) | `Product` + `CustomOption` + `CustomValue` | Tazza personalizzata, anello con incisione | Alta |
| **Composite** | Build-your-own (PC, furniture) | `Product` + `Component` + `Configuration` | PC custom, mobile componibile | Molto Alta |

§ 2.3 PRODUCT SERVICE (TYPESCRIPT COMPLETO)

typescript
// lib/services/product-service.ts
import { PrismaClient, Product, ProductStatus, ProductType } from '@prisma/client';
import { z } from 'zod';
import { NotFoundError, ValidationError, BusinessError } from '@/lib/errors';
import { generateSlug } from '@/lib/utils/slug';
import { redis } from '@/lib/redis';

const prisma = new PrismaClient();

// ============ VALIDATION SCHEMAS ============
const ProductOptionSchema = z.object({
  name: z.string().min(1).max(50),
  values: z.array(z.string().min(1)).min(1),
  position: z.number().int().min(0).optional(),
});

const ProductAttributeSchema = z.object({
  name: z.string().min(1).max(100),
  value: z.string().min(1),
});

const ProductImageSchema = z.object({
  url: z.string().url(),
  altText: z.string().max(125).optional(),
  position: z.number().int().min(0).optional(),
  publicId: z.string().optional(),
  width: z.number().int().positive().optional(),
  height: z.number().int().positive().optional(),
});

const ProductVariantSchema = z.object({
  sku: z.string().min(1).max(100),
  optionValues: z.record(z.string(), z.any()),
  price: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)),
  compareAtPrice: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)).optional(),
  costPrice: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)).optional(),
  quantity: z.number().int().min(0).optional(),
  weight: z.number().positive().optional(),
  width: z.number().positive().optional(),
  height: z.number().positive().optional(),
  depth: z.number().positive().optional(),
  isAvailable: z.boolean().optional(),
});

const CreateProductSchema = z.object({
  // Basic info
  name: z.string().min(1).max(200),
  description: z.string().max(5000).optional(),
  shortDescription: z.string().max(500).optional(),
  
  // Pricing
  basePrice: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)),
  compareAtPrice: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)).optional(),
  costPrice: z.number().positive().or(z.string().regex(/^\d+(\.\d{1,2})?$/)).optional(),
  
  // Categorization
  categoryId: z.string().cuid().optional(),
  brandId: z.string().cuid().optional(),
  
  // Inventory
  manageInventory: z.boolean().optional(),
  trackInventory: z.boolean().optional(),
  allowBackorder: z.boolean().optional(),
  lowStockThreshold: z.number().int().min(0).optional(),
  
  // Shipping
  weight: z.number().positive().optional(),
  width: z.number().positive().optional(),
  height: z.number().positive().optional(),
  depth: z.number().positive().optional(),
  
  // Type & Status
  type: z.nativeEnum(ProductType).optional(),
  isDigital: z.boolean().optional(),
  isSubscription: z.boolean().optional(),
  status: z.nativeEnum(ProductStatus).optional(),
  isFeatured: z.boolean().optional(),
  
  // SEO
  seoTitle: z.string().max(60).optional(),
  seoDescription: z.string().max(160).optional(),
  keywords: z.array(z.string()).optional(),
  
  // Relations
  options: z.array(ProductOptionSchema).optional(),
  attributes: z.array(ProductAttributeSchema).optional(),
  images: z.array(ProductImageSchema).optional(),
  variants: z.array(ProductVariantSchema).optional(),
  
  // SKU (auto-generated if not provided)
  sku: z.string().min(1).max(100).optional(),
});

const UpdateProductSchema = CreateProductSchema.partial();

const ProductFiltersSchema = z.object({
  // Pagination
  page: z.number().int().positive().default(1),
  limit: z.number().int().min(1).max(100).default(20),
  
  // Sorting
  sortBy: z.enum(['createdAt', 'updatedAt', 'price', 'name', 'popularity']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  
  // Filtering
  categoryId: z.string().cuid().optional(),
  brandId: z.string().cuid().optional(),
  status: z.nativeEnum(ProductStatus).optional(),
  isFeatured: z.boolean().optional(),
  isAvailable: z.boolean().optional(),
  minPrice: z.number().positive().optional(),
  maxPrice: z.number().positive().optional(),
  searchQuery: z.string().optional(),
  inStock: z.boolean().optional(),
  
  // Variant filters
  variantFilters: z.record(z.string(), z.any()).optional(),
});

type CreateProductInput = z.infer<typeof CreateProductSchema>;
type UpdateProductInput = z.infer<typeof UpdateProductSchema>;
type ProductFilters = z.infer<typeof ProductFiltersSchema>;

// ============ PRODUCT SERVICE ============
export class ProductService {
  private readonly CACHE_TTL = 60 * 5; // 5 minutes
  private readonly POPULAR_PRODUCTS_KEY = 'popular:products';
  
  // ============ CREATE PRODUCT ============
  async createProduct(data: CreateProductInput): Promise<Product> {
    try {
      // Validate input
      const validatedData = CreateProductSchema.parse(data);
      
      // Generate SKU if not provided
      const sku = validatedData.sku || await this.generateSKU(validatedData.name);
      
      // Generate slug from name
      const slug = await this.generateUniqueSlug(validatedData.name);
      
      // Prepare product data
      const productData = {
        ...validatedData,
        sku,
        slug,
        basePrice: parseFloat(validatedData.basePrice.toString()),
        compareAtPrice: validatedData.compareAtPrice ? parseFloat(validatedData.compareAtPrice.toString()) : null,
        costPrice: validatedData.costPrice ? parseFloat(validatedData.costPrice.toString()) : null,
        weight: validatedData.weight || null,
        width: validatedData.width || null,
        height: validatedData.height || null,
        depth: validatedData.depth || null,
        publishedAt: validatedData.status === ProductStatus.ACTIVE ? new Date() : null,
      };
      
      // Handle variants based on product type
      let variantsData = [];
      if (validatedData.type === ProductType.VARIABLE && validatedData.variants) {
        variantsData = validatedData.variants.map(variant => ({
          ...variant,
          price: parseFloat(variant.price.toString()),
          compareAtPrice: variant.compareAtPrice ? parseFloat(variant.compareAtPrice.toString()) : null,
          costPrice: variant.costPrice ? parseFloat(variant.costPrice.toString()) : null,
        }));
      } else if (validatedData.type === ProductType.SIMPLE) {
        // Create single variant for simple product
        variantsData = [{
          sku: `${sku}-001`,
          name: validatedData.name,
          optionValues: {},
          price: parseFloat(validatedData.basePrice.toString()),
          compareAtPrice: validatedData.compareAtPrice ? parseFloat(validatedData.compareAtPrice.toString()) : null,
          costPrice: validatedData.costPrice ? parseFloat(validatedData.costPrice.toString()) : null,
          quantity: 0,
          isAvailable: true,
        }];
      }
      
      // Create product with transaction
      const product = await prisma.$transaction(async (tx) => {
        // Create product
        const createdProduct = await tx.product.create({
          data: {
            ...productData,
            // Relations
            options: validatedData.options ? {
              create: validatedData.options.map(opt => ({
                name: opt.name,
                values: opt.values,
                position: opt.position || 0,
              }))
            } : undefined,
            attributes: validatedData.attributes ? {
              create: validatedData.attributes.map(attr => ({
                name: attr.name,
                value: attr.value,
              }))
            } : undefined,
            images: validatedData.images ? {
              create: validatedData.images.map(img => ({
                url: img.url,
                altText: img.altText,
                position: img.position || 0,
                publicId: img.publicId,
                width: img.width,
                height: img.height,
              }))
            } : undefined,
          },
        });
        
        // Create variants if any
        if (variantsData.length > 0) {
          await tx.productVariant.createMany({
            data: variantsData.map(variant => ({
              ...variant,
              productId: createdProduct.id,
            })),
          });
          
          // For variable products, create inventory records
          if (validatedData.type === ProductType.VARIABLE) {
            const variants = await tx.productVariant.findMany({
              where: { productId: createdProduct.id },
            });
            
            if (createdProduct.manageInventory) {
              await Promise.all(
                variants.map(variant =>
                  tx.inventory.create({
                    data: {
                      variantId: variant.id,
                      quantity: variant.quantity || 0,
                      available: variant.quantity || 0,
                    },
                  })
                )
              );
            }
          }
        }
        
        // Create price history entry
        await tx.priceHistory.create({
          data: {
            productId: createdProduct.id,
            newPrice: createdProduct.basePrice,
            changeType: 'REGULAR_UPDATE',
            reason: 'Initial product creation',
          },
        });
        
        return createdProduct;
      });
      
      // Clear relevant caches
      await this.clearProductCaches();
      
      return product;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid product data', error.errors);
      }
      if (error instanceof Prisma.PrismaClientKnownRequestError) {
        if (error.code === 'P2002') {
          throw new ValidationError('SKU or slug already exists');
        }
      }
      throw error;
    }
  }
  
  // ============ UPDATE PRODUCT ============
  async updateProduct(id: string, data: UpdateProductInput): Promise<Product> {
    try {
      // Check if product exists
      const existingProduct = await prisma.product.findUnique({
        where: { id },
        include: { variants: true },
      });
      
      if (!existingProduct) {
        throw new NotFoundError(`Product with ID ${id} not found`);
      }
      
      // Validate input
      const validatedData = UpdateProductSchema.parse(data);
      
      // Generate new slug if name changed
      let slug = existingProduct.slug;
      if (validatedData.name && validatedData.name !== existingProduct.name) {
        slug = await this.generateUniqueSlug(validatedData.name, id);
      }
      
      // Handle price change history
      let priceHistoryData = null;
      if (validatedData.basePrice && parseFloat(validatedData.basePrice.toString()) !== existingProduct.basePrice.toNumber()) {
        priceHistoryData = {
          productId: id,
          oldPrice: existingProduct.basePrice,
          newPrice: parseFloat(validatedData.basePrice.toString()),
          changeType: 'REGULAR_UPDATE',
          reason: 'Product price update',
        };
      }
      
      // Update product with transaction
      const updatedProduct = await prisma.$transaction(async (tx) => {
        // Update product
        const product = await tx.product.update({
          where: { id },
          data: {
            ...validatedData,
            slug,
            basePrice: validatedData.basePrice ? parseFloat(validatedData.basePrice.toString()) : undefined,
            compareAtPrice: validatedData.compareAtPrice !== undefined 
              ? validatedData.compareAtPrice ? parseFloat(validatedData.compareAtPrice.toString()) : null 
              : undefined,
            costPrice: validatedData.costPrice !== undefined 
              ? validatedData.costPrice ? parseFloat(validatedData.costPrice.toString()) : null 
              : undefined,
            publishedAt: validatedData.status === ProductStatus.ACTIVE && !existingProduct.publishedAt
              ? new Date()
              : undefined,
          },
        });
        
        // Create price history if price changed
        if (priceHistoryData) {
          await tx.priceHistory.create({
            data: priceHistoryData,
          });
        }
        
        // Update variants if provided
        if (validatedData.variants && existingProduct.type === ProductType.VARIABLE) {
          // Note: In production, you'd want more sophisticated variant management
          // This is a simplified version
          for (const variantData of validatedData.variants) {
            await tx.productVariant.updateMany({
              where: {
                productId: id,
                sku: variantData.sku,
              },
              data: {
                price: parseFloat(variantData.price.toString()),
                compareAtPrice: variantData.compareAtPrice ? parseFloat(variantData.compareAtPrice.toString()) : null,
                costPrice: variantData.costPrice ? parseFloat(variantData.costPrice.toString()) : null,
                quantity: variantData.quantity,
                isAvailable: variantData.isAvailable,
              },
            });
          }
        }
        
        return product;
      });
      
      // Clear caches
      await this.clearProductCaches(id);
      
      return updatedProduct;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid product data', error.errors);
      }
      throw error;
    }
  }
  
  // ============ DELETE PRODUCT (SOFT DELETE) ============
  async deleteProduct(id: string): Promise<void> {
    try {
      // Check if product exists
      const product = await prisma.product.findUnique({
        where: { id },
      });
      
      if (!product) {
        throw new NotFoundError(`Product with ID ${id} not found`);
      }
      
      // Soft delete
      await prisma.product.update({
        where: { id },
        data: {
          status: ProductStatus.ARCHIVED,
          deletedAt: new Date(),
          isAvailable: false,
        },
      });
      
      // Also mark variants as unavailable
      await prisma.productVariant.updateMany({
        where: { productId: id },
        data: { isAvailable: false },
      });
      
      // Clear caches
      await this.clearProductCaches(id);
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET PRODUCT WITH DETAILS ============
  async getProduct(idOrSlug: string): Promise<any> {
    try {
      // Try to get from cache first
      const cacheKey = `product:${idOrSlug}`;
      const cached = await redis.get(cacheKey);
      
      if (cached) {
        return JSON.parse(cached);
      }
      
      // Determine if it's an ID or slug
      const isCuid = idOrSlug.length === 25 && idOrSlug.startsWith('c');
      const whereClause = isCuid ? { id: idOrSlug } : { slug: idOrSlug };
      
      // Fetch from database
      const product = await prisma.product.findUnique({
        where: whereClause,
        include: {
          category: true,
          brand: true,
          variants: {
            where: { isAvailable: true },
            include: {
              inventory: {
                where: { warehouse: { isDefault: true } },
              },
            },
          },
          options: true,
          attributes: true,
          images: {
            orderBy: { position: 'asc' },
          },
          reviews: {
            where: { status: 'APPROVED' },
            take: 5,
            include: {
              user: {
                select: {
                  id: true,
                  name: true,
                  image: true,
                },
              },
            },
          },
        },
      });
      
      if (!product) {
        throw new NotFoundError(`Product not found`);
      }
      
      // Calculate average rating
      const avgRating = await this.calculateProductRating(product.id);
      
      // Get related products
      const relatedProducts = await this.getRelatedProducts(product.id, 4);
      
      // Combine data
      const result = {
        ...product,
        avgRating,
        relatedProducts,
      };
      
      // Cache the result
      await redis.setex(cacheKey, this.CACHE_TTL, JSON.stringify(result));
      
      return result;
    } catch (error) {
      throw error;
    }
  }
  
  // ============ LIST PRODUCTS WITH FILTERS ============
  async listProducts(filters: ProductFilters): Promise<{
    products: any[];
    total: number;
    page: number;
    totalPages: number;
    hasMore: boolean;
  }> {
    try {
      // Validate filters
      const validatedFilters = ProductFiltersSchema.parse(filters);
      
      const { page, limit, sortBy, sortOrder, ...filterCriteria } = validatedFilters;
      const skip = (page - 1) * limit;
      
      // Build where clause
      const where: any = {
        deletedAt: null,
        status: ProductStatus.ACTIVE,
        isAvailable: true,
      };
      
      // Apply filters
      if (filterCriteria.categoryId) {
        // Get all subcategories if hierarchical
        const categoryIds = await this.getCategoryAndSubcategoryIds(filterCriteria.categoryId);
        where.categoryId = { in: categoryIds };
      }
      
      if (filterCriteria.brandId) {
        where.brandId = filterCriteria.brandId;
      }
      
      if (filterCriteria.isFeatured !== undefined) {
        where.isFeatured = filterCriteria.isFeatured;
      }
      
      if (filterCriteria.minPrice !== undefined || filterCriteria.maxPrice !== undefined) {
        where.basePrice = {};
        if (filterCriteria.minPrice !== undefined) {
          where.basePrice.gte = filterCriteria.minPrice;
        }
        if (filterCriteria.maxPrice !== undefined) {
          where.basePrice.lte = filterCriteria.maxPrice;
        }
      }
      
      if (filterCriteria.inStock !== undefined) {
        if (filterCriteria.inStock) {
          // For simple products
          where.variants = {
            some: {
              quantity: { gt: 0 },
              isAvailable: true,
            },
          };
        }
      }
      
      // Search query
      if (filterCriteria.searchQuery) {
        where.OR = [
          { name: { contains: filterCriteria.searchQuery, mode: 'insensitive' } },
          { description: { contains: filterCriteria.searchQuery, mode: 'insensitive' } },
          { sku: { contains: filterCriteria.searchQuery, mode: 'insensitive' } },
        ];
      }
      
      // Build orderBy
      let orderBy: any = {};
      switch (sortBy) {
        case 'price':
          orderBy = { basePrice: sortOrder };
          break;
        case 'name':
          orderBy = { name: sortOrder };
          break;
        case 'popularity':
          // You'd need a popularity metric - using createdAt as fallback
          orderBy = { createdAt: 'desc' };
          break;
        default:
          orderBy = { [sortBy]: sortOrder };
      }
      
      // Execute queries in parallel
      const [products, total] = await Promise.all([
        prisma.product.findMany({
          where,
          include: {
            category: true,
            brand: true,
            variants: {
              where: { isAvailable: true },
              take: 1, // Get first variant for price display
            },
            images: {
              take: 1,
              orderBy: { position: 'asc' },
            },
            reviews: {
              select: {
                rating: true,
              },
            },
          },
          orderBy,
          skip,
          take: limit,
        }),
        prisma.product.count({ where }),
      ]);
      
      // Calculate average ratings
      const productsWithRating = products.map(product => ({
        ...product,
        avgRating: product.reviews.length > 0
          ? product.reviews.reduce((sum, review) => sum + review.rating, 0) / product.reviews.length
          : 0,
      }));
      
      return {
        products: productsWithRating,
        total,
        page,
        totalPages: Math.ceil(total / limit),
        hasMore: page * limit < total,
      };
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid filter parameters', error.errors);
      }
      throw error;
    }
  }
  
  // ============ SEARCH PRODUCTS (FULL-TEXT) ============
  async searchProducts(query: string, limit: number = 20): Promise<any[]> {
    try {
      if (!query || query.trim().length < 2) {
        return [];
      }
      
      // For PostgreSQL full-text search (simplified version)
      // In production, use PostgreSQL full-text search or Algolia
      const products = await prisma.product.findMany({
        where: {
          OR: [
            { name: { contains: query, mode: 'insensitive' } },
            { description: { contains: query, mode: 'insensitive' } },
            { shortDescription: { contains: query, mode: 'insensitive' } },
            { sku: { contains: query, mode: 'insensitive' } },
          ],
          status: ProductStatus.ACTIVE,
          isAvailable: true,
          deletedAt: null,
        },
        include: {
          category: true,
          brand: true,
          variants: {
            where: { isAvailable: true },
            take: 1,
          },
          images: {
            take: 1,
            orderBy: { position: 'asc' },
          },
        },
        take: limit,
      });
      
      return products;
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET PRODUCTS BY CATEGORY ============
  async getProductsByCategory(categoryId: string, limit: number = 12): Promise<any[]> {
    try {
      // Get category and all subcategories
      const categoryIds = await this.getCategoryAndSubcategoryIds(categoryId);
      
      const products = await prisma.product.findMany({
        where: {
          categoryId: { in: categoryIds },
          status: ProductStatus.ACTIVE,
          isAvailable: true,
          deletedAt: null,
        },
        include: {
          category: true,
          brand: true,
          variants: {
            where: { isAvailable: true },
            take: 1,
          },
          images: {
            take: 1,
            orderBy: { position: 'asc' },
          },
          reviews: {
            select: {
              rating: true,
            },
          },
        },
        take: limit,
      });
      
      // Calculate average ratings
      return products.map(product => ({
        ...product,
        avgRating: product.reviews.length > 0
          ? product.reviews.reduce((sum, review) => sum + review.rating, 0) / product.reviews.length
          : 0,
      }));
    } catch (error) {
      throw error;
    }
  }
  
  // ============ GET RELATED PRODUCTS ============
  async getRelatedProducts(productId: string, limit: number = 4): Promise<any[]> {
    try {
      const product = await prisma.product.findUnique({
        where: { id: productId },
        include: { category: true, brand: true },
      });
      
      if (!product) {
        return [];
      }
      
      // Find related products by:
      // 1. Same category
      // 2. Same brand
      // 3. Similar price range (±20%)
      const priceMin = product.basePrice.toNumber() * 0.8;
      const priceMax = product.basePrice.toNumber() * 1.2;
      
      const related = await prisma.product.findMany({
        where: {
          AND: [
            { id: { not: productId } },
            { status: ProductStatus.ACTIVE },
            { isAvailable: true },
            { deletedAt: null },
            {
              OR: [
                { categoryId: product.categoryId },
                { brandId: product.brandId },
                {
                  basePrice: {
                    gte: priceMin,
                    lte: priceMax,
                  },
                },
              ],
            },
          ],
        },
        include: {
          category: true,
          brand: true,
          variants: {
            where: { isAvailable: true },
            take: 1,
          },
          images: {
            take: 1,
            orderBy: { position: 'asc' },
          },
        },
        take: limit,
      });
      
      return related;
    } catch (error) {
      throw error;
    }
  }
  
  // ============ UPDATE INVENTORY ============
  async updateInventory(variantId: string, quantity: number, warehouseId?: string): Promise<void> {
    try {
      await prisma.$transaction(async (tx) => {
        // Get current inventory
        const whereClause = warehouseId
          ? { variantId_warehouseId: { variantId, warehouseId } }
          : { variantId, warehouse: { isDefault: true } };
        
        const inventory = await tx.inventory.findFirst({
          where: whereClause,
        });
        
        if (!inventory) {
          throw new NotFoundError(`Inventory record not found for variant ${variantId}`);
        }
        
        // Update inventory
        await tx.inventory.update({
          where: { id: inventory.id },
          data: {
            quantity,
            available: quantity - inventory.reserved,
          },
        });
        
        // Create inventory movement record
        await tx.inventoryMovement.create({
          data: {
            variantId,
            warehouseId,
            type: 'ADJUSTMENT',
            quantity: quantity - inventory.quantity,
            previousQty: inventory.quantity,
            newQty: quantity,
            reason: 'Manual inventory adjustment',
          },
        });
        
        // Update variant quantity
        await tx.productVariant.update({
          where: { id: variantId },
          data: { quantity },
        });
      });
      
      // Clear relevant caches
      await this.clearProductCaches();
      
    } catch (error) {
      throw error;
    }
  }
  
  // ============ CHECK AVAILABILITY ============
  async checkAvailability(variantId: string, quantity: number, warehouseId?: string): Promise<{
    available: boolean;
    availableQuantity: number;
    canBackorder: boolean;
    estimatedRestockDate?: Date;
  }> {
    try {
      const variant = await prisma.productVariant.findUnique({
        where: { id: variantId },
        include: {
          product: {
            select: {
              allowBackorder: true,
              manageInventory: true,
            },
          },
          inventory: {
            where: warehouseId 
              ? { warehouseId }
              : { warehouse: { isDefault: true } },
          },
        },
      });
      
      if (!variant) {
        throw new NotFoundError(`Variant with ID ${variantId} not found`);
      }
      
      if (!variant.isAvailable) {
        return {
          available: false,
          availableQuantity: 0,
          canBackorder: false,
        };
      }
      
      if (!variant.product.manageInventory) {
        return {
          available: true,
          availableQuantity: Number.MAX_SAFE_INTEGER,
          canBackorder: false,
        };
      }
      
      const inventory = variant.inventory[0];
      const availableQuantity = inventory ? inventory.available : 0;
      
      const canBackorder = variant.product.allowBackorder;
      
      return {
        available: availableQuantity >= quantity,
        availableQuantity,
        canBackorder,
        estimatedRestockDate: canBackorder ? new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) : undefined,
      };
    } catch (error) {
      throw error;
    }
  }
  
  // ============ UTILITY METHODS ============
  private async generateSKU(name: string): Promise<string> {
    const prefix = name.substring(0, 3).toUpperCase();
    const timestamp = Date.now().toString().slice(-6);
    const random = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
    return `${prefix}-${timestamp}-${random}`;
  }
  
  private async generateUniqueSlug(name: string, excludeId?: string): Promise<string> {
    const baseSlug = generateSlug(name);
    let slug = baseSlug;
    let counter = 1;
    
    while (true) {
      const existing = await prisma.product.findUnique({
        where: { slug },
      });
      
      if (!existing || (excludeId && existing.id === excludeId)) {
        break;
      }
      
      slug = `${baseSlug}-${counter}`;
      counter++;
    }
    
    return slug;
  }
  
  private async getCategoryAndSubcategoryIds(categoryId: string): Promise<string[]> {
    const categories = await prisma.category.findMany({
      where: {
        OR: [
          { id: categoryId },
          { parentId: categoryId },
        ],
      },
      select: { id: true },
    });
    
    return categories.map(c => c.id);
  }
  
  private async calculateProductRating(productId: string): Promise<number> {
    const reviews = await prisma.review.aggregate({
      where: {
        productId,
        status: 'APPROVED',
      },
      _avg: {
        rating: true,
      },
      _count: {
        rating: true,
      },
    });
    
    return reviews._avg.rating || 0;
  }
  
  private async clearProductCaches(productId?: string): Promise<void> {
    const pipeline = redis.pipeline();
    
    if (productId) {
      pipeline.del(`product:${productId}`);
    }
    
    // Clear product lists caches
    const pattern = 'products:*';
    const keys = await redis.keys(pattern);
    keys.forEach(key => pipeline.del(key));
    
    pipeline.del(this.POPULAR_PRODUCTS_KEY);
    
    await pipeline.exec();
  }
}

§ 2.4 API ROUTES/TRPC ROUTER PER PRODUCTS

typescript
// app/api/trpc/trpc-router.ts
import { initTRPC, TRPCError } from '@trpc/server';
import { z } from 'zod';
import superjson from 'superjson';
import { ProductService } from '@/lib/services/product-service';
import { requireAuth, requireAdmin } from '@/lib/auth/middleware';

const t = initTRPC.create({
  transformer: superjson,
});

const productService = new ProductService();

export const productRouter = t.router({
  // ============ PUBLIC ROUTES ============
  list: t.procedure
    .input(z.object({
      page: z.number().min(1).default(1),
      limit: z.number().min(1).max(100).default(20),
      sortBy: z.enum(['createdAt', 'price', 'name', 'popularity']).default('createdAt'),
      sortOrder: z.enum(['asc', 'desc']).default('desc'),
      categoryId: z.string().cuid().optional(),
      brandId: z.string().cuid().optional(),
      minPrice: z.number().positive().optional(),
      maxPrice: z.number().positive().optional(),
      inStock: z.boolean().optional(),
      searchQuery: z.string().optional(),
    }))
    .query(async ({ input }) => {
      try {
        const result = await productService.listProducts(input);
        return {
          success: true,
          data: result,
        };
      } catch (error) {
        console.error('Error listing products:', error);
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch products',
        });
      }
    }),
  
  get: t.procedure
    .input(z.object({
      idOrSlug: z.string(),
    }))
    .query(async ({ input }) => {
      try {
        const product = await productService.getProduct(input.idOrSlug);
        return {
          success: true,
          data: product,
        };
      } catch (error) {
        if (error instanceof NotFoundError) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Product not found',
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch product',
        });
      }
    }),
  
  search: t.procedure
    .input(z.object({
      query: z.string().min(2),
      limit: z.number().min(1).max(50).default(20),
    }))
    .query(async ({ input }) => {
      try {
        const products = await productService.searchProducts(input.query, input.limit);
        return {
          success: true,
          data: products,
        };
      } catch (error) {
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Search failed',
        });
      }
    }),
  
  byCategory: t.procedure
    .input(z.object({
      categoryId: z.string().cuid(),
      limit: z.number().min(1).max(50).default(12),
    }))
    .query(async ({ input }) => {
      try {
        const products = await productService.getProductsByCategory(input.categoryId, input.limit);
        return {
          success: true,
          data: products,
        };
      } catch (error) {
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch products by category',
        });
      }
    }),
  
  related: t.procedure
    .input(z.object({
      productId: z.string().cuid(),
      limit: z.number().min(1).max(12).default(4),
    }))
    .query(async ({ input }) => {
      try {
        const products = await productService.getRelatedProducts(input.productId, input.limit);
        return {
          success: true,
          data: products,
        };
      } catch (error) {
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to fetch related products',
        });
      }
    }),
  
  checkAvailability: t.procedure
    .input(z.object({
      variantId: z.string().cuid(),
      quantity: z.number().int().positive(),
      warehouseId: z.string().cuid().optional(),
    }))
    .query(async ({ input }) => {
      try {
        const availability = await productService.checkAvailability(
          input.variantId,
          input.quantity,
          input.warehouseId
        );
        return {
          success: true,
          data: availability,
        };
      } catch (error) {
        if (error instanceof NotFoundError) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Product variant not found',
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to check availability',
        });
      }
    }),
  
  // ============ ADMIN ROUTES ============
  create: t.procedure
    .use(requireAdmin)
    .input(z.any()) // Use CreateProductSchema from service
    .mutation(async ({ input }) => {
      try {
        const product = await productService.createProduct(input);
        return {
          success: true,
          data: product,
          message: 'Product created successfully',
        };
      } catch (error) {
        if (error instanceof ValidationError) {
          throw new TRPCError({
            code: 'BAD_REQUEST',
            message: error.message,
            cause: error.cause,
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to create product',
        });
      }
    }),
  
  update: t.procedure
    .use(requireAdmin)
    .input(z.object({
      id: z.string().cuid(),
      data: z.any(), // Use UpdateProductSchema from service
    }))
    .mutation(async ({ input }) => {
      try {
        const product = await productService.updateProduct(input.id, input.data);
        return {
          success: true,
          data: product,
          message: 'Product updated successfully',
        };
      } catch (error) {
        if (error instanceof NotFoundError) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Product not found',
          });
        }
        if (error instanceof ValidationError) {
          throw new TRPCError({
            code: 'BAD_REQUEST',
            message: error.message,
            cause: error.cause,
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to update product',
        });
      }
    }),
  
  delete: t.procedure
    .use(requireAdmin)
    .input(z.object({
      id: z.string().cuid(),
    }))
    .mutation(async ({ input }) => {
      try {
        await productService.deleteProduct(input.id);
        return {
          success: true,
          message: 'Product deleted successfully',
        };
      } catch (error) {
        if (error instanceof NotFoundError) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Product not found',
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to delete product',
        });
      }
    }),
  
  updateInventory: t.procedure
    .use(requireAdmin)
    .input(z.object({
      variantId: z.string().cuid(),
      quantity: z.number().int(),
      warehouseId: z.string().cuid().optional(),
    }))
    .mutation(async ({ input }) => {
      try {
        await productService.updateInventory(
          input.variantId,
          input.quantity,
          input.warehouseId
        );
        return {
          success: true,
          message: 'Inventory updated successfully',
        };
      } catch (error) {
        if (error instanceof NotFoundError) {
          throw new TRPCError({
            code: 'NOT_FOUND',
            message: 'Inventory record not found',
          });
        }
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Failed to update inventory',
        });
      }
    }),
});

§ 2.5 VARIANT MATRIX GENERATOR

typescript
// lib/utils/variant-generator.ts
import { z } from 'zod';

export interface VariantOption {
  name: string;
  values: string[];
}

export interface GeneratedVariant {
  id?: string;
  sku?: string;
  optionValues: Record<string, string>;
  price: number;
  compareAtPrice?: number;
  quantity: number;
  [key: string]: any;
}

// ============ VALIDATION SCHEMA ============
const VariantOptionsSchema = z.record(
  z.string().min(1), // Option name
  z.array(z.string().min(1)).min(1) // Option values
);

// ============ VARIANT MATRIX GENERATOR ============
export class VariantMatrixGenerator {
  /**
   * Generate all possible combinations of variant options
   * @param options - Object mapping option names to arrays of values
   * @returns Array of all combinations
   */
  static generateCombinations(options: Record<string, string[]>): Record<string, string>[] {
    try {
      // Validate input
      const validatedOptions = VariantOptionsSchema.parse(options);
      
      const optionNames = Object.keys(validatedOptions);
      const optionValues = Object.values(validatedOptions);
      
      if (optionNames.length === 0) {
        return [];
      }
      
      // Calculate total number of combinations
      const totalCombinations = optionValues.reduce((total, values) => total * values.length, 1);
      
      if (totalCombinations > 1000) {
        throw new Error(`Too many combinations: ${totalCombinations}. Maximum allowed is 1000.`);
      }
      
      // Generate combinations using cartesian product
      const combinations: Record<string, string>[] = [];
      
      const generate = (index: number, current: Record<string, string>) => {
        if (index === optionNames.length) {
          combinations.push({ ...current });
          return;
        }
        
        const optionName = optionNames[index];
        const values = optionValues[index];
        
        for (const value of values) {
          current[optionName] = value;
          generate(index + 1, current);
        }
      };
      
      generate(0, {});
      
      return combinations;
    } catch (error) {
      if (error instanceof z.ZodError) {
        throw new ValidationError('Invalid variant options', error.errors);
      }
      throw error;
    }
  }
  
  /**
   * Generate SKUs for each variant combination
   * @param baseSKU - Base product SKU
   * @param combinations - Array of variant combinations
   * @param existingSKUs - Set of existing SKUs to avoid collisions
   * @returns Array of variants with generated SKUs
   */
  static generateVariantsWithSKUs(
    baseSKU: string,
    combinations: Record<string, string>[],
    existingSKUs: Set<string> = new Set()
  ): GeneratedVariant[] {
    const variants: GeneratedVariant[] = [];
    
    for (let i = 0; i < combinations.length; i++) {
      const combination = combinations[i];
      
      // Generate SKU based on combination
      let sku = baseSKU;
      const optionCodes: string[] = [];
      
      Object.entries(combination).forEach(([optionName, value]) => {
        // Create code from first 2 letters of option name and value
        const optionCode = optionName.substring(0, 2).toUpperCase();
        const valueCode = value.substring(0, 3).toUpperCase().replace(/\s/g, '');
        optionCodes.push(`${optionCode}-${valueCode}`);
      });
      
      if (optionCodes.length > 0) {
        sku += `-${optionCodes.join('-')}`;
      }
      
      // Ensure SKU is unique
      let uniqueSku = sku;
      let counter = 1;
      while (existingSKUs.has(uniqueSku)) {
        uniqueSku = `${sku}-${counter}`;
        counter++;
      }
      
      existingSKUs.add(uniqueSku);
      
      variants.push({
        sku: uniqueSku,
        optionValues: combination,
        price: 0, // Default price, should be set by product
        compareAtPrice: undefined,
        quantity: 0,
        isAvailable: true,
        position: i,
      });
    }
    
    return variants;
  }
  
  /**
   * Generate variant names based on combinations
   * @param combinations - Array of variant combinations
   * @param productName - Base product name
   * @returns Array of variant names
   */
  static generateVariantNames(
    combinations: Record<string, string>[],
    productName: string
  ): string[] {
    return combinations.map(combination => {
      const optionStrings = Object.entries(combination)
        .map(([key, value]) => `${key}: ${value}`)
        .join(', ');
      
      return `${productName} (${optionStrings})`;
    });
  }
  
  /**
   * Calculate total number of possible combinations
   * @param options - Variant options
   * @returns Total number of combinations
   */
  static calculateTotalCombinations(options: Record<string, string[]>): number {
    const optionValues = Object.values(options);
    if (optionValues.length === 0) return 0;
    
    return optionValues.reduce((total, values) => total * values.length, 1);
  }
  
  /**
   * Filter combinations based on exclusion rules
   * @param combinations - All possible combinations
   * @param exclusions - Array of exclusion rules
   * @returns Filtered combinations
   */
  static filterCombinations(
    combinations: Record<string, string>[],
    exclusions: Array<Record<string, string>>
  ): Record<string, string>[] {
    if (exclusions.length === 0) {
      return combinations;
    }
    
    return combinations.filter(combination => {
      return !exclusions.some(exclusion => {
        // Check if combination matches all properties in exclusion
        return Object.entries(exclusion).every(([key, value]) => {
          return combination[key] === value;
        });
      });
    });
  }
}

// ============ EXAMPLE USAGE ============
/*
const options = {
  size: ['S', 'M', 'L', 'XL'],
  color: ['Red', 'Blue', 'Green'],
  material: ['Cotton', 'Polyester'],
};

const combinations = VariantMatrixGenerator.generateCombinations(options);
console.log(`Total combinations: ${combinations.length}`);
// Output: 24 combinations (4 * 3 * 2)

const variants = VariantMatrixGenerator.generateVariantsWithSKUs('TSHIRT-001', combinations);
console.log(variants[0]);
// Output: { sku: 'TSHIRT-001-SI-S-RED-CO-COTTON', optionValues: { size: 'S', color: 'Red', material: 'Cotton' }, ... }

const names = VariantMatrixGenerator.generateVariantNames(combinations, 'T-Shirt');
console.log(names[0]);
// Output: 'T-Shirt (size: S, color: Red, material: Cotton)'

// Filter out invalid combinations
const exclusions = [
  { size: 'XL', material: 'Polyester' }, // No XL in Polyester
];
const filtered = VariantMatrixGenerator.filterCombinations(combinations, exclusions);
console.log(`Filtered combinations: ${filtered.length}`);
// Output: 20 combinations (removes 4 XL+Polyester combos)
*/

---

**NOTA:** Questo è solo il §2 (Product Catalog System) della documentazione completa. La risposta completa richiederebbe oltre 3.500 righe di codice e documentazione. Continuerei con le altre sezioni se vuoi vedere il resto del catalogo.