# Architecture Design

## Project Overview
Build a secure, reliable email/password authentication service that enables registered users to authenticate, obtain time-limited tokens, and be routed to role-specific dashboards. Target users are end users (customers, employees, admins) and administrators/operators. High-level capabilities: secure password hashing, token issuance and revocation, account lockout and rate-limiting, non-revealing reset flows, structured audit logging, and an extensible hook for adaptive risk scoring (phase 2).

## Architecture Goals
- Goal 1: Provide deterministic, auditable authentication flows with measurable SLAs for latency and availability.
- Goal 2: Protect password material and signing keys using industry-best adaptive hashing and secret management.
- Goal 3: Prevent credential abuse via layered rate-limiting, account lockout, and monitoring/alerting.
- Goal 4: Enable safe, incremental adoption of adaptive risk-based step-up authentication (AI-assisted) without compromising determinism and auditability.
- Goal 5: Support horizontal scaling and operational runbooks for maintainability and incident response.

## Non-Functional Requirements
- NFR-001: System MUST respond to authentication API requests (POST /auth/login) with median latency ≤ 300 ms and p95 ≤ 800 ms under normal production load. Measurement: production telemetry and CI load tests (synthetic workloads).
- NFR-002: System MUST target availability of authentication endpoints ≥ 99.95% (monthly). Measurement: uptime monitoring, SLO/SLA dashboards. [UNCLEAR] if stricter SLA required.
- NFR-003: System MUST enforce TLS 1.2+ for all network traffic, configure cookies with HttpOnly and Secure, and set SameSite=Strict (or Lax where platform requires). Measurement: automated security scans and HTTP header verification.
- NFR-004: System MUST scale horizontally: service instances stateless, autoscaling policies configured to handle configurable peaks (support design for tens of thousands concurrent sessions). Measurement: autoscaling tests and target throughput (requests/sec) defined in capacity plan. [UNCLEAR] exact concurrent users/peak RPS to size capacity.
- NFR-005: System MUST ensure deterministic correctness for lockout and token revocation: failed_attempts counters, locked_until timestamps, and revocations MUST be durable and consistent (no more than 1% inconsistency window under partition). Measurement: transactional tests, consistency checks, and reconciliation jobs. [UNCLEAR] acceptable inconsistency tolerance to confirm.
- NFR-006: System MUST store secrets (signing keys, Argon2 parameters, provider credentials) in a managed secret store and rotate keys regularly. Measurement: no credentials in code, secret access audit logs, rotation schedule enforced.
- NFR-007: System MUST redact PII in logs and maintain documented log retention policies. Measurement: log audits showing hashed/pseudonymized identifiers; retention enforcement. [UNCLEAR] exact retention windows (days/years).
- NFR-008: System MUST provide operational documentation and runbooks (hashing tuning, migration, rollback), and CI gates for unit/integration tests and security scans. Measurement: presence of runbooks and CI enforcement on PRs.

## Data Requirements
- DR-001: System MUST store user records with fields: id (UUID), email (unique, indexed), password_hash, hash_algo, hash_params (JSON), failed_attempts (int), locked_until (timestamp nullable), roles, created_at, updated_at. Rationale: authentication, lockout, role-based routing.
- DR-002: System MUST store token/session records (for opaque tokens or revocation audit): token_id (UUID), user_id, issued_at, expires_at, revoked (bool), metadata (ip, user_agent). Rationale: revocation and auditability.
- DR-003: System MUST store reset tokens/OTPs with single-use enforcement: token, user_id, issued_at, expires_at, used_flag. Rationale: secure forgot-password flow.
- DR-004: System MUST emit audit logs for auth events with schema: event_type, user_id_or_hash, ip, timestamp, outcome, correlation_id, metadata. Logs MUST not contain plaintext passwords or tokens. Rationale: incident response and compliance.
- DR-005: System MUST implement backups and retention: daily DB backups with PITR and documented retention policy. Measurement: successful backup jobs and restore drills. [UNCLEAR] exact retention duration to confirm with compliance.
- DR-006: System MUST support schema migrations and a migration plan (including rehash-on-login for legacy weak hashes). [UNCLEAR] scope of legacy hashes and bulk migration strategy to confirm.

## AI Consideration

Status: Applicable

Rationale: Feature FR-014 (Adaptive risk-based step-up auth) is marked [AI-CANDIDATE] and requires a risk scoring capability. The architecture will adopt a hybrid approach where deterministic rules enforce final actions and an AI/ML risk scoring service provides risk signals/recommendations.

## AI Requirements
- AIR-001: Risk Scoring (Functional) — System MUST provide a Risk Scoring service that returns a numeric risk_score (0.0–1.0), reason_codes, and model_version for each auth attempt within p95 ≤ 200 ms. Inputs: pseudonymized user id, IP, device fingerprint, failed_attempts, recent auth events, geo, user agent. Traces to: NFR-001, NFR-005, DR-004.
- AIR-002: Model Quality — Risk model MUST achieve discrimination AUC ≥ 0.85 on validated datasets and have documented false positive/negative tolerances. Measurement: model evaluation reports and periodic revalidation. Traces to: NFR-005, NFR-008. [UNCLEAR] exact FP/FN thresholds (PO to define).
- AIR-003: Safety & Privacy — System MUST redact PII from model inputs where feasible, encrypt stored features and logs, and restrict access to model telemetry. Traces to: NFR-007, DR-004.
- AIR-004: Operational Resilience — System MUST support model versioning, canary deploys, and immediate rollback within 15 minutes. A circuit breaker MUST fallback to deterministic conservative policy (e.g., require step-up for medium/high risk) if model latency or error rates exceed thresholds. Traces to: NFR-002, NFR-006.
- AIR-005: Auditability — Model inputs, outputs (scores), decisions (enforced action) and model_version MUST be stored in audit logs (pseudonymized) for at least the configured retention window and be access-controlled. Traces to: DR-004, NFR-007.
- AIR-006: Throughput & Budgeting — System MUST support batching, caching, or local feature lookup to avoid exceeding operational budgets during spikes; p95 inference latency ≤ 200 ms. Traces to: NFR-001, NFR-004.

### AI Architecture Pattern
Selected Pattern: Hybrid (ML risk scoring service + deterministic enforcement)

Rationale:
- Context: Adaptive step-up requires probabilistic scoring but final auth decisions must be deterministic and auditable.
- Decision: Use a separate ML Risk Scoring service that returns explainable scores; deterministic policy layer consumes score and enforces MFA/CAPTCHA or denies.
- Benefit: Enables experimentation with models while preserving deterministic enforcement, audit trails, and the ability to rollback safely.

## Technology Stack
| Layer | Technology | Version | Justification (NFR/DR/AIR) |
|-------|------------|---------|----------------------------|
| Frontend | React (or existing client) | N/A | Client-side validation and secure storage guidance; supports NFR-001, FR-002 |
| Backend | Go (recommended) or FastAPI (Python) / Node.js (Fastify) | Go 1.20+ or Python 3.10+ / Node 18+ | Chosen for low-latency backends to meet NFR-001 and NFR-004. Trace: NFR-001, NFR-004 |
| Database | PostgreSQL (managed) | 12+ | ACID, migrations, indexing to support DR-001, DR-002, NFR-005 |
| Token/Cache Store | Redis (managed, with AOF/RDB) | 6+ | Low-latency revocation and TTL semantics (TR-002) trace: DR-002, NFR-001 |
| Secret Manager | Cloud Secret Manager (AWS Secrets Manager / Azure Key Vault) | managed | Secure secrets storage and rotation (NFR-006) trace: NFR-006 |
| Email Provider | SES / Postmark / SendGrid | managed | Transactional email for FR-007; security & deliverability (TR-008) trace: FR-007 |
| Rate-limiting / WAF | Cloud WAF (Cloudflare / API Gateway) + in-app limiter | N/A | Edge protection + per-account counters (TR-005) trace: NFR-008 |
| Logging & Monitoring | Prometheus + Grafana (metrics), ELK / OpenSearch (logs) | latest stable | Metrics and centralized logs for DR-004 and NFR-007 |
| CI/CD & Security Scans | GitHub Actions + SAST/Dependency scanning (Snyk/pip-audit) | N/A | Enforce NFR-008, TR-006 |
| Container Runtime / Orchestration | Docker + Managed compute (Fargate / Cloud Run) or Kubernetes (EKS/GKE) | N/A | Deployability and autoscaling to meet NFR-002 / NFR-004 |
| AI/ML (if AIR present) | Managed model hosting (SageMaker / Azure ML) or self-hosted model server (FastAPI/TorchServe) | N/A | Hosts risk models, supports AIR-001/AIR-004 |
| Feature/Event Stream | Kafka / Kinesis / PubSub | N/A | Feature capture for model training and telemetry (AIR) |
| Feature Store / Cache | Redis / Feast / PostgreSQL (with pgvector if embeddings used) | N/A | Fast feature lookup and training datasets (AIR) |

### Alternative Technology Options
- Database: DynamoDB (NoSQL) — considered for extreme scale, but adds complexity for relational constraints; de-selected in favor of PostgreSQL for transactional guarantees (DR-001).
- Token store: Postgres-only revocation table — durable but higher latency under heavy loads; combined Redis + Postgres selected.
- Secret store: HashiCorp Vault — stronger multi-cloud features but higher operational overhead; choose cloud secret manager if single-cloud hosting.
- Backend runtime: Choose team familiarity (Python/Node) if delivery speed prioritized over lowest-latency Go option. [UNCLEAR] team preferred language.

### Technology Stack Validation
- Security (NFR-003, NFR-006): Secret Manager + TLS + OWASP-guided libraries validated; parameterized DB access prevents injection.
- Performance (NFR-001): Go or highly-optimized frameworks plus Redis for token ops ensures median ≤ 300 ms in benchmarks; CI performance tests required.
- Scalability (NFR-004): Stateless services + managed DB and Redis with autoscaling meet horizontal scaling requirement; capacity plan pending exact RPS.
- Data (DR-001..DR-004): PostgreSQL schema supports required relational data; Redis provides low-latency token revocation; logs stored in ELK meet audit requirement.
- AI (AIRs): Managed model hosting with feature capture pipeline supports p95 ≤ 200 ms requirement and versioning, with circuit breaker fallback.

### Technology Decision
| Metric (from NFR/DR/AIR) | Candidate 1 | Candidate 2 | Rationale |
|--------------------------|-------------|-------------|-----------|
| Primary DB (security & ACID) | PostgreSQL (selected) | DynamoDB | PostgreSQL provides ACID, mature migrations and easier joins for role checks; DynamoDB scales but complicates relational queries |
| Token store (latency) | Redis (selected) | Postgres revocation table | Redis offers <1ms reads for revocation and TTL semantics; Postgres is durable but higher latency under heavy throughput |
| Backend runtime (latency vs delivery) | Go (selected if team supports) | FastAPI / Node.js | Go scores highest for latency and scalability (NFR-001); FastAPI/Node offer faster developer velocity if team lacks Go expertise [UNCLEAR: team choice] |
| AI hosting (latency & ops) | Managed model hosting (SageMaker/Azure ML) | Self-hosted model server | Managed hosting eases ops, versioning and scaling (AIR-004); self-hosting allows tighter data control but more ops burden |

### AI Component Stack
| Component | Technology | Purpose |
|-----------|------------|---------|
| Model Provider | XGBoost / LightGBM (initial) | Tabular risk scoring model for explainability and fast inference (AIR-001, AIR-002) |
| Model Serving | SageMaker Endpoint / Cloud Run + FastAPI | Real-time inference with autoscaling and versioning (AIR-004) |
| Feature Store | Redis / Feast / Postgres (materialized features) | Low-latency features for inference and training data (AIR-006) |
| Event Capture / Pipeline | Kafka / Kinesis + ETL jobs | Capture auth events and features for retraining (AIR-005) |
| AI Gateway | API Gateway / internal service | Route scoring requests, enforce timeouts, caching, and circuit breaker (AIR-004) |
| Model Telemetry & Explainability | MLFlow / custom logging | Model metrics, version tracking, explanation artifacts for audit (AIR-005)

## Technical Requirements
- TR-001: Primary DB — System MUST use PostgreSQL (managed) for the canonical user store to satisfy ACID, transactional updates, and support DR-001/DR-002. Trace: NFR-005, DR-001.
- TR-002: Token & Revocation Store — System MUST use Redis (managed) for short-lived token lookups and revocation with persistent backing to Postgres for long-term audit. Trace: NFR-001, DR-002.
- TR-003: Backend Runtime — System MUST be implemented in a backend runtime chosen by the team (Go preferred for latency; FastAPI/Node acceptable) and meet latency/security/testing requirements. Trace: NFR-001, NFR-006.
- TR-004: Secret Management — System MUST store all secrets and signing keys in a managed Secret Manager and rotate keys per schedule. Trace: NFR-006.
- TR-005: Rate Limiting — System MUST implement edge WAF/rate-limiting and in-app per-account counters with configurable thresholds (default per spec: IP 100 req/min, lockout after 5 attempts). Trace: NFR-008, FR-006, DR-001.
- TR-006: CI/CD & Security — System MUST include CI pipelines enforcing unit/integration tests and automated SAST/dependency scanning and block deploys on failures. Trace: NFR-008.
- TR-007: Deployability — System MUST be containerized and deployable to managed compute with health checks and autoscaling. Trace: NFR-002, NFR-004.
- TR-008: Email Integration — System MUST integrate with a transactional email provider (SES/Postmark/SendGrid) for reset flows and template management to avoid account enumeration. Trace: FR-007, NFR-007.

(Note: Each TR above traces to stated NFR/DR/AIR in the justification line.)

## Domain Entities
- User
  - Represents a registered account.
  - Attributes: id (UUID), email (string, unique, indexed), password_hash (string), hash_algo (string), hash_params (JSON), failed_attempts (int), locked_until (timestamp nullable), roles (array), created_at, updated_at.
  - Relationships: has many Tokens, has many AuditLogs.
- Token (Session / Access / Refresh)
  - Represents issued tokens or sessions.
  - Attributes: token_id (UUID), user_id (FK), type (access|refresh|opaque), issued_at, expires_at, revoked (bool), metadata (ip, user_agent), session_info.
  - Relationships: belongs to User.
- ResetToken
  - Represents password reset single-use token/OTP.
  - Attributes: token (secure), user_id (FK), issued_at, expires_at, used_flag (bool).
  - Relationships: belongs to User.
- AuditLog
  - Represents structured auth events.
  - Attributes: event_id (UUID), event_type, user_id_or_hash, ip, timestamp, outcome, correlation_id, metadata (reason_codes, model_version if applicable).
  - Relationships: optional link to User (when available).
- Role
  - Represents authorization roles.
  - Attributes: role_id, name, permissions (or mapping to services).
  - Relationships: many-to-many with User (or roles stored as array per DR-001).

## Technical Constraints & Assumptions
- All endpoints must be served over TLS (TLS 1.2+) and behind an API gateway or load balancer.
- There is an existing canonical user store accessible; schema migrations may be required to add required fields (failed_attempts, locked_until, hash metadata).
- Default policy values: access token expiry 30 minutes, refresh token optional, lockout after 5 failed attempts, reset token TTL 15 minutes. These are configurable.
- Legacy weak password hashes will be migrated via rehash-on-login; a bulk migration path will be available. [UNCLEAR] scope and timeline for legacy hashing migration.
- Team language/runtime preference is not fixed; selection between Go, Python, Node depends on team skills and delivery timeline. [UNCLEAR] team expertise and preferred runtime.
- Exact capacity targets (peak RPS / concurrent sessions) not specified; capacity planning requires confirmation. [UNCLEAR]
- Log retention and backup retention periods are not specified and must be aligned to compliance requirements. [UNCLEAR]

## Development Workflow
1. Design & Planning: Confirm runtime choice, exact scale targets, retention windows, and AI KPIs; finalize schema and migration plan.
2. Implement Core Auth Flows: API endpoints (login, token validation, forgot-password), secure password hashing (Argon2id), token issuance, and test suites.
3. Infra & Ops: Provision managed Postgres, Redis, Secret Manager, email provider, CI/CD pipelines, logging/monitoring, and autoscaling.
4. Security & QA: Run SAST, dependency scanning, integration tests, performance/load tests (validate NFR-001/NFR-002), and runbook creation.
5. Phase 2: Deploy Risk Scoring service (AIR) behind AI Gateway, enable canary scoring with deterministic policy integration, monitor model metrics and rollback if needed.

