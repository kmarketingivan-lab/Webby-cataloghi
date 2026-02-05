# ═══════════════════════════════════════════════════════════════════════════════
# STATO PROGETTO: TOOL GENERAZIONE PIATTAFORME AUTONOME
# Data: 2026-01-29
# ═══════════════════════════════════════════════════════════════════════════════

## 🎯 OBIETTIVO FINALE

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                                                                             │
│   UTENTE → [PROMPT] → TOOL → [PIATTAFORMA COMPLETA PRODUCTION-READY]       │
│                                                                             │
│   Esempio:                                                                  │
│   "Voglio un e-commerce per vendere scarpe con auth, pagamenti Stripe,     │
│    dashboard admin, reviews prodotti"                                       │
│                                                                             │
│   Output:                                                                   │
│   - Codebase Next.js 14 completo                                           │
│   - Database Prisma configurato                                            │
│   - Auth funzionante                                                       │
│   - Pagamenti Stripe integrati                                             │
│   - Dashboard admin                                                        │
│   - Deploy su Vercel pronto                                                │
│   - ZERO TODO, ZERO placeholder                                            │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📊 COSA ABBIAMO (LA KNOWLEDGE BASE - IL "CERVELLO")

### Statistiche
| Metrica | Valore | Status |
|---------|--------|--------|
| Cataloghi | 68 | ✅ |
| Righe totali | 235,445 | ✅ |
| Blocchi codice | 2,071 | ✅ |
| Dimensione | 8.03 MB | ✅ |
| **VOTO QUALITÀ** | **93/100** | ✅ |

### Copertura Aree
| Area | Cataloghi | Status |
|------|-----------|--------|
| Orchestrazione | Master Orchestrator, Decision Tree | ✅ |
| Boilerplate | Config, Struttura | ✅ |
| Autenticazione | Auth completo (Auth.js v5) | ✅ |
| Database | Prisma, Data Model, Selection | ✅ |
| API | tRPC, REST Design | ✅ |
| UI/UX | Shadcn, Tailwind, Patterns, Tokens | ✅ |
| E-commerce | Products, Cart, Orders, Payments, Admin | ✅ |
| Social | Profiles, Posts, Comments, Notifications | ✅ |
| Blog | Rich Text Editor, Blog completo | ✅ |
| DevOps | CI/CD, Docker, Deploy | ✅ |
| Testing | Vitest, Testing Library, E2E | ✅ |
| Security | Sicurezza, Legal Compliance | ✅ |
| Performance | SEO, Analytics, Monitoring | ✅ |
| Features | Search, i18n, Forms, Files, Real-time | ✅ |

### File Chiave Creati
```
cataloghi per tool/
├── CATALOGO-MASTER-ORCHESTRATOR-v1.md    # Come orchestrare la generazione
├── CATALOGO-DECISION-TREE-AI-v1.md       # Logica decisionale
├── CATALOGO-BOILERPLATE-*.md             # Template base progetto
├── CATALOGO-AUTHENTICATION-v1.md         # Auth completo
├── CATALOGO-ECOMMERCE-*.md               # 4 cataloghi e-commerce
├── CATALOGO-SOCIAL-*.md                  # 2 cataloghi social
├── CATALOGO-TESTING-VITEST-v1.md         # Testing completo
├── CATALOGO-DEVOPS-CI-CD-v1.md           # Deploy pipeline
└── ... (68 cataloghi totali)
```

---

## ❌ COSA MANCA (IL TOOL - IL "CORPO")

### Architettura Target
```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PLATFORM GENERATOR TOOL                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │   1. INPUT   │───▶│  2. PARSER   │───▶│  3. PLANNER  │                  │
│  │              │    │              │    │              │                  │
│  │ User Prompt  │    │ NLP/Intent   │    │ Decision     │                  │
│  │ + Config     │    │ Recognition  │    │ Tree Logic   │                  │
│  └──────────────┘    └──────────────┘    └──────────────┘                  │
│                                                 │                           │
│                                                 ▼                           │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐                  │
│  │ 6. DEPLOYER  │◀───│ 5. VALIDATOR │◀───│ 4. GENERATOR │                  │
│  │              │    │              │    │              │                  │
│  │ Vercel/AWS   │    │ TypeCheck    │    │ Code Gen     │                  │
│  │ Auto Deploy  │    │ Lint/Test    │    │ via AI APIs  │                  │
│  └──────────────┘    └──────────────┘    └──────────────┘                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Componenti Mancanti

| # | Componente | Descrizione | Priorità | Effort |
|---|------------|-------------|----------|--------|
| 1 | **PROMPT PARSER** | Analizza il prompt utente ed estrae: tipo piattaforma, features, requisiti | CRITICO | 8h |
| 2 | **DECISION ENGINE** | Usa il Decision Tree per selezionare i cataloghi/moduli necessari | CRITICO | 12h |
| 3 | **CODE GENERATOR** | Orchestra le chiamate AI (Claude/Gemini) per generare codice | CRITICO | 20h |
| 4 | **FILE ASSEMBLER** | Assembla i file generati nella struttura progetto corretta | CRITICO | 8h |
| 5 | **VALIDATOR** | Esegue TypeScript check, Prisma validate, ESLint | ALTO | 6h |
| 6 | **TEST RUNNER** | Esegue i test generati automaticamente | ALTO | 4h |
| 7 | **BUILDER** | Compila il progetto (next build) | MEDIO | 2h |
| 8 | **DEPLOYER** | Deploy automatico su Vercel/AWS | MEDIO | 6h |
| 9 | **CLI/UI** | Interfaccia per lanciare il tool | ALTO | 8h |
| 10 | **RALPH INTEGRATION** | Integrazione con Claude Code CLI | ALTO | 10h |

**TOTALE EFFORT STIMATO: ~85 ore**

---

## 🔧 PIANO D'AZIONE PER COMPLETARE

### FASE 1: CORE ENGINE (40h) - PRIORITÀ MASSIMA
```
1.1 Prompt Parser (8h)
    - Input: stringa prompt utente
    - Output: JSON strutturato {tipo, features[], requisiti{}}
    - Usa: Claude API per NLP
    
1.2 Decision Engine (12h)
    - Input: JSON dal parser
    - Output: lista cataloghi da usare + ordine generazione
    - Usa: CATALOGO-DECISION-TREE-AI-v1.md
    
1.3 Code Generator (20h)
    - Input: lista cataloghi + context
    - Output: file generati
    - Usa: Gemini API (quota alta) + Claude per review
```

### FASE 2: ASSEMBLY & VALIDATION (20h)
```
2.1 File Assembler (8h)
    - Crea struttura cartelle
    - Posiziona file generati
    - Risolve import/export
    
2.2 Validator (6h)
    - tsc --noEmit
    - prisma validate
    - eslint
    
2.3 Test Runner (4h)
    - vitest run
    - Report coverage
    
2.4 Builder (2h)
    - next build
    - Report errori
```

### FASE 3: INTERFACE & DEPLOY (25h)
```
3.1 CLI Interface (8h)
    - npx create-platform "prompt"
    - Interactive mode
    - Progress feedback
    
3.2 Ralph Integration (10h)
    - Wrapper per Claude Code
    - Context injection
    - Error recovery
    
3.3 Auto Deploy (6h)
    - Vercel CLI integration
    - GitHub repo creation
    - Environment setup
```

---

## 📈 ROADMAP PROPOSTA

```
SETTIMANA 1-2: CORE ENGINE
├── Day 1-2: Prompt Parser
├── Day 3-5: Decision Engine
├── Day 6-10: Code Generator (il più complesso)
└── Milestone: Genera primo progetto base

SETTIMANA 3: ASSEMBLY & VALIDATION
├── Day 1-2: File Assembler
├── Day 3-4: Validator + Test Runner
├── Day 5: Builder
└── Milestone: Progetto che compila e passa test

SETTIMANA 4: INTERFACE & POLISH
├── Day 1-3: CLI Interface
├── Day 4-5: Ralph Integration
├── Day 6-7: Deploy automation
└── Milestone: Tool completo end-to-end
```

---

## 🎯 PROSSIMO PASSO IMMEDIATO

**Iniziare con il PROMPT PARSER** - il componente più semplice ma fondamentale:

```python
# Esempio output atteso dal Parser

INPUT: "Voglio un e-commerce per vendere scarpe con auth, 
        pagamenti Stripe, dashboard admin, reviews prodotti"

OUTPUT:
{
    "platform_type": "ecommerce",
    "name": "shoe-store",
    "features": [
        {"id": "auth", "provider": "authjs", "methods": ["email", "google"]},
        {"id": "payments", "provider": "stripe", "features": ["checkout", "subscriptions"]},
        {"id": "admin", "type": "dashboard", "features": ["products", "orders", "users"]},
        {"id": "reviews", "type": "product-reviews", "features": ["rating", "comments"]}
    ],
    "entities": ["User", "Product", "Order", "Review", "Payment"],
    "catalogs_needed": [
        "CATALOGO-BOILERPLATE-CONFIG-v1.md",
        "CATALOGO-BOILERPLATE-STRUTTURA-v1.md",
        "CATALOGO-AUTHENTICATION-v1.md",
        "CATALOGO-ECOMMERCE-PRODUCTS-v1.md",
        "CATALOGO-ECOMMERCE-CART-v1.md",
        "CATALOGO-ECOMMERCE-ORDERS-v1.md",
        "CATALOGO-PAYMENTS-v1.md",
        "CATALOGO-ADMIN-BACKOFFICE-v1.md",
        "CATALOGO-DATABASE-PRISMA-v1.md",
        "CATALOGO-TESTING-VITEST-v1.md",
        "CATALOGO-DEVOPS-CI-CD-v1.md"
    ]
}
```

---

## ✅ RIEPILOGO

| Componente | Status | Voto |
|------------|--------|------|
| Knowledge Base (Cervello) | ✅ COMPLETO | 93/100 |
| Tool Engine (Corpo) | ❌ DA FARE | 0/100 |
| **PROGETTO TOTALE** | **~45% COMPLETO** | - |

**La knowledge base è pronta. Ora serve costruire il TOOL che la usa.**
