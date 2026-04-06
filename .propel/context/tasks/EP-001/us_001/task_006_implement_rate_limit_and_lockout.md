# Task - [TASK_006]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/us_001/us_001.md]
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
    - What happens when a user attempts login during/after account lockout or multi-factor enforcement? — If account locked, return HTTP 423 with a generic message. (MFA-specific flows are out of scope; if required, return generic 403/202 as per policy and mark as dependency.)

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
| Backend | Node.js (TypeScript) + Express | 18.x, Express 4.x |
| Database | PostgreSQL | 14.x - 16.x |
| Library | Redis (client: ioredis) | redis v4 / ioredis v5 |
| Library | JWT (jsonwebtoken or jose) | jsonwebtoken v9 or jose latest |
| Library | Password hashing (argon2 / bcrypt) | argon2 v0.30 / bcrypt v5.x |
| Library | Validation (zod or Joi) | zod 3.x or Joi v17 |
| Observability | Prometheus metrics client | prom-client v14 |
| Background Jobs | BullMQ (if used for async re-hash) | v1.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All code and libraries MUST be compatible with Node.js 18+ and TypeScript support.

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
Implement per-user and per-IP failed-login counters, enforce rate-limit thresholds and account lockouts backed by Redis (or DB fallback), persist lock state, and emit structured metrics/logs/alerts for suspected brute-force activity. Integrate with the existing login flow so invalid credentials increment counters, thresholds return 429/423, and successful login clears relevant counters and emits audit logs. Provide unit/integration tests and a small migration/schema for persisted lock state if using DB fallback.

## Dependent Tasks
- Artefacts/Tasks required before starting:
  - task_001_be_login_route_controller (POST /auth/login route/controller implemented)
  - task_002_be_input_validation_schema (request validation schema implemented)
  - task_003_be_auth_service_password_and_token (password verification + JWT issuance service implemented)
  - infra_task_redis_provision (Redis instance provisioned & connection details available)
  - task_db_migrations_audit_logs (DB table for auth_audit_logs created if not present)
  - task_010_secrets_and_config (signing keys and TTL config available in secrets manager)

## Impacted Components
- src/controllers/authController.ts (login endpoint will call new rate limiter)
- src/services/authService.ts (increment/clear counters integration)
- src/services/auth/rateLimiter.ts (new; per-user and per-IP counters and evaluation)
- src/services/auth/lockStore.ts (new; persisted lock state handling; Redis primary, DB fallback)
- src/lib/redisClient.ts (may be added/modified to expose required operations)
- src/lib/metrics.ts (add auth.failure, auth.locked, auth.throttle, auth.success counters; auth.latency histogram)
- migrations/xxxx_create_auth_audit_and_lock_tables.sql (if DB persistence used)
- tests/unit/rateLimiter.spec.ts (unit tests)
- tests/integration/auth_rate_limit.spec.ts (integration tests for thresholds/lockout responses)

## Implementation Plan
- Design data model for counters and lock state:
  - Use Redis keys: login:fail:ip:{ip}, login:fail:user:{userIdOrNormalizedUsername} with TTLs (rolling window).
  - Lock key: login:lock:user:{userId} and login:lock:ip:{ip} storing lock expiry and reason.
- Implement Redis-backed rateLimiter module:
  - Increment counters atomically (INCR, EXPIRE) and return current count + TTL remaining.
  - Evaluate thresholds: pre-configured values for per-IP and per-user (e.g., 5 attempts per 15m, lock after 10 attempts).
- Implement lockStore module:
  - Set lock with expiry, check lock state, persist metadata (timestamp, attempts, source IP).
  - Provide fallback to DB table if Redis unavailable (ensure eventual consistency).
- Integrate rateLimiter into authService/login controller:
  - On incoming login: first check lockStore for user or IP lock -> return 423/429.
  - Validate input -> if invalid return 400 (existing).
  - Verify credentials:
    - On invalid credential: increment counters for user (if exists) and IP, evaluate thresholds and set locks if thresholds crossed; emit metric auth.failure and increment Prometheus counters and log structured event.
    - On valid credential: clear counters for user and IP, issue token, emit auth.success metric and write audit log (user id, timestamp, client IP, user-agent).
    - Ensure legacy hash re-hash logic remains in authService (dependency).
- Error handling:
  - If DB or Redis unavailable during verification, return 503 and increment incident metric; do not leak internals.
- Alerts/metrics:
  - Emit auth.bruteforce.alert metric or log with severity when lock created.
- Tests:
  - Unit tests for rateLimiter/lockStore logic including TTL behavior.
  - Integration tests to simulate repeated failed attempts from same IP and same user and assert responses 401->429/423 and that locks are persisted and subsequent attempts are rejected.
- Configuration:
  - Add config entries: RATE_LIMIT_USER_WINDOW, RATE_LIMIT_USER_MAX, RATE_LIMIT_IP_WINDOW, RATE_LIMIT_IP_MAX, LOCK_DURATION, REDIS_FALLBACK_ENABLED.
- Documentation:
  - Update OpenAPI spec for /auth/login to include 429/423 responses.

## Current Project State
- Project root (placeholder, update during execution):
  - src/
    - controllers/
      - authController.ts (exists from dependent task_001)
    - services/
      - authService.ts (exists from dependent task_003)
    - lib/
      - dbClient.ts
    - config/
      - index.ts
  - migrations/
  - tests/
- Note: This task assumes login route, validation, and token issuance exist (dependencies above). This tree will be updated with new files in Expected Changes.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/services/auth/rateLimiter.ts | Redis-backed module to increment and evaluate per-user and per-IP failed-login counters and thresholds. |
| CREATE | src/services/auth/lockStore.ts | Module to persist and query lock state (Redis primary, Postgres fallback) and set lock metadata. |
| MODIFY | src/services/authService.ts | Integrate rateLimiter and lockStore: check locks before verification, increment counters on failure, clear on success, emit metrics/logs. |
| MODIFY | src/controllers/authController.ts | Add pre-check for lock state and map lock/throttle responses (429/423). Ensure IP/user context passed. |
| CREATE | src/lib/redisClient.ts | Initialize and export Redis client with reconnect/backoff and helper for atomic operations. |
| MODIFY | src/lib/metrics.ts | Add new metrics: auth_failure_total, auth_success_total, auth_lock_total, auth_throttle_total, auth_latency_seconds histogram. |
| CREATE | migrations/2026xx_create_auth_lock_table.sql | Optional DB table for persisted locks fallback: auth_locks (id, subject, lock_type, expires_at, metadata, created_at). |
| CREATE | tests/unit/rateLimiter.spec.ts | Unit tests for rateLimiter behavior, TTL, atomic increments. |
| CREATE | tests/integration/auth_rate_limit.spec.ts | Integration tests covering threshold crossing, lock persistence, proper HTTP codes (429/423) and audit log produced. |
| MODIFY | README.md | Document configuration flags and operational notes for rate-limiting and Redis fallback.

## External References
- OWASP Account Lockout Guidance: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#account-lockout
- Redis INCR / EXPIRE atomic patterns: https://redis.io/commands/incr
- Rate limiting with Redis examples: https://redis.io/docs/manual/patterns/rate-limiting/
- Prometheus client for Node.js: https://github.com/siimon/prom-client
- JWT best practices: https://www.rfc-editor.org/rfc/rfc7519
- Argon2 password hashing recommendations: https://www.password-hashing.net/

## Build Commands
- Install dependencies: npm ci
- Build: npm run build
- Run tests: npm test
- Start (dev): npm run dev
- See .propel build reference: ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass (rateLimiter, lockStore)
- [ ] Integration tests pass (simulate failed attempts -> thresholds -> lock responses)
- [ ] Metrics (auth_failure_total, auth_lock_total, auth_throttle_total, auth_latency_seconds) emitted and testable via prom-client registry in tests
- [ ] 429/423 responses returned when thresholds or locks apply
- [ ] 503 returned when DB or Redis unavailable during critical path (verified by integration test mocking unavailability)
- [ ] Audit log entry created on successful login with required fields (user id, timestamp, client IP, user-agent)

## Implementation Checklist
- [ ] Add Redis client module (src/lib/redisClient.ts) with connection, retry/backoff, and helper atomic ops (INCR+EXPIRE).
- [ ] Implement rateLimiter.ts to increment counts and evaluate per-user and per-IP thresholds with configurable window and caps.
- [ ] Implement lockStore.ts to set/check locks (Redis primary, DB fallback) and persist metadata for alerts.
- [ ] Integrate checks into authController/authService: pre-check locks, increment counters on failure, set locks when thresholds reached, clear counters on success, write audit log, emit metrics.
- [ ] Add unit and integration tests for atomicity, TTL behavior, thresholds, lock persistence, and expected HTTP responses (401 -> 429/423).
- [ ] Add configuration entries and document operational runbook in README.
- [ ] Verify behavior for service unavailability (DB/Redis) returns 503 with no sensitive details.

Effort estimate: 6 hours

## RULES Compliance
- Implementation Checklist contains 7 items (<=8).
- Effort estimate is 6 hours (<=8).
- Acceptance Criteria are drawn from parent User Story us_001.

