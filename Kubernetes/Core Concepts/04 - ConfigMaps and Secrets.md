# 04 - ConfigMaps and Secrets

## The Configuration Problem

When building applications, you need to handle different types of configuration:
- **Database connection strings** that change per environment
- **API keys and passwords** that must be kept secure
- **Feature flags** that toggle functionality
- **Configuration files** that applications need to read

**Hard-coding these in container images is a bad practice.** Kubernetes provides ConfigMaps and Secrets to externalize configuration.

## ConfigMaps: Non-Sensitive Configuration

### What is a ConfigMap?
A **ConfigMap** stores configuration data as key-value pairs that pods can consume. Think of it as a way to inject configuration into your containers without rebuilding images.

### Basic ConfigMap Example
```yaml
# app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "postgres.example.com"
  database_port: "5432"
  log_level: "info"
  feature_new_ui: "true"
  app.properties: |
    server.port=8080
    server.servlet.context-path=/api
    logging.level.root=INFO
```

```bash
# Create the ConfigMap
kubectl apply -f app-config.yaml

# View ConfigMap
kubectl get configmaps
kubectl describe configmap app-config
```

## Secrets: Sensitive Configuration

### What is a Secret?
A **Secret** stores sensitive data like passwords, tokens, and keys. Similar to ConfigMaps but with additional security measures (base64 encoding, access controls).

### Basic Secret Example
```yaml
# app-secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  database_password: cGFzc3dvcmQxMjM=  # base64 encoded "password123"
  api_key: YWJjZGVmZ2hpams=             # base64 encoded "abcdefghijk"
```

```bash
# Create the Secret
kubectl apply -f app-secrets.yaml

# View Secret (data is encoded)
kubectl get secrets
kubectl describe secret app-secrets

# Decode secret values
kubectl get secret app-secrets -o jsonpath='{.data.database_password}' | base64 -d
```

**💡 Mentor Tip**: Base64 is encoding, not encryption. Secrets provide access control and audit trails, but the data isn't encrypted at rest by default.

## Creating ConfigMaps and Secrets

### ConfigMap Creation Methods

#### From Literal Values
```bash
kubectl create configmap app-config \
  --from-literal=database_host=postgres.example.com \
  --from-literal=database_port=5432 \
  --from-literal=log_level=info
```

#### From Files
```bash
# Create a config file
echo "server.port=8080" > app.properties
echo "logging.level=INFO" >> app.properties

# Create ConfigMap from file
kubectl create configmap app-config --from-file=app.properties

# Create from entire directory
kubectl create configmap app-config --from-file=config-dir/
```

#### From Environment File
```bash
# Create env file
cat <<EOF > app.env
DATABASE_HOST=postgres.example.com
DATABASE_PORT=5432
LOG_LEVEL=info
EOF

# Create ConfigMap from env file
kubectl create configmap app-config --from-env-file=app.env
```

### Secret Creation Methods

#### From Literal Values
```bash
kubectl create secret generic app-secrets \
  --from-literal=database_password=password123 \
  --from-literal=api_key=abcdefghijk
```

#### From Files
```bash
# Create files with sensitive data
echo -n 'password123' > db_password.txt
echo -n 'abcdefghijk' > api_key.txt

# Create Secret from files
kubectl create secret generic app-secrets \
  --from-file=database_password=db_password.txt \
  --from-file=api_key=api_key.txt

# Clean up files
rm db_password.txt api_key.txt
```

#### TLS Secrets
```bash
# Create TLS secret from certificate files
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

## Using ConfigMaps and Secrets in Pods

### Environment Variables

#### ConfigMap as Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sleep", "3600"]
    env:
    # Single value from ConfigMap
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_host
    
    # All keys from ConfigMap as env vars
    envFrom:
    - configMapRef:
        name: app-config
```

#### Secret as Environment Variables
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sleep", "3600"]
    env:
    # Single value from Secret
    - name: DATABASE_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: database_password
    
    # All keys from Secret as env vars
    envFrom:
    - secretRef:
        name: app-secrets
```

### Volume Mounts

#### ConfigMap as Volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

**Result**: Files created in `/etc/config/` with ConfigMap keys as filenames.

#### Secret as Volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
      defaultMode: 0400  # Read-only for owner
```

### Selective Volume Mounting
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sleep", "3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
      - key: app.properties      # Only mount this key
        path: application.conf   # Rename the file
        mode: 0644
```

## Real-World Example: Web Application

### Complete Configuration Setup
```yaml
# web-app-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: web-app-config
data:
  ENVIRONMENT: "production"
  LOG_LEVEL: "info"
  REDIS_HOST: "redis-service"
  REDIS_PORT: "6379"
  nginx.conf: |
    server {
        listen 80;
        server_name localhost;
        
        location / {
            proxy_pass http://app-backend:8080;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }

---
apiVersion: v1
kind: Secret
metadata:
  name: web-app-secrets
type: Opaque
data:
  DATABASE_PASSWORD: cGFzc3dvcmQxMjM=  # password123
  JWT_SECRET: bXlzdXBlcnNlY3JldGp3dA==     # mysupersecretjwt
  API_KEY: YWJjZGVmZ2hpams=              # abcdefghijk

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
      - name: app
        image: myapp:v1.0
        ports:
        - containerPort: 8080
        
        # Environment variables from ConfigMap and Secret
        envFrom:
        - configMapRef:
            name: web-app-config
        - secretRef:
            name: web-app-secrets
        
        # Additional specific env vars
        env:
        - name: DATABASE_HOST
          value: "postgres-service"
        - name: DATABASE_USER
          value: "app_user"
        
        # Mount config files
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: app-secrets
          mountPath: /etc/secrets
          readOnly: true
      
      volumes:
      - name: nginx-config
        configMap:
          name: web-app-config
          items:
          - key: nginx.conf
            path: default.conf
      - name: app-secrets
        secret:
          secretName: web-app-secrets
          defaultMode: 0400
```

### Testing the Configuration
```bash
# Deploy everything
kubectl apply -f web-app-config.yaml

# Check environment variables
kubectl exec -it deployment/web-app -- env | grep -E "(ENVIRONMENT|DATABASE|API_KEY)"

# Check mounted files
kubectl exec -it deployment/web-app -- ls -la /etc/nginx/conf.d/
kubectl exec -it deployment/web-app -- cat /etc/nginx/conf.d/default.conf

# Check secret files
kubectl exec -it deployment/web-app -- ls -la /etc/secrets/
kubectl exec -it deployment/web-app -- cat /etc/secrets/DATABASE_PASSWORD
```

## ConfigMap and Secret Updates

### Updating ConfigMaps
```bash
# Method 1: Edit directly
kubectl edit configmap app-config

# Method 2: Replace from file
kubectl create configmap app-config --from-file=new-config.properties --dry-run=client -o yaml | kubectl apply -f -

# Method 3: Update YAML and apply
kubectl apply -f updated-config.yaml
```

### Automatic Reload (Volume Mounts)
```yaml
# ConfigMaps mounted as volumes update automatically
# Pods will see new files within ~60 seconds
apiVersion: v1
kind: Pod
metadata:
  name: auto-reload-pod
spec:
  containers:
  - name: app
    image: alpine:latest
    command: ["sh", "-c", "while true; do cat /config/database_host; sleep 10; done"]
    volumeMounts:
    - name: config
      mountPath: /config
  volumes:
  - name: config
    configMap:
      name: app-config
```

**💡 Mentor Tip**: Environment variables don't update automatically - you need to restart pods. Volume mounts update automatically.

### Rolling Restart After Config Changes
```bash
# Force pods to restart and pick up new environment variables
kubectl rollout restart deployment/web-app

# Check rollout status
kubectl rollout status deployment/web-app
```

## Security Best Practices

### ConfigMap Security
- **Never put sensitive data** in ConfigMaps
- **Use meaningful names** for organization
- **Version control** ConfigMap YAML files
- **Validate configuration** before applying

### Secret Security
- **Use RBAC** to control access to secrets
- **Enable encryption at rest** in production clusters
- **Rotate secrets regularly**
- **Use external secret management** for production (HashiCorp Vault, AWS Secrets Manager)
- **Never commit secrets** to version control

### Secret Management Example
```yaml
# RBAC for secret access
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: secret-reader
rules:
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-secrets
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
roleRef:
  kind: Role
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
```

## Multi-Environment Configuration

### Environment-Specific ConfigMaps
```yaml
# development-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: development
data:
  DATABASE_HOST: "dev-postgres"
  LOG_LEVEL: "debug"
  CACHE_SIZE: "small"

---
# production-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  DATABASE_HOST: "prod-postgres-cluster"
  LOG_LEVEL: "warn"
  CACHE_SIZE: "large"
```

### Using Kustomize for Environment Management
```yaml
# base/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"

# overlays/development/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- ../../base
patchesStrategicMerge:
- configmap-patch.yaml

# overlays/development/configmap-patch.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "debug"
```

## Troubleshooting Configuration Issues

### Common Problems and Solutions

#### ConfigMap/Secret Not Found
```bash
# Check if resource exists
kubectl get configmaps
kubectl get secrets

# Check namespace
kubectl get configmaps -n <namespace>

# Describe to see details
kubectl describe configmap <name>
```

#### Environment Variables Not Set
```bash
# Check pod environment
kubectl exec -it <pod-name> -- env

# Check for configuration errors
kubectl describe pod <pod-name>

# Look for events
kubectl get events --sort-by=.metadata.creationTimestamp
```

#### Volume Mount Issues
```bash
# Check mounted files
kubectl exec -it <pod-name> -- ls -la <mount-path>

# Check volume configuration
kubectl describe pod <pod-name>

# Common issues:
# - Wrong mount path
# - File permissions
# - ConfigMap key names
```

#### Configuration Not Updating
```bash
# For environment variables - restart deployment
kubectl rollout restart deployment/<name>

# For volume mounts - wait or restart
kubectl delete pod <pod-name>  # Will be recreated by deployment
```

## Legacy and Version Notes

### API Evolution
- **v1**: Current stable API for ConfigMaps and Secrets
- **Immutable ConfigMaps/Secrets**: Added in Kubernetes 1.19+

### Immutable ConfigMaps (Kubernetes 1.19+)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: immutable-config
immutable: true  # Cannot be updated after creation
data:
  key: value
```

**Benefits**: Better performance, prevents accidental updates

## Integration with Other Tools

### Helm Integration
```yaml
# values.yaml
config:
  database_host: postgres.example.com
  log_level: info

secrets:
  database_password: password123
  api_key: secret-key

# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Chart.Name }}-config
data:
  {{- range $key, $value := .Values.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
```

### External Secret Operators
```yaml
# Example: External Secrets Operator
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: vault-backend
spec:
  provider:
    vault:
      server: "https://vault.example.com"
      path: "secret"
      auth:
        kubernetes:
          mountPath: "kubernetes"
          role: "demo"
```

## Best Practices Summary

### Development
1. **Use ConfigMaps** for non-sensitive configuration
2. **Use Secrets** for passwords, tokens, certificates
3. **Mount as volumes** when possible (automatic updates)
4. **Test configuration changes** in development first

### Production
1. **Enable secret encryption at rest**
2. **Use external secret management** systems
3. **Implement secret rotation** procedures
4. **Monitor configuration access** with audit logging
5. **Use immutable ConfigMaps** for stable configuration

## What's Next?

You now understand how to manage configuration in Kubernetes. Next, you'll learn about persistent storage:

**Next Step**: [[Kubernetes/Core Concepts/05 - Storage and Volumes]]

**Related Topics**:
- [[Kubernetes/Core Concepts/03 - Deployments and Scaling]] - Using config in deployments
- [[Security/HTTP/CSRF/The Beginner's Bible to CSRF (Cross-Site Request Forgery)]] - Security concepts
- [[Kubernetes/Kubernetes Command Reference]] - ConfigMap and Secret commands

## Summary

Configuration management in Kubernetes:

- **ConfigMaps**: Non-sensitive configuration data
- **Secrets**: Sensitive data with access controls
- **Environment variables**: Simple key-value injection
- **Volume mounts**: File-based configuration with auto-updates
- **Security**: RBAC, encryption, external secret management
- **Multi-environment**: Namespace separation and Kustomize

Proper configuration management is crucial for secure, maintainable applications!