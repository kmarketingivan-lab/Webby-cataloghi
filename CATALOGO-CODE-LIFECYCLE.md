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

## § ADVANCED PATTERNS: CODE LIFECYCLE

### Server Actions con Validazione

```typescript
// app/actions/code-lifecycle.ts
"use server";

import { z } from "zod";
import { revalidatePath } from "next/cache";
import { redirect } from "next/navigation";

const ItemSchema = z.object({
  name: z.string().min(2).max(100),
  description: z.string().min(10).max(5000),
  category: z.enum(["general", "premium", "enterprise"]),
  price: z.number().positive().max(999999),
  metadata: z.record(z.string(), z.unknown()).optional(),
  tags: z.array(z.string().max(50)).max(10).optional(),
  isActive: z.boolean().default(true),
});

type ItemInput = z.infer<typeof ItemSchema>;

interface ActionResult<T = unknown> {
  success: boolean;
  data?: T;
  error?: string;
  fieldErrors?: Record<string, string[]>;
}

export async function createItem(formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.safeParse(raw);
  if (!validation.success) {
    return {
      success: false,
      error: "Validation failed",
      fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]>,
    };
  }

  try {
    const { db } = await import("@/lib/db");
    const item = await db.insert("items").values({
      ...validation.data,
      createdAt: new Date(),
      updatedAt: new Date(),
    }).returning();

    revalidatePath("/items");
    return { success: true, data: item[0] };
  } catch (error) {
    console.error("Failed to create item:", error);
    return { success: false, error: "Failed to create item. Please try again." };
  }
}

export async function updateItem(id: string, formData: FormData): Promise<ActionResult> {
  const raw = {
    name: formData.get("name") as string,
    description: formData.get("description") as string,
    category: formData.get("category") as string,
    price: parseFloat(formData.get("price") as string),
    tags: (formData.get("tags") as string)?.split(",").map((t) => t.trim()).filter(Boolean),
    isActive: formData.get("isActive") === "true",
  };

  const validation = ItemSchema.partial().safeParse(raw);
  if (!validation.success) {
    return { success: false, error: "Validation failed", fieldErrors: validation.error.flatten().fieldErrors as Record<string, string[]> };
  }

  try {
    const { db } = await import("@/lib/db");
    const updated = await db.update("items")
      .set({ ...validation.data, updatedAt: new Date() })
      .where({ id })
      .returning();

    if (!updated[0]) return { success: false, error: "Item not found" };

    revalidatePath("/items");
    revalidatePath(`/items/${id}`);
    return { success: true, data: updated[0] };
  } catch (error) {
    console.error("Failed to update item:", error);
    return { success: false, error: "Failed to update item" };
  }
}

export async function deleteItem(id: string): Promise<ActionResult> {
  try {
    const { db } = await import("@/lib/db");
    await db.update("items").set({ deletedAt: new Date() }).where({ id });
    revalidatePath("/items");
    return { success: true };
  } catch (error) {
    console.error("Failed to delete item:", error);
    return { success: false, error: "Failed to delete item" };
  }
}

export async function bulkUpdateItems(
  ids: string[],
  updates: Partial<ItemInput>
): Promise<ActionResult<{ updated: number }>> {
  if (ids.length === 0) return { success: false, error: "No items selected" };
  if (ids.length > 100) return { success: false, error: "Maximum 100 items at once" };

  try {
    const { db } = await import("@/lib/db");
    let updatedCount = 0;
    for (const id of ids) {
      const result = await db.update("items").set({ ...updates, updatedAt: new Date() }).where({ id }).returning();
      if (result[0]) updatedCount++;
    }
    revalidatePath("/items");
    return { success: true, data: { updated: updatedCount } };
  } catch (error) {
    console.error("Bulk update failed:", error);
    return { success: false, error: "Bulk update failed" };
  }
}
```

### Hook useOptimisticList

```typescript
// hooks/useOptimisticList.ts
"use client";

import { useOptimistic, useTransition, useCallback, useState } from "react";

interface ListItem {
  id: string;
  [key: string]: unknown;
}

type OptimisticAction<T> =
  | { type: "add"; item: T }
  | { type: "update"; id: string; updates: Partial<T> }
  | { type: "remove"; id: string }
  | { type: "reorder"; fromIndex: number; toIndex: number };

export function useOptimisticList<T extends ListItem>(initialItems: T[]) {
  const [isPending, startTransition] = useTransition();
  const [error, setError] = useState<string | null>(null);

  const [optimisticItems, dispatch] = useOptimistic<T[], OptimisticAction<T>>(
    initialItems,
    (state, action) => {
      switch (action.type) {
        case "add":
          return [...state, action.item];
        case "update":
          return state.map((item) =>
            item.id === action.id ? { ...item, ...action.updates } : item
          );
        case "remove":
          return state.filter((item) => item.id !== action.id);
        case "reorder": {
          const newState = [...state];
          const [moved] = newState.splice(action.fromIndex, 1);
          newState.splice(action.toIndex, 0, moved);
          return newState;
        }
        default:
          return state;
      }
    }
  );

  const addOptimistic = useCallback(
    (item: T, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "add", item });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to add item");
        }
      });
    },
    [dispatch]
  );

  const updateOptimistic = useCallback(
    (id: string, updates: Partial<T>, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "update", id, updates });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to update item");
        }
      });
    },
    [dispatch]
  );

  const removeOptimistic = useCallback(
    (id: string, serverAction: () => Promise<void>) => {
      setError(null);
      startTransition(async () => {
        dispatch({ type: "remove", id });
        try {
          await serverAction();
        } catch (err) {
          setError(err instanceof Error ? err.message : "Failed to remove item");
        }
      });
    },
    [dispatch]
  );

  return {
    items: optimisticItems,
    isPending,
    error,
    addOptimistic,
    updateOptimistic,
    removeOptimistic,
  };
}
```

### Data Table con Sorting, Filtering e Pagination

```typescript
// components/DataTable.tsx
"use client";

import { useState, useMemo, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  ChevronLeft,
  ChevronRight,
  ChevronsLeft,
  ChevronsRight,
  ArrowUpDown,
  ArrowUp,
  ArrowDown,
  Search,
  X,
} from "lucide-react";

interface Column<T> {
  key: keyof T & string;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
  render?: (value: T[keyof T], row: T) => React.ReactNode;
  width?: string;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  searchable?: boolean;
  selectable?: boolean;
  onRowClick?: (row: T) => void;
  onSelectionChange?: (selected: T[]) => void;
  emptyMessage?: string;
}

export function DataTable<T extends { id: string }>({
  data,
  columns,
  pageSize = 10,
  searchable = true,
  selectable = false,
  onRowClick,
  onSelectionChange,
  emptyMessage = "No data found",
}: DataTableProps<T>) {
  const [currentPage, setCurrentPage] = useState(1);
  const [sortKey, setSortKey] = useState<string | null>(null);
  const [sortDirection, setSortDirection] = useState<"asc" | "desc">("asc");
  const [searchQuery, setSearchQuery] = useState("");
  const [selectedIds, setSelectedIds] = useState<Set<string>>(new Set());
  const [filters, setFilters] = useState<Record<string, string>>({});

  const filteredData = useMemo(() => {
    let result = [...data];

    if (searchQuery) {
      const query = searchQuery.toLowerCase();
      result = result.filter((row) =>
        columns.some((col) => {
          const value = row[col.key];
          return value !== null && value !== undefined && String(value).toLowerCase().includes(query);
        })
      );
    }

    for (const [key, filterValue] of Object.entries(filters)) {
      if (!filterValue) continue;
      result = result.filter((row) => String(row[key as keyof T]).toLowerCase().includes(filterValue.toLowerCase()));
    }

    if (sortKey) {
      result.sort((a, b) => {
        const aVal = a[sortKey as keyof T];
        const bVal = b[sortKey as keyof T];
        if (aVal === bVal) return 0;
        if (aVal === null || aVal === undefined) return 1;
        if (bVal === null || bVal === undefined) return -1;
        const comparison = aVal < bVal ? -1 : 1;
        return sortDirection === "asc" ? comparison : -comparison;
      });
    }
    return result;
  }, [data, searchQuery, filters, sortKey, sortDirection, columns]);

  const totalPages = Math.ceil(filteredData.length / pageSize);
  const paginatedData = filteredData.slice((currentPage - 1) * pageSize, currentPage * pageSize);

  const handleSort = useCallback((key: string) => {
    if (sortKey === key) {
      setSortDirection((prev) => (prev === "asc" ? "desc" : "asc"));
    } else {
      setSortKey(key);
      setSortDirection("asc");
    }
    setCurrentPage(1);
  }, [sortKey]);

  const toggleSelection = useCallback((id: string) => {
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (next.has(id)) next.delete(id); else next.add(id);
      if (onSelectionChange) {
        onSelectionChange(data.filter((item) => next.has(item.id)));
      }
      return next;
    });
  }, [data, onSelectionChange]);

  const toggleAll = useCallback(() => {
    const allIds = paginatedData.map((item) => item.id);
    const allSelected = allIds.every((id) => selectedIds.has(id));
    setSelectedIds((prev) => {
      const next = new Set(prev);
      if (allSelected) { allIds.forEach((id) => next.delete(id)); }
      else { allIds.forEach((id) => next.add(id)); }
      if (onSelectionChange) { onSelectionChange(data.filter((item) => next.has(item.id))); }
      return next;
    });
  }, [paginatedData, selectedIds, data, onSelectionChange]);

  const SortIcon = ({ columnKey }: { columnKey: string }) => {
    if (sortKey !== columnKey) return <ArrowUpDown className="h-3 w-3 ml-1 opacity-50" />;
    return sortDirection === "asc" ? <ArrowUp className="h-3 w-3 ml-1" /> : <ArrowDown className="h-3 w-3 ml-1" />;
  };

  return (
    <Card>
      <CardHeader className="pb-3">
        <div className="flex items-center justify-between">
          <CardTitle className="text-lg">
            {filteredData.length} result{filteredData.length !== 1 ? "s" : ""}
          </CardTitle>
          {searchable && (
            <div className="relative w-64">
              <Search className="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-muted-foreground" />
              <Input
                placeholder="Search..."
                value={searchQuery}
                onChange={(e) => { setSearchQuery(e.target.value); setCurrentPage(1); }}
                className="pl-9 pr-9"
              />
              {searchQuery && (
                <button onClick={() => setSearchQuery("")} className="absolute right-3 top-1/2 -translate-y-1/2">
                  <X className="h-4 w-4 text-muted-foreground" />
                </button>
              )}
            </div>
          )}
        </div>
        {selectedIds.size > 0 && (
          <div className="flex items-center gap-2 mt-2">
            <Badge variant="secondary">{selectedIds.size} selected</Badge>
            <Button variant="ghost" size="sm" onClick={() => setSelectedIds(new Set())}>Clear</Button>
          </div>
        )}
      </CardHeader>
      <CardContent>
        <div className="overflow-x-auto">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b bg-muted/50">
                {selectable && (
                  <th className="w-10 py-3 px-3">
                    <input type="checkbox" onChange={toggleAll}
                      checked={paginatedData.length > 0 && paginatedData.every((item) => selectedIds.has(item.id))} />
                  </th>
                )}
                {columns.map((col) => (
                  <th key={col.key} className="text-left py-3 px-3 font-medium" style={{ width: col.width }}>
                    {col.sortable ? (
                      <button onClick={() => handleSort(col.key)} className="flex items-center hover:text-foreground">
                        {col.label} <SortIcon columnKey={col.key} />
                      </button>
                    ) : col.label}
                  </th>
                ))}
              </tr>
            </thead>
            <tbody>
              {paginatedData.length === 0 ? (
                <tr><td colSpan={columns.length + (selectable ? 1 : 0)} className="text-center py-12 text-muted-foreground">{emptyMessage}</td></tr>
              ) : (
                paginatedData.map((row) => (
                  <tr key={row.id} onClick={() => onRowClick?.(row)}
                    className={`border-b transition-colors hover:bg-muted/50 ${onRowClick ? "cursor-pointer" : ""} ${selectedIds.has(row.id) ? "bg-primary/5" : ""}`}>
                    {selectable && (
                      <td className="py-3 px-3" onClick={(e) => e.stopPropagation()}>
                        <input type="checkbox" checked={selectedIds.has(row.id)} onChange={() => toggleSelection(row.id)} />
                      </td>
                    )}
                    {columns.map((col) => (
                      <td key={col.key} className="py-3 px-3">
                        {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                      </td>
                    ))}
                  </tr>
                ))
              )}
            </tbody>
          </table>
        </div>

        {totalPages > 1 && (
          <div className="flex items-center justify-between mt-4">
            <p className="text-sm text-muted-foreground">
              Page {currentPage} of {totalPages}
            </p>
            <div className="flex items-center gap-1">
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(1)} disabled={currentPage === 1}>
                <ChevronsLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p - 1)} disabled={currentPage === 1}>
                <ChevronLeft className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage((p) => p + 1)} disabled={currentPage === totalPages}>
                <ChevronRight className="h-4 w-4" />
              </Button>
              <Button variant="outline" size="sm" onClick={() => setCurrentPage(totalPages)} disabled={currentPage === totalPages}>
                <ChevronsRight className="h-4 w-4" />
              </Button>
            </div>
          </div>
        )}
      </CardContent>
    </Card>
  );
}
```

### API Route con Middleware Pattern

```typescript
// lib/api/middleware.ts
import { NextRequest, NextResponse } from "next/server";
import { z } from "zod";

type Handler = (
  req: NextRequest,
  context: { params: Record<string, string>; user?: { id: string; role: string } }
) => Promise<NextResponse>;

type Middleware = (handler: Handler) => Handler;

export function withValidation<T>(schema: z.ZodSchema<T>, source: "body" | "query" = "body"): Middleware {
  return (handler) => async (req, context) => {
    try {
      let data: unknown;
      if (source === "body") {
        data = await req.json();
      } else {
        const searchParams = Object.fromEntries(req.nextUrl.searchParams);
        data = searchParams;
      }
      const parsed = schema.parse(data);
      (req as any).validated = parsed;
      return handler(req, context);
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          { error: "Validation failed", details: error.errors.map((e) => ({ path: e.path.join("."), message: e.message })) },
          { status: 400 }
        );
      }
      return NextResponse.json({ error: "Invalid request body" }, { status: 400 });
    }
  };
}

export function withAuth(requiredRole?: string): Middleware {
  return (handler) => async (req, context) => {
    const authHeader = req.headers.get("authorization");
    if (!authHeader?.startsWith("Bearer ")) {
      return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
    }

    const token = authHeader.slice(7);
    try {
      const { verifyToken } = await import("@/lib/auth");
      const user = await verifyToken(token);
      if (!user) return NextResponse.json({ error: "Invalid token" }, { status: 401 });
      if (requiredRole && user.role !== requiredRole && user.role !== "admin") {
        return NextResponse.json({ error: "Forbidden" }, { status: 403 });
      }
      context.user = user;
      return handler(req, context);
    } catch {
      return NextResponse.json({ error: "Authentication failed" }, { status: 401 });
    }
  };
}

export function withRateLimit(maxRequests: number = 60, windowMs: number = 60000): Middleware {
  const requests = new Map<string, { count: number; resetAt: number }>();

  return (handler) => async (req, context) => {
    const ip = req.headers.get("x-forwarded-for") || req.headers.get("x-real-ip") || "unknown";
    const now = Date.now();
    const entry = requests.get(ip);

    if (!entry || now > entry.resetAt) {
      requests.set(ip, { count: 1, resetAt: now + windowMs });
    } else if (entry.count >= maxRequests) {
      return NextResponse.json(
        { error: "Too many requests" },
        {
          status: 429,
          headers: {
            "X-RateLimit-Limit": maxRequests.toString(),
            "X-RateLimit-Remaining": "0",
            "X-RateLimit-Reset": new Date(entry.resetAt).toISOString(),
            "Retry-After": Math.ceil((entry.resetAt - now) / 1000).toString(),
          },
        }
      );
    } else {
      entry.count++;
    }

    return handler(req, context);
  };
}

export function withErrorHandler(): Middleware {
  return (handler) => async (req, context) => {
    try {
      return await handler(req, context);
    } catch (error) {
      console.error(`[API Error] ${req.method} ${req.url}:`, error);

      if (error instanceof z.ZodError) {
        return NextResponse.json({ error: "Validation error", details: error.errors }, { status: 400 });
      }

      const message = error instanceof Error ? error.message : "Internal server error";
      const status = (error as any).status || 500;
      return NextResponse.json({ error: message }, { status });
    }
  };
}

export function compose(...middlewares: Middleware[]): Middleware {
  return (handler) => {
    let composed = handler;
    for (let i = middlewares.length - 1; i >= 0; i--) {
      composed = middlewares[i](composed);
    }
    return composed;
  };
}

// Esempio d'uso:
// const handler = compose(withErrorHandler(), withAuth("admin"), withRateLimit(30))(async (req, ctx) => {
//   const items = await db.findMany("items", { userId: ctx.user!.id });
//   return NextResponse.json({ items });
// });
```


### CODE LIFECYCLE - Utility Helper #845

```typescript
// lib/utils/code-lifecycle-helper-845.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #874

```typescript
// lib/utils/code-lifecycle-helper-874.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #750

```typescript
// lib/utils/code-lifecycle-helper-750.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #912

```typescript
// lib/utils/code-lifecycle-helper-912.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #202

```typescript
// lib/utils/code-lifecycle-helper-202.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #41

```typescript
// lib/utils/code-lifecycle-helper-41.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #993

```typescript
// lib/utils/code-lifecycle-helper-993.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #723

```typescript
// lib/utils/code-lifecycle-helper-723.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #767

```typescript
// lib/utils/code-lifecycle-helper-767.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #991

```typescript
// lib/utils/code-lifecycle-helper-991.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #558

```typescript
// lib/utils/code-lifecycle-helper-558.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #489

```typescript
// lib/utils/code-lifecycle-helper-489.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #577

```typescript
// lib/utils/code-lifecycle-helper-577.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #500

```typescript
// lib/utils/code-lifecycle-helper-500.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #100

```typescript
// lib/utils/code-lifecycle-helper-100.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #788

```typescript
// lib/utils/code-lifecycle-helper-788.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #123

```typescript
// lib/utils/code-lifecycle-helper-123.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Date(),
    };
  }

  getConfig(): Readonly<CODELIFECYCLEConfig> {
    return Object.freeze({ ...this.config });
  }
}
```


### CODE LIFECYCLE - Utility Helper #583

```typescript
// lib/utils/code-lifecycle-helper-583.ts
import { z } from "zod";

interface CODELIFECYCLEConfig {
  enabled: boolean;
  maxRetries: number;
  timeout: number;
  debug: boolean;
  features: Map<string, boolean>;
  metadata: Record<string, unknown>;
}

interface ProcessResult<T> {
  success: boolean;
  data?: T;
  error?: string;
  duration: number;
  retries: number;
  timestamp: Date;
}

const ConfigSchema = z.object({
  enabled: z.boolean().default(true),
  maxRetries: z.number().min(0).max(10).default(3),
  timeout: z.number().min(1000).max(60000).default(15000),
  debug: z.boolean().default(false),
});

export class CODELIFECYCLEProcessor<TInput, TOutput> {
  private config: CODELIFECYCLEConfig;
  private processors: Array<(input: TInput) => Promise<TOutput>> = [];
  private middlewares: Array<(input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>> = [];

  constructor(config: Partial<CODELIFECYCLEConfig> = {}) {
    const validated = ConfigSchema.parse(config);
    this.config = {
      ...validated,
      features: new Map(),
      metadata: {},
    };
  }

  use(middleware: (input: TInput, next: () => Promise<TOutput>) => Promise<TOutput>): this {
    this.middlewares.push(middleware);
    return this;
  }

  addProcessor(processor: (input: TInput) => Promise<TOutput>): this {
    this.processors.push(processor);
    return this;
  }

  async process(input: TInput): Promise<ProcessResult<TOutput>> {
    const startTime = Date.now();
    let retries = 0;

    while (retries <= this.config.maxRetries) {
      try {
        const controller = new AbortController();
        const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);

        let result: TOutput | undefined;
        for (const processor of this.processors) {
          result = await processor(input);
        }

        clearTimeout(timeoutId);

        return {
          success: true,
          data: result,
          duration: Date.now() - startTime,
          retries,
          timestamp: new Date(),
        };
      } catch (error) {
        retries++;
        if (retries > this.config.maxRetries) {
          return {
            success: false,
            error: error instanceof Error ? error.message : "Processing failed",
            duration: Date.now() - startTime,
            retries: retries - 1,
            timestamp: new Date(),
          };
        }
        const delay = Math.min(1000 * Math.pow(2, retries), 10000);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }

    return {
      success: false,
      error: "Max retries exceeded",
      duration: Date.now() - startTime,
      retries,
      timestamp: new Da