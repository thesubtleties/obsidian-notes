# K3s for Development

## Why K3s is Perfect for Development

K3s eliminates the complexity barrier that often prevents developers from using Kubernetes locally. Unlike full Kubernetes clusters that require multiple nodes and complex setup, K3s gives you a production-like environment on your development machine.

### Development Advantages
- **Fast startup**: K3s starts in seconds, not minutes
- **Low resource usage**: Runs well on laptops and development VMs
- **Single binary**: No complex installation or dependency management
- **Production compatibility**: Same APIs as full Kubernetes
- **Easy reset**: Simple to tear down and recreate clean environments

**💡 Mentor Tip**: K3s bridges the gap between Docker Compose (simple but limited) and full Kubernetes (powerful but complex). Perfect for learning and development.

## Development Environment Setup

### Local Development Setup
```bash
# Install K3s (if not already done)
curl -sfL https://get.k3s.io | sh -

# Set up kubectl for your user
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(whoami) ~/.kube/config

# Verify setup
kubectl get nodes
kubectl get pods --all-namespaces
```

### Multi-Node Development (Optional)
```bash
# On first node (server)
curl -sfL https://get.k3s.io | sh -
sudo cat /var/lib/rancher/k3s/server/node-token

# On additional nodes (workers)
curl -sfL https://get.k3s.io | K3S_URL=https://<server-ip>:6443 K3S_TOKEN=<token> sh -
```

### Development Configuration
```bash
# Disable Traefik (use your own ingress)
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -

# Disable ServiceLB (if you want to manage load balancing differently)
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=servicelb" sh -

# Custom data directory
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--data-dir=/custom/path" sh -
```

## Development Workflows

### Iterative Development Pattern

#### 1. Create Development Namespace
```bash
# Create isolated namespace for your project
kubectl create namespace my-app-dev

# Set as default for current context
kubectl config set-context --current --namespace=my-app-dev
```

#### 2. Basic Application Deployment
```yaml
# dev-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: my-app-dev
spec:
  replicas: 1                    # Single replica for development
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:dev          # Use development tag
        ports:
        - containerPort: 8080
        env:
        - name: ENV
          value: "development"
        - name: DEBUG
          value: "true"
        resources:
          requests:                # Minimal resources for dev
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
  namespace: my-app-dev
spec:
  selector:
    app: my-app
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```

#### 3. Development Access Patterns
```bash
# Port forwarding for local access
kubectl port-forward service/my-app-service 8080:8080

# Test your application
curl http://localhost:8080

# View logs
kubectl logs deployment/my-app -f
```

### Hot Reloading and Live Development

#### Using Skaffold (Recommended)
```yaml
# skaffold.yaml
apiVersion: skaffold/v2beta26
kind: Config
metadata:
  name: my-app
build:
  artifacts:
  - image: my-app
    docker:
      dockerfile: Dockerfile.dev
deploy:
  kubectl:
    manifests:
    - k8s/dev/*.yaml
portForward:
- resourceType: service
  resourceName: my-app-service
  port: 8080
```

```bash
# Start development with auto-reload
skaffold dev

# Skaffold will:
# 1. Build your image when code changes
# 2. Deploy to K3s
# 3. Forward ports automatically
# 4. Stream logs
```

#### Manual Development Workflow
```bash
# Build and deploy cycle
docker build -t my-app:dev .
kubectl set image deployment/my-app app=my-app:dev
kubectl rollout status deployment/my-app

# Streamline with a script
cat << 'EOF' > dev-deploy.sh
#!/bin/bash
docker build -t my-app:dev .
kubectl set image deployment/my-app app=my-app:dev --namespace=my-app-dev
kubectl rollout status deployment/my-app --namespace=my-app-dev
kubectl port-forward service/my-app-service 8080:8080 --namespace=my-app-dev
EOF
chmod +x dev-deploy.sh
```

## Database Development

### Development Database Setup
```yaml
# dev-postgres.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: my-app-dev
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
        env:
        - name: POSTGRES_PASSWORD
          value: "devpassword"      # Simple password for dev
        - name: POSTGRES_DB
          value: "myapp_dev"
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        emptyDir: {}               # Ephemeral storage for dev

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
  namespace: my-app-dev
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### Database Access Patterns
```bash
# Connect to database from your local machine
kubectl port-forward service/postgres-service 5432:5432 --namespace=my-app-dev

# Connect with psql
psql -h localhost -U postgres -d myapp_dev

# Run database migrations
kubectl exec -it deployment/postgres --namespace=my-app-dev -- psql -U postgres -d myapp_dev -c "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(100));"
```

### Development Data Management
```bash
# Backup development data
kubectl exec deployment/postgres --namespace=my-app-dev -- pg_dump -U postgres myapp_dev > dev-backup.sql

# Restore development data
kubectl exec -i deployment/postgres --namespace=my-app-dev -- psql -U postgres myapp_dev < dev-backup.sql

# Reset development database
kubectl delete deployment postgres --namespace=my-app-dev
kubectl apply -f dev-postgres.yaml
```

## Environment Configuration

### Development vs Production Configuration
```yaml
# config/development.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: my-app-dev
data:
  LOG_LEVEL: "debug"
  CACHE_SIZE: "small"
  EXTERNAL_API_TIMEOUT: "30s"
  FEATURE_NEW_UI: "true"

---
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: my-app-dev
type: Opaque
data:
  API_KEY: ZGV2LWFwaS1rZXk=        # dev-api-key (base64)
  DB_PASSWORD: ZGV2cGFzc3dvcmQ=    # devpassword (base64)
```

### Environment-Specific Deployments
```bash
# Deploy to development
kubectl apply -f config/development.yaml
kubectl apply -f deployments/development.yaml

# Switch to staging namespace for testing
kubectl config set-context --current --namespace=my-app-staging
kubectl apply -f config/staging.yaml
kubectl apply -f deployments/staging.yaml
```

## Testing in K3s

### Integration Testing Setup
```yaml
# test-job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: integration-tests
  namespace: my-app-dev
spec:
  template:
    spec:
      containers:
      - name: tests
        image: my-app:test
        command: ["npm", "run", "test:integration"]
        env:
        - name: API_URL
          value: "http://my-app-service:8080"
        - name: DB_HOST
          value: "postgres-service"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: DB_PASSWORD
      restartPolicy: Never
  backoffLimit: 3
```

```bash
# Run integration tests
kubectl apply -f test-job.yaml

# Check test results
kubectl logs job/integration-tests --namespace=my-app-dev

# Clean up test job
kubectl delete job integration-tests --namespace=my-app-dev
```

### Load Testing in Development
```yaml
# load-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-test
  namespace: my-app-dev
spec:
  containers:
  - name: load-test
    image: williamyeh/wrk
    command: ["wrk"]
    args: ["-t4", "-c100", "-d30s", "http://my-app-service:8080/api/health"]
  restartPolicy: Never
```

## Debugging and Troubleshooting

### Development Debugging Tools
```bash
# Interactive debugging pod
kubectl run debug-pod --image=nicolaka/netshoot --rm -it --restart=Never --namespace=my-app-dev

# Inside the debug pod:
# Test network connectivity
nslookup my-app-service
curl my-app-service:8080/health

# Test database connectivity
nslookup postgres-service
nc -zv postgres-service 5432
```

### Log Management for Development
```bash
# View application logs
kubectl logs deployment/my-app --namespace=my-app-dev -f

# View logs from specific container
kubectl logs deployment/my-app -c app --namespace=my-app-dev

# View logs from all pods with label
kubectl logs -l app=my-app --namespace=my-app-dev

# Export logs for analysis
kubectl logs deployment/my-app --namespace=my-app-dev > app-logs.txt
```

### Performance Monitoring
```bash
# Check resource usage
kubectl top pods --namespace=my-app-dev
kubectl top nodes

# Describe pods for resource information
kubectl describe pod -l app=my-app --namespace=my-app-dev

# Monitor events
kubectl get events --sort-by=.metadata.creationTimestamp --namespace=my-app-dev
```

## Development Best Practices

### Namespace Management
```bash
# Create feature-specific namespaces
kubectl create namespace feature-auth-dev
kubectl create namespace feature-payments-dev

# Clean up when done
kubectl delete namespace feature-auth-dev
```

### Resource Management
```yaml
# Development resource quotas
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: my-app-dev
spec:
  hard:
    requests.cpu: "2"           # Total CPU requests
    requests.memory: 4Gi        # Total memory requests
    limits.cpu: "4"             # Total CPU limits
    limits.memory: 8Gi          # Total memory limits
    pods: "10"                  # Max pods
    persistentvolumeclaims: "5" # Max PVCs
```

### Development Shortcuts
```bash
# Useful aliases for development
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kaf='kubectl apply -f'
alias kdf='kubectl delete -f'
alias klogs='kubectl logs -f'

# Quick pod creation for testing
alias krun='kubectl run tmp-pod --image=alpine --rm -it --restart=Never --'

# Example usage:
krun wget -qO- my-app-service:8080/health
```

## CI/CD Integration

### GitLab CI Example
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy-dev

build:
  stage: build
  script:
    - docker build -t my-app:$CI_COMMIT_SHA .
    - docker push my-app:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - kubectl config use-context development
    - kubectl set image deployment/my-app app=my-app:$CI_COMMIT_SHA --namespace=my-app-dev
    - kubectl rollout status deployment/my-app --namespace=my-app-dev
    - kubectl apply -f test-job.yaml
    - kubectl wait --for=condition=complete job/integration-tests --timeout=300s --namespace=my-app-dev

deploy-dev:
  stage: deploy-dev
  script:
    - kubectl config use-context development
    - kubectl set image deployment/my-app app=my-app:$CI_COMMIT_SHA --namespace=my-app-dev
  only:
    - develop
```

### GitHub Actions Example
```yaml
# .github/workflows/dev-deploy.yml
name: Deploy to Development
on:
  push:
    branches: [ develop ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - name: Build and push image
      run: |
        docker build -t my-app:${{ github.sha }} .
        docker push my-app:${{ github.sha }}
    
    - name: Deploy to K3s
      run: |
        kubectl config use-context development
        kubectl set image deployment/my-app app=my-app:${{ github.sha }} --namespace=my-app-dev
        kubectl rollout status deployment/my-app --namespace=my-app-dev
```

## Development Environment Cleanup

### Regular Cleanup Tasks
```bash
# Clean up completed pods
kubectl delete pods --field-selector=status.phase=Succeeded --namespace=my-app-dev

# Clean up failed pods
kubectl delete pods --field-selector=status.phase=Failed --namespace=my-app-dev

# Clean up old ReplicaSets
kubectl delete replicaset $(kubectl get rs --namespace=my-app-dev -o jsonpath='{.items[?(@.spec.replicas==0)].metadata.name}') --namespace=my-app-dev

# Reset entire development environment
kubectl delete namespace my-app-dev
kubectl create namespace my-app-dev
```

### Storage Cleanup
```bash
# List persistent volumes in use
kubectl get pvc --namespace=my-app-dev

# Clean up unused PVCs
kubectl delete pvc --all --namespace=my-app-dev

# Check K3s storage usage
sudo du -sh /var/lib/rancher/k3s/storage/
```

## Migration from Docker Compose

### Comparison: Docker Compose vs K3s Development
```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "8080:8080"
    environment:
      - ENV=development
    depends_on:
      - postgres
  
  postgres:
    image: postgres:13
    environment:
      - POSTGRES_PASSWORD=devpassword
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**Equivalent K3s setup** provides the same functionality with additional benefits:
- Service discovery through DNS
- Rolling updates and health checks
- Resource management and scaling
- Production-like environment

## Next Steps

Ready to move beyond development? Explore:

**Next Topics**:
- [[Kubernetes/K3s Topics/K3s Production Considerations]] - Production deployment
- [[Kubernetes/Practical Guides/From Docker Compose to K3s]] - Migration guide
- [[Kubernetes/K3s Topics/Migrating K3s to Full K8s]] - When to upgrade

**Related Topics**:
- [[Kubernetes/Core Concepts/03 - Deployments and Scaling]] - Understanding deployments
- [[Docker/Docker Overview]] - Container fundamentals
- [[Git-Github/Git Commands Overview]] - Version control for configs

## Summary

K3s development workflow advantages:

- **Fast iteration** with port-forwarding and log streaming
- **Production-like environment** without complexity
- **Easy database integration** with proper service discovery
- **Environment isolation** through namespaces
- **CI/CD ready** with standard Kubernetes APIs
- **Resource efficient** for local development
- **Simple cleanup** and environment reset

K3s makes Kubernetes development accessible and practical for everyday use!