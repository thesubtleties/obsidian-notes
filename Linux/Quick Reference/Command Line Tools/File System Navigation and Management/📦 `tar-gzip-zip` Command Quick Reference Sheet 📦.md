
## Installation 📥
```bash
# Linux
sudo apt install tar gzip zip unzip    # Debian/Ubuntu
sudo yum install tar gzip zip unzip    # RHEL/CentOS
sudo pacman -S tar gzip zip unzip      # Arch Linux

# macOS (usually pre-installed)
brew install gnu-tar gzip zip          # Using Homebrew

# Windows
choco install 7zip                     # Using Chocolatey
winget install 7zip.7zip               # Using winget
```

## Basic Usage
```bash
# tar - create/extract archives
tar -cf archive.tar files/             # Create tar archive
tar -xf archive.tar                    # Extract tar archive

# gzip - compress files
gzip file.txt                          # Compress to file.txt.gz
gunzip file.txt.gz                     # Decompress file.txt.gz

# zip - create/extract zip archives
zip archive.zip files/                 # Create zip archive
unzip archive.zip                      # Extract zip archive
```

## Common Options 🔍
```bash
# tar options
tar -c                                 # Create archive
tar -x                                 # Extract archive
tar -f                                 # Specify filename
tar -v                                 # Verbose output
tar -z                                 # Use gzip compression
tar -j                                 # Use bzip2 compression
tar -t                                 # List contents

# gzip options
gzip -d                                # Decompress (same as gunzip)
gzip -k                                # Keep original file
gzip -1 to -9                          # Compression level (1=fast, 9=best)

# zip options
zip -r                                 # Recursive (include directories)
zip -e                                 # Encrypt with password
zip -9                                 # Best compression
```

## Common Examples 🚀
```bash
# Create compressed tar archives
tar -czf archive.tar.gz directory/     # Create tar.gz (gzip)
tar -cjf archive.tar.bz2 directory/    # Create tar.bz2 (bzip2)
tar -cJf archive.tar.xz directory/     # Create tar.xz (xz)

# Extract compressed tar archives
tar -xzf archive.tar.gz                # Extract tar.gz
tar -xjf archive.tar.bz2               # Extract tar.bz2
tar -xJf archive.tar.xz                # Extract tar.xz

# List contents of archives
tar -tf archive.tar                    # List tar contents
tar -tzf archive.tar.gz                # List tar.gz contents
unzip -l archive.zip                   # List zip contents

# Create password-protected zip
zip -er archive.zip directory/         # Create encrypted zip

# Compress multiple files
gzip file1 file2 file3                 # Create separate .gz files
zip archive.zip file1 file2 file3      # Add to single zip
```

## Advanced Usage 💡
```bash
# Exclude files/patterns
tar -czf archive.tar.gz dir/ --exclude="*.log"

# Update existing archives
zip -u archive.zip newfile.txt

# Split large archives
zip -s 1g archive.zip directory/       # Split into 1GB chunks

# Test archive integrity
gzip -t file.gz
unzip -t archive.zip

# Preserve permissions and links
tar -czpf archive.tar.gz directory/    # Preserve permissions
```

## Troubleshooting 🔧
```bash
# Fix corrupted zip files
zip -FF damaged.zip --out=fixed.zip

# Verbose extraction for debugging
tar -xzvf archive.tar.gz
unzip -v archive.zip

# Force overwrite during extraction
unzip -o archive.zip
```

## Further Reading 📚
- Man Pages: `man tar`, `man gzip`, `man zip`, `man unzip`
- GNU Tar Documentation: [GNU Tar Manual](https://www.gnu.org/software/tar/manual/)
- Gzip Documentation: [GNU Gzip Manual](https://www.gnu.org/software/gzip/manual/)
- Info Zip Documentation: [Info-ZIP](http://www.info-zip.org/mans/zip.html)
- Additional Resources:
  - [Linux Archive Management Tutorial](https://www.digitalocean.com/community/tutorials/how-to-compress-and-extract-files-using-the-tar-command-on-linux)
  - [7-Zip Documentation](https://www.7-zip.org/docs.html) (Windows alternative)

---