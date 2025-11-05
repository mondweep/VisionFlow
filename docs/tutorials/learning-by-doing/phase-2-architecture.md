# Phase 2: Core Architecture Deep Dive

**Learn VisionFlow by exploring the codebase**

This tutorial takes you through the source directories, mapping the running application back to the code. You'll trace operations through the Hexagonal Architecture, understand the ontology reasoning pipeline, identify GPU compute integration, and explore the client-server synchronization.

---

## 🎯 Learning Objectives

By the end of this tutorial, you will:

✅ Navigate the VisionFlow codebase with confidence
✅ Trace CQRS operations through the Hexagonal Architecture
✅ Understand the ontology validation and reasoning pipeline
✅ Identify GPU kernel integration points
✅ Comprehend the Binary WebSocket Protocol for real-time sync
✅ Map the running application to specific source files

**Time Required**: 3-4 hours
**Difficulty**: Intermediate
**Prerequisites**: Completed Phase 1, basic Rust and JavaScript knowledge

---

## Prerequisites Checklist

Before starting, ensure:

- [ ] VisionFlow is running (`docker-compose ps` shows all services up)
- [ ] You completed Phase 1 or understand the user-facing features
- [ ] You have a code editor (VS Code, IntelliJ, etc.)
- [ ] You're comfortable reading Rust and JavaScript/Vue.js code

**Don't have Rust/JS experience?** That's okay! This tutorial explains concepts as it goes.

---

## Architecture Overview Refresher

VisionFlow's architecture has four major layers:

```
┌─────────────────────────────────────────────────┐
│          Client Layer (Vue.js/Three.js)         │
│  • 3D Visualization                             │
│  • User Interaction                             │
│  • Real-time Updates via WebSocket              │
└─────────────────────────────────────────────────┘
                      ▼ Binary WebSocket Protocol V2
┌─────────────────────────────────────────────────┐
│         Server Layer (Rust/Actix-web)           │
│  • Hexagonal Architecture                       │
│  • CQRS Pattern (Commands/Queries)              │
│  • Business Logic                               │
└─────────────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────┐
│        Ontology Layer (Whelk-rs Reasoner)       │
│  • Semantic Validation                          │
│  • Logical Inference                            │
│  • Ontology Storage (SQLite)                    │
└─────────────────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────┐
│         GPU Layer (CUDA Kernels)                │
│  • Physics Simulation (39 kernels)             │
│  • Semantic Forces                              │
│  • Clustering & Pathfinding                     │
└─────────────────────────────────────────────────┘
```

**Key Insight**: Each layer is isolated through **ports** and **adapters**, allowing them to evolve independently.

---

## Layer 1: Backend Logic (Hexagonal Architecture + CQRS)

**Duration**: 60 minutes
**Focus**: Understanding how the server processes requests

### 1.1 Project Structure Overview

Open the project in your code editor and explore the structure:

```bash
# View the Rust source structure
tree -L 2 src/

# Expected structure:
src/
├── main.rs                  # Application entry point
├── cqrs/                    # CQRS directives and queries
├── services/                # Business logic (core domain)
├── repositories/            # Data access layer
├── adapters/                # External integrations (HTTP, WebSocket)
├── ontology/                # Ontology system
├── gpu/                     # GPU compute integration
└── protocols/               # Binary protocol implementation
```

**Key Directories**:

| Directory | Purpose | Example Files |
|-----------|---------|---------------|
| `src/main.rs` | App bootstrapping | Server initialization, config |
| `src/cqrs/` | Command/Query definitions | `directives.rs`, `queries.rs` |
| `src/services/` | Business logic | `graph_service.rs`, `ontology_service.rs` |
| `src/repositories/` | Data persistence | `graph_repository.rs`, `ontology_repository.rs` |
| `src/adapters/` | External interfaces | `http_adapter.rs`, `websocket_adapter.rs` |
| `src/ontology/` | Reasoning pipeline | `whelk_reasoner.rs`, `validator.rs` |
| `src/gpu/` | CUDA integration | `physics.rs`, `kernels/` |

### 1.2 Trace a Read Operation (Query)

**Scenario**: When a user loads the graph in the UI, what happens?

**Step 1**: Find the HTTP endpoint
```bash
# Search for the graph query endpoint
grep -r "get_graph" src/adapters/
```

Open `src/adapters/http_adapter.rs` (or similar):

```rust
// HTTP Adapter (Port to external world)
#[get("/api/graph")]
async fn get_graph(
    query: web::Json<GetGraphQuery>,
    graph_service: web::Data<GraphService>,
) -> Result<HttpResponse, Error> {
    // 1. Adapter receives HTTP request
    // 2. Translates to domain query
    let nodes = graph_service.query_graph(query.into_inner()).await?;

    // 3. Returns response via port
    Ok(HttpResponse::Ok().json(nodes))
}
```

**Step 2**: Follow the service layer
Open `src/services/graph_service.rs`:

```rust
// Business Logic (Core Domain)
impl GraphService {
    pub async fn query_graph(&self, query: GetGraphQuery) -> Result<Vec<Node>, ServiceError> {
        // 1. Business logic validates query
        self.validate_query(&query)?;

        // 2. Calls repository port
        let nodes = self.repository.get_nodes(query.filters).await?;

        // 3. Applies ontology rules
        let enriched = self.ontology.enrich_nodes(nodes).await?;

        Ok(enriched)
    }
}
```

**Step 3**: Check the repository
Open `src/repositories/graph_repository.rs`:

```rust
// Data Access (Adapter to SQLite)
impl GraphRepository {
    pub async fn get_nodes(&self, filters: NodeFilters) -> Result<Vec<Node>, RepoError> {
        // Raw SQL query to unified.db
        let query = "SELECT * FROM nodes WHERE type = ?1";
        let nodes = sqlx::query_as::<_, Node>(query)
            .bind(filters.node_type)
            .fetch_all(&self.pool)
            .await?;

        Ok(nodes)
    }
}
```

**🎓 What You Learned**:
- **Hexagonal Architecture**: Outer adapters (HTTP) → Core services → Outer adapters (Database)
- **Separation of Concerns**: HTTP details don't leak into business logic
- **Ports**: Interfaces that define contracts between layers

**Practice Exercise**: Trace the same flow for creating a node (POST request).

### 1.3 Trace a Write Operation (Directive)

**Scenario**: A user creates a new node via the UI.

**Step 1**: Find the HTTP endpoint
```bash
grep -r "create_node\|CreateNode" src/adapters/
```

Open `src/adapters/http_adapter.rs`:

```rust
#[post("/api/node")]
async fn create_node(
    directive: web::Json<CreateNodeDirective>,
    graph_service: web::Data<GraphService>,
) -> Result<HttpResponse, Error> {
    // Adapter receives directive
    let node_id = graph_service.execute_directive(directive.into_inner()).await?;
    Ok(HttpResponse::Created().json(node_id))
}
```

**Step 2**: Check the CQRS directive
Open `src/cqrs/directives.rs`:

```rust
// Directive = Command with validation
#[derive(Serialize, Deserialize)]
pub struct CreateNodeDirective {
    pub label: String,
    pub node_type: String,
    pub properties: HashMap<String, Value>,
}

impl Directive for CreateNodeDirective {
    type Output = NodeId;

    fn validate(&self) -> Result<(), ValidationError> {
        // Pre-execution validation
        if self.label.is_empty() {
            return Err(ValidationError::EmptyLabel);
        }
        Ok(())
    }
}
```

**Step 3**: Service executes directive
Open `src/services/graph_service.rs`:

```rust
impl GraphService {
    pub async fn execute_directive(&self, directive: CreateNodeDirective) -> Result<NodeId, ServiceError> {
        // 1. Validate directive
        directive.validate()?;

        // 2. Validate against ontology
        self.ontology.validate_node_type(&directive.node_type).await?;

        // 3. Create in repository
        let node_id = self.repository.create_node(directive).await?;

        // 4. Trigger reasoning
        self.ontology.infer_relationships(node_id).await?;

        // 5. Notify via WebSocket
        self.broadcast_update(NodeCreated { node_id }).await?;

        Ok(node_id)
    }
}
```

**🎓 What You Learned**:
- **CQRS Pattern**: Separates reads (queries) from writes (directives/commands)
- **Validation Pipeline**: HTTP → Directive validation → Ontology validation
- **Side Effects**: Creating a node triggers reasoning and WebSocket updates

**Practice Exercise**: Find where the `broadcast_update` is implemented. Hint: Check `src/adapters/websocket_adapter.rs`.

### 1.4 Explore the Database Schema

VisionFlow uses a single SQLite database: `unified.db`

```bash
# Connect to the database
docker exec -it visionflow-container sqlite3 /app/data/unified.db

# List tables
.tables

# Expected:
# nodes  edges  ontology_axioms  reasoning_cache  settings
```

**Key Tables**:

```sql
-- Nodes table
CREATE TABLE nodes (
    id TEXT PRIMARY KEY,
    label TEXT NOT NULL,
    type TEXT NOT NULL,
    properties JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Edges table
CREATE TABLE edges (
    id TEXT PRIMARY KEY,
    source_id TEXT NOT NULL,
    target_id TEXT NOT NULL,
    relationship_type TEXT NOT NULL,
    FOREIGN KEY (source_id) REFERENCES nodes(id),
    FOREIGN KEY (target_id) REFERENCES nodes(id)
);

-- Ontology axioms
CREATE TABLE ontology_axioms (
    id INTEGER PRIMARY KEY,
    axiom_type TEXT NOT NULL,
    subject TEXT NOT NULL,
    object TEXT,
    content BLOB
);
```

**Practice Exercise**:
```sql
-- Find all nodes of a specific type
SELECT label, properties FROM nodes WHERE type = 'Concept';

-- Find all edges with a specific relationship
SELECT n1.label, e.relationship_type, n2.label
FROM edges e
JOIN nodes n1 ON e.source_id = n1.id
JOIN nodes n2 ON e.target_id = n2.id
WHERE e.relationship_type = 'subClassOf';

-- Exit sqlite
.quit
```

**🎓 What You Learned**:
- The data model: nodes, edges, and ontology axioms stored in SQLite
- How to query the database directly for debugging
- The schema supports generic key-value properties (JSON)

---

## Layer 2: Ontology System (Semantic Validation)

**Duration**: 60 minutes
**Focus**: Understanding the "intelligence layer"

### 2.1 What is the Ontology System?

Think of it as a **"grammar checker for your data"**:

- **Validates** that data follows logical rules
- **Infers** new relationships automatically
- **Organizes** the 3D visualization based on semantic types
- **Guides** AI agents with domain knowledge

**Example**: If you define "Person cannot also be an Organization", the ontology prevents you from accidentally creating an entity that's both.

### 2.2 Locate the Ontology Code

```bash
# View ontology directory structure
tree src/ontology/

# Expected:
src/ontology/
├── mod.rs                   # Module exports
├── whelk_reasoner.rs        # Integration with Whelk-rs
├── validator.rs             # Validation logic
├── axiom_parser.rs          # OWL/RDF parsing
├── reasoning_cache.rs       # Performance optimization
└── semantic_physics.rs      # Translation to 3D forces
```

### 2.3 Examine the Whelk-rs Reasoner

Open `src/ontology/whelk_reasoner.rs`:

```rust
use whelk::{Reasoner, Axiom};

pub struct WhelkReasoner {
    reasoner: Reasoner,
    cache: ReasoningCache,
}

impl WhelkReasoner {
    pub fn new() -> Self {
        let reasoner = Reasoner::new();
        Self {
            reasoner,
            cache: ReasoningCache::new(),
        }
    }

    /// Load ontology axioms from database
    pub async fn load_ontology(&mut self, axioms: Vec<OntologyAxiom>) -> Result<(), ReasonerError> {
        for axiom in axioms {
            // Parse OWL axiom and add to reasoner
            let whelk_axiom = self.parse_axiom(axiom)?;
            self.reasoner.add_axiom(whelk_axiom);
        }

        // Classify the ontology (builds hierarchy)
        self.reasoner.classify();
        Ok(())
    }

    /// Check if a node type is valid
    pub fn validate_node_type(&self, node_type: &str) -> Result<(), ValidationError> {
        if !self.reasoner.contains_class(node_type) {
            return Err(ValidationError::UnknownNodeType(node_type.to_string()));
        }
        Ok(())
    }

    /// Infer new relationships based on ontology rules
    pub async fn infer_relationships(&self, node_id: NodeId) -> Result<Vec<InferredEdge>, ReasonerError> {
        // Get node type
        let node_type = self.get_node_type(node_id).await?;

        // Query reasoner for implied relationships
        let superclasses = self.reasoner.get_superclasses(&node_type);
        let properties = self.reasoner.get_applicable_properties(&node_type);

        // Generate inferred edges
        let mut inferred = Vec::new();
        for property in properties {
            if let Some(inverse) = self.reasoner.get_inverse_property(&property) {
                // Create bidirectional edges automatically
                inferred.push(InferredEdge { property, inverse });
            }
        }

        Ok(inferred)
    }
}
```

**🎓 What You Learned**:
- **Whelk-rs** is a description logic reasoner (similar to Pellet/HermiT in Java)
- **Classification** builds the class hierarchy (subclass relationships)
- **Inference** discovers new facts based on rules
- **Validation** ensures data consistency

### 2.4 Run an Ontology Validation Example

Let's see validation in action:

**Step 1**: Create a test ontology file
```bash
# Create a simple ontology
cat > /tmp/test-ontology.owl <<EOF
<?xml version="1.0"?>
<Ontology xmlns="http://www.w3.org/2002/07/owl#">
  <!-- Define classes -->
  <Class IRI="#Person"/>
  <Class IRI="#Organization"/>

  <!-- Define disjoint classes (mutually exclusive) -->
  <DisjointClasses>
    <Class IRI="#Person"/>
    <Class IRI="#Organization"/>
  </DisjointClasses>

  <!-- Define property -->
  <ObjectProperty IRI="#worksFor">
    <Domain IRI="#Person"/>
    <Range IRI="#Organization"/>
  </ObjectProperty>
</Ontology>
EOF
```

**Step 2**: Load it via API
```bash
# Upload ontology
curl -X POST http://localhost:3030/api/ontology/load \
  -H "Content-Type: application/xml" \
  --data-binary @/tmp/test-ontology.owl

# Expected response:
# {"status":"loaded","classes":2,"properties":1}
```

**Step 3**: Test validation
```bash
# Try to create a node that's both Person and Organization (should fail)
curl -X POST http://localhost:3030/api/node \
  -H "Content-Type: application/json" \
  -d '{
    "label": "Invalid Entity",
    "node_type": "Person",
    "properties": {
      "also_type": "Organization"
    }
  }'

# Expected error:
# {"error":"ValidationError","message":"Node cannot be both Person and Organization (disjoint classes)"}
```

**🎓 What You Learned**:
- The ontology prevents logically inconsistent data
- Disjoint classes create mutual exclusion rules
- Validation happens before data enters the database

### 2.5 Examine Semantic Physics Translation

Open `src/ontology/semantic_physics.rs`:

```rust
/// Translates ontology constraints into 3D physics forces
pub struct SemanticPhysicsEngine {
    reasoner: Arc<WhelkReasoner>,
}

impl SemanticPhysicsEngine {
    /// Convert ontology axioms to physics forces
    pub fn generate_forces(&self, nodes: &[Node]) -> Vec<Force> {
        let mut forces = Vec::new();

        // For each pair of nodes, check ontology relationships
        for (node_a, node_b) in nodes.iter().tuple_combinations() {
            // Disjoint classes → Repulsion force
            if self.reasoner.are_disjoint(&node_a.node_type, &node_b.node_type) {
                forces.push(Force::Repulsion {
                    from: node_a.id,
                    to: node_b.id,
                    strength: 500.0,
                });
            }

            // SubClassOf → Weak attraction (hierarchical clustering)
            if self.reasoner.is_subclass_of(&node_a.node_type, &node_b.node_type) {
                forces.push(Force::Attraction {
                    from: node_a.id,
                    to: node_b.id,
                    strength: 0.3,
                });
            }
        }

        forces
    }
}
```

**🎓 What You Learned**:
- **Eight constraint types** translate to physics forces:
  - `DisjointWith` → Strong repulsion
  - `SubClassOf` → Weak attraction
  - `EquivalentTo` → Very strong attraction
  - `Domain/Range` → Directional forces
- This is what makes the 3D visualization "intelligent"—it organizes itself based on logical rules

**Practice Exercise**: Find where these forces are passed to the GPU layer. Hint: Look in `src/gpu/physics.rs`.

---

## Layer 3: GPU Compute Integration

**Duration**: 45 minutes
**Focus**: Understanding hardware acceleration

### 3.1 GPU Architecture Overview

VisionFlow uses **39 production CUDA kernels** for:

1. **Physics simulation** (10 kernels): Forces, velocities, positions
2. **Semantic forces** (8 kernels): Ontology-based attractions/repulsions
3. **Clustering** (12 kernels): Community detection, hierarchy
4. **Pathfinding** (5 kernels): Shortest paths, traversal
5. **Utilities** (4 kernels): Sorting, reduction, memory ops

**Performance**: ~100x speedup vs CPU for large graphs (10,000+ nodes)

### 3.2 Locate GPU Code

```bash
# View GPU directory structure
tree src/gpu/

# Expected:
src/gpu/
├── mod.rs                    # Module exports and feature flags
├── physics.rs                # Main physics engine
├── semantic_analyzer.rs      # Semantic force computation
├── kernels/                  # CUDA kernel implementations
│   ├── forces.cu             # Force calculation kernels
│   ├── integration.cu        # Velocity/position integration
│   └── clustering.cu         # Community detection
└── adapters/
    ├── gpu_physics_adapter.rs   # Port to GPU hardware
    └── gpu_semantic_adapter.rs  # Semantic forces port
```

**Note**: The actual CUDA kernels (`.cu` files) are compiled separately using `nvcc` (NVIDIA CUDA Compiler).

### 3.3 Examine the Physics Engine

Open `src/gpu/physics.rs`:

```rust
use cudarc::driver::{CudaDevice, LaunchAsync, LaunchConfig};

pub struct GpuPhysicsEngine {
    device: Arc<CudaDevice>,
    force_kernel: CudaFunction,
    integration_kernel: CudaFunction,
    nodes_gpu: CudaBuffer<Node>,
    forces_gpu: CudaBuffer<Force>,
}

impl GpuPhysicsEngine {
    pub fn new() -> Result<Self, GpuError> {
        // Initialize CUDA device
        let device = CudaDevice::new(0)?; // GPU 0

        // Load compiled kernels
        let force_kernel = device.get_func("compute_forces_kernel")?;
        let integration_kernel = device.get_func("integrate_kernel")?;

        Ok(Self {
            device,
            force_kernel,
            integration_kernel,
            nodes_gpu: CudaBuffer::new(),
            forces_gpu: CudaBuffer::new(),
        })
    }

    /// Run one physics simulation step
    pub async fn step(&mut self, nodes: &mut [Node], forces: &[Force], dt: f32) -> Result<(), GpuError> {
        // 1. Copy data to GPU
        self.nodes_gpu.copy_from_host(nodes)?;
        self.forces_gpu.copy_from_host(forces)?;

        // 2. Launch force computation kernel
        let grid_size = (nodes.len() + 255) / 256;
        let launch_config = LaunchConfig {
            grid_dim: (grid_size as u32, 1, 1),
            block_dim: (256, 1, 1),
            shared_mem_bytes: 0,
        };

        unsafe {
            self.force_kernel.launch(
                launch_config,
                (&self.nodes_gpu, &self.forces_gpu, nodes.len()),
            )?;
        }

        // 3. Launch integration kernel (update positions)
        unsafe {
            self.integration_kernel.launch(
                launch_config,
                (&mut self.nodes_gpu, &self.forces_gpu, dt),
            )?;
        }

        // 4. Copy results back to CPU
        self.nodes_gpu.copy_to_host(nodes)?;

        Ok(())
    }
}
```

**🎓 What You Learned**:
- **CUDA workflow**: Copy data to GPU → Launch kernels → Copy results back
- **Kernel launches** specify grid and block dimensions (parallelism config)
- **async/await** used for overlapping compute and data transfer

### 3.4 Examine a CUDA Kernel (Optional)

If you have GPU development experience, look at `src/gpu/kernels/forces.cu`:

```cuda
// CUDA kernel: compute forces between nodes
__global__ void compute_forces_kernel(
    Node* nodes,
    Force* forces,
    int num_nodes
) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx >= num_nodes) return;

    Node& node = nodes[idx];
    float3 total_force = make_float3(0.0f, 0.0f, 0.0f);

    // Compute repulsion from all other nodes (O(n²) but parallelized)
    for (int i = 0; i < num_nodes; i++) {
        if (i == idx) continue;

        Node& other = nodes[i];
        float3 delta = make_float3(
            other.position.x - node.position.x,
            other.position.y - node.position.y,
            other.position.z - node.position.z
        );

        float distance = length(delta);
        if (distance < 0.01f) distance = 0.01f; // Avoid division by zero

        // Coulomb's law: F = k / r²
        float repulsion = 100.0f / (distance * distance);
        float3 direction = normalize(delta);
        total_force -= direction * repulsion; // Repel
    }

    // Apply spring forces from connected edges
    // (Stored in shared memory for performance)
    // ...

    // Write result
    forces[idx] = make_force(total_force);
}
```

**🎓 What You Learned**:
- Each CUDA thread handles one node
- Thousands of nodes computed in parallel
- Physics formulae (Coulomb, Hooke's law) implemented directly

**Performance Note**: Without GPU, a 10,000-node graph would take ~500ms per frame. With GPU: ~5ms (100x faster).

### 3.5 Check GPU Availability

```bash
# Check if GPU is detected
docker exec visionflow-container nvidia-smi

# Expected (if GPU available):
# +-----------------------------------------------------------------------------+
# | NVIDIA-SMI 535.54       Driver Version: 535.54       CUDA Version: 12.2    |
# |-------------------------------+----------------------+----------------------+
# | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
# | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
# |===============================+======================+======================|
# |   0  NVIDIA GeForce ... Off  | 00000000:01:00.0  On |                  N/A |
# | 30%   45C    P8    12W / 320W |    500MiB / 16384MiB |      0%      Default |
# +-------------------------------+----------------------+----------------------+

# If not available:
# Error: nvidia-smi not found
```

**Note**: GPU is optional. VisionFlow falls back to CPU physics if no GPU is detected.

---

## Layer 4: Client Synchronization (Binary WebSocket Protocol)

**Duration**: 45 minutes
**Focus**: Real-time updates at 60 FPS

### 4.1 Why Binary WebSocket?

**Problem**: JSON WebSocket is too slow for real-time 3D at scale
- 1000 nodes × 60 FPS = 60,000 updates/second
- JSON encoding: ~200 bytes per node update
- Total bandwidth: 12 MB/s (too much!)

**Solution**: Custom binary protocol
- Binary encoding: **36 bytes per node update**
- Total bandwidth: 2.16 MB/s (5.5x reduction)
- Enables 60 FPS rendering at 100,000+ nodes

### 4.2 Locate Protocol Code

```bash
# Server-side protocol
ls -lh src/protocols/

# Expected:
# binary_websocket.rs     - Protocol implementation
# protocol_v2.rs          - V2 specification (36-byte format)
# serialization.rs        - Encoding/decoding

# Client-side protocol
ls -lh client/src/services/websocket/

# Expected:
# BinaryWebSocketProtocol.ts
# decoder.ts
# encoder.ts
```

### 4.3 Understand the Binary Format

Open `src/protocols/protocol_v2.rs`:

```rust
/// Binary WebSocket Protocol V2
/// Total size: 36 bytes per node update

#[repr(C, packed)]
pub struct NodeUpdate {
    pub node_id: u64,      // 8 bytes - unique identifier
    pub flags: u32,        // 4 bytes - bit flags
    pub position_x: f32,   // 4 bytes - X coordinate
    pub position_y: f32,   // 4 bytes - Y coordinate
    pub position_z: f32,   // 4 bytes - Z coordinate
    pub velocity_x: f32,   // 4 bytes - velocity vector
    pub velocity_y: f32,   // 4 bytes
    pub velocity_z: f32,   // 4 bytes
}
// Total: 36 bytes

/// Flags bitfield (32 bits)
pub mod flags {
    pub const NODE_TYPE_MASK: u32 = 0x0000_00FF; // Bits 0-7: node type ID
    pub const SELECTED: u32       = 0x0000_0100; // Bit 8: selected state
    pub const HIGHLIGHTED: u32    = 0x0000_0200; // Bit 9: highlighted
    pub const AGENT_NODE: u32     = 0x8000_0000; // Bit 31: AI agent node
    pub const KNOWLEDGE_NODE: u32 = 0x4000_0000; // Bit 30: Knowledge graph node
}

impl NodeUpdate {
    /// Serialize to bytes
    pub fn to_bytes(&self) -> [u8; 36] {
        unsafe {
            std::mem::transmute(*self)
        }
    }

    /// Deserialize from bytes
    pub fn from_bytes(bytes: &[u8; 36]) -> Self {
        unsafe {
            std::mem::transmute(*bytes)
        }
    }
}
```

**🎓 What You Learned**:
- **Fixed-size binary format**: No parsing overhead
- **Bit flags**: Pack multiple booleans into 32 bits
- **`#[repr(C, packed)]`**: Ensures no padding, matches C layout
- **unsafe transmute**: Zero-copy serialization (very fast)

### 4.4 Trace WebSocket Updates

Open `src/adapters/websocket_adapter.rs`:

```rust
use actix_web_actors::ws;

pub struct WebSocketSession {
    node_updates: UnboundedReceiver<NodeUpdate>,
}

impl StreamHandler<Result<ws::Message, ws::ProtocolError>> for WebSocketSession {
    fn handle(&mut self, msg: Result<ws::Message, ws::ProtocolError>, ctx: &mut Self::Context) {
        // Handle incoming messages from client
        match msg {
            Ok(ws::Message::Binary(bin)) => {
                // Decode binary message (e.g., user interaction)
                let interaction = UserInteraction::from_bytes(&bin);
                self.handle_interaction(interaction);
            }
            _ => {}
        }
    }
}

impl WebSocketSession {
    /// Broadcast node updates to all connected clients
    pub async fn broadcast_updates(&mut self, ctx: &mut ws::WebsocketContext<Self>) {
        // Collect all pending updates
        let mut updates = Vec::new();
        while let Ok(update) = self.node_updates.try_recv() {
            updates.push(update);
        }

        if updates.is_empty() {
            return;
        }

        // Serialize to binary
        let mut buffer = Vec::with_capacity(updates.len() * 36);
        for update in updates {
            buffer.extend_from_slice(&update.to_bytes());
        }

        // Send binary WebSocket message
        ctx.binary(buffer);
    }
}
```

**🎓 What You Learned**:
- **Batch updates**: Multiple nodes sent in one WebSocket message
- **Try_recv**: Non-blocking, processes all available updates
- **Binary WebSocket**: Uses `ctx.binary()` instead of `ctx.text()`

### 4.5 Examine Client Decoder

Open `client/src/services/websocket/decoder.ts`:

```typescript
// Client-side binary decoder
export class BinaryDecoder {
  // Decode binary WebSocket message into node updates
  decodeNodeUpdates(buffer: ArrayBuffer): NodeUpdate[] {
    const view = new DataView(buffer);
    const updateCount = buffer.byteLength / 36; // Each update is 36 bytes
    const updates: NodeUpdate[] = [];

    for (let i = 0; i < updateCount; i++) {
      const offset = i * 36;

      // Read fields using DataView (handles endianness)
      const nodeId = view.getBigUint64(offset + 0, true); // true = little-endian
      const flags = view.getUint32(offset + 8, true);
      const posX = view.getFloat32(offset + 12, true);
      const posY = view.getFloat32(offset + 16, true);
      const posZ = view.getFloat32(offset + 20, true);
      const velX = view.getFloat32(offset + 24, true);
      const velY = view.getFloat32(offset + 28, true);
      const velZ = view.getFloat32(offset + 32, true);

      // Check bit flags
      const isAgent = (flags & 0x80000000) !== 0; // Bit 31
      const isKnowledgeNode = (flags & 0x40000000) !== 0; // Bit 30

      updates.push({
        nodeId: nodeId.toString(),
        flags,
        position: { x: posX, y: posY, z: posZ },
        velocity: { x: velX, y: velY, z: velZ },
        isAgent,
        isKnowledgeNode,
      });
    }

    return updates;
  }
}
```

**🎓 What You Learned**:
- **DataView**: JavaScript API for reading binary data
- **Endianness**: Little-endian (Intel/AMD standard)
- **Bit masking**: Extract individual flags from packed bits

### 4.6 Test WebSocket Communication

Open browser DevTools (F12) on `http://localhost:3030`:

```javascript
// Connect to WebSocket
const ws = new WebSocket('ws://localhost:3030/ws');
ws.binaryType = 'arraybuffer';

// Listen for binary messages
ws.onmessage = (event) => {
  if (event.data instanceof ArrayBuffer) {
    const view = new DataView(event.data);
    console.log(`Received ${event.data.byteLength} bytes`);
    console.log(`Updates: ${event.data.byteLength / 36}`);

    // Decode first update
    if (event.data.byteLength >= 36) {
      const nodeId = view.getBigUint64(0, true);
      const posX = view.getFloat32(12, true);
      const posY = view.getFloat32(16, true);
      const posZ = view.getFloat32(20, true);
      console.log(`Node ${nodeId}: position (${posX}, ${posY}, ${posZ})`);
    }
  }
};

// Send a command (e.g., select node)
const selectNode = new Uint8Array(12);
const view = new DataView(selectNode.buffer);
view.setUint32(0, 1, true); // Command: SELECT_NODE
view.setBigUint64(4, 42n, true); // Node ID: 42
ws.send(selectNode);
```

**🎓 What You Learned**:
- Real-time inspection of WebSocket traffic
- Binary data flows continuously at 60 FPS
- You can send commands back to the server

---

## 🧪 Hands-On Exercises

### Exercise 1: Add a New Query

**Goal**: Add an endpoint to count nodes by type

1. **Define query** in `src/cqrs/queries.rs`:
```rust
pub struct CountNodesByTypeQuery {
    pub node_type: String,
}
```

2. **Implement in service** `src/services/graph_service.rs`:
```rust
pub async fn count_nodes_by_type(&self, query: CountNodesByTypeQuery) -> Result<usize, ServiceError> {
    self.repository.count_by_type(&query.node_type).await
}
```

3. **Add HTTP endpoint** in `src/adapters/http_adapter.rs`:
```rust
#[get("/api/nodes/count/{type}")]
async fn count_nodes(
    type_param: web::Path<String>,
    graph_service: web::Data<GraphService>,
) -> Result<HttpResponse, Error> {
    let count = graph_service.count_nodes_by_type(CountNodesByTypeQuery {
        node_type: type_param.into_inner(),
    }).await?;
    Ok(HttpResponse::Ok().json(count))
}
```

4. **Test**:
```bash
curl http://localhost:3030/api/nodes/count/Concept
# Expected: 42
```

### Exercise 2: Trace a Complete User Action

**Scenario**: User clicks a node in the UI

1. **Client** (`client/src/components/Graph.vue`): Click handler sends WebSocket message
2. **WebSocket Adapter** (`src/adapters/websocket_adapter.rs`): Receives binary message
3. **Service** (`src/services/graph_service.rs`): Updates node state
4. **Repository** (`src/repositories/graph_repository.rs`): Persists to SQLite
5. **Broadcast**: Sends update to all connected clients
6. **Client** receives update and highlights node

**Task**: Trace this flow through the actual source files.

### Exercise 3: Modify Ontology Rules

**Goal**: Add a new semantic constraint and see it affect the 3D visualization

1. Create ontology file with new rule:
```xml
<DisjointClasses>
  <Class IRI="#Software"/>
  <Class IRI="#Hardware"/>
</DisjointClasses>
```

2. Load via API
3. Create nodes of each type
4. Observe repulsion in 3D view

---

## 🎓 What You've Mastered

You now understand:

✅ **Hexagonal Architecture**: How outer adapters isolate core logic
✅ **CQRS Pattern**: Separation of reads (queries) and writes (directives)
✅ **Ontology Reasoning**: Whelk-rs validation and inference
✅ **GPU Acceleration**: CUDA kernel integration for 100x speedup
✅ **Binary Protocol**: Real-time synchronization at 60 FPS
✅ **Data Flow**: How a user action flows through all layers

---

## 🚀 Next Steps

You've mastered the architecture! Now apply this knowledge:

### Build Something (Recommended)
**[→ Phase 3: Feature Implementation](./phase-3-build-feature.md)**
Implement a custom semantic physics constraint (spans all layers!)

### Dive Deeper into Specific Topics
- [Services Architecture](../../concepts/architecture/services-architecture.md) - Detailed service layer patterns
- [GPU Semantic Forces](../../concepts/architecture/gpu-semantic-forces.md) - Complete GPU documentation
- [WebSocket Protocol Spec](../../concepts/architecture/components/websocket-protocol.md) - Protocol V2 specification

### Start Contributing
- [Development Workflow](../../guides/development-workflow.md) - Git workflow and PR process
- [Testing Guide](../../guides/developer/05-testing-guide.md) - Writing tests
- [Contributing Guidelines](../../guides/developer/06-contributing.md) - Contribution standards

---

## 📚 Reference Materials

- [Project Structure](../../guides/developer/02-project-structure.md) - Complete directory reference
- [Architecture Overview](../../concepts/architecture/00-architecture-overview.md) - High-level architecture docs
- [API Reference](../../reference/api/) - REST API documentation

---

## 📝 Tutorial Metadata

- **Phase**: 2 of 3 (Core Architecture Deep Dive)
- **Difficulty**: Intermediate
- **Time Required**: 3-4 hours
- **Prerequisites**: Phase 1, basic Rust/JS knowledge
- **Last Updated**: 2025-11-05

---

**Navigation**: [← Phase 1](./phase-1-foundation.md) | [Learning Plan](./README.md) | [Phase 3 →](./phase-3-build-feature.md)
