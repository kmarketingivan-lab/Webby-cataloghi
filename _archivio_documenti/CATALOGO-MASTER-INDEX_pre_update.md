# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO MASTER INDEX v1.0
# ═══════════════════════════════════════════════════════════════════════════════
#
# PANORAMICA COMPLETA DEI CATALOGHI PIATTAFORMA
# Data creazione: 2026-01-25
#
# ═══════════════════════════════════════════════════════════════════════════════

# ═══════════════════════════════════════════════════════════════════════════════
# RIEPILOGO CATALOGHI
# ═══════════════════════════════════════════════════════════════════════════════

CATALOGHI = {
    
    "CATALOGO-INTERFACCIA-PROMPT-USABILITA-v1.md": {
        "dimensione": "~55 KB",
        "linee": "~1820",
        "scopo": "Interfaccia operativa per usare i cataloghi con Ralph",
        "contenuto": [
            "Template Prompt per Ralph (progetto completo, backend, frontend, infra)",
            "Workflow operativi (nuovo progetto, feature, bug fix, refactoring)",
            "Framework domande preliminari (discovery)",
            "Checklist pre-esecuzione",
            "Troubleshooting completo",
            "Workflow review e fix finale",
            "Esempi pronti all'uso"
        ]
    },
    
    "CATALOGO-DESIGN-REFERENCE-v1.md": {
        "dimensione": "~60 KB",
        "linee": "~1700",
        "scopo": "Reference visivi e layout templates per composizione UI",
        "contenuto": [
            "8 stili di reference (Stripe, Linear, Notion, Vercel, Airbnb, Shopify, Slack, GitHub)",
            "12 layout templates per tipo pagina (Homepage, Dashboard, Listing, Detail, Auth, etc.)",
            "Wireframe testuali ASCII dettagliati",
            "Composizioni complete per 6 categorie (E-commerce, SaaS, Social, Dashboard, Healthcare, FinTech)",
            "Sezione reference utente per screenshot custom",
            "Guida rapida selezione stile"
        ]
    },
    
    "CATALOGO-REQUISITI-FUNZIONALI-v1.md": {
        "dimensione": "~130 KB",
        "linee": "~3100",
        "scopo": "Requisiti funzionali completi per tutti i moduli",
        "contenuto": [
            "7 moduli: Identity, Content, Social, Messaging, Marketplace, Booking, Commerce",
            "Operazioni CRUD per ogni entità",
            "Business rules e vincoli",
            "Composizioni per piattaforme complete"
        ]
    },
    
    "CATALOGO-DATA-MODEL-v1.md": {
        "dimensione": "~148 KB",
        "linee": "~3149",
        "scopo": "Schemi database DynamoDB e Aurora PostgreSQL",
        "contenuto": [
            "DynamoDB single-table design con 3 GSI",
            "Aurora PostgreSQL schemas normalizzati",
            "32 tipi entità mappati",
            "Access patterns documentati",
            "Best practices incluse"
        ]
    },
    
    "CATALOGO-API-v1.md": {
        "dimensione": "~122 KB",
        "linee": "~3566",
        "scopo": "Specifiche OpenAPI per API gratuite",
        "contenuto": [
            "119 endpoint totali",
            "Solo API che NON richiedono servizi a pagamento",
            "Schemas request/response completi",
            "Esempi per ogni endpoint"
        ]
    },
    
    "CATALOGO-CODICE-v1.md": {
        "dimensione": "~130 KB",
        "linee": "~3542",
        "scopo": "Template implementazione serverless AWS",
        "contenuto": [
            "Struttura progetto Python/Lambda",
            "Pattern: Repository, Service, Handler",
            "DynamoDB client completo",
            "Auth service con JWT",
            "CDK infrastructure",
            "Unit e integration tests"
        ]
    },
    
    "CATALOGO-REQUISITI-ARCHITETTURA-v2.md": {
        "dimensione": "~85 KB",
        "linee": "~2200",
        "scopo": "Architettura e pattern di sistema",
        "contenuto": [
            "Layer architetturali (Presentation, Business, Data)",
            "Pattern: Repository, Service, Handler",
            "Security e compliance (HIPAA, PCI-DSS)",
            "Scalabilità e performance",
            "Deployment strategies"
        ]
    },
    
    "CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md": {
        "dimensione": "~65 KB",
        "linee": "~1800",
        "scopo": "Design tokens per UI consistente",
        "contenuto": [
            "Colori (semantic + palette)",
            "Typography (scale, weights)",
            "Spacing (4px grid)",
            "Border radius, shadows",
            "Breakpoints, z-index",
            "Dark mode support"
        ]
    },
    
    "CATALOGO-UI-PATTERN-PRIMITIVI-v1.md": {
        "dimensione": "~241 KB",
        "linee": "~8745",
        "scopo": "Pattern UI universali e specifici per categoria",
        "contenuto": [
            "Pattern Primitivi: Layout, Navigation, Content, Input, Feedback, Overlay, Data, Action",
            "Pattern per Categoria: E-Commerce, SaaS, Dashboard, Social/Chat, Video, Healthcare, FinTech, IoT",
            "Responsive patterns",
            "Animation patterns",
            "Accessibility patterns (WCAG 2.1 AA)"
        ]
    },
    
    "CATALOGO-AWS-DETERMINISTICO-v3.md": {
        "dimensione": "~150 KB",
        "linee": "~4000",
        "scopo": "Infrastruttura AWS deterministica",
        "contenuto": [
            "Servizi Always Free Tier only",
            "Pattern CDK per ogni categoria",
            "Security best practices",
            "Cost optimization",
            "Monitoring e logging"
        ]
    }
}

TOTALE_DOCUMENTAZIONE = {
    "file": 11,
    "cataloghi": [
        "CATALOGO-INTERFACCIA-PROMPT-USABILITA-v1.md",
        "CATALOGO-DESIGN-REFERENCE-v1.md",
        "CATALOGO-REQUISITI-FUNZIONALI-v1.md",
        "CATALOGO-REQUISITI-ARCHITETTURA-v2.md",
        "CATALOGO-DATA-MODEL-v1.md",
        "CATALOGO-API-v1.md",
        "CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md",
        "CATALOGO-UI-PATTERN-PRIMITIVI-v1.md",
        "CATALOGO-CODICE-v1.md",
        "CATALOGO-AWS-DETERMINISTICO-v3.md",
        "CATALOGO-MASTER-INDEX.md"
    ],
    "dimensione_totale": "~850 KB",
    "linee_totali": "~25,000+"
}

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK REFERENCE: MODULI
# ═══════════════════════════════════════════════════════════════════════════════

MODULI = {
    
    "IDENTITY": {
        "descrizione": "Autenticazione e gestione utenti",
        "entità": ["User", "Profile", "Session", "Settings", "VerificationToken"],
        "operazioni_chiave": [
            "Registrazione con email verification",
            "Login con JWT (access + refresh tokens)",
            "Gestione sessioni multiple",
            "Reset password",
            "Profilo pubblico/privato"
        ],
        "endpoints": 18,
        "tabelle_aurora": 5,
        "note": "Completamente implementato nel codice"
    },
    
    "CONTENT": {
        "descrizione": "Post, commenti, reazioni, media",
        "entità": ["Post", "Comment", "Reaction", "MediaItem", "Hashtag"],
        "operazioni_chiave": [
            "CRUD post (text, image, video, link, poll)",
            "Commenti threaded (ltree)",
            "Reazioni multiple (like, love, etc.)",
            "Upload media con presigned URLs",
            "Hashtag trending"
        ],
        "endpoints": 18,
        "tabelle_aurora": 6,
        "note": "Template handler inclusi"
    },
    
    "SOCIAL": {
        "descrizione": "Follow, block, feed",
        "entità": ["Follow", "Block", "UserStats", "FeedItem"],
        "operazioni_chiave": [
            "Follow/unfollow con richieste pending",
            "Block/unblock utenti",
            "Feed personalizzato (fan-out on write)",
            "Statistiche utente",
            "Suggerimenti utenti"
        ],
        "endpoints": 15,
        "tabelle_aurora": 4,
        "note": "Template handler follow inclusi"
    },
    
    "MESSAGING": {
        "descrizione": "Conversazioni e messaggi",
        "entità": ["Conversation", "ConversationParticipant", "Message", "ReadReceipt", "Notification"],
        "operazioni_chiave": [
            "Conversazioni 1:1 e gruppi",
            "Messaggi con reply e reactions",
            "Read receipts",
            "Notifiche in-app (no push)",
            "Typing indicators (WebSocket)"
        ],
        "endpoints": 18,
        "tabelle_aurora": 5,
        "note": "Push notifications escluse (a pagamento)"
    },
    
    "MARKETPLACE": {
        "descrizione": "Annunci e offerte",
        "entità": ["Listing", "Category", "SellerProfile", "Offer", "Review", "Favorite"],
        "operazioni_chiave": [
            "CRUD annunci con categorie",
            "Offerte e negoziazione",
            "Profilo venditore con rating",
            "Preferiti",
            "Ricerca avanzata"
        ],
        "endpoints": 17,
        "tabelle_aurora": 7,
        "note": "Transazioni/pagamenti esclusi"
    },
    
    "BOOKING": {
        "descrizione": "Prenotazioni risorse",
        "entità": ["BookableResource", "AvailabilityRule", "AvailabilityException", "Booking"],
        "operazioni_chiave": [
            "Risorse prenotabili (servizi, stanze, etc.)",
            "Disponibilità con regole settimanali",
            "Eccezioni (festività, chiusure)",
            "Prenotazioni con codice",
            "Conferma/cancellazione"
        ],
        "endpoints": 17,
        "tabelle_aurora": 4,
        "note": "Pagamento in loco (no online)"
    },
    
    "COMMERCE": {
        "descrizione": "E-commerce B2C",
        "entità": ["Product", "ProductVariant", "Cart", "CartItem", "ProductCategory", "ProductReview"],
        "operazioni_chiave": [
            "Catalogo prodotti con varianti",
            "Carrello persistente",
            "Wishlist",
            "Recensioni con rating",
            "Categorie gerarchiche"
        ],
        "endpoints": 16,
        "tabelle_aurora": 9,
        "note": "Checkout/ordini esclusi (richiedono pagamenti)"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK REFERENCE: COMPOSIZIONI PIATTAFORMA
# ═══════════════════════════════════════════════════════════════════════════════

COMPOSIZIONI = {
    
    "SOCIAL_NETWORK": {
        "tipo": "Instagram/Twitter-like",
        "moduli": ["Identity", "Content", "Social", "Messaging"],
        "entità_totali": 18,
        "endpoints_totali": 69,
        "costo_stimato_free_tier": "$0/mese",
        "esempio": "App social con post, follow, messaggi"
    },
    
    "MARKETPLACE_C2C": {
        "tipo": "Subito/Vinted-like",
        "moduli": ["Identity", "Content", "Social", "Messaging", "Marketplace"],
        "entità_totali": 24,
        "endpoints_totali": 86,
        "costo_stimato_free_tier": "$0/mese",
        "esempio": "Marketplace annunci tra privati"
    },
    
    "ECOMMERCE_B2C": {
        "tipo": "Shopify-like",
        "moduli": ["Identity", "Commerce"],
        "entità_totali": 14,
        "endpoints_totali": 34,
        "costo_stimato_free_tier": "$0/mese (senza checkout)",
        "esempio": "Catalogo prodotti con carrello"
    },
    
    "BOOKING_PLATFORM": {
        "tipo": "Booking/Calendly-like",
        "moduli": ["Identity", "Content", "Social", "Booking"],
        "entità_totali": 12,
        "endpoints_totali": 68,
        "costo_stimato_free_tier": "$0/mese",
        "esempio": "Prenotazioni servizi/risorse"
    },
    
    "FULL_PLATFORM": {
        "tipo": "Super-app",
        "moduli": ["Identity", "Content", "Social", "Messaging", "Marketplace", "Booking", "Commerce"],
        "entità_totali": 32,
        "endpoints_totali": 119,
        "costo_stimato_free_tier": "$0/mese",
        "esempio": "Piattaforma completa modulare"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK REFERENCE: SERVIZI AWS
# ═══════════════════════════════════════════════════════════════════════════════

AWS_SERVICES = {
    
    "ALWAYS_FREE": {
        "descrizione": "Servizi sempre gratuiti entro limiti",
        "servizi": {
            "Lambda": "1M requests/mese, 400K GB-seconds",
            "DynamoDB": "25 GB storage, 25 RCU/WCU",
            "API Gateway": "1M HTTP API calls/mese",
            "S3": "5 GB storage (primi 12 mesi)",
            "CloudWatch": "10 custom metrics, 5 GB logs",
            "SNS": "1M publish (mobile push)",
            "SQS": "1M requests/mese",
            "Cognito": "50K MAU"
        }
    },
    
    "FREE_TIER_12_MESI": {
        "descrizione": "Gratuiti per primi 12 mesi",
        "servizi": {
            "EC2": "750 ore t2.micro/mese",
            "RDS": "750 ore db.t2.micro/mese",
            "S3": "5 GB storage",
            "SES": "62K email/mese (da EC2)"
        }
    },
    
    "ESCLUSI_PAGAMENTO": {
        "descrizione": "Richiedono pagamento (non implementati)",
        "servizi": {
            "Stripe/PayPal": "Pagamenti",
            "Twilio/SNS SMS": "Verifica telefono",
            "FCM/APNs": "Push notifications (oltre free tier)",
            "Google Maps API": "Geocoding, directions",
            "AWS Rekognition": "Moderazione immagini"
        }
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK REFERENCE: PATTERN ARCHITETTURALI
# ═══════════════════════════════════════════════════════════════════════════════

PATTERNS = {
    
    "DYNAMODB_SINGLE_TABLE": {
        "descrizione": "Tutte le entità in una tabella",
        "pk_pattern": "ENTITY_TYPE#id",
        "sk_pattern": "METADATA | RELATION_TYPE#id",
        "gsi": {
            "GSI1": "Inverted index (find by owner/parent)",
            "GSI2": "Temporal/filter queries",
            "GSI3": "Sparse unique lookups"
        },
        "vantaggi": ["Single query per dati correlati", "Costi ridotti", "Scaling automatico"]
    },
    
    "REPOSITORY_PATTERN": {
        "descrizione": "Astrazione accesso dati",
        "struttura": "BaseRepository → DynamoDBRepository → EntityRepository",
        "vantaggi": ["Testabilità", "Sostituibilità DB", "Clean code"]
    },
    
    "SERVICE_LAYER": {
        "descrizione": "Business logic separata da handlers",
        "struttura": "Handler → Service → Repository",
        "vantaggi": ["Riutilizzo logica", "Unit testing", "Separation of concerns"]
    },
    
    "LAMBDA_DECORATOR": {
        "descrizione": "Middleware per Lambda handlers",
        "funzionalità": ["Auth", "Validation", "Error handling", "Logging"],
        "esempio": "@lambda_handler(require_auth=True, validate_body=Schema)"
    },
    
    "JWT_AUTH": {
        "descrizione": "Autenticazione stateless",
        "tokens": {
            "access_token": "15 minuti, per API calls",
            "refresh_token": "30 giorni, per rinnovo"
        },
        "storage": "DynamoDB sessions per revoca"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# WORKFLOW: DA ZERO A PRODUZIONE
# ═══════════════════════════════════════════════════════════════════════════════

WORKFLOW = """
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WORKFLOW: DA ZERO A PRODUZIONE                       │
└─────────────────────────────────────────────────────────────────────────────┘

FASE 1: DEFINIZIONE REQUISITI
├── Input: Idea/Concept
├── Azione: Consulta CATALOGO-REQUISITI-FUNZIONALI-v1.md
├── Output: Lista moduli necessari, operazioni richieste
└── Esempio: "Voglio un marketplace" → IDENTITY + CONTENT + SOCIAL + MESSAGING + MARKETPLACE

FASE 2: DESIGN DATABASE
├── Input: Moduli selezionati
├── Azione: Consulta CATALOGO-DATA-MODEL-v1.md
├── Output: Schema DynamoDB o Aurora per le entità scelte
└── Esempio: Copia schema LISTING, OFFER, REVIEW per marketplace

FASE 3: DEFINIZIONE API
├── Input: Entità e operazioni
├── Azione: Consulta CATALOGO-API-v1.md
├── Output: Specifiche OpenAPI per endpoint necessari
└── Esempio: Seleziona /listings/*, /offers/*, /sellers/* endpoints

FASE 4: IMPLEMENTAZIONE
├── Input: API specs, DB schema
├── Azione: Usa template da CATALOGO-CODICE-v1.md
├── Output: Codice Lambda con repository, services, handlers
└── Esempio: Copia auth_service.py, aggiungi marketplace_service.py

FASE 5: DEPLOY
├── Input: Codice completo
├── Azione: CDK deploy da infrastructure/
├── Output: Stack AWS funzionante
└── Comando: cdk deploy PlatformApiStack-dev

FASE 6: TEST
├── Input: Endpoint live
├── Azione: Esegui test suite, test manuali
├── Output: Report test, fix bugs
└── Comando: pytest tests/

FASE 7: PRODUZIONE
├── Input: Tests passati
├── Azione: CDK deploy prod con approval
├── Output: API in produzione
└── Comando: cdk deploy PlatformApiStack-prod --require-approval broadening
"""

# ═══════════════════════════════════════════════════════════════════════════════
# LOOKUP: TROVA VELOCEMENTE
# ═══════════════════════════════════════════════════════════════════════════════

LOOKUP = {
    
    "Come implemento login?": {
        "requisiti": "CATALOGO-REQUISITI → IDENTITY → LOGIN",
        "schema": "CATALOGO-DATA-MODEL → USER, SESSION",
        "api": "CATALOGO-API → POST /auth/login",
        "codice": "CATALOGO-CODICE → auth_service.py → login()"
    },
    
    "Come creo un post?": {
        "requisiti": "CATALOGO-REQUISITI → CONTENT → CREATE_POST",
        "schema": "CATALOGO-DATA-MODEL → POST, HASHTAG",
        "api": "CATALOGO-API → POST /posts",
        "codice": "CATALOGO-CODICE → handlers/content/posts.py → create_post()"
    },
    
    "Come gestisco follow?": {
        "requisiti": "CATALOGO-REQUISITI → SOCIAL → FOLLOW_USER",
        "schema": "CATALOGO-DATA-MODEL → FOLLOW, USER_STATS",
        "api": "CATALOGO-API → POST /users/{id}/follow",
        "codice": "CATALOGO-CODICE → handlers/social/follow.py → follow_user()"
    },
    
    "Come creo un annuncio marketplace?": {
        "requisiti": "CATALOGO-REQUISITI → MARKETPLACE → CREATE_LISTING",
        "schema": "CATALOGO-DATA-MODEL → LISTING, CATEGORY",
        "api": "CATALOGO-API → POST /listings",
        "codice": "Da implementare seguendo pattern auth_service"
    },
    
    "Come gestisco prenotazioni?": {
        "requisiti": "CATALOGO-REQUISITI → BOOKING → CREATE_BOOKING",
        "schema": "CATALOGO-DATA-MODEL → BOOKABLE_RESOURCE, BOOKING",
        "api": "CATALOGO-API → POST /bookings",
        "codice": "Da implementare seguendo pattern auth_service"
    },
    
    "Come aggiungo al carrello?": {
        "requisiti": "CATALOGO-REQUISITI → COMMERCE → ADD_TO_CART",
        "schema": "CATALOGO-DATA-MODEL → CART, CART_ITEM",
        "api": "CATALOGO-API → POST /cart/items",
        "codice": "Da implementare seguendo pattern auth_service"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# CHECKLIST: NUOVO PROGETTO
# ═══════════════════════════════════════════════════════════════════════════════

CHECKLIST_NUOVO_PROGETTO = """
□ 1. Definisci moduli necessari (consulta COMPOSIZIONI)
□ 2. Crea account AWS (se non esistente)
□ 3. Configura AWS CLI con profilo
□ 4. Crea repository Git
□ 5. Copia struttura da CATALOGO-CODICE
□ 6. Seleziona schema DB (DynamoDB consigliato per free tier)
□ 7. Copia schema entità necessarie
□ 8. Implementa handlers per API selezionate
□ 9. Configura CDK con stage dev
□ 10. Deploy dev e test
□ 11. Aggiungi dominio custom (opzionale)
□ 12. Deploy produzione
"""

# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO MASTER INDEX
# ═══════════════════════════════════════════════════════════════════════════════
#
# UTILIZZO:
# 1. Consulta questo file per panoramica rapida
# 2. Usa LOOKUP per trovare velocemente cosa cerchi
# 3. Segui WORKFLOW per sviluppo completo
# 4. Usa CHECKLIST per nuovi progetti
#
# ═══════════════════════════════════════════════════════════════════════════════



# ═══════════════════════════════════════════════════════════════════════════════
# AGGIORNAMENTO POST-COMPLETAMENTO
# ═══════════════════════════════════════════════════════════════════════════════

CATALOGHI_AGGIORNATI = {
    
    "CATALOGO-CODICE-v1.md": {
        "dimensione": "~517 KB",
        "linee": "~13,220",
        "scopo": "Template implementazione serverless AWS COMPLETI",
        "sezioni": [
            "1-7: Codice base (config, middleware, auth_service, CDK)",
            "8: REPOSITORY COMPLETI (6 moduli)",
            "9: HANDLERS COMPLETI (119 endpoint)",
            "10: GUIDA DEPLOYMENT PASSO-PASSO"
        ],
        "repository_implementati": [
            "user_repository.py - Identity (esistente)",
            "post_repository.py - Content (NUOVO)",
            "social_repository.py - Social (NUOVO)",
            "messaging_repository.py - Messaging (NUOVO)",
            "marketplace_repository.py - Marketplace (NUOVO)",
            "booking_repository.py - Booking (NUOVO)",
            "commerce_repository.py - Commerce (NUOVO)"
        ],
        "handlers_implementati": {
            "IDENTITY": "18 endpoint - auth, profile, sessions",
            "CONTENT": "18 endpoint - posts, comments, reactions, hashtags",
            "SOCIAL": "15 endpoint - follow, block, feed, suggestions",
            "MESSAGING": "18 endpoint - conversations, messages, notifications",
            "MARKETPLACE": "17 endpoint - listings, offers, favorites, sellers",
            "BOOKING": "17 endpoint - resources, availability, bookings",
            "COMMERCE": "16 endpoint - products, cart, wishlist, reviews"
        }
    }
}

# Statistiche finali cataloghi
STATISTICHE_FINALI = {
    "CATALOGO-REQUISITI-FUNZIONALI-v1.md": "752 KB, 3,100+ linee",
    "CATALOGO-DATA-MODEL-v1.md": "148 KB, 3,149 linee",
    "CATALOGO-API-v1.md": "122 KB, 3,566 linee",
    "CATALOGO-CODICE-v1.md": "517 KB, 13,220 linee",
    "CATALOGO-MASTER-INDEX.md": "20 KB, 458 linee",
    "TOTALE": "~1.56 MB, ~23,493 linee"
}


# ═══════════════════════════════════════════════════════════════════════════════
# AGGIORNAMENTO FINALE - CATALOGO-CODICE v2.0 COMPLETATO
# ═══════════════════════════════════════════════════════════════════════════════

"""
DATA: 2026-01-25
VERSIONE: 2.0 FINALE

Il CATALOGO-CODICE è ora COMPLETO con tutte le implementazioni necessarie
per deployare una piattaforma social/marketplace/booking serverless su AWS.
"""

CATALOGO_CODICE_V2_FINALE = {
    "file": "CATALOGO-CODICE-v1.md",
    "dimensione": "685 KB",
    "linee": "17,747",
    
    "sezioni_complete": {
        "1-7": "Codice Base (config, middleware, auth, CDK)",
        "8": "7 Repository DynamoDB (~150 metodi)",
        "9": "119 Lambda Handlers (tutti i moduli)",
        "10": "Guida Deployment AWS (6 fasi)",
        "11": "Test Suite (~145 test, 100% locale, $0)",
        "12": "Servizi Aggiuntivi (S3, SES, WebSocket, CI/CD)"
    },
    
    "moduli_coperti": {
        "Identity": {"repository": "✅", "handlers": "18", "tests": "✅"},
        "Content": {"repository": "✅", "handlers": "18", "tests": "✅"},
        "Social": {"repository": "✅", "handlers": "15", "tests": "✅"},
        "Messaging": {"repository": "✅", "handlers": "18", "tests": "✅"},
        "Marketplace": {"repository": "✅", "handlers": "17", "tests": "✅"},
        "Booking": {"repository": "✅", "handlers": "17", "tests": "✅"},
        "Commerce": {"repository": "✅", "handlers": "16", "tests": "✅"}
    },
    
    "servizi_aggiuntivi": [
        "MediaService - Upload S3 con presigned URLs",
        "EmailService - Template email SES",
        "WebSocket Handlers - Real-time messaging",
        "WebSocket CDK Stack - Infrastructure",
        "Seed Data Script - Dati di test con Faker",
        "GitHub Actions - CI/CD Pipeline completa"
    ],
    
    "file_configurazione": [
        ".env.example - Variabili ambiente",
        "pytest.ini - Configurazione test",
        "requirements-test.txt - Dipendenze test",
        "cdk.json - Configurazione CDK"
    ]
}

# ═══════════════════════════════════════════════════════════════════════════════
# STATISTICHE FINALI COMPLETE
# ═══════════════════════════════════════════════════════════════════════════════

STATISTICHE_CATALOGHI_FINALI = {
    "CATALOGO-REQUISITI-FUNZIONALI-v1.md": {
        "dimensione": "752 KB",
        "linee": "3,100+",
        "contenuto": "Specifiche funzionali 7 moduli"
    },
    "CATALOGO-DATA-MODEL-v1.md": {
        "dimensione": "148 KB", 
        "linee": "3,149",
        "contenuto": "Schema DynamoDB single-table"
    },
    "CATALOGO-API-v1.md": {
        "dimensione": "122 KB",
        "linee": "3,566",
        "contenuto": "119 endpoint REST API"
    },
    "CATALOGO-CODICE-v1.md": {
        "dimensione": "685 KB",
        "linee": "17,747",
        "contenuto": "Implementazione completa + test + deploy"
    },
    "CATALOGO-MASTER-INDEX.md": {
        "dimensione": "~25 KB",
        "linee": "~600",
        "contenuto": "Indice navigazione cataloghi"
    },
    "TOTALE": {
        "dimensione": "~1.73 MB",
        "linee": "~28,000"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# PROSSIMI PASSI CONSIGLIATI
# ═══════════════════════════════════════════════════════════════════════════════

NEXT_STEPS = """
1. SETUP AMBIENTE
   - Crea progetto locale con struttura directory
   - Copia codice dal catalogo ai file .py
   - pip install -r requirements.txt

2. TEST LOCALE
   - pip install -r requirements-test.txt
   - pytest tests/ (100% locale, $0)
   - Verifica che tutti i test passino

3. DEPLOY DEV
   - aws configure --profile platform-dev
   - cdk bootstrap
   - cdk deploy PlatformApiStack-dev
   - Salva API URL

4. TEST FUNZIONALE
   - curl POST /auth/register
   - curl POST /auth/login
   - Testa endpoint principali

5. SEED DATA (opzionale)
   - python scripts/seed_data.py --stage dev
   - Popola DB con dati di test

6. CI/CD (opzionale)
   - Configura GitHub secrets
   - Push su main → deploy automatico
"""



# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE QUICK START - GUIDA RAPIDA ALL'USO
# ═══════════════════════════════════════════════════════════════════════════════

"""
Questa sezione permette di iniziare rapidamente con il sistema di cataloghi.
Segui questi passi per utilizzare i cataloghi nel modo corretto.
"""

# ═══════════════════════════════════════════════════════════════════════════════
# QUICK START: 5 MINUTI PER INIZIARE
# ═══════════════════════════════════════════════════════════════════════════════

## PASSO 1: IDENTIFICA LA CATEGORIA DEL TUO PROGETTO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Cosa stai costruendo?                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ □ E-Commerce      → Vendita prodotti online                                 │
│ □ SaaS            → Software as a Service, dashboard, tool                  │
│ □ Marketplace     → Venditori multipli, commissioni                         │
│ □ Social          → Community, feed, messaggi                               │
│ □ Healthcare      → Settore sanitario (HIPAA compliance)                    │
│ □ FinTech         → Finanza, banking (PCI-DSS compliance)                   │
│ □ Content/Blog    → Pubblicazione contenuti                                 │
│ □ Landing Page    → Pagina singola, marketing                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

## PASSO 2: SCEGLI IL TUO STILE VISIVO

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ Che feeling vuoi dare?                                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│ → Professionale, Enterprise     = STRIPE STYLE                              │
│ → Moderno, Dark, Developer      = LINEAR STYLE                              │
│ → Pulito, Content-first         = NOTION STYLE                              │
│ → Tecnico, Cutting-edge         = VERCEL STYLE                              │
│ → Friendly, Visual, Immagini    = AIRBNB STYLE                              │
│ → Commerce, Funzionale          = SHOPIFY STYLE                             │
│ → Colorato, Team, Chat          = SLACK STYLE                               │
│ → Dense, Developer, Code        = GITHUB STYLE                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

## PASSO 3: LEGGI I CATALOGHI NELL'ORDINE CORRETTO

```
ORDINE DI LETTURA PER RALPH:

1️⃣ DESIGN-REFERENCE
   └─ Sezione stile scelto + Layout pagine necessarie

2️⃣ UI-PATTERN-PRIMITIVI  
   └─ Componenti per la tua categoria

3️⃣ DESIGN-TOKEN-SYSTEM
   └─ Tokens da customizzare (colori, fonts)

4️⃣ REQUISITI-FUNZIONALI (se necessario)
   └─ Moduli specifici per la tua categoria

5️⃣ DATA-MODEL (se backend)
   └─ Schema entità rilevanti

6️⃣ API (se backend)
   └─ Endpoints necessari

7️⃣ CODICE
   └─ Pattern implementativi

8️⃣ AWS-DETERMINISTICO (se deploy AWS)
   └─ Infrastructure patterns
```

## PASSO 4: USA IL TEMPLATE PROMPT CORRETTO

Vai a: **CATALOGO-INTERFACCIA-PROMPT-USABILITA** → Sezione 2 (Template Prompt)

```
Template disponibili:
- Progetto Completo (frontend + backend + infra)
- Solo Frontend
- Solo Backend  
- Solo Infrastruttura AWS
```

## PASSO 5: SEGUI IL WORKFLOW

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          WORKFLOW STANDARD                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  1. DISCOVERY                                                               │
│     └─ Rispondi alle domande preliminari                                    │
│     └─ Conferma riepilogo progetto                                          │
│                                                                             │
│  2. SETUP                                                                   │
│     └─ Verifica environment (Node, pnpm, Git)                               │
│     └─ Crea struttura progetto                                              │
│                                                                             │
│  3. SVILUPPO (per fasi)                                                     │
│     └─ Fase 1: Setup e configurazione                                       │
│     └─ Fase 2: Database/API (se backend)                                    │
│     └─ Fase 3: Componenti UI base                                           │
│     └─ Fase 4: Pagine principali                                            │
│     └─ Fase 5: Funzionalità complete                                        │
│     └─ Fase 6: Testing                                                      │
│                                                                             │
│  4. REVIEW                                                                  │
│     └─ Esegui checklist review                                              │
│     └─ Fix problemi trovati                                                 │
│                                                                             │
│  5. DEPLOY                                                                  │
│     └─ Build production                                                     │
│     └─ Deploy su hosting scelto                                             │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

# ═══════════════════════════════════════════════════════════════════════════════
# CHEAT SHEET - RIFERIMENTO RAPIDO
# ═══════════════════════════════════════════════════════════════════════════════

## COMANDI SETUP RAPIDO

```bash
# Next.js + TypeScript + Tailwind
pnpm create next-app@latest my-app --typescript --tailwind --eslint --app --src-dir

# Dipendenze comuni
pnpm add zod @tanstack/react-query lucide-react clsx tailwind-merge

# Supabase (auth + database)
pnpm add @supabase/supabase-js @supabase/auth-helpers-nextjs

# Stripe (payments)
pnpm add @stripe/stripe-js stripe
```

## STRUTTURA PROGETTO STANDARD

```
my-app/
├── src/
│   ├── app/              # Next.js App Router
│   ├── components/
│   │   ├── primitives/   # Button, Card, Input, etc.
│   │   ├── layout/       # Header, Footer, Sidebar
│   │   └── [category]/   # Domain-specific components
│   ├── lib/              # Utilities, clients
│   ├── hooks/            # Custom React hooks
│   ├── schemas/          # Zod schemas
│   └── styles/           # CSS, tokens
├── public/
├── tailwind.config.js
└── package.json
```

## DESIGN TOKENS BASE

```css
:root {
  /* Cambia questi per il tuo brand */
  --color-primary: #6366F1;      /* Indigo - default */
  --color-primary-hover: #4F46E5;
  
  /* Superfici */
  --color-surface-default: #FFFFFF;
  --color-surface-subtle: #F9FAFB;
  
  /* Testo */
  --color-text-default: #111827;
  --color-text-muted: #6B7280;
  
  /* Border radius (adatta allo stile) */
  --radius-md: 0.5rem;  /* 8px - default */
  --radius-lg: 0.75rem; /* 12px - cards */
}
```

## MAPPA CATALOGO → USO

```
┌─────────────────────────┬─────────────────────────────────────────────────────┐
│ DEVO...                 │ LEGGO...                                            │
├─────────────────────────┼─────────────────────────────────────────────────────┤
│ Scegliere lo stile      │ DESIGN-REFERENCE → Sezione 1                        │
│ Strutturare una pagina  │ DESIGN-REFERENCE → Sezione 2                        │
│ Creare un Button        │ UI-PATTERN-PRIMITIVI → Sezione 8.1                  │
│ Creare una Card         │ UI-PATTERN-PRIMITIVI → Sezione 3.1                  │
│ Creare un Form          │ UI-PATTERN-PRIMITIVI → Sezione 4                    │
│ Creare una Modal        │ UI-PATTERN-PRIMITIVI → Sezione 6.1                  │
│ Creare una Table        │ UI-PATTERN-PRIMITIVI → Sezione 7.1                  │
│ Definire i colori       │ DESIGN-TOKEN-SYSTEM → Colors                        │
│ Definire typography     │ DESIGN-TOKEN-SYSTEM → Typography                    │
│ Schema database         │ DATA-MODEL                                          │
│ Endpoint API            │ API                                                 │
│ Pattern codice          │ CODICE                                              │
│ Infra AWS               │ AWS-DETERMINISTICO                                  │
│ Troubleshooting         │ INTERFACCIA-PROMPT-USABILITA → Sezione 6            │
│ Esempi completi         │ INTERFACCIA-PROMPT-USABILITA → Sezione 11           │
└─────────────────────────┴─────────────────────────────────────────────────────┘
```

# ═══════════════════════════════════════════════════════════════════════════════
# STATISTICHE SISTEMA CATALOGHI (AGGIORNATE)
# ═══════════════════════════════════════════════════════════════════════════════

STATISTICHE_AGGIORNATE = {
    "data_aggiornamento": "2026-01-26",
    
    "cataloghi": {
        "CATALOGO-API-v1.md": {"size_kb": 119.3, "lines": 3403},
        "CATALOGO-AWS-DETERMINISTICO-v3.md": {"size_kb": 80.1, "lines": 1714},
        "CATALOGO-CODICE-v1.md": {"size_kb": 669.3, "lines": 16415},
        "CATALOGO-DATA-MODEL-v1.md": {"size_kb": 144.7, "lines": 3086},
        "CATALOGO-DESIGN-REFERENCE-v1.md": {"size_kb": 93.1, "lines": 1558},
        "CATALOGO-DESIGN-TOKEN-SYSTEM-v1.md": {"size_kb": 70.0, "lines": 2038},
        "CATALOGO-INTERFACCIA-PROMPT-USABILITA-v1.md": {"size_kb": 93.7, "lines": 2779},
        "CATALOGO-MASTER-INDEX.md": {"size_kb": 30.5, "lines": 681},
        "CATALOGO-REQUISITI-ARCHITETTURA-v2.md": {"size_kb": 137.8, "lines": 1511},
        "CATALOGO-REQUISITI-FUNZIONALI-v1.md": {"size_kb": 733.9, "lines": 17551},
        "CATALOGO-UI-PATTERN-PRIMITIVI-v1.md": {"size_kb": 235.8, "lines": 8007}
    },
    
    "totale": {
        "size_mb": 2.41,
        "lines": 58743,
        "cataloghi_count": 11
    },
    
    "copertura": {
        "categorie_piattaforma": 8,  # E-commerce, SaaS, Social, Healthcare, FinTech, Marketplace, Content, IoT
        "ui_patterns": 78,
        "api_endpoints": 119,
        "entita_database": 32,
        "stili_reference": 8,
        "layout_templates": 14
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# CHANGELOG
# ═══════════════════════════════════════════════════════════════════════════════

CHANGELOG = """
## [1.0.0] - 2026-01-26

### Added
- CATALOGO-DESIGN-REFERENCE-v1.md (NUOVO)
  - 8 stili di reference (Stripe, Linear, Notion, Vercel, Airbnb, Shopify, Slack, GitHub)
  - 14 layout templates con wireframe ASCII
  - Composizioni per 6 categorie piattaforma
  - Sezione per reference utente custom
  - Guida selezione stile

- CATALOGO-INTERFACCIA-PROMPT-USABILITA - Nuove sezioni:
  - Sezione 9: Framework Domande Preliminari (Discovery)
  - Sezione 10: Workflow Review e Fix Finale
  - Sezione 11: Esempi Pratici Completi (PetShop E-commerce, TaskFlow SaaS)
  - Sezione 12: Cross-Reference tra Cataloghi
  - Sezione 13: Snippet Codice Pronti all'Uso

- CATALOGO-MASTER-INDEX:
  - Quick Start Guide
  - Cheat Sheet riferimento rapido
  - Statistiche aggiornate
  - Changelog

### Changed
- CATALOGO-UI-PATTERN-PRIMITIVI-v1.md completato (Sezioni 3-13)
- Statistiche totali aggiornate

### Totali
- 11 cataloghi
- 2.41 MB di documentazione
- 58,743 linee di contenuto
- 78 UI patterns
- 119 API endpoints
- 8 stili di reference
- 14 layout templates

---

## [0.9.0] - 2026-01-25

### Added
- Tutti i cataloghi base completati
- CATALOGO-UI-PATTERN-PRIMITIVI Sezioni 1-2

---

## [0.1.0] - 2026-01-24

### Added
- Struttura iniziale cataloghi
- Prime bozze di tutti i cataloghi
"""


# ═══════════════════════════════════════════════════════════════════════════════
# AGGIORNAMENTO 2026-01-27 - NUOVI CATALOGHI v2 + FORM VALIDATION
# ═══════════════════════════════════════════════════════════════════════════════

NUOVI_CATALOGHI_2026_01_27 = {

    # ═══════════════════════════════════════════════════════════════════════════
    # CATALOGHI AGGIORNATI A v2 (ESPANSIONI)
    # ═══════════════════════════════════════════════════════════════════════════
    
    "CATALOGO-DATABASE-SELECTION-v2.md": {
        "versione": "2.0",
        "linee": 1127,
        "size_kb": 42,
        "sezioni": "1-20 (12 originali + 8 espansione)",
        "espansione": [
            "§13 Connection Pooling (PgBouncer, Prisma, RDS Proxy, pool sizing)",
            "§14 Database Migrations (Prisma, Drizzle, workflow CI/CD)",
            "§15 Index Optimization (B-tree, GIN, BRIN, partial, covering)",
            "§16 Query Optimization (N+1, pagination, anti-patterns)",
            "§17 Multi-tenancy (RLS, tenant isolation, schema patterns)",
            "§18 Backup & Disaster Recovery (pg_dump, WAL, PITR)",
            "§19 Database Monitoring (metrics, queries, alerting)",
            "§20 Database Expansion Checklist"
        ]
    },
    
    "CATALOGO-UI-UX-DESIGN-SYSTEM-v2.md": {
        "versione": "2.0",
        "linee": 948,
        "size_kb": 32,
        "sezioni": "1-18 (12 originali + 6 espansione)",
        "espansione": [
            "§13 Animation & Motion System (duration tokens, easing, Framer Motion)",
            "§14 Dark Mode Implementation (tokens, CSS variables, next-themes)",
            "§15 Responsive Patterns (breakpoints, useMediaQuery, container queries)",
            "§16 Icon System (library comparison, wrapper components)",
            "§17 Accessibility Tokens (focus ring, contrast ratios, touch targets)",
            "§18 Design System Expansion Checklist"
        ]
    },
    
    "CATALOGO-REST-API-DESIGN-v2.md": {
        "versione": "2.0",
        "linee": 1151,
        "size_kb": 36,
        "sezioni": "1-20 (13 originali + 7 espansione)",
        "espansione": [
            "§14 REST vs GraphQL vs tRPC Comparison (decision matrix)",
            "§15 tRPC Implementation (server, router, client, React Query)",
            "§16 OpenAPI 3.1 Specification (complete template)",
            "§17 Webhook Implementation (service, signature, retry)",
            "§18 API Testing (unit, integration, contract, load)",
            "§19 API Gateway Patterns (rate limiting, BFF)",
            "§20 REST API Expansion Checklist"
        ]
    },

    # ═══════════════════════════════════════════════════════════════════════════
    # NUOVO CATALOGO
    # ═══════════════════════════════════════════════════════════════════════════
    
    "CATALOGO-FORM-VALIDATION-v1.md": {
        "versione": "1.0",
        "linee": 990,
        "size_kb": 27,
        "sezioni": "1-10",
        "contenuto": [
            "§1 Validation Library Comparison (Zod, Yup, Valibot, ArkType)",
            "§2 Zod Fundamentals (primitives, objects, advanced patterns)",
            "§3 React Hook Form Integration (useZodForm, zodResolver)",
            "§4 Common Validation Patterns (email, phone, password, etc.)",
            "§5 Error Display Patterns (inline, summary, toast)",
            "§6 Conditional Validation (dependent fields, discriminated unions)",
            "§7 Multi-step Form Validation (step schemas, state persistence)",
            "§8 Server-side Validation (API routes, Server Actions)",
            "§9 Form Accessibility (ARIA, focus management, screen readers)",
            "§10 Form Validation Checklist"
        ]
    },

    # ═══════════════════════════════════════════════════════════════════════════
    # CATALOGHI ESISTENTI (già presenti ma non nel master index)
    # ═══════════════════════════════════════════════════════════════════════════
    
    "CATALOGO-SICUREZZA-v1.md": {
        "versione": "1.0",
        "linee": 3899,
        "scopo": "Sicurezza applicativa completa",
        "contenuto": [
            "Security Fundamentals (Defense in Depth, Least Privilege)",
            "OWASP Top 10:2025",
            "Security Headers (CSP, HSTS, etc.)",
            "Authentication & Authorization",
            "JWT Security",
            "Session Management",
            "Input Validation & Output Encoding",
            "API Security",
            "Secrets Management",
            "Database Security",
            "File Upload Security",
            "Rate Limiting & DDoS Protection",
            "Security Audit Checklist",
            "Incident Response"
        ]
    },
    
    "CATALOGO-TESTING-QA-v1.md": {
        "versione": "1.0",
        "scopo": "Testing e Quality Assurance",
        "contenuto": [
            "Testing Strategy",
            "Unit Testing (Vitest)",
            "Integration Testing",
            "E2E Testing (Playwright)",
            "Component Testing",
            "API Testing",
            "Performance Testing",
            "Accessibility Testing"
        ]
    },
    
    "CATALOGO-PERFORMANCE-v1.md": {
        "versione": "1.0",
        "scopo": "Performance optimization",
        "contenuto": [
            "Core Web Vitals",
            "Bundle Optimization",
            "Image Optimization",
            "Caching Strategies",
            "Database Performance",
            "API Performance"
        ]
    },
    
    "CATALOGO-SEO-v1.md": {
        "versione": "1.0",
        "scopo": "Search Engine Optimization",
        "contenuto": [
            "Technical SEO",
            "Metadata Management",
            "Structured Data",
            "Sitemap & Robots",
            "Performance for SEO"
        ]
    },
    
    "CATALOGO-ANALYTICS-MONITORING-v1.md": {
        "versione": "1.0",
        "scopo": "Analytics e Monitoring",
        "contenuto": [
            "Analytics Strategy",
            "Event Tracking",
            "Error Monitoring",
            "Performance Monitoring",
            "Logging"
        ]
    },
    
    "CATALOGO-DEVOPS-CI-CD-v1.md": {
        "versione": "1.0",
        "scopo": "DevOps e CI/CD pipelines",
        "contenuto": [
            "CI/CD Strategy",
            "GitHub Actions",
            "Docker",
            "Deployment Strategies",
            "Environment Management"
        ]
    },
    
    "CATALOGO-LEGAL-COMPLIANCE-v1.md": {
        "versione": "1.0",
        "scopo": "Legal e Compliance",
        "contenuto": [
            "GDPR Compliance",
            "Cookie Policy",
            "Privacy Policy",
            "Terms of Service",
            "Data Protection"
        ]
    },
    
    "CATALOGO-REAL-TIME-IMPLEMENTATION-v1.md": {
        "versione": "1.0",
        "scopo": "Real-time e WebSocket",
        "contenuto": [
            "WebSocket Patterns",
            "Server-Sent Events",
            "Real-time Sync",
            "Presence Systems"
        ]
    },
    
    "CATALOGO-FILE-UPLOAD-STORAGE-v1.md": {
        "versione": "1.0",
        "scopo": "File upload e storage",
        "contenuto": [
            "Upload Strategies",
            "Storage Providers (S3, Cloudflare R2)",
            "Image Processing",
            "Security"
        ]
    },
    
    "CATALOGO-STATE-MANAGEMENT-v1.md": {
        "versione": "1.0",
        "scopo": "State management React",
        "contenuto": [
            "State Strategy Decision",
            "Zustand",
            "Jotai",
            "React Query / TanStack Query",
            "URL State"
        ]
    },
    
    "CATALOGO-UI-UX-A11Y-PATTERNS-v1.md": {
        "versione": "1.0",
        "scopo": "Accessibility patterns",
        "contenuto": [
            "WCAG 2.1 AA Compliance",
            "Keyboard Navigation",
            "Screen Reader Support",
            "Focus Management",
            "Color Contrast"
        ]
    },
    
    "CATALOGO-UI-UX-COMPONENT-PATTERNS-v1.md": {
        "versione": "1.0",
        "scopo": "Component patterns avanzati",
        "contenuto": [
            "Compound Components",
            "Render Props",
            "Headless Components",
            "Controlled vs Uncontrolled"
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# STATISTICHE AGGIORNATE 2026-01-27
# ═══════════════════════════════════════════════════════════════════════════════

STATISTICHE_TOTALI_AGGIORNATE = {
    "data_aggiornamento": "2026-01-27",
    
    "totale": {
        "cataloghi_count": 35,
        "linee_stimate": 78000,
        "size_mb": 3.1
    },
    
    "per_categoria": {
        "Fondamenta & Index": ["MASTER-INDEX", "QUICK-START", "INTERFACCIA-PROMPT"],
        "Requisiti": ["REQUISITI-FUNZIONALI-v1", "REQUISITI-ARCHITETTURA-v2"],
        "Data Layer": ["DATA-MODEL-v1", "DATABASE-SELECTION-v2"],
        "API": ["API-v1", "REST-API-DESIGN-v2"],
        "Codice": ["CODICE-v1", "STATE-MANAGEMENT-v1", "FORM-VALIDATION-v1"],
        "UI/UX": ["DESIGN-SYSTEM-v2", "COMPONENT-PATTERNS-v1", "A11Y-PATTERNS-v1", 
                  "UI-PRIMITIVI-v1", "DESIGN-REFERENCE-v1", "DESIGN-TOKENS-v1"],
        "Infrastruttura": ["AWS-DETERMINISTICO-v3", "DEVOPS-CI-CD-v1"],
        "Sicurezza": ["SICUREZZA-v1", "LEGAL-COMPLIANCE-v1"],
        "Performance": ["PERFORMANCE-v1", "SEO-v1", "ANALYTICS-MONITORING-v1"],
        "Specializzati": ["REAL-TIME-v1", "FILE-UPLOAD-STORAGE-v1", "TESTING-QA-v1"]
    },
    
    "versioni_v2": [
        "DATABASE-SELECTION-v2",
        "UI-UX-DESIGN-SYSTEM-v2",
        "REST-API-DESIGN-v2",
        "REQUISITI-ARCHITETTURA-v2"
    ],
    
    "lacune_identificate": [
        "Authentication & Authorization (Auth.js, Passkeys, MFA, RBAC)",
        "Internationalization (i18n, next-intl, RTL)",
        "Notifications (Email, Push, In-App, SMS)",
        "Search (Meilisearch, Typesense, Algolia)",
        "Payments (Stripe, subscriptions, invoicing)"
    ]
}

# ═══════════════════════════════════════════════════════════════════════════════
# CHANGELOG AGGIORNATO
# ═══════════════════════════════════════════════════════════════════════════════

CHANGELOG_2026_01_27 = """
## [1.2.0] - 2026-01-27

### Added - Espansioni v2
- CATALOGO-DATABASE-SELECTION-v2.md (da v1, +8 sezioni)
  - Connection Pooling, Migrations, Index Optimization
  - Query Optimization, Multi-tenancy, Backup/DR, Monitoring
  
- CATALOGO-UI-UX-DESIGN-SYSTEM-v2.md (da v1, +6 sezioni)
  - Animation & Motion, Dark Mode, Responsive Patterns
  - Icon System, Accessibility Tokens
  
- CATALOGO-REST-API-DESIGN-v2.md (da v1, +7 sezioni)
  - REST vs GraphQL vs tRPC, tRPC Implementation
  - OpenAPI 3.1, Webhooks, API Testing, Gateway Patterns

### Added - Nuovi Cataloghi
- CATALOGO-FORM-VALIDATION-v1.md (NUOVO)
  - Zod fundamentals, React Hook Form
  - Validation patterns, Error display, Multi-step, Server-side, A11y

### Updated
- MASTER-INDEX con tutti i cataloghi esistenti
- Statistiche totali: 35 cataloghi, ~78k righe, ~3.1 MB

### Identified Gaps (prompt creati per espansione futura)
- Authentication & Authorization
- Internationalization (i18n)
- Notifications
- Search
- Payments
"""


# ═══════════════════════════════════════════════════════════════════════════════
# AGGIORNAMENTO FINALE 2026-01-27 - NUOVI CATALOGHI INTEGRATI
# ═══════════════════════════════════════════════════════════════════════════════

NUOVI_CATALOGHI_INTEGRATI = {

    "CATALOGO-AUTHENTICATION-v1.md": {
        "versione": "1.0",
        "linee": 3160,
        "size_kb": 86,
        "sezioni": 10,
        "contenuto": [
            "§1 Authentication Strategy Comparison (decision matrix)",
            "§2 Auth.js (NextAuth) v5 Implementation (complete setup)",
            "§3 Passkeys / WebAuthn (registration, authentication)",
            "§4 Multi-Factor Authentication (TOTP, backup codes)",
            "§5 Role-Based Access Control (RBAC)",
            "§6 Attribute-Based Access Control (ABAC)",
            "§7 Session Management (security, invalidation)",
            "§8 Password Security (hashing, reset flow)",
            "§9 OAuth 2.0 / OIDC Deep Dive (PKCE, refresh)",
            "§10 Authentication Checklist"
        ],
        "qualita": "⭐⭐⭐⭐⭐ Eccellente"
    },

    "CATALOGO-NOTIFICATIONS-v1.md": {
        "versione": "1.0",
        "linee": 3366,
        "size_kb": 99,
        "sezioni": 11,
        "contenuto": [
            "§1 Notification Channels Comparison",
            "§2 Email Providers Comparison",
            "§3 Transactional Email (Resend, React Email)",
            "§4 Web Push Notifications (Service Worker)",
            "§5 In-App Notifications (Sonner, Notification Center)",
            "§6 SMS Notifications (Twilio)",
            "§7 Notification Preferences (user settings)",
            "§8 Notification Service Architecture",
            "§9 Delivery Tracking & Analytics",
            "§10 Rate Limiting & Throttling",
            "§11 Notifications Checklist"
        ],
        "qualita": "⭐⭐⭐⭐⭐ Eccellente"
    },

    "CATALOGO-PAYMENTS-v1.md": {
        "versione": "1.0",
        "linee": 1153,
        "size_kb": 34,
        "sezioni": 14,
        "contenuto": [
            "§1 Payment Provider Comparison (Stripe, Paddle, etc.)",
            "§2 Stripe Setup",
            "§3 One-Time Payments (Checkout Session)",
            "§4 Subscriptions (lifecycle, billing)",
            "§5 Webhooks (critical events, idempotency)",
            "§6 Customer Management",
            "§7 Pricing Page Components",
            "§8 Invoicing",
            "§9 Refunds & Disputes",
            "§10 Tax Handling (Stripe Tax)",
            "§11 Testing & Development",
            "§12 Security & PCI Compliance",
            "§13 Payment Flows Summary",
            "§14 Payments Checklist"
        ],
        "qualita": "⭐⭐⭐⭐ Molto buono"
    },

    "CATALOGO-SEARCH-v1.md": {
        "versione": "1.0",
        "linee": 5226,
        "size_kb": 162,
        "sezioni": 12,
        "contenuto": [
            "§1 Search Engine Comparison (Meilisearch, Typesense, Algolia)",
            "§2 Meilisearch Implementation (complete setup)",
            "§3 Typesense Implementation",
            "§4 PostgreSQL Full-Text Search",
            "§5 Search API Design",
            "§6 Search UI Components (Input, Results, Facets)",
            "§7 Autocomplete & Suggestions",
            "§8 Search Analytics",
            "§9 Search Optimization (relevance, performance)",
            "§10 Multi-Tenant Search",
            "§11 Vector/Semantic Search (hybrid)",
            "§12 Search Checklist"
        ],
        "qualita": "⭐⭐⭐⭐⭐ Eccellente - il più completo"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# STATISTICHE FINALI COMPLETE
# ═══════════════════════════════════════════════════════════════════════════════

STATISTICHE_FINALI = {
    "data_aggiornamento": "2026-01-27",
    
    "cataloghi": {
        "totale_file": 40,
        "totale_linee_stimate": 95000,
        "totale_size_mb": 3.8,
    },
    
    "nuovi_cataloghi_oggi": {
        "AUTHENTICATION-v1": {"linee": 3160, "kb": 86},
        "NOTIFICATIONS-v1": {"linee": 3366, "kb": 99},
        "PAYMENTS-v1": {"linee": 1153, "kb": 34},
        "SEARCH-v1": {"linee": 5226, "kb": 162},
        "totale_nuove_linee": 12905,
        "totale_nuovi_kb": 381
    },
    
    "cataloghi_per_categoria": {
        "Core/Index": ["MASTER-INDEX", "QUICK-START", "INTERFACCIA-PROMPT"],
        "Requisiti": ["REQUISITI-FUNZIONALI-v1", "REQUISITI-ARCHITETTURA-v2"],
        "Data Layer": ["DATA-MODEL-v1", "DATABASE-SELECTION-v2", "db_architecture_v1"],
        "API": ["API-v1", "REST-API-DESIGN-v2"],
        "Backend": ["CODICE-v1", "STATE-MANAGEMENT-v1", "AUTHENTICATION-v1"],
        "Frontend Forms": ["FORM-VALIDATION-v1"],
        "UI/UX": ["DESIGN-SYSTEM-v2", "COMPONENT-PATTERNS-v1", "A11Y-PATTERNS-v1", 
                  "UI-PRIMITIVI-v1", "DESIGN-REFERENCE-v1", "DESIGN-TOKENS-v1"],
        "Infrastruttura": ["AWS-DETERMINISTICO-v3", "DEVOPS-CI-CD-v1"],
        "Sicurezza": ["SICUREZZA-v1", "LEGAL-COMPLIANCE-v1"],
        "Performance": ["PERFORMANCE-v1", "SEO-v1", "ANALYTICS-MONITORING-v1"],
        "Features": ["REAL-TIME-v1", "FILE-UPLOAD-STORAGE-v1", "NOTIFICATIONS-v1", 
                     "SEARCH-v1", "PAYMENTS-v1"],
        "Testing": ["TESTING-QA-v1"]
    },
    
    "lacuna_rimanente": "I18N (Internationalization) - file output vuoto, da rigenerare"
}

# ═══════════════════════════════════════════════════════════════════════════════
# CHANGELOG
# ═══════════════════════════════════════════════════════════════════════════════

CHANGELOG_FINALE = """
## [1.3.0] - 2026-01-27 (Sessione finale)

### Added - 4 Nuovi Cataloghi Completi
- CATALOGO-AUTHENTICATION-v1.md (3,160 righe)
  - Auth.js v5, Passkeys/WebAuthn, MFA, RBAC/ABAC
  - Password security, OAuth 2.0/OIDC, Session management
  
- CATALOGO-NOTIFICATIONS-v1.md (3,366 righe)
  - Multi-channel: Email, Push, SMS, In-App
  - Resend, React Email, Twilio, Service Workers
  - Notification center, preferences, analytics
  
- CATALOGO-PAYMENTS-v1.md (1,153 righe)
  - Stripe complete implementation
  - Subscriptions, webhooks, customer portal
  - Tax handling, PCI compliance
  
- CATALOGO-SEARCH-v1.md (5,226 righe) - IL PIÙ COMPLETO
  - Meilisearch, Typesense, Algolia comparison
  - Full implementation patterns
  - Vector/semantic search, autocomplete, facets

### Statistics
- Nuove righe aggiunte: 12,905
- Totale cataloghi: 40 file
- Totale stimato: ~95,000 righe (~3.8 MB)

### Still Missing
- I18N (Internationalization) - output vuoto, prompt da rieseguire
"""
