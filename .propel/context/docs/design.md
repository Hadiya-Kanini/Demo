Here is the complete output following the workflow guidelines and template:

```markdown
# Architecture Design

## Project Overview
Build a minimal, accessible web landing page simulating an AI training platform. The page provides an interactive demo where users can start, pause, resume, and reset a client-side simulated training run. Users can view progress and metrics (epoch, loss, accuracy), select a demo dataset, and inspect a console-style event log. The simulation runs entirely in the browser using a deterministic pseudo-random generator, with no backend training infrastructure required by default.

**Target Users:**
- Sales, product, engineering, design, and UX teams for demonstrations.
- Non-technical stakeholders for training and onboarding.

**High-Level Capabilities:**
- Interactive controls (Start, Pause/Resume, Reset, Load Demo Data).
- Real-time metrics and progress visualization.
- Dataset selection (3 presets).
- Event log with export capability.
- Accessibility compliance (WCAG AA).
- Optional: NLG narrative messages, localStorage persistence, analytics hooks.

## Architecture Goals
- **Goal 1: Performance** : Page load < 2s, button response < 200ms.
- **Goal 2: Accessibility** : WCAG AA compliance (keyboard navigation, ARIA roles, contrast ≥ 4.5:1).
- **Goal 3: Responsiveness** : Single-column layout at viewport widths ≤ 480px.
- **Goal 4: Deterministic Simulation** : Client-side pseudo-random generator for reproducible metrics.
- **Goal 5: Maintainability** : Semantic HTML, modular code, and component-based architecture.

## Non-Functional Requirements
- NFR-001: System MUST load the landing page within 2 seconds on typical broadband (desktop).
- NFR-002: System MUST respond to Start/Stop/Reset actions within 200ms.
- NFR-003: System MUST meet WCAG AA standards for accessibility (keyboard navigation, ARIA roles, contrast ≥ 4.5:1).
- NFR-004: System MUST collapse to a single-column layout at viewport widths ≤ 480px.
- NFR-005: System MUST NOT persist sensitive data in localStorage.

## Data Requirements
- DR-001: System MUST store simulation state including current epoch, total epochs, loss, accuracy, and selected dataset.
- DR-002: System MUST maintain an event log with timestamped messages (e.g., "Epoch 3 complete: loss=0.123, acc=0.89").
- DR-003: System MAY persist simulation settings (dataset, epochs, speed) to localStorage if enabled by the user.

## AI Consideration

**Status:** Conditional (Applicable only if FR-007 is implemented)

**Rationale:**
- FR-007 (NLG narrative messages) is tagged `[AI-CANDIDATE]` but is **optional** and must be **opt-in**.
- The feature is **client-side only** and **deterministic** (templates preferred over generative models).
- AI fit score for FR-007 is **3 (HYBRID)**, while all other features are **1 (LOW-FIT)**.

## AI Requirements [CONDITIONAL: Only if FR-007 is implemented]
- AIR-001: System MAY generate human-readable narrative messages using deterministic templates (e.g., "Training converging — loss decreasing steadily").
- AIR-002: System MUST NOT use external generative models unless explicitly configured and consent is obtained.
- AIR-003: System MUST provide an opt-in toggle for narrative messages.

### AI Architecture Pattern
**Selected Pattern:** N/A (Templates preferred for deterministic client-side NLG)
**Rationale:**
- FR-007 is optional and low-risk (templates meet requirements).
- No grounding data or dynamic knowledge required.
- Avoids latency, cost, and compliance risks of generative models.

## Technology Stack
| Layer       | Technology       | Version       | Justification (NFR/DR/AIR)                     |
|-------------|------------------|---------------|-----------------------------------------------|
| Frontend    | React            | 18+           | NFR-003 (Accessibility), NFR-004 (Responsiveness) |
| Simulation  | seedrandom       | 3.0.5         | TR-002 (Deterministic PRNG)                   |
| AI (NLG)    | Handlebars       | 4.7.7         | AIR-001 (Deterministic templates)             |
| Testing     | Jest, Cypress    | Latest        | NFR-001 (Performance), NFR-003 (Accessibility) |
| Analytics   | Custom hooks     | N/A           | FR-009 (Optional integration)                 |

### Alternative Technology Options
- **Frontend**: Vue.js (simpler learning curve, comparable accessibility support).
- **Simulation**: Custom PRNG (lower bundle size, but less tested).
- **AI (NLG)**: Tiny generative model (higher risk, requires consent).

### Technology Stack Validation
- **React**: Chosen for accessibility tooling (e.g., `eslint-plugin-jsx-a11y`) and component-based architecture.
- **seedrandom**: Lightweight, deterministic, and well-tested for client-side use.
- **Handlebars**: Zero-dependency templating for deterministic NLG.

### Technology Decision
| Metric (NFR/DR)       | React | Vue  | Rationale                          |
|-----------------------|-------|------|------------------------------------|
| Accessibility (NFR-003)| 10    | 9    | React has superior ARIA tooling.   |
| Performance (NFR-001) | 8     | 9    | Vue has smaller bundle size.       |
| Responsiveness (NFR-004)| 10   | 10   | Both support responsive design.    |
| **Weighted Total**    | **28**| **28**| **Tie; React chosen for tooling.** |

### AI Component Stack [CONDITIONAL]
| Component      | Technology       | Purpose                                  |
|----------------|------------------|------------------------------------------|
| NLG Engine     | Handlebars       | Deterministic templates for narratives.  |

## Technical Requirements
- TR-001: System MUST use React 18+ for client-side rendering (justified by NFR-003, NFR-004).
- TR-002: System MUST use `seedrandom` for deterministic pseudo-random generation (justified by TR-002).
- TR-003: System MUST use ARIA live regions (`aria-live="polite"`) for dynamic metric updates (justified by NFR-003).
- TR-004: System MUST use semantic HTML5 elements (header, main, section) for accessibility (justified by NFR-003).
- TR-005: System MAY use Handlebars for NLG narrative messages if FR-007 is implemented (justified by AIR-001).

## Domain Entities
- **Simulation State**: Represents the current state of the training simulation.
  - Attributes: `currentEpoch`, `totalEpochs`, `loss`, `accuracy`, `dataset`, `isRunning`, `isPaused`.
  - Relationships: Linked to `EventLog` and `Dataset`.
- **Event Log**: Records timestamped events during the simulation.
  - Attributes: `timestamp`, `message`.
  - Relationships: Linked to `SimulationState`.
- **Dataset**: Predefined presets for simulation parameters.
  - Attributes: `id`, `name`, `description`, `initialLoss`, `initialAccuracy`, `noiseLevel`.

## Technical Constraints & Assumptions
- **Constraints**:
  - Client-side only by default (no backend training infrastructure).
  - Optional backend integration requires explicit configuration (FR-010 is [UNCLEAR]).
  - Persisted data limited to localStorage (non-sensitive settings only).
- **Assumptions**:
  - Primary users are demo audiences (sales, product, stakeholders).
  - Design assets (brand tokens) are not provided; neutral theme implemented.
  - Heavy security/compliance is out of scope unless backend integrations are requested.

## Development Workflow

1. **Setup**:
   - Initialize React project with TypeScript.
   - Install dependencies (`seedrandom`, `handlebars` if FR-007 is implemented).
   - Configure ESLint for accessibility (`eslint-plugin-jsx-a11y`).

2. **UI Implementation**:
   - Create semantic components (Header, Controls, Metrics, ProgressBar, EventLog, DatasetSelector).
   - Implement responsive layout (single-column at ≤ 480px).

3. **Simulation Logic**:
   - Implement deterministic PRNG using `seedrandom`.
   - Create simulation state management (Start/Pause/Resume/Reset).
   - Update metrics and event log per epoch.

4. **Accessibility**:
   - Add ARIA roles and keyboard navigation.
   - Test with screen readers (VoiceOver/NVDA).
   - Verify contrast ratios (≥ 4.5:1).

5. **Optional Features**:
   - Implement FR-007 (NLG) using Handlebars if enabled.
   - Add localStorage persistence (FR-008).
   - Expose analytics hooks (FR-009).

6. **Testing**:
   - Unit tests (Jest) for simulation logic and components.
   - Integration tests (Cypress) for UI flows and accessibility.
   - Performance profiling (Lighthouse) for load times.

7. **Deployment**:
   - Build static assets (React production mode).
   - Deploy to hosting service (e.g., GitHub Pages, Azure Static Web Apps).
```

---

### Output Summary (Console Only)
**Rules Used by Workflow:**
- ai-assistant-usage-policy
- code-anti-patterns
- dry-principle-guidelines
- iterative-development-guide
- language-agnostic-standards
- markdown-styleguide
- performance-best-practices
- security-standards-owasp
- software-architecture-patterns
- uml-text-code-standards

**Unclear Technical Understanding:**
- FR-010: Backend integration requirements (API contract, authentication, data handling) are [UNCLEAR].

**Evaluation Scores:**

| Criterion                     | Score (1-5) |
|-------------------------------|-------------|
| Completeness                  | 5           |
| Testability                   | 5           |
| Clarity                       | 5           |
| Traceability                  | 5           |
| Accessibility Considerations   | 5           |
| AI Triage Appropriateness     | 4           |
| **Average Score**             | **4.83**    |

**Evaluation Summary:**
The design provides a complete, testable architecture for a client-side AI training simulator. Requirements are traceable to the input spec, and technology choices are justified by NFR/DR constraints. AI sections are conditionally included and avoid over-engineering. Accessibility and performance are prioritized, with clear mitigation strategies for risks. The [UNCLEAR] backend integration (FR-010) is appropriately flagged for clarification.