# Task - TASK_003

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-DATA/us_001/us_001.md
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is executed, Then a users table is created with these columns and types including updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts).
    - Given the schema, When failed_attempts is updated, Then there is a CHECK constraint failed_attempts >= 0 so negative values cannot be persisted.
    - Given a database migration runner with valid credentials, When this task's migration is applied, Then the migration creates trigger/function artifacts to maintain updated_at automatically and provides helper(s) demonstrating atomic increment semantics and includes a down migration to remove those artifacts.
- Edge Case:
    - How does the system handle concurrent updates to failed_attempts and locked_at? - The schema supports atomic increment semantics; application must perform DB-level atomic UPDATEs (e.g., UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and use row-level locking where necessary. This task provides a helper demonstrating atomic UPDATE; concurrency tests should validate behavior.
    - What happens when a legacy password hash format is encountered? - N/A to this task beyond ensuring updated_at/failed_attempts helpers don't break legacy records; schema-level support for legacy formats is handled elsewhere in us_001.
    - What happens when reset token hash collisions or race conditions occur? - N/A to this task (this task focuses on triggers and helper functions).

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
| Backend | SQL migration scripts (plain SQL) | N/A |
| Database | PostgreSQL | 14.x (>= 12 recommended) |
| Library | psql client / migration runner (project default) | N/A |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the stated versions.

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
Create a database migration that:
- Adds (or ensures) a trigger function set_updated_at() that sets NEW.updated_at = now() on row updates and installs a trigger on the users table to invoke it BEFORE UPDATE.
- Adds helper function increment_failed_attempts(uid UUID) that atomically increments failed_attempts and returns the new value (demonstrates correct atomic UPDATE semantics).
- Adds (if missing) a CHECK constraint users_failed_attempts_nonnegative ensuring failed_attempts >= 0.
- Provide a reversible down migration that drops the trigger and functions and optionally removes the constraint if added by this migration.
- Include short SQL test/demo snippets/comments showing recommended usage (atomic UPDATE usage and RETURNING usage) and a small integration test SQL script to validate the helper.

Purpose: ensure updated_at is reliably maintained and provide a DB-side helper to illustrate an atomic increment pattern required by the broader user story (us_001). Provide up and down migration artifacts for deployment.

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/us_001.md (users table migration must exist or be applied prior to installing triggers/helpers)
- Task 000 / discovery confirming PostgreSQL version and migration tooling (must have permissions to create functions and triggers)
- If users table was created in a prior migration, ensure that migration has been applied (migration runner state)

## Impacted Components
- New SQL migration files in the repository migrations/ (up/down)
- DB functions:
  - public.set_updated_at()
  - public.increment_failed_attempts(uid UUID)
- Trigger:
  - trigger on users table: trg_set_updated_at
- Optional: Constraint users_failed_attempts_nonnegative added to users table if absent

## Implementation Plan
- Create an UP migration SQL file that:
  1. Creates a plpgsql function public.set_updated_at() RETURN TRIGGER which sets NEW.updated_at = now() and returns NEW.
  2. Creates the trigger trg_set_updated_at BEFORE UPDATE ON users FOR EACH ROW WHEN (OLD.* IS DISTINCT FROM NEW.*) or simply execute on every update (simpler: always set NEW.updated_at = now()).
  3. Creates a plpgsql function public.increment_failed_attempts(p_id UUID) RETURNS integer that atomically runs UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = p_id RETURNING failed_attempts INTO result; returns result.
  4. Adds CHECK constraint users_failed_attempts_nonnegative if not already present: failed_attempts >= 0.
  5. Add GRANT/SECURITY considerations comment: functions marked as SECURITY DEFINER only if project policy allows; by default create as normal functions.
  6. Add demonstration SQL snippet (as comment) showing how to use increment_failed_attempts and the equivalent UPDATE ... RETURNING approach.
- Create a DOWN migration SQL file that:
  1. Drops the trigger trg_set_updated_at if exists.
  2. Drops function public.set_updated_at() if exists.
  3. Drops function public.increment_failed_attempts(UUID) if exists.
  4. Optionally removes the constraint users_failed_attempts_nonnegative if it was created by this migration (guarded with IF EXISTS and check for constraint owner).
- Create a tests SQL file (integration script) that:
  1. Inserts a test user (if not present) or uses an existing seeded user.
  2. Calls increment_failed_attempts concurrently (or in quick succession) and asserts returned values/increment semantics.
  3. Verifies updated_at changes on update and that the trigger sets it to a newer timestamp.
- Write clear comments in migration files describing assumptions and rollback semantics.
- Run migration apply/rollback locally in dev DB to validate.

## Current Project State
- app/ (frontend) — N/A for this task
- Server/ or db/ migration folder exists per project conventions. (Replace with repository actual structure during implementation.)
- Assume there is an existing migration that created users table per us_001 or it will be applied before this task.

Example (placeholder) project tree context:
- migrations/
  - 2026xxxx_us_001_create_users_table.up.sql (exists or will exist)
  - 2026xxxx_us_001_create_users_table.down.sql
  - migrations/... (new files to be added by this task)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | migrations/20260406_add_updated_at_trigger_and_helpers.up.sql | UP migration: creates set_updated_at() trigger function, trg_set_updated_at trigger on users, increment_failed_attempts(UUID) helper, and adds CHECK constraint users_failed_attempts_nonnegative (guarded if not exists). Includes comments and usage samples. |
| CREATE | migrations/20260406_add_updated_at_trigger_and_helpers.down.sql | DOWN migration: drops trigger, drops functions, and removes CHECK constraint if created. |
| CREATE | db/tests/sql/test_increment_failed_attempts.sql | SQL script to validate increment_failed_attempts, demonstrate atomic increments and trigger-updated updated_at behavior (to be run in dev CI). |
| MODIFY | README.md (database section) | Add short documentation snippet describing the new functions, how to use increment helper, and notes about security/permissions and rollback. |

## External References
- PostgreSQL docs: CREATE FUNCTION (https://www.postgresql.org/docs/current/sql-createfunction.html)
- PostgreSQL docs: Triggers (https://www.postgresql.org/docs/current/sql-createtrigger.html)
- PostgreSQL docs: CHECK constraint (https://www.postgresql.org/docs/current/ddl-constraints.html)
- Example atomic increment pattern: UPDATE ... SET failed_attempts = failed_attempts + 1 WHERE id = $1 RETURNING failed_attempts;

## Build Commands
- Apply migration (project-specific migration runner). Example using psql:
  - psql "postgresql://user:pass@localhost:5432/dbname" -f migrations/20260406_add_updated_at_trigger_and_helpers.up.sql
- Rollback migration:
  - psql "postgresql://user:pass@localhost:5432/dbname" -f migrations/20260406_add_updated_at_trigger_and_helpers.down.sql
- Run tests:
  - psql "postgresql://user:pass@localhost:5432/dbname" -f db/tests/sql/test_increment_failed_attempts.sql

(Refer to project migration tooling and CI build steps at ../.propel/build/)

## Implementation Validation Strategy
- [ ] Unit tests pass (SQL test script run locally)
- [ ] Integration tests pass (test_increment_failed_attempts.sql verifies concurrency/sequence)
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px — N/A
- [ ] Run /analyze-ux to validate wireframe alignment — N/A
- [ ] Prompt templates validated with test inputs — N/A
- [ ] Guardrails tested for input sanitization and output validation — N/A
- [ ] Fallback logic tested with low-confidence/error scenarios — N/A
- [ ] Token budget enforcement verified — N/A
- [ ] Audit logging verified (no PII in logs) — N/A

## Implementation Checklist
- Effort estimate: 6 hours
- [ ] Create UP migration SQL file implementing set_updated_at trigger function, trg_set_updated_at trigger, increment_failed_attempts(UUID) helper, and CHECK constraint (if absent).
- [ ] Create DOWN migration SQL file that safely reverts the changes (drop trigger, drop functions, drop constraint if created).
- [ ] Create a small SQL test script (db/tests/sql/test_increment_failed_attempts.sql) demonstrating atomic increments and updated_at behavior; run locally against dev DB.
- [ ] Update README.md (database section) with usage notes and permissions/security considerations for created functions.
- [ ] Run migration apply and rollback in local dev DB and verify no residual objects remain after rollback.
- [ ] Verify that increment_failed_attempts() returns the incremented value and that multiple rapid calls increment monotonically (run sample concurrency test).
- [ ] Validate that updated_at is updated on row updates and the trigger does not error on legacy rows (NULLable fields).
- [ ] Commit migration and test artifacts and open PR with migration SQL and test script for review.

## RULES:
- Implementation Checklist has <=8 items
- Effort <=8 hours
- Acceptance Criteria are from the parent User Story (us_001) and focused on updated_at and atomic increment semantics.