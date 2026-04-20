# Design Reference

> **Note**: This template should only be used for User Stories and Tasks that have **UI impact**.
> Backend-only, API-only, or data processing tasks do not require design references.

## UI Impact Assessment
**Has UI Changes**: [x] Yes [ ] No
- This feature introduces a responsive, accessible interactive landing page (AI Training Demo) with new UI components and layout changes. Complete the sections below.

## User Story Design Context
**Story ID**: US-001
**Story Title**: Interactive AI Training Demo Landing Page
**UI Impact Type**: New UI

### Design Source References
**Choose applicable reference type:**
- **Figma Project**: [https://figma.com/file/PROJECT_ID/ai-training-demo] (primary design source; frames referenced below)
- **Design Images**: .propel/context/Design/images/ (fallback/static images used where Figma frames are not available)
- **Design System**: .propel/context/docs/designsystem.md (this document)
- **Brand Guidelines**: .propel/context/docs/brand_guidelines.md (if/when available)

### Screen-to-Design Mappings
**Option A: Figma Frame Mappings**
| Screen/Feature | Figma Frame ID | Direct Link | Description | Implementation Priority |
|---------------|---------------|-------------|-------------|----------------------|
| Landing / Hero (SCR-001) | node-id=1:10 | https://figma.com/file/PROJECT_ID?node-id=1:10 | Top-level landing area: title, short description, primary CTA, preview image | High |
| Controls / Simulation Panel (SCR-002) | node-id=1:25 | https://figma.com/file/PROJECT_ID?node-id=1:25 | Start, Pause/Resume, Reset, Load Data buttons and speed/epoch controls | High |
| Progress & Metrics (SCR-003) | node-id=2:12 | https://figma.com/file/PROJECT_ID?node-id=2:12 | Progress bar, epoch counter, loss/accuracy metric tiles | High |
| Event / Console Log Panel (SCR-004) | node-id=2:30 | https://figma.com/file/PROJECT_ID?node-id=2:30 | Scrollable console-style event log with timestamped entries | High |
| Dataset Selector & Settings (SCR-005) | node-id=3:5 | https://figma.com/file/PROJECT_ID?node-id=3:5 | Dropdown for demo datasets, toggles for "Remember settings" and telemetry opt-in | Medium |
| Accessibility / Confirmation Modal (SCR-006) | node-id=3:20 | https://figma.com/file/PROJECT_ID?node-id=3:20 | Modals: confirmation for dataset change mid-run, Reset confirmation, accessibility help | Medium |

**Option B: Design Image References**
| Screen/Feature | Image File | Image Path | Description | Implementation Priority |
|---------------|------------|------------|-------------|----------------------|
| Landing / Hero | preview.png | .propel/context/Design/images/preview.png | Landing page preview used for marketing preview | Medium |
| Controls Panel | controls_states.png | .propel/context/Design/images/controls_states.png | Button states and control layout | High |

### Design Tokens
```yaml
# Only include tokens that are used in this specific UI implementation
colors:
  primary:
    value: "#0066FF"
    usage: "Primary CTA, interactive accents"
    affected_components: ["Button", "Links", "ProgressBar"]
  secondary:
    value: "#00A3A3"
    usage: "Secondary actions, metric accents"
    affected_components: ["MetricCard", "Badges"]
  background:
    value: "#FFFFFF"
    usage: "Page background (light theme)"
    affected_components: ["Page", "Cards"]
  surface:
    value: "#F5F7FA"
    usage: "Card and panel backgrounds"
    affected_components: ["Cards", "Panels", "EventLog"]
  text_primary:
    value: "#0B1220"
    usage: "Primary text"
    affected_components: ["Headers", "BodyText"]
  text_secondary:
    value: "#5B6670"
    usage: "Secondary text and labels"
    affected_components: ["HelperText", "Timestamps"]
  success:
    value: "#0F9D58"
    usage: "Positive metric (accuracy), success states"
    affected_components: ["MetricCard", "Badges"]
  warning:
    value: "#F59E0B"
    usage: "Warning/attention states"
    affected_components: ["MetricCard", "Badges"]
  danger:
    value: "#E53E3E"
    usage: "Error states"
    affected_components: ["Validation", "EventLog Error Rows"]

typography:
  heading1:
    family: "Inter"
    size: "28px"
    weight: "600"
    line-height: "36px"
    used_in: ["Page Headers", "Hero Title"]
  heading2:
    family: "Inter"
    size: "20px"
    weight: "600"
    line-height: "28px"
    used_in: ["Section Titles", "Modal Titles"]
  body:
    family: "Inter"
    size: "14px"
    weight: "400"
    line-height: "20px"
    used_in: ["Body Text", "Metric Labels"]
  caption:
    family: "Inter"
    size: "12px"
    weight: "400"
    line-height: "16px"
    used_in: ["Timestamps", "Helper Text"]

spacing:
  base: "8px"
  scale: [8px, 12px, 16px, 24px, 32px, 40px]
  affected_layouts: ["Form spacing", "Card padding", "Control gaps"]

radii:
  small: "4px"
  medium: "8px"
  large: "12px"
  affected_components: ["Buttons", "Cards", "Inputs"]

shadows:
  card:
    value: "0 2px 8px rgba(11,18,32,0.06)"
    usage: "Cards and panels"
  focus:
    value: "0 0 0 3px rgba(0,102,255,0.12)"
    usage: "Focus states for keyboard navigation"

breakpoints:
  mobile: 375
  tablet: 768
  desktop: 1440
```

### Component References
**Option A: Figma Component References**
| Component Name | Figma Component | Code Location | UI Changes Required |
|---------------|-----------------|---------------|-------------------|
| Button | node-id=C:100 | src/components/Button.tsx | Variants: primary, secondary, outline; add loading & disabled states |
| IconButton | node-id=C:101 | src/components/IconButton.tsx | Add accessible labels and focus ring; icon-only sizing tokens |
| Toggle / Switch | node-id=C:110 | src/components/Toggle.tsx | Telemetry opt-in toggle with ARIA switch role |
| ProgressBar | node-id=C:130 | src/components/ProgressBar.tsx | Determinate progress with animated transitions |
| MetricCard | node-id=C:140 | src/components/MetricCard.tsx | New compact layout for epoch/metric tiles |
| DatasetSelector (Dropdown) | node-id=C:150 | src/components/DatasetSelector.tsx | Add keyboard navigation, persistent selection option |
| EventLogPanel | node-id=C:160 | src/components/EventLogPanel.tsx | Virtualized scroll, timestamped entries, filter by level |
| Modal (Confirm) | node-id=C:170 | src/components/Modal.tsx | Confirmation patterns for destructive actions |
| Header | node-id=C:180 | src/components/Header.tsx | Responsive header with global CTA |

**Option B: Image-Based Component References**
| Component Name | Reference Image | Code Location | UI Changes Required |
|---------------|-----------------|---------------|-------------------|
| Button states | components/button_states.png | src/components/Button.tsx | Implement hover/active/disabled visuals |
| Event Log rows | components/eventlog_states.png | src/components/EventLogPanel.tsx | Add severity color markers and compact rows |

### New Visual Assets
```yaml
screenshots:
  location: ".propel/context/Design/US-001/"
  files:
    - name: "before_after.png"
      description: "Comparison of current static marketing page vs new interactive demo UI"
      source: "figma_frame: node-id=1:10"
    - name: "controls_states.png"
      description: "Button and control states (default/hover/active/disabled/loading)"
      source: "figma_frame: node-id=C:100"

new_assets:
  icons:
    - name: "icon_play.svg"
      source: "figma_export: node-id=I:201"
      purpose: "Start simulation"
    - name: "icon_pause.svg"
      source: "figma_export: node-id=I:202"
      purpose: "Pause/Resume"
    - name: "icon_reset.svg"
      source: "figma_export: node-id=I:203"
      purpose: "Reset run"
    - name: "icon_dataset.svg"
      source: "figma_export: node-id=I:210"
      purpose: "Dataset selector"
  images:
    - name: "hero_banner.jpg"
      source: "figma_export: node-id=IMG:300"
      requirements: "1920x600px, optimized for web (JPEG/WEBP), <200KB encouraged"
```

### Task Design Mapping
```yaml
# Only include tasks that have UI implementation
TASK-001:
  title: "Implement Landing UI"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=1:10"]  # Landing / Hero
  components_affected:
    - Header
    - Button (primary CTA)
    - Hero image
  visual_validation_required: true

TASK-002:
  title: "Implement Simulation Controls"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=1:25"]  # Controls Panel
  components_affected:
    - Button (primary/secondary/outline)
    - Toggle (telemetry)
    - Input (epoch count)
  visual_validation_required: true

TASK-003:
  title: "Metrics & Progress Visualization"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=2:12"]  # Progress & Metrics
  components_affected:
    - ProgressBar
    - MetricCard
  visual_validation_required: true

TASK-004:
  title: "Event Log Panel"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=2:30"]  # Event / Console Log
  components_affected:
    - EventLogPanel
  visual_validation_required: true

TASK-005:
  title: "Dataset Selector & Settings"
  ui_impact: true
  visual_references:
    figma_frames: ["node-id=3:5"]  # Dataset Selector
  components_affected:
    - DatasetSelector
    - Modal (confirmation)
  visual_validation_required: true

TASK-006:
  title: "Accessibility & Visual QA"
  ui_impact: true
  visual_references:
    # visual validation uses screenshots from multiple breakpoints
    design_images: [".propel/context/Design/US-001/before_after.png"]
  components_affected:
    - All interactive components updated above
  visual_validation_required: true

TASK-999:
  title: "Telemetry / Analytics Hook (Optional)"
  ui_impact: false
  # Backend integration only (telemetry) — UI has toggle only; hooks are backend work
  note: "UI includes opt-in toggle; analytics implementation is backend-only"
```

### Visual Validation Criteria
```typescript
// Only apply visual validation for UI-impacting tasks
const requiresVisualValidation = task.ui_impact === true;

if (requiresVisualValidation) {
  const visualValidation = {
    screenshotComparison: {
      maxDifference: "5%",
      breakpoints: [375, 768, 1440]
    },
    componentValidation: {
      colorAccuracy: true,
      spacingAccuracy: true,
      typographyMatch: true
    }
  };
}
```

### Implementation Scenarios

#### For New UI Components
```yaml
new_components:
  - name: "ProgressBar"
    figma_reference: "node-id=C:130"
    file_location: "src/components/ProgressBar.tsx"
    design_specifications:
      width: "100% container"
      height: "12px"
      border_radius: "8px"
      track_color: "#E6EEF8"
      fill_color: "{colors.primary.value}"
      animation: "smooth transition on value change (200ms)"
      states: ["empty", "in-progress", "complete", "indeterminate"]

  - name: "MetricCard"
    figma_reference: "node-id=C:140"
    file_location: "src/components/MetricCard.tsx"
    design_specifications:
      size: "compact: 120x80px; responsive full-width on mobile"
      padding: "16px"
      border_radius: "8px"
      states: ["default", "highlighted", "muted"]
      elements: ["label", "value", "delta_indicator", "sparkline_optional"]

  - name: "EventLogPanel"
    figma_reference: "node-id=C:160"
    file_location: "src/components/EventLogPanel.tsx"
    design_specifications:
      height: "320px on desktop, 240px on tablet, auto on mobile"
      row_height: "40px (compact)"
      virtualization: "true for long logs"
      row_elements: ["timestamp", "level_dot", "message"]
      filters: ["All", "Info", "Warning", "Error"]

  - name: "DatasetSelector"
    figma_reference: "node-id=C:150"
    file_location: "src/components/DatasetSelector.tsx"
    design_specifications:
      width: "auto with max-width 320px"
      dropdown_item_height: "44px"
      keyboard_support: "arrow navigation, enter to select, esc to close"
      accessibility: "aria-expanded, aria-controls, role=listbox"
```

#### For UI Enhancements
```yaml
ui_enhancements:
  existing_component: "Button"
  changes_required:
    - "Add loading state with spinner (centered on label)"
    - "Update hover animation to 140ms ease"
    - "Add disabled state styling and aria-disabled attribute"
    - "Ensure focus ring uses 'focus' shadow token"
  figma_reference: "node-id=C:100-updated"
```

#### For Backend/API Tasks (No Design Needed)
```yaml
backend_task:
  ui_impact: false
  design_references: "Not applicable - Analytics/Telemetry backend implementation only"
  validation_type: "Unit tests, API contract tests"
```

### Accessibility Requirements
- **WCAG Level**: AA (applies to all modified and new UI elements)
- **Color Contrast**: Minimum 4.5:1 for body text; 3:1 for large text (>=18pt/24px or bold >=14pt)
  - Validate only new/modified UI elements and tokens in colors section
- **Focus States**: Visible focus ring for all interactive controls (use focus shadow token)
- **Keyboard Navigation**: Full keyboard operability for controls, dataset_selector, and modals (tab order, arrow navigation for lists)
- **ARIA**:
  - ProgressBar: role="progressbar", aria-valuenow/aria-valuemin/aria-valuemax
  - EventLog: aria-live="polite" for new entries when log is visible; provide toggle to disable announcements
  - Toggles/Switches: use aria-checked and role="switch"
  - Buttons: include aria-labels where icon-only
- **Screen Reader**: Provide short, non-verbose update messages for metric changes (e.g., "Epoch 3 complete. Loss 0.42.")

### Design Review Checklist
**Complete only if ui_impact: true**
- [ ] Figma frames reviewed for all UI changes
- [ ] Design tokens extracted for affected components
- [ ] Component specifications documented
- [ ] Visual validation criteria defined
- [ ] Responsive behavior specified
- [ ] Accessibility requirements noted

**Skip if ui_impact: false**
- No design review required for backend/API tasks

## PRIMARY OUTPUT (Reference - already generated)
The Figma Design Specification - AI Training Demo (figma_spec.md) has been generated and should be used as the authoritative screen inventory and UX requirements reference for implementation. Ensure component names, breakpoints, and tokens used here match the figma_spec.md output.