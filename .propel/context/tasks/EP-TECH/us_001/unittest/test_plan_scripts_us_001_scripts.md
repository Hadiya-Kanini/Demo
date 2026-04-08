File: test_plan_scripts_scaffold_generation.md

# Unit Test Plan - [TASK_us_001_scripts]

## Requirement Reference
- **User Story**: [us_001] Project Repository Scaffolding & README
- **Story Location**: [.propel/context/tasks/us_001/us_001.md]
- **Layer**: Scripts
- **Related Test Plans**: Infra and Docs layer test plans required (not included here)
- **Acceptance Criteria Covered**:
  - AC-1: Scaffolding produces repository root artifacts and directory structure → TC-001
  - AC-2: README contains required sections and example commands → TC-002
  - AC-3: Documented local setup commands present (Linux/macOS) → TC-003
  - AC-4: CONTRIBUTING checklist and PR template referenced → TC-004
  - AC-5: Governance artifacts (CI badges placeholders, pre-commit config, .github/workflows/*) created → TC-005
  - Edge Case - Windows guidance → EC-001
  - Edge Case - Missing CI tokens behavior (skip remote ops + warnings) → ES-001
  - Edge Case - Conflict/idempotency/backup behavior → TC-006

## Test Plan Overview
Purpose: Unit-test the scaffolding scripts' core logic: template rendering, filesystem writes with idempotency/backup behaviors, OS detection, CLI flags (--dry-run, --force), remote-step guard (token detection and skip behavior), and basic content validations for generated files (README, CONTRIBUTING, Dockerfile.template, .github workflow placeholders). Scope is limited to in-process logic and filesystem interactions simulated via mocks; no external network or real CI interactions will run.

## Dependent Tasks
- Implement scaffolder entrypoints and public functions (generate_repo / scaffold.apply / file_writer API)
- Add template fixtures used by tests
- Infra and Docs layer validators to be tested in their own plans

## Components Under Test

| Component | Type | File Path | Responsibilities |
|-----------|------|-----------|------------------|
| scaffolder.generate_repo | function/CLI entry | scripts/scaffold.py (or scripts/scaffold.js) | Orchestrates scaffold generation, parses CLI flags (--dry-run, --force), calls renderer & file_writer |
| template_renderer.render | function | scripts/renderer.py (or scripts/renderer.ts) | Renders README, CONTRIBUTING, Dockerfile.template and workflow YAMLs from templates and variables |
| file_writer.write_with_idempotency | function | scripts/file_writer.py | Writes files atomically, creates backups if conflict, respects --dry-run and --force |
| os_detector.detect | function | scripts/os_detector.py | Detects platform and returns guidance/warnings for unsupported OS (Windows) |
| remote_step_guard.check_tokens | function | scripts/remote_guard.py | Detects CI tokens in env and decides whether to skip remote-only steps (returns warnings) |
| cli.parser | module | scripts/cli.py | Parses flags and exposes config to scaffolder

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | Create full scaffold in empty target dir | Empty temp dir, valid templates present, env tokens set or not required | Call generate_repo() with default flags | Files and directories created | Expected files exist: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/, Dockerfile.template; correct exit/success return value |
| TC-002 | positive/content | README renderer includes required sections and example commands | Template contains placeholders; project config includes Makefile/npm scripts mapping | Render README via template_renderer | Produced README text inspected | README contains headings: Prerequisites, Local Setup, Run Unit Tests, Run Locally (example commands), Linters/Formatters, Contribution section, Operational runbooks reference |
| TC-003 | positive/commands_consistency | Example commands in README match provided script targets | Project config object exposes commands (Makefile/package.json fixture) | Render README | Validate examples against config | Each example command maps to an actual script/Make target in the provided fixture |
| TC-004 | positive | CONTRIBUTING.md references PR template and contains checklist | Valid PR template in templates folder | Render CONTRIBUTING | Validate content | CONTRIBUTING contains checklist items, reference/link to .github/PULL_REQUEST_TEMPLATE.md |
| TC-005 | positive | Governance artifacts created (pre-commit + workflow placeholders) | Templates for .pre-commit-config and workflows present | generate_repo() | Files created | .pre-commit-config.yaml present with expected hook names; .github/workflows/ contains at least one YAML file with top-level keys (name, on, jobs) |
| TC-006 | positive/edge | Idempotent behavior: existing files backed up and not overwritten without --force | Temp dir with existing README.md and other files (different content) | generate_repo() without --force | Backups created; original preserved | file_writer creates backup (e.g., README.md.bak or README.md.orig); returns diff summary; generated-new file not overwrite original unless --force |
| EC-001 | edge_case | Windows detection prints guidance and skips unsupported native steps | Simulate Windows (os_detector returns 'Windows') | generate_repo() | Warnings emitted; no failing exception | stdout/stderr contains Windows guidance recommending WSL2/Docker; returns success with warning code (non-zero only for real fatal errors) |
| ES-001 | error | Missing CI tokens: remote-only steps skipped and warnings logged (do not fail hard) | Env missing CI_TOKEN and templates include remote-only steps | generate_repo() | Remote steps skipped | scaffolder logs warning, skips remote operations, returns success; no network calls attempted |
| ES-002 | error | Missing template assets cause controlled error | Remove a required template file from templates set | Call template_renderer.render for that artifact | Controlled exception raised | Exception type (TemplateNotFoundError) thrown with actionable message; no partial writes performed |

Notes:
- Each test must be isolated: use temporary filesystem sandbox or in-memory FS (pyfakefs / mock-fs) and fresh env vars per test.
- Tests should exercise both paths where --dry-run prevents writes and where --force overwrites.

## AI Component Test Cases [CONDITIONAL: Not applicable]
- AI components are out of scope for the Scripts layer in this story. Skip AI tests.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/scripts/test_scaffold_generate.py (or .ts/.js) | Unit tests for scaffolder core behaviors |
| CREATE | tests/unit/scripts/fixtures/templates/* | Minimal template fixtures used by renderer tests |
| CREATE | tests/mocks/fs_mock.py | Filesystem mocking helper (pyfakefs wrappers) |
| CREATE | tests/mocks/env_mock.py | Helpers to set/clear environment variables for token scenarios |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value |
|------------|-----------|---------------|--------------|
| Filesystem | in-memory FS (pyfakefs / mock-fs) | Simulate empty and pre-populated directories; assert atomic writes | Virtual files created, backed up, or preserved |
| OS detection | stub / monkeypatch | Return 'Linux', 'Darwin', or 'Windows' as needed | Deterministic behavior per test |
| Environment / Secrets | env patch | Provide CI_TOKEN or not; control remote_guard decisions | Presence/absence toggles skip behavior |
| Template rendering engine | stub/spies | For content-focused tests, use real renderer with template fixtures; for error tests, stub to throw TemplateNotFoundError | Rendered string or thrown error |
| Logger | spy | Capture warnings and errors for assertions | Logged messages captured |
| Subprocess / network calls | mock | Ensure no external network calls occur; simulate any remote step side effects | No-op or controlled responses |

Mocking guidance by implementation language:
- Python: pytest + pyfakefs, monkeypatch, unittest.mock for spies
- Node (JS/TS): Jest + mock-fs, jest.spyOn
- Shell scripts: use temporary directories and wrap external calls; for unit-level script functions use shUnit2 or small JS/Python wrappers to test logic

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Empty repo | {} (empty dir) | All required files and directories created |
| Existing conflicting files | README.md with content "old" | Backup created; original preserved unless --force |
| Windows | os_detector => Windows | Warning message recommending WSL2/Docker; no native unsupported steps |
| Missing templates | Template set missing README.tpl | TemplateNotFoundError; no partial writes |
| Missing CI token | ENV: CI_TOKEN unset | Remote steps skipped; warning logged |

## Test Commands
- Python (recommended default for scripts impl. in Python):
  - Run tests: pytest tests/unit/scripts -q
  - Run with coverage: pytest --cov=scripts --cov-report=term-missing tests/unit/scripts
  - Run single test: pytest tests/unit/scripts/test_scaffold_generate.py::test_create_full_scaffold -q
- Node (if scaffolder is JS/TS):
  - Run tests: npm test -- tests/unit/scripts
  - Run with coverage (jest): npm run test:coverage -- tests/unit/scripts
  - Single test: npx jest -t "create full scaffold"
- Shell (if scaffolder is shell):
  - Run bats tests: bats tests/unit/scripts
  - Single test via bats filter as supported

Include these commands in CI job that runs the scripts-layer unit tests in PRs.

## Coverage Target
- Line Coverage: 80%
- Branch Coverage: 70%
- Critical Paths (target 100%):
  - file_writer idempotency and backup branches
  - conflict detection and --force vs no-force branches
  - remote_step_guard token-present vs absent branches
  - os_detector branches (Linux, macOS, Windows)
  - template_renderer happy/error flows

## Documentation References
- pytest documentation (if Python) — pytest.org
- pyfakefs documentation (Python filesystem mocking)
- Jest documentation (if Node) — jestjs.io
- mock-fs documentation (Node)
- bats-core documentation (shell tests)
- Template engine docs (jinja2 / handlebars) depending on implementation
- Project coding & security guidelines (referenced in repo docs)

## Implementation Checklist
- [ ] Create unit test files under tests/unit/scripts per Expected Changes
- [ ] Set up filesystem and env mocks (pyfakefs / mock-fs / monkeypatch) for isolation
- [ ] Implement positive test cases (TC-001..TC-006) covering artifact creation and content checks
- [ ] Implement edge & error tests (EC-001, ES-001, ES-002) for Windows guidance, missing tokens, missing templates
- [ ] Ensure each test is independent (fresh temp FS and env) and uses Arrange-Act-Assert
- [ ] Configure test command(s) and coverage collection in local test runner
- [ ] Add CI job (or update existing) to run scripts-layer unit tests and enforce coverage
- [ ] Additional layer-specific plans required for Infra and Docs validators (Infra, BE/Docs)