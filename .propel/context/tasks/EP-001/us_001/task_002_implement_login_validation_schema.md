# Task - task_002

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-001/us_001/us_001.md
- Acceptance Criteria:  
    - Given malformed input (missing fields, invalid JSON, invalid email format), When the client POSTs to /auth/login, Then the server responds with HTTP 400 with a validation error that identifies invalid fields (e.g., "password: required", "username: invalid email format") and does not proceed to password verification.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable" and a retryable error code; do not leak internal details.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.
    - Password hash algorithm/version mismatch (legacy records) — handle by verifying using the legacy algorithm, force re-hash on next successful login, and record upgrade event without revealing algorithm differences.

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
| Backend | Node.js + TypeScript + Express | Node 18.x, TypeScript 5.x, Express 4.18 |
| Database | PostgreSQL (for context) | 16.x |
| Library | Zod (validation) | v3.x |
| Library | Jest (unit tests) | 29.x |
| Library | Supertest (integration helpers) | 6.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the specified versions or newer.

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
Implement and export a robust, re-usable login request validation schema for POST /auth/login that enforces required fields and formats (username/email and password), returns field-level error messages, and is accompanied by unit tests that confirm malformed input maps to HTTP 400 responses. Provide integration points (middleware / exported schema) so route/controller code can use the schema to short-circuit invalid requests before any password verification or DB access.

## Dependent Tasks
- .propel/context/tasks/setup/task_000_setup_validation_lib.md — ensure Zod (or chosen validation lib) is installed and project linting/typing configured.
- .propel/context/tasks/setup/task_000_setup_tests.md — ensure Jest and test runner setup is available for unit tests.
- .propel/context/tasks/EP-001/us_001/task_001_be_login_route_controller.md — controller/route will use this schema; coordinate API error mapping with that task.

## Impacted Components
- New: src/validators/auth.ts — contains the login request schema and helper mappers
- New: tests/unit/validators/auth.spec.ts — unit tests for validation behavior and error mapping
- Modified: src/middleware/validation.ts — add generic validation middleware (or update to accept exported schemas)
- Modified: src/controllers/authController.ts — small update to import/export the validation schema (no business logic change)
- Modified: src/routes/auth.ts — import and apply validation middleware to POST /auth/login route

## Implementation Plan
- Choose Zod as the validation library and create a strict schema for the login payload:
  - Accept properties: username (string, must be email format OR allow username pattern if project supports), password (string, min length 8 by default but configurable).
  - Provide clear, localized field-level error messages (e.g., "username: invalid email format", "password: required").
- Export both the Zod schema and a TypeScript type (inferred) for use across the codebase.
- Implement a small adapter/utility to convert Zod error details into a standardized HTTP 400 payload:
  - Format: { errors: [{ field: "username", message: "invalid email format" }, ...] }
- Add unit tests covering:
  - Missing fields -> required field error messages and HTTP 400 mapping
  - Invalid email format -> specific message for username
  - Non-string types (number/null) -> type error messages
  - Extra unexpected fields -> depending on policy, either strip or return an error (decide to strip unknown but warn in tests)
- Update or add a validation middleware function (src/middleware/validation.ts) which:
  - Accepts a Zod schema, validates req.body, attaches parsed data to req.validatedBody, and calls next()
  - On validation failure, responds 400 with the standardized field-level error payload; does not call downstream services (no DB calls)
- Wire the middleware into POST /auth/login route (modify route file). Add TODO comments linking to audit/rate-limit behavior for other tasks.
- Run unit tests, ensure full code coverage for schema edge cases.

## Current Project State
- Server/
  - src/
    - controllers/
      - authController.ts
    - routes/
      - auth.ts
    - middleware/
      - validation.ts
    - validators/
      - (empty for now)
    - index.ts
  - tests/
    - unit/
      - (empty for now)
  - package.json
  - tsconfig.json

(Above is a placeholder snapshot — update actual tree if files differ)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/validators/auth.ts | Exports Zod schema `loginRequestSchema`, exported `LoginRequest` type, and helper `formatZodErrors` for mapping to field-level messages. |
| CREATE | tests/unit/validators/auth.spec.ts | Unit tests for validation schema covering missing fields, invalid email, non-string values, and extra fields handling. |
| MODIFY | src/middleware/validation.ts | Add `validateBody(schema)` function (or extend existing) that runs schema.parseAsync/ safeParseAsync and returns HTTP 400 with structured errors on failure. |
| MODIFY | src/routes/auth.ts | Apply `validateBody(loginRequestSchema)` middleware to POST /auth/login route and update imports. Add comment referencing this validation test coverage. |
| MODIFY | src/controllers/authController.ts | Use req.validatedBody (typed) where appropriate; add guard comment to ensure controller does not perform password verification on invalid payloads. |
| MODIFY | package.json | Add unit test script entry if missing and ensure jest & zod are in devDependencies/dependencies. |

## External References
- Zod docs: https://github.com/colinhacks/zod
- Express middleware pattern: https://expressjs.com/en/guide/writing-middleware.html
- Jest testing: https://jestjs.io/docs/getting-started
- RFC 8259 — JSON Data Interchange Format: https://tools.ietf.org/html/rfc8259
- OWASP Input Validation Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html

## Build Commands
- Install deps: npm install
- Run unit tests: npm run test
- Typecheck: npm run build:type (or tsc --noEmit)
- Local dev server: npm run dev (uses ts-node / nodemon when configured)
(Refer to ../.propel/build/ for repository-specific build steps)

## Implementation Validation Strategy
- [ ] Unit tests pass (tests/unit/validators/auth.spec.ts)
- [ ] Integration tests pass where login route uses middleware (if integration test exists)
- [ ] **[UI Tasks]** N/A
- [ ] **[AI Tasks]** N/A

## Implementation Checklist
- [ ] Create src/validators/auth.ts with Zod schema and exported types/helpers
- [ ] Implement formatZodErrors(...) to map Zod issues into { field, message } items
- [ ] Add/extend src/middleware/validation.ts with validateBody(schema) middleware that returns HTTP 400 with structured error body on failure
- [ ] Modify src/routes/auth.ts to apply validation middleware to POST /auth/login
- [ ] Add unit tests tests/unit/validators/auth.spec.ts covering required-field, invalid-email, wrong-type, and extra-field cases
- [ ] Update package.json scripts/devDependencies as needed and run npm install
- [ ] Run tests and fix issues; ensure validation short-circuits controller logic (no DB calls) on invalid input

Notes:
- Effort estimate: 2.5 - 3.5 hours (within single developer 8-hour limit)
- Keep error responses generic (no internal debug details). Example error response shape on validation failure:
  {
    "errors": [
      { "field": "username", "message": "invalid email format" },
      { "field": "password", "message": "required" }
    ]
  }

Rules compliance:
- Checklist contains <=8 items
- Effort <=8 hours

