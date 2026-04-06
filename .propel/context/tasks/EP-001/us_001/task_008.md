## Task ID: task_008
## Task Title: Implement resilient error handling and DB-unavailable behavior
## Category: Backend

## Description
Implement service-wide resilient error handling for the Auth/login flow so that database failures and degraded DB connectivity return an HTTP 503 with a standardized, non-revealing error envelope. Add a circuit-breaker around DB access used by authentication (credential verification) so repeated DB failures short-circuit attempts and return 503 immediately until the DB dependency recovers. Provide centralized error mapping and middleware to ensure all error responses follow the same envelope and do not leak stack traces or internal diagnostics in production. Add unit, integration, and E2E tests that simulate DB unavailability (both transient errors and complete outage) and validate behavior, metrics/logging, and circuit-breaker state transitions.

Traceability:
- Parent User Story: us_001 (Login API)
- Relevant Edge Case from US: "Database unavailable on credential verification — return HTTP 503 with 'Service temporarily unavailable' and a retryable error code; do not leak internal details."

## Acceptance Criteria
1. DB-failure response semantics:
   - Given a simulated DB connection error during POST /auth/login, the API returns HTTP 503 and a JSON error envelope:
     {
       "traceId": "<uuid>",
       "timestamp": "<ISO8601>",
       "code": "SERVICE_UNAVAILABLE",
       "message": "Service temporarily unavailable",
       "details": []
     }
   - No stack traces, SQL error messages, DB hostnames, or internal exception messages appear in the response body or headers.

2. Circuit-breaker behavior:
   - The circuit-breaker trips (opens) after N consecutive DB errors (N is configurable; default N = 5).
   - When open, requests that would call the DB are short-circuited and immediately return HTTP 503 with the same error envelope and code "SERVICE_UNAVAILABLE".
   - The circuit-breaker transitions to HALF_OPEN after a configurable cooldown (default 30s). On a successful DB call in HALF_OPEN, the breaker closes; on failure it re-opens and doubles cooldown (exponential backoff, capped).
   - Circuit-breaker configuration is externally configurable (env/config): threshold, cooldown, timeout, successThreshold.

3. Standardized error envelope:
   - All error responses from the auth endpoints conform to the envelope schema above.
   - Client-visible messages are generic and non-sensitive; internal error details are logged (structured log) but not returned to client.

4. Observability:
   - Each error response includes a traceId; that traceId is present in structured logs for correlation.
   - Circuit-breaker state change events (OPEN, HALF_OPEN, CLOSED) are emitted to logs and metrics (RED metrics for errors/durations).

5. Tests:
   - Unit tests validate error mapping and middleware behavior for a thrown DB error.
   - Integration tests (with test database or test container) simulate a DB outage and assert HTTP 503 and circuit-breaker behavior (open after configured threshold).
   - E2E tests (local or CI environment) simulate DB down scenario and assert clients receive 503 and that normal responses return after DB recovery and circuit-breaker transitions back to CLOSED.

6. Backward compatibility:
   - Existing success responses (HTTP 200, HTTP 400, HTTP 401) for the login API remain unchanged when DB is healthy.

## Technical Specifications

APIs/Endpoints
- POST /api/auth/login
  - Signature: POST /api/auth/login
  - Request: { "username": string, "password": string }
  - Responses:
    - 200 OK: { access_token, token_type, expires_in }
    - 400 Bad Request: validation envelope
    - 401 Unauthorized: { traceId, code: "INVALID_CREDENTIALS", message: "Invalid credentials", details: [] }
    - 503 Service Unavailable (DB failure): standardized envelope (see Data Models)

- (Optional) GET /health/readiness
  - Should return non-ready when DB circuit-breaker is OPEN (status 503) and ready only when DB reachable (200). This is optional but recommended to allow orchestration to detect degraded state.

Components/Classes (Node.js / Express naming conventions assumed; adapt to language if different)
- Controllers:
  - AuthController (controllers/auth.controller.ts) — existing; ensure it throws domain errors instead of writing HTTP responses directly (if currently doing).
- Services:
  - AuthService (services/auth.service.ts)
    - verifyCredentials(username, password): calls UserRepository and maps domain outcomes to domain errors.
- Infrastructure:
  - DbClient