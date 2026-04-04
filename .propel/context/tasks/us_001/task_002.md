## Task ID: task_002
## Task Title: Implement password verification & legacy-hash migration
## Category: Backend
## Description:
Implement a server-side abstraction verifyPassword(userRecord, plaintext) in the authentication domain that:
- Verifies passwords primarily using Argon2 (current algorithm).
- Detects legacy bcrypt hashes and verifies them using bcrypt as a fallback.
- On successful verification of a legacy bcrypt hash, performs a secure re-hash using Argon2 and updates the user record in the database (atomic/transactional where possible) to upgrade the stored hash.
- Returns structured verification metadata indicating outcome, whether a legacy algorithm was used, and whether a re-hash/upgrade occurred.
The implementation must be safe (no timing leaks beyond library guarantees), robust under concurrent logins, and testable (unit, integration, E2E).

## Acceptance Criteria:
1. verifyPassword(userRecord, plaintext) returns an object with:
   - verified: boolean
   - algorithm: "argon2" | "bcrypt" | "unknown"
   - usedLegacy: boolean (true when bcrypt was used)
   - rehashPerformed: boolean (true if bcrypt verified and new Argon2 hash saved)
   - newHashStored: boolean (true if DB update succeeded)
   - message?: string for diagnostic reasons (non-sensitive)
2. If userRecord.password_hash is Argon2, verifyPassword uses Argon2 verify and does not re-hash.
3. If userRecord.password_hash is bcrypt:
   - verifyPassword verifies using bcrypt.
   - On success, it computes a new Argon2 hash and updates the user row with the new hash and algorithm metadata.
   - The DB update occurs only after successful verification and is done in a safe, idempotent way (no partial updates on failure).
4. If bcrypt verification fails, no DB changes occur and verifyPassword returns verified: false.
5. Operations are resilient to concurrent requests: if two logins concurrently verify a legacy hash, at most one successful write should be persisted or both must converge to same valid Argon2 hash without corrupting the user record—use optimistic concurrency (updated_at or row version) or retry logic.
6. Unit tests cover:
   - Argon2 success/failure
   - bcrypt legacy success/failure
   - metadata correctness
7. Integration tests cover DB update of hash on legacy verification and rollback behavior on partial failure.
8. E2E test (login flow) demonstrates that after a successful login for a legacy-hashed account, subsequent logins use Argon2 verification.
9. Implementation must use Argon2id with explicit, documented parameters and bcrypt verification for legacy hashes. Libraries and versions must be compatible with the existing backend stack (documented in Implementation).
10. No secrets or plaintext passwords are logged. Audit logging must record an event (user id, timestamp, reason "legacy-hash-upgrade" when rehash occurs) without including sensitive data.

## Technical Specifications:

- APIs/Endpoints
  - POST /auth/login (existing) — NOTE: This task does not introduce a new endpoint but modifies the authentication code path to call verifyPassword. verifyPassword must be a drop-in routine usable by the existing login flow.
  - Internal API: verifyPassword(userRecord, plaintext) : Promise<VerifyResult>

- Components/Classes (suggested module-names, adjust to codebase conventions)
  - services/PasswordVerifier (class or module)
    - verifyPassword(userRecord, plaintext): Promise<VerifyResult>
  - services/hashers/Argon2Hasher
    - hash(plaintext): Promise<string>
    - verify(hash, plaintext): Promise<boolean>
  - services/hashers/BcryptVerifier (legacy)
    - verify(hash, plaintext): Promise<boolean>
  - repositories/UserRepository (existing)
    - updatePasswordHash(userId, newHash, algorithm, expectedVersion?): Promise<boolean>
    - getById(userId): Promise<UserRecord>
  - services/AuditLogger (existing or new helper)
    - logAuthEvent({ userId, eventType, meta })
  - types/VerifyResult
    - { verified: boolean; algorithm: string; usedLegacy: boolean; rehashPerformed: boolean; newHashStored: boolean; message?: string }

- Data Models (DB-level)
  - users (existing table) — changes proposed
    - id (PK)
    - email
    - password_hash