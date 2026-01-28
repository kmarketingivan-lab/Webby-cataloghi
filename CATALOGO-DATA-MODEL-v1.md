# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO DATA MODEL v1.0
# ═══════════════════════════════════════════════════════════════════════════════
# 
# SCOPO: Schemi database per tutte le entità del CATALOGO-REQUISITI-FUNZIONALI
# ARCHITETTURE: DynamoDB (serverless) + Aurora PostgreSQL (traditional)
# DIPENDENZA: CATALOGO-REQUISITI-FUNZIONALI-v1.md
#
# ═══════════════════════════════════════════════════════════════════════════════

# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 0: ISTRUZIONI PER AI
# ═══════════════════════════════════════════════════════════════════════════════

"""
COME USARE QUESTO CATALOGO:

1. SELEZIONE ARCHITETTURA
   - DynamoDB: Per architetture serverless, scalabilità automatica, costi pay-per-use
   - Aurora PostgreSQL: Per query complesse, transazioni ACID, reporting

2. STRUTTURA PER OGNI MODULO
   - Schema DynamoDB: Single-table design con PK/SK patterns
   - Schema Aurora: Tabelle relazionali normalizzate
   - Access Patterns: Query comuni con implementazione per entrambi
   - Indici: GSI per DynamoDB, B-tree/GIN per Aurora

3. CONVENZIONI DYNAMODB
   - PK: Partition Key (sempre stringa)
   - SK: Sort Key (sempre stringa)
   - GSI: Global Secondary Index
   - Formato PK: ENTITY#id
   - Formato SK: METADATA | RELATION#timestamp | etc.

4. CONVENZIONI AURORA
   - snake_case per nomi tabelle e colonne
   - UUID per chiavi primarie
   - TIMESTAMPTZ per date/time
   - JSONB per dati semi-strutturati
"""

# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 1: CONFIGURAZIONE GLOBALE
# ═══════════════════════════════════════════════════════════════════════════════

CONFIGURAZIONE_DYNAMODB = {
    "table_name": "MainTable",
    "billing_mode": "PAY_PER_REQUEST",
    "point_in_time_recovery": True,
    "encryption": "AWS_OWNED_KMS",
    
    "primary_key": {
        "PK": {"type": "S", "description": "Partition Key"},
        "SK": {"type": "S", "description": "Sort Key"}
    },
    
    "global_secondary_indexes": [
        {
            "name": "GSI1",
            "key": {"GSI1PK": "S", "GSI1SK": "S"},
            "projection": "ALL",
            "description": "Indice invertito per query inverse"
        },
        {
            "name": "GSI2",
            "key": {"GSI2PK": "S", "GSI2SK": "S"},
            "projection": "ALL",
            "description": "Indice per query temporali/filtri"
        },
        {
            "name": "GSI3",
            "key": {"GSI3PK": "S", "GSI3SK": "S"},
            "projection": "KEYS_ONLY",
            "description": "Indice sparse per lookup specifici"
        }
    ],
    
    "ttl_attribute": "expires_at_epoch"
}

CONFIGURAZIONE_AURORA = {
    "engine": "aurora-postgresql",
    "engine_version": "15.4",
    "instance_class": "db.r6g.large",
    "storage_encrypted": True,
    
    "extensions": [
        "uuid-ossp",      # Generazione UUID
        "pgcrypto",       # Funzioni crittografiche
        "pg_trgm",        # Full-text search trigram
        "btree_gin"       # Indici GIN per tipi scalari
    ],
    
    "common_columns": """
        id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
        created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
        updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
    """,
    
    "audit_trigger": """
        CREATE OR REPLACE FUNCTION update_updated_at()
        RETURNS TRIGGER AS $$
        BEGIN
            NEW.updated_at = NOW();
            RETURN NEW;
        END;
        $$ LANGUAGE plpgsql;
    """
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO IDENTITY - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

IDENTITY_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: USER
    # ───────────────────────────────────────────────────────────────────────────
    "USER": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "METADATA",
            "entity_type": "USER",
            
            # Attributi
            "user_id": "string (UUID)",
            "email": "string",
            "email_verified": "boolean",
            "phone": "string | null",
            "phone_verified": "boolean",
            "password_hash": "string",
            "status": "string (pending|active|suspended|deleted)",
            "role": "string (user|moderator|admin)",
            "last_login_at": "string (ISO8601)",
            "login_count": "number",
            "failed_login_attempts": "number",
            "locked_until": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "EMAIL#<email>",
            "GSI1SK": "USER",
            "GSI2PK": "STATUS#<status>",
            "GSI2SK": "<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova utente per ID",
                "query": "PK = USER#<user_id>, SK = METADATA"
            },
            {
                "pattern": "Trova utente per email",
                "query": "GSI1: GSI1PK = EMAIL#<email>, GSI1SK = USER"
            },
            {
                "pattern": "Lista utenti per status",
                "query": "GSI2: GSI2PK = STATUS#<status>, ordina per GSI2SK"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: PROFILE
    # ───────────────────────────────────────────────────────────────────────────
    "PROFILE": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "PROFILE",
            "entity_type": "PROFILE",
            
            "user_id": "string (UUID)",
            "username": "string",
            "display_name": "string",
            "bio": "string | null",
            "avatar_url": "string | null",
            "cover_url": "string | null",
            "website": "string | null",
            "location": "string | null",
            "date_of_birth": "string (YYYY-MM-DD) | null",
            "gender": "string | null",
            "is_private": "boolean",
            "is_verified": "boolean",
            "metadata": "map",
            
            # GSI Keys
            "GSI1PK": "USERNAME#<username>",
            "GSI1SK": "PROFILE"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova profilo per user_id",
                "query": "PK = USER#<user_id>, SK = PROFILE"
            },
            {
                "pattern": "Trova profilo per username",
                "query": "GSI1: GSI1PK = USERNAME#<username>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: SESSION
    # ───────────────────────────────────────────────────────────────────────────
    "SESSION": {
        "item_structure": {
            "PK": "SESSION#<session_id>",
            "SK": "METADATA",
            "entity_type": "SESSION",
            
            "session_id": "string (UUID)",
            "user_id": "string (UUID)",
            "device_info": "map {device_type, os, browser, app_version}",
            "ip_address": "string",
            "location": "map {country, city, lat, lng}",
            "is_active": "boolean",
            "last_activity_at": "string (ISO8601)",
            "expires_at": "string (ISO8601)",
            "created_at": "string (ISO8601)",
            
            # TTL
            "expires_at_epoch": "number (Unix timestamp)",
            
            # GSI Keys
            "GSI1PK": "USER#<user_id>",
            "GSI1SK": "SESSION#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova sessione per ID",
                "query": "PK = SESSION#<session_id>"
            },
            {
                "pattern": "Lista sessioni utente",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with SESSION#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: USER_SETTINGS
    # ───────────────────────────────────────────────────────────────────────────
    "USER_SETTINGS": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "SETTINGS",
            "entity_type": "USER_SETTINGS",
            
            "user_id": "string (UUID)",
            "language": "string (it|en|...)",
            "timezone": "string (Europe/Rome|...)",
            "theme": "string (light|dark|system)",
            "notifications": "map {email, push, sms, in_app}",
            "privacy": "map {show_online_status, show_last_seen, ...}",
            "preferences": "map"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova settings utente",
                "query": "PK = USER#<user_id>, SK = SETTINGS"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: VERIFICATION_TOKEN
    # ───────────────────────────────────────────────────────────────────────────
    "VERIFICATION_TOKEN": {
        "item_structure": {
            "PK": "VERIFY#<token>",
            "SK": "METADATA",
            "entity_type": "VERIFICATION_TOKEN",
            
            "token": "string",
            "user_id": "string (UUID)",
            "type": "string (email|phone|password_reset|...)",
            "target": "string (email o phone)",
            "code": "string (6 digits) | null",
            "attempts": "number",
            "max_attempts": "number",
            "is_used": "boolean",
            "expires_at": "string (ISO8601)",
            "created_at": "string (ISO8601)",
            
            # TTL
            "expires_at_epoch": "number",
            
            # GSI Keys
            "GSI1PK": "USER#<user_id>",
            "GSI1SK": "VERIFY#<type>#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova token per valore",
                "query": "PK = VERIFY#<token>"
            },
            {
                "pattern": "Lista token utente per tipo",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with VERIFY#<type>#"
            }
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# IDENTITY - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

IDENTITY_AURORA = {
    
    "tables": {
        
        "users": """
            CREATE TABLE users (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                email VARCHAR(255) NOT NULL,
                email_verified BOOLEAN NOT NULL DEFAULT FALSE,
                phone VARCHAR(20),
                phone_verified BOOLEAN NOT NULL DEFAULT FALSE,
                password_hash VARCHAR(255) NOT NULL,
                status VARCHAR(20) NOT NULL DEFAULT 'pending' 
                    CHECK (status IN ('pending', 'active', 'suspended', 'deleted')),
                role VARCHAR(20) NOT NULL DEFAULT 'user'
                    CHECK (role IN ('user', 'moderator', 'admin')),
                last_login_at TIMESTAMPTZ,
                login_count INTEGER NOT NULL DEFAULT 0,
                failed_login_attempts INTEGER NOT NULL DEFAULT 0,
                locked_until TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT users_email_unique UNIQUE (email),
                CONSTRAINT users_phone_unique UNIQUE (phone)
            );
            
            CREATE INDEX idx_users_email ON users(email);
            CREATE INDEX idx_users_status ON users(status);
            CREATE INDEX idx_users_created_at ON users(created_at);
            
            CREATE TRIGGER users_updated_at
                BEFORE UPDATE ON users
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "profiles": """
            CREATE TABLE profiles (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                username VARCHAR(30) NOT NULL,
                display_name VARCHAR(100),
                bio TEXT,
                avatar_url VARCHAR(500),
                cover_url VARCHAR(500),
                website VARCHAR(255),
                location VARCHAR(100),
                date_of_birth DATE,
                gender VARCHAR(20),
                is_private BOOLEAN NOT NULL DEFAULT FALSE,
                is_verified BOOLEAN NOT NULL DEFAULT FALSE,
                metadata JSONB DEFAULT '{}',
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT profiles_user_id_unique UNIQUE (user_id),
                CONSTRAINT profiles_username_unique UNIQUE (username)
            );
            
            CREATE INDEX idx_profiles_username ON profiles(username);
            CREATE INDEX idx_profiles_username_trgm ON profiles USING gin(username gin_trgm_ops);
            CREATE INDEX idx_profiles_display_name_trgm ON profiles USING gin(display_name gin_trgm_ops);
            
            CREATE TRIGGER profiles_updated_at
                BEFORE UPDATE ON profiles
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "sessions": """
            CREATE TABLE sessions (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                device_info JSONB DEFAULT '{}',
                ip_address INET,
                location JSONB,
                is_active BOOLEAN NOT NULL DEFAULT TRUE,
                last_activity_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                expires_at TIMESTAMPTZ NOT NULL,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT sessions_expires_check CHECK (expires_at > created_at)
            );
            
            CREATE INDEX idx_sessions_user_id ON sessions(user_id);
            CREATE INDEX idx_sessions_expires_at ON sessions(expires_at);
            CREATE INDEX idx_sessions_active ON sessions(user_id, is_active) WHERE is_active = TRUE;
        """,
        
        "user_settings": """
            CREATE TABLE user_settings (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                language VARCHAR(10) NOT NULL DEFAULT 'it',
                timezone VARCHAR(50) NOT NULL DEFAULT 'Europe/Rome',
                theme VARCHAR(20) NOT NULL DEFAULT 'system'
                    CHECK (theme IN ('light', 'dark', 'system')),
                notifications JSONB NOT NULL DEFAULT '{
                    "email": true,
                    "push": true,
                    "sms": false,
                    "in_app": true
                }',
                privacy JSONB NOT NULL DEFAULT '{
                    "show_online_status": true,
                    "show_last_seen": true
                }',
                preferences JSONB DEFAULT '{}',
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT user_settings_user_id_unique UNIQUE (user_id)
            );
            
            CREATE TRIGGER user_settings_updated_at
                BEFORE UPDATE ON user_settings
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "verification_tokens": """
            CREATE TABLE verification_tokens (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                token VARCHAR(255) NOT NULL,
                type VARCHAR(30) NOT NULL 
                    CHECK (type IN ('email', 'phone', 'password_reset', 'two_factor')),
                target VARCHAR(255) NOT NULL,
                code VARCHAR(10),
                attempts INTEGER NOT NULL DEFAULT 0,
                max_attempts INTEGER NOT NULL DEFAULT 3,
                is_used BOOLEAN NOT NULL DEFAULT FALSE,
                expires_at TIMESTAMPTZ NOT NULL,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT verification_tokens_token_unique UNIQUE (token)
            );
            
            CREATE INDEX idx_verification_tokens_user_type ON verification_tokens(user_id, type);
            CREATE INDEX idx_verification_tokens_expires ON verification_tokens(expires_at);
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO CONTENT - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

CONTENT_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: POST
    # ───────────────────────────────────────────────────────────────────────────
    "POST": {
        "item_structure": {
            "PK": "POST#<post_id>",
            "SK": "METADATA",
            "entity_type": "POST",
            
            "post_id": "string (UUID)",
            "author_id": "string (UUID)",
            "post_type": "string (text|image|video|link|poll)",
            "content": "string",
            "media": "list [{id, type, url, thumbnail_url, alt_text, order}]",
            "link_preview": "map {url, title, description, image_url, domain}",
            "poll": "map {question, options[], votes_count, ends_at}",
            "hashtags": "list [string]",
            "mentions": "list [{user_id, username, position}]",
            "location": "map {name, lat, lng}",
            "status": "string (draft|published|archived|deleted)",
            "visibility": "string (public|followers|private)",
            "comments_count": "number",
            "reactions_count": "number",
            "shares_count": "number",
            "views_count": "number",
            "is_pinned": "boolean",
            "is_comments_disabled": "boolean",
            "scheduled_at": "string (ISO8601) | null",
            "published_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "USER#<author_id>",
            "GSI1SK": "POST#<published_at>",
            "GSI2PK": "STATUS#<status>",
            "GSI2SK": "<published_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova post per ID",
                "query": "PK = POST#<post_id>"
            },
            {
                "pattern": "Lista post utente (cronologico)",
                "query": "GSI1: GSI1PK = USER#<author_id>, SK begins_with POST#, ScanIndexForward=false"
            },
            {
                "pattern": "Lista post per hashtag",
                "nota": "Usa tabella separata HASHTAG_POSTS o GSI sparse"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: COMMENT
    # ───────────────────────────────────────────────────────────────────────────
    "COMMENT": {
        "item_structure": {
            "PK": "POST#<post_id>",
            "SK": "COMMENT#<created_at>#<comment_id>",
            "entity_type": "COMMENT",
            
            "comment_id": "string (UUID)",
            "post_id": "string (UUID)",
            "author_id": "string (UUID)",
            "parent_id": "string (UUID) | null",
            "content": "string",
            "media": "list",
            "mentions": "list",
            "reactions_count": "number",
            "replies_count": "number",
            "is_edited": "boolean",
            "is_deleted": "boolean",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # Per threading (risposte a commenti)
            "thread_path": "string (root_id/parent_id/comment_id)",
            
            # GSI Keys
            "GSI1PK": "USER#<author_id>",
            "GSI1SK": "COMMENT#<created_at>",
            "GSI2PK": "COMMENT#<comment_id>",
            "GSI2SK": "METADATA"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista commenti post (cronologico)",
                "query": "PK = POST#<post_id>, SK begins_with COMMENT#"
            },
            {
                "pattern": "Trova commento per ID",
                "query": "GSI2: GSI2PK = COMMENT#<comment_id>"
            },
            {
                "pattern": "Lista commenti utente",
                "query": "GSI1: GSI1PK = USER#<author_id>, SK begins_with COMMENT#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: REACTION
    # ───────────────────────────────────────────────────────────────────────────
    "REACTION": {
        "item_structure": {
            "PK": "<target_type>#<target_id>",  # POST#xxx o COMMENT#xxx
            "SK": "REACTION#<user_id>",
            "entity_type": "REACTION",
            
            "reaction_id": "string (UUID)",
            "user_id": "string (UUID)",
            "target_type": "string (post|comment)",
            "target_id": "string (UUID)",
            "reaction_type": "string (like|love|haha|wow|sad|angry)",
            "created_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "USER#<user_id>",
            "GSI1SK": "REACTION#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Verifica se utente ha reagito",
                "query": "PK = POST#<post_id>, SK = REACTION#<user_id>"
            },
            {
                "pattern": "Lista reazioni per target",
                "query": "PK = POST#<post_id>, SK begins_with REACTION#"
            },
            {
                "pattern": "Lista reazioni utente",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with REACTION#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: MEDIA_ITEM
    # ───────────────────────────────────────────────────────────────────────────
    "MEDIA_ITEM": {
        "item_structure": {
            "PK": "MEDIA#<media_id>",
            "SK": "METADATA",
            "entity_type": "MEDIA_ITEM",
            
            "media_id": "string (UUID)",
            "uploader_id": "string (UUID)",
            "file_type": "string (image|video|audio|document)",
            "mime_type": "string",
            "original_filename": "string",
            "file_size_bytes": "number",
            "storage_key": "string (S3 key)",
            "url": "string (CloudFront URL)",
            "thumbnail_url": "string | null",
            "dimensions": "map {width, height}",
            "duration_seconds": "number | null",
            "alt_text": "string | null",
            "processing_status": "string (pending|processing|completed|failed)",
            "is_public": "boolean",
            "created_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "USER#<uploader_id>",
            "GSI1SK": "MEDIA#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova media per ID",
                "query": "PK = MEDIA#<media_id>"
            },
            {
                "pattern": "Lista media utente",
                "query": "GSI1: GSI1PK = USER#<uploader_id>, SK begins_with MEDIA#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: HASHTAG (tabella separata per conteggi)
    # ───────────────────────────────────────────────────────────────────────────
    "HASHTAG": {
        "item_structure": {
            "PK": "HASHTAG#<hashtag_normalized>",
            "SK": "METADATA",
            "entity_type": "HASHTAG",
            
            "hashtag": "string (originale)",
            "hashtag_normalized": "string (lowercase)",
            "posts_count": "number",
            "trending_score": "number",
            "last_used_at": "string (ISO8601)",
            "created_at": "string (ISO8601)"
        },
        
        "hashtag_post_relation": {
            "PK": "HASHTAG#<hashtag_normalized>",
            "SK": "POST#<published_at>#<post_id>",
            "entity_type": "HASHTAG_POST",
            "post_id": "string",
            "author_id": "string"
        }
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# CONTENT - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

CONTENT_AURORA = {
    
    "tables": {
        
        "posts": """
            CREATE TABLE posts (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                author_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                post_type VARCHAR(20) NOT NULL DEFAULT 'text'
                    CHECK (post_type IN ('text', 'image', 'video', 'link', 'poll')),
                content TEXT,
                media JSONB DEFAULT '[]',
                link_preview JSONB,
                poll JSONB,
                hashtags TEXT[] DEFAULT '{}',
                mentions JSONB DEFAULT '[]',
                location JSONB,
                status VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'published', 'archived', 'deleted')),
                visibility VARCHAR(20) NOT NULL DEFAULT 'public'
                    CHECK (visibility IN ('public', 'followers', 'private')),
                comments_count INTEGER NOT NULL DEFAULT 0,
                reactions_count INTEGER NOT NULL DEFAULT 0,
                shares_count INTEGER NOT NULL DEFAULT 0,
                views_count INTEGER NOT NULL DEFAULT 0,
                is_pinned BOOLEAN NOT NULL DEFAULT FALSE,
                is_comments_disabled BOOLEAN NOT NULL DEFAULT FALSE,
                scheduled_at TIMESTAMPTZ,
                published_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_posts_author_id ON posts(author_id);
            CREATE INDEX idx_posts_author_published ON posts(author_id, published_at DESC) 
                WHERE status = 'published';
            CREATE INDEX idx_posts_status_published ON posts(status, published_at DESC);
            CREATE INDEX idx_posts_hashtags ON posts USING gin(hashtags);
            CREATE INDEX idx_posts_content_trgm ON posts USING gin(content gin_trgm_ops);
            
            CREATE TRIGGER posts_updated_at
                BEFORE UPDATE ON posts
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "comments": """
            CREATE TABLE comments (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
                author_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
                content TEXT NOT NULL,
                media JSONB DEFAULT '[]',
                mentions JSONB DEFAULT '[]',
                reactions_count INTEGER NOT NULL DEFAULT 0,
                replies_count INTEGER NOT NULL DEFAULT 0,
                is_edited BOOLEAN NOT NULL DEFAULT FALSE,
                is_deleted BOOLEAN NOT NULL DEFAULT FALSE,
                thread_path LTREE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            -- Richiede estensione ltree per threading efficiente
            CREATE EXTENSION IF NOT EXISTS ltree;
            
            CREATE INDEX idx_comments_post_id ON comments(post_id, created_at);
            CREATE INDEX idx_comments_author_id ON comments(author_id);
            CREATE INDEX idx_comments_parent_id ON comments(parent_id);
            CREATE INDEX idx_comments_thread_path ON comments USING gist(thread_path);
            
            CREATE TRIGGER comments_updated_at
                BEFORE UPDATE ON comments
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "reactions": """
            CREATE TABLE reactions (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                target_type VARCHAR(20) NOT NULL CHECK (target_type IN ('post', 'comment')),
                target_id UUID NOT NULL,
                reaction_type VARCHAR(20) NOT NULL DEFAULT 'like'
                    CHECK (reaction_type IN ('like', 'love', 'haha', 'wow', 'sad', 'angry')),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT reactions_unique UNIQUE (user_id, target_type, target_id)
            );
            
            CREATE INDEX idx_reactions_target ON reactions(target_type, target_id);
            CREATE INDEX idx_reactions_user_id ON reactions(user_id);
        """,
        
        "media_items": """
            CREATE TABLE media_items (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                uploader_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                file_type VARCHAR(20) NOT NULL 
                    CHECK (file_type IN ('image', 'video', 'audio', 'document')),
                mime_type VARCHAR(100) NOT NULL,
                original_filename VARCHAR(255) NOT NULL,
                file_size_bytes BIGINT NOT NULL,
                storage_key VARCHAR(500) NOT NULL,
                url VARCHAR(500) NOT NULL,
                thumbnail_url VARCHAR(500),
                dimensions JSONB,
                duration_seconds INTEGER,
                alt_text TEXT,
                processing_status VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (processing_status IN ('pending', 'processing', 'completed', 'failed')),
                is_public BOOLEAN NOT NULL DEFAULT FALSE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_media_items_uploader ON media_items(uploader_id);
            CREATE INDEX idx_media_items_processing ON media_items(processing_status) 
                WHERE processing_status != 'completed';
        """,
        
        "hashtags": """
            CREATE TABLE hashtags (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                hashtag VARCHAR(100) NOT NULL,
                hashtag_normalized VARCHAR(100) NOT NULL,
                posts_count INTEGER NOT NULL DEFAULT 0,
                trending_score DECIMAL(10,2) NOT NULL DEFAULT 0,
                last_used_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT hashtags_normalized_unique UNIQUE (hashtag_normalized)
            );
            
            CREATE INDEX idx_hashtags_trending ON hashtags(trending_score DESC);
            CREATE INDEX idx_hashtags_normalized_trgm ON hashtags USING gin(hashtag_normalized gin_trgm_ops);
        """,
        
        "post_hashtags": """
            CREATE TABLE post_hashtags (
                post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
                hashtag_id UUID NOT NULL REFERENCES hashtags(id) ON DELETE CASCADE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                PRIMARY KEY (post_id, hashtag_id)
            );
            
            CREATE INDEX idx_post_hashtags_hashtag ON post_hashtags(hashtag_id);
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO SOCIAL - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

SOCIAL_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: FOLLOW
    # ───────────────────────────────────────────────────────────────────────────
    "FOLLOW": {
        "item_structure": {
            "PK": "USER#<follower_id>",
            "SK": "FOLLOWING#<following_id>",
            "entity_type": "FOLLOW",
            
            "follower_id": "string (UUID)",
            "following_id": "string (UUID)",
            "status": "string (pending|active|rejected)",
            "created_at": "string (ISO8601)",
            "accepted_at": "string (ISO8601) | null",
            
            # GSI per query inversa (chi mi segue)
            "GSI1PK": "USER#<following_id>",
            "GSI1SK": "FOLLOWER#<follower_id>"
        },
        
        "access_patterns": [
            {
                "pattern": "Verifica se A segue B",
                "query": "PK = USER#<A>, SK = FOLLOWING#<B>"
            },
            {
                "pattern": "Lista following di utente (chi seguo)",
                "query": "PK = USER#<user_id>, SK begins_with FOLLOWING#"
            },
            {
                "pattern": "Lista followers di utente (chi mi segue)",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with FOLLOWER#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: BLOCK
    # ───────────────────────────────────────────────────────────────────────────
    "BLOCK": {
        "item_structure": {
            "PK": "USER#<blocker_id>",
            "SK": "BLOCK#<blocked_id>",
            "entity_type": "BLOCK",
            
            "blocker_id": "string (UUID)",
            "blocked_id": "string (UUID)",
            "reason": "string | null",
            "created_at": "string (ISO8601)",
            
            # GSI per verificare se sono bloccato da qualcuno
            "GSI1PK": "BLOCKED#<blocked_id>",
            "GSI1SK": "BY#<blocker_id>"
        },
        
        "access_patterns": [
            {
                "pattern": "Verifica se A ha bloccato B",
                "query": "PK = USER#<A>, SK = BLOCK#<B>"
            },
            {
                "pattern": "Lista utenti bloccati da me",
                "query": "PK = USER#<user_id>, SK begins_with BLOCK#"
            },
            {
                "pattern": "Verifica se sono bloccato da qualcuno",
                "query": "GSI1: GSI1PK = BLOCKED#<my_id>, SK begins_with BY#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: USER_STATS
    # ───────────────────────────────────────────────────────────────────────────
    "USER_STATS": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "STATS",
            "entity_type": "USER_STATS",
            
            "user_id": "string (UUID)",
            "followers_count": "number",
            "following_count": "number",
            "posts_count": "number",
            "media_count": "number",
            "likes_received_count": "number",
            "comments_received_count": "number",
            "updated_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Ottieni statistiche utente",
                "query": "PK = USER#<user_id>, SK = STATS"
            }
        ],
        
        "note": "Contatori aggiornati atomicamente con UpdateItem ADD/SET"
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: FEED_ITEM
    # ───────────────────────────────────────────────────────────────────────────
    "FEED_ITEM": {
        "item_structure": {
            "PK": "FEED#<user_id>",
            "SK": "<relevance_score>#<timestamp>#<post_id>",
            "entity_type": "FEED_ITEM",
            
            "user_id": "string (UUID)",
            "post_id": "string (UUID)",
            "author_id": "string (UUID)",
            "action_type": "string (post|repost|like)",
            "relevance_score": "number (0-999)",
            "is_seen": "boolean",
            "created_at": "string (ISO8601)",
            
            # TTL per pulizia automatica
            "expires_at_epoch": "number (Unix timestamp, ~7 giorni)"
        },
        
        "access_patterns": [
            {
                "pattern": "Ottieni feed utente (per rilevanza)",
                "query": "PK = FEED#<user_id>, ScanIndexForward=false, Limit=50"
            }
        ],
        
        "note": """
            Feed pre-calcolato (fan-out on write).
            Quando un utente pubblica, inserisce in FEED di tutti i followers.
            Per utenti con molti followers, usare fan-out on read ibrido.
        """
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# SOCIAL - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

SOCIAL_AURORA = {
    
    "tables": {
        
        "follows": """
            CREATE TABLE follows (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                follower_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                following_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                status VARCHAR(20) NOT NULL DEFAULT 'active'
                    CHECK (status IN ('pending', 'active', 'rejected')),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                accepted_at TIMESTAMPTZ,
                
                CONSTRAINT follows_unique UNIQUE (follower_id, following_id),
                CONSTRAINT follows_no_self CHECK (follower_id != following_id)
            );
            
            CREATE INDEX idx_follows_follower ON follows(follower_id) WHERE status = 'active';
            CREATE INDEX idx_follows_following ON follows(following_id) WHERE status = 'active';
            CREATE INDEX idx_follows_pending ON follows(following_id, status) WHERE status = 'pending';
        """,
        
        "blocks": """
            CREATE TABLE blocks (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                blocker_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                blocked_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                reason TEXT,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT blocks_unique UNIQUE (blocker_id, blocked_id),
                CONSTRAINT blocks_no_self CHECK (blocker_id != blocked_id)
            );
            
            CREATE INDEX idx_blocks_blocker ON blocks(blocker_id);
            CREATE INDEX idx_blocks_blocked ON blocks(blocked_id);
        """,
        
        "user_stats": """
            CREATE TABLE user_stats (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                followers_count INTEGER NOT NULL DEFAULT 0,
                following_count INTEGER NOT NULL DEFAULT 0,
                posts_count INTEGER NOT NULL DEFAULT 0,
                media_count INTEGER NOT NULL DEFAULT 0,
                likes_received_count INTEGER NOT NULL DEFAULT 0,
                comments_received_count INTEGER NOT NULL DEFAULT 0,
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT user_stats_user_unique UNIQUE (user_id),
                CONSTRAINT user_stats_positive CHECK (
                    followers_count >= 0 AND 
                    following_count >= 0 AND 
                    posts_count >= 0
                )
            );
            
            -- Funzione per aggiornare contatori atomicamente
            CREATE OR REPLACE FUNCTION update_user_stat(
                p_user_id UUID,
                p_stat VARCHAR,
                p_delta INTEGER
            ) RETURNS VOID AS $$
            BEGIN
                INSERT INTO user_stats (user_id)
                VALUES (p_user_id)
                ON CONFLICT (user_id) DO NOTHING;
                
                EXECUTE format(
                    'UPDATE user_stats SET %I = GREATEST(0, %I + $1), updated_at = NOW() WHERE user_id = $2',
                    p_stat, p_stat
                ) USING p_delta, p_user_id;
            END;
            $$ LANGUAGE plpgsql;
        """,
        
        "feed_items": """
            -- Tabella per feed pre-calcolato
            CREATE TABLE feed_items (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                post_id UUID NOT NULL REFERENCES posts(id) ON DELETE CASCADE,
                author_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                action_type VARCHAR(20) NOT NULL DEFAULT 'post'
                    CHECK (action_type IN ('post', 'repost', 'like')),
                relevance_score INTEGER NOT NULL DEFAULT 500,
                is_seen BOOLEAN NOT NULL DEFAULT FALSE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '7 days'
            );
            
            -- Indice principale per query feed
            CREATE INDEX idx_feed_items_user_relevance ON feed_items(user_id, relevance_score DESC, created_at DESC);
            
            -- Indice per pulizia periodica
            CREATE INDEX idx_feed_items_expires ON feed_items(expires_at);
            
            -- Previene duplicati
            CREATE UNIQUE INDEX idx_feed_items_unique ON feed_items(user_id, post_id);
            
            -- Pulizia automatica con pg_cron o job esterno
            -- DELETE FROM feed_items WHERE expires_at < NOW();
        """,
        
        "feed_generation_views": """
            -- Vista materializzata per feed on-demand (alternativa a pre-calcolo)
            CREATE MATERIALIZED VIEW user_feed_view AS
            SELECT 
                f.follower_id as user_id,
                p.id as post_id,
                p.author_id,
                'post' as action_type,
                COALESCE(
                    500 + 
                    (EXTRACT(EPOCH FROM (NOW() - p.published_at)) / 3600 * -1)::int +
                    (p.reactions_count * 2) +
                    (p.comments_count * 3),
                    500
                )::int as relevance_score,
                p.published_at as created_at
            FROM follows f
            JOIN posts p ON p.author_id = f.following_id
            WHERE f.status = 'active'
            AND p.status = 'published'
            AND p.published_at > NOW() - INTERVAL '7 days';
            
            CREATE INDEX idx_user_feed_view ON user_feed_view(user_id, relevance_score DESC);
            
            -- Refresh periodico
            -- REFRESH MATERIALIZED VIEW CONCURRENTLY user_feed_view;
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO MESSAGING - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

MESSAGING_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: CONVERSATION
    # ───────────────────────────────────────────────────────────────────────────
    "CONVERSATION": {
        "item_structure": {
            "PK": "CONV#<conversation_id>",
            "SK": "METADATA",
            "entity_type": "CONVERSATION",
            
            "conversation_id": "string (UUID)",
            "type": "string (direct|group)",
            "name": "string | null",
            "avatar_url": "string | null",
            "created_by": "string (UUID)",
            "participants_count": "number",
            "last_message": "map {id, sender_id, content_preview, sent_at}",
            "last_activity_at": "string (ISO8601)",
            "is_archived": "boolean",
            "settings": "map {mute_until, pin_order}",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova conversazione per ID",
                "query": "PK = CONV#<conversation_id>, SK = METADATA"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: CONVERSATION_PARTICIPANT
    # ───────────────────────────────────────────────────────────────────────────
    "CONVERSATION_PARTICIPANT": {
        "item_structure": {
            "PK": "CONV#<conversation_id>",
            "SK": "PARTICIPANT#<user_id>",
            "entity_type": "CONVERSATION_PARTICIPANT",
            
            "conversation_id": "string (UUID)",
            "user_id": "string (UUID)",
            "role": "string (member|admin|owner)",
            "nickname": "string | null",
            "joined_at": "string (ISO8601)",
            "last_read_at": "string (ISO8601)",
            "last_read_message_id": "string (UUID) | null",
            "unread_count": "number",
            "is_muted": "boolean",
            "muted_until": "string (ISO8601) | null",
            
            # GSI per lista conversazioni utente
            "GSI1PK": "USER#<user_id>",
            "GSI1SK": "CONV#<last_activity_at>"  # ordinato per attività recente
        },
        
        "access_patterns": [
            {
                "pattern": "Lista partecipanti conversazione",
                "query": "PK = CONV#<conversation_id>, SK begins_with PARTICIPANT#"
            },
            {
                "pattern": "Lista conversazioni utente (recenti prima)",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with CONV#, ScanIndexForward=false"
            },
            {
                "pattern": "Verifica se utente è in conversazione",
                "query": "PK = CONV#<conversation_id>, SK = PARTICIPANT#<user_id>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: MESSAGE
    # ───────────────────────────────────────────────────────────────────────────
    "MESSAGE": {
        "item_structure": {
            "PK": "CONV#<conversation_id>",
            "SK": "MSG#<sent_at>#<message_id>",
            "entity_type": "MESSAGE",
            
            "message_id": "string (UUID)",
            "conversation_id": "string (UUID)",
            "sender_id": "string (UUID)",
            "message_type": "string (text|image|video|audio|file|location|system)",
            "content": "string | null",
            "media": "map {url, thumbnail_url, file_name, file_size, mime_type}",
            "reply_to_id": "string (UUID) | null",
            "reply_preview": "map {sender_id, content_preview}",
            "mentions": "list [{user_id, username}]",
            "reactions": "map {emoji: [user_ids]}",
            "is_edited": "boolean",
            "is_deleted": "boolean",
            "deleted_for": "list [user_ids]",
            "sent_at": "string (ISO8601)",
            "edited_at": "string (ISO8601) | null",
            
            # GSI per cercare messaggio specifico
            "GSI2PK": "MSG#<message_id>",
            "GSI2SK": "METADATA"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista messaggi conversazione (cronologico)",
                "query": "PK = CONV#<conversation_id>, SK begins_with MSG#, ScanIndexForward=true"
            },
            {
                "pattern": "Messaggi recenti (paginazione inversa)",
                "query": "PK = CONV#<conversation_id>, SK begins_with MSG#, ScanIndexForward=false, Limit=50"
            },
            {
                "pattern": "Trova messaggio per ID",
                "query": "GSI2: GSI2PK = MSG#<message_id>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: MESSAGE_READ_RECEIPT (opzionale, per gruppi grandi evitare)
    # ───────────────────────────────────────────────────────────────────────────
    "MESSAGE_READ_RECEIPT": {
        "item_structure": {
            "PK": "MSG#<message_id>",
            "SK": "READ#<user_id>",
            "entity_type": "MESSAGE_READ_RECEIPT",
            
            "message_id": "string (UUID)",
            "user_id": "string (UUID)",
            "read_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista chi ha letto un messaggio",
                "query": "PK = MSG#<message_id>, SK begins_with READ#"
            }
        ],
        
        "note": "Per conversazioni 1:1, usare solo last_read_message_id nel PARTICIPANT"
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: NOTIFICATION
    # ───────────────────────────────────────────────────────────────────────────
    "NOTIFICATION": {
        "item_structure": {
            "PK": "USER#<recipient_id>",
            "SK": "NOTIF#<created_at>#<notification_id>",
            "entity_type": "NOTIFICATION",
            
            "notification_id": "string (UUID)",
            "recipient_id": "string (UUID)",
            "type": "string (follow|like|comment|mention|message|...)",
            "title": "string",
            "body": "string",
            "source_user_id": "string (UUID) | null",
            "target_type": "string (post|comment|user|conversation|...)",
            "target_id": "string (UUID)",
            "action_url": "string",
            "metadata": "map",
            "is_read": "boolean",
            "read_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            
            # TTL
            "expires_at_epoch": "number (Unix timestamp, ~90 giorni)",
            
            # GSI per contare non lette
            "GSI1PK": "USER#<recipient_id>#UNREAD",
            "GSI1SK": "<created_at>"  # Solo se is_read = false
        },
        
        "access_patterns": [
            {
                "pattern": "Lista notifiche utente (recenti prima)",
                "query": "PK = USER#<recipient_id>, SK begins_with NOTIF#, ScanIndexForward=false"
            },
            {
                "pattern": "Conta notifiche non lette",
                "query": "GSI1: GSI1PK = USER#<recipient_id>#UNREAD, Select=COUNT"
            }
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# MESSAGING - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

MESSAGING_AURORA = {
    
    "tables": {
        
        "conversations": """
            CREATE TABLE conversations (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                type VARCHAR(20) NOT NULL DEFAULT 'direct'
                    CHECK (type IN ('direct', 'group')),
                name VARCHAR(100),
                avatar_url VARCHAR(500),
                created_by UUID NOT NULL REFERENCES users(id),
                participants_count INTEGER NOT NULL DEFAULT 0,
                last_message_id UUID,
                last_message_preview TEXT,
                last_message_sender_id UUID,
                last_activity_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                is_archived BOOLEAN NOT NULL DEFAULT FALSE,
                settings JSONB DEFAULT '{}',
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_conversations_last_activity ON conversations(last_activity_at DESC);
            
            CREATE TRIGGER conversations_updated_at
                BEFORE UPDATE ON conversations
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "conversation_participants": """
            CREATE TABLE conversation_participants (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                role VARCHAR(20) NOT NULL DEFAULT 'member'
                    CHECK (role IN ('member', 'admin', 'owner')),
                nickname VARCHAR(50),
                joined_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                last_read_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                last_read_message_id UUID,
                unread_count INTEGER NOT NULL DEFAULT 0,
                is_muted BOOLEAN NOT NULL DEFAULT FALSE,
                muted_until TIMESTAMPTZ,
                
                CONSTRAINT conv_participants_unique UNIQUE (conversation_id, user_id)
            );
            
            -- Indice principale per lista conversazioni utente
            CREATE INDEX idx_conv_participants_user ON conversation_participants(user_id);
            
            -- Join efficiente per lista conversazioni con dettagli
            CREATE INDEX idx_conv_participants_user_conv ON conversation_participants(user_id, conversation_id);
            
            -- Query per trovare conversazione diretta tra due utenti
            CREATE INDEX idx_conv_participants_conv ON conversation_participants(conversation_id);
        """,
        
        "messages": """
            CREATE TABLE messages (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
                sender_id UUID NOT NULL REFERENCES users(id),
                message_type VARCHAR(20) NOT NULL DEFAULT 'text'
                    CHECK (message_type IN ('text', 'image', 'video', 'audio', 'file', 'location', 'system')),
                content TEXT,
                media JSONB,
                reply_to_id UUID REFERENCES messages(id),
                reply_preview JSONB,
                mentions JSONB DEFAULT '[]',
                reactions JSONB DEFAULT '{}',
                is_edited BOOLEAN NOT NULL DEFAULT FALSE,
                is_deleted BOOLEAN NOT NULL DEFAULT FALSE,
                deleted_for UUID[] DEFAULT '{}',
                sent_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                edited_at TIMESTAMPTZ
            );
            
            -- Indice principale per lista messaggi (paginazione)
            CREATE INDEX idx_messages_conv_sent ON messages(conversation_id, sent_at DESC);
            
            -- Per ricerca full-text nei messaggi
            CREATE INDEX idx_messages_content_trgm ON messages USING gin(content gin_trgm_ops);
            
            -- Trigger per aggiornare last_message in conversations
            CREATE OR REPLACE FUNCTION update_conversation_last_message()
            RETURNS TRIGGER AS $$
            BEGIN
                UPDATE conversations SET
                    last_message_id = NEW.id,
                    last_message_preview = LEFT(NEW.content, 100),
                    last_message_sender_id = NEW.sender_id,
                    last_activity_at = NEW.sent_at,
                    updated_at = NOW()
                WHERE id = NEW.conversation_id;
                
                -- Incrementa unread_count per tutti tranne sender
                UPDATE conversation_participants SET
                    unread_count = unread_count + 1
                WHERE conversation_id = NEW.conversation_id
                AND user_id != NEW.sender_id;
                
                RETURN NEW;
            END;
            $$ LANGUAGE plpgsql;
            
            CREATE TRIGGER messages_after_insert
                AFTER INSERT ON messages
                FOR EACH ROW EXECUTE FUNCTION update_conversation_last_message();
        """,
        
        "message_read_receipts": """
            CREATE TABLE message_read_receipts (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                message_id UUID NOT NULL REFERENCES messages(id) ON DELETE CASCADE,
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                read_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT message_read_receipts_unique UNIQUE (message_id, user_id)
            );
            
            CREATE INDEX idx_message_read_receipts_message ON message_read_receipts(message_id);
        """,
        
        "notifications": """
            CREATE TABLE notifications (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                recipient_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                type VARCHAR(50) NOT NULL,
                title VARCHAR(255) NOT NULL,
                body TEXT,
                source_user_id UUID REFERENCES users(id) ON DELETE SET NULL,
                target_type VARCHAR(50),
                target_id UUID,
                action_url VARCHAR(500),
                metadata JSONB DEFAULT '{}',
                is_read BOOLEAN NOT NULL DEFAULT FALSE,
                read_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '90 days'
            );
            
            -- Indice principale per lista notifiche
            CREATE INDEX idx_notifications_recipient ON notifications(recipient_id, created_at DESC);
            
            -- Indice per conteggio non lette
            CREATE INDEX idx_notifications_unread ON notifications(recipient_id, is_read) 
                WHERE is_read = FALSE;
            
            -- Indice per pulizia periodica
            CREATE INDEX idx_notifications_expires ON notifications(expires_at);
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO MARKETPLACE - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

MARKETPLACE_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: LISTING
    # ───────────────────────────────────────────────────────────────────────────
    "LISTING": {
        "item_structure": {
            "PK": "LISTING#<listing_id>",
            "SK": "METADATA",
            "entity_type": "LISTING",
            
            "listing_id": "string (UUID)",
            "seller_id": "string (UUID)",
            "listing_type": "string (product|service|job|housing|vehicle)",
            "title": "string",
            "description": "string",
            "category_id": "string (UUID)",
            "subcategory_id": "string (UUID) | null",
            "price": "map {amount, currency, type, is_negotiable}",
            "condition": "string (new|like_new|good|fair|for_parts) | null",
            "media": "list [{id, type, url, thumbnail_url, order, is_cover}]",
            "location": "map {city, region, country, lat, lng, address}",
            "attributes": "map (categoria-specifico)",
            "shipping_options": "list [{method, price, estimated_days}]",
            "status": "string (draft|pending_review|active|reserved|sold|expired|deleted)",
            "visibility": "string (public|unlisted|private)",
            "views_count": "number",
            "favorites_count": "number",
            "inquiries_count": "number",
            "promoted_until": "string (ISO8601) | null",
            "expires_at": "string (ISO8601) | null",
            "published_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "USER#<seller_id>",
            "GSI1SK": "LISTING#<status>#<published_at>",
            "GSI2PK": "CAT#<category_id>",
            "GSI2SK": "<published_at>",
            "GSI3PK": "STATUS#<status>",
            "GSI3SK": "<published_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova listing per ID",
                "query": "PK = LISTING#<listing_id>"
            },
            {
                "pattern": "Lista annunci utente per status",
                "query": "GSI1: GSI1PK = USER#<seller_id>, SK begins_with LISTING#<status>#"
            },
            {
                "pattern": "Lista annunci per categoria",
                "query": "GSI2: GSI2PK = CAT#<category_id>, ScanIndexForward=false"
            },
            {
                "pattern": "Tutti gli annunci attivi",
                "query": "GSI3: GSI3PK = STATUS#active, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: CATEGORY
    # ───────────────────────────────────────────────────────────────────────────
    "CATEGORY": {
        "item_structure": {
            "PK": "CATEGORY#<category_id>",
            "SK": "METADATA",
            "entity_type": "CATEGORY",
            
            "category_id": "string (UUID)",
            "parent_id": "string (UUID) | null",
            "name": "string",
            "slug": "string",
            "description": "string | null",
            "icon": "string | null",
            "image_url": "string | null",
            "listing_count": "number",
            "attribute_schema": "map (schema attributi per questa categoria)",
            "display_order": "number",
            "is_active": "boolean",
            "created_at": "string (ISO8601)",
            
            # GSI per albero categorie
            "GSI1PK": "PARENT#<parent_id|ROOT>",
            "GSI1SK": "CAT#<display_order>#<category_id>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova categoria per ID",
                "query": "PK = CATEGORY#<category_id>"
            },
            {
                "pattern": "Lista categorie root",
                "query": "GSI1: GSI1PK = PARENT#ROOT, ScanIndexForward=true"
            },
            {
                "pattern": "Lista sottocategorie",
                "query": "GSI1: GSI1PK = PARENT#<parent_id>, ScanIndexForward=true"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: SELLER_PROFILE
    # ───────────────────────────────────────────────────────────────────────────
    "SELLER_PROFILE": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "SELLER_PROFILE",
            "entity_type": "SELLER_PROFILE",
            
            "user_id": "string (UUID)",
            "display_name": "string | null",
            "bio": "string | null",
            "avatar_url": "string | null",
            "rating_average": "number (0-5, 2 decimali)",
            "rating_count": "number",
            "response_rate": "number (0-100)",
            "response_time_hours": "number",
            "active_listings_count": "number",
            "total_sales": "number",
            "is_verified": "boolean",
            "verification_badges": "list [string]",
            "member_since": "string (ISO8601)",
            "last_active_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Ottieni profilo venditore",
                "query": "PK = USER#<user_id>, SK = SELLER_PROFILE"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: OFFER
    # ───────────────────────────────────────────────────────────────────────────
    "OFFER": {
        "item_structure": {
            "PK": "LISTING#<listing_id>",
            "SK": "OFFER#<created_at>#<offer_id>",
            "entity_type": "OFFER",
            
            "offer_id": "string (UUID)",
            "listing_id": "string (UUID)",
            "buyer_id": "string (UUID)",
            "seller_id": "string (UUID)",
            "amount": "number (centesimi)",
            "currency": "string",
            "message": "string | null",
            "status": "string (pending|accepted|rejected|countered|withdrawn|expired)",
            "counter_amount": "number | null",
            "counter_message": "string | null",
            "expires_at": "string (ISO8601)",
            "responded_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "BUYER#<buyer_id>",
            "GSI1SK": "OFFER#<status>#<created_at>",
            "GSI2PK": "SELLER#<seller_id>",
            "GSI2SK": "OFFER#<status>#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista offerte per listing",
                "query": "PK = LISTING#<listing_id>, SK begins_with OFFER#"
            },
            {
                "pattern": "Lista offerte fatte (come buyer)",
                "query": "GSI1: GSI1PK = BUYER#<buyer_id>, SK begins_with OFFER#"
            },
            {
                "pattern": "Lista offerte ricevute (come seller)",
                "query": "GSI2: GSI2PK = SELLER#<seller_id>, SK begins_with OFFER#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: TRANSACTION
    # ───────────────────────────────────────────────────────────────────────────
    "TRANSACTION": {
        "item_structure": {
            "PK": "TRANSACTION#<transaction_id>",
            "SK": "METADATA",
            "entity_type": "TRANSACTION",
            
            "transaction_id": "string (UUID)",
            "listing_id": "string (UUID)",
            "offer_id": "string (UUID) | null",
            "buyer_id": "string (UUID)",
            "seller_id": "string (UUID)",
            "amount": "number (centesimi)",
            "platform_fee": "number",
            "seller_payout": "number",
            "currency": "string",
            "status": "string (pending_payment|paid|shipped|delivered|completed|disputed|refunded)",
            "payment_method": "string",
            "payment_id": "string | null",
            "shipping_info": "map {method, address, tracking_number, carrier}",
            "buyer_confirmed_at": "string (ISO8601) | null",
            "seller_shipped_at": "string (ISO8601) | null",
            "delivered_at": "string (ISO8601) | null",
            "completed_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "BUYER#<buyer_id>",
            "GSI1SK": "TXN#<created_at>",
            "GSI2PK": "SELLER#<seller_id>",
            "GSI2SK": "TXN#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova transazione per ID",
                "query": "PK = TRANSACTION#<transaction_id>"
            },
            {
                "pattern": "Lista acquisti utente",
                "query": "GSI1: GSI1PK = BUYER#<buyer_id>, ScanIndexForward=false"
            },
            {
                "pattern": "Lista vendite utente",
                "query": "GSI2: GSI2PK = SELLER#<seller_id>, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: REVIEW (Marketplace)
    # ───────────────────────────────────────────────────────────────────────────
    "REVIEW": {
        "item_structure": {
            "PK": "TRANSACTION#<transaction_id>",
            "SK": "REVIEW#<review_type>#<reviewer_id>",
            "entity_type": "REVIEW",
            
            "review_id": "string (UUID)",
            "transaction_id": "string (UUID)",
            "reviewer_id": "string (UUID)",
            "reviewee_id": "string (UUID)",
            "review_type": "string (buyer_to_seller|seller_to_buyer)",
            "rating": "number (1-5)",
            "title": "string | null",
            "content": "string | null",
            "aspects": "map {communication, shipping, item_as_described, ...}",
            "is_public": "boolean",
            "response": "string | null",
            "response_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "REVIEWEE#<reviewee_id>",
            "GSI1SK": "REVIEW#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova recensioni per transazione",
                "query": "PK = TRANSACTION#<transaction_id>, SK begins_with REVIEW#"
            },
            {
                "pattern": "Lista recensioni ricevute",
                "query": "GSI1: GSI1PK = REVIEWEE#<reviewee_id>, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: FAVORITE
    # ───────────────────────────────────────────────────────────────────────────
    "FAVORITE": {
        "item_structure": {
            "PK": "USER#<user_id>",
            "SK": "FAV#<listing_id>",
            "entity_type": "FAVORITE",
            
            "user_id": "string (UUID)",
            "listing_id": "string (UUID)",
            "created_at": "string (ISO8601)",
            
            # GSI per sapere chi ha messo nei preferiti un listing
            "GSI1PK": "LISTING#<listing_id>",
            "GSI1SK": "FAV#<user_id>"
        },
        
        "access_patterns": [
            {
                "pattern": "Verifica se listing è nei preferiti",
                "query": "PK = USER#<user_id>, SK = FAV#<listing_id>"
            },
            {
                "pattern": "Lista preferiti utente",
                "query": "PK = USER#<user_id>, SK begins_with FAV#"
            },
            {
                "pattern": "Lista utenti che hanno salvato listing",
                "query": "GSI1: GSI1PK = LISTING#<listing_id>, SK begins_with FAV#"
            }
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# MARKETPLACE - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

MARKETPLACE_AURORA = {
    
    "tables": {
        
        "categories": """
            CREATE TABLE categories (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
                name VARCHAR(100) NOT NULL,
                slug VARCHAR(100) NOT NULL,
                description TEXT,
                icon VARCHAR(100),
                image_url VARCHAR(500),
                listing_count INTEGER NOT NULL DEFAULT 0,
                attribute_schema JSONB DEFAULT '{}',
                display_order INTEGER NOT NULL DEFAULT 0,
                is_active BOOLEAN NOT NULL DEFAULT TRUE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT categories_slug_unique UNIQUE (slug)
            );
            
            CREATE INDEX idx_categories_parent ON categories(parent_id);
            CREATE INDEX idx_categories_active ON categories(is_active, display_order);
        """,
        
        "listings": """
            CREATE TABLE listings (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                seller_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                listing_type VARCHAR(20) NOT NULL
                    CHECK (listing_type IN ('product', 'service', 'job', 'housing', 'vehicle')),
                title VARCHAR(200) NOT NULL,
                description TEXT NOT NULL,
                category_id UUID NOT NULL REFERENCES categories(id),
                subcategory_id UUID REFERENCES categories(id),
                price JSONB NOT NULL,
                condition VARCHAR(20)
                    CHECK (condition IN ('new', 'like_new', 'good', 'fair', 'for_parts')),
                media JSONB DEFAULT '[]',
                location JSONB,
                attributes JSONB DEFAULT '{}',
                shipping_options JSONB DEFAULT '[]',
                status VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'pending_review', 'active', 'reserved', 'sold', 'expired', 'deleted')),
                visibility VARCHAR(20) NOT NULL DEFAULT 'public'
                    CHECK (visibility IN ('public', 'unlisted', 'private')),
                views_count INTEGER NOT NULL DEFAULT 0,
                favorites_count INTEGER NOT NULL DEFAULT 0,
                inquiries_count INTEGER NOT NULL DEFAULT 0,
                promoted_until TIMESTAMPTZ,
                expires_at TIMESTAMPTZ,
                published_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_listings_seller ON listings(seller_id, status);
            CREATE INDEX idx_listings_category ON listings(category_id, published_at DESC) WHERE status = 'active';
            CREATE INDEX idx_listings_status ON listings(status, published_at DESC);
            CREATE INDEX idx_listings_title_trgm ON listings USING gin(title gin_trgm_ops);
            CREATE INDEX idx_listings_location ON listings USING gin(location);
            CREATE INDEX idx_listings_expires ON listings(expires_at) WHERE status = 'active';
            
            CREATE TRIGGER listings_updated_at
                BEFORE UPDATE ON listings
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "seller_profiles": """
            CREATE TABLE seller_profiles (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                display_name VARCHAR(100),
                bio TEXT,
                avatar_url VARCHAR(500),
                rating_average DECIMAL(3,2) NOT NULL DEFAULT 0,
                rating_count INTEGER NOT NULL DEFAULT 0,
                response_rate INTEGER NOT NULL DEFAULT 0,
                response_time_hours INTEGER,
                active_listings_count INTEGER NOT NULL DEFAULT 0,
                total_sales INTEGER NOT NULL DEFAULT 0,
                is_verified BOOLEAN NOT NULL DEFAULT FALSE,
                verification_badges TEXT[] DEFAULT '{}',
                member_since TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT seller_profiles_user_unique UNIQUE (user_id)
            );
            
            CREATE TRIGGER seller_profiles_updated_at
                BEFORE UPDATE ON seller_profiles
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "offers": """
            CREATE TABLE offers (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                listing_id UUID NOT NULL REFERENCES listings(id) ON DELETE CASCADE,
                buyer_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                seller_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                amount INTEGER NOT NULL,
                currency VARCHAR(3) NOT NULL DEFAULT 'EUR',
                message TEXT,
                status VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'accepted', 'rejected', 'countered', 'withdrawn', 'expired')),
                counter_amount INTEGER,
                counter_message TEXT,
                expires_at TIMESTAMPTZ NOT NULL,
                responded_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT offers_positive_amount CHECK (amount > 0)
            );
            
            CREATE INDEX idx_offers_listing ON offers(listing_id, status);
            CREATE INDEX idx_offers_buyer ON offers(buyer_id, status);
            CREATE INDEX idx_offers_seller ON offers(seller_id, status);
            CREATE INDEX idx_offers_expires ON offers(expires_at) WHERE status = 'pending';
        """,
        
        "transactions": """
            CREATE TABLE transactions (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                listing_id UUID NOT NULL REFERENCES listings(id),
                offer_id UUID REFERENCES offers(id),
                buyer_id UUID NOT NULL REFERENCES users(id),
                seller_id UUID NOT NULL REFERENCES users(id),
                amount INTEGER NOT NULL,
                platform_fee INTEGER NOT NULL DEFAULT 0,
                seller_payout INTEGER NOT NULL,
                currency VARCHAR(3) NOT NULL DEFAULT 'EUR',
                status VARCHAR(30) NOT NULL DEFAULT 'pending_payment'
                    CHECK (status IN ('pending_payment', 'paid', 'shipped', 'delivered', 'completed', 'disputed', 'refunded')),
                payment_method VARCHAR(50),
                payment_id VARCHAR(255),
                shipping_info JSONB,
                buyer_confirmed_at TIMESTAMPTZ,
                seller_shipped_at TIMESTAMPTZ,
                delivered_at TIMESTAMPTZ,
                completed_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_transactions_buyer ON transactions(buyer_id, created_at DESC);
            CREATE INDEX idx_transactions_seller ON transactions(seller_id, created_at DESC);
            CREATE INDEX idx_transactions_status ON transactions(status);
            
            CREATE TRIGGER transactions_updated_at
                BEFORE UPDATE ON transactions
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "marketplace_reviews": """
            CREATE TABLE marketplace_reviews (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                transaction_id UUID NOT NULL REFERENCES transactions(id) ON DELETE CASCADE,
                reviewer_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                reviewee_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                review_type VARCHAR(20) NOT NULL
                    CHECK (review_type IN ('buyer_to_seller', 'seller_to_buyer')),
                rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
                title VARCHAR(100),
                content TEXT,
                aspects JSONB DEFAULT '{}',
                is_public BOOLEAN NOT NULL DEFAULT TRUE,
                response TEXT,
                response_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT marketplace_reviews_unique UNIQUE (transaction_id, reviewer_id)
            );
            
            CREATE INDEX idx_marketplace_reviews_reviewee ON marketplace_reviews(reviewee_id, created_at DESC);
            
            CREATE TRIGGER marketplace_reviews_updated_at
                BEFORE UPDATE ON marketplace_reviews
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "favorites": """
            CREATE TABLE favorites (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                listing_id UUID NOT NULL REFERENCES listings(id) ON DELETE CASCADE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT favorites_unique UNIQUE (user_id, listing_id)
            );
            
            CREATE INDEX idx_favorites_user ON favorites(user_id);
            CREATE INDEX idx_favorites_listing ON favorites(listing_id);
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO BOOKING - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

BOOKING_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: BOOKABLE_RESOURCE
    # ───────────────────────────────────────────────────────────────────────────
    "BOOKABLE_RESOURCE": {
        "item_structure": {
            "PK": "RESOURCE#<resource_id>",
            "SK": "METADATA",
            "entity_type": "BOOKABLE_RESOURCE",
            
            "resource_id": "string (UUID)",
            "owner_id": "string (UUID)",
            "resource_type": "string (service|room|equipment|person|vehicle|table)",
            "name": "string",
            "description": "string",
            "category": "string",
            "media": "list [{url, thumbnail_url, order}]",
            "location": "map {address, city, lat, lng}",
            "pricing": "map {base_price, currency, price_type, min_duration, max_duration, deposit}",
            "booking_settings": "map {min_advance_hours, max_advance_days, buffer_minutes, cancellation_policy, requires_approval, max_bookings_per_user, allow_recurring}",
            "capacity": "number",
            "status": "string (draft|active|paused|deleted)",
            "timezone": "string (Europe/Rome)",
            "rating_average": "number",
            "rating_count": "number",
            "bookings_count": "number",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "OWNER#<owner_id>",
            "GSI1SK": "RESOURCE#<status>#<created_at>",
            "GSI2PK": "CAT#<category>",
            "GSI2SK": "RESOURCE#<rating_average>"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova risorsa per ID",
                "query": "PK = RESOURCE#<resource_id>"
            },
            {
                "pattern": "Lista risorse owner",
                "query": "GSI1: GSI1PK = OWNER#<owner_id>, SK begins_with RESOURCE#"
            },
            {
                "pattern": "Lista risorse per categoria (ordinate per rating)",
                "query": "GSI2: GSI2PK = CAT#<category>, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: AVAILABILITY_RULE
    # ───────────────────────────────────────────────────────────────────────────
    "AVAILABILITY_RULE": {
        "item_structure": {
            "PK": "RESOURCE#<resource_id>",
            "SK": "AVAIL_RULE#<day_of_week>#<start_time>",
            "entity_type": "AVAILABILITY_RULE",
            
            "rule_id": "string (UUID)",
            "resource_id": "string (UUID)",
            "day_of_week": "number (0-6, 0=domenica)",
            "start_time": "string (HH:MM)",
            "end_time": "string (HH:MM)",
            "is_available": "boolean",
            "valid_from": "string (YYYY-MM-DD) | null",
            "valid_until": "string (YYYY-MM-DD) | null",
            "created_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista regole disponibilità risorsa",
                "query": "PK = RESOURCE#<resource_id>, SK begins_with AVAIL_RULE#"
            },
            {
                "pattern": "Regole per giorno specifico",
                "query": "PK = RESOURCE#<resource_id>, SK begins_with AVAIL_RULE#<day_of_week>#"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: AVAILABILITY_EXCEPTION
    # ───────────────────────────────────────────────────────────────────────────
    "AVAILABILITY_EXCEPTION": {
        "item_structure": {
            "PK": "RESOURCE#<resource_id>",
            "SK": "AVAIL_EXC#<date>",
            "entity_type": "AVAILABILITY_EXCEPTION",
            
            "exception_id": "string (UUID)",
            "resource_id": "string (UUID)",
            "date": "string (YYYY-MM-DD)",
            "exception_type": "string (closed|modified_hours|extra_availability)",
            "start_time": "string (HH:MM) | null",
            "end_time": "string (HH:MM) | null",
            "reason": "string | null",
            "created_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista eccezioni risorsa",
                "query": "PK = RESOURCE#<resource_id>, SK begins_with AVAIL_EXC#"
            },
            {
                "pattern": "Eccezioni per range date",
                "query": "PK = RESOURCE#<resource_id>, SK BETWEEN AVAIL_EXC#<start_date> AND AVAIL_EXC#<end_date>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: BOOKING
    # ───────────────────────────────────────────────────────────────────────────
    "BOOKING": {
        "item_structure": {
            "PK": "BOOKING#<booking_id>",
            "SK": "METADATA",
            "entity_type": "BOOKING",
            
            "booking_id": "string (UUID)",
            "booking_code": "string (es: BK-2024-ABC123)",
            "resource_id": "string (UUID)",
            "booker_id": "string (UUID)",
            "provider_id": "string (UUID)",
            "start_datetime": "string (ISO8601)",
            "end_datetime": "string (ISO8601)",
            "duration_minutes": "number",
            "party_size": "number",
            "status": "string (pending|confirmed|cancelled|completed|no_show)",
            "price": "map {subtotal, deposit, total, currency}",
            "payment_status": "string (pending|deposit_paid|paid|refunded)",
            "payment_id": "string | null",
            "notes": "string | null",
            "provider_notes": "string | null",
            "recurring_id": "string (UUID) | null",
            "reminder_sent": "boolean",
            "cancelled_at": "string (ISO8601) | null",
            "cancellation_reason": "string | null",
            "cancelled_by": "string (UUID) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "BOOKER#<booker_id>",
            "GSI1SK": "BOOKING#<start_datetime>",
            "GSI2PK": "PROVIDER#<provider_id>",
            "GSI2SK": "BOOKING#<start_datetime>",
            "GSI3PK": "RESOURCE#<resource_id>",
            "GSI3SK": "BOOKING#<start_datetime>"
        },
        
        "resource_booking_slot": {
            "descrizione": "Item per bloccare slot (prevenire doppia prenotazione)",
            "PK": "RESOURCE#<resource_id>",
            "SK": "SLOT#<date>#<start_time>",
            "entity_type": "BOOKING_SLOT",
            
            "booking_id": "string (UUID)",
            "end_time": "string (HH:MM)",
            "status": "string (reserved|confirmed)"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova prenotazione per ID",
                "query": "PK = BOOKING#<booking_id>"
            },
            {
                "pattern": "Trova per codice (booking_code)",
                "nota": "Necessita GSI dedicato o tabella lookup"
            },
            {
                "pattern": "Prenotazioni utente (come booker)",
                "query": "GSI1: GSI1PK = BOOKER#<booker_id>, ScanIndexForward=false"
            },
            {
                "pattern": "Prenotazioni provider",
                "query": "GSI2: GSI2PK = PROVIDER#<provider_id>, ScanIndexForward=false"
            },
            {
                "pattern": "Prenotazioni risorsa (per controllare disponibilità)",
                "query": "GSI3: GSI3PK = RESOURCE#<resource_id>, SK BETWEEN BOOKING#<start> AND BOOKING#<end>"
            },
            {
                "pattern": "Slot occupati risorsa per data",
                "query": "PK = RESOURCE#<resource_id>, SK begins_with SLOT#<date>#"
            }
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# BOOKING - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

BOOKING_AURORA = {
    
    "tables": {
        
        "bookable_resources": """
            CREATE TABLE bookable_resources (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                owner_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                resource_type VARCHAR(30) NOT NULL
                    CHECK (resource_type IN ('service', 'room', 'equipment', 'person', 'vehicle', 'table')),
                name VARCHAR(200) NOT NULL,
                description TEXT,
                category VARCHAR(100),
                media JSONB DEFAULT '[]',
                location JSONB,
                pricing JSONB NOT NULL DEFAULT '{
                    "base_price": 0,
                    "currency": "EUR",
                    "price_type": "per_hour"
                }',
                booking_settings JSONB NOT NULL DEFAULT '{
                    "min_advance_hours": 1,
                    "max_advance_days": 30,
                    "buffer_minutes": 0,
                    "cancellation_policy": "flexible",
                    "requires_approval": false,
                    "max_bookings_per_user": null,
                    "allow_recurring": false
                }',
                capacity INTEGER NOT NULL DEFAULT 1,
                status VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'active', 'paused', 'deleted')),
                timezone VARCHAR(50) NOT NULL DEFAULT 'Europe/Rome',
                rating_average DECIMAL(3,2) NOT NULL DEFAULT 0,
                rating_count INTEGER NOT NULL DEFAULT 0,
                bookings_count INTEGER NOT NULL DEFAULT 0,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_bookable_resources_owner ON bookable_resources(owner_id, status);
            CREATE INDEX idx_bookable_resources_category ON bookable_resources(category, rating_average DESC) 
                WHERE status = 'active';
            CREATE INDEX idx_bookable_resources_location ON bookable_resources USING gin(location);
            
            CREATE TRIGGER bookable_resources_updated_at
                BEFORE UPDATE ON bookable_resources
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "availability_rules": """
            CREATE TABLE availability_rules (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                resource_id UUID NOT NULL REFERENCES bookable_resources(id) ON DELETE CASCADE,
                day_of_week INTEGER NOT NULL CHECK (day_of_week BETWEEN 0 AND 6),
                start_time TIME NOT NULL,
                end_time TIME NOT NULL,
                is_available BOOLEAN NOT NULL DEFAULT TRUE,
                valid_from DATE,
                valid_until DATE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT availability_rules_time_check CHECK (end_time > start_time),
                CONSTRAINT availability_rules_valid_range CHECK (valid_until IS NULL OR valid_until >= valid_from)
            );
            
            CREATE INDEX idx_availability_rules_resource ON availability_rules(resource_id, day_of_week);
        """,
        
        "availability_exceptions": """
            CREATE TABLE availability_exceptions (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                resource_id UUID NOT NULL REFERENCES bookable_resources(id) ON DELETE CASCADE,
                date DATE NOT NULL,
                exception_type VARCHAR(30) NOT NULL
                    CHECK (exception_type IN ('closed', 'modified_hours', 'extra_availability')),
                start_time TIME,
                end_time TIME,
                reason VARCHAR(255),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT availability_exceptions_unique UNIQUE (resource_id, date, start_time)
            );
            
            CREATE INDEX idx_availability_exceptions_resource_date ON availability_exceptions(resource_id, date);
        """,
        
        "bookings": """
            CREATE TABLE bookings (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                booking_code VARCHAR(20) NOT NULL,
                resource_id UUID NOT NULL REFERENCES bookable_resources(id),
                booker_id UUID NOT NULL REFERENCES users(id),
                provider_id UUID NOT NULL REFERENCES users(id),
                start_datetime TIMESTAMPTZ NOT NULL,
                end_datetime TIMESTAMPTZ NOT NULL,
                duration_minutes INTEGER NOT NULL,
                party_size INTEGER NOT NULL DEFAULT 1,
                status VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'confirmed', 'cancelled', 'completed', 'no_show')),
                price JSONB NOT NULL,
                payment_status VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (payment_status IN ('pending', 'deposit_paid', 'paid', 'refunded')),
                payment_id VARCHAR(255),
                notes TEXT,
                provider_notes TEXT,
                recurring_id UUID,
                reminder_sent BOOLEAN NOT NULL DEFAULT FALSE,
                cancelled_at TIMESTAMPTZ,
                cancellation_reason TEXT,
                cancelled_by UUID REFERENCES users(id),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT bookings_code_unique UNIQUE (booking_code),
                CONSTRAINT bookings_time_check CHECK (end_datetime > start_datetime)
            );
            
            CREATE INDEX idx_bookings_booker ON bookings(booker_id, start_datetime DESC);
            CREATE INDEX idx_bookings_provider ON bookings(provider_id, start_datetime DESC);
            CREATE INDEX idx_bookings_resource_time ON bookings(resource_id, start_datetime, end_datetime)
                WHERE status IN ('pending', 'confirmed');
            CREATE INDEX idx_bookings_status ON bookings(status, start_datetime);
            
            -- Funzione per generare booking_code
            CREATE OR REPLACE FUNCTION generate_booking_code()
            RETURNS TRIGGER AS $$
            BEGIN
                NEW.booking_code := 'BK-' || TO_CHAR(NOW(), 'YYYY') || '-' || 
                    UPPER(SUBSTRING(MD5(RANDOM()::TEXT) FROM 1 FOR 6));
                RETURN NEW;
            END;
            $$ LANGUAGE plpgsql;
            
            CREATE TRIGGER bookings_generate_code
                BEFORE INSERT ON bookings
                FOR EACH ROW
                WHEN (NEW.booking_code IS NULL)
                EXECUTE FUNCTION generate_booking_code();
            
            CREATE TRIGGER bookings_updated_at
                BEFORE UPDATE ON bookings
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
            
            -- Constraint per evitare doppia prenotazione
            CREATE EXTENSION IF NOT EXISTS btree_gist;
            
            ALTER TABLE bookings ADD CONSTRAINT bookings_no_overlap
                EXCLUDE USING gist (
                    resource_id WITH =,
                    tstzrange(start_datetime, end_datetime) WITH &&
                ) WHERE (status IN ('pending', 'confirmed'));
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO COMMERCE - DATA MODEL
# ═══════════════════════════════════════════════════════════════════════════════

COMMERCE_DYNAMODB = {
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: PRODUCT
    # ───────────────────────────────────────────────────────────────────────────
    "PRODUCT": {
        "item_structure": {
            "PK": "PRODUCT#<product_id>",
            "SK": "METADATA",
            "entity_type": "PRODUCT",
            
            "product_id": "string (UUID)",
            "seller_id": "string (UUID)",
            "sku": "string",
            "name": "string",
            "slug": "string",
            "description": "string",
            "short_description": "string | null",
            "category_id": "string (UUID)",
            "brand": "string | null",
            "media": "list [{id, type, url, thumbnail_url, alt_text, order}]",
            "price": "map {amount, currency, compare_at_amount, cost_per_item}",
            "inventory": "map {track_quantity, quantity, allow_backorder, low_stock_threshold}",
            "shipping": "map {weight_grams, dimensions, requires_shipping, is_fragile}",
            "attributes": "map (attributi categoria-specifici)",
            "tags": "list [string]",
            "status": "string (draft|active|archived|deleted)",
            "visibility": "string (visible|hidden|catalog_only)",
            "is_digital": "boolean",
            "digital_file_url": "string | null",
            "seo": "map {meta_title, meta_description, keywords}",
            "rating_average": "number",
            "rating_count": "number",
            "sales_count": "number",
            "views_count": "number",
            "has_variants": "boolean",
            "published_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "SELLER#<seller_id>",
            "GSI1SK": "PRODUCT#<status>#<created_at>",
            "GSI2PK": "CAT#<category_id>",
            "GSI2SK": "PRODUCT#<sales_count>",
            "GSI3PK": "SKU#<sku>",
            "GSI3SK": "PRODUCT"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova prodotto per ID",
                "query": "PK = PRODUCT#<product_id>"
            },
            {
                "pattern": "Trova prodotto per SKU",
                "query": "GSI3: GSI3PK = SKU#<sku>"
            },
            {
                "pattern": "Lista prodotti seller",
                "query": "GSI1: GSI1PK = SELLER#<seller_id>, SK begins_with PRODUCT#"
            },
            {
                "pattern": "Lista prodotti categoria (per vendite)",
                "query": "GSI2: GSI2PK = CAT#<category_id>, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: PRODUCT_VARIANT
    # ───────────────────────────────────────────────────────────────────────────
    "PRODUCT_VARIANT": {
        "item_structure": {
            "PK": "PRODUCT#<product_id>",
            "SK": "VARIANT#<variant_id>",
            "entity_type": "PRODUCT_VARIANT",
            
            "variant_id": "string (UUID)",
            "product_id": "string (UUID)",
            "sku": "string | null",
            "name": "string",
            "options": "map {Colore: 'Rosso', Taglia: 'XL'}",
            "price": "map {amount, currency} | null (override)",
            "inventory": "map {quantity, allow_backorder}",
            "image_url": "string | null",
            "weight_grams": "number | null",
            "is_default": "boolean",
            "status": "string (active|inactive)",
            "created_at": "string (ISO8601)",
            
            # GSI per lookup per SKU variante
            "GSI3PK": "VSKU#<sku>",
            "GSI3SK": "VARIANT"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista varianti prodotto",
                "query": "PK = PRODUCT#<product_id>, SK begins_with VARIANT#"
            },
            {
                "pattern": "Trova variante per SKU",
                "query": "GSI3: GSI3PK = VSKU#<sku>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: CART
    # ───────────────────────────────────────────────────────────────────────────
    "CART": {
        "item_structure": {
            "PK": "CART#<cart_id>",
            "SK": "METADATA",
            "entity_type": "CART",
            
            "cart_id": "string (UUID)",
            "user_id": "string (UUID) | null",
            "session_id": "string | null",
            "status": "string (active|abandoned|converted)",
            "currency": "string",
            "subtotal": "number",
            "discount_amount": "number",
            "coupon_code": "string | null",
            "shipping_amount": "number",
            "tax_amount": "number",
            "total": "number",
            "items_count": "number",
            "notes": "string | null",
            "expires_at": "string (ISO8601)",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # TTL
            "expires_at_epoch": "number",
            
            # GSI Keys
            "GSI1PK": "USER#<user_id>",
            "GSI1SK": "CART#<status>#<updated_at>",
            "GSI2PK": "SESSION#<session_id>",
            "GSI2SK": "CART"
        },
        
        "cart_item": {
            "PK": "CART#<cart_id>",
            "SK": "ITEM#<product_id>#<variant_id|NOVARIANT>",
            "entity_type": "CART_ITEM",
            
            "item_id": "string (UUID)",
            "cart_id": "string (UUID)",
            "product_id": "string (UUID)",
            "variant_id": "string (UUID) | null",
            "quantity": "number",
            "unit_price": "number",
            "total_price": "number",
            "product_snapshot": "map {name, sku, image_url, variant_name}",
            "added_at": "string (ISO8601)"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova carrello per ID con items",
                "query": "PK = CART#<cart_id>"
            },
            {
                "pattern": "Trova carrello attivo utente",
                "query": "GSI1: GSI1PK = USER#<user_id>, SK begins_with CART#active"
            },
            {
                "pattern": "Trova carrello per sessione",
                "query": "GSI2: GSI2PK = SESSION#<session_id>"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: ORDER
    # ───────────────────────────────────────────────────────────────────────────
    "ORDER": {
        "item_structure": {
            "PK": "ORDER#<order_id>",
            "SK": "METADATA",
            "entity_type": "ORDER",
            
            "order_id": "string (UUID)",
            "order_number": "string (ORD-2024-000001)",
            "customer_id": "string (UUID)",
            "status": "string (pending|confirmed|processing|shipped|delivered|completed|cancelled|refunded)",
            "payment_status": "string (pending|paid|failed|refunded|partially_refunded)",
            "fulfillment_status": "string (unfulfilled|partial|fulfilled)",
            "currency": "string",
            "subtotal": "number",
            "discount_amount": "number",
            "coupon_code": "string | null",
            "shipping_amount": "number",
            "tax_amount": "number",
            "total": "number",
            "billing_address": "map",
            "shipping_address": "map",
            "shipping_method": "map {id, name, carrier, estimated_days}",
            "payment_method": "map {type, last4}",
            "payment_id": "string | null",
            "notes": "string | null",
            "internal_notes": "string | null",
            "tracking_number": "string | null",
            "tracking_url": "string | null",
            "paid_at": "string (ISO8601) | null",
            "shipped_at": "string (ISO8601) | null",
            "delivered_at": "string (ISO8601) | null",
            "cancelled_at": "string (ISO8601) | null",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "CUSTOMER#<customer_id>",
            "GSI1SK": "ORDER#<created_at>",
            "GSI2PK": "STATUS#<status>",
            "GSI2SK": "ORDER#<created_at>",
            "GSI3PK": "ORDNUM#<order_number>",
            "GSI3SK": "ORDER"
        },
        
        "order_item": {
            "PK": "ORDER#<order_id>",
            "SK": "ITEM#<order_item_id>",
            "entity_type": "ORDER_ITEM",
            
            "order_item_id": "string (UUID)",
            "order_id": "string (UUID)",
            "product_id": "string (UUID)",
            "variant_id": "string (UUID) | null",
            "seller_id": "string (UUID)",
            "quantity": "number",
            "unit_price": "number",
            "total_price": "number",
            "product_snapshot": "map (completo)",
            "fulfillment_status": "string (unfulfilled|fulfilled|returned)"
        },
        
        "access_patterns": [
            {
                "pattern": "Trova ordine per ID con items",
                "query": "PK = ORDER#<order_id>"
            },
            {
                "pattern": "Trova ordine per numero",
                "query": "GSI3: GSI3PK = ORDNUM#<order_number>"
            },
            {
                "pattern": "Lista ordini cliente",
                "query": "GSI1: GSI1PK = CUSTOMER#<customer_id>, ScanIndexForward=false"
            },
            {
                "pattern": "Lista ordini per status",
                "query": "GSI2: GSI2PK = STATUS#<status>, ScanIndexForward=false"
            }
        ]
    },
    
    # ───────────────────────────────────────────────────────────────────────────
    # ENTITÀ: PRODUCT_REVIEW
    # ───────────────────────────────────────────────────────────────────────────
    "PRODUCT_REVIEW": {
        "item_structure": {
            "PK": "PRODUCT#<product_id>",
            "SK": "REVIEW#<created_at>#<review_id>",
            "entity_type": "PRODUCT_REVIEW",
            
            "review_id": "string (UUID)",
            "product_id": "string (UUID)",
            "order_item_id": "string (UUID) | null",
            "reviewer_id": "string (UUID)",
            "rating": "number (1-5)",
            "title": "string | null",
            "content": "string | null",
            "media": "list [{url, thumbnail_url}]",
            "is_verified_purchase": "boolean",
            "helpful_count": "number",
            "seller_response": "string | null",
            "seller_response_at": "string (ISO8601) | null",
            "status": "string (pending|published|hidden|deleted)",
            "created_at": "string (ISO8601)",
            "updated_at": "string (ISO8601)",
            
            # GSI Keys
            "GSI1PK": "REVIEWER#<reviewer_id>",
            "GSI1SK": "REVIEW#<created_at>"
        },
        
        "access_patterns": [
            {
                "pattern": "Lista recensioni prodotto",
                "query": "PK = PRODUCT#<product_id>, SK begins_with REVIEW#, ScanIndexForward=false"
            },
            {
                "pattern": "Lista recensioni utente",
                "query": "GSI1: GSI1PK = REVIEWER#<reviewer_id>, ScanIndexForward=false"
            }
        ]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# COMMERCE - SCHEMA AURORA POSTGRESQL
# ═══════════════════════════════════════════════════════════════════════════════

COMMERCE_AURORA = {
    
    "tables": {
        
        "product_categories": """
            CREATE TABLE product_categories (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                parent_id UUID REFERENCES product_categories(id) ON DELETE SET NULL,
                name VARCHAR(100) NOT NULL,
                slug VARCHAR(100) NOT NULL,
                description TEXT,
                image_url VARCHAR(500),
                attribute_schema JSONB DEFAULT '{}',
                display_order INTEGER NOT NULL DEFAULT 0,
                is_active BOOLEAN NOT NULL DEFAULT TRUE,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT product_categories_slug_unique UNIQUE (slug)
            );
            
            CREATE INDEX idx_product_categories_parent ON product_categories(parent_id);
        """,
        
        "products": """
            CREATE TABLE products (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                seller_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                sku VARCHAR(100),
                name VARCHAR(300) NOT NULL,
                slug VARCHAR(300) NOT NULL,
                description TEXT,
                short_description VARCHAR(500),
                category_id UUID REFERENCES product_categories(id),
                brand VARCHAR(100),
                media JSONB DEFAULT '[]',
                price JSONB NOT NULL DEFAULT '{"amount": 0, "currency": "EUR"}',
                inventory JSONB NOT NULL DEFAULT '{
                    "track_quantity": true,
                    "quantity": 0,
                    "allow_backorder": false,
                    "low_stock_threshold": 5
                }',
                shipping JSONB DEFAULT '{}',
                attributes JSONB DEFAULT '{}',
                tags TEXT[] DEFAULT '{}',
                status VARCHAR(20) NOT NULL DEFAULT 'draft'
                    CHECK (status IN ('draft', 'active', 'archived', 'deleted')),
                visibility VARCHAR(20) NOT NULL DEFAULT 'visible'
                    CHECK (visibility IN ('visible', 'hidden', 'catalog_only')),
                is_digital BOOLEAN NOT NULL DEFAULT FALSE,
                digital_file_url VARCHAR(500),
                seo JSONB DEFAULT '{}',
                rating_average DECIMAL(3,2) NOT NULL DEFAULT 0,
                rating_count INTEGER NOT NULL DEFAULT 0,
                sales_count INTEGER NOT NULL DEFAULT 0,
                views_count INTEGER NOT NULL DEFAULT 0,
                has_variants BOOLEAN NOT NULL DEFAULT FALSE,
                published_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT products_sku_unique UNIQUE (sku),
                CONSTRAINT products_slug_unique UNIQUE (slug)
            );
            
            CREATE INDEX idx_products_seller ON products(seller_id, status);
            CREATE INDEX idx_products_category ON products(category_id, sales_count DESC) WHERE status = 'active';
            CREATE INDEX idx_products_name_trgm ON products USING gin(name gin_trgm_ops);
            CREATE INDEX idx_products_tags ON products USING gin(tags);
            
            CREATE TRIGGER products_updated_at
                BEFORE UPDATE ON products
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "product_variants": """
            CREATE TABLE product_variants (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
                sku VARCHAR(100),
                name VARCHAR(200) NOT NULL,
                options JSONB NOT NULL DEFAULT '{}',
                price JSONB,
                inventory JSONB NOT NULL DEFAULT '{"quantity": 0, "allow_backorder": false}',
                image_url VARCHAR(500),
                weight_grams INTEGER,
                is_default BOOLEAN NOT NULL DEFAULT FALSE,
                status VARCHAR(20) NOT NULL DEFAULT 'active'
                    CHECK (status IN ('active', 'inactive')),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT product_variants_sku_unique UNIQUE (sku),
                CONSTRAINT product_variants_options_unique UNIQUE (product_id, options)
            );
            
            CREATE INDEX idx_product_variants_product ON product_variants(product_id);
        """,
        
        "carts": """
            CREATE TABLE carts (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                user_id UUID REFERENCES users(id) ON DELETE SET NULL,
                session_id VARCHAR(255),
                status VARCHAR(20) NOT NULL DEFAULT 'active'
                    CHECK (status IN ('active', 'abandoned', 'converted')),
                currency VARCHAR(3) NOT NULL DEFAULT 'EUR',
                subtotal INTEGER NOT NULL DEFAULT 0,
                discount_amount INTEGER NOT NULL DEFAULT 0,
                coupon_code VARCHAR(50),
                shipping_amount INTEGER NOT NULL DEFAULT 0,
                tax_amount INTEGER NOT NULL DEFAULT 0,
                total INTEGER NOT NULL DEFAULT 0,
                items_count INTEGER NOT NULL DEFAULT 0,
                notes TEXT,
                expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '30 days',
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
            );
            
            CREATE INDEX idx_carts_user ON carts(user_id, status) WHERE user_id IS NOT NULL;
            CREATE INDEX idx_carts_session ON carts(session_id, status) WHERE session_id IS NOT NULL;
            CREATE INDEX idx_carts_expires ON carts(expires_at) WHERE status = 'active';
            
            CREATE TRIGGER carts_updated_at
                BEFORE UPDATE ON carts
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "cart_items": """
            CREATE TABLE cart_items (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                cart_id UUID NOT NULL REFERENCES carts(id) ON DELETE CASCADE,
                product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
                variant_id UUID REFERENCES product_variants(id) ON DELETE CASCADE,
                quantity INTEGER NOT NULL DEFAULT 1,
                unit_price INTEGER NOT NULL,
                total_price INTEGER NOT NULL,
                product_snapshot JSONB NOT NULL,
                added_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT cart_items_unique UNIQUE (cart_id, product_id, variant_id),
                CONSTRAINT cart_items_quantity_positive CHECK (quantity > 0)
            );
            
            CREATE INDEX idx_cart_items_cart ON cart_items(cart_id);
        """,
        
        "orders": """
            CREATE TABLE orders (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                order_number VARCHAR(20) NOT NULL,
                customer_id UUID NOT NULL REFERENCES users(id),
                status VARCHAR(20) NOT NULL DEFAULT 'pending'
                    CHECK (status IN ('pending', 'confirmed', 'processing', 'shipped', 'delivered', 'completed', 'cancelled', 'refunded')),
                payment_status VARCHAR(30) NOT NULL DEFAULT 'pending'
                    CHECK (payment_status IN ('pending', 'paid', 'failed', 'refunded', 'partially_refunded')),
                fulfillment_status VARCHAR(20) NOT NULL DEFAULT 'unfulfilled'
                    CHECK (fulfillment_status IN ('unfulfilled', 'partial', 'fulfilled')),
                currency VARCHAR(3) NOT NULL DEFAULT 'EUR',
                subtotal INTEGER NOT NULL,
                discount_amount INTEGER NOT NULL DEFAULT 0,
                coupon_code VARCHAR(50),
                shipping_amount INTEGER NOT NULL DEFAULT 0,
                tax_amount INTEGER NOT NULL DEFAULT 0,
                total INTEGER NOT NULL,
                billing_address JSONB NOT NULL,
                shipping_address JSONB NOT NULL,
                shipping_method JSONB,
                payment_method JSONB,
                payment_id VARCHAR(255),
                notes TEXT,
                internal_notes TEXT,
                tracking_number VARCHAR(100),
                tracking_url VARCHAR(500),
                paid_at TIMESTAMPTZ,
                shipped_at TIMESTAMPTZ,
                delivered_at TIMESTAMPTZ,
                cancelled_at TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT orders_number_unique UNIQUE (order_number)
            );
            
            -- Sequenza per order_number
            CREATE SEQUENCE order_number_seq;
            
            CREATE OR REPLACE FUNCTION generate_order_number()
            RETURNS TRIGGER AS $$
            BEGIN
                NEW.order_number := 'ORD-' || TO_CHAR(NOW(), 'YYYY') || '-' || 
                    LPAD(nextval('order_number_seq')::TEXT, 6, '0');
                RETURN NEW;
            END;
            $$ LANGUAGE plpgsql;
            
            CREATE TRIGGER orders_generate_number
                BEFORE INSERT ON orders
                FOR EACH ROW
                WHEN (NEW.order_number IS NULL)
                EXECUTE FUNCTION generate_order_number();
            
            CREATE INDEX idx_orders_customer ON orders(customer_id, created_at DESC);
            CREATE INDEX idx_orders_status ON orders(status, created_at DESC);
            CREATE INDEX idx_orders_payment_status ON orders(payment_status) WHERE payment_status = 'pending';
            
            CREATE TRIGGER orders_updated_at
                BEFORE UPDATE ON orders
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "order_items": """
            CREATE TABLE order_items (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
                product_id UUID NOT NULL REFERENCES products(id),
                variant_id UUID REFERENCES product_variants(id),
                seller_id UUID NOT NULL REFERENCES users(id),
                quantity INTEGER NOT NULL,
                unit_price INTEGER NOT NULL,
                total_price INTEGER NOT NULL,
                product_snapshot JSONB NOT NULL,
                fulfillment_status VARCHAR(20) NOT NULL DEFAULT 'unfulfilled'
                    CHECK (fulfillment_status IN ('unfulfilled', 'fulfilled', 'returned'))
            );
            
            CREATE INDEX idx_order_items_order ON order_items(order_id);
            CREATE INDEX idx_order_items_seller ON order_items(seller_id);
        """,
        
        "product_reviews": """
            CREATE TABLE product_reviews (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
                order_item_id UUID REFERENCES order_items(id),
                reviewer_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
                rating INTEGER NOT NULL CHECK (rating BETWEEN 1 AND 5),
                title VARCHAR(150),
                content TEXT,
                media JSONB DEFAULT '[]',
                is_verified_purchase BOOLEAN NOT NULL DEFAULT FALSE,
                helpful_count INTEGER NOT NULL DEFAULT 0,
                seller_response TEXT,
                seller_response_at TIMESTAMPTZ,
                status VARCHAR(20) NOT NULL DEFAULT 'published'
                    CHECK (status IN ('pending', 'published', 'hidden', 'deleted')),
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT product_reviews_unique UNIQUE (product_id, reviewer_id)
            );
            
            CREATE INDEX idx_product_reviews_product ON product_reviews(product_id, created_at DESC) 
                WHERE status = 'published';
            CREATE INDEX idx_product_reviews_reviewer ON product_reviews(reviewer_id);
            
            CREATE TRIGGER product_reviews_updated_at
                BEFORE UPDATE ON product_reviews
                FOR EACH ROW EXECUTE FUNCTION update_updated_at();
        """,
        
        "coupons": """
            CREATE TABLE coupons (
                id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
                code VARCHAR(50) NOT NULL,
                description TEXT,
                discount_type VARCHAR(20) NOT NULL
                    CHECK (discount_type IN ('percentage', 'fixed', 'free_shipping')),
                discount_value INTEGER NOT NULL,
                max_discount_amount INTEGER,
                minimum_order_amount INTEGER,
                usage_limit INTEGER,
                usage_count INTEGER NOT NULL DEFAULT 0,
                is_active BOOLEAN NOT NULL DEFAULT TRUE,
                valid_from TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                valid_until TIMESTAMPTZ,
                created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
                
                CONSTRAINT coupons_code_unique UNIQUE (code)
            );
            
            CREATE INDEX idx_coupons_code ON coupons(code) WHERE is_active = TRUE;
        """
    }
}



# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE FINALE: RIFERIMENTI E BEST PRACTICES
# ═══════════════════════════════════════════════════════════════════════════════

DYNAMODB_BEST_PRACTICES = {
    
    "single_table_design": {
        "descrizione": "Tutti gli item in una sola tabella con PK/SK polimorfici",
        "vantaggi": [
            "Riduce costi (una sola tabella)",
            "Permette transazioni multi-item",
            "Semplifica backup e gestione"
        ],
        "pattern_pk_sk": {
            "ENTITY#id + METADATA": "Item principale entità",
            "ENTITY#id + RELATION#id": "Relazione 1:N",
            "ENTITY#id + RELATION#timestamp#id": "Relazione 1:N ordinata"
        }
    },
    
    "gsi_strategy": {
        "GSI1": "Indice invertito (es: trova entità per owner/parent)",
        "GSI2": "Indice per filtri comuni (es: status, categoria)",
        "GSI3": "Indice sparse per lookup specifici (es: email, SKU)"
    },
    
    "contatori": {
        "metodo": "UpdateItem con ADD/SET atomico",
        "esempio": """
            UpdateItem:
              Key: {PK: 'USER#123', SK: 'STATS'}
              UpdateExpression: 'SET followers_count = followers_count + :inc'
              ExpressionAttributeValues: {':inc': 1}
        """
    },
    
    "ttl": {
        "attributo": "expires_at_epoch",
        "formato": "Unix timestamp (secondi)",
        "uso": "Sessioni, token verifica, carrelli abbandonati, feed items"
    },
    
    "transactions": {
        "uso": "TransactWriteItems per operazioni multi-item atomiche",
        "limite": "25 item per transazione",
        "esempio": "Creazione ordine (ORDER + ORDER_ITEMS + inventory update)"
    }
}

AURORA_BEST_PRACTICES = {
    
    "indici": {
        "btree": "Default, per equality e range queries",
        "gin": "Per array, JSONB, full-text search (trigram)",
        "gist": "Per range overlapping (prenotazioni), geometria"
    },
    
    "jsonb": {
        "uso": "Dati semi-strutturati (settings, metadata, snapshot)",
        "indici": "CREATE INDEX ON table USING gin(column)",
        "query": "column->>'key' = 'value' oppure column @> '{\"key\": \"value\"}'"
    },
    
    "triggers": {
        "updated_at": "Trigger automatico per aggiornare timestamp",
        "contatori": "Trigger per mantenere count denormalizzati",
        "generazione_codici": "Trigger per booking_code, order_number"
    },
    
    "constraints": {
        "check": "Validazione valori (status, rating range)",
        "unique": "Unicità (email, SKU, composite keys)",
        "exclude": "Prevenire overlap (prenotazioni)"
    },
    
    "performance": {
        "partial_indexes": "WHERE clause per indici su subset",
        "covering_indexes": "INCLUDE per evitare table lookup",
        "connection_pooling": "PgBouncer per Lambda"
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# INDICE TABELLE PER ARCHITETTURA
# ═══════════════════════════════════════════════════════════════════════════════

INDICE_TABELLE = {
    
    "DynamoDB": {
        "tabella_principale": "MainTable",
        "gsi": ["GSI1", "GSI2", "GSI3"],
        "entita_per_modulo": {
            "IDENTITY": ["USER", "PROFILE", "SESSION", "USER_SETTINGS", "VERIFICATION_TOKEN"],
            "CONTENT": ["POST", "COMMENT", "REACTION", "MEDIA_ITEM", "HASHTAG"],
            "SOCIAL": ["FOLLOW", "BLOCK", "USER_STATS", "FEED_ITEM"],
            "MESSAGING": ["CONVERSATION", "CONVERSATION_PARTICIPANT", "MESSAGE", "MESSAGE_READ_RECEIPT", "NOTIFICATION"],
            "MARKETPLACE": ["LISTING", "CATEGORY", "SELLER_PROFILE", "OFFER", "TRANSACTION", "REVIEW", "FAVORITE"],
            "BOOKING": ["BOOKABLE_RESOURCE", "AVAILABILITY_RULE", "AVAILABILITY_EXCEPTION", "BOOKING"],
            "COMMERCE": ["PRODUCT", "PRODUCT_VARIANT", "CART", "CART_ITEM", "ORDER", "ORDER_ITEM", "PRODUCT_REVIEW"]
        }
    },
    
    "Aurora_PostgreSQL": {
        "tabelle_per_modulo": {
            "IDENTITY": ["users", "profiles", "sessions", "user_settings", "verification_tokens"],
            "CONTENT": ["posts", "comments", "reactions", "media_items", "hashtags", "post_hashtags"],
            "SOCIAL": ["follows", "blocks", "user_stats", "feed_items"],
            "MESSAGING": ["conversations", "conversation_participants", "messages", "message_read_receipts", "notifications"],
            "MARKETPLACE": ["categories", "listings", "seller_profiles", "offers", "transactions", "marketplace_reviews", "favorites"],
            "BOOKING": ["bookable_resources", "availability_rules", "availability_exceptions", "bookings"],
            "COMMERCE": ["product_categories", "products", "product_variants", "carts", "cart_items", "orders", "order_items", "product_reviews", "coupons"]
        },
        "estensioni_richieste": ["uuid-ossp", "pgcrypto", "pg_trgm", "btree_gin", "ltree", "btree_gist"]
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# MAPPING COMPOSIZIONI -> TABELLE
# ═══════════════════════════════════════════════════════════════════════════════

MAPPING_COMPOSIZIONI = {
    
    "SOCIAL_NETWORK": {
        "DynamoDB": {
            "tabelle": 1,
            "gsi": 3,
            "entita": ["USER", "PROFILE", "SESSION", "USER_SETTINGS", "VERIFICATION_TOKEN",
                      "POST", "COMMENT", "REACTION", "MEDIA_ITEM", "HASHTAG",
                      "FOLLOW", "BLOCK", "USER_STATS", "FEED_ITEM",
                      "CONVERSATION", "CONVERSATION_PARTICIPANT", "MESSAGE", "NOTIFICATION"]
        },
        "Aurora": {
            "tabelle": 18,
            "note": "Feed può usare vista materializzata invece di tabella"
        }
    },
    
    "MARKETPLACE_C2C": {
        "DynamoDB": {
            "tabelle": 1,
            "gsi": 3,
            "entita": ["USER", "PROFILE", "SESSION", 
                      "LISTING", "CATEGORY", "SELLER_PROFILE", "OFFER", "TRANSACTION", "REVIEW", "FAVORITE",
                      "CONVERSATION", "CONVERSATION_PARTICIPANT", "MESSAGE", "NOTIFICATION"]
        },
        "Aurora": {
            "tabelle": 15
        }
    },
    
    "ECOMMERCE_B2C": {
        "DynamoDB": {
            "tabelle": 1,
            "gsi": 3,
            "entita": ["USER", "PROFILE", "SESSION",
                      "PRODUCT", "PRODUCT_VARIANT", "CART", "CART_ITEM", 
                      "ORDER", "ORDER_ITEM", "PRODUCT_REVIEW"]
        },
        "Aurora": {
            "tabelle": 14
        }
    },
    
    "BOOKING_PLATFORM": {
        "DynamoDB": {
            "tabelle": 1,
            "gsi": 3,
            "entita": ["USER", "PROFILE", "SESSION",
                      "BOOKABLE_RESOURCE", "AVAILABILITY_RULE", "AVAILABILITY_EXCEPTION", "BOOKING",
                      "CONVERSATION", "CONVERSATION_PARTICIPANT", "MESSAGE", "NOTIFICATION"]
        },
        "Aurora": {
            "tabelle": 12
        }
    }
}

# ═══════════════════════════════════════════════════════════════════════════════
# FINE CATALOGO DATA MODEL v1.0
# ═══════════════════════════════════════════════════════════════════════════════
# 
# STATISTICHE:
# - Moduli: 7
# - Entità DynamoDB: 32 (single-table design)
# - Tabelle Aurora: 35+
# - Access patterns documentati: 60+
#
# PROSSIMI CATALOGHI:
# - CATALOGO-API: OpenAPI specs derivate dalle operazioni
# - CATALOGO-CODICE: Template Lambda/ECS con logica
#
# ═══════════════════════════════════════════════════════════════════════════════

