## Route Testing

### Basic Test Setup
```python
# tests/test_sprint_routes.py
import pytest
from app.models import db, User, Project, Sprint
from app.api.routes.sprint_routes import sprint_routes

@pytest.fixture
def client(app):
    """Test client fixture"""
    return app.test_client()

@pytest.fixture
def auth_headers(client, test_user):
    """Get authentication headers"""
    response = client.post('/api/auth/login', json={
        'email': test_user.email,
        'password': 'password'
    })
    token = response.json['token']
    return {'Authorization': f'Bearer {token}'}

@pytest.fixture
def test_project(test_user):
    """Create test project"""
    project = Project(
        name="Test Project",
        owner_id=test_user.id
    )
    db.session.add(project)
    db.session.commit()
    return project
```

### Route Tests
```python
def test_get_sprints(client, auth_headers, test_project):
    """Test GET /projects/:id/sprints"""
    # Create test sprints
    sprints = [
        Sprint(
            project_id=test_project.id,
            name=f"Sprint {i}"
        ) for i in range(3)
    ]
    db.session.add_all(sprints)
    db.session.commit()

    # Test route
    response = client.get(
        f'/api/projects/{test_project.id}/sprints',
        headers=auth_headers
    )
    
    assert response.status_code == 200
    data = response.json
    assert len(data) == 3
    assert all(s['project_id'] == test_project.id for s in data)

def test_create_sprint(client, auth_headers, test_project):
    """Test POST /projects/:id/sprints"""
    response = client.post(
        f'/api/projects/{test_project.id}/sprints',
        headers=auth_headers,
        json={
            'name': 'New Sprint',
            'start_date': '2024-01-01',
            'end_date': '2024-01-15'
        }
    )
    
    assert response.status_code == 201
    data = response.json
    assert data['name'] == 'New Sprint'
    assert data['project_id'] == test_project.id

def test_validation_errors(client, auth_headers, test_project):
    """Test validation error handling"""
    response = client.post(
        f'/api/projects/{test_project.id}/sprints',
        headers=auth_headers,
        json={}  # Missing required name
    )
    
    assert response.status_code == 400
    data = response.json
    assert data['message'] == 'Validation error'
    assert 'name' in data['errors']
```

## API Documentation

### Route Documentation Decorators
```python
# app/utils/docs.py
from functools import wraps
from flask import current_app

def document_route(summary, params=None, responses=None):
    """Document route for API documentation"""
    def decorator(f):
        f.__doc__ = f"""
        {summary}
        
        Parameters:
        {params or 'None'}
        
        Responses:
        {responses or 'Standard JSON response'}
        """
        return f
    return decorator

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
@require_project_access
@document_route(
    summary="Get all sprints for a project",
    params="""
    - project_id: ID of the project (path parameter)
    - status: Filter by status (query parameter, optional)
    """,
    responses="""
    200: List of sprints
    404: Project not found
    403: Unauthorized
    """
)
def get_sprints(project_id):
    # ... route logic ...
```

### OpenAPI/Swagger Documentation
```python
# app/docs/sprint_routes.yml
paths:
  /projects/{projectId}/sprints:
    get:
      summary: Get all sprints for a project
      parameters:
        - name: projectId
          in: path
          required: true
          schema:
            type: integer
      responses:
        '200':
          description: List of sprints
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Sprint'
        '404':
          $ref: '#/components/responses/NotFound'
        '403':
          $ref: '#/components/responses/Unauthorized'
```

## Error Tracking and Monitoring

### Error Logging
```python
# app/utils/logging.py
import logging
from functools import wraps

logger = logging.getLogger(__name__)

def log_route_errors(f):
    """Log route errors with context"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except Exception as e:
            logger.error(
                f"Route error in {f.__name__}",
                extra={
                    'args': args,
                    'kwargs': kwargs,
                    'error': str(e)
                },
                exc_info=True
            )
            raise
    return decorated_function

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
@require_project_access
@log_route_errors
def get_sprints(project_id):
    # ... route logic ...
```

### Performance Monitoring
```python
# app/utils/monitoring.py
from time import time
from functools import wraps

def monitor_performance(f):
    """Monitor route performance"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        start_time = time()
        result = f(*args, **kwargs)
        duration = time() - start_time
        
        # Log or send to monitoring service
        logger.info(
            f"Route performance: {f.__name__}",
            extra={
                'duration': duration,
                'path': request.path,
                'method': request.method
            }
        )
        
        return result
    return decorated_function

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
@require_project_access
@monitor_performance
def get_sprints(project_id):
    # ... route logic ...
```
