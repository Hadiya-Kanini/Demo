---
title: "Epics for Authentication & Account Recovery"
author: "Senior Product Owner"
date: 2026-04-06
summary: "Decomposition of authentication, account protection, recovery, observability, security, data, and UI requirements into prioritized epics with requirement-to-epic mappings. UNCLEAR items are recorded for refinement."
ai_note: "Generated with AI assistance; validated against provided requirement checklist. [UNCLEAR] items are listed for clarification."
---

## Epic Summary Table

| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Project Foundation & Platform Scaffolding | NFR-001, NFR-002, NFR-003, NFR-008, NFR-010, TR-002 |
| EP-DATA | Core Data & Persistence Layer | DR-001, DR-003, DR-004, DR-005, DR-006 |
| EP-001 | Authentication Core (Login, Token, Authorization) | FR-001, FR-002, FR-003, FR-005, FR-006, TR-001, UC-001, UC-003, UC-007, UC-008 |
| EP-002 | Account Protection & Rate Limiting (Lockout, Abuse Mitigation, Detection Hooks) | FR-004, FR-010, TR-004, AIR-001, AIR-002, UC-002, UC-004 |
| EP-003 | Security & Compliance Controls | FR-009, NFR-004, NFR-006 |
| EP-004 | Observability & Audit Pipeline | FR-008, NFR-005, DR-002, TR-003 |
| EP-005 | Account Recovery (Forgot / Reset Flow) | FR-007, UC-005, UC-006 |
| EP-006 | Admin Backend & Investigation APIs | UC-009 |
| EP-007 | UI — Forms & Accessibility (Login, Forgot, Reset, Locked States) | UXR-001, UXR-101, UXR-200, UXR-201, UXR-202, UXR-401, UXR-501, UXR-601, UXR-900, UXR-901 |
| EP-008 | UI — Role Dashboards & Session UX | UXR-300, UXR-301, UXR-801 |
| EP-009 | UI — Admin Console & Investigation UX | UXR-400, UXR-500, UXR-600, UXR-700, UXR-701 |

## Epic Description

### EP-TECH: Project Foundation & Platform Scaffolding
**Business Value**: Enables all subsequent development by establishing the project foundation (scaffolding, CI/CD, secure deployment patterns and runtime configuration), reducing time-to-market and operational risk.

**Description**: Create the baseline infrastructure and developer experience required to build and operate the Authentication service: repo layout, CI/CD pipelines, environment configuration, secrets management, platform observability hooks, and deployment baseline (Kubernetes/managed containers). Provide runtime configuration support for auth parameters (lock thresholds, token ttl, rate limits) and ensure integration points for Redis and PostgreSQL are available.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Project repository scaffold, coding standards, and linters
- CI/CD pipelines with SAST/DAST and automated testing gates
- Secrets management integration (Vault / cloud secrets) and access patterns
- Infrastructure IaC templates for managed Postgres, Redis, and message queue
- Runtime configuration mechanism and feature flags for auth thresholds
- Onboarding documentation for devs and runbook stubs

**Dependent EPICS**: []

---

### EP-DATA: Core Data & Persistence Layer
**Business Value**: Enables data operations for all feature epics requiring persistence; provides authoritative storage for user records, tokens, counters, reset tokens and backups.

**Description**: Define and implement the DB schemas, migrations, and ephemeral data stores required by authentication features. Implement user table changes, token storage (opaque token hashes or session table), reset-token storage (hashed), Redis structures and persistence approach for counters/locks, and backup & PITR configuration.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Versioned DB schema and migrations for User, SessionToken, PasswordResetToken, AuditEvent
- Redis schema and LUA scripts for atomic counters and token revocation primitives
- Seed/mock data and migration strategy for legacy password hash migration
- Encrypted backup configuration and PITR tested documentation
- Data access patterns and repository layer interfaces for services

**Dependent EPICS**: [EP-TECH]

---

### EP-001: Authentication Core (Login, Token, Authorization)
**Business Value**: Delivers the core user-facing capability: reliable, secure authentication with deterministic tokens and role-based routing — the main product value for end users.

**Description**: Implement API endpoints and server logic for email/password authentication, token issuance (short-lived), claims population with roles, client/server validation, password hashing strategy and migration-on-login, token verification, session invalidation, and server-side input validation. Provide API contracts, error codes, and tests ensuring deterministic behavior.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/login implementation with deterministic token issuance (claims: sub, roles, iat, exp)
- Password hashing module (Argon2id) and migration-on-login for legacy hashes (TR-001)
- Server- and client-side validation patterns and standardized error payloads (400 / field-level errors)
- Token expiry enforcement and configurable TTLs (default 30 minutes)
- Automated tests: unit, integration, and contract tests for login flows
- API docs (OpenAPI) and sample responses for engineering tickets

**Dependent EPICS**: [EP-TECH, EP-DATA]

---

### EP-002: Account Protection & Rate Limiting (Lockout, Abuse Mitigation, Detection Hooks)
**Business Value**: Protects users and platform from credential abuse and brute-force attacks, reducing risk of account compromise and operational incidents.

**Description**: Implement failed-attempt tracking, account lockout policy (configurable threshold), per-account and per-IP rate limiting, progressive backoff, and pluggable suspicious-activity detection hooks (rule-based by default; ML as an integration point). Emit protection events to audit pipeline for investigation and support admin remediation paths.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Atomic counter and lock semantics implemented in Redis + persistence to Postgres
- Account lock/unlock logic (locked_at, failed_count) and 423 response behavior
- Per-IP and per-account token-bucket rate limiter with Retry-After headers
- Detection hooks for suspicious activity events and integration points for AIR components
- Tests simulating brute-force attacks and verifying 429/423 behaviors
- Configuration and operational runbooks for thresholds and progressive delays

**Dependent EPICS**: [EP-TECH, EP-DATA, EP-001]

---

### EP-003: Security & Compliance Controls
**Business Value**: Ensures the authentication service meets regulatory and security standards (OWASP mitigations, secrets handling, encryption, and privacy), enabling production readiness and compliance.

**Description**: Apply and validate security controls across transport, storage, and code: TLS enforcement, secrets vault usage for keys, production-grade password handling practices, PII masking in logs, at-rest encryption, SAST/DAST pipeline integration, and auditability of security changes. Implement policies and procedures required for compliance.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- TLS enforcement and edge configuration (API Gateway) and automated checks
- Secrets management policies, key rotation pattern, and no-secrets-in-config verification
- PII masking and log redaction rules implemented and verified
- SAST/DAST pipeline integration and remediation plan
- Documentation of security controls, compliance mapping, and runbooks for incident response

**Dependent EPICS**: [EP-TECH]

---

### EP-004: Observability & Audit Pipeline
**Business Value**: Provides operational visibility and an auditable trail for authentication events to support monitoring, incident response, compliance, and investigations.

**Description**: Design and implement structured audit logging, metrics, traces, searchable audit store (append-only), and dashboards/alerts for auth KPIs (success/failure rates, lockouts, reset requests). Ensure PII masking in audit logs, retention policy enforcement, and near-real-time availability for admin query.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Structured audit event producer integrated with async ingestion pipeline
- Audit storage (searchable store) with retention and RBAC controls (DR-002)
- Metrics and dashboards (login_success_rate, login_failure_rate, account_lock_rate, rate_limit_events)
- Alerts configured for anomalous spikes and operational thresholds
- Integration docs for admin tools and investigators to consume audit data

**Dependent EPICS**: [EP-TECH, EP-DATA]

---

### EP-005: Account Recovery (Forgot / Reset Flow)
**Business Value**: Restores user access securely and reliably, reducing support costs and improving user experience for locked/forgotten accounts.

**Description**: Implement forgot-password and reset endpoints, generate single-use cryptographically-random reset tokens (hashed server-side), enforce TTL (default 1 hour), validate tokens, allow password updates subject to password policy, clear failed counters, and revoke existing sessions. Integrate with email delivery via background worker with retry.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/forgot endpoint that enqueues secure reset email (no user enumeration)
- POST /auth/reset endpoint validating hashed tokens and enforcing password policy
- Reset token hashing, expiry, used_flag enforcement, and revocation behavior
- Background worker for email delivery with retry and observability metrics
- Tests for token lifecycle, expiry, single-use enforcement and session invalidation

**Dependent EPICS**: [EP-TECH, EP-DATA, EP-001]

---

### EP-006: Admin Backend & Investigation APIs
**Business Value**: Empowers administrators to remediate account issues and investigate incidents, improving operational response time and reducing user-impacting downtime.

**Description**: Provide admin APIs to list locked accounts, view attempt history, unlock accounts, and query audit events. Ensure actions are authorized, audited, and rate-limited. Backend endpoints will be consumed by the Admin UI epic.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /admin/users/{id}/unlock implementation with audit logging
- Admin endpoints for listing locked accounts and recent attempts (paginated)
- Authorization checks (admin role enforcement) and safe error responses
- Audit linkage to Observability (EP-004) and tests for authorization/auditing
- API documentation for frontend consumption

**Dependent EPICS**: [EP-004, EP-002, EP-DATA]

---

### EP-007: UI — Forms & Accessibility (Login, Forgot, Reset, Locked States)
**Business Value**: Delivers accessible, localized and performant user-facing forms for authentication and recovery, ensuring compliance with WCAG and increasing successful self-service recovery.

**Description**: Implement the Login, Forgot Password, Reset Password, Locked Account, Rate Limit and related form screens with client-side validation, accessible error handling (aria-live), spinners/skeletons for >200ms feedback, and i18n-ready copy. Map design tokens into components and ensure keyboard/screen-reader support.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-003, SCR-004, SCR-005, SCR-011

**Key Deliverables**:
- Implement accessible form components and form flows for SCR-001..SCR-005
- Client-side validation matching server validation with consistent error payload display
- Spinner/skeleton patterns for loading states and progressive feedback (UXR-401)
- Non-enumerating generic error messages (UXR-501) and aria-live announcements (UXR-202)
- Design token integration and localization-ready copy mapping (UXR-101, UXR-900/901)
- Usability and accessibility test artifacts (WCAG AA compliance evidence)

**Dependent EPICS**: [EP-001, EP-005]

---

### EP-008: UI — Role Dashboards & Session UX
**Business Value**: Ensures users land in the correct role-specific area and experience clear session expiry flows, improving task completion and reducing support friction.

**Description**: Implement lightweight role-based dashboard scaffolds, session-expired UX, and progressive loading patterns. Ensure responsiveness across breakpoints, skeletons for slow loads, and accessible navigation.

**UI Impact**: Yes

**Screen References**: SCR-006, SCR-010

**Key Deliverables**:
- Role-based landing page scaffolds for admin/user roles
- Session expiry prompt and re-authentication flow (SCR-010)
- Responsive layout and skeletons for slow-loading content (UXR-301, UXR-801)
- Integration tests ensuring correct redirect/URI behavior based on role claims

**Dependent EPICS**: [EP-001, EP-004]

---

### EP-009: UI — Admin Console & Investigation UX
**Business Value**: Provides administrators with efficient tools to find, triage, and remediate locked or suspicious accounts, reducing mean time to remediation and improving security posture.

**Description**: Build the Admin UI for locked accounts list, account detail and attempts, unlock/force-reset modal with audit confirmation, and suspicious-activity queue UI. Focus on filter/search efficiency, keyboard accessibility, and minimal actions to unlock (≤2 actions).

**UI Impact**: Yes

**Screen References**: SCR-007, SCR-008, SCR-009, SCR-012

**Key Deliverables**:
- Admin locked-accounts list with search, filters, pagination (SCR-007)
- Account detail view with recent attempts and context (SCR-008)
- Unlock / Force Reset modal with confirmation and reason capture (SCR-009)
- Suspicious activity queue UI for triage (SCR-012)
- Accessibility & usability testing artifacts (UXR-701)

**Dependent EPICS**: [EP-006, EP-004, EP-DATA]

---

## Backlog Refinement Required

- DR-007: [UNCLEAR] Audit indexing scale — Clarify expected query volume, retention window and SLA for audit search indexing to decide Elasticsearch vs DB-based indexing. Question: "What is the expected daily write volume and peak query rate for audit searches, and the retention/availability SLA for indexed audit data?"

- TR-005: [UNCLEAR] Production-grade SLOs for reset email delivery and suspicious-activity false-positive targets — Clarify SLO targets. Question: "Please confirm production SLOs for reset email delivery (e.g., % delivered within X minutes) and required false-positive/negative thresholds for suspicious-activity detection before enabling ML components."

- AIR-003: [UNCLEAR] Acceptable false positive/negative thresholds for ML detector — Clarify thresholds and rollout gating. Question: "What are the acceptable FP/FN thresholds, and what rollback or manual review policy should apply during ML rollout?"

- FR-011: [UNCLEAR] Referenced optional suspicious-activity detection feature — FR-011 referenced in design notes but not defined in spec. Question: "Confirm whether FR-011 (suspicious-activity detection) should be an explicit functional requirement for the initial release, and if so, provide acceptance criteria and priority."