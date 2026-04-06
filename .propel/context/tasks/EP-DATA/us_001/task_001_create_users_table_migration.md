# Task - [TASK_001]

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
| Backend | Node.js (migration runner integration) | 18.x |
| Database | PostgreSQL | 15.x (>=12 recommended for partial index support) |
| Library | SQL migrations (plain SQL / migration runner) | N/A |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and SQL must be compatible with PostgreSQL (partial index and CREATE TYPE supported). If a different RDBMS is used, include capability detection and fallback (non-partial index).

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
Add a new forward and rollback database migration that creates the users table according to us_001 requirements. The migration must:
- Create the users table with specified columns, types, defaults.
- Add a UNIQUE constraint on email_normalized.
- Add a CHECK constraint failed_attempts >= 0.
- Add a domain-level constraint for allowed hash_algo values (Postgres enum preferred; fallback to CHECK).
- Create an index on reset_token_hash and, when PostgreSQL supports it, create a partial index to cover only active (non-null and not-expired) tokens. Provide a non-partial index fallback for DBs without partial-index support.
- Ensure forward and rollback scripts are fully transactional (or manage ordering if migration runner requires separate up/down files).
- Document allowed hash algorithms in the migration header comments.

Purpose: provide a robust DB schema foundation to support authentication, account protection, and secure recovery flows, while enabling migration-from-legacy-hashes workflows in application code.

## Dependent Tasks
- Ensure DB migration runner is configured and accessible (e.g., Flyway, Knex, Goose, custom runner). (If not present, configure migration runner before applying this migration.)
- Ensure the DB environment has extensions available if using gen_random_uuid() (pgcrypto) — either enable extension in a prior migration or require application to supply UUIDs.
- Backup/staging DB available for testing migrations.

## Impacted Components
- migrations/us_001_create_users_table.up.sql (new)
- migrations/us_001_create_users_table.down.sql (new)
- db/schema (logical): users table and associated indexes/constraints
- docs/migrations/us_001_readme.md (new) — documents allowed hash_algos and operational notes
- CI migration verification job (may need to reference the new migration)

## Implementation Plan
- Create SQL forward migration file (migrations/us_001_create_users_table.up.sql) that:
  - Creates enum type users_hash_algo_t (if PostgreSQL) with allowed values: 'argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256' — include comment to adjust if policy changes.
  - Creates users table with specified columns and defaults. Use id UUID PRIMARY KEY; prefer gen_random_uuid() if pgcrypto enabled, otherwise omit server default and let app set UUID.
  - Adds UNIQUE(email_normalized).
  - Adds CHECK (failed_attempts >= 0).
  - Creates index on reset_token_hash. Also create partial index: CREATE INDEX ... ON users(reset_token_hash) WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now(); include IF NOT EXISTS guards where appropriate.
  - Wrap in transaction (BEGIN/COMMIT) or rely on migration runner's transactional guarantees.
  - Add comments on columns where helpful.
- Create SQL rollback migration file (migrations/us_001_create_users_table.down.sql) that:
  - Drops indexes if exist, drops table users, drops enum type users_hash_algo_t (only if created by this migration and no other dependency).
  - Wrap rollback in transaction.
- Provide fallback for DBs without ENUM support or where enum removal is undesirable — implement an alternative CHECK on hash_algo IN (...) if CREATE TYPE not possible.
- Add a short README (docs/migrations/us_001_readme.md) documenting:
  - Allowed hash_algos and rationale.
  - Notes on gen_random_uuid() and pgcrypto extension.
  - Partial-index behavior and fallback semantics.
  - Guidance for application-level enforcement of reset_token_expiry > now().
- Add minimal migration verification steps (SQL or small script) to validate constraints and indexes on a dev DB (examples included in README).
- Run migration locally against a dev Postgres instance and run the verification SQL snippets (insert valid row, attempt duplicate normalized email insert to assert unique violation, simulate atomic increment via UPDATE ... SET failed_attempts = failed_attempts + 1, insert reset token and query index scan plans if needed).
- Commit migration files and README.

## Current Project State
- app/ (frontend placeholder)
- server/ (backend placeholder)
- migrations/ (migration scripts directory; may exist or be created)
- docs/migrations/ (migration docs directory; may exist or be created)

Note: Update the tree above based on the actual project layout if migration runner expects a different path. The "migrations/" directory is used as the canonical location for this task.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | migrations/us_001_create_users_table.up.sql | Forward migration creating users table, constraints, indexes, and enum/check for hash_algo. |
| CREATE | migrations/us_001_create_users_table.down.sql | Rollback migration to drop indexes, table, and enum/type (if created). |
| CREATE | docs/migrations/us_001_readme.md | Documentation for allowed hash_algos, pgcrypto/UUID notes, partial index fallback, and verification steps. |

## External References
- PostgreSQL CREATE INDEX documentation (partial index): https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL CREATE TYPE (ENUM): https://www.postgresql.org/docs/current/sql-createtype.html
- PostgreSQL gen_random_uuid() (pgcrypto/pgcrypto alternative): https://www.postgresql.org/docs/current/pgcrypto.html
- Best practices for password storage (OWASP): https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html

## Build Commands
- Run migrations using the project's migration runner. Examples (adjust to your runner):
  - psql -d $DB_URL -f migrations/us_001_create_users_table.up.sql
  - psql -d $DB_URL -f migrations/us_001_create_users_table.down.sql (for rollback)
  - If using a JS migration runner (example): npm run migrate:up (ensure migrations/ is configured)
- See repo build scripts at: ../.propel/build/ (follow project-specific migration invocation there)

## Implementation Validation Strategy
- [ ] Unit tests pass (if migration tests included)
- [ ] Integration tests pass (migration applied and rollback tested on ephemeral DB)
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A)
- [ ] Run /analyze-ux to validate wireframe alignment (N/A)
- [ ] Prompt templates validated with test inputs (N/A)
- [ ] Guardrails tested for input sanitization and output validation (N/A)
- [ ] Fallback logic tested with low-confidence/error scenarios (N/A)
- [ ] Token budget enforcement verified (N/A)
- [ ] Audit logging verified (no PII in logs) (N/A)

Tests / verification steps to perform:
- Apply forward migration to a dev Postgres instance.
- Insert a valid user row; assert success.
- Attempt to insert a second row with same email_normalized; assert unique-violation (SQLSTATE 23505).
- Run concurrent increments simulation: spawn parallel UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts and assert final value equals expected number of increments.
- Insert reset_token_hash with reset_token_expiry > now(); verify index lookup performance (EXPLAIN ANALYZE SELECT ... WHERE reset_token_hash = '...' LIMIT 1).
- Verify hash_algo constraint by attempting inserts with allowed and disallowed values and asserting success/failure.

## Implementation Checklist
- [ ] Create migrations/us_001_create_users_table.up.sql implementing forward migration (columns, defaults, UNIQUE(email_normalized), CHECK failed_attempts >= 0, enum or CHECK for hash_algo, reset_token_hash index + partial index fallback).
- [ ] Create migrations/us_001_create_users_table.down.sql implementing rollback (drop indexes, table, drop enum type if created).
- [ ] Add docs/migrations/us_001_readme.md documenting allowed hash_algos, pgcrypto/UUID notes, partial-index fallback, and verification steps.
- [ ] Run migrations locally against a development Postgres and verify acceptance criteria (unique constraint, atomic increments, reset token index).
- [ ] Ensure migration is transactional (or correctly ordered) and safe to apply/rollback in staging.
- [ ] Commit migration and docs, open PR with migration SQL and README for review.
- [ ] Complete within 6 hours of focused implementation time.

Effort estimate: 6 hours

## RULES:
- Implementation Checklist has 7 items (<=8).
- Effort is 6 hours (<=8 hours).
- All Acceptance Criteria listed in Requirement Reference are extracted from the parent user story.