File: .propel/context/tasks/EP-TECH/us_001/unittest/test_plan_scripts_us_001_scripts.md

# Unit Test Plan - [TASK_us_001_scripts]

## Requirement Reference
- **User Story**: [us_001] Project Repository Scaffolding & README
- **Story Location**: [.propel/context/tasks/us_001/us_001.md]
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

## Test Plan Overview
Purpose: Unit-test the scripts layer that implements repository scaffolding. Focus areas:
- Template rendering correctness for README, CONTRIBUTING, Dockerfile.template and workflow placeholders.
- Filesystem writer behavior: atomic writes, idempotency, backup on conflict, respect for --dry-run and --force.
- OS detection behavior and guidance for Windows.
- Remote-step guard: detection of missing CI tokens and skipping remote-only steps with warnings.
- CLI parsing orchestration and return codes.

Scope boundaries:
- Included: in-process logic of renderer, file_writer, os_detector, remote_guard, CLI parsing, and scaffolder orchestration.
- Excluded: real network calls, real Git operations, actual CI execution; those are for integration/infra tests.

Test isolation principle:
- All tests must be independent: use temporary filesystem sandboxes or in-memory FS, isolate environment variables per test, and inject test doubles for logging and external operations.

## Dependent Tasks
- Implement scaffolder public functions (generate_repo / scaffold.apply / file_writer API)
- Provide template fixtures used by tests
- Ensure renderer and file_writer expose testable public functions (no hidden side effects)
- Ensure CLI entrypoint supports dependency injection for logger, FS, and prompts for easier unit testing

## Components Under Test

| Component | Type | File Path | Responsibilities |
|-----------|------|-----------|------------------|
| scaffolder.generate_repo / scaffold.apply | function / CLI entry | scripts/scaffold.py OR scripts/scaffold.js | Orchestrates scaffold generation, parses CLI flags (--dry-run, --force), calls renderer & file_writer |
| template_renderer.render / render_readme / render_contributing | function | scripts/renderer.py OR scripts/renderer.ts | Renders README, CONTRIBUTING, Dockerfile.template and workflow YAMLs from templates and variables |
| file_writer.write_with_idempotency / safe_write | function | scripts/file_writer.py OR scripts/file_writer.ts | Writes files atomically, creates backups if conflict, respects --dry-run and --force |
| os_detector.detect / is_windows | function | scripts/os_detector.py OR scripts/os_detector.ts | Detects platform and returns guidance/warnings for unsupported OS (Windows) |
| remote_step_guard.check_tokens / should_skip_remote_ops | function | scripts/remote_guard.py OR scripts/remote_guard.ts | Detects CI tokens in env and decides whether to skip remote-only steps (returns warnings) |
| cli.parser | module | scripts/cli.py OR scripts/cli.ts | Parses flags and exposes config to scaffolder; supports injection for testing |
| logger | module | scripts/logger.py OR scripts/logger.ts | Emits structured logs/messages captured by tests (or injected spy) |

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | Create full scaffold in empty target dir (AC-1) | Empty temp dir, valid template fixtures present, env tokens present or not required | Call generate_repo() with default flags | Files and directories created successfully | All expected artifacts exist: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/, Dockerfile.template; function returns success status; file modes as expected |
| TC-002 | positive/content | README renderer includes required sections and example commands (AC-2) | Template contains placeholders; project config fixture contains script mappings | Call template_renderer.render('README.md', config) | Render returns text | README contains headings/sections: Prerequisites, Local Setup, Run Unit Tests, Run Locally (example commands), Integration Tests (or mocks), Linters/Formatters, Contribution, Operational runbooks reference |
| TC-003 | positive/commands_consistency | Example commands in README map to available script targets (AC-2, AC-3) | Project config fixture exposes Makefile/package.json script targets | Render README | Inspect example commands | Each example command string in README maps to an existing target in the config fixture (e.g., "make test" exists or "npm test") |
| TC-004 | positive | CONTRIBUTING.md references PR template and contains checklist (AC-4) | PR template fixture exists in templates | Render CONTRIBUTING | Inspect content | CONTRIBUTING contains checklist items and a reference/link to .github/PULL_REQUEST_TEMPLATE.md |
| TC-005 | positive | Governance artifacts created and README verification step present (AC-5) | Templates for .pre-commit-config and workflows present | Call generate_repo() | Files created | .pre-commit-config.yaml present with expected hook names; .github/workflows/ contains at least one YAML file with top-level keys (name, on, jobs) and README contains verification step reference |
| TC-006 | positive/edge | Idempotent behavior: existing files backed up and not overwritten without --force (AC-1 edge) | Temp dir with existing README.md and other files (different content) | Call generate_repo() without --force | Backups created and originals preserved | file_writer creates backup files (e.g., README.md.bak or README.md.orig); generate_repo returns diff summary; original file content unchanged |
| EC-001 | edge_case | Windows detection prints guidance and skips unsupported native steps | Simulate Windows environment (os_detector stub returns 'Windows') | Call generate_repo() | Warnings emitted; no failure | Captured logs include Windows guidance recommending WSL2/Docker; generate_repo returns success with warning metadata (no unhandled exception) |
| ES-001 | error | Missing CI tokens: remote-only steps skipped and warnings logged (Edge case) | Env missing CI_TOKEN and templates include remote-only steps | Call generate_repo() | Remote steps skipped gracefully | remote_step_guard returns skip decisions; scaffolder logs warning; no network calls attempted; success return with warnings |
| ES-002 | error | Missing template assets cause controlled error (failure scenario) | Remove a required template file from template fixtures | Call template_renderer.render for that artifact | Controlled exception raised | TemplateNotFoundError (or equivalent) thrown with actionable message; no partial writes performed; exception includes missing-path detail |

Notes:
- For TC-001 / TC-006 variants, repeat assertions for both --dry-run (no writes) and --force (overwrite) flows in dedicated unit tests.
- Each test must be isolated: use tmp_path/tmpdir or pyfakefs/mock-fs and fresh env per test. Use dependency injection or monkeypatching for logger and remote steps.

## AI Component Test Cases [CONDITIONAL: Not applicable]
AI components are not in scope for the Scripts layer in this user story. No AI/LLM tests required.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/scripts/test_scaffold_generate.py (or .ts/.js) | Unit tests for scaffolder core behaviors (TC-001, TC-006, ES-001) |
| CREATE | tests/unit/scripts/test_template_renderer.py (or .ts/.js) | Renderer tests validating README/CONTRIBUTING content (TC-002, TC-004, ES-002) |
| CREATE | tests/unit/scripts/test_commands_consistency.py (or .ts/.js) | Tests that README example commands map to available script targets (TC-003) |
| CREATE | tests/unit/scripts/test_file_writer.py (or .ts/.js) | Tests for file_writer idempotency, backups, dry-run and force behaviors (TC-006) |
| CREATE | tests/unit/scripts/test_os_and_remote_guard.py (or .ts/.js) | Tests for Windows guidance and missing CI token behavior (EC-001, ES-001) |
| CREATE | tests/fixtures/scripts_templates/* | Template fixtures for README, CONTRIBUTING, pre-commit, workflows, PR template |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value |
|------------|-----------|---------------|--------------|
| Filesystem | in-memory FS / tmpdir | Provide empty or pre-populated directories; simulate permission errors | Realistic FS state for assertions |
| Template store | fixture files | Provide valid templates and selectively remove to test ES-002 | Template text or missing-file error |
| Environment variables | stub/monkeypatch | Toggle CI token presence (CI_TOKEN, GITHUB_TOKEN) | present / absent |
| Remote operations (network/git) | spy / stub | Ensure no network calls during unit tests; assert they are not invoked when tokens absent | No-op or mocked response if needed |
| Logger | spy | Capture warnings/info messages for assertion | Structured log entries |
| OS detection | stub | Return 'Windows' or 'Linux' to exercise branching | 'Windows' / 'Linux' / 'Darwin' |
| CLI prompts | fake confirmation function | Return yes/no for overwrite confirmation tests | True/False |

Mocking guidance by language:
- Python: pytest + monkeypatch; use pyfakefs for FS heavy tests; unittest.mock for stubs/spies.
- Node: Jest/Vitest + mock-fs or memfs; jest.spyOn for logger and subprocess mocks.

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Empty repo | {} (empty tmp dir) | All scaffold artifacts created (TC-001) |
| Pre-populated repo | README.md with different content | Backups created; original preserved (TC-006) |
| README template | README template fixture with placeholders and script examples | Rendered README contains expected sections and commands (TC-002, TC-003) |
| Missing template | Remove CONTRIBUTING template | TemplateNotFoundError thrown; no writes (ES-002) |
| Missing tokens | Env without CI_TOKEN | Remote steps skipped with warnings (ES-001) |
| Windows env | os_detector stub returns Windows | Guidance log present; unsupported native steps skipped (EC-001) |

Fixture location suggestion:
- .propel/context/tasks/us_001/unittest/fixtures/scripts_templates/

## Test Commands
- Python (if repository implements scripts in Python):
  - Run all script tests: pytest tests/unit/scripts -q
  - Run with coverage: pytest --cov=scripts --cov-report=term-missing tests/unit/scripts
  - Run single test: pytest tests/unit/scripts/test_template_renderer.py::test_readme_contains_sections -q
- Node (if repository implements scripts in JS/TS):
  - Run all script tests: npm test -- tests/unit/scripts or npx jest tests/unit/scripts
  - Run with coverage: npm test -- --coverage --testPathPattern=tests/unit/scripts
  - Run single test: npx jest -t "renders README sections"
- Note: Test runner chosen should match repository language; CI should detect package.json or pyproject.toml to pick commands automatically.

## Coverage Targets
- Overall Line Coverage (Scripts layer): 80%
- Branch Coverage (Scripts layer): 70%
- Critical utilities (file_writer, template_renderer, remote_guard, os_detector): 95% line coverage or as close as practicable
- Rationale: high coverage on idempotency, token-checking, and file-write branching reduces risk of destructive behavior in repo scaffolding.

## Documentation References
- Testing frameworks:
  - Python/pytest: https://docs.pytest.org/
  - pytest-cov/pyfakefs docs (if used) referenced in project test README
  - Jest: https://jestjs.io/ (or Vitest equivalent)
- Mocking & FS:
  - pyfakefs: https://pyfakefs.readthedocs.io/
  - mock-fs / memfs for Node: https://github.com/tschaub/mock-fs / https://github.com/streamich/memfs
- Project Test Patterns: Refer to existing tests under tests/unit/ for examples of DI and fixture usage (link in repo)
- Template authoring: internal templates README describing variables and placeholders

## Implementation Checklist
- [ ] Create test files under tests/unit/scripts per Expected Changes and add the fixture directory (.propel/context/tasks/us_001/unittest/fixtures/scripts_templates/)
- [ ] Implement template renderer tests validating presence of required README/CONTRIBUTING sections and command consistency (TC-002, TC-003, TC-004)
- [ ] Implement scaffolder orchestration tests for happy path, dry-run, and force behaviors (TC-001, variants)
- [ ] Implement file_writer tests for idempotency, backup creation, and error handling (TC-006, ES-002)
- [ ] Implement os_detector and remote_guard tests (EC-001, ES-001) using environment and platform stubs
- [ ] Use in-memory FS (pyfakefs or mock-fs/memfs) or tmpdir fixtures to ensure test isolation
- [ ] Capture and assert logs/warnings via injected logger spy to validate guidance messages and skip decisions
- [ ] Add coverage collection configuration and ensure coverage targets are enforced in CI for the Scripts layer