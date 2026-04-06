## Task ID: task_005
## Task Title: Implement rate-limiting and lockout service for failed logins
## Category: Backend

## Description
Implement a configurable, production-ready failed-login rate-limiting and account/IP lockout service integrated into the authentication flow. The service must:

- Track failed authentication attempts per user identifier (user id / email) and per client IP.
- Use an abstracted persistence layer (primary: Redis; fallback/optional: relational DB) with atomic increments to avoid race conditions under concurrency.
- Apply a lockout/backoff policy that:
  - Throttles requests (429 Too Many Requests) when transient thresholds exceeded.
  - Temporarily locks accounts (423 Locked) when stricter thresholds exceeded.
  - Supports exponential backoff with jitter and configurable thresholds/TTL.
- Expose admin APIs to query and clear locks.
- Emit audit logs and metrics for monitoring.
- Be feature-flaggable and fully covered by unit, integration, and E2E tests.

Integrate with the existing POST /auth/login handler so invalid credentials increment counters and valid logins reset counters and remove any temporary throttles for that user.

## Acceptance Criteria
1. Given an invalid login attempt, the system increments both per-user and per-IP failed-attempt counters atomically and returns HTTP 401 to the client (business logic unchanged).
2. When the per-user or per-IP transient threshold is exceeded, subsequent login attempts receive HTTP 429 with a Retry-After header indicating when retry is allowed.
3. When the per-user lockout threshold is exceeded, further attempts for that user receive HTTP 423 Locked with a generic message ("Account temporarily locked") and an admin/audit event is recorded.
4. Successful authentication resets the per-user failed-attempt counter and removes transient throttles for that user.
5. Counters and lock states persist in the configured store (Redis by default) and survive short restarts consistent with TTL semantics.
6. Store implementations honor atomicity under concurrent requests (no lost increments).
7. Exponential backoff parameters, thresholds, TTLs, and store selection are configurable via application configuration (YAML/ENV).
8. Admin endpoints:
   - GET /admin/lockouts?user_id=<id>&ip=<ip> returns current counters/lock state (200).
   - POST /admin/lockouts/unlock accepts { user_id | ip } and clears locks (200).
9. All changes are covered by automated unit tests, integration tests against Redis (and DB adapter if present), and at least one E2E test that demonstrates lockout behavior.
10. Metrics emitted: failed_login_attempts_total (labels: user_id, ip, outcome), lockout_events_total, rate_limiter_checks_duration_ms. Audit log entry created for lock and unlock events.

## Technical Specifications

APIs/Endpoints
- Integration point (no new public auth endpoint required):
  - POST /auth/login (existing)
    - Behavior addition: Before credential verification completes, check rate/lock state. After failed credential verification, increment counters. After successful verification, reset user counters.
    - Response semantics:
      - 401 Invalid credentials (when auth fails but not throttled/locked)
      - 429 Too Many Requests (when transient throttle applies) — include Retry-After (seconds)
      - 423 Locked (when account locked) — include Retry-After if applicable
- Admin endpoints (backend-only, protected by admin auth/role):
  - GET /admin/lockouts
    - Query params: user_id (optional), ip (optional)
    - 200 JSON:
      {
        "user_id": "user@example.com" | null,
        "ip": "1.2.3.4" | null,
        "user_counter": 7,
        "ip_counter": 23,
        "user_expires_at": "2026-04-