## Task ID: task_001

## Task Title: Implement input validation middleware for POST /auth/login

## Category: Backend

## Description:
Implement a server-side input validation middleware for the POST /auth/login endpoint. The middleware must validate Content-Type and JSON payload shape against a defined request DTO / JSON Schema, enforce required fields and formats (email or username present, password present), and return a standardized HTTP 400 error envelope containing machine-friendly error codes and per-field validation details for any malformed or missing fields. On valid input the middleware must call next() to hand off to authentication logic. Implementation should be in TypeScript for the Node.js/Express backend and be lightweight, testable, and consistent with existing error-envelope conventions (see Acceptance Criteria). Provide schema, middleware, route integration, and unit/integration/E2E tests.

Parent Story: us_001

## Acceptance Criteria:
1. POST /auth/login accepts only application/json requests. Requests without Content-Type: application/json receive HTTP 400 with an error code CONTENT_TYPE_INVALID and a clear message.
2. Request body schema:
   - Required: password (non-empty string, max 128 chars)
   - Required: at least one identifier: email OR username
   - If email provided: must be a valid RFC-5322-ish email format
   - If username provided: must be a non-empty string, allowed characters [A-Za-z0-9_.-], max 64 chars
   Requests violating the schema return HTTP 400 with error code VALIDATION_ERROR and details array listing each invalid field with a message and a field-specific error code (e.g., email_format_invalid, password_missing).
3. Error envelope format for HTTP 400:
   {
     "error": {
       "code": "<TOP_LEVEL_ERROR_CODE>",
       "message": "<human_readable_summary>",
       "details": [
         { "field": "<json-path>", "code": "<field_error_code>", "message": "<message>" }, ...
       ],
       "timestamp": "<ISO8601>"
     }
   }
   - No internal stack traces, DB errors, or debug details must be present in responses.
4. On valid payloads, middleware passes control to next() without modifying the request body (except optional safe normalization, e.g., trimming).
5. Middleware logs validation failures (structured log) including request-id/traceId if present, client IP, and user-agent, but does not log passwords or full payloads.
6. Unit tests cover schema rules and error formatting for at least 90% of middleware logic branches.
7. Integration tests using the real express /auth/login route verify:
  