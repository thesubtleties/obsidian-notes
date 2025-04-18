## Installation 📥
```bash
# Usually pre-installed on Unix-like systems
# For macOS
brew install bash       # Using Homebrew

# For Windows
choco install git bash  # Using Chocolatey (Git Bash)
winget install git.git  # Using winget (includes Git Bash)
```

## Basic Usage
```bash
bash                    # Start a new bash shell
bash script.sh          # Execute a bash script
bash -c "command"       # Execute command in a new bash shell
```

## Common Options 🔍
```bash
bash -x script.sh       # Print commands as they execute (debug mode)
bash -v script.sh       # Print shell input lines as they're read
bash -n script.sh       # Check syntax without executing
bash -l                 # Act as a login shell
bash -i                 # Interactive shell
```

## Variables & Environment 🔧
```bash
VAR="value"             # Set a variable
echo $VAR               # Access a variable
export VAR="value"      # Export to child processes
readonly VAR="value"    # Create a constant
$HOME, $PATH, $USER     # Common environment variables
```

## Control Structures 🔄
```bash
# Conditionals
if [ condition ]; then
    commands
elif [ condition ]; then
    commands
else
    commands
fi

# Loops
for i in {1..5}; do
    echo $i
done

while [ condition ]; do
    commands
done

until [ condition ]; do
    commands
done

# Case statement
case $VAR in
    pattern1) commands ;;
    pattern2) commands ;;
    *) default commands ;;
esac
```

## Functions 📋
```bash
# Define a function
function_name() {
    commands
    return value
}

# Call a function
function_name arg1 arg2

# Access arguments
$1, $2      # First and second arguments
$@          # All arguments as separate strings
$#          # Number of arguments
```

## Common Examples 🚀
```bash
# Read user input
read -p "Enter your name: " name

# Process files in a directory
for file in *.txt; do
    echo "Processing $file"
done

# Check if a file exists
if [ -f "filename" ]; then
    echo "File exists"
fi

# Simple backup script
backup() {
    cp "$1" "$1.bak"
    echo "Backed up $1"
}
```

## Special Characters 💡
```bash
$?          # Exit status of last command
$$          # PID of current shell
$0          # Name of the script
&           # Run command in background
&&          # Logical AND
||          # Logical OR
>           # Redirect output
>>          # Append output
<           # Redirect input
|           # Pipe output to another command
```

## Further Reading 📚
- Official Documentation: [GNU Bash Manual](https://www.gnu.org/software/bash/manual/)
- Man Page: `man bash` in your terminal
- Additional Resources:
  - [Bash Guide for Beginners](https://tldp.org/LDP/Bash-Beginners-Guide/html/)
  - [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
  - [Bash Hackers Wiki](https://wiki.bash-hackers.org/)
  - [ShellCheck](https://www.shellcheck.net/) - Online shell script analyzer

---