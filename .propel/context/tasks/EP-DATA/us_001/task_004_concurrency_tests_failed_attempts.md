# Task - [TASK_004]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-DATA/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is executed, Then a users table is created with these columns and types: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL DEFAULT ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the users table exists, When simultaneous login attempts increment failed_attempts, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given the new schema, When failed_attempts is updated by concurrent login attempts, Then the database enforces a CHECK constraint failed_attempts >= 0 so negative values cannot be persisted.
- Edge Case:
    - How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and use row-level locking (SELECT ... FOR UPDATE) where necessary to prevent lost updates. Tests must validate concurrent increments and lockout threshold behavior.
    - What happens when a legacy password hash format is encountered? - The schema allows storing legacy hash formats (hash_algo captures legacy type; salt field is nullable). (Note: this task focuses on concurrency tests; legacy handling is out-of-scope for implementation but acknowledged.)

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
Build an automated integration test harness that simulates concurrent operations which increment users.failed_attempts. The tests will verify:
- Atomicity of concurrent UPDATE ... SET failed_attempts = failed_attempts + 1 operations (no lost updates).
- Correct behavior with and without explicit row-level locking (SELECT ... FOR UPDATE in transactional scenarios).
- Enforcement of the CHECK constraint failed_attempts >= 0 when attempts to set negative values are made.
- Proper returned values from UPDATE ... RETURNING failed_attempts under concurrency.

The harness will run against a transient PostgreSQL test instance (docker-compose) and use multiple parallel DB connections to simulate concurrency. Tests should be repeatable and deterministic as possible.

## Dependent Tasks
- Artefacts/Tasks:
  - task_001_db_create_users_table (the migration that creates the users table with required columns, UNIQUE index on email_normalized, and CHECK constraint failed_attempts >= 0). This migration must be applied to the test DB before running these tests.
  - Setup of test database runner / docker-compose test DB environment (if not already present).

## Impacted Components
- Server/test/integration/failed_attempts_concurrency.test.js (NEW)
- Server/test/helpers/db.js (NEW) - small DB helper to obtain pooled clients and run transactions
- Server/docker/test-postgres/docker-compose.yml (NEW) - local test postgres configuration for CI/local development
- Server/package.json (MODIFY) - add/ensure test script and test dependencies (jest, pg)
- Migrations artifacts (READ-ONLY for tests) - .migrations/us_001_create_users_table.sql (DEPENDENT, must exist)

## Implementation Plan
- Create a lightweight DB helper that can:
  - create a connection pool using node-postgres (pg)
  - provide convenience functions to run SQL, begin/commit/rollback transactions, and run concurrent tasks
- Add a docker-compose configuration to spin up a local PostgreSQL instance for tests (configured with a known user/db).
- Write integration tests that:
  - Ensure the users table exists (sanity check) and seed a user row with failed_attempts = 0.
  - Run N concurrent workers (e.g., 20) each issuing UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ...; allow them to run in parallel (not sequentially) and assert at the end failed_attempts == N.
  - Run the same scenario but inside explicit transactions using SELECT failed_attempts FROM users WHERE id = ... FOR UPDATE; read-modify-write-UPDATE to demonstrate need for locking and assert no lost updates.
  - Test behavior when a malicious UPDATE attempts to set failed_attempts = -1 and verify the DB CHECK constraint prevents the change (expect a constraint violation).
  - Measure returned values from UPDATE ... RETURNING failed_attempts in concurrent runs to confirm values are monotonically increasing or consistent with atomic increments.
- Add or modify package.json test script to start the test DB, run migrations (if automation available), run Jest, and teardown DB.
- Document how to run tests locally and in CI.

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
| CREATE | Server/test/helpers/db.js | DB helper that creates a pg Pool and provides runQuery, getClient, transaction helpers for tests. |
| CREATE | Server/test/integration/failed_attempts_concurrency.test.js | Jest integration test file implementing concurrent increment tests, FOR UPDATE tests, and CHECK constraint negative-set test. |
| CREATE | Server/docker/test-postgres/docker-compose.yml | docker-compose config to spin up a Postgres 16 container for integration tests (with known credentials). |
| MODIFY | Server/package.json | Add devDependencies (jest, pg), add "test:integration" script that spins up DB, runs migrations, runs Jest integration tests, then tears down. |
| MODIFY | .github/workflows/ci.yml (if present) | OPTIONAL - add step to run integration tests (note: only modify if CI exists; otherwise document setup). |

## External References
- PostgreSQL UPDATE concurrency considerations: https://www.postgresql.org/docs/current/explicit-locking.html
- PostgreSQL CHECK constraint docs: https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS
- Using RETURNING with UPDATE in Postgres: https://www.postgresql.org/docs/current/sql-update.html
- node-postgres (pg) pooling & transactions: https://node-postgres.com/

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
- [ ] Integration tests pass (concurrent increment scenarios succeed)
- [ ] Verify final failed_attempts equals expected total after concurrent increments
- [ ] Verify transactional SELECT ... FOR UPDATE path prevents lost updates under simulated read-modify-write patterns
- [ ] Verify CHECK constraint prevents failed_attempts from becoming negative (attempt to set -1 yields constraint violation)
- [ ] Tests are repeatable locally using the provided docker-compose test DB

## Implementation Checklist
- [ ] Create Server/test/helpers/db.js: implement pg Pool, runQuery, getClient, transactional helper functions (commit/rollback).
- [ ] Create Server/test/integration/failed_attempts_concurrency.test.js: implement concurrent UPDATE test, FOR UPDATE transactional test, and negative-value check constraint test (use Jest).
- [ ] Add Server/docker/test-postgres/docker-compose.yml to provision PostgreSQL 16 for tests.
- [ ] Modify Server/package.json to add devDependencies (jest, pg) and test:integration script that starts DB, runs migrations, runs Jest, and tears down. Keep script idempotent.
- [ ] Document test run steps in Server/README.md (short section) with commands and prerequisites.
- [ ] Run tests locally against the test DB and iterate until stable and deterministic under typical CI runner limits.
- [ ] Confirm dependent migration task (task_001_db_create_users_table) has been applied to the test DB before running the suite.

Effort: 6 hours

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Effort estimate is <=8 hours.

