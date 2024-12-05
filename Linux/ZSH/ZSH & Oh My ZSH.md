# ZSH & Oh My ZSH Cheat Sheet

## Installation & Setup
```bash
# Install ZSH
sudo apt install zsh    # Ubuntu/Debian
brew install zsh       # macOS

# Install Oh My ZSH
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Set ZSH as default shell
chsh -s $(which zsh)
```

## Configuration
```bash
# Main config file location
~/.zshrc

# Oh My ZSH directory
~/.oh-my-zsh

# Reload ZSH configuration
source ~/.zshrc
# or
exec zsh
```

## Theme Management
```bash
# Change theme (edit ~/.zshrc)
ZSH_THEME="theme-name"

# Popular themes
ZSH_THEME="robbyrussell"  # Default
ZSH_THEME="agnoster"      # Popular powerline theme
ZSH_THEME="powerlevel10k" # Highly customizable

# Random theme
ZSH_THEME="random"
```

## Plugin Management
```bash
# Add plugins (edit ~/.zshrc)
plugins=(
    git
    docker
    npm
    yarn
    node
    python
    vscode
)

# Custom plugin installation directory
~/.oh-my-zsh/custom/plugins/
```

## Directory Navigation
```bash
# Smart directory switching
cd -       # Go to previous directory
cd ~       # Go to home directory
cd /       # Go to root directory

# Directory stack
pushd dir  # Push directory to stack
popd       # Pop directory from stack
dirs -v    # List directory stack

# Quick directory jumping
d          # Show recently used directories
1          # Jump to first directory in stack
2          # Jump to second directory in stack
```

## Aliases and Functions
```bash
# List all aliases
alias

# Common default aliases
ll        # ls -l
la        # ls -la
l         # ls -lah
md        # mkdir -p
rd        # rmdir

# Create new alias (add to ~/.zshrc)
alias myalias="command"

# Create function (add to ~/.zshrc)
function myfunc() {
    echo "Hello $1"
}
```

## History Management
```bash
# History commands
history         # Show command history
history 0       # Show all history
history 1       # Show last command
!!              # Run last command
!$              # Last argument of previous command
!*              # All arguments of previous command
!string         # Run last command starting with 'string'
!?string        # Run last command containing 'string'

# Search history
Ctrl + R        # Reverse history search
```

## Globbing (File Pattern Matching)
```bash
# Extended globbing
*.txt           # All .txt files
**/*.txt        # Recursive search for .txt files
*(.)            # All regular files
*(/)            # All directories
*(/@)           # All symlinks
```

## Key Shortcuts
```bash
# Navigation
Ctrl + A        # Move to beginning of line
Ctrl + E        # Move to end of line
Alt + B         # Move backward one word
Alt + F         # Move forward one word

# Editing
Ctrl + U        # Clear line before cursor
Ctrl + K        # Clear line after cursor
Ctrl + W        # Delete word before cursor
Alt + D         # Delete word after cursor
Ctrl + Y        # Paste previously cut text
```

## Common Oh My ZSH Plugins
```bash
# git plugin commands
gst             # git status
ga              # git add
gcmsg           # git commit -m
gp              # git push
gl              # git pull

# docker plugin commands
dps             # docker ps
dpa             # docker ps -a
di              # docker images

# npm plugin commands
ni              # npm install
nig             # npm install -g
nr              # npm run
```

## Key Points & Tips
- Always backup `.zshrc` before making major changes
- Use `omz update` to update Oh My ZSH
- Custom scripts can be added to `~/.oh-my-zsh/custom/`
- Plugins can significantly slow down shell startup time
- Use `time zsh -i -c exit` to measure startup time
- Some themes require installing custom fonts
- Use `zsh_stats` to see most used commands

## Troubleshooting
```bash
# Reset ZSH configuration
mv ~/.zshrc ~/.zshrc.backup
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc

# Check ZSH version
zsh --version

# Debug mode
zsh -x
```