# VisionFlow Learning-by-Doing Plan

**Learn VisionFlow through hands-on tutorials following the Diátaxis framework**

This learning plan follows a **"learning by doing"** approach, moving from basic deployment to modifying core components. You'll build confidence by interacting with all major layers of the system.

---

## 🎯 Learning Approach

VisionFlow is an **Enterprise-Grade Multi-User Multi-Agent Knowledge Graphing** engine—a comprehensive codebase spanning:
- **Rust** (server/GPU)
- **Vue.js/Three.js** (client)
- **Docker-orchestrated AI agents**

This tutorial series takes you from **zero to contributor** in three progressive phases.

---

## 📚 Tutorial Structure

### Phase 1: Foundation and Confidence
**Duration**: 1-2 hours | **Difficulty**: Beginner

Learn the system's external behavior and key concepts through hands-on deployment.

- [**Phase 1: Foundation Tutorial**](./phase-1-foundation.md)
  - Setup: Clone and deploy with Docker
  - First Graph: Create and populate your first knowledge graph
  - Agent Interaction: Deploy specialized AI agents
  - Visualization: Explore the 3D interface

**What You'll Build**: A working VisionFlow installation with a populated knowledge graph and active AI agents.

---

### Phase 2: Core Architecture Deep Dive
**Duration**: 3-4 hours | **Difficulty**: Intermediate

Dive into the source directories and understand how the application works internally.

- [**Phase 2: Architecture Tutorial**](./phase-2-architecture.md)
  - Backend Logic: Trace CQRS operations through the Hexagonal Architecture
  - Ontology System: Run validation and reasoning examples
  - GPU Compute: Identify CUDA kernel integration
  - Client Sync: Understand the Binary WebSocket Protocol

**What You'll Learn**: How to navigate the codebase, understand data flow, and identify key components for modification.

---

### Phase 3: Building Something
**Duration**: 4-8 hours | **Difficulty**: Advanced

Build confidence through a capstone project: implementing a custom semantic physics constraint.

- [**Phase 3: Feature Implementation Tutorial**](./phase-3-build-feature.md)
  - Define the Constraint: Add a new ontological relationship
  - Update the Reasoning Pipeline: Modify Whelk-rs integration
  - Implement GPU Kernel: Accelerate constraint calculation (optional)
  - Client Visualization: Update the 3D renderer
  - Testing and Documentation: Maintain >80% test coverage

**What You'll Build**: A ninth semantic physics constraint that demonstrates end-to-end mastery of the system.

---

## 🧠 Conceptual Framework

### The Warehouse Analogy

Think of VisionFlow as an **ultra-high-speed freight distribution centre for knowledge**:

| Component | Warehouse Analogy | VisionFlow Reality |
|-----------|-------------------|-------------------|
| **Hexagonal Architecture** | Warehouse layout protecting core operations | Business logic isolated from transport mechanisms |
| **Ontology System** | Quality control + intelligence hub | Validates data and infers new knowledge automatically |
| **GPU Compute Layer** | Robotic sorting system | 39 CUDA kernels for 100x speedup |
| **3D Client** | Real-time control tower | Multi-user visualization with voice commands |

**Key Insight**: The architecture ensures that changing how goods arrive (REST vs WebSocket) doesn't affect how they're processed internally.

---

## 🛠️ Prerequisites

Before starting, ensure you have:

### Required
- **Docker** 20.10+ with Docker Compose
- **8GB RAM** minimum (16GB recommended)
- **Modern browser** with WebGL 2.0 support
- **Git** for version control
- **Basic programming knowledge** (any language)

### For Phase 3 (Optional)
- **Rust** 1.70+ toolchain
- **Node.js** 18+ with npm
- **NVIDIA GPU** (RTX 4080+ recommended for GPU work)
- **CUDA** 11.0+ toolkit

### Installation

If you haven't installed VisionFlow yet, start here:
- [**Installation Guide**](../../getting-started/01-installation.md)

---

## 📖 How to Use This Tutorial Series

### For Beginners
1. Complete **Phase 1** to understand what VisionFlow does
2. Experiment with the demo data and UI
3. Skip directly to Phase 3 if you want to build something immediately (with frequent reference to Phase 2)

### For Experienced Developers
1. Skim **Phase 1** to verify your installation
2. Focus on **Phase 2** to map mental models to the codebase
3. Jump to **Phase 3** to contribute a feature

### For Contributors
1. Use **Phase 2** as a codebase reference
2. Complete **Phase 3** to understand the full development workflow
3. Reference individual sections as needed

---

## 🎓 Learning Outcomes

By completing all three phases, you will be able to:

✅ **Deploy and operate** VisionFlow in production
✅ **Navigate the codebase** with confidence across Rust, Vue.js, and Docker layers
✅ **Understand the architecture** including Hexagonal/CQRS, ontology reasoning, and GPU compute
✅ **Modify core components** spanning backend, reasoning pipeline, GPU kernels, and frontend
✅ **Test and document** changes following project standards
✅ **Contribute features** through the established development workflow

---

## 🗺️ Navigation

### Start Your Learning Journey

Choose your starting point based on your experience:

| Experience Level | Start Here | Time Commitment |
|-----------------|------------|-----------------|
| **New to VisionFlow** | [Phase 1: Foundation](./phase-1-foundation.md) | 1-2 hours |
| **Exploring the codebase** | [Phase 2: Architecture](./phase-2-architecture.md) | 3-4 hours |
| **Ready to build** | [Phase 3: Build Feature](./phase-3-build-feature.md) | 4-8 hours |

### Related Documentation

- [Getting Started Guides](../../getting-started/) - Installation and first steps
- [Developer Guides](../../guides/developer/) - Development workflow and patterns
- [Architecture Concepts](../../concepts/architecture/) - Deep technical documentation
- [API Reference](../../reference/api/) - Complete API documentation

---

## 💡 Tips for Success

### Learning Strategies

1. **Hands-On First**: Type the commands yourself rather than copy-pasting
2. **Experiment Freely**: Docker makes it easy to reset if something breaks
3. **Read the Errors**: VisionFlow has comprehensive error messages
4. **Ask Questions**: Use GitHub Discussions for community support

### Common Pitfalls

❌ **Skipping Phase 1**: Don't jump to code without understanding the user experience
❌ **Reading Instead of Doing**: Theory without practice doesn't build confidence
❌ **Rushing**: Each phase builds on the previous one
❌ **Ignoring Tests**: Tests are documentation that always stays current

---

## 🆘 Getting Help

Stuck? Here's where to look:

1. **Error Messages**: Read them carefully—they often include solutions
2. **Logs**: `docker-compose logs -f` shows real-time system output
3. **Documentation**: Search the [docs index](../../readme.md)
4. **GitHub Issues**: Check existing issues or create a new one
5. **Community**: Join discussions for peer support

---

## 🚀 Ready to Begin?

Start with Phase 1 to build your foundation:

**[→ Begin Phase 1: Foundation and Confidence](./phase-1-foundation.md)**

---

## 📝 Tutorial Metadata

- **Framework**: [Diátaxis](https://diataxis.fr/) - Tutorial Category
- **Last Updated**: 2025-11-05
- **Tested With**: VisionFlow 0.1.0
- **Authors**: VisionFlow Core Team
- **License**: Mozilla Public License 2.0

---

**Navigation**: [📖 Documentation Index](../../readme.md) | [🏠 Project Home](../../../README.md)
