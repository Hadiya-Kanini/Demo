test_plan_scripts_scaffold_and_validation.md

# Unit Test Plan - [US_us_001 - Scripts Layer]

## Requirement Reference
- **User Story**: us_001
- **Story Location**: .propel/context/tasks/us_001/us_001.md
- **Layer**: Scripts
- **Related Test Plans**: test_plan_infra_ci_and_repo_protection.md (Infra layer — required)
- **Acceptance Criteria Covered**:
  - AC-1: Scaffold generator or initial commit produces standard top-level layout (README.md, CONTRIBUTING.md, CODE_OF_CONDUCT.md, LICENSE, SECURITY.md, .gitignore, /src, /infra, /charts or /helm, /tests, /docs, scripts/ including bootstrap/validate scripts, .github/workflows).
  - AC-2: README 'Get Started' + devcontainer/local steps allow developer to boot dev environment and run documented test command (e.g., make test) that exits 0 on first-run when using Docker devcontainer; scripts/validate_scaffold.* supports local verification.
  - AC-3 (partial, scripts-related): Local validation scripts provide clear error messages and remediation links when preconditions fail (missing tools, unsupported OS).
  - AC-5: Pre-commit hook / secret detector in scripts flags secrets and returns clear remediation instructions linking to SECURITY.md.

## Test Plan Overview
This plan verifies the scripts layer: scaffold generation, dry-run behavior, conflict handling, local validation scripts (devcontainer + make test checks), and pre-commit/secret-detection behavior. Scope is unit-level testing of script logic and helpers (filesystem operations, templating, validation rules). Integration tests and CI/branch-protection checks are covered in the Infra test plan.

## Dependent Tasks
- Infra plan tests (CI workflows, branch protection, secret-scan in CI) must exist and run separately.
- Implementation of scaffold generator and validation scripts with public function boundaries (functions that accept inputs and return values rather than performing untestable shell side effects).
- Ensure scripts expose injectable points for filesystem and subprocess interactions (or refactor to allow injection).

## Components Under Test

| Component | Type | File Path (examples) | Responsibilities |
|-----------|------|----------------------|------------------|
| scaffold_generator | module / CLI wrapper | scripts/scaffold.py OR scripts/scaffold.js OR scripts/scaffold.sh (refactor into testable module) | Generate repository layout, render templates, support --dry-run, detect conflicts, produce migration plan/docs |
| validate_scaffold | module / script | scripts/validate_scaffold.py OR scripts/validate_scaffold.sh | Verify devcontainer config, Docker availability, presence of make targets, run quick smoke of test command (or simulate), produce user-facing remediation messages |
| secret_detector | function/library wrapper | scripts/hooks/secret_detector.py OR .pre-commit-config.yaml + scripts/precommit_hook.py | Detect common secret patterns, return structured findings (file, line, pattern), provide remediation link to SECURITY.md |
| fs_utils | helper module | scripts/lib/fs_utils.py OR scripts/lib/fs_utils.js | Abstract filesystem operations (create_file, list_dirs, exists) for easier mocking |
| subprocess_utils | helper module | scripts/lib/subprocess_utils.py | Abstract calls to external programs (docker, make, git) so they can be mocked in unit tests |

## Test Cases

| Test-ID | Type | Description | Given | When | Then | Assertions |
|---------|------|-------------|-------|------|------|------------|
| TC-001 | positive | Scaffold generator happy-path produces required top-level files and folders | Empty target directory mock | scaffold.generate(target, templates=default) | Files are created in target | All expected paths exist; README contains "Get Started"; templates rendered (non-empty); function returns success code/object |
| TC-002 | positive | Dry-run reports intended actions and does not write to disk | Target contains sample files; filesystem mock records writes | scaffold.generate(target, dry_run=True) | No filesystem writes performed; diff/plan returned | No create_file calls; return contains list of planned creates/overwrites with correct paths |
| TC-003 | negative | Conflict detection when repository has existing non-conforming files | Target contains legacy README.md and src/ files | scaffold.generate(target, default) | Generator fails with structured conflict report | Function returns non-zero/error object; migration.md content produced (in-memory) describing conflicts |
| TC-004 | edge_case | Merge guidance/merge-mode produces merge plan rather than blind overwrite | Target contains modified files and --merge flag passed | scaffold.generate(target, merge_mode=True) | Merge plan generated and returned (no destructive writes) | Returned plan lists file-by-file merge actions, conflicts marked, no filesystem writes when merge_mode only |
| TC-005 | positive | validate_scaffold detects devcontainer and docker available and reports success | devcontainer.json present; subprocess docker --version returns success | validate_scaffold.run(checks=['docker','devcontainer','make_test']) | Validation returns success | Return code 0; messages include "devcontainer detected" and "make test target available" (or simulated run passed) |
| TC-006 | negative | validate_scaffold handles missing docker / unsupported OS and prints remediation | docker check fails; platform marked unsupported | validate_scaffold.run() | Returns non-zero and remediation message instructing use of devcontainer | Return code != 0; message contains link to README and SECURITY.md if relevant; suggested command to use devcontainer |
| TC-007 | positive/negative | secret_detector flags secrets and blocks commit; passes clean files | File with API_KEY pattern and clean file | secret_detector.scan([file1,file2]) | Secrets found in file1; scan returns findings | findings list contains file, line, matched pattern; severity and remediation link to SECURITY.md present |
| TC-008 | edge_case / error | secret_detector ignores false positives when allow-list present and logs occurrence | File resembles credential but matches allow-list rule | secret_detector.scan(files, allowlist=...) | No findings returned | findings empty; logger recorded allow-list event |

Notes:
- Each test must be isolated: use filesystem and subprocess mocks; do not touch real disk or run docker/make.
- Map to Acceptance Criteria: TC-001/TC-002/TC-003/TC-004 → AC-1; TC-005/TC-006 → AC-2 & AC-3 (local validation); TC-007/TC-008 → AC-5.

## AI Component Test Cases [CONDITIONAL: If AIR-XXX in scope]
This story does not require AI/LLM components. Skip AI test cases.

## Expected Changes

| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/unit/test_scaffold_generator.py | Unit tests for scaffold_generator module (pytest) |
| CREATE | tests/unit/test_validate_scaffold.py | Unit tests for validation script logic |
| CREATE | tests/unit/test_secret_detector.py | Unit tests for secret detection logic |
| CREATE | tests/mocks/fs_mock.py | Shared fake filesystem setup (pyfakefs or mock-fs wrapper) |
| CREATE | tests/fixtures/sample_templates/* | Minimal template fixtures used by scaffold tests |

## Mocking Strategy

| Dependency | Mock Type | Mock Behavior | Return Value / Notes |
|------------|-----------|---------------|----------------------|
| Filesystem (OS) | in-memory FS (pyfakefs for Python / mock-fs for Node) | Simulate directories/files; assert create/write calls | Start each test with clean FS snapshot; independent per test |
| Subprocess (docker, make, git) | function stub / monkeypatch | Return configurable success/failure responses, exit codes, stdout/stderr | docker --version SUCCESS, docker absent -> raise CalledProcessError or return non-zero |
| Templating engine | stub | Render returns deterministic content given fixture variables | Return known string; validate placeholders replaced |
| Logger | spy | Capture messages for assertions | Verify remediation link logged |
| Git operations | stub | Simulate repo state and existing files | Return commit list or file contents as needed |
| Pre-commit hook runner | isolated function test | Simulate pre-commit invocation by calling secret_detector.scan | Return findings, and ensure hook would stop commit (simulate exit code) |

Mocking guidelines:
- Prefer injection of dependencies into functions (pass fs/subprocess clients) to avoid mocking internals.
- For shell-based scripts, refactor core logic into testable modules exposing pure functions; keep thin CLI wrappers minimal.
- Ensure no real network, Docker or git calls occur during unit tests.

## Test Data

| Scenario | Input Data | Expected Output |
|----------|------------|-----------------|
| Valid scaffold | Empty dir + default templates | List of created paths; README contains "Get Started" |
| Dry-run with conflicts | Target with legacy files | No writes; planned actions list including conflicts |
| Conflict case | Existing README with divergent content | migration plan content describing manual merge steps |
| Valid validation | devcontainer present, docker available, make target exists | validate_scaffold returns success (0) and messages confirming readiness |
| Missing docker | No docker available | validate_scaffold returns non-zero and remediation pointing to devcontainer usage |
| Secret found | File with "API_KEY = 'AKIA...'" | secret_detector returns finding with file, line, pattern and remediation link |
| False positive ignored | File matches allow-list pattern | secret_detector returns empty findings |

## Test Commands
- Run unit tests (Python / pytest example):
  - Run all tests: pytest -q
  - Run tests with coverage: pytest --cov=scripts --cov-report=term-missing
  - Run single test file: pytest tests/unit/test_scaffold_generator.py -q
  - Run single test by name: pytest -q -k "test_scaffold_happy_path"
- If implementation is Node/Jest:
  - Run all: npm test
  - Run single: npm test -- -t "scaffold generator"
  - Coverage: npm run test:coverage
- CI note: Unit tests must be runnable locally without Docker; any Docker-dependent behavior must be mocked.

## Coverage Target
- **Line Coverage**: 90%
- **Branch Coverage**: 80%
- **Critical Paths (must be 100%)**:
  - validate_scaffold: docker check, devcontainer detection, make test exit handling
  - secret_detector: detection of high-confidence secret patterns and allow-list handling
  - scaffold_generator: dry-run vs apply paths and conflict detection branches

## Documentation References
- pytest docs: https://docs.pytest.org/
- pyfakefs docs: https://pyfakefs.readthedocs.io/
- mock-fs (Node): https://www.npmjs.com/package/mock-fs
- Pre-commit: https://pre-commit.com/
- Project patterns: refer to existing tests in tests/ (if present)

## Implementation Checklist
- [ ] Create unit test files: tests/unit/test_scaffold_generator.*, tests/unit/test_validate_scaffold.*, tests/unit/test_secret_detector.*
- [ ] Add filesystem and subprocess mocks (pyfakefs/mock-fs and monkeypatch/stub helpers) and shared fixtures
- [ ] Ensure scaffold generator and validation scripts expose injectable FS/subprocess abstractions for unit testing (refactor if necessary)
- [ ] Implement positive, negative, edge, and error scenario tests listed in Test Cases (TC-001..TC-008)
- [ ] Implement secret detection allow-list tests and remediation message assertions
- [ ] Configure pytest (or Jest) coverage collection and verify coverage targets locally
- [ ] Run test suite and document any gaps; create Infra layer test plan to cover CI workflows and branch-protection tests

Additional layer-specific plans required: Infra tests for .github/workflows, branch protection, and CI-level secret scanning are out of scope for this scripts-layer plan and must be implemented in test_plan_infra_ci_and_repo_protection.md.