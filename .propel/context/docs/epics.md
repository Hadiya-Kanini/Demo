---
post_title: "Epics for Authentication Service (Brown-field)"
author1: "Senior Product Owner"
post_slug: "epics-auth-service"
microsoft_alias: "sprod.owner"
featured_image: ""
categories: ["product", "security", "backend"]
tags: ["epics", "authentication", "security", "brown-field", "data"]
ai_note: "Generated with assistance from an AI epic generator"
summary: "Comprehensive epic decomposition for the Authentication Service (brown-field). Maps all FR/NFR/UC requirements to prioritized epics with scope, acceptance criteria, dependencies, sizing, risks, metrics, dependency graph and timeline."
post_date: "2026-04-03"
---

## Rules Used by the Workflow
- ai-assistant-usage-policy (primacy of user directives, minimal output, no decorative glyphs)
- dry-principle-guidelines (single source of truth, delta updates, minimal-impact changes)
- iterative-development-guide (strict phased execution, exact outputs)
- markdown-styleguide (YAML front matter, H2/H3 headings, tables)
- software-architecture-patterns (pattern selection, NFR mapping, trade-offs)
- Core Principles: Zero orphaned requirements, Business Value Ordering, Manageable Scope (~12 reqs/epic)

## Executive Summary
This document decomposes the provided authentication specification into a set of prioritized, traceable epics for a brown-field project. No EP-TECH is created (brown-field). EP-DATA is included as a critical early epic. Every FR-, NFR- and UC- requirement from the spec is mapped to exactly one epic. The plan targets an MVP delivering secure email/password login, RBAC enforcement, account protection, password reset, token revocation and required infra hardening, with an optional risk-scoring module deferred.

## Validation Snapshot
- Project type: Brown-field (integrates with existing user datastore; migration considerations present)
- [UNCLEAR] requirements: None detected
- EP-TECH: Not included (brown-field)
- EP-DATA: Included (schema changes, encryption, migration)
- All FR/NFR/UC mapped: Yes (zero orphaned requirements)

## Epic Summary Table
| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-DATA | Data Layer & Schema Migration | FR-003, NFR-002, NFR-006 |
| EP-AUTH-CORE | Authentication Core & RBAC | FR-001, FR-004, FR-005, NFR-003, UC-001 |
| EP-VALIDATION-UX | Client & Server Validation, UX & Accessibility | FR-002, FR-008, NFR-007, UC-003 |
| EP-PROTECTION | Rate-limiting, Lockout & Anti-Abuse | FR-006, FR-010, UC-002, UC-004 |
| EP-PASSWORD-RESET | Password Reset & Email Integration | FR-007, UC-005 |
| EP-TOKEN-REVOCATION | Logout & Token Revocation | FR-011, UC-006 |
| EP-AUDIT-ADMIN | Audit Logging & Admin Operations | FR-009, FR-013, NFR-005 |
| EP-INFRA-NFRS | Infrastructure & Security Baseline | FR-012, NFR-001, NFR-004, NFR-008 |
| EP-OPTIONAL-RISK | Optional Risk Scoring Module (opt-in) | FR-014 |

Notes:
1. Epics are ordered by business value and blocking dependencies.
2. Each requirement appears in exactly one epic; [UNCLEAR] items would be placed in Backlog Refinement if present.

## Epic Mapped Requirement Counts
| Epic ID | Req Count |
|---------|-----------|
| EP-DATA | 3 |
| EP-AUTH-CORE | 5 |
| EP-VALIDATION-UX | 4 |
| EP-PROTECTION | 4 |
| EP-PASSWORD-RESET | 2 |
| EP-TOKEN-REVOCATION | 2 |
| EP-AUDIT-ADMIN | 3 |
| EP-INFRA-NFRS | 4 |
| EP-OPTIONAL-RISK | 1 |

## Evaluation Scores
| Criterion | Score (1-5) |
|-----------|-------------|
| Requirement Coverage | 5 |
| Traceability (one-to-one mapping) | 5 |
| Dependency Completeness | 5 |
| Epic Sizing & Split Accuracy | 4 |
| Risk Assessment Coverage | 4 |
| Average Score | 4.6 |

Evaluation summary:
All functional, non-functional and use-case requirements are mapped with traceability and dependency coverage. Sizing and risk coverage are conservative; recommended refinement during sprint planning for EP-DATA migration complexity and token revocation design decisions.

---

## Detailed Epics

### EP-DATA: Data Layer & Schema Migration
**Business Value**: Enables persistent, secure storage for authentication entities and supports all features requiring data (password hashes, lock state, token versioning, reset tokens).

**Description**: Implement required database schema changes and migration scripts to support secure password storage, account lock metadata, token/versioning fields, reset token references, and encryption-at-rest for sensitive fields. Provide migration plan for legacy hashes and test datasets for staging.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- DB schema changes: add password_hash, hash_params, failed_attempts, locked_until, token_version, reset_token references (or table).
- Migration scripts and rollback procedures for legacy password hashes.
- DB encryption configuration for sensitive columns and documentation of key rotation.
- Seed/mock data for testing and CI.
- Integration with secret store (vault/KMS) for encryption keys.
- Unit/integration migration tests and verification tooling.

**In-scope**:
- Schema design and migrations for authentication fields.
- Encryption-at-rest enabling where supported.
- Migration verification and staging runbook.

**Out-of-scope**:
- Enterprise-wide user data migration beyond authentication footprint (flag as separate migration epic if required).
- Frontend changes.

**Acceptance Criteria**:
- Schema deployed to staging with migration run and automated verification showing no data loss.
- All sensitive columns encrypted or documented as platform-limited; keys managed in KMS/vault.
- Migration automated with rollback path and runbook.
- Integration tests validate password_hash presence and token_version behavior.

**Dependent EPICs**: EP-AUTH-CORE, EP-PROTECTION, EP-PASSWORD-RESET, EP-TOKEN-REVOCATION, EP-AUDIT-ADMIN

**T-shirt Sizing**: L (Complexity due to migrations, key management, encryption)

**Risk Assessment**:
- Risk: Data loss or inconsistent migration for legacy hashes.
  - Mitigation: Staging dry-runs, incremental migrations, backups, and automated verification checks.
- Risk: Key management misconfiguration.
  - Mitigation: Use managed KMS/vault patterns and rotate keys in staging before prod.

**Success Metrics**:
- Migration executed on staging with 0% critical errors and <0.1% data inconsistencies.
- Sensitive columns encrypted and key rotation documented.
- Migration rollback tested and successful in staging.

---

### EP-AUTH-CORE: Authentication Core & RBAC
**Business Value**: Delivers the core user-facing and API authentication functionality enabling users to sign in and access role-specific application surfaces.

**Description**: Implement /auth/login endpoint with secure password verification, authentication token issuance (TTL), role-based redirect, and server-side RBAC enforcement for protected APIs. Include unit and integration tests; meet latency NFRs.

**UI Impact**: Yes (login flow integration)

**Screen References**: SCR-N/A (coordinate with frontend)

**Key Deliverables**:
- /auth/login API with secure constant-time verify of password hashes.
- Token issuance mechanism with configurable TTL (default 30m).
- Role-to-redirect mapping and integration test for at least three roles.
- Server-side RBAC middleware to protect endpoints.
- Performance tests validating NFR-003.

**In-scope**:
- Login endpoint and RBAC enforcement.
- Token format decision (JWT vs opaque) documented and implemented as agreed.

**Out-of-scope**:
- Refresh token implementation (deferred unless confirmed).
- MFA/SSO integrations.

**Acceptance Criteria**:
- Given valid credentials, POST /auth/login returns 200 with token and redirect target; processing median < 1s.
- Given invalid credentials, returns 401 "Invalid credentials".
- Token contains expiry or server enforces TTL; expired tokens rejected (401).
- RBAC enforced: protected endpoints return 403 for insufficient roles (integration tests for customer, employee, admin).

**Dependent EPICs**: EP-DATA (must exist for storage), EP-INFRA-NFRS (TLS, secrets), EP-VALIDATION-UX (frontend validation contract)

**T-shirt Sizing**: M

**Risk Assessment**:
- Risk: Token model choice (JWT vs opaque) impacts revocation complexity.
  - Mitigation: Document trade-offs; if JWT chosen, implement token versioning/revocation pattern in EP-TOKEN-REVOCATION.
- Risk: Performance breach of NFR-003.
  - Mitigation: Profiling, caching where appropriate, load testing.

**Success Metrics**:
- Authentication success rate ≥ 99% for valid credentials.
- Median login processing < 1s; P95 < 2s under baseline load.
- RBAC tests pass for defined roles.

---

### EP-VALIDATION-UX: Client & Server Validation, UX & Accessibility
**Business Value**: Reduces friction and security exposure by enforcing consistent validation and accessible messaging across login/reset flows.

**Description**: Implement client-side validation, server-side structured validation errors, and standardized, non-revealing messages. Ensure login and reset pages meet WCAG 2.1 AA requirements.

**UI Impact**: Yes

**Screen References**: SCR-N/A (use figma assets if available)

**Key Deliverables**:
- Client-side validation rules and messages (email format, max lengths).
- Server-side validation API responses (400 structured errors).
- Accessibility fixes and audit report (WCAG 2.1 AA).
- Error message templates: "Invalid credentials", "Account is locked", "If an account exists, a reset link has been sent."

**In-scope**:
- Form validation on login and reset pages and server validation handlers.

**Out-of-scope**:
- Full UI redesign.

**Acceptance Criteria**:
- Client prevents submission on empty/malformed fields.
- Server returns structured 400 validation errors when bypassed.
- Accessibility audit passes with no critical failures.

**Dependent EPICs**: EP-AUTH-CORE, EP-PASSWORD-RESET

**T-shirt Sizing**: S

**Risk Assessment**:
- Risk: Accessibility gaps leading to compliance issues.
  - Mitigation: Perform accessibility audit early and include fixes in same sprint.

**Success Metrics**:
- Accessibility audit: no critical failures.
- Validation unit tests cover missing fields, invalid formats, long values.

---

### EP-PROTECTION: Rate-limiting, Lockout & Anti-Abuse
**Business Value**: Protects user accounts and system stability by preventing credential-stuffing and brute-force attacks.

**Description**: Implement per-account failed-attempt counters, configurable lockout window (default 5 attempts / 15 minutes), per-IP and per-account rate limiting, exponential backoff, and CAPTCHA insertion hooks. Integrate counters with DB (EP-DATA) and logging (EP-AUDIT-ADMIN).

**UI Impact**: Minimal (error states and optional CAPTCHA flow on UI)

**Screen References**: SCR-N/A

**Key Deliverables**:
- Failed attempts increment logic and account lock state.
- Configurable lock thresholds and windows.
- Rate limiting middleware (per-IP and per-account).
- CAPTCHA insertion points and config toggles.
- Exponential backoff mechanics for retry behavior.

**In-scope**:
- Server-side throttling and lockout behavior.
- Hooks for CAPTCHA; integration deferred to frontend.

**Out-of-scope**:
- Integration with external anti-abuse vendors (unless chosen later).

**Acceptance Criteria**:
- After N failed attempts (default 5), account locked and further attempts return 423 or approved 401 with "Account is locked".
- Per-IP hitting threshold enforces throttling.
- CAPTCHA insertion supported and configurable.
- Tests validate lockout, throttling, and exponential backoff.

**Dependent EPICs**: EP-DATA, EP-AUTH-CORE, EP-AUDIT-ADMIN

**T-shirt Sizing**: M

**Risk Assessment**:
- Risk: Lockout abuse causing denial of service for legitimate users.
  - Mitigation: Combine per-IP and per-account rules; provide admin unlock and self-service reset paths; monitor lockout rate.
- Risk: Misconfiguration of rate limits.
  - Mitigation: Default conservative thresholds and telemetry.

**Success Metrics**:
- Lockout triggers reliably after configured failed attempts.
- False-positive lockouts under normal traffic < 0.1%.
- Rate-limiting behavior validated under simulated attack scenarios.

---

### EP-PASSWORD-RESET: Password Reset & Email Integration
**Business Value**: Enables account recovery without disclosing account existence; essential for user retention.

**Description**: Implement reset-request and reset-complete flows with single-use expiring tokens (default 1 hour), SMTP/email provider integration, rate-limiting of reset requests, and token invalidation after use.

**UI Impact**: Yes (reset request & reset pages)

**Screen References**: SCR-N/A

**Key Deliverables**:
- /auth/reset-request and /auth/reset-validate endpoints.
- Secure, single-use, unguessable reset tokens stored or verifiable.
- Email templates and SMTP integration, including delivery retry logic.
- Rate-limiting on reset requests and non-revealing response text.

**In-scope**:
- Token generation/invalidation, email sending, token validation, password update flow.

**Out-of-scope**:
- SMS or alternative channels.

**Acceptance Criteria**:
- Reset tokens expire after TTL and are single-use.
- Reset request responses do not leak existence of account.
- Tests validate token expiry and single-use behavior.

**Dependent EPICs**: EP-DATA, EP-PROTECTION, EP-VALIDATION-UX

**T-shirt Sizing**: M

**Risk Assessment**:
- Risk: Email deliverability failures causing support tickets.
  - Mitigation: Use reliable SMTP provider, retries, and monitoring.
- Risk: Token leakage.
  - Mitigation: Use unguessable tokens, encrypt storage, enforce single-use.

**Success Metrics**:
- Reset tokens invalid after first use and after TTL.
- Reset request rate-limited under attack simulations.
- User self-service success rate > 95% for valid reset attempts.

---

### EP-TOKEN-REVOCATION: Logout & Token Revocation
**Business Value**: Ensures immediate revocation of credentials to limit exposure and meet security expectations.

**Description**: Provide logout endpoint and server-side revocation strategy: revocation list, token versioning, or hybrid. Implement tests ensuring revoked tokens are rejected.

**UI Impact**: Minimal (logout action)

**Screen References**: N/A

**Key Deliverables**:
- /auth/logout endpoint that invalidates tokens.
- Chosen revocation approach implemented and documented.
- Tests verifying immediate invalidation and idempotency.

**In-scope**:
- Revocation logic and storage of revocation metadata.

**Out-of-scope**:
- Long-lived refresh token rotation flows (unless accepted).

**Acceptance Criteria**:
- Logout invalidates token such that subsequent requests return 401.
- If JWTs used, versioning or revocation list ensures immediate invalidation.
- Tests validate behavior including idempotent logout.

**Dependent EPICs**: EP-DATA, EP-AUTH-CORE, EP-AUDIT-ADMIN

**T-shirt Sizing**: S-M (depends on revocation strategy)

**Risk Assessment**:
- Risk: Performance cost of revocation list checks.
  - Mitigation: Use efficient stores (Redis) and token versioning to minimize lookups.

**Success Metrics**:
- Revoked tokens rejected immediately in tests.
- Logout endpoints idempotent and < 500ms median response.

---

### EP-AUDIT-ADMIN: Audit Logging & Admin Operations
**Business Value**: Enables operational visibility and compliance via audit trails and admin account management.

**Description**: Emit structured audit events (login success/failure, lock events, resets, unlocks), export logs to centralized logging, and provide admin API/UI for unlocking accounts and viewing lockout history. Ensure retention and privacy policies are enforced.

**UI Impact**: Yes (admin UI/API)

**Screen References**: N/A

**Key Deliverables**:
- Audit event schema and log export pipeline.
- Admin endpoint/UI for account unlock and lockout history.
- Dashboards/alerts for abnormal failed attempt spikes.
- Retention and privacy handling documentation.

**In-scope**:
- Audit emission, admin unlock API, dashboards and alerts.

**Out-of-scope**:
- Full SIEM onboarding beyond log export.

**Acceptance Criteria**:
- Audit events include timestamp, user-id/identifier, source IP, outcome, event type.
- Admin unlock requires admin authentication and logs unlock event.
- Dashboards and alerts for failed attempt spikes in place.

**Dependent EPICs**: EP-DATA, EP-INFRA-NFRS, EP-PROTECTION

**T-shirt Sizing**: M

**Risk Assessment**:
- Risk: Audit log volume and retention costs.
  - Mitigation: Define retention policy and log sampling for high-volume events.
- Risk: Unauthorized admin access.
  - Mitigation: Enforce strong admin RBAC and audit every admin action.

**Success Metrics**:
- 100% of defined auth events emitted to logging pipeline.
- Admin unlock actions require admin auth and are audited.
- Alerts trigger on configured thresholds and are verified in tests.

---

### EP-INFRA-NFRS: Infrastructure & Security Baseline
**Business Value**: Provides environment and security controls required for production deployment and compliance (TLS, secrets, scalability, CI test gates).

**Description**: Enforce TLS in production, configure security headers, integrate secrets with KMS/vault, enable autoscaling patterns and service SLOs, add CI checks for HTTP exposure and include test coverage enforcement.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- TLS enforcement, HSTS, CSP and other recommended headers applied.
- Secrets integration with vault/KMS and key rotation runbook.
- Autoscaling configuration and SLIs/SLOs for auth service.
- CI checks gating plaintext HTTP exposure and test coverage.
- Test harness for integration and load tests.

**In-scope**:
- Platform-level security and SRE collaboration activities.

**Out-of-scope**:
- Platform-wide upgrades not necessary for auth features.

**Acceptance Criteria**:
- Production endpoints only accessible via HTTPS.
- Secrets stored in vault/KMS and accessible for staging and prod.
- CI pipeline fails if HTTP endpoints detected.
- Autoscaling validated via load tests.

**Dependent EPICs**: EP-DATA, EP-AUTH-CORE, EP-AUDIT-ADMIN

**T-shirt Sizing**: M

**Risk Assessment**:
- Risk: TLS misconfiguration or exposure.
  - Mitigation: CI checks and external scans before release.
- Risk: Secret access errors.
  - Mitigation: IAM policies and staged key rotation.

**Success Metrics**:
- 0 HTTP endpoints in production.
- Secrets managed via vault with rotation procedures documented.
- Load tests demonstrate autoscaling without error.

---

### EP-OPTIONAL-RISK: Optional Risk Scoring Module (opt-in)
**Business Value**: Provides advanced adaptive controls (CAPTCHA/MFA triggers) based on risk scoring to reduce fraud; optional and deferred.

**Description**: Create an opt-in service to compute risk scores (IP reputation, velocity) and map scores to actions (allow, require CAPTCHA, block). Provide deterministic mapping and manual review interface.

**UI Impact**: Yes (administration and optional front-end hooks)

**Screen References**: N/A

**Key Deliverables**:
- Risk scoring module (POC) with deterministic score buckets.
- Integration hooks for EP-PROTECTION to trigger CAPTCHA/MFA.
- Manual review UI for flagged attempts.

**In-scope**:
- POC risk scoring logic and deterministic action mapping.

**Out-of-scope**:
- ML model training and production ML pipelines.

**Acceptance Criteria**:
- Risk scores generated per attempt and actions executed for three score bands.
- Manual review available for flagged events.

**Dependent EPICs**: EP-PROTECTION, EP-AUDIT-ADMIN

**T-shirt Sizing**: S

**Risk Assessment**:
- Risk: Over-blocking legitimate users.
  - Mitigation: Conservative thresholds and manual review process; start as opt-in POC.

**Success Metrics**:
- POC accurately classifies test scenarios into expected buckets.
- Integration tests validate action mapping for each bucket.

---

## Backlog Refinement Required
- None of the requirements are tagged [UNCLEAR] in the provided spec. If token model (JWT vs opaque) and lockout semantics (temporary vs admin-only) decisions are not confirmed during refinement, these will require acceptance decisions before EP-AUTH-CORE and EP-TOKEN-REVOCATION implementation.

---

## Dependency Graph (blocking / must-complete-before)
EP-INFRA-NFRS  ──┐
                 ├─> EP-AUDIT-ADMIN ──> EP-OPTIONAL-RISK
EP-DATA        ──┼─> EP-AUTH-CORE ──> EP-TOKEN-REVOCATION
                 ├─> EP-PROTECTION ──┐
EP-AUTH-CORE   ──┘                   ├─> EP-PASSWORD-RESET
                                     └─> EP-VALIDATION-UX

Textual edges (primary):
- EP-DATA → EP-AUTH-CORE, EP-PROTECTION, EP-PASSWORD-RESET, EP-TOKEN-REVOCATION, EP-AUDIT-ADMIN
- EP-INFRA-NFRS → EP-AUTH-CORE, EP-AUDIT-ADMIN
- EP-AUTH-CORE → EP-TOKEN-REVOCATION
- EP-PROTECTION → EP-PASSWORD-RESET (rate-limiting)
- EP-AUDIT-ADMIN → EP-OPTIONAL-RISK (telemetry consumption)

Notes:
- EP-DATA and EP-INFRA-NFRS should begin in parallel (critical enablers).
- EP-AUTH-CORE is the next critical path after initial infra/data work.

---

## Timeline Overview (8-week delivery window — example plan)
Assumptions: 2-week sprints, cross-functional team(s), EPICs executed in parallel where non-blocking.

| Sprint (2-week) | Week Range | Primary Focus |
|-----------------|------------|---------------|
| Sprint 