## Installation 📥
```bash
# Built into all Unix/Linux systems and macOS
# Windows users can use ls via WSL or Git Bash
```

## Basic Usage
```bash
ls                      # List contents of current directory
ls /path/to/dir         # List contents of specific directory
```

## Common Options 🔍
```bash
ls -l                   # Long format (permissions, owner, size, date)
ls -a                   # Show all files (including hidden)
ls -h                   # Human-readable file sizes (KB, MB, GB)
ls -t                   # Sort by modification time (newest first)
ls -r                   # Reverse sort order
ls -S                   # Sort by file size (largest first)
```

## Combining Options
```bash
ls -la                  # Long format + show all files
ls -lh                  # Long format + human-readable sizes
ls -ltr                 # Long format + sort by time + reverse (oldest first)
ls -lSh                 # Long format + sort by size + human-readable
```

## Display Options 💅
```bash
ls -F                   # Add indicators (/ for dirs, * for executables)
ls --color=auto         # Colorize output by file type
ls -d */                # List only directories
ls -1                   # One file per line
```

## Advanced Filtering 🔎
```bash
ls *.txt                # List only .txt files
ls -l | grep "^d"       # List only directories (with pipe to grep)
ls -l --block-size=M    # Show sizes in MB
ls -R                   # Recursive listing (all subdirectories)
```

## Output Redirection 📄
```bash
ls -la > files.txt      # Save listing to a file
ls -la | less           # Pipe to less for scrollable output
```

## Useful Aliases to Add to Your .bashrc/.zshrc
```bash
alias ll='ls -la'       # Detailed list including hidden files
alias lt='ls -ltr'      # List by time (oldest first)
alias lh='ls -lSh'      # List by size with human-readable sizes
alias ldir='ls -la | grep "^d"'  # List only directories
```

## Understanding ls -l Output
```
drwxr-xr-x  2 user group  4096 Jan 1 10:00 directory
-rw-r--r--  1 user group  1024 Jan 2 15:30 file.txt
|[-][-][-]  | |    |      |    |           |
| |  |  |   | |    |      |    |           └─ Name
| |  |  |   | |    |      |    └─ Modification time
| |  |  |   | |    |      └─ Size
| |  |  |   | |    └─ Group
| |  |  |   | └─ Owner
| |  |  |   └─ Number of links
| |  |  └─ Other permissions (r=read, w=write, x=execute)
| |  └─ Group permissions
| └─ Owner permissions
└─ File type (d=directory, -=file, l=link)
```

---

Print this sheet for easy reference! 📋