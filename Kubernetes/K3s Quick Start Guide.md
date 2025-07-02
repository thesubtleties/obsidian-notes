# K3s Quick Start Guide

## What You'll Learn
By the end of this guide, you'll have K3s running and deploy your first application. This takes about 15 minutes and gives you hands-on Kubernetes experience without the complexity.

## Prerequisites
- **Linux machine** (Ubuntu, Debian, or similar) - [[Linux/Linux Overview]]
- **Root/sudo access** for installation
- **Basic command line comfort** - [[Linux/Quick Reference/Linux Commands Quick Reference]]
- **Understanding of containers** - [[Docker/Docker Overview]]

**💡 Mentor Tip**: K3s works great on a VPS, Raspberry Pi, or local Linux VM. If you're on Windows/Mac, consider using a Linux VM or WSL2.

## Installation (The Easy Way)

### Single Command Install
```bash
curl -sfL https://get.k3s.io | sh -
```

That's it! K3s is now running. The install script:
- Downloads and installs K3s
- Creates a systemd service
- Starts the K3s server
- Sets up kubectl configuration

### Verify Installation
```bash
# Check if K3s is running
sudo systemctl status k3s

# Check nodes (should show your machine)
sudo kubectl get nodes

# Check system pods
sudo kubectl get pods --all-namespaces
```

**🔍 What's Happening**: K3s creates a single-node cluster with your machine as both master and worker. The `--all-namespaces` flag shows system pods that K3s runs internally.

## Your First Application: Hello World Web Server

### Create a Simple Pod
Create a file called `hello-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-world
  labels:
    app: hello
spec:
  containers:
  - name: hello-container
    image: nginx:alpine
    ports:
    - containerPort: 80
```

### Deploy the Pod
```bash
# Apply the configuration
sudo kubectl apply -f hello-pod.yaml

# Check if it's running
sudo kubectl get pods

# Get detailed information
sudo kubectl describe pod hello-world
```

### Access Your Application
```bash
# Forward port from pod to your machine
sudo kubectl port-forward hello-world 8080:80

# In another terminal, test it
curl http://localhost:8080
```

**🎉 Success!** You've deployed your first application to Kubernetes.

## Understanding What Just Happened

### The YAML File Breakdown
```yaml
apiVersion: v1          # Kubernetes API version
kind: Pod               # Type of resource (Pod is basic unit)
metadata:               # Information about the resource
  name: hello-world     # Name of this pod
  labels:               # Tags for organization
    app: hello
spec:                   # Desired state specification
  containers:           # List of containers in this pod
  - name: hello-container
    image: nginx:alpine # Docker image to run
    ports:              # Ports the container exposes
    - containerPort: 80
```

**💡 Mentor Tip**: This YAML pattern is fundamental to Kubernetes. Every resource follows this structure: `apiVersion`, `kind`, `metadata`, `spec`.

### Key Concepts Introduced
- **Pod**: Smallest deployable unit in Kubernetes (like a wrapper around containers)
- **kubectl**: Command-line tool for managing Kubernetes
- **YAML manifests**: Declarative way to define what you want
- **Port forwarding**: Way to access applications during development

## Common Commands You'll Use

### Pod Management
```bash
# List pods
sudo kubectl get pods

# Get detailed pod info
sudo kubectl describe pod <pod-name>

# See pod logs
sudo kubectl logs <pod-name>

# Delete a pod
sudo kubectl delete pod <pod-name>

# Get pod status with more details
sudo kubectl get pods -o wide
```

### Cluster Information
```bash
# Show cluster info
sudo kubectl cluster-info

# Show all resources
sudo kubectl get all

# Show nodes
sudo kubectl get nodes -o wide
```

## Cleaning Up
```bash
# Delete your test pod
sudo kubectl delete -f hello-pod.yaml

# Or delete by name
sudo kubectl delete pod hello-world
```

## What's Next?

### Immediate Next Steps
1. **[[Kubernetes/Core Concepts/01 - Pods and Containers]]** - Understand pods deeply
2. **[[Kubernetes/Core Concepts/02 - Services and Networking]]** - Learn how pods communicate
3. **[[Kubernetes/Kubernetes Command Reference]]** - Master kubectl commands

### Real-World Application
4. **[[Kubernetes/Practical Guides/From Docker Compose to K3s]]** - Migrate existing projects
5. **[[Kubernetes/K3s Topics/K3s for Development]]** - Development workflows

## Common Issues and Solutions

### Permission Denied
**Problem**: `kubectl` commands fail with permission errors
**Solution**: Use `sudo` with kubectl, or copy kubeconfig:
```bash
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami) ~/.kube/config
```

### Port Already in Use
**Problem**: Port forwarding fails
**Solution**: Use a different port or kill existing processes:
```bash
sudo kubectl port-forward hello-world 8081:80  # Different port
# OR
sudo lsof -i :8080  # Find what's using port 8080
```

### Pod Stuck in Pending
**Problem**: Pod shows "Pending" status
**Solution**: Check resources and events:
```bash
sudo kubectl describe pod <pod-name>
sudo kubectl get events --sort-by=.metadata.creationTimestamp
```

## Real-World Context

### When to Use K3s
- **Development environments** - Fast, lightweight, easy reset
- **Edge computing** - IoT devices, remote locations
- **Learning Kubernetes** - Full K8s concepts, simpler setup
- **Small production workloads** - Personal projects, startups

### Production Considerations
K3s is production-ready but consider:
- **High availability**: Single point of failure with one node
- **Resource limits**: Memory and CPU constraints on single machine
- **Backup strategy**: How to backup your cluster state

**💡 Mentor Tip**: Many companies use K3s for edge deployments and development environments, then migrate to full Kubernetes for large-scale production.

## Troubleshooting Resources
- **[[Kubernetes/Practical Guides/Troubleshooting Guide]]** - Common issues and solutions
- **[[Linux/Quick Reference/Linux Commands Quick Reference]]** - System debugging
- **[[Docker/Docker Overview]]** - Container troubleshooting

## Related Topics
- **[[Kubernetes/Kubernetes Overview]]** - Big picture understanding
- **[[Docker/Docker Overview]]** - Container fundamentals
- **[[VPS-Linux/VPS Setup and Management]]** - Server management context