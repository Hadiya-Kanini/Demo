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

> **Wireframe Status Legend:**
> - **AVAILABLE**: Local file exists at specified path
> - **PENDING**: UI-impacting task awaiting wireframe (provide file or URL)
> - **EXTERNAL**: Wireframe provided via external URL
> - **N/A**: Task has no UI impact
>
> If UI Impact = No, all design references should be N/A

### **CRITICAL: Wireframe Implementation Requirement (UI Tasks Only)**
**IF Wireframe Status = AVAILABLE or EXTERNAL:**
- **MUST** open and reference the wireframe file/URL during UI implementation
- **MUST** match layout, spacing, typography, and colors from the wireframe
- **MUST** implement all states shown in wireframe (default, hover, focus, error, loading)
- **MUST** validate implementation against wireframe at breakpoints: 375px, 768px, 1440px
- Run `/analyze-ux` after implementation to verify pixel-perfect alignment

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

> **AI Impact Legend:**
> - **Yes**: Task involves LLM integration, RAG pipeline, prompt engineering, or AI infrastructure
- **No**: Task is deterministic (FE/BE/DB only)
>
> If AI Impact = No, all AI references should be N/A

### **CRITICAL: AI Implementation Requirement (AI Tasks Only)**
**IF AI Impact = Yes:**
- **MUST** reference prompt templates from Prompt Template Path during implementation
- **MUST** implement guardrails for input sanitization and output validation
- **MUST** enforce token budget limits per AIR-O01 requirements
- **MUST** implement fallback logic for low-confidence responses
- **MUST** log all prompts/responses for audit (redact PII)
- **MUST** handle model failures gracefully (timeout, rate limit, 5xx errors)

## Task Overview
Implement a reversible (UP/DOWN) PostgreSQL-compatible SQL migration to create the canonical users table for us_001. The migration will:
- Create the users table with the specified columns, types, defaults and constraints.
- Add UNIQUE constraint/index on email_normalized.
- Add CHECK constraint failed_attempts >= 0.
- Enforce allowed hash algorithms via an enum type OR a CHECK constraint (document choice).
- Add index(es) for reset_token_hash (regular index plus a partial index for active tokens).
- Provide created_at and updated_at defaults and implement an updated_at trigger (function + trigger) that sets updated_at = now() on UPDATE.
- Ensure migration is reversible: DOWN migration must remove table, triggers, functions, types, and any created extension artifacts in the correct order.
- Include short documentation in migrations/README.md noting allowed hash algorithms and rationale for design choices.

Purpose: Provide the canonical DB schema for user accounts to support authentication, lockout, and password-reset flows with safe defaults and reversible migration.

## Dependent Tasks
- task_000 - Confirm DB platform & migration tooling (Postgres version, migration runner, allowed extensions)
- (Ops) Ensure DB role used by migrations has privileges to CREATE EXTENSION (if gen_random_uuid() is used) or adjust UUID generation method.

## Impacted Components
- migrations/V002__create_users_table.sql (new migration file to create users table, up/down)
- migrations/README.md (document allowed hash algorithms and migration notes)
- tests/db/migrations/us_001_migration_test.sql (migration verification scripts/SQL for CI)
- db/sql/functions/trigger_set_updated_at.sql (trigger function created/dropped by migration if not inlined)

## Implementation Plan
- Create migration SQL file (Flyway-style naming) that implements both UP and DOWN steps in a single file following project conventions. Use transactional DDL (Postgres supports DDL in transactions).
- UP steps:
  - Create extension pgcrypto if allowed (CREATE EXTENSION IF NOT EXISTS pgcrypto) to support gen_random_uuid(); if extension creation is not permitted, fall back to uuid-ossp or require application to set id on insert. Implement id UUID DEFAULT gen_random_uuid() guarded by CREATE EXTENSION IF NOT EXISTS pgcrypto.
  - Create an enum type users_hash_algo AS (...) OR implement a CHECK constraint restricting hash_algo to the allowed list ('argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256'). Prefer CHECK if project guidelines avoid creating many enum types; include clear documentation in migrations/README.md explaining the choice.
  - Create users table with the specified columns:
    - id UUID PRIMARY KEY DEFAULT gen_random_uuid()
    - email text NOT NULL
    - email_normalized text NOT NULL
    - password_hash text NOT NULL
    - hash_algo text NOT NULL (or users_hash_algo)
    - salt text NULL
    - roles jsonb NOT NULL DEFAULT '["user"]' (use jsonb for flexibility; document rationale)
    - failed_attempts integer NOT NULL DEFAULT 0
    - locked_at timestamptz NULL
    - reset_token_hash text NULL
    - reset_token_expiry timestamptz NULL
    - created_at timestamptz NOT NULL DEFAULT now()
    - updated_at timestamptz NOT NULL DEFAULT now()
  - Add UNIQUE INDEX ON email_normalized (CREATE UNIQUE INDEX CONCURRENTLY not used inside transaction by default; use CREATE UNIQUE INDEX to ensure atomicity in migration — consider CONCURRENTLY in a follow-up rolling migration if table is large and downtime concerns exist).
  - Add CHECK constraint failed_attempts >= 0.
  - Add index on reset_token_hash for general lookup.
  - Add partial index on reset_token_hash WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now() to optimize active-token lookup. Note: partial index uses runtime predicate now() at index creation time; to ensure correctness use predicate reset_token_expiry > now() — acknowledge that entries expire relative to time and partial index membership will change over time (acceptable for optimizing active-token lookups).
  - Add trigger function trigger_set_updated_at() that sets NEW.updated_at = now() and returns NEW; then add BEFORE UPDATE trigger on users to execute this function.
  - Add CHECK or ENUM constraint limiting hash_algo to allowed values; include a sensible comment on the constraint for future maintainers.
  - Add CHECK constraints or triggers/documentation to ensure reset_token_expiry > now() at insertion time cannot be enforced purely by a CHECK on column vs now() because now() is stable per statement; implement application-side enforcement and include database-level protection via a CHECK (reset_token_expiry IS NULL OR reset_token_expiry > now()) to catch obvious mistakes.
- DOWN steps:
  - Drop trigger on users.
  - Drop trigger function if created by this migration.
  - Drop indexes and constraints introduced.
  - Drop users table.
  - Drop enum type if created by migration.
  - Optionally drop extension only if this migration created it (careful with environment policies — prefer to NOT drop global extensions that may be shared).
- Tests & docs:
  - Add migrations/README.md documenting allowed hash algorithms, rationale for roles jsonb vs text[], notes about extension usage (pgcrypto vs uuid-ossp), and migration roll-back caveats.
  - Add tests/tests/db/migrations/us_001_migration_test.sql to validate table/columns/defaults, uniqueness on email_normalized, check constraint prevents negative failed_attempts, indexes existence (using pg_indexes), trigger updated_at behavior (update row and assert updated_at changed), and that DOWN removes artifacts.
- Idempotency & safety:
  - Use IF NOT EXISTS guards for CREATE EXTENSION, CREATE TYPE IF NOT EXISTS (if using type), and CREATE INDEX IF NOT EXISTS where supported to make repeated dry-runs safer.
  - Be explicit about not using CREATE INDEX CONCURRENTLY in a transaction, and choose non-concurrent creation for test environments; document if production requires CONCURRENTLY and a follow-up non-transactional migration.
- Deliverables:
  - migrations/V002__create_users_table.sql
  - db/sql/functions/trigger_set_updated_at.sql (if extracting function)
  - tests/db/migrations/us_001_migration_test.sql
  - migrations/README.md update documenting choices and allowed hash_algorithms
- Timebox: Entire task scoped to <= 8 hours (create migration file, function, README updates, and basic test SQL).

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
| CREATE | migrations/V002__create_users_table.sql | SQL migration implementing UP (create extension/type if needed, create table, indexes, constraints, trigger function or reference) and DOWN (drop trigger, drop function, drop indexes, drop table, drop type if created). |
| CREATE | db/sql/functions/trigger_set_updated_at.sql | SQL for trigger function that sets NEW.updated_at = now() on UPDATE (if extracting function out of migration file). |
| CREATE | tests/db/migrations/us_001_migration_test.sql | SQL scripts used by CI to verify migration applied: column checks, constraint checks, uniqueness test, failed_attempts check, trigger behavior. |
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
- [ ] Create migrations/V002__create_users_table.sql implementing UP and DOWN following plan (include CREATE EXTENSION IF NOT EXISTS, CREATE TYPE or CHECK, table, constraints, indexes, trigger).
- [ ] Add trigger function file db/sql/functions/trigger_set_updated_at.sql or inline function in migration and wire up trigger on users.updated_at.
- [ ] Add partial index on reset_token_hash WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now() and a regular index for general lookup.
- [ ] Add UNIQUE index on email_normalized.
- [ ] Add CHECK constraint for failed_attempts >= 0 and enforce NOT NULL/DEFAULT 0.
- [ ] Document allowed hash algorithms and migration notes in migrations/README.md.
- [ ] Add tests in tests/db/migrations/us_001_migration_test.sql to verify columns, defaults, uniqueness violation behavior, CHECK constraint behavior, and trigger updated_at behavior.

Notes / Provision to update: Update this checklist when migration naming conventions or DB extension policies (pgcrypto vs uuid-ossp) are confirmed in task_000.

## RULES:
- Implementation Checklist must have <=8 items
- Effort must be <=8 hours
- Be specific and actionable — no vague descriptions
- Expected Changes table must list concrete file paths (CREATE/MODIFY/DELETE)
- Acceptance Criteria must come from the parent User Story us_001

- Effort: Estimated <= 8 hours (create migration, docs, and basic tests).
- Implementation Checklist contains 7 actionable items (<=8).
- Acceptance Criteria are taken from the parent User Story us_001.