# CATALOGO-MARKETPLACE-MULTIVENDOR

Next.js 14 + Prisma Multi-Vendor Marketplace Catalog
§ MULTI-VENDOR DATA MODEL
prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  ADMIN
  VENDOR
  CUSTOMER
}

enum VendorStatus {
  PENDING
  UNDER_REVIEW
  APPROVED
  SUSPENDED
  REJECTED
}

enum ProductStatus {
  DRAFT
  PENDING_REVIEW
  APPROVED
  REJECTED
  ARCHIVED
}

enum OrderStatus {
  PENDING
  PROCESSING
  PARTIALLY_FULFILLED
  FULFILLED
  CANCELLED
  REFUNDED
}

enum PayoutStatus {
  PENDING
  PROCESSING
  COMPLETED
  FAILED
  ON_HOLD
}

enum CommissionType {
  PERCENTAGE
  FIXED
  TIERED
}

model User {
  id                String        @id @default(cuid())
  email             String        @unique
  name              String?
  password          String
  role              UserRole      @default(CUSTOMER)
  emailVerified     DateTime?
  image             String?
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt

  // Vendor specific
  vendorProfile     Vendor?
  customerProfile   Customer?
  adminProfile      Admin?

  // Auth
  sessions          Session[]
  accounts          Account[]
}

model Vendor {
  id                String        @id @default(cuid())
  userId            String        @unique
  user              User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  businessName      String
  businessEmail     String
  businessPhone     String?
  businessAddress   Json?
  taxId             String?
  status            VendorStatus  @default(PENDING)
  verifiedAt        DateTime?
  verificationNotes String?
  
  // Store
  store             Store?
  
  // Financial
  commissionRate    Float?        @default(15.0)
  balance           Decimal       @default(0)
  pendingBalance    Decimal       @default(0)
  
  // Documents
  documents         VendorDocument[]
  
  // Relationships
  products          Product[]
  orders            VendorOrder[]
  payouts           Payout[]
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Store {
  id                String        @id @default(cuid())
  vendorId          String        @unique
  vendor            Vendor        @relation(fields: [vendorId], references: [id], onDelete: Cascade)
  name              String
  slug              String        @unique
  description       String?
  logo              String?
  banner            String?
  contactEmail      String
  contactPhone      String?
  returnPolicy      String?
  shippingPolicy    String?
  
  // Settings
  settings          Json?         // Store-specific settings
  
  // Stats
  rating            Float?        @default(0)
  totalReviews      Int           @default(0)
  
  // Social
  socialLinks       Json?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model VendorDocument {
  id                String        @id @default(cuid())
  vendorId          String
  vendor            Vendor        @relation(fields: [vendorId], references: [id], onDelete: Cascade)
  type              String        // 'business_license', 'id_card', 'tax_certificate', etc.
  url               String
  verified          Boolean       @default(false)
  verifiedBy        String?
  verifiedAt        DateTime?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Customer {
  id                String        @id @default(cuid())
  userId            String        @unique
  user              User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  shippingAddresses Address[]
  billingAddresses  Address[]
  phone             String?
  
  // Relationships
  orders            Order[]
  reviews           Review[]
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Admin {
  id                String        @id @default(cuid())
  userId            String        @unique
  user              User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  permissions       String[]      // JSON array of permissions
  department        String?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Product {
  id                String        @id @default(cuid())
  vendorId          String
  vendor            Vendor        @relation(fields: [vendorId], references: [id], onDelete: Cascade)
  storeId           String
  store             Store         @relation(fields: [storeId], references: [id], onDelete: Cascade)
  
  // Basic info
  name              String
  slug              String        @unique
  description       String?
  shortDescription  String?
  
  // Categories
  categoryId        String?
  category          Category?     @relation(fields: [categoryId], references: [id])
  
  // Status
  status            ProductStatus @default(DRAFT)
  rejectionReason   String?
  publishedAt       DateTime?
  
  // Inventory
  sku               String?
  stockQuantity     Int           @default(0)
  lowStockThreshold Int           @default(5)
  manageStock       Boolean       @default(true)
  
  // Pricing
  price             Decimal
  compareAtPrice    Decimal?
  costPrice         Decimal?
  
  // Media
  images            Json          // Array of image URLs
  featuredImage     String?
  
  // Attributes
  attributes        Json?         // Product attributes
  
  // Variants
  hasVariants       Boolean       @default(false)
  variants          ProductVariant[]
  
  // Relationships
  reviews           Review[]
  
  // Stats
  viewCount         Int           @default(0)
  salesCount        Int           @default(0)
  
  // SEO
  seoTitle          String?
  seoDescription    String?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([vendorId])
  @@index([storeId])
  @@index([categoryId])
  @@index([status])
}

model ProductVariant {
  id                String        @id @default(cuid())
  productId         String
  product           Product       @relation(fields: [productId], references: [id], onDelete: Cascade)
  
  // Variant options
  options           Json          // { size: "M", color: "Red" }
  sku               String?
  
  // Pricing & Inventory
  price             Decimal?
  compareAtPrice    Decimal?
  costPrice         Decimal?
  stockQuantity     Int           @default(0)
  
  // Media
  image             String?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([productId])
}

model Category {
  id                String        @id @default(cuid())
  name              String
  slug              String        @unique
  description       String?
  parentId          String?
  parent            Category?     @relation("CategoryToCategory", fields: [parentId], references: [id])
  children          Category[]    @relation("CategoryToCategory")
  image             String?
  
  // Ordering
  position          Int           @default(0)
  
  // SEO
  seoTitle          String?
  seoDescription    String?
  
  // Relationships
  products          Product[]
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Order {
  id                String        @id @default(cuid())
  orderNumber       String        @unique @default(uuid())
  customerId        String
  customer          Customer      @relation(fields: [customerId], references: [id], onDelete: Cascade)
  
  // Status
  status            OrderStatus   @default(PENDING)
  
  // Pricing
  subtotal          Decimal
  taxAmount         Decimal       @default(0)
  shippingAmount    Decimal       @default(0)
  discountAmount    Decimal       @default(0)
  total             Decimal
  
  // Addresses
  shippingAddress   Json
  billingAddress    Json
  
  // Payment
  paymentMethod     String
  paymentStatus     String        @default("pending")
  transactionId     String?
  
  // Vendor orders
  vendorOrders      VendorOrder[]
  
  // Notes
  customerNote      String?
  adminNote         String?
  
  // Fulfillment
  fulfilledAt       DateTime?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([customerId])
  @@index([status])
}

model VendorOrder {
  id                String        @id @default(cuid())
  orderId           String
  order             Order         @relation(fields: [orderId], references: [id], onDelete: Cascade)
  vendorId          String
  vendor            Vendor        @relation(fields: [vendorId], references: [id])
  storeId           String
  store             Store         @relation(fields: [storeId], references: [id])
  
  // Status
  status            OrderStatus   @default(PENDING)
  
  // Pricing
  subtotal          Decimal
  taxAmount         Decimal
  shippingAmount    Decimal
  discountAmount    Decimal
  total             Decimal
  
  // Commission
  commissionRate    Float
  commissionAmount  Decimal
  vendorEarnings    Decimal
  
  // Items
  items             VendorOrderItem[]
  
  // Shipping
  shippingMethod    String?
  trackingNumber    String?
  shippedAt         DateTime?
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@unique([orderId, vendorId])
  @@index([vendorId])
  @@index([status])
}

model VendorOrderItem {
  id                String        @id @default(cuid())
  vendorOrderId     String
  vendorOrder       VendorOrder   @relation(fields: [vendorOrderId], references: [id], onDelete: Cascade)
  
  // Product info
  productId         String
  productName       String
  productSku        String?
  variantId         String?
  variantOptions    Json?
  
  // Pricing
  unitPrice         Decimal
  quantity          Int
  totalPrice        Decimal
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([vendorOrderId])
}

model Payout {
  id                String        @id @default(cuid())
  vendorId          String
  vendor            Vendor        @relation(fields: [vendorId], references: [id], onDelete: Cascade)
  
  // Amount
  amount            Decimal
  currency          String        @default("USD")
  
  // Status
  status            PayoutStatus  @default(PENDING)
  
  // Method
  payoutMethod      String        // 'stripe', 'paypal', 'bank_transfer'
  payoutDetails     Json          // Method-specific details
  
  // Timing
  scheduledFor      DateTime
  processedAt       DateTime?
  
  // Reference
  referenceId       String?
  failureReason     String?
  
  // Related orders
  vendorOrderIds    String[]      // Array of vendor order IDs
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([vendorId])
  @@index([status])
  @@index([scheduledFor])
}

model Review {
  id                String        @id @default(cuid())
  productId         String
  product           Product       @relation(fields: [productId], references: [id], onDelete: Cascade)
  customerId        String
  customer          Customer      @relation(fields: [customerId], references: [id], onDelete: Cascade)
  orderId           String?
  order             Order?        @relation(fields: [orderId], references: [id])
  
  // Rating
  rating            Int           // 1-5
  title             String?
  comment           String?
  
  // Verification
  verifiedPurchase  Boolean       @default(false)
  
  // Status
  status            String        @default("pending") // pending, approved, rejected
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@unique([productId, customerId, orderId])
  @@index([productId])
  @@index([customerId])
}

model Address {
  id                String        @id @default(cuid())
  customerId        String
  customer          Customer      @relation(fields: [customerId], references: [id], onDelete: Cascade)
  
  // Address fields
  firstName         String
  lastName          String
  company           String?
  address1          String
  address2          String?
  city              String
  state             String
  postalCode        String
  country           String
  phone             String?
  
  // Type
  type              String        @default("shipping") // shipping or billing
  isDefault         Boolean       @default(false)
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
  
  @@index([customerId])
}

// Auth models (for NextAuth)
model Account {
  id                String        @id @default(cuid())
  userId            String
  user              User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?       @db.Text
  access_token      String?       @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?       @db.Text
  session_state     String?
  
  @@unique([provider, providerAccountId])
  @@index([userId])
}

model Session {
  id                String        @id @default(cuid())
  sessionToken      String        @unique
  userId            String
  user              User          @relation(fields: [userId], references: [id], onDelete: Cascade)
  expires           DateTime
  
  @@index([userId])
}

model VerificationToken {
  identifier        String
  token             String        @unique
  expires           DateTime
  
  @@unique([identifier, token])
}
typescript
// types/index.ts
export type VendorStatus = 'PENDING' | 'UNDER_REVIEW' | 'APPROVED' | 'SUSPENDED' | 'REJECTED';
export type ProductStatus = 'DRAFT' | 'PENDING_REVIEW' | 'APPROVED' | 'REJECTED' | 'ARCHIVED';
export type OrderStatus = 'PENDING' | 'PROCESSING' | 'PARTIALLY_FULFILLED' | 'FULFILLED' | 'CANCELLED' | 'REFUNDED';
export type PayoutStatus = 'PENDING' | 'PROCESSING' | 'COMPLETED' | 'FAILED' | 'ON_HOLD';
export type CommissionType = 'PERCENTAGE' | 'FIXED' | 'TIERED';

export interface Vendor {
  id: string;
  userId: string;
  businessName: string;
  businessEmail: string;
  status: VendorStatus;
  store?: Store;
  commissionRate: number;
  balance: number;
  pendingBalance: number;
}

export interface Store {
  id: string;
  vendorId: string;
  name: string;
  slug: string;
  description?: string;
  rating?: number;
  totalReviews: number;
}

export interface Product {
  id: string;
  vendorId: string;
  storeId: string;
  name: string;
  slug: string;
  status: ProductStatus;
  price: number;
  stockQuantity: number;
  hasVariants: boolean;
  variants?: ProductVariant[];
}

export interface Order {
  id: string;
  customerId: string;
  status: OrderStatus;
  total: number;
  vendorOrders: VendorOrder[];
}

export interface VendorOrder {
  id: string;
  orderId: string;
  vendorId: string;
  status: OrderStatus;
  total: number;
  commissionAmount: number;
  vendorEarnings: number;
}

export interface CommissionRule {
  id: string;
  type: CommissionType;
  rate: number;
  fixedAmount?: number;
  tieredRates?: Array<{
    minAmount: number;
    maxAmount?: number;
    rate: number;
  }>;
  categoryId?: string;
  vendorId?: string;
}

export interface Payout {
  id: string;
  vendorId: string;
  amount: number;
  status: PayoutStatus;
  payoutMethod: string;
  scheduledFor: Date;
}
§ VENDOR ONBOARDING
typescript
// app/api/vendor/onboarding/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { hash } from 'bcryptjs';
import { z } from 'zod';

const prisma = new PrismaClient();

const vendorOnboardingSchema = z.object({
  // User info
  email: z.string().email(),
  password: z.string().min(8),
  name: z.string().min(2),
  
  // Business info
  businessName: z.string().min(2),
  businessEmail: z.string().email(),
  businessPhone: z.string().optional(),
  
  // Address
  businessAddress: z.object({
    street: z.string(),
    city: z.string(),
    state: z.string(),
    postalCode: z.string(),
    country: z.string(),
  }),
  
  // Legal
  taxId: z.string().optional(),
  businessType: z.enum(['individual', 'company', 'llc']),
  
  // Store info
  storeName: z.string().min(2),
  storeSlug: z.string().min(2).regex(/^[a-z0-9-]+$/),
  storeDescription: z.string().optional(),
  
  // Documents
  documents: z.array(z.object({
    type: z.enum(['business_license', 'id_card', 'tax_certificate', 'bank_statement']),
    url: z.string().url(),
  })).min(1),
  
  // Terms
  acceptTerms: z.literal(true),
  acceptPrivacyPolicy: z.literal(true),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const data = vendorOnboardingSchema.parse(body);

    // Check if email already exists
    const existingUser = await prisma.user.findUnique({
      where: { email: data.email },
    });

    if (existingUser) {
      return NextResponse.json(
        { error: 'Email already registered' },
        { status: 400 }
      );
    }

    // Check if store slug is available
    const existingStore = await prisma.store.findUnique({
      where: { slug: data.storeSlug },
    });

    if (existingStore) {
      return NextResponse.json(
        { error: 'Store slug already taken' },
        { status: 400 }
      );
    }

    // Hash password
    const hashedPassword = await hash(data.password, 12);

    // Create user and vendor in transaction
    const result = await prisma.$transaction(async (tx) => {
      // Create user
      const user = await tx.user.create({
        data: {
          email: data.email,
          name: data.name,
          password: hashedPassword,
          role: 'VENDOR',
        },
      });

      // Create vendor
      const vendor = await tx.vendor.create({
        data: {
          userId: user.id,
          businessName: data.businessName,
          businessEmail: data.businessEmail,
          businessPhone: data.businessPhone,
          businessAddress: data.businessAddress,
          taxId: data.taxId,
          status: 'PENDING',
        },
      });

      // Create store
      const store = await tx.store.create({
        data: {
          vendorId: vendor.id,
          name: data.storeName,
          slug: data.storeSlug,
          description: data.storeDescription,
          contactEmail: data.businessEmail,
        },
      });

      // Create documents
      if (data.documents.length > 0) {
        await tx.vendorDocument.createMany({
          data: data.documents.map(doc => ({
            vendorId: vendor.id,
            type: doc.type,
            url: doc.url,
          })),
        });
      }

      // Create vendor terms acceptance record
      await tx.vendorTermsAcceptance.create({
        data: {
          vendorId: vendor.id,
          termsVersion: '1.0',
          privacyPolicyVersion: '1.0',
          acceptedAt: new Date(),
        },
      });

      // Send notification to admin
      await tx.notification.create({
        data: {
          type: 'VENDOR_SIGNUP',
          title: 'New Vendor Registration',
          message: `${data.businessName} has registered as a vendor`,
          metadata: {
            vendorId: vendor.id,
            storeName: data.storeName,
          },
          recipients: ['ADMIN'],
        },
      });

      return { user, vendor, store };
    });

    // Send verification email
    // TODO: Implement email sending

    return NextResponse.json({
      success: true,
      message: 'Vendor registration submitted for review',
      vendorId: result.vendor.id,
    });
  } catch (error) {
    console.error('Vendor onboarding error:', error);
    return NextResponse.json(
      { error: 'Registration failed' },
      { status: 500 }
    );
  }
}

// Additional models needed
const vendorTermsAcceptanceSchema = `
model VendorTermsAcceptance {
  id                String        @id @default(cuid())
  vendorId          String        @unique
  vendor            Vendor        @relation(fields: [vendorId], references: [id], onDelete: Cascade)
  termsVersion      String
  privacyPolicyVersion String
  acceptedAt        DateTime      @default(now())
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}

model Notification {
  id                String        @id @default(cuid())
  type              String        // VENDOR_SIGNUP, ORDER_PLACED, etc.
  title             String
  message           String
  metadata          Json?
  recipients        String[]      // ADMIN, VENDOR, CUSTOMER or specific IDs
  read              Boolean       @default(false)
  readBy            String[]      // User IDs who read it
  
  createdAt         DateTime      @default(now())
  updatedAt         DateTime      @updatedAt
}
`;
typescript
// components/vendor/onboarding/VendorWizard.tsx
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Stepper, Step } from '@/components/ui/stepper';

const steps = [
  { id: 1, name: 'Account', description: 'Create your account' },
  { id: 2, name: 'Business', description: 'Business information' },
  { id: 3, name: 'Store', description: 'Store setup' },
  { id: 4, name: 'Documents', description: 'Upload documents' },
  { id: 5, name: 'Review', description: 'Review and submit' },
];

const formSchema = z.object({
  // Step 1
  email: z.string().email(),
  password: z.string().min(8),
  confirmPassword: z.string(),
  name: z.string().min(2),
  
  // Step 2
  businessName: z.string().min(2),
  businessEmail: z.string().email(),
  businessPhone: z.string().optional(),
  businessAddress: z.object({
    street: z.string(),
    city: z.string(),
    state: z.string(),
    postalCode: z.string(),
    country: z.string(),
  }),
  taxId: z.string().optional(),
  businessType: z.enum(['individual', 'company', 'llc']),
  
  // Step 3
  storeName: z.string().min(2),
  storeSlug: z.string().min(2).regex(/^[a-z0-9-]+$/),
  storeDescription: z.string().optional(),
  categories: z.array(z.string()).min(1),
  
  // Step 4
  documents: z.array(z.object({
    type: z.string(),
    file: z.instanceof(File),
    previewUrl: z.string(),
  })),
  
  // Step 5
  acceptTerms: z.boolean(),
  acceptPrivacyPolicy: z.boolean(),
}).refine(data => data.password === data.confirmPassword, {
  message: "Passwords don't match",
  path: ["confirmPassword"],
});

type FormData = z.infer<typeof formSchema>;

export default function VendorOnboardingWizard() {
  const [currentStep, setCurrentStep] = useState(1);
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  const { register, handleSubmit, watch, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      categories: [],
      documents: [],
      acceptTerms: false,
      acceptPrivacyPolicy: false,
    },
  });

  const onSubmit = async (data: FormData) => {
    setIsSubmitting(true);
    try {
      // Upload documents first
      const documentUrls = await Promise.all(
        data.documents.map(async (doc) => {
          const formData = new FormData();
          formData.append('file', doc.file);
          const response = await fetch('/api/upload', {
            method: 'POST',
            body: formData,
          });
          const result = await response.json();
          return {
            type: doc.type,
            url: result.url,
          };
        })
      );

      // Submit vendor application
      const response = await fetch('/api/vendor/onboarding', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          ...data,
          documents: documentUrls,
        }),
      });

      if (response.ok) {
        // Redirect to confirmation page
        window.location.href = '/vendor/onboarding/success';
      } else {
        throw new Error('Submission failed');
      }
    } catch (error) {
      console.error('Submission error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <Stepper steps={steps} currentStep={currentStep} />
      
      <form onSubmit={handleSubmit(onSubmit)} className="mt-8">
        {currentStep === 1 && (
          <div className="space-y-4">
            <h2 className="text-2xl font-bold">Account Information</h2>
            {/* Account form fields */}
          </div>
        )}
        
        {currentStep === 2 && (
          <div className="space-y-4">
            <h2 className="text-2xl font-bold">Business Information</h2>
            {/* Business form fields */}
          </div>
        )}
        
        {/* Additional steps */}
        
        <div className="mt-8 flex justify-between">
          <button
            type="button"
            onClick={() => setCurrentStep(prev => prev - 1)}
            disabled={currentStep === 1}
            className="btn btn-secondary"
          >
            Previous
          </button>
          
          {currentStep < steps.length ? (
            <button
              type="button"
              onClick={() => setCurrentStep(prev => prev + 1)}
              className="btn btn-primary"
            >
              Next
            </button>
          ) : (
            <button
              type="submit"
              disabled={isSubmitting}
              className="btn btn-primary"
            >
              {isSubmitting ? 'Submitting...' : 'Submit Application'}
            </button>
          )}
        </div>
      </form>
    </div>
  );
}
§ VENDOR DASHBOARD
typescript
// app/vendor/dashboard/page.tsx
import { getCurrentVendor } from '@/lib/auth';
import { prisma } from '@/lib/prisma';
import DashboardLayout from '@/components/vendor/DashboardLayout';
import StatsCards from '@/components/vendor/dashboard/StatsCards';
import RecentOrders from '@/components/vendor/dashboard/RecentOrders';
import PerformanceChart from '@/components/vendor/dashboard/PerformanceChart';
import PendingTasks from '@/components/vendor/dashboard/PendingTasks';

export default async function VendorDashboard() {
  const vendor = await getCurrentVendor();
  
  if (!vendor) {
    redirect('/vendor/login');
  }

  // Fetch dashboard data
  const [
    stats,
    recentOrders,
    performanceData,
    pendingProducts
  ] = await Promise.all([
    // Statistics
    prisma.$queryRaw`
      SELECT 
        COUNT(*) as total_orders,
        SUM(CASE WHEN status = 'FULFILLED' THEN 1 ELSE 0 END) as completed_orders,
        SUM(total) as total_revenue,
        AVG(vendor_earnings) as avg_order_value
      FROM "VendorOrder" 
      WHERE vendor_id = ${vendor.id}
      AND created_at >= CURRENT_DATE - INTERVAL '30 days'
    `,
    
    // Recent orders
    prisma.vendorOrder.findMany({
      where: { vendorId: vendor.id },
      include: {
        order: {
          include: { customer: true }
        },
        items: true
      },
      orderBy: { createdAt: 'desc' },
      take: 10
    }),
    
    // Performance data (last 30 days)
    prisma.$queryRaw`
      SELECT 
        DATE(created_at) as date,
        COUNT(*) as order_count,
        SUM(vendor_earnings) as revenue
      FROM "VendorOrder" 
      WHERE vendor_id = ${vendor.id}
      AND created_at >= CURRENT_DATE - INTERVAL '30 days'
      GROUP BY DATE(created_at)
      ORDER BY date
    `,
    
    // Pending products
    prisma.product.findMany({
      where: {
        vendorId: vendor.id,
        status: { in: ['DRAFT', 'PENDING_REVIEW'] }
      },
      take: 5
    })
  ]);

  return (
    <DashboardLayout vendor={vendor}>
      <div className="space-y-6">
        <div>
          <h1 className="text-3xl font-bold">Dashboard</h1>
          <p className="text-gray-600">
            Welcome back, {vendor.businessName}
          </p>
        </div>
        
        <StatsCards stats={stats} />
        
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          <div className="lg:col-span-2">
            <PerformanceChart data={performanceData} />
          </div>
          <div>
            <PendingTasks 
              pendingProducts={pendingProducts}
              vendor={vendor}
            />
          </div>
        </div>
        
        <RecentOrders orders={recentOrders} />
      </div>
    </DashboardLayout>
  );
}
typescript
// components/vendor/dashboard/StatsCards.tsx
interface StatsCardsProps {
  stats: {
    total_orders: number;
    completed_orders: number;
    total_revenue: number;
    avg_order_value: number;
  };
}

export default function StatsCards({ stats }: StatsCardsProps) {
  const cards = [
    {
      title: 'Total Orders',
      value: stats.total_orders,
      change: '+12%',
      icon: '📦',
      color: 'blue',
    },
    {
      title: 'Revenue',
      value: `$${stats.total_revenue.toFixed(2)}`,
      change: '+8%',
      icon: '💰',
      color: 'green',
    },
    {
      title: 'Avg Order Value',
      value: `$${stats.avg_order_value.toFixed(2)}`,
      change: '+5%',
      icon: '📊',
      color: 'purple',
    },
    {
      title: 'Conversion Rate',
      value: '3.2%',
      change: '+0.5%',
      icon: '📈',
      color: 'orange',
    },
  ];

  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
      {cards.map((card) => (
        <div
          key={card.title}
          className="bg-white rounded-lg shadow p-6"
        >
          <div className="flex items-center justify-between">
            <div>
              <p className="text-sm text-gray-600">{card.title}</p>
              <p className="text-2xl font-bold mt-2">{card.value}</p>
              <p className={`text-sm mt-1 ${
                card.change.startsWith('+') 
                  ? 'text-green-600' 
                  : 'text-red-600'
              }`}>
                {card.change} from last month
              </p>
            </div>
            <div className="text-3xl">{card.icon}</div>
          </div>
        </div>
      ))}
    </div>
  );
}
typescript
// components/vendor/products/ProductManager.tsx
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';
import { Product, ProductVariant } from '@prisma/client';

interface ProductManagerProps {
  initialProducts: (Product & { variants: ProductVariant[] })[];
  vendorId: string;
}

export default function ProductManager({ 
  initialProducts, 
  vendorId 
}: ProductManagerProps) {
  const [products, setProducts] = useState(initialProducts);
  const [isLoading, setIsLoading] = useState(false);
  const router = useRouter();

  const handleDelete = async (productId: string) => {
    if (!confirm('Are you sure you want to delete this product?')) return;
    
    setIsLoading(true);
    try {
      const response = await fetch(`/api/vendor/products/${productId}`, {
        method: 'DELETE',
      });
      
      if (response.ok) {
        setProducts(products.filter(p => p.id !== productId));
        router.refresh();
      }
    } catch (error) {
      console.error('Delete failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  const handleStatusChange = async (productId: string, status: string) => {
    setIsLoading(true);
    try {
      const response = await fetch(`/api/vendor/products/${productId}/status`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ status }),
      });
      
      if (response.ok) {
        setProducts(products.map(p => 
          p.id === productId ? { ...p, status } : p
        ));
        router.refresh();
      }
    } catch (error) {
      console.error('Status update failed:', error);
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <div className="bg-white rounded-lg shadow">
      <div className="p-6 border-b">
        <div className="flex justify-between items-center">
          <h2 className="text-xl font-bold">Products</h2>
          <a
            href="/vendor/products/new"
            className="btn btn-primary"
          >
            Add Product
          </a>
        </div>
      </div>
      
      <div className="overflow-x-auto">
        <table className="min-w-full divide-y divide-gray-200">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Product
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Status
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Price
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Stock
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                Actions
              </th>
            </tr>
          </thead>
          <tbody className="divide-y divide-gray-200">
            {products.map((product) => (
              <tr key={product.id}>
                <td className="px-6 py-4">
                  <div className="flex items-center">
                    {product.featuredImage && (
                      <img
                        src={product.featuredImage}
                        alt={product.name}
                        className="h-10 w-10 rounded object-cover mr-3"
                      />
                    )}
                    <div>
                      <p className="font-medium">{product.name}</p>
                      <p className="

## § ADVANCED PATTERNS: MARKETPLACE MULTIVENDOR

### Server Actions con Validazione

```typescript
// app/actions/marketplace-multivendor.ts
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


### MARKETPLACE MULTIVENDOR - Utility Helper #169

```typescript
// lib/utils/marketplace-multivendor-helper-169.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #465

```typescript
// lib/utils/marketplace-multivendor-helper-465.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #494

```typescript
// lib/utils/marketplace-multivendor-helper-494.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #501

```typescript
// lib/utils/marketplace-multivendor-helper-501.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #674

```typescript
// lib/utils/marketplace-multivendor-helper-674.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #638

```typescript
// lib/utils/marketplace-multivendor-helper-638.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #702

```typescript
// lib/utils/marketplace-multivendor-helper-702.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #204

```typescript
// lib/utils/marketplace-multivendor-helper-204.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #699

```typescript
// lib/utils/marketplace-multivendor-helper-699.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #665

```typescript
// lib/utils/marketplace-multivendor-helper-665.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #132

```typescript
// lib/utils/marketplace-multivendor-helper-132.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #979

```typescript
// lib/utils/marketplace-multivendor-helper-979.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #557

```typescript
// lib/utils/marketplace-multivendor-helper-557.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #460

```typescript
// lib/utils/marketplace-multivendor-helper-460.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #246

```typescript
// lib/utils/marketplace-multivendor-helper-246.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(): Readonly<MARKETPLACEMULTIVENDORConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### MARKETPLACE MULTIVENDOR - Utility Helper #943

```typescript
// lib/utils/marketplace-multivendor-helper-943.ts
import { z } from "zod";

interface MARKETPLACEMULTIVENDORConfig {
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

export class MARKETPLACEMULTIVENDORProcessor<TInput, TOutput> {
  private config: MARKETPLACEMULTIVENDORConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<MARKETPLACEMULTIVENDORConfig> = {}) {
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

  getConfig(