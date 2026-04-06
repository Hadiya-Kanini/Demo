# Task - [TASK_004]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a registered user with valid credentials, When the client sends a POST /auth/login with a JSON payload, Then the server responds with HTTP 200 and a JSON body containing:
      {
        "access_token": "<JWT>",
        "token_type": "Bearer",
        "expires_in": 3600
      }
      - The token is signed with the system's configured signing key, contains standard claims (sub, iat, exp, roles), and expires after the configured TTL.
      - The response body contains no stack traces or internal error details.
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
| Backend | Node.js + TypeScript (recommended) | 18.x / TypeScript 5.x |
| Database | PostgreSQL (used elsewhere) | 16.x |
| Library | jsonwebtoken (or jose) | jsonwebtoken v9.x (or jose v4.x) |
| Library | dotenv / config loader | latest |
| Library | jest (unit tests) | 29.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the versions above.

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
Implement TokenService.createAccessToken(user) that builds and signs JWT access tokens using project configuration (signing key, algorithm, TTL). The service will return the signed token and a metadata object suitable for login responses: token_type ("Bearer") and expires_in (seconds). Tokens must include standard claims: sub (user id), iat, exp, and roles (array). The service must determine expiry from configured TTL, calculate iat/exp in seconds since epoch, and not leak internal errors. Unit tests will validate claims, TTL, and returned structure.

## Dependent Tasks
- .propel/context/tasks/EP-001/us_001/task_001_be_login_route_controller.md (Login controller/route) — TokenService used by controller
- US_010 (User data & password storage/hash) — user object shape and roles must be available
- US_011 (Token storage & revocation) — if token revocation/persistence required, ensure TokenService integrates (this task assumes stateless issuance; revocation is handled separately)
- config/secrets provisioning (private key / secret must be available in environment/secrets manager)

## Impacted Components
- New/Updated:
  - src/services/TokenService.ts (NEW) — core implementation
  - src/config/jwt.ts (MODIFY/CREATE) — expose jwt config (ttl, algorithm, privateKey/secret)
  - src/types/auth.ts (MODIFY/CREATE) — TokenResponse type
  - src/di/container.ts (MODIFY) — register TokenService for DI (if DI is used)
  - tests/unit/tokenService.spec.ts (NEW) — unit tests
- Indirect:
  - src/controllers/AuthController.ts (READ/USE) — will call TokenService (controller modification done in dependent task_001)

## Implementation Plan
- Add configuration contract for JWT: TTL in seconds (JWT_TTL_SECONDS), algorithm (JWT_ALG), and signing secret/private key (JWT_SECRET or JWT_PRIVATE_KEY).
- Implement TokenService.createAccessToken(user: User, opts?: { now?: number }) -> { token: string, token_type: 'Bearer', expires_in: number }:
  - Validate required user properties: id (sub) and roles (array|string).
  - Compute iat = now || Math.floor(Date.now() / 1000).
  - Compute exp = iat + TTL.
  - Build payload claims: sub (string), iat, exp, roles (array).
  - Use jsonwebtoken.sign(payload, key, { algorithm, noTimestamp: true }) — noTimestamp true to use our iat/exp.
  - Return token and metadata: { access_token: token, token_type: 'Bearer', expires_in: TTL } (or { token, token_type, expires_in } as internal return).
- Handle errors securely:
  - Wrap sign errors and throw a generic error for callers; do not include internal stack traces in responses.
  - If signing key missing, throw an implementation error that will be mapped to 503 by controllers/handlers.
- Unit tests:
  - Verify token contains sub, iat, exp, roles.
  - Verify exp - iat === TTL.
  - Verify returned token_type and expires_in values.
  - Use a deterministic "now" override in tests to assert numeric values.
- Update DI/container (or export default instance) so AuthController (dependent task) can acquire TokenService.
- Add docs/comments referencing security considerations and required env vars.

## Current Project State
- Placeholder project structure (update as real project files exist). Focus under 'src' and 'tests':
  - src/
    - controllers/
      - AuthController.ts (exists or will be created by dependent task)
    - services/
      - (new) TokenService.ts
    - config/
      - index.ts (may exist)
    - di/
      - container.ts (may exist)
    - types/
      - user.ts (may exist)
  - tests/
    - unit/
      - (new) tokenService.spec.ts

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/services/TokenService.ts | TokenService implementation with createAccessToken(user) that builds, signs JWTs, and returns token + metadata |
| CREATE | src/types/auth.ts | TokenResponse and TokenPayload type definitions used by TokenService and controllers |
| CREATE | tests/unit/tokenService.spec.ts | Unit tests for TokenService.createAccessToken verifying claims, TTL and return shape |
| MODIFY | src/config/jwt.ts | Add/read JWT config (TTL, algorithm, secret/key) from environment/config provider |
| MODIFY | src/di/container.ts | Register TokenService so it can be injected/required by controllers (if project uses DI) |
| MODIFY | package.json | Add test script entry if missing (e.g., "test": "jest --runInBand") |

## External References
- JWT RFC 7519: https://datatracker.ietf.org/doc/html/rfc7519
- jsonwebtoken (npm): https://www.npmjs.com/package/jsonwebtoken
- jose (alternative): https://github.com/panva/jose
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html

## Build Commands
- Install & build:
  - npm install
  - npm run build
- Run tests:
  - npm test
- Note: refer to repository build commands in ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass (tests/unit/tokenService.spec.ts)
- [ ] Integration tests pass when run with AuthController (dependent task)
- [ ] Token payload contains sub, iat, exp, roles
- [ ] Token expiry equals configured TTL (exp - iat === TTL)
- [ ] Response shape matches acceptance criteria: access_token, token_type: "Bearer", expires_in (seconds)
- [ ] Signing key missing or misconfiguration results in a controlled error mapped to 503 (no internals leaked)

## Implementation Checklist
- [ ] Add JWT configuration module: read TTL (seconds), algorithm, and signing key from env/config.
- [ ] Implement src/services/TokenService.ts with createAccessToken(user) per plan and security handling.
- [ ] Create src/types/auth.ts with TokenResponse and TokenPayload definitions.
- [ ] Add unit tests in tests/unit/tokenService.spec.ts that assert claims and TTL using deterministic now override.
- [ ] Register TokenService in DI/container (or export factory) so controllers can use it.
- [ ] Update package.json test script if needed and run tests locally.
- [ ] Document required env vars in README or config file (JWT_TTL_SECONDS, JWT_ALG, JWT_SECRET or JWT_PRIVATE_KEY).

Effort estimate: 4 hours

## RULES:
- Implementation Checklist has <=8 items
- Effort <=8 hours

