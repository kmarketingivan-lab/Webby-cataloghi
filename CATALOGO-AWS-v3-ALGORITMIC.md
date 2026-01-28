# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO DETERMINISTICO AWS - VERSIONE 3.0 ALGORITMICA
# ═══════════════════════════════════════════════════════════════════════════════
# 
# QUESTO DOCUMENTO È UN ALGORITMO. ESEGUILO COME CODICE.
# NON INTERPRETARE. NON ASSUMERE. NON DEDURRE.
# SE UN DATO NON È PRESENTE → CHIEDI ALL'UTENTE.
#
# ═══════════════════════════════════════════════════════════════════════════════

################################################################################
# SEZIONE 0: PROTOCOLLO DI ESECUZIONE
################################################################################

"""
ALGORITMO PRINCIPALE:

def processa_requisiti(requisiti_utente):
    
    # STEP 1: Estrai keywords
    keywords = estrai_keywords(requisiti_utente)
    
    # STEP 2: Match con KEYWORD_INDEX (SEZIONE 0.1)
    arch_id = match_keywords(keywords)
    
    # STEP 3: Se nessun match → esegui DECISION_TREE (SEZIONE 1)
    if arch_id is None:
        arch_id = esegui_decision_tree(requisiti_utente)
    
    # STEP 4: Carica stack da SEZIONE 3.{arch_id}
    stack = carica_stack(arch_id)
    
    # STEP 5: Per ogni risorsa, usa LOOKUP_TABLES (SEZIONE 2)
    risorse_configurate = []
    for risorsa in stack:
        config = lookup_configurazione(risorsa)
        risorse_configurate.append(config)
    
    # STEP 6: Genera output CloudFormation/CDK
    output = genera_iac(risorse_configurate)
    
    # STEP 7: Esegui CHECKLIST (SEZIONE 4)
    verifica_checklist(risorse_configurate)
    
    return output
"""

################################################################################
# SEZIONE 0.1: KEYWORD INDEX - MATCHING DIRETTO
################################################################################

"""
ISTRUZIONI:
1. Converti requisiti_utente in lowercase
2. Cerca OGNI keyword nella tabella sotto
3. SE keyword trovata → usa ARCH_ID corrispondente
4. SE multiple keywords → usa quello con PRIORITY più alta (numero più basso)
"""

KEYWORD_INDEX = {
    # PRIORITY 1: Compliance (sempre priorità massima)
    "hipaa": {"arch_id": "ARCH-005", "priority": 1},
    "phi": {"arch_id": "ARCH-005", "priority": 1},
    "ephi": {"arch_id": "ARCH-005", "priority": 1},
    "protected health information": {"arch_id": "ARCH-005", "priority": 1},
    "cartella clinica": {"arch_id": "ARCH-005", "priority": 1},
    "paziente": {"arch_id": "ARCH-005", "priority": 1},
    "ospedale": {"arch_id": "ARCH-005", "priority": 1},
    "clinica": {"arch_id": "ARCH-005", "priority": 1},
    "medico": {"arch_id": "ARCH-005", "priority": 1},
    "diagnosi": {"arch_id": "ARCH-005", "priority": 1},
    "healthcare": {"arch_id": "ARCH-005", "priority": 1},
    "telehealth": {"arch_id": "ARCH-005", "priority": 1},
    "telemedicina": {"arch_id": "ARCH-005", "priority": 1},
    "fhir": {"arch_id": "ARCH-005", "priority": 1},
    "hl7": {"arch_id": "ARCH-005", "priority": 1},
    
    "pci": {"arch_id": "ARCH-006", "priority": 1},
    "pci-dss": {"arch_id": "ARCH-006", "priority": 1},
    "pci dss": {"arch_id": "ARCH-006", "priority": 1},
    "carta di credito": {"arch_id": "ARCH-006", "priority": 1},
    "credit card": {"arch_id": "ARCH-006", "priority": 1},
    "pan": {"arch_id": "ARCH-006", "priority": 1},
    "cardholder": {"arch_id": "ARCH-006", "priority": 1},
    "payment processing": {"arch_id": "ARCH-006", "priority": 1},
    "pagamenti": {"arch_id": "ARCH-006", "priority": 1},
    "transazioni finanziarie": {"arch_id": "ARCH-006", "priority": 1},
    "acquiring": {"arch_id": "ARCH-006", "priority": 1},
    "issuing": {"arch_id": "ARCH-006", "priority": 1},
    "tokenization": {"arch_id": "ARCH-006", "priority": 1},
    
    # PRIORITY 2: Domain-specific
    "multiplayer": {"arch_id": "ARCH-004", "priority": 2},
    "game server": {"arch_id": "ARCH-004", "priority": 2},
    "matchmaking": {"arch_id": "ARCH-004", "priority": 2},
    "videogioco": {"arch_id": "ARCH-004", "priority": 2},
    "gaming": {"arch_id": "ARCH-004", "priority": 2},
    "lobby": {"arch_id": "ARCH-004", "priority": 2},
    "player session": {"arch_id": "ARCH-004", "priority": 2},
    "gamelift": {"arch_id": "ARCH-004", "priority": 2},
    
    "video streaming": {"arch_id": "ARCH-003", "priority": 2},
    "live streaming": {"arch_id": "ARCH-003", "priority": 2},
    "vod": {"arch_id": "ARCH-003", "priority": 2},
    "hls": {"arch_id": "ARCH-003", "priority": 2},
    "dash": {"arch_id": "ARCH-003", "priority": 2},
    "transcode": {"arch_id": "ARCH-003", "priority": 2},
    "transcoding": {"arch_id": "ARCH-003", "priority": 2},
    "ott": {"arch_id": "ARCH-003", "priority": 2},
    "broadcast": {"arch_id": "ARCH-003", "priority": 2},
    "media streaming": {"arch_id": "ARCH-003", "priority": 2},
    "netflix": {"arch_id": "ARCH-003", "priority": 2},
    "twitch": {"arch_id": "ARCH-003", "priority": 2},
    
    "iot": {"arch_id": "ARCH-007", "priority": 2},
    "sensor": {"arch_id": "ARCH-007", "priority": 2},
    "sensore": {"arch_id": "ARCH-007", "priority": 2},
    "mqtt": {"arch_id": "ARCH-007", "priority": 2},
    "telemetry": {"arch_id": "ARCH-007", "priority": 2},
    "telemetria": {"arch_id": "ARCH-007", "priority": 2},
    "plc": {"arch_id": "ARCH-007", "priority": 2},
    "opc-ua": {"arch_id": "ARCH-007", "priority": 2},
    "scada": {"arch_id": "ARCH-007", "priority": 2},
    "industrial": {"arch_id": "ARCH-007", "priority": 2},
    "factory": {"arch_id": "ARCH-007", "priority": 2},
    "manufacturing": {"arch_id": "ARCH-007", "priority": 2},
    "predictive maintenance": {"arch_id": "ARCH-007", "priority": 2},
    "manutenzione predittiva": {"arch_id": "ARCH-007", "priority": 2},
    
    "smart home": {"arch_id": "ARCH-008", "priority": 2},
    "domotica": {"arch_id": "ARCH-008", "priority": 2},
    "alexa": {"arch_id": "ARCH-008", "priority": 2},
    "voice assistant": {"arch_id": "ARCH-008", "priority": 2},
    "wearable": {"arch_id": "ARCH-008", "priority": 2},
    "consumer iot": {"arch_id": "ARCH-008", "priority": 2},
    
    # PRIORITY 3: Architecture patterns
    "chat": {"arch_id": "ARCH-002", "priority": 3},
    "messaging": {"arch_id": "ARCH-002", "priority": 3},
    "messaggistica": {"arch_id": "ARCH-002", "priority": 3},
    "real-time": {"arch_id": "ARCH-002", "priority": 3},
    "realtime": {"arch_id": "ARCH-002", "priority": 3},
    "websocket": {"arch_id": "ARCH-002", "priority": 3},
    "presenza online": {"arch_id": "ARCH-002", "priority": 3},
    "typing indicator": {"arch_id": "ARCH-002", "priority": 3},
    "social network": {"arch_id": "ARCH-002", "priority": 3},
    "feed": {"arch_id": "ARCH-002", "priority": 3},
    "notification push": {"arch_id": "ARCH-002", "priority": 3},
    
    "multi-tenant": {"arch_id": "ARCH-009", "priority": 3},
    "multitenant": {"arch_id": "ARCH-009", "priority": 3},
    "saas": {"arch_id": "ARCH-009", "priority": 3},
    "tenant isolation": {"arch_id": "ARCH-009", "priority": 3},
    "white-label": {"arch_id": "ARCH-009", "priority": 3},
    "whitelabel": {"arch_id": "ARCH-009", "priority": 3},
    "b2b platform": {"arch_id": "ARCH-009", "priority": 3},
    
    "data lake": {"arch_id": "ARCH-010", "priority": 3},
    "datalake": {"arch_id": "ARCH-010", "priority": 3},
    "data warehouse": {"arch_id": "ARCH-010", "priority": 3},
    "lakehouse": {"arch_id": "ARCH-010", "priority": 3},
    "etl": {"arch_id": "ARCH-010", "priority": 3},
    "analytics": {"arch_id": "ARCH-010", "priority": 3},
    "bi": {"arch_id": "ARCH-010", "priority": 3},
    "business intelligence": {"arch_id": "ARCH-010", "priority": 3},
    "iceberg": {"arch_id": "ARCH-010", "priority": 3},
    "athena": {"arch_id": "ARCH-010", "priority": 3},
    "redshift": {"arch_id": "ARCH-010", "priority": 3},
    "glue": {"arch_id": "ARCH-010", "priority": 3},
    
    "mlops": {"arch_id": "ARCH-011", "priority": 3},
    "ml pipeline": {"arch_id": "ARCH-011", "priority": 3},
    "model training": {"arch_id": "ARCH-011", "priority": 3},
    "model inference": {"arch_id": "ARCH-011", "priority": 3},
    "sagemaker": {"arch_id": "ARCH-011", "priority": 3},
    "feature store": {"arch_id": "ARCH-011", "priority": 3},
    "model registry": {"arch_id": "ARCH-011", "priority": 3},
    "machine learning platform": {"arch_id": "ARCH-011", "priority": 3},
    
    "kubernetes": {"arch_id": "ARCH-014", "priority": 3},
    "k8s": {"arch_id": "ARCH-014", "priority": 3},
    "eks": {"arch_id": "ARCH-014", "priority": 3},
    "kubectl": {"arch_id": "ARCH-014", "priority": 3},
    "helm": {"arch_id": "ARCH-014", "priority": 3},
    "container orchestration": {"arch_id": "ARCH-014", "priority": 3},
    
    # PRIORITY 4: Business domains
    "e-commerce": {"arch_id": "ARCH-001", "priority": 4},
    "ecommerce": {"arch_id": "ARCH-001", "priority": 4},
    "shop": {"arch_id": "ARCH-001", "priority": 4},
    "negozio online": {"arch_id": "ARCH-001", "priority": 4},
    "carrello": {"arch_id": "ARCH-001", "priority": 4},
    "cart": {"arch_id": "ARCH-001", "priority": 4},
    "checkout": {"arch_id": "ARCH-001", "priority": 4},
    "product catalog": {"arch_id": "ARCH-001", "priority": 4},
    "catalogo prodotti": {"arch_id": "ARCH-001", "priority": 4},
    "ordini": {"arch_id": "ARCH-001", "priority": 4},
    "inventory": {"arch_id": "ARCH-001", "priority": 4},
    "magazzino": {"arch_id": "ARCH-001", "priority": 4},
    
    "disaster recovery": {"arch_id": "ARCH-012", "priority": 4},
    "dr": {"arch_id": "ARCH-012", "priority": 4},
    "business continuity": {"arch_id": "ARCH-012", "priority": 4},
    "failover": {"arch_id": "ARCH-012", "priority": 4},
    "rto": {"arch_id": "ARCH-012", "priority": 4},
    "rpo": {"arch_id": "ARCH-012", "priority": 4},
    "backup": {"arch_id": "ARCH-012", "priority": 4},
    "multi-region": {"arch_id": "ARCH-012", "priority": 4},
}

# DEFAULT se nessuna keyword matchata
DEFAULT_ARCH_ID = "ARCH-013"  # Serverless Microservices


################################################################################
# SEZIONE 1: DECISION TREE - ESEGUI SE KEYWORD_INDEX NON HA MATCH
################################################################################

"""
ESECUZIONE:
- Parti da NODE_1
- Rispondi TRUE o FALSE
- Segui la freccia corrispondente
- FERMATI quando raggiungi RESULT

REGOLE:
- TRUE = la condizione è soddisfatta
- FALSE = la condizione NON è soddisfatta
- Se non sai → chiedi all'utente
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_1: COMPLIANCE HEALTHCARE
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema memorizza, trasmette o elabora dati sanitari protetti?

DEFINIZIONE "dati sanitari protetti":
- Nome paziente + condizione medica
- Cartelle cliniche
- Risultati esami
- Prescrizioni
- Dati assicurazione sanitaria
- Qualsiasi dato che identifica un paziente + informazione sanitaria

SE TRUE  → RESULT: ARCH-005
SE FALSE → VAI A NODE_2
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_2: COMPLIANCE PAYMENT
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema memorizza, trasmette o elabora dati di carte di pagamento?

DEFINIZIONE "dati carte di pagamento":
- PAN (Primary Account Number) - numero carta 16 cifre
- CVV/CVC
- Data scadenza
- Nome titolare
- PIN
- Track data (banda magnetica)

SE TRUE  → RESULT: ARCH-006
SE FALSE → VAI A NODE_3
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_3: GAMING
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema è un videogioco multiplayer online?

CRITERI (almeno 2 devono essere TRUE):
- Richiede game servers dedicati
- Ha sessioni di gioco con più giocatori simultanei
- Richiede matchmaking
- Latenza < 50ms è critica

SE TRUE  → RESULT: ARCH-004
SE FALSE → VAI A NODE_4
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_4: VIDEO STREAMING
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema eroga contenuti video in streaming?

CRITERI (almeno 1 deve essere TRUE):
- Live streaming video
- Video on demand (VOD)
- Richiede transcoding video
- Adaptive bitrate streaming (HLS/DASH)

SE TRUE  → RESULT: ARCH-003
SE FALSE → VAI A NODE_5
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_5: IoT
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema connette dispositivi fisici (sensori, attuatori)?

CRITERI (almeno 1 deve essere TRUE):
- Dispositivi inviano dati telemetrici
- Comunicazione MQTT/CoAP
- Edge computing richiesto
- Gestione flotta dispositivi

SE TRUE  → VAI A NODE_5A (sub-decision)
SE FALSE → VAI A NODE_6
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_5A: TIPO IoT
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: È IoT industriale?

CRITERI IoT INDUSTRIALE (almeno 1 deve essere TRUE):
- Ambiente factory/manufacturing
- Protocolli OPC-UA, Modbus, PROFINET
- Integrazione con PLC/SCADA
- Manutenzione predittiva macchinari
- MES (Manufacturing Execution System)

SE TRUE  → RESULT: ARCH-007 (Industrial IoT)
SE FALSE → RESULT: ARCH-008 (Consumer IoT / Smart Home)
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_6: REAL-TIME COMMUNICATION
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema richiede comunicazione real-time bidirezionale?

CRITERI (almeno 1 deve essere TRUE):
- Chat/messaggistica istantanea
- Notifiche push real-time
- Presenza online (chi è online)
- Typing indicators
- WebSocket connections persistenti
- Social feed real-time

SE TRUE  → RESULT: ARCH-002
SE FALSE → VAI A NODE_7
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_7: MULTI-TENANT SaaS
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema serve multiple organizzazioni (tenant) isolate?

CRITERI (TUTTI devono essere TRUE):
- Più clienti/organizzazioni usano la stessa applicazione
- I dati di un tenant NON devono essere visibili ad altri tenant
- Ogni tenant può avere configurazioni diverse

SE TRUE  → RESULT: ARCH-009
SE FALSE → VAI A NODE_8
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_8: DATA PLATFORM
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema è primariamente una piattaforma dati/analytics?

CRITERI (almeno 2 devono essere TRUE):
- Ingestione dati da multiple sorgenti
- ETL/ELT pipelines
- Data warehouse
- Business Intelligence/Reporting
- Machine Learning su larga scala
- Data lake

SE TRUE  → RESULT: ARCH-010
SE FALSE → VAI A NODE_9
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_9: ML PLATFORM
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema è una piattaforma ML/AI dedicata?

CRITERI (almeno 2 devono essere TRUE):
- Training modelli ML
- Serving modelli (inference)
- Feature engineering
- Model registry/versioning
- A/B testing modelli
- Monitoring drift

SE TRUE  → RESULT: ARCH-011
SE FALSE → VAI A NODE_10
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_10: KUBERNETES REQUIRED
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Kubernetes è un requisito esplicito?

CRITERI (almeno 1 deve essere TRUE):
- Cliente ha specificato Kubernetes/K8s/EKS
- Migrazione da cluster Kubernetes esistente
- Team ha expertise Kubernetes
- Portabilità multi-cloud richiesta

SE TRUE  → RESULT: ARCH-014
SE FALSE → VAI A NODE_11
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_11: E-COMMERCE
# ══════════════════════════════════════════════════════════════════════════════
"""
CONDIZIONE: Il sistema è un e-commerce/negozio online?

CRITERI (almeno 2 devono essere TRUE):
- Catalogo prodotti
- Carrello acquisti
- Checkout/pagamento (non memorizzato, via gateway)
- Gestione ordini
- Inventario

SE TRUE  → RESULT: ARCH-001
SE FALSE → VAI A NODE_12
"""

# ══════════════════════════════════════════════════════════════════════════════
# NODE_12: DEFAULT
# ══════════════════════════════════════════════════════════════════════════════
"""
NESSUNA CONDIZIONE PRECEDENTE SODDISFATTA

→ RESULT: ARCH-013 (Serverless Microservices - architettura generica)
"""


################################################################################
# SEZIONE 2: LOOKUP TABLES - CONFIGURAZIONI ESATTE
################################################################################

"""
ISTRUZIONI:
1. Identifica il tipo di risorsa necessaria
2. Trova la tabella corrispondente
3. Verifica le condizioni dall'alto verso il basso
4. USA la configurazione della PRIMA riga che matcha
5. NON modificare i valori - sono ESATTI
"""

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.1: COMPUTE SERVICE SELECTION
# ══════════════════════════════════════════════════════════════════════════════

COMPUTE_SELECTION = [
    # Verifica condizioni in ordine. Prima che matcha = servizio da usare.
    {
        "condizioni": {
            "durata_esecuzione_max_secondi": 900,  # <= 15 minuti
            "stateless": True,
            "evento_driven": True
        },
        "servizio": "Lambda",
        "configurazione": {
            "runtime": "nodejs20.x",  # OPPURE python3.12
            "memory_mb": 512,
            "timeout_seconds": 30,
            "architecture": "arm64",  # Graviton, 20% risparmio
            "ephemeral_storage_mb": 512
        }
    },
    {
        "condizioni": {
            "container": True,
            "kubernetes": False,
            "scaling_automatico": True
        },
        "servizio": "ECS_Fargate",
        "configurazione": {
            "cpu": 256,  # 0.25 vCPU
            "memory_mb": 512,
            "platform_version": "1.4.0",
            "network_mode": "awsvpc"
        }
    },
    {
        "condizioni": {
            "container": True,
            "kubernetes": True
        },
        "servizio": "EKS",
        "configurazione": {
            "version": "1.29",
            "endpoint_access": "private",
            "logging": ["api", "audit", "authenticator", "controllerManager", "scheduler"]
        }
    },
    {
        "condizioni": {
            "gpu_required": True,
            "ml_training": True
        },
        "servizio": "EC2",
        "configurazione": {
            "instance_type": "p5.48xlarge",  # ML Training
            "ami": "Deep Learning AMI GPU PyTorch 2.0"
        }
    },
    {
        "condizioni": {
            "gpu_required": True,
            "ml_inference": True
        },
        "servizio": "EC2",
        "configurazione": {
            "instance_type": "g5.xlarge",  # ML Inference
            "ami": "Deep Learning AMI GPU PyTorch 2.0"
        }
    },
    {
        "condizioni": {
            "high_memory": True,  # > 64GB RAM needed
            "database_in_memory": True
        },
        "servizio": "EC2",
        "configurazione": {
            "instance_type": "r7g.2xlarge",  # 64GB RAM
            "ami": "Amazon Linux 2023"
        }
    },
    {
        "condizioni": {
            "cpu_intensive": True,
            "compute_optimized": True
        },
        "servizio": "EC2",
        "configurazione": {
            "instance_type": "c7g.xlarge",  # Compute optimized
            "ami": "Amazon Linux 2023"
        }
    },
    {
        "condizioni": {
            "batch_job": True,
            "scheduled": True
        },
        "servizio": "AWS_Batch",
        "configurazione": {
            "compute_environment_type": "FARGATE",
            "max_vcpus": 256
        }
    },
    # DEFAULT
    {
        "condizioni": {},  # Nessuna condizione = default
        "servizio": "Lambda",
        "configurazione": {
            "runtime": "nodejs20.x",
            "memory_mb": 512,
            "timeout_seconds": 30,
            "architecture": "arm64"
        }
    }
]

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.2: DATABASE SERVICE SELECTION
# ══════════════════════════════════════════════════════════════════════════════

DATABASE_SELECTION = [
    {
        "condizioni": {
            "sql": True,
            "acid": True,
            "high_performance": True,
            "auto_scaling": True
        },
        "servizio": "Aurora_PostgreSQL_Serverless_v2",
        "configurazione": {
            "engine": "aurora-postgresql",
            "engine_version": "15.4",
            "serverless_v2_scaling": {
                "min_capacity": 0.5,
                "max_capacity": 16
            },
            "storage_encrypted": True,
            "deletion_protection": True,
            "backup_retention_days": 7
        }
    },
    {
        "condizioni": {
            "sql": True,
            "acid": True,
            "high_performance": True,
            "auto_scaling": False  # Provisioned
        },
        "servizio": "Aurora_PostgreSQL",
        "configurazione": {
            "engine": "aurora-postgresql",
            "engine_version": "15.4",
            "instance_class": "db.r6g.large",
            "multi_az": True,
            "storage_encrypted": True,
            "deletion_protection": True,
            "backup_retention_days": 7,
            "performance_insights": True,
            "monitoring_interval": 60
        }
    },
    {
        "condizioni": {
            "sql": True,
            "mysql_compatibility": True
        },
        "servizio": "Aurora_MySQL",
        "configurazione": {
            "engine": "aurora-mysql",
            "engine_version": "8.0.mysql_aurora.3.04.0",
            "instance_class": "db.r6g.large",
            "multi_az": True,
            "storage_encrypted": True
        }
    },
    {
        "condizioni": {
            "nosql": True,
            "key_value": True,
            "latency_ms_max": 10
        },
        "servizio": "DynamoDB_with_DAX",
        "configurazione": {
            "billing_mode": "PAY_PER_REQUEST",
            "point_in_time_recovery": True,
            "encryption": "AWS_OWNED",
            "dax_cluster": {
                "node_type": "dax.r5.large",
                "node_count": 3
            }
        }
    },
    {
        "condizioni": {
            "nosql": True,
            "key_value": True
        },
        "servizio": "DynamoDB",
        "configurazione": {
            "billing_mode": "PAY_PER_REQUEST",
            "point_in_time_recovery": True,
            "encryption": "AWS_OWNED",
            "stream_enabled": False  # Abilitare se event-driven
        }
    },
    {
        "condizioni": {
            "time_series": True
        },
        "servizio": "Timestream",
        "configurazione": {
            "memory_retention_hours": 24,
            "magnetic_retention_days": 365
        }
    },
    {
        "condizioni": {
            "graph": True
        },
        "servizio": "Neptune",
        "configurazione": {
            "instance_class": "db.r5.large",
            "multi_az": True,
            "storage_encrypted": True
        }
    },
    {
        "condizioni": {
            "full_text_search": True
        },
        "servizio": "OpenSearch",
        "configurazione": {
            "engine_version": "OpenSearch_2.11",
            "instance_type": "r6g.large.search",
            "instance_count": 2,
            "zone_awareness": True,
            "encrypt_at_rest": True
        }
    },
    {
        "condizioni": {
            "document_store": True,
            "mongodb_compatible": True
        },
        "servizio": "DocumentDB",
        "configurazione": {
            "engine_version": "6.0",
            "instance_class": "db.r6g.large",
            "instance_count": 3,
            "storage_encrypted": True
        }
    },
    {
        "condizioni": {
            "cache": True,
            "session_store": True
        },
        "servizio": "ElastiCache_Redis",
        "configurazione": {
            "engine": "redis",
            "engine_version": "7.0",
            "node_type": "cache.r6g.large",
            "num_cache_nodes": 2,
            "multi_az": True,
            "at_rest_encryption": True,
            "transit_encryption": True
        }
    },
    {
        "condizioni": {
            "data_warehouse": True,
            "olap": True
        },
        "servizio": "Redshift_Serverless",
        "configurazione": {
            "base_capacity": 32,  # RPU
            "namespace_name": "default",
            "workgroup_name": "default"
        }
    },
    {
        "condizioni": {
            "ledger": True,
            "immutable": True
        },
        "servizio": "QLDB",
        "configurazione": {
            "permissions_mode": "STANDARD"
        }
    },
    # DEFAULT
    {
        "condizioni": {},
        "servizio": "DynamoDB",
        "configurazione": {
            "billing_mode": "PAY_PER_REQUEST",
            "point_in_time_recovery": True,
            "encryption": "AWS_OWNED"
        }
    }
]

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.3: STORAGE SERVICE SELECTION
# ══════════════════════════════════════════════════════════════════════════════

STORAGE_SELECTION = [
    {
        "condizioni": {
            "object_storage": True,
            "access_frequency": "frequent"  # accesso giornaliero
        },
        "servizio": "S3",
        "configurazione": {
            "storage_class": "STANDARD",
            "versioning": True,
            "encryption": "AES256",
            "public_access_block": {
                "block_public_acls": True,
                "block_public_policy": True,
                "ignore_public_acls": True,
                "restrict_public_buckets": True
            }
        }
    },
    {
        "condizioni": {
            "object_storage": True,
            "access_frequency": "unknown"  # pattern sconosciuto
        },
        "servizio": "S3",
        "configurazione": {
            "storage_class": "INTELLIGENT_TIERING",
            "versioning": True,
            "encryption": "AES256",
            "public_access_block": {
                "block_public_acls": True,
                "block_public_policy": True,
                "ignore_public_acls": True,
                "restrict_public_buckets": True
            }
        }
    },
    {
        "condizioni": {
            "object_storage": True,
            "archive": True,
            "retrieval_time": "hours"  # 3-5 ore OK
        },
        "servizio": "S3_Glacier",
        "configurazione": {
            "storage_class": "GLACIER_FLEXIBLE_RETRIEVAL",
            "versioning": True,
            "encryption": "AES256"
        }
    },
    {
        "condizioni": {
            "object_storage": True,
            "archive": True,
            "retrieval_time": "days"  # 12+ ore OK
        },
        "servizio": "S3_Glacier",
        "configurazione": {
            "storage_class": "DEEP_ARCHIVE",
            "versioning": True,
            "encryption": "AES256"
        }
    },
    {
        "condizioni": {
            "file_system": True,
            "shared": True,
            "protocol": "nfs"
        },
        "servizio": "EFS",
        "configurazione": {
            "performance_mode": "generalPurpose",
            "throughput_mode": "bursting",
            "encrypted": True,
            "lifecycle_policy": "AFTER_30_DAYS"
        }
    },
    {
        "condizioni": {
            "file_system": True,
            "shared": True,
            "protocol": "smb",
            "windows": True
        },
        "servizio": "FSx_Windows",
        "configurazione": {
            "deployment_type": "MULTI_AZ_1",
            "storage_type": "SSD",
            "storage_capacity_gb": 32,
            "throughput_capacity_mbps": 32
        }
    },
    {
        "condizioni": {
            "file_system": True,
            "hpc": True,
            "high_throughput": True
        },
        "servizio": "FSx_Lustre",
        "configurazione": {
            "deployment_type": "PERSISTENT_1",
            "storage_capacity_gb": 1200,
            "per_unit_storage_throughput": 200
        }
    },
    {
        "condizioni": {
            "block_storage": True,
            "ec2": True
        },
        "servizio": "EBS",
        "configurazione": {
            "volume_type": "gp3",
            "iops": 3000,
            "throughput_mbps": 125,
            "encrypted": True
        }
    }
]


# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.4: NETWORKING SERVICE SELECTION
# ══════════════════════════════════════════════════════════════════════════════

NETWORKING_SELECTION = [
    {
        "condizioni": {
            "api": True,
            "rest": True,
            "features_advanced": True  # caching, request validation, API keys
        },
        "servizio": "API_Gateway_REST",
        "configurazione": {
            "endpoint_type": "REGIONAL",
            "api_key_required": False,
            "throttling_burst": 5000,
            "throttling_rate": 10000,
            "logging_level": "INFO",
            "metrics_enabled": True
        }
    },
    {
        "condizioni": {
            "api": True,
            "rest": True,
            "features_advanced": False  # semplice, basso costo
        },
        "servizio": "API_Gateway_HTTP",
        "configurazione": {
            "protocol_type": "HTTP",
            "cors_enabled": True,
            "throttling_burst": 5000,
            "throttling_rate": 10000
        }
    },
    {
        "condizioni": {
            "api": True,
            "websocket": True
        },
        "servizio": "API_Gateway_WebSocket",
        "configurazione": {
            "protocol_type": "WEBSOCKET",
            "route_selection_expression": "$request.body.action"
        }
    },
    {
        "condizioni": {
            "api": True,
            "graphql": True
        },
        "servizio": "AppSync",
        "configurazione": {
            "authentication_type": "AMAZON_COGNITO_USER_POOLS",
            "xray_enabled": True,
            "logging_level": "ALL"
        }
    },
    {
        "condizioni": {
            "cdn": True,
            "static_content": True
        },
        "servizio": "CloudFront",
        "configurazione": {
            "price_class": "PriceClass_100",  # US, Canada, Europe
            "viewer_protocol_policy": "redirect-to-https",
            "default_ttl": 86400,
            "compress": True,
            "http_version": "http2and3"
        }
    },
    {
        "condizioni": {
            "global_acceleration": True,
            "tcp_udp": True
        },
        "servizio": "Global_Accelerator",
        "configurazione": {
            "flow_logs_enabled": True,
            "endpoint_group_health_check_interval": 30
        }
    },
    {
        "condizioni": {
            "dns": True
        },
        "servizio": "Route53",
        "configurazione": {
            "hosted_zone_type": "public",  # o "private" per VPC internal
            "alias_records_preferred": True
        }
    },
    {
        "condizioni": {
            "load_balancer": True,
            "http_https": True
        },
        "servizio": "ALB",
        "configurazione": {
            "scheme": "internet-facing",  # o "internal"
            "ip_address_type": "ipv4",
            "deletion_protection": True,
            "access_logs_enabled": True,
            "idle_timeout_seconds": 60
        }
    },
    {
        "condizioni": {
            "load_balancer": True,
            "tcp_udp": True,
            "ultra_low_latency": True
        },
        "servizio": "NLB",
        "configurazione": {
            "scheme": "internet-facing",
            "cross_zone": True,
            "deletion_protection": True
        }
    },
    {
        "condizioni": {
            "vpc_connectivity": True,
            "multiple_vpcs": True
        },
        "servizio": "Transit_Gateway",
        "configurazione": {
            "auto_accept_shared_attachments": "disable",
            "default_route_table_association": "enable",
            "dns_support": "enable",
            "vpn_ecmp_support": "enable"
        }
    },
    {
        "condizioni": {
            "on_premises": True,
            "bandwidth_mbps": 1000  # > 1Gbps
        },
        "servizio": "Direct_Connect",
        "configurazione": {
            "connection_bandwidth": "1Gbps",  # o "10Gbps"
            "lag_enabled": False
        }
    },
    {
        "condizioni": {
            "on_premises": True,
            "bandwidth_mbps": 500  # <= 1Gbps, budget limitato
        },
        "servizio": "Site_to_Site_VPN",
        "configurazione": {
            "tunnel_options": {
                "pre_shared_key": "auto",
                "ike_versions": ["ikev2"]
            }
        }
    }
]

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.5: SECURITY - SERVIZI OBBLIGATORI (TUTTI I PROGETTI)
# ══════════════════════════════════════════════════════════════════════════════

SECURITY_MANDATORY = {
    "encryption_keys": {
        "servizio": "KMS",
        "configurazione": {
            "key_spec": "SYMMETRIC_DEFAULT",
            "key_usage": "ENCRYPT_DECRYPT",
            "enable_key_rotation": True,
            "multi_region": False  # True solo se DR multi-region
        }
    },
    "secrets": {
        "servizio": "Secrets_Manager",
        "configurazione": {
            "rotation_enabled": True,
            "rotation_lambda_arn": "auto",
            "rotation_days": 30
        }
    },
    "user_authentication": {
        "servizio": "Cognito",
        "configurazione": {
            "mfa_configuration": "OPTIONAL",  # "ON" per HIPAA/PCI
            "password_policy": {
                "minimum_length": 12,
                "require_lowercase": True,
                "require_uppercase": True,
                "require_numbers": True,
                "require_symbols": True,
                "temporary_password_validity_days": 7
            },
            "account_recovery": "verified_email",
            "advanced_security_mode": "AUDIT"
        }
    },
    "waf": {
        "servizio": "WAF",
        "configurazione": {
            "managed_rules": [
                "AWSManagedRulesCommonRuleSet",
                "AWSManagedRulesKnownBadInputsRuleSet"
            ],
            "rate_limit_rule": {
                "limit": 2000,
                "aggregate_key_type": "IP"
            }
        }
    },
    "ddos_protection": {
        "servizio": "Shield_Standard",
        "configurazione": {
            "automatic": True,
            "cost": 0  # Gratuito
        }
    },
    "threat_detection": {
        "servizio": "GuardDuty",
        "configurazione": {
            "enable": True,
            "finding_publishing_frequency": "FIFTEEN_MINUTES",
            "s3_protection": True,
            "kubernetes_protection": True,  # Se EKS
            "malware_protection": True
        }
    },
    "api_logging": {
        "servizio": "CloudTrail",
        "configurazione": {
            "is_multi_region_trail": True,
            "enable_log_file_validation": True,
            "include_global_service_events": True,
            "read_write_type": "All",
            "data_resources": []  # Aggiungi S3/Lambda se necessario
        }
    },
    "compliance_monitoring": {
        "servizio": "Config",
        "configurazione": {
            "all_supported": True,
            "include_global_resources": True,
            "recording_frequency": "CONTINUOUS"
        }
    }
}

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.6: MESSAGING SERVICE SELECTION
# ══════════════════════════════════════════════════════════════════════════════

MESSAGING_SELECTION = [
    {
        "condizioni": {
            "queue": True,
            "order_guaranteed": False,
            "at_least_once": True
        },
        "servizio": "SQS_Standard",
        "configurazione": {
            "visibility_timeout_seconds": 30,
            "message_retention_seconds": 345600,  # 4 giorni
            "max_message_size_bytes": 262144,  # 256 KB
            "receive_wait_time_seconds": 20,  # Long polling
            "redrive_policy": {
                "max_receive_count": 3,
                "dead_letter_queue": True  # OBBLIGATORIO
            }
        }
    },
    {
        "condizioni": {
            "queue": True,
            "order_guaranteed": True,
            "exactly_once": True
        },
        "servizio": "SQS_FIFO",
        "configurazione": {
            "fifo_queue": True,
            "content_based_deduplication": True,
            "visibility_timeout_seconds": 30,
            "message_retention_seconds": 345600,
            "deduplication_scope": "messageGroup",
            "redrive_policy": {
                "max_receive_count": 3,
                "dead_letter_queue": True
            }
        }
    },
    {
        "condizioni": {
            "pub_sub": True,
            "fan_out": True
        },
        "servizio": "SNS",
        "configurazione": {
            "topic_type": "standard",  # o "fifo" se order needed
            "kms_encryption": True
        }
    },
    {
        "condizioni": {
            "event_bus": True,
            "event_routing": True,
            "filtering": True
        },
        "servizio": "EventBridge",
        "configurazione": {
            "event_bus_type": "custom",  # o "default"
            "archive_enabled": True,
            "archive_retention_days": 30
        }
    },
    {
        "condizioni": {
            "streaming": True,
            "real_time": True,
            "retention_hours": 24  # > 24h retention needed
        },
        "servizio": "Kinesis_Data_Streams",
        "configurazione": {
            "stream_mode": "ON_DEMAND",
            "retention_period_hours": 168,  # 7 giorni
            "encryption_type": "KMS"
        }
    },
    {
        "condizioni": {
            "streaming": True,
            "etl_destination": True  # S3, Redshift, etc
        },
        "servizio": "Kinesis_Firehose",
        "configurazione": {
            "destination": "extended_s3",  # o redshift, opensearch, http
            "buffering_hints": {
                "size_mb": 5,
                "interval_seconds": 300
            },
            "compression": "GZIP",
            "s3_backup_mode": "Disabled"
        }
    },
    {
        "condizioni": {
            "streaming": True,
            "kafka": True
        },
        "servizio": "MSK",
        "configurazione": {
            "kafka_version": "3.5.1",
            "instance_type": "kafka.m5.large",
            "number_of_broker_nodes": 3,
            "ebs_volume_size_gb": 100,
            "encryption_in_transit": "TLS",
            "encryption_at_rest": True
        }
    },
    {
        "condizioni": {
            "workflow": True,
            "orchestration": True,
            "duration_minutes": 5  # <= 5 min
        },
        "servizio": "Step_Functions_Express",
        "configurazione": {
            "type": "EXPRESS",
            "logging_level": "ALL",
            "include_execution_data": True
        }
    },
    {
        "condizioni": {
            "workflow": True,
            "orchestration": True,
            "duration_minutes": 365  # > 5 min (up to 1 year)
        },
        "servizio": "Step_Functions_Standard",
        "configurazione": {
            "type": "STANDARD",
            "logging_level": "ALL",
            "tracing_enabled": True
        }
    }
]

# ══════════════════════════════════════════════════════════════════════════════
# TABELLA 2.7: MONITORING - OBBLIGATORIO TUTTI I PROGETTI
# ══════════════════════════════════════════════════════════════════════════════

MONITORING_MANDATORY = {
    "metrics": {
        "servizio": "CloudWatch_Metrics",
        "configurazione": {
            "namespace": "{project}-{env}",
            "retention_days": 15,  # Free tier: 15 days high-res
            "standard_resolution": True  # 1 minuto
        }
    },
    "logs": {
        "servizio": "CloudWatch_Logs",
        "configurazione": {
            "retention_days": 30,  # MINIMO 30 giorni
            "kms_encryption": True,
            "log_groups": [
                "/aws/lambda/{project}-{env}-*",
                "/aws/apigateway/{project}-{env}-*",
                "/ecs/{project}-{env}/*"
            ]
        }
    },
    "tracing": {
        "servizio": "X_Ray",
        "configurazione": {
            "sampling_rule": {
                "rule_name": "default",
                "priority": 10000,
                "fixed_rate": 0.05,  # 5% sampling
                "reservoir_size": 1,
                "service_name": "*",
                "service_type": "*",
                "host": "*",
                "http_method": "*",
                "url_path": "*"
            }
        }
    },
    "alarms_mandatory": {
        "servizio": "CloudWatch_Alarms",
        "alarms": [
            {
                "name": "{project}-{env}-lambda-errors",
                "metric": "Errors",
                "namespace": "AWS/Lambda",
                "statistic": "Sum",
                "period": 300,
                "threshold": 1,
                "comparison": "GreaterThanOrEqualToThreshold",
                "evaluation_periods": 1
            },
            {
                "name": "{project}-{env}-api-5xx",
                "metric": "5XXError",
                "namespace": "AWS/ApiGateway",
                "statistic": "Sum",
                "period": 300,
                "threshold": 1,
                "comparison": "GreaterThanOrEqualToThreshold",
                "evaluation_periods": 1
            },
            {
                "name": "{project}-{env}-dynamodb-throttle",
                "metric": "ThrottledRequests",
                "namespace": "AWS/DynamoDB",
                "statistic": "Sum",
                "period": 60,
                "threshold": 1,
                "comparison": "GreaterThanOrEqualToThreshold",
                "evaluation_periods": 1
            }
        ]
    }
}

