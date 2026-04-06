# Task - [TASK_001]

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

### **CRITICAL: Wireframe Implementation Requirement (UI Tasks Only)**
IF Wireframe Status = AVAILABLE or EXTERNAL: N/A

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | N/A | N/A |
| Backend | Node.js + Express | 18.x / 4.18.x |
| Database | PostgreSQL | 15.x |
| Cache / Rate-limit Store | Redis | 7.x |
| Library | jsonwebtoken | 9.x |
| Library | argon2 or bcrypt | argon2 v0.30.x (preferred) / bcrypt v5.x (fallback) |
| Library | joi / zod / express-validator | joi v17.x (example) |
| Observability | prom-client / pino | prom-client 14.x / pino 8.x |
| Testing | jest / supertest | jest 29.x / supertest 7.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All code and libraries MUST be compatible with the above versions or newer.

## AI References (AI Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **AI Impact** | No |
| **AIR Requirements** | N/A |
| **AI Pattern** | N/A |
| **Prompt Template Path** | N/A |
| **Guardrails Config** | N/A |
| **Model Provider** | N/A |

### **CRITICAL: AI Implementation Requirement (AI Tasks Only)**
IF AI Impact = Yes: N/A

## Task Overview
Add a backend API route POST /auth/login and its controller to accept application/json credentials payloads, validate input, invoke the authentication service, enforce rate-limiting/lockout checks, issue JWT access tokens with configured TTL/claims, create audit log entries (user id, timestamp, client IP, user-agent), and return standardized responses for success (200), validation errors (400), authentication failures (401), throttling/lockout (429/423), and service unavailable (503). Ensure error responses do not leak internal details or stack traces. Update OpenAPI contract snippet and add unit/integration tests for behavior.

## Dependent Tasks
- .propel/context/tasks/us_001/task_002_be_input_validation_schema.md (input validation schema and middleware)
- .propel/context/tasks/us_001/task_003_be_auth_service.md (AuthService: verify credentials, legacy-hash handling, failed-attempt counting)
- .propel/context/tasks/us_001/task_004_be_token_service.md (JWT issuance and config, secret management)
- US_010 (User data & password storage/hash) — user repository readiness
- US_011 (Token storage & revocation) — token storage/blacklist integration (if required)
- EP-TECH readiness (logging, metrics, secrets management) — secrets and observability infra available
- Redis or equivalent fast store provisioned for rate-limiting/lockout counters

## Impacted Components
- src/routes/auth.routes.ts (new or modified to register route)
- src/controllers/auth.controller.ts (new)
- src/services/auth.service.ts (modified or used — dependency)
- src/services/token.service.ts (JWT generation — dependency)
- src/middleware/validation.middleware.ts (use/modify to validate JSON)
- src/middleware/rateLimiter.middleware.ts (new or modify to enforce thresholds)
- src/models/audit.model.ts (create/modify to record audit entries)
- src/logger/index.ts (ensure structured logs)
- src/openapi/auth.yml or src/docs/openapi.yml (modify to include /auth/login)
- tests/auth/login.controller.spec.ts (new unit/integration tests)
- src/errors/httpErrors.ts (standardized error shapes)

## Implementation Plan
- 1) Add route registration: register POST /auth/login in the router module and wire validation and rate-limit middleware.
- 2) Implement src/controllers/auth.controller.ts:
  - Parse JSON (ensure Content-Type application/json),
  - Use validation middleware to enforce required fields and email format,
  - Extract client IP and user-agent safely,
  - Call AuthService.authenticate({ username, password, clientContext }),
  - Based on AuthService result, respond with:
    - 200 + token payload (access_token, token_type, expires_in) on success,
    - 401 with { error: "Invalid credentials" } on auth failure,
    - 429 or 423 on lockout/throttled,
    - 503 on DB/unavailable errors,
    - 400 on validation errors (handled by validation middleware).
  - Ensure no stack traces or internal details are returned.
- 3) Ensure controller triggers audit logging via AuditModel or AuthService: user id, timestamp, client IP, user-agent.
- 4) Integrate with token service for JWT issuance (token signed with configured key, includes sub, iat, exp, roles).
- 5) Hook into existing metrics/logging: record auth.success, auth.failure counters and latency histogram/timer.
- 6) Update OpenAPI docs to include request/response contract for POST /auth/login.
- 7) Add unit tests and an integration test exercise for:
  - valid login => 200 + token payload and audit record,
  - invalid credentials => 401 + generic error and failed-login recorded,
  - malformed input => 400 with field-level errors,
  - DB unavailable => 503,
  - threshold exceeded => 429/423.
- 8) Run tests, lint, and measure auth median latency during integration tests (assert median < 1s under standard load profile).

## Current Project State
- Server/
  - src/
    - controllers/ (may contain other controllers)
    - routes/
    - services/
    - middleware/
    - models/
    - logger/
    - errors/
    - config/
    - index.ts
  - tests/
  - package.json
  - tsconfig.json
- app/ (frontend - not impacted)
- .propel/context/tasks/us_001/ (task descriptions and dependent task files)

Note: This structure is a placeholder representing current project layout. Update during execution as dependencies are completed.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/controllers/auth.controller.ts | Controller implementing POST /auth/login logic (parsing, calling AuthService, mapping responses, audit logging). |
| CREATE | src/routes/auth.routes.ts | Route registration file exporting router with POST /auth/login wired to controller and middleware. |
| MODIFY | src/services/auth.service.ts | (If exists) Add/ensure authenticate(...) API signature that returns success/failure/lockout and records failed attempts; otherwise create if missing. |
| MODIFY | src/services/token.service.ts | Ensure a generateAccessToken(user, ttl) method exists that signs JWT with configured key and returns token and expires_in. |
| CREATE | src/middleware/rateLimiter.middleware.ts | Middleware to enforce per-IP and per-user thresholds and attach lockout status to request context. |
| CREATE | src/models/audit.model.ts | Simple audit insert function to record auth events (user id, timestamp, client IP, user-agent). |
| MODIFY | src/errors/httpErrors.ts | Add standardized error response shapes used by controller (400, 401, 429/423, 503). |
| MODIFY | src/openapi/auth.yml | Add/update OpenAPI snippet for POST /auth/login request and response schemas. |
| CREATE | tests/auth/login.controller.spec.ts | Unit/integration tests for the route/controller covering success, validation error, invalid credentials, rate-limit, and DB unavailable scenarios. |

## External References
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- JWT RFC / jsonwebtoken docs: https://jwt.io/ and https://github.com/jsonwebtoken/jsonwebtoken
- Argon2 Node bindings: https://github.com/ranisalt/node-argon2
- bcrypt: https://github.com/kelektiv/node.bcrypt.js/
- Example rate limiting patterns (Redis): https://redis.io/docs/manual/eviction/
- OpenAPI 3.0 specification: https://swagger.io/specification/

## Build Commands
- Install deps: npm ci
- Build: npm run build
- Run tests: npm test
- Start dev server: npm run dev
- Lint: npm run lint
(Refer to ../.propel/build/ for CI-specific build steps)

## Implementation Validation Strategy
- [ ] Unit tests pass (jest)
- [ ] Integration tests pass (supertest hitting /auth/login with mocked DB/Redis or test instances)
- [ ] auth.success, auth.failure, auth.latency metrics are emitted; validate metrics exist in test harness
- [ ] Audit entries recorded for successful and failed logins (verify test DB or mocked model calls)
- [ ] Response shapes and HTTP status codes conform to Acceptance Criteria (200, 400, 401, 429/423, 503)
- [ ] No internal stack traces or debug details are present in any API response
- [ ] Median authentication latency < 1s in integration test load profile (measure and assert)

## Implementation Checklist
- [ ] Add POST /auth/login route in src/routes/auth.routes.ts and register in main router (accept application/json). (Estimated 1h)
- [ ] Implement src/controllers/auth.controller.ts to parse input, call AuthService.authenticate, map results to 200/400/401/429/423/503 responses, and call audit model. (Estimated 2h)
- [ ] Ensure validation middleware (joi/validator) enforces required fields and email format and returns 400 with field-level messages; wire into route. (Estimated 1h)
- [ ] Integrate token.service.generateAccessToken to sign JWT with sub, iat, exp, roles and TTL from config; include expires_in in response. (Estimated 0.5h)
- [ ] Implement/plug-in rateLimiter.middleware to enforce thresholds and return 429/423 with generic message; increment failed attempts on authentication failure. (Estimated 1h)
- [ ] Add/modify tests in tests/auth/login.controller.spec.ts covering success, invalid credentials, validation errors, rate-limit, and DB unavailable. (Estimated 1.5h)
- [ ] Update OpenAPI documentation snippet and run lint/tests. (Estimated 0.5h)

Rules:
- Total effort <= 8 hours (listed per checklist estimates sum ~7.5h)
- Checklist contains <= 8 items and actionable steps.

## RULES:
- Implementation Checklist must have <=8 items
- Effort must be <=8 hours
- Be specific and actionable — no vague descriptions
- Expected Changes table lists concrete file paths (CREATE/MODIFY)
- Acceptance Criteria are from the parent User Story

