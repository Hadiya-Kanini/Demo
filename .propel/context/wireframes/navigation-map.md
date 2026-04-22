File: .propel/context/wireframes/navigation-map.md
```markdown
# Navigation Map - AI Training Demo

Version: 1.0  
Figma Wireframe (reference): https://figma.com/file/PROJECT_ID/Wireframe-AI-Training-Demo  
Generated screens: SCR-001..SCR-007 (7 screens)  
Viewport baseline: 1440 x 900 (responsive)

This navigation map complements the Hi-Fi wireframes and information architecture. It defines routes, navigation hierarchy, transitions, ARIA/focus behavior, keyboard shortcuts, deep-linking, and component specifications tied to design tokens and UX requirements (UXR-*). Use this as a single-source reference for implementation and QA.

--------------------------------------------------------------------------------
1. Navigation Overview (hierarchy & routes)
--------------------------------------------------------------------------------

Primary Shell (single-page app / progressive enhancement):
- Route: / (SCR-001 Landing / Demo Shell)
  - Purpose: Entry point; header, hero, CTA, controls preview, dataset selector
- Route: /controls (SCR-002 Controls Panel)
  - Purpose: Full controls workspace; start/pause/resume/reset, speed & epoch inputs
- Route: /metrics (SCR-003 Metrics & Progress)
  - Purpose: Expanded metrics view; epoch, loss, accuracy, progress, sparkline
- Route: /logs (SCR-004 Event / Log Panel)
  - Purpose: Virtualized log viewer and export tools
- Route: /dataset (SCR-005 Dataset Selector)
  - Purpose: Dataset management and confirm modal on mid-run switch
- Route: /settings (SCR-006 Settings / Persist & Telemetry)
  - Purpose: Toggles for "Remember settings", telemetry opt-in, reset settings
- Overlays / Modal Route (SCR-007 Modal)
  - Pattern: overlay or route /modal/:type (confirm/error)
  - Purpose: Confirmation and error dialogs; reusable modal component

Routing notes:
- All core routes are optional client-side routes — the demo can run as a single view with internal panels. Use pushState for route changes so back/forward navigation is consistent with user expectations.
- Deep links:
  - /?dataset=NAME sets selected dataset on load.
  - /?runState=paused|running|complete can be used to restore UI state for demos (non-authoritative; client-only).
- Route guards:
  - Dataset change when runState=running triggers SCR-007 confirm modal before navigating.

--------------------------------------------------------------------------------
2. Primary Navigation Flows & Screen Transitions
--------------------------------------------------------------------------------

Flows (from Information Architecture):
- FL-001 Run Simulation
  - / -> /controls (Start) -> /metrics (live updates) -> /logs -> /metrics (complete)
- FL-002 Pause / Resume / Reset
  - /controls (Pause) -> (Resume) -> /controls (Reset -> SCR-007 confirm) -> /
- FL-003 Dataset Change
  - /dataset -> SCR-007 (confirm if running) -> (apply -> reset run) -> /controls or /
- FL-004 Export Log
  - /logs -> Export -> toast success -> remain on /logs
- FL-005 Persist Settings
  - /settings -> Toggle "Remember settings" -> store in localStorage (when enabled)

State transitions and side-effects:
- Starting a run: runState: idle -> running. Enable Pause, disable Start, lock dataset changes until paused or confirmed to switch.
- Pausing a run: running -> paused. Resume available.
- Reset: any state -> idle after confirmation; clears metrics and logs unless flagged to persist for analysis.
- Run complete: running -> complete. Show final metrics and a "Download Report" CTA.

--------------------------------------------------------------------------------
3. Keyboard & Shortcut Map
--------------------------------------------------------------------------------

Global (when focus is within demo app):
- Space: Toggle Start / Pause (context-sensitive). Only when focus is not inside a text input.
- S: Start (when idle)
- P: Pause (when running)
- R: Resume (when paused) or Reset (with confirmation flow if held with Shift+R)
- Shift+R: Reset (immediate keyboard-triggered open of reset confirmation modal)
- E: Open Logs panel (/logs)
- D: Open Dataset selector (/dataset)
- ?: Open keyboard help overlay (a11y help)
- Esc: Close modal or overlay, collapse dropdowns, or clear focus from interactive panel
Accessibility note: Shortcuts must be discoverable (keyboard help overlay) and not override browser or assistive-tech expected defaults. Provide ability to disable shortcuts (Settings).

Focus guidance for shortcuts:
- When a shortcut triggers a modal, move focus to the modal's first focusable control.
- When closing a modal, return focus to the element that triggered it.

--------------------------------------------------------------------------------
4. Focus Management & ARIA Landmarks
--------------------------------------------------------------------------------

Document landmarks:
- header: role="banner"
- main content: role="main"
- nav (if present for top-level actions): role="navigation"
- complementary controls panel: role="complementary" with aria-label="Simulation controls"
- status/metrics region: role="region" aria-live="polite" aria-atomic="false" aria-label="Training metrics updates"
- event/log list: role="log" aria-live="polite" aria-atomic="false"
- toasts: role="status" aria-live="polite" (non-interruptive; stacked but announced sequentially)

Modal behavior (SCR-007):
- Modal role="dialog" aria-modal="true" aria-labelledby="[modal-title-id]" aria-describedby="[modal-desc-id]"
- Trap focus within modal while open. Return focus to invoker after close.
- Initial focus: primary action button in confirm dialogs; first invalid input for error dialogs.

ARIA Live & announcements:
- Metrics updates (epoch, loss, accuracy): announce succinct summary every epoch via aria-live region when visible (e.g., "Epoch 5 complete. Loss 0.342, Accuracy 78%.").
- Log additions: log region with role="log" will be appended; allow screen reader users to opt-out of automatic announcements in settings (to reduce noise).
- Start/Stop/Complete: announce run state changes via toasts + aria-live.

Focus order:
- Logical DOM order should follow reading order: header -> hero -> primary controls -> metrics -> logs -> settings
- Ensure skip-links or "Skip to controls" for keyboard users.

--------------------------------------------------------------------------------
5. Component Inventory & Specifications
--------------------------------------------------------------------------------

Each component below lists anatomy, states, interactions, ARIA roles, keyboard support, accessibility checks, and performance notes.

1) Header
- Anatomy: left: brand/logo (link to /), center: page title/optional breadcrumb, right: dataset selector trigger, settings button (opens /settings)
- States: default, compact (sticky on scroll), mobile collapsed
- Interactions: logo navigates to /, settings opens modal/panel
- ARIA: role="banner", brand link labeled "AI Training Demo home"
- Keyboard: Tab focus sequence; Enter/Space activates links
- Notes: Keep header height <= 64px on desktop; use sticky header for persistent controls.

2) Hero / Landing Shell (SCR-001)
- Anatomy: headline, subhead, primary CTA (Start Demo), secondary: "View Controls" or "Load Dataset", hero illustration (decorative)
- States: default, narrow viewport stacked
- ARIA: decorative images aria-hidden="true"; CTA with aria-describedby for contextual hint
- Notes: CTA should be prominent and reachable in first viewport (UXR-101)

3) Controls Panel (SCR-002)
- Anatomy: Start / Pause / Resume / Reset / Load Data, numeric inputs for epochs/speed, confirm buttons
- States: idle, running, paused, disabled (invalid input)
- Interactions:
  - Start: validates inputs, starts run
  - Pause: suspends run loop
  - Resume: resumes
  - Reset: opens SCR-007 confirm modal
  - Load Data: opens dataset selector
- ARIA:
  - Container role="complementary" aria-label="Simulation controls"
  - Buttons use aria-pressed for toggle-like states where appropriate
- Keyboard:
  - Buttons focusable; Enter/Space to activate
  - Inputs: Arrow keys to adjust numeric steppers
  - Shortcuts (see section 3)
- Performance: debounce rapid input changes; don't rebuild logs or metrics on every keystroke.

4) Metric Tile (SCR-003)
- Anatomy: label (Epoch/Loss/Accuracy), numeric value, small delta or unit, optional sparkline thumbnail
- States: normal, updating (pulse animation), highlight on new best accuracy
- ARIA:
  - role="status" within metrics region or contained in a parent aria-live region
  - Provide aria-label summarizing value and unit
- Interaction: click to expand historical sparkline (optional)
- Accessibility: ensure contrast >= WCAG AA for all text and iconography.

5) ProgressBar + Sparkline (SCR-003)
- Anatomy: progress track, progress fill, percentage badge, sparkline SVG canvas
- States: determinate (progress value), indeterminate (loading)
- ARIA:
  - progressbar role with aria-valuemin="0", aria-valuemax="100", aria-valuenow set to percent complete
  - sparkline should be accessible: include <title> and aria-hidden for decorative small visuals; provide a textual alternative in metrics region for full data summaries
- Performance: keep sparkline sampling to reasonable points (e.g., max 200 points) to avoid expensive redraws.

6) Event / Log Panel (SCR-004)
- Anatomy: virtualized list, timestamp, severity icon, message, filters, export & copy buttons
- States: empty, streaming, filtered, export-success, export-error
- ARIA:
  - role="log" aria-live="polite" aria-atomic="false"
  - Export/Copy controls must be announced with aria-describedby referencing result toast
- Keyboard:
  - Arrow keys to move within log if focusable; Page Up/Down for rapid navigation
  - Export via keyboard: Enter/Space on button
- Performance:
  - Virtualization required for large logs; keep DOM nodes limited (UXR-001/002)
  - Export should stream chunked data to avoid memory spikes for long logs.

7) Dataset Selector (SCR-005)
- Anatomy: dropdown trigger, preset list, confirm modal when switching mid-run
- States: closed, open, selected, confirm-required
- ARIA:
  - button aria-haspopup="listbox" aria-expanded
  - listbox with role="listbox" and options role="option"
- Interactions:
  - Selecting a dataset while runState=running opens SCR-007 confirm modal with details about resetting run.
- Keyboard:
  - Arrow keys navigate options, Enter selects, Esc closes
- Security: sanitize dataset names and metadata before rendering.

8) Settings / Persist & Telemetry (SCR-006)
- Anatomy: "Remember settings" toggle, "Telemetry opt-in" toggle, Reset settings button
- States: enabled/disabled, confirmation on reset
- ARIA:
  - toggles have role="switch" aria-checked values and descriptive labels
- Persistence:
  - Only persist when "Remember settings" is enabled (UXR-801)
  - Telemetry default: off (UXR-701)
- Privacy: display short privacy note linking to telemetry policies before enabling.

9) Modal (SCR-007)
- Anatomy: title, description, primary action, secondary action (cancel), optional details expand
- States: confirm, error, info
- ARIA:
  - role="dialog" aria-modal="true"
- Keyboard:
  - Esc closes (if allowed), Tab cycles, Enter activates primary
- Focus: initial focus to primary action; return to invoker on close.

10) Toasts & Inline Feedback
- Anatomy: list of non-blocking messages; types: success/info/warn/error
- ARIA:
  - role="status" aria-live="polite"; ensure no aggressive interruptive behavior
- Lifecycle:
  - Auto-dismiss after a11y-appropriate duration (see design tokens below), but persistent on hover or focus.
- Security:
  - Do not include sensitive data in toasts.

--------------------------------------------------------------------------------
6. Design Tokens & Branding
--------------------------------------------------------------------------------

Tokens are applied from the design system (designsystem.md). Use tokens as single source-of-truth.

Color (semantic tokens):
- color-bg: #FFFFFF (light), color-bg-900: #0B1220 (dark)
- color-surface: #F7FAFC
- color-primary: #0066FF
- color-primary-600: #0050CC
- color-accent: #00C2A8
- color-success: #16A34A
- color-warning: #F59E0B
- color-danger: #DC2626
- color-text-primary: #111827
- color-text-secondary: #6B7280
Accessibility:
- Ensure contrast ratios >= 4.5:1 for body text and >=3:1 for large text (WCAG AA).

Typography:
- font-family-base: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial
- type-scale:
  - h1: 28px / 36px
  - h2: 22px / 28px
  - body: 16px / 24px
  - caption: 12px / 16px
- line-heights: default 1.4–1.6 as appropriate

Spacing & sizing:
- spacing-unit: 8px
- radii:
  - radius-sm: 4px
  - radius-md: 8px
  - radius-lg: 12px
- touch target min: 44x44px (UXR-302)

Elevation & shadows:
- elevation-sm: 0 1px 2px rgba(16, 24, 40, 0.05)
- elevation-md: 0 4px 12px rgba(16, 24, 40, 0.08)

Motion:
- motion-duration-fast: 120ms
- motion-duration-medium: 240ms
- motion-ease: cubic-bezier(0.2, 0.8, 0.2, 1)

Timing for UI elements:
- toast-duration-default: 4s (extend if focused or with important action)
- input-debounce: 100ms (for UI responsiveness without excess reflow)
- aria-live debounce: avoid flooding SRs — group updates per epoch (prefer 500ms grouping if epoch updates are rapid)

Branding notes:
- Brand assets not fully provided (FR-010 flagged UNCLEAR). Use placeholder logo with accessible name "AI Training Demo" until brand deliverables are available.
- Provide light/dark token variants and switch preference default to system.

--------------------------------------------------------------------------------
7. Accessibility & Security Considerations
--------------------------------------------------------------------------------

Accessibility:
- WCAG 2.2 AA baseline coverage for all screens (UXR-201).
- ARIA live regions for metrics and logs (UXR-202); allow user control to reduce verbosity.
- Focus visible states across all interactive elements. Respect reduced-motion preference (prefers-reduced-motion).
- Provide keyboard-only flows for all interactions and shortcut discoverability.

Security & Privacy:
- Telemetry opt-in default: off. Telemetry consent flow and policy link required (UXR-701).
- Exported logs: sanitize or redaction hooks to remove personally identifiable information before download.
- No external network calls required for deterministic in-browser simulation; any optional backend contracts (FR-010) must follow OWASP guidelines; surface failure states in SCR-007 modal.
- Avoid storing sensitive data in localStorage; if storing settings, encrypt if policy requires. Prefer localStorage only for non-sensitive preferences.

--------------------------------------------------------------------------------
8. Performance & Implementation Notes
--------------------------------------------------------------------------------

- Critical rendering path: load core UI assets <= 2s (UXR-001). Use code-splitting to lazy-load non-critical panels (logs, large analytics charts).
- Virtualize logs (SCR-004) and limit DOM nodes. Use requestAnimationFrame for UI updates tied to simulated run ticks.
- Batch metric updates: update DOM at a constrained cadence (e.g., 10–30Hz max) and group updates per epoch to avoid layout thrash (UXR-002).
- Sparkline rendering: use canvas or lightweight SVG; reduce point count for long runs.
- Avoid long synchronous CPU tasks on main thread; use Web Workers for heavy log exports if needed.
- Telemetry: do not emit telemetry without opt-in; allow anonymized opt-in if backend contract exists.

--------------------------------------------------------------------------------
9. Traceability (screens -> UXRs / FRs / Flows)
--------------------------------------------------------------------------------

- SCR-001 Landing -> UXR-101, UXR-001, FL-001
- SCR-002 Controls -> UXR-002, UXR-301, FL-001, FL-002
- SCR-003 Metrics -> UXR-202, UXR-501, FL-001
- SCR-004 Logs -> UXR-202, UXR-001, FL-004
- SCR-005 Dataset Selector -> UXR-601, FL-003
- SCR-006 Settings -> UXR-701, UXR-801, FL-005
- SCR-007 Modal -> UXR-601, UXR-901

Reference FRs: FR-001..FR-010 mapping exists in the requirements spec; FR-010 (backend contract/brand assets) remains outstanding.

--------------------------------------------------------------------------------
10. Implementation Checklist (deliverables & links)
--------------------------------------------------------------------------------

Wireframe HTML (Hi-Fi) files (generated):
- SCR-001 Landing: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-001-landing.html
- SCR-002 Controls: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-002-controls.html
- SCR-003 Metrics: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-003-metrics.html
- SCR-004 Log: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-004-log.html
- SCR-005 Dataset Selector: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-005-dataset-selector.html
- SCR-006 Settings: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-006-settings.html
- SCR-007 Modal: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-007-modal.html

Figma reference:
- https://figma.com/file/PROJECT_ID/Wireframe-AI-Training-Demo

Pre-launch checklist:
- Confirm brand assets and token overrides (FR-010)
- Accessibility audit (WCAG AA) and keyboard-only walkthrough
- Performance test cold-load and run simulation cadence (UXR-001, UXR-002)
- Privacy review for telemetry and export flows

--------------------------------------------------------------------------------
End of navigation map
--------------------------------------------------------------------------------
```