# Task - task_002

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
    - How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and use row-level locking (SELECT ... FOR UPDATE) where necessary to prevent lost updates. Tests must validate concurrent increments and lockout threshold behavior.
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
| Backend | Migration SQL (PostgreSQL migration scripts, Flyway-compatible) | SQL / Flyway 9.x compatible |
| Database | PostgreSQL | 12.x+ (prefer 13+) |
| Library | N/A | N/A |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and SQL must be compatible with PostgreSQL 12+ and Flyway-compatible migration runners.

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
Implement a reversible (up/down) SQL migration to create the users table per us_001. The migration must:
- Create the users table with the specified columns, types, defaults.
- Add a UNIQUE constraint/index on email_normalized.
- Add a CHECK constraint (failed_attempts >= 0).
- Provide an enum or CHECK constraint limiting hash_algo values to an allowed set.
- Add index(es) for reset_token_hash (regular index and, if supported, a partial index for non-null AND reset_token_expiry > now()).
- Provide created_at and updated_at defaults and implement an updated_at trigger that sets updated_at = now() on UPDATE.
- Ensure the migration is reversible: DOWN migration must drop the table and any created types/functions/triggers in correct order.
- Include short documentation in migrations/README.md noting allowed hash algorithms and rationale.

Purpose: Provide the canonical DB schema for user accounts to support authentication, lockout, and password-reset flows.

## Dependent Tasks
- task_000 - Confirm DB platform & migration tooling (Postgres version, migration runner, allowed extensions)
- (Ops) Ensure DB role used by migrations has privileges to CREATE EXTENSION (if gen_random_uuid() is used) or adjust UUID generation method.

## Impacted Components
- migrations/V002__create_users_table.sql (new migration file to create users table, up/down)
- migrations/README.md (document allowed hash algorithms and migration notes)
- tests/db/migrations/us_001_migration_test.sql (migration verification scripts/SQL for CI)
- db/sql/functions/trigger_set_updated_at.sql (trigger function created/dropped by migration if not inlined)

## Implementation Plan
- Create migration SQL file (Flyway-style naming) that implements the UP and DOWN steps in a single file or separate up/down files per project convention. Use explicit transactional DDL where supported.
- In UP:
  - Optionally create extension pgcrypto if allowed and needed for gen_random_uuid(); otherwise use uuid_generate_v4() if uuid-ossp allowed. If extension creation is not allowed in prod, fallback to UUID generation on application side. For portability, implement id UUID DEFAULT gen_random_uuid() with a guard to create extension only if allowed.
  - Create an enum type users_hash_algo AS (...) OR implement a CHECK constraint restricting hash_algo to the allowed list ('argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256').
  - Create users table with specified columns, types, NOT NULL and default values. Use jsonb for roles with default '["user"]' or text[] — choose jsonb for flexibility.
  - Add UNIQUE INDEX on lower(email_normalized) or on email_normalized as provided. The story requires unique on email_normalized; assume normalized value provided by application (store as-is) and create unique index on email_normalized.
  - Add CHECK constraint failed_attempts >= 0.
  - Create index on reset_token_hash. Also create a partial index for tokens WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now() to optimize active-token lookups (Postgres supports partial indexes).
  - Add trigger function trigger_set_updated_at() and trigger to update updated_at on row UPDATE.
- In DOWN:
  - Drop trigger, drop trigger function, drop indexes, drop table, drop enum/type if created, and drop extension only if it was created by this migration.
- Create/update migrations/README.md documenting allowed hash algorithms, reasoning for jsonb roles, and notes about extension usage.
- Add simple SQL test scripts under tests/db to validate: table exists with columns and defaults, unique constraint enforcement on email_normalized, check constraint enforcement (cannot insert negative failed_attempts), index presence for reset_token_hash (EXPLAIN not required but presence checked), and that DOWN migration removes artifacts.
- Keep migration idempotent where possible (use CREATE TYPE IF NOT EXISTS, CREATE EXTENSION IF NOT EXISTS guards where acceptable).
- Ensure the migration runs within a transaction block (Postgres supports DDL transactions) to avoid partial application.

## Current Project State
- app/ (frontend placeholder - not applicable)
- server/ (backend placeholder - not applicable)
- migrations/
  - V001__initial_placeholder.sql
  - README.md
- tests/
  - db/
    - migrations/
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story file - present)

(The above is a placeholder snapshot — update during execution if dependent tasks add files or change conventions.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | migrations/V002__create_users_table.sql | SQL migration implementing UP (create type if needed, create table, indexes, constraints, trigger function) and DOWN (drop trigger, drop table, drop type if created). |
| CREATE | db/sql/functions/trigger_set_updated_at.sql | SQL for trigger function that sets NEW.updated_at = now() on UPDATE (if extracting function out of migration file). |
| CREATE | tests/db/migrations/us_001_migration_test.sql | SQL scripts used by CI to verify migration applied: column checks, constraint checks, uniqueness test, failed_attempts check. |
| MODIFY | migrations/README.md | Document allowed hash_algorithms, explanation of roles column (jsonb), note about extension usage and rollback behavior. |

## External References
- PostgreSQL CREATE INDEX partial indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL CHECK constraints: https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL CREATE TYPE (enum) docs: https://www.postgresql.org/docs/current/sql-createtype.html
- gen_random_uuid() (pgcrypto) vs uuid_generate_v4() (uuid-ossp): https://www.postgresql.org/docs/current/functions-uuid.html
- Trigger functions and updated_at pattern: https://www.postgresql.org/docs/current/plpgsql-trigger.html

## Build Commands
- Run migration (example using Flyway CLI):
  - flyway -url="jdbc:postgresql://$DB_HOST:$DB_PORT/$DB_NAME" -user="$DB_USER" -password="$DB_PASS" migrate
- Rollback (if down migrations supported by runner):
  - flyway -url="jdbc:postgresql://$DB_HOST:$DB_PORT/$DB_NAME" -user="$DB_USER" -password="$DB_PASS" undo
- Run tests (example using psql against test DB):
  - psql "$TEST_DB_CONN" -f tests/db/migrations/us_001_migration_test.sql

(Adjust commands for the project's migration runner if different.)

## Implementation Validation Strategy
- [ ] Unit tests pass (tests/db/migrations/us_001_migration_test.sql executed in CI).
- [ ] Integration tests pass (apply migration to test DB and verify behavior).
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A).
- [ ] Run /analyze-ux to validate wireframe alignment (N/A).
- [ ] Prompt templates validated with test inputs (N/A).
- [ ] Guardrails tested for input sanitization and output validation (N/A).
- [ ] Fallback logic tested with low-confidence/error scenarios (N/A).
- [ ] Token budget enforcement verified (N/A).
- [ ] Audit logging verified (no PII in logs) (N/A).

## Implementation Checklist
- [ ] Create migrations/V002__create_users_table.sql implementing UP and DOWN following plan (include CREATE TYPE or CHECK, table, constraints, indexes, trigger).
- [ ] Add trigger function file db/sql/functions/trigger_set_updated_at.sql or inline function in migration and wire up trigger on users.updated_at.
- [ ] Add partial index on reset_token_hash WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now() and a regular index for general lookup.
- [ ] Add UNIQUE index on email_normalized.
- [ ] Add CHECK constraint for failed_attempts >= 0 and enforce NOT NULL/DEFAULT 0.
- [ ] Document allowed hash algorithms and migration notes in migrations/README.md.
- [ ] Add tests in tests/db/migrations/us_001_migration_test.sql to verify columns, defaults, uniqueness violation behavior, CHECK constraint behavior, and trigger updated_at behavior.

Notes / Provision to update: Update this checklist when migration naming conventions or DB extension policies (pgcrypto vs uuid-ossp) are confirmed in task_000.

## RULES:
- Effort: Estimated <= 8 hours (create migration, docs, and basic tests).
- Implementation Checklist contains 7 actionable items (<=8).
- Acceptance Criteria are taken from the parent User Story us_001.