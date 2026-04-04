## Task ID: task_001
## Task Title: Implement POST /auth/login controller and route
## Category: Backend

## Description:
Add a production-ready POST /auth/login endpoint that:
- Validates incoming JSON payloads (email/username + password) with explicit, field-level errors for 400 responses.
- Coordinates credential verification against the users store (password hash verification, legacy-hash fallback if required).
- Issues signed JWT access tokens with TTL derived from configuration (JWT_SECRET, JWT_TTL).
- Records an audit event on successful login (user_id, timestamp, client IP, user-agent, token_id).
- Increments and observes failed-login counters for rate limiting and lockout policies (per account and per IP).
- Returns precise, non-revealing responses for 200 / 400 / 401 / 429 / 503 conditions, and never leaks internal exception details.
- Is instrumented for observability (structured logs with traceId, RED metrics hooks), uses timeouts for DB calls, and follows OWASP and project backend standards.

Parent story: us_001

## Acceptance Criteria:
1. Given valid credentials, POST /auth/login returns 200 OK with JSON:
   - { "access_token": "<JWT>", "token_type": "Bearer", "expires_in": <seconds> }
   - No sensitive debug data returned. Response contains no password, hash, or internal error traces.
   - An audit log entry exists with user_id, timestamp, client_ip, user_agent, and token_id.
   - Token TTL equals configured JWT_TTL (integration test verifies expiry).
2. Given invalid credentials, POST /auth/login returns 401 Unauthorized with body:
   - { "code": "AUTH_INVALID_CREDENTIALS", "message": "Invalid credentials" }
   - Failed-login counters incremented for user and source IP (used by rate limiter).
3. Given malformed/missing fields (e.g., missing password, invalid email format), POST /auth/login returns 400 Bad Request with field-specific validation errors (JSON schema / array).
4. If failed attempts exceed configured threshold, return 429 Too Many Requests (or 423 Locked depending on policy) with generic message and proper Retry-After header when applicable. Failed attempts and lock events are logged for monitoring.
5. If the credential verification layer (DB) is unavailable or times out, return 503 Service Unavailable with:
   - { "code": "SERVICE_UNAVAILABLE", "message": "Service temporarily unavailable. Please try again later." }
   - Do not return stack traces or internal errors.
6. All responses follow the standard error envelope: { traceId, code, message, details? } in addition to normal payloads.
7. Unit, integration, and E2E tests cover:
   - success, invalid creds, validation failures, rate-limit / lockout, DB unavailable (503), and token TTL verification.
8. Response latency median in integration tests remains below 1s for auth happy path (can be asserted in CI integration suite).

## Technical Specifications:

- APIs/Endpoints
  - POST /auth/login
    - Request Headers:
      - Content-Type: application/json
      - Optional: X-Request-ID or Accept-Language
    - Request Body (JSON):
      - Either:
        - { "email": "user@example.com", "password": "plainText" }
        - OR { "username": "alice", "password": "plainText" } (if username supported)
    - Response 200 (success):
      - { "access_token": "<JWT>", "token_type": "Bearer", "expires_in": 3600 }
    - Response 400 (validation):
      - { "traceId": "<id>", "code": "VALIDATION_ERROR", "message": "Invalid request", "details": [ { "field": "email", "message": "Invalid email format" } ] }
    - Response 401 (invalid credentials):
      - { "traceId": "<id>", "code": "AUTH_INVALID_CREDENTIALS", "message": "Invalid credentials" }
    - Response 429/423 (rate-limited/locked):
      - { "traceId": "<id>", "code": "TOO_MANY_ATTEMPTS", "message": "Too many attempts. Please try again later." }
      - Add Retry-After header where applicable.
    -