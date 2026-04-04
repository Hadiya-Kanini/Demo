## Task ID: task_004
## Task Title: Implement Redis-backed rate-limiting and account lockout
## Category: Backend

## Description
Implement a Redis-backed failed-login rate-limiting and account lockout subsystem used by the authentication flow. Provide:
- Atomic increment and check operations for failed-login counters using two supported algorithms: sliding-window (sorted-set) and token-bucket (counter + refill policy).
- Public/internal APIs to increment failed-login attempts, check current rate-limit/lockout status, and optionally reset counters after successful login.
- Configurable thresholds (per-user, per-IP, global), time windows, lockout durations, and backoff strategy (exponential/preset).
- Enforcement of lockout/backoff at check time (service returns whether to allow an auth attempt; auth flow will use this).
- Alerting hooks (pluggable dispatcher) that are invoked when suspicious activity is detected (e.g., threshold crossed, rapid-fire attempts).
- An in-memory implementation that mirrors Redis behavior for unit tests and CI (same interface).
- Robustness: Redis connection failures must fall back to in-memory logic (configurable) and emit a telemetry/logging event.
- Observability: structured logs/metrics for increments, hits, lockouts, and fallback events.

Traceability: Parent Story us_001 — this task implements AC mapping for failed-login counters and lockout/backoff in the authentication acceptance criteria.

## Acceptance Criteria
All criteria must be verifiable in automated tests or integration runs:

1. Increment/Check API:
   - Given a failed login attempt (userID/email or IP), calling the Increment API stores a token/event in Redis (or in-memory fallback) and returns the current count and lockout status.
   - Given N failed attempts within the configured window W (configurable), Check API returns "locked" (or shouldBlock=true) when threshold exceeded.

2. Sliding-window and token-bucket algorithms:
   - Both algorithms are selectable via configuration (e.g., RATE_LIMIT_ALGO = sliding_window | token_bucket) and produce equivalent blocking behavior given same thresholds and windows in integration tests.

3. Configurability:
   - Thresholds (attempts), window (seconds), lockout duration (seconds), backoff multiplier, per-IP and per-user caps are read from config/env and can be updated without code changes.
   - Defaults are provided and documented.

4. Lockout enforcement:
   - After lockout begins for a user or IP, Check API returns blocked until lockout TTL expires.
   - Backoff can increase lockout duration after repeated lockouts (exponential or configured increments).

5. Alerting hooks:
   - When a configured alerting threshold is crossed (e.g., >X attempts in Y seconds), the alert hook is invoked with standardized payload: {subject: userId|ip, metric, window, timestamp, sampleEvents}.
   - Unit tests assert that the alert hook receives calls under correct conditions.

6. In-memory fallback:
   - When Redis is unreachable or disabled in config, the system falls back to in-memory implementation with identical semantics for the duration of process uptime and emits a telemetry/log event.
   - Integration tests simulate Redis failure and verify fallback behavior.

7. Atomicity & correctness:
   - Redis operations used are atomic (use of Lua script or strict Redis primitives) to avoid race conditions under concurrent increments; tests simulate concurrent attempts and verify count consistency.

8. Observability:
   - The module emits structured logs for key events (increment, blocked, lockoutStarted, fallbackActivated, alertDispatched) and exposes simple metrics counters (rate_limiter.increments, rate_limiter.blocked, rate_limiter.lockouts, rate_limiter.fallbacks).

9. API surface for auth flow:
   - The authentication endpoint can call Check before verifying credentials and increment on failed verification; documented examples provided in README and code comments.

## Technical Specifications

APIs/Endpoints
- Internal service functions (preferred):
  - RateLimiter.increment(subject: Subject): Promise<IncrementResult>
    - Subject = { type: "user" | "ip", id: string }
    - IncrementResult = { count: number, allowed: boolean, lockedUntil?: ISO8601 }
  - RateLimiter.check(subject: Subject): Promise<CheckResult>
    - CheckResult = { count: number, allowed: boolean, lockedUntil?: ISO8601, nextAllowedAt?: ISO8601 }
  - RateLimiter.reset(subject: Subject): Promise<void> (used on successful login)
  - RateLimiter.forceLock(subject: Subject, durationSeconds: number, reason?: string): Promise<void> (admin/hooks)
  - Health/diagnostic: RateLimiter.status(): Promise<{ backend: "redis"|"memory", redisConnected?: boolean }>

- Optional HTTP internal endpoints (if project requires internal HTTP access):
  - POST /internal/rate-limit/increment
    - body: { type: "user"|"ip", id: string, meta?: { ip?:string, userAgent?:string } }
    - response: IncrementResult (as above)
  - GET /internal/rate-limit/status?type=user&id=...
    - response: CheckResult

Components/Classes
- Interface: IRateLimiter
  - Methods: increment(), check(), reset(), forceLock(), status()
  - Designed for DI (constructor-injected), testable via mocks.

- Implementations:
  - RedisRateLimiter implements IRateLimiter
    - Uses Redis client (connection pooled)
    - Uses:
      - Sliding-window implementation: Redis Sorted Set per subject key: ZADD timestamp member, ZREMRANGEBYSCORE old, ZCOUNT range
      - Token-bucket implementation: Key storing tokens + last_refill timestamp; implemented with Lua script for atomic refill/consume
    - Atomic operations via Lua scripts (bundled under /server/security/redis_lua/*.lua)
    - Keys:
      - failed:{type}:{id}:events -> ZSET (timestamps)
      - bucket:{type}:{id} -> HASH (tokens, last_refill)
      - lock:{type}:{id} -> STRING (lockedUntil timestamp)
    - TTLs and expirations are applied to keys to avoid unbounded