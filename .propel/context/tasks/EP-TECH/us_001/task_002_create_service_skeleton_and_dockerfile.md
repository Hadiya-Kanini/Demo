# Task - [task_002]

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
| Backend | Python (FastAPI) | 3.11 / FastAPI >=0.95 |
| Database | N/A | N/A |
| Library | Uvicorn / Gunicorn | uvicorn 0.22+, gunicorn 22+ |
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

## Task Overview
Add a minimal service skeleton under src/, a multi-stage Dockerfile.template that uses a non-root runtime user, a docker-compose.dev.yml example for local development, and portable run_local scripts for Linux/macOS and a PowerShell helper for Windows. The purpose is to provide a reproducible local developer experience that satisfies the repository scaffold requirements in us_001 and can be extended by other scaffolding tasks.

## Dependent Tasks
- Artefacts/Tasks: None mandatory. This task is self-contained for initial skeleton and templates.
- Note: Higher-level documentation and CONTRIBUTING.md updates are covered in us_001 but not required to complete this task.

## Impacted Components
- New/Updated modules and files:
  - src/app/main.py (new)
  - src/app/api/v1/health.py (new)
  - src/app/config.py (new)
  - src/app/__init__.py (new)
  - requirements.txt (new)
  - Dockerfile.template (new)
  - docker-compose.dev.yml (new)
  - scripts/run_local.sh (new)
  - scripts/run_local.ps1 (new)
  - scripts/bootstrap_dev_env.sh (new)
  - .gitignore (create if not present)
  - README.md (small "How to run locally" snippet may be added or referenced; primary README updates are part of us_001 but minimal guidance will be included in README section created here)

## Implementation Plan
- Create minimal FastAPI-based service skeleton:
  - src/app/main.py: ASGI app with a single /health endpoint and simple startup/shutdown handlers.
  - src/app/api/v1/health.py: health-check route returning JSON status.
  - src/app/config.py: environment-config loader using pydantic BaseSettings (simple).
  - requirements.txt with pinned minimal dependencies (fastapi, uvicorn, pydantic).
- Author Dockerfile.template:
  - Multi-stage build: builder stage (install deps, run tests optionally), final stage using slim runtime base image.
  - Create a non-root runtime user and set proper file permissions.
  - Include ARG placeholders for PYPI extra index or build args.
  - Include comments and usage examples in template header.
- Create docker-compose.dev.yml:
  - Compose file bringing up service on port 8000, mounting local src/ for code reload, environment variable examples, and a depends_on for optional local services.
  - Service uses Dockerfile.template via a generated Dockerfile (documentation shows how to render or build using --build-arg).
- Create run_local scripts:
  - POSIX script scripts/run_local.sh that:
    - Detects OS (Linux/macOS), warns on Windows and suggests using WSL2 or scripts/run_local.ps1.
    - Creates a virtualenv, installs dev deps from requirements.txt, and starts uvicorn pointing to src.app.main:app.
    - Accepts --reload flag to use uvicorn --reload.
    - Provides dry-run and verbose modes.
  - PowerShell script scripts/run_local.ps1 for Windows/PowerShell/WSL usage with similar behavior.
  - scripts/bootstrap_dev_env.sh to set up .env.example -> .env and handle missing CI tokens by creating stub values and printing warnings.
- Include minimal .gitignore and README fragment:
  - .gitignore contains common entries for Python and local Docker artifacts.
  - README snippet in repository root (or a README service section) describing how to run the service locally with the scripts and how to build the Docker image using the Dockerfile.template.
- Unit smoke test file template (tests/test_health.py) to demonstrate running pytest in CI and locally (very small).
- Ensure all created files are idempotent: scripts must not overwrite existing files without confirmation. Scripts should detect if files exist and back them up or print diff instructions.

## Current Project State
- Placeholder tree (update after dependent tasks complete):
  - .
    - .propel/
    - src/  <-- currently may not exist; will be created
    - scripts/ <-- will be created
    - tests/ <-- will be created
    - Dockerfile.template <-- will be created
    - docker-compose.dev.yml <-- will be created
    - requirements.txt <-- will be created
    - README.md <-- may exist; will have service section added if missing

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | src/app/main.py | Minimal ASGI FastAPI app exposing health endpoint and startup/shutdown hooks. |
| CREATE | src/app/api/v1/health.py | /health route implementation returning {"status":"ok","uptime":n}. |
| CREATE | src/app/config.py | Simple pydantic BaseSettings config loader reading env vars. |
| CREATE | src/app/__init__.py | Package marker for app. |
| CREATE | requirements.txt | Minimal pinned dependencies: fastapi, uvicorn, pydantic, pytest (for tests). |
| CREATE | Dockerfile.template | Multi-stage Dockerfile template with builder and runtime stages, non-root runtime user, ARG placeholders and comments. |
| CREATE | docker-compose.dev.yml | Developer docker-compose example with service, volume mounts for code, environment var example, and ports mapping. |
| CREATE | scripts/run_local.sh | POSIX script to create venv, install deps, and run uvicorn locally; detects unsupported OS and warns. |
| CREATE | scripts/run_local.ps1 | PowerShell script for Windows/WSL local start-up instructions. |
| CREATE | scripts/bootstrap_dev_env.sh | Creates .env from .env.example, inserts stub secrets if CI tokens absent, prints warnings. |
| CREATE | tests/test_health.py | Minimal pytest test asserting health endpoint returns status ok when called via TestClient. |
| CREATE | .gitignore | Python and docker related ignore entries (venv, __pycache__, .env, .env.local, .venv, .pytest_cache). |
| MODIFY | README.md | Add a "Run service locally" section if README exists; otherwise create README.md with minimal run instructions referencing scripts. |

## External References
- FastAPI docs - https://fastapi.tiangolo.com/
- Docker multi-stage builds - https://docs.docker.com/develop/develop-images/multistage-build/
- Dockerfile best practices - https://docs.docker.com/develop/dev-best-practices/
- docker-compose reference - https://docs.docker.com/compose/compose-file/
- Pydantic BaseSettings - https://docs.pydantic.dev/latest/usage/settings/
- Python venv docs - https://docs.python.org/3/library/venv.html

## Build Commands
- Create virtualenv and install deps:
  - python3.11 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
- Run locally with script:
  - bash scripts/run_local.sh --reload
- Run tests:
  - .venv/bin/pytest -q
- Build Docker image from template (example flow):
  - cp Dockerfile.template Dockerfile
  - docker build -t myservice:dev .
- Using docker-compose:
  - docker compose -f docker-compose.dev.yml up --build
- (Reference build commands file) ../.propel/build/

## Implementation Validation Strategy
- [ ] Unit tests (tests/test_health.py) run and pass locally via pytest.
- [ ] scripts/run_local.sh works on Linux/macOS and prints Windows guidance on Windows.
- [ ] scripts/run_local.ps1 works on Windows/PowerShell and prints WSL recommendation if native unsupported.
- [ ] Dockerfile.template builds (after copying to Dockerfile) producing a runnable image: docker build -t service:test . && docker run --rm -p 8000:8000 service:test and health endpoint returns 200.
- [ ] docker-compose.dev.yml starts service with code mount and hot-reload (uvicorn --reload).
- [ ] Scaffolding scripts (bootstrap_dev_env.sh) create .env from .env.example and insert stub tokens with clear warnings.
- [ ] Files are created in idempotent mode — running scripts again does not overwrite without confirmation (backup created or prompt).

## Implementation Checklist
- [ ] Create src/ service skeleton files (src/app/main.py, src/app/api/v1/health.py, src/app/config.py, src/app/__init__.py) and tests/test_health.py. (Estimated 2.0 hours)
- [ ] Add requirements.txt and .gitignore with recommended entries. (Estimated 0.5 hours)
- [ ] Implement Dockerfile.template (multi-stage) with non-root runtime user and clear inline comments. (Estimated 1.5 hours)
- [ ] Create docker-compose.dev.yml with volume mount, env example, and port mapping. (Estimated 0.5 hours)
- [ ] Implement scripts/run_local.sh and scripts/run_local.ps1 plus scripts/bootstrap_dev_env.sh with idempotency and platform detection. (Estimated 1.5 hours)
- [ ] Add or modify README.md to include "Run service locally" steps referencing the scripts and Dockerfile.template example. (Estimated 0.5 hours)
- [ ] Validate locally: run unit tests, run run_local script on Linux/macOS, build image from Dockerfile.template, and bring up docker-compose.dev.yml. Create backup if re-run of generation. (Estimated 0.5 hours)
- Total effort estimate: 7.0 hours

## RULES:
- Implementation Checklist must have <=8 items (7 items listed).
- Effort must be <=8 hours (7.0 hours total).
- Be specific and actionable — items above are actionable and discrete.
- Expected Changes table lists concrete file paths.

