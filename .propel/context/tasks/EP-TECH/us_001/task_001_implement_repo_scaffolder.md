# Task - task_001

## Requirement Reference
- User Story: us_001 (extracted from input)
- Story Location: .propel/context/tasks/EP-TECH/us_001/us_001.md
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
| Backend | N/A | N/A |
| Database | N/A | N/A |
| Library | Python (click) | 3.11 / click 8.1.x |
| Library | Git integration | GitPython 3.x |
| Library | Diff/patch | python-Levenshtein / difflib (stdlib) |
| AI/ML | N/A | N/A |
| Vector Store | N/A | N/A |
| AI Gateway | N/A | N/A |

Note: Implementation will use Python 3.11 for the CLI/tooling. Libraries chosen (click, GitPython) must be compatible with Python 3.11+. Tests use pytest 7.x.

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
Implement an idempotent repository scaffolding CLI (Python-based) that applies file/templates to a target repository, creates the required directory skeleton and files (README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, Dockerfile.template, etc.), handles conflicts by producing backups and a diff summary, supports dry-run and --force modes, detects OS (Linux/macOS/Windows/WSL) and prints platform-appropriate guidance, and safely skips remote-only operations (like attempting to push or add CI tokens) if credentials are missing. The CLI should be friendly for both interactive developer use and non-interactive CI usage.

## Dependent Tasks
- Artefacts/Templates: Provide canonical template content (placeholders allowed) for README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, Dockerfile.template, .github/workflows/basic.yml, and pre-commit config in templates/ (If not provided, this task will create baseline templates as part of implementation).
- EP-TECH: Confirm license type and preferred CONTRIBUTING checklist items (to populate templates).
- N/A: No other blocking engineering tasks; scaffolding must be designed to be idempotent so it can run on existing repos.

## Impacted Components
- New module: scripts/scaffold.py (main CLI)
- New package: templates/ (file templates and tokens)
- New helper: scripts/scaffold_utils.py (file operations, diff/backup, OS detection)
- New config: requirements-dev.txt (for developer dependencies)
- New verification script: scripts/verify_scaffold.py (basic smoke/verification run)
- Repo top-level: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, Dockerfile.template (created by tool)

## Implementation Plan
- Initialize Python CLI (+ packaging basics):
  - Create scripts/scaffold.py using click to provide commands: init, apply, verify. Flags: --dry-run, --force, --yes, --non-interactive, --backup-dir.
  - Add OS detection logic using platform and environment detection for WSL (check /proc/version or environment variables).
- File/template engine:
  - Place templates in templates/ with simple Jinja2-like token replacement using str.format or Python Template (stdlib) to avoid heavy dependencies.
  - Implement mapping replacements: {PROJECT_NAME}, {MAINTAINER}, {YEAR}, etc.
- Idempotency and conflict handling:
  - For each target path, check existence. If missing create. If exists compare contents; if identical skip. If differs, create timestamped backup in backups/ or user-provided backup-dir, emit unified diff (difflib.unified_diff) to stdout and to a scaffold.log file, and prompt (unless --force or --yes) to overwrite, keep both, or skip.
  - Ensure --dry-run shows what would be changed without writing files.
- OS detection & guidance:
  - Detect Windows native vs WSL and print actionable guidance; for Windows native, do not attempt to run Linux/macOS-specific scripts — instead print recommended steps and exit non-fatally.
- Token/CI handling:
  - Detect presence of environment variables for known CI tokens (GITHUB_TOKEN, GH_TOKEN). If missing, skip remote-only steps and log warning. Do not fail the whole run.
- Verification and smoke tests:
  - Create scripts/verify_scaffold.py to run lint check placeholder (e.g., run pre-commit or flake8 if available) and license check (presence of LICENSE) — this script must be safe to run locally and used in acceptance validation.
- Logging and tests:
  - Add unit tests under tests/ for: dry-run behavior, conflict detection and backup creation, OS detection, template rendering, and non-interactive mode behavior.
  - Add requirements-dev.txt with pytest and click for contributors.

## Current Project State
- (Placeholder project root)
/
├── scripts/ (new, will contain scaffold.py, scaffold_utils.py, verify_scaffold.py)
├── templates/ (new, will contain README.md.template, CONTRIBUTING.md.template, LICENSE.template, CODE_OF_CONDUCT.md.template, Dockerfile.template, .gitignore.template, .github workflows)
├── tests/ (new, unit tests for the scaffolder)
├── requirements-dev.txt (new)
└── .propel/context/tasks/EP-TECH/us_001/us_001.md (user story)

Note: This is a placeholder structure. The task will create the listed files and folders if they do not exist.

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | scripts/scaffold.py | Main CLI entrypoint implementing init/apply/verify with flags (--dry-run, --force, --yes, --non-interactive). |
| CREATE | scripts/scaffold_utils.py | Helper functions: file write/backup/diff, OS detection (Windows/WSL/Linux/macOS), token checks, logging utilities. |
| CREATE | scripts/verify_scaffold.py | Simple verification script that checks for required files and runs local lint/license checks (placeholder commands). |
| CREATE | templates/README.md.template | README template with sections: prerequisites, local setup, running tests, integration tests (mocks), linters, contributing, runbooks, CI expectations, Windows guidance. |
| CREATE | templates/CONTRIBUTING.md.template | CONTRIBUTING template with checklist, PR template references, commit message conventions, branch naming. |
| CREATE | templates/PR_TEMPLATE.md | .github/PULL_REQUEST_TEMPLATE.md template referenced by CONTRIBUTING. |
| CREATE | templates/LICENSE.template | LICENSE placeholder (MIT or org default) with token replacements. |
| CREATE | templates/CODE_OF_CONDUCT.md.template | Code of Conduct template. |
| CREATE | templates/.gitignore.template | Common entries for Python/Node/build artifacts. |
| CREATE | templates/Dockerfile.template | Dockerfile.template with multi-stage build placeholders and comments. |
| CREATE | templates/.github/workflows/basic.yml | Placeholder GitHub Actions workflow demonstrating lint/test steps and badges placeholder. |
| CREATE | requirements-dev.txt | Developer dependencies (click, GitPython, pytest). |
| CREATE | tests/test_scaffold.py | Unit tests for core behaviors (dry-run, conflict handling, OS detection, template rendering). |
| CREATE | scaffold.log | (created at run-time) Logs of operations and diffs (tool will produce when run). |
| MODIFY | .propel/context/tasks/EP-TECH/us_001/us_001.md | (Optional) No change required; included for traceability. |
| CREATE | docs/onboarding_checklist.md | Short onboarding checklist produced from README template (deliverable). |

## External References
- Python packaging and click: https://click.palletsprojects.com/
- GitPython (git operations): https://gitpython.readthedocs.io/
- difflib (stdlib unified_diff): https://docs.python.org/3/library/difflib.html
- Pre-commit framework: https://pre-commit.com/
- GitHub Actions starter workflows: https://docs.github.com/en/actions/using-workflows
- WSL detection guidance: https://docs.microsoft.com/en-us/windows/wsl/about
- Example repository scaffolding references and best practices: https://opensource.guide/ and https://about.sourcegraph.com/blog/how-to-create-a-repository-template/

## Build Commands
- Create venv: python -m venv .venv
- Activate venv:
  - Linux/macOS: source .venv/bin/activate
  - Windows (PowerShell): .venv\Scripts\Activate.ps1
- Install dev deps: pip install -r requirements-dev.txt
- Run scaffold (dry-run): python scripts/scaffold.py apply --target . --dry-run
- Run scaffold (interactive): python scripts/scaffold.py apply --target .
- Run verify: python scripts/verify_scaffold.py
- Run tests: pytest -q

Refer to ../.propel/build/ for project-wide build steps (if present).

## Implementation Validation Strategy
- [ ] Unit tests pass (pytest)
- [ ] Integration tests pass (simulate a target repo in tmpdir)
- [ ] Dry-run shows intended changes without writing files
- [ ] Conflict handling creates timestamped backups and produces unified diffs in scaffold.log
- [ ] OS detection correctly identifies Windows native vs WSL vs Linux/macOS and prints actionable guidance
- [ ] CLI supports --force to overwrite without prompts and non-interactive mode for CI
- [ ] Verification script confirms expected files exist and runs placeholder lint/license checks

## Implementation Checklist
- (Effort estimate: <=8 hours)
1. Implement scripts/scaffold.py with click commands and flags (--dry-run, --force, --yes, --non-interactive) and basic logging. (2.0h)
2. Implement scripts/scaffold_utils.py with functions: render_template(), write_with_backup(), create_unified_diff(), detect_os_and_wsl(), check_ci_tokens(). (1.5h)
3. Add templates/ files (README.md.template, CONTRIBUTING.md.template, Dockerfile.template, LICENSE.template, CODE_OF_CONDUCT.md.template, .gitignore.template, .github/workflows/basic.yml, PR_TEMPLATE.md). Populate with required sections and placeholders. (1.5h)
4. Add scripts/verify_scaffold.py that checks presence of required files and runs placeholder lint/license checks. (0.5h)
5. Add tests/test_scaffold.py to assert dry-run, conflict behavior (backup & diff), template rendering, and OS detection. (1.0h)
6. Create requirements-dev.txt and update README/instructions for developers to run the CLI and tests. (0.5h)
7. Perform manual validation: run CLI in an empty temp repo and in a repo with conflicting files; verify backups and diffs and Windows/WSL guidance (interactive and non-interactive modes). (0.5h)

## RULES:
- Implementation Checklist contains 7 items (<=8).
- Estimated effort <=8 hours.
- Checklist items are specific and actionable.

