# Design Modelling

## UML Models Overview
This document provides a unified set of textual UML models for the Client-Side Demo Landing Page described in the requirements. It includes: a System Context (PlantUML), Component Architecture (Mermaid), Deployment Architecture (PlantUML), Data Flow (PlantUML), Logical Data Model / ERD (Mermaid), and one Mermaid sequence diagram per use case (UC-001..UC-004). Diagrams map directly to the use cases and design decisions in the project spec to ensure traceability and implementation-ready views.

## Architectural Views

### System Context Diagram
```plantuml
@startuml System Context
left to right direction
skinparam packageStyle rectangle

actor "Demo User" as User
actor "Product Owner / Presenter" as Presenter
actor "Analytics Service (optional)" as Analytics

rectangle "Client-Side Demo Landing Page (Browser)" as Client {
  component "UI / Controls" as UI
  component "Simulation Engine (client-side)" as Sim
  component "Event Log / Persistence (LocalStorage)" as Log
}

User --> UI : Interact (Start, Pause, Reset, Select Dataset)
Presenter --> UI : Demonstrate flows
UI --> Sim : Start / Pause / Resume / Reset commands
Sim --> Log : Append epoch events
UI --> Log : View / Export logs
UI ..> Analytics : Emit telemetry (optional)

@enduml
```

### Component Architecture Diagram
```mermaid
flowchart LR
    subgraph Frontend["Presentation Layer"]
        UI[UI / Controls\n(React) - Render + ARIA]
        Metrics[Metrics & Visualizations\n(Sparklines, Tiles)]
        LogPanel[Event Log Panel\n(Export/Copy)]
        DatasetSelector[Dataset Selector\n(3 Presets)]
    end

    subgraph Backend["Business / Simulation Layer (Client)"]
        SimEngine[Simulation Engine\n(seedrandom) - Deterministic PRNG]
        NLG[Optional NLG (Handlebars)\nTemplates - Opt-in]
        Telemetry[Telemetry Adapter\n(no-op by default)]
    end

    subgraph Data["Persistence & Storage"]
        LocalStore[(localStorage)] 
    end

    UI -->|User actions| SimEngine
    UI --> Metrics
    UI --> LogPanel
    DatasetSelector --> SimEngine
    SimEngine -->|epoch events| LogPanel
    SimEngine -->|persist settings| LocalStore
    SimEngine -->|telemetry events| Telemetry
    NLG --> UI
    Telemetry -->|optional HTTP| Analytics[(Analytics Service)]
    
    classDef actor fill:#add8e6,stroke:#000;
    classDef core fill:#90ee90,stroke:#000;
    classDef data fill:#ffffe0,stroke:#000;
    class UI,Metrics,LogPanel,DatasetSelector class actor;
    class SimEngine,NLG,Telemetry class core;
    class LocalStore class data;
```

### Deployment Architecture Diagram
```plantuml
@startuml
left to right direction
skinparam packageStyle rectangle

node "User Device (Browser)" as Browser {
  component "Client Single-Page App\n(Static JS/CSS/HTML)" as SPA
}

cloud "CDN / Static Hosting" {
  node "Hosting" as Hosting
}

package "Optional Integrations" {
  database "Analytics Endpoint" as Analytics
  node "Backend Training API (optional)" as BackendAPI
}

SPA --> Hosting : GET static assets (HTML, JS, CSS)
Browser --> SPA : Runs simulation client-side
SPA --> Analytics : POST telemetry (optional / configured)
SPA --> BackendAPI : POST logs or export (optional; requires auth)

' Hub-and-spoke representation for shared services
cloud "Cloud Landing Zone" {
  node "Shared Services (Hub)" as Hub
  node "Workloads (Spokes)" as Spokes
  Hub ..> Spokes : Identity, Logging, Monitoring
}

@enduml
```

### Data Flow Diagram
```plantuml
@startuml
!define EXTERNAL component
!define PROCESS rectangle
!define DATASTORE database

EXTERNAL "Demo User (Browser)" as User
PROCESS "UI / Controls" as UI
PROCESS "Simulation Engine" as Sim
DATASTORE "Event Log (localStorage)" as Log
EXTERNAL "Analytics Service (optional)" as Analytics
DATASTORE "Dataset Presets (client-bundled)" as Datasets

User -> UI : Click Start / Pause / Reset / Select Dataset
UI -> Sim : Start / Control commands
Sim -> Datasets : Read preset parameters (startLoss, noise)
Sim -> Log : Append epoch event (timestamp, epoch, loss, acc)
Sim -> UI : Update metrics (epoch, loss, accuracy)
UI -> User : Render updated metrics & progress
UI -> Log : Export / Copy log request
UI -> Analytics : Emit telemetry event (simulation_started, metric_report) [optional]
Log -->|persist| Log : localStorage writes/reads

@enduml
```

### Logical Data Model (ERD)
```mermaid
erDiagram
    SIMULATION_STATE {
        string id PK "session id"
        int currentEpoch
        int totalEpochs
        float loss
        float accuracy
        string dataset
        boolean isRunning
        boolean isPaused
        datetime updatedAt
    }

    EVENT_LOG {
        string id PK "entry id"
        string simulationId FK "SIMULATION_STATE.id"
        datetime timestamp
        string message
        json metadata
    }

    DATASET {
        string id PK
        string name
        string description
        float initialLoss
        float initialAccuracy
        float noiseLevel
    }

    SIMULATION_STATE ||--o{ EVENT_LOG : "has events"
    SIMULATION_STATE }o--|| DATASET : "uses"
```

### Use Case Sequence Diagrams

> **Note**: Create one sequence diagram for each use case (UC-XXX) defined in `.propel/context/docs/spec.md` or `.propel/context/docs/codeanalysis.md`. Each sequence diagram details the dynamic message flow and timing for its corresponding use case. Do NOT duplicate the use case diagrams (those remain in spec.md/codeanalysis.md only).

#### UC-001: Start Simulation
**Source**: [spec.md#UC-001](.propel/context/docs/spec.md#UC-001)

```mermaid
sequenceDiagram
    participant DemoUser as Demo User
    participant UI as Client UI
    participant Sim as Simulation Engine
    participant Log as Event Log (LocalStorage)
    participant Telemetry as Telemetry Adapter

    Note over DemoUser,Log: UC-001 - Start Simulation

    DemoUser->>UI: Click "Start"
    UI->>Sim: initialize(state: currentEpoch=0, dataset)
    Sim-->>Log: append("Simulation started", timestamp)
    Sim->>Telemetry: emit(simulation_started, payload)
    Sim->>Sim: begin epoch loop
    Sim->>UI: update(metrics: epoch=1, loss, accuracy)
    UI-->>DemoUser: render(metrics, progress)
    Sim->>Log: append("Epoch 1 complete: loss=..., acc=...", timestamp)
    alt reaches completion
        Sim->>UI: notify("Simulation complete")
        Sim-->>Telemetry: emit(simulation_completed)
    end
```

#### UC-002: Control Simulation (Pause / Resume / Reset)
**Source**: [spec.md#UC-002](.propel/context/docs/spec.md#UC-002)

```mermaid
sequenceDiagram
    participant DemoUser as Demo User
    participant UI as Client UI
    participant Sim as Simulation Engine
    participant Log as Event Log (LocalStorage)

    Note over DemoUser,Sim: UC-002 - Pause / Resume / Reset

    DemoUser->>UI: Click "Pause"
    UI->>Sim: command(pause)
    Sim-->>Log: append("Simulation paused by user", timestamp)
    Sim--xUI: halt epoch updates
    UI-->>DemoUser: show paused state

    DemoUser->>UI: Click "Resume"
    UI->>Sim: command(resume)
    Sim-->>Log: append("Simulation resumed", timestamp)
    Sim->>UI: resume epoch updates

    DemoUser->>UI: Click "Reset"
    UI->>Sim: command(reset)
    Sim-->>Log: append("Simulation reset by user", timestamp)
    Sim->>UI: set metrics to baseline
    UI-->>DemoUser: render(baseline metrics)
```

#### UC-003: View Metrics & Logs
**Source**: [spec.md#UC-003](.propel/context/docs/spec.md#UC-003)

```mermaid
sequenceDiagram
    participant DemoUser as Demo User
    participant UI as Client UI
    participant Sim as Simulation Engine
    participant Log as Event Log (LocalStorage)

    Note over DemoUser,Log: UC-003 - View Metrics & Logs

    Sim->>UI: push(metrics update)
    UI->>Log: read(latest entries)
    UI-->>DemoUser: render(metrics, sparkline, log entries)
    DemoUser->>UI: filter logs / export
    UI->>Log: read(filter)
    UI-->>DemoUser: show filtered entries
    UI->>Log: export(copy to clipboard / download)
```

#### UC-004: Select Demo Dataset
**Source**: [spec.md#UC-004](.propel/context/docs/spec.md#UC-004)

```mermaid
sequenceDiagram
    participant DemoUser as Demo User
    participant UI as Client UI
    participant Sim as Simulation Engine
    participant DStore as Dataset Presets

    Note over DemoUser,Sim: UC-004 - Select Demo Dataset

    DemoUser->>UI: open dataset selector
    UI->>DStore: read(presets)
    DStore-->>UI: return(presets)
    DemoUser->>UI: select("Medium")
    UI->>Sim: update(config: dataset="Medium")
    alt Simulation Running
        UI->>DemoUser: confirm("Switching dataset will reset run")
        DemoUser-->>UI: confirmYes
        UI->>Sim: apply config + reset or schedule
    else Idle
        UI->>Sim: apply config
    end
    UI-->>DemoUser: show selected dataset
```

## Arch Content
The models above are aligned to the spec's constraints: client-side-first, deterministic simulation, optional telemetry and optional opt-in NLG. Component names are consistent across diagrams (UI, Simulation Engine, Event Log, Telemetry). Sequence diagrams link to source use cases for traceability. The ERD models the minimal persisted state (session state, event log, dataset presets). Deployment shows a static-hosting-first approach with optional integrations gated by explicit configuration.

---

Output Summary (Console Only)

- Rules Used by Workflow:
  - ai-assistant-usage-policy
  - dry-principle-guidelines
  - markdown-styleguide
  - uml-text-code-standards
  - iterative-development-guide
  - security-standards-owasp
  - performance-best-practices
  - software-architecture-patterns

- Use Cases Processed:
  - UC-001
  - UC-002
  - UC-003
  - UC-004

- Evaluation Scores:

| Criterion                     | Score (1-5) |
|------------------------------:|:-----------:|
| Completeness                   | 5           |
| Testability                    | 5           |
| Clarity                        | 5           |
| Traceability                   | 5           |
| Accessibility Considerations   | 5           |
| AI Triage Appropriateness      | 4           |
| **Average Score**              | **4.83**    |

- Evaluation Summary:
The generated models are complete, traceable to use cases, and aligned with the design constraints (client-side deterministic simulation). Accessibility and performance are emphasized; AI features are optional and opt-in. Backend integration (FR-010) remains flagged for clarification.