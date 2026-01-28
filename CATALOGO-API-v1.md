# ═══════════════════════════════════════════════════════════════════════════════
# CATALOGO API v1.0
# ═══════════════════════════════════════════════════════════════════════════════
# 
# SCOPO: Specifiche OpenAPI per tutte le operazioni del CATALOGO-REQUISITI-FUNZIONALI
# NOTA: Include SOLO API interne gratuite (no integrazioni esterne a pagamento)
# DIPENDENZE: 
#   - CATALOGO-REQUISITI-FUNZIONALI-v1.md
#   - CATALOGO-DATA-MODEL-v1.md
#
# ═══════════════════════════════════════════════════════════════════════════════

# ═══════════════════════════════════════════════════════════════════════════════
# SEZIONE 0: CONFIGURAZIONE OPENAPI BASE
# ═══════════════════════════════════════════════════════════════════════════════

openapi: "3.1.0"

info:
  title: "Platform API"
  version: "1.0.0"
  description: |
    API RESTful per piattaforma modulare.
    Moduli: IDENTITY, CONTENT, SOCIAL, MESSAGING, MARKETPLACE, BOOKING, COMMERCE
    
    NOTA: Questo catalogo include solo API gratuite (nessuna integrazione esterna a pagamento).
    Esclusi: pagamenti (Stripe), SMS (Twilio), email transazionali (SendGrid), etc.

servers:
  - url: "https://api.example.com/v1"
    description: "Production"
  - url: "https://api.staging.example.com/v1"
    description: "Staging"

# ═══════════════════════════════════════════════════════════════════════════════
# AUTENTICAZIONE
# ═══════════════════════════════════════════════════════════════════════════════

security_schemes:
  BearerAuth:
    type: http
    scheme: bearer
    bearerFormat: JWT
    description: "JWT token ottenuto da /auth/login"

  ApiKeyAuth:
    type: apiKey
    in: header
    name: X-API-Key
    description: "API Key per integrazioni server-to-server"

# ═══════════════════════════════════════════════════════════════════════════════
# COMPONENTI COMUNI
# ═══════════════════════════════════════════════════════════════════════════════

components:
  
  # ─────────────────────────────────────────────────────────────────────────────
  # HEADERS COMUNI
  # ─────────────────────────────────────────────────────────────────────────────
  headers:
    X-Request-Id:
      description: "ID univoco della richiesta per tracciamento"
      schema:
        type: string
        format: uuid
    
    X-RateLimit-Limit:
      description: "Numero massimo di richieste per finestra temporale"
      schema:
        type: integer
    
    X-RateLimit-Remaining:
      description: "Richieste rimanenti nella finestra corrente"
      schema:
        type: integer
    
    X-RateLimit-Reset:
      description: "Timestamp Unix di reset del rate limit"
      schema:
        type: integer

  # ─────────────────────────────────────────────────────────────────────────────
  # PARAMETRI COMUNI
  # ─────────────────────────────────────────────────────────────────────────────
  parameters:
    PathId:
      name: id
      in: path
      required: true
      schema:
        type: string
        format: uuid
      description: "ID univoco della risorsa"
    
    QueryCursor:
      name: cursor
      in: query
      required: false
      schema:
        type: string
      description: "Cursore per paginazione"
    
    QueryLimit:
      name: limit
      in: query
      required: false
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
      description: "Numero massimo di risultati"
    
    QuerySort:
      name: sort
      in: query
      required: false
      schema:
        type: string
        enum: [asc, desc]
        default: desc
      description: "Ordinamento risultati"

  # ─────────────────────────────────────────────────────────────────────────────
  # RISPOSTE COMUNI
  # ─────────────────────────────────────────────────────────────────────────────
  responses:
    Success:
      description: "Operazione completata con successo"
      content:
        application/json:
          schema:
            type: object
            properties:
              success:
                type: boolean
                example: true
              data:
                type: object
    
    Created:
      description: "Risorsa creata con successo"
      headers:
        Location:
          description: "URL della risorsa creata"
          schema:
            type: string
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/SuccessResponse"
    
    BadRequest:
      description: "Richiesta non valida"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "VALIDATION_ERROR"
              message: "Dati di input non validi"
              details:
                - field: "email"
                  message: "Email non valida"
    
    Unauthorized:
      description: "Autenticazione richiesta"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "AUTH_001"
              message: "Autenticazione richiesta"
    
    Forbidden:
      description: "Accesso negato"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "AUTH_002"
              message: "Non autorizzato"
    
    NotFound:
      description: "Risorsa non trovata"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "NOT_FOUND"
              message: "Risorsa non trovata"
    
    Conflict:
      description: "Conflitto (risorsa già esistente)"
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "CONFLICT"
              message: "Risorsa già esistente"
    
    TooManyRequests:
      description: "Rate limit superato"
      headers:
        Retry-After:
          description: "Secondi da attendere"
          schema:
            type: integer
      content:
        application/json:
          schema:
            $ref: "#/components/schemas/ErrorResponse"
          example:
            success: false
            error:
              code: "RATE_LIMIT"
              message: "Troppe richieste, riprova più tardi"

  # ─────────────────────────────────────────────────────────────────────────────
  # SCHEMI COMUNI
  # ─────────────────────────────────────────────────────────────────────────────
  schemas:
    
    SuccessResponse:
      type: object
      required: [success]
      properties:
        success:
          type: boolean
          example: true
        data:
          type: object
        meta:
          $ref: "#/components/schemas/PaginationMeta"
    
    ErrorResponse:
      type: object
      required: [success, error]
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          required: [code, message]
          properties:
            code:
              type: string
              description: "Codice errore univoco"
            message:
              type: string
              description: "Messaggio leggibile"
            details:
              type: array
              items:
                type: object
                properties:
                  field:
                    type: string
                  message:
                    type: string
    
    PaginationMeta:
      type: object
      properties:
        total_count:
          type: integer
          description: "Numero totale di risultati"
        has_more:
          type: boolean
          description: "Ci sono altri risultati"
        next_cursor:
          type: string
          description: "Cursore per pagina successiva"
          nullable: true
        prev_cursor:
          type: string
          description: "Cursore per pagina precedente"
          nullable: true
    
    Timestamp:
      type: string
      format: date-time
      example: "2024-01-15T10:30:00Z"
    
    UUID:
      type: string
      format: uuid
      example: "550e8400-e29b-41d4-a716-446655440000"



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO IDENTITY - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════
# ESCLUSI: SMS verification, 2FA via SMS, email transazionali a pagamento

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # REGISTRAZIONE E LOGIN
  # ─────────────────────────────────────────────────────────────────────────────

  /auth/register:
    post:
      tags: [Identity]
      summary: "Registrazione nuovo utente"
      operationId: registerUser
      security: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password, username]
              properties:
                email:
                  type: string
                  format: email
                  maxLength: 255
                password:
                  type: string
                  minLength: 8
                  maxLength: 128
                username:
                  type: string
                  minLength: 3
                  maxLength: 30
                  pattern: "^[a-zA-Z0-9_]+$"
                display_name:
                  type: string
                  maxLength: 100
      responses:
        "201":
          description: "Utente creato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      user_id: {type: string, format: uuid}
                      email: {type: string}
                      username: {type: string}
                      status: {type: string, enum: [pending, active]}
        "400": {$ref: "#/components/responses/BadRequest"}
        "409":
          description: "Email o username già esistente"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              examples:
                email_exists:
                  value: {success: false, error: {code: "AUTH_003", message: "Email già registrata"}}
                username_exists:
                  value: {success: false, error: {code: "AUTH_004", message: "Username già in uso"}}

  /auth/login:
    post:
      tags: [Identity]
      summary: "Login utente"
      operationId: loginUser
      security: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [email, password]
              properties:
                email:
                  type: string
                  format: email
                password:
                  type: string
                device_info:
                  type: object
                  properties:
                    device_type: {type: string, enum: [web, ios, android, desktop]}
                    os: {type: string}
                    browser: {type: string}
      responses:
        "200":
          description: "Login effettuato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean, example: true}
                  data:
                    type: object
                    properties:
                      access_token: {type: string}
                      refresh_token: {type: string}
                      token_type: {type: string, default: "Bearer"}
                      expires_in: {type: integer, description: "Secondi"}
                      user:
                        $ref: "#/components/schemas/UserPublic"
        "401":
          description: "Credenziali non valide"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/ErrorResponse"
              example:
                success: false
                error: {code: "AUTH_010", message: "Email o password non corretti"}
        "403":
          description: "Account non attivo"

  /auth/logout:
    post:
      tags: [Identity]
      summary: "Logout (invalida sessione)"
      operationId: logoutUser
      security: [{BearerAuth: []}]
      responses:
        "200":
          description: "Logout effettuato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean, example: true}
                  data:
                    type: object
                    properties:
                      message: {type: string, example: "Logout effettuato"}
        "401": {$ref: "#/components/responses/Unauthorized"}

  /auth/refresh:
    post:
      tags: [Identity]
      summary: "Rinnova access token"
      operationId: refreshToken
      security: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [refresh_token]
              properties:
                refresh_token: {type: string}
      responses:
        "200":
          description: "Token rinnovato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      access_token: {type: string}
                      expires_in: {type: integer}
        "401":
          description: "Refresh token non valido"

  # ─────────────────────────────────────────────────────────────────────────────
  # PROFILO UTENTE
  # ─────────────────────────────────────────────────────────────────────────────

  /users/me:
    get:
      tags: [Identity]
      summary: "Profilo utente corrente"
      operationId: getCurrentUser
      security: [{BearerAuth: []}]
      responses:
        "200":
          description: "Profilo utente"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/UserFull"
        "401": {$ref: "#/components/responses/Unauthorized"}
    
    patch:
      tags: [Identity]
      summary: "Aggiorna profilo"
      operationId: updateCurrentUser
      security: [{BearerAuth: []}]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                display_name: {type: string, maxLength: 100}
                bio: {type: string, maxLength: 500}
                website: {type: string, format: uri}
                location: {type: string, maxLength: 100}
                is_private: {type: boolean}
      responses:
        "200":
          description: "Profilo aggiornato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/UserFull"
        "400": {$ref: "#/components/responses/BadRequest"}
        "401": {$ref: "#/components/responses/Unauthorized"}

  /users/me/avatar:
    put:
      tags: [Identity]
      summary: "Carica avatar"
      operationId: uploadAvatar
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              required: [file]
              properties:
                file:
                  type: string
                  format: binary
                  description: "Immagine JPG/PNG, max 5MB"
      responses:
        "200":
          description: "Avatar caricato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      avatar_url: {type: string, format: uri}
        "400":
          description: "File non valido"
    
    delete:
      tags: [Identity]
      summary: "Rimuovi avatar"
      operationId: deleteAvatar
      security: [{BearerAuth: []}]
      responses:
        "200":
          description: "Avatar rimosso"

  /users/me/password:
    put:
      tags: [Identity]
      summary: "Cambia password"
      operationId: changePassword
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [current_password, new_password]
              properties:
                current_password: {type: string}
                new_password: {type: string, minLength: 8}
      responses:
        "200":
          description: "Password cambiata"
        "400":
          description: "Password corrente errata o nuova password non valida"

  /users/me/sessions:
    get:
      tags: [Identity]
      summary: "Lista sessioni attive"
      operationId: getSessions
      security: [{BearerAuth: []}]
      responses:
        "200":
          description: "Lista sessioni"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      sessions:
                        type: array
                        items:
                          $ref: "#/components/schemas/Session"
                      current_session_id: {type: string, format: uuid}

  /users/me/sessions/{session_id}:
    delete:
      tags: [Identity]
      summary: "Revoca sessione"
      operationId: revokeSession
      security: [{BearerAuth: []}]
      parameters:
        - name: session_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Sessione revocata"
        "404": {$ref: "#/components/responses/NotFound"}

  # ─────────────────────────────────────────────────────────────────────────────
  # PROFILI PUBBLICI
  # ─────────────────────────────────────────────────────────────────────────────

  /users/{user_id}:
    get:
      tags: [Identity]
      summary: "Profilo pubblico utente"
      operationId: getUserProfile
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Profilo utente"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/UserPublic"
        "404": {$ref: "#/components/responses/NotFound"}

  /users/username/{username}:
    get:
      tags: [Identity]
      summary: "Profilo per username"
      operationId: getUserByUsername
      parameters:
        - name: username
          in: path
          required: true
          schema: {type: string}
      responses:
        "200":
          description: "Profilo utente"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/UserPublic"
        "404": {$ref: "#/components/responses/NotFound"}

  /users/search:
    get:
      tags: [Identity]
      summary: "Cerca utenti"
      operationId: searchUsers
      parameters:
        - name: q
          in: query
          required: true
          schema: {type: string, minLength: 2}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Risultati ricerca"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/UserPublic"
                  meta:
                    $ref: "#/components/schemas/PaginationMeta"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS IDENTITY
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    UserPublic:
      type: object
      properties:
        id: {type: string, format: uuid}
        username: {type: string}
        display_name: {type: string}
        bio: {type: string, nullable: true}
        avatar_url: {type: string, format: uri, nullable: true}
        is_private: {type: boolean}
        is_verified: {type: boolean}
        followers_count: {type: integer}
        following_count: {type: integer}
        posts_count: {type: integer}
        created_at: {type: string, format: date-time}
        # Campi relazionali (se autenticato)
        is_following: {type: boolean, description: "Se segui questo utente"}
        is_followed_by: {type: boolean, description: "Se questo utente ti segue"}
        is_blocked: {type: boolean, description: "Se hai bloccato questo utente"}
    
    UserFull:
      allOf:
        - $ref: "#/components/schemas/UserPublic"
        - type: object
          properties:
            email: {type: string, format: email}
            email_verified: {type: boolean}
            phone: {type: string, nullable: true}
            phone_verified: {type: boolean}
            website: {type: string, nullable: true}
            location: {type: string, nullable: true}
            settings:
              $ref: "#/components/schemas/UserSettings"
            updated_at: {type: string, format: date-time}
    
    UserSettings:
      type: object
      properties:
        language: {type: string, default: "it"}
        timezone: {type: string, default: "Europe/Rome"}
        theme: {type: string, enum: [light, dark, system]}
        notifications:
          type: object
          properties:
            email: {type: boolean}
            push: {type: boolean}
            in_app: {type: boolean}
        privacy:
          type: object
          properties:
            show_online_status: {type: boolean}
            show_last_seen: {type: boolean}
    
    Session:
      type: object
      properties:
        id: {type: string, format: uuid}
        device_info:
          type: object
          properties:
            device_type: {type: string}
            os: {type: string}
            browser: {type: string}
        ip_address: {type: string}
        location:
          type: object
          properties:
            country: {type: string}
            city: {type: string}
        is_current: {type: boolean}
        last_activity_at: {type: string, format: date-time}
        created_at: {type: string, format: date-time}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO CONTENT - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # POSTS
  # ─────────────────────────────────────────────────────────────────────────────

  /posts:
    post:
      tags: [Content]
      summary: "Crea post"
      operationId: createPost
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/PostCreate"
      responses:
        "201":
          description: "Post creato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Post"
        "400": {$ref: "#/components/responses/BadRequest"}
        "401": {$ref: "#/components/responses/Unauthorized"}
    
    get:
      tags: [Content]
      summary: "Feed pubblico"
      operationId: getPublicFeed
      parameters:
        - name: hashtag
          in: query
          schema: {type: string}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista post"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Post"
                  meta:
                    $ref: "#/components/schemas/PaginationMeta"

  /posts/{post_id}:
    get:
      tags: [Content]
      summary: "Dettaglio post"
      operationId: getPost
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Post"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Post"
        "404": {$ref: "#/components/responses/NotFound"}
    
    patch:
      tags: [Content]
      summary: "Modifica post"
      operationId: updatePost
      security: [{BearerAuth: []}]
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                content: {type: string, maxLength: 5000}
                visibility: {type: string, enum: [public, followers, private]}
      responses:
        "200":
          description: "Post aggiornato"
        "403": {$ref: "#/components/responses/Forbidden"}
        "404": {$ref: "#/components/responses/NotFound"}
    
    delete:
      tags: [Content]
      summary: "Elimina post"
      operationId: deletePost
      security: [{BearerAuth: []}]
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Post eliminato"
        "403": {$ref: "#/components/responses/Forbidden"}

  /users/{user_id}/posts:
    get:
      tags: [Content]
      summary: "Post di un utente"
      operationId: getUserPosts
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista post"

  # ─────────────────────────────────────────────────────────────────────────────
  # COMMENTI
  # ─────────────────────────────────────────────────────────────────────────────

  /posts/{post_id}/comments:
    get:
      tags: [Content]
      summary: "Commenti di un post"
      operationId: getPostComments
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: sort
          in: query
          schema: {type: string, enum: [newest, oldest, top], default: newest}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista commenti"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Comment"
    
    post:
      tags: [Content]
      summary: "Aggiungi commento"
      operationId: createComment
      security: [{BearerAuth: []}]
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [content]
              properties:
                content: {type: string, minLength: 1, maxLength: 2000}
                parent_id: {type: string, format: uuid, description: "Per risposte"}
      responses:
        "201":
          description: "Commento creato"
        "404":
          description: "Post non trovato"

  /comments/{comment_id}:
    patch:
      tags: [Content]
      summary: "Modifica commento"
      operationId: updateComment
      security: [{BearerAuth: []}]
      parameters:
        - name: comment_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [content]
              properties:
                content: {type: string, maxLength: 2000}
      responses:
        "200":
          description: "Commento aggiornato"
        "403": {$ref: "#/components/responses/Forbidden"}
    
    delete:
      tags: [Content]
      summary: "Elimina commento"
      operationId: deleteComment
      security: [{BearerAuth: []}]
      parameters:
        - name: comment_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Commento eliminato"
        "403": {$ref: "#/components/responses/Forbidden"}

  # ─────────────────────────────────────────────────────────────────────────────
  # REAZIONI
  # ─────────────────────────────────────────────────────────────────────────────

  /posts/{post_id}/reactions:
    post:
      tags: [Content]
      summary: "Aggiungi reazione"
      operationId: addReaction
      security: [{BearerAuth: []}]
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [reaction_type]
              properties:
                reaction_type: {type: string, enum: [like, love, haha, wow, sad, angry]}
      responses:
        "201":
          description: "Reazione aggiunta"
        "409":
          description: "Reazione già presente"
    
    delete:
      tags: [Content]
      summary: "Rimuovi reazione"
      operationId: removeReaction
      security: [{BearerAuth: []}]
      parameters:
        - name: post_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Reazione rimossa"

  # ─────────────────────────────────────────────────────────────────────────────
  # MEDIA
  # ─────────────────────────────────────────────────────────────────────────────

  /media/upload:
    post:
      tags: [Content]
      summary: "Carica media"
      operationId: uploadMedia
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          multipart/form-data:
            schema:
              type: object
              required: [file]
              properties:
                file:
                  type: string
                  format: binary
                alt_text: {type: string, maxLength: 500}
      responses:
        "201":
          description: "Media caricato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/MediaItem"
        "400":
          description: "File non valido"
        "413":
          description: "File troppo grande"

  # ─────────────────────────────────────────────────────────────────────────────
  # HASHTAG
  # ─────────────────────────────────────────────────────────────────────────────

  /hashtags/trending:
    get:
      tags: [Content]
      summary: "Hashtag trending"
      operationId: getTrendingHashtags
      parameters:
        - name: limit
          in: query
          schema: {type: integer, minimum: 1, maximum: 50, default: 10}
      responses:
        "200":
          description: "Lista hashtag"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      type: object
                      properties:
                        hashtag: {type: string}
                        posts_count: {type: integer}
                        trending_score: {type: number}

  /hashtags/{hashtag}/posts:
    get:
      tags: [Content]
      summary: "Post per hashtag"
      operationId: getHashtagPosts
      parameters:
        - name: hashtag
          in: path
          required: true
          schema: {type: string}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista post"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS CONTENT
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    PostCreate:
      type: object
      required: [post_type]
      properties:
        post_type: {type: string, enum: [text, image, video, link, poll]}
        content: {type: string, maxLength: 5000}
        media_ids:
          type: array
          items: {type: string, format: uuid}
          maxItems: 10
        visibility: {type: string, enum: [public, followers, private], default: public}
        location:
          type: object
          properties:
            name: {type: string}
            lat: {type: number}
            lng: {type: number}
    
    Post:
      type: object
      properties:
        id: {type: string, format: uuid}
        author:
          $ref: "#/components/schemas/UserPublic"
        post_type: {type: string}
        content: {type: string}
        media:
          type: array
          items:
            $ref: "#/components/schemas/MediaItem"
        hashtags:
          type: array
          items: {type: string}
        visibility: {type: string}
        comments_count: {type: integer}
        reactions_count: {type: integer}
        reactions_summary:
          type: object
          description: "{like: 10, love: 5}"
        user_reaction: {type: string, nullable: true}
        is_edited: {type: boolean}
        published_at: {type: string, format: date-time}
        created_at: {type: string, format: date-time}
    
    Comment:
      type: object
      properties:
        id: {type: string, format: uuid}
        post_id: {type: string, format: uuid}
        author:
          $ref: "#/components/schemas/UserPublic"
        parent_id: {type: string, format: uuid, nullable: true}
        content: {type: string}
        reactions_count: {type: integer}
        replies_count: {type: integer}
        is_edited: {type: boolean}
        created_at: {type: string, format: date-time}
    
    MediaItem:
      type: object
      properties:
        id: {type: string, format: uuid}
        file_type: {type: string, enum: [image, video, audio, document]}
        mime_type: {type: string}
        url: {type: string, format: uri}
        thumbnail_url: {type: string, format: uri}
        dimensions:
          type: object
          properties:
            width: {type: integer}
            height: {type: integer}
        duration_seconds: {type: integer, nullable: true}
        file_size_bytes: {type: integer}
        alt_text: {type: string}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO SOCIAL - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # FOLLOW
  # ─────────────────────────────────────────────────────────────────────────────

  /users/{user_id}/follow:
    post:
      tags: [Social]
      summary: "Segui utente"
      operationId: followUser
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Follow creato"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      status: {type: string, enum: [active, pending]}
                      message: {type: string}
        "400":
          description: "Non puoi seguire te stesso"
        "409":
          description: "Già segui questo utente"
    
    delete:
      tags: [Social]
      summary: "Smetti di seguire"
      operationId: unfollowUser
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Unfollow effettuato"
        "404":
          description: "Non segui questo utente"

  /users/{user_id}/followers:
    get:
      tags: [Social]
      summary: "Lista followers"
      operationId: getUserFollowers
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista followers"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/UserPublic"
        "403":
          description: "Profilo privato"

  /users/{user_id}/following:
    get:
      tags: [Social]
      summary: "Lista following"
      operationId: getUserFollowing
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista following"
        "403":
          description: "Profilo privato"

  /users/me/follow-requests:
    get:
      tags: [Social]
      summary: "Richieste di follow ricevute"
      operationId: getFollowRequests
      security: [{BearerAuth: []}]
      parameters:
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista richieste"

  /users/me/follow-requests/{user_id}:
    post:
      tags: [Social]
      summary: "Accetta richiesta follow"
      operationId: acceptFollowRequest
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Richiesta accettata"
        "404":
          description: "Richiesta non trovata"
    
    delete:
      tags: [Social]
      summary: "Rifiuta richiesta follow"
      operationId: rejectFollowRequest
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Richiesta rifiutata"

  # ─────────────────────────────────────────────────────────────────────────────
  # BLOCK
  # ─────────────────────────────────────────────────────────────────────────────

  /users/{user_id}/block:
    post:
      tags: [Social]
      summary: "Blocca utente"
      operationId: blockUser
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                reason: {type: string, maxLength: 500}
      responses:
        "200":
          description: "Utente bloccato"
        "400":
          description: "Non puoi bloccare te stesso"
        "409":
          description: "Utente già bloccato"
    
    delete:
      tags: [Social]
      summary: "Sblocca utente"
      operationId: unblockUser
      security: [{BearerAuth: []}]
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Utente sbloccato"

  /users/me/blocked:
    get:
      tags: [Social]
      summary: "Lista utenti bloccati"
      operationId: getBlockedUsers
      security: [{BearerAuth: []}]
      parameters:
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista bloccati"

  # ─────────────────────────────────────────────────────────────────────────────
  # FEED
  # ─────────────────────────────────────────────────────────────────────────────

  /feed:
    get:
      tags: [Social]
      summary: "Feed personalizzato"
      operationId: getFeed
      security: [{BearerAuth: []}]
      parameters:
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Feed utente"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/FeedItem"
                  meta:
                    $ref: "#/components/schemas/PaginationMeta"

  /feed/refresh:
    post:
      tags: [Social]
      summary: "Controlla nuovi post"
      operationId: refreshFeed
      security: [{BearerAuth: []}]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                since: {type: string, format: date-time}
      responses:
        "200":
          description: "Conteggio nuovi post"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      new_posts_count: {type: integer}

  # ─────────────────────────────────────────────────────────────────────────────
  # STATISTICHE E SUGGERIMENTI
  # ─────────────────────────────────────────────────────────────────────────────

  /users/{user_id}/stats:
    get:
      tags: [Social]
      summary: "Statistiche utente"
      operationId: getUserStats
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Statistiche"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/UserStats"

  /users/suggestions:
    get:
      tags: [Social]
      summary: "Suggerimenti utenti"
      operationId: getSuggestedUsers
      security: [{BearerAuth: []}]
      parameters:
        - name: limit
          in: query
          schema: {type: integer, minimum: 1, maximum: 20, default: 10}
      responses:
        "200":
          description: "Suggerimenti"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS SOCIAL
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    FeedItem:
      type: object
      properties:
        id: {type: string, format: uuid}
        action_type: {type: string, enum: [post, repost, like]}
        action_user:
          $ref: "#/components/schemas/UserPublic"
        post:
          $ref: "#/components/schemas/Post"
        relevance_score: {type: number}
        is_seen: {type: boolean}
        created_at: {type: string, format: date-time}
    
    UserStats:
      type: object
      properties:
        user_id: {type: string, format: uuid}
        followers_count: {type: integer}
        following_count: {type: integer}
        posts_count: {type: integer}
        media_count: {type: integer}
        likes_received_count: {type: integer}
        comments_received_count: {type: integer}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO MESSAGING - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════
# ESCLUSI: Push notifications (FCM/APNs) - incluse solo notifiche in-app

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # CONVERSAZIONI
  # ─────────────────────────────────────────────────────────────────────────────

  /conversations:
    get:
      tags: [Messaging]
      summary: "Lista conversazioni"
      operationId: getConversations
      security: [{BearerAuth: []}]
      parameters:
        - name: filter
          in: query
          schema: {type: string, enum: [all, unread, archived], default: all}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista conversazioni"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Conversation"
                  meta:
                    allOf:
                      - $ref: "#/components/schemas/PaginationMeta"
                      - type: object
                        properties:
                          unread_total: {type: integer}
    
    post:
      tags: [Messaging]
      summary: "Crea conversazione"
      operationId: createConversation
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [participant_ids]
              properties:
                participant_ids:
                  type: array
                  items: {type: string, format: uuid}
                  minItems: 1
                  maxItems: 50
                name: {type: string, maxLength: 100}
                initial_message: {type: string, maxLength: 5000}
      responses:
        "201":
          description: "Conversazione creata"
        "200":
          description: "Conversazione esistente (per DM)"
        "403":
          description: "Utente bloccato"

  /conversations/{conversation_id}:
    get:
      tags: [Messaging]
      summary: "Dettaglio conversazione"
      operationId: getConversation
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Conversazione"
        "403":
          description: "Non partecipante"
        "404": {$ref: "#/components/responses/NotFound"}
    
    patch:
      tags: [Messaging]
      summary: "Aggiorna conversazione"
      operationId: updateConversation
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                name: {type: string, maxLength: 100}
                is_muted: {type: boolean}
                is_archived: {type: boolean}
      responses:
        "200":
          description: "Aggiornata"

  /conversations/{conversation_id}/leave:
    post:
      tags: [Messaging]
      summary: "Abbandona gruppo"
      operationId: leaveConversation
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Uscito dal gruppo"
        "400":
          description: "Non puoi lasciare una conversazione diretta"

  # ─────────────────────────────────────────────────────────────────────────────
  # MESSAGGI
  # ─────────────────────────────────────────────────────────────────────────────

  /conversations/{conversation_id}/messages:
    get:
      tags: [Messaging]
      summary: "Lista messaggi"
      operationId: getMessages
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: before
          in: query
          schema: {type: string, format: date-time}
        - {$ref: "#/components/parameters/QueryLimit"}
      responses:
        "200":
          description: "Lista messaggi"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Message"
    
    post:
      tags: [Messaging]
      summary: "Invia messaggio"
      operationId: sendMessage
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [message_type]
              properties:
                message_type: {type: string, enum: [text, image, video, audio, file, location]}
                content: {type: string, maxLength: 5000}
                media_id: {type: string, format: uuid}
                reply_to_id: {type: string, format: uuid}
      responses:
        "201":
          description: "Messaggio inviato"
        "403":
          description: "Non puoi inviare messaggi"

  /messages/{message_id}:
    patch:
      tags: [Messaging]
      summary: "Modifica messaggio"
      operationId: editMessage
      security: [{BearerAuth: []}]
      parameters:
        - name: message_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              required: [content]
              properties:
                content: {type: string, maxLength: 5000}
      responses:
        "200":
          description: "Messaggio modificato"
        "403": {$ref: "#/components/responses/Forbidden"}
    
    delete:
      tags: [Messaging]
      summary: "Elimina messaggio"
      operationId: deleteMessage
      security: [{BearerAuth: []}]
      parameters:
        - name: message_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: for_everyone
          in: query
          schema: {type: boolean, default: false}
      responses:
        "200":
          description: "Messaggio eliminato"

  /conversations/{conversation_id}/read:
    post:
      tags: [Messaging]
      summary: "Marca come letti"
      operationId: markAsRead
      security: [{BearerAuth: []}]
      parameters:
        - name: conversation_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Messaggi marcati come letti"

  # ─────────────────────────────────────────────────────────────────────────────
  # NOTIFICHE IN-APP (gratuite)
  # ─────────────────────────────────────────────────────────────────────────────

  /notifications:
    get:
      tags: [Messaging]
      summary: "Lista notifiche in-app"
      operationId: getNotifications
      security: [{BearerAuth: []}]
      parameters:
        - name: filter
          in: query
          schema: {type: string, enum: [all, unread], default: all}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista notifiche"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Notification"
                  meta:
                    allOf:
                      - $ref: "#/components/schemas/PaginationMeta"
                      - type: object
                        properties:
                          unread_count: {type: integer}

  /notifications/read:
    post:
      tags: [Messaging]
      summary: "Marca notifiche come lette"
      operationId: markNotificationsRead
      security: [{BearerAuth: []}]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                notification_ids:
                  type: array
                  items: {type: string, format: uuid}
                  description: "Vuoto = tutte"
      responses:
        "200":
          description: "Notifiche marcate come lette"

  /notifications/unread-count:
    get:
      tags: [Messaging]
      summary: "Conteggio non lette"
      operationId: getUnreadCount
      security: [{BearerAuth: []}]
      responses:
        "200":
          description: "Conteggio"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      unread_count: {type: integer}

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS MESSAGING
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    Conversation:
      type: object
      properties:
        id: {type: string, format: uuid}
        type: {type: string, enum: [direct, group]}
        name: {type: string, nullable: true}
        avatar_url: {type: string, nullable: true}
        participants_count: {type: integer}
        participants_preview:
          type: array
          items:
            $ref: "#/components/schemas/UserPublic"
        last_message:
          type: object
          nullable: true
          properties:
            id: {type: string, format: uuid}
            sender:
              $ref: "#/components/schemas/UserPublic"
            content_preview: {type: string}
            sent_at: {type: string, format: date-time}
        unread_count: {type: integer}
        is_muted: {type: boolean}
        is_archived: {type: boolean}
        last_activity_at: {type: string, format: date-time}
    
    Message:
      type: object
      properties:
        id: {type: string, format: uuid}
        conversation_id: {type: string, format: uuid}
        sender:
          $ref: "#/components/schemas/UserPublic"
        message_type: {type: string, enum: [text, image, video, audio, file, location, system]}
        content: {type: string, nullable: true}
        media:
          type: object
          nullable: true
          properties:
            url: {type: string}
            thumbnail_url: {type: string}
            file_name: {type: string}
            file_size: {type: integer}
        reply_to:
          type: object
          nullable: true
          properties:
            id: {type: string, format: uuid}
            sender_name: {type: string}
            content_preview: {type: string}
        is_edited: {type: boolean}
        is_deleted: {type: boolean}
        sent_at: {type: string, format: date-time}
    
    Notification:
      type: object
      properties:
        id: {type: string, format: uuid}
        type: {type: string, enum: [follow, like, comment, mention, message, follow_request, system]}
        title: {type: string}
        body: {type: string}
        source_user:
          $ref: "#/components/schemas/UserPublic"
          nullable: true
        target_type: {type: string}
        target_id: {type: string, format: uuid, nullable: true}
        action_url: {type: string}
        is_read: {type: boolean}
        created_at: {type: string, format: date-time}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO MARKETPLACE - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════
# ESCLUSI: Pagamenti (Stripe/PayPal), transazioni finanziarie

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # ANNUNCI
  # ─────────────────────────────────────────────────────────────────────────────

  /listings:
    post:
      tags: [Marketplace]
      summary: "Crea annuncio"
      operationId: createListing
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ListingCreate"
      responses:
        "201":
          description: "Annuncio creato"
        "400": {$ref: "#/components/responses/BadRequest"}
    
    get:
      tags: [Marketplace]
      summary: "Cerca annunci"
      operationId: searchListings
      parameters:
        - name: q
          in: query
          schema: {type: string}
        - name: category_id
          in: query
          schema: {type: string, format: uuid}
        - name: listing_type
          in: query
          schema: {type: string, enum: [product, service, job, housing, vehicle]}
        - name: price_min
          in: query
          schema: {type: integer}
        - name: price_max
          in: query
          schema: {type: integer}
        - name: condition
          in: query
          schema: {type: string, enum: [new, like_new, good, fair, for_parts]}
        - name: city
          in: query
          schema: {type: string}
        - name: sort
          in: query
          schema: {type: string, enum: [relevance, price_asc, price_desc, newest], default: relevance}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista annunci"

  /listings/{listing_id}:
    get:
      tags: [Marketplace]
      summary: "Dettaglio annuncio"
      operationId: getListing
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Annuncio"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Listing"
        "404": {$ref: "#/components/responses/NotFound"}
    
    patch:
      tags: [Marketplace]
      summary: "Modifica annuncio"
      operationId: updateListing
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ListingUpdate"
      responses:
        "200":
          description: "Aggiornato"
        "403": {$ref: "#/components/responses/Forbidden"}
    
    delete:
      tags: [Marketplace]
      summary: "Elimina annuncio"
      operationId: deleteListing
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Eliminato"

  /listings/{listing_id}/publish:
    post:
      tags: [Marketplace]
      summary: "Pubblica annuncio"
      operationId: publishListing
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Pubblicato"
        "400":
          description: "Annuncio incompleto"

  /users/{user_id}/listings:
    get:
      tags: [Marketplace]
      summary: "Annunci di un utente"
      operationId: getUserListings
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: status
          in: query
          schema: {type: string, enum: [active, sold, expired]}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista annunci"

  # ─────────────────────────────────────────────────────────────────────────────
  # CATEGORIE
  # ─────────────────────────────────────────────────────────────────────────────

  /categories:
    get:
      tags: [Marketplace]
      summary: "Lista categorie"
      operationId: getCategories
      parameters:
        - name: parent_id
          in: query
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Lista categorie"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/Category"

  /categories/{category_id}:
    get:
      tags: [Marketplace]
      summary: "Dettaglio categoria"
      operationId: getCategory
      parameters:
        - name: category_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Categoria"

  # ─────────────────────────────────────────────────────────────────────────────
  # OFFERTE (negoziazione senza pagamento)
  # ─────────────────────────────────────────────────────────────────────────────

  /listings/{listing_id}/offers:
    post:
      tags: [Marketplace]
      summary: "Fai offerta"
      operationId: makeOffer
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [amount]
              properties:
                amount: {type: integer, description: "Centesimi"}
                message: {type: string, maxLength: 1000}
      responses:
        "201":
          description: "Offerta inviata"
        "400":
          description: "Offerte non permesse"
        "409":
          description: "Hai già un'offerta attiva"
    
    get:
      tags: [Marketplace]
      summary: "Lista offerte (solo seller)"
      operationId: getListingOffers
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Lista offerte"

  /offers/{offer_id}/respond:
    post:
      tags: [Marketplace]
      summary: "Rispondi a offerta"
      operationId: respondToOffer
      security: [{BearerAuth: []}]
      parameters:
        - name: offer_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [action]
              properties:
                action: {type: string, enum: [accept, reject, counter]}
                counter_amount: {type: integer}
                message: {type: string}
      responses:
        "200":
          description: "Risposta inviata"

  /offers/{offer_id}/withdraw:
    post:
      tags: [Marketplace]
      summary: "Ritira offerta"
      operationId: withdrawOffer
      security: [{BearerAuth: []}]
      parameters:
        - name: offer_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Offerta ritirata"

  # ─────────────────────────────────────────────────────────────────────────────
  # PREFERITI
  # ─────────────────────────────────────────────────────────────────────────────

  /listings/{listing_id}/favorite:
    post:
      tags: [Marketplace]
      summary: "Aggiungi ai preferiti"
      operationId: addToFavorites
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Aggiunto"
        "409":
          description: "Già nei preferiti"
    
    delete:
      tags: [Marketplace]
      summary: "Rimuovi dai preferiti"
      operationId: removeFromFavorites
      security: [{BearerAuth: []}]
      parameters:
        - name: listing_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Rimosso"

  /users/me/favorites:
    get:
      tags: [Marketplace]
      summary: "I miei preferiti"
      operationId: getMyFavorites
      security: [{BearerAuth: []}]
      parameters:
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista preferiti"

  # ─────────────────────────────────────────────────────────────────────────────
  # PROFILO VENDITORE
  # ─────────────────────────────────────────────────────────────────────────────

  /sellers/{user_id}:
    get:
      tags: [Marketplace]
      summary: "Profilo venditore"
      operationId: getSellerProfile
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Profilo venditore"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/SellerProfile"
        "404": {$ref: "#/components/responses/NotFound"}

  /sellers/{user_id}/reviews:
    get:
      tags: [Marketplace]
      summary: "Recensioni venditore"
      operationId: getSellerReviews
      parameters:
        - name: user_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista recensioni"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS MARKETPLACE
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    ListingCreate:
      type: object
      required: [listing_type, title, description, category_id, price]
      properties:
        listing_type: {type: string, enum: [product, service, job, housing, vehicle]}
        title: {type: string, minLength: 5, maxLength: 200}
        description: {type: string, minLength: 20, maxLength: 10000}
        category_id: {type: string, format: uuid}
        price:
          type: object
          required: [amount, currency]
          properties:
            amount: {type: integer}
            currency: {type: string, default: "EUR"}
            type: {type: string, enum: [fixed, negotiable, auction, free, contact]}
        condition: {type: string, enum: [new, like_new, good, fair, for_parts]}
        media_ids:
          type: array
          items: {type: string, format: uuid}
          maxItems: 20
        location:
          type: object
          properties:
            city: {type: string}
            address: {type: string}
            lat: {type: number}
            lng: {type: number}
        publish: {type: boolean, default: false}
    
    ListingUpdate:
      type: object
      properties:
        title: {type: string}
        description: {type: string}
        price: {type: object}
        condition: {type: string}
        media_ids: {type: array, items: {type: string, format: uuid}}
        location: {type: object}
        status: {type: string, enum: [active, sold, deleted]}
    
    Listing:
      type: object
      properties:
        id: {type: string, format: uuid}
        listing_type: {type: string}
        title: {type: string}
        description: {type: string}
        category:
          $ref: "#/components/schemas/Category"
        price:
          type: object
          properties:
            amount: {type: integer}
            currency: {type: string}
            type: {type: string}
        condition: {type: string}
        media:
          type: array
          items:
            $ref: "#/components/schemas/MediaItem"
        location: {type: object}
        seller:
          $ref: "#/components/schemas/SellerProfile"
        status: {type: string}
        views_count: {type: integer}
        favorites_count: {type: integer}
        is_favorite: {type: boolean}
        published_at: {type: string, format: date-time}
        created_at: {type: string, format: date-time}
    
    Category:
      type: object
      properties:
        id: {type: string, format: uuid}
        parent_id: {type: string, format: uuid, nullable: true}
        name: {type: string}
        slug: {type: string}
        icon: {type: string}
        listing_count: {type: integer}
    
    SellerProfile:
      type: object
      properties:
        user_id: {type: string, format: uuid}
        username: {type: string}
        display_name: {type: string}
        avatar_url: {type: string}
        rating_average: {type: number}
        rating_count: {type: integer}
        response_rate: {type: integer}
        active_listings_count: {type: integer}
        total_sales: {type: integer}
        is_verified: {type: boolean}
        member_since: {type: string, format: date-time}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO BOOKING - API GRATUITE
# ═══════════════════════════════════════════════════════════════════════════════
# ESCLUSI: Pagamenti online - prenotazioni con pagamento in loco

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # RISORSE PRENOTABILI
  # ─────────────────────────────────────────────────────────────────────────────

  /resources:
    post:
      tags: [Booking]
      summary: "Crea risorsa prenotabile"
      operationId: createResource
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ResourceCreate"
      responses:
        "201":
          description: "Risorsa creata"
    
    get:
      tags: [Booking]
      summary: "Cerca risorse"
      operationId: searchResources
      parameters:
        - name: q
          in: query
          schema: {type: string}
        - name: resource_type
          in: query
          schema: {type: string, enum: [service, room, equipment, person, vehicle, table]}
        - name: category
          in: query
          schema: {type: string}
        - name: city
          in: query
          schema: {type: string}
        - name: date
          in: query
          schema: {type: string, format: date}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista risorse"

  /resources/{resource_id}:
    get:
      tags: [Booking]
      summary: "Dettaglio risorsa"
      operationId: getResource
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Risorsa"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/BookableResource"
        "404": {$ref: "#/components/responses/NotFound"}
    
    patch:
      tags: [Booking]
      summary: "Modifica risorsa"
      operationId: updateResource
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/ResourceUpdate"
      responses:
        "200":
          description: "Aggiornata"
        "403": {$ref: "#/components/responses/Forbidden"}
    
    delete:
      tags: [Booking]
      summary: "Elimina risorsa"
      operationId: deleteResource
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Eliminata"
        "400":
          description: "Prenotazioni attive presenti"

  /users/me/resources:
    get:
      tags: [Booking]
      summary: "Le mie risorse (provider)"
      operationId: getMyResources
      security: [{BearerAuth: []}]
      parameters:
        - name: status
          in: query
          schema: {type: string, enum: [draft, active, paused]}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista risorse"

  # ─────────────────────────────────────────────────────────────────────────────
  # DISPONIBILITÀ
  # ─────────────────────────────────────────────────────────────────────────────

  /resources/{resource_id}/availability:
    get:
      tags: [Booking]
      summary: "Regole disponibilità"
      operationId: getAvailability
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Regole"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      rules:
                        type: array
                        items:
                          $ref: "#/components/schemas/AvailabilityRule"
                      exceptions:
                        type: array
                        items:
                          $ref: "#/components/schemas/AvailabilityException"
    
    put:
      tags: [Booking]
      summary: "Imposta disponibilità"
      operationId: setAvailability
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [rules]
              properties:
                rules:
                  type: array
                  items:
                    type: object
                    required: [day_of_week, start_time, end_time]
                    properties:
                      day_of_week: {type: integer, minimum: 0, maximum: 6}
                      start_time: {type: string, pattern: "^([01]?[0-9]|2[0-3]):[0-5][0-9]$"}
                      end_time: {type: string}
                      is_available: {type: boolean, default: true}
      responses:
        "200":
          description: "Aggiornata"

  /resources/{resource_id}/availability/exceptions:
    post:
      tags: [Booking]
      summary: "Aggiungi eccezione"
      operationId: addException
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [date, exception_type]
              properties:
                date: {type: string, format: date}
                exception_type: {type: string, enum: [closed, modified_hours, extra_availability]}
                start_time: {type: string}
                end_time: {type: string}
                reason: {type: string}
      responses:
        "201":
          description: "Eccezione creata"

  /resources/{resource_id}/slots:
    get:
      tags: [Booking]
      summary: "Slot disponibili"
      operationId: getAvailableSlots
      parameters:
        - name: resource_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: start_date
          in: query
          required: true
          schema: {type: string, format: date}
        - name: end_date
          in: query
          required: true
          schema: {type: string, format: date}
        - name: duration_minutes
          in: query
          schema: {type: integer, default: 60}
      responses:
        "200":
          description: "Slot disponibili"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      slots:
                        type: array
                        items:
                          type: object
                          properties:
                            date: {type: string, format: date}
                            times:
                              type: array
                              items:
                                type: object
                                properties:
                                  start_time: {type: string}
                                  end_time: {type: string}
                                  is_available: {type: boolean}

  # ─────────────────────────────────────────────────────────────────────────────
  # PRENOTAZIONI
  # ─────────────────────────────────────────────────────────────────────────────

  /bookings:
    post:
      tags: [Booking]
      summary: "Crea prenotazione"
      operationId: createBooking
      security: [{BearerAuth: []}]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [resource_id, start_datetime, end_datetime]
              properties:
                resource_id: {type: string, format: uuid}
                start_datetime: {type: string, format: date-time}
                end_datetime: {type: string, format: date-time}
                party_size: {type: integer, default: 1}
                notes: {type: string, maxLength: 500}
      responses:
        "201":
          description: "Prenotazione creata"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Booking"
        "400":
          description: "Slot non disponibile"
        "409":
          description: "Slot già prenotato"

  /bookings/{booking_id}:
    get:
      tags: [Booking]
      summary: "Dettaglio prenotazione"
      operationId: getBooking
      security: [{BearerAuth: []}]
      parameters:
        - name: booking_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Prenotazione"
        "403": {$ref: "#/components/responses/Forbidden"}
        "404": {$ref: "#/components/responses/NotFound"}

  /bookings/{booking_id}/cancel:
    post:
      tags: [Booking]
      summary: "Cancella prenotazione"
      operationId: cancelBooking
      security: [{BearerAuth: []}]
      parameters:
        - name: booking_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                reason: {type: string, maxLength: 500}
      responses:
        "200":
          description: "Cancellata"
        "400":
          description: "Cancellazione non permessa"

  /bookings/{booking_id}/confirm:
    post:
      tags: [Booking]
      summary: "Conferma prenotazione (provider)"
      operationId: confirmBooking
      security: [{BearerAuth: []}]
      parameters:
        - name: booking_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                provider_notes: {type: string}
      responses:
        "200":
          description: "Confermata"

  /bookings/{booking_id}/complete:
    post:
      tags: [Booking]
      summary: "Completa prenotazione (provider)"
      operationId: completeBooking
      security: [{BearerAuth: []}]
      parameters:
        - name: booking_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Completata"

  /users/me/bookings:
    get:
      tags: [Booking]
      summary: "Le mie prenotazioni (booker)"
      operationId: getMyBookings
      security: [{BearerAuth: []}]
      parameters:
        - name: status
          in: query
          schema: {type: string, enum: [pending, confirmed, cancelled, completed]}
        - name: time
          in: query
          schema: {type: string, enum: [upcoming, past], default: upcoming}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista prenotazioni"

  /users/me/provider-bookings:
    get:
      tags: [Booking]
      summary: "Prenotazioni ricevute (provider)"
      operationId: getProviderBookings
      security: [{BearerAuth: []}]
      parameters:
        - name: resource_id
          in: query
          schema: {type: string, format: uuid}
        - name: status
          in: query
          schema: {type: string}
        - name: date
          in: query
          schema: {type: string, format: date}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista prenotazioni"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS BOOKING
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    ResourceCreate:
      type: object
      required: [resource_type, name, pricing]
      properties:
        resource_type: {type: string, enum: [service, room, equipment, person, vehicle, table]}
        name: {type: string, maxLength: 200}
        description: {type: string, maxLength: 5000}
        category: {type: string}
        media_ids:
          type: array
          items: {type: string, format: uuid}
        location:
          type: object
          properties:
            address: {type: string}
            city: {type: string}
            lat: {type: number}
            lng: {type: number}
        pricing:
          type: object
          required: [base_price, currency]
          properties:
            base_price: {type: integer}
            currency: {type: string, default: "EUR"}
            price_type: {type: string, enum: [per_hour, per_session, fixed], default: per_hour}
        booking_settings:
          type: object
          properties:
            min_advance_hours: {type: integer, default: 1}
            max_advance_days: {type: integer, default: 30}
            buffer_minutes: {type: integer, default: 0}
            cancellation_policy: {type: string, enum: [flexible, moderate, strict], default: flexible}
            requires_approval: {type: boolean, default: false}
        capacity: {type: integer, default: 1}
    
    ResourceUpdate:
      type: object
      properties:
        name: {type: string}
        description: {type: string}
        pricing: {type: object}
        booking_settings: {type: object}
        capacity: {type: integer}
        status: {type: string, enum: [draft, active, paused]}
    
    BookableResource:
      type: object
      properties:
        id: {type: string, format: uuid}
        resource_type: {type: string}
        name: {type: string}
        description: {type: string}
        category: {type: string}
        media:
          type: array
          items:
            $ref: "#/components/schemas/MediaItem"
        location: {type: object}
        owner:
          $ref: "#/components/schemas/UserPublic"
        pricing: {type: object}
        booking_settings: {type: object}
        capacity: {type: integer}
        rating_average: {type: number}
        rating_count: {type: integer}
        status: {type: string}
        created_at: {type: string, format: date-time}
    
    AvailabilityRule:
      type: object
      properties:
        id: {type: string, format: uuid}
        day_of_week: {type: integer}
        start_time: {type: string}
        end_time: {type: string}
        is_available: {type: boolean}
    
    AvailabilityException:
      type: object
      properties:
        id: {type: string, format: uuid}
        date: {type: string, format: date}
        exception_type: {type: string}
        start_time: {type: string}
        end_time: {type: string}
        reason: {type: string}
    
    Booking:
      type: object
      properties:
        id: {type: string, format: uuid}
        booking_code: {type: string}
        resource:
          type: object
          properties:
            id: {type: string, format: uuid}
            name: {type: string}
            cover_image_url: {type: string}
        booker:
          $ref: "#/components/schemas/UserPublic"
        provider:
          $ref: "#/components/schemas/UserPublic"
        start_datetime: {type: string, format: date-time}
        end_datetime: {type: string, format: date-time}
        duration_minutes: {type: integer}
        party_size: {type: integer}
        status: {type: string, enum: [pending, confirmed, cancelled, completed, no_show]}
        price:
          type: object
          properties:
            subtotal: {type: integer}
            total: {type: integer}
            currency: {type: string}
        notes: {type: string}
        provider_notes: {type: string}
        cancelled_at: {type: string, format: date-time, nullable: true}
        cancellation_reason: {type: string, nullable: true}
        created_at: {type: string, format: date-time}



# ═══════════════════════════════════════════════════════════════════════════════
# MODULO COMMERCE - API GRATUITE (SOLO CATALOGO)
# ═══════════════════════════════════════════════════════════════════════════════
# ESCLUSI: Checkout, pagamenti, ordini con transazioni finanziarie

paths:

  # ─────────────────────────────────────────────────────────────────────────────
  # PRODOTTI
  # ─────────────────────────────────────────────────────────────────────────────

  /products:
    get:
      tags: [Commerce]
      summary: "Cerca prodotti"
      operationId: searchProducts
      parameters:
        - name: q
          in: query
          schema: {type: string}
        - name: category_id
          in: query
          schema: {type: string, format: uuid}
        - name: brand
          in: query
          schema: {type: string}
        - name: price_min
          in: query
          schema: {type: integer}
        - name: price_max
          in: query
          schema: {type: integer}
        - name: in_stock
          in: query
          schema: {type: boolean}
        - name: sort
          in: query
          schema: {type: string, enum: [relevance, price_asc, price_desc, newest, best_selling, rating], default: relevance}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista prodotti"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/ProductSummary"
                  meta:
                    $ref: "#/components/schemas/PaginationMeta"

  /products/{product_id}:
    get:
      tags: [Commerce]
      summary: "Dettaglio prodotto"
      operationId: getProduct
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Prodotto"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Product"
        "404": {$ref: "#/components/responses/NotFound"}

  /products/slug/{slug}:
    get:
      tags: [Commerce]
      summary: "Prodotto per slug (SEO)"
      operationId: getProductBySlug
      parameters:
        - name: slug
          in: path
          required: true
          schema: {type: string}
      responses:
        "200":
          description: "Prodotto"
        "404": {$ref: "#/components/responses/NotFound"}

  /products/{product_id}/variants:
    get:
      tags: [Commerce]
      summary: "Varianti prodotto"
      operationId: getProductVariants
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Lista varianti"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: object
                    properties:
                      variants:
                        type: array
                        items:
                          $ref: "#/components/schemas/ProductVariant"
                      options:
                        type: object
                        description: "{Colore: ['Rosso', 'Blu'], Taglia: ['S', 'M', 'L']}"

  /products/{product_id}/reviews:
    get:
      tags: [Commerce]
      summary: "Recensioni prodotto"
      operationId: getProductReviews
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
        - name: rating
          in: query
          schema: {type: integer, minimum: 1, maximum: 5}
        - name: verified_only
          in: query
          schema: {type: boolean, default: false}
        - name: sort
          in: query
          schema: {type: string, enum: [newest, oldest, rating_high, rating_low, helpful], default: newest}
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Lista recensioni"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/ProductReview"
                  meta:
                    allOf:
                      - $ref: "#/components/schemas/PaginationMeta"
                      - type: object
                        properties:
                          summary:
                            type: object
                            properties:
                              average_rating: {type: number}
                              total_count: {type: integer}
                              rating_distribution: {type: object}
    
    post:
      tags: [Commerce]
      summary: "Scrivi recensione"
      operationId: createProductReview
      security: [{BearerAuth: []}]
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [rating]
              properties:
                rating: {type: integer, minimum: 1, maximum: 5}
                title: {type: string, maxLength: 150}
                content: {type: string, maxLength: 5000}
                media_ids:
                  type: array
                  items: {type: string, format: uuid}
                  maxItems: 5
      responses:
        "201":
          description: "Recensione creata"
        "409":
          description: "Già recensito"

  # ─────────────────────────────────────────────────────────────────────────────
  # CATEGORIE PRODOTTI
  # ─────────────────────────────────────────────────────────────────────────────

  /product-categories:
    get:
      tags: [Commerce]
      summary: "Lista categorie prodotti"
      operationId: getProductCategories
      parameters:
        - name: parent_id
          in: query
          schema: {type: string, format: uuid}
        - name: include_count
          in: query
          schema: {type: boolean, default: false}
      responses:
        "200":
          description: "Lista categorie"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    type: array
                    items:
                      $ref: "#/components/schemas/ProductCategory"

  /product-categories/{category_id}:
    get:
      tags: [Commerce]
      summary: "Dettaglio categoria"
      operationId: getProductCategory
      parameters:
        - name: category_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Categoria con sottocategorie"

  # ─────────────────────────────────────────────────────────────────────────────
  # CARRELLO (senza checkout)
  # ─────────────────────────────────────────────────────────────────────────────

  /cart:
    get:
      tags: [Commerce]
      summary: "Carrello corrente"
      operationId: getCart
      description: "Funziona per utenti autenticati e anonimi (session cookie)"
      responses:
        "200":
          description: "Carrello"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Cart"
    
    delete:
      tags: [Commerce]
      summary: "Svuota carrello"
      operationId: clearCart
      responses:
        "200":
          description: "Carrello svuotato"

  /cart/items:
    post:
      tags: [Commerce]
      summary: "Aggiungi al carrello"
      operationId: addToCart
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [product_id]
              properties:
                product_id: {type: string, format: uuid}
                variant_id: {type: string, format: uuid}
                quantity: {type: integer, minimum: 1, default: 1}
      responses:
        "200":
          description: "Aggiunto"
          content:
            application/json:
              schema:
                type: object
                properties:
                  success: {type: boolean}
                  data:
                    $ref: "#/components/schemas/Cart"
        "400":
          description: "Prodotto non disponibile"

  /cart/items/{item_id}:
    patch:
      tags: [Commerce]
      summary: "Aggiorna quantità"
      operationId: updateCartItem
      parameters:
        - name: item_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [quantity]
              properties:
                quantity: {type: integer, minimum: 0}
      responses:
        "200":
          description: "Aggiornato"
        "400":
          description: "Quantità non disponibile"
    
    delete:
      tags: [Commerce]
      summary: "Rimuovi dal carrello"
      operationId: removeFromCart
      parameters:
        - name: item_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Rimosso"

  /cart/merge:
    post:
      tags: [Commerce]
      summary: "Unisci carrello anonimo dopo login"
      operationId: mergeCart
      security: [{BearerAuth: []}]
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                anonymous_cart_id: {type: string, format: uuid}
      responses:
        "200":
          description: "Carrelli uniti"

  # ─────────────────────────────────────────────────────────────────────────────
  # WISHLIST
  # ─────────────────────────────────────────────────────────────────────────────

  /wishlist:
    get:
      tags: [Commerce]
      summary: "La mia wishlist"
      operationId: getWishlist
      security: [{BearerAuth: []}]
      parameters:
        - {$ref: "#/components/parameters/QueryLimit"}
        - {$ref: "#/components/parameters/QueryCursor"}
      responses:
        "200":
          description: "Wishlist"

  /wishlist/{product_id}:
    post:
      tags: [Commerce]
      summary: "Aggiungi a wishlist"
      operationId: addToWishlist
      security: [{BearerAuth: []}]
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Aggiunto"
        "409":
          description: "Già in wishlist"
    
    delete:
      tags: [Commerce]
      summary: "Rimuovi da wishlist"
      operationId: removeFromWishlist
      security: [{BearerAuth: []}]
      parameters:
        - name: product_id
          in: path
          required: true
          schema: {type: string, format: uuid}
      responses:
        "200":
          description: "Rimosso"

# ═══════════════════════════════════════════════════════════════════════════════
# SCHEMAS COMMERCE
# ═══════════════════════════════════════════════════════════════════════════════

components:
  schemas:
    
    ProductSummary:
      type: object
      properties:
        id: {type: string, format: uuid}
        name: {type: string}
        slug: {type: string}
        short_description: {type: string}
        cover_image_url: {type: string}
        price:
          type: object
          properties:
            amount: {type: integer}
            currency: {type: string}
            compare_at_amount: {type: integer, nullable: true}
        in_stock: {type: boolean}
        rating_average: {type: number}
        rating_count: {type: integer}
        is_in_wishlist: {type: boolean}
    
    Product:
      allOf:
        - $ref: "#/components/schemas/ProductSummary"
        - type: object
          properties:
            sku: {type: string}
            description: {type: string}
            brand: {type: string}
            category:
              $ref: "#/components/schemas/ProductCategory"
            media:
              type: array
              items:
                $ref: "#/components/schemas/MediaItem"
            attributes: {type: object}
            tags:
              type: array
              items: {type: string}
            inventory:
              type: object
              properties:
                quantity: {type: integer}
                low_stock: {type: boolean}
                allow_backorder: {type: boolean}
            has_variants: {type: boolean}
            seller:
              $ref: "#/components/schemas/UserPublic"
            created_at: {type: string, format: date-time}
    
    ProductVariant:
      type: object
      properties:
        id: {type: string, format: uuid}
        sku: {type: string}
        name: {type: string}
        options: {type: object}
        price:
          type: object
          nullable: true
          properties:
            amount: {type: integer}
            currency: {type: string}
        image_url: {type: string}
        in_stock: {type: boolean}
        quantity: {type: integer, nullable: true}
        is_default: {type: boolean}
    
    ProductCategory:
      type: object
      properties:
        id: {type: string, format: uuid}
        parent_id: {type: string, format: uuid, nullable: true}
        name: {type: string}
        slug: {type: string}
        description: {type: string}
        image_url: {type: string}
        products_count: {type: integer}
    
    ProductReview:
      type: object
      properties:
        id: {type: string, format: uuid}
        reviewer:
          $ref: "#/components/schemas/UserPublic"
        rating: {type: integer}
        title: {type: string}
        content: {type: string}
        media:
          type: array
          items:
            $ref: "#/components/schemas/MediaItem"
        is_verified_purchase: {type: boolean}
        helpful_count: {type: integer}
        seller_response: {type: string, nullable: true}
        created_at: {type: string, format: date-time}
    
    Cart:
      type: object
      properties:
        id: {type: string, format: uuid}
        items:
          type: array
          items:
            $ref: "#/components/schemas/CartItem"
        items_count: {type: integer}
        subtotal: {type: integer}
        discount_amount: {type: integer}
        coupon_code: {type: string, nullable: true}
        total: {type: integer}
        currency: {type: string}
        expires_at: {type: string, format: date-time}
    
    CartItem:
      type: object
      properties:
        id: {type: string, format: uuid}
        product:
          $ref: "#/components/schemas/ProductSummary"
        variant:
          $ref: "#/components/schemas/ProductVariant"
          nullable: true
        quantity: {type: integer}
        unit_price: {type: integer}
        total_price: {type: integer}
        in_stock: {type: boolean}
        max_quantity: {type: integer}

