## Figma Design Specification — AuthService

## 1. Figma Specification
**Platform**: Responsive (Web primary, Mobile secondary)

---

## 2. Source References

### Primary Source
| Document | Path | Purpose |
|----------|------|---------|
| Requirements | `.propel/context/docs/spec.md` | Personas, use cases, epics with UI impact flags; source of truth for auth features and flows |

### Optional Sources
| Document | Path | Purpose |
|----------|------|---------|
| Wireframes | `.propel/context/wireframes/` | Early layout drafts used to verify content structure |
| Design Assets | `.propel/context/Design/` | Visual references for branding, icons, and illustration styles |

### Related Documents
| Document | Path | Purpose |
|----------|------|---------|
| Design System | `.propel/context/docs/designsystem.md` | Canonical tokens, component specs and naming conventions |
| Security Guidelines | `.propel/context/docs/security.md` | Notes on messaging, non-revealing error text, lockout guidance |

---

## 3. UX Requirements

### UXR Requirements Table
| UXR-ID | Category | Requirement | Acceptance Criteria | Screens Affected |
|--------|----------|-------------|---------------------|------------------|
| UXR-001 | Project-wide | Every use case in spec.md must map to a screen or a prototype flow | All UC-001..UC-008 referenced by SCR-001..SCR-012; prototype covers success/error paths | All SCR-XXX |
| UXR-002 | Project-wide | Core tasks (login, forgot password, reset) reachable within 3 interactions from landing | Prototype shows ≤3 clicks/taps from entry to task completion in 90% of test runs | SCR-001, SCR-002, SCR-004 |
| UXR-101 | Usability | Primary actions on auth screens must be visually prominent and first in tab order | Sign-in, Submit, Verify CTAs are highest visual weight and first tab stop; usability test success ≥90% | SCR-001, SCR-004, SCR-007 |
| UXR-102 | Usability | Provide clear fallback CTAs (Forgot password, Contact support) adjacent to primary actions | Links/buttons present, accessible, and keyboard-focusable; documented in handoff | SCR-001, SCR-006 |
| UXR-103 | Usability | Provide progressive disclosure for MFA enrollment and recovery codes | MFA enrollment includes stepwise UI: QR -> verify -> show recovery codes; confirmation state included | SCR-008 |
| UXR-201 | Accessibility | All screens must meet WCAG 2.2 AA (text >=4.5:1, UI >=3:1) | Color contrast checks pass; focus states visible; verified in 06_Handoff | All SCR-XXX |
| UXR-202 | Accessibility | Error and status messages must be announced to screen readers and accessible via aria-live | Error messages use polite/assertive live regions and associate with inputs via aria-describedby | SCR-001, SCR-002, SCR-004, SCR-007 |
| UXR-301 | Responsiveness | Layout adapts for Mobile (320–414), Tablet (768), Desktop (1024+) breakpoints | Frames defined for mobile/tablet/web; Auto Layout behaviors documented; tests pass at listed widths | All SCR-XXX |
| UXR-401 | Visual Design | Use design system tokens for all visual styles; no hard-coded colors/spacing | All components reference color.primary, spacing.base, typography.h1 etc; design tokens documented | 01_Foundations, All SCR-XXX |
| UXR-402 | Visual Design | Focus outlines meet contrast and thickness requirements and are present for all interactive elements | Focus outline ≥3:1 against background, 2px minimum offset; documented on components | All interactive components |
| UXR-501 | Interaction | Provide immediate visual feedback (spinner or skeleton) for actions that take >200ms | Loading states implemented for submit/verify actions with spinner or skeleton; spinner shown within 200ms | SCR-001, SCR-002, SCR-004, SCR-007 |
| UXR-502 | Interaction | Disable primary submit while async request is pending to prevent double-submit | Submit button disabled and shows loading variant; duplicate requests prevented | SCR-001, SCR-002, SCR-004 |
| UXR-601 | Error Handling | Authentication errors must be non-revealing and consistent to prevent account enumeration | Error copy: "Invalid credentials" or "Account is locked"; server returns 401/423; UI shows same messages for missing accounts | SCR-001, SCR-002, SCR-006 |
| UXR-602 | Error Handling | Inline validation must highlight fields and provide actionable guidance | Field-level messages shown under inputs; inputs get aria-invalid and aria-describedby | SCR-001, SCR-004 |
| UXR-603 | Error Handling | Lockout messaging must provide guidance and support options without exposing sensitive details | Lock screen/banners show lock reason, ETA and support contact; admins see unlock controls | SCR-006, SCR-011 |
| UXR-701 | Interaction | CAPTCHA escalation UI must be accessible and have alternative keyboard flows | CAPTCHA modal/dedicated challenge supports keyboard navigation and has audio/accessible alternative where supported | SCR-001, SCR-011 |
| UXR-801 | Interaction | MFA flows must show recovery-code generation and require user confirmation to store codes | Recovery codes are displayed once with "Download/Copy" CTA and confirmation checkbox before completing enrollment | SCR-008 |

### UXR Categories
- Usability (1XX): Navigation, discoverability, efficiency
- Accessibility (2XX): WCAG 2.2 AA compliance, ARIA/live regions
- Responsiveness (3XX): Breakpoints and auto layout behaviors
- Visual Design (4XX): Token adherence and visual consistency
- Interaction (5XX): Loading feedback, disabling, micro-interactions
- Error Handling (6XX): Non-revealing errors, lockout messaging

### UXR Derivation Logic
- Usability: Derived from UC-001..UC-008 success paths and success criteria in FRs
- Accessibility: Derived from WCAG 2.2 AA and web-accessibility-standards in related guidelines
- Responsiveness: Derived from figma-design-standards frame sizes and required platform support
- Visual: Derived from designsystem.md token requirements and branding constraints
- Interaction: Derived from flow complexity and security timing (e.g., token verification)
- Error Handling: Derived from FR-002, FR-006, FR-007 and secure messaging constraints

### UXR Numbering Convention
- UXR-001 to UXR-099: Project-wide requirements
- UXR-1XX: Usability requirements
- UXR-2XX: Accessibility requirements
- UXR-3XX: Responsiveness requirements
- UXR-4XX: Visual design requirements
- UXR-5XX: Interaction requirements
- UXR-6XX: Error handling requirements

---

## 4. Personas Summary

| Persona | Role | Primary Goals | Key Screens |
|---------|------|---------------|-------------|
| End User | Customer / Employee | Authenticate quickly, recover password, reach role dashboard | Login, ForgotPassword, ResetPassword, MFAChallenge |
| Privileged User | Admin/Privileged employee | Enroll MFA, use elevated features securely | MFA Enrollment, MFA Challenge, Login |
| Support Operator | Admin/Support staff | Unlock accounts, view audit context, help users recover | Admin User Management, Unlock Modal, Audit Event Detail |
| Security Analyst | Compliance/Security team | Verify audit trails and suspicious patterns | Audit Event Detail, Admin User Management |

---

## 5. Information Architecture

### Site Map
[AuthService]
+-- Auth
|   +-- Login
|   +-- Forgot Password
|   +-- Password Reset
|   +-- MFA Enrollment
|   +-- MFA Challenge
+-- Account
|   +-- Account Locked (status page)
|   +-- Logout
+-- Admin
|   +-- User Management
|   +-- Audit Events

### Navigation Patterns
| Pattern | Type | Platform Behavior |
|---------|------|-------------------|
| Primary Auth Nav | Header (or focused center card) | Web: centered auth card with top-left logo; Mobile: full-bleed card with back navigation |
| Secondary Actions | Inline links under primary CTA | All: "Forgot password" and "Contact support" shown as secondary links |
| Utility Nav | Top-right (when authenticated) | Desktop: profile menu with logout; Mobile: hamburger with account actions |
| Admin Nav | Admin Sidebar | Desktop: persistent left sidebar; Mobile: collapsible drawer for admin pages |

---

## 6. Screen Inventory

### Screen List
| Screen ID | Screen Name | Derived From | Personas Covered | Priority | States Required |
|-----------|-------------|--------------|------------------|----------|-----------------|
| SCR-001 | Login | UC-001, UC-002, UC-003 | End User, Privileged User | P0 | Login/Default, Login/Loading, Login/Empty, Login/Error, Login/Validation |
| SCR-002 | Forgot Password (Request) | UC-005 | End User | P0 | ForgotPassword/Default, /Loading, /Empty, /Error, /Validation |
| SCR-003 | Forgot Password Confirmation | UC-005 | End User | P0 | ForgotPasswordConfirmation/Default, /Loading, /Empty, /Error, /Validation |
| SCR-004 | Password Reset (Set New Password) | UC-006 | End User | P0 | ResetPassword/Default, /Loading, /Empty, /Error, /Validation |
| SCR-005 | Token Invalid / Expired | UC-006 | End User | P0 | TokenInvalid/Default, /Loading, /Empty, /Error, /Validation |
| SCR-006 | Account Locked | UC-004 | End User, Support Operator | P0 | AccountLocked/Default, /Loading, /Empty, /Error, /Validation |
| SCR-007 | MFA Challenge (TOTP) | FR-012 | Privileged User | P1 | MFAChallenge/Default, /Loading, /Empty, /Error, /Validation |
| SCR-008 | MFA Enrollment (Provisioning) | FR-012 | Privileged User | P1 | MFAEnroll/Default, /Loading, /Empty, /Error, /Validation |
| SCR-009 | Role Selection (Ambiguous) | UC-008 | End User | P1 | RoleSelect/Default, /Loading, /Empty, /Error, /Validation |
| SCR-010 | Logout Confirmation | UC-007 | End User | P1 | LogoutConfirm/Default, /Loading, /Empty, /Error, /Validation |
| SCR-011 | Admin — User Management (Unlock) | UC-004 (admin) | Support Operator, Admin | P1 | AdminUserMgmt/Default, /Loading, /Empty, /Error, /Validation |
| SCR-012 | Admin — Audit Event Detail | FR-010 | Security Analyst, Admin | P2 | AuditEvents/Default, /Loading, /Empty, /Error, /Validation |

### Priority Legend
- P0: Critical path (must-have for MVP)
- P1: Core functionality (high priority)
- P2: Important features (medium priority)
- P3: Nice-to-have (low priority)

### Screen-to-Persona Coverage Matrix
| Screen | End User | Privileged User | Support Operator | Security Analyst | Notes |
|--------|---------:|----------------:|-----------------:|-----------------:|-------|
| Login | Primary | Primary | Secondary | - | Entry point for all users |
| Forgot Password | Primary | Primary | - | - | Self-service recovery |
| Reset Password | Primary | Primary | - | - | Token-based flow |
| Account Locked | Primary | - | Primary | - | Supportable by admin |
| MFA Enroll | - | Primary | - | - | Privileged users only |
| Admin User Management | - | Secondary | Primary | Secondary | Admin consoles not part of public auth UI |
| Audit Events | - | - | Secondary | Primary | Compliance-focused screens |

### Modal/Overlay Inventory
| Name | Type | Trigger | Parent Screen(s) | Priority |
|------|------|---------|-----------------|----------|
| Login Error Toast | Toast | Failed login | Login | P0 |
| Confirm Logout | Modal | Click "Logout" | Nav/Authenticated pages | P1 |
| Unlock Confirmation | Dialog | Admin unlock action | Admin User Management | P0 |
| CAPTCHA Challenge | Modal / Inline | Rate-limit/CAPTCHA escalation | Login | P1 |
| Recovery Codes Download | Modal | MFA enrollment complete | MFA Enrollment | P1 |

---

## 7. Content & Tone

### Voice & Tone
- Overall Tone: Professional and supportive — secure, concise, and non-blaming.
- Error Messages: Clear, actionable, non-revealing. Example: "Invalid credentials. Please try again or use Forgot password."
- Empty States: Encouraging and instructive with clear CTAs (e.g., "No recent activity. Go to Dashboard.")
- Success Messages: Brief and actionable (e.g., "Password changed. Sign in with your new password.")

### Content Guidelines
- Headings: Sentence case for clarity (e.g., "Sign in to your account")
- CTAs: Action-oriented, specific verbs (e.g., "Sign in", "Send reset email", "Verify code")
- Labels: Short, descriptive (e.g., "Email address", "Password", "Remember this device")
- Placeholder Text: Helpful examples (e.g., "name@example.com"); avoid lorem ipsum in final designs

---

## 8. Data & Edge Cases

### Data Scenarios
| Scenario | Description | Handling |
|----------|-------------|----------|
| No Data | User has no account history or items | Show empty state with instructions or CTA (e.g., "Create account" if applicable) |
| First Use | New user with no history | Provide contextual helper text and progressive disclosure (MFA opt-in shown but optional) |
| Large Data | Admin audit results >1000 records | Provide pagination, filters, and export; use virtualization for performance |
| Slow Connection | Network >3s for auth calls | Show skeletons or spinner and informative message "Checking credentials..." |
| Offline | No network available | Show offline state and retry option; prevent sensitive actions until online |

### Edge Cases
| Case | Screen(s) Affected | Solution |
|------|-------------------|----------|
| Long email/username strings | Login, Admin lists | Truncate with ellipses, provide full value on hover/clipboard copy |
| Missing avatar/image | Admin lists, user profiles | Fallback avatar with initials SVG |
| Password policy violations | ResetPassword | Inline validation with policy checklist and strength meter |
| Token reuse or replay | ResetPassword | Server-side single-use enforcement; UI shows token invalid/expired screen |
| Concurrent unlock attempts | Admin User Management | Optimistic UI with server confirmation and conflict message if already unlocked |

---

## 9. Branding & Visual Direction
*See `.propel/context/docs/designsystem.md` for tokens and full brand assets.*

- Logo: Primary wordmark + icon lock symbol; path: `.propel/context/Design/logo/logo_primary.svg` [TO BE UPDATED]
- Icon Style: Filled glyphs with 2px stroke where applicable, consistent 16/24/32px grid
- Illustration Style: Minimal flat illustrations for empty states; single-color line illustrations for error states
- Photography Style: Not used on auth screens; if used in admin dashboards, prefer high-clarity, business-focused imagery

---

## 10. Component Specifications

### Component Library Reference
**Source**: `.propel/context/docs/designsystem.md` (Component Specifications section)

### Required Components per Screen
| Screen ID | Components Required | Notes |
|-----------|---------------------|-------|
| SCR-001 | C/Inputs/TextField (email), C/Inputs/TextField (password), C/Actions/Button (Primary), Link | Login form; show forgot-password link |
| SCR-002 | C/Inputs/TextField (email), C/Actions/Button (Primary) | Generic confirmation flow on submit |
| SCR-004 | C/Inputs/TextField (new password), C/Inputs/TextField (confirm password), PasswordPolicyChecklist, C/Actions/Button (Primary) | Password strength & policy inline |
| SCR-007 | C/Inputs/TextField (OTP), C/Actions/Button (Primary), C/Actions/Button (Secondary) | Retry/resend and "Use recovery code" options |
| SCR-008 | C/Image/QRCode, C/Inputs/TextField (verification), RecoveryCodesPanel, C/Actions/Button (Primary) | QR SVG must scale and be accessible |
| SCR-011 | C/Table/UserList, C/Actions/Button (Unlock), AdminToolbar | Bulk actions and search |

### Component Summary
| Category | Components | Variants |
|----------|------------|----------|
| Actions | C/Actions/Button, C/Actions/IconButton, C/Actions/Link | Primary/Secondary/Tertiary, Sizes S/M/L, States |
| Inputs | C/Inputs/TextField, C/Inputs/PasswordField, C/Inputs/Checkbox | States: Default/Focus/Error/Disabled/Loading |
| Navigation | C/Navigation/Header, C/Navigation/Sidebar | Desktop & mobile variants |
| Content | C/Content/Card, C/Content/Table, C/Content/ListItem | Row/Expanded/Compact variants |
| Feedback | C/Feedback/Modal, C/Feedback/Toast, C/Feedback/Skeleton | Variants per use case |

### Component Constraints
- Use only components defined in designsystem.md (C/<Category>/<Name>)
- All components must support states: Default, Hover, Focus, Active, Disabled, Loading
- Auto Layout must be used for components with documented resize behavior (Hug/Fill)
- Touch targets >=44x44px on mobile for actionable elements
- All focusable components must expose focus-visible styles and ARIA attributes where appropriate

---

## 11. Prototype Flows

### Flow: FL-001 - User Login (Happy Path)
**Flow ID**: FL-001  
**Derived From**: UC-001  
**Personas Covered**: End User, Privileged User  
**Description**: Enter credentials, authenticate, receive token, redirect to role-specific dashboard.  

#### Flow Sequence
1. Entry: Login/Default  
   - Trigger: User enters email and password and clicks "Sign in"  
   - Action: Client-side validation passes  
   |
   v
2. Login/Loading  
   - Action: POST /auth/login; show button loading state + skeleton for redirect processing  
   |
   v
3. Decision Point: Auth success?  
   +-- Success -> Redirect to role-specific Dashboard (external to auth file)  
   +-- Failure -> Login/Error (show "Invalid credentials") and increment failed_attempts  

#### Required Interactions
- Submit button disabled while request pending
- Error message announced via aria-live

---

### Flow: FL-002 - Forgot Password Request & Confirmation
**Flow ID**: FL-002  
**Derived From**: UC-005  
**Personas Covered**: End User  

#### Flow Sequence
1. Entry: Login/Default -> click "Forgot password"  
2. ForgotPassword/Default -> Enter email -> click "Send reset email"  
   - Show validation for empty/invalid email  
3. ForgotPassword/Loading -> show spinner on submit  
4. ForgotPasswordConfirmation/Default -> show generic message "If an account exists..." with CTA "Return to sign in"  
Decision: Email delivery success not disclosed to user; Admin logs delivery status

#### Required Interactions
- Generic response to avoid account enumeration
- Email delivery logged in backend (not exposed)

---

### Flow: FL-003 - Reset Password via Token
**Flow ID**: FL-003  
**Derived From**: UC-006  
**Personas Covered**: End User  

#### Flow Sequence
1. Entry: ResetPassword/Default via link with token  
2. ResetPassword/Validation -> Client validates password policy in real time  
3. ResetPassword/Loading -> POST /auth/reset-password (token, new password)  
4. Decision: Token valid?  
   +-- Yes -> show success message and link to Login/Default  
   +-- No -> TokenInvalid/Default with CTA "Request a new reset"  

#### Required Interactions
- Password policy checklist updates live
- Token validation shows explicit invalid/expired state

---

### Flow: FL-004 - Account Lockout and Admin Unlock
**Flow ID**: FL-004  
**Derived From**: UC-004  
**Personas Covered**: End User, Support Operator  

#### Flow Sequence
1. Entry: Repeated login failures -> Login/Error shows lock threshold reached  
2. AccountLocked/Default visible on next login attempt (or banner on Login/Default)  
3. Admin: AdminUserMgmt/Default -> select user -> Unlock Confirmation dialog -> API unlock -> success toast  
Decision: Auto-unlock after configured duration vs manual unlock

#### Required Interactions
- Lock banner with ETA and support contact
- Admin unlock confirmation with audit link

---

### Flow: FL-005 - MFA Enrollment & Challenge
**Flow ID**: FL-005  
**Derived From**: FR-012 / UC-001 (privileged)  
**Personas Covered**: Privileged User  

#### Flow Sequence
1. Entry: MFAEnroll/Default -> show QR and secret key (masked)  
2. User scans QR with authenticator -> enters code in Verify input -> click "Verify"  
3. MFAEnroll/Loading -> verify TOTP -> Decision: Verified?  
   +-- Yes -> show RecoveryCodesPanel (Download/Copy) and completion confirmation  
   +-- No -> MFAEnroll/Error with guidance  
4. Subsequent login triggers MFAChallenge/Default to enter TOTP

#### Required Interactions
- Recovery codes visible once with "I stored these codes" confirmation before finish
- QR image has alt text and accessible fallback secret

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
`AuthService__<Platform>__<ScreenName>__<State>__v<Version>.jpg`

### Export Manifest
| Screen | State | Platform | Filename |
|--------|-------|----------|----------|
| Login | Default | Mobile | AuthService__Mobile__Login__Default__v1.jpg |
| Login | Loading | Mobile | AuthService__Mobile__Login__Loading__v1.jpg |
| Login | Error | Mobile | AuthService__Mobile__Login__Error__v1.jpg |
| ForgotPassword | Default | Web | AuthService__Web__ForgotPassword__Default__v1.jpg |
| ResetPassword | Default | Web | AuthService__Web__ResetPassword__Default__v1.jpg |
| MFAEnroll | Default | Web | AuthService__Web__MFAEnroll__Default__v1.jpg |
| AdminUserMgmt | Default | Web | AuthService__Web__AdminUserMgmt__Default__v1.jpg |

### Total Export Count
- Screens: 12
- States per screen: 5
- Total JPGs: 12 * 5 = 60

---

## 13. Figma File Structure

```
AuthService Figma File
+-- 00_Cover
|   +-- Project info, version, stakeholders
+-- 01_Foundations
|   +-- color.primary, color.semantic, neutral 50-900 (Light + Dark)
|   +-- typography scale (H1→Caption)
|   +-- spacing tokens (4,8,12,16,20,24,32,40)
|   +-- radius tokens
|   +-- elevation/shadows
|   +-- grid definitions for Mobile/Tablet/Web
+-- 02_Components
|   +-- C/Actions/Button (Primary/Secondary/Tertiary)
|   +-- C/Inputs/TextField, PasswordField, Checkbox
|   +-- C/Navigation/Header, Sidebar
|   +-- C/Content/Card, Table, ListItem
|   +-- C/Feedback/Modal, Toast, Skeleton
+-- 03_Patterns
|   +-- Auth form pattern (Login, Forgot, Reset)
|   +-- MFA pattern (QR, verify, recovery codes)
|   +-- Error/Empty/Loading patterns
+-- 04_Screens
|   +-- Login/Default, Login/Loading, Login/Empty, Login/Error, Login/Validation
|   +-- ForgotPassword/Default, ... (all screens and states)
+-- 05_Prototype
|   +-- Flow FL-001: Login
|   +-- Flow FL-002: Forgot Password
|   +-- Flow FL-005: MFA Enrollment and Challenge
+-- 06_Handoff
    +-- Token usage rules
    +-- Component guidelines and code mappings
    +-- Responsive specs and breakpoints
    +-- Edge cases and accessibility notes
```

---

## 14. Quality Checklist

### Pre-Export Validation
- [ ] All screens have required states (Default/Loading/Empty/Error/Validation)
- [ ] All components reference design tokens (no hard-coded values)
- [ ] Color contrast meets WCAG AA (>=4.5:1 text, >=3:1 UI)
- [ ] Focus states defined for all interactive elements
- [ ] Touch targets >= 44x44px (mobile)
- [ ] Prototype flows wired and functional (FL-001 .. FL-005)
- [ ] Naming conventions for components and screens followed
- [ ] Export manifest complete

### Post-Generation
- [ ] designsystem.md updated with Figma references
- [ ] Export manifest generated and files exported per naming convention
- [ ] Handoff documentation includes token usage rules and code mappings
- [ ] Accessibility annotations included: focus order, ARIA suggestions, heading hierarchy

---

Notes & Next Steps:
- Add Figma node links and replace "node-id=TBD" placeholders in 02_Components and 04_Screens once frames are created.
- Ensure admin console screens (SCR-011, SCR-012) include data privacy masking and audit-link affordances.
- Schedule accessibility review pass with Accessibility Insights and include findings in 06_Handoff.