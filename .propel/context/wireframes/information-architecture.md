# Information Architecture

## Screen Inventory

| ID | Name | Description | Priority |
|------|------|------|------|
| SCR-001 | Login | Primary entry point for users to authenticate with email/password. Includes show/hide password, submit with loading state, inline validation, non-revealing error copy, and optional CAPTCHA area when rate-limited. | P0 |
| SCR-002 | Forgot Password (Request) | Request a password reset. Non-revealing success states and inline validation. Submits request and shows generic confirmation flow. | P0 |
| SCR-003 | Reset Password (Form) | Password reset form reached via secure token. Includes new password, confirm password, password strength helper (SCR-014), inline validation, and submit with loading/disable behavior. | P0 |
| SCR-004 | Password Reset Confirmation | Generic confirmation screen after password reset request completes. Non-revealing copy and CTA to return to login. | P0 |
| SCR-005 | Account Locked / Lockout Guidance | Lockout guidance screen showing TTL (if available) and recovery options (forgot password, contact support, admin unlock). Non-revealing copy and admin contact CTA. | P0 |

## User Flows

### FL-001: Login -> Role Redirect -> Dashboard (Customer)
Sequence: SCR-001 → SCR-006 → SCR-007

### FL-002: Login -> Role Redirect -> Dashboard (Admin)
Sequence: SCR-001 → SCR-006 → SCR-008

### FL-003: Forgot Password -> Reset -> Confirmation
Sequence: SCR-002 → SCR-003 → SCR-004

### FL-004: Login -> Rate Limit -> CAPTCHA
Sequence: SCR-001 → SCR-011 → SCR-016

### FL-005: Admin Unlock Flow
Sequence: SCR-008 → SCR-012 → SCR-015

### FL-006: Session Expiry -> Reauthenticate
Sequence: SCR-010 → SCR-001 → SCR-006
