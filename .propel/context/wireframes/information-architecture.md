# Information Architecture

## Screen Inventory

| ID | Name | Description | Priority |
|------|------|------|------|
| SCR-001 | Login | Primary email/password login screen. Accepts email and password, client-side validation, inline errors, CTA to submit. Includes show/hide password toggle and link to Forgot Password. May surface CAPTCHA or rate-limit notices. | P0 |
| SCR-002 | Forgot Password - Request | User supplies email to request password reset. Always returns a non-revealing confirmation message regardless of account existence. | P0 |
| SCR-003 | Password Reset Email Sent | Confirmation screen shown after requesting password reset. Provides generic guidance and CTA to return to login. Notes that email will arrive if account exists. | P0 |
| SCR-004 | Reset Password (token) | Password reset form reached via secure single-use token or OTP. Validates token, enforces password policy and shows password strength helper. On success invalidates token and redirects to confirmation. | P0 |
| SCR-005 | Password Reset Success | Confirmation that password was updated. Provides CTA to login. Notes that failed_attempts are reset and suggests contacting support if issues persist. | P0 |

## User Flows

### FL-001: Login -> Customer Dashboard
Sequence: SCR-001 → SCR-010

### FL-002: Login -> Admin Dashboard
Sequence: SCR-001 → SCR-011

### FL-003: Login -> Employee Dashboard
Sequence: SCR-001 → SCR-012

### FL-004: Forgot Password / Reset Flow
Sequence: SCR-002 → SCR-003 → SCR-004 → SCR-005

### FL-005: Account Lockout & Recovery
Sequence: SCR-001 → SCR-006 → SCR-002 → SCR-009

### FL-006: CAPTCHA Step-up during Login
Sequence: SCR-001 → SCR-007 → SCR-001 → SCR-010|SCR-011|SCR-012

### FL-007: Session Expiry and Re-authentication
Sequence: SCR-010 → SCR-013 → SCR-001

### FL-008: Rate Limit / Too Many Requests
Sequence: SCR-001 → SCR-014

### FL-009: Error Handling (Server/Unauthorized)
Sequence: SCR-001 → SCR-015 → SCR-016

### FL-010: Admin Unlock Flow
Sequence: SCR-011 → SCR-009 → SCR-006
