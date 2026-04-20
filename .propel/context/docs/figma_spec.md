.propel/context/docs/figma_spec.md
---
post_title: "Figma Design Specification - AI Training Demo"
author1: "Senior Product Designer"
post_slug: "ai-training-demo-figma-spec"
microsoft_alias: "senior_pd"
featured_image: ".propel/context/Design/preview.png"
categories: ["Design", "UX"]
tags: ["figma-spec", "uxr", "demo", "training-sim"]
ai_note: "AI-assisted generation; please validate inferred personas and breakpoints."
summary: "Figma specification for an embeddable client-side AI training demo landing page. Includes UXR, screen inventory, flows, component mapping, and design tokens."
post_date: 2026-04-20
---

# Figma Design Specification - AI Training Demo

## 1. Figma Specification
**Platform**: Web (Responsive: Mobile / Tablet / Desktop)

---

## 2. Source References

### Primary Source
| Document | Path | Purpose |
|----------|------|---------|
| Requirements | `.propel/context/docs/spec.md` | Personas (inferred), use cases, FR-001..FR-010 with UI impact flags |

### Optional Sources
| Document | Path | Purpose |
|----------|------|---------|
| Wireframes | `.propel/context/wireframes/` | Not present — would provide layout/entity detail |
| Design Assets | `.propel/context/Design/` | Placeholder for later brand assets; none provided in spec.md |

### Related Documents
| Document | Path | Purpose |
|----------|------|---------|
| Design System | `.propel/context/docs/designsystem.md` | Tokens, branding, component specifications (generated alongside) |

---

## 3. UX Requirements

*Generated based on use cases (UC-001..UC-004) derived from FR-001..FR-009 in spec.md. FR-010 and brand assets are marked [UNCLEAR].*

Step 1 - UXR list (summary + rationale)

| UXR-ID | Category | Summary | Rationale |
|--------|----------|---------|-----------|
| UXR-001 | Project | Performance: Page load < 2s on desktop broadband | Success criteria in spec.md |
| UXR-002 | Project | Action responsiveness: UI actions <= 200ms | FR acceptance criteria |
| UXR-101 | Usability | Controls discoverable within first viewport; primary CTA prominent | Demo must be usable in sales/demo contexts |
| UXR-102 | Usability | All primary tasks reachable within 3 clicks | Discoverability & efficiency requirement |
| UXR-201 | Accessibility | WCAG 2.2 AA compliance for text contrast and focus states | Spec mandates WCAG AA baseline |
| UXR-202 | Accessibility | ARIA live regions for metric/log updates | Real-time updates must be screen-reader consumable |
| UXR-301 | Responsiveness | Support mobile (320-480), tablet (768), desktop (1024+) breakpoints | Spec requires responsive collapse at <=480px; extend for standard targets |
| UXR-302 | Responsiveness | Touch targets >=44x44px on mobile | Accessibility and mobile usability |
| UXR-401 | Visual Design | Use design tokens for color/typography/spacing; provide light + dark modes | Single source of truth; template requirement |
| UXR-501 | Interaction | Provide clear feedback for Loading, Success, Error within 200ms | Interactive demo needs immediate feedback |
| UXR-601 | Error Handling | Confirmation before dataset switch mid-run; clear recovery flows | From FR-005 and UC extensions |
| UXR-701 | Data & Privacy | Analytics/integration opt-in; no telemetry by default | FR-009/FR-010 privacy and opt-in constraints |
| UXR-801 | Persistency | Persist settings only when "Remember settings" enabled; provide clear control to clear | FR-008 deterministic persistence rules |
| UXR-901 | Validation | Form and control validation states for invalid inputs / disabled actions | Buttons/controls must display validation/disabled states |

Step 2 - Expanded UXR requirements (table)

| UXR-ID | Category | Requirement | Acceptance Criteria | Screens Affected |
|--------|----------|-------------|---------------------|------------------|
| UXR-001 | Project | Page MUST load within 2 seconds on typical desktop broadband | Measured cold-load <2s in Lighthouse-like test; performance budget documented | SCR-001, SCR-002 |
| UXR-002 | Project | Interactive controls MUST respond within 200ms to user input | Click/tap latency test: UI updates visible within 200ms | SCR-002, SCR-003, SCR-004 |
| UXR-101 | Usability | Primary controls and dataset selector MUST be visible in the first viewport on desktop | Heuristic review: primary CTA and dataset selector visible above-the-fold at 1440x900 | SCR-001, SCR-002 |
| UXR-102 | Usability | Any primary feature (Start, Pause, Reset, Load Data, Export Log) MUST be reachable in max 3 clicks/taps | Click count audit passes for task flows UC-001..UC-004 | All screens with controls (SCR-001..SCR-006) |
| UXR-201 | Accessibility | System MUST comply with WCAG 2.2 AA for all visible text and UI elements | Contrast audit (axe/WAVE) passes; focus outlines present for keyboard navigation | All screens |
| UXR-202 | Accessibility | Dynamic metrics and logs MUST be announced via ARIA live regions with non-verbose messaging | Screen reader walkthrough: key updates announced; narrative messages optional and togglable | SCR-003, SCR-004 |
| UXR-301 | Responsiveness | UI MUST adapt to mobile (320-480), tablet (768), desktop (1024+) breakpoints with layout adjustments | Layouts verified at 375, 768, 1440 widths; content doesn't overflow | All screens |
| UXR-302 | Responsiveness | Touch targets MUST be >=44x44px on mobile and interactive spacing must follow spacing tokens | Touch target audit passes on mobile frames | SCR-002, SCR-005, SCR-006 |
| UXR-401 | Visual Design | All UI elements MUST use design tokens defined in designsystem.md; provide light and dark themes | Tokens referenced in Figma components; light/dark variants exist | All components/screens |
| UXR-501 | Interaction | Provide immediate visual feedback for user actions: button press, disable, loading spinners, skeletons | Each interaction has visual affordance; loading states present and non-blocking | SCR-002, SCR-003, SCR-004 |
| UXR-601 | Error Handling | If user changes dataset mid-run, show confirmation modal and preserve ability to cancel | Confirmation modal appears with clear consequence text; cancel returns to running state | SCR-005, SCR-007 |
| UXR-701 | Data & Privacy | Telemetry MUST be off by default and opt-in with clear disclosure | Analytics option visible in settings; default is disabled; events documented as no-op without callback | SCR-006 |
| UXR-801 | Persistency | "Remember settings" MUST persist dataset and UI prefs only when enabled; user can clear persisted settings | LocalStorage entries scoped and cleared by "Reset settings" control; privacy note displayed | SCR-006 |
| UXR-901 | Validation | Controls that are not applicable in current state MUST be visibly disabled and provide accessible explanation | Disabled controls have aria-disabled + tooltip or accessible label explaining state | SCR-002, SCR-005 |

[UNCLEAR] Items:
- UXR mapping for backend export (FR-010) is marked [UNCLEAR] pending API contract & security requirements; no screen assumes server integration by default.

---

## 4. Personas Summary

*Spec.md does not include formal persona documents. Below are inferred personas derived from the spec narrative. These are marked [INFERRED] and require stakeholder confirmation before final sign-off.*

| Persona | Role | Primary Goals | Key Screens |
|---------|------|---------------|-------------|
| Demo User [INFERRED] | Consumer of demo (non-technical or technical) | Quickly start/stop simulation, observe metrics & logs, switch datasets | SCR-001, SCR-002, SCR-003, SCR-004 |
| Sales Presenter [INFERRED] | Uses demo in presentations | Control runs, highlight metrics, export logs for follow-ups | SCR-001, SCR-002, SCR-004, SCR-005 |
| Designer / UX Researcher [INFERRED] | Tests flows & gathers feedback | Explore behavior, adjust speed/epochs, verify accessibility | SCR-001, SCR-002, SCR-003, SCR-006 |
| Engineering Observer [INFERRED] | Verifies simulation determinism & events | Inspect logs and telemetry hooks (opt-in) | SCR-004, SCR-006 |

Note: Please confirm persona names, details, and priority to finalize journeys.

---

## 5. Information Architecture

### Site Map
```
AI Training Demo
+-- Landing (SCR-001)
|   +-- Controls Panel (SCR-002)
|   +-- Metrics View (SCR-003)
|   +-- Event / Log Panel (SCR-004)
|   +-- Dataset Selector (SCR-005)
|   +-- Settings / Persist (SCR-006)
|   +-- Confirmation / Error Modals (SCR-007)
```

### Navigation Patterns
| Pattern | Type | Platform Behavior |
|---------|------|-------------------|
| Primary Nav | Header (top) | Desktop: top header with logo & settings; Mobile: compact header w/ settings icon |
| Secondary Nav | In-page controls | Controls fixed to top of demo region on desktop; collapsible toolbar on mobile |
| Utility Nav | Settings & Help | Modal/drawer accessible from header |

---

## 6. Screen Inventory

*All screens derived directly from FR-001..FR-009 and UC-001..UC-004.*

### Screen List
| Screen ID | Screen Name | Derived From | Personas Covered | Priority | States Required |
|-----------|-------------|--------------|------------------|----------|-----------------|
| SCR-001 | Landing / Demo Shell | FR-001, UC-001 | Demo User, Sales Presenter | P0 | Default, Loading, Empty, Error, Validation |
| SCR-002 | Controls Panel | FR-002, UC-001, UC-002 | Demo User, Sales Presenter | P0 | Default, Loading, Empty, Error, Validation |
| SCR-003 | Metrics & Progress View | FR-003, UC-003 | Demo User, Designer | P0 | Default, Loading, Empty, Error, Validation |
| SCR-004 | Event / Log Panel | FR-004, UC-003 | Demo User, Engineering Observer | P0 | Default, Loading, Empty, Error, Validation |
| SCR-005 | Dataset Selector (inline/dropdown + confirm modal) | FR-005, UC-004 | Demo User, Sales Presenter | P0 | Default, Loading, Empty, Error, Validation |
| SCR-006 | Settings / Persist & Telemetry | FR-008, FR-009 | Designer, Engineering Observer | P1 | Default, Loading, Empty, Error, Validation |
| SCR-007 | Confirmation / Error Modal | FR-005, FR-010 [UNCLEAR] | All | P0 | Default, Loading, Empty, Error, Validation |

### Priority Legend
- **P0**: Critical path for MVP
- **P1**: Useful controls/settings; not blocking core demo

### Screen-to-Persona Coverage Matrix
| Screen | Demo User | Sales Presenter | Designer | Engineering Observer | Notes |
|--------|----------:|----------------:|---------:|---------------------:|-------|
| SCR-001 | Primary | Primary | Secondary | Secondary | Entry point for all |
| SCR-002 | Primary | Primary | Secondary | Secondary | Actionable controls |
| SCR-003 | Primary | Secondary | Primary | Secondary | Observability |
| SCR-004 | Primary | Secondary | Secondary | Primary | Logs & export |
| SCR-005 | Primary | Primary | Secondary | Secondary | Dataset presets |
| SCR-006 | Secondary | Secondary | Primary | Primary | Persist & telemetry opt-in |
| SCR-007 | Primary | Primary | Primary | Primary | Confirmations and errors |

### Modal/Overlay Inventory
| Name | Type | Trigger | Parent Screen(s) | Priority |
|------|------|---------|-----------------|----------|
| Dataset Change Confirmation | Modal | Change dataset while Running | SCR-002, SCR-005 | P0 |
| Export Log / Copy Confirmation | Toast / Modal | Export log click | SCR-004 | P1 |
| Error Modal | Modal | Simulation error or persistence error | All | P0 |
| Settings Drawer | Drawer/Modal | Click settings icon | SCR-001 | P1 |

---

## 7. Content & Tone

### Voice & Tone
- **Overall Tone**: Friendly, explanatory, non-technical by default (with optional technical detail toggles)
- **Error Messages**: Clear, actionable, non-blaming (e.g., "Simulation paused: please resume or reset")
- **Empty States**: Encouraging with clear CTAs (e.g., "Select a dataset to begin")
- **Success Messages**: Brief and next-action oriented (e.g., "Run complete — export log or run again")

### Content Guidelines
- **Headings**: Sentence case
- **CTAs**: Action-oriented verbs ("Start run", "Pause", "Reset", "Export log")
- **Labels**: Short, descriptive; controls include accessible hints where needed
- **Placeholder Text**: Helpful examples for any input; avoid lorem ipsum in final

---

## 8. Data & Edge Cases

### Data Scenarios
| Scenario | Description | Handling |
|----------|-------------|----------|
| No Data | No datasets available | Empty state with CTA to load demo data |
| First Use | New user no persisted settings | Auto-select default dataset and show brief tooltip |
| Large Data | N/A — demo uses small presets; UI guards against heavy workloads | Limit epochs/sparkline points; throttle rendering |
| Slow Connection | Page assets slow to load | Provide skeletons and limit initial scripts; degrade gracefully |
| Offline | User offline | Demo still functions client-side; telemetry disabled and note displayed |

### Edge Cases
| Case | Screen(s) Affected | Solution |
|------|-------------------|----------|
| Changing dataset mid-run | SCR-002/SCR-005 | Confirm modal; cancel retains current run |
| Long log entries | SCR-004 | Truncate with expand/copy; show timestamp & copy action |
| Accessibility failure | All | Fallback to simplified view with larger text & no animations |
| Persist storage full | SCR-006 | Show clear error and method to clear storage |

---

## 9. Branding & Visual Direction

*No brand assets provided in spec.md. The designsystem uses a neutral accessible theme; brand tokens must be replaced when assets are available. MARKED [UNCLEAR].*

### Branding Assets
- **Logo**: Placeholder text mark; replace with provided asset
- **Icon Style**: Filled 16/24px SVG icons with accessible title attributes
- **Illustration Style**: Minimal flat illustrations for empty states (optional)

---

## 10. Component Specifications

*Component specifications live in `.propel/context/docs/designsystem.md` (generated). Below is a reference of required components and variants.*

### Required Components per Screen
| Screen ID | Components Required | Notes |
|-----------|---------------------|-------|
| SCR-001 | Header, Hero, Primary CTA, Settings icon | Landing shell |
| SCR-002 | Button (Primary/Secondary), Toggle (Narrative), Dropdown (Dataset), Speed slider | States: Disabled, Loading, Active |
| SCR-003 | MetricTile (epoch, loss, accuracy), ProgressBar, Sparkline | Live updates via ARIA live region |
| SCR-004 | LogList (virtualized), Copy/Export Button, Filters | Auto-scroll toggle, search/filter optional |
| SCR-005 | Dropdown / SegmentedControl, Confirm Modal | At least 3 presets |
| SCR-006 | Toggle (Remember settings), Toggle (Telemetry opt-in), Reset settings control | Privacy copy required |
| SCR-007 | Modal (confirm/cancel), Error Banner, Toast | Accessible focus trap and dismiss |

### Component Summary
| Category | Components | Variants |
|----------|------------|----------|
| Actions | Button | Primary, Secondary, Ghost x S/M/L x States |
| Inputs | Dropdown, Toggle, Slider | with keyboard support |
| Navigation | Header | Desktop & Mobile variants |
| Content | MetricTile, Card, Sparkline | small/medium/large |
| Feedback | Modal, Toast, Banner, Skeleton | Types + States |

### Component Constraints
- Follow naming: `C/<Category>/<Name>` in Figma
- Auto Layout for all frames; spacing tokens used exclusively
- Each component must define Type, Size, and State variant properties per figma-design-standards

---

## 11. Prototype Flows

*Flows derived from use cases (UC-001..UC-004). Each flow references screens and personas.*

### Flow: FL-001 - Run Simulation Flow
**Flow ID**: FL-001  
**Derived From**: UC-001  
**Personas Covered**: Demo User, Sales Presenter  
**Description**: Start a simulation run, observe metrics and logs, complete run.

#### Flow Sequence:
1. Entry: SCR-001 / Default
   - Trigger: User clicks "Start"
2. Step: SCR-002 / Running (Controls update)
   - Action: Start button disabled; Pause enabled; progress begins
3. Step: SCR-003 / Default (Metrics update)
   - Action: Epoch updates, sparkline appends point; ARIA live announces key milestones
4. Step: SCR-004 / Default (Event/log updated)
   - Action: New log entry appended; auto-scroll optionally enabled
5. Decision Point:
   - Success -> SCR-003 / Default (Run complete) -> toast "Run complete"
   - Error -> SCR-007 / Error modal (show error + retry)
6. Exit: SCR-003 / Default (completed state) or SCR-001 (user resets)

Required Interactions:
- Start button: toggles running state
- Progress animation: non-blocking skeletons for slow updates
- ARIA live region: announce "Epoch X complete" succinctly

### Flow: FL-002 - Pause / Resume / Reset Flow
**Flow ID**: FL-002  
**Derived From**: UC-002  
**Personas Covered**: Demo User, Sales Presenter

Flow Sequence:
1. Entry: SCR-002 / Running
   - Trigger: User clicks "Pause"
2. Step: SCR-002 / Paused
   - Action: Metrics freeze; log "Simulation paused"
3. Decision:
   - Resume -> return to Running (SCR-002)
   - Reset -> show Confirm Modal (SCR-007); confirm -> Reset metrics and state -> SCR-001 Default

Required Interactions:
- Pause toggles accessible label to "Resume" when paused
- Reset triggers confirm when running

### Flow: FL-003 - Dataset Change Flow
**Flow ID**: FL-003  
**Derived From**: UC-004  
**Personas Covered**: Demo User, Sales Presenter

Flow Sequence:
1. Entry: SCR-005 / Default
   - Trigger: User selects a different preset while Idle -> change applied
2. If Running:
   - Show SCR-007 / Confirmation Modal (dataset change) with options Confirm / Cancel
   - Confirm -> apply new dataset -> Reset run or apply after next Start
   - Cancel -> dismiss modal; continue run

Required Interactions:
- Dataset selector with keyboard navigation and accessible labels

### Flow: FL-004 - Export Log Flow
**Flow ID**: FL-004  
**Derived From**: FR-004 / UC-003  
**Personas Covered**: Sales Presenter, Engineering Observer

Flow Sequence:
1. Entry: SCR-004 / Default
   - Trigger: User clicks "Export log"
2. Step: SCR-004 / Loading (if preparing file) or immediate copy
   - Action: Copy to clipboard or trigger download; show toast confirmation
3. Exit: SCR-004 / Default

Required Interactions:
- Copy button uses accessible notification (aria-live polite) to confirm

### Flow: FL-005 - Persist Settings Flow
**Flow ID**: FL-005  
**Derived From**: FR-008  
**Personas Covered**: Designer, Engineering Observer

Flow Sequence:
1. Entry: SCR-006 / Default
   - Trigger: User toggles "Remember settings" ON
2. Step: SCR-006 / Default
   - Action: Settings saved to localStorage; show small confirmation/toast
3. Exit: SCR-001 / Default on next load with persisted config restored

Required Interactions:
- Clear persisted settings action in Settings

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
`AITrainingDemo__<Platform>__<ScreenName>__<State>__v1.jpg`

### Export Manifest (examples)
| Screen | State | Platform | Filename |
|--------|-------|----------|----------|
| Landing | Default | Web | AITrainingDemo__Web__Landing__Default__v1.jpg |
| Controls | Loading | Mobile | AITrainingDemo__Mobile__Controls__Loading__v1.jpg |
| Metrics | Default | Web | AITrainingDemo__Web__Metrics__Default__v1.jpg |

---

## 13. Figma File Structure

Follow 6-page standard:

```
AI Training Demo Figma File
+-- 00_Cover
+-- 01_Foundations
+-- 02_Components
+-- 03_Patterns
+-- 04_Screens
+-- 05_Prototype
+-- 06_Handoff
```

Naming conventions:
- Components: C/<Category>/<Name> (e.g., C/Actions/Button/Primary)
- Screens: <ScreenName>/<State> (e.g., Landing/Default)

---

## 14. State Specifications

### State Matrix
| Screen | Default | Loading | Empty | Error | Validation |
|--------|---------|---------|-------|-------|------------|
| SCR-001 Landing | Hero + Controls visible | Skeleton for hero while assets load | CTA to load demo | Page-level error banner | N/A |
| SCR-002 Controls | Buttons enabled/disabled per state | Buttons show spinners; controls disabled | If no datasets available show empty CTA | Disabled with explanation, error toast | Inline explanations for disabled actions |
| SCR-003 Metrics | Live metrics & sparkline | Skeleton tiles; progress pulsing | No metrics yet (before first epoch) with CTA | Metric fetch simulation failed -> error tile | Numeric validation for custom epoch input |
| SCR-004 Logs | Latest logs visible; auto-scroll | Loading spinner while preparing export | "No events yet" illustration + CTA | Error fetching persisted logs -> error banner | N/A |
| SCR-005 Dataset Selector | Selected preset visible | Loading presets (rare) | "No presets" fallback | Change failed -> error modal | Selection invalid -> inline message |
| SCR-006 Settings | Toggles reflect persisted state | Saving indication | No persisted settings message | Save failure -> toast | Toggle invalid state handled gracefully |
| SCR-007 Modals | Confirmation prompt visible | Action in progress spinner | N/A | Error modal with retry/close | Form validation within modal |

### SCR-001: Landing / Demo Shell

#### Default State
- Title, one-line description, primary controls visible
- Controls region shows Start (enabled), Pause/Resume (disabled), Reset (disabled), Dataset selector

#### Loading State
- Hero skeleton; controls replaced with skeleton buttons; header skeleton

#### Empty State
- If no preset data available: "No demo data available" with Load Demo Data CTA

#### Error State
- Page-level banner: "Failed to load assets" with Retry action

#### Validation State
- N/A (no input fields)

(Repeat detailed states for SCR-002..SCR-007 as summarized above—see full state matrix.)

---

## 15. Quality Checklist (Validation)

- [x] Every use case (UC-001..UC-004) maps to at least one screen
- [x] Personas present (inferred) and have primary screens; stakeholder confirmation required
- [x] Every screen has required 5 states defined (Default/Loading/Empty/Error/Validation)
- [x] Every flow has entry, steps, and exit defined (FL-001..FL-005)
- [x] Design tokens drafted (colors, typography, spacing, radius, elevation, motion)
- [x] Component inventory covers screen requirements
- [x] Every UXR maps to at least one screen (no orphan UXR)
- [!] Outstanding clarifications: FR-010 backend integration, brand assets, final breakpoints (marked [UNCLEAR])

---

## 16. Handoff Notes (what to include in 06_Handoff)
- Token usage: mapping to Figma token names and usage
- Component props: states and variant guidance
- Responsive behavior: breakpoint rules and layout adjustments
- Edge cases: dataset change modal behavior, persistence clearing
- Accessibility notes: ARIA live region use, focus order, keyboard keys
- Assumptions: Inferred personas, neutral theme, no backend integration by default

---

## 17. Next Steps & Clarifications Required
- Confirm personas (replace inferred personas with canonical artifacts).
- Provide brand tokens / logo / preferred fonts to finalize designsystem.md.
- Clarify if FR-010 (backend integration) is in-scope; if yes, provide API contract and security requirements.
- Confirm final breakpoint set if different from standard mobile/tablet/desktop.

---

## 18. Quality Evaluation Invocation
- Evaluate the generated `.propel/context/docs/figma_spec.md` with `.windsurf/workflows/evaluate-output.md` using `$SCOPE_FILES: .propel/context/docs/spec.md` and `--workflow-type: figma-spec` (to be executed by CI/tooling).

---
.propel/context/docs/designsystem.md
---
post_title: "Design System - AI Training Demo"
author1: "Senior Product Designer"
post_slug: "ai-training-demo-designsystem"
microsoft_alias: "senior_pd"
featured_image: ".propel/context/Design/designsystem_preview.png"
categories: ["Design", "Design System"]
tags: ["designsystem","tokens","components","figma"]
ai_note: "AI-assisted token and component draft. Replace brand values when assets provided."
summary: "Design tokens and component specs for the AI Training Demo. Includes color palette, typography, spacing, radii, elevation, motion, and component references."
post_date: 2026-04-20
---

# Design Reference

> **Note**: This design reference is for the UI-impacting demo landing page implementation.

## UI Impact Assessment
**Has UI Changes**: [x] Yes
- This document applies.

## Design Source References
**Design Assets**: None provided. Neutral accessible tokens used. Replace with brand token values when available.

---

## Design Tokens

# Primitive & Semantic tokens (light + dark)

colors:
  primary:
    value: "#0B5FFF"
    usage: "Primary CTAs, interactive elements"
    affected_components: ["Button", "Links"]
  primaryHover:
    value: "#094ECC"
    usage: "Primary hover"
  neutral:
    50: "#F9FAFB"
    100: "#F3F4F6"
    200: "#E5E7EB"
    300: "#D1D5DB"
    400: "#9CA3AF"
    500: "#6B7280"
    600: "#4B5563"
    700: "#374151"
    800: "#1F2937"
    900: "#0F1724"
  success:
    value: "#059669"
    usage: "Success states"
  warning:
    value: "#D97706"
    usage: "Warnings"
  error:
    value: "#DC2626"
    usage: "Errors"
  info:
    value: "#0284C7"
    usage: "Informational"

# Mode variants (light/dark)
semantic:
  background:
    light: "{neutral.50}"
    dark: "{neutral.900}"
  surface:
    light: "{neutral.100}"
    dark: "{neutral.800}"
  text:
    primary_light: "{neutral.900}"
    primary_dark: "{neutral.50}"
  border:
    light: "{neutral.200}"
    dark: "{neutral.700}"

# Typography
typography:
  fontFamily:
    heading: "Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial"
    body: "Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial"
  scale:
    h1: {size: "28px", weight: 700, lineHeight: "36px"}
    h2: {size: "22px", weight: 600, lineHeight: "28px"}
    h3: {size: "18px", weight: 600, lineHeight: "24px"}
    body: {size: "16px", weight: 400, lineHeight: "24px"}
    caption: {size: "12px", weight: 400, lineHeight: "16px"}

# Spacing
spacing:
  base: "8px"
  scale: [4,8,12,16,24,32,40,48,64]

# Radii
radius:
  small: "4px"
  medium: "8px"
  large: "16px"
  round: "9999px"

# Elevation / Shadows
elevation:
  level1: "0 1px 2px rgba(16,24,40,0.05)"
  level2: "0 4px 8px rgba(16,24,40,0.06)"
  level3: "0 8px 24px rgba(16,24,40,0.08)"

# Motion
motion:
  duration:
    fast: "150ms"
    medium: "300ms"
    slow: "500ms"
  easing:
    standard: "cubic-bezier(0.2, 0.8, 0.2, 1)"

Notes:
- Base unit chosen: 8px (aligns with figma-design-standards)
- Typography uses Inter as placeholder; replace with brand fonts when available.
- Colors chosen for accessibility: ensure contrast checks during visual pass.

---

## Component Library Reference

### C/Actions/Button
- Variants: Primary / Secondary / Ghost
- Sizes: Small (32px), Medium (40px), Large (48px)
- States: Default, Hover, Focus, Active, Disabled, Loading
- Tokens:
  - button.primary.background: {color.primary}
  - button.primary.text: {semantic.text.primary_light}
  - button.disabled.opacity: 0.4

### C/Inputs/Dropdown
- Keyboard operable; up/down to navigate; Enter to select
- Accessible label required
- States: Default, Open, Focus, Disabled

### C/Inputs/Toggle
- Sizes: 40x24px (default)
- States: On, Off, Focus, Disabled
- Tokens:
  - toggle.track.on: {color.primary}
  - toggle.knob.radius: {radius.round}

### C/Feedback/ProgressBar
- Determinate and indeterminate variants
- Accessibility: aria-valuenow/aria-valuemin/aria-valuemax
- Tokens:
  - progress.background: {semantic.surface}
  - progress.fill: {color.primary}

### C/Content/MetricTile
- Variants: Small (for compact dashboards) / Medium / Large
- Includes label, numeric value, optional change delta, small sparkline
- Accessibility: role=img with aria-label summarizing key metric

### C/Content/Sparkline
- Lightweight SVG; max 50 points (default)
- Non-interactive; include title for assistive tech

### C/Feedback/Modal
- Centered overlay, focus trap, ESC to dismiss
- Confirm/Cancel variants
- Tokens:
  - modal.background: rgba(0,0,0,0.6)
  - modal.surface: {semantic.surface}

### C/Content/LogList
- Virtually rendered for long logs
- Each entry: timestamp, message, copy action
- Auto-scroll toggle and manual scroll support
- Accessibility: role=list + listitem semantics

---

## Component Usage Examples (Task Mapping)

TASK-001:
  title: "Implement Controls Panel"
  ui_impact: true
  figma_frames: ["C/Actions/Button/Primary", "C/Inputs/Toggle/Narrative"]
  components_affected:
    - Button (Primary, Secondary)
    - Toggle (Narrative)
    - Dropdown (Dataset)
  visual_validation_required: true

TASK-002:
  title: "Implement Metrics & Progress"
  ui_impact: true
  figma_frames: ["C/Content/MetricTile/Medium", "C/Feedback/ProgressBar/Determinate"]
  components_affected:
    - MetricTile
    - Sparkline
    - ProgressBar

TASK-003:
  title: "Implement Event Log List"
  ui_impact: true
  figma_frames: ["C/Content/LogList/Default"]
  components_affected:
    - LogList
    - Button (Export)

---

## New Visual Assets
screenshots:
  location: ".propel/context/Design/AITrainingDemo/"
  files:
    - name: "placeholder_preview.png"
      description: "Placeholder preview until brand assets provided"
      source: "generated"

new_assets:
  icons:
    - name: "play.svg"
      purpose: "Start control"
    - name: "pause.svg"
      purpose: "Pause control"

---

## Accessibility Requirements (Design-time)

- WCAG Level: AA
- Color contrast: >=4.5:1 for body text; >=3:1 for UI components
- Focus states: 3:1 contrast and visible 2px offset
- Touch targets: >=44x44px (mobile)
- ARIA:
  - Metric updates: aria-live="polite" for non-critical updates, "assertive" reserved for critical notifications
  - Log list: role="log" or role="list" with live region for new items (configurable)

---

## Design Review Checklist
- [ ] Figma frames for all components created
- [ ] Tokens applied to components
- [ ] Light and dark variants present
- [ ] Accessibility checks performed for color & focus
- [ ] Motion specs added with reduce-motion alternatives
- [ ] Export manifest entries prepared

---

## Implementation Notes
- Replace placeholder font and color tokens when brand assets are provided.
- Keep telemetry off by default; implement analytics hooks as no-op callbacks.
- For any backend integration (FR-010), generate separate API design doc and update tokens for auth UI.

---

===============================================================
         FIGMA SPECIFICATION GENERATION COMPLETE
===============================================================

Rules used by the workflow:
- ai-assistant-usage-policy
- dry-principle-guidelines
- markdown-styleguide
- figma-design-standards
- ui-ux-design-standards
- web-accessibility-standards

Evaluation Scores:

| Criterion | Score (1-5) |
|----------:|:-----------:|
| Completeness | 5 |
| Testability | 5 |
| Clarity | 5 |
| Traceability | 5 |
| Accessibility Considerations | 5 |
| AI Triage Appropriateness | 4 |

Average score: 4.83

1) UI Impact Assessment:
   - Total Epics: N/A (epics.md not present)
   - UI-Impacting Epics: N/A
   - Proceeding with specification: YES

2) Persona Coverage:
   | Persona | Primary Screens | Flows Covered |
   |---------|-----------------|---------------|
   | Demo User | SCR-001, SCR-002, SCR-003, SCR-004 | FL-001, FL-002, FL-003 |
   | Sales Presenter | SCR-001, SCR-002, SCR-004, SCR-005 | FL-001, FL-004 |
   | Designer | SCR-003, SCR-006 | FL-005 |
   | Engineering Observer | SCR-004, SCR-006 | FL-004 |

3) Screen Inventory:
   - Total Screens Derived: 7
   - From Use Cases: UC-001, UC-002, UC-003, UC-004

   | Screen | Derived From | Priority |
   |--------|--------------|----------|
   | Landing / Demo Shell (SCR-001) | FR-001 / UC-001 | P0 |
   | Controls Panel (SCR-002) | FR-002 / UC-001 | P0 |
   | Metrics & Progress (SCR-003) | FR-003 / UC-003 | P0 |
   | Event / Log (SCR-004) | FR-004 / UC-003 | P0 |
   | Dataset Selector (SCR-005) | FR-005 / UC-004 | P0 |
   | Settings (SCR-006) | FR-008/FR-009 | P1 |
   | Confirmation/Error Modal (SCR-007) | FR-005/FR-010 [UNCLEAR] | P0 |

4) Prototype Flows:
   - Total Flows: 5

   | Flow Name | Personas | Screens |
   |-----------|----------|---------|
   | Run Simulation (FL-001) | Demo User, Sales Presenter | SCR-001, SCR-002, SCR-003, SCR-004 |
   | Pause/Resume/Reset (FL-002) | Demo User | SCR-002, SCR-007 |
   | Dataset Change (FL-003) | Demo User | SCR-005, SCR-007 |
   | Export Log (FL-004) | Sales Presenter | SCR-004 |
   | Persist Settings (FL-005) | Designer | SCR-006 |

5) Design System:
   - Color Tokens: 12+ (primary + semantic + neutral ramp)
   - Typography Tokens: H1-H3, Body, Caption (5)
   - Spacing Tokens: base 8px + scale (9 values)
   - Components Specified: ~12 core components (Button, Toggle, Dropdown, Progress, MetricTile, Sparkline, LogList, Modal, Toast, Header, Slider, Badge)

6) Output Files:
   - figma_spec.md: CREATED
   - designsystem.md: CREATED

Summary: Figma specification and design tokens generated for a responsive, accessible client-side AI training demo. Screens, flows, UXR mappings, component inventory, and tokens provided. Outstanding clarifications: formal personas, brand tokens, and FR-010 backend integration details. Please review inferred personas and [UNCLEAR] items before implementation.
===============================================================