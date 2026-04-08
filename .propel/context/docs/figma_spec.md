## Figma Design Specification — AuthApp

## 1. Figma Specification
**Platform**: Responsive (Web / Mobile - iOS & Android)

---

## 2. Source References

### Primary Source
| Document | Path | Purpose |
|----------|------|---------|
| Requirements | `.propel/context/docs/spec.md` | Personas, use cases, functional requirements (login/auth flows, FR-001..FR-014) |

### Optional Sources
| Document | Path | Purpose |
|----------|------|---------|
| Wireframes | `.propel/context/wireframes/` | Low-fidelity layout and early structure for auth screens |
| Design Assets | `.propel/context/Design/` | Visual references used to derive tokens and component visuals |

### Related Documents
| Document | Path | Purpose |
|----------|------|---------|
| Design System | `.propel/context/docs/designsystem.md` | Canonical tokens, components, and usage guidance |
| Security Runbook | `.propel/context/docs/security_runbook.md` | Token lifecycle, hashing guidance for copy/notes in UI flows |

---

## 3. UX Requirements

### UXR Requirements Table
| UXR-ID | Category | Requirement | Acceptance Criteria | Screens Affected |
|--------|----------|-------------|---------------------|------------------|
| UXR-001 | Project | Maintain a canonical Figma file and token library for AuthApp | Figma file pages 00-06 present; tokens defined in 01_Foundations; component variants in 02_Components | All screens |
| UXR-101 | Usability | Login to dashboard reachable in ≤ 3 interactions from entry | Measured click/tap count ≤3 including submit and redirect; documented flow in prototype | SCR-001, SCR-006, SCR-007/008/009 |
| UXR-102 | Usability | Post-login routing shows progress and fallback safe landing | Loading skeleton shown during server role resolution; fallback page with support CTA if mapping missing | SCR-006, SCR-013 |
| UXR-103 | Usability | Inline client validation and focus-first-invalid behavior | On invalid submit focus moves to first invalid field; inline message visible; SR announcement present | SCR-001, SCR-003, SCR-014 |
| UXR-201 | Accessibility | All UI meets WCAG 2.2 AA baseline | Automated contrast checks pass; keyboard navigation validated; focus visible on all interactive elements | All screens |
| UXR-202 | Accessibility | Error and status messages announced to screen readers | ARIA live regions used; error dialogs trap focus; tests with screen reader assertions pass | SCR-001, SCR-002, SCR-003, SCR-005 |
| UXR-301 | Responsiveness | Layout adapts to mobile, tablet, desktop breakpoints | Visual QA at 320/375/768/1024 widths; no horizontal scroll; components reflow | All screens |
| UXR-302 | Responsiveness | Microcopy informs users about token storage & session persistence | Microcopy present on login and session-expired screens describing cookie vs secure storage | SCR-001, SCR-010 |
| UXR-401 | Visual | Use tokenized color/typography/spacing; components reference tokens | All components reference tokens in 01_Foundations; design library updated | All screens |
| UXR-402 | Visual | Copy is non-revealing and privacy-preserving | Forgot-password and failed-login copy audited; messages do not indicate account existence | SCR-002, SCR-004, SCR-013 |
| UXR-501 | Interaction | Immediate feedback for primary actions; prevent double submit | Submit buttons disable on submit and show spinner within 200ms; network states handled | SCR-001, SCR-003, SCR-011 |
| UXR-502 | Interaction | Respect prefers-reduced-motion and provide reduced animation variants | All motion documented; reduced-motion variant available in components | All screens |
| UXR-601 | Error Handling | Use generic error responses and provide recovery paths | Login errors present generic message; forgot-password returns generic success message; retry paths present | SCR-001, SCR-002, SCR-013 |
| UXR-602 | Error Handling | Account lockout guidance clearly explains next steps and support options | Lockout screen shows locked_until TTL or admin support CTA and link to forgot-password | SCR-005, SCR-012 |
| UXR-603 | Error Handling | Rate-limit UI provides Retry-After and CAPTCHA flow when required | 429 UI displays Retry-After; CAPTCHA widget area reserved and accessible | SCR-011, SCR-001 |

### UXR Categories
- Usability, Accessibility, Responsiveness, Visual Design, Interaction, Error Handling, Project-wide

### UXR Derivation Logic
- Usability UXRs are derived from UC-001 success path requirements (login → dashboard).
- Accessibility UXRs derived from WCAG 2.2 AA and FR-002/FR-013.
- Responsiveness from platform targets and designsystem breakpoints.
- Visual UXRs map to designsystem token usage and brand constraints.
- Interaction UXRs map to FR-005 (submission) and performance targets.
- Error UXRs derived from FR-006, FR-007, FR-008 security constraints.

### UXR Numbering Convention
- UXR-001 to UXR-099: Project-wide
- UXR-1XX: Usability
- UXR-2XX: Accessibility
- UXR-3XX: Responsiveness
- UXR-4XX: Visual design
- UXR-5XX: Interaction
- UXR-6XX: Error handling

---

## 4. Personas Summary

| Persona | Role | Primary Goals | Key Screens |
|---------|------|---------------|-------------|
| End User | Registered customer/employee | Authenticate quickly, reach role-specific dashboard, recover access if needed | SCR-001, SCR-002, SCR-003, SCR-006, SCR-007 |
| Administrator | Admin/support operator | Unlock accounts, view audit context, assist users | SCR-012, SCR-005, SCR-008 |
| Security Operator | Security/monitoring | Monitor auth events, respond to anomalies | SCR-011, SCR-008, SCR-013 |

---

## 5. Information Architecture

### Site Map
```
AuthApp
+-- Public
|   +-- Landing
|   +-- Login (SCR-001)
|   +-- Forgot Password (SCR-002)
|   +-- Reset Password (SCR-003)
|   +-- Reset Confirmation (SCR-004)
|   +-- Account Locked (SCR-005)
+-- Authenticated
|   +-- Role Redirect (SCR-006)
|   +-- Customer Dashboard (SCR-007)
|   +-- Admin Dashboard (SCR-008)
|   +-- Employee Dashboard (SCR-009)
|   +-- Session Expired (SCR-010)
|   +-- Rate Limit / CAPTCHA (SCR-011)
|   +-- Admin: User Detail & Unlock (SCR-012)
|   +-- Generic Error (SCR-013)
|   +-- Password Policy Helper (SCR-014)
```

### Navigation Patterns
| Pattern | Type | Platform Behavior |
|---------|------|-------------------|
| Primary Nav | Header (desktop) / BottomNav (mobile) | Desktop shows header + optional sidebar; mobile shows simplified bottom nav with auth actions hidden when unauthenticated |
| Utility Nav | User menu | Access to profile, sign out, session info (desktop right corner; mobile in bottom drawer) |
| Secondary Nav | Tabs | Use inside dashboards for section switching (desktop & mobile responsive) |
| Modal/Dialog | Centered Modal | Used for confirm/unlock; traps focus; Escape closes |

---

## 6. Screen Inventory

### Screen List
| Screen ID | Screen Name | Derived From | Personas Covered | Priority | States Required |
|-----------|-------------|--------------|------------------|----------|-----------------|
| SCR-001 | Login | UC-001, FR-001/002/005/013 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-002 | Forgot Password (Request) | UC-002, FR-007/013 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-003 | Reset Password (Form) | UC-002, FR-003/007 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-004 | Password Reset Confirmation | UC-002, FR-007 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-005 | Account Locked / Lockout Guidance | UC-003, FR-006/013 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-006 | Role Redirect / Loading Landing | UC-004, FR-005 | End User | P0 | Default, Loading, Empty, Error, Validation |
| SCR-007 | Customer Dashboard (placeholder) | UC-004 | End User (Customer) | P0 | Default, Loading, Empty, Error, Validation |
| SCR-008 | Admin Dashboard (placeholder) | UC-004, FR-009 | Administrator | P0 | Default, Loading, Empty, Error, Validation |
| SCR-009 | Employee Dashboard (placeholder) | UC-004 | Employee | P0 | Default, Loading, Empty, Error, Validation |
| SCR-010 | Session Expired / Reauthenticate | FR-004/011 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-011 | Rate Limit / CAPTCHA | FR-008 | End User | P1 | Default, Loading, Empty, Error, Validation |
| SCR-012 | Admin: User Detail & Unlock | FR-012 | Administrator | P1 | Default, Loading, Empty, Error, Validation |
| SCR-013 | Generic Error / Non-revealing Error Banner | FR-013 | All | P0 | Default, Loading, Empty, Error, Validation |
| SCR-014 | Password Policy & Strength Helper | FR-002/003 | End User | P1 | Default, Loading, Empty, Error, Validation |

### Priority Legend
- P0: Critical path (must-have for MVP)
- P1: Core functionality (high priority)
- P2: Important features (medium priority)
- P3: Nice-to-have (low priority)

### Screen-to-Persona Coverage Matrix
| Screen | End User | Administrator | Security Operator | Notes |
|--------|----------|---------------|-------------------|-------|
| SCR-001 | Primary | Secondary | - | Entry point for authentication |
| SCR-002 | Primary | - | - | Account recovery (non-revealing) |
| SCR-005 | Primary | Secondary | - | Lockout info with admin contact |
| SCR-008 | - | Primary | Primary | Admin and security monitoring |

### Modal/Overlay Inventory
| Name | Type | Trigger | Parent Screen(s) | Priority |
|------|------|---------|------------------|----------|
| Login Modal (optional) | Modal | "Login" click on header | Header / Protected pages | P0 |
| Confirm Unlock | Dialog | Admin "Unlock" action | SCR-012 | P0 |
| CAPTCHA Overlay | Inline widget | Trigger after rate-limit threshold | SCR-001 / SCR-011 | P1 |
| Session Expiry Banner | Banner/Modal | Token expiry detected | Any authenticated screen | P1 |

---

## 7. Content & Tone

### Voice & Tone
- Overall Tone: Professional, approachable, security-conscious
- Error Messages: Helpful, non-blaming, actionable, non-revealing
- Empty States: Encouraging, guiding, with clear CTA to recover or contact support
- Success Messages: Brief, confirmatory, next-action oriented

### Content Guidelines
- Headings: Sentence case for clarity (e.g., "Reset your password")
- CTAs: Action-oriented, specific verbs (e.g., "Send reset instructions", "Sign in")
- Labels: Concise, descriptive (e.g., "Email address", "Password")
- Placeholder Text: Provide examples where helpful (e.g., "name@example.com"); not used for critical labels

---

## 8. Data & Edge Cases

### Data Scenarios
| Scenario | Description | Handling |
|----------|-------------|----------|
| No Data | User has no account history | Empty state with CTA to sign up or contact support |
| First Use | New user with no prior sessions | Onboarding microcopy and guidance; password helper shown on reset first-use |
| Large Data | Admin views large audit logs | Pagination, server-side filtering, virtualization for lists |
| Slow Connection | >3s load time | Skeleton screens for lists and login submit shows spinner; offline hint if offline detected |
| Offline | No network | Informational state with retry; persistent queued actions not allowed for auth |

### Edge Cases
| Case | Screen(s) Affected | Solution |
|------|-------------------|----------|
| Long email/name | All | Truncate with tooltip; allow full copy on focus |
| Missing profile image | Dashboards | Use avatar fallback token (icon or initials) |
| Token reuse attempt | SCR-003/SCR-010 | Show generic token-invalid message with re-request CTA |
| Legacy weak hash present | Admin/UIs | Show progress/status in admin tools; rehash on next login (ops note) |
| Concurrent unlock attempts | SCR-012 | Optimistic UI with server confirmation and audit logging |

---

## 9. Branding & Visual Direction
- Reference canonical tokens in `.propel/context/docs/designsystem.md` — use token names (e.g., color.primary).
- Logo: Provide primary and stacked variants in 01_Foundations assets; path: `.propel/context/Design/logo/` [TO BE UPDATED]
- Icon Style: Filled, geometric icons (consistent stroke widths) — guidelines in designsystem.md
- Illustration Style: Minimal flat illustrations for empty & error states; limited palette using semantic colors
- Photography Style: Not required for MVP; if used, prefer neutral contextual images cropped to focus on subject

---

## 10. Component Specifications

### Component Library Reference
**Source**: `.propel/context/docs/designsystem.md` — Component Specifications section (C/<Category>/<Name> naming)

### Required Components per Screen
| Screen ID | Components Required | Notes |
|-----------|---------------------|-------|
| SCR-001 | C/Inputs/TextField (email), C/Inputs/TextField (password with toggle), C/Actions/Button (Primary), Link | Login form; password show/hide |
| SCR-002 | C/Inputs/TextField (email), C/Actions/Button (Primary) | Non-revealing confirmation flow |
| SCR-003 | C/Inputs/TextField (password), C/Inputs/TextField (confirm password), Password strength (C/Content/PasswordHelper), Button | Token validation and inline policy hints |
| SCR-005 | C/Feedback/Alert (Lockout), C/Actions/Button (Contact Support) | Lockout guidance with TTL or admin unlock CTA |
| SCR-011 | C/Feedback/CAPTCHA, C/Feedback/Alert, Button | Rate-limit recovery flow |

### Component Summary
| Category | Components | Variants |
|----------|------------|----------|
| Actions | C/Actions/Button | Primary, Secondary, Ghost x S/M/L x States (Default, Hover, Focus, Active, Disabled, Loading) |
| Inputs | C/Inputs/TextField, C/Inputs/PasswordField, C/Inputs/Select | Sizes S/M/L; States incl. Validation/Error |
| Feedback | C/Feedback/Alert, C/Feedback/Toast, C/Feedback/Skeleton | Info/Success/Warning/Error variants |
| Content | C/Content/PasswordHelper, C/Content/Card | Inline helper and policy guidance |

### Component Constraints
- Use only tokens from designsystem.md (e.g., color.primary, type.scale.h6)
- Name components per C/<Category>/<Name>
- All components must support the required states: Default, Hover, Focus, Active, Disabled, Loading
- Auto Layout required; spacing using spacing tokens scale (4,8,12,16,24,32...)

---

## 11. Prototype Flows

### Flow: FL-001 - Login Success Flow
**Flow ID**: FL-001  
**Derived From**: UC-001  
**Personas Covered**: End User  
**Description**: User submits valid credentials and is redirected to role-specific dashboard.

#### Flow Sequence
1. Entry: SCR-001 / Default — User fills email & password.
   - Trigger: Tap “Sign in”
   |
   v
2. SCR-001 / Loading — Button shows spinner; client disables inputs.
   - Action: POST /auth/login
   |
   v
3. Decision Point:
   +-- Success -> SCR-006 / Loading (role resolution), then redirect to SCR-007 or SCR-008 or SCR-009 / Default
   +-- Invalid Credentials -> SCR-001 / Error (generic "Invalid credentials") and focus first invalid
   +-- Account Locked -> SCR-005 / Default (lockout guidance)

#### Required Interactions
- Submit button shows loading state and is disabled.
- Error message announced via ARIA live region.
- Role redirect skeleton shown during resolution.

### Flow: FL-002 - Forgot Password & Reset
**Flow ID**: FL-002  
**Derived From**: UC-002  
**Personas Covered**: End User  
**Description**: User requests reset, receives link, resets password via token.

#### Flow Sequence
1. Entry: SCR-002 / Default — User submits email.
   |
   v
2. SCR-002 / Loading — Show spinner; POST /auth/forgot-password returns 200 (generic)
   |
   v
3. SCR-004 / Default — "If an account exists, you will receive reset instructions" shown
   - User receives email and navigates to reset link (SCR-003)
   |
   v
4. SCR-003 / Default — User submits new password
   |
   v
5. Decision Point:
   +-- Success -> SCR-004 / Default (confirmation)
   +-- Token expired/invalid -> SCR-003 / Error with re-request CTA

#### Required Interactions
- Non-revealing confirmation message.
- Token validation errors handled generically with re-request option.

### Flow: FL-003 - Account Lockout & Admin Unlock
**Flow ID**: FL-003  
**Derived From**: UC-003  
**Personas Covered**: End User, Administrator  
**Description**: User hits lockout after failed attempts; admin can unlock.

#### Flow Sequence
1. SCR-001 / Default -> repeated failed submits increment failed_attempts
   |
   v
2. SCR-005 / Default — Account locked message with TTL and support link
   |
   v
3. Admin Flow:
   - Entry: SCR-012 / Default — Admin searches user and triggers unlock
   - Confirm Unlock (modal) -> on success admin sees audit entry and user can log in again

#### Required Interactions
- Lockout notification is non-revealing and provides next-step CTAs.
- Admin unlock requires confirm dialog and logs event.

---

## 12. Export Requirements

### JPG Export Settings
| Setting | Value |
|---------|-------|
| Format | JPG |
| Quality | High (85%) |
| Scale - Mobile | 2x |
| Scale - Web | 2x |
| Color Profile | sRGB |

### Export Naming Convention
`<AppName>__<Platform>__<ScreenName>__<State>__v<Version>.jpg`

### Export Manifest
| Screen | State | Platform | Filename |
|--------|-------|----------|----------|
| Login | Default | Mobile | AuthApp__Mobile__Login__Default__v1.jpg |
| Login | Loading | Mobile | AuthApp__Mobile__Login__Loading__v1.jpg |
| Login | Empty | Mobile | AuthApp__Mobile__Login__Empty__v1.jpg |
| Login | Error | Mobile | AuthApp__Mobile__Login__Error__v1.jpg |
| Login | Validation | Mobile | AuthApp__Mobile__Login__Validation__v1.jpg |
| Forgot Password | Default | Mobile | AuthApp__Mobile__ForgotPassword__Default__v1.jpg |
| ... | ... | ... | ... |

### Total Export Count
- Screens (defined): 14
- States per screen: 5
- **Total JPGs**: 14 * 5 = 70

---

## 13. Figma File Structure

```
AuthApp Figma File
+-- 00_Cover
|   +-- Project info, version, stakeholders
+-- 01_Foundations
|   +-- Color tokens (Light + Dark)
|   +-- Typography scale
|   +-- Spacing scale
|   +-- Radius tokens
|   +-- Elevation/shadows
|   +-- Grid definitions
+-- 02_Components
|   +-- C/Actions/[Button, IconButton, Link, FAB]
|   +-- C/Inputs/[TextField, PasswordField, Select, Checkbox, Radio, Toggle]
|   +-- C/Navigation/[Header, Sidebar, Tabs, BottomNav]
|   +-- C/Content/[Card, ListItem, Table, Avatar]
|   +-- C/Feedback/[Modal, Drawer, Toast, Alert, CAPTCHA, Skeleton]
+-- 03_Patterns
|   +-- Auth form pattern
|   +-- Reset flow pattern
|   +-- Error/Empty/Loading patterns
+-- 04_Screens
|   +-- Login/Default
|   +-- Login/Loading
|   +-- Login/Empty
|   +-- Login/Error
|   +-- Login/Validation
|   +-- ... (all other screens with states)
+-- 05_Prototype
|   +-- Flow FL-001: Login Success
|   +-- Flow FL-002: Forgot Password & Reset
|   +-- Flow FL-003: Lockout & Unlock
+-- 06_Handoff
    +-- Token usage rules
    +-- Component guidelines
    +-- Responsive specs
    +-- Edge cases
    +-- Accessibility notes
```

---

## 14. Quality Checklist

### Pre-Export Validation
- [ ] All screens have required states (Default/Loading/Empty/Error/Validation)
- [ ] All components reference design tokens (no hard-coded values)
- [ ] Color contrast meets WCAG AA (>=4.5:1 for text, >=3:1 for UI)
- [ ] Focus states defined for all interactive elements
- [ ] Touch targets >= 44x44px (mobile)
- [ ] Prototype flows wired: FL-001, FL-002, FL-003
- [ ] Naming conventions followed (C/<Category>/<Name>, ScreenName/State)
- [ ] Export manifest complete

### Post-Generation
- [ ] designsystem.md updated with Figma references
- [ ] Export manifest generated and stored in 06_Handoff
- [ ] JPG files exported and named according to convention
- [ ] Handoff documentation (token usage, responsive rules, accessibility notes) completed

---