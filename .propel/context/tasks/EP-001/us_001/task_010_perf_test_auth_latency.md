# Task - [TASK_010]

## Requirement Reference
- User Story: [us_001] (extracted from input)
- Story Location: [.propel/context/tasks/EP-001/us_001/us_001.md]
- Acceptance Criteria:  
    - Given a registered user with valid credentials, When the client sends a POST /auth/login with a JSON payload:
      {
        "username": "user@example.com",
        "password": "correct-password"
      }
      Then the server responds with HTTP 200 and a JSON body:
      {
        "access_token": "<JWT>",
        "token_type": "Bearer",
        "expires_in": 3600
      }
      - The token is signed with the system's configured signing key, contains standard claims (sub, iat, exp, roles), and expires after the configured TTL.
      - The response body contains no stack traces or internal error details.
    - Given a user supplies incorrect credentials, When the client POSTs to /auth/login, Then the server responds with HTTP 401 and a generic error message:
      {
        "error": "Invalid credentials"
      }
      - Failed login attempt is recorded (user identifier or IP) for rate-limiting and monitoring.
      - Response time and payload size remain within acceptable limits to avoid timing/side-channel leaks.
    - Given malformed input (missing fields, invalid JSON, invalid email format), When the client POSTs to /auth/login, Then the server responds with HTTP 400 with a validation error that identifies invalid fields (e.g., "password: required", "username: invalid email format") and does not proceed to password verification.
    - Given repeated failed attempts that exceed configured thresholds, When threshold is reached, Then subsequent requests return HTTP 429 (or 423 if account locked) with a generic message ("Too many attempts, try again later"), and the system enforces backoff/lockout per security policy. Alerts and audit logs must be created for suspected brute-force activity.
    - Given a successful login, When the server issues a token, Then an audit record is created (user id, timestamp, client IP, user-agent), and integration tests demonstrate median authentication latency meets NFR (auth median < 1s) under standard load.
- Edge Case:
    - Database unavailable on credential verification — return HTTP 503 with "Service temporarily unavailable" and a retryable error code; do not leak internal details.
    - Multiple failed attempts exceed threshold — temporarily lock or throttle the account/IP per security policy and return HTTP 429 or 423 with generic message; log event for monitoring and alerting.
    - Password hash algorithm/version mismatch (legacy records) — handle by verifying using the legacy algorithm, force re-hash on next successful login, and record upgrade event without revealing algorithm differences.

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

### **CRITICAL: Wireframe Implementation Requirement (UI Tasks Only)**
IF Wireframe Status = AVAILABLE or EXTERNAL:
- N/A (Task has no UI impact)

## Applicable Technology Stack
| Layer | Technology | Version |
|-------|------------|---------|
| Frontend | N/A | N/A |
| Backend | N/A | N/A |
| Database | N/A | N/A |
| Library | k6 (primary) | 0.43.1 |
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
Create and execute an automated integration/performance test using k6 (primary) to measure authentication (POST /auth/login) median latency under a defined "standard load" profile and verify it meets the NFR: median < 1s. Capture and persist environment metadata and test artifacts (raw k6 output, summary metrics, CSV/JSON), fail CI if median >= 1s, and produce a short results report containing pass/fail, p50/p90/p99, error rate, throughput, tested commit/tag, and environment details.

## Dependent Tasks
- .propel/context/tasks/EP-001/us_001/task_001_be_login_route_controller.md (POST /auth/login implemented and deployed to test environment)
- .propel/context/tasks/EP-001/us_001/task_003_be_auth_service.md (Auth service & token issuance available)
- Infrastructure: test environment (staging/test cluster) provisioned with app image and network access
- Secrets: test service signing key or test-key configured in test environment
- Test data: test user(s) with known credentials created (or seed script available)
- Observability endpoints / metrics exporter available (optional but recommended)

## Impacted Components
- API: POST /auth/login endpoint (interaction only)
- AuthService: credential verification, token issuance (read-only for test)
- CI pipeline: new perf test job (GitHub Actions / GitLab CI)
- Test artifacts dir: artifacts/perf/auth_latency/
- Test infra: k6 scripts and runner configuration
- Monitoring/metrics: optional scraping of app metrics (auth.latency) if available

## Implementation Plan
- Select primary tool: k6 (JS) for deterministic scripting and easy CI integration. Provide a locust alternative README if requested.
- Define test scenarios:
  - Standard load (constant VUs e.g., 50 users for 5 minutes) — align with team-defined "standard load" (confirm with infra/PO); include ramp-up stage.
  - Single-request latency measurement: POST /auth/login with valid test credentials; ensure token handling but do not reuse token for subsequent auth calls in the scenario to avoid caching effects.
- Implement k6 script:
  - Use k6 thresholds: http_req_duration p(50) < 1000 (ms), and set fail condition.
  - Collect response errors, HTTP status counts, and custom tags (git_commit, env).
  - Capture and export metrics to JSON and CSV via k6 reporters.
- Add runner scripts:
  - ./tests/perf/auth_latency/run_k6_locally.sh (wraps docker k6 run or local k6)
  - ./tests/perf/auth_latency/ci_k6_run.sh (used by CI, writes artifacts to workspace)
- CI integration:
  - Add a lightweight GitHub Actions workflow: .github/workflows/perf-auth.yml that runs the k6 script against the staging/test endpoint when a release candidate tag is created or on-demand (manual workflow_dispatch).
  - Fail the job if k6 exits with threshold failure.
- Artifacts & reporting:
  - Persist k6 output JSON and a generated summary (p50/p90/p99, errors, throughput) to artifacts/perf/auth_latency/{timestamp}/
  - Produce a one-page report file artifacts/perf/auth_latency/{timestamp}/report.md
- Test execution & capture:
  - Run locally once for validation, then run in CI. Capture environment metadata: git commit SHA, image tag, test endpoint URL, runtime (k6 version), and runner machine type.
- Validation:
  - Verify median (p50) < 1000ms. If not, capture traces, sample responses, and raise incident/issue.

## Current Project State
- Project root (placeholder — update as dependent tasks complete):
  .
  ├─ app/
  │  ├─ src/
  │  └─ Dockerfile
  ├─ server/
  │  ├─ src/
  │  └─ Dockerfile
  ├─ tests/
  │  └─ perf/ (empty — to be created)
  ├─ .github/
  │  └─ workflows/
  └─ .propel/
      └─ context/
          └─ tasks/EP-001/us_001/

## Expected Changes
| Action | File Path | Description |
|--------|-----------|-------------|
| CREATE | tests/perf/auth_latency/k6_auth_login_test.js | k6 JS script performing POST /auth/login under defined scenarios, with thresholds (p50 < 1000ms), tags, and JSON/CSV output configuration. |
| CREATE | tests/perf/auth_latency/run_k6_locally.sh | Helper script to run k6 locally or via docker and write artifacts to ./artifacts/perf/auth_latency/{{ts}}/. |
| CREATE | tests/perf/auth_latency/ci_k6_run.sh | CI wrapper that runs k6, collects outputs, sets exit code based on thresholds, and archives artifacts for CI. |
| CREATE | artifacts/perf/auth_latency/.gitkeep | Placeholder to allow artifact directory to be tracked (CI will populate actual results). |
| CREATE | .github/workflows/perf-auth.yml | Optional GitHub Actions workflow to invoke ci_k6_run.sh against configured test environment (manual/dispatch). |
| CREATE | tests/perf/auth_latency/README.md | Documentation: how to run locally, env variables required, interpretation of results, and locust alternative note. |
| MODIFY | .propel/context/tasks/EP-001/us_001/us_001.md | Append reference to this performance test task (traceability). |
| MODIFY | .gitignore | Ensure artifacts/perf/auth_latency/* is allowed for CI artifacts but not committed (if not already configured). |

## External References
- k6 documentation: https://k6.io/docs/
- k6 thresholds and checks: https://k6.io/docs/using-k6/checks-and-thresholds
- Running k6 in Docker: https://hub.docker.com/r/loadimpact/k6
- Example k6 script for JSON POST: https://k6.io/docs/examples/http-requests/#post-json
- GitHub Actions: https://docs.github.com/en/actions
- Locust (alternative): https://docs.locust.io/en/stable/

## Build Commands
- Local docker execution:
  - docker run --rm -i -v $(pwd)/tests/perf/auth_latency:/scripts -v $(pwd)/artifacts/perf/auth_latency:/output loadimpact/k6 run /scripts/k6_auth_login_test.js --out json=/output/result.json
- Local k6 execution (if k6 installed):
  - K6_OPTIONS="--vus 50 --duration 5m" k6 run tests/perf/auth_latency/k6_auth_login_test.js
- CI will call: bash tests/perf/auth_latency/ci_k6_run.sh

## Implementation Validation Strategy
- [ ] k6 script syntactically valid and runs locally
- [ ] CI workflow executes and archives artifacts
- [ ] k6 results JSON/CSV are produced in artifacts/perf/auth_latency/{timestamp}/
- [ ] median (p50) http_req_duration reported < 1000 ms (pass criteria)
- [ ] Error rate (HTTP 5xx/4xx unexpected) below acceptable threshold (configurable)
- [ ] Report file generated with git commit, endpoint, k6 version, p50/p90/p99, throughput, and pass/fail
- [ ] If applicable: correlate k6 results with app metrics (auth.latency) if metrics endpoint available

## Implementation Checklist
- [ ] Implement k6 test script tests/perf/auth_latency/k6_auth_login_test.js with thresholds: http_req_duration p(50) < 1000ms and checks for HTTP 200.
- [ ] Add run scripts (run_k6_locally.sh, ci_k6_run.sh) that produce artifacts and environment metadata.
- [ ] Add CI workflow .github/workflows/perf-auth.yml (manual dispatch) to execute ci_k6_run.sh against test environment and archive results.
- [ ] Run the test locally against staging endpoint, verify artifacts, and capture p50/p90/p99; adjust load profile if needed.
- [ ] Validate that failing threshold causes non-zero exit code in CI