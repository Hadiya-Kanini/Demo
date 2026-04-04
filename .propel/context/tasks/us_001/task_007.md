## Task ID: task_007
## Task Title: Instrument metrics and structured logging for auth events
## Category: Backend

## Description:
Implement structured JSON logging and metrics instrumentation for authentication events. Emit the following metrics to the existing metrics backend using the project's metrics client per TR-004 naming conventions: auth.success, auth.failure, auth.lockout, and auth.latency (histogram). Add structured logs for each auth attempt containing correlation id, outcome, duration, and safe context while redacting sensitive fields (passwords, tokens, PII). Integrate instrumentation into the POST /auth/login flow and other auth-related code paths that perform verification and lockout logic (e.g., account lock path). Use the existing logging and metrics clients/adapters (do not add a new external metrics system). Ensure low-cardinality metric labels and trace/correlation propagation (X-Correlation-ID or traceId). Provide unit, integration, and E2E tests that validate metrics emission, structured log contents, redaction, and latency measurement.

Parent story: us_001

## Acceptance Criteria:
- AC-1: On successful authentication, a counter metric named auth.success is emitted once with labels: service, env, auth_method. A structured JSON log entry is written with event_type="auth.success", correlation_id present, user_id (internal id), and latency_ms. No sensitive values (password, token) are present in the log.
- AC-2: On authentication failure (invalid credentials), a counter metric named auth.failure is emitted once with labels: service, env, failure_reason (low-cardinality: "invalid_credentials"/"validation_error"/"internal_error"), auth_method. A structured JSON log entry is written with event_type="auth.failure", correlation_id present, and failure_reason. Password and other sensitive fields are redacted from logs.
- AC-3: When an account/IP lockout is triggered, a counter metric named auth.lockout is emitted with labels: service, env, lockout_type ("account"/"ip"). A structured JSON log with event_type="auth.lockout" is produced and includes correlation_id and lockout policy id if applicable. No sensitive fields are logged.
- AC-4: Each auth attempt records latency in auth.latency as a histogram (or summary) metric with duration in milliseconds and labels service, env, auth_method. Integration tests must show median latency < 1s (configurable NFR) for a successful login under test conditions.
- AC-5: All emitted logs include correlation_id: if request contains header X-Correlation-ID use it; otherwise generate a UUID and return it in response headers as X-Correlation-ID. Correlation id appears in logs and is accepted by trace propagation utilities.
- AC-6: Sensitive fields are redacted in all logs and metrics labels. Redaction list: password, plain_token, refresh_token, credit_card, ssn, full_name (if marked sensitive). Logs may include user_id and masked client_ip (last octet replaced with 0) but never raw passwords or tokens.
- AC-7: Instrumentation uses the project's existing metrics client / logging abstraction. No hard-coded direct calls to external metrics endpoints. Follow TR-004 naming conventions for metric names and label keys.
- AC-8: Automated tests (unit + integration + E2E) validate: metric counters increment, latency recorded, logs contain correlation_id and required fields, and redaction is applied. Tests must mock/stub the metrics backend and structured logger.

## Technical Specifications:

APIs/Endpoints
- POST /auth/login
  - Request: JSON { "username": string, "password": string, ... } (validate per us_001)
  - Response headers: X-Correlation-ID set if not provided by client
  - Behavior: instrument metrics + logs for success/failure + latency
- Any lockout enforcement function(s) invoked by authentication flow
  - Emit auth.lockout metric when threshold reached

Components/Classes (to modify/create)
- AuthController (or AuthRoute handler) — instrument start/stop timer, capture correlation id, call AuthService
- AuthService — perform credential verification, determine outcome, call metrics/logging helpers
- AuthRepository / CredentialVerifier — unchanged business logic; ensure it returns reason codes used for metrics labels
- telemetry/MetricsClient (existing) — use for:
  - incrementCounter(name: string, value: number, labels?: Record<string,string>)
  - recordHistogram(name: string, valueMs: number, labels?: Record<string,string>)
  - (If existing client uses different API names, adapt to those; do not add a new client.)
- logging/StructuredLogger (existing) — use to emit structured JSON logs with allowed fields; extend helper to support redact(list) if not present
- middleware/CorrelationIdMiddleware — ensure correlation_id propagation (reads X-Correlation-ID or generates one)
- security/RedactionUtil (create if not present) — canonical redaction function used by logger to redact sensitive keys before logging

Data Models (logging/metrics payload shapes)
- LogEntry (JSON)
  - timestamp: ISO8601
  - level: "info" | "error" | "warn