## Task ID: task_007

## Task Title: Update OpenAPI/Swagger and define error contract for /auth/login

## Category: API

## Description:
Update the project's OpenAPI (Swagger) specification and server API contract for POST /auth/login to (1) include precise request and response JSON Schemas and examples, (2) add documented error responses for HTTP 400, 401, 429, 423, and 503 with a standardized, non-revealing error envelope, and (3) ensure the implementation (controllers/middleware) emits responses that match the spec. Deliverables: openapi.yml/json updates (components/schemas + responses + examples), server-side schema validation mapping, standardized ErrorResponse envelope, and tests validating contract behavior and non-revealing error payloads.

Parent Story: us_001

## Acceptance Criteria:
- AC-1: OpenAPI spec contains:
  - Login request schema (email / username, password) with examples.
  - Login success 200 response schema: access_token (JWT), token_type ("Bearer"), expires_in (seconds) and example.
  - Error response definitions for 400, 401, 429, 423, 503 using a single ErrorResponse envelope in components with examples.
  - Each error response in the /auth/login path references the component responses.
- AC-2: Server returns the documented HTTP status codes and response bodies exactly matching the ErrorResponse envelope fields (traceId, code, message, details[] optional, timestamp) and no internal stack traces or debug data in production.
