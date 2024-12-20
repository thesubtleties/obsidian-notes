## Common Beginner Pitfalls

### 1. Route Parameter Handling
```python
# WRONG - Missing parameters in function definition
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>")
def update_sprint():  # Missing parameters!
    sprint_id = request.view_args.get('sprint_id')  # Don't do this

# CORRECT - Parameters match route
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>")
def update_sprint(project_id, sprint_id):  # Parameters match URL
    # Now you can use project_id and sprint_id directly
```

### 2. Circular Import Issues
```python
# WRONG - circular imports
# models/feature.py
from .sprint import Sprint  # Don't import models directly

# CORRECT - use strings in relationships
class Feature(db.Model):
    sprint = db.relationship('Sprint', back_populates='features')  # Use string
```

### 3. Session Management
```python
# WRONG - not handling session errors
@sprint_routes.route("/", methods=["POST"])
def create_sprint():
    sprint = Sprint(name="New Sprint")
    db.session.add(sprint)
    db.session.commit()  # Could fail!

# CORRECT - proper session handling
@sprint_routes.route("/", methods=["POST"])
def create_sprint():
    try:
        sprint = Sprint(name="New Sprint")
        db.session.add(sprint)
        db.session.commit()
    except SQLAlchemyError:
        db.session.rollback()
        return {"message": "Database error"}, 500
```

## Deep Dives

### 1. Blueprint Registration Deep Dive
```python
# Multiple ways to structure blueprints

# Option 1: Flat structure
app.register_blueprint(sprint_routes, url_prefix='/api/sprints')

# Option 2: Nested structure (what we used)
api = Blueprint('api', __name__)
api.register_blueprint(sprint_routes, url_prefix='/sprints')
app.register_blueprint(api, url_prefix='/api')

# Option 3: Resource-based nesting
project_routes.register_blueprint(sprint_routes, url_prefix='/<int:project_id>/sprints')

# Pros and cons:
# Flat: Simple but less organized
# Nested: More organized but more complex
# Resource-based: Most RESTful but most complex
```

### 2. Error Handling Deep Dive
```python
# Custom error handling class
class APIError(Exception):
    def __init__(self, message, status_code=400, payload=None):
        super().__init__()
        self.message = message
        self.status_code = status_code
        self.payload = payload

    def to_dict(self):
        rv = dict(self.payload or ())
        rv['message'] = self.message
        return rv

# Register error handler
@app.errorhandler(APIError)
def handle_api_error(error):
    response = jsonify(error.to_dict())
    response.status_code = error.status_code
    return response

# Usage in routes
@sprint_routes.route("/")
def get_sprints():
    if some_error_condition:
        raise APIError("Custom error message", status_code=400)
```

### 3. Request Data Handling Gotchas
```python
# WRONG - assuming data exists
@sprint_routes.route("/", methods=["POST"])
def create_sprint():
    name = request.json['name']  # Will raise KeyError if missing

# WRONG - not checking content type
@sprint_routes.route("/", methods=["POST"])
def create_sprint():
    if not request.is_json:  # Always check content type!
        return {"message": "Content type must be application/json"}, 415

# CORRECT - full request validation
@sprint_routes.route("/", methods=["POST"])
def create_sprint():
    if not request.is_json:
        return {"message": "Content type must be application/json"}, 415
    
    data = request.json
    if not isinstance(data, dict):
        return {"message": "Invalid JSON format"}, 400
        
    name = data.get('name')
    if not name or not isinstance(name, str):
        return {"message": "Valid name required"}, 400
```

### 4. Authentication/Authorization Deep Dive
```python
# Multiple levels of protection

# Level 1: Basic auth
@login_required

# Level 2: Resource access
@require_project_access

# Level 3: Specific permissions
def require_permission(permission):
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            if not current_user.has_permission(permission):
                return {"message": "Permission denied"}, 403
            return f(*args, **kwargs)
        return decorated_function
    return decorator

# Usage
@sprint_routes.route("/", methods=["DELETE"])
@login_required
@require_project_access
@require_permission('delete_sprint')
def delete_sprint(project_id, sprint_id):
    # Now we know:
    # 1. User is logged in
    # 2. User has access to project
    # 3. User has delete permission
```

### 5. Common Query Pattern Gotchas
```python
# N+1 Query Problem
# WRONG
sprints = Sprint.query.all()
for sprint in sprints:
    print(sprint.features)  # Makes a new query for each sprint!

# CORRECT
sprints = Sprint.query.options(joinedload(Sprint.features)).all()
for sprint in sprints:
    print(sprint.features)  # No additional queries

# Lazy Loading Gotcha
# WRONG - might raise error after session closes
def get_sprint_data(sprint_id):
    sprint = Sprint.query.get(sprint_id)
    return {
        'id': sprint.id,
        'features': [f.to_dict() for f in sprint.features]  # Session might be closed!
    }

# CORRECT - load everything needed within session
def get_sprint_data(sprint_id):
    sprint = Sprint.query.options(joinedload(Sprint.features)).get(sprint_id)
    return {
        'id': sprint.id,
        'features': [f.to_dict() for f in sprint.features]
    }
```

### 6. Response Formatting Gotchas
```python
# WRONG - inconsistent response formats
@sprint_routes.route("/success")
def success():
    return {"data": "success"}

@sprint_routes.route("/error")
def error():
    return "error", 400  # Different format!

# CORRECT - consistent response format
def api_response(data=None, message=None, status=200):
    response = {}
    if data is not None:
        response['data'] = data
    if message is not None:
        response['message'] = message
    return response, status

@sprint_routes.route("/success")
def success():
    return api_response(data="success")

@sprint_routes.route("/error")
def error():
    return api_response(message="error", status=400)
```

