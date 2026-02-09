# CATALOGO-DOCUMENTATION

Catalogo Documentazione per Next.js 14 + TypeScript
§ README STANDARDS
Project README Template
markdown
# Nome Progetto

[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-blue)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![CI/CD](https://github.com/username/repo/actions/workflows/ci.yml/badge.svg)](https://github.com/username/repo/actions)

Breve descrizione del progetto, scopo e caratteristiche principali.

§ ✨ FUNZIONALITÀ

- ✅ Funzionalità 1
- ✅ Funzionalità 2  
- 🚧 Funzionalità 3 (in sviluppo)

§ 🚀 QUICK START

§ PREREQUISITI
- Node.js 18.17 o superiore
- npm, yarn, o pnpm
- Git

§ INSTALLAZIONE

bash
# Clona il repository
git clone https://github.com/username/repo.git
cd repo

# Installa le dipendenze
npm install

# Avvia l'ambiente di sviluppo
npm run dev

Visita http://localhost:3000 nel browser.

⚙️ Configurazione

Crea un file .env.local nella root del progetto:

env
# Database
DATABASE_URL="postgresql://..."

# Autenticazione
NEXTAUTH_SECRET="your-secret-here"
NEXTAUTH_URL="http://localhost:3000"

# API
API_BASE_URL="http://localhost:3000/api"
📁 Struttura del Progetto
text
src/
├── app/              # App Router di Next.js 14
├── components/       # Componenti React
├── lib/             # Utility e configurazioni
├── types/           # TypeScript definitions
├── hooks/           # Custom React hooks
└── styles/          # Stili globali
🧪 Testing
bash
# Esegui i test
npm run test

# Test con watch mode
npm run test:watch

# Test di integrazione
npm run test:e2e
📦 Build e Deployment
bash
# Build per produzione
npm run build

# Avvia in produzione
npm start
🤝 Contribuire

Vedi CONTRIBUTING.md per linee guida dettagliate.

Fork il progetto

Crea un branch (git checkout -b feature/AmazingFeature)

Commit (git commit -m 'Add AmazingFeature')

Push (git push origin feature/AmazingFeature)

Apri una Pull Request

📄 Licenza

Distribuito sotto licenza MIT. Vedi LICENSE per dettagli.

📞 Supporto

Issue Tracker

Discussions

text

### Badges Standards
<!-- Shields.io badges -->
![Version](https://img.shields.io/badge/version-1.0.0-blue)
![Next.js](https://img.shields.io/badge/Next.js-14.0.0-black?logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5.3-blue?logo=typescript)
![React](https://img.shields.io/badge/React-18.2-blue?logo=react)
![Testing](https://img.shields.io/badge/tests-95%25%20✓-green)
![Coverage](https://img.shields.io/badge/coverage-92%25-green)
![License](https://img.shields.io/badge/license-MIT-green)
![CI](https://github.com/user/repo/actions/workflows/ci.yml/badge.svg)
![Deploy](https://github.com/user/repo/actions/workflows/deploy.yml/badge.svg)
§ API DOCUMENTATION
JSDoc/TSDoc Usage
typescript
/**
 * Recupera un utente per ID dall'API
 * 
 * @template T - Tipo di dato restituito
 * @param {string} userId - ID univoco dell'utente
 * @param {RequestInit} [options] - Opzioni di fetch aggiuntive
 * @returns {Promise<UserResponse<T>>} Promise con i dati dell'utente
 * @throws {ApiError} Se la richiesta fallisce
 * @example
 * const user = await getUser('123', { cache: 'no-store' });
 * 
 * @since 1.2.0
 * @deprecated Utilizzare `fetchUser` invece. Sarà rimosso nella v2.0.0
 */
export async function getUser<T = User>(
  userId: string, 
  options?: RequestInit
): Promise<UserResponse<T>> {
  // Implementazione
}
Function Documentation Template
typescript
/**
 * Calcola il prezzo totale con tasse e sconti
 * 
 * @param {number} basePrice - Prezzo base del prodotto
 * @param {TaxOptions} taxOptions - Opzioni per il calcolo delle tasse
 * @param {Discount[]} [discounts] - Array di sconti applicabili (opzionale)
 * @param {CalculationOptions} [options] - Opzioni aggiuntive di calcolo
 * 
 * @returns {CalculatedPrice} Oggetto con breakdown del prezzo
 * 
 * @example
 * ```typescript
 * const price = calculateTotalPrice(100, {
 *   taxRate: 0.22,
 *   includeVAT: true
 * }, [
 *   { type: 'percentage', value: 10 }
 * ]);
 * ```
 * 
 * @see {@link TaxOptions} per le opzioni tasse
 * @see {@link calculateSubtotal} per calcoli parziali
 * 
 * @category Pricing
 * @version 2.1.0
 */
export function calculateTotalPrice(
  basePrice: number,
  taxOptions: TaxOptions,
  discounts?: Discount[],
  options?: CalculationOptions
): CalculatedPrice {
  // Implementazione
}
Type Documentation
typescript
/**
 * Configurazione per l'autenticazione dell'applicazione
 * 
 * @remarks
 * Questo tipo definisce tutte le opzioni necessarie per configurare
 * il sistema di autenticazione NextAuth.js.
 * 
 * @property {string} secret - Segreto per la crittografia delle sessioni
 * @property {Provider[]} providers - Array di provider OAuth abilitati
 * @property {Adapter} [adapter] - Adapter per il database (opzionale)
 * @property {PagesOptions} pages - Personalizzazione delle pagine di auth
 * 
 * @example
 * ```typescript
 * const authConfig: AuthConfig = {
 *   secret: process.env.AUTH_SECRET,
 *   providers: [GoogleProvider({ clientId: '...' })],
 *   pages: { signIn: '/login' }
 * };
 * ```
 */
export interface AuthConfig {
  secret: string;
  providers: Provider[];
  adapter?: Adapter;
  pages: PagesOptions;
  callbacks?: CallbacksOptions;
}
Deprecation Notices
typescript
/**
 * @deprecated Utilizzare `useNewHook` invece. 
 * Questa hook sarà rimossa nella versione 3.0.0.
 * 
 * Migrazione:
 * ```typescript
 * // Vecchio
 * const { data } = useOldHook();
 * 
 * // Nuovo
 * const { data } = useNewHook();
 * ```
 * 
 * @removalVersion 3.0.0
 */
export function useOldHook() {
  console.warn('useOldHook è deprecato. Usare useNewHook invece.');
  return useNewHook();
}
§ COMPONENT DOCUMENTATION
Storybook Setup (.storybook/main.ts)
typescript
import type { StorybookConfig } from '@storybook/nextjs';

const config: StorybookConfig = {
  stories: [
    '../src/components/**/*.mdx',
    '../src/components/**/*.stories.@(js|jsx|ts|tsx)',
    '../src/app/**/*.stories.@(js|jsx|ts|tsx)'
  ],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/addon-interactions',
    '@storybook/addon-a11y',
    '@storybook/addon-coverage',
    'storybook-addon-next-router'
  ],
  framework: {
    name: '@storybook/nextjs',
    options: {},
  },
  docs: {
    autodocs: 'tag',
    defaultName: 'Documentazione'
  },
  staticDirs: ['../public'],
  webpackFinal: async (config) => {
    // Configurazione personalizzata
    return config;
  },
};

export default config;
Stories Writing (Button.stories.tsx)
typescript
import type { Meta, StoryObj } from '@storybook/react';
import { fn } from '@storybook/test';
import { Button } from './Button';

/**
 * Componente Button riutilizzabile con varianti e dimensioni
 * 
 * Usare per azioni primarie, secondarie e terziarie nell'interfaccia.
 */
const meta = {
  title: 'Components/UI/Button',
  component: Button,
  parameters: {
    layout: 'centered',
    docs: {
      description: {
        component: 'Un componente pulsante accessibile con supporto per loading stati e icona.',
      },
    },
    a11y: {
      config: {
        rules: [
          {
            id: 'color-contrast',
            enabled: false, // Disabilita regola specifica per questo componente
          },
        ],
      },
    },
  },
  tags: ['autodocs', 'ui', 'button'],
  argTypes: {
    variant: {
      control: 'select',
      options: ['primary', 'secondary', 'outline', 'ghost'],
      description: 'Variante visiva del bottone',
      table: {
        type: { summary: 'string' },
        defaultValue: { summary: 'primary' },
      },
    },
    size: {
      control: 'radio',
      options: ['sm', 'md', 'lg'],
      description: 'Dimensione del bottone',
    },
    disabled: {
      control: 'boolean',
      description: 'Stato di disabilitazione',
    },
    isLoading: {
      control: 'boolean',
      description: 'Mostra spinner di loading',
    },
    onClick: {
      action: 'clicked',
      description: 'Callback al click',
    },
  },
  args: {
    onClick: fn(),
  },
} satisfies Meta<typeof Button>;

export default meta;
type Story = StoryObj<typeof meta>;

/**
 * Bottone primario per azioni principali
 */
export const Primary: Story = {
  args: {
    variant: 'primary',
    children: 'Button',
  },
};

/**
 * Bottone in stato di loading
 */
export const Loading: Story = {
  args: {
    variant: 'primary',
    isLoading: true,
    children: 'Loading...',
  },
};

/**
 * Bottone disabilitato
 */
export const Disabled: Story = {
  args: {
    variant: 'primary',
    disabled: true,
    children: 'Disabled',
  },
};

/**
 * Bottone con icona
 */
export const WithIcon: Story = {
  args: {
    variant: 'secondary',
    children: 'Download',
    icon: 'download',
  },
  parameters: {
    docs: {
      storyDescription: 'Utilizzare per azioni con icona rappresentativa.',
    },
  },
};
Docs Addon Configuration
typescript
// .storybook/preview.tsx
import type { Preview } from '@storybook/react';
import { withThemeByClassName } from '@storybook/addon-themes';
import '../src/app/globals.css';

const preview: Preview = {
  parameters: {
    actions: { argTypesRegex: '^on[A-Z].*' },
    controls: {
      matchers: {
        color: /(background|color)$/i,
        date: /Date$/i,
      },
      expanded: true, // Espande i controlli di default
    },
    docs: {
      toc: true, // Tabella dei contenuti
      source: {
        state: 'open', // Mostra codice sorgente aperto
      },
    },
    backgrounds: {
      default: 'light',
      values: [
        { name: 'light', value: '#ffffff' },
        { name: 'dark', value: '#1a1a1a' },
        { name: 'gray', value: '#f5f5f5' },
      ],
    },
    viewport: {
      viewports: {
        mobile: {
          name: 'Mobile',
          styles: { width: '375px', height: '667px' },
        },
        tablet: {
          name: 'Tablet',
          styles: { width: '768px', height: '1024px' },
        },
        desktop: {
          name: 'Desktop',
          styles: { width: '1440px', height: '900px' },
        },
      },
    },
  },
  decorators: [
    withThemeByClassName({
      themes: {
        light: 'light',
        dark: 'dark',
      },
      defaultTheme: 'light',
    }),
    (Story) => (
      <div style={{ margin: '2em', fontFamily: 'system-ui' }}>
        <Story />
      </div>
    ),
  ],
};

export default preview;
Accessibility Addon
typescript
// .storybook/test-runner.ts
import { injectAxe, checkA11y } from 'axe-playwright';

module.exports = {
  async preRender(page) {
    await injectAxe(page);
  },
  async postRender(page) {
    await checkA11y(page, '#storybook-root', {
      detailedReport: true,
      detailedReportOptions: {
        html: true,
      },
    });
  },
};

// In package.json
{
  "scripts": {
    "test:a11y": "storybook test --runner a11y"
  }
}
§ ARCHITECTURE DOCS
System Overview Template
markdown
# Panoramica del Sistema

## Scopo
Descrizione ad alto livello del sistema, obiettivi e scope.

## Architettura Tecnica
- **Framework**: Next.js 14 (App Router)
- **Linguaggio**: TypeScript 5.x
- **Database**: PostgreSQL + Prisma ORM
- **Cache**: Redis / Upstash
- **Autenticazione**: NextAuth.js v5
- **Styling**: Tailwind CSS + CSS Modules
- **Testing**: Jest + React Testing Library + Cypress

§ PATTERN ARCHITETTURALI
- Feature-based organization
- Server Components per data fetching
- Client Components per interattività
- API Routes per backend logic
- Middleware per autenticazione e routing

§ PRINCIPI DI DESIGN
1. **Server First**: Prioritizzare Server Components
2. **Type Safety**: TypeScript in tutta la codebase
3. **Performance**: Ottimizzazione automatica di Next.js
4. **Accessibilità**: WCAG 2.1 AA compliance
5. **Security**: Best practices OWASP
Component Diagram (Mermaid)
markdown
mermaid
graph TB
    subgraph "Client Side"
        CC[Client Components]
        CH[Custom Hooks]
        CS[Client State - Zustand]
    end
    
    subgraph "Server Side"
        SC[Server Components]
        AP[API Routes]
        MW[Middleware]
        AL[Server Actions]
    end
    
    subgraph "Data Layer"
        DB[(PostgreSQL)]
        PR[Prisma ORM]
        CA[(Redis Cache)]
    end
    
    subgraph "External Services"
        AUTH[Auth Provider]
        CDN[CDN]
        EMAIL[Email Service]
    end
    
    CC -->|props| SC
    SC -->|fetch| AP
    AP --> PR
    PR --> DB
    MW --> AUTH
    AL --> EMAIL
    CS --> CA
Data Flow Diagram
markdown
sequenceDiagram
    participant U as Utente
    participant B as Browser
    participant MW as Middleware
    participant SC as Server Component
    participant API as API Route
    participant DB as Database
    participant EXT as External API
    
    U->>B: Naviga a /dashboard
    B->>MW: Request
    MW->>MW: Verifica auth token
    MW->>SC: Render pagina
    
    SC->>API: GET /api/user
    API->>DB: Query user data
    DB->>API: Return data
    API->>SC: JSON response
    
    SC->>EXT: Fetch analytics
    EXT->>SC: Analytics data
    
    SC->>B: HTML + CSS + JS
    B->>U: Visualizza dashboard
    
    U->>B: Clica "Aggiorna"
    B->>API: PUT /api/data
    API->>DB: Update record
    API->>B: Success response
    B->>U: Conferma aggiornamento
Dependency Graph
json
// .dependency-cruiser.json
{
  "forbidden": [
    {
      "name": "no-circular",
      "severity": "error",
      "from": {},
      "to": { "circular": true }
    },
    {
      "name": "no-server-in-client",
      "comment": "Server modules non dovrebbero importare client modules",
      "severity": "error",
      "from": { "path": "src/app/.*" },
      "to": { "path": "\\.client\\.(ts|tsx|js|jsx)$" }
    },
    {
      "name": "no-external-in-app",
      "comment": "App Router dovrebbe usare solo internal modules",
      "severity": "warn",
      "from": { "path": "src/app" },
      "to": { "path": "src/components/ui" }
    }
  ],
  "options": {
    "doNotFollow": "node_modules",
    "tsConfig": {
      "fileName": "tsconfig.json"
    },
    "tsPreCompilationDeps": true
  }
}
§ ADR (Architecture Decision Records)
ADR Template
markdown
# [ADR-001]: Scelta di Next.js App Router vs Pages Router

## Stato
Accettato | In Revisione | Deprecato | Sostituito da [ADR-002]

## Data
2024-01-15

§ DECISORI
- Marco Rossi (Tech Lead)
- Sara Bianchi (Senior Developer)
- Luca Verdi (Product Manager)

§ CONTESTO
Il progetto richiede:
- Miglior performance SEO
- Streaming server components
- Layouts annidati
- Loading states built-in
- Simplified data fetching

Opzioni considerate:
1. **App Router (Next.js 14)**: Nuovo paradigma con React Server Components
2. **Pages Router**: Approccio tradizionale ben consolidato
3. **Remix.run**: Alternativa con focus UX

§ DECISIONE
Adottare **Next.js App Router** per:

**Vantaggi:**
- Server Components di default
- Migliore performance out-of-the-box
- Code splitting automatico
- Streaming e Suspense integrati
- Supporto React 18 features
- Community e backing Vercel

**Svantaggi accettati:**
- Learning curve per team
- Alcune librerie non ancora compatibili
- Documentazione in evoluzione

§ CONSEGUENZE
§ POSITIVE
- Performance migliorata del 40% in lighthouse
- Bundle size ridotto del 30%
- Developer experience migliorata
- Allineamento con roadmap Next.js

§ NEGATIVE
- Riscrittura di alcuni componenti
- Training per team (2 settimane)
- Monitoring aggiuntivo necessario

§ NEUTRAL
- Supporto sia Server che Client Components
- Backward compatibility con Pages Router

§ LINK CORRELATI
- [Next.js App Router Docs](https://nextjs.org/docs/app)
- [RFC Server Components](https://github.com/reactjs/rfcs/pull/188)
- [Migration Guide](/docs/migration/app-router.md)
- [Performance Benchmark](/docs/benchmarks/app-vs-pages.md)
Decision Tracking Table
markdown
| ADR | Titolo | Stato | Data | Decisori | Link |
|-----|--------|-------|------|----------|------|
| 001 | App Router vs Pages Router | ✅ Accettato | 2024-01-15 | Tech Lead, Senior Dev | [/docs/adr/001.md] |
| 002 | State Management: Zustand vs Redux | ✅ Accettato | 2024-01-20 | Frontend Team | [/docs/adr/002.md] |
| 003 | Database: PostgreSQL vs MongoDB | ✅ Accettato | 2024-01-25 | Backend Team | [/docs/adr/003.md] |
| 004 | Styling: Tailwind vs Styled Components | 🔄 In Revisione | 2024-02-01 | UI/UX Team | [/docs/adr/004.md] |
| 005 | Testing Strategy | 📝 Proposed | 2024-02-05 | QA Lead | [/docs/adr/005.md] |
ADR Status Lifecycle
Diagramma
Codice
Schermo intero

Accettata

Rifiutata

Sì

No

Proposta

Sotto Revisione

Decisione

Accettato

Rifiutato

Implementato

Necessario Cambiamento?

Deprecato

Stabile

Sostituito

§ CHANGELOG
Keep a Changelog Format
markdown
# Changelog

Tutti i cambiamenti notevoli a questo progetto saranno documentati in questo file.

Il formato è basato su [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
e questo progetto aderisce al [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

§ ADDED
- Supporto per dark mode nei componenti UI
- Nuovo hook `useLocalStorage` per state persistence
- Integration test con Playwright

§ CHANGED
- Aggiornato Next.js da 13.4 a 14.0.0
- Migliorata performance di data fetching del 25%
- Refactor cartella `lib/utils` in moduli separati

§ DEPRECATED
- `OldComponent` - da rimuovere in v2.0.0
- API endpoint `/api/v1/users` - migrare a `/api/v2/users`

§ FIXED
- Memory leak nei Server Components (#234)
- Bug di hydration in Safari (#189)
- Type safety in `useQuery` hook (#201)

§ SECURITY
- Aggiornate dipendenze con vulnerabilità
- Aggiunto CSP headers in middleware

§ [1.2.0] - 2024-01-20

§ ADDED
- **Feature**: Paginazione avanzata con infinite scroll
- **Component**: `DataTable` con sorting e filtering
- **API**: Nuovo endpoint `/api/analytics`
- **Config**: Supporto per multiple environments

§ CHANGED
- **Performance**: Lazy loading per immagini sopra la fold
- **Accessibility**: Migliorato contrasto colori (WCAG AA)
- **DX**: Migliori error messages in dev mode

§ FIXED
- Cross-browser issue con CSS Grid
- Race condition in authentication flow
- Incorrect cache invalidation

§ MIGRATION NOTES
Per migrare da v1.1.0 a v1.2.0:

1. Aggiornare dipendenze:
bash
npm install next@14.0.0 react@18.2.0

Eseguire migration script:

bash
npm run db:migrate

Aggiornare environment variables:

env
# Aggiungere
NEXT_PUBLIC_ANALYTICS_ID="your-id"
[1.1.0] - 2023-12-15
[1.0.0] - 2023-11-10
text

### Automatic Generation (package.json)
{
  "scripts": {
    "changelog": "auto-changelog --template keepachangelog --commit-limit false --unreleased",
    "release:patch": "npm version patch && npm run changelog && git add CHANGELOG.md",
    "release:minor": "npm version minor && npm run changelog && git add CHANGELOG.md",
    "release:major": "npm version major && npm run changelog && git add CHANGELOG.md"
  },
  "devDependencies": {
    "auto-changelog": "^2.4.0",
    "standard-version": "^9.5.0"
  }
}
Version Linking
markdown
## Versioni

- [v2.0.0 (Current)](/docs/releases/2.0.0.md)
- [v1.2.0](/docs/releases/1.2.0.md) | [Migration Guide](/docs/migrations/1.2.0.md)
- [v1.1.0](/docs/releases/1.1.0.md) | [Breaking Changes](/docs/migrations/1.1.0.md#breaking)
- [v1.0.0](/docs/releases/1.0.0.md) | [Initial Release](/docs/releases/1.0.0.md)

§ ROADMAP

§ Q1 2024 (V2.0.0)
- [ ] Internationalization (i18n)
- [ ] PWA support
- [ ] Real-time updates

§ Q2 2024 (V2.1.0)
- [ ] AI integration
- [ ] Advanced analytics
- [ ] Mobile app (React Native)
§ INLINE DOCUMENTATION
Code Comments Guidelines
typescript
// ❌ Cattivo - Commento ovvio
// Incrementa i di 1
i++;

// ✅ Buono - Commento che spiega il perché
// Aggiusta l'indice per l'array 1-based dell'API legacy
i++;

// ❌ Troppo commento
/*
 * Funzione che salva l'utente
 * @param user L'utente da salvare
 * @returns Promise con risultato
 */
async function saveUser(user: User) {}

// ✅ Documentazione concisa
/** Salva l'utente nel database e aggiorna la cache */
async function saveUser(user: User): Promise<SaveResult> {}

/**
 * Calcola lo sconto progressivo basato sul volume d'acquisto
 * 
 * NOTA: Questa logica implementa la formula aziendale:
 * - Prima tranche: 5% su primi 1000€
 * - Seconda tranche: 10% su successivi 2000€
 * - Terza tranche: 15% su eccedenza
 * 
 * @see Business Rules Doc: BR-2023-45
 */
function calculateVolumeDiscount(amount: number): number {
  // Implementazione complessa con commenti esplicativi
  if (amount <= 1000) {
    return amount * 0.05;
  } else if (amount <= 3000) {
    // Calcola tranche separatamente
    const firstTranche = 1000 * 0.05;
    const secondTranche = (amount - 1000) * 0.10;
    return firstTranche + secondTranche;
  } else {
    // Gestisci tutte e tre le tranches
    const firstTranche = 1000 * 0.05;
    const secondTranche = 2000 * 0.10;
    const thirdTranche = (amount - 3000) * 0.15;
    return firstTranche + secondTranche + thirdTranche;
  }
}
TODO/FIXME Tracking
typescript
// TODO Template
// TODO(marco, 2024-Q2): Implementare cache invalidation
// TODO(#123): Aggiungere test per edge case
// TODO(P2): Ottimizzare query per grandi dataset

// FIXME Template  
// FIXME: Workaround per bug di Safari, rimuovere quando risolto
// FIXME(#456): Race condition in produzione
// FIXME(high): Memory leak in dev mode

// HACK Template
// HACK: Soluzione temporanea per compatibilità API v1
// HACK: Bypassa validazione per MVP

// NOTE Template
// NOTE: Importante per il corretto funzionamento di X
// NOTE: Non modificare senza consultare team backend

// PERF Template
// PERF: Ottimizzare questo loop O(n^2)
// PERF: Considerare memoization

// Esempio completo
export async function fetchUserData(userId: string) {
  // FIXME(#789): Timeout occasionali sotto carico
  // TODO(perf): Implementare cache layer Redis
  // HACK: Doppio fetch per bug API esterna
  // NOTE: L'endpoint ritorna dati in formato legacy
  
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    // PERF: Validazione sincrona, considerare web worker
    const data = await validateResponse(response);
    
    return data;
  } catch (error) {
    // TODO(#234): Migliorare error handling
    console.error('Fetch failed:', error);
    throw error;
  }
}
Complex Logic Explanation
typescript
/**
 * Processa pagamenti con circuiti multipli e fallback
 * 
 * Questa funzione implementa una strategia di fallback complessa:
 * 1. Prima prova con Stripe (circuito primario)
 * 2. Se fallisce, prova con PayPal (circuito secondario)
 * 3. Se PayPal è offline, usa sistema legacy (fallback)
 * 4. Se tutto fallisce, crea pending payment
 * 
 * Sequence Diagram:
 * 
 * Client -> Stripe: Charge
 * alt Successo
 *   Stripe -> DB: Update payment
 * else Fallimento
 *   Client -> PayPal: Charge
 *   alt Successo
 *     PayPal -> DB: Update payment
 *   else Fallimento
 *     Client -> Legacy: Process
 *     Legacy -> DB: Create pending
 *   end
 * end
 * 
 * @complexity O(n) dove n è il numero di circuiti
 * @idempotencyKey Per prevenire duplicati
 */
async function processPayment(
  payment: PaymentRequest,
  circuits: PaymentCircuit[],
  idempotencyKey: string
): Promise<PaymentResult> {
  // Logica complessa con commenti step-by-step
  for (const circuit of circuits) {
    try {
      // Step 1: Validazione circuito
      if (!circuit.isAvailable) {
        // NOTE: Saltiamo circuiti offline
        continue;
      }
      
      // Step 2: Tentativo pagamento
      const result = await attemptPayment(circuit, payment, idempotencyKey);
      
      // Step 3: Gestione risultato
      if (result.success) {
        // SUCCESS: Aggiorna stato e ritorna
        await updatePaymentStatus(payment.id, 'completed', circuit.name);
        return { success: true, circuit: circuit.name };
      }
      
      // Step 4: Logica di fallback per errori specifici
      if (shouldRetryWithDifferentCircuit(result.error)) {
        // CONTINUA: Prova circuito successivo
        console.warn(`Circuit ${circuit.name} failed, trying next...`);
        continue;
      }
      
      // FATAL: Errore irrecoverable
      throw new PaymentError(result.error);
      
    } catch (error) {
      // ERROR: Gestione eccezioni per circuito
      if (isLastCircuit(circuit, circuits)) {
        // ULTIMA SPERANZA: Sistema legacy
        return await fallbackToLegacySystem(payment);
      }
      // Altrimenti continua al prossimo circuito
    }
  }
  
  // Se tutti i circuiti falliscono
  return createPendingPayment(payment);
}
Edge Case Documentation
typescript
interface UserPreferences {
  theme: 'light' | 'dark' | 'auto';
  notifications: boolean;
  language: string;
}

/**
 * Applica le preferenze utente con gestione edge cases
 * 
 * Edge Cases gestiti:
 * 1. Preferenze null/undefined → defaults
 * 2. Lingua non supportata → fallback a 'en'
 * 3. Tema 'auto' → detect system preference
 * 4. Notifiche non permesse → silent fallback
 * 5. Storage quota exceeded → memory fallback
 * 
 * @throws {QuotaExceededError} Se localStorage è pieno
 * @returns {boolean} Successo dell'operazione
 */
function applyUserPreferences(prefs: Partial<UserPreferences> | null): boolean {
  // Edge Case 1: Input null o undefined
  if (!prefs) {
    prefs = getDefaultPreferences();
  }
  
  // Edge Case 2: Lingua non supportata
  const supportedLanguages = ['en', 'it', 'es', 'fr', 'de'];
  const language = prefs.language || 'en';
  if (!supportedLanguages.includes(language)) {
    console.warn(`Language ${language} not supported, falling back to 'en'`);
    prefs.language = 'en';
  }
  
  // Edge Case 3: Tema 'auto'
  if (prefs.theme === 'auto') {
    prefs.theme = window.matchMedia('(prefers-color-scheme: dark)').matches 
      ? 'dark' 
      : 'light';
  }
  
  // Edge Case 4: Notifiche non permesse
  if (prefs.notifications && !('Notification' in window)) {
    console.warn('Notifications API not available');
    prefs.notifications = false;
  }
  
  try {
    // Edge Case 5: Storage quota
    localStorage.setItem('userPrefs', JSON.stringify(prefs));
    return true;
  } catch (error) {
    if (error.name === 'QuotaExceededError') {
      // Fallback in-memory
      memoryCache.set('userPrefs', prefs);
      console.error('LocalStorage full, using memory cache');
      return true;
    }
    throw error;
  }
}
§ ONBOARDING DOCS
Development Setup Guide
markdown
# Guida Setup Sviluppatore

## Prerequisiti

### Software Richiesto
- Node.js 18.17+ [Download](https://nodejs.org/)
- Git 2.40+ [Download](https://git-scm.com/)
- Docker Desktop (opzionale) [Download](https://www.docker.com/)
- VS Code (consigliato) [Download](https://code.visualstudio.com/)

§ ESTENSIONI VS CODE CONSIGLIATE
json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "bradlc.vscode-tailwindcss",
    "ms-playwright.playwright",
    "yoavbls.pretty-ts-errors"
  ]
}
Setup Locale Passo-Passo
1. Clona Repository
bash
git clone https://github.com/company/project.git
cd project
2. Installa Dipendenze
bash
npm install
# oppure
yarn install
# oppure
pnpm install
3. Configura Environment
bash
cp .env.example .env.local
# Modifica .env.local con i tuoi valori
4. Setup Database
bash
# Con Docker (consigliato)
docker-compose up -d postgres redis

# Oppure localmente
npm run db:setup
npm run db:migrate
npm run db:seed
5. Avvia Sviluppo
bash
npm run dev
# Visita http://localhost:3000
6. Verifica Installazione
bash
npm