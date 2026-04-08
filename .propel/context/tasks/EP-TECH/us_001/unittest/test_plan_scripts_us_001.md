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
Purpose: Unit-test the scaffolding scripts' core logic: template rendering, filesystem writes with idempotency/backup behaviors, OS detection, CLI flags (--dry-run, --force), remote-step guard (token detection and skip behavior), and basic content validations for generated files (README, CONTRIBUTING, Dockerfile.template, .github workflow placeholders). Scope is limited to in-process logic and filesystem interactions simulated via mocks; no real network calls, no actual Git operations, and no CI execution.

Scope boundaries:
- Included: renderer, file writer/idempotency, OS detection, remote guard, CLI parsing orchestration.
- Excluded: end-to-end execution of generated CI pipelines, real network/template downloads, manual confirmation UIs — those are covered by higher-level integration tests or Infra plan.

## Dependent Tasks
- Implement scaffolder public functions (generate_repo / scaffold.apply / file_writer API)
- Provide template fixtures used by tests
- Ensure renderer and file_writer expose testable public functions (no hidden side effects)

## Components Under Test

| Component | Type | File Path | Responsibilities |
|-----------|------|-----------|------------------|
| scaffolder.generate_repo | function / CLI entry | scripts/scaffold.py OR scripts/scaffold.js | Orchestrates scaffold generation, parses CLI flags (--dry-run, --force), calls renderer & file_writer |
| template_renderer.render | function | scripts/renderer.py OR scripts/renderer.ts | Renders README, CONTRIBUTING, Dockerfile.template and workflow YAMLs from templates and variables |
| file_writer.write_with_idempotency | function | scripts/file_writer.py OR scripts/file_writer.ts | Writes files atomically, creates backups if conflict, respects --dry-run and --force |
| os_detector.detect | function | scripts/os_detector.py OR scripts/os_detector.ts | Detects platform and returns guidance/warnings for unsupported OS (Windows) |
| remote_step_guard.check_tokens | function | scripts/remote_guard.py OR scripts/remote_guard.ts | Detects CI tokens in env and decides whether to skip remote-only steps (returns warnings) |
| cli.parser | module | scripts/cli.py OR scripts/cli.ts | Parses flags and exposes config to scaffolder |

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | Create full scaffold in empty target dir (AC-1) | Empty temp dir, valid template fixtures present, env tokens present or not required | Call generate_repo() with default flags | Files and directories created successfully | All expected artifacts exist: README.md, CONTRIBUTING.md, LICENSE, CODE_OF_CONDUCT.md, .gitignore, src/, tests/, docs/, .github/workflows/, scripts/, charts/ or deploy/, Dockerfile.template; function returns success status |
| TC-002 | positive/content | README renderer includes required sections and example commands (AC-2) | Template contains placeholders; project config fixture contains script mappings | Call template_renderer.render('README.md', config) | Render returns text | README contains headings: Prerequisites, Local Setup, Run Unit Tests, Run Locally (example commands), Integration Tests (or mocks), Linters/Formatters, Contribution section, Operational runbooks reference |
| TC-003 | positive/commands_consistency | Example commands in README map to available script targets (AC-2, AC-3) | Project config fixture exposes Makefile/package.json script targets | Render README | Inspect example commands | Each example command in README string maps to an existing target in config fixture (e.g., "make test" exists in Makefile fixture or "npm test" in package.json fixture) |
| TC-004 | positive | CONTRIBUTING.md references PR template and contains checklist (AC-4) | PR template fixture exists in templates | Render CONTRIBUTING | Inspect content | CONTRIBUTING contains checklist items and a reference/link to .github/PULL_REQUEST_TEMPLATE.md |
| TC-005 | positive | Governance artifacts created: pre-commit and workflow placeholders (AC-5) | Templates for .pre-commit-config and workflows present | Call generate_repo() | Files created | .pre-commit-config.yaml present with expected hook names; .github/workflows/ contains at least one YAML file with top-level keys (name, on, jobs) and README contains a verification step reference |
| TC-006 | positive/edge | Idempotent behavior: existing files backed up and not overwritten without --force (AC-1 edge) | Temp dir with existing README.md and other files (different content) | Call generate_repo() without --force | Backups created and originals preserved | file_writer creates backup files (e.g., README.md.bak or README.md.orig); generate_repo returns diff summary; original file content unchanged |
| EC-001 | edge_case | Windows detection prints guidance and skips unsupported native steps (Edge case) | Simulate Windows environment (os_detector stub returns 'Windows') | Call generate_repo() | Warnings emitted; no failure | Captured logs/stdout include Windows guidance recommending WSL2/Docker; generate_repo returns success with warning metadata (not an unhandled exception) |
| ES-001 | error | Missing CI tokens: remote-only steps skipped and warnings logged (Edge case) | Env missing CI_TOKEN and templates include remote-only steps | Call generate_repo() | Remote steps skipped gracefully | remote_step_guard returns skip decisions; scaffolder logs warning; no network calls attempted; success return with warnings |
| ES-002 | error | Missing template assets cause controlled error (failure scenario) | Remove a required template file from template fixtures | Call template_renderer.render for that artifact | Controlled exception raised | TemplateNotFoundError (or equivalent) thrown with actionable message; no partial writes performed; exception contains missing-path detail |

Notes:
- Each test must be isolated: use temporary filesystem sandbox or in-memory FS (pyfakefs / mock-fs) and fresh env vars per test.
- Tests should exercise both --dry-run (no writes) and --force (overwrite) behaviors where applicable (these are covered within TC-001 / TC-006 variants).

## AI Component Test Cases [CONDITIONAL: Not applicable]
AI components are not in scope for the Scripts layer in this user story. No AI/LLM tests required.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/scripts/test_scaffold_generate.py (or .ts/.js) | Unit tests for scaffolder core behaviors (TC-001, TC-006, ES-001) |
| CREATE | tests/unit/scripts/test_template_renderer.py (or .ts/.js) | Renderer tests validating README/CONTRIBUTING content (TC-002, TC-004, ES-002) |
| CREATE | tests/unit/scripts/test_commands_consistency.py (or .ts/.js) | Tests that README example commands map to project scripts (TC-003) |
| CREATE | tests/unit/scripts/test_os_detection.py (or .ts/.js) | Tests for EC-001 Windows guidance and other platform branches |
| CREATE | tests/unit/scripts/fixtures/templates/* | Minimal template fixtures used by renderer tests |
| CREATE | tests/mocks/fs_mock.py (or .ts/.js) | Filesystem mocking helpers (pyfakefs / mock-fs wrappers) |
| CREATE | tests/mocks/env_mock.py | Helpers to set/clear environment variables for token scenarios |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value |
|------------|-----------|---------------|--------------|
| Filesystem | in-memory FS (pyfakefs / mock-fs) or tmpdir fixtures | Simulate empty and pre-populated directories; assert atomic writes and backups | Virtual files created/backed up or preserved |
| OS detection | stub / monkeypatch | Return 'linux' / 'darwin' / 'windows' per test | Deterministic platform string |
| Environment / Secrets | env patch | Provide CI_TOKEN present / absent to toggle remote guard | Presence/absence toggles skip behavior |
| Template rendering engine | use real renderer with fixture templates or stub to throw | For content tests: use real renderer + fixture templates; for error tests: stub to throw TemplateNotFoundError | Rendered string or exception |
| Logger | spy | Capture warnings and errors for assertions | Captured log messages |
| Subprocess / network calls | mock / stub | Prevent any real network/git calls; simulate successful or failing remote fetches if needed | No-op or controlled responses |

Mocking notes:
- Prefer in-process mocks over patching subprocess calls.
- Do not mock internal logic under test (test contracts, not internals).
- Ensure no test performs actual network/Git operations.

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Valid scaffold | Minimal template fixtures + config fixture (project name, scripts map) | Full set of files rendered and created (TC-001) |
| README content validation | README template fixture + config with commands | README contains required headings & commands (TC-002) |
| Commands mapping | package.json or Makefile fixture with scripts | README example commands match existing targets (TC-003) |
| Pre-populated repo | Temp dir with existing README.md (different content) | Backup created; original preserved unless --force (TC-006) |
| Platform boundary | os_detector stub = 'Windows' | Warnings emitted, guidance present (EC-001) |
| Missing tokens | os.environ without CI_TOKEN | Remote steps skipped, warnings logged (ES-001) |
| Missing template | Remove a required template file | TemplateNotFoundError thrown with actionable message (ES-002) |

Fixture locations:
- .propel/context/tasks/us_001/unittest/test_data/templates/
- .propel/context/tasks/us_001/unittest/test_data/configs/
- .propel/context/tasks/us_001/unittest/test_data/existing_repo_samples/

## Test Commands
(Recommend commands for both common implementation languages; pick the command set matching actual implementation.)

- Python / pytest:
  - Run all script unit tests: pytest tests/unit/scripts -q
  - Run single test file: pytest tests/unit/scripts/test_scaffold_generate.py::TestClass::test_case -q
  - Run with coverage: pytest --cov=scripts --cov-report=term-missing tests/unit/scripts

- Node (TypeScript/JavaScript) / Jest:
  - Run all script unit tests: npm test -- tests/unit/scripts --silent
  - Run single test file: npm test -- -t "test_scaffold_generate"
  - Run with coverage: npm run test:coverage -- --collectCoverageFrom='scripts/**' --testPathPattern=tests/unit/scripts

Notes:
- Use CI config to pin environment (python v3.9+ / node 16+) and test runner versions.
- Ensure pyfakefs/mock-fs and HTTP/Git stubs are installed for test environment.

## Coverage Target
- Line Coverage (scripts modules): 80% overall
- Branch Coverage: 70% overall
- Critical functions (file_writer.write_with_idempotency, template_renderer.render, os_detector.detect, remote_step_guard.check_tokens): 90–100% line and branch coverage
- Rationale: focus critical logic with high coverage; overall scripts package at 80% is achievable and meaningful.

## Documentation References
- Template engine docs (Jinja2/Handlebars/EJS) — use project's chosen template engine docs
- pytest docs: https://docs.pytest.org/ (if Python)
- Jest docs: https://jestjs.io/ (if Node)
- pyfakefs docs / mock-fs docs for filesystem mocking
- Project test patterns: reference existing tests under tests/unit/ if present

## Implementation Checklist
- [ ] Create unit test files per Expected Changes and add template fixtures
- [ ] Implement filesystem sandboxing via pyfakefs or mock-fs and env patch helpers
- [ ] Implement positive tests for full scaffold generation and content presence (TC-001..TC-005)
- [ ] Implement idempotency and backup tests including --dry-run and --force variants (TC-006)
- [ ] Implement platform detection tests for Windows guidance (EC-001)
- [ ] Implement missing tokens behavior tests to ensure remote steps are skipped (ES-001)
- [ ] Implement template-missing error tests asserting controlled exceptions (ES-002)
- Additional layer-specific plans required: Infra and Docs layer test plans must be created/updated to cover CI workflow YAML validation, README badge placeholders verification, and docs-level cross-file traceability (see Related Test Plans)