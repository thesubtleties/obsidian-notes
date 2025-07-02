# Kubernetes Command Reference

## Quick Reference for kubectl & K3s

**💡 Mentor Tip**: Most commands work identically between K3s and full Kubernetes. K3s-specific differences are noted with 🚀.

## Essential Commands

### Cluster Information
```bash
# Check cluster status
kubectl cluster-info

# View cluster nodes
kubectl get nodes
kubectl get nodes -o wide  # More details

# Show cluster configuration
kubectl config view

# 🚀 K3s: Check K3s service status
sudo systemctl status k3s
```

### Working with Pods

#### Basic Pod Operations
```bash
# List all pods
kubectl get pods
kubectl get pods -o wide           # Show node placement and IPs
kubectl get pods --all-namespaces  # Pods in all namespaces

# Get detailed pod information
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -f         # Follow logs (like tail -f)
kubectl logs <pod-name> --previous # Previous container instance logs

# Execute commands in running pod
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -- sh  # If bash not available

# Delete pods
kubectl delete pod <pod-name>
kubectl delete pod <pod-name> --grace-period=0 --force  # Force delete
```

#### Advanced Pod Operations
```bash
# Copy files to/from pods
kubectl cp <local-file> <pod-name>:<pod-path>
kubectl cp <pod-name>:<pod-path> <local-file>

# Port forwarding (access pod from local machine)
kubectl port-forward <pod-name> <local-port>:<pod-port>
kubectl port-forward <pod-name> 8080:80

# Watch pod status changes
kubectl get pods -w
```

### Working with Deployments

#### Basic Deployment Operations
```bash
# List deployments
kubectl get deployments
kubectl get deploy  # Short form

# Create deployment
kubectl create deployment <name> --image=<image>
kubectl create deployment nginx-app --image=nginx:alpine

# Scale deployments
kubectl scale deployment <name> --replicas=3

# Update deployment image
kubectl set image deployment/<name> <container>=<new-image>
kubectl set image deployment/nginx-app nginx=nginx:latest

# Check rollout status
kubectl rollout status deployment/<name>

# Rollback deployment
kubectl rollout undo deployment/<name>
kubectl rollout history deployment/<name>  # View rollout history
```

### Working with Services

#### Service Operations
```bash
# List services
kubectl get services
kubectl get svc  # Short form

# Expose deployment as service
kubectl expose deployment <name> --port=80 --type=NodePort
kubectl expose deployment <name> --port=80 --type=LoadBalancer

# Get service details
kubectl describe service <service-name>

# Delete services
kubectl delete service <service-name>
```

### Resource Management

#### Apply and Manage YAML Files
```bash
# Apply configuration from file
kubectl apply -f <file.yaml>
kubectl apply -f <directory>/     # Apply all YAML files in directory

# Delete resources from file
kubectl delete -f <file.yaml>

# Validate YAML without applying
kubectl apply -f <file.yaml> --dry-run=client -o yaml

# Compare current state with file
kubectl diff -f <file.yaml>
```

#### Get Resource Information
```bash
# List all resource types
kubectl api-resources

# Get all resources in current namespace
kubectl get all

# Get resources across all namespaces
kubectl get all --all-namespaces

# Get specific resource types
kubectl get pods,services,deployments
```

### Namespace Management

```bash
# List namespaces
kubectl get namespaces
kubectl get ns  # Short form

# Create namespace
kubectl create namespace <name>

# Set default namespace for current context
kubectl config set-context --current --namespace=<namespace>

# Run command in specific namespace
kubectl get pods -n <namespace>

# Delete namespace (and all resources in it)
kubectl delete namespace <namespace>
```

### Debugging and Troubleshooting

#### Events and Logs
```bash
# Get cluster events (most recent first)
kubectl get events --sort-by=.metadata.creationTimestamp

# Get events for specific resource
kubectl get events --field-selector involvedObject.name=<pod-name>

# Describe resource for troubleshooting
kubectl describe <resource-type> <resource-name>
kubectl describe pod <pod-name>
kubectl describe node <node-name>
```

#### Resource Usage
```bash
# Check node resource usage
kubectl top nodes

# Check pod resource usage
kubectl top pods
kubectl top pods --all-namespaces

# 🚀 K3s: Check system resources
df -h  # Disk usage
free -h  # Memory usage
```

### ConfigMaps and Secrets

#### ConfigMaps
```bash
# Create configmap from literal values
kubectl create configmap <name> --from-literal=key1=value1

# Create configmap from file
kubectl create configmap <name> --from-file=<file-path>

# List configmaps
kubectl get configmaps
kubectl get cm  # Short form

# View configmap contents
kubectl describe configmap <name>
kubectl get configmap <name> -o yaml
```

#### Secrets
```bash
# Create secret
kubectl create secret generic <name> --from-literal=key1=value1

# Create secret from file
kubectl create secret generic <name> --from-file=<file-path>

# List secrets
kubectl get secrets

# View secret (base64 encoded)
kubectl get secret <name> -o yaml

# Decode secret
kubectl get secret <name> -o jsonpath='{.data.key1}' | base64 -d
```

## K3s Specific Commands

### K3s Service Management
```bash
# 🚀 Start/stop/restart K3s
sudo systemctl start k3s
sudo systemctl stop k3s
sudo systemctl restart k3s

# 🚀 Check K3s logs
sudo journalctl -u k3s -f

# 🚀 K3s configuration location
sudo cat /etc/rancher/k3s/k3s.yaml
```

### K3s Installation Management
```bash
# 🚀 Uninstall K3s
sudo /usr/local/bin/k3s-uninstall.sh

# 🚀 Install K3s with options
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -

# 🚀 Get K3s token (for adding nodes)
sudo cat /var/lib/rancher/k3s/server/node-token
```

### K3s Kubeconfig Setup
```bash
# 🚀 Copy kubeconfig for regular user
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami) ~/.kube/config

# 🚀 Export kubeconfig for current session
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Useful Command Patterns

### Quick Debugging Workflow
```bash
# 1. Check overall cluster health
kubectl get nodes
kubectl get pods --all-namespaces

# 2. Look at specific problem
kubectl describe pod <failing-pod>
kubectl logs <failing-pod>

# 3. Check events for context
kubectl get events --sort-by=.metadata.creationTimestamp

# 4. Check resource usage if needed
kubectl top nodes
kubectl top pods
```

### Development Workflow
```bash
# 1. Apply changes
kubectl apply -f app.yaml

# 2. Check deployment
kubectl get pods -w

# 3. Debug if needed
kubectl logs <pod-name> -f

# 4. Test connectivity
kubectl port-forward <pod-name> 8080:80
```

## Command Aliases (Time Savers)

Add these to your shell profile (`~/.bashrc` or `~/.zshrc`):

```bash
# Basic aliases
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kdp='kubectl describe pod'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'

# K3s specific
alias k3s-status='sudo systemctl status k3s'
alias k3s-logs='sudo journalctl -u k3s -f'
```

## Common Patterns and Examples

### Quick Pod for Testing
```bash
# Run temporary pod for testing
kubectl run test-pod --image=busybox --rm -it --restart=Never -- sh

# Test network connectivity from inside cluster
kubectl run netshoot --image=nicolaka/netshoot --rm -it --restart=Never
```

### Resource Monitoring
```bash
# Watch pods in real-time
watch 'kubectl get pods'

# Monitor events continuously
kubectl get events -w

# Check resource usage continuously
watch 'kubectl top nodes && echo && kubectl top pods'
```

## Legacy and Version Notes

### Kubernetes API Versions
- **apps/v1** - Current stable API for Deployments, ReplicaSets
- **v1** - Core API for Pods, Services, ConfigMaps
- **extensions/v1beta1** - ⚠️ Deprecated, avoid in new deployments

### kubectl Version Compatibility
```bash
# Check kubectl version
kubectl version --client

# Check server version
kubectl version

# 💡 Tip: kubectl should be within 1 minor version of cluster
```

### K3s Version Notes
- K3s tracks Kubernetes releases (usually 1-2 versions behind)
- Check compatibility: `k3s --version`
- Major changes typically noted in K3s GitHub releases

## Related Topics
- **[[Kubernetes/K3s Quick Start Guide]]** - Getting started
- **[[Kubernetes/Core Concepts/01 - Pods and Containers]]** - Understanding resources
- **[[Kubernetes/Practical Guides/Troubleshooting Guide]]** - Debugging help
- **[[Linux/Quick Reference/Linux Commands Quick Reference]]** - System commands