# Dora Studio - Trackable Issues

> **GitHub-Ready Issues** - Copy these to create GitHub issues with proper labels and dependencies

## Labels Reference

| Label | Color | Description |
|-------|-------|-------------|
| `phase-0` | `#0E8A16` | Foundation phase |
| `phase-1` | `#1D76DB` | Core services phase |
| `phase-2` | `#5319E7` | Mini-apps phase |
| `phase-3` | `#FBCA04` | AI integration phase |
| `phase-4` | `#D93F0B` | Polish phase |
| `app-dataflow` | `#C2E0C6` | Dataflow Manager app |
| `app-yaml` | `#BFD4F2` | YAML Editor app |
| `app-logs` | `#FEF2C0` | Log Viewer app |
| `app-telemetry` | `#F9D0C4` | Telemetry Dashboard app |
| `component-client` | `#E6E6E6` | DoraClient component |
| `component-storage` | `#E6E6E6` | Storage component |
| `component-ai` | `#D4C5F9` | AI Agent component |
| `priority-high` | `#B60205` | High priority |
| `priority-medium` | `#FBCA04` | Medium priority |
| `good-first-issue` | `#7057FF` | Good for newcomers |

---

## Phase 0: Foundation

### F-01: Create workspace structure

**Labels**: `phase-0`, `priority-high`
**Dependencies**: None
**Blocks**: F-02, F-03, F-04

#### Description

Set up the Cargo workspace with all crate directories.

#### Acceptance Criteria

- [ ] Root `Cargo.toml` with workspace members
- [ ] Directory structure created:
  ```
  dora-studio/
  ├── dora-studio-shell/
  ├── dora-studio-widgets/
  ├── dora-studio-client/
  └── apps/
      ├── dataflow-manager/
      ├── yaml-editor/
      ├── log-viewer/
      └── telemetry-dashboard/
  ```
- [ ] Each crate has minimal `Cargo.toml` and `src/lib.rs` (or `main.rs`)
- [ ] `cargo check` passes on empty workspace

#### Files to Create

```
Cargo.toml
dora-studio-shell/Cargo.toml
dora-studio-shell/src/main.rs
dora-studio-widgets/Cargo.toml
dora-studio-widgets/src/lib.rs
dora-studio-client/Cargo.toml
dora-studio-client/src/lib.rs
apps/dataflow-manager/Cargo.toml
apps/dataflow-manager/src/lib.rs
apps/yaml-editor/Cargo.toml
apps/yaml-editor/src/lib.rs
apps/log-viewer/Cargo.toml
apps/log-viewer/src/lib.rs
apps/telemetry-dashboard/Cargo.toml
apps/telemetry-dashboard/src/lib.rs
```

---

### F-02: Add external dependencies

**Labels**: `phase-0`, `priority-high`
**Dependencies**: F-01
**Blocks**: F-03, F-04, B-02, D-02

#### Description

Configure workspace dependencies including makepad-flow and makepad-chart.

#### Acceptance Criteria

- [ ] `makepad-widgets` added to workspace dependencies
- [ ] `makepad-flow` added (for dataflow visualization)
- [ ] `makepad-chart` added (for dashboard charts)
- [ ] All other deps from Cargo.toml scaffold added
- [ ] `cargo fetch` succeeds

#### Cargo.toml Updates

```toml
[workspace.dependencies]
# UI Framework
makepad-widgets = { git = "https://github.com/makepad/makepad", branch = "rik" }
makepad-flow = { git = "https://github.com/mofa-org/makepad-flow" }
makepad-chart = { git = "https://github.com/mofa-org/makepad-chart" }

# Async Runtime
tokio = { version = "1", features = ["full"] }

# Storage (100% Rust)
datafusion = "44"
arrow = "53"
parquet = "53"

# Serialization
serde = { version = "1", features = ["derive"] }
serde_json = "1"
serde_yaml = "0.9"
```

---

### F-03: Create dora-studio-shell crate

**Labels**: `phase-0`, `priority-high`
**Dependencies**: F-01, F-02
**Blocks**: F-05

#### Description

Create the main shell application that hosts all mini-apps.

#### Acceptance Criteria

- [ ] Binary crate that launches Makepad window
- [ ] Implements `LiveRegister` to register all widgets
- [ ] Basic app structure with `App` struct
- [ ] Window displays with title "Dora Studio"
- [ ] Dark theme background

#### Key Code

```rust
// src/main.rs
use makepad_widgets::*;

live_design! {
    use link::theme::*;
    use link::widgets::*;

    App = {{App}} {
        ui: <Root> {
            <Window> {
                window: { title: "Dora Studio" }
                body = <View> {
                    flow: Down
                    // Will add sidebar + content area
                }
            }
        }
    }
}

#[derive(Live, LiveHook)]
pub struct App {
    #[live] ui: WidgetRef,
}

impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        self.ui.handle_event(cx, event, &mut Scope::empty());
    }
}

app_main!(App);
```

---

### F-04: Create dora-studio-widgets crate

**Labels**: `phase-0`, `priority-medium`
**Dependencies**: F-01
**Blocks**: F-05, F-06, F-07

#### Description

Create the shared widgets library.

#### Acceptance Criteria

- [ ] Library crate with `live_design` export
- [ ] Basic module structure
- [ ] Compiles with makepad-widgets dependency

#### Files

```
dora-studio-widgets/src/
├── lib.rs
├── theme.rs      # F-06
└── app_trait.rs  # F-07
```

---

### F-05: Implement sidebar navigation

**Labels**: `phase-0`, `priority-medium`
**Dependencies**: F-03, F-04
**Blocks**: Phase 2 apps

#### Description

Create the sidebar for switching between mini-apps.

#### Acceptance Criteria

- [ ] Vertical sidebar on left side
- [ ] 4 navigation buttons (icons + labels):
  - Dataflow Manager
  - YAML Editor
  - Log Viewer
  - Telemetry Dashboard
- [ ] Active state highlighting
- [ ] Click switches content area
- [ ] Collapse/expand toggle (optional)

#### Mockup

```
┌───────────────────────────────────────────┐
│ ┌────┐                                    │
│ │ ≡  │ Dora Studio                        │
│ ├────┼────────────────────────────────────│
│ │ ◉  │                                    │
│ │Data│                                    │
│ │flow│      (Content Area)                │
│ ├────┤                                    │
│ │ ○  │                                    │
│ │YAML│                                    │
│ ├────┤                                    │
│ │ ○  │                                    │
│ │Logs│                                    │
│ ├────┤                                    │
│ │ ○  │                                    │
│ │Tel │                                    │
│ └────┴────────────────────────────────────│
└───────────────────────────────────────────┘
```

---

### F-06: Implement theme system

**Labels**: `phase-0`, `priority-medium`, `good-first-issue`
**Dependencies**: F-04
**Blocks**: All apps

#### Description

Define the color palette and typography for dark mode.

#### Acceptance Criteria

- [ ] Dark mode color constants:
  - `DARK_BG`: `#0D1117`
  - `PANEL_BG`: `#161B22`
  - `BORDER`: `#30363D`
  - `TEXT_PRIMARY`: `#E6EDF3`
  - `TEXT_SECONDARY`: `#8B949E`
  - `ACCENT_BLUE`: `#58A6FF`
  - `ACCENT_GREEN`: `#3FB950`
  - `ACCENT_RED`: `#F85149`
  - `ACCENT_YELLOW`: `#D29922`
- [ ] Font constants for regular/bold weights
- [ ] Exported for use in all apps

#### Code

```rust
// src/theme.rs
pub const DARK_BG: Vec4 = vec4(0.051, 0.067, 0.090, 1.0);
pub const PANEL_BG: Vec4 = vec4(0.086, 0.106, 0.133, 1.0);
pub const BORDER: Vec4 = vec4(0.188, 0.212, 0.239, 1.0);
pub const TEXT_PRIMARY: Vec4 = vec4(0.902, 0.929, 0.953, 1.0);
pub const TEXT_SECONDARY: Vec4 = vec4(0.545, 0.580, 0.620, 1.0);
pub const ACCENT_BLUE: Vec4 = vec4(0.345, 0.651, 1.0, 1.0);
pub const ACCENT_GREEN: Vec4 = vec4(0.247, 0.725, 0.314, 1.0);
pub const ACCENT_RED: Vec4 = vec4(0.973, 0.318, 0.286, 1.0);
pub const ACCENT_YELLOW: Vec4 = vec4(0.824, 0.600, 0.133, 1.0);
```

---

### F-07: Define DoraApp trait

**Labels**: `phase-0`, `priority-high`
**Dependencies**: F-04
**Blocks**: All apps

#### Description

Create the plugin trait that all mini-apps implement.

#### Acceptance Criteria

- [ ] `DoraApp` trait with `info()` and `live_design(cx)` methods
- [ ] `AppInfo` struct with name, id, description, icon
- [ ] `AppRegistry` for managing registered apps
- [ ] Documentation and examples

#### Code

```rust
// src/app_trait.rs
use makepad_widgets::*;

pub struct AppInfo {
    pub name: &'static str,
    pub id: &'static str,
    pub description: &'static str,
    pub icon: &'static str,
}

pub trait DoraApp {
    fn info() -> AppInfo where Self: Sized;
    fn live_design(cx: &mut Cx);
}

pub struct AppRegistry {
    apps: Vec<AppInfo>,
}

impl AppRegistry {
    pub fn new() -> Self {
        Self { apps: Vec::new() }
    }

    pub fn register(&mut self, info: AppInfo) {
        self.apps.push(info);
    }

    pub fn apps(&self) -> &[AppInfo] {
        &self.apps
    }

    pub fn find_by_id(&self, id: &str) -> Option<&AppInfo> {
        self.apps.iter().find(|a| a.id == id)
    }
}
```

---

## Phase 1: Core Services

### C-01: Create dora-studio-client crate

**Labels**: `phase-1`, `component-client`, `priority-high`
**Dependencies**: F-01
**Blocks**: C-02, C-05, C-07

#### Description

Create the client library crate structure.

#### Acceptance Criteria

- [ ] Library crate with module structure:
  ```
  src/
  ├── lib.rs
  ├── client.rs    # DoraClient
  ├── storage.rs   # DataFusion storage
  ├── otlp.rs      # OTLP receiver
  └── models.rs    # Data types
  ```
- [ ] Compiles with dependencies

---

### C-02: Implement Coordinator TCP client

**Labels**: `phase-1`, `component-client`, `priority-high`
**Dependencies**: C-01
**Blocks**: C-03, C-04, A-02

#### Description

Implement TCP connection to Dora Coordinator.

#### Acceptance Criteria

- [ ] `DoraClient::connect(addr)` establishes TCP connection
- [ ] JSON message framing (length-prefixed or newline)
- [ ] Connection pooling or reconnection logic
- [ ] Error handling for network failures
- [ ] Unit tests with mock server

#### Code

```rust
pub struct DoraClient {
    stream: TcpStream,
    addr: SocketAddr,
}

impl DoraClient {
    pub async fn connect(addr: &str) -> Result<Self, ClientError> {
        let addr: SocketAddr = addr.parse()?;
        let stream = TcpStream::connect(addr).await?;
        Ok(Self { stream, addr })
    }

    async fn send_request<T: DeserializeOwned>(&mut self, req: Request) -> Result<T, ClientError> {
        // Send JSON request, receive JSON response
    }
}
```

---

### C-03: Implement dataflow operations

**Labels**: `phase-1`, `component-client`, `priority-high`
**Dependencies**: C-02
**Blocks**: A-02, A-03

#### Description

Implement dataflow CRUD operations.

#### Acceptance Criteria

- [ ] `list_dataflows()` returns all dataflows with status
- [ ] `start_dataflow(yaml_path)` starts from YAML file
- [ ] `stop_dataflow(uuid)` gracefully stops
- [ ] `destroy_dataflow(uuid)` forcefully terminates
- [ ] `reload_dataflow(uuid)` hot-reloads
- [ ] Integration tests against real Dora daemon

#### API

```rust
impl DoraClient {
    pub async fn list_dataflows(&mut self) -> Result<Vec<DataflowEntry>, ClientError>;
    pub async fn start_dataflow(&mut self, yaml_path: &str) -> Result<Uuid, ClientError>;
    pub async fn stop_dataflow(&mut self, uuid: &Uuid) -> Result<(), ClientError>;
    pub async fn destroy_dataflow(&mut self, uuid: &Uuid) -> Result<(), ClientError>;
    pub async fn reload_dataflow(&mut self, uuid: &Uuid) -> Result<(), ClientError>;
}

pub struct DataflowEntry {
    pub uuid: Uuid,
    pub name: Option<String>,
    pub status: DataflowStatus,
    pub node_count: usize,
    pub created_at: DateTime<Utc>,
}

pub enum DataflowStatus {
    Running,
    Finished,
    Failed(String),
}
```

---

### C-04: Implement log subscription

**Labels**: `phase-1`, `component-client`, `priority-medium`
**Dependencies**: C-02
**Blocks**: C-06 (Log Viewer)

#### Description

Implement real-time log streaming from Dora.

#### Acceptance Criteria

- [ ] `subscribe_logs()` returns async stream
- [ ] Logs include timestamp, level, node_id, message
- [ ] Backpressure handling for high-volume logs
- [ ] Reconnection on disconnect

#### API

```rust
impl DoraClient {
    pub async fn subscribe_logs(&mut self) -> Result<impl Stream<Item = LogMessage>, ClientError>;
}

pub struct LogMessage {
    pub timestamp: DateTime<Utc>,
    pub dataflow_id: Option<Uuid>,
    pub node_id: Option<String>,
    pub level: LogLevel,
    pub target: Option<String>,
    pub message: String,
}

pub enum LogLevel {
    Trace, Debug, Info, Warn, Error,
}
```

---

### C-05: Set up DataFusion storage

**Labels**: `phase-1`, `component-storage`, `priority-high`
**Dependencies**: C-01
**Blocks**: C-06, D-03

#### Description

Initialize DataFusion query engine with storage directory.

#### Acceptance Criteria

- [ ] `Storage::new(data_dir)` initializes DataFusion context
- [ ] Creates `~/.dora/studio/` directory structure
- [ ] Registers table schemas for metrics, logs, spans
- [ ] Basic query functionality works

#### Code

```rust
use datafusion::prelude::*;

pub struct Storage {
    ctx: SessionContext,
    data_dir: PathBuf,
}

impl Storage {
    pub async fn new(data_dir: &Path) -> Result<Self, StorageError> {
        let ctx = SessionContext::new();

        // Create directory structure
        let dirs = ["metrics", "logs", "spans"];
        for dir in dirs {
            tokio::fs::create_dir_all(data_dir.join(dir)).await?;
        }

        Ok(Self {
            ctx,
            data_dir: data_dir.to_path_buf(),
        })
    }

    pub async fn query(&self, sql: &str) -> Result<RecordBatch, StorageError> {
        let df = self.ctx.sql(sql).await?;
        let batches = df.collect().await?;
        // Return first batch or merge
    }
}
```

---

### C-06: Create Parquet schemas

**Labels**: `phase-1`, `component-storage`, `priority-medium`
**Dependencies**: C-05
**Blocks**: C-06 (Log Viewer), D-03

#### Description

Define Arrow schemas for persistent data.

#### Acceptance Criteria

- [ ] Metrics schema: timestamp, dataflow_id, node_id, cpu, memory, etc.
- [ ] Logs schema: timestamp, level, node_id, message, fields
- [ ] Spans schema: trace_id, span_id, parent_id, operation, start, duration
- [ ] Write functions to append Parquet files
- [ ] Read functions to scan Parquet directories

#### Schemas

```rust
use arrow::datatypes::{DataType, Field, Schema};

pub fn metrics_schema() -> Schema {
    Schema::new(vec![
        Field::new("timestamp", DataType::Timestamp(TimeUnit::Microsecond, None), false),
        Field::new("dataflow_id", DataType::Utf8, true),
        Field::new("node_id", DataType::Utf8, false),
        Field::new("cpu_percent", DataType::Float32, false),
        Field::new("memory_mb", DataType::Float64, false),
        Field::new("disk_read_mb_s", DataType::Float64, true),
        Field::new("disk_write_mb_s", DataType::Float64, true),
    ])
}

pub fn logs_schema() -> Schema {
    Schema::new(vec![
        Field::new("timestamp", DataType::Timestamp(TimeUnit::Microsecond, None), false),
        Field::new("dataflow_id", DataType::Utf8, true),
        Field::new("node_id", DataType::Utf8, true),
        Field::new("level", DataType::Utf8, false),
        Field::new("target", DataType::Utf8, true),
        Field::new("message", DataType::Utf8, false),
    ])
}

pub fn spans_schema() -> Schema {
    Schema::new(vec![
        Field::new("trace_id", DataType::Utf8, false),
        Field::new("span_id", DataType::Utf8, false),
        Field::new("parent_span_id", DataType::Utf8, true),
        Field::new("operation_name", DataType::Utf8, false),
        Field::new("service_name", DataType::Utf8, false),
        Field::new("start_time", DataType::Timestamp(TimeUnit::Microsecond, None), false),
        Field::new("duration_us", DataType::Int64, false),
        Field::new("status", DataType::Utf8, false),
    ])
}
```

---

### C-07: Implement OTLP receiver

**Labels**: `phase-1`, `component-client`, `priority-low`
**Dependencies**: C-01, C-05
**Blocks**: D-05

#### Description

Implement gRPC server to receive OpenTelemetry traces.

#### Acceptance Criteria

- [ ] gRPC server on port 4317 (OTLP default)
- [ ] Implements `TraceService` from opentelemetry-proto
- [ ] Parses incoming spans and stores via Storage
- [ ] Graceful shutdown

#### Note

This is lower priority - can use external OTLP collector initially.

---

## Phase 2: Mini-Apps

---

## App 2A: Dataflow Manager

### A-01: Create dataflow-manager crate

**Labels**: `phase-2`, `app-dataflow`, `priority-high`
**Dependencies**: F-07
**Blocks**: A-02

#### Description

Set up the Dataflow Manager mini-app crate.

#### Acceptance Criteria

- [ ] Implements `DoraApp` trait
- [ ] Registered in shell
- [ ] Empty screen renders

---

### A-02: Implement dataflow list table

**Labels**: `phase-2`, `app-dataflow`, `priority-high`
**Dependencies**: A-01, C-03
**Blocks**: A-03, A-04

#### Description

Create the main table showing all dataflows.

#### Acceptance Criteria

- [ ] Table columns: UUID, Name, Status, Nodes, CPU%, Memory
- [ ] Status badges with colors (Running=green, Finished=gray, Failed=red)
- [ ] Row selection
- [ ] Sort by column click
- [ ] Loading state

#### Mockup

```
│ UUID      │ Name        │ Status   │ Nodes │ CPU  │ Memory │
│ a1b2c3... │ yolo-detect │ ● Running│ 4     │ 45%  │ 2.1GB  │
│ d4e5f6... │ audio-proc  │ ○ Stopped│ 3     │ 0%   │ 0MB    │
```

---

### A-03: Implement start/stop actions

**Labels**: `phase-2`, `app-dataflow`, `priority-high`
**Dependencies**: A-02
**Blocks**: None

#### Description

Add action buttons for dataflow control.

#### Acceptance Criteria

- [ ] Start button (opens file picker for YAML)
- [ ] Stop button (selected dataflow)
- [ ] Destroy button (with confirmation)
- [ ] Reload button (hot reload)
- [ ] Disabled states when no selection
- [ ] Loading states during operations
- [ ] Error handling with toast/snackbar

---

### A-04: Implement node expansion

**Labels**: `phase-2`, `app-dataflow`, `priority-medium`
**Dependencies**: A-02
**Blocks**: None

#### Description

Add expandable rows to show per-node metrics.

#### Acceptance Criteria

- [ ] Expand/collapse chevron on each row
- [ ] Nested table with node details:
  - Node ID
  - Status
  - CPU%
  - Memory
  - Message rate
- [ ] Smooth expand/collapse animation

---

### A-05: Add refresh and auto-update

**Labels**: `phase-2`, `app-dataflow`, `priority-medium`
**Dependencies**: A-02
**Blocks**: None

#### Description

Keep the dataflow list up to date.

#### Acceptance Criteria

- [ ] Manual refresh button
- [ ] Auto-refresh toggle (every 5s)
- [ ] Real-time updates via subscription (if available)
- [ ] Last updated timestamp display

---

### A-06: Add infrastructure status

**Labels**: `phase-2`, `app-dataflow`, `priority-low`
**Dependencies**: A-01, C-02
**Blocks**: None

#### Description

Show Coordinator and Daemon health.

#### Acceptance Criteria

- [ ] Status panel at top or sidebar
- [ ] Coordinator: connected/disconnected
- [ ] Daemon count and status
- [ ] Reconnect button if disconnected

---

## App 2B: YAML Editor

### B-01: Create yaml-editor crate

**Labels**: `phase-2`, `app-yaml`, `priority-high`
**Dependencies**: F-07
**Blocks**: B-02, B-03

#### Description

Set up the YAML Editor mini-app crate.

#### Acceptance Criteria

- [ ] Implements `DoraApp` trait
- [ ] Registered in shell
- [ ] Split view layout (editor left, graph right)

---

### B-02: Integrate makepad-flow

**Labels**: `phase-2`, `app-yaml`, `priority-high`
**Dependencies**: B-01, F-02
**Blocks**: B-04, B-05

#### Description

Embed FlowCanvas widget from makepad-flow.

#### Acceptance Criteria

- [ ] FlowCanvas renders in right panel
- [ ] Pan with shift+drag
- [ ] Zoom with mouse wheel
- [ ] Node selection works
- [ ] Category colors display correctly

#### Integration Code

```rust
use makepad_flow::FlowCanvas;

live_design! {
    GraphPanel = <View> {
        flow_canvas = <FlowCanvas> {
            // Configure FlowCanvas
        }
    }
}
```

---

### B-03: Implement YAML text editor

**Labels**: `phase-2`, `app-yaml`, `priority-high`
**Dependencies**: B-01
**Blocks**: B-04, B-07

#### Description

Create the YAML editing panel.

#### Acceptance Criteria

- [ ] Multi-line text input
- [ ] Syntax highlighting (YAML)
- [ ] Line numbers
- [ ] Monospace font
- [ ] Scroll for large files
- [ ] Cursor and selection

---

### B-04: Implement YAML → graph sync

**Labels**: `phase-2`, `app-yaml`, `priority-high`
**Dependencies**: B-02, B-03
**Blocks**: None

#### Description

Update graph when YAML changes.

#### Acceptance Criteria

- [ ] Parse YAML on change (debounced)
- [ ] Extract nodes and edges
- [ ] Update FlowCanvas data model
- [ ] Layout algorithm (hierarchical/Dagre-style)
- [ ] Handle parse errors gracefully

#### Data Flow

```
YAML Text → Parse → NodeGraph → FlowCanvas.set_data()
```

---

### B-05: Implement graph → YAML sync

**Labels**: `phase-2`, `app-yaml`, `priority-medium`
**Dependencies**: B-02, B-03
**Blocks**: None

#### Description

Update YAML when graph is edited.

#### Acceptance Criteria

- [ ] Node drag updates position (if positions stored)
- [ ] Connection creation adds input mapping
- [ ] Node deletion removes from YAML
- [ ] Maintains YAML formatting/comments where possible

---

### B-06: Add node inspector panel

**Labels**: `phase-2`, `app-yaml`, `priority-medium`
**Dependencies**: B-02
**Blocks**: None

#### Description

Show selected node details in sidebar.

#### Acceptance Criteria

- [ ] Displays when node selected
- [ ] Shows: ID, path, inputs, outputs, env vars
- [ ] Editable fields update YAML
- [ ] Collapsible sections

---

### B-07: Add validation feedback

**Labels**: `phase-2`, `app-yaml`, `priority-medium`
**Dependencies**: B-03
**Blocks**: None

#### Description

Show YAML validation errors inline.

#### Acceptance Criteria

- [ ] Parse errors highlighted in editor
- [ ] Error line marked with red
- [ ] Error message in status bar
- [ ] Schema validation (optional)

---

### B-08: Implement file operations

**Labels**: `phase-2`, `app-yaml`, `priority-medium`, `good-first-issue`
**Dependencies**: B-03
**Blocks**: None

#### Description

Add New/Open/Save file operations.

#### Acceptance Criteria

- [ ] New: Clear editor, reset graph
- [ ] Open: File picker, load YAML
- [ ] Save: Write to file (overwrite or save-as)
- [ ] Unsaved changes indicator (*)
- [ ] Confirm before discard

---

## App 2C: Log Viewer

### C-01 (Log): Create log-viewer crate

**Labels**: `phase-2`, `app-logs`, `priority-high`
**Dependencies**: F-07
**Blocks**: C-02 (Log)

#### Description

Set up the Log Viewer mini-app crate.

#### Acceptance Criteria

- [ ] Implements `DoraApp` trait
- [ ] Registered in shell
- [ ] Basic layout renders

---

### C-02 (Log): Implement virtualized log list

**Labels**: `phase-2`, `app-logs`, `priority-high`
**Dependencies**: C-01 (Log)
**Blocks**: C-03 (Log), C-05 (Log)

#### Description

Create high-performance log display for 100K+ entries.

#### Acceptance Criteria

- [ ] Virtual scrolling (only render visible rows)
- [ ] Smooth scrolling at 60 FPS
- [ ] Handle 100K+ log entries
- [ ] Columns: Timestamp, Node, Level, Message
- [ ] Level-based row coloring

---

### C-03 (Log): Implement level filtering

**Labels**: `phase-2`, `app-logs`, `priority-high`
**Dependencies**: C-02 (Log)
**Blocks**: None

#### Description

Filter logs by level.

#### Acceptance Criteria

- [ ] Level dropdown: ALL, DEBUG+, INFO+, WARN+, ERROR
- [ ] Instant filtering (no reload)
- [ ] Count badges per level

---

### C-04 (Log): Implement node/dataflow filter

**Labels**: `phase-2`, `app-logs`, `priority-medium`
**Dependencies**: C-02 (Log)
**Blocks**: None

#### Description

Filter logs by source.

#### Acceptance Criteria

- [ ] Dataflow dropdown (all + each running)
- [ ] Node dropdown (all + nodes in selected dataflow)
- [ ] Combined filtering with level

---

### C-05 (Log): Implement text search

**Labels**: `phase-2`, `app-logs`, `priority-medium`
**Dependencies**: C-02 (Log)
**Blocks**: None

#### Description

Search log messages.

#### Acceptance Criteria

- [ ] Search input field
- [ ] Real-time filtering as you type
- [ ] Regex support (toggle)
- [ ] Highlight matches
- [ ] Match count display

---

### C-06 (Log): Add real-time streaming

**Labels**: `phase-2`, `app-logs`, `priority-medium`
**Dependencies**: C-02 (Log), C-04 (Phase 1)
**Blocks**: None

#### Description

Stream new logs in real-time.

#### Acceptance Criteria

- [ ] Play/Pause button
- [ ] New logs append at bottom
- [ ] Auto-scroll when at bottom
- [ ] "N new logs" indicator when scrolled up
- [ ] Clear button

---

### C-07 (Log): Add export functionality

**Labels**: `phase-2`, `app-logs`, `priority-low`, `good-first-issue`
**Dependencies**: C-02 (Log)
**Blocks**: None

#### Description

Export filtered logs to file.

#### Acceptance Criteria

- [ ] Export button
- [ ] Format options: CSV, JSON, Plain text
- [ ] Respects current filters
- [ ] File save dialog

---

## App 2D: Telemetry Dashboard

### D-01: Create telemetry-dashboard crate

**Labels**: `phase-2`, `app-telemetry`, `priority-high`
**Dependencies**: F-07
**Blocks**: D-02, D-05

#### Description

Set up the Telemetry Dashboard mini-app crate.

#### Acceptance Criteria

- [ ] Implements `DoraApp` trait
- [ ] Registered in shell
- [ ] Grid layout for panels

---

### D-02: Integrate makepad-chart

**Labels**: `phase-2`, `app-telemetry`, `priority-high`
**Dependencies**: D-01, F-02
**Blocks**: D-03, D-07

#### Description

Embed chart widgets from makepad-chart.

#### Acceptance Criteria

- [ ] Line chart renders
- [ ] Bar chart renders
- [ ] Animations work
- [ ] Theme colors applied

#### Integration Code

```rust
use makepad_chart::{LineChart, BarChart, ChartData, ChartOptions};

live_design! {
    MetricsPanel = <View> {
        cpu_chart = <LineChart> {}
        memory_chart = <LineChart> {}
    }
}

// In event handler:
let data = ChartData::new()
    .labels(timestamps)
    .dataset("CPU %", cpu_values);
self.cpu_chart.set_data(data);
```

---

### D-03: Implement metrics panel

**Labels**: `phase-2`, `app-telemetry`, `priority-high`
**Dependencies**: D-02, C-06 (Phase 1)
**Blocks**: None

#### Description

Show CPU, Memory, Throughput charts.

#### Acceptance Criteria

- [ ] CPU usage line chart (per node)
- [ ] Memory usage line chart
- [ ] Throughput bar chart (messages/sec)
- [ ] Legend with node names
- [ ] Hover tooltips with values

---

### D-04: Implement time range selector

**Labels**: `phase-2`, `app-telemetry`, `priority-medium`
**Dependencies**: D-03
**Blocks**: None

#### Description

Select time window for metrics.

#### Acceptance Criteria

- [ ] Preset buttons: 5m, 15m, 1h, 6h, 24h
- [ ] Custom range picker (optional)
- [ ] Updates all charts on change
- [ ] Loading indicator during fetch

---

### D-05: Implement trace timeline

**Labels**: `phase-2`, `app-telemetry`, `priority-medium`
**Dependencies**: D-01, C-07 (Phase 1)
**Blocks**: None

#### Description

Show distributed traces as waterfall/Gantt.

#### Acceptance Criteria

- [ ] Trace list with search
- [ ] Waterfall view of spans
- [ ] Span details on click
- [ ] Duration bars with scale
- [ ] Color by service/node

#### Mockup

```
│ Trace Timeline                                            │
│ camera ████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 12ms          │
│ yolo   ░░░░████████████████████████████████░░ 145ms       │
│ plot   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░███ 8ms          │
```

---

### D-06: Implement topic stats panel

**Labels**: `phase-2`, `app-telemetry`, `priority-low`
**Dependencies**: D-01
**Blocks**: None

#### Description

Show message rates per topic.

#### Acceptance Criteria

- [ ] Table: Topic, Messages/sec, Bandwidth, Avg Size
- [ ] Sort by column
- [ ] Sparkline for rate over time

---

### D-07: Add golden signals panel

**Labels**: `phase-2`, `app-telemetry`, `priority-medium`
**Dependencies**: D-03
**Blocks**: None

#### Description

Show key health indicators.

#### Acceptance Criteria

- [ ] Request rate (msg/s)
- [ ] Error rate (%)
- [ ] Latency percentiles (P50, P95, P99)
- [ ] Saturation (queue depths)
- [ ] Visual gauges or big numbers

---

## Phase 3: AI Integration

### AI-01: Create LlmClient trait

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: C-01 (Phase 1)
**Blocks**: AI-02, AI-03

#### Description

Define the LLM provider abstraction.

#### Acceptance Criteria

- [ ] `LlmClient` async trait
- [ ] Methods: `chat()`, `continue_with_result()`
- [ ] Tool calling support
- [ ] Streaming support
- [ ] Error types

#### Code

```rust
#[async_trait]
pub trait LlmClient: Send + Sync {
    async fn chat(
        &self,
        context: &Context,
        message: &str,
        tools: &[ToolDefinition],
    ) -> Result<AgentResponse, LlmError>;

    async fn continue_with_result(
        &self,
        context: &Context,
        tool_call: &ToolCall,
        result: &ToolResult,
    ) -> Result<AgentResponse, LlmError>;
}

pub enum AgentResponse {
    Text(String),
    ToolCall(ToolCall),
    Stream(Pin<Box<dyn Stream<Item = String>>>),
}
```

---

### AI-02: Implement Claude API client

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: AI-01
**Blocks**: AI-04

#### Description

Implement Claude as primary LLM provider.

#### Acceptance Criteria

- [ ] Implements `LlmClient` trait
- [ ] Uses Claude API with tool use
- [ ] Handles rate limits
- [ ] API key from config/env
- [ ] Streaming responses

---

### AI-03: Implement Ollama client

**Labels**: `phase-3`, `component-ai`, `priority-low`
**Dependencies**: AI-01
**Blocks**: AI-04

#### Description

Implement local LLM via Ollama.

#### Acceptance Criteria

- [ ] Implements `LlmClient` trait
- [ ] Connects to local Ollama server
- [ ] Tool calling (if model supports)
- [ ] Fallback to basic completion

---

### AI-04: Create AgentCoordinator

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: AI-01, AI-02
**Blocks**: AI-05, AI-07

#### Description

Central coordinator for AI agent operations.

#### Acceptance Criteria

- [ ] Manages LLM client
- [ ] Dispatches tool calls
- [ ] Handles agent loop (user → LLM → tool → LLM → response)
- [ ] Thread-safe for concurrent requests

#### Code

```rust
pub struct AgentCoordinator {
    llm_client: Arc<dyn LlmClient>,
    tool_registry: ToolRegistry,
    context_manager: ContextManager,
}

impl AgentCoordinator {
    pub async fn process_message(
        &self,
        app_id: &str,
        message: &str,
    ) -> Result<AgentResponse, AgentError> {
        let context = self.context_manager.build_context(app_id);
        let tools = self.tool_registry.tools_for_app(app_id);

        let response = self.llm_client.chat(&context, message, &tools).await?;

        match response {
            AgentResponse::ToolCall(call) => {
                let result = self.execute_tool(&call).await?;
                self.llm_client.continue_with_result(&context, &call, &result).await
            }
            other => Ok(other),
        }
    }
}
```

---

### AI-05: Define tools per mini-app

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: AI-04, Phase 2 apps
**Blocks**: AI-07

#### Description

Implement the 20+ AI tools.

#### Acceptance Criteria

**Dataflow Manager (6 tools):**
- [ ] `list_dataflows`
- [ ] `start_dataflow`
- [ ] `stop_dataflow`
- [ ] `get_dataflow_status`
- [ ] `get_node_metrics`
- [ ] `restart_dataflow`

**YAML Editor (6 tools):**
- [ ] `validate_yaml`
- [ ] `explain_dataflow`
- [ ] `suggest_fix`
- [ ] `generate_dataflow`
- [ ] `add_node`
- [ ] `connect_nodes`

**Log Viewer (5 tools):**
- [ ] `search_logs`
- [ ] `analyze_logs`
- [ ] `filter_logs`
- [ ] `export_logs`
- [ ] `count_by_level`

**Telemetry Dashboard (6 tools):**
- [ ] `query_metrics`
- [ ] `find_bottleneck`
- [ ] `get_trace`
- [ ] `analyze_latency`
- [ ] `compare_metrics`
- [ ] `get_topic_stats`

---

### AI-06: Create ChatBar widget

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: F-04
**Blocks**: AI-07

#### Description

Create the bottom chat bar widget.

#### Acceptance Criteria

- [ ] Input field with placeholder "Ask AI..."
- [ ] Send button (or Enter to send)
- [ ] Response area (expandable)
- [ ] Typing indicator
- [ ] Message history (collapsible)
- [ ] Clear button

#### Mockup

```
├─────────────────────────────────────────────────────────────┤
│ 💬 Ask AI: [Start the camera pipeline________________] [↵]  │
│                                                             │
│ AI: Starting dataflow... ✓ Started (uuid: abc123)          │
│     4 nodes running. Camera capturing at 30 FPS.           │
└─────────────────────────────────────────────────────────────┘
```

---

### AI-07: Integrate chat bar into apps

**Labels**: `phase-3`, `component-ai`, `priority-high`
**Dependencies**: AI-04, AI-05, AI-06
**Blocks**: None

#### Description

Add ChatBar to each mini-app.

#### Acceptance Criteria

- [ ] ChatBar at bottom of each app
- [ ] Context includes current app state
- [ ] Only shows tools relevant to current app
- [ ] Handles loading/error states

---

### AI-08: Implement context manager

**Labels**: `phase-3`, `component-ai`, `priority-medium`
**Dependencies**: AI-04
**Blocks**: None

#### Description

Manage context window and summarization.

#### Acceptance Criteria

- [ ] Token counting
- [ ] Context truncation strategy
- [ ] App state summarization (not raw data)
- [ ] Conversation history management

---

## Phase 4: Polish

### P-01: Write unit tests

**Labels**: `phase-4`, `priority-high`
**Dependencies**: Phase 2
**Blocks**: P-07

#### Description

Comprehensive unit test coverage.

#### Acceptance Criteria

- [ ] DoraClient tests (with mocks)
- [ ] Storage tests
- [ ] Tool execution tests
- [ ] Widget state tests
- [ ] >70% coverage

---

### P-02: Write integration tests

**Labels**: `phase-4`, `priority-high`
**Dependencies**: Phase 2
**Blocks**: P-07

#### Description

End-to-end integration tests.

#### Acceptance Criteria

- [ ] Full workflow: start dataflow → view logs → stop
- [ ] YAML edit → graph update → save
- [ ] Metrics query → chart render
- [ ] AI conversation flow

---

### P-03: Performance optimization

**Labels**: `phase-4`, `priority-medium`
**Dependencies**: Phase 2
**Blocks**: P-07

#### Description

Ensure 60 FPS and fast operations.

#### Acceptance Criteria

- [ ] Profile and optimize hot paths
- [ ] Log viewer handles 100K entries smoothly
- [ ] Chart updates don't cause frame drops
- [ ] Memory stays under 200MB typical use

---

### P-04: Accessibility review

**Labels**: `phase-4`, `priority-low`, `good-first-issue`
**Dependencies**: Phase 2
**Blocks**: P-07

#### Description

Keyboard navigation and screen reader support.

#### Acceptance Criteria

- [ ] Tab navigation through all interactive elements
- [ ] Keyboard shortcuts documented
- [ ] Focus indicators visible
- [ ] ARIA labels where applicable

---

### P-05: User documentation

**Labels**: `phase-4`, `priority-medium`
**Dependencies**: Phase 2, Phase 3
**Blocks**: P-07

#### Description

Create user guide.

#### Acceptance Criteria

- [ ] Getting started guide
- [ ] Feature documentation per app
- [ ] AI assistant usage examples
- [ ] Keyboard shortcuts reference
- [ ] Troubleshooting FAQ

---

### P-06: API documentation

**Labels**: `phase-4`, `priority-medium`, `good-first-issue`
**Dependencies**: Phase 1, Phase 2, Phase 3
**Blocks**: P-07

#### Description

Generate and review Rustdoc.

#### Acceptance Criteria

- [ ] All public APIs documented
- [ ] Examples in doc comments
- [ ] `cargo doc` generates clean output
- [ ] README links to docs

---

### P-07: Release packaging

**Labels**: `phase-4`, `priority-high`
**Dependencies**: P-01 through P-06
**Blocks**: None

#### Description

Build release binaries.

#### Acceptance Criteria

- [ ] Release build configuration
- [ ] macOS binary (universal)
- [ ] Linux binary (x86_64)
- [ ] Windows binary (optional)
- [ ] GitHub release automation
- [ ] Version tagging

---

## Issue Creation Script

Run this to create GitHub issues:

```bash
# Install GitHub CLI if needed
# brew install gh

# Create labels
gh label create phase-0 --color 0E8A16 --description "Foundation phase"
gh label create phase-1 --color 1D76DB --description "Core services phase"
gh label create phase-2 --color 5319E7 --description "Mini-apps phase"
gh label create phase-3 --color FBCA04 --description "AI integration phase"
gh label create phase-4 --color D93F0B --description "Polish phase"
gh label create app-dataflow --color C2E0C6 --description "Dataflow Manager app"
gh label create app-yaml --color BFD4F2 --description "YAML Editor app"
gh label create app-logs --color FEF2C0 --description "Log Viewer app"
gh label create app-telemetry --color F9D0C4 --description "Telemetry Dashboard app"
gh label create component-client --color E6E6E6 --description "DoraClient component"
gh label create component-storage --color E6E6E6 --description "Storage component"
gh label create component-ai --color D4C5F9 --description "AI Agent component"
gh label create priority-high --color B60205 --description "High priority"
gh label create priority-medium --color FBCA04 --description "Medium priority"
gh label create good-first-issue --color 7057FF --description "Good for newcomers"

# Example issue creation (repeat for each issue):
gh issue create \
  --title "F-01: Create workspace structure" \
  --label "phase-0,priority-high" \
  --body "See ISSUES.md for full description"
```

---

## Summary Statistics

| Phase | Issues | High Priority | Good First Issue |
|-------|--------|---------------|------------------|
| Phase 0 | 7 | 4 | 1 |
| Phase 1 | 7 | 4 | 0 |
| Phase 2 (Apps) | 27 | 10 | 3 |
| Phase 3 (AI) | 8 | 5 | 0 |
| Phase 4 (Polish) | 7 | 3 | 2 |
| **Total** | **56** | **26** | **6** |
