# Task - [TASK_008]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-001/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a registered user with valid credentials, When the client sends a POST /auth/login with a JSON payload:
      {
        "username": "user@example.com",
        "password": "correct-password"
      }
      Then the server responds with HTTP 200 and a JSON body containing access_token, token_type ("Bearer"), and expires_in (seconds). The token is signed with the configured signing key, contains standard claims (sub, iat, exp, roles), expires after the configured TTL, and the response body contains no stack traces or internal error details.
    - Given a user supplies incorrect credentials, When the client POSTs to /auth/login, Then the server responds with HTTP 401 and a generic error message {"error":"Invalid credentials"}; failed login attempt is recorded for rate-limiting and monitoring; response time and payload size avoid timing/side-channel leaks.
    - Given malformed input (missing fields, invalid JSON, invalid email format), When the client POSTs to /auth/login, Then the server responds with HTTP 400 with a validation error identifying invalid fields and does not proceed to password verification.
    - Given repeated failed attempts that exceed configured thresholds, When threshold is reached, Then subsequent requests return HTTP 429 (or 423 if account locked) with a generic message ("Too many attempts, try again later"), and the system enforces backoff/lockout per policy; alerts and audit logs must be created.
    - Given a successful login, When the server issues a token, Then an audit record is created (user id, timestamp, client IP, user-agent), and integration tests demonstrate median authentication latency meets NFR (auth median < 1s) under standard load.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable, please try again later". Do not leak internal details. Trigger an incident metric and circuit-breaker behavior to prevent cascading failures.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.
    - Password hash algorithm/version mismatch (legacy records) — detect legacy hash and re-hash on successful login; log the re-hash event without revealing algorithm differences.

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
| Backend | Node.js + Express + TypeScript | Node 18.x, Express 4.x, TypeScript 5.x |
| Database | PostgreSQL | 16.x |
| Library | pg (Postgres client), redis (optional) | pg v8, ioredis v5 |
| Library | Circuit-breaker library option / custom | opossum v6 (optional) or custom lightweight implementation |
| Testing | Jest | v29.x |
| Observability | Prometheus metrics / structured logging | prom-client (Node) / pino |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries referenced must be compatible with the versions above.

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
Implement graceful handling when the user-store (database) is unavailable during credential verification. Add a fail-fast/circuit-breaker mechanism around the user-repository calls used by the authentication flow to prevent cascading failures. Map DB-unavailability errors to a safe HTTP 503 response with a generic message. Emit metrics and logs for incident detection. Provide unit and integration tests that simulate DB errors and verify 503 responses and circuit-breaker behavior (open/half-open/close transitions or simplified fail-fast counts).

## Dependent Tasks
- .propel/context/tasks/EP-001/us_001/task_001_be_login_route_controller.md (Login route/controller implemented)
- .propel/context/tasks/EP-001/us_001/task_002_be_input_validation_schema.md (Validation schema implemented)
- Implementation or interface of UserRepository (US_010) — repository methods used by AuthService must exist
- AuthService / authentication flow (issue tokens, call user repository)
- Observability/logging infra available (metrics exporter, structured logger)
- CI test harness (Jest) configured

## Impacted Components
- src/services/authService.ts — will call userRepo via circuit-breaker
- src/repositories/userRepository.ts — may surface DBUnavailableError
- src/lib/circuitBreaker.ts — new module (circuit-breaker abstraction)
- src/errors/DBUnavailableError.ts — new error type
- src/middleware/errorHandler.ts — map DBUnavailableError -> 503
- src/metrics/authMetrics.ts — emit auth.service_unavailable, auth.circuit_open metrics
- tests/auth/db-unavailable.test.ts — new tests simulating DB failures
- tests/circuit/circuit-breaker.test.ts — new tests for circuit-breaker behavior

## Implementation Plan
- Define a specific error class DBUnavailableError to represent transient/unavailable user-store failures.
- Implement a lightweight circuit-breaker module (src/lib/circuitBreaker.ts) with configurable thresholds: failureThreshold, resetTimeout, and optional sliding window. Support states: CLOSED, OPEN, HALF_OPEN. Expose an execute(fn) method that fails fast when OPEN.
  - Option A: Wrap an existing library (opossum) if present in repo; Option B: implement small in-process circuit-breaker if cross-process Redis is not available.
- Integrate the circuit-breaker around calls from AuthService to UserRepository (e.g., circuitBreaker.execute(() => userRepo.findByEmail(...))).
- Update UserRepository to throw DBUnavailableError when DB client throws connection/timeout errors (catch pg errors like 'ECONNREFUSED', '57P01', 'ETIMEDOUT', etc.) and bubble those errors.
- Update global error handling middleware to:
  - Map DBUnavailableError to HTTP 503 with body {"error":"Service temporarily unavailable, please try again later"} and an idempotent retryable error code header (e.g., Retry-After: 5).
  - For circuit-breaker OPEN state, return same 503 message and emit metric auth.circuit_open.
  - Ensure no internal error details or stack traces are returned.
- Emit metrics for incident detection:
  - auth.service_unavailable counter (increment on DBUnavailableError)
  - auth.circuit_open counter/gauge
  - auth.db_error_latency histogram (optional)
- Write unit tests:
  - Simulate userRepository throwing DBUnavailableError and assert that POST /auth/login returns HTTP 503 with correct body and no internals.
  - Simulate repeated DB errors to trip the circuit-breaker; assert subsequent requests fail fast and return 503 without hitting the repository.
  - Test half-open behavior by simulating a success after resetTimeout and verify circuit transitions back to CLOSED.
- Add integration test(s) if repo/DB mocking infra exists to emulate actual DB connection failures.
- Document configuration knobs in README or config schema: CB_FAILURE_THRESHOLD, CB_RESET_TIMEOUT_MS, CB_WINDOW_SIZE.
- Add/modify package.json test script references if necessary.

## Current Project State
- app/
  - src/
    - services/
      - authService.ts
    - repositories/
      - userRepository.ts
    - middleware/
      - errorHandler.ts
    - metrics/
      - authMetrics.ts
    - index.ts (Express app entry)
- Server/
  - package.json
  - tsconfig.json
  - jest.config.js
  - .propel/context/tasks/EP-001/us_001/ (task docs)
- tests/
  - auth/
    - existing tests for login flow
  - utils/
    - httpClientMock.ts

(Actual files may differ; update this placeholder when dependent tasks have completed.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/errors/DBUnavailableError.ts | Defines DBUnavailableError extends Error with metadata for mapping to 503. |
| CREATE | src/lib/circuitBreaker.ts | Lightweight circuit-breaker implementation with execute(fn) and state management. |
| MODIFY | src/services/authService.ts | Wrap userRepository calls with circuitBreaker.execute(...) and translate repository errors upward. |
| MODIFY | src/repositories/userRepository.ts | Catch DB/pg client connection errors and rethrow DBUnavailableError for upstream handling. |
| MODIFY | src/middleware/errorHandler.ts | Map DBUnavailableError and circuit-open responses to HTTP 503 with generic message; add Retry-After header. |
| MODIFY | src/metrics/authMetrics.ts | Add counters/gauges for auth.service_unavailable and auth.circuit_open; increment where appropriate. |
| CREATE | tests/auth/db-unavailable.test.ts | Tests to simulate DBUnavailableError and verify HTTP 503 response and no internal details. |
| CREATE | tests/circuit/circuit-breaker.test.ts | Tests to simulate repeated failures to trip circuit and verify fail-fast behavior and recovery. |

## External References
- Circuit Breaker pattern — Martin Fowler: https://martinfowler.com/bliki/CircuitBreaker.html
- Opossum (Node circuit-breaker lib): https://nodeshift.dev/opossum/
- HTTP 503 Service Unavailable — RFC7231: https://datatracker.ietf.org/doc/html/rfc7231#section-6.6.4
- Resilience patterns in Node: https://www.npmjs.com/package/opossum
- Handling Postgres connection errors (pg): https://node-postgres.com/features/connecting

## Build Commands
- Refer to project build file: ../.propel/build/
- Local commands:
  - Install: npm ci
  - Build: npm run build
  - Test: npm test
  - Lint: npm run lint
  - Run dev server: npm run dev

## Implementation Validation Strategy
- [ ] Unit tests pass (jest)
- [ ] Integration tests pass (if applicable)
- [ ] auth.service_unavailable metric emitted when DB errors occur
- [ ] Circuit-breaker trips after configured failure threshold and returns HTTP 503 for fail-fast behavior
- [ ] Error responses for DB/circuit failures are HTTP 503 with body {"error":"Service temporarily unavailable, please try again later"} and no internal details
- [ ] Retry-After header present on 503 responses (configurable)
- [ ] Tests validate circuit transition (OPEN -> HALF_OPEN -> CLOSED) where applicable

## Implementation Checklist
- [ ] Create DBUnavailableError class (src/errors/DBUnavailableError.ts) and unit tests for error shape
- [ ] Implement lightweight circuit-breaker (src/lib/circuitBreaker.ts) with config: failureThreshold, resetTimeoutMs, windowSize
- [ ] Update userRepository to catch DB client errors and throw DBUnavailableError
- [ ] Wrap userRepository calls in authService with circuitBreaker.execute(...) and emit metrics on errors
- [ ] Update global errorHandler to map DBUnavailableError and circuit-open to HTTP 503 with generic message and Retry-After header
- [ ] Add/modify metrics (auth.service_unavailable, auth.circuit_open) and emit in error paths
- [ ] Add tests: tests/auth/db-unavailable.test.ts and tests/circuit/circuit-breaker.test.ts to simulate DB failures, assert 503, and verify circuit behavior
- [ ] Run test suite