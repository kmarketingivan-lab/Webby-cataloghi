<!-- AUDIT: Catalogo privo di code block. Necessita espansione con snippet TypeScript/Next.js 14. -->
# CATALOGO DETERMINISTICO: REQUISITI → ARCHITETTURA AWS
# VERSIONE: 2.0-DETERMINISTIC
# DATA: 2026-01-25
# REGOLA: ZERO INTERPRETAZIONE - SOLO LOOKUP E DECISION TREES BINARI

================================================================================
SEZIONE 0: ISTRUZIONI DI UTILIZZO
================================================================================

REGOLA FONDAMENTALE:
- OGNI decisione architetturale DEVE seguire i decision trees in SEZIONE 1
- NON interpretare - ESEGUI la lookup table corrispondente
- Se un requisito non è nella tabella → CHIEDI all'utente

FLUSSO OBBLIGATORIO:
1. LEGGI requisiti utente
2. ESEGUI SEZIONE 1 (Decision Tree Master)
3. OTTIENI ARCH-ID (es: ARCH-001)
4. VAI A SEZIONE 3 → cerca ARCH-ID
5. COPIA lo stack ESATTO specificato
6. NON modificare, NON interpretare

================================================================================
SEZIONE 1: DECISION TREE MASTER - SELEZIONE ARCHITETTURA
================================================================================

ESEGUI QUESTO ALBERO DALL'ALTO VERSO IL BASSO.
FERMATI ALLA PRIMA CONDIZIONE TRUE.

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 1: Il sistema gestisce dati sanitari (PHI/ePHI)?                    │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [paziente, medico, ospedale,          │
│    diagnosi, cartella clinica, PHI, ePHI, HIPAA, healthcare, telehealth]    │
│ → RISULTATO: ARCH-005                                                        │
│ → VAI A: SEZIONE 3.5                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 2                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 2: Il sistema elabora pagamenti con carta di credito?               │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [carta di credito, PCI, payment,      │
│    transazione finanziaria, PAN, CVV, cardholder, acquiring, issuing]       │
│ → RISULTATO: ARCH-006                                                        │
│ → VAI A: SEZIONE 3.6                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 3                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 3: Il sistema è un videogioco multiplayer?                          │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [game server, multiplayer,            │
│    matchmaking, game session, player, lobby, latency < 50ms]                │
│ → RISULTATO: ARCH-004                                                        │
│ → VAI A: SEZIONE 3.4                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 4                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 4: Il sistema gestisce streaming video/live?                        │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [video streaming, live streaming,     │
│    VOD, HLS, DASH, transcode, OTT, broadcast, media delivery]               │
│ → RISULTATO: ARCH-003                                                        │
│ → VAI A: SEZIONE 3.3                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 5                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 5: Il sistema connette dispositivi fisici (IoT)?                    │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [IoT, sensor, device, MQTT,           │
│    telemetry, edge, PLC, OPC-UA, smart home, industrial]                    │
│ → ESEGUI SUB-DECISION 5A                                                     │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 6                                   │
└─────────────────────────────────────────────────────────────────────────────┘

  ┌───────────────────────────────────────────────────────────────────────────┐
  │ SUB-DECISION 5A: Tipo di IoT                                              │
  │                                                                            │
  │ SE contiene parole: [industrial, factory, manufacturing, OPC-UA, PLC,     │
  │    SCADA, predictive maintenance, MES]                                    │
  │ → RISULTATO: ARCH-007                                                      │
  │ → VAI A: SEZIONE 3.7                                                       │
  │                                                                            │
  │ ALTRIMENTI (consumer IoT, smart home, wearable):                          │
  │ → RISULTATO: ARCH-008                                                      │
  │ → VAI A: SEZIONE 3.8                                                       │
  └───────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 6: Il sistema richiede chat/messaggistica real-time?                │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [chat, messaging, real-time,          │
│    WebSocket, presenza online, typing indicator, social network, feed]      │
│ → RISULTATO: ARCH-002                                                        │
│ → VAI A: SEZIONE 3.2                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 7                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 7: Il sistema è un SaaS multi-tenant?                               │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [multi-tenant, SaaS, tenant           │
│    isolation, tenant_id, white-label, B2B platform]                         │
│ → RISULTATO: ARCH-009                                                        │
│ → VAI A: SEZIONE 3.9                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 8                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 8: Il sistema è primariamente analytics/data lake?                  │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [data lake, data warehouse, ETL,      │
│    analytics, BI, lakehouse, Iceberg, data pipeline]                        │
│ → RISULTATO: ARCH-010                                                        │
│ → VAI A: SEZIONE 3.10                                                        │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 9                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 9: Il sistema è una piattaforma ML/AI?                              │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [MLOps, model training, inference,    │
│    SageMaker, ML pipeline, model registry, feature store]                   │
│ → RISULTATO: ARCH-011                                                        │
│ → VAI A: SEZIONE 3.11                                                        │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 10                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 10: È richiesto Kubernetes?                                         │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [Kubernetes, K8s, EKS, kubectl,       │
│    Helm, container orchestration]                                           │
│ → RISULTATO: ARCH-014                                                        │
│ → VAI A: SEZIONE 3.14                                                        │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 11                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 11: Il sistema è un e-commerce/shop online?                         │
│                                                                              │
│ SE risposta = "sì" O contiene parole: [e-commerce, shop, cart, checkout,    │
│    product catalog, order, inventory]                                       │
│ → RISULTATO: ARCH-001                                                        │
│ → VAI A: SEZIONE 3.1                                                         │
│                                                                              │
│ SE risposta = "no" → CONTINUA A DOMANDA 12                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA 12: DEFAULT - Web application generica                              │
│                                                                              │
│ SE nessuna condizione precedente è TRUE:                                    │
│ → RISULTATO: ARCH-013 (Serverless Microservices)                            │
│ → VAI A: SEZIONE 3.13                                                        │
└─────────────────────────────────────────────────────────────────────────────┘


================================================================================
SEZIONE 2: LOOKUP TABLES - SELEZIONE SERVIZI AWS
================================================================================

ISTRUZIONI: USA queste tabelle per selezionare il servizio ESATTO.
NON scegliere servizi non elencati. NON interpretare.

--------------------------------------------------------------------------------
TABELLA 2.1: COMPUTE - SELEZIONE OBBLIGATORIA
--------------------------------------------------------------------------------

| CONDIZIONE (verifica in ordine)                    | SERVIZIO      | ARN PREFIX        |
|----------------------------------------------------|---------------|-------------------|
| durata_esecuzione <= 15 minuti AND stateless       | Lambda        | arn:aws:lambda    |
| container = TRUE AND kubernetes = FALSE            | ECS Fargate   | arn:aws:ecs       |
| container = TRUE AND kubernetes = TRUE             | EKS           | arn:aws:eks       |
| GPU_required = TRUE                                | EC2 P/G-family| arn:aws:ec2       |
| stato_persistente = TRUE AND high_memory = TRUE    | EC2 R-family  | arn:aws:ec2       |
| batch_job = TRUE AND schedule = TRUE               | AWS Batch     | arn:aws:batch     |
| DEFAULT (nessuna condizione sopra)                 | Lambda        | arn:aws:lambda    |

MAPPING ISTANZE EC2 (SE EC2 selezionato):
| REQUISITO                          | FAMIGLIA | ESEMPIO         |
|------------------------------------|----------|-----------------|
| CPU-intensive                      | C7g      | c7g.large       |
| Memory-intensive                   | R7g      | r7g.large       |
| General purpose                    | M7g      | m7g.large       |
| GPU ML training                    | P5       | p5.48xlarge     |
| GPU inference                      | G5       | g5.xlarge       |
| Gaming                             | C7g      | c7g.2xlarge     |
| Graviton (cost savings 40%)        | *7g      | m7g.medium      |

--------------------------------------------------------------------------------
TABELLA 2.2: DATABASE - SELEZIONE OBBLIGATORIA
--------------------------------------------------------------------------------

| CONDIZIONE (verifica in ordine)                              | SERVIZIO             |
|--------------------------------------------------------------|----------------------|
| schema_fisso = TRUE AND transazioni_ACID = TRUE AND SQL      | Aurora PostgreSQL    |
| schema_fisso = TRUE AND legacy_MySQL = TRUE                  | Aurora MySQL         |
| schema_fisso = TRUE AND Oracle_required = TRUE               | RDS Oracle           |
| schema_fisso = TRUE AND SQLServer_required = TRUE            | RDS SQL Server       |
| key_value = TRUE AND latency < 10ms                          | DynamoDB + DAX       |
| key_value = TRUE AND latency >= 10ms                         | DynamoDB             |
| time_series = TRUE (IoT, telemetry, metrics)                 | Timestream           |
| graph_relationships = TRUE (social, fraud)                   | Neptune              |
| full_text_search = TRUE                                      | OpenSearch           |
| document_store = TRUE AND MongoDB_compatible                 | DocumentDB           |
| ledger_immutable = TRUE (audit, blockchain)                  | QLDB                 |
| session_cache = TRUE                                         | ElastiCache Redis    |
| analytics_OLAP = TRUE                                        | Redshift             |
| DEFAULT                                                      | DynamoDB             |

CONFIGURAZIONE OBBLIGATORIA AURORA:
- Engine: aurora-postgresql (versione 15.4 o superiore)
- Instance class: db.r6g.large (MINIMO per produzione)
- Multi-AZ: TRUE (OBBLIGATORIO per produzione)
- Encryption: TRUE (KMS key)
- Backup retention: 7 giorni (MINIMO)

CONFIGURAZIONE OBBLIGATORIA DYNAMODB:
- Billing mode: PAY_PER_REQUEST (per carichi variabili)
- Point-in-time recovery: TRUE
- Encryption: AWS_OWNED_KEY (default) o KMS
- Global tables: TRUE (se multi-region)

--------------------------------------------------------------------------------
TABELLA 2.3: STORAGE - SELEZIONE OBBLIGATORIA
--------------------------------------------------------------------------------

| CONDIZIONE                                    | SERVIZIO     | STORAGE CLASS      |
|-----------------------------------------------|--------------|-------------------|
| object_storage = TRUE AND access_freq = HIGH  | S3           | STANDARD          |
| object_storage = TRUE AND access_freq = LOW   | S3           | INTELLIGENT_TIERING|
| object_storage = TRUE AND archive = TRUE      | S3 Glacier   | DEEP_ARCHIVE      |
| file_system_shared = TRUE AND NFS             | EFS          | STANDARD          |
| file_system_shared = TRUE AND Windows         | FSx Windows  | SSD               |
| file_system_shared = TRUE AND HPC             | FSx Lustre   | PERSISTENT_1      |
| block_storage = TRUE (EC2 only)               | EBS          | gp3               |

CONFIGURAZIONE OBBLIGATORIA S3:
- Versioning: Enabled (OBBLIGATORIO)
- Encryption: AES256 (SSE-S3) o aws:kms
- Public access: Block ALL (OBBLIGATORIO)
- Lifecycle rules: CONFIGURARE entro 30 giorni

--------------------------------------------------------------------------------
TABELLA 2.4: NETWORKING - SELEZIONE OBBLIGATORIA
--------------------------------------------------------------------------------

| CONDIZIONE                                    | SERVIZIO            |
|-----------------------------------------------|---------------------|
| internet_facing_api = TRUE                    | API Gateway         |
| internet_facing_static = TRUE                 | CloudFront + S3     |
| websocket = TRUE                              | API Gateway WS      |
| graphql = TRUE                                | AppSync             |
| global_latency_critical = TRUE                | Global Accelerator  |
| multi_vpc_connectivity = TRUE                 | Transit Gateway     |
| on_premises_connection = TRUE AND bandwidth > 1Gbps | Direct Connect |
| on_premises_connection = TRUE AND bandwidth <= 1Gbps | Site-to-Site VPN |
| private_aws_service_access = TRUE             | VPC Endpoints       |

CONFIGURAZIONE OBBLIGATORIA VPC:
- CIDR: /16 (65,536 IPs) - NON modificabile dopo creazione
- Subnets pubbliche: 2 (una per AZ) - CIDR /24 ciascuna
- Subnets private: 2 (una per AZ) - CIDR /24 ciascuna
- NAT Gateway: 1 per AZ (OBBLIGATORIO per private subnets)
- Internet Gateway: 1 (OBBLIGATORIO per public subnets)
- Route tables: 2 (public + private) MINIMO

CIDR OBBLIGATORI (eu-south-1 esempio):
- VPC: 10.0.0.0/16
- Public Subnet AZ-a: 10.0.1.0/24
- Public Subnet AZ-b: 10.0.2.0/24
- Private Subnet AZ-a: 10.0.10.0/24
- Private Subnet AZ-b: 10.0.20.0/24

--------------------------------------------------------------------------------
TABELLA 2.5: SECURITY - SELEZIONE OBBLIGATORIA (TUTTI I PROGETTI)
--------------------------------------------------------------------------------

| COMPONENTE                | SERVIZIO           | CONFIGURAZIONE                    |
|---------------------------|--------------------|------------------------------------|
| Encryption keys           | KMS                | aws/service-name (AWS managed)     |
| Secrets                   | Secrets Manager    | Rotation: 30 giorni                |
| Web firewall              | WAF                | AWSManagedRulesCommonRuleSet       |
| DDoS protection           | Shield Standard    | Automatico (FREE)                  |
| Threat detection          | GuardDuty          | Enable ALL findings                |
| Compliance monitoring     | Config             | Enable ALL rules                   |
| API logging               | CloudTrail         | Multi-region: TRUE                 |
| User authentication       | Cognito            | MFA: REQUIRED                      |

REGOLE SECURITY GROUPS OBBLIGATORIE:
- Ingress: DENY ALL di default
- Egress: DENY ALL di default (aggiungi solo necessario)
- NO 0.0.0.0/0 su porte diverse da 80, 443
- Descrizione: OBBLIGATORIA per ogni regola

--------------------------------------------------------------------------------
TABELLA 2.6: MESSAGING - SELEZIONE OBBLIGATORIA
--------------------------------------------------------------------------------

| CONDIZIONE                                    | SERVIZIO        | CONFIGURAZIONE     |
|-----------------------------------------------|-----------------|--------------------| 
| async_decoupling = TRUE AND order_matters = FALSE | SQS Standard | VisibilityTimeout:30|
| async_decoupling = TRUE AND order_matters = TRUE  | SQS FIFO     | FIFO suffix        |
| fan_out = TRUE (1 to N)                       | SNS             | Topic              |
| event_routing = TRUE AND filtering = TRUE     | EventBridge     | Rules + Targets    |
| streaming = TRUE AND retention > 24h          | Kinesis Streams | ON_DEMAND mode     |
| streaming = TRUE AND ETL_destination = TRUE   | Kinesis Firehose| Delivery stream    |
| workflow_orchestration = TRUE                 | Step Functions  | EXPRESS o STANDARD |

SCELTA STEP FUNCTIONS:
- SE durata_workflow <= 5 minuti AND esecuzioni > 100K/mese → EXPRESS
- SE durata_workflow > 5 minuti OR audit_trail_required = TRUE → STANDARD

--------------------------------------------------------------------------------
TABELLA 2.7: MONITORING - OBBLIGATORIO PER TUTTI I PROGETTI
--------------------------------------------------------------------------------

| COMPONENTE            | SERVIZIO       | CONFIGURAZIONE OBBLIGATORIA          |
|-----------------------|----------------|---------------------------------------|
| Metrics               | CloudWatch     | Retention: 15 mesi                    |
| Logs                  | CloudWatch Logs| Retention: 30 giorni MINIMO           |
| Distributed tracing   | X-Ray          | Sampling rate: 5% MINIMO              |
| Alarms                | CloudWatch     | SNS notification topic                |
| Dashboards            | CloudWatch     | 1 dashboard per ambiente              |

ALARMS OBBLIGATORI (crea SEMPRE):
1. Lambda errors > 1% in 5 minuti
2. API Gateway 5xx > 1% in 5 minuti
3. DynamoDB throttled requests > 0 in 1 minuto
4. RDS CPU > 80% in 10 minuti
5. ECS/EKS unhealthy tasks > 0


================================================================================
SEZIONE 3: ARCHITETTURE ESATTE - STACK COMPLETI
================================================================================

ISTRUZIONI:
- USA lo stack ESATTO specificato
- NON aggiungere servizi non elencati
- NON rimuovere servizi elencati come OBBLIGATORIO
- I nomi delle risorse seguono il pattern: {project}-{env}-{service}-{resource}

================================================================================
3.1 ARCH-001: E-COMMERCE SERVERLESS
================================================================================

PREREQUISITI:
- Account AWS con accesso a eu-south-1 (o region specificata)
- AWS CLI configurato
- Dominio registrato (opzionale per custom domain)

STACK OBBLIGATORIO (20 risorse):
┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: FRONTEND                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│ 1. S3 Bucket (static website)                                                │
│    Nome: {project}-{env}-frontend                                            │
│    Versioning: Enabled                                                       │
│    Public access: BLOCKED                                                    │
│    Encryption: AES256                                                        │
│                                                                              │
│ 2. CloudFront Distribution                                                   │
│    Origin: S3 bucket above (OAC)                                             │
│    Price class: PriceClass_100 (solo EU/NA)                                  │
│    Default TTL: 86400 (24h)                                                  │
│    Viewer protocol: redirect-to-https                                        │
│    WAF: Attached (vedi sotto)                                                │
│                                                                              │
│ 3. WAF WebACL                                                                │
│    Rules: AWSManagedRulesCommonRuleSet                                       │
│           AWSManagedRulesKnownBadInputsRuleSet                               │
│    Scope: CLOUDFRONT                                                         │
│                                                                              │
│ 4. Route 53 Hosted Zone (se custom domain)                                   │
│    Record: A alias → CloudFront                                              │
│    Record: AAAA alias → CloudFront                                           │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: API                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 5. API Gateway (HTTP API)                                                    │
│    Nome: {project}-{env}-api                                                 │
│    Protocol: HTTP                                                            │
│    CORS: Enabled (origin: CloudFront domain)                                 │
│    Throttling: 1000 req/s burst, 500 req/s rate                              │
│    Stage: $default (auto-deploy)                                             │
│                                                                              │
│ 6. Lambda Function: ProductService                                           │
│    Nome: {project}-{env}-product-service                                     │
│    Runtime: nodejs20.x                                                       │
│    Memory: 512 MB                                                            │
│    Timeout: 30 seconds                                                       │
│    Handler: index.handler                                                    │
│    Environment: TABLE_NAME={project}-{env}-products                          │
│                                                                              │
│ 7. Lambda Function: CartService                                              │
│    Nome: {project}-{env}-cart-service                                        │
│    Runtime: nodejs20.x                                                       │
│    Memory: 512 MB                                                            │
│    Timeout: 30 seconds                                                       │
│    Environment: TABLE_NAME={project}-{env}-carts                             │
│                                                                              │
│ 8. Lambda Function: OrderService                                             │
│    Nome: {project}-{env}-order-service                                       │
│    Runtime: nodejs20.x                                                       │
│    Memory: 1024 MB                                                           │
│    Timeout: 60 seconds                                                       │
│    Environment: TABLE_NAME={project}-{env}-orders                            │
│                 QUEUE_URL={SQS queue URL}                                    │
│                                                                              │
│ 9. Lambda Function: UserService                                              │
│    Nome: {project}-{env}-user-service                                        │
│    Runtime: nodejs20.x                                                       │
│    Memory: 512 MB                                                            │
│    Timeout: 30 seconds                                                       │
│    Environment: USER_POOL_ID={Cognito pool ID}                               │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: DATA                                                                  │
├──────────────────────────────────────────────────────────────────────────────┤
│ 10. DynamoDB Table: Products                                                 │
│     Nome: {project}-{env}-products                                           │
│     Partition key: productId (S)                                             │
│     Billing: PAY_PER_REQUEST                                                 │
│     Encryption: AWS_OWNED                                                    │
│     PITR: Enabled                                                            │
│     GSI: category-index (category, createdAt)                                │
│                                                                              │
│ 11. DynamoDB Table: Carts                                                    │
│     Nome: {project}-{env}-carts                                              │
│     Partition key: userId (S)                                                │
│     Billing: PAY_PER_REQUEST                                                 │
│     TTL: expiresAt (7 giorni)                                                │
│                                                                              │
│ 12. DynamoDB Table: Orders                                                   │
│     Nome: {project}-{env}-orders                                             │
│     Partition key: orderId (S)                                               │
│     Sort key: createdAt (S)                                                  │
│     Billing: PAY_PER_REQUEST                                                 │
│     Stream: NEW_AND_OLD_IMAGES                                               │
│     GSI: userId-index (userId, createdAt)                                    │
│                                                                              │
│ 13. ElastiCache Redis (session cache)                                        │
│     Nome: {project}-{env}-cache                                              │
│     Node type: cache.t3.micro (dev) / cache.r6g.large (prod)                 │
│     Engine: 7.0                                                              │
│     Multi-AZ: TRUE (prod only)                                               │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ASYNC PROCESSING                                                      │
├──────────────────────────────────────────────────────────────────────────────┤
│ 14. SQS Queue: OrderProcessing                                               │
│     Nome: {project}-{env}-order-processing                                   │
│     Type: Standard                                                           │
│     Visibility timeout: 300 seconds                                          │
│     DLQ: {project}-{env}-order-processing-dlq                                │
│     Max receive count: 3                                                     │
│                                                                              │
│ 15. SQS Queue: DLQ                                                           │
│     Nome: {project}-{env}-order-processing-dlq                               │
│     Message retention: 14 days                                               │
│                                                                              │
│ 16. EventBridge Rule: OrderCreated                                           │
│     Nome: {project}-{env}-order-created                                      │
│     Event pattern: {"source": ["ecommerce.orders"]}                          │
│     Target: SNS topic (notifications)                                        │
│                                                                              │
│ 17. SNS Topic: Notifications                                                 │
│     Nome: {project}-{env}-notifications                                      │
│     Subscriptions: Email, Lambda (optional)                                  │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: SECURITY                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│ 18. Cognito User Pool                                                        │
│     Nome: {project}-{env}-users                                              │
│     MFA: OPTIONAL (prod: REQUIRED)                                           │
│     Password policy: Length 8, uppercase, lowercase, number                  │
│     Email verification: REQUIRED                                             │
│     App client: {project}-{env}-web-client (no secret)                       │
│                                                                              │
│ 19. KMS Key                                                                  │
│     Alias: alias/{project}-{env}-key                                         │
│     Key usage: ENCRYPT_DECRYPT                                               │
│     Rotation: Enabled (annual)                                               │
│                                                                              │
│ 20. IAM Roles                                                                │
│     - {project}-{env}-lambda-execution-role                                  │
│     - {project}-{env}-api-gateway-role                                       │
│     Policies: Least privilege (vedi SEZIONE 4)                               │
└──────────────────────────────────────────────────────────────────────────────┘

COSTI MENSILI ESATTI (eu-south-1, carico medio):
| Servizio         | Quantità              | Costo USD/mese |
|------------------|-----------------------|----------------|
| Lambda           | 1M invocations, 512MB | $8.35          |
| API Gateway      | 1M requests           | $3.50          |
| DynamoDB         | 25 WCU, 25 RCU avg    | $18.67         |
| ElastiCache      | t3.micro              | $12.41         |
| CloudFront       | 100GB transfer        | $8.50          |
| S3               | 10GB storage          | $0.24          |
| SQS              | 1M requests           | $0.40          |
| Cognito          | 10K MAU               | $0.00 (free)   |
| WAF              | 1M requests           | $6.00          |
| CloudWatch       | Base                  | $3.00          |
| TOTALE STIMATO   |                       | ~$61.07        |


================================================================================
3.2 ARCH-002: SOCIAL MEDIA / REAL-TIME CHAT
================================================================================

PREREQUISITI:
- Account AWS
- Mobile/Web app con WebSocket support

STACK OBBLIGATORIO (18 risorse):
┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: REAL-TIME                                                             │
├──────────────────────────────────────────────────────────────────────────────┤
│ 1. API Gateway WebSocket API                                                 │
│    Nome: {project}-{env}-ws-api                                              │
│    Route selection: $request.body.action                                     │
│    Routes OBBLIGATORIE:                                                      │
│      - $connect → Lambda: ConnectionHandler                                  │
│      - $disconnect → Lambda: DisconnectionHandler                            │
│      - $default → Lambda: MessageHandler                                     │
│      - sendMessage → Lambda: SendMessageHandler                              │
│                                                                              │
│ 2. Lambda: ConnectionHandler                                                 │
│    Nome: {project}-{env}-ws-connect                                          │
│    Runtime: nodejs20.x                                                       │
│    Memory: 256 MB                                                            │
│    Timeout: 10 seconds                                                       │
│    Azione: Salva connectionId in DynamoDB                                    │
│                                                                              │
│ 3. Lambda: DisconnectionHandler                                              │
│    Nome: {project}-{env}-ws-disconnect                                       │
│    Runtime: nodejs20.x                                                       │
│    Memory: 256 MB                                                            │
│    Timeout: 10 seconds                                                       │
│    Azione: Rimuove connectionId da DynamoDB                                  │
│                                                                              │
│ 4. Lambda: SendMessageHandler                                                │
│    Nome: {project}-{env}-ws-send                                             │
│    Runtime: nodejs20.x                                                       │
│    Memory: 512 MB                                                            │
│    Timeout: 30 seconds                                                       │
│    Azione: Salva messaggio + broadcast a recipients                          │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: DATA                                                                  │
├──────────────────────────────────────────────────────────────────────────────┤
│ 5. DynamoDB Table: Connections                                               │
│    Nome: {project}-{env}-connections                                         │
│    Partition key: connectionId (S)                                           │
│    TTL: expiresAt (2 hours)                                                  │
│    GSI: userId-index (userId, connectedAt)                                   │
│                                                                              │
│ 6. DynamoDB Table: Messages                                                  │
│    Nome: {project}-{env}-messages                                            │
│    Partition key: conversationId (S)                                         │
│    Sort key: timestamp (S)                                                   │
│    Billing: PAY_PER_REQUEST                                                  │
│    Stream: NEW_IMAGE                                                         │
│                                                                              │
│ 7. DynamoDB Table: Conversations                                             │
│    Nome: {project}-{env}-conversations                                       │
│    Partition key: conversationId (S)                                         │
│    Attributes: participants (SS), lastMessageAt (S)                          │
│                                                                              │
│ 8. DynamoDB Table: UserConversations                                         │
│    Nome: {project}-{env}-user-conversations                                  │
│    Partition key: userId (S)                                                 │
│    Sort key: conversationId (S)                                              │
│    GSI: unreadCount-index                                                    │
│                                                                              │
│ 9. ElastiCache Redis                                                         │
│    Nome: {project}-{env}-presence                                            │
│    Uso: Presence (online/offline), typing indicators                         │
│    Node type: cache.t3.micro (dev) / cache.r6g.large (prod)                  │
│    TTL default: 60 seconds                                                   │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: NOTIFICATIONS                                                         │
├──────────────────────────────────────────────────────────────────────────────┤
│ 10. SNS Platform Application (iOS)                                           │
│     Nome: {project}-{env}-apns                                               │
│     Platform: APNS_SANDBOX (dev) / APNS (prod)                               │
│     Credential: p8 key file                                                  │
│                                                                              │
│ 11. SNS Platform Application (Android)                                       │
│     Nome: {project}-{env}-fcm                                                │
│     Platform: GCM                                                            │
│     Credential: FCM server key                                               │
│                                                                              │
│ 12. Lambda: PushNotificationHandler                                          │
│     Nome: {project}-{env}-push-handler                                       │
│     Trigger: DynamoDB Stream (Messages table)                                │
│     Azione: Invia push a utenti offline                                      │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: MEDIA STORAGE                                                         │
├──────────────────────────────────────────────────────────────────────────────┤
│ 13. S3 Bucket: Media                                                         │
│     Nome: {project}-{env}-media                                              │
│     CORS: Enabled (app origins)                                              │
│     Lifecycle: Transition to IA after 90 days                                │
│                                                                              │
│ 14. CloudFront Distribution (media)                                          │
│     Origin: S3 media bucket                                                  │
│     Signed URLs: Enabled (per content privati)                               │
│                                                                              │
│ 15. Lambda: MediaProcessor                                                   │
│     Nome: {project}-{env}-media-processor                                    │
│     Trigger: S3 Event (ObjectCreated)                                        │
│     Azione: Thumbnail generation, image resize                               │
│     Memory: 1024 MB                                                          │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: SECURITY                                                              │
├──────────────────────────────────────────────────────────────────────────────┤
│ 16. Cognito User Pool                                                        │
│     Nome: {project}-{env}-users                                              │
│     Social providers: Google, Apple (opzionale)                              │
│     Custom attributes: displayName, avatarUrl                                │
│                                                                              │
│ 17. Lambda Authorizer (WebSocket)                                            │
│     Nome: {project}-{env}-ws-authorizer                                      │
│     Tipo: REQUEST                                                            │
│     Cache: 300 seconds                                                       │
│                                                                              │
│ 18. IAM Roles                                                                │
│     - Lambda execution role                                                  │
│     - API Gateway invoke role                                                │
└──────────────────────────────────────────────────────────────────────────────┘

SCHEMA MESSAGGIO OBBLIGATORIO (DynamoDB Messages):
{
  "conversationId": "conv_123",        // Partition key
  "timestamp": "2026-01-25T18:30:00Z", // Sort key (ISO 8601)
  "messageId": "msg_uuid",
  "senderId": "user_123",
  "content": "Hello!",
  "type": "text",                      // text | image | video | file
  "mediaUrl": null,                    // S3 URL se type != text
  "readBy": ["user_456"],              // Set di userId
  "createdAt": "2026-01-25T18:30:00Z"
}

================================================================================
3.5 ARCH-005: HEALTHCARE HIPAA
================================================================================

PREREQUISITI:
- BAA (Business Associate Agreement) firmato con AWS
- Account in region HIPAA-eligible (us-east-1, us-west-2, eu-west-1)
- Compliance officer designato

STACK OBBLIGATORIO (25 risorse):
┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: VPC (ISOLAMENTO OBBLIGATORIO)                                         │
├──────────────────────────────────────────────────────────────────────────────┤
│ 1. VPC                                                                       │
│    Nome: {project}-{env}-vpc                                                 │
│    CIDR: 10.0.0.0/16                                                         │
│    DNS hostnames: Enabled                                                    │
│    DNS resolution: Enabled                                                   │
│                                                                              │
│ 2. Subnets (4 OBBLIGATORIE)                                                  │
│    - {project}-{env}-public-a: 10.0.1.0/24 (AZ-a)                            │
│    - {project}-{env}-public-b: 10.0.2.0/24 (AZ-b)                            │
│    - {project}-{env}-private-a: 10.0.10.0/24 (AZ-a) ← PHI qui               │
│    - {project}-{env}-private-b: 10.0.20.0/24 (AZ-b) ← PHI qui               │
│                                                                              │
│ 3. NAT Gateways (2 OBBLIGATORI per HA)                                       │
│    - {project}-{env}-nat-a in public-a                                       │
│    - {project}-{env}-nat-b in public-b                                       │
│                                                                              │
│ 4. Internet Gateway                                                          │
│    Nome: {project}-{env}-igw                                                 │
│                                                                              │
│ 5. VPC Flow Logs (OBBLIGATORIO per audit)                                    │
│    Destination: CloudWatch Logs                                              │
│    Traffic type: ALL                                                         │
│    Retention: 365 giorni MINIMO                                              │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ENCRYPTION (OBBLIGATORIO PER HIPAA)                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 6. KMS Key (Customer Managed)                                                │
│    Alias: alias/{project}-{env}-hipaa-key                                    │
│    Key spec: SYMMETRIC_DEFAULT                                               │
│    Key usage: ENCRYPT_DECRYPT                                                │
│    Rotation: ENABLED (annuale)                                               │
│    Policy: Solo ruoli autorizzati                                            │
│    USARE PER: RDS, S3, EBS, CloudWatch Logs, Secrets Manager                 │
│                                                                              │
│ 7. Secrets Manager Secret                                                    │
│    Nome: {project}/{env}/db-credentials                                      │
│    Rotation: 30 giorni                                                       │
│    KMS: {project}-{env}-hipaa-key                                            │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: DATA STORAGE (PHI)                                                    │
├──────────────────────────────────────────────────────────────────────────────┤
│ 8. Aurora PostgreSQL (Primary PHI store)                                     │
│    Identifier: {project}-{env}-db                                            │
│    Engine: aurora-postgresql 15.4                                            │
│    Instance: db.r6g.large MINIMO (prod)                                      │
│    Multi-AZ: TRUE (OBBLIGATORIO)                                             │
│    Encryption: KMS key above (OBBLIGATORIO)                                  │
│    Backup retention: 35 giorni MINIMO                                        │
│    Deletion protection: TRUE                                                 │
│    Performance Insights: ENABLED                                             │
│    Enhanced monitoring: 60 seconds                                           │
│    Subnet group: Private subnets only                                        │
│    Security group: Only from application layer                               │
│                                                                              │
│ 9. S3 Bucket: PHI Documents                                                  │
│    Nome: {project}-{env}-phi-documents                                       │
│    Encryption: aws:kms (key above)                                           │
│    Versioning: ENABLED                                                       │
│    Public access: BLOCK ALL                                                  │
│    Object lock: COMPLIANCE mode (7 years)                                    │
│    Logging: Access logs to separate bucket                                   │
│    Lifecycle: NO auto-delete (compliance)                                    │
│                                                                              │
│ 10. S3 Bucket: Audit Logs                                                    │
│     Nome: {project}-{env}-audit-logs                                         │
│     Object lock: COMPLIANCE mode                                             │
│     Retention: 7 years MINIMO                                                │
│                                                                              │
│ 11. HealthLake Data Store (FHIR)                                             │
│     Nome: {project}-{env}-healthlake                                         │
│     Type: FHIR_R4                                                            │
│     Encryption: KMS key above                                                │
│     Preload: TRUE (FHIR resources)                                           │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: COMPUTE                                                               │
├──────────────────────────────────────────────────────────────────────────────┤
│ 12. ECS Cluster (Fargate)                                                    │
│     Nome: {project}-{env}-cluster                                            │
│     Capacity providers: FARGATE, FARGATE_SPOT                                │
│     Container Insights: ENABLED                                              │
│                                                                              │
│ 13. ECS Service: API                                                         │
│     Nome: {project}-{env}-api-service                                        │
│     Launch type: FARGATE                                                     │
│     Desired count: 2 MINIMO (prod)                                           │
│     Network: Private subnets                                                 │
│     Task definition: vedi sotto                                              │
│                                                                              │
│ 14. ECS Task Definition                                                      │
│     Family: {project}-{env}-api                                              │
│     CPU: 512 (0.5 vCPU)                                                      │
│     Memory: 1024 (1 GB)                                                      │
│     Network mode: awsvpc                                                     │
│     Execution role: Con accesso a Secrets Manager                            │
│     Task role: Con accesso a RDS, S3, HealthLake                             │
│     Logging: awslogs driver → CloudWatch                                     │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ACCESS CONTROL                                                        │
├──────────────────────────────────────────────────────────────────────────────┤
│ 15. Cognito User Pool                                                        │
│     Nome: {project}-{env}-users                                              │
│     MFA: REQUIRED (OBBLIGATORIO per HIPAA)                                   │
│     Advanced security: AUDIT mode MINIMO                                     │
│     Account recovery: Admin only                                             │
│     Password: 12 chars, upper, lower, number, symbol                         │
│     Token validity: Access 1h, ID 1h, Refresh 30d                            │
│                                                                              │
│ 16. API Gateway (REST)                                                       │
│     Nome: {project}-{env}-api                                                │
│     Authorizer: Cognito User Pool                                            │
│     Endpoint type: PRIVATE (VPC endpoint)                                    │
│     Logging: ENABLED (full request/response)                                 │
│     Throttling: 1000/s burst, 500/s rate                                     │
│                                                                              │
│ 17. VPC Endpoints (OBBLIGATORI - no internet per PHI)                        │
│     - com.amazonaws.{region}.execute-api                                     │
│     - com.amazonaws.{region}.s3 (Gateway)                                    │
│     - com.amazonaws.{region}.secretsmanager                                  │
│     - com.amazonaws.{region}.kms                                             │
│     - com.amazonaws.{region}.logs                                            │
│     - com.amazonaws.{region}.ecr.api                                         │
│     - com.amazonaws.{region}.ecr.dkr                                         │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: AUDIT & COMPLIANCE (OBBLIGATORIO)                                     │
├──────────────────────────────────────────────────────────────────────────────┤
│ 18. CloudTrail                                                               │
│     Nome: {project}-{env}-trail                                              │
│     Multi-region: TRUE                                                       │
│     Log file validation: TRUE                                                │
│     S3 bucket: {project}-{env}-audit-logs                                    │
│     KMS encryption: TRUE                                                     │
│     Data events: S3 (PHI bucket), Lambda                                     │
│     Management events: All                                                   │
│                                                                              │
│ 19. AWS Config                                                               │
│     Recording: All resources, all regions                                    │
│     Rules OBBLIGATORIE:                                                      │
│       - s3-bucket-ssl-requests-only                                          │
│       - encrypted-volumes                                                    │
│       - rds-storage-encrypted                                                │
│       - cloudtrail-enabled                                                   │
│       - iam-password-policy                                                  │
│       - mfa-enabled-for-iam-console-access                                   │
│       - vpc-flow-logs-enabled                                                │
│                                                                              │
│ 20. GuardDuty                                                                │
│     ENABLED in all regions                                                   │
│     S3 protection: TRUE                                                      │
│     Kubernetes protection: TRUE (se EKS)                                     │
│     Malware protection: TRUE                                                 │
│                                                                              │
│ 21. Security Hub                                                             │
│     Standards: AWS Foundational Security Best Practices                      │
│     Standards: CIS AWS Foundations                                           │
│     Integrations: GuardDuty, Config, Inspector                               │
│                                                                              │
│ 22. CloudWatch Logs                                                          │
│     Log groups per: VPC Flow, API Gateway, ECS, Application                  │
│     Retention: 365 giorni MINIMO                                             │
│     Encryption: KMS key                                                      │
│                                                                              │
│ 23. SNS Topic: Security Alerts                                               │
│     Nome: {project}-{env}-security-alerts                                    │
│     Subscriptions: Security team email                                       │
│                                                                              │
│ 24. CloudWatch Alarms                                                        │
│     - Root account login                                                     │
│     - Unauthorized API calls                                                 │
│     - S3 bucket policy changes                                               │
│     - CloudTrail config changes                                              │
│     - Security group changes                                                 │
│     - IAM policy changes                                                     │
│                                                                              │
│ 25. AWS Backup                                                               │
│     Vault: {project}-{env}-backup-vault                                      │
│     Encryption: KMS key                                                      │
│     Plan: Daily backup, 35 days retention                                    │
│     Cross-region: TRUE (DR region)                                           │
└──────────────────────────────────────────────────────────────────────────────┘

SECURITY GROUP RULES OBBLIGATORIE:
┌───────────────────────────────────────────────────────────────────────────────┐
│ SG: {project}-{env}-alb-sg (se ALB pubblico)                                  │
│   Ingress: 443/tcp from 0.0.0.0/0                                             │
│   Egress: All to ECS SG                                                       │
├───────────────────────────────────────────────────────────────────────────────┤
│ SG: {project}-{env}-ecs-sg                                                    │
│   Ingress: 8080/tcp from ALB SG                                               │
│   Egress: 5432/tcp to RDS SG                                                  │
│   Egress: 443/tcp to VPC endpoints                                            │
├───────────────────────────────────────────────────────────────────────────────┤
│ SG: {project}-{env}-rds-sg                                                    │
│   Ingress: 5432/tcp from ECS SG only                                          │
│   Egress: None                                                                │
└───────────────────────────────────────────────────────────────────────────────┘


================================================================================
3.6 ARCH-006: FINTECH PCI-DSS
================================================================================

PREREQUISITI:
- Account AWS PCI DSS compliant (verifica AOC)
- QSA engagement per certificazione
- Penetration testing autorizzato

STACK OBBLIGATORIO (22 risorse):
┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: NETWORK SEGMENTATION (CDE - Cardholder Data Environment)              │
├──────────────────────────────────────────────────────────────────────────────┤
│ 1. VPC (dedicato per CDE)                                                    │
│    Nome: {project}-{env}-cde-vpc                                             │
│    CIDR: 10.100.0.0/16 (separato da altri workload)                          │
│                                                                              │
│ 2. Subnets (6 OBBLIGATORIE per CDE)                                          │
│    - {project}-{env}-dmz-a: 10.100.1.0/24 (WAF, ALB)                         │
│    - {project}-{env}-dmz-b: 10.100.2.0/24 (WAF, ALB)                         │
│    - {project}-{env}-app-a: 10.100.10.0/24 (Application)                     │
│    - {project}-{env}-app-b: 10.100.20.0/24 (Application)                     │
│    - {project}-{env}-data-a: 10.100.30.0/24 (Database - CDE)                 │
│    - {project}-{env}-data-b: 10.100.40.0/24 (Database - CDE)                 │
│                                                                              │
│ 3. NACLs (OBBLIGATORIE per ogni subnet tier)                                 │
│    DMZ NACL:                                                                 │
│      Inbound: 443 from 0.0.0.0/0, deny all else                              │
│      Outbound: 8443 to app subnets, deny all else                            │
│    APP NACL:                                                                 │
│      Inbound: 8443 from DMZ only                                             │
│      Outbound: 5432 to data subnets                                          │
│    DATA NACL:                                                                │
│      Inbound: 5432 from APP only                                             │
│      Outbound: None (stateful responses only)                                │
│                                                                              │
│ 4. PrivateLink (NO internet per CDE)                                         │
│    Endpoints OBBLIGATORI:                                                    │
│      - com.amazonaws.{region}.kms                                            │
│      - com.amazonaws.{region}.secretsmanager                                 │
│      - com.amazonaws.{region}.logs                                           │
│      - com.amazonaws.{region}.monitoring                                     │
│      - com.amazonaws.{region}.s3 (Gateway)                                   │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ENCRYPTION (PCI DSS Requirement 3 & 4)                                │
├──────────────────────────────────────────────────────────────────────────────┤
│ 5. KMS Key (Customer Managed - PAN encryption)                               │
│    Alias: alias/{project}-{env}-pan-key                                      │
│    Key policy: Solo applicazione autorizzata                                 │
│    Rotation: ENABLED                                                         │
│    Usage: Tokenization/detokenization di PAN                                 │
│                                                                              │
│ 6. CloudHSM Cluster (SE PCI PIN required)                                    │
│    Nome: {project}-{env}-hsm                                                 │
│    HSM type: hsm1.medium                                                     │
│    Subnets: Data subnets                                                     │
│    Uso: PIN translation, key management                                      │
│                                                                              │
│ 7. Secrets Manager                                                           │
│    Secrets OBBLIGATORI:                                                      │
│      - {project}/{env}/db-master-credentials                                 │
│      - {project}/{env}/payment-gateway-api-key                               │
│      - {project}/{env}/encryption-keys                                       │
│    Rotation: 30 giorni                                                       │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: COMPUTE (Requirement 6 - Secure Systems)                              │
├──────────────────────────────────────────────────────────────────────────────┤
│ 8. EKS Cluster                                                               │
│    Nome: {project}-{env}-eks                                                 │
│    Version: 1.29 o superiore                                                 │
│    Endpoint access: Private only                                             │
│    Encryption: Secrets encryption con KMS                                    │
│    Logging: ALL (api, audit, authenticator, controllerManager, scheduler)    │
│    Subnets: APP subnets only                                                 │
│                                                                              │
│ 9. EKS Node Group                                                            │
│    Instance type: m6g.large MINIMO                                           │
│    Disk encryption: TRUE                                                     │
│    AMI: Amazon Linux 2 (EKS optimized)                                       │
│    Scaling: min 2, max 10, desired 3                                         │
│                                                                              │
│ 10. Kubernetes Namespaces (OBBLIGATORI)                                      │
│     - {project}-payment: Payment processing                                  │
│     - {project}-api: API services                                            │
│     - {project}-monitoring: Observability                                    │
│     Network policies: DENY ALL default, allow specifici                      │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: DATA (Requirement 3 - Protect Stored Data)                            │
├──────────────────────────────────────────────────────────────────────────────┤
│ 11. Aurora PostgreSQL (Non-CDE data)                                         │
│     Identifier: {project}-{env}-db                                           │
│     Subnets: Data subnets                                                    │
│     Encryption: KMS                                                          │
│     Multi-AZ: TRUE                                                           │
│     Backup: 35 giorni                                                        │
│                                                                              │
│ 12. DynamoDB (Tokenized PAN vault)                                           │
│     Table: {project}-{env}-token-vault                                       │
│     Partition key: token (S)                                                 │
│     Encryption: KMS CMK                                                      │
│     NOTA: Memorizza solo TOKEN, MAI PAN in chiaro                            │
│                                                                              │
│ 13. S3 Bucket (Audit logs)                                                   │
│     Nome: {project}-{env}-pci-audit-logs                                     │
│     Object lock: COMPLIANCE                                                  │
│     Retention: 1 year MINIMO (PCI requirement)                               │
│     Versioning: ENABLED                                                      │
│     Replication: Cross-region (DR)                                           │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ACCESS CONTROL (Requirement 7 & 8)                                    │
├──────────────────────────────────────────────────────────────────────────────┤
│ 14. WAF (DMZ protection)                                                     │
│     WebACL: {project}-{env}-pci-waf                                          │
│     Rules OBBLIGATORIE:                                                      │
│       - AWSManagedRulesCommonRuleSet                                         │
│       - AWSManagedRulesKnownBadInputsRuleSet                                 │
│       - AWSManagedRulesSQLiRuleSet                                           │
│       - AWSManagedRulesAnonymousIpList                                       │
│       - Rate limit: 1000 req/5min per IP                                     │
│                                                                              │
│ 15. Network Load Balancer                                                    │
│     Nome: {project}-{env}-nlb                                                │
│     Scheme: internal                                                         │
│     Subnets: DMZ subnets                                                     │
│     Cross-zone: ENABLED                                                      │
│                                                                              │
│ 16. Cognito User Pool (Admin access)                                         │
│     MFA: REQUIRED                                                            │
│     Password: 12 chars, complexity                                           │
│     Session timeout: 15 minuti MAX                                           │
│     Failed attempts lockout: 6 attempts                                      │
│                                                                              │
│ 17. IAM Roles (Least privilege)                                              │
│     - {project}-{env}-payment-processor-role                                 │
│     - {project}-{env}-api-service-role                                       │
│     - {project}-{env}-audit-reader-role                                      │
│     Policy: DENY by default, explicit ALLOW                                  │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: LOGGING & MONITORING (Requirement 10)                                 │
├──────────────────────────────────────────────────────────────────────────────┤
│ 18. CloudTrail                                                               │
│     Multi-region: TRUE                                                       │
│     Data events: S3, Lambda, DynamoDB                                        │
│     Log file integrity: ENABLED                                              │
│     KMS encryption: TRUE                                                     │
│                                                                              │
│ 19. Config                                                                   │
│     Rules OBBLIGATORIE PCI:                                                  │
│       - pci-dss-3-2-1-config-rules (AWS managed pack)                        │
│                                                                              │
│ 20. GuardDuty                                                                │
│     All protections: ENABLED                                                 │
│     Finding export: S3 bucket                                                │
│                                                                              │
│ 21. Security Hub                                                             │
│     Standards: PCI DSS v3.2.1                                                │
│     Auto-remediation: Configure per critical findings                        │
│                                                                              │
│ 22. CloudWatch                                                               │
│     Alarms OBBLIGATORI:                                                      │
│       - Payment processing failures                                          │
│       - Unauthorized access attempts                                         │
│       - HSM health                                                           │
│       - Encryption key usage anomalies                                       │
│       - Network anomalies                                                    │
│     Retention: 365 giorni MINIMO                                             │
└──────────────────────────────────────────────────────────────────────────────┘

TOKENIZATION FLOW OBBLIGATORIO:
1. Card → API Gateway (TLS 1.2+)
2. Lambda Tokenizer → KMS (encrypt PAN)
3. Store TOKEN → DynamoDB (solo token, MAI PAN)
4. Payment Gateway → usa token per transazione
5. MAI loggare PAN, CVV, PIN in chiaro

================================================================================
3.9 ARCH-009: SAAS MULTI-TENANT
================================================================================

PREREQUISITI:
- Definire tier model (es: Basic, Professional, Enterprise)
- Decidere isolation model (vedi sotto)

DECISION TREE ISOLATION MODEL:
┌──────────────────────────────────────────────────────────────────────────────┐
│ DOMANDA: Qual è il requisito di isolation?                                   │
│                                                                              │
│ SE compliance = (HIPAA OR PCI-DSS OR SOC2) → SILO MODEL                     │
│ SE customer_size = Enterprise AND paying > $10K/month → SILO MODEL           │
│ SE noisy_neighbor_critical = TRUE → BRIDGE MODEL                             │
│ DEFAULT → POOL MODEL                                                         │
└──────────────────────────────────────────────────────────────────────────────┘

POOL MODEL STACK (15 risorse - shared everything):
┌──────────────────────────────────────────────────────────────────────────────┐
│ 1. Cognito User Pool (SHARED)                                                │
│    Nome: {project}-{env}-shared-users                                        │
│    Custom attribute: tenant_id (immutable)                                   │
│    Groups: Per-tenant groups                                                 │
│                                                                              │
│ 2. API Gateway (SHARED)                                                      │
│    Nome: {project}-{env}-api                                                 │
│    Authorizer: Cognito                                                       │
│    Usage plans: Per tier (Basic: 1000/day, Pro: 10000/day, Ent: unlimited)   │
│    API keys: Per tenant                                                      │
│                                                                              │
│ 3. Lambda Functions (SHARED)                                                 │
│    Pattern: Extract tenant_id from JWT → pass to all operations              │
│    Layer: {project}-{env}-tenant-context-layer                               │
│    Environment: Shared, tenant determined at runtime                         │
│                                                                              │
│ 4. DynamoDB Tables (SHARED - tenant_id as partition key)                     │
│    Table: {project}-{env}-data                                               │
│    Partition key: tenant_id (S)                                              │
│    Sort key: entity_id (S)                                                   │
│    Item schema: {tenant_id, entity_id, entity_type, data, created_at}        │
│    IAM condition: dynamodb:LeadingKeys = ${cognito:custom:tenant_id}         │
│                                                                              │
│ 5. S3 Bucket (SHARED - prefix per tenant)                                    │
│    Nome: {project}-{env}-tenant-data                                         │
│    Structure: s3://bucket/{tenant_id}/...                                    │
│    Bucket policy: Condition on prefix                                        │
│                                                                              │
│ 6-15. Standard: VPC, CloudWatch, Secrets Manager, KMS, etc.                  │
└──────────────────────────────────────────────────────────────────────────────┘

SILO MODEL STACK (risorse per tenant):
┌──────────────────────────────────────────────────────────────────────────────┐
│ PER OGNI TENANT (provisioned via CodePipeline):                              │
│                                                                              │
│ 1. Cognito User Pool: {project}-{env}-{tenant_id}-users                      │
│ 2. API Gateway: {project}-{env}-{tenant_id}-api                              │
│ 3. Lambda Functions: {project}-{env}-{tenant_id}-*                           │
│ 4. DynamoDB Tables: {project}-{env}-{tenant_id}-data                         │
│ 5. S3 Bucket: {project}-{env}-{tenant_id}-storage                            │
│ 6. KMS Key: alias/{project}-{env}-{tenant_id}-key                            │
│ 7. CloudWatch Log Group: /aws/{project}/{env}/{tenant_id}                    │
│                                                                              │
│ ONBOARDING PIPELINE:                                                         │
│ CodePipeline: {project}-{env}-tenant-onboarding                              │
│   Source: CodeCommit (tenant-infrastructure repo)                            │
│   Build: CloudFormation package                                              │
│   Deploy: CloudFormation stack per tenant                                    │
│   Parameters: tenant_id, tier, configuration                                 │
└──────────────────────────────────────────────────────────────────────────────┘

TENANT CONTEXT LAMBDA LAYER (OBBLIGATORIO per Pool Model):
```javascript
// /opt/nodejs/tenant-context.js
exports.extractTenantId = (event) => {
  // UNICO modo per ottenere tenant_id: dal JWT validato
  const claims = event.requestContext.authorizer.claims;
  const tenantId = claims['custom:tenant_id'];
  
  if (!tenantId) {
    throw new Error('UNAUTHORIZED: Missing tenant_id');
  }
  
  return tenantId;
};

exports.validateTenantAccess = (tenantId, resourceTenantId) => {
  // DEVE essere uguale - no cross-tenant access
  if (tenantId !== resourceTenantId) {
    throw new Error('FORBIDDEN: Cross-tenant access denied');
  }
};
```

COST ALLOCATION TAGS (OBBLIGATORI):
| Tag Key        | Value             | Scopo                    |
|----------------|-------------------|--------------------------|
| tenant_id      | {tenant_id}       | Cost attribution         |
| tier           | basic/pro/ent     | Tier tracking            |
| environment    | dev/prod          | Environment separation   |
| project        | {project}         | Project tracking         |


================================================================================
3.13 ARCH-013: SERVERLESS MICROSERVICES (DEFAULT)
================================================================================

PREREQUISITI:
- Nessuno specifico

STACK OBBLIGATORIO (14 risorse):
┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: API                                                                   │
├──────────────────────────────────────────────────────────────────────────────┤
│ 1. API Gateway (HTTP API)                                                    │
│    Nome: {project}-{env}-api                                                 │
│    Protocol: HTTP (NON REST - costa meno)                                    │
│    Throttling: 1000 burst, 500 rate                                          │
│    CORS: Enabled                                                             │
│    Logging: Enabled → CloudWatch                                             │
│                                                                              │
│ 2-5. Lambda Functions (microservices)                                        │
│    Pattern nome: {project}-{env}-{service}-handler                           │
│    Runtime: nodejs20.x (DEFAULT) o python3.12                                │
│    Memory: 512 MB (DEFAULT - aumenta se necessario)                          │
│    Timeout: 30 seconds (DEFAULT - aumenta se necessario)                     │
│    Reserved concurrency: NON impostare (scaling automatico)                  │
│    Provisioned concurrency: SOLO se cold start critico                       │
│                                                                              │
│    Architettura microservizi OBBLIGATORIA:                                   │
│    - 1 Lambda = 1 bounded context                                            │
│    - NO Lambda che chiama Lambda (usa SQS o Step Functions)                  │
│    - Idempotent (DynamoDB condition expressions)                             │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: DATA                                                                  │
├──────────────────────────────────────────────────────────────────────────────┤
│ 6. DynamoDB Table                                                            │
│    Nome: {project}-{env}-data                                                │
│    Single-table design: TRUE                                                 │
│    Partition key: PK (S)                                                     │
│    Sort key: SK (S)                                                          │
│    Billing: PAY_PER_REQUEST                                                  │
│    GSI1: GSI1PK-GSI1SK (per access patterns)                                 │
│    Stream: NEW_AND_OLD_IMAGES (se event-driven)                              │
│                                                                              │
│    Item pattern (single-table):                                              │
│    | PK              | SK               | Type   | Data...      |            │
│    |-----------------|------------------|--------|--------------|            │
│    | USER#123        | PROFILE          | user   | name, email  |            │
│    | USER#123        | ORDER#456        | order  | total, items |            │
│    | ORDER#456       | ORDER#456        | order  | full details |            │
│                                                                              │
│ 7. S3 Bucket (se file storage necessario)                                    │
│    Nome: {project}-{env}-storage                                             │
│    Versioning: Enabled                                                       │
│    Encryption: AES256                                                        │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: ASYNC PROCESSING                                                      │
├──────────────────────────────────────────────────────────────────────────────┤
│ 8. SQS Queue                                                                 │
│    Nome: {project}-{env}-processing-queue                                    │
│    Type: Standard (FIFO solo se order critical)                              │
│    Visibility timeout: 6x Lambda timeout                                     │
│    DLQ: {project}-{env}-processing-dlq                                       │
│    Max receive: 3                                                            │
│                                                                              │
│ 9. SQS DLQ                                                                   │
│    Nome: {project}-{env}-processing-dlq                                      │
│    Retention: 14 days                                                        │
│    Alarm: MessageVisible > 0 → SNS alert                                     │
│                                                                              │
│ 10. EventBridge Rule (se event-driven)                                       │
│     Nome: {project}-{env}-events                                             │
│     Event bus: default (o custom)                                            │
│     Pattern: source-based routing                                            │
│     Targets: SQS (per durability), Lambda (per low-latency)                  │
│                                                                              │
│ 11. Step Functions (se workflow)                                             │
│     Nome: {project}-{env}-workflow                                           │
│     Type: EXPRESS (se < 5 min, > 100K exec/month)                            │
│           STANDARD (se > 5 min o audit required)                             │
│     Logging: ALL                                                             │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│ LAYER: SECURITY & MONITORING                                                 │
├──────────────────────────────────────────────────────────────────────────────┤
│ 12. Cognito User Pool (se auth)                                              │
│     Nome: {project}-{env}-users                                              │
│     MFA: OPTIONAL                                                            │
│                                                                              │
│ 13. CloudWatch Dashboard                                                     │
│     Nome: {project}-{env}-dashboard                                          │
│     Widgets OBBLIGATORI:                                                     │
│       - Lambda invocations (all functions)                                   │
│       - Lambda errors (all functions)                                        │
│       - Lambda duration (p99)                                                │
│       - DynamoDB consumed capacity                                           │
│       - SQS visible messages                                                 │
│       - API Gateway 4xx/5xx                                                  │
│                                                                              │
│ 14. X-Ray                                                                    │
│     Tracing: Active on API Gateway + Lambda                                  │
│     Sampling: 5% (production)                                                │
└──────────────────────────────────────────────────────────────────────────────┘

ANTI-PATTERNS DA EVITARE (CHECK LIST):
┌──────────────────────────────────────────────────────────────────────────────┐
│ ❌ ERRORE                              │ ✅ CORREZIONE                        │
├────────────────────────────────────────┼─────────────────────────────────────┤
│ Lambda A chiama Lambda B sync          │ Usa SQS tra Lambda A e B            │
│ Lambda A chiama Lambda B sync          │ Usa Step Functions                  │
│ Monolithic Lambda (>1000 righe)        │ Split in più Lambda per context     │
│ S3 trigger Lambda → scrive su S3       │ Usa prefix/suffix diversi           │
│ Batch processing in Lambda             │ Usa Kinesis/SQS per eventi singoli  │
│ Nessun DLQ su SQS                      │ SEMPRE configurare DLQ              │
│ Nessun idempotency key                 │ Usa DynamoDB conditional writes     │
│ Lambda timeout = default (3s)          │ Imposta timeout appropriato         │
│ Log non strutturati                    │ Usa JSON structured logging         │
└──────────────────────────────────────────────────────────────────────────────┘

================================================================================
3.12 ARCH-012: DISASTER RECOVERY
================================================================================

DECISION TREE DR STRATEGY:
┌──────────────────────────────────────────────────────────────────────────────┐
│ INPUT: RTO_required (minuti), RPO_required (minuti), budget_monthly ($)      │
│                                                                              │
│ SE RTO_required <= 1 AND RPO_required <= 1:                                  │
│    → ACTIVE/ACTIVE (multi-region)                                            │
│    → Costo: 200% del primary                                                 │
│                                                                              │
│ SE RTO_required <= 30 AND RPO_required <= 5:                                 │
│    → WARM STANDBY                                                            │
│    → Costo: 50-80% del primary                                               │
│                                                                              │
│ SE RTO_required <= 240 AND RPO_required <= 60:                               │
│    → PILOT LIGHT                                                             │
│    → Costo: 10-30% del primary                                               │
│                                                                              │
│ SE RTO_required > 240 OR budget_limited = TRUE:                              │
│    → BACKUP & RESTORE                                                        │
│    → Costo: 5-10% del primary                                                │
└──────────────────────────────────────────────────────────────────────────────┘

BACKUP & RESTORE STACK:
┌──────────────────────────────────────────────────────────────────────────────┐
│ SERVIZI OBBLIGATORI:                                                         │
│                                                                              │
│ 1. AWS Backup Vault                                                          │
│    Nome: {project}-{env}-backup-vault                                        │
│    KMS: Customer managed key                                                 │
│    Cross-region copy: TRUE (verso DR region)                                 │
│                                                                              │
│ 2. AWS Backup Plan                                                           │
│    Nome: {project}-{env}-backup-plan                                         │
│    Schedule: cron(0 5 * * ? *) - daily alle 5 AM                             │
│    Retention: 35 days                                                        │
│    Lifecycle: Cold storage dopo 7 days                                       │
│    Resources: Tag-based (backup=true)                                        │
│                                                                              │
│ 3. S3 Replication                                                            │
│    Source: {project}-{env}-data (primary region)                             │
│    Destination: {project}-{env}-data-dr (DR region)                          │
│    Replication: Entire bucket                                                │
│    RTC: Enabled (15 min SLA)                                                 │
│                                                                              │
│ 4. CloudFormation Templates                                                  │
│    Location: S3 bucket (versioned, replicated)                               │
│    Templates per: VPC, ECS, RDS, Lambda, API Gateway                         │
│    Parametrizzati per region                                                 │
│                                                                              │
│ RPO: 24 hours (backup frequency)                                             │
│ RTO: 4-24 hours (infrastructure rebuild time)                                │
└──────────────────────────────────────────────────────────────────────────────┘

PILOT LIGHT STACK:
┌──────────────────────────────────────────────────────────────────────────────┐
│ PRIMARY REGION: Tutto attivo                                                 │
│ DR REGION: Solo data replication attiva                                      │
│                                                                              │
│ 1. Aurora Global Database                                                    │
│    Primary: {primary-region}                                                 │
│    Secondary: {dr-region} (read-only)                                        │
│    Replication lag: < 1 second tipico                                        │
│    Failover: Promuovi secondary a primary                                    │
│                                                                              │
│ 2. DynamoDB Global Tables                                                    │
│    Regions: [{primary-region}, {dr-region}]                                  │
│    Replication: Automatic, multi-master                                      │
│                                                                              │
│ 3. S3 Cross-Region Replication                                               │
│    (come Backup & Restore)                                                   │
│                                                                              │
│ 4. Route 53 Health Checks                                                    │
│    Nome: {project}-{env}-primary-health                                      │
│    Type: HTTPS                                                               │
│    Endpoint: Primary region ALB                                              │
│    Failure threshold: 3                                                      │
│    Interval: 30 seconds                                                      │
│                                                                              │
│ 5. Route 53 Failover Record                                                  │
│    Primary: ALB primary region (health check attached)                       │
│    Secondary: DR region endpoint (failover)                                  │
│                                                                              │
│ FAILOVER PROCEDURE:                                                          │
│ 1. Health check fails (automatic detection)                                  │
│ 2. Trigger CloudFormation in DR region                                       │
│ 3. Promote Aurora secondary                                                  │
│ 4. Route 53 automatic failover                                               │
│                                                                              │
│ RPO: < 5 minutes (replication lag)                                           │
│ RTO: 30-60 minutes (infrastructure spin-up)                                  │
└──────────────────────────────────────────────────────────────────────────────┘

WARM STANDBY STACK:
┌──────────────────────────────────────────────────────────────────────────────┐
│ DR REGION: Infrastruttura attiva ma scaled-down                              │
│                                                                              │
│ Come Pilot Light, PIÙ:                                                       │
│                                                                              │
│ 1. ECS Service (DR region)                                                   │
│    Desired count: 1 (minimo, scaled down)                                    │
│    Auto Scaling: Pronto a scalare                                            │
│                                                                              │
│ 2. ALB (DR region)                                                           │
│    Attivo, healthy                                                           │
│                                                                              │
│ 3. Lambda Functions (DR region)                                              │
│    Deployed, pronte (no cold start penalty)                                  │
│                                                                              │
│ FAILOVER: Solo scale-up (no provisioning)                                    │
│                                                                              │
│ RPO: < 1 minute (synchronous replication)                                    │
│ RTO: 5-15 minutes (scale-up time)                                            │
└──────────────────────────────────────────────────────────────────────────────┘

ACTIVE/ACTIVE STACK:
┌──────────────────────────────────────────────────────────────────────────────┐
│ ENTRAMBE LE REGION: Full capacity, serving traffic                           │
│                                                                              │
│ 1. Route 53 Latency Routing                                                  │
│    Record: A record per region                                               │
│    Routing: Latency-based                                                    │
│    Health checks: Per region                                                 │
│                                                                              │
│ 2. Global Accelerator (alternativa)                                          │
│    Endpoints: ALB per region                                                 │
│    Traffic dial: 50/50 (o custom)                                            │
│    Health checks: Built-in                                                   │
│                                                                              │
│ 3. DynamoDB Global Tables                                                    │
│    Multi-master writes                                                       │
│    Conflict resolution: Last writer wins                                     │
│                                                                              │
│ 4. Aurora Global Database                                                    │
│    Write forwarding: Enabled                                                 │
│    O: Application-level write routing                                        │
│                                                                              │
│ DATA CONSISTENCY PATTERN:                                                    │
│ - Usa partition key che include region hint                                  │
│ - Oppure: Singola write region, read da tutte                                │
│                                                                              │
│ RPO: Near-zero                                                               │
│ RTO: Near-zero (automatic failover)                                          │
└──────────────────────────────────────────────────────────────────────────────┘


================================================================================
SEZIONE 4: CHECKLIST DI VERIFICA DETERMINISTICA
================================================================================

ISTRUZIONI: ESEGUI questa checklist DOPO aver creato ogni risorsa.
SE una verifica FALLISCE → CORREGGI prima di procedere.
NON procedere con verifiche FALSE.

================================================================================
4.1 CHECKLIST VPC (OBBLIGATORIA)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | VPC creato con CIDR corretto                     | aws ec2 describe-vpcs --vpc-ids {vpc-id}                 | CIDR match|
| 2 | DNS hostnames enabled                            | aws ec2 describe-vpc-attribute --vpc-id {vpc-id} --attribute enableDnsHostnames | true |
| 3 | DNS resolution enabled                           | aws ec2 describe-vpc-attribute --vpc-id {vpc-id} --attribute enableDnsSupport | true |
| 4 | Almeno 2 public subnets                          | aws ec2 describe-subnets --filters Name=vpc-id,Values={vpc-id} | count >= 2 public |
| 5 | Almeno 2 private subnets                         | aws ec2 describe-subnets --filters Name=vpc-id,Values={vpc-id} | count >= 2 private |
| 6 | Internet Gateway attached                        | aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values={vpc-id} | attached |
| 7 | NAT Gateway per ogni AZ privata                  | aws ec2 describe-nat-gateways --filter Name=vpc-id,Values={vpc-id} | count >= 2 |
| 8 | Route table public → IGW                         | aws ec2 describe-route-tables --route-table-id {rtb-id} | 0.0.0.0/0 → igw |
| 9 | Route table private → NAT                        | aws ec2 describe-route-tables --route-table-id {rtb-id} | 0.0.0.0/0 → nat |
| 10| Flow logs enabled                                | aws ec2 describe-flow-logs --filter Name=resource-id,Values={vpc-id} | ACTIVE |

================================================================================
4.2 CHECKLIST SECURITY (OBBLIGATORIA - TUTTI I PROGETTI)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | CloudTrail enabled                               | aws cloudtrail describe-trails                           | IsMultiRegionTrail=true |
| 2 | CloudTrail log validation                        | aws cloudtrail describe-trails                           | LogFileValidationEnabled=true |
| 3 | GuardDuty enabled                                | aws guardduty list-detectors                             | count >= 1 |
| 4 | Config recording                                 | aws configservice describe-configuration-recorders       | recording=true |
| 5 | S3 public access blocked                         | aws s3api get-public-access-block --bucket {bucket}      | BlockAll=true |
| 6 | KMS key rotation                                 | aws kms get-key-rotation-status --key-id {key-id}        | KeyRotationEnabled=true |
| 7 | No security groups with 0.0.0.0/0 ingress        | aws ec2 describe-security-groups --query 'SecurityGroups[*].IpPermissions[?IpRanges[?CidrIp==`0.0.0.0/0`]]' | solo 80,443 |
| 8 | IAM password policy                              | aws iam get-account-password-policy                      | MinimumPasswordLength>=12 |
| 9 | Root account MFA                                 | aws iam get-account-summary                              | AccountMFAEnabled=1 |
| 10| No access keys for root                          | aws iam get-account-summary                              | AccountAccessKeysPresent=0 |

================================================================================
4.3 CHECKLIST LAMBDA (per ogni funzione)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | Timeout configurato                              | aws lambda get-function --function-name {fn}             | Timeout != 3 (default) |
| 2 | Memory configurato                               | aws lambda get-function --function-name {fn}             | MemorySize >= 512 |
| 3 | DLQ configurato (se async)                       | aws lambda get-function --function-name {fn}             | DeadLetterConfig present |
| 4 | X-Ray tracing                                    | aws lambda get-function --function-name {fn}             | TracingConfig.Mode=Active |
| 5 | Environment encryption                           | aws lambda get-function --function-name {fn}             | KMSKeyArn present |
| 6 | VPC config (se necessario)                       | aws lambda get-function --function-name {fn}             | VpcConfig.SubnetIds not empty |
| 7 | Reserved concurrency (se critico)                | aws lambda get-function-concurrency --function-name {fn} | configured |

================================================================================
4.4 CHECKLIST DYNAMODB (per ogni tabella)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | Billing mode corretto                            | aws dynamodb describe-table --table-name {table}         | PAY_PER_REQUEST o PROVISIONED |
| 2 | PITR enabled                                     | aws dynamodb describe-continuous-backups --table-name {table} | PointInTimeRecoveryStatus=ENABLED |
| 3 | Encryption                                       | aws dynamodb describe-table --table-name {table}         | SSEDescription.Status=ENABLED |
| 4 | Stream enabled (se event-driven)                 | aws dynamodb describe-table --table-name {table}         | StreamSpecification present |
| 5 | TTL configured (se necessario)                   | aws dynamodb describe-time-to-live --table-name {table}  | TimeToLiveStatus=ENABLED |
| 6 | GSI definiti                                     | aws dynamodb describe-table --table-name {table}         | GlobalSecondaryIndexes present |

================================================================================
4.5 CHECKLIST RDS/AURORA (per ogni istanza)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | Multi-AZ (production)                            | aws rds describe-db-instances --db-instance-id {id}      | MultiAZ=true |
| 2 | Encryption at rest                               | aws rds describe-db-instances --db-instance-id {id}      | StorageEncrypted=true |
| 3 | Automated backups                                | aws rds describe-db-instances --db-instance-id {id}      | BackupRetentionPeriod >= 7 |
| 4 | Deletion protection                              | aws rds describe-db-instances --db-instance-id {id}      | DeletionProtection=true |
| 5 | Performance Insights                             | aws rds describe-db-instances --db-instance-id {id}      | PerformanceInsightsEnabled=true |
| 6 | Enhanced monitoring                              | aws rds describe-db-instances --db-instance-id {id}      | MonitoringInterval=60 |
| 7 | Private subnet only                              | aws rds describe-db-instances --db-instance-id {id}      | PubliclyAccessible=false |
| 8 | Security group restricted                        | Verify SG allows only app layer                          | No 0.0.0.0/0 |

================================================================================
4.6 CHECKLIST S3 (per ogni bucket)
================================================================================

| # | Verifica                                         | Comando AWS CLI                                          | Expected |
|---|--------------------------------------------------|----------------------------------------------------------|----------|
| 1 | Versioning enabled                               | aws s3api get-bucket-versioning --bucket {bucket}        | Status=Enabled |
| 2 | Public access blocked                            | aws s3api get-public-access-block --bucket {bucket}      | All=true |
| 3 | Encryption default                               | aws s3api get-bucket-encryption --bucket {bucket}        | SSEAlgorithm present |
| 4 | Logging enabled (se sensitive)                   | aws s3api get-bucket-logging --bucket {bucket}           | LoggingEnabled present |
| 5 | Lifecycle rules (se applicable)                  | aws s3api get-bucket-lifecycle-configuration --bucket {bucket} | Rules present |

================================================================================
SEZIONE 5: NAMING CONVENTION OBBLIGATORIA
================================================================================

PATTERN: {project}-{environment}-{service}-{resource}

| Componente   | Valori Ammessi                    | Esempio                        |
|--------------|-----------------------------------|--------------------------------|
| project      | [a-z0-9-], max 20 chars           | myapp, ecommerce, healthsys    |
| environment  | dev, staging, prod                | dev, prod                      |
| service      | Nome servizio, max 15 chars       | api, auth, orders, payments    |
| resource     | Tipo risorsa AWS                  | lambda, table, bucket, sg      |

ESEMPI CORRETTI:
- Lambda: myapp-prod-orders-handler
- DynamoDB: myapp-prod-orders-table
- S3: myapp-prod-media-bucket
- SG: myapp-prod-api-sg
- IAM Role: myapp-prod-lambda-execution-role

ESEMPI ERRATI:
- MyApp_Prod_Orders (underscore, maiuscole)
- myapp-orders (manca environment)
- production-lambda (manca project, service)

================================================================================
SEZIONE 6: COST ESTIMATION LOOKUP
================================================================================

TABELLA COSTI MENSILI STIMATI (eu-south-1, USD):

| Risorsa                  | Configurazione          | Costo/mese |
|--------------------------|-------------------------|------------|
| Lambda                   | 1M invocations, 512MB   | $8.35      |
| Lambda                   | 10M invocations, 512MB  | $63.50     |
| API Gateway HTTP         | 1M requests             | $1.00      |
| API Gateway REST         | 1M requests             | $3.50      |
| DynamoDB On-Demand       | 1M reads, 1M writes     | $1.50      |
| DynamoDB Provisioned     | 25 RCU, 25 WCU          | $14.04     |
| Aurora Serverless v2     | 0.5 ACU baseline        | $43.80     |
| Aurora Provisioned       | db.r6g.large            | $175.20    |
| RDS PostgreSQL           | db.t3.micro             | $12.41     |
| RDS PostgreSQL           | db.r6g.large            | $138.70    |
| ElastiCache              | cache.t3.micro          | $12.41     |
| ElastiCache              | cache.r6g.large         | $119.46    |
| S3 Standard              | 100 GB                  | $2.40      |
| S3 Standard              | 1 TB                    | $24.00     |
| CloudFront               | 100 GB transfer         | $8.50      |
| NAT Gateway              | 1 (processing)          | $32.85     |
| ALB                      | Base + LCU              | $22.27+    |
| EKS Cluster              | Control plane           | $73.00     |
| Fargate                  | 0.5 vCPU, 1GB, 24/7     | $14.83     |
| CloudWatch Logs          | 10 GB ingestion         | $5.00      |
| Secrets Manager          | 1 secret                | $0.40      |
| KMS CMK                  | 1 key                   | $1.00      |

FORMULA STIMA RAPIDA:
- Serverless (Lambda+DynamoDB+API GW): $50-200/mese per 1M MAU
- Container (ECS/EKS+RDS): $300-800/mese per applicazione base
- Full production (multi-AZ, DR): 2-3x costo base

================================================================================
SEZIONE 7: ERRORI COMUNI E CORREZIONI
================================================================================

| ERRORE                                      | CONSEGUENZA                  | CORREZIONE                           |
|---------------------------------------------|------------------------------|--------------------------------------|
| S3 bucket public                            | Data breach                  | Block all public access              |
| Security group 0.0.0.0/0 su porta DB        | Database esposto             | Solo IP/SG specifici                 |
| Lambda senza timeout customizzato           | Timeout 3s, errori           | Imposta timeout appropriato          |
| DynamoDB senza PITR                         | Data loss non recuperabile   | Abilita PITR                         |
| RDS senza Multi-AZ in prod                  | Downtime su failure          | Abilita Multi-AZ                     |
| CloudTrail non abilitato                    | No audit trail               | Abilita con log validation           |
| Nessun DLQ su SQS/Lambda                    | Messaggi persi               | Configura sempre DLQ                 |
| VPC senza NAT Gateway                       | Lambda/ECS senza internet    | Aggiungi NAT per ogni AZ             |
| API Gateway senza throttling                | DDoS vulnerability           | Configura rate limiting              |
| Secrets hardcoded in codice                 | Security breach              | Usa Secrets Manager                  |

================================================================================
FINE CATALOGO DETERMINISTICO v2.0
================================================================================

CHANGELOG:
- v2.0 (2026-01-25): Versione deterministica completa
  - Decision trees binari
  - Lookup tables esatte
  - Checklist di verifica
  - Naming convention obbligatoria
  - Costi esatti
  - Zero interpretazione

USAGE:
1. LEGGI requisiti
2. ESEGUI SEZIONE 1 → ottieni ARCH-ID
3. VAI A SEZIONE 3 → copia stack ESATTO
4. USA SEZIONE 2 per scelte servizi specifici
5. VERIFICA con SEZIONE 4 checklist
6. STIMA costi con SEZIONE 6

