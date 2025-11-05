# Phase 1: Foundation and Confidence

**Learn VisionFlow by deploying and using the system**

This tutorial builds confidence through hands-on interaction with VisionFlow's external behavior and key concepts. You'll deploy the system, create your first knowledge graph, interact with AI agents, and explore the 3D visualization.

---

## 🎯 Learning Objectives

By the end of this tutorial, you will:

✅ Successfully deploy VisionFlow using Docker
✅ Create and populate your first knowledge graph
✅ Deploy and observe specialized AI agents
✅ Navigate and interact with the 3D visualization
✅ Understand VisionFlow's core capabilities from a user perspective

**Time Required**: 1-2 hours
**Difficulty**: Beginner
**Prerequisites**: Docker installed, 8GB RAM, modern browser

---

## Step 1: Setup and Deployment

**Duration**: 15 minutes
**Goal**: Get VisionFlow running locally

### 1.1 Clone the Repository

```bash
# Clone VisionFlow
git clone https://github.com/visionflow/visionflow.git
cd visionflow

# Verify you're in the right directory
ls -la
# Expected: You should see Cargo.toml, docker-compose.yml, client/, src/, etc.
```

### 1.2 Configure Environment

```bash
# Copy the example environment file
cp .env.example .env

# Quick verification (optional)
cat .env | head -20
```

**What's in `.env`?**
- `CLAUDE-FLOW-HOST`: Multi-agent container hostname
- `MCP-TCP-PORT`: Model Context Protocol port
- `ENABLE-GPU`: GPU acceleration flag (false by default)
- API keys for external services (optional)

**For this tutorial**: The defaults work fine. Don't modify anything yet.

### 1.3 Start Docker Services

```bash
# Start all services in detached mode
docker-compose up -d

# Watch the startup logs (Ctrl+C to exit)
docker-compose logs -f
```

**Expected Output**:
```
multi-agent-container | MCP Bridge listening on port 3002
visionflow-container  | Server listening on 0.0.0.0:3030
postgres-container    | database system is ready to accept connections
redis-container       | Ready to accept connections
```

**If you see errors**: Check the [Troubleshooting section](#troubleshooting) below.

### 1.4 Verify Installation

```bash
# Check all services are running
docker-compose ps

# Test the health endpoint
curl http://localhost:3030/api/health

# Expected response:
# {"status":"healthy","version":"0.1.0","timestamp":"..."}
```

### 1.5 Access the Web Interface

Open your browser and navigate to:
```
http://localhost:3030
```

**What You Should See**:
- Dark 3D visualization area (currently empty)
- Control panels on left and right
- Green "Connected" status indicator
- Bottom status bar

**🎉 Success!** VisionFlow is now running.

---

## Step 2: Your First Knowledge Graph

**Duration**: 20 minutes
**Goal**: Create and populate a knowledge graph with data

### 2.1 Load Demo Data (Recommended)

The fastest way to see VisionFlow in action:

1. **Click** the "Load Demo Graph" button in the left control panel
2. **Watch** as nodes materialize in 3D space
3. **Observe** the physics simulation organizing the graph

**What You're Seeing**:
- **50-100 nodes** representing concepts and entities
- **Color-coded relationships** based on edge types
- **Physics simulation** with spring forces pulling connected nodes together
- **Repulsion forces** pushing unconnected nodes apart

### 2.2 Understanding the Demo Data

The demo graph typically contains:

| Node Type | Description | Visual Indicator |
|-----------|-------------|------------------|
| **Concepts** | Abstract ideas (AI, Machine Learning) | Blue spheres |
| **Documents** | Files or pages | Green cubes |
| **People** | Authors or contributors | Yellow spheres |
| **Projects** | Software projects | Purple pyramids |

**Try This**:
- **Click a node** to select it (highlights connections)
- **Hover over a node** to see metadata
- **Double-click a node** to focus the camera

### 2.3 Alternative: Connect Logseq Data

If you use Logseq for knowledge management:

1. **Click** "Connect to GitHub" in the control panel
2. **Authorize** VisionFlow to access your repository
3. **Select** your Logseq markdown directory
4. **Wait** for VisionFlow to parse and visualize your notes

**What Happens**:
- VisionFlow parses markdown `[[links]]` and `#tags`
- Creates a node for each page
- Visualizes bi-directional links
- Applies semantic physics based on content

**Example**: If you have notes on "Machine Learning" and "Neural Networks" with links between them, you'll see nodes connected in 3D space.

### 2.4 Alternative: Build Manually

For maximum understanding, create nodes manually:

1. **Click** "New Graph" (creates empty graph)
2. **Click** "Add Node" button
3. **Enter** a label: "Artificial Intelligence"
4. **Repeat** to add more nodes:
   - "Machine Learning"
   - "Deep Learning"
   - "Neural Networks"
   - "Computer Vision"

**Now Connect Them**:
1. **Select** "Artificial Intelligence" node (click it)
2. **Hold Shift** and click "Machine Learning"
3. **Click** "Add Edge" button
4. **Repeat** to create this structure:
   - AI → Machine Learning
   - Machine Learning → Deep Learning
   - Deep Learning → Neural Networks
   - Neural Networks → Computer Vision

**What You're Learning**: The basic data model—nodes (entities/concepts) and edges (relationships).

---

## Step 3: Navigate the 3D Visualization

**Duration**: 15 minutes
**Goal**: Master navigation and understand Semantic Physics

### 3.1 Camera Controls

**Mouse Navigation**:
```
Left Click + Drag    → Rotate camera around the graph
Right Click + Drag   → Pan camera position
Scroll Wheel         → Zoom in and out
Double Click Node    → Focus camera on that node
```

**Keyboard Shortcuts**:
```
Space    → Pause/resume physics simulation
R        → Reset camera to default position
F        → Toggle fullscreen mode
G        → Toggle grid display
H        → Show/hide help overlay
Ctrl+K   → Open command palette
```

**Practice Exercise**:
1. **Rotate** the graph 360 degrees
2. **Zoom in** on a specific node cluster
3. **Pan** to see nodes at the edge
4. **Press Space** to pause physics
5. **Press R** to reset view

### 3.2 Understanding Semantic Physics

VisionFlow uses **physics simulation** to organize your graph based on **ontological rules**.

**The Forces at Play**:

| Force | Description | Visual Effect |
|-------|-------------|---------------|
| **Spring Attraction** | Connected nodes pull together | Clusters form around relationships |
| **Repulsion** | Unconnected nodes push apart | Prevents overcrowding |
| **Gravity** | Gentle downward force | Stabilizes vertical layout |
| **Central Force** | Weak attraction to origin | Keeps graph centered |
| **Ontology Forces** | Semantic rules from ontology | Disjoint concepts separate visually |

**Example**: If your ontology defines `Person` and `Organization` as disjoint classes, nodes of these types will actively repel each other, creating spatial separation.

### 3.3 Interactive Features

**Selection and Highlighting**:
1. **Click any node** → Node highlights, connected edges glow
2. **Look at the info panel** → Shows node properties:
   - Label
   - Type (from ontology)
   - Properties (key-value pairs)
   - Connected nodes

**Edge Filtering**:
1. **Open the "Filters" panel** (right side)
2. **Toggle relationship types** on/off
3. **Watch** as edges disappear/reappear

**Physics Controls**:
1. **Open "Physics" panel** (right side)
2. **Adjust sliders**:
   - **Spring Strength** (0.0-1.0): Stronger = tighter clusters
   - **Repulsion** (0-1000): Higher = more spread out
   - **Damping** (0.0-1.0): Higher = less bouncy
3. **Observe** how the graph reorganizes in real-time

**Practice Exercise**:
1. Set **Spring Strength to 0.1** → Graph spreads out
2. Set **Repulsion to 50** → Nodes separate more
3. Set **Damping to 0.9** → Movement slows down
4. **Press Space** to see the final stable layout

### 3.4 Visual Settings

Make the graph look how you want:

1. **Open "Visual" panel** (right side)
2. **Node Color**: Change base color
3. **Node Size**: Adjust default size (1-20)
4. **Edge Thickness**: Set line width (0.1-5.0)
5. **Glow Effects**: Add intensity (0.0-1.0)
6. **Bloom**: Enable post-processing bloom

**Tip**: For large graphs (1000+ nodes), use smaller node sizes and thinner edges for better performance.

---

## Step 4: Deploy and Observe AI Agents

**Duration**: 20 minutes
**Goal**: Launch a multi-agent system and observe continuous AI analysis

### 4.1 Understanding Multi-Agent Workflows

VisionFlow orchestrates **50+ specialized AI agents** that work continuously to:
- **Research** topics and gather information
- **Analyze** relationships and patterns
- **Generate** insights and recommendations
- **Validate** data quality
- **Optimize** graph structure

**Agent Types**:
| Agent | Role | When to Use |
|-------|------|-------------|
| **Coordinator** | Orchestrates workflow | Every multi-agent task |
| **Researcher** | Gathers information | When you need data |
| **Analyst** | Identifies patterns | For insight discovery |
| **Coder** | Writes code | Automation tasks |
| **Architect** | Designs systems | Complex projects |

### 4.2 Access the Multi-Agent Panel

1. **Look** for "VisionFlow (MCP)" in the left panel
2. **Check** connection status:
   - **Green "Connected"** = MCP bridge is active (port 3002)
   - **Red "Disconnected"** = Check services: `docker-compose logs multi-agent-container`
3. **Click** "Initialize multi-agent" button

### 4.3 Configure Your First Agent Task

**Start Simple**: We'll create a research task to analyze the current graph.

**Fill in the form**:

```
Task Description:
"Analyze the current knowledge graph and identify:
1. The three most central concepts (highest connectivity)
2. Any isolated nodes that should be connected
3. Suggested new relationships based on semantic similarity"

Topology: Mesh
Strategy: Consensus
Priority: Medium
Agent Count: 3
```

**What These Settings Mean**:
- **Mesh Topology**: Agents collaborate freely (good for research)
- **Consensus Strategy**: Agents vote on decisions (higher quality)
- **Medium Priority**: Standard processing speed
- **3 Agents**: Coordinator + Researcher + Analyst

### 4.4 Launch the Multi-Agent System

1. **Review** your configuration
2. **Click** "Launch Multi-Agent System"
3. **Watch** the visualization change:
   - **New green nodes appear**: These are agents (bit 31 flag set)
   - **Blue nodes remain**: Your knowledge graph (bit 30 flag)
   - **Animated connections**: Agent communication

**Agent Status Colors**:
```
Green  → Active and processing
Yellow → Waiting for input
Blue   → Idle/ready
Red    → Error or blocked
Grey   → Completed/terminated
```

### 4.5 Monitor Agent Execution

**Real-Time Monitoring**:
1. **Click any green agent node** to see:
   - Current task
   - Progress percentage
   - Execution logs
   - Communication with other agents

2. **Open "Agent Status" panel** (left side) for overview:
   - All active agents
   - Resource usage
   - Estimated completion time

**What to Look For**:
- **Coordinator node** at the center, orchestrating
- **Specialized agents** positioned around it
- **Message lines** showing communication
- **Progress updates** in the status panel

**Typical Timeline**:
- **0-30 seconds**: Agents initialize and plan
- **30 seconds - 2 minutes**: Active research and analysis
- **2-5 minutes**: Generating and validating results
- **5+ minutes**: Finalizing and formatting output

### 4.6 View Agent Results

When agents complete (nodes turn grey):

1. **Success notification** appears
2. **Results panel** opens automatically
3. **Review the output**:
   - **Analysis Summary**: Key findings
   - **Central Concepts**: Most connected nodes
   - **Isolated Nodes**: Suggestions for connections
   - **New Relationships**: Proposed edges

**Example Output**:
```
Central Concepts:
1. "Machine Learning" (8 connections)
2. "Artificial Intelligence" (6 connections)
3. "Neural Networks" (5 connections)

Isolated Nodes:
- "Quantum Computing" (consider linking to "Machine Learning")

Suggested Relationships:
- "Deep Learning" → "Computer Vision" (semantic similarity: 0.87)
```

### 4.7 Apply Agent Recommendations

VisionFlow doesn't automatically modify your graph. You decide:

1. **Review** each suggestion
2. **Accept** recommendations by clicking "Apply"
3. **Reject** if they don't make sense
4. **Watch** as new edges appear in the visualization

**Human-in-the-Loop**: This is a key design principle. Agents suggest, humans decide.

---

## Step 5: Explore Advanced Features (Optional)

**Duration**: 15 minutes
**Goal**: Discover additional capabilities

### 5.1 Voice Interaction (If Enabled)

If you set `ENABLE-VOICE=true` in `.env`:

1. **Click** the microphone icon (top right)
2. **Grant** browser microphone permissions
3. **Speak** a command:
   - "Show me all nodes about machine learning"
   - "Create a new node called quantum computing"
   - "Connect artificial intelligence to machine learning"
4. **Listen** to agent responses via speakers

**Voice Services**:
- **Whisper** (port 8000): Speech-to-text
- **Kokoro** (port 8880): Text-to-speech with multiple voices

### 5.2 XR/VR Mode (Meta Quest 3)

If you have a Meta Quest 3:

1. **Enable** XR in `.env`: `ENABLE-XR=true`
2. **Restart** services: `docker-compose restart`
3. **Open** VisionFlow in Quest browser
4. **Click** the VR icon in the interface
5. **Use** hand tracking to interact with nodes

**XR Experience**:
- **Walk through** your knowledge graph
- **Point and pinch** to select nodes
- **Spatial audio** for agent communication
- **Collaborative** viewing with multiple users

### 5.3 Command Palette

Press **Ctrl+K** to open the command palette:

```
Type to search commands:
- "agent status" → Show global agent overview
- "export graph" → Download graph as JSON/GraphML
- "settings" → Open preferences
- "help" → Show documentation
- "theme" → Change color scheme
```

**Power User Tip**: The command palette is the fastest way to access features.

---

## 🎓 What You've Learned

Congratulations! You now understand:

✅ **Deployment**: How to install and run VisionFlow with Docker
✅ **Knowledge Graphs**: The node/edge data model and how to create graphs
✅ **3D Visualization**: Navigation controls and Semantic Physics
✅ **AI Agents**: How to deploy multi-agent systems and interpret results
✅ **Core Capabilities**: The system's user-facing features

---

## 🧪 Practice Challenges

Test your understanding:

### Challenge 1: Custom Graph
**Task**: Create a knowledge graph about your favorite topic (movies, sports, technology)
- Add at least 10 nodes
- Create meaningful relationships
- Adjust physics for optimal layout

### Challenge 2: Agent Workflow
**Task**: Deploy agents to analyze your custom graph
- Use a hierarchical topology
- Request specific insights
- Apply at least one recommendation

### Challenge 3: Visualization Mastery
**Task**: Make your graph visually distinctive
- Customize colors for different node types
- Adjust edge filtering
- Find the perfect physics settings
- Export a screenshot

---

## 🚀 Next Steps

You've mastered the foundation! Choose your path:

### Continue the Learning Journey
**[→ Phase 2: Core Architecture Deep Dive](./phase-2-architecture.md)**
Learn how VisionFlow works internally by exploring the codebase.

### Dive Deeper into Specific Topics
- [Agent Orchestration Guide](../../guides/orchestrating-agents.md) - Advanced agent patterns
- [Configuration Reference](../../reference/configuration.md) - All .env options
- [XR Setup Guide](../../guides/xr-setup.md) - Immersive VR configuration

### Start Building
**[→ Phase 3: Build a Feature](./phase-3-build-feature.md)**
Skip ahead if you're ready to modify VisionFlow's code.

---

## 🐛 Troubleshooting

### Docker Services Won't Start

**Problem**: `docker-compose up` fails

**Solutions**:
```bash
# Check Docker is running
docker ps

# Check for port conflicts
sudo lsof -i :3030
sudo lsof -i :3002

# Clean up old containers
docker-compose down -v
docker-compose up -d

# View detailed logs
docker-compose logs visionflow-container
```

### Web Interface Shows "Disconnected"

**Problem**: Red status indicator in UI

**Solutions**:
```bash
# Check backend health
curl http://localhost:3030/api/health

# Restart services
docker-compose restart visionflow-container

# Check firewall isn't blocking ports 3030, 3002
```

### Graph Won't Load

**Problem**: 3D area stays empty

**Solutions**:
1. **Check browser console** (F12) for errors
2. **Verify WebGL support**: Visit https://get.webgl.org/
3. **Try a different browser** (Chrome recommended)
4. **Disable browser extensions** that might block WebGL

### Agents Won't Initialize

**Problem**: "Multi-agent initialization failed"

**Solutions**:
```bash
# Check MCP bridge is running
docker-compose logs multi-agent-container | grep "MCP"

# Verify connection
curl http://localhost:3002/health

# Restart multi-agent container
docker-compose restart multi-agent-container
```

### Performance Issues

**Problem**: Slow/choppy 3D rendering

**Solutions**:
1. **Lower render quality** in Visual settings
2. **Reduce node count** with filtering
3. **Enable GPU** if you have NVIDIA card: `ENABLE-GPU=true`
4. **Close** other GPU-intensive applications
5. **Check FPS**: Press Ctrl+Shift+D for debug overlay

---

## 📚 Additional Resources

- [Installation Guide](../../getting-started/01-installation.md) - Detailed installation instructions
- [First Graph and Agents](../../getting-started/02-first-graph-and-agents.md) - Alternative beginner guide
- [Documentation Index](../../readme.md) - All documentation
- [GitHub Issues](https://github.com/visionflow/visionflow/issues) - Report problems

---

## 📝 Tutorial Metadata

- **Phase**: 1 of 3 (Foundation and Confidence)
- **Difficulty**: Beginner
- **Time Required**: 1-2 hours
- **Prerequisites**: Docker, modern browser
- **Last Updated**: 2025-11-05

---

**Navigation**: [← Learning Plan](./README.md) | [Phase 2 →](./phase-2-architecture.md)
