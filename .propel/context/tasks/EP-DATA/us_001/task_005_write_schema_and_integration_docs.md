# Task - TASK_005

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is executed, Then a users table is created with these columns and types: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL DEFAULT ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the users table exists, When an insert is attempted with an email whose normalized form duplicates an existing normalized email, Then the DB rejects the insert with a unique-violation error due to a UNIQUE index on email_normalized and demonstration tests assert the expected error code.
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given a reset token generation flow, When the application stores reset_token_hash and reset_token_expiry, Then there exists an index on reset_token_hash to allow efficient lookup and the application enforces reset_token_expiry to be a future timestamp at time of insertion; tests verify lookup performance on seeded token records and expiry enforcement in application logic.
    - Given hash algorithm policy, When creating or updating records, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to the allowed set (document the allowed list in migration or application config) and password_hash column size/format supports storing modern algorithm outputs and legacy values; unit tests verify permitted and rejected hash_algo values.
- Edge Case:
    - What happens when a legacy password hash format is encountered?
        - Handling: Schema permits storing legacy formats by capturing the format in hash_algo and allowing salt to be nullable. The application must detect legacy hash_algo during authentication and perform migration-on-login (rehash into modern algorithm) in a separate story. The DB will accept the legacy record; migration is handled at application layer, not via destructive DB migration.
    - How does system handle concurrent updates to failed_attempts and lockout threshold races?
        - Handling: Application must use DB atomic increments or SELECT ... FOR UPDATE to avoid lost updates. The migration includes a CHECK constraint (failed_attempts >= 0) and tests simulate concurrent increments to validate correctness. If DB-level distributed concurrency issues appear in scale testing, a follow-up story will add optimistic concurrency control or centralized counters.
    - What happens when reset token hash collisions or race conditions occur?
        - Handling: The DB provides an index to look up tokens but the application is responsible for generating sufficiently random tokens, hashing them, and retrying generation on collision. Optionally implement a partial unique index on reset_token_hash for active (non-expired) tokens in DBs that support partial indexes; otherwise handle collisions in application logic with retries and logging for monitoring.

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
| Backend | Node.js (recommended runtime for docs/examples) | 18.x |
| Database | PostgreSQL | 12.x - 15.x (15.x recommended) |
| Library | Knex.js (migration examples), or SQL files for Flyway/Alembic | Knex v2.x / compatible SQL |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All code and examples in documentation MUST be compatible with the versions above. If the project uses a different stack (e.g., Python/Alembic), adapt SQL snippets and migration commands accordingly.

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
Create comprehensive schema and integration documentation describing the users table schema and constraints (as required by us_001), allowed hash algorithms and how they are represented, email normalization strategy, token lifecycle (generation, storage, expiry, lookup), and responsibilities between the database and application (including rehash-on-login behavior, token collision handling, atomic failed_attempts increment semantics). Produce clear examples (SQL snippets and app-level pseudocode), an integration checklist for application teams, test vectors to validate constraints, and actionable migration notes (extensions required, partial index guidance, rollback considerations).

## Dependent Tasks
- Task 000 — Confirm DB Platform & Migration Tooling (must confirm PostgreSQL version and allowed extensions)
- Task 001 — Create users table migration (SQL/knex/flyway skeleton) OR a minimally applied migration to derive examples (documentation references the migration file path)

## Impacted Components
- docs/db/users_schema.md (new)
- docs/db/users_integration.md (new)
- docs/snippets/migrations/us_001_users_table.sql (new example migration snippet)
- migrations/us_001_create_users_table.sql or migrations/xxxx_us_001_create_users_table.js (referenced; migration should exist from Task 001)
- backend/auth/README.md (modify or create to reference contract with DB for auth behavior)
- CI/CD migration step docs (.propel/build/ or CI config) — reference only

## Implementation Plan
- Collect canonical schema and constraints from us_001 and the Reasoning section. Confirm SQL dialect features available (partial index, enum type, gen_random_uuid()) from Task 000 before finalizing SQL snippets.
- Draft docs/db/users_schema.md:
  - Include column-by-column description, SQL types, defaults, nullability, CHECK constraints and rationale.
  - Provide concrete SQL CREATE TABLE example compatible with PostgreSQL (include CREATE EXTENSION if necessary).
  - Include index definitions: unique index on email_normalized, index (or partial unique index) on reset_token_hash, optional GIN index for roles when jsonb chosen.
  - Include enum or check for hash_algo with an allowed list example and guidance to choose DB-level vs application-level enforcement.
  - Document triggers/functions: updated_at trigger snippet and optional helper functions (atomic increment examples).
- Draft docs/db/users_integration.md:
  - Describe email normalization strategy (trim, unicode normalization NFKC, lower-case, collapse whitespace) with deterministic pseudocode and example implementation in JS/Python.
  - Define allowed hash_algorithm list (recommended: argon2id, bcrypt, pbkdf2, legacy_sha256) with examples of hash storage formats and salt usage rules.
  - Document token lifecycle: generation (entropy recommendations), hashing (store only H(token)), storage, expiry semantics, lookup flow, and collision retry policy.
  - State application responsibilities: rehash-on-login process (detect legacy hash_algo, verify, rehash with modern algorithm, atomically update record), reset token single-use verification and invalidation, atomic failed_attempts increment flow (SQL examples), lockout handling.
  - Provide test recipes for constraint validation, concurrency tests, and EXPLAIN suggestions for reset token lookup performance.
- Provide docs/snippets/migrations/us_001_users_table.sql as a ready-to-adapt SQL file with CREATE TABLE and indexes (and DROP TABLE for rollback).
- Provide a short integration checklist (QA/devops) with concrete steps for app devs and operators to follow when implementing auth flows against this schema.
- Review with DB/migration owner (confirm any organization-specific naming conventions, extension policies).
- Finalize and push files; mention locations for follow-up stories (rehash-on-login implementation, app-level validations, test harnesses).

## Current Project State
- app/ (frontend placeholder)
- server/ (backend placeholder)
- migrations/ (existing migration folder; may contain prior migrations)
- docs/ (documentation root)
  - docs/README.md
- .propel/context/tasks/EP-DATA/us_001/us_001.md (story file present)
- CI/ (CI pipeline configs)

(These are placeholders to be updated during execution depending on actual repo structure discovered in Task 000.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | docs/db/users_schema.md | Authoritative schema doc: column definitions, constraints, SQL CREATE TABLE example, index definitions, rationale. |
| CREATE | docs/db/users_integration.md | Integration contract: email normalization, allowed hash algorithms, token lifecycle, app responsibilities (rehash-on-login, token handling), test recipes, examples. |
| CREATE | docs/snippets/migrations/us_001_users_table.sql | Example PostgreSQL migration SQL (CREATE TABLE + CREATE INDEX + DROP TABLE rollback + extension notes + partial index example). |
| MODIFY | docs/README.md | Add links to the new schema and integration documentation (append reference section). |
| MODIFY | .propel/context/tasks/EP-DATA/us_001/us_001.md | Add links to created docs and snippet files for traceability (non-destructive editorial update). |

## External References
- PostgreSQL documentation - CREATE INDEX & partial indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL documentation - pgcrypto / gen_random_uuid(): https://www.postgresql.org/docs/current/pgcrypto.html
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- Argon2 RFC / reference: https://www.ietf.org/archive/id/draft-irtf-cfrg-argon2-03.html
- bcrypt reference & recommended parameters: https://en.wikipedia.org/wiki/Bcrypt
- NFKC Unicode normalization: https://unicode.org/reports/tr15/
- Example: EXPLAIN ANALYZE in Postgres for index performance: https://www.postgresql.org/docs/current/using-explain.html

## Build Commands
- For Node.js / Knex examples:
  - npm ci
  - npm run migrate (run the project's migration runner; adapt name if different)
  - npm run test
- Direct PostgreSQL (for example snippets):
  - psql $DATABASE_URL -f docs/snippets/migrations/us_001_users_table.sql
- CI: Ensure migration step runs as part of pipeline (.propel/build/ or project CI configuration)

## Implementation Validation Strategy
- [ ] Unit tests pass (where unit tests are added for normalization functions and any helper code).
- [ ] Integration tests pass (migration apply/rollback in a test DB).
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A).
- [ ] Run /analyze-ux to validate wireframe alignment (N/A).
- [ ] Prompt templates validated with test inputs (N/A).
- [ ] Guardrails tested for input sanitization and output validation (N/A).
- [ ] Fallback logic tested with low-confidence/error scenarios (N/A).
- [ ] Token budget enforcement verified (N/A).
- [ ] Audit logging verified (no PII in logs) (N/A).

## Implementation Checklist
- Effort estimate: 6-8 hours (documentation + SQL snippets + review)
- [ ] Draft docs/db/users_schema.md with concrete SQL CREATE TABLE example and indexes (2.0h)
- [ ] Draft docs/db/users_integration.md describing normalization, hash algo list, token lifecycle, and app responsibilities with pseudocode (2.5h)
- [ ] Produce docs/snippets/migrations/us_001_users_table.sql (Postgres-compatible) including rollback and partial index examples (1.0h)
- [ ] Update docs/README.md and .propel/context/tasks/EP-DATA/us_001/us_001.md to reference new docs (0.5h)
- [ ] Run quick verification: apply example SQL to a local Postgres instance (docker/postgres), check DDL, indexes, and constraints (0.5h)
- [ ] Review with DB/migration owner and incorporate feedback (0.5h)
- [ ] Finalize and push changes; mark task complete only after review feedback is resolved (0.5h)

Notes:
- Keep checklist <= 8 items and total effort <= 8 hours.
- This task is documentation-focused; migration execution and application code changes are dependent tasks handled elsewhere.
- The documentation must explicitly call out where the application MUST enforce behavior vs. what the DB enforces (e.g., DB-level unique index vs. app-level hash_algo validation if DB-level enum is not used).

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Effort estimate is <=8 hours.
- Acceptance Criteria and Edge Cases copied from the parent User Story us_001.