# CATALOGO-CRM-PATTERNS

CRM CATALOGO PATTERNS - Next.js 14 + Prisma
§ CRM DATA MODEL
Prisma Schema
prisma
// schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  ADMIN
  MANAGER
  SALES
  VIEWER
}

enum ActivityType {
  CALL
  EMAIL
  MEETING
  TASK
  NOTE
}

enum DealStage {
  QUALIFICATION
  NEEDS_ANALYSIS
  PROPOSAL
  NEGOTIATION
  CLOSED_WON
  CLOSED_LOST
}

enum FieldType {
  TEXT
  NUMBER
  DATE
  SELECT
  MULTI_SELECT
  BOOLEAN
  URL
  PHONE
  EMAIL
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  role          UserRole  @default(SALES)
  avatarUrl     String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  
  // Relationships
  ownedContacts Contact[] @relation("ContactOwner")
  ownedCompanies Company[] @relation("CompanyOwner")
  ownedDeals     Deal[]     @relation("DealOwner")
  activities    Activity[]
  notes         Note[]
  
  team          Team?     @relation(fields: [teamId], references: [id])
  teamId        String?
  
  @@index([email])
  @@index([teamId])
}

model Team {
  id          String   @id @default(cuid())
  name        String
  description String?
  createdAt   DateTime @default(now())
  
  // Relationships
  users       User[]
  companies   Company[]
  deals       Deal[]
  
  @@index([name])
}

model Contact {
  id          String   @id @default(cuid())
  firstName   String
  lastName    String
  email       String?  @unique
  phone       String?
  mobile      String?
  title       String?
  department  String?
  avatarUrl   String?
  source      String?
  status      String?  @default("lead")
  timezone    String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  lastActivityAt DateTime?
  
  // Custom fields
  customFields Json?
  
  // Relationships
  company      Company? @relation(fields: [companyId], references: [id])
  companyId    String?
  
  owner        User?    @relation("ContactOwner", fields: [ownerId], references: [id])
  ownerId      String?
  
  deals        Deal[]
  activities   Activity[]
  notes        Note[]
  
  // Tags and segments
  tags         Tag[]
  segments     Segment[]
  
  // Address
  address      Address?
  
  @@index([email])
  @@index([companyId])
  @@index([ownerId])
  @@index([lastActivityAt])
  @@index([status])
  @@fulltext([firstName, lastName, email, phone])
}

model Company {
  id           String   @id @default(cuid())
  name         String
  description  String?
  website      String?
  industry     String?
  employeeCount Int?
  annualRevenue Decimal?
  foundedYear  Int?
  logoUrl      String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  
  // Custom fields
  customFields Json?
  
  // Hierarchy
  parent       Company? @relation("CompanyHierarchy", fields: [parentId], references: [id])
  parentId     String?
  subsidiaries Company[] @relation("CompanyHierarchy")
  
  // Relationships
  contacts     Contact[]
  deals        Deal[]
  activities   Activity[]
  notes        Note[]
  
  owner        User?    @relation("CompanyOwner", fields: [ownerId], references: [id])
  ownerId      String?
  
  team         Team?    @relation(fields: [teamId], references: [id])
  teamId       String?
  
  // Tags
  tags         Tag[]
  
  // Address
  address      Address?
  
  @@index([name])
  @@index([industry])
  @@index([parentId])
  @@index([ownerId])
  @@index([teamId])
  @@fulltext([name, description, website])
}

model Deal {
  id              String   @id @default(cuid())
  name            String
  description     String?
  value           Decimal  @default(0)
  probability     Int      @default(0) // 0-100
  stage           DealStage @default(QUALIFICATION)
  expectedClose   DateTime?
  actualClose     DateTime?
  closedReason    String?
  isWon           Boolean  @default(false)
  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt
  
  // Custom fields
  customFields Json?
  
  // Relationships
  company         Company? @relation(fields: [companyId], references: [id])
  companyId       String?
  
  contact         Contact? @relation(fields: [contactId], references: [id])
  contactId       String?
  
  owner           User?    @relation("DealOwner", fields: [ownerId], references: [id])
  ownerId         String?
  
  team            Team?    @relation(fields: [teamId], references: [id])
  teamId          String?
  
  activities      Activity[]
  notes           Note[]
  
  // Tags
  tags            Tag[]
  
  // Pipeline
  pipeline        Pipeline? @relation(fields: [pipelineId], references: [id])
  pipelineId      String?
  stageOrder      Int      @default(0)
  
  @@index([companyId])
  @@index([contactId])
  @@index([ownerId])
  @@index([teamId])
  @@index([stage])
  @@index([expectedClose])
  @@index([pipelineId, stageOrder])
}

model Pipeline {
  id           String   @id @default(cuid())
  name         String
  description  String?
  isDefault    Boolean  @default(false)
  stages       Json     // Array of stage definitions
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  
  // Relationships
  deals        Deal[]
  team         Team?    @relation(fields: [teamId], references: [id])
  teamId       String?
  
  @@index([teamId])
  @@index([isDefault])
}

model Activity {
  id           String       @id @default(cuid())
  type         ActivityType
  subject      String
  description  String?
  dueDate      DateTime?
  completedAt  DateTime?
  duration     Int?         // in minutes
  priority     String?      @default("medium")
  createdAt    DateTime     @default(now())
  updatedAt    DateTime     @updatedAt
  
  // Email specific
  emailId      String?
  emailStatus  String?      // sent, delivered, opened, clicked
  
  // Meeting specific
  meetingStart DateTime?
  meetingEnd   DateTime?
  meetingUrl   String?
  
  // Call specific
  callOutcome  String?
  recordingUrl String?
  
  // Relationships
  contact      Contact?     @relation(fields: [contactId], references: [id])
  contactId    String?
  
  company      Company?     @relation(fields: [companyId], references: [id])
  companyId    String?
  
  deal         Deal?        @relation(fields: [dealId], references: [id])
  dealId       String?
  
  owner        User         @relation(fields: [ownerId], references: [id])
  ownerId      String
  
  participants Contact[]    @relation("ActivityParticipants")
  
  @@index([contactId])
  @@index([companyId])
  @@index([dealId])
  @@index([ownerId])
  @@index([dueDate])
  @@index([type])
}

model Note {
  id           String   @id @default(cuid())
  content      String
  isPrivate    Boolean  @default(false)
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
  
  // Relationships
  contact      Contact? @relation(fields: [contactId], references: [id])
  contactId    String?
  
  company      Company? @relation(fields: [companyId], references: [id])
  companyId    String?
  
  deal         Deal?    @relation(fields: [dealId], references: [id])
  dealId       String?
  
  author       User     @relation(fields: [authorId], references: [id])
  authorId     String
  
  @@index([contactId])
  @@index([companyId])
  @@index([dealId])
  @@index([authorId])
}

model Address {
  id          String   @id @default(cuid())
  street      String?
  city        String?
  state       String?
  postalCode  String?
  country     String?
  
  // Relationships
  contact     Contact? @relation(fields: [contactId], references: [id])
  contactId   String?  @unique
  
  company     Company? @relation(fields: [companyId], references: [id])
  companyId   String?  @unique
  
  @@index([contactId])
  @@index([companyId])
}

model Tag {
  id          String   @id @default(cuid())
  name        String
  color       String   @default("#3B82F6")
  type        String   @default("custom") // system, custom
  
  // Relationships
  contacts    Contact[]
  companies   Company[]
  deals       Deal[]
  
  @@unique([name, type])
  @@index([name])
}

model Segment {
  id          String   @id @default(cuid())
  name        String
  description String?
  rules       Json     // Segment rules definition
  isDynamic   Boolean  @default(true)
  memberCount Int      @default(0)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Relationships
  contacts    Contact[]
  
  @@index([name])
  @@index([isDynamic])
}

model CustomField {
  id          String    @id @default(cuid())
  name        String
  type        FieldType
  entityType  String    // contact, company, deal
  options     String[]? // For SELECT/MULTI_SELECT types
  isRequired  Boolean   @default(false)
  order       Int       @default(0)
  teamId      String?
  
  @@unique([name, entityType, teamId])
  @@index([entityType])
  @@index([teamId])
}

model EmailTemplate {
  id          String   @id @default(cuid())
  name        String
  subject     String
  body        String
  variables   String[] // Available template variables
  isDefault   Boolean  @default(false)
  teamId      String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([teamId])
  @@index([isDefault])
}

model Workflow {
  id          String   @id @default(cuid())
  name        String
  description String?
  triggerType String   // contact.created, deal.stage_changed, etc.
  triggerConfig Json   // Trigger configuration
  actions     Json     // Array of actions
  isActive    Boolean  @default(true)
  teamId      String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([teamId])
  @@index([triggerType])
  @@index([isActive])
}
TypeScript Types Support
typescript
// types/crm.ts
export type CustomFieldValue = string | number | boolean | Date | string[] | null;

export interface CustomFields {
  [fieldName: string]: CustomFieldValue;
}

export interface PipelineStage {
  id: string;
  name: string;
  color: string;
  probability: number;
  order: number;
}

export interface SegmentRule {
  field: string;
  operator: 'equals' | 'not_equals' | 'contains' | 'greater_than' | 'less_than';
  value: string | number | boolean | Date;
}

export interface ActivityCreateInput {
  type: ActivityType;
  subject: string;
  description?: string;
  dueDate?: Date;
  contactId?: string;
  companyId?: string;
  dealId?: string;
  participants?: string[];
  metadata?: Record<string, any>;
}
§ CONTACT MANAGEMENT
Contact CRUD Operations
typescript
// app/api/contacts/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { z } from 'zod';

const prisma = new PrismaClient();

const contactSchema = z.object({
  firstName: z.string().min(1),
  lastName: z.string().min(1),
  email: z.string().email().optional().or(z.literal('')),
  phone: z.string().optional(),
  companyId: z.string().optional(),
  ownerId: z.string().optional(),
  customFields: z.record(z.any()).optional(),
  tags: z.array(z.string()).optional(),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const data = contactSchema.parse(body);

    // Check for duplicates
    if (data.email) {
      const existing = await prisma.contact.findFirst({
        where: {
          email: data.email,
          companyId: data.companyId,
        },
      });
      
      if (existing) {
        return NextResponse.json(
          { error: 'Duplicate contact found', contact: existing },
          { status: 409 }
        );
      }
    }

    const contact = await prisma.contact.create({
      data: {
        ...data,
        email: data.email || null,
        tags: data.tags ? {
          connectOrCreate: data.tags.map(tag => ({
            where: { name: tag },
            create: { name: tag },
          })),
        } : undefined,
      },
      include: {
        company: true,
        tags: true,
        owner: true,
      },
    });

    return NextResponse.json(contact);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create contact' },
      { status: 500 }
    );
  }
}

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '50');
  const search = searchParams.get('search');
  const companyId = searchParams.get('companyId');
  const ownerId = searchParams.get('ownerId');
  const tags = searchParams.getAll('tag');

  const where: any = {};

  if (search) {
    where.OR = [
      { firstName: { contains: search, mode: 'insensitive' } },
      { lastName: { contains: search, mode: 'insensitive' } },
      { email: { contains: search, mode: 'insensitive' } },
      { phone: { contains: search } },
    ];
  }

  if (companyId) where.companyId = companyId;
  if (ownerId) where.ownerId = ownerId;
  if (tags.length > 0) {
    where.tags = {
      some: {
        name: { in: tags },
      },
    };
  }

  const [contacts, total] = await Promise.all([
    prisma.contact.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      include: {
        company: true,
        owner: true,
        tags: true,
        _count: {
          select: { activities: true, deals: true },
        },
      },
      orderBy: {
        lastActivityAt: 'desc',
      },
    }),
    prisma.contact.count({ where }),
  ]);

  return NextResponse.json({
    contacts,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  });
}
Contact Search & Filter
typescript
// app/api/contacts/search/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const query = searchParams.get('q') || '';
  const limit = parseInt(searchParams.get('limit') || '10');

  if (!query) {
    return NextResponse.json([]);
  }

  const contacts = await prisma.contact.findMany({
    where: {
      OR: [
        { firstName: { contains: query, mode: 'insensitive' } },
        { lastName: { contains: query, mode: 'insensitive' } },
        { email: { contains: query, mode: 'insensitive' } },
        { phone: { contains: query } },
        { company: { name: { contains: query, mode: 'insensitive' } } },
      ],
    },
    take: limit,
    include: {
      company: true,
      tags: true,
    },
    orderBy: {
      lastActivityAt: 'desc',
    },
  });

  return NextResponse.json(contacts);
}

// app/api/contacts/filters/route.ts
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const filters = JSON.parse(searchParams.get('filters') || '{}');

  const where: any = {};

  if (filters.status) where.status = filters.status;
  if (filters.companyId) where.companyId = filters.companyId;
  if (filters.ownerId) where.ownerId = filters.ownerId;
  
  if (filters.tags && filters.tags.length > 0) {
    where.tags = {
      some: {
        name: { in: filters.tags },
      },
    };
  }

  if (filters.createdAfter) {
    where.createdAt = {
      gte: new Date(filters.createdAfter),
    };
  }

  if (filters.lastActivityAfter) {
    where.lastActivityAt = {
      gte: new Date(filters.lastActivityAfter),
    };
  }

  const contacts = await prisma.contact.findMany({
    where,
    include: {
      company: true,
      owner: true,
      tags: true,
    },
  });

  return NextResponse.json(contacts);
}
Contact Import (CSV)
typescript
// app/api/contacts/import/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { parse } from 'csv-parse/sync';

const prisma = new PrismaClient();

export async function POST(request: NextRequest) {
  try {
    const formData = await request.formData();
    const file = formData.get('file') as File;
    
    if (!file) {
      return NextResponse.json(
        { error: 'No file provided' },
        { status: 400 }
      );
    }

    const text = await file.text();
    const records = parse(text, {
      columns: true,
      skip_empty_lines: true,
    });

    const results = {
      success: 0,
      failed: 0,
      duplicates: 0,
      errors: [] as string[],
    };

    for (const [index, record] of records.entries()) {
      try {
        // Check for required fields
        if (!record.firstName || !record.lastName) {
          throw new Error('Missing required fields');
        }

        // Check for duplicate
        if (record.email) {
          const existing = await prisma.contact.findFirst({
            where: { email: record.email },
          });
          
          if (existing) {
            results.duplicates++;
            continue;
          }
        }

        // Create or find company
        let companyId = null;
        if (record.company) {
          const company = await prisma.company.upsert({
            where: { name: record.company },
            update: {},
            create: { name: record.company },
          });
          companyId = company.id;
        }

        // Create contact
        await prisma.contact.create({
          data: {
            firstName: record.firstName,
            lastName: record.lastName,
            email: record.email || null,
            phone: record.phone || null,
            title: record.title || null,
            department: record.department || null,
            companyId,
            customFields: record.customFields ? 
              JSON.parse(record.customFields) : undefined,
          },
        });

        results.success++;
      } catch (error) {
        results.failed++;
        results.errors.push(`Row ${index + 1}: ${error instanceof Error ? error.message : 'Unknown error'}`);
      }
    }

    return NextResponse.json(results);
  } catch (error) {
    return NextResponse.json(
      { error: 'Import failed' },
      { status: 500 }
    );
  }
}
Duplicate Detection & Merge
typescript
// app/api/contacts/duplicates/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(request: NextRequest) {
  // Find potential duplicates based on email, phone, or name
  const contacts = await prisma.contact.findMany({
    where: {
      OR: [
        { email: { not: null } },
        { phone: { not: null } },
      ],
    },
    orderBy: { createdAt: 'asc' },
  });

  const duplicates: Array<{
    field: string;
    value: string;
    contacts: any[];
  }> = [];

  // Group by email
  const emailGroups = new Map<string, any[]>();
  contacts.forEach(contact => {
    if (contact.email) {
      const group = emailGroups.get(contact.email) || [];
      group.push(contact);
      emailGroups.set(contact.email, group);
    }
  });

  // Group by phone
  const phoneGroups = new Map<string, any[]>();
  contacts.forEach(contact => {
    if (contact.phone) {
      const group = phoneGroups.get(contact.phone) || [];
      group.push(contact);
      phoneGroups.set(contact.phone, group);
    }
  });

  // Add groups with multiple contacts to duplicates
  emailGroups.forEach((group, email) => {
    if (group.length > 1) {
      duplicates.push({
        field: 'email',
        value: email,
        contacts: group,
      });
    }
  });

  phoneGroups.forEach((group, phone) => {
    if (group.length > 1) {
      duplicates.push({
        field: 'phone',
        value: phone,
        contacts: group,
      });
    }
  });

  return NextResponse.json({ duplicates });
}

// app/api/contacts/merge/route.ts
export async function POST(request: NextRequest) {
  try {
    const { primaryId, mergeIds } = await request.json();

    // Get all contacts to merge
    const [primary, ...toMerge] = await Promise.all([
      prisma.contact.findUnique({
        where: { id: primaryId },
        include: {
          activities: true,
          deals: true,
          notes: true,
          tags: true,
        },
      }),
      ...mergeIds.map((id: string) => 
        prisma.contact.findUnique({
          where: { id },
          include: {
            activities: true,
            deals: true,
            notes: true,
            tags: true,
          },
        })
      ),
    ]);

    if (!primary) {
      throw new Error('Primary contact not found');
    }

    // Merge data (keep primary's data unless empty)
    const mergedData = {
      firstName: primary.firstName,
      lastName: primary.lastName,
      email: primary.email || toMerge.find(c => c?.email)?.email || null,
      phone: primary.phone || toMerge.find(c => c?.phone)?.phone || null,
      mobile: primary.mobile || toMerge.find(c => c?.mobile)?.mobile || null,
      title: primary.title || toMerge.find(c => c?.title)?.title || null,
      department: primary.department || toMerge.find(c => c?.department)?.department || null,
      companyId: primary.companyId || toMerge.find(c => c?.companyId)?.companyId || null,
      ownerId: primary.ownerId || toMerge.find(c => c?.ownerId)?.ownerId || null,
    };

    // Collect all related data
    const allActivities = [
      ...primary.activities,
      ...toMerge.flatMap(c => c?.activities || []),
    ];
    
    const allDeals = [
      ...primary.deals,
      ...toMerge.flatMap(c => c?.deals || []),
    ];
    
    const allNotes = [
      ...primary.notes,
      ...toMerge.flatMap(c => c?.notes || []),
    ];
    
    const allTags = [
      ...primary.tags,
      ...toMerge.flatMap(c => c?.tags || []),
    ];

    // Use transaction to ensure data consistency
    const result = await prisma.$transaction(async (tx) => {
      // Update primary contact
      const updatedContact = await tx.contact.update({
        where: { id: primaryId },
        data: {
          ...mergedData,
          // Update relationships
          activities: {
            connect: allActivities.map(a => ({ id: a.id })),
          },
          deals: {
            connect: allDeals.map(d => ({ id: d.id })),
          },
          notes: {
            connect: allNotes.map(n => ({ id: n.id })),
          },
          tags: {
            connect: allTags.map(t => ({ id: t.id })),
          },
        },
      });

      // Delete merged contacts
      await tx.contact.deleteMany({
        where: {
          id: { in: mergeIds },
        },
      });

      return updatedContact;
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Merge failed' },
      { status: 500 }
    );
  }
}
Contact Timeline
typescript
// app/api/contacts/[id]/timeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;
  
  const timeline = await prisma.activity.findMany({
    where: {
      OR: [
        { contactId: id },
        { company: { contacts: { some: { id } } } },
        { deal: { contactId: id } },
      ],
    },
    include: {
      owner: true,
      company: true,
      deal: true,
    },
    orderBy: {
      createdAt: 'desc',
    },
  });

  // Also include notes
  const notes = await prisma.note.findMany({
    where: {
      contactId: id,
    },
    include: {
      author: true,
    },
    orderBy: {
      createdAt: 'desc',
    },
  });

  // Combine and sort all timeline items
  const timelineItems = [
    ...timeline.map(item => ({
      type: 'activity' as const,
      id: item.id,
      date: item.createdAt,
      data: item,
    })),
    ...notes.map(note => ({
      type: 'note' as const,
      id: note.id,
      date: note.createdAt,
      data: note,
    })),
  ].sort((a, b) => b.date.getTime() - a.date.getTime());

  return NextResponse.json(timelineItems);
}
§ COMPANY MANAGEMENT
Company CRUD Operations
typescript
// app/api/companies/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { z } from 'zod';

const prisma = new PrismaClient();

const companySchema = z.object({
  name: z.string().min(1),
  description: z.string().optional(),
  website: z.string().url().optional().or(z.literal('')),
  industry: z.string().optional(),
  employeeCount: z.number().int().positive().optional(),
  annualRevenue: z.number().positive().optional(),
  parentId: z.string().optional(),
  ownerId: z.string().optional(),
  teamId: z.string().optional(),
  customFields: z.record(z.any()).optional(),
  tags: z.array(z.string()).optional(),
  address: z.object({
    street: z.string().optional(),
    city: z.string().optional(),
    state: z.string().optional(),
    postalCode: z.string().optional(),
    country: z.string().optional(),
  }).optional(),
});

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const data = companySchema.parse(body);

    const company = await prisma.company.create({
      data: {
        ...data,
        website: data.website || null,
        address: data.address ? {
          create: data.address,
        } : undefined,
        tags: data.tags ? {
          connectOrCreate: data.tags.map(tag => ({
            where: { name: tag },
            create: { name: tag },
          })),
        } : undefined,
      },
      include: {
        parent: true,
        subsidiaries: true,
        contacts: true,
        deals: true,
        tags: true,
        address: true,
      },
    });

    return NextResponse.json(company);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create company' },
      { status: 500 }
    );
  }
}

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '50');
  const search = searchParams.get('search');
  const industry = searchParams.get('industry');
  const ownerId = searchParams.get('ownerId');

  const where: any = {};

  if (search) {
    where.OR = [
      { name: { contains: search, mode: 'insensitive' } },
      { description: { contains: search, mode: 'insensitive' } },
      { website: { contains: search, mode: 'insensitive' } },
    ];
  }

  if (industry) where.industry = industry;
  if (ownerId) where.ownerId = ownerId;

  const [companies, total] = await Promise.all([
    prisma.company.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      include: {
        parent: true,
        contacts: { take: 5 },
        deals: { take: 5 },
        tags: true,
        address: true,
        _count: {
          select: {
            contacts: true,
            deals: true,
            subsidiaries: true,
          },
        },
      },
      orderBy: {
        updatedAt: 'desc',
      },
    }),
    prisma.company.count({ where }),
  ]);

  return NextResponse.json({
    companies,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  });
}
Company Hierarchy Management
typescript
// app/api/companies/[id]/hierarchy/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;

  const company = await prisma.company.findUnique({
    where: { id },
    include: {
      parent: {
        include: {
          parent: true,
        },
      },
      subsidiaries: {
        include: {
          subsidiaries: true,
          _count: {
            select: {
              contacts: true,
              deals: true,
            },
          },
        },
      },
    },
  });

  if (!company) {
    return NextResponse.json(
      { error: 'Company not found' },
      { status: 404 }
    );
  }

  // Build hierarchy tree
  const buildTree = async (companyId: string, depth = 0) => {
    if (depth > 5) return null; // Prevent infinite recursion

    const company = await prisma.company.findUnique({
      where: { id: companyId },
      include: {
        subsidiaries: {
          include: {
            _count: {
              select: {
                contacts: true,
                deals: true,
              },
            },
          },
        },
      },
    });

    if (!company) return null;

    const children = await Promise.all(
      company.subsidiaries.map(subsidiary => 
        buildTree(subsidiary.id, depth + 1)
      )
    );

    return {
      ...company,
      subsidiaries: children.filter(Boolean),
    };
  };

  const hierarchy = await buildTree(id);

  return NextResponse.json(hierarchy);
}

// Update hierarchy
export async function PATCH(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;
  const { parentId, subsidiaries } = await request.json();

  const company = await prisma.company.update({
    where: { id },
    data: {
      parentId: parentId || null,
      subsidiaries: subsidiaries ? {
        set: subsidiaries.map((subId: string) => ({ id: subId })),
      } : undefined,
    },
    include: {
      parent: true,
      subsidiaries: true,
    },
  });

  return NextResponse.json(company);
}
Company Contacts & Deals
typescript
// app/api/companies/[id]/contacts/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;
  const { searchParams } = new URL(request.url);
  const page = parseInt(searchParams.get('page') || '1');
  const limit = parseInt(searchParams.get('limit') || '50');

  const [contacts, total] = await Promise.all([
    prisma.contact.findMany({
      where: { companyId: id },
      skip: (page - 1) * limit,
      take: limit,
      include: {
        owner: true,
        tags: true,
        _count: {
          select: { activities: true, deals: true },
        },
      },
      orderBy: {
        lastName: 'asc',
      },
    }),
    prisma.contact.count({ where: { companyId: id } }),
  ]);

  return NextResponse.json({
    contacts,
    pagination: {
      page,
      limit,
      total,
      pages: Math.ceil(total / limit),
    },
  });
}

// app/api/companies/[id]/deals/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const { id } = params;
  const { searchParams } = new URL(request.url);
  const stage = searchParams.get('stage');

  const where: any = { companyId: id };
  if (stage) where.stage = stage;

  const deals = await prisma.deal.findMany({
    where,
    include: {
      contact: true,
      owner: true,
      pipeline: true,
      tags: true,
      _count: {
        select: { activities: true },
      },
    },
    orderBy: {
      expectedClose: 'asc',
    },
  });

  // Calculate pipeline metrics
  const metrics = {
    totalValue: deals.reduce((sum, deal) => sum + Number(deal.value), 0),
    wonValue: deals
      .filter(deal => deal.isWon)
      .reduce((sum, deal) => sum + Number(deal.value), 0),
    activeDeals: deals.filter(deal => !deal.isWon && deal.stage !== 'CLOSED_LOST').length,
    wonDeals: deals.filter(deal => deal.isWon).length,
    lostDeals: deals.filter(deal => !deal.isWon && deal.stage === 'CLOSED_LOST').length,
  };

  return NextResponse.json({ deals, metrics });
}
§ DEAL/PIPELINE
Pipeline Stages Management
typescript
// app/api/pipelines/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { PrismaClient } from '@prisma/client';
import { z } from 'zod';

const prisma = new PrismaClient();

const pipelineSchema = z.object({
  name: z.string().min(1),
  description: z.string().optional(),
  stages: z.array(
    z.object({
      id: z.string(),
      name: z.string(),
      color: z.string(),
      probability: z.number().min(0).max(100),
      order: