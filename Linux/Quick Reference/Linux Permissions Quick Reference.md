## Basic Structure
```
drwxrwxrwx  1 owner group  4096  Jan 1 12:00 filename
│└┬┘└┬┘└┬┘  │  │     │     │    └── Date
│ │  │  │   │  │     │     └──── Size
│ │  │  │   │  │     └────────── Group
│ │  │  │   │  └─────────────── Owner
│ │  │  │   └────────────────── Number of links
│ │  │  └─── Permissions for others
│ │  └────── Permissions for group
│ └───────── Permissions for owner
└─────────── File type (d=directory, -=file, l=link)
```

## Permission Types
```
r (read)    = 4
w (write)   = 2
x (execute) = 1
- (none)    = 0
```

## Common Permission Combinations
```
777 (rwxrwxrwx) = Full permissions for everyone (dangerous!)
755 (rwxr-xr-x) = Owner can do anything, others can read/execute
644 (rw-r--r--) = Owner can read/write, others can read
700 (rwx------) = Owner can do anything, others nothing
600 (rw-------) = Owner can read/write, others nothing
666 (rw-rw-rw-) = Everyone can read/write (dangerous!)
```

## Checking Permissions
```bash
# List files with permissions
ls -l filename
ls -la  # Include hidden files

# See numeric permissions
stat -c "%a %n" filename
```

## Changing Permissions

### Using chmod with Numbers
```bash
# Format: chmod ### filename
chmod 755 file.txt  # rwxr-xr-x
chmod 644 file.txt  # rw-r--r--
chmod -R 755 directory/  # Recursive for directories
```

### Using chmod with Letters
```bash
# u (user/owner), g (group), o (others), a (all)
# + (add), - (remove), = (set exactly)

chmod u+x file.txt    # Add execute for owner
chmod g-w file.txt    # Remove write for group
chmod o-rwx file.txt  # Remove all for others
chmod a+r file.txt    # Add read for everyone
```

## Changing Ownership
```bash
# Change owner
chown user filename
chown user:group filename  # Change both owner and group
chown -R user directory/  # Recursive

# Change group only
chgrp group filename
chgrp -R group directory/  # Recursive
```

## Special Permissions
```
4000 (SUID) = Run as owner
2000 (SGID) = Run as group
1000 (Sticky) = Only owner can delete
```

## Common Use Cases
```bash
# Web server files
chmod 755 /var/www/html/
chmod 644 /var/www/html/*.html

# Configuration files
chmod 600 config.ini

# Shell scripts
chmod 755 script.sh

# Private SSH keys
chmod 600 ~/.ssh/id_rsa
```

## Safety Tips
- Never use 777 unless absolutely necessary
- Avoid 666 for security reasons
- Most files should be 644
- Most directories should be 755
- Sensitive files should be 600
- Scripts should be 755

## Quick Reference for Numbers
```
0 = ---
1 = --x
2 = -w-
3 = -wx
4 = r--
5 = r-x
6 = rw-
7 = rwx
```

## Troubleshooting Commands
```bash
# Find files with specific permissions
find . -type f -perm 777

# Find files owned by user
find . -user username

# Fix common permission issues
find . -type f -exec chmod 644 {} \;
find . -type d -exec chmod 755 {} \;
```

## Docker-specific
```bash
# Fix Docker-created files
sudo chown -R $USER:$USER directory/
sudo chmod -R u+rw directory/
```

Remember: The principle of least privilege suggests using the most restrictive permissions necessary for functionality.