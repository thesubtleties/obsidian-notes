## Installation 📥
```bash
# Linux
sudo apt install tree   # Debian/Ubuntu
sudo yum install tree   # CentOS/RHEL
sudo pacman -S tree     # Arch

# macOS
brew install tree

# Windows
choco install tree      # Using Chocolatey
winget install tree     # Using winget
```

## Basic Usage
```bash
tree                    # Show entire directory structure
tree /path/to/dir       # Show structure of specific directory
```

## Control Depth 📏
```bash
tree -L 2               # Limit to 2 levels deep
```

## Filter Content 🔍
```bash
tree -d                 # Show directories only
tree -I "node_modules"  # Exclude node_modules
tree -I "node_modules|.git|dist"  # Exclude multiple patterns
tree -P "*.js"          # Show only .js files
```

## Formatting Options 💅
```bash
tree -h                 # Human-readable file sizes
tree -s                 # Show file sizes
tree -C                 # Add colors
tree -f                 # Show full paths
tree -D                 # Show last modified date
```

## Output Options 📄
```bash
tree > output.txt       # Save to file
tree -H "." -o out.html # Generate HTML output
```

## Common Combinations 🔥
```bash
# Project overview (no deps, 3 levels)
tree -L 3 -I "node_modules|.git"

# Directory sizes
tree -d -h --du -L 2

# Code files only with sizes
tree -P "*.js|*.css|*.html" -h
```

## Windows Built-in Tree
```
tree /F                 # Show files
tree /A                 # Use ASCII characters
```

---

