# Task - task_007

## Requirement Reference
- User Story: us_001
- Story Location: .propel/context/tasks/EP-001/us_001/us_001.md
- Acceptance Criteria:  
    - Given a registered user with valid credentials, When the client sends a POST /auth/login with a JSON payload:
      {
        "username": "user@example.com",
        "password": "correct-password"
      }
      Then the server responds with HTTP 200 and a JSON body:
      {
        "access_token": "<JWT>",
        "token_type": "Bearer",
        "expires_in": 3600
      }
      - The token is signed with the system's configured signing key, contains standard claims (sub, iat, exp, roles), and expires after the configured TTL.
      - The response body contains no stack traces or internal error details.
    - Given a user supplies incorrect credentials, When the client POSTs to /auth/login, Then the server responds with HTTP 401 and a generic error message:
      {
        "error": "Invalid credentials"
      }
      - Failed login attempt is recorded (user identifier or IP) for rate-limiting and monitoring.
      - Response time and payload size remain within acceptable limits to avoid timing/side-channel leaks.
    - Given malformed input (missing fields, invalid JSON, invalid email format), When the client POSTs to /auth/login, Then the server responds with HTTP 400 with a validation error that identifies invalid fields (e.g., "password: required", "username: invalid email format") and does not proceed to password verification.
    - Given repeated failed attempts that exceed configured thresholds, When threshold is reached, Then subsequent requests return HTTP 429 (or 423 if account locked) with a generic message ("Too many attempts, try again later"), and the system enforces backoff/lockout per security policy. Alerts and audit logs must be created for suspected brute-force activity.
    - Given a successful login, When the server issues a token, Then an audit record is created (user id, timestamp, client IP, user-agent), and integration tests demonstrate median authentication latency meets NFR (auth median < 1s) under standard load.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable" and a retryable error code; do not leak internal details.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.
    - Password hash algorithm/version mismatch (legacy records) — handle by verifying using the legacy algorithm, force re-hash on next successful login, and record upgrade event without revealing algorithm differences.

## Design References (Frontend Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **UI Impact** | No |
| **Figma URL** | N/A |
| **Wireframe Status** | N/A |
| **Wireframe Type** | N/A |
| **Wireframe Path/URL** | N/A |
| **Screen Spec** | N/A |
| **UXR Requirements** | N/A |
| **Design Tokens** | N/A |

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | N/A | N/A |
| Backend | Node.js + Express | 18.x / 4.x (Express) |
| Database | PostgreSQL | 16.x |
| Library | pg (node-postgres) | 8.x |
| Library | argon2 (password hashing) | 0.30.x |
| Library | jsonwebtoken | 9.x |
| Library | prom-client (Prometheus metrics) | 14.x |
| Library | ioredis (rate-limit counters) | 5.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All code and libraries MUST be compatible with Node 18+ and the listed versions.

## AI References (AI Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **AI Impact** | No |
| **AIR Requirements** | N/A |
| **AI Pattern** | N/A |
| **Prompt Template Path** | N/A |
| **Guardrails Config** | N/A |
| **Model Provider** | N/A |

## Task Overview
Implement structured audit logging and observability for authentication flows. Specifically:
- Persist structured audit entries for authentication events to a new auth_audit_logs table (fields: id, user_id (nullable), event_type, timestamp, client_ip, user_agent, metadata JSON).
- Emit Prometheus metrics for auth.success, auth.failure and auth.latency (histogram or summary) with labels (reason, client_ip masked, user_id nullable).
- Integrate audit logging and metrics emission into the existing authentication flow (login controller/service) without recording or logging sensitive information (passwords, raw tokens, PII beyond allowed user_id and masked IP).
- Ensure behavior aligns with acceptance criteria: failed-login counters for rate-limiting, audit entry on successful login, and 503 handling when DB unavailable.

## Dependent Tasks
- US_010 (User data & password storage/hash) — required to verify credentials and legacy-hash handling.
- US_011 (Token storage & revocation) — required for token issuance and revocation hooks if metrics require token state.
- EP-TECH readiness (observability & platform infra) — Redis, Prometheus, and logging pipeline must be available.
- Database migration task to create auth_audit_logs table (can be part of this task if migration workflow exists).

## Impacted Components
- server/src/controllers/authController.ts (MODIFY) — capture request context and pass to auth service.
- server/src/services/authService.ts (MODIFY) — emit metrics and create audit entries.
- server/src/lib/auditLogger.ts (CREATE) — DB helper for writing audit log entries.
- server/src/lib/metrics.ts (CREATE) — Prometheus metrics registration and utilities.
- server/migrations/2026xxxx_create_auth_audit_logs.sql (CREATE) — DB migration for audit table.
- server/src/config/observability.ts (MODIFY) — expose Prometheus metrics endpoint and init metrics client.
- server/src/tests/auth.audit.test.ts (CREATE) — unit/integration tests for audit logging & metrics.
- server/src/lib/rateLimiter.ts (MODIFY or CREATE) — ensure failed-login counters are incremented securely (uses Redis).

## Implementation Plan
1. Add DB migration to create auth_audit_logs table with columns: id (uuid), user_id (uuid, nullable), event_type (enum: login_success, login_failure, lockout, rehash), timestamp (timestamptz default now()), client_ip (inet or varchar masked), user_agent (text), metadata JSONB (nullable). Ensure retention policy comment for DR compliance.
2. Create server/src/lib/auditLogger.ts:
   - Expose async function writeAuthAudit({ userId, eventType, clientIp, userAgent, metadata }) that masks clientIp (e.g., zero out last octet for IPv4) before persisting and never stores password/token.
   - Use parameterized queries via pg client.
   - On DB errors, propagate a typed error (AuditWriteError) and ensure upper layers handle it gracefully (do not fail login if audit write fails unless policy requires).
3. Create server/src/lib/metrics.ts:
   - Register metrics: auth_success_total (counter with labels), auth_failure_total (counter with labels), auth_latency_seconds (histogram with sensible buckets).
   - Provide helpers: observeSuccess(labels, latency), observeFailure(labels, latency).
   - Ensure metrics avoid storing PII: mask IP label (e.g., client_ip_cidr or "masked") and limit user_id label to "user:<hashed-or-id>" or “unknown”.
4. Integrate into auth flow:
   - In authController, measure start time, pass client IP and user-agent to authService.
   - In authService, after verification success: emit auth.success metric, call auditLogger.writeAuthAudit with event_type=login_success and minimal metadata (token expiry seconds, token_id if applicable but not raw token), and return token.
   - On failure: emit auth.failure metric, increment failed-login counters in Redis and call auditLogger.writeAuthAudit with event_type=login_failure (user_id nullable), include reason label (invalid_credentials, lockout, db_unavailable).
   - Ensure timing side-channel mitigation: keep response timing consistent (apply constant-time behavior where appropriate).
5. Rate-limiting integration:
   - Use existing rateLimiter (or create) that uses Redis to increment per-IP and per-user counters and returns lockout status.
   - When lockout triggered, emit auth.failure, write audit log with lockout event, return 429/423 as configured with generic messages.
6. Error handling and 503:
   - If DB is unavailable for credential verification or audit write fails critically (per policy), return 503 with generic message and emit an incident metric.
7. Tests:
   - Unit tests for auditLogger masking and DB insert query.
   - Integration test that simulates login success and verifies audit row exists and metrics updated.
   - Test that failed login increments failure metric and creates audit entry.
   - Test DB unavailability returns 503 and no internal errors exposed.
8. Documentation:
   - Update observability docs to specify metrics names and labels, and retention/pseudonymization rules for audit logs.

## Current Project State
- server/
  - src/
    - controllers/
      - authController.ts
    - services/
      - authService.ts
    - lib/
      - db.ts
      - logger.ts
    - config/
      - observability.ts
    - routes/
      - auth.ts
    - tests/
      - auth.test.ts
  - migrations/
    - README.md

(Actual tree may vary; update paths when dependent tasks finish. This snapshot is a placeholder to guide changes.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | server/migrations/20260406_create_auth_audit_logs.sql | Migration to create auth_audit_logs table with columns (id, user_id, event_type, timestamp, client_ip_masked, user_agent, metadata JSONB) and retention note. |
| CREATE | server/src/lib/auditLogger.ts | Module exposing writeAuthAudit(...) to persist audit entries; includes IP masking logic and parameterized SQL. |
| CREATE | server/src/lib/metrics.ts | Module to register and expose Prometheus metrics: auth_success_total, auth_failure_total, auth_latency_seconds; helpers for observing. |
| MODIFY | server/src/services/authService.ts | Integrate metrics & audit logger calls into auth flow (on success and failure); ensure no sensitive data logged. |
| MODIFY | server/src/controllers/authController.ts | Capture client IP and user-agent; measure latency and forward to authService; standardize error responses (401/429/423/503). |
| MODIFY | server/src/config/observability.ts | Initialize prom-client, expose /metrics endpoint (if not present), and export metrics registry. |
| CREATE | server/src/tests/auth.audit.test.ts | Tests verifying audit log entries and metrics emission for success and failure flows, and 503 behavior when DB unavailable. |
| MODIFY | server/src/lib/rateLimiter.ts | (If present) increment failed-login counters on failure and return lockout status; otherwise create new module to do this. |

## External References
- Prometheus Node client (prom-client): https://github.com/siimon/prom-client
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Logging best practices & GDPR/PII guidance: https://www.owasp.org/index.php/Logging_Cheat_Sheet
- PostgreSQL inet type docs: https://www.postgresql.org/docs/current/datatype-net.html
- Parameterized queries with node-postgres: https://node-postgres.com/features/queries

## Build Commands
- Install dependencies: npm ci
- Run tests: npm test
- Build: npm run build
- Run locally (dev): npm run dev
- Run migrations: npm run migrate
- Start with Docker Compose (if app uses it): docker-compose up -d
- Metrics endpoint: GET http://localhost:PORT/metrics

(Refer to ../.propel/build/ for CI-specific build steps.)

## Implementation Validation Strategy
- [ ] Unit tests pass for auditLogger (IP masking, SQL parameterization).
- [ ] Integration tests pass verifying audit row creation on success and failure.
- [ ] Metrics are exposed on /metrics and values increment for auth.success/auth.failure and observe latency histogram.
- [ ] Rate-limiter increments failed-login counters and returns lockout when threshold exceeded.
- [ ] 503 behavior tested: simulate DB unavailability and verify HTTP 503 with generic message.
- [ ] No sensitive data (passwords, raw tokens) appear in logs or audit table.
- [ ] Performance validation: integration tests show median auth latency < 1s under standard load (separate perf test).

## Implementation Checklist
- Effort estimate: 8 hours
1. [ ] Create DB migration server/migrations/20260406_create_auth_audit_logs.sql and run locally to ensure table creation.
2. [ ] Implement server/src/lib/auditLogger.ts with IP masking and parameterized insert; add unit tests for masking.
3. [ ] Implement server/src/lib/metrics.ts registering required Prometheus metrics and helper functions.
4. [ ] Modify authController and authService to call metrics helpers and auditLogger on success/failure; ensure client IP and user-agent are forwarded.
5. [ ] Integrate failed-login counter increments into rateLimiter (Redis); emit lockout audit event and metric when threshold reached.
6. [ ] Add integration tests server/src/tests/auth.audit.test.ts to verify DB row and metrics after login success/failure, and 503 behavior.
7. [ ] Update server/src/config/observability.ts to initialize prom-client and ensure /metrics endpoint is available.
8. [ ] Run full test suite and a short integration latency check; fix issues and ensure no sensitive data in logs.

## RULES:
- Implementation Checklist contains 8 items (<=8).
- Effort: 8 hours (<=8 hours).
- Checklist items are specific and actionable.

