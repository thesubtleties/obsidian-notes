# Kubernetes Overview

## What is Kubernetes?

Kubernetes (K8s) is a container orchestration platform that automates deployment, scaling, and management of containerized applications. Think of it as a sophisticated manager for your [[Docker/Docker Overview|Docker containers]] that handles the complexities of running applications across multiple servers.

## The Kubernetes Ecosystem

### Core Distributions
- **Full Kubernetes** - The complete, enterprise-grade orchestration platform
- **K3s** - Lightweight Kubernetes designed for edge computing, IoT, and development
- **Minikube** - Single-node Kubernetes for local development
- **Kind** - Kubernetes in Docker for testing

## K3s vs Full Kubernetes: Which Should You Learn?

### Start with K3s If:
- You're new to container orchestration
- Working on personal projects or small teams
- Need something that "just works" quickly
- Running on resource-constrained environments
- Want to understand Kubernetes concepts without complexity overhead

### Move to Full Kubernetes When:
- Managing enterprise applications at scale
- Need advanced networking features
- Require fine-grained security controls
- Working with multiple teams and complex deployments
- Need specific cloud provider integrations

**💡 Mentor Tip**: K3s uses the same APIs and concepts as full Kubernetes. Learning K3s first gives you 90% of Kubernetes knowledge with 10% of the complexity. You can always migrate later.

## Why Kubernetes Matters

### Problems It Solves
- **Manual Deployment Headaches** - No more SSH'ing into servers to deploy
- **Scaling Challenges** - Automatically handle traffic spikes
- **Service Discovery** - Applications find each other automatically  
- **Self-Healing** - Restarts failed containers and replaces unhealthy nodes
- **Resource Management** - Efficiently pack applications onto available hardware

### Real-World Context
Moving from [[Docker/Docker Overview|Docker Compose]] to Kubernetes is like upgrading from a personal car to a fleet management system. More complex, but handles enterprise needs.

## Learning Path Recommendations

### Beginner Path (Start Here)
1. **[[Kubernetes/K3s Quick Start Guide]]** - Get hands-on experience
2. **[[Kubernetes/Core Concepts/01 - Pods and Containers]]** - Understanding the basics
3. **[[Kubernetes/Core Concepts/02 - Services and Networking]]** - How applications communicate
4. **[[Kubernetes/Practical Guides/From Docker Compose to K3s]]** - Bridge existing knowledge

### Intermediate Path
1. Complete Core Concepts (01-05)
2. **[[Kubernetes/K3s Topics/K3s for Development]]** - Development workflows
3. **[[Kubernetes/K3s Topics/K3s Production Considerations]]** - Running in production

### Advanced Path
1. **[[Kubernetes/K3s Topics/Migrating K3s to Full K8s]]** - When to make the jump
2. **[[Kubernetes/Full Kubernetes Topics/Advanced Networking]]** - Complex networking
3. **[[Kubernetes/Full Kubernetes Topics/RBAC and Security]]** - Enterprise security

## Prerequisites

### Essential Knowledge
- **[[Docker/Docker Overview]]** - Container fundamentals
- **[[Linux/Linux Overview]]** - Command line comfort
- **[[HTTP/HTTP Overview]]** - Understanding web services
- **[[Git-Github/Git Commands Overview]]** - Version control for configurations

### Nice to Have
- **[[SQL/SQL Docs & Resources]]** - For database deployments
- **[[VSCode/VSCode Overview - Tips and Tricks]]** - YAML editing
- **[[VPS-Linux/VPS Setup and Management]]** - Server management concepts

## Common Misconceptions

**"Kubernetes is just Docker with extra steps"**  
→ Kubernetes solves production problems Docker can't: service discovery, load balancing, secrets management, automatic scaling

**"K3s is a toy version of Kubernetes"**  
→ K3s runs production workloads at scale. It's architecturally simplified, not feature-limited

**"You need a cluster to learn Kubernetes"**  
→ K3s runs perfectly on a single machine for learning and development

## Version and Legacy Considerations

### Current Landscape (2024)
- **Kubernetes**: Stable API (v1), quarterly releases
- **K3s**: Tracks Kubernetes releases closely, usually 1-2 versions behind
- **kubectl**: Command-line tool works with both distributions

### Legacy Notes
- **Pre-1.0 Kubernetes** (before 2015) had different APIs - avoid old tutorials
- **Docker Swarm** was Docker's orchestration attempt - largely deprecated
- **Kubernetes 1.20+** deprecated Docker as container runtime (doesn't affect users)

## Next Steps

Ready to get started? **[[Kubernetes/K3s Quick Start Guide]]** will have you running containers in Kubernetes within 15 minutes.

Need quick command reference? Jump to **[[Kubernetes/Kubernetes Command Reference]]**.

## Related Topics
- **[[Docker/Docker Overview]]** - Container fundamentals
- **[[Deployment Guides/VPS Deployment]]** - Deploying applications
- **[[Web Development Learning Path]]** - Full-stack development context