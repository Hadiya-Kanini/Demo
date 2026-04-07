# Figma Design Specification

## 1. UI Impact Assessment

Summary
- UI impact: YES — authentication and account recovery flows are UI-critical.
- Primary UI features: Login (email/password), inline validation, account lockout notices, forgot-reset flows, optional MFA enrollment/verify, CAPTCHA escalation, admin unlock and audit-list UIs, role-based landing/redirect interstitial.
- Source references: Requirements Specification (Auth Service FR-001..FR-013, UC-001..UC-008).
- Stakeholders: Product Owner, Security Lead, Support Lead, Compliance, Frontend Engineering.

High-level Decision Constraints
- Must follow Figma 6-Page Standard (00_Cover → 06_Handoff).
- All frames use Auto Layout, tokens only (no hard-coded spacing/colors).
- All screens include five states (Default, Loading, Empty, Error, Validation).
- Accessible by design: WCAG 2.2 AA baseline, dark-mode parity required.

UI Impact Mapping (quick)
- Authentication flows (SCR-001..SCR-010) — P0: End User primary path.
- Admin flows (SCR-011, SCR-012) — P1/P2: Support/Admin workflows.
- Security elements (CAPTCHA, MFA) — P1: escalation & enrollment flows.

---

## 2. UX Requirements

UXR Requirements Table

| UXR-ID | Category | Requirement | Acceptance Criteria | Screens Affected |
|--------|----------|-------------|---------------------|------------------|
| UXR-001 | Performance | Auth flows must render next-screen redirect <= 2s (95th pct) | Measured 95th pct ≤ 2s; show loading skeleton >300ms | SCR-001, SCR-010 |
| UXR-101 | Usability | Primary auth actions reachable within 3 taps/clicks | Task test: 90% users within 3 steps | SCR-001, SCR-002 |
| UXR-102 | Usability | Primary CTA hierarchy: Sign in primary; auxiliary links secondary | Visual audit: primary CTA prominence & size | SCR-001 |
| UXR-103 | Usability | Password field: visibility toggle; new-password strength meter | Elements present in UI; strength meter shows levels & suggestions | SCR-001, SCR-003 |
| UXR-201 | Accessibility | All screens meet WCAG 2.2 AA | Axe/WAVE audit pass; keyboard & screen reader test pass | All screens |
| UXR-202 | Accessibility | Error/success messages announced via aria-live | Screen reader reads messages when they appear | All screens with dynamic messages |
| UXR-203 | Accessibility | Keyboard-only operation and visible focus states | Tab order logical; focus visible >=3:1 contrast | All interactive screens |
| UXR-301 | Responsiveness | Support Mobile (390x844), Tablet (768x1024), Web (1440x1024) | Layouts adapt per breakpoints; no overlap/clipping | All screens |
| UXR-401 | Visual Design | Use design tokens for all styling; no hard-coded values | Audit: tokens only in Figma styles | All components |
| UXR-501 | Interaction | Immediate client-side validation, server fallback, button-disabled-on-submit | Validations fire on blur & submit; CTA disables while submitting | SCR-001..SCR-003 |
| UXR-502 | Interaction | Loading indicators within CTAs and full-page skeletons for >300ms | Spinner in button & skeleton for main card | SCR-001, SCR-002, SCR-012 |
| UXR-601 | Error Handling | Generic non-revealing error messages for auth failures | "Invalid credentials" used; no account enumeration | SCR-001, SCR-002 |
| UXR-602 | Error Handling | Clear account-locked UX with next steps | Locked state shows duration/admin support CTA | SCR-005, SCR-011 |
| UXR-701 | Security UX | Session-expired interstitial with re-auth prompt | Interstitial presented; ARIA focus set to heading | SCR-009 |
| UXR-702 | Security UX | CAPTCHA and rate-limit escalation with contextual help | CAPTCHA appears after adaptive threshold; help link provided | SCR-008 |
| UXR-801 | Admin UX | Admin unlock flow supports search, unlock, and audit confirmation | Admin task completion within 3 steps; audit event logged | SCR-011 |
| UXR-901 | Localization | All user-facing strings must be localizable | All visible strings use tokenized strings in Figma | All screens |
| UXR-902 | Motion Pref | Honor prefers-reduced-motion | Animations disabled/scaled per user preference | All animated components |

UXR Derivation Logic
- Derived from Use Cases UC-001..UC-008 and Functional Requirements FR-001..FR-012.
- Each UXR maps to specific screens and acceptance checks (visual & functional).

---

## 3. Persona Analysis and User Journeys

Personas Summary

| Persona | Role | Primary Goals | Key Screens |
|---------|------|---------------|-------------|
| End User | Customer/Employee | Authenticate quickly, recover password, reach role dashboard | SCR-001, SCR-002, SCR-003, SCR-005, SCR-006, SCR-007 |
| Admin/Support | Support Operator | Unlock accounts, view audit entries, assist users | SCR-011, SCR-012 |
| Security/Compliance | Reviewer | Verify audit trails & non-revealing errors | SCR-012, SCR-011 |

Persona Journeys (high-level)

- End User — Happy Path
  1. Entry: Landing or /login → SCR-001 Default.
  2. Validate: Inline field validation; show errors as needed.
  3. Submit: CTA disabled, spinner in button (SCR-001 Loading).
  4. Success: Token issued; role redirect (SCR-010 Default) → dashboard.

- End User — Forgot Password
  1. From SCR-001 click "Forgot password" → SCR-002 Default.
  2. Submit email → show generic confirmation (SCR-002 Default / Empty).
  3. Receive email → follow token link → SCR-003 Default (Reset form).
  4. Submit new password → SCR-004 Confirmation / Redirect to Login.

- End User — Lockout
  1. Multiple failed attempts → SCR-005 Error (Account locked).
  2. Provide next steps: "Contact support" (SCR-011) or auto-unlock timer.

- End User — MFA Enrollment
  1. SCR-006 Default (QR + secret) with copy & download recovery codes.
  2. Verify TOTP in SCR-007; success returns to protected flow.

- Admin — Unlock Flow
  1. SCR-012: Search user → open SCR-011 (Account detail).
  2. Unlock → confirmation modal → audit logged.

---

## 4. Information Architecture & Screen Inventory

Site Map (high level)
[Auth App]
+-- Login (SCR-001)
+-- Forgot Password (SCR-002)
+-- Reset Password (SCR-003)
+-- Reset Confirmation (SCR-004)
+-- Account Locked (SCR-005)
+-- MFA Enrollment (SCR-006)
+-- MFA Verify (SCR-007)
+-- CAPTCHA Challenge (SCR-008)
+-- Logout / Session Expired (SCR-009)
+-- Role Redirect Gate (SCR-010)
+-- Admin Account Detail & Unlock (SCR-011)
+-- Admin Audit Log (SCR-012)

Screen Inventory

| Screen ID | Name | Derived From | Personas | Priority | States |
|-----------|------|--------------|----------|----------|--------|
| SCR-001 | Login (Email + Password) | UC-001, UC-002, UC-003 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-002 | Forgot Password Request | UC-005 | End User | P0 | Default, Loading, Empty (confirmation), Error, Validation |
| SCR-003 | Reset Password (token) | UC-006 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-004 | Reset Confirmation / Success | UC-006 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-005 | Account Locked / Lockout Notice | UC-004 | End User | P0 | Default (locked view), Loading, Empty (info), Error, Validation |
| SCR-006 | MFA Enrollment (QR + recovery) | FR-012 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-007 | MFA Verification (TOTP) | FR-012 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-008 | CAPTCHA Challenge | FR-011 | End User | P1 | Default (widget), Loading, Empty, Error, Validation |
| SCR-009 | Logout / Session Expired | UC-007 | End User | P1 | Default (interstitial), Loading, Empty, Error, Validation |
| SCR-010 | Role Redirect / Landing Gate | UC-008 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-011 | Admin - Account Detail & Unlock | UC-004 | Admin | P1 | Default, Loading, Empty (no user), Error, Validation |
| SCR-012 | Admin - Audit Log (list/filter) | FR-010 | Admin / Security | P2 | Default, Loading (skeleton table), Empty (no results), Error, Validation |

Modal/Overlay Inventory
- Confirm Unlock Modal — centered modal used by SCR-011 (fade + scale).
- CAPTCHA Modal (if external widget requires overlay) — slide/fade variation.
- Toasts/Alerts — top fixed, slide+fade.

---

## 5. Screen Specifications (all 5 states per screen)

Notes on consistent design primitives
- Container card width: mobile fill to safe area; desktop max-width 520px for auth card.
- Spacing tokens used (see designsystem.md): base 8px scale.
- Form fields: C/Inputs/TextField variants S/M; password field includes icon button trailing.
- Focus outline: 2px offset, color token focus (>=3:1 contrast).

SCR-001 — Login (Email + Password)
- Default
  - Elements: App logo, H1 "Sign in", subtitle, Email TextField (label, placeholder), Password TextField (visibility toggle), Remember me checkbox (optional), Primary Button "Sign in" (C/Button/Primary), Secondary link "Forgot password?", SSO/Other providers row (if configured), help link.
  - Behavior: Primary CTA enabled only when fields non-empty and client-side validation passes.
  - Keyboard: Enter submits; Tab order: Email → Password → Remember → Sign in → Forgot.
- Loading
  - Behavior: On submit, Primary button shows spinner + label "Signing in..."; full-card skeleton overlay if backend call >300ms (skeleton: avatar placeholder header and two row placeholders). Inputs disabled.
- Empty
  - Interpretation: Initial state (empty fields). Show helpful placeholder examples under inputs (aria-describedby).
- Error
  - Trigger: Server 401 / lockout.
  - Variants:
    - Invalid credentials: Top banner (error token) with "Invalid credentials" (aria-live="assertive"); field-level highlighting not used to avoid account enumeration.
    - Account Locked: Banner "Account is locked until [time]" with "Contact support" CTA and "Unlock via admin" info.
- Validation
  - Field-level inline errors: Required field messages (e.g., "Email is required"); email-format "Enter a valid email".
  - Errors appear on blur and on submit (aria-live polite for inline).
  - Password visibility toggle and strength meter appear per UXR-103 for password setting only (on Reset form).

SCR-002 — Forgot Password Request
- Default
  - Elements: H1 "Reset your password", Email TextField, Primary CTA "Send reset link", Secondary "Back to sign in".
  - UX: After submit show generic confirmation (Empty state) regardless of existence.
- Loading
  - CTA shows spinner; inputs disabled.
- Empty
  - Confirmation: Illustration + message "If an account exists, a reset email has been sent" + CTA "Back to sign in".
- Error
  - Email send failure: Banner "We couldn't send email right now. Try again." and "Retry" CTA. Provide help link for support.
- Validation
  - Inline field: required / email format errors.

SCR-003 — Reset Password (token)
- Default
  - Elements: H1 "Create a new password", New Password field (strength meter), Confirm Password field, Primary CTA "Set new password".
  - Behavior: Strength meter uses three levels (Weak/Medium/Strong) with suggestions.
- Loading
  - Button spinner; skeleton for main content if server validating token.
- Empty
  - If token already used/expired — show Empty/Info view with CTA "Request new link".
- Error
  - Invalid token: Card with error message and CTA to request new reset.
- Validation
  - Password mismatch inline, weak password guidance; ARIA mapping to inputs via aria-describedby.

SCR-004 — Reset Confirmation / Success
- Default
  - Message: "Your password has been reset. You may now sign in." CTA "Back to sign in".
- Loading
  - On finalizing update, spinner in CTA.
- Empty / Error / Validation states included similar to above.

SCR-005 — Account Locked / Lockout Notice
- Default (locked view)
  - Banner modal-like card describing lock reason, lock expiry if available, next steps (contact support link), link to unlock request modal.
- Loading
  - If checking unlock status, show spinner.
- Empty
  - If no additional support available, provide help contact.
- Error
  - If admin unlock fails, show error with retry.
- Validation
  - If user tries to submit unlock self-service form, inline validation.

SCR-006 — MFA Enrollment (QR + Recovery)
- Default
  - QR Graphic area, secret key displayed (masked with reveal), copy/download recovery codes, Primary CTA "Verify" to prompt code entry (SCR-007).
- Loading
  - QR generation spinner.
- Empty
  - If service cannot provision, show info and fallback instructions.
- Error
  - QR verify failure: inline error "Code didn't match".
- Validation
  - TOTP field validation (6-digit numeric, aria descriptions).

SCR-007 — MFA Verification (TOTP)
- Default
  - Input for TOTP code, Primary CTA "Verify", resend link for backup codes.
- Loading
  - CTA spinner during verification.
- Empty
  - After successful verify show success card and return to landing.
- Error / Validation
  - Code invalid: inline error + increment failed_attempts per FR-006.

SCR-008 — CAPTCHA Challenge
- Default
  - Embedded widget (C/Feedback/CAPTCHA). Provide accessible label and fallback description. If external (reCAPTCHA), use an overlay placeholder with link to provider.
- Loading
  - Widget loading skeleton.
- Empty
  - If no challenge available, fallback to additional friction (email verify).
- Error
  - Challenge failure: show retry and support CTA.
- Validation
  - Missing verification: inline prompt.

SCR-009 — Logout / Session Expired
- Default
  - Session-expired interstitial with CTA "Sign in again". ARIA focus set to H1.
- Loading
  - While re-authenticating, spinner.
- Empty / Error / Validation consistent with spec.

SCR-010 — Role Redirect / Landing Gate
- Default
  - Transitional view: show brief message "Redirecting to [role] dashboard" with skeletons; fallback link to dashboard.
- Loading
  - Show progress; handle mapping failure gracefully.
- Empty
  - If no mapped role, show guidance for admin contact (HTTP 403 case).
- Error
  - Role resolution failure: show error and support CTA.
- Validation
  - Role mapping mismatch fallback.

SCR-011 — Admin - Account Detail & Unlock
- Default
  - Search input, account detail card (status, failed_attempts, locked_until), Unlock button (confirm modal), audit event preview.
- Loading
  - Skeleton table and card placeholders.
- Empty
  - No results: illustration + help text.
- Error
  - API error: banner + retry.
- Validation
  - Search validation, unlock confirmation text validation (must type 'UNLOCK' for destructive action if configured).

SCR-012 — Admin - Audit Log
- Default
  - Table (C/Content/Table) with filters, date range picker, search, pagination. Rows: timestamp, event_type, user_id (hashed/obfuscated where needed), IP, outcome.
- Loading
  - Skeleton rows.
- Empty
  - "No events found" with timeframe controls.
- Error
  - Fetch error: banner + retry.
- Validation
  - Filter validation (date range rules).

State Naming & Figma Frames
- Frame names per figma-design-standards: e.g., Login/Default, Login/Loading, Login/Empty, Login/Error, Login/Validation.
- Export naming per standard: <AppName>__<Platform>__<ScreenName>__<State>__v1.jpg.

---

## 6. User Flows and Navigation

Flow: FL-001 — Sign In (UC-001)
1. Entry: SCR-001 Default.
2. Input validation (client-side).
3. Submit → SCR-001 Loading (CTA spinner).
4. Outcomes:
   - Success → SCR-010 Default (Role redirect) → dashboard.
   - Invalid credentials → SCR-001 Error (Invalid credentials).
   - Reached failed threshold → SCR-005 Default (Account Locked) → Admin unlock or auto-unlock.

Flow: FL-002 — Forgot / Reset (UC-005, UC-006)
1. Entry: SCR-001 click Forgot → SCR-002 Default.
2. Submit email → SCR-002 Loading.
3. Server sends email → SCR-002 Empty confirmation.
4. User opens email → SCR-003 Default via token link.
5. Submit new password → SCR-004 Default success.

Flow: FL-003 — MFA Enrollment & Verify (FR-012)
1. Entry: After login or profile settings → SCR-006 Default.
2. User scans QR, copies secret or download recovery codes → Verify (SCR-007).
3. On success, MFA enforced per role.

Flow: FL-004 — CAPTCHA Escalation (FR-011)
1. Detect suspicious pattern → show SCR-008 Default (CAPTCHA).
2. User completes widget → resume flow.

Flow: FL-005 — Admin Unlock & Audit (UC-004, FR-010)
1. Admin enters SCR-012, filters logs → locate user → open SCR-011.
2. Unlock via confirm modal → success and audit record created.

Prototype Entry Points
- /login
- /forgot-password
- /reset?token=
- /admin/audit

Prototype Requirements
- At least one validation flow (Login with errors).
- One empty state flow (no audit logs).
- One error retry flow (email send failure -> retry -> success).

---

## 7. Component Mapping

Figma Page & Component Organization
- 00_Cover: metadata
- 01_Foundations: tokens
- 02_Components: C/Actions, C/Inputs, C/Navigation, C/Content, C/Feedback
- 03_Patterns: Auth patterns, error/empty/loading patterns
- 04_Screens: all screen frames with states
- 05_Prototype: flows
- 06_Handoff: dev notes

Key Components & Variants (must exist)
- C/Actions/Button
  - Properties: Type (Primary, Secondary, Ghost), Size (S/M/L), State (Default/Hover/Focus/Active/Disabled/Loading), Icon (None/Leading/Trailing)
- C/Inputs/TextField
  - Variants: single-line, password (with visibility toggle), with prefix/suffix, state variants including error & success.
- C/Feedback/Banner
  - Types: Info, Success, Error, Warning. Must have aria-live variants.
- C/Feedback/Skeleton
  - Variants per content type: card, table row, form row.
- C/Feedback/Modal
  - Variants: Confirm, Informational; animation fade+scale.
- C/Content/Table
  - Variants: Default, Empty, Loading (skeleton rows). Column props and row actions (unlock).
- C/Feedback/CAPTCHA
  - Widget placeholder for integrations (reCAPTCHA, hCaptcha); include accessible label and fallback UI.
- C/Security/MFAEnrollment
  - QR container, secret copy, recovery code list (download/copy).
- C/Forms/PasswordStrength
  - Levels and messages; accessible text alternatives.

Component Properties Checklist
- Each component must define variant properties from figma-design-standards (Type, Size, State, Icon).
- All interactive components must document focus order and keyboard interactions in 06_Handoff.

Component-to-Screen Mapping (examples)
- Login: C/Inputs/TextField x2, C/Actions/Button Primary, C/Feedback/Banner.
- Reset Password: C/Inputs/TextField (password), C/Forms/PasswordStrength.
- Admin Audit: C/Content/Table, C/Actions/Button Secondary, C/Feedback/Modal (unlock confirm).

---

## 8. Interaction Patterns and Micro-animations

Motion Principles
- Durations: 150ms (micro-interactions, hover), 200-300ms (modals, skeleton reveal).
- Easing: ease-out for entrances, ease-in-out for toggles.
- Motion reduction: Honor prefers-reduced-motion: reduce animations to fade-only, or instant state change.
- Feedback latency: Immediate for client-side; server-loading patterns after 300ms show skeletons; button-level spinners for local actions.

Micro-interactions
- Button press: 150ms depressed state with subtle scale (0.98) + opacity change.
- Input focus: 2px outline with shadow token (elevation 1) and color token focus.
- Banner in/out: slide + fade 200ms.
- Modal: scale from 0.96 → 1 and fade-in 200ms (prefers-reduced-motion alternate: fade only).
- Skeleton shimmer: subtle linear gradient (disabled in reduced-motion).

Interaction Patterns
- Button-disabled-on-submit prevents double submits.
- Inline validation: on blur show errors; on submit show all relevant errors.
- Error banners: dismissible, appear at top of card and announced (aria-live="assertive").
- Toasts: used for transient success messages; auto-dismiss 4s, focusable when opened via keyboard.

Prototype Interactions to Implement
- Login submit with invalid credentials → Error banner (aria-live assertive) + keyboard focus stays on Email field.
- Forgot password submit → Confirmation empty state flow.
- Admin unlock → modal confirmation with typed confirmation if configured.

---

## 9. Accessibility Requirements (WCAG Compliance)

WCAG Baseline: 2.2 Level AA for all screens and components.

Key Requirements & Implementation Notes
- Contrast: Text ≥ 4.5:1; large text ≥ 3:1; UI components ≥ 3:1 when needed. Verify with contrast analyzer.
- Focus: Visible focus indicator for all interactive controls (>=3:1 contrast with adjacent backgrounds).
- Labels: All inputs have associated labels (visible or aria-label). Use aria-describedby for helper/error text.
- Live regions: Error banners use aria-live="assertive"; inline errors use aria-live="polite".
- Keyboard: Full keyboard operability. Modal focus trap (Escape closes modal), but ensure Escape is documented in 06_Handoff.
- Touch targets: Minimum 44x44px tappable area on mobile.
- Screen reader: Semantic structure: <main>, <form>, <nav> labels in handoff. Provide ARIA suggestions for each component.
- Forms: Required fields marked and programmatically indicated (aria-required). Error messages linked via aria-describedby to inputs.
- Images: Alt text for logos and meaningful images; decorative images alt="" or aria-hidden.
- Motion: Respect prefers-reduced-motion; provide accessible alternatives for critical feedback.
- Color independence: Do not rely on color alone for error states; combine with icons/text.
- Language & Localization: Directionality and text expansion considered.

Accessibility Annotations (to add in Figma)
- Focus order notes on each screen.
- ARIA role and label suggestions in component instances.
- Heading hierarchy and landmark annotations.

Testing & Acceptance
- Axe and Accessibility Insights passes for AA level on all default screens.
- Manual keyboard and NVDA/VoiceOver walkthroughs for critical flows (Login, Reset, MFA, Admin unlock).

---

## 10. Responsive Design Breakpoints

Baseline Frames (per figma-design-standards)
- Mobile: 390 x 844 (iPhone 14 baseline)
- Tablet: 768 x 1024 (iPad portrait)
- Web: 1440 x 1024 (Desktop baseline)

Layout Rules
- Grid: 12-column responsive grid on web (gutter 24/32 per breakpoint); mobile single column.
- Auth card: centered, max-width 520px on desktop, full-bleed with safe padding on mobile.
- Auto Layout: All frames use Auto Layout; nesting limited to 4 levels.

Behavior Per Breakpoint
- Mobile: Stack vertical, use bottom-anchored CTAs where appropriate, simplify visuals (hide extra help text behind "?" links).
- Tablet: Two-column patterns allowed for Admin tables (filters left, results right).
- Desktop: Full table with columns, multi-column forms supported.

Responsive Token Pitfalls
- Don’t change token values per breakpoint—apply different component layouts while reusing tokens.

---

## 11. Prototype & Export Requirements

Prototype Flows (must be wired)
- FL-001, FL-002, FL-003 (see User Flows).
- Include one validation flow, one empty-state flow, one error-retry flow.

Export Guidelines
- JPG exports: Quality 85%, 2x scale for mobile, 2x for web; sRGB color profile.
- Filenames per standard: <AppName>__<Platform>__<ScreenName>__<State>__v1.jpg.
- Produce export_manifest.md in 06_Handoff listing all exported screens with timestamps.

Handoff Notes
- 06_Handoff to include token usage guidelines, component props mapping, responsive behavior, accessibility notes, and code mapping suggestions.

---

## 12. Quality Checklist (Pre-Export)

- [ ] All screens include Default/Loading/Empty/Error/Validation frames.
- [ ] Components use tokens only (no literals).
- [ ] Contrast checks applied and pass WCAG AA.
- [ ] Focus states defined and visible for all interactive elements.
- [ ] Touch targets >=44x44px on mobile frames.
- [ ] Prototype flows wired and include required validation/empty/error flows.
- [ ] Naming conventions followed (C/..., Screen/State).
- [ ] Export manifest prepared.

---

## 13. Mapping to Personas & Acceptance Flow Matrix

Sample Screen → Persona Coverage
- SCR-001 (Login): End User primary, Admin secondary (admin may sign-in).
- SCR-011 (Admin Unlock): Admin primary only.
- SCR-012 (Audit Log): Admin & Security primary.

UX Acceptance Mapping
- UXR-201 (Accessibility) → All screens must pass accessibility acceptance (manual & automated).
- UXR-501 (Interaction) → Login & Reset must demonstrate client validation and server fallback in prototype.

---

## 14. Assumptions, Open Questions, & Next Steps

Assumptions
- External Email Service is available and supports TLS.
- SSO/OIDC is out-of-scope until FR-013 clarified.
- CAPTCHA provider config provided by infra.

Open Questions (flagged for product/security)
- FR-013 SSO details: which providers and claim mappings? [UNCLEAR]
- Admin unlock UX: require typed confirmation or simple confirm? (Implement simple confirm; add typed confirmation as configurable pattern.)
- Token storage strategy for client (HttpOnly cookie vs in-memory): document preferred strategy in Handoff.

Next Steps
1. Create Figma file with 6 pages and seed tokens in 01_Foundations.
2. Build component library in 02_Components following naming & variant rules.
3. Implement patterns and wire FL-001..FL-005 in prototype page.
4. Run accessibility checks and iterate.

End of Figma Design Specification.