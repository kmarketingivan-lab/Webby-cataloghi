# CATALOGO-DB-ARCHITECTURE

CATALOGO-DB-ARCHITECTURE per Next.js 14 + Prisma
§ DATABASE DESIGN PRINCIPLES
Normalization
prisma
// 1NF: Atomic values, no repeating groups
model User1NF {
  id        String   @id @default(cuid())
  email     String   @unique
  // Violazione 1NF: array di stringhe
  // phones   String[]  // NON VALIDO
  phones    Phone[]  // Corretto: tabella separata
}

model Phone {
  id       String  @id @default(cuid())
  userId   String
  number   String
  user     User    @relation(fields: [userId], references: [id])
}

// 2NF: Eliminare dipendenze parziali
model Order2NF {
  id         String    @id @default(cuid())
  orderDate  DateTime
  customerId String
  // Violazione 2NF: productName dipende da productId, non dalla chiave intera
  productId  String
  productName String  // NON VALIDO - spostare in Product
  
  // Soluzione corretta
  orderItems OrderItem[]
}

model Product {
  id   String @id @default(cuid())
  name String
}

// 3NF: Eliminare dipendenze transitive
model Employee3NF {
  id           String  @id @default(cuid())
  name         String
  departmentId String
  // Violazione 3NF: managerName dipende da managerId
  managerId    String
  managerName  String  // NON VALIDO
  
  department   Department @relation(fields: [departmentId], references: [id])
}

// BCNF: Tutte le determinanti sono superchiavi
model EnrollmentBCNF {
  studentId String
  courseId  String
  professorId String
  
  @@id([studentId, courseId])
  // Violazione BCNF se professorId determina courseId
  // Soluzione: separare in due tabelle
}
Denormalization when needed
prisma
// Per performance, denormalizzare dati frequentemente letti
model PostDenormalized {
  id          String   @id @default(cuid())
  title       String
  content     String
  authorId    String
  authorName  String  // Denormalizzato
  likesCount  Int     @default(0) // Denormalizzato
  commentsCount Int   @default(0) // Denormalizzato
  
  // Aggiornamento tramite trigger/Prisma middleware
  @@index([authorId])
}
ACID Properties
sql
-- Atomicity: Transazioni complete o rollback
BEGIN TRANSACTION;
INSERT INTO orders (id, amount) VALUES ('ord_123', 100);
INSERT INTO order_items (order_id, product_id) VALUES ('ord_123', 'prod_456');
-- Se fallisce, rollback completo
COMMIT;

-- Consistency: Vincoli DB
ALTER TABLE orders ADD CONSTRAINT check_amount_positive 
CHECK (amount >= 0);

-- Isolation: Livelli di isolamento
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Durability: WAL (Write-Ahead Logging) garantito
CAP Theorem Applicato
prisma
// CP (Consistency + Partition Tolerance): Database relazionale
model CPExample {
  id        String   @id @default(cuid())
  data      String
  version   Int      @default(1) // Per consistenza
  updatedAt DateTime @updatedAt
  
  @@index([updatedAt])
}

// AP (Availability + Partition Tolerance): Cache/Redis
// Schema per dati cache
model CachedData {
  key       String   @id
  value     Json
  expiresAt DateTime
  version   String
  
  @@index([expiresAt])
}

// Strategia ibrida: Consistency con eventual consistency
model EventualConsistent {
  id        String   @id @default(cuid())
  mainData  String
  cache     Json?    // Per disponibilità immediata
  syncVersion Int    @default(0)
  
  @@index([syncVersion])
}
§ SCHEMA DESIGN PATTERNS
User/Account/Profile Pattern
prisma
// Separazione responsabilità
model Account {
  id            String    @id @default(cuid())
  email         String    @unique
  passwordHash  String
  isActive      Boolean   @default(true)
  emailVerified Boolean   @default(false)
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relazioni
  profiles      Profile[]
  sessions      Session[]
  authentications Authentication[]
  
  // Indici per sicurezza
  @@index([email, isActive])
  @@index([createdAt])
}

model Profile {
  id          String   @id @default(cuid())
  accountId   String
  type        ProfileType // 'PERSONAL', 'BUSINESS', etc.
  firstName   String?
  lastName    String?
  avatarUrl   String?
  phone       String?
  locale      String   @default("en-US")
  timezone    String   @default("UTC")
  
  // Metadata
  metadata    Json?
  preferences Json?
  
  // Relazioni
  account     Account  @relation(fields: [accountId], references: [id])
  
  // Vincoli
  @@unique([accountId, type])
  @@index([accountId])
  @@index([type])
}

model Session {
  id           String   @id @default(cuid())
  accountId    String
  token        String   @unique @db.VarChar(512)
  expiresAt    DateTime
  userAgent    String?
  ipAddress    String?
  createdAt    DateTime @default(now())
  
  account      Account  @relation(fields: [accountId], references: [id])
  
  @@index([accountId])
  @@index([expiresAt])
  @@index([token])
}

model Authentication {
  id         String   @id @default(cuid())
  accountId  String
  provider   String   // 'google', 'github', etc.
  providerId String
  metadata   Json?
  
  account    Account  @relation(fields: [accountId], references: [id])
  
  @@unique([provider, providerId])
  @@index([accountId])
}

enum ProfileType {
  PERSONAL
  BUSINESS
  ADMIN
}
Multi-tenancy Schemas
prisma
// Approccio 1: Schema per tenant
// (Separate database schemas - isolamento completo)

// Approccio 2: Tenant ID in ogni tabella
model Tenant {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  plan        PlanType @default(FREE)
  settings    Json?
  createdAt   DateTime @default(now())
  
  // Relazioni
  users       TenantUser[]
  products    Product[]
  orders      Order[]
  
  @@index([slug])
}

model TenantUser {
  id        String   @id @default(cuid())
  tenantId  String
  userId    String
  role      UserRole @default(MEMBER)
  
  // Tenant-specific data
  settings  Json?
  
  // Relazioni
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  user      User     @relation(fields: [userId], references: [id])
  
  @@unique([tenantId, userId])
  @@index([tenantId])
  @@index([userId])
}

// Approccio 3: Row-level security
model ProductMultiTenant {
  id        String   @id @default(cuid())
  tenantId  String   // Filtro a livello applicativo
  name      String
  price     Decimal  @db.Decimal(10,2)
  
  tenant    Tenant   @relation(fields: [tenantId], references: [id])
  
  @@index([tenantId, name])
  @@index([tenantId, price])
}

enum PlanType {
  FREE
  PRO
  ENTERPRISE
}

enum UserRole {
  OWNER
  ADMIN
  MEMBER
  GUEST
}
Audit Trail Design
prisma
// Pattern: Audit table separata
model AuditLog {
  id          String    @id @default(cuid())
  entityType  String    // 'User', 'Order', etc.
  entityId    String
  action      AuditAction
  userId      String?
  userEmail   String?   // Denormalizzato per performance
  tenantId    String?
  
  // Dati cambio
  oldData     Json?
  newData     Json?
  changedFields String[] // Campi modificati
  
  // Metadata
  ipAddress   String?
  userAgent   String?
  timestamp   DateTime  @default(now())
  
  // Indici per query
  @@index([entityType, entityId])
  @@index([userId])
  @@index([tenantId])
  @@index([timestamp])
  @@index([action])
}

// Alternative: Versioning pattern
model DocumentWithHistory {
  id          String   @id @default(cuid())
  currentVersion Int   @default(1)
  content     String
  status      DocumentStatus
  
  // Relazione con storico
  versions    DocumentVersion[]
  
  @@index([status])
}

model DocumentVersion {
  id          String   @id @default(cuid())
  documentId  String
  version     Int
  content     String
  authorId    String
  createdAt   DateTime @default(now())
  changeNote  String?
  
  document    DocumentWithHistory @relation(fields: [documentId], references: [id])
  
  @@unique([documentId, version])
  @@index([documentId])
}

enum AuditAction {
  CREATE
  UPDATE
  DELETE
  SOFT_DELETE
  RESTORE
}

enum DocumentStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}
Soft Delete Pattern
prisma
// Implementazione completa soft delete
model SoftDeleteBase {
  id          String   @id @default(cuid())
  deletedAt   DateTime?
  deletedBy   String?
  
  // Indice parziale per escludere eliminati
  @@index([id], where: "deleted_at IS NULL")
}

// Estensione del pattern
model ProductSoftDelete {
  id          String    @id @default(cuid())
  name        String
  price       Decimal   @db.Decimal(10,2)
  
  // Soft delete fields
  deletedAt   DateTime?
  deletedBy   String?
  deleteReason String?
  
  // Version per ottimismo
  version     Int       @default(1)
  
  // Vincoli unici condizionali
  @@unique([name], where: "deleted_at IS NULL")
  @@index([deletedAt])
  
  // Middleware Prisma per gestione soft delete
}

// View SQL per dati attivi
// CREATE VIEW active_products AS 
// SELECT * FROM products WHERE deleted_at IS NULL;

// Stored procedure per cleanup
model DeletedItemsArchive {
  id          String   @id @default(cuid())
  originalId  String
  tableName   String
  data        Json     // Dati completi
  deletedAt   DateTime @default(now())
  deletedBy   String?
  
  @@index([tableName, originalId])
  @@index([deletedAt])
}
Versioning Pattern
prisma
// Pattern: Versioned entities
model VersionedProduct {
  id          String   @id @default(cuid())
  sku         String   @unique
  currentVersionId String?
  
  // Relazioni
  versions    ProductVersion[]
  current     ProductVersion? @relation("CurrentVersion", fields: [currentVersionId], references: [id])
  
  @@index([sku])
}

model ProductVersion {
  id          String   @id @default(cuid())
  productId   String
  version     Int      @default(1)
  name        String
  description String?
  price       Decimal  @db.Decimal(10,2)
  cost        Decimal  @db.Decimal(10,2)?
  attributes  Json?
  
  // Metadata versione
  isPublished Boolean  @default(false)
  publishedAt DateTime?
  authorId    String
  changeLog   String?
  
  // Validità temporale
  validFrom   DateTime @default(now())
  validUntil  DateTime?
  
  product     VersionedProduct @relation(fields: [productId], references: [id])
  nextVersion ProductVersion?  @relation("VersionChain")
  
  @@unique([productId, version])
  @@index([productId])
  @@index([validFrom, validUntil])
  @@index([isPublished])
}

// Pattern: Effective dating
model PricingEffectiveDate {
  id          String   @id @default(cuid())
  productId   String
  price       Decimal  @db.Decimal(10,2)
  
  // Date di validità
  effectiveFrom DateTime
  effectiveTo   DateTime?
  
  product     Product  @relation(fields: [productId], references: [id])
  
  @@index([productId, effectiveFrom])
  @@index([effectiveFrom, effectiveTo])
  
  // Vincolo: no sovrapposizioni date
  // (gestito a livello applicativo o trigger)
}
State Machine in DB
prisma
// State machine con validazione
model OrderStateMachine {
  id          String        @id @default(cuid())
  status      OrderStatus   @default(PENDING)
  previousStatus OrderStatus?
  
  // Timestamps per stato
  pendingAt   DateTime?
  confirmedAt DateTime?
  shippedAt   DateTime?
  deliveredAt DateTime?
  cancelledAt DateTime?
  
  // Transizioni permesse
  validTransitions OrderTransition[]
  
  // Eventi di stato
  stateEvents OrderStateEvent[]
  
  // Vincolo: solo transizioni valide
  @@index([status])
}

model OrderTransition {
  id          String      @id @default(cuid())
  fromState   OrderStatus
  toState     OrderStatus
  description String?
  isAllowed   Boolean     @default(true)
  
  @@unique([fromState, toState])
  @@index([fromState])
  @@index([toState])
}

model OrderStateEvent {
  id          String      @id @default(cuid())
  orderId     String
  fromState   OrderStatus?
  toState     OrderStatus
  triggeredBy String
  reason      String?
  metadata    Json?
  timestamp   DateTime    @default(now())
  
  order       OrderStateMachine @relation(fields: [orderId], references: [id])
  
  @@index([orderId])
  @@index([timestamp])
}

// Enumerazione stati
enum OrderStatus {
  DRAFT
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
  REFUNDED
}

// Validazione con CHECK constraint SQL
// ALTER TABLE orders ADD CONSTRAINT valid_status_transition
// CHECK (
//   (previous_status = 'DRAFT' AND status IN ('PENDING', 'CANCELLED')) OR
//   (previous_status = 'PENDING' AND status IN ('CONFIRMED', 'CANCELLED')) OR
//   ...
// );
§ RELATIONSHIP PATTERNS
One-to-one (Profile)
prisma
// Pattern: Uno-a-uno con chiave esterna
model UserOneToOne {
  id      String   @id @default(cuid())
  email   String   @unique
  
  // Relazione 1:1
  profile UserProfile?
}

model UserProfile {
  id        String   @id @default(cuid())
  userId    String   @unique  // Vincolo unique per 1:1
  bio       String?
  avatar    String?
  settings  Json?
  
  user      UserOneToOne @relation(fields: [userId], references: [id])
}

// Pattern: Tabella singola con colonne nullable
model UserCombined {
  id          String   @id @default(cuid())
  email       String   @unique
  password    String
  
  // Dati profilo (opzionali)
  bio         String?
  avatar      String?
  dateOfBirth DateTime?
  
  // Indici
  @@index([email])
}

// Pattern: Ereditarietà (Table per Type)
model BaseUser {
  id          String   @id @default(cuid())
  email       String   @unique
  userType    UserType
  
  // Discriminatore
  @@index([userType])
}

model CustomerUser {
  id          String   @id
  company     String?
  vatNumber   String?
  
  baseUser    BaseUser @relation(fields: [id], references: [id])
}

model AdminUser {
  id          String   @id
  permissions String[]
  department  String?
  
  baseUser    BaseUser @relation(fields: [id], references: [id])
}

enum UserType {
  CUSTOMER
  ADMIN
  MODERATOR
}
One-to-many (User → Posts)
prisma
// Pattern: Relazione standard 1:N
model AuthorOneToMany {
  id        String   @id @default(cuid())
  name      String
  email     String   @unique
  
  // Relazione 1:N
  posts     Post[]
  comments  Comment[]
}

model Post {
  id          String   @id @default(cuid())
  title       String
  content     String
  published   Boolean  @default(false)
  authorId    String
  
  // Relazioni
  author      AuthorOneToMany @relation(fields: [authorId], references: [id])
  comments    Comment[]
  tags        Tag[]
  
  // Indici
  @@index([authorId])
  @@index([published, createdAt])
}

model Comment {
  id        String   @id @default(cuid())
  content   String
  postId    String
  authorId  String
  parentId  String?  // Per commenti nidificati
  
  // Relazioni
  post      Post     @relation(fields: [postId], references: [id])
  author    AuthorOneToMany @relation(fields: [authorId], references: [id])
  parent    Comment? @relation("NestedComments", fields: [parentId], references: [id])
  replies   Comment[] @relation("NestedComments")
  
  @@index([postId])
  @@index([authorId])
  @@index([parentId])
}

// Pattern: Con ordering
model OrderedItems {
  id        String   @id @default(cuid())
  name      String
  position  Int      // Per ordinamento esplicito
  listId    String
  
  list      ItemList @relation(fields: [listId], references: [id])
  
  @@index([listId, position])
}
Many-to-many (Tags, Categories)
prisma
// Pattern: Tabella di join esplicita
model PostManyToMany {
  id        String   @id @default(cuid())
  title     String
  content   String
  
  // Relazioni many-to-many
  tags      PostTag[]
  categories PostCategory[]
}

model Tag {
  id          String   @id @default(cuid())
  name        String   @unique
  slug        String   @unique
  description String?
  
  // Relazioni
  posts       PostTag[]
  
  @@index([slug])
}

model PostTag {
  id        String   @id @default(cuid())
  postId    String
  tagId     String
  assignedAt DateTime @default(now())
  assignedBy String?
  
  // Relazioni
  post      PostManyToMany @relation(fields: [postId], references: [id])
  tag       Tag            @relation(fields: [tagId], references: [id])
  
  @@unique([postId, tagId])
  @@index([postId])
  @@index([tagId])
}

// Pattern: Many-to-many con attributi extra
model UserRoleManyToMany {
  id        String   @id @default(cuid())
  userId    String
  roleId    String
  assignedAt DateTime @default(now())
  assignedBy String
  expiresAt  DateTime?
  
  // Attributi extra
  metadata  Json?
  
  user      User    @relation(fields: [userId], references: [id])
  role      Role    @relation(fields: [roleId], references: [id])
  
  @@unique([userId, roleId], where: "expires_at IS NULL")
  @@index([userId])
  @@index([roleId])
  @@index([expiresAt])
}

// Pattern: Self-referencing many-to-many
model CategoryHierarchy {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  parentId    String?
  
  // Auto-relazione
  parent      CategoryHierarchy? @relation("CategoryTree", fields: [parentId], references: [id])
  children    CategoryHierarchy[] @relation("CategoryTree")
  
  // Many-to-many con prodotti
  products    ProductCategory[]
  
  // Materialized path per query efficienti
  path        String   @default("") // es: "1/2/3"
  
  @@index([slug])
  @@index([parentId])
  @@index([path])
}

// Closure table pattern per alberi profondi
model CategoryClosure {
  id          String   @id @default(cuid())
  ancestorId  String
  descendantId String
  depth       Int
  
  ancestor    Category @relation("Ancestor", fields: [ancestorId], references: [id])
  descendant  Category @relation("Descendant", fields: [descendantId], references: [id])
  
  @@unique([ancestorId, descendantId])
  @@index([ancestorId])
  @@index([descendantId])
  @@index([depth])
}
Self-referential (Comments, Categories tree)
prisma
// Pattern: Commenti annidati
model NestedComment {
  id          String   @id @default(cuid())
  content     String
  authorId    String
  postId      String
  parentId    String?
  
  // Auto-relazione per gerarchia
  parent      NestedComment? @relation("CommentReplies", fields: [parentId], references: [id])
  replies     NestedComment[] @relation("CommentReplies")
  
  // Per query ricorsive efficienti
  path        String   @default("") // Materialized path
  depth       Int      @default(0)
  
  // Indici
  @@index([postId])
  @@index([parentId])
  @@index([path])
  @@index([depth])
}

// Pattern: Adjacency list con CTE (Common Table Expressions)
// Query ricorsiva SQL:
/*
WITH RECURSIVE comment_tree AS (
  SELECT id, content, parent_id, 1 as depth
  FROM comments 
  WHERE parent_id IS NULL AND post_id = ?
  
  UNION ALL
  
  SELECT c.id, c.content, c.parent_id, ct.depth + 1
  FROM comments c
  INNER JOIN comment_tree ct ON c.parent_id = ct.id
)
SELECT * FROM comment_tree ORDER BY depth, created_at;
*/

// Pattern: Modified Preorder Tree Traversal (MPTT)
model CategoryMPTT {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  
  // MPTT fields
  lft         Int      // Left boundary
  rgt         Int      // Right boundary
  depth       Int      @default(0)
  
  // Vincolo: lft < rgt
  @@index([lft])
  @@index([rgt])
  @@index([depth])
  @@unique([lft, rgt])
}

// Query per sotto-albero MPTT:
// SELECT * FROM categories WHERE lft > ? AND rgt < ? ORDER BY lft;
Polymorphic associations
prisma
// Pattern: Join table polimorfica
model PolymorphicComment {
  id            String   @id @default(cuid())
  content       String
  authorId      String
  
  // Relazione polimorfica
  commentableType CommentableType
  commentableId   String
  
  // Indice composto
  @@index([commentableType, commentableId])
  @@index([authorId])
}

enum CommentableType {
  POST
  PRODUCT
  PAGE
  ARTICLE
}

// Pattern: Tabella di join polimorfica
model Attachment {
  id            String   @id @default(cuid())
  filename      String
  url           String
  mimeType      String
  size          Int
  
  // Polimorfismo
  attachableType AttachableType
  attachableId   String
  
  @@index([attachableType, attachableId])
}

// Pattern: Ereditarietà con Single Table Inheritance (STI)
model ContentItem {
  id          String   @id @default(cuid())
  type        ContentType
  title       String
  content     String?
  
  // Campi specifici per tipo (nullable)
  // Per Article
  excerpt     String?
  readTime    Int?
  
  // Per Video
  videoUrl    String?
  duration    Int?
  
  // Per Podcast
  audioUrl    String?
  episodeNumber Int?
  
  @@index([type])
}

enum ContentType {
  ARTICLE
  VIDEO
  PODCAST
  EBOOK
}

// Pattern: Class Table Inheritance (CTI)
model BaseContent {
  id          String   @id @default(cuid())
  type        ContentType
  title       String
  createdAt   DateTime @default(now())
  
  @@index([type])
}

model ArticleContent {
  id          String   @id
  content     String
  excerpt     String
  readTime    Int
  
  baseContent BaseContent @relation(fields: [id], references: [id])
}

model VideoContent {
  id          String   @id
  videoUrl    String
  duration    Int
  transcript  String?
  
  baseContent BaseContent @relation(fields: [id], references: [id])
}
§ E-COMMERCE SCHEMA
Products, Variants, Attributes
prisma
model ProductECommerce {
  id            String    @id @default(cuid())
  sku           String    @unique
  name          String
  description   String?
  slug          String    @unique
  
  // Pricing
  basePrice     Decimal   @db.Decimal(10,2)
  comparePrice  Decimal   @db.Decimal(10,2)?
  costPrice     Decimal   @db.Decimal(10,2)?
  
  // Inventory
  trackInventory Boolean  @default(true)
  inventoryType  InventoryType @default(SINGLE)
  
  // Shipping
  weight        Decimal?  @db.Decimal(8,3) // kg
  dimensions    Json?     // {length, width, height, unit}
  
  // Media
  images        Json      // Array di URL
  mainImage     String?
  
  // SEO
  seoTitle      String?
  seoDescription String?
  
  // Status
  status        ProductStatus @default(DRAFT)
  isFeatured    Boolean  @default(false)
  
  // Timestamps
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
  publishedAt   DateTime?
  
  // Relazioni
  variants      ProductVariant[]
  attributes    ProductAttribute[]
  categories    ProductCategory[]
  reviews       ProductReview[]
  
  // Indici
  @@index([sku])
  @@index([slug])
  @@index([status])
  @@index([createdAt])
  @@index([isFeatured])
  @@index([publishedAt])
}

model ProductVariant {
  id            String    @id @default(cuid())
  productId     String
  sku           String    @unique
  name          String?
  
  // Pricing override
  price         Decimal?  @db.Decimal(10,2)
  comparePrice  Decimal?  @db.Decimal(10,2)
  
  // Inventory per variant
  inventory     ProductInventory?
  
  // Attributes specifici
  attributes    Json      // {color: "red", size: "M"}
  
  // Media specifici
  image         String?
  
  // Status
  isActive      Boolean   @default(true)
  
  product       ProductECommerce @relation(fields: [productId], references: [id])
  
  @@unique([productId, sku])
  @@index([productId])
  @@index([sku])
  @@index([isActive])
}

model ProductAttribute {
  id            String    @id @default(cuid())
  productId     String
  name          String    // "Color", "Size", etc.
  values        String[]  // ["Red", "Blue", "Green"]
  isRequired    Boolean   @default(false)
  displayOrder  Int       @default(0)
  
  product       ProductECommerce @relation(fields: [productId], references: [id])
  
  @@index([productId])
  @@index([displayOrder])
}

model ProductInventory {
  id            String    @id @default(cuid())
  variantId     String    @unique
  quantity      Int       @default(0)
  lowStockThreshold Int   @default(5)
  isAvailable   Boolean   @default(true)
  
  // Location tracking
  location      String?
  
  // Reservation system
  reserved      Int       @default(0)
  
  variant       ProductVariant @relation(fields: [variantId], references: [id])
  
  @@index([quantity])
  @@index([isAvailable])
}

enum InventoryType {
  SINGLE      // Un solo prodotto
  VARIANT     // Varianti con inventory separato
  BUNDLE      // Bundle di prodotti
  DIGITAL     // Prodotto digitale
}

enum ProductStatus {
  DRAFT
  ACTIVE
  INACTIVE
  ARCHIVED
  OUT_OF_STOCK
}
Categories hierarchy
prisma
model CategoryECommerce {
  id            String    @id @default(cuid())
  name          String
  slug          String    @unique
  description   String?
  image         String?
  
  // Gerarchia
  parentId      String?
  parent        CategoryECommerce? @relation("CategoryHierarchy", fields: [parentId], references: [id])
  children      CategoryECommerce[] @relation("CategoryHierarchy")
  
  // MPTT per query efficienti
  lft           Int?
  rgt           Int?
  depth         Int       @default(0)
  
  // Display
  displayOrder  Int       @default(0)
  isActive      Boolean   @default(true)
  isFeatured    Boolean   @default(false)
  
  // SEO
  seoTitle      String?
  seoDescription String?
  
  // Relazioni
  products      ProductCategory[]
  
  // Indici
  @@index([slug])
  @@index([parentId])
  @@index([displayOrder])
  @@index([isActive])
  @@index([lft, rgt])
}

model ProductCategory {
  id            String    @id @default(cuid())
  productId     String
  categoryId    String
  isPrimary     Boolean   @default(false)
  
  product       ProductECommerce @relation(fields: [productId], references: [id])
  category      CategoryECommerce @relation(fields: [categoryId], references: [id])
  
  @@unique([productId, categoryId])
  @@index([productId])
  @@index([categoryId])
  @@index([isPrimary])
}
Cart and CartItems
prisma
model Cart {
  id            String    @id @default(cuid())
  userId        String?
  sessionId     String?   // Per utenti non loggati
  customerEmail String?
  
  // Currency e locale
  currency      String    @default("USD")
  language      String    @default("en")
  
  // Metadata
  metadata      Json?
  
  // Timestamps
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  expiresAt     DateTime  // Abbandono carrello
  
  // Relazioni
  items         CartItem[]
  coupons       CartCoupon[]
  
  // Indici
  @@index([userId])
  @@index([sessionId])
  @@index([expiresAt])
  @@unique([userId], where: "user_id IS NOT NULL")
  @@unique([sessionId], where: "session_id IS NOT NULL")
}

model CartItem {
  id            String    @id @default(cuid())
  cartId        String
  productId     String
  variantId     String?
  
  // Quantità e pricing
  quantity      Int       @default(1)
  unitPrice     Decimal   @db.Decimal(10,2)
  comparePrice  Decimal?  @db.Decimal(10,2)
  
  // Dati snapshot (immutabili)
  productName   String
  productImage  String?
  productSku    String
  
  // Customizzazioni
  customizations Json?
  
  // Timestamps
  addedAt       DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  cart          Cart      @relation(fields: [cartId], references: [id])
  product       ProductECommerce @relation(fields: [productId], references: [id])
  variant       ProductVariant? @relation(fields: [variantId], references: [id])
  
  @@index([cartId])
  @@index([productId])
  @@index([variantId])
  
  // Vincolo: quantità positiva
  // @@map("cart_items")
}

model CartCoupon {
  id            String    @id @default(cuid())
  cartId        String
  couponCode    String
  discountAmount Decimal  @db.Decimal(10,2)
  discountType  DiscountType
  
  appliedAt     DateTime  @default(now())
  
  cart          Cart      @relation(fields: [cartId], references: [id])
  
  @@unique([cartId, couponCode])
  @@index([cartId])
}

enum DiscountType {
  PERCENTAGE
  FIXED_AMOUNT
  FREE_SHIPPING
}
Orders and OrderItems
prisma
model OrderECommerce {
  id            String    @id @default(cuid())
  orderNumber   String    @unique // Formattato: ORD-YYYYMMDD-0001
  customerId    String?
  
  // Status
  status        OrderStatus @default(PENDING)
  fulfillmentStatus FulfillmentStatus @default(UNFULFILLED)
  paymentStatus PaymentStatus @default(PENDING)
  
  // Customer info
  customerEmail String
  customerName  String?
  customerPhone String?
  
  // Shipping
  shippingAddress Json
  billingAddress  Json
  shippingMethod  String?
  shippingPrice   Decimal   @db.Decimal(10,2) @default(0)
  
  // Pricing
  subtotal       Decimal   @db.Decimal(10,2)
  taxTotal       Decimal   @db.Decimal(10,2) @default(0)
  discountTotal  Decimal   @db.Decimal(10,2) @default(0)
  total          Decimal   @db.Decimal(10,2)
  
  // Currency
  currency       String    @default("USD")
  
  // Payment
  paymentMethod  String?
  transactionId  String?
  
  // Metadata
  notes          String?
  metadata       Json?
  
  // Timestamps
  createdAt      DateTime  @default(now())
  updatedAt      DateTime  @updatedAt
  paidAt         DateTime?
  shippedAt      DateTime?
  deliveredAt    DateTime?
  cancelledAt    DateTime?
  
  // Relazioni
  items          OrderItem[]
  payments       Payment[]
  fulfillments   Fulfillment[]
  
  // Indici
  @@index([orderNumber])
  @@index([customerId])
  @@index([customerEmail])
  @@index([status])
  @@index([createdAt])
  @@index([paymentStatus])
}

model OrderItem {
  id            String    @id @default(cuid())
  orderId       String
  productId     String
  variantId     String?
  
  // Dati snapshot (immutabili)
  productName   String
  productSku    String
  variantName   String?
  variantSku    String?
  productImage  String?
  
  // Pricing snapshot
  unitPrice     Decimal   @db.Decimal(10,2)
  comparePrice  Decimal?  @db.Decimal(10,2)
  quantity      Int
  
  // Calcolati
  subtotal      Decimal   @db.Decimal(10,2)
  tax           Decimal   @db.Decimal(10,2) @default(0)
  discount      Decimal   @db.Decimal(10,2) @default(0)
  total         Decimal   @db.Decimal(10,2)
  
  // Customizzazioni
  customizations Json?
  
  // Inventory tracking
  inventoryStatus InventoryStatus @default(UNPROCESSED)
  
  order         OrderECommerce @relation(fields: [orderId], references: [

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-12-DB-ARCHITECTURE
Prompt ID: 12 / 48
Parte: 2
Exported: 2026-02-06T13:17:40.279Z
Characters: 325
════════════════════════════════════════════════════════════

  WEEKLY
}
Messages/Conversations
prisma
model Conversation {
  id            String    @id @default(cuid())
  type          ConversationType @default(DIRECT)
  name          String?   // Per gruppi
  description   String?   // Per gruppi
  avatarUrl     String?   // Per gruppi
  
  // Settings
  settings     

## § ADVANCED PATTERNS: DB ARCHITECTURE

### Server Actions con Validazione

```typescript
// app/actions/db-architecture.ts
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


### DB ARCHITECTURE - Utility Helper #6

```typescript
// lib/utils/db-architecture-helper-6.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #23

```typescript
// lib/utils/db-architecture-helper-23.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #419

```typescript
// lib/utils/db-architecture-helper-419.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #487

```typescript
// lib/utils/db-architecture-helper-487.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #204

```typescript
// lib/utils/db-architecture-helper-204.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #23

```typescript
// lib/utils/db-architecture-helper-23.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #757

```typescript
// lib/utils/db-architecture-helper-757.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #725

```typescript
// lib/utils/db-architecture-helper-725.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #71

```typescript
// lib/utils/db-architecture-helper-71.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #864

```typescript
// lib/utils/db-architecture-helper-864.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #373

```typescript
// lib/utils/db-architecture-helper-373.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #691

```typescript
// lib/utils/db-architecture-helper-691.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #680

```typescript
// lib/utils/db-architecture-helper-680.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #334

```typescript
// lib/utils/db-architecture-helper-334.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #54

```typescript
// lib/utils/db-architecture-helper-54.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #443

```typescript
// lib/utils/db-architecture-helper-443.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #313

```typescript
// lib/utils/db-architecture-helper-313.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  getConfig(): Readonly<DBARCHITECTUREConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### DB ARCHITECTURE - Utility Helper #839

```typescript
// lib/utils/db-architecture-helper-839.ts
import { z } from "zod";

interface DBARCHITECTUREConfig {
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

export class DBARCHITECTUREProcessor<TInput, TOutput> {
  private config: DBARCHITECTUREConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<DBARCHITECTUREConfig> = {}) {
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

  addProcessor(processor: (input: TInput) => Prom