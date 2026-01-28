# ============================================================================
# CATALOGO DETERMINISTICO AWS v3.0
# ============================================================================
# TIPO: ALGORITMO ESEGUIBILE
# TARGET: Modelli AI (Claude, GPT, etc.)
# REGOLA: ESEGUI ISTRUZIONI LETTERALMENTE - MAI INTERPRETARE
# ============================================================================

"""
ISTRUZIONI PER IL MODELLO AI:

1. LEGGI questo documento come CODICE, non come documentazione
2. ESEGUI ogni istruzione LETTERALMENTE
3. NON aggiungere servizi non specificati
4. NON rimuovere servizi marcati OBBLIGATORIO
5. NON modificare valori specificati
6. SE una condizione non è coperta → CHIEDI all'utente
7. MAI usare parole: "consiglio", "suggerirei", "potresti", "considera"
8. SEMPRE usare: "DEVI", "ESEGUI", "IL VALORE È", "LA RISPOSTA È"
"""

# ============================================================================
# SEZIONE 0: ALGORITMO PRINCIPALE
# ============================================================================

def PROCESSA_RICHIESTA_INFRASTRUTTURA(user_input: str) -> dict:
    """
    ENTRY POINT - Esegui questa funzione per ogni richiesta infrastruttura.
    
    Args:
        user_input: Testo della richiesta utente
        
    Returns:
        dict con architettura completa
    """
    
    # STEP 1: Estrai requisiti
    requisiti = ESTRAI_REQUISITI(user_input)
    
    # STEP 2: Determina architettura
    arch_id = SELEZIONA_ARCHITETTURA(requisiti)
    
    # STEP 3: Ottieni stack
    stack = OTTIENI_STACK(arch_id)
    
    # STEP 4: Configura parametri
    stack_configurato = CONFIGURA_PARAMETRI(stack, requisiti)
    
    # STEP 5: Genera output
    output = GENERA_OUTPUT(stack_configurato)
    
    return output

# ============================================================================
# SEZIONE 1: ESTRAZIONE REQUISITI
# ============================================================================

def ESTRAI_REQUISITI(user_input: str) -> dict:
    """
    Estrae requisiti dal testo utente usando pattern matching.
    NON interpretare - usa solo le keyword definite.
    """
    
    input_lower = user_input.lower()
    requisiti = {}
    
    # DOMANDA 1: COMPLIANCE
    HIPAA_KEYWORDS = ["hipaa", "phi", "ephi", "paziente", "patient", "medico", 
                      "doctor", "ospedale", "hospital", "clinica", "clinic",
                      "diagnosi", "diagnosis", "cartella clinica", "medical record",
                      "healthcare", "sanità", "salute", "health"]
    
    PCI_KEYWORDS = ["pci", "pci-dss", "carta di credito", "credit card", 
                    "payment card", "pan", "cvv", "cardholder", "pagamento",
                    "transazione finanziaria", "acquiring", "issuing"]
    
    requisiti["compliance_hipaa"] = any(kw in input_lower for kw in HIPAA_KEYWORDS)
    requisiti["compliance_pci"] = any(kw in input_lower for kw in PCI_KEYWORDS)
    
    # DOMANDA 2: TIPO APPLICAZIONE
    GAMING_KEYWORDS = ["game", "gioco", "multiplayer", "matchmaking", 
                       "player", "lobby", "game server", "gaming"]
    
    VIDEO_KEYWORDS = ["video streaming", "live streaming", "vod", "hls", 
                      "dash", "transcode", "broadcast", "ott", "netflix",
                      "twitch", "youtube"]
    
    IOT_KEYWORDS = ["iot", "sensor", "sensore", "device", "dispositivo",
                    "mqtt", "telemetry", "telemetria", "edge"]
    
    IOT_INDUSTRIAL_KEYWORDS = ["industrial", "industriale", "factory", 
                               "fabbrica", "manufacturing", "opc-ua", "plc",
                               "scada", "predictive maintenance", "mes"]
    
    CHAT_KEYWORDS = ["chat", "messaging", "messaggistica", "real-time",
                     "websocket", "presenza", "presence", "typing",
                     "social network", "feed", "notifiche push"]
    
    SAAS_KEYWORDS = ["multi-tenant", "saas", "tenant", "white-label",
                     "b2b platform", "piattaforma b2b"]
    
    ANALYTICS_KEYWORDS = ["data lake", "data warehouse", "etl", "analytics",
                          "bi", "business intelligence", "lakehouse", "iceberg"]
    
    ML_KEYWORDS = ["mlops", "machine learning", "ml pipeline", "training",
                   "inference", "sagemaker", "model", "modello ml"]
    
    K8S_KEYWORDS = ["kubernetes", "k8s", "eks", "kubectl", "helm",
                    "container orchestration"]
    
    ECOMMERCE_KEYWORDS = ["e-commerce", "ecommerce", "shop", "negozio",
                          "cart", "carrello", "checkout", "product catalog",
                          "catalogo prodotti", "order", "ordine", "inventory"]
    
    requisiti["is_gaming"] = any(kw in input_lower for kw in GAMING_KEYWORDS)
    requisiti["is_video"] = any(kw in input_lower for kw in VIDEO_KEYWORDS)
    requisiti["is_iot"] = any(kw in input_lower for kw in IOT_KEYWORDS)
    requisiti["is_iot_industrial"] = any(kw in input_lower for kw in IOT_INDUSTRIAL_KEYWORDS)
    requisiti["is_chat"] = any(kw in input_lower for kw in CHAT_KEYWORDS)
    requisiti["is_saas"] = any(kw in input_lower for kw in SAAS_KEYWORDS)
    requisiti["is_analytics"] = any(kw in input_lower for kw in ANALYTICS_KEYWORDS)
    requisiti["is_ml"] = any(kw in input_lower for kw in ML_KEYWORDS)
    requisiti["is_k8s"] = any(kw in input_lower for kw in K8S_KEYWORDS)
    requisiti["is_ecommerce"] = any(kw in input_lower for kw in ECOMMERCE_KEYWORDS)
    
    # DOMANDA 3: AMBIENTE
    if "prod" in input_lower or "production" in input_lower:
        requisiti["environment"] = "prod"
    elif "staging" in input_lower or "stage" in input_lower:
        requisiti["environment"] = "staging"
    else:
        requisiti["environment"] = "dev"
    
    # DOMANDA 4: REGION
    REGION_MAP = {
        "milano": "eu-south-1",
        "milan": "eu-south-1",
        "italia": "eu-south-1",
        "italy": "eu-south-1",
        "frankfurt": "eu-central-1",
        "francoforte": "eu-central-1",
        "ireland": "eu-west-1",
        "irlanda": "eu-west-1",
        "london": "eu-west-2",
        "londra": "eu-west-2",
        "paris": "eu-west-3",
        "parigi": "eu-west-3",
        "virginia": "us-east-1",
        "oregon": "us-west-2"
    }
    
    requisiti["region"] = "eu-south-1"  # DEFAULT
    for keyword, region in REGION_MAP.items():
        if keyword in input_lower:
            requisiti["region"] = region
            break
    
    return requisiti

# ============================================================================
# SEZIONE 2: SELEZIONE ARCHITETTURA (DECISION TREE)
# ============================================================================

def SELEZIONA_ARCHITETTURA(requisiti: dict) -> str:
    """
    Decision tree BINARIO per selezionare l'architettura.
    ESEGUI dall'alto verso il basso, FERMATI alla prima condizione TRUE.
    
    Returns:
        arch_id: Identificatore architettura (es: "ARCH-005")
    """
    
    # NODO 1: HIPAA?
    if requisiti.get("compliance_hipaa") == True:
        return "ARCH-005"  # Healthcare HIPAA
    
    # NODO 2: PCI-DSS?
    if requisiti.get("compliance_pci") == True:
        return "ARCH-006"  # FinTech PCI-DSS
    
    # NODO 3: Gaming?
    if requisiti.get("is_gaming") == True:
        return "ARCH-004"  # Gaming Multiplayer
    
    # NODO 4: Video Streaming?
    if requisiti.get("is_video") == True:
        return "ARCH-003"  # Video Streaming
    
    # NODO 5: IoT?
    if requisiti.get("is_iot") == True:
        if requisiti.get("is_iot_industrial") == True:
            return "ARCH-007"  # IoT Industrial
        else:
            return "ARCH-008"  # IoT Consumer
    
    # NODO 6: Chat/Real-time?
    if requisiti.get("is_chat") == True:
        return "ARCH-002"  # Social/Chat
    
    # NODO 7: SaaS Multi-tenant?
    if requisiti.get("is_saas") == True:
        return "ARCH-009"  # SaaS Multi-Tenant
    
    # NODO 8: Analytics/Data Lake?
    if requisiti.get("is_analytics") == True:
        return "ARCH-010"  # Data Lake
    
    # NODO 9: MLOps?
    if requisiti.get("is_ml") == True:
        return "ARCH-011"  # MLOps
    
    # NODO 10: Kubernetes richiesto?
    if requisiti.get("is_k8s") == True:
        return "ARCH-014"  # EKS
    
    # NODO 11: E-commerce?
    if requisiti.get("is_ecommerce") == True:
        return "ARCH-001"  # E-Commerce
    
    # NODO 12: DEFAULT
    return "ARCH-013"  # Serverless Microservices


# ═══════════════════════════════════════════════════════════════════════════════
# 3.6 ARCH-006: FINTECH PCI-DSS
# ═══════════════════════════════════════════════════════════════════════════════

ARCH_006 = {
    "nome": "FinTech PCI-DSS",
    "descrizione": "Piattaforma pagamenti con conformità PCI-DSS Level 1",
    "risorse_totali": 26,
    "costo_mensile_stimato_usd": 600,
    "prerequisiti": [
        "Account AWS PCI DSS compliant (AOC disponibile)",
        "QSA (Qualified Security Assessor) ingaggiato",
        "Penetration testing autorizzato",
        "VPC dedicata separata da altri workload"
    ],
    
    "risorse": [
        # ═══════════════════════════════════════════════════════════════════════
        # VPC CDE (Cardholder Data Environment) - ISOLAMENTO TOTALE
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 1,
            "nome": "{project}-{env}-cde-vpc",
            "tipo": "AWS::EC2::VPC",
            "obbligatorio": True,
            "configurazione": {
                "CidrBlock": "10.100.0.0/16",
                "EnableDnsHostnames": True,
                "EnableDnsSupport": True,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-cde-vpc"},
                    {"Key": "PCI-DSS", "Value": "CDE"},
                    {"Key": "Compliance", "Value": "PCI-DSS-3.2.1"}
                ]
            }
        },
        {
            "id": 2,
            "nome": "{project}-{env}-dmz-subnet-a",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.1.0/24",
                "AvailabilityZone": "!Select [0, !GetAZs '']",
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-dmz-a"}, {"Key": "Tier", "Value": "DMZ"}]
            }
        },
        {
            "id": 3,
            "nome": "{project}-{env}-dmz-subnet-b",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.2.0/24",
                "AvailabilityZone": "!Select [1, !GetAZs '']",
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-dmz-b"}, {"Key": "Tier", "Value": "DMZ"}]
            }
        },
        {
            "id": 4,
            "nome": "{project}-{env}-app-subnet-a",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.10.0/24",
                "AvailabilityZone": "!Select [0, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-app-a"}, {"Key": "Tier", "Value": "Application"}]
            }
        },
        {
            "id": 5,
            "nome": "{project}-{env}-app-subnet-b",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.20.0/24",
                "AvailabilityZone": "!Select [1, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-app-b"}, {"Key": "Tier", "Value": "Application"}]
            }
        },
        {
            "id": 6,
            "nome": "{project}-{env}-data-subnet-a",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.100.0/24",
                "AvailabilityZone": "!Select [0, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-data-a"},
                    {"Key": "Tier", "Value": "Data"},
                    {"Key": "PCI-DSS", "Value": "CDE-Data"}
                ]
            }
        },
        {
            "id": 7,
            "nome": "{project}-{env}-data-subnet-b",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "CidrBlock": "10.100.200.0/24",
                "AvailabilityZone": "!Select [1, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-data-b"},
                    {"Key": "Tier", "Value": "Data"},
                    {"Key": "PCI-DSS", "Value": "CDE-Data"}
                ]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # NACL - SEGMENTAZIONE TIER (PCI-DSS Requirement 1)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 8,
            "nome": "{project}-{env}-dmz-nacl",
            "tipo": "AWS::EC2::NetworkAcl",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-dmz-nacl"}]
            }
        },
        {
            "id": 9,
            "nome": "{project}-{env}-dmz-nacl-inbound-https",
            "tipo": "AWS::EC2::NetworkAclEntry",
            "obbligatorio": True,
            "configurazione": {
                "NetworkAclId": "!Ref DMZNACL",
                "RuleNumber": 100,
                "Protocol": 6,
                "RuleAction": "allow",
                "Egress": False,
                "CidrBlock": "0.0.0.0/0",
                "PortRange": {"From": 443, "To": 443}
            }
        },
        {
            "id": 10,
            "nome": "{project}-{env}-data-nacl",
            "tipo": "AWS::EC2::NetworkAcl",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref CDEVPC",
                "Tags": [{"Key": "Name", "Value": "{project}-{env}-data-nacl"}]
            }
        },
        {
            "id": 11,
            "nome": "{project}-{env}-data-nacl-inbound",
            "tipo": "AWS::EC2::NetworkAclEntry",
            "obbligatorio": True,
            "configurazione": {
                "NetworkAclId": "!Ref DataNACL",
                "RuleNumber": 100,
                "Protocol": 6,
                "RuleAction": "allow",
                "Egress": False,
                "CidrBlock": "10.100.10.0/24",
                "PortRange": {"From": 5432, "To": 5432}
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # KMS & HSM - ENCRYPTION (PCI-DSS Requirement 3 & 4)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 12,
            "nome": "{project}-{env}-pan-key",
            "tipo": "AWS::KMS::Key",
            "obbligatorio": True,
            "configurazione": {
                "Description": "KMS CMK for PAN tokenization",
                "EnableKeyRotation": True,
                "KeyPolicy": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Sid": "AllowAdminAccess",
                            "Effect": "Allow",
                            "Principal": {"AWS": "!Sub arn:aws:iam::${AWS::AccountId}:root"},
                            "Action": "kms:*",
                            "Resource": "*"
                        },
                        {
                            "Sid": "AllowTokenizationService",
                            "Effect": "Allow",
                            "Principal": {"AWS": "!GetAtt TokenizationRole.Arn"},
                            "Action": ["kms:Encrypt", "kms:Decrypt", "kms:GenerateDataKey"],
                            "Resource": "*"
                        }
                    ]
                },
                "Tags": [{"Key": "PCI-DSS", "Value": "Requirement-3"}]
            }
        },
        {
            "id": 13,
            "nome": "{project}-{env}-cloudhsm-cluster",
            "tipo": "AWS::CloudHSM::Cluster",
            "obbligatorio": False,
            "condizione": "pci_pin_required = true",
            "configurazione": {
                "HsmType": "hsm1.medium",
                "SubnetIds": ["!Ref DataSubnetA", "!Ref DataSubnetB"],
                "Tags": [{"Key": "PCI-DSS", "Value": "PIN-Translation"}]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # DATABASE - TOKENIZZAZIONE (MAI PAN IN CHIARO)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 14,
            "nome": "{project}-{env}-token-vault",
            "tipo": "AWS::DynamoDB::Table",
            "obbligatorio": True,
            "configurazione": {
                "TableName": "{project}-{env}-token-vault",
                "BillingMode": "PAY_PER_REQUEST",
                "AttributeDefinitions": [
                    {"AttributeName": "token", "AttributeType": "S"}
                ],
                "KeySchema": [
                    {"AttributeName": "token", "KeyType": "HASH"}
                ],
                "SSESpecification": {
                    "SSEEnabled": True,
                    "SSEType": "KMS",
                    "KMSMasterKeyId": "!Ref PANKey"
                },
                "PointInTimeRecoverySpecification": {"PointInTimeRecoveryEnabled": True},
                "Tags": [
                    {"Key": "PCI-DSS", "Value": "Token-Vault"},
                    {"Key": "Data-Classification", "Value": "CRITICAL"}
                ]
            }
        },
        {
            "id": 15,
            "nome": "{project}-{env}-transactions-db",
            "tipo": "AWS::RDS::DBCluster",
            "obbligatorio": True,
            "configurazione": {
                "DBClusterIdentifier": "{project}-{env}-transactions",
                "Engine": "aurora-postgresql",
                "EngineVersion": "15.4",
                "DatabaseName": "transactions",
                "MasterUsername": "!Join ['', ['{{resolve:secretsmanager:', !Ref DBSecret, ':SecretString:username}}']]",
                "MasterUserPassword": "!Join ['', ['{{resolve:secretsmanager:', !Ref DBSecret, ':SecretString:password}}']]",
                "StorageEncrypted": True,
                "KmsKeyId": "!Ref PANKey",
                "DeletionProtection": True,
                "BackupRetentionPeriod": 35,
                "DBSubnetGroupName": "!Ref DataSubnetGroup",
                "VpcSecurityGroupIds": ["!Ref DBSecurityGroup"],
                "EnableCloudwatchLogsExports": ["postgresql"],
                "Tags": [{"Key": "PCI-DSS", "Value": "CDE-Database"}]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # COMPUTE - EKS PER PAYMENT PROCESSING
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 16,
            "nome": "{project}-{env}-eks-cluster",
            "tipo": "AWS::EKS::Cluster",
            "obbligatorio": True,
            "configurazione": {
                "Name": "{project}-{env}-payment-cluster",
                "Version": "1.29",
                "RoleArn": "!GetAtt EKSRole.Arn",
                "ResourcesVpcConfig": {
                    "SubnetIds": ["!Ref AppSubnetA", "!Ref AppSubnetB"],
                    "SecurityGroupIds": ["!Ref EKSSecurityGroup"],
                    "EndpointPrivateAccess": True,
                    "EndpointPublicAccess": False
                },
                "EncryptionConfig": [{
                    "Provider": {"KeyArn": "!GetAtt PANKey.Arn"},
                    "Resources": ["secrets"]
                }],
                "Logging": {
                    "ClusterLogging": {
                        "EnabledTypes": [
                            {"Type": "api"},
                            {"Type": "audit"},
                            {"Type": "authenticator"},
                            {"Type": "controllerManager"},
                            {"Type": "scheduler"}
                        ]
                    }
                },
                "Tags": [{"Key": "PCI-DSS", "Value": "Payment-Processing"}]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # WAF & SECURITY (PCI-DSS Requirement 6)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 17,
            "nome": "{project}-{env}-waf",
            "tipo": "AWS::WAFv2::WebACL",
            "obbligatorio": True,
            "configurazione": {
                "Name": "{project}-{env}-pci-waf",
                "Scope": "REGIONAL",
                "DefaultAction": {"Block": {}},
                "Rules": [
                    {
                        "Name": "AWSManagedRulesCommonRuleSet",
                        "Priority": 1,
                        "Statement": {
                            "ManagedRuleGroupStatement": {
                                "VendorName": "AWS",
                                "Name": "AWSManagedRulesCommonRuleSet"
                            }
                        },
                        "OverrideAction": {"None": {}},
                        "VisibilityConfig": {
                            "SampledRequestsEnabled": True,
                            "CloudWatchMetricsEnabled": True,
                            "MetricName": "CommonRules"
                        }
                    },
                    {
                        "Name": "AWSManagedRulesSQLiRuleSet",
                        "Priority": 2,
                        "Statement": {
                            "ManagedRuleGroupStatement": {
                                "VendorName": "AWS",
                                "Name": "AWSManagedRulesSQLiRuleSet"
                            }
                        },
                        "OverrideAction": {"None": {}},
                        "VisibilityConfig": {
                            "SampledRequestsEnabled": True,
                            "CloudWatchMetricsEnabled": True,
                            "MetricName": "SQLiRules"
                        }
                    },
                    {
                        "Name": "RateLimit",
                        "Priority": 3,
                        "Statement": {
                            "RateBasedStatement": {
                                "Limit": 1000,
                                "AggregateKeyType": "IP"
                            }
                        },
                        "Action": {"Block": {}},
                        "VisibilityConfig": {
                            "SampledRequestsEnabled": True,
                            "CloudWatchMetricsEnabled": True,
                            "MetricName": "RateLimit"
                        }
                    }
                ],
                "Tags": [{"Key": "PCI-DSS", "Value": "Requirement-6"}]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # COGNITO - ACCESS CONTROL (PCI-DSS Requirement 7 & 8)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 18,
            "nome": "{project}-{env}-user-pool",
            "tipo": "AWS::Cognito::UserPool",
            "obbligatorio": True,
            "configurazione": {
                "UserPoolName": "{project}-{env}-pci-users",
                "MfaConfiguration": "ON",
                "EnabledMfas": ["SOFTWARE_TOKEN_MFA"],
                "Policies": {
                    "PasswordPolicy": {
                        "MinimumLength": 14,
                        "RequireLowercase": True,
                        "RequireUppercase": True,
                        "RequireNumbers": True,
                        "RequireSymbols": True,
                        "TemporaryPasswordValidityDays": 1
                    }
                },
                "UserPoolAddOns": {"AdvancedSecurityMode": "ENFORCED"},
                "AccountRecoverySetting": {
                    "RecoveryMechanisms": [{"Name": "admin_only", "Priority": 1}]
                },
                "AdminCreateUserConfig": {
                    "AllowAdminCreateUserOnly": True
                },
                "Tags": [{"Key": "PCI-DSS", "Value": "Requirement-8"}]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # AUDIT & LOGGING (PCI-DSS Requirement 10)
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 19,
            "nome": "{project}-{env}-audit-bucket",
            "tipo": "AWS::S3::Bucket",
            "obbligatorio": True,
            "configurazione": {
                "BucketName": "{project}-{env}-pci-audit-logs",
                "ObjectLockEnabled": True,
                "ObjectLockConfiguration": {
                    "ObjectLockEnabled": "Enabled",
                    "Rule": {
                        "DefaultRetention": {
                            "Mode": "COMPLIANCE",
                            "Years": 1
                        }
                    }
                },
                "VersioningConfiguration": {"Status": "Enabled"},
                "BucketEncryption": {
                    "ServerSideEncryptionConfiguration": [{
                        "ServerSideEncryptionByDefault": {
                            "SSEAlgorithm": "aws:kms",
                            "KMSMasterKeyID": "!Ref PANKey"
                        }
                    }]
                },
                "PublicAccessBlockConfiguration": {
                    "BlockPublicAcls": True,
                    "BlockPublicPolicy": True,
                    "IgnorePublicAcls": True,
                    "RestrictPublicBuckets": True
                },
                "Tags": [{"Key": "PCI-DSS", "Value": "Requirement-10"}]
            }
        },
        {
            "id": 20,
            "nome": "{project}-{env}-cloudtrail",
            "tipo": "AWS::CloudTrail::Trail",
            "obbligatorio": True,
            "configurazione": {
                "TrailName": "{project}-{env}-pci-trail",
                "IsMultiRegionTrail": True,
                "EnableLogFileValidation": True,
                "IncludeGlobalServiceEvents": True,
                "S3BucketName": "!Ref AuditBucket",
                "KMSKeyId": "!Ref PANKey",
                "EventSelectors": [
                    {
                        "ReadWriteType": "All",
                        "IncludeManagementEvents": True,
                        "DataResources": [
                            {"Type": "AWS::S3::Object", "Values": ["!Sub 'arn:aws:s3:::${TokenVault}/*'"]},
                            {"Type": "AWS::Lambda::Function", "Values": ["arn:aws:lambda"]}
                        ]
                    }
                ],
                "Tags": [{"Key": "PCI-DSS", "Value": "Requirement-10"}]
            }
        },
        {
            "id": 21,
            "nome": "{project}-{env}-config",
            "tipo": "AWS::Config::ConfigurationRecorder",
            "obbligatorio": True,
            "configurazione": {
                "Name": "{project}-{env}-pci-config",
                "RecordingGroup": {
                    "AllSupported": True,
                    "IncludeGlobalResourceTypes": True
                },
                "RoleARN": "!GetAtt ConfigRole.Arn"
            }
        },
        {
            "id": 22,
            "nome": "{project}-{env}-securityhub",
            "tipo": "AWS::SecurityHub::Hub",
            "obbligatorio": True,
            "configurazione": {
                "Tags": {"PCI-DSS": "Compliance-Dashboard"}
            }
        },
        {
            "id": 23,
            "nome": "{project}-{env}-securityhub-pci-standard",
            "tipo": "AWS::SecurityHub::Standard",
            "obbligatorio": True,
            "configurazione": {
                "StandardsArn": "arn:aws:securityhub:::ruleset/pci-dss/v/3.2.1"
            }
        }
    ],
    
    # ═══════════════════════════════════════════════════════════════════════
    # FLOW TOKENIZZAZIONE - SEQUENZA OBBLIGATORIA
    # ═══════════════════════════════════════════════════════════════════════
    "tokenization_flow": """
    SEQUENZA OBBLIGATORIA (MAI DEVIARE):
    
    1. CARD DATA ARRIVES
       ├── Input: PAN, CVV, Expiry
       ├── Channel: TLS 1.2+ ONLY
       └── Destination: API Gateway (WAF protected)
    
    2. TOKENIZATION SERVICE
       ├── Lambda receives card data
       ├── IMMEDIATELY encrypt PAN with KMS
       ├── Generate unique token (UUID v4)
       ├── Store mapping: token → encrypted_PAN in DynamoDB
       └── Return token to client
    
    3. STORAGE RULES
       ├── NEVER store CVV (anywhere, ever)
       ├── NEVER store PAN in clear text
       ├── NEVER log PAN, CVV, PIN
       └── Token is ONLY reference to card
    
    4. PAYMENT PROCESSING
       ├── Application uses TOKEN only
       ├── Payment service retrieves encrypted PAN
       ├── Decrypts with KMS (audit logged)
       ├── Sends to payment gateway
       └── Response uses token reference
    
    REGOLA ASSOLUTA: SE TROVI PAN IN CHIARO OVUNQUE → SECURITY INCIDENT
    """
}


# ═══════════════════════════════════════════════════════════════════════════════
# 3.9 ARCH-009: SAAS MULTI-TENANT
# ═══════════════════════════════════════════════════════════════════════════════

"""
DECISION TREE PER ISOLATION MODEL:

ESEGUI IN ORDINE:
SE compliance IN [HIPAA, PCI-DSS, SOC2-Type2]:
    model = "SILO"
ELSE SE customer_revenue_monthly > 10000:
    model = "SILO"
ELSE SE noisy_neighbor_risk = "critical":
    model = "BRIDGE"
ELSE:
    model = "POOL"
"""

ARCH_009 = {
    "nome": "SaaS Multi-Tenant",
    "descrizione": "Piattaforma SaaS con isolamento tenant configurabile",
    "modelli_disponibili": ["POOL", "BRIDGE", "SILO"],
    
    # ═══════════════════════════════════════════════════════════════════════
    # MODELLO POOL - RISORSE CONDIVISE (Default, più economico)
    # ═══════════════════════════════════════════════════════════════════════
    "POOL": {
        "descrizione": "Tutti i tenant condividono le stesse risorse",
        "costo_per_tenant_usd": 5,
        "risorse_totali": 12,
        "risorse": [
            {
                "id": 1,
                "nome": "{project}-{env}-user-pool",
                "tipo": "AWS::Cognito::UserPool",
                "obbligatorio": True,
                "configurazione": {
                    "UserPoolName": "{project}-{env}-tenants",
                    "Schema": [
                        {
                            "Name": "tenant_id",
                            "AttributeDataType": "String",
                            "Mutable": False,
                            "Required": True,
                            "StringAttributeConstraints": {"MinLength": "1", "MaxLength": "64"}
                        },
                        {
                            "Name": "tenant_tier",
                            "AttributeDataType": "String",
                            "Mutable": True,
                            "Required": False
                        }
                    ],
                    "Policies": {
                        "PasswordPolicy": {
                            "MinimumLength": 12,
                            "RequireLowercase": True,
                            "RequireUppercase": True,
                            "RequireNumbers": True,
                            "RequireSymbols": True
                        }
                    }
                }
            },
            {
                "id": 2,
                "nome": "{project}-{env}-api",
                "tipo": "AWS::ApiGatewayV2::Api",
                "obbligatorio": True,
                "configurazione": {
                    "Name": "{project}-{env}-api",
                    "ProtocolType": "HTTP"
                }
            },
            {
                "id": 3,
                "nome": "{project}-{env}-usage-plan-basic",
                "tipo": "AWS::ApiGateway::UsagePlan",
                "obbligatorio": True,
                "configurazione": {
                    "UsagePlanName": "{project}-{env}-basic",
                    "Description": "Basic tier - 1000 requests/day",
                    "Quota": {"Limit": 1000, "Period": "DAY"},
                    "Throttle": {"BurstLimit": 10, "RateLimit": 5}
                }
            },
            {
                "id": 4,
                "nome": "{project}-{env}-usage-plan-pro",
                "tipo": "AWS::ApiGateway::UsagePlan",
                "obbligatorio": True,
                "configurazione": {
                    "UsagePlanName": "{project}-{env}-pro",
                    "Description": "Pro tier - 10000 requests/day",
                    "Quota": {"Limit": 10000, "Period": "DAY"},
                    "Throttle": {"BurstLimit": 100, "RateLimit": 50}
                }
            },
            {
                "id": 5,
                "nome": "{project}-{env}-usage-plan-enterprise",
                "tipo": "AWS::ApiGateway::UsagePlan",
                "obbligatorio": True,
                "configurazione": {
                    "UsagePlanName": "{project}-{env}-enterprise",
                    "Description": "Enterprise tier - unlimited",
                    "Throttle": {"BurstLimit": 1000, "RateLimit": 500}
                }
            },
            {
                "id": 6,
                "nome": "{project}-{env}-tenant-context-layer",
                "tipo": "AWS::Lambda::LayerVersion",
                "obbligatorio": True,
                "configurazione": {
                    "LayerName": "{project}-{env}-tenant-context",
                    "Description": "Tenant isolation and context extraction",
                    "CompatibleRuntimes": ["nodejs20.x", "python3.12"]
                },
                "codice_obbligatorio": """
                    // TENANT CONTEXT LAYER - COPIA ESATTA
                    exports.extractTenantId = (event) => {
                        const claims = event.requestContext?.authorizer?.claims;
                        if (!claims) {
                            throw new Error('UNAUTHORIZED: No claims found');
                        }
                        const tenantId = claims['custom:tenant_id'];
                        if (!tenantId) {
                            throw new Error('UNAUTHORIZED: Missing tenant_id in token');
                        }
                        return tenantId;
                    };
                    
                    exports.validateTenantAccess = (requestTenantId, resourceTenantId) => {
                        if (requestTenantId !== resourceTenantId) {
                            throw new Error('FORBIDDEN: Cross-tenant access attempt');
                        }
                        return true;
                    };
                    
                    exports.addTenantContext = (params, tenantId) => {
                        return {
                            ...params,
                            ExpressionAttributeValues: {
                                ...params.ExpressionAttributeValues,
                                ':tenantId': tenantId
                            },
                            FilterExpression: params.FilterExpression 
                                ? `${params.FilterExpression} AND tenant_id = :tenantId`
                                : 'tenant_id = :tenantId'
                        };
                    };
                """
            },
            {
                "id": 7,
                "nome": "{project}-{env}-data",
                "tipo": "AWS::DynamoDB::Table",
                "obbligatorio": True,
                "configurazione": {
                    "TableName": "{project}-{env}-data",
                    "BillingMode": "PAY_PER_REQUEST",
                    "AttributeDefinitions": [
                        {"AttributeName": "PK", "AttributeType": "S"},
                        {"AttributeName": "SK", "AttributeType": "S"},
                        {"AttributeName": "tenant_id", "AttributeType": "S"}
                    ],
                    "KeySchema": [
                        {"AttributeName": "PK", "KeyType": "HASH"},
                        {"AttributeName": "SK", "KeyType": "RANGE"}
                    ],
                    "GlobalSecondaryIndexes": [{
                        "IndexName": "tenant-index",
                        "KeySchema": [
                            {"AttributeName": "tenant_id", "KeyType": "HASH"},
                            {"AttributeName": "SK", "KeyType": "RANGE"}
                        ],
                        "Projection": {"ProjectionType": "ALL"}
                    }]
                },
                "schema_item_obbligatorio": {
                    "PK": "{tenant_id}#{entity_type}#{entity_id}",
                    "SK": "{sort_key}",
                    "tenant_id": "{tenant_id}",
                    "data": "..."
                }
            },
            {
                "id": 8,
                "nome": "{project}-{env}-tenant-data-policy",
                "tipo": "AWS::IAM::Policy",
                "obbligatorio": True,
                "configurazione": {
                    "PolicyName": "{project}-{env}-tenant-isolation-policy",
                    "PolicyDocument": {
                        "Version": "2012-10-17",
                        "Statement": [{
                            "Effect": "Allow",
                            "Action": [
                                "dynamodb:GetItem",
                                "dynamodb:PutItem",
                                "dynamodb:UpdateItem",
                                "dynamodb:DeleteItem",
                                "dynamodb:Query"
                            ],
                            "Resource": "!GetAtt DataTable.Arn",
                            "Condition": {
                                "ForAllValues:StringEquals": {
                                    "dynamodb:LeadingKeys": ["${cognito-identity.amazonaws.com:sub}"]
                                }
                            }
                        }]
                    }
                }
            },
            {
                "id": 9,
                "nome": "{project}-{env}-media-bucket",
                "tipo": "AWS::S3::Bucket",
                "obbligatorio": True,
                "configurazione": {
                    "BucketName": "{project}-{env}-tenant-media"
                },
                "struttura_prefissi": "s3://{bucket}/{tenant_id}/..."
            },
            {
                "id": 10,
                "nome": "{project}-{env}-bucket-policy",
                "tipo": "AWS::S3::BucketPolicy",
                "obbligatorio": True,
                "configurazione": {
                    "Bucket": "!Ref MediaBucket",
                    "PolicyDocument": {
                        "Statement": [{
                            "Effect": "Allow",
                            "Principal": {"AWS": "!GetAtt TenantRole.Arn"},
                            "Action": ["s3:GetObject", "s3:PutObject"],
                            "Resource": "!Sub arn:aws:s3:::${MediaBucket}/${cognito-identity.amazonaws.com:sub}/*",
                            "Condition": {
                                "StringLike": {
                                    "s3:prefix": ["${cognito-identity.amazonaws.com:sub}/*"]
                                }
                            }
                        }]
                    }
                }
            }
        ],
        "tags_obbligatori": {
            "tenant_id": "REQUIRED - tag su ogni risorsa per cost allocation",
            "tier": "basic|pro|enterprise",
            "environment": "dev|staging|prod",
            "project": "{project}"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════
    # MODELLO SILO - RISORSE DEDICATE PER TENANT
    # ═══════════════════════════════════════════════════════════════════════
    "SILO": {
        "descrizione": "Ogni tenant ha risorse dedicate isolate",
        "costo_per_tenant_usd": 150,
        "quando_usare": [
            "Compliance HIPAA/PCI-DSS",
            "Enterprise customer > $10K/month",
            "Requisiti di isolamento regolamentari"
        ],
        "risorse_per_tenant": [
            {
                "id": 1,
                "nome": "{project}-{env}-{tenant_id}-user-pool",
                "tipo": "AWS::Cognito::UserPool",
                "per_tenant": True
            },
            {
                "id": 2,
                "nome": "{project}-{env}-{tenant_id}-api",
                "tipo": "AWS::ApiGatewayV2::Api",
                "per_tenant": True
            },
            {
                "id": 3,
                "nome": "{project}-{env}-{tenant_id}-data",
                "tipo": "AWS::DynamoDB::Table",
                "per_tenant": True
            },
            {
                "id": 4,
                "nome": "{project}-{env}-{tenant_id}-bucket",
                "tipo": "AWS::S3::Bucket",
                "per_tenant": True
            },
            {
                "id": 5,
                "nome": "{project}-{env}-{tenant_id}-kms-key",
                "tipo": "AWS::KMS::Key",
                "per_tenant": True
            },
            {
                "id": 6,
                "nome": "{project}-{env}-{tenant_id}-logs",
                "tipo": "AWS::Logs::LogGroup",
                "per_tenant": True
            }
        ],
        "onboarding_automation": {
            "tipo": "AWS::CodePipeline::Pipeline",
            "descrizione": "Pipeline per creare stack tenant automaticamente",
            "input_parameters": ["tenant_id", "tier", "config"]
        }
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# 3.14 ARCH-014: CONTAINER ORCHESTRATION EKS
# ═══════════════════════════════════════════════════════════════════════════════

ARCH_014 = {
    "nome": "Container Orchestration EKS",
    "descrizione": "Cluster Kubernetes gestito con EKS",
    "risorse_totali": 18,
    "costo_mensile_stimato_usd": 400,
    
    "risorse": [
        # ═══════════════════════════════════════════════════════════════════════
        # VPC PER EKS
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 1,
            "nome": "{project}-{env}-vpc",
            "tipo": "AWS::EC2::VPC",
            "obbligatorio": True,
            "configurazione": {
                "CidrBlock": "10.0.0.0/16",
                "EnableDnsHostnames": True,
                "EnableDnsSupport": True,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-vpc"},
                    {"Key": "kubernetes.io/cluster/{project}-{env}", "Value": "shared"}
                ]
            }
        },
        {
            "id": 2,
            "nome": "{project}-{env}-public-subnet-a",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref VPC",
                "CidrBlock": "10.0.1.0/24",
                "AvailabilityZone": "!Select [0, !GetAZs '']",
                "MapPublicIpOnLaunch": True,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-public-a"},
                    {"Key": "kubernetes.io/role/elb", "Value": "1"},
                    {"Key": "kubernetes.io/cluster/{project}-{env}", "Value": "shared"}
                ]
            }
        },
        {
            "id": 3,
            "nome": "{project}-{env}-public-subnet-b",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref VPC",
                "CidrBlock": "10.0.2.0/24",
                "AvailabilityZone": "!Select [1, !GetAZs '']",
                "MapPublicIpOnLaunch": True,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-public-b"},
                    {"Key": "kubernetes.io/role/elb", "Value": "1"},
                    {"Key": "kubernetes.io/cluster/{project}-{env}", "Value": "shared"}
                ]
            }
        },
        {
            "id": 4,
            "nome": "{project}-{env}-private-subnet-a",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref VPC",
                "CidrBlock": "10.0.10.0/24",
                "AvailabilityZone": "!Select [0, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-private-a"},
                    {"Key": "kubernetes.io/role/internal-elb", "Value": "1"},
                    {"Key": "kubernetes.io/cluster/{project}-{env}", "Value": "shared"}
                ]
            }
        },
        {
            "id": 5,
            "nome": "{project}-{env}-private-subnet-b",
            "tipo": "AWS::EC2::Subnet",
            "obbligatorio": True,
            "configurazione": {
                "VpcId": "!Ref VPC",
                "CidrBlock": "10.0.20.0/24",
                "AvailabilityZone": "!Select [1, !GetAZs '']",
                "MapPublicIpOnLaunch": False,
                "Tags": [
                    {"Key": "Name", "Value": "{project}-{env}-private-b"},
                    {"Key": "kubernetes.io/role/internal-elb", "Value": "1"},
                    {"Key": "kubernetes.io/cluster/{project}-{env}", "Value": "shared"}
                ]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # EKS CLUSTER
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 6,
            "nome": "{project}-{env}-eks-cluster",
            "tipo": "AWS::EKS::Cluster",
            "obbligatorio": True,
            "configurazione": {
                "Name": "{project}-{env}",
                "Version": "1.29",
                "RoleArn": "!GetAtt EKSClusterRole.Arn",
                "ResourcesVpcConfig": {
                    "SubnetIds": [
                        "!Ref PrivateSubnetA",
                        "!Ref PrivateSubnetB",
                        "!Ref PublicSubnetA",
                        "!Ref PublicSubnetB"
                    ],
                    "SecurityGroupIds": ["!Ref ClusterSecurityGroup"],
                    "EndpointPrivateAccess": True,
                    "EndpointPublicAccess": True,
                    "PublicAccessCidrs": ["0.0.0.0/0"]
                },
                "Logging": {
                    "ClusterLogging": {
                        "EnabledTypes": [
                            {"Type": "api"},
                            {"Type": "audit"},
                            {"Type": "authenticator"},
                            {"Type": "controllerManager"},
                            {"Type": "scheduler"}
                        ]
                    }
                },
                "EncryptionConfig": [{
                    "Provider": {"KeyArn": "!GetAtt KMSKey.Arn"},
                    "Resources": ["secrets"]
                }]
            }
        },
        {
            "id": 7,
            "nome": "{project}-{env}-eks-cluster-role",
            "tipo": "AWS::IAM::Role",
            "obbligatorio": True,
            "configurazione": {
                "RoleName": "{project}-{env}-eks-cluster-role",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                        "Effect": "Allow",
                        "Principal": {"Service": "eks.amazonaws.com"},
                        "Action": "sts:AssumeRole"
                    }]
                },
                "ManagedPolicyArns": [
                    "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy",
                    "arn:aws:iam::aws:policy/AmazonEKSVPCResourceController"
                ]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # NODE GROUP
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 8,
            "nome": "{project}-{env}-node-group",
            "tipo": "AWS::EKS::Nodegroup",
            "obbligatorio": True,
            "configurazione": {
                "NodegroupName": "{project}-{env}-nodes",
                "ClusterName": "!Ref EKSCluster",
                "NodeRole": "!GetAtt NodeRole.Arn",
                "Subnets": ["!Ref PrivateSubnetA", "!Ref PrivateSubnetB"],
                "ScalingConfig": {
                    "MinSize": 2,
                    "MaxSize": 10,
                    "DesiredSize": 3
                },
                "InstanceTypes": ["m6g.large"],
                "AmiType": "AL2_ARM_64",
                "DiskSize": 50,
                "Labels": {
                    "role": "worker",
                    "environment": "{env}"
                },
                "Tags": {
                    "kubernetes.io/cluster/{project}-{env}": "owned"
                }
            }
        },
        {
            "id": 9,
            "nome": "{project}-{env}-node-role",
            "tipo": "AWS::IAM::Role",
            "obbligatorio": True,
            "configurazione": {
                "RoleName": "{project}-{env}-eks-node-role",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [{
                        "Effect": "Allow",
                        "Principal": {"Service": "ec2.amazonaws.com"},
                        "Action": "sts:AssumeRole"
                    }]
                },
                "ManagedPolicyArns": [
                    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
                    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
                    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly",
                    "arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore"
                ]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # ADDONS OBBLIGATORI
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 10,
            "nome": "{project}-{env}-vpc-cni",
            "tipo": "AWS::EKS::Addon",
            "obbligatorio": True,
            "configurazione": {
                "AddonName": "vpc-cni",
                "ClusterName": "!Ref EKSCluster",
                "AddonVersion": "v1.16.0-eksbuild.1"
            }
        },
        {
            "id": 11,
            "nome": "{project}-{env}-coredns",
            "tipo": "AWS::EKS::Addon",
            "obbligatorio": True,
            "configurazione": {
                "AddonName": "coredns",
                "ClusterName": "!Ref EKSCluster",
                "AddonVersion": "v1.11.1-eksbuild.4"
            }
        },
        {
            "id": 12,
            "nome": "{project}-{env}-kube-proxy",
            "tipo": "AWS::EKS::Addon",
            "obbligatorio": True,
            "configurazione": {
                "AddonName": "kube-proxy",
                "ClusterName": "!Ref EKSCluster",
                "AddonVersion": "v1.29.0-eksbuild.1"
            }
        },
        {
            "id": 13,
            "nome": "{project}-{env}-ebs-csi",
            "tipo": "AWS::EKS::Addon",
            "obbligatorio": True,
            "configurazione": {
                "AddonName": "aws-ebs-csi-driver",
                "ClusterName": "!Ref EKSCluster",
                "ServiceAccountRoleArn": "!GetAtt EBSCSIRole.Arn"
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # LOAD BALANCER CONTROLLER
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 14,
            "nome": "{project}-{env}-alb-controller-policy",
            "tipo": "AWS::IAM::Policy",
            "obbligatorio": True,
            "configurazione": {
                "PolicyName": "{project}-{env}-alb-controller-policy",
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Action": [
                                "elasticloadbalancing:*",
                                "ec2:Describe*",
                                "ec2:AuthorizeSecurityGroupIngress",
                                "ec2:RevokeSecurityGroupIngress"
                            ],
                            "Resource": "*"
                        }
                    ]
                }
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # MONITORING
        # ═══════════════════════════════════════════════════════════════════════
        {
            "id": 15,
            "nome": "{project}-{env}-container-insights",
            "tipo": "AWS::EKS::Addon",
            "obbligatorio": True,
            "configurazione": {
                "AddonName": "amazon-cloudwatch-observability",
                "ClusterName": "!Ref EKSCluster"
            }
        }
    ],
    
    "helm_charts_consigliati": [
        {
            "nome": "aws-load-balancer-controller",
            "repository": "eks",
            "versione": "1.7.1",
            "namespace": "kube-system"
        },
        {
            "nome": "metrics-server",
            "repository": "metrics-server",
            "versione": "3.12.0",
            "namespace": "kube-system"
        },
        {
            "nome": "external-dns",
            "repository": "external-dns",
            "versione": "1.14.3",
            "namespace": "external-dns"
        }
    ]
}


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 9: ALGORITMO PRINCIPALE - ESECUZIONE COMPLETA
# ═══════════════════════════════════════════════════════════════════════════════

"""
███████████████████████████████████████████████████████████████████████████████
█                                                                             █
█   ALGORITMO MASTER - SEGUI QUESTO FLUSSO PER OGNI RICHIESTA                 █
█                                                                             █
███████████████████████████████████████████████████████████████████████████████

FUNZIONE: processa_richiesta_architettura(input_utente)

INPUT: stringa con requisiti utente
OUTPUT: CloudFormation template + checklist verifiche

"""

def processa_richiesta_architettura(input_utente):
    """
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║ STEP 1: ESTRAI PARAMETRI BASE                                           ║
    ╚═════════════════════════════════════════════════════════════════════════╝
    """
    # 1.1 Estrai nome progetto
    project = estrai_project_name(input_utente)
    # SE non trovato → CHIEDI: "Qual è il nome del progetto?"
    
    # 1.2 Estrai environment
    env = estrai_environment(input_utente)
    # SE non trovato → DEFAULT: "prod"
    # VALORI AMMESSI: ["dev", "staging", "prod"]
    
    # 1.3 Estrai region
    region = estrai_region(input_utente)
    # SE non trovato → DEFAULT: "eu-south-1"
    # SE HIPAA/PCI → VERIFICA region compliance
    
    """
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║ STEP 2: CLASSIFICA ARCHITETTURA                                         ║
    ╚═════════════════════════════════════════════════════════════════════════╝
    """
    # Esegui funzione classifica_requisiti da Sezione 1
    arch_id = classifica_requisiti(input_utente)
    
    # OUTPUT: "ARCH-001" | "ARCH-002" | ... | "ARCH-014"
    
    """
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║ STEP 3: VERIFICA PREREQUISITI                                           ║
    ╚═════════════════════════════════════════════════════════════════════════╝
    """
    prerequisiti = ARCHITETTURE[arch_id].get("prerequisiti", [])
    
    for prerequisito in prerequisiti:
        # CHIEDI conferma all'utente
        risposta = chiedi_utente(f"Confermi: {prerequisito}? (sì/no)")
        if risposta == "no":
            # STOP e spiega cosa fare
            return f"STOP: Prerequisito mancante: {prerequisito}"
    
    """
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║ STEP 4: GENERA STACK RISORSE                                            ║
    ╚═════════════════════════════════════════════════════════════════════════╝
    """
    template = ARCHITETTURE[arch_id]
    risorse = template["risorse"]
    
    risorse_output = []
    for risorsa in risorse:
        # 4.1 Sostituisci placeholder
        nome = risorsa["nome"].replace("{project}", project)
        nome = nome.replace("{env}", env)
        
        # 4.2 Copia configurazione ESATTA
        config = risorsa["configurazione"].copy()
        
        # 4.3 Aggiungi a output
        risorse_output.append({
            "nome": nome,
            "tipo": risorsa["tipo"],
            "configurazione": config
        })
    
    """
    ╔═════════════════════════════════════════════════════════════════════════╗
    ║ STEP 5: GENERA OUTPUT                                                   ║
    ╚═════════════════════════════════════════════════════════════════════════╝
    """
    output = {
        "architettura": arch_id,
        "project": project,
        "env": env,
        "region": region,
        "risorse_totali": len(risorse_output),
        "costo_stimato_usd": template["costo_mensile_stimato_usd"],
        "risorse": risorse_output,
        "checklist": genera_checklist(risorse_output)
    }
    
    return output


# ═══════════════════════════════════════════════════════════════════════════════
# FUNZIONI HELPER
# ═══════════════════════════════════════════════════════════════════════════════

def estrai_project_name(input_utente):
    """
    REGOLE ESTRAZIONE:
    1. Cerca pattern "per {nome}" o "chiamato {nome}" o "progetto {nome}"
    2. SE non trovato → usa prima parola sostantivo
    3. SE ambiguo → CHIEDI
    
    NORMALIZZAZIONE:
    - lowercase
    - sostituisci spazi con -
    - rimuovi caratteri speciali
    - max 20 caratteri
    """
    import re
    
    # Pattern comuni
    patterns = [
        r"progetto\s+(\w+)",
        r"chiamato\s+(\w+)",
        r"per\s+(\w+)",
        r"app\s+(\w+)",
        r"sistema\s+(\w+)"
    ]
    
    for pattern in patterns:
        match = re.search(pattern, input_utente.lower())
        if match:
            nome = match.group(1)
            # Normalizza
            nome = re.sub(r'[^a-z0-9-]', '', nome.lower())
            return nome[:20]
    
    return None  # Richiede input utente


def estrai_environment(input_utente):
    """
    LOOKUP DIRETTO:
    """
    input_lower = input_utente.lower()
    
    if "produzione" in input_lower or "production" in input_lower or "prod" in input_lower:
        return "prod"
    if "staging" in input_lower or "stage" in input_lower or "test" in input_lower:
        return "staging"
    if "sviluppo" in input_lower or "development" in input_lower or "dev" in input_lower:
        return "dev"
    
    return "prod"  # DEFAULT


def estrai_region(input_utente):
    """
    LOOKUP DIRETTO:
    """
    input_lower = input_utente.lower()
    
    REGION_MAP = {
        "milano": "eu-south-1",
        "milan": "eu-south-1",
        "italia": "eu-south-1",
        "italy": "eu-south-1",
        "irlanda": "eu-west-1",
        "ireland": "eu-west-1",
        "francoforte": "eu-central-1",
        "frankfurt": "eu-central-1",
        "parigi": "eu-west-3",
        "paris": "eu-west-3",
        "londra": "eu-west-2",
        "london": "eu-west-2",
        "virginia": "us-east-1",
        "oregon": "us-west-2",
        "california": "us-west-1"
    }
    
    for keyword, region in REGION_MAP.items():
        if keyword in input_lower:
            return region
    
    return "eu-south-1"  # DEFAULT


# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 10: MAPPA ARCHITETTURE - REFERENCE COMPLETA
# ═══════════════════════════════════════════════════════════════════════════════

ARCHITETTURE_SUMMARY = """
┌──────────────┬────────────────────────────────┬──────────┬───────────────────────────┐
│ ARCH_ID      │ NOME                           │ RISORSE  │ COSTO USD/MESE            │
├──────────────┼────────────────────────────────┼──────────┼───────────────────────────┤
│ ARCH-001     │ E-Commerce Serverless          │ 22       │ ~65                       │
│ ARCH-002     │ Social Media / Chat            │ 18       │ ~120                      │
│ ARCH-003     │ Video Streaming                │ 14       │ ~800                      │
│ ARCH-004     │ Gaming Multiplayer             │ 16       │ ~350                      │
│ ARCH-005     │ Healthcare HIPAA               │ 28       │ ~450                      │
│ ARCH-006     │ FinTech PCI-DSS                │ 26       │ ~600                      │
│ ARCH-007     │ IoT Industrial                 │ 18       │ ~200                      │
│ ARCH-008     │ IoT Smart Home                 │ 12       │ ~100                      │
│ ARCH-009     │ SaaS Multi-Tenant              │ 12-∞     │ ~5/tenant (pool)          │
│ ARCH-010     │ Data Lake / Lakehouse          │ 15       │ ~300                      │
│ ARCH-011     │ MLOps Pipeline                 │ 14       │ ~500                      │
│ ARCH-012     │ Disaster Recovery              │ 3-13     │ 10-200% di primary        │
│ ARCH-013     │ Serverless Microservices (DEF) │ 14       │ ~50                       │
│ ARCH-014     │ Container Orchestration EKS    │ 18       │ ~400                      │
└──────────────┴────────────────────────────────┴──────────┴───────────────────────────┘
"""

KEYWORD_TO_ARCH_QUICK_REFERENCE = """
┌────────────────────────────────────────────────────────────────────────────────┐
│ KEYWORD → ARCH_ID (ordine priorità)                                            │
├────────────────────────────────────────────────────────────────────────────────┤
│ PRIORITÀ 1 (COMPLIANCE - sempre prima):                                        │
│   hipaa, paziente, medico, clinica, healthcare     → ARCH-005                  │
│   pci, carta credito, payment, pagamenti           → ARCH-006                  │
├────────────────────────────────────────────────────────────────────────────────┤
│ PRIORITÀ 2 (DOMAIN-SPECIFIC):                                                  │
│   multiplayer, game server, matchmaking            → ARCH-004                  │
│   video streaming, vod, hls, transcode             → ARCH-003                  │
│   iot + industrial, opc-ua, plc, scada             → ARCH-007                  │
│   iot, sensor, smart home, alexa                   → ARCH-008                  │
├────────────────────────────────────────────────────────────────────────────────┤
│ PRIORITÀ 3 (ARCHITECTURE PATTERNS):                                            │
│   chat, websocket, real-time, social               → ARCH-002                  │
│   multi-tenant, saas, tenant                       → ARCH-009                  │
│   data lake, etl, analytics, bi                    → ARCH-010                  │
│   mlops, sagemaker, model training                 → ARCH-011                  │
│   kubernetes, k8s, eks, helm                       → ARCH-014                  │
├────────────────────────────────────────────────────────────────────────────────┤
│ PRIORITÀ 4 (BUSINESS):                                                         │
│   e-commerce, shop, carrello, checkout             → ARCH-001                  │
│   disaster recovery, failover, rto, rpo            → ARCH-012                  │
├────────────────────────────────────────────────────────────────────────────────┤
│ DEFAULT (nessun match):                                                        │
│   qualsiasi altro caso                             → ARCH-013                  │
└────────────────────────────────────────────────────────────────────────────────┘
"""

# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 11: REGOLE FINALI - COMPORTAMENTO MODELLO AI
# ═══════════════════════════════════════════════════════════════════════════════

REGOLE_COMPORTAMENTO = """
╔═════════════════════════════════════════════════════════════════════════════════╗
║                         REGOLE ASSOLUTE PER IL MODELLO AI                       ║
╚═════════════════════════════════════════════════════════════════════════════════╝

1. MAI INTERPRETARE
   ❌ "Potresti considerare..."
   ❌ "Ti consiglio di..."
   ❌ "Tipicamente si usa..."
   ✅ "Usa ARCH-005 con configurazione ESATTA dalla Sezione 3.5"

2. MAI MODIFICARE VALORI
   ❌ Cambiare memory_mb da 512 a 256
   ❌ Cambiare timeout da 30 a 60
   ❌ Aggiungere risorse non elencate
   ✅ COPIA configurazione ESATTA dal template

3. MAI SALTARE STEP
   ❌ Saltare verifica prerequisiti
   ❌ Saltare checklist verifica
   ❌ Saltare anti-pattern check
   ✅ ESEGUI ogni step nell'ordine specificato

4. SEMPRE CHIEDERE SE AMBIGUO
   ❌ Assumere il significato
   ❌ Indovinare l'architettura
   ✅ "Specifica se intendi X o Y"

5. SEMPRE USARE NAMING CONVENTION
   ❌ MyApp_Prod_Orders
   ❌ my app orders
   ✅ {project}-{env}-{component}

6. SEMPRE VERIFICARE COMPLIANCE
   SE keyword HIPAA → VERIFICA prerequisiti HIPAA
   SE keyword PCI → VERIFICA prerequisiti PCI-DSS
   SE nessuna compliance → procedi normalmente

7. SEMPRE OUTPUT STRUTTURATO
   ✅ arch_id
   ✅ risorse (lista completa)
   ✅ configurazioni (esatte)
   ✅ costi (stimati)
   ✅ checklist (comandi verifica)

╔═════════════════════════════════════════════════════════════════════════════════╗
║                              FORMATO OUTPUT STANDARD                             ║
╚═════════════════════════════════════════════════════════════════════════════════╝

ARCHITETTURA: {ARCH_ID}
PROJECT: {project}
ENVIRONMENT: {env}
REGION: {region}

RISORSE ({totale}):
1. {nome_risorsa_1} ({tipo})
2. {nome_risorsa_2} ({tipo})
...

COSTO STIMATO: ${costo}/mese

CHECKLIST VERIFICA:
[ ] Comando 1: {comando} → Expected: {valore}
[ ] Comando 2: {comando} → Expected: {valore}
...

╔═════════════════════════════════════════════════════════════════════════════════╗
║                              FINE CATALOGO v3.0                                  ║
╚═════════════════════════════════════════════════════════════════════════════════╝
"""

# ═══════════════════════════════════════════════════════════════════════════════
# INDEX COMPLETO
# ═══════════════════════════════════════════════════════════════════════════════

INDEX = """
CATALOGO AWS DETERMINISTICO v3.0 - INDEX

SEZIONE 0: Regole di esecuzione per il modello AI
SEZIONE 1: Algoritmo classificazione (keyword → ARCH_ID)
SEZIONE 2: Tabelle di lookup (configurazioni esatte)
  - 2.1: Compute
  - 2.2: Database
  - 2.3: Messaging
SEZIONE 3: Stack architetturali completi
  - 3.1: ARCH-001 E-Commerce Serverless
  - 3.2: ARCH-002 Social Media / Chat
  - 3.3: ARCH-003 Video Streaming
  - 3.4: ARCH-004 Gaming Multiplayer
  - 3.5: ARCH-005 Healthcare HIPAA
  - 3.6: ARCH-006 FinTech PCI-DSS
  - 3.7: ARCH-007 IoT Industrial
  - 3.8: ARCH-008 IoT Smart Home
  - 3.9: ARCH-009 SaaS Multi-Tenant
  - 3.10: ARCH-010 Data Lake / Lakehouse
  - 3.11: ARCH-011 MLOps Pipeline
  - 3.12: ARCH-012 Disaster Recovery
  - 3.13: ARCH-013 Serverless Microservices (DEFAULT)
  - 3.14: ARCH-014 Container Orchestration EKS
SEZIONE 4: Checklist di verifica (comandi AWS CLI)
SEZIONE 5: Esempi input → output
SEZIONE 6: Naming convention
SEZIONE 7: Anti-pattern (errori da evitare)
SEZIONE 8: Costi esatti (USD/mese)
SEZIONE 9: Algoritmo principale
SEZIONE 10: Mappa architetture (reference)
SEZIONE 11: Regole finali comportamento AI

TOTALE LINEE: ~3500
ARCHITETTURE: 14
SERVIZI AWS COPERTI: 50+
"""

