# Task - task_008

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
| Backend | Node.js (server-side guidance; language-agnostic acceptance) | 18.x |
| Database | PostgreSQL | 15.x |
| Library | Knex (migration helper) or SQL migration runner (project-specific) | 2.x / project runner |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and documentation MUST be compatible with the versions above. If the project uses a different migration runner (Flyway, Liquibase, Goose, Rails migrations), adapt SQL and run commands accordingly.

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
Publish the canonical DDL for the users table, document all constraints/indexes and the rationale behind design decisions (allowed hash algorithms, salt semantics, roles type choice, failed_attempts constraint, reset token indexing strategy including partial-index tradeoffs), describe token collision handling and retry policy, outline pruning schedule for expired tokens, provide a migration runbook (how to apply/rollback/verify migrations), and enumerate expected DB error codes and how the application should interpret them (SQLSTATE mappings). Deliverables: migration SQL (forward/rollback), a comprehensive markdown documentation file under docs/ddl, a runbook snippet for on-call/operators, and a short example test checklist for QA to validate acceptance criteria.

## Dependent Tasks
- task_001_db_create_users_table (migration script forward/rollback) — must be implemented or co-developed so DDL matches migration
- task_002_db_migration_tests (migration unit/integration tests) — to provide test cases and expected results used in documentation

## Impacted Components
- migrations/us_001_create_users_table.sql (migration file - new)
- docs/ddl/us_001_users_schema.md (new documentation)
- docs/ops/migration_runbook.md (append runbook steps)
- app/config/auth/hash_algos.yml or app/config/auth.ts (document allowed algorithms)
- db/schema/README.md (schema overview)
- tests/db/us_001_schema_tests.md (test checklist & SQL snippets for QA)
- CI: .github/workflows/migration-verify.yml (possible update to include migration verification step) — optional

## Implementation Plan
- 1) Extract canonical DDL from migration script (migrations/us_001_create_users_table.sql). Confirm forward and rollback SQL; if migration not yet present, author canonical SQL in migration file.
- 2) Write docs/ddl/us_001_users_schema.md containing:
    - Full DDL (CREATE TABLE + constraints + indexes + partial-index variants)
    - Explanations/rationale for each column and constraint (why salt nullable, why jsonb roles vs text[], PK choice)
    - Allowed hash_algos list and recommendation (argon2id, bcrypt, pbkdf2, legacy_sha256) with recommended canonical identifiers
    - Data format guidance for password_hash storage (text vs bytea) and recommended encoding
    - Concurrency guidance for failed_attempts (example SQL: UPDATE ... SET failed_attempts = failed_attempts + 1 RETURNING failed_attempts) and SELECT ... FOR UPDATE patterns
- 3) Document reset token indexing strategy:
    - Provide Postgres partial index example: CREATE INDEX idx_users_reset_token_active ON users (reset_token_hash) WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now();
    - Provide fallback non-partial index if DB lacks partial index support and describe tradeoffs (maintenance, bloat)
    - Include guidance for optional partial unique index for active tokens and caveats
- 4) Token collision handling guidance: recommended token length/entropy, retry algorithm pseudocode (retry up to N times), logging and monitoring for collisions, and sample SQL to check collisions before insert (transactional approach).
- 5) Pruning schedule and automation: recommended cron or scheduled job SQL to DELETE or NULL-out reset_token_hash/reset_token_expiry for expired tokens (retain audit if needed) with example commands and recommended frequency (daily), plus VACUUM/ANALYZE guidance.
- 6) Migration runbook: step-by-step apply/rollback commands for the project migration runner and raw psql commands, verification queries to validate schema and indexes, test steps to simulate acceptance criteria (unique-violation insert test, concurrent increment test, reset-token lookup test).
- 7) Expected DB error codes mapping: list common PostgreSQL SQLSTATE codes and how the application should handle them (unique_violation - 23505; check_violation - 23514; not_null_violation - 23502; foreign_key_violation - 23503; deadlock_detected - 40P01; serialization_failure - 40001) and sample application-level retry/backoff policies.
- 8) Deliver files and include short QA checklist with SQL snippets to verify acceptance criteria and reproduction steps (to be committed under tests/db).

## Current Project State
- app/
  - config/
  - src/
- migrations/
  - (existing migration files)
- docs/
  - (existing docs)
- tests/
  - db/
- .propel/
  - context/
    - tasks/
      - EP-DATA/
        - us_001/
          - us_001.md

(Actual files to be updated/created as part of this task per Expected Changes section.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | migrations/us_001_create_users_table.sql | Canonical forward + rollback migration SQL for creating users table with all constraints and indexes (including partial-index variant commented). |
| CREATE | docs/ddl/us_001_users_schema.md | Comprehensive documentation containing DDL, rationale for decisions, allowed hash_algo list, store-format guidance, concurrency guidance, and examples. |
| CREATE | docs/ops/migration_runbook.md | Short operational runbook: how to apply/rollback migration, verification queries, rollback caveats, and emergency procedures. |
| CREATE | tests/db/us_001_schema_tests.md | QA checklist and SQL snippets to validate Acceptance Criteria (unique-violation test, atomic increment test, reset-token lookup test). |
| MODIFY | app/config/auth/hash_algos.yml | Add canonical allowed hash_algos list (if file exists; if not, create as part of follow-up). Document location in docs. |
| MODIFY | .github/workflows/migration-verify.yml | (Optional) Add step reference to run migration verification; document change if CI update is required. |

## External References
- PostgreSQL Partial Indexes: https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL CHECK constraints: https://www.postgresql.org/docs/current/ddl-constraints.html
- PostgreSQL SQLSTATE error codes: https://www.postgresql.org/docs/current/errcodes-appendix.html
- pgcrypto / gen_random_uuid: https://www.postgresql.org/docs/current/pgcrypto.html and https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-UUID
- OWASP Password Storage Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html
- Recommended Argon2 parameters (external guidance): https://www.ietf.org/archive/id/draft-irtf-cfrg-argon2-05.html

## Build Commands
- Use project migration runner per repository conventions. Examples:
  - Using Knex: npx knex migrate:latest --knexfile ./knexfile.js
  - Using raw psql (for verification): psql "$DATABASE_URL" -f migrations/us_001_create_users_table.sql
  - Rollback example (Knex): npx knex migrate:down --knexfile ./knexfile.js --to migrations/us_001_create_users_table.js
- CI: run migration verification job (if present): .github/workflows/migration-verify.yml
- Documentation build (if site): make docs or yarn docs:build depending on project docs runner. See ../.propel/build/ for project-specific build commands.

## Implementation Validation Strategy
- [ ] Unit tests pass (where applicable to migration runner)
- [ ] Integration tests pass (migration applied to test DB)
- [ ] Visual comparison: N/A
- [ ] /analyze-ux: N/A
- [ ] Prompt templates validated: N/A
- [ ] Guardrails tested: N/A
- [ ] Fallback logic tested: N/A
- [ ] Token budget enforcement verified: N/A
- [ ] Audit logging verified (no PII in logs): Ensure logs redact P