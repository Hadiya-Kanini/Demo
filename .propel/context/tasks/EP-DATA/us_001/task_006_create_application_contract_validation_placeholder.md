# Task - task_006

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:  
    - Given a database migration runner with valid DB credentials, When the migration for us_001 is executed, Then a users table is created with these columns and types: id (UUID PK), email (text, NOT NULL), email_normalized (text, NOT NULL), password_hash (text, NOT NULL), hash_algo (text, NOT NULL), salt (text, NULLABLE), roles (jsonb or text[] NOT NULL DEFAULT ['user']), failed_attempts (integer NOT NULL DEFAULT 0), locked_at (timestamptz NULLABLE), reset_token_hash (text NULLABLE), reset_token_expiry (timestamptz NULLABLE), created_at (timestamptz NOT NULL DEFAULT now()), updated_at (timestamptz NOT NULL DEFAULT now()).
    - Given the users table exists, When an insert is attempted with an email whose normalized form duplicates an existing normalized email, Then the DB rejects the insert with a unique-violation error due to a UNIQUE index on email_normalized and demonstration tests assert the expected error code.
    - Given simultaneous login attempts that increment failed_attempts, When multiple increments occur concurrently, Then failed_attempts reflects the total number of attempts (no lost updates) because updates use atomic DB operations (e.g., UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = ... RETURNING failed_attempts) and tests simulate concurrency to verify correctness.
    - Given a reset token generation flow, When the application stores reset_token_hash and reset_token_expiry, Then there exists an index on reset_token_hash to allow efficient lookup and the application enforces reset_token_expiry to be a future timestamp at time of insertion; tests verify lookup performance on seeded token records and expiry enforcement in application logic.
    - Given hash algorithm policy, When creating or updating records, Then either a database-level CHECK/ENUM constraint or application-level validation restricts hash_algo to the allowed set (document the allowed list in migration or application config) and password_hash column size/format supports storing modern algorithm outputs and legacy values; unit tests verify permitted and rejected hash_algo values.
- Edge Case:
    - What happens when a legacy password hash format is encountered?
        - Handling: Schema permits storing legacy formats by capturing the format in hash_algo and allowing salt to be nullable. The application must detect legacy hash_algo during authentication and perform migration-on-login (rehash into modern algorithm) in a separate story. The DB will accept the legacy record; migration is handled at application layer, not via destructive DB migration.
    - How does system handle concurrent updates to failed_attempts and lockout threshold races?
        - Handling: Application must use DB atomic increments or SELECT ... FOR UPDATE to avoid lost updates. The migration includes a CHECK constraint (failed_attempts >= 0) and tests simulate concurrent increments to validate correctness. If DB-level distributed concurrency issues appear in scale testing, a follow-up story will add optimistic concurrency control or centralized counters.
    - What happens when reset token hash collisions or race conditions occur?
        - Handling: The DB provides an index to look up tokens but the application is responsible for generating sufficiently random tokens, hashing them, and retrying generation on collision. Optionally implement a partial unique index on reset_token_hash for active (non-expired) tokens in DBs that support partial indexes; otherwise handle collisions in application logic with retries and logging for monitoring.

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
| Backend | Node.js (TypeScript) | 18.x / 5.x (ts-node/tsc compatible) |
| Database | PostgreSQL | 13+ (prefer 12+) |
| Library | node-postgres (pg) | v8.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: All example pseudocode and guidance in this contract assume PostgreSQL features (partial indexes, enum/check constraints). If another RDBMS is used, adapt SQL examples accordingly.

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
Produce a concise, consumable application contract and pseudocode document for backend teams that defines:
- Allowed hash_algo identifiers and validation rules,
- Reset token expiry semantics and insertion/verification pseudocode,
- Recommended atomic increment patterns for failed_attempts and locking guidance,
- Collision handling guidance and retry pseudocode for reset token generation,
- Examples of DB-level vs. application-level enforcement (enum/check vs validation),
- Minimal test cases to assert the rules (unit test stubs/pseudocode).

This task produces artifacts (documentation and pseudocode) that implementation teams will use to implement migrations, application validators, and tests in subsequent tasks.

## Dependent Tasks
- .propel/context/tasks/EP-DATA/us_001/us_001.md (user story specification) — read before starting
- Task 000 — Confirm DB Platform & Migration Tooling (must validate PostgreSQL usage and extensions policy)
- Task 001 — Create users table migration (draft or design must exist so contract matches DB choices)

## Impacted Components
- server/src/auth/ (new docs and pseudocode references for auth subsystem)
- server/src/db/ (migration guidance references)
- docs/contracts/ (new contract document)
- tests/unit/auth/ (test stubs to be created referencing the contract)
- CI pipeline (reference to add migration validation step — informational)

## Implementation Plan
- Draft a single authoritative contract document (docs/contracts/user_validation_contract.md) with sections:
  - Summary and intent
  - Allowed hash_algo list (rationale + identifiers)
  - DB-level constraints vs application validation guidance
  - Pseudocode: create/update user validation (hash_algo check example)
  - Pseudocode: failed_attempts atomic increment patterns and example SQL
  - Pseudocode: reset token creation, storage, expiry enforcement and lookup
  - Pseudocode: reset token collision retry loop with max attempts and logging
  - Partial unique index recommendation (SQL snippet) and fallback if DB doesn't support partial unique indexes
  - Minimal test cases and queries to validate behavior
- Provide explicit allowed hash_algo list and mapping to expected hash formats:
  - 'argon2' -> Argon2id (encoded string, verify with argon2 lib)
  - 'bcrypt' -> bcrypt hash (string prefix $2b$ or $2y$)
  - 'pbkdf2' -> PBKDF2 (store algorithm label + params in string)
  - 'legacy_sha256' -> legacy SHA256 hex/base64 (salt nullable)
  - Document: each identifier, expected storage format, and validation predicate
- Provide concrete SQL and pseudocode examples:
  - SQL CHECK/ENUM example for hash_algo
  - UPDATE users SET failed_attempts = failed_attempts + 1 WHERE id = $1 RETURNING failed_attempts
  - SELECT ... FOR UPDATE example when needing to update failed_attempts and set locked_at
  - Partial unique index example for reset_token_hash WHERE reset_token_hash IS NOT NULL AND reset_token_expiry > now()
- Provide reset token generation pseudocode:
  - Generate random bytes -> base64/urlsafe -> hash with HMAC-SHA256 or bcrypt/argon2 as design chooses -> store reset_token_hash, set reset_token_expiry = now() + ttl
  - Collision handling: loop with max_retries (default 5); on collision generate new token, log retries, raise alert if exhausted
- Provide test stubs (pseudocode) to validate:
  - Insert with allowed and disallowed hash_algo values (assert rejection/acceptance)
  - Concurrent increments test outline (spawn N concurrent UPDATEs, assert final failed_attempts)
  - Reset token insertion and lookup and expiry behavior
- Deliverables:
  - Contract document (MD) with examples and SQL snippets
  - Pseudocode files under server/src/auth/validation_contract.md (for devs)
  - Minimal test stubs under tests/unit/auth/ referencing the contract
- Timebox: produce the contract and stubs in <= 8 hours; no migrations are applied in this task.

## Current Project State
- app/ (frontend — not used)
- server/
  - src/
    - auth/ (authentication code)
    - db/ (DB access and migrations)
    - utils/
  - migrations/
- docs/
  - contracts/
- tests/
  - unit/
  - integration/

(Above is a placeholder project layout; update during execution if actual repo differs.)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | docs/contracts/user_validation_contract.md | Authoritative contract document describing allowed hash_algos, reset token rules, atomic increment patterns, collision handling, SQL examples, and test stubs. |
| CREATE | server/src/auth/validation_contract.md | Developer-focused pseudocode and snippets (SQL + TypeScript pseudocode) used by backend implementers. |
| CREATE | tests/unit/auth/contract_test_stubs.test.md | Test stubs/pseudocode outlining unit and concurrency tests to be implemented later. |
| MODIFY | docs/README.md | Add link/reference to new contract document under contracts section. |

## External References
- OWASP Authentication Cheat Sheet: https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html
- Argon2 spec / libs: https://www.argon2.id/
- bcrypt: https://en.wikipedia.org/wiki/Bcrypt
- PostgreSQL partial index docs: https://www.postgresql.org/docs/current/indexes-partial.html
- PostgreSQL CHECK/ENUM docs: https://www.postgresql.org/docs/current/ddl-constraints.html
- Secure token generation guidance (NIST SP 800-63B): https://pages.nist.gov/800-63-3/sp800-63b.html

## Build Commands
- Validate docs locally: markdown linter (repo-specific) e.g., npx markdownlint docs/contracts/user_validation_contract.md
- Run unit test stubs linter: npm test (if applicable)
- Migration run/rollback validation: per project migration tool (see .propel/context/tasks/EP-DATA/us_001/us_001.md for tooling). For Node projects typical commands:
  - Install deps: npm ci
  - Run tests: npm test
  - Run linter: npm run lint

## Implementation Validation Strategy
- [ ] Unit tests pass (test stubs updated into runnable tests in follow-up)
- [ ] Integration tests pass (if applicable in follow-up tasks when migrations/tests implemented)
- [ ] Concurrency/integration test design reviewed and approved
- [ ] Contract reviewed and accepted by backend/auth team (PR review)
- [ ] Migration SQL snippets validated against PostgreSQL 13+ syntax

## Implementation Checklist
- [ ] Create docs/contracts/user_validation_contract.md containing: allowed hash_algo list, SQL and application-level validation guidance, reset token lifecycle, collision retry pseudocode, and atomic increment patterns. (2.5h)
- [ ] Create server/src/auth/validation_contract.md with concise, copy-pasteable pseudocode (SQL + TypeScript examples). (1.5h)
- [ ] Create tests/unit/auth/contract_test_stubs.test.md containing test scenarios and pseudocode for unit and concurrency tests. (1h)
- [ ] Add link to the contract in docs/README.md and open PR for peer review. (0.5h)
- [ ] Run a quick review session with one backend engineer to validate clarity and make small edits. (1h)
- [ ] Address feedback and finalize document; mark task complete. (1h)

Effort: <= 8 hours total.

## RULES:
- Implementation Checklist has 6 items (<=8)
- Effort estimated <=8 hours
- Checklist items are specific and actionable

