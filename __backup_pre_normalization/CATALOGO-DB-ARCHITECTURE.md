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