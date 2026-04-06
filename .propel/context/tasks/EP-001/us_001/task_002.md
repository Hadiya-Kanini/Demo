## Task ID: task_002
## Task Title: Implement password verification & legacy-hash upgrade flow
## Category: Backend

## Description
Implement secure password verification for POST /auth/login that:
- Looks up a user by username or email.
- Validates incoming request payload and returns 400 on malformed requests.
- Verifies supplied password against the stored hash using the current hashing algorithm (e.g., Argon2id / bcrypt) and supports verification against one or more legacy algorithms (e.g., PBKDF2, SHA-256+salt) when user records indicate an older hash version.
- When a legacy hash is successfully verified, re-hash the password using the current algorithm and atomically update the user's stored password_hash and hash_algo_version.
- On every failed authentication attempt increment failed-login counters (per-user if user exists, and per-IP / global metric as configured), and apply lockout/throttling policy when thresholds are exceeded.
- Emit audit log entries for successful logins (user id, timestamp, client IP, user-agent) and for legacy-hash upgrades.
- Return appropriate status codes and non-revealing error messages for invalid credentials, malformed input, and service errors.

Traceability: parent user story us_001 (Implement Login API).

## Acceptance Criteria
1. API behavior
   - Given valid credentials (existing user, correct password) POST /auth/login returns HTTP 200 with JSON:
     {
       "access_token": "<JWT>",
       "token_type": "Bearer",
       "expires_in": <seconds>
     }
     and no sensitive debug data in the body.
   - Given invalid username/email or password, POST /auth/login returns HTTP 401 with body { "error": "Invalid credentials" } and increments failed-login counters for the user (if user exists) and IP metrics.
   - Given malformed or missing required fields (e.g., missing password, invalid email format), POST /auth/login returns HTTP 400 with validation error referencing fields.
   - Given the database is unavailable during verification, API returns HTTP 503 with { "error": "Service temporarily unavailable" }.

2. Legacy-hash upgrade
   - If a user record has hash_algo_version != CURRENT_ALGO and password verification succeeds using the legacy algorithm, the system:
     - Re-hashes password using CURRENT_ALGO.
     - Atomically updates password_hash and hash_algo_version in the users table without creating a second session gap.
     - Logs an audit entry indicating "password_hash_upgraded" with user id and timestamp.
     - Does not create any additional authentication sessions beyond normal successful-login behavior.

3. Security & observability
   - Successful login emits an auth audit log entry with fields: user_id, timestamp(UTC), client_ip, user_agent, outcome="success".
   - Failed login attempts increment user.failed_login_count and set last_failed_at; IP-level metrics are incremented separately.
   - Lockout or throttling is triggered after configured threshold (configurable constant) and produces HTTP 429 or 423 with a generic message when applied.
   - JWT issued uses configured TTL and contains user id and standard claims; expires_in equals configured TTL in seconds.

4. Correctness & atomicity
   - Legacy re-hash and user record update happen inside a single transactional update; no race allows an attacker to reuse stale data.
   - No plaintext passwords are logged or persisted anywhere.

5. Performance
   - Integration tests measure median response latency for successful auth and assert median < 1s (NFR target); if hashing parameters exceed this, document performance tradeoffs and propose configuration tuning.

## Technical Specifications

APIs/Endpoints
- POST /api/auth/login
  - Request JSON:
    - username_or_email: string (required)
    - password: string (required)
    - client_metadata: { user_agent?: string, client_ip?: string } (optional; server should prefer actual request headers)
  - Responses:
    - 200 OK: { access_token, token_type: "Bearer", expires_in }
    - 400 Bad Request: { error: "Validation error", details: { field: message } }
    - 401 Unauthorized: { error: "Invalid credentials" }
    - 423 Locked or 429 Too Many Requests: { error: "Too many attempts" } (depending on policy)
    - 503 Service Unavailable: { error: "Service temporarily unavailable" }

Components/Classes (suggested)
- AuthController (or AuthHandler)
  - Endpoint binding for POST /api/auth/login; input validation; uses AuthService.
- AuthService
  - Orchestrates lookup, verification, legacy upgrade, JWT creation, and audit logging.
- UserRepository (interface + implementation)
  - Methods: findByUsernameOrEmail(identifier), updatePasswordHash(userId, newHash, newAlgoVersion), incrementFailedLogin(userId), setLockout(userId, until).
- PasswordVerifier (interface)
  - verify(password, storedHash, algoVersion): Promise<boolean>
- PasswordHasher (interface)
  - hash(password): Promise<{ hash: string, algoVersion: string }>
- LegacyHashAdapters (one per legacy algo)
  - verifyLegacy(password, storedHash, legacyParams): boolean
- AuditLogger
  - logAuthEvent(userId?, eventType, metadata)
- MetricsService
  - incrementCounter(name, labels)
- RateLimiter/LockoutService
  - enforceIpAndUserLimits(ip, userId) => possibly throws RateLimitError
- JwtService
  - createToken(payload, ttlSeconds)
- TransactionManager / Database session wrapper
  - Ensure updatePasswordHash is atomic w.r.t. verification/upgrades.

Data Models
- users table (existing; required fields used here)
  - id: UUID / bigint
  - username: string (unique)
  - email: string (unique)
  - password_hash: string
  - hash_algo_version: string (e.g., "argon2id_v1", "pbkdf2_v0")
  - failed_login_count: int (default 0)
  - last_failed_at: timestamp nullable
  - locked_until: timestamp nullable
  - other fields (created_at, updated_at)
- auth_audit_logs table (new or existing)
  - id: UUID