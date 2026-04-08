# Task - [TASK_004]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-TECH/us_001/us_001.md]
- Acceptance Criteria:  
    - Given the repository is created, When a developer reads README.md, Then it documents: prerequisites, local setup commands, how to run unit tests, how to run the service locally (with example commands), how to run integration tests (or use mocks), how to run linters/formatters, how to contribute (branching, PR template, commit message conventions), and where to find operational runbooks and CI expectations.
    - Given a developer follows the documented local setup on a supported platform (Linux or macOS), When they execute the documented commands, Then they can: install dev dependencies, run unit tests, and start the service locally within 30 minutes using only the documented steps and using test/stub credentials where applicable.
    - Given a contributor opens a pull request, When they follow CONTRIBUTING.md, Then there is a CONTRIBUTING checklist and PR template referenced in docs, and the README references the CI pipeline, expected static analysis/security scan behavior, and how to add/obfuscate secrets for local development (using environment files or local secret stubs).
    - Given an empty/new repository template, When the scaffolding is applied, Then the repository root contains: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, a clear directory structure (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/), and a Dockerfile.template.
- Edge Case:
    - What happens when a developer is on Windows (boundary condition)? -- README includes explicit Windows guidance and recommends WSL2 or Docker-based development; scaffolding scripts detect unsupported OS and print clear guidance rather than failing silently.
    - How does system handle missing CI tokens or permissions (error scenario)? -- Documentation outlines required CI tokens and permissions; local dev flow uses mocked/stub credentials and scaffolding scripts skip remote-only steps with clear warnings if tokens are absent.
    - What happens when scaffold generation conflicts with existing files? -- Scaffolding script is idempotent and will not overwrite existing files without explicit confirmation; it produces a diff/backup and logs resolution steps for the developer.

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
| Backend | Node.js | 18.x |
| Database | PostgreSQL | 16.x |
| Library / Migration Tool | node-pg-migrate (or Flyway example) | v5 / N/A |
| Orchestration | Docker Compose | v2.x |
| Env Management | dotenv (examples) | latest |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All example files and scripts must be compatible with the versions above.

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
Add local database development examples and an opinionated .env strategy to the repository scaffold so new developers can start and test the service locally without production secrets. Deliverables: .env.example with test/stub credentials, an optional docker-compose DB service example, migration skeleton folder and sample migration, docs describing secrets handling/obfuscation for local development, and README updates that integrate these artifacts into the local setup flow.

## Dependent Tasks
- .propel/context/tasks/EP-TECH/us_001/us_001.md (parent user story content must be present)
- Initial repository scaffold baseline (README.md, CONTRIBUTING.md, .gitignore, templates) - ensure these root files exist before running this task
- If automated scaffolding script exists: ensure task that creates templates/ and scripts/ folders is completed

## Impacted Components
- docs/local-db.md (new)
- .env.example (new)
- docker-compose.db.yml (new, optional example)
- migrations/ (new folder with skeleton and sample migration)
- scripts/local-db.sh (new helper script to start/stop optional local DB)
- README.md (MODIFY - add Local DB section and steps)
- .gitignore (MODIFY - ensure .env is included, local migration artifacts excluded)
- docs/secrets.md (new - guidance on secrets/obfuscation for local dev)
- templates/migration/ (CREATE - migration template file for CLI-driven migrations)

## Implementation Plan
- Create .env.example:
  - Include clearly marked test/stub credentials for PostgreSQL (DB_HOST, DB_PORT, DB_NAME, DB_USER, DB_PASSWORD), include entries for DATABASE_URL and a SAMPLE_OBFUSCATED_SECRET.
  - Add comments and instructions inside .env.example showing how to use local stub values and how to override them for Docker vs direct local runs.
- Add docker-compose.db.yml:
  - Provide an optional, minimal docker-compose file that spins up a PostgreSQL service with named volume and credentials matching .env.example.
  - Include healthcheck and exposed port mapping (5432 -> 5432) and a named network sample; mark the file as optional and safe to run locally.
- Add migrations skeleton and sample:
  - Create migrations/README.md explaining local migration commands and link to chosen migration tooling (node-pg-migrate and an example CLI command).
  - Add a sample "0001_init.sql" or JS migration demonstrating a simple table creation and down-script.
  - Add a migration-tool config (e.g., migrator.config.js or flyway.conf example) with placeholders linking to env vars.
- Add scripts/local-db.sh (POSIX) and scripts/local-db.ps1 (PowerShell):
  - Start/stop helper scripts that read .env (or .env.local) and run docker-compose -f docker-compose.db.yml up -d or down; detect OS and recommend WSL2 on Windows if running non-WSL.
  - Implement a dry-run flag and a --skip-pull flag; ensure scripts do not fail hard when docker not present—print clear guidance.
- Write docs/secrets.md and docs/local-db.md:
  - docs/secrets.md: describe when to use .env (local), how to obfuscate secrets for PRs (example using SOPS GPG stub or masked values), guidance on never committing production secrets, and sample pre-commit hooks to block secrets.
  - docs/local-db.md: step-by-step local setup for DB on Linux/macOS/WSL2/Docker, commands to run migrations, how to use .env.example to create .env, and how to connect from local app to DB.
- Update README.md and .gitignore:
  - Add a "Local Database" section linking to docs/local-db.md, brief example commands to spin up the docker DB, create .env from .env.example, run migrations, and start app with stub creds.
  - Ensure .gitignore includes .env and any local migration artifact patterns.
- Add verification examples to CI docs:
  - Update docs/verify.md or README verification step to mention running the local DB example is optional and tests should be runnable with mocked DB or sqlite example if present.

## Current Project State
- app/
  - src/ (placeholder)
  - tests/ (placeholder)
- docs/
  - (may contain onboarding.md etc.)
- scripts/
  - (may contain existing helpers)
- templates/
  - (may contain file templates)
- .github/
  - workflows/ (may contain placeholders)
- README.md (exists or will be created by dependent task)
- .gitignore (exists or will be created)
Note: This tree is a placeholder — update during task execution if dependent tasks add or change structure.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | .env.example | Example env file with test/stub DB credentials, DATABASE_URL sample and comments describing usage and obfuscation guidance. |
| CREATE | docker-compose.db.yml | Optional docker-compose file that defines a PostgreSQL service configured to match .env.example for local dev. |
| CREATE | migrations/0001_init.sql | Sample SQL migration (CREATE TABLE ...) with reversible down statements or matching down migration file. |
| CREATE | migrations/README.md | Instructions for running migrations locally, tooling options, and env var usage. |
| CREATE | migration.config.js | Example migration tool config (node-pg-migrate or comment pointing to Flyway) referencing env vars. |
| CREATE | docs/local-db.md | Step-by-step doc for starting a local DB, applying migrations, and connecting the app using .env. |
| CREATE | docs/secrets.md | Documentation for secrets handling, obfuscation strategies for local dev, pre-commit checks, and PR guidance. |
| CREATE | scripts/local-db.sh | POSIX helper script to spin up / tear down local docker DB, parse .env, and perform healthchecks. |
| CREATE | scripts/local-db.ps1 | PowerShell equivalent helper with Windows/WSL guidance. |
| MODIFY | README.md | Add "Local Database" section linking to docs/local-db.md and include quick start commands using .env.example and docker-compose.db.yml. |
| MODIFY | .gitignore | Ensure .env and migration local artifact patterns are included (e.g., .migrations/, *.local.sql). |

## External References
- PostgreSQL Docker image docs: https://hub.docker.com/_/postgres
- dotenv (env file) usage: https://www.npmjs.com/package/dotenv
- Docker Compose reference: https://docs.docker.com/compose/compose-file/
- node-pg-migrate: https://github.com/salsita/node-pg-migrate
- Flyway documentation (if used as alternative): https://flywaydb.org/documentation/
- Best practices for local secrets and gitignore: https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html

## Build Commands
- Refer to applicable technology stack specific build commands: ../.propel/build/
- Local DB quick commands (examples to include in docs/ and README):
  - Copy example env: cp .env.example .env
  - Start DB: docker compose -f docker-compose.db.yml up -d
  - Stop DB: docker compose -f docker-compose.db.yml down
  - Run migrations (example): npx node-pg-migrate up
  - Run tests: npm test
  - Run app: npm start (reads .env)

## Implementation Validation Strategy
- [ ] Unit tests pass (project tests should not require production secrets)
- [ ] Integration tests pass (if integration tests require DB, run them against docker-compose.db.yml or use mocked DB per docs)
- [ ] **[UI Tasks]** Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A)
- [ ] **[UI Tasks]** Run /analyze-ux to validate wireframe alignment (N/A)
- [ ] **[AI Tasks]** Prompt templates validated with test inputs (N/A)
- [ ] **[AI Tasks]** Guardrails tested for input sanitization and output validation (N/A)
- [ ] Verify README steps allow a developer on Linux/macOS to install deps, run unit tests, and start service within ~30 minutes using the provided stub credentials
- [ ] Verify scripts/local-db.sh and scripts/local-db.ps1 are idempotent and provide clear guidance when docker is not installed or when on unsupported OS
- [ ] Validate that .env.example contains no real secrets and that docs/secrets.md covers obfuscation and PR-safe workflows

## Implementation Checklist
- [ ] Create .env.example with clear comments and stub credentials for PostgreSQL and DATABASE_URL (Effort: 1 hour)
- [ ] Add docker-compose.db.yml optional example and scripts/local-db.{sh,ps1} helper scripts with OS detection and dry-run (Effort: 2 hours)
- [ ] Create migrations/ skeleton with sample migration and migration.config.js plus migrations/README.md (Effort: 1.5 hours)
- [ ] Add docs/local-db.md and docs/secrets.md explaining local setup, obfuscation, and PR-safe secrets workflow, including Windows/WSL guidance (Effort: 1.5 hours)
- [ ] Modify README.md to reference the local DB docs and provide quick-start commands; update .gitignore to exclude .env and migration artifacts (Effort: 0.5 hours)
- [ ] Validate scripts and docs by performing a local run-through on Linux/macOS and document any manual steps or limitations in docs/local-db.md (Effort: 1.5 hours)
- [ ] Commit files under a single feature branch and open a PR referencing the CONTRIBUTING checklist; ensure scaffolding is idempotent and does not overwrite existing files without confirmation (Effort: 0.5 hours)

Note: Total estimated effort <= 8 hours.

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Total effort estimate across checklist items <= 8 hours.
- Acceptance Criteria are taken from the parent User Story us_001.

