## Design Reference

## 1. UI Impact Assessment
**Has UI Changes**: [x] Yes [ ] No

- Summary: The authentication feature introduces multiple UI screens (Login, Forgot Password, Reset, MFA enrollment/challenge, Account Locked) and admin surfaces (User Management, Audit Events). These screens require new components, token usage, and prototype flows for validation.

---

## 2. User Story Design Context
**Story ID**: US-AUTH-001  
**Story Title**: Implement Email + Password Authentication Flows (Login, Forgot/Reset, MFA)  
**UI Impact Type**: New UI | UI Enhancement

---

## 3. Design Source References
**Choose applicable reference type:**
- **Figma Project**: [https://figma.com/file/TBD/AuthService] (node-id=TBD) [TO BE UPDATED]
- **Design Images**: `.propel/context/Design/auth_screens/` [TO BE UPDATED]
- **Design System**: `.propel/context/docs/designsystem.md`
- **Brand Guidelines**: `.propel/context/docs/brand_guidelines.md` [TO BE UPDATED]

---

## 4. Screen-to-Design Mappings
| Screen/Feature | Figma Frame ID / Image | Description | Implementation Priority |
|---------------|-------------------------|-------------|-------------------------|
| Login | node-id=TBD | Centralized sign-in card with email/password, forgot link, support link | High |
| Forgot Password | node-id=TBD | Request form with generic confirmation screen after submit | High |
| Reset Password | node-id=TBD | Token-driven reset with password policy checklist | High |
| MFA Enrollment | node-id=TBD | QR provisioning, secret key, verify step, recovery codes | Medium |
| Admin User Management | node-id=TBD | Searchable user list with lock/unlock actions and audit links | Medium |
| Audit Event Detail | node-id=TBD | Event list and event detail drawer for security analysis | Low |

---

## 5. Design Tokens
```yaml
# Only include tokens that are used in this specific UI implementation
colors:
  color.brand.500:
    value: "#0A6CF5"
    usage: "Primary CTAs, links"
    affected_components: ["C/Actions/Button", "C/Navigation/Header"]
  color.neutral.900:
    value: "#111827"
    usage: "Primary text"
    affected_components: ["Body text", "Headers"]
  color.error.500:
    value: "#DC2626"
    usage: "Errors, validation"
    affected_components: ["C/Inputs/TextField", "C/Feedback/Toast"]
  color.success.500:
    value: "#16A34A"
    usage: "Success states"
    affected_components: ["Toasts", "Badges"]

typography:
  heading1:
    family: "Inter" # production token references; replace with approved brand family in design system
    size: "28px"
    weight: "600"
    line-height: "36px"
    used_in: ["Page headers", "Modal titles"]
  body:
    family: "Inter"
    size: "16px"
    weight: "400"
    line-height: "24px"
    used_in: ["Body copy", "Form labels"]
  caption:
    family: "Inter"
    size: "12px"
    weight: "400"
    line-height: "16px"
    used_in: ["Helper text", "Metadata"]

spacing:
  base: "8px"
  scale: [4,8,12,16,20,24,32,40]
  affected_layouts: ["Auth card padding", "Form spacing"]

radius:
  small: "4px"
  medium: "8px"
  large: "16px"

motion:
  duration.short: "150ms"
  duration.medium: "300ms"
  easing: "cubic-bezier(0.2, 0, 0, 1)"
```

---

## 6. Component References
| Component Name | Design Reference | Code Location | UI Changes Required |
|----------------|------------------|---------------|---------------------|
| C/Actions/Button | node-id=TBD | `components/Button.tsx` | Add 'loading' variant; primary/secondary styles per tokens |
| C/Inputs/TextField | node-id=TBD | `components/TextField.tsx` | Add inline validation state, aria-describedby support |
| C/Inputs/PasswordField | node-id=TBD | `components/PasswordField.tsx` | Password policy checklist support and reveal toggle |
| C/Feedback/Toast | node-id=TBD | `components/Toast.tsx` | Non-revealing error styling and aria-live region integration |
| C/Feedback/Modal | node-id=TBD | `components/Modal.tsx` | Focus trap, accessible close, confirm unlock dialog |
| C/Content/Table | node-id=TBD | `components/Table.tsx` | Row actions for Unlock, sortable columns, pagination |

---

## 7. New Visual Assets
```yaml
screenshots:
  location: ".propel/context/Design/US-AUTH-001/"
  files:
    - name: "login_default.png"
      description: "Login form default state"
      source: "figma_frame: node-id=TBD"
    - name: "mfa_enroll.png"
      description: "MFA enrollment QR view"
      source: "figma_frame: node-id=TBD"

new_assets:
  icons:
    - name: "lock_icon.svg"
      source: "figma_export: node-id=I:TBD"
      purpose: "Auth header logo"
  images:
    - name: "empty_state_illustration.svg"
      source: "design_file: empty_illustration.svg"
      requirements: "SVG, single-color adaptable via color tokens"
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
  title: "Add Forgot Password Flow"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD"]  # Forgot password form + confirmation
  components_affected:
    - C/Inputs/TextField
    - C/Actions/Button
  visual_validation_required: true

TASK-003:
  title: "Add MFA Enrollment & Challenge"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=TBD", "node-id=TBD"]  # MFA enroll + challenge
  components_affected:
    - C/Image/QRCode
    - C/Inputs/TextField
    - RecoveryCodesPanel
  visual_validation_required: true
```

---

## 9. Implementation Scenarios
```yaml
new_components:
  - name: "PasswordPolicyChecklist"
    figma_reference: "node-id=TBD"
    file_location: "components/PasswordPolicyChecklist.tsx"
    design_specifications:
      width: "100%"
      height: "auto"
      states: ["all_pass", "partial", "none_pass"]
      token_usage: ["color.error.500", "color.success.500"]

ui_enhancements:
  existing_component: "Button"
  changes_required:
    - "Add loading spinner variant"
    - "Disabled hover state behavior"
    - "Ensure accessible label 'aria-label' support"
  figma_reference: "node-id=TBD"

backend_task:
  ui_impact: false
  design_references: "Not applicable - server side only"
  validation_type: "API tests and integration"
```

---

## 10. Accessibility Requirements
- **WCAG Level**: AA (WCAG 2.2) for all new/modified UI elements
- **Color Contrast**: Text contrast >= 4.5:1; large text >= 3:1; UI components >= 3:1 where applicable
- **Focus States**: Visible focus for all interactive elements; focus outline contrast >=3:1
- **Screen Reader**: All inputs with labels; errors announced via aria-live; modal dialogs manage focus trap and restore focus on close
- **Keyboard**: Full keyboard navigation with logical tab order and skip links where pages repeat content
- **Form Accessibility**: Each input has an associated label and aria-required when required; validation messages linked with aria-describedby

---

## 11. Design Review Checklist
**Complete only if ui_impact: true**
- [ ] Figma frames reviewed for all UI changes
- [ ] Design tokens extracted for affected components
- [ ] Component specifications documented with variants and states
- [ ] Visual validation criteria defined (pixel checks, breakpoints)
- [ ] Responsive behavior specified for mobile/tablet/web
- [ ] Accessibility requirements noted and ARIA suggestions provided
- [ ] Handoff notes prepared for developers (token mapping, code locations)
- [ ] Placeholder node-ids replaced with final frame links before handoff

Notes:
- Replace all `node-id=TBD` placeholders with final Figma links prior to final handoff.
- Coordinate with security and backend teams for audit event field naming to ensure UI-to-logging traceability.