# CATALOGO-FORM-VALIDATION

Catalogo Avanzato: Form Validation per Next.js 14 + React Hook Form + Zod
§ REACT HOOK FORM ADVANCED
useFieldArray per array dinamici
typescript
import { useFieldArray, useForm } from "react-hook-form";
import { z } from "zod";
import { zodResolver } from "@hookform/resolvers/zod";

const schema = z.object({
  contacts: z.array(
    z.object({
      name: z.string().min(2, "Nome minimo 2 caratteri"),
      email: z.string().email("Email non valida"),
      phone: z.string().optional(),
    })
  ).min(1, "Aggiungi almeno un contatto"),
});

type FormValues = z.infer<typeof schema>;

export function DynamicArrayForm() {
  const {
    control,
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormValues>({
    resolver: zodResolver(schema),
    defaultValues: {
      contacts: [{ name: "", email: "" }],
    },
  });

  const { fields, append, remove, swap, move } = useFieldArray({
    control,
    name: "contacts",
  });

  const onSubmit = (data: FormValues) => {
    console.log(data);
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {fields.map((field, index) => (
        <div key={field.id} className="dynamic-field-group">
          <input
            {...register(`contacts.${index}.name`)}
            placeholder="Nome"
          />
          {errors.contacts?.[index]?.name && (
            <p>{errors.contacts[index].name.message}</p>
          )}

          <input
            {...register(`contacts.${index}.email`)}
            placeholder="Email"
            type="email"
          />
          {errors.contacts?.[index]?.email && (
            <p>{errors.contacts[index].email.message}</p>
          )}

          <button type="button" onClick={() => remove(index)}>
            Rimuovi
          </button>

          {index > 0 && (
            <button type="button" onClick={() => move(index, index - 1)}>
              Su
            </button>
          )}
        </div>
      ))}

      <button
        type="button"
        onClick={() => append({ name: "", email: "" })}
      >
        Aggiungi contatto
      </button>

      <button type="submit">Invia</button>
    </form>
  );
}
useWatch per reactive fields
typescript
import { useWatch, useForm } from "react-hook-form";

export function ReactiveForm() {
  const { control, register, setValue } = useForm({
    defaultValues: {
      subscriptionType: "basic",
      price: 10,
      features: ["feature1"],
      discountCode: "",
      finalPrice: 10,
    },
  });

  const subscriptionType = useWatch({
    control,
    name: "subscriptionType",
  });

  const discountCode = useWatch({
    control,
    name: "discountCode",
  });

  // Calcolo reattivo del prezzo
  React.useEffect(() => {
    let price = subscriptionType === "premium" ? 25 : 10;
    
    if (discountCode === "SAVE20") {
      price = price * 0.8;
    }
    
    setValue("finalPrice", price);
  }, [subscriptionType, discountCode, setValue]);

  return (
    <form>
      <select {...register("subscriptionType")}>
        <option value="basic">Basic (€10)</option>
        <option value="premium">Premium (€25)</option>
      </select>

      <input
        {...register("discountCode")}
        placeholder="Codice sconto"
      />

      <div>
        <strong>Prezzo finale: €{finalPrice}</strong>
      </div>
    </form>
  );
}
useFormContext per nested forms
typescript
import { createContext, useContext } from "react";
import {
  FormProvider,
  useFormContext,
  useForm,
} from "react-hook-form";

const FormContext = createContext(null);

export function ComplexForm() {
  const methods = useForm();
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(console.log)}>
        <PersonalSection />
        <AddressSection />
        <PreferencesSection />
        <button type="submit">Salva</button>
      </form>
    </FormProvider>
  );
}

function PersonalSection() {
  const { register, formState: { errors } } = useFormContext();
  
  return (
    <fieldset>
      <legend>Informazioni personali</legend>
      <input {...register("firstName")} />
      {errors.firstName && <span>{errors.firstName.message}</span>}
      
      <input {...register("lastName")} />
      {errors.lastName && <span>{errors.lastName.message}</span>}
    </fieldset>
  );
}

function AddressSection() {
  const { register, watch } = useFormContext();
  const country = watch("country");
  
  return (
    <fieldset>
      <legend>Indirizzo</legend>
      <select {...register("country")}>
        <option value="IT">Italia</option>
        <option value="US">USA</option>
      </select>
      
      <input {...register("address")} />
      
      {country === "IT" && (
        <input {...register("province")} placeholder="Provincia" />
      )}
    </fieldset>
  );
}

function PreferencesSection() {
  const { register } = useFormContext();
  
  return (
    <fieldset>
      <legend>Preferenze</legend>
      <label>
        <input type="checkbox" {...register("newsletter")} />
        Newsletter
      </label>
    </fieldset>
  );
}
Form state persistence
typescript
import { useEffect } from "react";
import { useForm } from "react-hook-form";

const STORAGE_KEY = "form_draft";

export function PersistentForm() {
  const {
    register,
    handleSubmit,
    watch,
    setValue,
    reset,
  } = useForm();

  // Salva automaticamente ad ogni cambiamento
  const allValues = watch();
  useEffect(() => {
    const timeoutId = setTimeout(() => {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(allValues));
    }, 500);
    
    return () => clearTimeout(timeoutId);
  }, [allValues]);

  // Carica al mount
  useEffect(() => {
    const saved = localStorage.getItem(STORAGE_KEY);
    if (saved) {
      try {
        const parsed = JSON.parse(saved);
        reset(parsed);
      } catch (error) {
        console.error("Errore nel caricamento:", error);
      }
    }
  }, [reset]);

  const clearDraft = () => {
    localStorage.removeItem(STORAGE_KEY);
    reset();
  };

  const onSubmit = async (data: any) => {
    // Invia i dati
    await fetch("/api/submit", {
      method: "POST",
      body: JSON.stringify(data),
    });
    
    // Pulisci il draft dopo l'invio
    clearDraft();
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <textarea
        {...register("content")}
        placeholder="Scrivi qui..."
        rows={5}
      />
      
      <div className="form-actions">
        <button type="submit">Invia</button>
        <button type="button" onClick={clearDraft}>
          Cancella bozza
        </button>
      </div>
    </form>
  );
}
Controlled vs uncontrolled
typescript
import { useState } from "react";
import { Controller, useForm } from "react-hook-form";

export function ControlledUncontrolledForm() {
  const { control, register, handleSubmit } = useForm();

  // Esempio Uncontrolled (semplice)
  const UncontrolledInput = () => (
    <input
      {...register("uncontrolledField")}
      placeholder="Uncontrolled"
    />
  );

  // Esempio Controlled (complessi)
  const ControlledInput = () => (
    <Controller
      name="controlledField"
      control={control}
      defaultValue=""
      render={({ field }) => (
        <input
          {...field}
          placeholder="Controlled"
          onChange={(e) => {
            // Logica custom
            const value = e.target.value.toUpperCase();
            field.onChange(value);
          }}
        />
      )}
    />
  );

  // Esempio con componenti di terze parti
  const DatePickerControlled = () => (
    <Controller
      name="date"
      control={control}
      render={({ field }) => (
        <ThirdPartyDatePicker
          selected={field.value}
          onChange={field.onChange}
          dateFormat="dd/MM/yyyy"
        />
      )}
    />
  );

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <UncontrolledInput />
      <ControlledInput />
      <DatePickerControlled />
      <button type="submit">Invia</button>
    </form>
  );
}
§ ZOD SCHEMAS AVANZATI
Refinements e superRefine
typescript
import { z } from "zod";

// Refinement per validazione singola
const passwordSchema = z.string()
  .min(8, "Password minimo 8 caratteri")
  .refine(
    (val) => /[A-Z]/.test(val),
    "Password deve contenere almeno una maiuscola"
  )
  .refine(
    (val) => /[0-9]/.test(val),
    "Password deve contenere almeno un numero"
  );

// Super refine per validazioni multiple con contesto
const registrationSchema = z.object({
  email: z.string().email(),
  password: passwordSchema,
  confirmPassword: z.string(),
  birthDate: z.string(),
  acceptTerms: z.boolean(),
})
.refine(
  (data) => data.password === data.confirmPassword,
  {
    message: "Le password non corrispondono",
    path: ["confirmPassword"],
  }
)
.superRefine((data, ctx) => {
  // Validazione età minima
  const birthDate = new Date(data.birthDate);
  const today = new Date();
  let age = today.getFullYear() - birthDate.getFullYear();
  const monthDiff = today.getMonth() - birthDate.getMonth();
  
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birthDate.getDate())) {
    age--;
  }
  
  if (age < 18) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Devi avere almeno 18 anni",
      path: ["birthDate"],
    });
  }

  // Validazione termini
  if (!data.acceptTerms) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Devi accettare i termini e condizioni",
      path: ["acceptTerms"],
    });
  }

  // Validazione business logic complessa
  if (data.email.endsWith(".test") && data.password.length < 12) {
    ctx.addIssue({
      code: z.ZodIssueCode.custom,
      message: "Account test richiedono password più lunghe",
      path: ["password"],
    });
  }
});

// Refine con async
const asyncEmailSchema = z.string().email().refine(
  async (email) => {
    const response = await fetch(`/api/check-email?email=${email}`);
    const data = await response.json();
    return !data.exists;
  },
  { message: "Email già registrata" }
);
Transform e preprocess
typescript
const advancedSchema = z.object({
  // Transform per normalizzazione dati
  phone: z.string()
    .transform((val) => val.replace(/\s+/g, ""))
    .refine((val) => /^[0-9+]+$/.test(val), "Telefono non valido"),

  // Preprocess per dati grezzi
  age: z.preprocess(
    (val) => {
      // Converte stringa in numero
      if (typeof val === "string") {
        const parsed = parseInt(val, 10);
        return isNaN(parsed) ? val : parsed;
      }
      return val;
    },
    z.number().min(0).max(120)
  ),

  // Date transformation
  dateString: z.string()
    .transform((str) => new Date(str))
    .refine((date) => !isNaN(date.getTime()), "Data non valida"),

  // Complex transformation chain
  price: z.string()
    .transform((val) => {
      // Rimuove simboli e converte in centesimi
      const cleaned = val.replace(/[€$,]/g, "");
      const floatValue = parseFloat(cleaned);
      return Math.round(floatValue * 100);
    })
    .refine((val) => val > 0, "Prezzo deve essere positivo"),

  // Conditional transformation
  discount: z.string()
    .transform((val, ctx) => {
      if (val.endsWith("%")) {
        const percent = parseFloat(val.slice(0, -1));
        if (percent > 100) {
          ctx.addIssue({
            code: z.ZodIssueCode.custom,
            message: "Sconto non può superare 100%",
          });
          return z.NEVER;
        }
        return { type: "percent", value: percent };
      }
      const amount = parseFloat(val);
      return { type: "amount", value: amount };
    }),
});
Discriminated unions
typescript
// Schema per pagamenti diversi
const paymentSchema = z.discriminatedUnion("method", [
  // Carta di credito
  z.object({
    method: z.literal("credit_card"),
    cardNumber: z.string()
      .length(16, "Numero carta richiede 16 cifre")
      .regex(/^[0-9]+$/, "Solo numeri"),
    expiration: z.string()
      .regex(/^(0[1-9]|1[0-2])\/([0-9]{2})$/, "Formato MM/AA"),
    cvv: z.string().length(3, "CVV richiede 3 cifre"),
    holder: z.string().min(2, "Nome titolare obbligatorio"),
  }),

  // PayPal
  z.object({
    method: z.literal("paypal"),
    email: z.string().email("Email PayPal non valida"),
    agreementId: z.string().optional(),
  }),

  // Bonifico
  z.object({
    method: z.literal("bank_transfer"),
    bankName: z.string().min(2, "Nome banca obbligatorio"),
    iban: z.string()
      .regex(/^IT[0-9]{2}[A-Z][0-9]{10}[0-9A-Z]{12}$/, "IBAN italiano non valido"),
    causal: z.string().max(140, "Causale max 140 caratteri"),
  }),

  // Contrassegno
  z.object({
    method: z.literal("cash_on_delivery"),
    maxAmount: z.number().max(1000, "Max €1000 per contrassegno"),
    instructions: z.string().optional(),
  }),
]);

// Schema per tipi utente
const userSchema = z.discriminatedUnion("role", [
  z.object({
    role: z.literal("admin"),
    adminLevel: z.enum(["basic", "super", "owner"]),
    permissions: z.array(z.string()),
    canDeleteUsers: z.boolean(),
  }),
  z.object({
    role: z.literal("moderator"),
    moderatedSections: z.array(z.string()),
    reportAccess: z.boolean(),
  }),
  z.object({
    role: z.literal("user"),
    profileCompleted: z.boolean(),
    subscription: z.enum(["free", "premium", "enterprise"]),
  }),
  z.object({
    role: z.literal("guest"),
    sessionExpiry: z.date(),
    limitedAccess: z.literal(true),
  }),
]);
Recursive schemas
typescript
import { z } from "zod";

// Schema per commenti nidificati
const commentSchema: z.ZodType<Comment> = z.lazy(() =>
  z.object({
    id: z.string(),
    text: z.string().min(1, "Commento non può essere vuoto"),
    author: z.string(),
    createdAt: z.date(),
    replies: z.array(commentSchema).default([]),
    depth: z.number().max(5, "Max 5 livelli di nidificazione"),
  })
);

// Schema per struttura ad albero
type TreeNode = {
  value: string;
  children: TreeNode[];
};

const treeSchema: z.ZodType<TreeNode> = z.object({
  value: z.string().min(1),
  children: z.array(z.lazy(() => treeSchema)).max(100, "Max 100 nodi"),
});

// Schema per categorie gerarchiche
const categorySchema = z.object({
  id: z.string(),
  name: z.string().min(2),
  slug: z.string().regex(/^[a-z0-9-]+$/),
  parentId: z.string().optional(),
  children: z.array(z.lazy(() => categorySchema)).optional(),
});

// Funzione helper per validazione ciclica
const createRecursiveSchema = <T extends z.ZodType<any>>(
  schemaFactory: (self: z.ZodType<any>) => T
): T => {
  const schema = z.lazy(() => schemaFactory(schema));
  return schema as T;
};

// Esempio d'uso
const menuItemSchema = createRecursiveSchema((self) =>
  z.object({
    label: z.string(),
    href: z.string().optional(),
    subItems: z.array(self).optional(),
    icon: z.string().optional(),
  })
);
Custom error messages i18n
typescript
import { z } from "zod";

type Locale = "it" | "en" | "fr" | "de";

const errorMessages = {
  it: {
    required: "Campo obbligatorio",
    email: "Email non valida",
    min: (min: number) => `Minimo ${min} caratteri`,
    max: (max: number) => `Massimo ${max} caratteri`,
    url: "URL non valido",
    phone: "Numero di telefono non valido",
  },
  en: {
    required: "Required field",
    email: "Invalid email",
    min: (min: number) => `Minimum ${min} characters`,
    max: (max: number) => `Maximum ${max} characters`,
    url: "Invalid URL",
    phone: "Invalid phone number",
  },
  // Aggiungi altre lingue...
};

class I18nZod {
  private locale: Locale = "it";

  setLocale(locale: Locale) {
    this.locale = locale;
  }

  string() {
    return z.string({
      errorMap: (issue, ctx) => {
        const messages = errorMessages[this.locale];
        
        switch (issue.code) {
          case z.ZodIssueCode.too_small:
            return { message: messages.min(issue.minimum as number) };
          case z.ZodIssueCode.too_big:
            return { message: messages.max(issue.maximum as number) };
          case z.ZodIssueCode.invalid_string:
            if (issue.validation === "email") {
              return { message: messages.email };
            }
            if (issue.validation === "url") {
              return { message: messages.url };
            }
            break;
          default:
            return { message: ctx.defaultError };
        }
      },
    });
  }

  email() {
    return this.string().email();
  }

  phone() {
    return this.string().regex(
      /^[+]?[(]?[0-9]{1,4}[)]?[-\s.]?[0-9]{1,4}[-\s.]?[0-9]{1,9}$/,
      errorMessages[this.locale].phone
    );
  }
}

// Esempio d'uso
const i18nZod = new I18nZod();
i18nZod.setLocale("it");

const userSchema = z.object({
  name: i18nZod.string().min(2),
  email: i18nZod.email(),
  phone: i18nZod.phone(),
});

// Oppure con factory pattern
const createLocalizedSchema = (locale: Locale) => {
  const messages = errorMessages[locale];
  
  return {
    string: (params?: { min?: number; max?: number }) => {
      let schema = z.string();
      
      if (params?.min) {
        schema = schema.min(params.min, messages.min(params.min));
      }
      if (params?.max) {
        schema = schema.max(params.max, messages.max(params.max));
      }
      
      return schema;
    },
    
    email: () => z.string().email(messages.email),
    
    url: () => z.string().url(messages.url),
    
    number: (params?: { min?: number; max?: number }) => {
      let schema = z.number();
      
      if (params?.min) {
        schema = schema.min(params.min, messages.min(params.min));
      }
      if (params?.max) {
        schema = schema.max(params.max, messages.max(params.max));
      }
      
      return schema;
    },
  };
};

// Schema dinamico per lingua
const getUserSchema = (locale: Locale) => {
  const localized = createLocalizedSchema(locale);
  
  return z.object({
    name: localized.string({ min: 2 }),
    email: localized.email(),
    website: localized.url().optional(),
    age: localized.number({ min: 18, max: 120 }),
  });
};
§ COMPLEX FORM PATTERNS
Multi-step wizard form
typescript
import { useState } from "react";
import { useForm, FormProvider } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

// Step schemas
const personalSchema = z.object({
  firstName: z.string().min(2, "Nome obbligatorio"),
  lastName: z.string().min(2, "Cognome obbligatorio"),
  email: z.string().email("Email non valida"),
});

const addressSchema = z.object({
  street: z.string().min(5, "Via obbligatoria"),
  city: z.string().min(2, "Città obbligatoria"),
  zipCode: z.string().regex(/^[0-9]{5}$/, "CAP non valido"),
  country: z.enum(["IT", "US", "DE", "FR"]),
});

const preferencesSchema = z.object({
  newsletter: z.boolean().default(false),
  notifications: z.boolean().default(true),
  marketing: z.boolean().default(false),
});

// Combined schema
const wizardSchema = personalSchema
  .merge(addressSchema)
  .merge(preferencesSchema);

type WizardFormData = z.infer<typeof wizardSchema>;

export function WizardForm() {
  const [step, setStep] = useState(1);
  const [completedSteps, setCompletedSteps] = useState<number[]>([]);

  const methods = useForm<WizardFormData>({
    resolver: zodResolver(wizardSchema),
    mode: "onChange",
  });

  const { trigger, getValues } = methods;

  const steps = [
    { id: 1, title: "Personale", schema: personalSchema },
    { id: 2, title: "Indirizzo", schema: addressSchema },
    { id: 3, title: "Preferenze", schema: preferencesSchema },
  ];

  const validateStep = async (stepNumber: number) => {
    const currentSchema = steps[stepNumber - 1].schema;
    const currentData = getValues();
    
    try {
      await currentSchema.parseAsync(currentData);
      setCompletedSteps((prev) => [...prev, stepNumber]);
      return true;
    } catch (error) {
      return false;
    }
  };

  const nextStep = async () => {
    const isValid = await validateStep(step);
    if (isValid && step < steps.length) {
      setStep(step + 1);
    }
  };

  const prevStep = () => {
    if (step > 1) {
      setStep(step - 1);
    }
  };

  const onSubmit = async (data: WizardFormData) => {
    console.log("Form completo:", data);
    // Submit logic
  };

  return (
    <FormProvider {...methods}>
      <div className="wizard-container">
        {/* Progress indicator */}
        <div className="wizard-progress">
          {steps.map((s) => (
            <div
              key={s.id}
              className={`step ${s.id === step ? "active" : ""} ${
                completedSteps.includes(s.id) ? "completed" : ""
              }`}
            >
              <div className="step-number">{s.id}</div>
              <div className="step-title">{s.title}</div>
            </div>
          ))}
        </div>

        <form onSubmit={methods.handleSubmit(onSubmit)}>
          {/* Step 1: Personal */}
          {step === 1 && (
            <div className="step-content">
              <h2>Informazioni Personali</h2>
              <input
                {...methods.register("firstName")}
                placeholder="Nome"
              />
              <input
                {...methods.register("lastName")}
                placeholder="Cognome"
              />
              <input
                {...methods.register("email")}
                placeholder="Email"
                type="email"
              />
            </div>
          )}

          {/* Step 2: Address */}
          {step === 2 && (
            <div className="step-content">
              <h2>Indirizzo</h2>
              <input
                {...methods.register("street")}
                placeholder="Via e numero"
              />
              <input
                {...methods.register("city")}
                placeholder="Città"
              />
              <input
                {...methods.register("zipCode")}
                placeholder="CAP"
              />
              <select {...methods.register("country")}>
                <option value="IT">Italia</option>
                <option value="US">USA</option>
                <option value="DE">Germania</option>
                <option value="FR">Francia</option>
              </select>
            </div>
          )}

          {/* Step 3: Preferences */}
          {step === 3 && (
            <div className="step-content">
              <h2>Preferenze</h2>
              <label>
                <input
                  type="checkbox"
                  {...methods.register("newsletter")}
                />
                Newsletter
              </label>
              <label>
                <input
                  type="checkbox"
                  {...methods.register("notifications")}
                />
                Notifiche
              </label>
              <label>
                <input
                  type="checkbox"
                  {...methods.register("marketing")}
                />
                Marketing
              </label>
            </div>
          )}

          {/* Navigation */}
          <div className="wizard-navigation">
            {step > 1 && (
              <button type="button" onClick={prevStep}>
                Indietro
              </button>
            )}
            
            {step < steps.length ? (
              <button type="button" onClick={nextStep}>
                Avanti
              </button>
            ) : (
              <button type="submit">Completa</button>
            )}
          </div>

          {/* Step indicator */}
          <div className="step-indicator">
            Step {step} di {steps.length}
          </div>
        </form>
      </div>
    </FormProvider>
  );
}
Conditional fields (show/hide)
typescript
import { useWatch } from "react-hook-form";

export function ConditionalForm() {
  const { register, control } = useForm();

  // Watch multiple fields for conditional logic
  const hasCompany = useWatch({ control, name: "hasCompany" });
  const country = useWatch({ control, name: "country" });
  const subscription = useWatch({ control, name: "subscription" });

  return (
    <form>
      {/* Toggle field */}
      <label>
        <input type="checkbox" {...register("hasCompany")} />
        Sono una società
      </label>

      {/* Conditional field group */}
      {hasCompany && (
        <div className="company-fields">
          <input
            {...register("companyName")}
            placeholder="Ragione sociale"
            required={hasCompany}
          />
          <input
            {...register("vatNumber")}
            placeholder="Partita IVA"
            required={hasCompany}
          />
        </div>
      )}

      {/* Country-specific fields */}
      <select {...register("country")}>
        <option value="">Seleziona paese</option>
        <option value="IT">Italia</option>
        <option value="US">USA</option>
        <option value="UK">Regno Unito</option>
      </select>

      {country === "IT" && (
        <input
          {...register("codiceFiscale")}
          placeholder="Codice Fiscale"
          required={country === "IT"}
        />
      )}

      {country === "US" && (
        <input
          {...register("ssn")}
          placeholder="Social Security Number"
          required={country === "US"}
        />
      )}

      {/* Subscription-dependent fields */}
      <select {...register("subscription")}>
        <option value="free">Free</option>
        <option value="pro">Pro</option>
        <option value="enterprise">Enterprise</option>
      </select>

      {subscription === "enterprise" && (
        <>
          <input
            {...register("enterpriseLicense")}
            placeholder="License key"
          />
          <input
            {...register("adminContact")}
            placeholder="Admin contact email"
          />
        </>
      )}

      {/* Multi-level conditions */}
      <label>
        <input type="checkbox" {...register("hasPromo")} />
        Ho un codice promozionale
      </label>

      {hasPromo && subscription !== "free" && (
        <input
          {...register("promoCode")}
          placeholder="Codice promozionale"
        />
      )}
    </form>
  );
}
Dynamic field dependencies
typescript
export function DependentFieldsForm() {
  const { register, watch, setValue, control } = useForm({
    defaultValues: {
      category: "",
      subcategory: "",
      product: "",
      features: {},
      price: 0,
    },
  });

  // Watch dependencies
  const category = watch("category");
  const subcategory = watch("subcategory");

  // Dynamic options based on category
  const subcategories = {
    electronics: ["smartphone", "laptop", "tablet"],
    clothing: ["men", "women", "kids"],
    books: ["fiction", "non-fiction", "academic"],
  };

  // Dynamic options based on subcategory
  const products = {
    smartphone: ["iPhone", "Samsung", "Google Pixel"],
    laptop: ["MacBook", "Dell", "Lenovo"],
    tablet: ["iPad", "Samsung Tab", "Surface"],
    men: ["Shirts", "Pants", "Jackets"],
    women: ["Dresses", "Blouses", "Skirts"],
    kids: ["T-shirts", "Shorts", "Sweaters"],
    fiction: ["Novels", "Short Stories", "Poetry"],
    "non-fiction": ["Biography", "History", "Science"],
    academic: ["Textbooks", "Research Papers", "Journals"],
  };

  // Reset dependent fields when parent changes
  useEffect(() => {
    setValue("subcategory", "");
    setValue("product", "");
    setValue("features", {});
    setValue("price", 0);
  }, [category, setValue]);

  useEffect(() => {
    setValue("product", "");
    setValue("features", {});
    setValue("price", 0);
  }, [subcategory, setValue]);

  // Dynamic feature fields based on product
  const productFeatures = {
    iPhone: ["storage", "color", "model"],
    MacBook: ["ram", "storage", "processor"],
    "Google Pixel": ["storage", "color", "edition"],
    Shirts: ["size", "color", "material"],
    Dresses: ["size", "color", "length"],
    Novels: ["format", "language", "edition"],
  };

  // Calculate price based on selections
  useEffect(() => {
    const selectedProduct = watch("product");
    const selectedFeatures = watch("features");
    
    if (selectedProduct) {
      let basePrice = 0;
      
      // Base prices
      const prices = {
        iPhone: 999,
        MacBook: 1299,
        "Google Pixel": 699,
        Shirts: 29,
        Dresses: 79,
        Novels: 15,
      };
      
      basePrice = prices[selectedProduct] || 0;
      
      // Add feature costs
      let featureCost = 0;
      Object.values(selectedFeatures).forEach((value) => {
        if (value && typeof value === "string") {
          featureCost += value.length * 10; // Example calculation
        }
      });
      
      setValue("price", basePrice + featureCost);
    }
  }, [watch("product"), watch("features")]);

  return (
    <form>
      {/* Category selection */}
      <select {...register("category")}>
        <option value="">Select Category</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
        <option value="books">Books</option>
      </select>

      {/* Subcategory - dynamic based on category */}
      {category && (
        <select {...register("subcategory")}>
          <option value="">Select Subcategory</option>
          {subcategories[category]?.map((sub) => (
            <option key={sub} value={sub}>
              {sub}
            </option>
          ))}
        </select>
      )}

      {/* Product - dynamic based on subcategory */}
      {subcategory && (
        <select {...register("product")}>
          <option value="">Select Product</option>
          {products[subcategory]?.map((prod) => (
            <option key={prod} value={prod}>
              {prod}
            </option>
          ))}
        </select>
      )}

      {/* Dynamic feature fields */}
      {watch("product") && productFeatures[watch("product")] && (
        <div className="features">
          <h3>Features</h3>
          {productFeatures[watch("product")].map((feature) => (
            <input
              key={feature}
              {...register(`features.${feature}`)}
              placeholder={feature}
            />
          ))}
        </div>
      )}

      {/* Price display */}
      <div className="price-display">
        <strong>Price: ${watch("price")}</strong>
      </div>
    </form>
  );
}
Cross-field validation
typescript
import { z } from "zod";

const crossFieldSchema = z
  .object({
    password: z.string().min(8, "Password minimo 8 caratteri"),
    confirmPassword: z.string(),
    startDate: z.string(),
    endDate: z.string(),
    minPrice: z.number().min(0),
    maxPrice: z.number().min(0),
    email: z.string().email(),
    confirmEmail: z.string(),
    discountCode: z.string().optional(),
    discountType: z.enum(["percent", "fixed"]),
    discountValue: z.number().min(0),
    adults: z.number().min(1),
    children: z.number().min(0),
  })
  .superRefine((data, ctx) => {
    // Password match
    if (data.password !== data.confirmPassword) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Le password non corrispondono",
        path: ["confirmPassword"],
      });
    }

    // Email match
    if (data.email !== data.confirmEmail) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: "Le email non corrispondono",
        path: ["confirmEmail"],
      });
    }

    // Date range validation
    if (data.startDate && data.endDate) {
      const start = new Date(data.startDate);
      const end = new Date(data.endDate);
      
      if (end < start) {
        ctx.addIssue({
          code: z.ZodIssueCode.custom,
          message: "La data fine deve essere successiva alla data inizio",
          path: ["endDate"],
        });
      }
      

════════════════════════════════════════════════════════════
FIGMA CATALOG: BATCH2-06-FORM-VALIDATION
Prompt ID: 6 / 19
Parte: 2
Exported: 2026-02-06T17:10:08.368Z
Characters: 2300
════════════════════════════════════════════════════════════

 = useState<Todo[]>([]);
  const [optimisticTodos, setOptimisticTodos] = useState<Todo[]>([]);
  const [pendingIds, setPendingIds] = useState<Set<string>>(new Set());
  const [isPending, startTransition] = useTransition();

  const { register, handleSubmit, reset } = useForm();

  // Add todo with optimistic update
  const addTodo = async (data: any) => {
    const tempId = `temp-${Date.now()}`;
    const newTodo: Todo = {
      id: tempId,
      text: data.todo,
      completed: false,
    };

    // Optimistic update
    setOptimisticTodos(prev => [...prev, newTodo]);
    setPendingIds(prev => new Set(prev).add(tempId));

    try {
      const response = await fetch('/api/todos', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: data.todo }),
      });

      const result = await response.json();

      // Replace optimistic todo with real one
      setTodos(prev => [...prev, result]);
      setOptimisticTodos(prev => prev.filter(t => t.id !== tempId));
      setPendingIds(prev => {
        const next = new Set(prev);
        next.delete(tempId);
        return next;
      });

      reset();
    } catch (error) {
      // Rollback on error
      setOptimisticTodos(prev => prev.filter(t => t.id !== tempId));
      setPendingIds(prev => {
        const next = new Set(prev);
        next.delete(tempId);
        return next;
      });
      alert('Errore nell\'aggiunta del todo');
    }
  };

  // Toggle todo with optimistic update
  const toggleTodo = async (id: string) => {
    const todo = [...todos, ...optimisticTodos].find(t => t.id === id);
    if (!todo) return;

    // Optimistic update
    setOptimisticTodos(prev => 
      prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );
    setTodos(prev => 
      prev.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );

    try {
      await fetch(`/api/todos/${id}`, {
        method: 'PATCH',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ completed: !todo.completed }),
      });

      // Sync with server
      const response = await fetch('/api/todos');
      const updatedTodos = await response.json();
      setTodos(updatedTodos);
      setOptimisticTodos([]);
   

## § ADVANCED PATTERNS: FORM VALIDATION

### Server Actions con Validazione

```typescript
// app/actions/form-validation.ts
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


### FORM VALIDATION - Utility Helper #346

```typescript
// lib/utils/form-validation-helper-346.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #938

```typescript
// lib/utils/form-validation-helper-938.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #524

```typescript
// lib/utils/form-validation-helper-524.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #91

```typescript
// lib/utils/form-validation-helper-91.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #37

```typescript
// lib/utils/form-validation-helper-37.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #656

```typescript
// lib/utils/form-validation-helper-656.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #395

```typescript
// lib/utils/form-validation-helper-395.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #818

```typescript
// lib/utils/form-validation-helper-818.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #346

```typescript
// lib/utils/form-validation-helper-346.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #333

```typescript
// lib/utils/form-validation-helper-333.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #959

```typescript
// lib/utils/form-validation-helper-959.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #40

```typescript
// lib/utils/form-validation-helper-40.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #794

```typescript
// lib/utils/form-validation-helper-794.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #580

```typescript
// lib/utils/form-validation-helper-580.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #270

```typescript
// lib/utils/form-validation-helper-270.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### FORM VALIDATION - Utility Helper #863

```typescript
// lib/utils/form-validation-helper-863.ts
import { z } from "zod";

interface FORMVALIDATIONConfig {
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

export class FORMVALIDATIONProcessor<TInput, TOutput> {
  private config: FORMVALIDATIONConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<FORMVALIDATIONConfig> = {}) {
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

  getConfig(): Readonly<FORMVALIDATIONConfig> {
    return Object.freeze({ ...this.config });
  }
}
```
