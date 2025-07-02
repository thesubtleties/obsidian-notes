# From Docker Compose to K3s

## Why Migrate from Docker Compose?

[[Docker/Docker Overview|Docker Compose]] is great for development, but has limitations:
- **Single host** - can't scale across multiple servers
- **No health checking** - manual intervention when containers fail
- **Limited networking** - basic container-to-container communication
- **No rolling updates** - downtime during deployments
- **Manual scaling** - no automatic scaling based on load

**K3s provides production-grade features** while maintaining operational simplicity.

**💡 Mentor Tip**: Don't migrate everything at once. Start with stateless applications, then move databases and stateful services.

## Migration Strategy Overview

### Phase 1: Prepare and Plan
1. **Audit your Docker Compose** setup
2. **Identify stateful vs stateless** services
3. **Plan persistent storage** migration
4. **Set up K3s** environment

### Phase 2: Migrate Stateless Services
1. **Convert applications** to Kubernetes manifests
2. **Migrate configuration** to ConfigMaps/Secrets
3. **Set up service discovery**
4. **Test networking** and communication

### Phase 3: Migrate Stateful Services
1. **Plan data migration** strategy
2. **Convert databases** with persistent storage
3. **Update application** connections
4. **Verify data integrity**

### Phase 4: Production Features
1. **Add health checks** and monitoring
2. **Implement rolling updates**
3. **Set up ingress** and load balancing
4. **Configure backups**

## Example Migration: Full-Stack Web App

### Original Docker Compose Setup
```yaml
# docker-compose.yml
version: '3.8'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://backend:8080
    depends_on:
      - backend

  backend:
    build: ./backend
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/myapp
      - JWT_SECRET=supersecret
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:13
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

### Step 1: Create Namespace and Configuration
```yaml
# 01-namespace-and-config.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: myapp

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  REACT_APP_API_URL: "http://backend-service:8080"
  DATABASE_HOST: "postgres-service"
  DATABASE_PORT: "5432"
  DATABASE_NAME: "myapp"
  REDIS_URL: "redis://redis-service:6379"

---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: myapp
type: Opaque
data:
  DATABASE_PASSWORD: cGFzc3dvcmQ=    # base64 encoded "password"
  JWT_SECRET: c3VwZXJzZWNyZXQ=       # base64 encoded "supersecret"
```

### Step 2: Migrate Redis (Stateless Cache)
```yaml
# 02-redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:6-alpine
        ports:
        - containerPort: 6379
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"

---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: myapp
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
  type: ClusterIP
```

### Step 3: Migrate PostgreSQL (Stateful Database)
```yaml
# 03-postgres.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: myapp
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_USER
          value: "user"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DATABASE_PASSWORD
        - name: POSTGRES_DB
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: DATABASE_NAME
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        # Health checks
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - user
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - user
          initialDelaySeconds: 5
          periodSeconds: 5
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: myapp
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### Step 4: Migrate Backend API
```yaml
# 04-backend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: myapp
spec:
  replicas: 2                      # Scale horizontally
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: myapp/backend:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          value: "postgresql://user:$(DATABASE_PASSWORD)@postgres-service:5432/myapp"
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DATABASE_PASSWORD
        - name: JWT_SECRET
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: JWT_SECRET
        envFrom:
        - configMapRef:
            name: app-config
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        # Health checks
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
  namespace: myapp
spec:
  selector:
    app: backend
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

### Step 5: Migrate Frontend
```yaml
# 05-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: myapp/frontend:latest
        ports:
        - containerPort: 3000
        envFrom:
        - configMapRef:
            name: app-config
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        # Health checks
        livenessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: myapp
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 3000
  type: LoadBalancer        # External access
```

## Data Migration Strategies

### Database Migration Options

#### Option 1: Backup and Restore (Downtime)
```bash
# 1. Backup from Docker Compose
docker-compose exec postgres pg_dump -U user myapp > backup.sql

# 2. Deploy PostgreSQL in K3s
kubectl apply -f 03-postgres.yaml

# 3. Wait for pod to be ready
kubectl wait --for=condition=ready pod -l app=postgres --namespace=myapp

# 4. Restore to K3s PostgreSQL
kubectl exec -i deployment/postgres --namespace=myapp -- psql -U user myapp < backup.sql

# 5. Verify data
kubectl exec -it deployment/postgres --namespace=myapp -- psql -U user myapp -c "SELECT COUNT(*) FROM users;"
```

#### Option 2: Live Migration (Minimal Downtime)
```bash
# 1. Set up replication from Docker Compose to K3s
# Configure Docker Compose PostgreSQL as master
# Configure K3s PostgreSQL as replica

# 2. Let replication sync

# 3. Quick switch:
#    - Stop applications
#    - Promote K3s PostgreSQL to master
#    - Start K3s applications
#    - Update DNS/routing
```

#### Option 3: Parallel Running (Zero Downtime)
```bash
# 1. Run both systems in parallel
# 2. Gradually shift traffic to K3s
# 3. Sync data between systems
# 4. Decommission Docker Compose when confident
```

### File Storage Migration
```yaml
# If you have file uploads or shared storage
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage-pvc
  namespace: myapp
spec:
  accessModes:
  - ReadWriteMany              # Multiple pods can access
  resources:
    requests:
      storage: 10Gi

---
# Copy data from old system
apiVersion: batch/v1
kind: Job
metadata:
  name: migrate-files
  namespace: myapp
spec:
  template:
    spec:
      containers:
      - name: file-migration
        image: alpine:latest
        command: ["sh", "-c"]
        args:
        - |
          apk add --no-cache rsync openssh-client
          rsync -avz --progress /old-data/ /new-data/
        volumeMounts:
        - name: old-data
          mountPath: /old-data
        - name: new-data
          mountPath: /new-data
      volumes:
      - name: old-data
        hostPath:
          path: /path/to/docker/volumes    # Mount Docker volume
      - name: new-data
        persistentVolumeClaim:
          claimName: shared-storage-pvc
      restartPolicy: Never
```

## Common Migration Patterns

### Environment Variables → ConfigMaps/Secrets
```bash
# Docker Compose environment
environment:
  - DATABASE_URL=postgresql://user:password@postgres:5432/myapp
  - API_KEY=secret123
  - DEBUG=true

# Becomes Kubernetes ConfigMap + Secret
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=postgres-service \
  --from-literal=DATABASE_PORT=5432 \
  --from-literal=DEBUG=true

kubectl create secret generic app-secrets \
  --from-literal=DATABASE_PASSWORD=password \
  --from-literal=API_KEY=secret123
```

### Docker Compose Networks → Kubernetes Services
```yaml
# Docker Compose
services:
  frontend:
    depends_on:
      - backend      # Can reach backend by name

# Kubernetes equivalent - Service provides DNS
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - port: 8080
```

### Docker Compose Volumes → PersistentVolumes
```yaml
# Docker Compose
volumes:
  postgres_data:
services:
  postgres:
    volumes:
      - postgres_data:/var/lib/postgresql/data

# Kubernetes equivalent
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

### Docker Compose Ports → Services and Ingress
```yaml
# Docker Compose
services:
  frontend:
    ports:
      - "3000:3000"    # Host port mapping

# Kubernetes - Use Service + optional Ingress
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer    # Or NodePort for external access
  ports:
  - port: 80
    targetPort: 3000
```

## Deployment Workflow Comparison

### Docker Compose Workflow
```bash
# Update and deploy
docker-compose build
docker-compose up -d

# View logs
docker-compose logs -f

# Scale service
docker-compose up -d --scale backend=3
```

### K3s Workflow
```bash
# Build and push image
docker build -t myapp/backend:v1.1 .
docker push myapp/backend:v1.1

# Rolling update
kubectl set image deployment/backend backend=myapp/backend:v1.1 --namespace=myapp

# View logs
kubectl logs -f deployment/backend --namespace=myapp

# Scale service
kubectl scale deployment backend --replicas=3 --namespace=myapp
```

## Migration Testing Strategy

### Parallel Testing Setup
```yaml
# Test deployment alongside production
apiVersion: v1
kind: Namespace
metadata:
  name: myapp-test

# Deploy test version with different configurations
# Use same database with different schema
# Test all functionality before switching
```

### Validation Checklist
```bash
#!/bin/bash
# validate-migration.sh

NAMESPACE="myapp"

echo "=== Checking Pod Status ==="
kubectl get pods --namespace=$NAMESPACE

echo "=== Checking Services ==="
kubectl get services --namespace=$NAMESPACE

echo "=== Testing Database Connection ==="
kubectl exec deployment/backend --namespace=$NAMESPACE -- \
  sh -c 'curl -f http://localhost:8080/health'

echo "=== Testing Frontend Access ==="
kubectl port-forward service/frontend-service 3000:80 --namespace=$NAMESPACE &
FORWARD_PID=$!
sleep 5
curl -f http://localhost:3000
kill $FORWARD_PID

echo "=== Checking Logs for Errors ==="
kubectl logs --since=10m deployment/backend --namespace=$NAMESPACE | grep -i error

echo "=== Performance Test ==="
kubectl exec deployment/backend --namespace=$NAMESPACE -- \
  sh -c 'time curl -s http://localhost:8080/api/users >/dev/null'
```

## Rollback Strategy

### Quick Rollback to Docker Compose
```bash
# Keep Docker Compose ready during migration
# If issues occur, quickly switch back:

# 1. Update DNS/load balancer to point to old system
# 2. Stop K3s services
# 3. Start Docker Compose services
# 4. Restore database if needed

docker-compose up -d
```

### Gradual Rollback
```bash
# Reduce K3s replicas while bringing back Compose
kubectl scale deployment backend --replicas=1 --namespace=myapp
kubectl scale deployment frontend --replicas=1 --namespace=myapp

# Start Compose services
docker-compose up -d

# Gradually shift traffic back
```

## Advanced Migration Features

### Blue-Green Migration
```bash
# Deploy new version alongside old
kubectl apply -f k8s-manifests/ --namespace=myapp-green

# Test green environment
./validate-migration.sh myapp-green

# Switch traffic
kubectl patch service frontend-service -p '{"spec":{"selector":{"version":"green"}}}' --namespace=myapp

# Clean up blue environment
kubectl delete namespace myapp-blue
```

### Monitoring During Migration
```yaml
# Add monitoring to K3s deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: monitoring
  namespace: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: monitoring
  template:
    metadata:
      labels:
        app: monitoring
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
```

## Troubleshooting Common Issues

### Service Discovery Problems
```bash
# Test DNS resolution
kubectl run test-pod --image=busybox --rm -it --restart=Never --namespace=myapp -- nslookup backend-service

# Check service endpoints
kubectl get endpoints --namespace=myapp

# Verify label selectors
kubectl get pods --show-labels --namespace=myapp
```

### Storage Issues
```bash
# Check PVC status
kubectl get pvc --namespace=myapp

# Check storage class
kubectl get storageclass

# Verify volume mounts
kubectl describe pod postgres-xxx --namespace=myapp
```

### Performance Issues
```bash
# Compare resource usage
docker stats
kubectl top pods --namespace=myapp

# Check resource limits
kubectl describe pod backend-xxx --namespace=myapp
```

## Post-Migration Optimization

### Enable Production Features
```yaml
# Add horizontal pod autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: backend-hpa
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: backend
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

### Implement Ingress
```yaml
# Add ingress for better routing
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  namespace: myapp
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8080
```

## Next Steps

After successful migration, explore:

**Next Topics**:
- [[Kubernetes/K3s Topics/K3s for Development]] - Development workflows
- [[Kubernetes/K3s Topics/K3s Production Considerations]] - Production optimization
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - Debug issues

**Related Topics**:
- [[Docker/Docker Overview]] - Container fundamentals
- [[Kubernetes/Core Concepts/02 - Services and Networking]] - Service discovery
- [[Kubernetes/Core Concepts/05 - Storage and Volumes]] - Persistent storage

## Summary

Docker Compose to K3s migration provides:

- **Horizontal scaling** across multiple nodes
- **Self-healing** with automatic restarts
- **Rolling updates** with zero downtime
- **Advanced networking** with service discovery
- **Resource management** with limits and quotas
- **Production features** like health checks and monitoring

The migration effort pays off with significantly improved reliability and scalability!