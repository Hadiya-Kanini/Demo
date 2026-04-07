# Design System

Purpose
This design system defines the tokens, components, patterns, accessibility rules, and usage guidance used by the Authentication UI described in the Figma Design Specification. It is intended for designers and engineers to implement consistent, accessible, and maintainable UI.

## 1. Design Tokens

Token Principles
- Hierarchy: Primitive → Semantic → Component tokens.
- All tokens defined as named styles in Figma and exported to code variables (e.g., CSS vars).
- Light & Dark variants must be present; dark parity required.

Color Tokens (selected)
- Primitive
  - color.blue.500: #0A66C2
  - neutral.900: #111827
  - neutral.700: #374151
  - neutral.300: #D1D5DB
  - neutral.100: #F3F4F6
  - success.500: #16A34A
  - warning.500: #F59E0B
  - danger.500: #DC2626
- Semantic
  - color.primary: {color.blue.500}
  - color.text.primary: {neutral.900}
  - color.text.onPrimary: #FFFFFF
  - color.surface: {neutral.100}
  - color.surface.alt: {neutral.300}
  - color.border: {neutral.300}
  - color.success: {success.500}
  - color.warning: {warning.500}
  - color.error: {danger.500}
- Component
  - button.primary.background: {color.primary}
  - button.primary.text: {color.text.onPrimary}
  - input.background: #FFFFFF (light) / #1F2937 (dark)
  - focus.outline: color.blue.500 (with opacity for shadow)

Light & Dark mapping
- Provide counterpart tokens for dark mode (e.g., color.text.primary → neutral.100 in dark mode).
- All contrast must meet WCAG AA.

Typography Tokens
- Base font family: "Inter UI" is allowed; product rule (ui-ux-design-standards) suggested distinct fonts — however authentication flows require legibility and broad platform availability. Use system + accessible fonts:
  - font.family.primary: "IBM Plex Sans", system-fallbacks
- Scale
  - H1: 28px / 36px line-height / 600
  - H2: 22px / 28px / 600
  - H3: 18px / 24px / 600
  - Body1: 16px / 24px / 400
  - Body2: 14px / 20px / 400
  - Caption: 12px / 16px / 400
- Weights: 400 (Regular), 600 (SemiBold), 700 (Bold)
- Accessibility: ensure large text scale corresponds to 18pt+ or 14pt bold for 3:1 contrast checks.

Spacing Tokens
- base: 8px
- scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64
- Use tokens for padding/margins on components: e.g., card.padding = 24px (3x base).

Radius Tokens
- radius.small: 4px
- radius.medium: 8px
- radius.large: 16px
- radius.pill: 9999px

Elevation / Shadows
- elevation.01: 0 1px 2px rgba(16,24,40,0.04)
- elevation.02: 0 4px 8px rgba(16,24,40,0.06)
- elevation.03: 0 8px 24px rgba(16,24,40,0.08)

Motion Tokens
- motion.duration.fast: 150ms
- motion.duration.medium: 200ms
- motion.duration.slow: 300ms
- motion.easing: cubic-bezier(0.22, 0.12, 0.2, 1)
- Motion reduction: provide reduced-motion boolean switch.

Token YAML Example (for dev integration)
```yaml
colors:
  primary:
    value: "#0A66C2"
    usage: "Primary CTAs, active states"
typography:
  heading1:
    family: "IBM Plex Sans"
    size: "28px"
    weight: "600"
spacing:
  base: "8px"
radius:
  small: "4px"
motion:
  duration:
    fast: "150ms"
```

---

## 2. Component Specifications

Overview
- All components must implement variant properties: Type, Size, State, Icon.
- Naming convention: C/<Category>/<Name>.

Actions — Button (C/Actions/Button)
- Variants: Primary, Secondary, Ghost.
- Sizes: Small (32px height), Medium (40px), Large (48px).
- States: Default, Hover, Focus, Active, Disabled, Loading.
- Accessibility:
  - Role: <button>
  - Focus: 2px outline color.focus with 2px offset; visible contrast.
  - Disabled: aria-disabled="true" and grayed styling (40% opacity).
- Behavior:
  - Loading: spinner left of label, label text changes to "Signing in..." optionally.
  - Keyboard: Enter activates when focused; Space triggers click.

Inputs — TextField (C/Inputs/TextField)
- Variants: standard, password (visibility toggle), email, with prefix/suffix.
- States: Default, Focus, Error, Disabled, Readonly.
- Sizing: Height = 40px (M), 48px (L).
- Accessibility:
  - Use <label for="id"> and input id mapping in code.
  - Error messages are linked via aria-describedby.
  - aria-required for required fields.
- Error Treatment:
  - Border color uses color.error and an error icon (combined with text).
  - Do not reveal authentication-specific information in errors.

Password Component (C/Inputs/PasswordField)
- Contains trailing icon toggle (eye).
- Strength meter: 3 levels; visually and textually represented (aria-live updates).

Checkbox / Toggle / Radio
- Touch target min 44x44px.
- Keyboard focusable; aria-checked / aria-pressed as appropriate.

Feedback — Banner (C/Feedback/Banner)
- Types: Info, Success, Error, Warning.
- Placement: Top of card; dismissible.
- Accessibility: role="status" for informative, role="alert" for errors (assertive).

Modal (C/Feedback/Modal)
- Variants: Confirm (destructive), Informational.
- Behavior: Focus trap; Escape closes; initial focus on first actionable control.
- Animation: Fade + scale (200ms) with reduced-motion alternative.

Skeleton (C/Feedback/Skeleton)
- Variants: Card, TableRow, InputRow.
- Use shimmer only when not in reduced-motion; skeleton color token neutral.300.

CAPTCHA Component (C/Feedback/CAPTCHA)
- Integration wrapper around provider widget.
- Fallback accessible description and a "can't complete?" support flow.
- Marked as focusable and announced via aria-describedby.

Table (C/Content/Table)
- Headings use <th scope="col">.
- Left-align text, right-align numeric values.
- Pagination & filters available on admin audit screens.
- Rows support keyboard selection and row actions.

MFA Components (C/Security/MFAEnrollment & C/Security/MFAVerify)
- Enrollment:
  - QR image container with alt text "QR code for authenticator app".
  - Secret string in monospace with copy button.
  - Recovery codes list with copy & download.
- Verify:
  - Numeric input (6 digits). Allow paste and numeric keypad on mobile.
  - Rate-limit errors map into account lockout logic.

Icons & Imagery
- Icon set: outline style consistent across UI.
- Illustrations: flat, minimal (for empty states) with muted palette matching brand.

Component Documentation (what to include)
- Purpose, variants, token references, accessibility notes, states, spacing rules, and example usages.

---

## 3. Brand Guidelines

Logo Usage
- Provide full-color and monochrome assets.
- Clear space: minimum 16px around logo (2x base token).
- Do not stretch, recolor, or rotate the logo.
- Use monochrome white on dark backgrounds and full-color on light surfaces.

Brand Colors & Roles
- Primary: color.primary used for CTAs and links.
- Supportive semantic colors: success, warning, error, info.
- Neutral palette: used for surfaces, borders, and text.

Tone of Voice
- Professional, helpful, and non-blaming.
- Error messages: concise, actionable, and privacy-preserving. Example: "Invalid credentials" vs "Email not found".
- Empty states: encouraging and instructive; include clear CTA.

Illustration & Photography
- Illustration style: flat, minimal, geometric shapes, muted accents.
- Photography: professional, contextual images for marketing screens only (not used in auth flows).

Iconography
- Use line-based icons (24px). Provide accessible label strings for each icon usage.

---

## 4. Accessibility Standards (WCAG Compliance Details)

WCAG 2.2 Level AA (minimum)
- Text contrast:
  - Normal text: ≥ 4.5:1
  - Large text (18pt+ or 14pt bold): ≥ 3:1
- UI component contrast: ≥ 3:1 where perceptible.
- Keyboard:
  - All interactive components keyboard navigable.
  - Logical tab order; skip link to main content on pages with header.
- Forms:
  - Labels programmatically associated; error messages linked with aria-describedby.
  - Required fields use aria-required and visual mark.
- Live Regions:
  - Use aria-live="assertive" for important errors; aria-live="polite" for non-critical notifications.
- Motion:
  - Honor prefers-reduced-motion. Provide non-animated alternatives and avoid motion-triggered dizziness.
- Screen Reader:
  - Use semantic HTML equivalents for Figma handoff suggestions (button, nav, main, form).
- Testing:
  - Automated (Axe, Accessibility Insights) and manual screen reader (VoiceOver & NVDA) tests before release.
- Localization:
  - Allow for text expansion; avoid fixed-size containers that truncate localized content.

Accessibility Implementation Guide
- For each component in 02_Components, include:
  - ARIA roles/props
  - Keyboard interactions
  - Announcements & live region behavior
  - Focus management strategy

---

## 5. Usage Guidelines and Examples

Token Usage Rules
- Use semantic tokens (color.primary) in components rather than primitives.
- Example:
  - Button background: button.primary.background (semantic) → color.primary
  - Input border error: input.border.error → color.error

Component Usage Examples (concise)
- Login Button (Primary, Medium, Default)
  - Component: C/Actions/Button
  - Props: type=Primary, size=M, state=Default
  - Token refs: button.primary.background, typography.body1, radius.medium
  - Accessibility: aria-label="Sign in"

- Password Field with Strength Meter
  - Component: C/Inputs/PasswordField + C/Forms/PasswordStrength
  - Behavior: show strength only on new passwords (Reset flow) per UXR-103.

- Admin Audit Table
  - Component: C/Content/Table
  - Behavior: sorted by default on timestamp desc, filters on left, pagination below.
  - Accessibility: header cells use scope; row actions accessible via keyboard and labelled.

Dark Mode & High Contrast
- Provide token mappings for dark mode versions of each color token.
- High contrast mode: provide alternate tokens with higher saturation & contrast.

Do's and Don'ts
- DO use tokens for spacing/typography/colors.
- DO provide accessible names for icons and interactive elements.
- DO respect reduced-motion preferences.
- DON'T expose secrets or tokens in UI or logs.
- DON'T indicate account existence in error messages.

Examples of Error Messaging (Tone & Safety)
- Authentication failure: "Invalid credentials" (non-revealing).
- Password reset request confirmation: "If an account exists, a reset email has been sent."
- Account locked: "Your account is locked until [time]. Contact support for help."

Versioning & Governance
- Token changes must be versioned. Update 06_Handoff with changelog.
- Component modifications require design review and accessibility sign-off.
- Design System owner: Product Design lead; Engineering owner: Frontend Lead.

Handoff & Implementation Notes
- Provide code-name mapping for each Figma component in 06_Handoff (node IDs → component paths).
- Provide example CSS variables (prefixed with --auth-).
- Provide example props mapping: <Button type="primary" size="M" loading={true} disabled={false} />

Maintenance & Contributions
- Changes to tokens or components require documented justification and release notes.
- Encourage reuse; no ad-hoc components without approval.

Closing Notes
This design system is tailored to the auth/product security context: it emphasizes clarity, minimalism, accessibility, and auditable patterns. All assets, tokens, and components must be present in Figma under the 6-page structure and linked to implementation artifacts in 06_Handoff.

End of Design System.