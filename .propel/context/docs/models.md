---
description: UML architectural diagrams and sequence diagrams for the system generated from spec.md and design.md
---

# Design Modelling

## UML Models Overview
This document contains the UML visual models for the Email/Password Authentication Service: system context, component view, deployment, data flows, logical data model (ERD), AI risk-scoring pipeline, and one sequence diagram per use case (UC-001..UC-004). Diagrams follow architecture decisions: Argon2id for password hashing, Redis for low-latency revocation and counters, PostgreSQL as canonical store, Secret Manager for signing keys, and an optional Risk Scoring service (AIR-*). Rationale (context → decision → benefit):
- NFR drivers: latency, availability, security, auditability.
- Decision: stateless Auth Service + Redis caching + Postgres canonical store + Secret Manager + optional ML risk scorer.
- Benefit: low-latency auth checks, durable audit trail, secure secrets handling, safe AI integration with deterministic enforcement.

NFR-to-architecture decision mapping:
| NFR | Architectural Decision |
|-----|------------------------|
| NFR-001 (latency) | Go or low-latency runtime + Redis for token checks and caching; local signing via Secret Manager keys |
| NFR-002 (availability) | Stateless auth containers + autoscaling, multi-AZ DB and Redis |
| NFR-003 (security) | Argon2id hashing, TLS 1.2+, secure cookies, Secret Manager for keys |
| NFR-005 (consistency) | Postgres as authoritative store for failed_attempts/locked_until; reconcile jobs for caches |
| AIR-* (AI requirements) | Isolated Risk Scoring service + deterministic policy layer and circuit-breaker

## Architectural Views

### System Context Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle
skinparam linetype ortho

actor "End User (Browser/Mobile)" as User #add8e6
actor "Administrator" as Admin #add8e6

cloud "API Gateway / WAF" as APIGW #d3d3d3

package "Authentication System" {
  component "Auth Service (Stateless)\n(API handlers, Orchestrator)" as Auth #90ee90
  database "User Store (Postgres)\n(User, Token, ResetToken, AuditLog)" as DB #ffffe0
  database "Token/Cache (Redis)\n(revocation, rate counters)" as REDIS #ffffe0
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
Auth -> Mon : Emit audit event (pseudonymized)
Auth --> Risk : (optional) Risk scoring request (pseudonymized features)
Admin -> APIGW : HTTPS / Admin UI
APIGW -> Auth : Admin endpoints (unlock, audit)

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

  Client[Client\n(Web/Mobile)]:::actor
  APIGW[API Gateway / WAF]:::external

  subgraph "Auth Platform"
    AuthAPI[HTTP API Layer\n(Handlers, Routing)]:::core
    Validator[Input Validator\n(email/password rules)]:::core
    Orchestrator[Auth Orchestrator\n(login/reset flows)]:::core
    Hasher[Hashing Service\n(Argon2id wrapper)]:::core
    TokenSvc[Token Service\n(JWT sign / opaque)]:::core
    LockoutSvc[Lockout Service\n(failed_attempts, locked_until)]:::core
    RateLimiter[RateLimiter\n(Per-IP/account Redis)]:::core
    Revocation[RevocationService\n(Redis + Postgres)]:::core
    Audit[Audit Logger\n(structured events)]:::core
    SecretsClient[Secrets Client\n(KMS/Secret Manager)]:::infra
    AdminAPI[Admin API\n(unlock & audit)]:::core
  end

  DB[(Postgres\nUser, Tokens, ResetTokens, AuditLog)]:::data
  REDIS[(Redis\ncounters, revocation)]:::data
  EmailSvc[Email Provider\n(SES/SendGrid)]:::external
  Monitoring[Monitoring & Logs\nPrometheus / ELK]:::infra
  RiskSvc[Risk Scoring Service\n(optional)]:::external

  Client --> APIGW --> AuthAPI
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
    component "CDN / WAF / Edge" as EDGE #d3d3d3
    component "API Gateway / ALB" as ALB #d3d3d3
  }

  node "Private Subnets (Multi-AZ)" {
    node "Auth Service Cluster (Autoscaled)" {
      component "Auth Container Instance 1" as AUTH1 #90ee90
      component "Auth Container Instance 2" as AUTH2 #90ee90
      component "Auth Container Instance N" as AUTHN #90ee90
    }

    node "Data Plane" {
      database "Postgres (Managed Multi-AZ)" as PG #ffffe0
      database "Redis (Managed cluster)" as RDS #ffffe0
    }

    node "AI / Model Serving (Optional)" {
      component "Risk Scoring Endpoint" as RISK #d3d3d3
    }
  }

  node "Management Subnet" {
    component "Admin Console (restricted)" as ADMIN #add8e6
    component "Backup & Scheduler" as BACKUP #ffd27f
  }
}

EDGE --> ALB : HTTPS
ALB --> AUTH1 : HTTPS
ALB --> AUTH2 : HTTPS
ALB --> AUTHN : HTTPS

AUTH1 --> PG : Parameterized SQL
AUTH1 --> RDS : GET/SET counters, revocation
AUTH1 ..> KMS : Retrieve signing keys / secrets
AUTH1 --> MON : Metrics & logs
AUTH1 --> "Email Provider (External)" : SMTP/HTTPS
AUTH1 --> RISK : HTTPS (optional)
BACKUP --> PG : backup / PITR

@enduml
```

### Data Flow Diagram
```plantuml
@startuml
!define PROCESS rectangle
!define DATASTORE database
!define EXTERNAL component

left to right direction
EXTERNAL "Client (Browser / Mobile)" as client
PROCESS "Auth API\n(Validate & Orchestrate)" as auth
PROCESS "Hashing Module\n(Argon2id verifier/rehash)" as hash
PROCESS "Token Service\n(issue/verify tokens)" as token
DATASTORE "Postgres\n(User, Token, ResetToken, AuditLog)" as db
DATASTORE "Redis\n(revocation, counters, short cache)" as redis
EXTERNAL "Secret Manager / KMS" as kms
EXTERNAL "Email Provider" as email
EXTERNAL "Risk Scoring Service (optional)" as risk
EXTERNAL "Monitoring / Logging" as mon

client -> auth : POST /auth/login\nemail,password
auth -> db : SELECT user by email
db --> auth : user row
auth -> hash : Verify password with hash params
hash --> auth : match / no match
opt password match
  auth -> risk : (optional) request risk score\n(pseudonymized features)
  risk --> auth : risk_score
  alt low risk
    auth -> token : Sign token (get key)
    token -> kms : Get signing key (or fetch cached)
    kms --> token : signing key
    token --> auth : access token + refresh token metadata
    auth -> redis : write revocation/TTL for refresh token
    auth -> db : persist refresh token metadata (if used)
    auth --> client : 200 OK + token
  else step-up / high risk
    auth --> client : 401 + challenge (MFA/CAPTCHA)
  end
else password mismatch
  auth -> db : increment failed_attempts
  auth -> redis : increment IP counter
  alt lockout threshold reached
    auth -> db : set locked_until
    auth --> client : 423 Account locked
  else
    auth --> client : 401 Invalid credentials
  end
end

client -> auth : POST /auth/forgot-password\nemail
auth -> db : lookup email (if exists)
alt exists
  auth -> db : create ResetToken
  auth -> email : send reset link
end
auth --> client : 200 (non-revealing)

auth -> mon : emit audit events
mon -> db : (optional) archive / indexing
@enduml
```

### Logical Data Model (ERD)
```mermaid
erDiagram
  USER {
    UUID id PK "Primary key"
    string email "unique"
    string password_hash
    string hash_algo
    json hash_params
    int failed_attempts
    timestamp locked_until
    timestamp last_login_at
    timestamp created_at
    timestamp updated_at
  }

  ROLE {
    UUID id PK
    string name "unique"
    json permissions
  }

  USER_ROLE {
    UUID id PK
    UUID user_id FK
    UUID role_id FK
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
    string token "secure single-use token"
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

  REVOCATION {
    UUID revocation_id PK
    UUID token_id FK
    timestamp revoked_at
    string revoked_by
    string reason
  }

  USER ||--o{ TOKEN : "has"
  USER ||--o{ RESET_TOKEN : "owns"
  USER ||--o{ AUDIT_LOG : "emits"
  USER ||--o{ USER_ROLE : "assigned"
  ROLE ||--o{ USER_ROLE : "maps to"
  TOKEN ||--o{ REVOCATION : "may have"
```

## AI Architecture Diagrams

### Risk Scoring Pipeline (Mermaid flow)
```mermaid
graph LR
  classDef external fill:#d3d3d3
  classDef infra fill:#ffd27f
  classDef core fill:#90ee90

  Auth[Auth Service\n(Event source)]:::core
  Stream[Event Stream\n(Kafka/Kinesis)]:::infra
  FS[Feature Store / Cache\n(Redis/Feast)]:::infra
  ETL[ETL / Feature Jobs]:::infra
  Train[Training Pipeline\n(Batch)]:::infra
  Registry[Model Registry / MLFlow]:::infra
  ModelServe[Model Server\n(Endpoint)]:::infra
  AIGW[AI Gateway\n(timeout, CB, cache)]:::infra
  Telemetry[Model Telemetry & Explainability]:::infra
  Audit[Audit Store / DB]:::infra

  Auth -->|publish events| Stream
  Stream --> ETL
  ETL --> FS
  ETL --> Train
  Train --> Registry
  Registry --> ModelServe
  Auth -->|scoring request| AIGW
  AIGW --> ModelServe
  ModelServe -->|score+meta| AIGW
  AIGW --> Auth
  ModelServe --> Telemetry
  Auth --> Audit
  Telemetry --> Registry
```

### AI Sequence Diagram — Risk-scored Login (Mermaid)
```mermaid
sequenceDiagram
    participant User as End User
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant DB as Postgres
    participant Redis as Redis
    participant Secrets as Secret Manager
    participant Risk as Risk Scoring
    participant Token as Token Service

    Note over User,Token: UC-001 (AI path) - Login with Risk Scoring

    User->>APIGW: POST /auth/login (email,password)
    APIGW->>Auth: Forward request
    Auth->>DB: SELECT user by email
    DB-->>Auth: user row
    Auth->>Secrets: Fetch hash params & signing key
    Secrets-->>Auth: params, key meta
    Auth->>Auth: Verify password (Argon2id)
    alt password valid and not locked
      Auth->>Redis: read failed_attempts, ip counters
      Auth->>Risk: Request risk score (pseudonymized features)
      Risk-->>Auth: {score, reason_codes, model_version}
      alt score >= threshold_high
        Auth-->>User: 401 + step-up challenge (MFA/CAPTCHA)
      else score < threshold_high
        Auth->>Token: Issue access & refresh tokens (sign)
        Token-->>Auth: tokens
        Auth-->>User: 200 OK + token + role
      end
    else invalid credentials
      Auth->>DB: increment failed_attempts
      Auth->>Redis: increment IP counter
      Auth-->>User: 401 Invalid credentials
    end
```

## Use Case Sequence Diagrams

### UC-001: Login with Email & Password
**Source**: spec.md#UC-001

```mermaid
sequenceDiagram
    participant User as End User
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant DB as User Store (Postgres)
    participant Redis as Redis
    participant Secrets as Secret Manager
    participant Token as Token Service
    participant Risk as Risk Scoring (optional)

    Note over User,DB: UC-001 - Login with Email & Password

    User->>APIGW: POST /auth/login (email,password)
    APIGW->>Auth: Forward request (HTTPS)
    Auth->>DB: SELECT * FROM users WHERE email=?
    DB-->>Auth: user row / not found
    Auth->>Secrets: Get hashing params / signing key
    Secrets-->>Auth: hash params / key meta
    Auth->>Auth: Verify password (Argon2id)
    alt password valid and account not locked
        Auth->>Redis: RESET failed_attempts for user/ip
        opt Risk Scoring enabled
            Auth->>Risk: POST /score (pseudonymized features)
            Risk-->>Auth: {risk_score, reason_codes}
            alt high risk
                Auth-->>User: 401 Challenge (MFA/CAPTCHA)
                Note right of Auth: Deterministic policy enforces step-up
            else low/ok risk
                Auth->>Token: Issue tokens (sign with key)
                Token-->>Auth: access_token / refresh_token
                Auth->>DB: persist refresh metadata (if used)
                Auth-->>APIGW: 200 + token + role+redirect
                APIGW-->>User: 200 OK + token
            end
        end
    else invalid credentials or user not found
        Auth->>DB: increment failed_attempts (if user exists)
        Auth->>Redis: increment IP counter
        alt failed_attempts >= threshold
            Auth->>DB: set locked_until = now + lock_duration
            Auth-->>User: 423 Account locked
        else
            Auth-->>User: 401 Invalid credentials
        end
    end
    Auth->>Monitoring: emit audit event (pseudonymized)
```

### UC-002: Forgot Password / Reset Flow
**Source**: spec.md#UC-002

```mermaid
sequenceDiagram
    participant User as End User
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant DB as User Store (Postgres)
    participant Reset as ResetToken Store
    participant Email as Email Provider

    Note over User,Email: UC-002 - Forgot Password / Reset

    User->>APIGW: POST /auth/forgot-password (email)
    APIGW->>Auth: Forward request
    Auth->>DB: SELECT user by email
    alt user exists
        Auth->>Reset: CREATE reset_token (single-use, TTL)
        Reset-->>Auth: token created
        Auth->>Email: send reset link (token)
        Email-->>Auth: 202 Accepted
    end
    Auth-->>APIGW: 200 OK (non-revealing)
    APIGW-->>User: 200 "If an account exists, you will receive reset instructions"

    %% Reset usage
    User->>APIGW: POST /auth/reset (token, new_password)
    APIGW->>Auth: Forward
    Auth->>Reset: validate token (exists, not used, not expired)
    alt valid token
        Auth->>Auth: validate new password policy
        Auth->>Auth: hash new password (Argon2id)
        Auth->>DB: UPDATE users SET password_hash..., failed_attempts=0
        Auth->>Reset: mark token used
        Auth-->>User: 200 Password updated
        Auth->>Monitoring: emit audit event
    else invalid/expired
        Auth-->>User: 400 Invalid or expired token
    end
```

### UC-003: Account Lockout Handling (includes Admin Unlock)
**Source**: spec.md#UC-003

```mermaid
sequenceDiagram
    participant User as End User
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant DB as User Store (Postgres)
    participant Redis as Redis
    participant Admin as Administrator

    Note over User,Admin: UC-003 - Account Lockout Handling

    %% Failed attempts flow (invoked by login failures)
    User->>APIGW: POST /auth/login (bad credentials)
    APIGW->>Auth: Forward
    Auth->>DB: increment failed_attempts
    Auth->>Redis: increment IP counter
    alt failed_attempts < threshold
        Auth-->>User: 401 Invalid credentials
    else failed_attempts >= threshold
        Auth->>DB: set locked_until = now + lock_duration
        Auth->>Redis: mark lock
        Auth->>Email: optional notify user (non-revealing)
        Auth-->>User: 423 Account locked
        Auth->>Monitoring: emit lockout audit event
    end

    %% Admin unlock
    Admin->>APIGW: POST /admin/users/{id}/unlock
    APIGW->>Auth: Forward (admin auth)
    Auth->>DB: verify admin role
    alt admin authorized
        Auth->>DB: set failed_attempts=0; locked_until=NULL
        Auth-->>Admin: 200 Unlocked
        Auth->>Monitoring: emit admin unlock audit event
    else
        Auth-->>Admin: 403 Forbidden
    end

    %% Auto-unlock on expiry (background)
    loop periodic checks
      Auth->>DB: SELECT users WHERE locked_until <= now
      DB-->>Auth: list unlocked_users
      alt entries exist
        Auth->>DB: reset failed_attempts, locked_until=NULL
        Auth->>Monitoring: emit unlock audit event
      end
    end
```

### UC-004: Role-Based Authorization Enforcement & Redirect
**Source**: spec.md#UC-004

```mermaid
sequenceDiagram
    participant User as End User
    participant Client as Client App
    participant APIGW as API Gateway
    participant Auth as Auth Service
    participant TokenSvc as Token Service
    participant AppSvc as Application Service
    participant DB as User Store (Postgres)

    Note over User,AppSvc: UC-004 - Role-Based Authorization & Redirect

    %% Post-login redirect decision
    User->>Client: interacts with login page
    Client->>APIGW: POST /auth/login
    APIGW->>Auth: Forward
    Auth->>DB: SELECT role mapping for user
    Auth->>TokenSvc: Issue token including role claim
    TokenSvc-->>Auth: token
    Auth-->>Client: 200 {token, role, redirect_url}
    Client->>User: redirect to role dashboard (redirect_url)

    %% Protected resource access
    User->>AppSvc: GET /protected/resource (with token)
    AppSvc->>TokenSvc: validate token signature & expiry
    TokenSvc-->>AppSvc: {valid, claims}
    alt token valid and role in claims allowed
      AppSvc-->>User: 200 resource
      AppSvc->>Monitoring: emit auth success event
    else invalid token or insufficient role
      AppSvc-->>User: 401/403 Unauthorized
      AppSvc->>Monitoring: emit auth failure event
    end
```

---