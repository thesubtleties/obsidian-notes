## Installation 📥
```bash
# Linux
sudo apt install rsync     # Debian/Ubuntu
sudo yum install rsync     # CentOS/RHEL
sudo pacman -S rsync       # Arch Linux

# macOS
brew install rsync         # Using Homebrew

# Windows
choco install rsync        # Using Chocolatey
```

## Basic Usage
```bash
rsync [options] source destination
rsync [options] source [user@]host:destination    # Remote transfer (push)
rsync [options] [user@]host:source destination    # Remote transfer (pull)
```

## Common Options 🔍
```bash
rsync -a         # Archive mode (preserves permissions, timestamps, etc.)
rsync -v         # Verbose output
rsync -z         # Compress data during transfer
rsync -P         # Show progress and keep partially transferred files
rsync -n         # Dry run (simulation)
rsync --delete   # Delete files at destination that don't exist at source
```

## Advanced Options 💡
```bash
rsync -r         # Recursive (directories and subdirectories)
rsync -l         # Copy symlinks as symlinks
rsync -p         # Preserve permissions
rsync -t         # Preserve modification times
rsync -g         # Preserve group
rsync -o         # Preserve owner
rsync -h         # Human-readable output
rsync -e         # Specify remote shell (e.g., -e "ssh -p 2222")
rsync --exclude  # Exclude files/directories matching pattern
rsync --include  # Include files/directories matching pattern
```

## Common Examples 🚀
```bash
# Sync local directories (with trailing slash behavior)
rsync -av /source/ /destination/    # Contents of source to destination
rsync -av /source /destination/     # Source directory inside destination

# Remote transfers
rsync -avz ~/Documents user@remote:/backup/    # Push to remote
rsync -avz user@remote:/data/ ~/local_data/    # Pull from remote

# Backup with deletion and exclusions
rsync -avz --delete --exclude="*.tmp" /source/ /backup/

# Using SSH with non-standard port
rsync -avz -e "ssh -p 2222" /local/ user@remote:/remote/
```

## Bandwidth and Performance 🚀
```bash
rsync --bwlimit=1000    # Limit bandwidth to 1000 KB/s
rsync -avz --compress-level=9    # Maximum compression
rsync --partial         # Keep partially transferred files
rsync --append-verify   # Append data to shorter files, verify
```

## Troubleshooting 🔧
```bash
rsync -avvv             # Very verbose for debugging
rsync --info=progress2  # Detailed transfer progress
rsync --checksum        # Use checksums instead of mod-time & size
rsync --timeout=120     # Set I/O timeout in seconds
```

## Further Reading 📚
- Official Documentation: [rsync Website](https://rsync.samba.org/)
- Man Page: `man rsync` in your terminal
- Additional Resources:
  - [Rsync Examples - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories)
  - [Rsync Tutorial - Linux Journal](https://www.linuxjournal.com/content/using-rsync-backups)
  - [Rsync Algorithm Technical Report](https://rsync.samba.org/tech_report/)

---