# Design System

---
post_title: "Authentication Design System"
author1: "Senior Product Designer"
post_slug: "auth-design-system"
microsoft_alias: "designer.alias"
featured_image: ""
categories: ["design-system"]
tags: ["tokens","components","accessibility"]
ai_note: "AI-assisted: token mapping and component templates"
summary: "Design system tokens and component specifications for Authentication feature: color, typography, spacing, radius, elevation, motion, component variants, brand usage, accessibility guidelines and examples."
post_date: "2026-04-07"
---

## 1. Design Tokens

Token hierarchy follows primitive → semantic → component tokens. Use variables in Figma and export to code.

Primitive Colors (Light mode)
- color.blue.500: #2563EB
- color.green.500: #10B981
- color.red.500: #EF4444
- color.yellow.500: #F59E0B
- neutral.900: #0F172A
- neutral.800: #111827
- neutral.700: #374151
- neutral.500: #6B7280
- neutral.300: #D1D5DB
- neutral.100: #F3F4F6
- white: #FFFFFF

Dark mode primitives (mapped)
- neutralDark.900: #000000
- neutralDark.700: #111827
- neutralDark.300: #9CA3AF

Semantic tokens
- color.primary: {color.blue.500} — Primary CTAs, active states
- color.success: {color.green.500}
- color.error: {color.red.500}
- color.warning: {color.yellow.500}
- color.surface: {white}
- color.onSurface: {neutral.900}
- color.muted: {neutral.500}
- color.input.bg: {neutral.100}
- color.input.border: {neutral.300}
- color.focus: #60A5FA (accessible focus ring)

Component tokens (examples)
- button.primary.background: {color.primary}
- button.primary.text: {white}
- input.background: {color.input.bg}
- banner.error.background: #FFF1F2
- banner.error.border: {color.error}

Typography
- Family: System stack for auth flows (to maximize accessibility): "Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial"
  - Note: Product-level decision may substitute brand fonts; system fonts used to ensure legibility in early releases.
- Scale:
  - H1: 28px / 36px line-height / weight 600
  - H2: 22px / 28px / 600
  - H3: 18px / 24px / 600
  - Body: 16px / 24px / 400
  - Small: 14px / 20px / 400
  - Caption: 12px / 16px / 400
- Accessibility: Use 16px base body; large text rules for 18pt+ or 14pt+ bold for 3:1 contrast exceptions.

Spacing
- Base: 8px
- Scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64
- Form spacing:
  - Vertical gap between form controls: 16px
  - Form outer padding (mobile): 24px
  - Card padding: 20px

Radius
- radius.small: 4px (inputs, small buttons)
- radius.medium: 8px (cards, modals)
- radius.large: 16px (dialogs)
- radius.full: 9999px (pills/avatars)

Elevation / Shadows
- elevation.1: 0 1px 2px rgba(16,24,40,0.04)
- elevation.2: 0 2px 6px rgba(16,24,40,0.08)
- elevation.3: 0 8px 20px rgba(16,24,40,0.12)

Motion
- duration.micro: 150ms
- duration.short: 200ms
- duration.medium: 300ms
- duration.long: 400-500ms
- easing.standard: cubic-bezier(0.2, 0.8, 0.2, 1)
- reduced-motion: respects prefers-reduced-motion

Shadow & Border tokens for states
- border.default: 1px solid {neutral.300}
- border.focus: 2px solid {color.focus}
- input.error.border: 1px solid {color.error}

Token usage rules:
- Always reference semantic tokens in components (no hex literals).
- Map light/dark tokens in 01_Foundations and implement theme swap variables.

## 2. Component Specifications

Each component lists variants, states, accessibility, anatomy and usage guidance.

C/Actions/Button
- Anatomy: Label text, optional icon (leading/trailing), spinner for loading.
- Variants: Primary (filled), Secondary (outline), Ghost (transparent), Link (inline)
- Sizes: Small (32px), Medium (40px), Large (48px)
- States: Default, Hover (elevation + color lighten 6%), Focus (2px focus border color.focus), Active (press scale 0.98), Disabled (40% opacity), Loading (spinner replaces label; keep width stable)
- Accessibility:
  - role="button", aria-disabled when disabled
  - Spinner should have aria-hidden="true"; button should provide aria-busy="true" when loading
- Examples:
  - Primary/Medium: background={color.primary}, color={white}, radius=8px

C/Inputs/TextField
- Anatomy: Label, input, helper text, error text
- Variants: Email, Password (with reveal toggle), Search
- Sizes: Medium default (40px)
- States: Default (border: neutral.300), Focus (border focus token + ring), Error (border error + icon), Disabled
- Accessibility:
  - <label for="id">, input id maps, aria-describedby links helper/error
  - Use aria-invalid="true" on error
- PasswordInput:
  - Reveal toggle is a button with aria-pressed state and accessible label "Show password" / "Hide password"

C/Checkbox
- Default & disabled states; focus ring visible
- Hit area minimum 44x44px on mobile

C/Feedback/Banner
- Types: Info (blue), Success (green), Warning (yellow), Error (red)
- Placement: top of page or form-level; should be dismissible when non-critical
- Accessibility:
  - role="status" or role="alert" depending on severity; use aria-live accordingly

C/Feedback/Modal
- Use for Unlock confirmation, session expiry, CAPTCHA overlay
- Behavioral rules:
  - Focus trap while open; restore focus on close
  - Dismiss with ESC
  - Provide accessible heading and close button with aria-label

C/Content/Card
- Use for admin result rows or contextual info
- Variants: default, compact
- Accessibility:
  - Ensure card actions are keyboard accessible and have aria-labels

C/DataTable (Admin Audit Summary)
- Features: sortable columns, filters, pagination, export
- Accessibility:
  - Use semantic table markup (<table>, <thead>, <tbody>) and scope attributes for headers
  - Row actions accessible via keyboard; provide summary for screen readers
- Performance:
  - Use virtualization if events count >1000 (but validate per product constraints)

C/Media/QRCodeContainer
- Must have alt text and aria-describedby pointing to instructions
- Provide manual code in plain text for users who cannot scan QR

C/Feedback/Skeleton
- Shimmer animation (respects reduced motion) used for SCR-013 loading

Naming & Properties:
- Component names: C/<Category>/<Name> (e.g., C/Inputs/TextField)
- Each component includes variant properties: Type, Size, State, Icon

## 3. Brand Guidelines

Logo usage:
- Primary logo: use on header in light/dark variants.
- Clearspace: 2x logo height on all sides.
- Minimum size: 32px height for header use.

Brand colors:
- Primary: color.primary
- Accent: neutral.700 for text, green for success, red for errors.
- Do not use purple gradients on white backgrounds (per ui-ux-design-standards constraint).

Iconography:
- Style: Outline with 2px stroke; fill for status badges only.
- Labels: all icons must have text labels or aria-label.

Illustrations:
- Style: Flat and friendly for empty states; avoid decorative heavy textures on auth surfaces.
- Photography: N/A for authentication screens.

Brand voice:
- Tone: Confident, supportive, non-blaming.
- Error messages: short, helpful, non-accusatory (e.g., "Invalid credentials. Try again or reset your password.")
- Empty states: encouraging, instructive with clear CTA.

Typography recommendations:
- Early-release: system font stack for performance and accessibility.
- Later brand update: introduce production fonts via tokenized family variables.

## 4. Accessibility Standards (WCAG compliance details)

- Target: WCAG 2.2 Level AA compliance for all new/modified UI.
- Contrast:
  - Normal text: >=4.5:1
  - Large text: >=3:1
  - UI components: >=3:1
- Focus:
  - Visible focus indicator for all interactive elements; minimum 2px outline, color.focus token.
- Keyboard:
  - Full keyboard operability for modals, forms, lists, and tables.
  - Use roving tabindex patterns for complex composites only when necessary; otherwise prefer semantic controls.
- Screen Reader:
  - Labels, role attributes, and aria-describedby for error instructions.
  - Use aria-live regions for status and banner updates.
- Forms:
  - Inputs associated with labels, required fields use aria-required.
  - Inline errors include pointer to invalid inputs.
- Images:
  - Alt text for informative images; decorative images marked alt="".
- Motion:
  - Respect prefers-reduced-motion; provide non-animated fallbacks for skeletons and micro-animations.
- Testing checklist to include:
  - Color contrast verification
  - Keyboard-only navigation walkthrough
  - Screen reader read-through (NVDA/VoiceOver)
  - Touch target audit for mobile
  - Automated scans (Axe) and manual validation

## 5. Usage Guidelines and Examples

Example: Login Form (component composition)
- Compose: Page container -> C/Content/Card (form card) -> C/Inputs/TextField (email) -> C/Inputs/PasswordInput -> C/Actions/Button (Primary) -> C/Actions/Button (Link) for Forgot
- Behavior:
  - Primary button disabled until inputs valid.
  - On click: button shows spinner and aria-busy set to true.
  - On server error: show C/Feedback/Banner error with aria-live.

Example: Password Reset Flow
- Flow composition:
  - SCR-002: Email input -> submit -> show SCR-003 confirmation banner (info)
  - SCR-004: Reset page uses PasswordInput with strength meter (aria-live region) and confirm field.

Admin Unlock example
- Table row selection -> open Confirm Unlock Modal (C/Feedback/Modal)
- Confirm modal requires audit note (TextArea) — validate non-empty -> Unlock API call -> toast success

Error copy rules (security)
- Authentication failures: "Invalid credentials" — do NOT state whether email exists.
- Forgot password response: "If an account exists, a reset link has been sent."
- Account locked: provide next steps and contact support link but not internal lock reasons.

Dark mode support
- Provide 1:1 parity tokens for dark mode; verify contrast in dark backgrounds.
- Toggle or auto-detect via OS preference.

Token mapping examples (YAML snippet)
```yaml
colors:
  primary:
    value: "{color.blue.500}"
    usage: "Primary CTAs"
typography:
  heading1:
    family: "Inter"
    size: "28px"
    weight: "600"
spacing:
  base: "8px"
```

(Include actual token files under 01_Foundations in Figma project and export for dev.)

Governance & Contribution
- Single source of truth: designsystem.md and 01_Foundations are canonical.
- Change requests: file an issue with rationale; design owner approves token changes.
- Versioning: increment token version when primitives change; include migration notes.

Examples and Patterns
- Error banner + inline error pattern approved for auth screens and must be reused.
- MFA QR pattern: provide QR + manual code + accessible copy/download of recovery codes.
- CAPTCHA overlay: use modal variant; ensure keyboard trap and accessible challenge fallback.

End of Design System.