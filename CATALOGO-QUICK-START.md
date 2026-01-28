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
