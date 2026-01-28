# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO REQUISITI FUNZIONALI v1.0
# ═══════════════════════════════════════════════════════════════════════════════
# FORMATO: STRUTTURE DATI DETERMINISTICHE PER MODELLI IA
# REGOLA: ZERO INTERPRETAZIONE - OGNI CAMPO HA UN SOLO SIGNIFICATO POSSIBILE
# DATA: 2026-01-25
# ═══════════════════════════════════════════════════════════════════════════════

# ═══════════════════════════════════════════════════════════════════════════════
# ISTRUZIONI PER IL MODELLO IA
# ═══════════════════════════════════════════════════════════════════════════════

"""
COME LEGGERE QUESTO CATALOGO:

1. OGNI MODULO È INDIPENDENTE
   - Puoi usare IDENTITY da solo
   - Puoi combinare IDENTITY + SOCIAL + MESSAGING
   - L'ordine di lettura NON importa

2. STRUTTURA DI OGNI MODULO
   - SCOPO: cosa fa il modulo (1 frase)
   - DIPENDENZE: quali altri moduli richiede
   - ENTITÀ: oggetti del dominio con tutti i campi
   - RELAZIONI: come le entità si collegano
   - WORKFLOW: processi step-by-step
   - OPERAZIONI: azioni possibili (CRUD + custom)
   - REGOLE: vincoli di business
   - [RISERVATO] EDGE_CASES: spazio per implementazione futura

3. COME LEGGERE UN'ENTITÀ
   Ogni attributo ha:
   - nome: nome del campo
   - tipo: tipo di dato (vedi TIPI_DATO sotto)
   - required: true = obbligatorio, false = opzionale
   - unique: true = nessun duplicato permesso
   - default: valore se non fornito (null = nessun default)
   - vincoli: regole sul valore
   - errore: codice errore se validazione fallisce

4. COME LEGGERE UN WORKFLOW
   Ogni step ha:
   - STEP N: nome dell'azione
   - ATTORE: chi esegue (user, system, admin)
   - INPUT: dati richiesti
   - PRE_CONDIZIONI: cosa deve essere vero PRIMA
   - AZIONI: cosa succede (in ordine)
   - OUTPUT_SUCCESSO: risultato se tutto ok
   - OUTPUT_ERRORE: risultato se fallisce

5. TIPI DATO AMMESSI
"""

TIPI_DATO = {
    "string": "Testo UTF-8",
    "string:email": "Testo formato email RFC 5322",
    "string:url": "Testo formato URL",
    "string:phone": "Testo formato E.164 (+39...)",
    "string:enum": "Testo da lista valori predefiniti",
    "integer": "Numero intero",
    "float": "Numero decimale",
    "boolean": "true o false",
    "uuid": "Identificatore univoco UUID v4",
    "timestamp": "Data/ora ISO 8601 UTC",
    "date": "Data YYYY-MM-DD",
    "json": "Oggetto JSON",
    "array": "Lista di elementi",
    "binary": "Dati binari (file)"
}

# ═══════════════════════════════════════════════════════════════════════════════
# INDICE MODULI
# ═══════════════════════════════════════════════════════════════════════════════

INDICE_MODULI = {
    "IDENTITY": {
        "descrizione": "Gestione utenti, autenticazione, profili",
        "dipendenze": [],
        "sezione": "MODULO_001"
    },
    "CONTENT": {
        "descrizione": "Creazione e pubblicazione contenuti",
        "dipendenze": ["IDENTITY"],
        "sezione": "MODULO_002"
    },
    "SOCIAL": {
        "descrizione": "Connessioni, follow, feed, reazioni",
        "dipendenze": ["IDENTITY", "CONTENT"],
        "sezione": "MODULO_003"
    },
    "MESSAGING": {
        "descrizione": "Chat, conversazioni, notifiche",
        "dipendenze": ["IDENTITY"],
        "sezione": "MODULO_004"
    },
    "MARKETPLACE": {
        "descrizione": "Annunci, offerte, transazioni",
        "dipendenze": ["IDENTITY"],
        "sezione": "MODULO_005"
    },
    "BOOKING": {
        "descrizione": "Disponibilità, prenotazioni, calendario",
        "dipendenze": ["IDENTITY"],
        "sezione": "MODULO_006"
    },
    "COMMERCE": {
        "descrizione": "Prodotti, carrello, ordini, pagamenti",
        "dipendenze": ["IDENTITY"],
        "sezione": "MODULO_007"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# COMPOSIZIONI PIATTAFORME
# ═══════════════════════════════════════════════════════════════════════════════

"""
COME COMPORRE UNA PIATTAFORMA:

1. Identifica i moduli necessari
2. Unisci le entità (IDENTITY.User è sempre la base)
3. Unisci i workflow
4. Le relazioni tra moduli sono definite in RELAZIONI_CROSS_MODULO

ESEMPI:
"""

COMPOSIZIONI = {
    "SOCIAL_NETWORK": {
        "moduli": ["IDENTITY", "CONTENT", "SOCIAL", "MESSAGING"],
        "descrizione": "Piattaforma social con post, connessioni, chat"
    },
    "MARKETPLACE": {
        "moduli": ["IDENTITY", "MARKETPLACE", "MESSAGING"],
        "descrizione": "Piattaforma annunci con transazioni"
    },
    "ECOMMERCE": {
        "moduli": ["IDENTITY", "COMMERCE"],
        "descrizione": "Negozio online con carrello e ordini"
    },
    "BOOKING_PLATFORM": {
        "moduli": ["IDENTITY", "BOOKING", "MESSAGING"],
        "descrizione": "Piattaforma prenotazioni"
    },
    "FREELANCE_PLATFORM": {
        "moduli": ["IDENTITY", "CONTENT", "SOCIAL", "MARKETPLACE", "BOOKING", "MESSAGING"],
        "descrizione": "Piattaforma freelance completa"
    },
    "CONTENT_PLATFORM": {
        "moduli": ["IDENTITY", "CONTENT", "SOCIAL"],
        "descrizione": "Piattaforma pubblicazione contenuti (blog, video)"
    }
}



# ███████████████████████████████████████████████████████████████████████████████
# MODULO_001: IDENTITY
# ███████████████████████████████████████████████████████████████████████████████

MODULO_IDENTITY = {
    
    "id": "MODULO_001",
    "nome": "IDENTITY",
    "scopo": "Gestire utenti, autenticazione, sessioni e profili",
    "dipendenze": [],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ═══════════════════════════════════════════════════════════════════════
        # USER - Utente registrato nel sistema
        # ═══════════════════════════════════════════════════════════════════════
        
        "USER": {
            "descrizione": "Un utente registrato nel sistema",
            
            "attributi": {
                
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "formato": "UUID v4 (esempio: 550e8400-e29b-41d4-a716-446655440000)"
                    },
                    "errore_se_invalido": None
                },
                
                "email": {
                    "tipo": "string:email",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "formato": "RFC 5322 (esempio: nome@dominio.com)",
                        "lunghezza_min": 5,
                        "lunghezza_max": 254,
                        "regex": "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
                    },
                    "errore_se_invalido": {
                        "codice": "USER_EMAIL_INVALID",
                        "messaggio": "Formato email non valido"
                    },
                    "errore_se_duplicato": {
                        "codice": "USER_EMAIL_EXISTS",
                        "messaggio": "Email già registrata"
                    }
                },
                
                "email_verified": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "default": False,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "valori_ammessi": [True, False]
                    },
                    "errore_se_invalido": None
                },
                
                "username": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_min": 3,
                        "lunghezza_max": 30,
                        "regex": "^[a-zA-Z0-9_]+$",
                        "caratteri_ammessi": "lettere (a-z, A-Z), numeri (0-9), underscore (_)",
                        "caratteri_vietati": "spazi, caratteri speciali, emoji"
                    },
                    "errore_se_invalido": {
                        "codice": "USER_USERNAME_INVALID",
                        "messaggio": "Username non valido. Usa solo lettere, numeri e underscore (3-30 caratteri)"
                    },
                    "errore_se_duplicato": {
                        "codice": "USER_USERNAME_EXISTS",
                        "messaggio": "Username già in uso"
                    }
                },
                
                "password_hash": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": True,
                    "visibile_a": ["sistema"],
                    "vincoli": {
                        "algoritmo": "bcrypt",
                        "salt_rounds": 12
                    },
                    "errore_se_invalido": None,
                    "nota": "MAI esporre all'esterno. MAI salvare password in chiaro."
                },
                
                "status": {
                    "tipo": "string:enum",
                    "required": True,
                    "unique": False,
                    "default": "pending_verification",
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {
                        "valori_ammessi": ["pending_verification", "active", "suspended", "deleted"],
                        "transizioni_valide": {
                            "pending_verification": ["active", "deleted"],
                            "active": ["suspended", "deleted"],
                            "suspended": ["active", "deleted"],
                            "deleted": []
                        }
                    },
                    "errore_se_invalido": {
                        "codice": "USER_STATUS_INVALID",
                        "messaggio": "Stato utente non valido"
                    }
                },
                
                "role": {
                    "tipo": "string:enum",
                    "required": True,
                    "unique": False,
                    "default": "user",
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {
                        "valori_ammessi": ["user", "moderator", "admin", "super_admin"],
                        "gerarchia": {
                            "user": 1,
                            "moderator": 2,
                            "admin": 3,
                            "super_admin": 4
                        }
                    },
                    "errore_se_invalido": {
                        "codice": "USER_ROLE_INVALID",
                        "messaggio": "Ruolo non valido"
                    }
                },
                
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "formato": "ISO 8601 UTC (esempio: 2026-01-25T10:30:00Z)"
                    },
                    "errore_se_invalido": None
                },
                
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "regola": "Aggiornato automaticamente ad ogni modifica"
                    },
                    "errore_se_invalido": None
                },
                
                "last_login_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "deleted_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "regola": "Popolato solo quando status = 'deleted'"
                    },
                    "errore_se_invalido": None
                }
            },
            
            "indici_database": [
                {"campi": ["id"], "tipo": "PRIMARY KEY"},
                {"campi": ["email"], "tipo": "UNIQUE"},
                {"campi": ["username"], "tipo": "UNIQUE"},
                {"campi": ["status"], "tipo": "INDEX"},
                {"campi": ["created_at"], "tipo": "INDEX"}
            ],
            
            "relazioni": {
                "ha_uno": ["PROFILE"],
                "ha_molti": ["SESSION", "EMAIL_VERIFICATION_TOKEN", "PASSWORD_RESET_TOKEN"]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # PROFILE - Profilo pubblico dell'utente
        # ═══════════════════════════════════════════════════════════════════════
        
        "PROFILE": {
            "descrizione": "Informazioni pubbliche e private dell'utente",
            
            "attributi": {
                
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "riferimento": "USER.id",
                        "on_delete": "CASCADE"
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_USER_NOT_FOUND",
                        "messaggio": "Utente non trovato"
                    }
                },
                
                "display_name": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_min": 1,
                        "lunghezza_max": 100
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_DISPLAY_NAME_INVALID",
                        "messaggio": "Nome visualizzato troppo lungo (max 100 caratteri)"
                    }
                },
                
                "bio": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_max": 500
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_BIO_INVALID",
                        "messaggio": "Biografia troppo lunga (max 500 caratteri)"
                    }
                },
                
                "avatar_url": {
                    "tipo": "string:url",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_max": 500,
                        "protocolli_ammessi": ["https"]
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_AVATAR_URL_INVALID",
                        "messaggio": "URL avatar non valido"
                    }
                },
                
                "location": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_max": 100
                    },
                    "errore_se_invalido": None
                },
                
                "website": {
                    "tipo": "string:url",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "vincoli": {
                        "lunghezza_max": 200,
                        "protocolli_ammessi": ["http", "https"]
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_WEBSITE_INVALID",
                        "messaggio": "URL sito web non valido"
                    }
                },
                
                "date_of_birth": {
                    "tipo": "date",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "visibile_a": ["proprietario", "sistema"],
                    "vincoli": {
                        "min": "1900-01-01",
                        "max": "TODAY - 13 anni"
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_DOB_INVALID",
                        "messaggio": "Data di nascita non valida"
                    }
                },
                
                "phone": {
                    "tipo": "string:phone",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "utente",
                    "modificabile": True,
                    "visibile_a": ["proprietario", "sistema"],
                    "vincoli": {
                        "formato": "E.164 (esempio: +393331234567)"
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_PHONE_INVALID",
                        "messaggio": "Formato numero telefono non valido"
                    }
                },
                
                "privacy_settings": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "default": {
                        "profile_visibility": "public",
                        "show_email": False,
                        "show_phone": False,
                        "show_date_of_birth": False,
                        "allow_messages_from": "everyone"
                    },
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {
                        "schema": {
                            "profile_visibility": {
                                "tipo": "string:enum",
                                "valori_ammessi": ["public", "followers_only", "private"]
                            },
                            "show_email": {
                                "tipo": "boolean"
                            },
                            "show_phone": {
                                "tipo": "boolean"
                            },
                            "show_date_of_birth": {
                                "tipo": "boolean"
                            },
                            "allow_messages_from": {
                                "tipo": "string:enum",
                                "valori_ammessi": ["everyone", "followers_only", "nobody"]
                            }
                        }
                    },
                    "errore_se_invalido": {
                        "codice": "PROFILE_PRIVACY_INVALID",
                        "messaggio": "Impostazioni privacy non valide"
                    }
                },
                
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                }
            },
            
            "indici_database": [
                {"campi": ["id"], "tipo": "PRIMARY KEY"},
                {"campi": ["user_id"], "tipo": "UNIQUE FOREIGN KEY"}
            ],
            
            "relazioni": {
                "appartiene_a": ["USER"]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # SESSION - Sessione di autenticazione
        # ═══════════════════════════════════════════════════════════════════════
        
        "SESSION": {
            "descrizione": "Una sessione di autenticazione attiva",
            
            "attributi": {
                
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "riferimento": "USER.id",
                        "on_delete": "CASCADE"
                    },
                    "errore_se_invalido": None
                },
                
                "access_token_hash": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "visibile_a": ["sistema"],
                    "vincoli": {
                        "algoritmo": "SHA256"
                    },
                    "errore_se_invalido": None
                },
                
                "refresh_token_hash": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": True,
                    "visibile_a": ["sistema"],
                    "vincoli": {
                        "algoritmo": "SHA256"
                    },
                    "errore_se_invalido": None
                },
                
                "device_info": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "schema": {
                            "user_agent": "string",
                            "ip_address": "string",
                            "device_type": "string (desktop, mobile, tablet)",
                            "browser": "string",
                            "os": "string"
                        }
                    },
                    "errore_se_invalido": None
                },
                
                "access_token_expires_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW() + 15 minuti",
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "refresh_token_expires_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW() + 7 giorni",
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "is_active": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "default": True,
                    "generato_da": "sistema",
                    "modificabile": True,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                }
            },
            
            "indici_database": [
                {"campi": ["id"], "tipo": "PRIMARY KEY"},
                {"campi": ["access_token_hash"], "tipo": "UNIQUE"},
                {"campi": ["refresh_token_hash"], "tipo": "UNIQUE"},
                {"campi": ["user_id", "is_active"], "tipo": "INDEX"},
                {"campi": ["access_token_expires_at"], "tipo": "INDEX"}
            ],
            
            "relazioni": {
                "appartiene_a": ["USER"]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # EMAIL_VERIFICATION_TOKEN - Token per verifica email
        # ═══════════════════════════════════════════════════════════════════════
        
        "EMAIL_VERIFICATION_TOKEN": {
            "descrizione": "Token per verificare l'indirizzo email di un utente",
            
            "attributi": {
                
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "riferimento": "USER.id",
                        "on_delete": "CASCADE"
                    },
                    "errore_se_invalido": None
                },
                
                "email": {
                    "tipo": "string:email",
                    "required": True,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "token": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "lunghezza": 64,
                        "algoritmo": "crypto.randomBytes(32).toString('hex')"
                    },
                    "errore_se_invalido": None
                },
                
                "expires_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW() + 24 ore",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "used_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "regola": "Popolato quando il token viene usato"
                    },
                    "errore_se_invalido": None
                },
                
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                }
            },
            
            "indici_database": [
                {"campi": ["id"], "tipo": "PRIMARY KEY"},
                {"campi": ["token"], "tipo": "UNIQUE"},
                {"campi": ["user_id"], "tipo": "INDEX"},
                {"campi": ["expires_at"], "tipo": "INDEX"}
            ],
            
            "relazioni": {
                "appartiene_a": ["USER"]
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # PASSWORD_RESET_TOKEN - Token per reset password
        # ═══════════════════════════════════════════════════════════════════════
        
        "PASSWORD_RESET_TOKEN": {
            "descrizione": "Token per reimpostare la password di un utente",
            
            "attributi": {
                
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "riferimento": "USER.id",
                        "on_delete": "CASCADE"
                    },
                    "errore_se_invalido": None
                },
                
                "token": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {
                        "lunghezza": 64,
                        "algoritmo": "crypto.randomBytes(32).toString('hex')"
                    },
                    "errore_se_invalido": None
                },
                
                "expires_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW() + 1 ora",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "used_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "default": None,
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                },
                
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "default": "NOW()",
                    "generato_da": "sistema",
                    "modificabile": False,
                    "vincoli": {},
                    "errore_se_invalido": None
                }
            },
            
            "indici_database": [
                {"campi": ["id"], "tipo": "PRIMARY KEY"},
                {"campi": ["token"], "tipo": "UNIQUE"},
                {"campi": ["user_id"], "tipo": "INDEX"},
                {"campi": ["expires_at"], "tipo": "INDEX"}
            ],
            
            "relazioni": {
                "appartiene_a": ["USER"]
            }
        }
    },

    # ═══════════════════════════════════════════════════════════════════════════
    # WORKFLOW - MODULO IDENTITY
    # ═══════════════════════════════════════════════════════════════════════════
    
    "workflow": {
        
        # ═══════════════════════════════════════════════════════════════════════
        # WORKFLOW: REGISTRAZIONE UTENTE
        # ═══════════════════════════════════════════════════════════════════════
        
        "REGISTRAZIONE_UTENTE": {
            "nome": "Registrazione nuovo utente",
            "descrizione": "Processo completo per registrare un nuovo utente nel sistema",
            "trigger": "Utente invia form di registrazione",
            
            "step": {
                
                "STEP_1": {
                    "nome": "Ricezione dati registrazione",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema riceve i dati dal form di registrazione",
                    
                    "input": {
                        "email": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "esempio": "mario.rossi@email.com"
                        },
                        "username": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "esempio": "mario_rossi"
                        },
                        "password": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "esempio": "MiaPassword123!"
                        }
                    },
                    
                    "azioni": [
                        "1. Ricevi dati HTTP POST dal client",
                        "2. Estrai email, username, password dal body della richiesta",
                        "3. Passa al STEP_2"
                    ],
                    
                    "output": {
                        "email": "valore ricevuto",
                        "username": "valore ricevuto",
                        "password": "valore ricevuto (in chiaro, solo in memoria)"
                    },
                    
                    "prossimo_step": "STEP_2"
                },
                
                "STEP_2": {
                    "nome": "Validazione formato dati",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che i dati abbiano il formato corretto",
                    
                    "input": {
                        "email": "da STEP_1",
                        "username": "da STEP_1",
                        "password": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. VALIDA email:",
                        "   - Verifica che non sia vuota",
                        "   - Verifica che corrisponda al regex ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$",
                        "   - Verifica che lunghezza <= 254 caratteri",
                        "   - SE non valida: ERRORE USER_EMAIL_INVALID, vai a STEP_ERRORE",
                        "",
                        "2. VALIDA username:",
                        "   - Verifica che lunghezza >= 3",
                        "   - Verifica che lunghezza <= 30",
                        "   - Verifica che corrisponda al regex ^[a-zA-Z0-9_]+$",
                        "   - SE non valido: ERRORE USER_USERNAME_INVALID, vai a STEP_ERRORE",
                        "",
                        "3. VALIDA password:",
                        "   - Verifica che lunghezza >= 8",
                        "   - Verifica che contenga almeno 1 lettera maiuscola (A-Z)",
                        "   - Verifica che contenga almeno 1 lettera minuscola (a-z)",
                        "   - Verifica che contenga almeno 1 numero (0-9)",
                        "   - SE non valida: ERRORE USER_PASSWORD_WEAK, vai a STEP_ERRORE",
                        "",
                        "4. SE tutte le validazioni passano: vai a STEP_3"
                    ],
                    
                    "output": {
                        "validazione_ok": True
                    },
                    
                    "prossimo_step_successo": "STEP_3",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_3": {
                    "nome": "Verifica unicità email",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che l'email non sia già registrata",
                    
                    "input": {
                        "email": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT COUNT(*) FROM USER WHERE email = {email} AND status != 'deleted'",
                        "",
                        "2. SE count > 0:",
                        "   - ERRORE USER_EMAIL_EXISTS",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE count == 0:",
                        "   - vai a STEP_4"
                    ],
                    
                    "output": {
                        "email_disponibile": True
                    },
                    
                    "prossimo_step_successo": "STEP_4",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_4": {
                    "nome": "Verifica unicità username",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che l'username non sia già in uso",
                    
                    "input": {
                        "username": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT COUNT(*) FROM USER WHERE username = {username} AND status != 'deleted'",
                        "",
                        "2. SE count > 0:",
                        "   - ERRORE USER_USERNAME_EXISTS",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE count == 0:",
                        "   - vai a STEP_5"
                    ],
                    
                    "output": {
                        "username_disponibile": True
                    },
                    
                    "prossimo_step_successo": "STEP_5",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_5": {
                    "nome": "Hash della password",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema calcola l'hash sicuro della password",
                    
                    "input": {
                        "password": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Genera salt con bcrypt (rounds = 12)",
                        "2. Calcola hash: password_hash = bcrypt.hash(password, salt)",
                        "3. IMPORTANTE: Elimina la password in chiaro dalla memoria",
                        "4. vai a STEP_6"
                    ],
                    
                    "output": {
                        "password_hash": "stringa hash bcrypt (esempio: $2b$12$...)"
                    },
                    
                    "prossimo_step": "STEP_6"
                },
                
                "STEP_6": {
                    "nome": "Generazione ID utente",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema genera un identificatore univoco per l'utente",
                    
                    "input": {},
                    
                    "azioni": [
                        "1. Genera UUID v4: user_id = uuid.v4()",
                        "2. vai a STEP_7"
                    ],
                    
                    "output": {
                        "user_id": "UUID v4 (esempio: 550e8400-e29b-41d4-a716-446655440000)"
                    },
                    
                    "prossimo_step": "STEP_7"
                },
                
                "STEP_7": {
                    "nome": "Creazione record USER",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema inserisce il nuovo utente nel database",
                    
                    "input": {
                        "user_id": "da STEP_6",
                        "email": "da STEP_1",
                        "username": "da STEP_1",
                        "password_hash": "da STEP_5"
                    },
                    
                    "azioni": [
                        "1. INSERT INTO USER:",
                        "   - id = {user_id}",
                        "   - email = {email}",
                        "   - email_verified = false",
                        "   - username = {username}",
                        "   - password_hash = {password_hash}",
                        "   - status = 'pending_verification'",
                        "   - role = 'user'",
                        "   - created_at = NOW()",
                        "   - updated_at = NOW()",
                        "   - last_login_at = NULL",
                        "   - deleted_at = NULL",
                        "",
                        "2. SE insert fallisce:",
                        "   - ERRORE DATABASE_ERROR",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE insert OK:",
                        "   - vai a STEP_8"
                    ],
                    
                    "output": {
                        "user_creato": True
                    },
                    
                    "prossimo_step_successo": "STEP_8",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_8": {
                    "nome": "Creazione record PROFILE",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema crea il profilo associato all'utente",
                    
                    "input": {
                        "user_id": "da STEP_6",
                        "username": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Genera UUID per profile: profile_id = uuid.v4()",
                        "",
                        "2. INSERT INTO PROFILE:",
                        "   - id = {profile_id}",
                        "   - user_id = {user_id}",
                        "   - display_name = {username}",
                        "   - bio = NULL",
                        "   - avatar_url = NULL",
                        "   - location = NULL",
                        "   - website = NULL",
                        "   - date_of_birth = NULL",
                        "   - phone = NULL",
                        "   - privacy_settings = {",
                        "       'profile_visibility': 'public',",
                        "       'show_email': false,",
                        "       'show_phone': false,",
                        "       'show_date_of_birth': false,",
                        "       'allow_messages_from': 'everyone'",
                        "     }",
                        "   - created_at = NOW()",
                        "   - updated_at = NOW()",
                        "",
                        "3. vai a STEP_9"
                    ],
                    
                    "output": {
                        "profile_id": "UUID del profilo creato"
                    },
                    
                    "prossimo_step": "STEP_9"
                },
                
                "STEP_9": {
                    "nome": "Generazione token verifica email",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema genera un token per verificare l'email",
                    
                    "input": {
                        "user_id": "da STEP_6",
                        "email": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Genera token sicuro: token = crypto.randomBytes(32).toString('hex')",
                        "2. Genera UUID: token_id = uuid.v4()",
                        "3. Calcola scadenza: expires_at = NOW() + 24 ore",
                        "",
                        "4. INSERT INTO EMAIL_VERIFICATION_TOKEN:",
                        "   - id = {token_id}",
                        "   - user_id = {user_id}",
                        "   - email = {email}",
                        "   - token = {token}",
                        "   - expires_at = {expires_at}",
                        "   - used_at = NULL",
                        "   - created_at = NOW()",
                        "",
                        "5. vai a STEP_10"
                    ],
                    
                    "output": {
                        "verification_token": "stringa 64 caratteri hex"
                    },
                    
                    "prossimo_step": "STEP_10"
                },
                
                "STEP_10": {
                    "nome": "Invio email di verifica",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema invia l'email con il link di verifica",
                    
                    "input": {
                        "email": "da STEP_1",
                        "username": "da STEP_1",
                        "verification_token": "da STEP_9"
                    },
                    
                    "azioni": [
                        "1. Costruisci URL verifica:",
                        "   verification_url = '{BASE_URL}/verify-email?token={verification_token}'",
                        "",
                        "2. Prepara email:",
                        "   - destinatario = {email}",
                        "   - oggetto = 'Verifica il tuo indirizzo email'",
                        "   - corpo = 'Ciao {username}, clicca qui per verificare: {verification_url}'",
                        "",
                        "3. Invia email (asincrono, non bloccare)",
                        "",
                        "4. vai a STEP_SUCCESSO"
                    ],
                    
                    "output": {
                        "email_inviata": True
                    },
                    
                    "prossimo_step": "STEP_SUCCESSO"
                },
                
                "STEP_SUCCESSO": {
                    "nome": "Registrazione completata",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema conferma la registrazione avvenuta",
                    
                    "input": {
                        "user_id": "da STEP_6"
                    },
                    
                    "azioni": [
                        "1. Prepara risposta di successo",
                        "2. Restituisci al client"
                    ],
                    
                    "output": {
                        "success": True,
                        "http_status": 201,
                        "body": {
                            "message": "Registrazione completata. Controlla la tua email per verificare l'account.",
                            "user_id": "{user_id}"
                        }
                    },
                    
                    "prossimo_step": "FINE"
                },
                
                "STEP_ERRORE": {
                    "nome": "Gestione errore",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema gestisce un errore durante la registrazione",
                    
                    "input": {
                        "codice_errore": "codice dell'errore verificatosi",
                        "messaggio_errore": "descrizione dell'errore"
                    },
                    
                    "azioni": [
                        "1. Log dell'errore per debugging",
                        "2. Prepara risposta di errore",
                        "3. Restituisci al client"
                    ],
                    
                    "output": {
                        "success": False,
                        "http_status": "400 o 409 a seconda dell'errore",
                        "body": {
                            "error": {
                                "code": "{codice_errore}",
                                "message": "{messaggio_errore}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                }
            },
            
            "errori_possibili": {
                "USER_EMAIL_INVALID": {
                    "http_status": 400,
                    "messaggio": "Formato email non valido"
                },
                "USER_USERNAME_INVALID": {
                    "http_status": 400,
                    "messaggio": "Username non valido. Usa solo lettere, numeri e underscore (3-30 caratteri)"
                },
                "USER_PASSWORD_WEAK": {
                    "http_status": 400,
                    "messaggio": "Password troppo debole. Richiesti: min 8 caratteri, 1 maiuscola, 1 minuscola, 1 numero"
                },
                "USER_EMAIL_EXISTS": {
                    "http_status": 409,
                    "messaggio": "Email già registrata"
                },
                "USER_USERNAME_EXISTS": {
                    "http_status": 409,
                    "messaggio": "Username già in uso"
                },
                "DATABASE_ERROR": {
                    "http_status": 500,
                    "messaggio": "Errore interno del server"
                }
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # WORKFLOW: VERIFICA EMAIL
        # ═══════════════════════════════════════════════════════════════════════
        
        "VERIFICA_EMAIL": {
            "nome": "Verifica indirizzo email",
            "descrizione": "Processo per verificare l'email di un utente tramite token",
            "trigger": "Utente clicca sul link di verifica email",
            
            "step": {
                
                "STEP_1": {
                    "nome": "Ricezione token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema riceve il token dalla URL",
                    
                    "input": {
                        "token": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "origine": "query parameter URL"
                        }
                    },
                    
                    "azioni": [
                        "1. Estrai token dalla URL: /verify-email?token={token}",
                        "2. Verifica che token non sia vuoto",
                        "3. SE vuoto: ERRORE TOKEN_MISSING, vai a STEP_ERRORE",
                        "4. SE presente: vai a STEP_2"
                    ],
                    
                    "output": {
                        "token": "stringa token"
                    },
                    
                    "prossimo_step_successo": "STEP_2",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_2": {
                    "nome": "Ricerca token nel database",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema cerca il token nel database",
                    
                    "input": {
                        "token": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT * FROM EMAIL_VERIFICATION_TOKEN WHERE token = {token}",
                        "",
                        "2. SE nessun risultato:",
                        "   - ERRORE TOKEN_INVALID",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE trovato:",
                        "   - Salva record come verification_record",
                        "   - vai a STEP_3"
                    ],
                    
                    "output": {
                        "verification_record": "record dal database"
                    },
                    
                    "prossimo_step_successo": "STEP_3",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_3": {
                    "nome": "Verifica scadenza token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che il token non sia scaduto",
                    
                    "input": {
                        "verification_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Confronta expires_at con NOW():",
                        "   SE verification_record.expires_at < NOW():",
                        "   - ERRORE TOKEN_EXPIRED",
                        "   - vai a STEP_ERRORE",
                        "",
                        "2. SE non scaduto:",
                        "   - vai a STEP_4"
                    ],
                    
                    "output": {
                        "token_valido": True
                    },
                    
                    "prossimo_step_successo": "STEP_4",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_4": {
                    "nome": "Verifica token non usato",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che il token non sia già stato usato",
                    
                    "input": {
                        "verification_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Controlla used_at:",
                        "   SE verification_record.used_at != NULL:",
                        "   - ERRORE TOKEN_ALREADY_USED",
                        "   - vai a STEP_ERRORE",
                        "",
                        "2. SE used_at == NULL:",
                        "   - vai a STEP_5"
                    ],
                    
                    "output": {
                        "token_non_usato": True
                    },
                    
                    "prossimo_step_successo": "STEP_5",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_5": {
                    "nome": "Aggiornamento stato utente",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema aggiorna l'utente come verificato",
                    
                    "input": {
                        "verification_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. UPDATE USER SET:",
                        "   - email_verified = true",
                        "   - status = 'active' (se era 'pending_verification')",
                        "   - updated_at = NOW()",
                        "   WHERE id = {verification_record.user_id}",
                        "",
                        "2. vai a STEP_6"
                    ],
                    
                    "output": {
                        "user_aggiornato": True
                    },
                    
                    "prossimo_step": "STEP_6"
                },
                
                "STEP_6": {
                    "nome": "Marcatura token come usato",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema marca il token come usato",
                    
                    "input": {
                        "verification_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. UPDATE EMAIL_VERIFICATION_TOKEN SET:",
                        "   - used_at = NOW()",
                        "   WHERE id = {verification_record.id}",
                        "",
                        "2. vai a STEP_SUCCESSO"
                    ],
                    
                    "output": {
                        "token_marcato": True
                    },
                    
                    "prossimo_step": "STEP_SUCCESSO"
                },
                
                "STEP_SUCCESSO": {
                    "nome": "Verifica completata",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": True,
                        "http_status": 200,
                        "body": {
                            "message": "Email verificata con successo. Puoi ora effettuare il login."
                        }
                    },
                    
                    "prossimo_step": "FINE"
                },
                
                "STEP_ERRORE": {
                    "nome": "Gestione errore",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": False,
                        "http_status": "400",
                        "body": {
                            "error": {
                                "code": "{codice_errore}",
                                "message": "{messaggio_errore}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                }
            },
            
            "errori_possibili": {
                "TOKEN_MISSING": {
                    "http_status": 400,
                    "messaggio": "Token di verifica mancante"
                },
                "TOKEN_INVALID": {
                    "http_status": 400,
                    "messaggio": "Token di verifica non valido"
                },
                "TOKEN_EXPIRED": {
                    "http_status": 400,
                    "messaggio": "Token di verifica scaduto"
                },
                "TOKEN_ALREADY_USED": {
                    "http_status": 400,
                    "messaggio": "Token di verifica già utilizzato"
                }
            }
        },

        # ═══════════════════════════════════════════════════════════════════════
        # WORKFLOW: LOGIN
        # ═══════════════════════════════════════════════════════════════════════
        
        "LOGIN": {
            "nome": "Autenticazione utente",
            "descrizione": "Processo per autenticare un utente e creare una sessione",
            "trigger": "Utente invia credenziali di accesso",
            
            "step": {
                
                "STEP_1": {
                    "nome": "Ricezione credenziali",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema riceve le credenziali di login",
                    
                    "input": {
                        "identifier": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "descrizione": "Email o username",
                            "esempio": "mario.rossi@email.com oppure mario_rossi"
                        },
                        "password": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "esempio": "MiaPassword123!"
                        },
                        "device_info": {
                            "tipo": "json",
                            "obbligatorio": False,
                            "esempio": {"user_agent": "Mozilla/5.0...", "ip_address": "192.168.1.1"}
                        }
                    },
                    
                    "azioni": [
                        "1. Ricevi dati HTTP POST",
                        "2. Estrai identifier, password, device_info",
                        "3. SE identifier vuoto: ERRORE CREDENTIALS_MISSING, vai a STEP_ERRORE",
                        "4. SE password vuota: ERRORE CREDENTIALS_MISSING, vai a STEP_ERRORE",
                        "5. vai a STEP_2"
                    ],
                    
                    "output": {
                        "identifier": "email o username",
                        "password": "password in chiaro (solo in memoria)",
                        "device_info": "informazioni dispositivo"
                    },
                    
                    "prossimo_step_successo": "STEP_2",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_2": {
                    "nome": "Ricerca utente",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema cerca l'utente per email o username",
                    
                    "input": {
                        "identifier": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Determina se identifier è email o username:",
                        "   SE identifier contiene '@': cerca per email",
                        "   ALTRIMENTI: cerca per username",
                        "",
                        "2. QUERY database:",
                        "   SELECT * FROM USER",
                        "   WHERE (email = {identifier} OR username = {identifier})",
                        "   AND status != 'deleted'",
                        "",
                        "3. SE nessun risultato:",
                        "   - ERRORE CREDENTIALS_INVALID (non rivelare se utente esiste)",
                        "   - vai a STEP_ERRORE",
                        "",
                        "4. SE trovato:",
                        "   - Salva come user_record",
                        "   - vai a STEP_3"
                    ],
                    
                    "output": {
                        "user_record": "record utente dal database"
                    },
                    
                    "prossimo_step_successo": "STEP_3",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_3": {
                    "nome": "Verifica stato account",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che l'account sia attivo",
                    
                    "input": {
                        "user_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Controlla status dell'utente:",
                        "",
                        "   SE user_record.status == 'pending_verification':",
                        "   - ERRORE EMAIL_NOT_VERIFIED",
                        "   - vai a STEP_ERRORE",
                        "",
                        "   SE user_record.status == 'suspended':",
                        "   - ERRORE ACCOUNT_SUSPENDED",
                        "   - vai a STEP_ERRORE",
                        "",
                        "   SE user_record.status == 'active':",
                        "   - vai a STEP_4"
                    ],
                    
                    "output": {
                        "account_attivo": True
                    },
                    
                    "prossimo_step_successo": "STEP_4",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_4": {
                    "nome": "Verifica password",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che la password sia corretta",
                    
                    "input": {
                        "password": "da STEP_1",
                        "user_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Verifica password con bcrypt:",
                        "   is_valid = bcrypt.compare(password, user_record.password_hash)",
                        "",
                        "2. IMPORTANTE: Elimina password in chiaro dalla memoria",
                        "",
                        "3. SE is_valid == false:",
                        "   - ERRORE CREDENTIALS_INVALID",
                        "   - vai a STEP_ERRORE",
                        "",
                        "4. SE is_valid == true:",
                        "   - vai a STEP_5"
                    ],
                    
                    "output": {
                        "password_valida": True
                    },
                    
                    "prossimo_step_successo": "STEP_5",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_5": {
                    "nome": "Generazione access token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema genera un JWT access token",
                    
                    "input": {
                        "user_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Prepara payload JWT:",
                        "   payload = {",
                        "     'sub': user_record.id,",
                        "     'email': user_record.email,",
                        "     'username': user_record.username,",
                        "     'role': user_record.role,",
                        "     'iat': NOW() (in secondi epoch),",
                        "     'exp': NOW() + 15 minuti (in secondi epoch)",
                        "   }",
                        "",
                        "2. Genera JWT:",
                        "   access_token = jwt.sign(payload, JWT_SECRET, algorithm='HS256')",
                        "",
                        "3. Calcola hash del token per storage:",
                        "   access_token_hash = sha256(access_token)",
                        "",
                        "4. vai a STEP_6"
                    ],
                    
                    "output": {
                        "access_token": "stringa JWT",
                        "access_token_hash": "hash SHA256 del token",
                        "access_token_expires_at": "NOW() + 15 minuti"
                    },
                    
                    "prossimo_step": "STEP_6"
                },
                
                "STEP_6": {
                    "nome": "Generazione refresh token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema genera un refresh token",
                    
                    "input": {},
                    
                    "azioni": [
                        "1. Genera refresh token casuale:",
                        "   refresh_token = crypto.randomBytes(64).toString('hex')",
                        "",
                        "2. Calcola hash per storage:",
                        "   refresh_token_hash = sha256(refresh_token)",
                        "",
                        "3. vai a STEP_7"
                    ],
                    
                    "output": {
                        "refresh_token": "stringa 128 caratteri hex",
                        "refresh_token_hash": "hash SHA256 del token",
                        "refresh_token_expires_at": "NOW() + 7 giorni"
                    },
                    
                    "prossimo_step": "STEP_7"
                },
                
                "STEP_7": {
                    "nome": "Creazione sessione",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema salva la sessione nel database",
                    
                    "input": {
                        "user_record": "da STEP_2",
                        "access_token_hash": "da STEP_5",
                        "refresh_token_hash": "da STEP_6",
                        "access_token_expires_at": "da STEP_5",
                        "refresh_token_expires_at": "da STEP_6",
                        "device_info": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Genera session_id: session_id = uuid.v4()",
                        "",
                        "2. INSERT INTO SESSION:",
                        "   - id = {session_id}",
                        "   - user_id = {user_record.id}",
                        "   - access_token_hash = {access_token_hash}",
                        "   - refresh_token_hash = {refresh_token_hash}",
                        "   - device_info = {device_info}",
                        "   - access_token_expires_at = {access_token_expires_at}",
                        "   - refresh_token_expires_at = {refresh_token_expires_at}",
                        "   - is_active = true",
                        "   - created_at = NOW()",
                        "",
                        "3. vai a STEP_8"
                    ],
                    
                    "output": {
                        "session_id": "UUID della sessione"
                    },
                    
                    "prossimo_step": "STEP_8"
                },
                
                "STEP_8": {
                    "nome": "Aggiornamento last_login",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema aggiorna la data dell'ultimo login",
                    
                    "input": {
                        "user_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. UPDATE USER SET:",
                        "   - last_login_at = NOW()",
                        "   - updated_at = NOW()",
                        "   WHERE id = {user_record.id}",
                        "",
                        "2. vai a STEP_SUCCESSO"
                    ],
                    
                    "output": {
                        "last_login_aggiornato": True
                    },
                    
                    "prossimo_step": "STEP_SUCCESSO"
                },
                
                "STEP_SUCCESSO": {
                    "nome": "Login completato",
                    "attore": "SISTEMA",
                    
                    "input": {
                        "user_record": "da STEP_2",
                        "access_token": "da STEP_5",
                        "refresh_token": "da STEP_6"
                    },
                    
                    "output": {
                        "success": True,
                        "http_status": 200,
                        "body": {
                            "access_token": "{access_token}",
                            "refresh_token": "{refresh_token}",
                            "token_type": "Bearer",
                            "expires_in": 900,
                            "user": {
                                "id": "{user_record.id}",
                                "email": "{user_record.email}",
                                "username": "{user_record.username}",
                                "role": "{user_record.role}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                },
                
                "STEP_ERRORE": {
                    "nome": "Gestione errore",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": False,
                        "http_status": "401 o 403",
                        "body": {
                            "error": {
                                "code": "{codice_errore}",
                                "message": "{messaggio_errore}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                }
            },
            
            "errori_possibili": {
                "CREDENTIALS_MISSING": {
                    "http_status": 400,
                    "messaggio": "Email/username e password sono richiesti"
                },
                "CREDENTIALS_INVALID": {
                    "http_status": 401,
                    "messaggio": "Email/username o password non corretti"
                },
                "EMAIL_NOT_VERIFIED": {
                    "http_status": 403,
                    "messaggio": "Email non verificata. Controlla la tua casella email."
                },
                "ACCOUNT_SUSPENDED": {
                    "http_status": 403,
                    "messaggio": "Account sospeso. Contatta il supporto."
                }
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # WORKFLOW: LOGOUT
        # ═══════════════════════════════════════════════════════════════════════
        
        "LOGOUT": {
            "nome": "Disconnessione utente",
            "descrizione": "Processo per terminare la sessione corrente",
            "trigger": "Utente richiede logout",
            
            "step": {
                
                "STEP_1": {
                    "nome": "Estrazione token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema estrae l'access token dalla richiesta",
                    
                    "input": {
                        "authorization_header": {
                            "tipo": "string",
                            "obbligatorio": True,
                            "formato": "Bearer {access_token}",
                            "origine": "HTTP Header Authorization"
                        }
                    },
                    
                    "azioni": [
                        "1. Estrai header Authorization dalla richiesta HTTP",
                        "",
                        "2. SE header assente o non inizia con 'Bearer ':",
                        "   - ERRORE TOKEN_MISSING",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. Estrai token: access_token = header.substring(7)",
                        "",
                        "4. vai a STEP_2"
                    ],
                    
                    "output": {
                        "access_token": "stringa JWT"
                    },
                    
                    "prossimo_step_successo": "STEP_2",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_2": {
                    "nome": "Validazione token",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema verifica che il token sia valido",
                    
                    "input": {
                        "access_token": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. Verifica firma JWT:",
                        "   decoded = jwt.verify(access_token, JWT_SECRET)",
                        "",
                        "2. SE verifica fallisce:",
                        "   - ERRORE TOKEN_INVALID",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. Calcola hash del token:",
                        "   access_token_hash = sha256(access_token)",
                        "",
                        "4. vai a STEP_3"
                    ],
                    
                    "output": {
                        "decoded_token": "payload decodificato",
                        "access_token_hash": "hash del token"
                    },
                    
                    "prossimo_step_successo": "STEP_3",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_3": {
                    "nome": "Ricerca sessione",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema trova la sessione associata al token",
                    
                    "input": {
                        "access_token_hash": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT * FROM SESSION",
                        "   WHERE access_token_hash = {access_token_hash}",
                        "   AND is_active = true",
                        "",
                        "2. SE nessun risultato:",
                        "   - ERRORE SESSION_NOT_FOUND",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE trovata:",
                        "   - Salva come session_record",
                        "   - vai a STEP_4"
                    ],
                    
                    "output": {
                        "session_record": "record sessione"
                    },
                    
                    "prossimo_step_successo": "STEP_4",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_4": {
                    "nome": "Disattivazione sessione",
                    "attore": "SISTEMA",
                    "descrizione": "Il sistema disattiva la sessione",
                    
                    "input": {
                        "session_record": "da STEP_3"
                    },
                    
                    "azioni": [
                        "1. UPDATE SESSION SET:",
                        "   - is_active = false",
                        "   WHERE id = {session_record.id}",
                        "",
                        "2. vai a STEP_SUCCESSO"
                    ],
                    
                    "output": {
                        "sessione_disattivata": True
                    },
                    
                    "prossimo_step": "STEP_SUCCESSO"
                },
                
                "STEP_SUCCESSO": {
                    "nome": "Logout completato",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": True,
                        "http_status": 200,
                        "body": {
                            "message": "Logout effettuato con successo"
                        }
                    },
                    
                    "prossimo_step": "FINE"
                },
                
                "STEP_ERRORE": {
                    "nome": "Gestione errore",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": False,
                        "http_status": 401,
                        "body": {
                            "error": {
                                "code": "{codice_errore}",
                                "message": "{messaggio_errore}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                }
            },
            
            "errori_possibili": {
                "TOKEN_MISSING": {
                    "http_status": 401,
                    "messaggio": "Token di autenticazione mancante"
                },
                "TOKEN_INVALID": {
                    "http_status": 401,
                    "messaggio": "Token di autenticazione non valido"
                },
                "SESSION_NOT_FOUND": {
                    "http_status": 401,
                    "messaggio": "Sessione non trovata o già terminata"
                }
            }
        },
        
        # ═══════════════════════════════════════════════════════════════════════
        # WORKFLOW: REFRESH TOKEN
        # ═══════════════════════════════════════════════════════════════════════
        
        "REFRESH_TOKEN": {
            "nome": "Rinnovo access token",
            "descrizione": "Processo per ottenere un nuovo access token usando il refresh token",
            "trigger": "Access token scaduto, client invia refresh token",
            
            "step": {
                
                "STEP_1": {
                    "nome": "Ricezione refresh token",
                    "attore": "SISTEMA",
                    
                    "input": {
                        "refresh_token": {
                            "tipo": "string",
                            "obbligatorio": True
                        }
                    },
                    
                    "azioni": [
                        "1. Estrai refresh_token dal body",
                        "2. SE vuoto: ERRORE REFRESH_TOKEN_MISSING, vai a STEP_ERRORE",
                        "3. Calcola hash: refresh_token_hash = sha256(refresh_token)",
                        "4. vai a STEP_2"
                    ],
                    
                    "output": {
                        "refresh_token_hash": "hash del token"
                    },
                    
                    "prossimo_step_successo": "STEP_2",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_2": {
                    "nome": "Ricerca sessione",
                    "attore": "SISTEMA",
                    
                    "input": {
                        "refresh_token_hash": "da STEP_1"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT * FROM SESSION",
                        "   WHERE refresh_token_hash = {refresh_token_hash}",
                        "   AND is_active = true",
                        "",
                        "2. SE nessun risultato:",
                        "   - ERRORE REFRESH_TOKEN_INVALID",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. SE session.refresh_token_expires_at < NOW():",
                        "   - ERRORE REFRESH_TOKEN_EXPIRED",
                        "   - vai a STEP_ERRORE",
                        "",
                        "4. vai a STEP_3"
                    ],
                    
                    "output": {
                        "session_record": "record sessione"
                    },
                    
                    "prossimo_step_successo": "STEP_3",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_3": {
                    "nome": "Verifica stato utente",
                    "attore": "SISTEMA",
                    
                    "input": {
                        "session_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. QUERY database:",
                        "   SELECT * FROM USER WHERE id = {session_record.user_id}",
                        "",
                        "2. SE user.status != 'active':",
                        "   - ERRORE ACCOUNT_NOT_ACTIVE",
                        "   - vai a STEP_ERRORE",
                        "",
                        "3. vai a STEP_4"
                    ],
                    
                    "output": {
                        "user_record": "record utente"
                    },
                    
                    "prossimo_step_successo": "STEP_4",
                    "prossimo_step_errore": "STEP_ERRORE"
                },
                
                "STEP_4": {
                    "nome": "Generazione nuovi token",
                    "attore": "SISTEMA",
                    
                    "input": {
                        "user_record": "da STEP_3",
                        "session_record": "da STEP_2"
                    },
                    
                    "azioni": [
                        "1. Genera nuovo access_token (come in LOGIN STEP_5)",
                        "2. Genera nuovo refresh_token (come in LOGIN STEP_6)",
                        "3. Calcola hash di entrambi",
                        "",
                        "4. UPDATE SESSION SET:",
                        "   - access_token_hash = {nuovo_access_token_hash}",
                        "   - refresh_token_hash = {nuovo_refresh_token_hash}",
                        "   - access_token_expires_at = NOW() + 15 minuti",
                        "   - refresh_token_expires_at = NOW() + 7 giorni",
                        "   WHERE id = {session_record.id}",
                        "",
                        "5. vai a STEP_SUCCESSO"
                    ],
                    
                    "output": {
                        "new_access_token": "nuovo JWT",
                        "new_refresh_token": "nuovo refresh token"
                    },
                    
                    "prossimo_step": "STEP_SUCCESSO"
                },
                
                "STEP_SUCCESSO": {
                    "nome": "Refresh completato",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": True,
                        "http_status": 200,
                        "body": {
                            "access_token": "{new_access_token}",
                            "refresh_token": "{new_refresh_token}",
                            "token_type": "Bearer",
                            "expires_in": 900
                        }
                    },
                    
                    "prossimo_step": "FINE"
                },
                
                "STEP_ERRORE": {
                    "nome": "Gestione errore",
                    "attore": "SISTEMA",
                    
                    "output": {
                        "success": False,
                        "http_status": 401,
                        "body": {
                            "error": {
                                "code": "{codice_errore}",
                                "message": "{messaggio_errore}"
                            }
                        }
                    },
                    
                    "prossimo_step": "FINE"
                }
            },
            
            "errori_possibili": {
                "REFRESH_TOKEN_MISSING": {
                    "http_status": 400,
                    "messaggio": "Refresh token mancante"
                },
                "REFRESH_TOKEN_INVALID": {
                    "http_status": 401,
                    "messaggio": "Refresh token non valido"
                },
                "REFRESH_TOKEN_EXPIRED": {
                    "http_status": 401,
                    "messaggio": "Refresh token scaduto. Effettua nuovamente il login."
                },
                "ACCOUNT_NOT_ACTIVE": {
                    "http_status": 403,
                    "messaggio": "Account non attivo"
                }
            }
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # REGOLE DI BUSINESS - MODULO IDENTITY
    # ═══════════════════════════════════════════════════════════════════════════
    
    "regole_business": {
        
        "RB_001": {
            "nome": "Unicità email",
            "descrizione": "Ogni email può essere registrata una sola volta nel sistema",
            "entita_coinvolte": ["USER"],
            "regola": "NON possono esistere due USER con la stessa email AND status != 'deleted'"
        },
        
        "RB_002": {
            "nome": "Unicità username",
            "descrizione": "Ogni username può essere usato da un solo utente",
            "entita_coinvolte": ["USER"],
            "regola": "NON possono esistere due USER con lo stesso username AND status != 'deleted'"
        },
        
        "RB_003": {
            "nome": "Verifica email obbligatoria",
            "descrizione": "Un utente non può accedere finché non verifica l'email",
            "entita_coinvolte": ["USER"],
            "regola": "SE USER.email_verified == false ALLORA LOGIN bloccato"
        },
        
        "RB_004": {
            "nome": "Transizioni stato valide",
            "descrizione": "Lo stato utente può cambiare solo secondo transizioni definite",
            "entita_coinvolte": ["USER"],
            "regola": """
                pending_verification → active (email verificata)
                pending_verification → deleted (cancellazione)
                active → suspended (violazione o admin)
                active → deleted (richiesta utente o admin)
                suspended → active (admin)
                suspended → deleted (admin)
                deleted → NESSUNA TRANSIZIONE
            """
        },
        
        "RB_005": {
            "nome": "Password sicura",
            "descrizione": "Le password devono rispettare requisiti minimi di sicurezza",
            "entita_coinvolte": ["USER"],
            "regola": "Password DEVE avere: min 8 caratteri, almeno 1 maiuscola, almeno 1 minuscola, almeno 1 numero"
        },
        
        "RB_006": {
            "nome": "Scadenza token verifica email",
            "descrizione": "I token di verifica email scadono dopo 24 ore",
            "entita_coinvolte": ["EMAIL_VERIFICATION_TOKEN"],
            "regola": "SE NOW() > expires_at ALLORA token non valido"
        },
        
        "RB_007": {
            "nome": "Scadenza token reset password",
            "descrizione": "I token di reset password scadono dopo 1 ora",
            "entita_coinvolte": ["PASSWORD_RESET_TOKEN"],
            "regola": "SE NOW() > expires_at ALLORA token non valido"
        },
        
        "RB_008": {
            "nome": "Token monouso",
            "descrizione": "I token possono essere usati una sola volta",
            "entita_coinvolte": ["EMAIL_VERIFICATION_TOKEN", "PASSWORD_RESET_TOKEN"],
            "regola": "SE used_at != NULL ALLORA token non valido"
        },
        
        "RB_009": {
            "nome": "Sessione multipla",
            "descrizione": "Un utente può avere più sessioni attive contemporaneamente",
            "entita_coinvolte": ["USER", "SESSION"],
            "regola": "NESSUN limite al numero di SESSION con is_active = true per un USER"
        },
        
        "RB_010": {
            "nome": "Invalidazione sessioni su reset password",
            "descrizione": "Quando un utente resetta la password, tutte le sessioni vengono invalidate",
            "entita_coinvolte": ["USER", "SESSION", "PASSWORD_RESET_TOKEN"],
            "regola": "Quando PASSWORD_RESET_TOKEN viene usato, SET is_active = false per TUTTE le SESSION del USER"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # [RISERVATO] EDGE CASES - MODULO IDENTITY
    # ═══════════════════════════════════════════════════════════════════════════
    
    "edge_cases": {
        "_nota": "Questa sezione è riservata per implementazione futura",
        "_versione_pianificata": "v1.1",
        "_esempi_previsti": [
            "Utente tenta registrazione con email eliminata soft-delete",
            "Token email verification usato dopo cambio email",
            "Sessione refresh mentre utente viene sospeso",
            "Race condition su registrazione username"
        ]
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: SOCIAL
# VERSIONE: 1.0
# DESCRIZIONE: Connessioni tra utenti, follow/follower, feed personalizzato
# DIPENDENZE: IDENTITY, CONTENT
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_SOCIAL = {
    "nome": "SOCIAL",
    "versione": "1.0",
    "descrizione": "Connessioni tra utenti, follow/follower, feed personalizzato",
    "dipendenze": ["IDENTITY", "CONTENT"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FOLLOW
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW": {
            "descrizione": "Relazione di follow tra due utenti (unidirezionale)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco della relazione"
                },
                "follower_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che segue"
                },
                "followed_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente seguito"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["pending", "active", "blocked"],
                    "descrizione": "Stato della relazione (pending se account privato)"
                },
                "notifications_enabled": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": True,
                    "descrizione": "Se ricevere notifiche per i post di questo utente"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di creazione del follow"
                },
                "accepted_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di accettazione (per account privati)"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["follower_id", "followed_id"],
                    "descrizione": "Un utente può seguire un altro utente una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "follower_id != followed_id",
                    "descrizione": "Un utente non può seguire se stesso"
                }
            ],
            
            "stati": ["pending", "active", "blocked"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["active", "blocked"],
                    "transizione_a_active": {
                        "condizione": "followed_user accetta la richiesta",
                        "azione": "Imposta status = 'active', accepted_at = now()"
                    },
                    "transizione_a_blocked": {
                        "condizione": "followed_user rifiuta la richiesta",
                        "azione": "Elimina record FOLLOW"
                    }
                },
                "active": {
                    "prossimi_stati_possibili": ["blocked"],
                    "transizione_a_blocked": {
                        "condizione": "follower_user smette di seguire OPPURE followed_user blocca",
                        "azione": "Elimina record FOLLOW"
                    }
                },
                "blocked": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale, il record viene eliminato"
                }
            },
            
            "indici": [
                {"campi": ["follower_id", "followed_id"], "tipo": "unique"},
                {"campi": ["followed_id", "status"], "tipo": "composite"},
                {"campi": ["follower_id", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: BLOCK
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK": {
            "descrizione": "Blocco di un utente da parte di un altro",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco del blocco"
                },
                "blocker_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che blocca"
                },
                "blocked_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente bloccato"
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 500,
                    "visible": "owner_only",
                    "descrizione": "Motivo del blocco (opzionale, privato)"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora del blocco"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["blocker_id", "blocked_id"],
                    "descrizione": "Un utente può bloccare un altro utente una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "blocker_id != blocked_id",
                    "descrizione": "Un utente non può bloccare se stesso"
                }
            ],
            
            "indici": [
                {"campi": ["blocker_id", "blocked_id"], "tipo": "unique"},
                {"campi": ["blocked_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: USER_STATS
        # ───────────────────────────────────────────────────────────────────────
        "USER_STATS": {
            "descrizione": "Statistiche aggregate per un utente (contatori)",
            
            "attributi": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente"
                },
                "followers_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di follower"
                },
                "following_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di utenti seguiti"
                },
                "posts_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di post pubblicati"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora dell'ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["user_id"], "tipo": "unique"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FEED_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "FEED_ITEM": {
            "descrizione": "Elemento nel feed di un utente (denormalizzato per performance)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente proprietario del feed"
                },
                "post_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "POST.id",
                    "descrizione": "ID del post nel feed"
                },
                "post_author_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'autore del post (denormalizzato)"
                },
                "feed_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["following", "suggested", "promoted"],
                    "descrizione": "Tipo di feed item"
                },
                "relevance_score": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Score di rilevanza per ordinamento"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di inserimento nel feed"
                },
                "seen_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora in cui l'utente ha visto l'item"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "POST"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["user_id", "post_id"],
                    "descrizione": "Un post appare una sola volta nel feed di un utente"
                }
            ],
            
            "indici": [
                {"campi": ["user_id", "created_at"], "tipo": "composite"},
                {"campi": ["user_id", "relevance_score"], "tipo": "composite"},
                {"campi": ["post_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO SOCIAL
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: FOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW_USER": {
            "descrizione": "Inizia a seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "validazione": "UUID dell'utente da seguire"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_FOLLOW_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id",
                    "errore_se_falsa": "ERROR_ALREADY_FOLLOWING"
                },
                {
                    "id": "PRE_5",
                    "condizione": "NON esiste BLOCK con blocker_id == input.user_id E blocked_id == current_user.id",
                    "errore_se_falsa": "ERROR_USER_BLOCKED_YOU"
                },
                {
                    "id": "PRE_6",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_YOU_BLOCKED_USER"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera profilo dell'utente target",
                    "dettaglio": """
                        target_profile = SELECT privacy_settings FROM PROFILE WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Determina stato iniziale del follow",
                    "dettaglio": """
                        SE target_profile.privacy_settings.profile_visibility == 'private':
                            initial_status = 'pending'
                            accepted_at = null
                        ALTRIMENTI:
                            initial_status = 'active'
                            accepted_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record FOLLOW",
                    "dettaglio": """
                        follow_id = genera_uuid_v4()
                        
                        INSERT INTO FOLLOW:
                        - id = follow_id
                        - follower_id = current_user.id
                        - followed_id = input.user_id
                        - status = initial_status
                        - notifications_enabled = true
                        - created_at = now()
                        - accepted_at = accepted_at
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna contatori SE status è active",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Incrementa following_count del follower
                            UPDATE USER_STATS SET 
                            - following_count = following_count + 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                            
                            # Incrementa followers_count del followed
                            UPDATE USER_STATS SET 
                            - followers_count = followers_count + 1
                            - updated_at = now()
                            WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Crea notifica",
                    "dettaglio": """
                        SE initial_status == 'pending':
                            crea_notifica(
                                recipient_id = input.user_id,
                                type = 'follow_request',
                                source_user_id = current_user.id
                            )
                        ALTRIMENTI:
                            crea_notifica(
                                recipient_id = input.user_id,
                                type = 'new_follower',
                                source_user_id = current_user.id
                            )
                    """
                },
                {
                    "numero": 6,
                    "azione": "Popola feed se follow è attivo",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Aggiungi ultimi N post dell'utente seguito al feed
                            recent_posts = SELECT id, author_id, published_at FROM POST
                            WHERE author_id == input.user_id
                            AND status == 'published'
                            AND visibility IN ['public', 'followers_only']
                            ORDER BY published_at DESC
                            LIMIT 20
                            
                            PER OGNI post IN recent_posts:
                                INSERT INTO FEED_ITEM:
                                - id = genera_uuid_v4()
                                - user_id = current_user.id
                                - post_id = post.id
                                - post_author_id = post.author_id
                                - feed_type = 'following'
                                - relevance_score = calcola_score(post)
                                - created_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "follow": "oggetto FOLLOW",
                "status": "active | pending"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_FOLLOW_SELF": {
                    "codice": "SOCIAL_002",
                    "messaggio": "Non puoi seguire te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_003",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FOLLOWING": {
                    "codice": "SOCIAL_004",
                    "messaggio": "Stai già seguendo questo utente",
                    "http_status": 409
                },
                "ERROR_USER_BLOCKED_YOU": {
                    "codice": "SOCIAL_005",
                    "messaggio": "Non puoi seguire questo utente",
                    "http_status": 403
                },
                "ERROR_YOU_BLOCKED_USER": {
                    "codice": "SOCIAL_006",
                    "messaggio": "Devi prima sbloccare questo utente",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNFOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNFOLLOW_USER": {
            "descrizione": "Smetti di seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id",
                    "errore_se_falsa": "ERROR_NOT_FOLLOWING"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera record FOLLOW",
                    "dettaglio": """
                        follow = SELECT * FROM FOLLOW 
                        WHERE follower_id == current_user.id 
                        AND followed_id == input.user_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatori SE follow era attivo",
                    "dettaglio": """
                        SE follow.status == 'active':
                            # Decrementa following_count del follower
                            UPDATE USER_STATS SET 
                            - following_count = following_count - 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                            
                            # Decrementa followers_count del followed
                            UPDATE USER_STATS SET 
                            - followers_count = followers_count - 1
                            - updated_at = now()
                            WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Elimina record FOLLOW",
                    "dettaglio": """
                        DELETE FROM FOLLOW WHERE id == follow.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Rimuovi post dal feed",
                    "dettaglio": """
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == current_user.id 
                        AND post_author_id == input.user_id
                        AND feed_type == 'following'
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Non segui più questo utente"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_FOLLOWING": {
                    "codice": "SOCIAL_011",
                    "messaggio": "Non stai seguendo questo utente",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ACCEPT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "ACCEPT_FOLLOW_REQUEST": {
            "descrizione": "Accetta una richiesta di follow (per account privati)",
            "attore": "utente_autenticato",
            
            "input": {
                "follower_id": {
                    "tipo": "uuid",
                    "required": True,
                    "descrizione": "ID dell'utente che ha richiesto di seguirti"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id E status == 'pending'",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Aggiorna FOLLOW a status active",
                    "dettaglio": """
                        UPDATE FOLLOW SET
                        - status = 'active'
                        - accepted_at = now()
                        WHERE follower_id == input.follower_id 
                        AND followed_id == current_user.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatori",
                    "dettaglio": """
                        # Incrementa following_count del follower
                        UPDATE USER_STATS SET 
                        - following_count = following_count + 1
                        - updated_at = now()
                        WHERE user_id == input.follower_id
                        
                        # Incrementa followers_count del current_user
                        UPDATE USER_STATS SET 
                        - followers_count = followers_count + 1
                        - updated_at = now()
                        WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Notifica il follower",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = input.follower_id,
                            type = 'follow_request_accepted',
                            source_user_id = current_user.id
                        )
                    """
                },
                {
                    "numero": 4,
                    "azione": "Popola feed del nuovo follower",
                    "dettaglio": """
                        recent_posts = SELECT id, author_id FROM POST
                        WHERE author_id == current_user.id
                        AND status == 'published'
                        ORDER BY published_at DESC
                        LIMIT 20
                        
                        PER OGNI post IN recent_posts:
                            INSERT INTO FEED_ITEM...
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "FOLLOW.status == 'active'"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow accettata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_021",
                    "messaggio": "Richiesta di follow non trovata",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: REJECT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "REJECT_FOLLOW_REQUEST": {
            "descrizione": "Rifiuta una richiesta di follow",
            "attore": "utente_autenticato",
            
            "input": {
                "follower_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id E status == 'pending'",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record FOLLOW",
                    "dettaglio": """
                        DELETE FROM FOLLOW 
                        WHERE follower_id == input.follower_id 
                        AND followed_id == current_user.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow rifiutata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_031",
                    "messaggio": "Richiesta di follow non trovata",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: BLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK_USER": {
            "descrizione": "Blocca un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 500 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_BLOCK_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.user_id",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_ALREADY_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Rimuovi follow reciproci",
                    "dettaglio": """
                        # Se current_user segue target
                        follow1 = SELECT * FROM FOLLOW WHERE follower_id == current_user.id AND followed_id == input.user_id
                        SE follow1 esiste E follow1.status == 'active':
                            # Aggiorna contatori
                            UPDATE USER_STATS SET following_count = following_count - 1 WHERE user_id == current_user.id
                            UPDATE USER_STATS SET followers_count = followers_count - 1 WHERE user_id == input.user_id
                        DELETE FROM FOLLOW WHERE follower_id == current_user.id AND followed_id == input.user_id
                        
                        # Se target segue current_user
                        follow2 = SELECT * FROM FOLLOW WHERE follower_id == input.user_id AND followed_id == current_user.id
                        SE follow2 esiste E follow2.status == 'active':
                            # Aggiorna contatori
                            UPDATE USER_STATS SET following_count = following_count - 1 WHERE user_id == input.user_id
                            UPDATE USER_STATS SET followers_count = followers_count - 1 WHERE user_id == current_user.id
                        DELETE FROM FOLLOW WHERE follower_id == input.user_id AND followed_id == current_user.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea record BLOCK",
                    "dettaglio": """
                        INSERT INTO BLOCK:
                        - id = genera_uuid_v4()
                        - blocker_id = current_user.id
                        - blocked_id = input.user_id
                        - reason = input.reason
                        - created_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Rimuovi post dal feed",
                    "dettaglio": """
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == current_user.id 
                        AND post_author_id == input.user_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id"
                },
                {
                    "id": "POST_2",
                    "condizione": "NON esiste FOLLOW tra i due utenti"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente bloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_BLOCK_SELF": {
                    "codice": "SOCIAL_041",
                    "messaggio": "Non puoi bloccare te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_042",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_BLOCKED": {
                    "codice": "SOCIAL_043",
                    "messaggio": "Utente già bloccato",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNBLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNBLOCK_USER": {
            "descrizione": "Sblocca un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_NOT_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record BLOCK",
                    "dettaglio": """
                        DELETE FROM BLOCK 
                        WHERE blocker_id == current_user.id 
                        AND blocked_id == input.user_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente sbloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_BLOCKED": {
                    "codice": "SOCIAL_051",
                    "messaggio": "Utente non bloccato",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_FEED
        # ───────────────────────────────────────────────────────────────────────
        "GET_FEED": {
            "descrizione": "Recupera il feed personalizzato dell'utente",
            "attore": "utente_autenticato",
            
            "input": {
                "cursor": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Cursore per paginazione (ID dell'ultimo item visto)"
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20,
                    "validazione": "1-50"
                },
                "feed_type": {
                    "tipo": "enum",
                    "required": False,
                    "default": "all",
                    "validazione": "all | following | suggested"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query per feed items",
                    "dettaglio": """
                        query = SELECT fi.*, p.* 
                        FROM FEED_ITEM fi
                        JOIN POST p ON fi.post_id = p.id
                        WHERE fi.user_id == current_user.id
                        AND p.status == 'published'
                        
                        SE input.feed_type != 'all':
                            query += AND fi.feed_type == input.feed_type
                        
                        SE input.cursor fornito:
                            cursor_item = SELECT created_at FROM FEED_ITEM WHERE id == input.cursor
                            query += AND fi.created_at < cursor_item.created_at
                        
                        query += ORDER BY fi.created_at DESC
                        query += LIMIT input.limit + 1  # +1 per determinare se ci sono altri risultati
                    """
                },
                {
                    "numero": 2,
                    "azione": "Esegui query",
                    "dettaglio": """
                        results = esegui_query(query)
                        
                        SE results.length > input.limit:
                            has_more = true
                            results = results[0:input.limit]  # Rimuovi l'elemento extra
                        ALTRIMENTI:
                            has_more = false
                    """
                },
                {
                    "numero": 3,
                    "azione": "Arricchisci dati per ogni post",
                    "dettaglio": """
                        PER OGNI item IN results:
                            # Aggiungi dati autore
                            item.author = SELECT id, username FROM USER WHERE id == item.post_author_id
                            item.author_profile = SELECT display_name, avatar_url FROM PROFILE WHERE user_id == item.post_author_id
                            
                            # Aggiungi reazione dell'utente corrente
                            item.user_reaction = SELECT reaction_type FROM REACTION 
                            WHERE user_id == current_user.id 
                            AND target_type == 'post' 
                            AND target_id == item.post_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Marca items come visti",
                    "dettaglio": """
                        PER OGNI item IN results:
                            SE item.seen_at == null:
                                UPDATE FEED_ITEM SET seen_at = now() WHERE id == item.id
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "items": "array di feed items con post e dati autore",
                "next_cursor": "ID dell'ultimo item (per paginazione) oppure null",
                "has_more": "boolean"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_FOLLOWERS
        # ───────────────────────────────────────────────────────────────────────
        "GET_FOLLOWERS": {
            "descrizione": "Recupera la lista dei follower di un utente",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica permessi di visibilità",
                    "dettaglio": """
                        profile = SELECT privacy_settings FROM PROFILE WHERE user_id == input.user_id
                        
                        SE profile.privacy_settings.profile_visibility == 'private':
                            SE NON utente_autenticato:
                                ERRORE: ERROR_PRIVATE_PROFILE
                            SE current_user.id != input.user_id:
                                is_follower = EXISTS FOLLOW 
                                WHERE follower_id == current_user.id 
                                AND followed_id == input.user_id 
                                AND status == 'active'
                                
                                SE NOT is_follower:
                                    ERRORE: ERROR_PRIVATE_PROFILE
                    """
                },
                {
                    "numero": 2,
                    "azione": "Query follower",
                    "dettaglio": """
                        followers = SELECT u.id, u.username, p.display_name, p.avatar_url, f.created_at
                        FROM FOLLOW f
                        JOIN USER u ON f.follower_id = u.id
                        JOIN PROFILE p ON u.id = p.user_id
                        WHERE f.followed_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                        OFFSET based on cursor
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiungi stato relazione se autenticato",
                    "dettaglio": """
                        SE utente_autenticato:
                            PER OGNI follower IN followers:
                                # Verifico se io seguo questo follower
                                follower.is_following = EXISTS FOLLOW 
                                WHERE follower_id == current_user.id 
                                AND followed_id == follower.id 
                                AND status == 'active'
                                
                                # Verifico se questo follower mi segue
                                follower.follows_me = EXISTS FOLLOW 
                                WHERE follower_id == follower.id 
                                AND followed_id == current_user.id 
                                AND status == 'active'
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "followers": "array di utenti con profilo e stato relazione",
                "total_count": "numero totale di follower",
                "next_cursor": "cursore per paginazione"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_070",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_PRIVATE_PROFILE": {
                    "codice": "SOCIAL_071",
                    "messaggio": "Profilo privato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_FOLLOWING
        # ───────────────────────────────────────────────────────────────────────
        "GET_FOLLOWING": {
            "descrizione": "Recupera la lista degli utenti seguiti",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica permessi (stessa logica di GET_FOLLOWERS)",
                    "dettaglio": "..."
                },
                {
                    "numero": 2,
                    "azione": "Query following",
                    "dettaglio": """
                        following = SELECT u.id, u.username, p.display_name, p.avatar_url, f.created_at
                        FROM FOLLOW f
                        JOIN USER u ON f.followed_id = u.id
                        JOIN PROFILE p ON u.id = p.user_id
                        WHERE f.follower_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiungi stato relazione se autenticato",
                    "dettaglio": "..."
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "following": "array di utenti seguiti",
                "total_count": "numero totale",
                "next_cursor": "cursore"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_080",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_PRIVATE_PROFILE": {
                    "codice": "SOCIAL_081",
                    "messaggio": "Profilo privato",
                    "http_status": 403
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo SOCIAL"
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: MESSAGING
# VERSIONE: 1.0
# DESCRIZIONE: Messaggistica diretta, conversazioni, notifiche
# DIPENDENZE: IDENTITY
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_MESSAGING = {
    "nome": "MESSAGING",
    "versione": "1.0",
    "descrizione": "Messaggistica diretta, conversazioni, notifiche",
    "dipendenze": ["IDENTITY"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: CONVERSATION
        # ───────────────────────────────────────────────────────────────────────
        "CONVERSATION": {
            "descrizione": "Conversazione tra due o più utenti",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco della conversazione"
                },
                "type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["direct", "group"],
                    "descrizione": "Tipo di conversazione"
                },
                "name": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Nome della conversazione (solo per gruppi)"
                },
                "avatar_url": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "url",
                    "descrizione": "Immagine della conversazione (solo per gruppi)"
                },
                "created_by": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che ha creato la conversazione"
                },
                "last_message_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora dell'ultimo messaggio"
                },
                "last_message_preview": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "max_length": 100,
                    "descrizione": "Anteprima dell'ultimo messaggio (troncato)"
                },
                "message_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero totale di messaggi"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["active", "archived", "deleted"],
                    "descrizione": "Stato della conversazione"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora dell'ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["MESSAGE", "CONVERSATION_PARTICIPANT"],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["last_message_at"], "tipo": "standard"},
                {"campi": ["status"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: CONVERSATION_PARTICIPANT
        # ───────────────────────────────────────────────────────────────────────
        "CONVERSATION_PARTICIPANT": {
            "descrizione": "Partecipante a una conversazione",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "conversation_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "CONVERSATION.id",
                    "descrizione": "ID della conversazione"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del partecipante"
                },
                "role": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "member",
                    "enum_values": ["admin", "member"],
                    "descrizione": "Ruolo nella conversazione"
                },
                "nickname": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 50,
                    "descrizione": "Soprannome personalizzato per questo utente nella conversazione"
                },
                "last_read_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "descrizione": "Data e ora dell'ultimo messaggio letto"
                },
                "last_read_message_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "riferimento": "MESSAGE.id",
                    "descrizione": "ID dell'ultimo messaggio letto"
                },
                "unread_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di messaggi non letti"
                },
                "notifications_enabled": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": True,
                    "descrizione": "Se ricevere notifiche per questa conversazione"
                },
                "is_muted": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se la conversazione è silenziata"
                },
                "muted_until": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data e ora fino a cui la conversazione è silenziata"
                },
                "joined_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di ingresso nella conversazione"
                },
                "left_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di uscita dalla conversazione"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["CONVERSATION", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["conversation_id", "user_id"],
                    "descrizione": "Un utente può essere partecipante una sola volta per conversazione"
                }
            ],
            
            "indici": [
                {"campi": ["conversation_id", "user_id"], "tipo": "unique"},
                {"campi": ["user_id", "last_read_at"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: MESSAGE
        # ───────────────────────────────────────────────────────────────────────
        "MESSAGE": {
            "descrizione": "Messaggio in una conversazione",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco del messaggio"
                },
                "conversation_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "CONVERSATION.id",
                    "descrizione": "ID della conversazione"
                },
                "sender_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del mittente"
                },
                "message_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["text", "image", "video", "audio", "file", "system"],
                    "descrizione": "Tipo di messaggio"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 5000,
                    "descrizione": "Contenuto testuale del messaggio"
                },
                "media": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Media allegato al messaggio",
                    "schema": {
                        "id": "uuid del media",
                        "type": "image | video | audio | file",
                        "url": "URL del file",
                        "filename": "nome file originale",
                        "size_bytes": "dimensione",
                        "mime_type": "MIME type"
                    }
                },
                "reply_to_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "MESSAGE.id",
                    "descrizione": "ID del messaggio a cui si risponde"
                },
                "forwarded_from_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "MESSAGE.id",
                    "descrizione": "ID del messaggio originale se inoltrato"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "sent",
                    "enum_values": ["sending", "sent", "delivered", "read", "deleted"],
                    "descrizione": "Stato del messaggio"
                },
                "edited_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora dell'ultima modifica"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di invio"
                },
                "deleted_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di eliminazione"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["MESSAGE_READ_RECEIPT"],
                "appartiene_a": ["CONVERSATION", "USER"]
            },
            
            "stati": ["sending", "sent", "delivered", "read", "deleted"],
            
            "transizioni_stato": {
                "sending": {
                    "prossimi_stati_possibili": ["sent", "deleted"],
                    "transizione_a_sent": {
                        "condizione": "messaggio salvato con successo",
                        "azione": "Imposta status = 'sent'"
                    }
                },
                "sent": {
                    "prossimi_stati_possibili": ["delivered", "deleted"],
                    "transizione_a_delivered": {
                        "condizione": "almeno un destinatario ha ricevuto il messaggio",
                        "azione": "Imposta status = 'delivered'"
                    }
                },
                "delivered": {
                    "prossimi_stati_possibili": ["read", "deleted"],
                    "transizione_a_read": {
                        "condizione": "almeno un destinatario ha letto il messaggio",
                        "azione": "Imposta status = 'read'"
                    }
                },
                "read": {
                    "prossimi_stati_possibili": ["deleted"]
                },
                "deleted": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale"
                }
            },
            
            "indici": [
                {"campi": ["conversation_id", "created_at"], "tipo": "composite"},
                {"campi": ["sender_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: MESSAGE_READ_RECEIPT
        # ───────────────────────────────────────────────────────────────────────
        "MESSAGE_READ_RECEIPT": {
            "descrizione": "Conferma di lettura di un messaggio",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "message_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "MESSAGE.id",
                    "descrizione": "ID del messaggio"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che ha letto"
                },
                "read_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di lettura"
                }
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["message_id", "user_id"],
                    "descrizione": "Una sola conferma di lettura per utente per messaggio"
                }
            ],
            
            "indici": [
                {"campi": ["message_id", "user_id"], "tipo": "unique"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: NOTIFICATION
        # ───────────────────────────────────────────────────────────────────────
        "NOTIFICATION": {
            "descrizione": "Notifica per un utente",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "recipient_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente destinatario"
                },
                "type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "enum_values": [
                        "new_follower",
                        "follow_request",
                        "follow_request_accepted",
                        "mention",
                        "reaction",
                        "comment",
                        "reply",
                        "repost",
                        "message",
                        "system"
                    ],
                    "descrizione": "Tipo di notifica"
                },
                "title": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "max_length": 200,
                    "descrizione": "Titolo della notifica"
                },
                "body": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "max_length": 500,
                    "descrizione": "Corpo della notifica"
                },
                "source_user_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che ha generato la notifica"
                },
                "target_type": {
                    "tipo": "enum",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "enum_values": ["post", "comment", "conversation", "user"],
                    "descrizione": "Tipo di oggetto target"
                },
                "target_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "ID dell'oggetto target"
                },
                "action_url": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "format": "url",
                    "descrizione": "URL per l'azione (deep link)"
                },
                "metadata": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Dati aggiuntivi specifici per tipo"
                },
                "is_read": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se la notifica è stata letta"
                },
                "read_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di lettura"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora di creazione"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["recipient_id", "created_at"], "tipo": "composite"},
                {"campi": ["recipient_id", "is_read"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO MESSAGING
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: START_CONVERSATION
        # ───────────────────────────────────────────────────────────────────────
        "START_CONVERSATION": {
            "descrizione": "Avvia una nuova conversazione diretta o di gruppo",
            "attore": "utente_autenticato",
            
            "input": {
                "participant_ids": {
                    "tipo": "array",
                    "required": True,
                    "validazione": "array di UUID, minimo 1 elemento, massimo 50",
                    "descrizione": "ID degli utenti da aggiungere alla conversazione"
                },
                "name": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 100 caratteri, richiesto se più di 1 partecipante"
                },
                "initial_message": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 5000 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.participant_ids.length >= 1",
                    "errore_se_falsa": "ERROR_NO_PARTICIPANTS"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id NON è in input.participant_ids",
                    "errore_se_falsa": "ERROR_CANNOT_ADD_SELF"
                },
                {
                    "id": "PRE_4",
                    "condizione": "Tutti gli user_id in participant_ids esistono e hanno status == 'active'",
                    "errore_se_falsa": "ERROR_INVALID_PARTICIPANT"
                },
                {
                    "id": "PRE_5",
                    "condizione": "Per ogni participant_id: NON esiste BLOCK dove blocked_id == current_user.id",
                    "errore_se_falsa": "ERROR_BLOCKED_BY_USER"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Determina tipo conversazione",
                    "dettaglio": """
                        SE input.participant_ids.length == 1:
                            conversation_type = 'direct'
                        ALTRIMENTI:
                            conversation_type = 'group'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Per conversazioni dirette, verifica se esiste già",
                    "dettaglio": """
                        SE conversation_type == 'direct':
                            other_user_id = input.participant_ids[0]
                            
                            existing = SELECT c.* FROM CONVERSATION c
                            JOIN CONVERSATION_PARTICIPANT cp1 ON c.id = cp1.conversation_id
                            JOIN CONVERSATION_PARTICIPANT cp2 ON c.id = cp2.conversation_id
                            WHERE c.type == 'direct'
                            AND cp1.user_id == current_user.id
                            AND cp2.user_id == other_user_id
                            AND cp1.left_at IS NULL
                            AND cp2.left_at IS NULL
                            
                            SE existing esiste:
                                # Ritorna conversazione esistente
                                RETURN existing
                    """
                },
                {
                    "numero": 3,
                    "azione": "Verifica permessi messaggistica per ogni partecipante",
                    "dettaglio": """
                        PER OGNI participant_id IN input.participant_ids:
                            profile = SELECT privacy_settings FROM PROFILE WHERE user_id == participant_id
                            allow_messages_from = profile.privacy_settings.allow_messages_from
                            
                            SE allow_messages_from == 'nobody':
                                ERRORE: ERROR_USER_DOES_NOT_ACCEPT_MESSAGES
                            ALTRIMENTI SE allow_messages_from == 'followers':
                                is_follower = EXISTS FOLLOW 
                                WHERE follower_id == participant_id 
                                AND followed_id == current_user.id 
                                AND status == 'active'
                                
                                SE NOT is_follower:
                                    ERRORE: ERROR_USER_ONLY_ACCEPTS_FOLLOWER_MESSAGES
                    """
                },
                {
                    "numero": 4,
                    "azione": "Crea record CONVERSATION",
                    "dettaglio": """
                        conversation_id = genera_uuid_v4()
                        
                        INSERT INTO CONVERSATION:
                        - id = conversation_id
                        - type = conversation_type
                        - name = input.name SE conversation_type == 'group' ALTRIMENTI null
                        - created_by = current_user.id
                        - message_count = 0
                        - status = 'active'
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 5,
                    "azione": "Aggiungi partecipanti",
                    "dettaglio": """
                        # Aggiungi current_user come admin
                        INSERT INTO CONVERSATION_PARTICIPANT:
                        - id = genera_uuid_v4()
                        - conversation_id = conversation_id
                        - user_id = current_user.id
                        - role = 'admin'
                        - unread_count = 0
                        - notifications_enabled = true
                        - is_muted = false
                        - joined_at = now()
                        
                        # Aggiungi altri partecipanti
                        PER OGNI participant_id IN input.participant_ids:
                            INSERT INTO CONVERSATION_PARTICIPANT:
                            - id = genera_uuid_v4()
                            - conversation_id = conversation_id
                            - user_id = participant_id
                            - role = 'member'
                            - unread_count = 0
                            - notifications_enabled = true
                            - is_muted = false
                            - joined_at = now()
                    """
                },
                {
                    "numero": 6,
                    "azione": "Invia messaggio iniziale se fornito",
                    "dettaglio": """
                        SE input.initial_message fornito E non vuoto:
                            esegui_operazione SEND_MESSAGE(
                                conversation_id = conversation_id,
                                content = input.initial_message
                            )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste CONVERSATION con id == conversation_id"
                },
                {
                    "id": "POST_2",
                    "condizione": "current_user è partecipante con role == 'admin'"
                },
                {
                    "id": "POST_3",
                    "condizione": "Tutti i participant_ids sono partecipanti"
                }
            ],
            
            "output": {
                "success": True,
                "conversation": "oggetto CONVERSATION con partecipanti",
                "is_new": "true se conversazione appena creata, false se esistente"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NO_PARTICIPANTS": {
                    "codice": "MSG_002",
                    "messaggio": "Almeno un partecipante richiesto",
                    "http_status": 400
                },
                "ERROR_CANNOT_ADD_SELF": {
                    "codice": "MSG_003",
                    "messaggio": "Non puoi aggiungere te stesso come partecipante",
                    "http_status": 400
                },
                "ERROR_INVALID_PARTICIPANT": {
                    "codice": "MSG_004",
                    "messaggio": "Uno o più partecipanti non validi",
                    "http_status": 400
                },
                "ERROR_BLOCKED_BY_USER": {
                    "codice": "MSG_005",
                    "messaggio": "Non puoi inviare messaggi a questo utente",
                    "http_status": 403
                },
                "ERROR_USER_DOES_NOT_ACCEPT_MESSAGES": {
                    "codice": "MSG_006",
                    "messaggio": "Questo utente non accetta messaggi",
                    "http_status": 403
                },
                "ERROR_USER_ONLY_ACCEPTS_FOLLOWER_MESSAGES": {
                    "codice": "MSG_007",
                    "messaggio": "Questo utente accetta messaggi solo dai follower",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: SEND_MESSAGE
        # ───────────────────────────────────────────────────────────────────────
        "SEND_MESSAGE": {
            "descrizione": "Invia un messaggio in una conversazione",
            "attore": "utente_autenticato",
            
            "input": {
                "conversation_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 5000 caratteri, richiesto se media_id non fornito"
                },
                "media_id": {
                    "tipo": "uuid",
                    "required": False,
                    "descrizione": "ID del media da allegare"
                },
                "reply_to_id": {
                    "tipo": "uuid",
                    "required": False,
                    "descrizione": "ID del messaggio a cui rispondere"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste CONVERSATION con id == input.conversation_id E status == 'active'",
                    "errore_se_falsa": "ERROR_CONVERSATION_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user è partecipante attivo (left_at == null)",
                    "errore_se_falsa": "ERROR_NOT_PARTICIPANT"
                },
                {
                    "id": "PRE_4",
                    "condizione": "input.content non vuoto OPPURE input.media_id fornito",
                    "errore_se_falsa": "ERROR_EMPTY_MESSAGE"
                },
                {
                    "id": "PRE_5",
                    "condizione": "SE input.reply_to_id: Esiste MESSAGE con id == input.reply_to_id E conversation_id == input.conversation_id",
                    "errore_se_falsa": "ERROR_INVALID_REPLY_TARGET"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Determina tipo messaggio",
                    "dettaglio": """
                        SE input.media_id fornito:
                            media = SELECT * FROM MEDIA_ITEM WHERE id == input.media_id
                            SE media.file_type == 'image':
                                message_type = 'image'
                            ALTRIMENTI SE media.file_type == 'video':
                                message_type = 'video'
                            ALTRIMENTI SE media.file_type == 'audio':
                                message_type = 'audio'
                            ALTRIMENTI:
                                message_type = 'file'
                        ALTRIMENTI:
                            message_type = 'text'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Costruisci oggetto media se fornito",
                    "dettaglio": """
                        SE input.media_id fornito:
                            media_obj = {
                                id: media.id,
                                type: media.file_type,
                                url: media.url,
                                filename: media.original_filename,
                                size_bytes: media.file_size_bytes,
                                mime_type: media.mime_type
                            }
                        ALTRIMENTI:
                            media_obj = null
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record MESSAGE",
                    "dettaglio": """
                        message_id = genera_uuid_v4()
                        
                        INSERT INTO MESSAGE:
                        - id = message_id
                        - conversation_id = input.conversation_id
                        - sender_id = current_user.id
                        - message_type = message_type
                        - content = input.content
                        - media = media_obj
                        - reply_to_id = input.reply_to_id
                        - status = 'sent'
                        - created_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna CONVERSATION",
                    "dettaglio": """
                        preview = SE input.content: tronca(input.content, 100) ALTRIMENTI "[Media]"
                        
                        UPDATE CONVERSATION SET
                        - last_message_at = now()
                        - last_message_preview = preview
                        - message_count = message_count + 1
                        - updated_at = now()
                        WHERE id == input.conversation_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Aggiorna contatori unread per altri partecipanti",
                    "dettaglio": """
                        UPDATE CONVERSATION_PARTICIPANT SET
                        - unread_count = unread_count + 1
                        WHERE conversation_id == input.conversation_id
                        AND user_id != current_user.id
                        AND left_at IS NULL
                    """
                },
                {
                    "numero": 6,
                    "azione": "Invia notifiche push agli altri partecipanti",
                    "dettaglio": """
                        participants = SELECT * FROM CONVERSATION_PARTICIPANT
                        WHERE conversation_id == input.conversation_id
                        AND user_id != current_user.id
                        AND left_at IS NULL
                        AND notifications_enabled == true
                        AND (is_muted == false OR muted_until < now())
                        
                        PER OGNI participant IN participants:
                            crea_notifica(
                                recipient_id = participant.user_id,
                                type = 'message',
                                source_user_id = current_user.id,
                                target_type = 'conversation',
                                target_id = input.conversation_id,
                                title = "{current_user.username}",
                                body = preview
                            )
                            
                            invia_push_notification(participant.user_id, notifica)
                    """
                },
                {
                    "numero": 7,
                    "azione": "Broadcast real-time via WebSocket",
                    "dettaglio": """
                        PER OGNI participant IN all_participants:
                            SE participant.user_id != current_user.id:
                                websocket_send(
                                    user_id = participant.user_id,
                                    event = 'new_message',
                                    data = message
                                )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste MESSAGE con id == message_id"
                },
                {
                    "id": "POST_2",
                    "condizione": "CONVERSATION.last_message_at aggiornato"
                }
            ],
            
            "output": {
                "success": True,
                "message": "oggetto MESSAGE creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CONVERSATION_NOT_FOUND": {
                    "codice": "MSG_011",
                    "messaggio": "Conversazione non trovata",
                    "http_status": 404
                },
                "ERROR_NOT_PARTICIPANT": {
                    "codice": "MSG_012",
                    "messaggio": "Non sei partecipante di questa conversazione",
                    "http_status": 403
                },
                "ERROR_EMPTY_MESSAGE": {
                    "codice": "MSG_013",
                    "messaggio": "Il messaggio non può essere vuoto",
                    "http_status": 400
                },
                "ERROR_INVALID_REPLY_TARGET": {
                    "codice": "MSG_014",
                    "messaggio": "Messaggio di risposta non valido",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_CONVERSATIONS
        # ───────────────────────────────────────────────────────────────────────
        "GET_CONVERSATIONS": {
            "descrizione": "Recupera la lista delle conversazioni dell'utente",
            "attore": "utente_autenticato",
            
            "input": {
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                },
                "filter": {
                    "tipo": "enum",
                    "required": False,
                    "default": "all",
                    "validazione": "all | unread"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Query conversazioni",
                    "dettaglio": """
                        query = SELECT c.*, cp.unread_count, cp.is_muted
                        FROM CONVERSATION c
                        JOIN CONVERSATION_PARTICIPANT cp ON c.id = cp.conversation_id
                        WHERE cp.user_id == current_user.id
                        AND cp.left_at IS NULL
                        AND c.status == 'active'
                        
                        SE input.filter == 'unread':
                            query += AND cp.unread_count > 0
                        
                        query += ORDER BY c.last_message_at DESC NULLS LAST
                        query += LIMIT input.limit
                    """
                },
                {
                    "numero": 2,
                    "azione": "Arricchisci con dati partecipanti",
                    "dettaglio": """
                        PER OGNI conversation IN results:
                            conversation.participants = SELECT u.id, u.username, p.display_name, p.avatar_url
                            FROM CONVERSATION_PARTICIPANT cp
                            JOIN USER u ON cp.user_id = u.id
                            JOIN PROFILE p ON u.id = p.user_id
                            WHERE cp.conversation_id == conversation.id
                            AND cp.left_at IS NULL
                            AND u.status == 'active'
                    """
                },
                {
                    "numero": 3,
                    "azione": "Per conversazioni dirette, imposta nome dal partecipante",
                    "dettaglio": """
                        PER OGNI conversation IN results:
                            SE conversation.type == 'direct':
                                other = primo elemento di conversation.participants dove user_id != current_user.id
                                conversation.display_name = other.display_name OR other.username
                                conversation.display_avatar = other.avatar_url
                            ALTRIMENTI:
                                conversation.display_name = conversation.name
                                conversation.display_avatar = conversation.avatar_url
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "conversations": "array di conversazioni con partecipanti",
                "next_cursor": "cursore per paginazione",
                "total_unread": "numero totale messaggi non letti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_MESSAGES
        # ───────────────────────────────────────────────────────────────────────
        "GET_MESSAGES": {
            "descrizione": "Recupera i messaggi di una conversazione",
            "attore": "utente_autenticato",
            
            "input": {
                "conversation_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "ID dell'ultimo messaggio visto (per paginazione)"
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 50
                },
                "direction": {
                    "tipo": "enum",
                    "required": False,
                    "default": "older",
                    "validazione": "older | newer",
                    "descrizione": "Direzione di paginazione rispetto al cursore"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste CONVERSATION con id == input.conversation_id",
                    "errore_se_falsa": "ERROR_CONVERSATION_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user è partecipante (anche se left_at non null, può vedere messaggi precedenti)",
                    "errore_se_falsa": "ERROR_NOT_PARTICIPANT"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Query messaggi",
                    "dettaglio": """
                        query = SELECT m.*, u.username, p.display_name, p.avatar_url
                        FROM MESSAGE m
                        JOIN USER u ON m.sender_id = u.id
                        JOIN PROFILE p ON u.id = p.user_id
                        WHERE m.conversation_id == input.conversation_id
                        AND m.status != 'deleted'
                        
                        SE input.cursor fornito:
                            cursor_msg = SELECT created_at FROM MESSAGE WHERE id == input.cursor
                            SE input.direction == 'older':
                                query += AND m.created_at < cursor_msg.created_at
                                query += ORDER BY m.created_at DESC
                            ALTRIMENTI:
                                query += AND m.created_at > cursor_msg.created_at
                                query += ORDER BY m.created_at ASC
                        ALTRIMENTI:
                            query += ORDER BY m.created_at DESC
                        
                        query += LIMIT input.limit
                    """
                },
                {
                    "numero": 2,
                    "azione": "Ordina messaggi cronologicamente",
                    "dettaglio": """
                        # I messaggi vanno restituiti in ordine cronologico (dal più vecchio al più recente)
                        # indipendentemente dalla direzione di paginazione
                        SE input.direction == 'older' OR cursor non fornito:
                            messages = reverse(results)
                        ALTRIMENTI:
                            messages = results
                    """
                },
                {
                    "numero": 3,
                    "azione": "Arricchisci con reply_to se presente",
                    "dettaglio": """
                        PER OGNI message IN messages:
                            SE message.reply_to_id != null:
                                message.reply_to = SELECT id, content, sender_id
                                FROM MESSAGE WHERE id == message.reply_to_id
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "messages": "array di messaggi ordinati cronologicamente",
                "has_more_older": "boolean - ci sono messaggi più vecchi",
                "has_more_newer": "boolean - ci sono messaggi più recenti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CONVERSATION_NOT_FOUND": {
                    "codice": "MSG_031",
                    "messaggio": "Conversazione non trovata",
                    "http_status": 404
                },
                "ERROR_NOT_PARTICIPANT": {
                    "codice": "MSG_032",
                    "messaggio": "Non hai accesso a questa conversazione",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: MARK_MESSAGES_READ
        # ───────────────────────────────────────────────────────────────────────
        "MARK_MESSAGES_READ": {
            "descrizione": "Marca i messaggi come letti",
            "attore": "utente_autenticato",
            
            "input": {
                "conversation_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "last_read_message_id": {
                    "tipo": "uuid",
                    "required": True,
                    "descrizione": "ID dell'ultimo messaggio letto"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "current_user è partecipante attivo",
                    "errore_se_falsa": "ERROR_NOT_PARTICIPANT"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Aggiorna CONVERSATION_PARTICIPANT",
                    "dettaglio": """
                        UPDATE CONVERSATION_PARTICIPANT SET
                        - last_read_at = now()
                        - last_read_message_id = input.last_read_message_id
                        - unread_count = 0
                        WHERE conversation_id == input.conversation_id
                        AND user_id == current_user.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea read receipts per messaggi non letti",
                    "dettaglio": """
                        unread_messages = SELECT id FROM MESSAGE
                        WHERE conversation_id == input.conversation_id
                        AND sender_id != current_user.id
                        AND created_at <= (SELECT created_at FROM MESSAGE WHERE id == input.last_read_message_id)
                        AND id NOT IN (SELECT message_id FROM MESSAGE_READ_RECEIPT WHERE user_id == current_user.id)
                        
                        PER OGNI message IN unread_messages:
                            INSERT INTO MESSAGE_READ_RECEIPT:
                            - id = genera_uuid_v4()
                            - message_id = message.id
                            - user_id = current_user.id
                            - read_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna status messaggi a 'read' se tutti hanno letto",
                    "dettaglio": """
                        # Per conversazioni dirette, il messaggio diventa 'read' quando l'altro utente legge
                        SE conversation.type == 'direct':
                            UPDATE MESSAGE SET status = 'read'
                            WHERE conversation_id == input.conversation_id
                            AND sender_id != current_user.id
                            AND status IN ['sent', 'delivered']
                            AND created_at <= (SELECT created_at FROM MESSAGE WHERE id == input.last_read_message_id)
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica mittenti via WebSocket",
                    "dettaglio": """
                        PER OGNI message IN messages_marked_as_read:
                            websocket_send(
                                user_id = message.sender_id,
                                event = 'message_read',
                                data = {
                                    message_id: message.id,
                                    read_by: current_user.id,
                                    read_at: now()
                                }
                            )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "CONVERSATION_PARTICIPANT.unread_count == 0"
                }
            ],
            
            "output": {
                "success": True,
                "messages_marked": "numero di messaggi marcati come letti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_PARTICIPANT": {
                    "codice": "MSG_041",
                    "messaggio": "Non sei partecipante",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_NOTIFICATIONS
        # ───────────────────────────────────────────────────────────────────────
        "GET_NOTIFICATIONS": {
            "descrizione": "Recupera le notifiche dell'utente",
            "attore": "utente_autenticato",
            
            "input": {
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                },
                "filter": {
                    "tipo": "enum",
                    "required": False,
                    "default": "all",
                    "validazione": "all | unread"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Query notifiche",
                    "dettaglio": """
                        query = SELECT n.*, u.username, p.display_name, p.avatar_url
                        FROM NOTIFICATION n
                        LEFT JOIN USER u ON n.source_user_id = u.id
                        LEFT JOIN PROFILE p ON u.id = p.user_id
                        WHERE n.recipient_id == current_user.id
                        
                        SE input.filter == 'unread':
                            query += AND n.is_read == false
                        
                        query += ORDER BY n.created_at DESC
                        query += LIMIT input.limit
                    """
                },
                {
                    "numero": 2,
                    "azione": "Raggruppa notifiche simili (opzionale)",
                    "dettaglio": """
                        # Esempio: "Mario e altri 5 hanno reagito al tuo post"
                        # Implementazione opzionale per UX migliore
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "notifications": "array di notifiche",
                "unread_count": "numero totale non lette",
                "next_cursor": "cursore paginazione"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: MARK_NOTIFICATIONS_READ
        # ───────────────────────────────────────────────────────────────────────
        "MARK_NOTIFICATIONS_READ": {
            "descrizione": "Marca le notifiche come lette",
            "attore": "utente_autenticato",
            
            "input": {
                "notification_ids": {
                    "tipo": "array",
                    "required": False,
                    "descrizione": "Lista di ID notifiche da marcare. Se vuoto, marca tutte."
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Aggiorna notifiche",
                    "dettaglio": """
                        SE input.notification_ids fornito E non vuoto:
                            UPDATE NOTIFICATION SET
                            - is_read = true
                            - read_at = now()
                            WHERE id IN input.notification_ids
                            AND recipient_id == current_user.id
                        ALTRIMENTI:
                            UPDATE NOTIFICATION SET
                            - is_read = true
                            - read_at = now()
                            WHERE recipient_id == current_user.id
                            AND is_read == false
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "marked_count": "numero di notifiche marcate"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MSG_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo MESSAGING"
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: MARKETPLACE
# VERSIONE: 1.0
# DESCRIZIONE: Annunci, offerte, transazioni, recensioni
# DIPENDENZE: IDENTITY, MESSAGING
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_MARKETPLACE = {
    "nome": "MARKETPLACE",
    "versione": "1.0",
    "descrizione": "Annunci, offerte, transazioni, recensioni",
    "dipendenze": ["IDENTITY", "MESSAGING"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: LISTING
        # ───────────────────────────────────────────────────────────────────────
        "LISTING": {
            "descrizione": "Annuncio di vendita prodotto/servizio",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco dell'annuncio"
                },
                "seller_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del venditore"
                },
                "listing_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "enum_values": ["product", "service", "job", "housing", "vehicle"],
                    "descrizione": "Tipo di annuncio"
                },
                "title": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_length": 5,
                    "max_length": 200,
                    "descrizione": "Titolo dell'annuncio"
                },
                "description": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_length": 20,
                    "max_length": 10000,
                    "descrizione": "Descrizione dettagliata"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "riferimento": "CATEGORY.id",
                    "descrizione": "Categoria dell'annuncio"
                },
                "subcategory_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "riferimento": "CATEGORY.id",
                    "descrizione": "Sottocategoria"
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Informazioni sul prezzo",
                    "schema": {
                        "amount": "numero decimale (centesimi)",
                        "currency": "codice ISO 4217 (EUR, USD, etc.)",
                        "type": "fixed | negotiable | auction | free | contact",
                        "min_amount": "prezzo minimo per auction (opzionale)",
                        "original_amount": "prezzo originale se scontato (opzionale)"
                    }
                },
                "condition": {
                    "tipo": "enum",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "enum_values": ["new", "like_new", "good", "fair", "for_parts"],
                    "descrizione": "Condizione del prodotto (solo per product)"
                },
                "media": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Immagini e video dell'annuncio",
                    "schema": {
                        "tipo": "array",
                        "max_items": 20,
                        "items": {
                            "id": "uuid",
                            "type": "image | video",
                            "url": "URL",
                            "thumbnail_url": "URL thumbnail",
                            "order": "ordine visualizzazione",
                            "is_cover": "boolean - se è immagine principale"
                        }
                    }
                },
                "location": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Posizione geografica",
                    "schema": {
                        "address": "indirizzo (opzionale)",
                        "city": "città",
                        "state": "stato/provincia",
                        "country": "paese",
                        "postal_code": "CAP",
                        "latitude": "latitudine",
                        "longitude": "longitudine",
                        "radius_km": "raggio di servizio (per servizi)"
                    }
                },
                "attributes": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Attributi specifici per categoria (es: taglia, marca, anno)"
                },
                "shipping_options": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Opzioni di spedizione disponibili",
                    "schema": {
                        "tipo": "array",
                        "items": {
                            "method": "pickup | shipping | delivery",
                            "price": "costo spedizione",
                            "estimated_days": "giorni stimati"
                        }
                    }
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "draft",
                    "enum_values": ["draft", "pending_review", "active", "reserved", "sold", "expired", "deleted"],
                    "descrizione": "Stato dell'annuncio"
                },
                "visibility": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "public",
                    "enum_values": ["public", "unlisted", "private"],
                    "descrizione": "Visibilità dell'annuncio"
                },
                "views_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di visualizzazioni"
                },
                "favorites_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di utenti che hanno salvato nei preferiti"
                },
                "inquiries_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di richieste informazioni"
                },
                "promoted_until": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data fine promozione (annuncio in evidenza)"
                },
                "expires_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "descrizione": "Data scadenza annuncio"
                },
                "published_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data pubblicazione"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["OFFER", "FAVORITE", "LISTING_INQUIRY"],
                "appartiene_a": ["USER", "CATEGORY"]
            },
            
            "stati": ["draft", "pending_review", "active", "reserved", "sold", "expired", "deleted"],
            
            "transizioni_stato": {
                "draft": {
                    "prossimi_stati_possibili": ["pending_review", "active", "deleted"],
                    "transizione_a_pending_review": {
                        "condizione": "seller pubblica E moderazione_abilitata == true",
                        "azione": "Imposta status = 'pending_review'"
                    },
                    "transizione_a_active": {
                        "condizione": "seller pubblica E moderazione_abilitata == false",
                        "azione": "Imposta status = 'active', published_at = now()"
                    }
                },
                "pending_review": {
                    "prossimi_stati_possibili": ["active", "draft", "deleted"],
                    "transizione_a_active": {
                        "condizione": "moderator approva",
                        "azione": "Imposta status = 'active', published_at = now()"
                    },
                    "transizione_a_draft": {
                        "condizione": "moderator richiede modifiche",
                        "azione": "Imposta status = 'draft', notifica seller"
                    }
                },
                "active": {
                    "prossimi_stati_possibili": ["reserved", "sold", "expired", "deleted", "draft"],
                    "transizione_a_reserved": {
                        "condizione": "buyer riserva l'articolo",
                        "azione": "Imposta status = 'reserved'"
                    },
                    "transizione_a_sold": {
                        "condizione": "transazione completata",
                        "azione": "Imposta status = 'sold'"
                    },
                    "transizione_a_expired": {
                        "condizione": "expires_at < now()",
                        "azione": "Imposta status = 'expired'"
                    }
                },
                "reserved": {
                    "prossimi_stati_possibili": ["active", "sold", "deleted"],
                    "transizione_a_active": {
                        "condizione": "reservation_timeout scaduto OPPURE buyer annulla",
                        "azione": "Imposta status = 'active'"
                    },
                    "transizione_a_sold": {
                        "condizione": "transazione completata",
                        "azione": "Imposta status = 'sold'"
                    }
                },
                "sold": {
                    "prossimi_stati_possibili": ["deleted"],
                    "note": "Articolo venduto, non modificabile"
                },
                "expired": {
                    "prossimi_stati_possibili": ["active", "deleted"],
                    "transizione_a_active": {
                        "condizione": "seller rinnova",
                        "azione": "Imposta status = 'active', expires_at = now() + 30 giorni"
                    }
                },
                "deleted": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale"
                }
            },
            
            "indici": [
                {"campi": ["seller_id", "status"], "tipo": "composite"},
                {"campi": ["category_id", "status", "created_at"], "tipo": "composite"},
                {"campi": ["status", "published_at"], "tipo": "composite"},
                {"campi": ["location.city", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: CATEGORY
        # ───────────────────────────────────────────────────────────────────────
        "CATEGORY": {
            "descrizione": "Categoria di annunci",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "parent_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "riferimento": "CATEGORY.id",
                    "descrizione": "ID categoria genitore (null se root)"
                },
                "name": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Nome della categoria"
                },
                "slug": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": True,
                    "format": "slug",
                    "descrizione": "Slug per URL"
                },
                "icon": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Nome icona o URL"
                },
                "attribute_schema": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Schema attributi specifici per questa categoria"
                },
                "listing_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di annunci attivi"
                },
                "order": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": 0,
                    "descrizione": "Ordine di visualizzazione"
                },
                "is_active": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": True,
                    "descrizione": "Se la categoria è attiva"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["LISTING", "CATEGORY"],
                "appartiene_a": ["CATEGORY"]
            },
            
            "indici": [
                {"campi": ["slug"], "tipo": "unique"},
                {"campi": ["parent_id", "order"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: OFFER
        # ───────────────────────────────────────────────────────────────────────
        "OFFER": {
            "descrizione": "Offerta di acquisto per un annuncio",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "listing_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "LISTING.id",
                    "descrizione": "ID dell'annuncio"
                },
                "buyer_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'acquirente"
                },
                "seller_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del venditore (denormalizzato)"
                },
                "amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "min_value": 1,
                    "descrizione": "Importo offerto in centesimi"
                },
                "currency": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Valuta (copiata dal listing)"
                },
                "message": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 1000,
                    "descrizione": "Messaggio opzionale all'acquirente"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending",
                    "enum_values": ["pending", "accepted", "rejected", "countered", "withdrawn", "expired"],
                    "descrizione": "Stato dell'offerta"
                },
                "counter_amount": {
                    "tipo": "integer",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Importo della contro-offerta"
                },
                "expires_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Scadenza offerta (default: 48h)"
                },
                "responded_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data risposta del venditore"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                }
            },
            
            "relazioni": {
                "ha_uno": ["TRANSACTION"],
                "ha_molti": [],
                "appartiene_a": ["LISTING", "USER", "USER"]
            },
            
            "stati": ["pending", "accepted", "rejected", "countered", "withdrawn", "expired"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["accepted", "rejected", "countered", "withdrawn", "expired"],
                    "transizione_a_accepted": {
                        "condizione": "seller accetta",
                        "azione": "Imposta status = 'accepted', responded_at = now(), crea TRANSACTION"
                    },
                    "transizione_a_rejected": {
                        "condizione": "seller rifiuta",
                        "azione": "Imposta status = 'rejected', responded_at = now()"
                    },
                    "transizione_a_countered": {
                        "condizione": "seller fa contro-offerta",
                        "azione": "Imposta status = 'countered', counter_amount = X, responded_at = now()"
                    },
                    "transizione_a_withdrawn": {
                        "condizione": "buyer ritira offerta",
                        "azione": "Imposta status = 'withdrawn'"
                    },
                    "transizione_a_expired": {
                        "condizione": "expires_at < now()",
                        "azione": "Imposta status = 'expired'"
                    }
                },
                "countered": {
                    "prossimi_stati_possibili": ["accepted", "rejected", "withdrawn", "expired"],
                    "transizione_a_accepted": {
                        "condizione": "buyer accetta contro-offerta",
                        "azione": "Imposta status = 'accepted', crea TRANSACTION con counter_amount"
                    }
                }
            },
            
            "indici": [
                {"campi": ["listing_id", "status"], "tipo": "composite"},
                {"campi": ["buyer_id", "status"], "tipo": "composite"},
                {"campi": ["seller_id", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: TRANSACTION
        # ───────────────────────────────────────────────────────────────────────
        "TRANSACTION": {
            "descrizione": "Transazione di acquisto/vendita",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "listing_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "LISTING.id",
                    "descrizione": "ID dell'annuncio"
                },
                "offer_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "OFFER.id",
                    "descrizione": "ID dell'offerta (se da offerta)"
                },
                "buyer_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID acquirente"
                },
                "seller_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID venditore"
                },
                "amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Importo transazione in centesimi"
                },
                "currency": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Valuta"
                },
                "platform_fee": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Commissione piattaforma in centesimi"
                },
                "seller_payout": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Importo netto al venditore"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending_payment",
                    "enum_values": [
                        "pending_payment",
                        "payment_processing",
                        "paid",
                        "shipped",
                        "delivered",
                        "completed",
                        "disputed",
                        "refunded",
                        "cancelled"
                    ],
                    "descrizione": "Stato della transazione"
                },
                "payment_method": {
                    "tipo": "enum",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "enum_values": ["card", "paypal", "bank_transfer", "cash", "crypto"],
                    "descrizione": "Metodo di pagamento"
                },
                "payment_id": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "visible": "system_only",
                    "descrizione": "ID pagamento esterno (Stripe, PayPal, etc.)"
                },
                "shipping_address": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Indirizzo di spedizione"
                },
                "tracking_number": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Numero tracking spedizione"
                },
                "notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "descrizione": "Note sulla transazione"
                },
                "paid_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data pagamento"
                },
                "shipped_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data spedizione"
                },
                "delivered_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data consegna"
                },
                "completed_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data completamento"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["REVIEW"],
                "appartiene_a": ["LISTING", "OFFER", "USER", "USER"]
            },
            
            "stati": ["pending_payment", "payment_processing", "paid", "shipped", "delivered", "completed", "disputed", "refunded", "cancelled"],
            
            "transizioni_stato": {
                "pending_payment": {
                    "prossimi_stati_possibili": ["payment_processing", "cancelled"],
                    "transizione_a_payment_processing": {
                        "condizione": "buyer avvia pagamento",
                        "azione": "Imposta status = 'payment_processing'"
                    },
                    "transizione_a_cancelled": {
                        "condizione": "timeout 24h OPPURE buyer annulla",
                        "azione": "Imposta status = 'cancelled'"
                    }
                },
                "payment_processing": {
                    "prossimi_stati_possibili": ["paid", "pending_payment", "cancelled"],
                    "transizione_a_paid": {
                        "condizione": "pagamento confermato",
                        "azione": "Imposta status = 'paid', paid_at = now()"
                    },
                    "transizione_a_pending_payment": {
                        "condizione": "pagamento fallito",
                        "azione": "Imposta status = 'pending_payment', notifica buyer"
                    }
                },
                "paid": {
                    "prossimi_stati_possibili": ["shipped", "refunded", "disputed"],
                    "transizione_a_shipped": {
                        "condizione": "seller conferma spedizione",
                        "azione": "Imposta status = 'shipped', shipped_at = now()"
                    }
                },
                "shipped": {
                    "prossimi_stati_possibili": ["delivered", "disputed"],
                    "transizione_a_delivered": {
                        "condizione": "buyer conferma ricezione OPPURE tracking mostra consegna",
                        "azione": "Imposta status = 'delivered', delivered_at = now()"
                    }
                },
                "delivered": {
                    "prossimi_stati_possibili": ["completed", "disputed"],
                    "transizione_a_completed": {
                        "condizione": "buyer conferma soddisfazione OPPURE timeout 7 giorni senza dispute",
                        "azione": "Imposta status = 'completed', completed_at = now(), rilascia fondi a seller"
                    }
                },
                "completed": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale positivo"
                },
                "disputed": {
                    "prossimi_stati_possibili": ["completed", "refunded"],
                    "note": "Richiede intervento supporto"
                },
                "refunded": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale - rimborso effettuato"
                },
                "cancelled": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale - transazione annullata"
                }
            },
            
            "indici": [
                {"campi": ["buyer_id", "status"], "tipo": "composite"},
                {"campi": ["seller_id", "status"], "tipo": "composite"},
                {"campi": ["listing_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "REVIEW": {
            "descrizione": "Recensione post-transazione",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "transaction_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "TRANSACTION.id",
                    "descrizione": "ID della transazione"
                },
                "reviewer_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID di chi scrive la recensione"
                },
                "reviewee_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID di chi riceve la recensione"
                },
                "review_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["buyer_to_seller", "seller_to_buyer"],
                    "descrizione": "Direzione della recensione"
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_value": 1,
                    "max_value": 5,
                    "descrizione": "Valutazione da 1 a 5 stelle"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Titolo recensione"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 2000,
                    "descrizione": "Testo della recensione"
                },
                "aspects": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Valutazioni per aspetti specifici",
                    "schema": {
                        "communication": "1-5",
                        "item_as_described": "1-5 (solo buyer_to_seller)",
                        "shipping_time": "1-5 (solo buyer_to_seller)",
                        "payment_speed": "1-5 (solo seller_to_buyer)"
                    }
                },
                "response": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "descrizione": "Risposta del reviewee"
                },
                "response_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data risposta"
                },
                "is_public": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": True,
                    "descrizione": "Se la recensione è pubblica"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["TRANSACTION", "USER", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["transaction_id", "reviewer_id"],
                    "descrizione": "Un utente può lasciare una sola recensione per transazione"
                }
            ],
            
            "indici": [
                {"campi": ["reviewee_id", "created_at"], "tipo": "composite"},
                {"campi": ["transaction_id", "reviewer_id"], "tipo": "unique"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FAVORITE
        # ───────────────────────────────────────────────────────────────────────
        "FAVORITE": {
            "descrizione": "Annuncio salvato nei preferiti",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID utente"
                },
                "listing_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "LISTING.id",
                    "descrizione": "ID annuncio"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data aggiunta"
                }
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["user_id", "listing_id"],
                    "descrizione": "Un utente può salvare un annuncio una sola volta"
                }
            ],
            
            "indici": [
                {"campi": ["user_id", "listing_id"], "tipo": "unique"},
                {"campi": ["user_id", "created_at"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: SELLER_PROFILE
        # ───────────────────────────────────────────────────────────────────────
        "SELLER_PROFILE": {
            "descrizione": "Profilo venditore con statistiche e reputazione",
            
            "attributi": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID utente"
                },
                "display_name": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Nome commerciale/negozio"
                },
                "bio": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "descrizione": "Descrizione attività"
                },
                "is_verified": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": False,
                    "descrizione": "Se il venditore è verificato"
                },
                "is_professional": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se è un venditore professionale/azienda"
                },
                "rating_average": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Media recensioni (1-5)"
                },
                "rating_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di recensioni ricevute"
                },
                "total_sales": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero totale di vendite"
                },
                "active_listings_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Annunci attualmente attivi"
                },
                "response_rate": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Percentuale di risposte alle richieste"
                },
                "response_time_hours": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Tempo medio di risposta in ore"
                },
                "member_since": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data iscrizione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento stats"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["user_id"], "tipo": "unique"},
                {"campi": ["rating_average", "total_sales"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },


# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: SOCIAL
# VERSIONE: 1.0
# DESCRIZIONE: Connessioni tra utenti, follow, feed, blocchi
# DIPENDENZE: IDENTITY, CONTENT
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_SOCIAL = {
    "nome": "SOCIAL",
    "versione": "1.0",
    "descrizione": "Connessioni tra utenti, follow, feed, blocchi",
    "dipendenze": ["IDENTITY", "CONTENT"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FOLLOW
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW": {
            "descrizione": "Relazione di following tra utenti (unidirezionale)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "follower_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che segue"
                },
                "followed_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente seguito"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["pending", "active", "rejected"],
                    "descrizione": "Stato del follow (pending solo se account privato)"
                },
                "notify_posts": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se ricevere notifiche per ogni nuovo post"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora del follow"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["follower_id", "followed_id"],
                    "descrizione": "Un utente può seguire un altro utente una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "follower_id != followed_id",
                    "descrizione": "Un utente non può seguire se stesso"
                }
            ],
            
            "stati": ["pending", "active", "rejected"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["active", "rejected"],
                    "transizione_a_active": {
                        "condizione": "followed_user accetta la richiesta",
                        "azione": "Imposta status = 'active'"
                    },
                    "transizione_a_rejected": {
                        "condizione": "followed_user rifiuta la richiesta",
                        "azione": "Imposta status = 'rejected'"
                    }
                },
                "active": {
                    "prossimi_stati_possibili": [],
                    "note": "Per smettere di seguire si elimina il record"
                },
                "rejected": {
                    "prossimi_stati_possibili": [],
                    "note": "Può essere eliminato per permettere nuova richiesta"
                }
            },
            
            "indici": [
                {"campi": ["follower_id", "followed_id"], "tipo": "unique"},
                {"campi": ["followed_id", "status"], "tipo": "composite"},
                {"campi": ["follower_id", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: BLOCK
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK": {
            "descrizione": "Blocco di un utente da parte di un altro",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "blocker_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che blocca"
                },
                "blocked_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente bloccato"
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 500,
                    "visible": "blocker_only",
                    "descrizione": "Motivo del blocco (privato)"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora del blocco"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["blocker_id", "blocked_id"],
                    "descrizione": "Un utente può bloccare un altro una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "blocker_id != blocked_id",
                    "descrizione": "Un utente non può bloccare se stesso"
                }
            ],
            
            "effetti_blocco": {
                "descrizione": "Quando USER_A blocca USER_B:",
                "effetti": [
                    "USER_B non può vedere i post di USER_A",
                    "USER_B non può vedere il profilo di USER_A",
                    "USER_B non può inviare messaggi a USER_A",
                    "USER_B non può seguire USER_A",
                    "USER_B non può commentare i post di USER_A",
                    "USER_B non può reagire ai post di USER_A",
                    "USER_B non può menzionare USER_A",
                    "Eventuali FOLLOW esistenti tra i due vengono rimossi",
                    "Eventuali conversazioni esistenti vengono nascoste"
                ]
            },
            
            "indici": [
                {"campi": ["blocker_id", "blocked_id"], "tipo": "unique"},
                {"campi": ["blocked_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: USER_STATS
        # ───────────────────────────────────────────────────────────────────────
        "USER_STATS": {
            "descrizione": "Contatori e statistiche dell'utente (denormalizzati per performance)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente"
                },
                "followers_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di follower"
                },
                "following_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di utenti seguiti"
                },
                "posts_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di post pubblicati"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["user_id"], "tipo": "unique"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FEED_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "FEED_ITEM": {
            "descrizione": "Elemento nel feed di un utente (materializzato per performance)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente proprietario del feed"
                },
                "post_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "POST.id",
                    "descrizione": "ID del post nel feed"
                },
                "author_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'autore del post"
                },
                "feed_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "enum_values": ["following", "suggested", "sponsored"],
                    "descrizione": "Tipo di contenuto nel feed"
                },
                "relevance_score": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Score di rilevanza per ordinamento"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data di inserimento nel feed"
                },
                "expires_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data di scadenza (per pulizia periodica)"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "POST"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["user_id", "post_id"],
                    "descrizione": "Un post appare una sola volta nel feed di un utente"
                }
            ],
            
            "indici": [
                {"campi": ["user_id", "created_at"], "tipo": "composite"},
                {"campi": ["user_id", "relevance_score"], "tipo": "composite"},
                {"campi": ["expires_at"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO SOCIAL
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: FOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW_USER": {
            "descrizione": "Inizia a seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "validazione": "UUID dell'utente da seguire"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_FOLLOW_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id",
                    "errore_se_falsa": "ERROR_ALREADY_FOLLOWING"
                },
                {
                    "id": "PRE_5",
                    "condizione": "NON esiste BLOCK con blocker_id == input.user_id E blocked_id == current_user.id",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_6",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_USER_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera profilo dell'utente da seguire",
                    "dettaglio": """
                        target_user = SELECT * FROM USER WHERE id == input.user_id
                        target_profile = SELECT * FROM PROFILE WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Determina stato iniziale del follow",
                    "dettaglio": """
                        SE target_profile.privacy_settings.profile_visibility == 'private':
                            initial_status = 'pending'
                        ALTRIMENTI:
                            initial_status = 'active'
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record FOLLOW",
                    "dettaglio": """
                        follow_id = genera_uuid_v4()
                        
                        INSERT INTO FOLLOW:
                        - id = follow_id
                        - follower_id = current_user.id
                        - followed_id = input.user_id
                        - status = initial_status
                        - notify_posts = false
                        - created_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna contatori se follow è attivo",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Incrementa following_count del follower
                            UPDATE USER_STATS SET following_count = following_count + 1
                            WHERE user_id == current_user.id
                            
                            # Incrementa followers_count del seguito
                            UPDATE USER_STATS SET followers_count = followers_count + 1
                            WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Crea notifica",
                    "dettaglio": """
                        SE initial_status == 'active':
                            crea_notifica(
                                recipient_id = input.user_id,
                                type = 'new_follower',
                                source_user_id = current_user.id,
                                target_type = 'user',
                                target_id = current_user.id
                            )
                        ALTRIMENTI:
                            crea_notifica(
                                recipient_id = input.user_id,
                                type = 'follow_request',
                                source_user_id = current_user.id,
                                target_type = 'follow',
                                target_id = follow_id
                            )
                    """
                },
                {
                    "numero": 6,
                    "azione": "Popola feed se follow è attivo",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Aggiungi ultimi N post dell'utente seguito al feed del follower
                            recent_posts = SELECT id, author_id FROM POST 
                                WHERE author_id == input.user_id 
                                AND status == 'published'
                                AND visibility IN ['public', 'followers_only']
                                ORDER BY published_at DESC
                                LIMIT 10
                            
                            PER OGNI post IN recent_posts:
                                INSERT INTO FEED_ITEM (se non esiste già):
                                - user_id = current_user.id
                                - post_id = post.id
                                - author_id = post.author_id
                                - feed_type = 'following'
                                - relevance_score = calcola_score(post)
                                - created_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "follow": "oggetto FOLLOW creato",
                "status": "active | pending"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_FOLLOW_SELF": {
                    "codice": "SOCIAL_002",
                    "messaggio": "Non puoi seguire te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_003",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FOLLOWING": {
                    "codice": "SOCIAL_004",
                    "messaggio": "Stai già seguendo questo utente",
                    "http_status": 409
                },
                "ERROR_USER_BLOCKED": {
                    "codice": "SOCIAL_005",
                    "messaggio": "Non puoi seguire un utente che hai bloccato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNFOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNFOLLOW_USER": {
            "descrizione": "Smetti di seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "validazione": "UUID dell'utente da smettere di seguire"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id",
                    "errore_se_falsa": "ERROR_NOT_FOLLOWING"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera record FOLLOW",
                    "dettaglio": """
                        follow = SELECT * FROM FOLLOW 
                        WHERE follower_id == current_user.id 
                        AND followed_id == input.user_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatori se follow era attivo",
                    "dettaglio": """
                        SE follow.status == 'active':
                            # Decrementa following_count del follower
                            UPDATE USER_STATS SET following_count = following_count - 1
                            WHERE user_id == current_user.id
                            
                            # Decrementa followers_count del seguito
                            UPDATE USER_STATS SET followers_count = followers_count - 1
                            WHERE user_id == input.user_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Elimina record FOLLOW",
                    "dettaglio": """
                        DELETE FROM FOLLOW WHERE id == follow.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Rimuovi post dal feed",
                    "dettaglio": """
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == current_user.id 
                        AND author_id == input.user_id
                        AND feed_type == 'following'
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Non segui più questo utente"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_FOLLOWING": {
                    "codice": "SOCIAL_011",
                    "messaggio": "Non stai seguendo questo utente",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ACCEPT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "ACCEPT_FOLLOW_REQUEST": {
            "descrizione": "Accetta una richiesta di follow (per account privati)",
            "attore": "utente_autenticato",
            
            "input": {
                "follow_id": {
                    "tipo": "uuid",
                    "required": True,
                    "validazione": "UUID della richiesta di follow"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con id == input.follow_id E status == 'pending'",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "FOLLOW.followed_id == current_user.id",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera FOLLOW",
                    "dettaglio": "follow = SELECT * FROM FOLLOW WHERE id == input.follow_id"
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna status a active",
                    "dettaglio": """
                        UPDATE FOLLOW SET status = 'active' WHERE id == follow.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna contatori",
                    "dettaglio": """
                        UPDATE USER_STATS SET following_count = following_count + 1
                        WHERE user_id == follow.follower_id
                        
                        UPDATE USER_STATS SET followers_count = followers_count + 1
                        WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica il richiedente",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = follow.follower_id,
                            type = 'follow_accepted',
                            source_user_id = current_user.id,
                            target_type = 'user',
                            target_id = current_user.id
                        )
                    """
                },
                {
                    "numero": 5,
                    "azione": "Popola feed del nuovo follower",
                    "dettaglio": """
                        # (stessa logica di FOLLOW_USER passo 6)
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "FOLLOW.status == 'active'"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow accettata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_021",
                    "messaggio": "Richiesta non trovata",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "SOCIAL_022",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: REJECT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "REJECT_FOLLOW_REQUEST": {
            "descrizione": "Rifiuta una richiesta di follow",
            "attore": "utente_autenticato",
            
            "input": {
                "follow_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con id == input.follow_id E status == 'pending' E followed_id == current_user.id",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina o marca come rejected",
                    "dettaglio": """
                        DELETE FROM FOLLOW WHERE id == input.follow_id
                        # Oppure: UPDATE FOLLOW SET status = 'rejected' WHERE id == input.follow_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "FOLLOW non esiste più o ha status == 'rejected'"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow rifiutata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_031",
                    "messaggio": "Richiesta non trovata",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: BLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK_USER": {
            "descrizione": "Blocca un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 500 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_BLOCK_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.user_id",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_ALREADY_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Rimuovi eventuali follow reciproci",
                    "dettaglio": """
                        # Se io seguo lui
                        follow1 = SELECT * FROM FOLLOW WHERE follower_id == current_user.id AND followed_id == input.user_id
                        SE follow1 esiste:
                            esegui UNFOLLOW_USER(user_id = input.user_id)
                        
                        # Se lui segue me
                        follow2 = SELECT * FROM FOLLOW WHERE follower_id == input.user_id AND followed_id == current_user.id
                        SE follow2 esiste:
                            DELETE FROM FOLLOW WHERE id == follow2.id
                            UPDATE USER_STATS SET following_count = following_count - 1 WHERE user_id == input.user_id
                            UPDATE USER_STATS SET followers_count = followers_count - 1 WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea record BLOCK",
                    "dettaglio": """
                        INSERT INTO BLOCK:
                        - id = genera_uuid_v4()
                        - blocker_id = current_user.id
                        - blocked_id = input.user_id
                        - reason = input.reason
                        - created_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Rimuovi contenuti dal feed reciproco",
                    "dettaglio": """
                        # Rimuovi i suoi post dal mio feed
                        DELETE FROM FEED_ITEM WHERE user_id == current_user.id AND author_id == input.user_id
                        
                        # Rimuovi i miei post dal suo feed
                        DELETE FROM FEED_ITEM WHERE user_id == input.user_id AND author_id == current_user.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id"
                },
                {
                    "id": "POST_2",
                    "condizione": "NON esiste FOLLOW tra i due utenti in nessuna direzione"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente bloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_BLOCK_SELF": {
                    "codice": "SOCIAL_041",
                    "messaggio": "Non puoi bloccare te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_042",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_BLOCKED": {
                    "codice": "SOCIAL_043",
                    "messaggio": "Utente già bloccato",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNBLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNBLOCK_USER": {
            "descrizione": "Sblocca un utente precedentemente bloccato",
            "attore": "utente_autenticato",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id",
                    "errore_se_falsa": "ERROR_NOT_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record BLOCK",
                    "dettaglio": """
                        DELETE FROM BLOCK WHERE blocker_id == current_user.id AND blocked_id == input.user_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente sbloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_BLOCKED": {
                    "codice": "SOCIAL_051",
                    "messaggio": "Utente non bloccato",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_FEED
        # ───────────────────────────────────────────────────────────────────────
        "GET_FEED": {
            "descrizione": "Recupera il feed dell'utente corrente",
            "attore": "utente_autenticato",
            
            "input": {
                "cursor": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Cursore per paginazione (timestamp dell'ultimo item)"
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20,
                    "validazione": "min 1, max 50"
                },
                "feed_type": {
                    "tipo": "enum",
                    "required": False,
                    "default": "mixed",
                    "validazione": "following | suggested | mixed"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query base",
                    "dettaglio": """
                        query = SELECT fi.*, p.*, u.username, pr.display_name, pr.avatar_url
                        FROM FEED_ITEM fi
                        JOIN POST p ON fi.post_id == p.id
                        JOIN USER u ON fi.author_id == u.id
                        JOIN PROFILE pr ON u.id == pr.user_id
                        WHERE fi.user_id == current_user.id
                        AND p.status == 'published'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Applica filtro tipo feed",
                    "dettaglio": """
                        SE input.feed_type == 'following':
                            query = query AND fi.feed_type == 'following'
                        ALTRIMENTI SE input.feed_type == 'suggested':
                            query = query AND fi.feed_type == 'suggested'
                        # Se 'mixed', non aggiungere filtro
                    """
                },
                {
                    "numero": 3,
                    "azione": "Applica cursore per paginazione",
                    "dettaglio": """
                        SE input.cursor fornito:
                            cursor_timestamp = decodifica(input.cursor)
                            query = query AND fi.created_at < cursor_timestamp
                    """
                },
                {
                    "numero": 4,
                    "azione": "Escludi utenti bloccati",
                    "dettaglio": """
                        blocked_ids = SELECT blocked_id FROM BLOCK WHERE blocker_id == current_user.id
                        query = query AND fi.author_id NOT IN blocked_ids
                    """
                },
                {
                    "numero": 5,
                    "azione": "Ordina e limita",
                    "dettaglio": """
                        query = query ORDER BY fi.relevance_score DESC, fi.created_at DESC
                        query = query LIMIT (input.limit + 1)  # +1 per sapere se ci sono altre pagine
                    """
                },
                {
                    "numero": 6,
                    "azione": "Esegui query e prepara risposta",
                    "dettaglio": """
                        results = esegui_query(query)
                        
                        SE results.length > input.limit:
                            has_more = true
                            results = results[0:input.limit]  # Rimuovi l'elemento extra
                            next_cursor = codifica(results[-1].created_at)
                        ALTRIMENTI:
                            has_more = false
                            next_cursor = null
                    """
                },
                {
                    "numero": 7,
                    "azione": "Aggiungi informazioni utente corrente per ogni post",
                    "dettaglio": """
                        PER OGNI item IN results:
                            item.user_reaction = SELECT reaction_type FROM REACTION 
                                WHERE user_id == current_user.id 
                                AND target_type == 'post' 
                                AND target_id == item.post_id
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "items": "array di post con info autore",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_USER_FOLLOWERS
        # ───────────────────────────────────────────────────────────────────────
        "GET_USER_FOLLOWERS": {
            "descrizione": "Recupera la lista dei follower di un utente",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Profilo non è privato OPPURE richiedente è il proprietario OPPURE richiedente è follower",
                    "errore_se_falsa": "ERROR_PRIVATE_PROFILE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Query follower",
                    "dettaglio": """
                        SELECT u.id, u.username, pr.display_name, pr.avatar_url, f.created_at
                        FROM FOLLOW f
                        JOIN USER u ON f.follower_id == u.id
                        JOIN PROFILE pr ON u.id == pr.user_id
                        WHERE f.followed_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiungi stato follow per utente corrente",
                    "dettaglio": """
                        SE utente_autenticato:
                            PER OGNI follower IN results:
                                follower.is_following = EXISTS FOLLOW 
                                    WHERE follower_id == current_user.id 
                                    AND followed_id == follower.id
                                    AND status == 'active'
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "followers": "array di utenti",
                "total_count": "numero totale di follower",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_070",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_PRIVATE_PROFILE": {
                    "codice": "SOCIAL_071",
                    "messaggio": "Profilo privato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_USER_FOLLOWING
        # ───────────────────────────────────────────────────────────────────────
        "GET_USER_FOLLOWING": {
            "descrizione": "Recupera la lista degli utenti seguiti da un utente",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Query following",
                    "dettaglio": """
                        SELECT u.id, u.username, pr.display_name, pr.avatar_url, f.created_at
                        FROM FOLLOW f
                        JOIN USER u ON f.followed_id == u.id
                        JOIN PROFILE pr ON u.id == pr.user_id
                        WHERE f.follower_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "following": "array di utenti",
                "total_count": "numero totale di following",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_080",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo SOCIAL"
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: SOCIAL
# VERSIONE: 1.0
# DESCRIZIONE: Connessioni tra utenti, follow, feed, suggerimenti
# DIPENDENZE: IDENTITY (richiede USER), CONTENT (richiede POST)
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_SOCIAL = {
    "nome": "SOCIAL",
    "versione": "1.0",
    "descrizione": "Connessioni tra utenti, follow, feed, suggerimenti",
    "dipendenze": ["IDENTITY", "CONTENT"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FOLLOW
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW": {
            "descrizione": "Relazione di follow unidirezionale tra utenti",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco del follow"
                },
                "follower_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che segue"
                },
                "followed_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente seguito"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["pending", "active", "blocked"],
                    "descrizione": "Stato del follow (pending se account privato)"
                },
                "notify_posts": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se ricevere notifiche per ogni nuovo post"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora del follow"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["follower_id", "followed_id"],
                    "descrizione": "Un utente può seguire un altro utente una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "follower_id != followed_id",
                    "descrizione": "Un utente non può seguire se stesso"
                }
            ],
            
            "stati": ["pending", "active", "blocked"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["active", "blocked"],
                    "transizione_a_active": {
                        "condizione": "followed_user accetta richiesta",
                        "azione": "Imposta status = 'active'"
                    },
                    "transizione_a_blocked": {
                        "condizione": "followed_user rifiuta richiesta",
                        "azione": "Elimina record OPPURE imposta status = 'blocked'"
                    }
                },
                "active": {
                    "prossimi_stati_possibili": ["blocked"],
                    "transizione_a_blocked": {
                        "condizione": "followed_user blocca follower",
                        "azione": "Imposta status = 'blocked'"
                    }
                },
                "blocked": {
                    "prossimi_stati_possibili": [],
                    "note": "Per sbloccare, il record viene eliminato e deve essere ricreato"
                }
            },
            
            "indici": [
                {"campi": ["follower_id", "followed_id"], "tipo": "unique"},
                {"campi": ["followed_id", "status"], "tipo": "composite"},
                {"campi": ["follower_id", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: BLOCK
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK": {
            "descrizione": "Blocco di un utente da parte di un altro",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "blocker_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente che blocca"
                },
                "blocked_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente bloccato"
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 500,
                    "visible": "blocker_only",
                    "descrizione": "Motivo del blocco (opzionale, privato)"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data e ora del blocco"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["blocker_id", "blocked_id"],
                    "descrizione": "Un utente può bloccare un altro una sola volta"
                },
                {
                    "tipo": "check",
                    "condizione": "blocker_id != blocked_id",
                    "descrizione": "Un utente non può bloccare se stesso"
                }
            ],
            
            "indici": [
                {"campi": ["blocker_id", "blocked_id"], "tipo": "unique"},
                {"campi": ["blocked_id"], "tipo": "standard"}
            ],
            
            "effetti_blocco": {
                "descrizione": "Cosa succede quando A blocca B",
                "effetti": [
                    "B non può vedere i post di A",
                    "B non può vedere il profilo di A",
                    "B non può inviare messaggi ad A",
                    "B non può seguire A",
                    "B non può menzionare A",
                    "B non può commentare i post di A",
                    "B non può reagire ai post di A",
                    "Eventuali follow esistenti tra A e B vengono rimossi",
                    "Eventuali conversazioni tra A e B vengono nascoste ad entrambi"
                ]
            },
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: USER_STATS
        # ───────────────────────────────────────────────────────────────────────
        "USER_STATS": {
            "descrizione": "Statistiche aggregate dell'utente (contatori)",
            
            "attributi": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente"
                },
                "followers_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di follower"
                },
                "following_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di utenti seguiti"
                },
                "posts_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "min_value": 0,
                    "descrizione": "Numero di post pubblicati"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Ultimo aggiornamento delle statistiche"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["user_id"], "tipo": "unique"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: FEED_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "FEED_ITEM": {
            "descrizione": "Elemento nel feed di un utente (materializzato o calcolato)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID dell'utente proprietario del feed"
                },
                "post_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "POST.id",
                    "descrizione": "ID del post nel feed"
                },
                "source_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["following", "suggested", "promoted", "shared_by_friend"],
                    "descrizione": "Perché questo post è nel feed"
                },
                "source_user_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID utente che ha causato l'inserimento (es: chi ha condiviso)"
                },
                "relevance_score": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Score di rilevanza per ordinamento"
                },
                "is_seen": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se l'utente ha visto questo item"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Quando è stato aggiunto al feed"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["USER", "POST"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["user_id", "post_id"],
                    "descrizione": "Un post può apparire una sola volta nel feed di un utente"
                }
            ],
            
            "indici": [
                {"campi": ["user_id", "created_at"], "tipo": "composite"},
                {"campi": ["user_id", "relevance_score"], "tipo": "composite"},
                {"campi": ["post_id"], "tipo": "standard"}
            ],
            
            "note": "Questa entità può essere materializzata (fan-out on write) o calcolata (fan-out on read) in base alla strategia scelta",
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO SOCIAL
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: FOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "FOLLOW_USER": {
            "descrizione": "Inizia a seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "target_user_id": {
                    "tipo": "uuid",
                    "required": True,
                    "validazione": "UUID valido"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.target_user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_FOLLOW_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.target_user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.target_user_id",
                    "errore_se_falsa": "ERROR_ALREADY_FOLLOWING"
                },
                {
                    "id": "PRE_5",
                    "condizione": "NON esiste BLOCK con blocker_id == input.target_user_id E blocked_id == current_user.id",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica se target ha profilo privato",
                    "dettaglio": """
                        target_profile = SELECT privacy_settings FROM PROFILE WHERE user_id == input.target_user_id
                        is_private = target_profile.privacy_settings.profile_visibility == 'private'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Determina stato iniziale del follow",
                    "dettaglio": """
                        SE is_private == true:
                            initial_status = 'pending'
                        ALTRIMENTI:
                            initial_status = 'active'
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record FOLLOW",
                    "dettaglio": """
                        follow_id = genera_uuid_v4()
                        
                        INSERT INTO FOLLOW:
                        - id = follow_id
                        - follower_id = current_user.id
                        - followed_id = input.target_user_id
                        - status = initial_status
                        - notify_posts = false
                        - created_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna contatori se follow è attivo",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Incrementa following_count del follower
                            UPDATE USER_STATS SET 
                            - following_count = following_count + 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                            
                            # Incrementa followers_count del followed
                            UPDATE USER_STATS SET 
                            - followers_count = followers_count + 1
                            - updated_at = now()
                            WHERE user_id == input.target_user_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Crea notifica",
                    "dettaglio": """
                        SE initial_status == 'active':
                            notification_type = 'new_follower'
                        ALTRIMENTI:
                            notification_type = 'follow_request'
                        
                        crea_notifica(
                            recipient_id = input.target_user_id,
                            type = notification_type,
                            source_user_id = current_user.id,
                            target_type = 'user',
                            target_id = current_user.id
                        )
                    """
                },
                {
                    "numero": 6,
                    "azione": "Popola feed se follow è attivo (opzionale)",
                    "dettaglio": """
                        SE initial_status == 'active':
                            # Opzionale: aggiungi ultimi N post del followed al feed del follower
                            recent_posts = SELECT id FROM POST 
                                WHERE author_id == input.target_user_id 
                                AND status == 'published'
                                AND visibility IN ('public', 'followers_only')
                                ORDER BY published_at DESC
                                LIMIT 10
                            
                            PER OGNI post IN recent_posts:
                                INSERT INTO FEED_ITEM (se non esiste già):
                                - id = genera_uuid_v4()
                                - user_id = current_user.id
                                - post_id = post.id
                                - source_type = 'following'
                                - relevance_score = calcola_relevance(post)
                                - created_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.target_user_id"
                }
            ],
            
            "output": {
                "success": True,
                "follow": "oggetto FOLLOW creato",
                "status": "active | pending"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_FOLLOW_SELF": {
                    "codice": "SOCIAL_002",
                    "messaggio": "Non puoi seguire te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_003",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FOLLOWING": {
                    "codice": "SOCIAL_004",
                    "messaggio": "Stai già seguendo questo utente",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNFOLLOW_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNFOLLOW_USER": {
            "descrizione": "Smetti di seguire un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "target_user_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == current_user.id E followed_id == input.target_user_id",
                    "errore_se_falsa": "ERROR_NOT_FOLLOWING"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera record FOLLOW",
                    "dettaglio": """
                        follow = SELECT * FROM FOLLOW 
                        WHERE follower_id == current_user.id 
                        AND followed_id == input.target_user_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatori se follow era attivo",
                    "dettaglio": """
                        SE follow.status == 'active':
                            # Decrementa following_count del follower
                            UPDATE USER_STATS SET 
                            - following_count = following_count - 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                            
                            # Decrementa followers_count del followed
                            UPDATE USER_STATS SET 
                            - followers_count = followers_count - 1
                            - updated_at = now()
                            WHERE user_id == input.target_user_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Elimina record FOLLOW",
                    "dettaglio": """
                        DELETE FROM FOLLOW WHERE id == follow.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Rimuovi post dal feed (opzionale)",
                    "dettaglio": """
                        # Opzionale: rimuovi i post del unfollowed dal feed
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == current_user.id 
                        AND post_id IN (SELECT id FROM POST WHERE author_id == input.target_user_id)
                        AND source_type == 'following'
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FOLLOW con follower_id == current_user.id E followed_id == input.target_user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Non segui più questo utente"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_FOLLOWING": {
                    "codice": "SOCIAL_011",
                    "messaggio": "Non stai seguendo questo utente",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ACCEPT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "ACCEPT_FOLLOW_REQUEST": {
            "descrizione": "Accetta una richiesta di follow (per account privati)",
            "attore": "utente_autenticato",
            
            "input": {
                "follower_id": {
                    "tipo": "uuid",
                    "required": True,
                    "descrizione": "ID dell'utente che ha richiesto di seguire"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id E status == 'pending'",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Aggiorna stato FOLLOW",
                    "dettaglio": """
                        UPDATE FOLLOW SET
                        - status = 'active'
                        WHERE follower_id == input.follower_id 
                        AND followed_id == current_user.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatori",
                    "dettaglio": """
                        # Incrementa following_count del follower
                        UPDATE USER_STATS SET following_count = following_count + 1
                        WHERE user_id == input.follower_id
                        
                        # Incrementa followers_count del current_user
                        UPDATE USER_STATS SET followers_count = followers_count + 1
                        WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Notifica il follower",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = input.follower_id,
                            type = 'follow_request_accepted',
                            source_user_id = current_user.id,
                            target_type = 'user',
                            target_id = current_user.id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "FOLLOW.status == 'active'"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow accettata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_021",
                    "messaggio": "Richiesta di follow non trovata",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: REJECT_FOLLOW_REQUEST
        # ───────────────────────────────────────────────────────────────────────
        "REJECT_FOLLOW_REQUEST": {
            "descrizione": "Rifiuta una richiesta di follow",
            "attore": "utente_autenticato",
            
            "input": {
                "follower_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id E status == 'pending'",
                    "errore_se_falsa": "ERROR_REQUEST_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record FOLLOW",
                    "dettaglio": """
                        DELETE FROM FOLLOW
                        WHERE follower_id == input.follower_id 
                        AND followed_id == current_user.id
                        AND status == 'pending'
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FOLLOW con follower_id == input.follower_id E followed_id == current_user.id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Richiesta di follow rifiutata"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_025",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_REQUEST_NOT_FOUND": {
                    "codice": "SOCIAL_026",
                    "messaggio": "Richiesta di follow non trovata",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: BLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "BLOCK_USER": {
            "descrizione": "Blocca un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "target_user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 500 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.target_user_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_BLOCK_SELF"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste USER con id == input.target_user_id",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.target_user_id",
                    "errore_se_falsa": "ERROR_ALREADY_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Crea record BLOCK",
                    "dettaglio": """
                        INSERT INTO BLOCK:
                        - id = genera_uuid_v4()
                        - blocker_id = current_user.id
                        - blocked_id = input.target_user_id
                        - reason = input.reason
                        - created_at = now()
                    """
                },
                {
                    "numero": 2,
                    "azione": "Rimuovi follow esistenti in entrambe le direzioni",
                    "dettaglio": """
                        # Se current_user seguiva target
                        follow_1 = SELECT * FROM FOLLOW 
                            WHERE follower_id == current_user.id 
                            AND followed_id == input.target_user_id
                        
                        SE follow_1 esiste E follow_1.status == 'active':
                            UPDATE USER_STATS SET following_count = following_count - 1
                            WHERE user_id == current_user.id
                            
                            UPDATE USER_STATS SET followers_count = followers_count - 1
                            WHERE user_id == input.target_user_id
                        
                        DELETE FROM FOLLOW 
                        WHERE follower_id == current_user.id 
                        AND followed_id == input.target_user_id
                        
                        # Se target seguiva current_user
                        follow_2 = SELECT * FROM FOLLOW 
                            WHERE follower_id == input.target_user_id 
                            AND followed_id == current_user.id
                        
                        SE follow_2 esiste E follow_2.status == 'active':
                            UPDATE USER_STATS SET following_count = following_count - 1
                            WHERE user_id == input.target_user_id
                            
                            UPDATE USER_STATS SET followers_count = followers_count - 1
                            WHERE user_id == current_user.id
                        
                        DELETE FROM FOLLOW 
                        WHERE follower_id == input.target_user_id 
                        AND followed_id == current_user.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Rimuovi post dal feed reciprocamente",
                    "dettaglio": """
                        # Rimuovi post del bloccato dal feed del bloccante
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == current_user.id 
                        AND post_id IN (SELECT id FROM POST WHERE author_id == input.target_user_id)
                        
                        # Rimuovi post del bloccante dal feed del bloccato
                        DELETE FROM FEED_ITEM 
                        WHERE user_id == input.target_user_id 
                        AND post_id IN (SELECT id FROM POST WHERE author_id == current_user.id)
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.target_user_id"
                },
                {
                    "id": "POST_2",
                    "condizione": "NON esistono FOLLOW tra i due utenti"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente bloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CANNOT_BLOCK_SELF": {
                    "codice": "SOCIAL_031",
                    "messaggio": "Non puoi bloccare te stesso",
                    "http_status": 400
                },
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_032",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_BLOCKED": {
                    "codice": "SOCIAL_033",
                    "messaggio": "Utente già bloccato",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UNBLOCK_USER
        # ───────────────────────────────────────────────────────────────────────
        "UNBLOCK_USER": {
            "descrizione": "Sblocca un utente",
            "attore": "utente_autenticato",
            
            "input": {
                "target_user_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BLOCK con blocker_id == current_user.id E blocked_id == input.target_user_id",
                    "errore_se_falsa": "ERROR_NOT_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record BLOCK",
                    "dettaglio": """
                        DELETE FROM BLOCK
                        WHERE blocker_id == current_user.id 
                        AND blocked_id == input.target_user_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste BLOCK con blocker_id == current_user.id E blocked_id == input.target_user_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Utente sbloccato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_BLOCKED": {
                    "codice": "SOCIAL_041",
                    "messaggio": "Utente non bloccato",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_FEED
        # ───────────────────────────────────────────────────────────────────────
        "GET_FEED": {
            "descrizione": "Recupera il feed personalizzato dell'utente",
            "attore": "utente_autenticato",
            
            "input": {
                "cursor": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Cursore per paginazione (timestamp o score)"
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20,
                    "validazione": "1-50"
                },
                "feed_type": {
                    "tipo": "enum",
                    "required": False,
                    "default": "for_you",
                    "validazione": "for_you | following | chronological"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera lista utenti bloccati (da escludere)",
                    "dettaglio": """
                        blocked_users = SELECT blocked_id FROM BLOCK WHERE blocker_id == current_user.id
                        users_who_blocked_me = SELECT blocker_id FROM BLOCK WHERE blocked_id == current_user.id
                        excluded_users = blocked_users + users_who_blocked_me
                    """
                },
                {
                    "numero": 2,
                    "azione": "Recupera lista utenti seguiti",
                    "dettaglio": """
                        following_ids = SELECT followed_id FROM FOLLOW 
                        WHERE follower_id == current_user.id 
                        AND status == 'active'
                    """
                },
                {
                    "numero": 3,
                    "azione": "Costruisci query in base a feed_type",
                    "dettaglio": """
                        SE input.feed_type == 'following':
                            # Solo post degli utenti seguiti
                            posts = SELECT p.*, u.username, pr.display_name, pr.avatar_url
                            FROM POST p
                            JOIN USER u ON p.author_id = u.id
                            JOIN PROFILE pr ON p.author_id = pr.user_id
                            WHERE p.author_id IN following_ids
                            AND p.author_id NOT IN excluded_users
                            AND p.status == 'published'
                            AND (p.visibility == 'public' OR p.visibility == 'followers_only')
                            ORDER BY p.published_at DESC
                            LIMIT input.limit
                        
                        ALTRIMENTI SE input.feed_type == 'chronological':
                            # Post degli utenti seguiti in ordine cronologico
                            posts = (stessa query di 'following')
                        
                        ALTRIMENTI: # for_you (default)
                            # Mix di post seguiti + suggeriti + trending
                            posts = SELECT p.*, u.username, pr.display_name, pr.avatar_url,
                                    calcola_relevance_score(p, current_user) as score
                            FROM POST p
                            JOIN USER u ON p.author_id = u.id
                            JOIN PROFILE pr ON p.author_id = pr.user_id
                            WHERE p.author_id NOT IN excluded_users
                            AND p.status == 'published'
                            AND p.visibility == 'public'
                            AND (
                                p.author_id IN following_ids  # Post di chi seguo
                                OR p.stats.reactions_count > soglia_trending  # Post trending
                                OR p.id IN (SELECT post_id FROM suggerimenti WHERE user_id == current_user.id)
                            )
                            ORDER BY score DESC, p.published_at DESC
                            LIMIT input.limit
                    """
                },
                {
                    "numero": 4,
                    "azione": "Applica cursore per paginazione",
                    "dettaglio": """
                        SE input.cursor fornito:
                            Aggiungi condizione WHERE appropriata per paginazione
                    """
                },
                {
                    "numero": 5,
                    "azione": "Recupera reazioni dell'utente corrente per ogni post",
                    "dettaglio": """
                        PER OGNI post IN posts:
                            post.user_reaction = SELECT reaction_type FROM REACTION
                            WHERE user_id == current_user.id
                            AND target_type == 'post'
                            AND target_id == post.id
                    """
                },
                {
                    "numero": 6,
                    "azione": "Calcola next_cursor",
                    "dettaglio": """
                        SE posts non è vuoto:
                            last_post = posts[ultimo]
                            SE feed_type == 'for_you':
                                next_cursor = "{last_post.score}_{last_post.id}"
                            ALTRIMENTI:
                                next_cursor = last_post.published_at
                        ALTRIMENTI:
                            next_cursor = null
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "posts": "array di oggetti POST con dati autore e reazione utente",
                "next_cursor": "cursore per prossima pagina o null",
                "has_more": "boolean"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "SOCIAL_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_USER_FOLLOWERS
        # ───────────────────────────────────────────────────────────────────────
        "GET_USER_FOLLOWERS": {
            "descrizione": "Recupera lista follower di un utente",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica privacy profilo",
                    "dettaglio": """
                        target_profile = SELECT privacy_settings FROM PROFILE WHERE user_id == input.user_id
                        
                        SE target_profile.privacy_settings.profile_visibility == 'private':
                            SE NON utente_autenticato:
                                ERRORE: ERROR_PROFILE_PRIVATE
                            ALTRIMENTI SE current_user.id != input.user_id:
                                is_following = EXISTS FOLLOW 
                                    WHERE follower_id == current_user.id 
                                    AND followed_id == input.user_id 
                                    AND status == 'active'
                                SE is_following == false:
                                    ERRORE: ERROR_PROFILE_PRIVATE
                    """
                },
                {
                    "numero": 2,
                    "azione": "Recupera follower",
                    "dettaglio": """
                        followers = SELECT u.id, u.username, p.display_name, p.avatar_url
                        FROM FOLLOW f
                        JOIN USER u ON f.follower_id = u.id
                        JOIN PROFILE p ON f.follower_id = p.user_id
                        WHERE f.followed_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiungi info follow per utente corrente (se autenticato)",
                    "dettaglio": """
                        SE utente_autenticato:
                            PER OGNI follower IN followers:
                                follower.is_following = EXISTS FOLLOW 
                                    WHERE follower_id == current_user.id 
                                    AND followed_id == follower.id 
                                    AND status == 'active'
                                follower.is_followed_by = EXISTS FOLLOW 
                                    WHERE follower_id == follower.id 
                                    AND followed_id == current_user.id 
                                    AND status == 'active'
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "followers": "array di oggetti utente con info follow",
                "next_cursor": "cursore per paginazione",
                "total_count": "numero totale follower"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_060",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                },
                "ERROR_PROFILE_PRIVATE": {
                    "codice": "SOCIAL_061",
                    "messaggio": "Profilo privato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_USER_FOLLOWING
        # ───────────────────────────────────────────────────────────────────────
        "GET_USER_FOLLOWING": {
            "descrizione": "Recupera lista utenti seguiti da un utente",
            "attore": "qualsiasi",
            
            "input": {
                "user_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste USER con id == input.user_id E status == 'active'",
                    "errore_se_falsa": "ERROR_USER_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica privacy (stessa logica di GET_USER_FOLLOWERS)",
                    "dettaglio": "..."
                },
                {
                    "numero": 2,
                    "azione": "Recupera utenti seguiti",
                    "dettaglio": """
                        following = SELECT u.id, u.username, p.display_name, p.avatar_url
                        FROM FOLLOW f
                        JOIN USER u ON f.followed_id = u.id
                        JOIN PROFILE p ON f.followed_id = p.user_id
                        WHERE f.follower_id == input.user_id
                        AND f.status == 'active'
                        AND u.status == 'active'
                        ORDER BY f.created_at DESC
                        LIMIT input.limit
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiungi info follow per utente corrente",
                    "dettaglio": "(stessa logica di GET_USER_FOLLOWERS)"
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "following": "array di oggetti utente",
                "next_cursor": "cursore per paginazione",
                "total_count": "numero totale"
            },
            
            "errori": {
                "ERROR_USER_NOT_FOUND": {
                    "codice": "SOCIAL_070",
                    "messaggio": "Utente non trovato",
                    "http_status": 404
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo SOCIAL"
}



    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO MARKETPLACE
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_LISTING": {
            "descrizione": "Crea un nuovo annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_type": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "product | service | job | housing | vehicle"
                },
                "title": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "5-200 caratteri"
                },
                "description": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "20-10000 caratteri"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "subcategory_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "schema": {
                        "amount": "numero (in centesimi)",
                        "currency": "string (EUR, USD, etc.)",
                        "type": "fixed | negotiable | auction | free | contact"
                    }
                },
                "condition": {
                    "tipo": "enum",
                    "required": False,
                    "validazione": "new | like_new | good | fair | for_parts"
                },
                "media_ids": {
                    "tipo": "array",
                    "required": False,
                    "validazione": "max 20 elementi, UUID di MEDIA_ITEM"
                },
                "location": {
                    "tipo": "json",
                    "required": False
                },
                "attributes": {
                    "tipo": "json",
                    "required": False,
                    "descrizione": "Attributi specifici per categoria"
                },
                "shipping_options": {
                    "tipo": "json",
                    "required": False
                },
                "publish": {
                    "tipo": "boolean",
                    "required": False,
                    "default": False,
                    "descrizione": "Se pubblicare immediatamente"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "USER.status == 'active'",
                    "errore_se_falsa": "ERROR_ACCOUNT_NOT_ACTIVE"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste CATEGORY con id == input.category_id E is_active == true",
                    "errore_se_falsa": "ERROR_INVALID_CATEGORY"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.subcategory_id: Esiste CATEGORY con id == input.subcategory_id E parent_id == input.category_id",
                    "errore_se_falsa": "ERROR_INVALID_SUBCATEGORY"
                },
                {
                    "id": "PRE_5",
                    "condizione": "SE input.media_ids: Tutti i media appartengono a current_user E processing_status == 'completed'",
                    "errore_se_falsa": "ERROR_INVALID_MEDIA"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida attributi specifici per categoria",
                    "dettaglio": """
                        category = SELECT attribute_schema FROM CATEGORY WHERE id == input.category_id
                        
                        SE category.attribute_schema definito:
                            valida(input.attributes, category.attribute_schema)
                    """
                },
                {
                    "numero": 2,
                    "azione": "Costruisci array media",
                    "dettaglio": """
                        media_array = []
                        SE input.media_ids fornito:
                            PER OGNI media_id IN input.media_ids (con indice i):
                                media_item = SELECT * FROM MEDIA_ITEM WHERE id == media_id
                                media_array.append({
                                    id: media_item.id,
                                    type: media_item.file_type,
                                    url: media_item.url,
                                    thumbnail_url: media_item.thumbnail_url,
                                    order: i,
                                    is_cover: (i == 0)
                                })
                    """
                },
                {
                    "numero": 3,
                    "azione": "Determina stato iniziale",
                    "dettaglio": """
                        SE input.publish == true:
                            SE moderazione_richiesta():
                                initial_status = 'pending_review'
                            ALTRIMENTI:
                                initial_status = 'active'
                        ALTRIMENTI:
                            initial_status = 'draft'
                    """
                },
                {
                    "numero": 4,
                    "azione": "Crea record LISTING",
                    "dettaglio": """
                        listing_id = genera_uuid_v4()
                        
                        INSERT INTO LISTING:
                        - id = listing_id
                        - seller_id = current_user.id
                        - listing_type = input.listing_type
                        - title = input.title
                        - description = input.description
                        - category_id = input.category_id
                        - subcategory_id = input.subcategory_id
                        - price = input.price
                        - condition = input.condition
                        - media = media_array
                        - location = input.location
                        - attributes = input.attributes
                        - shipping_options = input.shipping_options
                        - status = initial_status
                        - visibility = 'public'
                        - views_count = 0
                        - favorites_count = 0
                        - inquiries_count = 0
                        - expires_at = now() + 30 giorni (se active)
                        - published_at = now() (se active)
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 5,
                    "azione": "Aggiorna contatore categoria",
                    "dettaglio": """
                        SE initial_status == 'active':
                            UPDATE CATEGORY SET listing_count = listing_count + 1
                            WHERE id == input.category_id
                            
                            UPDATE SELLER_PROFILE SET active_listings_count = active_listings_count + 1
                            WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 6,
                    "azione": "Crea/Aggiorna SELLER_PROFILE se non esiste",
                    "dettaglio": """
                        SE NON esiste SELLER_PROFILE WHERE user_id == current_user.id:
                            INSERT INTO SELLER_PROFILE:
                            - user_id = current_user.id
                            - member_since = now()
                            - updated_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste LISTING con id == listing_id"
                }
            ],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_ACCOUNT_NOT_ACTIVE": {
                    "codice": "MKT_002",
                    "messaggio": "Account non attivo",
                    "http_status": 403
                },
                "ERROR_INVALID_CATEGORY": {
                    "codice": "MKT_003",
                    "messaggio": "Categoria non valida",
                    "http_status": 400
                },
                "ERROR_INVALID_SUBCATEGORY": {
                    "codice": "MKT_004",
                    "messaggio": "Sottocategoria non valida",
                    "http_status": 400
                },
                "ERROR_INVALID_MEDIA": {
                    "codice": "MKT_005",
                    "messaggio": "Uno o più media non validi",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "GET_LISTING": {
            "descrizione": "Recupera dettagli di un annuncio",
            "attore": "qualsiasi",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste LISTING con id == input.listing_id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing con dati correlati",
                    "dettaglio": """
                        listing = SELECT l.*, 
                                  u.username as seller_username,
                                  sp.display_name as seller_display_name,
                                  sp.rating_average,
                                  sp.rating_count,
                                  sp.total_sales,
                                  sp.response_rate,
                                  sp.response_time_hours,
                                  sp.is_verified,
                                  c.name as category_name,
                                  c.slug as category_slug
                        FROM LISTING l
                        JOIN USER u ON l.seller_id = u.id
                        LEFT JOIN SELLER_PROFILE sp ON l.seller_id = sp.user_id
                        JOIN CATEGORY c ON l.category_id = c.id
                        WHERE l.id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Verifica visibilità",
                    "dettaglio": """
                        SE listing.status NOT IN ['active', 'reserved', 'sold']:
                            SE NON utente_autenticato OR current_user.id != listing.seller_id:
                                SE current_user.role NOT IN ['admin', 'moderator']:
                                    ERRORE: ERROR_LISTING_NOT_FOUND
                        
                        SE listing.visibility == 'private':
                            SE NON utente_autenticato OR current_user.id != listing.seller_id:
                                ERRORE: ERROR_LISTING_NOT_FOUND
                    """
                },
                {
                    "numero": 3,
                    "azione": "Incrementa contatore visualizzazioni (se non proprietario)",
                    "dettaglio": """
                        SE NON utente_autenticato OR current_user.id != listing.seller_id:
                            UPDATE LISTING SET views_count = views_count + 1
                            WHERE id == listing.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiungi info utente corrente",
                    "dettaglio": """
                        SE utente_autenticato:
                            listing.is_owner = (current_user.id == listing.seller_id)
                            listing.is_favorited = EXISTS FAVORITE 
                                WHERE user_id == current_user.id 
                                AND listing_id == listing.id
                        ALTRIMENTI:
                            listing.is_owner = false
                            listing.is_favorited = false
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING con dati seller e categoria"
            },
            
            "errori": {
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_010",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: SEARCH_LISTINGS
        # ───────────────────────────────────────────────────────────────────────
        "SEARCH_LISTINGS": {
            "descrizione": "Cerca annunci con filtri",
            "attore": "qualsiasi",
            
            "input": {
                "query": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Testo da cercare in titolo e descrizione"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "listing_type": {
                    "tipo": "enum",
                    "required": False
                },
                "price_min": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Prezzo minimo in centesimi"
                },
                "price_max": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Prezzo massimo in centesimi"
                },
                "condition": {
                    "tipo": "enum",
                    "required": False
                },
                "location": {
                    "tipo": "json",
                    "required": False,
                    "schema": {
                        "latitude": "number",
                        "longitude": "number",
                        "radius_km": "number"
                    }
                },
                "sort_by": {
                    "tipo": "enum",
                    "required": False,
                    "default": "relevance",
                    "validazione": "relevance | price_asc | price_desc | newest | closest"
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query base",
                    "dettaglio": """
                        query = SELECT l.*, sp.rating_average, sp.is_verified
                        FROM LISTING l
                        LEFT JOIN SELLER_PROFILE sp ON l.seller_id = sp.user_id
                        WHERE l.status == 'active'
                        AND l.visibility == 'public'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Applica filtri",
                    "dettaglio": """
                        SE input.query fornito:
                            query += AND (
                                l.title ILIKE '%{input.query}%'
                                OR l.description ILIKE '%{input.query}%'
                            )
                        
                        SE input.category_id fornito:
                            # Include anche sottocategorie
                            category_ids = getCategoryAndDescendants(input.category_id)
                            query += AND l.category_id IN category_ids
                        
                        SE input.listing_type fornito:
                            query += AND l.listing_type == input.listing_type
                        
                        SE input.price_min fornito:
                            query += AND l.price->>'amount' >= input.price_min
                        
                        SE input.price_max fornito:
                            query += AND l.price->>'amount' <= input.price_max
                        
                        SE input.condition fornito:
                            query += AND l.condition == input.condition
                        
                        SE input.location fornito:
                            query += AND calcola_distanza(
                                l.location->>'latitude', 
                                l.location->>'longitude',
                                input.location.latitude,
                                input.location.longitude
                            ) <= input.location.radius_km
                    """
                },
                {
                    "numero": 3,
                    "azione": "Applica ordinamento",
                    "dettaglio": """
                        SE input.sort_by == 'relevance':
                            query += ORDER BY calcola_relevance_score(l) DESC
                        ALTRIMENTI SE input.sort_by == 'price_asc':
                            query += ORDER BY l.price->>'amount' ASC
                        ALTRIMENTI SE input.sort_by == 'price_desc':
                            query += ORDER BY l.price->>'amount' DESC
                        ALTRIMENTI SE input.sort_by == 'newest':
                            query += ORDER BY l.published_at DESC
                        ALTRIMENTI SE input.sort_by == 'closest':
                            query += ORDER BY distanza ASC
                    """
                },
                {
                    "numero": 4,
                    "azione": "Applica paginazione",
                    "dettaglio": """
                        query += LIMIT input.limit + 1  # +1 per has_more
                        
                        SE input.cursor fornito:
                            Applica offset basato su cursor
                    """
                },
                {
                    "numero": 5,
                    "azione": "Esegui query e formatta risultati",
                    "dettaglio": """
                        results = esegui_query(query)
                        
                        has_more = results.length > input.limit
                        SE has_more:
                            results = results[0:input.limit]
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listings": "array di oggetti LISTING",
                "total_count": "numero totale risultati (se disponibile)",
                "next_cursor": "cursore per paginazione",
                "has_more": "boolean"
            },
            
            "errori": {}
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: MAKE_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "MAKE_OFFER": {
            "descrizione": "Fai un'offerta per un annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "amount": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "maggiore di 0 (in centesimi)"
                },
                "message": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 1000 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_AVAILABLE"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id != LISTING.seller_id",
                    "errore_se_falsa": "ERROR_CANNOT_OFFER_OWN_LISTING"
                },
                {
                    "id": "PRE_4",
                    "condizione": "LISTING.price.type IN ['negotiable', 'auction']",
                    "errore_se_falsa": "ERROR_OFFERS_NOT_ACCEPTED"
                },
                {
                    "id": "PRE_5",
                    "condizione": "NON esiste OFFER con buyer_id == current_user.id E listing_id == input.listing_id E status == 'pending'",
                    "errore_se_falsa": "ERROR_OFFER_ALREADY_EXISTS"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea record OFFER",
                    "dettaglio": """
                        offer_id = genera_uuid_v4()
                        
                        INSERT INTO OFFER:
                        - id = offer_id
                        - listing_id = input.listing_id
                        - buyer_id = current_user.id
                        - seller_id = listing.seller_id
                        - amount = input.amount
                        - currency = listing.price.currency
                        - message = input.message
                        - status = 'pending'
                        - expires_at = now() + 48 ore
                        - created_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna contatore inquiries",
                    "dettaglio": """
                        UPDATE LISTING SET inquiries_count = inquiries_count + 1
                        WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica venditore",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = listing.seller_id,
                            type = 'new_offer',
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer_id,
                            metadata = {
                                listing_id: input.listing_id,
                                amount: input.amount
                            }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste OFFER con id == offer_id"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_AVAILABLE": {
                    "codice": "MKT_021",
                    "messaggio": "Annuncio non disponibile",
                    "http_status": 404
                },
                "ERROR_CANNOT_OFFER_OWN_LISTING": {
                    "codice": "MKT_022",
                    "messaggio": "Non puoi fare offerte sui tuoi annunci",
                    "http_status": 400
                },
                "ERROR_OFFERS_NOT_ACCEPTED": {
                    "codice": "MKT_023",
                    "messaggio": "Questo annuncio non accetta offerte",
                    "http_status": 400
                },
                "ERROR_OFFER_ALREADY_EXISTS": {
                    "codice": "MKT_024",
                    "messaggio": "Hai già un'offerta in attesa per questo annuncio",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: RESPOND_TO_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "RESPOND_TO_OFFER": {
            "descrizione": "Rispondi a un'offerta (accetta, rifiuta, contro-offerta)",
            "attore": "utente_autenticato",
            
            "input": {
                "offer_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "action": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "accept | reject | counter"
                },
                "counter_amount": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Richiesto se action == 'counter'"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste OFFER con id == input.offer_id E status IN ['pending', 'countered']",
                    "errore_se_falsa": "ERROR_OFFER_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id == OFFER.seller_id",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.action == 'counter': input.counter_amount fornito E > 0",
                    "errore_se_falsa": "ERROR_COUNTER_AMOUNT_REQUIRED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera offerta e listing",
                    "dettaglio": """
                        offer = SELECT * FROM OFFER WHERE id == input.offer_id
                        listing = SELECT * FROM LISTING WHERE id == offer.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Esegui azione in base a input.action",
                    "dettaglio": """
                        SE input.action == 'accept':
                            # Accetta l'offerta
                            final_amount = SE offer.status == 'countered' E offer.counter_amount:
                                offer.counter_amount
                            ALTRIMENTI:
                                offer.amount
                            
                            UPDATE OFFER SET
                            - status = 'accepted'
                            - responded_at = now()
                            WHERE id == offer.id
                            
                            # Crea transazione
                            transaction_id = genera_uuid_v4()
                            platform_fee = calcola_commissione(final_amount)
                            
                            INSERT INTO TRANSACTION:
                            - id = transaction_id
                            - listing_id = listing.id
                            - offer_id = offer.id
                            - buyer_id = offer.buyer_id
                            - seller_id = offer.seller_id
                            - amount = final_amount
                            - currency = offer.currency
                            - platform_fee = platform_fee
                            - seller_payout = final_amount - platform_fee
                            - status = 'pending_payment'
                            - created_at = now()
                            - updated_at = now()
                            
                            # Aggiorna listing
                            UPDATE LISTING SET status = 'reserved' WHERE id == listing.id
                            
                            # Rifiuta altre offerte pending
                            UPDATE OFFER SET status = 'rejected', responded_at = now()
                            WHERE listing_id == listing.id
                            AND id != offer.id
                            AND status == 'pending'
                        
                        ALTRIMENTI SE input.action == 'reject':
                            UPDATE OFFER SET
                            - status = 'rejected'
                            - responded_at = now()
                            WHERE id == offer.id
                        
                        ALTRIMENTI SE input.action == 'counter':
                            UPDATE OFFER SET
                            - status = 'countered'
                            - counter_amount = input.counter_amount
                            - responded_at = now()
                            WHERE id == offer.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Notifica acquirente",
                    "dettaglio": """
                        notification_type = SE input.action == 'accept': 'offer_accepted'
                                           ALTRIMENTI SE input.action == 'reject': 'offer_rejected'
                                           ALTRIMENTI: 'offer_countered'
                        
                        crea_notifica(
                            recipient_id = offer.buyer_id,
                            type = notification_type,
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer.id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "OFFER.status aggiornato"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER aggiornato",
                "transaction": "oggetto TRANSACTION se action == 'accept'"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_OFFER_NOT_FOUND": {
                    "codice": "MKT_031",
                    "messaggio": "Offerta non trovata",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "MKT_032",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_COUNTER_AMOUNT_REQUIRED": {
                    "codice": "MKT_033",
                    "messaggio": "Importo contro-offerta richiesto",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ADD_TO_FAVORITES
        # ───────────────────────────────────────────────────────────────────────
        "ADD_TO_FAVORITES": {
            "descrizione": "Aggiungi annuncio ai preferiti",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "NON esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id",
                    "errore_se_falsa": "ERROR_ALREADY_FAVORITED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Crea record FAVORITE",
                    "dettaglio": """
                        INSERT INTO FAVORITE:
                        - id = genera_uuid_v4()
                        - user_id = current_user.id
                        - listing_id = input.listing_id
                        - created_at = now()
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna contatore",
                    "dettaglio": """
                        UPDATE LISTING SET favorites_count = favorites_count + 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FAVORITE"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio aggiunto ai preferiti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_041",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FAVORITED": {
                    "codice": "MKT_042",
                    "messaggio": "Annuncio già nei preferiti",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: LEAVE_REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "LEAVE_REVIEW": {
            "descrizione": "Lascia una recensione dopo una transazione",
            "attore": "utente_autenticato",
            
            "input": {
                "transaction_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "1-5"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 100 caratteri"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 2000 caratteri"
                },
                "aspects": {
                    "tipo": "json",
                    "required": False,
                    "descrizione": "Valutazioni aspetti specifici"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste TRANSACTION con id == input.transaction_id E status == 'completed'",
                    "errore_se_falsa": "ERROR_TRANSACTION_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id == TRANSACTION.buyer_id OR current_user.id == TRANSACTION.seller_id",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste REVIEW con transaction_id == input.transaction_id E reviewer_id == current_user.id",
                    "errore_se_falsa": "ERROR_ALREADY_REVIEWED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera transazione e determina tipo recensione",
                    "dettaglio": """
                        transaction = SELECT * FROM TRANSACTION WHERE id == input.transaction_id
                        
                        SE current_user.id == transaction.buyer_id:
                            review_type = 'buyer_to_seller'
                            reviewee_id = transaction.seller_id
                        ALTRIMENTI:
                            review_type = 'seller_to_buyer'
                            reviewee_id = transaction.buyer_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea record REVIEW",
                    "dettaglio": """
                        review_id = genera_uuid_v4()
                        
                        INSERT INTO REVIEW:
                        - id = review_id
                        - transaction_id = input.transaction_id
                        - reviewer_id = current_user.id
                        - reviewee_id = reviewee_id
                        - review_type = review_type
                        - rating = input.rating
                        - title = input.title
                        - content = input.content
                        - aspects = input.aspects
                        - is_public = true
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna statistiche SELLER_PROFILE",
                    "dettaglio": """
                        SE review_type == 'buyer_to_seller':
                            # Ricalcola media recensioni
                            UPDATE SELLER_PROFILE SET
                            - rating_count = rating_count + 1
                            - rating_average = (rating_average * (rating_count - 1) + input.rating) / rating_count
                            - updated_at = now()
                            WHERE user_id == reviewee_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica recensito",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = reviewee_id,
                            type = 'new_review',
                            source_user_id = current_user.id,
                            target_type = 'review',
                            target_id = review_id,
                            metadata = { rating: input.rating }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste REVIEW con id == review_id"
                }
            ],
            
            "output": {
                "success": True,
                "review": "oggetto REVIEW creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_TRANSACTION_NOT_FOUND": {
                    "codice": "MKT_051",
                    "messaggio": "Transazione non trovata o non completata",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "MKT_052",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_ALREADY_REVIEWED": {
                    "codice": "MKT_053",
                    "messaggio": "Hai già lasciato una recensione",
                    "http_status": 409
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo MARKETPLACE"
}


    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO MARKETPLACE
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_LISTING": {
            "descrizione": "Crea un nuovo annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_type": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "product | service | job | housing | vehicle"
                },
                "title": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "5-200 caratteri"
                },
                "description": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "20-10000 caratteri"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "subcategory_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "schema": {
                        "amount": "integer (centesimi)",
                        "currency": "string ISO 4217",
                        "type": "fixed | negotiable | auction | free | contact"
                    }
                },
                "condition": {
                    "tipo": "enum",
                    "required": False,
                    "validazione": "new | like_new | good | fair | for_parts"
                },
                "media_ids": {
                    "tipo": "array",
                    "required": False,
                    "validazione": "array di UUID, max 20"
                },
                "location": {
                    "tipo": "json",
                    "required": False
                },
                "attributes": {
                    "tipo": "json",
                    "required": False
                },
                "shipping_options": {
                    "tipo": "json",
                    "required": False
                },
                "publish": {
                    "tipo": "boolean",
                    "required": False,
                    "default": False,
                    "descrizione": "Se true, pubblica immediatamente"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "current_user.status == 'active'",
                    "errore_se_falsa": "ERROR_ACCOUNT_NOT_ACTIVE"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste CATEGORY con id == input.category_id E is_active == true",
                    "errore_se_falsa": "ERROR_INVALID_CATEGORY"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.subcategory_id: Esiste CATEGORY con id == input.subcategory_id E parent_id == input.category_id",
                    "errore_se_falsa": "ERROR_INVALID_SUBCATEGORY"
                },
                {
                    "id": "PRE_5",
                    "condizione": "input.price.amount >= 0",
                    "errore_se_falsa": "ERROR_INVALID_PRICE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida e costruisci oggetto media",
                    "dettaglio": """
                        media_array = []
                        SE input.media_ids fornito E non vuoto:
                            PER OGNI media_id IN input.media_ids (con indice i):
                                media = SELECT * FROM MEDIA_ITEM WHERE id == media_id AND uploader_id == current_user.id
                                SE media non esiste:
                                    ERRORE: ERROR_INVALID_MEDIA
                                
                                media_array.push({
                                    id: media.id,
                                    type: media.file_type,
                                    url: media.url,
                                    thumbnail_url: media.thumbnail_url,
                                    order: i,
                                    is_cover: (i == 0)
                                })
                    """
                },
                {
                    "numero": 2,
                    "azione": "Determina stato iniziale",
                    "dettaglio": """
                        SE input.publish == true:
                            SE configurazione.moderazione_annunci_abilitata:
                                initial_status = 'pending_review'
                            ALTRIMENTI:
                                initial_status = 'active'
                        ALTRIMENTI:
                            initial_status = 'draft'
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record LISTING",
                    "dettaglio": """
                        listing_id = genera_uuid_v4()
                        
                        INSERT INTO LISTING:
                        - id = listing_id
                        - seller_id = current_user.id
                        - listing_type = input.listing_type
                        - title = input.title
                        - description = input.description
                        - category_id = input.category_id
                        - subcategory_id = input.subcategory_id
                        - price = input.price
                        - condition = input.condition
                        - media = media_array
                        - location = input.location
                        - attributes = input.attributes
                        - shipping_options = input.shipping_options
                        - status = initial_status
                        - visibility = 'public'
                        - views_count = 0
                        - favorites_count = 0
                        - inquiries_count = 0
                        - expires_at = SE initial_status == 'active': now() + 30 giorni ALTRIMENTI null
                        - published_at = SE initial_status == 'active': now() ALTRIMENTI null
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna contatori categoria se pubblicato",
                    "dettaglio": """
                        SE initial_status == 'active':
                            UPDATE CATEGORY SET listing_count = listing_count + 1
                            WHERE id == input.category_id
                            
                            SE input.subcategory_id:
                                UPDATE CATEGORY SET listing_count = listing_count + 1
                                WHERE id == input.subcategory_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Aggiorna profilo venditore",
                    "dettaglio": """
                        SE initial_status == 'active':
                            UPDATE SELLER_PROFILE SET 
                            - active_listings_count = active_listings_count + 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 6,
                    "azione": "Crea SELLER_PROFILE se non esiste",
                    "dettaglio": """
                        SE NON esiste SELLER_PROFILE con user_id == current_user.id:
                            INSERT INTO SELLER_PROFILE:
                            - user_id = current_user.id
                            - active_listings_count = SE initial_status == 'active': 1 ALTRIMENTI 0
                            - member_since = now()
                            - updated_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste LISTING con id == listing_id"
                }
            ],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_ACCOUNT_NOT_ACTIVE": {
                    "codice": "MKT_002",
                    "messaggio": "Account non attivo",
                    "http_status": 403
                },
                "ERROR_INVALID_CATEGORY": {
                    "codice": "MKT_003",
                    "messaggio": "Categoria non valida",
                    "http_status": 400
                },
                "ERROR_INVALID_SUBCATEGORY": {
                    "codice": "MKT_004",
                    "messaggio": "Sottocategoria non valida per questa categoria",
                    "http_status": 400
                },
                "ERROR_INVALID_PRICE": {
                    "codice": "MKT_005",
                    "messaggio": "Prezzo non valido",
                    "http_status": 400
                },
                "ERROR_INVALID_MEDIA": {
                    "codice": "MKT_006",
                    "messaggio": "Media non valido o non di proprietà",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UPDATE_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "UPDATE_LISTING": {
            "descrizione": "Modifica un annuncio esistente",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "title": {"tipo": "string", "required": False},
                "description": {"tipo": "string", "required": False},
                "price": {"tipo": "json", "required": False},
                "condition": {"tipo": "enum", "required": False},
                "media_ids": {"tipo": "array", "required": False},
                "location": {"tipo": "json", "required": False},
                "attributes": {"tipo": "json", "required": False},
                "shipping_options": {"tipo": "json", "required": False}
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.seller_id == current_user.id",
                    "errore_se_falsa": "ERROR_NOT_OWNER"
                },
                {
                    "id": "PRE_4",
                    "condizione": "LISTING.status IN ['draft', 'active', 'expired']",
                    "errore_se_falsa": "ERROR_CANNOT_EDIT"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing esistente",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna campi forniti",
                    "dettaglio": """
                        UPDATE LISTING SET
                        - title = input.title SE fornito ALTRIMENTI listing.title
                        - description = input.description SE fornito ALTRIMENTI listing.description
                        - price = input.price SE fornito ALTRIMENTI listing.price
                        - condition = input.condition SE fornito ALTRIMENTI listing.condition
                        - media = costruisci_media_array(input.media_ids) SE fornito ALTRIMENTI listing.media
                        - location = input.location SE fornito ALTRIMENTI listing.location
                        - attributes = input.attributes SE fornito ALTRIMENTI listing.attributes
                        - shipping_options = input.shipping_options SE fornito ALTRIMENTI listing.shipping_options
                        - updated_at = now()
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "LISTING.updated_at aggiornato"
                }
            ],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING aggiornato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_011",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_NOT_OWNER": {
                    "codice": "MKT_012",
                    "messaggio": "Non sei il proprietario di questo annuncio",
                    "http_status": 403
                },
                "ERROR_CANNOT_EDIT": {
                    "codice": "MKT_013",
                    "messaggio": "Impossibile modificare annuncio in questo stato",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: PUBLISH_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "PUBLISH_LISTING": {
            "descrizione": "Pubblica un annuncio in bozza",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E seller_id == current_user.id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.status IN ['draft', 'expired']",
                    "errore_se_falsa": "ERROR_CANNOT_PUBLISH"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida completezza annuncio",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                        
                        SE listing.title vuoto O listing.description vuoto O listing.price vuoto:
                            ERRORE: ERROR_INCOMPLETE_LISTING
                    """
                },
                {
                    "numero": 2,
                    "azione": "Determina stato post-pubblicazione",
                    "dettaglio": """
                        SE configurazione.moderazione_annunci_abilitata:
                            new_status = 'pending_review'
                            published_at = null
                        ALTRIMENTI:
                            new_status = 'active'
                            published_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna LISTING",
                    "dettaglio": """
                        UPDATE LISTING SET
                        - status = new_status
                        - published_at = published_at
                        - expires_at = SE new_status == 'active': now() + 30 giorni ALTRIMENTI null
                        - updated_at = now()
                        WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiorna contatori se attivo",
                    "dettaglio": """
                        SE new_status == 'active':
                            UPDATE CATEGORY SET listing_count = listing_count + 1
                            WHERE id == listing.category_id
                            
                            UPDATE SELLER_PROFILE SET active_listings_count = active_listings_count + 1
                            WHERE user_id == current_user.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "LISTING.status IN ['pending_review', 'active']"
                }
            ],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING",
                "status": "pending_review | active"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_021",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_CANNOT_PUBLISH": {
                    "codice": "MKT_022",
                    "messaggio": "Annuncio non può essere pubblicato da questo stato",
                    "http_status": 400
                },
                "ERROR_INCOMPLETE_LISTING": {
                    "codice": "MKT_023",
                    "messaggio": "Annuncio incompleto",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: DELETE_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "DELETE_LISTING": {
            "descrizione": "Elimina un annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.seller_id == current_user.id OPPURE current_user.role IN ['admin', 'moderator']",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esistono TRANSACTION con listing_id == input.listing_id E status IN ['paid', 'shipped']",
                    "errore_se_falsa": "ERROR_ACTIVE_TRANSACTION"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing e stato precedente",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                        was_active = (listing.status == 'active')
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna status a deleted",
                    "dettaglio": """
                        UPDATE LISTING SET
                        - status = 'deleted'
                        - updated_at = now()
                        WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna contatori se era attivo",
                    "dettaglio": """
                        SE was_active:
                            UPDATE CATEGORY SET listing_count = listing_count - 1
                            WHERE id == listing.category_id
                            
                            UPDATE SELLER_PROFILE SET active_listings_count = active_listings_count - 1
                            WHERE user_id == listing.seller_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Cancella offerte pendenti",
                    "dettaglio": """
                        UPDATE OFFER SET status = 'withdrawn'
                        WHERE listing_id == input.listing_id
                        AND status == 'pending'
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "LISTING.status == 'deleted'"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio eliminato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_031",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "MKT_032",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_ACTIVE_TRANSACTION": {
                    "codice": "MKT_033",
                    "messaggio": "Impossibile eliminare: transazione in corso",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "GET_LISTING": {
            "descrizione": "Recupera dettagli di un annuncio",
            "attore": "qualsiasi",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste LISTING con id == input.listing_id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "LISTING.status == 'active' OPPURE LISTING.seller_id == current_user.id OPPURE current_user.role IN ['admin', 'moderator']",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing con dati correlati",
                    "dettaglio": """
                        listing = SELECT l.*, 
                            u.username as seller_username,
                            sp.display_name as seller_display_name,
                            sp.rating_average as seller_rating,
                            sp.rating_count as seller_rating_count,
                            sp.is_verified as seller_is_verified,
                            c.name as category_name,
                            c.slug as category_slug
                        FROM LISTING l
                        JOIN USER u ON l.seller_id = u.id
                        LEFT JOIN SELLER_PROFILE sp ON u.id = sp.user_id
                        JOIN CATEGORY c ON l.category_id = c.id
                        WHERE l.id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Incrementa visualizzazioni se utente diverso dal proprietario",
                    "dettaglio": """
                        SE NOT utente_autenticato OR current_user.id != listing.seller_id:
                            UPDATE LISTING SET views_count = views_count + 1
                            WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiungi stato preferito se autenticato",
                    "dettaglio": """
                        SE utente_autenticato:
                            listing.is_favorite = EXISTS FAVORITE 
                                WHERE user_id == current_user.id 
                                AND listing_id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING completo con dati venditore e categoria"
            },
            
            "errori": {
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_040",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: SEARCH_LISTINGS
        # ───────────────────────────────────────────────────────────────────────
        "SEARCH_LISTINGS": {
            "descrizione": "Cerca annunci con filtri",
            "attore": "qualsiasi",
            
            "input": {
                "query": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Testo di ricerca"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "listing_type": {
                    "tipo": "enum",
                    "required": False
                },
                "price_min": {
                    "tipo": "integer",
                    "required": False
                },
                "price_max": {
                    "tipo": "integer",
                    "required": False
                },
                "condition": {
                    "tipo": "enum",
                    "required": False
                },
                "location": {
                    "tipo": "json",
                    "required": False,
                    "schema": {
                        "city": "string",
                        "radius_km": "integer"
                    }
                },
                "sort_by": {
                    "tipo": "enum",
                    "required": False,
                    "default": "relevance",
                    "validazione": "relevance | newest | price_asc | price_desc | distance"
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query base",
                    "dettaglio": """
                        query = SELECT l.*, sp.rating_average, sp.is_verified
                        FROM LISTING l
                        LEFT JOIN SELLER_PROFILE sp ON l.seller_id = sp.user_id
                        WHERE l.status == 'active'
                        AND l.visibility == 'public'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Applica filtri",
                    "dettaglio": """
                        SE input.query fornito:
                            query += AND (l.title ILIKE '%{input.query}%' OR l.description ILIKE '%{input.query}%')
                        
                        SE input.category_id fornito:
                            query += AND (l.category_id == input.category_id OR l.subcategory_id == input.category_id)
                        
                        SE input.listing_type fornito:
                            query += AND l.listing_type == input.listing_type
                        
                        SE input.price_min fornito:
                            query += AND (l.price->>'amount')::int >= input.price_min
                        
                        SE input.price_max fornito:
                            query += AND (l.price->>'amount')::int <= input.price_max
                        
                        SE input.condition fornito:
                            query += AND l.condition == input.condition
                        
                        SE input.location fornito:
                            # Filtro per città o raggio geografico
                            query += AND l.location->>'city' == input.location.city
                    """
                },
                {
                    "numero": 3,
                    "azione": "Escludi utenti bloccati se autenticato",
                    "dettaglio": """
                        SE utente_autenticato:
                            blocked_ids = SELECT blocked_id FROM BLOCK WHERE blocker_id == current_user.id
                            query += AND l.seller_id NOT IN blocked_ids
                    """
                },
                {
                    "numero": 4,
                    "azione": "Applica ordinamento",
                    "dettaglio": """
                        SE input.sort_by == 'newest':
                            query += ORDER BY l.published_at DESC
                        ALTRIMENTI SE input.sort_by == 'price_asc':
                            query += ORDER BY (l.price->>'amount')::int ASC
                        ALTRIMENTI SE input.sort_by == 'price_desc':
                            query += ORDER BY (l.price->>'amount')::int DESC
                        ALTRIMENTI:  # relevance
                            query += ORDER BY l.promoted_until DESC NULLS LAST, l.views_count DESC, l.published_at DESC
                    """
                },
                {
                    "numero": 5,
                    "azione": "Applica paginazione ed esegui",
                    "dettaglio": """
                        query += LIMIT input.limit + 1
                        
                        results = esegui_query(query)
                        
                        SE results.length > input.limit:
                            has_more = true
                            results = results[0:input.limit]
                        ALTRIMENTI:
                            has_more = false
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listings": "array di annunci",
                "total_count": "numero totale (stimato)",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {}
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: MAKE_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "MAKE_OFFER": {
            "descrizione": "Fai un'offerta per un annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "amount": {
                    "tipo": "integer",
                    "required": True,
                    "descrizione": "Importo in centesimi"
                },
                "message": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 1000 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.seller_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_BID_OWN"
                },
                {
                    "id": "PRE_4",
                    "condizione": "LISTING.price.type IN ['negotiable', 'auction']",
                    "errore_se_falsa": "ERROR_NOT_NEGOTIABLE"
                },
                {
                    "id": "PRE_5",
                    "condizione": "input.amount > 0",
                    "errore_se_falsa": "ERROR_INVALID_AMOUNT"
                },
                {
                    "id": "PRE_6",
                    "condizione": "NON esiste OFFER con listing_id == input.listing_id E buyer_id == current_user.id E status == 'pending'",
                    "errore_se_falsa": "ERROR_OFFER_EXISTS"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea OFFER",
                    "dettaglio": """
                        offer_id = genera_uuid_v4()
                        
                        INSERT INTO OFFER:
                        - id = offer_id
                        - listing_id = input.listing_id
                        - buyer_id = current_user.id
                        - seller_id = listing.seller_id
                        - amount = input.amount
                        - currency = listing.price.currency
                        - message = input.message
                        - status = 'pending'
                        - expires_at = now() + 48 ore
                        - created_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Incrementa contatore richieste",
                    "dettaglio": """
                        UPDATE LISTING SET inquiries_count = inquiries_count + 1
                        WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica venditore",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = listing.seller_id,
                            type = 'new_offer',
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer_id,
                            metadata = {
                                listing_id: listing.id,
                                listing_title: listing.title,
                                amount: input.amount,
                                currency: listing.price.currency
                            }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste OFFER con id == offer_id"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_051",
                    "messaggio": "Annuncio non trovato o non attivo",
                    "http_status": 404
                },
                "ERROR_CANNOT_BID_OWN": {
                    "codice": "MKT_052",
                    "messaggio": "Non puoi fare offerte sui tuoi annunci",
                    "http_status": 400
                },
                "ERROR_NOT_NEGOTIABLE": {
                    "codice": "MKT_053",
                    "messaggio": "Questo annuncio non accetta offerte",
                    "http_status": 400
                },
                "ERROR_INVALID_AMOUNT": {
                    "codice": "MKT_054",
                    "messaggio": "Importo non valido",
                    "http_status": 400
                },
                "ERROR_OFFER_EXISTS": {
                    "codice": "MKT_055",
                    "messaggio": "Hai già un'offerta pendente per questo annuncio",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: RESPOND_TO_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "RESPOND_TO_OFFER": {
            "descrizione": "Rispondi a un'offerta (accetta, rifiuta, contro-offerta)",
            "attore": "utente_autenticato",
            
            "input": {
                "offer_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "action": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "accept | reject | counter"
                },
                "counter_amount": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Richiesto se action == 'counter'"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste OFFER con id == input.offer_id E status == 'pending'",
                    "errore_se_falsa": "ERROR_OFFER_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "OFFER.seller_id == current_user.id",
                    "errore_se_falsa": "ERROR_NOT_SELLER"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.action == 'counter': input.counter_amount fornito E > 0",
                    "errore_se_falsa": "ERROR_COUNTER_AMOUNT_REQUIRED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera offerta e listing",
                    "dettaglio": """
                        offer = SELECT * FROM OFFER WHERE id == input.offer_id
                        listing = SELECT * FROM LISTING WHERE id == offer.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Gestisci azione",
                    "dettaglio": """
                        SE input.action == 'accept':
                            # Accetta offerta
                            UPDATE OFFER SET status = 'accepted', responded_at = now()
                            WHERE id == input.offer_id
                            
                            # Crea transazione
                            transaction_id = genera_uuid_v4()
                            platform_fee = calcola_commissione(offer.amount)
                            
                            INSERT INTO TRANSACTION:
                            - id = transaction_id
                            - listing_id = listing.id
                            - offer_id = offer.id
                            - buyer_id = offer.buyer_id
                            - seller_id = offer.seller_id
                            - amount = offer.amount
                            - currency = offer.currency
                            - platform_fee = platform_fee
                            - seller_payout = offer.amount - platform_fee
                            - status = 'pending_payment'
                            - created_at = now()
                            - updated_at = now()
                            
                            # Aggiorna listing a reserved
                            UPDATE LISTING SET status = 'reserved' WHERE id == listing.id
                            
                            # Rifiuta altre offerte pendenti
                            UPDATE OFFER SET status = 'rejected', responded_at = now()
                            WHERE listing_id == listing.id AND id != input.offer_id AND status == 'pending'
                        
                        ALTRIMENTI SE input.action == 'reject':
                            UPDATE OFFER SET status = 'rejected', responded_at = now()
                            WHERE id == input.offer_id
                        
                        ALTRIMENTI SE input.action == 'counter':
                            UPDATE OFFER SET 
                            - status = 'countered'
                            - counter_amount = input.counter_amount
                            - responded_at = now()
                            - expires_at = now() + 48 ore
                            WHERE id == input.offer_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Notifica acquirente",
                    "dettaglio": """
                        notification_type = 
                            SE input.action == 'accept': 'offer_accepted'
                            ALTRIMENTI SE input.action == 'reject': 'offer_rejected'
                            ALTRIMENTI: 'offer_countered'
                        
                        crea_notifica(
                            recipient_id = offer.buyer_id,
                            type = notification_type,
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer.id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "OFFER.status IN ['accepted', 'rejected', 'countered']"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER aggiornato",
                "transaction": "oggetto TRANSACTION se action == 'accept'"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_OFFER_NOT_FOUND": {
                    "codice": "MKT_061",
                    "messaggio": "Offerta non trovata o già gestita",
                    "http_status": 404
                },
                "ERROR_NOT_SELLER": {
                    "codice": "MKT_062",
                    "messaggio": "Solo il venditore può rispondere all'offerta",
                    "http_status": 403
                },
                "ERROR_COUNTER_AMOUNT_REQUIRED": {
                    "codice": "MKT_063",
                    "messaggio": "Importo contro-offerta richiesto",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ADD_TO_FAVORITES
        # ───────────────────────────────────────────────────────────────────────
        "ADD_TO_FAVORITES": {
            "descrizione": "Aggiungi annuncio ai preferiti",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "NON esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id",
                    "errore_se_falsa": "ERROR_ALREADY_FAVORITE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Crea FAVORITE",
                    "dettaglio": """
                        INSERT INTO FAVORITE:
                        - id = genera_uuid_v4()
                        - user_id = current_user.id
                        - listing_id = input.listing_id
                        - created_at = now()
                    """
                },
                {
                    "numero": 2,
                    "azione": "Incrementa contatore",
                    "dettaglio": """
                        UPDATE LISTING SET favorites_count = favorites_count + 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio aggiunto ai preferiti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_070",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MKT_071",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FAVORITE": {
                    "codice": "MKT_072",
                    "messaggio": "Annuncio già nei preferiti",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: REMOVE_FROM_FAVORITES
        # ───────────────────────────────────────────────────────────────────────
        "REMOVE_FROM_FAVORITES": {
            "descrizione": "Rimuovi annuncio dai preferiti",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id",
                    "errore_se_falsa": "ERROR_NOT_FAVORITE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina FAVORITE",
                    "dettaglio": """
                        DELETE FROM FAVORITE 
                        WHERE user_id == current_user.id 
                        AND listing_id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Decrementa contatore",
                    "dettaglio": """
                        UPDATE LISTING SET favorites_count = favorites_count - 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio rimosso dai preferiti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_080",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_FAVORITE": {
                    "codice": "MKT_081",
                    "messaggio": "Annuncio non nei preferiti",
                    "http_status": 404
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: LEAVE_REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "LEAVE_REVIEW": {
            "descrizione": "Lascia una recensione dopo una transazione completata",
            "attore": "utente_autenticato",
            
            "input": {
                "transaction_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "1-5"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 100 caratteri"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 2000 caratteri"
                },
                "aspects": {
                    "tipo": "json",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste TRANSACTION con id == input.transaction_id E status == 'completed'",
                    "errore_se_falsa": "ERROR_TRANSACTION_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id IN [TRANSACTION.buyer_id, TRANSACTION.seller_id]",
                    "errore_se_falsa": "ERROR_NOT_PARTICIPANT"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste REVIEW con transaction_id == input.transaction_id E reviewer_id == current_user.id",
                    "errore_se_falsa": "ERROR_ALREADY_REVIEWED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera transazione e determina direzione recensione",
                    "dettaglio": """
                        transaction = SELECT * FROM TRANSACTION WHERE id == input.transaction_id
                        
                        SE current_user.id == transaction.buyer_id:
                            review_type = 'buyer_to_seller'
                            reviewee_id = transaction.seller_id
                        ALTRIMENTI:
                            review_type = 'seller_to_buyer'
                            reviewee_id = transaction.buyer_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea REVIEW",
                    "dettaglio": """
                        review_id = genera_uuid_v4()
                        
                        INSERT INTO REVIEW:
                        - id = review_id
                        - transaction_id = input.transaction_id
                        - reviewer_id = current_user.id
                        - reviewee_id = reviewee_id
                        - review_type = review_type
                        - rating = input.rating
                        - title = input.title
                        - content = input.content
                        - aspects = input.aspects
                        - is_public = true
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna statistiche SELLER_PROFILE del recensito",
                    "dettaglio": """
                        # Calcola nuova media
                        current_stats = SELECT rating_average, rating_count FROM SELLER_PROFILE WHERE user_id == reviewee_id
                        
                        new_count = current_stats.rating_count + 1
                        new_average = ((current_stats.rating_average * current_stats.rating_count) + input.rating) / new_count
                        
                        UPDATE SELLER_PROFILE SET
                        - rating_average = new_average
                        - rating_count = new_count
                        - updated_at = now()
                        WHERE user_id == reviewee_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica recensito",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = reviewee_id,
                            type = 'new_review',
                            source_user_id = current_user.id,
                            target_type = 'review',
                            target_id = review_id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste REVIEW con id == review_id"
                }
            ],
            
            "output": {
                "success": True,
                "review": "oggetto REVIEW creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MKT_090",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_TRANSACTION_NOT_FOUND": {
                    "codice": "MKT_091",
                    "messaggio": "Transazione non trovata o non completata",
                    "http_status": 404
                },
                "ERROR_NOT_PARTICIPANT": {
                    "codice": "MKT_092",
                    "messaggio": "Non sei un partecipante di questa transazione",
                    "http_status": 403
                },
                "ERROR_ALREADY_REVIEWED": {
                    "codice": "MKT_093",
                    "messaggio": "Hai già lasciato una recensione per questa transazione",
                    "http_status": 409
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo MARKETPLACE"
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: BOOKING
# VERSIONE: 1.0
# DESCRIZIONE: Prenotazioni, disponibilità, calendario, appuntamenti
# DIPENDENZE: IDENTITY, MESSAGING
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_BOOKING = {
    "nome": "BOOKING",
    "versione": "1.0",
    "descrizione": "Prenotazioni, disponibilità, calendario, appuntamenti",
    "dipendenze": ["IDENTITY", "MESSAGING"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: BOOKABLE_RESOURCE
        # ───────────────────────────────────────────────────────────────────────
        "BOOKABLE_RESOURCE": {
            "descrizione": "Risorsa prenotabile (servizio, stanza, attrezzatura, persona)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "owner_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del proprietario/fornitore"
                },
                "resource_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "enum_values": ["service", "room", "equipment", "person", "vehicle", "table"],
                    "descrizione": "Tipo di risorsa"
                },
                "name": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 200,
                    "descrizione": "Nome della risorsa"
                },
                "description": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 5000,
                    "descrizione": "Descrizione dettagliata"
                },
                "category": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Categoria (es: 'Parrucchiere', 'Sala riunioni')"
                },
                "media": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Immagini della risorsa"
                },
                "location": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Posizione fisica",
                    "schema": {
                        "address": "string",
                        "city": "string",
                        "postal_code": "string",
                        "country": "string",
                        "latitude": "number",
                        "longitude": "number"
                    }
                },
                "pricing": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Configurazione prezzi",
                    "schema": {
                        "base_price": "integer (centesimi)",
                        "currency": "string ISO 4217",
                        "price_type": "per_hour | per_day | per_session | fixed",
                        "min_duration_minutes": "integer",
                        "max_duration_minutes": "integer",
                        "deposit_required": "boolean",
                        "deposit_amount": "integer (centesimi)"
                    }
                },
                "booking_settings": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "{}",
                    "descrizione": "Impostazioni prenotazione",
                    "schema": {
                        "min_advance_hours": "integer (minimo anticipo, default: 1)",
                        "max_advance_days": "integer (massimo anticipo, default: 90)",
                        "buffer_minutes": "integer (pausa tra prenotazioni, default: 0)",
                        "cancellation_policy": "flexible | moderate | strict",
                        "cancellation_hours": "integer (ore per cancellazione gratuita)",
                        "requires_approval": "boolean",
                        "max_bookings_per_user": "integer (null = illimitato)",
                        "allow_recurring": "boolean"
                    }
                },
                "capacity": {
                    "tipo": "integer",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "default": 1,
                    "descrizione": "Capacità (persone, posti, etc.)"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["draft", "active", "paused", "deleted"],
                    "descrizione": "Stato della risorsa"
                },
                "timezone": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "default": "Europe/Rome",
                    "descrizione": "Fuso orario della risorsa"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["AVAILABILITY_RULE", "BOOKING", "AVAILABILITY_EXCEPTION"],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["owner_id", "status"], "tipo": "composite"},
                {"campi": ["resource_type", "status"], "tipo": "composite"},
                {"campi": ["location.city", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: AVAILABILITY_RULE
        # ───────────────────────────────────────────────────────────────────────
        "AVAILABILITY_RULE": {
            "descrizione": "Regola di disponibilità ricorrente (orari settimanali)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "resource_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "BOOKABLE_RESOURCE.id",
                    "descrizione": "ID della risorsa"
                },
                "day_of_week": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_value": 0,
                    "max_value": 6,
                    "descrizione": "Giorno della settimana (0=Lunedì, 6=Domenica)"
                },
                "start_time": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "HH:MM",
                    "descrizione": "Ora inizio disponibilità"
                },
                "end_time": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "HH:MM",
                    "descrizione": "Ora fine disponibilità"
                },
                "is_available": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": True,
                    "descrizione": "Se true = disponibile, se false = blocco"
                },
                "valid_from": {
                    "tipo": "date",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data inizio validità regola"
                },
                "valid_until": {
                    "tipo": "date",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data fine validità regola"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["BOOKABLE_RESOURCE"]
            },
            
            "indici": [
                {"campi": ["resource_id", "day_of_week"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: AVAILABILITY_EXCEPTION
        # ───────────────────────────────────────────────────────────────────────
        "AVAILABILITY_EXCEPTION": {
            "descrizione": "Eccezione alla disponibilità (giorno specifico, ferie, chiusura)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "resource_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "BOOKABLE_RESOURCE.id",
                    "descrizione": "ID della risorsa"
                },
                "date": {
                    "tipo": "date",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data dell'eccezione"
                },
                "exception_type": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "enum_values": ["closed", "modified_hours", "extra_availability"],
                    "descrizione": "Tipo di eccezione"
                },
                "start_time": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "HH:MM",
                    "descrizione": "Ora inizio (solo per modified_hours o extra_availability)"
                },
                "end_time": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "HH:MM",
                    "descrizione": "Ora fine"
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 200,
                    "descrizione": "Motivo dell'eccezione (es: 'Ferie', 'Manutenzione')"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["BOOKABLE_RESOURCE"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["resource_id", "date", "start_time"],
                    "descrizione": "Una sola eccezione per risorsa/data/ora"
                }
            ],
            
            "indici": [
                {"campi": ["resource_id", "date"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: BOOKING
        # ───────────────────────────────────────────────────────────────────────
        "BOOKING": {
            "descrizione": "Prenotazione di una risorsa",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "booking_code": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Codice prenotazione leggibile (es: BK-A1B2C3)"
                },
                "resource_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "BOOKABLE_RESOURCE.id",
                    "descrizione": "ID della risorsa prenotata"
                },
                "booker_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID di chi prenota"
                },
                "provider_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del fornitore (denormalizzato)"
                },
                "start_datetime": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data e ora inizio prenotazione"
                },
                "end_datetime": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Data e ora fine prenotazione"
                },
                "duration_minutes": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Durata in minuti"
                },
                "party_size": {
                    "tipo": "integer",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "default": 1,
                    "descrizione": "Numero di persone/posti"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending",
                    "enum_values": ["pending", "confirmed", "cancelled", "completed", "no_show"],
                    "descrizione": "Stato della prenotazione"
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Dettaglio prezzo",
                    "schema": {
                        "subtotal": "integer (centesimi)",
                        "currency": "string",
                        "deposit_amount": "integer",
                        "total": "integer"
                    }
                },
                "payment_status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending",
                    "enum_values": ["pending", "deposit_paid", "paid", "refunded", "partially_refunded"],
                    "descrizione": "Stato pagamento"
                },
                "notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "descrizione": "Note del cliente"
                },
                "provider_notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "visible": "provider_only",
                    "descrizione": "Note interne del fornitore"
                },
                "recurring_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "ID prenotazione ricorrente (se parte di serie)"
                },
                "reminder_sent": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": False,
                    "descrizione": "Se il promemoria è stato inviato"
                },
                "cancelled_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data cancellazione"
                },
                "cancellation_reason": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 500,
                    "descrizione": "Motivo cancellazione"
                },
                "cancelled_by": {
                    "tipo": "enum",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "enum_values": ["booker", "provider", "system"],
                    "descrizione": "Chi ha cancellato"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["BOOKABLE_RESOURCE", "USER", "USER"]
            },
            
            "stati": ["pending", "confirmed", "cancelled", "completed", "no_show"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["confirmed", "cancelled"],
                    "transizione_a_confirmed": {
                        "condizione": "provider approva (se requires_approval) OPPURE pagamento completato OPPURE auto-conferma",
                        "azione": "Imposta status = 'confirmed'"
                    },
                    "transizione_a_cancelled": {
                        "condizione": "booker annulla OPPURE provider rifiuta OPPURE timeout",
                        "azione": "Imposta status = 'cancelled', cancelled_at = now()"
                    }
                },
                "confirmed": {
                    "prossimi_stati_possibili": ["cancelled", "completed", "no_show"],
                    "transizione_a_cancelled": {
                        "condizione": "booker o provider annullano prima dell'appuntamento",
                        "azione": "Imposta status = 'cancelled', calcola eventuale rimborso"
                    },
                    "transizione_a_completed": {
                        "condizione": "end_datetime passato E servizio erogato",
                        "azione": "Imposta status = 'completed'"
                    },
                    "transizione_a_no_show": {
                        "condizione": "booker non si presenta",
                        "azione": "Imposta status = 'no_show', nessun rimborso"
                    }
                },
                "cancelled": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale"
                },
                "completed": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale"
                },
                "no_show": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale"
                }
            },
            
            "indici": [
                {"campi": ["booking_code"], "tipo": "unique"},
                {"campi": ["resource_id", "start_datetime"], "tipo": "composite"},
                {"campi": ["booker_id", "status"], "tipo": "composite"},
                {"campi": ["provider_id", "start_datetime"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },
    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO BOOKING
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_BOOKABLE_RESOURCE
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_BOOKABLE_RESOURCE": {
            "descrizione": "Crea una nuova risorsa prenotabile",
            "attore": "utente_autenticato",
            
            "input": {
                "resource_type": {"tipo": "enum", "required": True},
                "name": {"tipo": "string", "required": True, "validazione": "max 200 caratteri"},
                "description": {"tipo": "string", "required": False},
                "category": {"tipo": "string", "required": False},
                "media_ids": {"tipo": "array", "required": False},
                "location": {"tipo": "json", "required": False},
                "pricing": {"tipo": "json", "required": True},
                "booking_settings": {"tipo": "json", "required": False},
                "capacity": {"tipo": "integer", "required": False, "default": 1},
                "timezone": {"tipo": "string", "required": False, "default": "Europe/Rome"}
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "input.pricing.base_price >= 0",
                    "errore_se_falsa": "ERROR_INVALID_PRICING"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida e costruisci oggetto media",
                    "dettaglio": """
                        media_array = []
                        SE input.media_ids fornito:
                            PER OGNI media_id IN input.media_ids:
                                media = SELECT * FROM MEDIA_ITEM WHERE id == media_id AND uploader_id == current_user.id
                                SE media esiste:
                                    media_array.push({id: media.id, url: media.url, thumbnail_url: media.thumbnail_url})
                    """
                },
                {
                    "numero": 2,
                    "azione": "Imposta defaults per booking_settings",
                    "dettaglio": """
                        booking_settings = merge(
                            {
                                min_advance_hours: 1,
                                max_advance_days: 90,
                                buffer_minutes: 0,
                                cancellation_policy: 'flexible',
                                cancellation_hours: 24,
                                requires_approval: false,
                                max_bookings_per_user: null,
                                allow_recurring: false
                            },
                            input.booking_settings
                        )
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea BOOKABLE_RESOURCE",
                    "dettaglio": """
                        resource_id = genera_uuid_v4()
                        
                        INSERT INTO BOOKABLE_RESOURCE:
                        - id = resource_id
                        - owner_id = current_user.id
                        - resource_type = input.resource_type
                        - name = input.name
                        - description = input.description
                        - category = input.category
                        - media = media_array
                        - location = input.location
                        - pricing = input.pricing
                        - booking_settings = booking_settings
                        - capacity = input.capacity
                        - status = 'draft'
                        - timezone = input.timezone
                        - created_at = now()
                        - updated_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste BOOKABLE_RESOURCE con id == resource_id"
                }
            ],
            
            "output": {
                "success": True,
                "resource": "oggetto BOOKABLE_RESOURCE creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_INVALID_PRICING": {
                    "codice": "BOOK_002",
                    "messaggio": "Configurazione prezzi non valida",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: SET_AVAILABILITY
        # ───────────────────────────────────────────────────────────────────────
        "SET_AVAILABILITY": {
            "descrizione": "Imposta le regole di disponibilità settimanali",
            "attore": "utente_autenticato",
            
            "input": {
                "resource_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "rules": {
                    "tipo": "array",
                    "required": True,
                    "descrizione": "Array di regole di disponibilità",
                    "schema": {
                        "day_of_week": "0-6 (Lunedì=0)",
                        "start_time": "HH:MM",
                        "end_time": "HH:MM",
                        "is_available": "boolean"
                    }
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BOOKABLE_RESOURCE con id == input.resource_id E owner_id == current_user.id",
                    "errore_se_falsa": "ERROR_RESOURCE_NOT_FOUND"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida regole",
                    "dettaglio": """
                        PER OGNI rule IN input.rules:
                            SE rule.start_time >= rule.end_time:
                                ERRORE: ERROR_INVALID_TIME_RANGE
                            SE rule.day_of_week < 0 OR rule.day_of_week > 6:
                                ERRORE: ERROR_INVALID_DAY
                    """
                },
                {
                    "numero": 2,
                    "azione": "Elimina regole esistenti",
                    "dettaglio": """
                        DELETE FROM AVAILABILITY_RULE WHERE resource_id == input.resource_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Inserisci nuove regole",
                    "dettaglio": """
                        PER OGNI rule IN input.rules:
                            INSERT INTO AVAILABILITY_RULE:
                            - id = genera_uuid_v4()
                            - resource_id = input.resource_id
                            - day_of_week = rule.day_of_week
                            - start_time = rule.start_time
                            - end_time = rule.end_time
                            - is_available = rule.is_available SE fornito ALTRIMENTI true
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "AVAILABILITY_RULE aggiornate per la risorsa"
                }
            ],
            
            "output": {
                "success": True,
                "rules": "array di AVAILABILITY_RULE create"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_010",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_RESOURCE_NOT_FOUND": {
                    "codice": "BOOK_011",
                    "messaggio": "Risorsa non trovata o non di proprietà",
                    "http_status": 404
                },
                "ERROR_INVALID_TIME_RANGE": {
                    "codice": "BOOK_012",
                    "messaggio": "Intervallo orario non valido",
                    "http_status": 400
                },
                "ERROR_INVALID_DAY": {
                    "codice": "BOOK_013",
                    "messaggio": "Giorno della settimana non valido",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_AVAILABLE_SLOTS
        # ───────────────────────────────────────────────────────────────────────
        "GET_AVAILABLE_SLOTS": {
            "descrizione": "Recupera gli slot disponibili per una risorsa in un periodo",
            "attore": "qualsiasi",
            
            "input": {
                "resource_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "start_date": {
                    "tipo": "date",
                    "required": True
                },
                "end_date": {
                    "tipo": "date",
                    "required": True,
                    "descrizione": "Max 31 giorni da start_date"
                },
                "duration_minutes": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Durata desiderata (se non fornita, usa min_duration)"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste BOOKABLE_RESOURCE con id == input.resource_id E status == 'active'",
                    "errore_se_falsa": "ERROR_RESOURCE_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "(input.end_date - input.start_date) <= 31 giorni",
                    "errore_se_falsa": "ERROR_DATE_RANGE_TOO_LARGE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera risorsa e regole",
                    "dettaglio": """
                        resource = SELECT * FROM BOOKABLE_RESOURCE WHERE id == input.resource_id
                        rules = SELECT * FROM AVAILABILITY_RULE WHERE resource_id == input.resource_id
                        exceptions = SELECT * FROM AVAILABILITY_EXCEPTION 
                            WHERE resource_id == input.resource_id 
                            AND date BETWEEN input.start_date AND input.end_date
                    """
                },
                {
                    "numero": 2,
                    "azione": "Recupera prenotazioni esistenti",
                    "dettaglio": """
                        existing_bookings = SELECT start_datetime, end_datetime FROM BOOKING
                        WHERE resource_id == input.resource_id
                        AND status IN ['pending', 'confirmed']
                        AND start_datetime >= input.start_date
                        AND start_datetime <= input.end_date + 1 giorno
                    """
                },
                {
                    "numero": 3,
                    "azione": "Genera slot per ogni giorno",
                    "dettaglio": """
                        duration = input.duration_minutes SE fornito ALTRIMENTI resource.pricing.min_duration_minutes
                        buffer = resource.booking_settings.buffer_minutes
                        available_slots = []
                        
                        PER OGNI giorno DA input.start_date A input.end_date:
                            day_of_week = giorno.getDay()  # 0-6
                            
                            # Controlla eccezioni
                            exception = exceptions.find(e => e.date == giorno)
                            SE exception E exception.exception_type == 'closed':
                                CONTINUA  # Salta questo giorno
                            
                            # Trova regole per questo giorno
                            day_rules = rules.filter(r => r.day_of_week == day_of_week AND r.is_available)
                            
                            # Applica eccezione con orari modificati
                            SE exception E exception.exception_type == 'modified_hours':
                                day_rules = [{
                                    start_time: exception.start_time,
                                    end_time: exception.end_time
                                }]
                            
                            # Genera slot per ogni regola
                            PER OGNI rule IN day_rules:
                                current_time = combina(giorno, rule.start_time)
                                end_time = combina(giorno, rule.end_time)
                                
                                MENTRE current_time + duration <= end_time:
                                    slot_end = current_time + duration
                                    
                                    # Verifica conflitti con prenotazioni esistenti
                                    has_conflict = existing_bookings.some(b =>
                                        (current_time < b.end_datetime AND slot_end > b.start_datetime)
                                    )
                                    
                                    # Verifica anticipo minimo
                                    min_advance = now() + resource.booking_settings.min_advance_hours ore
                                    
                                    SE NOT has_conflict E current_time >= min_advance:
                                        available_slots.push({
                                            start: current_time,
                                            end: slot_end,
                                            duration_minutes: duration
                                        })
                                    
                                    current_time = slot_end + buffer
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "slots": "array di slot disponibili raggruppati per giorno",
                "resource": "info base della risorsa"
            },
            
            "errori": {
                "ERROR_RESOURCE_NOT_FOUND": {
                    "codice": "BOOK_020",
                    "messaggio": "Risorsa non trovata",
                    "http_status": 404
                },
                "ERROR_DATE_RANGE_TOO_LARGE": {
                    "codice": "BOOK_021",
                    "messaggio": "Intervallo date troppo ampio (max 31 giorni)",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_BOOKING
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_BOOKING": {
            "descrizione": "Crea una nuova prenotazione",
            "attore": "utente_autenticato",
            
            "input": {
                "resource_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "start_datetime": {
                    "tipo": "timestamp",
                    "required": True
                },
                "end_datetime": {
                    "tipo": "timestamp",
                    "required": True
                },
                "party_size": {
                    "tipo": "integer",
                    "required": False,
                    "default": 1
                },
                "notes": {
                    "tipo": "string",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BOOKABLE_RESOURCE con id == input.resource_id E status == 'active'",
                    "errore_se_falsa": "ERROR_RESOURCE_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "input.start_datetime < input.end_datetime",
                    "errore_se_falsa": "ERROR_INVALID_TIME_RANGE"
                },
                {
                    "id": "PRE_4",
                    "condizione": "input.start_datetime >= now() + min_advance_hours",
                    "errore_se_falsa": "ERROR_TOO_SOON"
                },
                {
                    "id": "PRE_5",
                    "condizione": "input.start_datetime <= now() + max_advance_days giorni",
                    "errore_se_falsa": "ERROR_TOO_FAR_AHEAD"
                },
                {
                    "id": "PRE_6",
                    "condizione": "input.party_size <= resource.capacity",
                    "errore_se_falsa": "ERROR_EXCEEDS_CAPACITY"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica disponibilità slot",
                    "dettaglio": """
                        resource = SELECT * FROM BOOKABLE_RESOURCE WHERE id == input.resource_id
                        
                        # Verifica che lo slot sia effettivamente disponibile
                        slots = esegui GET_AVAILABLE_SLOTS(
                            resource_id = input.resource_id,
                            start_date = input.start_datetime.date(),
                            end_date = input.start_datetime.date()
                        )
                        
                        slot_available = slots.some(s => 
                            s.start == input.start_datetime AND s.end == input.end_datetime
                        )
                        
                        SE NOT slot_available:
                            ERRORE: ERROR_SLOT_NOT_AVAILABLE
                    """
                },
                {
                    "numero": 2,
                    "azione": "Verifica limiti prenotazioni utente",
                    "dettaglio": """
                        SE resource.booking_settings.max_bookings_per_user != null:
                            user_bookings = SELECT COUNT(*) FROM BOOKING
                            WHERE booker_id == current_user.id
                            AND resource_id == input.resource_id
                            AND status IN ['pending', 'confirmed']
                            AND start_datetime >= now()
                            
                            SE user_bookings >= resource.booking_settings.max_bookings_per_user:
                                ERRORE: ERROR_MAX_BOOKINGS_REACHED
                    """
                },
                {
                    "numero": 3,
                    "azione": "Calcola prezzo",
                    "dettaglio": """
                        duration_minutes = (input.end_datetime - input.start_datetime) in minuti
                        
                        SE resource.pricing.price_type == 'per_hour':
                            subtotal = (duration_minutes / 60) * resource.pricing.base_price
                        ALTRIMENTI SE resource.pricing.price_type == 'per_session':
                            subtotal = resource.pricing.base_price
                        ALTRIMENTI:
                            subtotal = resource.pricing.base_price
                        
                        deposit_amount = 0
                        SE resource.pricing.deposit_required:
                            deposit_amount = resource.pricing.deposit_amount
                        
                        price = {
                            subtotal: subtotal,
                            currency: resource.pricing.currency,
                            deposit_amount: deposit_amount,
                            total: subtotal
                        }
                    """
                },
                {
                    "numero": 4,
                    "azione": "Genera codice prenotazione",
                    "dettaglio": """
                        booking_code = "BK-" + genera_codice_alfanumerico(6)  # es: BK-A1B2C3
                        # Verifica unicità
                        MENTRE EXISTS BOOKING WHERE booking_code == booking_code:
                            booking_code = "BK-" + genera_codice_alfanumerico(6)
                    """
                },
                {
                    "numero": 5,
                    "azione": "Determina stato iniziale",
                    "dettaglio": """
                        SE resource.booking_settings.requires_approval:
                            initial_status = 'pending'
                        ALTRIMENTI:
                            initial_status = 'confirmed'
                    """
                },
                {
                    "numero": 6,
                    "azione": "Crea BOOKING",
                    "dettaglio": """
                        booking_id = genera_uuid_v4()
                        
                        INSERT INTO BOOKING:
                        - id = booking_id
                        - booking_code = booking_code
                        - resource_id = input.resource_id
                        - booker_id = current_user.id
                        - provider_id = resource.owner_id
                        - start_datetime = input.start_datetime
                        - end_datetime = input.end_datetime
                        - duration_minutes = duration_minutes
                        - party_size = input.party_size
                        - status = initial_status
                        - price = price
                        - payment_status = 'pending'
                        - notes = input.notes
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 7,
                    "azione": "Notifica provider",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = resource.owner_id,
                            type = SE initial_status == 'pending': 'booking_request' ALTRIMENTI 'new_booking',
                            source_user_id = current_user.id,
                            target_type = 'booking',
                            target_id = booking_id,
                            metadata = {
                                booking_code: booking_code,
                                resource_name: resource.name,
                                start_datetime: input.start_datetime
                            }
                        )
                    """
                },
                {
                    "numero": 8,
                    "azione": "Invia email conferma",
                    "dettaglio": """
                        invia_email(
                            to = current_user.email,
                            template = 'booking_confirmation',
                            data = {
                                booking: booking,
                                resource: resource
                            }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste BOOKING con id == booking_id"
                }
            ],
            
            "output": {
                "success": True,
                "booking": "oggetto BOOKING creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_RESOURCE_NOT_FOUND": {
                    "codice": "BOOK_031",
                    "messaggio": "Risorsa non trovata o non attiva",
                    "http_status": 404
                },
                "ERROR_INVALID_TIME_RANGE": {
                    "codice": "BOOK_032",
                    "messaggio": "Intervallo orario non valido",
                    "http_status": 400
                },
                "ERROR_TOO_SOON": {
                    "codice": "BOOK_033",
                    "messaggio": "Prenotazione troppo ravvicinata",
                    "http_status": 400
                },
                "ERROR_TOO_FAR_AHEAD": {
                    "codice": "BOOK_034",
                    "messaggio": "Prenotazione troppo in anticipo",
                    "http_status": 400
                },
                "ERROR_EXCEEDS_CAPACITY": {
                    "codice": "BOOK_035",
                    "messaggio": "Numero di persone supera la capacità",
                    "http_status": 400
                },
                "ERROR_SLOT_NOT_AVAILABLE": {
                    "codice": "BOOK_036",
                    "messaggio": "Slot non disponibile",
                    "http_status": 409
                },
                "ERROR_MAX_BOOKINGS_REACHED": {
                    "codice": "BOOK_037",
                    "messaggio": "Numero massimo di prenotazioni raggiunto",
                    "http_status": 429
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CANCEL_BOOKING
        # ───────────────────────────────────────────────────────────────────────
        "CANCEL_BOOKING": {
            "descrizione": "Cancella una prenotazione",
            "attore": "utente_autenticato",
            
            "input": {
                "booking_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "reason": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 500 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BOOKING con id == input.booking_id",
                    "errore_se_falsa": "ERROR_BOOKING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id IN [BOOKING.booker_id, BOOKING.provider_id]",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "BOOKING.status IN ['pending', 'confirmed']",
                    "errore_se_falsa": "ERROR_CANNOT_CANCEL"
                },
                {
                    "id": "PRE_5",
                    "condizione": "BOOKING.start_datetime > now()",
                    "errore_se_falsa": "ERROR_ALREADY_STARTED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera booking e risorsa",
                    "dettaglio": """
                        booking = SELECT * FROM BOOKING WHERE id == input.booking_id
                        resource = SELECT * FROM BOOKABLE_RESOURCE WHERE id == booking.resource_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Calcola politica di rimborso",
                    "dettaglio": """
                        cancelled_by = SE current_user.id == booking.booker_id: 'booker' ALTRIMENTI 'provider'
                        hours_until_booking = (booking.start_datetime - now()) in ore
                        
                        SE cancelled_by == 'provider':
                            # Provider cancella: rimborso completo sempre
                            refund_percentage = 100
                        ALTRIMENTI:
                            # Booker cancella: dipende dalla policy
                            SE resource.booking_settings.cancellation_policy == 'flexible':
                                SE hours_until_booking >= resource.booking_settings.cancellation_hours:
                                    refund_percentage = 100
                                ALTRIMENTI:
                                    refund_percentage = 50
                            ALTRIMENTI SE resource.booking_settings.cancellation_policy == 'moderate':
                                SE hours_until_booking >= resource.booking_settings.cancellation_hours:
                                    refund_percentage = 100
                                ALTRIMENTI SE hours_until_booking >= 24:
                                    refund_percentage = 50
                                ALTRIMENTI:
                                    refund_percentage = 0
                            ALTRIMENTI:  # strict
                                SE hours_until_booking >= resource.booking_settings.cancellation_hours:
                                    refund_percentage = 50
                                ALTRIMENTI:
                                    refund_percentage = 0
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna BOOKING",
                    "dettaglio": """
                        UPDATE BOOKING SET
                        - status = 'cancelled'
                        - cancelled_at = now()
                        - cancelled_by = cancelled_by
                        - cancellation_reason = input.reason
                        - updated_at = now()
                        WHERE id == input.booking_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Processa rimborso se necessario",
                    "dettaglio": """
                        SE booking.payment_status IN ['deposit_paid', 'paid']:
                            refund_amount = (booking.price.total * refund_percentage) / 100
                            
                            SE refund_amount > 0:
                                # Processa rimborso tramite payment provider
                                processa_rimborso(booking.id, refund_amount)
                                
                                UPDATE BOOKING SET
                                - payment_status = SE refund_percentage == 100: 'refunded' ALTRIMENTI 'partially_refunded'
                                WHERE id == input.booking_id
                    """
                },
                {
                    "numero": 5,
                    "azione": "Notifica altra parte",
                    "dettaglio": """
                        recipient_id = SE cancelled_by == 'booker': booking.provider_id ALTRIMENTI booking.booker_id
                        
                        crea_notifica(
                            recipient_id = recipient_id,
                            type = 'booking_cancelled',
                            source_user_id = current_user.id,
                            target_type = 'booking',
                            target_id = booking.id,
                            metadata = {
                                booking_code: booking.booking_code,
                                reason: input.reason,
                                refund_percentage: refund_percentage
                            }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "BOOKING.status == 'cancelled'"
                }
            ],
            
            "output": {
                "success": True,
                "booking": "oggetto BOOKING aggiornato",
                "refund_percentage": "percentuale rimborso",
                "refund_amount": "importo rimborso"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_BOOKING_NOT_FOUND": {
                    "codice": "BOOK_041",
                    "messaggio": "Prenotazione non trovata",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "BOOK_042",
                    "messaggio": "Non autorizzato a cancellare questa prenotazione",
                    "http_status": 403
                },
                "ERROR_CANNOT_CANCEL": {
                    "codice": "BOOK_043",
                    "messaggio": "Prenotazione non può essere cancellata in questo stato",
                    "http_status": 400
                },
                "ERROR_ALREADY_STARTED": {
                    "codice": "BOOK_044",
                    "messaggio": "Prenotazione già iniziata",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CONFIRM_BOOKING
        # ───────────────────────────────────────────────────────────────────────
        "CONFIRM_BOOKING": {
            "descrizione": "Conferma una prenotazione in attesa (provider)",
            "attore": "utente_autenticato",
            
            "input": {
                "booking_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste BOOKING con id == input.booking_id E status == 'pending'",
                    "errore_se_falsa": "ERROR_BOOKING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "BOOKING.provider_id == current_user.id",
                    "errore_se_falsa": "ERROR_NOT_PROVIDER"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Aggiorna BOOKING",
                    "dettaglio": """
                        UPDATE BOOKING SET
                        - status = 'confirmed'
                        - updated_at = now()
                        WHERE id == input.booking_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Notifica booker",
                    "dettaglio": """
                        booking = SELECT * FROM BOOKING WHERE id == input.booking_id
                        
                        crea_notifica(
                            recipient_id = booking.booker_id,
                            type = 'booking_confirmed',
                            source_user_id = current_user.id,
                            target_type = 'booking',
                            target_id = booking.id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "BOOKING.status == 'confirmed'"
                }
            ],
            
            "output": {
                "success": True,
                "booking": "oggetto BOOKING aggiornato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_BOOKING_NOT_FOUND": {
                    "codice": "BOOK_051",
                    "messaggio": "Prenotazione non trovata o non in attesa",
                    "http_status": 404
                },
                "ERROR_NOT_PROVIDER": {
                    "codice": "BOOK_052",
                    "messaggio": "Solo il fornitore può confermare",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_MY_BOOKINGS
        # ───────────────────────────────────────────────────────────────────────
        "GET_MY_BOOKINGS": {
            "descrizione": "Recupera le prenotazioni dell'utente (come booker o provider)",
            "attore": "utente_autenticato",
            
            "input": {
                "role": {
                    "tipo": "enum",
                    "required": False,
                    "default": "booker",
                    "validazione": "booker | provider"
                },
                "status": {
                    "tipo": "enum",
                    "required": False,
                    "validazione": "pending | confirmed | cancelled | completed | all"
                },
                "start_date": {
                    "tipo": "date",
                    "required": False
                },
                "end_date": {
                    "tipo": "date",
                    "required": False
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query",
                    "dettaglio": """
                        query = SELECT b.*, r.name as resource_name, r.media, u.username
                        FROM BOOKING b
                        JOIN BOOKABLE_RESOURCE r ON b.resource_id = r.id
                        
                        SE input.role == 'booker':
                            query += JOIN USER u ON b.provider_id = u.id
                            query += WHERE b.booker_id == current_user.id
                        ALTRIMENTI:
                            query += JOIN USER u ON b.booker_id = u.id
                            query += WHERE b.provider_id == current_user.id
                        
                        SE input.status fornito E input.status != 'all':
                            query += AND b.status == input.status
                        
                        SE input.start_date fornito:
                            query += AND b.start_datetime >= input.start_date
                        
                        SE input.end_date fornito:
                            query += AND b.start_datetime <= input.end_date + 1 giorno
                        
                        query += ORDER BY b.start_datetime DESC
                        query += LIMIT input.limit
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "bookings": "array di prenotazioni",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "BOOK_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo BOOKING"
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO: COMMERCE
# VERSIONE: 1.0
# DESCRIZIONE: E-commerce completo con prodotti, carrello, checkout, ordini
# DIPENDENZE: IDENTITY, MESSAGING
# ═══════════════════════════════════════════════════════════════════════════════

MODULO_COMMERCE = {
    "nome": "COMMERCE",
    "versione": "1.0",
    "descrizione": "E-commerce completo con prodotti, carrello, checkout, ordini",
    "dipendenze": ["IDENTITY", "MESSAGING"],
    
    # ═══════════════════════════════════════════════════════════════════════════
    # ENTITÀ
    # ═══════════════════════════════════════════════════════════════════════════
    
    "entita": {
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: PRODUCT
        # ───────────────────────────────────────────────────────────────────────
        "PRODUCT": {
            "descrizione": "Prodotto in vendita",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "seller_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID del venditore"
                },
                "sku": {
                    "tipo": "string",
                    "required": False,
                    "unique": True,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Stock Keeping Unit (codice prodotto)"
                },
                "name": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 300,
                    "descrizione": "Nome prodotto"
                },
                "slug": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": True,
                    "format": "slug",
                    "descrizione": "URL-friendly name"
                },
                "description": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 10000,
                    "descrizione": "Descrizione dettagliata (supporta HTML/Markdown)"
                },
                "short_description": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 500,
                    "descrizione": "Descrizione breve"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "riferimento": "PRODUCT_CATEGORY.id",
                    "descrizione": "ID categoria"
                },
                "brand": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "Marca/Brand"
                },
                "media": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Immagini e video del prodotto",
                    "schema": {
                        "tipo": "array",
                        "max_items": 30,
                        "items": {
                            "id": "uuid",
                            "url": "string",
                            "thumbnail_url": "string",
                            "type": "image | video",
                            "alt_text": "string",
                            "order": "integer",
                            "is_primary": "boolean"
                        }
                    }
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Informazioni prezzo",
                    "schema": {
                        "amount": "integer (centesimi)",
                        "currency": "string ISO 4217",
                        "compare_at_amount": "integer (prezzo originale se scontato)",
                        "cost_per_item": "integer (costo per il venditore, privato)"
                    }
                },
                "inventory": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Gestione inventario",
                    "schema": {
                        "track_quantity": "boolean",
                        "quantity": "integer",
                        "allow_backorder": "boolean",
                        "low_stock_threshold": "integer"
                    }
                },
                "shipping": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Info spedizione",
                    "schema": {
                        "weight_grams": "integer",
                        "dimensions": {
                            "length_cm": "number",
                            "width_cm": "number",
                            "height_cm": "number"
                        },
                        "requires_shipping": "boolean",
                        "is_fragile": "boolean"
                    }
                },
                "attributes": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Attributi personalizzati (es: colore, materiale)"
                },
                "tags": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Array di tag per ricerca"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "draft",
                    "enum_values": ["draft", "active", "archived", "deleted"],
                    "descrizione": "Stato del prodotto"
                },
                "visibility": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "visible",
                    "enum_values": ["visible", "hidden", "catalog_only"],
                    "descrizione": "Visibilità prodotto"
                },
                "is_digital": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se è un prodotto digitale"
                },
                "digital_file_url": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "visible": "owner_only",
                    "descrizione": "URL file digitale (solo per is_digital=true)"
                },
                "seo": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Meta SEO",
                    "schema": {
                        "title": "string (max 70)",
                        "description": "string (max 160)",
                        "keywords": "array di string"
                    }
                },
                "rating_average": {
                    "tipo": "number",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Media recensioni"
                },
                "rating_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero recensioni"
                },
                "sales_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero vendite"
                },
                "views_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero visualizzazioni"
                },
                "published_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data pubblicazione"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["PRODUCT_VARIANT", "PRODUCT_REVIEW"],
                "appartiene_a": ["USER", "PRODUCT_CATEGORY"]
            },
            
            "indici": [
                {"campi": ["slug"], "tipo": "unique"},
                {"campi": ["seller_id", "status"], "tipo": "composite"},
                {"campi": ["category_id", "status"], "tipo": "composite"},
                {"campi": ["status", "published_at"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: PRODUCT_VARIANT
        # ───────────────────────────────────────────────────────────────────────
        "PRODUCT_VARIANT": {
            "descrizione": "Variante di un prodotto (taglia, colore, etc.)",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "product_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT.id",
                    "descrizione": "ID prodotto padre"
                },
                "sku": {
                    "tipo": "string",
                    "required": False,
                    "unique": True,
                    "generated": False,
                    "editable": True,
                    "max_length": 100,
                    "descrizione": "SKU variante"
                },
                "name": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 200,
                    "descrizione": "Nome variante (es: 'Rosso - XL')"
                },
                "options": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Opzioni della variante",
                    "esempio": {"Colore": "Rosso", "Taglia": "XL"}
                },
                "price": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Prezzo variante (se diverso dal prodotto)"
                },
                "inventory": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "schema": {
                        "quantity": "integer",
                        "allow_backorder": "boolean"
                    }
                },
                "image_url": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "url",
                    "descrizione": "Immagine specifica della variante"
                },
                "weight_grams": {
                    "tipo": "integer",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Peso variante (se diverso)"
                },
                "is_default": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": False,
                    "descrizione": "Se è la variante di default"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["active", "inactive"],
                    "descrizione": "Stato variante"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["PRODUCT"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["product_id", "options"],
                    "descrizione": "Combinazione opzioni unica per prodotto"
                }
            ],
            
            "indici": [
                {"campi": ["product_id", "status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: CART
        # ───────────────────────────────────────────────────────────────────────
        "CART": {
            "descrizione": "Carrello dell'utente",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "user_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID utente (null per carrello anonimo)"
                },
                "session_id": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "ID sessione (per carrelli anonimi)"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "active",
                    "enum_values": ["active", "abandoned", "converted"],
                    "descrizione": "Stato del carrello"
                },
                "currency": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "EUR",
                    "descrizione": "Valuta del carrello"
                },
                "subtotal": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Totale prodotti in centesimi"
                },
                "discount_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Sconto totale in centesimi"
                },
                "coupon_code": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Codice coupon applicato"
                },
                "shipping_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Costo spedizione in centesimi"
                },
                "tax_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Totale tasse in centesimi"
                },
                "total": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Totale finale in centesimi"
                },
                "items_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di items nel carrello"
                },
                "notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 1000,
                    "descrizione": "Note del cliente"
                },
                "expires_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Scadenza carrello (per carrelli anonimi)"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["CART_ITEM"],
                "appartiene_a": ["USER"]
            },
            
            "indici": [
                {"campi": ["user_id", "status"], "tipo": "composite"},
                {"campi": ["session_id"], "tipo": "standard"},
                {"campi": ["expires_at"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: CART_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "CART_ITEM": {
            "descrizione": "Elemento nel carrello",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "cart_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "CART.id",
                    "descrizione": "ID carrello"
                },
                "product_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT.id",
                    "descrizione": "ID prodotto"
                },
                "variant_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT_VARIANT.id",
                    "descrizione": "ID variante (se applicabile)"
                },
                "quantity": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_value": 1,
                    "descrizione": "Quantità"
                },
                "unit_price": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Prezzo unitario al momento dell'aggiunta (centesimi)"
                },
                "total_price": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Prezzo totale (unit_price * quantity)"
                },
                "product_snapshot": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Snapshot dati prodotto al momento dell'aggiunta",
                    "schema": {
                        "name": "string",
                        "sku": "string",
                        "image_url": "string",
                        "variant_name": "string"
                    }
                },
                "added_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data aggiunta al carrello"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["CART", "PRODUCT", "PRODUCT_VARIANT"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["cart_id", "product_id", "variant_id"],
                    "descrizione": "Uno stesso prodotto/variante una sola volta per carrello"
                }
            ],
            
            "indici": [
                {"campi": ["cart_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: ORDER
        # ───────────────────────────────────────────────────────────────────────
        "ORDER": {
            "descrizione": "Ordine completato",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "order_number": {
                    "tipo": "string",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Numero ordine leggibile (es: ORD-2024-000123)"
                },
                "customer_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID cliente"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending",
                    "enum_values": [
                        "pending",
                        "confirmed",
                        "processing",
                        "shipped",
                        "delivered",
                        "completed",
                        "cancelled",
                        "refunded"
                    ],
                    "descrizione": "Stato ordine"
                },
                "payment_status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "pending",
                    "enum_values": ["pending", "paid", "failed", "refunded", "partially_refunded"],
                    "descrizione": "Stato pagamento"
                },
                "fulfillment_status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "unfulfilled",
                    "enum_values": ["unfulfilled", "partial", "fulfilled"],
                    "descrizione": "Stato evasione"
                },
                "currency": {
                    "tipo": "string",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Valuta"
                },
                "subtotal": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Subtotale prodotti (centesimi)"
                },
                "discount_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Sconto totale"
                },
                "coupon_code": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Coupon utilizzato"
                },
                "shipping_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Costo spedizione"
                },
                "tax_amount": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Tasse"
                },
                "total": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Totale finale"
                },
                "billing_address": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Indirizzo fatturazione",
                    "schema": {
                        "first_name": "string",
                        "last_name": "string",
                        "company": "string (opzionale)",
                        "address_line_1": "string",
                        "address_line_2": "string",
                        "city": "string",
                        "state": "string",
                        "postal_code": "string",
                        "country": "string",
                        "phone": "string",
                        "email": "string"
                    }
                },
                "shipping_address": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Indirizzo spedizione (stesso schema di billing)"
                },
                "shipping_method": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Metodo spedizione scelto",
                    "schema": {
                        "id": "string",
                        "name": "string",
                        "carrier": "string",
                        "estimated_days": "integer"
                    }
                },
                "payment_method": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Metodo pagamento",
                    "schema": {
                        "type": "card | paypal | bank_transfer | cod",
                        "last_four": "string (ultime 4 cifre carta)",
                        "brand": "string (Visa, Mastercard, etc.)"
                    }
                },
                "payment_id": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "visible": "system_only",
                    "descrizione": "ID transazione payment gateway"
                },
                "notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "max_length": 1000,
                    "descrizione": "Note del cliente"
                },
                "internal_notes": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 2000,
                    "visible": "admin_only",
                    "descrizione": "Note interne"
                },
                "tracking_number": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Numero tracking spedizione"
                },
                "tracking_url": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "format": "url",
                    "descrizione": "URL tracking"
                },
                "paid_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data pagamento"
                },
                "shipped_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data spedizione"
                },
                "delivered_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data consegna"
                },
                "cancelled_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data cancellazione"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": ["ORDER_ITEM"],
                "appartiene_a": ["USER"]
            },
            
            "stati": ["pending", "confirmed", "processing", "shipped", "delivered", "completed", "cancelled", "refunded"],
            
            "transizioni_stato": {
                "pending": {
                    "prossimi_stati_possibili": ["confirmed", "cancelled"],
                    "transizione_a_confirmed": {
                        "condizione": "pagamento confermato",
                        "azione": "Imposta status = 'confirmed', payment_status = 'paid', paid_at = now()"
                    }
                },
                "confirmed": {
                    "prossimi_stati_possibili": ["processing", "cancelled", "refunded"],
                    "transizione_a_processing": {
                        "condizione": "venditore inizia preparazione",
                        "azione": "Imposta status = 'processing'"
                    }
                },
                "processing": {
                    "prossimi_stati_possibili": ["shipped", "cancelled", "refunded"],
                    "transizione_a_shipped": {
                        "condizione": "ordine spedito",
                        "azione": "Imposta status = 'shipped', shipped_at = now(), fulfillment_status = 'fulfilled'"
                    }
                },
                "shipped": {
                    "prossimi_stati_possibili": ["delivered", "refunded"],
                    "transizione_a_delivered": {
                        "condizione": "consegna confermata",
                        "azione": "Imposta status = 'delivered', delivered_at = now()"
                    }
                },
                "delivered": {
                    "prossimi_stati_possibili": ["completed", "refunded"],
                    "transizione_a_completed": {
                        "condizione": "periodo reso scaduto o cliente conferma",
                        "azione": "Imposta status = 'completed'"
                    }
                },
                "completed": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale positivo"
                },
                "cancelled": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale negativo"
                },
                "refunded": {
                    "prossimi_stati_possibili": [],
                    "note": "Stato finale con rimborso"
                }
            },
            
            "indici": [
                {"campi": ["order_number"], "tipo": "unique"},
                {"campi": ["customer_id", "created_at"], "tipo": "composite"},
                {"campi": ["status", "created_at"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: ORDER_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "ORDER_ITEM": {
            "descrizione": "Elemento di un ordine",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "order_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "ORDER.id",
                    "descrizione": "ID ordine"
                },
                "product_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT.id",
                    "descrizione": "ID prodotto"
                },
                "variant_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT_VARIANT.id",
                    "descrizione": "ID variante"
                },
                "seller_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID venditore (denormalizzato)"
                },
                "quantity": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "min_value": 1,
                    "descrizione": "Quantità ordinata"
                },
                "unit_price": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "descrizione": "Prezzo unitario al momento dell'ordine"
                },
                "total_price": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Prezzo totale linea"
                },
                "product_snapshot": {
                    "tipo": "json",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Snapshot completo prodotto al momento dell'ordine",
                    "schema": {
                        "name": "string",
                        "sku": "string",
                        "variant_name": "string",
                        "image_url": "string",
                        "attributes": "json"
                    }
                },
                "fulfillment_status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "unfulfilled",
                    "enum_values": ["unfulfilled", "fulfilled", "returned"],
                    "descrizione": "Stato evasione item"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["ORDER", "PRODUCT", "PRODUCT_VARIANT", "USER"]
            },
            
            "indici": [
                {"campi": ["order_id"], "tipo": "standard"},
                {"campi": ["seller_id", "fulfillment_status"], "tipo": "composite"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # ENTITÀ: PRODUCT_REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "PRODUCT_REVIEW": {
            "descrizione": "Recensione prodotto",
            
            "attributi": {
                "id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": True,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Identificativo univoco"
                },
                "product_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "PRODUCT.id",
                    "descrizione": "ID prodotto recensito"
                },
                "order_item_id": {
                    "tipo": "uuid",
                    "required": False,
                    "unique": True,
                    "generated": False,
                    "editable": False,
                    "riferimento": "ORDER_ITEM.id",
                    "descrizione": "ID order item (per verificare acquisto)"
                },
                "reviewer_id": {
                    "tipo": "uuid",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": False,
                    "riferimento": "USER.id",
                    "descrizione": "ID recensore"
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "min_value": 1,
                    "max_value": 5,
                    "descrizione": "Valutazione 1-5"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 150,
                    "descrizione": "Titolo recensione"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 5000,
                    "descrizione": "Testo recensione"
                },
                "media": {
                    "tipo": "json",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "descrizione": "Foto/video allegati"
                },
                "is_verified_purchase": {
                    "tipo": "boolean",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Se il recensore ha effettivamente acquistato"
                },
                "helpful_count": {
                    "tipo": "integer",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "default": 0,
                    "descrizione": "Numero di 'utile'"
                },
                "seller_response": {
                    "tipo": "string",
                    "required": False,
                    "unique": False,
                    "generated": False,
                    "editable": True,
                    "max_length": 2000,
                    "descrizione": "Risposta del venditore"
                },
                "seller_response_at": {
                    "tipo": "timestamp",
                    "required": False,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data risposta venditore"
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": True,
                    "default": "published",
                    "enum_values": ["pending", "published", "hidden", "deleted"],
                    "descrizione": "Stato recensione"
                },
                "created_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data creazione"
                },
                "updated_at": {
                    "tipo": "timestamp",
                    "required": True,
                    "unique": False,
                    "generated": True,
                    "editable": False,
                    "descrizione": "Data ultimo aggiornamento"
                }
            },
            
            "relazioni": {
                "ha_uno": [],
                "ha_molti": [],
                "appartiene_a": ["PRODUCT", "ORDER_ITEM", "USER"]
            },
            
            "vincoli": [
                {
                    "tipo": "unique_composite",
                    "campi": ["product_id", "reviewer_id"],
                    "descrizione": "Un utente può recensire un prodotto una sola volta"
                }
            ],
            
            "indici": [
                {"campi": ["product_id", "status", "created_at"], "tipo": "composite"},
                {"campi": ["reviewer_id"], "tipo": "standard"}
            ],
            
            "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases"
        }
    },

    
    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO MARKETPLACE
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_LISTING": {
            "descrizione": "Crea un nuovo annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_type": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "product | service | job | housing | vehicle"
                },
                "title": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "min 5, max 200 caratteri"
                },
                "description": {
                    "tipo": "string",
                    "required": True,
                    "validazione": "min 20, max 10000 caratteri"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "subcategory_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "price": {
                    "tipo": "json",
                    "required": True,
                    "validazione": "{ amount, currency, type }"
                },
                "condition": {
                    "tipo": "enum",
                    "required": False,
                    "validazione": "new | like_new | good | fair | for_parts"
                },
                "media_ids": {
                    "tipo": "array",
                    "required": False,
                    "validazione": "array di UUID media, max 20"
                },
                "location": {
                    "tipo": "json",
                    "required": False
                },
                "attributes": {
                    "tipo": "json",
                    "required": False
                },
                "shipping_options": {
                    "tipo": "json",
                    "required": False
                },
                "publish": {
                    "tipo": "boolean",
                    "required": False,
                    "default": False,
                    "descrizione": "Se true, pubblica immediatamente"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "USER.status == 'active'",
                    "errore_se_falsa": "ERROR_ACCOUNT_NOT_ACTIVE"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Esiste CATEGORY con id == input.category_id E is_active == true",
                    "errore_se_falsa": "ERROR_INVALID_CATEGORY"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.subcategory_id: CATEGORY.parent_id == input.category_id",
                    "errore_se_falsa": "ERROR_INVALID_SUBCATEGORY"
                },
                {
                    "id": "PRE_5",
                    "condizione": "input.price.amount >= 0",
                    "errore_se_falsa": "ERROR_INVALID_PRICE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida categoria e recupera schema attributi",
                    "dettaglio": """
                        category = SELECT * FROM CATEGORY WHERE id == input.category_id
                        
                        SE category.attribute_schema definito:
                            valida input.attributes contro category.attribute_schema
                            SE validazione fallisce:
                                ERRORE: ERROR_INVALID_ATTRIBUTES
                    """
                },
                {
                    "numero": 2,
                    "azione": "Costruisci array media",
                    "dettaglio": """
                        media_array = []
                        SE input.media_ids fornito:
                            order = 0
                            PER OGNI media_id IN input.media_ids:
                                media = SELECT * FROM MEDIA_ITEM WHERE id == media_id
                                SE media non esiste O media.uploader_id != current_user.id:
                                    ERRORE: ERROR_INVALID_MEDIA
                                
                                media_array.append({
                                    id: media.id,
                                    type: media.file_type,
                                    url: media.url,
                                    thumbnail_url: media.thumbnail_url,
                                    order: order,
                                    is_cover: order == 0
                                })
                                order = order + 1
                    """
                },
                {
                    "numero": 3,
                    "azione": "Determina stato iniziale",
                    "dettaglio": """
                        SE input.publish == true:
                            SE moderazione_abilitata:
                                initial_status = 'pending_review'
                            ALTRIMENTI:
                                initial_status = 'active'
                        ALTRIMENTI:
                            initial_status = 'draft'
                    """
                },
                {
                    "numero": 4,
                    "azione": "Calcola data scadenza",
                    "dettaglio": """
                        SE initial_status == 'active':
                            expires_at = now() + 30 giorni
                            published_at = now()
                        ALTRIMENTI:
                            expires_at = null
                            published_at = null
                    """
                },
                {
                    "numero": 5,
                    "azione": "Crea record LISTING",
                    "dettaglio": """
                        listing_id = genera_uuid_v4()
                        
                        INSERT INTO LISTING:
                        - id = listing_id
                        - seller_id = current_user.id
                        - listing_type = input.listing_type
                        - title = input.title
                        - description = input.description
                        - category_id = input.category_id
                        - subcategory_id = input.subcategory_id
                        - price = input.price
                        - condition = input.condition
                        - media = media_array
                        - location = input.location
                        - attributes = input.attributes
                        - shipping_options = input.shipping_options
                        - status = initial_status
                        - visibility = 'public'
                        - views_count = 0
                        - favorites_count = 0
                        - inquiries_count = 0
                        - expires_at = expires_at
                        - published_at = published_at
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 6,
                    "azione": "Aggiorna contatori categoria",
                    "dettaglio": """
                        SE initial_status == 'active':
                            UPDATE CATEGORY SET listing_count = listing_count + 1
                            WHERE id == input.category_id
                            
                            SE input.subcategory_id:
                                UPDATE CATEGORY SET listing_count = listing_count + 1
                                WHERE id == input.subcategory_id
                    """
                },
                {
                    "numero": 7,
                    "azione": "Aggiorna profilo venditore",
                    "dettaglio": """
                        SE initial_status == 'active':
                            UPDATE SELLER_PROFILE SET 
                            - active_listings_count = active_listings_count + 1
                            - updated_at = now()
                            WHERE user_id == current_user.id
                    """
                },
                {
                    "numero": 8,
                    "azione": "Crea SELLER_PROFILE se non esiste",
                    "dettaglio": """
                        SE NOT EXISTS SELLER_PROFILE WHERE user_id == current_user.id:
                            INSERT INTO SELLER_PROFILE:
                            - user_id = current_user.id
                            - member_since = now()
                            - active_listings_count = SE initial_status == 'active' ALLORA 1 ALTRIMENTI 0
                            - updated_at = now()
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste LISTING con id == listing_id"
                }
            ],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_001",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_ACCOUNT_NOT_ACTIVE": {
                    "codice": "MARKET_002",
                    "messaggio": "Account non attivo",
                    "http_status": 403
                },
                "ERROR_INVALID_CATEGORY": {
                    "codice": "MARKET_003",
                    "messaggio": "Categoria non valida",
                    "http_status": 400
                },
                "ERROR_INVALID_SUBCATEGORY": {
                    "codice": "MARKET_004",
                    "messaggio": "Sottocategoria non valida",
                    "http_status": 400
                },
                "ERROR_INVALID_PRICE": {
                    "codice": "MARKET_005",
                    "messaggio": "Prezzo non valido",
                    "http_status": 400
                },
                "ERROR_INVALID_ATTRIBUTES": {
                    "codice": "MARKET_006",
                    "messaggio": "Attributi non validi per questa categoria",
                    "http_status": 400
                },
                "ERROR_INVALID_MEDIA": {
                    "codice": "MARKET_007",
                    "messaggio": "Media non valido o non autorizzato",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_LISTING
        # ───────────────────────────────────────────────────────────────────────
        "GET_LISTING": {
            "descrizione": "Recupera dettagli di un annuncio",
            "attore": "qualsiasi",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste LISTING con id == input.listing_id",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "LISTING.status != 'deleted' OPPURE richiedente è il seller",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.visibility == 'public' OPPURE richiedente è il seller OPPURE ha link diretto (unlisted)",
                    "errore_se_falsa": "ERROR_LISTING_PRIVATE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing con dati correlati",
                    "dettaglio": """
                        listing = SELECT l.*, 
                            c.name as category_name, c.slug as category_slug,
                            u.username as seller_username,
                            p.display_name as seller_display_name, p.avatar_url as seller_avatar
                        FROM LISTING l
                        JOIN CATEGORY c ON l.category_id = c.id
                        JOIN USER u ON l.seller_id = u.id
                        JOIN PROFILE p ON u.id = p.user_id
                        WHERE l.id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Recupera profilo venditore",
                    "dettaglio": """
                        listing.seller_profile = SELECT rating_average, rating_count, total_sales, 
                            response_rate, response_time_hours, is_verified
                        FROM SELLER_PROFILE WHERE user_id == listing.seller_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Incrementa views se non è il seller",
                    "dettaglio": """
                        SE NOT utente_autenticato OR current_user.id != listing.seller_id:
                            UPDATE LISTING SET views_count = views_count + 1
                            WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Aggiungi info utente corrente",
                    "dettaglio": """
                        SE utente_autenticato:
                            listing.is_favorite = EXISTS FAVORITE 
                                WHERE user_id == current_user.id 
                                AND listing_id == input.listing_id
                            
                            listing.user_offer = SELECT * FROM OFFER
                                WHERE listing_id == input.listing_id
                                AND buyer_id == current_user.id
                                AND status IN ['pending', 'countered']
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listing": "oggetto LISTING con dati correlati"
            },
            
            "errori": {
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MARKET_010",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_LISTING_PRIVATE": {
                    "codice": "MARKET_011",
                    "messaggio": "Annuncio privato",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: SEARCH_LISTINGS
        # ───────────────────────────────────────────────────────────────────────
        "SEARCH_LISTINGS": {
            "descrizione": "Cerca annunci con filtri",
            "attore": "qualsiasi",
            
            "input": {
                "query": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 200 caratteri"
                },
                "category_id": {
                    "tipo": "uuid",
                    "required": False
                },
                "listing_type": {
                    "tipo": "enum",
                    "required": False
                },
                "price_min": {
                    "tipo": "integer",
                    "required": False
                },
                "price_max": {
                    "tipo": "integer",
                    "required": False
                },
                "condition": {
                    "tipo": "array",
                    "required": False,
                    "validazione": "array di enum condition"
                },
                "location": {
                    "tipo": "json",
                    "required": False,
                    "descrizione": "{ city, radius_km, latitude, longitude }"
                },
                "sort_by": {
                    "tipo": "enum",
                    "required": False,
                    "default": "relevance",
                    "validazione": "relevance | price_asc | price_desc | newest | closest"
                },
                "cursor": {
                    "tipo": "string",
                    "required": False
                },
                "limit": {
                    "tipo": "integer",
                    "required": False,
                    "default": 20
                }
            },
            
            "pre_condizioni": [],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Costruisci query base",
                    "dettaglio": """
                        query = SELECT l.*, u.username, p.display_name, p.avatar_url,
                            sp.rating_average, sp.is_verified
                        FROM LISTING l
                        JOIN USER u ON l.seller_id = u.id
                        JOIN PROFILE p ON u.id = p.user_id
                        LEFT JOIN SELLER_PROFILE sp ON u.id = sp.user_id
                        WHERE l.status == 'active'
                        AND l.visibility == 'public'
                    """
                },
                {
                    "numero": 2,
                    "azione": "Applica filtro ricerca testuale",
                    "dettaglio": """
                        SE input.query fornito:
                            # Full-text search su title e description
                            query = query AND (
                                l.title ILIKE '%' || input.query || '%'
                                OR l.description ILIKE '%' || input.query || '%'
                            )
                    """
                },
                {
                    "numero": 3,
                    "azione": "Applica filtri categoria e tipo",
                    "dettaglio": """
                        SE input.category_id:
                            # Include anche sottocategorie
                            category_ids = [input.category_id] + SELECT id FROM CATEGORY WHERE parent_id == input.category_id
                            query = query AND l.category_id IN category_ids
                        
                        SE input.listing_type:
                            query = query AND l.listing_type == input.listing_type
                    """
                },
                {
                    "numero": 4,
                    "azione": "Applica filtri prezzo",
                    "dettaglio": """
                        SE input.price_min:
                            query = query AND l.price->>'amount' >= input.price_min
                        
                        SE input.price_max:
                            query = query AND l.price->>'amount' <= input.price_max
                    """
                },
                {
                    "numero": 5,
                    "azione": "Applica filtro condizione",
                    "dettaglio": """
                        SE input.condition fornito E non vuoto:
                            query = query AND l.condition IN input.condition
                    """
                },
                {
                    "numero": 6,
                    "azione": "Applica filtro località",
                    "dettaglio": """
                        SE input.location fornito:
                            SE input.location.city:
                                query = query AND l.location->>'city' ILIKE input.location.city
                            ALTRIMENTI SE input.location.latitude E input.location.longitude E input.location.radius_km:
                                # Calcola distanza con formula Haversine
                                query = query AND calcola_distanza(
                                    l.location->>'latitude', l.location->>'longitude',
                                    input.location.latitude, input.location.longitude
                                ) <= input.location.radius_km
                    """
                },
                {
                    "numero": 7,
                    "azione": "Escludi annunci di utenti bloccati",
                    "dettaglio": """
                        SE utente_autenticato:
                            blocked_ids = SELECT blocked_id FROM BLOCK WHERE blocker_id == current_user.id
                            query = query AND l.seller_id NOT IN blocked_ids
                    """
                },
                {
                    "numero": 8,
                    "azione": "Applica ordinamento",
                    "dettaglio": """
                        SE input.sort_by == 'price_asc':
                            query = query ORDER BY l.price->>'amount' ASC
                        ALTRIMENTI SE input.sort_by == 'price_desc':
                            query = query ORDER BY l.price->>'amount' DESC
                        ALTRIMENTI SE input.sort_by == 'newest':
                            query = query ORDER BY l.published_at DESC
                        ALTRIMENTI SE input.sort_by == 'closest' E input.location:
                            query = query ORDER BY distanza ASC
                        ALTRIMENTI:  # relevance
                            query = query ORDER BY 
                                (l.promoted_until > now()) DESC,  # Promossi prima
                                sp.rating_average DESC,
                                l.published_at DESC
                    """
                },
                {
                    "numero": 9,
                    "azione": "Applica paginazione",
                    "dettaglio": """
                        query = query LIMIT input.limit + 1
                        SE input.cursor:
                            query = query OFFSET decodifica(cursor)
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "listings": "array di annunci",
                "total_count": "numero totale risultati (approssimato)",
                "has_more": "boolean",
                "next_cursor": "string o null"
            },
            
            "errori": {}
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: MAKE_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "MAKE_OFFER": {
            "descrizione": "Fai un'offerta per un annuncio",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "amount": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "importo in centesimi, > 0"
                },
                "message": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 1000 caratteri"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "LISTING.seller_id != current_user.id",
                    "errore_se_falsa": "ERROR_CANNOT_OFFER_OWN_LISTING"
                },
                {
                    "id": "PRE_4",
                    "condizione": "LISTING.price.type IN ['fixed', 'negotiable']",
                    "errore_se_falsa": "ERROR_OFFERS_NOT_ALLOWED"
                },
                {
                    "id": "PRE_5",
                    "condizione": "NON esiste OFFER con listing_id E buyer_id == current_user.id E status IN ['pending', 'countered']",
                    "errore_se_falsa": "ERROR_OFFER_EXISTS"
                },
                {
                    "id": "PRE_6",
                    "condizione": "NON esiste BLOCK tra buyer e seller",
                    "errore_se_falsa": "ERROR_USER_BLOCKED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera listing",
                    "dettaglio": """
                        listing = SELECT * FROM LISTING WHERE id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Valida importo offerta",
                    "dettaglio": """
                        SE listing.price.type == 'fixed':
                            SE input.amount < listing.price.amount:
                                # Per prezzo fisso, offerta deve essere >= prezzo richiesto
                                ERRORE: ERROR_OFFER_TOO_LOW
                    """
                },
                {
                    "numero": 3,
                    "azione": "Crea record OFFER",
                    "dettaglio": """
                        offer_id = genera_uuid_v4()
                        
                        INSERT INTO OFFER:
                        - id = offer_id
                        - listing_id = input.listing_id
                        - buyer_id = current_user.id
                        - seller_id = listing.seller_id
                        - amount = input.amount
                        - currency = listing.price.currency
                        - message = input.message
                        - status = 'pending'
                        - expires_at = now() + 48 ore
                        - created_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica venditore",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = listing.seller_id,
                            type = 'new_offer',
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer_id,
                            metadata = {
                                listing_id: listing.id,
                                listing_title: listing.title,
                                amount: input.amount
                            }
                        )
                    """
                },
                {
                    "numero": 5,
                    "azione": "Aggiorna contatore richieste",
                    "dettaglio": """
                        UPDATE LISTING SET inquiries_count = inquiries_count + 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste OFFER con id == offer_id"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_020",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MARKET_021",
                    "messaggio": "Annuncio non trovato o non disponibile",
                    "http_status": 404
                },
                "ERROR_CANNOT_OFFER_OWN_LISTING": {
                    "codice": "MARKET_022",
                    "messaggio": "Non puoi fare offerte sui tuoi annunci",
                    "http_status": 400
                },
                "ERROR_OFFERS_NOT_ALLOWED": {
                    "codice": "MARKET_023",
                    "messaggio": "Offerte non permesse per questo annuncio",
                    "http_status": 400
                },
                "ERROR_OFFER_EXISTS": {
                    "codice": "MARKET_024",
                    "messaggio": "Hai già un'offerta attiva per questo annuncio",
                    "http_status": 409
                },
                "ERROR_OFFER_TOO_LOW": {
                    "codice": "MARKET_025",
                    "messaggio": "Offerta inferiore al prezzo richiesto",
                    "http_status": 400
                },
                "ERROR_USER_BLOCKED": {
                    "codice": "MARKET_026",
                    "messaggio": "Non puoi fare offerte a questo venditore",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: RESPOND_TO_OFFER
        # ───────────────────────────────────────────────────────────────────────
        "RESPOND_TO_OFFER": {
            "descrizione": "Rispondi a un'offerta (accetta, rifiuta, contro-offerta)",
            "attore": "utente_autenticato (seller)",
            
            "input": {
                "offer_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "action": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "accept | reject | counter"
                },
                "counter_amount": {
                    "tipo": "integer",
                    "required": False,
                    "descrizione": "Richiesto se action == 'counter'"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste OFFER con id == input.offer_id E status == 'pending'",
                    "errore_se_falsa": "ERROR_OFFER_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "OFFER.seller_id == current_user.id",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "SE input.action == 'counter': input.counter_amount fornito E > 0",
                    "errore_se_falsa": "ERROR_COUNTER_AMOUNT_REQUIRED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera offerta e listing",
                    "dettaglio": """
                        offer = SELECT * FROM OFFER WHERE id == input.offer_id
                        listing = SELECT * FROM LISTING WHERE id == offer.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Esegui azione",
                    "dettaglio": """
                        SE input.action == 'accept':
                            UPDATE OFFER SET 
                            - status = 'accepted'
                            - responded_at = now()
                            WHERE id == offer.id
                            
                            # Crea transazione
                            esegui CREATE_TRANSACTION(
                                listing_id = listing.id,
                                offer_id = offer.id,
                                buyer_id = offer.buyer_id,
                                amount = offer.amount
                            )
                            
                            # Marca listing come reserved
                            UPDATE LISTING SET status = 'reserved' WHERE id == listing.id
                        
                        ALTRIMENTI SE input.action == 'reject':
                            UPDATE OFFER SET 
                            - status = 'rejected'
                            - responded_at = now()
                            WHERE id == offer.id
                        
                        ALTRIMENTI SE input.action == 'counter':
                            UPDATE OFFER SET 
                            - status = 'countered'
                            - counter_amount = input.counter_amount
                            - responded_at = now()
                            - expires_at = now() + 48 ore  # Resetta scadenza
                            WHERE id == offer.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Notifica buyer",
                    "dettaglio": """
                        notification_type = 'offer_' + input.action
                        
                        crea_notifica(
                            recipient_id = offer.buyer_id,
                            type = notification_type,
                            source_user_id = current_user.id,
                            target_type = 'offer',
                            target_id = offer.id,
                            metadata = {
                                listing_id: listing.id,
                                listing_title: listing.title,
                                counter_amount: input.counter_amount  # se presente
                            }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "OFFER.status corrisponde all'azione eseguita"
                }
            ],
            
            "output": {
                "success": True,
                "offer": "oggetto OFFER aggiornato",
                "transaction": "oggetto TRANSACTION se action == 'accept'"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_OFFER_NOT_FOUND": {
                    "codice": "MARKET_031",
                    "messaggio": "Offerta non trovata o non più valida",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "MARKET_032",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_COUNTER_AMOUNT_REQUIRED": {
                    "codice": "MARKET_033",
                    "messaggio": "Importo contro-offerta richiesto",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: LEAVE_REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "LEAVE_REVIEW": {
            "descrizione": "Lascia una recensione dopo una transazione",
            "attore": "utente_autenticato",
            
            "input": {
                "transaction_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "1-5"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 100 caratteri"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 2000 caratteri"
                },
                "aspects": {
                    "tipo": "json",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste TRANSACTION con id == input.transaction_id E status == 'completed'",
                    "errore_se_falsa": "ERROR_TRANSACTION_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "current_user.id IN [TRANSACTION.buyer_id, TRANSACTION.seller_id]",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "NON esiste REVIEW con transaction_id E reviewer_id == current_user.id",
                    "errore_se_falsa": "ERROR_REVIEW_EXISTS"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Determina tipo recensione e destinatario",
                    "dettaglio": """
                        transaction = SELECT * FROM TRANSACTION WHERE id == input.transaction_id
                        
                        SE current_user.id == transaction.buyer_id:
                            review_type = 'buyer_to_seller'
                            reviewee_id = transaction.seller_id
                        ALTRIMENTI:
                            review_type = 'seller_to_buyer'
                            reviewee_id = transaction.buyer_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea record REVIEW",
                    "dettaglio": """
                        review_id = genera_uuid_v4()
                        
                        INSERT INTO REVIEW:
                        - id = review_id
                        - transaction_id = input.transaction_id
                        - reviewer_id = current_user.id
                        - reviewee_id = reviewee_id
                        - review_type = review_type
                        - rating = input.rating
                        - title = input.title
                        - content = input.content
                        - aspects = input.aspects
                        - is_public = true
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna statistiche venditore se buyer_to_seller",
                    "dettaglio": """
                        SE review_type == 'buyer_to_seller':
                            # Ricalcola media
                            stats = SELECT AVG(rating) as avg_rating, COUNT(*) as count
                            FROM REVIEW WHERE reviewee_id == reviewee_id AND review_type == 'buyer_to_seller'
                            
                            UPDATE SELLER_PROFILE SET
                            - rating_average = stats.avg_rating
                            - rating_count = stats.count
                            - updated_at = now()
                            WHERE user_id == reviewee_id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica destinatario",
                    "dettaglio": """
                        crea_notifica(
                            recipient_id = reviewee_id,
                            type = 'new_review',
                            source_user_id = current_user.id,
                            target_type = 'review',
                            target_id = review_id
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste REVIEW con id == review_id"
                }
            ],
            
            "output": {
                "success": True,
                "review": "oggetto REVIEW creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_TRANSACTION_NOT_FOUND": {
                    "codice": "MARKET_041",
                    "messaggio": "Transazione non trovata o non completata",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "MARKET_042",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_REVIEW_EXISTS": {
                    "codice": "MARKET_043",
                    "messaggio": "Hai già lasciato una recensione",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ADD_TO_FAVORITES
        # ───────────────────────────────────────────────────────────────────────
        "ADD_TO_FAVORITES": {
            "descrizione": "Aggiungi un annuncio ai preferiti",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste LISTING con id == input.listing_id E status == 'active'",
                    "errore_se_falsa": "ERROR_LISTING_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "NON esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id",
                    "errore_se_falsa": "ERROR_ALREADY_FAVORITE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Crea record FAVORITE",
                    "dettaglio": """
                        INSERT INTO FAVORITE:
                        - id = genera_uuid_v4()
                        - user_id = current_user.id
                        - listing_id = input.listing_id
                        - created_at = now()
                    """
                },
                {
                    "numero": 2,
                    "azione": "Incrementa contatore",
                    "dettaglio": """
                        UPDATE LISTING SET favorites_count = favorites_count + 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste FAVORITE"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio aggiunto ai preferiti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_LISTING_NOT_FOUND": {
                    "codice": "MARKET_051",
                    "messaggio": "Annuncio non trovato",
                    "http_status": 404
                },
                "ERROR_ALREADY_FAVORITE": {
                    "codice": "MARKET_052",
                    "messaggio": "Annuncio già nei preferiti",
                    "http_status": 409
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: REMOVE_FROM_FAVORITES
        # ───────────────────────────────────────────────────────────────────────
        "REMOVE_FROM_FAVORITES": {
            "descrizione": "Rimuovi un annuncio dai preferiti",
            "attore": "utente_autenticato",
            
            "input": {
                "listing_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste FAVORITE con user_id == current_user.id E listing_id == input.listing_id",
                    "errore_se_falsa": "ERROR_NOT_FAVORITE"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Elimina record FAVORITE",
                    "dettaglio": """
                        DELETE FROM FAVORITE 
                        WHERE user_id == current_user.id 
                        AND listing_id == input.listing_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Decrementa contatore",
                    "dettaglio": """
                        UPDATE LISTING SET favorites_count = favorites_count - 1
                        WHERE id == input.listing_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "NON esiste FAVORITE"
                }
            ],
            
            "output": {
                "success": True,
                "message": "Annuncio rimosso dai preferiti"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "MARKET_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_NOT_FAVORITE": {
                    "codice": "MARKET_061",
                    "messaggio": "Annuncio non nei preferiti",
                    "http_status": 404
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo MARKETPLACE"
}



    # ═══════════════════════════════════════════════════════════════════════════
    # OPERAZIONI - MODULO COMMERCE
    # ═══════════════════════════════════════════════════════════════════════════
    
    "operazioni": {
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: ADD_TO_CART
        # ───────────────────────────────────────────────────────────────────────
        "ADD_TO_CART": {
            "descrizione": "Aggiungi un prodotto al carrello",
            "attore": "qualsiasi",
            
            "input": {
                "product_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "variant_id": {
                    "tipo": "uuid",
                    "required": False,
                    "descrizione": "Richiesto se prodotto ha varianti"
                },
                "quantity": {
                    "tipo": "integer",
                    "required": False,
                    "default": 1,
                    "validazione": "min 1"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste PRODUCT con id == input.product_id E status == 'active'",
                    "errore_se_falsa": "ERROR_PRODUCT_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "SE prodotto ha varianti: input.variant_id deve essere fornito",
                    "errore_se_falsa": "ERROR_VARIANT_REQUIRED"
                },
                {
                    "id": "PRE_3",
                    "condizione": "SE input.variant_id: PRODUCT_VARIANT.status == 'active'",
                    "errore_se_falsa": "ERROR_VARIANT_NOT_FOUND"
                },
                {
                    "id": "PRE_4",
                    "condizione": "Quantità richiesta <= stock disponibile O allow_backorder == true",
                    "errore_se_falsa": "ERROR_INSUFFICIENT_STOCK"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera o crea carrello",
                    "dettaglio": """
                        SE utente_autenticato:
                            cart = SELECT * FROM CART 
                                WHERE user_id == current_user.id 
                                AND status == 'active'
                            
                            SE cart non esiste:
                                cart = crea_nuovo_carrello(user_id = current_user.id)
                        ALTRIMENTI:
                            cart = recupera_carrello_da_sessione()
                            SE cart non esiste:
                                cart = crea_nuovo_carrello(session_id = genera_session_id())
                    """
                },
                {
                    "numero": 2,
                    "azione": "Recupera prodotto e prezzo",
                    "dettaglio": """
                        product = SELECT * FROM PRODUCT WHERE id == input.product_id
                        
                        SE input.variant_id:
                            variant = SELECT * FROM PRODUCT_VARIANT WHERE id == input.variant_id
                            unit_price = variant.price.amount SE variant.price ALTRIMENTI product.price.amount
                            sku = variant.sku SE variant.sku ALTRIMENTI product.sku
                            image_url = variant.image_url SE variant.image_url ALTRIMENTI product.media[0].url
                            variant_name = variant.name
                        ALTRIMENTI:
                            unit_price = product.price.amount
                            sku = product.sku
                            image_url = product.media[0].url SE product.media ALTRIMENTI null
                            variant_name = null
                    """
                },
                {
                    "numero": 3,
                    "azione": "Verifica se item già presente nel carrello",
                    "dettaglio": """
                        existing_item = SELECT * FROM CART_ITEM
                            WHERE cart_id == cart.id
                            AND product_id == input.product_id
                            AND (variant_id == input.variant_id OR (variant_id IS NULL AND input.variant_id IS NULL))
                        
                        SE existing_item esiste:
                            new_quantity = existing_item.quantity + input.quantity
                            
                            # Verifica stock per nuova quantità
                            SE product.inventory.track_quantity:
                                available = variant.inventory.quantity SE variant ALTRIMENTI product.inventory.quantity
                                SE new_quantity > available AND NOT product.inventory.allow_backorder:
                                    ERRORE: ERROR_INSUFFICIENT_STOCK
                            
                            UPDATE CART_ITEM SET
                            - quantity = new_quantity
                            - total_price = unit_price * new_quantity
                            WHERE id == existing_item.id
                        ALTRIMENTI:
                            # Crea nuovo item
                            INSERT INTO CART_ITEM:
                            - id = genera_uuid_v4()
                            - cart_id = cart.id
                            - product_id = input.product_id
                            - variant_id = input.variant_id
                            - quantity = input.quantity
                            - unit_price = unit_price
                            - total_price = unit_price * input.quantity
                            - product_snapshot = {
                                name: product.name,
                                sku: sku,
                                image_url: image_url,
                                variant_name: variant_name
                              }
                            - added_at = now()
                    """
                },
                {
                    "numero": 4,
                    "azione": "Ricalcola totali carrello",
                    "dettaglio": """
                        totals = SELECT 
                            SUM(total_price) as subtotal,
                            COUNT(*) as items_count
                        FROM CART_ITEM WHERE cart_id == cart.id
                        
                        UPDATE CART SET
                        - subtotal = totals.subtotal
                        - items_count = totals.items_count
                        - total = totals.subtotal - cart.discount_amount + cart.shipping_amount + cart.tax_amount
                        - updated_at = now()
                        WHERE id == cart.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste CART_ITEM per il prodotto nel carrello"
                }
            ],
            
            "output": {
                "success": True,
                "cart": "oggetto CART aggiornato con items"
            },
            
            "errori": {
                "ERROR_PRODUCT_NOT_FOUND": {
                    "codice": "COMM_001",
                    "messaggio": "Prodotto non trovato o non disponibile",
                    "http_status": 404
                },
                "ERROR_VARIANT_REQUIRED": {
                    "codice": "COMM_002",
                    "messaggio": "Seleziona una variante",
                    "http_status": 400
                },
                "ERROR_VARIANT_NOT_FOUND": {
                    "codice": "COMM_003",
                    "messaggio": "Variante non trovata o non disponibile",
                    "http_status": 404
                },
                "ERROR_INSUFFICIENT_STOCK": {
                    "codice": "COMM_004",
                    "messaggio": "Quantità non disponibile",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UPDATE_CART_ITEM
        # ───────────────────────────────────────────────────────────────────────
        "UPDATE_CART_ITEM": {
            "descrizione": "Modifica quantità di un item nel carrello",
            "attore": "qualsiasi",
            
            "input": {
                "item_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "quantity": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "min 0 (0 = rimuovi)"
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste CART_ITEM con id == input.item_id E appartiene al carrello dell'utente",
                    "errore_se_falsa": "ERROR_ITEM_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "SE input.quantity > 0: stock sufficiente",
                    "errore_se_falsa": "ERROR_INSUFFICIENT_STOCK"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera item e carrello",
                    "dettaglio": """
                        item = SELECT * FROM CART_ITEM WHERE id == input.item_id
                        cart = SELECT * FROM CART WHERE id == item.cart_id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna o rimuovi item",
                    "dettaglio": """
                        SE input.quantity == 0:
                            DELETE FROM CART_ITEM WHERE id == input.item_id
                        ALTRIMENTI:
                            UPDATE CART_ITEM SET
                            - quantity = input.quantity
                            - total_price = item.unit_price * input.quantity
                            WHERE id == input.item_id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Ricalcola totali carrello",
                    "dettaglio": """
                        totals = SELECT 
                            COALESCE(SUM(total_price), 0) as subtotal,
                            COUNT(*) as items_count
                        FROM CART_ITEM WHERE cart_id == cart.id
                        
                        UPDATE CART SET
                        - subtotal = totals.subtotal
                        - items_count = totals.items_count
                        - total = totals.subtotal - cart.discount_amount + cart.shipping_amount + cart.tax_amount
                        - updated_at = now()
                        WHERE id == cart.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "CART_ITEM aggiornato o rimosso"
                }
            ],
            
            "output": {
                "success": True,
                "cart": "oggetto CART aggiornato"
            },
            
            "errori": {
                "ERROR_ITEM_NOT_FOUND": {
                    "codice": "COMM_010",
                    "messaggio": "Item non trovato nel carrello",
                    "http_status": 404
                },
                "ERROR_INSUFFICIENT_STOCK": {
                    "codice": "COMM_011",
                    "messaggio": "Quantità non disponibile",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: APPLY_COUPON
        # ───────────────────────────────────────────────────────────────────────
        "APPLY_COUPON": {
            "descrizione": "Applica un codice coupon al carrello",
            "attore": "qualsiasi",
            
            "input": {
                "coupon_code": {
                    "tipo": "string",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Esiste carrello attivo per l'utente/sessione",
                    "errore_se_falsa": "ERROR_CART_NOT_FOUND"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste COUPON con code == input.coupon_code E is_active == true",
                    "errore_se_falsa": "ERROR_INVALID_COUPON"
                },
                {
                    "id": "PRE_3",
                    "condizione": "COUPON.valid_from <= now() <= COUPON.valid_until",
                    "errore_se_falsa": "ERROR_COUPON_EXPIRED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "COUPON.usage_count < COUPON.usage_limit",
                    "errore_se_falsa": "ERROR_COUPON_LIMIT_REACHED"
                },
                {
                    "id": "PRE_5",
                    "condizione": "cart.subtotal >= COUPON.minimum_order_amount",
                    "errore_se_falsa": "ERROR_MINIMUM_NOT_MET"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera carrello e coupon",
                    "dettaglio": """
                        cart = recupera_carrello_corrente()
                        coupon = SELECT * FROM COUPON WHERE code == input.coupon_code
                    """
                },
                {
                    "numero": 2,
                    "azione": "Calcola sconto",
                    "dettaglio": """
                        SE coupon.discount_type == 'percentage':
                            discount = (cart.subtotal * coupon.discount_value) / 100
                            SE coupon.max_discount_amount E discount > coupon.max_discount_amount:
                                discount = coupon.max_discount_amount
                        ALTRIMENTI SE coupon.discount_type == 'fixed':
                            discount = coupon.discount_value
                        ALTRIMENTI SE coupon.discount_type == 'free_shipping':
                            discount = cart.shipping_amount
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna carrello",
                    "dettaglio": """
                        UPDATE CART SET
                        - coupon_code = input.coupon_code
                        - discount_amount = discount
                        - total = cart.subtotal - discount + cart.shipping_amount + cart.tax_amount
                        - updated_at = now()
                        WHERE id == cart.id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "cart.coupon_code == input.coupon_code"
                }
            ],
            
            "output": {
                "success": True,
                "cart": "oggetto CART aggiornato",
                "discount_applied": "importo sconto applicato"
            },
            
            "errori": {
                "ERROR_CART_NOT_FOUND": {
                    "codice": "COMM_020",
                    "messaggio": "Carrello non trovato",
                    "http_status": 404
                },
                "ERROR_INVALID_COUPON": {
                    "codice": "COMM_021",
                    "messaggio": "Codice coupon non valido",
                    "http_status": 400
                },
                "ERROR_COUPON_EXPIRED": {
                    "codice": "COMM_022",
                    "messaggio": "Coupon scaduto",
                    "http_status": 400
                },
                "ERROR_COUPON_LIMIT_REACHED": {
                    "codice": "COMM_023",
                    "messaggio": "Limite utilizzo coupon raggiunto",
                    "http_status": 400
                },
                "ERROR_MINIMUM_NOT_MET": {
                    "codice": "COMM_024",
                    "messaggio": "Ordine minimo non raggiunto per questo coupon",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: CREATE_ORDER
        # ───────────────────────────────────────────────────────────────────────
        "CREATE_ORDER": {
            "descrizione": "Crea un ordine dal carrello (checkout)",
            "attore": "utente_autenticato",
            
            "input": {
                "billing_address": {
                    "tipo": "json",
                    "required": True,
                    "validazione": "indirizzo completo"
                },
                "shipping_address": {
                    "tipo": "json",
                    "required": False,
                    "descrizione": "Se non fornito, usa billing_address"
                },
                "shipping_method_id": {
                    "tipo": "string",
                    "required": True
                },
                "payment_method": {
                    "tipo": "json",
                    "required": True,
                    "descrizione": "{ type, token, ... }"
                },
                "notes": {
                    "tipo": "string",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste CART attivo con items_count > 0",
                    "errore_se_falsa": "ERROR_CART_EMPTY"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Tutti i prodotti nel carrello sono ancora disponibili",
                    "errore_se_falsa": "ERROR_PRODUCT_UNAVAILABLE"
                },
                {
                    "id": "PRE_4",
                    "condizione": "Quantità richieste sono ancora in stock",
                    "errore_se_falsa": "ERROR_INSUFFICIENT_STOCK"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera e valida carrello",
                    "dettaglio": """
                        cart = SELECT * FROM CART WHERE user_id == current_user.id AND status == 'active'
                        cart_items = SELECT * FROM CART_ITEM WHERE cart_id == cart.id
                        
                        # Verifica disponibilità prodotti
                        PER OGNI item IN cart_items:
                            product = SELECT * FROM PRODUCT WHERE id == item.product_id
                            SE product.status != 'active':
                                items_unavailable.push(item)
                            
                            SE product.inventory.track_quantity:
                                available = SE item.variant_id:
                                    SELECT quantity FROM PRODUCT_VARIANT WHERE id == item.variant_id
                                ALTRIMENTI:
                                    product.inventory.quantity
                                
                                SE item.quantity > available E NOT product.inventory.allow_backorder:
                                    items_insufficient.push({item, available})
                        
                        SE items_unavailable.length > 0:
                            ERRORE: ERROR_PRODUCT_UNAVAILABLE con lista items
                        
                        SE items_insufficient.length > 0:
                            ERRORE: ERROR_INSUFFICIENT_STOCK con lista items
                    """
                },
                {
                    "numero": 2,
                    "azione": "Calcola spedizione",
                    "dettaglio": """
                        shipping_method = recupera_metodo_spedizione(input.shipping_method_id)
                        SE NOT shipping_method:
                            ERRORE: ERROR_INVALID_SHIPPING_METHOD
                        
                        shipping_address = input.shipping_address SE fornito ALTRIMENTI input.billing_address
                        shipping_amount = calcola_costo_spedizione(cart_items, shipping_method, shipping_address)
                    """
                },
                {
                    "numero": 3,
                    "azione": "Calcola tasse",
                    "dettaglio": """
                        tax_amount = calcola_tasse(cart.subtotal, shipping_address)
                    """
                },
                {
                    "numero": 4,
                    "azione": "Calcola totale finale",
                    "dettaglio": """
                        total = cart.subtotal - cart.discount_amount + shipping_amount + tax_amount
                    """
                },
                {
                    "numero": 5,
                    "azione": "Genera numero ordine",
                    "dettaglio": """
                        year = now().year
                        sequence = SELECT MAX(sequence) FROM ORDER WHERE YEAR(created_at) == year
                        order_number = 'ORD-' + year + '-' + pad(sequence + 1, 6)
                        # Esempio: ORD-2024-000001
                    """
                },
                {
                    "numero": 6,
                    "azione": "Crea record ORDER",
                    "dettaglio": """
                        order_id = genera_uuid_v4()
                        
                        INSERT INTO ORDER:
                        - id = order_id
                        - order_number = order_number
                        - customer_id = current_user.id
                        - status = 'pending'
                        - payment_status = 'pending'
                        - fulfillment_status = 'unfulfilled'
                        - currency = cart.currency
                        - subtotal = cart.subtotal
                        - discount_amount = cart.discount_amount
                        - coupon_code = cart.coupon_code
                        - shipping_amount = shipping_amount
                        - tax_amount = tax_amount
                        - total = total
                        - billing_address = input.billing_address
                        - shipping_address = shipping_address
                        - shipping_method = {
                            id: shipping_method.id,
                            name: shipping_method.name,
                            carrier: shipping_method.carrier,
                            estimated_days: shipping_method.estimated_days
                          }
                        - payment_method = {
                            type: input.payment_method.type,
                            # Non salvare token o dati sensibili
                          }
                        - notes = input.notes
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 7,
                    "azione": "Crea ORDER_ITEMS",
                    "dettaglio": """
                        PER OGNI item IN cart_items:
                            product = SELECT * FROM PRODUCT WHERE id == item.product_id
                            
                            INSERT INTO ORDER_ITEM:
                            - id = genera_uuid_v4()
                            - order_id = order_id
                            - product_id = item.product_id
                            - variant_id = item.variant_id
                            - seller_id = product.seller_id
                            - quantity = item.quantity
                            - unit_price = item.unit_price
                            - total_price = item.total_price
                            - product_snapshot = {
                                name: product.name,
                                sku: item.product_snapshot.sku,
                                variant_name: item.product_snapshot.variant_name,
                                image_url: item.product_snapshot.image_url,
                                attributes: product.attributes
                              }
                            - fulfillment_status = 'unfulfilled'
                    """
                },
                {
                    "numero": 8,
                    "azione": "Processa pagamento",
                    "dettaglio": """
                        payment_result = processa_pagamento(
                            amount = total,
                            currency = cart.currency,
                            payment_method = input.payment_method,
                            order_id = order_id,
                            customer_email = current_user.email
                        )
                        
                        SE payment_result.success:
                            UPDATE ORDER SET
                            - payment_status = 'paid'
                            - payment_id = payment_result.transaction_id
                            - status = 'confirmed'
                            - paid_at = now()
                            WHERE id == order_id
                        ALTRIMENTI:
                            UPDATE ORDER SET
                            - payment_status = 'failed'
                            - status = 'cancelled'
                            WHERE id == order_id
                            
                            ERRORE: ERROR_PAYMENT_FAILED con dettagli
                    """
                },
                {
                    "numero": 9,
                    "azione": "Aggiorna inventario",
                    "dettaglio": """
                        PER OGNI item IN cart_items:
                            SE item.variant_id:
                                UPDATE PRODUCT_VARIANT SET
                                - inventory.quantity = inventory.quantity - item.quantity
                                WHERE id == item.variant_id
                            ALTRIMENTI:
                                UPDATE PRODUCT SET
                                - inventory.quantity = inventory.quantity - item.quantity
                                WHERE id == item.product_id
                    """
                },
                {
                    "numero": 10,
                    "azione": "Aggiorna statistiche prodotti",
                    "dettaglio": """
                        PER OGNI item IN cart_items:
                            UPDATE PRODUCT SET
                            - sales_count = sales_count + item.quantity
                            WHERE id == item.product_id
                    """
                },
                {
                    "numero": 11,
                    "azione": "Segna carrello come convertito",
                    "dettaglio": """
                        UPDATE CART SET
                        - status = 'converted'
                        - updated_at = now()
                        WHERE id == cart.id
                    """
                },
                {
                    "numero": 12,
                    "azione": "Incrementa utilizzo coupon",
                    "dettaglio": """
                        SE cart.coupon_code:
                            UPDATE COUPON SET usage_count = usage_count + 1
                            WHERE code == cart.coupon_code
                    """
                },
                {
                    "numero": 13,
                    "azione": "Invia notifiche e email",
                    "dettaglio": """
                        # Email conferma al cliente
                        invia_email(
                            to = current_user.email,
                            template = 'order_confirmation',
                            data = { order, items }
                        )
                        
                        # Notifica ai venditori (per ogni seller unico)
                        sellers = unique(cart_items.map(i => i.seller_id))
                        PER OGNI seller_id IN sellers:
                            seller_items = cart_items.filter(i => i.seller_id == seller_id)
                            crea_notifica(
                                recipient_id = seller_id,
                                type = 'new_order',
                                target_type = 'order',
                                target_id = order_id,
                                metadata = { items_count: seller_items.length }
                            )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste ORDER con payment_status == 'paid'"
                },
                {
                    "id": "POST_2",
                    "condizione": "Inventario decrementato"
                },
                {
                    "id": "POST_3",
                    "condizione": "Carrello status == 'converted'"
                }
            ],
            
            "output": {
                "success": True,
                "order": "oggetto ORDER completo"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "COMM_030",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_CART_EMPTY": {
                    "codice": "COMM_031",
                    "messaggio": "Carrello vuoto",
                    "http_status": 400
                },
                "ERROR_PRODUCT_UNAVAILABLE": {
                    "codice": "COMM_032",
                    "messaggio": "Alcuni prodotti non sono più disponibili",
                    "http_status": 400
                },
                "ERROR_INSUFFICIENT_STOCK": {
                    "codice": "COMM_033",
                    "messaggio": "Quantità non disponibile per alcuni prodotti",
                    "http_status": 400
                },
                "ERROR_INVALID_SHIPPING_METHOD": {
                    "codice": "COMM_034",
                    "messaggio": "Metodo di spedizione non valido",
                    "http_status": 400
                },
                "ERROR_PAYMENT_FAILED": {
                    "codice": "COMM_035",
                    "messaggio": "Pagamento fallito",
                    "http_status": 402
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: GET_ORDER
        # ───────────────────────────────────────────────────────────────────────
        "GET_ORDER": {
            "descrizione": "Recupera dettagli di un ordine",
            "attore": "utente_autenticato",
            
            "input": {
                "order_id": {
                    "tipo": "uuid",
                    "required": True
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste ORDER con id == input.order_id",
                    "errore_se_falsa": "ERROR_ORDER_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "ORDER.customer_id == current_user.id OPPURE current_user è seller di almeno un item",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Recupera ordine con items",
                    "dettaglio": """
                        order = SELECT * FROM ORDER WHERE id == input.order_id
                        order.items = SELECT * FROM ORDER_ITEM WHERE order_id == order.id
                    """
                },
                {
                    "numero": 2,
                    "azione": "Filtra items se utente è seller",
                    "dettaglio": """
                        SE current_user.id != order.customer_id:
                            # Utente è un seller, mostra solo i suoi items
                            order.items = order.items.filter(i => i.seller_id == current_user.id)
                            
                            # Nascondi informazioni cliente sensibili
                            order.billing_address = null
                            order.shipping_address = nascondi_dettagli_personali(order.shipping_address)
                    """
                }
            ],
            
            "post_condizioni": [],
            
            "output": {
                "success": True,
                "order": "oggetto ORDER con items"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "COMM_040",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_ORDER_NOT_FOUND": {
                    "codice": "COMM_041",
                    "messaggio": "Ordine non trovato",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "COMM_042",
                    "messaggio": "Non autorizzato a visualizzare questo ordine",
                    "http_status": 403
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: UPDATE_ORDER_STATUS
        # ───────────────────────────────────────────────────────────────────────
        "UPDATE_ORDER_STATUS": {
            "descrizione": "Aggiorna lo stato di un ordine (seller/admin)",
            "attore": "utente_autenticato (seller o admin)",
            
            "input": {
                "order_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "status": {
                    "tipo": "enum",
                    "required": True,
                    "validazione": "processing | shipped | delivered"
                },
                "tracking_number": {
                    "tipo": "string",
                    "required": False,
                    "descrizione": "Richiesto per status == 'shipped'"
                },
                "tracking_url": {
                    "tipo": "string",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste ORDER con id == input.order_id",
                    "errore_se_falsa": "ERROR_ORDER_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "Utente è seller di almeno un item O è admin",
                    "errore_se_falsa": "ERROR_NOT_AUTHORIZED"
                },
                {
                    "id": "PRE_4",
                    "condizione": "Transizione di stato è valida",
                    "errore_se_falsa": "ERROR_INVALID_TRANSITION"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Valida transizione stato",
                    "dettaglio": """
                        order = SELECT * FROM ORDER WHERE id == input.order_id
                        
                        transizioni_valide = {
                            'confirmed': ['processing'],
                            'processing': ['shipped'],
                            'shipped': ['delivered']
                        }
                        
                        SE input.status NOT IN transizioni_valide[order.status]:
                            ERRORE: ERROR_INVALID_TRANSITION
                    """
                },
                {
                    "numero": 2,
                    "azione": "Aggiorna ordine",
                    "dettaglio": """
                        update_data = {
                            status: input.status,
                            updated_at: now()
                        }
                        
                        SE input.status == 'shipped':
                            SE NOT input.tracking_number:
                                ERRORE: ERROR_TRACKING_REQUIRED
                            update_data.tracking_number = input.tracking_number
                            update_data.tracking_url = input.tracking_url
                            update_data.shipped_at = now()
                            update_data.fulfillment_status = 'fulfilled'
                        
                        SE input.status == 'delivered':
                            update_data.delivered_at = now()
                        
                        UPDATE ORDER SET update_data WHERE id == order.id
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna fulfillment_status items (se seller specifico)",
                    "dettaglio": """
                        SE NOT utente_è_admin:
                            UPDATE ORDER_ITEM SET
                            - fulfillment_status = 'fulfilled'
                            WHERE order_id == order.id AND seller_id == current_user.id
                    """
                },
                {
                    "numero": 4,
                    "azione": "Notifica cliente",
                    "dettaglio": """
                        notification_type = 'order_' + input.status
                        
                        crea_notifica(
                            recipient_id = order.customer_id,
                            type = notification_type,
                            target_type = 'order',
                            target_id = order.id,
                            metadata = {
                                tracking_number: input.tracking_number
                            }
                        )
                        
                        # Email
                        invia_email(
                            to = get_customer_email(order.customer_id),
                            template = notification_type,
                            data = { order, tracking_number, tracking_url }
                        )
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "ORDER.status aggiornato"
                }
            ],
            
            "output": {
                "success": True,
                "order": "oggetto ORDER aggiornato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "COMM_050",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_ORDER_NOT_FOUND": {
                    "codice": "COMM_051",
                    "messaggio": "Ordine non trovato",
                    "http_status": 404
                },
                "ERROR_NOT_AUTHORIZED": {
                    "codice": "COMM_052",
                    "messaggio": "Non autorizzato",
                    "http_status": 403
                },
                "ERROR_INVALID_TRANSITION": {
                    "codice": "COMM_053",
                    "messaggio": "Transizione di stato non valida",
                    "http_status": 400
                },
                "ERROR_TRACKING_REQUIRED": {
                    "codice": "COMM_054",
                    "messaggio": "Numero tracking richiesto per la spedizione",
                    "http_status": 400
                }
            }
        },
        
        # ───────────────────────────────────────────────────────────────────────
        # OPERAZIONE: LEAVE_PRODUCT_REVIEW
        # ───────────────────────────────────────────────────────────────────────
        "LEAVE_PRODUCT_REVIEW": {
            "descrizione": "Lascia una recensione per un prodotto acquistato",
            "attore": "utente_autenticato",
            
            "input": {
                "product_id": {
                    "tipo": "uuid",
                    "required": True
                },
                "order_item_id": {
                    "tipo": "uuid",
                    "required": False,
                    "descrizione": "Per verificare acquisto"
                },
                "rating": {
                    "tipo": "integer",
                    "required": True,
                    "validazione": "1-5"
                },
                "title": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 150 caratteri"
                },
                "content": {
                    "tipo": "string",
                    "required": False,
                    "validazione": "max 5000 caratteri"
                },
                "media_ids": {
                    "tipo": "array",
                    "required": False
                }
            },
            
            "pre_condizioni": [
                {
                    "id": "PRE_1",
                    "condizione": "Utente è autenticato",
                    "errore_se_falsa": "ERROR_UNAUTHORIZED"
                },
                {
                    "id": "PRE_2",
                    "condizione": "Esiste PRODUCT con id == input.product_id",
                    "errore_se_falsa": "ERROR_PRODUCT_NOT_FOUND"
                },
                {
                    "id": "PRE_3",
                    "condizione": "NON esiste PRODUCT_REVIEW con product_id E reviewer_id == current_user.id",
                    "errore_se_falsa": "ERROR_REVIEW_EXISTS"
                }
            ],
            
            "passi": [
                {
                    "numero": 1,
                    "azione": "Verifica se acquisto verificato",
                    "dettaglio": """
                        is_verified = EXISTS ORDER_ITEM oi
                            JOIN ORDER o ON oi.order_id = o.id
                            WHERE oi.product_id == input.product_id
                            AND o.customer_id == current_user.id
                            AND o.status IN ['delivered', 'completed']
                        
                        SE input.order_item_id:
                            # Verifica che l'order_item appartenga all'utente
                            order_item = SELECT oi.*, o.customer_id, o.status FROM ORDER_ITEM oi
                                JOIN ORDER o ON oi.order_id = o.id
                                WHERE oi.id == input.order_item_id
                            
                            SE order_item.customer_id != current_user.id:
                                ERRORE: ERROR_NOT_YOUR_ORDER
                            
                            is_verified = order_item.status IN ['delivered', 'completed']
                    """
                },
                {
                    "numero": 2,
                    "azione": "Crea PRODUCT_REVIEW",
                    "dettaglio": """
                        review_id = genera_uuid_v4()
                        
                        INSERT INTO PRODUCT_REVIEW:
                        - id = review_id
                        - product_id = input.product_id
                        - order_item_id = input.order_item_id
                        - reviewer_id = current_user.id
                        - rating = input.rating
                        - title = input.title
                        - content = input.content
                        - media = costruisci_media_array(input.media_ids)
                        - is_verified_purchase = is_verified
                        - status = 'published'
                        - created_at = now()
                        - updated_at = now()
                    """
                },
                {
                    "numero": 3,
                    "azione": "Aggiorna statistiche prodotto",
                    "dettaglio": """
                        stats = SELECT AVG(rating) as avg_rating, COUNT(*) as count
                        FROM PRODUCT_REVIEW 
                        WHERE product_id == input.product_id 
                        AND status == 'published'
                        
                        UPDATE PRODUCT SET
                        - rating_average = stats.avg_rating
                        - rating_count = stats.count
                        - updated_at = now()
                        WHERE id == input.product_id
                    """
                }
            ],
            
            "post_condizioni": [
                {
                    "id": "POST_1",
                    "condizione": "Esiste PRODUCT_REVIEW"
                }
            ],
            
            "output": {
                "success": True,
                "review": "oggetto PRODUCT_REVIEW creato"
            },
            
            "errori": {
                "ERROR_UNAUTHORIZED": {
                    "codice": "COMM_060",
                    "messaggio": "Autenticazione richiesta",
                    "http_status": 401
                },
                "ERROR_PRODUCT_NOT_FOUND": {
                    "codice": "COMM_061",
                    "messaggio": "Prodotto non trovato",
                    "http_status": 404
                },
                "ERROR_REVIEW_EXISTS": {
                    "codice": "COMM_062",
                    "messaggio": "Hai già recensito questo prodotto",
                    "http_status": 409
                },
                "ERROR_NOT_YOUR_ORDER": {
                    "codice": "COMM_063",
                    "messaggio": "Ordine non valido",
                    "http_status": 403
                }
            }
        }
    },
    
    "estensioni_future": "[EXTENSIBLE] Qui verranno aggiunti edge cases per il modulo COMMERCE"
}



# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 2: COMPOSIZIONI PIATTAFORMA
# ═══════════════════════════════════════════════════════════════════════════════
# Questa sezione descrive come combinare i moduli per creare piattaforme specifiche

COMPOSIZIONI_PIATTAFORMA = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # COMPOSIZIONE: SOCIAL_NETWORK
    # ───────────────────────────────────────────────────────────────────────────
    "SOCIAL_NETWORK": {
        "descrizione": "Piattaforma social tipo Instagram/Twitter",
        "moduli_richiesti": ["IDENTITY", "CONTENT", "SOCIAL", "MESSAGING"],
        "moduli_opzionali": [],
        "entita_principali": [
            "USER", "PROFILE", "POST", "COMMENT", "REACTION",
            "FOLLOW", "BLOCK", "USER_STATS", "FEED_ITEM",
            "CONVERSATION", "MESSAGE", "NOTIFICATION"
        ],
        "flussi_principali": [
            {
                "nome": "Onboarding utente",
                "operazioni": [
                    "REGISTER_USER",
                    "VERIFY_EMAIL",
                    "UPDATE_PROFILE"
                ]
            },
            {
                "nome": "Pubblicazione contenuto",
                "operazioni": [
                    "CREATE_POST",
                    "PUBLISH_POST"
                ]
            },
            {
                "nome": "Interazione sociale",
                "operazioni": [
                    "FOLLOW_USER",
                    "CREATE_COMMENT",
                    "CREATE_REACTION"
                ]
            },
            {
                "nome": "Messaggistica",
                "operazioni": [
                    "START_CONVERSATION",
                    "SEND_MESSAGE"
                ]
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # COMPOSIZIONE: MARKETPLACE_C2C
    # ───────────────────────────────────────────────────────────────────────────
    "MARKETPLACE_C2C": {
        "descrizione": "Marketplace consumer-to-consumer tipo Subito/Vinted",
        "moduli_richiesti": ["IDENTITY", "MESSAGING", "MARKETPLACE"],
        "moduli_opzionali": ["SOCIAL"],
        "entita_principali": [
            "USER", "PROFILE",
            "LISTING", "CATEGORY", "OFFER", "TRANSACTION", "REVIEW",
            "FAVORITE", "SELLER_PROFILE",
            "CONVERSATION", "MESSAGE", "NOTIFICATION"
        ],
        "flussi_principali": [
            {
                "nome": "Pubblica annuncio",
                "operazioni": [
                    "CREATE_LISTING",
                    "PUBLISH_LISTING"
                ]
            },
            {
                "nome": "Acquisto",
                "operazioni": [
                    "SEARCH_LISTINGS",
                    "GET_LISTING",
                    "MAKE_OFFER",
                    "RESPOND_TO_OFFER (accept)"
                ]
            },
            {
                "nome": "Post-vendita",
                "operazioni": [
                    "LEAVE_REVIEW"
                ]
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # COMPOSIZIONE: ECOMMERCE_B2C
    # ───────────────────────────────────────────────────────────────────────────
    "ECOMMERCE_B2C": {
        "descrizione": "E-commerce business-to-consumer tipo Amazon/Zalando",
        "moduli_richiesti": ["IDENTITY", "COMMERCE"],
        "moduli_opzionali": ["MESSAGING", "SOCIAL"],
        "entita_principali": [
            "USER", "PROFILE",
            "PRODUCT", "PRODUCT_VARIANT", "PRODUCT_CATEGORY",
            "CART", "CART_ITEM", "ORDER", "ORDER_ITEM",
            "PRODUCT_REVIEW", "COUPON"
        ],
        "flussi_principali": [
            {
                "nome": "Acquisto",
                "operazioni": [
                    "ADD_TO_CART",
                    "UPDATE_CART_ITEM",
                    "APPLY_COUPON",
                    "CREATE_ORDER"
                ]
            },
            {
                "nome": "Fulfillment",
                "operazioni": [
                    "UPDATE_ORDER_STATUS (processing)",
                    "UPDATE_ORDER_STATUS (shipped)",
                    "UPDATE_ORDER_STATUS (delivered)"
                ]
            },
            {
                "nome": "Post-vendita",
                "operazioni": [
                    "LEAVE_PRODUCT_REVIEW"
                ]
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # COMPOSIZIONE: BOOKING_PLATFORM
    # ───────────────────────────────────────────────────────────────────────────
    "BOOKING_PLATFORM": {
        "descrizione": "Piattaforma prenotazioni tipo Treatwell/Doctolib",
        "moduli_richiesti": ["IDENTITY", "BOOKING", "MESSAGING"],
        "moduli_opzionali": ["SOCIAL"],
        "entita_principali": [
            "USER", "PROFILE",
            "BOOKABLE_RESOURCE", "AVAILABILITY_RULE", "AVAILABILITY_EXCEPTION",
            "BOOKING",
            "CONVERSATION", "MESSAGE", "NOTIFICATION"
        ],
        "flussi_principali": [
            {
                "nome": "Setup fornitore",
                "operazioni": [
                    "CREATE_BOOKABLE_RESOURCE",
                    "SET_AVAILABILITY"
                ]
            },
            {
                "nome": "Prenotazione cliente",
                "operazioni": [
                    "GET_AVAILABLE_SLOTS",
                    "CREATE_BOOKING"
                ]
            },
            {
                "nome": "Gestione prenotazione",
                "operazioni": [
                    "CONFIRM_BOOKING",
                    "CANCEL_BOOKING"
                ]
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # COMPOSIZIONE: FULL_PLATFORM
    # ───────────────────────────────────────────────────────────────────────────
    "FULL_PLATFORM": {
        "descrizione": "Super-app con tutte le funzionalità",
        "moduli_richiesti": ["IDENTITY", "CONTENT", "SOCIAL", "MESSAGING", "MARKETPLACE", "BOOKING", "COMMERCE"],
        "moduli_opzionali": [],
        "note": "Richiede tutte le entità e operazioni di tutti i moduli"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 3: INDICE DI RIFERIMENTO RAPIDO
# ═══════════════════════════════════════════════════════════════════════════════

INDICE_RIFERIMENTO = {
    
    "moduli": {
        "IDENTITY": {
            "entita": ["USER", "SESSION", "PROFILE", "USER_SETTINGS", "VERIFICATION_TOKEN"],
            "operazioni_count": 10,
            "codici_errore_range": "AUTH_001 - AUTH_999"
        },
        "CONTENT": {
            "entita": ["POST", "COMMENT", "REACTION", "MEDIA_ITEM", "HASHTAG", "MENTION"],
            "operazioni_count": 10,
            "codici_errore_range": "CONTENT_001 - CONTENT_999"
        },
        "SOCIAL": {
            "entita": ["FOLLOW", "BLOCK", "USER_STATS", "FEED_ITEM"],
            "operazioni_count": 10,
            "codici_errore_range": "SOCIAL_001 - SOCIAL_999"
        },
        "MESSAGING": {
            "entita": ["CONVERSATION", "CONVERSATION_PARTICIPANT", "MESSAGE", "MESSAGE_READ_RECEIPT", "NOTIFICATION"],
            "operazioni_count": 6,
            "codici_errore_range": "MSG_001 - MSG_999"
        },
        "MARKETPLACE": {
            "entita": ["LISTING", "CATEGORY", "OFFER", "TRANSACTION", "REVIEW", "FAVORITE", "SELLER_PROFILE"],
            "operazioni_count": 12,
            "codici_errore_range": "MARKET_001 - MARKET_999"
        },
        "BOOKING": {
            "entita": ["BOOKABLE_RESOURCE", "AVAILABILITY_RULE", "AVAILABILITY_EXCEPTION", "BOOKING"],
            "operazioni_count": 7,
            "codici_errore_range": "BOOK_001 - BOOK_999"
        },
        "COMMERCE": {
            "entita": ["PRODUCT", "PRODUCT_VARIANT", "CART", "CART_ITEM", "ORDER", "ORDER_ITEM", "PRODUCT_REVIEW"],
            "operazioni_count": 7,
            "codici_errore_range": "COMM_001 - COMM_999"
        }
    },
    
    "entita_totali": 40,
    "operazioni_totali": 62,
    
    "lookup_operazioni_per_attore": {
        "qualsiasi": [
            "GET_LISTING", "SEARCH_LISTINGS", "GET_AVAILABLE_SLOTS",
            "ADD_TO_CART", "UPDATE_CART_ITEM", "APPLY_COUPON",
            "GET_USER_FOLLOWERS", "GET_USER_FOLLOWING", "GET_PROFILE"
        ],
        "utente_autenticato": [
            "UPDATE_PROFILE", "CREATE_POST", "CREATE_COMMENT", "CREATE_REACTION",
            "FOLLOW_USER", "UNFOLLOW_USER", "BLOCK_USER", "UNBLOCK_USER",
            "START_CONVERSATION", "SEND_MESSAGE",
            "CREATE_LISTING", "MAKE_OFFER", "ADD_TO_FAVORITES",
            "CREATE_BOOKABLE_RESOURCE", "CREATE_BOOKING",
            "CREATE_ORDER", "LEAVE_PRODUCT_REVIEW"
        ],
        "proprietario_risorsa": [
            "UPDATE_LISTING", "DELETE_LISTING", "RESPOND_TO_OFFER",
            "SET_AVAILABILITY", "CONFIRM_BOOKING", "CANCEL_BOOKING",
            "UPDATE_ORDER_STATUS"
        ],
        "admin": [
            "Tutte le operazioni con controllo aggiuntivo",
            "Gestione utenti",
            "Moderazione contenuti"
        ]
    },
    
    "lookup_stati_per_entita": {
        "USER": ["pending", "active", "suspended", "deleted"],
        "POST": ["draft", "published", "archived", "deleted"],
        "FOLLOW": ["pending", "active", "rejected"],
        "LISTING": ["draft", "pending_review", "active", "reserved", "sold", "expired", "deleted"],
        "OFFER": ["pending", "accepted", "rejected", "countered", "withdrawn", "expired"],
        "TRANSACTION": ["pending_payment", "paid", "shipped", "delivered", "completed", "disputed", "refunded"],
        "BOOKING": ["pending", "confirmed", "cancelled", "completed", "no_show"],
        "ORDER": ["pending", "confirmed", "processing", "shipped", "delivered", "completed", "cancelled", "refunded"]
    },
    
    "convenzioni_codici_errore": {
        "formato": "MODULO_XXX",
        "range_standard": {
            "001-009": "Errori autenticazione/autorizzazione",
            "010-019": "Errori validazione input",
            "020-029": "Errori risorse non trovate",
            "030-039": "Errori conflitto/stato",
            "040-049": "Errori business logic",
            "050-059": "Errori esterni (pagamenti, email, etc.)",
            "900-999": "Errori sistema/interni"
        }
    },
    
    "convenzioni_http_status": {
        "200": "Operazione completata con successo",
        "201": "Risorsa creata con successo",
        "400": "Input non valido / Business rule violation",
        "401": "Non autenticato",
        "403": "Non autorizzato",
        "404": "Risorsa non trovata",
        "409": "Conflitto (risorsa già esistente, stato non valido)",
        "429": "Limite raggiunto (rate limit, max items, etc.)",
        "500": "Errore interno server"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO REQUISITI FUNZIONALI v1.0
# ═══════════════════════════════════════════════════════════════════════════════
# 
# STATISTICHE FINALI:
# - Moduli: 7
# - Entità: 40
# - Operazioni: 62
# - Composizioni piattaforma: 5
#
# UTILIZZO:
# 1. Leggere SEZIONE 0 per istruzioni AI
# 2. Selezionare composizione piattaforma da SEZIONE 2
# 3. Implementare moduli richiesti
# 4. Usare SEZIONE 3 per riferimenti rapidi
#
# ESTENSIBILITÀ:
# Ogni modulo contiene marcatori [EXTENSIBLE] dove aggiungere edge cases
# durante l'implementazione reale.
#
# ═══════════════════════════════════════════════════════════════════════════════

