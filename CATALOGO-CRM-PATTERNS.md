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

## § ADVANCED PATTERNS: CRM PATTERNS

### Server Actions con Validazione

```typescript
// app/actions/crm-patterns.ts
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


### CRM PATTERNS - Utility Helper #926

```typescript
// lib/utils/crm-patterns-helper-926.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #146

```typescript
// lib/utils/crm-patterns-helper-146.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #733

```typescript
// lib/utils/crm-patterns-helper-733.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #559

```typescript
// lib/utils/crm-patterns-helper-559.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #406

```typescript
// lib/utils/crm-patterns-helper-406.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #845

```typescript
// lib/utils/crm-patterns-helper-845.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #686

```typescript
// lib/utils/crm-patterns-helper-686.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #879

```typescript
// lib/utils/crm-patterns-helper-879.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #33

```typescript
// lib/utils/crm-patterns-helper-33.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #116

```typescript
// lib/utils/crm-patterns-helper-116.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #889

```typescript
// lib/utils/crm-patterns-helper-889.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #854

```typescript
// lib/utils/crm-patterns-helper-854.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #514

```typescript
// lib/utils/crm-patterns-helper-514.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #297

```typescript
// lib/utils/crm-patterns-helper-297.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #234

```typescript
// lib/utils/crm-patterns-helper-234.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #420

```typescript
// lib/utils/crm-patterns-helper-420.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CRM PATTERNS - Utility Helper #895

```typescript
// lib/utils/crm-patterns-helper-895.ts
import { z } from "zod";

interface CRMPATTERNSConfig {
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

export class CRMPATTERNSProcessor<TInput, TOutput> {
  private config: CRMPATTERNSConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CRMPATTERNSConfig> = {}) {
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

  getConfig(): Readonly<CRMPATTERNSConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
