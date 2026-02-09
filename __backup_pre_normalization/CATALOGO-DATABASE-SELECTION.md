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