# Task - [task_005]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-001/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a user supplies incorrect credentials, When the client POSTs to /auth/login, Then the server responds with HTTP 401 and a generic error message:
      {
        "error": "Invalid credentials"
      }
      - Failed login attempt is recorded (user identifier or IP) for rate-limiting and monitoring.
    - Given repeated failed attempts that exceed configured thresholds, When threshold is reached, Then subsequent requests return HTTP 429 (or 423 if account locked) with a generic message ("Too many attempts, try again later"), and the system enforces backoff/lockout per security policy. Alerts and audit logs must be created for suspected brute-force activity.
    - Given a successful login, When the server issues a token, Then an audit record is created (user id, timestamp, client IP, user-agent), and integration tests demonstrate median authentication latency meets NFR (auth median < 1s) under standard load.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable" and a retryable error code; do not leak internal details.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.
    - What happens when a user attempts login during/after account lockout or multi-factor enforcement? — If account is locked due to policy, return HTTP 423 with a generic message. (Migration must support storing lock metadata.)

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
| Frontend | React (if present) | 18.x |
| Backend | Node.js | 18.x |
| Database | PostgreSQL | 16.x |
| Library | Knex (migrations) | v2.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries must be compatible with the versions above. Use parameterized SQL and transactional migrations.

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
Create and add PostgreSQL migrations to persist authentication audit events and failed-login counters. Implement both forward (up) and rollback (down) scripts. The migrations will add two tables:
- auth_audit_logs — stores audit events for authentication activity (success, failure, lockout, password-rehash events) with metadata and indexes for common query patterns.
- failed_login_attempts — stores per-entity counters and lockout metadata (supporting per-user and per-IP strategies) with unique constraints and index optimization.

Migrations must be safe to run in CI and local dev, include transactional guarantees, have appropriate constraints, and be accompanied by a short README section explaining intended semantics and how to use/rollback. Also update migration configuration if necessary.

## Dependent Tasks
- Ensure DB migration tool configured and accessible in the repo (Knex/Flyway) — Artefact: setup_db_migrations
- US_010 (User data & password storage) — schema for users.user_id referenced by FK
- CI/CD pipeline provisioned to run DB migrations (database credentials/secrets available)
- Server DB connection config present at server/db/* (so migration runner can connect)

## Impacted Components
- server/migrations/ (new migration files)
- server/db/migrate-config.js or server/knexfile.js (may need modification to include new SQL folder)
- server/db/README.md (migration usage docs)
- server/src/repos/auth_audit_repository.ts (optional: new repo to be added later — note only migration added in this task)
- CI pipeline migration step (documentation/update)

## Implementation Plan
- Design schema for auth_audit_logs and failed_login_attempts based on ACs and Edge Cases:
  - auth_audit_logs: id (uuid PK), user_id (uuid, nullable), event_type (varchar), event_ts (timestamptz default now()), client_ip (inet), user_agent (text), meta jsonb, source_service text
  - failed_login_attempts: id (uuid PK), user_id (uuid, nullable), ip inet nullable, attempt_count integer default 0, first_attempt_at timestamptz, last_attempt_at timestamptz, locked_until timestamptz nullable, updated_at timestamptz default now()
  - indexes: auth_audit_logs(user_id), auth_audit_logs(event_ts), auth_audit_logs(event_type), failed_login_attempts(user_id unique partial where user_id is not null), failed_login_attempts(ip unique partial where ip is not null)
- Create transactional SQL migration "up" script to create tables and indexes, and "down" script to drop them in reverse order.
- Add new migration files to server/migrations/sql/ with timestamped filenames.
- If repo uses JS migration runner, add/create Knex migration JS file that executes the SQL; else ensure SQL files are discoverable by runner and update migrate-config.
- Add tests (integration/migration) to verify:
  - Migrations run successfully (migrate:latest).
  - Rollback works (migrate:rollback).
  - Schema matches expected columns & indexes (simple queries against information_schema / pg_indexes).
- Document in server/db/README.md the meaning of columns, usage patterns, and guidance for locking strategies (per-user vs per-ip).
- Run migrations locally and in CI to validate.

## Current Project State
- app/ (frontend placeholder)
- server/
  - src/
    - controllers/
    - services/
    - repos/
  - db/
    - migrations/ (may exist or be empty)
    - knexfile.js or migrate-config.js
    - README.md
  - package.json
- .propel/context/tasks/EP-001/us_001/ (user story and related tasks)

(Note: This is a placeholder snapshot. Update paths based on actual repository structure before applying migrations.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.up.sql | SQL migration to create auth_audit_logs and failed_login_attempts tables, with indexes, constraints, and comments. Transactional (BEGIN/COMMIT). |
| CREATE | server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.down.sql | SQL rollback script to drop indexes and tables in reverse order. Transactional. |
| MODIFY | server/db/README.md | Add documentation about the new tables, columns, indexes, usage patterns, and rollback instructions. |
| MODIFY | server/knexfile.js (or server/db/migrate-config.js) | Ensure migration search path includes server/migrations/sql (only if current config doesn't already include it). Add a comment linking to the migration files. |
| CREATE | server/migrations/tests/migration_smoke.test.sql | SQL-based smoke test (to be run by CI) that asserts tables and key indexes exist (optional SQL or instruction to implement as part of CI). |

## External References
- PostgreSQL CREATE TABLE: https://www.postgresql.org/docs/current/sql-createtable.html
- PostgreSQL JSONB docs: https://www.postgresql.org/docs/current/datatype-json.html
- PostgreSQL inet type docs: https://www.postgresql.org/docs/current/datatype-net.html
- Best practices for audit logging and retention: https://martinfowler.com/articles/audit-logging.html
- Knex migrations: https://knexjs.org/#Migrations (if using Knex)

## Build Commands
- Run migrations (Knex):
  - npx knex migrate:latest --knexfile server/knexfile.js
  - npx knex migrate:rollback --knexfile server/knexfile.js
- Or run raw SQL against a local Postgres:
  - psql $DATABASE_URL -f server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.up.sql
  - psql $DATABASE_URL -f server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.down.sql
- CI: integrate migrate:latest step prior to running integration tests.

## Implementation Validation Strategy
- [ ] Migration "up" runs successfully on a fresh database.
- [ ] Migration "down" (rollback) successfully drops created artifacts.
- [ ] Integration smoke test validates that auth_audit_logs and failed_login_attempts tables exist with expected columns and indexes.
- [ ] Insert/select tests: inserting an audit event and incrementing a failed_login_attempts row works and constraints enforce uniqueness as expected.
- [ ] Migration files are idempotent for the expected migration tooling (only run once by migration runner).
- [ ] Documentation updated in server/db/README.md describing schema and intended usage.

## Implementation Checklist
- [ ] Create transactional "up" SQL migration at server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.up.sql that creates both tables, indexes, constraints, and comments. (Estimate: 3 hours)
- [ ] Create corresponding transactional "down" SQL migration at server/migrations/sql/20260406_create_auth_audit_and_failed_attempts.down.sql that cleanly drops indexes and tables. (Estimate: 1 hour)
- [ ] Update server/knexfile.js or server/db/migrate-config.js if necessary to include migrations/sql path and add a comment pointing to the new files. (Estimate: 0.5 hours)
- [ ] Add documentation to server/db/README.md describing column semantics, index rationale, lockout semantics, retention recommendations, and rollback instructions. (Estimate: 0.5 hours)
- [ ] Add a minimal migration smoke test (SQL or CI step) to assert table + index existence and that basic insert/update operations behave as intended. (Estimate: 1 hour)
- [ ] Run migrations locally and in CI to validate up and down behavior and update files if any environment-specific adjustments required. (Estimate: 0.5 hours)
- [ ] Record completion and link migration file names in the US_001 traceability notes. (Estimate: 0.5 hours)

Total estimated effort: 6 hours

## RULES:
- Implementation Checklist must have <=8 items (7 items).
- Effort must be <=8 hours (estimated 6 hours).
- Be specific and actionable — no vague descriptions.
- Expected Changes table lists concrete file paths (CREATE/MODIFY).
- Acceptance Criteria come from the parent User Story.

