# 🎯 k9s Terminal UI Beginners Bible 🎯

## What is k9s and Why You'll Love It

**k9s** is a terminal-based UI that makes managing Kubernetes clusters feel like a video game. Instead of typing long kubectl commands, you navigate with keyboard shortcuts, view live updates, and manage resources interactively.

Think of k9s as:
- **htop for Kubernetes** - See everything happening in real-time
- **File manager for K8s** - Navigate resources like directories
- **Interactive kubectl** - Execute commands with single keystrokes
- **Debug dashboard** - View logs, exec into pods, all from one interface

**💡 Mentor Tip**: If you're tired of typing `kubectl get pods -n namespace --watch` repeatedly, k9s will change your life. It's especially powerful when debugging issues across multiple namespaces.

## Installation

```bash
# macOS (Homebrew)
brew install k9s

# Linux (Homebrew)
brew install k9s

# Windows (Scoop)
scoop install k9s

# Download binary directly (all platforms)
# Visit: https://github.com/derailed/k9s/releases
# Download appropriate binary for your OS

# Verify installation
k9s version

# For K3s users - works perfectly!
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
k9s
```

## Getting Started (2 Minutes to Productivity)

### Launch k9s
```bash
# Basic launch (uses default kubeconfig)
k9s

# Specify context
k9s --context production

# Specify namespace
k9s -n my-namespace

# Read-only mode (safe for production)
k9s --readonly
```

### First Time Launch
When you start k9s, you'll see:
```
 ____  __.________       
|    |/ _/   __   \______
|      < \____    /  ___/
|    |  \   /    /\___ \ 
|____|__ \ /____//____  >
        \/            \/ 

# Shows cluster info, CPU/Memory usage
# Lists pods in current namespace
# Updates in real-time!
```

## Essential Navigation

### The k9s Mental Model
```
Cluster Info (top)
    ↓
Resource List (main area)
    ↓
Actions (keyboard shortcuts)
    ↓
Command/Filter Line (bottom)
```

### Core Navigation Keys
```bash
# Resource Navigation
:ns         # List namespaces
:po         # List pods (most used!)
:svc        # List services  
:deploy     # List deployments
:cm         # List configmaps
:sec        # List secrets
:pv         # List persistent volumes
:node       # List nodes

# Quick switches (type colon first)
:pod        # Same as :po
:service    # Same as :svc
:deployment # Same as :deploy

# Navigation within lists
↑/↓ or j/k  # Move up/down
Enter       # Drill down into resource
Esc         # Go back
/           # Filter resources (super useful!)
```

## Working with Pods (Most Common Use Case)

### View and Filter Pods
```bash
# In k9s, type:
:po         # Show all pods in current namespace

# Filter pods (press /)
/web        # Show only pods with "web" in name
/Running    # Show only running pods
/api|worker # Show pods matching "api" OR "worker"

# Clear filter
Esc or Ctrl+C
```

### Pod Actions (The Power Features)
```bash
# With a pod highlighted:
Enter       # Show pod details (containers, events, conditions)
l           # Show logs (game changer!)
s           # Shell into pod
d           # Describe pod
Ctrl+k      # Kill pod
e           # Edit pod (opens in $EDITOR)
y           # Show YAML

# In logs view:
0           # Show all containers
1,2,3...    # Show specific container
f           # Toggle follow mode
w           # Toggle word wrap
/           # Search in logs
Ctrl+s      # Save logs to file
```

### Advanced Pod Management
```bash
# Port forwarding
Shift+f     # Set up port forward

# Copy files
c           # Copy file from pod (prompts for path)

# Show pod metrics
:pulses     # CPU/Memory graphs!
```

## Namespace Management

### Switch Namespaces Quickly
```bash
:ns         # List all namespaces
Enter       # Switch to namespace
/           # Filter namespaces

# Pro tip: Set default namespace
# While on namespace, press:
u           # Use as default namespace
```

### Cross-Namespace View
```bash
:po -A      # Show pods across ALL namespaces
:deploy -A  # Show all deployments
:svc -A     # Show all services

# Or press:
0           # Toggle "all namespaces" mode
```

## Real-World Workflows

### Debugging a Failing Pod
```bash
# 1. Find the pod
:po
/myapp      # Filter to your app

# 2. Check status (it's shown in the list)
# Look for: Pending, CrashLoopBackOff, Error

# 3. View logs
l           # Show logs
f           # Follow logs
/error      # Search for errors

# 4. Describe for events
d           # Describe pod
/Events     # Jump to events section

# 5. Shell in if running
s           # Open shell
# Debug interactively

# 6. Edit to fix
e           # Edit pod definition
```

### Monitor Deployment Rollout
```bash
# 1. Watch deployments
:deploy

# 2. See rollout status
Enter       # Drill into deployment
# Shows replicas, conditions

# 3. Watch pods being created
:po
/app-name   # Filter to your app
# Watch STATUS column change

# 4. Check logs of new pods
l           # View logs as they start
```

### Resource Investigation Flow
```bash
# Start broad, narrow down:
:node       # Check node health
:ns         # Pick namespace
:po         # See pods
Enter       # Drill into problematic pod
l           # Check logs
d           # Describe for events
s           # Shell in to investigate
```

## Advanced Features

### Resource Editing and Management
```bash
# Edit any resource
e           # Opens in $EDITOR (vim, nano, etc)

# Delete resources (careful!)
Ctrl+k      # Kill/delete resource
# Confirms before deleting

# Scale deployments
:deploy
s           # Scale deployment
# Enter new replica count
```

### Powerful Filtering
```bash
# Regex filtering
/^web-      # Pods starting with "web-"
/prod$      # Ending with "prod"
/api.*v2    # "api" followed by "v2"

# Inverse filtering
/!Running   # NOT running pods

# Column-specific filtering
# While viewing pods:
/2:Running  # Filter by STATUS column (column 2)
```

### Custom Views and Sorting
```bash
# Sort by different columns
:ctx        # Contexts view
Shift+c     # Sort by CPU
Shift+m     # Sort by Memory
Shift+n     # Sort by name
Shift+a     # Sort by age

# Toggle wide view
Ctrl+w      # Show more columns
```

### Bookmarks (Quick Navigation)
```bash
# Save current view as bookmark
Ctrl+b      # Bookmark current view

# List bookmarks
:bookmarks  # Show saved bookmarks

# Jump to bookmark
# Select and press Enter
```

## k9s Configuration

### Config File Location
```bash
# k9s config location:
~/.config/k9s/config.yml

# Edit preferences:
vim ~/.config/k9s/config.yml
```

### Useful Configuration Options
```yaml
# ~/.config/k9s/config.yml
k9s:
  refreshRate: 2    # Refresh every 2 seconds
  maxConnRetry: 5   
  enableMouse: true # Enable mouse support
  headless: false   
  logoless: false   # Show the cool logo
  crumbsless: false 
  readOnly: false   # Set true for production safety
  noExitOnCtrlC: true
  
  # Set default namespace
  namespace:
    active: default
    favorites:
    - default
    - kube-system
    - production
    
  # Custom shortcuts
  hotkey:
    # Make 'x' show pod metrics
    x:
      shortCut: Shift-M
      description: Show Metrics
```

### Skin/Theme Configuration
```yaml
# ~/.config/k9s/skin.yml
k9s:
  body:
    fgColor: "#ffffff"
    bgColor: "#000000"
    logoColor: "#00ff00"
  info:
    fgColor: "#00ff00"
    sectionColor: "#ffffff"
```

## Command Mode (Power User)

### Execute kubectl Commands
```bash
# Press : then type command
:kubectl get nodes -o wide
:kubectl apply -f manifest.yaml

# Shortcuts for common commands
:q          # Quit
:help       # Show help
:alias      # Show aliases
```

### Resource Aliases
```bash
# Built-in aliases
:po  = :pods
:svc = :services
:cm  = :configmaps
:sec = :secrets
:ing = :ingresses
:pvc = :persistentvolumeclaims
```

## Integration with Other Tools

### Export and Piping
```bash
# While viewing resources:
Ctrl+s      # Save to file

# From command line:
k9s --command "pods" | grep Running
```

### Working with kubectl
```bash
# k9s respects KUBECONFIG
export KUBECONFIG=/path/to/config
k9s

# Multiple contexts
k9s --context staging
k9s --context production
```

### Complement with Other Tools
```bash
# Use k9s for exploration, then:
# - kubectl for scripting
# - helm for deployments  
# - stern for multi-pod logs

# Example workflow:
# 1. Find pod pattern in k9s
# 2. Use stern for aggregate logs:
stern "web-.*" --since 1h
```

## Troubleshooting k9s

### Connection Issues
```bash
# Check kubeconfig
echo $KUBECONFIG
kubectl cluster-info

# Try explicit config
k9s --kubeconfig ~/.kube/config

# Debug mode
k9s --logLevel debug
```

### Performance Issues
```bash
# Increase refresh rate
# In k9s, press:
:config
# Edit refreshRate to higher value (5-10 seconds)

# Disable unnecessary views
k9s --headless  # No header
k9s --logoless  # No logo
```

### Display Issues
```bash
# Terminal compatibility
export TERM=xterm-256color
k9s

# Font issues - ensure terminal supports:
# - UTF-8
# - Unicode characters
# - 256 colors
```

## k9s vs kubectl Commands

### Common Operations Comparison
```bash
# View pods
kubectl get pods -n namespace --watch    # Before
:po                                      # k9s (then filter with /)

# View logs
kubectl logs -f pod-name -c container    # Before  
# k9s: navigate to pod, press 'l'

# Exec into pod
kubectl exec -it pod-name -- /bin/bash   # Before
# k9s: navigate to pod, press 's'

# Describe resource
kubectl describe pod pod-name            # Before
# k9s: navigate to pod, press 'd'

# Edit resource
kubectl edit deployment deploy-name      # Before
# k9s: navigate to deployment, press 'e'
```

## Best Practices

### For Development
1. **Use filters liberally** - `/` is your friend
2. **Learn the shortcuts** - especially `:po`, `l`, `s`, `d`
3. **Keep multiple k9s instances** - different namespaces/contexts
4. **Use bookmarks** - save common views

### For Production
1. **Always use --readonly** for production clusters
2. **Be careful with Ctrl+k** - it really deletes things
3. **Use filters to avoid mistakes** - narrow scope first
4. **Set up different skins** - visual distinction for prod

### For Learning
1. **Explore with Enter** - drill into everything
2. **Use describe (d)** - understand resource relationships
3. **Watch events** - `:events` shows cluster activity
4. **Try all resource types** - `:api-resources` lists everything

## Quick Reference Card

### Essential Shortcuts
```
# Navigation
:po         List pods
:svc        List services
:deploy     List deployments
Enter       Drill down
Esc         Go back
/           Filter
0           All namespaces

# Pod Actions
l           Logs
s           Shell
d           Describe
y           YAML
e           Edit
Ctrl+k      Delete

# View Control
Ctrl+w      Wide view
f           Follow (in logs)
w           Wrap (in logs)
Ctrl+s      Save to file

# System
:q          Quit
?           Help
:ctx        Contexts
:ns         Namespaces
```

## Tips and Tricks

### Speed Tips
1. **Type partial commands**: `:po` instead of `:pods`
2. **Use number keys**: In logs, press 1,2,3 for containers
3. **Chain filters**: `/Running` then `/web` narrows further
4. **Use regex**: `/web-[0-9]+` for pattern matching

### Power User Tips
1. **Multiple windows**: Run k9s in tmux/screen panes
2. **Aliases in shell**: `alias k="k9s"` for quick access
3. **Context switching**: `k9s --context` in different terminals
4. **Readonly prod**: `alias k9sprod="k9s --context prod --readonly"`

### Debugging Tips
1. **Start with events**: `:events` shows recent cluster activity
2. **Check node resources**: `:node` then `Enter` for details
3. **Follow the chain**: Pod → ReplicaSet → Deployment
4. **Use pulses**: `:pulses` for resource usage graphs

## Next Steps

1. **Install k9s** and launch it against your cluster
2. **Master basic navigation**: `:po`, Enter, Esc, `/`
3. **Learn pod actions**: logs (`l`), shell (`s`), describe (`d`)
4. **Explore other resources**: services, deployments, configmaps
5. **Customize**: Edit config for your workflow

**💡 Final Tip**: k9s transforms Kubernetes management from command memorization to intuitive navigation. Spend 30 minutes exploring, and you'll never want to go back to raw kubectl!

## Related Topics
- [[Kubernetes/Kubernetes Command Reference]] - kubectl commands
- [[Kubernetes/K3s Quick Start Guide]] - K3s setup
- [[Linux/Quick Reference/Command Line Tools/Development Tools/🖥️ tmux Terminal Multiplexer Beginners Bible 🖥️]] - Terminal multiplexing
- [[Kubernetes/Practical Guides/Troubleshooting Guide]] - Debugging Kubernetes