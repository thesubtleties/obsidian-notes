## 1. Initial System Setup
```bash
# Update system and install required dependencies
sudo apt update && sudo apt upgrade -y

# Install build dependencies
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```
> These packages are required for building Python from source and installing various Python packages. The build-essential package includes necessary compilation tools.

## 2. Install and Configure pyenv
```bash
# Install pyenv
curl https://pyenv.run | bash

# Add debug logging to track Bash initialization
echo "# Debug Bash startup" >> ~/.bashrc
echo 'echo "Finished Bash initialization at $(date)" >> ~/.bash_debug_log' >> ~/.bashrc

# Add to ~/.bashrc
cat >> ~/.bashrc << 'EOL'

# Path configurations
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
PATH=~/.console-ninja/.bin:$PATH

# pyenv initialization
eval "$(pyenv init -)"
eval "$(pyenv init --path)"

# Enable programmable completion
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
EOL

# Reload shell configuration
exec "$SHELL"
```
> This configuration sets up pyenv and includes debug logging. The PATH configuration ensures all tools are accessible, and bash completion is enabled for better command-line functionality.

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
> This installs the specific Python version we need and ensures pip is up to date. Always upgrade pip first to avoid version warnings.

## 4. Install and Configure pipenv
```bash
# Install pipenv
pip install pipenv

# Add to ~/.bashrc
cat >> ~/.bashrc << 'EOL'

# pipenv configuration
export PIPENV_VENV_IN_PROJECT=1

# Add pipenv completions (only if pipenv is installed)
if command -v pipenv >/dev/null 2>&1; then
    eval "$(pipenv --completion)"
fi

# Enhanced prompt with git and virtual env info
parse_git_branch() {
    git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}

parse_venv() {
    if [ -n "$VIRTUAL_ENV" ]; then
        echo "($(basename $VIRTUAL_ENV))"
    fi
}

# Colorful prompt with git and venv info
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\] \[\033[01;35m\]$(parse_venv)\[\033[00m\]\[\033[01;33m\]$(parse_git_branch)\[\033[00m\]\$ '
EOL
```
> pipenv manages project dependencies and virtual environments. The custom prompt shows your current virtual environment and git branch.

## 5. Configure Bash for Python Development
```bash
# Add Python aliases and functions to ~/.bashrc
cat >> ~/.bashrc << 'EOL'

# Python aliases
alias py='python'
alias ptest='pytest'
alias pr='pipenv run'
alias psh='pipenv shell'

# Python file operations
clean_pyc() {
    find . -type f -name "*.pyc" -delete
    find . -type d -name "__pycache__" -exec rm -r {} +
    find . -type d -name ".pytest_cache" -exec rm -r {} +
    echo -e "\e[32mPython cache files cleaned\e[0m"
}

find_py() {
    find . -type f -name "*.py" "$@"
}

grep_py() {
    find . -type f -name "*.py" -exec grep --color -Hn "$@" {} \;
}

# Python package management
pip_upgrade_all() {
    local packages=()
    while IFS= read -r pkg; do
        packages+=("$pkg")
    done < <(pip list --outdated --format=freeze | cut -d = -f 1)
    
    for package in "${packages[@]}"; do
        echo -e "\e[33mUpgrading $package...\e[0m"
        pip install --upgrade "$package"
    done
}

# pip bash completion
_pip_completion() {
    COMPREPLY=( $( COMP_WORDS="${COMP_WORDS[*]}" \
                   COMP_CWORD=$COMP_CWORD \
                   PIP_AUTO_COMPLETE=1 $1 2>/dev/null ) )
}
complete -o default -F _pip_completion pip
EOL
```
> These settings provide useful aliases and functions for Python development in Bash, including file cleanup and package management tools.

## 6. Install VS Code Extensions
```bash
# Install essential Python extensions
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension njpwerner.autodocstring
```
> These VS Code extensions provide Python language support, intelligent code completion, and documentation tools.

## 7. Create Project Setup Script
```bash
# Create scripts directory
mkdir -p ~/scripts

# Create project setup script
cat > ~/scripts/create-python-project.sh << 'EOL'
#!/bin/bash

if [ -z "$1" ]; then
    echo -e "\e[31mPlease provide a project name\e[0m"
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

echo -e "\e[32mProject $PROJECT_NAME created successfully!\e[0m"
EOL

# Make script executable
chmod +x ~/scripts/create-python-project.sh

# Add scripts directory to PATH in ~/.bashrc
echo 'export PATH="$HOME/scripts:$PATH"' >> ~/.bashrc
```
> This script creates a new Python project with standard directory structure and configuration files.

## 8. Additional Bash Helper Functions
```bash
# Add to ~/.bashrc
cat >> ~/.bashrc << 'EOL'

# Create and activate new Python project
pynew() {
    if [ -z "$1" ]; then
        echo -e "\e[31mUsage: pynew project_name\e[0m"
        return 1
    fi
    create-python-project.sh "$1" && cd "$1" && pipenv shell
}

# List all virtual environments
list_venvs() {
    echo -e "\e[36mVirtual Environments:\e[0m"
    ls -1 ~/.local/share/virtualenvs
}

# Activate virtual environment
workon() {
    local venv_path
    venv_path="$(pipenv --venv 2>/dev/null)"
    if [ -n "$venv_path" ]; then
        source "$venv_path/bin/activate"
    else
        echo -e "\e[31mNo virtual environment found in current directory\e[0m"
    fi
}
EOL
```
> These Bash helper functions provide convenient shortcuts for common Python development tasks.

## 9. Verify Installation
```bash
# Reload shell configuration
source ~/.bashrc

# Verify all components
python --version  # Should show 3.9.4
pip --version    # Should show latest version
pipenv --version # Should show latest version
pyenv --version  # Should show latest version

# Create test project
create-python-project.sh testproject
cd testproject
pipenv shell
```
> These commands verify that everything is installed and configured correctly.

## Common Issues and Solutions:

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

4. If VS Code doesn't recognize Python:
```bash
# In VS Code command palette (Ctrl+Shift+P):
# Select Python: Select Interpreter
# Choose the pipenv environment
```

Remember to run `source ~/.bashrc` after making any changes to the configuration!
