# Task - task_007

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-DATA/us_001/us_001.md
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is applied, Then the users table is created with columns: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL with default ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the new schema, When a new user row is inserted with email, email_normalized, and password/hash fields, Then the database enforces a UNIQUE constraint on email_normalized and the insert succeeds for a single unique normalized email and fails with a unique-violation error for duplicates.
    - Given the new schema, When failed_attempts is updated by concurrent login attempts, Then increments are atomic (performed using a single UPDATE ... SET failed_attempts = failed_attempts + 1 or DB atomic increment) and the database enforces a CHECK constraint failed_attempts >= 0 so negative values cannot be persisted.
    - Given the schema and application policy, When a reset token is generated and its hash stored, Then there is an index on reset_token_hash (and a partial index for non-null and not-expired tokens if DB supports partial indexes) to allow efficient lookup by token hash and to support single-use token verification; application sets reset_token_expiry > now() at time of creation and enforces expiry semantics.
    - Given supported hash algorithms configured in the application, When a row is written or updated with a hash_algo value, Then the DB or migrations include a domain-level constraint (enum/check) listing allowed algorithm identifiers (e.g., 'argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256') or the application validates hash_algo against an allowed set and rejects unknown algorithms; password_hash column can store variable-length binary/text to accommodate algorithms and legacy formats.
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

## Implementation Checklist
- Add .github/workflows/migration-runner.yml with service container (Postgres) and steps to call ci/run_migration_and_verify.sh. (Estimated: 1.5h)
- Create ci/run_migration_and_verify.sh that: waits for DB, runs migration runner, runs verify_schema.sql, executes integration tests via npm script, attempts rollback/drop, and uploads artifacts on failure. (Estimated: 2h)
- Implement ci/verify_schema.sql containing concrete checks for table, columns, unique index on email_normalized, check constraint failed_attempts >= 0, reset_token_hash index, and presence of enum/check for hash_algo. Exit non-zero on mismatch. (Estimated: 1h)
- Add ci/artifact-collect.sh to dump schema (pg_dump/schema-only) and capture migration/test logs when a step fails; wire into workflow as post-failure step. (Estimated: 0.5h)
- Add package.json script "ci:migrations" that accepts CI_DATABASE_URL env var and runs integration tests against it; update README/README-CI snippet. (Estimated: 0.5h)
- Optional: add ci/docker-compose.ci.yml for local reproduction that mirrors GitHub Actions service container envs. (Estimated: 0.5h)
- Run the workflow locally or via GitHub Actions dry-run, iterate on flaky waits/timeouts, validate artifacts and verify cleanup behavior (drop schema or use ephemeral DB). (Estimated: 1h)
- Effort estimate (total): 7 hours

## RULES:
- Implementation Checklist must have <=8 items
- Effort must be <=8 hours
- Be specific and actionable — no vague descriptions
- Expected Changes table must list concrete file paths (CREATE/MODIFY/DELETE)
- Acceptance Criteria must come from the parent User Story