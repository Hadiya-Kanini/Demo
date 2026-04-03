---
post_title: "Epics for Authentication Service"
author1: "Senior Product Owner"
post_slug: "auth-service-epics"
microsoft_alias: "s_po"
featured_image: "N/A"
categories: ["Security","Auth","Platform"]
tags: ["epics","authentication","security","compliance"]
ai_note: "Assisted"
summary: "Epic decomposition for the Authentication Service covering all functional, non-functional, and use-case requirements in spec.md. Brown-field project; EP-TECH not included; all requirements mapped without orphans."
post_date: "2026-04-03"
---

## Rules Used by the Workflow
- Zero Orphaned Requirements: every requirement ID must map to exactly one epic.
- Group by business outcome (feature area) rather than technical layer.
- Epic sizing: target ~5–12 requirements; split if exceeded.
- Brown-field project detected → do NOT include EP-TECH.
- EP-DATA not triggered (no DR-XXX ≥ 2 or entity file) → data requirements embedded in feature epics.
- Skip and list [UNCLEAR] requirements in Backlog Refinement Required (none present here).
- Order epics by business value then dependency priority.
- UI Impact flagged only when figma_spec.md / UXR-XXX exist (none present → UI Impact = No; Screen References = N/A).
- Every UC-XXX appears exactly once and mapped to the functional epic that owns the flow.

## Executive Summary
This document decomposes the Authentication Service specification into nine epics that collectively cover all 28 requirement IDs (14 FR, 8 NFR, 6 UC). Project type: brown-field. Top priority focuses on Authentication Core and Password Management to enable secure login and recovery; Security & Compliance (TLS, secrets, encryption) is treated as a high-priority cross-cutting epic. An optional risk-based module is isolated as a separate epic. All requirements are assigned to exactly one epic and epic sizes comply with the ~5–12 requirement guideline.

## Validation Snapshot
- Total requirements discovered: 28
  - Functional (FR): 14 (FR-001..FR-014)
  - Non-Functional (NFR): 8 (NFR-001..NFR-008)
  - Use Cases (UC): 6 (UC-001..UC-006)
- [UNCLEAR] requirements: 0
- EP-TECH included: No (brown-field)
- EP-DATA included: No (not triggered)
- All requirements mapped: Yes — each requirement appears in exactly one epic.
- Epic sizing: All epics ≤ 12 requirements.
- Priority ordering applied: business value → dependency.

## Epic Summary Table
| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-001 | Authentication Core | FR-001, FR-002, FR-004, FR-005, FR-008, UC-001, UC-003, NFR-003, NFR-007 |
| EP-002 | Password Management & Reset | FR-003, FR-007, UC-005, NFR-002, NFR-006 |
| EP-007 | Security & Compliance Controls | FR-012, NFR-001 |
| EP-003 | Abuse Protection & Lockout | FR-006, FR-010, UC-002 |
| EP-006 | Session Management & Revocation | FR-011, UC-006 |
| EP-004 | Observability & Auditing | FR-009, NFR-005 |
| EP-005 | Admin Operations (Unlock & History) | FR-013, UC-004 |
| EP-008 | Platform Reliability & Testability | NFR-004, NFR-008 |
| EP-009 | Optional Risk-based Adaptive Auth (Opt-in) | FR-014 |

## Epic Description

### EP-001: Authentication Core
**Business Value**: Enables users to securely authenticate and reach role-specific workflows; foundational for all user-facing functionality and downstream features.

**Description**: Implement core email/password authentication flow including client and server validation, constant-time password verification against secure hashes, authentication token issuance with configurable TTL, role-based redirect target resolution, and non-revealing error messaging. Meet performance and accessibility NFRs for login flows.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/login API and integration tests.
- Client-side field validation logic and server-side validation responses (400 structured errors).
- Password verification logic (constant-time compare) integrated into login flow.
- Token issuance with expiry claim and redirect-target payload.
- Generic failure and locked account messaging behavior.
- Performance tuning to meet median <1s and accessibility baseline (WCAG 2.1 AA audits).
- Unit, integration, and E2E tests for happy path and error scenarios.

**Dependent EPICs**: EP-007

### EP-002: Password Management & Reset
**Business Value**: Protects credentials and provides safe recovery; reduces security risk and supports compliance.

**Description**: Design and implement secure password storage (Argon2id or bcrypt with configurable parameters), and the forgot-password / reset flow using single-use expiring tokens delivered via SMTP. Implement privacy-safe messaging and rate-limiting for reset requests.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- DB column and data model changes for password_hash, salt metadata, failed_attempts fields.
- Hashing library integration and configuration management for hashing parameters.
- /auth/reset-request, /auth/reset-validate, /auth/reset-complete endpoints with token generation, single-use invalidation, and TTL enforcement.
- Integration with SMTP provider and templates (privacy-safe).
- Rate-limiting around reset requests; tests for token expiry and single-use behavior.
- Documentation on crypto parameters and migration guidance for legacy hashes.

**Dependent EPICs**: EP-001, EP-007

### EP-007: Security & Compliance Controls
**Business Value**: Ensures secure transport, secret management, encryption at rest, and baseline security configuration required for safe production operation.

**Description**: Enforce TLS-only production traffic, integrate secret storage (vault/KMS) for signing keys and sensitive configuration, implement DB encryption for sensitive columns per platform capabilities, and establish key rotation practices and security headers (HSTS, CSP, X-Content-Type-Options).

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- TLS enforcement checks and automated validation in CI/CD.
- Vault/KMS integration for secrets and key rotation procedures.
- Database encryption configuration for sensitive columns and documentation.
- Security headers and platform-level hardening configuration.
- Security acceptance tests and audit checklist.

**Dependent EPICs**: (none)

### EP-003: Abuse Protection & Lockout
**Business Value**: Reduces account compromise risk and operational impact from brute-force and credential-stuffing attacks.

**Description**: Implement account lockout after configurable failed attempts, per-account and per-IP throttling, exponential backoff, support for optional CAPTCHA insertion points, and ensure lock behavior returns appropriate error codes/messages. Emit lock events to audit logs.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Failed-attempts counter and lock/unlock state machine.
- Configurable lock thresholds, windows, and lock duration behavior.
- Per-IP and per-account rate-limiting middleware/component (configurable).
- CAPTCHA integration hook and configuration.
- Tests validating lock triggers, prevention of login while locked, and throttling under simulated attack.

**Dependent EPICs**: EP-001, EP-007

### EP-006: Session Management & Revocation
**Business Value**: Enables secure logout and immediate invalidation of active credentials to reduce risk from leaked tokens.

**Description**: Provide logout endpoint and server-side revocation strategy (revocation list, token versioning, or opaque token store) to ensure tokens are invalidated immediately. Define and test revocation semantics (idempotency, edge cases with expired tokens).

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- POST /auth/logout endpoint and server-side revocation mechanism.
- If JWT used: implementation of revocation list or token versioning strategy; if opaque: session store integration.
- Tests proving logout causes immediate 401 on subsequent requests.
- Documentation on token model decision and operational implications.

**Dependent EPICs**: EP-001, EP-002

### EP-004: Observability & Auditing
**Business Value**: Provides auditability required for security/compliance and operational visibility for incident detection and response.

**Description**: Emit structured audit logs for authentication events (successful login, failed attempt, lock, reset request/completion, logout), export logs to centralized logging, create metrics and alerts for abnormal failed attempt spikes, and provide dashboards for SRE and security stakeholders.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Structured audit log schema and integration with centralized logging.
- Metrics (success rate, failure rate, lock events) and dashboards.
- Alert rules for abnormal activity (configurable thresholds).
- Integration tests verifying events emitted for key flows.

**Dependent EPICs**: EP-001

### EP-005: Admin Operations (Unlock & History)
**Business Value**: Enables support and security teams to remediate locked accounts and review audit history, lowering time-to-resolution and improving compliance evidence.

**Description**: Implement authenticated admin endpoint(s) for unlocking accounts and retrieving lockout/audit history. Ensure admin operations are properly authorized (admin role), audited, and subject to the same logging/retention policies.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Admin API for unlocking accounts and query endpoints for lockout history.
- RBAC enforcement for admin endpoints with audit logging on unlock actions.
- Admin operation tests verifying authorization, audit entry creation, and unlock behavior.
- Support docs for admin workflows.

**Dependent EPICs**: EP-003, EP-004

### EP-008: Platform Reliability & Testability
**Business Value**: Ensures the authentication service meets SLOs, is horizontally scalable, and is verifiable via CI gates; required for production readiness.

**Description**: Implement horizontal scalability patterns for the auth service (stateless design, session store HA if used), define SLIs/SLOs, create load test harnesses to validate performance targets, and include unit/integration/e2e tests in CI to validate key acceptance criteria.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Autoscaling and deployment guidance for stateless service and session store HA.
- Performance/load testing suites and documented baselines.
- CI pipeline inclusion of unit/integration/e2e tests covering lockout, token expiry, reset flows.
- Defined SLIs/SLOs and runbook for incidents.

**Dependent EPICs**: EP-001, EP-007

### EP-009: Optional Risk-based Adaptive Auth (Opt-in)
**Business Value**: Offers advanced risk scoring to adaptively strengthen defenses; high-value enhancement for reducing fraud when enabled.

**Description**: Implement an opt-in risk assessment component that scores login attempts (IP reputation, velocity, heuristics) and suggests deterministic actions (e.g., require CAPTCHA, require additional verification). The module is isolated and not required for initial rollout.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Risk scoring engine interface and configurable thresholds.
- Deterministic action mapping for at least three score ranges.
- Logging and manual review workflow for flagged attempts.
- Tests validating score-to-action mapping and isolation from core flows.

**Dependent EPICs**: EP-004

## Backlog Refinement Required
- No requirements tagged [UNCLEAR] were found in the provided inputs. All discovered requirement IDs have been mapped to epics above.