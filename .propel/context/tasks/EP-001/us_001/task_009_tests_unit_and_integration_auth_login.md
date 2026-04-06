# Task - task_009

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
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

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React | 18.x |
| Backend | Node.js / Express | Node.js 18.x, Express 4.x, TypeScript 5.x |
| Database | PostgreSQL | 15.x |
| Library | Jest / Supertest / ts-jest | Jest 29.x, Supertest 6.x, ts-jest |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All tests and libraries must be compatible with the specified versions.

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
Write unit tests for login-related modules (validation schema, AuthService, TokenService) and end-to-end/integration tests for POST /auth/login that cover:
- valid login (200 with correct JWT shape and TTL),
- invalid credentials (401 with generic message and failed-login counter increment),
- malformed input (400 with field-level validation errors),
- rate-limit / lockout scenarios (429 or 423 as applicable),
- DB unavailable (503).
Integration tests must assert audit log entry creation and measure median authentication latency to verify NFR (median < 1s) under standard test load.

## Dependent Tasks
- .propel/context/tasks/us_001/task_001_be_login_route_controller.md (POST /auth/login route & controller)
- .propel/context/tasks/us_001/task_002_be_input_validation_schema.md (validation schema)
- .propel/context/tasks/us_001/task_003_be_auth_service.md (AuthService: credential verification, legacy-hash handling)
- .propel/context/tasks/us_001/task_004_be_token_service.md (TokenService: JWT issuance, TTL)
- .propel/context/tasks/us_001/task_005_be_user_repository.md (User repository / test fixtures)
- .propel/context/tasks/us_001/task_006_infra_rate_limit_redis.md (Rate-limiter infra / in-memory or Redis stub)
- .propel/context/tasks/us_001/task_007_db_migrations_audit.md (Audit log schema/migration)

These must be available (or their test doubles/mocks) before integration tests run.

## Impacted Components
- src/controllers/auth.controller.ts (reads request, calls AuthService)
- src/services/auth.service.ts (verify credentials, record failures, trigger re-hash)
- src/services/token.service.ts (issue JWT, set TTL)
- src/middleware/validation.ts (request validation)
- src/repositories/user.repository.ts (user lookup)
- src/infra/rateLimiter.ts (failed attempts counter, lockout logic)
- src/models/audit.model.ts (audit log persistence)
- tests/unit/*.test.ts (new)
- tests/integration/login.integration.test.ts (new)
- test-utils/ (test setup helpers)

## Implementation Plan
- Create unit tests:
  - validation.test.ts: validate accepted/rejected payloads and assert validation error messages for malformed input.
  - auth.service.test.ts: mock user repository and password hashing to test success, invalid credentials path (ensuring increment of failed-login counter), legacy-hash success triggers re-hash call.
  - token.service.test.ts: verify JWT payload claims, signing, and TTL (exp - iat equals configured TTL).
- Create integration test suite using Supertest against a test instance of the app:
  - Start app with test configuration (test DB, test Redis or in-memory rate limiter).
  - Seed a test user with known password hash.
  - Tests to exercise: valid login (assert 200, response shape, JWT claims, audit log row), invalid credentials (401 and failed-login counter increment), malformed input (400 with field-level errors), repeated failures until rate-limiter threshold (assert 429/423).
  - Test DB-unavailable behavior: simulate DB down (stop DB or mock user repo to throw) and assert 503 response with generic message.
  - Capture request durations and compute median across N runs to assert median < 1s.
- Add test utilities:
  - test-utils/test-setup.ts for spinning test services/migrations, seeding and cleanup.
  - jest.config.ts and test script updates in package.json.
- Use mocking for unit tests (jest.mock) and real infra for integration (docker-compose.test.yml or in-memory alternatives).
- Ensure tests assert no debug/stack traces in responses and audit logs contain required fields (user id, timestamp, client IP, user-agent).
- Add CI-friendly commands to run tests and a short local README for running integration tests.

## Current Project State
- app/
  - src/
    - controllers/
      - auth.controller.ts
    - services/
      - auth.service.ts
      - token.service.ts
    - middleware/
      - validation.ts
    - repositories/
      - user.repository.ts
    - infra/
      - rateLimiter.ts
    - models/
      - audit.model.ts
    - app.ts (Express app bootstrap)
    - server.ts
- tests/
  - (empty — tests to be added)
- package.json
- tsconfig.json
- .propel/context/tasks/us_001/ (task artifacts)
Note: This is a placeholder snapshot; dependent tasks must be completed or test doubles provided.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/validation.test.ts | Unit tests for request validation covering missing fields and invalid email formats. |
| CREATE | tests/unit/auth.service.test.ts | Unit tests for AuthService: success, invalid creds, legacy-hash re-hash trigger, failure counting. |
| CREATE | tests/unit/token.service.test.ts | Unit tests for TokenService: JWT claims, signature, TTL correctness. |
| CREATE | tests/integration/login.integration.test.ts | Integration tests for POST /auth/login covering 200, 401, 400, 429/423, 503 and audit + latency checks. |
| CREATE | test-utils/test-setup.ts | Helpers to start test app, seed DB, reset rate-limiter, capture audit entries, measure timings. |
| CREATE | jest.config.ts | Jest configuration for TypeScript and integration test environment. |
| MODIFY | package.json | Add "test", "test:unit", "test:integration" scripts and test-related devDependencies. |
| CREATE | tests/fixtures/users.sql | SQL to seed test user(s) for integration tests. |
| CREATE | docker-compose.test.yml | (optional) Compose file to run test Postgres and Redis for integration tests. |

## External References
- Jest docs: https://jestjs.io/docs/getting-started
- Supertest docs: https://github.com/visionmedia/supertest
- JSON Web Token RFC / jsonwebtoken library: https://github.com/jsonwebtoken/jsonwebtoken
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Postgres documentation: https://www.postgresql.org/docs/
- Redis rate-limiting patterns: https://redis.io/docs/manual/patterns/

## Build Commands
- Local unit tests: npm run test:unit
- Integration tests (requires test infra): npm run test:integration
- Combined tests: npm test
- (Refer to repository build wrappers): ../.propel/build/
- Sample scripts to add to package.json:
  - "test": "jest --runInBand",
  - "test:unit": "jest --config=jest.config.ts --runTestsByPath tests/unit",
  - "test:integration": "NODE_ENV=test jest --config=jest.config.ts tests/integration --runInBand"

## Implementation Validation Strategy
- [ ] Unit tests pass (validation, AuthService, TokenService)
- [ ] Integration tests pass for all scenarios (200, 401, 400, 429/423, 503)
- [ ] Audit log entries verified in DB for successful logins (user id, timestamp, client IP, user-agent)
- [ ] Failed-login counter increments verified after invalid credential attempts
- [ ] Median authentication latency measured in integration tests < 1s
- [ ] Tests assert responses contain no internal stack traces or debug details

## Implementation Checklist
- [ ] Add unit tests in tests/unit for validation, AuthService, and TokenService (mock external dependencies).
- [ ] Add integration test tests/integration/login.integration.test.ts using Supertest and test setup helpers.
- [ ] Implement test-utils/test-setup.ts to start app in test mode, apply migrations, seed test user, and reset rate-limiter between tests.
- [ ] Add fixtures/tests/fixtures/users.sql and seed logic to create a known test user with a deterministic password hash.
- [ ] Update package.json and add jest.config.ts so CI can run unit and integration tests separately.
- [ ] Verify JWT issued in integration test contains claims (sub, iat, exp, roles) and TTL equals configured value.
- [ ] Assert audit log row created and failed-login counters incremented; simulate DB unavailable to assert 503.
- [ ] Measure and assert median latency across repeated valid-login requests is < 1s.

Effort estimate: 6 hours

## RULES:
- Implementation Checklist has <=8 items
- Effort <=8 hours

