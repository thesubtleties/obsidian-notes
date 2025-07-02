# 01 - Pods and Containers

## What is a Pod?

A **Pod** is the smallest deployable unit in Kubernetes. Think of it as a wrapper around one or more containers that share:
- Network (same IP address)
- Storage volumes
- Lifecycle (start/stop together)

**💡 Mentor Tip**: Most of the time, you'll have one container per pod. Multi-container pods are for specialized cases like sidecar patterns.

## Pod vs Container: The Key Difference

### Docker Container (What You Know)
```bash
docker run -d --name web-server nginx:alpine
docker run -d --name database postgres:13
```
Each container is isolated and independent.

### Kubernetes Pod (New Concept)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-server-pod
spec:
  containers:
  - name: nginx
    image: nginx:alpine
  - name: logging-sidecar
    image: fluent/fluent-bit
```
Containers in the same pod share network and can communicate via `localhost`.

## Why Pods Exist

### Problem Pods Solve
In [[Docker/Docker Overview|Docker]], if you need containers to work closely together, you have to:
- Manually create networks
- Share volumes explicitly
- Coordinate startup order
- Handle service discovery

### Pod Benefits
- **Shared networking**: Containers communicate via `localhost`
- **Shared storage**: Volumes mounted in multiple containers
- **Atomic deployment**: All containers start/stop together
- **Resource sharing**: CPU/memory limits apply to the whole pod

## Pod Lifecycle

### Pod States
```bash
# Check pod status
kubectl get pods

# Possible states:
# Pending    - Pod accepted but not yet scheduled
# Running    - Pod bound to node, at least one container running
# Succeeded  - All containers terminated successfully
# Failed     - All containers terminated, at least one failed
# Unknown    - Pod state cannot be determined
```

### Pod Lifecycle Events
1. **Created** - Pod definition submitted to Kubernetes
2. **Scheduled** - Kubernetes assigns pod to a node
3. **Image Pull** - Node downloads container images
4. **Container Start** - Containers start in the pod
5. **Running** - Pod is operational
6. **Termination** - Pod stops (gracefully or forcefully)

## Basic Pod Operations

### Create a Simple Pod
```yaml
# simple-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
    version: v1
spec:
  containers:
  - name: hello-container
    image: nginx:alpine
    ports:
    - containerPort: 80
```

```bash
# Deploy the pod
kubectl apply -f simple-pod.yaml

# Check status
kubectl get pods
kubectl describe pod hello-pod

# Access the application
kubectl port-forward hello-pod 8080:80
```

### Pod with Environment Variables
```yaml
# env-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo-pod
spec:
  containers:
  - name: demo
    image: alpine:latest
    command: ["sleep", "3600"]
    env:
    - name: DEMO_GREETING
      value: "Hello from Kubernetes"
    - name: DEMO_FAREWELL
      value: "Goodbye from the pod"
```

```bash
# Test environment variables
kubectl exec -it env-demo-pod -- env | grep DEMO
```

### Pod with Volume Mount
```yaml
# volume-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-demo-pod
spec:
  containers:
  - name: demo
    image: alpine:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: demo-volume
      mountPath: /data
  volumes:
  - name: demo-volume
    emptyDir: {}
```

## Resource Management

### CPU and Memory Limits
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo-pod
spec:
  containers:
  - name: demo
    image: nginx:alpine
    resources:
      requests:        # Minimum resources needed
        memory: "64Mi"
        cpu: "250m"
      limits:          # Maximum resources allowed
        memory: "128Mi"
        cpu: "500m"
```

### Understanding Resource Units
- **CPU**: `1000m` = 1 CPU core, `500m` = 0.5 CPU core
- **Memory**: `Mi` = Mebibytes, `Gi` = Gibibytes
- **Requests**: Guaranteed minimum resources
- **Limits**: Hard cap on resource usage

**💡 Mentor Tip**: Always set resource requests in production. Kubernetes uses this for scheduling decisions.

## Multi-Container Pods (Advanced)

### Sidecar Pattern Example
```yaml
# sidecar-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
spec:
  containers:
  # Main application container
  - name: web-app
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx

  # Sidecar container for log processing
  - name: log-processor
    image: alpine:latest
    command: ["tail", "-f", "/logs/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /logs

  volumes:
  - name: shared-logs
    emptyDir: {}
```

### When to Use Multi-Container Pods
- **Logging sidecars**: Collect and forward logs
- **Proxy/Ambassador**: Network proxy for main container
- **Init containers**: Setup tasks before main container starts
- **Monitoring agents**: Collect metrics from main application

## Common Pod Patterns

### Init Containers
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-setup
    image: busybox
    command: ['sh', '-c', 'echo Setting up... && sleep 10']
  
  containers:
  - name: main-app
    image: nginx:alpine
```

**Use Case**: Database migrations, configuration setup, waiting for dependencies.

### Health Checks
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: health-check-pod
spec:
  containers:
  - name: web
    image: nginx:alpine
    ports:
    - containerPort: 80
    
    # Readiness probe - when is pod ready to serve traffic?
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 10
    
    # Liveness probe - is the pod still healthy?
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 15
      periodSeconds: 20
```

## Debugging Pods

### Common Debugging Commands
```bash
# Get pod details
kubectl describe pod <pod-name>

# View pod logs
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>  # Multi-container pods

# Execute commands in pod
kubectl exec -it <pod-name> -- sh
kubectl exec -it <pod-name> -c <container-name> -- sh  # Multi-container

# Check pod events
kubectl get events --field-selector involvedObject.name=<pod-name>

# Get pod YAML (current state)
kubectl get pod <pod-name> -o yaml
```

### Common Issues and Solutions

#### Image Pull Errors
```bash
# Problem: ErrImagePull or ImagePullBackOff
kubectl describe pod <pod-name>

# Solutions:
# 1. Check image name/tag spelling
# 2. Ensure image exists in registry
# 3. Check network connectivity to registry
```

#### Pod Stuck in Pending
```bash
# Check node resources
kubectl describe nodes

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Common causes:
# - Insufficient resources
# - Node selector constraints
# - Volume mounting issues
```

#### Container Restart Loops
```bash
# Check logs for error messages
kubectl logs <pod-name> --previous

# Common causes:
# - Application crashes on startup
# - Failed health checks
# - Resource limits exceeded
```

## Best Practices

### Production Guidelines
1. **Always set resource requests and limits**
2. **Use health checks** (readiness and liveness probes)
3. **Meaningful labels** for organization and selection
4. **Avoid privileged containers** unless absolutely necessary
5. **Use specific image tags**, not `latest`

### Development Tips
1. **Use `kubectl port-forward`** for local testing
2. **Keep pod definitions in version control**
3. **Use `kubectl apply`** instead of `create` for updates
4. **Test resource limits** with realistic workloads

## Real-World Context

### When You'll Use Pods Directly
- **Learning and experimentation** (like now!)
- **One-off tasks** or debugging
- **Stateful applications** with specific requirements

### When You Won't Use Pods Directly
- **Production applications** - Use Deployments instead
- **Scalable services** - Use ReplicaSets/Deployments
- **Batch jobs** - Use Jobs or CronJobs

**💡 Mentor Tip**: Think of direct pod creation like writing HTML without a framework. It works, but higher-level abstractions (Deployments) are usually better for real applications.

## What's Next?

Now that you understand pods, you need to learn how they communicate:

**Next Step**: [[Kubernetes/Core Concepts/02 - Services and Networking]]

**Related Topics**:
- [[Kubernetes/Kubernetes Command Reference]] - Pod management commands
- [[Docker/Docker Overview]] - Container fundamentals
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - Debug pod issues

## Summary

Pods are the foundation of Kubernetes. Key takeaways:
- **Pods wrap containers** with shared networking and storage
- **Usually one container per pod** for simplicity
- **Resource management** is crucial for production
- **Health checks** ensure reliability
- **Direct pod usage** is mainly for learning and debugging

Understanding pods deeply will make everything else in Kubernetes make sense!