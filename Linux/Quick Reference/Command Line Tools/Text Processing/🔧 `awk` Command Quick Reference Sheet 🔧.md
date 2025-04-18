## Installation 📥
```bash
# Linux
sudo apt install gawk       # Debian/Ubuntu
sudo yum install gawk       # CentOS/RHEL
sudo pacman -S gawk         # Arch Linux

# macOS
brew install gawk           # Using Homebrew

# Windows
choco install gawk          # Using Chocolatey
winget install gawk         # Using winget
```

## Basic Usage
```bash
awk 'pattern {action}' file
awk '{print}' file.txt      # Print all lines
awk '/pattern/' file.txt    # Print lines matching pattern
awk -F: '{print $1}' file   # Use : as field separator and print first field
```

## Common Options 🔍
```bash
awk -F 'delimiter'          # Set field separator
awk -v var=value            # Set a variable
awk -f script.awk           # Use awk script from file
awk -W version              # Display version information
```

## Field Processing 📊
```bash
awk '{print $1}'            # Print first field
awk '{print $1, $3}'        # Print first and third fields
awk '{print $NF}'           # Print last field
awk '{print NF}'            # Print number of fields
awk '{print NR, $0}'        # Print line number and entire line
```

## Pattern Matching 🔍
```bash
awk '/regex/' file          # Lines matching regex
awk '!/regex/' file         # Lines NOT matching regex
awk '$1 == "value"' file    # First field equals "value"
awk 'NR > 10' file          # Lines after line 10
awk 'NR==1, NR==5' file     # Lines 1 through 5
```

## Common Examples 🚀
```bash
# Sum the values in column 3
awk '{sum += $3} END {print sum}' file.txt

# Calculate average of column 2
awk '{sum += $2; count++} END {print sum/count}' file.txt

# Print lines longer than 80 characters
awk 'length($0) > 80' file.txt

# Print CSV as formatted table
awk -F, '{printf "%-10s %-10s %s\n", $1, $2, $3}' file.csv

# Count occurrences of a pattern
awk '/pattern/ {count++} END {print count}' file.txt
```

## Built-in Variables 🧰
```bash
$0      # Entire line
$1-$n   # Individual fields
NF      # Number of fields in current line
NR      # Current line number
FS      # Field separator (default: whitespace)
OFS     # Output field separator (default: space)
RS      # Record separator (default: newline)
ORS     # Output record separator (default: newline)
```

## Control Structures 🔄
```bash
# If-else statement
awk '{if ($1 > 10) print "Big"; else print "Small"}' file

# For loop
awk '{for (i=1; i<=NF; i++) print $i}' file

# While loop
awk '{i=1; while (i<=NF) {print $i; i++}}' file
```

## Further Reading 📚
- Official Documentation: [GNU Awk User's Guide](https://www.gnu.org/software/gawk/manual/)
- Man Page: `man awk` in your terminal
- Additional Resources:
  - [AWK - A Tutorial and Introduction](https://www.grymoire.com/Unix/Awk.html)
  - [Awk One-Liners Explained](https://catonmat.net/awk-one-liners-explained-part-one)
  - [The AWK Programming Language](https://ia803404.us.archive.org/13/items/pdfy-MgN0H1joIoDVoIC7/The_AWK_Programming_Language.pdf) (book)

---