<!-- AUDIT: Catalogo privo di code block. Necessita espansione con snippet TypeScript/Next.js 14. -->
# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO QUICK START v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
# GUIDA RAPIDA - INIZIA IN 5 MINUTI
# Questo file è il punto di ingresso per usare il sistema di cataloghi.
#
# ═══════════════════════════════════════════════════════════════════════════════

# ═══════════════════════════════════════════════════════════════════════════════
# PANORAMICA SISTEMA
# ═══════════════════════════════════════════════════════════════════════════════

"""
COSA HAI A DISPOSIZIONE:

┌─────────────────────────────────────────────────────────────────────────────┐
│  11 CATALOGHI | 2.4 MB | 58,000+ RIGHE DI DOCUMENTAZIONE                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  📋 INTERFACCIA-PROMPT     → Come usare il sistema (leggi questo)           │
│  🎨 DESIGN-REFERENCE       → Stili e layout pagine                          │
│  🎯 DESIGN-TOKEN-SYSTEM    → Colori, typography, spacing                    │
│  🧩 UI-PATTERN-PRIMITIVI   → 78 componenti pronti (Button, Card, Table...)  │
│  📝 REQUISITI-FUNZIONALI   → Cosa costruire (user stories, business rules)  │
│  🏗️ REQUISITI-ARCHITETTURA → Come strutturare (layer, pattern, security)    │
│  💾 DATA-MODEL             → Schema database (DynamoDB, PostgreSQL)         │
│  🔌 API                    → 119 endpoint REST documentati                  │
│  💻 CODICE                 → Template implementazione TypeScript/Python     │
│  ☁️ AWS-DETERMINISTICO     → Infrastruttura AWS Free Tier                   │
│  📖 MASTER-INDEX           → Indice completo e statistiche                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

PRECISIONE STIMATA: 80-92% (progetti standard con buon discovery)
RISPARMIO TEMPO: 60-70% rispetto a partire da zero
"""

# ═══════════════════════════════════════════════════════════════════════════════
# STEP 1: CHE TIPO DI PROGETTO?
# ═══════════════════════════════════════════════════════════════════════════════

CATEGORIE = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ SELEZIONA LA TUA CATEGORIA:                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🛒 E-COMMERCE         Vendita prodotti, carrello, checkout                 │
│     → Stile: Airbnb/Shopify                                                 │
│     → Cataloghi: DESIGN-REFERENCE, UI-PATTERN (E-commerce), DATA-MODEL      │
│                                                                             │
│  📊 SAAS               Dashboard, tool, subscription                        │
│     → Stile: Stripe/Linear                                                  │
│     → Cataloghi: DESIGN-REFERENCE, UI-PATTERN (SaaS), API                   │
│                                                                             │
│  🏪 MARKETPLACE        Multi-vendor, commissioni                            │
│     → Stile: Airbnb                                                         │
│     → Cataloghi: REQUISITI-FUNZIONALI (Marketplace), DATA-MODEL             │
│                                                                             │
│  💬 SOCIAL             Community, feed, messaggi                            │
│     → Stile: Slack/Notion                                                   │
│     → Cataloghi: UI-PATTERN (Social), DATA-MODEL (Social entities)          │
│                                                                             │
│  🏥 HEALTHCARE         HIPAA compliance, dati sensibili                     │
│     → Stile: Notion (accessibility)                                         │
│     → Cataloghi: REQUISITI-ARCHITETTURA (HIPAA), UI-PATTERN (Healthcare)    │
│                                                                             │
│  💳 FINTECH            Banking, payments, PCI-DSS                           │
│     → Stile: Stripe/Vercel                                                  │
│     → Cataloghi: REQUISITI-ARCHITETTURA (PCI), UI-PATTERN (FinTech)         │
│                                                                             │
│  📝 CONTENT/BLOG       Pubblicazione, CMS                                   │
│     → Stile: Notion                                                         │
│     → Cataloghi: UI-PATTERN (Content), DESIGN-REFERENCE (Blog layout)       │
│                                                                             │
│  🚀 LANDING PAGE       Marketing, conversioni                               │
│     → Stile: Stripe/Vercel                                                  │
│     → Cataloghi: DESIGN-REFERENCE (Homepage layouts), UI-PATTERN            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# STEP 2: QUALE STILE VISIVO?
# ═══════════════════════════════════════════════════════════════════════════════

STILI = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ SCEGLI IL TUO STILE (vedi DESIGN-REFERENCE per dettagli):                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  STRIPE      Professional, gradient, illustrations                          │
│              Colore: Viola (#635BFF) | Target: B2B, Enterprise              │
│                                                                             │
│  LINEAR      Dark mode, minimal, developer-friendly                         │
│              Colore: Viola chiaro (#8B5CF6) | Target: Tech, SaaS            │
│                                                                             │
│  NOTION      Clean, whitespace, content-first                               │
│              Colore: Nero (#191919) | Target: Productivity, Docs            │
│                                                                             │
│  VERCEL      High contrast, terminal aesthetic, tech                        │
│              Colore: Bianco/Nero | Target: Developer tools                  │
│                                                                             │
│  AIRBNB      Friendly, visual, imagery-driven                               │
│              Colore: Rosa (#FF385C) | Target: Consumer, Travel              │
│                                                                             │
│  SHOPIFY     Commerce-focused, professional                                 │
│              Colore: Verde (#008060) | Target: E-commerce, Retail           │
│                                                                             │
│  SLACK       Colorful, playful, collaborative                               │
│              Colore: Viola (#4A154B) | Target: Communication, Teams         │
│                                                                             │
│  GITHUB      Dense, functional, developer-native                            │
│              Colore: Verde (#238636) | Target: Code, Open source            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# STEP 3: PROMPT RAPIDO PER RALPH
# ═══════════════════════════════════════════════════════════════════════════════

PROMPT_TEMPLATE = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ TEMPLATE PROMPT MINIMO (copia e personalizza):                              │
├─────────────────────────────────────────────────────────────────────────────┤

PROGETTO: [Nome del tuo progetto]
CATEGORIA: [E-Commerce | SaaS | Social | Healthcare | FinTech | Landing]
TIPO: [MVP | Production]

DESCRIZIONE:
[Descrivi in 2-3 frasi cosa deve fare il progetto]

FUNZIONALITÀ CORE (max 5):
1. [Funzionalità 1]
2. [Funzionalità 2]
3. [Funzionalità 3]

STILE VISIVO: [Stripe | Linear | Notion | Vercel | Airbnb | Shopify]

TECH STACK:
- Frontend: Next.js 14 + TypeScript + Tailwind
- Backend: [Next.js API Routes | Express | Lambda]
- Database: [Supabase | DynamoDB | PostgreSQL]
- Hosting: [Vercel | AWS Free Tier]

VINCOLI:
- [es: Solo servizi gratuiti AWS]
- [es: Mobile-first]
- [es: GDPR compliance]

---
ISTRUZIONI PER RALPH:
1. Leggi CATALOGO-DESIGN-REFERENCE per stile [STILE] e layout pagine
2. Leggi CATALOGO-UI-PATTERN-PRIMITIVI per componenti [CATEGORIA]
3. Leggi CATALOGO-DESIGN-TOKEN-SYSTEM per customizzare tokens
4. Segui workflow in CATALOGO-INTERFACCIA-PROMPT-USABILITA
5. Verifica ogni fase prima di procedere

└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# STEP 4: CHECKLIST PRE-SVILUPPO
# ═══════════════════════════════════════════════════════════════════════════════

CHECKLIST = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ CHECKLIST PRIMA DI INIZIARE:                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ENVIRONMENT:                                                               │
│  □ Node.js 20+ installato                                                   │
│  □ pnpm installato (npm install -g pnpm)                                    │
│  □ Git installato e configurato                                             │
│  □ VS Code o IDE preferito                                                  │
│                                                                             │
│  CATALOGHI:                                                                 │
│  □ CATALOGO-DESIGN-REFERENCE disponibile                                    │
│  □ CATALOGO-UI-PATTERN-PRIMITIVI disponibile                                │
│  □ CATALOGO-DESIGN-TOKEN-SYSTEM disponibile                                 │
│  □ CATALOGO-INTERFACCIA-PROMPT-USABILITA disponibile                        │
│                                                                             │
│  REQUISITI:                                                                 │
│  □ Categoria progetto identificata                                          │
│  □ Stile visivo scelto                                                      │
│  □ Funzionalità core definite (max 5)                                       │
│  □ Tech stack deciso                                                        │
│                                                                             │
│  OPZIONALE:                                                                 │
│  □ Screenshot/reference design (se hai preferenze specifiche)               │
│  □ Brand colors (se esistono)                                               │
│  □ Logo (se esiste)                                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# MAPPA RAPIDA: COSA CERCO → DOVE LO TROVO
# ═══════════════════════════════════════════════════════════════════════════════

MAPPA_CATALOGHI = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ CERCO...                           │ LO TROVO IN...                         │
├────────────────────────────────────┼────────────────────────────────────────┤
│ Quale stile usare                  │ DESIGN-REFERENCE → Sezione 1           │
│ Come strutturare una pagina        │ DESIGN-REFERENCE → Sezione 2           │
│ Colori, font, spacing              │ DESIGN-TOKEN-SYSTEM                    │
│ Componente Button                  │ UI-PATTERN-PRIMITIVI → 8.1             │
│ Componente Card                    │ UI-PATTERN-PRIMITIVI → 3.1             │
│ Componente Form/Input              │ UI-PATTERN-PRIMITIVI → 4               │
│ Componente Modal                   │ UI-PATTERN-PRIMITIVI → 6.1             │
│ Componente Table                   │ UI-PATTERN-PRIMITIVI → 7.1             │
│ Pattern E-commerce                 │ UI-PATTERN-PRIMITIVI → Categoria       │
│ Pattern SaaS/Dashboard             │ UI-PATTERN-PRIMITIVI → Categoria       │
│ Responsive design                  │ UI-PATTERN-PRIMITIVI → 9               │
│ Animazioni                         │ UI-PATTERN-PRIMITIVI → 10              │
│ Accessibilità                      │ UI-PATTERN-PRIMITIVI → 11              │
│ Schema database                    │ DATA-MODEL                             │
│ Endpoint API                       │ API                                    │
│ Codice boilerplate                 │ CODICE                                 │
│ Setup AWS                          │ AWS-DETERMINISTICO                     │
│ Domande discovery                  │ INTERFACCIA-PROMPT → 9                 │
│ Troubleshooting                    │ INTERFACCIA-PROMPT → 6                 │
│ Esempi completi                    │ INTERFACCIA-PROMPT → 11                │
│ Snippet codice                     │ INTERFACCIA-PROMPT → 13                │
└────────────────────────────────────┴────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# COMANDI RAPIDI
# ═══════════════════════════════════════════════════════════════════════════════

COMANDI = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ COMANDI SETUP RAPIDO:                                                       │
├─────────────────────────────────────────────────────────────────────────────┤

# Crea nuovo progetto Next.js
pnpm create next-app@latest my-app --typescript --tailwind --eslint --app --src-dir

# Entra nella directory
cd my-app

# Installa dipendenze comuni
pnpm add zod @tanstack/react-query lucide-react clsx tailwind-merge

# Se usi Supabase (auth + database)
pnpm add @supabase/supabase-js @supabase/auth-helpers-nextjs

# Se usi Stripe (payments)  
pnpm add @stripe/stripe-js stripe

# Avvia development
pnpm dev

└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# ESEMPI RAPIDI
# ═══════════════════════════════════════════════════════════════════════════════

ESEMPI = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ ESEMPI PRONTI (vedi INTERFACCIA-PROMPT-USABILITA Sezione 11):               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  🛒 PETSHOP MVP (E-Commerce)                                                │
│     - Catalogo prodotti, carrello, checkout Stripe                          │
│     - Stile: Airbnb | Tech: Next.js + Vercel                                │
│     - Tempo stimato: 4-6 ore                                                │
│                                                                             │
│  📊 TASKFLOW (SaaS Dashboard)                                               │
│     - Progetti, task, team, dashboard                                       │
│     - Stile: Linear | Tech: Next.js + Supabase                              │
│     - Tempo stimato: 8-12 ore                                               │
│                                                                             │
│  🚀 LANDING PAGE                                                            │
│     - Hero, features, pricing, CTA                                          │
│     - Stile: Stripe | Tech: Next.js statico                                 │
│     - Tempo stimato: 2-3 ore                                                │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# NEXT STEPS
# ═══════════════════════════════════════════════════════════════════════════════

NEXT_STEPS = """
┌─────────────────────────────────────────────────────────────────────────────┐
│ PROSSIMI PASSI:                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. Identifica la tua categoria e stile                                     │
│                                                                             │
│  2. Copia il template prompt e personalizzalo                               │
│                                                                             │
│  3. Dai il prompt a Ralph/Claude Code                                       │
│                                                                             │
│  4. Rispondi alle domande di discovery                                      │
│                                                                             │
│  5. Conferma il riepilogo progetto                                          │
│                                                                             │
│  6. Lascia che Ralph crei il progetto fase per fase                         │
│                                                                             │
│  7. Review finale con checklist                                             │
│                                                                             │
│  8. Deploy! 🚀                                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Per domande o problemi: consulta CATALOGO-INTERFACCIA-PROMPT-USABILITA → Sezione 6 (Troubleshooting)
"""

# ═══════════════════════════════════════════════════════════════════════════════
# FINE QUICK START
# ═══════════════════════════════════════════════════════════════════════════════

Catalogo Quick-Start Next.js 14 - Snippets Pronti All'Uso
1. Quick Start: Landing Page (5 file minimi)
// app/layout.tsx
tsx
Copia
Scarica
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { ThemeProvider } from '@/components/theme-provider'
import { Header } from '@/components/header'
import { Footer } from '@/components/footer'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'NextLaunch - Modern Landing Template',
  description: 'A beautifully crafted landing page built with Next.js 14',
}

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode
}>) {
  return (
    <html lang="it" suppressHydrationWarning>
      <body className={inter.className}>
        <ThemeProvider
          attribute="class"
          defaultTheme="light"
          enableSystem
          disableTransitionOnChange
        >
          <Header />
          <main className="min-h-screen">{children}</main>
          <Footer />
        </ThemeProvider>
      </body>
    </html>
  )
}
// app/page.tsx
tsx
Copia
Scarica
import { Button } from '@/components/ui/button'
import { Card, CardContent } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { ArrowRight, CheckCircle, Star } from 'lucide-react'

export default function HomePage() {
  return (
    <div className="flex flex-col">
      {/* Hero Section */}
      <section className="py-20 px-4 sm:px-6 lg:px-8 max-w-7xl mx-auto">
        <div className="text-center">
          <Badge className="mb-4" variant="secondary">
            Nuova Versione Disponibile
          </Badge>
          <h1 className="text-5xl md:text-7xl font-bold tracking-tight mb-6">
            Costruisci il tuo{' '}
            <span className="bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent">
              prodotto digitale
            </span>
          </h1>
          <p className="text-xl text-muted-foreground max-w-3xl mx-auto mb-10">
            Una soluzione completa per startup e imprese. Sviluppato con Next.js 14,
            TypeScript e le migliori tecnologie moderne.
          </p>
          <div className="flex flex-col sm:flex-row gap-4 justify-center">
            <Button size="lg" className="gap-2">
              Inizia Gratuitamente <ArrowRight className="h-4 w-4" />
            </Button>
            <Button size="lg" variant="outline">
              Guarda Demo
            </Button>
          </div>
        </div>
      </section>

      {/* Features Section */}
      <section className="py-16 bg-muted/50">
        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
          <h2 className="text-3xl font-bold text-center mb-12">Funzionalità Principali</h2>
          <div className="grid md:grid-cols-3 gap-8">
            {[
              {
                title: 'Performance Ottimali',
                description: '100/100 Lighthouse score',
                icon: Star,
              },
              {
                title: 'TypeScript Strict',
                description: 'Tipo sicurezza garantita',
                icon: CheckCircle,
              },
              {
                title: 'Responsive Design',
                description: 'Mobile-first approach',
                icon: CheckCircle,
              },
            ].map((feature) => (
              <Card key={feature.title}>
                <CardContent className="pt-6">
                  <feature.icon className="h-12 w-12 text-primary mb-4" />
                  <h3 className="text-xl font-semibold mb-2">{feature.title}</h3>
                  <p className="text-muted-foreground">{feature.description}</p>
                </CardContent>
              </Card>
            ))}
          </div>
        </div>
      </section>
    </div>
  )
}
// app/globals.css
css
Copia
Scarica
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --card: 0 0% 100%;
    --card-foreground: 222.2 84% 4.9%;
    --popover: 0 0% 100%;
    --popover-foreground: 222.2 84% 4.9%;
    --primary: 222.2 47.4% 11.2%;
    --primary-foreground: 210 40% 98%;
    --secondary: 210 40% 96.1%;
    --secondary-foreground: 222.2 47.4% 11.2%;
    --muted: 210 40% 96.1%;
    --muted-foreground: 215.4 16.3% 46.9%;
    --accent: 210 40% 96.1%;
    --accent-foreground: 222.2 47.4% 11.2%;
    --destructive: 0 84.2% 60.2%;
    --destructive-foreground: 210 40% 98%;
    --border: 214.3 31.8% 91.4%;
    --input: 214.3 31.8% 91.4%;
    --ring: 222.2 84% 4.9%;
    --radius: 0.5rem;
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    --card: 222.2 84% 4.9%;
    --card-foreground: 210 40% 98%;
    --popover: 222.2 84% 4.9%;
    --popover-foreground: 210 40% 98%;
    --primary: 210 40% 98%;
    --primary-foreground: 222.2 47.4% 11.2%;
    --secondary: 217.2 32.6% 17.5%;
    --secondary-foreground: 210 40% 98%;
    --muted: 217.2 32.6% 17.5%;
    --muted-foreground: 215 20.2% 65.1%;
    --accent: 217.2 32.6% 17.5%;
    --accent-foreground: 210 40% 98%;
    --destructive: 0 62.8% 30.6%;
    --destructive-foreground: 210 40% 98%;
    --border: 217.2 32.6% 17.5%;
    --input: 217.2 32.6% 17.5%;
    --ring: 212.7 26.8% 83.9%;
  }
}

@layer base {
  * {
    @apply border-border;
  }
  body {
    @apply bg-background text-foreground;
  }
}
// tailwind.config.ts
typescript
Copia
Scarica
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
    './src/**/*.{ts,tsx}',
  ],
  theme: {
    container: {
      center: true,
      padding: '2rem',
      screens: {
        '2xl': '1400px',
      },
    },
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        secondary: {
          DEFAULT: 'hsl(var(--secondary))',
          foreground: 'hsl(var(--secondary-foreground))',
        },
        destructive: {
          DEFAULT: 'hsl(var(--destructive))',
          foreground: 'hsl(var(--destructive-foreground))',
        },
        muted: {
          DEFAULT: 'hsl(var(--muted))',
          foreground: 'hsl(var(--muted-foreground))',
        },
        accent: {
          DEFAULT: 'hsl(var(--accent))',
          foreground: 'hsl(var(--accent-foreground))',
        },
        popover: {
          DEFAULT: 'hsl(var(--popover))',
          foreground: 'hsl(var(--popover-foreground))',
        },
        card: {
          DEFAULT: 'hsl(var(--card))',
          foreground: 'hsl(var(--card-foreground))',
        },
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
      keyframes: {
        'accordion-down': {
          from: { height: '0' },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: '0' },
        },
      },
      animation: {
        'accordion-down': 'accordion-down 0.2s ease-out',
        'accordion-up': 'accordion-up 0.2s ease-out',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
}

export default config
// next.config.ts
typescript
Copia
Scarica
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    domains: ['images.unsplash.com', 'localhost'],
    formats: ['image/avif', 'image/webp'],
  },
  experimental: {
    serverActions: {
      bodySizeLimit: '2mb',
    },
  },
}

export default nextConfig
2. Quick Start: SaaS App (8 file minimi)
// prisma/schema.prisma
prisma
Copia
Scarica
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  passwordHash  String
  emailVerified DateTime?
  image         String?
  role          Role      @default(USER)
  accounts      Account[]
  sessions      Session[]
  subscriptions Subscription[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  @@map("users")
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
  @@map("accounts")
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("sessions")
}

model Subscription {
  id           String   @id @default(cuid())
  userId       String
  plan         Plan     @default(FREE)
  status       SubscriptionStatus @default(ACTIVE)
  stripeId     String?  @unique
  currentPeriodEnd DateTime?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("subscriptions")
}

enum Role {
  USER
  ADMIN
}

enum Plan {
  FREE
  PRO
  ENTERPRISE
}

enum SubscriptionStatus {
  ACTIVE
  CANCELED
  PAST_DUE
}
// lib/auth.ts
typescript
Copia
Scarica
import { NextAuthOptions } from 'next-auth'
import { PrismaAdapter } from '@auth/prisma-adapter'
import { prisma } from '@/lib/prisma'
import CredentialsProvider from 'next-auth/providers/credentials'
import bcrypt from 'bcryptjs'

export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  session: {
    strategy: 'jwt',
  },
  pages: {
    signIn: '/login',
  },
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) {
          return null
        }

        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
        })

        if (!user || !user.passwordHash) {
          return null
        }

        const isPasswordValid = await bcrypt.compare(
          credentials.password,
          user.passwordHash
        )

        if (!isPasswordValid) {
          return null
        }

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          role: user.role,
        }
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      if (user) {
        return {
          ...token,
          id: user.id,
          role: user.role,
        }
      }
      return token
    },
    async session({ session, token }) {
      return {
        ...session,
        user: {
          ...session.user,
          id: token.id,
          role: token.role,
        },
      }
    },
  },
}
// app/dashboard/page.tsx
tsx
Copia
Scarica
'use client'

import { useSession } from 'next-auth/react'
import { redirect } from 'next/navigation'
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Button } from '@/components/ui/button'
import { BarChart3, CreditCard, Users, Activity } from 'lucide-react'

export default function DashboardPage() {
  const { data: session, status } = useSession()

  if (status === 'loading') {
    return <div>Loading...</div>
  }

  if (!session) {
    redirect('/login')
  }

  const stats = [
    {
      title: 'Revenue',
      value: '$45,231.89',
      description: '+20.1% from last month',
      icon: CreditCard,
      color: 'text-green-600',
    },
    {
      title: 'Active Users',
      value: '2,350',
      description: '+180 new users',
      icon: Users,
      color: 'text-blue-600',
    },
    {
      title: 'Conversion Rate',
      value: '12.3%',
      description: '+2.5% from last month',
      icon: BarChart3,
      color: 'text-purple-600',
    },
    {
      title: 'Server Uptime',
      value: '99.9%',
      description: 'All systems operational',
      icon: Activity,
      color: 'text-emerald-600',
    },
  ]

  return (
    <div className="flex-1 space-y-4 p-4 md:p-8 pt-6">
      <div className="flex items-center justify-between space-y-2">
        <h2 className="text-3xl font-bold tracking-tight">Dashboard</h2>
        <div className="flex items-center space-x-2">
          <Button>Download Report</Button>
        </div>
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        {stats.map((stat) => (
          <Card key={stat.title}>
            <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
              <CardTitle className="text-sm font-medium">{stat.title}</CardTitle>
              <stat.icon className={`h-4 w-4 ${stat.color}`} />
            </CardHeader>
            <CardContent>
              <div className="text-2xl font-bold">{stat.value}</div>
              <p className="text-xs text-muted-foreground">{stat.description}</p>
            </CardContent>
          </Card>
        ))}
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-7">
        <Card className="col-span-4">
          <CardHeader>
            <CardTitle>Overview</CardTitle>
          </CardHeader>
          <CardContent>
            <div className="h-[300px] bg-muted/50 rounded flex items-center justify-center">
              <p className="text-muted-foreground">Chart Component</p>
            </div>
          </CardContent>
        </Card>
        <Card className="col-span-3">
          <CardHeader>
            <CardTitle>Recent Activity</CardTitle>
            <CardDescription>Latest updates from your team</CardDescription>
          </CardHeader>
          <CardContent>
            <div className="space-y-4">
              {['User registration', 'Payment received', 'Server update', 'New feature'].map(
                (item) => (
                  <div key={item} className="flex items-center">
                    <div className="ml-4 space-y-1">
                      <p className="text-sm font-medium leading-none">{item}</p>
                      <p className="text-sm text-muted-foreground">2 hours ago</p>
                    </div>
                  </div>
                )
              )}
            </div>
          </CardContent>
        </Card>
      </div>
    </div>
  )
}
// app/api/users/route.ts
typescript
Copia
Scarica
import { NextRequest, NextResponse } from 'next/server'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'

const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(50),
  role: z.enum(['USER', 'ADMIN']).default('USER'),
})

export async function GET(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session || session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    const { searchParams } = new URL(request.url)
    const page = parseInt(searchParams.get('page') || '1')
    const limit = parseInt(searchParams.get('limit') || '10')
    const skip = (page - 1) * limit

    const [users, total] = await Promise.all([
      prisma.user.findMany({
        skip,
        take: limit,
        select: {
          id: true,
          email: true,
          name: true,
          role: true,
          createdAt: true,
        },
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count(),
    ])

    return NextResponse.json({
      users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit),
      },
    })
  } catch (error) {
    console.error('Error fetching users:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}

export async function POST(request: NextRequest) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session || session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    const body = await request.json()
    const validatedData = userSchema.parse(body)

    // Check if user exists
    const existingUser = await prisma.user.findUnique({
      where: { email: validatedData.email },
    })

    if (existingUser) {
      return NextResponse.json(
        { error: 'User already exists' },
        { status: 409 }
      )
    }

    const user = await prisma.user.create({
      data: validatedData,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
      },
    })

    return NextResponse.json(user, { status: 201 })
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      )
    }

    console.error('Error creating user:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
// app/api/users/[id]/route.ts
typescript
Copia
Scarica
import { NextRequest, NextResponse } from 'next/server'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'

const updateUserSchema = z.object({
  name: z.string().min(2).max(50).optional(),
  role: z.enum(['USER', 'ADMIN']).optional(),
})

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session || session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    const user = await prisma.user.findUnique({
      where: { id: params.id },
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
        createdAt: true,
        updatedAt: true,
      },
    })

    if (!user) {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      )
    }

    return NextResponse.json(user)
  } catch (error) {
    console.error('Error fetching user:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}

export async function PUT(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session || session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    const body = await request.json()
    const validatedData = updateUserSchema.parse(body)

    const user = await prisma.user.update({
      where: { id: params.id },
      data: validatedData,
      select: {
        id: true,
        email: true,
        name: true,
        role: true,
      },
    })

    return NextResponse.json(user)
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation error', details: error.errors },
        { status: 400 }
      )
    }

    if (error.code === 'P2025') {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      )
    }

    console.error('Error updating user:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const session = await getServerSession(authOptions)
    
    if (!session || session.user.role !== 'ADMIN') {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      )
    }

    await prisma.user.delete({
      where: { id: params.id },
    })

    return NextResponse.json({ success: true })
  } catch (error) {
    if (error.code === 'P2025') {
      return NextResponse.json(
        { error: 'User not found' },
        { status: 404 }
      )
    }

    console.error('Error deleting user:', error)
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    )
  }
}
// lib/prisma.ts
typescript
Copia
Scarica
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
// middleware.ts
typescript
Copia
Scarica
import { withAuth } from 'next-auth/middleware'
import { NextResponse } from 'next/server'

export default withAuth(
  function middleware(req) {
    const token = req.nextauth.token
    const isAdmin = token?.role === 'ADMIN'
    const isAuthPage = req.nextUrl.pathname.startsWith('/login') || 
                      req.nextUrl.pathname.startsWith('/register')

    // Redirect to dashboard if logged in and trying to access auth pages
    if (isAuthPage && token) {
      return NextResponse.redirect(new URL('/dashboard', req.url))
    }

    // Protect admin routes
    if (req.nextUrl.pathname.startsWith('/admin') && !isAdmin) {
      return NextResponse.redirect(new URL('/dashboard', req.url))
    }

    return NextResponse.next()
  },
  {
    callbacks: {
      authorized: ({ token }) => !!token,
    },
  }
)

export const config = {
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/api/admin/:path*',
    '/login',
    '/register',
  ],
}
// app/api/auth/[...nextauth]/route.ts
typescript
Copia
Scarica
import NextAuth from 'next-auth'
import { authOptions } from '@/lib/auth'

const handler = NextAuth(authOptions)

export { handler as GET, handler as POST }
3. Quick Start: E-commerce (10 file minimi)
// prisma/schema-ecommerce.prisma
prisma
Copia
Scarica
model Product {
  id          String   @id @default(cuid())
  name        String
  description String   @db.Text
  price       Decimal  @db.Decimal(10, 2)
  sku         String   @unique
  categoryId  String
  inventory   Int      @default(0)
  images      String[]
  featured    Boolean  @default(false)
  active      Boolean  @default(true)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt

  category   Category @relation(fields: [categoryId], references: [id])
  cartItems  CartItem[]
  orderItems OrderItem[]

  @@map("products")
}

model Category {
  id        String   @id @default(cuid())
  name      String   @unique
  slug      String   @unique
  products  Product[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("categories")
}

model Cart {
  id        String    @id @default(cuid())
  userId    String    @unique
  items     CartItem[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("carts")
}

model CartItem {
  id        String   @id @default(cuid())
  cartId    String
  productId String
  quantity  Int      @default(1)
  createdAt DateTime @default(now())

  cart    Cart    @relation(fields: [cartId], references: [id], onDelete: Cascade)
  product Product @relation(fields: [productId], references: [id])

  @@unique([cartId, productId])
  @@map("cart_items")
}

model Order {
  id         String      @id @default(cuid())
  userId     String
  status     OrderStatus @default(PENDING)
  total      Decimal     @db.Decimal(10, 2)
  stripeId   String?     @unique
  items      OrderItem[]
  shippingAddress Json?
  billingAddress  Json?
  createdAt  DateTime    @default(now())
  updatedAt  DateTime    @updatedAt

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("orders")
}

model OrderItem {
  id        String   @id @default(cuid())
  orderId   String
  productId String
  quantity  Int
  price     Decimal  @db.Decimal(10, 2)
  createdAt DateTime @default(now())

  order   Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product Product @relation(fields: [productId], references: [id])

  @@map("order_items")
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}
// lib/cart.ts
typescript
Copia
Scarica
import { prisma } from '@/lib/prisma'

export class CartService {
  static async getOrCreateCart(userId: string) {
    let cart = await prisma.cart.findUnique({
      where: { userId },
      include: {
        items: {
          include: {
            product: true,
          },
        },
      },
    })

    if (!cart) {
      cart = await prisma.cart.create({
        data: {
          userId,
        },
        include: {
          items: {
            include: {
              product: true,
            },
          },
        },
      })
    }

    return cart
  }

  static async addToCart(userId: string, productId: string, quantity: number = 1) {
    const cart = await this.getOrCreateCart(userId)

    const existingItem = await prisma.cartItem.findUnique({
      where: {
        cartId_productId: {
          cartId: cart.id,
          productId,
        },
      },
    })

    if (existingItem) {
      return await prisma.cartItem.update({
        where: { id: existingItem.id },
        data: { quantity: existingItem.quantity + quantity },
        include: { product: true },
      })
    }

    return await prisma.cartItem.create({
      data: {
        cartId: cart.id,
        productId,
        quantity,
      },
      include: { product: true },
    })
  }

  static async updateCartItem(cartItemId: string, quantity: number) {
    if (quantity <= 0) {
      return await prisma.cartItem.delete({
        where: { id: cartItemId },
      })
    }

    return await prisma.cartItem.update({
      where: { id: cartItemId },
      data: { quantity },
      include: { product: true },
    })
  }

  static async removeFromCart(cartItemId: string) {
    return await prisma.cartItem.delete({
      where: { id: cartItemId },
    })
  }

  static async clearCart(userId: string) {
    const cart = await prisma.cart.findUnique({
      where: { userId },
    })

    if (cart) {
      await prisma.cartItem.deleteMany({
        where: { cartId: cart.id },
      })
    }

    return true
  }

  static async getCartTotal(userId: string): Promise<number> {
    const cart = await this.getOrCreateCart(userId)
    
    const total = cart.items.reduce((sum, item) => {
      return sum + (Number(item.product.price) * item.quantity)
    }, 0)

    return total
  }
}
// app/products/page.tsx
tsx
Copia
Scarica
import { prisma } from '@/lib/prisma'
import { ProductCard } from '@/components/product-card'
import { FilterBar } from '@/components/filter-bar'
import { Pagination } from '@/components/pagination'

interface ProductsPageProps {
  searchParams: {
    category?: string
    page?: string
    sort?: 'price_asc' | 'price_desc' | 'newest'
    minPrice?: string
    maxPrice?: string
  }
}

export default async function ProductsPage({ searchParams }: ProductsPageProps) {
  const page = parseInt(searchParams.page || '1')
  const limit = 12
  const skip = (page - 1) * limit

  const where: any = {
    active: true,
  }

  if (searchParams.category) {
    where.category = {
      slug: searchParams.category,
    }
  }

  if (searchParams.minPrice || searchParams.maxPrice) {
    where.price = {}
    if (searchParams.minPrice) {
      where.price.gte = parseFloat(searchParams.minPrice)
    }
    if (searchParams.maxPrice) {
      where.price.lte = parseFloat(searchParams.maxPrice)
    }
  }

  const orderBy: any = {}
  switch (searchParams.sort) {
    case 'price_asc':
      orderBy.price = 'asc'
      break
    case 'price_desc':
      orderBy.price = 'desc'
      break
    case 'newest':
      orderBy.createdAt = 'desc'
      break
    default:
      orderBy.createdAt = 'desc'
  }

  const [products, total, categories] = await Promise.all([
    prisma.product.findMany({
      where,
      include: {
        category: true,
      },
      orderBy,
      skip,
      take: limit,
    }),
    prisma.product.count({ where }),
    prisma.category.findMany({
      where: {
        products: {
          some: {
            active: true,
          },
        },
      },
    }),
  ])

  return (
    <div className="container py-8">
      <div className="flex flex-col md:flex-row gap-8">
        <aside className="md:w-1/4">
          <FilterBar categories={categories} />
        </aside>

        <div className="md:w-3/4">
          <div className="mb-8">
            <h1 className="text-3xl font-bold">Prodotti</h1>
            <p className="text-muted-foreground mt-2">
              {total} prodotti trovati
            </p>
          </div>

          {products.length === 0 ? (
            <div className="text-center py-12">
              <h3 className="text-lg font-semibold">Nessun prodotto trovato</h3>
              <p className="text-muted-foreground mt-2">
                Prova a modificare i filtri di ricerca
              </p>
            </div>
          ) : (
            <>
              <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
                {products.map((product) => (
                  <ProductCard key={product.id} product={product} />
                ))}
              </div>

              <div className="mt-12">
                <Pagination
                  currentPage={page}
                  totalPages={Math.ceil(total / limit)}
                  totalItems={total}
                  itemsPerPage={limit}
                />
              </div>
            </>
          )}
        </div>
      </div>
    </div>
  )
}
// app/products/[id]/page.tsx
tsx
Copia
Scarica
import { prisma } from '@/lib/prisma'
import { notFound } from 'next/navigation'
import Image from 'next/image'
import { Button } from '@/components/ui/button'
import { AddToCartButton } from '@/components/add-to-cart-button'
import { ShoppingCart, Star, Truck, Shield } from 'lucide-react'

interface ProductPageProps {
  params: {
    id: string

className="my-2" {...props} />
  ),
  blockquote: (props: any) => (
    <blockquote className="border-l-4 border-primary pl-4 italic my-4" {...props} />
  ),
  code: ({ children, className }: any) => {
    const language = className?.replace('language-', '')
    return (
      <code className={`${className} bg-muted px-1 py-0.5 rounded text-sm`}>
        {children}
      </code>
    )
  },
  pre: (props: any) => (
    <pre className="bg-gray-900 text-gray-100 p-4 rounded-lg overflow-x-auto my-4" {...props} />
  ),
  table: (props: any) => (
    <div className="overflow-x-auto my-6">
      <table className="min-w-full divide-y divide-gray-200" {...props} />
    </div>
  ),
}

export async function processMDX(source: string) {
  const { content, frontmatter } = await compileMDX({
    source,
    components,
    options: {
      parseFrontmatter: true,
      mdxOptions: {
        remarkPlugins: [remarkGfm],
        rehypePlugins: [
          rehypeSlug,
          [rehypeAutolinkHeadings, { behavior: 'wrap' }],
          rehypePrism,
        ],
      },
    },
  })

  return { content, frontmatter }
}
// app/blog/page.tsx
tsx
Copia
Scarica
import { prisma } from '@/lib/prisma'
import { PostCard } from '@/components/post-card'
import { Pagination } from '@/components/pagination'
import { FeaturedPost } from '@/components/featured-post'

interface BlogPageProps {
  searchParams: {
    page?: string
    category?: string
    tag?: string
    q?: string
  }
}

export default async function BlogPage({ searchParams }: BlogPageProps) {
  const page = parseInt(searchParams.page || '1')
  const limit = 9
  const skip = (page - 1) * limit

  const where: any = {
    published: true,
    publishedAt: {
      lte: new Date(),
    },
  }

  if (searchParams.category) {
    where.category = {
      slug: searchParams.category,
    }
  }

  if (searchParams.tag) {
    where.tags = {
      has: searchParams.tag,
    }
  }

  if (searchParams.q) {
    where.OR = [
      {
        title: {
          contains: searchParams.q,
          mode: 'insensitive',
        },
      },
      {
        content: {
          contains: searchParams.q,
          mode: 'insensitive',
        },
      },
    ]
  }

  const [posts, total, featuredPost, categories] = await Promise.all([
    prisma.post.findMany({
      where,
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
          },
        },
        category: true,
      },
      orderBy: {
        publishedAt: 'desc',
      },
      skip,
      take: limit,
    }),
    prisma.post.count({ where }),
    prisma.post.findFirst({
      where: {
        published: true,
        publishedAt: {
          lte: new Date(),
        },
      },
      include: {
        author: {
          select: {
            id: true,
            name: true,
            image: true,
          },
        },
        category: true,
      },
      orderBy: {
        views