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