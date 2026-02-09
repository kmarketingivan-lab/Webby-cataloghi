# CATALOGO-ECOMMERCE-PRODUCTS-v1

> **Dominio**: E-Commerce
> **Stack**: Next.js, React, TypeScript, Prisma, tRPC, Zod
> **Versione**: 1.0
> **Data**: 2026-02-04

---

## 1. INDICE

| # | Sezione | Path |
|---|---------|------|
| 1 | [`prisma/schema-ecommerce-products.prisma`](#1-`prisma-schema-ecommerce-products-prisma`) | ``prisma/schema-ecommerce-products.prisma`` |
| 2 | [`src/server/services/product-service.ts`](#2-`src-server-services-product-service-ts`) | ``src/server/services/product-service.ts`` |
| 3 | [`src/server/services/category-service.ts`](#3-`src-server-services-category-service-ts`) | ``src/server/services/category-service.ts`` |
| 4 | [`src/server/trpc/routers/products.ts`](#4-`src-server-trpc-routers-products-ts`) | ``src/server/trpc/routers/products.ts`` |
| 5 | [`src/lib/validations/product.ts`](#5-`src-lib-validations-product-ts`) | ``src/lib/validations/product.ts`` |
| 6 | [`src/hooks/use-products.ts`](#6-`src-hooks-use-products-ts`) | ``src/hooks/use-products.ts`` |
| 7 | [`src/components/products/product-card.tsx`](#7-`src-components-products-product-card-tsx`) | ``src/components/products/product-card.tsx`` |
| 8 | [`src/components/products/product-grid.tsx`](#8-`src-components-products-product-grid-tsx`) | ``src/components/products/product-grid.tsx`` |
| 9 | [`src/components/products/product-filters.tsx`](#9-`src-components-products-product-filters-tsx`) | ``src/components/products/product-filters.tsx`` |
| 10 | [`src/components/products/product-detail.tsx`](#10-`src-components-products-product-detail-tsx`) | ``src/components/products/product-detail.tsx`` |
| 11 | [`src/app/(shop)/products/page.tsx`](#11-`src-app-(shop)-products-page-tsx`) | ``src/app/(shop)/products/page.tsx`` |
| 12 | [`src/app/(shop)/products/[slug]/page.tsx`](#12-`src-app-(shop)-products-slug-page-tsx`) | ``src/app/(shop)/products/[slug]/page.tsx`` |
| 13 | [`tests/products.test.ts`](#13-`tests-products-test-ts`) | ``tests/products.test.ts`` |

---

## 1. `prisma/schema-ecommerce-products.prisma`

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql" // Or your preferred database
  url      = env("DATABASE_URL")
}

// --- Enums ---

enum ProductStatus {
  DRAFT    // Product is being created/edited
  ACTIVE   // Product is visible and purchasable
  ARCHIVED // Product is no longer active, but kept for historical data
}

enum Visibility {
  VISIBLE      // Visible in catalog and search
  HIDDEN       // Not visible anywhere
  CATALOG_ONLY // Visible in catalog, but not in search results
  SEARCH_ONLY  // Visible in search results, but not directly in catalog
}

enum AttributeType {
  TEXT
  NUMBER
  COLOR
  BOOLEAN
  SELECT // For predefined options
}

// --- Core Models ---

// Product model completo
model Product {
  id                String             @id @default(cuid())
  name              String
  slug              String             @unique
  description       String?            @db.Text
  shortDescription  String?            @db.VarChar(500) // A brief summary for listings

  // Pricing
  price             Decimal            @db.Decimal(10, 2)
  compareAtPrice    Decimal?           @db.Decimal(10, 2) // Original price for sale display
  costPrice         Decimal?           @db.Decimal(10, 2) // Internal cost for profit calculation

  // Status & Visibility
  status            ProductStatus      @default(DRAFT)
  visibility        Visibility         @default(VISIBLE)
  isFeatured        Boolean            @default(false)
  isDigital         Boolean            @default(false) // If true, no shipping required

  // Inventory
  sku               String?            @unique // Stock Keeping Unit
  barcode           String?            @unique // EAN, UPC, ISBN, etc.
  quantity          Int                @default(0) // Available stock
  trackInventory    Boolean            @default(true) // Whether to manage stock levels
  allowBackorder    Boolean            @default(false) // Allow orders even if out of stock
  lowStockThreshold Int                @default(5) // Alert when stock falls below this

  // Shipping (if not digital)
  weight            Decimal?           @db.Decimal(10, 3) // in kg or lbs
  width             Decimal?           @db.Decimal(10, 3) // in cm or inches
  height            Decimal?           @db.Decimal(10, 3)
  depth             Decimal?           @db.Decimal(10, 3)

  // SEO
  seoTitle          String?            @db.VarChar(160)
  seoDescription    String?            @db.VarChar(300)

  // Relations
  categoryId        String?
  category          Category?          @relation(fields: [categoryId], references: [id], onDelete: SetNull)
  brandId           String?
  brand             Brand?             @relation(fields: [brandId], references: [id], onDelete: SetNull)

  images            ProductImage[]
  variants          ProductVariant[]
  options           ProductOption[]
  attributes        ProductAttribute[]
  reviews           Review[]           // Placeholder for Review model
  cartItems         CartItem[]         // Placeholder for CartItem model
  orderItems        OrderItem[]        // Placeholder for OrderItem model
  wishlistItems     WishlistItem[]     // Placeholder for WishlistItem model

  createdAt         DateTime           @default(now())
  updatedAt         DateTime           @updatedAt
  deletedAt         DateTime?          // Soft delete

  @@index([slug])
  @@index([status])
  @@index([visibility])
  @@index([categoryId])
  @@index([brandId])
  @@index([isFeatured])
  @@fulltext([name, description, shortDescription, seoTitle, seoDescription]) // For full-text search
}

model ProductImage {
  id          String   @id @default(cuid())
  productId   String
  product     Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  url         String
  altText     String?
  sortOrder   Int      @default(0) // For display order

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([productId])
  @@index([sortOrder])
}

model ProductOption {
  id          String           @id @default(cuid())
  productId   String
  product     Product          @relation(fields: [productId], references: [id], onDelete: Cascade)
  name        String           // e.g., "Color", "Size"
  values      String[]         // e.g., ["Red", "Blue"], ["S", "M", "L"]

  createdAt   DateTime         @default(now())
  updatedAt   DateTime         @updatedAt

  @@unique([productId, name]) // A product cannot have two options with the same name
  @@index([productId])
}

model ProductVariant {
  id              String           @id @default(cuid())
  productId       String
  product         Product          @relation(fields: [productId], references: [id], onDelete: Cascade)
  sku             String?          @unique // Variant specific SKU
  barcode         String?          @unique // Variant specific barcode
  price           Decimal          @db.Decimal(10, 2) // Variant specific price
  compareAtPrice  Decimal?         @db.Decimal(10, 2)
  costPrice       Decimal?         @db.Decimal(10, 2)
  quantity        Int              @default(0)
  weight          Decimal?         @db.Decimal(10, 3)
  image           String?          // URL of variant specific image
  isActive        Boolean          @default(true)

  optionValues    ProductVariantOptionValue[] // e.g., Color: Red, Size: M

  createdAt       DateTime         @default(now())
  updatedAt       DateTime         @updatedAt

  @@index([productId])
  @@index([isActive])
}

model ProductVariantOptionValue {
  id              String         @id @default(cuid())
  variantId       String
  variant         ProductVariant @relation(fields: [variantId], references: [id], onDelete: Cascade)
  optionName      String         // e.g., "Color"
  optionValue     String         // e.g., "Red"

  @@unique([variantId, optionName]) // A variant can only have one value for a given option
  @@index([variantId])
}

model ProductAttribute {
  id          String        @id @default(cuid())
  productId   String
  product     Product       @relation(fields: [productId], references: [id], onDelete: Cascade)
  name        String        // e.g., "Material", "Processor"
  value       String        // e.g., "Cotton", "Intel i7"
  type        AttributeType @default(TEXT) // Helps with frontend rendering/filtering

  createdAt   DateTime      @default(now())
  updatedAt   DateTime      @updatedAt

  @@unique([productId, name]) // A product cannot have two attributes with the same name
  @@index([productId])
  @@index([name])
}

// Category model with hierarchy
model Category {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?    @db.Text
  imageUrl    String?
  seoTitle    String?    @db.VarChar(160)
  seoDescription String? @db.VarChar(300)

  parentId    String?    // Self-referencing for hierarchy
  parent      Category?  @relation("Subcategories", fields: [parentId], references: [id], onDelete: SetNull)
  children    Category[] @relation("Subcategories")

  products    Product[]

  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  @@index([slug])
  @@index([parentId])
  @@fulltext([name, description, seoTitle, seoDescription])
}

model Brand {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?    @db.Text
  logoUrl     String?
  websiteUrl  String?

  products    Product[]

  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  @@index([slug])
  @@fulltext([name, description])
}

// --- Placeholder Models for Relations ---
// These would be fully defined in their respective modules (e.g., reviews, cart, orders, wishlist)

model Review {
  id          String   @id @default(cuid())
  productId   String
  product     Product  @relation(fields: [productId], references: [id])
  userId      String   // Assuming a User model exists
  rating      Int      @db.SmallInt @check(rating >= 1 AND rating <= 5)
  title       String?
  comment     String?  @db.Text
  isApproved  Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([productId])
  @@index([userId])
}

model CartItem {
  id          String   @id @default(cuid())
  cartId      String   // Assuming a Cart model exists
  productId   String?  // Can be null if variant is primary
  product     Product? @relation(fields: [productId], references: [id])
  variantId   String?  // If a specific variant is added
  variant     ProductVariant? @relation(fields: [variantId], references: [id])
  quantity    Int
  priceAtTime Decimal  @db.Decimal(10, 2) // Price when added to cart

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([cartId])
  @@index([productId])
  @@index([variantId])
}

model OrderItem {
  id          String   @id @default(cuid())
  orderId     String   // Assuming an Order model exists
  productId   String?
  product     Product? @relation(fields: [productId], references: [id])
  variantId   String?
  variant     ProductVariant? @relation(fields: [variantId], references: [id])
  name        String   // Product name at time of order
  sku         String?  // Product SKU at time of order
  quantity    Int
  price       Decimal  @db.Decimal(10, 2) // Price paid for this item
  total       Decimal  @db.Decimal(10, 2) // quantity * price

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@index([orderId])
  @@index([productId])
  @@index([variantId])
}

model WishlistItem {
  id          String   @id @default(cuid())
  wishlistId  String   // Assuming a Wishlist model exists
  productId   String
  product     Product  @relation(fields: [productId], references: [id])
  userId      String   // Assuming a User model exists

  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  @@unique([wishlistId, productId])
  @@index([userId])
  @@index([productId])
}
```

---

## 2. `src/server/services/product-service.ts`

```typescript
import { PrismaClient, Product, ProductStatus, ProductVariant, Visibility } from '@prisma/client';
import slugify from 'slugify';
import { z } from 'zod';

// --- DTOs (Data Transfer Objects) ---

// Base schema for product creation/update
const productBaseSchema = z.object({
  name: z.string().min(1).max(255),
  description: z.string().optional(),
  shortDescription: z.string().max(500).optional(),
  price: z.number().positive(),
  compareAtPrice: z.number().positive().optional(),
  costPrice: z.number().positive().optional(),
  status: z.nativeEnum(ProductStatus).optional(),
  visibility: z.nativeEnum(Visibility).optional(),
  isFeatured: z.boolean().optional(),
  isDigital: z.boolean().optional(),
  sku: z.string().optional(),
  barcode: z.string().optional(),
  quantity: z.number().int().min(0).optional(),
  trackInventory: z.boolean().optional(),
  allowBackorder: z.boolean().optional(),
  lowStockThreshold: z.number().int().min(0).optional(),
  weight: z.number().positive().optional(),
  width: z.number().positive().optional(),
  height: z.number().positive().optional(),
  depth: z.number().positive().optional(),
  seoTitle: z.string().max(160).optional(),
  seoDescription: z.string().max(300).optional(),
  categoryId: z.string().cuid().optional(),
  brandId: z.string().cuid().optional(),
  images: z.array(z.object({ url: z.string().url(), altText: z.string().optional(), sortOrder: z.number().int().optional() })).optional(),
  options: z.array(z.object({ name: z.string(), values: z.array(z.string()) })).optional(),
  attributes: z.array(z.object({ name: z.string(), value: z.string(), type: z.enum(['TEXT', 'NUMBER', 'COLOR', 'BOOLEAN', 'SELECT']).optional() })).optional(),
});

export const CreateProductInput = productBaseSchema.extend({
  slug: z.string().optional(), // Will be generated if not provided
  status: z.nativeEnum(ProductStatus).default(ProductStatus.DRAFT),
  visibility: z.nativeEnum(Visibility).default(Visibility.VISIBLE),
  isFeatured: z.boolean().default(false),
  isDigital: z.boolean().default(false),
  trackInventory: z.boolean().default(true),
  allowBackorder: z.boolean().default(false),
  lowStockThreshold: z.number().int().min(0).default(5),
  quantity: z.number().int().min(0).default(0),
});
export type CreateProductInput = z.infer<typeof CreateProductInput>;

export const UpdateProductInput = productBaseSchema.partial().extend({
  slug: z.string().optional(), // Can be updated, but unique check will apply
});
export type UpdateProductInput = z.infer<typeof UpdateProductInput>;

export const ProductListParams = z.object({
  page: z.number().int().min(1).default(1),
  pageSize: z.number().int().min(1).max(100).default(12),
  categoryId: z.string().cuid().optional(),
  brandId: z.string().cuid().optional(),
  status: z.nativeEnum(ProductStatus).optional(),
  visibility: z.nativeEnum(Visibility).optional(),
  minPrice: z.number().positive().optional(),
  maxPrice: z.number().positive().optional(),
  inStock: z.boolean().optional(),
  isFeatured: z.boolean().optional(),
  sortBy: z.enum(['price', 'name', 'createdAt', 'updatedAt', 'popularity']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().optional(),
});
export type ProductListParams = z.infer<typeof ProductListParams>;

export const SearchParams = z.object({
  page: z.number().int().min(1).default(1),
  pageSize: z.number().int().min(1).max(100).default(12),
});
export type SearchParams = z.infer<typeof SearchParams>;

export const PaginatedResult = z.object({
  data: z.array(z.any()), // Placeholder, will be Product[]
  total: z.number(),
  page: z.number(),
  pageSize: z.number(),
  totalPages: z.number(),
});
export type PaginatedResult<T> = {
  data: T[];
  total: number;
  page: number;
  pageSize: number;
  totalPages: number;
};

export const CreateVariantInput = z.object({
  sku: z.string().optional(),
  barcode: z.string().optional(),
  price: z.number().positive(),
  compareAtPrice: z.number().positive().optional(),
  costPrice: z.number().positive().optional(),
  quantity: z.number().int().min(0).default(0),
  weight: z.number().positive().optional(),
  image: z.string().url().optional(),
  isActive: z.boolean().default(true),
  optionValues: z.array(z.object({ optionName: z.string(), optionValue: z.string() })).min(1),
});
export type CreateVariantInput = z.infer<typeof CreateVariantInput>;

export const UpdateVariantInput = CreateVariantInput.partial();
export type UpdateVariantInput = z.infer<typeof UpdateVariantInput>;

// --- Product Service ---

// In-memory stock reservation system (for demonstration)
// In a real application, this would be persisted in a database or a dedicated service.
interface StockReservation {
  id: string;
  productId: string;
  quantity: number;
  expiresAt: Date;
}
const productReservations = new Map<string, StockReservation>(); // reservationId -> StockReservation

export class ProductService {
  constructor(private prisma: PrismaClient) {}

  // --- CRUD ---

  async create(data: CreateProductInput): Promise<Product> {
    const slug = data.slug || (await this.generateSlug(data.name));
    const product = await this.prisma.product.create({
      data: {
        ...data,
        slug,
        images: data.images ? { createMany: { data: data.images } } : undefined,
        options: data.options ? { createMany: { data: data.options } } : undefined,
        attributes: data.attributes ? { createMany: { data: data.attributes } } : undefined,
      },
      include: {
        images: true,
        options: true,
        attributes: true,
        variants: { include: { optionValues: true } },
        category: true,
        brand: true,
      },
    });
    return product;
  }

  async update(id: string, data: UpdateProductInput): Promise<Product> {
    const slug = data.name ? await this.generateSlug(data.name) : data.slug;

    // Handle nested updates for images, options, attributes
    const updateData: any = { ...data, slug };
    if (data.images) {
      // Clear existing images and create new ones
      await this.prisma.productImage.deleteMany({ where: { productId: id } });
      updateData.images = { createMany: { data: data.images } };
    }
    if (data.options) {
      await this.prisma.productOption.deleteMany({ where: { productId: id } });
      updateData.options = { createMany: { data: data.options } };
    }
    if (data.attributes) {
      await this.prisma.productAttribute.deleteMany({ where: { productId: id } });
      updateData.attributes = { createMany: { data: data.attributes } };
    }

    const product = await this.prisma.product.update({
      where: { id },
      data: updateData,
      include: {
        images: true,
        options: true,
        attributes: true,
        variants: { include: { optionValues: true } },
        category: true,
        brand: true,
      },
    });
    return product;
  }

  async delete(id: string): Promise<void> {
    // Soft delete
    await this.prisma.product.update({
      where: { id },
      data: { deletedAt: new Date() },
    });
    // Or hard delete:
    // await this.prisma.product.delete({ where: { id } });
  }

  async getById(id: string): Promise<Product | null> {
    return this.prisma.product.findUnique({
      where: { id, deletedAt: null },
      include: {
        images: { orderBy: { sortOrder: 'asc' } },
        options: true,
        attributes: true,
        variants: { include: { optionValues: true } },
        category: true,
        brand: true,
      },
    });
  }

  async getBySlug(slug: string): Promise<Product | null> {
    return this.prisma.product.findUnique({
      where: { slug, deletedAt: null },
      include: {
        images: { orderBy: { sortOrder: 'asc' } },
        options: true,
        attributes: true,
        variants: { include: { optionValues: true } },
        category: true,
        brand: true,
      },
    });
  }

  // --- Queries ---

  async list(params: ProductListParams): Promise<PaginatedResult<Product>> {
    const { page, pageSize, categoryId, brandId, status, visibility, minPrice, maxPrice, inStock, isFeatured, sortBy, sortOrder, search } = params;

    const where: any = {
      deletedAt: null,
      status: status || ProductStatus.ACTIVE, // Default to active products
      visibility: visibility || { in: [Visibility.VISIBLE, Visibility.CATALOG_ONLY, Visibility.SEARCH_ONLY] }, // Default visible
    };

    if (categoryId) where.categoryId = categoryId;
    if (brandId) where.brandId = brandId;
    if (isFeatured !== undefined) where.isFeatured = isFeatured;
    if (minPrice !== undefined || maxPrice !== undefined) {
      where.price = {};
      if (minPrice !== undefined) where.price.gte = minPrice;
      if (maxPrice !== undefined) where.price.lte = maxPrice;
    }
    if (inStock) {
      where.quantity = { gt: 0 };
    }
    if (search) {
      where.OR = [
        { name: { contains: search, mode: 'insensitive' } },
        { description: { contains: search, mode: 'insensitive' } },
        { shortDescription: { contains: search, mode: 'insensitive' } },
        { sku: { contains: search, mode: 'insensitive' } },
        { barcode: { contains: search, mode: 'insensitive' } },
      ];
    }

    const [products, total] = await this.prisma.$transaction([
      this.prisma.product.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { [sortBy === 'popularity' ? 'createdAt' : sortBy]: sortOrder }, // 'popularity' is complex, default to createdAt for now
        include: { images: { take: 1, orderBy: { sortOrder: 'asc' } }, category: true, brand: true }, // Only fetch primary image for list
      }),
      this.prisma.product.count({ where }),
    ]);

    return {
      data: products,
      total,
      page,
      pageSize,
      totalPages: Math.ceil(total / pageSize),
    };
  }

  async search(query: string, params?: SearchParams): Promise<Product[]> {
    const { page = 1, pageSize = 12 } = params || {};
    return this.prisma.product.findMany({
      where: {
        deletedAt: null,
        status: ProductStatus.ACTIVE,
        visibility: { in: [Visibility.VISIBLE, Visibility.SEARCH_ONLY] },
        OR: [
          { name: { contains: query, mode: 'insensitive' } },
          { description: { contains: query, mode: 'insensitive' } },
          { shortDescription: { contains: query, mode: 'insensitive' } },
          { seoTitle: { contains: query, mode: 'insensitive' } },
          { seoDescription: { contains: query, mode: 'insensitive' } },
        ],
      },
      skip: (page - 1) * pageSize,
      take: pageSize,
      include: { images: { take: 1, orderBy: { sortOrder: 'asc' } } },
    });
  }

  async getFeatured(limit: number = 8): Promise<Product[]> {
    return this.prisma.product.findMany({
      where: {
        isFeatured: true,
        deletedAt: null,
        status: ProductStatus.ACTIVE,
        visibility: { in: [Visibility.VISIBLE, Visibility.CATALOG_ONLY] },
      },
      take: limit,
      orderBy: { createdAt: 'desc' }, // Or a custom 'featured_order' field
      include: { images: { take: 1, orderBy: { sortOrder: 'asc' } } },
    });
  }

  async getByCategory(categoryId: string, params?: ProductListParams): Promise<PaginatedResult<Product>> {
    return this.list({ ...params, categoryId });
  }

  async getRelated(productId: string, limit: number = 4): Promise<Product[]> {
    const product = await this.prisma.product.findUnique({ where: { id: productId }, select: { categoryId: true, brandId: true } });
    if (!product) return [];

    const where: any = {
      id: { not: productId },
      deletedAt: null,
      status: ProductStatus.ACTIVE,
      visibility: { in: [Visibility.VISIBLE, Visibility.CATALOG_ONLY] },
      OR: [],
    };

    if (product.categoryId) {
      where.OR.push({ categoryId: product.categoryId });
    }
    if (product.brandId) {
      where.OR.push({ brandId: product.brandId });
    }

    if (where.OR.length === 0) return []; // No category or brand to relate by

    return this.prisma.product.findMany({
      where,
      take: limit,
      orderBy: { createdAt: 'desc' }, // Simple ordering, can be improved
      include: { images: { take: 1, orderBy: { sortOrder: 'asc' } } },
    });
  }

  // --- Inventory ---

  async updateStock(id: string, quantity: number, operation: 'set' | 'increment' | 'decrement'): Promise<void> {
    const product = await this.prisma.product.findUnique({ where: { id } });
    if (!product || !product.trackInventory) {
      throw new Error('Product not found or inventory tracking is disabled.');
    }

    let newQuantity: number;
    if (operation === 'set') {
      newQuantity = quantity;
    } else if (operation === 'increment') {
      newQuantity = product.quantity + quantity;
    } else { // decrement
      newQuantity = product.quantity - quantity;
    }

    if (newQuantity < 0 && !product.allowBackorder) {
      throw new Error('Not enough stock and backorders are not allowed.');
    }

    await this.prisma.product.update({
      where: { id },
      data: { quantity: newQuantity },
    });
  }

  async checkAvailability(id: string, quantity: number): Promise<boolean> {
    const product = await this.prisma.product.findUnique({ where: { id } });
    if (!product) return false;
    if (!product.trackInventory) return true; // Always available if not tracking
    return product.quantity >= quantity || product.allowBackorder;
  }

  async reserveStock(id: string, quantity: number): Promise<string> {
    const isAvailable = await this.checkAvailability(id, quantity);
    if (!isAvailable) {
      throw new Error(`Not enough stock for product ${id}.`);
    }

    // Decrement stock immediately (optimistic locking/transaction needed in real app)
    await this.updateStock(id, quantity, 'decrement');

    const reservationId = `res_${Date.now()}_${Math.random().toString(36).substring(2, 9)}`;
    const expiresAt = new Date(Date.now() + 15 * 60 * 1000); // Reserve for 15 minutes
    productReservations.set(reservationId, { id: reservationId, productId: id, quantity, expiresAt });

    // Schedule automatic release if not confirmed (e.g., via a cron job or message queue)
    setTimeout(() => this.releaseReservation(reservationId), 15 * 60 * 1000 + 5000); // 15 mins + 5s grace

    return reservationId;
  }

  async releaseReservation(reservationId: string): Promise<void> {
    const reservation = productReservations.get(reservationId);
    if (reservation) {
      // Only release if not already processed (e.g., converted to an order)
      // In a real system, you'd check if the order associated with this reservation is still pending.
      // For this example, we just remove it.
      productReservations.delete(reservationId);
      // Revert stock if reservation was not fulfilled
      // This needs careful handling to avoid double-releasing or releasing stock already sold.
      // For simplicity, we assume if it's still in the map, it needs to be released.
      try {
        await this.updateStock(reservation.productId, reservation.quantity, 'increment');
      } catch (error) {
        console.error(`Failed to release stock for product ${reservation.productId}:`, error);
      }
    }
  }

  // --- Variants ---

  async createVariant(productId: string, data: CreateVariantInput): Promise<ProductVariant> {
    const variant = await this.prisma.productVariant.create({
      data: {
        productId,
        sku: data.sku || this.generateSku(await this.prisma.product.findUniqueOrThrow({ where: { id: productId } }), data),
        barcode: data.barcode,
        price: data.price,
        compareAtPrice: data.compareAtPrice,
        costPrice: data.costPrice,
        quantity: data.quantity,
        weight: data.weight,
        image: data.image,
        isActive: data.isActive,
        optionValues: {
          createMany: {
            data: data.optionValues,
          },
        },
      },
      include: { optionValues: true },
    });
    return variant;
  }

  async updateVariant(variantId: string, data: UpdateVariantInput): Promise<ProductVariant> {
    const updateData: any = { ...data };
    if (data.optionValues) {
      await this.prisma.productVariantOptionValue.deleteMany({ where: { variantId } });
      updateData.optionValues = { createMany: { data: data.optionValues } };
    }

    const variant = await this.prisma.productVariant.update({
      where: { id: variantId },
      data: updateData,
      include: { optionValues: true },
    });
    return variant;
  }

  async deleteVariant(variantId: string): Promise<void> {
    await this.prisma.productVariant.delete({ where: { id: variantId } });
  }

  // --- Bulk operations ---

  async bulkUpdateStatus(ids: string[], status: ProductStatus): Promise<number> {
    const { count } = await this.prisma.product.updateMany({
      where: { id: { in: ids } },
      data: { status },
    });
    return count;
  }

  async bulkDelete(ids: string[]): Promise<number> {
    // Soft delete
    const { count } = await this.prisma.product.updateMany({
      where: { id: { in: ids } },
      data: { deletedAt: new Date() },
    });
    // Or hard delete:
    // const { count } = await this.prisma.product.deleteMany({ where: { id: { in: ids } } });
    return count;
  }

  async bulkUpdatePrices(updates: Array<{ id: string; price: number }>): Promise<number> {
    const updatePromises = updates.map(item =>
      this.prisma.product.update({
        where: { id: item.id },
        data: { price: item.price },
      })
    );
    await this.prisma.$transaction(updatePromises);
    return updates.length;
  }

  // --- Helpers ---

  private async generateSlug(name: string): Promise<string> {
    let baseSlug = slugify(name, { lower: true, strict: true });
    let slug = baseSlug;
    let counter = 1;
    while (await this.prisma.product.findUnique({ where: { slug } })) {
      slug = `${baseSlug}-${counter}`;
      counter++;
    }
    return slug;
  }

  private generateSku(product: Product, variant?: Partial<CreateVariantInput>): string {
    if (variant?.sku) return variant.sku; // Use provided variant SKU if available

    const productSkuPart = product.sku || product.name.substring(0, 3).toUpperCase();
    const variantSkuPart = variant?.optionValues
      ? variant.optionValues.map(ov => ov.optionValue.substring(0, 2).toUpperCase()).join('-')
      : '';

    return `${productSkuPart}${variantSkuPart ? `-${variantSkuPart}` : ''}-${Math.random().toString(36).substring(2, 6).toUpperCase()}`;
  }
}
```

---

## 3. `src/server/services/category-service.ts`

```typescript
import { PrismaClient, Category } from '@prisma/client';
import slugify from 'slugify';
import { z } from 'zod';

// --- DTOs ---

export const CreateCategoryInput = z.object({
  name: z.string().min(1).max(255),
  slug: z.string().optional(),
  description: z.string().optional(),
  imageUrl: z.string().url().optional(),
  seoTitle: z.string().max(160).optional(),
  seoDescription: z.string().max(300).optional(),
  parentId: z.string().cuid().optional().nullable(),
});
export type CreateCategoryInput = z.infer<typeof CreateCategoryInput>;

export const UpdateCategoryInput = CreateCategoryInput.partial();
export type UpdateCategoryInput = z.infer<typeof UpdateCategoryInput>;

export type CategoryNode = Category & {
  children?: CategoryNode[];
  productCount?: number; // Optional for tree views
};

// --- Category Service ---

export class CategoryService {
  constructor(private prisma: PrismaClient) {}

  // --- CRUD ---

  async create(data: CreateCategoryInput): Promise<Category> {
    const slug = data.slug || (await this.generateSlug(data.name));
    return this.prisma.category.create({
      data: {
        ...data,
        slug,
      },
    });
  }

  async update(id: string, data: UpdateCategoryInput): Promise<Category> {
    const slug = data.name ? await this.generateSlug(data.name) : data.slug;
    return this.prisma.category.update({
      where: { id },
      data: { ...data, slug },
    });
  }

  async delete(id: string): Promise<void> {
    // Check if category has children or products before deleting
    const childrenCount = await this.prisma.category.count({ where: { parentId: id } });
    if (childrenCount > 0) {
      throw new Error('Cannot delete category with subcategories. Please reassign or delete subcategories first.');
    }
    const productCount = await this.prisma.product.count({ where: { categoryId: id } });
    if (productCount > 0) {
      // Option 1: Throw error
      throw new Error('Cannot delete category with associated products. Please reassign or delete products first.');
      // Option 2: Disconnect products (set categoryId to null)
      // await this.prisma.product.updateMany({ where: { categoryId: id }, data: { categoryId: null } });
    }

    await this.prisma.category.delete({ where: { id } });
  }

  async getById(id: string): Promise<Category | null> {
    return this.prisma.category.findUnique({ where: { id } });
  }

  async getBySlug(slug: string): Promise<Category | null> {
    return this.prisma.category.findUnique({ where: { slug } });
  }

  // --- Tree operations ---

  async getTree(includeProductCount: boolean = false): Promise<CategoryNode[]> {
    const categories = await this.prisma.category.findMany({
      orderBy: { name: 'asc' },
      include: includeProductCount ? { _count: { select: { products: true } } } : undefined,
    });

    const categoryMap = new Map<string, CategoryNode>(
      categories.map(cat => [cat.id, { ...cat, children: [], productCount: includeProductCount ? cat._count.products : undefined }])
    );

    const rootCategories: CategoryNode[] = [];
    categoryMap.forEach(category => {
      if (category.parentId && categoryMap.has(category.parentId)) {
        categoryMap.get(category.parentId)?.children?.push(category);
      } else {
        rootCategories.push(category);
      }
    });

    return rootCategories;
  }

  async getChildren(parentId: string): Promise<Category[]> {
    return this.prisma.category.findMany({
      where: { parentId },
      orderBy: { name: 'asc' },
    });
  }

  async getAncestors(categoryId: string): Promise<Category[]> {
    let currentCategory = await this.prisma.category.findUnique({ where: { id: categoryId } });
    const ancestors: Category[] = [];

    while (currentCategory?.parentId) {
      const parent = await this.prisma.category.findUnique({ where: { id: currentCategory.parentId } });
      if (parent) {
        ancestors.unshift(parent); // Add to the beginning to maintain order from root to parent
        currentCategory = parent;
      } else {
        break;
      }
    }
    return ancestors;
  }

  async getDescendants(categoryId: string): Promise<Category[]> {
    const descendants: Category[] = [];
    const queue: string[] = [categoryId];

    while (queue.length > 0) {
      const currentId = queue.shift()!;
      const children = await this.prisma.category.findMany({ where: { parentId: currentId } });
      for (const child of children) {
        descendants.push(child);
        queue.push(child.id);
      }
    }
    return descendants;
  }

  async move(categoryId: string, newParentId: string | null): Promise<void> {
    // Prevent moving a category into itself or one of its descendants
    if (newParentId === categoryId) {
      throw new Error('Cannot move a category into itself.');
    }
    if (newParentId) {
      const descendants = await this.getDescendants(categoryId);
      if (descendants.some(d => d.id === newParentId)) {
        throw new Error('Cannot move a category into one of its descendants.');
      }
    }

    await this.prisma.category.update({
      where: { id: categoryId },
      data: { parentId: newParentId },
    });
  }

  // --- Product counts ---

  async getProductCount(categoryId: string, includeChildren: boolean = false): Promise<number> {
    if (!includeChildren) {
      const category = await this.prisma.category.findUnique({
        where: { id: categoryId },
        include: { _count: { select: { products: true } } },
      });
      return category?._count.products || 0;
    } else {
      const descendantIds = (await this.getDescendants(categoryId)).map(c => c.id);
      const allCategoryIds = [categoryId, ...descendantIds];
      return this.prisma.product.count({
        where: {
          categoryId: { in: allCategoryIds },
          deletedAt: null,
          status: ProductStatus.ACTIVE,
          visibility: { in: [Visibility.VISIBLE, Visibility.CATALOG_ONLY] },
        },
      });
    }
  }

  // --- Helpers ---

  private async generateSlug(name: string): Promise<string> {
    let baseSlug = slugify(name, { lower: true, strict: true });
    let slug = baseSlug;
    let counter = 1;
    while (await this.prisma.category.findUnique({ where: { slug } })) {
      slug = `${baseSlug}-${counter}`;
      counter++;
    }
    return slug;
  }
}
```

---

## 4. `src/server/trpc/routers/products.ts`

```typescript
import { z } from 'zod';
import { createTRPCRouter, publicProcedure, adminProcedure } from '../trpc'; // Assuming these are defined
import { ProductService, CreateProductInput, UpdateProductInput, ProductListParams, SearchParams, CreateVariantInput, UpdateVariantInput } from '../../services/product-service';
import { prisma } from '../../db'; // Assuming prisma client is initialized and exported
import { createProductSchema, updateProductSchema, productListSchema, paginationSchema, createVariantSchema, updateVariantSchema } from '../../../lib/validations/product';
import { ProductStatus } from '@prisma/client';

export const productsRouter = createTRPCRouter({
  // --- Queries ---

  list: publicProcedure
    .input(productListSchema)
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.list(input);
    }),

  getById: publicProcedure
    .input(z.object({ id: z.string().cuid() }))
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.getById(input.id);
    }),

  getBySlug: publicProcedure
    .input(z.object({ slug: z.string() }))
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.getBySlug(input.slug);
    }),

  search: publicProcedure
    .input(z.object({ query: z.string(), ...paginationSchema.shape }))
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.search(input.query, { page: input.page, pageSize: input.pageSize });
    }),

  getFeatured: publicProcedure
    .input(z.object({ limit: z.number().int().min(1).default(8) }))
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.getFeatured(input.limit);
    }),

  getByCategory: publicProcedure
    .input(productListSchema.extend({ categoryId: z.string().cuid() })) // categoryId is required here
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.getByCategory(input.categoryId, input);
    }),

  getRelated: publicProcedure
    .input(z.object({ productId: z.string().cuid(), limit: z.number().int().min(1).default(4) }))
    .query(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.getRelated(input.productId, input.limit);
    }),

  // --- Mutations (protected - admin only) ---

  create: adminProcedure
    .input(createProductSchema)
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      return productService.create(input as CreateProductInput); // Cast to internal DTO type
    }),

  update: adminProcedure
    .input(updateProductSchema.extend({ id: z.string().cuid() }))
    .mutation(async ({ input }) => {
      const { id, ...data } = input;
      const productService = new ProductService(prisma);
      return productService.update(id, data as UpdateProductInput); // Cast to internal DTO type
    }),

  delete: adminProcedure
    .input(z.object({ id: z.string().cuid() }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      await productService.delete(input.id);
      return { success: true };
    }),

  // --- Inventory Mutations ---

  updateStock: adminProcedure
    .input(z.object({
      id: z.string().cuid(),
      quantity: z.number().int(),
      operation: z.enum(['set', 'increment', 'decrement']),
    }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      await productService.updateStock(input.id, input.quantity, input.operation);
      return { success: true };
    }),

  // --- Variant Mutations ---

  createVariant: adminProcedure
    .input(createVariantSchema.extend({ productId: z.string().cuid() }))
    .mutation(async ({ input }) => {
      const { productId, ...data } = input;
      const productService = new ProductService(prisma);
      return productService.createVariant(productId, data as CreateVariantInput);
    }),

  updateVariant: adminProcedure
    .input(updateVariantSchema.extend({ id: z.string().cuid() }))
    .mutation(async ({ input }) => {
      const { id, ...data } = input;
      const productService = new ProductService(prisma);
      return productService.updateVariant(id, data as UpdateVariantInput);
    }),

  deleteVariant: adminProcedure
    .input(z.object({ id: z.string().cuid() }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      await productService.deleteVariant(input.id);
      return { success: true };
    }),

  // --- Bulk Operations ---

  bulkUpdateStatus: adminProcedure
    .input(z.object({
      ids: z.array(z.string().cuid()).min(1),
      status: z.nativeEnum(ProductStatus),
    }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      const count = await productService.bulkUpdateStatus(input.ids, input.status);
      return { count, success: true };
    }),

  bulkDelete: adminProcedure
    .input(z.object({ ids: z.array(z.string().cuid()).min(1) }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      const count = await productService.bulkDelete(input.ids);
      return { count, success: true };
    }),

  bulkUpdatePrices: adminProcedure
    .input(z.object({
      updates: z.array(z.object({
        id: z.string().cuid(),
        price: z.number().positive(),
      })).min(1),
    }))
    .mutation(async ({ input }) => {
      const productService = new ProductService(prisma);
      const count = await productService.bulkUpdatePrices(input.updates);
      return { count, success: true };
    }),
});
```

---

## 5. `src/lib/validations/product.ts`

```typescript
import { z } from 'zod';
import { ProductStatus, Visibility, AttributeType } from '@prisma/client';

// --- Reusable Schemas ---

export const paginationSchema = z.object({
  page: z.number().int().min(1).default(1),
  pageSize: z.number().int().min(1).max(100).default(12),
});

export const productImageSchema = z.object({
  url: z.string().url('Invalid image URL'),
  altText: z.string().optional(),
  sortOrder: z.number().int().min(0).optional(),
});

export const productOptionSchema = z.object({
  name: z.string().min(1).max(50),
  values: z.array(z.string().min(1)).min(1, 'At least one option value is required'),
});

export const productAttributeSchema = z.object({
  name: z.string().min(1).max(50),
  value: z.string().min(1),
  type: z.nativeEnum(AttributeType).default(AttributeType.TEXT).optional(),
});

export const productVariantOptionValueSchema = z.object({
  optionName: z.string().min(1),
  optionValue: z.string().min(1),
});

// --- Product Schemas ---

export const createProductSchema = z.object({
  name: z.string().min(1, 'Product name is required').max(255, 'Product name is too long'),
  slug: z.string().optional(), // Will be generated if not provided
  description: z.string().optional(),
  shortDescription: z.string().max(500, 'Short description is too long').optional(),

  price: z.number().positive('Price must be a positive number'),
  compareAtPrice: z.number().positive().optional(),
  costPrice: z.number().positive().optional(),

  status: z.nativeEnum(ProductStatus).default(ProductStatus.DRAFT).optional(),
  visibility: z.nativeEnum(Visibility).default(Visibility.VISIBLE).optional(),
  isFeatured: z.boolean().default(false).optional(),
  isDigital: z.boolean().default(false).optional(),

  sku: z.string().optional(),
  barcode: z.string().optional(),
  quantity: z.number().int().min(0, 'Quantity cannot be negative').default(0).optional(),
  trackInventory: z.boolean().default(true).optional(),
  allowBackorder: z.boolean().default(false).optional(),
  lowStockThreshold: z.number().int().min(0).default(5).optional(),

  weight: z.number().positive().optional(),
  width: z.number().positive().optional(),
  height: z.number().positive().optional(),
  depth: z.number().positive().optional(),

  seoTitle: z.string().max(160).optional(),
  seoDescription: z.string().max(300).optional(),

  categoryId: z.string().cuid('Invalid category ID').optional().nullable(),
  brandId: z.string().cuid('Invalid brand ID').optional().nullable(),

  images: z.array(productImageSchema).optional(),
  options: z.array(productOptionSchema).optional(),
  attributes: z.array(productAttributeSchema).optional(),
});

export const updateProductSchema = createProductSchema.partial().extend({
  // ID is typically passed as a path parameter or separate field in tRPC mutation
  // id: z.string().cuid(), // Not part of the input object for update in tRPC example
});

export const productListSchema = paginationSchema.extend({
  categoryId: z.string().cuid('Invalid category ID').optional(),
  brandId: z.string().cuid('Invalid brand ID').optional(),
  status: z.nativeEnum(ProductStatus).optional(),
  visibility: z.nativeEnum(Visibility).optional(),
  minPrice: z.number().positive().optional(),
  maxPrice: z.number().positive().optional(),
  inStock: z.boolean().optional(),
  isFeatured: z.boolean().optional(),
  sortBy: z.enum(['price', 'name', 'createdAt', 'updatedAt', 'popularity']).default('createdAt'),
  sortOrder: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().optional(),
});

// --- Variant Schemas ---

export const createVariantSchema = z.object({
  sku: z.string().optional(),
  barcode: z.string().optional(),
  price: z.number().positive('Variant price must be positive'),
  compareAtPrice: z.number().positive().optional(),
  costPrice: z.number().positive().optional(),
  quantity: z.number().int().min(0, 'Variant quantity cannot be negative').default(0),
  weight: z.number().positive().optional(),
  image: z.string().url('Invalid image URL').optional(),
  isActive: z.boolean().default(true).optional(),
  optionValues: z.array(productVariantOptionValueSchema).min(1, 'At least one option value is required for a variant'),
});

export const updateVariantSchema = createVariantSchema.partial();

// --- Category Schemas ---

export const createCategorySchema = z.object({
  name: z.string().min(1, 'Category name is required').max(255, 'Category name is too long'),
  slug: z.string().optional(),
  description: z.string().optional(),
  imageUrl: z.string().url('Invalid image URL').optional(),
  seoTitle: z.string().max(160).optional(),
  seoDescription: z.string().max(300).optional(),
  parentId: z.string().cuid('Invalid parent category ID').optional().nullable(),
});

export const updateCategorySchema = createCategorySchema.partial();

export const categoryListSchema = paginationSchema.extend({
  parentId: z.string().cuid('Invalid parent category ID').optional().nullable(),
  search: z.string().optional(),
});
```

---

## 6. `src/hooks/use-products.ts`

```typescript
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Product, ProductStatus } from '@prisma/client';
import { CreateProductInput, UpdateProductInput, ProductListParams, PaginatedResult, CreateVariantInput, UpdateVariantInput } from '../server/services/product-service';
import { trpc } from '../utils/trpc'; // Assuming trpc client is initialized

// --- Product Queries ---

export function useProducts(params?: ProductListParams) {
  return trpc.products.list.useQuery(params || {});
}

export function useProduct(id: string | undefined) {
  return trpc.products.getById.useQuery({ id }, {
    enabled: !!id, // Only run query if id is available
  });
}

export function useProductBySlug(slug: string | undefined) {
  return trpc.products.getBySlug.useQuery({ slug }, {
    enabled: !!slug,
  });
}

export function useFeaturedProducts(limit?: number) {
  return trpc.products.getFeatured.useQuery({ limit });
}

export function useProductSearch(query: string, params?: { page?: number; pageSize?: number }) {
  return trpc.products.search.useQuery({ query, ...params }, {
    enabled: !!query && query.length > 2, // Only search with a meaningful query
  });
}

export function useProductsByCategory(categoryId: string, params?: ProductListParams) {
  return trpc.products.getByCategory.useQuery({ ...params, categoryId }, {
    enabled: !!categoryId,
  });
}

export function useRelatedProducts(productId: string, limit?: number) {
  return trpc.products.getRelated.useQuery({ productId, limit }, {
    enabled: !!productId,
  });
}

// --- Product Mutations ---

export function useCreateProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateProductInput) => trpc.products.create.mutate(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}

export function useUpdateProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateProductInput }) => trpc.products.update.mutate({ id, ...data }),
    onSuccess: (updatedProduct) => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getById', { id: updatedProduct.id }] });
      queryClient.invalidateQueries({ queryKey: ['products.getBySlug', { slug: updatedProduct.slug }] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}

export function useDeleteProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => trpc.products.delete.mutate({ id }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}

export function useUpdateProductStock() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, quantity, operation }: { id: string; quantity: number; operation: 'set' | 'increment' | 'decrement' }) =>
      trpc.products.updateStock.mutate({ id, quantity, operation }),
    onSuccess: (_, { id }) => {
      queryClient.invalidateQueries({ queryKey: ['products.getById', { id }] });
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
    },
  });
}

// --- Variant Mutations ---

export function useCreateProductVariant() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ productId, data }: { productId: string; data: CreateVariantInput }) =>
      trpc.products.createVariant.mutate({ productId, ...data }),
    onSuccess: (newVariant) => {
      queryClient.invalidateQueries({ queryKey: ['products.getById', { id: newVariant.productId }] });
    },
  });
}

export function useUpdateProductVariant() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ id, data }: { id: string; data: UpdateVariantInput }) =>
      trpc.products.updateVariant.mutate({ id, ...data }),
    onSuccess: (updatedVariant) => {
      queryClient.invalidateQueries({ queryKey: ['products.getById', { id: updatedVariant.productId }] });
    },
  });
}

export function useDeleteProductVariant() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (id: string) => trpc.products.deleteVariant.mutate({ id }),
    onSuccess: () => {
      // Invalidate product detail query for the parent product
      // This requires knowing the parent product ID, which might need to be passed or fetched.
      // For simplicity, we'll just invalidate general product lists.
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
    },
  });
}

// --- Bulk Operations ---

export function useBulkUpdateProductStatus() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: ({ ids, status }: { ids: string[]; status: ProductStatus }) =>
      trpc.products.bulkUpdateStatus.mutate({ ids, status }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}

export function useBulkDeleteProducts() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (ids: string[]) => trpc.products.bulkDelete.mutate({ ids }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}

export function useBulkUpdateProductPrices() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (updates: Array<{ id: string; price: number }>) =>
      trpc.products.bulkUpdatePrices.mutate({ updates }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products.list'] });
      queryClient.invalidateQueries({ queryKey: ['products.getFeatured'] });
    },
  });
}
```

---

## 7. `src/components/products/product-card.tsx`

```tsx
import React from 'react';
import Image from 'next/image';
import Link from 'next/link';
import { Product } from '@prisma/client';
import { StarIcon } from '@heroicons/react/20/solid'; // Example icon
import { formatCurrency } from '@/utils/formatters'; // Assuming a utility for currency formatting

interface ProductCardProps {
  product: Product & {
    images: { url: string; altText: string | null }[];
    reviews?: { rating: number }[]; // Placeholder for actual review data
  };
  onAddToCart?: (productId: string) => void;
  onAddToWishlist?: (productId: string) => void;
}

const ProductCard: React.FC<ProductCardProps> = ({ product, onAddToCart, onAddToWishlist }) => {
  const mainImage = product.images[0]?.url || '/placeholder-product.png';
  const averageRating = product.reviews && product.reviews.length > 0
    ? product.reviews.reduce((acc, review) => acc + review.rating, 0) / product.reviews.length
    : 0;
  const isOnSale = product.compareAtPrice && product.price < product.compareAtPrice;
  const isNew = new Date(product.createdAt) > new Date(Date.now() - 7 * 24 * 60 * 60 * 1000); // Product created in last 7 days
  const isFeatured = product.isFeatured;

  return (
    <div className="group relative flex flex-col overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm transition-shadow hover:shadow-md">
      <Link href={`/products/${product.slug}`} className="relative aspect-square w-full overflow-hidden bg-gray-200">
        <Image
          src={mainImage}
          alt={product.images[0]?.altText || product.name}
          fill
          sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
          className="object-cover object-center transition-transform duration-300 group-hover:scale-105"
          priority={isFeatured} // Prioritize loading for featured products
        />
        <div className="absolute top-2 left-2 flex flex-wrap gap-1">
          {isOnSale && (
            <span className="inline-flex items-center rounded-full bg-red-500 px-2.5 py-0.5 text-xs font-medium text-white">
              Sale
            </span>
          )}
          {isNew && (
            <span className="inline-flex items-center rounded-full bg-blue-500 px-2.5 py-0.5 text-xs font-medium text-white">
              New
            </span>
          )}
          {isFeatured && (
            <span className="inline-flex items-center rounded-full bg-green-500 px-2.5 py-0.5 text-xs font-medium text-white">
              Featured
            </span>
          )}
        </div>
      </Link>
      <div className="flex flex-1 flex-col p-4">
        <h3 className="text-sm font-medium text-gray-900">
          <Link href={`/products/${product.slug}`}>
            <span aria-hidden="true" className="absolute inset-0" />
            {product.name}
          </Link>
        </h3>
        {product.shortDescription && (
          <p className="mt-1 text-sm text-gray-500 line-clamp-2">{product.shortDescription}</p>
        )}
        <div className="mt-auto flex items-center justify-between pt-2">
          <div className="flex items-baseline gap-x-1">
            <p className="text-lg font-bold text-gray-900">{formatCurrency(product.price)}</p>
            {isOnSale && product.compareAtPrice && (
              <p className="text-sm text-gray-500 line-through">{formatCurrency(product.compareAtPrice)}</p>
            )}
          </div>
          {product.reviews && product.reviews.length > 0 && (
            <div className="flex items-center">
              {[0, 1, 2, 3, 4].map((rating) => (
                <StarIcon
                  key={rating}
                  className={`${averageRating > rating ? 'text-yellow-400' : 'text-gray-200'} h-4 w-4 flex-shrink-0`}
                  aria-hidden="true"
                />
              ))}
              <span className="ml-1 text-xs text-gray-500">({product.reviews.length})</span>
            </div>
          )}
        </div>
        <div className="mt-4 flex gap-2">
          {onAddToCart && (
            <button
              onClick={() => onAddToCart(product.id)}
              className="flex-1 rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
            >
              Add to cart
            </button>
          )}
          {onAddToWishlist && (
            <button
              onClick={() => onAddToWishlist(product.id)}
              className="rounded-md border border-gray-300 bg-white p-2 text-gray-400 shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
              aria-label="Add to wishlist"
            >
              {/* Replace with a HeartIcon or similar */}
              <svg className="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" />
              </svg>
            </button>
          )}
        </div>
      </div>
    </div>
  );
};

export default ProductCard;
```

---

## 8. `src/components/products/product-grid.tsx`

```tsx
import React from 'react';
import { Product } from '@prisma/client';
import ProductCard from './product-card';

interface ProductGridProps {
  products: (Product & { images: { url: string; altText: string | null }[] })[];
  isLoading: boolean;
  error: Error | null;
  emptyMessage?: string;
  skeletonCount?: number;
  onAddToCart?: (productId: string) => void;
  onAddToWishlist?: (productId: string) => void;
}

const ProductGrid: React.FC<ProductGridProps> = ({
  products,
  isLoading,
  error,
  emptyMessage = 'No products found.',
  skeletonCount = 8,
  onAddToCart,
  onAddToWishlist,
}) => {
  if (error) {
    return (
      <div className="text-center text-red-600 p-8">
        <p>Error loading products: {error.message}</p>
      </div>
    );
  }

  if (isLoading) {
    return (
      <div className="grid grid-cols-1 gap-y-10 gap-x-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
        {Array.from({ length: skeletonCount }).map((_, i) => (
          <div key={i} className="group relative flex flex-col overflow-hidden rounded-lg border border-gray-200 bg-white shadow-sm animate-pulse">
            <div className="aspect-square w-full bg-gray-300"></div>
            <div className="flex flex-1 flex-col p-4">
              <div className="h-4 bg-gray-300 rounded w-3/4 mb-2"></div>
              <div className="h-3 bg-gray-300 rounded w-1/2 mb-4"></div>
              <div className="mt-auto flex items-center justify-between pt-2">
                <div className="h-4 bg-gray-300 rounded w-1/4"></div>
                <div className="h-4 bg-gray-300 rounded w-1/4"></div>
              </div>
              <div className="mt-4 h-10 bg-gray-300 rounded"></div>
            </div>
          </div>
        ))}
      </div>
    );
  }

  if (!products || products.length === 0) {
    return (
      <div className="text-center text-gray-500 p-8">
        <p>{emptyMessage}</p>
      </div>
    );
  }

  return (
    <div className="grid grid-cols-1 gap-y-10 gap-x-6 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4">
      {products.map((product) => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={onAddToCart}
          onAddToWishlist={onAddToWishlist}
        />
      ))}
    </div>
  );
};

export default ProductGrid;
```

---

## 9. `src/components/products/product-filters.tsx`

```tsx
import React, { useState, useEffect, Fragment } from 'react';
import { Disclosure, Transition } from '@headlessui/react';
import { ChevronDownIcon, FunnelIcon } from '@heroicons/react/20/solid';
import { ProductListParams } from '@/server/services/product-service';
import { Category } from '@prisma/client';
import { useCategoriesTree } from '@/hooks/use-categories'; // Assuming a hook for categories

interface ProductFiltersProps {
  currentFilters: ProductListParams;
  onApplyFilters: (filters: Partial<ProductListParams>) => void;
  brands: { id: string; name: string }[]; // Example: fetched from a brand service
}

const ProductFilters: React.FC<ProductFiltersProps> = ({
  currentFilters,
  onApplyFilters,
  brands,
}) => {
  const [localFilters, setLocalFilters] = useState<Partial<ProductListParams>>(currentFilters);
  const { data: categoriesTree, isLoading: isLoadingCategories } = useCategoriesTree();

  useEffect(() => {
    setLocalFilters(currentFilters);
  }, [currentFilters]);

  const handleFilterChange = (key: keyof ProductListParams, value: any) => {
    setLocalFilters(prev => ({ ...prev, [key]: value }));
  };

  const handlePriceRangeChange = (min: number | undefined, max: number | undefined) => {
    setLocalFilters(prev => ({ ...prev, minPrice: min, maxPrice: max }));
  };

  const handleCategoryClick = (categoryId: string | undefined) => {
    handleFilterChange('categoryId', categoryId === localFilters.categoryId ? undefined : categoryId);
  };

  const handleBrandToggle = (brandId: string) => {
    // For simplicity, assuming single brand selection or a simple toggle
    handleFilterChange('brandId', brandId === localFilters.brandId ? undefined : brandId);
  };

  const handleApply = () => {
    onApplyFilters(localFilters);
  };

  const handleClear = () => {
    const clearedFilters: Partial<ProductListParams> = {
      page: 1,
      pageSize: currentFilters.pageSize, // Keep current page size
      sortBy: currentFilters.sortBy,
      sortOrder: currentFilters.sortOrder,
    };
    setLocalFilters(clearedFilters);
    onApplyFilters(clearedFilters);
  };

  const renderCategoryTree = (categories: Category[], level: number = 0) => (
    <ul className={`space-y-1 ${level > 0 ? 'ml-4' : ''}`}>
      {categories.map(category => (
        <li key={category.id}>
          <button
            onClick={() => handleCategoryClick(category.id)}
            className={`block w-full text-left text-sm ${localFilters.categoryId === category.id ? 'font-semibold text-indigo-600' : 'text-gray-700 hover:text-gray-900'}`}
          >
            {category.name}
          </button>
          {category.children && category.children.length > 0 && (
            <Disclosure as="div" className="pt-2">
              {({ open }) => (
                <>
                  <Disclosure.Button className="flex w-full items-center justify-between text-sm text-gray-400 hover:text-gray-500">
                    <span className="sr-only">Toggle subcategories</span>
                    <ChevronDownIcon
                      className={`h-5 w-5 ${open ? 'rotate-180 transform' : ''}`}
                      aria-hidden="true"
                    />
                  </Disclosure.Button>
                  <Transition
                    as={Fragment}
                    enter="transition ease-out duration-100"
                    enterFrom="transform opacity-0 scale-95"
                    enterTo="transform opacity-100 scale-100"
                    leave="transition ease-in duration-75"
                    leaveFrom="transform opacity-100 scale-100"
                    leaveTo="transform opacity-0 scale-95"
                  >
                    <Disclosure.Panel className="pt-2">
                      {renderCategoryTree(category.children as Category[], level + 1)}
                    </Disclosure.Panel>
                  </Transition>
                </>
              )}
            </Disclosure>
          )}
        </li>
      ))}
    </ul>
  );

  return (
    <div className="lg:sticky lg:top-20 lg:self-start lg:w-64 p-4 bg-white rounded-lg shadow-sm">
      <h2 className="text-lg font-bold text-gray-900 mb-4 flex items-center gap-2">
        <FunnelIcon className="h-6 w-6 text-gray-700" /> Filters
      </h2>

      <div className="space-y-6">
        {/* Categories Filter */}
        <div>
          <h3 className="text-sm font-semibold text-gray-900 mb-2">Categories</h3>
          {isLoadingCategories ? (
            <div className="animate-pulse space-y-2">
              <div className="h-4 bg-gray-200 rounded w-3/4"></div>
              <div className="h-4 bg-gray-200 rounded w-1/2 ml-4"></div>
              <div className="h-4 bg-gray-200 rounded w-2/3"></div>
            </div>
          ) : (
            categoriesTree && categoriesTree.length > 0 ? renderCategoryTree(categoriesTree as Category[]) : <p className="text-sm text-gray-500">No categories available.</p>
          )}
        </div>

        {/* Price Range Filter */}
        <div>
          <h3 className="text-sm font-semibold text-gray-900 mb-2">Price Range</h3>
          <div className="flex items-center space-x-2">
            <input
              type="number"
              placeholder="Min"
              value={localFilters.minPrice || ''}
              onChange={(e) => handlePriceRangeChange(e.target.value ? parseFloat(e.target.value) : undefined, localFilters.maxPrice)}
              className="w-1/2 rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
            />
            <span>-</span>
            <input
              type="number"
              placeholder="Max"
              value={localFilters.maxPrice || ''}
              onChange={(e) => handlePriceRangeChange(localFilters.minPrice, e.target.value ? parseFloat(e.target.value) : undefined)}
              className="w-1/2 rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm"
            />
          </div>
        </div>

        {/* Brands Filter */}
        <div>
          <h3 className="text-sm font-semibold text-gray-900 mb-2">Brands</h3>
          <div className="space-y-2">
            {brands.map((brand) => (
              <div key={brand.id} className="flex items-center">
                <input
                  id={`brand-${brand.id}`}
                  name="brand"
                  type="checkbox"
                  checked={localFilters.brandId === brand.id}
                  onChange={() => handleBrandToggle(brand.id)}
                  className="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
                />
                <label htmlFor={`brand-${brand.id}`} className="ml-3 text-sm text-gray-600">
                  {brand.name}
                </label>
              </div>
            ))}
          </div>
        </div>

        {/* In Stock Toggle */}
        <div>
          <h3 className="text-sm font-semibold text-gray-900 mb-2">Availability</h3>
          <div className="flex items-center">
            <input
              id="in-stock"
              name="in-stock"
              type="checkbox"
              checked={!!localFilters.inStock}
              onChange={(e) => handleFilterChange('inStock', e.target.checked)}
              className="h-4 w-4 rounded border-gray-300 text-indigo-600 focus:ring-indigo-500"
            />
            <label htmlFor="in-stock" className="ml-3 text-sm text-gray-600">
              In Stock
            </label>
          </div>
        </div>

        {/* Sort Options */}
        <div>
          <h3 className="text-sm font-semibold text-gray-900 mb-2">Sort By</h3>
          <select
            value={localFilters.sortBy}
            onChange={(e) => handleFilterChange('sortBy', e.target.value as ProductListParams['sortBy'])}
            className="mt-1 block w-full rounded-md border-gray-300 py-2 pl-3 pr-10 text-base focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm"
          >
            <option value="createdAt">Newest</option>
            <option value="price">Price</option>
            <option value="name">Name</option>
            <option value="popularity">Popularity</option>
          </select>
          <select
            value={localFilters.sortOrder}
            onChange={(e) => handleFilterChange('sortOrder', e.target.value as ProductListParams['sortOrder'])}
            className="mt-2 block w-full rounded-md border-gray-300 py-2 pl-3 pr-10 text-base focus:border-indigo-500 focus:outline-none focus:ring-indigo-500 sm:text-sm"
          >
            <option value="desc">Descending</option>
            <option value="asc">Ascending</option>
          </select>
        </div>
      </div>

      <div className="mt-6 flex justify-between gap-2">
        <button
          onClick={handleApply}
          className="flex-1 rounded-md border border-transparent bg-indigo-600 px-4 py-2 text-sm font-medium text-white shadow-sm hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          Apply Filters
        </button>
        <button
          onClick={handleClear}
          className="flex-1 rounded-md border border-gray-300 bg-white px-4 py-2 text-sm font-medium text-gray-700 shadow-sm hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          Clear All
        </button>
      </div>
    </div>
  );
};

export default ProductFilters;
```

---

## 10. `src/components/products/product-detail.tsx`

```tsx
'use client';

import React, { useState, Fragment } from 'react';
import Image from 'next/image';
import { Product, ProductImage, ProductOption, ProductVariant, ProductVariantOptionValue } from '@prisma/client';
import { Tab } from '@headlessui/react';
import { StarIcon } from '@heroicons/react/20/solid';
import { formatCurrency } from '@/utils/formatters'; // Assuming this utility exists

// Helper to combine Product and its relations for full detail view
type ProductWithDetails = Product & {
  images: ProductImage[];
  options: ProductOption[];
  variants: (ProductVariant & { optionValues: ProductVariantOptionValue[] })[];
  category: { name: string; slug: string } | null;
  brand: { name: string; slug: string } | null;
  reviews?: { rating: number; title?: string; comment?: string; createdAt: Date }[]; // Placeholder
};

interface ProductDetailProps {
  product: ProductWithDetails;
  onAddToCart?: (productId: string, quantity: number, variantId?: string) => void;
}

function classNames(...classes: string[]) {
  return classes.filter(Boolean).join(' ');
}

const ProductDetail: React.FC<ProductDetailProps> = ({ product, onAddToCart }) => {
  const [selectedImage, setSelectedImage] = useState(product.images[0]?.url || '/placeholder-product.png');
  const [selectedOptions, setSelectedOptions] = useState<Record<string, string>>({});
  const [quantity, setQuantity] = useState(1);

  const availableVariants = product.variants.filter(v => v.isActive && v.quantity > 0);

  // Determine the currently selected variant based on selected options
  const selectedVariant = availableVariants.find(variant =>
    variant.optionValues.every(ov => selectedOptions[ov.optionName] === ov.optionValue)
  );

  const displayPrice = selectedVariant?.price || product.price;
  const displayCompareAtPrice = selectedVariant?.compareAtPrice || product.compareAtPrice;
  const displayQuantity = selectedVariant?.quantity || product.quantity;
  const isOutOfStock = displayQuantity <= 0 && !product.allowBackorder;

  const handleOptionChange = (optionName: string, value: string) => {
    setSelectedOptions(prev => ({ ...prev, [optionName]: value }));
  };

  const handleAddToCart = () => {
    if (onAddToCart && !isOutOfStock) {
      onAddToCart(product.id, quantity, selectedVariant?.id);
    }
  };

  const averageRating = product.reviews && product.reviews.length > 0
    ? product.reviews.reduce((acc, review) => acc + review.rating, 0) / product.reviews.length
    : 0;

  return (
    <div className="bg-white">
      <div className="mx-auto max-w-2xl px-4 py-16 sm:px-6 sm:py-24 lg:max-w-7xl lg:px-8">
        <div className="lg:grid lg:grid-cols-2 lg:items-start lg:gap-x-8">
          {/* Image gallery */}
          <Tab.Group as="div" className="flex flex-col-reverse">
            {/* Image selector */}
            <div className="mx-auto mt-6 hidden w-full max-w-2xl sm:block lg:max-w-none">
              <Tab.List className="grid grid-cols-4 gap-6">
                {product.images.map((image) => (
                  <Tab
                    key={image.id}
                    className="relative flex h-24 cursor-pointer items-center justify-center rounded-md bg-white text-sm font-medium uppercase text-gray-900 hover:bg-gray-50 focus:outline-none focus:ring focus:ring-opacity-50 focus:ring-offset-4"
                    onClick={() => setSelectedImage(image.url)}
                  >
                    {({ selected }) => (
                      <>
                        <span className="sr-only"> {image.altText} </span>
                        <span className="absolute inset-0 overflow-hidden rounded-md">
                          <Image src={image.url} alt={image.altText || product.name} fill className="object-cover object-center" />
                        </span>
                        <span
                          className={classNames(
                            selected ? 'ring-indigo-500' : 'ring-transparent',
                            'pointer-events-none absolute inset-0 rounded-md ring-2 ring-offset-2'
                          )}
                          aria-hidden="true"
                        />
                      </>
                    )}
                  </Tab>
                ))}
              </Tab.List>
            </div>

            <Tab.Panels className="aspect-h-1 aspect-w-1 w-full">
              <Tab.Panel>
                <div className="relative h-full w-full">
                  <Image
                    src={selectedImage}
                    alt={product.name}
                    fill
                    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
                    className="h-full w-full object-cover object-center sm:rounded-lg"
                  />
                </div>
              </Tab.Panel>
            </Tab.Panels>
          </Tab.Group>

          {/* Product info */}
          <div className="mt-10 px-4 sm:mt-16 sm:px-0 lg:mt-0">
            <h1 className="text-3xl font-bold tracking-tight text-gray-900">{product.name}</h1>

            <div className="mt-3">
              <h2 className="sr-only">Product information</h2>
              <p className="text-3xl tracking-tight text-gray-900">{formatCurrency(displayPrice)}</p>
              {displayCompareAtPrice && displayPrice < displayCompareAtPrice && (
                <p className="text-xl text-gray-500 line-through">{formatCurrency(displayCompareAtPrice)}</p>
              )}
            </div>

            {/* Reviews */}
            {product.reviews && product.reviews.length > 0 && (
              <div className="mt-3">
                <h3 className="sr-only">Reviews</h3>
                <div className="flex items-center">
                  <div className="flex items-center">
                    {[0, 1, 2, 3, 4].map((rating) => (
                      <StarIcon
                        key={rating}
                        className={classNames(
                          averageRating > rating ? 'text-yellow-400' : 'text-gray-200',
                          'h-5 w-5 flex-shrink-0'
                        )}
                        aria-hidden="true"
                      />
                    ))}
                  </div>
                  <p className="sr-only">{averageRating} out of 5 stars</p>
                  <a href="#reviews" className="ml-3 text-sm font-medium text-indigo-600 hover:text-indigo-500">
                    {product.reviews.length} reviews
                  </a>
                </div>
              </div>
            )}

            <div className="mt-6">
              <h3 className="sr-only">Description</h3>
              <div
                className="space-y-6 text-base text-gray-700"
                dangerouslySetInnerHTML={{ __html: product.shortDescription || product.description || '' }}
              />
            </div>

            <form className="mt-6">
              {/* Variant selector */}
              {product.options.map(option => (
                <div key={option.id} className="mt-4">
                  <h3 className="text-sm font-medium text-gray-900">{option.name}</h3>
                  <div className="mt-2 flex items-center space-x-3">
                    {option.values.map(value => (
                      <button
                        key={value}
                        type="button"
                        onClick={() => handleOptionChange(option.name, value)}
                        className={classNames(
                          selectedOptions[option.name] === value ? 'ring-indigo-500' : 'ring-gray-300',
                          'relative -m-0.5 flex items-center justify-center rounded-full border-2 p-0.5 focus:outline-none'
                        )}
                      >
                        <span className="sr-only">{value}</span>
                        <span
                          aria-hidden="true"
                          className="h-8 w-8 rounded-full border border-black border-opacity-10"
                          style={{ backgroundColor: option.name.toLowerCase() === 'color' ? value.toLowerCase() : undefined }}
                        >
                          {option.name.toLowerCase() !== 'color' && <span className="flex h-full w-full items-center justify-center text-xs">{value}</span>}
                        </span>
                      </button>
                    ))}
                  </div>
                </div>
              ))}

              {/* Quantity selector */}
              <div className="mt-8">
                <h3 className="text-sm font-medium text-gray-900">Quantity</h3>
                <input
                  type="number"
                  min="1"
                  value={quantity}
                  onChange={(e) => setQuantity(Math.max(1, parseInt(e.target.value) || 1))}
                  className="mt-2 w-24 rounded-md border border-gray-300 py-2 text-center text-sm focus:border-indigo-500 focus:ring-indigo-500"
                />
                <p className="mt-2 text-sm text-gray-500">
                  {displayQuantity} {displayQuantity === 1 ? 'item' : 'items'} in stock
                  {product.allowBackorder && displayQuantity <= 0 && ' (available for backorder)'}
                </p>
              </div>

              <div className="mt-10">
                <button
                  type="button"
                  onClick={handleAddToCart}
                  disabled={isOutOfStock && !product.allowBackorder}
                  className={classNames(
                    isOutOfStock && !product.allowBackorder
                      ? 'cursor-not-allowed bg-gray-400'
                      : 'bg-indigo-600 hover:bg-indigo-700 focus:ring-indigo-500',
                    'flex w-full items-center justify-center rounded-md border border-transparent px-8 py-3 text-base font-medium text-white focus:outline-none focus:ring-2 focus:ring-offset-2'
                  )}
                >
                  {isOutOfStock && !product.allowBackorder ? 'Out of Stock' : 'Add to cart'}
                </button>
              </div>
            </form>

            {/* Tabs (description, reviews, shipping) */}
            <section aria-labelledby="details-heading" className="mt-12">
              <h2 id="details-heading" className="sr-only">
                Additional details
              </h2>

              <Tab.Group as="div">
                <div className="border-b border-gray-200">
                  <Tab.List className="-mb-px flex space-x-8">
                    <Tab
                      className={({ selected }) =>
                        classNames(
                          selected
                            ? 'border-indigo-600 text-indigo-600'
                            : 'border-transparent text-gray-700 hover:border-gray-300 hover:text-gray-800',
                          'whitespace-nowrap border-b-2 py-6 text-sm font-medium'
                        )
                      }
                    >
                      Description
                    </Tab>
                    {product.reviews && product.reviews.length > 0 && (
                      <Tab
                        className={({ selected }) =>
                          classNames(
                            selected
                              ? 'border-indigo-600 text-indigo-600'
                              : 'border-transparent text-gray-700 hover:border-gray-300 hover:text-gray-800',
                            'whitespace-nowrap border-b-2 py-6 text-sm font-medium'
                          )
                        }
                      >
                        Reviews ({product.reviews.length})
                      </Tab>
                    )}
                    <Tab
                      className={({ selected }) =>
                        classNames(
                          selected
                            ? 'border-indigo-600 text-indigo-600'
                            : 'border-transparent text-gray-700 hover:border-gray-300 hover:text-gray-800',
                          'whitespace-nowrap border-b-2 py-6 text-sm font-medium'
                        )
                      }
                    >
                      Shipping & Returns
                    </Tab>
                  </Tab.List>
                </div>
                <Tab.Panels as={Fragment}>
                  <Tab.Panel className="pt-10">
                    <h3 className="sr-only">Description</h3>
                    <div
                      className="space-y-6 text-sm text-gray-700"
                      dangerouslySetInnerHTML={{ __html: product.description || 'No detailed description available.' }}
                    />
                  </Tab.Panel>

                  {product.reviews && product.reviews.length > 0 && (
                    <Tab.Panel className="pt-10" id="reviews">
                      <h3 className="sr-only">Customer Reviews</h3>
                      <div className="flow-root">
                        <div className="-my-12 divide-y divide-gray-200">
                          {product.reviews.map((review, index) => (
                            <div key={index} className="py-12">
                              <div className="flex items-center">
                                <div className="flex items-center">
                                  {[0, 1, 2, 3, 4].map((rating) => (
                                    <StarIcon
                                      key={rating}
                                      className={classNames(
                                        review.rating > rating ? 'text-yellow-400' : 'text-gray-200',
                                        'h-5 w-5 flex-shrink-0'
                                      )}
                                      aria-hidden="true"
                                    />
                                  ))}
                                </div>
                                <p className="ml-3 text-sm font-medium text-gray-900">{review.title}</p>
                              </div>
                              <div className="mt-4 text-sm text-gray-600">
                                <p>{review.comment}</p>
                              </div>
                              <p className="mt-4 text-xs text-gray-500">Reviewed on {new Date(review.createdAt).toLocaleDateString()}</p>
                            </div>
                          ))}
                        </div>
                      </div>
                    </Tab.Panel>
                  )}

                  <Tab.Panel className="pt-10">
                    <h3 className="sr-only">Shipping & Returns</h3>
                    <div className="space-y-6 text-sm text-gray-700">
                      <p>
                        <strong>Shipping:</strong> We offer various shipping options. Delivery times and costs vary based on your location and chosen method.
                        Typically, orders are processed within 1-2 business days.
                      </p>
                      <p>
                        <strong>Returns:</strong> You can return most new, unopened items within 30 days of delivery for a full refund.
                        We'll also pay the return shipping costs if the return is a result of our error (you received an incorrect or defective item, etc.).
                      </p>
                      <p>
                        Please refer to our full <Link href="/shipping-returns" className="text-indigo-600 hover:text-indigo-500">Shipping & Returns Policy</Link> for more details.
                      </p>
                    </div>
                  </Tab.Panel>
                </Tab.Panels>
              </Tab.Group>
            </section>
          </div>
        </div>
      </div>
    </div>
  );
};

export default ProductDetail;
```

---

## 11. `src/app/(shop)/products/page.tsx`

```tsx
import { Suspense } from 'react';
import ProductGrid from '@/components/products/product-grid';
import ProductFilters from '@/components/products/product-filters';
import { ProductListParams } from '@/server/services/product-service';
import { prisma } from '@/server/db'; // Assuming prisma client is initialized
import { ProductService } from '@/server/services/product-service';
import { CategoryService } from '@/server/services/category-service';
import { productListSchema } from '@/lib/validations/product';
import { redirect } from 'next/navigation';
import { z } from 'zod';

// Mock Brand data for filters (in a real app, this would come from a BrandService)
const mockBrands = [
  { id: 'brand_1', name: 'Brand A' },
  { id: 'brand_2', name: 'Brand B' },
  { id: 'brand_3', name: 'Brand C' },
];

interface ProductsPageProps {
  searchParams: { [key: string]: string | string[] | undefined };
}

export const metadata = {
  title: 'All Products - My E-commerce',
  description: 'Browse our wide selection of products.',
};

export default async function ProductsPage({ searchParams }: ProductsPageProps) {
  const productService = new ProductService(prisma);
  const categoryService = new CategoryService(prisma);

  // Parse and validate searchParams
  const parsedParams = productListSchema.safeParse({
    page: searchParams.page ? parseInt(searchParams.page as string) : undefined,
    pageSize: searchParams.pageSize ? parseInt(searchParams.pageSize as string) : undefined,
    categoryId: searchParams.categoryId,
    brandId: searchParams.brandId,
    minPrice: searchParams.minPrice ? parseFloat(searchParams.minPrice as string) : undefined,
    maxPrice: searchParams.maxPrice ? parseFloat(searchParams.maxPrice as string) : undefined,
    inStock: searchParams.inStock ? (searchParams.inStock === 'true') : undefined,
    sortBy: searchParams.sortBy,
    sortOrder: searchParams.sortOrder,
    search: searchParams.search,
  });

  if (!parsedParams.success) {
    // Handle validation errors, e.g., redirect to a clean URL or show error
    console.error('Invalid search params:', parsedParams.error);
    redirect('/products'); // Redirect to a clean products page
  }

  const currentFilters: ProductListParams = parsedParams.data;

  const { data: products, total, page, pageSize, totalPages } = await productService.list(currentFilters);

  // For filters, we might need to fetch categories and brands
  const categoriesTree = await categoryService.getTree(); // For the filter sidebar

  return (
    <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8 py-8">
      <h1 className="text-4xl font-extrabold tracking-tight text-gray-900 mb-8">All Products</h1>

      <div className="lg:grid lg:grid-cols-4 lg:gap-x-8">
        {/* Filters */}
        <aside className="lg:col-span-1 mb-8 lg:mb-0">
          <ProductFilters
            currentFilters={currentFilters}
            onApplyFilters={(newFilters) => {
              const newSearchParams = new URLSearchParams();
              Object.entries({ ...currentFilters, ...newFilters }).forEach(([key, value]) => {
                if (value !== undefined && value !== null && value !== '' && (typeof value !== 'boolean' || value === true)) {
                  newSearchParams.set(key, String(value));
                }
              });
              redirect(`/products?${newSearchParams.toString()}`);
            }}
            brands={mockBrands} // Pass actual brands from your data source
            // categories={categoriesTree} // Pass categories for the filter component
          />
        </aside>

        {/* Product Grid */}
        <main className="lg:col-span-3">
          <Suspense fallback={<ProductGrid products={[]} isLoading={true} />}>
            <ProductGrid
              products={products}
              isLoading={false}
              error={null} // Handle errors from service if any
              emptyMessage="No products match your criteria."
              onAddToCart={(productId) => console.log('Add to cart:', productId)} // Implement client-side cart logic
              onAddToWishlist={(productId) => console.log('Add to wishlist:', productId)} // Implement client-side wishlist logic
            />
            {/* Pagination */}
            {totalPages > 1 && (
              <div className="mt-8 flex justify-center">
                <nav className="isolate inline-flex -space-x-px rounded-md shadow-sm" aria-label="Pagination">
                  {Array.from({ length: totalPages }, (_, i) => i + 1).map((p) => (
                    <a
                      key={p}
                      href={`/products?${new URLSearchParams({ ...searchParams, page: String(p) }).toString()}`}
                      className={`relative inline-flex items-center px-4 py-2 text-sm font-semibold ${
                        p === page
                          ? 'z-10 bg-indigo-600 text-white focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-indigo-600'
                          : 'text-gray-900 ring-1 ring-inset ring-gray-300 hover:bg-gray-50 focus:outline-offset-0'
                      }`}
                    >
                      {p}
                    </a>
                  ))}
                </nav>
              </div>
            )}
          </Suspense>
        </main>
      </div>
    </div>
  );
}
```

---

## 12. `src/app/(shop)/products/[slug]/page.tsx`

```tsx
import { notFound } from 'next/navigation';
import { Metadata } from 'next';
import ProductDetail from '@/components/products/product-detail';
import { ProductService } from '@/server/services/product-service';
import { prisma } from '@/server/db'; // Assuming prisma client is initialized

interface ProductDetailPageProps {
  params: { slug: string };
}

// Generate static params for all product slugs at build time
export async function generateStaticParams() {
  const productService = new ProductService(prisma);
  const products = await productService.list({ pageSize: 1000, status: 'ACTIVE', visibility: 'VISIBLE' }); // Fetch all active, visible products
  return products.data.map((product) => ({
    slug: product.slug,
  }));
}

// Generate dynamic metadata for each product
export async function generateMetadata({ params }: ProductDetailPageProps): Promise<Metadata> {
  const productService = new ProductService(prisma);
  const product = await productService.getBySlug(params.slug);

  if (!product) {
    return {
      title: 'Product Not Found',
      description: 'The product you are looking for does not exist.',
    };
  }

  return {
    title: product.seoTitle || product.name,
    description: product.seoDescription || product.shortDescription || product.description?.substring(0, 160),
    openGraph: {
      title: product.seoTitle || product.name,
      description: product.seoDescription || product.shortDescription || product.description?.substring(0, 160),
      images: product.images[0]?.url ? [{ url: product.images[0].url, alt: product.images[0].altText || product.name }] : [],
      url: `${process.env.NEXT_PUBLIC_BASE_URL}/products/${product.slug}`,
      type: 'product',
    },
    twitter: {
      card: 'summary_large_image',
      title: product.seoTitle || product.name,
      description: product.seoDescription || product.shortDescription || product.description?.substring(0, 160),
      images: product.images[0]?.url ? [product.images[0].url] : [],
    },
  };
}

export default async function ProductDetailPage({ params }: ProductDetailPageProps) {
  const productService = new ProductService(prisma);
  const product = await productService.getBySlug(params.slug);

  if (!product) {
    notFound(); // Renders the not-found page
  }

  // Example of how to handle client-side interactions (e.g., add to cart)
  // In a real app, this would likely involve a global context or a tRPC mutation call.
  const handleAddToCart = (productId: string, quantity: number, variantId?: string) => {
    console.log(`Adding ${quantity} of product ${productId} (variant: ${variantId || 'none'}) to cart.`);
    // Implement actual cart logic here (e.g., call a tRPC mutation, update global state)
  };

  return (
    <ProductDetail
      product={product}
      onAddToCart={handleAddToCart}
    />
  );
}
```

---

## 13. `tests/products.test.ts`

```typescript
import { ProductService } from '../src/server/services/product-service';
import { PrismaClient, Product, ProductStatus, Visibility } from '@prisma/client';
import { DeepMockProxy, mockDeep } from 'jest-mock-extended';

// Mock Prisma Client
const prismaMock = mockDeep<PrismaClient>();
const productService = new ProductService(prismaMock as unknown as PrismaClient);

describe('ProductService', () => {
  let mockProduct: Product;
  let mockVariant: any; // Simplified mock for variant

  beforeEach(() => {
    mockProduct = {
      id: 'prod123',
      name: 'Test Product',
      slug: 'test-product',
      description: 'A description',
      shortDescription: 'Short desc',
      price: 100.00,
      compareAtPrice: null,
      costPrice: null,
      status: ProductStatus.ACTIVE,
      visibility: Visibility.VISIBLE,
      isFeatured: false,
      isDigital: false,
      sku: 'SKU123',
      barcode: 'BAR123',
      quantity: 10,
      trackInventory: true,
      allowBackorder: false,
      lowStockThreshold: 5,
      weight: null,
      width: null,
      height: null,
      depth: null,
      seoTitle: null,
      seoDescription: null,
      categoryId: null,
      brandId: null,
      createdAt: new Date(),
      updatedAt: new Date(),
      deletedAt: null,
    };

    mockVariant = {
      id: 'var123',
      productId: 'prod123',
      sku: 'SKU123-RED-M',
      price: 110.00,
      quantity: 5,
      isActive: true,
      optionValues: [{ id: 'ov1', optionName: 'Color', optionValue: 'Red' }],
      createdAt: new Date(),
      updatedAt: new Date(),
    };

    // Reset all mocks before each test
    jest.clearAllMocks();
  });

  // --- CRUD Tests ---

  it('should create a product', async () => {
    prismaMock.product.create.mockResolvedValue(mockProduct);
    const result = await productService.create({
      name: 'New Product',
      price: 50,
      slug: 'new-product',
    });
    expect(result).toEqual(expect.objectContaining({ name: 'Test Product' }));
    expect(prismaMock.product.create).toHaveBeenCalledWith(expect.objectContaining({
      data: expect.objectContaining({ name: 'New Product', slug: 'new-product' }),
    }));
  });

  it('should update a product', async () => {
    prismaMock.product.update.mockResolvedValue({ ...mockProduct, name: 'Updated Product' });
    const result = await productService.update(mockProduct.id, { name: 'Updated Product' });
    expect(result.name).toBe('Updated Product');
    expect(prismaMock.product.update).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: mockProduct.id },
      data: expect.objectContaining({ name: 'Updated Product' }),
    }));
  });

  it('should soft delete a product', async () => {
    prismaMock.product.update.mockResolvedValue({ ...mockProduct, deletedAt: new Date() });
    await productService.delete(mockProduct.id);
    expect(prismaMock.product.update).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: mockProduct.id },
      data: { deletedAt: expect.any(Date) },
    }));
  });

  it('should get a product by ID', async () => {
    prismaMock.product.findUnique.mockResolvedValue(mockProduct);
    const result = await productService.getById(mockProduct.id);
    expect(result).toEqual(mockProduct);
    expect(prismaMock.product.findUnique).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: mockProduct.id, deletedAt: null },
    }));
  });

  it('should get a product by slug', async () => {
    prismaMock.product.findUnique.mockResolvedValue(mockProduct);
    const result = await productService.getBySlug(mockProduct.slug);
    expect(result).toEqual(mockProduct);
    expect(prismaMock.product.findUnique).toHaveBeenCalledWith(expect.objectContaining({
      where: { slug: mockProduct.slug, deletedAt: null },
    }));
  });

  // --- Query Tests ---

  it('should list products with pagination', async () => {
    prismaMock.$transaction.mockResolvedValue([[mockProduct], 1]);
    const params = { page: 1, pageSize: 10 };
    const result = await productService.list(params);
    expect(result.data).toEqual([mockProduct]);
    expect(result.total).toBe(1);
    expect(prismaMock.product.findMany).toHaveBeenCalledWith(expect.objectContaining({
      skip: 0,
      take: 10,
      where: expect.objectContaining({ status: ProductStatus.ACTIVE }),
    }));
  });

  it('should search products by query', async () => {
    prismaMock.product.findMany.mockResolvedValue([mockProduct]);
    const result = await productService.search('Test');
    expect(result).toEqual([mockProduct]);
    expect(prismaMock.product.findMany).toHaveBeenCalledWith(expect.objectContaining({
      where: expect.objectContaining({
        OR: expect.arrayContaining([
          expect.objectContaining({ name: { contains: 'Test', mode: 'insensitive' } }),
        ]),
      }),
    }));
  });

  it('should get featured products', async () => {
    prismaMock.product.findMany.mockResolvedValue([mockProduct]);
    const result = await productService.getFeatured(5);
    expect(result).toEqual([mockProduct]);
    expect(prismaMock.product.findMany).toHaveBeenCalledWith(expect.objectContaining({
      where: expect.objectContaining({ isFeatured: true }),
      take: 5,
    }));
  });

  // --- Inventory Tests ---

  it('should decrement product stock', async () => {
    prismaMock.product.findUnique.mockResolvedValue(mockProduct);
    prismaMock.product.update.mockResolvedValue({ ...mockProduct, quantity: 8 });
    await productService.updateStock(mockProduct.id, 2, 'decrement');
    expect(prismaMock.product.update).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: mockProduct.id },
      data: { quantity: 8 },
    }));
  });

  it('should throw error if not enough stock and backorder not allowed', async () => {
    prismaMock.product.findUnique.mockResolvedValue({ ...mockProduct, quantity: 1, allowBackorder: false });
    await expect(productService.updateStock(mockProduct.id, 2, 'decrement')).rejects.toThrow('Not enough stock and backorders are not allowed.');
  });

  it('should check availability correctly', async () => {
    prismaMock.product.findUnique.mockResolvedValue({ ...mockProduct, quantity: 5 });
    const available = await productService.checkAvailability(mockProduct.id, 3);
    expect(available).toBe(true);
    const notAvailable = await productService.checkAvailability(mockProduct.id, 6);
    expect(notAvailable).toBe(false);
  });

  // --- Variant Tests ---

  it('should create a product variant', async () => {
    prismaMock.product.findUniqueOrThrow.mockResolvedValue(mockProduct);
    prismaMock.productVariant.create.mockResolvedValue(mockVariant);
    const result = await productService.createVariant(mockProduct.id, {
      price: 110,
      quantity: 5,
      optionValues: [{ optionName: 'Color', optionValue: 'Red' }],
    });
    expect(result).toEqual(expect.objectContaining({ productId: mockProduct.id, price: 110 }));
    expect(prismaMock.productVariant.create).toHaveBeenCalled();
  });

  it('should update a product variant', async () => {
    prismaMock.productVariant.update.mockResolvedValue({ ...mockVariant, price: 120 });
    const result = await productService.updateVariant(mockVariant.id, { price: 120 });
    expect(result.price).toBe(120);
    expect(prismaMock.productVariant.update).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: mockVariant.id },
      data: expect.objectContaining({ price: 120 }),
    }));
  });

  it('should delete a product variant', async () => {
    prismaMock.productVariant.delete.mockResolvedValue(mockVariant);
    await productService.deleteVariant(mockVariant.id);
    expect(prismaMock.productVariant.delete).toHaveBeenCalledWith({ where: { id: mockVariant.id } });
  });

  // --- Bulk Operations Tests ---

  it('should bulk update product status', async () => {
    prismaMock.product.updateMany.mockResolvedValue({ count: 2 });
    const count = await productService.bulkUpdateStatus(['id1', 'id2'], ProductStatus.ARCHIVED);
    expect(count).toBe(2);
    expect(prismaMock.product.updateMany).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: { in: ['id1', 'id2'] } },
      data: { status: ProductStatus.ARCHIVED },
    }));
  });

  it('should bulk delete products (soft delete)', async () => {
    prismaMock.product.updateMany.mockResolvedValue({ count: 2 });
    const count = await productService.bulkDelete(['id1', 'id2']);
    expect(count).toBe(2);
    expect(prismaMock.product.updateMany).toHaveBeenCalledWith(expect.objectContaining({
      where: { id: { in: ['id1', 'id2'] } },
      data: { deletedAt: expect.any(Date) },
    }));
  });

  it('should bulk update product prices', async () => {
    prismaMock.$transaction.mockResolvedValue([mockProduct, mockProduct]);
    const updates = [{ id: 'id1', price: 110 }, { id: 'id2', price: 120 }];
    const count = await productService.bulkUpdatePrices(updates);
    expect(count).toBe(2);
    expect(prismaMock.$transaction).toHaveBeenCalledTimes(1);
  });
});
```

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 31978_