# CATALOGO-SOCIAL-PROFILES

CATALOGO-SOCIAL-PROFILES per Next.js 14 + Prisma
Sezione Aggiunta: PROFILE COMPLETENESS
Modello Prisma
prisma
model ProfileCompleteness {
  id             String   @id @default(cuid())
  userId         String   @unique
  user           User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Score tracking
  overallScore   Int      @default(0) @min(0) @max(100)
  sectionScores  Json     // {basic: 50, contact: 75, ...}
  
  // Field completion tracking
  completedFields String[] // List of field names completed
  requiredFields  String[] // List of required fields
  optionalFields  String[] // List of optional fields
  
  // Gamification
  milestones     String[] // ["basic_complete", "contact_complete"]
  badges         String[] // ["profile_starter", "profile_expert"]
  lastUpdated    DateTime @updatedAt
  createdAt      DateTime @default(now())
}

enum ProfileMilestone {
  BASIC_INFO
  CONTACT_DETAILS
  SOCIAL_CONNECTIONS
  VERIFICATION
  MEDIA_UPLOAD
  COMPLETE_PROFILE
}

enum ProfileBadge {
  STARTER
  CONTRIBUTOR
  INFLUENCER
  VERIFIED_CHAMPION
  COMPLETIONIST
}
Componente Progress Indicator
tsx
// components/profile/ProfileCompleteness.tsx
'use client';

import { useState, useEffect } from 'react';
import { Progress } from '@/components/ui/progress';
import { Badge } from '@/components/ui/badge';
import { CheckCircle, Award, Target } from 'lucide-react';

interface ProfileCompletenessProps {
  userId: string;
  showDetails?: boolean;
}

export function ProfileCompleteness({ userId, showDetails = false }: ProfileCompletenessProps) {
  const [progress, setProgress] = useState(0);
  const [milestones, setMilestones] = useState<string[]>([]);
  const [badges, setBadges] = useState<string[]>([]);

  useEffect(() => {
    fetchProgress();
  }, [userId]);

  const fetchProgress = async () => {
    const res = await fetch(`/api/profile/${userId}/completeness`);
    const data = await res.json();
    setProgress(data.overallScore);
    setMilestones(data.milestones || []);
    setBadges(data.badges || []);
  };

  const sections = [
    { key: 'basic', label: 'Informazioni Base', weight: 30 },
    { key: 'contact', label: 'Contatti', weight: 20 },
    { key: 'social', label: 'Social', weight: 15 },
    { key: 'verification', label: 'Verifica', weight: 20 },
    { key: 'media', label: 'Media', weight: 15 },
  ];

  return (
    <div className="space-y-4 p-4 border rounded-lg bg-card">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-2">
          <Target className="h-5 w-5 text-primary" />
          <h3 className="font-semibold">Completamento Profilo</h3>
        </div>
        <span className="text-2xl font-bold">{progress}%</span>
      </div>

      <Progress value={progress} className="h-3" />

      {showDetails && (
        <>
          <div className="space-y-2 pt-4">
            <h4 className="text-sm font-medium">Sezioni</h4>
            {sections.map((section) => (
              <div key={section.key} className="flex items-center justify-between">
                <span className="text-sm">{section.label}</span>
                <span className="text-sm font-medium">
                  {Math.round(progress * section.weight / 100)}%
                </span>
              </div>
            ))}
          </div>

          {badges.length > 0 && (
            <div className="pt-4">
              <h4 className="text-sm font-medium mb-2 flex items-center gap-2">
                <Award className="h-4 w-4" />
                Badge
              </h4>
              <div className="flex flex-wrap gap-2">
                {badges.map((badge) => (
                  <Badge key={badge} variant="secondary">
                    {badge.replace('_', ' ')}
                  </Badge>
                ))}
              </div>
            </div>
          )}

          <div className="pt-4 text-sm text-muted-foreground">
            <div className="flex items-center gap-2">
              <CheckCircle className="h-4 w-4" />
              <span>Profilo completo sblocca funzionalità premium</span>
            </div>
          </div>
        </>
      )}
    </div>
  );
}
API Route
typescript
// app/api/profile/[userId]/completeness/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { calculateProfileScore } from '@/lib/profile-scoring';

export async function GET(
  request: NextRequest,
  { params }: { params: { userId: string } }
) {
  try {
    const userId = params.userId;
    
    // Recupera il profilo completo
    const profile = await prisma.user.findUnique({
      where: { id: userId },
      include: {
        profileCompleteness: true,
        socialProfiles: true,
        verifications: true,
      },
    });

    if (!profile) {
      return NextResponse.json({ error: 'Profilo non trovato' }, { status: 404 });
    }

    // Calcola score aggiornato
    const score = await calculateProfileScore(profile);
    
    // Aggiorna il record di completamento
    const completeness = await prisma.profileCompleteness.upsert({
      where: { userId },
      update: {
        overallScore: score.overall,
        sectionScores: score.sections,
        completedFields: score.completedFields,
      },
      create: {
        userId,
        overallScore: score.overall,
        sectionScores: score.sections,
        completedFields: score.completedFields,
        requiredFields: [
          'name',
          'email',
          'bio',
          'avatar',
          'location'
        ],
        optionalFields: [
          'phone',
          'website',
          'company',
          'socialLinks',
          'skills'
        ],
      },
    });

    // Controlla e assegna milestone
    const newMilestones = await checkMilestones(completeness, profile);
    const newBadges = await checkBadges(completeness);

    return NextResponse.json({
      ...completeness,
      milestones: newMilestones,
      badges: newBadges,
    });
  } catch (error) {
    console.error('Errore nel calcolo completamento:', error);
    return NextResponse.json(
      { error: 'Errore interno del server' },
      { status: 500 }
    );
  }
}

async function checkMilestones(completeness: any, profile: any) {
  const milestones = [];
  const scores = completeness.sectionScores as Record<string, number>;

  if (scores.basic >= 100) milestones.push('BASIC_INFO');
  if (scores.contact >= 100) milestones.push('CONTACT_DETAILS');
  if (profile.socialProfiles?.length >= 3) milestones.push('SOCIAL_CONNECTIONS');
  if (profile.verifications?.email && profile.verifications?.phone) {
    milestones.push('VERIFICATION');
  }
  if (completeness.overallScore >= 100) milestones.push('COMPLETE_PROFILE');

  return milestones;
}

async function checkBadges(completeness: any) {
  const badges = [];
  
  if (completeness.overallScore >= 25) badges.push('STARTER');
  if (completeness.overallScore >= 50) badges.push('CONTRIBUTOR');
  if (completeness.overallScore >= 75) badges.push('INFLUENCER');
  if (completeness.overallScore >= 90) badges.push('VERIFIED_CHAMPION');
  if (completeness.overallScore === 100) badges.push('COMPLETIONIST');

  return badges;
}
Utility Scoring
typescript
// lib/profile-scoring.ts
export async function calculateProfileScore(profile: any) {
  const sections: Record<string, number> = {
    basic: 0,
    contact: 0,
    social: 0,
    verification: 0,
    media: 0,
  };

  const completedFields: string[] = [];

  // Sezione Base (30%)
  if (profile.name) {
    sections.basic += 15;
    completedFields.push('name');
  }
  if (profile.bio && profile.bio.length > 50) {
    sections.basic += 10;
    completedFields.push('bio');
  }
  if (profile.avatar) {
    sections.basic += 5;
    completedFields.push('avatar');
  }

  // Sezione Contatti (20%)
  if (profile.email) {
    sections.contact += 10;
    completedFields.push('email');
  }
  if (profile.phone) {
    sections.contact += 5;
    completedFields.push('phone');
  }
  if (profile.location) {
    sections.contact += 5;
    completedFields.push('location');
  }

  // Sezione Social (15%)
  const socialCount = profile.socialProfiles?.length || 0;
  sections.social = Math.min(15, socialCount * 5);
  if (socialCount > 0) completedFields.push('socialLinks');

  // Sezione Verifica (20%)
  if (profile.verifications?.email) {
    sections.verification += 10;
    completedFields.push('email_verified');
  }
  if (profile.verifications?.phone) {
    sections.verification += 5;
    completedFields.push('phone_verified');
  }
  if (profile.verifications?.identity) {
    sections.verification += 5;
    completedFields.push('identity_verified');
  }

  // Sezione Media (15%)
  const mediaCount = profile.media?.length || 0;
  sections.media = Math.min(15, mediaCount * 3);
  if (mediaCount > 0) completedFields.push('media');

  // Calcola totale
  const overall = Object.values(sections).reduce((a, b) => a + b, 0);

  return {
    overall,
    sections,
    completedFields,
  };
}
Sezione Aggiunta: PROFILE VERIFICATION
Modello Prisma
prisma
model Verification {
  id             String   @id @default(cuid())
  userId         String   @unique
  user           User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Email verification
  email          String?
  emailVerified  Boolean  @default(false)
  emailToken     String?
  emailExpires   DateTime?
  
  // Phone verification
  phone          String?
  phoneVerified  Boolean  @default(false)
  phoneToken     String?
  phoneExpires   DateTime?
  
  // Identity verification (KYC light)
  identityStatus VerificationStatus @default(PENDING)
  identityMethod VerificationMethod?
  identityData   Json?   // Stores verification data securely
  verifiedAt     DateTime?
  
  // Social proof
  socialProof    Json?   // {twitter: true, github: true}
  
  // Badges
  badges         VerificationBadge[]
  
  createdAt      DateTime @default(now())
  updatedAt      DateTime @updatedAt
}

enum VerificationStatus {
  PENDING
  IN_REVIEW
  VERIFIED
  REJECTED
  EXPIRED
}

enum VerificationMethod {
  DOCUMENT_UPLOAD
  VIDEO_CALL
  BIOMETRIC
  MANUAL_REVIEW
}

enum VerificationBadge {
  EMAIL_VERIFIED
  PHONE_VERIFIED
  IDENTITY_VERIFIED
  SOCIAL_VERIFIED
  PREMIUM_VERIFIED
}
Componente Badge Verifica
tsx
// components/profile/VerificationBadges.tsx
'use client';

import { Badge } from '@/components/ui/badge';
import { 
  CheckCircle, 
  Shield, 
  Phone, 
  Mail, 
  UserCheck,
  Twitter,
  Github,
  Linkedin
} from 'lucide-react';
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from '@/components/ui/tooltip';

interface VerificationBadgesProps {
  verifications: any;
  size?: 'sm' | 'md' | 'lg';
}

export function VerificationBadges({ verifications, size = 'md' }: VerificationBadgesProps) {
  const sizeClasses = {
    sm: 'h-4 w-4',
    md: 'h-5 w-5',
    lg: 'h-6 w-6'
  };

  const getBadgeIcon = (type: string) => {
    switch (type) {
      case 'email':
        return <Mail className={sizeClasses[size]} />;
      case 'phone':
        return <Phone className={sizeClasses[size]} />;
      case 'identity':
        return <Shield className={sizeClasses[size]} />;
      case 'social':
        return <UserCheck className={sizeClasses[size]} />;
      default:
        return <CheckCircle className={sizeClasses[size]} />;
    }
  };

  const getBadgeColor = (type: string) => {
    switch (type) {
      case 'email':
        return 'bg-blue-100 text-blue-800 hover:bg-blue-100';
      case 'phone':
        return 'bg-green-100 text-green-800 hover:bg-green-100';
      case 'identity':
        return 'bg-purple-100 text-purple-800 hover:bg-purple-100';
      case 'social':
        return 'bg-orange-100 text-orange-800 hover:bg-orange-100';
      default:
        return 'bg-gray-100 text-gray-800 hover:bg-gray-100';
    }
  };

  const badges = [
    {
      type: 'email',
      label: 'Email verificata',
      verified: verifications?.emailVerified,
      date: verifications?.emailVerifiedAt,
    },
    {
      type: 'phone',
      label: 'Telefono verificato',
      verified: verifications?.phoneVerified,
      date: verifications?.phoneVerifiedAt,
    },
    {
      type: 'identity',
      label: 'Identità verificata',
      verified: verifications?.identityStatus === 'VERIFIED',
      date: verifications?.verifiedAt,
    },
    {
      type: 'social',
      label: 'Social verificati',
      verified: Object.values(verifications?.socialProof || {}).some(v => v),
      count: Object.values(verifications?.socialProof || {}).filter(v => v).length,
    },
  ];

  return (
    <TooltipProvider>
      <div className="flex flex-wrap gap-2">
        {badges
          .filter(badge => badge.verified)
          .map((badge) => (
            <Tooltip key={badge.type}>
              <TooltipTrigger asChild>
                <Badge className={`flex items-center gap-1 ${getBadgeColor(badge.type)}`}>
                  {getBadgeIcon(badge.type)}
                  <span className="hidden sm:inline">
                    {badge.type === 'social' ? `${badge.count} social` : badge.label.split(' ')[0]}
                  </span>
                </Badge>
              </TooltipTrigger>
              <TooltipContent>
                <div className="text-sm">
                  <p className="font-medium">{badge.label}</p>
                  {badge.date && (
                    <p className="text-xs text-muted-foreground">
                      Verificato il {new Date(badge.date).toLocaleDateString('it-IT')}
                    </p>
                  )}
                </div>
              </TooltipContent>
            </Tooltip>
          ))}
      </div>
    </TooltipProvider>
  );
}
API Verifica Email
typescript
// app/api/verify/email/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { generateToken, sendVerificationEmail } from '@/lib/verification';

export async function POST(request: NextRequest) {
  try {
    const { email, userId } = await request.json();

    // Genera token di verifica
    const token = generateToken();
    const expires = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 ore

    // Salva token nel database
    await prisma.verification.upsert({
      where: { userId },
      update: {
        email,
        emailToken: token,
        emailExpires: expires,
        emailVerified: false,
      },
      create: {
        userId,
        email,
        emailToken: token,
        emailExpires: expires,
      },
    });

    // Invia email di verifica
    await sendVerificationEmail(email, token);

    return NextResponse.json({ 
      success: true, 
      message: 'Email di verifica inviata' 
    });
  } catch (error) {
    console.error('Errore invio email verifica:', error);
    return NextResponse.json(
      { error: 'Errore nell\'invio dell\'email di verifica' },
      { status: 500 }
    );
  }
}

// Route per confermare email
export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    const token = searchParams.get('token');

    if (!token) {
      return NextResponse.redirect(new URL('/verification/error', request.url));
    }

    // Trova verifica con token
    const verification = await prisma.verification.findFirst({
      where: {
        emailToken: token,
        emailExpires: {
          gt: new Date(),
        },
      },
    });

    if (!verification) {
      return NextResponse.redirect(new URL('/verification/expired', request.url));
    }

    // Aggiorna stato verifica
    await prisma.verification.update({
      where: { id: verification.id },
      data: {
        emailVerified: true,
        emailToken: null,
        emailExpires: null,
        badges: {
          push: 'EMAIL_VERIFIED',
        },
      },
    });

    return NextResponse.redirect(new URL('/verification/success', request.url));
  } catch (error) {
    console.error('Errore conferma email:', error);
    return NextResponse.redirect(new URL('/verification/error', request.url));
  }
}
KYC Light Implementation
typescript
// app/api/verify/identity/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { verifyIdentity } from '@/lib/kyc-service';

export async function POST(request: NextRequest) {
  try {
    const { userId, documentType, documentData } = await request.json();

    // Inizia processo di verifica
    const verification = await prisma.verification.update({
      where: { userId },
      data: {
        identityStatus: 'IN_REVIEW',
        identityMethod: 'DOCUMENT_UPLOAD',
        identityData: {
          documentType,
          submittedAt: new Date(),
          // Nota: documentData dovrebbe essere già cifrato lato client
        },
      },
    });

    // Processa verifica in background
    processVerificationInBackground(userId, documentData);

    return NextResponse.json({ 
      success: true, 
      verificationId: verification.id,
      status: 'IN_REVIEW',
      message: 'Documento in revisione'
    });
  } catch (error) {
    console.error('Errore verifica identità:', error);
    return NextResponse.json(
      { error: 'Errore nella verifica dell\'identità' },
      { status: 500 }
    );
  }
}

async function processVerificationInBackground(userId: string, documentData: any) {
  try {
    // Integrazione con servizio KYC esterno
    const result = await verifyIdentity(documentData);

    await prisma.verification.update({
      where: { userId },
      data: {
        identityStatus: result.verified ? 'VERIFIED' : 'REJECTED',
        verifiedAt: result.verified ? new Date() : null,
        badges: result.verified ? {
          push: 'IDENTITY_VERIFIED',
        } : undefined,
        identityData: {
          ...documentData,
          verifiedAt: new Date(),
          verificationResult: result,
        },
      },
    });

    // Invia notifica all'utente
    await sendVerificationResult(userId, result.verified);
  } catch (error) {
    console.error('Errore nel background job:', error);
  }
}
Sezione Aggiunta: CUSTOM PROFILE FIELDS
Modelli Prisma
prisma
model CustomFieldDefinition {
  id          String   @id @default(cuid())
  
  // Metadati campo
  name        String   @unique
  label       String
  description String?
  type        FieldType
  category    String   // "personal", "professional", "social", "custom"
  
  // Configurazione
  isRequired  Boolean  @default(false)
  isSystem    Boolean  @default(false) // Campo di sistema non modificabile
  isActive    Boolean  @default(true)
  isPublic    Boolean  @default(true)
  
  // Validazione
  validation  Json?    // {min: 0, max: 100, pattern: "^[A-Za-z]+$"}
  options     Json?    // Per select, checkbox, radio
  
  // Condizioni
  conditions  Json?    // {field: "userType", value: "business"}
  
  // Ordinamento
  position    Int      @default(0)
  
  // Audit
  createdBy   String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  // Relazioni
  values      CustomFieldValue[]
}

model CustomFieldValue {
  id        String   @id @default(cuid())
  
  // Riferimenti
  fieldId   String
  field     CustomFieldDefinition @relation(fields: [fieldId], references: [id])
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  // Valore (tipizzato)
  stringValue   String?
  numberValue   Float?
  booleanValue  Boolean?
  dateValue     DateTime?
  jsonValue     Json?
  
  // Visibilità
  visibility    VisibilityLevel @default(PUBLIC)
  
  // Audit
  updatedAt     DateTime @updatedAt
  createdAt     DateTime @default(now())
  
  @@unique([fieldId, userId])
}

enum FieldType {
  TEXT
  TEXTAREA
  NUMBER
  EMAIL
  PHONE
  DATE
  DATETIME
  BOOLEAN
  SELECT
  MULTISELECT
  RADIO
  CHECKBOX
  URL
  COLOR
  RICH_TEXT
  FILE
  LOCATION
}

enum VisibilityLevel {
  PRIVATE
  FRIENDS_ONLY
  CONNECTIONS_ONLY
  PUBLIC
}
Componente Custom Fields Manager
tsx
// components/profile/CustomFieldsManager.tsx
'use client';

import { useState, useEffect } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import * as z from 'zod';
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from '@/components/ui/form';
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';
import { Switch } from '@/components/ui/switch';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';

interface CustomFieldsManagerProps {
  userId: string;
  fields: any[];
  values: Record<string, any>;
  onSave: (data: Record<string, any>) => Promise<void>;
}

const fieldTypeOptions = [
  { value: 'TEXT', label: 'Testo breve' },
  { value: 'TEXTAREA', label: 'Testo lungo' },
  { value: 'NUMBER', label: 'Numero' },
  { value: 'EMAIL', label: 'Email' },
  { value: 'PHONE', label: 'Telefono' },
  { value: 'DATE', label: 'Data' },
  { value: 'BOOLEAN', label: 'Sì/No' },
  { value: 'SELECT', label: 'Selezione' },
  { value: 'MULTISELECT', label: 'Selezione multipla' },
  { value: 'URL', label: 'URL' },
];

export function CustomFieldsManager({ userId, fields, values, onSave }: CustomFieldsManagerProps) {
  const [editingField, setEditingField] = useState<string | null>(null);

  // Crea schema dinamico in base ai campi
  const createDynamicSchema = () => {
    const schema: Record<string, any> = {};

    fields.forEach((field) => {
      let fieldSchema;

      switch (field.type) {
        case 'TEXT':
        case 'TEXTAREA':
          fieldSchema = z.string();
          if (field.validation?.minLength) {
            fieldSchema = fieldSchema.min(field.validation.minLength);
          }
          if (field.validation?.maxLength) {
            fieldSchema = fieldSchema.max(field.validation.maxLength);
          }
          break;
        case 'NUMBER':
          fieldSchema = z.number();
          if (field.validation?.min !== undefined) {
            fieldSchema = fieldSchema.min(field.validation.min);
          }
          if (field.validation?.max !== undefined) {
            fieldSchema = fieldSchema.max(field.validation.max);
          }
          break;
        case 'EMAIL':
          fieldSchema = z.string().email();
          break;
        case 'URL':
          fieldSchema = z.string().url();
          break;
        case 'BOOLEAN':
          fieldSchema = z.boolean();
          break;
        case 'DATE':
          fieldSchema = z.string().datetime();
          break;
        default:
          fieldSchema = z.string();
      }

      if (field.isRequired && !field.isSystem) {
        fieldSchema = fieldSchema.refine(val => val != null && val !== '', {
          message: `${field.label} è obbligatorio`,
        });
      }

      schema[field.name] = fieldSchema;
    });

    return z.object(schema);
  };

  const form = useForm({
    resolver: zodResolver(createDynamicSchema()),
    defaultValues: values,
  });

  const renderField = (field: any) => {
    const baseProps = {
      ...form.register(field.name),
      disabled: field.isSystem,
    };

    switch (field.type) {
      case 'TEXT':
        return (
          <FormField
            control={form.control}
            name={field.name}
            render={({ field: formField }) => (
              <FormItem>
                <FormLabel className="flex items-center gap-2">
                  {field.label}
                  {field.isRequired && <Badge variant="outline" className="text-xs">Obbligatorio</Badge>}
                  {field.isSystem && <Badge variant="secondary" className="text-xs">Sistema</Badge>}
                </FormLabel>
                <FormControl>
                  <Input {...formField} placeholder={field.placeholder} />
                </FormControl>
                {field.description && (
                  <FormDescription>{field.description}</FormDescription>
                )}
                <FormMessage />
              </FormItem>
            )}
          />
        );

      case 'TEXTAREA':
        return (
          <FormField
            control={form.control}
            name={field.name}
            render={({ field: formField }) => (
              <FormItem>
                <FormLabel>{field.label}</FormLabel>
                <FormControl>
                  <Textarea {...formField} rows={4} />
                </FormControl>
                <FormMessage />
              </FormItem>
            )}
          />
        );

      case 'SELECT':
        return (
          <FormField
            control={form.control}
            name={field.name}
            render={({ field: formField }) => (
              <FormItem>
                <FormLabel>{field.label}</FormLabel>
                <Select onValueChange={formField.onChange} defaultValue={formField.value}>
                  <FormControl>
                    <SelectTrigger>
                      <SelectValue placeholder={`Seleziona ${field.label}`} />
                    </SelectTrigger>
                  </FormControl>
                  <SelectContent>
                    {field.options?.map((option: string) => (
                      <SelectItem key={option} value={option}>
                        {option}
                      </SelectItem>
                    ))}
                  </SelectContent>
                </Select>
                <FormMessage />
              </FormItem>
            )}
          />
        );

      case 'BOOLEAN':
        return (
          <FormField
            control={form.control}
            name={field.name}
            render={({ field: formField }) => (
              <FormItem className="flex flex-row items-center justify-between rounded-lg border p-3">
                <div className="space-y-0.5">
                  <FormLabel>{field.label}</FormLabel>
                  {field.description && (
                    <FormDescription>{field.description}</FormDescription>
                  )}
                </div>
                <FormControl>
                  <Switch
                    checked={formField.value}
                    onCheckedChange={formField.onChange}
                  />
                </FormControl>
              </FormItem>
            )}
          />
        );

      default:
        return null;
    }
  };

  const onSubmit = async (data: any) => {
    try {
      await onSave(data);
      form.reset(data);
    } catch (error) {
      console.error('Errore nel salvataggio:', error);
    }
  };

  return (
    <div className="space-y-6">
      <Form {...form}>
        <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-6">
          {fields
            .filter(field => field.isActive)
            .sort((a, b) => a.position - b.position)
            .map((field) => (
              <div key={field.id} className="space-y-4">
                {renderField(field)}
              </div>
            ))}

          <Button type="submit" disabled={form.formState.isSubmitting}>
            {form.formState.isSubmitting ? 'Salvataggio...' : 'Salva modifiche'}
          </Button>
        </form>
      </Form>
    </div>
  );
}
API Admin Custom Fields
typescript
// app/api/admin/fields/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session?.user?.isAdmin) {
      return NextResponse.json({ error: 'Non autorizzato' }, { status: 403 });
    }

    const fields = await prisma.customFieldDefinition.findMany({
      orderBy: [
        { category: 'asc' },
        { position: 'asc' },
      ],
    });

    return NextResponse.json(fields);
  } catch (error) {
    console.error('Errore recupero campi:', error);
    return NextResponse.json(
      { error: 'Errore interno del server' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session?.user?.isAdmin) {
      return NextResponse.json({ error: 'Non autorizzato' }, { status: 403 });
    }

    const data = await request.json();
    
    const field = await prisma.customFieldDefinition.create({
      data: {
        ...data,
        createdBy: session.user.id,
      },
    });

    return NextResponse.json(field);
  } catch (error) {
    console.error('Errore creazione campo:', error);
    return NextResponse.json(
      { error: 'Errore interno del server' },
      { status: 500 }
    );
  }
}

export async function PUT(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions);
    if (!session?.user?.isAdmin) {
      return NextResponse.json({ error: 'Non autorizzato' }, { status: 403 });
    }

    const data = await request.json();
    const { id, ...updateData } = data;

    const field = await prisma.customFieldDefinition.update({
      where: { id },
      data: updateData,
    });

    return NextResponse.json(field);
  } catch (error) {
    console.error('Errore aggiornamento campo:', error);
    return NextResponse.json(
      { error: 'Errore interno del server' },
      { status: 500 }
    );
  }
}
Sezione Aggiunta: PROFILE SEO
Layout SEO Profile
tsx
// app/profiles/[username]/layout.tsx
import { Metadata } from 'next';
import { notFound } from 'next/navigation';
import { prisma } from '@/lib/prisma';
import { ProfileSEO } from '@/components/profile/ProfileSEO';

interface ProfileLayoutProps {
  params: { username: string };
  children: React.ReactNode;
}

export async function generateMetadata({ params }: ProfileLayoutProps): Promise<Metadata> {
  const profile = await prisma.user.findUnique({
    where: { username: params.username },
    include: {
      socialProfiles: true,
      customFields: {
        include: {
          field: true,
        },
      },
    },
  });

  if (!profile) {
    return {
      title: 'Profilo non trovato',
    };
  }

  const metadata: Metadata = {
    title: `${profile.name} | ${profile.title || 'Profilo'} ${profile.company ? `@ ${profile.company}` : ''}`,
    description: profile.bio?.substring(0, 160) || `Profilo di ${profile.name}`,
    keywords: [
      profile.name,
      ...(profile.skills || []),
      profile.location,
      profile.company,
    ].filter(Boolean) as string[],
    authors: [{ name: profile.name }],
    openGraph: {
      type: 'profile',
      title: profile.name,
      description: profile.bio?.substring(0, 160) || '',
      url: `https://yourdomain.com/profiles/${profile.username}`,
      siteName: 'Your Platform',
      images: profile.avatar ? [
        {
          url: profile.avatar,
          width: 1200,
          height: 630,
          alt: `Avatar di ${profile.name}`,
        },
      ] : [],
    },
    twitter: {
      card: 'summary_large_image',
      title: profile.name,
      description: profile.bio?.substring(0, 160) || '',
      images: profile.avatar ? [profile.avatar] : [],
      creator: profile.socialProfiles?.find(s => s.platform === 'twitter')?.username,
    },
    alternates: {
      canonical: `https://yourdomain.com/profiles/${profile.username}`,
    },
  };

  return metadata;
}

export default async function ProfileLayout({ params, children }: ProfileLayoutProps) {
  const profile = await prisma.user.findUnique({
    where: { username: params.username },
  });

  if (!profile) {
    notFound();
  }

  // Genera structured data Person
  const structuredData = {
    '@context': 'https://schema.org',
    '@type': 'Person',
    name: profile.name,
    description: profile.bio,
    url: `https://yourdomain.com/profiles/${profile.username}`,
    image: profile.avatar,
    email: profile.email,
    jobTitle: profile.title,
    worksFor: {
      '@type': 'Organization',
      name: profile.company,
    },
    sameAs: profile.socialProfiles?.map((sp: any) => sp.url),
    address: profile.location ? {
      '@type': 'PostalAddress',
      addressLocality: profile.location,
    } : undefined,
  };

  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(structuredData) }}

════════════════════════════════════════════════════════════
FIGMA CATALOG: WEBBY-31-SOCIAL-PROFILES
Prompt ID: 31 / 48
Parte: 2
Exported: 2026-02-06T12:31:47.176Z
Characters: 515
════════════════════════════════════════════════════════════

wnMenuItem onClick={handleBookmark}>
            <Bookmark className="h-4 w-4 mr-2" />
            Salva nei segnalibri
          </DropdownMenuItem>
          
          <DropdownMenuItem onClick={() => setShowContactDialog(true)}>
            <Mail className="h-4 w-4 mr-2" />
            Richiedi contatto
          </DropdownMenuItem>
          
          <DropdownMenuSeparator />
          
          <DropdownMenuItem 
            onClick={() => setShowReportDialog(true)}
            className="text-red-600