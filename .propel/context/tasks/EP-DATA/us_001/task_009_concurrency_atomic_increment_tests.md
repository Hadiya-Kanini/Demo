# Task - task_009

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-DATA/us_001/us_001.md
- Acceptance Criteria:  
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given the new schema, When failed_attempts is updated by concurrent login attempts, Then increments are atomic (performed using a single UPDATE ... SET failed_attempts = failed_attempts + 1 or DB atomic increment) and the database enforces a CHECK constraint failed_attempts >= 0 so negative values cannot be persisted.
    - Given a database migration runner with valid DB credentials, When the us_001 migration is applied, Then the users table is created with columns: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL with default ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
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
| Backend | Node.js (test runner scripts) | 18.x |
| Database | PostgreSQL | 13.x - 15.x (recommended 14) |
| Library | node-postgres (pg) | 8.x |
| Testing | Jest (or Mocha) | 29.x (Jest) |
| Container / Orchestration | Docker, docker-compose | Docker Engine 20.x, Compose V2 |
| Migration Tooling | Project migration runner (must have applied us_001) | N/A |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the specified versions or newer.

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
Simulate concurrent clients performing atomic increments of the users.failed_attempts counter and verify that no increments are lost. Implement an integration test that:

- Spins up (or connects to) a PostgreSQL test instance seeded with the us_001 users table and a single user row.
- Runs a configurable number (e.g., 100) of concurrent clients that each execute an atomic UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = $1 RETURNING failed_attempts.
- Validates the final failed_attempts value equals initial + N, and that the DB CHECK constraint failed_attempts >= 0 is present and enforced.
- Includes a negative-test variant that demonstrates what happens if an unsafe read-modify-write operation is used (optional) to illustrate lost updates for regression detection.

Purpose: Provide automated regression coverage for atomic increment behavior required by us_001 and to surface concurrency-related migration/schema issues.

## Dependent Tasks
- Apply migration for us_001 so that the users table and constraints/indexes exist (.propel/context/tasks/EP-DATA/us_001/us_001.md).
- Ensure test infrastructure is available: Docker & docker-compose and a Postgres test image accessible in CI.
- Ensure test DB credentials and migration runner are available for CI tests (CI job that can apply migrations to test DB).
- Task_006 or equivalent migration unit tests verifying the users table structure should be completed prior to running these concurrency tests.

## Impacted Components
- tests/concurrency/failedAttemptsConcurrency.test.ts (new)
- tests/helpers/dbClient.ts (new) - DB connection helper for tests
- docker/test-postgres/docker-compose.yml (new) - local test DB for CI/local runs
- .env.test or test configuration (CREATE or MODIFY) - DB connection strings for tests
- README or CONTRIBUTING docs (MODIFY) - instructions to run concurrency tests locally

## Implementation Plan
- Create a test-only DB helper that can:
  - apply or verify migration has been applied (optionally run migration runner or assert table exists),
  - create a user row with known id and initial failed_attempts = 0,
  - execute raw SQL queries and return results.
- Implement a Jest integration test (failedAttemptsConcurrency.test.ts) that:
  - starts with a fresh user row (failed_attempts = 0),
  - spawns N concurrent Promise tasks using node-postgres clients (pool or separate clients) where each task runs the single-statement atomic update:
    UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = $1 RETURNING failed_attempts;
  - awaits all tasks, then performs SELECT failed_attempts FROM users WHERE id = $1 and asserts equals N.
  - asserts the DB CHECK constraint exists (via information_schema or pg_catalog query) and that attempting to set failed_attempts = -1 is rejected.
  - includes a tear-down that truncates the users table or removes the test user.
- Add Docker Compose file for local/CI test Postgres so tests are reproducible in CI.
- Provide clear test configuration and npm script: "test:concurrency".
- Keep test time bounded (configurable concurrency and attempts) so it runs in CI within acceptable time (< 2 minutes).
- Document how to run the test locally and in CI.

## Current Project State
- app/ (placeholder for frontend; not used)
- server/ (backend code and migrations)
  - server/migrations/ (migration files; us_001 migration expected to exist)
  - server/src/ (backend code)
- tests/
  - existing unit tests (if any)
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story / migration spec)
Note: This is a placeholder snapshot. Update during execution if dependent migration or test infra files are added/changed.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/concurrency/failedAttemptsConcurrency.test.ts | Integration test that spawns concurrent UPDATE ... SET failed_attempts = failed_attempts + 1 requests and validates final counter equals expected value and DB constraints are enforced. |
| CREATE | tests/helpers/dbClient.ts | Test DB helper: connection pool, helper functions to ensure migrations applied, create/delete test user, run raw queries. |
| CREATE | docker/test-postgres/docker-compose.yml | Docker Compose definition for a test Postgres instance configured for CI/local runs with recommended Postgres version and a volume for ephemeral data. |
| CREATE | .env.test.example | Example test environment variables (PGHOST, PGPORT, PGUSER, PGPASSWORD, PGDATABASE) for running the concurrency tests locally. |
| MODIFY | README.md | Add section describing how to run the concurrency tests locally and CI prerequisites (apply migrations, start test DB). |

## External References
- PostgreSQL documentation: Concurrency control, UPDATE behavior and MVCC — https://www.postgresql.org/docs/current/mvcc.html
- PostgreSQL CHECK constraints — https://www.postgresql.org/docs/current/ddl-constraints.html#DDL-CONSTRAINTS-CHECK-CONSTRAINTS
- node-postgres (pg) documentation — https://node-postgres.com/
- Example discussion of lost updates and atomic increments — https://www.citusdata.com/blog/2018/11/21/avoid-race-conditions-in-postgresql/
- Docker Compose for Postgres examples — https://docs.docker.com/compose/compose-file/compose-file-v2/

## Build Commands
- Use project build scripts; additionally:
  - Start test DB: docker-compose -f docker/test-postgres/docker-compose.yml up -d
  - Run migrations (project migration runner): ./scripts/run-migrations.sh --env test (or project-specific command)
  - Run tests: npm run test:concurrency
- Reference: ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass
- [ ] Integration tests pass (concurrency test verifies no lost updates)
- [ ] Validate DB CHECK constraint failed_attempts >= 0 exists and rejects negatives
- [ ] Test runs reliably in CI with provided docker/test-postgres Compose file
- [ ] Test time remains bounded and configurable (default concurrency ≤ 100)
- [ ] Optional: Demonstrate a non-atomic read-modify-write pattern leads to lost updates in a controlled negative test

## Implementation Checklist
- [ ] Create tests/helpers/dbClient.ts with pool, helper functions to create/delete test user, run arbitrary SQL, and check for the existence of the users table and check constraint.
- [ ] Add docker/test-postgres/docker-compose.yml to provide a reproducible Postgres instance for CI/local runs.
- [ ] Implement tests/concurrency/failedAttemptsConcurrency.test.ts that runs N concurrent atomic UPDATE statements and asserts final failed_attempts == initial + N and verifies CHECK constraint behavior.
- [ ] Add .env.test.example and update README.md with exact commands to start DB, apply migrations, and run tests.
- [ ] Validate tests in CI (or locally) by running docker-compose up, applying us_001 migration, and executing npm run test:concurrency.
- [ ] Keep total implementation effort ≤ 8 hours (development + test runs).
- [ ] If a failing scenario is detected (lost updates or missing CHECK constraint), open an issue and block merging until migration/us_001 is corrected.

RULES followed:
- Implementation Checklist has 7 items (<=8).
- Effort capped to <=8 hours.

