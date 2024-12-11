Additional help for:  https://appacademy.instructure.com/courses/319/pages/introduction-to-flask-2?module_item_id=56598
## 1. What is Flask & Why Use It?

### Overview
Flask is a lightweight Python web framework that provides:
- Basic web server functionality
- URL routing
- Easy setup and configuration
- Extensibility through additional packages

### Why Choose Flask?
```plaintext
Advantages:
✓ Minimal setup required
✓ Flexible project structure
✓ Easy to learn, quick to deploy
✓ Scalable from small to large projects
✓ Database agnostic
```

## 2. Initial Setup

### Environment Setup
```bash
# Create project directory
mkdir flask_project
cd flask_project

# Set up virtual environment
pipenv install Flask~=2.2.2
pipenv install python-dotenv~=0.21

# Verify installation
pipenv run flask --version
```

### Basic Project Structure
```plaintext
flask_project/
├── simple.py         # Main application file
├── .flaskenv         # Environment variables
└── config.py         # Configuration file
```

## 3. Your First Flask Application

### Basic Application (simple.py)
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello, world!</h1>'
```

💡 Learning Check:
```python
# Try adding another route:
@app.route('/about')
def about():
    return '<h1>About Page</h1>'
```

## 4. Running Your Application

### Method 1: Using Environment Variables
```bash
# Set environment variables manually
export FLASK_APP=simple.py
export FLASK_DEBUG=True

# Run the application
pipenv run flask run
```

### Method 2: Using .flaskenv (Recommended)
Create `.flaskenv`:
```plaintext
FLASK_APP=simple.py
FLASK_DEBUG=True
```

Then run:
```bash
pipenv run flask run
```

### Running Options
```bash
# Default port (5000)
pipenv run flask run

# Custom port
pipenv run flask run -p 5001
```

## 5. Configuration Management

### Option 1: Direct Configuration
```python
app = Flask(__name__)
app.config["greeting"] = "Hello, World!"
```

### Option 2: Config Class (Recommended)
```python
# config.py
class Config(object):
    GREETING = 'Welcome to Flask!'
    SECRET_KEY = 'dev-key'

# simple.py
from config import Config
app.config.from_object(Config)
```

### Option 3: Environment Variables
```python
# config.py
import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'default-key'
```

## 6. Best Practices & Common Patterns

### Project Structure
```plaintext
flask_project/
├── app/
│   ├── __init__.py
│   ├── routes.py
│   └── templates/
├── config.py
├── .flaskenv
└── run.py
```

### Configuration Priority
1. Environment Variables
2. .flaskenv file
3. Config class defaults

### Debug Mode Benefits
- Auto-reload on code changes
- Detailed error pages
- Interactive debugger

## 7. Troubleshooting Guide

### Common Issues
```plaintext
Error: Could not locate Flask application
Solution: Check FLASK_APP environment variable

Error: Address already in use
Solution: Change port or find/close existing process
```

## 8. Quick Reference

### Essential Commands
```bash
# Start application
pipenv run flask run

# Start on specific port
pipenv run flask run -p 5001

# Check Flask version
pipenv run flask --version
```

### Common Routes Pattern
```python
@app.route('/')
def index():
    return 'Home Page'

@app.route('/about')
def about():
    return 'About Page'

@app.route('/user/<username>')
def user_profile(username):
    return f'Profile page of {username}'
```

## 9. Practice Exercises

1. Basic Route Creation
```python
# Create routes for:
# - Home page
# - About page
# - Contact page
```

2. Configuration Practice
```python
# Create a config class with:
# - Application name
# - Debug mode
# - Secret key
```

3. Environment Variables
```python
# Set up .flaskenv with:
# - Flask application file
# - Debug mode
# - Custom environment variable
```

## 10. Next Steps
- HTML Templates
- Forms handling
- Database integration
- User sessions
- Error handling

Remember:
- Always use virtual environments
- Keep configuration separate
- Enable debug mode in development
- Use .flaskenv for environment variables
- Follow Flask naming conventions
- Keep routes organized
- Document your configuration

