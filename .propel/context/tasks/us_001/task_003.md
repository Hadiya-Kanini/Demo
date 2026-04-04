## Task ID: task_003
## Task Title: Create auth_audit_logs migration and DB access layer
## Category: Database

## Parent Story
- us_001

## Description
Add a new persistent audit table auth_audit_logs to record authentication events from the login flow. Deliver a reversible DB migration (up/down) that creates the table, required indexes, and a documented retention/DR approach. Implement a small DB access layer (repository/helper) in the backend to append audit rows from the login flow. The helper must be safe, parameterized, non-blocking for the auth path (best-effort logging), and unit/integration/E2E tested.

The table must capture:
- id (surrogate PK)
- user_id (nullable FK when user exists)
- username_or_ip (string - the submitted username/email or client IP for anonymous attempts)
- event_type (string/enum e.g., login_success, login_failure, token_refresh, logout, lockout)
- client_ip
- user_agent
- meta (JSONB) for extensible context (e.g., oauth client id, reason for failure)
- created_at (timestamptz) and optionally updated_at

Also create appropriate indexes for common query patterns and a retention strategy (documented and optionally included as a scheduled cleanup/partitioning suggestion). Provide rollback (down migration) to drop created objects.

## Acceptance Criteria
1. Migration exists and is runnable by the project's migration tool (SQL or migration script) and, when applied, creates auth_audit_logs with all columns and constraints described in the Description.
2. Migration down path cleanly drops the table and associated indexes (no residual objects).
3. Indexes created:
   - index on user_id
   - index on event_type
   - index on created_at (or partitioning strategy)
   - optional GIN index on meta (JSONB) if meta queries are expected
4. A documented retention plan is included in the migration or in the DB notes (e.g., partition-by-month + scheduled drop or pg_cron cleanup SQL). The plan must specify default retention (e.g., 90 days) and how to perform DR/archival.
5. A backend DB helper (repository) exists with a typed API to append audit rows and is used by the login flow (successful and failed login paths). It must:
   - Use parameterized queries (no string concatenation).
   - Be non-blocking to the login response (fire-and-forget pattern with error logging).
   - Accept meta JSON objects and null user_id for anonymous attempts.
6. Unit tests for the repository verify correct SQL and parameter mapping; integration tests verify a real insert when migrations are applied; an E2E test verifies that POST /auth/login results in an auth_audit_logs row for success and failure cases.
7. The implemented solution follows DB standards: explicit names for constraints/indexes, reversible migrations, and migration safety notes (expand → migrate → switch pattern if future schema changes are needed).
8. All new code has concise documentation comments and a short usage example for the login flow.

## Technical Specifications

APIs/Endpoints
- No new external REST endpoints required.
- Integration point: existing POST /auth/login handler should call the helper:
  - authAuditRepository.appendAudit(audit: AuthAuditInsert): Promise<void>
  - Usage: called after credential verification result (success or failure). Should not alter login behavior if audit insert fails.

Components/Classes
- Migration:
  - server/db/migrations/XXXX