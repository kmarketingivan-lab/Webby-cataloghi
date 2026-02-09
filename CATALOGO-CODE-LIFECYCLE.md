# CATALOGO-CODE-LIFECYCLE

CATALOGO-CODE-LIFECYCLE: Next.js 14 + TypeScript
§ CODE ORGANIZATION
Feature-based structure
typescript
// ❌ ANTI-PATTERN: Type-based grouping
src/
  components/
    Button.tsx
    Header.tsx
    ProductCard.tsx
    UserProfile.tsx
  hooks/
    useAuth.ts
    useProducts.ts
  utils/
    formatters.ts
    validators.ts

// ✅ CORRETTO: Feature-based grouping
src/
  features/
    auth/
      components/
        LoginForm.tsx
        RegisterForm.tsx
      hooks/
        useAuth.ts
        useSession.ts
      utils/
        validators.ts
      index.ts // Public API
    products/
      components/
        ProductCard.tsx
        ProductGrid.tsx
      hooks/
        useProducts.ts
        useProductFilters.ts
      types/
        product.types.ts
      index.ts
    checkout/
      components/
        CartSummary.tsx
        PaymentForm.tsx
      hooks/
        useCart.ts
        useCheckout.ts
  shared/
    components/  // Cross-feature components
      ui/
        Button.tsx
        Input.tsx
        Modal.tsx
      layout/
        Header.tsx
        Footer.tsx
    hooks/
      useDebounce.ts
      useLocalStorage.ts
    utils/
      formatters.ts
      constants.ts
  app/  // Next.js 14 App Router
    (public)/
    (auth)/
    products/
    checkout/
Module boundaries
typescript
// ✅ CORRETTO: Feature boundary con barrel exports
// features/products/index.ts
export { ProductCard } from './components/ProductCard'
export { useProducts } from './hooks/useProducts'
export type { Product, ProductFilters } from './types/product.types'

// ❌ Import diretto interno
import { ProductCard } from '../../features/products/components/ProductCard'

// ✅ Import dal public API
import { ProductCard } from '@/features/products'

// ✅ Dependency direction: High-level → Low-level
// shared/utils/formatters.ts (Low-level, indipendente)
export function formatCurrency(amount: number): string {
  return `$${amount.toFixed(2)}`
}

// features/checkout/hooks/useCart.ts (Mid-level)
import { formatCurrency } from '@/shared/utils'

// app/checkout/page.tsx (High-level, App Router)
import { useCart } from '@/features/checkout'
Public APIs
typescript
// ✅ CORRETTO: Barrel exports controllate
// features/auth/index.ts
export { LoginForm } from './components/LoginForm'
export { RegisterForm } from './components/RegisterForm'
export { useAuth } from './hooks/useAuth'
// ❌ Non esportare componenti interni
// export { AuthProviderInternal } from './providers/AuthProviderInternal'

// ✅ Type-only exports separati
export type { User, Session } from './types/auth.types'

// ✅ Re-export di utility contenute
export { AUTH_CONSTANTS } from './utils/constants'
§ REFACTORING PATTERNS
Extract component
typescript
// ❌ Prima: Componente sovraccarico
export function UserProfile({ user }: { user: User }) {
  return (
    <div className="profile-card">
      <div className="profile-header">
        <img 
          src={user.avatar} 
          alt={user.name}
          className="avatar"
        />
        <div>
          <h2>{user.name}</h2>
          <p>{user.email}</p>
        </div>
      </div>
      <div className="profile-stats">
        <div>
          <span className="stat-label">Posts</span>
          <span className="stat-value">{user.postCount}</span>
        </div>
        <div>
          <span className="stat-label">Followers</span>
          <span className="stat-value">{user.followers}</span>
        </div>
      </div>
      {/* 50+ linee aggiuntive */}
    </div>
  )
}

// ✅ Dopo: Componenti estratti
export function UserProfile({ user }: { user: User }) {
  return (
    <div className="profile-card">
      <ProfileHeader user={user} />
      <ProfileStats user={user} />
      <ProfileActions userId={user.id} />
    </div>
  )
}

// Componente estratto 1
function ProfileHeader({ user }: { user: User }) {
  return (
    <div className="profile-header">
      <Avatar src={user.avatar} alt={user.name} />
      <UserInfo name={user.name} email={user.email} />
    </div>
  )
}

// Componente estratto 2
function ProfileStats({ user }: { user: User }) {
  return (
    <div className="profile-stats">
      <StatItem label="Posts" value={user.postCount} />
      <StatItem label="Followers" value={user.followers} />
    </div>
  )
}
Extract hook
typescript
// ❌ Prima: Logica nel componente
export function ProductList() {
  const [products, setProducts] = useState<Product[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)
  const [filters, setFilters] = useState<ProductFilters>({})
  const [pagination, setPagination] = useState({ page: 1, limit: 20 })

  useEffect(() => {
    const fetchProducts = async () => {
      setLoading(true)
      try {
        const response = await fetch(
          `/api/products?page=${pagination.page}&limit=${pagination.limit}`
        )
        const data = await response.json()
        setProducts(data)
      } catch (err) {
        setError('Failed to fetch products')
      } finally {
        setLoading(false)
      }
    }
    
    fetchProducts()
  }, [pagination.page, pagination.limit])

  // 30+ linee di logica aggiuntiva...
}

// ✅ Dopo: Hook estratto
export function ProductList() {
  const {
    products,
    loading,
    error,
    filters,
    setFilters,
    pagination,
    setPagination,
    refetch
  } = useProducts({
    initialFilters: {},
    initialPagination: { page: 1, limit: 20 }
  })

  // Componente rimane dichiarativo
}

// Hook estratto
export function useProducts(options: UseProductsOptions = {}) {
  const [state, setState] = useState<ProductsState>({
    products: [],
    loading: false,
    error: null,
    filters: options.initialFilters || {},
    pagination: options.initialPagination || { page: 1, limit: 20 }
  })

  const fetchProducts = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }))
    
    try {
      const query = new URLSearchParams({
        ...state.filters,
        page: state.pagination.page.toString(),
        limit: state.pagination.limit.toString()
      })
      
      const response = await fetch(`/api/products?${query}`)
      const data = await response.json()
      
      setState(prev => ({ ...prev, products: data, loading: false }))
    } catch (err) {
      setState(prev => ({ 
        ...prev, 
        error: 'Failed to fetch products', 
        loading: false 
      }))
    }
  }, [state.filters, state.pagination])

  useEffect(() => {
    fetchProducts()
  }, [fetchProducts])

  return {
    ...state,
    setFilters: (filters: ProductFilters) => 
      setState(prev => ({ ...prev, filters })),
    setPagination: (pagination: Pagination) =>
      setState(prev => ({ ...prev, pagination })),
    refetch: fetchProducts
  }
}
Extract utility
typescript
// ❌ Prima: Logica duplicata
export function OrderSummary({ order }: { order: Order }) {
  const subtotal = order.items.reduce(
    (sum, item) => sum + (item.price * item.quantity), 
    0
  )
  const tax = subtotal * 0.08
  const shipping = subtotal > 100 ? 0 : 9.99
  const total = subtotal + tax + shipping
  
  return (
    <div>
      <div>Subtotal: ${subtotal.toFixed(2)}</div>
      <div>Tax: ${tax.toFixed(2)}</div>
      <div>Shipping: ${shipping.toFixed(2)}</div>
      <div>Total: ${total.toFixed(2)}</div>
    </div>
  )
}

// ✅ Dopo: Utility estratta
export function OrderSummary({ order }: { order: Order }) {
  const { subtotal, tax, shipping, total } = calculateOrderTotals(order)
  
  return (
    <div>
      <div>Subtotal: ${subtotal.toFixed(2)}</div>
      <div>Tax: ${tax.toFixed(2)}</div>
      <div>Shipping: ${shipping.toFixed(2)}</div>
      <div>Total: ${total.toFixed(2)}</div>
    </div>
  )
}

// Utility estratta - riutilizzabile e testabile
export function calculateOrderTotals(order: Order): OrderTotals {
  const subtotal = order.items.reduce(
    (sum, item) => sum + (item.price * item.quantity), 
    0
  )
  const tax = subtotal * TAX_RATE
  const shipping = calculateShipping(subtotal)
  const total = subtotal + tax + shipping
  
  return { subtotal, tax, shipping, total }
}

export function calculateShipping(subtotal: number): number {
  return subtotal > FREE_SHIPPING_THRESHOLD ? 0 : STANDARD_SHIPPING_COST
}
Rename and move
typescript
// ❌ Prima: Nome non descrittivo e posizione errata
// src/utils/helpers.ts
export function procData(data: any): any {
  // ... implementazione
}

// ✅ Dopo: Rinominato e spostato nella feature corretta
// src/features/analytics/utils/data-processor.ts
export function processAnalyticsData(
  rawData: RawAnalyticsData
): ProcessedAnalyticsData {
  // ... implementazione con tipi specifici
}

// ✅ Pattern per spostamento sicuro
// 1. Crea nuovo file con nuovo nome/posizione
// 2. Mantieni vecchio file con deprecation warning
// 3. Aggiorna tutti gli import
// 4. Rimuovi vecchio file dopo migration
Remove duplication
typescript
// ❌ Prima: Codice duplicato
export function validateLoginForm(email: string, password: string) {
  const errors: string[] = []
  
  if (!email.includes('@')) {
    errors.push('Invalid email format')
  }
  
  if (password.length < 8) {
    errors.push('Password must be at least 8 characters')
  }
  
  return errors
}

export function validateRegisterForm(
  email: string, 
  password: string, 
  confirmPassword: string
) {
  const errors: string[] = []
  
  if (!email.includes('@')) {
    errors.push('Invalid email format')
  }
  
  if (password.length < 8) {
    errors.push('Password must be at least 8 characters')
  }
  
  if (password !== confirmPassword) {
    errors.push('Passwords do not match')
  }
  
  return errors
}

// ✅ Dopo: Eliminazione duplicazione
export function validateEmail(email: string): string | null {
  return email.includes('@') ? null : 'Invalid email format'
}

export function validatePassword(password: string): string | null {
  return password.length >= 8 ? null : 'Password must be at least 8 characters'
}

export function validateLoginForm(email: string, password: string) {
  return [
    validateEmail(email),
    validatePassword(password)
  ].filter(Boolean) as string[]
}

export function validateRegisterForm(
  email: string, 
  password: string, 
  confirmPassword: string
) {
  const errors = [
    validateEmail(email),
    validatePassword(password)
  ]
  
  if (password !== confirmPassword) {
    errors.push('Passwords do not match')
  }
  
  return errors.filter(Boolean) as string[]
}
Simplify conditionals
typescript
// ❌ Prima: Condizionali complessi
export function getUserStatus(user: User): string {
  if (user.isActive) {
    if (user.isPremium) {
      if (user.subscriptionEndDate > new Date()) {
        return 'Active Premium User'
      } else {
        return 'Premium Expired'
      }
    } else {
      return 'Active Free User'
    }
  } else {
    if (user.banned) {
      return 'Banned'
    } else {
      return 'Inactive'
    }
  }
}

// ✅ Dopo: Condizionali semplificati
export function getUserStatus(user: User): UserStatus {
  if (!user.isActive) {
    return user.banned ? 'Banned' : 'Inactive'
  }
  
  if (!user.isPremium) {
    return 'Active Free User'
  }
  
  return user.subscriptionEndDate > new Date() 
    ? 'Active Premium User' 
    : 'Premium Expired'
}

// ✅ Alternative: Lookup tables
const STATUS_MAP = {
  active_premium: 'Active Premium User',
  active_free: 'Active Free User',
  expired: 'Premium Expired',
  banned: 'Banned',
  inactive: 'Inactive'
} as const

export function getUserStatus(user: User): string {
  const statusKey = determineStatusKey(user)
  return STATUS_MAP[statusKey]
}

function determineStatusKey(user: User): keyof typeof STATUS_MAP {
  if (!user.isActive) return user.banned ? 'banned' : 'inactive'
  if (!user.isPremium) return 'active_free'
  return user.subscriptionEndDate > new Date() ? 'active_premium' : 'expired'
}
§ TECHNICAL DEBT
Types of tech debt
typescript
// 1. Design Debt - Scelte architetturali
// ❌ Componente accoppiato a API specifica
export async function ProductList() {
  const products = await fetch('/api/v1/products-legacy')
  // ... 
}

// 2. Code Debt - Implementazione scadente
export function calculate(items: any[]): any {
  // Tipo 'any', logica confusa
}

// 3. Test Debt - Mancanza di test
// ❌ Componente senza test
export function CriticalBusinessLogic() {
  // Logica complessa senza test
}

// 4. Documentation Debt - Documentazione mancante
export function complexAlgorithm(input: unknown): unknown {
  // Nessun JSDoc, parametri misteriosi
}

// 5. Dependency Debt - Dipendenze obsolete
// package.json
{
  "dependencies": {
    "old-library": "1.0.0", // 5 anni senza update
    "vulnerable-package": "^2.1.0" // CVE noto
  }
}
Debt tracking
typescript
// ✅ Template per ADR (Architecture Decision Record) sul debito
// docs/adr/0045-legacy-checkout-debt.md
/*
# ADR-0045: Legacy Checkout Technical Debt

## Status
Accepted | Pending | Deprecated

§ CONTEXT
Il checkout legacy utilizza jQuery e callback annidati
- Difficile da mantenere
- Nessuna validazione TypeScript
- Performance issues su mobile

§ DECISION
Accettiamo debito tecnico fino a Q3 2024
Piano di refactoring in 3 fasi...

§ CONSEQUENCES
**Positive:**
- Tempo per progettare nuova architettura

**Negative:**
- Aumento costo manutenzione
- Rischio bug in produzione

§ MITIGATION
- Test aggiuntivi per area critica
- Monitoraggio errori aumentato
- Documentazione aggiornata
*/
Prioritization matrix
typescript
// ✅ Matrice di priorità per debito tecnico
interface TechDebtItem {
  id: string
  description: string
  impact: 'high' | 'medium' | 'low'  // Impatto business
  effort: 'high' | 'medium' | 'low'   // Sforzo risoluzione
  risk: 'high' | 'medium' | 'low'     // Rischio se non risolto
  debtType: 'design' | 'code' | 'test' | 'doc' | 'dependency'
  createdAt: Date
  dueDate?: Date
}

// Calcolo priorità automatico
function calculatePriority(item: TechDebtItem): number {
  const impactScore = { high: 3, medium: 2, low: 1 }[item.impact]
  const riskScore = { high: 3, medium: 2, low: 1 }[item.risk]
  const effortScore = { high: 1, medium: 2, low: 3 }[item.effort] // Inverso
  
  return (impactScore + riskScore) * effortScore
}

// Quadrante Eisenhower per debito tecnico
type EisenhowerQuadrant = 'do' | 'decide' | 'delegate' | 'delete'

function categorizeDebt(item: TechDebtItem): EisenhowerQuadrant {
  if (item.impact === 'high' && item.risk === 'high') return 'do'
  if (item.impact === 'high' && item.risk === 'medium') return 'decide'
  if (item.impact === 'medium' && item.risk === 'low') return 'delegate'
  return 'delete'
}
Paying down debt
typescript
// ✅ Strategia: Boy Scout Rule
// "Lascia il campo più pulito di come l'hai trovato"

// 1. Fix mentre si lavora
export function refactorWhileWorking() {
  // Prima di modificare una funzione:
  // - Aggiungi tipi TypeScript
  // - Estrai logica complessa
  // - Aggiungi test mancanti
}

// 2. Dedicated refactoring sprints
// Riserva 20% capacity ogni sprint per refactoring

// 3. Automation
// scripts/tech-debt-reduction.ts
async function automateDebtReduction() {
  // Script per:
  // - Aggiornare dipendenze obsolete
  // - Convertire JS a TypeScript
  // - Rimuovere codice non usato
  // - Applicare linting fixes
}

// 4. Metric-based approach
interface RefactoringGoal {
  metric: 'testCoverage' | 'typeCoverage' | 'complexityScore'
  target: number
  current: number
  deadline: Date
}

const currentGoals: RefactoringGoal[] = [
  {
    metric: 'typeCoverage',
    target: 95, // Percentuale
    current: 82,
    deadline: new Date('2024-06-30')
  }
]
Preventing new debt
typescript
// ✅ Configurazione per prevenzione debito
// .eslintrc.js
module.exports = {
  rules: {
    'no-explicit-any': 'error',
    'max-depth': ['error', 3], // Previene nesting eccessivo
    'complexity': ['error', 10], // Complessità ciclomatica
    'max-lines-per-function': ['error', 50],
    '@typescript-eslint/explicit-function-return-type': 'error'
  }
}

// package.json - Politiche aggiornamento
{
  "scripts": {
    "dep-check": "npm outdated --long",
    "dep-audit": "npm audit --audit-level=moderate",
    "auto-update": "npm update --save"
  },
  "engines": {
    "node": ">=18.0.0",
    "npm": ">=9.0.0"
  }
}

// ✅ Git hooks per quality gate
// .husky/pre-commit
#!/bin/sh
npm run lint
npm run type-check
npm run test:changed

// ✅ Template PR con checklist debito tecnico
// .github/pull_request_template.md
"""
§ TECHNICAL DEBT ASSESSMENT

§ NEW DEBT INTRODUCED
- [ ] Usa `any` o `unknown` senza validazione
- [ ] Logica complessa senza commenti
- [ ] Duplicazione di codice
- [ ] Test mancanti per logica business

§ EXISTING DEBT REDUCED
- [ ] Refactoring di codice legacy
- [ ] Miglioramento tipi TypeScript
- [ ] Aumento test coverage
- [ ] Rimozione dipendenze non usate
"""
§ MAINTAINABILITY
Code readability
typescript
// ❌ Difficile da leggere
export function p(d:Date,i:number):string{
const m=d.getMonth()+i;const y=d.getFullYear()+Math.floor(m/12);
const nm=m%12;return`${y}-${String(nm+1).padStart(2,'0')}-01`}

// ✅ Leggibile
export function calculateFutureMonth(
  baseDate: Date, 
  monthsToAdd: number
): string {
  const targetMonth = baseDate.getMonth() + monthsToAdd
  const year = baseDate.getFullYear() + Math.floor(targetMonth / 12)
  const normalizedMonth = targetMonth % 12
  const monthString = String(normalizedMonth + 1).padStart(2, '0')
  
  return `${year}-${monthString}-01`
}

// ✅ Principi per leggibilità:
// 1. Nomi descrittivi (variabili, funzioni, tipi)
// 2. Funzioni piccole (< 20 linee)
// 3. Indentazione consistente (2 spazi per TypeScript)
// 4. Linee brevi (< 80 caratteri)
// 5. Spaziatura logica tra blocchi
Self-documenting code
typescript
// ❌ Codice che richiede commenti
function process(data) {
  // Calcola il totale
  let t = 0
  for (let i = 0; i < data.length; i++) {
    t += data[i].v
  }
  
  // Applica sconto se > 100
  if (t > 100) {
    t = t * 0.9
  }
  
  return t
}

// ✅ Codice auto-documentante
function calculateDiscountedTotal(orderItems: OrderItem[]): number {
  const subtotal = calculateSubtotal(orderItems)
  const discountedTotal = applyVolumeDiscount(subtotal)
  
  return discountedTotal
}

function calculateSubtotal(orderItems: OrderItem[]): number {
  return orderItems.reduce(
    (sum, item) => sum + (item.price * item.quantity), 
    0
  )
}

function applyVolumeDiscount(amount: number): number {
  const DISCOUNT_THRESHOLD = 100
  const DISCOUNT_RATE = 0.1
  
  return amount > DISCOUNT_THRESHOLD 
    ? amount * (1 - DISCOUNT_RATE) 
    : amount
}

// ✅ Tipi come documentazione
interface UserPreferences {
  theme: 'light' | 'dark' | 'system'
  notifications: {
    email: boolean
    push: boolean
    frequency: 'instant' | 'daily' | 'weekly'
  }
  privacy: {
    profileVisibility: 'public' | 'private' | 'friends-only'
    dataSharing: boolean
  }
}

// Documenta attraverso i tipi, non solo commenti
Comment guidelines
typescript
// ✅ COMMENTI CORRETTI

// 1. Commenti "perché", non "cosa"
// ❌ Male: Incrementa i
i++

// ✅ Bene: Gestisce offset per array 1-based dell'API
i++ // L'API backend usa indici 1-based

// 2. JSDoc per API pubbliche
/**
 * Calcola il prezzo scontato in base alle regole promozionali
 * 
 * @param basePrice - Prezzo base del prodotto
 * @param userTier - Livello dell'utente (standard, premium, vip)
 * @param campaignId - ID campagna promozionale opzionale
 * @returns Prezzo finale con sconti applicati
 * 
 * @example
 * ```typescript
 * const price = calculateDiscountedPrice(100, 'premium', 'SUMMER2024')
 * // Restituisce 85 (15% sconto premium + 5% sconto campagna)
 * ```
 * 
 * @throws {InvalidCampaignError} Se la campagna non è valida
 */
export function calculateDiscountedPrice(
  basePrice: number,
  userTier: UserTier,
  campaignId?: string
): number {
  // Implementazione...
}

// 3. TODO commenti strutturati
// TODO: [PERF-123] Ottimizzare algoritmo quando supporto WebAssembly
//       Data: 2024-06-30 | Assegnato: @alice
function expensiveCalculation() {
  // Implementazione corrente lenta
}

// 4. FIXME per problemi critici
// FIXME: [SEC-45] Vulnerabilità XSS in input non sanitizzato
//        Workaround temporaneo: escape manuale
function renderUserContent(content: string) {
  // Implementazione attuale con vulnerabilità
}

// 5. NOTE per chiarimenti importanti
// NOTE: Questa funzione assume che i dati siano già validati
//       dalla chiamata API. Non effettua validazioni aggiuntive.
function processValidatedData(data: ValidatedInput) {
  // Implementazione...
}

// ❌ Commenti da evitare
// Incrementa contatore <- Ovvio, inutile
counter++

// Funzione per sommare <- Documentazione ridondante
function sum(a, b) { return a + b }
Naming conventions
typescript
// ✅ CONVENZIONI CONSISTENTI

// 1. Variabili e funzioni: camelCase
const userName = 'John'
const itemCount = 5
function calculateTotalPrice() {}
function isValidEmail() {}

// 2. Classi, tipi, interfacce: PascalCase
class UserAccount {}
interface OrderDetails {}
type ApiResponse<T> = {}

// 3. Costanti: UPPER_SNAKE_CASE
const MAX_RETRY_ATTEMPTS = 3
const DEFAULT_TIMEOUT_MS = 5000
const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL

// 4. Booleani: prefisso is/has/should/can
const isLoading = true
const hasPermission = false
const shouldValidate = true
const canEdit = false

// 5. Array: plurale o suffisso List
const users: User[] = []
const errorMessages: string[] = []
const productList: Product[] = []

// 6. Funzioni: verbo + sostantivo
function getUserById() {}        // ✅
function validateEmail() {}      // ✅
function transformData() {}      // ✅
function handleClick() {}        // ✅

// ❌ Evitare
function user() {}              // ❌ Non descrittivo
function data() {}              // ❌ Troppo generico
function process() {}           // ❌ Vago

// 7. Event handlers: handle + EventName
function handleSubmit() {}
function handleClick() {}
function handleChange() {}

// 8. Hook personalizzati: prefisso use
function useAuth() {}
function useLocalStorage() {}
function useDebounce() {}

// 9. Componenti React: PascalCase
function UserProfile() {}        // ✅
function NavigationMenu() {}     // ✅
function PrimaryButton() {}      // ✅

// 10. File naming
// ✅
UserProfile.tsx
use-user-preferences.ts
product.utils.ts
auth.types.ts

// ❌
userProfile.tsx    // ❌ CamelCase per file
User_Profile.tsx   // ❌ Underscores
UTILS.ts           // ❌ Tutto maiuscolo
§ SCALABILITY PATTERNS
Horizontal scaling considerations
typescript
// ✅ Pattern per scaling orizzontale in Next.js

// 1. Stateless design
// ❌ Dipendenza da memoria locale
let sessionCache = new Map()

// ✅ Session in database esterno o Redis
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.REDIS_URL,
  token: process.env.REDIS_TOKEN,
})

export async function getSession(sessionId: string) {
  return await redis.get(`session:${sessionId}`)
}

// 2. File storage esterno
// ❌ File upload su filesystem dell'istanza
export async function uploadFile(file: File) {
  const path = `/uploads/${file.name}`
  await writeFile(path, file)
  // Problema: file accessibili solo su questa istanza
}

// ✅ Usa storage esterno (S3, Cloud Storage)
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3'

const s3 = new S3Client({ region: 'eu-west-1' })

export async function uploadFileToS3(file: File, userId: string) {
  const key = `users/${userId}/${Date.now()}-${file.name}`
  
  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET,
    Key: key,
    Body: file,
  }))
  
  return `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`
}

// 3. Connection pooling per database
// ❌ Connessione per ogni richiesta
export async function queryDatabase() {
  const client = new Client() // Nuova connessione
  await client.connect()
  // ...
}

// ✅ Connection pool condiviso
import { Pool } from 'pg'

// Singleton condiviso tra tutte le istanze
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Limite connessioni per istanza
  idleTimeoutMillis: 30000,
})

export async function queryDatabase(query: string) {
  const client = await pool.connect()
  try {
    return await client.query(query)
  } finally {
    client.release()
  }
}
Vertical scaling considerations
typescript
// ✅ Ottimizzazioni per scaling verticale

// 1. Memory management
// ❌ Memory leak potenziale
export function createDataProcessor() {
  const cache = new Map() // ❌ Never cleared
  
  return {
    process: (data: string) => {
      if (!cache.has(data)) {
        const result = expensiveComputation(data)
        cache.set(data, result)
      }
      return cache.get(data)
    }
  }
}

// ✅ Cache con limite e TTL
import { LRUCache } from 'lru-cache'

const cache = new LRUCache<string, any>({
  max: 100, // Limite elementi
  ttl: 1000 * 60 * 5, // 5 minuti
  allowStale: false,
})

// 2. CPU-intensive task offloading
// ❌ Task pesante nel main thread
export async function processLargeDataset(dataset: LargeDataset) {
  // Blocca l'event loop per secondi
  const result = performHeavyComputation(dataset)
  return result
}

// ✅ Worker threads per task CPU-intensive
// lib/worker.ts
import { Worker } from 'worker_threads'

export function runInWorker<T>(task: string, data: any): Promise<T> {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./worker-script.js', {
      workerData: { task, data }
    })
    
    worker.on('message', resolve)
    worker.on('error', reject)
    worker.on('exit', (code) => {
      if (code !== 0) {
        reject(new Error(`Worker stopped with exit code ${code}`))
      }
    })
  })
}

// 3. Streaming responses per grandi dataset
// ❌ Tutti i dati in memoria
export async function GET() {
  const data = await fetchLargeDataset() // 1GB in memoria
  return Response.json(data) // Memory spike
}

// ✅ Streaming response
export async function GET() {
  const stream = await createDatasetStream()
  
  return new Response(stream, {
    headers: {
      'Content-Type': 'application/json',
      'Transfer-Encoding': 'chunked',
    },
  })
}
Database scaling
typescript
// ✅ Strategie database scaling per Next.js

// 1. Read replicas per carichi di lettura
// ❌ Query pesanti sul database primario
export async function getAnalyticsReport() {
  // Query complessa che impiega 10 secondi
  return db.primary.query('SELECT ... complex analytics ...')
}

// ✅ Usa replica per query analitiche
const readReplica = new Pool({
  connectionString: process.env.DATABASE_READ_REPLICA_URL,
})

export async function getAnalyticsReport() {
  // Query su replica, non blocca operazioni CRUD
  return readReplica.query('SELECT ... complex analytics ...')
}

// 2. Connection pooling configuration
const primaryPool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: process.env.NODE_ENV === 'production' ? 10 : 5,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
})

// 3. Query optimization patterns
// ❌ N+1 query problem
export async function getUsersWithOrders() {
  const users = await db.user.findMany()
  
  const usersWithOrders = await Promise.all(
    users.map(async (user) => {
      const orders = await db.order.findMany({ where: { userId: user.id } })
      return { ...user, orders }
    })
  )
  
  return usersWithOrders
}

// ✅ Eager loading con joins
export async function getUsersWithOrders() {
  return db.user.findMany({
    include: {
      orders: true, // Single query con join
    },
    take: 100, // Pagination
    orderBy: { createdAt: 'desc' }
  })
}

// 4. Database indexing strategy
// models/user.model.ts
export const UserSchema = {
  name: 'User',
  properties: {
    _id: 'objectId',
    email: 'string',
    // Indice per query frequenti
    email_index: { type: 'index', property: 'email' },
    
    // Indice composto
    status_created_index: { 
      type: 'index', 
      properties: ['status', 'createdAt'] 
    },
  }
}

// 5. Caching layer per query frequenti
const queryCache = new Map<string, { data: any; timestamp: number }>()

export async function getCachedQuery<T>(
  key: string, 
  queryFn: () => Promise<T>,
  ttlMs: number = 30000
): Promise<T> {
  const cached = queryCache.get(key)
  
  if (cached && Date.now() - cached.timestamp < ttlMs) {
    return cached.data as T
  }
  
  const data = await queryFn()
  queryCache.set(key, { data, timestamp: Date.now() })
  
  return data
}
Caching strategies
typescript
// ✅ Strategie caching multilivello

// 1. CDN caching per asset statici
// next.config.js
module.exports = {
  headers: async () => [
    {
      source: '/_next/static/:path*',
      headers: [
        {
          key: 'Cache-Control',
          value: 'public, max-age=31536000, immutable',
        },
      ],
    },
    {
      source: '/static/:path*',
      headers: [
        {
          key: 'Cache-Control',
          value: 'public, max-age=86400',
        },
      ],
    },
  ],
}

// 2. Server-side caching con Redis
// lib/redis-cache.ts
import { Redis } from '@upstash/redis'

export class CacheService {
  private redis: Redis
  
  constructor() {
    this.redis = new Redis({
      url: process.env.REDIS_URL!,
      token: process.env.REDIS_TOKEN!,
    })
  }
  
  async getOrSet<T>(
    key: string,
    fetchFn: () => Promise<T