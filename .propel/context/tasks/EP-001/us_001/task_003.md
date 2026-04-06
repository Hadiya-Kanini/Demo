## Task ID: task_003
## Task Title: Create DB schema & migrations for auth_audit_logs and failed_login_attempts
## Category: Database

## Description
Add two new database tables and associated indexes, constraints, and reversible migrations to support authentication audit trails and failed-login tracking:

- auth_audit_logs: append-only audit log for authentication events (login success, logout, token_issued, password_change, lockout, etc.) with user_id (nullable), timestamp, client metadata (IP, user-agent), event_type and optional metadata JSON.
- failed_login_attempts: append-only records of failed authentication attempts (username/email used, ip_address, user_agent, failure_reason, attempt_at). Provide indexes to efficiently query recent failed attempts by username or IP for rate-limiting/lockout decisions.

Deliverables:
- SQL migration(s) with explicit up and down (rollback) steps (transactional).
- Paths / file names following existing project migration convention (e.g., db/migrations/V003__create_auth_audit_logs_failed_login_attempts.sql). If the project uses a migration tool, produce equivalent files for that tool.
- Minimal repository/DAO interfaces (signatures) that will be used by backend code to write/read records (no implementation of endpoint logic).
- Test instructions and example SQL / test scripts to validate schema, indexes, constraints, and rollback.
- Documentation notes for retention policy and recommended background purge job.

Parent Story: us_001

## Acceptance Criteria
1. Migration(s) create the two tables with the exact columns and constraints described in the Data Models section and must be idempotent when run against an empty database.
2. Each migration includes a reversible DOWN section (rollback) that cleanly removes the created objects and restores the DB to prior state.
3. Indexes exist to support:
   - fast queries of recent failed attempts by username and by ip_address (AC: count of attempts in last N minutes query executes with index usage).
   - fast lookup of audit entries by user_id and event_type and time-range (AC: reading last N audit events for a user is indexed).
4. Foreign key linking auth_audit_logs.user_id -> users.id is created with safe delete semantics (user_id nullable; ON DELETE SET NULL). If users table does not exist, migration fails with descriptive error.
5. Migration is transactional: either fully applied or fully rolled back on error.
6. Clear test instructions and scripts provided that:
   - validate table and index exist,
   - insert sample rows for both tables and verify constraints,
   - run rollback and confirm objects removed.
7. Provide example SQL queries for backend usage: count failed attempts in sliding windows, latest audit logs for a user, and a sample backfill/purge statement for logs older than retention period.
