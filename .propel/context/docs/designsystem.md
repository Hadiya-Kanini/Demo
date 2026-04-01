# Design System

Purpose
- Canonical set of tokens, components and usage rules to be used across the Figma file and implementation teams for Authentication & Admin features. Single source of truth for colors, typography, spacing, radii, elevation and motion.

---

## 1. Design Tokens

Primitive & Semantic Tokens (YAML excerpt)
```yaml
colors:
  brand:
    500: "#0A6CFF"
  semantic:
    success: "#16A34A"
    warning: "#F59E0B"
    error: "#DC2626"
    info: "#3B82F6"
  neutral:
    900: "#0B1220"
    700: "#223247"
    500: "#6B7280"
    300: "#E5E7EB"
    100: "#F9FAFB"

semantic_tokens:
  color.primary: "{colors.brand.500}"
  color.background: "{colors.neutral.100}"
  color.surface: "#FFFFFF"
  color.text.primary: "{colors.neutral.900}"
  color.text.secondary: "{colors.neutral.700}"
  color.error: "{colors.semantic.error}"
  color.success: "{colors.semantic.success}"
```

Typography
```yaml
typography:
  fonts:
    display: "Bricolage Grotesque, system-ui, sans-serif"
    body: "IBM Plex Sans, system-ui, sans-serif"
    monospace: "JetBrains Mono, monospace"
  scale:
    h1:
      size: 32px
      weight: 700
      lineHeight: 40px
    h2:
      size: 24px
      weight: 600
      lineHeight: 32px
    body:
      size: 16px
      weight: 400
      lineHeight: 24px
    caption:
      size: 12px
      weight: 400
      lineHeight: 16px
```

Spacing
- Base: 8px
- Scale: 4, 8, 12, 16, 20, 24, 32, 40, 48, 64

Radii
- radius.small: 4px
- radius.medium: 8px
- radius.large: 16px
- radius.pill: 9999px

Elevation / Shadows
- elevation.1: 0px 1px 2px rgba(11,18,32,0.04)
- elevation.2: 0px 4px 8px rgba(11,18,32,0.06)
- elevation.3: 0px 8px 24px rgba(11,18,32,0.08)

Motion
- duration.fast: 150ms
- duration.normal: 200ms
- duration.slow: 300ms
- easing.default: cubic-bezier(.2, .8, .2, 1)
- reduceMotion: respects prefers-reduced-motion

Mode Support
- Light mode: tokens above
- Dark mode: inverted semantic tokens + adjusted neutrals (documented in 01_Foundations)
- High contrast variants: ensure text >= 4.5:1

Token Usage Rules (Dev handoff)
- Always reference semantic tokens (color.primary) in components.
- Do NOT use hex literals in component styles.
- Provide fallback tokens in dark mode mapping.

---

## 2. Component Specifications

Naming: C/<Category>/<Name>

Components detail (selected)

1) C/Actions/Button
- Variants:
  - Primary (button.primary.* tokens)
  - Secondary
  - Ghost
- Sizes: Small (32px h), Medium (40px h), Large (48px h)
- States: Default, Hover (+8% brightness), Focus (2px outline using color.primary; meets 3:1 contrast), Active (scale 0.98), Disabled (40% opacity), Loading (spinner inside)
- Token mapping:
  - background: button.primary.background -> color.primary
  - text: button.primary.text -> color.surface
- Accessibility:
  - role="button", aria-pressed as needed, text label required, minimum 44x44 tappable area.

2) C/Inputs/TextField
- Props: label, placeholder, required, helpText, errorText, iconLeading, iconTrailing
- Sizes: Medium default; small variant for compact forms.
- States:
  - Default: neutral border (color.neutral.300)
  - Focus: border color.primary and focus ring (2px)
  - Error: border color.error; error message below, aria-describedby set
- Password field: toggle to show/hide with accessible label.
- Password rules: inline helper with password strength component (C/Forms/PasswordStrength).

3) C/Feedback/Banner
- Types: Info, Success, Warning, Error
- Placement: top of form area or global
- Roles: role="status" (non-blocking) or role="alert" (assertive for critical)
- Behavior: dismissable optional; includes CTA slot.

4) C/Content/Table
- Columns: selectable, primary, metadata, actions
- Desktop: horizontal table with sortable headers
- Mobile: collapsible rows; show primary info and overflow menu for actions.
- Accessibility: <table> semantics with <th scope="col">; keyboard row actions via menu button.

5) C/Modal/Confirm
- Always centralized overlay with focus trap, close on Esc, returns focus to trigger element.
- Motion: fade+scale 180ms.
- Props: title, body, confirmLabel, cancelLabel, destructive (changes button color).

6) C/Status/Badge
- Variants: info, success, error, locked
- Use for lock indicators beside usernames; include accessible label "Account locked".

7) C/Feedback/Skeleton
- Variants: skeleton.card, skeleton.row
- Behavior: shimmer (800ms) by default; disabled when prefers-reduced-motion.

Component states & Variants Checklist
- Each component must define: Type, Size, State, Icon positions per figma-design-standards.
- Limit nested autolayout to 4 levels.

---

## 3. Brand Guidelines

Brand Assets
- Logo: primary and compact (stacked) variations. Provide safe area, minimum sizes, and color variants (light/dark).
- Iconography: system of line icons, 24px baseline, stroke 2px for default; filled for status icons.
- Illustration style: flat, minimal line illustrations for empty states; optional brand-color accents.
- Photography: avoid people-identifiable images in auth flows; use abstract illustrations.

Logo Usage
- Primary logo on light backgrounds: use brand.500 colored logo.
- On dark backgrounds: use white logo version with sufficient contrast.
- Do not alter logo spacing, colors, or rotate/scaling beyond allowed min/max.

Brand Voice (copy guidelines)
- Tone: Clear, calm, helpful.
- Error copy: Non-blaming, action-oriented. Example: "Invalid credentials. Please try again or reset your password."
- Locked account copy: "Your account has been temporarily locked after multiple sign-in attempts. You can request a password reset or contact support."

---

## 4. Accessibility Standards (Design-level)

WCAG 2.2 Level AA checklist (design expectations)
- Contrast:
  - Text >= 4.5:1 for normal text; large text >=3:1
  - UI controls >=3:1 for visible distinctiveness
- Keyboard:
  - All interactive controls keyboard focusable; logical order in Figma annotations
  - Skip links and focus management documented
- Forms:
  - Label association, placeholder not used as label, required fields marked.
  - Error messages bound to inputs with aria-describedby notes.
- Motion:
  - Provide reduced-motion variant; avoid essential information conveyed only via motion.
- Images:
  - Decorative images: alt="" or role=presentation.
  - Illustrations: include short alt + link to longer description if critical.
- Testing:
  - Annotate screens with recommended assistive tech test cases (NVDA, VoiceOver, ChromeVox).

Accessibility Implementation Notes
- Announcements:
  - Use aria-live="polite" for non-blocking messages (info); aria-live="assertive" for blocking errors.
- Focus order:
  - On validation error, set focus to the first invalid field.
  - Modal: trap focus and return focus on close.
- Touch target sizes:
  - Ensure 44x44px minimum tap targets on mobile.

---

## 5. Usage Guidelines and Examples

Example: Login Form usage (mapping tokens)
- Container: max-width token.container.1000, padding: spacing.24
- Email TextField:
  - label: "Email"
  - text style: typography.body
  - placeholder color: color.text.secondary
  - error color: color.error
- Submit Button:
  - variant: C/Actions/Button.Primary, width: fill parent on mobile, size: medium
- Banner:
  - Use C/Feedback/Banner.Error on failed login; role="status" with aria-live.

Example: Admin Table - Locked Accounts
- Table columns: Checkbox | Username (primary) | Locked Since | Attempts | Actions
- Username cell: avatar (C/Content/Avatar small), username text (typography.body.medium), email hidden on row expand (privacy considerations).
- Actions: "View attempts" (opens SCR-008), "Unlock" (opens SCR-009 modal).
- Pagination: prefer pagination over virtualization unless >10k rows; use virtual scroll only if validated.

Form validation patterns
- Client-side:
  - Synchronous checks (empty, format) at input blur; prevent submit until valid.
  - On submit: show spinner; if server returns field errors, map to inputs and show inline errors.
- Server-side:
  - Generic credential error mapping to form-level banner (no email existence disclosure).
  - Locked account returns code + details to trigger Locked UI variant.

Empty State Guidelines
- Use illustration (C/Illustration/EmptyMinimal) + short explanatory copy + primary CTA.
- Example for SCR-007 empty: "No locked accounts — great! Use filters to broaden your search."

Dark Mode & High Contrast
- Ensure all tokens have dark equivalents; test all components to maintain minimum contrast.
- Avoid using color alone; use icons and text labels for status.

Versioning & Governance
- Token change process:
  - All token changes require design review + accessibility sign-off.
  - Version tokens by semantic release: v1, v2, etc. (update 01_Foundations page).
- Component changes:
  - If identical logic appears ≥2 times, extract into shared component per dry-principle-guidelines.

Why these guidelines
- Enforce consistent, accessible UI alignment with product security requirements and administrative workflows. Tokens and components reduce implementation mistakes and ensure compliance with acceptance criteria in the Figma spec.

---

End of Design System Summary.

Note: Full token files and component master frames to be created under 01_Foundations and 02_Components pages in the Figma file; include dev mapping in 06_Handoff.