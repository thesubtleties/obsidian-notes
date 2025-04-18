## Installation 📥
```bash
# Usually pre-installed on Unix-like systems
# Linux
sudo apt install sed    # Debian/Ubuntu
sudo yum install sed    # CentOS/RHEL
sudo pacman -S sed      # Arch Linux

# macOS
brew install gnu-sed    # For GNU sed via Homebrew

# Windows
choco install sed       # Using Chocolatey
winget install sed      # Using winget
```

## Basic Usage
```bash
sed 'command' file      # Apply command to file
sed -e 'cmd1' -e 'cmd2' # Apply multiple commands
sed 's/old/new/' file   # Substitute first occurrence on each line
sed 's/old/new/g' file  # Substitute all occurrences
```

## Common Options 🔍
```bash
sed -i                  # Edit files in-place
sed -i.bak              # Edit in-place but create backup with .bak extension
sed -n                  # Suppress automatic printing
sed -E                  # Use extended regular expressions
sed -f script.sed       # Read commands from file
```

## Substitution Commands 🔄
```bash
s/pattern/replacement/  # Substitute pattern with replacement
s/pattern/replacement/g # Global substitution (all occurrences)
s/pattern/replacement/I # Case-insensitive substitution
s/pattern/replacement/p # Print lines with successful substitutions
```

## Address Selection 📍
```bash
sed '2s/old/new/'       # Apply to line 2 only
sed '2,5s/old/new/'     # Apply to lines 2-5
sed '/pattern/s/old/new/' # Apply to lines matching pattern
sed '2,/pattern/s/old/new/' # From line 2 to pattern match
```

## Common Examples 🚀
```bash
# Replace all occurrences of "foo" with "bar"
sed 's/foo/bar/g' file.txt

# Delete all blank lines
sed '/^$/d' file.txt

# Delete comments (lines starting with #)
sed '/^#/d' file.txt

# Add line numbers
sed = file.txt | sed 'N; s/\n/\t/'

# Insert text before/after a pattern
sed '/pattern/i\New text before' file.txt
sed '/pattern/a\New text after' file.txt
```

## Advanced Usage 💡
```bash
# Multiple commands with -e
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt

# Using hold space to swap lines
sed 'N;s/\(.*\)\n\(.*\)/\2\n\1/' file.txt

# Conditional execution
sed '/pattern/{s/old/new/g; s/this/that/g}' file.txt
```

## Troubleshooting 🔧
```bash
# Escaping special characters
sed 's/\/path\/to\/file/\/new\/path/' file.txt
# Alternative delimiter to avoid escaping
sed 's|/path/to/file|/new/path|' file.txt

# Debugging with -n and p
sed -n 's/pattern/replacement/p' file.txt
```

## Further Reading 📚
- Official Documentation: [GNU sed Manual](https://www.gnu.org/software/sed/manual/sed.html)
- Man Page: `man sed` in your terminal
- Additional Resources:
  - [Sed - An Introduction and Tutorial](https://www.grymoire.com/Unix/Sed.html)
  - [Sed One-Liners Explained](https://catonmat.net/sed-one-liners-explained-part-one)
  - [Sed & Awk 101 Hacks](http://www.thegeekstuff.com/sed-awk-101-hacks-ebook/)

---