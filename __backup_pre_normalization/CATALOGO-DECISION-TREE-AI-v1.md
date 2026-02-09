---
# Integrato da: 02-OUTPUT-DECISION-TREE.md
# Data integrazione: 2026-01-29 14:51
# Generato con: Gemini 2.5 Flash / DeepSeek R1
---

Il presente documento definisce la logica esatta per la selezione dei moduli software da utilizzare in base alla richiesta dell'utente, agendo come un albero decisionale per l'architettura del sistema.

---

# DECISION-TREE.md

## 1. KEYWORDS DETECTION

Questa sezione definisce le tabelle di mapping tra le parole chiave rilevate nella richiesta dell'utente e i tipi di piattaforma associati, inclusi i moduli da caricare e un livello di confidenza. I moduli `CORE` sono sempre inclusi.

### E-COMMERCE Keywords:
| Keyword | Confidence | Moduli da caricare |
|:--------|:-----------|:-------------------|
| "e-commerce" | 100% | CORE + ECOMMERCE/* |
| "negozio" | 100% | CORE + ECOMMERCE/* |
| "vendita" | 95% | CORE + ECOMMERCE/* |
| "prodotti" | 90% | CORE + ECOMMERCE/PRODUCTS |
| "carrello" | 100% | CORE + ECOMMERCE/CART |
| "checkout" | 100% | CORE + ECOMMERCE/CHECKOUT |
| "pagamenti" | 95% | CORE + ECOMMERCE/PAYMENTS |
| "ordini" | 90% | CORE + ECOMMERCE/ORDERS |
| "inventario" | 85% | CORE + ECOMMERCE/INVENTORY |
| "shop" | 100% | CORE + ECOMMERCE/* |
| "store" | 95% | CORE + ECOMMERCE/* |
| "merce" | 80% | CORE + ECOMMERCE/PRODUCTS |
| "catalogo" | 90% | CORE + ECOMMERCE/PRODUCTS |
| "prezzi" | 85% | CORE + ECOMMERCE/PRODUCTS |
| "sconti" | 90% | CORE + ECOMMERCE/DISCOUNTS |
| "promozioni" | 85% | CORE + ECOMMERCE/DISCOUNTS |
| "spedizione" | 90% | CORE + ECOMMERCE/SHIPPING |
| "resi" | 85% | CORE + ECOMMERCE/RETURNS |
| "recensioni" | 80% | CORE + ECOMMERCE/REVIEWS |
| "valutazioni" | 80% | CORE + ECOMMERCE/REVIEWS |
| "wishlist" | 90% | CORE + ECOMMERCE/WISHLIST |
| "preferiti" | 85% | CORE + ECOMMERCE/WISHLIST |
| "confronta" | 75% | CORE + ECOMMERCE/PRODUCTS_COMPARE |
| "varianti" | 90% | CORE + ECOMMERCE/PRODUCTS_VARIANTS |
| "taglie" | 95% | CORE + ECOMMERCE/PRODUCTS_VARIANTS |
| "colori" | 95% | CORE + ECOMMERCE/PRODUCTS_VARIANTS |
| "magazzino" | 85% | CORE + ECOMMERCE/INVENTORY |
| "stock" | 85% | CORE + ECOMMERCE/INVENTORY |
| "fatturazione" | 70% | CORE + ECOMMERCE/ORDERS_INVOICING |
| "clienti" | 75% | CORE + ECOMMERCE/CUSTOMERS |
| "acquisti" | 90% | CORE + ECOMMERCE/ORDERS |
| "transazioni" | 85% | CORE + ECOMMERCE/PAYMENTS |
| "gateway" | 90% | CORE + ECOMMERCE/PAYMENTS |
| "coupon" | 95% | CORE + ECOMMERCE/DISCOUNTS |
| "voucher" | 90% | CORE + ECOMMERCE/DISCOUNTS |
| "abbonamenti" | 80% | CORE + ECOMMERCE/SUBSCRIPTIONS |
| "dropshipping" | 70% | CORE + ECOMMERCE/SUPPLIERS |
| "marketplace" | 75% | CORE + ECOMMERCE/MULTI_VENDOR |
| "vendor" | 70% | CORE + ECOMMERCE/MULTI_VENDOR |
| "fornitori" | 65% | CORE + ECOMMERCE/SUPPLIERS |
| "logistica" | 70% | CORE + ECOMMERCE/SHIPPING |
| "spedizioniere" | 70% | CORE + ECOMMERCE/SHIPPING |
| "tasse" | 70% | CORE + ECOMMERCE/TAXES |
| "iva" | 70% | CORE + ECOMMERCE/TAXES |
| "listino" | 75% | CORE + ECOMMERCE/PRODUCTS |
| "offerte" | 85% | CORE + ECOMMERCE/DISCOUNTS |
| "bestseller" | 70% | CORE + ECOMMERCE/PRODUCTS_LISTINGS |
| "novità" | 70% | CORE + ECOMMERCE/PRODUCTS_LISTINGS |
| "digitale" | 80% | CORE + ECOMMERCE/DIGITAL_PRODUCTS |
| "download" | 80% | CORE + ECOMMERCE/DIGITAL_PRODUCTS |
| "servizi" | 75% | CORE + ECOMMERCE/SERVICES |
| "prenotazioni" | 70% | CORE + ECOMMERCE/BOOKINGS |
| "appuntamenti" | 70% | CORE + ECOMMERCE/BOOKINGS |
| "gift card" | 85% | CORE + ECOMMERCE/GIFT_CARDS |
| "buono regalo" | 85% | CORE + ECOMMERCE/GIFT_CARDS |
| "affiliazione" | 60% | CORE + ECOMMERCE/AFFILIATES |
| "referral" | 60% | CORE + ECOMMERCE/AFFILIATES |
| "dashboard venditore" | 80% | CORE + ECOMMERCE/ADMIN_VENDOR |
| "gestione ordini" | 90% | CORE + ECOMMERCE/ADMIN_ORDERS |
| "gestione prodotti" | 90% | CORE + ECOMMERCE/ADMIN_PRODUCTS |
| "analisi vendite" | 80% | CORE + ECOMMERCE/ADMIN_REPORTS |
| "reportistica" | 75% | CORE + ECOMMERCE/ADMIN_REPORTS |

### SOCIAL Keywords:
| Keyword | Confidence | Moduli da caricare |
|:--------|:-----------|:-------------------|
| "social" | 100% | CORE + SOCIAL/* |
| "community" | 95% | CORE + SOCIAL/* |
| "utenti" | 70% | CORE + SOCIAL/PROFILES |
| "post" | 90% | CORE + SOCIAL/POSTS |
| "commenti" | 85% | CORE + SOCIAL/COMMENTS |
| "like" | 90% | CORE + SOCIAL/LIKES |
| "follow" | 95% | CORE + SOCIAL/FOLLOW |
| "feed" | 95% | CORE + SOCIAL/FEED |
| "profilo" | 80% | CORE + SOCIAL/PROFILES |
| "amici" | 90% | CORE + SOCIAL/FOLLOW |
| "connessioni" | 85% | CORE + SOCIAL/FOLLOW |
| "gruppi" | 95% | CORE + SOCIAL/GROUPS |
| "eventi" | 80% | CORE + SOCIAL/EVENTS |
| "messaggi" | 95% | CORE + SOCIAL/MESSAGING |
| "chat" | 95% | CORE + SOCIAL/MESSAGING |
| "notifiche" | 90% | CORE + SOCIAL/NOTIFICATIONS |
| "condividi" | 85% | CORE + SOCIAL/SHARING |
| "reazioni" | 80% | CORE + SOCIAL/LIKES |
| "stories" | 70% | CORE + SOCIAL/STORIES |
| "dirette" | 70% | CORE + SOCIAL/LIVE_STREAMING |
| "live" | 70% | CORE + SOCIAL/LIVE_STREAMING |
| "hashtag" | 85% | CORE + SOCIAL/POSTS_TAGGING |
| "menzioni" | 85% | CORE + SOCIAL/POSTS_TAGGING |
| "tag" | 80% | CORE + SOCIAL/POSTS_TAGGING |
| "moderazione" | 90% | CORE + SOCIAL/MODERATION |
| "privacy" | 85% | CORE + SOCIAL/PRIVACY |
| "impostazioni account" | 75% | CORE + SOCIAL/PROFILES_SETTINGS |
| "bacheca" | 80% | CORE + SOCIAL/FEED |
| "news feed" | 85% | CORE + SOCIAL/FEED |
| "interazioni" | 80% | CORE + SOCIAL/ENGAGEMENT |
| "engagement" | 75% | CORE + SOCIAL/ENGAGEMENT |
| "influencer" | 60% | CORE + SOCIAL/CREATORS |
| "creatori" | 65% | CORE + SOCIAL/CREATORS |
| "contenuti" | 70% | CORE + SOCIAL/POSTS |
| "media" | 75% | CORE + SOCIAL/MEDIA |
| "foto" | 80% | CORE + SOCIAL/MEDIA |
| "video" | 80% | CORE + SOCIAL/MEDIA |
| "album" | 75% | CORE + SOCIAL/MEDIA_GALLERY |
| "galleria" | 75% | CORE + SOCIAL/MEDIA_GALLERY |
| "sondaggi" | 80% | CORE + SOCIAL/POLLS |
| "quiz" | 70% | CORE + SOCIAL/QUIZZES |
| "forum" | 90% | CORE + SOCIAL/FORUM |
| "discussioni" | 85% | CORE + SOCIAL/FORUM |
| "badge" | 70% | CORE + SOCIAL/GAMIFICATION |
| "punti" | 65% | CORE + SOCIAL/GAMIFICATION |
| "livelli" | 65% | CORE + SOCIAL/GAMIFICATION |
| "classifiche" | 60% | CORE + SOCIAL/GAMIFICATION |
| "reputazione" | 60% | CORE + SOCIAL/GAMIFICATION |
| "segnalazioni" | 85% | CORE + SOCIAL/MODERATION |
| "blocco" | 85% | CORE + SOCIAL/PRIVACY |

### BLOG Keywords:
| Keyword | Confidence | Moduli da caricare |
|:--------|:-----------|:-------------------|
| "blog" | 100% | CORE + BLOG/* |
| "articoli" | 95% | CORE + BLOG/ARTICLES |
| "post" | 90% | CORE + BLOG/ARTICLES |
| "notizie" | 90% | CORE + BLOG/ARTICLES |
| "editoriale" | 85% | CORE + BLOG/ARTICLES |
| "redazione" | 80% | CORE + BLOG/ADMIN_EDITOR |
| "autore" | 85% | CORE + BLOG/AUTHORS |
| "lettori" | 70% | CORE + BLOG/READERS |
| "commenti" | 90% | CORE + BLOG/COMMENTS |
| "categorie" | 95% | CORE + BLOG/CATEGORIES |
| "tag" | 90% | CORE + BLOG/TAGS |
| "archivi" | 85% | CORE + BLOG/ARCHIVES |
| "ricerca" | 80% | CORE + BLOG/SEARCH |
| "newsletter" | 95% | CORE + BLOG/NEWSLETTER |
| "iscrizione" | 90% | CORE + BLOG/NEWSLETTER |
| "feed RSS" | 80% | CORE + BLOG/RSS |
| "SEO" | 90% | CORE + BLOG/SEO |
| "ottimizzazione" | 85% | CORE + BLOG/SEO |
| "pubblicazione" | 90% | CORE + BLOG/ADMIN_EDITOR |
| "bozze" | 85% | CORE + BLOG/ADMIN_EDITOR |
| "programmazione" | 80% | CORE + BLOG/ADMIN_EDITOR |
| "immagini" | 75% | CORE + BLOG/MEDIA |
| "video" | 75% | CORE + BLOG/MEDIA |
| "galleria" | 70% | CORE + BLOG/MEDIA_GALLERY |
| "embed" | 70% | CORE + BLOG/ARTICLES |
| "condivisione" | 80% | CORE + BLOG/SHARING |
| "link" | 70% | CORE + BLOG/ARTICLES |
| "backlink" | 60% | CORE + BLOG/SEO |
| "guest post" | 70% | CORE + BLOG/AUTHORS |
| "monetizzazione" | 60% | CORE + BLOG/MONETIZATION |
| "ads" | 60% | CORE + BLOG/MONETIZATION |
| "affiliazione" | 60% | CORE + BLOG/MONETIZATION |
| "podcast" | 70% | CORE + BLOG/PODCASTS |
| "webinar" | 65% | CORE + BLOG/WEBINARS |
| "tutorial" | 80% | CORE + BLOG/ARTICLES |
| "guide" | 80% | CORE + BLOG/ARTICLES |
| "recensioni" | 75% | CORE + BLOG/ARTICLES |
| "interviste" | 75% | CORE + BLOG/ARTICLES |
| "case study" | 70% | CORE + BLOG/ARTICLES |
| "whitepaper" | 70% | CORE + BLOG/ARTICLES |

### WEBSITE Keywords:
| Keyword | Confidence | Moduli da caricare |
|:--------|:-----------|:-------------------|
| "sito web" | 100% | CORE + WEBSITE/* |
| "pagina" | 95% | CORE + WEBSITE/PAGES |
| "contatti" | 100% | CORE + WEBSITE/CONTACT |
| "chi siamo" | 95% | CORE + WEBSITE/PAGES |
| "servizi" | 90% | CORE + WEBSITE/PAGES |
| "portfolio" | 85% | CORE + WEBSITE/PORTFOLIO |
| "galleria" | 80% | CORE + WEBSITE/GALLERY |
| "form" | 90% | CORE + WEBSITE/FORMS |
| "modulo" | 90% | CORE + WEBSITE/FORMS |
| "navigazione" | 95% | CORE + WEBSITE/MENU |
| "menu" | 95% | CORE + WEBSITE/MENU |
| "footer" | 80% | CORE + WEBSITE/LAYOUT |
| "header" | 80% | CORE + WEBSITE/LAYOUT |
| "layout" | 75% | CORE + WEBSITE/LAYOUT |
| "design" | 70% | CORE + WEBSITE/THEMING |
| "responsive" | 70% | CORE + WEBSITE/THEMING |
| "mobile" | 65% | CORE + WEBSITE/THEMING |
| "desktop" | 65% | CORE + WEBSITE/THEMING |
| "SEO" | 85% | CORE + WEBSITE/SEO |
| "analytics" | 80% | CORE + WEBSITE/ANALYTICS |
| "mappa del sito" | 80% | CORE + WEBSITE/SEO |
| "privacy policy" | 95% | CORE + WEBSITE/LEGAL |
| "termini e condizioni" | 95% | CORE + WEBSITE/LEGAL |
| "cookie policy" | 95% | CORE + WEBSITE/LEGAL |
| "multilingua" | 85% | CORE + WEBSITE/LOCALIZATION |
| "traduzione" | 80% | CORE + WEBSITE/LOCALIZATION |
| "accessibilità" | 75% | CORE + WEBSITE/ACCESSIBILITY |
| "ricerca interna" | 80% | CORE + WEBSITE/SEARCH |
| "sitemap" | 80% | CORE + WEBSITE/SEO |
| "hosting" | 60% | CORE |
| "dominio" | 60% | CORE |

## 2. DECISION ALGORITHM

Questo pseudocodice definisce la logica esatta per analizzare la richiesta dell'utente, determinare il tipo di piattaforma principale e secondaria, e selezionare i moduli appropriati.

```pseudocode
// Mappatura delle keyword ai loro tipi, confidenza e moduli associati.
// Queste strutture dati sono generate dalle tabelle della Sezione 1.
// Esempio: ECOMMERCE_KEYWORDS = { "e-commerce": { confidence: 100, modules: ["ECOMMERCE/*"] }, ... }
GLOBAL_KEYWORDS_MAP = {
    "ecommerce": ECOMMERCE_KEYWORDS,
    "social": SOCIAL_KEYWORDS,
    "blog": BLOG_KEYWORDS,
    "website": WEBSITE_KEYWORDS
}

// Moduli Core che sono sempre inclusi in qualsiasi piattaforma.
CORE_MODULES = ["AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE"]

FUNCTION extract_keywords(user_prompt):
    // Semplice tokenizzazione e normalizzazione del testo.
    // Potrebbe essere migliorato con NLP per stemming, lemmatizzazione, riconoscimento di entità.
    normalized_prompt = user_prompt.lower()
    tokens = normalized_prompt.split() // Suddivisione per spazi
    
    // Rimuovi duplicati e filtra parole comuni (stop words) se necessario.
    unique_keywords = SET()
    FOR token IN tokens:
        // Potenziale espansione per frasi chiave multiple
        // Esempio: "negozio online" -> "negozio", "online", "e-commerce"
        unique_keywords.add(token)
    
    RETURN LIST(unique_keywords)

FUNCTION analyze_request(user_prompt):
    keywords_found = extract_keywords(user_prompt)
    
    scores = {
        "ecommerce": 0,
        "social": 0,
        "blog": 0,
        "website": 0
    }
    
    // Raccoglie i moduli specifici suggeriti dalle keyword
    suggested_specific_modules = SET()
    
    FOR each keyword IN keywords_found:
        FOR platform_type, keyword_map IN GLOBAL_KEYWORDS_MAP:
            IF keyword IN keyword_map:
                entry = keyword_map[keyword]
                scores[platform_type] += entry.confidence
                FOR module IN entry.modules:
                    // Aggiungi solo moduli specifici, non CORE
                    IF module NOT IN CORE_MODULES:
                        suggested_specific_modules.add(module)
    
    // Determina il tipo primario
    IF NOT scores.values() OR max(scores.values()) == 0:
        // Nessuna keyword rilevante trovata, default a "website"
        primary_type = "website"
        // Se non ci sono score, i moduli specifici saranno solo quelli di base per un website
        suggested_specific_modules.add("WEBSITE/*") 
    ELSE:
        primary_type = max(scores, key=scores.get)
    
    // Determina i tipi secondari
    // Un tipo è secondario se il suo score è superiore a una soglia (es. 50)
    // e non è il tipo primario, e il suo score è almeno il 20% dello score primario
    // per evitare secondari irrilevanti.
    secondary_types = []
    primary_score = scores[primary_type]
    
    FOR platform_type, score IN scores.items():
        IF platform_type != primary_type AND score > 50 AND score >= (primary_score * 0.20):
            secondary_types.append(platform_type)
    
    // Ordina i tipi secondari per score decrescente
    secondary_types.sort(key=lambda t: scores[t], reverse=True)
    
    // Ottiene i moduli finali basandosi sulla matrice di selezione
    final_modules = get_modules_from_matrix(primary_type, secondary_types, suggested_specific_modules)
    
    // Estrae le feature flags basandosi sulle keyword e i moduli selezionati
    features = extract_feature_flags(keywords_found, final_modules)

    // Genera le route e lo schema DB
    routes = generate_routes(primary_type, secondary_types, features)
    schema_models = generate_db_schema(primary_type, secondary_types, features)

    RETURN {
        "primary": primary_type,
        "secondary": secondary_types,
        "modules": final_modules,
        "features": features,
        "routes": routes,
        "schema_models": schema_models
    }

// --- Funzioni ausiliarie (implementazione concettuale, dettagliate nelle sezioni successive) ---

FUNCTION get_modules_from_matrix(primary_type, secondary_types, suggested_specific_modules):
    // Questa funzione consulta la "MODULE SELECTION MATRIX" per determinare l'insieme finale di moduli.
    // Inizializza con i moduli CORE.
    modules = SET(CORE_MODULES)
    
    // Aggiunge i moduli specifici per il tipo primario.
    // La logica qui dovrebbe espandere "ECOMMERCE/*" in moduli specifici come PRODUCTS, CART, etc.
    // Questo è dettagliato nella Sezione 3.
    modules.update(get_base_modules_for_type(primary_type))
    
    // Aggiunge i moduli per i tipi secondari, gestendo le intersezioni o le versioni semplificate.
    FOR sec_type IN secondary_types:
        modules.update(get_base_modules_for_type(sec_type, is_secondary=TRUE))
    
    // Integra i moduli suggeriti direttamente dalle keyword, assicurandosi che siano compatibili
    // con i tipi primari/secondari selezionati.
    FOR suggested_module IN suggested_specific_modules:
        // Esempio: se "ECOMMERCE/PRODUCTS_VARIANTS" è suggerito, assicurati che ECOMMERCE sia primario/secondario
        IF suggested_module.startswith(primary_type.upper() + "/") OR \
           ANY(suggested_module.startswith(s.upper() + "/") FOR s IN secondary_types):
            modules.add(suggested_module)
        // Gestione di moduli generici come "COMMENTS" che possono essere in BLOG o SOCIAL
        IF "COMMENTS" in suggested_module AND ("BLOG" in modules OR "SOCIAL" in modules):
            modules.add(suggested_module) // Aggiunge il modulo specifico (es. BLOG/COMMENTS o SOCIAL/COMMENTS)
    
    RETURN sorted(LIST(modules))

FUNCTION extract_feature_flags(keywords_found, selected_modules):
    // Questa funzione analizza le keyword e i moduli selezionati per attivare feature flags.
    // Dettagliata nella Sezione 4.
    features = {}
    // Esempio: se "taglie" o "colori" è in keywords_found E "ECOMMERCE/PRODUCTS" è in selected_modules,
    // allora features["ECOMMERCE.PRODUCTS.variants"] = true.
    // ... logica per popolare l'oggetto features ...
    RETURN features

FUNCTION generate_routes(primary_type, secondary_types, features):
    // Genera le route basandosi sui tipi e le feature attivate.
    // Dettagliata nella Sezione 5.
    routes = []
    // ... logica per popolare la lista routes ...
    RETURN routes

FUNCTION generate_db_schema(primary_type, secondary_types, features):
    // Seleziona i modelli Prisma per lo schema del database.
    // Dettagliata nella Sezione 6.
    schema_models = []
    // ... logica per popolare la lista schema_models ...
    RETURN schema_models
```

## 3. MODULE SELECTION MATRIX

Questa tabella elenca in modo completo quali moduli includere per ogni combinazione di tipo primario e secondario. I moduli `CORE` sono sempre inclusi e non sono ripetuti nella colonna "Moduli Core" per brevità, ma sono implicitamente presenti.

**Moduli Core (sempre inclusi):** `AUTH`, `DB`, `UI`, `API`, `ERROR`, `NOTIFICATIONS_CORE`

| Tipo Primario | Tipo Secondario | Moduli Core | Moduli Specifici |
|:--------------|:----------------|:------------|:-----------------|
| ecommerce | - | (Impliciti) | ECOMMERCE/PRODUCTS, ECOMMERCE/CART, ECOMMERCE/CHECKOUT, ECOMMERCE/ORDERS, ECOMMERCE/PAYMENTS, ECOMMERCE/ADMIN, ECOMMERCE/INVENTORY, ECOMMERCE/SHIPPING, ECOMMERCE/DISCOUNTS, ECOMMERCE/CUSTOMERS |
| ecommerce | social | (Impliciti) | ECOMMERCE/PRODUCTS, ECOMMERCE/CART, ECOMMERCE/CHECKOUT, ECOMMERCE/ORDERS, ECOMMERCE/PAYMENTS, ECOMMERCE/ADMIN, ECOMMERCE/REVIEWS, SOCIAL/PROFILES, SOCIAL/COMMENTS (su prodotti), SOCIAL/LIKES (su prodotti), SOCIAL/SHARING |
| ecommerce | blog | (Impliciti) | ECOMMERCE/PRODUCTS, ECOMMERCE/CART, ECOMMERCE/CHECKOUT, ECOMMERCE/ORDERS, ECOMMERCE/PAYMENTS, ECOMMERCE/ADMIN, BLOG/ARTICLES (per news/guide prodotti), BLOG/CATEGORIES, BLOG/TAGS, BLOG/COMMENTS |
| ecommerce | website | (Impliciti) | ECOMMERCE/PRODUCTS, ECOMMERCE/CART, ECOMMERCE/CHECKOUT, ECOMMERCE/ORDERS, ECOMMERCE/PAYMENTS, ECOMMERCE/ADMIN, WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO |
| social | - | (Impliciti) | SOCIAL/PROFILES, SOCIAL/POSTS, SOCIAL/COMMENTS, SOCIAL/LIKES, SOCIAL/FOLLOW, SOCIAL/FEED, SOCIAL/MESSAGING, SOCIAL/NOTIFICATIONS, SOCIAL/GROUPS, SOCIAL/MODERATION, SOCIAL/PRIVACY |
| social | ecommerce | (Impliciti) | SOCIAL/PROFILES, SOCIAL/POSTS (per aggiornamenti), SOCIAL/COMMENTS, SOCIAL/LIKES, SOCIAL/FOLLOW, SOCIAL/FEED, ECOMMERCE/PRODUCTS (visualizzazione), ECOMMERCE/REVIEWS, ECOMMERCE/WISHLIST (integrazione) |
| social | blog | (Impliciti) | SOCIAL/PROFILES, SOCIAL/POSTS (per articoli), SOCIAL/COMMENTS, SOCIAL/LIKES, SOCIAL/FOLLOW, SOCIAL/FEED, BLOG/ARTICLES (condivisione), BLOG/AUTHORS, BLOG/COMMENTS (integrati) |
| social | website | (Impliciti) | SOCIAL/PROFILES, SOCIAL/POSTS (embed), SOCIAL/COMMENTS (embed), SOCIAL/SHARING, WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO |
| blog | - | (Impliciti) | BLOG/ARTICLES, BLOG/CATEGORIES, BLOG/TAGS, BLOG/AUTHORS, BLOG/COMMENTS, BLOG/ADMIN_EDITOR, BLOG/SEO, BLOG/NEWSLETTER, BLOG/SEARCH, BLOG/ARCHIVES |
| blog | ecommerce | (Impliciti) | BLOG/ARTICLES (guide, recensioni), BLOG/CATEGORIES, BLOG/TAGS, BLOG/COMMENTS, ECOMMERCE/PRODUCTS (embed), ECOMMERCE/REVIEWS (integrazione), ECOMMERCE/DISCOUNTS (promozione) |
| blog | social | (Impliciti) | BLOG/ARTICLES, BLOG/CATEGORIES, BLOG/TAGS, BLOG/AUTHORS, BLOG/COMMENTS (integrati), SOCIAL/PROFILES (autori), SOCIAL/SHARING, SOCIAL/LIKES (su articoli), SOCIAL/COMMENTS (su articoli) |
| blog | website | (Impliciti) | BLOG/ARTICLES, BLOG/CATEGORIES, BLOG/TAGS, BLOG/COMMENTS, WEBSITE/PAGES (integrazione), WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO |
| website | - | (Impliciti) | WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/FORMS, WEBSITE/MENU, WEBSITE/LAYOUT, WEBSITE/SEO, WEBSITE/ANALYTICS, WEBSITE/LEGAL, WEBSITE/SEARCH |
| website | ecommerce | (Impliciti) | WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO, ECOMMERCE/PRODUCTS (vetrina), ECOMMERCE/CART (widget), ECOMMERCE/CHECKOUT (link), ECOMMERCE/PAYMENTS (link) |
| website | social | (Impliciti) | WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO, SOCIAL/SHARING, SOCIAL/PROFILES (link), SOCIAL/FEED (embed), SOCIAL/COMMENTS (embed) |
| website | blog | (Impliciti) | WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU, WEBSITE/SEO, BLOG/ARTICLES (lista/featured), BLOG/CATEGORIES (link), BLOG/TAGS (link), BLOG/SEARCH (integrato) |
| ecommerce | social, blog | (Impliciti) | ECOMMERCE/* (completo), SOCIAL/PROFILES, SOCIAL/COMMENTS, SOCIAL/LIKES, SOCIAL/SHARING, BLOG/ARTICLES (news/guide), BLOG/COMMENTS (integrati) |
| social | ecommerce, blog | (Impliciti) | SOCIAL/* (completo), ECOMMERCE/PRODUCTS (visualizzazione), ECOMMERCE/REVIEWS, BLOG/ARTICLES (condivisione), BLOG/AUTHORS |
| blog | ecommerce, social | (Impliciti) | BLOG/* (completo), ECOMMERCE/PRODUCTS (embed), ECOMMERCE/DISCOUNTS, SOCIAL/PROFILES (autori), SOCIAL/SHARING, SOCIAL/LIKES |
| website | ecommerce, blog | (Impliciti) | WEBSITE/* (completo), ECOMMERCE/PRODUCTS (vetrina), ECOMMERCE/CART (widget), BLOG/ARTICLES (lista), BLOG/CATEGORIES |
| website | social, blog | (Impliciti) | WEBSITE/* (completo), SOCIAL/SHARING, SOCIAL/FEED (embed), BLOG/ARTICLES (lista), BLOG/COMMENTS (embed) |
| ecommerce | social, website | (Impliciti) | ECOMMERCE/* (completo), SOCIAL/PROFILES, SOCIAL/COMMENTS, SOCIAL/LIKES, WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU |
| social | ecommerce, website | (Impliciti) | SOCIAL/* (completo), ECOMMERCE/PRODUCTS (visualizzazione), ECOMMERCE/REVIEWS, WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU |
| blog | social, website | (Impliciti) | BLOG/* (completo), SOCIAL/PROFILES (autori), SOCIAL/SHARING, WEBSITE/PAGES, WEBSITE/CONTACT, WEBSITE/MENU |
| website | ecommerce, social, blog | (Impliciti) | WEBSITE/* (completo), ECOMMERCE/PRODUCTS (vetrina), SOCIAL/SHARING, BLOG/ARTICLES (lista) |

## 4. FEATURE FLAGS

Questa sezione definisce le sotto-feature opzionali per ogni modulo principale, che possono essere attivate o disattivate in base alle keyword rilevate e alla complessità desiderata.

```typescript
interface ModuleFeatures {
  ECOMMERCE: {
    PRODUCTS: {
      variants: boolean;      // "varianti", "taglie", "colori", "misure"
      reviews: boolean;       // "recensioni", "valutazioni", "stelle"
      wishlist: boolean;      // "preferiti", "wishlist", "lista dei desideri"
      compare: boolean;       // "confronta prodotti", "comparazione"
      bundles: boolean;       // "pacchetti", "bundle", "offerte combinate"
      subscriptions: boolean; // "abbonamenti", "ricorrente"
      digital_products: boolean; // "prodotti digitali", "download"
      services: boolean;      // "servizi", "prenotazioni", "appuntamenti"
    };
    CART: {
      guest_cart: boolean;    // "carrello ospite", "senza registrazione"
      saved_for_later: boolean; // "salva per dopo", "metti da parte"
      coupons: boolean;       // "coupon", "codice sconto", "voucher"
      gift_cards: boolean;    // "gift card", "buono regalo"
    };
    CHECKOUT: {
      one_page_checkout: boolean; // "checkout rapido", "checkout una pagina"
      guest_checkout: boolean;    // "checkout ospite", "senza account"
      multi_step_checkout: boolean; // "checkout a più passi"
    };
    ORDERS: {
      tracking: boolean;      // "tracciamento ordine", "stato spedizione"
      returns: boolean;       // "resi", "gestione resi"
      invoicing: boolean;     // "fatturazione", "ricevute"
    };
    PAYMENTS: {
      multiple_gateways: boolean; // "più metodi di pagamento", "paypal", "stripe"
      recurring_payments: boolean; // "pagamenti ricorrenti", "abbonamenti"
    };
    ADMIN: {
      user_management: boolean; // "gestione utenti", "clienti"
      analytics: boolean;     // "analisi vendite", "reportistica"
      inventory_management: boolean; // "gestione magazzino", "stock"
      shipping_management: boolean; // "gestione spedizioni", "corrieri"
    };
  };
  SOCIAL: {
    PROFILES: {
      custom_fields: boolean; // "campi personalizzati", "bio"
      privacy_settings: boolean; // "impostazioni privacy", "profilo privato"
      followers_list: boolean; // "lista follower"
      following_list: boolean; // "lista seguiti"
    };
    POSTS: {
      media_upload: boolean;  // "carica foto", "carica video"
      rich_text_editor: boolean; // "formattazione testo", "editor avanzato"
      scheduling: boolean;    // "programma post"
      hashtags: boolean;      // "hashtag", "tag"
      mentions: boolean;      // "menzioni", "@utenti"
    };
    COMMENTS: {
      nested_comments: boolean; // "risposte ai commenti", "thread"
      moderation: boolean;    // "moderazione commenti", "approvazione"
      guest_comments: boolean; // "commenti ospite"
    };
    LIKES: {
      reactions: boolean;     // "reazioni", "cuore", "emoji"
    };
    MESSAGING: {
      direct_messages: boolean; // "messaggi privati", "DM"
      group_chats: boolean;   // "chat di gruppo"
    };
    GROUPS: {
      private_groups: boolean; // "gruppi privati", "gruppi chiusi"
      moderation: boolean;    // "moderazione gruppi"
    };
    EVENTS: {
      event_creation: boolean; // "crea evento"
      rsvp: boolean;          // "partecipa evento"
    };
    GAMIFICATION: {
      badges: boolean;        // "badge", "riconoscimenti"
      leaderboards: boolean;  // "classifiche"
    };
  };
  BLOG: {
    ARTICLES: {
      rich_text_editor: boolean; // "editor articoli", "formattazione"
      scheduling: boolean;    // "programma articoli", "pubblicazione"
      drafts_revisions: boolean; // "bozze", "revisioni"
      featured_images: boolean; // "immagine in evidenza"
      embed_media: boolean;   // "embed video", "embed contenuti"
    };
    AUTHORS: {
      author_profiles: boolean; // "profili autore", "bio autore"
      guest_authors: boolean;   // "autori ospiti"
    };
    COMMENTS: {
      moderation: boolean;    // "moderazione commenti blog"
      guest_comments: boolean; // "commenti anonimi"
      social_login_comments: boolean; // "commenta con social"
    };
    SEO: {
      meta_tags: boolean;     // "meta description", "meta title"
      sitemap_generation: boolean; // "generazione sitemap"
      friendly_urls: boolean; // "url amichevoli", "slug"
    };
    NEWSLETTER: {
      subscription_form: boolean; // "modulo iscrizione newsletter"
      integration_mailchimp: boolean; // "integrazione mailchimp"
    };
    SEARCH: {
      full_text_search: boolean; // "ricerca testuale"
    };
  };
  WEBSITE: {
    PAGES: {
      custom_layouts: boolean; // "layout personalizzati", "template"
      drag_and_drop_editor: boolean; // "editor visuale", "costruttore pagine"
      page_sections: boolean; // "sezioni pagina", "blocchi"
    };
    FORMS: {
      custom_form_builder: boolean; // "costruttore moduli", "campi personalizzati"
      email_notifications: boolean; // "notifiche email"
      integrations: boolean;  // "integrazione CRM", "integrazione email marketing"
    };
    MENU: {
      multi_level_menu: boolean; // "menu a più livelli", "sottomenu"
      dynamic_menu: boolean;  // "menu dinamico", "gestione menu"
    };
    SEO: {
      global_settings: boolean; // "impostazioni SEO globali"
      analytics_integration: boolean; // "integrazione google analytics"
    };
    LOCALIZATION: {
      multi_language: boolean; // "sito multilingua", "traduzioni"
    };
    ACCESSIBILITY: {
      wcag_compliance: boolean; // "conformità WCAG", "accessibilità"
    };
  };
}
```

## 5. ROUTE GENERATION

Questa sezione definisce esattamente quali route generare per ogni tipo di piattaforma, basandosi sui moduli e le feature attivate.

#### E-COMMERCE Routes:
```
/                               → Homepage con prodotti featured o categorie
/products                       → Lista di tutti i prodotti
/products/[slug]                → Dettaglio prodotto specifico
/products/search                → Pagina dei risultati di ricerca prodotti
/categories                     → Lista di tutte le categorie di prodotti
/categories/[slug]              → Prodotti per categoria specifica
/cart                           → Pagina del carrello
/checkout                       → Processo di checkout (multi-step o one-page)
/checkout/success               → Pagina di conferma ordine
/account                        → Dashboard utente (richiede AUTH)
/account/orders                 → Lista degli ordini dell'utente
/account/orders/[id]            → Dettaglio di un ordine specifico
/account/settings               → Impostazioni del profilo utente
/account/addresses              → Gestione indirizzi di spedizione/fatturazione
/account/wishlist               → Lista dei desideri dell'utente (se feature.ECOMMERCE.PRODUCTS.wishlist è true)
/account/reviews                → Recensioni lasciate dall'utente
/admin                          → Dashboard amministrativa (richiede AUTH e ruolo admin)
/admin/products                 → Gestione prodotti (CRUD)
/admin/products/[id]/edit       → Modifica prodotto
/admin/categories               → Gestione categorie
/admin/orders                   → Gestione ordini
/admin/orders/[id]              → Dettaglio e gestione ordine
/admin/customers                → Gestione clienti
/admin/discounts                → Gestione sconti e coupon
/admin/inventory                → Gestione inventario
/admin/shipping                 → Gestione spedizioni
/admin/reports                  → Reportistica e analisi vendite
/login                          → Pagina di login
/register                       → Pagina di registrazione
/forgot-password                → Recupero password
```

#### SOCIAL Routes:
```
/                               → Feed principale dell'utente
/feed                           → Feed principale dell'utente
/profile/[username]             → Profilo pubblico dell'utente
/profile/[username]/posts       → Post di un utente specifico
/profile/[username]/followers   → Lista dei follower di un utente
/profile/[username]/following   → Lista degli utenti seguiti da un utente
/posts/[id]                     → Dettaglio di un post specifico
/posts/create                   → Creazione di un nuovo post
/groups                         → Lista di tutti i gruppi
/groups/create                  → Creazione di un nuovo gruppo
/groups/[slug]                  → Pagina di un gruppo specifico
/groups/[slug]/members          → Membri di un gruppo
/messages                       → Inbox dei messaggi privati
/messages/[conversation_id]     → Dettaglio di una conversazione
/notifications                  → Centro notifiche dell'utente
/search                         → Pagina di ricerca (utenti, post, gruppi)
/settings                       → Impostazioni del profilo e della privacy
/settings/privacy               → Impostazioni privacy
/settings/account               → Impostazioni account
/events                         → Lista di eventi (se feature.SOCIAL.EVENTS.event_creation è true)
/events/[slug]                  → Dettaglio evento
/login                          → Pagina di login
/register                       → Pagina di registrazione
/forgot-password                → Recupero password
```

#### BLOG Routes:
```
/                               → Homepage del blog con articoli recenti o featured
/blog                           → Homepage del blog
/blog/[slug]                    → Dettaglio articolo
/blog/category/[slug]           → Articoli per categoria
/blog/tag/[slug]                → Articoli per tag
/blog/author/[slug]             → Articoli di un autore specifico
/blog/archive                   → Archivio articoli per data
/blog/search                    → Pagina dei risultati di ricerca blog
/newsletter                     → Pagina di iscrizione alla newsletter
/admin                          → Dashboard amministrativa del blog
/admin/articles                 → Gestione articoli (CRUD)
/admin/articles/[id]/edit       → Modifica articolo
/admin/categories               → Gestione categorie
/admin/tags                     → Gestione tag
/admin/comments                 → Moderazione commenti
/admin/authors                  → Gestione autori
/admin/newsletter               → Gestione iscritti newsletter
/login                          → Pagina di login (per autori/admin)
/register                       → Pagina di registrazione (per autori/lettori con commenti)
/forgot-password                → Recupero password
```

#### WEBSITE Routes:
```
/                               → Homepage del sito
/about                          → Pagina "Chi Siamo"
/contact                        → Pagina "Contatti" con modulo
/services                       → Pagina "Servizi"
/portfolio                      → Pagina "Portfolio" o "Lavori"
/gallery                        → Pagina "Galleria Immagini"
/privacy-policy                 → Pagina "Privacy Policy"
/terms-of-service               → Pagina "Termini di Servizio"
/cookie-policy                  → Pagina "Cookie Policy"
/sitemap.xml                    → Sitemap XML per SEO
/search                         → Pagina di ricerca interna
/admin                          → Dashboard amministrativa del sito
/admin/pages                    → Gestione pagine (CRUD)
/admin/app/[id]/edit          → Modifica pagina
/admin/menu                     → Gestione menu di navigazione
/admin/forms                    → Gestione moduli e invii
/admin/settings                 → Impostazioni generali del sito
/login                          → Pagina di login (per admin/editor)
/register                       → Pagina di registrazione (se abilitata)
/forgot-password                → Recupero password
```

## 6. DATABASE SCHEMA SELECTION

Questa sezione definisce quali tabelle Prisma includere nello schema del database per ogni tipo di piattaforma e i moduli attivati.

```prisma
// CORE (sempre incluso)
// Modelli base per autenticazione, autorizzazione e gestione utenti.
model User {
  id            String    @id @default(cuid())
  name          String?
  email         String?   @unique
  emailVerified DateTime?
  image         String?
  password      String? // Hashed password
  role          String    @default("USER") // ADMIN, USER, EDITOR, VENDOR
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  accounts      Account[]
  sessions      Session[]
  // Relazioni comuni ad altri moduli
  posts         Post[]
  comments      Comment[]
  likes         Like[]
  orders        Order[]
  cart          Cart?
  profile       Profile?
  articles      Article[]
  reviews       Review[]
  wishlistItems WishlistItem[]
  messagesSent  Message[] @relation("SentMessages")
  messagesReceived Message[] @relation("ReceivedMessages")
  groupMembers  GroupMember[]
  notifications Notification[]
}

model Account {
  id                String    @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User      @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

// ECOMMERCE (se ECOMMERCE è primario o secondario)
model Product {
  id            String         @id @default(cuid())
  name          String
  slug          String         @unique
  description   String?
  price         Decimal        @db.Decimal(10, 2)
  stock         Int            @default(0)
  imageUrl      String?
  categoryId    String?
  category      Category?      @relation(fields: [categoryId], references: [id])
  variants      ProductVariant[]
  cartItems     CartItem[]
  orderItems    OrderItem[]
  reviews       Review[]
  wishlistItems WishlistItem[]
  createdAt     DateTime       @default(now())
  updatedAt     DateTime       @updatedAt
  isDigital     Boolean        @default(false)
  downloadUrl   String?
}

model ProductVariant {
  id        String  @id @default(cuid())
  productId String
  product   Product @relation(fields: [productId], references: [id], onDelete: Cascade)
  name      String // e.g., "Size", "Color"
  value     String // e.g., "M", "Red"
  priceDiff Decimal @db.Decimal(10, 2) @default(0.00)
  stock     Int     @default(0)
}

model Category {
  id        String    @id @default(cuid())
  name      String    @unique
  slug      String    @unique
  products  Product[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Cart {
  id        String     @id @default(cuid())
  userId    String?    @unique // Nullable for guest carts
  user      User?      @relation(fields: [userId], references: [id], onDelete: Cascade)
  items     CartItem[]
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
}

model CartItem {
  id        String  @id @default(cuid())
  cartId    String
  cart      Cart    @relation(fields: [cartId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id])
  variantId String?
  variant   ProductVariant? @relation(fields: [variantId], references: [id])
  quantity  Int
  price     Decimal @db.Decimal(10, 2) // Price at the time of adding to cart
}

model Order {
  id              String         @id @default(cuid())
  userId          String
  user            User           @relation(fields: [userId], references: [id])
  status          String         @default("PENDING") // PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED
  total           Decimal        @db.Decimal(10, 2)
  shippingAddress ShippingAddress @relation(fields: [shippingAddressId], references: [id])
  shippingAddressId String
  billingAddress  BillingAddress  @relation(fields: [billingAddressId], references: [id])
  billingAddressId String
  paymentId       String?        @unique
  payment         Payment?       @relation(fields: [paymentId], references: [id])
  items           OrderItem[]
  createdAt       DateTime       @default(now())
  updatedAt       DateTime       @updatedAt
}

model OrderItem {
  id        String  @id @default(cuid())
  orderId   String
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  productId String
  product   Product @relation(fields: [productId], references: [id])
  variantId String?
  variant   ProductVariant? @relation(fields: [variantId], references: [id])
  quantity  Int
  price     Decimal @db.Decimal(10, 2) // Price at the time of order
}

model Payment {
  id            String    @id @default(cuid())
  orderId       String    @unique
  order         Order     @relation(fields: [orderId], references: [id])
  amount        Decimal   @db.Decimal(10, 2)
  currency      String    @default("EUR")
  method        String    // e.g., "Stripe", "PayPal", "Bank Transfer"
  status        String    @default("PENDING") // PENDING, COMPLETED, FAILED, REFUNDED
  transactionId String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model ShippingAddress {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  fullName  String
  address1  String
  address2  String?
  city      String
  state     String?
  zipCode   String
  country   String
  phone     String?
  isDefault Boolean  @default(false)
  orders    Order[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model BillingAddress {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  fullName  String
  address1  String
  address2  String?
  city      String
  state     String?
  zipCode   String
  country   String
  phone     String?
  isDefault Boolean  @default(false)
  orders    Order[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Review {
  id        String   @id @default(cuid())
  productId String
  product   Product  @relation(fields: [productId], references: [id], onDelete: Cascade)
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  rating    Int      @db.Int @min(1) @max(5)
  comment   String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model WishlistItem {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  productId String
  product   Product  @relation(fields: [productId], references: [id])
  createdAt DateTime @default(now())
}

model Coupon {
  id          String    @id @default(cuid())
  code        String    @unique
  discountType String   // PERCENTAGE, FIXED
  discountValue Decimal @db.Decimal(10, 2)
  minAmount   Decimal?  @db.Decimal(10, 2)
  maxUses     Int?
  uses        Int       @default(0)
  expiresAt   DateTime?
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt
}

// SOCIAL (se SOCIAL è primario o secondario)
model Profile {
  id        String   @id @default(cuid())
  userId    String   @unique
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  username  String   @unique
  bio       String?
  avatarUrl String?
  isPrivate Boolean  @default(false)
  followers Follow[] @relation("Following")
  following Follow[] @relation("Followers")
  posts     Post[]
  groups    GroupMember[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        String    @id @default(cuid())
  content   String
  imageUrl  String?
  videoUrl  String?
  authorId  String
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  comments  Comment[]
  likes     Like[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Comment {
  id        String    @id @default(cuid())
  content   String
  authorId  String
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  postId    String?
  post      Post?     @relation(fields: [postId], references: [id], onDelete: Cascade)
  articleId String? // Per commenti su blog
  article   Article?  @relation(fields: [articleId], references: [id], onDelete: Cascade)
  parentId  String?
  parent    Comment?  @relation("ReplyComments", fields: [parentId], references: [id], onDelete: NoAction, onUpdate: NoAction)
  replies   Comment[] @relation("ReplyComments")
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Like {
  id        String   @id @default(cuid())
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  postId    String?
  post      Post?    @relation(fields: [postId], references: [id], onDelete: Cascade)
  commentId String?
  comment   Comment? @relation(fields: [commentId], references: [id], onDelete: Cascade)
  articleId String? // Per like su blog
  article   Article? @relation(fields: [articleId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@unique([userId, postId]) // Un like per utente per post
  @@unique([userId, commentId]) // Un like per utente per commento
  @@unique([userId, articleId]) // Un like per utente per articolo
}

model Follow {
  id          String   @id @default(cuid())
  followerId  String
  follower    Profile  @relation("Following", fields: [followerId], references: [id], onDelete: Cascade)
  followingId String
  following   Profile  @relation("Followers", fields: [followingId], references: [id], onDelete: Cascade)
  createdAt   DateTime @default(now())

  @@unique([followerId, followingId])
}

model Message {
  id           String    @id @default(cuid())
  conversationId String
  conversation Conversation @relation(fields: [conversationId], references: [id], onDelete: Cascade)
  senderId     String
  sender       User      @relation("SentMessages", fields: [senderId], references: [id])
  content      String
  read         Boolean   @default(false)
  createdAt    DateTime  @default(now())
}

model Conversation {
  id        String    @id @default(cuid())
  messages  Message[]
  participants User[]   @relation("ConversationParticipants")
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Group {
  id        String        @id @default(cuid())
  name      String        @unique
  slug      String        @unique
  description String?
  isPrivate Boolean       @default(false)
  creatorId String
  creator   User          @relation(fields: [creatorId], references: [id])
  members   GroupMember[]
  createdAt DateTime      @default(now())
  updatedAt DateTime      @updatedAt
}

model GroupMember {
  id        String   @id @default(cuid())
  groupId   String
  group     Group    @relation(fields: [groupId], references: [id], onDelete: Cascade)
  profileId String
  profile   Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade)
  role      String   @default("MEMBER") // MEMBER, ADMIN
  joinedAt  DateTime @default(now())

  @@unique([groupId, profileId])
}

model Notification {
  id        String    @id @default(cuid())
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  type      String    // e.g., "LIKE", "COMMENT", "FOLLOW", "MESSAGE"
  message   String
  link      String?   // URL to the related content
  read      Boolean   @default(false)
  createdAt DateTime  @default(now())
}

// BLOG (se BLOG è primario o secondario)
model Article {
  id            String          @id @default(cuid())
  title         String
  slug          String          @unique
  content       String          @db.Text
  authorId      String
  author        User            @relation(fields: [authorId], references: [id])
  categoryId    String?
  category      ArticleCategory? @relation(fields: [categoryId], references: [id])
  tags          Tag[]           @relation("ArticleToTag")
  comments      Comment[]
  likes         Like[]
  isPublished   Boolean         @default(false)
  publishedAt   DateTime?
  createdAt     DateTime        @default(now())
  updatedAt     DateTime        @updatedAt
  featuredImage String?
  seoTitle      String?
  seoDescription String?
}

model ArticleCategory {
  id        String    @id @default(cuid())
  name      String    @unique
  slug      String    @unique
  articles  Article[]
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model Tag {
  id        String    @id @default(cuid())
  name      String    @unique
  slug      String    @unique
  articles  Article[] @relation("ArticleToTag")
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

model NewsletterSubscription {
  id        String   @id @default(cuid())
  email     String   @unique
  isConfirmed Boolean @default(false)
  createdAt DateTime @default(now())
}

// WEBSITE (se WEBSITE è primario o secondario)
model Page {
  id        String   @id @default(cuid())
  title     String
  slug      String   @unique
  content   String   @db.Text
  template  String?  // e.g., "default", "full-width", "contact"
  isPublished Boolean @default(false)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  seoTitle  String?
  seoDescription String?
}

model Menu {
  id        String     @id @default(cuid())
  name      String     @unique
  location  String?    // e.g., "header", "footer"
  items     MenuItem[]
  createdAt DateTime   @default(now())
  updatedAt DateTime   @updatedAt
}

model MenuItem {
  id        String   @id @default(cuid())
  menuId    String
  menu      Menu     @relation(fields: [menuId], references: [id], onDelete: Cascade)
  label     String
  url       String
  order     Int
  parentId  String?
  parent    MenuItem? @relation("SubMenuItems", fields: [parentId], references: [id], onDelete: NoAction, onUpdate: NoAction)
  subItems  MenuItem[] @relation("SubMenuItems")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model ContactFormSubmission {
  id        String   @id @default(cuid())
  name      String
  email     String
  subject   String?
  message   String   @db.Text
  ipAddress String?
  createdAt DateTime @default(now())
}
```

## 7. ESEMPI CONCRETI

Di seguito sono presentati 5 esempi di richieste utente e l'output generato dall'algoritmo decisionale.

**Esempio 1:**
```
Input: "Voglio creare un negozio online per vendere scarpe con taglie e colori, e permettere ai clienti di lasciare recensioni."

Output:
{
  "primary": "ecommerce",
  "secondary": [],
  "modules": [
    "AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE",
    "ECOMMERCE/PRODUCTS", "ECOMMERCE/CART", "ECOMMERCE/CHECKOUT", "ECOMMERCE/ORDERS",
    "ECOMMERCE/PAYMENTS", "ECOMMERCE/ADMIN", "ECOMMERCE/INVENTORY", "ECOMMERCE/SHIPPING",
    "ECOMMERCE/DISCOUNTS", "ECOMMERCE/CUSTOMERS", "ECOMMERCE/PRODUCTS_VARIANTS",
    "ECOMMERCE/REVIEWS"
  ],
  "features": {
    "ECOMMERCE.PRODUCTS.variants": true,
    "ECOMMERCE.PRODUCTS.reviews": true,
    "ECOMMERCE.PRODUCTS.wishlist": false,
    "ECOMMERCE.CART.guest_cart": false,
    "ECOMMERCE.CHECKOUT.one_page_checkout": false,
    "ECOMMERCE.ORDERS.tracking": false,
    "ECOMMERCE.PAYMENTS.multiple_gateways": false,
    "ECOMMERCE.ADMIN.analytics": false
  },
  "routes": [
    "/", "/products", "/products/[slug]", "/categories", "/categories/[slug]",
    "/cart", "/checkout", "/checkout/success", "/account", "/account/orders",
    "/account/settings", "/admin", "/admin/products", "/admin/orders",
    "/admin/customers", "/login", "/register", "/forgot-password"
  ],
  "schema_models": [
    "User", "Account", "Session", "VerificationToken", "Product", "ProductVariant",
    "Category", "Cart", "CartItem", "Order", "OrderItem", "Payment",
    "ShippingAddress", "BillingAddress", "Review"
  ]
}
```

**Esempio 2:**
```
Input: "Vorrei una piattaforma social dove gli utenti possono creare profili, fare post, commentare e seguire altri utenti. Anche la messaggistica privata è importante."

Output:
{
  "primary": "social",
  "secondary": [],
  "modules": [
    "AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE",
    "SOCIAL/PROFILES", "SOCIAL/POSTS", "SOCIAL/COMMENTS", "SOCIAL/LIKES",
    "SOCIAL/FOLLOW", "SOCIAL/FEED", "SOCIAL/MESSAGING", "SOCIAL/NOTIFICATIONS",
    "SOCIAL/MODERATION", "SOCIAL/PRIVACY"
  ],
  "features": {
    "SOCIAL.PROFILES.custom_fields": false,
    "SOCIAL.PROFILES.privacy_settings": true,
    "SOCIAL.POSTS.media_upload": true,
    "SOCIAL.POSTS.rich_text_editor": false,
    "SOCIAL.COMMENTS.nested_comments": true,
    "SOCIAL.COMMENTS.moderation": true,
    "SOCIAL.LIKES.reactions": false,
    "SOCIAL.MESSAGING.direct_messages": true,
    "SOCIAL.MESSAGING.group_chats": false,
    "SOCIAL.GROUPS.private_groups": false
  },
  "routes": [
    "/", "/feed", "/profile/[username]", "/profile/[username]/posts",
    "/profile/[username]/followers", "/profile/[username]/following",
    "/posts/[id]", "/posts/create", "/messages", "/messages/[conversation_id]",
    "/notifications", "/search", "/settings", "/settings/privacy",
    "/login", "/register", "/forgot-password"
  ],
  "schema_models": [
    "User", "Account", "Session", "VerificationToken", "Profile", "Post",
    "Comment", "Like", "Follow", "Message", "Conversation", "Notification"
  ]
}
```

**Esempio 3:**
```
Input: "Ho bisogno di un sito web aziendale con una sezione blog per pubblicare articoli e notizie. Deve avere un modulo di contatto e una pagina 'Chi Siamo'."

Output:
{
  "primary": "website",
  "secondary": ["blog"],
  "modules": [
    "AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE",
    "WEBSITE/PAGES", "WEBSITE/CONTACT", "WEBSITE/FORMS", "WEBSITE/MENU",
    "WEBSITE/LAYOUT", "WEBSITE/SEO", "WEBSITE/ANALYTICS", "WEBSITE/LEGAL",
    "WEBSITE/SEARCH", "BLOG/ARTICLES", "BLOG/CATEGORIES", "BLOG/TAGS",
    "BLOG/AUTHORS", "BLOG/ADMIN_EDITOR", "BLOG/SEO", "BLOG/SEARCH"
  ],
  "features": {
    "WEBSITE.PAGES.custom_layouts": false,
    "WEBSITE.FORMS.custom_form_builder": true,
    "WEBSITE.FORMS.email_notifications": true,
    "WEBSITE.MENU.multi_level_menu": false,
    "WEBSITE.SEO.global_settings": true,
    "BLOG.ARTICLES.rich_text_editor": true,
    "BLOG.ARTICLES.scheduling": false,
    "BLOG.AUTHORS.author_profiles": true,
    "BLOG.COMMENTS.moderation": false,
    "BLOG.SEO.meta_tags": true
  },
  "routes": [
    "/", "/about", "/contact", "/services", "/privacy-policy",
    "/terms-of-service", "/blog", "/blog/[slug]", "/blog/category/[slug]",
    "/blog/search", "/admin", "/admin/pages", "/admin/articles",
    "/admin/menu", "/admin/forms", "/login", "/register"
  ],
  "schema_models": [
    "User", "Account", "Session", "VerificationToken", "Page", "Menu",
    "MenuItem", "ContactFormSubmission", "Article", "ArticleCategory", "Tag"
  ]
}
```

**Esempio 4:**
```
Input: "Voglio un blog per condividere le mie ricette, con la possibilità per gli utenti di commentare e iscriversi a una newsletter. Vorrei anche una sezione per i miei prodotti di cucina."

Output:
{
  "primary": "blog",
  "secondary": ["ecommerce"],
  "modules": [
    "AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE",
    "BLOG/ARTICLES", "BLOG/CATEGORIES", "BLOG/TAGS", "BLOG/AUTHORS",
    "BLOG/COMMENTS", "BLOG/ADMIN_EDITOR", "BLOG/SEO", "BLOG/NEWSLETTER",
    "BLOG/SEARCH", "ECOMMERCE/PRODUCTS", "ECOMMERCE/CART", "ECOMMERCE/CHECKOUT",
    "ECOMMERCE/PAYMENTS"
  ],
  "features": {
    "BLOG.ARTICLES.rich_text_editor": true,
    "BLOG.COMMENTS.moderation": true,
    "BLOG.NEWSLETTER.subscription_form": true,
    "ECOMMERCE.PRODUCTS.variants": false,
    "ECOMMERCE.PRODUCTS.reviews": false,
    "ECOMMERCE.CART.guest_cart": true
  },
  "routes": [
    "/", "/blog", "/blog/[slug]", "/blog/category/[slug]", "/newsletter",
    "/products", "/products/[slug]", "/cart", "/checkout", "/account",
    "/admin/articles", "/admin/comments", "/admin/newsletter", "/admin/products",
    "/login", "/register"
  ],
  "schema_models": [
    "User", "Account", "Session", "VerificationToken", "Article", "ArticleCategory",
    "Tag", "Comment", "NewsletterSubscription", "Product", "Cart", "CartItem",
    "Order", "OrderItem", "Payment", "ShippingAddress", "BillingAddress"
  ]
}
```

**Esempio 5:**
```
Input: "Crea un sito per la mia associazione, con pagine informative, un calendario eventi e la possibilità per i membri di interagire in un forum privato."

Output:
{
  "primary": "website",
  "secondary": ["social"],
  "modules": [
    "AUTH", "DB", "UI", "API", "ERROR", "NOTIFICATIONS_CORE",
    "WEBSITE/PAGES", "WEBSITE/CONTACT", "WEBSITE/MENU", "WEBSITE/SEO",
    "WEBSITE/ANALYTICS", "SOCIAL/GROUPS", "SOCIAL/FORUM", "SOCIAL/EVENTS",
    "SOCIAL/PROFILES", "SOCIAL/MESSAGING"
  ],
  "features": {
    "WEBSITE.PAGES.custom_layouts": true,
    "WEBSITE.FORMS.custom_form_builder": true,
    "WEBSITE.MENU.multi_level_menu": true,
    "SOCIAL.GROUPS.private_groups": true,
    "SOCIAL.EVENTS.event_creation": true,
    "SOCIAL.EVENTS.rsvp": true,
    "SOCIAL.PROFILES.privacy_settings": true,
    "SOCIAL.MESSAGING.direct_messages": true
  },
  "routes": [
    "/", "/about", "/contact", "/services", "/events", "/events/[slug]",
    "/forum", "/forum/topic/[id]", "/profile/[username]", "/messages",
    "/admin/pages", "/admin/menu", "/admin/events", "/admin/groups",
    "/login", "/register"
  ],
  "schema_models": [
    "User", "Account", "Session", "VerificationToken", "Page", "Menu",
    "MenuItem", "ContactFormSubmission", "Group", "GroupMember", "Profile",
    "Post", "Comment", "Message", "Conversation", "Notification" // Post/Comment per forum
  ]
}
```

---
_Modello: gemini-2.5-flash (Google AI Studio) | Token: 22792_

8. APP ROUTER ROUTE GENERATION v2
Route Structure con app/ Directory
text
Copia
Scarica
app/
├── (marketing)/
│   ├── page.tsx
│   ├── pricing/
│   │   └── page.tsx
│   └── blog/
│       └── [slug]/
│           └── page.tsx
├── (app)/
│   ├── (dashboard)/
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   │   └── analytics/
│   │       └── page.tsx
│   └── (admin)/
│       └── admin/
│           └── users/
│               └── [id]/
│                   └── page.tsx
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── api/
│   └── webhooks/
│       └── stripe/
│           └── route.ts
└── layout.tsx
Route Groups Pattern
typescript
Copia
Scarica
// app/(marketing)/layout.tsx
export default function MarketingLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="marketing-layout">
      <header>Marketing Header</header>
      <main>{children}</main>
      <footer>Marketing Footer</footer>
    </div>
  );
}
Dynamic Routes Implementation
typescript
Copia
Scarica
// app/products/[category]/[productId]/page.tsx
interface ProductPageProps {
  params: Promise<{
    category: string;
    productId: string;
  }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

export default async function ProductPage({
  params,
  searchParams,
}: ProductPageProps) {
  const { category, productId } = await params;
  const { variant } = await searchParams;
  
  // Fetch product data
  const product = await getProduct(productId);
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>Category: {category}</p>
    </div>
  );
}
Parallel Routes Pattern
typescript
Copia
Scarica
// app/@modal/default.tsx
export default function DefaultModal() {
  return null;
}

// app/@modal/(.)photo/[id]/page.tsx
interface ModalPhotoPageProps {
  params: Promise<{ id: string }>;
}

export default async function ModalPhotoPage({ params }: ModalPhotoPageProps) {
  const { id } = await params;
  const photo = await getPhoto(id);
  
  return (
    <div className="modal-overlay">
      <div className="modal-content">
        <img src={photo.url} alt={photo.title} />
      </div>
    </div>
  );
}

// app/layout.tsx
export default function RootLayout({
  children,
  modal,
}: {
  children: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
        {children}
        {modal}
        <div id="modal-root" />
      </body>
    </html>
  );
}
Intercepting Routes Pattern
typescript
Copia
Scarica
// app/(app)/dashboard/(..)photo/[id]/page.tsx
export default async function InterceptedPhotoPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  // Return a simplified view for dashboard context
  return (
    <div className="dashboard-photo-preview">
      <PhotoPreview id={id} />
    </div>
  );
}
Tabella: 6 Tipi Progetto × Routes Generate
Tipo Progetto	Route Base	Route Dinamiche	Route Parallele	Route Intercettate
E-commerce	/products, /cart	products/[category]/[id]	@cart-sidebar	(.)product/[id]
Blog CMS	/blog, /admin	blog/[...slug]	@admin-sidebar	(..)login
SaaS Dashboard	/dashboard, /settings	dashboard/[section]	@analytics	(.)settings/[tab]
Marketplace	/listings, /messages	listings/[[...filters]]	@messages	(..)checkout
Portfolio	/work, /about	work/[projectId]	@contact	(.)project/[id]
Social Network	/feed, /profile	profile/[username]/[...tabs]	@notifications	(.)post/[id]
Codice Completo: Generazione Cartella App/ per E-commerce
typescript
Copia
Scarica
// app/layout.tsx
import type { Metadata } from 'next';
import { Inter } from 'next/font/google';
import './globals.css';

const inter = Inter({ subsets: ['latin'] });

export const metadata: Metadata = {
  title: 'NextCommerce - Modern E-commerce',
  description: 'Built with Next.js 14 App Router',
};

export default function RootLayout({
  children,
  cart,
  modal,
}: {
  children: React.ReactNode;
  cart: React.ReactNode;
  modal: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <header className="main-header">
          <nav>Navigation</nav>
          {cart}
        </header>
        <main>{children}</main>
        {modal}
        <footer>Footer</footer>
      </body>
    </html>
  );
}
typescript
Copia
Scarica
// app/(marketing)/page.tsx
import { Suspense } from 'react';
import FeaturedProducts from '@/components/FeaturedProducts';
import HeroSection from '@/components/HeroSection';

export default function HomePage() {
  return (
    <div>
      <HeroSection />
      <Suspense fallback={<div>Loading products...</div>}>
        <FeaturedProducts />
      </Suspense>
    </div>
  );
}
typescript
Copia
Scarica
// app/products/[category]/page.tsx
import { Metadata } from 'next';
import ProductGrid from '@/components/ProductGrid';
import CategoryFilter from '@/components/CategoryFilter';

interface CategoryPageProps {
  params: Promise<{ category: string }>;
  searchParams: Promise<{
    sort?: string;
    page?: string;
  }>;
}

export async function generateMetadata({
  params,
}: CategoryPageProps): Promise<Metadata> {
  const { category } = await params;
  return {
    title: `${category.charAt(0).toUpperCase() + category.slice(1)} Products`,
    description: `Browse our ${category} collection`,
  };
}

export default async function CategoryPage({
  params,
  searchParams,
}: CategoryPageProps) {
  const { category } = await params;
  const { sort = 'newest', page = '1' } = await searchParams;
  
  const products = await getProductsByCategory(category, {
    sort,
    page: parseInt(page),
  });
  
  return (
    <div className="category-page">
      <h1>{category} Products</h1>
      <CategoryFilter currentCategory={category} />
      <ProductGrid products={products} />
    </div>
  );
}
typescript
Copia
Scarica
// app/products/[category]/[productId]/page.tsx
import { Metadata } from 'next';
import ProductDetails from '@/components/ProductDetails';
import RelatedProducts from '@/components/RelatedProducts';
import AddToCartButton from '@/components/AddToCartButton';

interface ProductPageProps {
  params: Promise<{
    category: string;
    productId: string;
  }>;
}

export async function generateMetadata({
  params,
}: ProductPageProps): Promise<Metadata> {
  const { productId } = await params;
  const product = await getProductMetadata(productId);
  
  return {
    title: product.name,
    description: product.description,
    openGraph: {
      images: [product.image],
    },
  };
}

export default async function ProductPage({ params }: ProductPageProps) {
  const { category, productId } = await params;
  const [product, relatedProducts] = await Promise.all([
    getProduct(productId),
    getRelatedProducts(productId, category),
  ]);
  
  return (
    <div className="product-page">
      <ProductDetails product={product} />
      <AddToCartButton productId={productId} />
      <RelatedProducts products={relatedProducts} />
    </div>
  );
}
typescript
Copia
Scarica
// app/@cart/default.tsx
export default function DefaultCart() {
  return null;
}

// app/@cart/page.tsx
'use client';

import { useCart } from '@/contexts/CartContext';
import CartItem from '@/components/CartItem';

export default function CartSidebar() {
  const { items, isOpen, closeCart } = useCart();
  
  if (!isOpen) return null;
  
  return (
    <div className="cart-sidebar">
      <div className="cart-header">
        <h2>Your Cart</h2>
        <button onClick={closeCart}>×</button>
      </div>
      <div className="cart-items">
        {items.map((item) => (
          <CartItem key={item.id} item={item} />
        ))}
      </div>
      <div className="cart-footer">
        <button className="checkout-button">Checkout</button>
      </div>
    </div>
  );
}
typescript
Copia
Scarica
// app/@modal/(.)product/[id]/page.tsx
'use client';

import { useRouter } from 'next/navigation';
import QuickView from '@/components/QuickView';

interface QuickViewModalProps {
  params: Promise<{ id: string }>;
}

export default function QuickViewModal({ params }: QuickViewModalProps) {
  const router = useRouter();
  const { id } = React.use(params);
  
  const handleClose = () => {
    router.back();
  };
  
  return (
    <div className="modal-overlay" onClick={handleClose}>
      <div className="modal-content" onClick={(e) => e.stopPropagation()}>
        <button className="modal-close" onClick={handleClose}>
          ×
        </button>
        <QuickView productId={id} />
      </div>
    </div>
  );
}
typescript
Copia
Scarica
// app/cart/page.tsx
import { Metadata } from 'next';
import CartSummary from '@/components/CartSummary';
import CartItemsList from '@/components/CartItemsList';

export const metadata: Metadata = {
  title: 'Shopping Cart',
  robots: {
    index: false,
    follow: true,
  },
};

export default async function CartPage() {
  const cartItems = await getCartItems();
  
  return (
    <div className="cart-page">
      <h1>Shopping Cart</h1>
      <div className="cart-layout">
        <CartItemsList items={cartItems} />
        <CartSummary items={cartItems} />
      </div>
    </div>
  );
}
typescript
Copia
Scarica
// app/checkout/[[...step]]/page.tsx
import { redirect } from 'next/navigation';
import CheckoutForm from '@/components/CheckoutForm';
import CheckoutSummary from '@/components/CheckoutSummary';

interface CheckoutPageProps {
  params: Promise<{ step?: string[] }>;
}

export default async function CheckoutPage({ params }: CheckoutPageProps) {
  const { step } = await params;
  const currentStep = step?.[0] || 'shipping';
  
  // Validate step
  const validSteps = ['shipping', 'payment', 'review'];
  if (!validSteps.includes(currentStep)) {
    redirect('/checkout/shipping');
  }
  
  return (
    <div className="checkout-page">
      <div className="checkout-steps">
        {validSteps.map((s) => (
          <div
            key={s}
            className={`step ${s === currentStep ? 'active' : ''}`}
          >
            {s.charAt(0).toUpperCase() + s.slice(1)}
          </div>
        ))}
      </div>
      
      <div className="checkout-content">
        <CheckoutForm step={currentStep} />
        <CheckoutSummary />
      </div>
    </div>
  );
}
typescript
Copia
Scarica
// app/account/orders/[orderId]/page.tsx
import { notFound } from 'next/navigation';
import OrderDetails from '@/components/OrderDetails';

interface OrderPageProps {
  params: Promise<{ orderId: string }>;
}

export default async function OrderPage({ params }: OrderPageProps) {
  const { orderId } = await params;
  const order = await getOrder(orderId);
  
  if (!order) {
    notFound();
  }
  
  return (
    <div className="order-page">
      <OrderDetails order={order} />
    </div>
  );
}
typescript
Copia
Scarica
// app/search/[[...query]]/page.tsx
import SearchResults from '@/components/SearchResults';
import SearchFilters from '@/components/SearchFilters';

interface SearchPageProps {
  searchParams: Promise<{
    q?: string;
    category?: string;
    price_min?: string;
    price_max?: string;
  }>;
}

export default async function SearchPage({
  searchParams,
}: SearchPageProps) {
  const params = await searchParams;
  const query = params.q || '';
  
  const results = await searchProducts({
    query,
    category: params.category,
    priceMin: params.price_min,
    priceMax: params.price_max,
  });
  
  return (
    <div className="search-page">
      <SearchFilters currentFilters={params} />
      <SearchResults query={query} results={results} />
    </div>
  );
}
typescript
Copia
Scarica
// app/(auth)/login/page.tsx
'use client';

import { useRouter } from 'next/navigation';
import { useAuth } from '@/contexts/AuthContext';
import LoginForm from '@/components/LoginForm';

export default function LoginPage() {
  const router = useRouter();
  const { login, isAuthenticated } = useAuth();
  
  if (isAuthenticated) {
    router.push('/account');
    return null;
  }
  
  const handleSubmit = async (credentials: {
    email: string;
    password: string;
  }) => {
    await login(credentials);
    router.push('/account');
  };
  
  return (
    <div className="auth-page">
      <LoginForm onSubmit={handleSubmit} />
    </div>
  );
}
typescript
Copia
Scarica
// app/api/webhooks/stripe/route.ts
import { headers } from 'next/headers';
import { NextRequest, NextResponse } from 'next/server';
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!);
const webhookSecret = process.env.STRIPE_WEBHOOK_SECRET!;

export async function POST(request: NextRequest) {
  const body = await request.text();
  const signature = (await headers()).get('stripe-signature')!;
  
  try {
    const event = stripe.webhooks.constructEvent(
      body,
      signature,
      webhookSecret
    );
    
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutCompleted(event.data.object);
        break;
      case 'payment_intent.succeeded':
        await handlePaymentSucceeded(event.data.object);
        break;
    }
    
    return NextResponse.json({ received: true });
  } catch (err) {
    console.error('Webhook error:', err);
    return NextResponse.json(
      { error: 'Webhook handler failed' },
      { status: 400 }
    );
  }
}
typescript
Copia
Scarica
// app/globals.css
:root {
  --primary-color: #0070f3;
  --secondary-color: #1a1a1a;
  /* Additional CSS variables */
}

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: var(--font-sans);
}
9. SERVER ACTIONS DECISION MATRIX
Quando Usare Server Actions vs Route Handlers

✅ Server Actions:

Mutazioni dati (create, update, delete)

Form submissions

Operazioni che richiedono revalidatePath/revalidateTag

Azioni che devono essere chiamate da componenti client

❌ Route Handlers:

API endpoints pubblici

Webhooks

Integrazioni con servizi esterni

Quando serve controllo preciso su metodi HTTP

Quando si serve JSON/XML puro

Pattern Decisionale
typescript
Copia
Scarica
// Tabella Decisionale: 10 Operazioni
const decisionMatrix = [
  {
    operation: 'Aggiungere prodotto al carrello',
    recommended: 'Server Action',
    reason: 'Mutazione stato, bisogno di revalidare UI',
    codePattern: 'useFormState + server action'
  },
  {
    operation: 'Fetch lista prodotti',
    recommended: 'Route Handler',
    reason: 'Solo lettura, può essere cacheable',
    codePattern: 'GET /api/products'
  },
  {
    operation: 'Processare pagamento Stripe',
    recommended: 'Route Handler',
    reason: 'Webhook, richiede verifica signature',
    codePattern: 'POST /api/webhooks/stripe'
  },
  {
    operation: 'Aggiornare profilo utente',
    recommended: 'Server Action',
    reason: 'Mutazione, bisogno di revalidatePath',
    codePattern: '<form action={updateProfile}>'
  },
  {
    operation: 'Upload immagine',
    recommended: 'Server Action',
    reason: 'Mutazione, processamento lato server',
    codePattern: 'useFormStatus + server action'
  },
  {
    operation: 'Fetch dati per dashboard',
    recommended: 'Direct fetch in Server Component',
    reason: 'Dati non sensibili, caching automatico',
    codePattern: 'fetch() in async component'
  },
  {
    operation: 'Export dati CSV',
    recommended: 'Route Handler',
    reason: 'Download file, content-type specifico',
    codePattern: 'GET /api/export?format=csv'
  },
  {
    operation: 'Login utente',
    recommended: 'Server Action',
    reason: 'Mutazione sessione, redirect dopo successo',
    codePattern: 'signIn action + redirect'
  },
  {
    operation: 'Ricerca prodotti',
    recommended: 'Route Handler',
    reason: 'API pubblica, filtro complesso',
    codePattern: 'GET /api/search?q=...'
  },
  {
    operation: 'Inviare email notifica',
    recommended: 'Server Action',
    reason: 'Azione server-side, non serve risposta JSON',
    codePattern: 'sendEmail server action'
  }
];
Codice: 3 Server Actions Tipo
typescript
Copia
Scarica
// app/actions/product-actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';
import { redirect } from 'next/navigation';
import { z } from 'zod';
import { db } from '@/lib/db';

const productSchema = z.object({
  name: z.string().min(1),
  description: z.string(),
  price: z.number().positive(),
  categoryId: z.string(),
});

// 1. Create Product Action
export async function createProduct(
  prevState: { errors?: string[]; success?: boolean },
  formData: FormData
) {
  try {
    const validatedFields = productSchema.parse({
      name: formData.get('name'),
      description: formData.get('description'),
      price: parseFloat(formData.get('price') as string),
      categoryId: formData.get('categoryId'),
    });

    await db.product.create({
      data: validatedFields,
    });

    // Revalidate relevant paths and tags
    revalidatePath('/products');
    revalidatePath('/admin/products');
    revalidateTag('products');
    
    return { success: true };
  } catch (error) {
    console.error('Create product error:', error);
    return {
      errors: ['Failed to create product'],
    };
  }
}

// 2. Update Product Action
export async function updateProduct(
  productId: string,
  updates: Partial<{
    name: string;
    description: string;
    price: number;
    stock: number;
  }>
) {
  try {
    const product = await db.product.update({
      where: { id: productId },
      data: updates,
    });

    // Revalidate product page and listings
    revalidatePath(`/products/${product.category}/${product.id}`);
    revalidatePath('/products');
    revalidateTag(`product-${productId}`);
    
    return { success: true, product };
  } catch (error) {
    console.error('Update product error:', error);
    return { error: 'Failed to update product' };
  }
}

// 3. Delete Product Action con protezione
export async function deleteProduct(
  productId: string,
  confirm: boolean = false
) {
  // Safety check
  if (!confirm) {
    throw new Error('Confirmation required');
  }

  try {
    // Check if product exists
    const product = await db.product.findUnique({
      where: { id: productId },
    });

    if (!product) {
      throw new Error('Product not found');
    }

    // Soft delete implementation
    await db.product.update({
      where: { id: productId },
      data: {
        deletedAt: new Date(),
        status: 'DELETED',
      },
    });

    // Revalidate all product-related pages
    revalidatePath('/products', 'layout');
    revalidatePath('/admin/products');
    revalidateTag('products');
    revalidateTag(`product-${productId}`);

    // Redirect if on product page
    redirect('/products');
  } catch (error) {
    console.error('Delete product error:', error);
    throw error;
  }
}

// 4. Optimistic Update con useOptimistic
export async function updateProductStock(
  productId: string,
  quantity: number,
  operation: 'increment' | 'decrement'
) {
  try {
    const product = await db.product.findUnique({
      where: { id: productId },
      select: { stock: true },
    });

    if (!product) {
      throw new Error('Product not found');
    }

    const newStock = operation === 'increment' 
      ? product.stock + quantity 
      : product.stock - quantity;

    if (newStock < 0) {
      throw new Error('Insufficient stock');
    }

    await db.product.update({
      where: { id: productId },
      data: { stock: newStock },
    });

    revalidateTag(`product-${productId}`);
    revalidatePath(`/products/[id]`, 'page');
    
    return { success: true, newStock };
  } catch (error) {
    console.error('Update stock error:', error);
    return { error: error instanceof Error ? error.message : 'Failed' };
  }
}
typescript
Copia
Scarica
// app/components/ProductForm.tsx
'use client';

import { useFormState, useFormStatus } from 'react-dom';
import { createProduct } from '@/app/actions/product-actions';

const initialState = {
  errors: [],
  success: false,
};

export function ProductForm({ categories }: { categories: Category[] }) {
  const [state, formAction] = useFormState(createProduct, initialState);
  const { pending } = useFormStatus();

  return (
    <form action={formAction} className="product-form">
      <div>
        <label htmlFor="name">Product Name</label>
        <input
          type="text"
          id="name"
          name="name"
          required
          disabled={pending}
        />
      </div>
      
      <div>
        <label htmlFor="categoryId">Category</label>
        <select id="categoryId" name="categoryId" required disabled={pending}>
          {categories.map((category) => (
            <option key={category.id} value={category.id}>
              {category.name}
            </option>
          ))}
        </select>
      </div>
      
      <div>
        <label htmlFor="price">Price</label>
        <input
          type="number"
          id="price"
          name="price"
          step="0.01"
          required
          disabled={pending}
        />
      </div>
      
      <div>
        <label htmlFor="description">Description</label>
        <textarea
          id="description"
          name="description"
          rows={4}
          disabled={pending}
        />
      </div>
      
      <button type="submit" disabled={pending}>
        {pending ? 'Creating...' : 'Create Product'}
      </button>
      
      {state.errors?.map((error, index) => (
        <p key={index} className="error">{error}</p>
      ))}
      
      {state.success && (
        <p className="success">Product created successfully!</p>
      )}
    </form>
  );
}
typescript
Copia
Scarica
// app/components/StockManager.tsx
'use client';

import { useOptimistic, startTransition } from 'react';
import { updateProductStock } from '@/app/actions/product-actions';

interface StockManagerProps {
  productId: string;
  initialStock: number;
}

export function StockManager({ productId, initialStock }: StockManagerProps) {
  const [optimisticStock, setOptimisticStock] = useOptimistic(
    initialStock,
    (state, newStock: number) => newStock
  );

  const handleStockUpdate = async (operation: 'increment' | 'decrement') => {
    startTransition(async () => {
      const newStock = operation === 'increment' 
        ? optimisticStock + 1 
        : optimisticStock - 1;
      
      setOptimisticStock(newStock);
      
      const result = await updateProductStock(
        productId,
        1,
        operation
      );
      
      if (result.error) {
        // Revert optimistic update on error
        setOptimisticStock(initialStock);
      }
    });
  };

  return (
    <div className="stock-manager">
      <p>Stock: {optimisticStock}</p>
      <div className="stock-buttons">
        <button
          onClick={() => handleStockUpdate('decrement')}
          disabled={optimisticStock <= 0}
        >
          -
        </button>
        <button onClick={() => handleStockUpdate('increment')}>
          +
        </button>
      </div>
    </div>
  );
}
10. METADATA GENERATION STRATEGY
Pattern per generateMetadata
typescript
Copia
Scarica
// app/products/[category]/[productId]/page.tsx - Metadata
import { Metadata, ResolvingMetadata } from 'next';

interface ProductPageProps {
  params: Promise<{ productId: string; category: string }>;
  searchParams: Promise<{ [key: string]: string | string[] | undefined }>;
}

export async function generateMetadata(
  { params, searchParams }: ProductPageProps,
  parent: ResolvingMetadata
): Promise<Metadata> {
  const { productId, category } = await params;
  const product = await getProductForMetadata(productId);
  
  const previousImages = (await parent).openGraph?.images || [];
  
  return {
    title: `${product.name} | ${product.brand}`,
    description: product.metaDescription || product.description.slice(0, 160),
    keywords: product.tags,
    
    openGraph: {
      title: product.name,
      description: product.description.slice(0, 160),
      images: [
        {
          url: product.images[0],
          width: 1200,
          height: 630,
          alt: product.name,
        },
        ...previousImages,
      ],
      type: 'product',
      price: {
        amount: product.price.toString(),
        currency: 'USD',
      },
      availability: product.stock > 0 ? 'in stock' : 'out of stock',
    },
    
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description.slice(0, 160),
      images: [product.images[0]],
    },
    
    alternates: {
      canonical: `/products/${category}/${productId}`,
      languages: {
        'en-US': `/en-US/products/${category}/${productId}`,
        'it-IT': `/it-IT/products/${category}/${productId}`,
      },
    },
    
    robots: {
      index: true,
      follow: true,
      googleBot: {
        index: true,
        follow: true,
        'max-video-preview': -1,
        'max-image-preview': 'large',
        'max-snippet': -1,
      },
    },
    
    verification: {
      google: process.env.GOOGLE_VERIFICATION_CODE,
    },
  };
}
Dynamic Metadata Factory Pattern
typescript
Copia
Scarica
// lib/metadata/factories.ts
import { Metadata } from 'next';

export interface MetadataFactoryConfig {
  type: 'product' | 'category' | 'blog' | 'page';
  data: any;
  parentMetadata?: Metadata;
}

export async function generateDynamicMetadata(
  config: MetadataFactoryConfig
): Promise<Metadata> {
  const baseMetadata: Metadata = {
    metadataBase: new URL(process.env.NEXT_PUBLIC_APP_URL!),
    generator: 'Next.js',
    applicationName: 'NextCommerce',
    referrer: 'origin-when-cross-origin',
  };

  switch (config.type) {
    case 'product':
      return generateProductMetadata(config.data, config.parentMetadata);
    case 'category':
      return generateCategoryMetadata(config.data, config.parentMetadata);
    case 'blog':
      return generateBlogMetadata(config.data, config.parentMetadata);
    default:
      return baseMetadata;
  }
}

async function generateProductMetadata(
  product: Product,
  parentMetadata?: Metadata
): Promise<Metadata> {
  const parentImages = parentMetadata?.openGraph?.images || [];
  
  return {
    title: `${product.name} | ${product.brand}`,
    description: product.metaDescription || truncate(product.description, 160),
    
    openGraph: {
      title: product.name,
      description: product.description,
      images: [
        {
          url: product.primaryImage,
          width: 1200,
          height: 630,
          alt: product.name,
        },
        ...product.images.map((img, index) => ({
          url: img,
          width: 1200,
          height: 630,
          alt: `${product.name} - Image ${index + 1}`,
        })),
        ...parentImages,
      ],
      type: 'product',
      price: {
        amount: product.price.toString(),
        currency: product.currency,
      },
      availability: product.inStock ? 'in stock' : 'out of stock',
      tags: product.tags,
    },
    
    twitter: {
      card: 'summary_large_image',
      title: product.name,
      description: product.description,
      images: [product.primaryImage],
    },
    
    alternates: {
      canonical: `/products/${product.categorySlug}/${product.slug}`,
    },
    
    robots: {
      index: !product.isDraft,
      follow: true,
    },
  };
}

async function generateCategoryMetadata(
  category: Category,
  parentMetadata?: Metadata
): Promise<Metadata> {
  return {
    title: `${category.name} Products`,
    description: `Browse our collection of ${category.name.toLowerCase()} products.`,
    
    openGraph: {
      title: category.name,
      description: category.description,
      images: category.image ? [
        {
          url: category.image,
          width: 1200,
          height: 630,
          alt: category.name,
        },
      ] : undefined,
      type: 'website',
    },
    
    alternates: {
      canonical: `/categories/${category.slug}`,
    },
  };
}
Open Graph Images Strategy
typescript
Copia
Scarica
// app/og/route.tsx
import { ImageResponse } from 'next/og';
import { NextRequest } from 'next/server';

export const runtime = 'edge';

export async function GET(request: NextRequest) {
  try {
    const { searchParams } = new URL(request.url);
    
    // Get dynamic data from query params
    const title = searchParams.get('title') || 'Default Title';
    const description = searchParams.get('description') || 'Default Description';
    const type = searchParams.get('type') || 'website';
    
    // Choose font
    const fontData = await fetch(
      new URL('@/assets/Inter-Bold.ttf', import.meta.url)
    ).then((res) => res.arrayBuffer());
    
    return new ImageResponse(
      (
        <div
          style={{
            height: '100%',
            width: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            backgroundColor: '#1a1a1a',
            color: 'white',
            padding: '60px',
          }}
        >
          <div
            style={{
              fontSize: 72,
              fontWeight: 'bold',
              marginBottom: 40,
              textAlign: 'center',
            }}
          >
            {title}
          </div>
          
          {description && (
            <div
              style={{
                fontSize: 36,
                maxWidth: '80%',
                textAlign: 'center',
                opacity: 0.8,
              }}
            >
              {description}
            </div>
          )}
          
          <div
            style={{
              position: 'absolute',
              bottom: 40,
              right: 40,
              fontSize: 24,
              opacity: 0.6,
            }}
          >
            {process.env.NEXT_PUBLIC_APP_NAME}
          </div>
        </div>
      ),
      {
        width: 1200,
        height: 630,
        fonts: [
          {
            name: 'Inter',
            data: fontData,
            style: 'normal',
            weight: 700,
          },
        ],
      }
    );
  } catch (error) {
    console.error('OG Image generation failed:', error);
    return new Response('Failed to generate OG image', { status: 500 });
  }
}

// app/og/product/[id]/route.tsx
import { ImageResponse } from 'next/og';
import { getProduct } from '@/lib/products';

export const runtime = 'edge';

export async function GET(
  request: Request,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const product = await getProduct(id);
  
  if (!product) {
    return new Response('Product not found', { status: 404 });
  }
  
  return new ImageResponse(
    (
      <div
        style={{
          display: 'flex',
          height: '100%',
          width: '100%',
          backgroundColor: 'white',
        }}
      >
        {/* Left side - Product image */}
        <div
          style={{
            display: 'flex',
            width: '50%',
            alignItems: 'center',
            justifyContent: 'center',
            padding: '40px',
          }}
        >
          <img
            src={product.images[0]}
            alt={product.name}
            style={{
              maxWidth: '100%',
              maxHeight: '100%',
              objectFit: 'contain',
            }}
          />
        </div>
        
        {/* Right side - Product info */}
        <div
          style={{
            display: 'flex',
            flexDirection: 'column',
            width: '50%',
            padding: '60px 40px',
            justifyContent: 'center',
            backgroundColor: '#f8f9fa',
          }}
        >
          <h1
            style={{
              fontSize: 48,
              fontWeight: 'bold',
              marginBottom: 20,
              color: '#1a1a1a',
            }}
          >
            {product.name}
          </h1>
          
          <p
            style={{
              fontSize: 28,
              color: '#666',
              marginBottom: 30,
            }}
          >
            {product.brand}
          </p>
          
          <div
            style={{
              fontSize: 36,
              fontWeight: 'bold',
              color: '#0070f3',
              marginBottom: 30,
            }}
          >
            ${product.price}
          </div>
          
          {product.inStock && (
            <div
              style={{
                fontSize: 24,
                color: '#10b981',
                backgroundColor: '#d1fae5',
                padding: '8px 16px',
                borderRadius: '4px',
                alignSelf: 'flex-start',
              }}
            >
              In Stock
            </div>
          )}
        </div>
      </div>
    ),
    {
      width: 1200,
      height: 630,
    }