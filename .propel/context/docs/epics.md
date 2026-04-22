- ai-assistant-usage-policy
- dry-principle-guidelines
- iterative-development-guide
- markdown-styleguide
- software-architecture-patterns
- uml-text-code-standards
- security-standards-owasp
- performance-best-practices

# Epic - [EP_XXX]

## Epic Summary Table
Provide a table of all in-scope epics. Each epic must have a unique ID (format: EP-###), a concise action-oriented title, and a complete, comma-separated list of mapped requirement IDs from all categories (FR-, NFR-, TR-, DR-, UXR-, AIR-). Exclude unclear ([UNCLEAR]) items until clarified. Split or add epics if any single epic maps to more than ~12 requirements or mixes unrelated outcomes. Order epics by business value, then dependency priority.

| Epic ID | Epic Title | Mapped Requirement IDs |
|---------|------------|------------------------|
| EP-TECH | Project Foundation & Developer Tooling | TR-001, NFR-001, NFR-002 |
| EP-001 | Landing Shell & Navigation (Controls Surface) | FR-001, UXR-101, UXR-102, UXR-901, SCR-001, SCR-002 |
| EP-002 | Simulation Engine (Client-side Deterministic Runner) | FR-002, NFR-005, TR-002 |
| EP-003 | Metrics & Visualization (Tiles, Progress, Sparklines) | FR-003, UXR-001, UXR-002, UXR-501, SCR-003 |
| EP-004 | Event Log & Export | FR-004, SCR-004 |
| EP-005 | Dataset Presets & Run Configuration | FR-005, UXR-601, SCR-005 |
| EP-006 | Accessibility, Responsiveness & UX Quality | FR-006, NFR-003, NFR-004, UXR-201, UXR-202, UXR-301, UXR-302, TR-003, TR-004 |
| EP-007 | Persistence & Settings (localStorage) | FR-008, DR-001, DR-002, DR-003, UXR-801, SCR-006 |
| EP-008 | Telemetry & Analytics Hooks (Opt-in) | FR-009, UXR-701, SCR-006 |
| EP-009 | Optional Narrative Messaging (Deterministic NLG) | FR-007, AIR-001, AIR-002, AIR-003, TR-005 |

Notes:
1. FR-010 is marked [UNCLEAR] and moved to Backlog Refinement (not mapped).
2. EP-TECH is first because this is a green-field project (no codeanalysis.md present).
3. EP-DATA not created: data requirements are small and scoped under EP-007 (persistence). If design.md later contains >2 DR-XXX, create dedicated EP-DATA.
4. Every non-[UNCLEAR] requirement above maps to exactly one epic (traceability maintained).

## Epic Description
### EP-TECH: Project Foundation & Developer Tooling
**Business Value**: Enables all subsequent development by establishing project foundation and developer workflows.

**Description**: Create the project scaffolding, baseline architecture, developer environment, and CI/CD pipeline required to deliver the client-side demo reliably. Deliver minimal, well-documented conventions so feature teams can iterate quickly and consistently.

**UI Impact**: No (infrastructure & developer-facing)

**Screen References**: N/A

**Key Deliverables**:
- Project scaffold (React + TypeScript starter, folder conventions)
- ESLint, Prettier, accessibility linting (eslint-plugin-jsx-a11y) configured
- CI pipeline: build, test, lint, accessibility checks, Lighthouse performance job
- Dev environment instructions and local run scripts
- Performance budget baseline (measured, documented) aligned to NFR-001 / NFR-002
- Security baseline and secrets handling guidance (OWASP basics)

**Dependent EPICs**: EP-001, EP-002, EP-003, EP-004, EP-005, EP-006, EP-007, EP-008, EP-009

---

### EP-001: Landing Shell & Navigation (Controls Surface)
**Business Value**: Provides the user-facing entrypoint and control affordances required for demos and stakeholder interactions.

**Description**: Implement the landing page shell and controls area: header, brief description, primary control region, dataset selector placement, and navigation affordances. Ensure semantic structure and keyboard-accessible control focus order.

**UI Impact**: Yes

**Screen References**: SCR-001, SCR-002

**Key Deliverables**:
- Landing shell with header, description, hero region
- Controls region layout and accessible buttons (Start, Pause/Resume, Reset, Load Demo Data)
- Keyboard and focus behaviors; proper semantic sections
- Control state rules (enable/disable per simulation state)
- Documentation for control contract (events emitted)

**Dependent EPICs**: EP-TECH (foundation), EP-002 (relies on simulation engine availability)

---

### EP-002: Simulation Engine (Client-side Deterministic Runner)
**Business Value**: Implements the deterministic simulation core that produces metrics and drives the demo's core behavior.

**Description**: Build the deterministic simulation engine that responds to control commands (Start, Pause, Resume, Reset), advances epochs, and emits metric and event payloads. Engine must be deterministic (seedable PRNG) and configurable by dataset presets and speed.

**UI Impact**: No (core logic), integration with UI required

**Screen References**: N/A (feeds SCR-003, SCR-004)

**Key Deliverables**:
- Deterministic PRNG integration (seedrandom) and seeded runs
- Simulation state machine: Idle, Running, Paused, Completed
- Public API for engine commands and events (simulation_started, epoch_complete, simulation_paused, simulation_resumed, simulation_reset)
- Configurable parameters (epochs, speed, dataset noise)
- Unit tests validating determinism and state transitions

**Dependent EPICs**: EP-TECH, EP-003, EP-004, EP-005

---

### EP-003: Metrics & Visualization (Tiles, Progress, Sparklines)
**Business Value**: Delivers the primary observability surfaces so users can understand simulated training behavior quickly.

**Description**: Implement metric tiles (epoch, loss, accuracy), progress bar, and sparklines. Ensure metrics update smoothly per epoch and meet performance and responsiveness targets.

**UI Impact**: Yes

**Screen References**: SCR-003

**Key Deliverables**:
- Metric tiles with ARIA-friendly labels and aria-live behavior for critical updates
- Progress bar reflecting epoch/totalEpochs percentage
- Sparkline component (bounded points, performant rendering)
- Visual states (loading, updating, completed)
- Performance profiling to meet UXR-001 and UXR-002 acceptance criteria

**Dependent EPICs**: EP-002 (simulation data), EP-006 (accessibility & responsiveness)

---

### EP-004: Event Log & Export
**Business Value**: Provides traceability and shareable artifacts for demos and follow-ups (export log).

**Description**: Implement the chronological event/log panel that records timestamped messages from the simulation and control interactions, with copy/export capability.

**UI Impact**: Yes

**Screen References**: SCR-004

**Key Deliverables**:
- Virtualized LogList component (timestamp, message, metadata)
- Auto-scroll behavior and toggle
- Export / Copy-to-Clipboard action with accessible confirmation
- Filtering and simple search (optional/minimal)
- Integration with persistence (EP-007) to optionally persist logs

**Dependent EPICs**: EP-002 (generates events), EP-007 (persistence), EP-008 (optional telemetry)

---

### EP-005: Dataset Presets & Run Configuration
**Business Value**: Enables consistent demo scenarios and quick contextual changes for presenters and users.

**Description**: Provide dataset presets (at least three) and configuration UI; ensure changing datasets updates simulation parameters, and mid-run changes present confirmation.

**UI Impact**: Yes

**Screen References**: SCR-005

**Key Deliverables**:
- Dataset selector UI and inline presets (Small Toy, Medium, Noisy)
- Mapping presets to simulation parameters (initialLoss, initialAccuracy, noise)
- Confirmation modal when switching datasets mid-run
- Documentation for presets and expected behavior

**Dependent EPICs**: EP-002 (applies configs), EP-004 (logs the change)

---

### EP-006: Accessibility, Responsiveness & UX Quality
**Business Value**: Ensures the demo is usable by all audiences and meets legal/alignment accessibility and responsiveness commitments.

**Description**: Implement and verify keyboard accessibility, ARIA usage, color contrast, responsive breakpoints (collapse to single-column ≤480px), and general UX polish with focus states and reduced-motion support.

**UI Impact**: Yes

**Screen References**: SCR-002, SCR-003, SCR-001 (global)

**Key Deliverables**:
- WCAG AA compliance checks and fixes (contrast, focus, keyboard flows)
- ARIA live regions for dynamic metric/log announcements (aria-live usage)
- Responsive CSS and layout variants for mobile/tablet/desktop
- Accessibility test suite and keyboard-only walkthroughs documented
- Performance tuning to maintain interaction response targets

**Dependent EPICs**: EP-TECH, EP-001, EP-003, EP-004

---

### EP-007: Persistence & Settings (localStorage)
**Business Value**: Improves demo continuity and presenter convenience via optional persisted settings and logs.

**Description**: Implement user-controlled persistence (Remember settings) for dataset and UI prefs; persist simulation state when opted in; provide controls to clear persisted data and ensure privacy constraints.

**UI Impact**: Yes

**Screen References**: SCR-006

**Key Deliverables**:
- Namespaced localStorage schema for SimulationState and EventLog (DR-001, DR-002)
- Toggle "Remember settings" with clear privacy copy
- "Reset settings" action to clear persisted data
- Safeguards: only non-sensitive configuration persisted (DR-003)
- Tests for persistence and clear flows

**Dependent EPICs**: EP-001, EP-004

---

### EP-008: Telemetry & Analytics Hooks (Opt-in)
**Business Value**: Provides instrumentation hooks for analytics while preserving privacy and opt-in defaults.

**Description**: Implement documented, no-op-by-default telemetry hooks (simulation_started, simulation_paused, simulation_resumed, simulation_reset, dataset_selected, metric_report) and an opt-in telemetry toggle in settings.

**UI Impact**: Yes (settings)

**Screen References**: SCR-006

**Key Deliverables**:
- Telemetry adapter interface (no-op default)
- Opt-in settings UI and privacy copy (UXR-701)
- Example integration docs showing required callback shape and minimal payload
- Tests verifying no-op default and opt-in activation

**Dependent EPICs**: EP-TECH (for integration security guidance), EP-007 (settings)

---

### EP-009: Optional Narrative Messaging (Deterministic NLG)
**Business Value**: Improves demo storytelling by optionally providing single-sentence narrative updates without external generative models.

**Description**: Deliver an opt-in deterministic narrative system using templates (Handlebars or equivalent), with consent/privacy guardrails if external models are later integrated.

**UI Impact**: Yes (toggle + inline messages)

**Screen References**: SCR-003 (metrics), SCR-002 (toggle)

**Key Deliverables**:
- Narrative toggle and UI affordance ("Narrative Messages")
- Deterministic template engine integration (TR-005) and template set (AIR-001)
- Consent and explicit configuration screen if external models are required (AIR-002, AIR-003)
- Tests ensuring narratives are limited to one sentence and contextually accurate
- Documentation for opt-in behavior and privacy constraints

**Dependent EPICs**: EP-006 (ARIA announcements & accessibility), EP-TECH

---

## Backlog Refinement Required
List of requirements tagged [UNCLEAR] that require clarification prior to implementation. These items are intentionally excluded from epic mapping until clarified.

- FR-010 [UNCLEAR] — Backend training integration endpoints and data exports
  - Clarification questions:
    1. Is backend integration in-scope for MVP or marked as future work?
    2. If in-scope, provide API contract: endpoints, authentication, payloads (log export format), and retention/compliance requirements.
    3. Are there specific storage, encryption, or PII constraints for exported logs?
    4. Expected throughput and rate-limiting requirements for ingestion?
- Brand assets & final breakpoints [UNCLEAR]
  - Clarification questions:
    1. Provide brand tokens (colors, fonts) and any mandatory layout breakpoints to finalize designs.
- Any additional TR- or DR- requirements not present in design.md
  - Request: If there are TR-XXX or DR-XXX items omitted from design.md, please surface them for mapping (EP-DATA decision).

## Validation Checklist (Pre-Delivery)
- [x] Project Type Detected: Green-field (EP-TECH generated)
- [x] EP-TECH Included: Yes, first epic
- [x] EP-DATA Included: Not triggered; DRs mapped into EP-007
- [x] Requirement Coverage: All FR (FR-001..FR-009), NFR (NFR-001..NFR-005), TR (TR-001..TR-005), DR (DR-001..DR-003), UXR and AIR mapped (except [UNCLEAR])
- [x] No Orphans: Zero non-[UNCLEAR] requirements without an epic assignment
- [x] No Duplicates: Each requirement appears in exactly one epic
- [x] Epic Sizing: All epics sized to keep ~3–9 requirements each (<=12)
- [x] UNCLEAR Handling: FR-010 and other unclear items listed above
- [x] Priority Ordering: EP-TECH first, EP-007 (persistence) blocking EP-004 where applicable
- [x] Template Adherence: Output follows epics-template structure

## Quality Evaluation

| Criterion                      | Score (1-5) |
|-------------------------------:|:-----------:|
| Completeness                   | 5 |
| Traceability                   | 5 |
| Clarity                        | 5 |
| Priority Alignment             | 5 |
| Accessibility Considerations   | 5 |
| Testability                    | 5 |
| AI Triage Appropriateness      | 4 |
| **Average Score**              | **4.86** |

Evaluation summary:
All in-scope functional, UX, technical, and data requirements (except FR-010 [UNCLEAR]) are mapped to epics with clear deliverables and dependencies. EP-TECH is prioritized first for this green-field effort. Accessibility, performance, and privacy opt-ins are explicitly handled. FR-010 must be clarified before backend work begins.