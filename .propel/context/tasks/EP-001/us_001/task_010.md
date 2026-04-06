## Task ID: task_010
## Task Title: Run integration, performance and chaos tests for authentication
## Category: Testing

## Description
Execute a coordinated test campaign against the authentication service (staging) to validate functional correctness, performance SLO (median latency < 1s), and resilience under dependency failures. Deliverables:
- Staging integration test suite (end-to-end against /api/auth/login) that validates success, failure, validation and audit logging behavior defined in us_001.
- Load test scripts and execution showing median auth latency < 1s (target) and p95 < 2s; publish results and thresholds.
- Chaos tests simulating database outages (connection drop / process stop / delayed responses) that verify returned HTTP 503, circuit-breaker behavior, and graceful degradation (no sensitive information leaked).
- Observability checks: ensure RED metrics, traces and auth audit logs are captured for test runs.

Parent Story: us_001 (Implement Login API /auth/login)

## Acceptance Criteria
1. Integration tests (run in staging) pass for:
   - Valid credentials → HTTP 200 with JSON { access_token, token_type:"Bearer", expires_in } and audit log entry (user id, timestamp, client IP, user-agent).
   - Invalid credentials → HTTP 401 and message "Invalid credentials"; failed-login counter incremented.
   - Malformed request (missing fields / invalid email) → HTTP 400 with field-specific validation error.
2. Performance: Load test run(s) against staging demonstrate:
   - Median (p50) authentication request latency < 1s.
   - p95 latency < 2s.
   - Error rate ≤ 1% under target load profile (see Technical Specifications).
   - Detailed report (graphs, raw metrics) uploaded to the test artifacts location.
3. Chaos: Under simulated DB outage conditions:
   - Auth endpoint returns HTTP 503 with message "Service temporarily unavailable" (no internal stack traces).
   - Circuit breaker (or equivalent resilience mechanism) transitions to OPEN after configured failure threshold and subsequent auth attempts return 503 without causing request queue buildup.
   - System emits circuit-breaker and dependency-failure metrics (events/alerts) and logs a structured error (no PII).
4. Observability:
   - Each scenario produces traceId-correlated logs, RED metrics (rate, errors, duration) for /api/auth/login, and audit log entries where applicable.
   - Test run artifacts include traces and sample auth audit log entries.
5. CI Gate:
   - Create pass/fail thresholds for CI: fail PR if load test median > 1s or if chaos test causes unexpected 5xx behavior other than 503 during DB outage scenarios.

## Technical Specifications

APIs/Endpoints
- POST /api/auth/login
  - Request: application/json { "email": string, "password": string }
  - Success response: 200 { "access_token": string (JWT), "token_type": "Bearer", "expires_in": integer }
  - Error responses:
    - 400 Validation error: { "errors": [{ "field": "email", "message": "invalid email" }, ...] }
    - 401 Invalid credentials: { "message": "Invalid credentials" }
    - 503 Service unavailable: { "message": "Service temporarily unavailable" }
- Health/readiness endpoints used for chaos orchestration:
  - GET /health/liveness
  - GET /health/readiness

Components/Classes (impacted / referenced)
- AuthController (controller/route handler for /auth/login)
- AuthService (business logic: verify creds, issue token)
- UserRepository / UserStore (DB access layer for user records)
- PasswordHasher (bcrypt / argon2 wrapper)
- TokenService (JWT creation and TTL config)
- AuditLogger (writes auth audit logs)
- MetricsCollector (RED metrics instrumentation)
- CircuitBreaker (library or custom middleware—e.g., resilience4j, opossum or internal implementation)
- DB Connection Pool / Driver (Postgres client or configured DB