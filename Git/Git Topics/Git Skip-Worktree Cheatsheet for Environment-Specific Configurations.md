## Core Concept
`skip-worktree` is a Git feature that tells Git to ignore local changes to specific files. This is ideal for environment-specific configurations that need to differ between development, staging, and production environments.

---

## Basic Commands

### Mark files to be ignored locally
```bash
git update-index --skip-worktree <file-path>
```

### Stop ignoring local changes
```bash
git update-index --no-skip-worktree <file-path>
```

### List all files marked with skip-worktree
```bash
git ls-files -v | grep '^S'
```

---

## Application in Our Atria Project

### Files We're Managing with skip-worktree
- `backend/atria/Dockerfile` - Contains environment-specific dependencies (eventlet for Socket.IO)
- `backend/atria/init.sh` - Contains Gunicorn configuration with eventlet worker
- `docker-compose.yml` - Contains environment-specific service configurations
- `frontend/Dockerfile` - Contains environment-specific build steps

### Why We're Using skip-worktree
1. **Socket.IO Configuration**: Production needs eventlet for WebSockets
2. **Environment Variables**: Different settings for dev/prod
3. **Resource Allocation**: Different worker/thread settings for different environments
4. **Logging**: Different verbosity levels between environments

---

## Workflow for Our Project

### Initial Setup
```bash
# After resolving conflicts and setting up files correctly for your environment
git add .
git commit -m "Resolve conflicts and configure for my environment"
git push  # If appropriate

# Mark files to be ignored
git update-index --skip-worktree backend/atria/Dockerfile
git update-index --skip-worktree backend/atria/init.sh
git update-index --skip-worktree docker-compose.yml
git update-index --skip-worktree frontend/Dockerfile
```

### When Pulling Updates
```bash
# Normal git pull won't affect your skipped files
git pull origin main
```

### When You Need to Update Skipped Files
```bash
# Temporarily track changes to get updates
git update-index --no-skip-worktree backend/atria/init.sh

# Pull changes
git pull origin main

# Resolve any conflicts
# ...

# Re-apply your environment-specific changes
# ...

# Mark file to be ignored again
git update-index --skip-worktree backend/atria/init.sh
```

---

## Socket.IO Specific Configuration

### Required Components
1. **Dockerfile Addition**:
   ```dockerfile
   RUN pip install --no-cache-dir eventlet
   ```

2. **init.sh Gunicorn Configuration**:
   ```bash
   exec python -u -m gunicorn "api.wsgi:app" \
       --bind 0.0.0.0:5000 \
       --worker-class eventlet \  # Critical for Socket.IO
       --workers 1 \
       --threads 2 \
       # other options...
   ```

3. **Flask App Configuration**:
   ```python
   socketio = SocketIO(app, cors_allowed_origins="*")  # Or specific origins
   ```

---

## Troubleshooting

### If skip-worktree isn't working
```bash
# Check if files are properly marked
git ls-files -v | grep '^S'

# If not listed, re-apply skip-worktree
git update-index --skip-worktree <file-path>
```

### If you get merge conflicts on skipped files
```bash
# Temporarily un-skip the file
git update-index --no-skip-worktree <file-path>

# Resolve the conflict
# ...

# Re-skip the file
git update-index --skip-worktree <file-path>
```

### If you need to reset a file to remote version
```bash
git update-index --no-skip-worktree <file-path>
git checkout origin/main -- <file-path>
# Make your environment-specific changes
git update-index --skip-worktree <file-path>
```

---

## Helper Script for Our Project

```bash
#!/bin/bash
# File: docker-config-toggle.sh

DOCKER_FILES=(
  "backend/atria/Dockerfile"
  "backend/atria/init.sh"
  "docker-compose.yml"
  "frontend/Dockerfile"
)

case "$1" in
  "ignore")
    echo "🔒 Ignoring local changes to Docker configuration files..."
    for file in "${DOCKER_FILES[@]}"; do
      git update-index --skip-worktree "$file"
      echo "  ✅ Now ignoring: $file"
    done
    ;;
  "track")
    echo "👁️ Tracking changes to Docker configuration files..."
    for file in "${DOCKER_FILES[@]}"; do
      git update-index --no-skip-worktree "$file"
      echo "  ✅ Now tracking: $file"
    done
    ;;
  "status")
    echo "📋 Files currently ignored with skip-worktree:"
    git ls-files -v | grep '^S'
    ;;
  *)
    echo "Usage: $0 [ignore|track|status]"
    echo "  ignore: Ignore local changes to Docker config files"
    echo "  track:  Track changes to Docker config files"
    echo "  status: Show which files are currently ignored"
    exit 1
    ;;
esac
```

Save as `docker-config-toggle.sh`, make executable with `chmod +x docker-config-toggle.sh`, and use with:
```bash
./docker-config-toggle.sh ignore  # Ignore changes
./docker-config-toggle.sh track   # Track changes
./docker-config-toggle.sh status  # Check status
```

---

## Best Practices

1. **Document which files are skipped** in your project README
2. **Commit a template version** of configuration files for new developers
3. **Use environment variables** where possible instead of hardcoding values
4. **Consider `.gitignore` for sensitive files** that should never be committed
5. **Use `skip-worktree` only for files that must exist in the repo** but need local modifications