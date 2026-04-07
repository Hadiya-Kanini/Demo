---
title: "Epics for Email+Password Authentication Service"
author: "Senior Product Owner"
date: "2026-04-07"
summary: "Complete epic decomposition for a green-field Email+Password Authentication Service. Includes EP-TECH and EP-DATA, maps all requirement IDs from the checklist (UNCLEAR items listed in Backlog Refinement)."
ai_note: "Generated with AI assistance. UNCLEAR-tagged requirements require clarification before implementation."
---

## Epic Summary Table

| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Foundation & Platform (Project scaffolding, CI/CD, security baseline) | TR-001, TR-002, TR-006, TR-008, NFR-001, NFR-002, NFR-003, NFR-007, NFR-008, DR-004 |
| EP-DATA | Data Schema & Persistence Layer | DR-001, DR-002, DR-005 |
| EP-001 | Authentication Core (Login, credential verification, role redirect) | FR-001, FR-002, FR-003, FR-004, FR-005, FR-008, UC-001, UC-002, UC-003, UC-008 |
| EP-002 | Token & Session Management (TTL, revocation, logout) | FR-009, TR-003, UC-007 |
| EP-003 | Password Recovery (Forgot / Reset flow) | FR-007, UC-005, UC-006 |
| EP-004 | Account Protection & Rate-Limit (Lockout, rate limiting, CAPTCHA baseline) | FR-006, FR-011, UC-004, TR-004, NFR-005 |
| EP-005 | Audit & Observability Pipeline (Structured logs, SIEM integration) | FR-010, NFR-004, TR-005, TR-007, DR-003, NFR-006 |
| EP-006 | Multi-Factor Authentication (TOTP enrollment & challenge) | FR-012 |
| EP-007 | UX - Core Interaction & Flows (Primary actions, task reachability, interaction patterns) | UXR-001, UXR-002, UXR-099, UXR-101, UXR-102, UXR-103, UXR-301, UXR-501, UXR-502 |
| EP-008 | UX - Accessibility & Visual Quality (WCAG, tokens, focus states) | UXR-201, UXR-202, UXR-401, UXR-402, UXR-601, UXR-602, UXR-603, UXR-701, UXR-801 |
| EP-009 | Adaptive Detection (Auditable anomaly scoring service for escalations) | AIR-001, AIR-002 |

## Epic Description

### EP-TECH: Foundation & Platform (Project scaffolding, CI/CD, security baseline)
**Business Value**: Enables all subsequent development by establishing the secure, observable, and testable project foundation required for delivery, compliance, and scale.

**Description**: Create the green-field project scaffolding, CI/CD pipelines, runtime platform choices, secrets/KMS integration, baseline security hardening and SAST gates, and operational patterns required to deliver secure authentication features. Implement infrastructure components and operational practices that satisfy latency/availability/NFR constraints, backups and DR, and provide integration points for adapters (email, MFA, CAPTCHA).

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Project repository structure, module/layout and developer README
- CI/CD pipelines with builds, unit/integration test runs, SAST and dependency scanning
- Kubernetes/infra IaC templates or cloud provisioning scripts
- KMS/secret store integration for signing keys and secrets
- Baseline security controls: TLS enforcement, API gateway integration, WAF recommendations
- Operational runbooks, monitoring collector bootstrap, baseline alerting
- Backup & PITR configuration for primary DB (DR-004)
- Platform-level configuration for scalability and testability

**Dependent EPICs**: []

### EP-DATA: Data Schema & Persistence Layer
**Business Value**: Enables durable and consistent storage of users, credentials, reset tokens and revocation records required by all feature epics.

**Description**: Design and implement database schema, migrations, and persistence patterns for user records, hashed reset tokens, token revocation metadata, and durable audit/event exports. Provide migration scripts, constraints and sample seed data to support feature development and testing.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Postgres schema migrations for User, ResetToken, TokenRevocation/AuditEvent
- Constraints, indexes (email uniqueness, token TTL support) and documentation (DR-001, DR-002, DR-005)
- Redis/fast-store design for revocation/atomic counters (integration notes)
- Seed and test data sets for dev/staging
- Data migration and rollback strategy; tooling (Flyway or equivalent)

**Dependent EPICs**: EP-TECH

### EP-001: Authentication Core (Login, credential verification, role redirect)
**Business Value**: Delivers the primary user-facing capability (secure login) that reduces unauthorized access risk and provides the baseline user experience for all authenticated flows.

**Description**: Implement the email+password login flow end-to-end: client/server input validation, secure password verification against stored hashes, token issuance (configurable type/TTL default 30m), role lookup and redirect mapping, and non-revealing error behavior. Include rehash-on-login logic for algorithm versioning.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-009

**Key Deliverables**:
- POST /auth/login endpoint with server-side validation and constant-time password verification
- Client-side validation guidance and API error contract (401 / "Invalid credentials")
- Token issuance integration with KMS-backed signing (configurable JWT/opaque)
- Role lookup and redirect payload for client consumption
- Rehash-on-login flow for algorithm/version upgrades
- Acceptance tests for happy path and failure modes (UC-001..UC-003, UC-008)

**Dependent EPICs**: EP-TECH, EP-DATA

### EP-002: Token & Session Management (TTL, revocation, logout)
**Business Value**: Ensures session security and compliance by enforcing token expiry, providing revocation/logout semantics, and supporting chosen revocation strategy.

**Description**: Implement token lifecycle management: token generation with token_id, TTL enforcement, logout endpoint with revocation mechanics (Redis blacklist or session store), and verification logic to reject revoked/expired tokens. Provide operational metrics for revocation propagation.

**UI Impact**: Yes

**Screen References**: SCR-010

**Key Deliverables**:
- Token generation & verification module (TR-003) with token_id claim
- Logout/revoke endpoint and revocation store integration (Redis + durable audit)
- Revocation propagation strategy and TTL alignment with tokens
- Tests covering expiry, revocation, and logout idempotency (UC-007)
- Documentation of chosen strategy and operational playbook for revocation checks

**Dependent EPICS**: EP-TECH, EP-DATA

### EP-003: Password Recovery (Forgot / Reset flow)
**Business Value**: Reduces support load and increases account recoverability by delivering a secure, privacy-preserving password reset flow.

**Description**: Build the forgot-password and reset-password end-to-end flow: generate cryptographically-random single-use tokens, store hashed tokens, enqueue and send reset emails, verify tokens on reset, update password with secure hashing, and invalidate tokens immediately on use.

**UI Impact**: Yes

**Screen References**: SCR-002, SCR-003, SCR-004, SCR-005

**Key Deliverables**:
- POST /auth/forgot-password and background worker to send emails
- Reset token generation (>=128 bits) and hashed storage (DR-002)
- Reset endpoint to validate hashed token, expire tokens, update password hash
- Generic responses to avoid account enumeration (acceptance criteria per FR-007)
- Email delivery logging and retry policy integration with message bus

**Dependent EPICS**: EP-TECH, EP-DATA

### EP-004: Account Protection & Rate-Limit (Lockout, rate limiting, CAPTCHA baseline)
**Business Value**: Protects accounts and system availability by preventing brute-force and credential-stuffing attacks and providing admin support workflows for locked accounts.

**Description**: Implement failed-attempt counters, distributed atomic lockout behavior, configurable thresholds/durations, admin unlock API and UI support, and deterministic rate-limiting + CAPTCHA escalation integration points. Ensure counters and lock state are consistent across instances.

**UI Impact**: Yes

**Screen References**: SCR-006, SCR-001, SCR-011

**Key Deliverables**:
- Atomic failed_attempts counters (Redis + Postgres fallback) and Lua scripts (TR-004)
- Account lock/unlock logic and admin unlock endpoint (UC-004)
- API gateway / per-IP and per-account rate-limit configuration (FR-011)
- CAPTCHA escalation hooks and accessibility considerations (UX integration)
- Tests for lockout thresholds, unlock flows, and race-condition handling
- Monitoring & alerts for anomalous failed-attempt spikes

**Dependent EPICS**: EP-TECH, EP-DATA

### EP-005: Audit & Observability Pipeline (Structured logs, SIEM integration)
**Business Value**: Provides auditable security events and observability necessary for compliance, security operations, and incident response.

**Description**: Implement structured JSON audit events for auth lifecycle events, forwarding to message bus and SIEM, metrics, tracing instrumentation, retention/PRIVACY controls, and queries needed by security teams. Ensure PII is hashed/redacted and logs are forwarded within the required SLA.

**UI Impact**: Yes

**Screen References**: SCR-012, SCR-011

**Key Deliverables**:
- Structured audit event schema and producers for login success/failure, lock/unlock, reset lifecycle (FR-010)
- Async eventing to message bus and SIEM with forwarding SLAs (TR-005)
- OpenTelemetry tracing and Prometheus metrics (TR-007)
- Audit retention and deletion support (DR-003), privacy controls and redaction rules (NFR-006)
- Dashboards and basic admin UI hooks (SCR-012) for security ops
- Tests and verification that logs do not contain secrets

**Dependent EPICS**: EP-TECH

### EP-006: Multi-Factor Authentication (TOTP enrollment & challenge)
**Business Value**: Strengthens account security for privileged users and supports compliance controls for high-risk roles.

**Description**: Deliver optional TOTP-based MFA: enrollment with QR provisioning, verification, recovery codes (one-time display & hashed storage), and challenge flow during login. Ensure MFA integrates with lockout policy and is pluggable.

**UI Impact**: Yes

**Screen References**: SCR-007, SCR-008

**Key Deliverables**:
- MFA enrollment endpoints and QR provisioning flow (FR-012)
- TOTP verification logic and enforcement hooks for privileged roles
- Recovery code generation, hashed storage, and one-time display UI behavior
- Integration tests for enrollment, challenge, and recovery flows
- Admin controls and rollout feature flags for mandated enforcement

**Dependent EPICS**: EP-TECH, EP-DATA

### EP-007: UX - Core Interaction & Flows (Primary actions, task reachability, interaction patterns)
**Business Value**: Ensures high conversion and low support friction by making core auth flows discoverable, fast, and consistent across devices.

**Description**: Implement the core UI/interaction patterns for login, forgot password and reset, MFA interactions and micro-interactions: prominence of primary CTAs, fallback CTAs, task reachability (≤3 interactions), loading states, disable-while-pending behavior and inline validation patterns required by acceptance criteria.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-004, SCR-007

**Key Deliverables**:
- Design-system-aligned components for auth flows and states (loading, disabled, validation)
- Interaction patterns: primary CTA prominence, fallback CTAs, 3-step reachability checks (UXR-101, UXR-102, UXR-002)
- Implement loading indicators and disable-submit behavior (UXR-501, UXR-502)
- Field-level inline validation and accessible error announcements (UXR-301, UXR-103)
- Usability acceptance test scripts and measurement plan

**Dependent EPICS**: EP-TECH

### EP-008: UX - Accessibility & Visual Quality (WCAG, tokens, focus states)
**Business Value**: Achieves accessibility, consistency, and compliance to reduce legal and operational risk and to improve usability for all users.

**Description**: Deliver WCAG 2.2 AA compliance across auth screens, implement design tokens and focus/contrast guidelines, ensure error announcements, accessible CAPTCHA flows and recovery code handling, and verify responsiveness at target breakpoints.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002, SCR-003, SCR-004, SCR-005, SCR-006, SCR-007, SCR-008, SCR-009, SCR-010, SCR-011, SCR-012

**Key Deliverables**:
- WCAG AA compliance checklist implementation and verification (UXR-201, UXR-202)
- Design token adoption and visual spec enforcement (UXR-401, UXR-402)
- Accessible error messaging and aria-live integration (UXR-601, UXR-602, UXR-603)
- CAPTCHA accessibility fallback and keyboard flows (UXR-701)
- MFA recovery-code display and confirm-store UX (UXR-801)
- Accessibility test reports and remediation plan

**Dependent EPICS**: EP-TECH

### EP-009: Adaptive Detection (Auditable anomaly scoring service for escalations)
**Business Value**: Improves detection of coordinated attacks and reduces false positives by adding auditable, explainable scoring to escalate mitigation (CAPTCHA/block) while remaining reviewable by security ops.

**Description**: Provide an optional auditable anomaly-scoring service and integration points to consume scores for adaptive rate-limiting/CAPTCHA. Ensure explainability, storage of inference metadata, and manual override paths so that ML-derived decisions are auditable and reversible.

**UI Impact**: No

**Screen References**: N/A

**Key Deliverables**:
- Anomaly scoring service API and feature flags (AIR-001)
- Explainability metadata attached to events and stored with audit logs (AIR-002)
- Integration hooks for API gateway and Auth Service to request scores and act on them
- Logging of inference details (no PII) and admin override controls
- Documentation of hosting, retraining constraints and governance (deferred for AIR-003 until clarified)

**Dependent EPICS**: EP-TECH, EP-005

## Backlog Refinement Required

The following requirements are tagged [UNCLEAR] in source documents and were NOT mapped to epics. Each item requires clarification before being assigned and planned.

- FR-013 [UNCLEAR] — Support SSO/OAuth2/OpenID Connect integration.
  - Clarification questions:
    - Which identity providers (IdP list) and protocols (OIDC, SAML) must be supported in initial scope?
    - Required claims mapping, session lifetime expectations, and admin provisioning behavior?
    - Is an enterprise-only subset acceptable for MVP or is broad SSO support required?

- DR-006 [UNCLEAR] — Data Residency & Compliance.
  - Clarification questions:
    - Which geographic regions and legal regimes must be supported (e.g., EU, US, APAC)?
    - Required retention windows and specific deletion/GDPR/CCPA requests handling rules?
    - Are dedicated region-based deployments required or is logical separation sufficient?

- AIR-003 [UNCLEAR] — Model Hosting & Training Data Policies.
  - Clarification questions:
    - Will ML models be hosted on cloud-managed services or on-premise? Preferred providers?
    - What PII/feature data retention and anonymization rules apply to model training datasets?
    - Which team or compliance controls must own model governance and retraining pipelines?

Please respond with clarifications for the three items above so they can be moved from Backlog Refinement into the appropriate epic(s).