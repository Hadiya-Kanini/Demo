# Task - task_003

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:  
    - Given a registered user with valid credentials, When the client sends a POST /auth/login with a JSON payload, Then the server responds with HTTP 200 and a JSON body containing an access_token (JWT), token_type ("Bearer"), and expires_in.  
      - The token is signed with the system's configured signing key, contains standard claims (sub, iat, exp, roles), and expires after the configured TTL.  
      - The response body contains no stack traces or internal error details.  
    - Given a user supplies incorrect credentials, When the client POSTs to /auth/login, Then the server responds with HTTP 401 and a generic error message: {"error":"Invalid credentials"}  
      - Failed login attempt is recorded (user identifier or IP) for rate-limiting and monitoring.  
      - Response time and payload size remain within acceptable limits to avoid timing/side-channel leaks.  
    - Given repeated failed attempts that exceed configured thresholds, When threshold is reached, Then subsequent requests return HTTP 429 (or 423 if account locked) with a generic message ("Too many attempts, try again later"), and the system enforces backoff/lockout per security policy. Alerts and audit logs must be created for suspected brute-force activity.  
    - Given a successful login, When the server issues a token, Then an audit record is created (user id, timestamp, client IP, user-agent), and integration tests demonstrate median authentication latency meets NFR (auth median < 1s) under standard load.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable" and a retryable error code; do not leak internal details.
    - Password hash algorithm/version mismatch (legacy records) — handle by verifying using the legacy algorithm, force re-hash on next successful login, and record upgrade event without revealing algorithm differences.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.

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
| Backend | Node.js + TypeScript (Express) | Node 18+, TypeScript 5.x, Express 4.x |
| Database | PostgreSQL | 13+ |
| Library | bcrypt, argon2, jsonwebtoken, pg (or ORM e.g., Prisma) | bcrypt v5+, argon2 v0.30+, jsonwebtoken v9+ |
| Library | Redis (rate-limiting/lockout) | Redis 6+ |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All code and libraries must be compatible with the versions above.

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
Implement AuthService.verifyCredentials(userIdentifier: string, password: string, context: { ip?: string; userAgent?: string }) to:
- Validate credentials against UserRepository.
- Support both current and legacy hash verification; when legacy verification succeeds, re-hash using the current algorithm and update the user record (re-hash-on-success).
- Emit structured events/metrics: auth.success, auth.failure, auth.rehash.
- Increment and consult failed-login counters (via Lockout/RateLimiter service) and return normalized failure responses for invalid credentials.
- Handle database/unavailable dependencies gracefully and surface a retryable ServiceUnavailable error for upstream handlers to translate to HTTP 503.
- Ensure failures are normalized (do not reveal whether username or password was wrong), and timing/size leakage is considered.

## Dependent Tasks
- .propel/context/tasks/us_001/task_001_be_login_route_controller.md (Login controller/route must exist to call AuthService)
- .propel/context/tasks/us_001/task_002_be_input_validation_schema.md (Input validation must run before calling verifyCredentials)
- US_010 (User data & password storage/hash) — repository methods: findByUsernameOrEmail, updatePasswordHash
- US_011 (Token storage & revocation) — token issuance integration (AuthService will call TokenService after successful verify)
- Infra tasks: Redis/Lockout service provision (LockoutService) and observability/metrics exporters

## Impacted Components
- Server/src/services/auth/AuthService.ts (new method verifyCredentials)
- Server/src/repositories/UserRepository.ts (ensure support for updatePasswordHash)
- Server/src/services/auth/passwordHasher.ts (compare/rehash implementations for current and legacy)
- Server/src/services/lockout/LockoutService.ts (increment/check failed attempts)
- Server/src/events/EventBus.ts (emit auth events)
- Server/src/services/metrics/AuthMetrics.ts (auth.success/auth.failure counters and latency timer)
- Server/test/services/auth/AuthService.verifyCredentials.spec.ts (unit tests)

## Implementation Plan
1. Define the method signature and error types:
   - verifyCredentials(identifier: string, password: string, ctx: {ip?: string; userAgent?: string}): Promise<{ userId: string } | AuthFailure>
   - AuthFailure should be a normalized error type (e.g., throw InvalidCredentialsError or return a well-known Result object).
2. Fetch user record via UserRepository.findByUsernameOrEmail(identifier). Handle not-found and DB errors:
   - On DB errors/timeouts, throw ServiceUnavailableError (to be mapped to HTTP 503 by controller).
   - If not found => treat as invalid credentials (do not leak).
3. Check account lockout/threshold using LockoutService.isLocked(userId | ip). If locked => throw AccountLockedError (mapped to 423) or RateLimitError (429).
4. Verify password:
   - Call PasswordHasher.verify(storedHash, suppliedPassword) which returns { matches: boolean, legacy: boolean }.
   - If matches && legacy === true => asynchronously re-hash with current algorithm (PasswordHasher.rehash) and call UserRepository.updatePasswordHash(userId, newHash). Emit auth.rehash event.
   - If matches => success: emit auth.success event/metric, reset failed-login counters LockoutService.reset(userId|ip), return success outcome (userId). Token issuance is responsibility of higher-level service/controller.
   - If not matches => increment failed-login counters LockoutService.increment(userId|ip), emit auth.failure, return/throw InvalidCredentialsError.
5. Ensure timing uniformity:
   - If user not found, perform a fake hash verification (verify against a constant dummy hash) to make timing similar to existing-user path.
6. Logging and metrics:
   - Do not log sensitive data (no passwords, no raw hashes). Emit structured logs with userId (if known), event type, and safe context (ip, userAgent).
   - Record auth.latency via AuthMetrics.timer for this method.
7. Tests:
   - Unit tests for success (current hash), success (legacy hash triggers rehash), failure increments counters, DB unavailable leads to ServiceUnavailableError, lockout path returns appropriate error.
8. Document method in AuthService interface and add OpenAPI/service-level notes that verifyCredentials returns normalized failures.

## Current Project State
- Server/
  - src/
    - controllers/
      - authController.ts (calls AuthService) [exists or planned]
    - services/
      - auth/
        - AuthService.ts (partial; implement verifyCredentials)
        - passwordHasher.ts (may be created/updated)
      - lockout/
        - LockoutService.ts (exists/used by auth)
      - metrics/
        - AuthMetrics.ts (exists)
    - repositories/
      - UserRepository.ts (exists: findByUsernameOrEmail, may need updatePasswordHash)
    - events/
      - EventBus.ts (publish/subscribe)
  - test/
    - services/
      - auth/
        - AuthService.verifyCredentials.spec.ts (to create)
- .propel/context/tasks/us_001/ (task files for the user story)

(Note: Update this tree as dependent tasks complete.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| MODIFY | Server/src/services/auth/AuthService.ts | Implement verifyCredentials(identifier, password, ctx). Add imports and use LockoutService, PasswordHasher, UserRepository, EventBus, AuthMetrics. |
| CREATE | Server/src/services/auth/passwordHasher.ts | Provide verify(storedHash, password) supporting current (argon2/bcrypt) and legacy verification; provide rehash(password) to produce current algorithm hash. Export verify and rehash functions. |
| MODIFY | Server/src/repositories/UserRepository.ts | Add or ensure updatePasswordHash(userId: string, newHash: string): Promise<void> is implemented and used by AuthService. |
| MODIFY | Server/src/services/lockout/LockoutService.ts | Ensure API: isLocked(userId|ip), increment(userId|ip), reset(userId|ip) are available. Add any small adapters if required. |
| MODIFY | Server/src/events/EventBus.ts | Ensure event types: auth.success, auth.failure, auth.rehash can be emitted; no functional change, may add constants. |
| CREATE | Server/test/services/auth/AuthService.verifyCredentials.spec.ts | Unit tests for success/current hash, success/legacy->rehash, invalid credentials increments counters and returns normalized error, DB unavailable => ServiceUnavailableError, lockout path. |

## External References
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- argon2 npm: https://www.npmjs.com/package/argon2
- bcrypt npm: https://www.npmjs.com/package/bcrypt
- jsonwebtoken docs: https://github.com/auth0/node-jsonwebtoken
- Password hash migration patterns: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html#migrating-passwords
- Security: constant-time comparison considerations: https://auth0.com/blog/stop-using-sha1-for-passwords/

## Build Commands
- Install: npm ci
- Build: npm run build
- Test: npm test
- Lint: npm run lint

(Refer to overarching build scripts at ../.propel/build/)

## Implementation Validation Strategy
- [ ] Unit tests pass (Server/test/services/auth/AuthService.verifyCredentials.spec.ts)
- [ ] Integration tests for login flow (when combined with controller) pass in separate task
- [ ] Metrics emitted: auth.success, auth.failure, auth.rehash, auth.latency
- [ ] Failed-login counters increment and reset behavior verified in tests
- [ ] Legacy-hash re-hash path is tested to confirm updatePasswordHash is invoked
- [ ] Database unavailable behavior propagates ServiceUnavailableError (mapped by controller to HTTP 503)
- [ ] Timing normalization: tests include a timing-roughness check to ensure not trivially revealing (unit-level approximate)

## Implementation Checklist
- [ ] Add/modify AuthService.verifyCredentials(identifier, password, ctx) in Server/src/services/auth/AuthService.ts per plan
- [ ] Implement PasswordHasher.verify and PasswordHasher.rehash in Server/src/services/auth/passwordHasher.ts supporting legacy detection
- [ ] Use LockoutService to check isLocked, increment on failure, reset on success
- [ ] Emit events via EventBus: auth.success, auth.failure, auth.rehash; and record metrics via AuthMetrics
- [ ] On legacy-password match, asynchronously update repository with new hash via UserRepository.updatePasswordHash and emit auth.rehash event
- [ ] On DB or repository errors, throw ServiceUnavailableError (do not leak details)
- [ ] Create unit tests covering success (current), success (legacy -> rehash), invalid credentials, DB unavailable, and lockout condition

Effort estimate: 6 hours

RULES:
- Implementation Checklist contains 7 items (<=8)
- Effort estimate <=8 hours

