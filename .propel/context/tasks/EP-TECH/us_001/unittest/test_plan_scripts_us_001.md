---
title: "Unit Test Plan - [TASK_us_001_scripts]"
layer: "Scripts"
story_id: "us_001"
date: "2026-04-08"
---

File: .propel/context/tasks/EP-TECH/us_001/unittest/test_plan_scripts_us_001.md

# Unit Test Plan - [TASK_us_001_scripts]

## Requirement Reference
- **User Story**: [us_001] Project Repository Scaffolding & README
- **Story Location**: [.propel/context/tasks/EP-TECH/us_001/us_001.md]
- **Layer**: Scripts
- **Related Test Plans**: Infra and Docs layer test plans required (separate plans)
- **Acceptance Criteria Covered**:
  - AC-1: Scaffolding produces repository root artifacts and directory structure → TC-001
  - AC-2: README contains required sections and example commands → TC-002, TC-003
  - AC-3: Documented local setup commands present (Linux/macOS) and workable flow (unit-test/run locally guidance) → TC-003
  - AC-4: CONTRIBUTING checklist and PR template referenced → TC-004
  - AC-5: Governance artifacts (CI badges placeholders, pre-commit config, .github/workflows/*) created and README verification step present → TC-005
  - Edge Case - Windows guidance → EC-001
  - Edge Case - Missing CI tokens behavior (skip remote ops + warnings) → ES-001
  - Edge Case - Conflict/idempotency/backup behavior → TC-006
  - Failure case - Missing template assets → ES-002
  - Security edge case - Path traversal / malicious path input handling → ES-003

## Test Plan Overview
Purpose: Unit-test the Scripts layer that implements repository scaffolding and README/CONTRIBUTING generation. Focus areas:
- Template rendering correctness for README, CONTRIBUTING, Dockerfile.template and workflow placeholders.
- Filesystem writer behavior: atomic writes, idempotency, backup on conflict, respect for --dry-run and --force.
- OS detection behavior and guidance for Windows.
- Remote-step guard: detection of missing CI tokens and skipping remote-only steps with warnings.
- CLI parsing orchestration and return codes.
- Defensive handling of user-provided paths (prevent path traversal).

Scope boundaries:
- Included: in-process logic of renderer, file_writer, os_detector, remote_guard, CLI parsing, and scaffolder orchestration (Python implementation).
- Excluded: real network calls, real Git operations, actual CI execution; those are for integration/infra tests.

Test isolation principle:
- All tests must be independent: use temporary filesystem sandboxes (pyfakefs) or pytest tmp_path, isolate environment variables per test, and inject test doubles for logging and external operations.

## Dependent Tasks
- Implement scaffolder public functions (generate_repo / scaffold.apply / file_writer API)
- Provide template fixtures used by tests
- Ensure renderer and file_writer expose testable public functions (no hidden side effects)
- Ensure CLI entrypoint supports dependency injection for logger, FS, and prompts for easier unit testing

## Components Under Test

| Component | Type | File Path | Responsibilities | Priority |
|-----------|------|-----------|------------------|----------|
| scaffolder.generate_repo / scaffold.apply | function / CLI entry | scripts/scaffold.py | Orchestrates scaffold generation, parses CLI flags (--dry-run, --force), calls renderer & file_writer | high |
| template_renderer.render / render_readme / render_contributing | function | scripts/renderer.py | Renders README, CONTRIBUTING, Dockerfile.template and workflow YAMLs from templates and variables | high |
| file_writer.write_with_idempotency / safe_write | function | scripts/file_writer.py | Writes files atomically, creates backups if conflict, respects --dry-run and --force; validates path inputs to prevent traversal | high |
| os_detector.detect / is_windows | function | scripts/os_detector.py | Detects platform and returns guidance/warnings for unsupported OS (Windows) | med |
| remote_step_guard.check_tokens / should_skip_remote_ops | function | scripts/remote_guard.py | Detects CI tokens in env and decides whether to skip remote-only steps (returns warnings) | med |
| cli.parser | module | scripts/cli.py | Parses flags and exposes config to scaffolder; supports injection for testing | med |
| logger | module | scripts/logger.py | Emits structured logs/messages captured by tests (or injected spy) | low |

Note: Technology stack committed for this plan: Python (pytest + pyfakefs + pytest-mock). All file paths and test examples below assume Python implementation.

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | Create full scaffold in empty target dir (AC-1) | Empty tmp_path, valid template fixtures present, env tokens present or not required | Call generate_repo() with default flags | Files and directories created successfully | All expected artifacts exist: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/, Dockerfile.template; function returns success status; file modes as expected |
| TC-002 | positive/content | README renderer includes required sections and example commands (AC-2) | Template contains placeholders; project config fixture contains script mappings | Call template_renderer.render('README.md', config) | Render returns text | README contains headings/sections: Prerequisites, Local Setup, Run Unit Tests, Run Locally (example commands), Integration Tests (or mocks), Linters/Formatters, Contribution, Operational runbooks reference |
| TC-003 | positive/commands_consistency | Example commands in README map to available script targets (AC-2, AC-3) | Project config fixture exposes Makefile/package.json script targets | Render README | Inspect example commands | Each example command string in README maps to an existing target in the config fixture (e.g., "make test" exists) |
| TC-004 | positive | CONTRIBUTING.md references PR template and contains checklist (AC-4) | PR template fixture exists in templates | Render CONTRIBUTING | Inspect content | CONTRIBUTING contains checklist items and a reference/link to .github/PULL_REQUEST_TEMPLATE.md |
| TC-005 | positive | Governance artifacts created and README verification step present (AC-5) | Templates for .pre-commit-config and workflows present | Call generate_repo() | Files created | .pre-commit-config.yaml present with expected hook names; .github/workflows/ contains at least one YAML file with top-level keys (name, on, jobs) and README contains verification step reference |
| TC-006 | positive/edge | Idempotent behavior: existing files backed up and not overwritten without --force (AC-1 edge) | tmp_path with existing README.md and other files (different content) | Call generate_repo() without --force | Backups created and originals preserved | file_writer creates backup files (e.g., README.md.bak or README.md.orig); generate_repo returns diff summary; original file content unchanged |
| EC-001 | edge_case | Windows detection prints guidance and skips unsupported native steps | Simulate Windows environment (os_detector stub returns 'Windows') | Call generate_repo() | Warnings emitted; no failure | Captured logs include Windows guidance recommending WSL2/Docker; generate_repo returns success with warning metadata (no unhandled exception) |
| ES-001 | error | Missing CI tokens: remote-only steps skipped and warnings logged | Env missing CI_TOKEN and templates include remote-only steps | Call generate_repo() | Remote steps skipped gracefully | remote_step_guard returns skip decisions; scaffolder logs warning; no network calls attempted; success return with warnings |
| ES-002 | error | Missing template assets cause controlled error (failure scenario) | Remove a required template file from template fixtures | Call template_renderer.render for that artifact | Controlled exception raised | TemplateNotFoundError (or equivalent) thrown with actionable message; no partial writes performed; exception includes missing-path detail |
| ES-003 | security | Path traversal / malicious path input is rejected (security edge) | Provide template target paths containing "../" or absolute path inputs from user config | Call file_writer.safe_write with malicious path | Operation fails fast and is not performed | file_writer raises InvalidPathError (or equivalent); no files are written outside target sandbox; logs include sanitized path info (no secrets) |

Notes:
- For TC-001 and TC-006 variants, include separate tests for --dry-run (no writes) and --force (overwrite) flows.
- Each test must be isolated: prefer pyfakefs or pytest tmp_path, monkeypatch env and inject fake prompt/confirm interfaces.
- Tests should assert one behavior per test (AAA).

## AI Component Test Cases [CONDITIONAL: Not applicable]
AI components are not in scope for the Scripts layer in this story.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/test_scaffold_generate.py | Unit tests for generate_repo / scaffolder orchestration |
| CREATE | tests/unit/test_renderer.py | Unit tests for template rendering (README, CONTRIBUTING) |
| CREATE | tests/unit/test_file_writer.py | Tests for atomic writes, idempotency, backups, and path validation |
| CREATE | tests/unit/test_os_detector_and_remote_guard.py | Tests for platform detection and missing CI token handling |
| CREATE | tests/fixtures/templates/* | Template fixtures used by renderer tests |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value |
|------------|-----------|---------------|--------------|
| Filesystem | pyfakefs or tmp_path | Simulate files/dirs; verify writes/backup behavior | In-memory FS state |
| Template loader | stub | Return fixture templates or raise TemplateNotFoundError | Template strings or exception |
| Logger | spy | Capture log messages for assertions | Structured log entries |
| Prompt/confirm | fake implementation | Return pre-configured responses (yes/no) | Boolean response |
| Network/GitHub | stub/mocked (responses or monkeypatch) | Ensure no real network calls; record attempted calls | No-op or recorded attempt |
| Environment | monkeypatch.env | Provide/omit CI tokens and other env vars per test | Dict of env vars |

Guidance:
- Use pytest fixtures to inject dependencies and keep tests deterministic.
- Do not mock internal implementation overly; mock only external interactions and IO.

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Valid template render | project config dict + README template fixture | README text with required sections |
| Existing conflicting file | tmp_path/README.md with "old" content | Backup created; original preserved unless --force |
| Missing templates | template store missing README template | TemplateNotFoundError raised |
| Windows env | monkeypatch platform to 'Windows' | Warning string recommending WSL2/Docker |
| Missing CI token | env without CI_TOKEN | remote steps skipped; warning log |

Fixtures:
- templates/readme.j2, templates/contributing.j2, templates/pre-commit.j2, templates/workflow_placeholder.j2
- project_config.json fixture including script targets mapping

## Test Commands
- Run all tests: pytest -q
- Run tests with coverage: pytest --cov=scripts --cov-report=term-missing
- Run single test file: pytest tests/unit/test_file_writer.py -q
- Run tests matching name: pytest -k "test_readme_contains_prerequisites" -q
- CI (coverage threshold): pytest --cov=scripts --cov-fail-under=80

Recommended: use tox or GitHub Actions job configured to run the above pytest commands in CI.

## Coverage Target
- Line Coverage: 90%
- Branch Coverage: 85%
- Critical Paths: scaffolder.generate_repo, file_writer.write_with_idempotency, template_renderer.render — aim for 95–100% branch coverage on these critical functions

## Documentation References
- pytest: https://docs.pytest.org/
- pyfakefs: https://pyfakefs.readthedocs.io/
- pytest-mock: https://pytest-mock.readthedocs.io/
- Jinja2 templating (if used): https://jinja.palletsprojects.com/
- PyYAML (workflow YAML validation): https://pyyaml.org/

## Implementation Checklist
- [ ] Create unit test files and fixtures listed in Expected Changes
- [ ] Configure pytest with pytest-cov and pyfakefs fixtures
- [ ] Implement positive, negative, edge, and security tests (TC-001..ES-003)
- [ ] Inject test doubles for logger, prompt, and environment variables for isolation
- [ ] Ensure file_writer path validation prevents traversal and tests assert it
- [ ] Add tests for --dry-run and --force flows
- [ ] Run tests and enforce coverage thresholds in CI (>=80% overall; critical paths >=95%)

Notes: Additional layer-specific test plans (Infra, Docs) are required for full story coverage — this plan focuses on the Scripts layer only.