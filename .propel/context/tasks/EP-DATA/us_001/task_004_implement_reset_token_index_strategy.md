# Task - task_004

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
| Backend | Migration SQL (PostgreSQL target) | PostgreSQL 12+ (recommend 13/14) |
| Database | PostgreSQL | 12.x / 13.x / 14.x |
| Library | Migration tooling (SQL-based migrations) | Flyway / native SQL migrations / psql |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All SQL and migration artifacts MUST be compatible with PostgreSQL 12+ and the project's chosen migration runner. Partial indexes and expression WHERE clauses assume PostgreSQL support.

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
Design and implement an index variant for users.reset_token_hash to support efficient lookup for token verification and single-use tokens. Evaluate and choose the best index strategy for PostgreSQL: either a partial index that only indexes active (non-expired, non-null) reset_token_hash values, or a composite index that includes reset_token_expiry. Implement the migration SQL (CREATE INDEX ... / DROP INDEX ...) with a safe rollback (down) that drops the index. Provide a small verification SQL/test to assert index usage (EXPLAIN ANALYZE) and ensure index creation is safe for large tables (CONCURRENTLY considerations) and idempotent in migration-runner context.

## Dependent Tasks
- Confirm DB platform and migration tooling support: Task 000 (prerequisite) — validate PostgreSQL version and allowed extensions.
- Ensure users table migration exists and is applied or included in same release: Task 001 (users table creation per us_001).
- Add migration-runner CI step to apply/rollback migrations in CI (Task 010) — for verification (recommended prior to production run).

## Impacted Components
- Server/migrations/ (new migration SQL file(s) for index create/drop)
- Server/tests/db/ (new integration test to verify index existence and EXPLAIN usage)
- Docs/internal/ (migration notes and index rationale)
- Migration runner configuration (may be modified if specifying CONCURRENTLY usage is controlled)

## Implementation Plan
- Choose index variant:
  - Preferred: Partial index on reset_token_hash for active tokens: CREATE INDEX CONCURRENTLY idx_users_reset_token_hash_active ON users (reset_token_hash) WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now();
  - Alternative (if partial index not desired): Composite index on (reset_token_hash, reset_token_expiry) to support queries that filter both columns.
  - Decide based on expected query patterns: verification usually queries WHERE reset_token_hash = $1 AND reset_token_expiry > now().
- Create migration SQL file:
  - Up: CREATE INDEX CONCURRENTLY ... (wrap in transaction policy depending on migration runner; provide non-CONCURRENTLY fallback with comment).
  - Down: DROP INDEX CONCURRENTLY IF EXISTS ...
  - Provide idempotency guards and instructions for runners that don't support CONCURRENTLY inside a transaction (most runners run each migration in a transaction — add guidance).
- Add a small verification script/test:
  - Seed minimal rows (active/expired) and run EXPLAIN ANALYZE for the token lookup query to assert the index is used.
  - Add a rollback verification that DROP INDEX removes the index and EXPLAIN shows sequential scan.
- Add migration metadata and docs:
  - Document the reason for partial vs composite choice and operations for backfill or reindex if needed.
- Review and test on a staging DB with representative data volume:
  - Validate that index creation CONCURRENTLY is recommended for production to avoid locks.
- Provide CI steps to run migration up/down and the verification SQL.

## Current Project State
- app/ (frontend — placeholder)
- Server/
  - migrations/ (existing migration SQL files; users table migration should exist or be applied)
  - tests/
    - db/ (existing DB tests)
  - src/ (backend application code)
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story location)
Note: Update this structure during execution if dependent tasks add/modify files.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | Server/migrations/V20260406__create_idx_users_reset_token_hash_active.sql | Migration SQL that creates a partial index on users.reset_token_hash for active tokens (and drops it on rollback). |
| CREATE | Server/tests/db/test_reset_token_index_usage.sql | SQL test that seeds sample rows and runs EXPLAIN ANALYZE for token lookup queries to assert index usage. |
| CREATE | Server/docs/migrations/README_reset_token_index.md | Short rationale and rollout notes (CONCURRENTLY usage, fallback, expected query patterns). |
| MODIFY | Server/migrations/README.md | Add reference to the new migration file and any runner-specific guidance (CONCURRENTLY caveat). |
| MODIFY | .propel/context/tasks/EP-DATA/us_001/us_001.md | Add a small note referencing that index migration task is implemented in task_004 (traceability update). |

## External References
- PostgreSQL Partial Indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- CREATE INDEX CONCURRENTLY: https://www.postgresql.org/docs/current/sql-createindex.html
- EXPLAIN and EXPLAIN ANALYZE: https://www.postgresql.org/docs/current/using-explain.html
- Discussion on partial vs composite index patterns: https://wiki.postgresql.org/wiki/Indexing_Strategies

## Build Commands
- Run SQL migration with psql (example):
  - psql "$DATABASE_URL" -f Server/migrations/V20260406__create_idx_users_reset_token_hash_active.sql
- Rollback (if migration runner supports down script execution):
  - psql "$DATABASE_URL" -f Server/migrations/V20260406__create_idx_users_reset_token_hash_active.sql --run-down
- Run verification test:
  - psql "$DATABASE_URL" -f Server/tests/db/test_reset_token_index_usage.sql
- CI: add migration apply + test steps to existing pipeline. See ../.propel/build/ for project-specific build scripts.

## Implementation Validation Strategy
- [ ] Unit tests pass (where applicable for migration test harness)
- [ ] Integration tests pass: run test_reset_token_index_usage.sql against a test DB
- [ ] EXPLAIN ANALYZE for token lookup shows index scan for active token query
- [ ] Rollback test: run migration down and verify index is removed and EXPLAIN ANALYZE falls back to sequential scan
- [ ] Migration is safe to run in production (CONCURRENTLY used or documented fallback if not supported)
- [ ] Documentation updated with rationale and operational notes

## Implementation Checklist
- Effort: <= 8 hours (task scoped to index creation, tests, docs)
- [ ] Choose index variant (partial active-index vs composite) and document decision (create README_reset_token_index.md)
- [ ] Create migration SQL file Server/migrations/V20260406__create_idx_users_reset_token_hash_active.sql implementing CREATE INDEX (with CONCURRENTLY variant and DROP INDEX on rollback)
- [ ] Add verification SQL Server/tests/db/test_reset_token_index_usage.sql that seeds rows and asserts index usage via EXPLAIN ANALYZE
- [ ] Run migration up against a staging DB and validate EXPLAIN shows index usage for queries: SELECT ... WHERE reset_token_hash = $1 AND reset_token_expiry > now()
- [ ] Run migration down (rollback) and validate index removal and query plan changes
- [ ] Add migration notes to Server/docs/migrations/README_reset_token_index.md and update Server/migrations/README.md with runner caveats (CONCURRENTLY)
- [ ] Push changes and include migration apply + verification step in CI (or document manual steps if CI change is out of scope)

## RULES:
- Implementation Checklist contains 7 items (<=8)
- Effort <= 8 hours
- Acceptance Criteria taken from the parent User Story (us_001) and included above