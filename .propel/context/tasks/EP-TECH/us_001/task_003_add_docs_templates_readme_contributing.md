# Task - task_003

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:  
    - Given an empty/new repository template, When the scaffolding is applied, Then the repository root contains: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, a clear directory structure (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/), and a Dockerfile.template.
    - Given the repository is created, When a developer reads README.md, Then it documents: prerequisites, local setup commands, how to run unit tests, how to run the service locally (with example commands), how to run integration tests (or use mocks), how to run linters/formatters, how to contribute (branching, PR template, commit message conventions), and where to find operational runbooks and CI expectations.
    - Given a developer follows the documented local setup on a supported platform (Linux or macOS), When they execute the documented commands, Then they can: install dev dependencies, run unit tests, and start the service locally within 30 minutes using only the documented steps and using test/stub credentials where applicable.
    - Given a contributor opens a pull request, When they follow CONTRIBUTING.md, Then there is a CONTRIBUTING checklist and PR template referenced in docs, and the README references the CI pipeline, expected static analysis/security scan behavior, and how to add/obfuscate secrets for local development (using environment files or local secret stubs).
    - Given repository governance requirements, When the template is used to create a new service repository, Then CI badges (placeholder allowed), pre-commit config, a basic pre-configured GitHub Actions workflow folder (.github/workflows/) and documentation for running the pipeline locally exist, and the README includes a verification step that the repo passes a basic lint and license-check as described.
- Edge Case:
    - README includes explicit Windows guidance and recommends WSL2 or Docker-based development; scaffolding/setup scripts detect unsupported OS and print clear guidance rather than failing silently.
    - Documentation outlines required CI tokens and permissions; local dev flow uses mocked/stub credentials and scaffolding scripts skip remote-only steps with clear warnings if tokens are absent.
    - Scaffolding script is idempotent and will not overwrite existing files without explicit confirmation; it produces a diff/backup and logs resolution steps for the developer.

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

> **Wireframe Status Legend:**
> - **AVAILABLE**: Local file exists at specified path
> - **PENDING**: UI-impacting task awaiting wireframe (provide file or URL)
> - **EXTERNAL**: Wireframe provided via external URL
> - **N/A**: Task has no UI impact
>
> If UI Impact = No, all design references should be N/A

### **CRITICAL: Wireframe Implementation Requirement (UI Tasks Only)**
IF Wireframe Status = AVAILABLE or EXTERNAL: N/A (UI Impact = No)

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | React | 18.x |
| Backend | N/A | N/A |
| Database | N/A | N/A |
| Library | Node (tooling) | 18.x |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code, and libraries, MUST be compatible with versions above.

## AI References (AI Tasks Only)
| Reference Type | Value |
|----------------|-------|
| **AI Impact** | No |
| **AIR Requirements** | N/A |
| **AI Pattern** | N/A |
| **Prompt Template Path** | N/A |
| **Guardrails Config** | N/A |
| **Model Provider** | N/A |

### **CRITICAL: AI Implementation Requirement (AI Tasks Only)**
IF AI Impact = Yes: N/A

## Task Overview
Add a set of repository documentation templates and supporting template artifacts to the project so new repositories started from this template will include standardized developer-facing docs and CI/DevEx placeholders. Deliver README.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, LICENSE, .gitignore templates, Dockerfile.template, a PULL_REQUEST_TEMPLATE, pre-commit config, basic .github/workflows placeholder, and docs/onboarding checklist sections covering prerequisites, setup, tests, linters, CI expectations, secrets handling, Windows guidance, and verification steps described in the acceptance criteria.

## Dependent Tasks
- None (this task implements repository templates described by parent user story us_001). If a separate scaffolding script task exists, integrate with it later; this deliverable is the template set required by that script.

## Impacted Components
- New files under repository root and .github:
  - templates/README.md.tpl
  - templates/CONTRIBUTING.md.tpl
  - templates/CODE_OF_CONDUCT.md.tpl
  - templates/LICENSE.tpl
  - templates/.gitignore.tpl
  - Dockerfile.template
  - .github/PULL_REQUEST_TEMPLATE.md
  - .github/workflows/ci-placeholder.yml
  - .pre-commit-config.yaml
  - docs/ONBOARDING.md
  - docs/CI_AND_SECURITY.md

## Implementation Plan
- Create a templates/ directory containing parameterized markdown files (placeholders for project name, license, maintainer contact) for README, CONTRIBUTING, CODE_OF_CONDUCT, LICENSE, and .gitignore. Use clear sections and example commands.
- Add Dockerfile.template with comments and multi-stage example appropriate for frontend Node/React projects; include guidance in README about replacing placeholders.
- Add .github/PULL_REQUEST_TEMPLATE.md to be referenced by CONTRIBUTING.md and README; include checklist (tests, lint, security scan, changelog).
- Add .github/workflows/ci-placeholder.yml: a minimal workflow file with placeholder jobs: lint, test, license-check; include badges placeholders in README.
- Add .pre-commit-config.yaml with common hooks (trailing-whitespace, end-of-file-fixer, black/ruff or prettier as examples) and instructions in README for installing pre-commit.
- Add docs/ONBOARDING.md with step-by-step local setup for Linux/macOS, Windows guidance recommending WSL2/Docker, stub credentials guidance, and a verification section that runs lint + license-check + unit tests.
- Add docs/CI_AND_SECURITY.md outlining required CI tokens, how to obfuscate secrets locally (.env.example), and how scaffolding scripts should behave when tokens are absent.
- Ensure all templates include idempotency and conflict guidance: "scaffolding must not overwrite without explicit confirmation; backup created".
- Validate content by running a smoke-check script locally: run linters and unit tests using the example commands in README (manual verification as part of task).

## Current Project State
- app/
  - (frontend sources - placeholder)
- Server/
  - (backend sources - placeholder)
- .propel/
  - context/
    - tasks/
      - EP-TECH/
        - us_001/
          - us_001.md
- (No templates/ directory currently — will be added by this task)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | templates/README.md.tpl | Parameterized README template with sections: prerequisites, setup (Linux/macOS/WSL/Docker), run service, run unit tests, run integration tests (with mocks), linters/formatters, contribution summary, CI badges placeholders, verification step. |
| CREATE | templates/CONTRIBUTING.md.tpl | CONTRIBUTING template with checklist, PR expectations, branching & commit message conventions, reference to PR template and pre-commit. |
| CREATE | templates/CODE_OF_CONDUCT.md.tpl | Standard Contributor Covenant v2.1 template with project placeholders. |
| CREATE | templates/LICENSE.tpl | SPDX-compatible MIT license template with placeholder for year and owner. |
| CREATE | templates/.gitignore.tpl | .gitignore with typical Node/React patterns and common OS/tool ignores. |
| CREATE | Dockerfile.template | Multi-stage Dockerfile template (build/runtime) with explanatory comments. |
| CREATE | .github/PULL_REQUEST_TEMPLATE.md | PR template with checklist (tests, lint, security scan, changelog, reviewers). |
| CREATE | .github/workflows/ci-placeholder.yml | Minimal GitHub Actions workflow placeholder for lint/test/license-check jobs and environment variable notes. |
| CREATE | .pre-commit-config.yaml | Basic pre-commit configuration with common hooks and install instructions in README. |
| CREATE | docs/ONBOARDING.md | Onboarding checklist for new devs: install dev deps, run tests, start service, Windows guidance, WSL2 recommendation. |
| CREATE | docs/CI_AND_SECURITY.md | Documentation on CI tokens, how to stub secrets locally (.env.example usage), and what to do if tokens are missing. |
| CREATE | .env.example | Example environment file showing stubbed/test credentials and notes about secrets obfuscation. |
| CREATE | templates/DIR_STRUCTURE.md | Short file listing recommended directory layout (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/). |
| MODIFY | .propel/context/tasks/EP-TECH/us_001/us_001.md | (Optional) Add a cross-reference note that these templates were added by task_003 (update traceability). |

## External References
- Contributor Covenant: https://www.contributor-covenant.org/
- SPDX Licenses: https://spdx.org/licenses/
- pre-commit hooks: https://pre-commit.com/
- GitHub Actions documentation: https://docs.github.com/actions
- GitHub PULL_REQUEST_TEMPLATE docs: https://docs.github.com/en/github/building-a-strong-community/about-issue-and-pull-request-templates
- Example Docker multi-stage: https://docs.docker.com/develop/develop-images/multistage-build/

## Build Commands
- Frontend (example) build commands referenced in README templates:
  - npm install
  - npm run build
  - npm test
  - npm run lint
- See .propel/build/ for project-level build orchestration: ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests pass (for example project tests if present)
- [ ] Integration tests pass (if applicable)
- [ ] Visual comparison against wireframe completed at 375px, 768px, 1440px — N/A (no UI impact)
- [ ] Run `/analyze-ux` to validate wireframe alignment — N/A
- [ ] Prompt templates validated with test inputs — N/A
- [ ] Guardrails tested for input sanitization and output validation — N/A
- [ ] Fallback logic tested with low-confidence/error scenarios — N/A
- [ ] Token budget enforcement verified — N/A
- [ ] Audit logging verified (no PII in logs) — N/A

## Implementation Checklist
- [ ] Create templates/ files (README.md.tpl, CONTRIBUTING.md.tpl, CODE_OF_CONDUCT.md.tpl, LICENSE.tpl, .gitignore.tpl, DIR_STRUCTURE.md) and verify placeholders render sensibly.
- [ ] Add operational docs (docs/ONBOARDING.md, docs/CI_AND_SECURITY.md, .env.example) including Windows guidance and CI token handling instructions.
- [ ] Add GitHub artifacts: .github/PULL_REQUEST_TEMPLATE.md and .github/workflows/ci-placeholder.yml and ensure CONTRIBUTING.md references them.
- [ ] Add Dockerfile.template and .pre-commit-config.yaml and include install/usage instructions in README template.
- [ ] Validate README example commands locally (npm install, npm test, npm run lint) on Linux/macOS and verify instructions for WSL2/Docker are clear (manual smoke test).
- [ ] Ensure templates include idempotency/conflict guidance and instructions for scaffolding scripts (backup/diff/prompt).
- [ ] Commit all created files with a clear commit message and update .propel/context/tasks/EP-TECH/us_001/us_001.md with a cross-reference note.
- [ ] Review templates for compliance with acceptance criteria and have one peer review (document reviewer) sign-off.

Notes:
- Estimated effort: 6-8 hours.
- Keep templates generic and language-agnostic where possible; provide explicit examples for a Node/React frontend repo.

## RULES:
- Implementation Checklist contains 8 items (<=8).
- Effort estimate is <=8 hours.

