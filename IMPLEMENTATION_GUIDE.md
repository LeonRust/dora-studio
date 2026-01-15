# Dora Studio - Implementation Guide

> **Team Collaboration Guide** - Roadmap, phases, and task breakdown for parallel development

## Quick Links

- [ISSUES.md](ISSUES.md) - Trackable issues with dependencies
- [PRD.md](PRD.md) - Product requirements
- [ARCHITECTURE.md](ARCHITECTURE.md) - Technical architecture

---

## Executive Summary

### Issue Breakdown

| Phase | Issues | Description |
|-------|--------|-------------|
| Phase 0 | 7 | Foundation (workspace, shell, widgets) |
| Phase 1 | 7 | Core services (DoraClient, Storage, OTLP) |
| Phase 2 | 27 | 4 mini-apps (parallel development) |
| Phase 3 | 8 | AI integration (agent, tools, chat bar) |
| Phase 4 | 7 | Polish (testing, docs, release) |
| **Total** | **56** | **6 good-first-issues for newcomers** |

### Parallel Development Strategy

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TEAM ASSIGNMENT                                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Dev A ──────► Foundation + Shell                                        │
│                                                                          │
│  Dev B ──────► DoraClient + Storage (DataFusion)                        │
│                                                                          │
│  Dev C ──────► YAML Editor ◄──── uses ──── makepad-flow                 │
│                                                                          │
│  Dev D ──────► Telemetry Dashboard ◄──── uses ──── makepad-chart        │
│                                                                          │
│  Dev E ──────► Dataflow Manager + Log Viewer                            │
│                                                                          │
│  Dev F ──────► AI Agent (Claude API, tools, chat bar)                   │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

### Sprint Overview

| Sprint | Focus | Deliverables |
|--------|-------|--------------|
| Sprint 1 | Phase 0 | Workspace, shell, widgets, DoraApp trait |
| Sprint 2-3 | Phase 1 + 2 start | DoraClient, Storage, app scaffolding |
| Sprint 4-5 | Phase 2 complete | All 4 mini-apps functional |
| Sprint 6 | Phase 3 + 4 | AI integration, testing, release |

---

## External Dependencies

### Makepad Components (CRITICAL)

| Component | Repository | Purpose |
|-----------|------------|---------|
| **makepad-flow** | [github.com/mofa-org/makepad-flow](https://github.com/mofa-org/makepad-flow) | Dataflow graph visualization |
| **makepad-chart** | [github.com/mofa-org/makepad-chart](https://github.com/mofa-org/makepad-chart) | Dashboard charts (Line, Bar, Pie, etc.) |
| **makepad-widgets** | [github.com/makepad/makepad](https://github.com/makepad/makepad) | Core UI framework |

### makepad-flow Features

```
FlowCanvas Widget:
- GPU-accelerated node graph rendering
- Node dragging, connection creation
- Zoom/pan (mouse wheel, shift+drag)
- YAML import for dataflow definitions
- Node categories with colors (MaaS=Blue, TTS=Green, etc.)
- Port search and batch toggle
- Multi-selection via drag boxes
```

### makepad-chart Features

```
11 Chart Types:
- Bar (vertical/horizontal, stacked/grouped)
- Line (with area fill, stepped variants)
- Pie / Doughnut
- Scatter / Bubble
- Radar / Polar Area
- Combo (Bar + Line)
- Chord Diagram

Features:
- 28 easing functions for animations
- Gradient effects (vertical, radial, angular)
- Progressive animation for 1000+ data points
- Hover and click detection
```

---

## Development Phases

```
┌─────────────────────────────────────────────────────────────────┐
│                      DEVELOPMENT TIMELINE                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Phase 0: Foundation        ████████░░░░░░░░░░░░░░░░░░░░░░░░░░  │
│  (Workspace, deps, shell)                                        │
│                                                                  │
│  Phase 1: Core Services     ░░░░░░░░████████░░░░░░░░░░░░░░░░░░  │
│  (DoraClient, Storage)                                           │
│                                                                  │
│  Phase 2: Mini-Apps         ░░░░░░░░░░░░░░░░████████████████░░  │
│  (4 apps in parallel)                                            │
│                                                                  │
│  Phase 3: AI Integration    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░████░░  │
│  (Agent, tools, chat UI)                                         │
│                                                                  │
│  Phase 4: Polish            ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░██  │
│  (Testing, docs, release)                                        │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 0: Foundation

**Goal**: Set up workspace, dependencies, and shell application

### Tasks

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| F-01 | Create workspace structure | - | None | `Cargo.toml`, directory tree |
| F-02 | Add external dependencies | - | F-01 | `makepad-flow`, `makepad-chart` integration |
| F-03 | Create `dora-studio-shell` crate | - | F-01 | Shell binary with window |
| F-04 | Create `dora-studio-widgets` crate | - | F-01 | Shared widget library |
| F-05 | Implement sidebar navigation | - | F-03 | App switching UI |
| F-06 | Implement theme system | - | F-04 | Dark mode, color palette |
| F-07 | Define `DoraApp` trait | - | F-04 | Plugin system for mini-apps |

### Directory Structure (After Phase 0)

```
dora-studio/
├── Cargo.toml                    # Workspace manifest
├── dora-studio-shell/            # F-03
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs               # Entry point
│       ├── app.rs                # LiveRegister, LiveHook
│       └── sidebar.rs            # F-05
├── dora-studio-widgets/          # F-04
│   ├── Cargo.toml
│   └── src/
│       ├── lib.rs
│       ├── theme.rs              # F-06
│       └── app_trait.rs          # F-07
└── apps/                         # Empty, ready for Phase 2
```

### Cargo.toml (Workspace)

```toml
[workspace]
resolver = "2"
members = [
    "dora-studio-shell",
    "dora-studio-widgets",
    "dora-studio-client",
    "apps/dataflow-manager",
    "apps/yaml-editor",
    "apps/log-viewer",
    "apps/telemetry-dashboard",
]

[workspace.dependencies]
# UI Framework
makepad-widgets = { git = "https://github.com/makepad/makepad", branch = "rik" }
makepad-flow = { git = "https://github.com/mofa-org/makepad-flow" }
makepad-chart = { git = "https://github.com/mofa-org/makepad-chart" }

# Async Runtime
tokio = { version = "1", features = ["full"] }

# Serialization
serde = { version = "1", features = ["derive"] }
serde_json = "1"
serde_yaml = "0.9"

# Storage (100% Rust)
datafusion = "44"
arrow = "53"
parquet = "53"

# AI Agent
async-trait = "0.1"

# Utilities
uuid = { version = "1", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
thiserror = "1"
anyhow = "1"
tracing = "0.1"
```

---

## Phase 1: Core Services

**Goal**: Implement DoraClient and Storage layer

### Tasks

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| C-01 | Create `dora-studio-client` crate | - | F-01 | Crate structure |
| C-02 | Implement Coordinator TCP client | - | C-01 | `DoraClient` struct |
| C-03 | Implement dataflow operations | - | C-02 | `list()`, `start()`, `stop()` |
| C-04 | Implement log subscription | - | C-02 | `subscribe_logs()` streaming |
| C-05 | Set up DataFusion storage | - | C-01 | `Storage` struct |
| C-06 | Create Parquet schemas | - | C-05 | Metrics, logs, spans tables |
| C-07 | Implement OTLP receiver | - | C-01 | gRPC server for traces |

### DoraClient API

```rust
// dora-studio-client/src/client.rs

pub struct DoraClient {
    coordinator_addr: SocketAddr,
}

impl DoraClient {
    pub async fn connect(addr: &str) -> Result<Self>;

    // Dataflow operations
    pub async fn list_dataflows(&self) -> Result<Vec<DataflowEntry>>;
    pub async fn start_dataflow(&self, yaml_path: &str) -> Result<Uuid>;
    pub async fn stop_dataflow(&self, uuid: &Uuid) -> Result<()>;
    pub async fn destroy_dataflow(&self, uuid: &Uuid) -> Result<()>;
    pub async fn reload_dataflow(&self, uuid: &Uuid) -> Result<()>;

    // Node operations
    pub async fn list_nodes(&self, dataflow_id: &Uuid) -> Result<Vec<NodeEntry>>;
    pub async fn get_node_metrics(&self, node_id: &str) -> Result<NodeMetrics>;

    // Logs
    pub async fn subscribe_logs(&self) -> Result<impl Stream<Item = LogMessage>>;
    pub async fn query_logs(&self, filter: LogFilter) -> Result<Vec<LogMessage>>;
}
```

### Storage Schema

```rust
// dora-studio-client/src/storage.rs

pub struct Storage {
    ctx: SessionContext,  // DataFusion
    data_dir: PathBuf,
}

impl Storage {
    pub async fn new(data_dir: &Path) -> Result<Self>;

    // Metrics
    pub async fn insert_metrics(&self, metrics: &[NodeMetrics]) -> Result<()>;
    pub async fn query_metrics(&self, sql: &str) -> Result<RecordBatch>;

    // Logs
    pub async fn insert_logs(&self, logs: &[LogMessage]) -> Result<()>;
    pub async fn query_logs(&self, filter: LogFilter) -> Result<Vec<LogMessage>>;

    // Spans (traces)
    pub async fn insert_spans(&self, spans: &[Span]) -> Result<()>;
    pub async fn query_spans(&self, trace_id: &str) -> Result<Vec<Span>>;
}
```

---

## Phase 2: Mini-Apps (Parallel Development)

**Goal**: Implement 4 mini-apps in parallel

```
┌────────────────────────────────────────────────────────────────┐
│                    PARALLEL DEVELOPMENT                         │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Developer A          Developer B          Developer C          │
│  ┌────────────┐       ┌────────────┐       ┌────────────┐      │
│  │ Dataflow   │       │   YAML     │       │    Log     │      │
│  │ Manager    │       │   Editor   │       │   Viewer   │      │
│  │            │       │            │       │            │      │
│  │ Uses:      │       │ Uses:      │       │ Uses:      │      │
│  │ DoraClient │       │ makepad-   │       │ DataFusion │      │
│  │            │       │ flow       │       │            │      │
│  └────────────┘       └────────────┘       └────────────┘      │
│                                                                 │
│                       Developer D                               │
│                       ┌────────────┐                            │
│                       │ Telemetry  │                            │
│                       │ Dashboard  │                            │
│                       │            │                            │
│                       │ Uses:      │                            │
│                       │ makepad-   │                            │
│                       │ chart      │                            │
│                       └────────────┘                            │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### App 2A: Dataflow Manager

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| A-01 | Create app crate structure | - | F-07 | `apps/dataflow-manager/` |
| A-02 | Implement dataflow list table | - | A-01, C-03 | Table widget with status |
| A-03 | Implement start/stop actions | - | A-02 | Button handlers |
| A-04 | Implement node expansion | - | A-02 | Per-node metrics display |
| A-05 | Add refresh and auto-update | - | A-02 | Polling/subscription |
| A-06 | Add infrastructure status | - | A-01, C-02 | Coordinator/Daemon health |

**Key Widget**: Dataflow table with expandable rows

```
┌─────────────────────────────────────────────────────────────────┐
│ Dataflows                                    [Refresh] [Start]  │
├─────────────────────────────────────────────────────────────────┤
│ ▼ UUID      │ Name        │ Status   │ Nodes │ CPU  │ Mem     │
├─────────────────────────────────────────────────────────────────┤
│ ▼ a1b2c3... │ yolo-detect │ ● Running│ 4     │ 45%  │ 2.1GB   │
│   ├─ camera │             │ ● Running│       │ 12%  │ 128MB   │
│   ├─ yolo   │             │ ● Running│       │ 98%  │ 1.8GB   │
│   ├─ plot   │             │ ● Running│       │ 8%   │ 64MB    │
│   └─ sink   │             │ ● Running│       │ 2%   │ 32MB    │
│ ▶ d4e5f6... │ audio-proc  │ ○ Stopped│ 3     │ 0%   │ 0MB     │
└─────────────────────────────────────────────────────────────────┘
```

### App 2B: YAML Editor + Graph Visualizer

**Uses: makepad-flow**

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| B-01 | Create app crate structure | - | F-07 | `apps/yaml-editor/` |
| B-02 | Integrate makepad-flow | - | B-01 | FlowCanvas widget |
| B-03 | Implement YAML text editor | - | B-01 | Syntax highlighting |
| B-04 | Implement YAML → graph sync | - | B-02, B-03 | Live preview |
| B-05 | Implement graph → YAML sync | - | B-02, B-03 | Visual editing |
| B-06 | Add node inspector panel | - | B-02 | Node details sidebar |
| B-07 | Add validation feedback | - | B-03 | Error highlighting |
| B-08 | Implement file operations | - | B-03 | Open/Save/New |

**Layout**: Split view with editor and graph

```
┌─────────────────────────────────────────────────────────────────┐
│ YAML Editor                              [New] [Open] [Save]    │
├───────────────────────────┬─────────────────────────────────────┤
│                           │                                     │
│  nodes:                   │    ┌─────────┐                      │
│    - id: camera           │    │ camera  │                      │
│      path: dora-webcam    │    └────┬────┘                      │
│      outputs: [image]     │         │ image                     │
│                           │         ▼                           │
│    - id: yolo             │    ┌─────────┐                      │
│      path: dora-yolo      │    │  yolo   │                      │
│      inputs:              │    └────┬────┘                      │
│        image: camera/img  │         │ bbox                      │
│                           │         ▼                           │
│                           │    ┌─────────┐                      │
│                           │    │  plot   │                      │
│                           │    └─────────┘                      │
│                           │                                     │
├───────────────────────────┴─────────────────────────────────────┤
│ ✓ Valid                                        Nodes: 3         │
└─────────────────────────────────────────────────────────────────┘
```

### App 2C: Log Viewer

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| C-01 | Create app crate structure | - | F-07 | `apps/log-viewer/` |
| C-02 | Implement virtualized log list | - | C-01 | Handle 100K+ entries |
| C-03 | Implement level filtering | - | C-02 | DEBUG/INFO/WARN/ERROR |
| C-04 | Implement node/dataflow filter | - | C-02 | Dropdown selectors |
| C-05 | Implement text search | - | C-02 | Regex support |
| C-06 | Add real-time streaming | - | C-02, C-04 (Phase 1) | WebSocket/gRPC |
| C-07 | Add export functionality | - | C-02 | CSV/JSON export |

**Layout**: Filter bar + virtualized log table

```
┌─────────────────────────────────────────────────────────────────┐
│ Logs                                                            │
├─────────────────────────────────────────────────────────────────┤
│ [All Dataflows ▼] [All Nodes ▼] [Level: WARN+ ▼] [Search: ___] │
├─────────────────────────────────────────────────────────────────┤
│ Timestamp        │ Node   │ Level │ Message                     │
├─────────────────────────────────────────────────────────────────┤
│ 14:32:01.234     │ yolo   │ ERROR │ Failed to load model: OOM   │
│ 14:32:00.892     │ camera │ WARN  │ Frame dropped (buffer full) │
│ 14:31:59.445     │ yolo   │ INFO  │ Processing frame 1842       │
│ 14:31:58.123     │ plot   │ DEBUG │ Rendered bbox overlay       │
│ ...              │        │       │                             │
├─────────────────────────────────────────────────────────────────┤
│ Showing 1,234 of 45,678 entries                    [Export CSV] │
└─────────────────────────────────────────────────────────────────┘
```

### App 2D: Telemetry Dashboard

**Uses: makepad-chart**

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| D-01 | Create app crate structure | - | F-07 | `apps/telemetry-dashboard/` |
| D-02 | Integrate makepad-chart | - | D-01 | Chart widgets |
| D-03 | Implement metrics panel | - | D-02 | CPU, Memory, Throughput |
| D-04 | Implement time range selector | - | D-03 | 5m/15m/1h/6h/24h |
| D-05 | Implement trace timeline | - | D-01 | Waterfall/Gantt view |
| D-06 | Implement topic stats panel | - | D-01 | Message rates, bandwidth |
| D-07 | Add golden signals panel | - | D-03 | Error rate, latency P50/95/99 |

**Layout**: Multi-panel dashboard with charts

```
┌─────────────────────────────────────────────────────────────────┐
│ Telemetry                               [5m] [15m] [1h] [24h]   │
├─────────────────────────────────────────────────────────────────┤
│ ┌─────────────────────────┐ ┌─────────────────────────────────┐ │
│ │ CPU Usage               │ │ Memory Usage                    │ │
│ │     ╭──────╮            │ │         ╭────────────╮          │ │
│ │ 100%│      │            │ │ 4GB     │            │          │ │
│ │     │      ╰────        │ │         │            ╰──        │ │
│ │  50%│                   │ │ 2GB     │                       │ │
│ │     │                   │ │         │                       │ │
│ │   0%└──────────────     │ │  0GB    └──────────────         │ │
│ └─────────────────────────┘ └─────────────────────────────────┘ │
│                                                                 │
│ ┌─────────────────────────┐ ┌─────────────────────────────────┐ │
│ │ Throughput (msg/s)      │ │ Error Rate                      │ │
│ │ ████████ 1,234          │ │ ▓░░░░░░░░░ 0.2%                 │ │
│ └─────────────────────────┘ └─────────────────────────────────┘ │
│                                                                 │
│ ┌───────────────────────────────────────────────────────────┐   │
│ │ Trace Timeline                                            │   │
│ │ camera ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 12ms          │   │
│ │ yolo   ░░░░████████████████████████████████░░ 145ms       │   │
│ │ plot   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░███ 8ms          │   │
│ └───────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

## Phase 3: AI Integration

**Goal**: Add AI chat bar to each mini-app

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| AI-01 | Create `LlmClient` trait | - | C-01 | Provider abstraction |
| AI-02 | Implement Claude API client | - | AI-01 | Primary provider |
| AI-03 | Implement Ollama client | - | AI-01 | Local LLM option |
| AI-04 | Create `AgentCoordinator` | - | AI-01 | Tool dispatch |
| AI-05 | Define tools per mini-app | - | AI-04 | 20+ tools |
| AI-06 | Create `ChatBar` widget | - | F-04 | Bottom chat UI |
| AI-07 | Integrate chat bar into apps | - | AI-06, AI-04 | Per-app chat |
| AI-08 | Implement context manager | - | AI-04 | Token budget |

### AI Tools by App

```
Dataflow Manager (6 tools):
├── list_dataflows
├── start_dataflow
├── stop_dataflow
├── get_dataflow_status
├── get_node_metrics
└── restart_dataflow

YAML Editor (6 tools):
├── validate_yaml
├── explain_dataflow
├── suggest_fix
├── generate_dataflow
├── add_node
└── connect_nodes

Log Viewer (5 tools):
├── search_logs
├── analyze_logs
├── filter_logs
├── export_logs
└── count_by_level

Telemetry Dashboard (6 tools):
├── query_metrics
├── find_bottleneck
├── get_trace
├── analyze_latency
├── compare_metrics
└── get_topic_stats
```

---

## Phase 4: Polish

**Goal**: Testing, documentation, release

| ID | Task | Owner | Dependencies | Outputs |
|----|------|-------|--------------|---------|
| P-01 | Write unit tests | - | Phase 2 | Test coverage |
| P-02 | Write integration tests | - | Phase 2 | E2E scenarios |
| P-03 | Performance optimization | - | Phase 2 | 60 FPS target |
| P-04 | Accessibility review | - | Phase 2 | Keyboard nav |
| P-05 | User documentation | - | Phase 2-3 | User guide |
| P-06 | API documentation | - | Phase 1-3 | Rustdoc |
| P-07 | Release packaging | - | P-01 to P-06 | Binaries |

---

## Dependency Graph

```
                    ┌─────────────────────────────────────────┐
                    │              Phase 0                     │
                    │           (Foundation)                   │
                    │                                          │
                    │  F-01 ──► F-02                          │
                    │    │       │                             │
                    │    ▼       │                             │
                    │  F-03 ◄───┘                              │
                    │    │                                     │
                    │    ▼                                     │
                    │  F-04 ──► F-05                          │
                    │    │       │                             │
                    │    ▼       ▼                             │
                    │  F-06    F-07                           │
                    └────┬──────┬─────────────────────────────┘
                         │      │
        ┌────────────────┴──────┴────────────────┐
        │              Phase 1                    │
        │          (Core Services)                │
        │                                         │
        │  C-01 ──► C-02 ──► C-03               │
        │    │       │        │                  │
        │    ▼       ▼        ▼                  │
        │  C-05    C-04     C-07                │
        │    │                                   │
        │    ▼                                   │
        │  C-06                                  │
        └────┬─────────────────┬─────────────────┘
             │                 │
             ▼                 ▼
┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
│   App 2A       │  │   App 2B       │  │   App 2C       │  │   App 2D       │
│   Dataflow     │  │   YAML         │  │   Log          │  │   Telemetry    │
│   Manager      │  │   Editor       │  │   Viewer       │  │   Dashboard    │
│                │  │                │  │                │  │                │
│ A-01→A-02→A-03 │  │ B-01→B-02→B-04 │  │ C-01→C-02→C-03 │  │ D-01→D-02→D-03 │
│      ↓    ↓    │  │      ↓    ↓    │  │      ↓    ↓    │  │      ↓    ↓    │
│    A-04  A-05  │  │    B-03→B-05   │  │    C-04  C-05  │  │    D-04  D-05  │
│      ↓         │  │      ↓    ↓    │  │      ↓    ↓    │  │      ↓    ↓    │
│    A-06        │  │    B-06  B-07  │  │    C-06  C-07  │  │    D-06  D-07  │
│                │  │      ↓         │  │                │  │                │
│                │  │    B-08        │  │                │  │                │
└────────────────┘  └────────────────┘  └────────────────┘  └────────────────┘
        │                  │                  │                  │
        └──────────────────┴──────────────────┴──────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────────────┐
                    │              Phase 3                     │
                    │          (AI Integration)                │
                    │                                          │
                    │  AI-01 ──► AI-02                        │
                    │    │        │                            │
                    │    ▼        ▼                            │
                    │  AI-04 ◄── AI-03                        │
                    │    │                                     │
                    │    ▼                                     │
                    │  AI-05 ──► AI-06 ──► AI-07              │
                    │             │                            │
                    │             ▼                            │
                    │           AI-08                          │
                    └─────────────────────────────────────────┘
                                    │
                                    ▼
                    ┌─────────────────────────────────────────┐
                    │              Phase 4                     │
                    │              (Polish)                    │
                    │                                          │
                    │  P-01 ──► P-02 ──► P-07                 │
                    │           ↑                              │
                    │  P-03 ────┤                              │
                    │  P-04 ────┤                              │
                    │  P-05 ────┤                              │
                    │  P-06 ────┘                              │
                    └─────────────────────────────────────────┘
```

---

## Team Assignment Recommendations

### Parallel Workstreams

| Developer | Focus Area | Skills Needed |
|-----------|-----------|---------------|
| **Dev A** | Foundation + Shell | Makepad, Rust async |
| **Dev B** | DoraClient + Storage | Tokio, DataFusion, gRPC |
| **Dev C** | YAML Editor (makepad-flow) | Makepad, YAML parsing |
| **Dev D** | Telemetry Dashboard (makepad-chart) | Data visualization |
| **Dev E** | Dataflow Manager + Log Viewer | Table widgets, streaming |
| **Dev F** | AI Agent | LLM APIs, tool calling |

### Sprint Suggestion

```
Sprint 1: Foundation (Phase 0)
  - All devs collaborate on workspace setup
  - Dev A leads shell implementation

Sprint 2-3: Core + Apps (Phase 1 + 2 start)
  - Dev B: DoraClient + Storage
  - Dev C: YAML Editor
  - Dev D: Telemetry Dashboard
  - Dev E: Dataflow Manager

Sprint 4-5: Apps + Integration (Phase 2 complete)
  - Continue app development
  - Dev E adds Log Viewer
  - Dev F starts AI infrastructure

Sprint 6: AI + Polish (Phase 3 + 4)
  - Dev F: AI integration
  - Others: Testing, docs, optimization
```

---

## Quick Start Commands

```bash
# Clone workspace
git clone https://github.com/dora-rs/dora-studio.git
cd dora-studio

# Add external dependencies
git submodule add https://github.com/mofa-org/makepad-flow.git external/makepad-flow
git submodule add https://github.com/mofa-org/makepad-chart.git external/makepad-chart

# Build (after crates are created)
cargo build --release

# Run
cargo run --bin dora-studio-shell
```

---

## Success Metrics

| Metric | Target |
|--------|--------|
| Build time (incremental) | < 10s |
| Startup time | < 1s |
| Frame rate | 60 FPS |
| Memory (idle) | < 100MB |
| Log viewer (100K entries) | < 1s scroll |
| AI response time | < 3s |

---

## Next Steps

1. Review this guide with team
2. Create GitHub issues from [ISSUES.md](ISSUES.md)
3. Assign owners to tasks
4. Start Phase 0 implementation
