## Task ID: task_006
## Task Title: Emit audit logs and authentication metrics
## Category: Backend

## Description:
Implement server-side audit logging and authentication metrics for the login flow. On each authentication attempt (POST /api/auth/login) write a structured audit record to the database containing: user_id (if known), timestamp, client IP, user-agent, outcome (success/failure), and authentication method. Do NOT store or log any sensitive values (passwords, plain tokens, or password hashes). Additionally, publish Prometheus-compatible metrics for auth.success, auth.failure (counters) and auth.latency (histogram) measuring handler latency. Hook metrics into existing /metrics endpoint (or create one if missing). Ensure low-cardinality metric labels and keep audit storage queryable for monitoring and incident investigation.

Traceability: Parent user story us_001 — update login flow to record audit events and auth metrics per acceptance criteria.

## Acceptance Criteria:
- AC-1: Given a successful login via POST /api/auth/login, the system inserts one row into auth_audit_logs with columns: id, user_id (not null for registered users), occurred_at (UTC timestamp), client_ip, user_agent, outcome = 'success', auth_method, and no sensitive fields. Testable by querying DB after request.
- AC-2: Given a failed login (invalid credentials), the system inserts one row into auth_audit_logs with user_id NULL (or user lookup id if identifiable but authentication failed) and outcome = 'failure'. Testable by querying DB after request.
- AC-3: auth metrics are published and observable via the metrics endpoint:
  - Counter metric (name: auth_success_total) increments on successful login attempts.
  - Counter metric (name: auth_failure_total) increments on failed login attempts.
  - Histogram metric (name: auth_latency_seconds) records end-to-end login handler latency with defined buckets (e.g., [0.01,0.05,0.1,0.25,0.5,1,2,5]).
  Testable by calling /metrics or scraping Prometheus test instance after login attempts.
- AC-4: No logs, DB columns, or metrics include passwords, raw credentials, or JWT tokens. Automated checks must validate that payload fields stored or emitted do not contain these sensitive values.
- AC-5: Integration test measuring median end-to-end auth latency (over a sample) must show median < 1s as specified by NFR; metrics histogram must reflect latency distribution used for this check.
- AC-6: When the database is unavailable, login returns HTTP 503 (or original service-unavailable behavior), and no partial audit row is left in the DB. If audit insertion fails but login succeeded, the system logs an error (non-sensitive) and increments a separate metric auth_audit_write_failures_total.

## Technical Specifications:

APIs/Endpoints
- POST /api/auth/login
  - Existing endpoint: augment flow to emit audit + metrics.
  - Request: { email: string, password: string } (do not store these).
  - Responses preserved: 200 on success with access_token, 401 on invalid credentials, 400 on validation errors, 503 on DB/service issues per us_001.
- GET /metrics
  - Expose Prometheus metrics (existing or add route). Must include auth_success_total, auth_failure_total, auth_latency_seconds, auth_audit_write_failures_total.

Components / Classes (suggested names & responsibilities)
- AuthController / authController.ts
  - Orchestrates request lifecycle for POST /api/auth/login.
  - Instrument start/end time and calls MetricsService and AuthAuditRepository after auth decision.
- AuthService / authService.ts
  - Performs credential verification, password hashing comparison, token issuance.
  - Returns { success: boolean,