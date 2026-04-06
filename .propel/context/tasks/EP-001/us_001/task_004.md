# Task - task_004

## Task ID: task_004
## Task Title: Implement JWT issuance and secret/config integration
## Category: Backend

## Requirement Reference
- User Story: us_001
- Story Location: .propel/context/tasks/us_001/us_001.md
- Acceptance Criteria:
  - Given valid credentials, POST /auth/login returns HTTP 200 with JSON containing:
    - access_token (JWT), token_type "Bearer", expires_in (seconds)
  - Token payload contains standard claims: sub, iat, exp, roles
  - Token expiry (exp - iat) equals configured TTL
  - Signing key is loaded from a secrets manager (no hard-coded keys)
  - Audit log entry recorded for token issuance (user id, timestamp, client IP, user-agent)
- Edge Cases:
  - Missing signing key → server returns 500 with non-sensitive message
  - Secrets manager unavailable → server returns 503 (with retry/backoff in infra)
  - Config TTL missing/invalid → fallback to safe default and log warning

## Description
Implement a JwtService and integrate it into the authentication flow so that:
- Signed JWTs are issued with standard claims (sub, iat, exp, roles).
- Token TTL (expiry) is sourced from application configuration.
- Private signing key (for RS256) is retrieved from the configured secrets manager (e.g., AWS Secrets Manager) — key material must not be stored in source or config files.
- Implement a small SecretsManagerAdapter to fetch (and cache) signing key material with rotation support.
- Ensure unit tests validate claims, expiry calculation, and signature verification (using test public key).
- Wire issuance into the existing POST /auth/login flow so response includes access_token, token_type, expires_in; record audit log entry.

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
| Backend | Node.js | 20.x |
| Web Framework | Express | 4.x |
| JWT Library | jose (recommended) | latest compatible |
| Secrets Manager SDK | AWS SDK v3 (Secrets Manager client) | v3 |
| Config | dotenv / config module | N/A |
| Test Runner | Jest | 29.x |
| Linter/Formatter | ESLint / Prettier | project default |

(Adjust to project stack if different — use project's versions as source of truth.)

## Task Overview
Implement backend components that:
- Load TTL from configuration (e.g., AUTH_TOKEN_TTL_SECONDS).
- Load the RS256 private key from a secrets manager at runtime.
- Build JWT payload with sub, iat, exp, roles