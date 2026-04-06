# Task - task_008

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-DATA/us_001/us_001.md
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is executed, Then a users table is created with these columns and types: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL DEFAULT ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the users table exists, When an insert is attempted with an email whose normalized form duplicates an existing normalized email, Then the DB rejects the insert with a unique-violation error due to a UNIQUE index on email_normalized and demonstration tests assert the expected error code.
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given a reset token generation flow, When the application stores reset_token_hash and reset_token_expiry, Then there exists an index on reset_token_hash to allow efficient lookup and the application enforces reset_token_expiry to be a future timestamp at time of insertion; tests verify lookup performance on seeded token records and expiry enforcement in application logic.
    - Given hash algorithm policy, When creating or updating records, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to the allowed set (document the allowed list in migration or application config) and password_hash column size/format supports storing modern algorithm outputs and legacy values; unit tests verify permitted and rejected hash_algo values.
- Edge Case:
    - What happens when a legacy password hash format is encountered? - The schema allows storing legacy hash formats (hash_algo captures legacy type; salt field is nullable). The application must detect legacy hash_algo during authentication and perform migration-on-login (rehash into modern algorithm) in a separate story. The DB will accept the legacy record; migration is handled at application layer, not via destructive DB migration.
    - How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and use row-level locking (SELECT ... FOR UPDATE) where necessary to prevent lost updates. Tests must validate concurrent increments and lockout threshold behavior.
    - What happens when a generated reset token collides (unlikely) or a race creates multiple identical hashed tokens? - The DB provides an index on reset_token_hash but does not rely on uniqueness alone. The application must handle collisions by regenerating the token and retrying up to a small retry limit; expired tokens should be periodically pruned. If a uniqueness constraint is required, implement a partial unique index only for active (non-expired) tokens.

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
| Backend | Node.js | 18.x |
| Database | PostgreSQL | 14.x |
| Library | node-postgres (pg) | v8.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code, and libraries, MUST be compatible with versions above.

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
Implement automated integration tests that validate database constraint behavior specified in us_001: verify UNIQUE constraint on email_normalized (duplicate rejection with expected SQL state), CHECK constraint preventing negative failed_attempts, and acceptance/rejection of allowed/disallowed hash_algo values (enum/check). Tests will run against a disposable PostgreSQL test database where the us_001 migration has been applied. Tests should assert precise SQLSTATE codes for constraint violations and confirm successful inserts/updates for allowed values.

## Dependent Tasks
- Artefacts/Tasks:
  - Task 001: Create migration for us_001 (migration file that creates users table with constraints and indexes). Path expected: Server/db/migrations/us_001_create_users_table.sql (or project-equivalent). This migration MUST be applied before running these tests.
  - Task 000: Confirm DB platform & migration tooling (ensure test DB, extensions)
  - CI step that can spin up a PostgreSQL instance for tests (Docker-based test DB)

## Impacted Components
- Server/tests/db/constraints.test.js (new)
- Server/tests/db/test-helpers.js (new)
- Server/db/migrations/us_001_create_users_table.sql (dependency — must exist)
- package.json (MODIFY) - add test script and test dependencies
- .github/workflows/test.yml or CI job (MODIFY) - ensure DB for tests (optional; note as impacted)

## Implementation Plan
- Create test helper to:
  - connect to PostgreSQL test DB using connection URL from env (TEST_DATABASE_URL)
  - run up/down migration for us_001 (invoke project's migration runner or execute SQL directly)
  - provide convenience functions: insertUser(payload), clearUsersTable()
- Write Jest-based integration tests (Server/tests/db/constraints.test.js) that:
  - beforeAll: create test DB schema by applying us_001 migration
  - afterAll: drop schema or rollback migration
  - Test 1: UNIQUE email_normalized
    - Insert a user with email_normalized X — expect success
    - Insert a second user with same email_normalized — expect error with SQLSTATE '23505' (unique_violation)
  - Test 2: CHECK failed_attempts >= 0
    - Attempt to insert user with failed_attempts = -1 — expect SQLSTATE '23514' (check_violation)
    - Insert user with default/0 and attempt UPDATE users SET failed_attempts = -1 WHERE id=... — expect check_violation
  - Test 3: hash_algo allowed/disallowed
    - Define allowed set consistent with migration (e.g., ['argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256'])
    - Insert user rows for each allowed value — expect success
    - Attempt insert with 'unknown_algo' — expect either check_violation (23514) or user-defined enum error (if enum used) with SQLSTATE '22P02' or appropriate; assert that failing operation does not succeed and assert the SQLSTATE is a constraint violation (prefer checking for 23514 or enum-specific)
- Make tests resilient: catch and assert SQLSTATE from error.code, and fail if insertion unexpectedly succeeds
- Add package.json scripts and test dependencies (jest, pg) and instructions in README/test instructions

## Current Project State
- app/ (frontend)  
- Server/  
  - db/  
    - migrations/  
      - (expected) us_001_create_users_table.sql  <-- migration dependency (not created by this task)  
  - src/  
  - tests/  
    - (empty or existing)  
- .propel/context/tasks/EP-DATA/us_001/us_001.md

(Note: the above is a placeholder project snapshot; actual repo layout may vary. The migration file must exist per Dependent Tasks before running tests.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | Server/tests/db/test-helpers.js | Test helper to manage DB connection, apply/rollback migration (or execute migration SQL), and helper functions for test operations (insertUser, clearUsersTable). |
| CREATE | Server/tests/db/constraints.test.js | Jest integration tests covering UNIQUE email_normalized, CHECK failed_attempts >= 0, and allowed/rejected hash_algo values. |
| MODIFY | package.json | Add test script ("test:db") and devDependencies (jest, pg, dotenv) or update existing test script to include DB setup. |
| MODIFY | .env.example | Add TEST_DATABASE_URL example for local test runs (if present in repo). |
| MODIFY | .github/workflows/test.yml | (Optional) Add or update CI job to spin up PostgreSQL Docker service for DB tests. (If repo uses CI; if not present, this line may be omitted.) |

## External References
- PostgreSQL: Constraints and Error Codes — https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL error codes: https://www.postgresql.org/docs/current/errcodes-appendix.html (23505 unique_violation, 23514 check_violation)
- node-postgres (pg) documentation: https://node-postgres.com/
- Jest docs: https://jestjs.io/docs/getting-started
- Example migration patterns for Postgres (partial index, enum type): https://www.postgresql.org/docs/current/sql-createenum.html

## Build Commands
- Install dependencies: npm ci
- Run tests locally (after providing TEST_DATABASE_URL env): npm run test:db
- Sample commands (project-specific):
  - npm install --save-dev jest pg dotenv
  - TEST_DATABASE_URL=postgres://user:pass@localhost:5432/test_db npm run test:db
- See project build metadata: ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass
- [ ] Integration tests pass (DB integration)
- [ ] **[UI Tasks]** Visual comparison against wireframe completed at 375px, 768px, 1440px
- [ ] **[UI Tasks]** Run `/analyze-ux` to validate wireframe alignment
- [ ] **[AI Tasks]** Prompt templates validated with test inputs
- [ ] **[AI Tasks]** Guardrails tested for input sanitization and output validation
- [ ] **[AI Tasks]** Fallback logic tested with low-confidence/error scenarios
- [ ] **[AI Tasks]** Token budget enforcement verified
- [ ] **[AI Tasks]** Audit logging verified (no PII in logs)

Notes:
- UI/AI specific checks are N/A for this task; they remain in the checklist template but are not applicable.

## Implementation Checklist
- [ ] Create Server/tests/db/test-helpers.js to manage DB connection, apply/rollback migration, and provide insertUser/clear functions (<=2 hours)
- [ ] Create Server/tests/db/constraints.test.js with Jest tests for UNIQUE, CHECK, and hash_algo acceptance/rejection (<=3 hours)
- [ ] Add/modify package.json to include "test:db" script and required devDependencies (<=0.5 hours)
- [ ] Run tests locally against a disposable Postgres instance and iterate on assertions for SQLSTATE codes (<=1.5 hours)
- [ ] Update .env.example with TEST_DATABASE_URL guidance and document test run steps in README or tests/README.md (<=0.5 hours)

Estimated effort: 7.5 hours

- **Note:** These tests assume the us_001 migration file (Server/db/migrations/us_001_create_users_table.sql) is present and creates enum/check/unique constraints as described. If migration uses an application migration runner, test-helpers must invoke it (adjust helper accordingly).

