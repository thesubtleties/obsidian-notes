## Basic Route Setup (app/api/routes/sprint_routes.py)

### Initial Route File Setup
```python
# Essential imports
from flask import Blueprint, request, jsonify
from flask_login import login_required, current_user
from app.models import db, Sprint, Project
from app.utils.auth_decorators import require_project_access
from datetime import datetime

# Create blueprint
sprint_routes = Blueprint("sprints", __name__)

# Optional: Error message constants
ERRORS = {
    "not_found": "Project couldn't be found",
    "unauthorized": "Unauthorized",
    "validation": "Validation error"
}
```
Key points:
- Import only what you need
- Blueprint name should be descriptive
- Consider centralizing error messages
- Group related functionality

### GET Route (Basic Retrieval)
```python
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required  # Authentication check
@require_project_access  # Authorization check
def get_all_sprints_project(project_id):
    """Get all sprints for a project"""
    # Using model class method (more flexible - can add filters)
    sprints = Sprint.get_all_sprints_for_project(project_id).all()
    
    # Alternative direct query (simpler but less flexible)
    # sprints = Sprint.query.filter_by(project_id=project_id).all()
    
    return [sprint.to_dict() for sprint in sprints]
```
Important considerations:
- Order of decorators matters (@route first, then @login_required)
- Use model methods for complex queries
- Return lists of dictionaries for JSON serialization
- Consider pagination for large datasets

### POST Route (Creation)
```python
@sprint_routes.route("/projects/<int:project_id>/sprints", methods=["POST"])
@login_required
@require_project_access
def create_sprint(project_id):
    """Create a new sprint"""
    data = request.json
    errors = {}

    # Validate required fields
    if not data.get("name"):
        errors["name"] = "Name is required"

    if errors:
        return {
            "message": "Validation error",
            "errors": errors
        }, 400

    try:
        new_sprint = Sprint.create_sprint(
            project_id=project_id,
            name=data.get("name"),
            start_date=data.get("start_date"),  # Optional field
            end_date=data.get("end_date")       # Optional field
        )
        return new_sprint.to_dict(), 201  # 201 Created
    except ValueError as e:
        return {
            "message": "Validation error",
            "errors": {"model_error": str(e)}
        }, 400
```
POST route patterns:
- Validate data before processing
- Use try/except for model operations
- Return 201 for successful creation
- Include validation error details
- Use .get() for optional fields

### PUT Route (Update)
```python
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>", methods=["PUT"])
@login_required
@require_project_access
def update_sprint(project_id, sprint_id):  # Need both IDs for nested routes
    """Update existing sprint"""
    sprint = Sprint.query.get(sprint_id)
    if not sprint:
        return {"message": "Sprint couldn't be found"}, 404

    data = request.json
    
    try:
        # Using instance method
        updated_sprint = sprint.update_sprint(
            name=data.get("name"),
            start_date=data.get("start_date"),
            end_date=data.get("end_date")
        )
        return updated_sprint.to_dict(), 200
    except ValueError as e:
        return {
            "message": "Validation error",
            "errors": {"model_error": str(e)}
        }, 400
```
Update considerations:
- Check resource exists before updating
- Use instance methods for updates
- Return 200 for successful updates
- Handle validation errors
- Consider partial updates

### DELETE Route
```python
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>", methods=["DELETE"])
@login_required
@require_project_access
def delete_sprint(project_id, sprint_id):
    """Delete a sprint"""
    sprint = Sprint.query.get(sprint_id)
    if not sprint:
        return {"message": "Sprint couldn't be found"}, 404

    # Using class method that returns boolean
    if Sprint.delete_sprint(sprint_id):
        return {"message": "Sprint successfully deleted"}, 200
        # Or for strict REST: return "", 204
    else:
        return {"message": "Delete failed"}, 400
```
Delete patterns:
- Verify resource exists
- Consider using 204 No Content for success
- Handle deletion failures
- Check for cascade implications
- Consider soft deletes for data preservation
