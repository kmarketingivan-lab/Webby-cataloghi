# CATALOGO REQUISITI → ARCHITETTURA AWS

## META
- **Versione**: 1.0
- **Data**: 2026-01-25
- **Autore**: Ivan (AWS Blueprint Automation)
- **Scopo**: Mappatura deterministica REQUISITI → ARCHITETTURA AWS

---

## 1. INDICE DEI PATTERN ARCHITETTURALI

| ID | Pattern | Requisiti Chiave | RTO/RPO | Costo Relativo |
|----|---------|------------------|---------|----------------|
| ARCH-001 | E-Commerce Serverless | High traffic, scalability, payments | < 1min / < 5min | $$ |
| ARCH-002 | Social Media Platform | Real-time, chat, notifications | < 1min / < 1min | $$$ |
| ARCH-003 | Video Streaming (Netflix-scale) | Global CDN, adaptive bitrate | < 30s / near-zero | $$$$ |
| ARCH-004 | Gaming Multiplayer | Low latency, matchmaking | < 1s / < 1s | $$$ |
| ARCH-005 | Healthcare HIPAA | Compliance, encryption, audit | < 4h / < 15min | $$ |
| ARCH-006 | FinTech PCI-DSS | Payment security, compliance | < 15min / < 5min | $$$ |
| ARCH-007 | IoT Industrial (IIoT) | Edge computing, telemetry | < 1min / near-zero | $$ |
| ARCH-008 | IoT Smart Home | Device management, voice | < 5min / < 1min | $ |
| ARCH-009 | SaaS Multi-Tenant | Tenant isolation, cost tracking | < 5min / < 5min | $$ |
| ARCH-010 | Data Lake/Lakehouse | Analytics, ML, governance | < 1h / < 15min | $$ |
| ARCH-011 | MLOps Pipeline | Training, inference, monitoring | < 30min / < 5min | $$$ |
| ARCH-012 | Disaster Recovery | Business continuity | Variable | Variable |
| ARCH-013 | Serverless Microservices | Event-driven, decoupling | < 1min / < 5min | $ |
| ARCH-014 | Container Orchestration (EKS) | Kubernetes, hybrid | < 5min / < 5min | $$ |

---

## 2. CATALOGO SERVIZI AWS PER CATEGORIA

### 2.1 COMPUTE
| Servizio | Caso d'Uso | Pricing Model |
|----------|------------|---------------|
| **EC2** | Workloads tradizionali, full control | On-Demand, Reserved, Spot |
| **Lambda** | Event-driven, serverless, < 15 min | Per invocation + duration |
| **Fargate** | Containers serverless | Per vCPU + memory |
| **ECS** | Container orchestration (AWS native) | Cluster + resources |
| **EKS** | Kubernetes managed | $0.10/hour + nodes |
| **Elastic Beanstalk** | PaaS, quick deploy | Underlying resources |
| **Lightsail** | VPS semplificato | Fixed monthly |
| **Batch** | Batch computing jobs | Underlying resources |
| **App Runner** | Container web apps | Per vCPU + memory |

### 2.2 STORAGE
| Servizio | Caso d'Uso | Durability |
|----------|------------|------------|
| **S3** | Object storage, data lake | 99.999999999% (11 9s) |
| **S3 Glacier** | Archival, compliance | 99.999999999% |
| **EBS** | Block storage per EC2 | 99.999% |
| **EFS** | Shared file system (NFS) | 99.999999999% |
| **FSx** | Windows/Lustre/NetApp/OpenZFS | 99.999999999% |
| **Storage Gateway** | Hybrid cloud storage | Underlying |

### 2.3 DATABASE
| Servizio | Tipo | Caso d'Uso |
|----------|------|------------|
| **RDS** | Relational (MySQL, PostgreSQL, Oracle, SQL Server) | OLTP, transactional |
| **Aurora** | MySQL/PostgreSQL compatible | High performance, 75% cost savings |
| **Aurora Serverless** | Auto-scaling relational | Variable workloads |
| **DynamoDB** | NoSQL key-value | High throughput, low latency |
| **DynamoDB DAX** | DynamoDB cache | Microsecond latency |
| **ElastiCache** | Redis/Memcached | Session, caching |
| **Neptune** | Graph database | Social networks, fraud |
| **Timestream** | Time series | IoT, telemetry |
| **QLDB** | Ledger (immutable) | Audit trails, blockchain |
| **DocumentDB** | MongoDB compatible | Document storage |
| **Keyspaces** | Cassandra compatible | Wide-column |
| **MemoryDB** | Redis durable | Ultra-fast + durable |
| **Redshift** | Data warehouse | OLAP, analytics |
| **OpenSearch** | Search & analytics | Full-text search, logs |

### 2.4 NETWORKING
| Servizio | Caso d'Uso |
|----------|------------|
| **VPC** | Isolated virtual network |
| **CloudFront** | Global CDN, edge caching |
| **Route 53** | DNS, health checks, routing policies |
| **API Gateway** | REST/HTTP/WebSocket APIs |
| **Global Accelerator** | TCP/UDP optimization (60% perf improvement) |
| **Transit Gateway** | Multi-VPC hub |
| **Direct Connect** | Dedicated connection |
| **VPN** | Site-to-site, client VPN |
| **PrivateLink** | Private service access |
| **App Mesh** | Service mesh |
| **Cloud Map** | Service discovery |

### 2.5 SECURITY & IDENTITY
| Servizio | Caso d'Uso |
|----------|------------|
| **IAM** | Identity, access control |
| **Cognito** | User authentication, OAuth, SAML |
| **KMS** | Encryption key management |
| **Secrets Manager** | Secrets rotation |
| **WAF** | Web application firewall |
| **Shield** | DDoS protection |
| **GuardDuty** | Threat detection |
| **Inspector** | Vulnerability assessment |
| **Macie** | Data security (ML-powered) |
| **Security Hub** | Unified security dashboard |
| **CloudHSM** | Hardware security module |
| **Certificate Manager** | SSL/TLS certificates |

### 2.6 MESSAGING & INTEGRATION
| Servizio | Pattern |
|----------|---------|
| **SQS** | Queue (decoupling, async) |
| **SNS** | Pub/Sub (fan-out) |
| **EventBridge** | Event bus (routing, filtering) |
| **MQ** | ActiveMQ/RabbitMQ managed |
| **Kinesis Data Streams** | Real-time streaming |
| **Kinesis Firehose** | Streaming ETL to destinations |
| **MSK** | Apache Kafka managed |
| **Step Functions** | Workflow orchestration |
| **AppSync** | GraphQL managed |

### 2.7 ANALYTICS
| Servizio | Caso d'Uso |
|----------|------------|
| **Athena** | SQL queries on S3 (serverless) |
| **Redshift** | Data warehouse |
| **EMR** | Hadoop/Spark clusters |
| **Glue** | ETL serverless, Data Catalog |
| **Lake Formation** | Data lake governance |
| **QuickSight** | Business intelligence |
| **Data Exchange** | Third-party data marketplace |
| **Kinesis Analytics** | Real-time stream processing |

### 2.8 AI/ML
| Servizio | Caso d'Uso |
|----------|------------|
| **SageMaker** | ML platform (train, deploy, monitor) |
| **SageMaker Pipelines** | MLOps automation |
| **Bedrock** | Foundation models, GenAI |
| **Comprehend** | NLP, sentiment analysis |
| **Comprehend Medical** | Healthcare NLP |
| **Rekognition** | Image/video analysis |
| **Textract** | Document extraction |
| **Transcribe** | Speech-to-text |
| **Polly** | Text-to-speech |
| **Translate** | Language translation |
| **Personalize** | Recommendations |
| **Forecast** | Time series forecasting |
| **Lex** | Chatbots |
| **Kendra** | Enterprise search |

### 2.9 IoT
| Servizio | Caso d'Uso |
|----------|------------|
| **IoT Core** | Device connectivity, MQTT |
| **IoT Greengrass** | Edge computing |
| **IoT SiteWise** | Industrial data collection |
| **IoT Analytics** | IoT data analytics |
| **IoT Events** | Event detection |
| **IoT Device Defender** | Security monitoring |
| **IoT Device Management** | Fleet management |
| **IoT TwinMaker** | Digital twins |

### 2.10 DEVELOPER TOOLS
| Servizio | Caso d'Uso |
|----------|------------|
| **CodeCommit** | Git repository |
| **CodeBuild** | Build service |
| **CodeDeploy** | Deployment automation |
| **CodePipeline** | CI/CD orchestration |
| **CloudFormation** | Infrastructure as Code |
| **CDK** | IaC with programming languages |
| **SAM** | Serverless Application Model |
| **Amplify** | Full-stack development |
| **X-Ray** | Distributed tracing |
| **CodeArtifact** | Package repository |


---

## 3. PATTERN ARCHITETTURALI DETTAGLIATI

### ARCH-001: E-COMMERCE SERVERLESS
```
REQUISITI:
- Scalabilità automatica
- Pagamenti sicuri (PCI-DSS se carte)
- Ricerca prodotti veloce
- Raccomandazioni personalizzate
- Alta disponibilità

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                     │
│  Route 53 → CloudFront → S3 (static) + WAF + Shield                 │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         API LAYER                                    │
│  API Gateway (REST/HTTP) → Lambda Functions                         │
│  - Product Service                                                   │
│  - Cart Service                                                      │
│  - Order Service                                                     │
│  - User Service                                                      │
│  - Search Service                                                    │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                   │
│  DynamoDB (products, carts, users) + DAX (cache)                    │
│  RDS/Aurora (orders, transactions)                                  │
│  ElastiCache Redis (sessions)                                       │
│  OpenSearch (product search)                                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         INTEGRATION                                  │
│  EventBridge → SQS → Lambda (async processing)                      │
│  Step Functions (order workflow)                                    │
│  SNS (notifications)                                                │
│  SES (email)                                                        │
│  Stripe/Payment Gateway                                             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         AI/ML                                        │
│  Amazon Personalize (recommendations)                               │
│  Amazon Comprehend (reviews sentiment)                              │
│  Amazon Fraud Detector                                              │
└─────────────────────────────────────────────────────────────────────┘

SERVIZI CHIAVE:
- CloudFront + S3 + WAF: CDN + static hosting + security
- API Gateway + Lambda: Serverless APIs
- DynamoDB + DAX: NoSQL + caching
- OpenSearch: Full-text search
- Personalize: ML recommendations
- Step Functions: Order orchestration
- Cognito: User authentication

COSTI STIMATI (100K users/month):
- Lambda: ~$50-100/month
- DynamoDB: ~$50-200/month (on-demand)
- CloudFront: ~$50-100/month
- API Gateway: ~$35-70/month
- Totale: ~$200-500/month
```

---

### ARCH-002: SOCIAL MEDIA / CHAT APPLICATION
```
REQUISITI:
- Real-time messaging (WebSocket)
- Offline support
- Push notifications
- User feed personalizzato
- Media storage (images/videos)
- Analytics

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         FRONTEND                                     │
│  CloudFront → S3 (PWA) + Amplify                                    │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         REAL-TIME LAYER                              │
│  AppSync (GraphQL + WebSocket subscriptions)                        │
│  OR                                                                  │
│  API Gateway WebSocket → Lambda                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                   │
│  DynamoDB: Users, Messages, Conversations, Feeds                    │
│  ElastiCache Redis: Presence, typing indicators                     │
│  Neptune: Social graph (friends, followers)                         │
│  S3: Media storage                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         MESSAGING                                    │
│  EventBridge: Event routing                                         │
│  SQS: Message queuing                                               │
│  SNS: Push notifications (APNS, FCM)                                │
│  SES: Email notifications                                           │
│  Pinpoint: Customer engagement                                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         ANALYTICS                                    │
│  Kinesis Firehose → S3 (data lake)                                  │
│  Redshift: Data warehouse                                           │
│  QuickSight: Dashboards                                             │
│  Athena: Ad-hoc queries                                             │
└─────────────────────────────────────────────────────────────────────┘

SERVIZI CHIAVE:
- AppSync: GraphQL con subscriptions real-time
- DynamoDB: 4 tables (User, Message, Conversation, UserConversations)
- Cognito: Auth con JWT tokens
- Neptune: Graph per relazioni sociali
- SNS: Push notifications multi-platform
- Kinesis: Streaming analytics
```

---

### ARCH-003: VIDEO STREAMING (Netflix-scale)
```
REQUISITI:
- Global delivery (CDN)
- Adaptive bitrate streaming (ABR)
- Live + VOD
- DRM protection
- Personalization
- 280M+ users scale

ARCHITETTURA (Netflix Reference):
┌─────────────────────────────────────────────────────────────────────┐
│                         INGEST                                       │
│  MediaConnect (SMPTE 2022-7 redundancy)                             │
│  Dual-pipeline redundancy (2 AWS regions)                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         TRANSCODE                                    │
│  MediaLive: ABR encoding (HLS, DASH)                                │
│  Elemental MediaConvert: VOD processing                             │
│  3 encoding profiles                                                │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         PACKAGE                                      │
│  MediaPackage: HLS, DASH, CMAF                                      │
│  DRM integration                                                     │
│  3 custom endpoints                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DELIVERY                                     │
│  CloudFront (global CDN)                                            │
│  S3 (origin storage, petabytes)                                     │
│  Live Origin Service (EC2 microservice)                             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         BACKEND (Microservices)                      │
│  EC2 Auto Scaling (4 AWS Regions)                                   │
│  Aurora (75% cost savings, 28% perf improvement)                    │
│  SageMaker: ML recommendations, personalization                     │
│  Kinesis: Real-time analytics                                       │
└─────────────────────────────────────────────────────────────────────┘

SERVIZI CHIAVE:
- MediaLive + MediaPackage + CloudFront: Video pipeline
- Aurora: High-performance database
- SageMaker: ML per recommendations
- EC2 microservices: Thousands running across 4 regions

SCALA NETFLIX:
- 280M+ members
- 190+ countries
- Petabytes of multimedia assets
- 65M concurrent streams (peak)
- Cost: ~$27.78M/month (2023 estimate)
```



---

### ARCH-004: GAMING MULTIPLAYER
```
REQUISITI:
- Session-based multiplayer
- Low latency (< 50ms)
- Matchmaking intelligente
- Global deployment
- Scaling predictive (100M concurrent players)

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         CLIENT FLOW                                  │
│  1. Client → Cognito (identity + temp credentials)                  │
│  2. Client → API Gateway (signed request + latency data)            │
│  3. Lambda → DynamoDB (player skill level)                          │
│  4. Lambda → GameLift FlexMatch (matchmaking)                       │
│  5. FlexMatch → Queue → Optimal region allocation                   │
│  6. Player → Game Server (direct connection)                        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                         HOSTING LAYER                                │
│  GameLift:                                                          │
│  - Multi-region fleets (global deployment)                          │
│  - Predictive scaling: 100M concurrent, 100K adds/second            │
│  - FlexMatch: 200 players/session, custom rules                     │
│  - FleetIQ: Spot instances (70% discount)                           │
│  - GameLift Streams: 1080p 60fps streaming                          │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         INFRASTRUCTURE                               │
│  EC2: C-family, R-family, Graviton instances                        │
│  ECS/EKS: Containerized builds                                      │
│  Global Accelerator: 60% performance improvement                    │
│  DynamoDB: Player data, skill levels                                │
│  S3: Build storage                                                  │
│  CloudWatch: Monitoring                                             │
└─────────────────────────────────────────────────────────────────────┘

SERVIZI CHIAVE:
- Cognito: Player identity
- GameLift: Dedicated game servers
- FlexMatch: Smart matchmaking
- Global Accelerator: Low latency routing
- DynamoDB: Player state
```

---

### ARCH-005: HEALTHCARE HIPAA COMPLIANT
```
REQUISITI:
- HIPAA compliance (BAA con AWS)
- PHI protection (encryption at rest + in transit)
- Audit trails completi
- Access control granulare
- 166+ HIPAA-eligible services

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         VPC DESIGN                                   │
│  Multi-VPC: dev/prod separation                                     │
│  Private subnets: Infrastructure + databases                        │
│  NAT Gateways: Controlled internet access                           │
│  Security Groups + NACLs: Traffic control                           │
│  VPC Flow Logs: Auditing                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         ACCESS LAYER                                 │
│  Cognito: Patient/provider authentication                           │
│  IAM: Least privilege, temporary access                             │
│  Lake Formation: Fine-grained data permissions                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         COMPUTE                                      │
│  Lambda + ECS: Application logic                                    │
│  Fargate: Serverless containers                                     │
│  AppSync: API layer                                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA LAYER                                   │
│  HealthLake: FHIR-based data lake                                   │
│  S3: Patient files (encrypted, lifecycle policies)                  │
│  DynamoDB: Patient metadata                                         │
│  RDS/Aurora: Transactional data (encrypted)                         │
│  ALL with KMS encryption                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA INGESTION                               │
│  Protocols: HL7v2 over MLLP, FHIR web services, SFTP               │
│  Site-to-Site VPN: Encrypted legacy protocols                       │
│  Kinesis: High-volume message batching                              │
│  IoT Core: Medical device data                                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         ANALYTICS                                    │
│  Glue Crawlers: Schema discovery                                    │
│  Glue ETL: Transform + normalize                                    │
│  Athena: Ad-hoc queries                                             │
│  QuickSight: Dashboards                                             │
│  SageMaker: Fraud detection, ML models                              │
│  Comprehend Medical: Healthcare NLP                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         SECURITY & AUDIT                             │
│  CloudTrail: API logging                                            │
│  Config: Compliance monitoring                                      │
│  GuardDuty: Threat detection                                        │
│  Security Hub: Unified compliance                                   │
│  SNS: Security alerts                                               │
│  AWS Backup: Automated, cross-region                                │
└─────────────────────────────────────────────────────────────────────┘

COMPLIANCE FRAMEWORKS:
- HIPAA (1996) + HITECH (2009)
- HITRUST CSF
- FedRAMP, GDPR, ENS High, HDS, C5
- GxP (life sciences)
```

---

### ARCH-006: FINTECH PCI-DSS
```
REQUISITI:
- PCI DSS Level 1 compliance
- Cardholder data protection
- Network segmentation
- Encryption everywhere
- Audit trails

ARCHITETTURA PCI-DSS:
┌─────────────────────────────────────────────────────────────────────┐
│                         NETWORK LAYER                                │
│  VPC: Logically isolated                                            │
│  Private subnets: CDE (Cardholder Data Environment)                 │
│  Security Groups: Virtual firewall                                  │
│  NACLs: Subnet-level control                                        │
│  PrivateLink: No public internet traversal                          │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         COMPUTE                                      │
│  EKS (multi-AZ): Containerized microservices                        │
│  Network Load Balancer: Frontend                                    │
│  Fargate: Serverless containers                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PROTECTION                              │
│  KMS: All encryption keys                                           │
│  CloudHSM: Hardware security module (PCI PIN certified)             │
│  Payment Cryptography: PCI P2PE certified                           │
│  Secrets Manager: Credentials rotation                              │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         AUDIT & MONITORING                           │
│  CloudTrail: API logging (all activities)                           │
│  CloudWatch: Metrics, alarms                                        │
│  Config: Resource compliance                                        │
│  Audit Manager: PCI DSS v4.0 framework                              │
│  Security Hub: Centralized findings                                 │
└─────────────────────────────────────────────────────────────────────┘

PCI DSS REQUIREMENTS MAPPING:
1. Network Security Controls: VPC, Security Groups, NACLs, IAM
2. Secure Configurations: Systems Manager, Config
3. Protect Stored Data: KMS, encryption at rest
4. Encryption in Transit: TLS 1.2+, Certificate Manager
5. Malware Protection: GuardDuty, Inspector
6. Secure Systems: Patch Manager, Inspector
7. Access Control: IAM, Cognito
8. User Authentication: MFA, temporary credentials
9. Physical Security: AWS manages (AOC available)
10. Logging & Monitoring: CloudTrail, CloudWatch
11. Security Testing: Inspector, penetration testing
12. Security Policies: Organizations, SCPs

CASE STUDY (FinTech Company):
- Migrated UPI + Reward points to AWS
- EKS cluster (multi-AZ, private subnets)
- Network Load Balancer + PrivateLink
- 99.95% availability guarantee
- DevSecOps via AWS ECR scanning
```



---

### ARCH-007: IoT INDUSTRIAL (IIoT)
```
REQUISITI:
- Secure device connectivity
- Edge computing
- Real-time telemetry
- Predictive maintenance
- OT/IT integration

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         EDGE LAYER                                   │
│  IoT Greengrass: Edge runtime                                       │
│  IoT SiteWise Edge: OPC-UA data collection                          │
│  Local ML inference                                                  │
│  KEPServerEX (gateway): Industrial protocols                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │ MQTT
┌─────────────────────────────────────────────────────────────────────┐
│                         CLOUD INGESTION                              │
│  IoT Core: Device connectivity, rules engine                        │
│  Kinesis Data Streams: High-throughput streaming                    │
│  Timestream: Time series storage                                    │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PROCESSING                              │
│  IoT Analytics: Data analysis                                       │
│  Lambda: Event processing                                           │
│  Step Functions: Workflow orchestration                             │
│  Glue: ETL to data lake                                             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         STORAGE                                      │
│  S3: Data lake (raw, processed)                                     │
│  DynamoDB: Device metadata                                          │
│  Timestream: Time series queries                                    │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         ANALYTICS & ML                               │
│  Amazon Lookout for Equipment: Predictive maintenance               │
│  Amazon Monitron: Vibration/temperature monitoring                  │
│  SageMaker: Custom ML models                                        │
│  QuickSight: Dashboards                                             │
│  Managed Grafana: Visualization                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DIGITAL TWIN                                 │
│  IoT TwinMaker: 3D visualization                                    │
│  Connect to real-time data                                          │
│  Simulation capabilities                                            │
└─────────────────────────────────────────────────────────────────────┘

USE CASES:
- Condition-based monitoring
- Predictive maintenance (40% cost savings with Karpenter)
- Energy optimization
- Quality control
- Smart grid integration
```

---

### ARCH-008: IoT SMART HOME
```
REQUISITI:
- Device onboarding scalabile
- OTA updates
- Voice integration (Alexa)
- Data privacy
- Consumer-grade reliability

ARCHITETTURA:
┌─────────────────────────────────────────────────────────────────────┐
│                         DEVICE LAYER                                 │
│  IoT Device SDK: Secure connectivity                                │
│  IoT certificates + policies                                        │
│  MQTT/WebSocket                                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         CLOUD SERVICES                               │
│  IoT Core: Device registry, message broker                          │
│  IoT Device Management: Fleet management, OTA                       │
│  IoT Device Defender: Security monitoring                           │
│  Cognito: User authentication                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PIPELINE                                │
│  Kinesis Data Streams → Kinesis Analytics → Kinesis Firehose       │
│  DynamoDB: Device state                                             │
│  S3: Telemetry data lake                                            │
│  Timestream: Time series                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                            │
│  Lambda + Fargate: Backend services                                 │
│  API Gateway: Mobile app APIs                                       │
│  AppSync: Real-time sync                                            │
│  Amplify: Mobile app hosting                                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         AI/ML                                        │
│  SageMaker: Personalization, predictive maintenance                 │
│  Self-service support (AI-powered diagnostics)                      │
│  Anomaly detection                                                  │
└─────────────────────────────────────────────────────────────────────┘

FEATURES:
- Zero-touch onboarding
- Fleet-wide OTA updates
- Centralized data lakehouse (S3 + Glue + Athena + Redshift)
- ML personalization per device
```

---

### ARCH-009: SaaS MULTI-TENANT
```
REQUISITI:
- Tenant isolation
- Cost attribution per tenant
- Scalability
- Tiering (Basic/Premium/Enterprise)
- Noisy neighbor prevention

MODELLI DI TENANCY:
1. SILO MODEL (strongest isolation, highest cost)
   - Separate resources per tenant
   - Own Cognito User Pool, API Gateway, Lambda, DynamoDB tables
   
2. POOL MODEL (shared resources, lowest cost)
   - Shared infrastructure
   - Tenant context in JWT tokens
   - Row-level security in database
   
3. BRIDGE MODEL (hybrid)
   - Shared compute, separate databases
   - Balance isolation/cost

ARCHITETTURA SERVERLESS MULTI-TENANT:
┌─────────────────────────────────────────────────────────────────────┐
│                         ONBOARDING                                   │
│  Landing Page → API Gateway → Lambda → Cognito User Pool            │
│  CodePipeline: Tenant stack provisioning                            │
│  DynamoDB: Tenant registry                                          │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         IDENTITY                                     │
│  Cognito: Per-tenant or shared user pool                            │
│  JWT tokens: tenant_id claim                                        │
│  IAM: Scoped permissions                                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         APPLICATION LAYER                            │
│  POOLED TENANTS:                                                    │
│  - Shared API Gateway                                               │
│  - Shared Lambda functions                                          │
│  - Tenant context extracted from JWT                                │
│                                                                     │
│  SILOED TENANTS (Premium):                                          │
│  - Dedicated API Gateway                                            │
│  - Dedicated Lambda functions                                       │
│  - Dedicated DynamoDB tables                                        │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PARTITIONING                            │
│  POOLED: Single table, tenant_id partition key                      │
│  SILOED: Separate tables per tenant                                 │
│  HYBRID: Shared for most, dedicated for premium                     │
│                                                                     │
│  Aurora Serverless: Variable workloads                              │
│  DynamoDB: Schema-less, per-tenant billing                          │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         OPERATIONS                                   │
│  CloudWatch: Tenant-aware metrics                                   │
│  Lambda Layers: Shared tenant context extraction                    │
│  Cost allocation tags: Per-tenant billing                           │
│  CodePipeline: Multi-tenant CI/CD                                   │
└─────────────────────────────────────────────────────────────────────┘

EKS MULTI-TENANT:
- Kubernetes namespaces per tenant
- NGINX Ingress Controller
- Network policies for isolation
- IRSA (IAM Roles for Service Accounts)
- Karpenter: Auto-scaling (40% cost savings)
```



---

### ARCH-010: DATA LAKE / LAKEHOUSE
```
REQUISITI:
- Unified data repository
- Multiple data sources
- Governance & security
- Analytics & ML ready
- Apache Iceberg compatibility

ARCHITETTURA MODERNA (6 LAYERS):
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 1: INGESTION                                                   │
│  - AWS DMS: Database migration                                      │
│  - DataSync: File transfer                                          │
│  - Kinesis/MSK: Streaming                                           │
│  - IoT Core: Device data                                            │
│  - AppFlow: SaaS integration                                        │
│  - Transfer Family: SFTP/FTPS                                       │
│  - Data Exchange: Third-party data                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 2: STORAGE (Medallion Architecture)                            │
│  RAW (Bronze): Original format, no transformation                   │
│  CURATED (Silver): Cleaned, validated                               │
│  REFINED (Gold): Business-ready, aggregated                         │
│                                                                     │
│  S3: Primary storage (11 9s durability)                             │
│  S3 Tables: Apache Iceberg native                                   │
│  Redshift Managed Storage: Data warehouse                           │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 3: CATALOG & GOVERNANCE                                        │
│  Glue Data Catalog: Metadata repository                             │
│  Lake Formation: Unified governance                                 │
│  - Fine-grained access (table/column level)                         │
│  - Cross-account sharing                                            │
│  - Audit trails                                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 4: PROCESSING                                                  │
│  Glue ETL: Serverless Spark                                         │
│  EMR: Hadoop/Spark clusters                                         │
│  Step Functions: Orchestration                                      │
│  Kinesis Analytics: Real-time processing                            │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 5: CONSUMPTION                                                 │
│  Athena: Serverless SQL queries                                     │
│  Redshift/Redshift Spectrum: Data warehouse + S3 queries            │
│  QuickSight: BI dashboards                                          │
│  SageMaker: ML training                                             │
│  OpenSearch: Search & operational analytics                         │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│ LAYER 6: SECURITY & MONITORING                                       │
│  IAM + Lake Formation: Access control                               │
│  KMS: Encryption                                                    │
│  CloudTrail + CloudWatch: Audit & monitoring                        │
│  GuardDuty: Threat detection                                        │
└─────────────────────────────────────────────────────────────────────┘

OPEN TABLE FORMATS:
- Apache Iceberg (recommended by AWS)
- Apache Hudi
- Delta Lake
- Benefits: ACID transactions, schema evolution, time travel

SAGEMAKER LAKEHOUSE (2024+):
- Unified analytics + AI experience
- Apache Iceberg native
- Federated sources (BigQuery, Snowflake)
- SageMaker Unified Studio
```

---

### ARCH-011: MLOps PIPELINE
```
REQUISITI:
- End-to-end ML lifecycle
- CI/CD/CT (Continuous Training)
- Model versioning & registry
- Monitoring & drift detection
- Multi-account governance

ARCHITETTURA MLOps (Multi-Account):
┌─────────────────────────────────────────────────────────────────────┐
│ DATA ACCOUNT                                                         │
│  S3: Training datasets                                              │
│  Lake Formation: Governance                                         │
│  Glue Catalog: Metadata                                             │
└─────────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│ ML EXECUTION ACCOUNT (DEV)                                           │
│  SageMaker Studio: Notebook IDE                                     │
│  Data Wrangler: 300+ built-in transformations                       │
│  Feature Store (Offline): Historical features                       │
│  SageMaker Pipelines: ML workflows as DAG                           │
│  - Data preprocessing                                               │
│  - Training                                                         │
│  - Evaluation                                                       │
│  - Bias detection (Clarify)                                         │
│  - Model registration                                               │
└─────────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│ CI/CD BLOCK                                                          │
│  CodeCommit: Model code repository                                  │
│  CodePipeline: Orchestration                                        │
│  CodeBuild: Build & test                                            │
│  Step Functions: ML-related tests                                   │
│  Model Registry: Versioning, approval workflows                     │
│  ML Lineage Tracking: Reproducibility                               │
└─────────────────────────────────────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│ STAGING ACCOUNT                                                      │
│  SageMaker Serverless Inference: Testing                            │
│  A/B testing                                                        │
│  Shadow deployment                                                  │
└─────────────────────────────────────────────────────────────────────┘
        │ (Manual Approval)
        ▼
┌─────────────────────────────────────────────────────────────────────┐
│ PRODUCTION ACCOUNT                                                   │
│  SageMaker Real-time Endpoints: Live inference                      │
│  Feature Store (Online): Low-latency features                       │
│  Auto Scaling: Handle load spikes                                   │
│  Model Monitor: Drift detection                                     │
│  Clarify: Bias monitoring                                           │
│  CloudWatch: Metrics & alarms                                       │
└─────────────────────────────────────────────────────────────────────┘

CONTINUOUS TRAINING (CT):
- EventBridge: Trigger on new data or drift
- Model Monitor: Data quality, bias, feature attribution
- Automatic retraining pipeline invocation
- MLflow integration: Experiment tracking

INFERENCE PATTERNS:
1. Real-time: SageMaker Endpoints (load balanced)
2. Serverless: Cold start OK workloads
3. Asynchronous: Large payloads, batch
4. Batch Transform: Offline predictions
5. Edge: Greengrass, IoT deployment

ML GATEWAY PATTERN:
API Gateway → Lambda → SageMaker Endpoint
Benefits: Caching, throttling, monitoring
```

---

### ARCH-012: DISASTER RECOVERY
```
4 STRATEGIE DR (da meno a più costoso):

┌─────────────────────────────────────────────────────────────────────┐
│ 1. BACKUP & RESTORE                                                  │
│    RPO: Hours | RTO: 24 hours or less                               │
│    Cost: $ (lowest)                                                  │
│                                                                     │
│    - S3 Cross-Region Replication                                    │
│    - AWS Backup: Automated, cross-region                            │
│    - RDS automated backups (cross-region)                           │
│    - Infrastructure as Code (CloudFormation/CDK)                    │
│    - EventBridge: Automation for faster recovery                    │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ 2. PILOT LIGHT                                                       │
│    RPO: Minutes | RTO: Tens of minutes to hours                     │
│    Cost: $$ (low)                                                    │
│                                                                     │
│    - Core infrastructure always running (minimal)                   │
│    - Data continuously replicated:                                  │
│      * Aurora Global Database                                       │
│      * DynamoDB Global Tables                                       │
│      * S3 Replication                                               │
│    - Scale up on failover (Auto Scaling, CloudFormation)            │
│    - Route 53 health checks + failover routing                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ 3. WARM STANDBY                                                      │
│    RPO: Seconds | RTO: Minutes                                       │
│    Cost: $$$ (medium)                                                │
│                                                                     │
│    - Reduced-capacity environment always running                    │
│    - Business-critical systems fully duplicated                     │
│    - Scale up quickly on failover                                   │
│    - AWS Elastic Disaster Recovery                                  │
│    - Route 53 weighted/failover routing                             │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│ 4. MULTI-SITE ACTIVE/ACTIVE                                          │
│    RPO: Near-zero | RTO: Near-zero                                   │
│    Cost: $$$$ (highest)                                              │
│                                                                     │
│    - Full capacity in multiple regions                              │
│    - Traffic served from all regions                                │
│    - Synchronous or asynchronous replication                        │
│    - Route 53 latency/geolocation routing                           │
│    - Global Accelerator: Automatic failover                         │
│                                                                     │
│    WRITE PATTERNS:                                                  │
│    - Write-global: Single write region                              │
│    - Write-partitioned: Each region owns subset                     │
│    - Write-local: Conflict resolution needed                        │
└─────────────────────────────────────────────────────────────────────┘

SERVIZI DR CHIAVE:
- Route 53: DNS failover, health checks
- Global Accelerator: Anycast IP, automatic failover
- Aurora Global Database: Cross-region replication
- DynamoDB Global Tables: Multi-region, multi-master
- S3 Replication: Cross-region, same-region
- AWS Backup: Centralized backup management
- Elastic Disaster Recovery: Block-level replication
- CloudFormation/CDK: Infrastructure as Code

SELEZIONE STRATEGIA:
IF RTO < 1 min AND RPO near-zero → Active/Active
ELSE IF RTO < 30 min AND RPO < 5 min → Warm Standby
ELSE IF RTO < 4 hours AND RPO < 15 min → Pilot Light
ELSE → Backup & Restore
```



---

### ARCH-013: SERVERLESS MICROSERVICES
```
REQUISITI:
- Event-driven
- Auto-scaling
- Pay-per-use
- Decoupling
- Low operational overhead

PATTERN FONDAMENTALI:

1. API + LAMBDA (Synchronous):
   API Gateway → Lambda → DynamoDB
   Use: RESTful CRUD operations

2. STORAGE-FIRST (Durable Ingestion):
   API Gateway → SQS → Lambda → S3/DynamoDB
   Use: No data loss on failures

3. FAN-OUT (Parallel Processing):
   SNS → Lambda (multiple)
         → SQS
         → EventBridge
   Use: Broadcast events

4. EVENT-DRIVEN PIPELINE:
   S3 Event → EventBridge → Step Functions → Lambda
   Use: ETL, data processing

5. TOPIC-QUEUE-CHAINING:
   EventBridge → SQS → Lambda
   Use: Durable pub/sub, no message loss

6. CIRCUIT BREAKER:
   CloudWatch Alarms → EventBridge → Step Functions
   Use: Fail fast, downstream protection

7. SAGA PATTERN:
   Step Functions: Orchestration of distributed transactions
   Use: Order processing, multi-service coordination

ARCHITETTURA E-COMMERCE SERVERLESS:
┌─────────────────────────────────────────────────────────────────────┐
│                         PRODUCT MICROSERVICE                         │
│  API Gateway → Lambda → DynamoDB                                    │
│  (CRUD operations)                                                  │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         BASKET MICROSERVICE                          │
│  API Gateway → Lambda → DynamoDB                                    │
│  Checkout → EventBridge (publish basketCheckout)                    │
└─────────────────────────────────────────────────────────────────────┘
                                    │ (async)
┌─────────────────────────────────────────────────────────────────────┐
│                         EVENT BUS                                    │
│  EventBridge → SQS (durable storage)                                │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         ORDERING MICROSERVICE                        │
│  SQS → Lambda → DynamoDB                                            │
│  (Process orders asynchronously)                                    │
└─────────────────────────────────────────────────────────────────────┘

ANTI-PATTERNS DA EVITARE:
❌ Lambda calling Lambda synchronously (cost + complexity)
❌ Monolithic Lambda functions
❌ Recursive loops (S3 → Lambda → S3)
❌ Batch processing in Lambda (use streaming)
❌ Functions without idempotency

BEST PRACTICES:
✅ Use SQS between Lambdas
✅ Step Functions for orchestration
✅ EventBridge for decoupling microservices
✅ Lambda Layers for shared code
✅ Idempotent functions (DynamoDB conditional writes)
✅ Dead Letter Queues (DLQ) for failures
✅ X-Ray for distributed tracing
```

---

### ARCH-014: CONTAINER ORCHESTRATION (EKS)
```
REQUISITI:
- Kubernetes workloads
- Microservices at scale
- Hybrid/multi-cloud
- Advanced orchestration
- CI/CD integration

ARCHITETTURA EKS:
┌─────────────────────────────────────────────────────────────────────┐
│                         CONTROL PLANE (AWS Managed)                  │
│  - API Server (multi-AZ)                                            │
│  - etcd (distributed)                                               │
│  - Controller Manager                                               │
│  - Scheduler                                                        │
│  - 99.95% SLA                                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA PLANE OPTIONS                           │
│                                                                     │
│  1. EKS Auto Mode (Recommended for most)                            │
│     - AWS manages nodes, networking, GPU support                    │
│     - Immutable AMIs, automatic updates                             │
│     - Respects Pod Disruption Budgets                               │
│                                                                     │
│  2. Managed Node Groups                                             │
│     - AWS manages patching, updating, scaling                       │
│     - You choose instance types                                     │
│                                                                     │
│  3. Karpenter (Advanced auto-scaling)                               │
│     - Just-in-time provisioning                                     │
│     - Right-sized instances                                         │
│     - 40% cost savings typical                                      │
│                                                                     │
│  4. Fargate (Serverless)                                            │
│     - No node management                                            │
│     - Per-pod pricing                                               │
│                                                                     │
│  5. Self-managed Nodes (Full control)                               │
│     - You manage everything                                         │
│                                                                     │
│  6. Hybrid Nodes (On-premises + AWS)                                │
│     - EKS control plane + on-prem nodes                             │
└─────────────────────────────────────────────────────────────────────┘

EKS MULTI-TENANT ARCHITECTURE:
┌─────────────────────────────────────────────────────────────────────┐
│                         NETWORKING                                   │
│  VPC: Public + Private subnets                                      │
│  ALB Ingress Controller: HTTP/HTTPS routing                         │
│  NGINX Ingress: Per-tenant namespace routing                        │
│  Network Policies: Tenant isolation                                 │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         TENANT ISOLATION                             │
│  - Kubernetes Namespaces per tenant                                 │
│  - Resource Quotas                                                  │
│  - Network Policies                                                 │
│  - IRSA: IAM Roles for Service Accounts                             │
│  - ABAC: Attribute-Based Access Control                             │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         WORKLOADS                                    │
│  Deployments: Microservices (product, order, user)                  │
│  Services: Stable endpoints                                         │
│  ConfigMaps/Secrets: Configuration                                  │
│  HPA: Horizontal Pod Autoscaler                                     │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         STORAGE                                      │
│  EBS CSI Driver: Block storage                                      │
│  EFS CSI Driver: Shared file system                                 │
│  S3 (via app): Object storage                                       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
┌─────────────────────────────────────────────────────────────────────┐
│                         CI/CD                                        │
│  CodePipeline → CodeBuild → ECR → kubectl apply                     │
│  FluxCD/ArgoCD: GitOps                                              │
│  Helm: Package management                                           │
│  Crossplane: Infrastructure as Kubernetes                           │
└─────────────────────────────────────────────────────────────────────┘

EKS vs ECS DECISION:
| Criteria              | EKS                    | ECS                    |
|-----------------------|------------------------|------------------------|
| Kubernetes expertise  | Required               | Not needed             |
| Portability           | High (standard K8s)    | AWS-specific           |
| Complexity            | Higher                 | Lower                  |
| Multi-cloud           | Yes                    | No                     |
| Learning curve        | Steep                  | Gentle                 |
| Cost (control plane)  | $0.10/hour             | Free                   |
| Advanced orchestration| Yes                    | Limited                |

CHOOSE EKS IF:
- Kubernetes expertise available
- Multi-cloud/hybrid requirements
- Complex orchestration needs
- Large-scale microservices

CHOOSE ECS IF:
- AWS-native preferred
- Simpler requirements
- Lower operational overhead
- Smaller teams
```



---

## 4. MAPPATURA REQUISITI NON-FUNZIONALI → SERVIZI

### 4.1 SCALABILITÀ
| Requisito | Servizi AWS |
|-----------|-------------|
| Auto-scaling compute | Lambda, Fargate, EC2 Auto Scaling, Karpenter |
| Auto-scaling database | DynamoDB on-demand, Aurora Serverless |
| Auto-scaling storage | S3 (unlimited), EFS |
| Global scale | CloudFront, Global Accelerator, DynamoDB Global Tables |

### 4.2 PERFORMANCE
| Requisito | Servizi AWS |
|-----------|-------------|
| Low latency (< 10ms) | ElastiCache, DAX, Global Accelerator |
| High throughput | Kinesis, DynamoDB, SQS |
| Edge computing | CloudFront, Lambda@Edge, Greengrass |
| In-memory | ElastiCache Redis, MemoryDB |

### 4.3 AVAILABILITY
| Requisito | Servizi AWS |
|-----------|-------------|
| Multi-AZ | RDS Multi-AZ, EKS multi-AZ, Aurora |
| Multi-Region | Route 53, Global Tables, Aurora Global |
| 99.99%+ SLA | S3, DynamoDB, Lambda |
| Self-healing | Auto Scaling, ECS, EKS |

### 4.4 SECURITY
| Requisito | Servizi AWS |
|-----------|-------------|
| Encryption at rest | KMS, S3 SSE, RDS encryption |
| Encryption in transit | ACM, TLS, VPN |
| Identity | IAM, Cognito, SSO |
| Network isolation | VPC, Security Groups, PrivateLink |
| WAF/DDoS | WAF, Shield, CloudFront |
| Secrets | Secrets Manager, Parameter Store |
| Audit | CloudTrail, Config, Security Hub |

### 4.5 COMPLIANCE
| Standard | Servizi Chiave |
|----------|----------------|
| HIPAA | 166+ eligible services, HealthLake, Comprehend Medical |
| PCI DSS | KMS, CloudHSM, VPC isolation, Audit Manager |
| SOC 1/2/3 | Most AWS services |
| GDPR | Macie, Lake Formation, Cognito |
| FedRAMP | GovCloud services |

### 4.6 COST OPTIMIZATION
| Requisito | Servizi/Strategie |
|-----------|-------------------|
| Pay-per-use | Lambda, Fargate, S3, DynamoDB on-demand |
| Reserved capacity | EC2 Reserved, Savings Plans |
| Spot instances | EC2 Spot, Fargate Spot |
| Right-sizing | Compute Optimizer, Trusted Advisor |
| Cost visibility | Cost Explorer, Budgets, Cost Allocation Tags |

---

## 5. WELL-ARCHITECTED FRAMEWORK - 6 PILASTRI

| Pilastro | Focus | Servizi Chiave |
|----------|-------|----------------|
| **Operational Excellence** | Run & monitor | CloudWatch, X-Ray, Systems Manager |
| **Security** | Protect | IAM, KMS, WAF, GuardDuty |
| **Reliability** | Recover from failures | Multi-AZ, Auto Scaling, Backup |
| **Performance Efficiency** | Use resources efficiently | Auto Scaling, ElastiCache, CloudFront |
| **Cost Optimization** | Avoid unnecessary costs | Spot, Reserved, Savings Plans |
| **Sustainability** | Minimize environmental impact | Graviton, Serverless |

### WELL-ARCHITECTED LENSES (Domain-Specific)
- Machine Learning Lens
- Generative AI Lens (new 2025)
- Responsible AI Lens (new 2025)
- Serverless Lens
- IoT Lens
- Gaming Lens
- Financial Services Lens
- Healthcare Industry Lens
- Streaming Media Lens
- Data Analytics Lens
- SaaS Lens

---

## 6. DECISION TREE: REQUISITI → ARCHITETTURA

```
START
│
├─ È un'applicazione web/API?
│   ├─ Serverless preferito? → ARCH-013 (Serverless Microservices)
│   ├─ Kubernetes richiesto? → ARCH-014 (EKS)
│   └─ E-commerce? → ARCH-001 (E-Commerce Serverless)
│
├─ Real-time/Chat richiesto?
│   └─ Sì → ARCH-002 (Social Media/Chat)
│
├─ Video streaming?
│   └─ Sì → ARCH-003 (Video Streaming)
│
├─ Gaming multiplayer?
│   └─ Sì → ARCH-004 (Gaming)
│
├─ Healthcare/PHI data?
│   └─ Sì → ARCH-005 (Healthcare HIPAA)
│
├─ Payment processing?
│   └─ Sì → ARCH-006 (FinTech PCI-DSS)
│
├─ IoT/Dispositivi?
│   ├─ Industrial → ARCH-007 (IIoT)
│   └─ Consumer → ARCH-008 (Smart Home)
│
├─ Multi-tenant SaaS?
│   └─ Sì → ARCH-009 (SaaS Multi-Tenant)
│
├─ Analytics/Data Lake?
│   └─ Sì → ARCH-010 (Data Lake/Lakehouse)
│
├─ Machine Learning?
│   └─ Sì → ARCH-011 (MLOps Pipeline)
│
└─ Business Continuity prioritario?
    └─ Sì → ARCH-012 (Disaster Recovery) + altro pattern
```

---

## 7. RIFERIMENTI E FONTI

### AWS Official
- AWS Architecture Center: https://aws.amazon.com/architecture/
- AWS Well-Architected Framework: https://aws.amazon.com/architecture/well-architected/
- AWS Solutions Library: https://aws.amazon.com/solutions/
- AWS Prescriptive Guidance: https://docs.aws.amazon.com/prescriptive-guidance/

### Reference Architectures Specifiche
- E-Commerce: https://aws.amazon.com/solutions/guidance/web-store-on-aws/
- Healthcare: https://aws.amazon.com/health/
- FinTech: https://aws.amazon.com/financial-services/
- IoT: https://aws.amazon.com/iot/
- SaaS: https://aws.amazon.com/partners/saas/
- Gaming: https://aws.amazon.com/gametech/
- Media: https://aws.amazon.com/media-services/

### Case Studies
- Netflix: https://aws.amazon.com/solutions/case-studies/innovators/netflix/
- Twitch: https://aws.amazon.com/solutions/case-studies/twitch/

---

## 8. VERSIONING

| Versione | Data | Note |
|----------|------|------|
| 1.0 | 2026-01-25 | Initial release - 14 pattern architetturali |

---

**FINE CATALOGO**
