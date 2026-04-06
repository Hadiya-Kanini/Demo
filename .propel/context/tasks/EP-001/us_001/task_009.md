## Task ID: task_009
## Task Title: Add unit and component tests for login flow
## Category: Testing

## Description
Create a focused test suite for the authentication/login flow. Add unit tests for input validation, password verification logic, and token issuance. Add component-level tests for the AuthService/AuthController that mock the database, secret management, and rate/lockout behavior. Tests must cover: successful login, invalid credentials, malformed input, account/IP lockout and DB-unavailable error paths. Use existing project test frameworks (Jest + Supertest for backend; Cypress for E2E if present) and dependency-injection-friendly mocks for secrets and repositories so tests are deterministic and safe for CI.

## Acceptance Criteria
- Unit tests (Jest) added for:
  - input validation function(s): pass/fail cases for valid payload, missing fields, invalid email format.
  - password verification: correct handling of bcrypt/argon2 compare, legacy-hash branch (if applicable), and incorrect password.
  - token issuance: JWT payload claims, expiry equals configured TTL, and uses configured secret (mocked).
- Component tests (Jest or Jest+Supertest) added for AuthService/AuthController that mock:
  - User repository / DB (returns user, or not found, or DB error).
  - Secrets manager / env (JWT secret, hashing params).
  - RateLimiter/Lockout store (simulate exceeded thresholds).
  Cover success, invalid credentials (401), malformed input (400), lockout (429 or 423 per policy), and DB unavailable (503).
- Integration tests (Supertest) validate HTTP contract for POST /api/auth/login:
  - 200 + response body { access_token, token_type: "Bearer", expires_in } on success.
  - 401 + generic message "Invalid credentials" for bad credentials.
  - 400 for malformed requests with field-specific validation messages.
  - 429/423 when lockout triggered.
  - 503 when DB is unavailable.
- Tests assert that audit logging is invoked on successful login with user id, timestamp, client IP, and user-agent (using mock/spied AuditLogger).
- Token TTL asserted equals configured TTL (use deterministic clock or mock Date).
- All new tests run in CI and pass. Test suite is deterministic (no network calls, no real secrets).
- Tests added do not exceed effort estimate and files follow repository testing conventions and naming patterns.

## Technical Specifications

- APIs/Endpoints
  - POST /api/auth/login
    - Request JSON: { "email": string, "password": string }
    - Success response: 200 { access_token: string, token_type: "Bearer", expires_in: number }
    - Error responses:
      - 400 for validation errors (response includes fields)
      - 401 for invalid credentials ("Invalid credentials")
      - 429 or 423 for lockout / throttle
      - 503 for DB unavailable

- Components/Classes (backend)
  - controllers/AuthController (handles HTTP request/response)
  - services/AuthService (orchestrates validation, repo lookup, password check, tokens, audit)
  - services/TokenService (JWT creation; reads TTL and secret from config)
  - repositories/UserRepository (findByEmail, incrementFailedLogin, resetFailedLogin)
  - services/PasswordVerifier (wraps bcrypt/argon2 compare and legacy handling)
  - services/RateLimiterService or LockoutStore (tracks failed attempts)
  - utils/validator (schema validation for login request)
  - services/AuditLogger (records login events)
  - config/index (exposes AUTH_JWT_TTL, AUTH_JWT_SECRET, HASH_ALGO_VERSION)

- Data Models
  - User (id: UUID, email: string, password_hash: string, hash_version?: string, is_locked?: boolean, failed_attempts?: number)
  - LoginRequest { email: string, password: string }
  - TokenResponse { access_token: string, token_type: "Bearer", expires_in: number }
  - FailedLoginRecord { user_id?: UUID, ip: string, attempts: number, locked_until?: datetime }

## Requirement Reference
-