---
post_title: "Epics for Authentication Service (Auth)"
author1: "Senior Product Owner"
post_slug: "epics-auth-service"
microsoft_alias: "sprod_owner"
featured_image: ""
categories: ["engineering","product"]
tags: ["epics","authentication","security","auth-service"]
ai_note: "AI-assisted epic decomposition"
summary: "Comprehensive epic decomposition for the Authentication Service, mapping all functional and non-functional requirements to actionable epics with priorities, dependencies, risks, and success metrics."
post_date: 2026-04-01
---

## Workflow Rules Used
- Follow Input Processing Instructions: default input files used (spec provided), file-type detection rules applied.
- Zero Orphaned Requirements: every FR and NFR mapped to exactly one epic.
- Business Value Ordering: epics ordered by business value then dependencies.
- Manageable Scope: epics sized to ~5-12 requirements; split where necessary.
- EP-TECH Inclusion: project scaffold & infra epic included (green-field detection due to no codeanalysis.md).
- EP-DATA Inclusion: data epic included because spec requires schema changes, token/revocation, and encrypted storage.
- UNCLEAR Handling: none tagged [UNCLEAR]; decision points documented in Backlog Refinement.
- Template Adherence: populated per .propel/templates/epics-template.md structure.
- Quality Evaluation: coverage, traceability, sizing, priority, and compliance assessed.

## Epic Summary Table
| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Project Foundation & Platform (scaffolding, CI/CD, infra) | NFR-001, NFR-004, NFR-005, NFR-008 |
| EP-DATA | Core Data & Persistence Layer (schema, encryption, tokens) | FR-003, FR-007, FR-011, FR-012, NFR-002 |
| EP-001 | Authentication Core (login, validation, tokens, RBAC) | FR-001, FR-002, FR-004, FR-005, FR-008, NFR-003, NFR-007 |
| EP-002 | Account Protection & Abuse Mitigation (lockout, rate-limits) | FR-006, FR-010 |
| EP-003 | Recovery & Administration (password reset, admin unlock) | FR-013 |
| EP-004 | Audit, Logging & Privacy Compliance | FR-009, NFR-006 |
| EP-005 | Optional Risk Scoring & Adaptive Controls (AI candidate) | FR-014 |

## Evaluation Scores
| Criterion | Score (1-5) | Notes |
|-----------|-------------|-------|
| Requirement Coverage | 5 | All FR-001..014 and NFR-001..008 mapped exactly once. |
| Traceability | 5 | Direct mapping from spec requirements to epics; no orphans. |
| Epic Sizing & Balance | 4 | Most epics within target; EP-TECH and EP-DATA are L (larger) by nature. |
| Priority & Dependency Ordering | 5 | EP-TECH first (green-field), EP-DATA early; feature epics ordered by business value. |
| Compliance with Template & Rules | 5 | Template fields populated; backlog refinement documented. |
| Average Score | 4.8 | Computed average of the above. |

Evaluation summary: Coverage and traceability are complete; epics are prioritized to unblock delivery (platform → data → features). Two large infrastructure epics (EP-TECH, EP-DATA) are expected; plan contains refinement points for token model and lockout semantics.

## Epic Overview (counts)
- Total epics: 7 (including EP-TECH and EP-DATA)
- Functional requirements mapped: 14 (FR-001..FR-014)
- Non-functional requirements mapped: 8 (NFR-001..NFR-008)
- Unmapped/UNCLEAR: 0

---

## Epic Description

### EP-TECH: Project Foundation & Platform (scaffolding, CI/CD, infra)
**Business Value**: Enables all subsequent development by establishing reliable, secure, and observable platform foundation.

**Description**: Create the project scaffolding, CI/CD pipelines, development environments, baseline architecture, secrets management, TLS enforcement, deployment patterns, observability, and test pipeline to enable safe, repeatable development and delivery of the authentication service.

**UI Impact**: No (platform-level), except test harnesses for UI validation.

**Screen References**: N/A

**Key Deliverables**:
- Project repository structure and README with contribution guide.
- CI pipeline (lint, unit tests, integration tests, security scans).
- CD pipeline for staging and production deployments.
- Infrastructure as code templates (Kubernetes/containers or cloud-native equivalent).
- Secrets management integration (Vault/KMS) and config management.
- TLS enforcement configuration and HSTS/CSP baseline (NFR-001).
- Observability baseline: centralized logging exporter, metrics (login success/failure, lock events), dashboards, and alerts (NFR-005).
- Test environments and CI flags for accessibility & performance tests (NFR-008).
- Security baseline hardening checklist.

**Dependent EPICs**: (None) — EP-TECH is first.

**Product Specification**:
- Define branching and release strategy.
- Document SLOs and SLI definitions for auth service (align with NFR-003, NFR-004).
- Provide deployment runbooks and rollback procedures.

---

### EP-DATA: Core Data & Persistence Layer
**Business Value**: Enables secure, compliant persistence of credentials, tokens and audit artifacts necessary for feature epics.

**Description**: Implement database schema and migrations for password hashes, failed-attempt counters, locked state, reset tokens, token revocation data, and encryption-at-rest for sensitive columns. Provide seed/mock data for integration tests and migration utilities for existing data if needed.

**UI Impact**: No (backend/data only).

**Screen References**: N/A

**Key Deliverables**:
- Data model and schema for user authentication fields: password_hash, password_salt (if used), failed_attempts, locked_until, reset_token (hashed), token_version / revocation table.
- Migration scripts and migration plan for existing datastore (if present).
- DB encryption for sensitive columns and proof-of-configuration (FR-012).
- Secrets & key rotation procedures in KMS/Vault for DB encryption keys and signing keys (FR-012).
- Integration tests with seeded data for password reset and login flows.
- Performance baseline for data layer operations (indexes, connection pool sizing).

**Mapped Requirements**: FR-003, FR-007, FR-011, FR-012, NFR-002

**Dependent EPICs**:
- Blocks EP-001 (Authentication Core) and EP-002 (Account Protection) — data must be in place before feature-level persistence-dependent flows can be validated.

**Product Specification**:
- Schema DDL and migration strategy.
- Define hashing configuration storage and migration for legacy hashes.
- Token revocation design (revocation list vs token versioning) to be decided.

---

### EP-001: Authentication Core (login, validation, tokens, RBAC)
**Business Value**: Core user-facing functionality allowing secure access and role-based entry into the system.

**Description**: Implement login endpoint, input validation, password verification using secure hash, token issuance with TTL, role-based redirection/authorization, logout flow, and accessibility requirements.

**UI Impact**: Yes — login and post-login redirection flows.

**Screen References**: SCR-LOGIN, SCR-DASHBOARD-ROLE (if figma exists; otherwise N/A)

**Key Deliverables**:
- POST /auth/login implementation with client/server validation (FR-001, FR-002).
- Password verification integrated with EP-DATA secure hash storage (FR-003 dependency).
- Token issuance with TTL and expiry enforcement (FR-004).
- Role-based redirect logic and RBAC enforcement on protected endpoints (FR-005).
- Logout endpoint and token revocation integration with EP-DATA (FR-011).
- Non-revealing error messages and client-side UX alignment (FR-008).
- Performance optimization to meet NFR-003 latency targets (auth <1s median).
- Accessibility compliance for login and reset flows (NFR-007).

**Dependent EPICs**:
- Depends on EP-TECH (platform and observability) and EP-DATA (schema and token storage).
- After EP-DATA and EP-TECH deliverables are available.

**Product Specification**:
- API contract for /auth/login, /auth/logout, token format and TTL.
- Integration tests validating happy path and error paths (link to CI).

---

### EP-002: Account Protection & Abuse Mitigation
**Business Value**: Protects user accounts and infrastructure from credential-stuffing and brute-force attacks, safeguarding reputation and users.

**Description**: Implement per-account lockout, per-IP and per-account rate limiting, throttling/backoff strategies, and CAPTCHA insertion points. Integrate monitoring and alerts for suspected attacks.

**UI Impact**: Yes — potential CAPTCHA insertion and lockout messaging.

**Screen References**: SCR-LOGIN (CAPTCHA UI insertion), SCR-LOCKED-INFO

**Key Deliverables**:
- Failed-attempt tracking and account lock marking (FR-006).
- Per-IP and per-account rate-limiting middleware and configuration (FR-010).
- Exponential backoff and optional CAPTCHA insertion strategy.
- Integration tests and load tests to validate throttling (FR-010 acceptance).
- Alerts and dashboards showing attack indicators (integrate with EP-TECH observability).

**Dependent EPICs**:
- Depends on EP-DATA for failed attempts and lock state persistence.
- Integrates with EP-TECH for rate-limit infra (Redis or edge rate limiting).

**Product Specification**:
- Rate-limits configurable via env/config; default example: per-IP 100 r/m, lock threshold 5 attempts in 15 minutes.
- Define CAPTCHA insertion thresholds and integration points.

---

### EP-003: Recovery & Administration (password reset, admin unlock)
**Business Value**: Enables secure account recovery and administrative remediation to reduce support cost and user friction.

**Description**: Implement secure forgot-password flow with single-use expiring tokens emailed via SMTP, admin endpoints for unlocking accounts and reviewing lockout history, and self-service recovery where appropriate.

**UI Impact**: Yes — reset request and reset form screens; admin UI or API.

**Screen References**: SCR-RESET-REQUEST, SCR-RESET-FORM, SCR-ADMIN-UNLOCK

**Key Deliverables**:
- POST /auth/reset-request and validation responses (FR-007).
- Generation and secure storage of single-use reset tokens; email templates and SMTP integration.
- Reset flow handling and immediate token invalidation after use.
- Admin endpoint(s) to unlock accounts and read lockout audit history (FR-013).
- Rate-limiting for reset requests and monitoring.

**Dependent EPICs**:
- Depends on EP-DATA (reset token storage) and EP-TECH (email secrets, observability).
- EP-003 can be developed in parallel after EP-DATA primitives are defined.

**Product Specification**:
- Reset token TTL default = 1 hour (configurable).
- Admin endpoints require admin RBAC and are audited.

---

### EP-004: Audit, Logging & Privacy Compliance
**Business Value**: Provides necessary auditability and compliance controls for security and regulatory needs.

**Description**: Implement audit event generation for authentication events, export to centralized logging, retention policies, and privacy controls (minimize PII, enforce retention). Provide dashboards and alerting for anomalous patterns.

**UI Impact**: No (back-office dashboards and logs).

**Screen References**: N/A

**Key Deliverables**:
- Emit audit logs for successful login, failed attempts, account locks, reset requests/completions (FR-009).
- Centralized logging export and retention policy alignment (NFR-006).
- Dashboards for security and operational metrics (login rates, lock events).
- Data retention and PII minimization controls and documentation.
- Test-suite validating audit event structure and export.

**Dependent EPICs**:
- Depends on EP-TECH (observability infra) and EP-DATA (log storage schema if needed).

**Product Specification**:
- Audit event schema with timestamp, user identifier (pseudonymized where necessary), source IP, outcome code.
- Retention policy documentation and compliance checks.

---

### EP-005: Optional Risk Scoring & Adaptive Controls (AI candidate)
**Business Value**: Provides advanced protection by assessing login risk in real time and enabling adaptive mitigations, reducing false positives and improving security posture.

**Description**: Implement an opt-in risk assessment module that scores login attempts (IP reputation, velocity, history) and returns risk levels to enforce adaptive controls (e.g., require CAPTCHA or MFA). This is scoped as an optional module for later release.

**UI Impact**: Yes — may alter login flow to require CAPTCHA/MFA or present challenge.

**Screen References**: SCR-LOGIN (adaptive challenge), SCR-ADMIN-REVIEW (flag review)

**Key Deliverables**:
- Risk scoring API integrated into login pipeline (FR-014).
- Deterministic mapping of score ranges to actions (low/medium/high).
- Admin review UI/queue for flagged attempts.
- Logging of risk scores and actions taken.
- Module designed to be opt-in and togglable via config.

**Dependent EPICs**:
- Requires EP-TECH (deployment infra) and EP-002 (CAPTCHA flows) for integration.

**Product Specification**:
- Define data sources and privacy constraints for scoring.
- Ensure modular design so core auth can run independently.

---

## Backlog Refinement Required
- Token model decision: JWT (opaque claims revocation complexity) vs Opaque tokens (session store). Impact: implementation complexity for FR-011 revocation and performance.
- Lockout semantics: temporary automatic unlock after X minutes vs admin-only/manual unlock vs self-service reset dependency. Clarify default behavior and policy.
- Password policy: minimum length and complexity requirements (affects FR-003 and UX).
- Reset token handling: confirm TTL, reuse policy (single-use already specified), retry limits.
- Migration scope: confirm whether there is an existing user datastore requiring migration and the timeline for migration.
- SMTP deliverability constraints and bounce handling policy.
(These are NOT mapped as [UNCLEAR] requirements but are required decisions before implementation.)

---

## Risk Assessment (per epic)
- EP-TECH Risks:
  - Risk: Misconfigured secrets or key material leakage.
    - Mitigation: Vault/KMS integration, automated scans, least-privilege IAM.
  - Risk: Insufficient CI security checks.
    - Mitigation: Add SAST, dependency scanning, and policy gating.
- EP-DATA Risks:
  - Risk: Data migration errors and production data loss.
    - Mitigation: Backups, dry-run migrations, migration rollbacks, staging verification.
  - Risk: Encryption key management errors.
    - Mitigation: Key rotation, access control, audit.
- EP-001 Risks:
  - Risk: Token leakage (XSS/localStorage).
    - Mitigation: Recommend HttpOnly secure cookies, short TTL, client guidance.
  - Risk: Performance regressions under load.
    - Mitigation: Performance testing in CI, caching, autoscaling.
- EP-002 Risks:
  - Risk: Account lockout abuse (denial of service to users).
    - Mitigation: Combine per-IP and per-account heuristics, CAPTCHA before hard lock, admin unlock.
- EP-003 Risks:
  - Risk: Reset email abuse or spamming.
    - Mitigation: Rate-limiting, CAPTCHAs, monitoring.
- EP-004 Risks:
  - Risk: Audit logs containing sensitive PII.
    - Mitigation: Pseudonymize identifiers, minimize PII logged, privacy review.
- EP-005 Risks:
  - Risk: False positives/negatives from risk scoring; privacy implications.
    - Mitigation: Opt-in, review thresholds, manual review pipeline, privacy safeguards.

Risk Severity Scale: High / Medium / Low noted implicitly in mitigations above.

---

## Success Metrics

Overall product-level targets:
- Authentication success rate (valid credentials) ≥ 99% under normal conditions.
- Median authentication processing latency < 1s; P95 < 2s (NFR-003).
- Account lock triggers after configured failed attempts and prevents login (verify via test suites).
- Token expiry enforcement: 100% rejection post-expiry in tests.
- Reset tokens single-use and expire per TTL; tests validate.
- Audit logging coverage: 100% of events (success, failure, lock, reset) emitted and exported.
- Accessibility: Login/reset pages meet WCAG 2.1 AA (audit pass).
- Automated tests: Unit+Integration+E2E coverage for all FR acceptance criteria included in CI.

Per-epic success metrics:
- EP-TECH: Successful CI runs and deployment to staging; alerts firing on misconfig with <1h Mean Time to Detect for production infra issues.
- EP-DATA: Migrations executed with 0% data loss in staging; encryption keys validated; schema regression tests pass.
- EP-001: Auth endpoint meets latency and throughput targets on baseline load; RBAC tests pass for customer/employee/admin.
- EP-002: Rate limiting prevents simulated brute-force attacks in load tests; false positive rate <2% in trial.
- EP-003: Reset flow success rate >98% for valid requests; admin unlock actions audited.
- EP-004: Audit event coverage >99%; retention policy enforced and privacy audit completed.
- EP-005: Risk scoring accuracy and operational cost evaluated during pilot; false positive/negative rates within agreed thresholds (to be defined during pilot).

---

## Dependency Graph (sequence & blocking)
- EP-TECH (start) --> EP-DATA (depends on infra for secrets/KMS) --> EP-001 (auth core)  
- EP-DATA --> EP-002 (account protection depends on persisted counters)  
- EP-DATA --> EP-003 (reset tokens stored)  
- EP-TECH --> EP-004 (observability and log export)  
- EP-002 & EP-001 --> EP-005 (AI risk module integrates with core and protection flows)

ASCII dependency graph:
EP-TECH
  |
  v
EP-DATA
  |   \
  v    v
EP-001  EP-003
  |
  v
EP-002
  \
   v
  EP-005
EP-TECH -> EP-004 (observability)
EP-DATA -> EP-002, EP-003
EP-001 -> EP-002 -> EP-005

Notes:
- EP-TECH is a strict prerequisite for production-grade deployments.
- EP-DATA must be delivered before core persistence-dependent features are validated end-to-end.

---

## Timeline Overview (high-level, sequential sprints)
Assumptions:
- Team: 2 backend engineers, 1 frontend engineer, 1 SRE/DevOps, 0.5 QA, 1 Product/PO.
- Sprint length: 2 weeks.
- Parallelization: EP-TECH runs first sprint and continues in parallel where possible; EP-DATA begins in sprint 1 once infra decisions are made.

Suggested plan (estimate):
- Sprint 0 (planning, spike): 1 sprint
  - EP-TECH: decisions (token model, secrets), CI scaffolding kickoff, infra spike.
  - EP-DATA: schema design spike, migration plan.
  - Backlog refinement and API contract finalization.

- Sprint 1:
  - EP-TECH: CI pipeline, dev infra, basic observability.
  - EP-DATA: baseline schema and migrations (dev/staging).
  - EP-001: implement basic /auth/login stub with validation (non-persistent).

- Sprint 2:
  - EP-DATA: complete persistence, encryption, test data.
  - EP-001: integrate password verification and token issuance (with storage/revocation hooks).
  - EP-002: start rate-limit middleware.

- Sprint 3:
  - EP-001: RBAC redirection and logout integration.
  - EP-002: account lockout implementation and CAPTCHA integration points.
  - EP-004: wire audit event emission to logging infra.

- Sprint 4:
  - EP-003: complete reset flow and admin unlock endpoints.
  - EP-004: dashboards and retention policy enforcement.
  - EP-TECH: production readiness gating and security scans.

- Sprint 5:
  - EP-002: harden throttling, run attack simulations and tuning.
  - EP-001/EP-003: full E2E tests, accessibility audits, performance tuning.
  - Optional EP-005: pilot (if prioritized) — risk scoring prototype.

- Rollout:
  - Staged rollout to canary/limited users (1 sprint).
  - Monitor metrics; iterate on thresholds and policies.
Estimated total time to MVP (core auth + data + protection + audit): 10-12 weeks (5-6 sprints), with EP-TECH and observability enabling safe production rollout. Optional AI risk pilot: +2-3 sprints.

---

## T-Shirt Sizing (effort estimate)
- EP-TECH: L
- EP-DATA: L
- EP-001 (Authentication Core): M
- EP-002 (Account Protection & Abuse Mitigation): M
- EP-003 (Recovery & Administration): S
- EP-004 (Audit, Logging & Privacy Compliance): M
- EP-005 (Optional Risk Scoring AI): XS

Sizing rationale:
- EP-TECH and EP-DATA cover cross-cutting and infra work requiring more coordination and QA.
- Feature epics scoped to ~1-3 sprints each as M/S where possible.
- EP-005 is optional pilot sized small for initial MVP.

---

## Acceptance Criteria Summary (per epic, condensed)
- EP-TECH: CI/CD pipelines pass; TLS enforced; secrets stored in Vault/KMS; observability exports present.
- EP-DATA: Schema and migrations applied on staging; sensitive columns encrypted; reset/token persistence validated.
- EP-001: