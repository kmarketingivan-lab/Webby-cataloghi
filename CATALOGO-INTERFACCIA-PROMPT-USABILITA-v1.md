# ============================================================================
§ CATALOGO INTERFACCIA PROMPT + REFERENCE USABILITÀ
# Versione: 1.0.0
# Ultima modifica: 2026-01-26
# ============================================================================

"""
SCOPO: Questo documento fornisce l'interfaccia completa per utilizzare
il sistema di cataloghi con Ralph (Claude Code CLI) in modo deterministico.

CONTENUTO:
1. Template Prompt per Ralph
2. Reference Guide - Navigazione Cataloghi
3. Workflow Usabilità - Flusso Operativo
4. Checklist Pre-Esecuzione
5. Troubleshooting e Soluzioni

REQUISITI:
- Tutti i cataloghi devono essere presenti nella stessa directory
- Ralph deve avere accesso in lettura/scrittura alla directory di lavoro
- PowerShell come shell predefinita (Windows)
"""

# ============================================================================
§ SEZIONE 1: INDICE CATALOGHI E RELAZIONI
# ============================================================================

§ 1.1 MAPPA CATALOGHI

CATALOGO-MASTER-INDEX.md
    │
    ├── CATALOGO-REQUISITI-FUNZIONALI-v1.md
    │       └── Definisce COSA costruire (features, user stories)
    │
    ├── CATALOGO-REQUISITI-ARCHITETTURA-v2.md
    │       └── Definisce COME strutturare (layer, pattern architetturali)
    │
    ├── CATALOGO-DATA-MODEL-v1.md
    │       └── Definisce i DATI (entità, relazioni, schemi)
    │
    ├── CATALOGO-API-v1.md
    │       └── Definisce le INTERFACCE (endpoints, contratti)
    │
    ├── CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
    │       └── Definisce lo STILE (colori, spacing, typography)
    │
    ├── CATALOGO-UI-PATTERN-PRIMITIVI-v1.md
    │       └── Definisce i COMPONENTI UI (pattern riutilizzabili)
    │
    ├── CATALOGO-CODICE-v1.md
    │       └── Definisce gli STANDARD CODICE (naming, struttura)
    │
    └── CATALOGO-AWS-DETERMINISTICO-v3.md
            └── Definisce l'INFRASTRUTTURA (servizi AWS, IaC)

§ 1.2 ORDINE DI LETTURA PER RALPH

Per ogni nuovo progetto, Ralph DEVE leggere i cataloghi in questo ordine:

1. CATALOGO-MASTER-INDEX.md          → Panoramica e navigazione
2. CATALOGO-REQUISITI-FUNZIONALI     → Capire cosa costruire
3. CATALOGO-REQUISITI-ARCHITETTURA   → Capire come strutturare
4. CATALOGO-DATA-MODEL               → Definire i dati
5. CATALOGO-API                      → Definire le interfacce
6. CATALOGO-DESIGN-TOKEN-SYSTEM      → Stile visivo
7. CATALOGO-UI-PATTERN-PRIMITIVI     → Componenti UI
8. CATALOGO-CODICE                   → Standard implementativi
9. CATALOGO-AWS-DETERMINISTICO       → Infrastruttura cloud

§ 1.3 LOOKUP RAPIDO PER CATEGORIA

§ QUANDO RALPH DEVE CREARE UN E-COMMERCE:
Leggere sezioni:
- REQUISITI-FUNZIONALI → Sezione E-Commerce
- DATA-MODEL → Entità: Product, Cart, Order, Payment
- API → Endpoints: /products, /cart, /orders, /checkout
- UI-PATTERN → Categoria E-Commerce (ProductCard, CartItem, Checkout)
- AWS → Pattern: API Gateway + Lambda + DynamoDB + S3

§ QUANDO RALPH DEVE CREARE UN SAAS:
Leggere sezioni:
- REQUISITI-FUNZIONALI → Sezione SaaS Multi-tenant
- DATA-MODEL → Entità: Tenant, User, Subscription, Usage
- API → Endpoints: /tenants, /users, /subscriptions
- UI-PATTERN → Categoria SaaS (Dashboard, Pricing, Onboarding)
- AWS → Pattern: Cognito + API Gateway + Lambda + RDS/DynamoDB

§ QUANDO RALPH DEVE CREARE HEALTHCARE (HIPAA):
Leggere sezioni:
- REQUISITI-FUNZIONALI → Sezione Healthcare
- REQUISITI-ARCHITETTURA → Compliance HIPAA
- DATA-MODEL → Entità: Patient, Provider, Appointment, MedicalRecord
- API → Endpoints con encryption e audit logging
- UI-PATTERN → Categoria Healthcare (PatientCard, Vitals, SecureMessage)
- AWS → Pattern: HIPAA-eligible services, encryption at rest/transit

§ QUANDO RALPH DEVE CREARE FINTECH (PCI-DSS):
Leggere sezioni:
- REQUISITI-FUNZIONALI → Sezione FinTech
- REQUISITI-ARCHITETTURA → Compliance PCI-DSS
- DATA-MODEL → Entità: Account, Transaction, Card (tokenized)
- API → Endpoints con MFA, rate limiting
- UI-PATTERN → Categoria FinTech (AccountCard, TransactionList, PaymentForm)
- AWS → Pattern: PCI-compliant architecture, KMS, WAF

# ============================================================================
§ SEZIONE 2: TEMPLATE PROMPT PER RALPH
# ============================================================================

§ 2.1 TEMPLATE MASTER - NUOVO PROGETTO COMPLETO

markdown
# ISTRUZIONI PER RALPH - NUOVO PROGETTO

## CONTESTO
Stai creando un nuovo progetto: [NOME_PROGETTO]
Categoria: [E-COMMERCE | SAAS | HEALTHCARE | FINTECH | IOT | SOCIAL | VIDEO]
Directory di lavoro: [PATH_ASSOLUTO]

## CATALOGHI DI RIFERIMENTO
Prima di scrivere qualsiasi codice, DEVI leggere questi cataloghi nell'ordine specificato:

1. Leggi: [PATH]/CATALOGO-MASTER-INDEX.md
2. Leggi: [PATH]/CATALOGO-REQUISITI-FUNZIONALI-v1.md → Sezione [CATEGORIA]
3. Leggi: [PATH]/CATALOGO-REQUISITI-ARCHITETTURA-v2.md
4. Leggi: [PATH]/CATALOGO-DATA-MODEL-v1.md → Sezione [CATEGORIA]
5. Leggi: [PATH]/CATALOGO-API-v1.md → Sezione [CATEGORIA]
6. Leggi: [PATH]/CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
7. Leggi: [PATH]/CATALOGO-UI-PATTERN-PRIMITIVI-v1.md → Sezione [CATEGORIA]
8. Leggi: [PATH]/CATALOGO-CODICE-v1.md
9. Leggi: [PATH]/CATALOGO-AWS-DETERMINISTICO-v3.md → Sezione [CATEGORIA]

## REQUISITI SPECIFICI
[Inserire qui i requisiti specifici del progetto]

## FASI DI ESECUZIONE
Esegui le seguenti fasi in ordine, verificando il completamento di ogni fase
prima di procedere alla successiva:

### FASE 1: Setup Progetto
- [ ] Creare struttura directory secondo CATALOGO-CODICE
- [ ] Inizializzare package.json con dipendenze standard
- [ ] Configurare TypeScript secondo standard
- [ ] Configurare ESLint e Prettier
- [ ] VERIFICA: Eseguire `pnpm install` senza errori

### FASE 2: Data Model
- [ ] Creare schemi Zod secondo CATALOGO-DATA-MODEL
- [ ] Creare tipi TypeScript derivati
- [ ] VERIFICA: Compilazione TypeScript senza errori

### FASE 3: Backend API
- [ ] Creare endpoint secondo CATALOGO-API
- [ ] Implementare validazione con Zod
- [ ] Implementare error handling standard
- [ ] VERIFICA: Test endpoint con curl/httpie

### FASE 4: Frontend UI
- [ ] Creare componenti secondo CATALOGO-UI-PATTERN
- [ ] Applicare Design Tokens da CATALOGO-DESIGN-TOKEN
- [ ] Implementare responsive design
- [ ] VERIFICA: Build frontend senza errori

### FASE 5: Integrazione
- [ ] Collegare frontend a backend
- [ ] Testare flussi completi
- [ ] VERIFICA: E2E test pass

### FASE 6: Infrastruttura (se richiesta)
- [ ] Creare IaC secondo CATALOGO-AWS
- [ ] Validare template CloudFormation/CDK
- [ ] VERIFICA: `cdk synth` o `cfn-lint` senza errori

## OUTPUT ATTESO
Al termine, la directory deve contenere:
[NOME_PROGETTO]/
├── package.json
├── tsconfig.json
├── src/
│   ├── backend/
│   ├── frontend/
│   └── shared/
├── tests/
├── infra/ (se richiesta)
└── README.md

## REGOLE CRITICHE
1. NON inventare pattern - usa SOLO quelli documentati nei cataloghi
2. NON saltare fasi - ogni fase deve essere completata e verificata
3. NON usare dipendenze non elencate nei cataloghi
4. OGNI file deve seguire gli standard di CATALOGO-CODICE
5. OGNI errore deve essere risolto prima di procedere

---

§ 2.2 TEMPLATE - SOLO BACKEND

markdown
# ISTRUZIONI PER RALPH - BACKEND API

## CONTESTO
Progetto: [NOME_PROGETTO]
Tipo: Backend API REST/GraphQL
Directory: [PATH_ASSOLUTO]

## CATALOGHI RICHIESTI
Leggi in ordine:
1. [PATH]/CATALOGO-REQUISITI-ARCHITETTURA-v2.md → Sezione Backend
2. [PATH]/CATALOGO-DATA-MODEL-v1.md → Entità rilevanti
3. [PATH]/CATALOGO-API-v1.md → Pattern e endpoint
4. [PATH]/CATALOGO-CODICE-v1.md → Standard TypeScript

## STACK TECNOLOGICO
- Runtime: Node.js 20 LTS
- Framework: Express.js / Fastify
- Language: TypeScript strict mode
- Validation: Zod
- Database: [DynamoDB | PostgreSQL | MongoDB]
- Package Manager: pnpm

## FASI
### FASE 1: Setup
- Inizializzare progetto TypeScript
- Configurare ESLint + Prettier
- VERIFICA: `pnpm build` OK

### FASE 2: Data Layer
- Creare schemi Zod per tutte le entità
- Creare repository pattern
- VERIFICA: Unit test schemi

### FASE 3: API Layer
- Creare router con endpoint
- Implementare middleware (auth, validation, error)
- VERIFICA: Test API con curl

### FASE 4: Business Logic
- Implementare service layer
- Aggiungere logging strutturato
- VERIFICA: Integration test

## OUTPUT
[NOME_PROGETTO]/
├── src/
│   ├── routes/
│   ├── services/
│   ├── repositories/
│   ├── schemas/
│   ├── middleware/
│   └── index.ts
├── tests/
├── package.json
└── tsconfig.json

---

§ 2.3 TEMPLATE - SOLO FRONTEND

markdown
# ISTRUZIONI PER RALPH - FRONTEND UI

## CONTESTO
Progetto: [NOME_PROGETTO]
Tipo: Frontend React/Next.js
Directory: [PATH_ASSOLUTO]
Categoria UI: [E-COMMERCE | SAAS | DASHBOARD | SOCIAL | etc.]

## CATALOGHI RICHIESTI
Leggi in ordine:
1. [PATH]/CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md → Tutti i token
2. [PATH]/CATALOGO-UI-PATTERN-PRIMITIVI-v1.md → Pattern rilevanti
3. [PATH]/CATALOGO-CODICE-v1.md → Standard React/TypeScript

## STACK TECNOLOGICO
- Framework: React 18 / Next.js 14
- Language: TypeScript strict mode
- Styling: Tailwind CSS v3
- State: [Zustand | React Query | Redux Toolkit]
- Package Manager: pnpm

## FASI
### FASE 1: Setup
- Creare progetto con Vite/Next.js
- Configurare Tailwind con Design Tokens
- VERIFICA: `pnpm dev` funziona

### FASE 2: Design System
- Configurare tailwind.config.js con token
- Creare file CSS con variabili custom
- VERIFICA: Token applicati correttamente

### FASE 3: Componenti Primitivi
- Creare Button, Input, Card, etc.
- Seguire esattamente pattern da catalogo
- VERIFICA: Storybook o test visuale

### FASE 4: Componenti Composti
- Creare componenti specifici categoria
- Comporre usando primitivi
- VERIFICA: Responsive e accessibile

### FASE 5: Pagine
- Creare layout e pagine
- Implementare routing
- VERIFICA: Navigazione funzionante

## OUTPUT
[NOME_PROGETTO]/
├── src/
│   ├── components/
│   │   ├── primitives/
│   │   └── composite/
│   ├── pages/
│   ├── layouts/
│   ├── hooks/
│   ├── styles/
│   │   ├── tokens.css
│   │   └── globals.css
│   └── App.tsx
├── tailwind.config.js
├── package.json
└── tsconfig.json

---

§ 2.4 TEMPLATE - INFRASTRUTTURA AWS

markdown
# ISTRUZIONI PER RALPH - INFRASTRUTTURA AWS

## CONTESTO
Progetto: [NOME_PROGETTO]
Tipo: Infrastructure as Code
Directory: [PATH_ASSOLUTO]
Regione: eu-south-1 (Milano) | eu-west-1 (Irlanda)

## CATALOGHI RICHIESTI
Leggi:
1. [PATH]/CATALOGO-AWS-DETERMINISTICO-v3.md → Pattern completo
2. [PATH]/CATALOGO-REQUISITI-ARCHITETTURA-v2.md → Requisiti infra

## VINCOLI CRITICI
- SOLO servizi AWS Always Free Tier (account 039288790906 - free tier scaduto)
- NO servizi a pagamento senza autorizzazione esplicita
- Regione preferita: eu-south-1 (Milano)

## SERVIZI CONSENTITI (Always Free)
- Lambda: 1M requests/mese
- DynamoDB: 25GB storage, 25 RCU/WCU
- S3: 5GB storage
- API Gateway: 1M calls/mese
- CloudWatch: 10 metriche custom
- SNS: 1M publish
- SQS: 1M requests
- Cognito: 50K MAU

## FASI
### FASE 1: Setup CDK
- Inizializzare progetto CDK TypeScript
- Configurare bootstrap se necessario
- VERIFICA: `cdk synth` OK

### FASE 2: Networking (se necessario)
- VPC con subnet pubbliche/private
- Security Groups
- VERIFICA: Template valido

### FASE 3: Compute
- Lambda functions
- API Gateway
- VERIFICA: Deploy dry-run

### FASE 4: Storage
- DynamoDB tables
- S3 buckets
- VERIFICA: IAM policies corrette

### FASE 5: Auth (se necessario)
- Cognito User Pool
- Identity Pool
- VERIFICA: Test auth flow

## OUTPUT
[NOME_PROGETTO]-infra/
├── bin/
│   └── app.ts
├── lib/
│   ├── stacks/
│   └── constructs/
├── cdk.json
├── package.json
└── tsconfig.json


# ============================================================================
§ SEZIONE 3: PROMPT SNIPPETS RIUTILIZZABILI
# ============================================================================

§ 3.1 SNIPPET - LETTURA CATALOGO SPECIFICA

markdown
## ISTRUZIONE LETTURA CATALOGO
Leggi il file [PATH]/[NOME_CATALOGO].md
Estrai e memorizza:
- Sezione: [NOME_SEZIONE]
- Pattern/Entità: [LISTA_SPECIFICA]
Conferma lettura con: "Ho letto [CATALOGO], sezione [SEZIONE]. Pattern estratti: [LISTA]"

§ 3.2 SNIPPET - VERIFICA PRE-FASE

markdown
## VERIFICA PRE-FASE [N]
Prima di iniziare la Fase [N], verifica:
1. Fase [N-1] completata senza errori
2. Tutti i file richiesti esistono
3. Tutti i test passano
4. Nessun TODO rimasto

Se tutte le verifiche passano, scrivi: "VERIFICA FASE [N-1]: PASSED - Procedo con Fase [N]"
Se fallisce, scrivi: "VERIFICA FASE [N-1]: FAILED - [MOTIVO]" e attendi istruzioni.

§ 3.3 SNIPPET - CREAZIONE FILE STANDARD

markdown
## CREAZIONE FILE
Crea il file: [PATH_COMPLETO]
Segui questi standard:
- Header comment con descrizione e data
- Import ordinati (built-in, external, internal)
- Export espliciti (no default export)
- Naming: [camelCase | PascalCase | kebab-case] secondo tipo file
- Max 300 righe per file (splitta se necessario)

Dopo creazione, verifica con: `tsc --noEmit [FILE]`

§ 3.4 SNIPPET - TEST ENDPOINT API

markdown
## TEST ENDPOINT
Testa l'endpoint: [METHOD] [URL]
Con payload: [JSON]
Aspettati:
- Status: [CODE]
- Response body contiene: [CAMPI]

Comando test:
curl -X [METHOD] [URL] \
  -H "Content-Type: application/json" \
  -d '[JSON]'

Se risposta corretta: "TEST PASSED: [ENDPOINT]"
Se errore: "TEST FAILED: [ENDPOINT] - [ERRORE]"

§ 3.5 SNIPPET - BUILD E VERIFICA

markdown
## BUILD E VERIFICA
Esegui in ordine:
1. `pnpm install` - Deve completare senza errori
2. `pnpm lint` - 0 errori, 0 warning
3. `pnpm build` - Build completata
4. `pnpm test` - Tutti i test passano

Report finale:
- Install: [OK/FAIL]
- Lint: [OK/FAIL] ([N] errori, [N] warning)
- Build: [OK/FAIL]
- Test: [OK/FAIL] ([N] passed, [N] failed)

§ 3.6 SNIPPET - GESTIONE ERRORE

markdown
## GESTIONE ERRORE
Si è verificato un errore: [DESCRIZIONE]

Azioni richieste:
1. NON procedere con altre fasi
2. Analizza l'errore e identifica la causa root
3. Proponi una soluzione specifica
4. Attendi approvazione prima di implementare
5. Dopo fix, ri-esegui la verifica della fase corrente

Output richiesto:
ERRORE RILEVATO
- Fase: [N]
- File: [PATH]
- Errore: [MESSAGGIO]
- Causa probabile: [ANALISI]
- Soluzione proposta: [FIX]
Attendo approvazione per procedere.

# ============================================================================
§ SEZIONE 4: WORKFLOW OPERATIVO COMPLETO
# ============================================================================

§ 4.1 WORKFLOW - NUOVO PROGETTO DA ZERO

┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW: NUOVO PROGETTO                              │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: PREPARAZIONE
├── 1.1 Definire categoria progetto
├── 1.2 Definire directory di lavoro
├── 1.3 Verificare prerequisiti (Node, pnpm, Git)
└── 1.4 Preparare prompt con template appropriato

STEP 2: LANCIO RALPH
├── 2.1 Eseguire: claude-code --prompt "[PROMPT]" --workdir "[DIR]"
├── 2.2 Attendere conferma lettura cataloghi
└── 2.3 Verificare output iniziale

STEP 3: MONITORAGGIO FASI
├── Per ogni fase:
│   ├── Attendere completamento
│   ├── Verificare output "VERIFICA FASE [N]: PASSED"
│   ├── Se FAILED: analizzare e autorizzare fix
│   └── Procedere solo dopo PASSED
└── Non interrompere durante esecuzione fase

STEP 4: VERIFICA FINALE
├── 4.1 Controllare struttura directory
├── 4.2 Eseguire build completa
├── 4.3 Eseguire test suite
├── 4.4 Verificare documentazione generata
└── 4.5 Commit iniziale se tutto OK

STEP 5: ITERAZIONE (se necessario)
├── Identificare gap o modifiche richieste
├── Creare prompt incrementale specifico
└── Ripetere da STEP 2

§ 4.2 WORKFLOW - AGGIUNTA FEATURE A PROGETTO ESISTENTE

┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW: NUOVA FEATURE                               │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: ANALISI
├── 1.1 Identificare cataloghi rilevanti per la feature
├── 1.2 Verificare se esistono pattern applicabili
└── 1.3 Mappare impatto su file esistenti

STEP 2: PROMPT INCREMENTALE
├── Template:
│   ```
│   # AGGIUNTA FEATURE: [NOME_FEATURE]
│   
│   ## CONTESTO
│   Progetto esistente: [PATH]
│   Feature da aggiungere: [DESCRIZIONE]
│   
│   ## CATALOGHI DA CONSULTARE
│   - [CATALOGO_1] → Sezione [X]
│   - [CATALOGO_2] → Sezione [Y]
│   
│   ## FILE DA MODIFICARE/CREARE
│   - [FILE_1]: [TIPO_MODIFICA]
│   - [FILE_2]: [TIPO_MODIFICA]
│   
│   ## VINCOLI
│   - Non modificare: [FILE_PROTETTI]
│   - Mantenere backward compatibility
│   - Aggiungere test per nuova feature
│   
│   ## VERIFICA
│   - Build deve passare
│   - Test esistenti devono continuare a passare
│   - Nuovi test devono coprire la feature
│   ```

STEP 3: ESECUZIONE
├── Lanciare Ralph con prompt incrementale
├── Monitorare modifiche
└── Verificare non ci siano regressioni

STEP 4: REVIEW
├── Diff dei file modificati
├── Esecuzione test completa
└── Merge se tutto OK

§ 4.3 WORKFLOW - FIX BUG

┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW: FIX BUG                                     │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: DIAGNOSI
├── Raccogliere informazioni bug:
│   - Descrizione comportamento atteso vs attuale
│   - Stack trace se disponibile
│   - Passi per riprodurre
└── Identificare file/componente coinvolto

STEP 2: PROMPT FIX
├── Template:
│   ```
│   # FIX BUG: [TITOLO_BUG]
│   
│   ## DESCRIZIONE
│   Comportamento attuale: [DESCRIZIONE]
│   Comportamento atteso: [DESCRIZIONE]
│   
│   ## RIPRODUZIONE
│   1. [STEP_1]
│   2. [STEP_2]
│   3. [STEP_N]
│   
│   ## FILE COINVOLTI
│   - [FILE_PATH]: [SOSPETTO]
│   
│   ## STACK TRACE (se disponibile)
│   ```
│   [STACK_TRACE]
│   ```
│   
│   ## ISTRUZIONI
│   1. Analizza il codice nei file coinvolti
│   2. Identifica la causa root
│   3. Proponi fix minimale
│   4. Aggiungi test che cattura il bug
│   5. Implementa fix
│   6. Verifica test passa
│   ```

STEP 3: VERIFICA
├── Test specifico per il bug passa
├── Test suite completa passa
└── Bug non si riproduce più

§ 4.4 WORKFLOW - REFACTORING

┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW: REFACTORING                                 │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: SCOPE
├── Definire esattamente cosa va refactorizzato
├── Verificare copertura test esistente
└── Identificare dipendenze

STEP 2: PROMPT REFACTORING
├── Template:
│   ```
│   # REFACTORING: [COMPONENTE/MODULO]
│   
│   ## OBIETTIVO
│   [DESCRIZIONE_OBIETTIVO]
│   
│   ## CATALOGHI RIFERIMENTO
│   - CATALOGO-CODICE → Standard da applicare
│   - [ALTRI_CATALOGHI] se pattern specifici
│   
│   ## SCOPE
│   File da refactorizzare:
│   - [FILE_1]
│   - [FILE_2]
│   
│   ## VINCOLI
│   - Zero breaking changes su API pubbliche
│   - Tutti i test esistenti devono passare
│   - Mantenere stessa funzionalità
│   
│   ## APPROCCIO
│   1. Prima i test (se mancanti)
│   2. Refactoring incrementale
│   3. Verifica dopo ogni step
│   ```

STEP 3: ESECUZIONE INCREMENTALE
├── Ogni modifica deve essere atomica
├── Verifica test dopo ogni modifica
└── Rollback se test falliscono

STEP 4: VALIDAZIONE
├── Code review delle modifiche
├── Test suite completa
└── Performance comparison se rilevante

# ============================================================================
§ SEZIONE 5: CHECKLIST PRE-ESECUZIONE
# ============================================================================

§ 5.1 CHECKLIST - AMBIENTE

markdown
## CHECKLIST AMBIENTE

### Sistema Operativo
- [ ] Windows 10/11 con PowerShell 7+
- [ ] Permessi amministratore se necessario
- [ ] Antivirus non blocca esecuzione script

### Runtime e Tools
- [ ] Node.js 20 LTS installato: `node --version` → v20.x.x
- [ ] pnpm installato: `pnpm --version` → 8.x.x o 9.x.x
- [ ] Git installato: `git --version` → 2.x.x
- [ ] TypeScript globale: `tsc --version` → 5.x.x

### AWS (se infrastruttura richiesta)
- [ ] AWS CLI configurato: `aws sts get-caller-identity`
- [ ] CDK installato: `cdk --version`
- [ ] Credenziali valide per account 039288790906
- [ ] Regione configurata: eu-south-1 o eu-west-1

### Claude Code
- [ ] Claude Code CLI installato e funzionante
- [ ] API key configurata
- [ ] Accesso a directory di lavoro

### Rete
- [ ] Accesso a npm/pnpm registry
- [ ] Accesso a GitHub (se necessario)
- [ ] Nessun proxy che blocca

§ 5.2 CHECKLIST - CATALOGHI

markdown
## CHECKLIST CATALOGHI

### Presenza File
- [ ] CATALOGO-MASTER-INDEX.md esiste
- [ ] CATALOGO-REQUISITI-FUNZIONALI-v1.md esiste
- [ ] CATALOGO-REQUISITI-ARCHITETTURA-v2.md esiste
- [ ] CATALOGO-DATA-MODEL-v1.md esiste
- [ ] CATALOGO-API-v1.md esiste
- [ ] CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md esiste
- [ ] CATALOGO-UI-PATTERN-PRIMITIVI-v1.md esiste
- [ ] CATALOGO-CODICE-v1.md esiste
- [ ] CATALOGO-AWS-DETERMINISTICO-v3.md esiste
- [ ] CATALOGO-INTERFACCIA-PROMPT-USABILITA-v1.md esiste

### Accessibilità
- [ ] Tutti i cataloghi nella stessa directory
- [ ] Directory accessibile in lettura
- [ ] Path assoluti noti e corretti

### Versioni
- [ ] Verificare che le versioni siano allineate
- [ ] Nessun catalogo obsoleto

§ 5.3 CHECKLIST - PROMPT

markdown
## CHECKLIST PROMPT

### Struttura
- [ ] Contesto chiaramente definito
- [ ] Categoria progetto specificata
- [ ] Directory di lavoro con path assoluto
- [ ] Cataloghi da leggere elencati in ordine

### Requisiti
- [ ] Requisiti specifici documentati
- [ ] Vincoli esplicitati (es. no servizi a pagamento)
- [ ] Output atteso descritto

### Fasi
- [ ] Fasi enumerate e ordinate
- [ ] Verifiche per ogni fase specificate
- [ ] Criteri di successo chiari

### Regole
- [ ] Regole critiche incluse
- [ ] Gestione errori specificata

§ 5.4 CHECKLIST - DIRECTORY PROGETTO

markdown
## CHECKLIST DIRECTORY PROGETTO

### Pre-Creazione
- [ ] Directory non esiste o è vuota
- [ ] Spazio disco sufficiente (almeno 1GB)
- [ ] Permessi scrittura sul path
- [ ] Path non contiene spazi o caratteri speciali

### Post-Creazione (da verificare)
- [ ] package.json presente e valido
- [ ] tsconfig.json presente e valido
- [ ] node_modules/ creato dopo install
- [ ] Struttura directory come da template
- [ ] README.md presente
- [ ] .gitignore presente


# ============================================================================
§ SEZIONE 6: TROUBLESHOOTING
# ============================================================================

§ 6.1 ERRORI COMUNI - SETUP

§ ERRORE: "PNPM: COMMAND NOT FOUND"
CAUSA: pnpm non installato o non nel PATH
SOLUZIONE:
1. Installare pnpm: npm install -g pnpm
2. Verificare: pnpm --version
3. Se ancora non funziona, aggiungere al PATH:
   - Windows: %APPDATA%\npm

§ ERRORE: "TSC: COMMAND NOT FOUND"
CAUSA: TypeScript non installato globalmente
SOLUZIONE:
1. Installare: pnpm add -g typescript
2. Verificare: tsc --version

§ ERRORE: "EACCES: PERMISSION DENIED"
CAUSA: Permessi insufficienti sulla directory
SOLUZIONE:
1. Windows: Eseguire PowerShell come amministratore
2. Verificare permessi cartella
3. Evitare directory di sistema

§ ERRORE: "ENOENT: NO SUCH FILE OR DIRECTORY"
CAUSA: Path non esiste o errato
SOLUZIONE:
1. Verificare path assoluto corretto
2. Creare directory parent se mancanti
3. Usare forward slash (/) anche su Windows

---

§ 6.2 ERRORI COMUNI - BUILD

§ ERRORE: "CANNOT FIND MODULE 'X'"
CAUSA: Dipendenza mancante o non installata
SOLUZIONE:
1. Verificare package.json contiene la dipendenza
2. Eseguire: pnpm install
3. Se persiste: pnpm add [PACKAGE_NAME]
4. Verificare import path corretto

§ ERRORE: "TYPE 'X' IS NOT ASSIGNABLE TO TYPE 'Y'"
CAUSA: Errore di tipizzazione TypeScript
SOLUZIONE:
1. Verificare tipi secondo CATALOGO-DATA-MODEL
2. Controllare che Zod schema e tipi siano allineati
3. Usare type assertion solo se necessario e documentato

§ ERRORE: "MODULE NOT FOUND: CAN'T RESOLVE 'X'"
CAUSA: Import errato o modulo non esportato
SOLUZIONE:
1. Verificare export nel modulo sorgente
2. Controllare path relativo/assoluto
3. Verificare tsconfig paths se configurati

§ ERRORE: "ESLINT: PARSING ERROR"
CAUSA: Sintassi non valida o config ESLint errata
SOLUZIONE:
1. Verificare sintassi TypeScript valida
2. Controllare .eslintrc.js configurazione
3. Verificare versione ESLint compatibile

---

§ 6.3 ERRORI COMUNI - RUNTIME

§ ERRORE: "TYPEERROR: CANNOT READ PROPERTY 'X' OF UNDEFINED"
CAUSA: Accesso a proprietà di oggetto null/undefined
SOLUZIONE:
1. Aggiungere null check: obj?.property
2. Verificare inizializzazione oggetto
3. Aggiungere validazione Zod in input

§ ERRORE: "CORS ERROR"
CAUSA: Cross-Origin Request bloccata
SOLUZIONE:
1. Backend: Aggiungere middleware CORS
   import cors from 'cors';
   app.use(cors({ origin: 'http://localhost:3000' }));
2. Verificare origin permessi corretti

§ ERRORE: "CONNECTION REFUSED" (DATABASE/API)
CAUSA: Servizio non raggiungibile
SOLUZIONE:
1. Verificare servizio in esecuzione
2. Controllare URL/porta corretti
3. Verificare firewall/security groups

---

§ 6.4 ERRORI COMUNI - AWS/CDK

§ ERRORE: "USER: ARN:AWS:IAM::... IS NOT AUTHORIZED"
CAUSA: Permessi IAM insufficienti
SOLUZIONE:
1. Verificare policy IAM dell'utente
2. Aggiungere permessi necessari
3. Controllare account ID corretto (039288790906)

§ ERRORE: "CDK BOOTSTRAP REQUIRED"
CAUSA: Account/regione non inizializzati per CDK
SOLUZIONE:
1. Eseguire: cdk bootstrap aws://039288790906/eu-south-1
2. Verificare: bucket CDK creato in S3

§ ERRORE: "RESOURCE LIMIT EXCEEDED"
CAUSA: Limiti AWS raggiunti
SOLUZIONE:
1. Verificare Free Tier limits
2. Eliminare risorse non utilizzate
3. Richiedere aumento limiti se necessario

§ ERRORE: "TEMPLATE FORMAT ERROR"
CAUSA: CloudFormation template non valido
SOLUZIONE:
1. Eseguire: cdk synth per vedere template
2. Validare con: cfn-lint template.yaml
3. Verificare riferimenti circolari

---

§ 6.5 ERRORI COMUNI - RALPH/CLAUDE CODE

§ ERRORE: RALPH NON LEGGE I CATALOGHI
CAUSA: Path errato o file non accessibile
SOLUZIONE:
1. Usare path assoluti nel prompt
2. Verificare file esistono con: ls [PATH]
3. Verificare permessi lettura

§ ERRORE: RALPH SALTA FASI
CAUSA: Prompt non chiaro o verifica non richiesta
SOLUZIONE:
1. Usare template con fasi esplicite
2. Richiedere output "VERIFICA FASE N: PASSED/FAILED"
3. Specificare: "Non procedere alla fase successiva senza verifica"

§ ERRORE: RALPH USA PATTERN NON IN CATALOGO
CAUSA: Istruzione non sufficientemente vincolante
SOLUZIONE:
1. Aggiungere regola: "USA SOLO pattern documentati nei cataloghi"
2. Specificare: "Se pattern non trovato, chiedere prima di inventare"
3. Elencare esplicitamente pattern consentiti

§ ERRORE: RALPH SI BLOCCA O LOOP INFINITO
CAUSA: Errore non gestito o condizione irrisolvibile
SOLUZIONE:
1. Interrompere esecuzione (Ctrl+C)
2. Analizzare ultimo output
3. Rilanciare con prompt più specifico
4. Aggiungere timeout se supportato

§ ERRORE: OUTPUT INCOMPLETO O TRONCATO
CAUSA: Limite token raggiunto
SOLUZIONE:
1. Dividere task in sotto-task più piccoli
2. Usare prompt incrementali
3. Chiedere output per file singolo

---

§ 6.6 MATRICE DIAGNOSTICA RAPIDA

┌────────────────────────┬─────────────────────┬───────────────────────────┐
│ SINTOMO                │ CAUSA PROBABILE     │ AZIONE IMMEDIATA          │
├────────────────────────┼─────────────────────┼───────────────────────────┤
│ Build fallisce         │ Dipendenza mancante │ pnpm install              │
│ Tipi non trovati       │ @types/ mancante    │ pnpm add -D @types/[pkg]  │
│ Import non risolto     │ Path errato         │ Verificare tsconfig paths │
│ CORS error             │ Backend config      │ Aggiungere cors middleware│
│ 401 Unauthorized       │ Auth mancante       │ Verificare token/header   │
│ 403 Forbidden          │ Permessi IAM        │ Verificare policy         │
│ 404 Not Found          │ Route non esiste    │ Verificare endpoint       │
│ 500 Internal Error     │ Bug nel codice      │ Controllare logs          │
│ Timeout                │ Servizio lento      │ Aumentare timeout/ottimizza│
│ Memory exceeded        │ Memory leak         │ Profile e ottimizza       │
└────────────────────────┴─────────────────────┴───────────────────────────┘

# ============================================================================
§ SEZIONE 7: ESEMPI PRONTI ALL'USO
# ============================================================================

§ 7.1 ESEMPIO COMPLETO - E-COMMERCE MVP

markdown
# ISTRUZIONI PER RALPH - E-COMMERCE MVP

## CONTESTO
Stai creando: ecommerce-mvp
Categoria: E-COMMERCE
Directory: C:\Users\cresc\Desktop\ecommerce-mvp

## CATALOGHI DI RIFERIMENTO
Leggi in ordine:
1. C:\Users\cresc\Desktop\CATALOGO-MASTER-INDEX.md
2. C:\Users\cresc\Desktop\CATALOGO-REQUISITI-FUNZIONALI-v1.md → Sezione E-Commerce
3. C:\Users\cresc\Desktop\CATALOGO-REQUISITI-ARCHITETTURA-v2.md
4. C:\Users\cresc\Desktop\CATALOGO-DATA-MODEL-v1.md → Entità: Product, Cart, Order
5. C:\Users\cresc\Desktop\CATALOGO-API-v1.md → Endpoints prodotti e ordini
6. C:\Users\cresc\Desktop\CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
7. C:\Users\cresc\Desktop\CATALOGO-UI-PATTERN-PRIMITIVI-v1.md → Sezione E-Commerce
8. C:\Users\cresc\Desktop\CATALOGO-CODICE-v1.md

## REQUISITI SPECIFICI
- Solo frontend (mock API per MVP)
- Pagine: Home, Catalogo Prodotti, Dettaglio Prodotto, Carrello, Checkout
- Mobile-first design
- Max 20 prodotti demo

## FASI

### FASE 1: Setup
- [ ] Creare progetto React + Vite + TypeScript
- [ ] Installare Tailwind CSS
- [ ] Configurare Design Tokens
- [ ] VERIFICA: `pnpm dev` avvia senza errori

### FASE 2: Componenti Base
- [ ] Creare Button, Input, Card secondo catalogo
- [ ] Creare Header e Footer
- [ ] VERIFICA: Componenti renderizzano correttamente

### FASE 3: Componenti E-Commerce
- [ ] ProductCard secondo pattern EC-1
- [ ] CartItem secondo pattern EC-2
- [ ] OrderSummary secondo pattern EC-4
- [ ] VERIFICA: Props e stili corretti

### FASE 4: Pagine
- [ ] HomePage con hero e prodotti featured
- [ ] CatalogPage con grid e filtri
- [ ] ProductPage con dettagli e add-to-cart
- [ ] CartPage con lista e summary
- [ ] CheckoutPage con form e stepper
- [ ] VERIFICA: Navigazione funziona

### FASE 5: State Management
- [ ] Cart state con Zustand
- [ ] Mock products data
- [ ] VERIFICA: Add/remove cart funziona

## OUTPUT
ecommerce-mvp/
├── src/
│   ├── components/
│   │   ├── primitives/
│   │   └── ecommerce/
│   ├── pages/
│   ├── store/
│   ├── data/
│   └── styles/
├── public/
└── package.json

## REGOLE
1. USA SOLO pattern da CATALOGO-UI-PATTERN-PRIMITIVI
2. USA SOLO token da CATALOGO-DESIGN-TOKEN-SYSTEM
3. Ogni fase deve avere VERIFICA PASSED prima di procedere

---

§ 7.2 ESEMPIO COMPLETO - SAAS DASHBOARD

markdown
# ISTRUZIONI PER RALPH - SAAS DASHBOARD

## CONTESTO
Stai creando: saas-dashboard
Categoria: SAAS
Directory: C:\Users\cresc\Desktop\saas-dashboard

## CATALOGHI DI RIFERIMENTO
Leggi in ordine:
1. C:\Users\cresc\Desktop\CATALOGO-MASTER-INDEX.md
2. C:\Users\cresc\Desktop\CATALOGO-REQUISITI-FUNZIONALI-v1.md → Sezione SaaS
3. C:\Users\cresc\Desktop\CATALOGO-DATA-MODEL-v1.md → Entità: User, Tenant, Subscription
4. C:\Users\cresc\Desktop\CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
5. C:\Users\cresc\Desktop\CATALOGO-UI-PATTERN-PRIMITIVI-v1.md → Sezione SaaS + Dashboard
6. C:\Users\cresc\Desktop\CATALOGO-CODICE-v1.md

## REQUISITI SPECIFICI
- Dashboard con sidebar navigation
- Pagine: Overview, Analytics, Settings, Team
- Grafici con Recharts
- Dark mode support

## FASI

### FASE 1: Setup
- [ ] Creare progetto Next.js 14 + TypeScript
- [ ] Configurare Tailwind con dark mode
- [ ] Setup Design Tokens
- [ ] VERIFICA: `pnpm dev` OK

### FASE 2: Layout
- [ ] AppShell con Sidebar
- [ ] Header con user menu
- [ ] Responsive sidebar (drawer su mobile)
- [ ] VERIFICA: Layout responsive corretto

### FASE 3: Dashboard Components
- [ ] StatCard/KPI cards
- [ ] ChartWrapper
- [ ] DataWidget
- [ ] ActivityFeed
- [ ] VERIFICA: Componenti funzionano

### FASE 4: Pages
- [ ] Overview con KPIs e charts
- [ ] Analytics con filtri e grafici
- [ ] Settings con form
- [ ] Team con member list
- [ ] VERIFICA: Navigazione completa

### FASE 5: Polish
- [ ] Dark mode toggle
- [ ] Loading states
- [ ] Empty states
- [ ] VERIFICA: UX completa

## OUTPUT
saas-dashboard/
├── src/
│   ├── app/
│   │   ├── dashboard/
│   │   ├── analytics/
│   │   ├── settings/
│   │   └── team/
│   ├── components/
│   └── lib/
└── package.json

---

§ 7.3 ESEMPIO COMPLETO - API BACKEND

markdown
# ISTRUZIONI PER RALPH - BACKEND API

## CONTESTO
Stai creando: api-backend
Categoria: BACKEND REST API
Directory: C:\Users\cresc\Desktop\api-backend

## CATALOGHI DI RIFERIMENTO
1. C:\Users\cresc\Desktop\CATALOGO-REQUISITI-ARCHITETTURA-v2.md
2. C:\Users\cresc\Desktop\CATALOGO-DATA-MODEL-v1.md
3. C:\Users\cresc\Desktop\CATALOGO-API-v1.md
4. C:\Users\cresc\Desktop\CATALOGO-CODICE-v1.md

## REQUISITI SPECIFICI
- Express.js + TypeScript
- Validazione con Zod
- In-memory database (per MVP)
- REST API per: Users, Products, Orders

## FASI

### FASE 1: Setup
- [ ] Init progetto TypeScript
- [ ] Installare Express, Zod, cors
- [ ] Configurare ESLint
- [ ] VERIFICA: `pnpm build` OK

### FASE 2: Schemas
- [ ] Creare schemas Zod per tutte le entità
- [ ] Derivare tipi TypeScript
- [ ] VERIFICA: tsc compila

### FASE 3: Routes
- [ ] GET/POST/PUT/DELETE /users
- [ ] GET/POST/PUT/DELETE /products
- [ ] GET/POST /orders
- [ ] VERIFICA: curl test passa

### FASE 4: Middleware
- [ ] Error handler globale
- [ ] Request validation middleware
- [ ] Logging middleware
- [ ] VERIFICA: Errori gestiti correttamente

## OUTPUT
api-backend/
├── src/
│   ├── routes/
│   ├── schemas/
│   ├── middleware/
│   ├── services/
│   └── index.ts
├── tests/
└── package.json

# ============================================================================
§ SEZIONE 8: REFERENCE CARD (QUICK LOOKUP)
# ============================================================================

§ 8.1 COMANDI FREQUENTI

bash
# Setup progetto
pnpm init
pnpm add typescript @types/node -D
pnpm add express zod cors
npx tsc --init

# Build e test
pnpm install
pnpm build
pnpm test
pnpm lint

# CDK
cdk init app --language typescript
cdk synth
cdk deploy
cdk destroy

# Verifica ambiente
node --version
pnpm --version
tsc --version
aws sts get-caller-identity

§ 8.2 TEMPLATE PROMPT MINIMO

markdown
# TASK: [DESCRIZIONE_BREVE]

## CATALOGHI
Leggi: [PATH]/[CATALOGO].md → Sezione [X]

## AZIONI
1. [AZIONE_1]
2. [AZIONE_2]

## VERIFICA
- [CRITERIO_1]
- [CRITERIO_2]

§ 8.3 CHECKLIST VELOCE

□ Ambiente: Node 20, pnpm, TypeScript
□ Cataloghi: Tutti presenti e accessibili
□ Prompt: Template appropriato compilato
□ Directory: Path valido, permessi OK
□ Rete: npm registry raggiungibile

§ 8.4 MAPPA CATALOGO → USO

Devo capire COSA costruire    → REQUISITI-FUNZIONALI
Devo capire COME strutturare  → REQUISITI-ARCHITETTURA
Devo definire i DATI          → DATA-MODEL
Devo definire le API          → API
Devo definire lo STILE        → DESIGN-TOKEN-SYSTEM
Devo creare COMPONENTI UI     → UI-PATTERN-PRIMITIVI
Devo scrivere CODICE          → CODICE
Devo creare INFRA AWS         → AWS-DETERMINISTICO
Devo creare PROMPT per Ralph  → INTERFACCIA-PROMPT-USABILITA

# ============================================================================
§ FINE CATALOGO INTERFACCIA PROMPT + REFERENCE USABILITÀ
# ============================================================================

"""
Questo catalogo fornisce tutto il necessario per utilizzare il sistema
di cataloghi con Ralph in modo efficace e deterministico.

Per aggiornamenti, verificare la versione nel CATALOGO-MASTER-INDEX.md

Versione: 1.0.0
Ultima modifica: 2026-01-26
"""


# ============================================================================
§ SEZIONE 9: FRAMEWORK DOMANDE PRELIMINARI (DISCOVERY)
# ============================================================================

"""
REGOLA FONDAMENTALE:
Prima di scrivere QUALSIASI codice, Ralph DEVE completare questa fase di discovery.
Le domande servono a raccogliere TUTTI i requisiti mancanti per massimizzare
la precisione della realizzazione.

PROCESSO:
1. Utente fornisce prompt iniziale
2. Ralph analizza prompt e identifica GAP
3. Ralph pone domande di discovery
4. Utente risponde
5. Ralph conferma comprensione con riepilogo
6. Solo dopo conferma utente → inizia sviluppo
"""

§ 9.1 DOMANDE OBBLIGATORIE - SEMPRE

§ CATEGORIA PROGETTO
□ Che tipo di piattaforma stai costruendo?
  - [ ] E-Commerce
  - [ ] SaaS / Dashboard
  - [ ] Social Network
  - [ ] Marketplace
  - [ ] Healthcare
  - [ ] FinTech
  - [ ] Video/Streaming
  - [ ] IoT
  - [ ] Blog/Content
  - [ ] Portfolio/Landing
  - [ ] Altro: ___________

□ È un MVP/prototipo o produzione?
  - [ ] MVP (velocità > perfezione)
  - [ ] Produzione (qualità > velocità)

□ Qual è la timeline?
  - [ ] ASAP / Prototipo veloce
  - [ ] 1-2 settimane
  - [ ] 1 mese
  - [ ] Timeline flessibile

§ UTENTI TARGET
□ Chi sono gli utenti principali?
  - Descrizione: ___________
  - Età/demografica: ___________
  - Competenza tecnica: [ ] Bassa [ ] Media [ ] Alta

□ Quanti utenti ti aspetti?
  - [ ] < 100 (piccolo)
  - [ ] 100-1000 (medio)
  - [ ] 1000-10000 (grande)
  - [ ] > 10000 (enterprise)

□ Ci sono utenti con ruoli diversi?
  - [ ] No, tutti uguali
  - [ ] Sì, quali ruoli? ___________

§ FUNZIONALITÀ CORE
□ Quali sono le 3-5 funzionalità ESSENZIALI (must-have)?
  1. ___________
  2. ___________
  3. ___________
  4. ___________
  5. ___________

□ Quali funzionalità sono nice-to-have (fase 2)?
  - ___________

□ C'è autenticazione utenti?
  - [ ] No
  - [ ] Sì, solo email/password
  - [ ] Sì, con social login (Google, etc.)
  - [ ] Sì, con MFA

□ C'è un sistema di pagamenti?
  - [ ] No
  - [ ] Sì, one-time payments
  - [ ] Sì, subscriptions
  - [ ] Sì, marketplace (multi-vendor)

§ DESIGN E UI
□ Hai un brand esistente?
  - [ ] No, crea tu
  - [ ] Sì, colori: ___________
  - [ ] Sì, ho logo/assets

□ Qual è lo stile visivo desiderato?
  - [ ] Minimal / Clean (tipo Linear, Notion)
  - [ ] Corporate / Professional (tipo Stripe)
  - [ ] Playful / Colorful (tipo Slack)
  - [ ] Dark / Developer (tipo Vercel, GitHub)
  - [ ] Luxury / Elegant
  - [ ] Altro/Reference: ___________

□ Mobile-first o Desktop-first?
  - [ ] Mobile-first (priorità mobile)
  - [ ] Desktop-first (priorità desktop)
  - [ ] Equal (entrambi importanti)

□ Hai screenshot/reference di design che ti piacciono?
  - [ ] No
  - [ ] Sì → [RICHIEDI INVIO IMMAGINI]

§ DATI E INTEGRAZIONI
□ Quali sono le entità/oggetti principali?
  (es: Prodotti, Utenti, Ordini, Post, etc.)
  - ___________

□ Ci sono integrazioni esterne richieste?
  - [ ] No
  - [ ] Payment gateway (Stripe, PayPal)
  - [ ] Email service (SendGrid, SES)
  - [ ] Analytics (Google Analytics, Mixpanel)
  - [ ] CRM (Salesforce, HubSpot)
  - [ ] Altro: ___________

□ Hai dati esistenti da importare?
  - [ ] No, partenza da zero
  - [ ] Sì, da CSV/Excel
  - [ ] Sì, da altro database
  - [ ] Sì, da API esterna

§ INFRASTRUTTURA
□ Dove vuoi hostare?
  - [ ] AWS (preferito, ho cataloghi)
  - [ ] Vercel/Netlify (frontend)
  - [ ] Altro cloud
  - [ ] Non so, decidi tu

□ Budget AWS?
  - [ ] Solo Free Tier (gratis)
  - [ ] Low cost (< $50/mese)
  - [ ] Standard (< $200/mese)
  - [ ] Enterprise (illimitato)

□ Requisiti di compliance?
  - [ ] Nessuno
  - [ ] GDPR
  - [ ] HIPAA (healthcare)
  - [ ] PCI-DSS (payments)
  - [ ] SOC2

---

§ 9.2 DOMANDE SPECIFICHE PER CATEGORIA

§ SE E-COMMERCE:
□ Quanti prodotti circa?
  - [ ] < 50
  - [ ] 50-500
  - [ ] 500-5000
  - [ ] > 5000

□ I prodotti hanno varianti (taglia, colore)?
  - [ ] No
  - [ ] Sì, semplici
  - [ ] Sì, complesse (combinazioni)

□ Gestione inventario?
  - [ ] No (prodotti sempre disponibili)
  - [ ] Sì, tracking stock
  - [ ] Sì, con notifiche low stock

□ Spedizioni?
  - [ ] Digitale (no spedizione)
  - [ ] Tariffa fissa
  - [ ] Calcolo per peso/zona
  - [ ] Integrazione corriere (UPS, FedEx)

□ Coupon/sconti?
  - [ ] No
  - [ ] Sì, codici sconto
  - [ ] Sì, sconti automatici

□ Checkout guest?
  - [ ] No, solo registrati
  - [ ] Sì, guest checkout permesso

§ SE SAAS:
□ Modello pricing?
  - [ ] Free only
  - [ ] Freemium
  - [ ] Trial + Paid
  - [ ] Solo paid

□ Multi-tenant?
  - [ ] No (single tenant)
  - [ ] Sì, database condiviso
  - [ ] Sì, database separati

□ Limiti per piano?
  - [ ] No
  - [ ] Sì, quali? (utenti, storage, features)

□ Team/collaborazione?
  - [ ] No, solo utenti singoli
  - [ ] Sì, inviti team
  - [ ] Sì, ruoli e permessi

□ Onboarding guidato?
  - [ ] No
  - [ ] Sì, wizard/checklist

§ SE MARKETPLACE:
□ Tipo marketplace?
  - [ ] Prodotti fisici
  - [ ] Prodotti digitali
  - [ ] Servizi
  - [ ] Booking/Prenotazioni

□ Commissioni?
  - [ ] No
  - [ ] Percentuale fissa
  - [ ] Variabile per categoria

□ Verifica venditori?
  - [ ] No, chiunque può vendere
  - [ ] Sì, approvazione manuale
  - [ ] Sì, KYC automatico

□ Gestione disputes?
  - [ ] No
  - [ ] Sì, sistema di reclami

§ SE HEALTHCARE:
□ Tipo dati sanitari?
  - [ ] Appuntamenti solo
  - [ ] Cartelle cliniche
  - [ ] Prescrizioni
  - [ ] Telemedicina

□ HIPAA compliance richiesta?
  - [ ] No (non USA)
  - [ ] Sì, necessaria
  - [ ] Non so → [ASSUMO SÌ]

□ Integrazione EMR/EHR?
  - [ ] No
  - [ ] Sì, quale sistema? ___________

□ Ruoli?
  - [ ] Solo pazienti
  - [ ] Pazienti + Medici
  - [ ] Pazienti + Medici + Admin
  - [ ] Più complesso: ___________

§ SE FINTECH:
□ Tipo operazioni finanziarie?
  - [ ] Solo visualizzazione
  - [ ] Trasferimenti interni
  - [ ] Trasferimenti esterni
  - [ ] Trading/Investimenti

□ PCI-DSS richiesto?
  - [ ] No (no card data)
  - [ ] Sì, gestisco carte
  - [ ] Parziale (uso Stripe Elements)

□ KYC/AML richiesto?
  - [ ] No
  - [ ] KYC base
  - [ ] KYC + AML completo

---

§ 9.3 CHECKLIST EDGE CASES

§ AUTENTICAZIONE
□ Cosa succede se:
  - Email già registrata? → [ERRORE | RECOVERY | MERGE]
  - Password dimenticata? → [LINK RESET | DOMANDA SEGRETA | SUPPORTO]
  - Link reset scaduto? → [NUOVO LINK | ERRORE | AUTO-REFRESH]
  - Troppi tentativi login? → [LOCKOUT TEMPO | CAPTCHA | BLOCK]
  - Sessione scade durante operazione? → [SALVA DRAFT | PERDI DATI | PROMPT LOGIN]
  - Utente cancella account? → [SOFT DELETE | HARD DELETE | ANONIMIZZA]

§ PAGAMENTI
□ Cosa succede se:
  - Pagamento fallisce? → [RETRY | CARRELLO SALVATO | NOTIFICA]
  - Pagamento pending lungo? → [TIMEOUT | WEBHOOK | POLLING]
  - Rimborso richiesto? → [AUTO | MANUALE | PARZIALE]
  - Subscription scade? → [GRACE PERIOD | DOWNGRADE | BLOCK]
  - Carta scade? → [NOTIFICA PRE | RETRY | SOSPENDI]
  - Disputa/chargeback? → [PROCESSO | NOTIFICA | AUTO-REFUND]

§ DATI E CONTENUTI
□ Cosa succede se:
  - Upload file troppo grande? → [ERRORE | COMPRESS | CHUNK]
  - Upload file tipo sbagliato? → [REJECT | CONVERT | WARN]
  - Contenuto inappropriato? → [MODERATION | FLAG | AUTO-REMOVE]
  - Database pieno? → [SCALE | NOTIFICA | BLOCK WRITE]
  - Dati corrotti? → [BACKUP | RETRY | NOTIFICA]

§ UI/UX
□ Cosa succede se:
  - JavaScript disabilitato? → [FALLBACK | MESSAGGIO | SSR]
  - Connessione lenta? → [LOADING | SKELETON | OFFLINE MODE]
  - Connessione persa durante form? → [AUTO-SAVE | WARN | QUEUE]
  - Browser non supportato? → [FALLBACK | MESSAGGIO | BLOCK]
  - Screen reader? → [ARIA COMPLETO | BASE | NON SUPPORTATO]

§ OPERAZIONI BUSINESS
□ Cosa succede se:
  - Stock esaurito durante checkout? → [ERRORE | WAITLIST | REMOVE]
  - Prezzo cambia durante checkout? → [PREZZO ORIGINALE | NUOVO | NOTIFICA]
  - Ordine annullato? → [REFUND AUTO | MANUALE | CREDITO]
  - Consegna fallita? → [RETRY | REFUND | PUNTO RITIRO]
  - Prodotto ritirato? → [NOTIFICA ACQUIRENTI | REFUND | SOSTITUZIONE]

---

§ 9.4 TEMPLATE RIEPILOGO CONFERMA

Dopo aver raccolto le risposte, Ralph deve produrre questo riepilogo:

markdown
# RIEPILOGO PROGETTO - CONFERMA PRIMA DI PROCEDERE

## Progetto
- **Nome:** [nome]
- **Categoria:** [categoria]
- **Tipo:** [MVP/Produzione]

## Utenti
- **Target:** [descrizione]
- **Ruoli:** [lista ruoli]
- **Volume atteso:** [numero]

## Funzionalità Core
1. [feature 1]
2. [feature 2]
3. [feature 3]

## Design
- **Stile:** [stile scelto]
- **Priorità:** [mobile/desktop]-first
- **Colori brand:** [colori o "da definire"]

## Tecnologia
- **Frontend:** [React/Next.js/etc]
- **Backend:** [Express/Lambda/etc]
- **Database:** [DynamoDB/PostgreSQL/etc]
- **Hosting:** [AWS/Vercel/etc]

## Integrazioni
- [lista integrazioni]

## Compliance
- [lista requisiti compliance]

## Edge Cases Critici Gestiti
- [lista decisioni edge cases]

---

**CONFERMI QUESTO RIEPILOGO?**
- [ ] Sì, procedi
- [ ] No, devo modificare: ___________

---

§ 9.5 WORKFLOW DISCOVERY COMPLETO

┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW DISCOVERY                                    │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: RICEZIONE PROMPT INIZIALE
├── Utente fornisce descrizione progetto
└── Ralph analizza e identifica:
    ├── Categoria (se chiara)
    ├── Informazioni già presenti
    └── GAP da colmare

STEP 2: DOMANDE DISCOVERY
├── Ralph pone domande obbligatorie (9.1)
├── Ralph pone domande specifiche categoria (9.2)
├── Ralph chiede conferma edge cases critici (9.3)
└── Utente risponde a tutto

STEP 3: RIEPILOGO E CONFERMA
├── Ralph produce riepilogo strutturato (9.4)
├── Utente conferma o corregge
└── Loop fino a conferma

STEP 4: MAPPING CATALOGHI
├── Ralph identifica sezioni cataloghi rilevanti
├── Ralph prepara piano di lettura
└── Output: lista cataloghi e sezioni da leggere

STEP 5: INIZIO SVILUPPO
├── Solo dopo conferma utente
├── Segue workflow standard (Sezione 4)
└── Verifica ogni fase

┌─────────────────────────────────────────────────────────────────────────────┐
│  REGOLA: MAI iniziare sviluppo senza aver completato Step 1-4              │
└─────────────────────────────────────────────────────────────────────────────┘


# ============================================================================
§ SEZIONE 10: WORKFLOW REVIEW E FIX FINALE
# ============================================================================

"""
Dopo che Ralph ha completato tutte le fasi di sviluppo, si esegue una review
sistematica per identificare e correggere eventuali problemi.
"""

§ 10.1 CHECKLIST REVIEW FINALE

§ STRUTTURA PROGETTO
□ Tutti i file previsti esistono
□ Struttura directory conforme a CATALOGO-CODICE
□ Nessun file orfano o non utilizzato
□ README.md presente e completo
□ .gitignore configurato correttamente
□ package.json ha tutti gli script necessari

§ QUALITÀ CODICE
□ `pnpm lint` passa senza errori
□ `pnpm build` completa senza errori
□ TypeScript strict mode attivo
□ Nessun `any` non necessario
□ Nessun `// @ts-ignore` non documentato
□ Nessun `console.log` di debug
□ Error handling presente ovunque
□ Commenti per logica complessa

§ FUNZIONALITÀ
□ Tutte le feature core funzionano
□ Autenticazione (se presente) funziona
□ Flussi principali testati manualmente
□ Form validation funziona
□ Error states mostrati correttamente
□ Loading states presenti
□ Empty states presenti

§ UI/UX
□ Design tokens applicati correttamente
□ Responsive su mobile (375px)
□ Responsive su tablet (768px)
□ Responsive su desktop (1280px)
□ Dark mode (se richiesto) funziona
□ Accessibilità base (keyboard nav, ARIA)
□ Immagini hanno alt text
□ Focus states visibili

§ PERFORMANCE
□ Bundle size ragionevole (< 500KB iniziale)
□ Immagini ottimizzate
□ Lazy loading dove appropriato
□ No memory leak evidenti
□ API calls non duplicate
□ Caching implementato dove serve

§ SICUREZZA
□ Input sanitizzati
□ No secrets in codice
□ CORS configurato correttamente
□ Auth tokens gestiti sicuramente
□ SQL injection prevention (se SQL)
□ XSS prevention

§ DATABASE
□ Schema corretto
□ Indici presenti per query frequenti
□ Relazioni corrette
□ Seed data funzionante (se presente)
□ Migration funzionanti (se presenti)

§ INFRASTRUTTURA (SE PRESENTE)
□ `cdk synth` passa
□ IAM policies least privilege
□ Security groups corretti
□ Environment variables configurate
□ Logging abilitato
□ Monitoring configurato

---

§ 10.2 PROCESSO FIX SISTEMATICO

┌─────────────────────────────────────────────────────────────────────────────┐
│                        PROCESSO FIX                                          │
└─────────────────────────────────────────────────────────────────────────────┘

STEP 1: ESEGUI CHECKLIST REVIEW
├── Completa ogni item della checklist 10.1
├── Documenta ogni problema trovato
└── Categorizza: [CRITICO | IMPORTANTE | MINORE]

STEP 2: PRIORITIZZA FIX
├── CRITICI → Fix immediato (bloccanti)
├── IMPORTANTI → Fix prima di release
└── MINORI → Fix se tempo permette

STEP 3: FIX CRITICI
├── Per ogni issue critico:
│   ├── Identifica file coinvolti
│   ├── Applica fix minimale
│   ├── Verifica fix non introduce regressioni
│   └── Documenta fix applicato

STEP 4: FIX IMPORTANTI
├── Stesso processo dei critici
└── Verifica che critici siano ancora OK

STEP 5: RE-RUN CHECKLIST
├── Ri-esegui checklist completa
├── Verifica tutti i fix applicati
└── Loop fino a checklist pulita

STEP 6: DOCUMENTAZIONE FINALE
├── Aggiorna README con eventuali note
├── Documenta decisioni tecniche
├── Lista known issues (se minori non fixati)
└── Istruzioni deploy/setup

---

§ 10.3 TEMPLATE REPORT REVIEW

markdown
# REVIEW REPORT - [NOME_PROGETTO]
Data: [DATA]

## SUMMARY
- Checklist items totali: [N]
- ✅ Passed: [N]
- ❌ Failed: [N]
- ⚠️ Warning: [N]

## ISSUES TROVATI

### CRITICI 🔴
| # | Descrizione | File | Fix Applicato |
|---|-------------|------|---------------|
| 1 | [desc] | [file] | [fix] |

### IMPORTANTI 🟡
| # | Descrizione | File | Fix Applicato |
|---|-------------|------|---------------|
| 1 | [desc] | [file] | [fix] |

### MINORI 🟢
| # | Descrizione | File | Status |
|---|-------------|------|--------|
| 1 | [desc] | [file] | [fixed/known issue] |

## KNOWN ISSUES (non fixati)
- [issue 1]: [motivo]
- [issue 2]: [motivo]

## RACCOMANDAZIONI POST-RELEASE
- [raccomandazione 1]
- [raccomandazione 2]

## SIGN-OFF
- [ ] Review completata
- [ ] Fix critici applicati
- [ ] Fix importanti applicati
- [ ] Pronto per release

---

§ 10.4 AUTOMAZIONE REVIEW (COMANDI)

bash
# Script review automatica (da eseguire in root progetto)

# 1. Lint check
echo "=== LINT CHECK ===" 
pnpm lint

# 2. Type check
echo "=== TYPE CHECK ===" 
pnpm tsc --noEmit

# 3. Build check
echo "=== BUILD CHECK ===" 
pnpm build

# 4. Test check
echo "=== TEST CHECK ===" 
pnpm test

# 5. Bundle analysis (se disponibile)
echo "=== BUNDLE SIZE ===" 
pnpm build --analyze || echo "No analyzer configured"

# 6. Security audit
echo "=== SECURITY AUDIT ===" 
pnpm audit

# 7. Outdated packages
echo "=== OUTDATED PACKAGES ===" 
pnpm outdated

---

# ============================================================================
§ FINE AGGIORNAMENTO CATALOGO INTERFACCIA PROMPT
# ============================================================================

"""
Sezioni aggiunte:
- 9: Framework Domande Preliminari (Discovery)
- 10: Workflow Review e Fix Finale

Il catalogo ora include il ciclo completo:
1. Discovery (domande preliminari)
2. Sviluppo (workflow esistenti)
3. Review e Fix (nuova sezione)
"""


# ============================================================================
§ SEZIONE 11: ESEMPI PRATICI DI UTILIZZO COMPLETO
# ============================================================================

"""
Questa sezione mostra esempi end-to-end di come usare il sistema di cataloghi
per creare progetti reali. Ogni esempio segue il workflow completo:
Discovery → Selezione Stile → Sviluppo → Review
"""

§ 11.1 ESEMPIO COMPLETO: E-COMMERCE MVP "PETSHOP"

§ FASE 1: PROMPT INIZIALE UTENTE
"Voglio creare un e-commerce per vendere prodotti per animali domestici.
Deve essere semplice, con catalogo prodotti, carrello e checkout.
Budget limitato, voglio usare solo servizi gratuiti AWS."

§ FASE 2: DISCOVERY (DOMANDE CHE RALPH PONE)

RALPH: Prima di iniziare, ho bisogno di alcune informazioni per assicurarmi
di creare esattamente ciò che ti serve.

CATEGORIA E SCOPE:
✓ Tipo: E-Commerce
✓ MVP o Produzione? MVP (velocità > perfezione)
✓ Timeline: ASAP / Prototipo veloce

UTENTI:
1. Chi sono gli utenti principali?
   → Proprietari di animali domestici, età 25-55, competenza tecnica media

2. Quanti utenti ti aspetti inizialmente?
   → < 100 (piccolo, per test)

3. Ci sono ruoli diversi?
   → No admin per ora, solo clienti

FUNZIONALITÀ CORE (must-have):
1. Catalogo prodotti con categorie (Cani, Gatti, Altro)
2. Carrello
3. Checkout con pagamento
4. Nessuna autenticazione richiesta (guest checkout)

DESIGN:
1. Hai un brand esistente?
   → No, crea tu. Colori: verde/naturale

2. Stile visivo desiderato?
   → Friendly, non troppo corporate

3. Mobile-first o Desktop-first?
   → Mobile-first (molti utenti da smartphone)

DATI:
1. Quanti prodotti circa?
   → ~50 prodotti iniziali

2. Prodotti con varianti?
   → Sì, semplici (taglia per cibo: S/M/L)

3. Gestione inventario?
   → No per MVP

4. Integrazioni esterne?
   → Stripe per pagamenti

INFRASTRUTTURA:
1. Hosting: AWS Free Tier only
2. Budget: €0 (solo free tier)
3. Compliance: GDPR base (privacy policy)

EDGE CASES CONFERMATI:
- Stock esaurito: mostra "Non disponibile" ma permetti wishlist
- Pagamento fallito: mostra errore, mantieni carrello
- Carrello abbandonato: nessuna email per MVP

§ FASE 3: RIEPILOGO CONFERMA

markdown
# RIEPILOGO PROGETTO - PetShop MVP

## Progetto
- **Nome:** PetShop
- **Categoria:** E-Commerce
- **Tipo:** MVP

## Utenti
- **Target:** Proprietari animali domestici, 25-55 anni
- **Ruoli:** Solo clienti (no admin)
- **Volume:** < 100 utenti test

## Funzionalità Core
1. Catalogo prodotti (~50) con 3 categorie
2. Carrello persistente (localStorage)
3. Checkout guest con Stripe
4. Varianti semplici (taglia)

## Design
- **Stile:** Airbnb-inspired (friendly, visual)
- **Priorità:** Mobile-first
- **Colori:** Verde naturale (#10B981), neutri

## Tecnologia
- **Frontend:** Next.js 14 + TypeScript
- **Backend:** API Routes Next.js (serverless)
- **Database:** JSON file per MVP (no DB)
- **Payments:** Stripe Checkout
- **Hosting:** Vercel (free tier)

## Edge Cases
- Stock esaurito → "Non disponibile" + wishlist
- Pagamento fallito → Errore + carrello mantenuto

---
**CONFERMI?** [Sì, procedi]

§ FASE 4: MAPPING CATALOGHI

yaml
cataloghi_da_leggere:
  
  1_requisiti:
    file: CATALOGO-REQUISITI-FUNZIONALI-v1.md
    sezioni:
      - E-Commerce module
      - Product entity
      - Cart operations
      - Checkout flow
  
  2_design_reference:
    file: CATALOGO-DESIGN-REFERENCE-v1.md
    sezioni:
      - Stile Airbnb (1.5)
      - E-Commerce Listing (2.3)
      - E-Commerce Detail (2.4)
      - Checkout Page (2.8)
      - Composizioni E-Commerce (3.1)
  
  3_design_tokens:
    file: CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md
    sezioni:
      - Colors (custom green primary)
      - Typography
      - Spacing
      - Border radius (grande per friendly feel)
  
  4_ui_patterns:
    file: CATALOGO-UI-PATTERN-PRIMITIVI-v1.md
    sezioni:
      - Layout patterns
      - Navigation (Header e-commerce)
      - Content (Card, List)
      - Input (Select, Number input)
      - Action (Button)
      - E-Commerce patterns (ProductCard, CartItem)
  
  5_codice:
    file: CATALOGO-CODICE-v1.md
    sezioni:
      - Next.js structure
      - TypeScript standards
      - Zod schemas
      - API routes pattern

§ FASE 5: STRUTTURA PROGETTO GENERATA

petshop-mvp/
├── src/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Homepage
│   │   ├── products/
│   │   │   ├── page.tsx          # Listing
│   │   │   └── [slug]/
│   │   │       └── page.tsx      # Product detail
│   │   ├── cart/
│   │   │   └── page.tsx          # Cart
│   │   ├── checkout/
│   │   │   ├── page.tsx          # Checkout form
│   │   │   └── success/
│   │   │       └── page.tsx      # Order confirmation
│   │   └── api/
│   │       ├── products/
│   │       │   └── route.ts
│   │       └── checkout/
│   │           └── route.ts      # Stripe integration
│   │
│   ├── components/
│   │   ├── primitives/
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Input.tsx
│   │   │   └── Badge.tsx
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Container.tsx
│   │   └── ecommerce/
│   │       ├── ProductCard.tsx
│   │       ├── ProductGrid.tsx
│   │       ├── CartItem.tsx
│   │       ├── CartSummary.tsx
│   │       └── CategoryFilter.tsx
│   │
│   ├── lib/
│   │   ├── products.ts           # Product data/API
│   │   ├── cart.ts               # Cart logic (localStorage)
│   │   └── stripe.ts             # Stripe client
│   │
│   ├── schemas/
│   │   ├── product.ts            # Zod schema
│   │   └── cart.ts               # Zod schema
│   │
│   ├── hooks/
│   │   ├── useCart.ts
│   │   └── useProducts.ts
│   │
│   └── styles/
│       ├── globals.css
│       └── tokens.css
│
├── data/
│   └── products.json             # Product data (MVP)
│
├── public/
│   └── images/
│       └── products/
│
├── tailwind.config.js
├── next.config.js
├── package.json
└── tsconfig.json

§ FASE 6: DESIGN TOKENS APPLICATI

css
/* styles/tokens.css - PetShop Theme */

:root {
  /* Brand Colors - Green Nature Theme */
  --color-primary: #10B981;        /* Emerald 500 */
  --color-primary-hover: #059669;  /* Emerald 600 */
  --color-primary-light: #D1FAE5;  /* Emerald 100 */
  
  /* Surfaces */
  --color-surface-default: #FFFFFF;
  --color-surface-subtle: #F9FAFB;
  --color-surface-muted: #F3F4F6;
  
  /* Text */
  --color-text-default: #111827;
  --color-text-muted: #6B7280;
  --color-text-subtle: #9CA3AF;
  
  /* Feedback */
  --color-success: #10B981;
  --color-error: #EF4444;
  --color-warning: #F59E0B;
  
  /* Border Radius - Friendly/Rounded */
  --radius-sm: 0.375rem;   /* 6px */
  --radius-md: 0.5rem;     /* 8px */
  --radius-lg: 0.75rem;    /* 12px */
  --radius-xl: 1rem;       /* 16px - for cards */
  --radius-full: 9999px;   /* pills */
  
  /* Shadows - Soft */
  --shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
  --shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
  --shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
}

§ FASE 7: COMPONENTE ESEMPIO (PRODUCTCARD)

tsx
// components/ecommerce/ProductCard.tsx
// Pattern: CATALOGO-UI-PATTERN-PRIMITIVI → E-Commerce → EC-1

import { Product } from '@/schemas/product';
import { Badge } from '@/components/primitives/Badge';
import { Button } from '@/components/primitives/Button';
import { useCart } from '@/hooks/useCart';

interface ProductCardProps {
  product: Product;
}

export function ProductCard({ product }: ProductCardProps) {
  const { addItem } = useCart();
  
  const isOutOfStock = product.stock === 0;
  const hasDiscount = product.compareAtPrice > product.price;
  
  return (
    <div className="group bg-surface-default rounded-xl border border-border-default 
                    overflow-hidden transition-shadow hover:shadow-lg">
      {/* Image */}
      <div className="relative aspect-square bg-surface-muted">
        <img
          src={product.image}
          alt={product.name}
          className="w-full h-full object-cover"
        />
        
        {/* Badges */}
        <div className="absolute top-3 left-3 flex flex-col gap-2">
          {hasDiscount && (
            <Badge variant="error">
              -{Math.round((1 - product.price / product.compareAtPrice) * 100)}%
            </Badge>
          )}
          {isOutOfStock && (
            <Badge variant="default">Esaurito</Badge>
          )}
        </div>
        
        {/* Quick Add - visible on hover */}
        {!isOutOfStock && (
          <div className="absolute inset-x-3 bottom-3 opacity-0 
                          group-hover:opacity-100 transition-opacity">
            <Button
              variant="primary"
              size="sm"
              fullWidth
              onClick={() => addItem(product)}
            >
              Aggiungi al carrello
            </Button>
          </div>
        )}
      </div>
      
      {/* Content */}
      <div className="p-4">
        <p className="text-xs text-text-muted mb-1">{product.category}</p>
        <h3 className="font-medium text-text-default line-clamp-2 mb-2">
          {product.name}
        </h3>
        
        {/* Price */}
        <div className="flex items-center gap-2">
          <span className="text-lg font-bold text-text-default">
            €{product.price.toFixed(2)}
          </span>
          {hasDiscount && (
            <span className="text-sm text-text-muted line-through">
              €{product.compareAtPrice.toFixed(2)}
            </span>
          )}
        </div>
      </div>
    </div>
  );
}

§ FASE 8: REVIEW CHECKLIST

markdown
## REVIEW - PetShop MVP

### Struttura ✅
- [x] Tutti i file previsti esistono
- [x] Struttura Next.js App Router corretta
- [x] package.json completo

### Codice ✅
- [x] TypeScript strict mode
- [x] ESLint passa
- [x] Build completa

### Funzionalità ✅
- [x] Homepage con prodotti featured
- [x] Listing con filtri categoria
- [x] Product detail con varianti
- [x] Carrello funzionante (localStorage)
- [x] Checkout Stripe integrato

### UI/UX ✅
- [x] Mobile responsive (375px testato)
- [x] Design tokens applicati
- [x] Loading states
- [x] Empty states

### Issues Trovati
- Nessun issue critico
- Minor: Aggiungere SEO meta tags (post-MVP)

### RISULTATO: READY FOR DEPLOY ✅

---

§ 11.2 ESEMPIO COMPLETO: SAAS DASHBOARD "TASKFLOW"

§ FASE 1: PROMPT INIZIALE
"Voglio creare una dashboard per gestione task/progetti per piccoli team.
Deve avere autenticazione, progetti, task con status, e un dashboard overview.
Stile moderno, developer-friendly."

§ FASE 2: DISCOVERY COMPLETATO

yaml
categoria: SaaS
tipo: MVP con auth

utenti:
  target: Piccoli team tech (3-10 persone)
  ruoli:
    - Admin (crea progetti, gestisce team)
    - Member (vede progetti assegnati, gestisce task)
  volume: < 50 utenti iniziali

funzionalità_core:
  1: Autenticazione (email/password, Google OAuth)
  2: Dashboard overview (task assegnati, scadenze)
  3: Progetti (CRUD, assegna membri)
  4: Task (CRUD, status, assegnee, due date)
  5: Niente billing per MVP

design:
  stile: Linear (dark mode, moderno)
  priorità: Desktop-first (team lavora da computer)
  colori: Viola/blu come Linear

tech:
  frontend: Next.js 14
  backend: Next.js API Routes
  database: Supabase (PostgreSQL + Auth)
  hosting: Vercel

edge_cases:
  - Task senza assegnee: permesso, mostra "Unassigned"
  - Progetto senza task: mostra empty state con CTA
  - Utente rimosso da progetto: task rimangono ma reassign

§ FASE 3: MAPPING CATALOGHI

yaml
cataloghi:
  
  design_reference:
    stile: Linear (1.2)
    layout: Dashboard Sidebar (2.2A)
    settings: Settings Page (2.6)
  
  ui_patterns:
    - Navigation > Sidebar
    - Navigation > Tabs
    - Content > Card, Badge, Avatar
    - Input > Text, Select, DatePicker
    - Data Display > Table, StatCard
    - Overlay > Modal, Command Palette
    - SaaS > Onboarding Checklist
  
  data_model:
    entità:
      - User (id, email, name, avatar, role)
      - Project (id, name, description, members[], createdAt)
      - Task (id, title, description, status, priority, assigneeId, projectId, dueDate)
  
  api:
    - GET/POST /api/projects
    - GET/POST /api/projects/[id]/tasks
    - PATCH /api/tasks/[id]
    - GET /api/dashboard/stats

§ FASE 4: STRUTTURA GENERATA

taskflow/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/page.tsx
│   │   │   └── signup/page.tsx
│   │   ├── (dashboard)/
│   │   │   ├── layout.tsx         # Sidebar layout
│   │   │   ├── page.tsx           # Dashboard overview
│   │   │   ├── projects/
│   │   │   │   ├── page.tsx       # Projects list
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx   # Project detail + tasks
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   └── api/
│   │       ├── auth/
│   │       ├── projects/
│   │       └── tasks/
│   │
│   ├── components/
│   │   ├── primitives/
│   │   ├── layout/
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Header.tsx
│   │   │   └── CommandPalette.tsx
│   │   ├── dashboard/
│   │   │   ├── StatCard.tsx
│   │   │   ├── TaskList.tsx
│   │   │   └── ActivityFeed.tsx
│   │   └── projects/
│   │       ├── ProjectCard.tsx
│   │       ├── TaskRow.tsx
│   │       └── TaskModal.tsx
│   │
│   ├── lib/
│   │   ├── supabase.ts
│   │   └── auth.ts
│   │
│   └── styles/
│       └── tokens.css      # Linear-style dark theme
│
└── ...

§ FASE 5: DESIGN TOKENS (LINEAR DARK)

css
/* Linear-inspired dark theme */
:root {
  --color-primary: #8B5CF6;        /* Violet */
  --color-primary-hover: #7C3AED;
  
  --color-bg-primary: #0A0A0B;     /* Near black */
  --color-bg-secondary: #1A1A1C;   /* Dark gray */
  --color-bg-tertiary: #2A2A2E;    /* Lighter gray */
  
  --color-text-primary: #FFFFFF;
  --color-text-secondary: #A1A1AA;
  --color-text-tertiary: #71717A;
  
  --color-border: #27272A;
  --color-border-hover: #3F3F46;
  
  /* Status colors */
  --color-status-todo: #71717A;
  --color-status-in-progress: #3B82F6;
  --color-status-done: #10B981;
  
  --radius-md: 6px;
  --radius-lg: 8px;
}

---

§ 11.3 TEMPLATE QUICK START

Per progetti futuri, usa questo template minimale:

markdown
# QUICK START - [Nome Progetto]

## 1. DISCOVERY RAPIDO
- Categoria: [E-Commerce | SaaS | Social | Dashboard | Healthcare | FinTech]
- Tipo: [MVP | Production]
- Stile: [Stripe | Linear | Notion | Vercel | Airbnb | Shopify | Slack | GitHub]

## 2. FUNZIONALITÀ (max 5)
1. _____________
2. _____________
3. _____________

## 3. TECH STACK
- Frontend: [Next.js | React + Vite]
- Backend: [Next.js API | Express | Lambda]
- Database: [Supabase | DynamoDB | PostgreSQL]
- Auth: [Supabase | Cognito | NextAuth]

## 4. PAGINE PRINCIPALI
1. _____________
2. _____________
3. _____________

## 5. CATALOGHI DA CONSULTARE
- [ ] DESIGN-REFERENCE → Stile _____, Layout _____
- [ ] UI-PATTERN-PRIMITIVI → Sezione _____
- [ ] DESIGN-TOKEN-SYSTEM → Customizzazioni _____
- [ ] CODICE → Pattern _____

## 6. GO!


# ============================================================================
§ SEZIONE 12: CROSS-REFERENCE TRA CATALOGHI
# ============================================================================

"""
Questa sezione fornisce mapping rapidi per trovare informazioni specifiche
attraverso tutti i cataloghi. Utile per lookup veloce durante lo sviluppo.
"""

§ 12.1 MAPPING: "HO BISOGNO DI..." → CATALOGO

┌─────────────────────────────────────────────────────────────────────────────┐
│ HO BISOGNO DI...                    │ CATALOGO DA CONSULTARE               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CAPIRE COSA COSTRUIRE                                                       │
│ ─────────────────────                                                       │
│ User stories e requisiti            │ REQUISITI-FUNZIONALI                 │
│ Flussi utente                       │ REQUISITI-FUNZIONALI                 │
│ Business rules                      │ REQUISITI-FUNZIONALI                 │
│ Casi d'uso per categoria            │ REQUISITI-FUNZIONALI + DESIGN-REFERENCE│
│                                                                             │
│ STRUTTURARE L'ARCHITETTURA                                                  │
│ ──────────────────────────                                                  │
│ Layer applicativi                   │ REQUISITI-ARCHITETTURA               │
│ Pattern architetturali              │ REQUISITI-ARCHITETTURA               │
│ Security patterns                   │ REQUISITI-ARCHITETTURA               │
│ Compliance (HIPAA, PCI)             │ REQUISITI-ARCHITETTURA               │
│                                                                             │
│ DEFINIRE I DATI                                                             │
│ ───────────────                                                             │
│ Schema entità                       │ DATA-MODEL                           │
│ Relazioni tra entità                │ DATA-MODEL                           │
│ DynamoDB design                     │ DATA-MODEL                           │
│ PostgreSQL schema                   │ DATA-MODEL                           │
│ Zod schemas                         │ CODICE                               │
│                                                                             │
│ CREARE API                                                                  │
│ ──────────                                                                  │
│ Endpoint definitions                │ API                                  │
│ Request/Response schemas            │ API + DATA-MODEL                     │
│ Error codes                         │ API                                  │
│ Auth patterns                       │ API + CODICE                         │
│                                                                             │
│ SCEGLIERE LO STILE VISIVO                                                   │
│ ─────────────────────────                                                   │
│ Stile generale                      │ DESIGN-REFERENCE (Sezione 1)         │
│ Palette colori                      │ DESIGN-TOKEN-SYSTEM                  │
│ Typography                          │ DESIGN-TOKEN-SYSTEM                  │
│ Spacing system                      │ DESIGN-TOKEN-SYSTEM                  │
│                                                                             │
│ STRUTTURARE LE PAGINE                                                       │
│ ─────────────────────                                                       │
│ Layout pagina                       │ DESIGN-REFERENCE (Sezione 2)         │
│ Wireframe                           │ DESIGN-REFERENCE (Sezione 2)         │
│ Pagine per categoria                │ DESIGN-REFERENCE (Sezione 3)         │
│                                                                             │
│ COSTRUIRE COMPONENTI UI                                                     │
│ ───────────────────────                                                     │
│ Button, Input, Card, etc.           │ UI-PATTERN-PRIMITIVI (1-8)           │
│ Navigation components               │ UI-PATTERN-PRIMITIVI (Sezione 2)     │
│ Form components                     │ UI-PATTERN-PRIMITIVI (Sezione 4)     │
│ Feedback components                 │ UI-PATTERN-PRIMITIVI (Sezione 5)     │
│ Data display                        │ UI-PATTERN-PRIMITIVI (Sezione 7)     │
│ E-commerce specific                 │ UI-PATTERN-PRIMITIVI (Categoria)     │
│ SaaS specific                       │ UI-PATTERN-PRIMITIVI (Categoria)     │
│ Responsive patterns                 │ UI-PATTERN-PRIMITIVI (Sezione 9)     │
│ Animation                           │ UI-PATTERN-PRIMITIVI (Sezione 10)    │
│ Accessibility                       │ UI-PATTERN-PRIMITIVI (Sezione 11)    │
│                                                                             │
│ SCRIVERE CODICE                                                             │
│ ──────────────                                                              │
│ Struttura progetto                  │ CODICE                               │
│ Naming conventions                  │ CODICE                               │
│ TypeScript patterns                 │ CODICE                               │
│ React patterns                      │ CODICE                               │
│ Testing patterns                    │ CODICE                               │
│                                                                             │
│ CREARE INFRASTRUTTURA AWS                                                   │
│ ─────────────────────────                                                   │
│ Servizi AWS free tier               │ AWS-DETERMINISTICO                   │
│ CDK templates                       │ AWS-DETERMINISTICO                   │
│ Lambda patterns                     │ AWS-DETERMINISTICO + CODICE          │
│ DynamoDB setup                      │ AWS-DETERMINISTICO + DATA-MODEL      │
│ API Gateway                         │ AWS-DETERMINISTICO                   │
│ Cognito auth                        │ AWS-DETERMINISTICO                   │
│                                                                             │
│ USARE I CATALOGHI                                                           │
│ ─────────────────                                                           │
│ Template prompt                     │ INTERFACCIA-PROMPT-USABILITA         │
│ Workflow operativi                  │ INTERFACCIA-PROMPT-USABILITA         │
│ Domande discovery                   │ INTERFACCIA-PROMPT-USABILITA         │
│ Troubleshooting                     │ INTERFACCIA-PROMPT-USABILITA         │
│ Esempi pratici                      │ INTERFACCIA-PROMPT-USABILITA         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 12.2 MAPPING: COMPONENTE UI → PATTERN + TOKENS

yaml
# Per ogni componente, dove trovare specifiche

Button:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 8.1
  tokens:
    colori: DESIGN-TOKEN-SYSTEM → colors.interactive
    spacing: DESIGN-TOKEN-SYSTEM → spacing
    radius: DESIGN-TOKEN-SYSTEM → borderRadius.md
    typography: DESIGN-TOKEN-SYSTEM → fontSize.sm, fontWeight.medium

Card:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 3.1
  tokens:
    background: DESIGN-TOKEN-SYSTEM → colors.surface.default
    border: DESIGN-TOKEN-SYSTEM → colors.border.default
    shadow: DESIGN-TOKEN-SYSTEM → shadows.md
    radius: DESIGN-TOKEN-SYSTEM → borderRadius.lg
    padding: DESIGN-TOKEN-SYSTEM → spacing.6

Input:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 4.1
  tokens:
    border: DESIGN-TOKEN-SYSTEM → colors.border.default/emphasis
    focus: DESIGN-TOKEN-SYSTEM → colors.border.brand
    error: DESIGN-TOKEN-SYSTEM → colors.feedback.error
    radius: DESIGN-TOKEN-SYSTEM → borderRadius.md
    
Modal:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 6.1
  tokens:
    backdrop: rgba(0,0,0,0.5)
    background: DESIGN-TOKEN-SYSTEM → colors.surface.default
    shadow: DESIGN-TOKEN-SYSTEM → shadows.xl
    z-index: DESIGN-TOKEN-SYSTEM → zIndex.modal
    radius: DESIGN-TOKEN-SYSTEM → borderRadius.xl

Table:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 7.1
  tokens:
    header-bg: DESIGN-TOKEN-SYSTEM → colors.surface.subtle
    border: DESIGN-TOKEN-SYSTEM → colors.border.default
    hover: DESIGN-TOKEN-SYSTEM → colors.interactive.hover
    
Badge:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 3.4
  tokens:
    variants: DESIGN-TOKEN-SYSTEM → colors.feedback.*
    radius: DESIGN-TOKEN-SYSTEM → borderRadius.full
    typography: DESIGN-TOKEN-SYSTEM → fontSize.xs

Avatar:
  pattern: UI-PATTERN-PRIMITIVI → Sezione 3.3
  tokens:
    sizes: w-6, w-8, w-10, w-12, w-16
    fallback-bg: DESIGN-TOKEN-SYSTEM → colors.surface.muted
    status-colors: DESIGN-TOKEN-SYSTEM → colors.feedback.*

§ 12.3 MAPPING: CATEGORIA PIATTAFORMA → CATALOGHI PRIORITARI

yaml
E-Commerce:
  priorità_1:
    - DESIGN-REFERENCE → Stile Airbnb/Shopify, Layout E-commerce
    - UI-PATTERN-PRIMITIVI → Categoria E-Commerce
  priorità_2:
    - REQUISITI-FUNZIONALI → Modulo Commerce
    - API → Endpoints products, cart, orders
    - DATA-MODEL → Entità Product, Cart, Order
  priorità_3:
    - CODICE → React patterns, Zod schemas
    - AWS-DETERMINISTICO → Serverless e-commerce

SaaS:
  priorità_1:
    - DESIGN-REFERENCE → Stile Stripe/Linear, Layout Dashboard
    - UI-PATTERN-PRIMITIVI → Categoria SaaS + Dashboard
  priorità_2:
    - REQUISITI-FUNZIONALI → Multi-tenant, Subscription
    - API → Auth, Tenants, Users
    - DATA-MODEL → Tenant, User, Subscription
  priorità_3:
    - CODICE → Auth patterns, API patterns
    - AWS-DETERMINISTICO → Cognito, Lambda

Healthcare:
  priorità_1:
    - REQUISITI-ARCHITETTURA → Compliance HIPAA
    - UI-PATTERN-PRIMITIVI → Categoria Healthcare
  priorità_2:
    - DESIGN-REFERENCE → Stile Notion (accessibility)
    - DATA-MODEL → Patient, Provider, Records
    - API → Secure endpoints, Audit logging
  priorità_3:
    - AWS-DETERMINISTICO → HIPAA-eligible services
    - CODICE → Encryption, Audit trails

FinTech:
  priorità_1:
    - REQUISITI-ARCHITETTURA → Compliance PCI-DSS
    - UI-PATTERN-PRIMITIVI → Categoria FinTech
  priorità_2:
    - DESIGN-REFERENCE → Stile Stripe/Vercel
    - DATA-MODEL → Account, Transaction, Card
    - API → Secure transactions, KYC
  priorità_3:
    - AWS-DETERMINISTICO → PCI-compliant architecture
    - CODICE → Security patterns

Social/Community:
  priorità_1:
    - DESIGN-REFERENCE → Stile Slack, Layout Chat/Feed
    - UI-PATTERN-PRIMITIVI → Categoria Social
  priorità_2:
    - REQUISITI-FUNZIONALI → Content, Social modules
    - DATA-MODEL → User, Post, Message
    - API → Feed, Messaging
  priorità_3:
    - CODICE → Real-time patterns
    - AWS-DETERMINISTICO → WebSocket, notifications

§ 12.4 CHECKLIST RAPIDA PRE-SVILUPPO

markdown
## CHECKLIST: Prima di scrivere codice

### Discovery ✓
- [ ] Categoria identificata
- [ ] Requisiti core definiti (max 5)
- [ ] Target users definiti
- [ ] Edge cases discussi

### Design ✓
- [ ] Stile selezionato (DESIGN-REFERENCE)
- [ ] Layout pagine identificati (DESIGN-REFERENCE)
- [ ] Tokens customizzati se necessario (DESIGN-TOKEN-SYSTEM)

### Dati ✓
- [ ] Entità principali identificate (DATA-MODEL)
- [ ] Relazioni definite
- [ ] Schema Zod preparato (CODICE)

### API ✓
- [ ] Endpoints necessari listati (API)
- [ ] Auth requirements definiti
- [ ] Error handling pianificato

### UI ✓
- [ ] Componenti primitivi identificati (UI-PATTERN-PRIMITIVI)
- [ ] Componenti categoria-specifici identificati
- [ ] Responsive requirements chiari

### Infra ✓
- [ ] Stack tecnologico deciso
- [ ] Hosting identificato
- [ ] Database scelto
- [ ] Budget confermato (free tier?)

### GO! ✓
- [ ] Tutto confermato con utente
- [ ] Struttura progetto chiara
- [ ] Prima fase definita

# ============================================================================
§ SEZIONE 13: SNIPPET CODICE PRONTI ALL'USO
# ============================================================================

§ 13.1 SETUP PROGETTO NEXT.JS

bash
# Comando setup completo
pnpm create next-app@latest my-project --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"

cd my-project

# Dipendenze comuni
pnpm add zod @tanstack/react-query lucide-react clsx tailwind-merge
pnpm add -D @types/node

# Se auth con Supabase
pnpm add @supabase/supabase-js @supabase/auth-helpers-nextjs

# Se payments con Stripe
pnpm add @stripe/stripe-js stripe

§ 13.2 UTILITY CN (CLASS NAMES)

typescript
// lib/utils.ts
import { clsx, type ClassValue } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

§ 13.3 ZOD SCHEMA TEMPLATE

typescript
// schemas/product.ts
import { z } from 'zod';

export const ProductSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(200),
  description: z.string().max(2000).optional(),
  price: z.number().positive(),
  compareAtPrice: z.number().positive().optional(),
  category: z.string(),
  image: z.string().url(),
  stock: z.number().int().min(0).default(0),
  variants: z.array(z.object({
    name: z.string(),
    value: z.string(),
    priceModifier: z.number().default(0),
  })).optional(),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
});

export type Product = z.infer<typeof ProductSchema>;

// Validation helper
export function validateProduct(data: unknown): Product {
  return ProductSchema.parse(data);
}

§ 13.4 REACT QUERY SETUP

typescript
// lib/query-client.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000, // 1 minute
      retry: 1,
    },
  },
});

// hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { Product } from '@/schemas/product';

export function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: async (): Promise<Product[]> => {
      const res = await fetch('/api/products');
      if (!res.ok) throw new Error('Failed to fetch products');
      return res.json();
    },
  });
}

export function useProduct(id: string) {
  return useQuery({
    queryKey: ['products', id],
    queryFn: async (): Promise<Product> => {
      const res = await fetch(`/api/products/${id}`);
      if (!res.ok) throw new Error('Failed to fetch product');
      return res.json();
    },
    enabled: !!id,
  });
}

§ 13.5 LOCAL STORAGE CART HOOK

typescript
// hooks/useCart.ts
import { useState, useEffect, useCallback } from 'react';

interface CartItem {
  productId: string;
  name: string;
  price: number;
  quantity: number;
  image: string;
  variant?: string;
}

interface Cart {
  items: CartItem[];
  total: number;
}

const CART_KEY = 'shopping_cart';

export function useCart() {
  const [cart, setCart] = useState<Cart>({ items: [], total: 0 });
  const [isLoaded, setIsLoaded] = useState(false);

  // Load cart from localStorage
  useEffect(() => {
    const stored = localStorage.getItem(CART_KEY);
    if (stored) {
      setCart(JSON.parse(stored));
    }
    setIsLoaded(true);
  }, []);

  // Save cart to localStorage
  useEffect(() => {
    if (isLoaded) {
      localStorage.setItem(CART_KEY, JSON.stringify(cart));
    }
  }, [cart, isLoaded]);

  const addItem = useCallback((item: Omit<CartItem, 'quantity'>) => {
    setCart(prev => {
      const existing = prev.items.find(i => 
        i.productId === item.productId && i.variant === item.variant
      );
      
      let newItems: CartItem[];
      if (existing) {
        newItems = prev.items.map(i =>
          i.productId === item.productId && i.variant === item.variant
            ? { ...i, quantity: i.quantity + 1 }
            : i
        );
      } else {
        newItems = [...prev.items, { ...item, quantity: 1 }];
      }
      
      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);
      return { items: newItems, total };
    });
  }, []);

  const removeItem = useCallback((productId: string, variant?: string) => {
    setCart(prev => {
      const newItems = prev.items.filter(i => 
        !(i.productId === productId && i.variant === variant)
      );
      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);
      return { items: newItems, total };
    });
  }, []);

  const updateQuantity = useCallback((productId: string, quantity: number, variant?: string) => {
    if (quantity < 1) return;
    setCart(prev => {
      const newItems = prev.items.map(i =>
        i.productId === productId && i.variant === variant
          ? { ...i, quantity }
          : i
      );
      const total = newItems.reduce((sum, i) => sum + i.price * i.quantity, 0);
      return { items: newItems, total };
    });
  }, []);

  const clearCart = useCallback(() => {
    setCart({ items: [], total: 0 });
  }, []);

  return {
    cart,
    isLoaded,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    itemCount: cart.items.reduce((sum, i) => sum + i.quantity, 0),
  };
}

§ 13.6 API ROUTE TEMPLATE (NEXT.JS)

typescript
// app/api/products/route.ts
import { NextResponse } from 'next/server';
import { z } from 'zod';
import { ProductSchema } from '@/schemas/product';

// GET /api/products
export async function GET(request: Request) {
  try {
    const { searchParams } = new URL(request.url);
    const category = searchParams.get('category');
    
    // Fetch from database (mock for now)
    let products = await getProducts();
    
    if (category) {
      products = products.filter(p => p.category === category);
    }
    
    return NextResponse.json(products);
  } catch (error) {
    console.error('GET /api/products error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch products' },
      { status: 500 }
    );
  }
}

// POST /api/products
export async function POST(request: Request) {
  try {
    const body = await request.json();
    
    // Validate with Zod
    const validated = ProductSchema.omit({ 
      id: true, 
      createdAt: true, 
      updatedAt: true 
    }).parse(body);
    
    // Create product
    const product = await createProduct(validated);
    
    return NextResponse.json(product, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    
    console.error('POST /api/products error:', error);
    return NextResponse.json(
      { error: 'Failed to create product' },
      { status: 500 }
    );
  }
}

§ 13.7 TAILWIND CONFIG CON DESIGN TOKENS

javascript
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'class',
  content: ['./src/**/*.{js,ts,jsx,tsx,mdx}'],
  theme: {
    extend: {
      colors: {
        // Brand
        brand: {
          DEFAULT: 'var(--color-primary)',
          hover: 'var(--color-primary-hover)',
          light: 'var(--color-primary-light)',
        },
        // Surfaces
        surface: {
          default: 'var(--color-surface-default)',
          subtle: 'var(--color-surface-subtle)',
          muted: 'var(--color-surface-muted)',
          inverse: 'var(--color-surface-inverse)',
        },
        // Text
        text: {
          default: 'var(--color-text-default)',
          muted: 'var(--color-text-muted)',
          subtle: 'var(--color-text-subtle)',
          inverse: 'var(--color-text-inverse)',
        },
        // Borders
        border: {
          default: 'var(--color-border-default)',
          muted: 'var(--color-border-muted)',
          emphasis: 'var(--color-border-emphasis)',
        },
        // Feedback
        feedback: {
          success: 'var(--color-feedback-success)',
          error: 'var(--color-feedback-error)',
          warning: 'var(--color-feedback-warning)',
          info: 'var(--color-feedback-info)',
        },
      },
      borderRadius: {
        DEFAULT: 'var(--radius-md)',
        sm: 'var(--radius-sm)',
        md: 'var(--radius-md)',
        lg: 'var(--radius-lg)',
        xl: 'var(--radius-xl)',
      },
      boxShadow: {
        sm: 'var(--shadow-sm)',
        DEFAULT: 'var(--shadow-md)',
        md: 'var(--shadow-md)',
        lg: 'var(--shadow-lg)',
        xl: 'var(--shadow-xl)',
      },
      transitionDuration: {
        fast: '150ms',
        normal: '200ms',
        slow: '300ms',
      },
    },
  },
  plugins: [],
};

# ============================================================================
§ FINE SEZIONI AGGIUNTIVE
# ============================================================================

"""
Sezioni aggiunte:
- 11: Esempi pratici completi (PetShop E-commerce, TaskFlow SaaS)
- 12: Cross-reference tra cataloghi
- 13: Snippet codice pronti all'uso

Il catalogo INTERFACCIA-PROMPT-USABILITA è ora completo con:
- Template prompt
- Workflow operativi
- Discovery framework
- Review/Fix workflow
- Esempi pratici
- Cross-reference
- Snippet codice

Versione finale: 1.0.0
"""
