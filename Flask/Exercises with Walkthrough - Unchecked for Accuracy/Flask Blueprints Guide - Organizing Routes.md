Additional help for: https://appacademy.instructure.com/courses/319/pages/routing-blueprints-in-flask-2?module_item_id=56603

## 1. Understanding Blueprints

### What are Blueprints?
```python
# Think of Blueprints like:
# Express: Router modules
# React: Route grouping
# Django: URL patterns
```
Blueprints are Flask's way of organizing related routes and views into modular components, similar to how Express uses Router.

## 2. Basic Blueprint Structure

### Creating a Blueprint
```python
# app/routes/admin.py
from flask import Blueprint

bp = Blueprint('admin',           # Blueprint name
               __name__,          # Where blueprint is defined
               url_prefix='/admin' # Optional: prefix for all routes
)
```
This creates a modular set of routes that all start with '/admin'. The name 'admin' helps identify the blueprint in your application.

### Registering with Flask
```python
# app/__init__.py
from flask import Flask
from app.routes import admin  # Import your blueprint

app = Flask(__name__)
app.register_blueprint(admin.bp)  # Register the blueprint
```
Registration connects your blueprint routes to the main application. Think of it like plugging in a new module.

## 3. Defining Routes

### Basic Route Definition
```python
# app/routes/admin.py
from flask import Blueprint

bp = Blueprint('admin', __name__, url_prefix='/admin')

@bp.route('/dashboard')
def dashboard():
    return 'Admin Dashboard'

# This route becomes: /admin/dashboard
```
Routes are defined just like regular Flask routes, but using the blueprint instead of the app object.

### Routes with Methods
```python
@bp.route('/users', methods=['GET', 'POST'])
def manage_users():
    if request.method == 'POST':
        # Handle user creation
        return 'User created'
    # Show users list
    return 'Users list'
```
Blueprints support all the same route features as regular Flask routes.

## 4. Blueprint Organization

### Recommended Structure
```plaintext
your_app/
├── app/
│   ├── __init__.py
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── auth.py
│   │   └── main.py
│   ├── templates/
│   │   ├── admin/
│   │   ├── auth/
│   │   └── main/
│   └── static/
└── run.py
```
This structure keeps related routes together and separates concerns.

### Multiple Blueprints
```python
# app/routes/admin.py
bp = Blueprint('admin', __name__, url_prefix='/admin')

# app/routes/auth.py
bp = Blueprint('auth', __name__, url_prefix='/auth')

# app/routes/main.py
bp = Blueprint('main', __name__)  # No prefix for main routes
```
Each blueprint can have its own prefix and handle different parts of your application.

## 5. Advanced Blueprint Features

### Template Organization
```python
# Blueprint with template folder
bp = Blueprint('admin', __name__,
               url_prefix='/admin',
               template_folder='templates/admin')

@bp.route('/dashboard')
def dashboard():
    return render_template('dashboard.html')  # Looks in templates/admin
```
Blueprints can have their own template directories for better organization.

### Static Files
```python
bp = Blueprint('admin', __name__,
               url_prefix='/admin',
               static_folder='static/admin')
```
Each blueprint can have its own static files, keeping assets organized.

## 6. Complete Example

### Blueprint Definition
```python
# app/routes/admin.py
from flask import Blueprint, render_template, request

bp = Blueprint('admin', __name__, 
               url_prefix='/admin',
               template_folder='templates/admin')

@bp.route('/dashboard')
def dashboard():
    return render_template('dashboard.html')

@bp.route('/users', methods=['GET', 'POST'])
def users():
    if request.method == 'POST':
        # Handle user creation
        pass
    return render_template('users.html')
```

### Application Setup
```python
# app/__init__.py
from flask import Flask
from app.routes import admin, auth, main

def create_app():
    app = Flask(__name__)
    
    # Register blueprints
    app.register_blueprint(admin.bp)
    app.register_blueprint(auth.bp)
    app.register_blueprint(main.bp)
    
    return app
```

## 7. Best Practices

### Blueprint Naming
```python
# Clear, descriptive names
bp = Blueprint('admin', __name__)      # Good
bp = Blueprint('a', __name__)          # Bad

# Consistent naming across files
admin_bp = Blueprint('admin', __name__)  # Alternative style
```

### Route Organization
```python
# Group related functionality
@bp.route('/users')
@bp.route('/users/<int:id>')
@bp.route('/users/<int:id>/edit')
def users():
    # Handle user operations
    pass
```

## 8. Quick Reference

### Blueprint Creation
```python
Blueprint(name,                # Unique identifier
          import_name,        # Usually __name__
          url_prefix=None,    # URL prefix for all routes
          template_folder=None,# Custom template directory
          static_folder=None)  # Custom static directory
```

### Common Patterns
```python
# URL Building
url_for('blueprint_name.view_function')

# Error Handlers
@bp.errorhandler(404)
def handle_404(error):
    return 'Not found', 404

# Before/After Requests
@bp.before_request
def before_request():
    # Do something before each request
    pass
```

Remember:
- Use meaningful blueprint names
- Keep related routes together
- Consider URL prefixes carefully
- Organize templates logically
- Register blueprints in application factory
- Use consistent naming conventions
- Document blueprint purposes
