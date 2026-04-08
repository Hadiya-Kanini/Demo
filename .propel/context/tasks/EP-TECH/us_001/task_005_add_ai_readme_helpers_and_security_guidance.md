# Task - [task_005]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/us_001/us_001.md]
- Acceptance Criteria:  
    - Given an empty/new repository template, When the scaffolding is applied, Then the repository root contains: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, a clear directory structure (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/), and a Dockerfile.template.
    - Given the repository is created, When a developer reads README.md, Then it documents: prerequisites, local setup commands, how to run unit tests, how to run the service locally (with example commands), how to run integration tests (or use mocks), how to run linters/formatters, how to contribute (branching, PR template, commit message conventions), and where to find operational runbooks and CI expectations.
    - Given a developer follows the documented local setup on a supported platform (Linux or macOS), When they execute the documented commands, Then they can: install dev dependencies, run unit tests, and start the service locally within 30 minutes using only the documented steps and using test/stub credentials where applicable.
    - Given a contributor opens a pull request, When they follow CONTRIBUTING.md, Then there is a CONTRIBUTING checklist and PR template referenced in docs, and the README references the CI pipeline, expected static analysis/security scan behavior, and how to add/obfuscate secrets for local development (using environment files or local secret stubs).
    - Given repository governance requirements, When the template is used to create a new service repository, Then CI badges (placeholder allowed), pre-commit config, a basic pre-configured GitHub Actions workflow folder (.github/workflows/) and documentation for running the pipeline locally exist, and the README includes a verification step that the repo passes a basic lint and license-check as described.
- Edge Case:
    - What happens when a developer is on Windows (boundary condition)? -- README includes explicit Windows guidance and recommends WSL2 or Docker-based development; scaffolding scripts detect unsupported OS and print clear guidance rather than failing silently.
    - How does system handle missing CI tokens or permissions (error scenario)? -- Documentation outlines required CI tokens and permissions; local dev flow uses mocked/stub secrets and scaffolding scripts skip remote-only steps with clear warnings if tokens are absent.
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
| Backend | Python (scaffolding & CLI) | 3.11+ |
| Database | N/A | N/A |
| Library | click (CLI) / difflib / jinja2 for templating | click v8.x, jinja2 v3.x |
| AI/ML | OpenAI GPT (API calls from CLI) | gpt-4-turbo |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the versions above.

## AI References (AI Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **AI Impact** | Yes |
| **AIR Requirements** | AIR-001, AIR-002 |
| **AI Pattern** | Tool Calling |
| **Prompt Template Path** | config/ai/prompts/readme_helpers/ |
| **Guardrails Config** | config/ai/guardrails.yaml |
| **Model Provider** | OpenAI (gpt-4-turbo) |

### CRITICAL: AI Implementation Requirements (applicable)
- MUST reference prompt templates from config/ai/prompts/readme_helpers/ during implementation.
- MUST implement guardrails for sanitizing user inputs and validating model outputs (config/ai/guardrails.yaml).
- MUST enforce a token budget (limit per call) as specified by AIR-001.
- MUST implement fallback logic when model confidence is low or model fails (error messages, local template fallback).
- MUST log all prompts/responses for audit in logs/ai_prompts.log (PII must be redacted).
- MUST handle model failures gracefully: timeouts, rate limits, 5xx errors with retries/exponential backoff and degrade to local templates.

## Task Overview
Add AI-assisted README helpers and security guidance to the repository scaffolding: provide templates and a small CLI tool that can (optionally) call an LLM to generate README/PR text snippets from structured inputs, include explicit security/secrets handling guidance, placeholders for static-analysis/security-scan behavior, CI/PR templates and pre-commit config, and ensure the scaffolding is idempotent and fails gracefully per user story edge cases.

## Dependent Tasks
- N/A

## Impacted Components
- scripts/scaffold.py (new idempotent scaffolding CLI)
- scripts/ai_readme_helper.py (new CLI wrapper that calls LLM with prompt templates)
- templates/README.md.tpl (README template with placeholders and sections)
- templates/CONTRIBUTING.md.tpl (CONTRIBUTING template and checklist)
- templates/Dockerfile.template (multi-stage Dockerfile template)
- .github/PULL_REQUEST_TEMPLATE.md (PR template)
- .pre-commit-config.yaml (pre-commit configuration)
- docs/SECURITY_AND_SECRETS.md (detailed security & secrets handling guidance)
- config/ai/prompts/readme_helpers/* (prompt templates)
- config/ai/guardrails.yaml (guardrails config)
- scripts/verify_repo.sh (verify lint + license checks)
- LICENSE, CODE_OF_CONDUCT.md, .gitignore (templates created in repo root)

## Implementation Plan
- Create templates for README, CONTRIBUTING, Dockerfile.template, .gitignore, LICENSE, CODE_OF_CONDUCT with tokens for project metadata. Use Jinja2 for safe templating and substitution.
- Implement scripts/scaffold.py (Python CLI using click):
  - Features: --dry-run, --force, --non-interactive, --project-name, --author.
  - Idempotent: detect existing files; when overwrite would occur, write a .bak and generate a diff summary using difflib; require confirmation unless --force.
  - OS detection: explicit handling for Windows (print guidance + recommended WSL flow).
  - Token detection: check environment for CI tokens; if missing, skip remote-only steps and log warnings.
- Implement scripts/ai_readme_helper.py (Python CLI using click and OpenAI client):
  - Load prompt templates from config/ai/prompts/readme_helpers/.
  - Validate inputs, sanitize via guardrails logic (config/ai/guardrails.yaml).
  - Enforce token budget and response max_token limits.
  - On API failure or low-confidence detection, fall back to local README template rendering or return structured suggested content.
  - Log prompts/responses to logs/ai_prompts.log with PII redaction.
- Add docs/SECURITY_AND_SECRETS.md with:
  - Local dev secret patterns (.env.example, envsubst, local secret stubs)
  - CI tokens required and how to create them
  - How to obfuscate secrets, run secret scans, and use pass-through or mock credentials in local dev.
- Add config/ai/prompts/readme_helpers/README_PROMPT.md and PR_PROMPT.md with examples and placeholders.
- Add config/ai/guardrails.yaml describing input validation rules (max length, forbidden patterns, PII filters), token limits, and logging policies.
- Add scripts/verify_repo.sh to run basic lint (flake8/black), license-check, and exit non-zero if failures—used in README verification step.
- Add .github/workflows/placeholder ci workflow file with comments and badges placeholders referenced in README.
- Write unit tests for templating and scaffolding dry-run behavior and for AI helper fallback logic (mock OpenAI responses).

## Current Project State
- app/ (placeholder)
- Server/ (placeholder)
- scripts/ (may contain legacy utilities)
- templates/ (empty or minimal)
- docs/ (may contain onboarding docs)
- .propel/context/tasks/EP-TECH/us_001/us_001.md (user story location)

Note: The tree above is a placeholder; actual structure should be updated as the dependent tasks run and files are created.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | scripts/scaffold.py | Idempotent scaffolding CLI that creates template files and directory skeleton; supports --dry-run, --force, --non-interactive. |
| CREATE | scripts/ai_readme_helper.py | Small CLI to generate README/PR snippets via OpenAI using prompt templates with guardrails and fallback logic. |
| CREATE | templates/README.md.tpl | README template with sections: prerequisites, local setup, tests, CI expectations, secrets handling, verification step. |
| CREATE | templates/CONTRIBUTING.md.tpl | CONTRIBUTING template with checklist, branch/PR/commit conventions, and linking to PR template. |
| CREATE | templates/Dockerfile.template | Dockerfile template (multi-stage) with comments and placeholders. |
| CREATE | templates/.gitignore | Common .gitignore for Python/Node and typical artifacts. |
| CREATE | .github/PULL_REQUEST_TEMPLATE.md | PR template referenced by CONTRIBUTING.md. |
| CREATE | .github/workflows/ci-placeholder.yml | Placeholder GitHub Actions workflow with comments & sample steps. |
| CREATE | .pre-commit-config.yaml | Pre-commit configuration with hooks for formatters/linters. |
| CREATE | docs/SECURITY_AND_SECRETS.md | Detailed security and secrets handling guidance, local stub instructions, Windows guidance, and CI token docs. |
| CREATE | docs/README_AI_HELPERS.md | Documentation for how to use the AI helper CLI and when to prefer generated vs. local templates. |
| CREATE | config/ai/prompts/readme_helpers/README_PROMPT.md | Prompt template for README generation. |
| CREATE | config/ai/prompts/readme_helpers/PR_PROMPT.md | Prompt template for PR description generation. |
| CREATE | config/ai/guardrails.yaml | Guardrails definition (input sanitization, token budgets, PII rules). |
| CREATE | scripts/verify_repo.sh | Script to run lint, license-check, and smoke tests for verification step. |
| CREATE | logs/.gitkeep | Ensure logs directory exists for AI audit logs. |
| CREATE | LICENSE | Placeholder LICENSE file (MIT by default) |
| CREATE | CODE_OF_CONDUCT.md | Basic code of conduct template |
| CREATE | .gitignore | Root .gitignore (if not created in templates) |
| MODIFY | README.md (if present) | Add references to CI badges (placeholders), CONTRIBUTING, verification steps, and link to docs/SECURITY_AND_SECRETS.md. |
| MODIFY | .propel/context/tasks/EP-TECH/us_001/us_001.md | Update traceability or add note linking to created AI helper artifacts (optional). |

## External References
- OpenAI API docs: https://platform.openai.com/docs
- Pre-commit: https://pre-commit.com/
- GitHub Actions: https://docs.github.com/actions
- Example repository scaffolds and templates: https://github.com/github/gitignore, https://opensource.guide/
- Secret scanning best practices: https://owasp.org/www-project-top-ten/

## Build Commands
- Setup Python virtualenv and install deps:
  - python3.11 -m venv .venv && source .venv/bin/activate
  - pip install -r scripts/requirements.txt
- Run scaffolder dry-run:
  - python scripts/scaffold.py --dry-run --project-name "myproj"
- Generate README via AI helper (requires OPENAI_API_KEY):
  - python scripts/ai_readme_helper.py --project-name "myproj" --audience "backend engineer"
- Run verification script:
  - bash scripts/verify_repo.sh
- See ../.propel/build/ for platform-specific build and CI wrapper commands.

## Implementation Validation Strategy
- [ ] Unit tests for scaffolding templating and idempotency pass.
- [ ] Integration tests: scaffolding CLI creates expected file list on a temporary directory.
- [ ] AI helper tests: mock OpenAI responses to validate fallback behavior and guardrails.
- [ ] Visual/manual check: README.md contains required sections and links to docs/SECURITY_AND_SECRETS.md.
- [ ] Security checks: verify verify_repo.sh performs lint and license-check step successfully in CI placeholder.
- [ ] AI Tasks: Prompt templates validated with test inputs.
- [ ] AI Tasks: Guardrails tested for input sanitization and output validation.
- [ ] AI Tasks: Fallback logic tested with simulated API failures and low-confidence outputs.
- [ ] AI Tasks: Token budget enforcement verified.
- [ ] AI Tasks: Audit logging verified (no PII stored in logs).

## Implementation Checklist
- Effort estimate: 6 hours
- [ ] Create Jinja2 templates under templates/ and ensure placeholders for project metadata.
- [ ] Implement scripts/scaffold.py with dry-run, --force, idempotency, Windows detection, and backup/diff behavior.
- [ ] Implement scripts/ai_readme_helper.py that calls OpenAI GPT-4-turbo using config/ai/prompts/readme_helpers/, applies guardrails, logs prompts/responses, and falls back to templates.
- [ ] Add docs/SECURITY_AND_SECRETS.md documenting secrets handling, local stubs, Windows/WSL guidance, and CI token instructions.
- [ ] Add .github/PULL_REQUEST_TEMPLATE.md, .pre-commit-config.yaml, and .github/workflows/ci-placeholder.yml and reference them in CONTRIBUTING.md and README template.
- [ ] Add scripts/verify_repo.sh implementing basic lint & license checks and reference it in README verification step.
- [ ] Write unit tests for templating, CLI dry-run behavior, and AI helper fallback; mock external calls.
- [ ] Run tests and perform manual verification of generated scaffold in a temp repo; verify guardrails and logging.

## RULES:
- Implementation Checklist has 8 items (<=8).
- Effort estimate is 6 hours (<=8 hours).
- Acceptance Criteria are taken from the parent User Story (us_001).

