# Unit Test Plan - [TASK_us_001_scripts]

## Requirement Reference
- **User Story**: [us_001]
- **Story Location**: [.propel/context/tasks/us_001/us_001.md]
- **Layer**: Scripts
- **Related Test Plans**: test_plan_infra_ci_precommit.md (Infra), test_plan_docs_readme_contrib.md (Docs)
- **Acceptance Criteria Covered**:
  - AC-1: Given a new repository created from the template, when the scaffold is applied, then repository root contains README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, directories (src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/), and Dockerfile.template — mapped to TC-002.
  - AC-3: Given a developer follows documented local setup on Linux/macOS, when they execute documented commands, then they can install dev deps, run unit tests, and start the service locally — mapped to TC-001 and TC-003.
  - AC-Edge (conflict/idempotency): Scaffolding does not overwrite without force; creates backups and shows diffs — mapped to TC-004.
  - AC-Edge (Windows handling): Scripts detect Windows and provide guidance without failing — mapped to TC-005.
  - AC-Edge (missing CI tokens): Scripts skip remote-only steps and log warnings — mapped to TC-006.

## Test Plan Overview
Purpose: Unit-test the scaffolding scripts responsible for creating a new repository skeleton. Scope covers CLI parsing, orchestration logic, template rendering, filesystem interactions, idempotency/conflict detection, OS detection logic, and remote-op guards. Tests are unit-only (no network or live CI), fully isolated via filesystem and environment mocking to ensure deterministic, fast execution.

Why: The scripts layer contains the primary decision logic (idempotency, safe-write, platform guards). Unit tests here provide early feedback, prevent destructive overwrites, and validate developer-facing behaviors before integration tests run.

## Dependent Tasks
- Coordinate with Docs plan to validate generated README content (docs plan will contain content-level assertions).
- Coordinate with Infra plan to validate workflow placeholders and pre-commit config insertion.
- Implementation of scaffold module with clear, testable function boundaries (CLI entrypoint delegates to orchestrator; FS operations abstracted).

## Components Under Test

| Component | Type | File Path | Responsibilities |
|-----------|------|-----------|------------------|
| CLI entrypoint | function/module | src/cli/scaffold.py | Parse args (dry-run, force, target-dir, confirm), translate flags to orchestrator calls, return machine-readable status |
| Orchestrator | class/function | src/scaffold/orchestrator.py | High-level flow: plan -> create directories -> render templates -> write files -> handle conflicts/backups |
| Template renderer | function/module | src/scaffold/templates.py | Render templates with provided context (project name, license, examples) |
| Filesystem helper | module | src/scaffold/fs.py | create_file, write_file_atomic, backup_file, exists, read_file, make_dirs — abstract FS for testability |
| Conflict detector | function | src/scaffold/conflict.py | detectExisting, generateDiff, decideOverwritePolicy |
| OS utils | function/module | src/scaffold/os_utils.py | is_windows, is_macos, is_linux, normalize_path, recommended_development_notice |
| Remote ops guard | function/module | src/scaffold/remote_guard.py | detect_tokens(env), skip_remote_steps_if_missing, warn_or_execute_remote_placeholders |
| Logger/Status | helper | src/scaffold/logger.py | structured logging, exit code mapping, JSON status output for verification step |

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | CLI & orchestration dry-run and normal run mapping | Empty target dir; env simulating Linux; no tokens required | Run CLI with --dry-run then without --dry-run | dry-run only simulates writes; normal run calls FS write helpers | dry-run: no write_file called; normal: write_file called for each expected path; exit code 0 |
| TC-002 | positive | Scaffold happy path creates expected structure & files | Empty dir; template context provided | Run orchestrator.create(target_dir, force=False) | Files and directories are created | exists() true for README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/, Dockerfile.template; template renderer called with correct context |
| TC-003 | positive / integration-ready unit | Template rendering populates placeholders for README and CONTRIBUTING | Sample template files with placeholders | Invoke templates.render(context) | Rendered content contains replaced placeholders | Rendered README contains project name, example commands, "Run unit tests" section and platform-specific instructions for Linux/macOS |
| TC-004 | negative/edge | Idempotency & conflict detection without force | Pre-populate target files with different content | Run orchestrator.create(target_dir, force=False) | No overwrite; backup created; diff generated; user prompt simulated as 'no' | backup_file called with original content preserved; generateDiff returned non-empty; write_file not called for existing files |
| TC-005 | edge | Windows platform handling | Simulate platform.system() == 'Windows' | Run CLI/orchestrator | Scripts do not execute unsupported native commands; show guidance | os_utils.is_windows true; logger.warn called with WSL2/Docker recommendation; exit code 0 with guidance status |
| TC-006 | error | Missing CI tokens -> skip remote-only steps | Env without CI tokens; orchestrator includes remote placeholder tasks | Run orchestrator | Remote actions skipped; warnings logged; rest of scaffold proceeds | remote_guard.detect_tokens returns False; remote_guard.skip_remote_steps invoked; no network calls attempted; exit code 0 |
| TC-007 | error | Filesystem write or template render failure | Simulate write_file throwing IOError or templates.render raising TemplateError | Run orchestrator | Orchestrator handles exception gracefully and exits non-zero | logger.error called with context; exit code != 0; no partial/half-written state left (atomic write behavior verified via write_file_atomic called) |

Notes:
- Each test must be independent; use fresh tmp_path or pyfakefs per test.
- One assertion per test behavior rule is preferred; additional assertions validate side effects (calls to mocks).

## AI Component Test Cases [CONDITIONAL: If AIR-XXX in scope]
This plan does not include AI/LLM components. No AI tests required.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/cli/scaffold_cli_test.py | Unit tests for CLI arg parsing and status output |
| CREATE | tests/unit/scaffold/test_orchestrator.py | Tests for orchestration flows (happy, conflict, error) |
| CREATE | tests/unit/scaffold/test_templates.py | Template-rendering unit tests with fixtures |
| CREATE | tests/unit/scaffold/test_fs_helpers.py | Tests for FS helper behavior (atomic write, backup) |
| CREATE | tests/mocks/fs.mock.py | Mock/stub implementations or pyfakefs fixture setup |
| CREATE | tests/fixtures/templates/*.md.tpl | Minimal template fixtures for README/CONTRIBUTING/Dockerfile.template |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value |
|------------|-----------|---------------|--------------|
| Filesystem helpers (write/read/exists) | fs stub / pyfakefs | Simulate file creation, atomic writes, errors | write_file: success or raise IOError; exists: True/False; backup_file: record calls |
| Template renderer | real with fixtures / mock | Use small test templates; optionally mock to force render error | Normal: return interpolated string; Error: raise TemplateError |
| OS/platform detection | monkeypatch | Control platform.system() / os.name values | 'Linux' / 'Darwin' / 'Windows' |
| Environment variables (tokens) | monkeypatch env | Simulate presence/absence of CI_TOKEN, GITHUB_TOKEN | Present: "token-abc"; Absent: None |
| Prompt/confirmation | stub | Simulate user input for overwrite confirmation | 'yes' or 'no' returned synchronously |
| Logger | spy | Capture warnings/errors for assertions | log entries captured; no real console output |
| External network/remote operations | stub/mock | Always not executed in unit tests; raise if called to ensure skipped | N/A (should not be called) |

Mocking notes:
- Prefer pyfakefs or tmp_path (pytest) to avoid touching real FS.
- Monkeypatch platform and os.environ to control environment.
- Use dependency injection where possible: orchestrator accepts FS object, template renderer, and prompt handler.

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Valid scaffold | context {project_name: "proj", license: "MIT"} | README with "proj", Dockerfile.template present, directories created |
| Existing files conflict | target README contains "old" | Backup created containing "old"; new file not written without force |
| Template error | malformed template string | TemplateError raised -> orchestrator logs error and exits non-zero |
| Windows developer | platform 'Windows' | Guidance message recommending WSL2/Docker; no native steps executed |
| Missing tokens | env empty | Remote-only steps skipped; scaffold still creates local artifacts |

## Test Commands
- Run all tests: pytest -q
- Run scaffold tests only: pytest tests/unit/scaffold -q
- Run CLI tests only: pytest tests/unit/cli -q
- Run with coverage: pytest --cov=src --cov-report=term-missing
- Run single test function: pytest tests/unit/scaffold/test_orchestrator.py::test_happy_path -q
(If project is Node.js implementation: use npm test or npx jest; detect implementation before running.)

## Coverage Target
- **Line Coverage**: 90%
- **Branch Coverage**: 80%
- **Critical Paths**: 100% coverage required for:
  - Conflict detection and backup logic (conflict.py)
  - Template rendering inputs/outputs (templates.py)
  - Filesystem write atomicity and failure handling (fs.py)
  - OS detection logic that changes execution flow (os_utils.py)

Rationale: Critical safety logic must be exhaustively tested to prevent destructive overwrites and to ensure predictable developer experience.

## Documentation References
- pytest docs: https://docs.pytest.org/
- pyfakefs: https://pyfakefs.readthedocs.io/ (if using Python)
- Python mocking: https://docs.python.org/3/library/unittest.mock.html
- Template engine docs: Jinja2 docs if Jinja used: https://jinja.palletsprojects.com/
- Project test patterns: refer to existing tests/ directory in repo (if present)

## Implementation Checklist
- [ ] Create test file structure per Expected Changes (tests/unit/cli, tests/unit/scaffold, fixtures)
- [ ] Configure pytest and pyfakefs (or tmp_path fixtures) in test environment
- [ ] Implement mocks/stubs for FS, platform detection, env variables, and prompt handling
- [ ] Implement positive tests for CLI dry-run and happy-path scaffold creation (TC-001, TC-002, TC-003)
- [ ] Implement negative and edge tests for idempotency/conflict, Windows handling, and missing tokens (TC-004, TC-005, TC-006)
- [ ] Implement error scenario tests for FS and template failures and assert graceful exit and logging (TC-007)
- [ ] Run test suite and validate coverage targets; add tests for any uncovered critical branches

Additional layer-specific plans required: This document covers only the Scripts layer. Separate unit test plans are required for Infra (CI, pre-commit, badges, workflows) and Docs (README content validations and CONTRIBUTING content). See related test plans referenced above.

Previous Analysis and Reasoning:
- Chosen Python/pytest as default test stack because scaffolding scripts are commonly implemented in Python and pytest supports strong filesystem fixtures (pyfakefs/tmp_path) and concise mocking. If repository shows package.json or an existing Node implementation, switch to Jest/memfs — tests and mocks will have equivalent structure and assertions. The goal is simple, deterministic unit tests focused on behavior (not implementation) and isolation to avoid side effects.