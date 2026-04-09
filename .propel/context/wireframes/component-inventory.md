# Component Inventory

| Screen | Components |
|--------|------------|
| SCR-001: Login | brand-header, form.email-input (C/Inputs/TextField), form.password-input (C/Inputs/PasswordField), password-show-toggle (icon), password-helper (inline), primary-button (C/Actions/Button - loading state), link.forgot-password, alert.inline (C/Feedback/Alert), captcha-placeholder (C/Feedback/CAPTCHA - conditional), loading-skeleton/spinner, aria-live region for errors |
| SCR-002: Forgot Password - Request | brand-header, form.email-input (C/Inputs/TextField), primary-button (Send Reset), secondary-link.back-to-login, alert.inline (non-revealing copy), aria-live region for confirmation |
| SCR-003: Password Reset Email Sent | brand-header, confirmation-illustration (optional), text.non-revealing-confirmation, primary-button.back-to-login, support-link/contact-support |
| SCR-004: Reset Password (token) | brand-header, token-validation (implicit via URL/OTP), form.new-password-input (C/Inputs/PasswordField), form.confirm-password-input (C/Inputs/PasswordField), password-strength/helper (C/Content/PasswordHelper), primary-button.reset-password, alert.inline (token invalid/expired), aria-live region for errors |
| SCR-005: Password Reset Success | brand-header, confirmation-illustration, text.success-message, primary-button.back-to-login, support-link |