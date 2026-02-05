# CATALOGO FORM VALIDATION v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Zod validation, React Hook Form, validation patterns, error handling, UX
> **Sezioni**: 1-10

---

## 1. VALIDATION LIBRARY COMPARISON

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

## 2. ZOD FUNDAMENTALS

### 2.1 Primitive Types

```typescript
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
```

### 2.2 Object Schemas

```typescript
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
```

### 2.3 Advanced Patterns

```typescript
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
```


---

## 3. REACT HOOK FORM INTEGRATION

### 3.1 Basic Setup

```typescript
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
```

### 3.2 Form Field Component

```typescript
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
```

### 3.3 Complete Form Example

```typescript
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
```

---

## 4. COMMON VALIDATION PATTERNS

### 4.1 Pattern Reference Table

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

### 4.2 Reusable Schema Library

```typescript
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
```



---

## 5. ERROR DISPLAY PATTERNS

### 5.1 Error Message Component

```typescript
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
```

### 5.2 Form Error Summary

```typescript
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
```

### 5.3 Inline vs Summary Display

| Strategy | Best For | UX Impact |
|----------|----------|-----------|
| Inline only | Simple forms (<5 fields) | Immediate feedback |
| Summary only | Long forms, wizard steps | Overview before submit |
| Inline + Summary | Complex forms | Best of both worlds |
| Toast notification | Background validation | Non-blocking |

---

## 6. CONDITIONAL VALIDATION

### 6.1 Dependent Field Validation

```typescript
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
```

### 6.2 Dynamic Schema Selection

```typescript
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
```

---

## 7. MULTI-STEP FORM VALIDATION

### 7.1 Step Schema Pattern

```typescript
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
```

### 7.2 Multi-Step Form Hook

```typescript
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
```

---

## 8. SERVER-SIDE VALIDATION

### 8.1 API Route Validation

```typescript
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
```

### 8.2 Server Action Validation

```typescript
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
```



---

## 9. FORM ACCESSIBILITY

### 9.1 A11y Requirements Table

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

### 9.2 Accessible Input Component

```typescript
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
```

### 9.3 Focus Error Field on Submit

```typescript
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
```

---

## 10. FORM VALIDATION CHECKLIST

```
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
```
