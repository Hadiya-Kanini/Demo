## Design Reference

## 1. UI Impact Assessment
**Has UI Changes**: [x] Yes [ ] No

Summary:
- UI changes required for authentication flows: Login, Forgot Password, Reset Password, Lockout, Rate-limit, Session expiry, Admin unlock. These changes include new or updated components (C/Inputs/PasswordField, C/Actions/Button variants, C/Feedback/Alert, C/Feedback/CAPTCHA) and corresponding token definitions. Preserve all existing tokens where applicable; add tokens necessary for error, success, and CAPTCHA states.

---

## 2. User Story Design Context
**Story ID**: US-AUTH-001  
**Story Title**: Implement Email/Password Authentication UI & Recovery Flows  
**UI Impact Type**: New UI (Auth screens) + Component Update (password field, button loading state)

---

## 3. Design Source References
- **Figma Project**: https://figma.com/file/TBD/AuthApp [node-id=TBD] [TO BE UPDATED]
- **Design Images**: `.propel/context/Design/auth_screens/` [TO BE UPDATED]
- **Design System**: `.propel/context/docs/designsystem.md`
- **Brand Guidelines**: `.propel/context/docs/brand_guidelines.md` [TO BE UPDATED]

---

## 4. Screen-to-Design Mappings
| Screen/Feature | Figma Frame ID / Image | Description | Implementation Priority |
|---------------|-------------------------|-------------|-------------------------|
| Login | node-id=TBD | Login form with email, password, show/hide toggle, CTA | High |
| Forgot Password | node-id=TBD | Request screen with non-revealing copy and CTA | High |
| Reset Password | node-id=TBD | Token-validated reset form with password helper | High |
| Account Locked | image: locked_screen.png | Lockout guidance with TTL and contact support | High |
| CAPTCHA Flow | node-id=TBD | CAPTCHA placeholder area and accessible label | Medium |

---

## 5. Design Tokens
```yaml
# Only include tokens that are used in this specific UI implementation
colors:
  color.primary:
    value: "#007BFF"
    usage: "Primary CTAs, active states"
    affected_components: ["C/Actions/Button", "Links"]
  color.primary-dark:
    value: "#0056B3"
    usage: "Primary hover/active"
    affected_components: ["C/Actions/Button"]
  color.success:
    value: "#16A34A"
    usage: "Success messages, confirmations"
    affected_components: ["C/Feedback/Toast", "C/Feedback/Alert"]
  color.warning:
    value: "#F59E0B"
    usage: "Warnings, attention"
    affected_components: ["C/Feedback/Alert"]
  color.error:
    value: "#DC2626"
    usage: "Validation and error visuals"
    affected_components: ["C/Inputs/TextField", "C/Feedback/Alert"]
  color.neutral-100:
    value: "#F8F9FA"
    usage: "Page background"
    affected_components: ["Layout", "Cards"]
  color.neutral-900:
    value: "#111827"
    usage: "Primary text"
    affected_components: ["Typography", "Buttons"]

typography:
  type.family.base: "InterVariable" # placeholder; swap per brand
  type.scale.h1:
    size: "28px"
    weight: "600"
    lineHeight: "36px"
    usage: "Page headers"
  type.scale.body:
    size: "16px"
    weight: "400"
    lineHeight: "24px"
    usage: "Body copy"
  type.scale.caption:
    size: "12px"
    weight: "400"
    lineHeight: "16px"
    usage: "Helper text, inline hints"

spacing:
  spacing.base: "8px"
  spacing.scale: ["4px","8px","12px","16px","24px","32px","40px","48px"]

radius:
  radius.small: "4px"
  radius.medium: "8px"
  radius.large: "16px"

elevation:
  elevation.level-1: "0 1px 2px rgba(16,24,40,0.05)"
  elevation.level-2: "0 4px 12px rgba(16,24,40,0.08)"

motion:
  motion.duration.short: "150ms"
  motion.duration.medium: "300ms"
  motion.easing: "ease-out"
```

---

## 6. Component References
| Component Name | Figma Component / Image | Code Location | UI Changes Required |
|----------------|-------------------------|---------------|---------------------|
| C/Actions/Button | node-id=C:Button:TBD | components/Button.tsx | Add 'loading' variant (spinner inside), ensure disabled state uses color.disabled token |
| C/Inputs/TextField | node-id=C:TextField:TBD | components/TextField.tsx | Ensure aria-describedby support; validation state styling |
| C/Inputs/PasswordField | node-id=C:PasswordField:TBD | components/PasswordField.tsx | Add show/hide toggle variant; include C/Content/PasswordHelper integration |
| C/Feedback/Alert | node-id=C:Alert:TBD | components/Alert.tsx | Add non-revealing copy patterns and role="alert" | 
| C/Feedback/CAPTCHA | node-id=C:CAPTCHA:TBD | components/CaptchaWrapper.tsx | Placeholder for CAPTCHA integration; accessible labeling required |
| C/Feedback/Skeleton | node-id=C:Skeleton:TBD | components/Skeleton.tsx | Skeleton for loading states in SCR-006 and dashboards |

---

## 7. New Visual Assets
```yaml
screenshots:
  location: ".propel/context/Design/US-AUTH-001/"
  files:
    - name: "login_form.png"
      description: "Login form reference"
      source: "figma_frame: node-id=TBD"
    - name: "reset_confirmation.png"
      description: "Generic confirmation screen for forgot password"
      source: "figma_frame: node-id=TBD"

new_assets:
  icons:
    - name: "visibility_toggle.svg"
      source: "figma_export: node-id=I:TBD"
      purpose: "Show/hide password"
    - name: "lockout_illustration.svg"
      source: "figma_export: node-id=I:TBD"
      purpose: "Account locked illustration for SCR-005"
  images:
    - name: "generic_empty.svg"
      source: "figma_export: node-id=IMG:TBD"
      purpose: "Empty/No data placeholder"
```

---

## 8. Task Design Mapping
```yaml
TASK-001:
  title: "Implement Login UI"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD"]  # Login form
  components_affected:
    - C/Inputs/TextField
    - C/Inputs/PasswordField
    - C/Actions/Button
  visual_validation_required: true

TASK-002:
  title: "Implement Forgot Password UI"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD"]
  components_affected:
    - C/Inputs/TextField
    - C/Actions/Button
  visual_validation_required: true

TASK-003:
  title: "Add Admin Unlock View"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD"]
  components_affected:
    - C/Content/Card
    - C/Actions/Button
  visual_validation_required: true

TASK-004:
  title: "Integrate CAPTCHA placeholder"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD"]
  components_affected:
    - C/Feedback/CAPTCHA
  visual_validation_required: true
```

---

## 9. Implementation Scenarios
```yaml
new_components:
  - name: "PasswordField"
    figma_reference: "node-id=TBD"
    file_location: "components/PasswordField.tsx"
    design_specifications:
      width: "100% with max-width: 480px"
      height: "48px"
      border_radius: "{radius.medium}"
      states: ["default", "focused", "error", "disabled", "loading"]
      accessibility:
        - label: "Password"
        - aria-describedby: "password-helper"
  - name: "CaptchaWrapper"
    figma_reference: "node-id=TBD"
    file_location: "components/CaptchaWrapper.tsx"
    design_specifications:
      placement: "Inline beneath submit button when triggered"
      accessibility: "Must provide accessible challenge alternatives"

ui_enhancements:
  existing_component: "Button"
  changes_required:
    - "Add loading state with spinner (use token color.primary on spinner)"
    - "Add disabled state styling using color.neutral-400"
    - "Ensure button has aria-busy when loading"

backend_task:
  ui_impact: false
  description: "Auth API endpoints and token lifecycle; no UI changes required"
```

---

## 10. Accessibility Requirements
- **WCAG Level**: AA (WCAG 2.2)
- **Color Contrast**: Text contrast >= 4.5:1 for body; large text >= 3:1; UI elements >= 3:1
- **Focus States**: All interactive elements must have visible focus outlines using token (focus.outline.color) with minimum 2px thickness
- **Screen Reader**: Semantic elements and ARIA where required:
  - Forms: <label for> associated with inputs
  - Errors: aria-live="assertive" for error banners
  - Modals: role="dialog", aria-modal="true", focus trap until closed
- **Keyboard**: All interactive controls reachable via keyboard; tab order follows logical reading order
- **Touch Targets**: Minimum 44x44px clickable/tappable area on mobile

---

## 11. Design Review Checklist
**Complete only for ui_impact: true tasks**
- [ ] Figma frames reviewed for all UI changes (node-id placeholders mapped)
- [ ] Design tokens extracted for affected components
- [ ] Component specifications documented and mapped to code locations
- [ ] Visual validation criteria defined (screenshot comparison thresholds)
- [ ] Responsive behavior specified and documented for breakpoints
- [ ] Accessibility requirements annotated (focus order, aria suggestions)
- [ ] Export manifest entries added to 06_Handoff

Notes:
- For any node-id or asset path marked [TO BE UPDATED], update entries before final handoff.
- Ensure component code locations exist in repo or coordinate with engineering to create scaffold files.

---