# CATALOGO DATABASE SELECTION GUIDE v2

> **Versione**: 2.0
> **Data**: 2026-01-27
> **Ambito**: Database selection, scaling, optimization, pooling, migrations, multi-tenancy, backup/DR, monitoring
> **Sezioni**: 1-20 (12 originali + 8 espansione)

---

§ 1. DATABASE DECISION MATRIX

| Database           | Tipo                | Modello Dati             | Scaling                    | ACID             | Query Language  | Latency  | Caso d'Uso Ideale                       |
| ------------------ | ------------------- | ------------------------ | -------------------------- | ---------------- | --------------- | -------- | --------------------------------------- |
| PostgreSQL         | Relational OLTP     | Tables/Rows              | Vertical + Read Replicas   | ✅ Full          | SQL             | 1–10ms   | SaaS multi-tenant con billing e audit   |
| MySQL              | Relational OLTP     | Tables/Rows              | Vertical + Read Replicas   | ✅ Full          | SQL             | 1–10ms   | CMS ad alto traffico con schema stabile |
| MariaDB            | Relational OLTP     | Tables/Rows              | Vertical + Galera Cluster  | ✅ Full          | SQL             | 1–10ms   | Fintech mid-scale                       |
| MongoDB            | Document            | JSON Documents           | Horizontal (Sharding)      | ⚠️ Per-document  | MongoDB Query   | 1–10ms   | Cataloghi prodotti schema flessibile    |
| Redis              | Key-Value/In-Memory | Keys → Values            | Cluster + Replication      | ❌ Volatile      | Redis Commands  | <1ms     | Session store real-time                 |
| Memcached          | Key-Value/In-Memory | Keys → Values            | Distributed                | ❌ Volatile      | Memcached Proto | <1ms     | Cache HTTP ad alta concorrenza          |
| Elasticsearch      | Search Engine       | JSON Documents           | Cluster                    | ❌               | Query DSL       | 10–100ms | Log analytics e full-text search        |
| OpenSearch         | Search Engine       | JSON Documents           | Cluster                    | ❌               | Query DSL       | 10–100ms | Observability stack self-hosted         |
| DynamoDB           | Key-Value/Document  | Items                    | Auto horizontal            | ⚠️ Per-item      | PartiQL         | 1–10ms   | Serverless backend throughput variabile |
| Cassandra          | Wide-Column         | Rows/Columns             | Linear horizontal          | ❌ Eventual      | CQL             | 1–10ms   | IoT telemetry write-heavy               |
| ScyllaDB           | Wide-Column         | Rows/Columns             | Linear horizontal          | ❌ Eventual      | CQL             | 1–10ms   | Gaming analytics ultra-low latency      |
| Neo4j              | Graph               | Nodes/Edges              | Cluster                    | ✅ Full          | Cypher          | 1–10ms   | Recommendation engine                   |
| ArangoDB           | Multi-model         | Graph + Document + KV    | Cluster                    | ⚠️ Partial       | AQL             | 1–10ms   | Knowledge graph aziendale               |
| ClickHouse         | Columnar OLAP       | Tables/Columns           | Cluster                    | ❌               | SQL             | 100ms–1s | BI analytics su big data                |
| BigQuery           | Columnar OLAP       | Tables/Columns           | Serverless                 | ❌               | SQL             | 100ms–2s | Data warehouse cloud-native             |
| Snowflake          | Columnar OLAP       | Tables/Columns           | Elastic compute            | ❌               | SQL             | 100ms–2s | Enterprise analytics multi-cloud        |
| TimescaleDB        | Time-series         | Tables/Hypertables       | Vertical + Partitioning    | ✅ Full          | SQL             | 1–20ms   | Metric storage per monitoring           |
| InfluxDB           | Time-series         | Measurements/Tags        | Cluster                    | ❌               | Flux            | 1–20ms   | Telemetria industriale                  |
| SQLite             | Embedded Relational | Tables/Rows              | ❌ None                    | ✅ Full          | SQL             | <1ms     | Mobile apps offline-first               |
| CockroachDB        | Distributed SQL     | Tables/Rows              | Automatic horizontal       | ✅ Full          | SQL             | 5–20ms   | SaaS globale multi-region               |
| YugabyteDB         | Distributed SQL     | Tables/Rows              | Automatic horizontal       | ✅ Full          | SQL             | 5–20ms   | Fintech multi-region con SLA elevato    |
| Firebase Firestore | Document            | Collections/Documents    | Auto horizontal            | ⚠️ Per-document  | Firestore Query | 5–20ms   | Mobile apps real-time                   |
| Supabase           | Postgres + BaaS     | Tables/Rows              | Vertical + Read Replicas   | ✅ Full          | SQL + REST      | 1–10ms   | Startup MVP con realtime                |

---

§ 2. USE CASE → DATABASE MAPPING

| #  | Scenario                             | Database Primario  | Database Secondario | Motivazione                                 |
| -- | ------------------------------------ | ------------------ | ------------------- | ------------------------------------------- |
| 1  | E-commerce con pagamenti e inventory | PostgreSQL         | Redis               | ACID per transazioni, cache per performance |
| 2  | Marketplace multi-vendor             | CockroachDB        | Redis               | Consistenza globale e latenza multi-region  |
| 3  | Real-time chat                       | Redis Streams      | PostgreSQL          | Latenza <1ms e persistence affidabile       |
| 4  | Social network                       | Neo4j              | PostgreSQL          | Graph traversal + dati relazionali          |
| 5  | Search engine interno                | Elasticsearch      | PostgreSQL          | Indici full-text + source of truth          |
| 6  | IoT telemetry                        | Cassandra          | Kafka               | Write-heavy e ingestion streaming           |
| 7  | Analytics dashboard                  | ClickHouse         | PostgreSQL          | OLAP separato da OLTP                       |
| 8  | Mobile app backend                   | Firestore          | BigQuery            | Realtime + analytics                        |
| 9  | SaaS multi-tenant                    | PostgreSQL         | Redis               | Schema isolation + caching                  |
| 10 | Event-driven architecture            | PostgreSQL         | Kafka               | Event sourcing + persistence                |
| 11 | Fraud detection                      | Neo4j              | ClickHouse          | Graph analysis + analytics                  |
| 12 | Log management                       | OpenSearch         | S3                  | Full-text logs + storage                    |
| 13 | Recommendation engine                | ArangoDB           | Redis               | Graph + fast lookup                         |
| 14 | Fintech ledger                       | PostgreSQL         | CockroachDB         | Strong consistency + global replication     |
| 15 | CMS headless                         | MongoDB            | Elasticsearch       | Schema flessibile + search                  |
| 16 | Fintech payments                     | PostgreSQL         | Redis               | Transazioni ACID + cache                    |
| 17 | Gaming leaderboard                   | Redis Sorted Sets  | PostgreSQL          | Ranking real-time                           |
| 18 | Event-driven microservices           | PostgreSQL         | Kafka               | Persistenza + streaming                     |
| 19 | Real-time analytics                  | ClickHouse         | Kafka               | OLAP streaming                              |
| 20 | Mobile offline-first                 | SQLite             | PostgreSQL sync     | Local-first architecture                    |

---

§ 3. SCALING PATTERNS

§ 3.1 PATTERN MATRIX

| Pattern                  | Quando Usare            | Database Compatibili          | Complessità | Latency Impact |
| ------------------------ | ----------------------- | ----------------------------- | ----------- | -------------- |
| Read Replicas            | Read >80%               | PostgreSQL, MySQL, MongoDB    | Low         | +1–5ms         |
| Write Replicas           | Write-heavy distributed | Cassandra, DynamoDB           | Medium      | +2–10ms        |
| Connection Pooling       | >5k connessioni         | PostgreSQL, MySQL             | Low         | -10–30%        |
| Vertical Scaling         | Dataset <2TB            | Tutti                         | Low         | 0ms            |
| Partitioning             | Tabelle >200GB          | PostgreSQL, MySQL, ClickHouse | Medium      | ±5ms           |
| Sharding                 | Dataset >5TB            | MongoDB, Cassandra            | High        | +5–20ms        |
| CQRS                     | Read/write divergenti   | PostgreSQL + Elasticsearch    | High        | Eventual       |
| Event Sourcing           | Audit completo          | PostgreSQL + Kafka            | High        | Eventual       |
| Multi-region replication | SLA globali             | CockroachDB, YugabyteDB       | High        | +10–40ms       |

§ 3.2 SCALING DECISION TREE

START
│
├─ Bottleneck READ?
│   ├─ Add Read Replicas
│   └─ Add Redis Cache
│
├─ Bottleneck WRITE?
│   ├─ Partitioning
│   └─ Sharding
│
├─ High latency?
│   ├─ Index optimization
│   ├─ Query rewrite
│   └─ Separate OLAP
│
├─ Connection saturation?
│   └─ PgBouncer / ProxySQL
│
└─ Global users?
    ├─ Multi-region DB
    └─ Edge caching

---

§ 4. MANAGED SERVICES COST COMPARISON

§ 4.1 RELATIONAL DATABASES

| Service            | Free Tier | Starter   | Production   | Enterprise     |
| ------------------ | --------- | --------- | ------------ | -------------- |
| AWS RDS PostgreSQL | ❌        | $15–40/mo | $120–600/mo  | $600–8000/mo   |
| Aurora PostgreSQL  | ❌        | $30–80/mo | $200–1200/mo | $1200–15000/mo |
| Supabase           | ✅ 500MB  | $25/mo    | $75–200/mo   | $500–5000/mo   |
| Neon               | ✅ 3GB    | $19/mo    | $69–300/mo   | $1000+/mo      |
| PlanetScale        | ✅ 5GB    | $29/mo    | $99–500/mo   | $1000+/mo      |

§ 4.2 NOSQL & SEARCH

| Service             | Free Tier     | Starter       | Production   | Notes           |
| ------------------- | ------------- | ------------- | ------------ | --------------- |
| MongoDB Atlas       | ✅ 512MB      | $9/mo         | $57–500/mo   | Auto-scaling    |
| DynamoDB            | ✅ 25GB       | $1–5/mo       | $50–2000/mo  | Pay-per-use     |
| Redis Cloud         | ✅ 30MB       | $5/mo         | $100–1500/mo | Low latency     |
| Upstash Redis       | ✅ 10k ops/d  | $0.2/100k ops | $10–500/mo   | Serverless      |
| Elasticsearch Cloud | ❌            | $16/mo        | $95–2000/mo  | Search engine   |
| OpenSearch AWS      | ❌            | $20/mo        | $100–3000/mo | OSS alternative |

§ 4.3 MONTHLY COST SCENARIOS

| App Type    | Users    | Architecture                          | Estimated Cost   |
| ----------- | -------- | ------------------------------------- | ---------------- |
| MVP         | <1k      | Supabase + Redis                      | $0–50/mo         |
| Small SaaS  | 1k–10k   | RDS + Redis                           | $100–300/mo      |
| Medium SaaS | 10k–100k | Aurora + Redis + Elasticsearch        | $500–3000/mo     |
| Large SaaS  | 100k–1M  | Aurora Multi-AZ + DynamoDB            | $3000–20000/mo   |
| Enterprise  | >1M      | Multi-region CockroachDB + ClickHouse | $20000–100000/mo |

---

§ 5. ARCHITECTURE COMPARISON MATRIX

| Requirement           | Relational | Document | Graph | Wide-Column | Columnar |
| --------------------- | ---------- | -------- | ----- | ----------- | -------- |
| Strong consistency    | ⭐⭐⭐⭐⭐ | ⭐⭐     | ⭐⭐⭐⭐| ⭐⭐        | ⭐       |
| Flexible schema       | ⭐⭐       | ⭐⭐⭐⭐⭐| ⭐⭐⭐ | ⭐⭐        | ⭐       |
| Relationship modeling | ⭐⭐⭐⭐⭐ | ⭐⭐     | ⭐⭐⭐⭐⭐| ⭐         | ⭐       |
| Write throughput      | ⭐⭐⭐     | ⭐⭐⭐⭐ | ⭐⭐  | ⭐⭐⭐⭐⭐   | ⭐⭐     |
| Analytics             | ⭐⭐       | ⭐       | ⭐    | ⭐⭐        | ⭐⭐⭐⭐⭐|
| Horizontal scaling    | ⭐⭐       | ⭐⭐⭐⭐ | ⭐⭐  | ⭐⭐⭐⭐⭐   | ⭐⭐⭐⭐ |

---

§ 6. PERFORMANCE CHARACTERISTICS

| Database      | Max Throughput | Write Perf | Read Perf  | Consistency | Availability | Durability             |
| ------------- | -------------- | ---------- | ---------- | ----------- | ------------ | ---------------------- |
| PostgreSQL    | 10K-50K ops/s  | 🟢 High    | 🟢 High    | Strong      | 99.95%       | WAL + Replication      |
| MySQL         | 20K-100K ops/s | 🟢 High    | 🟢 High    | Strong      | 99.95%       | Binlog + Replication   |
| MongoDB       | 50K-200K ops/s | 🟢 High    | 🟢 High    | Tunable     | 99.99%       | Replica Set            |
| DynamoDB      | 1M+ ops/s      | 🟢 V.High  | 🟢 V.High  | Tunable     | 99.999%      | Multi-AZ               |
| Cassandra     | 500K+ ops/s    | 🟢 V.High  | 🟡 Medium  | Eventual    | 99.99%       | Multi-node replication |
| Redis         | 1M+ ops/s      | 🟢 Extreme | 🟢 Extreme | Weak        | 99.9%        | Snapshot/AOF           |
| ClickHouse    | 100K+ ops/s    | 🟡 Medium  | 🟢 V.High  | Weak        | 99.9%        | Replication            |
| Neo4j         | 5K-20K ops/s   | 🟡 Medium  | 🟡 Medium  | Strong      | 99.9%        | Cluster replication    |
| CockroachDB   | 50K-150K ops/s | 🟢 High    | 🟢 High    | Strong      | 99.99%       | Raft consensus         |
| Elasticsearch | 100K+ ops/s    | 🟡 Medium  | 🟢 High    | Eventual    | 99.9%        | Shards + replicas      |

---

§ 7. ARCHITECTURE PATTERNS

| Architecture       | Database Core | Supporting DB | Messaging  | Cache   | Cost/mo     | Scale Ceiling |
| ------------------ | ------------- | ------------- | ---------- | ------- | ----------- | ------------- |
| Monolith SaaS      | PostgreSQL    | -             | -          | Redis   | $50-300     | 100K users    |
| Microservices      | PostgreSQL    | MongoDB       | Kafka      | Redis   | $300-2000   | 1M users      |
| Serverless         | DynamoDB      | S3            | SNS/SQS    | Upstash | $20-1000    | 10M users     |
| Event-driven       | PostgreSQL    | ClickHouse    | Kafka      | Redis   | $500-5000   | 5M users      |
| Real-time platform | Redis         | PostgreSQL    | WebSockets | Redis   | $200-3000   | 2M users      |
| Global SaaS        | CockroachDB   | Redis         | Kafka      | Redis   | $800-8000   | 10M+ users    |

---

§ 8. DATABASE SELECTION SCORECARD

| Requirement            | PostgreSQL | MongoDB | DynamoDB | Redis | ClickHouse | Neo4j | CockroachDB |
| ---------------------- | ---------- | ------- | -------- | ----- | ---------- | ----- | ----------- |
| Strong Consistency     | 10/10      | 7/10    | 8/10     | 3/10  | 2/10       | 9/10  | 10/10       |
| Scalability            | 7/10       | 9/10    | 10/10    | 8/10  | 9/10       | 6/10  | 9/10        |
| Cost Efficiency        | 8/10       | 6/10    | 5/10     | 7/10  | 7/10       | 5/10  | 6/10        |
| Developer Productivity | 9/10       | 8/10    | 6/10     | 7/10  | 6/10       | 6/10  | 7/10        |
| Global Distribution    | 5/10       | 7/10    | 9/10     | 4/10  | 6/10       | 4/10  | 10/10       |
| Real-time Performance  | 7/10       | 8/10    | 9/10     | 10/10 | 5/10       | 6/10  | 7/10        |

---

§ 9. SELECTION FORMULA

Score(Database) =
  (Consistency * 0.25) +
  (Scalability * 0.25) +
  (CostEfficiency * 0.20) +
  (DeveloperProductivity * 0.15) +
  (GlobalDistribution * 0.10) +
  (RealTimePerformance * 0.05)

Scelta finale = Database con score più alto per il caso d'uso specifico

---

§ 10. DATABASE SELECTION CHECKLIST

□ Modello dati richiesto (relazionale, document, graph, time-series)
□ Volume iniziale dati (GB/TB)
□ Crescita annua prevista (%)
□ Throughput richiesto (RPS/WPS)
□ Consistenza richiesta (strong vs eventual)
□ Latency target (<5ms, <50ms, <500ms)
□ Budget mensile (USD)
□ Competenza team (SQL/NoSQL)
□ Multi-region requirement (Sì/No)
□ Compliance (GDPR, PCI-DSS, SOC2, HIPAA)
□ Backup RPO/RTO richiesto
□ Vendor lock-in accettabile (Sì/No)
□ Serverless vs serverful
□ Query complexity (CRUD vs analytics)
□ Evoluzione schema prevista

---

§ 11. QUICK REFERENCE BY WORKLOAD

§ OLTP (TRANSACTIONAL)
- **Best**: PostgreSQL, MySQL, CockroachDB
- **Avoid**: ClickHouse, BigQuery

§ OLAP (ANALYTICS)
- **Best**: ClickHouse, BigQuery, Snowflake
- **Avoid**: Redis, MongoDB

§ REAL-TIME
- **Best**: Redis, DynamoDB, Cassandra
- **Avoid**: BigQuery, Snowflake

§ GRAPH TRAVERSAL
- **Best**: Neo4j, ArangoDB
- **Avoid**: ClickHouse, DynamoDB

§ TIME-SERIES
- **Best**: TimescaleDB, InfluxDB
- **Avoid**: MongoDB, Neo4j

§ FULL-TEXT SEARCH
- **Best**: Elasticsearch, OpenSearch
- **Avoid**: DynamoDB, Cassandra

---

§ 12. MIGRATION COMPLEXITY MATRIX

| From → To     | PostgreSQL | MongoDB | DynamoDB | Redis | Elasticsearch |
| ------------- | ---------- | ------- | -------- | ----- | ------------- |
| PostgreSQL    | -          | Medium  | High     | Low   | Medium        |
| MongoDB       | Medium     | -       | Medium   | Low   | Low           |
| DynamoDB      | High       | Medium  | -        | Low   | Medium        |
| MySQL         | Low        | Medium  | High     | Low   | Medium        |
| Redis         | N/A        | N/A     | N/A      | -     | N/A           |


---

§ 13. CONNECTION POOLING

§ 13.1 CONNECTION POOLER COMPARISON

| Tool | Database | Max Conn | Overhead | Pool Mode | Serverless | Best For |
|------|----------|----------|----------|-----------|------------|----------|
| PgBouncer | PostgreSQL | 10,000+ | <1ms | Transaction | ✅ | High-traffic apps |
| Prisma Pool | PostgreSQL/MySQL | 100 default | 2-5ms | Connection | ✅ | Serverless Node.js |
| pgpool-II | PostgreSQL | 5,000+ | 5-10ms | Session | ❌ | Load balancing |
| ProxySQL | MySQL | 10,000+ | <1ms | Transaction | ⚠️ | MySQL clusters |
| RDS Proxy | PostgreSQL/MySQL | 10,000+ | 1-3ms | Pin/Transaction | ✅ | AWS Lambda |
| Supabase Pooler | PostgreSQL | 1,000+ | 2-5ms | Transaction | ✅ | Supabase projects |
| Neon Pooler | PostgreSQL | 10,000+ | 1-2ms | Transaction | ✅ | Serverless Postgres |
| PlanetScale | MySQL | Unlimited | 1-3ms | Connection | ✅ | Vitess-based |

§ 13.2 PGBOUNCER PRODUCTION CONFIG

ini
; /etc/pgbouncer/pgbouncer.ini
[databases]
myapp = host=db.example.com port=5432 dbname=myapp_production

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt

; Pool settings
pool_mode = transaction
max_client_conn = 10000
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
reserve_pool_timeout = 3

; Connection limits
max_db_connections = 100
max_user_connections = 100

; Timeouts
server_idle_timeout = 600
server_lifetime = 3600
server_connect_timeout = 15
query_timeout = 300
client_idle_timeout = 0

; Logging
log_connections = 1
log_disconnections = 1
log_pooler_errors = 1
stats_period = 60

§ 13.3 PRISMA CONNECTION POOLING

typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

const prismaClientSingleton = () => {
  return new PrismaClient({
    datasources: {
      db: {
        url: `${process.env.DATABASE_URL}?connection_limit=10&pool_timeout=20`,
      },
    },
    log: process.env.NODE_ENV === 'development' 
      ? ['query', 'error', 'warn'] 
      : ['error'],
  });
};

export const prisma = globalForPrisma.prisma ?? prismaClientSingleton();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Graceful shutdown
const shutdown = async () => {
  await prisma.$disconnect();
  process.exit(0);
};

process.on('SIGINT', shutdown);
process.on('SIGTERM', shutdown);

§ 13.4 POOL SIZING FORMULA

Optimal Pool Size = (Number of CPU Cores * 2) + Effective Spindle Count

For SSDs (no spindle): Pool Size = CPU Cores * 2 + 1
Example (8 core, SSD): Pool Size = 8 * 2 + 1 = 17 connections
For Serverless: Pool Size = Max Concurrent Functions / 10


---

§ 14. DATABASE MIGRATIONS

§ 14.1 MIGRATION TOOLS COMPARISON

| Tool | Language | Rollback | Version Control | CI/CD | Type Safety | Best For |
|------|----------|----------|-----------------|-------|-------------|----------|
| Prisma Migrate | TypeScript | ✅ Auto | ✅ Git-friendly | ✅ | ✅ Full | Full-stack TS |
| Drizzle Kit | TypeScript | ✅ Manual | ✅ Git-friendly | ✅ | ✅ Full | Lightweight ORM |
| Flyway | Java/SQL | ✅ Manual | ✅ Numbered files | ✅ | ❌ | Enterprise Java |
| Liquibase | XML/YAML/SQL | ✅ Auto | ✅ Changelog | ✅ | ❌ | Complex schemas |
| golang-migrate | Go/SQL | ✅ Manual | ✅ Numbered files | ✅ | ❌ | Go backends |
| Knex.js | JavaScript | ✅ Manual | ✅ Timestamps | ✅ | ⚠️ Partial | Node.js legacy |
| TypeORM | TypeScript | ✅ Auto | ✅ Timestamps | ✅ | ⚠️ Partial | NestJS projects |
| Atlas | HCL/SQL | ✅ Auto | ✅ Declarative | ✅ | ❌ | Multi-DB |

§ 14.2 PRISMA MIGRATION WORKFLOW

bash
# Development: Create and apply migration
npx prisma migrate dev --name add_user_profile

# Preview migration SQL without applying
npx prisma migrate dev --create-only

# Production: Apply pending migrations
npx prisma migrate deploy

# Reset database (DEVELOPMENT ONLY)
npx prisma migrate reset

# Check migration status
npx prisma migrate status

§ 14.3 DRIZZLE MIGRATION WORKFLOW

typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';

export default {
  schema: './src/db/schema.ts',
  out: './drizzle/migrations',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  },
} satisfies Config;

---

§ 15. INDEX OPTIMIZATION

§ 15.1 INDEX TYPES REFERENCE

| Index Type | PostgreSQL | MySQL | Use Case | Complexity | When to Use |
|------------|------------|-------|----------|------------|-------------|
| B-tree | ✅ Default | ✅ Default | Equality, Range, Sorting | O(log n) | Most queries |
| Hash | ✅ | ✅ | Equality only | O(1) | Exact match lookups |
| GiST | ✅ | ❌ | Geometric, Full-text | O(log n) | PostGIS, fuzzy search |
| GIN | ✅ | ❌ | Arrays, JSONB, Full-text | O(log n) | JSONB queries |
| BRIN | ✅ | ❌ | Large sorted tables | O(1) | Time-series, logs |
| Partial | ✅ | ❌ | Filtered subset | O(log n) | Status = 'active' |
| Composite | ✅ | ✅ | Multi-column | O(log n) | Combined filters |
| Covering | ✅ | ✅ | Index-only scans | O(log n) | SELECT specific cols |

§ 15.2 INDEX STRATEGY PATTERNS

sql
-- COMPOSITE INDEX (column order matters!)
CREATE INDEX idx_orders_status_created 
ON orders (status, created_at DESC);

-- PARTIAL INDEX (filtered subset)
CREATE INDEX idx_active_users 
ON users (email) 
WHERE deleted_at IS NULL AND status = 'active';

-- COVERING INDEX (index-only scan)
CREATE INDEX idx_orders_covering 
ON orders (user_id, status) 
INCLUDE (total_amount, created_at);

-- GIN INDEX FOR JSONB
CREATE INDEX idx_products_metadata 
ON products USING GIN (metadata jsonb_path_ops);


---

§ 16. QUERY OPTIMIZATION

§ 16.1 QUERY ANTI-PATTERNS

| ❌ Anti-Pattern | ✅ Solution | Performance Impact |
|----------------|-------------|-------------------|
| SELECT * | SELECT specific columns | 10-50% faster |
| OFFSET pagination | Cursor-based pagination | 100x faster at high offsets |
| COUNT(*) on large tables | Approximate count | 1000x faster |
| N+1 queries | JOIN or batch loading | 10-100x faster |
| LIKE '%term%' | Full-text search | 10-100x faster |
| OR conditions | UNION ALL | 2-10x faster |
| NOT IN (subquery) | NOT EXISTS | 2-5x faster |

§ 16.2 OPTIMIZATION EXAMPLES

sql
-- ❌ OFFSET PAGINATION (slow at high pages)
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- ✅ CURSOR PAGINATION (consistent speed)
SELECT * FROM posts 
WHERE created_at < '2024-01-15T00:00:00Z'
ORDER BY created_at DESC LIMIT 20;

-- ❌ COUNT(*) ON LARGE TABLE
SELECT COUNT(*) FROM posts;

-- ✅ APPROXIMATE COUNT
SELECT reltuples::bigint AS estimate FROM pg_class WHERE relname = 'posts';

---

§ 17. MULTI-TENANCY PATTERNS

§ 17.1 STRATEGY COMPARISON

| Strategy | Isolation | Cost | Complexity | Scale | Best For |
|----------|-----------|------|------------|-------|----------|
| Shared Schema + tenant_id | 🔴 Low | 💰 Low | 🟢 Low | 🟢 Easy | B2C SaaS, startups |
| Schema per Tenant | 🟡 Medium | 💰💰 Medium | 🟡 Medium | 🟡 Medium | B2B SaaS mid-size |
| Database per Tenant | 🟢 High | 💰💰💰 High | 🔴 High | 🔴 Complex | Enterprise, regulated |
| Row-Level Security | 🟡 Medium | 💰 Low | 🟡 Medium | 🟢 Easy | PostgreSQL projects |

§ 17.2 ROW-LEVEL SECURITY IMPLEMENTATION

sql
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts FORCE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_policy ON posts
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant')::uuid)
  WITH CHECK (tenant_id = current_setting('app.current_tenant')::uuid);

---

§ 18. BACKUP & DISASTER RECOVERY

§ 18.1 BACKUP STRATEGY MATRIX

| Strategy | RPO | RTO | Complexity | Best For |
|----------|-----|-----|------------|----------|
| pg_dump (full) | 24h | 1-4h | 🟢 Low | Small DBs <10GB |
| WAL Archiving | Minutes | 30min-2h | 🟡 Medium | Production |
| Streaming Replication | Seconds | 5-30min | 🟡 Medium | HA setup |
| PITR (Point-in-Time) | Seconds | 15-60min | 🟡 Medium | Critical data |
| Cloud Snapshots | 1-24h | 30min-2h | 🟢 Low | Managed DBs |

---

§ 19. DATABASE MONITORING

§ 19.1 KEY METRICS TABLE

| Metric | Warning | Critical | Query/Tool |
|--------|---------|----------|------------|
| Connection count | >80% max | >95% max | pg_stat_activity |
| Active queries | >50 | >100 | pg_stat_activity |
| Long queries | >30s | >60s | pg_stat_activity |
| Dead tuples | >10% | >20% | pg_stat_user_tables |
| Cache hit ratio | <95% | <90% | pg_stat_database |
| Replication lag | >1min | >5min | pg_stat_replication |
| Disk usage | >80% | >90% | pg_database_size |

---

§ 20. DATABASE EXPANSION CHECKLIST

CONNECTION MANAGEMENT
□ Connection pooler deployed (PgBouncer/RDS Proxy)
□ Pool size tuned for workload
□ Connection timeout configured

MIGRATIONS
□ Migration tool selected and configured
□ Rollback strategy documented
□ CI/CD pipeline includes migrations

INDEX OPTIMIZATION  
□ Query patterns analyzed with EXPLAIN
□ Composite indexes for common filters
□ Unused indexes identified and dropped

QUERY PERFORMANCE
□ N+1 queries eliminated
□ Cursor pagination implemented
□ Slow query logging enabled

MULTI-TENANCY
□ Isolation strategy chosen
□ RLS policies implemented

BACKUP & DR
□ Backup schedule configured
□ RPO/RTO documented and tested

MONITORING
□ Connection metrics tracked
□ Query performance metrics
□ Disk usage alerts


---

§ 13. CONNECTION POOLING

§ 13.1 CONNECTION POOLER COMPARISON

| Tool | Database | Max Conn | Overhead | Pool Mode | Serverless | Best For |
|------|----------|----------|----------|-----------|------------|----------|
| PgBouncer | PostgreSQL | 10,000+ | <1ms | Transaction | ✅ | High-traffic apps |
| Prisma Pool | PostgreSQL/MySQL | 100 default | 2-5ms | Connection | ✅ | Serverless Node.js |
| pgpool-II | PostgreSQL | 5,000+ | 5-10ms | Session | ❌ | Load balancing |
| ProxySQL | MySQL | 10,000+ | <1ms | Transaction | ⚠️ | MySQL clusters |
| RDS Proxy | PostgreSQL/MySQL | 10,000+ | 1-3ms | Pin/Transaction | ✅ | AWS Lambda |
| Supabase Pooler | PostgreSQL | 1,000+ | 2-5ms | Transaction | ✅ | Supabase projects |
| Neon Pooler | PostgreSQL | 10,000+ | 1-2ms | Transaction | ✅ | Serverless Postgres |
| PlanetScale | MySQL | Unlimited | 1-3ms | Connection | ✅ | Vitess-based |

§ 13.2 PGBOUNCER PRODUCTION CONFIG

ini
; /etc/pgbouncer/pgbouncer.ini
[databases]
myapp = host=db.example.com port=5432 dbname=myapp_production

[pgbouncer]
listen_addr = 0.0.0.0
listen_port = 6432
auth_type = scram-sha-256
auth_file = /etc/pgbouncer/userlist.txt

; Pool settings
pool_mode = transaction
max_client_conn = 10000
default_pool_size = 25
min_pool_size = 5
reserve_pool_size = 5
reserve_pool_timeout = 3

; Connection limits
max_db_connections = 100
max_user_connections = 100

; Timeouts
server_idle_timeout = 600
server_lifetime = 3600
server_connect_timeout = 15
query_timeout = 300
client_idle_timeout = 0

; Logging
log_connections = 1
log_disconnections = 1
log_pooler_errors = 1
stats_period = 60

§ 13.3 PRISMA CONNECTION POOLING

typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient };

const prismaClientSingleton = () => {
  return new PrismaClient({
    datasources: {
      db: {
        url: `${process.env.DATABASE_URL}?connection_limit=10&pool_timeout=20`,
      },
    },
    log: process.env.NODE_ENV === 'development' 
      ? ['query', 'error', 'warn'] 
      : ['error'],
  });
};

export const prisma = globalForPrisma.prisma ?? prismaClientSingleton();

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Graceful shutdown
const shutdown = async () => {
  await prisma.$disconnect();
  process.exit(0);
};

process.on('SIGINT', shutdown);
process.on('SIGTERM', shutdown);

§ 13.4 POOL SIZING FORMULA

Optimal Pool Size = (Number of CPU Cores * 2) + Effective Spindle Count

For SSDs (no spindle): 
  Pool Size = CPU Cores * 2 + 1

Example (8 core, SSD):
  Pool Size = 8 * 2 + 1 = 17 connections

For Serverless:
  Pool Size = Max Concurrent Functions / 10


---

§ 14. DATABASE MIGRATIONS

§ 14.1 MIGRATION TOOLS COMPARISON

| Tool | Language | Rollback | Version Control | CI/CD | Type Safety | Best For |
|------|----------|----------|-----------------|-------|-------------|----------|
| Prisma Migrate | TypeScript | ✅ Auto | ✅ Git-friendly | ✅ | ✅ Full | Full-stack TS |
| Drizzle Kit | TypeScript | ✅ Manual | ✅ Git-friendly | ✅ | ✅ Full | Lightweight ORM |
| Flyway | Java/SQL | ✅ Manual | ✅ Numbered files | ✅ | ❌ | Enterprise Java |
| Liquibase | XML/YAML/SQL | ✅ Auto | ✅ Changelog | ✅ | ❌ | Complex schemas |
| golang-migrate | Go/SQL | ✅ Manual | ✅ Numbered files | ✅ | ❌ | Go backends |
| TypeORM | TypeScript | ✅ Auto | ✅ Timestamps | ✅ | ⚠️ Partial | NestJS projects |
| Atlas | HCL/SQL | ✅ Auto | ✅ Declarative | ✅ | ❌ | Multi-DB |

§ 14.2 PRISMA MIGRATION WORKFLOW

bash
# Development: Create and apply migration
npx prisma migrate dev --name add_user_profile

# Preview migration SQL without applying
npx prisma migrate dev --create-only

# Production: Apply pending migrations
npx prisma migrate deploy

# Reset database (DEVELOPMENT ONLY)
npx prisma migrate reset

# Check migration status
npx prisma migrate status

# Resolve failed migration
npx prisma migrate resolve --applied "20240115120000_failed_migration"

§ 14.3 PRISMA SCHEMA EXAMPLE

typescript
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  directUrl = env("DIRECT_URL") // For migrations (bypasses pooler)
}

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  role      Role     @default(USER)
  profile   Profile?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([email])
  @@index([createdAt(sort: Desc)])
  @@map("users")
}

enum Role {
  USER
  ADMIN
  MODERATOR
}

---

§ 15. INDEX OPTIMIZATION

§ 15.1 INDEX TYPES REFERENCE

| Index Type | PostgreSQL | MySQL | Use Case | Complexity | When to Use |
|------------|------------|-------|----------|------------|-------------|
| B-tree | ✅ Default | ✅ Default | Equality, Range, Sorting | O(log n) | Most queries |
| Hash | ✅ | ✅ | Equality only | O(1) | Exact match lookups |
| GiST | ✅ | ❌ | Geometric, Full-text | O(log n) | PostGIS, fuzzy search |
| GIN | ✅ | ❌ | Arrays, JSONB, Full-text | O(log n) | JSONB queries |
| BRIN | ✅ | ❌ | Large sorted tables | O(1) | Time-series, logs |
| Partial | ✅ | ❌ | Filtered subset | O(log n) | Status = 'active' |
| Composite | ✅ | ✅ | Multi-column | O(log n) | Combined filters |
| Covering | ✅ | ✅ | Index-only scans | O(log n) | SELECT specific cols |

§ 15.2 INDEX STRATEGY PATTERNS

sql
-- 1. COMPOSITE INDEX (column order matters!)
CREATE INDEX idx_orders_status_created 
ON orders (status, created_at DESC);

-- 2. PARTIAL INDEX (filtered subset)
CREATE INDEX idx_active_users 
ON users (email) 
WHERE deleted_at IS NULL AND status = 'active';

-- 3. COVERING INDEX (index-only scan)
CREATE INDEX idx_orders_covering 
ON orders (user_id, status) 
INCLUDE (total_amount, created_at);

-- 4. GIN INDEX FOR JSONB
CREATE INDEX idx_products_metadata 
ON products USING GIN (metadata jsonb_path_ops);

-- 5. FULL-TEXT SEARCH INDEX
CREATE INDEX idx_posts_search 
ON posts USING GIN (
  to_tsvector('english', title || ' ' || COALESCE(content, ''))
);

-- 6. BRIN INDEX (for large time-series)
CREATE INDEX idx_events_created_brin 
ON events USING BRIN (created_at) 
WITH (pages_per_range = 128);

§ 15.3 INDEX ANALYSIS QUERIES

sql
-- Find missing indexes (high seq scan tables)
SELECT 
  schemaname || '.' || relname AS table_name,
  seq_scan,
  idx_scan,
  CASE WHEN seq_scan > 0 
    THEN ROUND(100.0 * idx_scan / (seq_scan + idx_scan), 2) 
    ELSE 100 
  END AS index_usage_pct
FROM pg_stat_user_tables
WHERE seq_scan > 1000
ORDER BY seq_tup_read DESC
LIMIT 20;

-- Find unused indexes
SELECT 
  schemaname || '.' || indexrelname AS index_name,
  pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
  idx_scan AS times_used
FROM pg_stat_user_indexes
WHERE idx_scan < 50
  AND indexrelname NOT LIKE '%_pkey'
ORDER BY pg_relation_size(indexrelid) DESC
LIMIT 20;


---

§ 16. QUERY OPTIMIZATION

§ 16.1 QUERY ANTI-PATTERNS

| ❌ Anti-Pattern | ✅ Solution | Performance Impact |
|----------------|-------------|-------------------|
| SELECT * | SELECT specific columns | 10-50% faster |
| OFFSET pagination | Cursor-based pagination | 100x faster at high offsets |
| COUNT(*) on large tables | Approximate count | 1000x faster |
| N+1 queries | JOIN or batch loading | 10-100x faster |
| LIKE '%term%' | Full-text search | 10-100x faster |
| OR conditions | UNION ALL | 2-10x faster |
| NOT IN (subquery) | NOT EXISTS | 2-5x faster |
| Functions on indexed columns | Expression index | 10x faster |
| Missing indexes | Add appropriate indexes | 100-1000x faster |

§ 16.2 OPTIMIZATION EXAMPLES

sql
-- ❌ N+1 PROBLEM
SELECT * FROM users;
-- Then for EACH user: SELECT * FROM posts WHERE user_id = ?;

-- ✅ SOLUTION: JOIN
SELECT u.*, p.*
FROM users u
LEFT JOIN posts p ON p.user_id = u.id;

-- ❌ OFFSET PAGINATION (slow at high pages)
SELECT * FROM posts ORDER BY created_at DESC LIMIT 20 OFFSET 10000;

-- ✅ CURSOR PAGINATION (consistent speed)
SELECT * FROM posts 
WHERE created_at < '2024-01-15T00:00:00Z'
ORDER BY created_at DESC LIMIT 20;

-- ❌ COUNT(*) ON LARGE TABLE
SELECT COUNT(*) FROM posts;

-- ✅ APPROXIMATE COUNT
SELECT reltuples::bigint AS estimate 
FROM pg_class WHERE relname = 'posts';

---

§ 17. MULTI-TENANCY PATTERNS

§ 17.1 STRATEGY COMPARISON

| Strategy | Isolation | Cost | Complexity | Scale | Compliance | Best For |
|----------|-----------|------|------------|-------|------------|----------|
| Shared Schema + tenant_id | 🔴 Low | 💰 Low | 🟢 Low | 🟢 Easy | ⚠️ Medium | B2C SaaS, startups |
| Schema per Tenant | 🟡 Medium | 💰💰 Medium | 🟡 Medium | 🟡 Medium | ✅ Good | B2B SaaS mid-size |
| Database per Tenant | 🟢 High | 💰💰💰 High | 🔴 High | 🔴 Complex | ✅ Excellent | Enterprise, regulated |
| Row-Level Security | 🟡 Medium | 💰 Low | 🟡 Medium | 🟢 Easy | ✅ Good | PostgreSQL projects |

§ 17.2 ROW-LEVEL SECURITY IMPLEMENTATION

sql
-- Enable RLS on tables
ALTER TABLE posts ENABLE ROW LEVEL SECURITY;
ALTER TABLE posts FORCE ROW LEVEL SECURITY;

-- Create policy for tenant isolation
CREATE POLICY tenant_isolation_policy ON posts
  FOR ALL
  USING (tenant_id = current_setting('app.current_tenant')::uuid)
  WITH CHECK (tenant_id = current_setting('app.current_tenant')::uuid);

-- Bypass policy for admin users
CREATE POLICY admin_bypass ON posts
  FOR ALL
  USING (current_setting('app.is_admin', true)::boolean = true);

typescript
// Prisma extension for automatic tenant filtering
export const prismaWithTenant = (tenantId: string) => {
  return prisma.$extends({
    query: {
      $allModels: {
        async $allOperations({ model, operation, args, query }) {
          if (['findMany', 'findFirst', 'findUnique', 'count'].includes(operation)) {
            args.where = { ...args.where, tenantId };
          }
          if (['create', 'createMany'].includes(operation)) {
            if (Array.isArray(args.data)) {
              args.data = args.data.map(d => ({ ...d, tenantId }));
            } else {
              args.data = { ...args.data, tenantId };
            }
          }
          return query(args);
        },
      },
    },
  });
};

---

§ 18. BACKUP & DISASTER RECOVERY

§ 18.1 BACKUP STRATEGY MATRIX

| Strategy | RPO | RTO | Storage | Complexity | Best For |
|----------|-----|-----|---------|------------|----------|
| pg_dump (full) | 24h | 1-4h | Low | 🟢 Low | Small DBs <10GB |
| pg_dump + cron | 6-24h | 1-2h | Medium | 🟢 Low | Dev/staging |
| WAL Archiving | Minutes | 30min-2h | High | 🟡 Medium | Production |
| Streaming Replication | Seconds | 5-30min | High | 🟡 Medium | HA setup |
| PITR (Point-in-Time) | Seconds | 15-60min | High | 🟡 Medium | Critical data |
| pgBackRest | Minutes | 15-60min | Medium | 🟡 Medium | Large DBs |
| Cloud Snapshots | 1-24h | 30min-2h | Low | 🟢 Low | Managed DBs |

§ 18.2 AUTOMATED BACKUP SCRIPT

bash
#!/bin/bash
# backup-postgres.sh
set -euo pipefail

DB_HOST="${DB_HOST:-localhost}"
DB_NAME="${DB_NAME:-myapp}"
DB_USER="${DB_USER:-postgres}"
BACKUP_DIR="${BACKUP_DIR:-/backups}"
S3_BUCKET="${S3_BUCKET:-s3://my-backups}"
RETENTION_DAYS="${RETENTION_DAYS:-30}"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${TIMESTAMP}.sql.gz"

mkdir -p "${BACKUP_DIR}"

PGPASSWORD="${DB_PASSWORD}" pg_dump \
  -h "${DB_HOST}" -U "${DB_USER}" -d "${DB_NAME}" \
  -Fc -Z 9 --no-owner --no-acl \
  -f "${BACKUP_FILE}"

# Upload to S3
aws s3 cp "${BACKUP_FILE}" "${S3_BUCKET}/${DB_NAME}/"

# Cleanup old backups
find "${BACKUP_DIR}" -name "*.sql.gz" -mtime +${RETENTION_DAYS} -delete

echo "✅ Backup verified: ${BACKUP_FILE}"


---

§ 19. DATABASE MONITORING

§ 19.1 KEY METRICS TABLE

| Metric | Warning | Critical | Query/Tool |
|--------|---------|----------|------------|
| Connection count | >80% max | >95% max | pg_stat_activity |
| Active queries | >50 | >100 | pg_stat_activity |
| Long queries | >30s | >60s | pg_stat_activity |
| Dead tuples | >10% | >20% | pg_stat_user_tables |
| Cache hit ratio | <95% | <90% | pg_stat_database |
| Replication lag | >1min | >5min | pg_stat_replication |
| Disk usage | >80% | >90% | pg_database_size |
| Transaction rate | Baseline +50% | Baseline +100% | pg_stat_database |

§ 19.2 MONITORING QUERIES

sql
-- Current connections by state
SELECT state, COUNT(*) as count,
  MAX(EXTRACT(EPOCH FROM (now() - query_start))) as max_duration_sec
FROM pg_stat_activity
WHERE datname = current_database()
GROUP BY state;

-- Long running queries (>30 seconds)
SELECT pid, now() - query_start AS duration, state, LEFT(query, 100) AS query_preview
FROM pg_stat_activity
WHERE state != 'idle' AND query_start < now() - interval '30 seconds'
ORDER BY query_start;

-- Cache hit ratio
SELECT datname,
  ROUND(100.0 * blks_hit / NULLIF(blks_hit + blks_read, 0), 2) AS cache_hit_ratio
FROM pg_stat_database
WHERE datname = current_database();

-- Table bloat (dead tuples)
SELECT schemaname || '.' || relname AS table_name,
  n_live_tup, n_dead_tup,
  ROUND(100.0 * n_dead_tup / NULLIF(n_live_tup + n_dead_tup, 0), 2) AS dead_pct,
  last_vacuum, last_autovacuum
FROM pg_stat_user_tables
WHERE n_dead_tup > 1000
ORDER BY n_dead_tup DESC LIMIT 20;

-- Replication lag (on replica)
SELECT client_addr, state, sent_lsn, replay_lsn,
  pg_wal_lsn_diff(sent_lsn, replay_lsn) AS lag_bytes
FROM pg_stat_replication;

---

§ 20. DATABASE EXPANSION CHECKLIST

CONNECTION MANAGEMENT
□ Connection pooler deployed (PgBouncer/RDS Proxy)
□ Pool size tuned for workload
□ Connection timeout configured
□ Idle connection cleanup enabled
□ Health checks configured

MIGRATIONS
□ Migration tool selected and configured
□ directUrl for migrations (bypasses pooler)
□ Rollback strategy documented
□ CI/CD pipeline includes migrations
□ Migration dry-run before production

INDEX OPTIMIZATION  
□ Query patterns analyzed with EXPLAIN
□ Composite indexes for common filters
□ Partial indexes for filtered queries
□ Covering indexes for hot queries
□ Unused indexes identified and dropped
□ Index bloat monitored

QUERY PERFORMANCE
□ N+1 queries eliminated
□ Cursor pagination implemented
□ Full-text search for text queries
□ Slow query logging enabled
□ Query timeout configured

MULTI-TENANCY
□ Isolation strategy chosen
□ RLS policies implemented
□ Tenant context propagation
□ Cross-tenant access prevented

BACKUP & DR
□ Backup schedule configured
□ Backup verification automated
□ RPO/RTO documented and tested
□ Restore procedure documented
□ Cross-region backup (if required)

MONITORING
□ Connection metrics tracked
□ Query performance metrics
□ Disk usage alerts
□ Replication lag monitoring
□ Dead tuple alerting
