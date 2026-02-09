# CATALOGO FORM VALIDATION v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Zod validation, React Hook Form, validation patterns, error handling, UX
> **Sezioni**: 1-10

---

§ 1. VALIDATION LIBRARY COMPARISON

| Library | Size | TypeScript | Async | Composition | Performance | Best For |
|---------|------|------------|-------|-------------|-------------|----------|
| Zod | 12KB | ✅ Native | ✅ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | Full-stack TypeScript |
| Yup | 15KB | ⚠️ @types | ✅ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Legacy React projects |
| Valibot | 5KB | ✅ Native | ✅ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Bundle-sensitive apps |
| Joi | 35KB | ⚠️ @types | ✅ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Node.js backend only |
| ArkType | 8KB | ✅ Native | ✅ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Performance-critical |
| Superstruct | 4KB | ✅ Native | ❌ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Minimal validation |
| io-ts | 10KB | ✅ Native | ✅ | ⭐⭐⭐⭐ | ⭐⭐⭐ | Functional programming |

---

§ 2. ZOD FUNDAMENTALS

§ 2.1 PRIMITIVE TYPES

typescript
import { z } from 'zod';

// ===== STRINGS =====
const stringSchema = z.string();
const emailSchema = z.string().email('Invalid email address');
const urlSchema = z.string().url('Invalid URL');
const uuidSchema = z.string().uuid('Invalid UUID');
const cuidSchema = z.string().cuid('Invalid CUID');
const minMaxSchema = z.string().min(2, 'Min 2 chars').max(100, 'Max 100 chars');
const regexSchema = z.string().regex(/^[A-Z]{2}\d{4}$/, 'Format: XX0000');
const trimSchema = z.string().trim(); // Trims whitespace
const lowerSchema = z.string().toLowerCase();

// ===== NUMBERS =====
const numberSchema = z.number();
const intSchema = z.number().int('Must be integer');
const positiveSchema = z.number().positive('Must be positive');
const rangeSchema = z.number().min(0).max(100);
const multipleOfSchema = z.number().multipleOf(5);

// ===== BOOLEANS =====
const boolSchema = z.boolean();

// ===== DATES =====
const dateSchema = z.date();
const minDateSchema = z.date().min(new Date('2020-01-01'));
const dateStringSchema = z.string().datetime(); // ISO 8601

// ===== ENUMS =====
const roleSchema = z.enum(['USER', 'ADMIN', 'MODERATOR']);
type Role = z.infer<typeof roleSchema>; // 'USER' | 'ADMIN' | 'MODERATOR'

// Native TypeScript enum
enum Status { Active = 'ACTIVE', Inactive = 'INACTIVE' }
const statusSchema = z.nativeEnum(Status);

// ===== NULLABLES =====
const nullableSchema = z.string().nullable(); // string | null
const optionalSchema = z.string().optional(); // string | undefined
const nullishSchema = z.string().nullish(); // string | null | undefined
const defaultSchema = z.string().default('default value');

// ===== COERCION (auto-convert) =====
const coerceNumber = z.coerce.number(); // "123" → 123
const coerceBoolean = z.coerce.boolean(); // "true" → true
const coerceDate = z.coerce.date(); // "2024-01-01" → Date

§ 2.2 OBJECT SCHEMAS

typescript
import { z } from 'zod';

// Basic object
const userSchema = z.object({
  id: z.string().cuid(),
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().int().positive().optional(),
  role: z.enum(['USER', 'ADMIN']).default('USER'),
  tags: z.array(z.string()).default([]),
  createdAt: z.date().default(() => new Date()),
});

// Infer TypeScript type
type User = z.infer<typeof userSchema>;

// ===== OBJECT MODIFIERS =====

// Partial - all fields optional
const partialSchema = userSchema.partial();

// Pick - select specific fields
const publicUserSchema = userSchema.pick({ id: true, name: true });

// Omit - exclude fields
const createUserSchema = userSchema.omit({ id: true, createdAt: true });

// Extend - add new fields
const adminSchema = userSchema.extend({
  permissions: z.array(z.string()),
});

// Strict - error on unknown keys
const strictSchema = userSchema.strict();

// Strip - remove unknown keys silently
const stripSchema = userSchema.strip();

// Usage example:
const result = userSchema.safeParse({
  id: 'clx123',
  email: 'test@example.com',
  name: 'John Doe',
});

if (result.success) {
  console.log(result.data); // Typed as User
} else {
  console.log(result.error.flatten());
}

§ 2.3 ADVANCED PATTERNS

typescript
import { z } from 'zod';

// ===== DISCRIMINATED UNIONS =====
const notificationSchema = z.discriminatedUnion('type', [
  z.object({
    type: z.literal('email'),
    email: z.string().email(),
    subject: z.string(),
  }),
  z.object({
    type: z.literal('sms'),
    phone: z.string(),
    message: z.string().max(160),
  }),
  z.object({
    type: z.literal('push'),
    deviceId: z.string(),
    title: z.string(),
  }),
]);

// ===== RECURSIVE TYPES =====
interface Category {
  name: string;
  children: Category[];
}

const categorySchema: z.ZodType<Category> = z.lazy(() =>
  z.object({
    name: z.string(),
    children: z.array(categorySchema),
  })
);

// ===== TRANSFORM =====
const dateTransform = z.string().transform((str) => new Date(str));
const trimTransform = z.string().transform((s) => s.trim().toLowerCase());

// ===== REFINE (custom validation) =====
const passwordSchema = z.string()
  .min(8, 'Min 8 characters')
  .refine((val) => /[A-Z]/.test(val), { message: 'Must contain uppercase' })
  .refine((val) => /[a-z]/.test(val), { message: 'Must contain lowercase' })
  .refine((val) => /[0-9]/.test(val), { message: 'Must contain number' })
  .refine((val) => /[^A-Za-z0-9]/.test(val), { message: 'Must contain special char' });

// ===== SUPERREFINE (multiple errors) =====
const registerSchema = z.object({
  password: z.string().min(8),
  confirmPassword: z.string(),
}).superRefine((data, ctx) => {
  if (data.password !== data.confirmPassword) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: 'Passwords do not match',
      path: ['confirmPassword'],
    });
  }
});

// ===== ASYNC VALIDATION =====
const uniqueEmailSchema = z.string().email().refine(
  async (email) => {
    const exists = await checkEmailExists(email);
    return !exists;
  },
  { message: 'Email already registered' }
);


---

§ 3. REACT HOOK FORM INTEGRATION

§ 3.1 BASIC SETUP

typescript
// lib/form.ts
import { useForm, UseFormReturn, DefaultValues } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

export function useZodForm<TSchema extends z.ZodType>(
  schema: TSchema,
  options?: {
    defaultValues?: DefaultValues<z.infer<TSchema>>;
    mode?: 'onBlur' | 'onChange' | 'onSubmit' | 'onTouched' | 'all';
  }
): UseFormReturn<z.infer<TSchema>> {
  return useForm<z.infer<TSchema>>({
    resolver: zodResolver(schema),
    defaultValues: options?.defaultValues,
    mode: options?.mode ?? 'onBlur',
  });
}

// Usage:
const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

function LoginForm() {
  const form = useZodForm(loginSchema, {
    defaultValues: { email: '', password: '' },
  });

  const onSubmit = form.handleSubmit(async (data) => {
    await login(data.email, data.password);
  });

  return <form onSubmit={onSubmit}>...</form>;
}

§ 3.2 FORM FIELD COMPONENT

typescript
// components/ui/form-field.tsx
'use client';

import { Controller, FieldPath, FieldValues, useFormContext } from 'react-hook-form';
import { cn } from '@/lib/utils';

interface FormFieldProps<T extends FieldValues> {
  name: FieldPath<T>;
  label: string;
  description?: string;
  children: React.ReactNode;
}

export function FormField<T extends FieldValues>({
  name,
  label,
  description,
  children,
}: FormFieldProps<T>) {
  const { formState: { errors } } = useFormContext<T>();
  const error = errors[name];

  return (
    <div className="space-y-2">
      <label htmlFor={name} className="text-sm font-medium">
        {label}
      </label>
      {children}
      {description && !error && (
        <p className="text-sm text-muted-foreground">{description}</p>
      )}
      {error && (
        <p className="text-sm text-destructive" role="alert">
          {error.message as string}
        </p>
      )}
    </div>
  );
}

§ 3.3 COMPLETE FORM EXAMPLE

typescript
// components/contact-form.tsx
'use client';

import { z } from 'zod';
import { useZodForm } from '@/lib/form';
import { FormProvider } from 'react-hook-form';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Textarea } from '@/components/ui/textarea';

const contactSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Please enter a valid email'),
  subject: z.string().min(5, 'Subject must be at least 5 characters'),
  message: z.string().min(20, 'Message must be at least 20 characters').max(1000),
});

type ContactFormData = z.infer<typeof contactSchema>;

export function ContactForm() {
  const form = useZodForm(contactSchema, {
    defaultValues: { name: '', email: '', subject: '', message: '' },
  });

  const { register, handleSubmit, formState: { errors, isSubmitting } } = form;

  const onSubmit = async (data: ContactFormData) => {
    try {
      await fetch('/api/contact', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(data),
      });
      form.reset();
    } catch (error) {
      form.setError('root', { message: 'Failed to send message' });
    }
  };

  return (
    <FormProvider {...form}>
      <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
        <div>
          <label htmlFor="name">Name</label>
          <Input id="name" {...register('name')} aria-invalid={!!errors.name} />
          {errors.name && <p className="text-destructive text-sm">{errors.name.message}</p>}
        </div>

        <div>
          <label htmlFor="email">Email</label>
          <Input id="email" type="email" {...register('email')} aria-invalid={!!errors.email} />
          {errors.email && <p className="text-destructive text-sm">{errors.email.message}</p>}
        </div>

        <div>
          <label htmlFor="subject">Subject</label>
          <Input id="subject" {...register('subject')} aria-invalid={!!errors.subject} />
          {errors.subject && <p className="text-destructive text-sm">{errors.subject.message}</p>}
        </div>

        <div>
          <label htmlFor="message">Message</label>
          <Textarea id="message" {...register('message')} aria-invalid={!!errors.message} />
          {errors.message && <p className="text-destructive text-sm">{errors.message.message}</p>}
        </div>

        {errors.root && <p className="text-destructive">{errors.root.message}</p>}

        <Button type="submit" disabled={isSubmitting}>
          {isSubmitting ? 'Sending...' : 'Send Message'}
        </Button>
      </form>
    </FormProvider>
  );
}

---

§ 4. COMMON VALIDATION PATTERNS

§ 4.1 PATTERN REFERENCE TABLE

| Pattern | Schema | Example Valid | Example Invalid |
|---------|--------|---------------|-----------------|
| Email | `z.string().email()` | user@example.com | invalid-email |
| URL | `z.string().url()` | https://example.com | not-a-url |
| Phone (intl) | `z.string().regex(/^\+[1-9]\d{1,14}$/)` | +1234567890 | 123-456 |
| Password | `z.string().min(8).regex(/[A-Z]/).regex(/[0-9]/)` | MyPass123! | weak |
| Username | `z.string().regex(/^[a-z0-9_]{3,20}$/)` | john_doe | John Doe |
| Slug | `z.string().regex(/^[a-z0-9]+(?:-[a-z0-9]+)*$/)` | my-post-title | My Post! |
| Credit Card | `z.string().regex(/^\d{16}$/)` | 1234567890123456 | 123 |
| ZIP (US) | `z.string().regex(/^\d{5}(-\d{4})?$/)` | 12345 | 1234 |
| Date (ISO) | `z.string().datetime()` | 2024-01-15T00:00:00Z | 01/15/2024 |
| UUID | `z.string().uuid()` | 550e8400-e29b-41d4... | not-uuid |

§ 4.2 REUSABLE SCHEMA LIBRARY

typescript
// lib/schemas/common.ts
import { z } from 'zod';

export const schemas = {
  // Identity
  email: z.string().email('Invalid email address'),
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(/[A-Z]/, 'Password must contain at least one uppercase letter')
    .regex(/[a-z]/, 'Password must contain at least one lowercase letter')
    .regex(/[0-9]/, 'Password must contain at least one number'),
  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .max(20, 'Username must be at most 20 characters')
    .regex(/^[a-z0-9_]+$/, 'Username can only contain lowercase letters, numbers, and underscores'),
  
  // Contact
  phone: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),
  url: z.string().url('Invalid URL'),
  
  // Address
  zipCodeUS: z.string().regex(/^\d{5}(-\d{4})?$/, 'Invalid ZIP code'),
  state: z.string().length(2, 'State must be 2 letters').toUpperCase(),
  
  // Financial
  currency: z.number().positive().multipleOf(0.01),
  creditCard: z.string().regex(/^\d{13,19}$/, 'Invalid card number'),
  
  // Content
  slug: z.string().regex(/^[a-z0-9]+(?:-[a-z0-9]+)*$/, 'Invalid slug format'),
  
  // IDs
  cuid: z.string().cuid('Invalid ID format'),
  uuid: z.string().uuid('Invalid UUID format'),
};

// Usage:
const userSchema = z.object({
  email: schemas.email,
  password: schemas.password,
  username: schemas.username,
});



---

§ 5. ERROR DISPLAY PATTERNS

§ 5.1 ERROR MESSAGE COMPONENT

typescript
// components/ui/field-error.tsx
import { cn } from '@/lib/utils';
import { AlertCircle } from 'lucide-react';

interface FieldErrorProps {
  message?: string;
  className?: string;
}

export function FieldError({ message, className }: FieldErrorProps) {
  if (!message) return null;

  return (
    <div
      role="alert"
      className={cn(
        'flex items-center gap-1.5 text-sm text-destructive mt-1.5',
        className
      )}
    >
      <AlertCircle className="h-4 w-4 shrink-0" />
      <span>{message}</span>
    </div>
  );
}

§ 5.2 FORM ERROR SUMMARY

typescript
// components/ui/form-error-summary.tsx
import { FieldErrors, FieldValues } from 'react-hook-form';
import { AlertCircle } from 'lucide-react';

interface FormErrorSummaryProps<T extends FieldValues> {
  errors: FieldErrors<T>;
  fieldLabels?: Record<string, string>;
}

export function FormErrorSummary<T extends FieldValues>({
  errors,
  fieldLabels = {},
}: FormErrorSummaryProps<T>) {
  const errorList = Object.entries(errors)
    .filter(([_, error]) => error?.message)
    .map(([field, error]) => ({
      field: fieldLabels[field] || field,
      message: error?.message as string,
    }));

  if (errorList.length === 0) return null;

  return (
    <div
      role="alert"
      aria-live="polite"
      className="rounded-md bg-destructive/10 p-4 border border-destructive/20"
    >
      <div className="flex items-start gap-3">
        <AlertCircle className="h-5 w-5 text-destructive shrink-0 mt-0.5" />
        <div>
          <h3 className="font-medium text-destructive">
            Please fix the following errors:
          </h3>
          <ul className="mt-2 list-disc list-inside space-y-1 text-sm text-destructive/90">
            {errorList.map(({ field, message }) => (
              <li key={field}>
                <strong>{field}:</strong> {message}
              </li>
            ))}
          </ul>
        </div>
      </div>
    </div>
  );
}

§ 5.3 INLINE VS SUMMARY DISPLAY

| Strategy | Best For | UX Impact |
|----------|----------|-----------|
| Inline only | Simple forms (<5 fields) | Immediate feedback |
| Summary only | Long forms, wizard steps | Overview before submit |
| Inline + Summary | Complex forms | Best of both worlds |
| Toast notification | Background validation | Non-blocking |

---

§ 6. CONDITIONAL VALIDATION

§ 6.1 DEPENDENT FIELD VALIDATION

typescript
import { z } from 'zod';

// Shipping address required only if "ship to different address" is checked
const checkoutSchema = z.object({
  billingAddress: z.string().min(10),
  shipToDifferent: z.boolean(),
  shippingAddress: z.string().optional(),
}).refine(
  (data) => !data.shipToDifferent || (data.shippingAddress && data.shippingAddress.length >= 10),
  {
    message: 'Shipping address is required',
    path: ['shippingAddress'],
  }
);

// Business fields required only for business accounts
const accountSchema = z.discriminatedUnion('accountType', [
  z.object({
    accountType: z.literal('personal'),
    fullName: z.string().min(2),
    email: z.string().email(),
  }),
  z.object({
    accountType: z.literal('business'),
    fullName: z.string().min(2),
    email: z.string().email(),
    companyName: z.string().min(2),
    vatNumber: z.string().regex(/^[A-Z]{2}\d{8,12}$/),
  }),
]);

§ 6.2 DYNAMIC SCHEMA SELECTION

typescript
// hooks/use-dynamic-schema.ts
import { useMemo } from 'react';
import { z } from 'zod';

const baseSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
});

const businessExtension = z.object({
  companyName: z.string().min(2),
  taxId: z.string().min(5),
});

export function useDynamicSchema(isBusinessAccount: boolean) {
  return useMemo(() => {
    if (isBusinessAccount) {
      return baseSchema.merge(businessExtension);
    }
    return baseSchema;
  }, [isBusinessAccount]);
}

// Usage in component:
function AccountForm() {
  const [isBusiness, setIsBusiness] = useState(false);
  const schema = useDynamicSchema(isBusiness);
  const form = useZodForm(schema);
  // ...
}

---

§ 7. MULTI-STEP FORM VALIDATION

§ 7.1 STEP SCHEMA PATTERN

typescript
import { z } from 'zod';

// Step 1: Personal Info
const personalInfoSchema = z.object({
  firstName: z.string().min(2),
  lastName: z.string().min(2),
  email: z.string().email(),
});

// Step 2: Address
const addressSchema = z.object({
  street: z.string().min(5),
  city: z.string().min(2),
  zipCode: z.string().regex(/^\d{5}$/),
  country: z.string().min(2),
});

// Step 3: Payment
const paymentSchema = z.object({
  cardNumber: z.string().regex(/^\d{16}$/),
  expiryDate: z.string().regex(/^\d{2}\/\d{2}$/),
  cvv: z.string().regex(/^\d{3,4}$/),
});

// Combined schema for final submission
const completeSchema = personalInfoSchema
  .merge(addressSchema)
  .merge(paymentSchema);

// Step schemas array
const stepSchemas = [personalInfoSchema, addressSchema, paymentSchema] as const;

type FormData = z.infer<typeof completeSchema>;

§ 7.2 MULTI-STEP FORM HOOK

typescript
// hooks/use-multi-step-form.ts
import { useState, useCallback } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

interface UseMultiStepFormOptions<T extends z.ZodType[]> {
  schemas: T;
  onComplete: (data: z.infer<T[number]>) => void | Promise<void>;
}

export function useMultiStepForm<T extends z.ZodType[]>({
  schemas,
  onComplete,
}: UseMultiStepFormOptions<T>) {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState<Partial<z.infer<T[number]>>>({});

  const currentSchema = schemas[currentStep];
  const isLastStep = currentStep === schemas.length - 1;

  const form = useForm({
    resolver: zodResolver(currentSchema),
    defaultValues: formData,
  });

  const nextStep = useCallback(async () => {
    const isValid = await form.trigger();
    if (!isValid) return false;

    const stepData = form.getValues();
    setFormData((prev) => ({ ...prev, ...stepData }));

    if (isLastStep) {
      await onComplete({ ...formData, ...stepData });
    } else {
      setCurrentStep((prev) => prev + 1);
    }
    return true;
  }, [form, formData, isLastStep, onComplete]);

  const prevStep = useCallback(() => {
    if (currentStep > 0) {
      setCurrentStep((prev) => prev - 1);
    }
  }, [currentStep]);

  return {
    form,
    currentStep,
    totalSteps: schemas.length,
    isFirstStep: currentStep === 0,
    isLastStep,
    nextStep,
    prevStep,
    formData,
  };
}

---

§ 8. SERVER-SIDE VALIDATION

§ 8.1 API ROUTE VALIDATION

typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  password: z.string().min(8),
});

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const data = createUserSchema.parse(body);

    // Create user...
    const user = await createUser(data);

    return NextResponse.json({ success: true, data: user }, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        {
          success: false,
          error: {
            code: 'VALIDATION_ERROR',
            message: 'Invalid input',
            details: error.flatten().fieldErrors,
          },
        },
        { status: 422 }
      );
    }
    throw error;
  }
}

§ 8.2 SERVER ACTION VALIDATION

typescript
// app/actions/user.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';

const updateProfileSchema = z.object({
  name: z.string().min(2).max(100),
  bio: z.string().max(500).optional(),
});

export async function updateProfile(formData: FormData) {
  const rawData = {
    name: formData.get('name'),
    bio: formData.get('bio'),
  };

  const result = updateProfileSchema.safeParse(rawData);

  if (!result.success) {
    return {
      success: false,
      errors: result.error.flatten().fieldErrors,
    };
  }

  // Update user...
  await updateUser(result.data);
  revalidatePath('/profile');

  return { success: true };
}



---

§ 9. FORM ACCESSIBILITY

§ 9.1 A11Y REQUIREMENTS TABLE

| Requirement | Implementation | WCAG |
|-------------|----------------|------|
| Labels | `<label htmlFor={id}>` connected to input | 1.3.1 |
| Error identification | `aria-invalid="true"` on invalid fields | 3.3.1 |
| Error description | `aria-describedby` pointing to error message | 3.3.1 |
| Required fields | `aria-required="true"` or `required` attribute | 3.3.2 |
| Error announcements | `role="alert"` or `aria-live="polite"` | 4.1.3 |
| Focus management | Focus first error field on submit | 2.4.3 |
| Keyboard navigation | Tab order, Enter to submit | 2.1.1 |
| Instructions | Visible format hints before input | 3.3.2 |

§ 9.2 ACCESSIBLE INPUT COMPONENT

typescript
// components/ui/accessible-input.tsx
import { forwardRef } from 'react';
import { cn } from '@/lib/utils';

interface AccessibleInputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
  description?: string;
}

export const AccessibleInput = forwardRef<HTMLInputElement, AccessibleInputProps>(
  ({ label, error, description, id, className, required, ...props }, ref) => {
    const inputId = id || `input-${label.toLowerCase().replace(/\s/g, '-')}`;
    const errorId = `${inputId}-error`;
    const descriptionId = `${inputId}-description`;

    const describedBy = [
      error ? errorId : null,
      description ? descriptionId : null,
    ].filter(Boolean).join(' ') || undefined;

    return (
      <div className="space-y-1.5">
        <label
          htmlFor={inputId}
          className={cn('text-sm font-medium', error && 'text-destructive')}
        >
          {label}
          {required && <span className="text-destructive ml-1" aria-hidden="true">*</span>}
        </label>

        <input
          ref={ref}
          id={inputId}
          className={cn(
            'w-full px-3 py-2 border rounded-md',
            error && 'border-destructive focus:ring-destructive',
            className
          )}
          aria-invalid={error ? 'true' : undefined}
          aria-required={required}
          aria-describedby={describedBy}
          {...props}
        />

        {description && !error && (
          <p id={descriptionId} className="text-sm text-muted-foreground">
            {description}
          </p>
        )}

        {error && (
          <p id={errorId} role="alert" className="text-sm text-destructive">
            {error}
          </p>
        )}
      </div>
    );
  }
);

AccessibleInput.displayName = 'AccessibleInput';

§ 9.3 FOCUS ERROR FIELD ON SUBMIT

typescript
// hooks/use-focus-error.ts
import { useEffect } from 'react';
import { FieldErrors, FieldValues } from 'react-hook-form';

export function useFocusError<T extends FieldValues>(
  errors: FieldErrors<T>,
  isSubmitted: boolean
) {
  useEffect(() => {
    if (!isSubmitted) return;

    const firstErrorField = Object.keys(errors)[0];
    if (!firstErrorField) return;

    const element = document.querySelector(
      `[name="${firstErrorField}"], #${firstErrorField}`
    ) as HTMLElement | null;

    if (element) {
      element.focus();
      element.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
  }, [errors, isSubmitted]);
}

// Usage in form:
function MyForm() {
  const { formState: { errors, isSubmitted } } = useForm();
  useFocusError(errors, isSubmitted);
  // ...
}

---

§ 10. FORM VALIDATION CHECKLIST

SCHEMA DESIGN
□ Zod schemas defined for all forms
□ Type inference with z.infer<typeof schema>
□ Reusable schema patterns extracted
□ Custom error messages provided
□ Async validation for unique fields

REACT HOOK FORM SETUP
□ zodResolver configured
□ Mode set appropriately (onBlur recommended)
□ Default values provided
□ Form state destructured correctly
□ handleSubmit wrapper used

ERROR HANDLING
□ Field-level errors displayed
□ Form-level errors handled (root errors)
□ Error summary for complex forms
□ Errors cleared on field change
□ Server errors mapped to fields

VALIDATION PATTERNS
□ Email format validated
□ Password strength enforced
□ Phone number format checked
□ Required fields marked
□ Conditional validation implemented

ACCESSIBILITY
□ Labels connected to inputs (htmlFor)
□ aria-invalid on error fields
□ aria-describedby for error messages
□ aria-required on required fields
□ Error announcements with role="alert"
□ Focus moved to first error on submit
□ Tab order logical
□ Submit on Enter works

USER EXPERIENCE
□ Real-time validation (onBlur/onChange)
□ Clear error messages (user-friendly)
□ Loading states during submission
□ Success feedback provided
□ Form reset after success (if appropriate)

SERVER INTEGRATION
□ Same schema used client & server
□ Server validation errors displayed
□ Network errors handled gracefully
□ Optimistic UI (if applicable)

MULTI-STEP FORMS (if applicable)
□ Per-step validation schemas
□ Data persisted between steps
□ Back navigation preserves data
□ Progress indicator visible
□ Final validation on complete schema

TESTING
□ Schema unit tests
□ Form component tests
□ Error display tests
□ Accessibility audit passed
□ E2E form submission tests


§ 11. ADVANCED VALIDATION PATTERNS

§ 11.1 CONDITIONAL VALIDATION

typescript
import { z } from 'zod';

const conditionalSchema = z.object({
  isAdult: z.boolean(),
  age: z.number().refine((age, ctx) => {
    if (ctx.parent.isAdult) {
      return age >= 18;
    }
    return true;
  }, {
    message: 'Must be 18 or older to be considered an adult',
  }),
});

§ 11.2 DYNAMIC VALIDATION

typescript
import { z } from 'zod';

const dynamicSchema = z.object({
  type: z.string(),
  value: z.string().refine((value, ctx) => {
    if (ctx.parent.type === 'email') {
      return value.includes('@');
    }
    return true;
  }, {
    message: 'Invalid value for selected type',
  }),
});

§ 12. ERROR HANDLING AND DISPLAY

§ 12.1 CUSTOM ERROR MESSAGES

typescript
import { z } from 'zod';

const customErrorSchema = z.object({
  name: z.string().min(2, { message: 'Name must be at least 2 characters' }),
});

§ 12.2 ERROR HANDLING WITH REACT HOOK FORM

typescript
import { useForm } from 'react-hook-form';
import { z } from 'zod';

const MyForm = () => {
  const { register, handleSubmit, errors } = useForm({
    resolver: zodResolver(customErrorSchema),
  });

  const onSubmit = async (data) => {
    // Submit logic
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('name')} />
      {errors.name && <div>{errors.name.message}</div>}
    </form>
  );
};

§ 13. PERFORMANCE CONSIDERATIONS

§ 13.1 MEMOIZATION

typescript
import { useMemo } from 'react';
import { z } from 'zod';

const MyComponent = () => {
  const schema = useMemo(() => z.object({
    name: z.string(),
  }), []);

  return (
    <div>
      {/* Render schema-dependent components */}
    </div>
  );
};

§ 13.2 LAZY LOADING

typescript
import { useState, useEffect } from 'react';
import { z } from 'zod';

const MyComponent = () => {
  const [schema, setSchema] = useState(null);

  useEffect(() => {
    import('./schema').then((schemaModule) => {
      setSchema(schemaModule.default);
    });
  }, []);

  return (
    <div>
      {schema && (
        {/* Render schema-dependent components */}
      )}
    </div>
  );
};

§ 14. TESTING PATTERNS WITH VITEST

§ 14.1 UNIT TESTING

typescript
import { describe, it, expect } from 'vitest';
import { z } from 'zod';

describe('MySchema', () => {
  it('should validate correctly', () => {
    const schema = z.object({
      name: z.string(),
    });

    const validData = { name: 'John Doe' };
    const invalidData = { name: 123 };

    expect(schema.parse(validData)).toEqual(validData);
    expect(() => schema.parse(invalidData)).toThrowError();
  });
});

§ 14.2 INTEGRATION TESTING

typescript
import { describe, it, expect } from 'vitest';
import { render, fireEvent, waitFor } from '@testing-library/react';
import { MyForm } from './MyForm';

describe('MyForm', () => {
  it('should render correctly', () => {
    const { getByText } = render(<MyForm />);

    expect(getByText('Name')).toBeInTheDocument();
  });

  it('should handle submission correctly', async () => {
    const { getByText, getByLabelText } = render(<MyForm />);

    const nameInput = getByLabelText('Name');
    const submitButton = getByText('Submit');

    fireEvent.change(nameInput, { target: { value: 'John Doe' } });
    fireEvent.click(submitButton);

    await waitFor(() => expect(getByText('Form submitted successfully')).toBeInTheDocument());
  });
});

§ 15. BEST PRACTICES

§ 15.1 VALIDATION STRATEGY

| Strategy | Description | ✅ DO | ❌ DON'T |
| --- | --- | --- | --- |
| Client-side validation | Validate user input on the client-side | Use for immediate feedback and basic validation | Rely solely on client-side validation for security |
| Server-side validation | Validate user input on the server-side | Use for security-critical validation and data consistency | Use for client-side validation only |
| Dual validation | Validate user input on both client-side and server-side | Use for robust validation and security | Use for simple applications with minimal validation requirements |

§ 15.2 CODE ORGANIZATION

| Pattern | Description | ✅ DO | ❌ DON'T |
| --- | --- | --- | --- |
| Separate validation logic | Keep validation logic separate from business logic | Use for maintainability and reusability | Mix validation logic with business logic |
| Use validation libraries | Use established validation libraries like Zod or Joi | Use for consistency and community support | Roll your own validation library |

§ 16. COMMON PITFALLS AND TROUBLESHOOTING

§ 16.1 COMMON PITFALLS

* Not validating user input on the server-side
* Not handling validation errors correctly
* Not using established validation libraries
* Not testing validation logic thoroughly

§ 16.2 TROUBLESHOOTING

* Check the validation library documentation for common issues and solutions
* Use debugging tools like console logs or debuggers to identify issues
* Test validation logic thoroughly to catch errors early
* Use code reviews and pair programming to catch errors and improve code quality

§ 17. MIGRATION AND UPGRADE PATTERNS

§ 17.1 UPGRADING VALIDATION LIBRARIES

* Research the new library version and its changes
* Update the library version in your project
* Test your application thoroughly to catch any breaking changes
* Update your code to use new features and best practices

§ 17.2 MIGRATING TO A NEW VALIDATION STRATEGY

* Research the new validation strategy and its benefits
* Plan the migration and create a roadmap
* Update your code to use the new validation strategy
* Test your application thoroughly to catch any issues

Catalogo Tecnico Completo: Form Validation & Handling in Next.js 14
1. Zod Schema Patterns
1.1 Registration Form
typescript
Copia
Scarica
import { z } from 'zod';

export const registrationSchema = z.object({
  email: z.string()
    .min(1, { message: 'Email obbligatoria' })
    .email({ message: 'Formato email non valido' })
    .max(100, { message: 'Email troppo lunga' }),
  
  password: z.string()
    .min(8, { message: 'Password minima 8 caratteri' })
    .max(50, { message: 'Password troppo lunga' })
    .regex(/[A-Z]/, { message: 'Almeno una maiuscola' })
    .regex(/[0-9]/, { message: 'Almeno un numero' })
    .regex(/[^A-Za-z0-9]/, { message: 'Almeno un carattere speciale' }),
  
  confirmPassword: z.string()
    .min(1, { message: 'Conferma password obbligatoria' }),
  
  firstName: z.string()
    .min(1, { message: 'Nome obbligatorio' })
    .max(50, { message: 'Nome troppo lungo' })
    .regex(/^[a-zA-ZÀ-ÿ\s']+$/, { message: 'Nome non valido' }),
  
  lastName: z.string()
    .min(1, { message: 'Cognome obbligatorio' })
    .max(50, { message: 'Cognome troppo lungo' })
    .regex(/^[a-zA-ZÀ-ÿ\s']+$/, { message: 'Cognome non valido' }),
  
  acceptTerms: z.literal(true, {
    errorMap: () => ({ message: 'Devi accettare i termini e condizioni' })
  }),
  
  marketingOptIn: z.boolean().default(false),
  
  userType: z.enum(['personal', 'business', 'educator'], {
    errorMap: () => ({ message: 'Tipo utente non valido' })
  }).default('personal')
}).refine((data) => data.password === data.confirmPassword, {
  message: 'Le password non coincidono',
  path: ['confirmPassword']
});
1.2 Address Form
typescript
Copia
Scarica
export const addressSchema = z.object({
  label: z.string()
    .min(1, { message: 'Etichetta obbligatoria' })
    .max(30, { message: 'Etichetta troppo lunga' }),
  
  street: z.string()
    .min(1, { message: 'Via obbligatoria' })
    .max(100, { message: 'Via troppo lunga' }),
  
  streetNumber: z.string()
    .min(1, { message: 'Numero civico obbligatorio' })
    .max(10, { message: 'Numero civico troppo lungo' }),
  
  apartment: z.string().optional(),
  
  city: z.string()
    .min(1, { message: 'Città obbligatoria' })
    .max(50, { message: 'Città troppo lunga' }),
  
  province: z.string()
    .length(2, { message: 'Provincia: 2 lettere (es: RM)' })
    .toUpperCase()
    .regex(/^[A-Z]{2}$/, { message: 'Formato provincia non valido' }),
  
  postalCode: z.string()
    .length(5, { message: 'CAP: 5 cifre' })
    .regex(/^\d{5}$/, { message: 'CAP non valido' }),
  
  country: z.enum(['IT', 'DE', 'FR', 'ES', 'UK'], {
    errorMap: () => ({ message: 'Paese non supportato' })
  }).default('IT'),
  
  isDefault: z.boolean().default(false),
  
  phone: z.string()
    .min(1, { message: 'Telefono obbligatorio' })
    .regex(/^[+]?[0-9\s\-\(\)]{6,20}$/, { message: 'Telefono non valido' })
});
1.3 Product Form
typescript
Copia
Scarica
export const productSchema = z.object({
  name: z.string()
    .min(3, { message: 'Nome prodotto minimo 3 caratteri' })
    .max(200, { message: 'Nome troppo lungo' }),
  
  sku: z.string()
    .min(1, { message: 'SKU obbligatorio' })
    .max(50, { message: 'SKU troppo lungo' })
    .regex(/^[A-Z0-9\-_]+$/, { message: 'SKU: solo maiuscole, numeri, - e _' }),
  
  description: z.string()
    .min(10, { message: 'Descrizione minima 10 caratteri' })
    .max(5000, { message: 'Descrizione troppo lunga' }),
  
  categoryId: z.string().uuid({ message: 'Categoria non valida' }),
  
  price: z.number()
    .positive({ message: 'Prezzo deve essere positivo' })
    .max(100000, { message: 'Prezzo massimo 100.000' })
    .multipleOf(0.01, { message: 'Massimo 2 decimali' }),
  
  compareAtPrice: z.number()
    .positive()
    .max(100000)
    .multipleOf(0.01)
    .optional()
    .nullable(),
  
  costPerItem: z.number()
    .positive()
    .max(50000)
    .multipleOf(0.01)
    .optional(),
  
  quantity: z.number()
    .int({ message: 'Quantità deve essere intera' })
    .nonnegative({ message: 'Quantità non può essere negativa' })
    .max(1000000, { message: 'Quantità troppo alta' }),
  
  weight: z.number()
    .positive()
    .max(1000)
    .optional(),
  
  weightUnit: z.enum(['g', 'kg', 'lb', 'oz']).default('g'),
  
  tags: z.array(z.string().max(20))
    .max(10, { message: 'Massimo 10 tag' }),
  
  seoTitle: z.string().max(60, { message: 'SEO title max 60 caratteri' }).optional(),
  seoDescription: z.string().max(160, { message: 'SEO description max 160 caratteri' }).optional(),
  
  variants: z.array(z.object({
    name: z.string().min(1, { message: 'Nome variante obbligatorio' }),
    price: z.number().positive(),
    sku: z.string().optional(),
    quantity: z.number().int().nonnegative()
  })).optional()
}).refine(data => !data.compareAtPrice || data.compareAtPrice > data.price, {
  message: 'Prezzo di confronto deve essere maggiore del prezzo',
  path: ['compareAtPrice']
});
2. React Hook Form + Zod Integration
2.1 Setup Base con TypeScript Strict
typescript
Copia
Scarica
// app/components/form/BaseForm.tsx
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/ui/button';
import { Form, FormControl, FormDescription, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';

// Schema di esempio
const baseFormSchema = z.object({
  username: z.string().min(3, { message: 'Minimo 3 caratteri' }),
  email: z.string().email({ message: 'Email non valida' }),
  role: z.enum(['user', 'admin', 'moderator'])
});

type BaseFormValues = z.infer<typeof baseFormSchema>;

interface BaseFormProps {
  defaultValues?: Partial<BaseFormValues>;
  onSubmit: (data: BaseFormValues) => Promise<void>;
}

export function BaseForm({ defaultValues, onSubmit }: BaseFormProps) {
  const form = useForm<BaseFormValues>({
    resolver: zodResolver(baseFormSchema),
    defaultValues: {
      username: '',
      email: '',
      role: 'user',
      ...defaultValues,
    },
    mode: 'onChange', // Valida ad ogni cambiamento
    reValidateMode: 'onChange',
  });

  const {
    formState: { errors, isSubmitting, isValid, isDirty },
    reset,
    trigger,
  } = form;

  const handleSubmit = async (data: BaseFormValues) => {
    try {
      await onSubmit(data);
      reset(); // Reset form dopo submit successo
    } catch (error) {
      // Gestione errori server-side
      console.error('Submit error:', error);
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(handleSubmit)} className="space-y-6">
        {/* Campo Input */}
        <FormField
          control={form.control}
          name="username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Username *</FormLabel>
              <FormControl>
                <Input 
                  placeholder="Inserisci username" 
                  {...field} 
                  disabled={isSubmitting}
                  aria-invalid={!!errors.username}
                  className={errors.username ? 'border-destructive' : ''}
                />
              </FormControl>
              <FormDescription>
                Minimo 3 caratteri, massimo 20
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Campo Email */}
        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email *</FormLabel>
              <FormControl>
                <Input 
                  type="email" 
                  placeholder="esempio@email.com" 
                  {...field} 
                  disabled={isSubmitting}
                  aria-invalid={!!errors.email}
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Campo Select */}
        <FormField
          control={form.control}
          name="role"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Ruolo *</FormLabel>
              <Select
                onValueChange={field.onChange}
                defaultValue={field.value}
                disabled={isSubmitting}
              >
                <FormControl>
                  <SelectTrigger>
                    <SelectValue placeholder="Seleziona un ruolo" />
                  </SelectTrigger>
                </FormControl>
                <SelectContent>
                  <SelectItem value="user">Utente</SelectItem>
                  <SelectItem value="moderator">Moderatore</SelectItem>
                  <SelectItem value="admin">Amministratore</SelectItem>
                </SelectContent>
              </Select>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Azioni */}
        <div className="flex gap-4 pt-4">
          <Button 
            type="submit" 
            disabled={isSubmitting || !isValid}
            loading={isSubmitting}
          >
            Salva
          </Button>
          
          <Button 
            type="button" 
            variant="outline"
            onClick={() => reset()}
            disabled={isSubmitting || !isDirty}
          >
            Annulla
          </Button>
          
          <Button 
            type="button" 
            variant="ghost"
            onClick={async () => {
              const isValid = await trigger();
              if (isValid) {
                console.log('Validazione OK');
              }
            }}
          >
            Valida
          </Button>
        </div>
      </form>
    </Form>
  );
}
2.2 Form con Controller Personalizzati
typescript
Copia
Scarica
// app/components/form/AdvancedForm.tsx
'use client';

import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { Calendar } from '@/components/ui/calendar';
import { Textarea } from '@/components/ui/textarea';
import { Switch } from '@/components/ui/switch';
import { Slider } from '@/components/ui/slider';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { Badge } from '@/components/ui/badge';
import { CalendarIcon } from 'lucide-react';
import { format } from 'date-fns';
import { it } from 'date-fns/locale';

const advancedSchema = z.object({
  title: z.string().min(5).max(100),
  description: z.string().min(10).max(500),
  priority: z.enum(['low', 'medium', 'high', 'critical']),
  dateRange: z.object({
    from: z.date(),
    to: z.date().optional(),
  }),
  notify: z.boolean().default(true),
  notificationLevel: z.number().min(1).max(5).default(3),
  tags: z.array(z.string()).max(5),
});

export function AdvancedForm() {
  const form = useForm({
    resolver: zodResolver(advancedSchema),
    defaultValues: {
      title: '',
      description: '',
      priority: 'medium',
      dateRange: { from: new Date() },
      notify: true,
      notificationLevel: 3,
      tags: [],
    },
  });

  return (
    <Form {...form}>
      <form className="space-y-6">
        {/* Date Picker con Controller */}
        <Controller
          name="dateRange"
          control={form.control}
          render={({ field }) => (
            <FormItem className="flex flex-col">
              <FormLabel>Periodo</FormLabel>
              <div className="flex items-center gap-2">
                <Calendar
                  mode="range"
                  selected={{
                    from: field.value.from,
                    to: field.value.to,
                  }}
                  onSelect={(range) => {
                    field.onChange({
                      from: range?.from,
                      to: range?.to,
                    });
                  }}
                  locale={it}
                  className="rounded-md border"
                />
                <div className="text-sm text-muted-foreground">
                  {field.value.from && (
                    <>
                      Dal: {format(field.value.from, 'dd/MM/yyyy', { locale: it })}
                      {field.value.to && (
                        <> al: {format(field.value.to, 'dd/MM/yyyy', { locale: it })}</>
                      )}
                    </>
                  )}
                </div>
              </div>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Radio Group */}
        <Controller
          name="priority"
          control={form.control}
          render={({ field }) => (
            <FormItem className="space-y-3">
              <FormLabel>Priorità</FormLabel>
              <FormControl>
                <RadioGroup
                  onValueChange={field.onChange}
                  defaultValue={field.value}
                  className="flex flex-col space-y-1"
                >
                  <FormItem className="flex items-center space-x-3 space-y-0">
                    <FormControl>
                      <RadioGroupItem value="low" />
                    </FormControl>
                    <FormLabel className="font-normal">Bassa</FormLabel>
                  </FormItem>
                  <FormItem className="flex items-center space-x-3 space-y-0">
                    <FormControl>
                      <RadioGroupItem value="medium" />
                    </FormControl>
                    <FormLabel className="font-normal">Media</FormLabel>
                  </FormItem>
                  <FormItem className="flex items-center space-x-3 space-y-0">
                    <FormControl>
                      <RadioGroupItem value="high" />
                    </FormControl>
                    <FormLabel className="font-normal">Alta</FormLabel>
                  </FormItem>
                  <FormItem className="flex items-center space-x-3 space-y-0">
                    <FormControl>
                      <RadioGroupItem value="critical" />
                    </FormControl>
                    <FormLabel className="font-normal">Critica</FormLabel>
                  </FormItem>
                </RadioGroup>
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Slider */}
        <Controller
          name="notificationLevel"
          control={form.control}
          render={({ field }) => (
            <FormItem>
              <FormLabel>Livello notifiche: {field.value}/5</FormLabel>
              <FormControl>
                <Slider
                  min={1}
                  max={5}
                  step={1}
                  value={[field.value]}
                  onValueChange={([value]) => field.onChange(value)}
                  className="w-60"
                />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        {/* Switch */}
        <Controller
          name="notify"
          control={form.control}
          render={({ field }) => (
            <FormItem className="flex flex-row items-center justify-between rounded-lg border p-4">
              <div className="space-y-0.5">
                <FormLabel className="text-base">Notifiche attive</FormLabel>
                <FormDescription>
                  Ricevi notifiche per aggiornamenti
                </FormDescription>
              </div>
              <FormControl>
                <Switch
                  checked={field.value}
                  onCheckedChange={field.onChange}
                />
              </FormControl>
            </FormItem>
          )}
        />
      </form>
    </Form>
  );
}
3. Server Actions Form Pattern
3.1 Server Action con Zod Validation
typescript
Copia
Scarica
// app/actions/user-actions.ts
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';
import { createClient } from '@/lib/supabase/server';
import { cookies } from 'next/headers';

// Schema per server action
const createUserSchema = z.object({
  email: z.string().email(),
  firstName: z.string().min(1),
  lastName: z.string().min(1),
  role: z.enum(['user', 'admin', 'moderator']),
  departmentId: z.string().uuid().optional(),
});

export type CreateUserFormState = {
  errors?: {
    email?: string[];
    firstName?: string[];
    lastName?: string[];
    role?: string[];
    departmentId?: string[];
    _form?: string[];
  };
  success?: boolean;
  message?: string;
};

export async function createUser(
  prevState: CreateUserFormState,
  formData: FormData
): Promise<CreateUserFormState> {
  // Validazione client-side fallback
  const rawData = {
    email: formData.get('email'),
    firstName: formData.get('firstName'),
    lastName: formData.get('lastName'),
    role: formData.get('role'),
    departmentId: formData.get('departmentId'),
  };

  // Validazione con Zod
  const result = createUserSchema.safeParse(rawData);

  if (!result.success) {
    return {
      errors: result.error.flatten().fieldErrors,
      success: false,
      message: 'Correggi gli errori nel form',
    };
  }

  const { email, firstName, lastName, role, departmentId } = result.data;

  try {
    const supabase = await createClient();
    
    // Verifica esistenza email
    const { data: existingUser } = await supabase
      .from('users')
      .select('id')
      .eq('email', email)
      .single();

    if (existingUser) {
      return {
        errors: {
          email: ['Email già registrata'],
        },
        success: false,
        message: 'Utente già esistente',
      };
    }

    // Creazione utente
    const { error } = await supabase
      .from('users')
      .insert({
        email,
        first_name: firstName,
        last_name: lastName,
        role,
        department_id: departmentId,
        created_at: new Date().toISOString(),
      });

    if (error) throw error;

    // Revalidate e redirect
    revalidatePath('/users');
    revalidatePath('/dashboard');

    return {
      success: true,
      message: 'Utente creato con successo',
    };

  } catch (error) {
    console.error('Error creating user:', error);
    
    return {
      errors: {
        _form: ['Errore durante la creazione: ' + (error instanceof Error ? error.message : 'Errore sconosciuto')],
      },
      success: false,
      message: 'Errore durante la creazione',
    };
  }
}

// Action per ottenere CSRF token
export async function getCsrfToken() {
  const cookieStore = await cookies();
  return {
    csrfToken: Math.random().toString(36).substring(2),
  };
}
3.2 Component Client con useFormState
typescript
Copia
Scarica
// app/components/forms/UserForm.tsx
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { createUser, CreateUserFormState } from '@/app/actions/user-actions';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Button } from '@/components/ui/button';
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select';
import { Alert, AlertDescription } from '@/components/ui/alert';
import { Loader2 } from 'lucide-react';

// Componente per button con stato pending
function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <Button type="submit" disabled={pending} className="w-full">
      {pending ? (
        <>
          <Loader2 className="mr-2 h-4 w-4 animate-spin" />
          Creazione...
        </>
      ) : (
        'Crea Utente'
      )}
    </Button>
  );
}

export function UserForm({ departments }: { departments: Array<{ id: string; name: string }> }) {
  const [state, formAction] = useFormState<CreateUserFormState, FormData>(
    createUser,
    { errors: {}, success: false }
  );

  return (
    <form action={formAction} className="space-y-6">
      {/* Messaggi globali */}
      {state.success && (
        <Alert variant="default" className="bg-green-50 border-green-200">
          <AlertDescription className="text-green-800">
            ✅ {state.message}
          </AlertDescription>
        </Alert>
      )}

      {state.errors?._form && (
        <Alert variant="destructive">
          <AlertDescription>
            ❌ {state.errors._form.join(', ')}
          </AlertDescription>
        </Alert>
      )}

      {/* Campo Email */}
      <div className="space-y-2">
        <Label htmlFor="email">Email *</Label>
        <Input
          id="email"
          name="email"
          type="email"
          placeholder="mario.rossi@email.com"
          required
          aria-describedby="email-error"
          className={state.errors?.email ? 'border-destructive' : ''}
        />
        {state.errors?.email && (
          <p id="email-error" className="text-sm text-destructive">
            {state.errors.email.join(', ')}
          </p>
        )}
      </div>

      {/* Nome e Cognome */}
      <div className="grid grid-cols-2 gap-4">
        <div className="space-y-2">
          <Label htmlFor="firstName">Nome *</Label>
          <Input
            id="firstName"
            name="firstName"
            placeholder="Mario"
            required
            aria-describedby="firstName-error"
            className={state.errors?.firstName ? 'border-destructive' : ''}
          />
          {state.errors?.firstName && (
            <p id="firstName-error" className="text-sm text-destructive">
              {state.errors.firstName.join(', ')}
            </p>
          )}
        </div>

        <div className="space-y-2">
          <Label htmlFor="lastName">Cognome *</Label>
          <Input
            id="lastName"
            name="lastName"
            placeholder="Rossi"
            required
            aria-describedby="lastName-error"
            className={state.errors?.lastName ? 'border-destructive' : ''}
          />
          {state.errors?.lastName && (
            <p id="lastName-error" className="text-sm text-destructive">
              {state.errors.lastName.join(', ')}
            </p>
          )}
        </div>
      </div>

      {/* Ruolo */}
      <div className="space-y-2">
        <Label htmlFor="role">Ruolo *</Label>
        <Select name="role" defaultValue="user">
          <SelectTrigger className={state.errors?.role ? 'border-destructive' : ''}>
            <SelectValue placeholder="Seleziona ruolo" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="user">Utente</SelectItem>
            <SelectItem value="moderator">Moderatore</SelectItem>
            <SelectItem value="admin">Amministratore</SelectItem>
          </SelectContent>
        </Select>
        {state.errors?.role && (
          <p className="text-sm text-destructive">
            {state.errors.role.join(', ')}
          </p>
        )}
      </div>

      {/* Dipartimento */}
      <div className="space-y-2">
        <Label htmlFor="departmentId">Dipartimento</Label>
        <Select name="departmentId">
          <SelectTrigger>
            <SelectValue placeholder="Seleziona dipartimento" />
          </SelectTrigger>
          <SelectContent>
            {departments.map((dept) => (
              <SelectItem key={dept.id} value={dept.id}>
                {dept.name}
              </SelectItem>
            ))}
          </SelectContent>
        </Select>
        {state.errors?.departmentId && (
          <p className="text-sm text-destructive">
            {state.errors.departmentId.join(', ')}
          </p>
        )}
      </div>

      {/* Submit Button */}
      <SubmitButton />
    </form>
  );
}
4. Multi-Step Form
4.1 Wizard Form State Management
typescript
Copia
Scarica
// app/components/forms/MultiStepForm.tsx
'use client';

import { useState } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Progress } from '@/components/ui/progress';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { ChevronLeft, ChevronRight, Save } from 'lucide-react';
import { PersonalInfoStep } from './steps/PersonalInfoStep';
import { AddressStep } from './steps/AddressStep';
import { PreferencesStep } from './steps/PreferencesStep';
import { ReviewStep } from './steps/ReviewStep';

// Schemi per ogni step
const personalInfoSchema = z.object({
  firstName: z.string().min(2, { message: 'Nome minimo 2 caratteri' }),
  lastName: z.string().min(2, { message: 'Cognome minimo 2 caratteri' }),
  email: z.string().email({ message: 'Email non valida' }),
  phone: z.string().regex(/^[+]?[0-9\s\-\(\)]{6,20}$/, { message: 'Telefono non valido' }),
});

const addressSchema = z.object({
  street: z.string().min(1, { message: 'Via obbligatoria' }),
  city: z.string().min(1, { message: 'Città obbligatoria' }),
  postalCode: z.string().length(5, { message: 'CAP: 5 cifre' }),
  province: z.string().length(2, { message: 'Provincia: 2 lettere' }),
  country: z.string().default('IT'),
});

const preferencesSchema = z.object({
  newsletter: z.boolean().default(true),
  notifications: z.boolean().default(true),
  language: z.enum(['it', 'en', 'de', 'fr']).default('it'),
  timezone: z.string().default('Europe/Rome'),
});

// Schema completo
const completeSchema = personalInfoSchema.merge(addressSchema).merge(preferencesSchema);

type FormData = z.infer<typeof completeSchema>;

const STEPS = [
  { id: 'personal', title: 'Informazioni Personali', schema: personalInfoSchema },
  { id: 'address', title: 'Indirizzo', schema: addressSchema },
  { id: 'preferences', title: 'Preferenze', schema: preferencesSchema },
  { id: 'review', title: 'Riepilogo', schema: completeSchema },
] as const;

export function MultiStepForm() {
  const [currentStep, setCurrentStep] = useState(0);
  const [formData, setFormData] = useState<Partial<FormData>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isComplete, setIsComplete] = useState(false);

  const currentStepConfig = STEPS[currentStep];
  const isLastStep = currentStep === STEPS.length - 1;
  const isFirstStep = currentStep === 0;

  // Form hook per lo step corrente
  const form = useForm({
    resolver: zodResolver(currentStepConfig.schema),
    defaultValues: formData as any,
    mode: 'onChange',
  });

  const progress = ((currentStep + 1) / STEPS.length) * 100;

  const handleNext = async () => {
    const isValid = await form.trigger();
    if (!isValid) return;

    // Salva dati step corrente
    const currentValues = form.getValues();
    setFormData(prev => ({ ...prev, ...currentValues }));

    if (isLastStep) {
      await handleSubmit();
    } else {
      setCurrentStep(prev => prev + 1);
      // Reset form con nuovi valori per il prossimo step
      form.reset({ ...formData, ...currentValues });
    }
  };

  const handleBack = () => {
    if (!isFirstStep) {
      // Salva dati step corrente prima di tornare indietro
      const currentValues = form.getValues();
      setFormData(prev => ({ ...prev, ...currentValues }));
      
      setCurrentStep(prev => prev - 1);
      form.reset(formData as any);
    }
  };

  const handleSubmit = async () => {
    setIsSubmitting(true);
    try {
      const finalData = { ...formData, ...form.getValues() };
      const validatedData = completeSchema.parse(finalData);

      // Simula submit API
      await fetch('/api/register', {
        method: 'POST',
        body: JSON.stringify(validatedData),
      });

      setIsComplete(true);
    } catch (error) {
      console.error('Submit error:', error);
    } finally {
      setIsSubmitting(false);
    }
  };

  if (isComplete) {
    return (
      <Card className="w-full max-w-2xl mx-auto">
        <CardHeader>
          <CardTitle className="text-green-600">✅ Registrazione Completata!</CardTitle>
          <CardDescription>
            Grazie per esserti registrato. Ti abbiamo inviato una email di conferma.
          </CardDescription>
        </CardHeader>
      </Card>
    );
  }

  return (
    <Card className="w-full max-w-2xl mx-auto">
      <CardHeader>
        <div className="space-y-2">
          <CardTitle>Registrazione</CardTitle>
          <CardDescription>
            Step {currentStep + 1} di {STEPS.length}: {currentStepConfig.title}
          </CardDescription>
        </div>
        
        <Progress value={progress} className="h-2" />
      </CardHeader>

      <CardContent>
        <form onSubmit={form.handleSubmit(handleNext)}>
          {/* Step rendering condizionale */}
          {currentStep === 0 && <PersonalInfoStep form={form} />}
          {currentStep === 1 && <AddressStep form={form} />}
          {currentStep === 2 && <PreferencesStep form={form} />}
          {currentStep === 3 && <ReviewStep data={formData as FormData} />}

          {/* Navigation */}
          <div className="flex justify-between pt-6">
            <Button
              type="button"
              variant="outline"
              onClick={handleBack}
              disabled={isFirstStep || isSubmitting}
            >
              <ChevronLeft className="mr-2 h-4 w-4" />
              Indietro
            </Button>

            <Button
              type="submit"
              disabled={isSubmitting}
              loading={isSubmitting}
            >
              {isLastStep ? (
                <>
                  <Save className="mr-2 h-4 w-4" />
                  Completa Registrazione
                </>
              ) : (
                <>
                  Avanti
                  <ChevronRight className="ml-2 h-4 w-4" />
                </>
              )}
            </Button>
          </div>
        </form>

        {/* Step indicator */}
        <div className="flex justify-center gap-2 mt-6">
          {STEPS.map((step, index) => (
            <button
              key={step.id}
              type="button"
              onClick={() => {
                if (index < currentStep) {
                  setCurrentStep(index);
                  form.reset(formData as any);
                }
              }}
              className={`h-2 w-2 rounded-full transition-all ${
                index === currentStep
                  ? 'bg-primary w-8'
                  : index < currentStep
                  ? 'bg-primary/50 cursor-pointer'
                  : 'bg-muted'
              }`}
              aria-label={`Vai allo step ${index + 1}: ${step.title}`}
            />
          ))}
        </div>
      </CardContent>
    </Card>
  );
}
4.2 Step Component Esempio
typescript
Copia
Scarica
// app/components/forms/steps/PersonalInfoStep.tsx
'use client';

import { UseFormReturn } from 'react-hook-form';
import { FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form';
import { Input } from '@/components/ui/input';
import { PersonalInfoSchema } from '@/lib/schemas';

interface PersonalInfoStepProps {
  form: UseFormReturn<z.infer<typeof PersonalInfoSchema>>;
}

export function PersonalInfoStep({ form }: PersonalInfoStepProps) {
  return (
    <div className="space-y-6">
      <div className="grid grid-cols-2 gap-4">
        <FormField
          control={form.control}
          name="firstName"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Nome *</FormLabel>
              <FormControl>
                <Input placeholder="Mario" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="lastName"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Cognome *</FormLabel>
              <FormControl>
                <Input placeholder="Rossi" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
      </div>

      <FormField
        control={form.control}
        name="email"
        render={({ field }) => (
          <FormItem>
            <FormLabel>Email *</FormLabel>
            <FormControl>
              <Input type="email" placeholder="mario.rossi@email.com" {...field} />
            </FormControl>
            <FormMessage />
          </FormItem>
        )}
      />

      <FormField
        control={form.control}
        name="phone"
        render={({ field }) => (
          <FormItem>
            <FormLabel>Telefono *</FormLabel>
            <FormControl>
              <Input placeholder="+39 123 456 7890" {...field} />
            </FormControl>
            <FormMessage />
          </FormItem>
        )}
      />
    </div>
  );
}
5. Dynamic Form Fields
5.1 useFieldArray per Liste Dinamiche
typescript
Copia
Scarica
// app/components/forms/DynamicListForm.tsx
'use client';

import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { Button } from '@/components/ui/button';
import {