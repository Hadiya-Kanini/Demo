# Task - task_001

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
| Backend | N/A (DB-focused) | N/A |
| Database | PostgreSQL | 13.x or 14.x (prefer 14+) |
| Library | Flyway (or project migration runner) | 9.x or project-standard migration tool |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: If the project uses a different migration tool (knex/Sequelize/Alembic/Goose), adapt commands and conventions accordingly. Target PostgreSQL features assumed: jsonb, partial indexes, CREATE TYPE enum, pgcrypto/gen_random_uuid().

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
Verify and document the production database platform (engine and exact version), allowed/approved PostgreSQL extensions (pgcrypto, uuid-ossp, etc.), and confirm the project's migration tooling, repository layout, and runner conventions so the team can finalize the SQL dialect and write portable, correct migrations for us_001. Deliverables: short technical report with confirmed DB version, supported extensions and permission model, recommended SQL dialect choices (e.g., use gen_random_uuid() vs uuid_generate_v4()), migration filename/naming conventions, example migration snippet for UUID default and extension creation (up/down), and required CI/runner changes (if any).

## Dependent Tasks
- Provide access to production/staging database connection details (read-only metadata access) or a DBA contact to confirm engine/version and allowed extensions.
- Provide access to the migration repository and current migration runner configuration (credentials, runner type and version).
- Ensure CI account or environment variable access that runs migrations for verification (or provide a local dev DB that mirrors production).

## Impacted Components
- infra/db/platform.md (new documentation file)
- migrations/ (migration repository; possible convention update)
- .github/workflows/db-migration-check.yml (CI job to validate migrations - CREATE or MODIFY)
- docs/ (README or contributing docs describing migration conventions)
- ops/ (notes or runbook entries for DBAs about enabling extensions)
- CI/CD pipeline step that runs migration lint/check

## Implementation Plan
- Step 1 — Discovery: Obtain DB engine/version and list of allowed extensions from DBAs or environment. Run psql -c "select version();" and queries to list installed extensions (SELECT * FROM pg_available_extensions).
- Step 2 — Migration tool verification: Inspect migration repo to determine runner (Flyway/knex/Alembic/etc.), file naming conventions, placeholders, SQL vs JS/py migrations, and how rollbacks are implemented.
- Step 3 — Compatibility analysis: Based on confirmed Postgres version, document feature availability required by user story (jsonb, partial indexes, CREATE TYPE/ENUM, gen_random_uuid()) and pick preferred implementations (e.g., prefer pgcrypto's gen_random_uuid() if pgcrypto allowed; fallback to uuid-ossp if configured).
- Step 4 — Produce concrete recommendations and examples: create a short example migration snippet showing extension creation (if permitted), UUID default, enum/check for hash_algo, UNIQUE index on email_normalized, partial index template for reset_token_hash, created_at/updated_at default and updated_at trigger suggestion.
- Step 5 — CI/Runner validation: run the example migration (in a sandbox DB or dev replica) using the project's migration runner; capture logs and any warnings. If migration runner does not support rollbacks, document required manual rollback steps.
- Step 6 — Deliverables: create docs/infra/db/platform.md with findings, recommended SQL dialect, migration naming convention, and sample migration snippets (up + down). Add a short checklist for developers and CI.
- Step 7 — Update CI (if needed): propose or add a minimal workflow file that runs a migration dry-run or validate step against a sandbox DB to prevent syntax/dialect drift.

## Current Project State
- app/
  - README.md
- Server/
  - migrations/  (existing migration files - inspect)
  - db/  (db connection helpers)
- docs/
  - contributing.md
- .github/
  - workflows/
- .propel/
  - context/
    - tasks/EP-DATA/us_001/us_001.md

(Actual repository layout may differ; above is a placeholder to be validated at start of task.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | docs/infra/db/platform.md | Discovery report: confirmed DB engine/version, allowed extensions, recommended SQL dialect choices, and migration runner conventions. Includes sample migration snippets (up/down) and checklist. |
| CREATE | migrations/000_confirm_platform_example__us_001.sql | Example SQL migration snippet demonstrating extension enablement (conditional), UUID default, hash_algo enum/check template, UNIQUE index on email_normalized, reset_token_hash index (partial/regular), created_at/updated_at defaults (no destructive changes). Marked as example and not intended to be applied to prod without review. |
| MODIFY | .github/workflows/validate-migrations.yml | Add or update CI workflow to run a migration validation step (dry-run or lint) against a sandbox DB (create new workflow if none exists). |
| MODIFY | docs/contributing.md | Add brief section describing migration conventions (naming, rollback expectations, and required DB extensions). |
| CREATE | ops/db/notes_enable_extensions.md | Operational note for DBAs: required extensions and how to enable them safely in production (pgcrypto, uuid-ossp, any permission notes). |

## External References
- PostgreSQL official docs: https://www.postgresql.org/docs/
- pgcrypto docs (gen_random_uuid): https://www.postgresql.org/docs/current/pgcrypto.html
- uuid-ossp docs: https://www.postgresql.org/docs/current/uuid-ossp.html
- Jsonb docs: https://www.postgresql.org/docs/current/datatype-json.html
- Partial index docs: https://www.postgresql.org/docs/current/indexes-partial.html
- Flyway documentation (if Flyway used): https://flywaydb.org/documentation/
- Example updated_at trigger pattern: https://wiki.postgresql.org/wiki/Updating_a_timestamp_column
- Postgres enum vs check constraint guidance: https://www.citusdata.com/blog/2017/12/12/postgres-enums/

## Build Commands
- Check Postgres version: psql "host=... user=... dbname=... sslmode=require" -c "select version();"
- List available extensions: psql -c "SELECT name, default_version, installed_version FROM pg_available_extensions();"
- Run migration (Flyway example): flyway -url=jdbc:postgresql://HOST:PORT/DB -user=USER -password=PASS migrate
- Run migration (psql example for SQL file): psql "host=... user=... dbname=... sslmode=require" -f migrations/000_confirm_platform_example__us_001.sql
- CI validation: use project's migration runner command as configured in CI; if none, run a local dry-run against a dev DB.

(See ../.propel/build/ for project-specific build/runner commands if present.)

## Implementation Validation Strategy
- [ ] Unit tests pass (when applicable for migration helpers)
- [ ] Integration tests pass (apply example migration to dev/sandbox DB and rollback where supported)
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A)
- [ ] Run /analyze-ux to validate wireframe alignment (N/A)
- [ ] Prompt templates validated with test inputs (N/A)
- [ ] Guardrails tested for input sanitization and output validation (N/A)
- [ ] Fallback logic tested with low-confidence/error scenarios (N/A)
- [ ] Token budget enforcement verified (N/A)
- [ ] Audit logging verified (N/A)

## Implementation Checklist
- [ ] Obtain DB metadata: run queries to confirm engine and exact version, and capture output (psql SELECT version()).
- [ ] Confirm migration runner: identify tool, version, file naming and rollback conventions by inspecting repo and CI.
- [ ] Verify allowed extensions with DBA/ops and record whether pgcrypto and/or uuid-ossp are permitted in production.
- [ ] Produce platform.md with final SQL dialect choices, extension guidance, and sample migration (up + down) for us_001.
- [ ] Run the example migration against a sandbox/dev DB using the project's migration runner and capture logs/results.
- [ ] Update docs/contributing.md with migration conventions and required extensions checklist.
- [ ] Propose or add a CI validation workflow to lint/run migrations against a sandbox DB to prevent dialect drift.
- [ ] Share deliverables with EP-DATA stakeholders and DBAs for sign-off (attach samples and results).

Effort estimate: <= 8 hours.

## RULES:
- Implementation Checklist has 8 items (<=8) and effort <=8 hours.
- Acceptance Criteria are taken verbatim from the parent User Story us_001.