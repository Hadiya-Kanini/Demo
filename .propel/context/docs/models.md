Design Model Template

## UML Models Overview
This document contains a complete set of UML views for the Email+Password Authentication Service:
- System Context (PlantUML) — boundary and external actors
- Component Architecture (Mermaid) — service modules & responsibilities
- Deployment Topology (PlantUML) — cloud landing zone & network placement
- Data Flow (PlantUML) — major data transformations and control points
- Logical Data Model / ERD (Mermaid) — entities, attributes, relationships
- AI Architecture (Mermaid) — optional adaptive detection pipeline
- Use Case Sequence Diagrams (Mermaid) — one sequence per UC-001..UC-008 from spec

Architecture Pattern Rationale (<12 lines):
- Pattern Selected: Layered microservice (single focused Auth microservice) with modular subcomponents.
- NFR Drivers: Security (confidentiality/integrity), auditability, low-latency auth (2s p95), availability.
- Key Decisions: Use KMS for signing keys, Argon2id for password hashing + versioning and rehash-on-login, Redis for fast counters/blacklist, token strategy: short-lived JWT (30m) with token_id for revocation/blacklist.
- Trade-offs: Stateless tokens simplify scale (short TTL + blacklist adds cache dependency) — chosen for simplicity + revocation responsiveness.

NFR-to-Architecture Mapping:
| NFR | Architectural Decision | Justification |
|-----|-----------------------|---------------|
| Auth latency (2s p95) | API Gateway + Autoscaled Auth Service; short synchronous flows | Keeps request path minimal; autoscale to meet 2s p95 target |
| Password security | Argon2id with algorithm versioning + rehash-on-login | Strong hashing, upgrade path for legacy hashes |
| Token revocation | Short-lived JWT (30m) + Redis blacklist by token_id | Allows stateless verification with fast revocation within 1 minute |
| Brute-force protection | Redis counters + API Gateway rate limiting + account lockout (5 attempts) | Low-latency counters for atomic increments and global enforcement |
| Auditability | Structured JSON logs emitted to SIEM within 30s | Enables security/ops querying and compliance evidence |
| Email delivery | Background worker + external Email Provider | Asynchronous delivery with retry and delivery logging |

## Architectural Views

### System Context Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor "End User (Browser)" as EndUser #LightBlue
actor "Administrator / Support" as Admin #LightBlue
actor "Email Provider (External)" as EmailExt #LightGray
actor "API Gateway / Rate Limiter" as APIGW #LightGray

rectangle "Authentication Service" as AuthSvc #LightGreen {
  component "Auth API" as AuthAPI
  component "Auth Core" as AuthCore
}

database "User DB (RDBMS)" as UserDB #Yellow
component "Redis (Cache / Blacklist)" as Redis #Yellow
component "KMS / Secret Store" as KMS #LightGray
component "SIEM / Logging Pipeline" as SIEM #LightGray
component "Background Worker" as Worker #Orange

EndUser --> APIGW : HTTPS / POST /auth/login, /auth/forgot-password
APIGW --> AuthAPI : Forward request (rate-limit, WAF)
AuthAPI --> AuthCore : Validate / Business logic
AuthCore --> UserDB : Read / Write user, tokens, audit
AuthCore --> Redis : failed_attempts, token blacklist, rate counters
AuthCore --> KMS : Sign/verify tokens
AuthCore --> SIEM : Emit structured audit events
AuthCore --> Worker : Enqueue email / cleanup jobs
Worker --> EmailExt : Send reset email (TLS)
Admin --> AuthAPI : Admin unlock / audit queries
@enduml
```

### Component Architecture Diagram
```mermaid
graph LR
  subgraph Client Layer
    Browser[Web Client (SPA) - UI & client validation]
  end

  subgraph Edge Layer
    APIGW[API Gateway (Ingress) - Rate limiting, WAF, CAPTCHA gating]
  end

  subgraph "Authentication Service (Auth)" 
    direction TB
    AuthAPI[Auth API (HTTP) - Endpoints: /login, /logout, /forgot, /reset]
    AuthCore[Auth Core (C# / Go) - Credential verification, RBAC, token orchestration]
    PasswordSvc[Password Service (Argon2id) - Hash & verify, rehash-on-login]
    TokenSvc[Token Service - token_id generation, JWT creation/validation]
    LockCoordinator[Lock & Rate Coordinator - increments, lock semantics]
    ResetManager[Reset Manager - token create + hashed store]
    MFA[Optional MFA Module - TOTP enrollment & verify]
    AuditEmitter[Audit Logger - structured events -> SIEM]
    SessionStore[(Session / Blacklist Connector) - Redis]
  end

  subgraph Data Layer
    UserDB[(User DB - PostgreSQL) - Users, Roles, tokens (hashed), audit])
    Redis[(Redis) - counters, blacklist, rate windows]
    KMS[(KMS / Secret Store) - signing keys, OTP secrets]
  end

  subgraph Infra
    Worker[Background Worker - jobs: email, cleanup, retries]
    EmailProv[Email Provider - SMTP/API]
    SIEM[SIEM / Logging Pipeline]
  end

  Browser -->|HTTPS| APIGW
  APIGW -->|HTTPS| AuthAPI
  AuthAPI --> AuthCore
  AuthCore --> PasswordSvc
  AuthCore --> TokenSvc
  AuthCore --> LockCoordinator
  AuthCore --> ResetManager
  AuthCore --> MFA
  AuthCore --> AuditEmitter
  AuthCore --> SessionStore
  PasswordSvc -->|SQL| UserDB
  ResetManager -->|SQL| UserDB
  SessionStore -->|TCP| Redis
  TokenSvc -->|KMS API| KMS
  AuditEmitter -->|HTTPS| SIEM
  Worker -->|SMTP/API| EmailProv
  AuthCore -->|Enqueue| Worker
```

### Deployment Architecture Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle
skinparam linetype ortho

node "Public Internet" as Internet

cloud "CDN / Static Hosting" as CDN
Internet --> CDN : Browser assets

node "API Gateway (Public)" as APIGW #LightGray
APIGW --> "WAF" : Request filtering

folder "VPC (Private)" {
  node "LB / Ingress" as LB
  node "Auth Service Cluster (k8s) - Auth API & Core" as AuthCluster #LightGreen
  node "Background Worker (k8s Jobs)" as Workers #Orange
  database "User DB (RDS/Postgres) - Private Subnet" as UserDB #Yellow
  database "Redis (ElastiCache) - Private Subnet" as Redis #Yellow
  node "KMS / Secrets (Managed)" as KMS #LightGray
  node "SIEM Ingest / Logging" as SIEM #LightGray
}

CDN --> APIGW : TLS
APIGW --> LB : TLS
LB --> AuthCluster : HTTPS
AuthCluster --> UserDB : Private DB connections (TLS)
AuthCluster --> Redis : TLS / AUTH
AuthCluster --> KMS : Managed KMS API
AuthCluster --> SIEM : Secure Log Stream
Workers --> EmailProv : Outbound TLS
note right of UserDB : Backups, encryption at rest
note right of Redis : High-speed counters, blacklist
@enduml
```

### Data Flow Diagram
```plantuml
@startuml
!define PROCESS rectangle
!define DATASTORE database
!define EXTERNAL component

EXTERNAL "End User (Browser)" as user
PROCESS "API Gateway" as gw
PROCESS "Auth Service - Login Flow" as auth
DATASTORE "User DB" as db
DATASTORE "Redis (Counters/Blacklist)" as cache
EXTERNAL "KMS (Signing Keys)" as kms
EXTERNAL "Email Provider" as email
PROCESS "Background Worker" as worker
EXTERNAL "SIEM / Logging" as siem

user -> gw : POST /auth/login (email,password) over TLS
gw -> auth : Forward request (rate-limit, CAPTCHA)
auth -> cache : Check account lock / rate counters
auth -> db : SELECT user by email
db --> auth : user record (hash, roles, metadata)
auth -> auth : Verify password (Argon2id, constant-time)
alt password valid
  auth -> kms : Sign token (or retrieve key) 
  kms --> auth : signature
  auth -> cache : Reset failed_attempts
  auth -> db : Update last_login_at, optional rehash
  auth -> siem : Emit login_success event
  auth --> gw : 200 OK + token / redirect (role)
else invalid
  auth -> cache : Increment failed_attempts
  auth -> cache : If threshold -> set locked_until
  auth -> siem : Emit login_failure event
  auth --> gw : 401 Invalid credentials (generic)
end

user -> gw : POST /auth/forgot-password (email)
gw -> auth : Forward, validate
auth -> db : Create PasswordResetToken (store hashed token, expiry)
auth -> worker : Enqueue send-email job
worker -> email : Send reset link
worker -> siem : Emit reset_email_sent (delivery status)
@enduml
```

### Logical Data Model (ERD)
```mermaid
erDiagram
  USER {
    string user_id PK "UUID"
    string email "unique"
    string password_hash
    string hash_algo
    int failed_attempts
    datetime locked_until
    bool mfa_enabled
    datetime created_at
    datetime updated_at
    datetime last_login_at
  }
  ROLE {
    string role_id PK "UUID"
    string name
    string description
    string landing_path
  }
  USER_ROLE {
    string user_id FK
    string role_id FK
  }
  PASSWORD_RESET_TOKEN {
    string token_id PK "UUID"
    string user_id FK
    string token_hash
    datetime created_at
    datetime expires_at
    datetime used_at
    bool single_use
  }
  AUTH_SESSION {
    string session_id PK "UUID"
    string user_id FK
    datetime issued_at
    datetime expires_at
    bool revoked
    string token_type
    string device_info
    string ip_address
  }
  MFA_ENROLLMENT {
    string enrollment_id PK "UUID"
    string user_id FK
    string mfa_type
    string secret_ref
    string recovery_codes_hash
    datetime created_at
  }
  AUDIT_LOG {
    string audit_id PK "UUID"
    string user_id FK "nullable"
    string event_type
    datetime event_time
    string source_ip
    string user_agent
    string metadata_json
    string outcome
  }
  PASSWORD_HASH_ALGO {
    string algorithm_id PK "UUID"
    string name
    string version
    string params_json
  }

  USER ||--o{ PASSWORD_RESET_TOKEN : "issues"
  USER ||--o{ AUTH_SESSION : "has"
  USER ||--o{ MFA_ENROLLMENT : "enrolls"
  USER ||--o{ AUDIT_LOG : "owns"
  USER }|--|{ USER_ROLE : "assigned"
  ROLE ||--o{ USER_ROLE : "maps"
  USER ||--|{ PASSWORD_HASH_ALGO : "uses" 
```

## AI Architecture Diagrams (adaptive detection - optional)
Rationale: FR-011 labeled [HYBRID] — adaptive detection is optional and auditable. The pipeline below is an optional, auditable component that scores login risk and recommends CAPTCHA/block.

Component overview (Mermaid):
```mermaid
graph LR
  LoginEvents[Login Events Stream]
  FeatureStore[Feature Store (Redis/TS DB)]
  ModelService[Anomaly Detection Model (Scoring) - auditable]
  DecisionEngine[Decision Engine - rules + score thresholds]
  AuthSvc[Auth Service]
  SIEM[SIEM / Audit]
  AdminUI[Security Ops Console]

  LoginEvents --> FeatureStore
  FeatureStore --> ModelService
  ModelService --> DecisionEngine
  DecisionEngine --> AuthSvc : risk_score / action (allow, captcha, block)
  DecisionEngine --> SIEM : explanation + features
  SIEM --> AdminUI : alerts, audit
```

AI-assisted flow (Mermaid sequence):
```mermaid
sequenceDiagram
  participant Browser as "End User (Browser)"
  participant APIGW as "API Gateway"
  participant Auth as "Authentication Service"
  participant AD as "Anomaly Detector (Model Service)"
  participant Cache as "Redis (counters)"

  Browser->>APIGW: POST /auth/login (email,password)
  APIGW->>Auth: Forward request
  Auth->>Cache: fetch recent activity (per-IP/per-account)
  Auth->>AD: send features for scoring (async or sync)
  alt low risk
    AD-->>Auth: score=low
    Auth->>Cache: proceed with normal verification
  else high risk
    AD-->>Auth: score=high
    Auth-->>APIGW: respond-> require CAPTCHA / block
  end
```

## Use Case Sequence Diagrams

> Note: Each sequence diagram below references the relevant use case definition in the spec.

#### UC-001: User Login (Happy Path)
**Source**: [spec.md#UC-001](.propel/context/docs/spec.md#UC-001)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant DB as "User DB"
    participant Cache as "Redis"

    Note over EndUser,Cache: UC-001 - User Login (Happy Path)

    EndUser->>APIGateway: POST /auth/login (email,password)
    APIGateway->>Auth: Forward request (apply rate-limit/CAPTCHA)
    Auth->>Cache: check locked_until / failed_attempts
    Cache-->>Auth: not locked
    Auth->>DB: SELECT * FROM users WHERE email=?
    DB-->>Auth: user record (password_hash, roles, hash_algo)
    Auth->>Auth: verify password (Argon2id, constant-time)
    alt password valid
      Auth->>Cache: reset failed_attempts
      Auth->>Auth: issue token (JWT with token_id, exp=30m) - sign via KMS
      Auth->>DB: update last_login_at (and rehash if algorithm changed)
      Auth->>APIGateway: 200 OK + Set-Cookie / token + role redirect
      APIGateway->>EndUser: 200 OK + redirect
    else invalid credentials
      Auth->>Cache: increment failed_attempts
      alt threshold reached
        Auth->>Cache: set locked_until
        Auth-->>APIGateway: 423 Account is locked
        APIGateway-->>EndUser: 423 Account locked
      else
        Auth-->>APIGateway: 401 Invalid credentials
        APIGateway-->>EndUser: 401 Invalid credentials
      end
    end
```

#### UC-002: Login Failed — Invalid Credentials
**Source**: [spec.md#UC-002](.propel/context/docs/spec.md#UC-002)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant Cache as "Redis"

    Note over EndUser,Auth: UC-002 - Login Failed (Invalid Credentials)

    EndUser->>APIGateway: POST /auth/login (email,password)
    APIGateway->>Auth: Forward request
    Auth->>Cache: check locked_until / failed_attempts
    Cache-->>Auth: value
    Auth->>Auth: attempt credential lookup & verify
    Auth->>Cache: increment failed_attempts
    alt reached threshold
      Auth->>Cache: set locked_until
      Auth-->>APIGateway: 423 Account locked
      APIGateway-->>EndUser: 423 Account is locked
    else
      Auth-->>APIGateway: 401 Invalid credentials
      APIGateway-->>EndUser: 401 Invalid credentials
    end
```

#### UC-003: Login Validation — Empty/Invalid Fields
**Source**: [spec.md#UC-003](.propel/context/docs/spec.md#UC-003)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant Browser as "Client Validation"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"

    Note over EndUser,Auth: UC-003 - Client & Server Validation

    EndUser->>Browser: Enter email/password
    Browser-->>EndUser: show inline validation (required, email format)
    alt client validation passed
      Browser->>APIGateway: POST /auth/login
      APIGateway->>Auth: Forward request
      Auth->>Auth: server-side validate (email format, length)
      alt server validation fail
        Auth-->>APIGateway: 400 Bad Request (field errors)
        APIGateway-->>Browser: 400 Bad Request
      else
        Auth-->>APIGateway: proceed to auth flow (see UC-001)
      end
    else validation failed
      Browser-->>EndUser: prevent submit, inline errors
    end
```

#### UC-004: Account Lockout (Failed Attempts -> Locked)
**Source**: [spec.md#UC-004](.propel/context/docs/spec.md#UC-004)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant Cache as "Redis"
    participant Admin as "Administrator"

    Note over EndUser,Admin: UC-004 - Account Lockout & Admin Unlock

    EndUser->>APIGateway: POST /auth/login (bad creds)
    APIGateway->>Auth: Forward
    Auth->>Cache: increment failed_attempts
    Cache-->>Auth: failed_attempts count
    alt failed_attempts >= threshold
      Auth->>Cache: set locked_until
      Auth->>Auth: emit audit log (lock event)
      Auth-->>APIGateway: 423 Account is locked
      APIGateway-->>EndUser: 423 Account is locked
    else
      Auth-->>APIGateway: 401 Invalid credentials
      APIGateway-->>EndUser: 401 Invalid credentials
    end

    Admin->>APIGateway: POST /admin/unlock (user_id)
    APIGateway->>Auth: Forward admin request (authz)
    Auth->>Cache: reset failed_attempts; clear locked_until
    Auth->>Auth: emit audit log (unlock by admin)
    Auth-->>APIGateway: 200 OK
    APIGateway-->>Admin: 200 OK
```

#### UC-005: Forgot Password (Request Reset)
**Source**: [spec.md#UC-005](.propel/context/docs/spec.md#UC-005)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant DB as "User DB"
    participant Worker as "Background Worker"
    participant Email as "Email Provider"

    Note over EndUser,Email: UC-005 - Forgot Password Request

    EndUser->>APIGateway: POST /auth/forgot-password (email)
    APIGateway->>Auth: Forward (generic response)
    Auth->>DB: Find user by email (no existence leak)
    DB-->>Auth: user record or null
    Auth->>DB: insert PasswordResetToken (store token_hash, expiry) [if user exists]
    Auth->>Worker: enqueue send-reset-email job (include one-time token)
    Auth-->>APIGateway: 200 OK (generic: "If an account exists...")
    APIGateway-->>EndUser: 200 OK

    Worker->>Email: Send reset link (token) to email
    Email-->>Worker: delivery result
    Worker->>Auth: emit delivery status -> audit log
```

#### UC-006: Reset Password (via Emailed Token)
**Source**: [spec.md#UC-006](.propel/context/docs/spec.md#UC-006)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant DB as "User DB"
    participant Cache as "Redis"

    Note over EndUser,DB: UC-006 - Reset Password via Token

    EndUser->>APIGateway: POST /auth/reset-password (token, new_password)
    APIGateway->>Auth: Forward
    Auth->>DB: fetch PasswordResetToken by token_hash (compare hashed)
    alt token valid & not used & not expired
      Auth->>Auth: validate token hash & expiry
      Auth->>Auth: hash new password (Argon2id)
      Auth->>DB: update users.password_hash, clear/mark token used
      Auth->>Cache: invalidate sessions / add token_ids to blacklist (optional)
      Auth->>Auth: emit audit log (password reset)
      Auth-->>APIGateway: 200 OK
      APIGateway-->>EndUser: 200 OK
    else invalid/expired/used
      Auth-->>APIGateway: 400 Invalid or expired token
      APIGateway-->>EndUser: 400 Invalid or expired token
    end
```

#### UC-007: Logout / Token Revocation
**Source**: [spec.md#UC-007](.propel/context/docs/spec.md#UC-007)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant Redis as "Redis (blacklist)"

    Note over EndUser,Redis: UC-007 - Logout / Token Revocation

    EndUser->>APIGateway: POST /auth/logout (token / cookie)
    APIGateway->>Auth: Forward
    Auth->>Redis: add token_id to blacklist with expiry
    Auth->>Auth: emit audit log (logout)
    Auth-->>APIGateway: 200 OK
    APIGateway-->>EndUser: 200 OK

    par Subsequent request with revoked token
      Client->>APIGateway: request with token
      APIGateway->>Redis: check blacklist
      Redis-->>APIGateway: token found -> reject
      APIGateway-->>Client: 401 Unauthorized
    end
```

#### UC-008: Role-Based Access & Redirect
**Source**: [spec.md#UC-008](.propel/context/docs/spec.md#UC-008)

```mermaid
sequenceDiagram
    participant EndUser as "End User (Browser)"
    participant APIGateway as "API Gateway"
    participant Auth as "Authentication Service"
    participant DB as "User DB"
    participant Resource as "Protected Resource / App"

    Note over EndUser,Resource: UC-008 - Role-Based Redirect & Enforcement

    EndUser->>APIGateway: POST /auth/login (email,password)
    APIGateway->>Auth: Forward
    Auth->>DB: get user roles
    DB-->>Auth: roles (e.g., admin, user)
    Auth->>Auth: determine landing_path from roles
    Auth-->>APIGateway: 200 OK + token + landing_path
    APIGateway-->>EndUser: 200 OK + redirect to landing_path

    %% Accessing protected resource
    EndUser->>Resource: GET /admin/dashboard (with token)
    Resource->>Auth: optionally introspect token or validate signature
    Auth-->>Resource: token valid + claims (roles)
    alt role allowed
      Resource-->>EndUser: 200 OK (content)
    else role denied
      Resource-->>EndUser: 403 Forbidden
    end
```

## Previous Analysis and Reasoning
- The diagrams and entities above were derived directly from the provided Requirements Specification and align to the design choices: Argon2id hashing, KMS for keys, Redis for counters/blacklist, token_id for revocation, short JWT TTL, audit logging to SIEM, and optional adaptive detection (auditable). The ERD maps entities required for acceptance criteria (password reset single-use tokens, failed_attempts and locked_until, session/token records, roles, audit logs).

End of Design Model document.