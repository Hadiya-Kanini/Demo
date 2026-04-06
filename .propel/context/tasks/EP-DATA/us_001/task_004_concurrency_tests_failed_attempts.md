# Task - [TASK_004]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-DATA/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the us_001 migration is applied, Then the users table is created with columns: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL DEFAULT ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the users table exists, When an insert is attempted with an email whose normalized form duplicates an existing normalized email, Then the DB rejects the insert with a unique-violation error due to a UNIQUE index on email_normalized and demonstration tests assert the expected error code.
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given the new schema, When failed_attempts is updated by concurrent login attempts, Then the database enforces a CHECK constraint failed_attempts >= 0 so negative values cannot be persisted.
    - Given a reset token generation flow, When the application stores reset_token_hash and reset_token_expiry, Then there exists an index on reset_token_hash to allow efficient lookup and the application enforces reset_token_expiry to be a future timestamp at time of insertion; tests verify lookup/index existence and basic expiry semantics (presence of column and seeded record lookup).
    - Given hash algorithm policy, When creating or updating records, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to an allowed set (document the allowed list in migration or application config); tests verify presence of the column and, if DB-level constraint exists, that invalid values are rejected.
- Edge Case:
    - How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and use row-level locking (SELECT ... FOR UPDATE) where necessary to prevent lost updates. Tests must validate concurrent increments and lockout threshold behavior.
    - What happens when a legacy password hash format is encountered? - The schema allows storing legacy hash formats (hash_algo captures legacy type; salt field is nullable). (Note: this task focuses on migration and concurrency tests; legacy handling/re-hash-on-login is out-of-scope for implementation but acknowledged.)
    - What happens when reset token hash collisions or race conditions occur? - The DB provides an index to look up tokens but the application is responsible for token generation/retry. This task will verify the index existence and basic lookup; collision handling is application responsibility.

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
| Backend | Node.js | 18.x+ |
| Database | PostgreSQL | 16.x+ |
| Library | pg (node-postgres) | 8.x+ |
| Test Runner | Jest | 29.x+ |
| Migration Runner | N/A (migration task is dependent) | N/A |
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
Build an automated integration test harness that:
- Applies the us_001 migration to a transient PostgreSQL test instance, validates the created schema (columns, types, defaults, constraints, indexes), and then rolls the migration back to verify down/rollback behavior.
- Runs schema-level validations: presence of UNIQUE index on email_normalized, presence of index on reset_token_hash, presence of CHECK constraint failed_attempts >= 0 and presence/behavior of any hash_algo constraints if implemented in migration.
- Simulates concurrent operations which increment users.failed_attempts to verify atomicity of UPDATE ... SET failed_attempts = failed_attempts + 1 (no lost updates) and verifies behavior both with plain atomic UPDATEs and with explicit SELECT ... FOR UPDATE transactional flows.
- Verifies that attempting to set failed_attempts to a negative value is rejected by the DB (CHECK constraint violation).
- Runs against a transient PostgreSQL test instance (docker-compose) using multiple parallel DB connections to simulate concurrency; tests are repeatable and aim to be deterministic.

## Dependent Tasks
- Artefacts/Tasks:
  - task_001_db_create_users_table (the migration that creates the users table with required columns, UNIQUE index on email_normalized, CHECK constraint failed_attempts >= 0, index on reset_token_hash, and optional hash_algo constraint). This migration must be applied to the test DB before running these tests (tests will attempt to apply and roll back this migration as part of validation).
  - Setup of test database runner / docker-compose test DB environment (if not already present).

## Impacted Components
- Server/test/helpers/db.js (NEW) - DB helper to obtain pooled clients and run transactions
- Server/test/integration/users_migration_and_concurrency.test.js (NEW) - Jest integration test file implementing migration apply/rollback, schema validation, concurrent increment tests, FOR UPDATE transactional tests, and CHECK constraint negative-set test.
- Server/docker/test-postgres/docker-compose.yml (NEW) - local test postgres configuration for CI/local development
- Server/package.json (MODIFY) - add/ensure test script and test dependencies (jest, pg)
- Migrations artifacts (READ-ONLY for tests) - migrations/us_001_create_users_table.sql (DEPENDENT, must exist)

## Implementation Plan
- Create a lightweight DB helper (Server/test/helpers/db.js) that can:
  - create a pg Pool using node-postgres with configurable connection params via env vars
  - provide convenience functions: runQuery(sql, params), getClient(), withTransaction(fn) where fn receives a client and can commit/rollback automatically, and a runConcurrent(workers) helper to execute N promises concurrently returning results
  - expose a function to check for index/constraint existence via system catalogs
- Add docker-compose configuration to spin up a local PostgreSQL 16 instance for tests (configured with known user/db/port) and health-check settings.
- Write a single comprehensive integration test file (Server/test/integration/users_migration_and_concurrency.test.js) that:
  - Orchestrates bringing up the test DB (documented; script may start docker-compose externally), applies the migration SQL (via psql or migration runner if available), and verifies migration applied.
  - Validates schema: verifies columns and types exist (via information_schema or pg_catalog queries), verifies UNIQUE index on email_normalized exists, verifies index on reset_token_hash exists, verifies CHECK constraint failed_attempts >= 0 exists (query pg_constraint), and documents presence/absence of a hash_algo DB-level constraint (if exists assert it rejects invalid values).
  - Seeds one user row with failed_attempts = 0 (INSERT ... RETURNING id).
  - Runs N concurrent workers (e.g., 20) each performing the atomic UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ...; run them in parallel using separate pooled clients and assert after all complete that failed_attempts == N.
  - Repeats with explicit transactional SELECT ... FOR UPDATE read-modify-write flow to show locking prevents lost updates: each worker begins transaction, SELECT failed_attempts FOR UPDATE, compute +1 and UPDATE, commit. Assert final count.
  - Tests negative failed_attempts attempt: execute UPDATE users SET failed_attempts = -1 WHERE id = ... and assert DB rejects due to CHECK constraint (expect specific SQLSTATE or error text).
  - Tests UPDATE ... RETURNING failed_attempts values under concurrent atomic UPDATEs: collect returned values and assert they reflect increments in aggregate (e.g., distinct returned values count equals workers, and final max equals N). Note: due to concurrency, ordering may vary; tests should assert set-wise coverage rather than strict ordering to be robust.
  - Verifies migration rollback: run the migration DOWN (or DROP TABLE if migration runner not available), assert table no longer exists, then re-apply if needed for other tests.
- Modify Server/package.json to add devDependencies (jest, pg) and add an idempotent "test:integration" script that:
  - Brings up the docker-compose test DB (or instructs the user/CI how to start it)
  - Applies the migration (via psql or migration runner)
  - Runs Jest with the integration test file
  - Tears down DB (or leaves running depending on CI policy). Ensure environment variables for DB connection are consistent.
- Document how to run tests locally and in CI in Server/README.md (short section).
- Iterate tests locally to ensure stability and determinism; reduce flakiness (increase worker delays/retries if necessary) and set reasonable timeouts.

## Current Project State
- app/ (placeholder)
- Server/
  - package.json (may exist)
  - test/ (may exist)
  - migrations/ (dependent task creates users table migration)
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story file exists)

Note: The above is a placeholder snapshot. Actual tree may differ; tests should be written to not assume other test infra beyond docker-compose and node scripts.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | Server/test/helpers/db.js | DB helper that creates a pg Pool and provides runQuery, getClient, withTransaction, and runConcurrent helpers for tests. |
| CREATE | Server/test/integration/users_migration_and_concurrency.test.js | Jest integration test file implementing migration apply/rollback, schema validation, concurrent increment tests (atomic UPDATE), FOR UPDATE transactional tests, and CHECK constraint negative-set test. |
| CREATE | Server/docker/test-postgres/docker-compose.yml | docker-compose config to spin up a Postgres 16 container for integration tests (with known credentials and port mapping). |
| MODIFY | Server/package.json | Add devDependencies (jest, pg), add "test:integration" script that starts DB (or assumes started), runs migrations (or runs SQL), runs Jest integration tests, and optionally tears down. |
| MODIFY | Server/README.md | Add short section documenting how to run integration tests locally (start docker-compose, apply migrations, run npm script). |

> Only list concrete, verifiable file operations. No speculative directory trees.

## External References
- PostgreSQL UPDATE concurrency considerations: https://www.postgresql.org/docs/current/explicit-locking.html
- PostgreSQL CHECK constraint docs: https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS
- Using RETURNING with UPDATE in Postgres: https://www.postgresql.org/docs/current/sql-update.html
- node-postgres (pg) pooling & transactions: https://node-postgres.com/
- Example docker-compose Postgres setup: https://docs.docker.com/compose/compose-file/compose-file-v3/#services

## Build Commands
- Start test DB (from repo root):
  - cd Server/docker/test-postgres && docker-compose up -d
- Run migrations (migration runner must be available; otherwise run SQL manually):
  - (example) PGPASSWORD=test psql -h localhost -p 5433 -U test -d testdb -f ../../migrations/us_001_create_users_table.sql
- Run tests:
  - cd Server && npm run test:integration
- Teardown:
  - cd Server/docker/test-postgres && docker-compose down -v

(See Server/package.json test:integration script for automated orchestration.)

## Implementation Validation Strategy
- [ ] Unit tests pass (where applicable for helper functions)
- [ ] Integration tests pass (migration apply/rollback, schema validation, concurrent increment scenarios succeed)
- [ ] Verify final failed_attempts equals expected total after concurrent increments
- [ ] Verify transactional SELECT ... FOR UPDATE path prevents lost updates under simulated read-modify-write patterns
- [ ] Verify CHECK constraint prevents failed_attempts from becoming negative (attempt to set -1 yields constraint violation)
- [ ] Tests are repeatable locally using the provided docker-compose test DB

## Implementation Checklist
- [ ] Create Server/test/helpers/db.js: implement pg Pool, runQuery, getClient, withTransaction, and runConcurrent helpers.
- [ ] Create Server/test/integration/users_migration_and_concurrency.test.js: implement migration apply/rollback checks, schema validations (columns, indexes, constraints), concurrent UPDATE atomic increment test, FOR UPDATE transactional test, CHECK constraint negative-set test using Jest.
- [ ] Add Server/docker/test-postgres/docker-compose.yml to provision PostgreSQL 16 for tests.
- [ ] Modify Server/package.json to add devDependencies (jest, pg) and test:integration script that orchestrates DB and runs Jest integration tests.
- [ ] Update Server/README.md with short instructions to run integration tests locally and CI notes.
- [ ] Run tests locally against the test DB and iterate until stable and deterministic under typical CI runner limits.
- [ ] Confirm dependent migration task (task_001_db_create_users_table) has been applied to the test DB before running the suite (tests will attempt apply/rollback where possible).

Effort: 6 hours

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Effort estimate is <=8 hours.