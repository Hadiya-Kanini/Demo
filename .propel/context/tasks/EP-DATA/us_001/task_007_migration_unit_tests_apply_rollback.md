# Task - task_007

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
| Backend | Node.js (test runner and tooling) | 18.x or above |
| Database | PostgreSQL | 12.x / 13.x / 14.x (preferred >=12) |
| Library | node-postgres (pg) | 8.x or above |
| Library | migration runner (project standard, e.g., node-pg-migrate or knex) | v6+ (align with repo) |
| Library | Test Runner (Jest) | 29.x or above |
| Library | TypeScript (if project uses) | 5.x or above / or plain JS |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All test code and libraries MUST be compatible with the versions above. Adjust migration-runner-specific commands to match project conventions.

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
Create automated migration unit tests (apply/rollback + schema checks) that:
- Run the us_001 migration up against a throwaway test PostgreSQL database and assert the users table exists with exact columns, types, nullability, defaults.
- Verify presence of UNIQUE index on email_normalized and that duplicate-normalized inserts raise unique-violation (SQLSTATE 23505).
- Verify existence of CHECK/ENUM constraint for allowed hash_algo values (or other domain-level enforcement).
- Verify failed_attempts default and CHECK constraint failed_attempts >= 0 and test concurrent atomic increments do not lose updates.
- Verify index on reset_token_hash exists (and partial index metadata if migration includes it) and that reset_token_expiry insertion semantics (future timestamp) are enforced by test logic.
- Rollback the migration (down) and assert teardown (table dropped).

These tests are integration-level migration unit tests intended to run in CI against an ephemeral PostgreSQL instance (Docker or test DB) and should be runnable locally.

## Dependent Tasks
- Artefacts/Tasks:
  - Task: Migration file for us_001 must exist (create users table up + down). (e.g., migrations/2026xxxx_us_001_create_users_table.sql or equivalent JS migration). This task must be completed before these tests run.
  - Task: Test DB provisioning (Docker compose / CI job that provides a Postgres instance with credentials). If a test DB helper already exists, it must be available.

## Impacted Components
- New/Updated test files under test/ or tests/
- Test DB helper module for applying and rolling back migrations
- CI pipeline test step (optional: to include migration tests)
- No production code changes expected

Concretely impacted modules:
- tests/db/us_001_migration.test.(ts|js)
- tests/db/test-db.ts (helper)
- package.json (test script) — MODIFY if test scripts need registration (only if project lacks an entry; list as MODIFY)

## Implementation Plan
- Create a test DB helper that:
  - connects to a configurable Postgres test database using node-postgres (pg),
  - can run SQL files or migration runner commands to apply up/down migrations,
  - exposes utility functions to query pg_catalog and information_schema for columns, constraints, indexes,
  - runs raw SQL to insert/test constraints and performs concurrent operations via multiple client connections.
- Implement tests (Jest) that:
  - Setup: ensure clean test DB, apply migration up.
  - Schema assertions: query information_schema/pg_catalog to verify column types, nullability, defaults, presence of created_at/updated_at defaults, roles type (jsonb or text[]), and presence of UUID PK.
  - Index & constraint assertions: check unique index on email_normalized; check constraint or enum for hash_algo (via pg_type/pg_constraint); check CHECK constraint failed_attempts >= 0.
  - Functional assertions:
    - Attempt insert of duplicate email_normalized and assert error SQLSTATE = '23505'.
    - Concurrent increments: create a user row, spawn N parallel clients that run UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts, then assert final failed_attempts = initial + N.
    - reset_token_hash index: assert index exists, optionally run EXPLAIN ANALYZE on a SELECT by reset_token_hash seeded data (seed a sizeable number of rows and verify index used).
    - hash_algo enforcement: try inserting with allowed value(s) (e.g., 'argon2', 'bcrypt') and a disallowed value (e.g., 'unknown_algo') and assert DB rejects disallowed value (CHECK violation SQLSTATE 23514) OR if no DB-level constraint present, the test must assert that migration documents allowed values (fail test intentionally if neither DB nor documentation exists).
  - Teardown: rollback migration down and assert users table no longer exists.
- Keep tests deterministic and idempotent; ensure cleanup on failure.
- Keep total effort ≤ 8 hours by implementing core assertions (columns, uniqueness, failed_attempts concurrency, reset_token_hash index, hash_algo constraint) — skip deep EXPLAIN_ANLYZE perf if timeboxed.

## Current Project State
- Placeholder project structure (update when dependent tasks complete):
  - app/ (production app code)
  - migrations/ (existing migration scripts; must include us_001 migration)
  - tests/ or test/ (existing test suites)
  - scripts/ (helper scripts)
  - .propel/context/tasks/EP-DATA/us_001/us_001.md

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/db/test-db.ts | Test DB helper: connect to Postgres, run SQL, run migration up/down, query pg_catalog/information_schema, run concurrent updates helper. |
| CREATE | tests/db/us_001_migration.test.ts | Jest test suite implementing apply/rollback and schema/constraint/functional checks described in plan. |
| CREATE | tests/db/seed_reset_tokens.sql | SQL seed file used by tests to create many user rows with reset_token_hash values for index lookup verification. |
| MODIFY | package.json | (if necessary) add "test:migrations" script mapping to jest with appropriate env (e.g., "test:migrations": "NODE_ENV=test jest tests/db --runInBand"). If the repo already has an equivalent, adjust accordingly. |
| CREATE | .env.test.example | Example env with TEST_DB_URL or POSTGRES_* variables used by tests (to be copied to CI / local .env.test). |

## External References
- Postgres documentation: information_schema & pg_catalog (columns, constraints, indexes) — https://www.postgresql.org/docs/current/catalogs.html
- SQLSTATE codes: https://www.postgresql.org/docs/current/errcodes-appendix.html
- Example concurrent DB update pattern: UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts
- Node-postgres (pg) docs: https://node-postgres.com/
- Jest docs: https://jestjs.io/docs/getting-started

## Build Commands
- Reference for running tests against test DB (adjust to project):
  - Install deps: npm ci
  - Run migration tests: npm run test:migrations
- See project build commands: ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass (local)
- [ ] Integration tests pass against ephemeral Postgres (Docker/CI)
- [ ] **[UI Tasks]** Visual comparison against wireframe completed at 375px, 768px, 1440px — N/A
- [ ] **[UI Tasks]** Run `/analyze-ux` to validate wireframe alignment — N/A
- [ ] **[AI Tasks]** Prompt templates validated with test inputs — N/A
- [ ] **[AI Tasks]** Guardrails tested for input sanitization and output validation — N/A
- [ ] **[AI Tasks]** Fallback logic tested with low-confidence/error scenarios — N/A
- [ ] **[AI Tasks]** Token budget enforcement verified — N/A
- [ ] **[AI Tasks]** Audit logging verified (no PII in logs) — N/A

## Implementation Checklist
- [ ] Create tests/db/test-db.ts helper (connect, run SQL, migrations, query helpers)
- [ ] Implement tests/db/us_001_migration.test.ts with the schema, constraint, functional, concurrency and rollback tests
- [ ] Add seed file tests/db/seed_reset_tokens.sql and use it from the test suite for index lookup verification
- [ ] Ensure package.json has a test:migrations script and document required environment variables in .env.test.example
- [ ] Run tests locally against a Postgres instance (Docker) and iterate until deterministic and stable
- [ ] Ensure tests clean up (rollback down) on success or failure
- [ ] Commit files and update CI to run "test:migrations" in pipeline (CI change may be separate task)

Notes:
- Effort estimate: <= 8 hours
- Tests assume the us_001 migration file exists and follows repository migration conventions (migrations/...). If migration runner is project-specific (not SQL file), adapt test helper to call the runner CLI or JS API.

## RULES:
- Implementation Checklist contains 7 items (<=8)
- Effort: <=8 hours
- Acceptance Criteria were copied from the parent User Story us_001 and used to drive test assertions.