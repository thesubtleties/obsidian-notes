# Kubernetes Debugging Guide

## The Art of Kubernetes Debugging

Debugging Kubernetes applications is like being a detective - you follow clues from symptoms to root cause. This guide provides battle-tested commands and workflows for diagnosing common issues.

**💡 Mentor Tip**: Always start broad (cluster/node level) and narrow down to specific resources (pods/containers). Most issues fall into categories: configuration, networking, resources, or application errors.

## Quick Diagnosis with k9s

### Navigation and Quick Actions
```bash
# Launch k9s
k9s

# Essential k9s commands:
:deploy or :d    # View deployments
:po              # View pods
:svc             # View services
:cm              # View configmaps
:sec             # View secrets
:events          # View cluster events (great starting point!)

# Quick actions on resources:
r                # Restart deployment/pod
l                # View logs
d                # Describe resource
s                # Shell into pod
e                # Edit resource
Ctrl+k           # Delete resource
/                # Filter resources
```

**💡 Quick Tip**: Start debugging in k9s with `:events` to see recent cluster activity, then drill down to specific resources.

## Pod Debugging Workflow

### 1. Check Pod Status
```bash
# List pods with specific label
kubectl get pod -l app=myapp -n namespace

# Get more details (NODE, IP, etc.)
kubectl get pod -l app=myapp -n namespace -o wide

# Watch pods in real-time
kubectl get pod -l app=myapp -n namespace --watch

# Get pod events
kubectl describe pod pod-name -n namespace | grep -A 20 Events
```

### 2. Examine Logs
```bash
# View recent logs
kubectl logs -n namespace -l app=myapp --tail=50

# View previous container logs (if pod restarted)
kubectl logs -n namespace -l app=myapp --previous --tail=50

# Follow logs in real-time
kubectl logs -n namespace -l app=myapp -f

# Logs from specific container in multi-container pod
kubectl logs -n namespace pod-name -c container-name

# Logs with timestamps
kubectl logs -n namespace pod-name --timestamps

# Logs from all pods with label
kubectl logs -n namespace -l app=myapp --all-containers=true --prefix=true
```

### 3. Describe Pod for Details
```bash
# Full pod description
kubectl describe pod pod-name -n namespace

# Common sections to check:
# - Events (bottom of output)
# - Conditions
# - Container statuses
# - Resource limits/requests
# - Volume mounts
# - Environment variables
```

## Environment and Configuration Debugging

### Check Environment Variables
```bash
# View env vars in pod description
kubectl describe pod pod-name -n namespace | grep -A20 "Environment:"

# Execute printenv in running pod
kubectl exec -n namespace pod-name -- printenv | grep -E "(DATABASE|API|SECRET)"

# Check specific env var
kubectl exec -n namespace pod-name -- printenv DATABASE_URL

# Compare env vars across pods
for pod in $(kubectl get pods -n namespace -l app=myapp -o name); do
  echo "=== $pod ==="
  kubectl exec -n namespace ${pod##*/} -- printenv | grep DATABASE
done
```

### Secret Debugging
```bash
# List all secrets
kubectl get secret -n namespace

# Check secret exists and has expected keys
kubectl get secret secret-name -n namespace -o jsonpath='{.data}' | jq 'keys'

# Decode specific secret value
kubectl get secret secret-name -n namespace -o jsonpath='{.data.password}' | base64 -d

# View all decoded values (CAREFUL - sensitive data!)
kubectl get secret secret-name -n namespace -o json | jq '.data | map_values(@base64d)'

# Check if secret is mounted correctly
kubectl describe pod pod-name -n namespace | grep -A5 "Mounts:"
```

### ConfigMap Debugging
```bash
# List configmaps
kubectl get configmap -n namespace

# View configmap content
kubectl get configmap config-name -n namespace -o yaml

# Check specific key
kubectl get configmap config-name -n namespace -o jsonpath='{.data.application\.properties}'

# Verify configmap is mounted
kubectl exec -n namespace pod-name -- ls -la /path/to/config/
kubectl exec -n namespace pod-name -- cat /path/to/config/file.conf
```

## Database Connection Debugging

### MongoDB Specific Commands
```bash
# Get MongoDB pod name dynamically
MONGO_POD=$(kubectl get pod -n namespace -l app=mongodb -o jsonpath='{.items[0].metadata.name}')

# Test basic connection
kubectl exec -n namespace $MONGO_POD -- mongosh --eval "db.adminCommand('ping')"

# Connect with authentication
kubectl exec -n namespace $MONGO_POD -- mongosh \
  -u admin -p "password" \
  --authenticationDatabase admin \
  --eval "db.getSiblingDB('mydb').getUsers()"

# Test connection string
kubectl exec -n namespace $MONGO_POD -- mongosh \
  "mongodb://user:password@localhost:27017/mydb?authSource=mydb" \
  --eval "db.getName()"

# Check database users
kubectl exec -n namespace $MONGO_POD -- mongosh \
  -u admin -p "password" \
  --authenticationDatabase admin \
  --eval "db.getSiblingDB('mydb').getUsers().forEach(u => print(u.user + ': ' + u.roles.map(r => r.role).join(', ')))"

# List all databases
kubectl exec -n namespace $MONGO_POD -- mongosh \
  -u admin -p "password" \
  --authenticationDatabase admin \
  --eval "db.adminCommand('listDatabases')"
```

### PostgreSQL Debugging
```bash
# Get PostgreSQL pod
PG_POD=$(kubectl get pod -n namespace -l app=postgres -o jsonpath='{.items[0].metadata.name}')

# Test connection
kubectl exec -n namespace $PG_POD -- psql -U postgres -c "SELECT version();"

# List databases
kubectl exec -n namespace $PG_POD -- psql -U postgres -c "\l"

# Check users/roles
kubectl exec -n namespace $PG_POD -- psql -U postgres -c "\du"

# Test specific database connection
kubectl exec -n namespace $PG_POD -- psql -U myuser -d mydb -c "SELECT current_database();"
```

### General Database Debugging
```bash
# Test connectivity from application pod
kubectl exec -n namespace app-pod -- nc -zv database-service 5432
kubectl exec -n namespace app-pod -- nslookup database-service

# Check if database service exists
kubectl get svc -n namespace | grep database

# Verify endpoints
kubectl get endpoints database-service -n namespace
```

## Service and Networking Debugging

### Service Discovery
```bash
# List all services
kubectl get svc -n namespace

# Check service details
kubectl describe svc service-name -n namespace

# Verify endpoints (pods backing the service)
kubectl get endpoints service-name -n namespace

# Test service DNS from within cluster
kubectl run test-pod --image=busybox --rm -it --restart=Never -- \
  nslookup service-name.namespace.svc.cluster.local

# Test service connectivity
kubectl run test-pod --image=nicolaka/netshoot --rm -it --restart=Never -- \
  curl -v http://service-name.namespace.svc.cluster.local:port/health
```

### Ingress Debugging
```bash
# List ingresses
kubectl get ingress -n namespace

# Check ingress details
kubectl describe ingress ingress-name -n namespace

# Verify ingress controller
kubectl get pods -n ingress-nginx  # or your ingress namespace
kubectl logs -n ingress-nginx deployment/ingress-nginx-controller

# Test ingress routing
curl -H "Host: myapp.example.com" http://ingress-ip/
```

### Network Policy Debugging
```bash
# Check if network policies exist
kubectl get networkpolicy -n namespace

# Describe network policy
kubectl describe networkpolicy policy-name -n namespace

# Test connectivity between pods
kubectl exec -n namespace source-pod -- nc -zv target-pod 8080
kubectl exec -n namespace source-pod -- wget -O- http://target-pod:8080/health
```

## Deployment Management

### Restart and Scale Operations
```bash
# Rolling restart (keeps pods running during restart)
kubectl rollout restart deployment app-name -n namespace

# Check rollout status
kubectl rollout status deployment app-name -n namespace

# Scale deployment
kubectl scale deployment app-name --replicas=3 -n namespace

# Scale to zero (stop all pods)
kubectl scale deployment app-name --replicas=0 -n namespace

# Emergency: Delete all pods (they'll be recreated)
kubectl delete pod -l app=myapp -n namespace

# Wait for pods to be ready
kubectl wait --for=condition=ready pod -l app=myapp -n namespace --timeout=60s
```

### Rollback Deployments
```bash
# Check rollout history
kubectl rollout history deployment app-name -n namespace

# Rollback to previous version
kubectl rollout undo deployment app-name -n namespace

# Rollback to specific revision
kubectl rollout undo deployment app-name -n namespace --to-revision=2

# Pause rollout (for debugging)
kubectl rollout pause deployment app-name -n namespace

# Resume rollout
kubectl rollout resume deployment app-name -n namespace
```

## Storage and PVC Debugging

### Persistent Volume Claims
```bash
# List PVCs
kubectl get pvc -n namespace

# Check PVC details
kubectl describe pvc pvc-name -n namespace

# Check what pod is using a PVC
kubectl get pods -n namespace -o json | \
  jq '.items[] | select(.spec.volumes[]?.persistentVolumeClaim.claimName=="pvc-name") | .metadata.name'

# Delete PVC (careful - data loss!)
kubectl delete pvc pvc-name -n namespace

# Force delete stuck PVC
kubectl patch pvc pvc-name -n namespace -p '{"metadata":{"finalizers":null}}'
```

### Volume Debugging
```bash
# Check volume mounts in pod
kubectl describe pod pod-name -n namespace | grep -A10 "Volumes:"

# Verify volume contents
kubectl exec -n namespace pod-name -- ls -la /path/to/volume/

# Check disk usage
kubectl exec -n namespace pod-name -- df -h /path/to/volume/

# Copy files from volume
kubectl cp namespace/pod-name:/path/to/file ./local-file
```

## Resource Issues

### CPU and Memory Debugging
```bash
# Check node resources
kubectl top nodes

# Check pod resources
kubectl top pods -n namespace

# Get resource requests/limits
kubectl describe pod pod-name -n namespace | grep -A5 "Limits:"

# Check for evicted pods
kubectl get pods -n namespace | grep Evicted

# Find resource-hungry pods
kubectl top pods -n namespace --sort-by=cpu
kubectl top pods -n namespace --sort-by=memory
```

### Node Debugging
```bash
# Check node status
kubectl get nodes
kubectl describe node node-name

# Check node conditions
kubectl get nodes -o json | jq '.items[].status.conditions[]'

# SSH to node (if possible)
kubectl debug node/node-name -it --image=busybox

# Check kubelet logs
kubectl logs -n kube-system kubelet-xxxxx
```

## Application-Specific Debugging

### Exec Into Pods
```bash
# Basic shell access
kubectl exec -it -n namespace pod-name -- /bin/bash
kubectl exec -it -n namespace pod-name -- /bin/sh  # if bash not available

# Run commands
kubectl exec -n namespace pod-name -- ps aux
kubectl exec -n namespace pod-name -- netstat -tulpn
kubectl exec -n namespace pod-name -- curl localhost:8080/health

# Multi-container pods
kubectl exec -it -n namespace pod-name -c container-name -- /bin/bash
```

### Debug Containers
```bash
# Add ephemeral debug container
kubectl debug -n namespace pod-name -it --image=busybox --target=container-name

# Copy pod with debug image
kubectl debug -n namespace pod-name -it --image=nicolaka/netshoot --copy-to=debug-pod

# Node debugging
kubectl debug node/node-name -it --image=busybox
```

## Secret and ConfigMap Management

### Creating/Updating Secrets
```bash
# Create secret from literals
kubectl create secret generic secret-name -n namespace \
  --from-literal=username=admin \
  --from-literal=password='complex-password' \
  --dry-run=client -o yaml | kubectl apply -f -

# Create from files
kubectl create secret generic secret-name -n namespace \
  --from-file=tls.crt=./cert.pem \
  --from-file=tls.key=./key.pem

# Update existing secret
kubectl create secret generic secret-name -n namespace \
  --from-literal=password='new-password' \
  --dry-run=client -o yaml | kubectl apply -f -

# Create TLS secret
kubectl create secret tls tls-secret -n namespace \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

### Deployment Editing
```bash
# Edit deployment directly
kubectl edit deployment app-name -n namespace

# Export, modify, apply
kubectl get deployment app-name -n namespace -o yaml > deployment.yaml
# Edit deployment.yaml
kubectl apply -f deployment.yaml

# Patch deployment
kubectl patch deployment app-name -n namespace \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"app","image":"app:v2"}]}}}}'

# Add environment variable
kubectl set env deployment/app-name -n namespace NEW_VAR=value
```

## Backup and Restore Operations

### MongoDB Backup/Restore
```bash
# Backup MongoDB
MONGO_POD=$(kubectl get pod -n namespace -l app=mongodb -o jsonpath='{.items[0].metadata.name}')

# Create backup inside pod
kubectl exec -n namespace $MONGO_POD -- mongodump \
  -u admin -p "password" \
  --authenticationDatabase admin \
  --out /tmp/backup

# Copy backup locally
kubectl cp namespace/$MONGO_POD:/tmp/backup ./mongodb-backup

# Restore from backup
kubectl cp ./mongodb-backup namespace/$MONGO_POD:/tmp/restore
kubectl exec -n namespace $MONGO_POD -- mongorestore \
  -u admin -p "password" \
  --authenticationDatabase admin \
  /tmp/restore

# Drop and recreate user
kubectl exec -n namespace $MONGO_POD -- mongosh \
  -u admin -p "password" \
  --authenticationDatabase admin \
  --eval "
    use mydb;
    db.dropUser('myuser');
    db.createUser({
      user: 'myuser',
      pwd: 'password',
      roles: [
        {role: 'readWrite', db: 'mydb'},
        {role: 'dbAdmin', db: 'mydb'}
      ]
    });
  "
```

### File System Debugging
```bash
# Search for files in local backups
find /path/to/backups/ -name "*.yaml" -o -name "*.env*" | grep -E "(secret|env)"

# Search for specific content
grep -r "DATABASE_URL" /path/to/project/

# Find recently modified files
find /path/to/project/ -type f -mtime -1

# Check file permissions in pod
kubectl exec -n namespace pod-name -- ls -la /app/
kubectl exec -n namespace pod-name -- find /app -type f -name "*.env*"
```

## Advanced Debugging Techniques

### Event Stream Monitoring
```bash
# Watch all events in namespace
kubectl get events -n namespace --watch

# Filter events by object
kubectl get events -n namespace --field-selector involvedObject.name=pod-name

# Sort events by time
kubectl get events -n namespace --sort-by='.lastTimestamp'
```

### API Debugging
```bash
# Get raw API response
kubectl get pod pod-name -n namespace -o json

# Use JSONPath for specific fields
kubectl get pods -n namespace -o jsonpath='{.items[*].spec.containers[*].image}'

# Custom columns
kubectl get pods -n namespace \
  -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[0].image,STATUS:.status.phase
```

### Cluster-Wide Debugging
```bash
# Check all failing pods across cluster
kubectl get pods --all-namespaces | grep -v Running

# Find pods with restarts
kubectl get pods --all-namespaces | awk '$4>0'

# Check recent events cluster-wide
kubectl get events --all-namespaces --sort-by='.lastTimestamp' | tail -50

# Find resource usage by namespace
kubectl top pods --all-namespaces | sort -k3 -nr | head -20
```

## Emergency Recovery Procedures

### Stuck Resources
```bash
# Force delete stuck pod
kubectl delete pod pod-name -n namespace --grace-period=0 --force

# Remove finalizers from stuck resources
kubectl patch pod pod-name -n namespace -p '{"metadata":{"finalizers":null}}'

# Delete stuck namespace
kubectl get namespace stuck-namespace -o json | \
  jq '.spec.finalizers = []' | \
  kubectl replace --raw /api/v1/namespaces/stuck-namespace/finalize -f -
```

### Crash Recovery
```bash
# Get crash logs
kubectl logs -n namespace pod-name --previous

# Get pod termination reason
kubectl get pod pod-name -n namespace -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'

# Check OOM kills
kubectl describe pod pod-name -n namespace | grep -i oom
dmesg | grep -i "killed process"
```

## Debugging Checklist

### Initial Assessment
- [ ] Check cluster events: `kubectl get events -n namespace --sort-by='.lastTimestamp'`
- [ ] Check pod status: `kubectl get pods -n namespace`
- [ ] Check recent logs: `kubectl logs -n namespace -l app=myapp --tail=100`
- [ ] Check resource usage: `kubectl top pods -n namespace`

### Configuration Issues
- [ ] Verify ConfigMaps exist and have correct data
- [ ] Check Secrets are properly created and mounted
- [ ] Confirm environment variables are set correctly
- [ ] Validate volume mounts and permissions

### Networking Issues
- [ ] Check Service endpoints have pods
- [ ] Verify DNS resolution works
- [ ] Test connectivity between pods
- [ ] Check Ingress configuration and controller logs

### Application Issues
- [ ] Examine application logs for errors
- [ ] Check database connectivity
- [ ] Verify external service connections
- [ ] Test health/readiness endpoints

### Resource Issues
- [ ] Check if pods are being evicted
- [ ] Verify resource requests/limits are appropriate
- [ ] Check node capacity and pressure
- [ ] Look for OOMKilled containers

## Best Practices

### Systematic Approach
1. **Start with events** - They often point directly to the issue
2. **Check logs** - Both current and previous container logs
3. **Verify configuration** - Ensure configs, secrets, and env vars are correct
4. **Test connectivity** - Confirm network paths are working
5. **Check resources** - Look for CPU, memory, or storage issues

### Preventive Measures
1. **Set up monitoring** - Prometheus + Grafana for metrics
2. **Configure alerts** - Get notified before issues become critical
3. **Use health checks** - Proper liveness and readiness probes
4. **Resource limits** - Prevent single pods from consuming all resources
5. **Regular backups** - Especially for stateful services

### Documentation
1. **Document debugging steps** - Create runbooks for common issues
2. **Label resources** - Makes filtering and identification easier
3. **Use meaningful names** - Clear naming helps debugging
4. **Keep manifests in version control** - Track what changed
5. **Comment complex configurations** - Future you will thank you

**💡 Final Tip**: The key to effective Kubernetes debugging is understanding the relationships between resources. When something fails, trace the connections: Pod → Service → Ingress, or Pod → ConfigMap/Secret → Volume. Most issues become clear when you map out these dependencies.

## Related Topics
- [[Kubernetes/🎯 k9s Terminal UI Beginners Bible 🎯]] - Interactive debugging
- [[Kubernetes/Kubernetes Command Reference]] - Complete kubectl reference
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - General troubleshooting
- [[Kubernetes/Core Concepts/01 - Pods and Containers]] - Understanding pod lifecycle