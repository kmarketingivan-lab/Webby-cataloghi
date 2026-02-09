# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO SICUREZZA v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
# GUIDA COMPLETA ALLA SICUREZZA APPLICATIVA
# Basato su OWASP Top 10:2025 e best practices aggiornate
#
# Data creazione: 2026-01-26
# Ultima modifica: 2026-01-26
#
# ═══════════════════════════════════════════════════════════════════════════════

"""
INDICE:
═══════════════════════════════════════════════════════════════════════════════
1.  Security Fundamentals
2.  OWASP Top 10:2025
3.  Security Headers
4.  Authentication & Authorization
5.  JWT Security
6.  Session Management
7.  Input Validation & Output Encoding
8.  API Security
9.  Secrets Management
10. Database Security
11. File Upload Security
12. Rate Limiting & DDoS Protection
13. Security Audit Checklist
14. Incident Response
═══════════════════════════════════════════════════════════════════════════════
"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 1: SECURITY FUNDAMENTALS
# ═══════════════════════════════════════════════════════════════════════════════

SECURITY_FUNDAMENTALS = """

## 1.1 Principi di Sicurezza

### Defense in Depth (Difesa in Profondità)
Non affidarti mai a un singolo controllo di sicurezza. Implementa multiple
layer di protezione così che se uno fallisce, gli altri continuano a proteggere.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DEFENSE IN DEPTH                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│   ┌─────────────────────────────────────────────────────────────────┐       │
│   │  PERIMETER                                                      │       │
│   │  - Firewall, WAF, DDoS protection                              │       │
│   │  ┌─────────────────────────────────────────────────────────┐   │       │
│   │  │  NETWORK                                                 │   │       │
│   │  │  - TLS/HTTPS, VPN, Network segmentation                 │   │       │
│   │  │  ┌─────────────────────────────────────────────────┐    │   │       │
│   │  │  │  APPLICATION                                     │    │   │       │
│   │  │  │  - Auth, Input validation, Security headers     │    │   │       │
│   │  │  │  ┌─────────────────────────────────────────┐    │    │   │       │
│   │  │  │  │  DATA                                    │    │    │   │       │
│   │  │  │  │  - Encryption, Access control, Backup   │    │    │   │       │
│   │  │  │  └─────────────────────────────────────────┘    │    │   │       │
│   │  │  └─────────────────────────────────────────────────┘    │   │       │
│   │  └─────────────────────────────────────────────────────────┘   │       │
│   └─────────────────────────────────────────────────────────────────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Principle of Least Privilege
Ogni utente, processo o sistema deve avere solo i permessi minimi necessari
per svolgere la propria funzione.

```typescript
// ❌ SBAGLIATO: Permessi troppo ampi
const adminUser = {
  role: 'admin',
  permissions: ['*'] // Accesso a tutto
};

// ✅ CORRETTO: Permessi specifici
const editorUser = {
  role: 'editor',
  permissions: [
    'posts:read',
    'posts:create',
    'posts:update:own',  // Solo i propri post
    'media:upload',
    'comments:moderate'
  ]
};
```

### Fail Secure
In caso di errore o eccezione, il sistema deve fallire in uno stato sicuro,
negando l'accesso piuttosto che permetterlo.

```typescript
// ❌ SBAGLIATO: Fail open
async function checkPermission(userId: string, resource: string): Promise<boolean> {
  try {
    const user = await getUser(userId);
    return user.permissions.includes(resource);
  } catch (error) {
    console.error('Permission check failed:', error);
    return true; // ⚠️ PERICOLOSO: permette accesso in caso di errore
  }
}

// ✅ CORRETTO: Fail secure
async function checkPermission(userId: string, resource: string): Promise<boolean> {
  try {
    const user = await getUser(userId);
    return user.permissions.includes(resource);
  } catch (error) {
    console.error('Permission check failed:', error);
    return false; // ✅ Nega accesso in caso di errore
  }
}
```

### Security by Design
La sicurezza deve essere considerata fin dalle prime fasi di design,
non aggiunta come afterthought.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ SECURITY BY DESIGN CHECKLIST                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ □ Threat Modeling completato prima dello sviluppo                          │
│ □ Requisiti di sicurezza definiti insieme ai requisiti funzionali          │
│ □ Review di sicurezza del design architetturale                            │
│ □ Security patterns identificati e documentati                              │
│ □ Test di sicurezza pianificati nel ciclo di sviluppo                      │
│ □ Compliance requirements identificati (GDPR, HIPAA, PCI-DSS)              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.2 Threat Modeling (STRIDE)

STRIDE è un framework per identificare e classificare le minacce:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ STRIDE THREAT MODEL                                                         │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ S - Spoofing Identity                                                       │
│     "Qualcuno può fingersi un altro utente?"                               │
│     → Mitigazione: Strong authentication, MFA                               │
│                                                                             │
│ T - Tampering with Data                                                     │
│     "Qualcuno può modificare dati senza autorizzazione?"                   │
│     → Mitigazione: Input validation, checksums, digital signatures          │
│                                                                             │
│ R - Repudiation                                                             │
│     "Un utente può negare di aver fatto un'azione?"                        │
│     → Mitigazione: Audit logging, digital signatures                        │
│                                                                             │
│ I - Information Disclosure                                                  │
│     "Dati sensibili possono essere esposti?"                               │
│     → Mitigazione: Encryption, access control, data masking                 │
│                                                                             │
│ D - Denial of Service                                                       │
│     "Il sistema può essere reso non disponibile?"                          │
│     → Mitigazione: Rate limiting, resource quotas, redundancy               │
│                                                                             │
│ E - Elevation of Privilege                                                  │
│     "Un utente può ottenere permessi non autorizzati?"                     │
│     → Mitigazione: RBAC, least privilege, input validation                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 1.3 CIA Triad

```
                    CONFIDENTIALITY
                    (Riservatezza)
                         ▲
                        /│\\
                       / │ \\
                      /  │  \\
                     /   │   \\
                    /    │    \\
                   /     │     \\
                  /      │      \\
                 /       │       \\
                /        │        \\
               ▼─────────┴─────────▼
        INTEGRITY              AVAILABILITY
       (Integrità)            (Disponibilità)

CONFIDENTIALITY: Solo utenti autorizzati possono accedere ai dati
→ Encryption, Access Control, Authentication

INTEGRITY: I dati non possono essere modificati senza autorizzazione
→ Hashing, Digital Signatures, Input Validation

AVAILABILITY: Il sistema deve essere accessibile quando necessario
→ Redundancy, DDoS Protection, Backups
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 2: OWASP TOP 10:2025
# ═══════════════════════════════════════════════════════════════════════════════

OWASP_TOP_10_2025 = """

## 2.1 Overview OWASP Top 10:2025

L'OWASP Top 10 è lo standard di riferimento per i rischi di sicurezza
delle applicazioni web. La versione 2025 include:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ OWASP TOP 10:2025                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ A01:2025 - Broken Access Control                    ████████████████ #1    │
│ A02:2025 - Security Misconfiguration                ██████████████   #2    │
│ A03:2025 - Software Supply Chain Failures           █████████████    #3    │
│ A04:2025 - Insecure Design                          ████████████     #4    │
│ A05:2025 - Cryptographic Failures                   ███████████      #5    │
│ A06:2025 - Injection                                ██████████       #6    │
│ A07:2025 - Identification & Authentication Failures █████████        #7    │
│ A08:2025 - Software & Data Integrity Failures       ████████         #8    │
│ A09:2025 - Security Logging & Monitoring Failures   ███████          #9    │
│ A10:2025 - Server-Side Request Forgery (SSRF)       ██████           #10   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.2 A01:2025 - Broken Access Control

Il rischio #1: controlli di accesso non implementati correttamente.

### Vulnerabilità Comuni

```typescript
// ❌ VULNERABILE: IDOR (Insecure Direct Object Reference)
app.get('/api/users/:id', async (req, res) => {
  const user = await db.users.findById(req.params.id);
  res.json(user); // Chiunque può vedere qualsiasi utente!
});

// ✅ SICURO: Verifica ownership
app.get('/api/users/:id', authenticate, async (req, res) => {
  const userId = req.params.id;
  
  // Solo admin può vedere altri utenti
  if (userId !== req.user.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  const user = await db.users.findById(userId);
  res.json(user);
});
```

```typescript
// ❌ VULNERABILE: Path Traversal
app.get('/api/files/:filename', async (req, res) => {
  const filepath = `/uploads/${req.params.filename}`;
  res.sendFile(filepath); // ../../etc/passwd può essere letto!
});

// ✅ SICURO: Validazione e sanitizzazione path
import path from 'path';

app.get('/api/files/:filename', authenticate, async (req, res) => {
  const filename = path.basename(req.params.filename); // Rimuove path traversal
  const filepath = path.join('/uploads', filename);
  
  // Verifica che il file sia nella directory consentita
  if (!filepath.startsWith('/uploads/')) {
    return res.status(400).json({ error: 'Invalid filename' });
  }
  
  // Verifica ownership del file
  const file = await db.files.findOne({ name: filename, userId: req.user.id });
  if (!file) {
    return res.status(404).json({ error: 'File not found' });
  }
  
  res.sendFile(filepath);
});
```

### Checklist Prevenzione

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ACCESS CONTROL CHECKLIST                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ □ Nega accesso per default (whitelist, non blacklist)                      │
│ □ Implementa controlli di accesso server-side (mai solo client-side)       │
│ □ Verifica ownership per ogni risorsa acceduta                             │
│ □ Usa ID non predicibili (UUID v4, non incrementali)                       │
│ □ Implementa RBAC (Role-Based Access Control)                              │
│ □ Log tutti i fallimenti di access control                                 │
│ □ Rate limit API per prevenire enumeration                                 │
│ □ Disabilita directory listing sul web server                              │
│ □ Rimuovi file sensibili dalla root web (.git, .env)                       │
│ □ Testa regolarmente con tool automatici                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.3 A02:2025 - Security Misconfiguration

Configurazioni di sicurezza mancanti o errate.

### Configurazioni Critiche

```typescript
// next.config.js - Security headers
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  },
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=()'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

```yaml
# docker-compose.yml - Configurazione sicura
version: '3.8'
services:
  app:
    image: myapp:latest
    # ❌ MAI fare questo:
    # user: root
    # privileged: true
    
    # ✅ Configurazione sicura:
    user: "1000:1000"  # Non-root user
    read_only: true    # Filesystem read-only
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    tmpfs:
      - /tmp
    environment:
      - NODE_ENV=production
```

### Checklist Configurazione Sicura

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ SECURITY CONFIGURATION CHECKLIST                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ SERVER:                                                                     │
│ □ Rimuovi/disabilita feature non necessarie                                │
│ □ Disabilita directory listing                                             │
│ □ Rimuovi file di default (README, CHANGELOG)                              │
│ □ Configura error pages custom (no stack traces)                           │
│ □ Imposta timeout appropriati                                              │
│                                                                             │
│ DATABASE:                                                                   │
│ □ Cambia credenziali di default                                            │
│ □ Disabilita accesso remoto se non necessario                              │
│ □ Usa utenti con permessi minimi                                           │
│ □ Abilita audit logging                                                    │
│                                                                             │
│ CLOUD/CONTAINER:                                                            │
│ □ Principio least privilege per IAM roles                                  │
│ □ Container non-root                                                       │
│ □ Secrets non hardcoded                                                    │
│ □ Network policies restrittive                                             │
│                                                                             │
│ APPLICATION:                                                                │
│ □ Debug mode disabilitato in produzione                                    │
│ □ Verbose error messages disabilitati                                      │
│ □ Security headers configurati                                             │
│ □ CORS configurato correttamente                                           │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 2.4 A03:2025 - Software Supply Chain Failures

Vulnerabilità introdotte tramite dipendenze o build pipeline.

### Protezioni Supply Chain

```json
// package.json - Lock versions e audit
{
  "name": "secure-app",
  "scripts": {
    "preinstall": "npx npm-audit-resolver",
    "audit": "npm audit --audit-level=high",
    "audit:fix": "npm audit fix",
    "check-deps": "npx depcheck",
    "outdated": "npm outdated"
  },
  "overrides": {
    // Forza versioni sicure di dipendenze transitive
    "lodash": "^4.17.21",
    "minimist": "^1.2.6"
  }
}
```

```yaml
# .github/workflows/security.yml
name: Security Checks

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *'  # Daily scan

jobs:
  dependency-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run npm audit
        run: npm audit --audit-level=high
        
      - name: Check for known vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

## 2.5 A06:2025 - Injection

SQL, NoSQL, OS, LDAP injection e simili.

### SQL Injection Prevention

```typescript
// ❌ VULNERABILE: Concatenazione stringhe
async function getUser(email: string) {
  const query = `SELECT * FROM users WHERE email = '${email}'`;
  return db.query(query);
  // Input: ' OR '1'='1' -- 
  // Query risultante: SELECT * FROM users WHERE email = '' OR '1'='1' --'
}

// ✅ SICURO: Prepared statements / Parameterized queries
async function getUser(email: string) {
  const query = 'SELECT * FROM users WHERE email = $1';
  return db.query(query, [email]);
}

// ✅ SICURO: Con ORM (Prisma)
async function getUser(email: string) {
  return prisma.user.findUnique({
    where: { email }  // Prisma usa parameterized queries internamente
  });
}
```

### NoSQL Injection Prevention

```typescript
// ❌ VULNERABILE: Query object from user input
app.post('/api/login', async (req, res) => {
  const { username, password } = req.body;
  // Se password = { "$ne": "" }, bypassa l'autenticazione!
  const user = await db.users.findOne({ username, password });
  // ...
});

// ✅ SICURO: Validazione e sanitizzazione
import { z } from 'zod';

const loginSchema = z.object({
  username: z.string().min(3).max(50),
  password: z.string().min(8).max(100)
});

app.post('/api/login', async (req, res) => {
  const result = loginSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const { username, password } = result.data;
  
  // Password è garantita essere una stringa, non un oggetto
  const user = await db.users.findOne({ 
    username: String(username)
  });
  
  if (user && await bcrypt.compare(password, user.passwordHash)) {
    // Login success
  }
});
```

### Command Injection Prevention

```typescript
// ❌ VULNERABILE: exec con user input
import { exec } from 'child_process';

app.get('/api/ping', (req, res) => {
  const host = req.query.host;
  exec(`ping -c 4 ${host}`, (error, stdout) => {
    res.send(stdout);
  });
  // Input: google.com; rm -rf /
  // Comando: ping -c 4 google.com; rm -rf /
});

// ✅ SICURO: Usa execFile con argomenti separati
import { execFile } from 'child_process';

app.get('/api/ping', (req, res) => {
  const host = req.query.host;
  
  // Validazione whitelist
  const validHostRegex = /^[a-zA-Z0-9.-]+$/;
  if (!validHostRegex.test(host)) {
    return res.status(400).json({ error: 'Invalid host' });
  }
  
  // execFile non passa attraverso shell
  execFile('ping', ['-c', '4', host], (error, stdout) => {
    res.send(stdout);
  });
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 3: SECURITY HEADERS
# ═══════════════════════════════════════════════════════════════════════════════

SECURITY_HEADERS = """

## 3.1 Header Essenziali

### Strict-Transport-Security (HSTS)

Forza il browser a usare sempre HTTPS per il dominio.

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

Parametri:
- max-age=63072000    → 2 anni (minimo raccomandato: 6 mesi / 15768000)
- includeSubDomains   → Applica a tutti i sottodomini
- preload             → Permette inclusione nella HSTS preload list dei browser
```

```typescript
// Next.js middleware
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // HSTS - Solo in produzione con HTTPS
  if (process.env.NODE_ENV === 'production') {
    response.headers.set(
      'Strict-Transport-Security',
      'max-age=63072000; includeSubDomains; preload'
    );
  }
  
  return response;
}
```

### Content-Security-Policy (CSP)

Controlla quali risorse possono essere caricate sulla pagina.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ CSP DIRECTIVES REFERENCE                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ default-src    → Fallback per tutte le altre direttive                     │
│ script-src     → Sorgenti JavaScript consentite                            │
│ style-src      → Sorgenti CSS consentite                                   │
│ img-src        → Sorgenti immagini consentite                              │
│ font-src       → Sorgenti font consentite                                  │
│ connect-src    → URL per fetch/XHR/WebSocket                               │
│ frame-src      → Sorgenti per iframe consentite                            │
│ frame-ancestors→ Chi può embeddare questa pagina (anti-clickjacking)       │
│ form-action    → URL dove i form possono fare submit                       │
│ base-uri       → Valori consentiti per <base>                              │
│ object-src     → Sorgenti per <object>, <embed>, <applet>                  │
│ upgrade-insecure-requests → Upgrade HTTP a HTTPS automatico                │
│                                                                             │
│ VALUES:                                                                     │
│ 'self'         → Stesso origin                                             │
│ 'none'         → Blocca tutto                                              │
│ 'unsafe-inline'→ Permette inline (⚠️ EVITARE se possibile)                 │
│ 'unsafe-eval'  → Permette eval() (⚠️ EVITARE)                              │
│ 'strict-dynamic'→ Trust propagato via nonce                                │
│ 'nonce-xxx'    → Script con nonce specifico                                │
│ 'sha256-xxx'   → Script con hash specifico                                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```typescript
// CSP Progressivamente più restrittivo

// LIVELLO 1: Base (sito statico semplice)
const cspLevel1 = `
  default-src 'self';
  script-src 'self';
  style-src 'self';
  img-src 'self' data:;
  font-src 'self';
  object-src 'none';
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
`.replace(/\s+/g, ' ').trim();

// LIVELLO 2: Con CDN esterni (Analytics, Fonts)
const cspLevel2 = `
  default-src 'self';
  script-src 'self' https://www.googletagmanager.com https://www.google-analytics.com;
  style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
  img-src 'self' data: https://www.google-analytics.com;
  font-src 'self' https://fonts.gstatic.com;
  connect-src 'self' https://www.google-analytics.com;
  object-src 'none';
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
  upgrade-insecure-requests;
`.replace(/\s+/g, ' ').trim();

// LIVELLO 3: Strict con nonce (raccomandato)
function generateCSPWithNonce(nonce: string) {
  return `
    default-src 'self';
    script-src 'self' 'nonce-${nonce}' 'strict-dynamic';
    style-src 'self' 'nonce-${nonce}';
    img-src 'self' data: https:;
    font-src 'self';
    connect-src 'self';
    object-src 'none';
    frame-ancestors 'none';
    base-uri 'self';
    form-action 'self';
    upgrade-insecure-requests;
  `.replace(/\s+/g, ' ').trim();
}

// Uso in Next.js middleware
import { randomBytes } from 'crypto';

export function middleware(request: NextRequest) {
  const nonce = randomBytes(16).toString('base64');
  const csp = generateCSPWithNonce(nonce);
  
  const response = NextResponse.next();
  response.headers.set('Content-Security-Policy', csp);
  response.headers.set('x-nonce', nonce); // Per uso nel layout
  
  return response;
}
```

### X-Frame-Options (Legacy)

Previene clickjacking. Usa `frame-ancestors` in CSP se possibile.

```
X-Frame-Options: DENY              → Non permettere framing
X-Frame-Options: SAMEORIGIN        → Solo stesso origin può frammare
```

### X-Content-Type-Options

Previene MIME type sniffing.

```
X-Content-Type-Options: nosniff

→ Il browser rispetterà il Content-Type dichiarato
→ Previene attacchi dove file malevoli vengono interpretati come script
```

### Referrer-Policy

Controlla quante informazioni referrer vengono inviate.

```
Referrer-Policy: strict-origin-when-cross-origin   → Raccomandato
Referrer-Policy: no-referrer                        → Massima privacy

┌─────────────────────────────────────────────────────────────────────────────┐
│ REFERRER POLICY OPTIONS                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ no-referrer                    → Mai inviare referrer                       │
│ no-referrer-when-downgrade     → Non inviare su HTTP se origin è HTTPS     │
│ origin                         → Invia solo origin (no path)               │
│ origin-when-cross-origin       → Full URL same-origin, solo origin cross   │
│ same-origin                    → Invia solo per same-origin                │
│ strict-origin                  → Come origin, ma no downgrade              │
│ strict-origin-when-cross-origin→ ✅ RACCOMANDATO                           │
│ unsafe-url                     → Sempre full URL (⚠️ non usare)            │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Permissions-Policy (ex Feature-Policy)

Controlla quali API browser possono essere usate.

```
Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=()

→ () = disabilitato per tutti
→ (self) = solo stesso origin
→ ("https://example.com") = solo domini specifici
```

## 3.2 Configurazione Completa Headers

```typescript
// lib/security-headers.ts

export const securityHeaders = [
  // HTTPS enforcement
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  
  // Anti-clickjacking
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  
  // Prevent MIME sniffing
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  
  // Referrer control
  {
    key: 'Referrer-Policy',
    value: 'strict-origin-when-cross-origin'
  },
  
  // Feature restrictions
  {
    key: 'Permissions-Policy',
    value: 'camera=(), microphone=(), geolocation=(), payment=(self)'
  },
  
  // Cross-Origin policies
  {
    key: 'Cross-Origin-Opener-Policy',
    value: 'same-origin'
  },
  {
    key: 'Cross-Origin-Resource-Policy',
    value: 'same-origin'
  },
  {
    key: 'Cross-Origin-Embedder-Policy',
    value: 'require-corp'
  }
];

// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

```typescript
// Express.js con Helmet
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'strict-dynamic'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      frameAncestors: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  hsts: {
    maxAge: 63072000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: {
    policy: 'strict-origin-when-cross-origin',
  },
}));
```

## 3.3 Test Security Headers

```bash
# Test con curl
curl -I https://your-site.com

# Tool online
# https://securityheaders.com
# https://observatory.mozilla.org

# Script di verifica
#!/bin/bash
URL=$1
HEADERS=$(curl -sI $URL)

check_header() {
  if echo "$HEADERS" | grep -qi "$1"; then
    echo "✅ $1: presente"
  else
    echo "❌ $1: MANCANTE"
  fi
}

check_header "Strict-Transport-Security"
check_header "Content-Security-Policy"
check_header "X-Frame-Options"
check_header "X-Content-Type-Options"
check_header "Referrer-Policy"
check_header "Permissions-Policy"
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 4: AUTHENTICATION & AUTHORIZATION
# ═══════════════════════════════════════════════════════════════════════════════

AUTHENTICATION = """

## 4.1 Password Security

### Hashing Password

```typescript
// ✅ CORRETTO: bcrypt con cost factor appropriato
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12; // Aumenta per hardware più potente

async function hashPassword(password: string): Promise<string> {
  return bcrypt.hash(password, SALT_ROUNDS);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}

// Alternativa: Argon2 (più moderno, raccomandato)
import argon2 from 'argon2';

async function hashPasswordArgon2(password: string): Promise<string> {
  return argon2.hash(password, {
    type: argon2.argon2id,
    memoryCost: 65536,    // 64 MB
    timeCost: 3,
    parallelism: 4
  });
}

async function verifyPasswordArgon2(password: string, hash: string): Promise<boolean> {
  return argon2.verify(hash, password);
}
```

### Password Policy

```typescript
// schemas/auth.ts
import { z } from 'zod';

export const passwordSchema = z
  .string()
  .min(12, 'Password must be at least 12 characters')
  .max(128, 'Password must be at most 128 characters')
  .regex(/[a-z]/, 'Password must contain a lowercase letter')
  .regex(/[A-Z]/, 'Password must contain an uppercase letter')
  .regex(/[0-9]/, 'Password must contain a number')
  .regex(/[^a-zA-Z0-9]/, 'Password must contain a special character')
  .refine(
    (password) => !commonPasswords.includes(password.toLowerCase()),
    'Password is too common'
  );

// Verifica password compromesse con Have I Been Pwned
import crypto from 'crypto';

async function isPasswordCompromised(password: string): Promise<boolean> {
  const sha1 = crypto.createHash('sha1').update(password).digest('hex').toUpperCase();
  const prefix = sha1.slice(0, 5);
  const suffix = sha1.slice(5);
  
  const response = await fetch(`https://api.pwnedpasswords.com/range/${prefix}`);
  const text = await response.text();
  
  return text.split('\n').some(line => line.startsWith(suffix));
}
```

## 4.2 Multi-Factor Authentication (MFA)

```typescript
// TOTP (Time-based One-Time Password) con speakeasy
import speakeasy from 'speakeasy';
import QRCode from 'qrcode';

// Genera secret per utente
function generateMFASecret(userEmail: string) {
  const secret = speakeasy.generateSecret({
    name: `MyApp:${userEmail}`,
    issuer: 'MyApp',
    length: 32
  });
  
  return {
    base32: secret.base32,
    otpauthUrl: secret.otpauth_url
  };
}

// Genera QR code per app authenticator
async function generateQRCode(otpauthUrl: string): Promise<string> {
  return QRCode.toDataURL(otpauthUrl);
}

// Verifica TOTP
function verifyTOTP(secret: string, token: string): boolean {
  return speakeasy.totp.verify({
    secret,
    encoding: 'base32',
    token,
    window: 1  // Permette 30 secondi di tolleranza
  });
}

// Flow completo MFA
app.post('/api/auth/verify-mfa', authenticate, async (req, res) => {
  const { token } = req.body;
  const user = await db.users.findById(req.user.id);
  
  if (!user.mfaSecret) {
    return res.status(400).json({ error: 'MFA not enabled' });
  }
  
  const isValid = verifyTOTP(user.mfaSecret, token);
  
  if (!isValid) {
    // Log failed attempt
    await logSecurityEvent({
      type: 'MFA_FAILED',
      userId: user.id,
      ip: req.ip
    });
    
    return res.status(401).json({ error: 'Invalid MFA token' });
  }
  
  // Upgrade session to fully authenticated
  req.session.mfaVerified = true;
  res.json({ success: true });
});
```

## 4.3 Brute Force Protection

```typescript
// Rate limiting per login
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Rate limit globale per IP
const loginLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:login:'
  }),
  windowMs: 15 * 60 * 1000, // 15 minuti
  max: 5, // 5 tentativi per IP
  message: { error: 'Too many login attempts. Try again later.' },
  standardHeaders: true,
  legacyHeaders: false,
});

// Rate limit per account (prevenzione enumeration)
const accountLimiter = async (req: Request, res: Response, next: NextFunction) => {
  const { email } = req.body;
  if (!email) return next();
  
  const key = `login:account:${email.toLowerCase()}`;
  const attempts = await redis.incr(key);
  
  if (attempts === 1) {
    await redis.expire(key, 3600); // 1 ora
  }
  
  if (attempts > 10) {
    // Account temporaneamente bloccato
    await sendAccountLockedEmail(email);
    return res.status(429).json({ 
      error: 'Account temporarily locked. Check your email.' 
    });
  }
  
  next();
};

// Reset counter on successful login
async function onSuccessfulLogin(email: string) {
  await redis.del(`login:account:${email.toLowerCase()}`);
}

app.post('/api/auth/login', loginLimiter, accountLimiter, async (req, res) => {
  // ... login logic
});
```

## 4.4 Role-Based Access Control (RBAC)

```typescript
// types/auth.ts
export type Permission = 
  | 'users:read'
  | 'users:create'
  | 'users:update'
  | 'users:delete'
  | 'posts:read'
  | 'posts:create'
  | 'posts:update'
  | 'posts:update:own'
  | 'posts:delete'
  | 'posts:delete:own'
  | 'admin:access';

export type Role = 'admin' | 'editor' | 'author' | 'viewer';

export const rolePermissions: Record<Role, Permission[]> = {
  admin: [
    'users:read', 'users:create', 'users:update', 'users:delete',
    'posts:read', 'posts:create', 'posts:update', 'posts:delete',
    'admin:access'
  ],
  editor: [
    'users:read',
    'posts:read', 'posts:create', 'posts:update', 'posts:delete'
  ],
  author: [
    'posts:read', 'posts:create', 'posts:update:own', 'posts:delete:own'
  ],
  viewer: [
    'posts:read'
  ]
};

// Middleware di autorizzazione
function requirePermission(...permissions: Permission[]) {
  return async (req: Request, res: Response, next: NextFunction) => {
    const user = req.user;
    if (!user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    const userPermissions = rolePermissions[user.role as Role] || [];
    
    const hasPermission = permissions.some(p => {
      // Check exact permission
      if (userPermissions.includes(p)) return true;
      
      // Check :own variant
      if (p.endsWith(':own')) {
        const basePermission = p.replace(':own', '') as Permission;
        return userPermissions.includes(basePermission);
      }
      
      return false;
    });
    
    if (!hasPermission) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
}

// Uso
app.delete('/api/posts/:id', 
  authenticate,
  requirePermission('posts:delete', 'posts:delete:own'),
  async (req, res) => {
    const post = await db.posts.findById(req.params.id);
    
    // Se ha solo :own, verifica ownership
    const userPerms = rolePermissions[req.user.role];
    if (!userPerms.includes('posts:delete') && post.authorId !== req.user.id) {
      return res.status(403).json({ error: 'Can only delete own posts' });
    }
    
    await db.posts.delete(req.params.id);
    res.status(204).send();
  }
);
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 5: JWT SECURITY
# ═══════════════════════════════════════════════════════════════════════════════

JWT_SECURITY = """

## 5.1 JWT Best Practices

### Storage Sicuro JWT

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ JWT STORAGE COMPARISON                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ Storage         │ XSS Risk │ CSRF Risk │ Raccomandazione                   │
│ ────────────────┼──────────┼───────────┼────────────────────────────────── │
│ localStorage    │ ⚠️ ALTO   │ ✅ Basso   │ ❌ NON usare per token sensibili │
│ sessionStorage  │ ⚠️ ALTO   │ ✅ Basso   │ ❌ NON usare per token sensibili │
│ Cookie normale  │ ⚠️ MEDIO  │ ⚠️ ALTO    │ ❌ NON usare                      │
│ HttpOnly Cookie │ ✅ Basso  │ ⚠️ MEDIO   │ ✅ RACCOMANDATO (con CSRF token) │
│ Memory + Refresh│ ✅ Basso  │ ✅ Basso   │ ✅ GOLD STANDARD                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

GOLD STANDARD:
- Access Token: in memory (React state/context) - short-lived (15 min)
- Refresh Token: HttpOnly cookie - long-lived (7 days)
```

### Implementazione JWT Sicura

```typescript
// lib/jwt.ts
import jwt from 'jsonwebtoken';
import { randomBytes } from 'crypto';

const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET!;
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET!;

interface TokenPayload {
  userId: string;
  role: string;
  sessionId: string;
}

// Access Token - Short lived
export function generateAccessToken(payload: TokenPayload): string {
  return jwt.sign(payload, ACCESS_TOKEN_SECRET, {
    algorithm: 'HS256',
    expiresIn: '15m',
    issuer: 'myapp',
    audience: 'myapp-api'
  });
}

// Refresh Token - Long lived, stored in HttpOnly cookie
export function generateRefreshToken(payload: TokenPayload): string {
  return jwt.sign(
    { ...payload, tokenType: 'refresh' },
    REFRESH_TOKEN_SECRET,
    {
      algorithm: 'HS256',
      expiresIn: '7d',
      issuer: 'myapp',
      audience: 'myapp-api'
    }
  );
}

// Verifica Access Token
export function verifyAccessToken(token: string): TokenPayload {
  return jwt.verify(token, ACCESS_TOKEN_SECRET, {
    algorithms: ['HS256'],
    issuer: 'myapp',
    audience: 'myapp-api'
  }) as TokenPayload;
}

// Verifica Refresh Token
export function verifyRefreshToken(token: string): TokenPayload & { tokenType: string } {
  const payload = jwt.verify(token, REFRESH_TOKEN_SECRET, {
    algorithms: ['HS256'],
    issuer: 'myapp',
    audience: 'myapp-api'
  }) as TokenPayload & { tokenType: string };
  
  if (payload.tokenType !== 'refresh') {
    throw new Error('Invalid token type');
  }
  
  return payload;
}
```

### Cookie Configuration per Refresh Token

```typescript
// utils/cookies.ts
import { serialize, parse } from 'cookie';
import { NextApiResponse } from 'next';

const REFRESH_TOKEN_COOKIE = 'refresh_token';

export function setRefreshTokenCookie(res: NextApiResponse, token: string) {
  const cookie = serialize(REFRESH_TOKEN_COOKIE, token, {
    httpOnly: true,     // ⚠️ CRITICO: JavaScript non può accedere
    secure: process.env.NODE_ENV === 'production', // Solo HTTPS in prod
    sameSite: 'strict', // Protezione CSRF
    path: '/api/auth',  // Solo per endpoint auth
    maxAge: 60 * 60 * 24 * 7, // 7 giorni
  });
  
  res.setHeader('Set-Cookie', cookie);
}

export function clearRefreshTokenCookie(res: NextApiResponse) {
  const cookie = serialize(REFRESH_TOKEN_COOKIE, '', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    path: '/api/auth',
    maxAge: 0, // Scade immediatamente
  });
  
  res.setHeader('Set-Cookie', cookie);
}

export function getRefreshTokenFromCookie(req: { cookies: Record<string, string> }): string | null {
  return req.cookies[REFRESH_TOKEN_COOKIE] || null;
}
```

### Flow Completo Authentication

```typescript
// app/api/auth/login.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { generateAccessToken, generateRefreshToken } from '@/lib/jwt';
import { setRefreshTokenCookie } from '@/utils/cookies';
import { verifyPassword } from '@/lib/password';
import { randomUUID } from 'crypto';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { email, password } = req.body;
  
  // Find user
  const user = await db.users.findByEmail(email);
  if (!user) {
    // Timing attack prevention: same response time
    await verifyPassword(password, '$2b$12$dummy.hash.to.prevent.timing');
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Verify password
  const isValid = await verifyPassword(password, user.passwordHash);
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // Generate session ID
  const sessionId = randomUUID();
  
  // Store session in DB (for revocation)
  await db.sessions.create({
    id: sessionId,
    userId: user.id,
    createdAt: new Date(),
    expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days
  });
  
  const tokenPayload = {
    userId: user.id,
    role: user.role,
    sessionId
  };
  
  // Generate tokens
  const accessToken = generateAccessToken(tokenPayload);
  const refreshToken = generateRefreshToken(tokenPayload);
  
  // Set refresh token in HttpOnly cookie
  setRefreshTokenCookie(res, refreshToken);
  
  // Return access token in response body
  res.json({
    accessToken,
    user: {
      id: user.id,
      email: user.email,
      name: user.name,
      role: user.role
    }
  });
}
```

```typescript
// app/api/auth/refresh.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { generateAccessToken, verifyRefreshToken } from '@/lib/jwt';
import { getRefreshTokenFromCookie, setRefreshTokenCookie, clearRefreshTokenCookie } from '@/utils/cookies';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const refreshToken = getRefreshTokenFromCookie(req);
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'No refresh token' });
  }
  
  try {
    const payload = verifyRefreshToken(refreshToken);
    
    // Verify session is still valid in DB
    const session = await db.sessions.findById(payload.sessionId);
    if (!session || session.revokedAt) {
      clearRefreshTokenCookie(res);
      return res.status(401).json({ error: 'Session revoked' });
    }
    
    // Rotate refresh token (security best practice)
    const newRefreshToken = generateRefreshToken({
      userId: payload.userId,
      role: payload.role,
      sessionId: payload.sessionId
    });
    
    const accessToken = generateAccessToken({
      userId: payload.userId,
      role: payload.role,
      sessionId: payload.sessionId
    });
    
    setRefreshTokenCookie(res, newRefreshToken);
    
    res.json({ accessToken });
  } catch (error) {
    clearRefreshTokenCookie(res);
    return res.status(401).json({ error: 'Invalid refresh token' });
  }
}
```

## 5.2 JWT Vulnerabilità e Prevenzione

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ JWT VULNERABILITIES & MITIGATIONS                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ VULNERABILITÀ              │ MITIGAZIONE                                   │
│ ───────────────────────────┼───────────────────────────────────────────── │
│ Algorithm confusion        │ Specifica algoritmo allowed nella verifica   │
│ (alg: none)               │ algorithms: ['HS256'] mai 'none'              │
│                           │                                                │
│ Key confusion             │ Usa chiavi diverse per access/refresh         │
│ (RS256 → HS256)           │ Valida sempre l'algoritmo                     │
│                           │                                                │
│ Token replay              │ Exp claim breve + refresh token rotation      │
│                           │ Session ID + revocation list in DB            │
│                           │                                                │
│ Information disclosure    │ Non mettere dati sensibili nel payload        │
│                           │ Il JWT è encoded, NON encrypted               │
│                           │                                                │
│ Weak secret               │ Usa secrets >= 256 bit random                 │
│                           │ Mai hardcoded, usa env vars                   │
│                           │                                                │
│ Token theft               │ HttpOnly cookie + short exp                   │
│                           │ Binding a fingerprint browser                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```typescript
// ❌ VULNERABILE: Accetta qualsiasi algoritmo
jwt.verify(token, secret); // Potrebbe accettare "none"

// ✅ SICURO: Specifica algoritmi permessi
jwt.verify(token, secret, { algorithms: ['HS256'] });

// ❌ VULNERABILE: Secret debole
const secret = 'password123';

// ✅ SICURO: Secret forte
// Genera con: openssl rand -base64 64
const secret = process.env.JWT_SECRET; // >= 256 bit random

// ❌ VULNERABILE: Dati sensibili nel payload
const token = jwt.sign({
  userId: '123',
  creditCard: '4111-1111-1111-1111', // ⚠️ MAI!
  ssn: '123-45-6789' // ⚠️ MAI!
}, secret);

// ✅ SICURO: Solo dati necessari
const token = jwt.sign({
  userId: '123',
  role: 'user',
  sessionId: 'uuid'
}, secret);
```

## 5.3 Token Revocation

```typescript
// Revocation strategies

// 1. Session-based revocation (raccomandato)
// Ogni token ha sessionId, verifica in DB che non sia revocato

async function verifyTokenWithRevocationCheck(token: string): Promise<TokenPayload> {
  const payload = verifyAccessToken(token);
  
  // Check session not revoked
  const session = await redis.get(`session:${payload.sessionId}`);
  if (!session || JSON.parse(session).revoked) {
    throw new Error('Session revoked');
  }
  
  return payload;
}

// Revoke session (logout, password change, etc.)
async function revokeSession(sessionId: string) {
  await redis.set(
    `session:${sessionId}`,
    JSON.stringify({ revoked: true }),
    'EX',
    60 * 60 * 24 * 7 // Keep for max token lifetime
  );
}

// 2. Token blacklist (per casi specifici)
async function blacklistToken(jti: string, exp: number) {
  const ttl = exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) {
    await redis.set(`blacklist:${jti}`, '1', 'EX', ttl);
  }
}

async function isTokenBlacklisted(jti: string): Promise<boolean> {
  return (await redis.get(`blacklist:${jti}`)) !== null;
}

// 3. Global revocation (cambio secret - NUCLEAR OPTION)
// Cambia JWT_SECRET per invalidare TUTTI i token
// Usare solo in caso di compromissione
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 6: SESSION MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

SESSION_MANAGEMENT = """

## 6.1 Session Security

```typescript
// Express session con Redis (server-side sessions)
import session from 'express-session';
import RedisStore from 'connect-redis';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

app.use(session({
  store: new RedisStore({ client: redis }),
  name: 'sessionId', // Non usare nomi di default come 'connect.sid'
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000, // 24 ore
    path: '/',
    domain: process.env.COOKIE_DOMAIN // Specifica dominio in prod
  }
}));
```

## 6.2 Session Fixation Prevention

```typescript
// Rigenera session ID dopo login
app.post('/api/login', async (req, res) => {
  const { email, password } = req.body;
  
  const user = await authenticateUser(email, password);
  if (!user) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // ⚠️ CRITICO: Rigenera session dopo autenticazione
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ error: 'Session error' });
    }
    
    req.session.userId = user.id;
    req.session.role = user.role;
    req.session.createdAt = Date.now();
    
    res.json({ user });
  });
});

// Rigenera anche dopo elevazione privilegi
app.post('/api/elevate-privileges', authenticate, async (req, res) => {
  // Verifica MFA o password
  const verified = await verifyMFA(req.user.id, req.body.mfaCode);
  if (!verified) {
    return res.status(401).json({ error: 'Invalid MFA' });
  }
  
  req.session.regenerate((err) => {
    if (err) return res.status(500).json({ error: 'Session error' });
    
    req.session.userId = req.user.id;
    req.session.role = 'admin';
    req.session.elevated = true;
    req.session.elevatedAt = Date.now();
    
    res.json({ success: true });
  });
});
```

## 6.3 CSRF Protection

```typescript
// CSRF Token generation
import { randomBytes } from 'crypto';

function generateCSRFToken(): string {
  return randomBytes(32).toString('hex');
}

// Middleware CSRF
app.use((req, res, next) => {
  // Skip per GET, HEAD, OPTIONS (safe methods)
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
    return next();
  }
  
  const tokenFromHeader = req.headers['x-csrf-token'];
  const tokenFromSession = req.session.csrfToken;
  
  if (!tokenFromHeader || tokenFromHeader !== tokenFromSession) {
    return res.status(403).json({ error: 'Invalid CSRF token' });
  }
  
  next();
});

// Endpoint per ottenere token
app.get('/api/csrf-token', (req, res) => {
  const token = generateCSRFToken();
  req.session.csrfToken = token;
  res.json({ csrfToken: token });
});
```

```typescript
// Next.js: Double Submit Cookie pattern
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const response = NextResponse.next();
  
  // Set CSRF cookie if not present
  if (!request.cookies.get('csrf-token')) {
    const csrfToken = crypto.randomUUID();
    response.cookies.set('csrf-token', csrfToken, {
      httpOnly: false, // JS deve poterlo leggere
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      path: '/'
    });
  }
  
  return response;
}

// API route verification
export async function POST(request: NextRequest) {
  const csrfCookie = request.cookies.get('csrf-token')?.value;
  const csrfHeader = request.headers.get('x-csrf-token');
  
  if (!csrfCookie || csrfCookie !== csrfHeader) {
    return NextResponse.json({ error: 'Invalid CSRF token' }, { status: 403 });
  }
  
  // Process request...
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 7: INPUT VALIDATION & OUTPUT ENCODING
# ═══════════════════════════════════════════════════════════════════════════════

INPUT_VALIDATION = """

## 7.1 Validation Strategy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ VALIDATION LAYERS                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ CLIENT-SIDE (UX only, NON fidarsi)                                         │
│ ├── HTML5 validation (required, pattern, type)                             │
│ ├── JavaScript validation                                                   │
│ └── Purpose: Feedback immediato, NON sicurezza                             │
│                                                                             │
│ SERVER-SIDE (CRITICO - sempre obbligatorio)                                │
│ ├── Schema validation (Zod, Yup, Joi)                                      │
│ ├── Business logic validation                                               │
│ ├── Sanitization                                                            │
│ └── Purpose: Sicurezza, integrità dati                                     │
│                                                                             │
│ DATABASE LEVEL (Defense in depth)                                           │
│ ├── Constraints (NOT NULL, CHECK, UNIQUE)                                  │
│ ├── Triggers per validazione complessa                                     │
│ └── Purpose: Ultimo livello di protezione                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 7.2 Schema Validation con Zod

```typescript
// schemas/user.ts
import { z } from 'zod';

// Regex patterns sicuri
const SAFE_STRING_REGEX = /^[a-zA-Z0-9\s\-_.,!?@#$%&*()]+$/;
const USERNAME_REGEX = /^[a-zA-Z][a-zA-Z0-9_-]{2,29}$/;
const PHONE_REGEX = /^\+?[1-9]\d{1,14}$/;

// Schema per creazione utente
export const createUserSchema = z.object({
  username: z.string()
    .min(3, 'Username must be at least 3 characters')
    .max(30, 'Username must be at most 30 characters')
    .regex(USERNAME_REGEX, 'Username must start with a letter and contain only letters, numbers, _ or -'),
  
  email: z.string()
    .email('Invalid email format')
    .max(254, 'Email too long')
    .transform(email => email.toLowerCase().trim()),
  
  password: z.string()
    .min(12, 'Password must be at least 12 characters')
    .max(128, 'Password must be at most 128 characters')
    .regex(/[a-z]/, 'Must contain lowercase')
    .regex(/[A-Z]/, 'Must contain uppercase')
    .regex(/[0-9]/, 'Must contain number')
    .regex(/[^a-zA-Z0-9]/, 'Must contain special character'),
  
  name: z.string()
    .min(1, 'Name is required')
    .max(100, 'Name too long')
    .regex(SAFE_STRING_REGEX, 'Name contains invalid characters')
    .transform(name => name.trim()),
  
  phone: z.string()
    .regex(PHONE_REGEX, 'Invalid phone number')
    .optional(),
  
  bio: z.string()
    .max(500, 'Bio must be at most 500 characters')
    .transform(bio => sanitizeHtml(bio))
    .optional(),
  
  role: z.enum(['user', 'editor', 'admin']).default('user'),
  
  preferences: z.object({
    newsletter: z.boolean().default(false),
    notifications: z.boolean().default(true),
    theme: z.enum(['light', 'dark', 'system']).default('system')
  }).optional()
});

export type CreateUserInput = z.infer<typeof createUserSchema>;

// Schema per update (tutti i campi opzionali tranne ID)
export const updateUserSchema = createUserSchema
  .partial()
  .omit({ password: true }) // Password update separato
  .extend({
    id: z.string().uuid()
  });

// Schema per query parameters
export const userQuerySchema = z.object({
  page: z.coerce.number().int().positive().default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  sort: z.enum(['createdAt', 'name', 'email']).default('createdAt'),
  order: z.enum(['asc', 'desc']).default('desc'),
  search: z.string().max(100).optional(),
  role: z.enum(['user', 'editor', 'admin']).optional()
});
```

## 7.3 Sanitization

```typescript
// lib/sanitize.ts
import DOMPurify from 'isomorphic-dompurify';
import { JSDOM } from 'jsdom';

// Sanitize HTML (per contenuti rich text)
export function sanitizeHtml(dirty: string): string {
  const window = new JSDOM('').window;
  const purify = DOMPurify(window);
  
  return purify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br', 'ul', 'ol', 'li'],
    ALLOWED_ATTR: ['href', 'target', 'rel'],
    ALLOW_DATA_ATTR: false,
    ADD_ATTR: ['target'], // Aggiungi target="_blank"
    FORBID_TAGS: ['script', 'style', 'iframe', 'form', 'input'],
    FORBID_ATTR: ['onerror', 'onclick', 'onload', 'onmouseover']
  });
}

// Sanitize per plain text (rimuovi tutto HTML)
export function sanitizePlainText(dirty: string): string {
  const window = new JSDOM('').window;
  const purify = DOMPurify(window);
  return purify.sanitize(dirty, { ALLOWED_TAGS: [] });
}

// Escape HTML entities
export function escapeHtml(unsafe: string): string {
  return unsafe
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#039;');
}

// Sanitize filename
export function sanitizeFilename(filename: string): string {
  return filename
    .replace(/[^a-zA-Z0-9._-]/g, '_') // Solo caratteri sicuri
    .replace(/\.{2,}/g, '.') // No doppi punti
    .replace(/^\./, '') // No punto iniziale
    .slice(0, 255); // Lunghezza massima
}

// Sanitize URL
export function sanitizeUrl(url: string): string | null {
  try {
    const parsed = new URL(url);
    
    // Solo HTTP/HTTPS
    if (!['http:', 'https:'].includes(parsed.protocol)) {
      return null;
    }
    
    // Blocca localhost/internal in produzione
    if (process.env.NODE_ENV === 'production') {
      const hostname = parsed.hostname.toLowerCase();
      if (
        hostname === 'localhost' ||
        hostname === '127.0.0.1' ||
        hostname.startsWith('192.168.') ||
        hostname.startsWith('10.') ||
        hostname.endsWith('.local')
      ) {
        return null;
      }
    }
    
    return parsed.toString();
  } catch {
    return null;
  }
}
```

## 7.4 Output Encoding

```typescript
// Output encoding per diversi contesti

// 1. HTML Context - usa escapeHtml
// <div>{escapeHtml(userInput)}</div>

// 2. HTML Attribute Context
export function encodeForHtmlAttribute(value: string): string {
  return value
    .replace(/&/g, '&amp;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;');
}

// 3. JavaScript Context
export function encodeForJavaScript(value: string): string {
  return JSON.stringify(value).slice(1, -1); // Rimuovi quotes esterne
}

// 4. URL Context
export function encodeForUrl(value: string): string {
  return encodeURIComponent(value);
}

// 5. CSS Context
export function encodeForCss(value: string): string {
  return value.replace(/[^\w-]/g, char => 
    `\\${char.charCodeAt(0).toString(16)} `
  );
}

// React: Automatic XSS protection
// JSX automaticamente escapa i valori, MA:

// ❌ PERICOLOSO: dangerouslySetInnerHTML
function BadComponent({ html }: { html: string }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}

// ✅ SICURO: Sanitizza prima
function SafeComponent({ html }: { html: string }) {
  const sanitized = sanitizeHtml(html);
  return <div dangerouslySetInnerHTML={{ __html: sanitized }} />;
}

// ✅ MEGLIO: Usa libreria apposita
import parse from 'html-react-parser';

function BetterComponent({ html }: { html: string }) {
  const sanitized = sanitizeHtml(html);
  return <div>{parse(sanitized)}</div>;
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 8: API SECURITY
# ═══════════════════════════════════════════════════════════════════════════════

API_SECURITY = """

## 8.1 API Authentication

```typescript
// Middleware di autenticazione
import { NextApiRequest, NextApiResponse } from 'next';
import { verifyAccessToken } from '@/lib/jwt';

export interface AuthenticatedRequest extends NextApiRequest {
  user: {
    id: string;
    role: string;
    sessionId: string;
  };
}

export function withAuth(
  handler: (req: AuthenticatedRequest, res: NextApiResponse) => Promise<void>
) {
  return async (req: NextApiRequest, res: NextApiResponse) => {
    // Estrai token da header
    const authHeader = req.headers.authorization;
    
    if (!authHeader?.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'Missing authorization header' });
    }
    
    const token = authHeader.slice(7);
    
    try {
      const payload = verifyAccessToken(token);
      
      // Verifica sessione non revocata
      const session = await redis.get(`session:${payload.sessionId}`);
      if (!session) {
        return res.status(401).json({ error: 'Session expired' });
      }
      
      (req as AuthenticatedRequest).user = payload;
      return handler(req as AuthenticatedRequest, res);
    } catch (error) {
      return res.status(401).json({ error: 'Invalid token' });
    }
  };
}

// Uso
export default withAuth(async (req, res) => {
  const userId = req.user.id;
  // ...
});
```

## 8.2 Rate Limiting

```typescript
// lib/rate-limit.ts
import { RateLimiterRedis } from 'rate-limiter-flexible';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Rate limiter per endpoint
const rateLimiters = {
  // Generale: 100 req/min per IP
  general: new RateLimiterRedis({
    storeClient: redis,
    keyPrefix: 'rl:general',
    points: 100,
    duration: 60, // 1 minuto
    blockDuration: 60 * 5 // Block 5 minuti se superato
  }),
  
  // Auth: 5 req/15min per IP (più restrittivo)
  auth: new RateLimiterRedis({
    storeClient: redis,
    keyPrefix: 'rl:auth',
    points: 5,
    duration: 60 * 15, // 15 minuti
    blockDuration: 60 * 30 // Block 30 minuti
  }),
  
  // API key: 1000 req/min per key
  apiKey: new RateLimiterRedis({
    storeClient: redis,
    keyPrefix: 'rl:apikey',
    points: 1000,
    duration: 60
  }),
  
  // Upload: 10 req/ora per utente
  upload: new RateLimiterRedis({
    storeClient: redis,
    keyPrefix: 'rl:upload',
    points: 10,
    duration: 60 * 60
  })
};

// Middleware
export function rateLimit(limiterName: keyof typeof rateLimiters) {
  const limiter = rateLimiters[limiterName];
  
  return async (req: NextApiRequest, res: NextApiResponse, next: () => void) => {
    const key = getClientIdentifier(req, limiterName);
    
    try {
      const result = await limiter.consume(key);
      
      // Headers informativi
      res.setHeader('X-RateLimit-Limit', limiter.points);
      res.setHeader('X-RateLimit-Remaining', result.remainingPoints);
      res.setHeader('X-RateLimit-Reset', new Date(Date.now() + result.msBeforeNext).toISOString());
      
      next();
    } catch (error) {
      if (error instanceof Error) throw error;
      
      // Rate limit exceeded
      res.setHeader('Retry-After', Math.ceil(error.msBeforeNext / 1000));
      res.status(429).json({
        error: 'Too many requests',
        retryAfter: Math.ceil(error.msBeforeNext / 1000)
      });
    }
  };
}

function getClientIdentifier(req: NextApiRequest, limiterName: string): string {
  // Per auth: usa IP
  if (limiterName === 'auth') {
    return getClientIp(req);
  }
  
  // Per API key: usa la key
  if (limiterName === 'apiKey') {
    return req.headers['x-api-key'] as string || getClientIp(req);
  }
  
  // Per upload: usa user ID se autenticato
  if (limiterName === 'upload' && (req as any).user) {
    return (req as any).user.id;
  }
  
  // Default: IP
  return getClientIp(req);
}

function getClientIp(req: NextApiRequest): string {
  // Considera header proxy (Cloudflare, Vercel, etc.)
  const forwarded = req.headers['x-forwarded-for'];
  if (forwarded) {
    return (Array.isArray(forwarded) ? forwarded[0] : forwarded).split(',')[0].trim();
  }
  
  return req.headers['x-real-ip'] as string || 
         req.socket.remoteAddress || 
         'unknown';
}
```

## 8.3 CORS Configuration

```typescript
// lib/cors.ts
import Cors from 'cors';
import { NextApiRequest, NextApiResponse } from 'next';

const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  ...(process.env.NODE_ENV === 'development' ? ['http://localhost:3000'] : [])
];

const cors = Cors({
  origin: (origin, callback) => {
    // Permetti richieste senza origin (mobile apps, Postman)
    if (!origin) {
      return callback(null, true);
    }
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  },
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-CSRF-Token', 'X-API-Key'],
  credentials: true, // Necessario per cookies
  maxAge: 86400, // Cache preflight per 24h
});

// Helper per Next.js API routes
export function runCors(req: NextApiRequest, res: NextApiResponse) {
  return new Promise((resolve, reject) => {
    cors(req, res, (result: any) => {
      if (result instanceof Error) {
        return reject(result);
      }
      return resolve(result);
    });
  });
}

// Uso in API route
export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  try {
    await runCors(req, res);
  } catch {
    return res.status(403).json({ error: 'CORS not allowed' });
  }
  
  // Handler logic...
}
```

## 8.4 API Versioning & Security

```typescript
// Versioning sicuro
// /api/v1/users vs /api/v2/users

// middleware/api-version.ts
export function apiVersion(supportedVersions: string[]) {
  return (req: NextApiRequest, res: NextApiResponse, next: () => void) => {
    const version = req.headers['api-version'] as string || 'v1';
    
    if (!supportedVersions.includes(version)) {
      return res.status(400).json({
        error: 'Unsupported API version',
        supportedVersions
      });
    }
    
    (req as any).apiVersion = version;
    
    // Avvisa se versione deprecata
    if (version === 'v1') {
      res.setHeader('Deprecation', 'true');
      res.setHeader('Sunset', 'Sat, 01 Jan 2026 00:00:00 GMT');
    }
    
    next();
  };
}
```

## 8.5 Request/Response Security

```typescript
// Middleware di sicurezza API
export function apiSecurityMiddleware(
  req: NextApiRequest, 
  res: NextApiResponse, 
  next: () => void
) {
  // 1. Verifica Content-Type per POST/PUT/PATCH
  if (['POST', 'PUT', 'PATCH'].includes(req.method || '')) {
    const contentType = req.headers['content-type'];
    if (!contentType?.includes('application/json')) {
      return res.status(415).json({ error: 'Content-Type must be application/json' });
    }
  }
  
  // 2. Limita dimensione body
  const contentLength = parseInt(req.headers['content-length'] || '0', 10);
  const MAX_BODY_SIZE = 1024 * 1024; // 1MB
  if (contentLength > MAX_BODY_SIZE) {
    return res.status(413).json({ error: 'Request body too large' });
  }
  
  // 3. Security headers per response
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('Cache-Control', 'no-store'); // No cache per API autenticate
  
  // 4. Rimuovi header informativi
  res.removeHeader('X-Powered-By');
  
  next();
}

// Sanitize response (rimuovi dati sensibili)
export function sanitizeResponse<T extends Record<string, any>>(
  data: T,
  sensitiveFields: string[] = ['password', 'passwordHash', 'secret', 'token', 'apiKey']
): T {
  const sanitized = { ...data };
  
  for (const field of sensitiveFields) {
    if (field in sanitized) {
      delete sanitized[field];
    }
  }
  
  // Recursively sanitize nested objects
  for (const key of Object.keys(sanitized)) {
    if (typeof sanitized[key] === 'object' && sanitized[key] !== null) {
      sanitized[key] = sanitizeResponse(sanitized[key], sensitiveFields);
    }
  }
  
  return sanitized;
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 9: SECRETS MANAGEMENT
# ═══════════════════════════════════════════════════════════════════════════════

SECRETS_MANAGEMENT = """

## 9.1 Environment Variables

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ ENVIRONMENT VARIABLES BEST PRACTICES                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ ❌ MAI FARE:                                                                │
│ ├── Committare .env in git                                                 │
│ ├── Hardcodare secrets nel codice                                          │
│ ├── Loggare secrets                                                        │
│ ├── Esporre secrets al client (NEXT_PUBLIC_)                               │
│ └── Usare secrets deboli o predicibili                                     │
│                                                                             │
│ ✅ SEMPRE FARE:                                                             │
│ ├── Usare .env.local per sviluppo (in .gitignore)                         │
│ ├── Usare secrets manager in produzione                                    │
│ ├── Ruotare secrets regolarmente                                           │
│ ├── Usare secrets diversi per ogni ambiente                                │
│ └── Validare che tutti i secrets richiesti esistano all'avvio             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

```typescript
// lib/env.ts - Validazione environment variables
import { z } from 'zod';

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url(),
  
  // Auth
  JWT_SECRET: z.string().min(64, 'JWT_SECRET must be at least 64 characters'),
  REFRESH_TOKEN_SECRET: z.string().min(64),
  
  // External services
  STRIPE_SECRET_KEY: z.string().startsWith('sk_'),
  SENDGRID_API_KEY: z.string().startsWith('SG.'),
  
  // AWS (opzionale, per produzione)
  AWS_ACCESS_KEY_ID: z.string().optional(),
  AWS_SECRET_ACCESS_KEY: z.string().optional(),
  AWS_REGION: z.string().default('eu-north-1'),
  
  // App
  NODE_ENV: z.enum(['development', 'test', 'production']),
  APP_URL: z.string().url(),
  
  // Redis
  REDIS_URL: z.string().url().optional(),
});

// Valida all'avvio
function validateEnv() {
  const result = envSchema.safeParse(process.env);
  
  if (!result.success) {
    console.error('❌ Invalid environment variables:');
    console.error(result.error.format());
    process.exit(1);
  }
  
  return result.data;
}

export const env = validateEnv();

// Type-safe access
// env.DATABASE_URL ✅ TypeScript conosce il tipo
```

```bash
# .env.example (committare questo, NON .env)
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/myapp

# Auth - Genera con: openssl rand -base64 64
JWT_SECRET=your-jwt-secret-min-64-chars
REFRESH_TOKEN_SECRET=your-refresh-secret-min-64-chars

# External services
STRIPE_SECRET_KEY=sk_test_...
SENDGRID_API_KEY=SG....

# App
NODE_ENV=development
APP_URL=http://localhost:3000
```

## 9.2 AWS Secrets Manager

```typescript
// lib/secrets.ts
import { 
  SecretsManagerClient, 
  GetSecretValueCommand 
} from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: process.env.AWS_REGION });

// Cache secrets in memoria
const secretsCache = new Map<string, { value: string; expiresAt: number }>();
const CACHE_TTL = 5 * 60 * 1000; // 5 minuti

export async function getSecret(secretName: string): Promise<string> {
  // Check cache
  const cached = secretsCache.get(secretName);
  if (cached && cached.expiresAt > Date.now()) {
    return cached.value;
  }
  
  try {
    const command = new GetSecretValueCommand({ SecretId: secretName });
    const response = await client.send(command);
    
    const secretValue = response.SecretString || '';
    
    // Cache it
    secretsCache.set(secretName, {
      value: secretValue,
      expiresAt: Date.now() + CACHE_TTL
    });
    
    return secretValue;
  } catch (error) {
    console.error(`Failed to retrieve secret ${secretName}:`, error);
    throw new Error(`Secret ${secretName} not found`);
  }
}

// Per secrets JSON
export async function getSecretJson<T>(secretName: string): Promise<T> {
  const secret = await getSecret(secretName);
  return JSON.parse(secret);
}

// Uso
const dbCredentials = await getSecretJson<{ username: string; password: string }>(
  'myapp/production/database'
);
```

## 9.3 Secrets Rotation

```typescript
// Strategie per rotazione secrets

// 1. Dual secrets durante rotazione
// Accetta sia vecchio che nuovo durante transizione

async function verifyApiKey(apiKey: string): Promise<boolean> {
  const currentKey = await getSecret('api-key-current');
  const previousKey = await getSecret('api-key-previous');
  
  // Verifica timing-safe
  return timingSafeEqual(apiKey, currentKey) || 
         timingSafeEqual(apiKey, previousKey);
}

// Timing-safe comparison (previene timing attacks)
function timingSafeEqual(a: string, b: string): boolean {
  if (a.length !== b.length) {
    return false;
  }
  
  let result = 0;
  for (let i = 0; i < a.length; i++) {
    result |= a.charCodeAt(i) ^ b.charCodeAt(i);
  }
  
  return result === 0;
}

// 2. Graceful rotation per JWT secrets
// Accetta token firmati con vecchio secret durante transizione
function verifyTokenWithRotation(token: string): TokenPayload {
  const secrets = [
    process.env.JWT_SECRET_CURRENT,
    process.env.JWT_SECRET_PREVIOUS
  ].filter(Boolean);
  
  for (const secret of secrets) {
    try {
      return jwt.verify(token, secret, { algorithms: ['HS256'] }) as TokenPayload;
    } catch {
      continue;
    }
  }
  
  throw new Error('Invalid token');
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 10: DATABASE SECURITY
# ═══════════════════════════════════════════════════════════════════════════════

DATABASE_SECURITY = """

## 10.1 Connection Security

```typescript
// PostgreSQL con SSL
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  ssl: process.env.NODE_ENV === 'production' ? {
    rejectUnauthorized: true, // Verifica certificato
    ca: process.env.DATABASE_CA_CERT // CA certificate
  } : false,
  
  // Connection limits
  max: 20, // Max connections nel pool
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 5000,
});

// Prisma con SSL
// schema.prisma
// datasource db {
//   provider = "postgresql"
//   url      = env("DATABASE_URL")
// }

// DATABASE_URL="postgresql://user:pass@host:5432/db?sslmode=require&sslcert=/path/to/cert"
```

## 10.2 Query Security

```typescript
// ✅ SEMPRE usare parameterized queries

// Con pg (node-postgres)
async function getUserByEmail(email: string) {
  const result = await pool.query(
    'SELECT id, email, name FROM users WHERE email = $1',
    [email]
  );
  return result.rows[0];
}

// Con Prisma (automaticamente sicuro)
async function getUserByEmail(email: string) {
  return prisma.user.findUnique({
    where: { email },
    select: { id: true, email: true, name: true }
  });
}

// ⚠️ Raw queries in Prisma - usa $queryRaw con template literals
async function searchUsers(searchTerm: string) {
  // ✅ SICURO: Template literal con Prisma.sql
  return prisma.$queryRaw`
    SELECT id, email, name 
    FROM users 
    WHERE name ILIKE ${'%' + searchTerm + '%'}
    LIMIT 50
  `;
  
  // ❌ VULNERABILE: String concatenation
  // return prisma.$queryRawUnsafe(`SELECT * FROM users WHERE name LIKE '%${searchTerm}%'`);
}
```

## 10.3 Principle of Least Privilege per Database

```sql
-- Crea utente applicazione con permessi minimi
CREATE USER app_user WITH PASSWORD 'strong_password';

-- Schema dedicato per l'applicazione
CREATE SCHEMA app;

-- Permessi solo sullo schema app
GRANT USAGE ON SCHEMA app TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA app TO app_user;
GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA app TO app_user;

-- Default permissions per nuove tabelle
ALTER DEFAULT PRIVILEGES IN SCHEMA app 
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;

-- ❌ MAI dare questi permessi all'app user:
-- GRANT ALL PRIVILEGES ON DATABASE mydb TO app_user;
-- GRANT DROP ON ALL TABLES TO app_user;
-- GRANT CREATE ON SCHEMA public TO app_user;

-- Utente separato per migrations (usato solo in CI/CD)
CREATE USER migration_user WITH PASSWORD 'another_strong_password';
GRANT ALL PRIVILEGES ON SCHEMA app TO migration_user;
```

## 10.4 Data Encryption

```typescript
// Encryption at rest: Configurato a livello database/cloud

// Encryption in transit: SSL/TLS (vedi Connection Security)

// Application-level encryption per dati sensibili
import crypto from 'crypto';

const ENCRYPTION_KEY = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex'); // 32 bytes
const IV_LENGTH = 16;

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag().toString('hex');
  
  // Format: iv:authTag:encrypted
  return `${iv.toString('hex')}:${authTag}:${encrypted}`;
}

export function decrypt(encryptedText: string): string {
  const [ivHex, authTagHex, encrypted] = encryptedText.split(':');
  
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  
  const decipher = crypto.createDecipheriv('aes-256-gcm', ENCRYPTION_KEY, iv);
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

// Uso per dati sensibili (SSN, numeri carta, etc.)
// NOTA: Stripe/payment processors gestiscono carte - NON salvare numeri carta!

const user = {
  email: 'user@example.com',
  ssn_encrypted: encrypt('123-45-6789'), // Solo se strettamente necessario
};
```

## 10.5 Audit Logging

```typescript
// Tabella audit log
/*
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  user_id UUID REFERENCES users(id),
  action VARCHAR(50) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id VARCHAR(100),
  old_values JSONB,
  new_values JSONB,
  ip_address INET,
  user_agent TEXT,
  request_id UUID
);

CREATE INDEX idx_audit_logs_user_id ON audit_logs(user_id);
CREATE INDEX idx_audit_logs_resource ON audit_logs(resource_type, resource_id);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
*/

// Middleware di audit
export async function auditLog(params: {
  userId?: string;
  action: 'CREATE' | 'READ' | 'UPDATE' | 'DELETE' | 'LOGIN' | 'LOGOUT' | 'EXPORT';
  resourceType: string;
  resourceId?: string;
  oldValues?: Record<string, any>;
  newValues?: Record<string, any>;
  req?: NextApiRequest;
}) {
  const { userId, action, resourceType, resourceId, oldValues, newValues, req } = params;
  
  // Rimuovi campi sensibili prima di loggare
  const sanitizedOld = oldValues ? sanitizeForLog(oldValues) : null;
  const sanitizedNew = newValues ? sanitizeForLog(newValues) : null;
  
  await prisma.auditLog.create({
    data: {
      userId,
      action,
      resourceType,
      resourceId,
      oldValues: sanitizedOld,
      newValues: sanitizedNew,
      ipAddress: req ? getClientIp(req) : null,
      userAgent: req?.headers['user-agent'] || null,
      requestId: req?.headers['x-request-id'] as string || null
    }
  });
}

function sanitizeForLog(data: Record<string, any>): Record<string, any> {
  const sensitiveFields = ['password', 'passwordHash', 'token', 'secret', 'ssn', 'creditCard'];
  const sanitized = { ...data };
  
  for (const field of sensitiveFields) {
    if (field in sanitized) {
      sanitized[field] = '[REDACTED]';
    }
  }
  
  return sanitized;
}

// Uso
await auditLog({
  userId: req.user.id,
  action: 'UPDATE',
  resourceType: 'user',
  resourceId: targetUserId,
  oldValues: existingUser,
  newValues: updatedUser,
  req
});
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 11: FILE UPLOAD SECURITY
# ═══════════════════════════════════════════════════════════════════════════════

FILE_UPLOAD_SECURITY = """

## 11.1 File Upload Validation

```typescript
// lib/upload-security.ts
import { fileTypeFromBuffer } from 'file-type';
import path from 'path';

// Whitelist MIME types permessi
const ALLOWED_MIME_TYPES = new Map([
  // Images
  ['image/jpeg', ['.jpg', '.jpeg']],
  ['image/png', ['.png']],
  ['image/gif', ['.gif']],
  ['image/webp', ['.webp']],
  // Documents
  ['application/pdf', ['.pdf']],
  ['application/msword', ['.doc']],
  ['application/vnd.openxmlformats-officedocument.wordprocessingml.document', ['.docx']],
  // Spreadsheets
  ['application/vnd.ms-excel', ['.xls']],
  ['application/vnd.openxmlformats-officedocument.spreadsheetml.sheet', ['.xlsx']],
]);

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10 MB

interface ValidatedFile {
  originalName: string;
  safeName: string;
  mimeType: string;
  size: number;
  buffer: Buffer;
}

export async function validateUpload(
  file: { name: string; data: Buffer; size: number }
): Promise<ValidatedFile> {
  const { name, data, size } = file;
  
  // 1. Check file size
  if (size > MAX_FILE_SIZE) {
    throw new Error(`File too large. Maximum size is ${MAX_FILE_SIZE / 1024 / 1024}MB`);
  }
  
  if (size === 0) {
    throw new Error('Empty file');
  }
  
  // 2. Detect real MIME type from content (non fidarsi dell'estensione!)
  const detectedType = await fileTypeFromBuffer(data);
  
  if (!detectedType) {
    throw new Error('Could not determine file type');
  }
  
  // 3. Verify MIME type is allowed
  if (!ALLOWED_MIME_TYPES.has(detectedType.mime)) {
    throw new Error(`File type ${detectedType.mime} is not allowed`);
  }
  
  // 4. Verify extension matches MIME type
  const ext = path.extname(name).toLowerCase();
  const allowedExtensions = ALLOWED_MIME_TYPES.get(detectedType.mime)!;
  
  if (!allowedExtensions.includes(ext)) {
    throw new Error(`Extension ${ext} does not match detected type ${detectedType.mime}`);
  }
  
  // 5. Sanitize filename
  const safeName = sanitizeFilename(name);
  
  // 6. Check for malicious content (basic)
  await scanForMaliciousContent(data, detectedType.mime);
  
  return {
    originalName: name,
    safeName,
    mimeType: detectedType.mime,
    size,
    buffer: data
  };
}

function sanitizeFilename(filename: string): string {
  // Rimuovi path traversal
  const basename = path.basename(filename);
  
  // Solo caratteri sicuri
  const safe = basename
    .replace(/[^a-zA-Z0-9._-]/g, '_')
    .replace(/\.{2,}/g, '.') // No doppi punti
    .replace(/^\./, ''); // No punto iniziale
  
  // Aggiungi timestamp per unicità
  const ext = path.extname(safe);
  const name = path.basename(safe, ext);
  
  return `${name}_${Date.now()}${ext}`;
}

async function scanForMaliciousContent(buffer: Buffer, mimeType: string): Promise<void> {
  // Per immagini: verifica che sia realmente un'immagine valida
  if (mimeType.startsWith('image/')) {
    // Usa sharp o jimp per validare
    const sharp = await import('sharp');
    try {
      await sharp.default(buffer).metadata();
    } catch {
      throw new Error('Invalid or corrupted image file');
    }
  }
  
  // Check per PHP/script injection nei file
  const content = buffer.toString('utf8', 0, 1000); // Check primi 1000 bytes
  const dangerousPatterns = [
    /<\?php/i,
    /<script/i,
    /<%/,
    /javascript:/i,
    /data:text\/html/i
  ];
  
  for (const pattern of dangerousPatterns) {
    if (pattern.test(content)) {
      throw new Error('Potentially malicious content detected');
    }
  }
}
```

## 11.2 Secure File Storage

```typescript
// Upload a S3 con configurazione sicura
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({ region: process.env.AWS_REGION });
const BUCKET_NAME = process.env.S3_BUCKET_NAME!;

export async function uploadToS3(
  file: ValidatedFile,
  userId: string
): Promise<{ key: string; url: string }> {
  // Struttura: uploads/{userId}/{year}/{month}/{filename}
  const date = new Date();
  const key = `uploads/${userId}/${date.getFullYear()}/${String(date.getMonth() + 1).padStart(2, '0')}/${file.safeName}`;
  
  await s3.send(new PutObjectCommand({
    Bucket: BUCKET_NAME,
    Key: key,
    Body: file.buffer,
    ContentType: file.mimeType,
    ContentDisposition: `attachment; filename="${file.safeName}"`, // Force download
    Metadata: {
      'original-name': file.originalName,
      'uploaded-by': userId,
      'uploaded-at': new Date().toISOString()
    },
    // Server-side encryption
    ServerSideEncryption: 'AES256',
    // Oppure KMS:
    // ServerSideEncryption: 'aws:kms',
    // SSEKMSKeyId: process.env.KMS_KEY_ID
  }));
  
  // URL per accesso (presigned per files privati)
  const url = await getSignedUrl(
    s3,
    new GetObjectCommand({ Bucket: BUCKET_NAME, Key: key }),
    { expiresIn: 3600 } // 1 ora
  );
  
  return { key, url };
}

// S3 Bucket policy sicura
/*
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringNotEquals": {
          "s3:x-amz-server-side-encryption": "AES256"
        }
      }
    }
  ]
}
*/
```

## 11.3 Image Processing Security

```typescript
// Processa immagini in modo sicuro
import sharp from 'sharp';

export async function processImage(
  buffer: Buffer,
  options: {
    maxWidth?: number;
    maxHeight?: number;
    format?: 'jpeg' | 'png' | 'webp';
    quality?: number;
  } = {}
): Promise<Buffer> {
  const { maxWidth = 1920, maxHeight = 1080, format = 'webp', quality = 80 } = options;
  
  // Sharp rimuove automaticamente metadata EXIF (potenzialmente sensibili)
  let processor = sharp(buffer)
    .rotate() // Auto-rotate based on EXIF (then remove EXIF)
    .resize(maxWidth, maxHeight, {
      fit: 'inside',
      withoutEnlargement: true
    });
  
  switch (format) {
    case 'jpeg':
      processor = processor.jpeg({ quality, mozjpeg: true });
      break;
    case 'png':
      processor = processor.png({ compressionLevel: 9 });
      break;
    case 'webp':
      processor = processor.webp({ quality });
      break;
  }
  
  return processor.toBuffer();
}

// Genera thumbnail sicure
export async function generateThumbnail(buffer: Buffer): Promise<Buffer> {
  return sharp(buffer)
    .rotate()
    .resize(200, 200, {
      fit: 'cover',
      position: 'centre'
    })
    .webp({ quality: 70 })
    .toBuffer();
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 12: RATE LIMITING & DDOS PROTECTION
# ═══════════════════════════════════════════════════════════════════════════════

RATE_LIMITING = """

## 12.1 Multi-Layer Rate Limiting

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ RATE LIMITING LAYERS                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ LAYER 1: CDN/Edge (Cloudflare, Vercel Edge)                                │
│ ├── DDoS protection                                                        │
│ ├── Bot detection                                                          │
│ └── Geographic blocking                                                    │
│                                                                             │
│ LAYER 2: Load Balancer/API Gateway                                         │
│ ├── Connection limits                                                      │
│ ├── Request rate limits per IP                                             │
│ └── Bandwidth limits                                                       │
│                                                                             │
│ LAYER 3: Application                                                       │
│ ├── Endpoint-specific limits                                               │
│ ├── User-based limits                                                      │
│ ├── Resource-based limits                                                  │
│ └── Sliding window algorithms                                              │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 12.2 Sliding Window Rate Limiter

```typescript
// Sliding window più preciso del fixed window
import { RateLimiterRedis } from 'rate-limiter-flexible';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// Sliding window con penalty per abusi
class AdvancedRateLimiter {
  private limiter: RateLimiterRedis;
  private penaltyLimiter: RateLimiterRedis;
  
  constructor(options: {
    keyPrefix: string;
    points: number;
    duration: number;
    blockDuration?: number;
  }) {
    this.limiter = new RateLimiterRedis({
      storeClient: redis,
      keyPrefix: options.keyPrefix,
      points: options.points,
      duration: options.duration,
      blockDuration: options.blockDuration || 0
    });
    
    // Penalty tracker per repeat offenders
    this.penaltyLimiter = new RateLimiterRedis({
      storeClient: redis,
      keyPrefix: `${options.keyPrefix}:penalty`,
      points: 3, // 3 violations
      duration: 60 * 60, // in 1 ora
      blockDuration: 60 * 60 * 24 // Block per 24 ore
    });
  }
  
  async consume(key: string): Promise<{ allowed: boolean; retryAfter?: number }> {
    try {
      // Check se già in penalty
      try {
        await this.penaltyLimiter.get(key);
      } catch {
        return { allowed: false, retryAfter: 86400 }; // Blocked
      }
      
      // Normal rate limit
      await this.limiter.consume(key);
      return { allowed: true };
    } catch (error: any) {
      // Rate limit exceeded - add penalty point
      await this.penaltyLimiter.consume(key).catch(() => {});
      
      return {
        allowed: false,
        retryAfter: Math.ceil(error.msBeforeNext / 1000)
      };
    }
  }
}

// Istanze per diversi endpoint
const rateLimiters = {
  api: new AdvancedRateLimiter({
    keyPrefix: 'rl:api',
    points: 100,
    duration: 60,
    blockDuration: 300
  }),
  
  auth: new AdvancedRateLimiter({
    keyPrefix: 'rl:auth',
    points: 5,
    duration: 900, // 15 min
    blockDuration: 3600 // 1 ora
  }),
  
  expensive: new AdvancedRateLimiter({
    keyPrefix: 'rl:expensive',
    points: 10,
    duration: 3600, // 1 ora
    blockDuration: 7200
  })
};
```

## 12.3 Cloudflare Configuration

```javascript
// Cloudflare Workers rate limiting
// wrangler.toml
/*
name = "my-api-gateway"
main = "src/index.ts"

[[rate_limits]]
name = "api-rate-limit"
rate_limit = { per_minute = 100 }
action = { type = "block", response = { status_code = 429, body = '{"error":"Rate limit exceeded"}' } }
matching_rule = { path = "/api/*" }
*/

// Cloudflare Page Rules / Firewall Rules
/*
1. Challenge suspicious traffic:
   - (cf.threat_score gt 10) → Challenge

2. Block known bad bots:
   - (cf.client.bot) and not (cf.bot_management.verified_bot) → Block

3. Rate limit by country (se necessario):
   - (ip.geoip.country in {"CN" "RU"}) → Rate Limit

4. Protect login:
   - (http.request.uri.path eq "/api/auth/login") → 
     Rate Limit: 5 requests per 15 minutes per IP
*/
```

## 12.4 Request Validation & Abuse Prevention

```typescript
// Middleware anti-abuse
export function abusePreventionMiddleware(
  req: NextApiRequest,
  res: NextApiResponse,
  next: () => void
) {
  // 1. Block requests without User-Agent
  if (!req.headers['user-agent']) {
    return res.status(400).json({ error: 'User-Agent header required' });
  }
  
  // 2. Block suspicious User-Agents
  const ua = req.headers['user-agent'].toLowerCase();
  const suspiciousPatterns = [
    'sqlmap', 'nikto', 'nessus', 'acunetix',
    'nmap', 'masscan', 'zgrab'
  ];
  
  if (suspiciousPatterns.some(pattern => ua.includes(pattern))) {
    // Log per analisi
    console.warn('Suspicious UA blocked:', req.headers['user-agent'], getClientIp(req));
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  // 3. Validate Content-Length matches body
  const contentLength = parseInt(req.headers['content-length'] || '0', 10);
  if (req.method !== 'GET' && req.body) {
    const actualLength = Buffer.byteLength(JSON.stringify(req.body));
    if (Math.abs(contentLength - actualLength) > 100) {
      return res.status(400).json({ error: 'Content-Length mismatch' });
    }
  }
  
  // 4. Block requests with too many headers
  if (Object.keys(req.headers).length > 50) {
    return res.status(400).json({ error: 'Too many headers' });
  }
  
  next();
}
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 13: SECURITY AUDIT CHECKLIST
# ═══════════════════════════════════════════════════════════════════════════════

SECURITY_AUDIT_CHECKLIST = """

## 13.1 Pre-Deploy Security Checklist

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PRE-DEPLOY SECURITY CHECKLIST                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ AUTHENTICATION & AUTHORIZATION                                              │
│ □ Password hashing con bcrypt/argon2 (cost factor ≥ 12)                    │
│ □ Rate limiting su login (5 tentativi / 15 min)                            │
│ □ Session regeneration dopo login                                          │
│ □ CSRF protection implementata                                             │
│ □ JWT con exp breve + refresh token rotation                               │
│ □ Logout invalida sessione server-side                                     │
│ □ MFA disponibile per utenti                                               │
│ □ Password policy robusta (min 12 char, complessità)                       │
│                                                                             │
│ INPUT VALIDATION                                                            │
│ □ Tutti gli input validati server-side (Zod/Yup)                           │
│ □ Parameterized queries per tutti i DB access                              │
│ □ HTML sanitization per user content                                       │
│ □ File upload validation (MIME type da content, non extension)             │
│ □ URL validation per redirect                                              │
│                                                                             │
│ SECURITY HEADERS                                                            │
│ □ Strict-Transport-Security (HSTS)                                         │
│ □ Content-Security-Policy (CSP)                                            │
│ □ X-Frame-Options / frame-ancestors                                        │
│ □ X-Content-Type-Options: nosniff                                          │
│ □ Referrer-Policy                                                          │
│ □ Permissions-Policy                                                       │
│                                                                             │
│ DATA PROTECTION                                                             │
│ □ HTTPS everywhere (no mixed content)                                      │
│ □ Sensitive data encrypted at rest                                         │
│ □ Database connections use SSL                                             │
│ □ No secrets in code/logs/URLs                                             │
│ □ Environment variables per secrets                                        │
│ □ Secrets rotation procedure documentata                                   │
│                                                                             │
│ API SECURITY                                                                │
│ □ Rate limiting su tutti gli endpoint                                      │
│ □ CORS configurato correttamente                                           │
│ □ API versioning                                                           │
│ □ Error messages non espongono dettagli interni                            │
│ □ Request size limits                                                      │
│                                                                             │
│ INFRASTRUCTURE                                                              │
│ □ Firewall rules restrictive                                               │
│ □ Database non esposto pubblicamente                                       │
│ □ Container non-root                                                       │
│ □ Dependency audit (npm audit, Snyk)                                       │
│ □ Logging e monitoring configurati                                         │
│ □ Backup automatici                                                        │
│                                                                             │
│ COMPLIANCE                                                                  │
│ □ Privacy policy presente                                                  │
│ □ Cookie consent (se necessario)                                           │
│ □ Data retention policy                                                    │
│ □ GDPR compliance (se applicabile)                                         │
│ □ PCI-DSS compliance (se pagamenti)                                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 13.2 Automated Security Testing

```yaml
# .github/workflows/security.yml
name: Security Audit

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 0 * * *' # Daily

jobs:
  dependency-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: NPM Audit
        run: npm audit --audit-level=high
        continue-on-error: true
      
      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  code-security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
      
      - name: CodeQL Analysis
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript, typescript

  secrets-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: TruffleHog Secrets Scan
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified

  container-scan:
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'myapp:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'HIGH,CRITICAL'
      
      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

## 13.3 Manual Security Review

```typescript
// scripts/security-check.ts
// Script per verifiche manuali pre-deploy

import { execSync } from 'child_process';
import fs from 'fs';
import path from 'path';

interface SecurityCheck {
  name: string;
  check: () => boolean | Promise<boolean>;
  severity: 'critical' | 'high' | 'medium' | 'low';
}

const checks: SecurityCheck[] = [
  {
    name: 'No .env files in git',
    severity: 'critical',
    check: () => {
      const gitFiles = execSync('git ls-files').toString().split('\n');
      return !gitFiles.some(f => f.includes('.env') && !f.includes('.example'));
    }
  },
  {
    name: 'No hardcoded secrets in code',
    severity: 'critical',
    check: () => {
      const patterns = [
        /password\s*=\s*['"][^'"]+['"]/gi,
        /api[_-]?key\s*=\s*['"][^'"]+['"]/gi,
        /secret\s*=\s*['"][^'"]+['"]/gi,
        /sk_live_[a-zA-Z0-9]+/g, // Stripe live key
      ];
      
      const sourceFiles = execSync('find src -name "*.ts" -o -name "*.tsx" -o -name "*.js"')
        .toString()
        .split('\n')
        .filter(Boolean);
      
      for (const file of sourceFiles) {
        const content = fs.readFileSync(file, 'utf8');
        for (const pattern of patterns) {
          if (pattern.test(content)) {
            console.error(`  ⚠️ Potential secret in ${file}`);
            return false;
          }
        }
      }
      return true;
    }
  },
  {
    name: 'HTTPS enforced',
    severity: 'critical',
    check: () => {
      const nextConfig = fs.readFileSync('next.config.js', 'utf8');
      return nextConfig.includes('Strict-Transport-Security');
    }
  },
  {
    name: 'CSP header configured',
    severity: 'high',
    check: () => {
      const nextConfig = fs.readFileSync('next.config.js', 'utf8');
      return nextConfig.includes('Content-Security-Policy');
    }
  },
  {
    name: 'No console.log in production code',
    severity: 'low',
    check: () => {
      const result = execSync('grep -r "console.log" src/ --include="*.ts" --include="*.tsx" | wc -l')
        .toString()
        .trim();
      return parseInt(result) < 10; // Allow some for debugging
    }
  },
  {
    name: 'Dependencies up to date',
    severity: 'medium',
    check: () => {
      try {
        execSync('npm audit --audit-level=high', { stdio: 'pipe' });
        return true;
      } catch {
        return false;
      }
    }
  }
];

async function runSecurityChecks() {
  console.log('🔒 Running Security Checks...\n');
  
  let passed = 0;
  let failed = 0;
  
  for (const check of checks) {
    try {
      const result = await check.check();
      if (result) {
        console.log(`✅ ${check.name}`);
        passed++;
      } else {
        console.log(`❌ ${check.name} [${check.severity.toUpperCase()}]`);
        failed++;
      }
    } catch (error) {
      console.log(`❌ ${check.name} - Error: ${error}`);
      failed++;
    }
  }
  
  console.log(`\n📊 Results: ${passed} passed, ${failed} failed`);
  
  if (failed > 0) {
    process.exit(1);
  }
}

runSecurityChecks();
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 14: INCIDENT RESPONSE
# ═══════════════════════════════════════════════════════════════════════════════

INCIDENT_RESPONSE = """

## 14.1 Incident Response Plan

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ INCIDENT RESPONSE PHASES                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ 1. DETECTION                                                                │
│    ├── Monitoring alerts (errors, suspicious activity)                     │
│    ├── User reports                                                        │
│    ├── Security tool alerts                                                │
│    └── Log analysis                                                        │
│                                                                             │
│ 2. CONTAINMENT                                                              │
│    ├── Isolare sistemi compromessi                                         │
│    ├── Revocare credenziali compromesse                                    │
│    ├── Bloccare IP sospetti                                                │
│    └── Disabilitare funzionalità vulnerabili                              │
│                                                                             │
│ 3. ERADICATION                                                              │
│    ├── Identificare root cause                                             │
│    ├── Rimuovere malware/backdoor                                          │
│    ├── Patchare vulnerabilità                                              │
│    └── Validare fix                                                        │
│                                                                             │
│ 4. RECOVERY                                                                 │
│    ├── Restore da backup (se necessario)                                   │
│    ├── Ripristino graduale servizi                                         │
│    ├── Monitoring intensivo                                                │
│    └── Comunicazione utenti                                                │
│                                                                             │
│ 5. POST-INCIDENT                                                            │
│    ├── Post-mortem analysis                                                │
│    ├── Documentazione                                                      │
│    ├── Update procedure                                                    │
│    └── Training team                                                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

## 14.2 Emergency Response Scripts

```typescript
// scripts/emergency-response.ts
// Script per risposta rapida a incidenti

import { PrismaClient } from '@prisma/client';
import Redis from 'ioredis';

const prisma = new PrismaClient();
const redis = new Redis(process.env.REDIS_URL);

// 1. Revoca tutte le sessioni di un utente
async function revokeAllUserSessions(userId: string) {
  console.log(`🔒 Revoking all sessions for user ${userId}`);
  
  // Revoca sessioni in DB
  await prisma.session.updateMany({
    where: { userId, revokedAt: null },
    data: { revokedAt: new Date() }
  });
  
  // Invalida cache sessioni
  const keys = await redis.keys(`session:*:${userId}`);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
  
  // Log audit
  await prisma.auditLog.create({
    data: {
      action: 'EMERGENCY_SESSION_REVOKE',
      resourceType: 'user',
      resourceId: userId,
      metadata: { reason: 'Security incident response' }
    }
  });
  
  console.log(`✅ Revoked ${keys.length} sessions`);
}

// 2. Blocca un IP
async function blockIP(ip: string, reason: string, durationHours: number = 24) {
  console.log(`🚫 Blocking IP ${ip} for ${durationHours} hours`);
  
  await redis.set(
    `blocked:ip:${ip}`,
    JSON.stringify({ reason, blockedAt: new Date().toISOString() }),
    'EX',
    durationHours * 60 * 60
  );
  
  // Log
  await prisma.auditLog.create({
    data: {
      action: 'IP_BLOCKED',
      resourceType: 'ip',
      resourceId: ip,
      metadata: { reason, durationHours }
    }
  });
  
  console.log(`✅ IP ${ip} blocked`);
}

// 3. Force password reset per tutti gli utenti
async function forceGlobalPasswordReset() {
  console.log(`⚠️ Forcing password reset for all users`);
  
  await prisma.user.updateMany({
    data: { 
      mustResetPassword: true,
      passwordResetRequiredAt: new Date()
    }
  });
  
  // Revoca tutte le sessioni
  await prisma.session.updateMany({
    where: { revokedAt: null },
    data: { revokedAt: new Date() }
  });
  
  // Clear all session cache
  const keys = await redis.keys('session:*');
  if (keys.length > 0) {
    await redis.del(...keys);
  }
  
  console.log(`✅ Password reset required for all users`);
}

// 4. Rotate JWT secrets (NUCLEAR OPTION)
async function rotateJWTSecrets() {
  console.log(`🔐 JWT secret rotation - THIS WILL LOG OUT ALL USERS`);
  
  // Questo richiede update delle env vars e redeploy
  // Genera nuovi secrets
  const crypto = await import('crypto');
  const newAccessSecret = crypto.randomBytes(64).toString('base64');
  const newRefreshSecret = crypto.randomBytes(64).toString('base64');
  
  console.log(`
  ⚠️ UPDATE THESE IN YOUR SECRETS MANAGER:
  
  JWT_SECRET=${newAccessSecret}
  REFRESH_TOKEN_SECRET=${newRefreshSecret}
  
  Then redeploy the application.
  `);
}

// 5. Export user data (per GDPR breach notification)
async function exportUserDataForBreach(userIds: string[]) {
  console.log(`📤 Exporting data for ${userIds.length} affected users`);
  
  const users = await prisma.user.findMany({
    where: { id: { in: userIds } },
    select: {
      id: true,
      email: true,
      name: true,
      createdAt: true
    }
  });
  
  return users;
}

// CLI
const command = process.argv[2];
const args = process.argv.slice(3);

switch (command) {
  case 'revoke-sessions':
    revokeAllUserSessions(args[0]);
    break;
  case 'block-ip':
    blockIP(args[0], args[1] || 'Security incident', parseInt(args[2]) || 24);
    break;
  case 'force-password-reset':
    forceGlobalPasswordReset();
    break;
  case 'rotate-jwt':
    rotateJWTSecrets();
    break;
  default:
    console.log(`
Usage:
  npx ts-node scripts/emergency-response.ts revoke-sessions <userId>
  npx ts-node scripts/emergency-response.ts block-ip <ip> [reason] [hours]
  npx ts-node scripts/emergency-response.ts force-password-reset
  npx ts-node scripts/emergency-response.ts rotate-jwt
    `);
}
```

## 14.3 Security Monitoring

```typescript
// lib/security-monitoring.ts
// Monitoring per rilevare attività sospette

import { prisma } from './db';
import { redis } from './redis';

interface SecurityEvent {
  type: string;
  severity: 'low' | 'medium' | 'high' | 'critical';
  userId?: string;
  ip?: string;
  details: Record<string, any>;
}

export async function logSecurityEvent(event: SecurityEvent) {
  // Salva evento
  await prisma.securityEvent.create({
    data: {
      type: event.type,
      severity: event.severity,
      userId: event.userId,
      ipAddress: event.ip,
      details: event.details,
      timestamp: new Date()
    }
  });
  
  // Incrementa contatore per alerting
  const key = `security:${event.type}:${event.ip || 'unknown'}`;
  const count = await redis.incr(key);
  await redis.expire(key, 3600); // 1 ora
  
  // Alert se threshold superato
  const thresholds: Record<string, number> = {
    'LOGIN_FAILED': 10,
    'INVALID_TOKEN': 20,
    'RATE_LIMIT_EXCEEDED': 50,
    'SUSPICIOUS_USER_AGENT': 5,
    'SQL_INJECTION_ATTEMPT': 1,
    'XSS_ATTEMPT': 1
  };
  
  if (count >= (thresholds[event.type] || 10)) {
    await triggerSecurityAlert(event, count);
  }
}

async function triggerSecurityAlert(event: SecurityEvent, count: number) {
  console.error(`🚨 SECURITY ALERT: ${event.type} - ${count} occurrences`);
  
  // Invia alert (Slack, email, PagerDuty)
  if (process.env.SLACK_SECURITY_WEBHOOK) {
    await fetch(process.env.SLACK_SECURITY_WEBHOOK, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        text: `🚨 Security Alert: ${event.type}`,
        blocks: [
          {
            type: 'section',
            text: {
              type: 'mrkdwn',
              text: `*Security Alert*\n*Type:* ${event.type}\n*Severity:* ${event.severity}\n*Count:* ${count}\n*IP:* ${event.ip || 'Unknown'}`
            }
          }
        ]
      })
    });
  }
  
  // Auto-block per eventi critici
  if (event.severity === 'critical' && event.ip) {
    await redis.set(`blocked:ip:${event.ip}`, 'auto-blocked', 'EX', 3600);
  }
}

// Uso nei middleware
export function securityEventMiddleware(type: string) {
  return (req: Request, res: Response, next: () => void) => {
    // Detect suspicious patterns
    const patterns = {
      'SQL_INJECTION_ATTEMPT': /(\b(union|select|insert|update|delete|drop|create|alter|exec|execute)\b.*\b(from|into|set|table|database)\b)|(['"];\s*--)|(\bor\b\s+\d+\s*=\s*\d+)/i,
      'XSS_ATTEMPT': /<script|javascript:|on\w+\s*=/i,
      'PATH_TRAVERSAL': /\.\.\//
    };
    
    const body = JSON.stringify(req.body || {});
    const url = req.url;
    
    for (const [eventType, pattern] of Object.entries(patterns)) {
      if (pattern.test(body) || pattern.test(url)) {
        logSecurityEvent({
          type: eventType,
          severity: 'critical',
          ip: getClientIp(req),
          details: { url, bodySnippet: body.slice(0, 200) }
        });
      }
    }
    
    next();
  };
}
```

## 14.4 Post-Incident Template

```markdown
# Security Incident Report

## Summary
- **Incident ID:** INC-2026-001
- **Date Detected:** YYYY-MM-DD HH:MM UTC
- **Date Resolved:** YYYY-MM-DD HH:MM UTC
- **Severity:** Critical / High / Medium / Low
- **Status:** Resolved / In Progress / Monitoring

## Timeline
| Time | Event |
|------|-------|
| HH:MM | Initial detection |
| HH:MM | Team notified |
| HH:MM | Containment started |
| HH:MM | Root cause identified |
| HH:MM | Fix deployed |
| HH:MM | Incident resolved |

## Impact
- **Users affected:** X
- **Data exposed:** Description
- **Systems affected:** List
- **Duration:** X hours

## Root Cause
[Detailed description of what caused the incident]

## Actions Taken
1. [Containment action]
2. [Eradication action]
3. [Recovery action]

## Lessons Learned
- [What could have prevented this]
- [What worked well]
- [What needs improvement]

## Follow-up Actions
- [ ] Action item 1 - Owner - Due date
- [ ] Action item 2 - Owner - Due date
- [ ] Action item 3 - Owner - Due date

## Communication
- [ ] Users notified
- [ ] Stakeholders notified
- [ ] Regulatory bodies notified (if required)
```

"""


# ═══════════════════════════════════════════════════════════════════════════════
# APPENDICE: QUICK REFERENCE
# ═══════════════════════════════════════════════════════════════════════════════

QUICK_REFERENCE = """

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ SECURITY QUICK REFERENCE                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ PASSWORD HASHING                                                            │
│ bcrypt.hash(password, 12)          // Cost factor ≥ 12                     │
│ argon2.hash(password, {type: argon2id, memoryCost: 65536})                 │
│                                                                             │
│ JWT CONFIGURATION                                                           │
│ Access Token:  15 min expiry, in memory                                    │
│ Refresh Token: 7 days expiry, HttpOnly cookie                              │
│ Algorithm: HS256 (symmetric) o RS256 (asymmetric)                          │
│                                                                             │
│ SECURITY HEADERS                                                            │
│ Strict-Transport-Security: max-age=63072000; includeSubDomains; preload    │
│ Content-Security-Policy: default-src 'self'; script-src 'self'             │
│ X-Frame-Options: DENY                                                       │
│ X-Content-Type-Options: nosniff                                            │
│ Referrer-Policy: strict-origin-when-cross-origin                           │
│                                                                             │
│ COOKIE FLAGS                                                                │
│ HttpOnly: true     // No JS access                                         │
│ Secure: true       // HTTPS only                                           │
│ SameSite: 'strict' // CSRF protection                                      │
│ Path: '/api/auth'  // Limit scope                                          │
│                                                                             │
│ RATE LIMITS (suggeriti)                                                    │
│ Login: 5 req / 15 min / IP                                                 │
│ API generale: 100 req / min / IP                                           │
│ Upload: 10 req / ora / utente                                              │
│                                                                             │
│ INPUT VALIDATION                                                            │
│ z.string().min(1).max(100)         // Lunghezza                            │
│ z.string().email()                  // Email                                │
│ z.string().regex(/^[a-zA-Z0-9]+$/) // Pattern                              │
│ sanitizeHtml(input)                 // XSS prevention                       │
│                                                                             │
│ OWASP TOP 10:2025                                                          │
│ A01: Broken Access Control     → Verifica ownership, RBAC                  │
│ A02: Security Misconfiguration → Hardening, no defaults                    │
│ A03: Supply Chain Failures     → npm audit, Snyk                           │
│ A04: Insecure Design           → Threat modeling                           │
│ A05: Cryptographic Failures    → Strong encryption, HTTPS                  │
│ A06: Injection                 → Parameterized queries                     │
│ A07: Auth Failures             → MFA, strong passwords                     │
│ A08: Integrity Failures        → Signed updates, SRI                       │
│ A09: Logging Failures          → Audit logs, monitoring                    │
│ A10: SSRF                      → URL validation, blocklist                 │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

"""

# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO SICUREZZA
# ═══════════════════════════════════════════════════════════════════════════════

SUPPLEMENT: NEXT.JS 14 SECURITY PATTERNS
S1. SERVER ACTIONS SECURITY

✅ Input Validation Obbligatoria con Zod: Ogni Server Action deve validare input con Zod, anche se i dati provengono da form. La validazione lato client non è sufficiente.

typescript
Copia
Scarica
// lib/validations/user.ts
import { z } from 'zod';

export const createUserSchema = z.object({
  name: z.string().min(1, 'Name is required').max(100),
  email: z.string().email('Invalid email'),
  role: z.enum(['USER', 'ADMIN', 'EDITOR']),
  age: z.number().int().min(18).optional(),
});

export type CreateUserInput = z.infer<typeof createUserSchema>;

✅ Authentication Check in Ogni Server Action: Verificare sempre la sessione all'inizio di ogni Server Action.

typescript
Copia
Scarica
// app/actions/user-actions.ts
'use server';

import { auth } from '@/auth';
import { createUserSchema } from '@/lib/validations/user';
import { rateLimit } from '@/lib/rate-limit';
import { redirect } from 'next/navigation';

export async function createUser(formData: FormData) {
  // 1. Authentication check
  const session = await auth();
  if (!session?.user) {
    throw new Error('Unauthorized');
  }

  // 2. Rate limiting per user/session
  const identifier = session.user.id || session.user.email;
  const { success } = await rateLimit.limit(identifier);
  if (!success) {
    throw new Error('Rate limit exceeded');
  }

  // 3. Input validation
  const rawData = Object.fromEntries(formData);
  const validatedData = createUserSchema.parse({
    ...rawData,
    age: rawData.age ? parseInt(rawData.age as string) : undefined,
  });

  // 4. Authorization check
  if (session.user.role !== 'ADMIN') {
    throw new Error('Insufficient permissions');
  }

  // 5. Business logic
  try {
    // Database operation...
    return { success: true };
  } catch (error) {
    console.error('User creation failed:', error);
    throw new Error('Failed to create user');
  }
}

✅ Rate Limiting per Server Actions: Implementare rate limiting specifico per prevenire abuso.

typescript
Copia
Scarica
// lib/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

export const rateLimit = {
  createUser: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, '60 s'), // 5 requests per 60 seconds
    analytics: true,
    prefix: 'ratelimit:createUser',
  }),
  general: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, '10 s'),
    analytics: true,
    prefix: 'ratelimit:serverActions',
  }),
};

⚡ CSRF Protection: Next.js 14 fornisce CSRF protection automatica per Server Actions. Tuttavia:

typescript
Copia
Scarica
// app/layout.tsx - Verificare configurazione CSP
export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <head>
        <meta
          name="csrf-token"
          content={/* Generate token from server if needed */}
        />
      </head>
      <body>{children}</body>
    </html>
  );
}

// Per richieste API esterne, validare Origin header:
function validateOrigin(request: Request) {
  const origin = request.headers.get('origin');
  const validOrigins = [
    'https://yourdomain.com',
    'https://app.yourdomain.com',
  ];
  
  if (origin && !validOrigins.includes(origin)) {
    throw new Error('Invalid origin');
  }
}

✅ File Upload Security in Server Actions:

typescript
Copia
Scarica
// app/actions/upload-actions.ts
'use server';

import { auth } from '@/auth';
import { writeFile } from 'fs/promises';
import { join } from 'path';
import { createHash } from 'crypto';
import { rm } from 'fs/promises';

const ALLOWED_MIME_TYPES = [
  'image/jpeg',
  'image/png',
  'image/webp',
  'application/pdf',
];
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

export async function secureUpload(formData: FormData) {
  const session = await auth();
  if (!session?.user) throw new Error('Unauthorized');

  const file = formData.get('file') as File;
  if (!file) throw new Error('No file provided');

  // 1. Validate file size
  if (file.size > MAX_FILE_SIZE) {
    throw new Error('File too large');
  }

  // 2. Validate MIME type
  if (!ALLOWED_MIME_TYPES.includes(file.type)) {
    throw new Error('Invalid file type');
  }

  // 3. Validate file content (magic bytes)
  const buffer = await file.arrayBuffer();
  const uint8Array = new Uint8Array(buffer);
  
  // Check first bytes for actual file type
  const magicBytes = uint8Array.slice(0, 4);
  const hexSignature = Array.from(magicBytes)
    .map(b => b.toString(16).padStart(2, '0'))
    .join('')
    .toUpperCase();

  const validSignatures = {
    'FFD8FFE0': 'image/jpeg', // JPEG
    '89504E47': 'image/png',  // PNG
    '25504446': 'application/pdf', // PDF
  };

  if (!Object.values(validSignatures).includes(file.type)) {
    throw new Error('File signature mismatch');
  }

  // 4. Generate safe filename
  const fileHash = createHash('sha256')
    .update(uint8Array)
    .digest('hex')
    .slice(0, 16);
  
  const safeFileName = `${fileHash}-${Date.now()}${getExtension(file.type)}`;
  
  // 5. Save to temporary directory with restricted permissions
  const uploadDir = join(process.cwd(), 'uploads', session.user.id);
  const filePath = join(uploadDir, safeFileName);
  
  await writeFile(filePath, Buffer.from(buffer));
  
  // 6. Virus scanning (integrate with ClamAV or similar)
  // await scanForViruses(filePath);
  
  return { success: true, fileName: safeFileName };
}

function getExtension(mimeType: string): string {
  const extensions: Record<string, string> = {
    'image/jpeg': '.jpg',
    'image/png': '.png',
    'image/webp': '.webp',
    'application/pdf': '.pdf',
  };
  return extensions[mimeType] || '.bin';
}

✅ Secure Action Wrapper Riusabile:

typescript
Copia
Scarica
// lib/secure-action.ts
'use server';

import { auth } from '@/auth';
import { z, ZodSchema } from 'zod';
import { rateLimit } from './rate-limit';

type ActionResult<T> = {
  success: boolean;
  data?: T;
  error?: string;
};

type ActionOptions<T> = {
  schema: ZodSchema<T>;
  rateLimitKey?: string;
  requireAuth?: boolean;
  requiredRoles?: string[];
};

export async function secureAction<T, R>(
  options: ActionOptions<T>,
  action: (data: T, userId: string) => Promise<R>
): Promise<ActionResult<R>> {
  try {
    // 1. Authentication
    const session = await auth();
    if (options.requireAuth && !session?.user) {
      return { success: false, error: 'Unauthorized' };
    }

    // 2. Role-based authorization
    if (options.requiredRoles && session?.user?.role) {
      const hasRole = options.requiredRoles.includes(session.user.role);
      if (!hasRole) {
        return { success: false, error: 'Insufficient permissions' };
      }
    }

    // 3. Rate limiting
    if (options.rateLimitKey && session?.user) {
      const identifier = session.user.id || session.user.email;
      const { success } = await rateLimit.general.limit(identifier);
      if (!success) {
        return { success: false, error: 'Rate limit exceeded' };
      }
    }

    // 4. Schema validation (must be called with validated data)
    // Note: This wrapper expects pre-validated data
    // Use with: await secureAction({...}, (validatedData) => {...})

    // 5. Execute action
    const userId = session?.user?.id || 'anonymous';
    const result = await action(null as T, userId);
    
    return { success: true, data: result };
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { success: false, error: error.errors[0].message };
    }
    
    // Don't expose internal errors
    console.error('Action failed:', error);
    return { success: false, error: 'Action failed' };
  }
}

// Uso:
const createPostSchema = z.object({
  title: z.string().min(1).max(100),
  content: z.string().min(1),
});

export async function createPost(formData: FormData) {
  const rawData = Object.fromEntries(formData);
  const validatedData = createPostSchema.parse(rawData);
  
  return secureAction(
    {
      schema: createPostSchema,
      requireAuth: true,
      requiredRoles: ['ADMIN', 'EDITOR'],
      rateLimitKey: 'createPost',
    },
    async (data, userId) => {
      // Business logic here
      return { id: '123', ...data };
    }
  )(validatedData);
}
S2. MIDDLEWARE SECURITY LAYER

✅ Authentication Middleware Pattern:

typescript
Copia
Scarica
// middleware.ts
import { NextResponse, NextRequest } from 'next/server';
import { getToken } from 'next-auth/jwt';
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(100, '1 m'),
  analytics: true,
});

// IP-based geoblocking
const BLOCKED_COUNTRIES = ['CU', 'IR', 'KP', 'SY', 'RU'];
const ALLOWED_COUNTRIES = ['US', 'CA', 'GB', 'DE', 'FR']; // Se necessario

export async function middleware(request: NextRequest) {
  const response = NextResponse.next();
  const pathname = request.nextUrl.pathname;
  
  // 1. IP-based rate limiting
  const ip = request.ip ?? '127.0.0.1';
  const { success, limit, reset, remaining } = await ratelimit.limit(ip);
  
  if (!success) {
    return new NextResponse('Rate limit exceeded', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    });
  }
  
  // 2. Geoblocking (utilizzando Cloudflare o similar)
  const country = request.geo?.country;
  if (country && BLOCKED_COUNTRIES.includes(country)) {
    return new NextResponse('Access denied from your region', { status: 403 });
  }
  
  // 3. Request logging per audit trail
  const logEntry = {
    timestamp: new Date().toISOString(),
    ip,
    country,
    userAgent: request.headers.get('user-agent'),
    method: request.method,
    url: request.url,
    userId: 'anonymous', // Sarà aggiornato dopo auth
  };
  
  // Log asynchronously per non bloccare la richiesta
  console.log(JSON.stringify(logEntry));
  
  // 4. Authentication check per rotte protette
  const isAuthRoute = pathname.startsWith('/dashboard') ||
                     pathname.startsWith('/api/protected') ||
                     pathname.startsWith('/admin');
  
  if (isAuthRoute) {
    const token = await getToken({
      req: request,
      secret: process.env.NEXTAUTH_SECRET!,
    });
    
    if (!token) {
      const redirectUrl = new URL('/auth/login', request.url);
      redirectUrl.searchParams.set('callbackUrl', encodeURI(request.url));
      return NextResponse.redirect(redirectUrl);
    }
    
    // Update log entry with user ID
    logEntry.userId = token.sub || 'authenticated';
  }
  
  // 5. Security headers
  response.headers.set('X-Content-Type-Options', 'nosniff');
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-XSS-Protection', '1; mode=block');
  response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  
  // 6. CORS headers per API routes
  if (pathname.startsWith('/api/')) {
    response.headers.set('Access-Control-Allow-Credentials', 'true');
    response.headers.set('Access-Control-Allow-Origin', 
      process.env.NEXT_PUBLIC_APP_URL || 'https://yourdomain.com');
    response.headers.set('Access-Control-Allow-Methods', 
      'GET,POST,PUT,DELETE,OPTIONS');
    response.headers.set('Access-Control-Allow-Headers',
      'Content-Type, Authorization, X-API-Key');
  }
  
  return response;
}

// Configurazione matcher
export const config = {
  matcher: [
    /*
     * Match all request paths except:
     * 1. /api/auth (NextAuth endpoints)
     * 2. /_next/static (static files)
     * 3. /_next/image (image optimization files)
     * 4. /favicon.ico (favicon file)
     * 5. /public (public files)
     */
    '/((?!api/auth|_next/static|_next/image|favicon.ico|public).*)',
  ],
};

✅ Role-Based Access Control in Middleware:

typescript
Copia
Scarica
// lib/middleware/rbac.ts
import { NextRequest, NextResponse } from 'next/server';
import { getToken } from 'next-auth/jwt';

type RoutePermission = {
  path: string;
  methods: string[];
  roles: string[];
};

const routePermissions: RoutePermission[] = [
  {
    path: '/api/admin',
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    roles: ['ADMIN', 'SUPER_ADMIN'],
  },
  {
    path: '/api/users',
    methods: ['GET', 'POST'],
    roles: ['ADMIN', 'MANAGER'],
  },
  {
    path: '/api/users',
    methods: ['PUT', 'DELETE'],
    roles: ['ADMIN'],
  },
  {
    path: '/dashboard',
    methods: ['GET'],
    roles: ['USER', 'ADMIN', 'MANAGER'],
  },
  {
    path: '/dashboard/admin',
    methods: ['GET'],
    roles: ['ADMIN', 'SUPER_ADMIN'],
  },
];

export async function rbacMiddleware(request: NextRequest) {
  const pathname = request.nextUrl.pathname;
  const method = request.method;
  
  // Trova la regola applicabile
  const applicableRule = routePermissions.find(rule => {
    const pathMatches = pathname.startsWith(rule.path);
    const methodMatches = rule.methods.includes(method);
    return pathMatches && methodMatches;
  });
  
  if (!applicableRule) {
    // Nessuna regola specifica, permettere l'accesso
    return NextResponse.next();
  }
  
  // Verifica autenticazione e ruoli
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET!,
  });
  
  if (!token) {
    return new NextResponse('Unauthorized', { status: 401 });
  }
  
  const userRole = token.role as string;
  const hasPermission = applicableRule.roles.includes(userRole);
  
  if (!hasPermission) {
    return new NextResponse('Forbidden', { status: 403 });
  }
  
  return NextResponse.next();
}

// middleware.ts aggiornato
import { rbacMiddleware } from '@/lib/middleware/rbac';

export async function middleware(request: NextRequest) {
  // ... existing rate limiting, geoblocking, etc.
  
  // Apply RBAC
  const rbacResponse = await rbacMiddleware(request);
  if (rbacResponse.status !== 200) {
    return rbacResponse;
  }
  
  return NextResponse.next();
}

✅ IP-Based Rate Limiting Stratificato:

typescript
Copia
Scarica
// lib/middleware/rate-limiting.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL!,
  token: process.env.UPSTASH_REDIS_REST_TOKEN!,
});

// Rate limiters per diversi endpoint
export const rateLimiters = {
  // Limiter generico per tutte le richieste
  global: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(1000, '1 m'),
    prefix: 'ratelimit:global',
  }),
  
  // Limiter per login attempts
  auth: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, '5 m'),
    prefix: 'ratelimit:auth',
  }),
  
  // Limiter per API endpoints
  api: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(100, '1 m'),
    prefix: 'ratelimit:api',
  }),
  
  // Limiter per form submissions
  forms: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, '1 m'),
    prefix: 'ratelimit:forms',
  }),
};

export async function applyRateLimit(
  request: NextRequest,
  limiter: 'global' | 'auth' | 'api' | 'forms'
) {
  const ip = request.ip ?? '127.0.0.1';
  const pathname = request.nextUrl.pathname;
  
  // Usa user ID se autenticato, altrimenti IP
  let identifier = ip;
  
  // Per login endpoints, usa sempre IP
  if (pathname.includes('/api/auth') || pathname.includes('/auth/login')) {
    identifier = `auth:${ip}`;
  }
  
  const { success, limit, reset, remaining } = await rateLimiters[limiter]
    .limit(identifier);
  
  const response = success ? NextResponse.next() : 
    new NextResponse('Rate limit exceeded', { status: 429 });
  
  response.headers.set('X-RateLimit-Limit', limit.toString());
  response.headers.set('X-RateLimit-Remaining', remaining.toString());
  response.headers.set('X-RateLimit-Reset', reset.toString());
  
  return { success, response };
}

✅ Request Logging per Audit Trail:

typescript
Copia
Scarica
// lib/middleware/audit-log.ts
import { NextRequest, NextResponse } from 'next/server';

interface AuditLogEntry {
  timestamp: string;
  userId?: string;
  sessionId?: string;
  ip: string;
  userAgent?: string;
  method: string;
  url: string;
  path: string;
  query: Record<string, string>;
  statusCode: number;
  requestBody?: any;
  responseTime: number;
  country?: string;
  city?: string;
  userRole?: string;
}

export class AuditLogger {
  private static async logToDatabase(entry: AuditLogEntry) {
    // Implementare logging su database (PostgreSQL, MongoDB, etc.)
    // Usare batch inserts per performance
    try {
      // Esempio con Prisma
      // await prisma.auditLog.create({ data: entry });
    } catch (error) {
      console.error('Failed to log audit trail:', error);
    }
  }
  
  static async logRequest(
    request: NextRequest,
    response: NextResponse,
    userInfo?: { id?: string; role?: string; sessionId?: string }
  ) {
    const startTime = Date.now();
    
    // Non loggare richieste statiche
    const pathname = request.nextUrl.pathname;
    if (pathname.startsWith('/_next/') || 
        pathname.startsWith('/favicon.ico') ||
        pathname.endsWith('.css') || 
        pathname.endsWith('.js')) {
      return;
    }
    
    // Clonare la response per leggere il body
    const clonedResponse = response.clone();
    const responseTime = Date.now() - startTime;
    
    const logEntry: AuditLogEntry = {
      timestamp: new Date().toISOString(),
      userId: userInfo?.id,
      sessionId: userInfo?.sessionId,
      ip: request.ip ?? 'unknown',
      userAgent: request.headers.get('user-agent') || undefined,
      method: request.method,
      url: request.url,
      path: pathname,
      query: Object.fromEntries(request.nextUrl.searchParams),
      statusCode: clonedResponse.status,
      responseTime,
      country: request.geo?.country,
      city: request.geo?.city,
      userRole: userInfo?.role,
    };
    
    // Loggare body solo per POST/PUT e non per file upload
    if (['POST', 'PUT', 'PATCH'].includes(request.method)) {
      const contentType = request.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        try {
          const clone = request.clone();
          const body = await clone.json();
          // Non loggare password o dati sensibili
          logEntry.requestBody = this.sanitizeBody(body);
        } catch {
          // Non loggare se non è JSON
        }
      }
    }
    
    // Log asincrono per non bloccare la response
    setTimeout(() => this.logToDatabase(logEntry), 0);
  }
  
  private static sanitizeBody(body: any): any {
    const sensitiveFields = [
      'password',
      'token',
      'secret',
      'apiKey',
      'creditCard',
      'cvv',
      'ssn',
    ];
    
    const sanitized = { ...body };
    sensitiveFields.forEach(field => {
      if (sanitized[field]) {
        sanitized[field] = '***REDACTED***';
      }
    });
    
    return sanitized;
  }
}
S3. ROUTE HANDLER SECURITY

✅ API Authentication Pattern:

typescript
Copia
Scarica
// app/api/protected/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { validateApiKey, validateBearerToken } from '@/lib/auth/api-auth';

// Schema di validazione per input
const createResourceSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().max(500).optional(),
  tags: z.array(z.string()).max(10).optional(),
});

// Tipi di autenticazione supportati
type AuthType = 'bearer' | 'api-key' | 'session';

export async function GET(request: NextRequest) {
  try {
    // 1. Authentication
    const authResult = await authenticateRequest(request, ['bearer', 'api-key']);
    if (!authResult.authenticated) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    // 2. Authorization (se necessario)
    if (authResult.userId) {
      const canRead = await checkPermission(authResult.userId, 'resource:read');
      if (!canRead) {
        return NextResponse.json(
          { error: 'Forbidden' },
          { status: 403 }
        );
      }
    }
    
    // 3. Business logic
    const resources = await getResources();
    
    return NextResponse.json({ data: resources });
  } catch (error) {
    console.error('GET /api/protected failed:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

export async function POST(request: NextRequest) {
  try {
    // 1. Authentication
    const authResult = await authenticateRequest(request, ['bearer', 'api-key']);
    if (!authResult.authenticated) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    // 2. Input validation
    let body;
    try {
      body = await request.json();
    } catch {
      return NextResponse.json(
        { error: 'Invalid JSON body' },
        { status: 400 }
      );
    }
    
    const validationResult = createResourceSchema.safeParse(body);
    if (!validationResult.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: validationResult.error.errors },
        { status: 400 }
      );
    }
    
    // 3. Rate limiting
    const identifier = authResult.apiKeyId || authResult.userId || request.ip;
    const isLimited = await checkRateLimit(identifier, 'api-create');
    if (isLimited) {
      return NextResponse.json(
        { error: 'Rate limit exceeded' },
        { status: 429 }
      );
    }
    
    // 4. Business logic
    const resource = await createResource(validationResult.data, authResult.userId);
    
    return NextResponse.json(
      { data: resource },
      { status: 201 }
    );
  } catch (error) {
    console.error('POST /api/protected failed:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// Funzione di autenticazione riusabile
async function authenticateRequest(
  request: NextRequest,
  allowedMethods: AuthType[]
): Promise<{
  authenticated: boolean;
  userId?: string;
  apiKeyId?: string;
}> {
  // Prova Bearer token
  if (allowedMethods.includes('bearer')) {
    const authHeader = request.headers.get('authorization');
    if (authHeader?.startsWith('Bearer ')) {
      const token = authHeader.slice(7);
      const tokenData = await validateBearerToken(token);
      if (tokenData) {
        return {
          authenticated: true,
          userId: tokenData.userId,
        };
      }
    }
  }
  
  // Prova API Key
  if (allowedMethods.includes('api-key')) {
    const apiKey = request.headers.get('x-api-key');
    if (apiKey) {
      const keyData = await validateApiKey(apiKey);
      if (keyData) {
        return {
          authenticated: true,
          userId: keyData.userId,
          apiKeyId: keyData.keyId,
        };
      }
    }
  }
  
  // Prova session cookie (per chiamate dal frontend)
  if (allowedMethods.includes('session')) {
    // Implementare verifica sessione
  }
  
  return { authenticated: false };
}

// Middleware pattern per route handlers
export function withAuth(
  handler: Function,
  options: { methods: AuthType[]; requiredPermissions?: string[] }
) {
  return async (request: NextRequest, ...args: any[]) => {
    const authResult = await authenticateRequest(request, options.methods);
    
    if (!authResult.authenticated) {
      return NextResponse.json(
        { error: 'Unauthorized' },
        { status: 401 }
      );
    }
    
    // Check permissions se specificate
    if (options.requiredPermissions && authResult.userId) {
      const hasPermissions = await checkPermissions(
        authResult.userId,
        options.requiredPermissions
      );
      if (!hasPermissions) {
        return NextResponse.json(
          { error: 'Forbidden' },
          { status: 403 }
        );
      }
    }
    
    return handler(request, ...args, authResult);
  };
}

✅ Request Validation Middleware Pattern:

typescript
Copia
Scarica
// lib/api/middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { z, ZodSchema } from 'zod';

type ValidationConfig = {
  body?: ZodSchema;
  query?: ZodSchema;
  params?: ZodSchema;
  headers?: ZodSchema;
};

export function validateRequest(config: ValidationConfig) {
  return (
    handler: (request: NextRequest, parsed: any) => Promise<NextResponse>
  ) => {
    return async (request: NextRequest) => {
      try {
        const parsed: any = {};
        
        // Validate body
        if (config.body) {
          try {
            const body = await request.json();
            parsed.body = config.body.parse(body);
          } catch (error) {
            if (error instanceof z.ZodError) {
              return NextResponse.json(
                { error: 'Invalid request body', details: error.errors },
                { status: 400 }
              );
            }
            return NextResponse.json(
              { error: 'Invalid JSON body' },
              { status: 400 }
            );
          }
        }
        
        // Validate query parameters
        if (config.query) {
          const queryParams = Object.fromEntries(request.nextUrl.searchParams);
          try {
            parsed.query = config.query.parse(queryParams);
          } catch (error) {
            if (error instanceof z.ZodError) {
              return NextResponse.json(
                { error: 'Invalid query parameters', details: error.errors },
                { status: 400 }
              );
            }
          }
        }
        
        // Validate headers
        if (config.headers) {
          const headers: Record<string, string> = {};
          request.headers.forEach((value, key) => {
            headers[key] = value;
          });
          try {
            parsed.headers = config.headers.parse(headers);
          } catch (error) {
            if (error instanceof z.ZodError) {
              return NextResponse.json(
                { error: 'Invalid headers', details: error.errors },
                { status: 400 }
              );
            }
          }
        }
        
        return handler(request, parsed);
      } catch (error) {
        console.error('Validation middleware error:', error);
        return NextResponse.json(
          { error: 'Internal server error' },
          { status: 500 }
        );
      }
    };
  };
}

// Esempio d'uso:
const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

const querySchema = z.object({
  page: z.string().regex(/^\d+$/).transform(Number).optional(),
  limit: z.string().regex(/^\d+$/).transform(Number).max(100).optional(),
});

export const POST = validateRequest({
  body: createUserSchema,
  query: querySchema,
})(async (request: NextRequest, parsed) => {
  // parsed.body e parsed.query sono già validati
  const { name, email } = parsed.body;
  const { page = 1, limit = 10 } = parsed.query;
  
  // Business logic...
  return NextResponse.json({ success: true });
});

✅ CORS Configuration Avanzata:

typescript
Copia
Scarica
// lib/api/cors.ts
import { NextRequest, NextResponse } from 'next/server';

type CorsOptions = {
  origin?: string | string[] | RegExp | ((origin: string) => boolean);
  methods?: string[];
  allowedHeaders?: string[];
  exposedHeaders?: string[];
  credentials?: boolean;
  maxAge?: number;
};

export function configureCORS(options: CorsOptions = {}) {
  const {
    origin = '*',
    methods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
    allowedHeaders = ['Content-Type', 'Authorization', 'X-API-Key'],
    exposedHeaders = [],
    credentials = false,
    maxAge = 86400, // 24 hours
  } = options;
  
  return function corsMiddleware(
    handler: (request: NextRequest) => Promise<NextResponse>
  ) {
    return async (request: NextRequest) => {
      const response = request.method === 'OPTIONS'
        ? new NextResponse(null, { status: 204 })
        : await handler(request);
      
      const requestOrigin = request.headers.get('origin');
      
      // Determine allowed origin
      let allowedOrigin = '';
      if (typeof origin === 'function') {
        allowedOrigin = origin(requestOrigin || '') ? requestOrigin || '*' : '';
      } else if (typeof origin === 'string') {
        allowedOrigin = origin;
      } else if (Array.isArray(origin)) {
        allowedOrigin = origin.includes(requestOrigin || '') ? requestOrigin || '' : origin[0];
      } else if (origin instanceof RegExp) {
        allowedOrigin = origin.test(requestOrigin || '') ? requestOrigin || '' : '';
      }
      
      // Set CORS headers
      response.headers.set('Access-Control-Allow-Origin', allowedOrigin);
      response.headers.set('Access-Control-Allow-Methods', methods.join(', '));
      response.headers.set('Access-Control-Allow-Headers', allowedHeaders.join(', '));
      
      if (exposedHeaders.length > 0) {
        response.headers.set('Access-Control-Expose-Headers', exposedHeaders.join(', '));
      }
      
      if (credentials) {
        response.headers.set('Access-Control-Allow-Credentials', 'true');
      }
      
      response.headers.set('Access-Control-Max-Age', maxAge.toString());
      
      // Security headers aggiuntivi
      response.headers.set('Vary', 'Origin');
      
      return response;
    };
  };
}

// Uso in route handler:
export const GET = configureCORS({
  origin: ['https://app.example.com', 'https://admin.example.com'],
  credentials: true,
  methods: ['GET', 'OPTIONS'],
})(async (request: NextRequest) => {
  return NextResponse.json({ data: 'protected data' });
});

✅ Security Response Headers per Route Handlers:

typescript
Copia
Scarica
// lib/api/security-headers.ts
import { NextResponse } from 'next/server';

export function withSecurityHeaders(
  response: NextResponse,
  options: {
    csp?: string;
    permissionsPolicy?: Record<string, string[]>;
    featurePolicy?: Record<string, string[]>;
  } = {}
): NextResponse {
  // Content Security Policy
  const csp = options.csp || `
    default-src 'self';
    script-src 'self' 'unsafe-inline' 'unsafe-eval' https://cdn.example.com;
    style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
    img-src 'self' data: https://*.example.com;
    font-src 'self' https://fonts.gstatic.com;
    connect-src 'self' https://api.example.com wss://ws.example.com;
    frame-src 'none';
    object-src 'none';
    base-uri 'self';
    form-action 'self';
    frame-ancestors 'none';
    block-all-mixed-content;
    upgrade-insecure-requests;
  `.replace(/\s+/g, ' ').trim();
  
  response.headers.set('Content-Security-Policy', csp);
  
  // Permissions Policy / Feature Policy
  const permissionsPolicy = options.permissionsPolicy || {
    'camera': ['none'],
    'microphone': ['none'],
    'geolocation': ['none'],
    'payment': ['none'],
    'usb': ['none'],
    'autoplay': ['self'],
    'fullscreen': ['self'],
  };
  
  const permissionsPolicyHeader = Object.entries(permissionsPolicy)
    .map(([feature, origins]) => `${feature}=(${origins.join(' ')})`)
    .join(', ');
  
  response.headers.set('Permissions-Policy', permissionsPolicyHeader);
  
  // Security headers standard
  const securityHeaders = {
    'X-Content-Type-Options': 'nosniff',
    'X-Frame-Options': 'DENY',
    'X-XSS-Protection': '1; mode=block',
    'Referrer-Policy': 'strict-origin-when-cross-or

Route && !isAdminRoute) {
    // Route non specificata - permettere l'accesso
    return NextResponse.next();
  }
  
  // Verifica autenticazione
  const token = await getToken({
    req: request,
    secret: process.env.NEXTAUTH_SECRET!,
  });
  
  if (!token) {
    // Redirect al login con return URL
    const loginUrl = new URL('/auth/login', request.url);
    loginUrl.searchParams.set('callbackUrl', encodeURI(request.url));
    return NextResponse.redirect(loginUrl);
  }
  
  // Verifica se l'account è attivo
  if (token.accountStatus !== 'ACTIVE') {
    const blockedUrl = new URL('/auth/blocked', request.url);
    return NextResponse.redirect(blockedUrl);
  }
  
  // Verifica ruoli per admin routes
  if (isAdminRoute) {
    const userRole = token.role as string;
    const isAdmin = ['ADMIN', 'SUPER_ADMIN'].