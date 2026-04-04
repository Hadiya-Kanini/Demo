# Task - task_009

## Task ID: task_009
## Task Title: Update OpenAPI spec and developer docs for /auth/login
## Category: API

## Description
Update the project's OpenAPI specification and developer-facing documentation for POST /auth/login to fully specify request and response schemas, examples, and error envelopes for the following HTTP outcomes: 200, 400, 401, 429, 423, 503. Add explicit component schema definitions (LoginRequest, TokenResponse, ValidationError, ErrorEnvelope, JWTClaims), inline examples for each response, and runtime configuration documentation for token TTL and signing. Add operational notes for lockout/throttling behavior, metrics/audit logging requirements, and incident handling (monitoring and on-call runbook pointers). Ensure the OpenAPI file is valid (Spectral + openapi-parser), add checks/tests that examples validate against schemas, and update developer docs with cURL examples, response samples, environment variable names, and instructions for rotating signing keys.

## Acceptance Criteria (specific & testable)
- OpenAPI spec (openapi.yaml) contains:
  - POST /auth/login operation with requestBody referencing components/schemas/LoginRequest.
  - Responses defined for 200, 400, 401, 429, 423, 503 referencing appropriate components/schemas and including at least one example per response.
  - components/schemas include: LoginRequest, TokenResponse, ErrorEnvelope, ValidationError, JWTClaims.
  - components/securitySchemes includes BearerAuth (HTTP bearer JWT) and matches existing security patterns.
- Developer docs (docs/api/auth/login.md) updated and include:
  - Request/response examples for each outcome (200/400/401/429/423/503).
  - Error envelope definition and example, including fields traceId, code, message, details[], timestamp.
  - Environment/config keys for token TTL and signing (names, defaults, types) and guidance for rotating signing keys.
  - Operational notes: lockout thresholds, retry-after behavior, logging