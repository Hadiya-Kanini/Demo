# Task - task_005

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:  
    - "Given supported hash algorithms configured in the application, When a row is written or updated with a hash_algo value, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to the allowed set (document the allowed list in migration or application config) and password_hash column size/format supports storing modern algorithm outputs and legacy values; unit tests verify permitted and rejected hash_algo values."
    - "Given a reset token generation flow, When the application stores reset_token_hash and reset_token_expiry, Then there exists an index on reset_token_hash to allow efficient lookup and the application enforces reset_token_expiry to be a future timestamp at time of insertion; tests verify lookup performance on seeded token records and expiry enforcement in application logic."
    - "Given hash algorithm policy, When creating or updating records, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to the allowed set (document the allowed list in migration or application config) and password_hash column size/format supports storing modern algorithm outputs and legacy values; unit tests verify permitted and rejected hash_algo values."
- Edge Case:
    - "What happens when a legacy password hash format is encountered? - The schema allows storing legacy hash formats (hash_algo captures legacy type; salt field is nullable). The application must detect legacy hash_algo during authentication and perform migration-on-login (rehash into modern algorithm) in a separate story. The DB will accept the legacy record; migration is handled at application layer, not via destructive DB migration."
    - "How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE ... SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and use row-level locking (SELECT ... FOR UPDATE) where necessary to prevent lost updates. Tests must validate concurrent increments and lockout threshold behavior."
    - "What happens when a generated reset token collides (unlikely) or a race creates multiple identical hashed tokens? - The DB provides an index on reset_token_hash but does not rely on uniqueness alone. The application must handle collisions by regenerating the token and retrying up to a small retry limit; expired tokens should be periodically pruned. If a uniqueness constraint is required, implement a partial unique index only for active (non-expired) tokens."

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
| Backend | Node.js + TypeScript | Node 18.x, TS 5.x |
| Database | PostgreSQL | 16.x |
| Library | Prisma (ORM) / node-postgres | Prisma v5 / pg v8 |
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
Add application-side configuration for allowed hash algorithms, implement model-level validation to reject unknown hash_algo values, enforce that reset_token_expiry is strictly in the future at creation time, add structured logging for validation violations, and create unit tests that assert correct acceptance/rejection behavior and proper logging for violations. This task does not change DB migrations (assumes the users table exists with columns described in us_001) but implements the application-layer safeguards and tests required by the Acceptance Criteria when DB-level CHECK/ENUM is not used.

## Dependent Tasks
- Artefacts/Tasks/task_001_db_create_users_table.md (migration that creates users table per us_001) — users table must exist before model-level validation is relied upon in integration tests.
- Setup of project test harness (e.g., jest/mocha + test DB) — ensure test DB and migration runner are available to run unit/integration tests.

## Impacted Components
- server/config/allowedHashAlgos.ts (new)
- server/models/user.ts (modify: add validation)
- server/services/userService.ts (modify: enforce reset_token_expiry > now() at create)
- server/errors/validationError.ts (create if not present)
- server/tests/user.hashAlgo.spec.ts (new)
- server/tests/user.resetTokenExpiry.spec.ts (new)
- server/logging/logger.ts (modify: ensure structured logging for validation failures)
- server/utils/time.ts (create/modify: helper for now() in tests to allow deterministic checks)

## Implementation Plan
- Add a canonical allowed-hash-algos config file listing supported algorithms and exported helper functions (isAllowedHashAlgo).
- Add/extend a ValidationError class to capture violation type and metadata for logging and easier assertions in tests.
- Modify the user model/entity constructor or ORM hooks (Prisma middleware or model-level validation file) to validate hash_algo against config on create/update and throw ValidationError on violation.
- Update userService.createUser (or equivalent) to validate reset_token_expiry is > now() when a reset token is being stored on creation; if invalid, throw ValidationError and log violation with metadata.
- Add structured logging calls for all validation failures (include user id/email if present, attempted value, allowed values, timestamp, stack trace).
- Implement unit tests:
  - user.hashAlgo.spec.ts: positive cases for allowed algos and negative cases for unknown algos; assert ValidationError thrown and a log entry created.
  - user.resetTokenExpiry.spec.ts: insertion with expiry in past should be rejected; with future expiry accepted; also test null expiry allowed.
- Ensure tests run against an in-memory or test DB (use prisma testing utility or mock ORM if full DB is not available) and include deterministic now() via server/utils/time.ts mocking.

## Current Project State
- app/ (frontend) — N/A for this task
- server/
  - config/
    - existing config files...
  - models/
    - user.ts (may exist; to be modified)
  - services/
    - userService.ts (may exist; to be modified)
  - logging/
    - logger.ts (may exist; to be modified)
  - tests/
    - existing unit/integration tests...
  - prisma/
    - schema.prisma (migration artifacts)
  - package.json
  - tsconfig.json

(Keep this structure as a placeholder; update during execution as files are created/modified.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | server/config/allowedHashAlgos.ts | Exports the canonical allowed hash algorithm identifiers and helper functions (isAllowedHashAlgo, getAllowedHashAlgos). |
| MODIFY | server/models/user.ts | Add model-level validation (Prisma middleware or pre-save hook) to validate hash_algo against allowed list and throw ValidationError on unknown values. |
| MODIFY | server/services/userService.ts | Enforce reset_token_expiry > now() at user creation/update when a reset token is provided; log and throw ValidationError on violations. |
| CREATE | server/errors/validationError.ts | Define a ValidationError class extending Error with metadata (field, value, allowedValues). |
| MODIFY | server/logging/logger.ts | Add structured logging for validation violations (error level, include metadata). |
| CREATE | server/tests/user.hashAlgo.spec.ts | Unit tests validating acceptance and rejection of hash_algo values and asserting logs/ValidationError. |
| CREATE | server/tests/user.resetTokenExpiry.spec.ts | Unit tests validating reset_token_expiry enforcement and logs on violation. |
| CREATE | server/utils/time.ts | Export now() wrapper to allow deterministic mocking in tests. |

## External References
- PostgreSQL partial index docs: https://www.postgresql.org/docs/current/indexes-partial.html
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- Prisma middleware docs (for model-level validation): https://www.prisma.io/docs/concepts/components/prisma-client/middleware
- Node.js logging best practices (structured logging): https://www.freecodecamp.org/news/logging-in-node-js-with-pino/

## Build Commands
- Install deps: npm ci
- Build: npm run build
- Run tests: npm test
- Run migration (dev): npx prisma migrate dev --name apply-us_001 (if running full integration tests)
- Run linter: npm run lint
(See ../.propel/build/ for project-level build orchestration.)

## Implementation Validation Strategy
- [ ] Unit tests pass (user.hashAlgo.spec.ts, user.resetTokenExpiry.spec.ts)
- [ ] Integration tests pass if configured against test DB (optional)
- [ ] Logging verified: validation failures produce structured log entries (asserted in tests via logger mock)
- [ ] Validation behavior: unknown hash_algo is rejected at model/service layer and accepted values persist
- [ ] reset_token_expiry enforcement: past timestamps are rejected; future timestamps accepted; null allowed
- [ ] Tests cover legacy-hash acceptance (i.e., known legacy algorithm listed in allowed list)
- [ ] Review by backend/security lead to confirm allowed algorithms list

## Implementation Checklist
- [ ] Create server/config/allowedHashAlgos.ts with canonical allowed algorithms and exported isAllowedHashAlgo()
- [ ] Implement ValidationError class at server/errors/validationError.ts
- [ ] Add model-level validation for hash_algo in server/models/user.ts (Prisma middleware or model hook)
- [ ] Enforce reset_token_expiry > now() in server/services/userService.ts and throw/log ValidationError on violation
- [ ] Add structured logging calls in server/logging/logger.ts for validation failures
- [ ] Implement unit tests: server/tests/user.hashAlgo.spec.ts and server/tests/user.resetTokenExpiry.spec.ts
- [ ] Run tests and fix issues; ensure all tests complete within CI test runner

Notes:
- Reference server/config/allowedHashAlgos.ts from model/service code (MANDATORY).
- Effort estimate for this task: <= 8 hours.