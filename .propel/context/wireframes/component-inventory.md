File: .propel/context/wireframes/component-inventory.md
```markdown
# Component Inventory - AI Training Demo

Purpose: Complete component inventory, tokens, branding guidance, accessibility, interaction states, data contracts, and implementation notes for the high-fidelity wireframes (SCR-001..SCR-007). This file is the single source of truth for component designers and engineers building the demo.

Summary mapping
- Figma Wireframe URL: https://figma.com/file/PROJECT_ID/Wireframe-AI-Training-Demo
- Viewport used for specs: 1440 x 900
- Screens referenced: SCR-001 (Landing), SCR-002 (Controls), SCR-003 (Metrics), SCR-004 (Event Log), SCR-005 (Dataset Selector), SCR-006 (Settings), SCR-007 (Modals)

Important note
- Brand assets (logo, official color palette) and FR-010 backend contract remain outstanding. Placeholder tokens below are applied in wireframes and should be reconciled with final brand tokens when available.

1) Design tokens (applied across wireframes)
- Token naming follows BEM-like semantic tokens and common design system conventions.

Colors (light theme)
- color-primary: #2563EB (blue-600) — primary CTA, interactive highlights
- color-primary-600: #1749C6 (for active/pressed states)
- color-accent: #10B981 (green-500) — success indicators
- color-warning: #F59E0B (amber-500)
- color-danger: #EF4444 (red-500)
- color-surface: #FFFFFF — primary surface
- color-muted-surface: #F3F4F6 — secondary surfaces / panels
- color-text-primary: #0F172A (neutral-900)
- color-text-secondary: #374151 (neutral-700)
- color-text-muted: #6B7280 (neutral-500)
- color-border: #E5E7EB (neutral-200)
- color-overlay: rgba(15,23,42,0.6) — modal/backdrop
- focus-ring: #93C5FD (blue-300)
- toast-success-bg: #ECFDF5
- toast-error-bg: #FEF2F2

Dark theme (mapped tokens)
- color-surface-dark: #0B1220
- color-muted-surface-dark: #0F1724
- color-text-primary-dark: #E6EEF8
- color-border-dark: #14202B
- overlay-dark: rgba(2,6,23,0.7)

Contrast guidance
- All text and interactive elements must meet at least WCAG AA contrast ratios (4.5:1 for normal text, 3:1 for large text). Token choices in the wireframes were validated for AA; final brand colors must be validated and adjusted if needed.

Typography
- font-family-base: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial
- type-scale (desktop):
  - header-1: 32px / 40px / 600
  - header-2: 24px / 32px / 600
  - body-large: 18px / 28px / 400
  - body-regular: 16px / 24px / 400
  - label: 14px / 20px / 500
  - caption: 12px / 16px / 400
- line-height tokens follow accessibility: maintain >=1.25 for body text.

Spacing scale (px)
- spacing-xxs: 4
- spacing-xs: 8
- spacing-sm: 12
- spacing-md: 16
- spacing-lg: 24
- spacing-xl: 32
- spacing-xxl: 48

Radii & elevation
- radius-sm: 4px
- radius-md: 8px
- radius-lg: 12px
- elevation-1: 0 1px 2px rgba(2,6,23,0.06)
- elevation-2: 0 6px 18px rgba(2,6,23,0.08)

Breakpoints
- bp-mobile: 320px
- bp-small: 480px
- bp-tablet: 768px
- bp-desktop: 1024px
- bp-wide: 1440px

Motion & accessibility
- reduce-motion: respect user OS preference; avoid non-essential animations if prefers-reduced-motion is set.
- focus-outline-width: 2px (use focus-ring color with 3:1 contrast on surface)
- touch-target-minimum: 44x44px

2) Global behaviors & conventions
- Interaction latency target: UI actions should respond within 200ms (UXR-002).
- Live regions: metrics and logs use aria-live="polite" (or "assertive" for critical errors) and must be off by default for automated tests; toggled only while simulation is running.
- Persistency: "Remember settings" toggles store only UI preferences in localStorage after explicit opt-in (UXR-701). No telemetry is collected unless opted in (UXR-701).
- Security: sanitize any string inserted into logs or HTML. No untrusted HTML injection; display plain text with whitespace and line-break preservation. Use DOM APIs that escape by default.

3) Component specifications
Each component section below includes: Purpose, Anatomy, Variants, Props/Data contract, Interaction states, ARIA/keyboard guidance, Responsive behavior, Implementation notes, Testing notes.

A. Header with settings (SCR-001)
- Purpose: Global navigation and access to settings, dataset selector, and telemetry/remember toggles.
- Anatomy:
  - Left: Brand logo (link to root)
  - Center (optional): page title / breadcrumb
  - Right: Controls: Dataset dropdown (compact), Settings icon (opens SCR-006), user controls (if any), theme toggle
- Variants:
  - compact (mobile) — collapses center title, moves controls into menu
  - wide — full layout, dataset visible
- Props / Data:
  - activeDataset: { id, name }
  - onOpenSettings(): event
  - onDatasetChange(datasetId)
- Interaction states: hover, active, focus for all interactive items; disabled for dataset dropdown if operation locks dataset (e.g., in critical modal).
- ARIA / Keyboard:
  - header uses <header> semantic element
  - dataset dropdown: role="button" with aria-haspopup="listbox" and aria-expanded
  - keyboard: Tab to traverse, Enter/Space to open dropdown, Arrow keys within dropdown to navigate options, Esc to close.
- Responsive:
  - at <= bp-small dataset and settings collapse into hamburger menu
- Implementation notes:
  - lazy-load logo assets; fallback to text logo if asset missing
  - defer heavy event listeners; minimal DOM footprint
- Testing:
  - keyboard-only navigation, screen reader label correctness

B. Hero / Landing shell with CTA (SCR-001)
- Purpose: Entry area that introduces demo and includes primary CTA to start the simulation.
- Anatomy:
  - Heading, subheading, primary CTA (Start demo), secondary CTA (Load sample dataset / docs)
  - Visual area for showing demo summary or snapshot
- Variants:
  - condensed (small viewport) — stacked layout, single-column
  - expanded (desktop) — two-column with hero visual
- Props:
  - onStart(): starts simulation
  - primaryCTAState: enabled/disabled/loading
- States:
  - default, hover, active, focus, loading (show spinner/disable)
- ARIA:
  - primary CTA is a real button element (role="button" implicit); aria-pressed not needed for single-action
  - hero visual should have accessible alt text or be aria-hidden if decorative
- Implementation:
  - debounce rapid repeated clicks to meet performance target
  - prefetch minimal simulation code on hover/first focus to reduce cold-start latency
- Accessibility:
  - headings use H1/H2 semantic order; ensure skip links available from header

C. Controls Panel (SCR-002)
- Purpose: Primary interaction area for controlling the simulation: Start, Pause, Resume, Reset, Load Data; speed and epochs inputs.
- Anatomy:
  - Start / Pause / Resume toggle group (primary)
  - Reset (secondary/danger) with confirm modal (SCR-007)
  - Load Data (dropdown/modal)
  - Numeric inputs: epochs (min/max validation), simulation speed slider/select
  - Compact status indicator (running/paused/completed)
- Variants:
  - minimal (mobile): stacked buttons, controls condensed
  - full (desktop): horizontal grouping, inline inputs
- Props / API:
  - onStart(), onPause(), onResume(), onReset(), onLoadDataset(datasetId)
  - state: {status: 'idle'|'running'|'paused'|'complete', currentEpoch, totalEpochs}
  - controlsDisabled: boolean
- Interaction & validation:
  - Inputs validate client-side (epochs >= 1 && <= 10000)
  - Start button disabled when invalid inputs
  - Reset must open confirm modal if run has progressed past epoch 0
- ARIA & keyboard:
  - Buttons are native <button> elements
  - Role="group" for the control group with accessible label
  - Keyboard: Enter/Space activates, Left/Right arrow for slider
- Accessibility:
  - Ensure focus order is logical; visual focus visible via focus-ring token
- Implementation:
  - Use debounce/throttle for slider updates to avoid rendering heavy metric updates on every frame
  - Do not perform heavy computations on UI thread — use requestAnimationFrame or web worker for simulation loop
- Testing:
  - Validate disabled state, input validation messages and aria-invalid when invalid

D. Metric Tiles (Epoch, Loss, Accuracy) + ProgressBar + Sparkline (SCR-003)
- Purpose: Show live-updating scalar metrics and progress visualization.
- Anatomy:
  - MetricTile (label, primary value, delta or secondary value, small caption)
  - ProgressBar (percentage, accessible label)
  - Sparkline (small history chart, concise)
- Variants:
  - compact (mobile): stacked tiles with mini-progress
  - expanded (desktop): horizontal tiles with full sparkline
- Props / Data contract:
  - metrics: {
      epoch: { current: number, total: number },
      loss: { value: number, format: string },
      accuracy: { value: number, format: string },
      history: [{timestamp:number, loss:number, accuracy:number}, ...]
    }
  - onMetricClick(metricId) (optional drill-in)
- Interaction & states:
  - loading state: placeholders with aria-busy
  - highlight state: metric exceeding thresholds (color-accent/danger)
  - error state: show — when metric feed fails; use toast and tile-level error hint
- Accessibility:
  - Metric tiles: aria-live="polite" region to announce significant changes (configurable)
  - ProgressBar: role="progressbar", aria-valuemin="0", aria-valuemax={totalEpochs}, aria-valuenow={current}
  - Sparkline: use aria-hidden="true" and provide textual summary for screen readers (e.g., "Loss decreasing from 0.98 to 0.12 over 50 epochs")
- Keyboard:
  - Tiles focusable for drill-in; Enter/Space to open more detail
- Implementation:
  - Throttle updates to ARIA live regions to avoid overwhelming SRs (e.g., announce every N seconds or per epoch change)
  - Sparklines should be canvas or lightweight SVG; avoid re-rendering whole chart per datapoint—use incremental updates
- Performance:
  - Keep history limited to a configurable window (e.g., last 200 points) to bound memory and rendering
- Testing:
  - Verify aria attributes; test with screen readers for live updates prioritization

E. Event / Log Panel (virtualized) with Export/Copy (SCR-004)
- Purpose: Streamed timestamped events, ability to export or copy logs.
- Anatomy:
  - Filter controls (severity, search)
  - Virtualized list of events: timestamp, level, message
  - Export button (download JSON/CSV), Copy to clipboard
- Variants:
  - collapsed / expanded, docked to right or bottom (responsive)
- Data contract:
  - events: [{ id, timestampISO, level: 'info'|'warn'|'error', message, meta: {} }, ...]
  - onExport(format): returns file blob
- Interaction & states:
  - Streaming mode: auto-scroll when user at bottom; if user scrolled up, stop auto-scroll and show “Jump to live” control
  - Virtualization: render windowed rows for large log sizes to maintain performance
  - Copy/Export success/failure: show toast with accessible message
- ARIA / keyboard:
  - Use role="log" and aria-live for new entries when at live position (aria-live="polite")
  - Each log item is a listitem with readable timestamp and level
  - Keyboard: arrow keys to traverse virtualized rows; provide skip links or shortcut to jump to top/bottom
- Security:
  - Escape all user-content in log messages; do not render raw HTML
- Implementation:
  - Prefer virtualization libraries (react-window / fixed-list) or custom windowing
  - Batch append events every 100-200ms to reduce layout thrashing
  - Export: generate file on worker thread if large
- Testing:
  - Performance under simulated high-frequency event streams; correctness of copy/export formats

F. Dataset selector (dropdown + confirm modal) (SCR-005)
- Purpose: Allow user to load different datasets (presets) and confirm mid-run switches.
- Anatomy:
  - Dropdown with dataset list (id, name, brief description)
  - Confirm modal: explains mid-run consequences, buttons Confirm / Cancel
- Props / Data:
  - datasets: [{id, name, description, sampleSize}]
  - onSelect(datasetId)
  - confirmOnSwitchWhileRunning: boolean
- Interaction:
  - If simulation running and user selects a new dataset, show confirm modal (SCR-007) with clear consequences (progress lost or persisted)
- ARIA / keyboard:
  - Dropdown uses listbox pattern; modal must trap focus and have aria-modal="true"
- Accessibility:
  - Ensure the modal has role="dialog" and aria-labelledby / aria-describedby
- Implementation:
  - Confirm modal should be reusable component used by Reset and other destructive actions
- Testing:
  - Focus trap, dismiss on Esc, return focus to the invoking control

G. Settings (Remember settings, Telemetry opt-in) & Reset settings (SCR-006)
- Purpose: Persist preferences, manage telemetry opt-in, and reset to defaults.
- Anatomy:
  - Toggles: Remember settings, Telemetry opt-in (default: off)
  - Buttons: Reset settings (opens confirm modal), Save/Close
  - Short explanations inline for privacy choices
- Props:
  - settings: { rememberSettings: boolean, telemetryOptIn: boolean, theme: 'light'|'dark' }
  - onSave(settings), onReset()
- Accessibility:
  - Toggles are checkbox elements with clear labels and helper text
- Persistence:
  - Only persist to localStorage if rememberSettings === true. On reset, clear stored keys (explicit user action).
- Security & privacy:
  - Telemetry opt-in only toggles collection of anonymous usage metrics; no PII collection. Document minimal telemetry fields required (e.g., event name, timestamp, minor device metadata) — finalize after FR-010.
- Testing:
  - Verify storage only when opted in and reset clears data

H. Modal dialogs (confirmations, errors) (SCR-007)
- Purpose: Reusable modal for confirmation and error messaging.
- Anatomy:
  - Header (title), body (message), actions (primary/secondary)
  - Optional: checkbox to persist decision (e.g., "Don't ask again")
- Variants:
  - confirmation, destructive, informational, error
- Props:
  - isOpen: boolean
  - title: string
  - body: string | ReactNode (render plain text by default)
  - primaryAction: { label, onClick, type: 'primary'|'danger' }
  - secondaryAction: { label, onClick }
  - onClose()
- Accessibility:
  - role="dialog", aria-modal="true", aria-labelledby and aria-describedby required
  - Focus trap; focus initial on primary or first focusable element
  - Return focus to opener when closed
  - Close on Esc; clicking backdrop closes only for non-critical modals (configurable)
- Keyboard:
  - Enter activates primary (if not a destructive action without explicit confirm)
- Implementation:
  - Render modal in portal root to avoid overflow clipping
  - Ensure modality prevents background interactions (inert=true or aria-hidden on main content)
- Testing:
  - Focus trap and return-focus behavior; screen reader announcement order

I. Toasts and Inline feedback
- Purpose: Temporary feedback for non-blocking events (success, error, info).
- Anatomy:
  - Small container with message and optional action (e.g., Undo)
  - Stacking behavior at top-right or bottom depending on layout and theme
- Variants:
  - success, error, info, warning
- Props:
  - id, message, type, durationMs, action?: { label, handler }
- Accessibility:
  - role="status" or aria-live="polite"; do not auto-focus; allow screen reader to announce
  - Duration: default 4000ms; longer for error states that require attention
  - Respect prefers-reduced-motion and prefer to keep visible longer for SR users if needed
- Keyboard:
  - Provide focusable action buttons; toasts themselves should not steal focus
- Implementation:
  - Deduplicate identical toasts within a short window to avoid spam
  - Use CSS transform for enter/exit animations (hardware-accelerated) and respect reduced-motion
- Testing:
  - Announcements verify with SR; durations and action handlers

J. Common primitives & utility components
- Button
  - Variants: primary, secondary, danger, ghost, icon
  - Props: disabled, loading (show spinner, aria-busy), size: small/medium/large
  - Accessibility: aria-disabled for non-interactive states; use native <button>
  - Keyboard: standard behavior; support keyboard shortcuts where applicable
- Input / Number Input / Slider
  - Numeric input: min/max validation, aria-invalid when invalid, helper text
  - Slider: role="slider", aria-valuenow/min/max, keyboard left/right arrows
- Toggle / Checkbox
  - Native input[type=checkbox], with accessible label and description
- Dropdown / Listbox
  - Implement WAI-ARIA listbox pattern; support search/typeahead for long lists
- Icon button
  - Provide accessible label via aria-label; do not rely on icon-only visuals

4) Interaction patterns & micro-interactions
- Start -> Run flow:
  - Start triggers simulation loop; controls become Pause; metrics update.
  - During run, dataset selection opens a modal that requires explicit confirmation to switch (UXR-601).
  - When run completes, show a completion toast and update controls to allow restarting or resetting.
- Pause / Resume:
  - Pause stops simulation loop without clearing state; ARIA live region announces "Simulation paused at epoch X".
- Reset:
  - Reset is destructive; open confirmation modal. On confirm, stop loop, clear metrics and logs, reset epoch to 0.
- Export log:
  - Export triggers file download; show toast with link to file; copy shows in clipboard and toasts success/failure.

5) Data contracts & API suggestions (client-side)
- Metric update event (from simulation loop)
  - { type: 'metric', payload: { epoch: number, loss: number, accuracy: number, timestamp: ISOString } }
- Log event
  - { type: 'log', payload: { id: uuid, timestamp: ISOString, level: 'info'|'warn'|'error', message: string, meta?: {} } }
- Control events (UI -> simulation)
  - { type: 'command', payload: { action: 'start'|'pause'|'resume'|'reset'|'loadDataset', params?: {} } }
- Export contract:
  - Export API returns Promise<Blob> for JSON/CSV
- Note: FR-010 backend contract is outstanding; current wireframes assume an in-browser deterministic simulation (no backend required). If a backend is introduced, add explicit authentication and CORS rules and sanitize all inbound/outbound payloads.

6) Accessibility checklist (applies to all components)
- Semantic HTML: prefer native elements (<button>, <input>, <header>, <main>, <nav>) over divs with role when possible
- Focus management: define logical tab order; trap and restore focus on modals; visible focus indicators
- ARIA: only use when native semantics are insufficient; ensure aria-labels, roles, and statuses are correct
- Contrast: all text and interactive controls meet WCAG AA
- Live regions: use sparingly and throttle updates to avoid verbosity for assistive tech users
- Keyboard-only: full functionality must be reachable and operable via keyboard
- Reduced motion: respect prefers-reduced-motion

7) Performance & security considerations
- Performance:
  - Virtualize long lists (Event log)
  - Batch DOM updates for metric/log streams; use requestAnimationFrame and throttling
  - Limit in-memory history for metrics and logs (configurable window)
  - Lazy-load non-critical assets (logos, heavy libs)
- Security:
  - Escape/sanitize all user-provided or external text before rendering
  - Use secure download patterns for exports and avoid embedding executable content
  - Opt-in telemetry only and document collected fields; avoid PII by design

8) Developer guidance & implementation notes
- Component library compatibility: design tokens are framework-agnostic. Provide mapping to whichever design system (React/Stencil/Vue) is used — tokens are keys to be implemented as CSS variables or system tokens.
- The modal and toast systems should be implemented once and reused across the app.
- Unit & integration tests: cover keyboard interactions, ARIA attributes, focus management, and persistence behavior.
- End-to-end tests: verify user flows FL-001..FL-005 (Start run, Pause/Resume/Reset, Dataset switch, Export Log, Persist settings)
- File locations used by wireframes:
  - Hi-Fi HTML examples: ./.propel/context/wireframes/Hi-Fi/wireframe-SCR-00X-*.html
  - This component inventory: ./.propel/context/wireframes/component-inventory.md

9) Examples & usage notes
- MetricTile usage:
  - Input: { label: 'Loss', value: 0.12, delta: -0.02, status: 'improving' }
  - Visual: value large, delta smaller with aria-hidden icon; provide textual change for screen readers
- ProgressBar usage:
  - role="progressbar" aria-valuemin="0" aria-valuemax={totalEpochs} aria-valuenow={currentEpoch} aria-label="Training progress"

10) Outstanding items & acceptance criteria
- Finalize brand assets and color palette (FR-010). Acceptance: wireframes updated with final tokens and contrast validated.
- Backend contract (if introduced): define telemetry schema and API endpoints. Acceptance: contract doc + security review.
- Performance benchmarks: confirm cold-load time < 2s and action latency <= 200ms on target desktop environments (UXR-001/UXR-002).

11) Contact & ownership
- Primary contact for design decisions: Senior Product Designer (see figma_spec metadata)
- Engineering contact: (to be assigned)
- Handoff artifacts:
  - Figma file: https://figma.com/file/PROJECT_ID/Wireframe-AI-Training-Demo
  - Hi-Fi HTML wireframes: ./.propel/context/wireframes/Hi-Fi/

End of component inventory.
```