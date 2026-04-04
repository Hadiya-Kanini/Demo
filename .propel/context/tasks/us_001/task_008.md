## Task ID: task_008
## Task Title: Define and implement token storage & revocation adapter
## Category: Backend

## Description
Define a pluggable token provider interface and implement a production-ready adapter that supports:
- Integration with an external revocation service (US_011) when available.
- A safe, explicit fallback mode when the external service is not available: stateless token issuance (no server-side revocation), emit warning metrics/logging, and surface a capability flag so callers can act accordingly.
Provide concrete implementations (HTTP client adapter for US_011, DB-backed store for local environments, and a Noop/Stateless fallback). Add secure API endpoints to perform revocation operations and unit/integration tests covering adapter selection and behavior.

Parent Story: us_001

## Acceptance Criteria
All criteria must be verifiable via automated tests or manual verification steps:
1. A typed TokenStore interface exists with methods:
   - storeToken(jti: string, expiresAt: Date, opts?: { userId?: string, meta?: Record<string,any>}): Promise<void>
   - isRevoked(jti: string): Promise<boolean>
   - revokeToken(jti: string, opts?: { reason?: string }): Promise<void>
2. Adapter selection:
   - When REVOCATION_SERVICE_URL (or equivalent config) is present and reachable, the system instantiates the ExternalRevocationAdapter that calls US_011 APIs for store/isRevoked/revoke.
   - When REVOCATION_SERVICE_URL is absent/unreachable, the system uses StatelessFallbackAdapter and logs/emits a one-time warning metric indicating "revocation_service_unavailable".
3. Behavior:
   - ExternalRevocationAdapter persists revocations and returns isRevoked=true for revoked jtis until their configured expiry.
   - DB-backed adapter (Postgres/Redis) persists revocations with TTL and supports fast isRevoked checks.
   - StatelessFallbackAdapter does not persist revocations; isRevoked returns false; revokeToken is a no-op but records audit log + metric.
4. Endpoints:
   - POST /api/auth/revoke — accepts { jti } (authenticated) and triggers adapter.revokeToken.
   - GET /internal/auth/tokens/:jti/revoked — internal endpoint that returns revocation state (for health/debug).
   - Endpoint behavior covered by integration tests and protected by service-to-service auth (e.g., service token).
5. Tests:
   - Unit tests cover all TokenStore implementations and adapter selection logic (>= 90% of new code paths exercised).
   - Integration tests verify persistence behavior for DB-backed adapter: storeToken -> revokeToken -> isRevoked returns true.
   - A test asserts the fallback mode logs/emits a "revocation_service_unavailable" metric when external service cannot be contacted.
6. Migration/Schema:
   - If a DB-backed store is used, a database migration exists to create token_revocations table (or Redis key convention documented) with TTL behavior.
7. Documentation:
   - README or inline docs describe configuration keys, behavior differences (stateless vs stateful), and recommended operational notes (e.g., when using stateless tokens, rotate TTL and communicate inability to revoke).
8. Observability:
   - Metrics/logs emitted for storeToken, revokeToken, isRevoked calls and for the fallback warning. Traces include correlation id / request id.

## Technical Specifications

APIs/Endpoints
- POST /api/auth/revoke
  - Auth: required (service-level token or user session depending on caller context