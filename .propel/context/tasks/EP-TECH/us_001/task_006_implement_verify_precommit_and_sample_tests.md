# Task - [TASK_006]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-TECH/us_001/us_001.md]
- Acceptance Criteria:  
    - Given an empty/new repository template, When the scaffolding is applied, Then the repository root contains: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, a clear directory structure (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/), and a Dockerfile.template.
    - Given the repository is created, When a developer reads README.md, Then it documents: prerequisites, local setup commands, how to run unit tests, how to run the service locally (with example commands), how to run integration tests (or use mocks), how to run linters/formatters, how to contribute (branching, PR template, commit message conventions), and where to find operational runbooks and CI expectations.
    - Given a developer follows the documented local setup on a supported platform (Linux or macOS), When they execute the documented commands, Then they can: install dev dependencies, run unit tests, and start the service locally within 30 minutes using only the documented steps and using test/stub credentials where applicable.
    - Given a contributor opens a pull request, When they follow CONTRIBUTING.md, Then there is a CONTRIBUTING checklist and PR template referenced in docs, and the README references the CI pipeline, expected static analysis/security scan behavior, and how to add/obfuscate secrets for local development (using environment files or local secret stubs).
    - Given repository governance requirements, When the template is used to create a new service repository, Then CI badges (placeholder allowed), pre-commit config, a basic pre-configured GitHub Actions workflow folder (.github/workflows/) and documentation for running the pipeline locally exist, and the README includes a verification step that the repo passes a basic lint and license-check as described.
- Edge Case:
    - What happens when a developer is on Windows (boundary condition)? README includes explicit Windows guidance and recommends WSL2 or Docker-based development; scaffolding scripts detect unsupported OS and print clear guidance rather than failing silently.
    - How does system handle missing CI tokens or permissions (error scenario)? Documentation outlines required CI tokens and permissions; local dev flow uses mocked/stub credentials and scaffolding scripts skip remote-only steps with clear warnings if tokens are absent.
    - What happens when scaffold generation conflicts with existing files? Scaffolding script is idempotent and will not overwrite existing files without explicit confirmation; it produces a diff/backup and logs resolution steps for the developer.

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
| Backend | N/A | N/A |
| Database | N/A | N/A |
| Library | pre-commit, pytest, flake8, license-expression | pre-commit v3.x; pytest 7.x; flake8 6.x; license-expression/scan-basic (or use scancode-lite) |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

**Note**: All code and libraries MUST be compatible with the versions above.

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
Create a repository verification script and local developer checks to validate that a newly scaffolded repository meets basic governance and developer-experience requirements. Deliverables:
- scripts/verify_repo.sh (POSIX shell) and scripts/verify_repo.py (optional Python alternative) that:
  - verify presence of required files and directories,
  - run fast linters (flake8), run pre-commit in local mode, and run a minimal license check,
  - execute a sample unit test suite (tests/test_sample.py) to validate test runner and CI checks.
  - print a clear report and exit with non-zero only on fatal checks.
- Add .pre-commit-config.yaml configured with at least basic hooks (trailing-whitespace, end-of-file fixer, flake8, check-yaml, check-added-large-files).
- Add tests/test_sample.py with one passing unit test and one parametrized example that exercises pytest discovery.
- Update README.md to document local verification step and how to run the script and install pre-commit hooks.

Purpose: enable developers to locally validate that the repo scaffold is correct and to provide a quick "local CI" smoke test for linters/licenses/tests.

## Dependent Tasks
- Artefacts/Tasks required before starting:
  - Ensure repo skeleton exists (or the scaffolding task that created templates): README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, Dockerfile.template, and base directories (src/, tests/, docs/, .github/workflows/, scripts/).
  - Provide template linter configuration (e.g., .flake8) if not present (verify script will fallback if absent).
  - Add scripts/ directory to repository (if missing).

## Impacted Components
- scripts/verify_repo.sh (new)
- scripts/verify_repo.py (optional, new)
- .pre-commit-config.yaml (new)
- tests/test_sample.py (new)
- README.md (modify - add "Verification" section)
- .flake8 (optional create/modify to ensure flake8 baseline)
- docs/verification.md (optional create - referenced by README)

## Implementation Plan
- Create a POSIX-compliant shell script at scripts/verify_repo.sh that:
  - Detects OS and prints guidance on unsupported/Windows recommending WSL2/Docker.
  - Verifies presence of a defined list of files and directories; for each missing item print recommendation and mark as failed.
  - Detects if pre-commit is installed; if not, print install steps and run basic linters directly (flake8) if available.
  - Runs pre-commit run --all-files (if pre-commit available) in fast mode and captures non-zero status.
  - Runs flake8 (if present) targeting src/ and tests/ with a short timeout and prints summary.
  - Performs a basic license check by scanning LICENSE file presence and checking LICENSE contains SPDX identifier (simple grep), or warns if absent.
  - Runs pytest -q tests/test_sample.py and captures exit code.
  - Return aggregate exit code: non-zero if any fatal checks failed; non-fatal warnings should not fail script.
  - Support flags: --ci (non-interactive), --fix (run auto-fix hooks where safe), --skip-tests.
- Create .pre-commit-config.yaml with standard hooks:
  - end-of-file-fixer, trailing-whitespace, check-yaml, check-added-large-files, flake8.
- Add tests/test_sample.py with a minimal passing unit test and one example that fails intentionally only when an environment variable TEST_FAIL is set (to test CI failure mode).
- Modify README.md to include a "Verification" section with commands to run the verification script, how to install pre-commit and run it, and Windows guidance recommending WSL2/Docker.
- Add minimal .flake8 config if not present (max-line-length, ignore list).
- Include simple CI-friendly invocation in docs: scripts/verify_repo.sh --ci

## Current Project State
- Placeholder project structure (to be updated based on dependent tasks):
  - README.md
  - CONTRIBUTING.md
  - LICENSE
  - CODE_OF_CONDUCT.md
  - .gitignore
  - Dockerfile.template
  - src/
  - tests/
  - docs/
  - .github/
    - workflows/
  - scripts/
  - .flake8 (may or may not exist)

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | scripts/verify_repo.sh | POSIX-compliant verification script that checks files/dirs, runs pre-commit/flake8/license check and sample pytest tests; supports --ci, --fix, --skip-tests flags. |
| CREATE | .pre-commit-config.yaml | Pre-commit configuration with hooks: end-of-file-fixer, trailing-whitespace, check-yaml, check-added-large-files, flake8. |
| CREATE | tests/test_sample.py | Minimal pytest test file with one passing test and one conditional failing test controlled by env var TEST_FAIL. |
| CREATE | tests/__init__.py | Empty init to mark tests package (optional but explicit). |
| MODIFY | README.md | Add "Verification" section documenting how to run scripts/verify_repo.sh, pre-commit installation, Windows guidance (WSL2/Docker), and expected outputs. |
| CREATE | .flake8 | Baseline flake8 config (max-line-length=88, ignore E203,W503) — only create if absent. |
| MODIFY | .github/workflows/ci-placeholder.yml | (optional) add/ensure CI workflow calls scripts/verify_repo.sh --ci (create if placeholder missing). |

## External References
- pre-commit framework: https://pre-commit.com/
- pytest docs: https://docs.pytest.org/
- flake8 docs: https://flake8.pycqa.org/
- SPDX license list: https://spdx.org/licenses/
- Example verify script patterns: https://stackoverflow.com/questions/ (generic patterns), internal company examples (if available)

## Build Commands
- Install dev dependencies (Linux/macOS/WSL):
  - python3 -m venv .venv && source .venv/bin/activate
  - pip install --upgrade pip
  - pip install pre-commit pytest flake8
- Install pre-commit hooks:
  - pip install pre-commit
  - pre-commit install
- Run verification script:
  - bash scripts/verify_repo.sh
  - bash scripts/verify_repo.sh --ci
- Run sample tests directly:
  - pytest -q tests/test_sample.py

(Refer to applicable technology stack specific build commands in ../.propel/build/)

## Implementation Validation Strategy
- [ ] Unit tests pass (pytest -q)
- [ ] Integration tests pass (run scripts/verify_repo.sh --ci)
- [ ] **[UI Tasks]** Visual comparison against wireframe completed at 375px, 768px, 1440px (N/A)
- [ ] **[UI Tasks]** Run `/analyze-ux` to validate wireframe alignment (N/A)
- [ ] **[AI Tasks]** Prompt templates validated with test inputs (N/A)
- [ ] **[AI Tasks]** Guardrails tested for input sanitization and output validation (N/A)
- [ ] **[AI Tasks]** Fallback logic tested with low-confidence/error scenarios (N/A)
- [ ] **[AI Tasks]** Token budget enforcement verified (N/A)
- [ ] **[AI Tasks]** Audit logging verified (N/A)

## Implementation Checklist
- [ ] Create scripts/verify_repo.sh implementing file checks, pre-commit/flake8/license/test flow and flags (--ci, --fix, --skip-tests). (Estimated 3.0 hours)
- [ ] Add .pre-commit-config.yaml with the listed hooks and test locally (pre-commit run --all-files). (Estimated 1.0 hour)
- [ ] Add tests/test_sample.py (and tests/__init__.py) with one passing test and one conditional-fail test; run pytest to validate. (Estimated 0.5 hour)
- [ ] Add or update .flake8 with baseline config if absent. (Estimated 0.25 hour)
- [ ] Modify README.md: add "Verification" section with commands, Windows guidance, and CI note. (Estimated 0.5 hour)
- [ ] Validate verify script behavior on Linux and macOS; print WSL2 guidance on Windows. (Estimated 1.0 hour)
- [ ] Run end-to-end validation: install pre-commit hooks, run scripts/verify_repo.sh --ci to ensure exit codes and outputs match expectations. (Estimated 1.75 hours)

Effort: <=8 hours total (sum of estimates ~7.0 hours)

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Effort estimated <=8 hours.

