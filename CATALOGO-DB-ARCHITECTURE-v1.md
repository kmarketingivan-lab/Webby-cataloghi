┌─────────────────────────────────────────────────────────────────────────────┐
│ DATABASE & DATA ARCHITECTURE CATALOG                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│ TARGET: Full‑stack TypeScript / Node.js developers                          │
│ SCOPE: Database selection, scaling, cost & decision framework               │
│ VERSION: v1.0 (validated 2024–2025)                                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 1: DATABASE USE‑CASE MATRIX
═══════════════════════════════════════════════════════════════════════════════

| # | Use Case                     | Recommended DB                | Rationale |
|---|------------------------------|-------------------------------|-----------|
| 1 | E‑commerce / Inventory       | PostgreSQL / MySQL             | ACID, relations |
| 2 | Real‑time chat               | Redis + PostgreSQL             | Pub/Sub + durability |
| 3 | Analytics / BI               | ClickHouse / BigQuery          | OLAP performance |
| 4 | Social network (graph)       | Neo4j                          | Graph traversal |
| 5 | Session storage              | Redis                          | Low latency |
| 6 | Full‑text search             | Elasticsearch                  | Search engine |
| 7 | Time‑series / IoT            | Cassandra / TimescaleDB        | High write throughput |
| 8 | Document storage             | MongoDB                        | Flexible schema |
| 9 | Caching layer                | Redis                          | In‑memory |
|10 | Multi‑tenant SaaS            | PostgreSQL / CockroachDB       | Isolation + scaling |


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 2: SCALING PATTERNS
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ READ REPLICAS                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Read‑heavy workloads                                                  │
│ DBs: PostgreSQL, MySQL, CockroachDB                                         │
│ Complexity: LOW                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ SHARDING                                                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: >1TB data / extreme throughput                                        │
│ DBs: MongoDB, Cassandra, CockroachDB                                       │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ PARTITIONING                                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Large tables                                                          │
│ DBs: PostgreSQL, MySQL, ClickHouse                                         │
│ Complexity: MEDIUM                                                          │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ FEDERATION                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Multi‑domain data                                                     │
│ DBs: PostgreSQL, MySQL                                                     │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ CQRS                                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Read/write models diverge                                             │
│ DBs: PostgreSQL + Elasticsearch                                            │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT SOURCING                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ When: Auditability, event‑driven systems                                    │
│ DBs: PostgreSQL, DynamoDB, Kafka                                           │
│ Complexity: HIGH                                                            │
└─────────────────────────────────────────────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 3: COST COMPARISON (MANAGED SERVICES)
═══════════════════════════════════════════════════════════════════════════════

| Service        | Free Tier | Entry Price      | Scale Price        |
|----------------|-----------|------------------|--------------------|
| AWS RDS        | ❌        | ~15–30€/month    | 200–2000€/month    |
| PlanetScale    | ✅        | ~29$/month       | 300–3000$/month    |
| MongoDB Atlas  | ✅        | ~9$/month        | 200–5000$/month    |
| Supabase       | ✅        | ~25$/month       | 200–2000$/month    |
| Neon (Postgres)| ✅        | ~19$/month       | 200–1500$/month    |
| Turso (SQLite) | ✅        | 0–9$/month       | 100–500$/month     |


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 4: DECISION FRAMEWORK (CHECKLIST)
═══════════════════════════════════════════════════════════════════════════════

□ Strong ACID guarantees required → PostgreSQL / MySQL / CockroachDB
□ Flexible schema required         → MongoDB
□ Ultra‑low latency required       → Redis
□ Advanced search required         → Elasticsearch
□ Massive analytics required       → ClickHouse


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 5: ARCHITECTURAL NOTES (VALIDATED)
═══════════════════════════════════════════════════════════════════════════════

□ OLTP ≠ OLAP (never mix workloads)
□ Redis ≠ primary database
□ Sharding is last resort
□ Multi‑tenant isolation is a security concern
□ Cost grows non‑linearly with traffic


═══════════════════════════════════════════════════════════════════════════════
SEZIONE 6: SOURCES & VALIDATION
═══════════════════════════════════════════════════════════════════════════════

• PostgreSQL Official Docs
• AWS Well‑Architected Framework
• Google Cloud Architecture Center
• MongoDB Architecture Guide
• Martin Fowler – CQRS / Event Sourcing
• ClickHouse Documentation


