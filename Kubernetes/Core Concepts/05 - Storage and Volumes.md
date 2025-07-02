# 05 - Storage and Volumes

## The Storage Problem

Containers are ephemeral - when they restart, all data inside is lost. But real applications need to:
- **Store database files** that persist across restarts
- **Share files** between containers in the same pod
- **Cache data** to improve performance
- **Store logs** for debugging and monitoring

**Volumes solve the data persistence problem** in Kubernetes.

## Understanding Kubernetes Storage

### Storage Hierarchy
```
StorageClass → PersistentVolume → PersistentVolumeClaim → Pod Volume
```

- **StorageClass**: Defines types of storage available (SSD, HDD, cloud storage)
- **PersistentVolume (PV)**: Actual storage resource in the cluster
- **PersistentVolumeClaim (PVC)**: Request for storage by a pod
- **Volume**: Mount point in the pod that uses the PVC

**💡 Mentor Tip**: Think of it like ordering food - StorageClass is the menu, PVC is your order, PV is the actual meal, and Volume is putting it on your table.

## Volume Types Overview

### Ephemeral Volumes (Pod Lifetime)
- **emptyDir**: Shared storage within a pod
- **configMap/secret**: Configuration mounted as files
- **downwardAPI**: Pod metadata as files

### Persistent Volumes (Beyond Pod Lifetime)
- **hostPath**: Node's local filesystem (development only)
- **nfs**: Network File System
- **Cloud storage**: AWS EBS, Azure Disk, GCP Persistent Disk
- **Local**: Local SSDs with node affinity

## EmptyDir Volumes (Pod-Scoped Storage)

### Basic EmptyDir Example
```yaml
# emptydir-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ["sh", "-c", "while true; do echo $(date) >> /shared/log.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
  
  - name: reader
    image: busybox
    command: ["sh", "-c", "while true; do cat /shared/log.txt; sleep 10; done"]
    volumeMounts:
    - name: shared-storage
      mountPath: /shared
  
  volumes:
  - name: shared-storage
    emptyDir: {}
```

```bash
# Deploy and test
kubectl apply -f emptydir-pod.yaml

# Check writer logs
kubectl logs emptydir-demo -c writer

# Check reader logs
kubectl logs emptydir-demo -c reader

# The reader should see the same data the writer creates
```

### EmptyDir with Memory Storage
```yaml
volumes:
- name: memory-storage
  emptyDir:
    medium: Memory     # Store in RAM instead of disk
    sizeLimit: 1Gi     # Limit to 1GB
```

**Use Cases**: Caching, temporary processing, inter-container communication

## HostPath Volumes (Node Storage)

### Development Example
```yaml
# hostpath-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: nginx:alpine
    volumeMounts:
    - name: host-storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: host-storage
    hostPath:
      path: /tmp/nginx-data    # Path on the node
      type: DirectoryOrCreate  # Create if doesn't exist
```

**⚠️ Warning**: HostPath is only for development. In production, pods might move between nodes and lose access to the data.

## Persistent Volumes and Claims

### Creating a Persistent Volume
```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 5Gi
  accessModes:
  - ReadWriteOnce      # RWO: One pod can mount for read-write
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:        # Which node has this storage
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node1
```

### Creating a Persistent Volume Claim
```yaml
# persistent-volume-claim.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi      # Request 3Gi (available in the 5Gi PV)
  storageClassName: local-storage
```

### Using PVC in a Pod
```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo
spec:
  containers:
  - name: app
    image: nginx:alpine
    volumeMounts:
    - name: persistent-storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: local-pvc
```

```bash
# Deploy everything in order
kubectl apply -f persistent-volume.yaml
kubectl apply -f persistent-volume-claim.yaml
kubectl apply -f pod-with-pvc.yaml

# Check PV and PVC status
kubectl get pv
kubectl get pvc

# Test persistence
kubectl exec -it pvc-demo -- sh
echo "Hello from persistent storage" > /usr/share/nginx/html/index.html
exit

# Delete and recreate pod
kubectl delete pod pvc-demo
kubectl apply -f pod-with-pvc.yaml

# Data should still be there
kubectl exec -it pvc-demo -- cat /usr/share/nginx/html/index.html
```

## Access Modes Explained

### ReadWriteOnce (RWO)
- **One pod** can mount the volume for read-write
- **Most common** for databases, application data
- **Example**: Database storage, log files

### ReadOnlyMany (ROX)
- **Multiple pods** can mount the volume read-only
- **Use case**: Shared configuration, static content
- **Example**: Shared assets, documentation

### ReadWriteMany (RWX)
- **Multiple pods** can mount the volume for read-write
- **Requires special storage** (NFS, cloud file systems)
- **Use case**: Shared application data, multi-writer scenarios

```yaml
# RWX Example (requires NFS or similar)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: shared-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
  - ReadWriteMany
  nfs:
    server: nfs-server.example.com
    path: /shared/data
```

## Storage Classes (Dynamic Provisioning)

### What is a StorageClass?
A **StorageClass** defines different types of storage available in your cluster. Instead of manually creating PVs, StorageClasses can automatically provision storage when PVCs are created.

### Basic StorageClass Example
```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/no-provisioner  # For static provisioning
parameters:
  type: ssd
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

### Cloud StorageClass Examples

#### AWS EBS
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: aws-ebs-gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
```

#### Google Cloud Persistent Disk
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gcp-ssd
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
  replication-type: none
```

### Dynamic Provisioning with StorageClass
```yaml
# dynamic-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: fast-ssd  # Automatically provisions storage
```

**💡 K3s Note**: K3s includes a local-path storage class by default that creates hostPath volumes automatically.

## StatefulSets and Storage

### StatefulSet with Persistent Storage
```yaml
# statefulset-with-storage.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: database
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_PASSWORD
          value: password
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
  
  # Volume claim template - creates one PVC per pod
  volumeClaimTemplates:
  - metadata:
      name: postgres-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
      storageClassName: local-storage
```

**Key Difference**: StatefulSets create individual PVCs for each pod replica, ensuring each database instance has its own storage.

## Real-World Storage Patterns

### Database with Persistent Storage
```yaml
# Complete database setup
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: fast-ssd

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
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
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
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
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc

---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
  type: ClusterIP
```

### Web Application with Shared Storage
```yaml
# Shared storage for web app assets
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-assets-pvc
spec:
  accessModes:
  - ReadWriteMany      # Multiple pods can access
  resources:
    requests:
      storage: 5Gi
  storageClassName: nfs-storage

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web
        image: nginx:alpine
        volumeMounts:
        - name: shared-assets
          mountPath: /usr/share/nginx/html/assets
        - name: logs
          mountPath: /var/log/nginx
      volumes:
      - name: shared-assets
        persistentVolumeClaim:
          claimName: shared-assets-pvc
      - name: logs
        emptyDir: {}
```

## Storage Performance and Optimization

### Storage Performance Considerations
```yaml
# High-performance storage configuration
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: high-perf-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi
  storageClassName: premium-ssd  # Fast storage class
```

### Storage Resource Limits
```yaml
# Container with storage limits
apiVersion: v1
kind: Pod
metadata:
  name: storage-limited-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    resources:
      limits:
        ephemeral-storage: 1Gi    # Limit ephemeral storage
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    emptyDir:
      sizeLimit: 500Mi            # Limit emptyDir size
```

## Backup and Recovery Strategies

### Volume Snapshots (Kubernetes 1.17+)
```yaml
# Volume snapshot
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: postgres-snapshot
spec:
  volumeSnapshotClassName: csi-hostpath-snapclass
  source:
    persistentVolumeClaimName: postgres-pvc
```

### Backup Job Example
```yaml
# Backup job
apiVersion: batch/v1
kind: Job
metadata:
  name: backup-database
spec:
  template:
    spec:
      containers:
      - name: backup
        image: postgres:13
        command: 
        - pg_dump
        - -h
        - postgres-service
        - -U
        - postgres
        - mydb
        env:
        - name: PGPASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: backup-storage
          mountPath: /backup
      volumes:
      - name: backup-storage
        persistentVolumeClaim:
          claimName: backup-pvc
      restartPolicy: Never
```

## Troubleshooting Storage Issues

### Common Storage Problems

#### PVC Stuck in Pending
```bash
# Check PVC status
kubectl describe pvc <pvc-name>

# Common causes:
# - No available PV matches the PVC requirements
# - StorageClass not found
# - Insufficient resources
# - AccessMode not supported

# Check available PVs
kubectl get pv

# Check storage classes
kubectl get storageclass
```

#### Pod Stuck in Pending (Volume Issues)
```bash
# Check pod events
kubectl describe pod <pod-name>

# Look for volume mounting errors
kubectl get events --sort-by=.metadata.creationTimestamp

# Common issues:
# - PVC not bound
# - Volume already mounted by another pod (RWO)
# - Node doesn't have access to storage
```

#### Performance Issues
```bash
# Check storage metrics
kubectl top pods
kubectl top nodes

# Test storage performance inside pod
kubectl exec -it <pod-name> -- dd if=/dev/zero of=/data/test bs=1M count=100

# Check volume details
kubectl describe pv <pv-name>
```

### Storage Debugging Commands
```bash
# List all storage resources
kubectl get pv,pvc,storageclass

# Check volume details
kubectl describe pv <pv-name>
kubectl describe pvc <pvc-name>

# Check pod volume mounts
kubectl describe pod <pod-name>

# Get storage events
kubectl get events --field-selector reason=FailedMount

# Check node storage
kubectl describe node <node-name>
```

## K3s Storage Specifics

### Default Local-Path Storage
```bash
# K3s automatically creates a local-path storage class
kubectl get storageclass

# Default PVC uses local-path
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  # storageClassName: local-path (default)
```

### K3s Storage Location
```bash
# Default storage location in K3s
/var/lib/rancher/k3s/storage/

# Check storage usage
sudo du -sh /var/lib/rancher/k3s/storage/
```

## Best Practices

### Development Environment
1. **Use emptyDir** for temporary data
2. **Use local-path** storage class for simple persistence
3. **Test backup/restore** procedures
4. **Monitor storage usage** to avoid full disks

### Production Environment
1. **Use appropriate storage classes** (SSD for databases)
2. **Set resource requests and limits**
3. **Implement backup strategies**
4. **Monitor storage performance**
5. **Use StatefulSets** for databases
6. **Plan for disaster recovery**

### Security
1. **Encrypt data at rest** when possible
2. **Use appropriate access modes**
3. **Secure backup storage**
4. **Implement proper RBAC** for storage resources

## Legacy and Version Notes

### Storage API Evolution
- **storage.k8s.io/v1** - Current stable API (Kubernetes 1.6+)
- **VolumeSnapshots** - Beta in 1.17, GA in 1.20
- **CSI (Container Storage Interface)** - GA in 1.13

### Migration Considerations
- **In-tree to CSI drivers** - Cloud providers moving to CSI
- **Volume expansion** - Supported for most storage types (1.11+)
- **Volume cloning** - Supported in newer versions (1.15+)

## What's Next?

You now understand Kubernetes storage fundamentals. Let's explore K3s-specific topics and real-world deployment scenarios:

**Next Steps**:
- [[Kubernetes/K3s Topics/K3s for Development]] - Development workflows
- [[Kubernetes/Practical Guides/From Docker Compose to K3s]] - Migration guide

**Related Topics**:
- [[Kubernetes/Core Concepts/03 - Deployments and Scaling]] - Using storage in deployments
- [[SQL/SQL Docs & Resources]] - Database concepts
- [[Docker/Docker Overview]] - Container storage concepts

## Summary

Kubernetes storage provides data persistence:

- **EmptyDir**: Shared storage within pods
- **PersistentVolumes**: Cluster-wide storage resources
- **PersistentVolumeClaims**: Storage requests by pods
- **StorageClasses**: Dynamic storage provisioning
- **Access Modes**: Control how volumes can be mounted
- **StatefulSets**: Persistent storage for stateful applications

Understanding storage is crucial for running databases and stateful applications in Kubernetes!