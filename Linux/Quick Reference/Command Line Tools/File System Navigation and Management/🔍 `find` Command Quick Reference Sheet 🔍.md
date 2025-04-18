## Installation 📥
```bash
# Built into all Unix/Linux systems and macOS
# Windows users can use find via WSL or Git Bash
```

## Basic Usage
```bash
find . -name "filename"  # Find file named "filename" in current directory & subdirs
find /path -name "*.txt" # Find all .txt files in specified path
```

## Search Locations
```bash
find .                   # Current directory and subdirectories
find /home/user          # Specific directory
find /                   # Entire filesystem (use carefully!)
find . /usr/local        # Multiple locations
```

## Name and Pattern Matching 🔤
```bash
find . -name "*.jpg"     # Find by exact name (case-sensitive)
find . -iname "*.JPG"    # Find by name (case-insensitive)
find . -path "*/test/*"  # Match on path pattern
find . -regex ".*\.txt$" # Use regular expression
```

## Time-Based Search ⏱️
```bash
find . -mtime -7         # Modified less than 7 days ago
find . -mtime +30        # Modified more than 30 days ago
find . -mmin -60         # Modified less than 60 minutes ago
find . -newer file.txt   # Modified more recently than file.txt
```

## Size-Based Search 📏
```bash
find . -size +10M        # Files larger than 10 MB
find . -size -1k         # Files smaller than 1 KB
find . -empty            # Empty files and directories
```

## Type Filtering 📂
```bash
find . -type f           # Files only
find . -type d           # Directories only
find . -type l           # Symbolic links only
```

## Permission/Ownership 🔐
```bash
find . -perm 644         # Files with exact permissions
find . -perm -u+x        # Files that are executable by owner
find . -user username    # Files owned by specific user
find . -group groupname  # Files owned by specific group
```

## Logical Operators
```bash
find . -name "*.jpg" -o -name "*.png"  # OR: jpg OR png files
find . -name "*.txt" -a -size +1M      # AND: txt files AND larger than 1MB
find . -not -name "*.log"              # NOT: all except log files
find . \( -name "*.jpg" -o -name "*.png" \) -a -size +1M  # Complex: jpg OR png AND larger than 1MB
```

## Actions ⚡
```bash
find . -name "*.tmp" -delete           # Delete matching files
find . -name "*.txt" -print            # Print matches (default action)
find . -name "*.log" -exec rm {} \;    # Execute command on each file
find . -name "*.jpg" -exec chmod 644 {} \;  # Change permissions
find . -type f -exec grep "text" {} \; # Search within found files
```

## Optimization 🚀
```bash
find . -name "*.txt" -maxdepth 2       # Limit search depth
find . -name "*.log" -not -path "*/\.*" # Skip hidden directories
find . -type f -name "*.jpg" -print0 | xargs -0 -I{} cp {} /backup/  # Safely handle filenames with spaces
```

## Common Examples 💡
```bash
# Find and delete files older than 7 days
find /tmp -type f -mtime +7 -delete

# Find large files (>100MB)
find /home -type f -size +100M

# Find and compress log files
find /var/log -name "*.log" -exec gzip {} \;

# Find files modified in the last hour
find . -type f -mmin -60

# Find empty directories
find . -type d -empty
```

## Troubleshooting 🔧
```bash
# Handle "Permission denied" errors
find / -name "file" 2>/dev/null

# Increase verbosity
find . -name "*.txt" -print -ls
```

---

