# Task - [TASK_002]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/us_001/us_001.md]
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
| Backend | Node.js | 18.x (or compatible) |
| Database | PostgreSQL | 12.x - 15.x (detect at runtime) |
| Library | Knex (migration helper) or plain SQL migration scripts | 2.x (or SQL files) |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries must be compatible with the versions above. The implementation must be DB-feature-agnostic where possible, with explicit runtime detection for PostgreSQL capabilities. If another migration runner is in use in the repo, adapt the scripts/hooks accordingly.

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
Implement database migration compatibility checks and pre-check utilities used by the us_001 users-table migration so that migrations:
- detect DB engine and version/features at runtime,
- choose CREATE TYPE ENUM (Postgres) vs CHECK constraint for hash_algo based on support and safe upgrade semantics,
- choose partial index for reset_token_hash when the DB supports partial indexes (Postgres) and fall back to a non-partial index otherwise,
- make migration idempotent and safe to run against existing schemas (no destructive operations that fail if objects already exist),
- perform pre-check validations (e.g., required extensions like pgcrypto for gen_random_uuid, permission checks, and transactional behavior),
- provide clear logging and exit early with actionable messages when compatibility issues are detected.

This task does not create the final users table migration contents (that is task_001). Instead, it adds the compatibility/pre-check layer and modifies the us_001 migration to use it, ensuring the migration can target different Postgres versions and be re-run safely.

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/task_001_db_create_users_table.md (or equivalent): base users table migration script must be drafted so this compatibility layer can be integrated.
- Ensure migration runner is configured in the repo (e.g., Knex/Flyway/other). If not present, set up minimal migration runner configuration before starting.
- Access to a test PostgreSQL instance(s) for versions 12.x, 13.x, and 15.x to validate feature detection (or CI matrix).

## Impacted Components
- Server/db/migrations/helpers/db_compatibility.ts (new) — runtime DB capability detection and helpers
- Server/db/migrations/us_001_create_users_table.sql or .js migration — modified to call compatibility helpers and to be idempotent
- Server/db/migrations/README.md — new/update documentation for migration compatibility
- Server/test/db/migrations/compatibility.test.ts — new unit/integration tests for detection and fallback behavior
- .github/workflows/migration-ci.yml — modify/add a CI job to run migration pre-checks (create if missing)

## Implementation Plan
- Step 1 — Add a DB compatibility helper module:
  - Implement Server/db/migrations/helpers/db_compatibility.ts that exposes:
    - detectEngine(): returns engine name (postgres) and major/minor version
    - supportsEnums(): boolean (Postgres supports CREATE TYPE enum; but module will detect if running Postgres)
    - supportsPartialIndexes(): boolean (Postgres supports partial indexes; ensure version >=9.0)
    - supportsGenRandomUUID(): boolean (checks for pgcrypto extension or gen_random_uuid() function)
    - runPreChecks(client): performs permission checks (can CREATE, can CREATE TYPE, can CREATE INDEX), extension presence, and returns a structured report
    - safeExecute(sql, client): helper to run "CREATE ... IF NOT EXISTS" patterns or conditional DDL inside transaction when possible
  - Make detection SQL lightweight (SELECT version(), check pg_catalog.pg_type existence etc.)
- Step 2 — Update us_001 migration to use helpers:
  - Modify migration to import/require compatibility helpers and branch:
    - If supportsEnums() true: CREATE TYPE IF NOT EXISTS user_hash_algo AS ENUM (...); use that for hash_algo column.
    - Else: implement CHECK (hash_algo IN (...)) with idempotent creation (add constraint IF NOT EXISTS pattern where DB supports or perform conditional check in migration code).
    - For reset_token_hash index: if supportsPartialIndexes() create partial index WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now(); otherwise create normal index on reset_token_hash.
  - Ensure all DDL uses "IF NOT EXISTS" or guarded execution so migration is idempotent and safe to re-run.
  - Use gen_random_uuid() if available; otherwise migration leaves id UUID without server default and relies on app to set UUIDs. Document the fallback.
- Step 3 — Add tests:
  - Unit tests for the detection helpers using a test DB (mock or real) to assert capabilities detection.
  - Integration test that runs the modified us_001 migration against a clean DB and a DB with pre-existing partial objects to ensure idempotency.
  - Concurrency test (separate short test) to ensure CHECK constraint and atomic updates exist; simulate concurrent increments (best-effort within this task; otherwise add integration test that ensures failed_attempts constraint exists).
- Step 4 — Documentation & CI:
  - Add Server/db/migrations/README.md explaining the compatibility checks and recommended steps for operators.
  - Add/modify CI workflow to run pre-checks and run migrations in a dry-run mode to catch failures earlier.
- Step 5 — Logging & Exit Behavior:
  - Ensure pre-checks produce developer-friendly errors and non-zero exit codes when mandatory capabilities are missing (e.g., no permission to CREATE TYPE and app relies on enum).
  - Provide guidance text in errors (e.g., "Install pgcrypto or set config to let app generate UUIDs").

## Current Project State
- app/ (frontend) — placeholder (no UI impact)
- Server/
  - db/
    - migrations/
      - (existing migration runner config or SQL/JS migrations)
      - helpers/ (new to be created)
  - test/
    - db/
      - migrations/ (tests to be added)
- .github/
  - workflows/
    - (migration-ci.yml may be new/modified)
- .propel/
  - context/tasks/EP-DATA/us_001/us_001.md (story file)
- Note: This is a placeholder state. Update files when dependent tasks (migration runner, base migration) are available.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | Server/db/migrations/helpers/db_compatibility.ts | Node/TS helper that detects DB engine/version and feature support, exposes safeExecute and runPreChecks used by migrations |
| MODIFY | Server/db/migrations/us_001_create_users_table.sql | Wrap DDL to call compatibility checks (or create an equivalent JS migration that calls helper), choose ENUM vs CHECK and partial-index vs fallback, add idempotent guards |
| CREATE | Server/test/db/migrations/compatibility.test.ts | Tests for capability detection and migration idempotency (unit/integration tests) |
| CREATE | Server/db/migrations/README.md | Documentation of compatibility behavior, operator guidance for extensions and permission requirements |
| MODIFY | .github/workflows/migration-ci.yml | Add job to run migration pre-checks against test DB (or create new workflow if not present) |

## External References
- PostgreSQL docs: CREATE TYPE (enum) — https://www.postgresql.org/docs/current/sql-createtype.html
- PostgreSQL docs: Partial Indexes — https://www.postgresql.org/docs/current/indexes-partial.html
- gen_random_uuid() / pgcrypto extension — https://www.postgresql.org/docs/current/pgcrypto.html
- Safe DDL patterns in Postgres — use "CREATE ... IF NOT EXISTS", check catalog tables (pg_type, pg_index) for idempotency

## Build Commands
- Run migrations locally (example): npm run migrate (ensure migration runner is configured)
- Run tests: npm run test -- Server/test/db/migrations/compatibility.test.ts
- Run pre-checks only (example script): node Server/db/migrations/helpers/db_compatibility.js --precheck --dsn $DATABASE_URL
- CI: .github/workflows/migration-ci.yml will call the same npm scripts

## Implementation Validation Strategy
- [ ] Unit tests pass for db_compatibility helpers
- [ ] Integration tests pass: modified us_001 migration runs successfully on a clean DB
- [ ] Integration tests pass: modified us_001 migration is idempotent when re-run against DB with pre-existing objects
- [ ] Concurrency test: failed_attempts CHECK exists and atomic increment semantics are exercised (best-effort within migration tests)
- [ ] Migration pre-checks produce actionable error messages when required capabilities or permissions are missing

## Implementation Checklist
- [ ] Implement Server/db/migrations/helpers/db_compatibility.ts with detection APIs and safeExecute helpers
- [ ] Modify the us_001 migration to call compatibility helpers and implement enum vs CHECK and partial-index vs fallback logic with idempotent guards
- [ ] Add unit/integration tests at Server/test/db/migrations/compatibility.test.ts validating detection logic and idempotent migration behavior
- [ ] Add Server/db/migrations/README.md documenting requirements (pgcrypto, permissions) and fallbacks
- [ ] Add/modify CI workflow to run pre-checks and migration in a dry-run as part of PR validation
- [ ] Run tests locally against a PostgreSQL test instance and fix edge cases found
- [ ] Create a short PR description explaining compatibility decisions and operator action items (extensions/permissions)

Effort: <= 8 hours (targeted to be completed in a single developer day; keep implementation scoped to helpers, migration modifications, and basic tests)

## RULES COMPLIANCE
- Implementation Checklist has <=8 items
- Effort estimate <=8 hours
- Acceptance Criteria are sourced from the parent User Story and copied into Requirement Reference

