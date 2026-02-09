---
# Integrato da: 01-OUTPUT-MASTER-ORCHESTRATOR.md
# Data integrazione: 2026-01-29 14:51
# Generato con: Gemini 2.5 Flash / DeepSeek R1
---

# MASTER-ORCHESTRATOR.md

## 1. INTRODUZIONE

Il sistema **MASTER ORCHESTRATOR** è una piattaforma AI avanzata progettata per generare autonomamente piattaforme web complete, robuste e pronte per la produzione. Il suo scopo primario è automatizzare il processo di sviluppo, garantendo al contempo una qualità del codice impeccabile, l'aderenza a standard industriali elevati e la conformità a un'architettura predefinita e ottimizzata. MASTER ORCHESTRATOR elimina la necessità di boilerplate manuale e riduce drasticamente i tempi di sviluppo, permettendo ai team di concentrarsi sulla logica di business complessa piuttosto che sull'infrastruttura ripetitiva.

Ogni piattaforma generata da MASTER ORCHESTRATOR è costruita su uno stack tecnologico moderno e performante, scelto per la sua scalabilità, manutenibilità e robustezza:

*   **Next.js 14 App Router**: Framework React per la creazione di applicazioni web full-stack, sfruttando le ultime funzionalità di routing, server components e server actions per prestazioni ottimali e un'esperienza di sviluppo fluida.
*   **TypeScript strict**: Un superset di JavaScript che aggiunge tipizzazione statica, migliorando la leggibilità del codice, la rilevazione degli errori in fase di sviluppo e la manutenibilità a lungo termine. La configurazione `strict` è sempre abilitata per garantire la massima sicurezza e coerenza.
*   **Prisma ORM**: Un ORM (Object-Relational Mapper) di nuova generazione che semplifica l'interazione con il database, fornendo un'API intuitiva e type-safe per query, migrazioni e gestione dello schema.
*   **tRPC**: Una libreria che consente di creare API type-safe end-to-end senza la necessità di generare schemi o definizioni. Facilita la comunicazione tra client e server in modo efficiente e sicuro.
*   **Tailwind CSS**: Un framework CSS utility-first che permette di costruire interfacce utente personalizzate direttamente nel markup, accelerando lo sviluppo e garantendo una coerenza stilistica.
*   **Auth.js v5 (NextAuth.js)**: Soluzione di autenticazione completa e flessibile per Next.js, che supporta vari provider di autenticazione e strategie di sessione, garantendo sicurezza e facilità d'uso.
*   **shadcn/ui**: Una collezione di componenti UI riutilizzabili e accessibili, costruiti con Radix UI e Tailwind CSS. Non è una libreria di componenti nel senso tradizionale, ma una raccolta di codice che viene copiato direttamente nel progetto, permettendo una personalizzazione completa e un controllo granulare.

I principi guida fondamentali che governano ogni output di MASTER ORCHESTRATOR sono:

*   **Codice Production-Ready**: Ogni riga di codice generata è pensata per essere immediatamente distribuibile in un ambiente di produzione. Questo implica l'adozione di best practice, l'ottimizzazione delle prestazioni, la sicurezza e la scalabilità fin dalla fase di generazione.
*   **Zero Placeholder**: Non ci sono sezioni di codice vuote, commenti "TODO" o marcatori che richiedono un'implementazione manuale successiva. Il codice è completo e funzionale per lo scopo definito.
*   **Zero TODO**: Non vengono lasciati commenti `// TODO` o `// FIXME`. Ogni funzionalità è implementata completamente o non è presente. Se una funzionalità è richiesta, deve essere generata in modo completo e funzionante.

MASTER ORCHESTRATOR agisce come un architetto software virtuale, garantendo che ogni progetto rispetti una visione unificata di eccellenza tecnica, efficienza e manutenibilità.

## 2. ARCHITETTURA STANDARD

Ogni progetto generato da MASTER ORCHESTRATOR aderirà rigorosamente alla seguente struttura di cartelle, progettata per massimizzare la chiarezza, la manutenibilità e la scalabilità.

```
project-root/
├── src/
│   ├── app/                 # Next.js App Router
│   ├── components/          # React components
│   ├── lib/                 # Utilities e configurazioni
│   ├── server/              # tRPC routers e services
│   ├── hooks/               # Custom React hooks
│   ├── types/               # TypeScript types
│   └── styles/              # Global styles
├── prisma/                  # Database schema
├── public/                  # Static assets
└── tests/                   # Test files
```

### Dettaglio delle Cartelle:

#### `project-root/src/app/`
*   **Scopo Esatto**: Contiene tutte le route dell'applicazione Next.js, inclusi pagine, layout, loading states, error pages e API routes. Utilizza il paradigma App Router.
*   **Convenzioni Naming**:
    *   Le cartelle per le route dinamiche usano parentesi quadre: `[slug]`, `[id]`.
    *   Le cartelle per i gruppi di route usano parentesi tonde: `(auth)`, `(dashboard)`.
    *   I file di pagina sono sempre `page.tsx`.
    *   I file di layout sono sempre `layout.tsx`.
    *   I file di caricamento sono sempre `loading.tsx`.
    *   I file di errore sono sempre `error.tsx`.
    *   Le route API sono sempre `route.ts` (per API RESTful) o `route.tsx` (per Server Actions).
    *   I file `default.tsx` per i catch-all routes.
*   **File Obbligatori**:
    *   `src/app/layout.tsx`: Layout root dell'applicazione.
    *   `src/app/page.tsx`: Pagina iniziale dell'applicazione.

#### `project-root/src/components/`
*   **Scopo Esatto**: Contiene tutti i componenti React riutilizzabili. I componenti sono suddivisi in sottocartelle per categorie logiche (es. `ui`, `forms`, `layout`).
*   **Convenzioni Naming**:
    *   Le cartelle dei componenti usano PascalCase: `Button`, `UserCard`.
    *   I file dei componenti usano PascalCase: `Button.tsx`, `UserCard.tsx`.
    *   Per componenti complessi con file multipli (es. un componente con sottocomponenti o stili specifici), si usa una cartella con un file `index.tsx`: `Button/index.tsx`, `Button/button-icon.tsx`.
    *   I componenti `shadcn/ui` vengono generati qui, spesso in `components/ui/`.
*   **File Obbligatori**: Nessuno a livello root, ma ogni componente deve avere il suo file `.tsx`.

#### `project-root/src/lib/`
*   **Scopo Esatto**: Contiene utility generiche, configurazioni globali, client di database, costanti e helper non specifici per React o tRPC.
*   **Convenzioni Naming**:
    *   I file usano kebab-case: `db.ts`, `utils.ts`, `constants.ts`, `config.ts`.
    *   Le sottocartelle per raggruppare utility correlate: `lib/auth/`, `lib/mail/`.
*   **File Obbligatori**:
    *   `src/lib/db.ts`: Istanza del client Prisma.
    *   `src/lib/utils.ts`: Funzioni utility generiche (es. `cn` per Tailwind).
    *   `src/lib/auth.ts`: Configurazione Auth.js.

#### `project-root/src/server/`
*   **Scopo Esatto**: Contiene la logica server-side, inclusi i router tRPC, i servizi che incapsulano la logica di business e le interazioni con il database, e le utility server-side.
*   **Convenzioni Naming**:
    *   Le sottocartelle per i router tRPC: `server/api/routers/`.
    *   Le sottocartelle per i servizi: `server/services/`.
    *   I file dei router usano camelCase: `userRouter.ts`, `postRouter.ts`.
    *   I file dei servizi usano PascalCase: `UserService.ts`, `PostService.ts`.
*   **File Obbligatori**:
    *   `src/server/api/trpc.ts`: Contesto tRPC e definizioni delle procedure.
    *   `src/server/api/root.ts`: Router tRPC principale che aggrega tutti i sub-router.

#### `project-root/src/hooks/`
*   **Scopo Esatto**: Contiene tutti i custom React hooks riutilizzabili.
*   **Convenzioni Naming**:
    *   I file usano camelCase e iniziano con `use`: `useAuth.ts`, `useDebounce.ts`.
*   **File Obbligatori**: Nessuno a livello root.

#### `project-root/src/types/`
*   **Scopo Esatto**: Contiene tutte le definizioni di tipi e interfacce TypeScript globali o condivise tra più moduli.
*   **Convenzioni Naming**:
    *   I file usano kebab-case o PascalCase per i nomi dei tipi: `user.ts`, `post.ts`, `api-response.ts`.
    *   Le interfacce e i tipi all'interno dei file usano PascalCase.
*   **File Obbligatori**:
    *   `src/types/next-auth.d.ts`: Estensioni dei tipi per Auth.js.
    *   `src/types/global.d.ts`: Tipi globali generici.

#### `project-root/src/styles/`
*   **Scopo Esatto**: Contiene i file CSS globali e le configurazioni di stile (es. Tailwind CSS).
*   **Convenzioni Naming**:
    *   I file usano kebab-case: `globals.css`.
*   **File Obbligatori**:
    *   `src/styles/globals.css`: File CSS principale per stili globali e direttive Tailwind.

#### `project-root/prisma/`
*   **Scopo Esatto**: Contiene lo schema del database Prisma e le migrazioni.
*   **Convenzioni Naming**:
    *   Il file dello schema è sempre `schema.prisma`.
    *   Le cartelle delle migrazioni sono generate automaticamente da Prisma.
*   **File Obbligatori**:
    *   `prisma/schema.prisma`: Definizione dello schema del database.

#### `project-root/public/`
*   **Scopo Esatto**: Contiene asset statici come immagini, font, icone e file manifest che devono essere serviti direttamente.
*   **Convenzioni Naming**:
    *   I nomi dei file sono descrittivi e in kebab-case: `logo.png`, `favicon.ico`.
*   **File Obbligatori**:
    *   `public/favicon.ico`: Icona del sito.

#### `project-root/tests/`
*   **Scopo Esatto**: Contiene tutti i file di test (unit, integration, end-to-end).
*   **Convenzioni Naming**:
    *   I file di test usano il suffisso `.test.ts` o `.spec.ts`: `userService.test.ts`, `homePage.spec.tsx`.
    *   Le sottocartelle possono riflettere la struttura di `src/` per facilitare la navigazione.
*   **File Obbligatori**: Nessuno a livello root.

## 3. CONVENZIONI CODICE

Le seguenti convenzioni di codice sono **OBBLIGATORIE** e devono essere applicate rigorosamente a ogni riga di codice generata.

### 3.1 Naming Conventions

*   **PascalCase**:
    *   Componenti React (es. `Button`, `UserProfileCard`).
    *   Interfacce e Tipi TypeScript (es. `User`, `PostProps`).
    *   Classi (es. `UserService`, `AppError`).
    *   Enum.
*   **camelCase**:
    *   Funzioni (es. `getUserById`, `formatDate`).
    *   Variabili (es. `userName`, `isLoading`).
    *   Metodi di classi (es. `userService.createUser`).
    *   Proprietà di oggetti (es. `user.firstName`).
    *   Nomi di file per router tRPC (es. `userRouter.ts`).
*   **SCREAMING_SNAKE_CASE**:
    *   Costanti globali o di modulo (es. `API_BASE_URL`, `MAX_FILE_SIZE`).
    *   Valori di Enum (se non si usa PascalCase per i valori).

### 3.2 Convenzioni sui File

*   **kebab-case per i nomi dei file**: Tutti i nomi dei file devono essere in kebab-case (es. `my-component.tsx`, `user-service.ts`, `use-auth.ts`). Fanno eccezione i file `page.tsx`, `layout.tsx`, `route.ts`, `index.tsx` e i file di configurazione come `db.ts`, `trpc.ts`, `root.ts`.
*   **Un componente per file**: Ogni file `.tsx` nella cartella `src/components/` deve contenere un singolo componente React principale. Sottocomponenti strettamente correlati e di piccole dimensioni possono essere definiti nello stesso file, ma la pratica preferita è la separazione.
*   **File `index.tsx` per cartelle di componenti**: Se un componente è complesso e richiede una propria cartella (es. per stili specifici, sottocomponenti), il componente principale deve risiedere in `index.tsx` all'interno di quella cartella (es. `components/Button/index.tsx`).

### 3.3 Convenzioni sugli Export

*   **Named Exports Preferiti**: Per funzioni, variabili, tipi e classi, devono essere utilizzati gli `named exports`. Questo migliora la tracciabilità e la refactoring.
    ```typescript
    // src/lib/utils.ts
    export function formatCurrency(amount: number): string { /* ... */ }
    export const APP_NAME = "Master Orchestrator App";

    // src/components/Button.tsx
    export interface ButtonProps { /* ... */ }
    export function Button({ children }: ButtonProps) { /* ... */ }
    ```
*   **Default Exports solo per Pagine e Layout**: I `default exports` sono riservati esclusivamente per le pagine Next.js (`page.tsx`), i layout (`layout.tsx`) e, occasionalmente, per il file `index.tsx` di un componente che è l'export principale della sua cartella.
    ```typescript
    // src/app/dashboard/page.tsx
    export default function DashboardPage() { /* ... */ }

    // src/components/Button/index.tsx
    export { Button as default } from './Button'; // Se Button è definito in Button.tsx
    ```

### 3.4 Convenzioni sui Tipi TypeScript

*   **`interface` per oggetti**: Utilizzare `interface` per definire la forma di oggetti, classi e props di componenti. Le interfacce possono essere estese e implementate.
    ```typescript
    interface User {
      id: string;
      name: string;
      email: string;
    }

    interface ButtonProps {
      variant: 'primary' | 'secondary';
      onClick: () => void;
    }
    ```
*   **`type` per union, intersection e utility types**: Utilizzare `type` per definire alias di tipi primitivi, tipi unione (`union types`), tipi intersezione (`intersection types`) e tipi utility complessi.
    ```typescript
    type UserRole = 'ADMIN' | 'USER' | 'GUEST';
    type ID = string | number;
    type ApiResponse<T> = { data: T; message: string; };
    ```

### 3.5 Ordine degli Imports

Gli import devono essere raggruppati e ordinati in base alla loro provenienza, con una riga vuota tra ogni gruppo.

1.  **React**: `import React from 'react';`
2.  **Next.js**: `import { useRouter } from 'next/navigation';`, `import type { NextPage } from 'next';`
3.  **Librerie Esterne**: `import axios from 'axios';`, `import { format } from 'date-fns';`
4.  **Librerie Interne (`src/lib/`)**: `import { cn } from '@/lib/utils';`, `import { db } from '@/lib/db';`
5.  **Server-side (`src/server/`)**: `import { api } from '@/trpc/react';`, `import { userService } from '@/server/services/UserService';`
6.  **Componenti (`src/components/`)**: `import { Button } from '@/components/ui/button';`, `import { UserCard } from '@/components/user-card';`
7.  **Hooks (`src/hooks/`)**: `import { useAuth } from '@/hooks/use-auth';`
8.  **Tipi (`src/types/`)**: `import { type User } from '@/types/user';`
9.  **Stili (`src/styles/`)**: `import '@/styles/globals.css';`

```typescript
// Esempio di ordine degli import
import React from 'react';
import { useRouter } from 'next/navigation';
import type { NextPage } from 'next';

import { format } from 'date-fns';
import { toast } from 'sonner';

import { cn } from '@/lib/utils';
import { api } from '@/trpc/react';

import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';

import { useDebounce } from '@/hooks/use-debounce';

import { type User } from '@/types/user';

import '@/styles/globals.css';
```

## 4. PATTERN OBBLIGATORI

MASTER ORCHESTRATOR genera codice seguendo pattern specifici per garantire coerenza, manutenibilità e scalabilità.

### 4.1 Service Pattern

Il Service Pattern incapsula la logica di business e l'interazione con il database per una specifica risorsa. Ogni servizio è una classe che espone metodi per operazioni CRUD e altre logiche correlate.

```typescript
// src/server/services/UserService.ts
import { db } from '@/lib/db';
import { type UserCreateInput, type UserUpdateInput } from '@/types/user';
import { AppError, NotFoundError } from '@/lib/errors';

export class UserService {
  /**
   * Recupera tutti gli utenti.
   * @returns Una lista di utenti.
   */
  async getAllUsers() {
    try {
      const users = await db.user.findMany({
        select: {
          id: true,
          name: true,
          email: true,
          emailVerified: true,
          image: true,
          role: true,
          createdAt: true,
          updatedAt: true,
        },
      });
      return users;
    } catch (error) {
      throw new AppError('Failed to retrieve users.', 500, error);
    }
  }

  /**
   * Recupera un utente per ID.
   * @param id L'ID dell'utente.
   * @returns L'utente trovato.
   * @throws NotFoundError se l'utente non esiste.
   */
  async getUserById(id: string) {
    const user = await db.user.findUnique({
      where: { id },
      select: {
        id: true,
        name: true,
        email: true,
        emailVerified: true,
        image: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    });

    if (!user) {
      throw new NotFoundError(`User with ID ${id} not found.`);
    }
    return user;
  }

  /**
   * Crea un nuovo utente.
   * @param data I dati per la creazione dell'utente.
   * @returns L'utente creato.
   */
  async createUser(data: UserCreateInput) {
    try {
      const newUser = await db.user.create({ data });
      return newUser;
    } catch (error) {
      throw new AppError('Failed to create user.', 500, error);
    }
  }

  /**
   * Aggiorna un utente esistente.
   * @param id L'ID dell'utente da aggiornare.
   * @param data I dati per l'aggiornamento dell'utente.
   * @returns L'utente aggiornato.
   * @throws NotFoundError se l'utente non esiste.
   */
  async updateUser(id: string, data: UserUpdateInput) {
    try {
      const updatedUser = await db.user.update({
        where: { id },
        data,
      });
      return updatedUser;
    } catch (error) {
      if (error instanceof Error && 'code' in error && error.code === 'P2025') {
        throw new NotFoundError(`User with ID ${id} not found for update.`);
      }
      throw new AppError('Failed to update user.', 500, error);
    }
  }

  /**
   * Elimina un utente.
   * @param id L'ID dell'utente da eliminare.
   * @returns L'utente eliminato.
   * @throws NotFoundError se l'utente non esiste.
   */
  async deleteUser(id: string) {
    try {
      const deletedUser = await db.user.delete({
        where: { id },
      });
      return deletedUser;
    } catch (error) {
      if (error instanceof Error && 'code' in error && error.code === 'P2025') {
        throw new NotFoundError(`User with ID ${id} not found for deletion.`);
      }
      throw new AppError('Failed to delete user.', 500, error);
    }
  }
}

export const userService = new UserService();
```

### 4.2 tRPC Router Pattern

Il tRPC Router Pattern definisce gli endpoint API type-safe. Ogni risorsa avrà il proprio router, che verrà poi aggregato nel `root.ts`.

```typescript
// src/server/api/routers/userRouter.ts
import { z } from 'zod';
import { createTRPCRouter, publicProcedure, protectedProcedure } from '@/server/api/trpc';
import { userService } from '@/server/services/UserService';
import { createUserSchema, updateUserSchema } from '@/types/user'; // Schemi Zod per validazione

export const userRouter = createTRPCRouter({
  getAll: publicProcedure.query(async () => {
    return userService.getAllUsers();
  }),

  getById: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(async ({ input }) => {
      return userService.getUserById(input.id);
    }),

  create: protectedProcedure
    .input(createUserSchema) // Usa lo schema Zod per la validazione
    .mutation(async ({ input }) => {
      return userService.createUser(input);
    }),

  update: protectedProcedure
    .input(updateUserSchema) // Usa lo schema Zod per la validazione
    .mutation(async ({ input }) => {
      const { id, ...data } = input;
      return userService.updateUser(id, data);
    }),

  delete: protectedProcedure
    .input(z.object({ id: z.string().uuid() }))
    .mutation(async ({ input }) => {
      return userService.deleteUser(input.id);
    }),
});
```

### 4.3 React Component Pattern

Questo pattern definisce la struttura standard per i componenti React funzionali, inclusa la tipizzazione delle props e l'uso di Tailwind CSS con `clsx`/`tailwind-merge`.

```typescript
// src/components/ui/button.tsx
'use client'; // Se il componente è interattivo e usa hooks o eventi client-side

import * as React from 'react';
import { Slot } from '@radix-ui/react-slot';
import { cva, type VariantProps } from 'class-variance-authority';

import { cn } from '@/lib/utils'; // Funzione per unire classi Tailwind

const buttonVariants = cva(
  'inline-flex items-center justify-center whitespace-nowrap rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
      },
      size: {
        default: 'h-10 px-4 py-2',
        sm: 'h-9 rounded-md px-3',
        lg: 'h-11 rounded-md px-8',
        icon: 'h-10 w-10',
      },
    },
    defaultVariants: {
      variant: 'default',
      size: 'default',
    },
  },
);

export interface ButtonProps
  extends React.ButtonHTMLAttributes<HTMLButtonElement>,
    VariantProps<typeof buttonVariants> {
  asChild?: boolean;
}

const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ className, variant, size, asChild = false, ...props }, ref) => {
    const Comp = asChild ? Slot : 'button';
    return (
      <Comp
        className={cn(buttonVariants({ variant, size, className }))}
        ref={ref}
        {...props}
      />
    );
  },
);
Button.displayName = 'Button';

export { Button, buttonVariants };
```

### 4.4 Custom Hook Pattern

I custom hooks incapsulano la logica di stato riutilizzabile e gli effetti collaterali.

```typescript
// src/hooks/use-debounce.ts
import { useState, useEffect } from 'react';

/**
 * Hook per il debounce di un valore.
 * Utile per ritardare l'esecuzione di una funzione fino a quando un valore non smette di cambiare.
 * @param value Il valore da debounciare.
 * @param delay Il ritardo in millisecondi.
 * @returns Il valore debounced.
 */
export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    // Imposta un timer per aggiornare il valore debounced dopo il ritardo
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Pulisce il timer se il valore cambia o il componente si smonta
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]); // Rilancia l'effetto solo se il valore o il ritardo cambiano

  return debouncedValue;
}
```

### 4.5 Zod Schema Pattern

Zod è utilizzato per la validazione degli input in tutti i punti critici (API, Server Actions, forms).

```typescript
// src/types/user.ts
import { z } from 'zod';

// Schema per la creazione di un utente
export const createUserSchema = z.object({
  name: z.string().min(2, { message: 'Name must be at least 2 characters.' }).max(50, { message: 'Name must not exceed 50 characters.' }),
  email: z.string().email({ message: 'Invalid email address.' }),
  password: z.string().min(8, { message: 'Password must be at least 8 characters.' }),
  role: z.enum(['ADMIN', 'USER', 'GUEST']).default('USER'),
});

// Tipo derivato dallo schema di creazione
export type UserCreateInput = z.infer<typeof createUserSchema>;

// Schema per l'aggiornamento di un utente (tutti i campi opzionali tranne l'ID)
export const updateUserSchema = z.object({
  id: z.string().uuid({ message: 'Invalid user ID format.' }),
  name: z.string().min(2, { message: 'Name must be at least 2 characters.' }).max(50, { message: 'Name must not exceed 50 characters.' }).optional(),
  email: z.string().email({ message: 'Invalid email address.' }).optional(),
  password: z.string().min(8, { message: 'Password must be at least 8 characters.' }).optional(),
  role: z.enum(['ADMIN', 'USER', 'GUEST']).optional(),
}).refine(data => Object.keys(data).length > 1, {
  message: 'At least one field (name, email, password, or role) must be provided for update.',
  path: ['name'], // Indica il campo generico per l'errore
});

// Tipo derivato dallo schema di aggiornamento
export type UserUpdateInput = z.infer<typeof updateUserSchema>;

// Schema per l'autenticazione (login)
export const loginSchema = z.object({
  email: z.string().email({ message: 'Invalid email address.' }),
  password: z.string().min(1, { message: 'Password is required.' }),
});

export type LoginInput = z.infer<typeof loginSchema>;
```

## 5. ERROR HANDLING STANDARD

La gestione degli errori è cruciale per la robustezza di qualsiasi applicazione. MASTER ORCHESTRATOR implementa un sistema di error handling standardizzato e centralizzato.

### 5.1 Classi Errore Custom

Vengono definite classi di errore custom per tipizzare e categorizzare gli errori, facilitando la gestione e la presentazione all'utente.

```typescript
// src/lib/errors.ts
export class AppError extends Error {
  public readonly statusCode: number;
  public readonly isOperational: boolean;
  public readonly details?: unknown;

  constructor(message: string, statusCode: number = 500, details?: unknown, isOperational: boolean = true) {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    this.details = details; // Dettagli tecnici per il logging, non per l'utente finale
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(message: string = 'Resource not found.', details?: unknown) {
    super(message, 404, details);
  }
}

export class ValidationError extends AppError {
  constructor(message: string = 'Validation failed.', details?: unknown) {
    super(message, 400, details);
  }
}

export class UnauthorizedError extends AppError {
  constructor(message: string = 'Authentication required.', details?: unknown) {
    super(message, 401, details);
  }
}

export class ForbiddenError extends AppError {
  constructor(message: string = 'Access denied.', details?: unknown) {
    super(message, 403, details);
  }
}
```

### 5.2 Try-Catch Pattern

Tutte le operazioni che possono fallire (interazioni con DB, chiamate API esterne, logica complessa) devono essere avvolte in blocchi `try-catch`. Gli errori catturati devono essere trasformati nelle classi di errore custom appropriate o rilanciati come `AppError` per una gestione centralizzata.

```typescript
// Esempio in un Service
import { AppError, NotFoundError } from '@/lib/errors';
import { db } from '@/lib/db';

async function getUserData(userId: string) {
  try {
    const user = await db.user.findUnique({ where: { id: userId } });
    if (!user) {
      throw new NotFoundError(`User with ID ${userId} not found.`);
    }
    return user;
  } catch (error) {
    // Se è già un AppError, rilancialo. Altrimenti, incapsulalo.
    if (error instanceof AppError) {
      throw error;
    }
    // Log dell'errore originale per debugging
    console.error('Original error in getUserData:', error);
    throw new AppError('Failed to retrieve user data.', 500, error);
  }
}

// Esempio in una Server Action
import { AppError, ValidationError } from '@/lib/errors';
import { createUserSchema } from '@/types/user';
import { userService } from '@/server/services/UserService';
import { revalidatePath } from 'next/cache';

export async function createNewUser(formData: FormData) {
  try {
    const rawData = Object.fromEntries(formData.entries());
    const validatedData = createUserSchema.parse(rawData); // Zod throws on validation error

    await userService.createUser(validatedData);
    revalidatePath('/admin/users');
    return { success: true, message: 'User created successfully.' };
  } catch (error) {
    if (error instanceof AppError) {
      return { success: false, message: error.message, statusCode: error.statusCode };
    }
    if (error instanceof Error) { // ZodError è un tipo di Error
      if ('issues' in error) { // ZodError ha una proprietà 'issues'
        return { success: false, message: 'Validation failed.', errors: error.issues };
      }
      return { success: false, message: error.message, statusCode: 500 };
    }
    return { success: false, message: 'An unexpected error occurred.', statusCode: 500 };
  }
}
```

### 5.3 Error Boundaries (Client-side)

Per i componenti React client-side, devono essere utilizzati gli Error Boundaries per catturare errori nell'albero dei componenti e mostrare un fallback UI, prevenendo il crash dell'intera applicazione.

```typescript
// src/app/error.tsx (Next.js App Router Error Boundary)
'use client'; // Error components must be Client Components

import { useEffect } from 'react';
import { Button } from '@/components/ui/button';

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Log the error to an error reporting service
    console.error('Client-side error caught by boundary:', error);
  }, [error]);

  return (
    <div className="flex min-h-screen flex-col items-center justify-center space-y-4 text-center">
      <h2 className="text-2xl font-bold text-destructive">Something went wrong!</h2>
      <p className="text-muted-foreground">An unexpected error occurred. Please try again.</p>
      <Button
        onClick={
          // Attempt to recover by trying to re-render the segment
          () => reset()
        }
      >
        Try again
      </Button>
      <p className="text-sm text-gray-500">Error details: {error.message}</p>
    </div>
  );
}
```

### 5.4 API Error Responses

Le risposte API in caso di errore devono seguire uno standard JSON coerente per facilitare la gestione lato client.

```json
// Esempio di risposta API in caso di errore
{
  "success": false,
  "message": "Resource not found.",
  "code": "NOT_FOUND_ERROR",
  "statusCode": 404,
  "details": {
    "resource": "User",
    "id": "non-existent-id"
  }
}

// Esempio di risposta API in caso di errore di validazione Zod
{
  "success": false,
  "message": "Validation failed.",
  "code": "VALIDATION_ERROR",
  "statusCode": 400,
  "errors": [
    {
      "path": ["email"],
      "message": "Invalid email address."
    },
    {
      "path": ["password"],
      "message": "Password must be at least 8 characters."
    }
  ]
}
```
L'implementazione di tRPC gestirà automaticamente la serializzazione degli errori `AppError` in un formato coerente, ma è fondamentale che i servizi e le procedure tRPC lancino questi errori custom.

## 6. CHECKLIST PRE-GENERAZIONE

Prima di avviare il processo di generazione del codice, MASTER ORCHESTRATOR deve verificare che i seguenti requisiti siano stati definiti e siano coerenti. Questa checklist garantisce che l'input per l'AI sia completo e privo di ambiguità.

-   [x] **Definizione Requisiti Funzionali**: Tutti i requisiti funzionali dell'applicazione (es. gestione utenti, CRUD per entità specifiche, flussi di autenticazione) sono stati chiaramente specificati.
-   [x] **Schema Database Dettagliato**: Lo schema del database (`prisma/schema.prisma`) è stato completamente definito, inclusi modelli, relazioni, tipi di campo e attributi (es. `@id`, `@unique`, `@default`).
-   [x] **Requisiti di Autenticazione/Autorizzazione**: I metodi di autenticazione (es. email/password, OAuth providers) e le regole di autorizzazione (ruoli utente, permessi) sono stati specificati.
-   [x] **Validazione Input (Zod Schemas)**: Tutti gli schemi di validazione Zod necessari per gli input di creazione/aggiornamento di entità e per le API sono stati definiti in `src/types/`.
-   [x] **Definizione Entità e Tipi**: Tutti i tipi e le interfacce TypeScript per le entità di business e le strutture dati complesse sono stati definiti in `src/types/`.
-   [x] **Configurazioni Specifiche**: Eventuali configurazioni specifiche dell'applicazione (es. variabili d'ambiente, chiavi API esterne) sono state identificate e rese disponibili.
-   [x] **Flussi Utente Critici**: I flussi utente critici (es. registrazione, login, reset password, creazione di una risorsa principale) sono stati delineati.
-   [x] **Requisiti UI/UX (se applicabile)**: Se sono richiesti componenti UI specifici oltre a `shadcn/ui`, le loro specifiche (props, comportamento) sono state fornite.
-   [x] **Strategia di Error Handling**: La strategia di gestione degli errori per ogni flusso critico è stata definita (es. quali errori possono verificarsi e come devono essere gestiti/presentati).
-   [x] **Nessuna Ambiguità**: Tutti i requisiti sono chiari, non contraddittori e non lasciano spazio a interpretazioni.

## 7. CHECKLIST POST-GENERAZIONE

Dopo la generazione del codice, MASTER ORCHESTRATOR deve eseguire una serie di controlli automatici per garantire che l'output sia conforme a tutti gli standard e i principi guida.

-   [x] **Conformità Struttura Cartelle**: La struttura di cartelle generata corrisponde esattamente all'architettura standard definita nella Sezione 2.
-   [x] **Aderenza Convenzioni Naming**: Tutti i file, cartelle, componenti, funzioni, variabili e tipi rispettano le convenzioni di naming (PascalCase, camelCase, SCREAMING_SNAKE_CASE) della Sezione 3.1.
-   [x] **Zero Placeholder/TODO**: Il codice generato non contiene alcun placeholder, commento `// TODO`, `// FIXME` o sezioni "da implementare".
-   [x] **Tipizzazione Completa (TypeScript Strict)**: Non ci sono errori di tipizzazione TypeScript. Tutti i tipi sono espliciti o inferiti correttamente, e la configurazione `strict` è soddisfatta.
-   [x] **Validazione Input con Zod**: Tutti gli input utente (es. form, query params, body API) sono validati utilizzando schemi Zod appropriati.
-   [x] **Implementazione Error Handling**: Le classi di errore custom sono utilizzate. I blocchi `try-catch` sono presenti nei punti critici (servizi, API routes, server actions). Gli Error Boundaries sono configurati per le pagine client-side.
-   [x] **Correttezza degli Imports**: Tutti gli import sono risolti correttamente, seguono l'ordine specificato nella Sezione 3.5 e non ci sono import inutilizzati.
-   [x] **Utilizzo di Prisma ORM**: Tutte le interazioni con il database avvengono tramite il client Prisma, utilizzando i modelli definiti.
-   [x] **Utilizzo di tRPC**: Tutte le API server-side sono implementate tramite router e procedure tRPC, con input e output type-safe.
-   [x] **Coerenza UI/UX (shadcn/ui & Tailwind)**: I componenti UI utilizzano `shadcn/ui` e `Tailwind CSS` in modo coerente, applicando le utility class e le varianti definite.
-   [x] **Codice Formattato (Prettier/ESLint)**: Il codice è formattato automaticamente secondo le regole standard (es. Prettier) e non presenta warning o errori ESLint.
-   [x] **Funzionalità Base Operativa**: Le funzionalità di base (es. autenticazione, CRUD per le entità principali) sono implementate e potenzialmente testabili (se i test sono stati generati).

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 22399_