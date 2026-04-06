# Task - [TASK_010]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-DATA/us_001/us_001.md]
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
| Backend | Node.js | 18.x |
| Database | PostgreSQL | 14.x |
| Library | Knex (migration runner) | 2.x |
| CI | GitHub Actions | latest |
| Scripting | bash / node | bash + Node.js 18.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and scripts created in this task must be compatible with the versions above.

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
Add CI pipeline steps that:
- Launch an ephemeral PostgreSQL service in CI (GitHub Actions service container).
- Run the project migration set (us_001 migration must exist from dependent task) against the ephemeral DB.
- Execute a set of smoke checks (SQL queries or small Node.js script) to verify the users table, columns, constraints (unique index on email_normalized, check constraint on failed_attempts, enum/check for hash_algo), indexes (reset_token_hash index), and data types are present as required by us_001 acceptance criteria.
- Fail the CI job if migrations fail or any smoke check fails.
- Optionally run a migration rollback (down) as part of cleanup/verification.

Purpose: ensure migrations for user schema are automatically exercised in CI and schema guarantees are validated before merging.

Effort estimate: 4 hours

## Dependent Tasks
- Artefacts/Tasks that must be completed first:
  - Task 001: Create the us_001 DB migration file (up + down) that creates the users table, constraints, indexes, and types as specified in user story us_001 (this must exist and be runnable by the migration runner used below).
  - Task 000: Confirm DB platform and migration tooling (ensure project uses PostgreSQL and Knex or equivalent; confirm migration CLI command).
  - If project uses a different migration runner, ensure a small README or script exists to run migrations non-interactively in CI.

## Impacted Components
- CI: .github/workflows/db-migrations-check.yml (new)
- Package scripts: package.json (add migration CI script)
- CI scripts: scripts/ci/db-smoke-check.sh (new) and/or scripts/ci/db-smoke-check.js (new)
- Tests: tests/db/smoke_check.sql (new) or tests/db/smoke_check.test.js (new)
- Docs: docs/ci/README.md or README.md (modify to document CI job)
- Utilities: scripts/ci/wait-for-postgres.sh (optional helper)

## Implementation Plan
- Create a GitHub Actions workflow (.github/workflows/db-migrations-check.yml) that:
  - Triggers on push & pull_request for relevant branches.
  - Starts a postgres:14 service container with environment variables (POSTGRES_DB=ci_db, POSTGRES_USER=ci_user, POSTGRES_PASSWORD=ci_pass).
  - Waits for Postgres to be ready (use a wait-for script or psql retry loop).
  - Sets DB connection env vars used by migration runner (DATABASE_URL or PGHOST/PGUSER/PGPASSWORD/PGDATABASE).
  - Runs the project's migration command (e.g., npm run migrate:ci or npx knex migrate:latest) to apply migrations including us_001.
  - Runs the smoke-check script that connects to the DB and runs validation SQL statements; fail job on non-zero exit code.
  - Optionally runs migration rollback (migrate:down) to verify down migration (if safe).
- Add a smoke-check script (scripts/ci/db-smoke-check.sh or .js) that:
  - Executes SQL checks against the database:
    - Verify users table exists and has required columns and types (query information_schema.columns and compare types/defaults).
    - Verify UNIQUE constraint/index on email_normalized exists.
    - Verify CHECK constraint failed_attempts >= 0 exists.
    - Verify an index exists on reset_token_hash (and optionally partial index if migration created one).
    - Verify either an enum type or CHECK constraint exists for allowed hash_algo values.
  - Print clear diagnostics on failure (which check failed and query output).
  - Exit non-zero if any check fails.
- Add tests/db/smoke_check.sql or tests/db/smoke_check.js to contain the canonical SQL used for checks; the CI script will execute it (psql -f or node-run).
- Modify package.json to add an npm script "migrate:ci" (or update according to project's existing migration runner) that runs migrations non-interactively using env vars.
- Update docs/ci/README.md with steps for running these checks locally and in CI.
- Validate locally by running the workflow commands against a locally running Postgres container or using docker-compose to replicate CI flow.

## Current Project State
- Project root:
  - package.json (assumed to contain migration CLI entry; may need addition)
  - migrations/ (assumed to contain migration files; must include us_001 migration created by dependent task)
  - .github/workflows/ (may contain other workflows; new workflow will be added)
  - scripts/ (create scripts/ci/)
  - tests/db/ (create smoke_check files)
- Note: This is a placeholder snapshot. The dependent migration file (us_001) must exist prior to running this CI job.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | .github/workflows/db-migrations-check.yml | GitHub Actions workflow that starts a Postgres service, runs migrations, and runs DB smoke checks; fails CI on any failure. |
| CREATE | scripts/ci/db-smoke-check.sh | Bash script that runs SQL checks against the ephemeral CI Postgres instance and exits non-zero on failures. |
| CREATE | scripts/ci/wait-for-postgres.sh | Small helper script to wait for Postgres to accept connections (used by workflow). |
| CREATE | tests/db/smoke_check.sql | SQL file with the validation queries (column existence/types, unique/index checks, check constraints, index on reset_token_hash, enum/check for hash_algo). |
| CREATE | docs/ci/README.md | Short doc describing how the CI job runs migrations and smoke checks; how to run locally. |
| MODIFY | package.json | Add "migrate:ci" npm script (example: "DATABASE_URL=${CI_DATABASE_URL} knex migrate:latest") or update migration script to be usable non-interactively in CI. |
| MODIFY | README.md | Add a short note referencing the new CI check and how to run locally (or reference docs/ci/README.md). |

## External References
- GitHub Actions services: https://docs.github.com/en/actions/using-containerized-services/about-service-containers
- PostgreSQL information_schema: https://www.postgresql.org/docs/current/infoschema.html
- Inspecting constraints/indexes in Postgres: https://www.postgresql.org/docs/current/catalog-pg-index.html and using pg_constraint and pg_index
- Knex migrations docs (if project uses Knex): https://knexjs.org/#Migrations
- Example wait-for-postgres script pattern: https://github.com/vishnubob/wait-for-it (or use a simple psql retry loop)

## Build Commands
- Run migrations locally (example): npm run migrate:ci
- Run smoke checks locally (example): bash scripts/ci/wait-for-postgres.sh && bash scripts/ci/db-smoke-check.sh
- GitHub Actions will run the workflow defined in .github/workflows/db-migrations-check.yml
(Refer to project-specific migration command in package.json if different.)

## Implementation Validation Strategy
- [ ] Unit tests pass (if smoke-check unit tests are added)
- [ ] Integration tests pass (CI workflow completes successfully against ephemeral Postgres service)
- [ ] CI workflow runs migrations and smoke checks and exits 0 on success
- [ ] Smoke checks validate:
    - users table exists with required columns and types
    - UNIQUE constraint on email_normalized present
    - CHECK constraint failed_attempts >= 0 present
    - Index on reset_token_hash present (and partial index if migration created one)
    - Enum/check for allowed hash_algo present
- [ ] Migration runner invoked non-interactively and respects CI env vars
- [ ] Docs updated explaining how to run checks locally and how CI job behaves

## Implementation Checklist
- [ ] Create GitHub Actions workflow at .github/workflows/db-migrations-check.yml that starts postgres:14 service, waits for readiness, runs migration CLI, runs smoke-check script, and fails on any error. (Estimated: 1.5 hours)
- [ ] Add scripts/ci/wait-for-postgres.sh and scripts/ci/db-smoke-check.sh (or .js) that run the SQL checks described in tests/db/smoke_check.sql and produce actionable failure logs. (Estimated: 1.5 hours)
- [ ] Add tests/db/smoke_check.sql containing the exact validation SQL queries for table/column/types/constraints/indexes and ensure psql-compatible output for CI consumption. (Estimated: 1 hour)
- [ ] Modify package.json to add "migrate:ci" script that runs the migration runner against DATABASE_URL or PG* env vars used in CI. (Estimated: 0.5 hours)
- [ ] Update docs/ci/README.md and README.md with instructions for local testing and CI behavior. (Estimated: 0.5 hours)
- [ ] Run the workflow locally (or via a personal branch) to validate service readiness timing, migration success, and smoke-check behavior; iterate until stable. (Estimated: 1 hour)
- [ ] Finalize and push changes; monitor first CI run and fix any environment or timing issues (retries, connection wait). (Estimated: 0.5 hours)

Notes:
- Maximum checklist items = 7 (<=8) and estimated effort sum = 6.5 hours (<=8 hours).
- Reference the us_001 migration (dependent task) during implementation. The CI job must fail if us_001 migration is missing or errors.

## RULES:
- Implementation Checklist has <=8 items (7 items).
- Effort estimate is <=8 hours (total ~6.5 hours).
- Acceptance Criteria are pulled directly from the parent User Story (us_001) and included above.

