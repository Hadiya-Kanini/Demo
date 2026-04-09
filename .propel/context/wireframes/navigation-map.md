# Navigation Map

## Screen Cross-References

- [wireframe-SCR-001-login.html](Hi-Fi/wireframe-SCR-001-login.html)
- [wireframe-SCR-002-forgot-password-request.html](Hi-Fi/wireframe-SCR-002-forgot-password-request.html)
- [wireframe-SCR-003-forgot-password-confirmation.html](Hi-Fi/wireframe-SCR-003-forgot-password-confirmation.html)
- [wireframe-SCR-004-reset-password.html](Hi-Fi/wireframe-SCR-004-reset-password.html)
- [wireframe-SCR-005-reset-success.html](Hi-Fi/wireframe-SCR-005-reset-success.html)

## Navigation Flows

### Login -> Customer Dashboard
SCR-001 → SCR-010

### Login -> Admin Dashboard
SCR-001 → SCR-011

### Login -> Employee Dashboard
SCR-001 → SCR-012

### Forgot Password / Reset Flow
SCR-002 → SCR-003 → SCR-004 → SCR-005

### Account Lockout & Recovery
SCR-001 → SCR-006 → SCR-002 → SCR-009

### CAPTCHA Step-up during Login
SCR-001 → SCR-007 → SCR-001 → SCR-010|SCR-011|SCR-012

### Session Expiry and Re-authentication
SCR-010 → SCR-013 → SCR-001

### Rate Limit / Too Many Requests
SCR-001 → SCR-014

### Error Handling (Server/Unauthorized)
SCR-001 → SCR-015 → SCR-016

### Admin Unlock Flow
SCR-011 → SCR-009 → SCR-006
