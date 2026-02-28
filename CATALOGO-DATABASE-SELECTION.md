# CATALOGO-DATABASE-SELECTION

CATALOGO-DATABASE-SELECTION per Next.js 14 - Espansione
§ DECISION TREE COMPLETO
Flowchart Decisionale per Selezione Database
Diagramma
Codice
Schermo intero

Tipo di Applicazione

Struttura Dati

Strutturati/Relazionali

Documenti Semi-strutturati

Chiave-Valore

Grafi/Relazioni

Serie Temporali

Volume Dati

Piccolo < 1GB

Medio 1GB-100GB

Grande > 100GB

Requisiti Scaling

Verticale/VPS

Orizzontale/Serverless

Budget

Basso/Open Source

Medio/Managed

Alto/Enterprise

SQLite/PostgreSQL Self-hosted

Managed PostgreSQL/MySQL

Cloud Native/Distributed

Scelta Finale

Tabella Comparativa Tipologie Database
Categoria	Modello Dati	Punti di Forza	Casi d'Uso Tipici	Esempi
Relazionale	Tabelle, Schema Rigido	ACID, Join Complessi, Integrità Referenziale	ERP, CRM, E-commerce	PostgreSQL, MySQL, SQL Server
Document	Documenti JSON/BSON	Schema Flessibile, Scaling Orizzontale	Contenuti, Catalogo Prodotti, IoT	MongoDB, CouchDB, Firebase
Key-Value	Chiave → Valore	Bassa Latenza, Alta Velocità Lettura	Cache, Sessioni, Configurazioni	Redis, DynamoDB, etcd
Graph	Nodi + Relazioni	Query Relazioni Complesse	Social Network, Recommendation, Fraud Detection	Neo4j, Amazon Neptune, Dgraph
Time-Series	Timestamp + Valori	Query Temporali Efficienti	Metriche, Logs, Fintech	InfluxDB, TimescaleDB, Prometheus
Vector	Embeddings, Vettori	Ricerca Semantica, Similarità	AI, RAG, Ricerca Immagini	Pinecone, Weaviate, pgvector
Cloud-native vs Self-hosted
Aspetto	Cloud-Native	Self-Hosted
Deployment	Serverless/Container	VM/Bare Metal
Scaling	Automatico/Orizzontale	Manuale/Verticale
Costo	Pay-per-use/OpEx	Capex + Manutenzione
Manutenzione	Gestito dal Provider	Team Interno
Controllo	Limitato	Completo
Compliance	Dipende da Provider	Controllo Totale
Performance	Variabile, Network Latency	Prevedibile, Low Latency
Esempi	AWS RDS, MongoDB Atlas, Supabase	PostgreSQL on VM, Redis on K8s
§ POSTGRESQL DEEP DIVE
When to Choose PostgreSQL

Scenari ideali per PostgreSQL:

Applicazioni che richiedono transazioni ACID complesse

Sistema con relazioni dati complesse e molte JOIN

Necessità di vincoli di integrità referenziale avanzati

Progetti che potrebbero aver bisogno di estensioni specializzate

Applicazioni geospaziali (con PostGIS)

Progetti che combinano dati strutturati e JSON

Vantaggi specifici per Next.js:

Integrazione eccellente con Prisma ORM

Supporto nativo per JSON/JSONB per dati semi-strutturati

Connection pooling integrato per serverless functions

Estensioni per casi d'uso moderni (vector, full-text search)

Estensioni Utili per Next.js
Estensione	Scopo	Casi d'Uso Next.js
pgvector	Vettori e embedding similarity	Ricerca semantica, RAG, AI features
PostGIS	Dati geospaziali	App location-based, mappe, delivery
pg_trgm	Fuzzy text search	Ricerca utente, autocorrect, matching
pg_cron	Job scheduling	Task ricorrenti, cleanup, report
TimescaleDB	Time-series data	Analytics, metriche applicative
pg_stat_statements	Query performance	Monitoring e ottimizzazione
uuid-ossp	UUID generation	ID univoci distribuiti
Connection Pooling

PgBouncer Configuration per Next.js:

ini
# Configurazione ottimizzata per serverless
[databases]
nextjsdb = host=localhost dbname=nextjsapp

[pgbouncer]
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
reserve_pool_size = 5

Supabase Connection Pooling:

Pool gestito automaticamente

Supporto per Edge Functions

Configurazione ottimizzata per Vercel

Metriche integrate nel dashboard

Performance Tuning per Next.js

Ottimizzazioni specifiche:

sql
-- Indici per query comuni
CREATE INDEX idx_users_email ON users USING hash(lower(email));
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at DESC);

-- Ottimizzazione per JSONB
CREATE INDEX idx_properties_tags ON products USING gin((properties->'tags'));

-- Configuration per carichi web
ALTER SYSTEM SET shared_buffers = '4GB';
ALTER SYSTEM SET effective_cache_size = '12GB';
ALTER SYSTEM SET work_mem = '16MB';
ALTER SYSTEM SET maintenance_work_mem = '1GB';
Managed Options Comparison
Provider	Prezzo Base	Scalabilità	Feature Uniche	Integrazione Vercel
Supabase	Gratis fino a 500MB	Auto-scaling	Realtime, Auth, Storage	Ottima, one-click deploy
Neon	$0.10/GB-mese	Serverless, branching	Branching istantaneo, CDC	Integrazione diretta
Railway	$0.20/GB + $7/CPU	Vertical scaling	Multi-region, easy setup	Deploy da GitHub
AWS RDS	~$15/mese (t3.micro)	Manual/Auto	AWS ecosystem, backup	Vercel AWS Integration
DigitalOcean	$15/mese (Basic)	Vertical	Simple pricing, backups	Standard connection
CockroachDB	$0.25/GB-mese	Orizzontale	Distributed SQL, resiliency	Serverless compatible
§ MYSQL/MARIADB
When to Choose MySQL

Scenari preferenziali:

Applicazioni web tradizionali (WordPress, Laravel, Rails)

Team con esperienza MySQL esistente

Progetti che richiedono replica semplice master-slave

Ambienti con risorse limitate (MySQL è più leggero)

Quando si usano tool specifici (phpMyAdmin, MySQL Workbench)

Vantaggi per Next.js:

Supporto eccellente in Prisma

Performance ottime per read-heavy workloads

Ampia documentazione e community

Compatibilità con molti servizi PaaS

Differences vs PostgreSQL
Caratteristica	PostgreSQL	MySQL
JSON Support	JSON e JSONB nativo	JSON type (5.7+), meno feature
Full Text Search	TSVector/TSQuery avanzato	FULLTEXT index più semplice
Concorrenza	MVCC (Multi Version Concurrency)	Row-level locking
Estensioni	Ecosistema ricco (PostGIS, etc)	Più limitato
Replica	Logical replication + streaming	Binary log replication
Licenza	PostgreSQL License (liberale)	GPL (Oracle) / GPL (MariaDB)
Performance	Ottima per query complesse	Ottima per letture semplici
PlanetScale (Serverless MySQL)

Architettura:

Database serverless basato su Vitess

Branching come in Git

Schema changes senza downtime

Compatibile con driver MySQL standard

Configurazione per Next.js:

javascript
// next.config.js per PlanetScale
const config = {
  env: {
    DATABASE_URL: process.env.DATABASE_URL,
  },
}

// Prisma schema
datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
  relationMode = "prisma"
}

Vantaggi:

Scaling automatico

Branch per ogni environment (dev/staging/prod)

Deploy requests per schema changes

Integrazione con Vercel Preview Deployments

Vitess for Scaling

Quando usare Vitess:

Sharding automatico di database MySQL

Query routing intelligente

Connection pooling integrato

Compatibilità con Kubernetes

Architettura tipica:

text
Load Balancer → VTGate → VTablet → MySQL Instance
                    ↑
                Topology Service
§ MONGODB
When to Choose MongoDB

Scenari ideali:

Dati semi-strutturati o schema evolutivo

Rapida iterazione su schema dati

Documenti gerarchici complessi

Scaling orizzontale necessario

Aggregazioni complesse su documenti

Vantaggi per Next.js:

Schema flessibile per prototyping veloce

Integrazione con serverless functions

Changestream per real-time updates

Atlas Search integrato

Schema Design Patterns

Pattern comuni per Next.js:

Extended Reference Pattern:

javascript
// User con embedded orders summary
{
  _id: "user123",
  email: "user@example.com",
  recentOrders: [
    { orderId: "ord1", total: 100, date: "2024-01-01" },
    { orderId: "ord2", total: 200, date: "2024-01-02" }
  ]
}

Bucket Pattern per Time-Series:

javascript
// Metriche aggregate per giorno
{
  _id: { userId: "user123", date: "2024-01-01" },
  pageViews: 150,
  clicks: 45,
  sessions: 12
}

Polymorphic Pattern:

javascript
// Contenuti diversi nello stesso collection
{
  _id: "content1",
  type: "article",
  title: "Hello World",
  content: "...",
  tags: ["tech", "web"]
}
{
  _id: "content2", 
  type: "video",
  title: "Tutorial",
  url: "https://...",
  duration: 300
}
Aggregation Pipeline

Esempi per Next.js Analytics:

javascript
// Analytics per dashboard
db.orders.aggregate([
  {
    $match: {
      createdAt: { $gte: new Date("2024-01-01") }
    }
  },
  {
    $group: {
      _id: {
        year: { $year: "$createdAt" },
        month: { $month: "$createdAt" },
        day: { $dayOfMonth: "$createdAt" }
      },
      totalSales: { $sum: "$amount" },
      averageOrder: { $avg: "$amount" },
      orderCount: { $count: {} }
    }
  },
  {
    $sort: { "_id.year": -1, "_id.month": -1 }
  }
])
Atlas Serverless

Configurazione per Next.js:

javascript
// lib/mongodb.js
import { MongoClient } from 'mongodb'

const uri = process.env.MONGODB_URI
const options = {
  maxPoolSize: 10,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
}

let client
let clientPromise

if (process.env.NODE_ENV === 'development') {
  // In development, use global variable
  if (!global._mongoClientPromise) {
    client = new MongoClient(uri, options)
    global._mongoClientPromise = client.connect()
  }
  clientPromise = global._mongoClientPromise
} else {
  // In production, create new client
  client = new MongoClient(uri, options)
  clientPromise = client.connect()
}

export default clientPromise

Vantaggi Atlas Serverless:

Billing per operazione (lettura/scrittura)

Scaling automatico a zero

Integrazione con Vercel Edge Functions

Data API per accesso da edge

Prisma + MongoDB

Configurazione Prisma Schema:

prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
}

model Post {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  title     String
  content   String?
  author    User     @relation(fields: [authorId], references: [id])
  authorId  String   @db.ObjectId
  tags      String[]
  createdAt DateTime @default(now())
}

Considerazioni:

@db.ObjectId per referenze MongoDB

No support per transazioni multi-document in Prisma (usare native driver)

Schema validation opzionale

Migrazioni manuali (schema changes)

§ REDIS
Caching Patterns per Next.js

1. Page/Route Caching:

javascript
// lib/redis.js
import { Redis } from '@upstash/redis'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
})

// API Route con cache
export async function GET(request) {
  const cacheKey = `page:${request.url}`
  const cached = await redis.get(cacheKey)
  
  if (cached) {
    return new Response(cached, {
      headers: { 'x-cache': 'HIT' }
    })
  }
  
  // Generate page
  const data = await fetchData()
  const html = renderPage(data)
  
  // Cache for 5 minutes
  await redis.setex(cacheKey, 300, html)
  
  return new Response(html, {
    headers: { 'x-cache': 'MISS' }
  })
}

2. Data Caching (React Query style):

javascript
// app/products/page.js
async function getProducts() {
  const cacheKey = 'products:all'
  const cached = await redis.get(cacheKey)
  
  if (cached) return JSON.parse(cached)
  
  const products = await db.product.findMany()
  await redis.setex(cacheKey, 60, JSON.stringify(products))
  
  return products
}
Session Storage

Redis per Auth Sessions (NextAuth):

javascript
// pages/api/auth/[...nextauth].js
import RedisStore from "connect-redis"
import { createClient } from "redis"

let redisClient = createClient({
  url: process.env.REDIS_URL
})
redisClient.connect()

export default NextAuth({
  session: {
    strategy: "jwt",
    // O per sessioni database:
    // strategy: "database",
    // maxAge: 30 * 24 * 60 * 60, // 30 giorni
  },
  adapter: RedisAdapter(redisClient),
})
Rate Limiting

API Rate Limiting:

javascript
// middleware.ts
import { Redis } from '@upstash/redis'
import { Ratelimit } from '@upstash/ratelimit'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '10 s'),
})

export async function middleware(request: NextRequest) {
  const ip = request.ip ?? '127.0.0.1'
  const { success, limit, reset, remaining } = await ratelimit.limit(ip)
  
  if (!success) {
    return new Response('Too Many Requests', {
      status: 429,
      headers: {
        'X-RateLimit-Limit': limit.toString(),
        'X-RateLimit-Remaining': remaining.toString(),
        'X-RateLimit-Reset': reset.toString(),
      },
    })
  }
  
  return NextResponse.next()
}
Pub/Sub per Real-time

Real-time Notifications:

javascript
// Componente client per notifiche
'use client'

import { useEffect } from 'react'
import Redis from 'ioredis'

export function NotificationSubscriber({ userId }) {
  useEffect(() => {
    const redis = new Redis(process.env.REDIS_URL)
    const subscriber = redis.duplicate()
    
    subscriber.subscribe(`notifications:${userId}`, (err, count) => {
      if (err) console.error(err)
    })
    
    subscriber.on('message', (channel, message) => {
      const notification = JSON.parse(message)
      // Aggiorna UI
    })
    
    return () => {
      subscriber.unsubscribe()
      subscriber.quit()
    }
  }, [userId])
  
  return null
}
Redis Stack Features

RedisJSON per Document Storage:

javascript
// Store JSON documents
await redis.call('JSON.SET', 'user:123', '$', JSON.stringify({
  name: 'Alice',
  email: 'alice@example.com',
  preferences: { theme: 'dark', notifications: true }
}))

// Query specific fields
const email = await redis.call('JSON.GET', 'user:123', '$.email')

RediSearch per Full-text Search:

javascript
// Create index
await redis.call('FT.CREATE', 'idx:products', 'ON', 'JSON', 'PREFIX', '1', 'product:', 'SCHEMA', '$.name', 'TEXT', '$.description', 'TEXT')

// Search products
const results = await redis.call('FT.SEARCH', 'idx:products', 'laptop*', 'LIMIT', '0', '10')
Upstash (Serverless Redis)

Vantaggi per Next.js:

Serverless, paghi per request

Integrazione diretta con Vercel

REST API per edge functions

Durable Redis (persistenza automatica)

Configurazione:

javascript
// Per Vercel Edge Functions
import { Redis } from '@upstash/redis/edge'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
})

// Usa in edge runtime
export const config = {
  runtime: 'edge',
}

export default async function handler(req) {
  const count = await redis.incr('counter')
  return new Response(JSON.stringify({ count }))
}
§ SQLITE
When to Use SQLite

Scenari ideali:

Applicazioni local-first/offline

Prototipi e MVP rapidi

Siti statici con dati moderati

Applicazioni edge/embedded

Development e testing locale

Progressive Web Apps (PWA)

Vantaggi per Next.js:

Zero-configuration per sviluppo

File-based, facile backup/migrazione

Performance eccellente per carichi moderati

Supporto pieno per SQL (JOIN, transaction, etc.)

Turso (Distributed SQLite)

Architetura:

SQLite distribuito su edge

Replica automatica tra regioni

HTTP API per accesso da edge functions

Compatibile con librerie SQLite esistenti

Configurazione con Next.js:

javascript
// lib/turso.ts
import { createClient } from '@libsql/client/web'

const client = createClient({
  url: process.env.TURSO_DATABASE_URL!,
  authToken: process.env.TURSO_AUTH_TOKEN!,
})

// In Server Component
export async function getUsers() {
  const result = await client.execute('SELECT * FROM users LIMIT 100')
  return result.rows
}

// In Edge Function
export const runtime = 'edge'

export async function GET() {
  const client = createClient({
    url: process.env.TURSO_DATABASE_URL!,
    authToken: process.env.TURSO_AUTH_TOKEN!,
  })
  
  const result = await client.execute('SELECT * FROM posts')
  return Response.json(result.rows)
}

Vantaggi Turso:

Letture globalmente distribuite

Scritture a bassa latenza

Branching per ambienti

Integrazione con Vercel Edge

Litestream per Replication

Backup e replica automatici:

yaml
# litestream.yml
dbs:
  - path: /data/app.db
    replicas:
      - url: s3://mybucket/app.db
        retention: 24h
      - url: file:///backups/app.db
        retention: 7d

Integrazione con Next.js:

Database locale in sviluppo

Replica automatica in produzione

Point-in-time recovery

Zero downtime backup

Local-first Apps con SQLite

Pattern per app offline-first:

typescript
// hooks/useLocalDatabase.ts
import { useEffect, useState } from 'react'
import Database from 'better-sqlite3'

export function useLocalDatabase() {
  const [db, setDb] = useState(null)
  
  useEffect(() => {
    // In browser, use SQLite WASM
    if (typeof window !== 'undefined') {
      import('@sqlite.org/sqlite-wasm').then((sqlite3) => {
        const db = new sqlite3.oo1.DB(':memory:')
        setDb(db)
      })
    }
  }, [])
  
  return db
}

// Sync con server
async function syncLocalToServer(localDb, remoteUrl) {
  const changes = localDb.prepare(
    'SELECT * FROM changes WHERE synced = 0'
  ).all()
  
  await fetch(remoteUrl, {
    method: 'POST',
    body: JSON.stringify(changes),
  })
  
  localDb.exec('UPDATE changes SET synced = 1 WHERE synced = 0')
}
§ SUPABASE vs FIREBASE vs CONVEX
Feature Comparison Table
Feature	Supabase	Firebase	Convex
Database	PostgreSQL	Firestore (NoSQL)	Proprietary (Document)
Realtime	WebSockets + PostgreSQL CDC	Firestore Listeners	WebSockets automatic
Auth	Row Level Security + JWT	Firebase Auth	Integrato con database
Storage	S3-compatible	Firebase Storage	File storage integrato
Functions	Edge Functions	Cloud Functions	Actions & HTTP Endpoints
Pricing	Postgres + storage	Pay per read/write	Compute units + storage
Local Dev	Docker completo	Emulator suite	CLI con hot reload
Vercel Integration	Ottima (one-click)	Buona	Ottima (template)
Type Safety	Generated types	Basic	Full TypeScript
GraphQL	Hasura-like	Apollo possible	No (REST/WebSocket)
Pricing Comparison (mensile approssimativo)
Provider	Tier Gratuito	Entry Paid	Scalabilità	Costo Scalato (10K utenti)
Supabase	500MB DB, 1GB storage	$25/mese	Auto-scaling	~$50-100/mese
Firebase	1GB DB, 10GB storage	Pay-as-you-go	Automatica	~$30-200 (variabile)
Convex	1M function calls	$25/mese	Per compute units	~$75-150/mese
Appwrite	Self-host free	Self-host	Manuale	~$10 VPS
Real-time Capabilities

Supabase Realtime:

javascript
// Client-side subscription
const subscription = supabase
  .channel('posts')
  .on('postgres_changes', 
    { event: 'INSERT', schema: 'public', table: 'posts' },
    (payload) => {
      console.log('New post:', payload.new)
    }
  )
  .subscribe()

Firebase Firestore:

javascript
// Real-time listener
const unsubscribe = db.collection('posts')
  .where('published', '==', true)
  .onSnapshot((snapshot) => {
    snapshot.docChanges().forEach((change) => {
      if (change.type === 'added') {
        console.log('New post:', change.doc.data())
      }
    })
  })

Convex Real-time:

javascript
// Query reattiva
const posts = useQuery(api.posts.get, { limit: 10 })
// Si aggiorna automaticamente quando i dati cambiano
Auth Integration

Supabase Auth con RLS:

sql
-- Row Level Security policy
CREATE POLICY "Users can view own profile"
ON profiles FOR SELECT
USING (auth.uid() = id);

-- Next.js middleware
import { createMiddlewareClient } from '@supabase/auth-helpers-nextjs'

export async function middleware(req) {
  const res = NextResponse.next()
  const supabase = createMiddlewareClient({ req, res })
  await supabase.auth.getSession()
  return res
}

Firebase Auth:

javascript
// Next.js con Firebase Auth
import { getApp, getApps, initializeApp } from 'firebase/app'
import { getAuth } from 'firebase/auth'

const firebaseConfig = { /* config */ }
const app = !getApps().length ? initializeApp(firebaseConfig) : getApp()
const auth = getAuth(app)

// Protezione route
export async function getServerSideProps(context) {
  const session = await getSession(context)
  if (!session) {
    return { redirect: { destination: '/login' } }
  }
  return { props: {} }
}

Convex Auth:

javascript
// Integrazione con provider esterni
const mutation = useMutation(api.posts.create)
const createPost = async (title, content) => {
  await mutation({ title, content })
}

// Auth context automatico
When to Choose Each

Scegli Supabase se:

Hai bisogno di un database PostgreSQL vero

Vuoi SQL completo + migrazioni

Preferisci Row Level Security nativa

Hai bisogno di estensioni PostgreSQL

Vuoi self-hostare in futuro

Scegli Firebase se:

Hai già esperienza con Firebase

Hai bisogno di analytics/messaging integrati

L'app è mobile-first

Vuoi rapid prototyping senza schema

Non ti serve SQL complesso

Scegli Convex se:

Vuoi TypeScript end-to-end

Apprezzi l'astrazione completa

Vuoi real-time automatico

Preferisci meno configurazione

Lavori principalmente su frontend

Scegli Appwrite se:

Vuoi open-source self-hostable

Hai bisogno di controllo completo

Budget limitato ma vuoi feature complete

Multi-cloud deployment

§ VECTOR DATABASES
Panoramica Tecnologie
Database	Modello	Punti di Forza	Caso d'Uso Ideale
Pinecone	Managed Vector DB	Performance ottimizzata, semplice	Production RAG, search large-scale
Weaviate	Vector + Graph	Hybrid search, modules	Knowledge graphs, multimodal
Qdrant	Open-source Vector	Self-hostable, Rust performance	On-premise, data privacy
pgvector	PostgreSQL extension	SQL + vectors, ACID	Existing Postgres apps, consistency
Chroma	Embedding store	Developer-friendly, local-first	Prototyping, small projects
Milvus	Distributed Vector	Scalability, enterprise features	Large-scale similarity search
Pinecone

Integrazione con Next.js:

javascript
import { Pinecone } from '@pinecone-database/pinecone'

const pinecone = new Pinecone({
  apiKey: process.env.PINECONE_API_KEY!,
})

// Index per document embeddings
const index = pinecone.Index('documents')

// Inserimento vettori
await index.upsert([{
  id: 'doc1',
  values: embeddingArray,
  metadata: { title: 'Document 1', category: 'tech' }
}])

// Ricerca similarity
const results = await index.query({
  vector: queryEmbedding,
  topK: 10,
  includeMetadata: true
})

Use Cases:

Semantic search su documenti

RAG (Retrieval Augmented Generation)

Recommendation systems

Deduplicazione contenuti

Weaviate

Configurazione:

javascript
import weaviate from 'weaviate-ts-client'

const client = weaviate.client({
  scheme: 'https',
  host: process.env.WEAVIATE_HOST,
  apiKey: new weaviate.ApiKey(process.env.WEAVIATE_API_KEY),
})

// Schema con module text2vec-openai
const schemaConfig = {
  class: 'Document',
  vectorizer: 'text2vec-openai',
  moduleConfig: {
    'text2vec-openai': {
      model: 'text-embedding-ada-002',
      type: 'text'
    }
  },
  properties: [
    { name: 'title', dataType: ['text'] },
    { name: 'content', dataType: ['text'] },
  ]
}

// Ricerca ibrida (vector + keyword)
const result = await client.graphql
  .get()
  .withClassName('Document')
  .withHybrid({
    query: 'machine learning',
    alpha: 0.5, // balance tra vector e keyword
  })
  .withLimit(10)
  .do()
Qdrant

Self-hosted con Docker:

yaml
# docker-compose.yml
version: '3.8'
services:
  qdrant:
    image: qdrant/qdrant
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_storage:/qdrant/storage
    environment:
      - QDRANT__SERVICE__GRPC_PORT=6334

Client in Next.js:

javascript
import { QdrantClient } from '@qdrant/js-client-rest'

const client = new QdrantClient({
  url: process.env.QDRANT_URL,
  apiKey: process.env.QDRANT_API_KEY,
})

// Creazione collection
await client.createCollection('documents', {
  vectors: {
    size: 1536, // dimensioni embedding
    distance: 'Cosine', // o Euclid, Dot
  }
})

// Ricerca con filtro
const results = await client.search('documents', {
  vector: queryVector,
  limit: 10,
  filter: {
    must: [
      { key: 'category', match: { value: 'technical' } }
    ]
  }
})
pgvector per PostgreSQL

Setup:

sql
-- Abilita l'estensione
CREATE EXTENSION IF NOT EXISTS vector;

-- Crea tabella con vettori
CREATE TABLE document_embeddings (
  id BIGSERIAL PRIMARY KEY,
  document_id BIGINT REFERENCES documents(id),
  embedding vector(1536), -- dimensioni embedding
  content TEXT,
  metadata JSONB
);

-- Crea indice per ricerca efficiente
CREATE INDEX ON document_embeddings 
USING ivfflat (embedding vector_cosine_ops)
WITH (lists = 100);

-- Ricerca similarity
SELECT 
  document_id,
  content,
  1 - (embedding <=> '[0.1, 0.2, ...]') as similarity
FROM document_embeddings
ORDER BY embedding <=> '[0.1, 0.2, ...]'
LIMIT 10;

Integrazione con Prisma:

prisma
model DocumentEmbedding {
  id          Int      @id @default(autoincrement())
  documentId  Int
  document    Document @relation(fields: [documentId], references: [id])
  embedding   Unsupported("vector(1536)")?
  content     String
  
  @@index([embedding], type: Brin)
}
Use Cases per Next.js

1. Semantic Search:

javascript
// API Route per semantic search
export async function POST(req) {
  const { query } = await req.json()
  
  // Genera embedding della query
  const embedding = await generateEmbedding(query)
  
  // Cerca nel vector database
  const results = await vectorDB.query({
    vector: embedding,
    topK: 10,
    includeMetadata: true
  })
  
  return Response.json(results)
}

2. RAG (Retrieval Augmented Generation):

javascript
async function ragPipeline(question, context) {
  // 1. Retrieve relevant documents
  const relevantDocs = await vectorSearch(question)
  
  // 2. Costruisci context
  const contextText = relevantDocs.map(d => d.content).join('\n\n')
  
  // 3. Genera risposta con LLM
  const prompt = `
Context: ${contextText}
Question: ${question}
Answer:`
  
  return await generateWithLLM(prompt)
}

3. Recommendation System:

javascript
// Recommendation basata su similarity
async function getRecommendations(userId, itemType, limit = 5) {
  // Ottieni embedding dell'item corrente o user preferences
  const userEmbedding = await getUserEmbedding(userId)
  
  // Cerca item simili
  const similarItems = await vectorDB.query({
    vector: userEmbedding,
    filter: { type: itemType },
    topK: limit
  })
  
  return similarItems
}

4. Deduplicazione Contenuti:

javascript
async function checkDuplicate(content, threshold = 0.9) {
  const embedding = await generateEmbedding(content)
  
  const similar = await vectorDB.query({
    vector: embedding,
    topK: 1,
    includeMetadata: true
  })
  
  if (similar.length > 0 && similar[0].score > threshold) {
    return { isDuplicate: true, existingId: similar[0].metadata.id }
  }
  
  return { isDuplicate: false }
}
Considerazioni Performance

Scaling Vector Search:

Per dataset piccoli (<100K vettori): pgvector o Chroma

Per dataset medi (100K-1M): Qdrant self-hosted

Per dataset grandi (>1M): Pinecone o Weaviate cloud

Per ricerca in tempo reale: Considera HNSW indexing

Per multi-tenant: Isolation con namespace/filter

Cost Optimization:

Usa batch operations per inserimenti

Implementa cache per query frequenti

Considera hybrid search (vector + keyword)

Usa dimensionality reduction se possibile

Monitora usage e ottimiza index parameters

===PROMPT_11_COMPLETATO===

---

## § POSTGRESQL ADVANCED PATTERNS PER NEXT.JS

### Connection Manager con Pool Intelligente

```typescript
// lib/db/postgres-connection.ts
import { Pool, PoolClient, QueryResult, QueryResultRow } from "pg";

interface ConnectionConfig {
  connectionString: string;
  maxConnections: number;
  idleTimeoutMs: number;
  connectionTimeoutMs: number;
  statementTimeout: number;
  enableSSL: boolean;
}

interface QueryOptions {
  timeout?: number;
  retries?: number;
  retryDelay?: number;
}

class PostgresConnectionManager {
  private pool: Pool;
  private static instance: PostgresConnectionManager | null = null;
  private activeQueries: Map<string, { startTime: number; query: string }> = new Map();
  private metrics = {
    totalQueries: 0,
    failedQueries: 0,
    avgExecutionTime: 0,
    poolSize: 0,
    idleConnections: 0,
    waitingClients: 0,
  };

  private constructor(config: ConnectionConfig) {
    this.pool = new Pool({
      connectionString: config.connectionString,
      max: config.maxConnections,
      idleTimeoutMillis: config.idleTimeoutMs,
      connectionTimeoutMillis: config.connectionTimeoutMs,
      statement_timeout: config.statementTimeout,
      ssl: config.enableSSL ? { rejectUnauthorized: false } : undefined,
    });

    this.pool.on("error", (err: Error) => {
      console.error("[PostgreSQL] Pool error:", err.message);
    });

    this.pool.on("connect", () => {
      this.updateMetrics();
    });
  }

  static getInstance(config?: ConnectionConfig): PostgresConnectionManager {
    if (!PostgresConnectionManager.instance) {
      if (!config) {
        config = {
          connectionString: process.env.DATABASE_URL!,
          maxConnections: parseInt(process.env.DB_MAX_CONNECTIONS || "20"),
          idleTimeoutMs: 30000,
          connectionTimeoutMs: 10000,
          statementTimeout: 30000,
          enableSSL: process.env.NODE_ENV === "production",
        };
      }
      PostgresConnectionManager.instance = new PostgresConnectionManager(config);
    }
    return PostgresConnectionManager.instance;
  }

  private updateMetrics(): void {
    this.metrics.poolSize = this.pool.totalCount;
    this.metrics.idleConnections = this.pool.idleCount;
    this.metrics.waitingClients = this.pool.waitingCount;
  }

  async query<T extends QueryResultRow>(
    text: string,
    params?: unknown[],
    options: QueryOptions = {}
  ): Promise<QueryResult<T>> {
    const queryId = crypto.randomUUID();
    const startTime = Date.now();
    const { timeout = 30000, retries = 2, retryDelay = 500 } = options;

    this.activeQueries.set(queryId, { startTime, query: text });
    this.metrics.totalQueries++;

    let lastError: Error | null = null;

    for (let attempt = 0; attempt <= retries; attempt++) {
      try {
        const client = await this.pool.connect();
        try {
          if (timeout) {
            await client.query(`SET statement_timeout = ${timeout}`);
          }
          const result = await client.query<T>(text, params);
          const executionTime = Date.now() - startTime;
          this.metrics.avgExecutionTime =
            (this.metrics.avgExecutionTime * (this.metrics.totalQueries - 1) + executionTime) /
            this.metrics.totalQueries;
          return result;
        } finally {
          client.release();
        }
      } catch (error) {
        lastError = error as Error;
        this.metrics.failedQueries++;
        if (attempt < retries) {
          await new Promise((resolve) => setTimeout(resolve, retryDelay * (attempt + 1)));
        }
      } finally {
        this.activeQueries.delete(queryId);
      }
    }
    throw lastError;
  }

  async transaction<T>(callback: (client: PoolClient) => Promise<T>): Promise<T> {
    const client = await this.pool.connect();
    try {
      await client.query("BEGIN");
      const result = await callback(client);
      await client.query("COMMIT");
      return result;
    } catch (error) {
      await client.query("ROLLBACK");
      throw error;
    } finally {
      client.release();
    }
  }

  async healthCheck(): Promise<{ healthy: boolean; latency: number; metrics: typeof this.metrics }> {
    const start = Date.now();
    try {
      await this.pool.query("SELECT 1");
      return { healthy: true, latency: Date.now() - start, metrics: { ...this.metrics } };
    } catch {
      return { healthy: false, latency: Date.now() - start, metrics: { ...this.metrics } };
    }
  }

  async close(): Promise<void> {
    await this.pool.end();
    PostgresConnectionManager.instance = null;
  }
}

export const db = PostgresConnectionManager.getInstance();
```

### Repository Pattern con PostgreSQL

```typescript
// lib/db/base-repository.ts
import { QueryResultRow } from "pg";
import { db } from "./postgres-connection";

interface PaginationOptions {
  page: number;
  limit: number;
  sortBy?: string;
  sortOrder?: "ASC" | "DESC";
}

interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
  hasNext: boolean;
  hasPrevious: boolean;
}

interface WhereClause {
  field: string;
  operator: "=" | "!=" | ">" | "<" | ">=" | "<=" | "LIKE" | "ILIKE" | "IN" | "IS NULL" | "IS NOT NULL";
  value?: unknown;
}

export abstract class BaseRepository<T extends QueryResultRow> {
  constructor(
    protected readonly tableName: string,
    protected readonly primaryKey: string = "id"
  ) {}

  async findById(id: string | number): Promise<T | null> {
    const result = await db.query<T>(
      `SELECT * FROM ${this.tableName} WHERE ${this.primaryKey} = $1`,
      [id]
    );
    return result.rows[0] || null;
  }

  async findMany(
    where: WhereClause[] = [],
    pagination?: PaginationOptions
  ): Promise<PaginatedResult<T>> {
    const { whereSQL, params } = this.buildWhereClause(where);
    const countResult = await db.query<{ count: string }>(
      `SELECT COUNT(*) as count FROM ${this.tableName} ${whereSQL}`,
      params
    );
    const total = parseInt(countResult.rows[0].count, 10);
    const page = pagination?.page || 1;
    const limit = pagination?.limit || 25;
    const offset = (page - 1) * limit;
    const sortBy = pagination?.sortBy || this.primaryKey;
    const sortOrder = pagination?.sortOrder || "DESC";

    const dataResult = await db.query<T>(
      `SELECT * FROM ${this.tableName} ${whereSQL}
       ORDER BY ${sortBy} ${sortOrder}
       LIMIT $${params.length + 1} OFFSET $${params.length + 2}`,
      [...params, limit, offset]
    );

    const totalPages = Math.ceil(total / limit);
    return {
      data: dataResult.rows,
      total, page, limit, totalPages,
      hasNext: page < totalPages,
      hasPrevious: page > 1,
    };
  }

  async create(data: Partial<T>): Promise<T> {
    const keys = Object.keys(data);
    const values = Object.values(data);
    const placeholders = keys.map((_, i) => `$${i + 1}`).join(", ");
    const columns = keys.join(", ");

    const result = await db.query<T>(
      `INSERT INTO ${this.tableName} (${columns}) VALUES (${placeholders}) RETURNING *`,
      values
    );
    return result.rows[0];
  }

  async update(id: string | number, data: Partial<T>): Promise<T | null> {
    const keys = Object.keys(data);
    const values = Object.values(data);
    const setClause = keys.map((key, i) => `${key} = $${i + 1}`).join(", ");

    const result = await db.query<T>(
      `UPDATE ${this.tableName} SET ${setClause}, updated_at = NOW()
       WHERE ${this.primaryKey} = $${keys.length + 1} RETURNING *`,
      [...values, id]
    );
    return result.rows[0] || null;
  }

  async delete(id: string | number): Promise<boolean> {
    const result = await db.query(
      `DELETE FROM ${this.tableName} WHERE ${this.primaryKey} = $1`,
      [id]
    );
    return (result.rowCount ?? 0) > 0;
  }

  async softDelete(id: string | number): Promise<T | null> {
    const result = await db.query<T>(
      `UPDATE ${this.tableName} SET deleted_at = NOW()
       WHERE ${this.primaryKey} = $1 RETURNING *`,
      [id]
    );
    return result.rows[0] || null;
  }

  async upsert(data: Partial<T>, conflictColumns: string[]): Promise<T> {
    const keys = Object.keys(data);
    const values = Object.values(data);
    const placeholders = keys.map((_, i) => `$${i + 1}`).join(", ");
    const columns = keys.join(", ");
    const conflict = conflictColumns.join(", ");
    const updateClause = keys
      .filter((k) => !conflictColumns.includes(k))
      .map((key) => `${key} = EXCLUDED.${key}`)
      .join(", ");

    const result = await db.query<T>(
      `INSERT INTO ${this.tableName} (${columns}) VALUES (${placeholders})
       ON CONFLICT (${conflict}) DO UPDATE SET ${updateClause}, updated_at = NOW()
       RETURNING *`,
      values
    );
    return result.rows[0];
  }

  private buildWhereClause(conditions: WhereClause[]): { whereSQL: string; params: unknown[] } {
    if (conditions.length === 0) return { whereSQL: "", params: [] };
    const params: unknown[] = [];
    const clauses = conditions.map((c) => {
      if (c.operator === "IS NULL" || c.operator === "IS NOT NULL") {
        return `${c.field} ${c.operator}`;
      }
      if (c.operator === "IN") {
        const arr = c.value as unknown[];
        const ph = arr.map((_, i) => `$${params.length + i + 1}`).join(", ");
        params.push(...arr);
        return `${c.field} IN (${ph})`;
      }
      params.push(c.value);
      return `${c.field} ${c.operator} $${params.length}`;
    });
    return { whereSQL: `WHERE ${clauses.join(" AND ")}`, params };
  }
}

// Esempio concreto: UserRepository
interface UserRow extends QueryResultRow {
  id: string;
  email: string;
  name: string;
  role: "admin" | "user" | "moderator";
  avatar_url: string | null;
  created_at: Date;
  updated_at: Date;
  deleted_at: Date | null;
}

export class UserRepository extends BaseRepository<UserRow> {
  constructor() {
    super("users", "id");
  }

  async findByEmail(email: string): Promise<UserRow | null> {
    const result = await db.query<UserRow>(
      "SELECT * FROM users WHERE email = $1 AND deleted_at IS NULL",
      [email]
    );
    return result.rows[0] || null;
  }

  async findByRole(role: string, pagination: PaginationOptions): Promise<PaginatedResult<UserRow>> {
    return this.findMany(
      [{ field: "role", operator: "=", value: role }, { field: "deleted_at", operator: "IS NULL" }],
      pagination
    );
  }

  async searchByName(search: string, limit: number = 20): Promise<UserRow[]> {
    const result = await db.query<UserRow>(
      `SELECT * FROM users WHERE name ILIKE $1 AND deleted_at IS NULL ORDER BY name ASC LIMIT $2`,
      [`%${search}%`, limit]
    );
    return result.rows;
  }
}
```

### Migration System Completo

```typescript
// lib/db/migration-runner.ts
import { db } from "./postgres-connection";
import { readdir, readFile } from "fs/promises";
import { join } from "path";

interface Migration {
  id: number;
  name: string;
  executed_at: Date;
  checksum: string;
}

interface MigrationFile {
  version: number;
  name: string;
  upSQL: string;
  downSQL: string;
  checksum: string;
}

export class MigrationRunner {
  private migrationsDir: string;

  constructor(migrationsDir: string = join(process.cwd(), "migrations")) {
    this.migrationsDir = migrationsDir;
  }

  async initialize(): Promise<void> {
    await db.query(`
      CREATE TABLE IF NOT EXISTS _migrations (
        id SERIAL PRIMARY KEY,
        version INTEGER NOT NULL UNIQUE,
        name VARCHAR(255) NOT NULL,
        executed_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
        checksum VARCHAR(64) NOT NULL,
        execution_time_ms INTEGER
      )
    `);
  }

  private async getExecutedMigrations(): Promise<Migration[]> {
    const result = await db.query<Migration>("SELECT * FROM _migrations ORDER BY version ASC");
    return result.rows;
  }

  private async loadMigrationFiles(): Promise<MigrationFile[]> {
    const files = await readdir(this.migrationsDir);
    const migrationFiles: MigrationFile[] = [];

    for (const file of files.filter((f) => f.endsWith(".sql"))) {
      const match = file.match(/^(\d+)[-_](.+)\.sql$/);
      if (!match) continue;
      const version = parseInt(match[1], 10);
      const name = match[2];
      const content = await readFile(join(this.migrationsDir, file), "utf-8");
      const parts = content.split("-- DOWN");
      const upSQL = parts[0].replace("-- UP", "").trim();
      const downSQL = parts[1]?.trim() || "";
      const encoder = new TextEncoder();
      const data = encoder.encode(content);
      const hashBuffer = await crypto.subtle.digest("SHA-256", data);
      const checksum = Array.from(new Uint8Array(hashBuffer))
        .map((b) => b.toString(16).padStart(2, "0"))
        .join("");
      migrationFiles.push({ version, name, upSQL, downSQL, checksum });
    }
    return migrationFiles.sort((a, b) => a.version - b.version);
  }

  async migrate(): Promise<{ applied: string[]; skipped: string[] }> {
    await this.initialize();
    const executed = await this.getExecutedMigrations();
    const files = await this.loadMigrationFiles();
    const executedVersions = new Set(executed.map((m) => m.id));
    const applied: string[] = [];
    const skipped: string[] = [];

    for (const file of files) {
      if (executedVersions.has(file.version)) {
        skipped.push(file.name);
        continue;
      }
      const startTime = Date.now();
      await db.transaction(async (client) => {
        await client.query(file.upSQL);
        await client.query(
          `INSERT INTO _migrations (version, name, checksum, execution_time_ms) VALUES ($1, $2, $3, $4)`,
          [file.version, file.name, file.checksum, Date.now() - startTime]
        );
      });
      applied.push(file.name);
    }
    return { applied, skipped };
  }

  async rollback(steps: number = 1): Promise<string[]> {
    await this.initialize();
    const executed = await this.getExecutedMigrations();
    const files = await this.loadMigrationFiles();
    const rolledBack: string[] = [];
    const toRollback = executed.slice(-steps).reverse();

    for (const migration of toRollback) {
      const file = files.find((f) => f.version === migration.id);
      if (!file || !file.downSQL) throw new Error(`No down migration for ${migration.name}`);
      await db.transaction(async (client) => {
        await client.query(file.downSQL);
        await client.query("DELETE FROM _migrations WHERE version = $1", [file.version]);
      });
      rolledBack.push(migration.name);
    }
    return rolledBack;
  }
}
```

### Database Health API Route

```typescript
// app/api/db/health/route.ts
import { NextResponse } from "next/server";
import { db } from "@/lib/db/postgres-connection";

interface DatabaseStats {
  version: string;
  uptime: string;
  activeConnections: number;
  maxConnections: number;
  databaseSize: string;
  cacheHitRatio: number;
  transactionsPerSecond: number;
  deadlocks: number;
  slowQueries: number;
  topTables: Array<{
    tableName: string;
    rowCount: number;
    totalSize: string;
    indexSize: string;
  }>;
}

async function getDatabaseStats(): Promise<DatabaseStats> {
  const [versionResult, uptimeResult, connectionsResult, dbSizeResult, cacheResult, tpsResult, deadlockResult, tableStatsResult] =
    await Promise.all([
      db.query<{ version: string }>("SELECT version()"),
      db.query<{ uptime: string }>("SELECT now() - pg_postmaster_start_time() as uptime"),
      db.query<{ active: string; max: string }>(`
        SELECT
          (SELECT count(*) FROM pg_stat_activity WHERE state = 'active')::text as active,
          (SELECT setting FROM pg_settings WHERE name = 'max_connections') as max
      `),
      db.query<{ size: string }>("SELECT pg_size_pretty(pg_database_size(current_database())) as size"),
      db.query<{ ratio: number }>(`
        SELECT ROUND((sum(heap_blks_hit) / NULLIF(sum(heap_blks_hit) + sum(heap_blks_read), 0)) * 100, 2) as ratio
        FROM pg_statio_user_tables
      `),
      db.query<{ tps: number }>(`
        SELECT COALESCE((xact_commit + xact_rollback)::numeric /
          NULLIF(EXTRACT(EPOCH FROM (now() - stats_reset)), 0), 0)::numeric(10,2) as tps
        FROM pg_stat_database WHERE datname = current_database()
      `),
      db.query<{ deadlocks: string }>("SELECT deadlocks FROM pg_stat_database WHERE datname = current_database()"),
      db.query<{ table_name: string; row_count: string; total_size: string; index_size: string }>(`
        SELECT relname as table_name, n_live_tup::text as row_count,
          pg_size_pretty(pg_total_relation_size(relid)) as total_size,
          pg_size_pretty(pg_indexes_size(relid)) as index_size
        FROM pg_stat_user_tables ORDER BY pg_total_relation_size(relid) DESC LIMIT 10
      `),
    ]);

  return {
    version: versionResult.rows[0].version,
    uptime: uptimeResult.rows[0].uptime,
    activeConnections: parseInt(connectionsResult.rows[0].active),
    maxConnections: parseInt(connectionsResult.rows[0].max),
    databaseSize: dbSizeResult.rows[0].size,
    cacheHitRatio: cacheResult.rows[0].ratio || 0,
    transactionsPerSecond: tpsResult.rows[0].tps,
    deadlocks: parseInt(deadlockResult.rows[0].deadlocks),
    slowQueries: 0,
    topTables: tableStatsResult.rows.map((r) => ({
      tableName: r.table_name,
      rowCount: parseInt(r.row_count),
      totalSize: r.total_size,
      indexSize: r.index_size,
    })),
  };
}

export async function GET() {
  try {
    const stats = await getDatabaseStats();
    return NextResponse.json({ status: "healthy", timestamp: new Date().toISOString(), ...stats });
  } catch (error) {
    return NextResponse.json(
      { status: "unhealthy", timestamp: new Date().toISOString(), error: error instanceof Error ? error.message : "Unknown" },
      { status: 503 }
    );
  }
}
```

## § MONGODB COMPLETE SERVICE LAYER

### MongoDB Typed Service

```typescript
// lib/db/mongodb-service.ts
import {
  MongoClient, Db, Collection, Filter, Sort, UpdateFilter,
  OptionalUnlessRequiredId, WithId, ObjectId, Document,
} from "mongodb";

interface PaginationParams {
  page: number;
  limit: number;
  sort?: Sort;
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: { total: number; page: number; limit: number; pages: number; hasNext: boolean; hasPrev: boolean };
}

class MongoDBService {
  private static instance: MongoDBService;
  private client: MongoClient;
  private db: Db | null = null;

  private constructor(uri: string, private dbName: string) {
    this.client = new MongoClient(uri, {
      maxPoolSize: 20,
      minPoolSize: 5,
      serverSelectionTimeoutMS: 5000,
      socketTimeoutMS: 45000,
      retryWrites: true,
      retryReads: true,
    });
  }

  static getInstance(): MongoDBService {
    if (!MongoDBService.instance) {
      MongoDBService.instance = new MongoDBService(
        process.env.MONGODB_URI!,
        process.env.MONGODB_DB_NAME || "nextjs_app"
      );
    }
    return MongoDBService.instance;
  }

  async connect(): Promise<Db> {
    if (this.db) return this.db;
    await this.client.connect();
    this.db = this.client.db(this.dbName);
    return this.db;
  }

  async getCollection<T extends Document>(name: string): Promise<Collection<T>> {
    const db = await this.connect();
    return db.collection<T>(name);
  }

  async findWithPagination<T extends Document>(
    collectionName: string,
    filter: Filter<T>,
    pagination: PaginationParams
  ): Promise<PaginatedResponse<WithId<T>>> {
    const collection = await this.getCollection<T>(collectionName);
    const { page, limit, sort } = pagination;
    const skip = (page - 1) * limit;

    const [data, total] = await Promise.all([
      collection.find(filter).sort(sort || { _id: -1 }).skip(skip).limit(limit).toArray(),
      collection.countDocuments(filter),
    ]);

    const pages = Math.ceil(total / limit);
    return {
      data,
      pagination: { total, page, limit, pages, hasNext: page < pages, hasPrev: page > 1 },
    };
  }

  async bulkUpsert<T extends Document>(
    collectionName: string,
    documents: OptionalUnlessRequiredId<T>[],
    uniqueField: keyof T
  ): Promise<{ upserted: number; modified: number }> {
    const collection = await this.getCollection<T>(collectionName);
    const operations = documents.map((doc) => ({
      updateOne: {
        filter: { [uniqueField]: doc[uniqueField as string] } as Filter<T>,
        update: { $set: doc } as UpdateFilter<T>,
        upsert: true,
      },
    }));
    const result = await collection.bulkWrite(operations, { ordered: false });
    return { upserted: result.upsertedCount, modified: result.modifiedCount };
  }

  async aggregate<T extends Document, R extends Document>(
    collectionName: string,
    pipeline: Document[]
  ): Promise<R[]> {
    const collection = await this.getCollection<T>(collectionName);
    return collection.aggregate<R>(pipeline).toArray();
  }

  async healthCheck(): Promise<{ connected: boolean; latencyMs: number }> {
    const start = Date.now();
    try {
      const db = await this.connect();
      await db.command({ ping: 1 });
      return { connected: true, latencyMs: Date.now() - start };
    } catch {
      return { connected: false, latencyMs: Date.now() - start };
    }
  }
}

export const mongoService = MongoDBService.getInstance();
```

### MongoDB Analytics Aggregation

```typescript
// lib/db/mongodb-analytics.ts
import { mongoService } from "./mongodb-service";

interface DailyRevenueReport {
  _id: { year: number; month: number; day: number };
  totalRevenue: number;
  orderCount: number;
  averageOrderValue: number;
  uniqueCustomers: number;
}

interface CustomerSegment {
  _id: string;
  customerCount: number;
  totalSpent: number;
  averageSpent: number;
  retentionRate: number;
}

export class AnalyticsService {
  async getDailyRevenue(startDate: Date, endDate: Date): Promise<DailyRevenueReport[]> {
    return mongoService.aggregate("orders", [
      {
        $match: {
          createdAt: { $gte: startDate, $lte: endDate },
          status: { $in: ["completed", "shipped"] },
        },
      },
      {
        $group: {
          _id: {
            year: { $year: "$createdAt" },
            month: { $month: "$createdAt" },
            day: { $dayOfMonth: "$createdAt" },
          },
          totalRevenue: { $sum: "$totalAmount" },
          orderIds: { $addToSet: "$_id" },
          uniqueCustomers: { $addToSet: "$customerId" },
        },
      },
      {
        $project: {
          totalRevenue: 1,
          orderCount: { $size: "$orderIds" },
          uniqueCustomers: { $size: "$uniqueCustomers" },
          averageOrderValue: { $divide: ["$totalRevenue", { $size: "$orderIds" }] },
        },
      },
      { $sort: { "_id.year": 1, "_id.month": 1, "_id.day": 1 } },
    ]);
  }

  async getCustomerSegments(): Promise<CustomerSegment[]> {
    return mongoService.aggregate("orders", [
      {
        $group: {
          _id: "$customerId",
          totalSpent: { $sum: "$totalAmount" },
          orderCount: { $sum: 1 },
          lastOrder: { $max: "$createdAt" },
        },
      },
      {
        $addFields: {
          segment: {
            $switch: {
              branches: [
                { case: { $gte: ["$totalSpent", 10000] }, then: "vip" },
                { case: { $gte: ["$totalSpent", 5000] }, then: "premium" },
                { case: { $gte: ["$totalSpent", 1000] }, then: "regular" },
                { case: { $gte: ["$orderCount", 3] }, then: "returning" },
              ],
              default: "new",
            },
          },
        },
      },
      {
        $group: {
          _id: "$segment",
          customerCount: { $sum: 1 },
          totalSpent: { $sum: "$totalSpent" },
          averageSpent: { $avg: "$totalSpent" },
          activeCustomers: {
            $sum: {
              $cond: [
                { $gte: ["$lastOrder", { $subtract: [new Date(), 90 * 24 * 60 * 60 * 1000] }] },
                1, 0,
              ],
            },
          },
          total: { $sum: 1 },
        },
      },
      {
        $addFields: {
          retentionRate: { $round: [{ $multiply: [{ $divide: ["$activeCustomers", "$total"] }, 100] }, 2] },
        },
      },
      { $sort: { totalSpent: -1 } },
    ]);
  }

  async getConversionFunnel(startDate: Date, endDate: Date) {
    const dateFilter = { createdAt: { $gte: startDate, $lte: endDate } };

    const [visitors, signups, carts, checkouts, purchases] = await Promise.all([
      mongoService.aggregate("page_views", [
        { $match: dateFilter },
        { $group: { _id: null, count: { $addToSet: "$sessionId" } } },
        { $project: { count: { $size: "$count" } } },
      ]).then((r) => r[0]?.count || 0),
      mongoService.aggregate("users", [{ $match: dateFilter }, { $count: "count" }]).then((r) => r[0]?.count || 0),
      mongoService.aggregate("carts", [
        { $match: { ...dateFilter, "items.0": { $exists: true } } }, { $count: "count" },
      ]).then((r) => r[0]?.count || 0),
      mongoService.aggregate("checkouts", [{ $match: dateFilter }, { $count: "count" }]).then((r) => r[0]?.count || 0),
      mongoService.aggregate("orders", [
        { $match: { ...dateFilter, status: "completed" } }, { $count: "count" },
      ]).then((r) => r[0]?.count || 0),
    ]);

    const stages = [
      { stage: "Visitors", count: visitors },
      { stage: "Signups", count: signups },
      { stage: "Add to Cart", count: carts },
      { stage: "Checkout", count: checkouts },
      { stage: "Purchase", count: purchases },
    ];

    return stages.map((s, i) => ({
      ...s,
      conversionRate: i === 0 ? 100 : Math.round((s.count / stages[0].count) * 10000) / 100,
      dropoffRate: i === 0 ? 0 : Math.round(((stages[i - 1].count - s.count) / stages[i - 1].count) * 10000) / 100,
    }));
  }
}

export const analytics = new AnalyticsService();
```

## § REDIS MULTI-LAYER CACHE

### Cache Manager con Local + Redis

```typescript
// lib/cache/redis-cache-manager.ts
import { Redis } from "@upstash/redis";

type Serializable = string | number | boolean | null | object;

interface CacheEntry<T> {
  data: T;
  cachedAt: number;
  ttl: number;
  tags: string[];
}

interface CacheStats {
  hits: number;
  misses: number;
  localHits: number;
  redisHits: number;
  hitRate: number;
  avgLatency: number;
}

class RedisCacheManager {
  private redis: Redis;
  private localCache: Map<string, { data: unknown; expiresAt: number }> = new Map();
  private prefix: string;
  private localCacheMaxSize: number;
  private localCacheTTL: number;
  private defaultTTL: number;
  private stats: CacheStats = {
    hits: 0, misses: 0, localHits: 0, redisHits: 0, hitRate: 0, avgLatency: 0,
  };

  constructor(options?: { prefix?: string; defaultTTL?: number; localCacheMaxSize?: number; localCacheTTL?: number }) {
    this.redis = new Redis({
      url: process.env.UPSTASH_REDIS_REST_URL!,
      token: process.env.UPSTASH_REDIS_REST_TOKEN!,
    });
    this.prefix = options?.prefix || "app:";
    this.defaultTTL = options?.defaultTTL || 300;
    this.localCacheMaxSize = options?.localCacheMaxSize || 1000;
    this.localCacheTTL = options?.localCacheTTL || 30;
  }

  private getKey(key: string): string {
    return `${this.prefix}${key}`;
  }

  private getFromLocal<T>(key: string): T | null {
    const entry = this.localCache.get(key);
    if (!entry) return null;
    if (Date.now() > entry.expiresAt) {
      this.localCache.delete(key);
      return null;
    }
    return entry.data as T;
  }

  private setLocal(key: string, data: unknown): void {
    if (this.localCache.size >= this.localCacheMaxSize) {
      const firstKey = this.localCache.keys().next().value;
      if (firstKey) this.localCache.delete(firstKey);
    }
    this.localCache.set(key, { data, expiresAt: Date.now() + this.localCacheTTL * 1000 });
  }

  async get<T extends Serializable>(key: string): Promise<T | null> {
    const fullKey = this.getKey(key);
    const startTime = Date.now();

    const localResult = this.getFromLocal<CacheEntry<T>>(fullKey);
    if (localResult) {
      this.stats.hits++;
      this.stats.localHits++;
      this.updateLatency(Date.now() - startTime);
      return localResult.data;
    }

    try {
      const redisResult = await this.redis.get<CacheEntry<T>>(fullKey);
      if (redisResult) {
        this.stats.hits++;
        this.stats.redisHits++;
        this.setLocal(fullKey, redisResult);
        this.updateLatency(Date.now() - startTime);
        return redisResult.data;
      }
    } catch (error) {
      console.error("[Cache] Redis get error:", error);
    }

    this.stats.misses++;
    this.updateLatency(Date.now() - startTime);
    return null;
  }

  async set<T extends Serializable>(key: string, data: T, ttl?: number, tags: string[] = []): Promise<void> {
    const fullKey = this.getKey(key);
    const effectiveTTL = ttl || this.defaultTTL;
    const entry: CacheEntry<T> = { data, cachedAt: Date.now(), ttl: effectiveTTL, tags };

    try {
      await this.redis.setex(fullKey, effectiveTTL, JSON.stringify(entry));
      this.setLocal(fullKey, entry);
      for (const tag of tags) {
        const tagKey = `${this.prefix}tag:${tag}`;
        await this.redis.sadd(tagKey, fullKey);
        await this.redis.expire(tagKey, effectiveTTL * 2);
      }
    } catch (error) {
      console.error("[Cache] Redis set error:", error);
    }
  }

  async getOrSet<T extends Serializable>(key: string, fetcher: () => Promise<T>, ttl?: number, tags?: string[]): Promise<T> {
    const cached = await this.get<T>(key);
    if (cached !== null) return cached;
    const data = await fetcher();
    await this.set(key, data, ttl, tags);
    return data;
  }

  async invalidate(key: string): Promise<void> {
    const fullKey = this.getKey(key);
    await this.redis.del(fullKey);
    this.localCache.delete(fullKey);
  }

  async invalidateByTag(tag: string): Promise<number> {
    const tagKey = `${this.prefix}tag:${tag}`;
    const keys = await this.redis.smembers(tagKey);
    if (keys.length === 0) return 0;
    const pipeline = this.redis.pipeline();
    for (const key of keys) {
      pipeline.del(key);
      this.localCache.delete(key);
    }
    pipeline.del(tagKey);
    await pipeline.exec();
    return keys.length;
  }

  private updateLatency(latency: number): void {
    const total = this.stats.hits + this.stats.misses;
    this.stats.avgLatency = (this.stats.avgLatency * (total - 1) + latency) / total;
    this.stats.hitRate = total > 0 ? Math.round((this.stats.hits / total) * 10000) / 100 : 0;
  }

  getStats(): CacheStats {
    return { ...this.stats };
  }
}

export const cache = new RedisCacheManager();
```

### Redis Queue per Background Jobs

```typescript
// lib/queue/redis-queue.ts
import { Redis } from "@upstash/redis";

interface Job<T = unknown> {
  id: string;
  queue: string;
  payload: T;
  status: "pending" | "processing" | "completed" | "failed" | "retrying";
  attempts: number;
  maxAttempts: number;
  createdAt: number;
  completedAt: number | null;
  error: string | null;
  result: unknown | null;
  priority: number;
}

type JobHandler<T = unknown, R = unknown> = (payload: T) => Promise<R>;

class RedisQueue {
  private redis: Redis;
  private handlers: Map<string, JobHandler> = new Map();
  private prefix: string;

  constructor(prefix: string = "queue:") {
    this.redis = new Redis({
      url: process.env.UPSTASH_REDIS_REST_URL!,
      token: process.env.UPSTASH_REDIS_REST_TOKEN!,
    });
    this.prefix = prefix;
  }

  async enqueue<T>(queueName: string, payload: T, options?: { maxAttempts?: number; priority?: number; delay?: number }): Promise<string> {
    const jobId = crypto.randomUUID();
    const job: Job<T> = {
      id: jobId,
      queue: queueName,
      payload,
      status: "pending",
      attempts: 0,
      maxAttempts: options?.maxAttempts || 3,
      createdAt: Date.now(),
      completedAt: null,
      error: null,
      result: null,
      priority: options?.priority || 0,
    };

    const jobKey = `${this.prefix}job:${jobId}`;
    await this.redis.set(jobKey, JSON.stringify(job));

    if (options?.delay) {
      await this.redis.zadd(`${this.prefix}${queueName}:delayed`, { score: Date.now() + options.delay, member: jobId });
    } else {
      await this.redis.zadd(`${this.prefix}${queueName}:pending`, { score: job.priority, member: jobId });
    }
    return jobId;
  }

  registerHandler<T, R>(queueName: string, handler: JobHandler<T, R>): void {
    this.handlers.set(queueName, handler as JobHandler);
  }

  async processNext(queueName: string): Promise<Job | null> {
    const queueKey = `${this.prefix}${queueName}:pending`;
    const members = await this.redis.zpopmin<string[]>(queueKey, 1);
    if (!members || members.length === 0) return null;

    const jobId = members[0];
    const jobKey = `${this.prefix}job:${jobId}`;
    const jobData = await this.redis.get<string>(jobKey);
    if (!jobData) return null;

    const job: Job = typeof jobData === "string" ? JSON.parse(jobData) : jobData;
    job.status = "processing";
    job.attempts++;

    const handler = this.handlers.get(queueName);
    if (!handler) {
      job.status = "failed";
      job.error = "No handler registered";
      await this.redis.set(jobKey, JSON.stringify(job));
      return job;
    }

    try {
      const result = await handler(job.payload);
      job.status = "completed";
      job.completedAt = Date.now();
      job.result = result;
    } catch (error) {
      if (job.attempts < job.maxAttempts) {
        job.status = "retrying";
        job.error = error instanceof Error ? error.message : String(error);
        const retryDelay = Math.pow(2, job.attempts) * 1000;
        await this.redis.zadd(`${this.prefix}${queueName}:delayed`, { score: Date.now() + retryDelay, member: jobId });
      } else {
        job.status = "failed";
        job.error = error instanceof Error ? error.message : String(error);
      }
    }

    await this.redis.set(jobKey, JSON.stringify(job));
    return job;
  }

  async getQueueStats(queueName: string): Promise<{ pending: number; processing: number; failed: number; delayed: number }> {
    const [pending, processing, failed, delayed] = await Promise.all([
      this.redis.zcard(`${this.prefix}${queueName}:pending`),
      this.redis.scard(`${this.prefix}${queueName}:processing`),
      this.redis.scard(`${this.prefix}${queueName}:failed`),
      this.redis.zcard(`${this.prefix}${queueName}:delayed`),
    ]);
    return { pending, processing, failed, delayed };
  }
}

export const queue = new RedisQueue();
```

## § SUPABASE COMPLETE INTEGRATION

### Supabase Client Factory

```typescript
// lib/supabase/client.ts
import { createBrowserClient } from "@supabase/ssr";
import { createServerClient, type CookieOptions } from "@supabase/ssr";
import { cookies } from "next/headers";
import { type Database } from "@/types/supabase";

export function createSupabaseBrowserClient() {
  return createBrowserClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}

export async function createSupabaseServerClient() {
  const cookieStore = await cookies();
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) { return cookieStore.get(name)?.value; },
        set(name: string, value: string, options: CookieOptions) {
          try { cookieStore.set({ name, value, ...options }); } catch {}
        },
        remove(name: string, options: CookieOptions) {
          try { cookieStore.set({ name, value: "", ...options }); } catch {}
        },
      },
    }
  );
}

export async function createSupabaseAdminClient() {
  const cookieStore = await cookies();
  return createServerClient<Database>(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!,
    {
      cookies: {
        get(name: string) { return cookieStore.get(name)?.value; },
        set(name: string, value: string, options: CookieOptions) {
          try { cookieStore.set({ name, value, ...options }); } catch {}
        },
        remove(name: string, options: CookieOptions) {
          try { cookieStore.set({ name, value: "", ...options }); } catch {}
        },
      },
    }
  );
}
```

### Supabase CRUD Service

```typescript
// lib/supabase/crud-service.ts
import { SupabaseClient } from "@supabase/supabase-js";
import { Database } from "@/types/supabase";

type Tables = Database["public"]["Tables"];
type TableName = keyof Tables;

interface QueryOptions {
  select?: string;
  filters?: Array<{ column: string; operator: "eq" | "neq" | "gt" | "gte" | "lt" | "lte" | "like" | "ilike" | "in" | "is"; value: unknown }>;
  orderBy?: { column: string; ascending?: boolean };
  limit?: number;
  offset?: number;
}

export class SupabaseCrudService<T extends TableName> {
  constructor(private supabase: SupabaseClient<Database>, private table: T) {}

  async findAll(options: QueryOptions = {}) {
    let query = this.supabase.from(this.table).select(options.select || "*", { count: "exact" });

    if (options.filters) {
      for (const filter of options.filters) {
        if (filter.operator === "in") {
          query = query.in(filter.column, filter.value as unknown[]);
        } else if (filter.operator === "is") {
          query = query.is(filter.column, filter.value as null);
        } else {
          query = query.filter(filter.column, filter.operator, filter.value);
        }
      }
    }
    if (options.orderBy) query = query.order(options.orderBy.column, { ascending: options.orderBy.ascending ?? false });
    if (options.limit) query = query.limit(options.limit);
    if (options.offset) query = query.range(options.offset, options.offset + (options.limit || 25) - 1);

    const { data, error, count } = await query;
    return { data: (data as Tables[T]["Row"][]) || [], count: count || 0, error: error?.message || null };
  }

  async findById(id: string, select?: string) {
    const { data, error } = await this.supabase.from(this.table).select(select || "*").eq("id", id).single();
    return { data: data as Tables[T]["Row"] | null, error: error?.message || null, success: !error };
  }

  async create(record: Tables[T]["Insert"]) {
    const { data, error } = await this.supabase.from(this.table).insert(record as any).select().single();
    return { data: data as Tables[T]["Row"] | null, error: error?.message || null, success: !error };
  }

  async update(id: string, record: Tables[T]["Update"]) {
    const { data, error } = await this.supabase.from(this.table).update(record as any).eq("id", id).select().single();
    return { data: data as Tables[T]["Row"] | null, error: error?.message || null, success: !error };
  }

  async upsert(record: Tables[T]["Insert"], onConflict?: string) {
    const { data, error } = await this.supabase.from(this.table).upsert(record as any, { onConflict: onConflict || "id" }).select().single();
    return { data: data as Tables[T]["Row"] | null, error: error?.message || null, success: !error };
  }

  async delete(id: string) {
    const { error } = await this.supabase.from(this.table).delete().eq("id", id);
    return { success: !error, error: error?.message || null };
  }

  subscribe(event: "INSERT" | "UPDATE" | "DELETE" | "*", callback: (payload: { eventType: string; new: Tables[T]["Row"]; old: Tables[T]["Row"] }) => void) {
    const channel = this.supabase
      .channel(`${this.table}_changes`)
      .on("postgres_changes", { event, schema: "public", table: this.table as string }, (payload) => {
        callback({ eventType: payload.eventType, new: payload.new as Tables[T]["Row"], old: payload.old as Tables[T]["Row"] });
      })
      .subscribe();
    return () => { this.supabase.removeChannel(channel); };
  }
}
```

## § VECTOR DATABASE RAG PIPELINE

### RAG Pipeline con pgvector

```typescript
// lib/rag/vector-pipeline.ts
import { db } from "@/lib/db/postgres-connection";
import { OpenAI } from "openai";

interface SearchResult {
  id: string;
  content: string;
  metadata: Record<string, unknown>;
  similarity: number;
}

interface RAGResponse {
  answer: string;
  sources: SearchResult[];
  tokensUsed: number;
  latencyMs: number;
}

const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

export class VectorRAGPipeline {
  private embeddingModel: string;
  private completionModel: string;
  private chunkSize: number;
  private chunkOverlap: number;

  constructor(options?: { embeddingModel?: string; completionModel?: string; chunkSize?: number; chunkOverlap?: number }) {
    this.embeddingModel = options?.embeddingModel || "text-embedding-3-small";
    this.completionModel = options?.completionModel || "gpt-4o-mini";
    this.chunkSize = options?.chunkSize || 500;
    this.chunkOverlap = options?.chunkOverlap || 50;
  }

  async initializeSchema(): Promise<void> {
    await db.query("CREATE EXTENSION IF NOT EXISTS vector");
    await db.query(`
      CREATE TABLE IF NOT EXISTS document_chunks (
        id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
        document_id VARCHAR(255) NOT NULL,
        chunk_index INTEGER NOT NULL,
        content TEXT NOT NULL,
        metadata JSONB DEFAULT '{}',
        embedding vector(1536),
        created_at TIMESTAMPTZ DEFAULT NOW(),
        UNIQUE(document_id, chunk_index)
      )
    `);
    await db.query(`CREATE INDEX IF NOT EXISTS idx_chunks_embedding ON document_chunks USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100)`);
  }

  splitIntoChunks(text: string): string[] {
    const words = text.split(/\s+/);
    const chunks: string[] = [];
    let currentChunk: string[] = [];
    let wordCount = 0;

    for (const word of words) {
      currentChunk.push(word);
      wordCount++;
      if (wordCount >= this.chunkSize) {
        chunks.push(currentChunk.join(" "));
        const overlapWords = currentChunk.slice(-this.chunkOverlap);
        currentChunk = [...overlapWords];
        wordCount = overlapWords.length;
      }
    }
    if (currentChunk.length > 0) chunks.push(currentChunk.join(" "));
    return chunks;
  }

  async generateEmbedding(text: string): Promise<number[]> {
    const response = await openai.embeddings.create({ model: this.embeddingModel, input: text });
    return response.data[0].embedding;
  }

  async ingestDocument(documentId: string, content: string, metadata: Record<string, unknown> = {}): Promise<{ chunksCreated: number }> {
    const chunks = this.splitIntoChunks(content);
    await db.query("DELETE FROM document_chunks WHERE document_id = $1", [documentId]);

    for (let i = 0; i < chunks.length; i++) {
      const embedding = await this.generateEmbedding(chunks[i]);
      const embeddingStr = `[${embedding.join(",")}]`;
      await db.query(
        `INSERT INTO document_chunks (document_id, chunk_index, content, metadata, embedding) VALUES ($1, $2, $3, $4, $5::vector)`,
        [documentId, i, chunks[i], metadata, embeddingStr]
      );
    }
    return { chunksCreated: chunks.length };
  }

  async search(query: string, topK: number = 5, similarityThreshold: number = 0.7): Promise<SearchResult[]> {
    const queryEmbedding = await this.generateEmbedding(query);
    const embeddingStr = `[${queryEmbedding.join(",")}]`;

    const result = await db.query<{ id: string; content: string; metadata: Record<string, unknown>; similarity: number }>(
      `SELECT id, content, metadata, 1 - (embedding <=> $1::vector) as similarity
       FROM document_chunks
       WHERE 1 - (embedding <=> $1::vector) >= $3
       ORDER BY embedding <=> $1::vector LIMIT $2`,
      [embeddingStr, topK, similarityThreshold]
    );
    return result.rows;
  }

  async query(question: string, options?: { topK?: number; systemPrompt?: string; temperature?: number }): Promise<RAGResponse> {
    const startTime = Date.now();
    const sources = await this.search(question, options?.topK || 5);
    const contextText = sources.map((s, i) => `[Source ${i + 1}] (sim: ${s.similarity.toFixed(3)})\n${s.content}`).join("\n\n---\n\n");

    const systemPrompt = options?.systemPrompt || "You are a helpful assistant. Answer based ONLY on the provided context. Cite sources using [Source N] notation.";
    const response = await openai.chat.completions.create({
      model: this.completionModel,
      temperature: options?.temperature || 0.3,
      messages: [
        { role: "system", content: systemPrompt },
        { role: "user", content: `Context:\n${contextText}\n\nQuestion: ${question}` },
      ],
    });

    return {
      answer: response.choices[0].message.content || "",
      sources,
      tokensUsed: response.usage?.total_tokens || 0,
      latencyMs: Date.now() - startTime,
    };
  }
}

export const ragPipeline = new VectorRAGPipeline();
```

### RAG Search API Route

```typescript
// app/api/rag/search/route.ts
import { NextRequest, NextResponse } from "next/server";
import { ragPipeline } from "@/lib/rag/vector-pipeline";

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    if (!body.question || body.question.trim().length < 3) {
      return NextResponse.json({ error: "Question must be at least 3 characters" }, { status: 400 });
    }

    const result = await ragPipeline.query(body.question, {
      topK: body.topK || 5,
    });

    return NextResponse.json({
      answer: result.answer,
      sources: result.sources.map((s) => ({
        content: s.content.substring(0, 200) + "...",
        similarity: s.similarity,
        metadata: s.metadata,
      })),
      tokensUsed: result.tokensUsed,
      latencyMs: result.latencyMs,
    });
  } catch (error) {
    console.error("[RAG] Search error:", error);
    return NextResponse.json({ error: "Internal server error" }, { status: 500 });
  }
}

// app/api/rag/ingest/route.ts
export async function POST(request: NextRequest) {
  try {
    const { documentId, content, metadata } = await request.json();
    if (!documentId || !content) {
      return NextResponse.json({ error: "documentId and content are required" }, { status: 400 });
    }
    const result = await ragPipeline.ingestDocument(documentId, content, metadata || {});
    return NextResponse.json({ success: true, documentId, chunksCreated: result.chunksCreated });
  } catch (error) {
    console.error("[RAG] Ingest error:", error);
    return NextResponse.json({ error: "Failed to ingest document" }, { status: 500 });
  }
}
```

## § DATABASE MONITORING DASHBOARD

### Database Monitor Component

```typescript
// components/DatabaseMonitorDashboard.tsx
"use client";

import { useState, useEffect, useCallback } from "react";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Progress } from "@/components/ui/progress";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Activity, Database, Clock, AlertTriangle, CheckCircle, XCircle, TrendingUp, HardDrive } from "lucide-react";

interface HealthStatus {
  status: "healthy" | "degraded" | "unhealthy";
  timestamp: string;
  activeConnections: number;
  maxConnections: number;
  databaseSize: string;
  cacheHitRatio: number;
  transactionsPerSecond: number;
  deadlocks: number;
  slowQueries: number;
  topTables: Array<{ tableName: string; rowCount: number; totalSize: string; indexSize: string }>;
}

function StatusIcon({ status }: { status: string }) {
  switch (status) {
    case "healthy": return <CheckCircle className="h-5 w-5 text-green-500" />;
    case "degraded": return <AlertTriangle className="h-5 w-5 text-yellow-500" />;
    default: return <XCircle className="h-5 w-5 text-red-500" />;
  }
}

export function DatabaseMonitorDashboard() {
  const [health, setHealth] = useState<HealthStatus | null>(null);
  const [loading, setLoading] = useState(true);
  const [autoRefresh, setAutoRefresh] = useState(true);

  const fetchHealth = useCallback(async () => {
    try {
      const response = await fetch("/api/db/health");
      if (!response.ok) throw new Error("Failed to fetch");
      const data = await response.json();
      setHealth(data);
    } catch {
      setHealth(null);
    } finally {
      setLoading(false);
    }
  }, []);

  useEffect(() => { fetchHealth(); }, [fetchHealth]);

  useEffect(() => {
    if (!autoRefresh) return;
    const interval = setInterval(fetchHealth, 10000);
    return () => clearInterval(interval);
  }, [autoRefresh, fetchHealth]);

  if (loading) {
    return (
      <div className="flex items-center justify-center h-64">
        <Activity className="h-8 w-8 animate-spin text-muted-foreground" />
      </div>
    );
  }

  if (!health) {
    return (
      <Card className="border-red-500">
        <CardContent className="pt-6">
          <div className="flex items-center gap-3">
            <XCircle className="h-6 w-6 text-red-500" />
            <p className="font-semibold">Database Unreachable</p>
          </div>
        </CardContent>
      </Card>
    );
  }

  const connectionUsage = (health.activeConnections / health.maxConnections) * 100;

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <div className="flex items-center gap-3">
          <StatusIcon status={health.status} />
          <h2 className="text-2xl font-bold">Database Monitor</h2>
          <Badge variant={health.status === "healthy" ? "default" : "destructive"}>
            {health.status.toUpperCase()}
          </Badge>
        </div>
        <button
          onClick={() => setAutoRefresh(!autoRefresh)}
          className={`text-xs px-2 py-1 rounded ${autoRefresh ? "bg-green-100 text-green-700" : "bg-gray-100 text-gray-600"}`}
        >
          {autoRefresh ? "Auto-refresh ON" : "Auto-refresh OFF"}
        </button>
      </div>

      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium flex items-center gap-2">
              <Database className="h-4 w-4" /> Connections
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{health.activeConnections}/{health.maxConnections}</div>
            <Progress value={connectionUsage} className="mt-2 h-2" />
            <p className="text-xs text-muted-foreground mt-1">{connectionUsage.toFixed(1)}% utilized</p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium flex items-center gap-2">
              <TrendingUp className="h-4 w-4" /> Cache Hit Ratio
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{health.cacheHitRatio}%</div>
            <Progress value={health.cacheHitRatio} className="mt-2 h-2" />
            <p className="text-xs text-muted-foreground mt-1">
              {health.cacheHitRatio >= 99 ? "Excellent" : health.cacheHitRatio >= 95 ? "Good" : "Needs attention"}
            </p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium flex items-center gap-2">
              <HardDrive className="h-4 w-4" /> Database Size
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="text-2xl font-bold">{health.databaseSize}</div>
            <p className="text-xs text-muted-foreground mt-1">TPS: {health.transactionsPerSecond}/s</p>
          </CardContent>
        </Card>

        <Card>
          <CardHeader className="pb-2">
            <CardTitle className="text-sm font-medium flex items-center gap-2">
              <Clock className="h-4 w-4" /> Issues
            </CardTitle>
          </CardHeader>
          <CardContent>
            <div className="flex gap-2 mt-2">
              {health.deadlocks > 0 && <Badge variant="destructive">{health.deadlocks} deadlocks</Badge>}
              {health.slowQueries > 0 && <Badge variant="secondary">{health.slowQueries} slow queries</Badge>}
              {health.deadlocks === 0 && health.slowQueries === 0 && <Badge variant="default">All clear</Badge>}
            </div>
          </CardContent>
        </Card>
      </div>

      <Card>
        <CardHeader><CardTitle>Top Tables by Size</CardTitle></CardHeader>
        <CardContent>
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b">
                <th className="text-left py-2 px-3">Table</th>
                <th className="text-right py-2 px-3">Rows</th>
                <th className="text-right py-2 px-3">Total Size</th>
                <th className="text-right py-2 px-3">Index Size</th>
              </tr>
            </thead>
            <tbody>
              {health.topTables.map((table) => (
                <tr key={table.tableName} className="border-b">
                  <td className="py-2 px-3 font-mono">{table.tableName}</td>
                  <td className="text-right py-2 px-3">{table.rowCount.toLocaleString()}</td>
                  <td className="text-right py-2 px-3">{table.totalSize}</td>
                  <td className="text-right py-2 px-3">{table.indexSize}</td>
                </tr>
              ))}
            </tbody>
          </table>
        </CardContent>
      </Card>
    </div>
  );
}
```


## § ADVANCED PATTERNS: DATABASE SELECTION

### Server Actions con Validazione

```typescript
// app/actions/database-selection.ts
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


### DATABASE SELECTION - Utility Helper #172

```typescript
// lib/utils/database-selection-helper-172.ts
import { z } from "zod";

interface DATABASESELECTIONConfig {
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
  maxRetries: z.number().min(0).max(10).de