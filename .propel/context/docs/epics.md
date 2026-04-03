---
post_title: "Epics for Authentication Service"
author1: "Senior Product Owner"
post_slug: "auth-service-epics"
microsoft_alias: "sprod"
featured_image: "N/A"
categories: ["Authentication", "Security", "Platform"]
tags: ["epics", "authentication", "security", "platform", "brown-field"]
ai_note: "AI-assisted"
summary: "Decomposition of authentication feature set into epics with full requirement-to-epic mapping for a brown-field project."
post_date: "2026-04-03"
---

## Rules Used by the Workflow
- Zero orphaned requirements: every requirement ID mapped to exactly one epic.
- Brown-field rules applied: do NOT create EP-TECH (no EP-TECH included).
- Business-value ordering: epics ordered by business impact then dependency priority.
- Manageable scope: keep ~12 requirements max per epic.
- [UNCLEAR] handling: move any unclear items to Backlog Refinement Required (none present).
- UI impact determination: UI Impact = Yes only when figma_spec.md UXR/SCR items exist (none present).
- Dependency mapping: list dependent EPIC IDs only where they block start.

## Executive Summary
This document decomposes the provided authentication specification into seven epics for a brown-field project. It assigns all FR-, NFR-, and UC- requirement IDs to exactly one epic each, groups related functionality by business outcome, and orders epics by business value and dependency priority. EP-001 (Authentication Experience & RBAC) is prioritized for immediate user value; EP-006 (Platform Security, Compliance & Observability) is a high-priority enabler for secure production deployment. Optional AI risk scoring is isolated as EP-007.

## Validation Snapshot

| Criterion | Score (1-5) |
|----------:|------------:|
| Requirement Coverage | 5 |
| Consistency (no duplicates) | 5 |
| Epic Sizing (<= ~12 reqs) | 5 |
| Dependency Clarity | 4 |

Average Score: 4.75

Evaluation summary: All 28 identified requirements (FR-014, NFR-008, UC-006 totals) are mapped exactly once; epics are sized and ordered for incremental delivery. Two design decisions (token model and lockout semantics) remain prioritized for clarification.

## Epic Summary Table (with columns: Epic ID | Epic Title | Mapped Requirement IDs)
| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-001 | Authentication Experience & RBAC | FR-001, FR-002, FR-005, FR-008, UC-001, UC-002, UC-003 |
| EP-006 | Platform Security, Compliance & Observability | NFR-001, NFR-002, NFR-003, NFR-004, NFR-005, NFR-006, NFR-007, NFR-008, FR-012 |
| EP-004 | Session & Token Management | FR-004, FR-011, UC-006 |
| EP-002 | Credentials & Account Recovery | FR-003, FR-007, UC-005 |
| EP-003 | Account Security & Abuse Protections | FR-006, FR-010, UC-004 |
| EP-005 | Admin Tools & Audit | FR-009, FR-013 |
| EP-007 | Optional Risk-Based Assessment Module | FR-014 |

## Epic Description

### EP-001: Authentication Experience & RBAC
**Business Value**: Enables user access to the product by providing a reliable, clear, and role-aware authentication flow.

**Description**: Deliver the primary user-facing authentication flows: secure email/password login, client/server validation, non-revealing error messaging, and role-based redirect/authorization enforcement. Includes happy and error paths per UC-001, UC-002, UC-003 and integrates with token issuance produced by Session & Token Management.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Implement POST /auth/login endpoint with input validation and constant-time password verification.
- Client-side form validation for email format and non-empty password.
- Server returns structured 400 for malformed input; 401 for invalid credentials; clear "Account is locked" for locked accounts.
- Role-to-dashboard redirect mapping and server-side RBAC enforcement on protected endpoints.
- End-to-end tests for login success/failure paths and role-based redirect behavior.
- Integration points documented for token returned on successful login.

**Dependent EPICs**: EP-006, EP-004

### EP-006: Platform Security, Compliance & Observability
**Business Value**: Provides essential platform-level security, secret management, observability, and compliance controls required for safe production operations.

**Description**: Implement and configure platform capabilities: TLS enforcement and headers, secret storage (vault/KMS), DB encryption at rest, logging/metrics pipeline, monitoring dashboards/alerts, accessibility standards, CI testability, and performance/availability scaffolding required by the authentication service.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Enforce TLS 1.2+ with HSTS and security headers in environments; automated checks to fail on HTTP exposure.
- Integrate with vault/KMS for signing keys, secrets, and credential storage; document rotation procedures.
- Enable DB encryption for sensitive columns per platform capability.
- Provide centralized logging ingestion and retention configuration; dashboards for auth SLIs (login success/failure, lock events).
- Define SLOs/SLIs and performance test harness to validate median <1s; autoscaling patterns for availability.
- Accessibility guidance for login/reset pages (WCAG 2.1 AA) and inclusion in test suites.
- CI pipeline configuration ensuring unit/integration/e2e tests run and enforce NFR-008.

**Dependent EPICs**: (none)

### EP-004: Session & Token Management
**Business Value**: Ensures session security and control over authentication lifecycles, reducing risk of token abuse and enabling immediate revocation.

**Description**: Define and implement the token model (JWT vs. opaque), token issuance with configurable TTL, enforcement of token expiry, and server-side revocation strategy for logout. Provide APIs and tests to ensure tokens are rejected after expiry and revoked on logout.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Token issuance implementation with configurable TTL (default 30 minutes).
- Token expiry enforcement and automated tests validating expired tokens receive 401.
- Logout endpoint that revokes active tokens; implement revocation strategy (revocation list or token versioning).
- Documentation of chosen token model, security trade-offs, and revocation mechanics.
- Integration tests for token lifecycle (issue → use → expiry → revocation).
- Operational playbook for token revocation incidents.

**Dependent EPICs**: EP-006

### EP-002: Credentials & Account Recovery
**Business Value**: Protects user credentials and enables secure self-service account recovery to minimize user friction and support operational continuity.

**Description**: Implement secure password storage using configured slow hash (Argon2id or bcrypt), and build the forgot-password / reset workflow using single-use expiring tokens delivered via SMTP, ensuring rate-limiting and non-disclosure behavior.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Implement password hashing and storage using configurable parameters; migration plan for legacy hashes if needed.
- Build /auth/reset-request, /auth/reset-validate, and /auth/reset-complete endpoints with single-use reset tokens (default TTL 1 hour).
- Ensure reset requests receive non-disclosing responses and are rate-limited.
- Invalidate reset token immediately after successful use; tests for single-use and expiry behavior.
- SMTP integration for reset email delivery and templating guidance.
- Unit/integration tests covering reset flow and hashing verification.

**Dependent EPICs**: EP-006

### EP-003: Account Security & Abuse Protections
**Business Value**: Reduces account compromise risk and operational load by preventing brute-force and credential-stuffing attacks.

**Description**: Implement configurable account lockout after consecutive failed attempts, per-account and per-IP rate limiting, exponential backoff and CAPTCHA insertion hooks, and enforcement of lockout behavior in authentication flows.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Implement failed-attempt counters and lock marking after configurable threshold (default 5).
- Return appropriate responses for locked accounts (423 or 401 with "Account is locked").
- Per-IP rate limits and per-account throttling with configurable thresholds.
- CAPTCHA integration points and policy for insertion after configurable thresholds.
- Tests validating lock triggers, prevention of login when locked, and combined per-IP/account behavior under simulated attacks.
- Instrumentation for metrics and alerting on brute-force patterns.

**Dependent EPICs**: EP-006

### EP-005: Admin Tools & Audit
**Business Value**: Provides operational control and auditability needed for compliance, incident response, and account recovery support.

**Description**: Emit structured audit logs for authentication events, provide admin endpoint(s) to unlock accounts and view lockout history, and ensure admin actions are authenticated, authorized, and logged.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Emit audit events for login success/failure, account lock, password reset request/completion, and unlock events with required fields (timestamp, user-id, IP, outcome).
- Implement admin-only unlock API with authentication and logging of admin actions.
- Define audit log schema and integration with centralized logging pipeline (provided by EP-006).
- End-to-end tests validating audit event emission and admin unlock audit entry creation.
- Operational guidance for administrators to view lockout history securely.

**Dependent EPICs**: EP-006

### EP-007: Optional Risk-Based Assessment Module
**Business Value**: Improves adaptive security decisions by scoring login risk and enabling dynamic controls (opt-in, non-blocking to initial rollout).

**Description**: Provide an opt-in module that scores login attempts (IP reputation, velocity, heuristics) and maps scores to deterministic adaptive actions (e.g., require CAPTCHA, deny, flag). Module is isolated and optional; not required for initial authentication MVP.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Pluggable risk scoring service producing numeric score per attempt and deterministic action mapping for at least three ranges.
- Configurable policies to apply adaptive controls (CAPTCHA, additional verification).
- Logging and manual review pathway for flagged attempts.
- Tests verifying score ranges and resulting actions.
- Documentation for opt-in configuration and data retention for risk signals.

**Dependent EPICs**: EP-003, EP-005

## Backlog Refinement Required
- None. All provided requirements had clear intent and were mapped. If any of the following decisions remain unresolved, create refinement tickets before implementation:
  - Token model decision (JWT vs opaque tokens) — impacts EP-004 implementation details.
  - Lockout semantics (temporary auto-unlock duration vs admin-only unlock) — impacts EP-003 and EP-005 behavior.
  - Password complexity policy and migration plan for legacy hashes — impacts EP-002.
  - Confirmation of infrastructure responsibilities for TLS termination (load balancer vs service) — impacts EP-006.

