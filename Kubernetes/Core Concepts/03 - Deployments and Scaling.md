# 03 - Deployments and Scaling

## The Problem with Bare Pods

In [[Kubernetes/Core Concepts/01 - Pods and Containers|the previous lesson]], you learned about pods. But managing pods directly has problems:

- **Manual management**: You create each pod individually
- **No automatic restart**: If a pod dies, it stays dead
- **No scaling**: Want 5 copies? Create 5 separate pod files
- **No rolling updates**: Updating means deleting old pods and creating new ones

**Deployments solve all these problems.**

## What is a Deployment?

A **Deployment** is a higher-level controller that manages pods for you. Think of it as a smart manager that:
- **Ensures desired state**: Keeps the right number of pods running
- **Handles failures**: Restarts crashed pods automatically
- **Manages updates**: Rolling updates with zero downtime
- **Provides scaling**: Easy horizontal scaling

### Deployment → ReplicaSet → Pods
```
Deployment (manages) → ReplicaSet (manages) → Pods (run containers)
```

**💡 Mentor Tip**: You usually work with Deployments, not ReplicaSets directly. ReplicaSets are the behind-the-scenes mechanism.

## Basic Deployment Example

### Creating Your First Deployment
```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # How many pods we want
  selector:
    matchLabels:
      app: nginx                 # Which pods this deployment manages
  template:                      # Pod template
    metadata:
      labels:
        app: nginx               # Label must match selector
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

### Deploy and Observe
```bash
# Create the deployment
kubectl apply -f nginx-deployment.yaml

# Watch pods being created
kubectl get pods -w

# Check deployment status
kubectl get deployments
kubectl describe deployment nginx-deployment

# See the underlying ReplicaSet
kubectl get replicasets
```

## Understanding Deployment Status

### Deployment Conditions
```bash
kubectl get deployment nginx-deployment -o wide

# Output shows:
# NAME               READY   UP-TO-DATE   AVAILABLE   AGE
# nginx-deployment   3/3     3            3           2m
```

- **READY**: Pods ready to serve traffic / Total desired pods
- **UP-TO-DATE**: Pods updated to latest revision
- **AVAILABLE**: Pods available to users
- **AGE**: Time since deployment created

### Detailed Status
```bash
kubectl describe deployment nginx-deployment

# Look for:
# Conditions:
#   Type           Status  Reason
#   ----           ------  ------
#   Available      True    MinimumReplicasAvailable
#   Progressing    True    NewReplicaSetAvailable
```

## Scaling Applications

### Manual Scaling
```bash
# Scale up to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5

# Watch it happen
kubectl get pods -w

# Scale down to 2 replicas
kubectl scale deployment nginx-deployment --replicas=2
```

### Declarative Scaling (Recommended)
```yaml
# Update nginx-deployment.yaml
spec:
  replicas: 5    # Change from 3 to 5
```

```bash
# Apply the change
kubectl apply -f nginx-deployment.yaml

# Kubernetes handles the scaling automatically
kubectl get pods
```

**💡 Mentor Tip**: Always use declarative scaling in production. It's version-controlled and repeatable.

## Rolling Updates

### Update Strategy Types
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  strategy:
    type: RollingUpdate        # Default strategy
    rollingUpdate:
      maxUnavailable: 1        # Max pods that can be unavailable
      maxSurge: 1              # Max extra pods during update
```

### Performing an Update
```bash
# Method 1: Update image via kubectl
kubectl set image deployment/nginx-deployment nginx=nginx:1.21

# Method 2: Edit deployment directly
kubectl edit deployment nginx-deployment

# Method 3: Update YAML file and apply
# Change image: nginx:1.20 to nginx:1.21 in file
kubectl apply -f nginx-deployment.yaml
```

### Monitoring Updates
```bash
# Watch the rollout
kubectl rollout status deployment/nginx-deployment

# See rollout history
kubectl rollout history deployment/nginx-deployment

# Watch pods during update
kubectl get pods -w
```

### Rolling Back Updates
```bash
# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Rollback to specific revision
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Check rollback status
kubectl rollout status deployment/nginx-deployment
```

## Advanced Deployment Patterns

### Blue-Green Deployment Simulation
```yaml
# Method: Use labels to switch traffic
# Blue deployment (current)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: blue
  template:
    metadata:
      labels:
        app: myapp
        version: blue
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---
# Green deployment (new version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: green
  template:
    metadata:
      labels:
        app: myapp
        version: green
    spec:
      containers:
      - name: app
        image: myapp:v2.0
```

### Canary Deployment Pattern
```yaml
# Primary deployment (90% of traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-primary
spec:
  replicas: 9
  selector:
    matchLabels:
      app: myapp
      track: stable
  template:
    metadata:
      labels:
        app: myapp
        track: stable
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---
# Canary deployment (10% of traffic)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
      track: canary
  template:
    metadata:
      labels:
        app: myapp
        track: canary
    spec:
      containers:
      - name: app
        image: myapp:v2.0
```

## Health Checks and Readiness

### Adding Health Checks to Deployments
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:alpine
        ports:
        - containerPort: 80
        
        # Readiness probe - when is pod ready for traffic?
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 10
          failureThreshold: 3
        
        # Liveness probe - is pod still healthy?
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 15
          periodSeconds: 20
          failureThreshold: 3
```

**💡 Mentor Tip**: Readiness probes prevent traffic to unhealthy pods. Liveness probes restart unhealthy pods. Both are crucial for reliable deployments.

## Deployment + Service Integration

### Complete Web Application Stack
```yaml
# deployment-with-service.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web              # Matches deployment labels
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

```bash
# Deploy everything together
kubectl apply -f deployment-with-service.yaml

# Check the service gets endpoints from deployment
kubectl get endpoints web-service

# Test scaling affects service
kubectl scale deployment web-app --replicas=5
kubectl get endpoints web-service  # Should show 5 IPs now
```

## Resource Management for Deployments

### Setting Resource Limits
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: resource-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
      - name: app
        image: nginx:alpine
        resources:
          requests:              # Guaranteed resources
            memory: "128Mi"
            cpu: "100m"
          limits:                # Maximum resources
            memory: "256Mi"
            cpu: "200m"
```

### Horizontal Pod Autoscaler (HPA)
```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

```bash
# Apply HPA (requires metrics server)
kubectl apply -f hpa.yaml

# Check HPA status
kubectl get hpa

# Generate load to test scaling
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh
# Inside the pod:
while true; do wget -q -O- http://web-service; done
```

## Troubleshooting Deployments

### Common Issues and Solutions

#### Pods Not Starting
```bash
# Check deployment status
kubectl describe deployment <deployment-name>

# Check pods
kubectl get pods
kubectl describe pod <pod-name>

# Common causes:
# - Image pull errors
# - Resource constraints
# - Configuration errors
```

#### Rolling Update Stuck
```bash
# Check rollout status
kubectl rollout status deployment/<deployment-name>

# Check events
kubectl get events --sort-by=.metadata.creationTimestamp

# Force restart if needed
kubectl rollout restart deployment/<deployment-name>
```

#### Scaling Issues
```bash
# Check if HPA is interfering
kubectl get hpa

# Check resource quotas
kubectl describe quota

# Check node capacity
kubectl describe nodes
```

## Deployment Strategies Comparison

### Rolling Update (Default)
- **Pros**: No downtime, gradual rollout
- **Cons**: Mixed versions during update
- **Use when**: Most web applications

### Recreate Strategy
```yaml
spec:
  strategy:
    type: Recreate
```
- **Pros**: Simple, no mixed versions
- **Cons**: Downtime during update
- **Use when**: Stateful apps that can't run multiple versions

### Blue-Green (Manual)
- **Pros**: Instant rollback, full testing before switch
- **Cons**: Requires double resources
- **Use when**: Critical applications, complex testing needed

### Canary (Manual/External Tools)
- **Pros**: Risk mitigation, gradual rollout
- **Cons**: Complex to implement properly
- **Use when**: High-risk updates, large user base

## Best Practices

### Production Guidelines
1. **Always set resource requests and limits**
2. **Use health checks** (readiness and liveness probes)
3. **Set appropriate replica count** (at least 2 for availability)
4. **Use rolling updates** with proper maxUnavailable/maxSurge
5. **Monitor deployment rollouts**
6. **Have rollback procedures** ready

### Development Tips
1. **Start with small replica counts** for faster iteration
2. **Use development images** with shorter update cycles
3. **Test scaling** before deploying to production
4. **Practice rollback procedures** in development

## Real-World Scenarios

### Microservices Architecture
```bash
# Frontend deployment
kubectl create deployment frontend --image=myapp/frontend:v1.0
kubectl scale deployment frontend --replicas=3
kubectl expose deployment frontend --port=80 --type=LoadBalancer

# Backend API deployment
kubectl create deployment api --image=myapp/api:v1.0
kubectl scale deployment api --replicas=5
kubectl expose deployment api --port=8080 --type=ClusterIP

# Database deployment (stateful - we'll cover this later)
kubectl create deployment database --image=postgres:13
kubectl expose deployment database --port=5432 --type=ClusterIP
```

### Staging vs Production Differences
```yaml
# Staging: Smaller scale, faster updates
spec:
  replicas: 1
  strategy:
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

# Production: Higher scale, conservative updates
spec:
  replicas: 5
  strategy:
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

## Legacy and Version Notes

### Deployment API Evolution
- **apps/v1beta1** - ⚠️ Deprecated (pre-1.9)
- **apps/v1beta2** - ⚠️ Deprecated (1.9-1.15)
- **apps/v1** - ✅ Current stable API (1.9+)

### Migration from Old APIs
```bash
# Check for deprecated APIs
kubectl api-versions | grep apps

# Convert old deployments
kubectl convert -f old-deployment.yaml --output-version apps/v1
```

## What's Next?

You now understand how to manage and scale applications with Deployments. Next, you'll learn how to manage configuration and secrets:

**Next Step**: [[Kubernetes/Core Concepts/04 - ConfigMaps and Secrets]]

**Related Topics**:
- [[Kubernetes/Core Concepts/02 - Services and Networking]] - Exposing deployments
- [[Kubernetes/Kubernetes Command Reference]] - Deployment commands
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - Debugging deployments

## Summary

Deployments are the production way to run applications in Kubernetes:

- **Manage pod lifecycle** automatically
- **Handle scaling** declaratively
- **Rolling updates** with zero downtime
- **Automatic rollback** capabilities
- **Health checking** integration
- **Resource management** and autoscaling

Never use bare pods in production - always use Deployments (or other controllers) for reliability and manageability!