## pyenv Commands
| Command | Description | Example |
|---------|-------------|---------|
| `pyenv install` | Install Python version | `pyenv install 3.11.0` |
| `pyenv versions` | List installed versions | `pyenv versions` |
| `pyenv global` | Set global Python | `pyenv global 3.11.0` |
| `pyenv local` | Set local Python | `pyenv local 3.11.0` |
| `pyenv virtualenv` | Create virtualenv | `pyenv virtualenv 3.11.0 myproject` |
| `pyenv activate` | Activate virtualenv | `pyenv activate myproject` |
| `pyenv deactivate` | Deactivate virtualenv | `pyenv deactivate` |

## Project Setup Flow
```bash
# 1. Install Python version
pyenv install 3.11.0

# 2. Create virtual environment
pyenv virtualenv 3.11.0 projectname

# 3. In project directory, set local version
cd myproject
pyenv local projectname

# 4. Verify activation
python --version
which python

# 5. Install dependencies
pip install -r requirements.txt
# or
pip install pytest pytest-cov
```

## Requirements File Management
```bash
# Generate requirements.txt
pip freeze > requirements.txt

# Install from requirements.txt
pip install -r requirements.txt

# Update all packages
pip list --outdated
pip install -U package_name
```

## Common Directory Structure
```
project/
├── .python-version     # Created by pyenv local
├── requirements.txt
├── README.md
├── .gitignore
├── src/
│   └── package/
│       ├── __init__.py
│       └── module.py
└── tests/
    ├── __init__.py
    └── test_module.py
```

## Essential .gitignore
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
*.egg-info/
.installed.cfg
*.egg

# Virtual Environment
venv/
ENV/
.env

# IDE
.idea/
.vscode/
*.swp
*.swo
```

## Troubleshooting Commands
```bash
# Check Python version
python --version

# Check pip version
pip --version

# Check which Python
which python

# List virtual environments
pyenv virtualenvs

# Check installed packages
pip list

# Verify pip location
which pip
```

## Common Issues & Solutions
```bash
# If pip is outdated
pip install --upgrade pip

# If Python not found
pyenv rehash

# If SSL errors during install
pyenv install 3.11.0 --patch < <(curl -sSL https://github.com/python/cpython/commit/8ea6353.patch)

# If environment not activating
pyenv deactivate
pyenv activate projectname
```

Quick Tips:
- 📌 One project = one virtual environment
- 🔄 Always activate environment before installing packages
- 📝 Keep requirements.txt updated
- 🔒 Never commit virtual environment files
- ⭐ Use `.python-version` for local version
- 💡 Check Python/pip versions after activation

Remember:
- Create new virtualenv for each project
- Activate before installing packages
- Update requirements.txt when adding packages
- Don't install packages globally
- Check which python/pip you're using
- Keep virtual environments isolated

Common Commands Sequence:
```bash
# New project setup
mkdir project && cd project
pyenv install 3.11.0
pyenv virtualenv 3.11.0 projectname
pyenv local projectname
pip install pytest
pip freeze > requirements.txt

# Existing project setup
cd project
pyenv install 3.11.0  # if needed
pyenv virtualenv 3.11.0 projectname
pyenv local projectname
pip install -r requirements.txt
```