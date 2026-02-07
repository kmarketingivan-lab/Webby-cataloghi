# CATALOGO-DATABASE-PRISMA

Catalogo Completo Prisma con Next.js 14
§ PRISMA SETUP AVANZATO
Schema Organization (Multi-file)
prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["postgresqlExtensions"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  extensions = [pgcrypto, pg_trgm]
}

// Import modular schemas
import "./models/User.prisma"
import "./models/Product.prisma"
import "./models/Order.prisma"
import "./enums/Status.prisma"
prisma
// models/User.prisma
model User {
  id        String    @id @default(cuid())
  email     String    @unique
  name      String?
  profile   Profile?
  posts     Post[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  
  @@index([email])
  @@map("users")
}

// models/Profile.prisma
model Profile {
  id        String   @id @default(cuid())
  bio       String?
  userId    String   @unique
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@map("profiles")
}
Generators Configuration
prisma
generator client {
  provider = "prisma-client-js"
  previewFeatures = ["fieldReference", "postgresqlExtensions"]
  
  // Generate types for edge runtimes
  engineType = "library"
  
  // Custom output path
  output = "./src/generated/client"
}

// Generate Zod schemas
generator zod {
  provider = "zod-prisma-types"
  output   = "./src/generated/zod"
  // Optional: exclude models
  excludeModels = ["_Migration"]
}

// Generate TypeBox schemas
generator typebox {
  provider = "prisma-generator-typebox"
  output   = "./src/generated/typebox"
}
Preview Features
prisma
generator client {
  provider = "prisma-client-js"
  previewFeatures = [
    "fieldReference",          // Reference fields in operations
    "postgresqlExtensions",    // Use PostgreSQL extensions
    "improvedQueryRaw",        // Enhanced raw queries
    "metrics",                 // Query metrics
    "orderByRelation",         // Order by relation fields
    "filteredRelationCount",   // Filtered relation counts
  ]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  
  // Enable extensions
  extensions = [
    pgcrypto,     // Cryptographic functions
    pg_trgm,      // Text similarity search
    btree_gin,    // Indexing
    postgis,      // Geospatial data
  ]
}
Client Extensions Configuration
typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'
import { loggingExtension } from './extensions/logging'
import { softDeleteExtension } from './extensions/softDelete'
import { auditExtension } from './extensions/audit'

const prismaClientSingleton = () => {
  return new PrismaClient({
    log: [
      { level: 'query', emit: 'event' },
      { level: 'error', emit: 'stdout' },
      { level: 'warn', emit: 'stdout' },
    ],
  })
  .$extends(loggingExtension)
  .$extends(softDeleteExtension)
  .$extends(auditExtension)
}

declare global {
  var prisma: undefined | ReturnType<typeof prismaClientSingleton>
}

export const prisma = globalThis.prisma ?? prismaClientSingleton()

if (process.env.NODE_ENV !== 'production') globalThis.prisma = prisma
§ SCHEMA PATTERNS
One-to-One Relationship
prisma
model User {
  id      String   @id @default(cuid())
  email   String   @unique
  profile Profile?
  
  // Optional: Explicit relation for clarity
  // @@relation("user_profile")
}

model Profile {
  id        String  @id @default(cuid())
  bio       String?
  userId    String  @unique
  user      User    @relation(fields: [userId], references: [id], onDelete: Cascade)
}
One-to-Many Relationship
prisma
model User {
  id      String  @id @default(cuid())
  email   String  @unique
  posts   Post[]  // One user has many posts
}

model Post {
  id        String   @id @default(cuid())
  title     String
  content   String?
  authorId  String
  author    User     @relation(fields: [authorId], references: [id])
  
  @@index([authorId])
}
Many-to-Many Relationship
prisma
// Implicit many-to-many
model Post {
  id       String    @id @default(cuid())
  title    String
  tags     Tag[]     // Implicit join table will be created
}

model Tag {
  id    String   @id @default(cuid())
  name  String   @unique
  posts Post[]
}

// Explicit many-to-many with additional fields
model Post {
  id       String        @id @default(cuid())
  title    String
  tags     PostTag[]
}

model Tag {
  id       String        @id @default(cuid())
  name     String        @unique
  posts    PostTag[]
}

model PostTag {
  postId   String
  tagId    String
  assignedAt DateTime   @default(now())
  assignedBy String?
  
  post     Post         @relation(fields: [postId], references: [id])
  tag      Tag          @relation(fields: [tagId], references: [id])
  
  @@id([postId, tagId])
  @@index([tagId])
}
Self-Referential Relations
prisma
model Employee {
  id           String     @id @default(cuid())
  name         String
  email        String     @unique
  
  // Self-referential: Employee manages other employees
  managerId    String?
  manager      Employee?  @relation("Management", fields: [managerId], references: [id])
  subordinates Employee[] @relation("Management")
  
  // Another self-referential: Employee mentors other employees
  mentorId     String?
  mentor       Employee?  @relation("Mentorship", fields: [mentorId], references: [id])
  mentees      Employee[] @relation("Mentorship")
}
Polymorphic Relations Workarounds
prisma
// Pattern 1: Concrete table inheritance
model Attachment {
  id            String   @id @default(cuid())
  fileName      String
  fileSize      Int
  mimeType      String
  
  // Polymorphic relation using type discriminator
  attachedToId  String
  attachedToType AttachmentType
  
  // Optional: Generic relations (requires manual handling)
  post        Post?     @relation(fields: [attachedToId], references: [id])
  product     Product?  @relation(fields: [attachedToId], references: [id])
  user        User?     @relation(fields: [attachedToId], references: [id])
  
  @@index([attachedToId, attachedToType])
}

enum AttachmentType {
  POST
  PRODUCT
  USER
}

// Pattern 2: Single table inheritance
model Content {
  id        String   @id @default(cuid())
  title     String
  body      String?
  
  // Discriminator field
  type      ContentType
  
  // Post-specific fields
  published Boolean? @default(false)
  
  // Product-specific fields
  price     Decimal? @db.Decimal(10, 2)
  sku       String?
  
  // Article-specific fields
  readTime  Int?
  
  @@index([type])
}

enum ContentType {
  POST
  PRODUCT
  ARTICLE
}
JSON Fields Usage
prisma
model Product {
  id          String   @id @default(cuid())
  name        String
  price       Decimal  @db.Decimal(10, 2)
  
  // JSON for flexible metadata
  metadata    Json?
  
  // JSON array for tags/variants
  tags        Json?    @default("[]")
  
  // JSON for translations
  translations Json?   @default("{}")
  
  // JSON for specifications with validation
  specifications Json? @default("{}")
}

// TypeScript interface for JSON fields
interface ProductMetadata {
  dimensions?: {
    width: number;
    height: number;
    depth: number;
  };
  weight?: number;
  manufacturer?: {
    name: string;
    country: string;
  };
  certifications?: string[];
}

interface ProductSpecifications {
  [key: string]: string | number | boolean;
}
Enum Best Practices
prisma
// enums/OrderStatus.prisma
enum OrderStatus {
  DRAFT
  PENDING_PAYMENT
  PAYMENT_RECEIVED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
  
  @@map("order_statuses")
}

// enums/UserRole.prisma
enum UserRole {
  GUEST       @map("guest")
  USER        @map("user")
  MODERATOR   @map("moderator")
  ADMIN       @map("admin")
  SUPER_ADMIN @map("super_admin")
  
  @@map("user_roles")
}

// Bitmask pattern for permissions
enum Permission {
  READ    @map("read")    // 1 << 0
  WRITE   @map("write")   // 1 << 1
  DELETE  @map("delete")  // 1 << 2
  SHARE   @map("share")   // 1 << 3
  ADMIN   @map("admin")   // 1 << 4
  
  @@map("permissions")
}

// Composite enum with descriptions
enum NotificationType {
  EMAIL    @map("email")
  SMS      @map("sms")
  PUSH     @map("push")
  IN_APP   @map("in_app")
  
  @@map("notification_types")
  @@description("Types of notifications supported by the system")
}
§ QUERY PATTERNS
Select vs Include
typescript
// SELECT: Choose specific fields
const userWithEmail = await prisma.user.findUnique({
  where: { id: userId },
  select: {
    id: true,
    email: true,
    name: true,
    // Exclude sensitive fields
    // password: false
  }
});

// INCLUDE: Include related models
const userWithPosts = await prisma.user.findUnique({
  where: { id: userId },
  include: {
    posts: {
      select: {
        id: true,
        title: true,
        createdAt: true,
      },
      where: {
        published: true,
      },
      orderBy: {
        createdAt: 'desc',
      },
      take: 5,
    },
    profile: true,
  }
});

// Mixed: Combine select and include
const mixedSelection = await prisma.user.findMany({
  select: {
    id: true,
    email: true,
    // Include relations with specific fields
    posts: {
      select: {
        id: true,
        title: true,
      },
      take: 3,
    },
    // Count relations without loading data
    _count: {
      select: {
        posts: true,
        comments: true,
      },
    },
  },
  where: {
    email: {
      contains: '@company.com',
    },
  },
});
Nested Reads
typescript
// Deep nested queries
const deepNested = await prisma.user.findMany({
  include: {
    posts: {
      include: {
        comments: {
          include: {
            author: {
              select: {
                name: true,
                avatar: true,
              },
            },
            replies: {
              include: {
                author: true,
              },
              where: {
                approved: true,
              },
            },
          },
          where: {
            deletedAt: null,
          },
          orderBy: {
            createdAt: 'desc',
          },
        },
        tags: {
          include: {
            tag: true,
          },
        },
      },
    },
  },
  where: {
    active: true,
  },
  take: 10,
});

// Recursive nested queries (for self-referential)
const organizationalTree = await prisma.employee.findMany({
  where: {
    managerId: null, // Top-level managers
  },
  include: {
    subordinates: {
      include: {
        subordinates: {
          include: {
            subordinates: true, // 3 levels deep
          },
        },
      },
    },
  },
});
Pagination Patterns
typescript
// Offset Pagination
const offsetPagination = await prisma.post.findMany({
  skip: (page - 1) * pageSize,
  take: pageSize,
  orderBy: {
    createdAt: 'desc',
  },
  include: {
    author: {
      select: {
        name: true,
      },
    },
  },
});

// Cursor Pagination (better for large datasets)
const cursorPagination = await prisma.post.findMany({
  cursor: lastId ? { id: lastId } : undefined,
  skip: lastId ? 1 : 0, // Skip the cursor itself
  take: pageSize,
  orderBy: {
    createdAt: 'desc',
  },
  where: {
    published: true,
    createdAt: {
      lt: beforeDate, // For time-based cursors
    },
  },
});

// Keyset Pagination with multiple fields
const keysetPagination = await prisma.order.findMany({
  cursor: lastCursor ? {
    createdAt: lastCursor.createdAt,
    id: lastCursor.id,
  } : undefined,
  skip: lastCursor ? 1 : 0,
  take: pageSize,
  orderBy: [
    { createdAt: 'desc' },
    { id: 'desc' }, // Secondary sort for tie-breaking
  ],
});

// Page info for GraphQL-like APIs
interface PageInfo {
  hasNextPage: boolean;
  hasPreviousPage: boolean;
  startCursor?: string;
  endCursor?: string;
}

async function getPaginatedPosts(cursor?: string, limit: number = 20) {
  const posts = await prisma.post.findMany({
    cursor: cursor ? { id: cursor } : undefined,
    take: limit + 1, // Fetch one extra to check for next page
    orderBy: { createdAt: 'desc' },
  });

  const hasNextPage = posts.length > limit;
  const items = hasNextPage ? posts.slice(0, -1) : posts;

  return {
    items,
    pageInfo: {
      hasNextPage,
      hasPreviousPage: !!cursor,
      startCursor: items[0]?.id,
      endCursor: items[items.length - 1]?.id,
    },
  };
}
Filtering (where, contains, startsWith)
typescript
// Basic filtering
const basicFilter = await prisma.user.findMany({
  where: {
    email: 'user@example.com',
    active: true,
  },
});

// Complex filtering with operators
const complexFilter = await prisma.product.findMany({
  where: {
    AND: [
      { price: { gt: 100 } },
      { price: { lt: 1000 } },
      {
        OR: [
          { category: 'ELECTRONICS' },
          { category: 'COMPUTERS' },
        ],
      },
      { NOT: { discontinued: true } },
    ],
    stock: { gt: 0 },
  },
});

// String filtering
const stringFilters = await prisma.user.findMany({
  where: {
    email: {
      contains: '@gmail.com',
      // mode: 'insensitive', // Case-insensitive search
    },
    name: {
      startsWith: 'John',
      // endsWith: 'Doe',
    },
  },
});

// Array/List filtering
const arrayFilters = await prisma.post.findMany({
  where: {
    tags: {
      has: 'typescript', // For scalar lists
    },
    // For JSON arrays
    metadata: {
      path: ['tags'],
      array_contains: ['prisma', 'nextjs'],
    },
  },
});

// Date filtering
const dateFilters = await prisma.order.findMany({
  where: {
    createdAt: {
      gte: new Date('2024-01-01'),
      lt: new Date('2024-02-01'),
    },
    // Relative dates
    updatedAt: {
      gt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000), // Last 30 days
    },
  },
});

// Relation filtering
const relationFilters = await prisma.user.findMany({
  where: {
    posts: {
      some: {
        published: true,
        views: { gt: 1000 },
      },
    },
    // All posts must meet condition
    posts: {
      every: {
        deletedAt: null,
      },
    },
    // No posts meet condition
    posts: {
      none: {
        banned: true,
      },
    },
  },
  include: {
    posts: {
      where: {
        createdAt: {
          gte: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000),
        },
      },
    },
  },
});
Ordering (orderBy)
typescript
// Single field ordering
const singleOrder = await prisma.user.findMany({
  orderBy: {
    createdAt: 'desc',
  },
});

// Multiple field ordering
const multiOrder = await prisma.product.findMany({
  orderBy: [
    { category: 'asc' },
    { price: 'desc' },
    { name: 'asc' },
  ],
});

// Order by relation field
const relationOrder = await prisma.post.findMany({
  orderBy: {
    author: {
      name: 'asc',
    },
  },
  include: {
    author: true,
  },
});

// Order by aggregation
const aggregationOrder = await prisma.post.findMany({
  orderBy: {
    comments: {
      _count: 'desc',
    },
  },
  include: {
    _count: {
      select: { comments: true },
    },
  },
});

// Dynamic ordering
type OrderByField = 'name' | 'email' | 'createdAt';
type OrderDirection = 'asc' | 'desc';

async function getUsers(
  orderBy: OrderByField = 'createdAt',
  orderDir: OrderDirection = 'desc'
) {
  return prisma.user.findMany({
    orderBy: {
      [orderBy]: orderDir,
    },
  });
}

// Null ordering (PostgreSQL specific)
const nullsOrdering = await prisma.$queryRaw`
  SELECT * FROM "User" 
  ORDER BY "lastLogin" DESC NULLS LAST
  LIMIT 100
`;
Aggregations (count, sum, avg, min, max)
typescript
// Basic aggregations
const stats = await prisma.order.aggregate({
  _count: {
    _all: true,
    id: true,
    userId: true, // Count distinct users
  },
  _sum: {
    amount: true,
    tax: true,
  },
  _avg: {
    amount: true,
    itemsCount: true,
  },
  _min: {
    amount: true,
    createdAt: true,
  },
  _max: {
    amount: true,
    createdAt: true,
  },
  where: {
    status: 'COMPLETED',
    createdAt: {
      gte: new Date('2024-01-01'),
    },
  },
});

// Group by aggregations
const groupByStats = await prisma.order.groupBy({
  by: ['status', 'paymentMethod'],
  _count: {
    _all: true,
  },
  _sum: {
    amount: true,
  },
  _avg: {
    amount: true,
  },
  where: {
    createdAt: {
      gte: new Date('2024-01-01'),
    },
  },
  orderBy: {
    _count: {
      _all: 'desc',
    },
  },
  having: {
    amount: {
      _sum: {
        gt: 1000,
      },
    },
  },
});

// Multiple aggregations with filters
const detailedStats = await prisma.order.aggregate({
  where: {
    createdAt: {
      gte: startDate,
      lt: endDate,
    },
  },
  _count: {
    _all: true,
  },
  _sum: {
    amount: true,
  },
});

// Aggregation on relations
const userWithPostStats = await prisma.user.findMany({
  select: {
    id: true,
    name: true,
    email: true,
    _count: {
      select: {
        posts: {
          where: {
            published: true,
          },
        },
        comments: true,
      },
    },
    posts: {
      select: {
        _sum: {
          views: true,
        },
        _avg: {
          readingTime: true,
        },
      },
    },
  },
  where: {
    active: true,
  },
});

// Window functions (advanced)
const windowFunctions = await prisma.$queryRaw`
  SELECT 
    "id",
    "amount",
    "userId",
    SUM("amount") OVER (PARTITION BY "userId") as "user_total",
    RANK() OVER (ORDER BY "amount" DESC) as "rank",
    LAG("amount") OVER (ORDER BY "createdAt") as "prev_amount"
  FROM "Order"
  WHERE "status" = 'COMPLETED'
  ORDER BY "createdAt" DESC
`;
§ WRITE PATTERNS
Create with Relations
typescript
// Create with nested relations
const userWithPosts = await prisma.user.create({
  data: {
    email: 'user@example.com',
    name: 'John Doe',
    profile: {
      create: {
        bio: 'Software developer',
        avatar: 'avatar.jpg',
      },
    },
    posts: {
      create: [
        {
          title: 'First Post',
          content: 'Hello world!',
          tags: {
            create: [
              { name: 'introduction' },
              { name: 'getting-started' },
            ],
          },
        },
        {
          title: 'Second Post',
          content: 'Another post',
          published: true,
        },
      ],
    },
  },
  include: {
    profile: true,
    posts: {
      include: {
        tags: true,
      },
    },
  },
});

// Connect to existing relations
const postWithExistingTags = await prisma.post.create({
  data: {
    title: 'New Post',
    content: 'Content here',
    author: {
      connect: { id: userId },
    },
    tags: {
      connect: [
        { id: tag1Id },
        { id: tag2Id },
      ],
      // Or connectOrCreate
      connectOrCreate: [
        {
          where: { name: 'prisma' },
          create: { name: 'prisma' },
        },
        {
          where: { name: 'nextjs' },
          create: { name: 'nextjs' },
        },
      ],
    },
    category: {
      connectOrCreate: {
        where: { slug: 'tutorials' },
        create: {
          name: 'Tutorials',
          slug: 'tutorials',
          description: 'Learning materials',
        },
      },
    },
  },
});

// Create with many-to-many relations
const orderWithProducts = await prisma.order.create({
  data: {
    userId: user.id,
    status: 'PENDING',
    items: {
      create: [
        {
          productId: product1.id,
          quantity: 2,
          price: product1.price,
        },
        {
          productId: product2.id,
          quantity: 1,
          price: product2.price,
        },
      ],
    },
    shippingAddress: {
      create: {
        street: '123 Main St',
        city: 'New York',
        country: 'USA',
      },
    },
  },
  include: {
    items: {
      include: {
        product: true,
      },
    },
    shippingAddress: true,
  },
});
Upsert Patterns
typescript
// Basic upsert
const upsertUser = await prisma.user.upsert({
  where: {
    email: 'user@example.com',
  },
  update: {
    name: 'Updated Name',
    lastLogin: new Date(),
    loginCount: {
      increment: 1,
    },
  },
  create: {
    email: 'user@example.com',
    name: 'New User',
    lastLogin: new Date(),
    loginCount: 1,
  },
});

// Upsert with relations
const upsertWithRelations = await prisma.user.upsert({
  where: { email: 'user@example.com' },
  update: {
    profile: {
      upsert: {
        create: {
          bio: 'New bio',
          avatar: 'default.jpg',
        },
        update: {
          bio: 'Updated bio',
          lastUpdated: new Date(),
        },
      },
    },
  },
  create: {
    email: 'user@example.com',
    name: 'John Doe',
    profile: {
      create: {
        bio: 'Initial bio',
        avatar: 'default.jpg',
      },
    },
  },
  include: {
    profile: true,
  },
});

// Conditional upsert based on external data
async function syncUserFromSSO(ssoData: SSOData) {
  return prisma.user.upsert({
    where: { externalId: ssoData.externalId },
    update: {
      email: ssoData.email,
      name: ssoData.name,
      lastSyncedAt: new Date(),
      ssoMetadata: ssoData.metadata,
    },
    create: {
      externalId: ssoData.externalId,
      email: ssoData.email,
      name: ssoData.name,
      ssoProvider: ssoData.provider,
      ssoMetadata: ssoData.metadata,
      lastSyncedAt: new Date(),
    },
  });
}

// Batch upsert pattern
async function bulkUpsertProducts(products: ProductData[]) {
  const operations = products.map((product) =>
    prisma.product.upsert({
      where: { sku: product.sku },
      update: {
        name: product.name,
        price: product.price,
        stock: {
          increment: product.quantity,
        },
        updatedAt: new Date(),
      },
      create: {
        sku: product.sku,
        name: product.name,
        price: product.price,
        stock: product.quantity,
        category: product.category,
      },
    })
  );

  return prisma.$transaction(operations);
}
Update Many
typescript
// Update multiple records
const updatedUsers = await prisma.user.updateMany({
  where: {
    lastLogin: {
      lt: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000), // 1 year
    },
    active: true,
  },
  data: {
    active: false,
    deactivatedAt: new Date(),
    deactivationReason: 'INACTIVITY',
  },
});

// Increment/decrement values
const updateCounts = await prisma.post.updateMany({
  where: {
    published: true,
    createdAt: {
      gte: new Date(Date.now() - 24 * 60 * 60 * 1000), // Last 24h
    },
  },
  data: {
    views: {
      increment: 1,
    },
    // Other operations:
    // decrement: 1,
    // multiply: 2,
    // divide: 2,
  },
});

// Update with JSON fields
const updateJson = await prisma.product.updateMany({
  where: {
    category: 'ELECTRONICS',
  },
  data: {
    metadata: {
      // Merge with existing metadata
      set: {
        lastUpdated: new Date().toISOString(),
        updatedBy: 'system',
      },
    },
    // Or replace completely
    // metadata: newMetadata,
  },
});

// Update with array operations
const updateArray = await prisma.user.updateMany({
  where: {
    role: 'ADMIN',
  },
  data: {
    permissions: {
      // Add to array
      push: ['VIEW_AUDIT_LOG'],
      // Remove from array
      // set: currentPermissions.filter(p => p !== 'OLD_PERMISSION')
    },
  },
});

// Conditional update based on current values
const conditionalUpdate = await prisma.$executeRaw`
  UPDATE "Product"
  SET 
    "price" = CASE 
      WHEN "stock" < 10 THEN "price" * 0.9  -- 10% discount for low stock
      WHEN "stock" > 100 THEN "price" * 1.1 -- 10% increase for high stock
      ELSE "price"
    END,
    "updatedAt" = NOW()
  WHERE "category" = 'CLEARANCE'
`;
Delete Cascades
prisma
// Schema definition for cascade deletes
model User {
  id        String    @id @default(cuid())
  email     String    @unique
  // Cascade delete posts when user is deleted
  posts     Post[]    @relation(onDelete: Cascade)
  // Set null for optional relations
  profile   Profile?  @relation(onDelete: Cascade)
  // Restrict delete if there are orders
  orders    Order[]   @relation(onDelete: Restrict)
}

model Post {
  id        String    @id @default(cuid())
  title     String
  authorId  String
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  // Cascade delete comments
  comments  Comment[] @relation(onDelete: Cascade)
}

// Delete operations
const deleteWithCascade = await prisma.user.delete({
  where: { id: userId },
  include: {
    posts: {
      include: {
        comments: true, // Will be deleted due to cascade
      },
    },
    profile: true, // Will be deleted due to cascade
  },
});

// Delete many with cascade
const deleteMany = await prisma.post.deleteMany({
  where: {
    authorId: userId,
    createdAt: {
      lt: new Date(Date.now() - 30 * 24 * 60 * 60 * 1000), // Older than 30 days
    },
  },
});

// Safe delete with validation
async function safeDeleteUser(userId: string) {
  // Check for restrictions first
  const userWithOrders = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      _count: {
        select: {
          orders: {
            where: {
              status: {
                in: ['PENDING', 'PROCESSING'],
              },
            },
          },
        },
      },
    },
  });

  if (userWithOrders && userWithOrders._count.orders > 0) {
    throw new Error('Cannot delete user with pending orders');
  }

  // Proceed with delete
  return prisma.user.delete({
    where: { id: userId },
  });
}

// Batch delete with transaction
async function cleanupInactiveUsers() {
  return prisma.$transaction(async (tx) => {
    // First, delete related records
    await tx.post.deleteMany({
      where: {
        author: {
          lastLogin: {
            lt: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000), // 1 year
          },
        },
      },
    });

    await tx.profile.deleteMany({
      where: {
        user: {
          lastLogin: {
            lt: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000),
          },
        },
      },
    });

    // Then delete users
    return tx.user.deleteMany({
      where: {
        lastLogin: {
          lt: new Date(Date.now() - 365 * 24 * 60 * 60 * 1000),
        },
        active: false,
      },
    });
  });
}
Transactions
typescript
// Interactive transactions
const transferResult = await prisma.$transaction(async (tx) => {
  // 1. Lock and check sender balance
  const sender = await tx.account.update({
    where: {
      id: senderId,
      balance: {
        gte: amount,
      },
    },
    data: {
      balance: {
        decrement: amount,
      },
    },
  });

  if (!sender) {
    throw new Error('Insufficient funds');
  }

  // 2. Credit receiver
  const receiver = await tx.account.update({
    where: {
      id: receiverId,
    },
    data: {
      balance: {
        increment: amount,
      },
    },
  });

  // 3. Create transaction record
  const transaction = await tx.transaction.create({
    data: {
      fromAccountId: senderId,
      toAccountId: receiverId,
      amount,
      currency: 'USD',
      status: 'COMPLETED',
      reference: `TX-${Date.now()}`,
    },
  });

  return {
    sender,
    receiver,
    transaction,
  };
});

// Sequential transactions
const sequentialOps = await prisma.$transaction([
  // Operation 1
  prisma.user.create({
    data: {
      email: 'new@example.com',
      name: 'New User',
    },
  }),
  
  // Operation 2
  prisma.profile.create({
    data: {
      userId: 'temp-id', // Will be replaced
      bio: 'New bio',
    },
  }),
  
  // Operation 3
  prisma.log.create({
    data: {
      action: 'USER_CREATED',
      userId: 'temp-id',
      metadata: { source: 'api' },
    },
  }),
]);

// Optimistic concurrency control
async function updateWithOptimisticLock(
  productId: string,
  updateData: Partial<Product>,
  expectedVersion: number
) {
  return prisma.$transaction(async (tx) => {
    // Check version first
    const product = await tx.product.findUnique({
      where: { id: productId },
      select: { version: true },
    });

    if (!product || product.version !== expectedVersion) {
      throw new Error('Concurrent modification detected');
    }

    // Update with version increment
    return tx.product.update({
      where: { id: productId },
      data: {
        ...updateData,
        version: {
          increment: 1,
        },
        updatedAt: new Date(),
      },
    });
  });
}

// Batch operations with transaction
async function processBatchOrders(orderIds: string[]) {
  return prisma.$transaction(
    async (tx) => {
      const results = [];

      for (const orderId of orderIds) {
        // Process each order
        const updatedOrder = await tx.order.update({
          where: { id: orderId },
          data: {
            status: 'PROCESSING',
            processedAt: new Date(),
          },
          include: {
            items: true,
          },
        });

        // Update inventory
        for (const item of updatedOrder.items) {
          await tx.product.update({
            where: { id: item.productId },
            data: {
              stock: {
                decrement: item.quantity,
              },
            },
          });
        }

        results.push(updatedOrder);
      }

      return results;
    },
    {
      maxWait: 5000, // Maximum time to wait for transaction
      timeout: 10000, // Maximum time for transaction to complete
    }
  );
}

// Nested transactions pattern (using savepoints)
async function complexOperation() {
  return prisma.$transaction(async (tx) => {
    try {
      // Main operation
      const user = await tx.user.create({
        data: { email: 'user@example.com' },
      });

      // Nested operation that might fail
      await nestedOperation(tx, user.id);

      return user;
    } catch (error) {
      // Transaction will be rolled back automatically
      console.error('Operation failed:', error);
      throw error;
    }
  });
}

async function nestedOperation(tx: Prisma.TransactionClient, userId: string) {
  // This is executed within the parent transaction
  return tx.profile.create({
    data: {
      userId,
      bio: 'Nested operation',
    },
  });
}
§ SOFT DELETE
Schema with deletedAt
prisma
// Base model with soft delete fields
model BaseModel {
  id        String   @id @default(cuid())
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  deletedAt DateTime? @map("deleted_at")
  
  @@map("base_models")
  @@index([deletedAt])
}

// Extend base model
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  deletedAt DateTime?
  
  // Relations with soft delete considerations
  posts     Post[]
  profile   Profile?
  
  @@index([deletedAt])
 

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-10-DATABASE-PRISMA
Prompt ID: 10 / 48
Parte: 2
Exported: 2026-02-06T13:01:34.541Z
Characters: 1885
════════════════════════════════════════════════════════════

back plan
interface MigrationPlan {
  migrationName: string;
  appliedAt: Date;
  rollbackSteps: RollbackStep[];
  estimatedDowntime: number;
  verificationSteps: VerificationStep[];
}

// Safe migration with verification
async function safeMigration(migrationName: string) {
  console.log(`Starting migration: ${migrationName}`);
  
  try {
    // 1. Pre-migration checks
    await verifyDatabaseConnectivity();
    await verifyDiskSpace();
    await verifyReplicationLag();
    
    // 2. Create backup point
    const backupId = await createBackupPoint();
    
    // 3. Apply migration
    await applyMigration(migrationName);
    
    // 4. Post-migration verification
    await verifyDataIntegrity();
    await verifyApplicationFunctionality();
    
    // 5. Update migration tracking
    await recordSuccessfulMigration(migrationName, backupId);
    
    console.log('Migration completed successfully');
  } catch (error) {
    console.error('Migration failed:', error);
    
    // 6. Rollback if needed
    await rollbackMigration(migrationName);
    
    throw error;
  }
}

// Migration status endpoint
export async function GET(request: Request) {
  const status = await prisma.$queryRaw`
    SELECT 
      name,
      applied_at,
      checksum,
      finished_at,
      EXTRACT(EPOCH FROM (finished_at - applied_at)) as duration_seconds
    FROM "_prisma_migrations"
    ORDER BY applied_at DESC
    LIMIT 10
  `;
  
  const pending = await prisma.$queryRaw`
    SELECT 
      datname as database,
      usename as user,
      application_name,
      state,
      query,
      age(now(), query_start) as query_age
    FROM pg_stat_activity
    WHERE query LIKE '%ALTER TABLE%' 
       OR query LIKE '%CREATE INDEX%'
       OR query LIKE '%DROP COLUMN%'
    ORDER BY query_start DESC
  `;
  
  return Response.json({
    status: 'healthy',
    lastMigrations: status,
   