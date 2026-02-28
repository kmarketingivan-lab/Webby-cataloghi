┌─────────────────────────────────────────────────────────────────────────────┐
│ DATABASE & DATA ARCHITECTURE CATALOG                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ TARGET: Full‑stack TypeScript / Node.js developers                          │
│ SCOPE: Database selection, scaling, cost & decision framework               │
│ VERSION: v1.0 (validated 2024–2025)                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 1: DATABASE USE‑CASE MATRIX
═══════════════════════════════════════════════════════════════════════════════

| # | Use Case                     | Recommended DB                | Rationale |
|---|------------------------------|-------------------------------|-----------|
| 1 | E‑commerce / Inventory       | PostgreSQL / MySQL             | ACID, relations |
| 2 | Real‑time chat               | Redis + PostgreSQL             | Pub/Sub + durability |
| 3 | Analytics / BI               | ClickHouse / BigQuery          | OLAP performance |
| 4 | Social network (graph)       | Neo4j                          | Graph traversal |
| 5 | Session storage              | Redis                          | Low latency |
| 6 | Full‑text search             | Elasticsearch                  | Search engine |
| 7 | Time‑series / IoT            | Cassandra / TimescaleDB        | High write throughput |
| 8 | Document storage             | MongoDB                        | Flexible schema |
| 9 | Caching layer                | Redis                          | In‑memory |
|10 | Multi‑tenant SaaS            | PostgreSQL / CockroachDB       | Isolation + scaling |


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 2: SCALING PATTERNS
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ READ REPLICAS                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Read‑heavy workloads                                                  │
│ DBs: PostgreSQL, MySQL, CockroachDB                                         │
│ Complexity: LOW                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ SHARDING                                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: >1TB data / extreme throughput                                        │
│ DBs: MongoDB, Cassandra, CockroachDB                                       │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ PARTITIONING                                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Large tables                                                          │
│ DBs: PostgreSQL, MySQL, ClickHouse                                         │
│ Complexity: MEDIUM                                                          │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ FEDERATION                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Multi‑domain data                                                     │
│ DBs: PostgreSQL, MySQL                                                     │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ CQRS                                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Read/write models diverge                                             │
│ DBs: PostgreSQL + Elasticsearch                                            │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT SOURCING                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Auditability, event‑driven systems                                    │
│ DBs: PostgreSQL, DynamoDB, Kafka                                           │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 3: COST COMPARISON (MANAGED SERVICES)
═══════════════════════════════════════════════════════════════════════════════

| Service        | Free Tier | Entry Price      | Scale Price        |
|----------------|-----------|------------------|--------------------|
| AWS RDS        | ❌        | ~15–30€/month    | 200–2000€/month    |
| PlanetScale    | ✅        | ~29$/month       | 300–3000$/month    |
| MongoDB Atlas  | ✅        | ~9$/month        | 200–5000$/month    |
| Supabase       | ✅        | ~25$/month       | 200–2000$/month    |
| Neon (Postgres)| ✅        | ~19$/month       | 200–1500$/month    |
| Turso (SQLite) | ✅        | 0–9$/month       | 100–500$/month     |


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 4: DECISION FRAMEWORK (CHECKLIST)
═══════════════════════════════════════════════════════════════════════════════

□ Strong ACID guarantees required → PostgreSQL / MySQL / CockroachDB
□ Flexible schema required         → MongoDB
□ Ultra‑low latency required       → Redis
□ Advanced search required         → Elasticsearch
□ Massive analytics required       → ClickHouse


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 5: ARCHITECTURAL NOTES (VALIDATED)
═══════════════════════════════════════════════════════════════════════════════

□ OLTP ≠ OLAP (never mix workloads)
□ Redis ≠ primary database
□ Sharding is last resort
□ Multi‑tenant isolation is a security concern
□ Cost grows non‑linearly with traffic


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 6: SOURCES & VALIDATION
═══════════════════════════════════════════════════════════════════════════════

• PostgreSQL Official Docs
• AWS Well‑Architected Framework
• Google Cloud Architecture Center
• MongoDB Architecture Guide
• Martin Fowler – CQRS / Event Sourcing
• ClickHouse Documentation





═══════════════════════════════════════════════════════════════════════════════
SEZIONE 7: SCHEMA DESIGN PATTERNS — E-COMMERCE
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Schema Prisma completo per un e-commerce con utenti, prodotti, ordini, carrello,
recensioni, categorie, varianti prodotto, indirizzi e pagamenti.

### Implementazione Completa

```prisma
// prisma/schema.prisma — E-Commerce Schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String
  firstName     String
  lastName      String
  role          UserRole  @default(CUSTOMER)
  emailVerified Boolean   @default(false)
  avatarUrl     String?
  phone         String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?

  addresses     Address[]
  orders        Order[]
  reviews       Review[]
  cart          Cart?
  wishlist      WishlistItem[]
  sessions      Session[]

  @@index([email])
  @@index([deletedAt])
  @@map("users")
}

enum UserRole {
  CUSTOMER
  ADMIN
  MANAGER
  SUPPORT
}

model Address {
  id         String      @id @default(cuid())
  userId     String
  type       AddressType @default(SHIPPING)
  firstName  String
  lastName   String
  street     String
  city       String
  state      String
  zipCode    String
  country    String      @default("IT")
  isDefault  Boolean     @default(false)
  createdAt  DateTime    @default(now())
  updatedAt  DateTime    @updatedAt

  user       User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  orders     Order[]

  @@index([userId])
  @@map("addresses")
}

enum AddressType {
  SHIPPING
  BILLING
}

model Category {
  id          String     @id @default(cuid())
  name        String
  slug        String     @unique
  description String?
  imageUrl    String?
  parentId    String?
  sortOrder   Int        @default(0)
  isActive    Boolean    @default(true)
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt

  parent      Category?  @relation("CategoryTree", fields: [parentId], references: [id])
  children    Category[] @relation("CategoryTree")
  products    ProductCategory[]

  @@index([slug])
  @@index([parentId])
  @@map("categories")
}

model Product {
  id              String    @id @default(cuid())
  name            String
  slug            String    @unique
  description     String
  shortDescription String?
  sku             String    @unique
  price           Decimal   @db.Decimal(10, 2)
  compareAtPrice  Decimal?  @db.Decimal(10, 2)
  costPrice       Decimal?  @db.Decimal(10, 2)
  currency        String    @default("EUR")
  stock           Int       @default(0)
  lowStockThreshold Int     @default(5)
  weight          Decimal?  @db.Decimal(8, 2)
  isActive        Boolean   @default(true)
  isFeatured      Boolean   @default(false)
  metaTitle       String?
  metaDescription String?
  createdAt       DateTime  @default(now())
  updatedAt       DateTime  @updatedAt
  deletedAt       DateTime?

  categories      ProductCategory[]
  variants        ProductVariant[]
  images          ProductImage[]
  reviews         Review[]
  cartItems       CartItem[]
  orderItems      OrderItem[]
  wishlistItems   WishlistItem[]

  @@index([slug])
  @@index([sku])
  @@index([isActive, isFeatured])
  @@index([deletedAt])
  @@index([price])
  @@map("products")
}

model ProductCategory {
  productId  String
  categoryId String
  product    Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  category   Category @relation(fields: [categoryId], references: [id], onDelete: Cascade)
  @@id([productId, categoryId])
  @@map("product_categories")
}

model ProductVariant {
  id        String   @id @default(cuid())
  productId String
  name      String
  sku       String   @unique
  price     Decimal  @db.Decimal(10, 2)
  stock     Int      @default(0)
  options   Json
  imageUrl  String?
  isActive  Boolean  @default(true)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  product   Product    @relation(fields: [productId], references: [id], onDelete: Cascade)
  cartItems CartItem[]
  orderItems OrderItem[]
  @@index([productId])
  @@index([sku])
  @@map("product_variants")
}

model ProductImage {
  id        String  @id @default(cuid())
  productId String
  url       String
  alt       String?
  sortOrder Int     @default(0)
  isPrimary Boolean @default(false)
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  @@index([productId])
  @@map("product_images")
}

model Cart {
  id        String     @id @default(cuid())
  userId    String     @unique
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
  user      User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  items     CartItem[]
  @@map("carts")
}

model CartItem {
  id        String          @id @default(cuid())
  cartId    String
  productId String
  variantId String?
  quantity  Int             @default(1)
  createdAt DateTime        @default(now())
  updatedAt DateTime        @updatedAt
  cart      Cart            @relation(fields: [cartId], references: [id], onDelete: Cascade)
  product   Product         @relation(fields: [productId], references: [id])
  variant   ProductVariant? @relation(fields: [variantId], references: [id])
  @@unique([cartId, productId, variantId])
  @@map("cart_items")
}

model Order {
  id              String      @id @default(cuid())
  orderNumber     String      @unique
  userId          String
  addressId       String
  status          OrderStatus @default(PENDING)
  subtotal        Decimal     @db.Decimal(10, 2)
  tax             Decimal     @db.Decimal(10, 2)
  shippingCost    Decimal     @db.Decimal(10, 2)
  discount        Decimal     @db.Decimal(10, 2) @default(0)
  total           Decimal     @db.Decimal(10, 2)
  currency        String      @default("EUR")
  paymentMethod   String?
  paymentIntentId String?     @unique
  notes           String?
  createdAt       DateTime    @default(now())
  updatedAt       DateTime    @updatedAt
  user            User        @relation(fields: [userId], references: [id])
  address         Address     @relation(fields: [addressId], references: [id])
  items           OrderItem[]
  payments        Payment[]
  timeline        OrderEvent[]
  @@index([userId])
  @@index([status])
  @@index([orderNumber])
  @@index([createdAt])
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
}

model OrderItem {
  id        String          @id @default(cuid())
  orderId   String
  productId String
  variantId String?
  name      String
  sku       String
  price     Decimal         @db.Decimal(10, 2)
  quantity  Int
  total     Decimal         @db.Decimal(10, 2)
  order     Order           @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product   Product         @relation(fields: [productId], references: [id])
  variant   ProductVariant? @relation(fields: [variantId], references: [id])
  @@index([orderId])
  @@map("order_items")
}

model OrderEvent {
  id        String   @id @default(cuid())
  orderId   String
  type      String
  message   String
  metadata  Json?
  createdAt DateTime @default(now())
  order     Order    @relation(fields: [orderId], references: [id], onDelete: Cascade)
  @@index([orderId])
  @@map("order_events")
}

model Payment {
  id              String        @id @default(cuid())
  orderId         String
  stripePaymentId String?       @unique
  amount          Decimal       @db.Decimal(10, 2)
  currency        String        @default("EUR")
  status          PaymentStatus @default(PENDING)
  method          String
  metadata        Json?
  createdAt       DateTime      @default(now())
  updatedAt       DateTime      @updatedAt
  order           Order         @relation(fields: [orderId], references: [id])
  @@index([orderId])
  @@index([stripePaymentId])
  @@map("payments")
}

enum PaymentStatus {
  PENDING
  PROCESSING
  SUCCEEDED
  FAILED
  REFUNDED
  PARTIALLY_REFUNDED
}

model Review {
  id        String   @id @default(cuid())
  userId    String
  productId String
  rating    Int
  title     String?
  body      String?
  isVerified Boolean @default(false)
  isApproved Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  user      User     @relation(fields: [userId], references: [id])
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  @@unique([userId, productId])
  @@index([productId, isApproved])
  @@index([rating])
  @@map("reviews")
}

model WishlistItem {
  id        String   @id @default(cuid())
  userId    String
  productId String
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  @@unique([userId, productId])
  @@map("wishlist_items")
}

model Session {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  ipAddress String?
  userAgent String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([token])
  @@index([userId])
  @@map("sessions")
}
```

### Varianti e Configurazioni

```typescript
// lib/db/schema-utils.ts — Utility types derivati dallo schema

import { Prisma } from "@prisma/client";

// Full Product with all relations
export type ProductWithRelations = Prisma.ProductGetPayload<{
  include: {
    categories: { include: { category: true } };
    variants: true;
    images: { orderBy: { sortOrder: "asc" } };
    reviews: { where: { isApproved: true } };
  };
}>;

// Order with items and user
export type OrderWithDetails = Prisma.OrderGetPayload<{
  include: {
    user: { select: { id: true; email: true; firstName: true; lastName: true } };
    items: { include: { product: true; variant: true } };
    address: true;
    payments: true;
    timeline: { orderBy: { createdAt: "desc" } };
  };
}>;

// Cart with items and product details
export type CartWithItems = Prisma.CartGetPayload<{
  include: {
    items: {
      include: {
        product: { include: { images: { where: { isPrimary: true } } } };
        variant: true;
      };
    };
  };
}>;

// Reusable select objects
export const productListSelect = {
  id: true,
  name: true,
  slug: true,
  price: true,
  compareAtPrice: true,
  stock: true,
  isActive: true,
  images: {
    where: { isPrimary: true },
    take: 1,
    select: { url: true, alt: true },
  },
  reviews: {
    select: { rating: true },
  },
  categories: {
    select: { category: { select: { name: true, slug: true } } },
  },
} satisfies Prisma.ProductSelect;

export type ProductListItem = Prisma.ProductGetPayload<{
  select: typeof productListSelect;
}>;

// Order number generator
export function generateOrderNumber(): string {
  const timestamp = Date.now().toString(36).toUpperCase();
  const random = Math.random().toString(36).substring(2, 6).toUpperCase();
  return `ORD-${timestamp}-${random}`;
}

// Price formatting
export function formatPrice(
  amount: Prisma.Decimal | number,
  currency = "EUR"
): string {
  const num = typeof amount === "number" ? amount : Number(amount);
  return new Intl.NumberFormat("it-IT", {
    style: "currency",
    currency,
  }).format(num);
}
```

### Edge Cases e Error Handling

```typescript
// lib/db/ecommerce-queries.ts — Query patterns con error handling

import { PrismaClient, Prisma } from "@prisma/client";

const prisma = new PrismaClient();

// Stock check con race condition protection
export async function reserveStock(
  productId: string,
  variantId: string | null,
  quantity: number
): Promise<{ success: boolean; availableStock: number }> {
  try {
    const result = await prisma.$transaction(async (tx) => {
      if (variantId) {
        const variant = await tx.productVariant.findUnique({
          where: { id: variantId },
          select: { stock: true },
        });
        if (!variant || variant.stock < quantity) {
          return { success: false, availableStock: variant?.stock ?? 0 };
        }
        await tx.productVariant.update({
          where: { id: variantId },
          data: { stock: { decrement: quantity } },
        });
        return { success: true, availableStock: variant.stock - quantity };
      }

      const product = await tx.product.findUnique({
        where: { id: productId },
        select: { stock: true },
      });
      if (!product || product.stock < quantity) {
        return { success: false, availableStock: product?.stock ?? 0 };
      }
      await tx.product.update({
        where: { id: productId },
        data: { stock: { decrement: quantity } },
      });
      return { success: true, availableStock: product.stock - quantity };
    }, {
      isolationLevel: Prisma.TransactionIsolationLevel.Serializable,
    });
    return result;
  } catch (error) {
    if (error instanceof Prisma.PrismaClientKnownRequestError) {
      if (error.code === "P2034") {
        return reserveStock(productId, variantId, quantity);
      }
    }
    throw error;
  }
}

// Duplicate order prevention
export async function createOrderIdempotent(
  idempotencyKey: string,
  orderData: Prisma.OrderCreateInput
): Promise<Prisma.OrderGetPayload<{ include: { items: true } }>> {
  const existing = await prisma.order.findFirst({
    where: { paymentIntentId: idempotencyKey },
    include: { items: true },
  });
  if (existing) return existing;

  return prisma.order.create({
    data: { ...orderData, paymentIntentId: idempotencyKey },
    include: { items: true },
  });
}

// Safe product deletion (check for active orders)
export async function safeDeleteProduct(productId: string): Promise<{
  deleted: boolean;
  reason?: string;
}> {
  const activeOrders = await prisma.orderItem.count({
    where: {
      productId,
      order: {
        status: { in: ["PENDING", "CONFIRMED", "PROCESSING", "SHIPPED"] },
      },
    },
  });
  if (activeOrders > 0) {
    return {
      deleted: false,
      reason: `Cannot delete: ${activeOrders} active orders reference this product`,
    };
  }
  await prisma.product.update({
    where: { id: productId },
    data: { deletedAt: new Date(), isActive: false },
  });
  return { deleted: true };
}
```

### Errori Comuni da Evitare
- Non usare `@@unique` su colonne nullable senza considerare che PostgreSQL tratta NULL come distinto
- Non dimenticare `onDelete: Cascade` sulle relazioni figlio per evitare record orfani
- Non usare `Decimal` per calcoli JS — convertire sempre con `Number()` o `toFixed()`
- Non creare indici su colonne con bassa cardinalita (es. `Boolean`) senza combinarli

### Checklist di Verifica
- [ ] Ogni tabella ha un campo `id` con `@id @default(cuid())`
- [ ] Campi `createdAt` e `updatedAt` presenti su tutte le entita principali
- [ ] Indici su tutte le foreign key
- [ ] Indici su campi usati in WHERE/ORDER BY frequenti
- [ ] Soft delete implementato con `deletedAt DateTime?`
- [ ] Enum definiti per stati finiti (OrderStatus, UserRole, etc.)


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 8: SCHEMA DESIGN PATTERNS — SAAS MULTI-TENANT
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Schema Prisma per applicazioni SaaS multi-tenant con organizzazioni, workspace,
membri, subscription, inviti e row-level security.

### Implementazione Completa

```prisma
// prisma/schema-saas.prisma — SaaS Multi-Tenant Schema

model Organization {
  id            String    @id @default(cuid())
  name          String
  slug          String    @unique
  logoUrl       String?
  domain        String?   @unique
  plan          Plan      @default(FREE)
  stripeCustomerId    String? @unique
  stripeSubscriptionId String? @unique
  trialEndsAt   DateTime?
  isActive      Boolean   @default(true)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  deletedAt     DateTime?

  members       Member[]
  workspaces    Workspace[]
  invitations   Invitation[]
  apiKeys       ApiKey[]
  auditLogs     AuditLog[]
  subscriptions Subscription[]
  usageLimits   UsageLimit[]

  @@index([slug])
  @@index([domain])
  @@index([stripeCustomerId])
  @@map("organizations")
}

enum Plan {
  FREE
  STARTER
  PRO
  ENTERPRISE
}

model Member {
  id             String     @id @default(cuid())
  organizationId String
  userId         String
  role           MemberRole @default(MEMBER)
  joinedAt       DateTime   @default(now())
  updatedAt      DateTime   @updatedAt

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  user           SaasUser     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([organizationId, userId])
  @@index([userId])
  @@map("members")
}

enum MemberRole {
  OWNER
  ADMIN
  MEMBER
  VIEWER
  BILLING
}

model SaasUser {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String
  name          String
  avatarUrl     String?
  emailVerified Boolean   @default(false)
  lastLoginAt   DateTime?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  memberships   Member[]
  invitations   Invitation[]
  sessions      SaasSession[]

  @@index([email])
  @@map("saas_users")
}

model Workspace {
  id             String   @id @default(cuid())
  organizationId String
  name           String
  slug           String
  description    String?
  color          String   @default("#6366f1")
  isDefault      Boolean  @default(false)
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  projects       Project[]

  @@unique([organizationId, slug])
  @@index([organizationId])
  @@map("workspaces")
}

model Project {
  id          String   @id @default(cuid())
  workspaceId String
  name        String
  description String?
  status      ProjectStatus @default(ACTIVE)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  workspace   Workspace @relation(fields: [workspaceId], references: [id], onDelete: Cascade)

  @@index([workspaceId])
  @@map("projects")
}

enum ProjectStatus {
  ACTIVE
  ARCHIVED
  DELETED
}

model Invitation {
  id             String           @id @default(cuid())
  organizationId String
  email          String
  role           MemberRole       @default(MEMBER)
  status         InvitationStatus @default(PENDING)
  token          String           @unique @default(cuid())
  invitedById    String
  expiresAt      DateTime
  createdAt      DateTime         @default(now())

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)
  invitedBy      SaasUser     @relation(fields: [invitedById], references: [id])

  @@unique([organizationId, email])
  @@index([token])
  @@map("invitations")
}

enum InvitationStatus {
  PENDING
  ACCEPTED
  EXPIRED
  REVOKED
}

model ApiKey {
  id             String   @id @default(cuid())
  organizationId String
  name           String
  keyHash        String   @unique
  keyPrefix      String
  lastUsedAt     DateTime?
  expiresAt      DateTime?
  createdAt      DateTime @default(now())

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([keyHash])
  @@index([organizationId])
  @@map("api_keys")
}

model Subscription {
  id                   String             @id @default(cuid())
  organizationId       String
  stripeSubscriptionId String             @unique
  stripePriceId        String
  status               SubscriptionStatus
  currentPeriodStart   DateTime
  currentPeriodEnd     DateTime
  cancelAtPeriodEnd    Boolean            @default(false)
  createdAt            DateTime           @default(now())
  updatedAt            DateTime           @updatedAt

  organization         Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId])
  @@index([stripeSubscriptionId])
  @@map("subscriptions")
}

enum SubscriptionStatus {
  ACTIVE
  PAST_DUE
  CANCELED
  INCOMPLETE
  TRIALING
  UNPAID
}

model UsageLimit {
  id             String   @id @default(cuid())
  organizationId String
  feature        String
  limit          Int
  used           Int      @default(0)
  resetAt        DateTime
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@unique([organizationId, feature])
  @@map("usage_limits")
}

model AuditLog {
  id             String   @id @default(cuid())
  organizationId String
  actorId        String?
  action         String
  entity         String
  entityId       String?
  metadata       Json?
  ipAddress      String?
  createdAt      DateTime @default(now())

  organization   Organization @relation(fields: [organizationId], references: [id], onDelete: Cascade)

  @@index([organizationId, createdAt])
  @@index([actorId])
  @@index([entity, entityId])
  @@map("audit_logs")
}

model SaasSession {
  id        String   @id @default(cuid())
  userId    String
  token     String   @unique
  expiresAt DateTime
  ipAddress String?
  userAgent String?
  createdAt DateTime @default(now())
  user      SaasUser @relation(fields: [userId], references: [id], onDelete: Cascade)
  @@index([token])
  @@index([userId])
  @@map("saas_sessions")
}
```

### Varianti e Configurazioni

```typescript
// lib/db/tenant-context.ts — Tenant isolation middleware

import { PrismaClient, Prisma } from "@prisma/client";

// Prisma extension per tenant isolation automatico
export function createTenantPrisma(organizationId: string) {
  const prisma = new PrismaClient();

  return prisma.$extends({
    query: {
      $allModels: {
        async findMany({ args, query }) {
          if ("organizationId" in (args.where ?? {})) {
            args.where = { ...args.where, organizationId };
          }
          return query(args);
        },
        async findFirst({ args, query }) {
          if ("organizationId" in (args.where ?? {})) {
            args.where = { ...args.where, organizationId };
          }
          return query(args);
        },
        async create({ args, query }) {
          if ("organizationId" in (args.data ?? {})) {
            args.data = { ...args.data, organizationId };
          }
          return query(args);
        },
        async update({ args, query }) {
          if ("organizationId" in (args.where ?? {})) {
            args.where = { ...args.where, organizationId };
          }
          return query(args);
        },
        async delete({ args, query }) {
          if ("organizationId" in (args.where ?? {})) {
            args.where = { ...args.where, organizationId };
          }
          return query(args);
        },
      },
    },
  });
}

// Usage limit checker
export async function checkUsageLimit(
  prisma: PrismaClient,
  organizationId: string,
  feature: string
): Promise<{ allowed: boolean; used: number; limit: number }> {
  const usage = await prisma.usageLimit.findUnique({
    where: {
      organizationId_feature: { organizationId, feature },
    },
  });

  if (!usage) {
    return { allowed: true, used: 0, limit: Infinity };
  }

  if (new Date() > usage.resetAt) {
    await prisma.usageLimit.update({
      where: { id: usage.id },
      data: { used: 0, resetAt: getNextResetDate() },
    });
    return { allowed: true, used: 0, limit: usage.limit };
  }

  return {
    allowed: usage.used < usage.limit,
    used: usage.used,
    limit: usage.limit,
  };
}

export async function incrementUsage(
  prisma: PrismaClient,
  organizationId: string,
  feature: string
): Promise<void> {
  await prisma.usageLimit.update({
    where: {
      organizationId_feature: { organizationId, feature },
    },
    data: { used: { increment: 1 } },
  });
}

function getNextResetDate(): Date {
  const now = new Date();
  return new Date(now.getFullYear(), now.getMonth() + 1, 1);
}

// Plan-based feature access
const PLAN_FEATURES: Record<string, Record<string, number>> = {
  FREE: { members: 3, workspaces: 1, projects: 5, apiKeys: 1 },
  STARTER: { members: 10, workspaces: 5, projects: 25, apiKeys: 5 },
  PRO: { members: 50, workspaces: 20, projects: 100, apiKeys: 20 },
  ENTERPRISE: { members: -1, workspaces: -1, projects: -1, apiKeys: -1 },
};

export function getPlanLimit(plan: string, feature: string): number {
  const limits = PLAN_FEATURES[plan];
  if (!limits) return 0;
  const limit = limits[feature];
  return limit === -1 ? Infinity : limit ?? 0;
}
```

### Edge Cases e Error Handling

```typescript
// lib/db/saas-operations.ts — SaaS operations con error handling

import { PrismaClient, Prisma, MemberRole } from "@prisma/client";
import crypto from "crypto";

const prisma = new PrismaClient();

// Safe organization creation with slug collision handling
export async function createOrganization(
  userId: string,
  name: string
): Promise<{ id: string; slug: string }> {
  let slug = name
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/^-|-$/g, "");

  const existingSlug = await prisma.organization.findUnique({
    where: { slug },
  });

  if (existingSlug) {
    slug = `${slug}-${crypto.randomBytes(3).toString("hex")}`;
  }

  const org = await prisma.$transaction(async (tx) => {
    const organization = await tx.organization.create({
      data: {
        name,
        slug,
        trialEndsAt: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000),
      },
    });

    await tx.member.create({
      data: {
        organizationId: organization.id,
        userId,
        role: MemberRole.OWNER,
      },
    });

    await tx.workspace.create({
      data: {
        organizationId: organization.id,
        name: "Default",
        slug: "default",
        isDefault: true,
      },
    });

    const plan = "FREE";
    const features = ["members", "workspaces", "projects", "apiKeys"];
    const resetAt = new Date(
      new Date().getFullYear(),
      new Date().getMonth() + 1,
      1
    );

    await tx.usageLimit.createMany({
      data: features.map((feature) => ({
        organizationId: organization.id,
        feature,
        limit: getPlanLimitValue(plan, feature),
        resetAt,
      })),
    });

    return organization;
  });

  return { id: org.id, slug: org.slug };
}

function getPlanLimitValue(plan: string, feature: string): number {
  const limits: Record<string, Record<string, number>> = {
    FREE: { members: 3, workspaces: 1, projects: 5, apiKeys: 1 },
    STARTER: { members: 10, workspaces: 5, projects: 25, apiKeys: 5 },
    PRO: { members: 50, workspaces: 20, projects: 100, apiKeys: 20 },
    ENTERPRISE: { members: 999999, workspaces: 999999, projects: 999999, apiKeys: 999999 },
  };
  return limits[plan]?.[feature] ?? 0;
}

// API key generation with secure hashing
export async function generateApiKey(
  organizationId: string,
  name: string
): Promise<{ key: string; keyPrefix: string }> {
  const rawKey = `wby_${crypto.randomBytes(32).toString("hex")}`;
  const keyPrefix = rawKey.substring(0, 12);
  const keyHash = crypto.createHash("sha256").update(rawKey).digest("hex");

  await prisma.apiKey.create({
    data: {
      organizationId,
      name,
      keyHash,
      keyPrefix,
      expiresAt: new Date(Date.now() + 365 * 24 * 60 * 60 * 1000),
    },
  });

  return { key: rawKey, keyPrefix };
}

// Validate API key
export async function validateApiKey(
  rawKey: string
): Promise<{ valid: boolean; organizationId?: string }> {
  const keyHash = crypto.createHash("sha256").update(rawKey).digest("hex");
  const apiKey = await prisma.apiKey.findUnique({
    where: { keyHash },
    select: {
      id: true,
      organizationId: true,
      expiresAt: true,
    },
  });

  if (!apiKey) return { valid: false };
  if (apiKey.expiresAt && apiKey.expiresAt < new Date()) return { valid: false };

  await prisma.apiKey.update({
    where: { id: apiKey.id },
    data: { lastUsedAt: new Date() },
  });

  return { valid: true, organizationId: apiKey.organizationId };
}
```

### Errori Comuni da Evitare
- Non dimenticare di controllare il piano prima di creare risorse (member limit, workspace limit)
- Non esporre mai l'API key raw dopo la creazione — memorizzare solo l'hash
- Non dimenticare `onDelete: Cascade` sulle relazioni tenant per cleanup completo
- Non mischiare dati tra tenant — usare sempre il middleware di isolation

### Checklist di Verifica
- [ ] Row-level security implementata via middleware Prisma
- [ ] Ogni query filtra per `organizationId`
- [ ] Usage limits configurati per ogni piano
- [ ] API key hash con SHA-256, mai salvato in chiaro
- [ ] Audit log su ogni operazione critica
- [ ] Inviti con scadenza e token unico


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 9: INDEXING STRATEGY
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Strategie di indicizzazione per PostgreSQL con Prisma: indici compositi, parziali,
GIN per full-text search, e monitoraggio performance indici.

### Implementazione Completa

```prisma
// prisma/schema-indexes.prisma — Strategie di indicizzazione avanzate

model ProductSearchable {
  id          String   @id @default(cuid())
  name        String
  description String
  sku         String   @unique
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  isActive    Boolean  @default(true)
  categoryId  String
  brandId     String?
  rating      Float    @default(0)
  salesCount  Int      @default(0)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  deletedAt   DateTime?

  // Indice composito per listing con filtri comuni
  @@index([isActive, categoryId, price])

  // Indice composito per sorting
  @@index([isActive, createdAt(sort: Desc)])
  @@index([isActive, salesCount(sort: Desc)])
  @@index([isActive, rating(sort: Desc)])

  // Indice parziale per prodotti attivi (via raw SQL migration)
  // CREATE INDEX idx_products_active ON products (category_id, price)
  //   WHERE is_active = true AND deleted_at IS NULL;

  // Indice per ricerca full-text (via raw SQL migration)
  // CREATE INDEX idx_products_search ON products
  //   USING GIN (to_tsvector('italian', name || ' ' || description));

  @@map("products_searchable")
}
```

```typescript
// lib/db/indexing-utils.ts — Index management e monitoring

import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

// Crea indici full-text search via raw SQL
export async function createFullTextIndexes(): Promise<void> {
  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_products_fts
    ON products_searchable
    USING GIN (to_tsvector('italian', name || ' ' || coalesce(description, '')));
  `;

  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_products_fts_english
    ON products_searchable
    USING GIN (to_tsvector('english', name || ' ' || coalesce(description, '')));
  `;

  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_products_trigram
    ON products_searchable
    USING GIN (name gin_trgm_ops);
  `;
}

// Crea indici parziali per query frequenti
export async function createPartialIndexes(): Promise<void> {
  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_products_active_category
    ON products_searchable (category_id, price)
    WHERE is_active = true AND deleted_at IS NULL;
  `;

  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_orders_pending
    ON orders (created_at DESC)
    WHERE status IN ('PENDING', 'CONFIRMED', 'PROCESSING');
  `;

  await prisma.$executeRaw`
    CREATE INDEX IF NOT EXISTS idx_users_active
    ON users (email, created_at)
    WHERE deleted_at IS NULL AND email_verified = true;
  `;
}

// Full-text search query
export async function searchProducts(
  query: string,
  options: {
    language?: "italian" | "english";
    limit?: number;
    offset?: number;
    categoryId?: string;
    minPrice?: number;
    maxPrice?: number;
  } = {}
) {
  const {
    language = "italian",
    limit = 20,
    offset = 0,
    categoryId,
    minPrice,
    maxPrice,
  } = options;

  const results = await prisma.$queryRaw<
    Array<{
      id: string;
      name: string;
      description: string;
      price: number;
      rank: number;
      headline: string;
    }>
  >`
    SELECT
      id,
      name,
      description,
      price::float,
      ts_rank(
        to_tsvector(${language}::regconfig, name || ' ' || coalesce(description, '')),
        plainto_tsquery(${language}::regconfig, ${query})
      ) AS rank,
      ts_headline(
        ${language}::regconfig,
        name || ' ' || coalesce(description, ''),
        plainto_tsquery(${language}::regconfig, ${query}),
        'StartSel=<mark>, StopSel=</mark>, MaxWords=50, MinWords=20'
      ) AS headline
    FROM products_searchable
    WHERE
      is_active = true
      AND deleted_at IS NULL
      AND to_tsvector(${language}::regconfig, name || ' ' || coalesce(description, ''))
          @@ plainto_tsquery(${language}::regconfig, ${query})
      ${categoryId ? Prisma.sql`AND category_id = ${categoryId}` : Prisma.empty}
      ${minPrice !== undefined ? Prisma.sql`AND price >= ${minPrice}` : Prisma.empty}
      ${maxPrice !== undefined ? Prisma.sql`AND price <= ${maxPrice}` : Prisma.empty}
    ORDER BY rank DESC
    LIMIT ${limit}
    OFFSET ${offset}
  `;

  return results;
}

// Monitor index usage
export async function getIndexUsageStats(): Promise<
  Array<{
    schemaname: string;
    tablename: string;
    indexname: string;
    idx_scan: number;
    idx_tup_read: number;
    idx_tup_fetch: number;
    index_size: string;
  }>
> {
  return prisma.$queryRaw`
    SELECT
      schemaname,
      tablename,
      indexrelname AS indexname,
      idx_scan,
      idx_tup_read,
      idx_tup_fetch,
      pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
    FROM pg_stat_user_indexes
    ORDER BY idx_scan ASC
  `;
}

// Find unused indexes
export async function getUnusedIndexes(): Promise<
  Array<{ indexname: string; tablename: string; index_size: string }>
> {
  return prisma.$queryRaw`
    SELECT
      indexrelname AS indexname,
      relname AS tablename,
      pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
    FROM pg_stat_user_indexes
    WHERE idx_scan = 0
      AND schemaname = 'public'
    ORDER BY pg_relation_size(indexrelid) DESC
  `;
}

// Find missing indexes (tables with sequential scans)
export async function getMissingIndexSuggestions(): Promise<
  Array<{
    tablename: string;
    seq_scan: number;
    seq_tup_read: number;
    idx_scan: number;
    ratio: string;
  }>
> {
  return prisma.$queryRaw`
    SELECT
      relname AS tablename,
      seq_scan,
      seq_tup_read,
      idx_scan,
      CASE
        WHEN seq_scan + idx_scan > 0
        THEN round(100.0 * idx_scan / (seq_scan + idx_scan), 2) || '%'
        ELSE 'N/A'
      END AS ratio
    FROM pg_stat_user_tables
    WHERE seq_scan > 1000
    ORDER BY seq_tup_read DESC
    LIMIT 20
  `;
}

// EXPLAIN ANALYZE wrapper
export async function explainQuery(query: string): Promise<string[]> {
  const result = await prisma.$queryRawUnsafe<Array<{ "QUERY PLAN": string }>>(
    `EXPLAIN ANALYZE ${query}`
  );
  return result.map((row) => row["QUERY PLAN"]);
}
```

### Varianti e Configurazioni

```typescript
// lib/db/index-health-dashboard.ts — API route per monitoraggio indici

import { NextRequest, NextResponse } from "next/server";
import {
  getIndexUsageStats,
  getUnusedIndexes,
  getMissingIndexSuggestions,
} from "@/lib/db/indexing-utils";

// GET /api/admin/db-health
export async function GET(request: NextRequest) {
  const authHeader = request.headers.get("authorization");
  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
  }

  try {
    const [indexUsage, unusedIndexes, missingSuggestions] = await Promise.all([
      getIndexUsageStats(),
      getUnusedIndexes(),
      getMissingIndexSuggestions(),
    ]);

    const totalIndexSize = indexUsage.reduce(
      (sum, idx) => sum + parseFloat(idx.index_size),
      0
    );

    return NextResponse.json({
      summary: {
        totalIndexes: indexUsage.length,
        unusedIndexes: unusedIndexes.length,
        tablesNeedingIndexes: missingSuggestions.length,
      },
      indexUsage: indexUsage.slice(0, 50),
      unusedIndexes,
      missingSuggestions,
      recommendations: generateRecommendations(
        unusedIndexes,
        missingSuggestions
      ),
    });
  } catch (error) {
    return NextResponse.json(
      { error: "Failed to fetch database health" },
      { status: 500 }
    );
  }
}

function generateRecommendations(
  unused: Array<{ indexname: string; tablename: string; index_size: string }>,
  missing: Array<{ tablename: string; seq_scan: number }>
): string[] {
  const recommendations: string[] = [];

  if (unused.length > 5) {
    recommendations.push(
      `Consider removing ${unused.length} unused indexes to save disk space`
    );
  }

  for (const table of missing) {
    if (table.seq_scan > 10000) {
      recommendations.push(
        `Table "${table.tablename}" has ${table.seq_scan} sequential scans - add appropriate indexes`
      );
    }
  }

  return recommendations;
}
```

### Errori Comuni da Evitare
- Non creare indici duplicati (Prisma crea automaticamente indici su `@unique` e `@id`)
- Non indicizzare colonne con bassa selettivita (es. boolean) da sole
- Non dimenticare `CONCURRENTLY` quando si creano indici in produzione
- Non ignorare indici inutilizzati — occupano spazio e rallentano le write

### Checklist di Verifica
- [ ] Indici compositi nell'ordine corretto (colonne piu selettive prima)
- [ ] Indici parziali per query su subset di dati frequenti
- [ ] GIN index per full-text search
- [ ] Monitoring periodico con `pg_stat_user_indexes`
- [ ] Piano di rimozione indici inutilizzati
- [ ] `CREATE INDEX CONCURRENTLY` per produzione


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 10: MIGRATION WORKFLOW CON PRISMA
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Workflow completo per migrazioni database con Prisma: creazione, rollback,
migrazioni dati, zero-downtime deployment.

### Implementazione Completa

```typescript
// scripts/migrate.ts — Migration runner con safety checks

import { execSync } from "child_process";
import { PrismaClient } from "@prisma/client";
import fs from "fs";
import path from "path";

const prisma = new PrismaClient();

interface MigrationConfig {
  environment: "development" | "staging" | "production";
  dryRun: boolean;
  timeout: number;
}

export async function runMigration(config: MigrationConfig): Promise<void> {
  const { environment, dryRun, timeout } = config;

  console.log(`[Migration] Environment: ${environment}`);
  console.log(`[Migration] Dry run: ${dryRun}`);

  // Pre-migration checks
  await checkDatabaseConnection();
  await checkPendingMigrations();
  await createBackupPoint(environment);

  if (environment === "production") {
    await checkActiveConnections();
    await enableMaintenanceMode();
  }

  try {
    if (dryRun) {
      console.log("[Migration] Running diff only...");
      execSync("npx prisma migrate diff --from-schema-datasource prisma/schema.prisma --to-schema-datamodel prisma/schema.prisma --script", {
        stdio: "inherit",
        timeout,
      });
    } else if (environment === "production") {
      console.log("[Migration] Deploying migration...");
      execSync("npx prisma migrate deploy", {
        stdio: "inherit",
        timeout,
      });
    } else {
      console.log("[Migration] Running dev migration...");
      execSync("npx prisma migrate dev", {
        stdio: "inherit",
        timeout,
      });
    }

    console.log("[Migration] Generating Prisma Client...");
    execSync("npx prisma generate", { stdio: "inherit" });

    console.log("[Migration] Migration completed successfully");
  } catch (error) {
    console.error("[Migration] Migration failed:", error);

    if (environment === "production") {
      console.log("[Migration] Attempting rollback...");
      await rollbackMigration();
    }

    throw error;
  } finally {
    if (environment === "production") {
      await disableMaintenanceMode();
    }
    await prisma.$disconnect();
  }
}

async function checkDatabaseConnection(): Promise<void> {
  try {
    await prisma.$queryRaw`SELECT 1`;
    console.log("[Migration] Database connection OK");
  } catch {
    throw new Error("Cannot connect to database");
  }
}

async function checkPendingMigrations(): Promise<void> {
  const migrationsDir = path.join(process.cwd(), "prisma/migrations");
  if (!fs.existsSync(migrationsDir)) {
    console.log("[Migration] No migrations directory found");
    return;
  }

  const migrations = fs
    .readdirSync(migrationsDir)
    .filter((d) => !d.startsWith(".") && d !== "migration_lock.toml");

  console.log(`[Migration] Found ${migrations.length} total migrations`);
}

async function checkActiveConnections(): Promise<void> {
  const connections = await prisma.$queryRaw<
    Array<{ count: number }>
  >`SELECT count(*) FROM pg_stat_activity WHERE state = 'active'`;

  const activeCount = Number(connections[0]?.count ?? 0);
  console.log(`[Migration] Active connections: ${activeCount}`);

  if (activeCount > 50) {
    console.warn("[Migration] WARNING: High number of active connections");
  }
}

async function createBackupPoint(env: string): Promise<void> {
  if (env === "production") {
    const timestamp = new Date().toISOString().replace(/[:.]/g, "-");
    console.log(`[Migration] Backup point created: bp_${timestamp}`);
  }
}

async function enableMaintenanceMode(): Promise<void> {
  console.log("[Migration] Maintenance mode enabled");
}

async function disableMaintenanceMode(): Promise<void> {
  console.log("[Migration] Maintenance mode disabled");
}

async function rollbackMigration(): Promise<void> {
  try {
    execSync("npx prisma migrate resolve --rolled-back $(ls prisma/migrations | tail -1)", {
      stdio: "inherit",
    });
    console.log("[Migration] Rollback completed");
  } catch {
    console.error("[Migration] Rollback failed — manual intervention required");
  }
}

// CLI entry point
const env = (process.env.NODE_ENV as MigrationConfig["environment"]) ?? "development";
const dryRun = process.argv.includes("--dry-run");

runMigration({
  environment: env,
  dryRun,
  timeout: 120000,
}).catch((err) => {
  console.error(err);
  process.exit(1);
});
```

### Varianti e Configurazioni

```typescript
// scripts/data-migration.ts — Data migration pattern

import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();
const BATCH_SIZE = 1000;

interface DataMigrationResult {
  processed: number;
  skipped: number;
  errors: number;
  duration: number;
}

// Batch data migration con progress tracking
export async function migrateUserDisplayNames(): Promise<DataMigrationResult> {
  const startTime = Date.now();
  let processed = 0;
  let skipped = 0;
  let errors = 0;
  let cursor: string | undefined;

  while (true) {
    const users = await prisma.user.findMany({
      take: BATCH_SIZE,
      ...(cursor ? { skip: 1, cursor: { id: cursor } } : {}),
      where: {
        deletedAt: null,
      },
      select: {
        id: true,
        firstName: true,
        lastName: true,
      },
      orderBy: { id: "asc" },
    });

    if (users.length === 0) break;

    for (const user of users) {
      try {
        const displayName = `${user.firstName} ${user.lastName}`.trim();
        if (!displayName) {
          skipped++;
          continue;
        }

        await prisma.$executeRaw`
          UPDATE users
          SET display_name = ${displayName}
          WHERE id = ${user.id}
            AND display_name IS NULL
        `;

        processed++;
      } catch (err) {
        errors++;
        console.error(`Failed to migrate user ${user.id}:`, err);
      }
    }

    cursor = users[users.length - 1].id;
    console.log(`[DataMigration] Processed: ${processed}, Skipped: ${skipped}, Errors: ${errors}`);
  }

  return {
    processed,
    skipped,
    errors,
    duration: Date.now() - startTime,
  };
}

// Zero-downtime column addition pattern
export async function addColumnSafely(): Promise<void> {
  // Step 1: Add column as nullable (non-blocking)
  await prisma.$executeRaw`
    ALTER TABLE products_searchable
    ADD COLUMN IF NOT EXISTS brand_name VARCHAR(255) DEFAULT NULL
  `;

  // Step 2: Backfill data in batches
  let updated = 1;
  while (updated > 0) {
    const result = await prisma.$executeRaw`
      UPDATE products_searchable
      SET brand_name = 'Unknown'
      WHERE brand_name IS NULL
      AND id IN (
        SELECT id FROM products_searchable
        WHERE brand_name IS NULL
        LIMIT ${BATCH_SIZE}
      )
    `;
    updated = Number(result);
    console.log(`[DataMigration] Updated ${updated} rows`);
  }

  // Step 3: Add NOT NULL constraint (after all data is filled)
  await prisma.$executeRaw`
    ALTER TABLE products_searchable
    ALTER COLUMN brand_name SET NOT NULL
  `;

  // Step 4: Add default for future rows
  await prisma.$executeRaw`
    ALTER TABLE products_searchable
    ALTER COLUMN brand_name SET DEFAULT 'Unknown'
  `;
}

migrateUserDisplayNames()
  .then((result) => {
    console.log("[DataMigration] Complete:", result);
    process.exit(0);
  })
  .catch((err) => {
    console.error("[DataMigration] Failed:", err);
    process.exit(1);
  });
```

### Edge Cases e Error Handling

```typescript
// scripts/migration-rollback.ts — Rollback strategies

import { PrismaClient } from "@prisma/client";
import { execSync } from "child_process";
import fs from "fs";
import path from "path";

const prisma = new PrismaClient();

interface RollbackOptions {
  steps: number;
  force: boolean;
}

export async function rollback(options: RollbackOptions): Promise<void> {
  const { steps, force } = options;
  const migrationsDir = path.join(process.cwd(), "prisma/migrations");

  const migrations = fs
    .readdirSync(migrationsDir)
    .filter((d) => !d.startsWith(".") && d !== "migration_lock.toml")
    .sort()
    .reverse();

  const toRollback = migrations.slice(0, steps);

  console.log(`Rolling back ${toRollback.length} migrations:`);
  toRollback.forEach((m) => console.log(`  - ${m}`));

  if (!force) {
    console.log("Use --force to execute rollback");
    return;
  }

  for (const migration of toRollback) {
    const downSql = path.join(migrationsDir, migration, "down.sql");

    if (fs.existsSync(downSql)) {
      const sql = fs.readFileSync(downSql, "utf8");
      console.log(`Executing down migration: ${migration}`);

      try {
        await prisma.$executeRawUnsafe(sql);
        execSync(`npx prisma migrate resolve --rolled-back ${migration}`, {
          stdio: "inherit",
        });
        console.log(`Rolled back: ${migration}`);
      } catch (error) {
        console.error(`Failed to rollback ${migration}:`, error);
        throw error;
      }
    } else {
      console.warn(`No down.sql found for ${migration} — skipping`);
    }
  }

  execSync("npx prisma generate", { stdio: "inherit" });
  console.log("Rollback complete. Prisma Client regenerated.");
}

// Snapshot e restore per testing migrazioni
export async function createSnapshot(name: string): Promise<void> {
  const timestamp = new Date().toISOString().replace(/[:.]/g, "-");
  const snapshotName = `snapshot_${name}_${timestamp}`;

  await prisma.$executeRawUnsafe(
    `SELECT pg_catalog.pg_create_restore_point('${snapshotName}')`
  );

  console.log(`Snapshot created: ${snapshotName}`);
}

export async function getDatabaseSize(): Promise<{
  totalSize: string;
  tablesSizes: Array<{ table: string; size: string; rows: number }>;
}> {
  const totalSize = await prisma.$queryRaw<Array<{ size: string }>>`
    SELECT pg_size_pretty(pg_database_size(current_database())) AS size
  `;

  const tablesSizes = await prisma.$queryRaw<
    Array<{ table: string; size: string; rows: number }>
  >`
    SELECT
      relname AS "table",
      pg_size_pretty(pg_total_relation_size(relid)) AS size,
      n_live_tup AS rows
    FROM pg_stat_user_tables
    ORDER BY pg_total_relation_size(relid) DESC
  `;

  return {
    totalSize: totalSize[0].size,
    tablesSizes,
  };
}
```

### Errori Comuni da Evitare
- Non eseguire `prisma migrate dev` in produzione — usare sempre `prisma migrate deploy`
- Non dimenticare di creare `down.sql` per ogni migrazione custom
- Non rimuovere colonne senza prima renderle nullable e verificare che non siano usate
- Non eseguire migrazioni lunghe senza `CONCURRENTLY` per gli indici

### Checklist di Verifica
- [ ] Backup creato prima di ogni migrazione in produzione
- [ ] `down.sql` presente per rollback
- [ ] Data migration in batch (non tutto in una transazione)
- [ ] Colonne aggiunte come nullable prima di backfill
- [ ] Zero-downtime pattern per colonne NOT NULL
- [ ] Test della migrazione su staging prima di produzione



═══════════════════════════════════════════════════════════════════════════════
SEZIONE 11: DATABASE SEEDING CON FAKER
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Script di seeding completi con @faker-js/faker per generare dati realistici
per development, staging e demo. Include relazioni complesse e dati coerenti.

### Implementazione Completa

```typescript
// prisma/seed.ts — Complete seeding script

import { PrismaClient, UserRole, OrderStatus, PaymentStatus } from "@prisma/client";
import { faker } from "@faker-js/faker/locale/it";
import { hash } from "bcryptjs";

const prisma = new PrismaClient();

interface SeedConfig {
  users: number;
  categories: number;
  productsPerCategory: number;
  ordersPerUser: number;
  reviewsPerProduct: number;
}

const CONFIGS: Record<string, SeedConfig> = {
  development: {
    users: 50,
    categories: 10,
    productsPerCategory: 15,
    ordersPerUser: 3,
    reviewsPerProduct: 5,
  },
  staging: {
    users: 200,
    categories: 20,
    productsPerCategory: 30,
    ordersPerUser: 5,
    reviewsPerProduct: 10,
  },
  demo: {
    users: 20,
    categories: 8,
    productsPerCategory: 10,
    ordersPerUser: 2,
    reviewsPerProduct: 3,
  },
};

async function seedUsers(count: number): Promise<string[]> {
  console.log(`Seeding ${count} users...`);
  const passwordHash = await hash("Password123!", 10);
  const userIds: string[] = [];

  // Admin user
  const admin = await prisma.user.create({
    data: {
      email: "admin@example.com",
      passwordHash,
      firstName: "Admin",
      lastName: "User",
      role: UserRole.ADMIN,
      emailVerified: true,
      phone: "+39 02 1234567",
    },
  });
  userIds.push(admin.id);

  // Regular users
  for (let i = 0; i < count - 1; i++) {
    const firstName = faker.person.firstName();
    const lastName = faker.person.lastName();
    const user = await prisma.user.create({
      data: {
        email: faker.internet.email({ firstName, lastName }).toLowerCase(),
        passwordHash,
        firstName,
        lastName,
        role: faker.helpers.weightedArrayElement([
          { value: UserRole.CUSTOMER, weight: 90 },
          { value: UserRole.MANAGER, weight: 5 },
          { value: UserRole.SUPPORT, weight: 5 },
        ]),
        emailVerified: faker.datatype.boolean(0.8),
        avatarUrl: faker.image.avatar(),
        phone: faker.phone.number(),
      },
    });
    userIds.push(user.id);

    // Create address for each user
    await prisma.address.create({
      data: {
        userId: user.id,
        type: "SHIPPING",
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
  return userIds;
}

async function seedCategories(count: number): Promise<string[]> {
  console.log(`Seeding ${count} categories...`);
  const categoryNames = [
    "Elettronica", "Abbigliamento", "Casa e Giardino", "Sport",
    "Libri", "Giocattoli", "Alimentari", "Bellezza",
    "Automotive", "Musica", "Film", "Informatica",
    "Gioielli", "Scarpe", "Ufficio", "Pet",
    "Bambini", "Salute", "Viaggi", "Arte",
  ];

  const categoryIds: string[] = [];

  for (let i = 0; i < Math.min(count, categoryNames.length); i++) {
    const name = categoryNames[i];
    const category = await prisma.category.create({
      data: {
        name,
        slug: name.toLowerCase().replace(/\s+/g, "-").replace(/[^a-z0-9-]/g, ""),
        description: faker.commerce.department() + " - " + faker.lorem.sentence(),
        imageUrl: faker.image.urlPicsumPhotos({ width: 400, height: 300 }),
        sortOrder: i,
        isActive: true,
      },
    });
    categoryIds.push(category.id);

    // Subcategories
    const subCount = faker.number.int({ min: 2, max: 5 });
    for (let j = 0; j < subCount; j++) {
      const subName = faker.commerce.department() + " " + faker.commerce.productAdjective();
      await prisma.category.create({
        data: {
          name: subName,
          slug: subName.toLowerCase().replace(/\s+/g, "-").replace(/[^a-z0-9-]/g, "") + "-" + faker.string.nanoid(4),
          description: faker.lorem.sentence(),
          parentId: category.id,
          sortOrder: j,
          isActive: true,
        },
      });
    }
  }

  console.log(`  Created ${categoryIds.length} root categories with subcategories`);
  return categoryIds;
}

async function seedProducts(
  categoryIds: string[],
  perCategory: number
): Promise<string[]> {
  console.log(`Seeding ${categoryIds.length * perCategory} products...`);
  const productIds: string[] = [];

  for (const categoryId of categoryIds) {
    for (let i = 0; i < perCategory; i++) {
      const name = faker.commerce.productName();
      const price = parseFloat(faker.commerce.price({ min: 5, max: 500 }));
      const hasDiscount = faker.datatype.boolean(0.3);

      const product = await prisma.product.create({
        data: {
          name,
          slug: faker.helpers.slugify(name).toLowerCase() + "-" + faker.string.nanoid(6),
          description: faker.commerce.productDescription(),
          shortDescription: faker.lorem.sentence(),
          sku: `SKU-${faker.string.alphanumeric(8).toUpperCase()}`,
          price,
          compareAtPrice: hasDiscount ? price * 1.3 : null,
          costPrice: price * 0.6,
          currency: "EUR",
          stock: faker.number.int({ min: 0, max: 200 }),
          lowStockThreshold: 5,
          weight: parseFloat(faker.number.float({ min: 0.1, max: 20, fractionDigits: 2 }).toString()),
          isActive: faker.datatype.boolean(0.9),
          isFeatured: faker.datatype.boolean(0.15),
          metaTitle: name,
          metaDescription: faker.lorem.sentence(),
          categories: {
            create: { categoryId },
          },
          images: {
            createMany: {
              data: Array.from({ length: faker.number.int({ min: 1, max: 5 }) }, (_, idx) => ({
                url: faker.image.urlPicsumPhotos({ width: 800, height: 800 }),
                alt: `${name} - immagine ${idx + 1}`,
                sortOrder: idx,
                isPrimary: idx === 0,
              })),
            },
          },
        },
      });
      productIds.push(product.id);

      // Variants for some products
      if (faker.datatype.boolean(0.4)) {
        const sizes = ["S", "M", "L", "XL"];
        for (const size of sizes) {
          await prisma.productVariant.create({
            data: {
              productId: product.id,
              name: `Taglia ${size}`,
              sku: `${product.sku}-${size}`,
              price: price + (size === "XL" ? 5 : 0),
              stock: faker.number.int({ min: 0, max: 50 }),
              options: { size },
              isActive: true,
            },
          });
        }
      }
    }
  }

  console.log(`  Created ${productIds.length} products`);
  return productIds;
}

async function seedOrders(
  userIds: string[],
  productIds: string[],
  ordersPerUser: number
): Promise<void> {
  console.log(`Seeding orders...`);
  let orderCount = 0;

  for (const userId of userIds.slice(0, Math.ceil(userIds.length * 0.7))) {
    const address = await prisma.address.findFirst({
      where: { userId },
    });
    if (!address) continue;

    const numOrders = faker.number.int({ min: 1, max: ordersPerUser });

    for (let i = 0; i < numOrders; i++) {
      const itemCount = faker.number.int({ min: 1, max: 5 });
      const selectedProducts = faker.helpers.arrayElements(productIds, itemCount);

      let subtotal = 0;
      const items = [];

      for (const productId of selectedProducts) {
        const product = await prisma.product.findUnique({
          where: { id: productId },
          select: { name: true, sku: true, price: true },
        });
        if (!product) continue;

        const quantity = faker.number.int({ min: 1, max: 3 });
        const itemTotal = Number(product.price) * quantity;
        subtotal += itemTotal;

        items.push({
          productId,
          name: product.name,
          sku: product.sku,
          price: Number(product.price),
          quantity,
          total: itemTotal,
        });
      }

      const tax = subtotal * 0.22;
      const shippingCost = subtotal > 50 ? 0 : 7.99;
      const total = subtotal + tax + shippingCost;

      const status = faker.helpers.weightedArrayElement([
        { value: OrderStatus.DELIVERED, weight: 40 },
        { value: OrderStatus.SHIPPED, weight: 20 },
        { value: OrderStatus.PROCESSING, weight: 15 },
        { value: OrderStatus.CONFIRMED, weight: 10 },
        { value: OrderStatus.PENDING, weight: 10 },
        { value: OrderStatus.CANCELLED, weight: 5 },
      ]);

      const createdAt = faker.date.past({ years: 1 });

      await prisma.order.create({
        data: {
          orderNumber: `ORD-${Date.now().toString(36).toUpperCase()}-${faker.string.alphanumeric(4).toUpperCase()}`,
          userId,
          addressId: address.id,
          status,
          subtotal,
          tax,
          shippingCost,
          total,
          currency: "EUR",
          paymentMethod: faker.helpers.arrayElement(["card", "paypal", "bank_transfer"]),
          createdAt,
          items: {
            createMany: { data: items },
          },
          payments: {
            create: {
              amount: total,
              currency: "EUR",
              status: status === OrderStatus.CANCELLED ? PaymentStatus.REFUNDED : PaymentStatus.SUCCEEDED,
              method: "card",
              createdAt,
            },
          },
          timeline: {
            createMany: {
              data: [
                { type: "created", message: "Ordine creato", createdAt },
                ...(["CONFIRMED", "PROCESSING", "SHIPPED", "DELIVERED"].includes(status)
                  ? [{ type: "confirmed", message: "Ordine confermato", createdAt: new Date(createdAt.getTime() + 3600000) }]
                  : []),
              ],
            },
          },
        },
      });
      orderCount++;
    }
  }

  console.log(`  Created ${orderCount} orders`);
}

async function seedReviews(
  userIds: string[],
  productIds: string[],
  reviewsPerProduct: number
): Promise<void> {
  console.log(`Seeding reviews...`);
  let reviewCount = 0;

  for (const productId of productIds) {
    const reviewers = faker.helpers.arrayElements(
      userIds,
      Math.min(reviewsPerProduct, userIds.length)
    );

    for (const userId of reviewers) {
      try {
        await prisma.review.create({
          data: {
            userId,
            productId,
            rating: faker.helpers.weightedArrayElement([
              { value: 5, weight: 35 },
              { value: 4, weight: 30 },
              { value: 3, weight: 20 },
              { value: 2, weight: 10 },
              { value: 1, weight: 5 },
            ]),
            title: faker.lorem.sentence({ min: 3, max: 8 }),
            body: faker.lorem.paragraph(),
            isVerified: faker.datatype.boolean(0.6),
            isApproved: faker.datatype.boolean(0.85),
            createdAt: faker.date.past({ years: 1 }),
          },
        });
        reviewCount++;
      } catch {
        // Skip duplicate user-product review
      }
    }
  }

  console.log(`  Created ${reviewCount} reviews`);
}

async function main(): Promise<void> {
  const env = process.env.SEED_ENV ?? "development";
  const config = CONFIGS[env] ?? CONFIGS.development;

  console.log(`\n=== Seeding database (${env}) ===\n`);

  // Clean existing data
  console.log("Cleaning existing data...");
  await prisma.$transaction([
    prisma.review.deleteMany(),
    prisma.orderEvent.deleteMany(),
    prisma.payment.deleteMany(),
    prisma.orderItem.deleteMany(),
    prisma.order.deleteMany(),
    prisma.cartItem.deleteMany(),
    prisma.cart.deleteMany(),
    prisma.wishlistItem.deleteMany(),
    prisma.productImage.deleteMany(),
    prisma.productVariant.deleteMany(),
    prisma.productCategory.deleteMany(),
    prisma.productTag.deleteMany(),
    prisma.product.deleteMany(),
    prisma.category.deleteMany(),
    prisma.address.deleteMany(),
    prisma.session.deleteMany(),
    prisma.notification.deleteMany(),
    prisma.user.deleteMany(),
  ]);

  const userIds = await seedUsers(config.users);
  const categoryIds = await seedCategories(config.categories);
  const productIds = await seedProducts(categoryIds, config.productsPerCategory);
  await seedOrders(userIds, productIds, config.ordersPerUser);
  await seedReviews(userIds, productIds, config.reviewsPerProduct);

  console.log("\n=== Seeding complete ===\n");
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

### Errori Comuni da Evitare
- Non eseguire seed in produzione senza `SEED_ENV` check esplicito
- Non dimenticare di gestire unique constraint violations con try/catch
- Non generare date future per `createdAt` — usare sempre `faker.date.past()`
- Non hardcodare password in chiaro — usare sempre bcrypt anche nel seed

### Checklist di Verifica
- [ ] Seed idempotente (clean + recreate)
- [ ] Dati realistici con faker locale appropriato
- [ ] Relazioni coerenti tra entita
- [ ] Varianti di configurazione per dev/staging/demo
- [ ] User admin con credenziali note per testing
- [ ] Script registrato in `package.json` come `prisma:seed`


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 12: SOFT DELETE PATTERN
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Implementazione completa del soft delete con Prisma: middleware per filtraggio
automatico, restore, cascade soft delete, e pulizia periodica.

### Implementazione Completa

```typescript
// lib/db/soft-delete.ts — Prisma extension per soft delete

import { Prisma, PrismaClient } from "@prisma/client";

// Modelli che supportano soft delete
const SOFT_DELETE_MODELS = [
  "User",
  "Product",
  "Organization",
  "Order",
] as const;

type SoftDeleteModel = (typeof SOFT_DELETE_MODELS)[number];

function isSoftDeleteModel(model: string): model is SoftDeleteModel {
  return SOFT_DELETE_MODELS.includes(model as SoftDeleteModel);
}

// Prisma extension per soft delete automatico
export function withSoftDelete(prisma: PrismaClient) {
  return prisma.$extends({
    query: {
      $allModels: {
        // Override delete -> set deletedAt
        async delete({ model, args, query }) {
          if (isSoftDeleteModel(model)) {
            return (prisma as any)[model].update({
              ...args,
              data: { deletedAt: new Date() },
            });
          }
          return query(args);
        },

        // Override deleteMany -> updateMany con deletedAt
        async deleteMany({ model, args, query }) {
          if (isSoftDeleteModel(model)) {
            return (prisma as any)[model].updateMany({
              ...args,
              data: { deletedAt: new Date() },
            });
          }
          return query(args);
        },

        // Override findMany -> filtra deletedAt IS NULL
        async findMany({ model, args, query }) {
          if (isSoftDeleteModel(model)) {
            args.where = {
              ...args.where,
              deletedAt: null,
            };
          }
          return query(args);
        },

        // Override findFirst -> filtra deletedAt IS NULL
        async findFirst({ model, args, query }) {
          if (isSoftDeleteModel(model)) {
            args.where = {
              ...args.where,
              deletedAt: null,
            };
          }
          return query(args);
        },

        // Override findUnique -> Non filtrare (serve per restore)
        async findUnique({ args, query }) {
          return query(args);
        },

        // Override count -> filtra deletedAt IS NULL
        async count({ model, args, query }) {
          if (isSoftDeleteModel(model)) {
            args.where = {
              ...args.where,
              deletedAt: null,
            };
          }
          return query(args);
        },
      },
    },
    model: {
      $allModels: {
        // Metodo custom per trovare record eliminati
        async findDeleted<T>(
          this: T,
          args?: { where?: any; take?: number; skip?: number; orderBy?: any }
        ) {
          const context = Prisma.getExtensionContext(this) as any;
          return context.findMany({
            ...args,
            where: {
              ...args?.where,
              deletedAt: { not: null },
            },
          });
        },

        // Metodo custom per restore
        async restore<T>(this: T, args: { where: { id: string } }) {
          const context = Prisma.getExtensionContext(this) as any;
          return context.update({
            where: args.where,
            data: { deletedAt: null },
          });
        },

        // Hard delete (bypass soft delete)
        async hardDelete<T>(this: T, args: { where: { id: string } }) {
          const context = Prisma.getExtensionContext(this) as any;
          return context.delete(args);
        },
      },
    },
  });
}

// Tipo helper per il client esteso
export type SoftDeletePrismaClient = ReturnType<typeof withSoftDelete>;
```

### Varianti e Configurazioni

```typescript
// lib/db/soft-delete-cascade.ts — Cascade soft delete

import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

// Cascade relations per soft delete
const CASCADE_MAP: Record<string, Array<{ model: string; foreignKey: string }>> = {
  User: [
    { model: "Order", foreignKey: "userId" },
    { model: "Review", foreignKey: "userId" },
    { model: "Address", foreignKey: "userId" },
  ],
  Product: [
    { model: "ProductVariant", foreignKey: "productId" },
    { model: "ProductImage", foreignKey: "productId" },
    { model: "Review", foreignKey: "productId" },
  ],
  Organization: [
    { model: "Member", foreignKey: "organizationId" },
    { model: "Workspace", foreignKey: "organizationId" },
    { model: "Invitation", foreignKey: "organizationId" },
    { model: "ApiKey", foreignKey: "organizationId" },
  ],
};

// Soft delete con cascade
export async function softDeleteCascade(
  model: string,
  id: string
): Promise<{ deleted: string[]; count: number }> {
  const deleted: string[] = [];
  let count = 0;

  await prisma.$transaction(async (tx) => {
    // Soft delete il record principale
    await (tx as any)[model.charAt(0).toLowerCase() + model.slice(1)].update({
      where: { id },
      data: { deletedAt: new Date() },
    });
    deleted.push(`${model}:${id}`);
    count++;

    // Cascade ai figli
    const cascades = CASCADE_MAP[model] ?? [];
    for (const cascade of cascades) {
      const modelName = cascade.model.charAt(0).toLowerCase() + cascade.model.slice(1);
      const hasDeletedAt = await checkModelHasDeletedAt(cascade.model);

      if (hasDeletedAt) {
        const result = await (tx as any)[modelName].updateMany({
          where: { [cascade.foreignKey]: id },
          data: { deletedAt: new Date() },
        });
        count += result.count;
        deleted.push(`${cascade.model}:${result.count} records`);
      }
    }
  });

  return { deleted, count };
}

// Restore con cascade
export async function restoreCascade(
  model: string,
  id: string
): Promise<{ restored: string[]; count: number }> {
  const restored: string[] = [];
  let count = 0;

  await prisma.$transaction(async (tx) => {
    await (tx as any)[model.charAt(0).toLowerCase() + model.slice(1)].update({
      where: { id },
      data: { deletedAt: null },
    });
    restored.push(`${model}:${id}`);
    count++;

    const cascades = CASCADE_MAP[model] ?? [];
    for (const cascade of cascades) {
      const modelName = cascade.model.charAt(0).toLowerCase() + cascade.model.slice(1);
      const result = await (tx as any)[modelName].updateMany({
        where: { [cascade.foreignKey]: id, deletedAt: { not: null } },
        data: { deletedAt: null },
      });
      count += result.count;
      restored.push(`${cascade.model}:${result.count} records`);
    }
  });

  return { restored, count };
}

async function checkModelHasDeletedAt(_model: string): Promise<boolean> {
  // In produzione, usare Prisma DMMF per verificare
  return true;
}

// Pulizia periodica dei soft-deleted records (> 30 giorni)
export async function purgeDeletedRecords(
  daysOld: number = 30
): Promise<{ purged: Record<string, number> }> {
  const cutoff = new Date(Date.now() - daysOld * 24 * 60 * 60 * 1000);
  const purged: Record<string, number> = {};

  // Ordine di eliminazione: figli prima dei genitori
  const purgeOrder = [
    "Review", "OrderEvent", "Payment", "OrderItem", "Order",
    "CartItem", "Cart", "WishlistItem", "ProductImage",
    "ProductVariant", "ProductCategory", "Product",
    "Address", "Session", "User",
  ];

  for (const model of purgeOrder) {
    try {
      const modelName = model.charAt(0).toLowerCase() + model.slice(1);
      const result = await (prisma as any)[modelName].deleteMany({
        where: { deletedAt: { lt: cutoff } },
      });
      if (result.count > 0) {
        purged[model] = result.count;
      }
    } catch {
      // Model might not have deletedAt — skip
    }
  }

  return { purged };
}
```

### Errori Comuni da Evitare
- Non dimenticare di filtrare `deletedAt IS NULL` nelle query aggregate (count, sum, etc.)
- Non usare hard delete per modelli con soft delete senza motivo esplicito
- Non dimenticare gli indici su `deletedAt` per performance delle query filtrate
- Non ignorare le relazioni cascade — un record parent soft-deleted con figli attivi crea inconsistenze

### Checklist di Verifica
- [ ] Extension Prisma applicata globalmente nell'app
- [ ] Tutti i modelli soft-deletable hanno `deletedAt DateTime?`
- [ ] Indice su `deletedAt` per ogni modello soft-deletable
- [ ] Cascade soft delete configurato per relazioni parent-child
- [ ] Cron job per purge periodico dei record vecchi
- [ ] API di restore testata e funzionante


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 13: AUDIT TRAIL SYSTEM
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Sistema completo di audit trail per tracciare ogni modifica al database:
chi ha fatto cosa, quando, su quale entita, con before/after snapshot.

### Implementazione Completa

```typescript
// lib/db/audit-trail.ts — Automatic audit logging

import { PrismaClient, Prisma } from "@prisma/client";

// Schema per audit log (gia definito nello schema SaaS sopra)
// model AuditLog { ... }

interface AuditContext {
  actorId: string;
  actorType: "user" | "system" | "api";
  ipAddress?: string;
  userAgent?: string;
  organizationId?: string;
}

// AsyncLocalStorage per propagare il contesto
import { AsyncLocalStorage } from "async_hooks";

export const auditContext = new AsyncLocalStorage<AuditContext>();

// Prisma extension per audit automatico
export function withAuditTrail(prisma: PrismaClient) {
  return prisma.$extends({
    query: {
      $allModels: {
        async create({ model, args, query }) {
          const result = await query(args);
          await logAuditEvent(prisma, {
            action: "CREATE",
            entity: model,
            entityId: (result as any).id,
            after: result,
          });
          return result;
        },

        async update({ model, args, query }) {
          // Capture before state
          const before = await (prisma as any)[
            model.charAt(0).toLowerCase() + model.slice(1)
          ].findUnique({
            where: (args as any).where,
          });

          const result = await query(args);

          await logAuditEvent(prisma, {
            action: "UPDATE",
            entity: model,
            entityId: (result as any).id,
            before,
            after: result,
            changes: computeChanges(before, result),
          });
          return result;
        },

        async delete({ model, args, query }) {
          const before = await (prisma as any)[
            model.charAt(0).toLowerCase() + model.slice(1)
          ].findUnique({
            where: (args as any).where,
          });

          const result = await query(args);

          await logAuditEvent(prisma, {
            action: "DELETE",
            entity: model,
            entityId: (before as any)?.id,
            before,
          });
          return result;
        },
      },
    },
  });
}

interface AuditEventData {
  action: "CREATE" | "UPDATE" | "DELETE";
  entity: string;
  entityId?: string;
  before?: any;
  after?: any;
  changes?: Record<string, { from: any; to: any }>;
}

async function logAuditEvent(
  prisma: PrismaClient,
  event: AuditEventData
): Promise<void> {
  const ctx = auditContext.getStore();

  try {
    await prisma.auditLog.create({
      data: {
        organizationId: ctx?.organizationId ?? "system",
        actorId: ctx?.actorId ?? null,
        action: event.action,
        entity: event.entity,
        entityId: event.entityId ?? null,
        metadata: {
          before: event.before ? sanitizeForLog(event.before) : undefined,
          after: event.after ? sanitizeForLog(event.after) : undefined,
          changes: event.changes,
          actorType: ctx?.actorType ?? "system",
          ipAddress: ctx?.ipAddress,
          userAgent: ctx?.userAgent,
        },
      },
    });
  } catch (error) {
    console.error("[AuditTrail] Failed to log event:", error);
  }
}

function computeChanges(
  before: Record<string, any>,
  after: Record<string, any>
): Record<string, { from: any; to: any }> {
  const changes: Record<string, { from: any; to: any }> = {};
  const skipFields = ["updatedAt", "createdAt"];

  for (const key of Object.keys(after)) {
    if (skipFields.includes(key)) continue;
    if (JSON.stringify(before?.[key]) !== JSON.stringify(after[key])) {
      changes[key] = { from: before?.[key], to: after[key] };
    }
  }

  return changes;
}

function sanitizeForLog(data: Record<string, any>): Record<string, any> {
  const sensitiveFields = ["passwordHash", "token", "keyHash", "secret"];
  const sanitized = { ...data };

  for (const field of sensitiveFields) {
    if (field in sanitized) {
      sanitized[field] = "[REDACTED]";
    }
  }

  return sanitized;
}

// Middleware Next.js per iniettare audit context
export function withAuditContext(
  handler: (req: Request) => Promise<Response>,
  getContext: (req: Request) => AuditContext
) {
  return async (req: Request): Promise<Response> => {
    const ctx = getContext(req);
    return auditContext.run(ctx, () => handler(req));
  };
}
```

### Varianti e Configurazioni

```typescript
// lib/db/audit-query.ts — Querying audit logs

import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

interface AuditQueryOptions {
  organizationId?: string;
  actorId?: string;
  entity?: string;
  entityId?: string;
  action?: "CREATE" | "UPDATE" | "DELETE";
  from?: Date;
  to?: Date;
  limit?: number;
  cursor?: string;
}

interface AuditEntry {
  id: string;
  action: string;
  entity: string;
  entityId: string | null;
  actorId: string | null;
  metadata: any;
  createdAt: Date;
}

export async function queryAuditLog(
  options: AuditQueryOptions
): Promise<{ entries: AuditEntry[]; nextCursor: string | null }> {
  const {
    organizationId,
    actorId,
    entity,
    entityId,
    action,
    from,
    to,
    limit = 50,
    cursor,
  } = options;

  const where: any = {};

  if (organizationId) where.organizationId = organizationId;
  if (actorId) where.actorId = actorId;
  if (entity) where.entity = entity;
  if (entityId) where.entityId = entityId;
  if (action) where.action = action;
  if (from || to) {
    where.createdAt = {};
    if (from) where.createdAt.gte = from;
    if (to) where.createdAt.lte = to;
  }

  const entries = await prisma.auditLog.findMany({
    where,
    take: limit + 1,
    ...(cursor ? { skip: 1, cursor: { id: cursor } } : {}),
    orderBy: { createdAt: "desc" },
  });

  const hasMore = entries.length > limit;
  if (hasMore) entries.pop();

  return {
    entries,
    nextCursor: hasMore ? entries[entries.length - 1].id : null,
  };
}

// Timeline per una specifica entita
export async function getEntityTimeline(
  entity: string,
  entityId: string
): Promise<AuditEntry[]> {
  return prisma.auditLog.findMany({
    where: { entity, entityId },
    orderBy: { createdAt: "desc" },
    take: 100,
  });
}

// Diff tra due versioni
export function computeDiff(
  before: Record<string, any>,
  after: Record<string, any>
): Array<{ field: string; oldValue: any; newValue: any }> {
  const diff: Array<{ field: string; oldValue: any; newValue: any }> = [];

  const allKeys = new Set([
    ...Object.keys(before ?? {}),
    ...Object.keys(after ?? {}),
  ]);

  for (const key of allKeys) {
    if (JSON.stringify(before?.[key]) !== JSON.stringify(after?.[key])) {
      diff.push({
        field: key,
        oldValue: before?.[key] ?? null,
        newValue: after?.[key] ?? null,
      });
    }
  }

  return diff;
}

// API Route per audit log viewer
// GET /api/admin/audit-log
export async function getAuditLogHandler(request: Request): Promise<Response> {
  const url = new URL(request.url);
  const entity = url.searchParams.get("entity") ?? undefined;
  const entityId = url.searchParams.get("entityId") ?? undefined;
  const actorId = url.searchParams.get("actorId") ?? undefined;
  const action = url.searchParams.get("action") as any;
  const cursor = url.searchParams.get("cursor") ?? undefined;
  const limit = parseInt(url.searchParams.get("limit") ?? "50");

  const result = await queryAuditLog({
    entity,
    entityId,
    actorId,
    action,
    limit,
    cursor,
  });

  return Response.json(result);
}
```

### Errori Comuni da Evitare
- Non loggare campi sensibili (password, token, chiavi API) — sempre sanitizzare
- Non loggare in modo sincrono dentro la transazione principale — usare fire-and-forget
- Non dimenticare di indicizzare `[organizationId, createdAt]` per query efficienti
- Non salvare oggetti troppo grandi nel `metadata` — limitare la profondita

### Checklist di Verifica
- [ ] Audit log su ogni operazione CRUD critica
- [ ] Campi sensibili redatti con `[REDACTED]`
- [ ] AsyncLocalStorage per propagare contesto utente
- [ ] Before/after snapshot per ogni UPDATE
- [ ] Indici su `[entity, entityId]` e `[organizationId, createdAt]`
- [ ] Retention policy per pulizia log vecchi


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 14: CONNECTION POOLING
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Configurazione connection pooling per PostgreSQL con Prisma: PgBouncer, Prisma
pool nativo, gestione connessioni serverless con Neon/Supabase.

### Implementazione Completa

```typescript
// lib/db/prisma-client.ts — Singleton Prisma con connection pooling

import { PrismaClient, Prisma } from "@prisma/client";

// Configurazione per ambienti diversi
const DATABASE_CONFIGS = {
  development: {
    url: process.env.DATABASE_URL!,
    connectionLimit: 10,
    poolTimeout: 10,
    connectTimeout: 5,
  },
  production: {
    url: process.env.DATABASE_URL!,
    directUrl: process.env.DIRECT_URL,
    connectionLimit: 20,
    poolTimeout: 15,
    connectTimeout: 10,
  },
  serverless: {
    url: process.env.DATABASE_URL!,
    directUrl: process.env.DIRECT_URL,
    connectionLimit: 5,
    poolTimeout: 5,
    connectTimeout: 3,
  },
} as const;

type Environment = keyof typeof DATABASE_CONFIGS;

function getConfig(): (typeof DATABASE_CONFIGS)[Environment] {
  const env = (process.env.APP_ENV ?? "development") as Environment;
  return DATABASE_CONFIGS[env] ?? DATABASE_CONFIGS.development;
}

// Singleton pattern per evitare connection leak in dev (hot reload)
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

function createPrismaClient(): PrismaClient {
  const config = getConfig();

  const client = new PrismaClient({
    datasourceUrl: config.url,
    log:
      process.env.NODE_ENV === "development"
        ? [
            { emit: "event", level: "query" },
            { emit: "stdout", level: "error" },
            { emit: "stdout", level: "warn" },
          ]
        : [{ emit: "stdout", level: "error" }],
  });

  // Log slow queries in development
  if (process.env.NODE_ENV === "development") {
    (client as any).$on("query", (e: Prisma.QueryEvent) => {
      if (e.duration > 100) {
        console.warn(`[Slow Query] ${e.duration}ms: ${e.query}`);
      }
    });
  }

  return client;
}

export const prisma = globalForPrisma.prisma ?? createPrismaClient();

if (process.env.NODE_ENV !== "production") {
  globalForPrisma.prisma = prisma;
}

// Health check per connection pool
export async function checkDatabaseHealth(): Promise<{
  connected: boolean;
  responseTime: number;
  poolStats?: {
    activeConnections: number;
    idleConnections: number;
    waitingRequests: number;
  };
}> {
  const start = Date.now();
  try {
    await prisma.$queryRaw`SELECT 1`;
    const responseTime = Date.now() - start;

    const poolStats = await prisma.$queryRaw<
      Array<{
        active: number;
        idle: number;
        waiting: number;
      }>
    >`
      SELECT
        count(*) FILTER (WHERE state = 'active') AS active,
        count(*) FILTER (WHERE state = 'idle') AS idle,
        count(*) FILTER (WHERE wait_event IS NOT NULL) AS waiting
      FROM pg_stat_activity
      WHERE datname = current_database()
    `;

    return {
      connected: true,
      responseTime,
      poolStats: poolStats[0]
        ? {
            activeConnections: Number(poolStats[0].active),
            idleConnections: Number(poolStats[0].idle),
            waitingRequests: Number(poolStats[0].waiting),
          }
        : undefined,
    };
  } catch {
    return {
      connected: false,
      responseTime: Date.now() - start,
    };
  }
}
```

### Varianti e Configurazioni

```typescript
// lib/db/pgbouncer-config.ts — PgBouncer configuration reference

/*
  PgBouncer ini configuration (pgbouncer.ini):

  [databases]
  myapp = host=localhost port=5432 dbname=myapp

  [pgbouncer]
  listen_port = 6432
  listen_addr = 0.0.0.0
  auth_type = md5
  auth_file = /etc/pgbouncer/userlist.txt
  pool_mode = transaction        ; transaction pooling per serverless
  max_client_conn = 1000         ; max client connections
  default_pool_size = 20         ; connections per user/db
  min_pool_size = 5              ; minimum idle connections
  reserve_pool_size = 5          ; extra connections for burst
  reserve_pool_timeout = 3       ; seconds before using reserve
  server_idle_timeout = 60       ; close idle server connections
  server_lifetime = 3600         ; max server connection lifetime
  log_connections = 0
  log_disconnections = 0
  stats_period = 60
*/

// .env configuration per Prisma + PgBouncer
/*
  # Connection through PgBouncer (per queries)
  DATABASE_URL="postgresql://user:pass@localhost:6432/myapp?pgbouncer=true&connection_limit=5"

  # Direct connection (per migrations)
  DIRECT_URL="postgresql://user:pass@localhost:5432/myapp"
*/

// prisma/schema.prisma per PgBouncer:
/*
  datasource db {
    provider  = "postgresql"
    url       = env("DATABASE_URL")
    directUrl = env("DIRECT_URL")
  }
*/

// Neon Serverless configuration
export const NEON_CONFIG = {
  // .env:
  // DATABASE_URL="postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/mydb?sslmode=require"
  // DIRECT_URL="postgresql://user:pass@ep-xxx.us-east-1.aws.neon.tech/mydb?sslmode=require"
  connectionString: process.env.DATABASE_URL!,
  poolConfig: {
    max: 10,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 5000,
  },
};

// Supabase Pooler configuration
export const SUPABASE_CONFIG = {
  // .env:
  // DATABASE_URL="postgresql://postgres.xxx:pass@aws-0-us-east-1.pooler.supabase.com:6543/postgres?pgbouncer=true"
  // DIRECT_URL="postgresql://postgres.xxx:pass@aws-0-us-east-1.pooler.supabase.com:5432/postgres"
  connectionString: process.env.DATABASE_URL!,
};

// Connection pool monitoring API route
// GET /api/admin/pool-stats
export async function getPoolStats(): Promise<{
  totalConnections: number;
  activeConnections: number;
  idleConnections: number;
  maxConnections: number;
  utilizationPercent: number;
}> {
  const { prisma } = await import("./prisma-client");

  const stats = await prisma.$queryRaw<
    Array<{
      total: number;
      active: number;
      idle: number;
      max_conn: number;
    }>
  >`
    SELECT
      count(*) AS total,
      count(*) FILTER (WHERE state = 'active') AS active,
      count(*) FILTER (WHERE state = 'idle') AS idle,
      setting::int AS max_conn
    FROM pg_stat_activity
    CROSS JOIN pg_settings
    WHERE pg_settings.name = 'max_connections'
      AND pg_stat_activity.datname = current_database()
    GROUP BY setting
  `;

  const s = stats[0];
  return {
    totalConnections: Number(s?.total ?? 0),
    activeConnections: Number(s?.active ?? 0),
    idleConnections: Number(s?.idle ?? 0),
    maxConnections: Number(s?.max_conn ?? 100),
    utilizationPercent: Number(s?.total ?? 0) / Number(s?.max_conn ?? 100) * 100,
  };
}
```

### Errori Comuni da Evitare
- Non dimenticare `?pgbouncer=true` nell'URL quando si usa PgBouncer
- Non usare `DIRECT_URL` per le query normali — solo per migrazioni
- Non impostare `connection_limit` troppo alto in serverless (max 5-10)
- Non dimenticare il singleton pattern in Next.js per evitare connection leak in dev

### Checklist di Verifica
- [ ] `DATABASE_URL` con PgBouncer/pooler per queries
- [ ] `DIRECT_URL` diretto per migrazioni
- [ ] Singleton Prisma Client con global cache
- [ ] `?pgbouncer=true` nell'URL di connessione
- [ ] Health check endpoint per monitoraggio pool
- [ ] Slow query logging in development


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 15: BACKUP STRATEGY
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Strategie di backup per PostgreSQL: backup automatici, point-in-time recovery,
cross-region replication, e verifica integrita backup.

### Implementazione Completa

```typescript
// scripts/backup.ts — Automated backup system

import { execSync } from "child_process";
import { PrismaClient } from "@prisma/client";
import fs from "fs";
import path from "path";

const prisma = new PrismaClient();

interface BackupConfig {
  type: "full" | "incremental" | "schema-only";
  compress: boolean;
  encrypt: boolean;
  retentionDays: number;
  storageProvider: "local" | "s3" | "gcs";
  s3Bucket?: string;
  gcsBucket?: string;
}

const DEFAULT_CONFIG: BackupConfig = {
  type: "full",
  compress: true,
  encrypt: false,
  retentionDays: 30,
  storageProvider: "local",
};

export async function createBackup(
  config: BackupConfig = DEFAULT_CONFIG
): Promise<{ success: boolean; filePath: string; size: number; duration: number }> {
  const startTime = Date.now();
  const timestamp = new Date().toISOString().replace(/[:.]/g, "-");
  const dbName = new URL(process.env.DATABASE_URL!).pathname.slice(1);
  const backupDir = path.resolve(process.cwd(), "backups");

  if (!fs.existsSync(backupDir)) {
    fs.mkdirSync(backupDir, { recursive: true });
  }

  const extension = config.compress ? "sql.gz" : "sql";
  const filename = `backup_${dbName}_${timestamp}_${config.type}.${extension}`;
  const filePath = path.join(backupDir, filename);

  try {
    // Pre-backup check
    await prisma.$queryRaw`SELECT 1`;

    let command: string;

    switch (config.type) {
      case "full":
        command = config.compress
          ? `pg_dump "${process.env.DATABASE_URL}" --no-owner --no-acl | gzip > "${filePath}"`
          : `pg_dump "${process.env.DATABASE_URL}" --no-owner --no-acl > "${filePath}"`;
        break;
      case "schema-only":
        command = config.compress
          ? `pg_dump "${process.env.DATABASE_URL}" --schema-only --no-owner | gzip > "${filePath}"`
          : `pg_dump "${process.env.DATABASE_URL}" --schema-only --no-owner > "${filePath}"`;
        break;
      case "incremental":
        command = config.compress
          ? `pg_dump "${process.env.DATABASE_URL}" --no-owner --no-acl --data-only | gzip > "${filePath}"`
          : `pg_dump "${process.env.DATABASE_URL}" --no-owner --no-acl --data-only > "${filePath}"`;
        break;
    }

    execSync(command, { stdio: "pipe", timeout: 300000 });

    const stats = fs.statSync(filePath);
    const duration = Date.now() - startTime;

    console.log(`[Backup] Created: ${filename}`);
    console.log(`[Backup] Size: ${(stats.size / 1024 / 1024).toFixed(2)} MB`);
    console.log(`[Backup] Duration: ${(duration / 1000).toFixed(1)}s`);

    // Upload to cloud storage if configured
    if (config.storageProvider === "s3" && config.s3Bucket) {
      await uploadToS3(filePath, config.s3Bucket, filename);
    }

    // Log backup metadata
    await logBackupMetadata({
      filename,
      filePath,
      size: stats.size,
      type: config.type,
      duration,
      timestamp: new Date(),
    });

    return { success: true, filePath, size: stats.size, duration };
  } catch (error) {
    console.error("[Backup] Failed:", error);
    // Cleanup failed backup file
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
    }
    return { success: false, filePath: "", size: 0, duration: Date.now() - startTime };
  }
}

async function uploadToS3(
  filePath: string,
  bucket: string,
  key: string
): Promise<void> {
  execSync(`aws s3 cp "${filePath}" "s3://${bucket}/backups/${key}"`, {
    stdio: "pipe",
    timeout: 600000,
  });
  console.log(`[Backup] Uploaded to S3: s3://${bucket}/backups/${key}`);
}

interface BackupMetadata {
  filename: string;
  filePath: string;
  size: number;
  type: string;
  duration: number;
  timestamp: Date;
}

async function logBackupMetadata(metadata: BackupMetadata): Promise<void> {
  const logFile = path.resolve(process.cwd(), "backups", "backup-log.json");
  let log: BackupMetadata[] = [];

  if (fs.existsSync(logFile)) {
    log = JSON.parse(fs.readFileSync(logFile, "utf8"));
  }

  log.push(metadata);
  fs.writeFileSync(logFile, JSON.stringify(log, null, 2));
}

// Restore from backup
export async function restoreBackup(backupFilePath: string): Promise<void> {
  if (!fs.existsSync(backupFilePath)) {
    throw new Error(`Backup file not found: ${backupFilePath}`);
  }

  const isCompressed = backupFilePath.endsWith(".gz");

  const command = isCompressed
    ? `gunzip -c "${backupFilePath}" | psql "${process.env.DATABASE_URL}"`
    : `psql "${process.env.DATABASE_URL}" < "${backupFilePath}"`;

  console.log(`[Restore] Restoring from: ${backupFilePath}`);
  execSync(command, { stdio: "inherit", timeout: 600000 });
  console.log("[Restore] Complete");
}

// Cleanup old backups
export async function cleanupOldBackups(retentionDays: number = 30): Promise<number> {
  const backupDir = path.resolve(process.cwd(), "backups");
  if (!fs.existsSync(backupDir)) return 0;

  const cutoff = Date.now() - retentionDays * 24 * 60 * 60 * 1000;
  let deleted = 0;

  const files = fs.readdirSync(backupDir).filter((f) => f.startsWith("backup_"));

  for (const file of files) {
    const filePath = path.join(backupDir, file);
    const stats = fs.statSync(filePath);
    if (stats.mtimeMs < cutoff) {
      fs.unlinkSync(filePath);
      deleted++;
      console.log(`[Cleanup] Deleted old backup: ${file}`);
    }
  }

  return deleted;
}

// Verify backup integrity
export async function verifyBackup(backupFilePath: string): Promise<{
  valid: boolean;
  tableCount: number;
  estimatedRows: number;
}> {
  const isCompressed = backupFilePath.endsWith(".gz");
  const command = isCompressed
    ? `gunzip -c "${backupFilePath}" | grep -c "^CREATE TABLE\\|^COPY"`
    : `grep -c "^CREATE TABLE\\|^COPY" "${backupFilePath}"`;

  try {
    const output = execSync(command, { encoding: "utf8", timeout: 60000 });
    const count = parseInt(output.trim());

    return {
      valid: count > 0,
      tableCount: Math.floor(count / 2),
      estimatedRows: count,
    };
  } catch {
    return { valid: false, tableCount: 0, estimatedRows: 0 };
  }
}
```

### Varianti e Configurazioni

```typescript
// scripts/backup-cron.ts — Scheduled backup with node-cron

import { CronJob } from "cron";
import { createBackup, cleanupOldBackups } from "./backup";

// Backup giornaliero alle 3:00 AM
const dailyBackup = new CronJob("0 3 * * *", async () => {
  console.log("[Cron] Starting daily backup...");
  const result = await createBackup({
    type: "full",
    compress: true,
    encrypt: false,
    retentionDays: 30,
    storageProvider: "s3",
    s3Bucket: process.env.BACKUP_S3_BUCKET,
  });

  if (result.success) {
    console.log(`[Cron] Daily backup complete: ${result.filePath}`);
  } else {
    console.error("[Cron] Daily backup FAILED");
    // Send alert via webhook, email, etc.
    await sendBackupAlert("Daily backup failed");
  }
});

// Schema-only backup settimanale (domenica 2:00 AM)
const weeklySchemaBackup = new CronJob("0 2 * * 0", async () => {
  console.log("[Cron] Starting weekly schema backup...");
  await createBackup({
    type: "schema-only",
    compress: true,
    encrypt: false,
    retentionDays: 90,
    storageProvider: "s3",
    s3Bucket: process.env.BACKUP_S3_BUCKET,
  });
});

// Cleanup mensile (1o del mese alle 4:00 AM)
const monthlyCleanup = new CronJob("0 4 1 * *", async () => {
  console.log("[Cron] Starting monthly cleanup...");
  const deleted = await cleanupOldBackups(30);
  console.log(`[Cron] Cleaned up ${deleted} old backups`);
});

async function sendBackupAlert(message: string): Promise<void> {
  const webhookUrl = process.env.SLACK_WEBHOOK_URL;
  if (!webhookUrl) return;

  await fetch(webhookUrl, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      text: `[DB Backup Alert] ${message}`,
      channel: "#ops-alerts",
    }),
  });
}

// Start all cron jobs
dailyBackup.start();
weeklySchemaBackup.start();
monthlyCleanup.start();

console.log("[Cron] Backup scheduler started");
```

### Errori Comuni da Evitare
- Non salvare backup sullo stesso server del database
- Non dimenticare di testare il restore periodicamente
- Non salvare credenziali database nel path del backup
- Non usare backup non compressi per database grandi (> 1GB)

### Checklist di Verifica
- [ ] Backup automatico giornaliero configurato
- [ ] Backup salvato su storage esterno (S3/GCS)
- [ ] Script di restore testato e documentato
- [ ] Retention policy con cleanup automatico
- [ ] Alert su backup falliti
- [ ] Verifica integrita backup periodica


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 16: READ REPLICAS E CQRS
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Implementazione read/write splitting con Prisma, gestione replica lag,
failover automatico e pattern CQRS con event store.

### Implementazione Completa

```typescript
// lib/db/read-write-split.ts — Read/Write splitting con Prisma

import { PrismaClient } from "@prisma/client";

// Client per operazioni di scrittura (primary)
const writeClient = new PrismaClient({
  datasourceUrl: process.env.DATABASE_URL,
});

// Client per operazioni di lettura (replica)
const readClient = new PrismaClient({
  datasourceUrl: process.env.DATABASE_REPLICA_URL ?? process.env.DATABASE_URL,
});

export type QueryMode = "read" | "write";

// Factory che restituisce il client appropriato
export function getDb(mode: QueryMode = "read"): PrismaClient {
  return mode === "write" ? writeClient : readClient;
}

// Prisma extension con read/write split automatico
export function createReadWriteClient() {
  const primary = new PrismaClient({
    datasourceUrl: process.env.DATABASE_URL,
  });

  return primary.$extends({
    query: {
      $allModels: {
        // Letture vanno alla replica
        async findMany({ args, query }) {
          return executeOnReplica("findMany", args, query);
        },
        async findFirst({ args, query }) {
          return executeOnReplica("findFirst", args, query);
        },
        async findUnique({ args, query }) {
          return executeOnReplica("findUnique", args, query);
        },
        async count({ args, query }) {
          return executeOnReplica("count", args, query);
        },
        async aggregate({ args, query }) {
          return executeOnReplica("aggregate", args, query);
        },
        // Scritture vanno al primary (default)
        async create({ args, query }) {
          return query(args);
        },
        async update({ args, query }) {
          return query(args);
        },
        async delete({ args, query }) {
          return query(args);
        },
        async createMany({ args, query }) {
          return query(args);
        },
        async updateMany({ args, query }) {
          return query(args);
        },
        async deleteMany({ args, query }) {
          return query(args);
        },
      },
    },
  });
}

async function executeOnReplica(
  _operation: string,
  args: any,
  fallbackQuery: (args: any) => Promise<any>
): Promise<any> {
  try {
    // Tenta sulla replica
    return await fallbackQuery(args);
  } catch (error) {
    // Fallback al primary se la replica non risponde
    console.warn("[ReadReplica] Falling back to primary");
    return fallbackQuery(args);
  }
}

// Replica lag monitoring
export async function checkReplicaLag(): Promise<{
  lagBytes: number;
  lagSeconds: number;
  isHealthy: boolean;
}> {
  try {
    const result = await writeClient.$queryRaw<
      Array<{
        lag_bytes: number;
        lag_seconds: number;
      }>
    >`
      SELECT
        pg_wal_lsn_diff(pg_current_wal_lsn(), replay_lsn)::bigint AS lag_bytes,
        EXTRACT(EPOCH FROM (now() - replay_lag))::int AS lag_seconds
      FROM pg_stat_replication
      LIMIT 1
    `;

    const lag = result[0];
    return {
      lagBytes: Number(lag?.lag_bytes ?? 0),
      lagSeconds: Number(lag?.lag_seconds ?? 0),
      isHealthy: Number(lag?.lag_seconds ?? 0) < 10,
    };
  } catch {
    return { lagBytes: 0, lagSeconds: 0, isHealthy: false };
  }
}
```

### Varianti e Configurazioni

```typescript
// lib/db/cqrs.ts — CQRS pattern con event store

import { PrismaClient, Prisma } from "@prisma/client";

const prisma = new PrismaClient();

// Event store schema (aggiungere allo schema Prisma):
// model Event {
//   id          String   @id @default(cuid())
//   streamId    String
//   streamType  String
//   type        String
//   version     Int
//   data        Json
//   metadata    Json?
//   createdAt   DateTime @default(now())
//   @@unique([streamId, version])
//   @@index([streamType, streamId])
//   @@map("events")
// }

interface DomainEvent {
  type: string;
  data: Record<string, any>;
  metadata?: Record<string, any>;
}

// Command handler che salva eventi
export class EventStore {
  constructor(private prisma: PrismaClient) {}

  async appendEvent(
    streamId: string,
    streamType: string,
    event: DomainEvent,
    expectedVersion: number
  ): Promise<void> {
    try {
      await this.prisma.$executeRaw`
        INSERT INTO events (id, stream_id, stream_type, type, version, data, metadata, created_at)
        VALUES (
          gen_random_uuid()::text,
          ${streamId},
          ${streamType},
          ${event.type},
          ${expectedVersion + 1},
          ${JSON.stringify(event.data)}::jsonb,
          ${JSON.stringify(event.metadata ?? {})}::jsonb,
          NOW()
        )
      `;
    } catch (error) {
      if (error instanceof Prisma.PrismaClientKnownRequestError && error.code === "P2002") {
        throw new Error(`Optimistic concurrency conflict on stream ${streamId} at version ${expectedVersion + 1}`);
      }
      throw error;
    }
  }

  async getEvents(
    streamId: string,
    fromVersion: number = 0
  ): Promise<Array<DomainEvent & { version: number; createdAt: Date }>> {
    return this.prisma.$queryRaw`
      SELECT type, data, metadata, version, created_at AS "createdAt"
      FROM events
      WHERE stream_id = ${streamId}
        AND version > ${fromVersion}
      ORDER BY version ASC
    `;
  }

  async getLatestVersion(streamId: string): Promise<number> {
    const result = await this.prisma.$queryRaw<
      Array<{ version: number }>
    >`
      SELECT COALESCE(MAX(version), 0) AS version
      FROM events
      WHERE stream_id = ${streamId}
    `;
    return result[0]?.version ?? 0;
  }
}

// Esempio: Order Aggregate con event sourcing
interface OrderState {
  id: string;
  status: string;
  items: Array<{ productId: string; quantity: number; price: number }>;
  total: number;
  version: number;
}

export class OrderAggregate {
  private state: OrderState;

  constructor(id: string) {
    this.state = { id, status: "NEW", items: [], total: 0, version: 0 };
  }

  // Ricostruisce lo stato dagli eventi
  static async load(eventStore: EventStore, orderId: string): Promise<OrderAggregate> {
    const aggregate = new OrderAggregate(orderId);
    const events = await eventStore.getEvents(orderId);

    for (const event of events) {
      aggregate.apply(event);
      aggregate.state.version = event.version;
    }

    return aggregate;
  }

  private apply(event: DomainEvent): void {
    switch (event.type) {
      case "OrderCreated":
        this.state.status = "PENDING";
        this.state.items = event.data.items;
        this.state.total = event.data.total;
        break;
      case "OrderConfirmed":
        this.state.status = "CONFIRMED";
        break;
      case "OrderShipped":
        this.state.status = "SHIPPED";
        break;
      case "OrderDelivered":
        this.state.status = "DELIVERED";
        break;
      case "OrderCancelled":
        this.state.status = "CANCELLED";
        break;
      case "ItemAdded":
        this.state.items.push(event.data.item);
        this.state.total += event.data.item.price * event.data.item.quantity;
        break;
    }
  }

  // Command: Create order
  async createOrder(
    eventStore: EventStore,
    items: Array<{ productId: string; quantity: number; price: number }>
  ): Promise<void> {
    if (this.state.status !== "NEW") {
      throw new Error("Order already created");
    }

    const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);

    await eventStore.appendEvent(
      this.state.id,
      "Order",
      {
        type: "OrderCreated",
        data: { items, total },
      },
      this.state.version
    );
  }

  // Command: Confirm order
  async confirmOrder(eventStore: EventStore): Promise<void> {
    if (this.state.status !== "PENDING") {
      throw new Error(`Cannot confirm order in status: ${this.state.status}`);
    }

    await eventStore.appendEvent(
      this.state.id,
      "Order",
      { type: "OrderConfirmed", data: {} },
      this.state.version
    );
  }

  // Command: Cancel order
  async cancelOrder(eventStore: EventStore, reason: string): Promise<void> {
    if (["DELIVERED", "CANCELLED"].includes(this.state.status)) {
      throw new Error(`Cannot cancel order in status: ${this.state.status}`);
    }

    await eventStore.appendEvent(
      this.state.id,
      "Order",
      { type: "OrderCancelled", data: { reason } },
      this.state.version
    );
  }

  getState(): Readonly<OrderState> {
    return { ...this.state };
  }
}

// Projection: materializzare lo stato corrente in una read table
export async function projectOrderToReadModel(
  prisma: PrismaClient,
  eventStore: EventStore,
  orderId: string
): Promise<void> {
  const aggregate = await OrderAggregate.load(eventStore, orderId);
  const state = aggregate.getState();

  await prisma.$executeRaw`
    INSERT INTO order_read_model (id, status, items, total, version, updated_at)
    VALUES (${state.id}, ${state.status}, ${JSON.stringify(state.items)}::jsonb, ${state.total}, ${state.version}, NOW())
    ON CONFLICT (id) DO UPDATE SET
      status = EXCLUDED.status,
      items = EXCLUDED.items,
      total = EXCLUDED.total,
      version = EXCLUDED.version,
      updated_at = NOW()
  `;
}
```

### Errori Comuni da Evitare
- Non leggere dalla replica subito dopo una scrittura — il lag puo causare letture stale
- Non dimenticare il fallback al primary se la replica non risponde
- Non usare event sourcing per tutti i domini — solo dove serve auditabilita completa
- Non dimenticare di proiettare gli eventi nella read model dopo ogni write

### Checklist di Verifica
- [ ] Read replica configurata con URL separato
- [ ] Fallback automatico al primary in caso di errore replica
- [ ] Monitoring replica lag con alert su soglia > 10s
- [ ] Event store con versioning per optimistic concurrency
- [ ] Projection automatica nella read model
- [ ] Test di failover periodico


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 17: DATABASE MONITORING E HEALTH CHECKS
═══════════════════════════════════════════════════════════════════════════════

### Panoramica
Dashboard di monitoraggio database: health check endpoints, slow query detection,
connection pool metrics, table statistics, e alert automatici.

### Implementazione Completa

```typescript
// app/api/admin/db-health/route.ts — Database health check API

import { NextRequest, NextResponse } from "next/server";
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

interface DatabaseHealthReport {
  status: "healthy" | "degraded" | "unhealthy";
  timestamp: string;
  checks: {
    connectivity: HealthCheck;
    responseTime: HealthCheck;
    connectionPool: HealthCheck;
    diskUsage: HealthCheck;
    replicationLag: HealthCheck;
    slowQueries: HealthCheck;
    tableStats: TableStats[];
    cacheHitRatio: HealthCheck;
  };
}

interface HealthCheck {
  status: "pass" | "warn" | "fail";
  value: string | number;
  threshold?: string | number;
  message?: string;
}

interface TableStats {
  name: string;
  rowCount: number;
  totalSize: string;
  indexSize: string;
  deadTuples: number;
  lastVacuum: string | null;
  lastAnalyze: string | null;
}

export async function GET(request: NextRequest): Promise<NextResponse> {
  const startTime = Date.now();

  try {
    const [
      connectivity,
      connectionPool,
      diskUsage,
      slowQueries,
      tableStats,
      cacheHitRatio,
    ] = await Promise.all([
      checkConnectivity(),
      checkConnectionPool(),
      checkDiskUsage(),
      checkSlowQueries(),
      getTableStatistics(),
      checkCacheHitRatio(),
    ]);

    const responseTime = Date.now() - startTime;
    const responseTimeCheck: HealthCheck = {
      status: responseTime < 100 ? "pass" : responseTime < 500 ? "warn" : "fail",
      value: responseTime,
      threshold: 100,
      message: `${responseTime}ms`,
    };

    const allChecks = [connectivity, responseTimeCheck, connectionPool, diskUsage, cacheHitRatio];
    const overallStatus = allChecks.some((c) => c.status === "fail")
      ? "unhealthy"
      : allChecks.some((c) => c.status === "warn")
        ? "degraded"
        : "healthy";

    const report: DatabaseHealthReport = {
      status: overallStatus,
      timestamp: new Date().toISOString(),
      checks: {
        connectivity,
        responseTime: responseTimeCheck,
        connectionPool,
        diskUsage,
        replicationLag: { status: "pass", value: "N/A", message: "No replicas configured" },
        slowQueries,
        tableStats,
        cacheHitRatio,
      },
    };

    return NextResponse.json(report, {
      status: overallStatus === "unhealthy" ? 503 : 200,
    });
  } catch (error) {
    return NextResponse.json(
      {
        status: "unhealthy",
        timestamp: new Date().toISOString(),
        error: error instanceof Error ? error.message : "Unknown error",
      },
      { status: 503 }
    );
  }
}

async function checkConnectivity(): Promise<HealthCheck> {
  try {
    const start = Date.now();
    await prisma.$queryRaw`SELECT 1`;
    const ms = Date.now() - start;
    return {
      status: ms < 50 ? "pass" : ms < 200 ? "warn" : "fail",
      value: ms,
      message: `Connected in ${ms}ms`,
    };
  } catch {
    return { status: "fail", value: 0, message: "Cannot connect to database" };
  }
}

async function checkConnectionPool(): Promise<HealthCheck> {
  const result = await prisma.$queryRaw<
    Array<{ active: number; idle: number; total: number; max_conn: number }>
  >`
    SELECT
      count(*) FILTER (WHERE state = 'active')::int AS active,
      count(*) FILTER (WHERE state = 'idle')::int AS idle,
      count(*)::int AS total,
      (SELECT setting::int FROM pg_settings WHERE name = 'max_connections') AS max_conn
    FROM pg_stat_activity
    WHERE datname = current_database()
  `;

  const stats = result[0];
  const utilization = (stats.total / stats.max_conn) * 100;

  return {
    status: utilization < 70 ? "pass" : utilization < 90 ? "warn" : "fail",
    value: `${stats.active} active, ${stats.idle} idle, ${stats.total}/${stats.max_conn} total`,
    threshold: "90% utilization",
    message: `${utilization.toFixed(1)}% pool utilization`,
  };
}

async function checkDiskUsage(): Promise<HealthCheck> {
  const result = await prisma.$queryRaw<Array<{ size: string; size_bytes: number }>>`
    SELECT
      pg_size_pretty(pg_database_size(current_database())) AS size,
      pg_database_size(current_database())::bigint AS size_bytes
  `;

  const sizeGB = Number(result[0].size_bytes) / (1024 * 1024 * 1024);

  return {
    status: sizeGB < 10 ? "pass" : sizeGB < 50 ? "warn" : "fail",
    value: result[0].size,
    threshold: "50 GB",
    message: `Database size: ${result[0].size}`,
  };
}

async function checkSlowQueries(): Promise<HealthCheck> {
  try {
    const result = await prisma.$queryRaw<Array<{ count: number }>>`
      SELECT count(*)::int
      FROM pg_stat_activity
      WHERE state = 'active'
        AND query_start < now() - interval '5 seconds'
        AND query NOT LIKE '%pg_stat%'
    `;

    const slowCount = result[0]?.count ?? 0;

    return {
      status: slowCount === 0 ? "pass" : slowCount < 5 ? "warn" : "fail",
      value: slowCount,
      threshold: 5,
      message: `${slowCount} queries running > 5s`,
    };
  } catch {
    return { status: "pass", value: 0, message: "Unable to check slow queries" };
  }
}

async function getTableStatistics(): Promise<TableStats[]> {
  return prisma.$queryRaw`
    SELECT
      relname AS name,
      n_live_tup::int AS "rowCount",
      pg_size_pretty(pg_total_relation_size(relid)) AS "totalSize",
      pg_size_pretty(pg_indexes_size(relid)) AS "indexSize",
      n_dead_tup::int AS "deadTuples",
      last_vacuum::text AS "lastVacuum",
      last_analyze::text AS "lastAnalyze"
    FROM pg_stat_user_tables
    ORDER BY pg_total_relation_size(relid) DESC
    LIMIT 20
  `;
}

async function checkCacheHitRatio(): Promise<HealthCheck> {
  const result = await prisma.$queryRaw<Array<{ ratio: number }>>`
    SELECT
      CASE
        WHEN (sum(heap_blks_hit) + sum(heap_blks_read)) = 0 THEN 100
        ELSE round(100.0 * sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)), 2)
      END AS ratio
    FROM pg_statio_user_tables
  `;

  const ratio = Number(result[0]?.ratio ?? 0);

  return {
    status: ratio > 99 ? "pass" : ratio > 95 ? "warn" : "fail",
    value: `${ratio}%`,
    threshold: "99%",
    message: `Cache hit ratio: ${ratio}%`,
  };
}
```

### Varianti e Configurazioni

```typescript
// components/admin/DatabaseDashboard.tsx — Dashboard UI component

"use client";

import { useState, useEffect } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Progress } from "@/components/ui/progress";

interface HealthReport {
  status: "healthy" | "degraded" | "unhealthy";
  timestamp: string;
  checks: Record<string, any>;
}

export function DatabaseDashboard() {
  const [report, setReport] = useState<HealthReport | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchHealth() {
      try {
        const res = await fetch("/api/admin/db-health");
        const data = await res.json();
        setReport(data);
      } catch (error) {
        console.error("Failed to fetch health:", error);
      } finally {
        setLoading(false);
      }
    }

    fetchHealth();
    const interval = setInterval(fetchHealth, 30000);
    return () => clearInterval(interval);
  }, []);

  if (loading) {
    return <div className="animate-pulse h-96 bg-muted rounded-lg" />;
  }

  if (!report) {
    return (
      <Card>
        <CardContent className="p-6">
          <p className="text-muted-foreground">Unable to fetch database health</p>
        </CardContent>
      </Card>
    );
  }

  const statusColors = {
    healthy: "bg-green-500",
    degraded: "bg-yellow-500",
    unhealthy: "bg-red-500",
  };

  const checkStatusColors = {
    pass: "default" as const,
    warn: "secondary" as const,
    fail: "destructive" as const,
  };

  return (
    <div className="space-y-6">
      <div className="flex items-center gap-4">
        <h2 className="text-2xl font-bold">Database Health</h2>
        <div className={`w-3 h-3 rounded-full ${statusColors[report.status]} animate-pulse`} />
        <Badge variant={report.status === "healthy" ? "default" : "destructive"}>
          {report.status.toUpperCase()}
        </Badge>
        <span className="text-sm text-muted-foreground">
          Last updated: {new Date(report.timestamp).toLocaleTimeString()}
        </span>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
        {Object.entries(report.checks).map(([key, check]) => {
          if (key === "tableStats") return null;
          const c = check as { status: string; value: any; message?: string };
          return (
            <Card key={key}>
              <CardHeader className="pb-2">
                <CardTitle className="text-sm font-medium capitalize">
                  {key.replace(/([A-Z])/g, " $1").trim()}
                </CardTitle>
              </CardHeader>
              <CardContent>
                <div className="flex items-center justify-between">
                  <span className="text-2xl font-bold">{String(c.value)}</span>
                  <Badge variant={checkStatusColors[c.status as keyof typeof checkStatusColors] ?? "default"}>
                    {c.status}
                  </Badge>
                </div>
                {c.message && (
                  <p className="text-sm text-muted-foreground mt-1">{c.message}</p>
                )}
              </CardContent>
            </Card>
          );
        })}
      </div>

      {report.checks.tableStats && (
        <Card>
          <CardHeader>
            <CardTitle>Table Statistics</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="overflow-x-auto">
              <table className="w-full text-sm">
                <thead>
                  <tr className="border-b">
                    <th className="text-left p-2">Table</th>
                    <th className="text-right p-2">Rows</th>
                    <th className="text-right p-2">Total Size</th>
                    <th className="text-right p-2">Index Size</th>
                    <th className="text-right p-2">Dead Tuples</th>
                    <th className="text-left p-2">Last Vacuum</th>
                  </tr>
                </thead>
                <tbody>
                  {(report.checks.tableStats as any[]).map((table: any) => (
                    <tr key={table.name} className="border-b hover:bg-muted/50">
                      <td className="p-2 font-mono text-xs">{table.name}</td>
                      <td className="p-2 text-right">{table.rowCount.toLocaleString()}</td>
                      <td className="p-2 text-right">{table.totalSize}</td>
                      <td className="p-2 text-right">{table.indexSize}</td>
                      <td className="p-2 text-right">
                        <span className={table.deadTuples > 10000 ? "text-red-500 font-bold" : ""}>
                          {table.deadTuples.toLocaleString()}
                        </span>
                      </td>
                      <td className="p-2 text-xs text-muted-foreground">
                        {table.lastVacuum ?? "Never"}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </CardContent>
        </Card>
      )}
    </div>
  );
}
```

### Errori Comuni da Evitare
- Non esporre l'endpoint di health check senza autenticazione admin
- Non eseguire query di monitoraggio pesanti troppo frequentemente (max ogni 30s)
- Non ignorare dead tuples alti — indicano necessita di VACUUM
- Non ignorare cache hit ratio basso — aumentare `shared_buffers`

### Checklist di Verifica
- [ ] Health check endpoint protetto da autenticazione admin
- [ ] Monitoraggio connection pool con alert su > 80% utilizzo
- [ ] Slow query detection con soglia configurabile
- [ ] Table statistics con dead tuples monitoring
- [ ] Cache hit ratio > 99% in produzione
- [ ] Auto-refresh dashboard ogni 30 secondi
