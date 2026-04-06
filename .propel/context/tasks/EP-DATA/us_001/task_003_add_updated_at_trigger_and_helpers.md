# Task - TASK_003

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-DATA/us_001/us_001.md
- Acceptance Criteria:  
    - Given the users table exists, When creating or updating user records, Then the backend validates hash_algo against a documented allowed set (e.g., 'argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256') and rejects unknown algorithms with a clear, testable error.
    - Given the users table exists, When creating new users, Then the backend persists roles in the DB consistent with the schema (jsonb or text[]) and ensures a default of ['user'] is applied when roles are omitted.
    - Given the new schema and app-level validations, When hash_algo or roles are updated by clients, Then invalid values are rejected by the backend before attempting DB writes and unit/integration tests verify these behaviors.
- Edge Case:
    - What happens when a legacy password hash format is encountered? - The backend must accept rows with legacy hash_algo values when they are in the allowed set (e.g., 'legacy_sha256') and must not attempt destructive conversion; it should tag/record the algorithm and allow application-layer migration-on-login in a separate story.
    - How does backend handle missing roles or unexpected roles type (string vs array)? - The backend must normalize input (accept single role string or array) into the canonical server representation (JSON array or text[]), apply the default ['user'] when omitted or empty, and return a 4xx validation error for invalid types (e.g., object).
    - How to keep allowed hash_algo list in sync with DB-level constraints? - This task requires documenting and sourcing the allowed set from a single config location; DB-level enum/check should be referenced in documentation and mirrored in backend config. If DB-level constraint changes, update the backend config accordingly.

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
| Backend | Node.js (TypeScript) | 18.x / 5.x (TS) |
| Database | PostgreSQL | 14.x (>= 12 recommended) |
| Library | node-postgres (pg) / Zod or Joi for validation | pg v8.x, zod v4.x (or joi v17.x) |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the stated versions.

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
Mirror the allowed hash algorithm set and canonical roles handling in the backend code so application-level validation and normalization match the DB schema expectations described in us_001. Deliverables include a single source-of-truth configuration for allowed hash algorithms, input validation and normalization for roles on create/update flows, unit and integration tests validating acceptance criteria, and short documentation describing how to maintain parity with DB-level constraints.

Capabilities delivered:
- Backend validation for hash_algo using a central config and clear errors for unsupported algorithms.
- Normalization and persistence logic for roles (accept string/array, persist as JSONB/text[]), applying default ['user'].
- Tests that validate behavior and edge-cases (legacy algos accepted, invalid types rejected).
- Documentation updates describing where to update allowed algorithms when DB constraints change.

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/us_001.md (database migration creating users table, with roles column and hash_algo constraint or documentation)
- Task 000 / discovery confirming PostgreSQL version and whether users.roles is jsonb or text[] (implementation must target the actual DB column type)
- CI task to run integration tests against dev DB with seeded users table

## Impacted Components
- Backend configuration:
  - config/auth.ts | config/auth.yaml (allowedHashAlgos)
- Validation layer:
  - src/validation/schemas/user.ts (Zod/Joi schema for create/update user requests)
- Service layer:
  - src/services/userService.ts (createUser, updateUser) — normalize roles, validate hash_algo
  - src/services/authService.ts (if it validates hash_algo on login/signup)
- Database access layer / queries:
  - src/db/users.repo.ts (ensure roles persistence uses JSONB/text[] parameters)
- Tests:
  - tests/unit/user.validation.spec.ts
  - tests/integration/user.roles_and_hash_algo.spec.ts
- Documentation:
  - README.md (database section) — add note about allowed hash algorithms and roles handling

## Implementation Plan
- Confirm column type for roles in DB (jsonb vs text[]) by inspecting migration artifact from us_001. Choose mapping strategy:
  - If roles = jsonb: persist roles as JSON array.
  - If roles = text[]: map incoming array to Postgres array parameter, or use text[] handling in pg.
- Create a central configuration file (e.g., config/auth.ts) exporting:
  - allowedHashAlgos: string[] = ['argon2', 'bcrypt', 'pbkdf2', 'legacy_sha256'] (document rationale)
  - rolesDefault: string[] = ['user']
- Update validation schema (Zod/Joi) for user create/update:
  - hash_algo: string().refine(value => allowedHashAlgos.includes(value), { message: ... })
  - roles: accept string or array of strings; if omitted -> rolesDefault; validate each role against allowed role list if project has role policy (if none, allow any string matching ^[a-z_]+$).
- Update service layer:
  - createUser(data): normalize roles to array, apply default, validate hash_algo using config, then call users.repo.insert with appropriate parameter shapes.
  - updateUser(id, patch): if hash_algo present, revalidate; if roles present, normalize and persist.
- Update database repository code:
  - Ensure parameter binding uses JSON.stringify for jsonb or array for text[]; add conversion helpers if necessary.
  - Protect against SQL injection by using parameterized queries.
- Add unit tests:
  - Validation tests: accept allowed algos, reject disallowed; normalize roles variants.
  - Service tests: ensure default roles applied when omitted; ensure normalized persisted payload expected by repo is produced (can mock repo).
- Add integration tests (run against dev DB):
  - Create user request with various roles input (string, array, omitted) -> verify stored representation and default applied.
  - Attempt create/update with unsupported hash_algo -> backend returns 4xx and no DB write.
  - Create user with legacy hash_algo value included in allowed list -> backend accepts and persists algorithm identifier.
- Documentation:
  - Update README.md database section with the canonical allowed hash_algo list location and instructions to keep in sync with DB-level constraints and migrations.
- Run CI: unit tests and integration tests against test DB.

## Current Project State
- app/ (frontend) — N/A for this task
- Server/
  - src/
    - config/ (may exist)
    - services/
    - db/
    - validation/
    - controllers/
  - tests/
- migrations/
  - ... (us_001 users table migration exists or will be applied)
- NOTE: Exact repo paths may differ; implementers must adapt paths to repository conventions. Confirm whether roles column is jsonb or text[] before persisting.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/config/auth.ts | Exports allowedHashAlgos (single source-of-truth), rolesDefault, and optional allowedRoles pattern. |
| MODIFY | src/validation/schemas/user.ts | Add/refine hash_algo validation against allowedHashAlgos; normalize/validate roles input (string|array -> array). |
| MODIFY | src/services/userService.ts | Ensure createUser/updateUser validate hash_algo, normalize roles, apply default ['user'], and call repo with correctly shaped data. |
| MODIFY | src/db/users.repo.ts | Ensure insert/update queries serialize roles correctly for jsonb/text[] and preserve existing behavior; add comments about expected DB column type. |
| CREATE | tests/unit/user.validation.spec.ts | Unit tests for hash_algo validation and roles normalization. |
| CREATE | tests/integration/user.roles_and_hash_algo.spec.ts | Integration tests verifying persisted roles representation and rejection of disallowed hash_algo (runs against dev/test DB). |
| MODIFY | README.md | Database section: document allowed hash algorithms config location, roles canonical representation, and maintenance notes to keep backend and DB constraints in sync. |

## External References
- Zod docs: https://github.com/colinhacks/zod
- Joi docs: https://joi.dev
- node-postgres docs: https://node-postgres.com/
- PostgreSQL JSONB docs: https://www.postgresql.org/docs/current/datatype-json.html
- PostgreSQL text[] docs: https://www.postgresql.org/docs/current/arrays.html
- Usability note: bcrypt/argon2 differences — Argon2 recommended: https://www.argon2.net/

## Build Commands
- Install dependencies:
  - npm install
- Run unit tests:
  - npm test (or npm run test:unit)
- Run integration tests (requires dev/test DB configured):
  - npm run test:integration
- Lint/format (project specific):
  - npm run lint
- Example local run:
  - NODE_ENV=test DATABASE_URL="postgres://user:pass@localhost:5432/testdb" npm run test:integration

(Refer to project migration tooling and CI build steps at ../.propel/build/)

## Implementation Validation Strategy
- [ ] Unit tests pass (validation and service unit tests)
- [ ] Integration tests pass (create/update flows against dev/test DB)
- [ ] Backend validation rejects unsupported hash_algo values with a 4xx error and no DB write
- [ ] Roles normalization verified: single string -> ['role'], array preserved, omitted -> ['user']
- [ ] Persisted representation matches DB column type (jsonb array or text[]), verified by integration tests
- [ ] README.md updated with instructions to keep backend allowedHashAlgos in sync with DB-level constraints

## Implementation Checklist
- Effort estimate: 6 hours
- [ ] Add src/config/auth.ts with allowedHashAlgos and rolesDefault (single source-of-truth).
- [ ] Update src/validation/schemas/user.ts to validate hash_algo and normalize/validate roles input.
- [ ] Update src/services/userService.ts (and authService if needed) to enforce validation, normalization, and apply default roles before DB writes.
- [ ] Update src/db/users.repo.ts to ensure roles are serialized correctly for jsonb/text[] and add helpful comments.
- [ ] Add unit and integration tests (tests/unit/user.validation.spec.ts and tests/integration/user.roles_and_hash_algo.spec.ts) and run them against dev/test DB.
- [ ] Update README.md database section describing where allowedHashAlgos lives and how to keep it in sync with DB constraints.
- [ ] Run full test suite and a manual integration check against dev DB to confirm ACs (hash_algo validation, roles persistence).

## RULES:
- Implementation Checklist must have <=8 items
- Effort must be <=8 hours
- Be specific and actionable — no vague descriptions
- Expected Changes table lists concrete file paths (CREATE/MODIFY)
- Acceptance Criteria are from the parent User Story (us_001) and focused on hash_algo and roles handling