# CATALOGO DATABASE SELECTION GUIDE v1

> **Versione**: 1.0
> **Data**: 2026-01-27
> **Ambito**: Selezione database, scaling patterns, cost comparison, architecture patterns

---

## 1. DATABASE DECISION MATRIX

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

## 2. USE CASE → DATABASE MAPPING

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

## 3. SCALING PATTERNS

### 3.1 Pattern Matrix

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

### 3.2 Scaling Decision Tree

```
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
```

---

## 4. MANAGED SERVICES COST COMPARISON

### 4.1 Relational Databases

| Service            | Free Tier | Starter   | Production   | Enterprise     |
| ------------------ | --------- | --------- | ------------ | -------------- |
| AWS RDS PostgreSQL | ❌        | $15–40/mo | $120–600/mo  | $600–8000/mo   |
| Aurora PostgreSQL  | ❌        | $30–80/mo | $200–1200/mo | $1200–15000/mo |
| Supabase           | ✅ 500MB  | $25/mo    | $75–200/mo   | $500–5000/mo   |
| Neon               | ✅ 3GB    | $19/mo    | $69–300/mo   | $1000+/mo      |
| PlanetScale        | ✅ 5GB    | $29/mo    | $99–500/mo   | $1000+/mo      |

### 4.2 NoSQL & Search

| Service             | Free Tier     | Starter       | Production   | Notes           |
| ------------------- | ------------- | ------------- | ------------ | --------------- |
| MongoDB Atlas       | ✅ 512MB      | $9/mo         | $57–500/mo   | Auto-scaling    |
| DynamoDB            | ✅ 25GB       | $1–5/mo       | $50–2000/mo  | Pay-per-use     |
| Redis Cloud         | ✅ 30MB       | $5/mo         | $100–1500/mo | Low latency     |
| Upstash Redis       | ✅ 10k ops/d  | $0.2/100k ops | $10–500/mo   | Serverless      |
| Elasticsearch Cloud | ❌            | $16/mo        | $95–2000/mo  | Search engine   |
| OpenSearch AWS      | ❌            | $20/mo        | $100–3000/mo | OSS alternative |

### 4.3 Monthly Cost Scenarios

| App Type    | Users    | Architecture                          | Estimated Cost   |
| ----------- | -------- | ------------------------------------- | ---------------- |
| MVP         | <1k      | Supabase + Redis                      | $0–50/mo         |
| Small SaaS  | 1k–10k   | RDS + Redis                           | $100–300/mo      |
| Medium SaaS | 10k–100k | Aurora + Redis + Elasticsearch        | $500–3000/mo     |
| Large SaaS  | 100k–1M  | Aurora Multi-AZ + DynamoDB            | $3000–20000/mo   |
| Enterprise  | >1M      | Multi-region CockroachDB + ClickHouse | $20000–100000/mo |

---

## 5. ARCHITECTURE COMPARISON MATRIX

| Requirement           | Relational | Document | Graph | Wide-Column | Columnar |
| --------------------- | ---------- | -------- | ----- | ----------- | -------- |
| Strong consistency    | ⭐⭐⭐⭐⭐ | ⭐⭐     | ⭐⭐⭐⭐| ⭐⭐        | ⭐       |
| Flexible schema       | ⭐⭐       | ⭐⭐⭐⭐⭐| ⭐⭐⭐ | ⭐⭐        | ⭐       |
| Relationship modeling | ⭐⭐⭐⭐⭐ | ⭐⭐     | ⭐⭐⭐⭐⭐| ⭐         | ⭐       |
| Write throughput      | ⭐⭐⭐     | ⭐⭐⭐⭐ | ⭐⭐  | ⭐⭐⭐⭐⭐   | ⭐⭐     |
| Analytics             | ⭐⭐       | ⭐       | ⭐    | ⭐⭐        | ⭐⭐⭐⭐⭐|
| Horizontal scaling    | ⭐⭐       | ⭐⭐⭐⭐ | ⭐⭐  | ⭐⭐⭐⭐⭐   | ⭐⭐⭐⭐ |

---

## 6. PERFORMANCE CHARACTERISTICS

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

## 7. ARCHITECTURE PATTERNS

| Architecture       | Database Core | Supporting DB | Messaging  | Cache   | Cost/mo     | Scale Ceiling |
| ------------------ | ------------- | ------------- | ---------- | ------- | ----------- | ------------- |
| Monolith SaaS      | PostgreSQL    | -             | -          | Redis   | $50-300     | 100K users    |
| Microservices      | PostgreSQL    | MongoDB       | Kafka      | Redis   | $300-2000   | 1M users      |
| Serverless         | DynamoDB      | S3            | SNS/SQS    | Upstash | $20-1000    | 10M users     |
| Event-driven       | PostgreSQL    | ClickHouse    | Kafka      | Redis   | $500-5000   | 5M users      |
| Real-time platform | Redis         | PostgreSQL    | WebSockets | Redis   | $200-3000   | 2M users      |
| Global SaaS        | CockroachDB   | Redis         | Kafka      | Redis   | $800-8000   | 10M+ users    |

---

## 8. DATABASE SELECTION SCORECARD

| Requirement            | PostgreSQL | MongoDB | DynamoDB | Redis | ClickHouse | Neo4j | CockroachDB |
| ---------------------- | ---------- | ------- | -------- | ----- | ---------- | ----- | ----------- |
| Strong Consistency     | 10/10      | 7/10    | 8/10     | 3/10  | 2/10       | 9/10  | 10/10       |
| Scalability            | 7/10       | 9/10    | 10/10    | 8/10  | 9/10       | 6/10  | 9/10        |
| Cost Efficiency        | 8/10       | 6/10    | 5/10     | 7/10  | 7/10       | 5/10  | 6/10        |
| Developer Productivity | 9/10       | 8/10    | 6/10     | 7/10  | 6/10       | 6/10  | 7/10        |
| Global Distribution    | 5/10       | 7/10    | 9/10     | 4/10  | 6/10       | 4/10  | 10/10       |
| Real-time Performance  | 7/10       | 8/10    | 9/10     | 10/10 | 5/10       | 6/10  | 7/10        |

---

## 9. SELECTION FORMULA

```
Score(Database) =
  (Consistency * 0.25) +
  (Scalability * 0.25) +
  (CostEfficiency * 0.20) +
  (DeveloperProductivity * 0.15) +
  (GlobalDistribution * 0.10) +
  (RealTimePerformance * 0.05)

Scelta finale = Database con score più alto per il caso d'uso specifico
```

---

## 10. DATABASE SELECTION CHECKLIST

```
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
```

---

## 11. QUICK REFERENCE BY WORKLOAD

### OLTP (Transactional)
- **Best**: PostgreSQL, MySQL, CockroachDB
- **Avoid**: ClickHouse, BigQuery

### OLAP (Analytics)
- **Best**: ClickHouse, BigQuery, Snowflake
- **Avoid**: Redis, MongoDB

### Real-time
- **Best**: Redis, DynamoDB, Cassandra
- **Avoid**: BigQuery, Snowflake

### Graph Traversal
- **Best**: Neo4j, ArangoDB
- **Avoid**: ClickHouse, DynamoDB

### Time-series
- **Best**: TimescaleDB, InfluxDB
- **Avoid**: MongoDB, Neo4j

### Full-text Search
- **Best**: Elasticsearch, OpenSearch
- **Avoid**: DynamoDB, Cassandra

---

## 12. MIGRATION COMPLEXITY MATRIX

| From → To     | PostgreSQL | MongoDB | DynamoDB | Redis | Elasticsearch |
| ------------- | ---------- | ------- | -------- | ----- | ------------- |
| PostgreSQL    | -          | Medium  | High     | Low   | Medium        |
| MongoDB       | Medium     | -       | Medium   | Low   | Low           |
| DynamoDB      | High       | Medium  | -        | Low   | Medium        |
| MySQL         | Low        | Medium  | High     | Low   | Medium        |
| Redis         | N/A        | N/A     | N/A      | -     | N/A           |
