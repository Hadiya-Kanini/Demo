## Task ID: task_005

## Task Title: Add request validation and structured 400 error responses

## Category: Backend

## Description
Add strict request validation and consistent, structured client error responses for the login API (POST /auth/login). Use a schema validator (prefer zod; Joi acceptable if project already uses it) to validate { username, password } payloads. Handle invalid JSON (malformed body) and unsupported/incorrect Content-Type headers. Short-circuit the request pipeline on any parse/validation failure so that downstream authentication logic (password verification, user lookup) is not invoked. Return structured, field-level 4xx responses that include machine-readable error codes and per-field details. Log errors (with traceId) without leaking sensitive values (passwords, hashes, internal stack traces).

Parent story: us_001 (Implement Login API)

## Acceptance Criteria
1. POST /auth/login accepts only application/json content-type. If Content-Type is missing or not application/json the server responds with:
   - HTTP 415 (Unsupported Media Type) OR 400 if project convention uses 400 for content-type errors
   - JSON body matching the "Structured Error Response" format (see Technical Specifications) with code: "UNSUPPORTED_MEDIA_TYPE".
   - No authentication logic invoked.
2. Malformed JSON in request body (JSON parse error) results in:
   - HTTP 400
   - JSON structured error with code: "INVALID_JSON" and a human-friendly message.
   - No authentication logic invoked.
3. Missing or invalid fields (username, password) produce:
   - HTTP 400
   - JSON structured error with code: "INVALID_REQUEST" and a details[] array listing each invalid/missing field with field name and validation message.
   - Example details entry: { "field": "password", "message": "password must be >= 8 characters" }
   - No authentication logic invoked (verifyPassword/userLookup not called).
4. Valid request proceeds to authentication path and behaves per existing us_001 acceptance criteria (valid -> 200 with token, invalid creds -> 401). Validation failures never become 401 or other authentication errors.
5. Error responses must not include stack traces, password values, or other sensitive internals. Errors must include a traceId (request-scoped) to correlate logs.
6. OpenAPI / API documentation is updated to document the login request schema and structured 4xx error envelope.
7. Automated tests validate behavior: unit tests for validation middleware and schema, integration tests for endpoint behavior (invalid JSON/content-type/field errors), and an E2E test demonstrating password verification is not invoked when validation fails.

## Technical Specifications

APIs/Endpoints
- POST /auth/login
  - Request: Content-Type: application/json
  - Request Body Schema (LoginRequest):
    - username: string (non-empty; additional rules: email format if username is email in product context)
    - password: string (min length 8, enforce any existing password complexity policy)
  -