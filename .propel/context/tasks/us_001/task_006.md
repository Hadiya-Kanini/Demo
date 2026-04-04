## Task ID: task_006
## Task Title: Implement unit, integration and performance tests for auth flows
## Category: Testing

## Description:
Implement a comprehensive automated test suite and a performance script to validate authentication (POST /auth/login) flows. Deliverables include:
- Unit tests for controllers, services, and helpers involved in login/auth (happy path, validation, error handling, lockout counters).
- Integration tests that run against a test PostgreSQL database and Redis instance (or testcontainers) covering: successful login, invalid credentials, malformed request, account/IP lockout path, database unavailable (503), and Redis unavailable (cache/session failures).
- End-to-end API tests that start the app in a test environment and exercise the full login flow including audit logging and token issuance.
- A performance test (k6) that reproduces defined load and asserts median auth latency (p50) < 1s and acceptable success/error rates.
- CI job definitions or pipeline steps to run unit, integration, E2E and perf checks (perf may be gated/optional in CI depending on infra).

Parent Story: us_001

## Acceptance Criteria:
1. Unit tests:
   - Achieve focused coverage for auth-related modules (controllers, services, helpers). Each function has at least one test covering success and one covering failure path where applicable.
   - Unit tests run in CI and pass (exit code 0).

2. Integration tests:
   - Integration suite exercises: valid login -> HTTP 200 with access_token (JWT), token_type "Bearer", expires_in matches configured TTL; invalid credentials -> HTTP 401 with "Invalid credentials"; malformed payload -> HTTP 400 with validation details; account/IP lockout -> returns HTTP 429/423 per policy; DB unavailable -> HTTP 503 with "Service temporarily unavailable" and no internal details.
   - Tests run against ephemeral test DB + Redis (testcontainers/docker-compose) in CI and pass consistently.
   - Audit log entry is verified for successful login (user id, timestamp, client IP or stubbed IP, user-agent).

3. End-to-end tests:
   - E2E tests start application in test mode, run login flows end-to-end and validate JWT signature structure, expiry, and presence of auth audit entry.
   - E2E tests pass in CI job.

4. Performance test:
   - k6 script (perf/k6/auth-login.js) implements load profile: ramp to 200 VUs, sustain 2 minutes, ramp down. k6 thresholds must assert p(50) < 1000 ms and HTTP request success rate >= 95%.
   - Running the k6 script against the application in a controlled test environment produces median (p50) auth latency below 1s. Report attached (k6 summary) for the run.

5. CI integration:
   - Add or update CI pipeline to run unit tests on every push, integration and E2E on PRs (or nightly), and perf as a gated job (manual or nightly). Pipelines return non-zero on failures.

6. Documentation:
   - README updates with how to run: unit tests, integration tests (including docker/testcontainers setup), E2E tests, and the k6 perf script (required environment variables, endpoints, and how to interpret thresholds).
   - Clear troubleshooting notes for common failure modes (DB/Redis not reachable, port conflicts).

## Technical Specifications:

- APIs / Endpoints:
  - POST /auth/login
    - Request: JSON { "username": string | "email": string, "password": string }
    - Success response: HTTP 200 { "access_token": "<JWT>", "token_type": "Bearer", "expires_in": <seconds> }
    - Failure responses:
      - 400: validation errors
      - 401: Invalid credentials
      - 429 or 423: account or IP lockout (per policy)
      - 503: service temporarily unavailable (DB inaccessible)
  - Audit logging path: server writes to auth_audit table or logs (ensure testable).

