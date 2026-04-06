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
| Backend | Node.js (CI runner scripts & test harness) | 18.x |
| Database | PostgreSQL | 15.x |
| Library | node-pg-migrate (or plain psql SQL migrations) | v6 / N/A |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: CI job and scripts must be compatible with the listed versions. If the project uses a different runtime or migration runner (Flyway, Liquibase, Knex, etc.), adapt the CI script to call the project's existing migration runner.

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
Add a CI job (and supporting scripts) that applies the us_001 migration to an ephemeral PostgreSQL instance, runs the migration integration tests that verify schema, constraints, concurrency behavior, indexes and rollback, ensures cleanup or rollback occurs, and causes the CI job to fail if any schema or test regressions are detected. The job must be idempotent, secure (use CI secrets for DB credentials where needed), time-bounded, and must verify rollback or DB cleanup to avoid leaving dangling state in shared CI runners.

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/task_001_db_create_users_table.md (migration script: us_001_create_users_table.sql or equivalent) — MUST exist and be committed before this CI job is added.
- .propel/context/tasks/EP-DATA/us_001/task_002_tests_for_us_001.md (integration/unit tests that assert the acceptance criteria) — tests must be present to be executed by CI.
- Ensure project has an existing migration runner or document how to run raw SQL (e.g., psql). If not, add a small wrapper script to run migrations (see Expected Changes).

## Impacted Components
- .github/workflows/ci.yml (or other CI orchestration) — add new job or integrate into existing pipeline
- .github/workflows/migrate-runner.yml — new workflow file (recommended)
- ci/run_migration_and_verify.sh — new shell script to run migration, tests, verification, and teardown
- ci/docker-compose.ci.yml — new or modified compose file to spin up ephemeral Postgres for local CI (optional)
- migrations/us_001_create_users_table.sql or migrations/us_001_create_users_table.js — existing migration file (dependent)
- tests/integration/migration/us_001_migration.test.[js|ts] — integration tests to run in CI (dependent)
- Makefile or package.json scripts — add script entry to run CI migration checks (e.g., npm run ci:migrations)

## Implementation Plan
- Add a dedicated CI job workflow (recommended file: .github/workflows/migration-runner.yml) that:
  - Starts an ephemeral PostgreSQL instance (via GitHub Actions service container or docker-compose).
  - Waits until DB is ready (psql loop).
  - Runs migration runner to apply us_001 forward migration against the ephemeral DB.
  - Runs the integration/migration tests that validate ACs (unique constraint, atomic increments, indexes, allowed hash_algo constraint).
  - Optionally run sample SQL verification queries (psql) to validate indexes and constraints existence.
  - Executes migration rollback (if migration runner supports rollback) OR drops the ephemeral DB/schema to verify cleanup.
  - Fails the job if any step (apply, tests, rollback) fails; collects logs/artifacts for debugging.
- Create small helper scripts under ci/:
  - run_migration_and_verify.sh: encapsulates commands executed by CI (apply, run tests, run verification queries, rollback/drop).
  - verify_schema.sql: a SQL script with checks (e.g., check columns, constraints, index existence) that returns non-zero code on mismatch.
- Update package.json / Makefile to include a target that runs tests against a DB connection string (read from env var) so CI can execute tests with the ephemeral DB.
- Ensure the CI job uses least-privilege credentials where required (for production-like checks, ephemeral DB credentials may be ephemeral and generated by the job).
- Ensure timeouts and retries are present (DB readiness, migrations) to avoid flakiness.
- Add artifact collection on failures (psql schema dump and migration logs) to support debugging.
- Document the CI job in repo README or .propel/context/tasks/EP-DATA/us_001/README.md with how to reproduce locally.

## Current Project State
- app/ (frontend) — placeholder
- server/ or src/ (backend) — placeholder
- migrations/ — may contain migration files (assume migrations/us_001_create_users_table.sql will be present by dependency)
- tests/integration/ — may contain integration tests (must be added by dependent task)
- .github/workflows/ — existing CI workflows (ci.yml)
- ci/ — may not exist; will be created

Example tree (placeholder):
- .
  - .github/
    - workflows/
      - ci.yml
  - migrations/
    - (expected) us_001_create_users_table.sql
  - tests/
    - integration/
      - migration/
        - (expected) us_001_migration.test.ts
  - ci/
    - (will be created)
  - package.json / Makefile
  - .propel/context/tasks/EP-DATA/us_001/

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | .github/workflows/migration-runner.yml | New GitHub Actions workflow that runs the ephemeral Postgres, applies migrations, runs migration integration tests, verifies rollback/cleanup, and fails on regressions. |
| CREATE | ci/run_migration_and_verify.sh | Shell script invoked by CI: waits for DB, runs migration runner, executes verify_schema.sql, runs integration tests, attempts rollback/drop, and collects artifacts on failure. |
| CREATE | ci/verify_schema.sql | SQL script that validates the presence of users table columns, unique index on email_normalized, check constraint on failed_attempts, reset_token_hash index, and allowed hash_algo domain/constraint. Exits non-zero on any check failure. |
| CREATE | ci/docker-compose.ci.yml | Optional compose file to allow local reproduction of CI ephemeral DB environment (Postgres service with envs matching CI). |
| MODIFY | package.json | Add script "ci:migrations" that runs tests against DB using env CI_DATABASE_URL; also document usage. |
| MODIFY | .github/workflows/ci.yml | (Optional) Add dependency or notification step to incorporate migration-runner workflow or ensure ordering if pipeline requires it. |
| CREATE | ci/artifact-collect.sh | Small helper to dump schema and capture logs when a job step fails (created to be used by workflow on failure). |

## External References
- PostgreSQL CREATE TABLE docs: https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL partial indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- GitHub Actions services and service-containers: https://docs.github.com/en/actions/using-containerized-services/about-containerized-services
- node-pg-migrate (if used): https://github.com/salsita/node-pg-migrate
- psql CLI docs: https://www.postgresql.org/docs/current/app-psql.html

## Build Commands
- Run migration locally (example using psql): PGPASSWORD=$DB_PASS psql -h $DB_HOST -U $DB_USER -d $DB_NAME -f migrations/us_001_create_users_table.sql
- Run verify SQL: PGPASSWORD=$DB_PASS psql "$CI_DATABASE_URL" -f ci/verify_schema.sql
- Run integration tests against ephemeral DB: CI_DATABASE_URL=postgres://user:pass@localhost:5432/test npm run test:integration
- CI wrapper: bash ci/run_migration_and_verify.sh

(See ../.propel/build/ for project-specific build commands and CI standards)

## Implementation Validation Strategy
- [ ] Unit tests pass (project unit tests unaffected but run in CI as baseline)
- [ ] Integration tests pass (migration integration tests for us_001)
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px — N/A
- [ ] Run /analyze-ux to validate wireframe alignment — N/A
- [ ] Prompt templates validated with test inputs — N/A
- [ ] Guardrails tested for input sanitization and output validation — N/A
- [ ] Fallback logic tested with low-confidence/error scenarios — N/A
- [ ] Token budget enforcement verified — N/A
- [ ] Audit logging verified (no PII in logs) — ensure CI artifacts