## Standard Error Response Structure

### Error Constants and Types
```python
# app/utils/error_constants.py
ERROR_MESSAGES = {
    "not_found": {
        "project": "Project couldn't be found",
        "sprint": "Sprint couldn't be found",
        "feature": "Feature couldn't be found"
    },
    "auth": {
        "unauthorized": "Unauthorized",
        "forbidden": "Forbidden"
    },
    "validation": {
        "name_required": "Name is required",
        "invalid_date": "Invalid date format"
    }
}

HTTP_CODES = {
    "success": 200,
    "created": 201,
    "no_content": 204,
    "bad_request": 400,
    "unauthorized": 401,
    "forbidden": 403,
    "not_found": 404
}
```
Centralizing error messages:
- Ensures consistency
- Makes updates easier
- Supports internationalization
- Provides clear documentation
- Makes testing easier

### Standard Error Response Pattern
```python
# app/utils/error_handlers.py
from flask import jsonify

def create_error_response(message, errors=None, status_code=400):
    """Create standardized error response"""
    response = {"message": message}
    if errors:
        response["errors"] = errors
    return response, status_code

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
def get_sprints(project_id):
    project = Project.query.get(project_id)
    if not project:
        return create_error_response(
            ERROR_MESSAGES["not_found"]["project"],
            status_code=HTTP_CODES["not_found"]
        )
```

## Validation Patterns

### Request Validation
```python
# app/utils/validators.py
from datetime import datetime

class RequestValidator:
    """Validate request data"""
    
    @staticmethod
    def validate_sprint_data(data):
        errors = {}
        
        # Required fields
        if not data.get("name"):
            errors["name"] = ERROR_MESSAGES["validation"]["name_required"]
            
        # Optional date validation
        if data.get("start_date"):
            try:
                datetime.strptime(data["start_date"], "%Y-%m-%d")
            except ValueError:
                errors["start_date"] = ERROR_MESSAGES["validation"]["invalid_date"]
                
        return errors

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints", methods=["POST"])
@login_required
@require_project_access
def create_sprint(project_id):
    data = request.json
    errors = RequestValidator.validate_sprint_data(data)
    
    if errors:
        return create_error_response(
            "Validation error",
            errors=errors,
            status_code=HTTP_CODES["bad_request"]
        )
```
Validation best practices:
- Separate validation logic
- Clear error messages
- Handle both required and optional fields
- Type checking
- Format validation

### Exception Handling
```python
# app/utils/exceptions.py
class APIException(Exception):
    """Base exception for API errors"""
    def __init__(self, message, status_code=400, errors=None):
        super().__init__()
        self.message = message
        self.status_code = status_code
        self.errors = errors

class ResourceNotFound(APIException):
    """Resource not found exception"""
    def __init__(self, resource_type):
        super().__init__(
            f"{resource_type} couldn't be found",
            status_code=404
        )

# In your routes
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>")
@login_required
def get_sprint(project_id, sprint_id):
    try:
        sprint = Sprint.query.get(sprint_id)
        if not sprint:
            raise ResourceNotFound("Sprint")
        return sprint.to_dict()
    except APIException as e:
        return create_error_response(
            e.message,
            errors=e.errors,
            status_code=e.status_code
        )
```

## Response Formatting

### Success Response Pattern
```python
# app/utils/responses.py
def create_success_response(data, status_code=200, meta=None):
    """Create standardized success response"""
    response = {"data": data}
    if meta:
        response["meta"] = meta
    return response, status_code

# Usage for collection endpoints
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
def get_sprints(project_id):
    sprints = Sprint.get_all_sprints_for_project(project_id).all()
    return create_success_response(
        [sprint.to_dict() for sprint in sprints],
        meta={
            "total": len(sprints),
            "project_id": project_id
        }
    )
```

### Pagination Pattern
```python
# app/utils/pagination.py
def paginate_results(query, page=1, per_page=20):
    """Paginate query results"""
    pagination = query.paginate(
        page=page,
        per_page=per_page,
        error_out=False
    )
    
    return {
        "items": [item.to_dict() for item in pagination.items],
        "meta": {
            "page": page,
            "per_page": per_page,
            "total": pagination.total,
            "pages": pagination.pages
        }
    }

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
def get_sprints(project_id):
    page = request.args.get('page', 1, type=int)
    per_page = request.args.get('per_page', 20, type=int)
    
    query = Sprint.get_all_sprints_for_project(project_id)
    return create_success_response(**paginate_results(query, page, per_page))
```

