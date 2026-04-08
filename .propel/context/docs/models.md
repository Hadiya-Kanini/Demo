## Design Modelling

## UML Models Overview
This document contains the complete set of UML visual models for the Authentication Service: system context, component view, deployment, data flows, logical data model (ERD), AI flow diagrams (risk scoring), and one sequence diagram per use case (UC-001..UC-004). Diagrams follow the architecture decisions: Argon2id for password hashing (FR-003), Redis for low-latency revocation/rate counters (TR-002), PostgreSQL as canonical store (TR-001), secret manager for signing keys (TR-004), and an optional ML Risk Scoring service with deterministic policy enforcement (AIR-*). Rationale: NFR drivers (latency, auditability, safety) require fast ephemeral state (Redis), durable canonical records (Postgres), strong adaptive hashing (Argon2id) and managed secrets (KMS/Secret Manager).

NFR-to-architecture decision mapping:
| NFR | Architectural Decision |
|-----|------------------------|
| NFR-001 (latency) | Use Go or low-latency runtime + Redis for token checks and caching; token creation local signing via Secret Manager keys |
| NFR-002 (availability) | Stateless auth service containers + autoscaling behind API Gateway, multi-AZ DB and Redis |
| NFR-003 (security) | Argon2id hashing, TLS 1.2+, secure cookies, Secret Manager for keys |
| NFR-005 (consistency) | Postgres as authoritative store for failed_attempts/locked_until; reconcile jobs for edge caches |
| AIR-* (AI requirements) | Separate Risk Scoring service behind AI Gateway; deterministic policy layer for final decisions |

## Architectural Views

### System Context Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

actor "End User (Browser/Mobile)" as User #add8e6
actor "Administrator" as Admin #add8e6

cloud "API Gateway / WAF" as APIGW #d3d3d3

package "Authentication System" {
  component "Auth Service (Stateless)" as Auth #90ee90
  database "User Store (Postgres)" as DB #ffffe0
  database "Token/Cache (Redis)" as REDIS #ffffe0
  component "Secret Manager / KMS" as Secrets #ffd27f
  component "Email Provider (SES/SendGrid)" as Email #d3d3d3
  component "Monitoring / Logging" as Mon #ffcc99
  component "Risk Scoring Service (AI) - Optional" as Risk #d3d3d3
}

User -> APIGW : TLS / HTTPS\nPOST /auth/*
APIGW --> Auth : Forward request\n(HTTPS)
Auth --> DB : Parameterized query\nSELECT / UPDATE user
Auth --> REDIS : GET/SET counters, revocation, TTL
Auth ..> Secrets : Retrieve signing keys / hashing params
Auth -> Email : Send reset email (async) 
Auth -> Mon : Emit audit event (PII pseudonymized)
Auth --> Risk : (optional) Risk scoring request (pseudonymized features)
Admin -> APIGW : HTTPS / Admin UI
APIGW -> Auth : Admin endpoints (unlock, audit)

' External boundaries
APIGW ..> "CDN / Edge WAF" : Optional edge filtering
@enduml
```

### Component Architecture Diagram
```mermaid
graph LR
  classDef actor fill:#add8e6
  classDef core fill:#90ee90
  classDef data fill:#ffffe0
  classDef external fill:#d3d3d3
  classDef infra fill:#ffd27f

  User[Client\n(Web/Mobile)]:::actor
  APIGW[API Gateway / WAF]:::external

  subgraph "Auth Platform"
    AuthAPI[HTTP API Layer\n(Handlers, Routing)]:::core
    Validator[Input Validator\n(email/password rules)]:::core
    Orchestrator[Auth Orchestrator\n(login/reset flows)]:::core
    Hasher[Hashing Service\n(Argon2id wrapper)]:::core
    TokenSvc[Token Service\n(JWT sign/opaque store)]:::core
    LockoutSvc[LockoutService\nfailed_attempts / policy]:::core
    RateLimiter[RateLimiter\nper-IP/account Redis]:::core
    Revocation[RevocationService\nchecks Redis+Postgres]:::core
    Audit[Audit Logger\nstructured events]:::core
    SecretsClient[Secrets Client\n(KMS/Secret Manager)]:::infra
    AdminAPI[Admin API\nunlock & audit]:::core
  end

  DB[(Postgres\nUser, Tokens, ResetTokens, AuditLog)]:::data
  REDIS[(Redis\ncounters, revocation)]:::data
  EmailSvc[Email Provider\n(SES/SendGrid)]:::external
  Monitoring[Monitoring & Logs\nPrometheus / ELK]:::infra
  RiskSvc[Risk Scoring Service\n(optional)]:::external

  User --> APIGW --> AuthAPI
  AuthAPI --> Validator
  AuthAPI --> Orchestrator
  Orchestrator --> Hasher : verify/hash
  Orchestrator --> TokenSvc : issue / validate / revoke
  Orchestrator --> LockoutSvc : read/update failed_attempts
  RateLimiter --> REDIS
  LockoutSvc --> DB
  TokenSvc --> REDIS
  Revocation --> REDIS
  Orchestrator --> Audit
  SecretsClient --> Hasher
  SecretsClient --> TokenSvc
  Orchestrator --> EmailSvc
  Audit --> Monitoring
  Orchestrator --> RiskSvc : (opt) risk_score request
  AdminAPI --> DB
```

### Deployment Architecture Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle
skinparam linetype ortho

cloud "Public Cloud Provider" as Cloud

node "Shared Services (Hub)" {
  component "Secret Manager / KMS" as KMS #ffd27f
  component "CI/CD & Image Registry" as CICD #ffd27f
  component "Monitoring (Prometheus, ELK)" as MON #ffcc99
}

package "VPC" {
  node "Public Subnet" {
    node "API Gateway / Load Balancer" as ALB #d3d3d3
  }

  node "Private Subnets (multi-AZ)" {
    folder "Auth Service Cluster\n(Autoscaling pods / Fargate tasks)" as AUTHCLUSTER #90ee90
    database "Postgres (managed, Multi-AZ)" as PG #ffffe0
    database "Redis (managed, Multi-AZ)" as RDS #ffffe0
    node "AI Subnet (optional)" {
      component "Risk Scoring Endpoint" as RISK #d3d3d3
    }
  }

  node "Operations Subnet" {
    node "Bastion / Admin Console" as BASTION #add8e6
  }
}

node "External Services" {
  component "Transactional Email Provider (SES)" as EMAIL #d3d3d3
  component "Model Registry / Managed ML" as MLR #d3d3d3
}

' Networking arrows
ALB --> AUTHCLUSTER : HTTPS (TLS 1.2+)\nHealth checks
AUTHCLUSTER --> PG : SQL / Parameterized queries (private)
AUTHCLUSTER --> RDS : Redis protocol (private)
AUTHCLUSTER --> KMS : HTTPS\nGet signing keys
AUTHCLUSTER --> MON : Metrics & logs
AUTHCLUSTER --> EMAIL : HTTPS / SMTP (outbound)
AUTHCLUSTER --> RISK : HTTPS (if enabled)
CICD --> AUTHCLUSTER : Deploy images (secure pipeline)
BASTION --> AUTHCLUSTER : Admin / debug (VPN/SSO)
PG --> "Backup Service" : Snapshot / PITR

@enduml
```

### Data Flow Diagram
```plantuml
@startuml
!define PROCESS rectangle
!define DATASTORE database
!define EXTERNAL component

left to right direction

EXTERNAL "Client (Browser/Mobile)" as client
PROCESS "API Gateway / WAF" as gateway
PROCESS "Auth Service\n(Login / Forgot / Reset)" as auth
DATASTORE "Postgres - User/Token/Reset/Audit" as pg
DATASTORE "Redis - Revocation / Counters" as redis
EXTERNAL "Email Provider\n(SES/SendGrid)" as email
EXTERNAL "Secret Manager / KMS" as kms
EXTERNAL "Risk Scoring Service (optional)" as risk
EXTERNAL "Monitoring / Event Stream" as stream

client -> gateway : POST /auth/login\nTLS
gateway -> auth : Forward request\n(HTTPS)
auth -> kms : Fetch signing key / hash params
auth -> pg : SELECT user by email
auth -> auth : Verify password via Hashing Service
auth -> redis : Check rate-limit / revocation
alt password valid & not locked
  auth -> redis : Reset failed_attempts
  auth -> pg : Write token session record (if opaque)
  auth -> redis : Store revocation TTL (if needed)
  auth -> stream : Emit audit event (login_success)
  auth -> client : 200 OK + token / redirect
else invalid credentials
  auth -> pg : Increment failed_attempts
  auth -> stream : Emit audit event (login_failed)
  opt lockout threshold reached
    auth -> pg : Set locked_until
    auth -> email : (optional) send lockout notice
  end
  auth -> client : 401 Invalid credentials / 423 Locked
end

' Forgot password flow
client -> gateway : POST /auth/forgot-password
gateway -> auth : Forward
auth -> pg : Lookup user (if exists)
auth -> pg : Insert ResetToken (single-use, TTL)
auth -> email : Send reset link (if user exists)
auth -> stream : Emit audit event (reset_requested)
auth -> client : 200 "If account exists..."

' Reset flow
client -> gateway : POST /auth/reset (token + new password)
gateway -> auth : Forward
auth -> pg : Validate ResetToken (single-use)
auth -> auth : Hash new password (Argon2id)
auth -> pg : Update user.password_hash; invalidate ResetToken
auth -> stream : Emit audit event (reset_completed)
auth -> client : 200 OK

' Risk scoring (optional)
auth -> risk : Request risk_score (pseudonymized features)
risk --> auth : risk_score, reason_codes
auth -> stream : Emit audit entry with model_version
@enduml
```

### Logical Data Model (ERD)
```mermaid
erDiagram
  USER {
    UUID id PK "uuid"
    string email "unique, indexed"
    string password_hash
    string hash_algo
    json hash_params
    int failed_attempts
    timestamp locked_until
    timestamp created_at
    timestamp updated_at
    timestamp last_login_at
  }
  TOKEN {
    UUID token_id PK
    UUID user_id FK
    string type "access|refresh|opaque"
    timestamp issued_at
    timestamp expires_at
    boolean revoked
    json metadata
  }
  RESET_TOKEN {
    UUID id PK
    UUID user_id FK
    string token "secure single-use"
    timestamp issued_at
    timestamp expires_at
    boolean used_flag
  }
  AUDIT_LOG {
    UUID event_id PK
    string event_type
    string user_id_or_hash
    string ip
    timestamp timestamp
    string outcome
    string correlation_id
    json metadata
  }
  ROLE {
    UUID role_id PK
    string name
    json permissions
  }
  USER_ROLE {
    UUID id PK
    UUID user_id FK
    UUID role_id FK
  }
  REVOCATION_RECORD {
    UUID id PK
    UUID token_id FK
    timestamp revoked_at
    string reason
  }
  RISK_ASSESSMENT {
    UUID id PK
    string user_id_or_hash
    string request_id
    float risk_score
    string reason_codes
    string model_version
    timestamp timestamp
  }

  USER ||--o{ TOKEN : "has"
  USER ||--o{ RESET_TOKEN : "has"
  USER ||--o{ AUDIT_LOG : "generates"
  USER ||--o{ USER_ROLE : "assigned"
  ROLE ||--o{ USER_ROLE : "maps"
  TOKEN ||--o{ REVOCATION_RECORD : "may have"
  USER ||--o{ RISK_ASSESSMENT : "may have"
```

## AI Architecture Diagrams

### Risk Scoring Pipeline (Mermaid flow)
```mermaid
flowchart LR
  classDef data fill:#ffffe0
  classDef core fill:#90ee90
  classDef external fill:#d3d3d3

  Events[Auth Events\n(Kafka/Kinesis)]:::data
  FE[Feature Engineering\nJobs/Materialization]:::core
  FS[Feature Store / Cache\n(Feast / Redis)]:::data
  TRAIN[Model Training\n(batch jobs)]:::core
  MODEL_REG[Model Registry / MLFlow]:::external
  SERVE[Model Serving\n(SageMaker / FastAPI)]:::external
  AI_GW[AI Gateway\n(timeout, cache, CB)]:::core
  APISVC[Auth Service\n(consumer)]:::core
  AUDIT[Model Telemetry & Audit]:::external

  Events --> FE --> FS
  FE --> TRAIN --> MODEL_REG
  MODEL_REG --> SERVE
  APISVC --> AI_GW --> SERVE
  SERVE --> APISVC
  SERVE --> AUDIT
  APISVC --> AUDIT
```

### AI Runtime Sequence (Mermaid sequence)
```mermaid
sequenceDiagram
    participant Client as Client
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant AI_GW as AI Gateway
    participant Risk as Risk Scoring
    participant Policy as Policy Engine
    participant DB as Postgres
    participant Audit as Audit Logger

    Note over Client,Auth: AI Runtime - Risk scoring used for step-up decisions

    Client->>APIGW: POST /auth/login (email, pwd)
    APIGW->>Auth: Forward request
    Auth->>DB: SELECT user by email
    DB-->>Auth: user record
    Auth->>AI_GW: Request risk_score (pseudonymized features)
    alt AI Gateway cache hit
      AI_GW-->>Auth: Cached risk_score
    else AI call
      AI_GW->>Risk: Score request
      Risk-->>AI_GW: risk_score, reason_codes, model_version
      AI_GW-->>Auth: risk_score, reason_codes
    end
    Auth->>Policy: Evaluate score -> action (allow, require_mfa, block)
    alt allow
      Policy-->>Auth: allow
      Auth-->>Client: 200 OK + token
    else require_mfa
      Policy-->>Auth: require_mfa
      Auth-->>Client: 200 Pending step-up (MFA/CAPTCHA)
    else block
      Policy-->>Auth: block
      Auth-->>Client: 403 Forbidden
    end
    Auth->>Audit: Emit audit with model_version & decision
```

## Use Case Sequence Diagrams

> Note: Sources point to spec.md anchors for traceability.

#### UC-001: Login with Email & Password
**Source**: [.propel/context/docs/spec.md#UC-001](.propel/context/docs/spec.md#UC-001)

```mermaid
sequenceDiagram
    participant EndUser as End User
    participant API as API Gateway
    participant Auth as Auth Service
    participant Validator as Input Validator
    participant DB as User Store (Postgres)
    participant Hasher as Hashing Service (Argon2id)
    participant Token as Token Service
    participant Redis as Redis (revocation/counters)
    participant Secrets as Secret Manager
    participant Risk as Risk Scoring Service
    participant Audit as Audit Logger

    Note over EndUser,Token: UC-001 - Login with Email & Password

    EndUser->>API: POST /auth/login {email,password}
    API->>Auth: Forward request
    Auth->>Validator: Validate inputs
    alt Validation fails
      Validator-->>Auth: 400 validation.error
      Auth-->>API: 400 validation.error
      API-->>EndUser: 400 validation.error
    else Validation succeeds
      Validator-->>Auth: OK
      Auth->>DB: SELECT user by email
      DB-->>Auth: user record
      alt account locked (locked_until > now)
        Auth-->>API: 423 Account locked
        API-->>EndUser: 423 Account locked
        Auth->>Audit: log lockout_attempt
      else proceed
        Auth->>Secrets: Get hashing params / signing key
        Secrets-->>Auth: params / key
        Auth->>Hasher: Verify password (Argon2id)
        Hasher-->>Auth: match / no-match
        alt match (success)
          Auth->>Redis: Reset failed_attempts for user
          Auth->>Token: Issue access token (exp 30m)
          Token-->>Auth: token + role
          Auth->>Audit: log login_success
          Auth-->>API: 200 {token, role, redirect}
          API-->>EndUser: 200 {token, redirect}
        else no-match (invalid credentials)
          Auth->>DB: Increment failed_attempts
          Auth->>Redis: Increment per-IP counters
          alt threshold reached (failed_attempts >=5)
            Auth->>DB: Set locked_until = now + lock_duration
            Auth->>Audit: log lockout
            Auth-->>API: 423 Account locked
            API-->>EndUser: 423 Account locked
          else below threshold
            Auth->>Audit: log login_failed
            Auth-->>API: 401 Invalid credentials
            API-->>EndUser: 401 Invalid credentials
          end
        end
      end
    end
```

#### UC-002: Forgot Password / Reset Flow
**Source**: [.propel/context/docs/spec.md#UC-002](.propel/context/docs/spec.md#UC-002)

```mermaid
sequenceDiagram
    participant EndUser as End User
    participant API as API Gateway
    participant Auth as Auth Service
    participant DB as User Store (Postgres)
    participant Reset as ResetToken store (DB)
    participant TokenGen as Token Generator
    participant Email as Email Provider
    participant Audit as Audit Logger

    Note over EndUser,Email: UC-002 - Forgot Password / Reset Flow

    EndUser->>API: POST /auth/forgot-password {email}
    API->>Auth: Forward request
    Auth->>DB: SELECT user by email
    alt user exists
      DB-->>Auth: user record
      Auth->>TokenGen: Create ResetToken (single-use, TTL 15m)
      TokenGen-->>Auth: reset_token
      Auth->>Reset: Insert ResetToken record
      Auth->>Email: Send reset link (async)
      Auth->>Audit: log reset_requested (pseudonymized)
      Auth-->>API: 200 "If account exists..."
      API-->>EndUser: 200 "If account exists..."
    else user not found
      DB-->>Auth: no record
      Auth->>Audit: log reset_requested (hashed email)
      Auth-->>API: 200 "If account exists..."
      API-->>EndUser: 200 "If account exists..."
    end

    %% Reset usage
    EndUser->>API: POST /auth/reset {token, new_password}
    API->>Auth: Forward request
    Auth->>Reset: Validate token (exists && not used && not expired)
    alt invalid token
      Reset-->>Auth: invalid/expired
      Auth->>Audit: log reset_failed
      Auth-->>API: 400 Invalid or expired token
      API-->>EndUser: 400 Invalid or expired token
    else valid token
      Reset-->>Auth: valid + user_id
      Auth->>TokenGen: Hash new password (Argon2id)
      TokenGen-->>Auth: new_hash
      Auth->>DB: Update user.password_hash, reset failed_attempts
      Auth->>Reset: Mark token used
      Auth->>Audit: log reset_completed
      Auth-->>API: 200 Password reset complete
      API-->>EndUser: 200 Password reset complete
    end
```

#### UC-003: Account Lockout Handling
**Source**: [.propel/context/docs/spec.md#UC-003](.propel/context/docs/spec.md#UC-003)

```mermaid
sequenceDiagram
    participant EndUser as End User
    participant API as API Gateway
    participant Auth as Auth Service
    participant DB as User Store (Postgres)
    participant Email as Email Provider
    participant Admin as Administrator
    participant AdminAPI as Admin API
    participant Audit as Audit Logger
    participant Redis as Redis

    Note over EndUser,Admin: UC-003 - Account Lockout Handling

    EndUser->>API: POST /auth/login {email,password}
    API->>Auth: Forward request
    Auth->>DB: SELECT user
    Auth->>Redis: Increment per-account / per-IP counters
    alt failed attempts threshold reached
      Auth->>DB: Set locked_until = now + lock_duration
      Auth->>Email: (optional) send lockout notification
      Auth->>Audit: log account_locked
      Auth-->>API: 423 Account is locked
      API-->>EndUser: 423 Account is locked
    else normal processing
      Auth-->>API: continue login flow
    end

    Admin->>AdminAPI: POST /admin/users/{id}/unlock
    AdminAPI->>Auth: Authenticated admin request
    Auth->>DB: Verify admin role, set locked_until = NULL, reset failed_attempts
    Auth->>Audit: log admin_unlock with admin id
    Auth-->>AdminAPI: 200 Unlocked
    AdminAPI-->>Admin: 200 Unlocked
```

#### UC-004: Role-Based Authorization Enforcement & Redirect
**Source**: [.propel/context/docs/spec.md#UC-004](.propel/context/docs/spec.md#UC-004)

```mermaid
sequenceDiagram
    participant EndUser as End User
    participant Client as Client App
    participant API as API Gateway
    participant Auth as Auth Service
    participant TokenMiddleware as Token Validation Middleware
    participant AppSvc as App Services (Resource Server)
    participant DB as User Store (for role mapping)
    participant Audit as Audit Logger

    Note over EndUser,AppSvc: UC-004 - Role-Based Authorization & Redirect

    EndUser->>Client: User completes login (has token)
    Client->>API: GET /post-login-target with token
    API->>Auth: Validate token or forward to TokenMiddleware
    TokenMiddleware->>Auth: Verify signature / revocation
    alt token invalid or expired
      Auth-->>API: 401 Unauthorized
      API-->>Client: 401 Please re-authenticate
    else token valid
      Auth->>DB: (optional) fetch roles if not in token
      DB-->>Auth: role(s)
      Auth-->>API: 200 {redirect_url}
      API-->>Client: 200 {redirect_url}
      Client->>AppSvc: GET /dashboard with token
      AppSvc->>Auth: Validate token / role (TokenMiddleware)
      alt role allowed
        AppSvc-->>Client: 200 Dashboard content
      else role forbidden
        AppSvc-->>Client: 403 Forbidden
        AppSvc->>Audit: log authorization_denied
      end
    end
```

## Notes & Traceability
- All ERD entities correspond to definitions in design.md (User, Token, ResetToken, AuditLog, Role, etc.).
- Sequence diagrams include alternative paths (validation failure, invalid credentials, lockout) and optional AI interactions (UC-001 shows AI optional path via separate AI sequence diagram).
- Secret retrievals (signing keys, hashing params) use Secret Manager (KMS) and are shown where required.

## Change / Extension Guidance
- To enable refresh-token rotation (FR-011), extend Token entity with rotation_id and add rotation logic in Token Service; update sequence diagrams to include refresh rotation step.
- To enable AI enforcement in production, deploy Risk Scoring with canary (MODEL_REG) and enable AI Gateway circuit-breaker; policy remains deterministic and must be auditable (AIR-004 / AIR-005).

