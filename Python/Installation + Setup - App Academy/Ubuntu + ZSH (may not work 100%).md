# Complete Python 3.9.4 Development Setup Guide (WSL Ubuntu + Zsh)

## 1. Initial System Setup
```bash
# Update system and install required dependencies
sudo apt update && sudo apt upgrade -y

# Install build dependencies
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
> These packages are required for building Python from source and installing various Python packages.

## 2. Install and Configure pyenv
```bash
# Install pyenv
curl https://pyenv.run | bash

# Add to ~/.zshrc
cat >> ~/.zshrc << 'EOL'
# Debug Zsh startup
echo "Starting Zsh initialization at $(date)" >> ~/.zsh_debug_log

# Path configurations
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"

# pyenv initialization
eval "$(pyenv init -)"
eval "$(pyenv init --path)"
EOL

# Reload shell configuration
exec "$SHELL"
```
> Sets up pyenv and configures Zsh to use it. The debug logging helps track initialization issues.

## 3. Install Python 3.9.4 and Update pip
```bash
# Install Python 3.9.4
pyenv install 3.9.4

# Set as global version
pyenv global 3.9.4

# Verify Python installation
python --version  # Should show Python 3.9.4

# Upgrade pip to latest version
python -m pip install --upgrade pip
```
> Installs specific Python version and ensures pip is up to date.

## 4. Install and Configure pipenv
```bash
# Install pipenv
pip install pipenv

# Add to ~/.zshrc
cat >> ~/.zshrc << 'EOL'
# pipenv configuration
export PIPENV_VENV_IN_PROJECT=1

# Pipenv completion
_pipenv() {
  eval $(env COMMANDLINE="${words[1,$CURRENT]}" _PIPENV_COMPLETE=zsh_source pipenv)
}
compdef _pipenv pipenv
EOL
```
> Sets up pipenv with Zsh-specific completion support.

## 5. Create Project Setup Script
```bash
# Create scripts directory
mkdir -p ~/scripts

# Create the project setup script
cat > ~/scripts/create-python-project.sh << 'EOSCRIPT'
#!/usr/bin/env zsh

if [[ -z "$1" ]]; then
    print -P "%F{red}Please provide a project name%f"
    exit 1
fi

PROJECT_NAME=$1

# Create project directory
mkdir -p "$PROJECT_NAME"
cd "$PROJECT_NAME"

# Initialize pipenv with Python 3.9.4
pipenv --python 3.9.4

# Create project structure
mkdir -p src tests docs

# Create initial files
touch src/__init__.py
touch tests/__init__.py
touch README.md
touch .env

# Create .gitignore
cat > .gitignore << 'END'
.venv/
__pycache__/
*.pyc
.env
.pytest_cache/
.coverage
htmlcov/
dist/
build/
*.egg-info/
.DS_Store
END

# Create VS Code settings
mkdir -p .vscode
cat > .vscode/settings.json << 'END'
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
    "python.linting.enabled": true,
    "python.linting.flake8Enabled": true,
    "python.formatting.provider": "black",
    "editor.formatOnSave": true,
    "editor.rulers": [88],
    "python.testing.pytestEnabled": true,
    "python.testing.unittestEnabled": false,
    "python.testing.nosetestsEnabled": false,
    "python.testing.pytestArgs": [
        "tests"
    ]
}
END

# Install development packages
pipenv install flask python-dotenv pytest pytest-cov black flake8 mypy

# Initialize git repository
git init

print -P "%F{green}Project $PROJECT_NAME created successfully!%f"
EOSCRIPT

# Make script executable
chmod +x ~/scripts/create-python-project.sh

# Add scripts directory to PATH in ~/.zshrc
echo 'export PATH="$HOME/scripts:$PATH"' >> ~/.zshrc
```
> Creates a Zsh-specific project setup script with proper syntax and color output.

## 6. Add Zsh-Specific Python Development Settings
```bash
# Add to ~/.zshrc
cat >> ~/.zshrc << 'EOL'
# Python aliases
alias py='python'
alias ptest='pytest'
alias pr='pipenv run'
alias psh='pipenv shell'

# Zsh-specific file operations
alias pyc-clean='rm -rf **/*.pyc(N) **/__pycache__(N)'
alias pytest-clean='rm -rf **/.pytest_cache(N)'
alias py-clean='pyc-clean && pytest-clean'
alias pyfind='find . -name "*.py"(N) -not -path "*.venv*"'
alias pygrep='grep --include="*.py" -nr'

# Zsh-specific settings
setopt EXTENDED_GLOB  # Enhanced pattern matching

# Python helper functions
function pynew {
    if [[ -z "$1" ]]; then
        print -P "%F{red}Usage: pynew project_name%f"
        return 1
    fi
    ~/scripts/create-python-project.sh "$1" && cd "$1" && pipenv shell
}

function venvs {
    print -P "%F{cyan}Virtual Environments:%f"
    ls -1 ~/.local/share/virtualenvs
}

function workon {
    local venv_path="$(pipenv --venv 2>/dev/null)"
    if [[ -n "$venv_path" ]]; then
        source "$venv_path/bin/activate"
    else
        print -P "%F{red}No virtual environment found in current directory%f"
    fi
}

# Python completions
autoload -Uz compinit && compinit
compdef _pip pip
compdef _python python
EOL
```
> Sets up Zsh-specific aliases, functions, and completions for Python development.

## 7. Install VS Code Extensions
```bash
# Install essential Python extensions
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension njpwerner.autodocstring
```
> Installs necessary VS Code extensions for Python development.

## 8. Verify Installation
```bash
# Reload shell configuration
source ~/.zshrc

# Verify all components
python --version  # Should show 3.9.4
pip --version    # Should show latest version
pipenv --version # Should show latest version
pyenv --version  # Should show latest version

# Test project creation
pynew test_project
```

## Common Issues and Solutions

1. If pyenv command not found:
```bash
exec "$SHELL"  # Restart your shell
```

2. If pip install fails:
```bash
python -m pip install --upgrade pip  # Try upgrading pip again
```

3. If pipenv install fails:
```bash
pip install --user pipenv  # Install for current user only
```

4. If script permissions are wrong:
```bash
chmod +x ~/scripts/create-python-project.sh
```

5. If VS Code doesn't recognize Python:
```bash
# In VS Code command palette (Ctrl+Shift+P):
# Select Python: Select Interpreter
# Choose the pipenv environment
```

## Zsh-Specific Features to Remember

1. Globbing Patterns:
   - `**/*.py(N)` - Recursive file matching with null glob
   - `(N)` prevents errors if no matches found

2. Color Output:
   - `print -P "%F{color}text%f"` for colored output
   - Colors: red, green, blue, cyan, yellow

3. Function Definitions:
   - Use `function name {` or `name() {`
   - Use `[[` for conditions
   - Use `fi` to end if statements

4. Completion System:
   - More powerful than Bash
   - Uses `compdef` for custom completions
   - Autoloads completions with `compinit`

Would you like me to proceed with the Bash version now?