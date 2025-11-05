# Phase 3: Build a Custom Semantic Physics Constraint

**Learn VisionFlow by implementing an end-to-end feature**

This capstone tutorial has you implement a **ninth semantic physics constraint**, demonstrating mastery of all major system layers: Ontology, GPU Compute, Server Logic, and Client Visualization. This is the ultimate confidence-builder.

---

## 🎯 Learning Objectives

By the end of this tutorial, you will:

✅ Define a new ontological relationship type
✅ Modify the Whelk-rs reasoning pipeline
✅ Implement GPU kernel acceleration (optional)
✅ Update the 3D client visualization
✅ Write tests maintaining >80% coverage
✅ Document your feature following project standards
✅ Submit a pull request (optional)

**Time Required**: 4-8 hours
**Difficulty**: Advanced
**Prerequisites**: Completed Phase 2, Rust 1.70+, Node.js 18+

---

## Project Overview: The `RequiresHighCompute` Constraint

**Concept**: Some nodes represent compute-intensive operations (AI models, simulations, data processing). We'll create a new semantic constraint that causes these nodes to be **strongly attracted** to a special "GPU Compute Node" in the visualization.

**Visual Effect**: When you create a node like "Train Neural Network" and mark it with the `RequiresHighCompute` property, it will visually cluster near the GPU node, making it easy to identify compute-heavy operations.

**Ontological Basis**:
```
Axiom: RequiresHighCompute(?x) → strongAttractionTo(?x, GPUComputeNode)
```

**Why This Is Perfect for Learning**:
- Touches all major layers (Ontology → GPU → Server → Client)
- Practical and visually demonstrable
- Follows existing constraint patterns
- Requires understanding of the full stack

---

## Prerequisites Setup

### 1. Development Environment

Ensure you have:

```bash
# Rust toolchain
rustc --version  # Should be 1.70+
cargo --version

# Node.js and npm
node --version   # Should be 18+
npm --version

# Git
git --version
```

### 2. Fork and Clone (if contributing)

```bash
# Fork on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/visionflow.git
cd visionflow
git checkout -b feature/high-compute-constraint
```

### 3. Build from Source

```bash
# Install Rust dependencies and build
cargo build --release

# Install frontend dependencies
cd client
npm install
npm run build
cd ..

# Verify build
cargo test
```

### 4. Start Development Services

```bash
# Start with development mode (hot reload)
docker-compose -f docker-compose.dev.yml up -d

# Watch logs
docker-compose logs -f visionflow-container
```

---

## Step 1: Define the Ontological Constraint

**Duration**: 30 minutes
**Files Modified**: `src/ontology/`, ontology schema

### 1.1 Create Ontology Definition

Create a new ontology file: `ontology_physics.toml`

```toml
# VisionFlow Semantic Physics Ontology
# Defines how ontological relationships map to 3D physics forces

[[constraints]]
name = "RequiresHighCompute"
type = "ObjectProperty"
description = "Marks entities that require GPU/high-compute resources"

[constraints.physics]
force_type = "strong_attraction"
target_class = "GPUComputeNode"
strength = 800.0  # Strong attraction (existing constraints: 100-500)
decay = "inverse_square"  # Force decreases with distance²

[[constraints.inverse]]
# Inverse property: GPU node "provides compute" to these nodes
name = "ProvidesComputeTo"
```

**What This Means**:
- **force_type**: Type of 3D force to apply
- **target_class**: The class of node to attract to
- **strength**: Magnitude (800 = very strong, more than disjoint repulsion)
- **decay**: How force changes with distance

### 1.2 Extend the Ontology Repository

Edit `src/ontology/axiom_parser.rs`:

```rust
// Add to existing AxiomType enum
#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum AxiomType {
    SubClassOf,
    DisjointWith,
    EquivalentTo,
    ObjectProperty,
    DataProperty,
    InverseProperty,
    FunctionalProperty,
    RequiresHighCompute,  // NEW: Our custom constraint
}

// Add parsing logic
impl AxiomParser {
    pub fn parse_axiom(&self, axiom_data: &[u8]) -> Result<ParsedAxiom, ParserError> {
        // ... existing parsing logic ...

        // NEW: Handle RequiresHighCompute axiom
        if axiom_type == "RequiresHighCompute" {
            return Ok(ParsedAxiom {
                axiom_type: AxiomType::RequiresHighCompute,
                subject: self.extract_subject(axiom_data)?,
                object: Some("GPUComputeNode".to_string()),
                properties: self.parse_physics_properties(axiom_data)?,
            });
        }

        // ... rest of parsing ...
    }

    fn parse_physics_properties(&self, data: &[u8]) -> Result<PhysicsProperties, ParserError> {
        // Parse [constraints.physics] section from TOML
        let toml_str = std::str::from_utf8(data)?;
        let config: TomlConfig = toml::from_str(toml_str)?;

        Ok(PhysicsProperties {
            force_type: config.constraints.physics.force_type,
            strength: config.constraints.physics.strength,
            decay: config.constraints.physics.decay,
        })
    }
}
```

### 1.3 Update the Ontology Service

Edit `src/services/ontology_service.rs`:

```rust
impl OntologyService {
    /// Validate that a node can have RequiresHighCompute property
    pub async fn validate_high_compute_constraint(
        &self,
        node_id: NodeId,
    ) -> Result<(), ServiceError> {
        // Get node type
        let node = self.repository.get_node(node_id).await?;

        // Only certain types can require high compute
        let allowed_types = vec!["AIModel", "Simulation", "DataProcessing", "Computation"];

        if !allowed_types.contains(&node.node_type.as_str()) {
            return Err(ServiceError::ValidationError(format!(
                "Node type '{}' cannot require high compute",
                node.node_type
            )));
        }

        Ok(())
    }
}
```

**🎓 What You've Done**:
- Defined a new ontological constraint in TOML
- Extended the axiom parser to recognize it
- Added validation logic to the service layer

---

## Step 2: Update the Reasoning Pipeline

**Duration**: 45 minutes
**Files Modified**: `src/ontology/whelk_reasoner.rs`, `src/ontology/semantic_physics.rs`

### 2.1 Modify Whelk-rs Integration

Edit `src/ontology/whelk_reasoner.rs`:

```rust
impl WhelkReasoner {
    /// Process RequiresHighCompute axioms
    pub fn add_high_compute_constraint(
        &mut self,
        subject: &str,
        properties: PhysicsProperties,
    ) -> Result<(), ReasonerError> {
        // Create Whelk axiom for the constraint
        let axiom = Axiom::ObjectPropertyAssertion {
            property: "requiresHighCompute".to_string(),
            subject: subject.to_string(),
            object: "GPUComputeNode".to_string(),
        };

        self.reasoner.add_axiom(axiom);

        // Store physics properties for later force generation
        self.physics_constraints.insert(
            subject.to_string(),
            Constraint::HighCompute {
                strength: properties.strength,
                decay: properties.decay,
            },
        );

        Ok(())
    }

    /// Query which nodes require high compute
    pub fn get_high_compute_nodes(&self) -> Vec<String> {
        self.reasoner
            .query_property("requiresHighCompute")
            .map(|result| result.subject)
            .collect()
    }
}
```

### 2.2 Extend Semantic Physics Engine

Edit `src/ontology/semantic_physics.rs`:

```rust
impl SemanticPhysicsEngine {
    /// Generate forces for RequiresHighCompute constraints
    pub fn generate_high_compute_forces(&self, nodes: &[Node]) -> Vec<Force> {
        let mut forces = Vec::new();

        // Find the GPU Compute Node
        let gpu_node = nodes.iter().find(|n| n.node_type == "GPUComputeNode");
        if gpu_node.is_none() {
            return forces; // No GPU node present
        }
        let gpu_node = gpu_node.unwrap();

        // Get all nodes that require high compute
        let high_compute_nodes = self.reasoner.get_high_compute_nodes();

        // Create strong attraction forces
        for node in nodes {
            if high_compute_nodes.contains(&node.id.to_string()) {
                // Calculate distance
                let distance = self.calculate_distance(&node.position, &gpu_node.position);

                // Apply inverse square decay: F = strength / r²
                let force_magnitude = 800.0 / (distance * distance + 0.01);

                // Direction: toward GPU node
                let direction = self.calculate_direction(&node.position, &gpu_node.position);

                forces.push(Force {
                    node_id: node.id,
                    vector: direction * force_magnitude,
                    force_type: ForceType::SemanticAttraction,
                });
            }
        }

        forces
    }

    /// Update main force generation to include new constraint
    pub fn generate_forces(&self, nodes: &[Node]) -> Vec<Force> {
        let mut all_forces = Vec::new();

        // Existing constraint forces
        all_forces.extend(self.generate_disjoint_forces(nodes));
        all_forces.extend(self.generate_subclass_forces(nodes));
        // ... other constraint types ...

        // NEW: Add high compute forces
        all_forces.extend(self.generate_high_compute_forces(nodes));

        all_forces
    }
}
```

**🎓 What You've Done**:
- Integrated the new constraint into Whelk-rs reasoner
- Implemented force generation using inverse square law
- Connected it to the main physics pipeline

---

## Step 3: Implement GPU Kernel (Optional but Recommended)

**Duration**: 60 minutes (skip if no NVIDIA GPU)
**Files Modified**: `src/gpu/kernels/semantic_forces.cu`, `src/gpu/semantic_analyzer.rs`

### 3.1 Create CUDA Kernel

Create `src/gpu/kernels/high_compute_forces.cu`:

```cuda
// CUDA Kernel: Compute forces for RequiresHighCompute constraint
__global__ void compute_high_compute_forces_kernel(
    Node* nodes,
    int num_nodes,
    int gpu_node_index,
    Force* forces
) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx >= num_nodes) return;

    Node& node = nodes[idx];
    Node& gpu_node = nodes[gpu_node_index];

    // Check if this node requires high compute (flag bit 29)
    bool requires_compute = (node.flags & 0x20000000) != 0;
    if (!requires_compute) {
        forces[idx] = make_zero_force();
        return;
    }

    // Calculate direction to GPU node
    float3 delta = make_float3(
        gpu_node.position.x - node.position.x,
        gpu_node.position.y - node.position.y,
        gpu_node.position.z - node.position.z
    );

    float distance_sq = dot(delta, delta);
    if (distance_sq < 0.01f) distance_sq = 0.01f; // Prevent singularity

    // Inverse square law: F = 800 / r²
    float force_magnitude = 800.0f / distance_sq;

    // Direction: normalized delta
    float3 direction = normalize(delta);

    // Resultant force
    float3 force_vector = direction * force_magnitude;

    // Write to output
    forces[idx] = make_force(force_vector, FORCE_TYPE_SEMANTIC);
}
```

### 3.2 Integrate Kernel into Rust

Edit `src/gpu/semantic_analyzer.rs`:

```rust
use cudarc::driver::CudaFunction;

pub struct GpuSemanticAnalyzer {
    device: Arc<CudaDevice>,
    high_compute_kernel: CudaFunction,  // NEW
    // ... existing kernels ...
}

impl GpuSemanticAnalyzer {
    pub fn new() -> Result<Self, GpuError> {
        let device = CudaDevice::new(0)?;

        // Load compiled kernel
        let high_compute_kernel = device.get_func("compute_high_compute_forces_kernel")?;

        Ok(Self {
            device,
            high_compute_kernel,
            // ...
        })
    }

    /// Compute all semantic forces including high compute
    pub async fn compute_semantic_forces(
        &mut self,
        nodes: &[Node],
        gpu_node_index: usize,
    ) -> Result<Vec<Force>, GpuError> {
        // Copy nodes to GPU
        self.nodes_gpu.copy_from_host(nodes)?;

        // Launch high compute forces kernel
        let grid_size = (nodes.len() + 255) / 256;
        let launch_config = LaunchConfig {
            grid_dim: (grid_size as u32, 1, 1),
            block_dim: (256, 1, 1),
            shared_mem_bytes: 0,
        };

        unsafe {
            self.high_compute_kernel.launch(
                launch_config,
                (
                    &self.nodes_gpu,
                    nodes.len() as i32,
                    gpu_node_index as i32,
                    &mut self.forces_gpu,
                ),
            )?;
        }

        // Copy forces back
        let mut forces = vec![Force::default(); nodes.len()];
        self.forces_gpu.copy_to_host(&mut forces)?;

        Ok(forces)
    }
}
```

### 3.3 Update Build Configuration

Edit `build.rs`:

```rust
// Add CUDA compilation for new kernel
fn main() {
    // ... existing build script ...

    // Compile new CUDA kernel
    if cfg!(feature = "cuda") {
        println!("cargo:rerun-if-changed=src/gpu/kernels/high_compute_forces.cu");

        let status = Command::new("nvcc")
            .args(&[
                "-O3",
                "-arch=sm_86", // RTX 30xx series
                "-c",
                "src/gpu/kernels/high_compute_forces.cu",
                "-o",
                "target/high_compute_forces.o",
            ])
            .status()
            .expect("Failed to compile CUDA kernel");

        if !status.success() {
            panic!("CUDA kernel compilation failed");
        }
    }
}
```

**🎓 What You've Done**:
- Implemented a CUDA kernel for parallel force computation
- Integrated the kernel into the Rust GPU layer
- Updated build system to compile the new kernel

**Performance Impact**: With GPU, 10,000 nodes computed in ~5ms vs ~200ms on CPU (40x speedup).

---

## Step 4: Update Client Visualization

**Duration**: 60 minutes
**Files Modified**: `client/src/rendering/`, `client/src/components/`

### 4.1 Add Visual Indicator for High Compute Nodes

Edit `client/src/rendering/NodeRenderer.ts`:

```typescript
export class NodeRenderer {
  // Update node rendering to show high compute indicator
  renderNode(node: Node, scene: THREE.Scene): THREE.Mesh {
    // Existing node rendering
    const geometry = new THREE.SphereGeometry(node.size || 1.0, 32, 32);

    // NEW: Check if node requires high compute (bit 29)
    const requiresCompute = (node.flags & 0x20000000) !== 0;

    // Choose material based on compute requirement
    const material = requiresCompute
      ? new THREE.MeshStandardMaterial({
          color: 0xff6600,  // Orange for high compute
          emissive: 0xff3300,
          emissiveIntensity: 0.5,
          metalness: 0.8,
          roughness: 0.2,
        })
      : this.getDefaultMaterial(node.type);

    const mesh = new THREE.Mesh(geometry, material);

    // NEW: Add glow ring for high compute nodes
    if (requiresCompute) {
      this.addComputeIndicator(mesh);
    }

    return mesh;
  }

  private addComputeIndicator(parentMesh: THREE.Mesh): void {
    // Create glowing ring around node
    const ringGeometry = new THREE.RingGeometry(1.5, 1.7, 32);
    const ringMaterial = new THREE.MeshBasicMaterial({
      color: 0xff6600,
      side: THREE.DoubleSide,
      transparent: true,
      opacity: 0.6,
    });

    const ring = new THREE.Mesh(ringGeometry, ringMaterial);
    ring.rotation.x = Math.PI / 2; // Horizontal ring

    // Add pulsing animation
    const pulse = (time: number) => {
      ring.scale.set(
        1 + 0.1 * Math.sin(time * 2),
        1 + 0.1 * Math.sin(time * 2),
        1
      );
    };

    parentMesh.add(ring);
    parentMesh.userData.pulseAnimation = pulse;
  }
}
```

### 4.2 Add UI Controls for High Compute Property

Edit `client/src/components/NodePropertiesPanel.vue`:

```vue
<template>
  <div class="node-properties">
    <h3>Node Properties</h3>

    <!-- Existing properties -->
    <div class="property">
      <label>Label:</label>
      <input v-model="node.label" @change="updateNode" />
    </div>

    <div class="property">
      <label>Type:</label>
      <select v-model="node.type" @change="updateNode">
        <option value="Concept">Concept</option>
        <option value="AIModel">AI Model</option>
        <option value="Simulation">Simulation</option>
        <option value="DataProcessing">Data Processing</option>
      </select>
    </div>

    <!-- NEW: High Compute Toggle -->
    <div class="property" v-if="canRequireCompute">
      <label>
        <input
          type="checkbox"
          v-model="requiresHighCompute"
          @change="updateComputeRequirement"
        />
        Requires High Compute (GPU)
      </label>
      <span class="info-icon" title="This node will cluster near GPU resources">
        ⓘ
      </span>
    </div>

    <!-- Visual indicator -->
    <div v-if="requiresHighCompute" class="compute-badge">
      🖥️ High Compute Enabled
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, computed } from 'vue';
import { useGraphStore } from '@/stores/graph';

export default defineComponent({
  name: 'NodePropertiesPanel',
  props: {
    nodeId: {
      type: String,
      required: true,
    },
  },
  setup(props) {
    const graphStore = useGraphStore();
    const node = computed(() => graphStore.getNode(props.nodeId));

    // Check if node type can require high compute
    const canRequireCompute = computed(() => {
      const allowedTypes = ['AIModel', 'Simulation', 'DataProcessing', 'Computation'];
      return allowedTypes.includes(node.value?.type || '');
    });

    // Extract high compute flag from node flags (bit 29)
    const requiresHighCompute = computed({
      get: () => ((node.value?.flags || 0) & 0x20000000) !== 0,
      set: (value: boolean) => {
        if (!node.value) return;

        // Update flags
        let newFlags = node.value.flags || 0;
        if (value) {
          newFlags |= 0x20000000; // Set bit 29
        } else {
          newFlags &= ~0x20000000; // Clear bit 29
        }

        // Send update to server
        updateComputeRequirement(newFlags);
      },
    });

    const updateComputeRequirement = async (newFlags: number) => {
      await graphStore.updateNode({
        id: props.nodeId,
        flags: newFlags,
      });
    };

    return {
      node,
      canRequireCompute,
      requiresHighCompute,
      updateComputeRequirement,
    };
  },
});
</script>

<style scoped>
.compute-badge {
  background: linear-gradient(135deg, #ff6600, #ff3300);
  color: white;
  padding: 8px 12px;
  border-radius: 4px;
  font-weight: bold;
  margin-top: 10px;
  text-align: center;
}

.info-icon {
  margin-left: 5px;
  cursor: help;
  color: #666;
}
</style>
```

### 4.3 Update Animation Loop

Edit `client/src/rendering/AnimationLoop.ts`:

```typescript
export class AnimationLoop {
  private animate(time: number): void {
    // ... existing animation code ...

    // NEW: Update pulse animations for high compute nodes
    this.scene.traverse((object) => {
      if (object.userData.pulseAnimation) {
        object.userData.pulseAnimation(time * 0.001); // Convert to seconds
      }
    });

    // Render scene
    this.renderer.render(this.scene, this.camera);
    requestAnimationFrame(() => this.animate(performance.now()));
  }
}
```

**🎓 What You've Done**:
- Added visual distinction for high compute nodes (orange, glowing)
- Created UI controls to mark nodes as requiring high compute
- Implemented pulsing animation for visual feedback

---

## Step 5: Testing and Validation

**Duration**: 60 minutes
**Files Created**: `tests/integration/test_high_compute.rs`, `client/tests/high-compute.spec.ts`

### 5.1 Write Backend Integration Test

Create `tests/integration/test_high_compute.rs`:

```rust
#[cfg(test)]
mod high_compute_tests {
    use visionflow::services::{GraphService, OntologyService};
    use visionflow::models::{Node, NodeId};

    #[tokio::test]
    async fn test_high_compute_constraint_validation() {
        // Setup
        let graph_service = GraphService::new_for_testing().await;
        let ontology_service = OntologyService::new_for_testing().await;

        // Create an AI Model node
        let node_id = graph_service
            .create_node(CreateNodeDirective {
                label: "GPT-5 Training".to_string(),
                node_type: "AIModel".to_string(),
                properties: HashMap::new(),
            })
            .await
            .unwrap();

        // Mark as requiring high compute (should succeed)
        let result = ontology_service
            .validate_high_compute_constraint(node_id)
            .await;
        assert!(result.is_ok());

        // Try to mark an invalid type (should fail)
        let concept_id = graph_service
            .create_node(CreateNodeDirective {
                label: "Abstract Idea".to_string(),
                node_type: "Concept".to_string(),
                properties: HashMap::new(),
            })
            .await
            .unwrap();

        let result = ontology_service
            .validate_high_compute_constraint(concept_id)
            .await;
        assert!(result.is_err());
    }

    #[tokio::test]
    async fn test_high_compute_forces_generation() {
        // Setup
        let semantic_physics = SemanticPhysicsEngine::new_for_testing().await;

        // Create GPU compute node
        let gpu_node = Node {
            id: NodeId::new(),
            label: "GPU Cluster".to_string(),
            node_type: "GPUComputeNode".to_string(),
            position: Vector3::new(0.0, 0.0, 0.0),
            ..Default::default()
        };

        // Create high compute node
        let mut ai_node = Node {
            id: NodeId::new(),
            label: "Neural Network Training".to_string(),
            node_type: "AIModel".to_string(),
            position: Vector3::new(10.0, 10.0, 10.0),
            ..Default::default()
        };
        ai_node.flags |= 0x20000000; // Set high compute flag

        // Generate forces
        let forces = semantic_physics
            .generate_high_compute_forces(&[gpu_node, ai_node])
            .await
            .unwrap();

        // Assertions
        assert_eq!(forces.len(), 2);

        // AI node should have force toward GPU node
        let ai_force = forces.iter().find(|f| f.node_id == ai_node.id).unwrap();
        assert!(ai_force.vector.magnitude() > 0.0);

        // Force direction should point toward GPU node (negative, since it's at origin)
        assert!(ai_force.vector.x < 0.0);
        assert!(ai_force.vector.y < 0.0);
        assert!(ai_force.vector.z < 0.0);

        // GPU node should have no force (it's the attractor)
        let gpu_force = forces.iter().find(|f| f.node_id == gpu_node.id).unwrap();
        assert_eq!(gpu_force.vector.magnitude(), 0.0);
    }

    #[tokio::test]
    async fn test_inverse_square_decay() {
        let semantic_physics = SemanticPhysicsEngine::new_for_testing().await;

        let gpu_node = Node {
            id: NodeId::new(),
            node_type: "GPUComputeNode".to_string(),
            position: Vector3::new(0.0, 0.0, 0.0),
            ..Default::default()
        };

        // Node at distance 2
        let node_near = create_high_compute_node(Vector3::new(2.0, 0.0, 0.0));

        // Node at distance 4
        let node_far = create_high_compute_node(Vector3::new(4.0, 0.0, 0.0));

        let forces_near = semantic_physics
            .generate_high_compute_forces(&[gpu_node.clone(), node_near])
            .await
            .unwrap();

        let forces_far = semantic_physics
            .generate_high_compute_forces(&[gpu_node, node_far])
            .await
            .unwrap();

        // Force at distance 2 should be ~4x stronger than at distance 4
        // (inverse square: (4/2)² = 4)
        let force_near_mag = forces_near[1].vector.magnitude();
        let force_far_mag = forces_far[1].vector.magnitude();

        let ratio = force_near_mag / force_far_mag;
        assert!((ratio - 4.0).abs() < 0.1, "Ratio should be ~4.0, got {}", ratio);
    }
}
```

### 5.2 Write Frontend Test

Create `client/tests/high-compute.spec.ts`:

```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { mount } from '@vue/test-utils';
import NodePropertiesPanel from '@/components/NodePropertiesPanel.vue';
import { createPinia, setActivePinia } from 'pinia';

describe('High Compute Constraint', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
  });

  it('shows high compute toggle for eligible node types', async () => {
    const wrapper = mount(NodePropertiesPanel, {
      props: {
        nodeId: 'test-node-1',
      },
      global: {
        mocks: {
          node: {
            id: 'test-node-1',
            type: 'AIModel',
            flags: 0,
          },
        },
      },
    });

    // Should show checkbox for AIModel type
    expect(wrapper.find('input[type="checkbox"]').exists()).toBe(true);
  });

  it('hides high compute toggle for ineligible types', async () => {
    const wrapper = mount(NodePropertiesPanel, {
      props: {
        nodeId: 'test-node-2',
      },
      global: {
        mocks: {
          node: {
            id: 'test-node-2',
            type: 'Concept',
            flags: 0,
          },
        },
      },
    });

    // Should NOT show checkbox for Concept type
    expect(wrapper.find('input[type="checkbox"]').exists()).toBe(false);
  });

  it('correctly reads high compute flag from node', () => {
    const nodeWithFlag = {
      id: 'test-node-3',
      type: 'Simulation',
      flags: 0x20000000, // Bit 29 set
    };

    // Extract flag
    const requiresCompute = (nodeWithFlag.flags & 0x20000000) !== 0;
    expect(requiresCompute).toBe(true);
  });

  it('correctly sets high compute flag', () => {
    let flags = 0x00000000;

    // Set bit 29
    flags |= 0x20000000;
    expect((flags & 0x20000000) !== 0).toBe(true);

    // Clear bit 29
    flags &= ~0x20000000;
    expect((flags & 0x20000000) !== 0).toBe(false);
  });
});
```

### 5.3 Run Tests

```bash
# Backend tests
cargo test high_compute

# Expected output:
# running 3 tests
# test high_compute_tests::test_high_compute_constraint_validation ... ok
# test high_compute_tests::test_high_compute_forces_generation ... ok
# test high_compute_tests::test_inverse_square_decay ... ok

# Frontend tests
cd client
npm run test -- high-compute.spec.ts

# Expected output:
# PASS tests/high-compute.spec.ts
#   High Compute Constraint
#     ✓ shows high compute toggle for eligible node types
#     ✓ hides high compute toggle for ineligible types
#     ✓ correctly reads high compute flag from node
#     ✓ correctly sets high compute flag
```

### 5.4 Check Test Coverage

```bash
# Backend coverage
cargo tarpaulin --out Html

# Open target/tarpaulin/index.html
# Ensure coverage is >80%

# Frontend coverage
cd client
npm run test:coverage

# Open coverage/index.html
# Ensure coverage is >80%
```

**🎓 What You've Done**:
- Written comprehensive integration tests
- Validated force calculations with physics assertions
- Tested UI components in isolation
- Ensured >80% test coverage

---

## Step 6: Documentation

**Duration**: 30 minutes
**Files Created**: Documentation updates

### 6.1 Document the Feature

Create `docs/concepts/high-compute-constraint.md`:

```markdown
# High Compute Constraint

## Overview

The `RequiresHighCompute` semantic constraint enables nodes to visually cluster near GPU compute resources in the 3D visualization.

## Use Cases

- **AI/ML Workflows**: Identify compute-intensive training jobs
- **Simulation Pipelines**: Group heavy simulations near processing resources
- **Resource Planning**: Visualize computational dependencies

## Ontological Definition

```toml
[[constraints]]
name = "RequiresHighCompute"
type = "ObjectProperty"
force_type = "strong_attraction"
target_class = "GPUComputeNode"
strength = 800.0
```

## Physics Behavior

- **Force Type**: Strong attraction (800.0 strength)
- **Decay**: Inverse square (F = 800 / r²)
- **Target**: Nodes of type `GPUComputeNode`

## Visual Indicators

- **Color**: Orange (#ff6600)
- **Effect**: Emissive glow + pulsing ring
- **Badge**: "🖥️ High Compute Enabled" in properties panel

## API Usage

### Mark Node as High Compute

```bash
curl -X PATCH http://localhost:3030/api/node/{node_id} \
  -H "Content-Type: application/json" \
  -d '{"flags": 536870912}'  # 0x20000000

# Or via directive
curl -X POST http://localhost:3030/api/directive/set-high-compute \
  -d '{"node_id": "abc123"}'
```

### Query High Compute Nodes

```bash
curl http://localhost:3030/api/nodes?requires_compute=true
```

## Implementation Details

See [Phase 3 Tutorial](../../tutorials/learning-by-doing/phase-3-build-feature.md) for full implementation walkthrough.
```

### 6.2 Update Main Architecture Docs

Edit `docs/concepts/architecture/semantic-physics-system.md`:

Add section:

```markdown
### RequiresHighCompute Constraint

**Added**: 2025-11-05
**Axiom**: `RequiresHighCompute(?x) → strongAttractionTo(?x, GPUComputeNode)`

Nodes marked with this property experience strong attraction (800.0 strength) toward `GPUComputeNode` instances using inverse square decay. This visually clusters compute-intensive operations near computational resources.

**Valid Node Types**: AIModel, Simulation, DataProcessing, Computation

**Visual Indicators**: Orange color, emissive glow, pulsing ring animation
```

### 6.3 Update API Reference

Edit `docs/reference/api/03-graph-operations.md`:

Add endpoint documentation:

```markdown
## Set High Compute Requirement

Mark a node as requiring high computational resources.

**Endpoint**: `POST /api/directive/set-high-compute`

**Request Body**:
```json
{
  "node_id": "string"
}
```

**Response**:
```json
{
  "status": "success",
  "node_id": "abc123",
  "flags": 536870912
}
```

**Errors**:
- `400 Bad Request`: Invalid node type (must be AIModel, Simulation, etc.)
- `404 Not Found`: Node does not exist
```

**🎓 What You've Done**:
- Created comprehensive feature documentation
- Updated architecture diagrams
- Added API reference material
- Enabled future developers to understand your work

---

## Step 7: Manual Testing and Demo

**Duration**: 30 minutes

### 7.1 Create Test Data

```bash
# Start VisionFlow
docker-compose up -d

# Create GPU Compute Node
curl -X POST http://localhost:3030/api/node \
  -H "Content-Type: application/json" \
  -d '{
    "label": "GPU Cluster",
    "node_type": "GPUComputeNode",
    "properties": {
      "gpu_count": 8,
      "gpu_type": "NVIDIA A100"
    }
  }'

# Create several AI model nodes
for model in "GPT-5 Training" "DALL-E 4" "Stable Diffusion XL" "AlphaFold 3"
do
  curl -X POST http://localhost:3030/api/node \
    -H "Content-Type: application/json" \
    -d "{
      \"label\": \"$model\",
      \"node_type\": \"AIModel\",
      \"properties\": {}
    }"
done

# Mark them as requiring high compute
# (Get node IDs from previous responses and set flags)
```

### 7.2 Visual Verification

Open `http://localhost:3030` in your browser:

**Expected Behavior**:
1. **Orange nodes** appear for AI models
2. **Pulsing rings** animate around them
3. **Clustering**: Orange nodes gravitate toward the GPU node
4. **Properties panel**: Checkbox appears for high compute toggle
5. **Badge**: "🖥️ High Compute Enabled" shows when checked

### 7.3 Physics Verification

1. **Select GPU node** → Click and drag it to a new position
2. **Observe**: AI model nodes should follow, pulled by strong attraction
3. **Toggle high compute off** for one node → It should drift away
4. **Toggle back on** → It should return to cluster

### 7.4 Performance Check

Open browser DevTools (F12) → Performance tab:

1. **Record** a few seconds of interaction
2. **Check FPS**: Should maintain 60 FPS with <100 nodes
3. **GPU usage**: Run `nvidia-smi` to verify kernel execution

**Expected**:
- FPS: 55-60 (smooth)
- GPU utilization: 5-15% during simulation
- Memory: <500MB VRAM

---

## 🎉 Congratulations!

You've successfully implemented a **complete end-to-end feature** in VisionFlow!

### What You Accomplished

✅ **Ontology Layer**: Defined new semantic constraint
✅ **Reasoning Pipeline**: Integrated with Whelk-rs
✅ **GPU Compute**: Implemented CUDA kernel (optional)
✅ **Server Logic**: Updated services and adapters
✅ **Client Visualization**: Added visual indicators and UI
✅ **Testing**: Achieved >80% coverage with integration tests
✅ **Documentation**: Comprehensive docs for future developers

---

## 🚀 Next Steps

### Polish Your Implementation

1. **Performance Optimization**: Profile and optimize hot paths
2. **Edge Cases**: Handle scenarios like multiple GPU nodes
3. **User Feedback**: Gather feedback from other developers

### Share Your Work

1. **Create PR**: Submit your feature for review
2. **Demo Video**: Record a showcase of the feature
3. **Blog Post**: Write about your learning experience

### Continue Learning

#### Explore Advanced Topics
- [GPU Optimization Guide](../../concepts/architecture/gpu/optimizations.md)
- [Ontology Reasoning Deep Dive](../../concepts/ontology-reasoning.md)
- [Multi-Agent Integration](../../guides/agent-orchestration.md)

#### Build More Features
- Add other constraint types (e.g., `RequiresLowLatency`)
- Implement custom rendering shaders
- Create new AI agent specializations

#### Contribute to VisionFlow
- [Contributing Guidelines](../../guides/developer/06-contributing.md)
- [Development Workflow](../../guides/development-workflow.md)
- [Code Review Process](../../guides/code-review.md)

---

## 📚 Additional Resources

- [Semantic Physics System](../../concepts/architecture/semantic-physics-system.md)
- [CUDA Programming Guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/)
- [Whelk Reasoner Documentation](https://github.com/balhoff/whelk)
- [Three.js Documentation](https://threejs.org/docs/)

---

## 🐛 Troubleshooting

### Build Errors

**CUDA kernel compilation fails**:
```bash
# Check nvcc installation
nvcc --version

# Ensure CUDA_HOME is set
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
```

**Rust compilation errors**:
```bash
# Clean build
cargo clean
cargo build --release

# Check dependencies
cargo update
```

### Runtime Issues

**Forces not appearing**:
- Verify ontology loaded: Check `/api/ontology/status`
- Check node flags: Bit 29 should be set for high compute nodes
- Look at logs: `docker-compose logs visionflow-container | grep "high_compute"`

**Visual indicators not showing**:
- Clear browser cache
- Rebuild client: `cd client && npm run build`
- Check console for errors (F12)

### Performance Problems

**Low FPS**:
- Reduce node count for testing
- Enable GPU if available
- Lower render quality in settings

**High memory usage**:
- Check for memory leaks in animation loop
- Verify forces array is cleared each frame
- Use browser memory profiler

---

## 📝 Tutorial Metadata

- **Phase**: 3 of 3 (Building Something)
- **Difficulty**: Advanced
- **Time Required**: 4-8 hours
- **Prerequisites**: Phase 2, Rust 1.70+, Node.js 18+
- **Last Updated**: 2025-11-05

---

**Navigation**: [← Phase 2](./phase-2-architecture.md) | [Learning Plan](./README.md)
