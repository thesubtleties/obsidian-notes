# 🖥️ tmux Terminal Multiplexer Beginners Bible 🖥️

## What is tmux and Why Should You Care?

**tmux** (Terminal MUltipleXer) is like having multiple terminal windows inside a single terminal. Imagine being able to:
- **Split your terminal** into multiple panes (like VS Code but for the terminal)
- **Keep programs running** even after you disconnect from SSH
- **Switch between** multiple terminal sessions instantly
- **Save your layout** and restore it later

**💡 Mentor Tip**: If you've ever lost work because your SSH connection dropped, or wished you could see multiple terminals at once without alt-tabbing, tmux is your solution.

## Installation

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install tmux

# macOS (with Homebrew)
brew install tmux

# CentOS/RHEL/Fedora
sudo yum install tmux

# Check version
tmux -V
```

## Core Concepts (The Mental Model)

### Hierarchy
```
Server (runs in background)
  └── Sessions (workspaces)
        └── Windows (tabs)
              └── Panes (splits)
```

Think of it like:
- **Server**: The tmux program itself
- **Sessions**: Different projects (e.g., "web-app", "api-server", "monitoring")
- **Windows**: Different aspects of a project (e.g., "code", "logs", "database")
- **Panes**: Split views within a window (e.g., editor + terminal)

## Getting Started (5 Minutes to Productivity)

### Your First tmux Session
```bash
# Start tmux
tmux

# You're now inside tmux!
# Notice the green bar at the bottom - that's your status line
```

### Essential Key Concept: The Prefix Key
**EVERYTHING in tmux starts with the prefix key: `Ctrl+b`**

Think of it as telling tmux "Hey, the next key is for you, not the terminal"

```bash
# The pattern is always:
# 1. Press Ctrl+b
# 2. Release
# 3. Press the command key
```

### Most Important Commands (Learn These First)

#### Session Management
```bash
# From outside tmux:
tmux new -s myproject    # New session named "myproject"
tmux ls                  # List sessions
tmux attach -t myproject # Attach to session
tmux kill-session -t myproject # Kill session

# From inside tmux:
Ctrl+b d                 # Detach from session (it keeps running!)
Ctrl+b $                 # Rename current session
```

#### Window Management (Think Browser Tabs)
```bash
Ctrl+b c    # Create new window
Ctrl+b ,    # Rename current window
Ctrl+b n    # Next window
Ctrl+b p    # Previous window
Ctrl+b 0-9  # Switch to window by number
Ctrl+b w    # List all windows (interactive)
Ctrl+b &    # Kill current window (with confirmation)
```

#### Pane Management (The Game Changer)
```bash
Ctrl+b %    # Split vertically (side by side)
Ctrl+b "    # Split horizontally (top/bottom)
Ctrl+b o    # Switch between panes
Ctrl+b ;    # Toggle between last two panes
Ctrl+b x    # Kill current pane
Ctrl+b z    # Toggle pane zoom (fullscreen)

# Navigate panes with arrow keys
Ctrl+b ↑/↓/←/→    # Move between panes
```

## Real-World Workflows

### Development Setup
```bash
# Create a development session
tmux new -s dev

# Split into editor and terminal
Ctrl+b %           # Vertical split

# In left pane: open your editor
vim app.py         # or your preferred editor

# Switch to right pane
Ctrl+b →

# Split right pane for logs
Ctrl+b "           # Horizontal split

# Top right: run your app
python app.py

# Bottom right: tail logs
Ctrl+b ↓
tail -f app.log

# Name this window
Ctrl+b , 
# Type: main-app
```

### SSH Session Management
```bash
# On your local machine
tmux new -s remote-work

# SSH to server
ssh user@server

# Do your work...
# Connection drops? No problem!

# Reconnect later
ssh user@server
tmux attach -t remote-work
# Everything is still running!
```

### Multiple Project Management
```bash
# Project 1
tmux new -s project1
# ... set up your workspace ...
Ctrl+b d    # Detach

# Project 2
tmux new -s project2
# ... set up different workspace ...
Ctrl+b d    # Detach

# Switch between projects instantly
tmux attach -t project1    # Work on project 1
Ctrl+b d
tmux attach -t project2    # Switch to project 2
```

## Advanced Pane Management

### Resize Panes
```bash
# Hold Ctrl+b, then use arrow keys
Ctrl+b (hold) + ↑/↓/←/→    # Resize in steps

# Precise resizing
Ctrl+b : 
# Then type:
resize-pane -U 10    # Up 10 lines
resize-pane -D 10    # Down 10 lines
resize-pane -L 10    # Left 10 columns
resize-pane -R 10    # Right 10 columns
```

### Pane Layouts
```bash
Ctrl+b Space    # Cycle through layouts
Ctrl+b Alt+1    # Even horizontal
Ctrl+b Alt+2    # Even vertical
Ctrl+b Alt+3    # Main horizontal
Ctrl+b Alt+4    # Main vertical
Ctrl+b Alt+5    # Tiled
```

### Moving Panes
```bash
Ctrl+b {    # Move pane left
Ctrl+b }    # Move pane right
Ctrl+b !    # Break pane into new window
```

## Copy Mode (tmux's Killer Feature)

### Enable Mouse Support First
```bash
# In tmux, run:
Ctrl+b :
# Type:
set -g mouse on

# Now you can:
# - Click to switch panes
# - Scroll with mouse wheel
# - Select text to copy
```

### Keyboard Copy Mode
```bash
Ctrl+b [        # Enter copy mode
# Now use vim-like navigation:
# h,j,k,l or arrow keys to move
# / to search forward
# ? to search backward
# Space to start selection
# Enter to copy selection
Ctrl+b ]        # Paste

# In copy mode:
g               # Go to top
G               # Go to bottom
0               # Start of line
$               # End of line
w               # Next word
b               # Previous word
```

## Configuration (~/.tmux.conf)

### Starter Configuration
```bash
# Create config file
cat > ~/.tmux.conf << 'EOF'
# Enable mouse support
set -g mouse on

# Start windows and panes at 1, not 0
set -g base-index 1
setw -g pane-base-index 1

# Renumber windows when one is closed
set -g renumber-windows on

# Increase history limit
set -g history-limit 10000

# Use vim keys in copy mode
setw -g mode-keys vi

# Easier splitting
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Quick pane selection
bind -r h select-pane -L
bind -r j select-pane -D
bind -r k select-pane -U
bind -r l select-pane -R

# Reload config
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# Better colors
set -g default-terminal "screen-256color"

# Status bar customization
set -g status-bg black
set -g status-fg white
set -g status-left '#[fg=green]#S '
set -g status-right '#[fg=yellow]#(whoami)@#H #[fg=blue]%H:%M '
EOF

# Reload configuration
tmux source-file ~/.tmux.conf
```

### Power User Settings
```bash
# Add to ~/.tmux.conf

# Change prefix from Ctrl+b to Ctrl+a (easier to reach)
unbind C-b
set-option -g prefix C-a
bind-key C-a send-prefix

# Quick window switching
bind -n M-Left previous-window
bind -n M-Right next-window

# Synchronize panes (type in all panes at once)
bind y setw synchronize-panes

# Save and restore sessions
# Install plugin manager first:
# git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# Auto restore
set -g @continuum-restore 'on'

# Initialize plugin manager (keep at bottom)
run '~/.tmux/plugins/tpm/tpm'
```

## Common Workflows and Patterns

### Web Development Setup
```bash
#!/bin/bash
# save as ~/setup-webdev.sh

tmux new-session -d -s webdev -n editor
tmux send-keys -t webdev:editor 'cd ~/myproject' C-m
tmux send-keys -t webdev:editor 'vim .' C-m

tmux split-window -h -t webdev:editor
tmux send-keys -t webdev:editor.1 'npm run dev' C-m

tmux new-window -t webdev -n servers
tmux send-keys -t webdev:servers 'cd ~/myproject/backend' C-m
tmux send-keys -t webdev:servers 'npm start' C-m

tmux split-window -v -t webdev:servers
tmux send-keys -t webdev:servers.1 'cd ~/myproject' C-m
tmux send-keys -t webdev:servers.1 'npm run test:watch' C-m

tmux new-window -t webdev -n logs
tmux send-keys -t webdev:logs 'tail -f ~/myproject/logs/app.log' C-m

tmux attach-session -t webdev
```

### System Monitoring Dashboard
```bash
#!/bin/bash
# save as ~/monitor.sh

tmux new-session -d -s monitor -n system

# Top left: htop
tmux send-keys -t monitor:system 'htop' C-m

# Top right: disk usage
tmux split-window -h -t monitor:system
tmux send-keys -t monitor:system.1 'watch -n 5 df -h' C-m

# Bottom left: network
tmux split-window -v -t monitor:system.0
tmux send-keys -t monitor:system.2 'sudo iftop' C-m

# Bottom right: logs
tmux split-window -v -t monitor:system.1
tmux send-keys -t monitor:system.3 'journalctl -f' C-m

tmux attach-session -t monitor
```

### Remote Server Management
```bash
# On local machine
tmux new -s servers

# Window for each server
tmux rename-window web1
ssh user@web1.example.com

Ctrl+b c    # New window
tmux rename-window web2
ssh user@web2.example.com

Ctrl+b c    # New window
tmux rename-window db1
ssh user@db1.example.com

# Now easily switch between servers with Ctrl+b [0-2]
```

## Troubleshooting

### Common Issues

#### "Sessions should be nested with care"
```bash
# You're trying to run tmux inside tmux
# Either:
Ctrl+b d    # Detach from current session first
# Or use:
unset TMUX  # Temporarily allow nesting
```

#### Can't Type After Ctrl+b
```bash
# You're in command mode
# Press Esc to get back to normal mode
```

#### Lost Sessions After Reboot
```bash
# tmux sessions don't persist across reboots by default
# Use tmux-resurrect plugin (see configuration section)
# Or manually save session layout:
Ctrl+b :
# Type:
capture-pane -S - ; save-buffer ~/tmux-session.txt
```

#### Colors Look Wrong
```bash
# Add to ~/.tmux.conf:
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# Reload:
tmux source-file ~/.tmux.conf
```

### Emergency Commands
```bash
# Kill unresponsive pane
Ctrl+b x    # Then y to confirm

# Kill entire session (from outside)
tmux kill-session -t session-name

# Kill tmux server (nuclear option)
tmux kill-server

# List what's using tmux socket
lsof | grep tmux
```

## tmux vs Screen vs Native Terminal

### When to Use tmux
- **Remote work**: Keep sessions alive through disconnections
- **Complex layouts**: Multiple panes and windows
- **Project organization**: Separate sessions per project
- **Pair programming**: Share sessions with others
- **Long-running tasks**: Keep processes running in background

### When NOT to Use tmux
- **Simple commands**: Quick one-off tasks
- **GUI applications**: tmux is terminal-only
- **Performance critical**: Slight overhead (usually negligible)
- **Learning curve**: Don't force it on beginners

## Quick Reference Card

### Session Commands
```
tmux new -s name     # New named session
tmux ls              # List sessions
tmux attach -t name  # Attach to session
tmux detach          # Detach (or Ctrl+b d)
```

### Key Bindings (After Ctrl+b)
```
## Sessions
d    Detach
$    Rename session
s    List sessions

## Windows
c    Create window
n    Next window
p    Previous window
0-9  Go to window #
,    Rename window
&    Kill window
w    List windows

## Panes
%    Split vertical
"    Split horizontal
o    Next pane
;    Last pane
x    Kill pane
z    Zoom pane
{/}  Move pane
!    Break to window
Space Cycle layouts

## Copy Mode
[    Enter copy mode
]    Paste buffer

## Misc
?    List key bindings
:    Command prompt
```

## Next Steps

1. **Start simple**: Use tmux for one project this week
2. **Learn incrementally**: Master sessions first, then windows, then panes
3. **Customize gradually**: Add to ~/.tmux.conf as you find pain points
4. **Practice the prefix**: Ctrl+b will become muscle memory

**💡 Final Tip**: The best way to learn tmux is to use it. Start with `tmux new -s learn` and practice the commands. Within a week, you'll wonder how you lived without it!

## Related Topics
- [[Linux/Quick Reference/Linux Commands Quick Reference]] - General Linux commands
- [[VSCode/VSCode Overview - Tips and Tricks]] - IDE alternative
- [[Git-Github/Git Commands Overview]] - Version control in terminal
- [[Linux/ZSH/What is ZSH? An Introduction to the Z Shell]] - Terminal enhancement