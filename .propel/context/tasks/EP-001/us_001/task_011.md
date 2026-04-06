## Task ID: task_011
## Task Title: Create token revocation integration placeholder (BLOCKED by US_011)
## Category: Backend
## Description:
Define and scaffold the token-store / token-revocation integration contract and placeholder implementation so the codebase is prepared for final integration once US_011 (token-store service availability & spec) is delivered. Deliverables are interface definitions, a DI-friendly client adapter, a no-op / blocked runtime implementation that returns a well-defined "BLOCKED" error, and test placeholders (unit, integration, E2E) that assert the contract and clearly mark the feature as blocked by US_011. Do not implement production revocation logic; only scaffold the integration surface, error contract, telemetry hooks, and tests that will be updated when US_011 is completed.

This task is BLOCKED until US_011 provides the token-store API specification and credentials. Implement only scaffolding and mark runtime behavior as "BLOCKED - dependency missing".

## Acceptance Criteria:
- AC-1: An interface ITokenRevocationClient (or equivalent) is added and exported from src/services/revocation/interfaces (or project convention path) describing methods required for revocation and lookup (revokeToken, isTokenRevoked, bulkRevoke).
- AC-2: A concrete placeholder implementation TokenRevocationBlockedClient exists that implements the interface and, at runtime, throws or returns a deterministic BlockedIntegrationError containing:
    - code: "INTEGRATION_BLOCKED"
    - blocking_dependency: "US_011"
    - message: "Token-store integration blocked: await US_011"
  Unit tests assert the error shape.
- AC-3: A DI registration / factory is added so the placeholder can be swapped with a real implementation when US_011 is completed (e.g., via DI container, feature-flag, or environment variable). Documentation comment indicates switch point and required configuration keys.
- AC-4: API contract and method signatures are documented in code (TS interfaces or docstrings) with clear parameter/return shapes and error semantics (including timeouts, retries, TTL).
- AC-5: Unit test files exist that validate:
    - interface conformance (TokenRevocationBlockedClient implements all methods)
    - placeholder behavior returns/surfaces BlockedIntegrationError
  These tests must pass in CI.
- AC-6: Integration and E2E test files are present as placeholders that are skipped / marked pending and contain TODOs referencing US_011. Test runner should skip them by default (e.g., using describe.skip or @skip annotation) and assert that they are related to US_011.
- AC-7: README or inline developer note added at src/services/revocation/README.md describing required changes once US_011 is available: expected endpoints, auth, payload examples, metrics/events to produce, and migration steps.
- AC-8: No production behavior changes beyond adding scaffolding (no new runtime revocation behavior unless explicitly swapped in).

## Technical Specifications:

APIs/Endpoints
- No external HTTP endpoints are implemented as part of this placeholder task.
- Internal contract (to be used by auth/logout flow / session management):
  - revokeToken(token: string, meta?: RevokeMeta): Promise<RevokeResult>
  - isTokenRevoked(tokenId: string): Promise<boolean>
  - bulkRevoke(criteria: BulkRevokeCriteria): Promise<BulkRevokeResult>
- Error contract:
  - BlockedIntegrationError { code: "INTEGRATION_BLOCKED", blocking_dependency: "US_011", message: string, timestamp: ISOString, details?: any }

Components/Classes
- src/services/revocation/interfaces/ITokenRevocationClient.ts
  - Interface signatures and typedefs (RevokeMeta, RevokeResult, BulkRevokeCriteria, BulkRevokeResult)
- src/services/revocation/clients/TokenRevocationBlockedClient.ts
  - Placeholder implementation that implements ITokenRevocationClient and returns/throws BlockedIntegrationError for all operations
- src/services/revocation/errors/BlockedIntegrationError.ts
  - Typed error class matching error contract
- src/services/revocation/index.ts
  - Factory / DI registration wrapper (registerTokenRevocationClient, getTokenRevocationClient) with env var switch commented for real integration
- src/services/revocation/README.md
  - Developer notes & TODOs referencing US_011
- tests/unit/revocation/blocked-client.spec.ts
