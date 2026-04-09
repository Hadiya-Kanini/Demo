# Navigation Map

## Screen Cross-References

- [wireframe-SCR-001-login.html](Hi-Fi/wireframe-SCR-001-login.html)
- [wireframe-SCR-002-forgot-password.html](Hi-Fi/wireframe-SCR-002-forgot-password.html)
- [wireframe-SCR-003-reset-password.html](Hi-Fi/wireframe-SCR-003-reset-password.html)
- [wireframe-SCR-004-reset-confirmation.html](Hi-Fi/wireframe-SCR-004-reset-confirmation.html)
- [wireframe-SCR-005-account-locked.html](Hi-Fi/wireframe-SCR-005-account-locked.html)

## Navigation Flows

### Login -> Role Redirect -> Dashboard (Customer)
SCR-001 → SCR-006 → SCR-007

### Login -> Role Redirect -> Dashboard (Admin)
SCR-001 → SCR-006 → SCR-008

### Forgot Password -> Reset -> Confirmation
SCR-002 → SCR-003 → SCR-004

### Login -> Rate Limit -> CAPTCHA
SCR-001 → SCR-011 → SCR-016

### Admin Unlock Flow
SCR-008 → SCR-012 → SCR-015

### Session Expiry -> Reauthenticate
SCR-010 → SCR-001 → SCR-006
