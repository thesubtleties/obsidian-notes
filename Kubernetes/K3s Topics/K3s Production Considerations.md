# K3s Production Considerations

## Is K3s Production Ready?

**Yes, K3s is production ready.** Many companies run K3s in production for:
- **Edge computing** deployments
- **IoT** and resource-constrained environments
- **Small to medium** production workloads
- **Development** and **staging** environments
- **CI/CD** pipeline runners

However, K3s has different trade-offs compared to full Kubernetes that you need to understand.

**💡 Mentor Tip**: K3s is not "Kubernetes Lite" - it's a different distribution optimized for simplicity and resource efficiency. The core APIs are identical to full Kubernetes.

## K3s vs Full Kubernetes: Production Comparison

### When to Use K3s in Production
- **Single-node** or **small clusters** (2-5 nodes)
- **Resource-constrained** environments
- **Edge locations** with limited connectivity
- **Simple applications** without complex networking needs
- **Teams wanting** operational simplicity
- **Cost-sensitive** deployments

### When to Consider Full Kubernetes
- **Large clusters** (10+ nodes)
- **Complex networking** requirements (service mesh, advanced ingress)
- **Enterprise features** (advanced RBAC, admission controllers)
- **Multi-tenancy** with strict isolation
- **High availability** across multiple data centers
- **Managed services** integration (EKS, GKE, AKS)

## Production Architecture Patterns

### Single-Node Production
```bash
# High-performance single node
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik --disable=servicelb" sh -

# Use external load balancer and ingress
# Better resource utilization
# Suitable for:
# - Small applications
# - Development/staging
# - Edge deployments
```

**Pros**: Simple, cost-effective, easy to manage
**Cons**: Single point of failure, limited scalability

### Multi-Node High Availability
```bash
# Server node 1 (with embedded etcd)
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --cluster-init

# Server node 2
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://<node1-ip>:6443

# Server node 3
curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://<node1-ip>:6443

# Worker nodes
curl -sfL https://get.k3s.io | K3S_URL=https://<server-ip>:6443 K3S_TOKEN=SECRET sh -
```

**Pros**: High availability, better resource utilization
**Cons**: More complex setup, requires odd number of server nodes

### External Database Setup
```bash
# For larger productions, use external database
curl -sfL https://get.k3s.io | sh -s - server \
  --datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database"
  
# Supported databases:
# - MySQL
# - PostgreSQL
# - etcd (external cluster)
```

## Security Hardening

### Network Security
```bash
# Disable unnecessary services
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="
  --disable=traefik
  --disable=servicelb
  --disable=metrics-server
  --disable=local-storage
" sh -

# Custom CNI for advanced networking
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--flannel-backend=none" sh -
```

### Pod Security Standards
```yaml
# pod-security-policy.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### RBAC Configuration
```yaml
# production-rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: production-deployer
rules:
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "create", "update", "patch"]
- apiGroups: [""]
  resources: ["services", "configmaps", "secrets"]
  verbs: ["get", "list", "create", "update", "patch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: production-deployers
subjects:
- kind: User
  name: deploy-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: production-deployer
  apiGroup: rbac.authorization.k8s.io
```

### Secret Management
```bash
# Enable secret encryption at rest
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--secrets-encryption" sh -

# Use external secret management
kubectl apply -f https://raw.githubusercontent.com/external-secrets/external-secrets/main/deploy/crds/bundle.yaml
```

## Resource Management and Monitoring

### Production Resource Configuration
```yaml
# production-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: production-app
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: production-app
  template:
    metadata:
      labels:
        app: production-app
    spec:
      containers:
      - name: app
        image: myapp:v1.2.3        # Specific version, never :latest
        ports:
        - containerPort: 8080
        
        # Resource limits are crucial in production
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        
        # Health checks for reliability
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 3
        
        # Security context
        securityContext:
          allowPrivilegeEscalation: false
          runAsNonRoot: true
          runAsUser: 1000
          capabilities:
            drop:
            - ALL
        
        # Environment-specific configuration
        env:
        - name: ENV
          value: "production"
        - name: LOG_LEVEL
          value: "warn"
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
```

### Monitoring Setup
```yaml
# monitoring-stack.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring

---
# Prometheus for metrics
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: storage
          mountPath: /prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: storage
        persistentVolumeClaim:
          claimName: prometheus-storage

---
# Grafana for visualization
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:latest
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              name: grafana-secret
              key: admin-password
        volumeMounts:
        - name: storage
          mountPath: /var/lib/grafana
      volumes:
      - name: storage
        persistentVolumeClaim:
          claimName: grafana-storage
```

### Log Management
```yaml
# logging-stack.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: fluent-bit
  template:
    metadata:
      labels:
        name: fluent-bit
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:1.9
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
        - name: config
          mountPath: /fluent-bit/etc
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: config
        configMap:
          name: fluent-bit-config
```

## Backup and Disaster Recovery

### Automated Backup Strategy
```bash
# Backup script for K3s
#!/bin/bash
# backup-k3s.sh

BACKUP_DIR="/backup/k3s/$(date +%Y%m%d_%H%M%S)"
mkdir -p $BACKUP_DIR

# Backup K3s data
sudo cp -r /var/lib/rancher/k3s $BACKUP_DIR/

# Backup application data (if using local storage)
kubectl get pvc --all-namespaces -o json > $BACKUP_DIR/pvcs.json

# Backup configurations
kubectl get all --all-namespaces -o yaml > $BACKUP_DIR/all-resources.yaml
kubectl get configmaps --all-namespaces -o yaml > $BACKUP_DIR/configmaps.yaml
kubectl get secrets --all-namespaces -o yaml > $BACKUP_DIR/secrets.yaml

# Compress backup
tar -czf $BACKUP_DIR.tar.gz $BACKUP_DIR
rm -rf $BACKUP_DIR

echo "Backup completed: $BACKUP_DIR.tar.gz"
```

### Disaster Recovery Procedures
```bash
# Recovery script
#!/bin/bash
# restore-k3s.sh

BACKUP_FILE=$1
if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup-file.tar.gz>"
    exit 1
fi

# Stop K3s
sudo systemctl stop k3s

# Extract backup
tar -xzf $BACKUP_FILE

# Restore K3s data
sudo rm -rf /var/lib/rancher/k3s
sudo cp -r k3s /var/lib/rancher/

# Start K3s
sudo systemctl start k3s

# Wait for K3s to be ready
while ! kubectl get nodes; do
    sleep 5
done

# Restore applications
kubectl apply -f all-resources.yaml

echo "Recovery completed"
```

### External Backup Solutions
```yaml
# velero-backup.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: velero

---
# Velero for cluster backups
apiVersion: apps/v1
kind: Deployment
metadata:
  name: velero
  namespace: velero
spec:
  replicas: 1
  selector:
    matchLabels:
      app: velero
  template:
    metadata:
      labels:
        app: velero
    spec:
      containers:
      - name: velero
        image: velero/velero:latest
        command:
        - /velero
        - server
        - --default-backup-storage-location=aws
        - --backup-location-config=region=us-west-2,s3ForcePathStyle="true",s3Url=https://minio.example.com
        env:
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: cloud-credentials
              key: access-key
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: cloud-credentials
              key: secret-key
```

## Production Deployment Strategies

### Blue-Green Deployment
```bash
# Blue-green deployment script
#!/bin/bash

APP_NAME=$1
NEW_VERSION=$2
NAMESPACE=${3:-production}

# Deploy green version
kubectl create deployment ${APP_NAME}-green --image=${APP_NAME}:${NEW_VERSION} --namespace=${NAMESPACE}
kubectl scale deployment ${APP_NAME}-green --replicas=3 --namespace=${NAMESPACE}

# Wait for green to be ready
kubectl rollout status deployment/${APP_NAME}-green --namespace=${NAMESPACE}

# Switch service to green
kubectl patch service ${APP_NAME}-service -p '{"spec":{"selector":{"version":"green"}}}' --namespace=${NAMESPACE}

# Verify deployment
if [ $? -eq 0 ]; then
    echo "Deployment successful, cleaning up blue version"
    kubectl delete deployment ${APP_NAME}-blue --namespace=${NAMESPACE}
    kubectl create deployment ${APP_NAME}-blue --image=${APP_NAME}:${NEW_VERSION} --namespace=${NAMESPACE}
else
    echo "Deployment failed, rolling back"
    kubectl patch service ${APP_NAME}-service -p '{"spec":{"selector":{"version":"blue"}}}' --namespace=${NAMESPACE}
    kubectl delete deployment ${APP_NAME}-green --namespace=${NAMESPACE}
fi
```

### Canary Deployment
```yaml
# canary-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-stable
  namespace: production
spec:
  replicas: 9               # 90% of traffic
  selector:
    matchLabels:
      app: myapp
      version: stable
  template:
    metadata:
      labels:
        app: myapp
        version: stable
    spec:
      containers:
      - name: app
        image: myapp:v1.0

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-canary
  namespace: production
spec:
  replicas: 1               # 10% of traffic
  selector:
    matchLabels:
      app: myapp
      version: canary
  template:
    metadata:
      labels:
        app: myapp
        version: canary
    spec:
      containers:
      - name: app
        image: myapp:v2.0

---
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: production
spec:
  selector:
    app: myapp              # Routes to both stable and canary
  ports:
  - port: 80
    targetPort: 8080
```

## Performance Optimization

### Node Configuration
```bash
# Optimize K3s for production
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="
  --node-name production-node-1
  --kubelet-arg=max-pods=110
  --kubelet-arg=cluster-dns=10.43.0.10
  --kubelet-arg=cluster-domain=cluster.local
  --kube-apiserver-arg=service-node-port-range=30000-32767
  --kube-controller-manager-arg=node-monitor-period=2s
  --kube-controller-manager-arg=node-monitor-grace-period=16s
" sh -
```

### Resource Quotas and Limits
```yaml
# namespace-quotas.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "8"
    requests.memory: 16Gi
    limits.cpu: "16"
    limits.memory: 32Gi
    pods: "20"
    persistentvolumeclaims: "10"
    services: "10"
    secrets: "20"
    configmaps: "20"

---
apiVersion: v1
kind: LimitRange
metadata:
  name: production-limits
  namespace: production
spec:
  limits:
  - default:              # Default limits
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:       # Default requests
      cpu: "100m"
      memory: "128Mi"
    type: Container
```

### Horizontal Pod Autoscaler
```yaml
# production-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: production-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
      - type: Pods
        value: 2
        periodSeconds: 60
```

## High Availability Considerations

### Load Balancer Configuration
```yaml
# external-load-balancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: app-external
  namespace: production
  annotations:
    metallb.universe.tf/address-pool: production-pool
spec:
  type: LoadBalancer
  selector:
    app: production-app
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  externalTrafficPolicy: Local  # For better performance
```

### Database High Availability
```yaml
# postgres-ha.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-ha
  namespace: production
spec:
  serviceName: postgres-ha
  replicas: 3
  selector:
    matchLabels:
      app: postgres-ha
  template:
    metadata:
      labels:
        app: postgres-ha
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_REPLICATION_MODE
          value: master
        - name: POSTGRES_REPLICATION_USER
          value: replicator
        - name: POSTGRES_REPLICATION_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: replication-password
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 50Gi
      storageClassName: fast-ssd
```

## Maintenance and Updates

### K3s Update Strategy
```bash
# Update K3s with minimal downtime
#!/bin/bash

# Drain nodes one by one
for node in $(kubectl get nodes -o name); do
    kubectl drain $node --ignore-daemonsets --delete-emptydir-data
    
    # Update K3s on the node
    ssh $node "curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.25.4+k3s1 sh -"
    
    # Wait for node to be ready
    kubectl wait --for=condition=Ready $node --timeout=300s
    
    # Uncordon the node
    kubectl uncordon $node
done
```

### Application Update Procedures
```bash
# Rolling update script
#!/bin/bash

APP_NAME=$1
NEW_VERSION=$2
NAMESPACE=${3:-production}

# Update deployment
kubectl set image deployment/${APP_NAME} app=${APP_NAME}:${NEW_VERSION} --namespace=${NAMESPACE}

# Monitor rollout
kubectl rollout status deployment/${APP_NAME} --namespace=${NAMESPACE} --timeout=600s

if [ $? -ne 0 ]; then
    echo "Rollout failed, rolling back"
    kubectl rollout undo deployment/${APP_NAME} --namespace=${NAMESPACE}
    exit 1
fi

echo "Update completed successfully"
```

## Troubleshooting Production Issues

### Production Debugging Toolkit
```yaml
# debug-toolkit.yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-toolkit
  namespace: production
spec:
  containers:
  - name: debug
    image: nicolaka/netshoot
    command: ["sleep", "3600"]
    resources:
      requests:
        memory: "64Mi"
        cpu: "100m"
      limits:
        memory: "128Mi"
        cpu: "200m"
  restartPolicy: Never
```

### Log Analysis
```bash
# Production log analysis
#!/bin/bash

NAMESPACE=${1:-production}
APP=${2:-production-app}

echo "=== Recent Events ==="
kubectl get events --sort-by=.metadata.creationTimestamp --namespace=${NAMESPACE} | tail -20

echo "=== Pod Status ==="
kubectl get pods -l app=${APP} --namespace=${NAMESPACE} -o wide

echo "=== Resource Usage ==="
kubectl top pods -l app=${APP} --namespace=${NAMESPACE}

echo "=== Application Logs (last 100 lines) ==="
kubectl logs -l app=${APP} --namespace=${NAMESPACE} --tail=100

echo "=== Error Logs ==="
kubectl logs -l app=${APP} --namespace=${NAMESPACE} --since=1h | grep -i error
```

## Migration to Full Kubernetes

### When to Consider Migration
- **Cluster size** exceeds 10 nodes
- **Complex networking** requirements
- **Advanced features** needed (service mesh, advanced ingress)
- **Multi-tenancy** requirements
- **Managed service** benefits outweigh operational overhead

### Migration Strategy
```bash
# Export K3s resources for migration
kubectl get all --all-namespaces -o yaml > k3s-export.yaml

# Clean up K3s-specific resources
sed -i '/resourceVersion:/d' k3s-export.yaml
sed -i '/uid:/d' k3s-export.yaml
sed -i '/creationTimestamp:/d' k3s-export.yaml

# Apply to new cluster
kubectl apply -f k3s-export.yaml --validate=false
```

## Next Steps

Now that you understand K3s production considerations, explore:

**Next Topics**:
- [[Kubernetes/K3s Topics/Migrating K3s to Full K8s]] - When and how to migrate
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - Debug production issues
- [[Kubernetes/Practical Guides/From Docker Compose to K3s]] - Migration patterns

**Related Topics**:
- [[VPS-Linux/VPS Setup and Management]] - Server management
- [[Security/HTTP/CSRF/The Beginner's Bible to CSRF (Cross-Site Request Forgery)]] - Security concepts
- [[Deployment Guides/VPS Deployment]] - Alternative deployment strategies

## Summary

K3s production readiness depends on:

- **Proper architecture** for your scale and availability needs
- **Security hardening** with RBAC, encryption, and pod security
- **Resource management** with quotas, limits, and monitoring
- **Backup and recovery** procedures
- **Performance optimization** for your workload
- **Maintenance procedures** for updates and troubleshooting

K3s can handle serious production workloads when properly configured and maintained!