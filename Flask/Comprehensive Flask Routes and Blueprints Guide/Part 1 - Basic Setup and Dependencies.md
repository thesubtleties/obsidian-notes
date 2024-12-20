### Initial Project Structure
```
app/
├── __init__.py          # Main Flask app initialization
├── models/             
│   ├── __init__.py     # Model imports
│   ├── db.py           # Database instance
│   └── user.py         # Example model
├── api/
│   ├── __init__.py     # Blueprint registration
│   └── routes/
│       ├── __init__.py
│       └── sprint_routes.py
└── config.py           # Configuration
```

### Database Setup (app/models/db.py)
```python
from flask_sqlalchemy import SQLAlchemy
import os

environment = os.getenv("FLASK_ENV")
SCHEMA = os.environ.get("SCHEMA")

db = SQLAlchemy()

def add_prefix_for_prod(attr):
    """Add schema prefix for production database"""
    if environment == "production":
        return f"{SCHEMA}.{attr}"
    return attr
```
This creates our database instance that we'll use throughout the app. The production schema handling is specific to services like Render.

### Main App Setup (app/__init__.py)
```python
from flask import Flask
from flask_login import LoginManager
from flask_cors import CORS
from flask_migrate import Migrate
from .models import db, User
from .api import api
from .config import Config

app = Flask(__name__)
login = LoginManager(app)
login.login_view = 'auth.unauthorized'

# Setup login manager
@login.user_loader
def load_user(id):
    return User.query.get(int(id))

# Configure app
app.config.from_object(Config)

# Initialize extensions
db.init_app(app)
Migrate(app, db)
CORS(app)

# Register main API blueprint
app.register_blueprint(api, url_prefix='/api')
```
This is our main application setup. Key points:
- Initializes Flask app
- Sets up login management
- Configures database
- Registers main API blueprint
- Sets up CORS for frontend communication

### API Blueprint Registration (app/api/__init__.py)
```python
from flask import Blueprint
from .routes.sprint_routes import sprint_routes
# Import other route blueprints

api = Blueprint('api', __name__)

# Register route blueprints with URL prefixes
api.register_blueprint(sprint_routes, url_prefix='/sprints')
# Register other blueprints
```
This creates our main API blueprint and registers sub-blueprints. Important notes:
- URL prefixes stack (e.g., /api/sprints)
- Can have different prefixes than actual route paths
- Keeps routing modular and organized

### Common Utility Decorators (app/utils/auth_decorators.py)
```python
from functools import wraps
from flask_login import current_user
from flask import jsonify

def require_project_access(f):
    """Decorator to check project access"""
    @wraps(f)
    def decorated_function(project_id, *args, **kwargs):
        project = Project.query.get(project_id)
        if not project:
            return {"message": "Project couldn't be found"}, 404
        if not current_user.has_project_access(project_id):
            return {"message": "Unauthorized"}, 403
        return f(project_id, *args, **kwargs)
    return decorated_function
```
Utility decorators help:
- Reduce code duplication
- Centralize authorization logic
- Make routes cleaner
- Ensure consistent error handling
