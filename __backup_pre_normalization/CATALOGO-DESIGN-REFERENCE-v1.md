# ============================================================================
# CATALOGO DESIGN REFERENCE v1.0
# ============================================================================
#
# SCOPO: Fornire reference visivi e layout templates per guidare la 
#        composizione di UI complete a partire dai pattern primitivi.
#
# CONTENUTO:
# 1. Stili di Reference (ispirazioni da brand noti)
# 2. Layout Templates per Tipo Pagina
# 3. Wireframe Testuali Dettagliati
# 4. Composizioni per Categoria Piattaforma
# 5. Sezione Reference Utente (screenshot custom)
#
# UTILIZZO:
# - Ralph legge questo catalogo DOPO i pattern primitivi
# - Identifica lo stile/layout appropriato per il progetto
# - Compone i primitivi seguendo il layout reference
#
# ============================================================================

# ============================================================================
# SEZIONE 1: STILI DI REFERENCE
# ============================================================================

"""
Ogni stile ha caratteristiche distintive che definiscono:
- Palette colori
- Tipografia
- Spacing e proporzioni
- Elementi distintivi
- Mood/feeling
"""

## 1.1 STILE "STRIPE" - Corporate Elegante

```yaml
nome: Stripe Style
mood: Professionale, affidabile, elegante
target: B2B, SaaS, FinTech, Enterprise

caratteristiche:
  colori:
    primary: Viola/Indigo (#635BFF)
    background: Bianco con gradient sottili
    accenti: Gradient multicolore per hero
    testo: Grigio scuro (#425466)
  
  tipografia:
    font: Inter, -apple-system
    titoli: Bold, grandi (48-72px hero)
    body: Regular, 16-18px
    peso: Gerarchia forte (Bold titoli, Regular body)
  
  spacing:
    sezioni: Molto generoso (120-160px)
    elementi: Ariosi (32-48px tra card)
    padding: Abbondante nelle card
  
  elementi_distintivi:
    - Gradient sottili nei background
    - Illustrazioni 3D/isometriche
    - Code snippets con syntax highlighting
    - Animazioni micro-interaction fluide
    - Card con shadow molto soft
    - Border radius medio (8-12px)
  
  layout_tipico:
    hero: Testo sinistra, illustrazione destra
    features: Grid 3 colonne con icone
    pricing: 3 card affiancate
    testimonials: Logo strip + quote singola
    cta_finale: Centrato con gradient background

esempio_pagine:
  - Homepage: Hero gradient + features + pricing + testimonials + CTA
  - Pricing: Header + toggle annual/monthly + 3 piani + FAQ
  - Docs: Sidebar left + content + ToC right
```

---

## 1.2 STILE "LINEAR" - Minimal Dark

```yaml
nome: Linear Style
mood: Moderno, developer-friendly, minimal
target: Developer tools, SaaS tech, Productivity

caratteristiche:
  colori:
    primary: Viola chiaro (#8B5CF6)
    background: Nero/grigio molto scuro (#0A0A0B)
    surface: Grigio scuro (#1A1A1C)
    testo: Bianco/grigio chiaro
    accenti: Gradient viola-blu
  
  tipografia:
    font: Inter, SF Pro
    titoli: Medium weight, tracking tight
    body: Regular, 15-16px
    peso: Sottile ma leggibile
  
  spacing:
    sezioni: Moderato (80-100px)
    elementi: Compatto ma respirabile
    padding: Minimalista
  
  elementi_distintivi:
    - Dark mode nativo
    - Blur effects (glassmorphism sottile)
    - Border sottili luminosi
    - Animazioni smooth e veloci
    - Keyboard shortcuts prominenti
    - Micro-interazioni su hover
    - Border radius piccolo (4-6px)
  
  layout_tipico:
    hero: Centrato, headline potente, subhead breve
    features: Lista verticale con icone
    showcase: Screenshots con glow effect
    changelog: Timeline verticale

esempio_pagine:
  - Homepage: Hero centrato + feature list + screenshots + integrations
  - Dashboard: Sidebar compatta + header + content area scura
  - Changelog: Timeline con date e tag versione
```

---

## 1.3 STILE "NOTION" - Clean Whitespace

```yaml
nome: Notion Style  
mood: Pulito, focalizzato, content-first
target: Productivity, Notes, Documentation, Wiki

caratteristiche:
  colori:
    primary: Nero (#191919)
    background: Bianco puro (#FFFFFF)
    surface: Grigio molto chiaro (#F7F6F3)
    testo: Nero (#37352F)
    accenti: Colori pastello per tag/badge
  
  tipografia:
    font: System fonts, serif opzionale per content
    titoli: Bold, proporzioni editoriali
    body: Regular, 16px, line-height generoso (1.7)
    peso: Contrasto alto tra titoli e body
  
  spacing:
    sezioni: Molto generoso
    elementi: Ampio whitespace
    padding: Abbondante
    margini: Larghi (max-width contenuto ~720px)
  
  elementi_distintivi:
    - Whitespace come elemento design
    - Tipografia forte e leggibile
    - Icone emoji-style
    - Hover states sottili
    - Bordi quasi assenti
    - Focus su contenuto
    - Border radius minimo (3-4px)
  
  layout_tipico:
    hero: Centrato, solo testo, no immagini
    features: Card semplici con icona + testo
    content: Colonna singola centrata
    sidebar: Minimale, collassabile

esempio_pagine:
  - Homepage: Hero testuale + features minimaliste + social proof
  - Editor: Sidebar folders + content area larga
  - Settings: Lista semplice con toggle
```

---

## 1.4 STILE "VERCEL" - Developer Sleek

```yaml
nome: Vercel Style
mood: Tecnico, performante, cutting-edge
target: Developer tools, DevOps, Infrastructure

caratteristiche:
  colori:
    primary: Bianco su nero (#FFFFFF su #000000)
    background: Nero puro (#000000)
    surface: Grigio scurissimo (#111111)
    testo: Bianco (#FFFFFF)
    accenti: Gradient rainbow per effetti speciali
  
  tipografia:
    font: Geist, Inter
    titoli: Bold, molto grandi
    body: Regular, 14-16px
    code: Monospace prominente
  
  spacing:
    sezioni: Generoso (100-140px)
    elementi: Moderato
    padding: Standard
  
  elementi_distintivi:
    - Alto contrasto bianco/nero
    - Terminal/code aesthetics
    - Animazioni di deployment
    - Metriche real-time
    - Gradient rainbow per CTA
    - Glow effects
    - Border radius medio (8px)
  
  layout_tipico:
    hero: Grande statement + deploy button + terminal preview
    features: Grid con metriche
    showcase: Code + preview split
    pricing: Comparison table dettagliata

esempio_pagine:
  - Homepage: Hero + live demo + features + enterprise CTA
  - Dashboard: Deployments list + analytics + logs
  - Docs: Three-column (nav + content + ToC)
```

---

## 1.5 STILE "AIRBNB" - Friendly Visual

```yaml
nome: Airbnb Style
mood: Accogliente, visivo, user-friendly
target: Marketplace, Travel, E-commerce, Consumer

caratteristiche:
  colori:
    primary: Rosa/rosso (#FF385C)
    background: Bianco (#FFFFFF)
    surface: Grigio chiarissimo (#F7F7F7)
    testo: Grigio scuro (#222222)
    accenti: Colori vivaci per categorie
  
  tipografia:
    font: Circular, -apple-system
    titoli: Bold, amichevoli
    body: Regular, 16px
    peso: Caldo e accogliente
  
  spacing:
    sezioni: Generoso
    elementi: Confortevole (24-32px)
    padding: Abbondante
  
  elementi_distintivi:
    - Fotografie grandi e immersive
    - Card con immagini prominenti
    - Search bar prominente
    - Filtri orizzontali scrollabili
    - Avatar e social proof
    - Micro-animazioni deliziose
    - Border radius grande (12-16px)
  
  layout_tipico:
    hero: Search bar centrata + background image
    listing: Grid card con foto
    filters: Barra orizzontale sticky
    detail: Gallery + info split

esempio_pagine:
  - Homepage: Search + categories + featured listings + experiences
  - Search Results: Filters + grid/map split view
  - Listing Detail: Photo gallery + booking card sticky
```

---

## 1.6 STILE "SHOPIFY" - E-commerce Professional

```yaml
nome: Shopify Style
mood: Professionale, commerce-focused, trustworthy
target: E-commerce, Retail, SMB

caratteristiche:
  colori:
    primary: Verde (#008060)
    background: Bianco
    surface: Grigio chiaro (#F6F6F7)
    testo: Grigio scuro (#202223)
    accenti: Verde per azioni positive
  
  tipografia:
    font: -apple-system, Segoe UI
    titoli: Semibold
    body: Regular, 14-16px
    peso: Professionale ma non freddo
  
  spacing:
    sezioni: Standard (60-80px)
    elementi: Compatto ma chiaro
    padding: Funzionale
  
  elementi_distintivi:
    - Dashboard data-driven
    - Card con metriche
    - Grafici e analytics
    - Status badges colorati
    - Empty states illustrati
    - Azioni contestuali
    - Border radius medio (8px)
  
  layout_tipico:
    dashboard: Sidebar + KPIs + recent activity
    products: Table/grid view toggle
    orders: List con status badges
    settings: Grouped sections

esempio_pagine:
  - Admin Dashboard: Sidebar + stats + orders + products quick view
  - Products: Table con bulk actions
  - Storefront: Hero + featured + collections
```

---

## 1.7 STILE "SLACK" - Playful Productive

```yaml
nome: Slack Style
mood: Amichevole, colorato, collaborativo
target: Communication, Team tools, B2B friendly

caratteristiche:
  colori:
    primary: Viola (#4A154B)
    background: Bianco/crema
    surface: Vari colori per canali
    testo: Grigio scuro
    accenti: Palette vivace (giallo, verde, blu, rosso)
  
  tipografia:
    font: Lato, -apple-system
    titoli: Bold, friendly
    body: Regular, 15px
    peso: Caldo e approachable
  
  spacing:
    sezioni: Moderato
    elementi: Compatto per densità info
    padding: Funzionale
  
  elementi_distintivi:
    - Colori per categorizzazione
    - Emoji prominenti
    - Avatar ovunque
    - Notifiche e badges
    - Thread collapsibili
    - Shortcuts tastiera
    - Border radius medio (8px)
  
  layout_tipico:
    app: Three-column (workspaces + channels + chat)
    messages: Thread con reply
    landing: Hero + features + testimonials aziendali

esempio_pagine:
  - App: Sidebar canali + message list + thread panel
  - Landing: Hero + logos + features + pricing + security
```

---

## 1.8 STILE "GITHUB" - Developer Utility

```yaml
nome: GitHub Style
mood: Funzionale, denso, developer-native
target: Developer tools, Open source, Code platforms

caratteristiche:
  colori:
    primary: Verde (#238636) per azioni
    background: Bianco (light) / Grigio scuro (dark)
    surface: Grigio chiaro (#F6F8FA)
    testo: Grigio scuro (#24292F)
    accenti: Blu per link, colori per labels
  
  tipografia:
    font: -apple-system, Segoe UI
    titoli: Semibold
    body: Regular, 14px
    code: Monospace prominente
  
  spacing:
    sezioni: Compatto
    elementi: Dense (16-24px)
    padding: Minimale ma funzionale
  
  elementi_distintivi:
    - Alta densità informativa
    - Tabs per navigazione
    - Contribution graphs
    - Code syntax highlighting
    - Labels/tags colorati
    - Actions contestuali (...)
    - Border radius piccolo (6px)
  
  layout_tipico:
    repo: Header + tabs + content area + sidebar
    profile: Stats + pinned repos + activity
    issues: Filters + list + detail panel

esempio_pagine:
  - Repository: Header + tabs (Code, Issues, PR) + file tree + readme
  - Profile: Avatar + stats + repos grid + contributions
  - Issues: Filter bar + issue list + labels
```


# ============================================================================
# SEZIONE 2: LAYOUT TEMPLATES PER TIPO PAGINA
# ============================================================================

"""
Ogni template definisce:
- Struttura della pagina (sezioni e ordine)
- Wireframe testuale ASCII
- Componenti da usare (reference a UI-PATTERN-PRIMITIVI)
- Varianti per diversi stili
"""

## 2.1 HOMEPAGE / LANDING PAGE

### VARIANTE A: Hero Centered (Notion/Linear style)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                          [Nav] [CTA] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                              [BADGE opzionale]                              │
│                                                                             │
│                     ████████████████████████████████                        │
│                           HEADLINE GRANDE                                   │
│                     ████████████████████████████████                        │
│                                                                             │
│                  Subheadline con value proposition                          │
│                  che spiega il prodotto in 1-2 righe                        │
│                                                                             │
│                    [  CTA Primario  ] [CTA Ghost]                           │
│                                                                             │
│                         [Screenshot/Demo]                                   │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                          SOCIAL PROOF                                       │
│              "Trusted by" + [Logo] [Logo] [Logo] [Logo]                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐                    │
│   │   Feature   │    │   Feature   │    │   Feature   │                    │
│   │     Icon    │    │     Icon    │    │     Icon    │                    │
│   │   Title     │    │   Title     │    │   Title     │                    │
│   │   Desc      │    │   Desc      │    │   Desc      │                    │
│   └─────────────┘    └─────────────┘    └─────────────┘                    │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                        TESTIMONIAL SECTION                                  │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │  "Quote from happy customer about how amazing the product is"       │  │
│   │                                                                     │  │
│   │  [Avatar] Name, Title @ Company                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
├─────────────────────────────────────────────────────────────────────────────┤
│                              CTA FINALE                                     │
│                    Headline di chiusura forte                               │
│                       [  Get Started Free  ]                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ FOOTER                                   Links    Links    Links    Social  │
└─────────────────────────────────────────────────────────────────────────────┘

Componenti UI-PATTERN:
- Header: Navigation > Header
- Hero: Content > Hero Section (centered variant)
- Badge: Content > Badge
- CTA: Action > Button (primary + ghost)
- Logo strip: Custom (flex row)
- Features: Content > Card (icon-top variant) in Grid 3-col
- Testimonial: Content > Card (testimonial variant)
- Footer: Navigation > Footer
```

### VARIANTE B: Hero Split (Stripe style)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                          [Nav] [CTA] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ████████████████████           │                                          │
│   HEADLINE GRANDE                │      ┌─────────────────────┐             │
│   ████████████████████           │      │                     │             │
│                                  │      │    Illustration     │             │
│   Subheadline descrittiva        │      │        or           │             │
│   del prodotto e valore          │      │    Product Demo     │             │
│                                  │      │                     │             │
│   [CTA Primario] [CTA Link]      │      └─────────────────────┘             │
│                                  │                                          │
│   "Used by 10,000+ companies"    │                                          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                           FEATURES GRID                                     │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│   │ [Icon]           │  │ [Icon]           │  │ [Icon]           │         │
│   │ Feature Title    │  │ Feature Title    │  │ Feature Title    │         │
│   │ Description text │  │ Description text │  │ Description text │         │
│   │ goes here        │  │ goes here        │  │ goes here        │         │
│   └──────────────────┘  └──────────────────┘  └──────────────────┘         │
│   ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐         │
│   │ [Icon]           │  │ [Icon]           │  │ [Icon]           │         │
│   │ Feature Title    │  │ Feature Title    │  │ Feature Title    │         │
│   │ Description      │  │ Description      │  │ Description      │         │
│   └──────────────────┘  └──────────────────┘  └──────────────────┘         │
├─────────────────────────────────────────────────────────────────────────────┤
│                          SHOWCASE SECTION                                   │
│   ┌───────────────────────────────────────────────────────────────────┐    │
│   │                                                                   │    │
│   │                    PRODUCT SCREENSHOT                             │    │
│   │                                                                   │    │
│   └───────────────────────────────────────────────────────────────────┘    │
│                   [ Feature highlight tabs ]                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ FOOTER                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Componenti:
- Header: Navigation > Header
- Hero: Layout > Two Column (text left, image right)
- Features: Grid 3x2 di Content > Card
- Showcase: Content > Card (media variant) + Navigation > Tabs
- Footer: Navigation > Footer
```

---

## 2.2 DASHBOARD / ADMIN

### VARIANTE A: Sidebar Fixed (Shopify/Linear style)

```
┌────────────────┬────────────────────────────────────────────────────────────┐
│                │  HEADER BAR                    [Search] [Notif] [Avatar]   │
│   SIDEBAR      ├────────────────────────────────────────────────────────────┤
│                │                                                            │
│   [Logo]       │  Page Title                              [Actions]         │
│                │                                                            │
│   ────────     │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│                │  │   KPI    │ │   KPI    │ │   KPI    │ │   KPI    │      │
│   Dashboard    │  │  $12.5K  │ │  1,234   │ │  +12%    │ │   89%    │      │
│   > Analytics  │  │ Revenue  │ │  Users   │ │  Growth  │ │  NPS     │      │
│   > Reports    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
│                │                                                            │
│   Products     │  ┌────────────────────────────────────────────────────┐   │
│   Orders       │  │                                                    │   │
│   Customers    │  │                   MAIN CHART                       │   │
│                │  │                                                    │   │
│   ────────     │  └────────────────────────────────────────────────────┘   │
│                │                                                            │
│   Settings     │  ┌────────────────────────┐ ┌────────────────────────┐    │
│   Help         │  │    Recent Activity     │ │    Quick Actions       │    │
│                │  │                        │ │                        │    │
│   ────────     │  │  • Item 1              │ │  [Action 1]            │    │
│                │  │  • Item 2              │ │  [Action 2]            │    │
│   [Collapse]   │  │  • Item 3              │ │  [Action 3]            │    │
│                │  └────────────────────────┘ └────────────────────────┘    │
└────────────────┴────────────────────────────────────────────────────────────┘

Componenti:
- Sidebar: Navigation > Sidebar
- Header: Navigation > Header (inline variant)
- KPIs: Data Display > Stat Card (grid 4-col)
- Chart: Data Display > Chart Container
- Activity: Content > List (with-icon variant)
- Quick Actions: Content > Card with Action > Button list
```

### VARIANTE B: Header Top Only (GitHub style)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER    [Logo]    [Tab] [Tab] [Tab] [Tab]          [Search] [+] [Avatar] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Breadcrumb > Path > Current                                                │
│                                                                             │
│  Page Title                                            [Primary Action]     │
│  Description or metadata                                                    │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  [Tab] [Tab] [Tab] [Tab]                              [Filters ▼]   │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │                                                                     │   │
│  │  ┌─────────────────────────────────────────────────────────────┐   │   │
│  │  │  List Item 1                                    [Actions]   │   │   │
│  │  ├─────────────────────────────────────────────────────────────┤   │   │
│  │  │  List Item 2                                    [Actions]   │   │   │
│  │  ├─────────────────────────────────────────────────────────────┤   │   │
│  │  │  List Item 3                                    [Actions]   │   │   │
│  │  ├─────────────────────────────────────────────────────────────┤   │   │
│  │  │  List Item 4                                    [Actions]   │   │   │
│  │  └─────────────────────────────────────────────────────────────┘   │   │
│  │                                                                     │   │
│  │  [Pagination]                                                       │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

Componenti:
- Header: Navigation > Header (tabs variant)
- Breadcrumb: Navigation > Breadcrumb
- Tabs: Navigation > Tabs
- Table: Data Display > Table
- Pagination: Navigation > Pagination
```

---

## 2.3 E-COMMERCE PRODUCT LISTING

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER     [Logo]    [Categories ▼]    [Search............]    [Cart] [User]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  Breadcrumb > Category > Subcategory                                        │
│                                                                             │
│  Category Title                                         [Grid] [List] view  │
│  123 products found                                                         │
│                                                                             │
├──────────────────┬──────────────────────────────────────────────────────────┤
│                  │                                                          │
│  FILTERS         │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│                  │   │ [Img]   │ │ [Img]   │ │ [Img]   │ │ [Img]   │       │
│  ▼ Category      │   │ Title   │ │ Title   │ │ Title   │ │ Title   │       │
│    □ Option 1    │   │ $99     │ │ $149    │ │ $79     │ │ $199    │       │
│    □ Option 2    │   │ ★★★★☆  │ │ ★★★★★  │ │ ★★★☆☆  │ │ ★★★★☆  │       │
│    □ Option 3    │   └─────────┘ └─────────┘ └─────────┘ └─────────┘       │
│                  │                                                          │
│  ▼ Price         │   ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│    [$0] - [$500] │   │ [Img]   │ │ [Img]   │ │ [Img]   │ │ [Img]   │       │
│    [====●====]   │   │ Title   │ │ Title   │ │ Title   │ │ Title   │       │
│                  │   │ $99     │ │ $149    │ │ $79     │ │ $199    │       │
│  ▼ Brand         │   │ ★★★★☆  │ │ ★★★★★  │ │ ★★★☆☆  │ │ ★★★★☆  │       │
│    □ Brand A     │   └─────────┘ └─────────┘ └─────────┘ └─────────┘       │
│    □ Brand B     │                                                          │
│                  │   [1] [2] [3] ... [10] [Next →]                          │
│  ▼ Rating        │                                                          │
│    ★★★★☆ & up   │                                                          │
│                  │                                                          │
│  [Clear Filters] │                                                          │
│                  │                                                          │
└──────────────────┴──────────────────────────────────────────────────────────┘

Componenti:
- Header: Navigation > Header (e-commerce variant)
- Breadcrumb: Navigation > Breadcrumb
- Filters: Category > E-Commerce > Product Filters
- Products: Category > E-Commerce > Product Card (grid)
- Pagination: Navigation > Pagination
```

---

## 2.4 E-COMMERCE PRODUCT DETAIL

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                         [Cart] [User]│
├─────────────────────────────────────────────────────────────────────────────┤
│  Breadcrumb > Category > Product Name                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────┐    PRODUCT TITLE                           │
│  │                             │    ★★★★☆ (123 reviews)                     │
│  │                             │                                            │
│  │      MAIN IMAGE             │    $199.00  ($249 crossed)                 │
│  │                             │    Save $50 (20% off)                      │
│  │                             │                                            │
│  │                             │    Short product description that          │
│  └─────────────────────────────┘    explains the key benefits.              │
│  [thumb] [thumb] [thumb] [thumb]                                            │
│                                     Color: ● ● ● ●                          │
│                                     Size:  [S] [M] [L] [XL]                 │
│                                                                             │
│                                     Quantity: [-] [1] [+]                   │
│                                                                             │
│                                     [████ ADD TO CART ████]                 │
│                                     [♡ Add to Wishlist]                     │
│                                                                             │
│                                     ✓ Free shipping over $50                │
│                                     ✓ 30-day returns                        │
│                                     ✓ 2-year warranty                       │
├─────────────────────────────────────────────────────────────────────────────┤
│  [Description] [Specifications] [Reviews (123)]                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  PRODUCT DESCRIPTION                                                        │
│                                                                             │
│  Long form description with all the details about the product...            │
│  Multiple paragraphs of content explaining features and benefits.           │
├─────────────────────────────────────────────────────────────────────────────┤
│  YOU MAY ALSO LIKE                                                          │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐                           │
│  │ Product │ │ Product │ │ Product │ │ Product │                           │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ FOOTER                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Componenti:
- Gallery: Custom (main + thumbnails)
- Product Info: Custom composition
- Variants: Input > Radio Group (button variant)
- Quantity: Input > Number Input
- Add to Cart: Action > Button (primary, large)
- Tabs: Navigation > Tabs
- Related: Category > E-Commerce > Product Card (carousel)
```

---

## 2.5 PRICING PAGE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                          [Nav] [CTA] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                        Simple, transparent pricing                          │
│                 Choose the plan that's right for you                        │
│                                                                             │
│                   [ Monthly ]  [ Yearly - Save 20% ]                        │
│                                                                             │
│   ┌─────────────────┐ ┌─────────────────────┐ ┌─────────────────┐          │
│   │                 │ │ ★ MOST POPULAR      │ │                 │          │
│   │     STARTER     │ │                     │ │   ENTERPRISE    │          │
│   │                 │ │        PRO          │ │                 │          │
│   │     $9/mo       │ │                     │ │    Contact us   │          │
│   │                 │ │      $29/mo         │ │                 │          │
│   │  ✓ Feature 1    │ │                     │ │  Everything in  │          │
│   │  ✓ Feature 2    │ │  ✓ All Starter      │ │  Pro, plus:     │          │
│   │  ✓ Feature 3    │ │  ✓ Feature 4        │ │                 │          │
│   │  ✗ Feature 4    │ │  ✓ Feature 5        │ │  ✓ Custom       │          │
│   │  ✗ Feature 5    │ │  ✓ Feature 6        │ │  ✓ SLA          │          │
│   │                 │ │  ✓ Priority Support │ │  ✓ Dedicated    │          │
│   │                 │ │                     │ │                 │          │
│   │ [Get Started]   │ │ [███ Start Trial ██]│ │ [Contact Sales] │          │
│   │                 │ │                     │ │                 │          │
│   └─────────────────┘ └─────────────────────┘ └─────────────────┘          │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                          FEATURE COMPARISON                                 │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ Feature              │ Starter │ Pro     │ Enterprise │             │   │
│  ├─────────────────────────────────────────────────────────────────────┤   │
│  │ Users                │ 1       │ 10      │ Unlimited  │             │   │
│  │ Storage              │ 5GB     │ 50GB    │ Unlimited  │             │   │
│  │ API Access           │ ✗       │ ✓       │ ✓          │             │   │
│  │ Priority Support     │ ✗       │ ✓       │ ✓          │             │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────────┤
│                              FAQ                                            │
│  ▼ Can I change plans later?                                               │
│  ▼ What payment methods do you accept?                                     │
│  ▼ Is there a free trial?                                                  │
│  ▼ Can I cancel anytime?                                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│ FOOTER                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘

Componenti:
- Toggle: Action > Segmented Control
- Pricing Cards: Category > SaaS > Pricing Table
- Comparison: Data Display > Table
- FAQ: Overlay > Accordion
```

---

## 2.6 SETTINGS PAGE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                                      │
├────────────────┬────────────────────────────────────────────────────────────┤
│                │                                                            │
│   SETTINGS     │  Profile Settings                                          │
│   NAV          │                                                            │
│                │  ┌────────────────────────────────────────────────────┐   │
│   ● Profile    │  │  Profile Picture                                   │   │
│   ○ Account    │  │  ┌──────┐                                          │   │
│   ○ Security   │  │  │ IMG  │  [Upload new]  [Remove]                  │   │
│   ○ Billing    │  │  └──────┘                                          │   │
│   ○ Team       │  └────────────────────────────────────────────────────┘   │
│   ○ Notifiche  │                                                            │
│   ○ API        │  ┌────────────────────────────────────────────────────┐   │
│   ○ Integrazioni│ │  Personal Information                              │   │
│                │  │                                                    │   │
│                │  │  First Name          Last Name                     │   │
│                │  │  [John            ]  [Doe              ]           │   │
│                │  │                                                    │   │
│                │  │  Email                                             │   │
│                │  │  [john@example.com                     ]           │   │
│                │  │                                                    │   │
│                │  │  Bio                                               │   │
│                │  │  [                                     ]           │   │
│                │  │  [                                     ]           │   │
│                │  │                                                    │   │
│                │  └────────────────────────────────────────────────────┘   │
│                │                                                            │
│                │  ┌────────────────────────────────────────────────────┐   │
│                │  │  Preferences                                       │   │
│                │  │                                                    │   │
│                │  │  Language           [English        ▼]             │   │
│                │  │  Timezone           [UTC+1          ▼]             │   │
│                │  │  Dark Mode          [━━━━●]                        │   │
│                │  │                                                    │   │
│                │  └────────────────────────────────────────────────────┘   │
│                │                                                            │
│                │                              [Cancel]  [Save Changes]      │
│                │                                                            │
└────────────────┴────────────────────────────────────────────────────────────┘

Componenti:
- Settings Nav: Navigation > Sidebar (simple variant)
- Sections: Content > Card (grouped)
- Form Fields: Input > Text Input, Select, Toggle
- File Upload: Input > File Upload (avatar variant)
- Actions: Action > Button (primary + ghost)
```


---

## 2.7 AUTH PAGES (Login/Signup)

### VARIANTE A: Centered Minimal

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│                                                                             │
│                                                                             │
│                              [LOGO]                                         │
│                                                                             │
│                         Welcome back                                        │
│                   Sign in to your account                                   │
│                                                                             │
│               ┌─────────────────────────────────────┐                       │
│               │                                     │                       │
│               │  Email                              │                       │
│               │  [                               ]  │                       │
│               │                                     │                       │
│               │  Password                           │                       │
│               │  [                               ]  │                       │
│               │                                     │                       │
│               │  □ Remember me    Forgot password?  │                       │
│               │                                     │                       │
│               │  [████████ Sign In ████████████]    │                       │
│               │                                     │                       │
│               │  ─────────── or ───────────         │                       │
│               │                                     │                       │
│               │  [ G  Continue with Google    ]     │                       │
│               │  [    Continue with GitHub    ]     │                       │
│               │                                     │                       │
│               │  Don't have an account? Sign up     │                       │
│               │                                     │                       │
│               └─────────────────────────────────────┘                       │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### VARIANTE B: Split Screen (immagine laterale)

```
┌────────────────────────────────────┬────────────────────────────────────────┐
│                                    │                                        │
│                                    │                                        │
│                                    │              [LOGO]                    │
│        BRAND IMAGE                 │                                        │
│        or                          │         Create an account              │
│        ILLUSTRATION                │                                        │
│                                    │    Full Name                           │
│                                    │    [                              ]    │
│        ─────────────               │                                        │
│                                    │    Email                               │
│        "Testimonial quote          │    [                              ]    │
│         from a customer"           │                                        │
│                                    │    Password                            │
│        — Name, Company             │    [                              ]    │
│                                    │                                        │
│                                    │    □ I agree to Terms & Privacy        │
│                                    │                                        │
│                                    │    [█████ Create Account █████]        │
│                                    │                                        │
│                                    │    Already have account? Sign in       │
│                                    │                                        │
└────────────────────────────────────┴────────────────────────────────────────┘
```

---

## 2.8 CHECKOUT PAGE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER    [Logo]                        Secure Checkout 🔒    [Back to Cart]│
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│    [ 1. Shipping ●━━━━━━ 2. Payment ○━━━━━━ 3. Review ○ ]                   │
│                                                                             │
├────────────────────────────────────────────────┬────────────────────────────┤
│                                                │                            │
│  Contact Information                           │   ORDER SUMMARY            │
│  ┌──────────────────────────────────────────┐  │                            │
│  │ Email                                    │  │   ┌────┐ Product 1   $99  │
│  │ [                                     ]  │  │   └────┘ Size: M          │
│  └──────────────────────────────────────────┘  │                            │
│                                                │   ┌────┐ Product 2   $49  │
│  Shipping Address                              │   └────┘ Qty: 2           │
│  ┌──────────────────────────────────────────┐  │                            │
│  │ First Name         Last Name             │  │   ─────────────────────   │
│  │ [              ]   [                 ]   │  │   Subtotal        $197    │
│  │                                          │  │   Shipping         $10    │
│  │ Address                                  │  │   Tax              $20    │
│  │ [                                     ]  │  │                            │
│  │                                          │  │   ━━━━━━━━━━━━━━━━━━━━━   │
│  │ City               State      ZIP        │  │   TOTAL           $227    │
│  │ [            ]  [       ]  [       ]     │  │                            │
│  │                                          │  │   ┌────────────────────┐  │
│  │ Phone                                    │  │   │ Have a promo code? │  │
│  │ [                                     ]  │  │   │ [          ] Apply │  │
│  └──────────────────────────────────────────┘  │   └────────────────────┘  │
│                                                │                            │
│  Shipping Method                               │   🔒 Secure checkout       │
│  ┌──────────────────────────────────────────┐  │   ✓ SSL encrypted         │
│  │ ○ Standard (5-7 days)          FREE      │  │   ✓ PCI compliant         │
│  │ ○ Express (2-3 days)           $9.99     │  │                            │
│  │ ○ Overnight                    $19.99    │  │   [Visa][MC][Amex][PP]    │
│  └──────────────────────────────────────────┘  │                            │
│                                                │                            │
│  [← Back]                   [Continue to Payment →]                         │
│                                                │                            │
└────────────────────────────────────────────────┴────────────────────────────┘
```

---

## 2.9 BLOG / ARTICLE PAGE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                          [Nav] [CTA] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                             [Category Badge]                                │
│                                                                             │
│                    ████████████████████████████████                         │
│                         ARTICLE TITLE HERE                                  │
│                    ████████████████████████████████                         │
│                                                                             │
│                    Subtitle or excerpt that gives                           │
│                    more context about the article                           │
│                                                                             │
│                    [Avatar] John Doe · Jan 26, 2026 · 5 min read           │
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────────┐  │
│   │                                                                     │  │
│   │                        HERO IMAGE                                   │  │
│   │                                                                     │  │
│   └─────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ [Share] [Twitter] [LinkedIn] [Copy]                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│     ┌─────────────────────────────────────────────────────────────┐        │
│     │                                                             │        │
│     │  Article content goes here. This is the main body of the   │        │
│     │  article with proper typography and comfortable line       │        │
│     │  length for reading (around 65-75 characters per line).    │        │
│     │                                                             │        │
│     │  ## Subheading                                              │        │
│     │                                                             │        │
│     │  More content with proper spacing between paragraphs.      │        │
│     │  Images, code blocks, quotes can be embedded here.         │        │
│     │                                                             │        │
│     │  > This is a blockquote that stands out from the           │        │
│     │  > regular text content.                                   │        │
│     │                                                             │        │
│     │  ```                                                        │        │
│     │  // Code example                                            │        │
│     │  const hello = "world";                                     │        │
│     │  ```                                                        │        │
│     │                                                             │        │
│     └─────────────────────────────────────────────────────────────┘        │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  ABOUT THE AUTHOR                                                           │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  [Avatar]  John Doe                                                 │   │
│  │            Bio description here. Writer, developer, etc.            │   │
│  │            [Twitter] [LinkedIn]                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────────────────┤
│  RELATED ARTICLES                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                      │
│  │   [Image]    │  │   [Image]    │  │   [Image]    │                      │
│  │   Title      │  │   Title      │  │   Title      │                      │
│  │   Date       │  │   Date       │  │   Date       │                      │
│  └──────────────┘  └──────────────┘  └──────────────┘                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ FOOTER                                                                      │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2.10 CHAT / MESSAGING INTERFACE

```
┌────────────────┬────────────────────────────────────────────────────────────┐
│                │  HEADER   [Channel Name]          [Search] [Info] [Call]   │
│   CHANNELS     ├────────────────────────────────────────────────────────────┤
│                │                                                            │
│   ▼ Workspace  │    ┌─ Today ─────────────────────────────────────────┐    │
│                │    │                                                 │    │
│   # general    │    │  [Ava]  Hey team, how's the project going?      │    │
│   # random     │    │         10:30 AM                                │    │
│   # support    │    │                                                 │    │
│                │    │  [Bob]  Making good progress! Just finished     │    │
│   ▼ Direct     │    │         the new feature.                        │    │
│                │    │         10:35 AM                                │    │
│   ● John       │    │                                                 │    │
│   ○ Sarah      │    │         ┌────────────────────────┐              │    │
│   ○ Mike       │    │         │   screenshot.png       │              │    │
│                │    │         │   [Image preview]      │              │    │
│   ▼ Apps       │    │         └────────────────────────┘              │    │
│                │    │                                                 │    │
│   🤖 Bot       │    │  [You]  Awesome! Let me review it.              │    │
│   📊 Analytics │    │         10:40 AM  ✓✓                            │    │
│                │    │                                                 │    │
│                │    └─────────────────────────────────────────────────┘    │
│                │                                                            │
│                │                                                            │
│                ├────────────────────────────────────────────────────────────┤
│                │                                                            │
│  [+ Channel]   │  [+] [📎] [Type a message...                    ] [😀] [➤]│
│                │                                                            │
└────────────────┴────────────────────────────────────────────────────────────┘

Componenti:
- Channels sidebar: Navigation > Sidebar (messaging variant)
- Messages: Category > Social > Chat Message
- Input: Category > Social > Chat Input
- Attachment: Input > File Upload (inline)
```

---

## 2.11 PROFILE PAGE (Social)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER                                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌────────────────────────────────────────────────────────────────────────┐│
│  │                      COVER IMAGE                                       ││
│  │                                                                        ││
│  └────────────────────────────────────────────────────────────────────────┘│
│                                                                             │
│       ┌──────────┐                                                          │
│       │          │   John Doe                    [Edit Profile] [Share]     │
│       │  AVATAR  │   @johndoe · Designer                                    │
│       │          │                                                          │
│       └──────────┘   Bio text here describing the user and their           │
│                      interests. Links and hashtags can be highlighted.      │
│                                                                             │
│                      📍 San Francisco  🔗 johndoe.com  📅 Joined Jan 2024   │
│                                                                             │
│                      127 Following    1.2K Followers    45 Posts            │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│  [Posts]  [Replies]  [Media]  [Likes]                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ [Avatar] John Doe @johndoe · 2h                              [···]  │   │
│  │                                                                     │   │
│  │ Post content goes here. Can include text, images, links.           │   │
│  │                                                                     │   │
│  │ ┌─────────────────────────────────────────────────────────────┐    │   │
│  │ │                      [Image]                                 │    │   │
│  │ └─────────────────────────────────────────────────────────────┘    │   │
│  │                                                                     │   │
│  │  💬 12    🔄 34    ❤️ 156    📤                                    │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │ [Another post...]                                                   │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 2.12 404 / ERROR PAGE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ HEADER (minimal)                                                [Logo only] │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│                                                                             │
│                                                                             │
│                           ┌─────────────┐                                   │
│                           │             │                                   │
│                           │   [404]     │                                   │
│                           │   or        │                                   │
│                           │ Illustration│                                   │
│                           │             │                                   │
│                           └─────────────┘                                   │
│                                                                             │
│                          Page not found                                     │
│                                                                             │
│               Sorry, we couldn't find the page you're                       │
│               looking for. It might have been moved or deleted.             │
│                                                                             │
│                                                                             │
│                    [← Go back]    [Go to Homepage]                          │
│                                                                             │
│                                                                             │
│                         ─────────────────                                   │
│                                                                             │
│                    Or try these popular pages:                              │
│                                                                             │
│                    • Products                                               │
│                    • Pricing                                                │
│                    • Support                                                │
│                    • Blog                                                   │
│                                                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```


# ============================================================================
# SEZIONE 3: COMPOSIZIONI COMPLETE PER CATEGORIA
# ============================================================================

"""
Ogni categoria ha un set di pagine tipiche con layout consigliato.
Ralph usa questa sezione per capire quali pagine creare e come strutturarle.
"""

## 3.1 E-COMMERCE - Pagine Tipiche

```yaml
categoria: E-Commerce
stile_consigliato: Airbnb, Shopify
pagine_core:
  
  homepage:
    layout: "Hero Split o Centered"
    sezioni:
      - Hero con search o CTA
      - Categorie in evidenza (card grid 4-6)
      - Prodotti popolari (carousel)
      - Banner promo (full width)
      - Nuovi arrivi (grid)
      - Testimonials/Reviews
      - Newsletter signup
      - Footer con trust badges
  
  category_listing:
    layout: "2.3 E-Commerce Product Listing"
    sezioni:
      - Breadcrumb
      - Filtri sidebar (collapsible mobile)
      - Products grid (responsive 2-4 col)
      - Sorting dropdown
      - Pagination
  
  product_detail:
    layout: "2.4 E-Commerce Product Detail"
    sezioni:
      - Breadcrumb
      - Image gallery
      - Product info + variants
      - Add to cart
      - Description tabs
      - Related products
  
  cart:
    layout: "Two column (cart + summary)"
    sezioni:
      - Cart items list (editable quantity)
      - Order summary sidebar
      - Promo code input
      - Continue shopping / Checkout CTAs
  
  checkout:
    layout: "2.8 Checkout Page"
    sezioni:
      - Progress stepper (3-4 steps)
      - Shipping form
      - Payment form (Stripe Elements)
      - Order review
      - Order summary sticky
  
  order_confirmation:
    layout: "Centered success"
    sezioni:
      - Success icon + message
      - Order number
      - Order summary
      - Next steps
      - Continue shopping CTA
  
  account:
    layout: "2.6 Settings Page"
    sezioni:
      - Profile
      - Addresses
      - Orders history
      - Wishlist
      - Payment methods
```

---

## 3.2 SAAS - Pagine Tipiche

```yaml
categoria: SaaS
stile_consigliato: Stripe, Linear
pagine_core:
  
  landing:
    layout: "2.1 Homepage Variante B (Split)"
    sezioni:
      - Hero con demo/screenshot
      - Social proof (logo strip)
      - Features grid (3x2)
      - Product showcase (tabs + screenshots)
      - Testimonials
      - Pricing preview
      - CTA finale
  
  pricing:
    layout: "2.5 Pricing Page"
    sezioni:
      - Monthly/Yearly toggle
      - 3 pricing cards
      - Feature comparison table
      - FAQ accordion
      - Enterprise CTA
  
  features:
    layout: "Alternating sections"
    sezioni:
      - Feature 1 (text left, image right)
      - Feature 2 (image left, text right)
      - Feature 3 (text left, image right)
      - All features list
      - CTA
  
  dashboard:
    layout: "2.2 Dashboard Variante A"
    sezioni:
      - KPI cards row
      - Main chart
      - Activity feed
      - Quick actions
  
  settings:
    layout: "2.6 Settings Page"
    sezioni:
      - Profile
      - Team members
      - Billing
      - API keys
      - Integrations
      - Notifications
  
  onboarding:
    layout: "Centered wizard"
    sezioni:
      - Progress indicator
      - Step content
      - Skip/Back/Next buttons
      - Completion celebration
```

---

## 3.3 SOCIAL/COMMUNITY - Pagine Tipiche

```yaml
categoria: Social
stile_consigliato: Slack (app), Notion (content)
pagine_core:
  
  feed:
    layout: "Three column (sidebar + feed + suggestions)"
    sezioni:
      - Post composer
      - Feed with infinite scroll
      - Trending sidebar
      - Suggested follows
  
  profile:
    layout: "2.11 Profile Page"
    sezioni:
      - Cover image
      - Avatar + info
      - Stats (followers, etc)
      - Tabs (posts, media, likes)
      - Content grid
  
  messaging:
    layout: "2.10 Chat Interface"
    sezioni:
      - Conversations list
      - Chat area
      - Message input
      - User info panel (collapsible)
  
  notifications:
    layout: "Single column list"
    sezioni:
      - Filter tabs (all, mentions, etc)
      - Notification items
      - Mark all read
  
  settings:
    layout: "2.6 Settings Page"
    sezioni:
      - Account
      - Privacy
      - Notifications
      - Blocked users
      - Data export
```

---

## 3.4 DASHBOARD/ANALYTICS - Pagine Tipiche

```yaml
categoria: Dashboard
stile_consigliato: Linear, Vercel
pagine_core:
  
  overview:
    layout: "2.2 Dashboard Variante A"
    sezioni:
      - Date range picker
      - KPI cards (4-6)
      - Main charts (1-2)
      - Secondary metrics
      - Activity/Events
  
  analytics:
    layout: "Full width with filters"
    sezioni:
      - Filter bar (date, segments)
      - Comparison toggle
      - Large chart area
      - Breakdown tables
      - Export options
  
  reports:
    layout: "List with preview"
    sezioni:
      - Report templates
      - Saved reports
      - Report builder
      - Schedule options
  
  data_explorer:
    layout: "Split (query + results)"
    sezioni:
      - Query builder / SQL
      - Results table
      - Visualization options
      - Save/Export
```

---

## 3.5 HEALTHCARE - Pagine Tipiche

```yaml
categoria: Healthcare
stile_consigliato: Notion (clean), Shopify (dashboard)
compliance: HIPAA required
pagine_core:
  
  patient_portal_home:
    layout: "Dashboard con cards"
    sezioni:
      - Welcome + alerts
      - Upcoming appointments
      - Recent vitals
      - Medications
      - Messages (unread count)
  
  appointments:
    layout: "Calendar + list"
    sezioni:
      - Calendar view
      - Upcoming list
      - Past appointments
      - Book new (modal/page)
  
  medical_records:
    layout: "List with filters"
    sezioni:
      - Record types filter
      - Date range
      - Records list
      - Record detail (modal/page)
      - Download/Print
  
  messaging:
    layout: "2.10 Chat (secure variant)"
    sezioni:
      - Encryption indicator
      - Provider selection
      - Message thread
      - Attachment handling
  
  vitals:
    layout: "Charts + data entry"
    sezioni:
      - Vital charts (trend)
      - Add new reading
      - History table
      - Alerts/thresholds
```

---

## 3.6 FINTECH - Pagine Tipiche

```yaml
categoria: FinTech
stile_consigliato: Stripe, Vercel (dark)
compliance: PCI-DSS, KYC/AML
pagine_core:
  
  dashboard:
    layout: "2.2 Dashboard"
    sezioni:
      - Account balances (masked)
      - Quick actions
      - Recent transactions
      - Spending chart
      - Alerts
  
  accounts:
    layout: "Cards + detail"
    sezioni:
      - Account cards
      - Account details
      - Statements
      - Settings
  
  transactions:
    layout: "2.3 Listing with filters"
    sezioni:
      - Search + filters
      - Transaction list
      - Transaction detail (slide-over)
      - Export
  
  transfer:
    layout: "Wizard/Stepper"
    sezioni:
      - Select source
      - Enter recipient
      - Enter amount
      - Review + confirm
      - 2FA verification
      - Confirmation
  
  cards:
    layout: "Card management"
    sezioni:
      - Card visualization
      - Card controls (freeze, limits)
      - PIN management
      - Request new card
  
  kyc:
    layout: "Verification wizard"
    sezioni:
      - Document selection
      - Upload (front/back)
      - Selfie capture
      - Processing status
      - Approval/Rejection
```

# ============================================================================
# SEZIONE 4: REFERENCE UTENTE (Screenshot Custom)
# ============================================================================

"""
Questa sezione è riservata per aggiungere reference visivi forniti dall'utente.

COME USARE:
1. L'utente fornisce screenshot di design che gli piacciono
2. Ralph analizza gli screenshot e documenta:
   - Struttura layout
   - Palette colori
   - Tipografia
   - Pattern distintivi
   - Componenti usati
3. Le note vengono aggiunte qui per reference futuro
4. Ralph usa queste note per replicare lo stile
"""

## 4.1 TEMPLATE PER AGGIUNGERE REFERENCE

```yaml
# REFERENCE: [Nome Descrittivo]
# Fonte: [URL o descrizione]
# Data aggiunta: [Data]

analisi:
  
  layout:
    struttura: "[Descrizione struttura pagina]"
    colonne: "[N colonne, proporzioni]"
    spacing: "[Generoso/Moderato/Compatto]"
    max_width: "[px o % della pagina]"
  
  colori:
    primary: "[colore e hex]"
    background: "[colore e hex]"
    text: "[colore e hex]"
    accent: "[colore e hex]"
    note: "[note su uso colori]"
  
  tipografia:
    font_heading: "[font name]"
    font_body: "[font name]"
    sizes: "[range di dimensioni usate]"
    weights: "[pesi usati]"
    line_height: "[stretto/normale/generoso]"
  
  elementi_distintivi:
    - "[elemento 1]"
    - "[elemento 2]"
    - "[elemento 3]"
  
  componenti_identificati:
    - "[componente 1] → [pattern catalogo corrispondente]"
    - "[componente 2] → [pattern catalogo corrispondente]"
  
  note_implementazione:
    - "[nota 1]"
    - "[nota 2]"
```

---

## 4.2 REFERENCE AGGIUNTI

### (Sezione vuota - da popolare con reference utente)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   Nessun reference utente aggiunto.                                         │
│                                                                             │
│   Per aggiungere un reference:                                              │
│   1. Invia uno screenshot a Claude                                          │
│   2. Chiedi di analizzarlo e documentarlo                                   │
│   3. Il reference verrà aggiunto a questa sezione                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

# ============================================================================
# SEZIONE 5: GUIDA RAPIDA SELEZIONE STILE
# ============================================================================

## 5.1 DECISION TREE - Quale Stile Scegliere?

```
START: Che tipo di prodotto stai costruendo?
│
├─► B2B / Enterprise / SaaS
│   └─► Vuoi un look premium?
│       ├─► Sì → STRIPE STYLE
│       └─► No, più tecnico → VERCEL STYLE
│
├─► Developer Tool / API
│   └─► Target audience?
│       ├─► Developers → GITHUB STYLE o VERCEL STYLE
│       └─► Mixed → LINEAR STYLE
│
├─► Consumer Product / E-commerce
│   └─► Che feeling vuoi?
│       ├─► Friendly/Visual → AIRBNB STYLE
│       ├─► Professional → SHOPIFY STYLE
│       └─► Minimal → NOTION STYLE
│
├─► Productivity / Notes / Docs
│   └─► NOTION STYLE
│
├─► Communication / Team
│   └─► SLACK STYLE
│
├─► Dark Mode / Modern Tech
│   └─► LINEAR STYLE o VERCEL STYLE
│
└─► Non so / Generico
    └─► Parti con STRIPE STYLE (versatile e professionale)
```

## 5.2 QUICK MATCH TABLE

| Se il tuo prodotto è... | Usa questo stile | Perché |
|-------------------------|------------------|--------|
| SaaS B2B | Stripe | Professionale, affidabile |
| Developer tool | Linear/Vercel | Tecnico, moderno |
| E-commerce generico | Shopify | Commerce-focused |
| Marketplace visual | Airbnb | Imagery-driven |
| Productivity app | Notion | Content-first |
| Team communication | Slack | Friendly, colorful |
| Open source | GitHub | Dense, functional |
| Healthcare | Notion + custom | Clean, accessible |
| FinTech | Stripe/Vercel | Trust, professional |
| Content/Blog | Notion | Typography-focused |

## 5.3 COMBINAZIONI CONSIGLIATE

```yaml
ecommerce_mvp:
  stile_base: Shopify
  hero: Airbnb (visual)
  checkout: Stripe (trust)
  
saas_startup:
  stile_base: Stripe
  dashboard: Linear (modern)
  docs: Notion (clean)
  
developer_tool:
  stile_base: Linear
  api_docs: GitHub (dense)
  marketing: Vercel (impactful)
  
healthcare_portal:
  stile_base: Notion (accessibility)
  dashboard: Shopify (functional)
  forms: Stripe (clear)
  
fintech_app:
  stile_base: Stripe (trust)
  charts: Linear (modern)
  security: Vercel (serious)
```

# ============================================================================
# FINE CATALOGO DESIGN REFERENCE
# ============================================================================

"""
Questo catalogo fornisce i reference visivi necessari per comporre
UI complete a partire dai pattern primitivi.

Usato insieme a:
- CATALOGO-DESIGN-TOKEN-SYSTEM → per i valori esatti dei token
- CATALOGO-UI-PATTERN-PRIMITIVI → per i componenti da usare

Versione: 1.0.0
Ultima modifica: 2026-01-26
"""
