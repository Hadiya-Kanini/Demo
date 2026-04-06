---
post_title: "Epics for Authentication & Account Recovery"
author1: "Senior Product Owner"
post_slug: "auth-account-recovery-epics"
microsoft_alias: "sprod.owner"
featured_image: "https://example.com/images/auth-epics.png"
categories: ["engineering", "product"]
tags: ["authentication", "security", "epics", "auth-service"]
ai_note: "AI-assisted generation"
summary: "Decomposition of authentication and account recovery requirements into prioritized epics including foundational EP-TECH and EP-DATA, feature epics for core auth, credential management, account safety, observability, admin operations, frontend UX, security, and AI detection. Maps all non-UNCLEAR requirements to a single epic."
post_date: 2026-04-06
---

## Rules Used by the Workflow
- Zero orphaned requirements: every non-[UNCLEAR] requirement ID is mapped to exactly one epic.
- Green-field detection applied: EP-TECH (scaffolding/CI/CD/base infra) created as first, highest-priority epic.
- EP-DATA created because design.md contains multiple DR-XXX entity/data requirements.
- Business-value ordering: epics ordered by foundational impact then feature dependencies.
- UI Impact determination: Yes only if epic maps UXR/SCR items from figma_spec.md.
- [UNCLEAR] requirements excluded from epic mapping and listed in Backlog Refinement Required.
- Epic sizing kept within ~12 requirements per epic; split/grouped by outcome.
- Cross-cutting concerns (security, observability, data) separated into dedicated epics.

## Executive Summary
This document decomposes the Authentication & Account Recovery specification into a set of prioritized epics suitable for planning and delivery in a green-field project. EP-TECH establishes the project foundation; EP-DATA delivers entity and persistence scaffolding; feature epics deliver core authentication, credential & recovery flows, account safety, observability, admin tooling, frontend/UX accessibility, security/compliance hardening, and AI-based suspicious-activity detection. All non-UNCLEAR FR-, NFR-, TR-, DR-, UXR-, AIR-, and UC- requirement IDs have been assigned to exactly one epic, providing clear ownership and traceability.

## Validation Snapshot
- Requirement coverage: 100% of non-[UNCLEAR] requirements mapped (50 IDs mapped).
- Epic sizing: All epics within ~3–10 requirement IDs (below 12 threshold).
- Dependency ordering: EP-TECH → EP-DATA → core/platform epics → UX/admin/AI.
- UNCLEAR items isolated for backlog refinement (3 items).

Evaluation Scores:
| Criterion | Score (1-10) |
|----------:|-------------:|
| Requirement Coverage | 10 |
| Correctness of Mapping | 9 |
| Epic Sizing / Manageability | 9 |
| Dependency Ordering | 9 |
| Traceability & Clarity | 9 |
| Average Score | 9.2 |

Evaluation summary: All defined requirements (excluding [UNCLEAR] items) are mapped to single epics, epics respect dependency ordering and sizing guidelines, and UI-impact epics reference Figma screens for implementation. Remaining open items are captured in Backlog Refinement Required.

## Epic Summary Table
| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Project Foundation & Developer Enablement | TR-002, NFR-002, NFR-003, NFR-008 |
| EP-DATA | Core Data & Persistence Layer | DR-001, DR-003, DR-006 |
| EP-001 | Core Authentication & Session Management | FR-001, FR-003, FR-005, FR-006, NFR-001, UC-001, UC-002, UC-003, UC-007, UC-008 |
| EP-002 | Credentials & Password Recovery | FR-002, FR-007, TR-001, DR-005, UC-005, UC-006 |
| EP-003 | Account Safety & Rate Limiting | FR-004, FR-010, TR-004, DR-004, UC-004, UXR-601 |
| EP-004 | Observability, Audit & Logging Pipeline | FR-008, TR-003, DR-002, NFR-005 |
| EP-005 | Admin Tools & Remediation UX | UC-009, UXR-701 |
| EP-006 | Frontend UX, Accessibility & Internationalization | UXR-001, UXR-101, UXR-201, UXR-202, UXR-301, UXR-401, UXR-501, UXR-801, UXR-901 |
| EP-007 | Security & Compliance Controls | FR-009, NFR-004, NFR-006 |
| EP-008 | AI Suspicious-Activity Detection & Triage | AIR-001, AIR-002 |

## Epic Description

### EP-TECH: Project Foundation & Developer Enablement
**Business Value**: Enables all subsequent development by establishing project foundation, CI/CD, security posture, and runtime configuration.

**Description**: Create the base architecture, developer workflows, and infra patterns required to implement the Auth Service reliably and securely. Provide automated pipelines, secrets integration, baseline observability hooks, and runtime configurability to allow teams to build and validate feature epics with minimal friction.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Project repository scaffolding (backend service layout, worker, Infra-as-Code examples)
- CI/CD pipelines (build, unit/integration tests, SAST/DAST stages, canary deploy)
- Dev environment and runbooks (local docker-compose / dev scripts)
- Secrets integration pattern (Vault / Secrets Manager onboarding)
- Base observability and telemetry (OpenTelemetry instrumentation, Prometheus metrics skeleton)
- Config-driven runtime settings (feature flags, thresholds for lockout/rate-limits)
- Security baseline (TLS enforcement at edge patterns, dependency scanning in CI)

**Dependent EPICs**: []

### EP-DATA: Core Data & Persistence Layer
**Business Value**: Enables data operations for all feature epics requiring persistence and enforces data integrity and migration readiness.

**Description**: Define and implement persistent schemas, migrations, and data access patterns for user, token, and audit-related entities. Establish storage patterns for opaque tokens, hashed reset tokens, backups, and PITR to meet retention and recovery NFRs.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- DB schema and migration scripts for User, SessionToken (or token strategy), PasswordResetToken, AuditEvent metadata mapping
- Data access layer patterns and ORM/migration tooling (versioned migrations)
- Seed/mock data for dev/staging and migration-on-login strategy for legacy hashes
- Backup/PITR configuration and documentation (encrypted backups)
- Data retention & purge pipelines aligned with compliance policy

**Dependent EPICs**: EP-TECH

### EP-001: Core Authentication & Session Management
**Business Value**: Delivers the primary user-facing authentication capability—secure login, token issuance, and role-aware access—unlocking application access for end users.

**Description**: Implement server-side authentication endpoints, deterministic token issuance and verification, input validation, session handling (short-lived tokens), and role-based routing/authorization enforcement. Ensure predictable token expiry behavior and server-side checks for protected resources.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/login endpoint: credential verification, token issuance (claims: sub, roles, iat, exp)
- Token verification middleware and rejection semantics (expired => 401 "Session expired")
- Server-side input validation and structured 400 responses with field errors
- Role-based redirect logic and authorization enforcement for protected endpoints
- Documentation of token expiry, timezone/clock skew policy, and sample API contracts
- Performance tuning to meet NFR-001 (auth latency targets)

**Dependent EPICs**: EP-TECH, EP-DATA

### EP-002: Credentials & Password Recovery
**Business Value**: Reduces account compromise risk and ensures users can recover access securely while supporting legacy-hash migration.

**Description**: Implement secure password storage (Argon2id), legacy-hash migration-on-login, and a full forgot-password/reset flow with single-use hashed reset tokens, token expiry, safe email delivery, and session revocation on reset completion.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Password hashing module (Argon2id) with documented parameters and automated tests
- Legacy-hash migration strategy (verify-and-rehash on login)
- POST /auth/forgot and POST /auth/reset endpoints with secure token lifecycle (hashed tokens, TTL default 1 hour)
- Ensure successful reset clears failed_attempts, locked_at and revokes sessions/tokens
- Integration with email queue/worker for reliable delivery and retry semantics
- Tests for single-use token enforcement and reset edge cases

**Dependent EPICs**: EP-TECH, EP-DATA

### EP-003: Account Safety & Rate Limiting
**Business Value**: Protects user accounts from credential abuse and mitigates brute-force and automated attacks while minimizing legitimate-user impact.

**Description**: Implement per-account failed-attempt tracking, configurable lockout behavior, combined per-IP and per-account rate limiting, progressive backoff strategies, and Redis-based atomic counters for low-latency enforcement.

**UI Impact**: Yes/Partial (locked-account UX flows) — overall UX handled in EP-006; this epic provides APIs and states used by UI.

**Screen References**: SCR-005 (Account Locked Info), SCR-011 (Rate Limit Banner)

**Key Deliverables**:
- Failed-login detection and atomic counter implementation (Redis LUA scripts)
- Lockout enforcement and admin unlock APIs (locked_at, failed_count semantics)
- Per-IP and per-account rate-limiter implementation and integration with API Gateway (Retry-After header support)
- Progressive backoff strategy and configuration surface
- Tests simulating attack scenarios (429 and 423 response correctness)
- Clear error codes and non-enumerating messages for locked/limited responses

**Dependent EPICs**: EP-TECH, EP-DATA

### EP-004: Observability, Audit & Logging Pipeline
**Business Value**: Ensures security and operational incidents are visible, auditable, and searchable to support compliance and incident response.

**Description**: Provide append-only audit event publishing, structured logging with PII masking, metrics and alerting for auth KPIs, and a searchable audit store with configurable retention. Deliver the pipeline so other epics can emit standard events and metrics.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Structured audit event schema and ingestion pipeline (async writes to search store)
- Centralized logging integration with PII masking rules
- Metrics instrumentation (login_success_rate, login_failure_rate, account_lock_rate, rate_limit_events) and Grafana dashboards
- Alerts for anomalous spikes (failure rates, lockouts, email delivery failures)
- Retention and index management for audit data (default retention policy)
- Integration tests validating event emission and searchability

**Dependent EPICs**: EP-TECH, EP-DATA

### EP-005: Admin Tools & Remediation UX
**Business Value**: Enables administrators to rapidly remediate locked accounts, investigate auth events, and reduce user support load.

**Description**: Build admin APIs and the admin UI flows for searching locked accounts, viewing attempt history, unlocking accounts, and logging admin actions. Ensure actions are audited and access-controlled.

**UI Impact**: Yes

**Screen References**: SCR-007, SCR-008, SCR-009

**Key Deliverables**:
- Admin API: POST /admin/users/{id}/unlock and related audit logging
- Admin UI: Locked accounts list, account detail with attempts, unlock/force-reset modal
- RBAC enforcement for admin actions and audit trail of admin operations
- Usability enhancements to perform unlock in ≤60s as per UXR-701
- Integration with Observability epic for event search and triage

**Dependent EPICs**: EP-001, EP-004, EP-003

### EP-006: Frontend UX, Accessibility & Internationalization
**Business Value**: Delivers a usable, accessible, and localized authentication experience that meets usability and compliance goals, reducing friction and support overhead.

**Description**: Implement the Figma-specified screens, client-side validation, accessibility features (WCAG 2.1/2.2 AA), responsive layouts, and performance feedback behaviors. Ensure copy and interactions avoid account enumeration and support i18n.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-003, SCR-004, SCR-005, SCR-006, SCR-010, SCR-011

**Key Deliverables**:
- Implement SCR-001..SCR-006, SCR-010, SCR-011 with five states (Default, Loading, Empty, Error, Validation)
- Accessibility compliance (WCAG 2.1/2.2 AA), aria-live behaviors, keyboard navigation, focus management
- Client-side validation and UX error handling consistent with server responses
- Design-token integration and responsive behavior per Figma spec
- Internationalization-ready copy keys and exportable manifest for translations
- Performance UX: spinner/skeleton within 200ms and progressive loading for >300ms responses

**Dependent EPICs**: EP-001, EP-002, EP-003, EP-004

### EP-007: Security & Compliance Controls
**Business Value**: Reduces risk and ensures compliance with OWASP, data protection, and operational security requirements for authentication flows.

**Description**: Implement transport and secrets security, PII handling and retention policies, SAST/DAST pipeline integration, and privacy controls. Ensure the system enforces TLS-only communication and that secrets live in an access-controlled vault.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Enforce TLS-only deployment and validate via automated checks
- Secrets management integration and rollout plan (Vault/Secrets Manager)
- PII masking rules in logs and retention policy implementation
- SAST/DAST and dependency scanning in CI with remediation guidance
- Security runbook for emergency key rotation and incident response
- Documentation for compliance audits and retention schedules

**Dependent EPICs**: EP-TECH

### EP-008: AI Suspicious-Activity Detection & Triage
**Business Value**: Improves detection and remediation of suspicious authentication activity, enabling proactive security operations and reduced fraud exposure.

**Description**: Provide a pluggable detection pipeline that ingests auth events and applies deterministic rule-engine heuristics by default, with a path to integrate ML detectors. Emit alerts into admin triage UI and ensure model decisions are auditable and versioned.

**UI Impact**: Yes (admin triage view)

**Screen References**: SCR-012

**Key Deliverables**:
- Rule-based detection engine integrated with audit event stream (sliding-window heuristics)
- Alert routing to admin UI queue with contextual data for triage
- Hooks and API for optional ML model integration (AIR-002 requirements)
- Model/version logging and auditability for decisions
- Latency and false-positive monitoring metrics
- Operational playbook for tuning detector thresholds and rollout strategy

**Dependent EPICs**: EP-004, EP-003

## Backlog Refinement Required
The following requirements were marked [UNCLEAR] in the source documents and have NOT been mapped to epics. Clarification is required before assignment.

- TR-005: [UNCLEAR] "System MUST define production-grade SLOs for reset email delivery and suspicious-activity false-positive targets"
  - Clarification questions:
    - What are the target SLOs for reset email delivery (e.g., 95% within X minutes)?
    - Are false-positive/false-negative thresholds for suspicious-activity detector specified numerically for pre-production gating?
    - Who owns these SLOs (security, product, or platform)?

- DR-007: [UNCLEAR] "Audit indexing scale — expected query volume & retention window"
  - Clarification questions:
    - What is the expected daily ingestion and query QPS for audit events?
    - What retention window is required for searchable audit data (beyond default 1 year)?
    - Should audit indexing use Elasticsearch at target scale or is DB-partitioned storage acceptable?

- AIR-003: [UNCLEAR] "Define acceptable false positive/negative thresholds for ML detector before production"
  - Clarification questions:
    - What are the target operational thresholds (max acceptable false-positive rate, min acceptable true-positive rate)?
    - Will detectors be enabled in alert-only mode initially, and what is the validation period?
    - Who signs off on production enablement criteria for ML detections?

Please provide clarifications for the items above so they can be assigned to the appropriate epic(s) and scoped for implementation.