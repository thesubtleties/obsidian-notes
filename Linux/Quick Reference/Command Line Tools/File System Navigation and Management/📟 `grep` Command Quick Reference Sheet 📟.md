## Installation 📥
```bash
# Usually pre-installed on Linux/macOS
# For Ubuntu/Debian if needed:
sudo apt install grep

# For macOS if needed:
brew install grep
```

## Basic Usage
```bash
grep pattern file        # Search for pattern in file
grep "text string" file  # Search for text string in file
grep pattern file1 file2 # Search in multiple files
cat file | grep pattern  # Search in piped input
```

## Common Options 🔍
```bash
grep -i pattern file     # Case-insensitive search
grep -v pattern file     # Invert match (show non-matching lines)
grep -w pattern file     # Match whole words only
grep -n pattern file     # Show line numbers
grep -c pattern file     # Count matching lines only
grep -l pattern files    # Show only filenames with matches
grep -r pattern dir      # Recursive search in directory
grep -E pattern file     # Extended regex (same as egrep)
```

## Combining Options
```bash
grep -in pattern file    # Case-insensitive with line numbers
grep -rl pattern dir     # Recursive search showing only filenames
grep -rc pattern dir     # Count matches in each file recursively
```

## Advanced Usage 💡
```bash
grep -A3 pattern file    # Show 3 lines after match
grep -B2 pattern file    # Show 2 lines before match
grep -C1 pattern file    # Show 1 line before and after match
grep --color=auto pattern # Highlight matching text
grep -o pattern file     # Show only the matching part
grep -P pattern file     # Perl-compatible regex (PCRE)
```

## Common Examples 🚀
```bash
# Find all files containing "error" in logs directory
grep -r "error" /var/log/

# Count occurrences of "404" in access log
grep -c "404" access.log

# Find all Python files containing "import requests"
grep -l "import requests" *.py

# Search for IP addresses
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" log.txt

# Find lines not containing "success"
grep -v "success" results.log

# Search for whole word "function" in JavaScript files
grep -rw "function" --include="*.js" .
```

## Troubleshooting 🔧
```bash
# If getting "binary file matches" message
grep -a pattern binary_file

# If pattern contains special characters
grep -F "pattern with * and ?" file

# For large files, increase performance
grep -m 10 pattern large_file  # Stop after 10 matches

# Suppress error messages
grep pattern file 2>/dev/null
```

---

