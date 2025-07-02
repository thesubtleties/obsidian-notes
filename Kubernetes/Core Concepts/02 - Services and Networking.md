# 02 - Services and Networking

## The Networking Problem

Imagine you have a web application running in a [[Kubernetes/Core Concepts/01 - Pods and Containers|pod]]. The pod gets an IP address, but:
- **Pod IPs change** when pods restart
- **Pod IPs are internal** - not accessible from outside the cluster
- **Multiple pods** of the same app need load balancing

**Services solve this problem** by providing stable networking for pods.

## What is a Service?

A **Service** is a stable network endpoint that routes traffic to a set of pods. Think of it as a load balancer with a fixed IP address and DNS name.

### Key Concepts
- **Selector**: How the service finds pods to route to
- **Port**: What port the service listens on
- **TargetPort**: What port on the pods to forward to
- **Service Types**: How the service is exposed

## Service Types Explained

### 1. ClusterIP (Default)
**Purpose**: Internal communication within cluster
**Access**: Only from inside the cluster

```yaml
# clusterip-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web          # Routes to pods with label app=web
  ports:
  - port: 80          # Service listens on port 80
    targetPort: 8080  # Forwards to pod port 8080
  type: ClusterIP
```

**💡 Mentor Tip**: This is perfect for database services, internal APIs, or any service that doesn't need external access.

### 2. NodePort
**Purpose**: External access through node IP
**Access**: `<NodeIP>:<NodePort>` from outside cluster

```yaml
# nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080   # External port (30000-32767)
  type: NodePort
```

**Use Case**: Development, testing, or when you don't have a load balancer.

### 3. LoadBalancer
**Purpose**: External access through cloud load balancer
**Access**: External IP provided by cloud provider

```yaml
# loadbalancer-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-loadbalancer
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
```

**💡 K3s Note**: On K3s, LoadBalancer services work if you have a compatible load balancer (like MetalLB) installed.

## Practical Example: Web App with Service

### Step 1: Create the Application Pods
```yaml
# web-deployment.yaml
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
        app: web        # Important: Service selector matches this
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

### Step 2: Create the Service
```yaml
# web-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web            # Matches pods with label app=web
  ports:
  - port: 80            # Service port
    targetPort: 80      # Pod port
  type: ClusterIP
```

### Step 3: Deploy and Test
```bash
# Deploy everything
kubectl apply -f web-deployment.yaml
kubectl apply -f web-service.yaml

# Check services
kubectl get services

# Test internal connectivity
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- web-service
```

## Service Discovery

### DNS-Based Discovery
Kubernetes automatically creates DNS records for services:

```bash
# Inside any pod, you can access:
curl http://web-service           # Same namespace
curl http://web-service.default   # Specific namespace
curl http://web-service.default.svc.cluster.local  # Full DNS name
```

### Environment Variables
Kubernetes injects service information as environment variables:

```bash
# Check environment variables in a pod
kubectl exec -it <pod-name> -- env | grep SERVICE

# Example output:
# WEB_SERVICE_SERVICE_HOST=10.43.123.45
# WEB_SERVICE_SERVICE_PORT=80
```

## Advanced Service Configurations

### Service with Multiple Ports
```yaml
apiVersion: v1
kind: Service
metadata:
  name: multi-port-service
spec:
  selector:
    app: web
  ports:
  - name: http          # Name the ports for clarity
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  type: ClusterIP
```

### Service with Session Affinity
```yaml
apiVersion: v1
kind: Service
metadata:
  name: sticky-service
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 8080
  sessionAffinity: ClientIP  # Route same client to same pod
```

## External Services (ExternalName)

### Connect to External Database
```yaml
# external-db-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: external-database
spec:
  type: ExternalName
  externalName: database.example.com
```

Now pods can connect to `external-database` as if it were internal.

## Headless Services

### When You Need Pod IPs Directly
```yaml
# headless-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: headless-service
spec:
  clusterIP: None       # This makes it headless
  selector:
    app: database
  ports:
  - port: 5432
```

**Use Case**: Databases, stateful applications where you need to connect to specific pod instances.

## Networking Deep Dive

### How Pod-to-Pod Communication Works

1. **Same Pod**: Containers communicate via `localhost`
2. **Same Node**: Direct communication via pod IPs
3. **Different Nodes**: CNI (Container Network Interface) handles routing
4. **Service Communication**: Service proxy routes to healthy pods

### Network Policies (Security)
```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: web-netpol
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 80
```

**💡 Mentor Tip**: Network policies are like firewall rules for pods. Start simple and add restrictions as needed.

## Load Balancing Algorithms

### Default: Round Robin
Traffic is distributed evenly across healthy pods.

### Session Affinity Options
- **None** (default): Round robin
- **ClientIP**: Route same client IP to same pod

```bash
# Check service endpoints (actual pod IPs)
kubectl get endpoints web-service

# Watch traffic distribution
kubectl logs -f deployment/web-app
```

## Troubleshooting Services

### Common Service Issues

#### Service Not Accessible
```bash
# Debug checklist:
# 1. Check service exists
kubectl get services

# 2. Check service has endpoints
kubectl get endpoints <service-name>

# 3. Check pod labels match service selector
kubectl get pods --show-labels

# 4. Test from inside cluster
kubectl run debug --image=busybox --rm -it --restart=Never -- wget -qO- <service-name>
```

#### No Endpoints
```bash
# Problem: Service has no endpoints
kubectl describe service <service-name>

# Solutions:
# 1. Check pod labels match service selector
kubectl get pods --show-labels
kubectl describe service <service-name>

# 2. Check pods are running and ready
kubectl get pods
kubectl describe pod <pod-name>
```

#### Connection Refused
```bash
# Problem: Connection refused to service
# Solutions:
# 1. Check targetPort matches container port
kubectl describe service <service-name>
kubectl describe pod <pod-name>

# 2. Test direct pod connectivity
kubectl port-forward <pod-name> 8080:80
```

## Real-World Patterns

### Microservices Communication
```yaml
# Frontend service
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  selector:
    app: frontend
  ports:
  - port: 80
  type: LoadBalancer

---
# Backend API service
apiVersion: v1
kind: Service
metadata:
  name: backend-api
spec:
  selector:
    app: backend
  ports:
  - port: 8080
  type: ClusterIP

---
# Database service
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
  type: ClusterIP
```

### Development vs Production Services

**Development**: Use NodePort for easy external access
```yaml
type: NodePort
nodePort: 30080
```

**Production**: Use LoadBalancer or Ingress for proper load balancing
```yaml
type: LoadBalancer
```

## Service vs Ingress

### When to Use Services
- **Internal communication** between pods
- **Simple external access** (NodePort, LoadBalancer)
- **Database connections**
- **API services**

### When to Use Ingress (Coming Next)
- **HTTP/HTTPS routing** based on hostname or path
- **SSL termination**
- **Multiple services** behind one external IP
- **Advanced routing rules**

**💡 Mentor Tip**: Services handle network routing, Ingress handles HTTP routing. You often use both together.

## Performance Considerations

### Service Overhead
- **Minimal CPU impact** - handled by kube-proxy
- **Network latency** - small additional hop
- **Connection pooling** - reuse connections when possible

### Scaling Services
```bash
# Services automatically scale with pods
kubectl scale deployment web-app --replicas=10

# Check endpoints update
kubectl get endpoints web-service
```

## Best Practices

### Production Guidelines
1. **Use ClusterIP by default** - expose externally only when needed
2. **Name your ports** in multi-port services
3. **Set resource limits** on pods behind services
4. **Monitor service health** with readiness probes
5. **Use network policies** for security

### Development Tips
1. **Use port-forward** for testing instead of NodePort
2. **Check service endpoints** when debugging connectivity
3. **Use meaningful service names** for easy discovery
4. **Test internal communication** with temporary pods

## Legacy and Version Notes

### Kubernetes Service Evolution
- **v1.0+**: Basic service types (ClusterIP, NodePort, LoadBalancer)
- **v1.7+**: ExternalName services
- **v1.8+**: Headless services improvements
- **v1.9+**: IPVS load balancing option

### Common Migration Issues
- **iptables vs IPVS**: K3s uses iptables by default
- **Service CIDR**: Different clusters may use different IP ranges
- **DNS changes**: Service DNS format has been stable since v1.3

## What's Next?

You now understand how pods communicate through services. Next, you'll learn about managing multiple pod instances:

**Next Step**: [[Kubernetes/Core Concepts/03 - Deployments and Scaling]]

**Related Topics**:
- [[Kubernetes/Kubernetes Command Reference]] - Service management commands
- [[Kubernetes/Core Concepts/01 - Pods and Containers]] - Pod fundamentals
- [[HTTP/HTTP Overview]] - Understanding web protocols

## Summary

Services provide stable networking for pods:
- **ClusterIP**: Internal cluster communication
- **NodePort**: External access via node IP
- **LoadBalancer**: External access via cloud load balancer
- **Service discovery**: DNS and environment variables
- **Load balancing**: Automatic traffic distribution

Services are the foundation of Kubernetes networking - master them and the rest becomes much clearer!