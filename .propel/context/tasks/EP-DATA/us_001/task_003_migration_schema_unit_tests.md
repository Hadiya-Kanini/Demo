# Task - task_003

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
| Frontend | N/A | N/A |
| Backend | Node.js (test harness) | 18.x |
| Database | PostgreSQL | 14.x / 15.x |
| Library | node-postgres (pg) | v8.x |
| Library | Jest (test runner) | v29.x |
| Library | testcontainers (optional for CI) | v9.x (or N/A if not used) |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries referenced in tests MUST be compatible with the versions above.

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
Create a test suite that programmatically runs the us_001 migration (up), asserts the resulting schema objects (columns, types, defaults, constraints, indexes, enum/check for hash_algo) exist and have correct properties, verifies behavioral aspects (unique constraint on email_normalized causing unique-violation, atomic increments for failed_attempts under simulated concurrency), checks index existence for reset_token_hash (and partial index where supported), verifies accepted/rejected hash_algo values, and finally runs migration rollback (down) and asserts cleanup (table and related objects removed). Tests must be runnable locally and in CI given a PostgreSQL connection (preferably via testcontainers in CI).

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/task_001_db_create_users_table.md (migration script implementing us_001 must exist and be runnable by the project's migration runner)
- Migration runner setup task (e.g., project migration tooling configured: Knex/Umzug/Flyway) — if not present, add a prep task to make the runner available in CI.

## Impacted Components
- migrations/us_001_create_users_table.sql OR migrations/2023xxxx_us_001_create_users_table.(js|sql) (depending on migration tooling) — read by tests
- test/helpers/db.js (new test DB helper to connect, run migrations)
- tests/migrations/us_001_migration.test.js (new)
- package.json (modify to add test:migrations script) — optional
- CI pipeline config (optional) — recommend adding job to run migration tests (not included as file changes here unless CI modification task exists)

## Implementation Plan
- 1) Create a DB test helper that:
  - connects to PostgreSQL using env vars (TEST_DB_URL or derive from TEST_DB_HOST/PORT/USER/PASS/DB),
  - exposes methods to run migration up/down using the project's migration runner or by executing the migration SQL file directly if runner is unavailable.
- 2) Implement Jest test suite (tests/migrations/us_001_migration.test.js) with these tests:
  - Test A: Run migration up; assert table users exists.
  - Test B: Inspect columns and types via information_schema / pg_catalog and assert presence, nullability, defaults for created_at/updated_at, roles default, failed_attempts default.
  - Test C: Verify UNIQUE constraint on email_normalized by inserting two rows with same normalized email and asserting unique-violation SQLSTATE (23505).
  - Test D: Verify CHECK constraint failed_attempts >= 0 is present; attempt to insert negative failed_attempts and assert CHECK violation (SQLSTATE 23514) OR verify constraint definition in pg_constraint.
  - Test E: Simulate concurrent increments of failed_attempts: create a user, then in parallel run two UPDATE ... SET failed_attempts = failed_attempts + 1 operations (in separate DB connections/transactions) and assert final value increments by both attempts. Use transactions to simulate realistic concurrency and/or run updates in parallel promises.
  - Test F: Verify index on reset_token_hash exists; if partial indexes supported, check pg_index/pg_class/pg_constraint for predicate; seed token rows and ensure lookup by reset_token_hash returns expected row; also assert that storing reset_token_expiry <= now() is disallowed by application test (if DB cannot enforce, check test asserts application-level enforcement placeholder).
  - Test G: Verify hash_algo allowed values: attempt inserts with an allowed value and a disallowed value; expect disallowed insert to be rejected (check SQLSTATE 23514 if DB CHECK used, else assert application-level validation via separate test harness — document if DB-level constraint missing).
  - Test H: Run migration down (rollback); assert users table and associated types/indexes/constraints removed.
- 3) Add utility to run a single migration up/down by filename when migration runner is not present (e.g., execute SQL file directly in test).
- 4) Add package.json test script "test:migrations" that runs Jest with environment variable guidance.
- 5) Document test execution steps in README or test header comments (how to provide DB connection or run testcontainers-based PG).

## Current Project State
- app/ (placeholder)
- Server/ (placeholder)
- migrations/ (migration scripts folder — contains us_001 migration created by dependent task)
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story)
- package.json (project metadata)  
Note: The above is a placeholder snapshot. Ensure migrations/us_001_create_users_table.* exists before running tests.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/migrations/us_001_migration.test.js | Jest test suite that runs the us_001 migration up/down and contains assertions for columns, defaults, constraints, indexes, concurrency increment behavior, hash_algo validation, and rollback cleanup. |
| CREATE | tests/helpers/db.js | DB helper to connect to Postgres, run SQL files or invoke migration runner, run queries, and manage connections for concurrency simulation. |
| MODIFY | package.json | Add script "test:migrations": "TEST_DB_URL=... jest --config=jest.config.js tests/migrations/us_001_migration.test.js" and add devDependencies (jest, pg, testcontainers optional) in devDependencies. |
| CREATE | tests/jest.setup.js | (optional) Jest setup for environment variables and global DB teardown helpers used by migration tests. |

## External References
- PostgreSQL: information_schema and pg_catalog reference — https://www.postgresql.org/docs/current/catalogs.html
- PostgreSQL partial index docs — https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL CHECK constraint docs — https://www.postgresql.org/docs/current/ddl-constraints.html
- SQLSTATE error codes — https://www.postgresql.org/docs/current/errcodes-appendix.html
- testcontainers-node (if used) — https://www.testcontainers.org/modules/node/
- node-postgres (pg) — https://node-postgres.com/

## Build Commands
- npm install (to install devDependencies: jest, pg, testcontainers (optional))
- npm run test:migrations (script to run migration tests)
- Refer to project build commands in ../.propel/build/ for CI integration.

## Implementation Validation Strategy
- [ ] Unit tests pass (Jest suite for migration tests)
- [ ] Integration tests pass (tests run against PostgreSQL instance; concurrency simulations validated)
- [ ] **[UI Tasks]** Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A)
- [ ] **[UI Tasks]** Run /analyze-ux to validate wireframe alignment (N/A)
- [ ] **[AI Tasks]** Prompt templates validated with test inputs (N/A)
- [ ] **[AI Tasks]** Guardrails tested for input sanitization and output validation (N/A)
- [ ] **[AI Tasks]** Fallback logic tested with low-confidence/error scenarios (N/A)
- [ ] **[AI Tasks]** Token budget enforcement verified (N/A)
- [ ] **[AI Tasks]** Audit logging verified (N/A)

## Implementation Checklist
- [ ] Create tests/helpers/db.js to provide connection helpers and migration execution helpers.
- [ ] Implement tests/migrations/us_001_migration.test.js covering all Acceptance Criteria and Edge Cases (columns/types/defaults, unique constraint behavior, failed_attempts atomic increments & check constraint, reset_token_hash index & expiry behavior, hash_algo allowed list, migration rollback).
- [ ] Add/modify package.json script test:migrations to run the new Jest tests.
- [ ] Ensure tests can run against a local or ephemeral PostgreSQL instance (document use of TEST_DB_URL; optionally implement testcontainers to spin up Postgres in CI).
- [ ] Run test suite locally and iterate until all tests pass against the migration created by dependent task.
- [ ] Add brief README/test header documenting how to run the tests in CI and locally (env vars required).
- [ ] Mark task complete only after tests pass and CI job (if added) runs successfully.

Notes:
- Effort estimate for this task: <= 8 hours.
- Tests are scoped to verifying migration effects and schema-level behaviors; application-level validations not enforced by DB must be documented and tested at application layer in a follow-up story.
- If the migration implements hash_algo as an enum type, tests should query pg_type/pg_enum; if migration uses CHECK constraint instead, tests should verify constraint predicate accordingly.