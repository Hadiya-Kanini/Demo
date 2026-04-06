# Task - [TASK_006]

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
| Backend | N/A | N/A |
| Database | PostgreSQL | 14.x - 15.x (prefer 15) |
| Library | psql / pgbench / pg_stat_statements | psql v14+ |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: This task assumes PostgreSQL as the primary DB (partial indexes supported). If another RDBMS is in use, add a DB capability detection step and follow alternate index/pruning approaches.

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
Validate the reset_token_hash index performance and produce a safe, operational pruning plan. This includes seeding realistic test data (active tokens, expired tokens, many nulls), running EXPLAIN ANALYZE for common lookup queries used in password reset flows, confirming which index variant (simple index, partial index on IS NOT NULL, or application-maintained active flag + partial index) gives the best and correct behavior, verifying whether a partial index that filters by expiry (dynamic predicate) is viable on the current DB, and producing SQL snippets and operational recommendations to prune expired tokens safely and efficiently (batched deletes, appropriate vacuum/reindex guidance, scheduling, and monitoring metrics).

## Dependent Tasks
- Artefacts/Tasks:
  - task_001_db_create_users_table (migration that creates users table with reset_token_hash and reset_token_expiry)
  - DB instance accessible for running tests (development or CI ephemeral DB)
  - Ensure pg_stat_statements (optional) is enabled on test DB for monitoring (recommended)

## Impacted Components
- DB migration: migrations/us_001_create_users_table.sql (already created by dependent task)
- Scripts:
  - scripts/db/seed_reset_token_data.sql (new)
  - scripts/db/validate_reset_token_index.sql (new)
- Docs:
  - docs/db/reset_token_index_pruning.md (new)
- Tests:
  - tests/db/test_reset_token_index_performance.sql (new)
- CI:
  - .ci/db-migration-validate.yml (optional modification if adding an automated validation job)

## Implementation Plan
- Prepare environment:
  - Confirm a test Postgres instance and run the us_001 migration (dependent task) to ensure users table exists.
- Seed realistic test data:
  - Create a seed script that inserts:
    - N = 10k - 100k users with reset_token_hash = NULL (common case)
    - M = 1k active tokens (reset_token_hash set, reset_token_expiry in future)
    - K = 10k expired tokens (reset_token_hash set, reset_token_expiry in past)
  - Include various hash lengths/styles to model real token hash values.
- Index variants:
  - Ensure the baseline index exists: CREATE INDEX idx_users_reset_token_hash ON users(reset_token_hash);
  - Create alternative index variant for testing:
    - Partial index on reset_token_hash IS NOT NULL:
      CREATE INDEX idx_users_reset_token_hash_notnull ON users(reset_token_hash) WHERE reset_token_hash IS NOT NULL;
    - (Document why predicate using now() is not acceptable in PG and include approach with application-maintained boolean)
    - Optional: test an index on (reset_token_hash) WHERE reset_token_active = true after adding a temporary column and populating it for test.
- Performance validation:
  - Run representative queries with parameterized token hash lookup:
    - SELECT id FROM users WHERE reset_token_hash = $1 AND reset_token_expiry > now() LIMIT 1;
    - SELECT id FROM users WHERE reset_token_hash = $1 LIMIT 1; (fallback)
  - For each query and index variant run EXPLAIN (ANALYZE, BUFFERS, VERBOSE) and record:
    - Whether index scan is used
    - Actual timing (median of multiple runs)
    - I/O / buffer usage
  - Validate planner chooses the intended index when active tokens are sparse and when they are numerous.
- Partial-index viability:
  - Validate that predicates using now()/current_timestamp are not permitted for stable/immutable requirements in index predicates. If DB forbids it, document and test alternative:
    - Add reset_token_active boolean (maintained by application or trigger) and build partial index on that column to represent "active tokens".
- Pruning plan:
  - Provide safe SQL snippets:
    - Batched deletion: DELETE FROM users WHERE reset_token_expiry IS NOT NULL AND reset_token_expiry < now() LIMIT 1000; (wrapped in loop transaction or use pg_sleep to throttle) — plus example PL/pgSQL function for batched pruning.
    - Use safe VACUUM/ANALYZE guidance and estimate cost.
    - Show how to maintain reset_token_active boolean if adopted (update statement for toggle).
  - Estimate performance and recommended schedule (e.g., nightly off-peak job with batching).
- Documentation and artifacts:
  - Produce scripts, test outputs (EXPLAIN ANALYZE captures), and recommendations in docs/db/reset_token_index_pruning.md.
  - Add tests to tests/db/ that can be run in CI to validate index choice on a scaled dataset (smaller scale) and ensure pruner SQL behaves as expected.
- Deliverables:
  - SQL scripts: seed, validate/explain, pruning functions
  - Documentation and PR with findings and recommended index+pruning approach

## Current Project State
- app/ (placeholder - not modified by this task)
- server/ (placeholder)
- migrations/
  - migrations/us_001_create_users_table.sql (assumed created by dependent task)
- scripts/
  - (new files to be created under scripts/db/)
- docs/
  - (new file under docs/db/)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | scripts/db/seed_reset_token_data.sql | SQL script to seed users test data: many NULL tokens, active tokens (expiry in future), expired tokens (expiry in past). Parameterize counts for local/CI runs. |
| CREATE | scripts/db/validate_reset_token_index.sql | SQL script that creates test index variants (non-partial, partial on IS NOT NULL, optional temp active-flag index), runs EXPLAIN ANALYZE queries, and writes results to stdout / logfile. |
| CREATE | scripts/db/prune_reset_tokens.sql | SQL snippets and a PL/pgSQL batched prune function (safe delete by date, with configurable batch size and sleep interval). |
| CREATE | docs/db/reset_token_index_pruning.md | Document explaining test results, index recommendations (including why now() cannot be used in index predicate), pruning schedule, and operational guidance (VACUUM, REINDEX, monitoring). Include recommended SQL snippets for production. |
| CREATE | tests/db/test_reset_token_index_performance.sql | Test script (can be run in CI) that verifies the lookup query uses an index and that pruning function deletes expired tokens as expected on a small dataset. |
| MODIFY | .propel/context/tasks/EP-DATA/us_001/implementation_notes.md | Add brief summary pointing to these scripts and recommended next steps (if file exists; create if missing). |

## External References
- PostgreSQL Partial Indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL EXPLAIN and ANALYZE: https://www.postgresql.org/docs/current/using-explain.html
- Postgres VACUUM/Autovacuum: https://www.postgresql.org/docs/current/vacuum.html
- Discussion: immutability and index predicate functions (why now() is not allowed as immutable): https://www.postgresql.org/docs/current/functions-datetime.html#FUNCTIONS-DATETIME-TABLE
- Example batched delete pattern: https://wiki.postgresql.org/wiki/Deleting_large_objects_efficiently

## Build Commands
- Run migration (dependent task must be completed first) using project migration runner. Example placeholder commands:
  - psql "$PG_CONN" -f migrations/us_001_create_users_table.sql
  - psql "$PG_CONN" -f scripts/db/seed_reset_token_data.sql
  - psql "$PG_CONN" -f scripts/db/validate