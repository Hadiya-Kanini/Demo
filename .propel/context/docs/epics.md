---
title: "Epics for Email+Password Authentication Service"
author: "Senior Product Owner"
date: "2026-04-08"
summary: "Updated epic decomposition for the Email+Password Authentication Service. Preserves EP-TECH and EP-DATA, maps non-ambiguous requirements to epics, and lists [UNCLEAR] items for backlog refinement."
ai_note: "Generated with AI assistance. Items marked [UNCLEAR] in source docs are listed in Backlog Refinement and excluded from epic mappings until clarified."
---

## Epic Summary Table

| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Foundation & Platform (Project scaffolding, CI/CD, security baseline) | TR-003, TR-004, TR-006, TR-007, NFR-001, NFR-006, NFR-008 |
| EP-DATA | Data Schema & Persistence Layer | DR-001, DR-002, DR-003, DR-004, TR-001, TR-002 |
| EP-001 | Authentication Core (Login, credential verification, role redirect) | FR-001, FR-002, FR-003, FR-005, FR-013, UC-001, UC-004 |
| EP-002 | Token & Session Management (TTL, rotation, revocation) | FR-004, FR-011 |
| EP-003 | Password Recovery (Forgot / Reset flow, email integration) | FR-007, UC-002, TR-008 |
| EP-004 | Account Protection & Rate-Limit (Lockout, rate limiting, admin unlock) | FR-006, FR-008, FR-012, TR-005, UC-003 |
| EP-005 | Audit & Observability Pipeline (Structured logs, alerting) | FR-009 |
| EP-006 | Security Controls & Hardening (TLS, secure cookies, parameterized DB) | FR-010, NFR-003 |
| EP-007 | UX - Core Interaction & Flows (Primary actions, task reachability) | UXR-001, UXR-099, UXR-101, UXR-102, UXR-103, UXR-301, UXR-302, UXR-501, UXR-502 |
| EP-008 | UX - Accessibility & Visual Quality (WCAG, error announcements, tokens) | UXR-201, UXR-202, UXR-401, UXR-402, UXR-601, UXR-602, UXR-603 |
| EP-009 | Adaptive Risk & AI-Assisted Detection (phase 2 risk scoring + audit) | FR-014, AIR-001, AIR-003, AIR-004, AIR-005, AIR-006 |

## Epic Description

### EP-TECH: Foundation & Platform (Project scaffolding, CI/CD, security baseline)
**Business Value**: Establishes the secure, observable, and testable platform foundation so all feature work can be delivered, reviewed, and operated reliably.

**Description**: Create the green-field project scaffolding, CI/CD pipelines with SAST/dependency scanning, runtime choices and deployment patterns, secret management integration (KMS/Secret Manager), baseline security hardening, and operational runbooks required to deliver the authentication service at scale.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Project repository structure, developer README and contributor guide
- CI/CD pipelines (build, unit/integration tests, SAST, dependency scanning) and gating rules
- Containerization and deployment templates (health checks, readiness/liveness)
- Secret manager/KMS provisioning and integration for signing keys and secrets
- Baseline monitoring/metrics and synthetic test harness for NFR-001
- Operational runbooks (hashing tuning, key rotation, rollback)

**Dependent EPICs**: EP-DATA, EP-001, EP-002, EP-003, EP-004, EP-005, EP-006, EP-007, EP-008, EP-009

---

### EP-DATA: Data Schema & Persistence Layer
**Business Value**: Provides the canonical persistence model and storage foundation so features are durable, auditable, and consistent.

**Description**: Define and implement the canonical database schema and persistence patterns for users, tokens, reset tokens, and audit logs. Provision Postgres and Redis according to TR-001/TR-002, implement migrations, and create seed/mock data and data access patterns used by feature epics.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Postgres schema for User, Token/Session, ResetToken, AuditLog (DR-001..DR-004)
- Redis-backed token/counter integration for low-latency revocation and rate limiting
- Migration scripts and rehash-on-login hooks for legacy weak hashes
- Backup and recovery runbook (implementation of DR-005 retention specifics deferred)
- Data access layer and integration tests

**Dependent EPICs**: EP-TECH

---

### EP-001: Authentication Core (Login, credential verification, role redirect)
**Business Value**: Enables end users to authenticate and access role-specific resources — the primary product capability.

**Description**: Implement the login endpoint and server-side flow: input validation, secure credential verification against hashed passwords, failed_attempts handling, role inclusion in tokens, and safe post-login redirect decisioning.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/login endpoint and server-side validation (FR-001, FR-002)
- Password verification using Argon2id/bcrypt per FR-003 with parameters stored in DR-001
- Role claim inclusion and safe redirect scaffolding for role routing (FR-005)
- Non-revealing error messaging behavior implemented server-side (FR-013)
- Unit and integration tests covering success, invalid credential, and lockout paths

**Dependent EPICs**: EP-TECH, EP-DATA

---

### EP-002: Token & Session Management (TTL, rotation, revocation)
**Business Value**: Provides secure session semantics and lifecycle management required for session continuity and replay protection.

**Description**: Implement access token issuance, expiry behavior, refresh token rotation and revocation endpoints/mechanisms, and the operational strategy for revocation (blacklist/opaque tokens or rotation).

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Access token issuance and validation with configurable TTL (FR-004)
- Optional refresh token support with rotation and revocation endpoints (FR-011)
- Tests covering token expiry, rotation, revocation, and middleware enforcement
- Integration guidance with data layer (DR-002) and Redis for fast revocation checks

**Dependent EPICs**: EP-TECH, EP-DATA, EP-001

---

### EP-003: Password Recovery (Forgot / Reset flow, email integration)
**Business Value**: Reduces support load and restores user access in a privacy-preserving way.

**Description**: Implement non-revealing forgot-password endpoint, generation and storage of single-use reset tokens/OTPs, email delivery integration with transactional provider, and reset token validation and password update behavior.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/forgot-password flow with non-revealing response (FR-007)
- Reset token generation, single-use enforcement and TTL handling (DR-003)
- Integration with transactional email provider and template management (TR-008)
- Reset endpoint to validate token and update password securely
- End-to-end tests for token issuance, single-use behavior, expiry and non-disclosure

**Dependent EPICS**: EP-TECH, EP-DATA

---

### EP-004: Account Protection & Rate-Limit (Lockout, rate limiting, admin unlock)
**Business Value**: Protects users and the platform from brute-force and credential-stuffing attacks and provides safe support/administration flows.

**Description**: Implement per-account and per-IP rate limiting, failed_attempts counters and configurable account lockout, admin-only unlock endpoint, integration with CAPTCHA escalation points, and progressive backoff behavior.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Account lockout logic (lock after configured consecutive failed attempts) and locked_until tracking (FR-006)
- Per-IP and per-account rate limiting engine and integration with edge WAF (FR-008, TR-005)
- Admin unlock endpoint with RBAC and audit logging (FR-012)
- CAPTCHA integration points and escalation thresholds
- Tests simulating attacks, lockout timing, and admin unlock flows

**Dependent EPICs**: EP-TECH, EP-DATA, EP-001

---

### EP-005: Audit & Observability Pipeline (Structured logs, alerting)
**Business Value**: Enables forensicability, monitoring, and incident response for authentication events.

**Description**: Implement structured audit event emission for auth actions, centralized logging pipeline, alerts for anomalous patterns, PII redaction in logs, and integration with monitoring dashboards.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Emission of structured auth audit events (FR-009) with required fields (DR-004)
- Centralized log ingestion and retention (ELK/OpenSearch) and PII redaction rules
- Alerting rules for failed login spikes and suspicious patterns
- Correlation IDs and tracing integration for request-level diagnosis
- Observability tests and dashboards

**Dependent EPICs**: EP-TECH, EP-DATA

---

### EP-006: Security Controls & Hardening (TLS, secure cookies, parameterized DB)
**Business Value**: Ensures the authentication surface meets security standards and reduces risk of credential exposure.

**Description**: Enforce TLS across endpoints, set secure cookie flags, ensure parameterized DB access and CSP/HSTS headers where appropriate, and include security integration tests and CI gates.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Enforce TLS 1.2+ for all endpoints and certificate management guidance (FR-010)
- Secure cookie configuration (HttpOnly, Secure, SameSite) and storage guidance
- Parameterized DB access patterns / ORM usage to prevent injection
- Security test suites, header checks, and CI SAST integration (TR-006)
- Documentation and ops runbook for key rotation and secret management (TR-004 link back to EP-TECH)

**Dependent EPICs**: EP-TECH

---

### EP-007: UX - Core Interaction & Flows (Primary actions, task reachability)
**Business Value**: Delivers the primary user-facing flows with usable, accessible interactions so users can sign in and recover access efficiently.

**Description**: Implement UX behaviors and design integration for core auth flows — canonical component use, inline validation, focus-first-invalid field behavior, submit feedback and loading states, and microcopy about token storage and session persistence.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-003, SCR-004, SCR-014

**Key Deliverables**:
- Implement designs and interaction states for Login, Forgot Password, Reset Password, Reset Confirmation, and Password Helper screens (SCR-001, SCR-002, SCR-003, SCR-004, SCR-014)
- Client-side inline validation and focus-first-invalid behavior (UXR-103)
- Flow optimization to meet interaction budget (≤3 interactions) (UXR-101)
- Microcopy for token storage/session persistence (UXR-302)
- Prevent double-submit and show immediate feedback (UXR-501); reduced-motion variants (UXR-502)

**Dependent EPICs**: EP-001, EP-002, EP-003

---

### EP-008: UX - Accessibility & Visual Quality (WCAG, error announcements, tokens)
**Business Value**: Ensures the authentication UI is accessible, visually consistent, and non-revealing to meet legal and usability standards.

**Description**: Implement WCAG 2.2 AA baseline across auth screens, ARIA live regions for error/status announcements, accessible CAPTCHA integration area, and ensure design tokens and visual rules are applied across components.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-005, SCR-011, SCR-013

**Key Deliverables**:
- WCAG 2.2 AA compliance checks and fixes (UXR-201)
- Screen reader announcements for errors and statuses (UXR-202)
- Visual and token adherence in components (UXR-401)
- Non-revealing copy and accessible error/lockout guidance (UXR-402, UXR-602)
- Rate-limit and CAPTCHA UI patterns that are accessible (UXR-603)
- Automated accessibility test suites and verification steps

**Dependent EPICs**: EP-007, EP-001, EP-004

---

### EP-009: Adaptive Risk & AI-Assisted Detection (phase 2 risk scoring + audit)
**Business Value**: Provides advanced risk signals to reduce fraud and trigger appropriate step-up actions while preserving determinism and auditability (phase 2).

**Description**: Build a separate risk scoring service and gateway that returns explainable risk scores and reason codes, integrate model telemetry and versioning, implement circuit-breaker fallbacks, and persist model inputs/outputs for auditability in a privacy-preserving manner.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Risk scoring service API and low-latency inference path (AIR-001)
- Model operationalization: versioning, canary deploys, rollback policies and fallback policy (AIR-004)
- Model privacy & safety controls (AIR-003) and audit logging of inputs/outputs (AIR-005)
- Throughput/caching strategies and latency guarantees (AIR-006)
- Proof-of-concept integration with deterministic policy layer and demo actions for FR-014

**Dependent EPICs**: EP-TECH, EP-DATA, EP-001, EP-005

---

## Backlog Refinement Required

The following requirements were marked [UNCLEAR] in the source documents and must be clarified before mapping and implementation. They are NOT mapped to any epic until clarified.

- NFR-002 — "System MUST target availability of authentication endpoints ≥ 99.95% (monthly). [UNCLEAR if stricter SLA required.]"  
  Clarification needed: Confirm target SLA (is 99.95% final or must a stricter SLA be enforced), and any contractual SLA obligations.

- NFR-004 — "System MUST scale horizontally... [UNCLEAR exact concurrent users/peak RPS to size capacity.]"  
  Clarification needed: Provide target peak RPS / concurrent sessions and expected traffic patterns for capacity planning.

- NFR-005 — "Deterministic correctness for lockout and token revocation... [UNCLEAR acceptable inconsistency tolerance]."  
  Clarification needed: Define acceptable consistency window (e.g., eventual consistency tolerance) and recovery constraints under partitions.

- NFR-007 — "Log retention policies. [UNCLEAR exact retention windows (days/years)]."  
  Clarification needed: Specify retention durations for logs and audit records per compliance.

- DR-005 — "Backups and retention: daily DB backups with PITR... [UNCLEAR exact retention duration to confirm with compliance]."  
  Clarification needed: Confirm backup retention policy (retention period, offsite retention, restoration RTO/RPO requirements).

- DR-006 — "Schema migrations and migration plan (including rehash-on-login). [UNCLEAR scope of legacy hashes and bulk migration strategy]."  
  Clarification needed: Provide scope of legacy hash formats, percentage of users affected, and preferred migration strategy (rehash-on-login vs bulk migrate).

- AIR-002 — "Model Quality — AUC ≥ 0.85 and documented FP/FN tolerances. [UNCLEAR exact FP/FN thresholds (PO to define)]."  
  Clarification needed: Define acceptable false positive / false negative thresholds and policy for when model results can be enforced vs recommended.

Please provide answers or updated requirements for the items above so they can be assigned to epics and included in planning.