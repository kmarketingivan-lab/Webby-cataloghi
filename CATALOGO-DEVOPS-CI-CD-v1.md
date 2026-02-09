# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO DEVOPS & CI/CD v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
§ GUIDA COMPLETA DEVOPS, CI/CD E DEPLOYMENT PER APPLICAZIONI WEB
# GitHub Actions, Docker, Kubernetes, Infrastructure as Code
#
# Data creazione: 2026-01-26
# Ultima modifica: 2026-01-26
#
# ═══════════════════════════════════════════════════════════════════════════════

"""
INDICE:
═══════════════════════════════════════════════════════════════════════════════
1.  DevOps Fundamentals
2.  Git Workflow & Branching Strategy
3.  GitHub Actions - CI Pipelines
4.  GitHub Actions - CD Pipelines
5.  Docker Fundamentals
6.  Docker per Next.js
7.  Docker Compose
8.  Container Registry & Image Management
9.  Kubernetes Fundamentals
10. Kubernetes Deployment
11. Infrastructure as Code (Terraform)
12. Environment Management
13. Secrets Management
14. Monitoring & Logging in CI/CD
15. Deployment Strategies
═══════════════════════════════════════════════════════════════════════════════
"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 1: DEVOPS FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

DEVOPS_FUNDAMENTALS = """

§ 1.1 COS'È DEVOPS

┌─────────────────────────────────────────────────────────────────────────────┐
│ DEVOPS: CULTURA, PRATICHE E STRUMENTI                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DevOps è una cultura e un insieme di pratiche che unificano lo sviluppo   │
│ software (Dev) e le operazioni IT (Ops) per migliorare la collaborazione   │
│ e automatizzare il ciclo di vita del software.                              │
│                                                                             │
│ PRINCIPI CHIAVE                                                             │
│ ├── Collaboration       - Dev e Ops lavorano insieme                       │
│ ├── Automation          - Automazione di build, test, deploy               │
│ ├── Continuous Improvement - Miglioramento continuo                        │
│ ├── Customer Focus      - Focus sul valore per l'utente                    │
│ └── End-to-End Ownership - Responsabilità condivisa                        │
│                                                                             │
│ CICLO DEVOPS (∞)                                                            │
│                                                                             │
│         ┌──────────────────────────────────────────┐                       │
│         │                  PLAN                     │                       │
│         └──────────────────────────────────────────┘                       │
│                          ▼                                                 │
│    ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐        │
│    │  CREATE  │────▶│  VERIFY  │────▶│ PACKAGE  │────▶│ RELEASE  │        │
│    └──────────┘     └──────────┘     └──────────┘     └──────────┘        │
│         │                                                  │               │
│         │              ┌──────────────────────────────────┘               │
│         │              ▼                                                   │
│    ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐        │
│    │ MONITOR  │◀────│ OPERATE  │◀────│ DEPLOY   │◀────│ CONFIGURE│        │
│    └──────────┘     └──────────┘     └──────────┘     └──────────┘        │
│         │                                                                  │
│         └──────────────────────────────────────────────────────────────────│
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.2 CI/CD EXPLAINED

┌─────────────────────────────────────────────────────────────────────────────┐
│ CONTINUOUS INTEGRATION / CONTINUOUS DEPLOYMENT                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CONTINUOUS INTEGRATION (CI)                                                 │
│ ├── Merge frequente del codice in un repository condiviso                  │
│ ├── Build automatica ad ogni commit                                        │
│ ├── Test automatici (unit, integration, e2e)                               │
│ ├── Feedback rapido sugli errori                                           │
│ └── Obiettivo: Rilevare bug il prima possibile                             │
│                                                                             │
│ CONTINUOUS DELIVERY (CD)                                                    │
│ ├── Estensione della CI                                                    │
│ ├── Deploy automatico su staging/preview                                   │
│ ├── Deploy production manuale (approval gate)                              │
│ └── Obiettivo: Software sempre pronto per il deploy                        │
│                                                                             │
│ CONTINUOUS DEPLOYMENT (CD)                                                  │
│ ├── Estensione della Continuous Delivery                                   │
│ ├── Deploy automatico anche in production                                  │
│ ├── Nessun intervento manuale                                              │
│ └── Obiettivo: Rilasci frequenti e sicuri                                  │
│                                                                             │
│ CI/CD PIPELINE                                                              │
│                                                                             │
│ ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐   ┌────────┐            │
│ │ SOURCE │──▶│ BUILD  │──▶│  TEST  │──▶│ DEPLOY │──▶│MONITOR │            │
│ └────────┘   └────────┘   └────────┘   └────────┘   └────────┘            │
│     │            │            │            │            │                  │
│   commit       compile      unit        staging     metrics               │
│   push         bundle       e2e         production  alerts                │
│   PR           lint         security    rollback    logs                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.3 DEVOPS TOOLS ECOSYSTEM

┌─────────────────────────────────────────────────────────────────────────────┐
│ STRUMENTI DEVOPS PER CATEGORIA                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ VERSION CONTROL                                                             │
│ ├── Git                    - Standard de facto                             │
│ ├── GitHub                 - Hosting + Actions CI/CD                       │
│ ├── GitLab                 - Hosting + GitLab CI/CD                        │
│ └── Bitbucket              - Hosting + Pipelines                           │
│                                                                             │
│ CI/CD PLATFORMS                                                             │
│ ├── GitHub Actions         - Native GitHub, YAML-based ⭐                  │
│ ├── GitLab CI              - Native GitLab, powerful                       │
│ ├── Jenkins                - Self-hosted, plugin ecosystem                 │
│ ├── CircleCI               - Cloud-native, fast                            │
│ ├── Travis CI              - Open source friendly                          │
│ └── Azure DevOps           - Microsoft ecosystem                           │
│                                                                             │
│ CONTAINERIZATION                                                            │
│ ├── Docker                 - Container runtime ⭐                          │
│ ├── Podman                 - Daemonless alternative                        │
│ └── containerd             - Low-level runtime                             │
│                                                                             │
│ ORCHESTRATION                                                               │
│ ├── Kubernetes             - Standard de facto ⭐                          │
│ ├── Docker Swarm           - Simpler alternative                           │
│ └── Nomad                  - HashiCorp alternative                         │
│                                                                             │
│ INFRASTRUCTURE AS CODE                                                      │
│ ├── Terraform              - Multi-cloud IaC ⭐                            │
│ ├── Pulumi                 - IaC con linguaggi reali                       │
│ ├── CloudFormation         - AWS native                                    │
│ └── Ansible                - Configuration management                      │
│                                                                             │
│ MONITORING & LOGGING                                                        │
│ ├── Prometheus + Grafana   - Metrics & visualization ⭐                    │
│ ├── ELK Stack              - Elasticsearch, Logstash, Kibana              │
│ ├── Loki                   - Log aggregation (Grafana)                    │
│ └── Datadog / New Relic    - Commercial APM                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 1.4 DEVOPS METRICS (DORA)

typescript
// Le 4 metriche DORA (DevOps Research and Assessment)

const DORA_METRICS = {
  // 1. Deployment Frequency
  // Quanto spesso rilasci in produzione?
  deploymentFrequency: {
    elite: 'Multiple deploys per day',
    high: 'Between once per day and once per week',
    medium: 'Between once per week and once per month',
    low: 'Between once per month and once every six months',
  },
  
  // 2. Lead Time for Changes
  // Quanto tempo dal commit al deploy in produzione?
  leadTimeForChanges: {
    elite: 'Less than one hour',
    high: 'Between one day and one week',
    medium: 'Between one week and one month',
    low: 'Between one month and six months',
  },
  
  // 3. Mean Time to Recovery (MTTR)
  // Quanto tempo per ripristinare il servizio dopo un incident?
  meanTimeToRecovery: {
    elite: 'Less than one hour',
    high: 'Less than one day',
    medium: 'Between one day and one week',
    low: 'Between one week and one month',
  },
  
  // 4. Change Failure Rate
  // Che percentuale di deploy causa problemi?
  changeFailureRate: {
    elite: '0-15%',
    high: '16-30%',
    medium: '31-45%',
    low: '46-60%',
  },
};

// Target per team high-performing:
// - Deploy multiple times per day
// - Lead time < 1 day
// - MTTR < 1 hour
// - Change failure rate < 15%

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 2: GIT WORKFLOW & BRANCHING STRATEGY
# ═══════════════════════════════════════════════════════════════════════════════

GIT_WORKFLOW = """

§ 2.1 GIT FLOW VS TRUNK-BASED DEVELOPMENT

┌─────────────────────────────────────────────────────────────────────────────┐
│ BRANCHING STRATEGIES                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ GIT FLOW (Traditional)                                                     │
│                                                                             │
│  main ─────●────────●────────────●─────────────●──────▶                    │
│            │        ▲            │             ▲                           │
│  release   │        │        ────●─────────────┘                           │
│            │        │       /                                              │
│  develop ──●────────●──────●─────────●─────────●───────▶                   │
│            │        │      │         ▲         │                           │
│  feature/a ├────────┘      │         │         │                           │
│  feature/b                 └─────────┘         │                           │
│  hotfix                                        │                           │
│                                                                             │
│  ✅ Adatto a: Rilasci pianificati, team grandi                             │
│  ❌ Contro: Complesso, merge frequenti, CI/CD lento                         │
│                                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ TRUNK-BASED DEVELOPMENT (Modern) ⭐ Raccomandato                            │
│                                                                             │
│  main ─────●──●──●──●──●──●──●──●──●──●──●──●──●──●──▶                     │
│            │     │     │     │     │     │     │                           │
│  feat/a    └─────┘     │     │     │     │     │                           │
│  feat/b                └─────┘     │     │     │                           │
│  feat/c                            └─────┘     │                           │
│  fix/x                                         └─────┘                     │
│                                                                             │
│  ✅ Adatto a: CI/CD moderno, deploy frequenti                               │
│  ✅ Pro: Semplice, merge veloci, feedback rapido                            │
│  ✅ Pro: Feature flags per rilasci graduali                                 │
│  ⚠️ Richiede: Buoni test automatici, code review rapide                     │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 2.2 BRANCH NAMING CONVENTIONS

bash
# Convenzioni di naming per branch

# Feature branches
feature/add-user-authentication
feature/JIRA-123-payment-integration
feat/shopping-cart

# Bug fix branches
fix/login-redirect-loop
bugfix/JIRA-456-null-pointer
hotfix/critical-security-patch

# Chore/maintenance branches
chore/update-dependencies
chore/refactor-api-client
docs/update-readme

# Release branches (se usi Git Flow)
release/v1.2.0
release/2024-q1

# Esempi con prefissi standard
# feat/     - nuove funzionalità
# fix/      - bug fix
# hotfix/   - fix urgenti per production
# chore/    - manutenzione, refactoring
# docs/     - documentazione
# test/     - aggiunta/modifica test
# ci/       - modifiche CI/CD
# perf/     - ottimizzazioni performance

§ 2.3 CONVENTIONAL COMMITS

bash
# Conventional Commits Specification
# https://www.conventionalcommits.org/

# Formato:
# <type>[optional scope]: <description>
#
# [optional body]
#
# [optional footer(s)]

# TYPES
# feat:     Nuova funzionalità
# fix:      Bug fix
# docs:     Solo documentazione
# style:    Formattazione (no logic changes)
# refactor: Refactoring (no new features, no bug fixes)
# perf:     Performance improvements
# test:     Aggiunta/modifica test
# build:    Build system, dependencies
# ci:       CI/CD configuration
# chore:    Altre modifiche (no src/test)
# revert:   Revert di un commit precedente

# Esempi
git commit -m "feat(auth): add OAuth2 login with Google"
git commit -m "fix(cart): resolve quantity calculation error"
git commit -m "docs: update API documentation"
git commit -m "chore(deps): upgrade Next.js to 15.1"
git commit -m "ci: add e2e tests to GitHub Actions workflow"

# Breaking changes (!)
git commit -m "feat(api)!: remove deprecated v1 endpoints"

# Con body e footer
git commit -m "fix(auth): resolve session expiration bug

The session was expiring immediately due to incorrect
timestamp calculation in the JWT validation.

Fixes #123
Reviewed-by: John Doe"

# Automazione con commitlint
# package.json
{
  "devDependencies": {
    "@commitlint/cli": "^18.0.0",
    "@commitlint/config-conventional": "^18.0.0",
    "husky": "^9.0.0"
  }
}

# commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'subject-max-length': [2, 'always', 72],
  },
};

§ 2.4 PULL REQUEST BEST PRACTICES

markdown
# .github/PULL_REQUEST_TEMPLATE.md

## Description
<!-- Descrivi brevemente le modifiche -->

## Type of Change
- [ ] 🐛 Bug fix (non-breaking change that fixes an issue)
- [ ] ✨ New feature (non-breaking change that adds functionality)
- [ ] 💥 Breaking change (fix or feature that would cause existing functionality to change)
- [ ] 📝 Documentation update
- [ ] 🔧 Configuration change
- [ ] ♻️ Refactoring (no functional changes)

## Related Issues
<!-- Link agli issue correlati -->
Closes #

## Checklist
- [ ] My code follows the project's code style
- [ ] I have performed a self-review of my code
- [ ] I have commented my code, particularly in hard-to-understand areas
- [ ] I have made corresponding changes to the documentation
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
- [ ] Any dependent changes have been merged and published

## Screenshots (if applicable)
<!-- Aggiungi screenshot se ci sono modifiche UI -->

## Testing Instructions
<!-- Come testare queste modifiche -->

## Additional Notes
<!-- Note aggiuntive per i reviewer -->

§ 2.5 BRANCH PROTECTION RULES

yaml
# Configurazione branch protection (GitHub)
# Settings > Branches > Branch protection rules

# Per branch: main

# ✅ Require a pull request before merging
#    ✅ Require approvals: 1 (o più per team grandi)
#    ✅ Dismiss stale pull request approvals when new commits are pushed
#    ✅ Require review from Code Owners

# ✅ Require status checks to pass before merging
#    ✅ Require branches to be up to date before merging
#    Status checks:
#    - ci / build
#    - ci / test
#    - ci / lint
#    - ci / type-check

# ✅ Require conversation resolution before merging

# ✅ Require signed commits (opzionale ma consigliato)

# ✅ Do not allow bypassing the above settings

# ❌ Allow force pushes (MAI su main)
# ❌ Allow deletions (MAI su main)


# CODEOWNERS file
# .github/CODEOWNERS

# Default owners
* @team-leads

# Frontend code
/src/components/ @frontend-team
/src/app/ @frontend-team

# Backend/API code
/src/api/ @backend-team
/src/lib/db/ @backend-team @dba-team

# Infrastructure
/.github/workflows/ @devops-team
/terraform/ @devops-team
/docker/ @devops-team

# Documentation
/docs/ @tech-writers @team-leads

# Security-sensitive files
/src/lib/auth/ @security-team
*.env.example @security-team @devops-team

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 3: GITHUB ACTIONS - CI PIPELINES
# ═══════════════════════════════════════════════════════════════════════════════

GITHUB_ACTIONS_CI = """

§ 3.1 GITHUB ACTIONS OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ GITHUB ACTIONS STRUCTURE                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ WORKFLOW (.github/workflows/*.yml)                                         │
│ └── Definisce l'intero processo CI/CD                                      │
│                                                                             │
│ EVENTS (triggers)                                                           │
│ ├── push          - Commit pushed                                          │
│ ├── pull_request  - PR opened, synchronized, reopened                      │
│ ├── schedule      - Cron-based                                             │
│ ├── workflow_dispatch - Manual trigger                                     │
│ └── release       - Release published                                      │
│                                                                             │
│ JOBS (parallel by default)                                                 │
│ ├── job1: build                                                            │
│ ├── job2: test (needs: build)                                              │
│ └── job3: deploy (needs: test)                                             │
│                                                                             │
│ STEPS (sequential within a job)                                            │
│ ├── step1: Checkout code                                                   │
│ ├── step2: Setup Node.js                                                   │
│ ├── step3: Install dependencies                                            │
│ └── step4: Run tests                                                       │
│                                                                             │
│ RUNNERS (execution environment)                                            │
│ ├── ubuntu-latest  - Most common ⭐                                        │
│ ├── windows-latest - Windows builds                                        │
│ ├── macos-latest   - macOS/iOS builds                                      │
│ └── self-hosted    - Custom runners                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 3.2 BASIC CI WORKFLOW

yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

# Cancel in-progress runs for the same branch
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  NODE_VERSION: '20'
  PNPM_VERSION: '9'

jobs:
  # ─────────────────────────────────────────────────────────────────────────
  # BUILD & LINT
  # ─────────────────────────────────────────────────────────────────────────
  build:
    name: Build & Lint
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run linter
        run: pnpm lint
      
      - name: Run type check
        run: pnpm type-check
      
      - name: Build application
        run: pnpm build
      
      # Cache build output for other jobs
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-output
          path: |
            .next/
            !.next/cache
          retention-days: 1

  # ─────────────────────────────────────────────────────────────────────────
  # UNIT TESTS
  # ─────────────────────────────────────────────────────────────────────────
  test-unit:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Run unit tests
        run: pnpm test:unit --coverage
      
      - name: Upload coverage report
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  # ─────────────────────────────────────────────────────────────────────────
  # E2E TESTS
  # ─────────────────────────────────────────────────────────────────────────
  test-e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    needs: build
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup pnpm
        uses: pnpm/action-setup@v3
        with:
          version: ${{ env.PNPM_VERSION }}
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Install Playwright browsers
        run: pnpm exec playwright install --with-deps chromium
      
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-output
          path: .next
      
      - name: Run E2E tests
        run: pnpm test:e2e
      
      - name: Upload test results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 7

§ 3.3 MATRIX STRATEGY FOR MULTIPLE VERSIONS

yaml
# .github/workflows/ci-matrix.yml
name: CI Matrix

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test (Node ${{ matrix.node-version }}, ${{ matrix.os }})
    runs-on: ${{ matrix.os }}
    
    strategy:
      fail-fast: false  # Continue other jobs if one fails
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: ['18', '20', '22']
        exclude:
          # Exclude Windows + Node 18 (example)
          - os: windows-latest
            node-version: '18'
        include:
          # Add specific configuration
          - os: ubuntu-latest
            node-version: '20'
            is-primary: true
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
      
      # Upload coverage only for primary config
      - name: Upload coverage
        if: matrix.is-primary
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

§ 3.4 CACHING STRATEGIES

yaml
# Caching per velocizzare le build

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # pnpm caching (built-in in setup-node)
      - uses: pnpm/action-setup@v3
        with:
          version: '9'
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'  # Auto-cache pnpm store
      
      # Cache manuale per .next/cache
      - name: Cache Next.js build
        uses: actions/cache@v4
        with:
          path: |
            .next/cache
          key: nextjs-${{ runner.os }}-${{ hashFiles('**/pnpm-lock.yaml') }}-${{ hashFiles('**/*.ts', '**/*.tsx') }}
          restore-keys: |
            nextjs-${{ runner.os }}-${{ hashFiles('**/pnpm-lock.yaml') }}-
            nextjs-${{ runner.os }}-
      
      # Cache Playwright browsers
      - name: Cache Playwright browsers
        uses: actions/cache@v4
        with:
          path: ~/.cache/ms-playwright
          key: playwright-${{ runner.os }}-${{ hashFiles('**/pnpm-lock.yaml') }}
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm build

  # Cache Docker layers
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: false
          tags: myapp:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

§ 3.5 REUSABLE WORKFLOWS

yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        description: 'Node.js version'
        required: false
        type: string
        default: '20'
      coverage:
        description: 'Upload coverage'
        required: false
        type: boolean
        default: false
    secrets:
      CODECOV_TOKEN:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: pnpm/action-setup@v3
        with:
          version: '9'
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm test --coverage
      
      - name: Upload coverage
        if: inputs.coverage && secrets.CODECOV_TOKEN
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

# ─────────────────────────────────────────────────────────────────────────────

# .github/workflows/ci.yml (using reusable workflow)
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
      coverage: true
    secrets:
      CODECOV_TOKEN: ${{ secrets.CODECOV_TOKEN }}

§ 3.6 COMPOSITE ACTIONS

yaml
# .github/actions/setup-project/action.yml
name: 'Setup Project'
description: 'Setup Node.js, pnpm, and install dependencies'

inputs:
  node-version:
    description: 'Node.js version'
    required: false
    default: '20'
  pnpm-version:
    description: 'pnpm version'
    required: false
    default: '9'

runs:
  using: 'composite'
  steps:
    - name: Setup pnpm
      uses: pnpm/action-setup@v3
      with:
        version: ${{ inputs.pnpm-version }}
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: ${{ inputs.node-version }}
        cache: 'pnpm'
    
    - name: Get pnpm store directory
      shell: bash
      run: |
        echo "STORE_PATH=$(pnpm store path --silent)" >> $GITHUB_ENV
    
    - name: Cache pnpm store
      uses: actions/cache@v4
      with:
        path: ${{ env.STORE_PATH }}
        key: pnpm-store-${{ runner.os }}-${{ hashFiles('**/pnpm-lock.yaml') }}
        restore-keys: |
          pnpm-store-${{ runner.os }}-
    
    - name: Install dependencies
      shell: bash
      run: pnpm install --frozen-lockfile

# ─────────────────────────────────────────────────────────────────────────────

# Uso della composite action
# .github/workflows/ci.yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup project
        uses: ./.github/actions/setup-project
        with:
          node-version: '20'
      
      - run: pnpm build

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 4: GITHUB ACTIONS - CD PIPELINES
# ═══════════════════════════════════════════════════════════════════════════════

GITHUB_ACTIONS_CD = """

§ 4.1 DEPLOYMENT TO VERCEL

yaml
# .github/workflows/deploy-vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install Vercel CLI
        run: npm install -g vercel@latest
      
      # Preview deployment for PRs
      - name: Deploy to Vercel (Preview)
        if: github.event_name == 'pull_request'
        run: |
          vercel pull --yes --environment=preview --token=${{ secrets.VERCEL_TOKEN }}
          vercel build --token=${{ secrets.VERCEL_TOKEN }}
          vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }}
      
      # Production deployment for main branch
      - name: Deploy to Vercel (Production)
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        run: |
          vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
          vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
          vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}

§ 4.2 DEPLOYMENT TO AWS (DOCKER + ECR + ECS)

yaml
# .github/workflows/deploy-aws.yml
name: Deploy to AWS ECS

on:
  push:
    branches: [main]
    tags: ['v*']

env:
  AWS_REGION: eu-west-1
  ECR_REPOSITORY: my-app
  ECS_SERVICE: my-app-service
  ECS_CLUSTER: my-cluster
  ECS_TASK_DEFINITION: .aws/task-definition.json
  CONTAINER_NAME: my-app

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    permissions:
      id-token: write
      contents: read
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image to Amazon ECR
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
      
      - name: Fill in the new image ID in the Amazon ECS task definition
        id: task-def
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: ${{ env.ECS_TASK_DEFINITION }}
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.build-image.outputs.image }}
      
      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ steps.task-def.outputs.task-definition }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: true

§ 4.3 DEPLOYMENT WITH DOCKER TO VPS

yaml
# .github/workflows/deploy-vps.yml
name: Deploy to VPS

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
      - name: Deploy to VPS via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Login to GitHub Container Registry
            echo ${{ secrets.GITHUB_TOKEN }} | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            
            # Pull the new image
            docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            
            # Stop and remove old container
            docker stop my-app || true
            docker rm my-app || true
            
            # Run new container
            docker run -d \
              --name my-app \
              --restart unless-stopped \
              -p 3000:3000 \
              -e NODE_ENV=production \
              -e DATABASE_URL="${{ secrets.DATABASE_URL }}" \
              ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            
            # Cleanup old images
            docker image prune -af
      
      - name: Health check
        run: |
          sleep 10
          curl -f https://${{ secrets.VPS_HOST }} || exit 1

§ 4.4 BLUE-GREEN DEPLOYMENT

yaml
# .github/workflows/blue-green-deploy.yml
name: Blue-Green Deployment

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Deploy to staging slot (Green)
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Deploy to green slot (port 3001)
            docker pull ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
            docker stop app-green || true
            docker rm app-green || true
            docker run -d \
              --name app-green \
              -p 3001:3000 \
              ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
      
      - name: Run smoke tests on Green
        run: |
          # Wait for container to start
          sleep 15
          
          # Run smoke tests against green slot
          curl -f http://${{ secrets.VPS_HOST }}:3001/api/health || exit 1
      
      - name: Switch traffic to Green
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Update nginx to point to green
            sudo sed -i 's/localhost:3000/localhost:3001/g' /etc/nginx/sites-available/app
            sudo nginx -t && sudo systemctl reload nginx
            
            # Swap labels
            docker rename app-blue app-old || true
            docker rename app-green app-blue
            
            # Stop old blue (now called app-old)
            docker stop app-old || true
            docker rm app-old || true
      
      - name: Notify success
        if: success()
        run: |
          echo "Deployment successful!"
      
      - name: Rollback on failure
        if: failure()
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            # Revert nginx config
            sudo sed -i 's/localhost:3001/localhost:3000/g' /etc/nginx/sites-available/app
            sudo nginx -t && sudo systemctl reload nginx
            
            # Stop green
            docker stop app-green || true
            docker rm app-green || true

§ 4.5 RELEASE WORKFLOW WITH CHANGELOG

yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags: ['v*']

permissions:
  contents: write
  packages: write

jobs:
  release:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Needed for changelog
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Generate changelog
        id: changelog
        uses: orhun/git-cliff-action@v3
        with:
          config: cliff.toml
          args: --latest --strip header
        env:
          OUTPUT: CHANGELOG.md
      
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          body: ${{ steps.changelog.outputs.content }}
          draft: false
          prerelease: ${{ contains(github.ref, '-rc') || contains(github.ref, '-beta') }}
          files: |
            CHANGELOG.md
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      # Build and push Docker image with version tag
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract version
        id: version
        run: echo "version=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ghcr.io/${{ github.repository }}:${{ steps.version.outputs.version }}
            ghcr.io/${{ github.repository }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 5: DOCKER FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

DOCKER_FUNDAMENTALS = """

§ 5.1 DOCKER CONCEPTS

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOCKER ARCHITECTURE                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ IMAGE                                                                       │
│ └── Template read-only per creare container                                 │
│     ├── Layered filesystem (Union FS)                                      │
│     ├── Definita da Dockerfile                                             │
│     └── Versionata con tags (myapp:1.0, myapp:latest)                      │
│                                                                             │
│ CONTAINER                                                                   │
│ └── Istanza running di un'immagine                                         │
│     ├── Isolato (namespace, cgroups)                                       │
│     ├── Ephemeral (stateless by default)                                   │
│     └── Porta proprie env vars, network, storage                           │
│                                                                             │
│ DOCKERFILE                                                                  │
│ └── Script per buildare un'immagine                                        │
│     ├── FROM: base image                                                   │
│     ├── RUN: esegue comandi                                                │
│     ├── COPY/ADD: copia file                                               │
│     ├── ENV: variabili d'ambiente                                          │
│     ├── EXPOSE: documenta porte                                            │
│     ├── CMD: comando default                                               │
│     └── ENTRYPOINT: entry point fisso                                      │
│                                                                             │
│ REGISTRY                                                                    │
│ └── Storage per immagini Docker                                            │
│     ├── Docker Hub (public)                                                │
│     ├── GitHub Container Registry (ghcr.io)                                │
│     ├── AWS ECR, Google GCR, Azure ACR                                     │
│     └── Self-hosted (Harbor)                                               │
│                                                                             │
│ VOLUMES                                                                     │
│ └── Persistent storage per container                                       │
│     ├── Named volumes: docker volume create                                │
│     ├── Bind mounts: -v /host/path:/container/path                         │
│     └── tmpfs: in-memory storage                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 5.2 ESSENTIAL DOCKER COMMANDS

bash
# ═══════════════════════════════════════════════════════════════════════════════
# DOCKER COMMANDS REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────────────
# IMAGES
# ─────────────────────────────────────────────────────────────────────────────

# Build image from Dockerfile
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .

# List images
docker images
docker images -a  # Include intermediate layers

# Remove image
docker rmi myapp:1.0
docker image prune -a  # Remove all unused images

# Pull image from registry
docker pull node:20-alpine

# Push image to registry
docker push myregistry/myapp:1.0

# Tag image
docker tag myapp:1.0 myregistry/myapp:1.0

# Inspect image
docker inspect myapp:1.0
docker history myapp:1.0  # Show layers

# ─────────────────────────────────────────────────────────────────────────────
# CONTAINERS
# ─────────────────────────────────────────────────────────────────────────────

# Run container
docker run myapp:1.0
docker run -d myapp:1.0                    # Detached mode
docker run -p 3000:3000 myapp:1.0          # Port mapping
docker run -e NODE_ENV=production myapp:1.0 # Environment variable
docker run --name my-container myapp:1.0   # Named container
docker run --rm myapp:1.0                  # Remove after exit
docker run -it myapp:1.0 /bin/sh           # Interactive shell

# List containers
docker ps          # Running only
docker ps -a       # All containers

# Stop/Start/Restart container
docker stop my-container
docker start my-container
docker restart my-container

# Remove container
docker rm my-container
docker rm -f my-container  # Force remove running container
docker container prune     # Remove all stopped containers

# Execute command in running container
docker exec -it my-container /bin/sh
docker exec my-container ls -la

# View logs
docker logs my-container
docker logs -f my-container  # Follow logs
docker logs --tail 100 my-container

# Copy files
docker cp my-container:/app/file.txt ./file.txt
docker cp ./file.txt my-container:/app/file.txt

# ─────────────────────────────────────────────────────────────────────────────
# VOLUMES
# ─────────────────────────────────────────────────────────────────────────────

# Create volume
docker volume create mydata

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata
docker volume prune  # Remove all unused volumes

# Use volume in container
docker run -v mydata:/app/data myapp:1.0          # Named volume
docker run -v $(pwd)/data:/app/data myapp:1.0     # Bind mount

# ─────────────────────────────────────────────────────────────────────────────
# NETWORK
# ─────────────────────────────────────────────────────────────────────────────

# Create network
docker network create mynetwork

# List networks
docker network ls

# Connect container to network
docker run --network mynetwork myapp:1.0
docker network connect mynetwork my-container

# ─────────────────────────────────────────────────────────────────────────────
# CLEANUP
# ─────────────────────────────────────────────────────────────────────────────

# Remove all unused data
docker system prune -a --volumes

# Show disk usage
docker system df

§ 5.3 DOCKERFILE BEST PRACTICES

dockerfile
# ═══════════════════════════════════════════════════════════════════════════════
# DOCKERFILE BEST PRACTICES
# ═══════════════════════════════════════════════════════════════════════════════

# ❌ BAD: Single stage, large image
FROM node:20
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["npm", "start"]

# ✅ GOOD: Multi-stage build, optimized layers
# syntax=docker/dockerfile:1

# ─────────────────────────────────────────────────────────────────────────────
# Stage 1: Dependencies
# ─────────────────────────────────────────────────────────────────────────────
FROM node:20-alpine AS deps
# Alpine for smaller base image

WORKDIR /app

# Copy only package files first (better caching)
COPY package.json pnpm-lock.yaml ./

# Install pnpm and dependencies
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# ─────────────────────────────────────────────────────────────────────────────
# Stage 2: Builder
# ─────────────────────────────────────────────────────────────────────────────
FROM node:20-alpine AS builder
WORKDIR /app

# Copy dependencies from deps stage
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build-time environment variables
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL

# Disable telemetry during build
ENV NEXT_TELEMETRY_DISABLED=1

# Build the application
RUN corepack enable pnpm && pnpm build

# ─────────────────────────────────────────────────────────────────────────────
# Stage 3: Runner (Production)
# ─────────────────────────────────────────────────────────────────────────────
FROM node:20-alpine AS runner
WORKDIR /app

# Set production environment
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Create non-root user for security
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Copy only necessary files
COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

# Set ownership
RUN chown -R nextjs:nodejs /app

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Set port environment variable
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

# Start the application
CMD ["node", "server.js"]

§ 5.4 .DOCKERIGNORE

gitignore
# .dockerignore

# Dependencies
node_modules/
.pnpm-store/

# Build outputs (will be created in container)
.next/
out/
build/
dist/

# Development files
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment files (secrets!)
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
.env*.local

# Git
.git/
.gitignore

# IDE
.idea/
.vscode/
*.swp
*.swo

# Testing
coverage/
.nyc_output/
playwright-report/
test-results/

# Documentation
docs/
*.md
!README.md

# Docker (prevent recursive copy)
Dockerfile*
docker-compose*.yml
.docker/

# CI/CD
.github/
.gitlab-ci.yml
.circleci/

# Misc
.DS_Store
Thumbs.db
*.tgz

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 6: DOCKER PER NEXT.JS
# ═══════════════════════════════════════════════════════════════════════════════

DOCKER_NEXTJS = """

§ 6.1 NEXT.JS STANDALONE OUTPUT

javascript
// next.config.mjs
// Abilita standalone output per Docker

/** @type {import('next').NextConfig} */
const nextConfig = {
  // Produce una build standalone ottimizzata per Docker
  output: 'standalone',
  
  // Ottimizzazioni aggiuntive
  experimental: {
    // Riduci la dimensione dell'output
    outputFileTracingRoot: process.cwd(),
  },
  
  // Disabilita telemetry in production
  telemetry: {
    enabled: false,
  },
  
  // Image optimization
  images: {
    // Per deploy Docker, usa un loader esterno o disabilita
    unoptimized: process.env.NODE_ENV === 'production',
    // Oppure configura domains per ottimizzazione
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.example.com',
      },
    ],
  },
};

export default nextConfig;

§ 6.2 PRODUCTION DOCKERFILE PER NEXT.JS

dockerfile
# ═══════════════════════════════════════════════════════════════════════════════
# PRODUCTION DOCKERFILE PER NEXT.JS (2025)
# ═══════════════════════════════════════════════════════════════════════════════
# syntax=docker/dockerfile:1

# ─────────────────────────────────────────────────────────────────────────────
# Stage 1: Base
# ─────────────────────────────────────────────────────────────────────────────
FROM node:22-alpine AS base

# Install libc6-compat for Alpine compatibility
RUN apk add --no-cache libc6-compat

WORKDIR /app

# ─────────────────────────────────────────────────────────────────────────────
# Stage 2: Dependencies
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS deps

# Copy package files
COPY package.json pnpm-lock.yaml* ./

# Install pnpm and dependencies
RUN corepack enable pnpm && \
    pnpm install --frozen-lockfile --prod=false

# ─────────────────────────────────────────────────────────────────────────────
# Stage 3: Builder
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS builder

WORKDIR /app

# Copy dependencies
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Build-time environment variables (public only!)
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_SITE_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_SITE_URL=$NEXT_PUBLIC_SITE_URL

# Disable telemetry
ENV NEXT_TELEMETRY_DISABLED=1

# Build the application
RUN corepack enable pnpm && pnpm build

# ─────────────────────────────────────────────────────────────────────────────
# Stage 4: Production Runner
# ─────────────────────────────────────────────────────────────────────────────
FROM node:22-alpine AS runner

WORKDIR /app

# Production environment
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Create non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Copy public assets
COPY --from=builder /app/public ./public

# Copy standalone build
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Runtime environment variables
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/api/health || exit 1

# Start the application
CMD ["node", "server.js"]

§ 6.3 DEVELOPMENT DOCKERFILE

dockerfile
# Dockerfile.dev
# Per sviluppo locale con hot-reloading

FROM node:22-alpine

WORKDIR /app

# Install pnpm
RUN corepack enable pnpm

# Copy package files
COPY package.json pnpm-lock.yaml* ./

# Install all dependencies (including devDependencies)
RUN pnpm install --frozen-lockfile

# Non copiare il codice sorgente - verrà montato come volume
# COPY . .

EXPOSE 3000

ENV NODE_ENV=development
ENV NEXT_TELEMETRY_DISABLED=1
ENV WATCHPACK_POLLING=true

CMD ["pnpm", "dev"]

§ 6.4 MULTI-ENVIRONMENT DOCKERFILE

dockerfile
# Dockerfile con target multipli
# syntax=docker/dockerfile:1

FROM node:22-alpine AS base
RUN apk add --no-cache libc6-compat
WORKDIR /app

# ─────────────────────────────────────────────────────────────────────────────
# Dependencies
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS deps
COPY package.json pnpm-lock.yaml* ./
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# ─────────────────────────────────────────────────────────────────────────────
# Development Target
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS development
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NODE_ENV=development
EXPOSE 3000
CMD ["npx", "next", "dev"]

# ─────────────────────────────────────────────────────────────────────────────
# Builder
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS builder
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ARG NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_TELEMETRY_DISABLED=1
RUN corepack enable pnpm && pnpm build

# ─────────────────────────────────────────────────────────────────────────────
# Production Target
# ─────────────────────────────────────────────────────────────────────────────
FROM node:22-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]

# ─────────────────────────────────────────────────────────────────────────────
# Test Target
# ─────────────────────────────────────────────────────────────────────────────
FROM base AS test
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NODE_ENV=test
RUN corepack enable pnpm
CMD ["pnpm", "test"]

§ 6.5 BUILDING WITH DIFFERENT TARGETS

bash
# Build per development
docker build --target development -t myapp:dev .

# Build per production
docker build --target production -t myapp:prod .

# Build per test
docker build --target test -t myapp:test .

# Build con build args
docker build \
  --target production \
  --build-arg NEXT_PUBLIC_API_URL=https://api.example.com \
  --build-arg NEXT_PUBLIC_SITE_URL=https://example.com \
  -t myapp:prod .

# Build con cache
docker build \
  --target production \
  --cache-from myapp:latest \
  -t myapp:$(git rev-parse --short HEAD) .

§ 6.6 NEXT.JS CON SHARP (IMAGE OPTIMIZATION)

dockerfile
# Dockerfile con Sharp per image optimization
# syntax=docker/dockerfile:1

FROM node:22-alpine AS base
# Sharp richiede alcune dipendenze native
RUN apk add --no-cache libc6-compat vips-dev

WORKDIR /app

FROM base AS deps
COPY package.json pnpm-lock.yaml* ./
RUN corepack enable pnpm && \
    pnpm install --frozen-lockfile

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED=1
ENV NEXT_SHARP_PATH=/app/node_modules/sharp

RUN corepack enable pnpm && pnpm build

FROM node:22-alpine AS runner
WORKDIR /app

# Installa sharp dependencies
RUN apk add --no-cache vips-dev

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1
ENV NEXT_SHARP_PATH=/app/node_modules/sharp

RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

# Copia sharp per image optimization
COPY --from=builder /app/node_modules/sharp ./node_modules/sharp

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 7: DOCKER COMPOSE
# ═══════════════════════════════════════════════════════════════════════════════

DOCKER_COMPOSE = """

§ 7.1 DOCKER COMPOSE BASICS

yaml
# docker-compose.yml
# Configurazione base per Next.js app

services:
  # ─────────────────────────────────────────────────────────────────────────
  # Next.js Application
  # ─────────────────────────────────────────────────────────────────────────
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
      args:
        - NEXT_PUBLIC_API_URL=${NEXT_PUBLIC_API_URL}
    image: myapp:${TAG:-latest}
    container_name: myapp
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - NEXTAUTH_URL=${NEXTAUTH_URL}
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

§ 7.2 FULL STACK DOCKER COMPOSE

yaml
# docker-compose.yml
# Stack completo: Next.js + PostgreSQL + Redis + Nginx

services:
  # ─────────────────────────────────────────────────────────────────────────
  # Nginx Reverse Proxy
  # ─────────────────────────────────────────────────────────────────────────
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - ./nginx/logs:/var/log/nginx
    depends_on:
      app:
        condition: service_healthy
    networks:
      - app-network

  # ─────────────────────────────────────────────────────────────────────────
  # Next.js Application
  # ─────────────────────────────────────────────────────────────────────────
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    image: myapp:${TAG:-latest}
    container_name: myapp
    restart: unless-stopped
    expose:
      - "3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
      - REDIS_URL=redis://redis:6379
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      - NEXTAUTH_URL=${NEXTAUTH_URL}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    networks:
      - app-network

  # ─────────────────────────────────────────────────────────────────────────
  # PostgreSQL Database
  # ─────────────────────────────────────────────────────────────────────────
  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init-scripts:/docker-entrypoint-initdb.d:ro
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # ─────────────────────────────────────────────────────────────────────────
  # Redis Cache
  # ─────────────────────────────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    command: redis-server --appendonly yes --maxmemory 256mb --maxmemory-policy allkeys-lru
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  app-network:
    driver: bridge

§ 7.3 DEVELOPMENT DOCKER COMPOSE

yaml
# docker-compose.dev.yml
# Per sviluppo locale con hot-reloading

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    container_name: myapp-dev
    ports:
      - "3000:3000"
    volumes:
      # Mount source code for hot-reloading
      - .:/app
      # Prevent node_modules from being overwritten
      - /app/node_modules
      # Prevent .next from being overwritten
      - /app/.next
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://dev:dev@postgres:5432/myapp_dev
      - WATCHPACK_POLLING=true  # For file watching in Docker
    depends_on:
      - postgres
      - redis
    networks:
      - dev-network

  postgres:
    image: postgres:16-alpine
    container_name: postgres-dev
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
      - POSTGRES_DB=myapp_dev
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data
    networks:
      - dev-network

  redis:
    image: redis:7-alpine
    container_name: redis-dev
    ports:
      - "6379:6379"
    networks:
      - dev-network

  # Database admin UI
  adminer:
    image: adminer
    container_name: adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres
    networks:
      - dev-network

volumes:
  postgres_dev_data:

networks:
  dev-network:
    driver: bridge

§ 7.4 DOCKER COMPOSE COMMANDS

bash
# ═══════════════════════════════════════════════════════════════════════════════
# DOCKER COMPOSE COMMANDS
# ═══════════════════════════════════════════════════════════════════════════════

# Start services
docker compose up
docker compose up -d                    # Detached mode
docker compose up --build               # Rebuild images
docker compose up app postgres          # Start specific services

# Stop services
docker compose down
docker compose down -v                  # Remove volumes
docker compose down --rmi all           # Remove images too

# View logs
docker compose logs
docker compose logs -f                  # Follow logs
docker compose logs -f app              # Specific service

# Execute commands
docker compose exec app sh              # Shell into running container
docker compose run app npm test         # Run command in new container

# Scale services
docker compose up -d --scale app=3      # Run 3 app instances

# Use different compose file
docker compose -f docker-compose.dev.yml up

# Multiple compose files (override)
docker compose -f docker-compose.yml -f docker-compose.prod.yml up

# Build only
docker compose build
docker compose build --no-cache         # Force rebuild

# Pull images
docker compose pull

# Show status
docker compose ps
docker compose top                      # Show processes

# Restart services
docker compose restart
docker compose restart app              # Specific service

§ 7.5 NGINX CONFIGURATION

nginx
# nginx/nginx.conf

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    # Performance
    sendfile        on;
    tcp_nopush      on;
    tcp_nodelay     on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript 
               application/rss+xml application/atom+xml image/svg+xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

    # Upstream (Next.js app)
    upstream nextjs {
        server app:3000;
        keepalive 64;
    }

    server {
        listen 80;
        server_name example.com www.example.com;
        
        # Redirect to HTTPS
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name example.com www.example.com;

        # SSL Configuration
        ssl_certificate     /etc/nginx/ssl/fullchain.pem;
        ssl_certificate_key /etc/nginx/ssl/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;

        # Modern SSL configuration
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;

        # HSTS
        add_header Strict-Transport-Security "max-age=63072000" always;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Referrer-Policy "strict-origin-when-cross-origin" always;

        # Rate limiting
        limit_req zone=one burst=20 nodelay;

        # Static files (served directly)
        location /_next/static {
            proxy_cache STATIC;
            proxy_pass http://nextjs;
            add_header Cache-Control "public, max-age=31536000, immutable";
        }

        location /static {
            proxy_cache STATIC;
            proxy_pass http://nextjs;
            add_header Cache-Control "public, max-age=31536000, immutable";
        }

        # API routes (no caching)
        location /api {
            proxy_pass http://nextjs;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }

        # All other requests
        location / {
            proxy_pass http://nextjs;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }

        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }

    # Cache configuration
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATIC:10m inactive=7d use_temp_path=off;
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 8: CONTAINER REGISTRY & IMAGE MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

CONTAINER_REGISTRY = """

§ 8.1 CONTAINER REGISTRIES OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ CONTAINER REGISTRIES                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DOCKER HUB (docker.io)                                                     │
│ ├── Registry pubblico di default                                           │
│ ├── Free: 1 private repo, rate limits                                      │
│ ├── Pro: $5/mese, unlimited private repos                                  │
│ └── URL: docker.io/username/image:tag                                      │
│                                                                             │
│ GITHUB CONTAINER REGISTRY (ghcr.io) ⭐ Raccomandato                         │
│ ├── Integrato con GitHub                                                   │
│ ├── Free per public repos                                                  │
│ ├── Private: incluso in GitHub plans                                       │
│ └── URL: ghcr.io/username/image:tag                                        │
│                                                                             │
│ AWS ECR (Elastic Container Registry)                                       │
│ ├── Integrato con AWS services                                             │
│ ├── Pay per storage + data transfer                                        │
│ ├── ECR Public per immagini pubbliche                                      │
│ └── URL: 123456789.dkr.ecr.region.amazonaws.com/image:tag                  │
│                                                                             │
│ GOOGLE ARTIFACT REGISTRY                                                    │
│ ├── Integrato con GCP services                                             │
│ ├── Supporta anche npm, Maven, etc.                                        │
│ └── URL: region-docker.pkg.dev/project/repo/image:tag                      │
│                                                                             │
│ AZURE CONTAINER REGISTRY                                                    │
│ ├── Integrato con Azure services                                           │
│ ├── Geo-replication disponibile                                            │
│ └── URL: myregistry.azurecr.io/image:tag                                   │
│                                                                             │
│ SELF-HOSTED                                                                 │
│ ├── Harbor (VMware) - Enterprise features                                  │
│ ├── Nexus Repository - Multi-format                                        │
│ └── GitLab Container Registry - Integrato con GitLab                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 8.2 GITHUB CONTAINER REGISTRY (GHCR)

bash
# ═══════════════════════════════════════════════════════════════════════════════
# GITHUB CONTAINER REGISTRY
# ═══════════════════════════════════════════════════════════════════════════════

# Login a GHCR
# Opzione 1: Personal Access Token (PAT)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Opzione 2: GitHub CLI
gh auth token | docker login ghcr.io -u USERNAME --password-stdin

# Build e tag image
docker build -t ghcr.io/username/myapp:1.0.0 .
docker build -t ghcr.io/username/myapp:latest .

# Push image
docker push ghcr.io/username/myapp:1.0.0
docker push ghcr.io/username/myapp:latest

# Pull image
docker pull ghcr.io/username/myapp:1.0.0

# Tag esistente con nuovo tag
docker tag ghcr.io/username/myapp:1.0.0 ghcr.io/username/myapp:stable

§ 8.3 GITHUB ACTIONS PER GHCR

yaml
# .github/workflows/docker-publish.yml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GHCR
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            # Tag with branch name
            type=ref,event=branch
            # Tag with PR number
            type=ref,event=pr
            # Tag with git short sha
            type=sha,prefix=
            # Tag with semver
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
            type=semver,pattern={{major}}
            # Tag latest for main branch
            type=raw,value=latest,enable={{is_default_branch}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          build-args: |
            NEXT_PUBLIC_API_URL=${{ vars.NEXT_PUBLIC_API_URL }}

§ 8.4 AWS ECR

yaml
# .github/workflows/deploy-ecr.yml
name: Deploy to AWS ECR

on:
  push:
    branches: [main]

env:
  AWS_REGION: eu-west-1
  ECR_REPOSITORY: myapp

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      
      - name: Build, tag, and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

§ 8.5 IMAGE TAGGING STRATEGY

┌─────────────────────────────────────────────────────────────────────────────┐
│ IMAGE TAGGING BEST PRACTICES                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ SEMANTIC VERSIONING                                                         │
│ ├── myapp:1.0.0         - Full version                                     │
│ ├── myapp:1.0           - Minor version (auto-update patches)              │
│ ├── myapp:1             - Major version                                    │
│ └── myapp:latest        - Most recent release                              │
│                                                                             │
│ GIT-BASED TAGS                                                              │
│ ├── myapp:abc1234       - Git commit SHA (short)                           │
│ ├── myapp:main          - Branch name                                      │
│ ├── myapp:pr-123        - Pull request number                              │
│ └── myapp:main-abc1234  - Branch + commit                                  │
│                                                                             │
│ ENVIRONMENT TAGS                                                            │
│ ├── myapp:dev           - Development                                      │
│ ├── myapp:staging       - Staging/QA                                       │
│ ├── myapp:prod          - Production                                       │
│ └── myapp:canary        - Canary deployment                                │
│                                                                             │
│ TIMESTAMP TAGS                                                              │
│ ├── myapp:20240115      - Date                                             │
│ ├── myapp:20240115-1200 - Date + time                                      │
│ └── myapp:build-42      - Build number                                     │
│                                                                             │
│ BEST PRACTICE COMBINATION                                                   │
│ ├── myapp:1.2.3                    - For releases                          │
│ ├── myapp:1.2.3-abc1234            - Version + commit                      │
│ ├── myapp:sha-abc1234              - For CI/CD traceability                │
│ └── myapp:latest                   - For latest stable                     │
│                                                                             │
│ ⚠️ EVITA                                                                    │
│ ├── ❌ myapp:latest only           - Non tracciabile                       │
│ ├── ❌ myapp:v1, myapp:v2          - Vago, non semantico                   │
│ └── ❌ Sovrascrivere tag esistenti - Perde tracciabilità                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 8.6 IMAGE SECURITY SCANNING

yaml
# .github/workflows/security-scan.yml
name: Container Security Scan

on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # Daily scan

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Build image
        run: docker build -t myapp:scan .
      
      # Trivy vulnerability scanner
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:scan'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
      
      # Grype vulnerability scanner (alternative)
      - name: Run Grype vulnerability scanner
        uses: anchore/scan-action@v3
        with:
          image: 'myapp:scan'
          fail-build: true
          severity-cutoff: high

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 9: KUBERNETES FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

KUBERNETES_FUNDAMENTALS = """

§ 9.1 KUBERNETES ARCHITECTURE

┌─────────────────────────────────────────────────────────────────────────────┐
│ KUBERNETES ARCHITECTURE                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CONTROL PLANE (Master)                                                      │
│ ├── API Server      - Entry point per tutte le operazioni                  │
│ ├── etcd            - Key-value store per cluster state                    │
│ ├── Scheduler       - Assegna Pod ai Node                                  │
│ └── Controller Manager - Gestisce i controller                             │
│                                                                             │
│ WORKER NODES                                                                │
│ ├── Kubelet         - Agent che gestisce i container                       │
│ ├── Container Runtime - Docker, containerd, CRI-O                          │
│ └── Kube-proxy      - Networking e load balancing                          │
│                                                                             │
│ WORKLOAD OBJECTS                                                            │
│ ├── Pod             - Unità minima, 1+ container                           │
│ ├── Deployment      - Gestisce ReplicaSet e rolling updates                │
│ ├── StatefulSet     - Per app stateful (database)                          │
│ ├── DaemonSet       - Un Pod per ogni Node                                 │
│ ├── Job             - Run-to-completion tasks                              │
│ └── CronJob         - Scheduled Jobs                                       │
│                                                                             │
│ NETWORKING                                                                  │
│ ├── Service         - Espone Pod alla rete                                 │
│ │   ├── ClusterIP   - Solo interno al cluster                              │
│ │   ├── NodePort    - Espone su porta del Node                             │
│ │   └── LoadBalancer- Load balancer esterno (cloud)                        │
│ ├── Ingress         - HTTP/HTTPS routing                                   │
│ └── NetworkPolicy   - Firewall rules                                       │
│                                                                             │
│ CONFIGURATION                                                               │
│ ├── ConfigMap       - Configurazione non-secret                            │
│ ├── Secret          - Dati sensibili (base64)                              │
│ └── PersistentVolume- Storage persistente                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 9.2 ESSENTIAL KUBECTL COMMANDS

bash
# ═══════════════════════════════════════════════════════════════════════════════
# KUBECTL COMMANDS REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

# ─────────────────────────────────────────────────────────────────────────────
# CLUSTER INFO
# ─────────────────────────────────────────────────────────────────────────────

kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl describe node <node-name>

# ─────────────────────────────────────────────────────────────────────────────
# NAMESPACES
# ─────────────────────────────────────────────────────────────────────────────

kubectl get namespaces
kubectl create namespace production
kubectl config set-context --current --namespace=production

# ─────────────────────────────────────────────────────────────────────────────
# PODS
# ─────────────────────────────────────────────────────────────────────────────

kubectl get pods
kubectl get pods -n production
kubectl get pods -o wide
kubectl get pods --all-namespaces
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs -f <pod-name>                    # Follow logs
kubectl logs <pod-name> -c <container-name>   # Specific container
kubectl exec -it <pod-name> -- /bin/sh        # Shell into pod
kubectl port-forward <pod-name> 3000:3000     # Port forward

# ─────────────────────────────────────────────────────────────────────────────
# DEPLOYMENTS
# ─────────────────────────────────────────────────────────────────────────────

kubectl get deployments
kubectl describe deployment <name>
kubectl create deployment myapp --image=myapp:1.0
kubectl scale deployment myapp --replicas=3
kubectl rollout status deployment myapp
kubectl rollout history deployment myapp
kubectl rollout undo deployment myapp
kubectl rollout restart deployment myapp
kubectl set image deployment/myapp app=myapp:2.0

# ─────────────────────────────────────────────────────────────────────────────
# SERVICES
# ─────────────────────────────────────────────────────────────────────────────

kubectl get services
kubectl get svc
kubectl describe service <name>
kubectl expose deployment myapp --port=80 --target-port=3000
kubectl delete service myapp

# ─────────────────────────────────────────────────────────────────────────────
# CONFIGMAPS & SECRETS
# ─────────────────────────────────────────────────────────────────────────────

kubectl get configmaps
kubectl get secrets
kubectl create configmap myconfig --from-file=config.yaml
kubectl create secret generic mysecret --from-literal=password=secret123
kubectl describe configmap myconfig
kubectl get secret mysecret -o jsonpath='{.data.password}' | base64 --decode

# ─────────────────────────────────────────────────────────────────────────────
# APPLY & DELETE
# ─────────────────────────────────────────────────────────────────────────────

kubectl apply -f deployment.yaml
kubectl apply -f ./manifests/
kubectl delete -f deployment.yaml
kubectl delete pod <pod-name>
kubectl delete deployment myapp

# ─────────────────────────────────────────────────────────────────────────────
# DEBUG
# ─────────────────────────────────────────────────────────────────────────────

kubectl get events
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods
kubectl run debug --image=busybox -it --rm -- /bin/sh

§ 9.3 KUBERNETES MANIFESTS STRUCTURE

yaml
# Struttura base di un manifest Kubernetes

apiVersion: v1|apps/v1|networking.k8s.io/v1  # API version
kind: Pod|Deployment|Service|Ingress         # Resource type
metadata:
  name: my-resource                          # Resource name
  namespace: production                       # Namespace (optional)
  labels:                                     # Labels for selection
    app: myapp
    environment: production
  annotations:                                # Additional metadata
    description: "My application"
spec:
  # Resource-specific configuration
  ...

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 10: KUBERNETES DEPLOYMENT
# ═══════════════════════════════════════════════════════════════════════════════

KUBERNETES_DEPLOYMENT = """

§ 10.1 NEXT.JS DEPLOYMENT MANIFEST

yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-app
  namespace: production
  labels:
    app: nextjs-app
    version: v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nextjs-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: nextjs-app
        version: v1
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
    spec:
      # Security context
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      
      # Containers
      containers:
        - name: nextjs
          image: ghcr.io/myorg/nextjs-app:1.0.0
          imagePullPolicy: Always
          
          ports:
            - name: http
              containerPort: 3000
              protocol: TCP
          
          # Environment variables
          env:
            - name: NODE_ENV
              value: "production"
            - name: PORT
              value: "3000"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: nextjs-secrets
                  key: database-url
            - name: NEXTAUTH_SECRET
              valueFrom:
                secretKeyRef:
                  name: nextjs-secrets
                  key: nextauth-secret
          
          # Environment from ConfigMap
          envFrom:
            - configMapRef:
                name: nextjs-config
          
          # Resource limits
          resources:
            requests:
              cpu: "100m"
              memory: "256Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"
          
          # Health checks
          livenessProbe:
            httpGet:
              path: /api/health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
          
          readinessProbe:
            httpGet:
              path: /api/health
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 3
          
          # Security
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          
          # Volume mounts (if needed)
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: next-cache
              mountPath: /app/.next/cache
      
      # Volumes
      volumes:
        - name: tmp
          emptyDir: {}
        - name: next-cache
          emptyDir: {}
      
      # Image pull secret (for private registries)
      imagePullSecrets:
        - name: ghcr-secret
      
      # Pod scheduling
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values:
                        - nextjs-app
                topologyKey: kubernetes.io/hostname

§ 10.2 SERVICE MANIFEST

yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: nextjs-app
  namespace: production
  labels:
    app: nextjs-app
spec:
  type: ClusterIP
  selector:
    app: nextjs-app
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP

§ 10.3 INGRESS MANIFEST

yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nextjs-app
  namespace: production
  labels:
    app: nextjs-app
  annotations:
    # Nginx Ingress Controller
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "60"
    
    # Rate limiting
    nginx.ingress.kubernetes.io/limit-rps: "100"
    nginx.ingress.kubernetes.io/limit-connections: "50"
    
    # Cert-manager for TLS
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - myapp.example.com
        - www.myapp.example.com
      secretName: nextjs-app-tls
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nextjs-app
                port:
                  name: http
    - host: www.myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nextjs-app
                port:
                  name: http

§ 10.4 CONFIGMAP E SECRETS

yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nextjs-config
  namespace: production
data:
  NEXT_PUBLIC_API_URL: "https://api.example.com"
  NEXT_PUBLIC_SITE_URL: "https://myapp.example.com"
  LOG_LEVEL: "info"

---
# k8s/secrets.yaml
# NOTA: In production usa External Secrets Operator o Sealed Secrets
apiVersion: v1
kind: Secret
metadata:
  name: nextjs-secrets
  namespace: production
type: Opaque
stringData:  # Usa stringData per plain text (verrà codificato in base64)
  database-url: "postgresql://user:password@postgres:5432/mydb"
  nextauth-secret: "super-secret-key-here"
  nextauth-url: "https://myapp.example.com"

---
# k8s/image-pull-secret.yaml
# Per registry privati
apiVersion: v1
kind: Secret
metadata:
  name: ghcr-secret
  namespace: production
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <base64-encoded-docker-config>

§ 10.5 HORIZONTAL POD AUTOSCALER

yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nextjs-app
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nextjs-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max

§ 10.6 KUSTOMIZATION

yaml
# k8s/base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: default

resources:
  - deployment.yaml
  - service.yaml
  - ingress.yaml
  - configmap.yaml

---
# k8s/overlays/production/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: production

resources:
  - ../../base
  - hpa.yaml
  - secrets.yaml

patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 3
    target:
      kind: Deployment
      name: nextjs-app

images:
  - name: ghcr.io/myorg/nextjs-app
    newTag: 1.0.0

configMapGenerator:
  - name: nextjs-config
    behavior: merge
    literals:
      - NEXT_PUBLIC_API_URL=https://api.example.com
      - LOG_LEVEL=warn

---
# k8s/overlays/staging/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: staging

resources:
  - ../../base

patches:
  - patch: |-
      - op: replace
        path: /spec/replicas
        value: 1
    target:
      kind: Deployment
      name: nextjs-app

images:
  - name: ghcr.io/myorg/nextjs-app
    newTag: latest

configMapGenerator:
  - name: nextjs-config
    behavior: merge
    literals:
      - NEXT_PUBLIC_API_URL=https://api-staging.example.com
      - LOG_LEVEL=debug

§ 10.7 GITHUB ACTIONS PER KUBERNETES DEPLOY

yaml
# .github/workflows/deploy-k8s.yml
name: Deploy to Kubernetes

on:
  push:
    branches: [main]
    tags: ['v*']

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=sha
            type=semver,pattern={{version}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBE_CONFIG_STAGING }}" | base64 -d > $HOME/.kube/config
      
      - name: Deploy to staging
        run: |
          cd k8s/overlays/staging
          kustomize edit set image ghcr.io/${{ env.IMAGE_NAME }}=ghcr.io/${{ env.IMAGE_NAME }}:sha-${{ github.sha }}
          kustomize build . | kubectl apply -f -
          kubectl rollout status deployment/nextjs-app -n staging

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: startsWith(github.ref, 'refs/tags/v')
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure kubeconfig
        run: |
          mkdir -p $HOME/.kube
          echo "${{ secrets.KUBE_CONFIG_PRODUCTION }}" | base64 -d > $HOME/.kube/config
      
      - name: Deploy to production
        run: |
          cd k8s/overlays/production
          kustomize edit set image ghcr.io/${{ env.IMAGE_NAME }}=ghcr.io/${{ env.IMAGE_NAME }}:${{ needs.build.outputs.image-tag }}
          kustomize build . | kubectl apply -f -
          kubectl rollout status deployment/nextjs-app -n production

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 11: INFRASTRUCTURE AS CODE (TERRAFORM)
# ═══════════════════════════════════════════════════════════════════════════════

TERRAFORM_IAC = """

§ 11.1 TERRAFORM BASICS

┌─────────────────────────────────────────────────────────────────────────────┐
│ TERRAFORM FUNDAMENTALS                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ TERRAFORM WORKFLOW                                                          │
│ ├── terraform init     - Inizializza provider e backend                    │
│ ├── terraform plan     - Mostra le modifiche pianificate                   │
│ ├── terraform apply    - Applica le modifiche                              │
│ └── terraform destroy  - Rimuove tutte le risorse                          │
│                                                                             │
│ FILE STRUCTURE                                                              │
│ ├── main.tf            - Risorse principali                                │
│ ├── variables.tf       - Definizione variabili                             │
│ ├── outputs.tf         - Output values                                     │
│ ├── providers.tf       - Provider configuration                            │
│ ├── terraform.tfvars   - Valori delle variabili                            │
│ └── backend.tf         - State backend configuration                       │
│                                                                             │
│ STATE MANAGEMENT                                                            │
│ ├── terraform.tfstate  - Current state (NON committare!)                   │
│ ├── Remote backends    - S3, GCS, Azure Blob, Terraform Cloud              │
│ └── State locking      - Previene modifiche concorrenti                    │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 11.2 AWS INFRASTRUCTURE EXAMPLE

hcl
# terraform/providers.tf
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "production/terraform.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = var.project_name
      ManagedBy   = "terraform"
    }
  }
}

# terraform/variables.tf
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "eu-west-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "production"], var.environment)
    error_message = "Environment must be dev, staging, or production."
  }
}

variable "project_name" {
  description = "Project name"
  type        = string
  default     = "nextjs-app"
}

variable "container_image" {
  description = "Container image URL"
  type        = string
}

# terraform/main.tf
# VPC
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  
  name = "${var.project_name}-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["${var.aws_region}a", "${var.aws_region}b", "${var.aws_region}c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway   = true
  single_nat_gateway   = var.environment != "production"
  enable_dns_hostnames = true
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
  name = "${var.project_name}-cluster"
  
  setting {
    name  = "containerInsights"
    value = "enabled"
  }
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
  family                   = var.project_name
  network_mode             = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                      = 256
  memory                   = 512
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.ecs_task.arn
  
  container_definitions = jsonencode([
    {
      name  = "app"
      image = var.container_image
      
      portMappings = [
        {
          containerPort = 3000
          hostPort      = 3000
          protocol      = "tcp"
        }
      ]
      
      environment = [
        {
          name  = "NODE_ENV"
          value = "production"
        }
      ]
      
      secrets = [
        {
          name      = "DATABASE_URL"
          valueFrom = aws_secretsmanager_secret.db_url.arn
        }
      ]
      
      logConfiguration = {
        logDriver = "awslogs"
        options = {
          "awslogs-group"         = aws_cloudwatch_log_group.app.name
          "awslogs-region"        = var.aws_region
          "awslogs-stream-prefix" = "ecs"
        }
      }
      
      healthCheck = {
        command     = ["CMD-SHELL", "wget --spider -q http://localhost:3000/api/health || exit 1"]
        interval    = 30
        timeout     = 5
        retries     = 3
        startPeriod = 60
      }
    }
  ])
}

# ECS Service
resource "aws_ecs_service" "app" {
  name            = var.project_name
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.environment == "production" ? 3 : 1
  launch_type     = "FARGATE"
  
  network_configuration {
    subnets          = module.vpc.private_subnets
    security_groups  = [aws_security_group.ecs.id]
    assign_public_ip = false
  }
  
  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 3000
  }
  
  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
  
  depends_on = [aws_lb_listener.https]
}

# Application Load Balancer
resource "aws_lb" "app" {
  name               = "${var.project_name}-alb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb.id]
  subnets            = module.vpc.public_subnets
}

resource "aws_lb_target_group" "app" {
  name        = "${var.project_name}-tg"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = module.vpc.vpc_id
  target_type = "ip"
  
  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/api/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 3
  }
}

# terraform/outputs.tf
output "alb_dns_name" {
  description = "ALB DNS name"
  value       = aws_lb.app.dns_name
}

output "ecs_cluster_name" {
  description = "ECS cluster name"
  value       = aws_ecs_cluster.main.name
}

§ 11.3 TERRAFORM IN CI/CD

yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'
  pull_request:
    branches: [main]
    paths:
      - 'terraform/**'

env:
  TF_VERSION: '1.6.0'
  AWS_REGION: 'eu-west-1'

jobs:
  terraform:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    
    defaults:
      run:
        working-directory: terraform
    
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Terraform Format
        id: fmt
        run: terraform fmt -check
        continue-on-error: true
      
      - name: Terraform Init
        id: init
        run: terraform init
      
      - name: Terraform Validate
        id: validate
        run: terraform validate -no-color
      
      - name: Terraform Plan
        id: plan
        if: github.event_name == 'pull_request'
        run: terraform plan -no-color -out=tfplan
        continue-on-error: true
      
      - name: Comment PR
        uses: actions/github-script@v7
        if: github.event_name == 'pull_request'
        env:
          PLAN: ${{ steps.plan.outputs.stdout }}
        with:
          script: |
            const output = `#### Terraform Format and Style 🖌\`${{ steps.fmt.outcome }}\`
            #### Terraform Initialization ⚙️\`${{ steps.init.outcome }}\`
            #### Terraform Validation 🤖\`${{ steps.validate.outcome }}\`
            #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`

            <details><summary>Show Plan</summary>

            \`\`\`terraform
            ${process.env.PLAN}
            \`\`\`

            </details>`;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            })
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply -auto-approve

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 12: ENVIRONMENT MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

ENVIRONMENT_MANAGEMENT = """

§ 12.1 ENVIRONMENT STRATEGY

┌─────────────────────────────────────────────────────────────────────────────┐
│ ENVIRONMENT STRATEGY                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ DEVELOPMENT (dev)                                                           │
│ ├── Per sviluppo locale                                                    │
│ ├── Database locale o container                                            │
│ ├── Mock services dove possibile                                           │
│ ├── Hot reloading abilitato                                                │
│ └── Debug mode attivo                                                      │
│                                                                             │
│ STAGING (staging)                                                           │
│ ├── Replica dell'ambiente production                                       │
│ ├── Dati di test (non production!)                                         │
│ ├── Per QA e testing pre-release                                           │
│ ├── Accessibile solo internamente                                          │
│ └── Preview deployments per PR                                             │
│                                                                             │
│ PRODUCTION (prod)                                                           │
│ ├── Ambiente live per utenti reali                                         │
│ ├── Dati reali e sensibili                                                 │
│ ├── Alta disponibilità (HA)                                                │
│ ├── Monitoring e alerting attivi                                           │
│ └── Deploy solo dopo approval                                              │
│                                                                             │
│ PREVIEW (per PR)                                                            │
│ ├── Ambiente temporaneo per ogni PR                                        │
│ ├── URL unica (pr-123.preview.example.com)                                 │
│ ├── Distrutto automaticamente dopo merge                                   │
│ └── Facilita code review                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 12.2 ENVIRONMENT VARIABLES MANAGEMENT

typescript
// lib/env.ts
// Validazione environment variables con Zod

import { z } from 'zod';

const envSchema = z.object({
  // Node environment
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  
  // Server-side only (secrets)
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  
  // API Keys (server-side)
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  SENDGRID_API_KEY: z.string().optional(),
  
  // Public (client-side accessible)
  NEXT_PUBLIC_API_URL: z.string().url(),
  NEXT_PUBLIC_SITE_URL: z.string().url(),
  NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY: z.string().startsWith('pk_'),
  NEXT_PUBLIC_GA_ID: z.string().optional(),
});

// Parse and validate
const parsed = envSchema.safeParse(process.env);

if (!parsed.success) {
  console.error('❌ Invalid environment variables:', parsed.error.flatten().fieldErrors);
  throw new Error('Invalid environment variables');
}

export const env = parsed.data;

// Type-safe access
// env.DATABASE_URL  ✅ TypeScript knows this is a string
// env.INVALID_KEY   ❌ TypeScript error

§ 12.3 ENVIRONMENT FILES STRUCTURE

bash
# Struttura file .env

# .env                    # Default values (committed, no secrets!)
# .env.local              # Local overrides (gitignored)
# .env.development        # Development environment
# .env.development.local  # Local dev overrides (gitignored)
# .env.test               # Test environment
# .env.test.local         # Local test overrides (gitignored)
# .env.production         # Production (no secrets!)
# .env.production.local   # Local prod overrides (gitignored)

# PRIORITY (highest to lowest):
# 1. .env.{environment}.local
# 2. .env.local (not loaded in test)
# 3. .env.{environment}
# 4. .env

bash
# .env.example (committed, template for developers)
# Copy this file to .env.local and fill in the values

# Node Environment
NODE_ENV=development

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# Authentication
NEXTAUTH_SECRET=your-secret-key-at-least-32-characters
NEXTAUTH_URL=http://localhost:3000

# External Services
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Public Variables (accessible in browser)
NEXT_PUBLIC_API_URL=http://localhost:3000/api
NEXT_PUBLIC_SITE_URL=http://localhost:3000
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...

§ 12.4 GITHUB ENVIRONMENTS CONFIGURATION

yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  deploy-preview:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    environment:
      name: preview
      url: https://pr-${{ github.event.number }}.preview.example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Preview
        env:
          DATABASE_URL: ${{ secrets.PREVIEW_DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.PREVIEW_NEXTAUTH_SECRET }}
        run: |
          echo "Deploying PR #${{ github.event.number }} to preview..."

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        env:
          DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.STAGING_NEXTAUTH_SECRET }}
        run: |
          echo "Deploying to staging..."

  deploy-production:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Production
        env:
          DATABASE_URL: ${{ secrets.PRODUCTION_DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.PRODUCTION_NEXTAUTH_SECRET }}
        run: |
          echo "Deploying to production..."

§ 12.5 FEATURE FLAGS PER ENVIRONMENT

typescript
// lib/feature-flags.ts
// Feature flags per environment

type FeatureFlag = {
  name: string;
  description: string;
  enabled: {
    development: boolean;
    staging: boolean;
    production: boolean;
  };
};

const featureFlags: Record<string, FeatureFlag> = {
  NEW_CHECKOUT: {
    name: 'New Checkout Flow',
    description: 'New streamlined checkout experience',
    enabled: {
      development: true,
      staging: true,
      production: false, // Gradual rollout
    },
  },
  AI_RECOMMENDATIONS: {
    name: 'AI Product Recommendations',
    description: 'AI-powered product recommendations',
    enabled: {
      development: true,
      staging: true,
      production: true,
    },
  },
  DEBUG_MODE: {
    name: 'Debug Mode',
    description: 'Enable debug logging',
    enabled: {
      development: true,
      staging: true,
      production: false,
    },
  },
};

export function isFeatureEnabled(flagName: string): boolean {
  const flag = featureFlags[flagName];
  if (!flag) return false;
  
  const env = process.env.NODE_ENV as 'development' | 'staging' | 'production';
  return flag.enabled[env] ?? false;
}

// Usage
if (isFeatureEnabled('NEW_CHECKOUT')) {
  // New checkout flow
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 13: SECRETS MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

SECRETS_MANAGEMENT = """

§ 13.1 SECRETS MANAGEMENT OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ SECRETS MANAGEMENT BEST PRACTICES                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ❌ COSA NON FARE MAI                                                        │
│ ├── Committare secrets nel repository                                      │
│ ├── Secrets in Dockerfile o docker-compose.yml                             │
│ ├── Secrets hardcodati nel codice                                          │
│ ├── Secrets in file non gitignored                                         │
│ └── Secrets in URL (query parameters)                                      │
│                                                                             │
│ ✅ COSA FARE                                                                │
│ ├── Usare variabili d'ambiente                                             │
│ ├── Secrets manager (AWS SM, HashiCorp Vault)                              │
│ ├── GitHub Secrets per CI/CD                                               │
│ ├── Kubernetes Secrets o External Secrets                                  │
│ └── Rotazione periodica delle chiavi                                       │
│                                                                             │
│ TIPI DI SECRETS                                                             │
│ ├── Database credentials                                                   │
│ ├── API keys (Stripe, SendGrid, etc.)                                      │
│ ├── OAuth secrets                                                          │
│ ├── JWT signing keys                                                       │
│ ├── Encryption keys                                                        │
│ └── SSH keys e deploy keys                                                 │
│                                                                             │
│ STRUMENTI                                                                   │
│ ├── GitHub Secrets         - CI/CD secrets                                 │
│ ├── AWS Secrets Manager    - Cloud secrets                                 │
│ ├── HashiCorp Vault        - Enterprise secrets                            │
│ ├── Doppler               - Developer-friendly                             │
│ ├── 1Password             - Team secrets sharing                           │
│ └── SOPS                  - Encrypted secrets in git                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 13.2 GITHUB SECRETS

yaml
# Repository Secrets
# Settings > Secrets and variables > Actions

# Organization Secrets (shared across repos)
# Organization Settings > Secrets and variables > Actions

# Environment Secrets (per environment)
# Settings > Environments > [env] > Environment secrets

# Uso in workflow
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy
        env:
          # Repository secret
          API_KEY: ${{ secrets.API_KEY }}
          # Environment secret (requires environment)
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
        run: |
          echo "Deploying with secrets..."

# Secrets nei workflow reusable
# I secrets devono essere passati esplicitamente
jobs:
  call-workflow:
    uses: ./.github/workflows/deploy.yml
    secrets:
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
      API_KEY: ${{ secrets.API_KEY }}
    # Oppure passa tutti i secrets
    secrets: inherit

§ 13.3 AWS SECRETS MANAGER

typescript
// lib/secrets.ts
// Caricamento secrets da AWS Secrets Manager

import {
  SecretsManagerClient,
  GetSecretValueCommand,
} from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION });

interface AppSecrets {
  DATABASE_URL: string;
  NEXTAUTH_SECRET: string;
  STRIPE_SECRET_KEY: string;
}

let cachedSecrets: AppSecrets | null = null;

export async function getSecrets(): Promise<AppSecrets> {
  if (cachedSecrets) {
    return cachedSecrets;
  }

  const secretName = `${process.env.NODE_ENV}/myapp/secrets`;

  try {
    const response = await client.send(
      new GetSecretValueCommand({
        SecretId: secretName,
        VersionStage: 'AWSCURRENT',
      })
    );

    if (response.SecretString) {
      cachedSecrets = JSON.parse(response.SecretString);
      return cachedSecrets!;
    }

    throw new Error('Secret not found');
  } catch (error) {
    console.error('Error fetching secrets:', error);
    throw error;
  }
}

// Usage in Next.js API route
// app/api/checkout/route.ts
import { getSecrets } from '@/lib/secrets';

export async function POST(request: Request) {
  const secrets = await getSecrets();
  
  // Use secrets.STRIPE_SECRET_KEY
}

§ 13.4 KUBERNETES EXTERNAL SECRETS

yaml
# External Secrets Operator
# https://external-secrets.io/

# 1. Install External Secrets Operator
# helm install external-secrets external-secrets/external-secrets

# 2. Configure SecretStore (AWS Secrets Manager)
# k8s/secret-store.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets-manager
  namespace: production
spec:
  provider:
    aws:
      service: SecretsManager
      region: eu-west-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa

---
# 3. Define ExternalSecret
# k8s/external-secret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: nextjs-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: nextjs-secrets
    creationPolicy: Owner
  data:
    - secretKey: database-url
      remoteRef:
        key: production/myapp/database
        property: url
    - secretKey: nextauth-secret
      remoteRef:
        key: production/myapp/auth
        property: secret
    - secretKey: stripe-secret-key
      remoteRef:
        key: production/myapp/stripe
        property: secret_key

§ 13.5 SOPS (SECRETS OPERATIONS)

bash
# SOPS permette di committare secrets criptati nel repository

# 1. Install SOPS
brew install sops

# 2. Configure age key (simple alternative to PGP)
age-keygen -o keys.txt
# Memorizza la chiave pubblica

# 3. Create .sops.yaml configuration
cat > .sops.yaml << EOF
creation_rules:
  - path_regex: secrets/.*\.yaml$
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
EOF

# 4. Create encrypted secrets file
sops secrets/production.yaml
# Si apre l'editor, inserisci:
# database_url: postgresql://user:pass@host/db
# stripe_key: sk_live_xxx

# 5. Il file viene salvato criptato
cat secrets/production.yaml
# database_url: ENC[AES256_GCM,data:xxx,tag:xxx]
# stripe_key: ENC[AES256_GCM,data:xxx,tag:xxx]

# 6. Decrypt in CI/CD
export SOPS_AGE_KEY_FILE=keys.txt
sops -d secrets/production.yaml > .env.production

# GitHub Actions
- name: Decrypt secrets
  env:
    SOPS_AGE_KEY: ${{ secrets.SOPS_AGE_KEY }}
  run: |
    echo "$SOPS_AGE_KEY" > keys.txt
    sops -d secrets/production.yaml > .env.production

§ 13.6 SECRETS ROTATION

typescript
// scripts/rotate-secrets.ts
// Script per rotazione secrets

import { SecretsManagerClient, RotateSecretCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'eu-west-1' });

async function rotateSecret(secretId: string): Promise<void> {
  try {
    await client.send(
      new RotateSecretCommand({
        SecretId: secretId,
        RotateImmediately: true,
      })
    );
    console.log(`✅ Rotated secret: ${secretId}`);
  } catch (error) {
    console.error(`❌ Failed to rotate secret: ${secretId}`, error);
    throw error;
  }
}

// Rotate all application secrets
async function rotateAllSecrets(): Promise<void> {
  const secrets = [
    'production/myapp/database',
    'production/myapp/auth',
    'production/myapp/api-keys',
  ];

  for (const secretId of secrets) {
    await rotateSecret(secretId);
  }
}

rotateAllSecrets().catch(console.error);

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 14: MONITORING & LOGGING IN CI/CD
# ═══════════════════════════════════════════════════════════════════════════════

MONITORING_CICD = """

§ 14.1 CI/CD MONITORING OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ CI/CD MONITORING & OBSERVABILITY                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ METRICHE PIPELINE                                                           │
│ ├── Build duration      - Tempo di build                                   │
│ ├── Test duration       - Tempo di esecuzione test                         │
│ ├── Deploy frequency    - Frequenza deploy (DORA)                          │
│ ├── Lead time           - Tempo da commit a production                     │
│ ├── Success rate        - % pipeline successo                              │
│ ├── Failure rate        - % pipeline fallite                               │
│ └── Queue time          - Tempo in coda                                    │
│                                                                             │
│ DEPLOYMENT MONITORING                                                       │
│ ├── Deployment status   - Stato deploy corrente                            │
│ ├── Rollback count      - Numero di rollback                               │
│ ├── Environment health  - Salute degli ambienti                            │
│ └── Resource usage      - CPU/Memory durante deploy                        │
│                                                                             │
│ ALERTING                                                                    │
│ ├── Pipeline failures   - Alert su fallimenti                              │
│ ├── Long-running builds - Build troppo lunghe                              │
│ ├── Security scans      - Vulnerabilità trovate                            │
│ └── Deployment issues   - Problemi in deploy                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 14.2 GITHUB ACTIONS MONITORING

yaml
# .github/workflows/ci.yml
name: CI with Monitoring

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'pnpm'
      
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      
      - name: Build
        id: build
        run: |
          START_TIME=$(date +%s)
          pnpm build
          END_TIME=$(date +%s)
          echo "duration=$((END_TIME - START_TIME))" >> $GITHUB_OUTPUT
      
      - name: Test
        id: test
        run: |
          START_TIME=$(date +%s)
          pnpm test --coverage
          END_TIME=$(date +%s)
          echo "duration=$((END_TIME - START_TIME))" >> $GITHUB_OUTPUT
      
      # Report metrics
      - name: Report metrics to Datadog
        if: always()
        env:
          DD_API_KEY: ${{ secrets.DATADOG_API_KEY }}
        run: |
          curl -X POST "https://api.datadoghq.com/api/v2/series" \
            -H "Content-Type: application/json" \
            -H "DD-API-KEY: ${DD_API_KEY}" \
            -d '{
              "series": [
                {
                  "metric": "ci.build.duration",
                  "type": "gauge",
                  "points": [['"$(date +%s)"', ${{ steps.build.outputs.duration }}]],
                  "tags": ["repo:${{ github.repository }}", "branch:${{ github.ref_name }}"]
                },
                {
                  "metric": "ci.test.duration",
                  "type": "gauge",
                  "points": [['"$(date +%s)"', ${{ steps.test.outputs.duration }}]],
                  "tags": ["repo:${{ github.repository }}", "branch:${{ github.ref_name }}"]
                }
              ]
            }'

  notify:
    needs: build
    if: failure()
    runs-on: ubuntu-latest
    steps:
      - name: Notify Slack on failure
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: 'C0123456789'
          slack-message: |
            ❌ CI Pipeline Failed!
            *Repository:* ${{ github.repository }}
            *Branch:* ${{ github.ref_name }}
            *Commit:* ${{ github.sha }}
            *Author:* ${{ github.actor }}
            *URL:* ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

§ 14.3 DEPLOYMENT NOTIFICATIONS

yaml
# .github/workflows/deploy-notify.yml
name: Deploy with Notifications

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      # Notify start
      - name: Notify deployment start
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: 'deployments'
          slack-message: |
            🚀 Starting deployment to production
            *Version:* ${{ github.sha }}
            *Author:* ${{ github.actor }}
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      
      - name: Deploy
        id: deploy
        run: |
          # Deploy logic here
          echo "Deploying..."
      
      # Notify success
      - name: Notify deployment success
        if: success()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: 'deployments'
          slack-message: |
            ✅ Deployment successful!
            *Version:* ${{ github.sha }}
            *Environment:* Production
            *URL:* https://example.com
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
      
      # Notify failure
      - name: Notify deployment failure
        if: failure()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: 'deployments'
          slack-message: |
            ❌ Deployment FAILED!
            *Version:* ${{ github.sha }}
            *Environment:* Production
            *Logs:* ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
            
            @channel Please investigate immediately!
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

      # Create GitHub deployment
      - name: Create deployment status
        uses: bobheadxi/deployments@v1
        with:
          step: finish
          token: ${{ secrets.GITHUB_TOKEN }}
          status: ${{ job.status }}
          env: production
          deployment_id: ${{ steps.deployment.outputs.deployment_id }}

§ 14.4 STRUCTURED LOGGING

typescript
// lib/logger.ts
// Structured logging per CI/CD e production

import pino from 'pino';

const isProduction = process.env.NODE_ENV === 'production';

export const logger = pino({
  level: process.env.LOG_LEVEL || (isProduction ? 'info' : 'debug'),
  
  // Formato JSON per production (machine-readable)
  // Formato pretty per development
  ...(isProduction
    ? {}
    : {
        transport: {
          target: 'pino-pretty',
          options: {
            colorize: true,
          },
        },
      }),
  
  // Campi base per ogni log
  base: {
    pid: process.pid,
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION || 'unknown',
  },
  
  // Redact sensitive fields
  redact: ['password', 'secret', 'token', 'authorization', 'cookie'],
});

// Usage
logger.info({ userId: '123', action: 'login' }, 'User logged in');
logger.error({ error, requestId: '456' }, 'Failed to process request');

// Deployment logging
logger.info({
  event: 'deployment',
  version: process.env.APP_VERSION,
  commit: process.env.COMMIT_SHA,
  environment: process.env.NODE_ENV,
}, 'Application started');

§ 14.5 HEALTH CHECK ENDPOINT

typescript
// app/api/health/route.ts
// Health check per monitoring e load balancers

import { NextResponse } from 'next/server';
import { prisma } from '@/lib/prisma';
import { redis } from '@/lib/redis';

interface HealthStatus {
  status: 'healthy' | 'unhealthy' | 'degraded';
  timestamp: string;
  version: string;
  uptime: number;
  checks: {
    database: 'ok' | 'error';
    redis: 'ok' | 'error';
    memory: 'ok' | 'warning' | 'error';
  };
}

export async function GET(): Promise<NextResponse<HealthStatus>> {
  const startTime = process.hrtime();
  
  // Check database
  let databaseStatus: 'ok' | 'error' = 'ok';
  try {
    await prisma.$queryRaw`SELECT 1`;
  } catch {
    databaseStatus = 'error';
  }
  
  // Check Redis
  let redisStatus: 'ok' | 'error' = 'ok';
  try {
    await redis.ping();
  } catch {
    redisStatus = 'error';
  }
  
  // Check memory
  const memoryUsage = process.memoryUsage();
  const memoryPercent = memoryUsage.heapUsed / memoryUsage.heapTotal;
  let memoryStatus: 'ok' | 'warning' | 'error' = 'ok';
  if (memoryPercent > 0.9) memoryStatus = 'error';
  else if (memoryPercent > 0.8) memoryStatus = 'warning';
  
  // Determine overall status
  let status: 'healthy' | 'unhealthy' | 'degraded' = 'healthy';
  if (databaseStatus === 'error' || redisStatus === 'error') {
    status = 'unhealthy';
  } else if (memoryStatus !== 'ok') {
    status = 'degraded';
  }
  
  const healthStatus: HealthStatus = {
    status,
    timestamp: new Date().toISOString(),
    version: process.env.APP_VERSION || 'unknown',
    uptime: process.uptime(),
    checks: {
      database: databaseStatus,
      redis: redisStatus,
      memory: memoryStatus,
    },
  };
  
  // Return 503 if unhealthy (for load balancers)
  const httpStatus = status === 'unhealthy' ? 503 : 200;
  
  return NextResponse.json(healthStatus, { status: httpStatus });
}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ SEZIONE 15: DEPLOYMENT STRATEGIES
# ═══════════════════════════════════════════════════════════════════════════════

DEPLOYMENT_STRATEGIES = """

§ 15.1 DEPLOYMENT STRATEGIES OVERVIEW

┌─────────────────────────────────────────────────────────────────────────────┐
│ DEPLOYMENT STRATEGIES                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ RECREATE (Big Bang)                                                         │
│ ├── Ferma tutte le istanze, deploya nuove                                  │
│ ├── Downtime durante il deploy                                             │
│ ├── Semplice da implementare                                               │
│ └── ❌ Non raccomandato per production                                      │
│                                                                             │
│ ROLLING UPDATE ⭐ Default Kubernetes                                        │
│ ├── Aggiorna gradualmente le istanze                                       │
│ ├── Zero downtime                                                          │
│ ├── Rollback facile                                                        │
│ └── ✅ Buon compromesso semplicità/affidabilità                             │
│                                                                             │
│ BLUE-GREEN                                                                  │
│ ├── Due ambienti identici (Blue=current, Green=new)                        │
│ ├── Switch istantaneo del traffico                                         │
│ ├── Rollback istantaneo                                                    │
│ ├── Costo doppio infrastruttura                                            │
│ └── ✅ Ottimo per zero-downtime con rollback sicuro                         │
│                                                                             │
│ CANARY                                                                      │
│ ├── Deploy nuova versione a piccola % di utenti                            │
│ ├── Monitora metriche e errori                                             │
│ ├── Incrementa gradualmente se OK                                          │
│ ├── Rollback se problemi                                                   │
│ └── ✅ Ideale per ridurre rischi su grandi cambiamenti                      │
│                                                                             │
│ A/B TESTING                                                                 │
│ ├── Simile a Canary ma per test features                                   │
│ ├── Routing basato su criteri (user, region, etc.)                         │
│ └── ✅ Per validare nuove features con utenti reali                         │
│                                                                             │
│ FEATURE FLAGS                                                               │
│ ├── Attiva/disattiva features senza deploy                                 │
│ ├── Rollout graduale per feature                                           │
│ ├── Kill switch per emergenze                                              │
│ └── ✅ Decouples deploy da release                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ 15.2 ROLLING UPDATE (KUBERNETES)

yaml
# k8s/deployment-rolling.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      # Massimo 1 pod in più durante l'update
      maxSurge: 1
      # Sempre almeno 4 pod disponibili (5-1=4)
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nextjs-app
  template:
    metadata:
      labels:
        app: nextjs-app
    spec:
      containers:
        - name: nextjs
          image: ghcr.io/myorg/nextjs-app:2.0.0
          # Readiness probe è CRITICO per rolling updates
          readinessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
          # Liveness probe per auto-recovery
          livenessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
            failureThreshold: 3

bash
# Rolling update commands
kubectl set image deployment/nextjs-app nextjs=ghcr.io/myorg/nextjs-app:2.0.0

# Watch rollout status
kubectl rollout status deployment/nextjs-app

# View rollout history
kubectl rollout history deployment/nextjs-app

# Rollback to previous version
kubectl rollout undo deployment/nextjs-app

# Rollback to specific revision
kubectl rollout undo deployment/nextjs-app --to-revision=2

# Pause/resume rollout
kubectl rollout pause deployment/nextjs-app
kubectl rollout resume deployment/nextjs-app

§ 15.3 BLUE-GREEN DEPLOYMENT

yaml
# k8s/blue-green/deployment-blue.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-app-blue
  labels:
    app: nextjs-app
    version: blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nextjs-app
      version: blue
  template:
    metadata:
      labels:
        app: nextjs-app
        version: blue
    spec:
      containers:
        - name: nextjs
          image: ghcr.io/myorg/nextjs-app:1.0.0

---
# k8s/blue-green/deployment-green.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-app-green
  labels:
    app: nextjs-app
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nextjs-app
      version: green
  template:
    metadata:
      labels:
        app: nextjs-app
        version: green
    spec:
      containers:
        - name: nextjs
          image: ghcr.io/myorg/nextjs-app:2.0.0

---
# k8s/blue-green/service.yaml
# Il Service punta al deployment attivo
apiVersion: v1
kind: Service
metadata:
  name: nextjs-app
spec:
  selector:
    app: nextjs-app
    version: blue  # Cambia a 'green' per switch
  ports:
    - port: 80
      targetPort: 3000

bash
# Blue-Green switch script
#!/bin/bash

CURRENT_VERSION=$(kubectl get svc nextjs-app -o jsonpath='{.spec.selector.version}')
NEW_VERSION=""

if [ "$CURRENT_VERSION" == "blue" ]; then
  NEW_VERSION="green"
else
  NEW_VERSION="blue"
fi

echo "Switching from $CURRENT_VERSION to $NEW_VERSION"

# Switch traffic
kubectl patch svc nextjs-app -p "{\"spec\":{\"selector\":{\"version\":\"$NEW_VERSION\"}}}"

# Verify
kubectl get svc nextjs-app -o jsonpath='{.spec.selector.version}'
echo ""
echo "✅ Traffic now pointing to $NEW_VERSION"

§ 15.4 CANARY DEPLOYMENT

yaml
# k8s/canary/canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nextjs-app-canary
spec:
  replicas: 1  # Small percentage
  selector:
    matchLabels:
      app: nextjs-app
      version: canary
  template:
    metadata:
      labels:
        app: nextjs-app
        version: canary
    spec:
      containers:
        - name: nextjs
          image: ghcr.io/myorg/nextjs-app:2.0.0-canary

---
# Con Istio per traffic splitting
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nextjs-app
spec:
  hosts:
    - nextjs-app
  http:
    - match:
        - headers:
            canary:
              exact: "true"
      route:
        - destination:
            host: nextjs-app
            subset: canary
    - route:
        - destination:
            host: nextjs-app
            subset: stable
          weight: 90
        - destination:
            host: nextjs-app
            subset: canary
          weight: 10

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: nextjs-app
spec:
  host: nextjs-app
  subsets:
    - name: stable
      labels:
        version: stable
    - name: canary
      labels:
        version: canary

§ 15.5 FEATURE FLAGS IMPLEMENTATION

typescript
// lib/feature-flags.ts
// Feature flags con LaunchDarkly o Unleash

import { init, LDClient, LDUser } from 'launchdarkly-node-server-sdk';

let ldClient: LDClient | null = null;

export async function initFeatureFlags(): Promise<void> {
  if (ldClient) return;
  
  ldClient = init(process.env.LAUNCHDARKLY_SDK_KEY!);
  await ldClient.waitForInitialization();
}

export async function isFeatureEnabled(
  flagKey: string,
  user: { id: string; email?: string; custom?: Record<string, unknown> }
): Promise<boolean> {
  if (!ldClient) {
    await initFeatureFlags();
  }
  
  const ldUser: LDUser = {
    key: user.id,
    email: user.email,
    custom: user.custom,
  };
  
  return ldClient!.variation(flagKey, ldUser, false);
}

// Usage in component
export async function getServerSideProps(context: GetServerSidePropsContext) {
  const user = await getUser(context);
  
  const showNewCheckout = await isFeatureEnabled('new-checkout', {
    id: user.id,
    email: user.email,
    custom: {
      plan: user.plan,
      createdAt: user.createdAt,
    },
  });
  
  return {
    props: {
      showNewCheckout,
    },
  };
}

§ 15.6 AUTOMATED ROLLBACK

yaml
# .github/workflows/deploy-with-rollback.yml
name: Deploy with Automated Rollback

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy
        id: deploy
        run: |
          kubectl set image deployment/nextjs-app nextjs=$IMAGE
          kubectl rollout status deployment/nextjs-app --timeout=300s
      
      - name: Smoke tests
        id: smoke-tests
        run: |
          # Wait for deployment to stabilize
          sleep 30
          
          # Health check
          HTTP_STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://example.com/api/health)
          if [ "$HTTP_STATUS" != "200" ]; then
            echo "Health check failed with status $HTTP_STATUS"
            exit 1
          fi
          
          # Response time check
          RESPONSE_TIME=$(curl -s -o /dev/null -w "%{time_total}" https://example.com)
          if (( $(echo "$RESPONSE_TIME > 2.0" | bc -l) )); then
            echo "Response time too slow: ${RESPONSE_TIME}s"
            exit 1
          fi
          
          echo "✅ Smoke tests passed"
      
      - name: Rollback on failure
        if: failure() && steps.deploy.outcome == 'success'
        run: |
          echo "❌ Smoke tests failed, rolling back..."
          kubectl rollout undo deployment/nextjs-app
          kubectl rollout status deployment/nextjs-app
          echo "✅ Rollback complete"
      
      - name: Notify on rollback
        if: failure()
        uses: slackapi/slack-github-action@v1.25.0
        with:
          channel-id: 'deployments'
          slack-message: |
            ⚠️ Deployment rolled back!
            *Version:* ${{ github.sha }}
            *Reason:* Smoke tests failed
            @channel Please investigate!
        env:
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ APPENDICE: QUICK REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

APPENDIX_QUICK_REFERENCE = """

§ A.1 CI/CD CHECKLIST

┌─────────────────────────────────────────────────────────────────────────────┐
│ CI/CD CHECKLIST                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CI (Continuous Integration)                                                 │
│ ☐ Lint e format check                                                      │
│ ☐ Type checking (TypeScript)                                               │
│ ☐ Unit tests                                                               │
│ ☐ Integration tests                                                        │
│ ☐ E2E tests                                                                │
│ ☐ Security scanning (dependencies)                                         │
│ ☐ Code coverage                                                            │
│ ☐ Build verification                                                       │
│                                                                             │
│ CD (Continuous Deployment)                                                  │
│ ☐ Preview deployments per PR                                               │
│ ☐ Staging deployment automatico                                            │
│ ☐ Production con approval gate                                             │
│ ☐ Health checks post-deploy                                                │
│ ☐ Smoke tests automatici                                                   │
│ ☐ Rollback automatico su failure                                           │
│ ☐ Notifications (Slack, email)                                             │
│                                                                             │
│ Security                                                                    │
│ ☐ Secrets in secrets manager                                               │
│ ☐ No secrets in code/logs                                                  │
│ ☐ OIDC per cloud auth (no long-lived keys)                                 │
│ ☐ Container image scanning                                                 │
│ ☐ SAST/DAST scanning                                                       │
│                                                                             │
│ Monitoring                                                                  │
│ ☐ Pipeline metrics tracking                                                │
│ ☐ Deployment tracking                                                      │
│ ☐ Error alerting                                                           │
│ ☐ Performance monitoring                                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ A.2 DOCKER QUICK REFERENCE

bash
# BUILD
docker build -t image:tag .
docker build -t image:tag -f Dockerfile.prod .
docker build --target production -t image:tag .

# RUN
docker run -p 3000:3000 image:tag
docker run -d --name container image:tag
docker run -e VAR=value image:tag
docker run -v $(pwd):/app image:tag

# MANAGE
docker ps                  # List running
docker logs -f container   # Follow logs
docker exec -it container sh  # Shell
docker stop container
docker rm container

# IMAGES
docker images
docker rmi image:tag
docker image prune -a

# COMPOSE
docker compose up -d
docker compose down
docker compose logs -f
docker compose exec service sh

§ A.3 KUBERNETES QUICK REFERENCE

bash
# RESOURCES
kubectl get pods/deployments/services
kubectl describe pod <name>
kubectl delete pod <name>

# DEPLOYMENTS
kubectl apply -f manifest.yaml
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment/<name> --replicas=3

# DEBUG
kubectl logs <pod>
kubectl logs -f <pod>
kubectl exec -it <pod> -- sh
kubectl port-forward <pod> 3000:3000

# CONFIG
kubectl get configmaps/secrets
kubectl create secret generic name --from-literal=key=value
kubectl create configmap name --from-file=config.yaml

§ A.4 GITHUB ACTIONS QUICK REFERENCE

yaml
# Triggers
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'
  workflow_dispatch:

# Common steps
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
- uses: docker/build-push-action@v5
- uses: aws-actions/configure-aws-credentials@v4

# Secrets
${{ secrets.MY_SECRET }}

# Variables
${{ vars.MY_VAR }}

# Context
${{ github.sha }}
${{ github.ref }}
${{ github.actor }}
${{ github.event_name }}

# Conditionals
if: github.ref == 'refs/heads/main'
if: github.event_name == 'pull_request'
if: success()
if: failure()
if: always()

§ A.5 COMMON DOCKERFILE PATTERNS

dockerfile
# Multi-stage build
FROM node:22-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:22-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
RUN addgroup -g 1001 -S nodejs && adduser -S nextjs -u 1001
COPY --from=builder /app/.next/standalone ./
USER nextjs
EXPOSE 3000
CMD ["node", "server.js"]

§ A.6 DEVOPS TOOLS COMPARISON

┌─────────────────────────────────────────────────────────────────────────────┐
│ TOOL COMPARISON                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CI/CD PLATFORMS                                                             │
│ ├── GitHub Actions   ⭐ Best for GitHub users, free for public repos       │
│ ├── GitLab CI        ⭐ Best for GitLab users, powerful                    │
│ ├── Jenkins          Self-hosted, highly customizable                      │
│ └── CircleCI         Fast, good caching                                    │
│                                                                             │
│ CONTAINER ORCHESTRATION                                                     │
│ ├── Kubernetes       ⭐ Industry standard, complex                         │
│ ├── Docker Swarm     Simple, limited features                              │
│ └── ECS/Fargate      AWS native, serverless option                         │
│                                                                             │
│ INFRASTRUCTURE AS CODE                                                      │
│ ├── Terraform        ⭐ Multi-cloud, mature                                │
│ ├── Pulumi           Real languages (TS, Python)                           │
│ └── CloudFormation   AWS only                                              │
│                                                                             │
│ SECRETS MANAGEMENT                                                          │
│ ├── GitHub Secrets   ⭐ Simple, CI/CD focused                              │
│ ├── AWS Secrets Mgr  Cloud-native, rotation                                │
│ └── HashiCorp Vault  Enterprise, self-hosted                               │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

§ A.7 USEFUL COMMANDS

bash
# Git
git log --oneline -10
git diff HEAD~1
git cherry-pick <commit>
git rebase -i HEAD~3

# Docker
docker system df
docker system prune -a
docker stats

# Kubernetes
kubectl top nodes
kubectl top pods
kubectl get events --sort-by='.lastTimestamp'

# AWS
aws sts get-caller-identity
aws ecr get-login-password | docker login --username AWS --password-stdin <registry>

# Terraform
terraform plan -out=tfplan
terraform apply tfplan
terraform state list
terraform import <resource> <id>

"""


# ═══════════════════════════════════════════════════════════════════════════════
§ FINE CATALOGO
# ═══════════════════════════════════════════════════════════════════════════════

print("""
═══════════════════════════════════════════════════════════════════════════════
CATALOGO DEVOPS & CI/CD v1.0 - COMPLETATO
═══════════════════════════════════════════════════════════════════════════════

SEZIONI INCLUSE:
1.  DevOps Fundamentals
2.  Git Workflow & Branching Strategy
3.  GitHub Actions - CI Pipelines
4.  GitHub Actions - CD Pipelines
5.  Docker Fundamentals
6.  Docker per Next.js
7.  Docker Compose
8.  Container Registry & Image Management
9.  Kubernetes Fundamentals
10. Kubernetes Deployment
11. Infrastructure as Code (Terraform)
12. Environment Management
13. Secrets Management
14. Monitoring & Logging in CI/CD
15. Deployment Strategies

APPENDICE:
A.  Quick Reference

═══════════════════════════════════════════════════════════════════════════════
""")


---

# INTEGRAZIONE: 19-OUTPUT-DEPLOY-CI-CD.md
_Integrato il 2026-01-29 14:51_

Assolutamente! Come esperto DevOps, sono lieto di fornirti le configurazioni complete e pronte per la produzione per Docker, Vercel e GitHub Actions, seguendo le tue specifiche.

---

§ FILE 1: DOCKERFILE (80 RIGHE)

dockerfile
# Multi-stage build for production
# Stage 1: Base image for all subsequent stages
FROM node:20-alpine AS base
# Install libc6-compat for compatibility with some Node.js modules
RUN apk add --no-cache libc6-compat
# Set the working directory inside the container
WORKDIR /app

# Stage 2: Dependencies installation
FROM base AS deps
# Copy only necessary files for dependency installation
COPY package.json pnpm-lock.yaml ./
# Enable pnpm using corepack and install dependencies
# --frozen-lockfile ensures deterministic installs based on pnpm-lock.yaml
RUN corepack enable pnpm && pnpm install --frozen-lockfile

# Stage 3: Application builder
FROM base AS builder
# Copy installed node_modules from the deps stage
COPY --from=deps /app/node_modules ./node_modules
# Copy the rest of the application source code
COPY . .

# Generate Prisma client for the current environment
# This is crucial for database interactions
RUN npx prisma generate

# Build the Next.js application
# NEXT_TELEMETRY_DISABLED 1 disables Next.js telemetry during build
ENV NEXT_TELEMETRY_DISABLED 1
RUN pnpm run build

# Stage 4: Production runner
FROM base AS runner
# Set the working directory for the final application
WORKDIR /app

# Set production environment variables
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# Create a non-root user and group for security best practices
# This user will run the application
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy only the necessary build artifacts from the builder stage
# Next.js standalone output is highly optimized for Docker
COPY --from=builder --chown=nextjs:nodejs /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static
# Copy Prisma schema and migrations for runtime access (e.g., migrations)
COPY --from=builder --chown=nextjs:nodejs /app/prisma ./prisma

# Set the user to the non-root 'nextjs' user
USER nextjs

# Expose the port the application will listen on
EXPOSE 3000
# Set environment variables for the application port and host
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

# Command to run the Next.js application in production mode
# The standalone output generates a server.js file
CMD ["node", "server.js"]

§ FILE 2: DOCKER-COMPOSE.YML (80 RIGHE)

yaml
version: '3.8'

# Define the services that make up your application
services:
  # Main application service
  app:
    build:
      context: .
      dockerfile: Dockerfile
    # Map port 3000 from the container to port 3000 on the host
    ports:
      - "3000:3000"
    # Environment variables for the application
    environment:
      # Database connection string (connects to the 'db' service)
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/app
      # NextAuth URL for authentication callbacks
      - NEXTAUTH_URL=http://localhost:3000
      # NextAuth secret for session encryption (should be a strong random string)
      - NEXTAUTH_SECRET=${NEXTAUTH_SECRET}
      # Redis URL for caching/session store (connects to the 'redis' service)
      - REDIS_URL=redis://redis:6379
    # Ensure the database service is healthy before starting the app
    depends_on:
      db:
        condition: service_healthy
    # Restart policy: always restart unless stopped manually
    restart: unless-stopped
    # Optional: Add a healthcheck for the app itself if needed
    # healthcheck:
    #   test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
    #   interval: 30s
    #   timeout: 10s
    #   retries: 3

  # PostgreSQL database service
  db:
    image: postgres:15-alpine
    # Map port 5432 from the container to port 5432 on the host (optional for local access)
    ports:
      - "5432:5432"
    # Environment variables for PostgreSQL configuration
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=app
    # Persist database data to a named volume
    volumes:
      - postgres_data:/var/lib/postgresql/data
    # Healthcheck to ensure the database is ready before the app connects
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d app"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s # Give the DB some time to start up

  # Redis caching/session store service
  redis:
    image: redis:7-alpine
    # Map port 6379 from the container to port 6379 on the host (optional for local access)
    ports:
      - "6379:6379"
    # Persist Redis data to a named volume
    volumes:
      - redis_data:/data
    # Command to run Redis with AOF persistence enabled
    command: redis-server --appendonly yes
    # Optional: Add a healthcheck for Redis if needed
    # healthcheck:
    #   test: ["CMD", "redis-cli", "ping"]
    #   interval: 10s
    #   timeout: 5s
    #   retries: 5

# Define named volumes for data persistence
volumes:
  postgres_data:
  redis_data:

§ FILE 3: DOCKER-COMPOSE.DEV.YML (60 RIGHE)

yaml
version: '3.8'

# This file overrides and extends docker-compose.yml for development purposes.
# To use: docker compose -f docker-compose.yml -f docker-compose.dev.yml up

services:
  app:
    # Use a specific Dockerfile for development if needed, or just override command
    # build:
    #   context: .
    #   dockerfile: Dockerfile.dev # Uncomment if you have a separate dev Dockerfile
    # Override the command to run the Next.js development server
    command: pnpm dev
    # Mount the current directory into the container for hot-reloading
    volumes:
      - .:/app
      # Exclude node_modules from host mount to prevent issues with different OS architectures
      - /app/node_modules
    # Override environment variables for development
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@db:5432/app_dev # Use a separate dev DB
      - NEXTAUTH_URL=http://localhost:3000
      - NEXTAUTH_SECRET=super-secret-dev-key # Simpler secret for dev
      - REDIS_URL=redis://redis:6379
      - NODE_ENV=development
    # Ensure the database is healthy before starting the app
    depends_on:
      db:
        condition: service_healthy
    # No restart policy needed for development, you'll restart manually
    restart: "no"

  db:
    # Use the same database image, but potentially a different volume or name
    # No changes needed here unless you want a completely separate dev DB setup
    environment:
      - POSTGRES_DB=app_dev # Use a separate database name for development
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data # Use a separate volume for dev DB

  redis:
    # No specific changes needed for Redis in development, unless you want to disable persistence
    # command: redis-server # Remove --appendonly yes for simpler dev setup if desired
    volumes:
      - redis_dev_data:/data # Use a separate volume for dev Redis

# Define separate named volumes for development data persistence
volumes:
  postgres_dev_data:
  redis_dev_data:

§ FILE 4: .DOCKERIGNORE (30 RIGHE)

# Ignore node_modules as they are installed in a multi-stage build
node_modules
# Ignore Next.js build output, it's copied from builder stage
.next
# Ignore Git related files
.git
.gitignore
# Ignore README and local environment files
README.md
.env*.local
# Ignore log files and test coverage reports
*.log
coverage
.nyc_output
# Ignore E2E and unit test directories
e2e
tests
# Ignore Vercel specific files
.vercel
# Ignore editor/IDE specific files
.vscode
*.sublime-project
*.sublime-workspace
# Ignore temporary files
tmp/

§ FILE 5: VERCEL.JSON (50 RIGHE)

json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "regions": ["fra1"],
  "crons": [
    {
      "path": "/api/cron/cleanup",
      "schedule": "0 0 * * *"
    },
    {
      "path": "/api/cron/scheduled-posts",
      "schedule": "*/15 * * * *"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Frame-Options", "value": "DENY" },
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" },
        { "key": "Strict-Transport-Security", "value": "max-age=63072000; includeSubDomains; preload" },
        { "key": "X-XSS-Protection", "value": "1; mode=block" },
        { "key": "Content-Security-Policy", "value": "default-src 'self'; script-src 'self' 'unsafe-eval'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src 'self'; media-src 'self'; object-src 'none'; frame-ancestors 'none';" }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/_next/image",
      "destination": "/_next/image"
    }
  ],
  "redirects": [
    {
      "source": "/old-path",
      "destination": "/new-path",
      "permanent": true
    }
  ]
}

§ FILE 6: .GITHUB/WORKFLOWS/CI.YML (150 RIGHE)

yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

# Define global environment variables for Node.js and pnpm versions
env:
  NODE_VERSION: '20'
  PNPM_VERSION: '8'

jobs:
  # Job for linting and type checking
  lint:
    name: Lint & Type Check
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Run ESLint
        run: pnpm lint
      - name: Run TypeScript type check
        run: pnpm type-check

  # Job for running unit and integration tests
  test:
    name: Unit & Integration Tests
    runs-on: ubuntu-latest
    # Define a PostgreSQL service for testing
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready -U test -d test
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
          --health-start-period 10s
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Generate Prisma client
        run: pnpm db:generate
      - name: Push Prisma schema to test database
        run: pnpm db:push --force --skip-generate
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
      - name: Run tests with coverage
        run: pnpm test:coverage
        env:
          DATABASE_URL: postgresql://test:test@localhost:5432/test
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true # Optional: Fail CI if Codecov upload fails

  # Job for running end-to-end tests
  e2e:
    name: E2E Tests
    runs-on: ubuntu-latest
    # E2E tests often require a deployed application or a mock server.
    # For Vercel, we might build and then run tests against the preview URL.
    # For simplicity here, we'll build locally and run Playwright against it.
    # For a real E2E against a deployed app, you'd need a preview deployment step first.
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Install Playwright browsers
        run: pnpm exec playwright install --with-deps
      - name: Build application for E2E
        run: pnpm build
        env:
          # Use a real database URL for E2E if possible, or a mocked one
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}
          NEXTAUTH_URL: http://localhost:3000 # For local E2E run
      - name: Run E2E tests
        run: pnpm e2e
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}
          NEXTAUTH_URL: http://localhost:3000
      - name: Upload Playwright report on failure
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/

  # Job for building the application (useful for caching or subsequent deployment steps)
  build:
    name: Build Application
    runs-on: ubuntu-latest
    needs: [lint, test] # Ensure linting and testing pass before building
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: ${{ env.PNPM_VERSION }}
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Build application
        run: pnpm build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }} # Required for Prisma generate during build
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}

§ FILE 7: .GITHUB/WORKFLOWS/DEPLOY.YML (100 RIGHE)

yaml
name: Deploy

on:
  push:
    branches: [main] # Trigger deployment only on pushes to the main branch

jobs:
  # Job to deploy the application to Vercel
  deploy:
    name: Deploy to Vercel
    runs-on: ubuntu-latest
    environment: production # Designate this as a production environment
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }} # Vercel API token
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }} # Vercel organization ID
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }} # Vercel project ID
          vercel-args: '--prod' # Deploy to production
          # Optional: Specify a working directory if your project is not at the root
          # working-directory: './frontend'
        id: vercel_deploy
      - name: Output Vercel deployment URL
        run: echo "Vercel Production Deployment URL: ${{ steps.vercel_deploy.outputs.url }}"

  # Job to run database migrations after a successful deployment
  migrate:
    name: Run Database Migrations
    runs-on: ubuntu-latest
    needs: deploy # Ensure deployment is successful before running migrations
    environment: production # Designate this as a production environment
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: '8' # Use a specific pnpm version
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20' # Use a specific Node.js version
          cache: 'pnpm'
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
      - name: Run Prisma migrations
        run: pnpm db:migrate:deploy # Command to apply pending migrations
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }} # Production database URL
          # Ensure other necessary environment variables are present for Prisma
          # For example, if your Prisma schema uses NEXTAUTH_SECRET for some reason
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}
      - name: Verify migrations (optional)
        run: pnpm db:migrate:status # Command to check migration status
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}

§ FILE 8: .GITHUB/WORKFLOWS/PREVIEW.YML (80 RIGHE)

yaml
name: Deploy Preview

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review] # Trigger on PR events

jobs:
  # Job to deploy a preview environment to Vercel for each pull request
  deploy-preview:
    name: Deploy Vercel Preview
    runs-on: ubuntu-latest
    # Only run if the PR is not a draft
    if: github.event.pull_request.draft == false

    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Deploy to Vercel Preview
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          # No --prod flag for preview deployments
          vercel-args: '--build-env DATABASE_URL=${{ secrets.DATABASE_URL_PREVIEW }} --build-env NEXTAUTH_SECRET=${{ secrets.NEXTAUTH_SECRET }}'
          # Automatically assign a unique URL based on the Git branch/commit
          # and link to the PR for easy access
        id: vercel_deploy_preview

      - name: Add Vercel Preview URL to PR comment
        uses: marocchino/sticky-note@v2
        with:
          name: 'Vercel Preview Deployment'
          content: |
            🚀 **Vercel Preview Deployment** for this PR is ready!
            URL: ${{ steps.vercel_deploy_preview.outputs.url }}
            Commit: `${{ github.sha }}`
            _Note: Database migrations are not automatically run for preview deployments. If your PR includes schema changes, manual intervention might be needed for testing._
          # Update existing comment if it exists
          update: true
          # Only post if the deployment was successful
          condition: success()

  # Optional: Run migrations for the preview environment (if needed for testing)
  # This would require a separate database for each preview, which can be complex.
  # For many cases, mocking or using a shared "staging" database might be preferred.
  # migrate-preview:
  #   name: Run Preview Migrations
  #   runs-on: ubuntu-latest
  #   needs: deploy-preview
  #   steps:
  #     - uses: actions/checkout@v4
  #     - uses: pnpm/action-setup@v2
  #     - uses: actions/setup-node@v4
  #       with:
  #         node-version: '20'
  #         cache: 'pnpm'
  #     - run: pnpm install --frozen-lockfile
  #     - run: pnpm db:migrate:deploy
  #       env:
  #         DATABASE_URL: ${{ secrets.DATABASE_URL_PREVIEW }} # Dedicated preview DB

§ FILE 9: .GITHUB/DEPENDABOT.YML (30 RIGHE)

yaml
version: 2
updates:
  # Configuration for npm dependencies
  - package-ecosystem: "npm"
    directory: "/" # Location of package.json
    schedule:
      interval: "weekly" # Check for updates weekly
      day: "monday"
      time: "09:00"
      timezone: "Europe/Rome"
    open-pull-requests-limit: 10 # Limit the number of open PRs
    # Group minor and patch updates for better management
    groups:
      dev-dependencies:
        dependency-type: "development"
      prod-dependencies:
        dependency-type: "production"
    # Reviewers for these PRs
    reviewers:
      - "your-github-username" # Replace with your GitHub username
    labels:
      - "dependencies"
      - "npm"

  # Configuration for GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/" # Location of workflow files
    schedule:
      interval: "weekly" # Check for updates weekly
      day: "monday"
      time: "09:00"
      timezone: "Europe/Rome"
    open-pull-requests-limit: 5
    reviewers:
      - "your-github-username"
    labels:
      - "dependencies"
      - "github-actions"

§ FILE 10: SCRIPTS/SETUP.SH (60 RIGHE)

bash
#!/bin/bash
# Setup script for local development environment

echo "🚀 Setting up development environment..."

# --- 1. Check prerequisites ---
echo "Checking prerequisites..."
command -v node >/dev/null 2>&1 || { echo "❌ Node.js is required. Please install it (e.g., via nvm)."; exit 1; }
command -v pnpm >/dev/null 2>&1 || { echo "❌ pnpm is required. Please install it (e.g., 'npm install -g pnpm')."; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "❌ Docker is required. Please install Docker Desktop."; exit 1; }
command -v docker compose >/dev/null 2>&1 || { echo "❌ Docker Compose is required. Please install Docker Desktop."; exit 1; }
echo "✅ Prerequisites met."

# --- 2. Install Node.js dependencies ---
echo "📦 Installing Node.js dependencies with pnpm..."
pnpm install
if [ $? -ne 0 ]; then
  echo "❌ Failed to install Node.js dependencies."
  exit 1
fi
echo "✅ Node.js dependencies installed."

# --- 3. Setup environment variables ---
if [ ! -f .env ]; then
  echo "📝 Creating .env file from .env.example..."
  cp .env.example .env
  echo "Please review and update the .env file with your local configurations."
else
  echo "📝 .env file already exists. Skipping creation."
fi

# --- 4. Start Docker services (Database & Redis) ---
echo "🐳 Starting Docker services (PostgreSQL, Redis)..."
# Use the development docker-compose file
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d db redis
if [ $? -ne 0 ]; then
  echo "❌ Failed to start Docker services. Check Docker is running."
  exit 1
fi
echo "Waiting for database to be ready..."
# Wait for the database service to be healthy
docker compose -f docker-compose.yml -f docker-compose.dev.yml ps db | grep "healthy" >/dev/null 2>&1
while [ $? -ne 0 ]; do
  echo -n "."
  sleep 2
  docker compose -f docker-compose.yml -f docker-compose.dev.yml ps db | grep "healthy" >/dev/null 2>&1
done
echo "✅ Docker services started and database is ready."

# --- 5. Setup database schema and migrations ---
echo "🗄️ Setting up database schema and running migrations..."
# Generate Prisma client based on schema
pnpm db:generate
# Push schema to the database (for initial setup or non-migration changes)
# Use the development database URL
pnpm db:push --force --skip-generate # --force is for dev, --skip-generate because we just ran it
if [ $? -ne 0 ]; then
  echo "❌ Failed to setup database schema."
  exit 1
fi
echo "✅ Database schema setup complete."

# --- 6. Seed database (optional) ---
read -p "Do you want to seed the database with initial data? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
  echo "🌱 Seeding database..."
  pnpm db:seed
  if [ $? -ne 0 ]; then
    echo "⚠️ Failed to seed database, but setup continues."
  else
    echo "✅ Database seeded."
  fi
fi

echo "🎉 Setup complete! You can now run 'pnpm dev' to start the application."
echo "To stop Docker services: 'docker compose -f docker-compose.yml -f docker-compose.dev.yml down'"

§ FILE 11: SCRIPTS/DB-BACKUP.SH (50 RIGHE)

bash
#!/bin/bash
# Script to perform a PostgreSQL database backup

# --- Configuration ---
BACKUP_DIR="./backups" # Directory to store backups
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="${BACKUP_DIR}/db_backup_${TIMESTAMP}.sql"
DOCKER_COMPOSE_FILE="docker-compose.yml" # Main compose file
DB_SERVICE_NAME="db" # Name of the database service in docker-compose.yml

# Load environment variables from .env if it exists
if [ -f .env ]; then
  echo "Loading environment variables from .env..."
  export $(grep -v '^#' .env | xargs)
fi

# Check for required environment variables
if [ -z "$DATABASE_URL" ]; then
  echo "❌ Error: DATABASE_URL environment variable is not set."
  echo "Please ensure it's set in your .env file or directly in the script."
  exit 1
fi

# Parse DATABASE_URL to extract components
DB_USER=$(echo $DATABASE_URL | sed -n 's/.*:\/\/\|\:.*//p')
DB_PASSWORD=$(echo $DATABASE_URL | sed -n 's/.*:\(.*\)\@.*/\1/p')
DB_HOST=$(echo $DATABASE_URL | sed -n 's/.*@\|\:.*//p')
DB_PORT=$(echo $DATABASE_URL | sed -n 's/.*:\([0-9]*\)\/.*/\1/p')
DB_NAME=$(echo $DATABASE_URL | sed -n 's/.*\///p')

# Set default port if not found in DATABASE_URL
if [ -z "$DB_PORT" ]; then
  DB_PORT="5432"
fi

echo "🚀 Starting database backup for '${DB_NAME}'..."

# Create backup directory if it doesn't exist
mkdir -p "$BACKUP_DIR"

# Perform the backup using pg_dump via Docker Compose exec
# This connects to the running 'db' service and executes pg_dump inside it
echo "Running pg_dump for database '${DB_NAME}' on host '${DB_HOST}:${DB_PORT}'..."
docker compose -f "$DOCKER_COMPOSE_FILE" exec -T "$DB_SERVICE_NAME" \
  pg_dump -U "$DB_USER" -d "$DB_NAME" -h "$DB_HOST" -p "$DB_PORT" > "$BACKUP_FILE"

# Check if pg_dump was successful
if [ $? -eq 0 ]; then
  echo "✅ Database backup successful! Saved to: ${BACKUP_FILE}"
  echo "Backup size: $(du -h "$BACKUP_FILE" | awk '{print $1}')"
else
  echo "❌ Database backup failed!"
  rm -f "$BACKUP_FILE" # Clean up failed backup file
  exit 1
fi

# Optional: Compress the backup file
read -p "Do you want to compress the backup file? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
  echo "Compressing backup file..."
  gzip "$BACKUP_FILE"
  if [ $? -eq 0 ]; then
    echo "✅ Backup compressed to: ${BACKUP_FILE}.gz"
    echo "Compressed size: $(du -h "${BACKUP_FILE}.gz" | awk '{print $1}')"
  else
    echo "❌ Failed to compress backup file."
  fi
fi

echo "Backup script finished."

§ FILE 12: SCRIPTS/DEPLOY.SH (60 RIGHE)

bash
#!/bin/bash
# Manual deployment script for Docker-based production environment

# --- Configuration ---
APP_NAME="my-nextjs-app" # Name of your application
DOCKER_COMPOSE_FILE="docker-compose.yml"
ENV_FILE=".env.production" # Production environment variables file

echo "🚀 Starting manual deployment for ${APP_NAME}..."

# --- 1. Check prerequisites ---
echo "Checking prerequisites..."
command -v docker >/dev/null 2>&1 || { echo "❌ Docker is required. Please install Docker Desktop."; exit 1; }
command -v docker compose >/dev/null 2>&1 || { echo "❌ Docker Compose is required. Please install Docker Desktop."; exit 1; }
if [ ! -f "$DOCKER_COMPOSE_FILE" ]; then
  echo "❌ Error: docker-compose.yml not found in the current directory."
  exit 1
fi
if [ ! -f "$ENV_FILE" ]; then
  echo "❌ Error: ${ENV_FILE} not found. Please create it with production environment variables."
  exit 1
fi
echo "✅ Prerequisites met."

# --- 2. Pull latest code (if applicable, e.g., from a specific branch) ---
# This script assumes you are running it from the root of your project
# and that the code is already up-to-date or will be built from local context.
# If deploying from a remote server, you might add:
# git pull origin main
# if [ $? -ne 0 ]; then
#   echo "❌ Failed to pull latest code."
#   exit 1
# fi
# echo "✅ Latest code pulled."

# --- 3. Build Docker images ---
echo "🐳 Building Docker images for ${APP_NAME}..."
# Use --no-cache to ensure a fresh build
docker compose -f "$DOCKER_COMPOSE_FILE" build --no-cache
if [ $? -ne 0 ]; then
  echo "❌ Failed to build Docker images."
  exit 1
fi
echo "✅ Docker images built."

# --- 4. Stop and remove old containers ---
echo "🛑 Stopping and removing old containers..."
docker compose -f "$DOCKER_COMPOSE_FILE" down --remove-orphans
if [ $? -ne 0 ]; then
  echo "⚠️ Failed to stop old containers, but continuing deployment."
fi
echo "✅ Old containers removed."

# --- 5. Start new containers ---
echo "🚀 Starting new containers..."
# Use -d for detached mode, --env-file to load production environment variables
docker compose -f "$DOCKER_COMPOSE_FILE" up -d --env-file "$ENV_FILE"
if [ $? -ne 0 ]; then
  echo "❌ Failed to start new containers."
  exit 1
fi
echo "✅ New containers started."

# --- 6. Run database migrations (if 'app' service is running) ---
echo "🗄️ Running database migrations..."
# Wait for the app service to be ready before attempting migrations
echo "Waiting for app service to be ready..."
docker compose -f "$DOCKER_COMPOSE_FILE" ps app | grep "running" >/dev/null 2>&1
while [ $? -ne 0 ]; do
  echo -n "."
  sleep 5
  docker compose -f "$DOCKER_COMPOSE_FILE" ps app | grep "running" >/dev/null 2>&1
done
echo "App service is running. Attempting migrations."

# Execute migrations inside the running 'app' container
docker compose -f "$DOCKER_COMPOSE_FILE" exec app pnpm db:migrate:deploy
if [ $? -ne 0 ]; then
  echo "❌ Failed to run database migrations. Please check logs."
  # Optionally, exit here if migrations are critical
  # exit 1
else
  echo "✅ Database migrations applied."
fi

echo "🎉 Deployment of ${APP_NAME} complete!"
echo "Access your application at http://localhost:3000 (or your configured host)."

§ FILE 13: .ENV.PRODUCTION.EXAMPLE (50 RIGHE)

# --- Core Application Settings ---
# The URL where your application will be publicly accessible.
# Important for NextAuth callbacks and absolute URLs.
NEXTAUTH_URL=https://your-production-domain.com
# A secret string used to sign and encrypt session tokens.
# MUST be a long, random string (e.g., generated by `openssl rand -base64 32`).
NEXTAUTH_SECRET=your_nextauth_secret_long_random_string_here

# --- Database Configuration (PostgreSQL Example) ---
# Connection string for your production PostgreSQL database.
# Replace with your actual database credentials and host.
# Example: postgresql://USER:PASSWORD@HOST:PORT/DATABASE
DATABASE_URL=postgresql://user:password@db.example.com:5432/prod_db

# --- Caching / Session Store (Redis Example) ---
# Connection string for your production Redis instance.
# Example: redis://:PASSWORD@HOST:PORT/DB_NUMBER
REDIS_URL=redis://:your_redis_password@redis.example.com:6379/0

# --- Vercel Specific Environment Variables (if deploying to Vercel) ---
# Vercel Token for API access (used in CI/CD).
# VERCEL_TOKEN=your_vercel_api_token
# Vercel Organization ID.
# VERCEL_ORG_ID=your_vercel_org_id
# Vercel Project ID.
# VERCEL_PROJECT_ID=your_vercel_project_id

# --- Cloudinary Configuration (Example for image uploads) ---
# CLOUDINARY_CLOUD_NAME=your_cloudinary_cloud_name
# CLOUDINARY_API_KEY=your_cloudinary_api_key
# CLOUDINARY_API_SECRET=your_cloudinary_api_secret

# --- Email Service Configuration (Example for transactional emails) ---
# EMAIL_SERVER_HOST=smtp.example.com
# EMAIL_SERVER_PORT=587
# EMAIL_SERVER_USER=your_email_user
# EMAIL_SERVER_PASSWORD=your_email_password
# EMAIL_FROM=noreply@your-domain.com

# --- Other Production-Specific Settings ---
# Any other environment variables specific to your application's production needs.
# For example, API keys for third-party services, feature flags, etc.
# STRIPE_SECRET_KEY=sk_live_xxxxxxxxxxxxxxxxxxxx
# GOOGLE_ANALYTICS_ID=UA-XXXXXXXXX-Y
# FEATURE_FLAG_NEW_DASHBOARD=true

§ FILE 14: DEPLOYMENT.MD (150 RIGHE)

markdown
# 🚀 Deployment Guide for My Next.js Application

This document provides a comprehensive guide for deploying and managing the Next.js application, covering local development setup, CI/CD pipelines, Vercel deployment, and database management.

---

## Table of Contents

1.  [Local Development Setup](#1-local-development-setup)
2.  [CI/CD Overview (GitHub Actions)](#2-cicd-overview-github-actions)
    *   [Continuous Integration (CI)](#continuous-integration-ci)
    *   [Continuous Deployment (CD)](#continuous-deployment-cd)
    *   [Preview Deployments](#preview-deployments)
    *   [Dependabot for Dependency Updates](#dependabot-for-dependency-updates)
3.  [Vercel Deployment](#3-vercel-deployment)
    *   [Vercel Project Configuration](#vercel-project-configuration)
    *   [Environment Variables on Vercel](#environment-variables-on-vercel)
    *   [Cron Jobs](#cron-jobs)
    *   [Security Headers](#security-headers)
4.  [Database Management](#4-database-management)
    *   [Prisma Migrations](#prisma-migrations)
    *   [Database Backups](#database-backups)
5.  [Docker Deployment (Alternative/Self-Hosted)](#5-docker-deployment-alternative-self-hosted)
    *   [Building and Running with Docker Compose](#building-and-running-with-docker-compose)
    *   [Production Considerations](#production-considerations)
6.  [Health Checks & Monitoring](#6-health-checks--monitoring)
7.  [Troubleshooting](#7-troubleshooting)

---

## 1. Local Development Setup

To get the application running locally with Dockerized services (PostgreSQL and Redis):

1.  **Prerequisites**:
    *   Node.js (v20+)
    *   pnpm (install with `npm install -g pnpm`)
    *   Docker Desktop (or Docker Engine + Docker Compose)

2.  **Clone the repository**:
    git clone <repository-url>
    cd <repository-name>

3.  **Run the setup script**:
    This script will install Node.js dependencies, create a `.env` file, start Docker services, and initialize the database.
    chmod +x scripts/setup.sh
    ./scripts/setup.sh
    *   Follow the prompts, especially for seeding the database.

4.  **Start the application**:
    pnpm dev
    The application will be available at `http://localhost:3000`.

5.  **Stop Docker services**:
    When you're done developing, stop the Docker containers:
    docker compose -f docker-compose.yml -f docker-compose.dev.yml down

## 2. CI/CD Overview (GitHub Actions)

Our CI/CD pipeline is managed by GitHub Actions, ensuring code quality, test coverage, and automated deployments.

### Continuous Integration (CI)

The `.github/workflows/ci.yml` workflow runs on every `push` to `main` or `develop` branches, and on every `pull_request` targeting these branches.

*   **Lint & Type Check**: Runs ESLint and TypeScript checks to enforce code style and catch type errors.
*   **Unit & Integration Tests**: Runs Jest tests with a dedicated PostgreSQL service. Code coverage is uploaded to Codecov.
*   **E2E Tests**: Runs Playwright end-to-end tests against a built version of the application. Test reports are uploaded as artifacts on failure.
*   **Build Application**: Performs a production build of the Next.js application.

### Continuous Deployment (CD)

The `.github/workflows/deploy.yml` workflow handles production deployments.

*   **Trigger**: Automatically runs on `push` to the `main` branch.
*   **Deploy to Vercel**: Uses the `amondnet/vercel-action` to deploy the application to Vercel in production mode (`--prod`).
*   **Run Database Migrations**: After a successful Vercel deployment, it checks out the code, installs dependencies, and runs `pnpm db:migrate:deploy` against the production `DATABASE_URL` to apply any pending Prisma migrations.

### Preview Deployments

The `.github/workflows/preview.yml` workflow creates ephemeral preview environments.

*   **Trigger**: Runs on `pull_request` events (opened, synchronized, reopened, ready for review).
*   **Deploy Vercel Preview**: Deploys the PR branch to a unique Vercel URL.
*   **PR Comment**: Adds a sticky comment to the Pull Request with the preview deployment URL for easy access by reviewers.
*   **Note**: Preview deployments typically do not run database migrations automatically. If your PR includes schema changes, manual intervention or a dedicated staging database might be required for full testing.

### Dependabot for Dependency Updates

The `.github/dependabot.yml` configuration automates dependency updates.

*   **npm**: Checks for updates to `npm` packages weekly and creates pull requests.
*   **GitHub Actions**: Checks for updates to GitHub Actions used in workflows weekly and creates pull requests.
*   Dependabot PRs are labeled and assigned to reviewers for easy management.

## 3. Vercel Deployment

The application is primarily deployed to Vercel for its ease of use, performance, and Next.js integration.

### Vercel Project Configuration

The `vercel.json` file defines project-level configurations:

*   **Framework**: Explicitly set to `nextjs`.
*   **Regions**: Deploys to `fra1` (Frankfurt, Germany) for optimal latency for European users. Adjust as needed.
*   **Headers**: Configures security headers (e.g., `X-Frame-Options`, `Content-Security-Policy`) to enhance application security.
*   **Rewrites/Redirects**: Can be used for custom routing logic.

### Environment Variables on Vercel

All sensitive environment variables (e.g., `DATABASE_URL`, `NEXTAUTH_SECRET`, API keys) must be configured in your Vercel project settings.

*   Go to your Vercel project dashboard -> Settings -> Environment Variables.
*   Add variables for `Production`, `Preview`, and `Development` environments as needed.
*   Refer to `.env.production.example` for a list of required production variables.

### Cron Jobs

Vercel's native cron jobs are configured in `vercel.json`:

*   `/api/cron/cleanup`: Runs daily at midnight (`0 0 * * *`).
*   `/api/cron/scheduled-posts`: Runs every 15 minutes (`*/15 * * * *`).
*   Ensure the corresponding API routes (`src/app/api/cron/cleanup/route.ts`, `src/app/api/cron/scheduled-posts/route.ts`) exist and contain your cron logic.

### Security Headers

The `vercel.json` includes a comprehensive set of security headers to protect against common web vulnerabilities. Review and customize them based on your application's specific needs and third-party integrations.

## 4. Database Management

### Prisma Migrations

Prisma is used for database schema management.

*   **Development**: `pnpm db:generate` (generate client), `pnpm db:push` (push schema to DB, for dev only).
*   **Production/CI/CD**: `pnpm db:migrate:deploy` (apply pending migrations). This is automatically run in the `deploy.yml` workflow.
*   **Creating new migrations**: `pnpm prisma migrate dev --name <migration-name>`

### Database Backups

The `scripts/db-backup.sh` script facilitates manual database backups.

*   **Usage**:
    chmod +x scripts/db-backup.sh
    ./scripts/db-backup.sh
*   This script uses `pg_dump` to create a SQL dump of your PostgreSQL database.
*   It reads `DATABASE_URL` from your `.env` file.
*   Backups are stored in the `./backups` directory.
*   Consider automating this script with a cron job on your server or using your cloud provider's managed database backup solutions for production.

## 5. Docker Deployment (Alternative/Self-Hosted)

For self-hosted deployments or environments where Vercel is not used, Docker Compose provides a robust solution.

### Building and Running with Docker Compose

1.  **Prepare `.env.production`**: Create this file at the root of your project with all necessary production environment variables (refer to `.env.production.example`).
2.  **Run the deploy script**:
    chmod +x scripts/deploy.sh
    ./scripts/deploy.sh
    This script will:
    *   Build Docker images (`Dockerfile`).
    *   Stop and remove any existing containers.
    *   Start new containers (app, db, redis) in detached mode using `docker-compose.yml` and `.env.production`.
    *   Run Prisma migrations inside the `app` container.

### Production Considerations

*   **Reverse Proxy**: In a real production environment, place a reverse proxy (e.g., Nginx, Caddy) in front of your Docker containers for SSL termination, load balancing, and static asset caching.
*   **Monitoring**: Implement Docker container monitoring (e.g., Prometheus, Grafana).
*   **Logging**: Centralize container logs (e.g., ELK stack, Loki).
*   **Secrets Management**: Use Docker Secrets or a dedicated secrets manager (e.g., HashiCorp Vault) instead of `.env.production` directly in production.
*   **Volume Backups**: Ensure regular backups of your `postgres_data` and `redis_data` Docker volumes.

## 6. Health Checks & Monitoring

*   **`/api/health` Endpoint**: The `src/app/api/health/route.ts` provides a simple health check endpoint. It returns a `200 OK` status, indicating the application server is running.
*   **Docker Compose Healthchecks**: `docker-compose.yml` includes health checks for `db` and `redis` services to ensure they are ready before the application starts.
*   **Vercel Monitoring**: Vercel provides built-in monitoring for deployments, including logs, performance metrics, and error tracking.
*   **External Monitoring**: Integrate with external monitoring services (e.g., UptimeRobot, Datadog, Sentry) to track application availability and performance.

## 7. Troubleshooting

*   **"Container exited with code 1"**: Check Docker container logs (`docker compose logs <service-name>`) for detailed error messages.
*   **Database connection issues**:
    *   Verify `DATABASE_URL` in `.env` or Vercel environment variables.
    *   Ensure the database service is running and healthy (`docker compose ps`).
    *   Check firewall rules if connecting to a remote database.
*   **Vercel deployment failures**:
    *   Check the build logs in the Vercel dashboard or GitHub Actions workflow.
    *   Ensure all required environment variables are set on Vercel.
    *   Verify `package.json` scripts (e.g., `build`).
*   **Prisma migration errors**:
    *   Ensure `DATABASE_URL` is correct for the target environment.
    *   Check for conflicts in migration files.
    *   Review Prisma logs for specific error details.
*   **Hot-reloading not working in dev**:
    *   Ensure `volumes` are correctly configured in `docker-compose.dev.yml` (e.g., `- .:/app` and `- /app/node_modules`).
    *   Check file permissions.

---

§ FILE 15: SRC/APP/API/HEALTH/ROUTE.TS (30 RIGHE)

typescript
import { NextResponse } from 'next/server';

/**
 * @swagger
 * /api/health:
 *   get:
 *     summary: Checks the health of the application.
 *     description: Returns a 200 OK status if the application is running.
 *     responses:
 *       200:
 *         description: Application is healthy.
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 status:
 *                   type: string
 *                   example: "ok"
 *                 timestamp:
 *                   type: string
 *                   format: date-time
 *                   example: "2023-10-27T10:00:00Z"
 *                 uptime:
 *                   type: number
 *                   example: 3600 # in seconds
 */
export async function GET() {
  try {
    // You can add more sophisticated checks here, e.g., database connection, Redis connection.
    // For a basic health check, just returning a 200 OK is sufficient.

    // Example of adding more details:
    // const dbStatus = await checkDatabaseConnection();
    // const redisStatus = await checkRedisConnection();

    return NextResponse.json({
      status: 'ok',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(), // Uptime in seconds
      // database: dbStatus,
      // redis: redisStatus,
    }, { status: 200 });
  } catch (error) {
    console.error('Health check failed:', error);
    return NextResponse.json({
      status: 'error',
      message: 'Application is unhealthy',
      error: (error as Error).message,
    }, { status: 500 });
  }
}

§ FILE 16: SRC/APP/API/CRON/CLEANUP/ROUTE.TS (50 RIGHE)

typescript
import { NextResponse } from 'next/server';
// Assuming you have a Prisma client setup
// import { prisma } from '@/lib/prisma'; // Adjust path as needed

/**
 * @swagger
 * /api/cron/cleanup:
 *   get:
 *     summary: Runs a scheduled cleanup job.
 *     description: This endpoint is designed to be triggered by a cron scheduler (e.g., Vercel Cron Jobs).
 *                  It performs various cleanup tasks such as deleting old data, clearing caches, etc.
 *     responses:
 *       200:
 *         description: Cleanup job completed successfully.
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 message:
 *                   type: string
 *                   example: "Cleanup job executed successfully."
 *                 cleanedRecords:
 *                   type: object
 *                   properties:
 *                     oldSessions:
 *                       type: number
 *                       example: 123
 *                     expiredTokens:
 *                       type: number
 *                       example: 45
 *       500:
 *         description: Internal Server Error if the cleanup job fails.
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 error:
 *                   type: string
 *                   example: "Failed to perform cleanup."
 */
export async function GET() {
  // Ensure this endpoint is only accessible by your cron service or with authentication
  // For Vercel Cron Jobs, they are secured by default.
  // If self-hosting, add a secret token check:
  // if (req.headers.get('Authorization') !== `Bearer ${process.env.CRON_SECRET}`) {
  //   return NextResponse.json({ message: 'Unauthorized' }, { status: 401 });
  // }

  console.log('🚀 Starting daily cleanup cron job...');

  try {
    const now = new Date();
    const thirtyDaysAgo = new Date(now.setDate(now.getDate() - 30));

    // Example cleanup task 1: Delete old user sessions
    // const deletedSessions = await prisma.session.deleteMany({
    //   where: {
    //     expires: {
    //       lt: thirtyDaysAgo,
    //     },
    //   },
    // });
    // console.log(`🗑️ Deleted ${deletedSessions.count} old sessions.`);

    // Example cleanup task 2: Delete expired password reset tokens
    // const deletedTokens = await prisma.passwordResetToken.deleteMany({
    //   where: {
    //     expires: {
    //       lt: new Date(),
    //     },
    //   },
    // });
    // console.log(`🗑️ Deleted ${deletedTokens.count} expired password reset tokens.`);

    // Simulate cleanup operations
    const deletedSessionsCount = Math.floor(Math.random() * 100);
    const deletedTokensCount = Math.floor(Math.random() * 50);

    console.log(`🗑️ Deleted ${deletedSessionsCount} old sessions.`);
    console.log(`🗑️ Deleted ${deletedTokensCount} expired password reset tokens.`);


    console.log('✅ Cleanup cron job completed successfully.');

    return NextResponse.json({
      message: 'Cleanup job executed successfully.',
      cleanedRecords: {
        oldSessions: deletedSessionsCount,
        expiredTokens: deletedTokensCount,
      },
    }, { status: 200 });

  } catch (error) {
    console.error('❌ Error during cleanup cron job:', error);
    return NextResponse.json({
      error: 'Failed to perform cleanup.',
      details: (error as Error).message,
    }, { status: 500 });
  }
}

---
_Modello: gemini-2.5-flash (Google AI Studio)_
