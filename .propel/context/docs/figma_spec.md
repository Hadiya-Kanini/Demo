# Figma Design Specification

Platform: Responsive (Web / Mobile - iOS baseline)

---

## 1. UI Impact Assessment
Has UI Changes: Yes

Summary
- Source: Product Specification - Authentication & Account Recovery (FR-001 .. FR-011; UC-001 .. UC-009).
- UI Impact: High — multiple user-facing forms (Login, Forgot Password, Reset), role-based landing screens, and an Admin console to view/unlock accounts and inspect authentication events.
- Target platforms: Web (desktop first, 1440px baseline), Mobile (iPhone 14: 390x844), Tablet (iPad: 768x1024).
- Figma File Organization required: 6-page standard (00_Cover → 06_Handoff) per figma-design-standards.

Stakeholders
- Product Owner, Security Lead, Backend Lead, Frontend Lead, UX Researcher, Accessibility Specialist, Admin SME.

UI Impact Rationale
- Front-end validation, error messaging, lockout & rate-limit feedback, admin remediation UI are all explicitly required in functional requirements. All screens below are derived from Use Cases UC-001..UC-009.

---

## 2. UX Requirements

UXR Requirements Table
| UXR-ID | Category | Requirement | Acceptance Criteria | Screens Affected |
|--------|----------|-------------|---------------------|------------------|
| UXR-001 | Usability | Users must complete core auth flows (Login / Forgot password / Reset) within 3 clicks/taps from entry | Test script: login, forgot-password, reset each ≤3 interactions (click/tap) to reach CTA; observed in usability test (n=5) | SCR-001, SCR-002, SCR-003 |
| UXR-101 | Visual Design | Use design tokens for all components; no hard-coded colors | Figma file tokens populated; design audit shows 100% token usage | All screens |
| UXR-201 | Accessibility | All screens meet WCAG 2.2 AA for text and UI contrast & keyboard support | Automated axe / manual audit passes; keyboard-only navigation completes flows; focus visible | All screens |
| UXR-202 | Accessibility | Auth error messages must be announced via aria-live and not cause focus loss | Screen reader test: error announced; focus placed on first invalid field | SCR-001, SCR-002, SCR-003 |
| UXR-301 | Responsiveness | Layouts responsive at Mobile/Tablet/Desktop breakpoints with defined behavior | Screens adapt per breakpoints; components use Auto Layout with fill/hug directives | All screens |
| UXR-401 | Interaction | Submits must provide immediate feedback (spinner or skeleton) within 200ms | Prototype measurement: spinner/skeleton visible within 200ms; CTA disabled while pending | SCR-001, SCR-002, SCR-003, SCR-007 |
| UXR-501 | Error Handling | Credential mismatch errors must be generic and not leak account existence | Copy review: message "Invalid credentials"; tests confirm no email existence clues | SCR-001 |
| UXR-601 | Security UX | Account locked flows must present recovery options: Forgot Password & Contact Admin | Locked state shows CTAs; unlock path test completes | SCR-001 (Locked state), SCR-005 |
| UXR-701 | Admin UX | Admin must filter and find locked accounts in ≤2 actions and unlock with confirmation | Usability test: unlock complete in ≤60s; audit entry logged | SCR-007, SCR-009, SCR-008 |
| UXR-801 | Performance UX | Lists should use skeletons and progressive loading for >300ms loads | When API >300ms, skeleton visible; list populates without layout shift | SCR-006, SCR-007, SCR-008 |
| UXR-901 | Internationalization | All text must be replaceable; avoid embedding text in images | All copy in text layers; no hard-coded strings in images | All screens |

Notes:
- Tagging: UXR-200+ Accessibility; UXR-300+ Responsiveness; UXR-400+ Visual Design; UXR-500+ Interaction; UXR-600+ Error Handling; UXR-700+ Admin UX; UXR-900+ i18n.

---

## 3. Persona Analysis and User Journeys

Personas (summary)
- Persona A: End User (Customer / Employee)
  - Role: Primary consumer; goal: authenticate quickly and reach role-specific workspace.
  - Key needs: Fast login, clear errors, easy recovery for locked accounts.
  - Key screens: SCR-001, SCR-002, SCR-003, SCR-006, SCR-010.
- Persona B: Administrator
  - Role: Operator managing accounts and responding to alerts.
  - Key needs: Fast search/filter, clear attempt history, safe unlock/force reset actions with audit.
  - Key screens: SCR-007, SCR-008, SCR-009, SCR-012.

User Journey Examples
- End User - Successful Login
  1. Entry: Landing (public) -> SCR-001 (Login Default)
  2. Submit -> SCR-001 (Loading) -> SCR-006 (Role-based Dashboard Default)
  3. Post: optional session expiry flow -> SCR-010 (Session Expired)
- End User - Locked Account Recovery
  1. Attempt Login -> SCR-001 (Error Locked)
  2. Choose "Forgot Password" -> SCR-002 -> SCR-003 (Reset) -> SCR-004 (Success)
- Admin - Unlock Flow
  1. Admin navigates to Admin console -> SCR-007 (Locked Accounts list)
  2. Select account -> SCR-008 (Detail + attempts) -> open SCR-009 (Confirm Unlock) -> action logged

Mapping persona to screens included in Screen Inventory.

---

## 4. Information Architecture & Screen Inventory

Site Map (top-level)
[Auth Area]
+-- Login (SCR-001)
+-- Forgot Password (SCR-002)
+-- Reset Password (SCR-003)
+-- Reset Confirmation (SCR-004)
+-- Locked Info (SCR-005)
+-- Role Dashboards (SCR-006)
[Admin Area]
+-- Locked Accounts List (SCR-007)
+-- Account Detail / Attempts (SCR-008)
+-- Unlock Confirm (SCR-009)
+-- Session Expired Prompt (SCR-010)
+-- Rate Limit / Too Many Requests (SCR-011)
+-- Suspicious Activity Queue (SCR-012)

Screen List (derived from Use Cases)
| Screen ID | Name | Derived From | Personas | Priority | States |
|-----------|------|--------------|----------|----------|--------|
| SCR-001 | Login Screen | UC-001, UC-002, UC-003, UC-004 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-002 | Forgot Password Request | UC-005 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-003 | Reset Password Form | UC-006 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-004 | Reset Success Confirmation | UC-006 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-005 | Account Locked Info (variant of Login) | UC-004 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-006 | Role-based Dashboard (scaffold) | FR-003 | End User (roles) | P0 | Default, Loading, Empty, Error, Validation |
| SCR-007 | Admin - Locked Accounts List | FR-009, UC-007 | Admin | P0 | Default, Loading, Empty, Error, Validation |
| SCR-008 | Admin - Account Detail & Attempts | FR-009, FR-008 | Admin | P0 | Default, Loading, Empty, Error, Validation |
| SCR-009 | Admin - Unlock / Force Reset Modal | UC-007 | Admin | P0 | Default, Loading, Empty, Error, Validation |
| SCR-010 | Session Expired / Token Refresh Prompt | UC-008 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-011 | Rate Limit Banner / Guidance | FR-010 | All | P1 | Default, Loading, Empty, Error, Validation |
| SCR-012 | Admin - Suspicious Activity Queue | FR-011 (optional) | Admin | P1 | Default, Loading, Empty, Error, Validation |

Notes:
- SCR-005 is implemented as a Login variant/modal when server returns locked status, not a completely separate page unless admin/UX decides otherwise.
- All screens must include header landmark, main, and footer landmarks for accessibility.

---

## 5. Screen Specifications (each with 5 states)

Design conventions for all screens:
- Default desktop width: 1040px content container centered on 1440px canvas.
- Mobile viewport: 390x844; touch targets min 44x44px.
- All frames must use Auto Layout; nested auto-layout no deeper than 4 levels.
- Spacing tokens: 8px base; use multiples: 8,12,16,20,24,32,40.
- Typography tokens from Design System (see designsystem.md).
- State naming: <ScreenName>/<State> (e.g., Login/Default).

SCR-001 — Login Screen
- Purpose: Primary credential entry point.
- Elements: Header (logo, skip link), Form (Email, Password, Remember me checkbox), Primary CTA (Login - C/Actions/Button/Primary), Secondary Link (Forgot password), Error/Info banner area, Footer help links.
- Behavior: Enter submits; Enter key triggers Login; accessible labels for inputs.
- States:
  - Default: Pre-filled placeholder examples (email placeholder), CTA enabled. Focus order: email -> password -> remember -> login -> forgot.
    - Acceptance: Focus visible on first interactable; fields have clear labels.
  - Loading: CTA disabled, inline spinner within CTA, fields disabled, aria-busy on form container. Show skeleton of banner space preserved.
    - Acceptance: Spinner appears <200ms; button disabled.
  - Empty: Minimal help text shown when no saved accounts detected; optional "Create account" CTA if available (out-of-scope).
    - Acceptance: Layout preserved; no shift on load.
  - Error: Generic error banner "Invalid credentials" surfaced; variant for Locked shows "Account is locked" with CTAs: "Forgot password" and "Contact support". Banners are role-neutral and use aria-live="polite".
    - Acceptance: Screen reader reads the error; message does not indicate email existence.
  - Validation: Inline field errors for required/format (e.g., "Please enter a valid email"). Inputs get error border (token color.error), error text with aria-describedby, first invalid field receives focus.
    - Acceptance: Field-level error announced; client prevents submit.

SCR-002 — Forgot Password Request
- Elements: Email input, Submit CTA ("Send reset link"), info copy about rate limit, Retry-After area.
- States: Default (form), Loading (sending), Empty (no input helper), Error (429 rate-limit or delivery failure with Retry-After), Validation (invalid email).
- Acceptance: On success show confirmation screen (SCR-004 style) or inline confirmation message; retry guidance when header Retry-After present.

SCR-003 — Reset Password Form
- Elements: Password, Confirm Password, password-strength/requirements helper, CTA Submit.
- States:
  - Default: Shows criteria (length, character classes).
  - Loading: CTA disabled, spinner, show progress.
  - Empty: Token missing / malformed -> show "Invalid link" state with CTA to request new.
  - Error: Token expired -> show "Reset token expired" with "Request new link" CTA.
  - Validation: Invalid password inline errors, strength meter updates; aria-live for policy violations.
- Acceptance: On success, existing sessions invalidated (backend); UI shows success state.

SCR-004 — Reset Success Confirmation
- Elements: Success message, CTA to Login, note about session invalidation.
- States: Default (confirmation), Loading (n/a), Empty/Error/Validation present where relevant (e.g., account missing).
- Acceptance: Clear next-step CTA.

SCR-005 — Account Locked Info (Login Variant)
- Elements: Banner with locked message, unlock options, time remaining if available, "Contact Support" link.
- States: Default (locked message), Loading (sending unlock email), Empty (no recovery options), Error (email sending failure), Validation (invalid contact input if inline form present).
- Acceptance: Contains options to recover; no private data displayed.

SCR-006 — Role-based Dashboard (scaffold)
- Elements: Header, sidebar or top nav, main content placeholder. Landing payload per role.
- States: Default (loaded content), Loading (skeleton content), Empty (no data cards), Error (fetch failed), Validation (n/a).
- Acceptance: Redirect after login lands here; role-specific permissions enforced.

SCR-007 — Admin - Locked Accounts List
- Elements: Search, filters (time window, role), paginated/sortable table or list, bulk actions, badges for lock count.
- States: Default (rows), Loading (skeleton rows), Empty (no locked accounts), Error (permission or server error), Validation (search input).
- Acceptance: Rows show last_failed_at, attempts_count, last_ip (masked), actions menu.

SCR-008 — Admin - Account Detail & Attempts
- Elements: Account header, recent attempts list (timestamp, IP, user-agent, outcome), metadata, unlock/force-reset action.
- States: Default, Loading (skeleton), Empty (no attempts), Error, Validation.
- Acceptance: Attempts list shows up to configured N entries; PII minimized (IP masked partial).

SCR-009 — Admin - Unlock / Force Reset Modal
- Elements: Modal title, confirm copy, input (optional reason), Confirm & Cancel buttons.
- States: Default (confirm), Loading (processing), Empty (n/a), Error (failure), Validation (required confirmation).
- Acceptance: Action requires one-step confirmation; action logged with admin id.

SCR-010 — Session Expired / Token Refresh Prompt
- Elements: Inline banner or modal, CTA to refresh or re-login, optional silent-refresh attempt.
- States: Default (prompt), Loading (refresh in progress), Empty, Error (refresh failed), Validation.
- Acceptance: If refresh fails, redirect to SCR-001.

SCR-011 — Rate Limit / Too Many Requests Banner
- Elements: Banner with Retry-After guidance, suggested back-off and contact support link.
- States: Default, Loading (n/a), Empty, Error, Validation.
- Acceptance: Banner shows Retry-After header if provided.

SCR-012 — Admin - Suspicious Activity Queue
- Elements: Alert list, triage actions (Acknowledge, Investigate), link to account detail.
- States: Default, Loading, Empty (no alerts), Error, Validation.
- Acceptance: Alerts contain sufficient context for triage; action logging required.

Design Tokens referenced in above screens live in designsystem.md; all screen styles must map to tokens via token keys (e.g., color.error, typography.body.medium).

---

## 6. User Flows and Navigation

Flow: FL-001 — Login Success (derived UC-001)
1. SCR-001/Default -> user submits -> SCR-001/Loading
2. Server returns success -> app redirects to SCR-006/Default based on role.
3. Metrics: login_success event logged; UI shows spinner until redirect.

Flow: FL-002 — Login Invalid / Lockout (UC-002 / UC-004)
1. SCR-001/Default -> submit -> SCR-001/Loading
2. Server returns 401 -> SCR-001/Error ("Invalid credentials")
3. After threshold reached -> SCR-001/Error ("Account is locked") with unlock CTAs -> user chooses Forgot -> SCR-002.

Flow: FL-003 — Forgot Password & Reset (UC-005 / UC-006)
1. SCR-001 -> "Forgot password" -> SCR-002/Default -> submit -> SCR-002/Loading
2. On success -> confirmation inline or SCR-004 -> email link -> SCR-003/Default with token
3. Submit new password -> SCR-003/Loading -> SCR-004/Default on success.

Flow: FL-004 — Admin Unlock (UC-007)
1. Admin -> SCR-007/Default -> select account -> SCR-008/Default -> open SCR-009 -> confirm -> SCR-009/Loading -> success toast -> SCR-007 refresh.

Flow: FL-005 — Token Expiry (UC-008)
1. SCR-006 receives 401 -> SCR-010/Default prompt -> attempt silent refresh (if implemented) -> success redirect or fall back to SCR-001.

Prototype interactions to include:
- Validation flow (FL-002 & FL-003)
- Empty state flow (SCR-007 empty to create help task)
- Error retry flow (SCR-002 429 -> retry after visible -> retry success)

Navigation Patterns
- Web: Top header with contextual "Help" / "Contact support"; primary nav in admin is left sidebar; auth area minimal header.
- Mobile: Minimal header; "Back" navigation accessible; bottom sheet used for admin actions if screen constrained.

---

## 7. Component Mapping (C/<Category>/<Name>)

All components must follow naming convention and live under 02_Components in Figma file.

Core components required
- C/Actions/Button — Variants: Primary / Secondary / Ghost / Tertiary; Sizes: S/M/L; States: Default/Hover/Focus/Active/Disabled/Loading. Used on SCR-001..SCR-009.
- C/Inputs/TextField — Variants: Email, Password (masked), Search; States: Default/Focus/Error/Disabled/ReadOnly. Includes helper text slot and icon leading/trailing.
- C/Forms/Checkbox — Small checkbox with label and error state.
- C/Feedback/Banner — Info / Error / Success / Warning with CTA slot; accessible role and aria-live variants.
- C/Feedback/SkeletonRow — Table/row skeleton for lists.
- C/Layouts/Container — Content container with responsive behaviors (max-width tokens).
- C/Navigation/Header — Public header with skip link.
- C/Navigation/Sidebar — Admin nav with active state and keyboard focus handling.
- C/Content/Table — Rows with actions column; responsive collapse to list on mobile.
- C/Modal/Confirm — Centered modal with confirm/cancel.
- C/Status/Badge — Lock badge, error badge, count badge.
- C/Forms/PasswordStrength — Strength meter with accessible text alternative.
- C/Toast/Notification — Top/bottom anchored messages for success/error; aria-live polite.

Component Usage Notes
- Buttons must map to token: button.primary.background, button.primary.text, etc.
- Text fields must expose props: label, placeholder, helpText, errorText, required, aria-describedby id.
- All components to provide focus state with min 3:1 contrast relative to background.

---

## 8. Interaction Patterns & Micro-animations

Interaction Principles
- Keep micro-interactions short: 150ms for hover/press, 200-300ms for enter/exit transitions.
- Feedback latency: first visual response within 200ms (spinner, skeleton).
- Motion respects prefers-reduced-motion.

Micro-animations
- Button Press: subtle scale to 0.98 over 120ms, ease-out.
- Banner appear/disappear: slide down + fade 200ms, easing ease-out.
- Modal open/close: fade + scale (0.96 → 1) over 180ms.
- Skeleton shimmer: slow linear gradient 800ms loop, respects reduced motion.
- Password strength updates: width animation 150ms, color transition 150ms.

Interaction Edge Rules
- Loading states preserve layout to avoid content shift.
- For long operations (>3s) show progress or secondary message.
- All animated state transitions must be documented per component variant.

---

## 9. Accessibility Requirements (WCAG & Implementations)

Target: WCAG 2.2 Level AA (per web-accessibility-standards) for all UI screens.

Design-time checks (must be annotated in Figma)
- Color contrast:
  - Body text: >= 4.5:1
  - Large text: >= 3:1
  - UI components (controls, icons): >= 3:1 for visual distinction
- Focus:
  - Visible focus indicator for every interactive element (min 2px outline, >=3:1 contrast).
  - Focus order follows DOM reading order; annotate focus order on screens.
- Keyboard:
  - All interactive elements reachable by keyboard; modal traps must be closable with Esc.
  - Skip-to-main link present on all pages.
- Screen reader:
  - Error/success banners: role="status" or aria-live="polite" for non-blocking; aria-live="assertive" for blocking errors.
  - Inputs: label elements bound programmatically; error messages referenced by aria-describedby.
- Forms:
  - Required fields use aria-required and visible hint.
  - Inline validation: announce via aria-live and set focus to first error.
- Images:
  - Illustrations have alt text or role="img" with aria-label.
- Touch targets:
  - Minimum 44x44px tappable area on mobile for all CTAs.
- No information by color alone:
  - Use icons, text, or badges for status (locked/unlocked).

Accessibility Annotations Required in Figma
- Focus order on each screen frame
- ARIA label suggestions for custom components
- Landmark annotations (<header>, <main>, <nav>, <footer>)
- Error behavior notes and screen reader text
- High contrast / dark mode variant checks

---

## 10. Responsive Design Breakpoints & Behavior

Breakpoints
- Mobile: 375-420px (iPhone 14 baseline 390px)
- Tablet: 768px
- Desktop: 1024px and up; design baseline 1440px canvas with 1040px content width

Behavior rules
- Navigation:
  - Desktop: Admin sidebar persistent (C/Navigation/Sidebar) + top header.
  - Tablet: collapsed sidebar or top tab nav.
  - Mobile: sidebar collapses into menu; admin actions exposed via bottom sheet or secondary modal.
- Tables:
  - Desktop: full table.
  - Tablet: responsive table with horizontal scroll OR row expansion.
  - Mobile: collapsed list with primary info and contextual actions in overflow menu.
- Forms:
  - Inputs stack vertically on mobile; use full width.
  - Buttons: primary CTA full-width on mobile for single primary action.
- Components resizing:
  - Use Auto Layout Fill for content areas and Hug for content-driven sizes; document each component's resize behavior.

---

## 11. Export & Naming Conventions

Per figma-design-standards:
- Export filenames: <AppName>__<Platform>__<ScreenName>__<State>__v<Version>.jpg
- Example: AuthApp__Mobile__Login__Default__v1.jpg
- JPG settings: Quality 85%, scale 2x for mobile, sRGB.

Export Manifest to be generated in 06_Handoff page listing all screens and states.

---

## 12. Handoff Notes (06_Handoff)

Developer Notes to include:
- Token usage map (token key -> design intent)
- Component props mapping (Button.primary -> buttonPrimary in code)
- Responsive behavior and breakpoints
- Accessibility mapping: aria- attributes suggestions and focus order
- API contract pointers: POST /login, POST /forgot-password, POST /reset-password, GET /admin/login-attempts
- Edge cases: token expiration copy, rate-limit messages, account locked copy

Quality Checklist before export:
- [ ] All screens include Default/Loading/Empty/Error/Validation frames
- [ ] No hard-coded colors/text inside images
- [ ] Focus and ARIA annotations present
- [ ] Export manifest complete

---

## 13. Prototype Standards & Required Flows

Prototype must demonstrate:
- Login success and failure flow including Validation (FL-001, FL-002)
- Forgot Password -> Reset -> Success flow (FL-003)
- Admin unlock flow (FL-004)
- Error retry flow for rate-limited action (FL-005)
- Include reduced-motion toggles in prototype

Prototype interactions:
- Input validation and aria-live announcements
- Banner error flows with retry
- Modal confirm unlock flow with audit confirmation

---

## 14. Appendix: Screen-to-Persona Matrix (summary)
| Screen | End User | Admin |
|--------|----------|-------|
| SCR-001 Login | Primary | Secondary (admin logs in here) |
| SCR-002 Forgot Password | Primary | - |
| SCR-003 Reset Password | Primary | - |
| SCR-006 Dashboard | Primary (role) | Primary (admin) |
| SCR-007 Locked Accounts | - | Primary |
| SCR-008 Account Detail | - | Primary |
| SCR-009 Unlock Modal | - | Primary |
| SCR-012 Suspicious Queue | - | Primary |

---

Versioning and next steps
- Create Figma file with pages: 00_Cover → 06_Handoff
- Populate 01_Foundations from designsystem.md tokens
- Build components in 02_Components with required variants and states
- Implement all screens in 04_Screens including five state frames
- Wire prototypes in 05_Prototype to cover flows listed
- Run accessibility audit (axe) and iterate

Why this approach
- All screens are derived from use cases (UC-001..UC-009); states required by figma-design-standards are enumerated to ensure handoff completeness and accessibility compliance. This minimizes development ambiguity and supports measurable acceptance criteria.