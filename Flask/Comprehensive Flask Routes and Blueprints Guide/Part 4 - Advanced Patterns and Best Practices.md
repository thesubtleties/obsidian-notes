## Query Optimization and Relationship Loading

### Efficient Query Patterns
```python
# app/api/routes/sprint_routes.py
from sqlalchemy.orm import joinedload, selectinload

@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
@require_project_access
def get_sprints_with_features(project_id):
    """Get sprints with efficient relationship loading"""
    
    # Bad: N+1 query problem
    sprints = Sprint.query.filter_by(project_id=project_id).all()
    return [{
        **sprint.to_dict(),
        'features': [f.to_dict() for f in sprint.features]  # Causes N queries!
    } for sprint in sprints]

    # Good: Eager loading with joinedload
    sprints = Sprint.query\
        .filter_by(project_id=project_id)\
        .options(joinedload(Sprint.features))\
        .all()
    return [{
        **sprint.to_dict(),
        'features': [f.to_dict() for f in sprint.features]  # No extra queries
    } for sprint in sprints]
```
Query optimization tips:
- Use joinedload for immediate relationships
- Use selectinload for collections
- Avoid N+1 queries
- Consider query performance in to_dict methods
- Use appropriate indexes

### Transaction Management
```python
# app/utils/db_utils.py
from contextlib import contextmanager
from sqlalchemy.exc import SQLAlchemyError

@contextmanager
def db_transaction():
    """Transaction context manager"""
    try:
        yield
        db.session.commit()
    except SQLAlchemyError as e:
        db.session.rollback()
        raise APIException(f"Database error: {str(e)}")

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints", methods=["POST"])
@login_required
@require_project_access
def create_sprint_with_features(project_id):
    data = request.json
    
    with db_transaction():
        # Create sprint
        sprint = Sprint.create_sprint(
            project_id=project_id,
            name=data['name']
        )
        
        # Create associated features
        for feature_data in data.get('features', []):
            Feature.create_feature(
                sprint_id=sprint.id,
                **feature_data
            )
            
        return sprint.to_dict(), 201
```

## Advanced Route Patterns

### Resource Relationship Routes
```python
@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>/features")
@login_required
@require_project_access
def get_sprint_features(project_id, sprint_id):
    """Get features for a specific sprint"""
    sprint = Sprint.query.get_or_404(sprint_id)
    
    # Optional: Filter features
    status = request.args.get('status')
    features = sprint.features
    
    if status:
        features = [f for f in features if f.status == status]
        
    return [feature.to_dict() for feature in features]

@sprint_routes.route("/projects/<int:project_id>/sprints/<int:sprint_id>/features", methods=["POST"])
@login_required
@require_project_access
def add_feature_to_sprint(project_id, sprint_id):
    """Add a feature to a sprint"""
    sprint = Sprint.query.get_or_404(sprint_id)
    data = request.json
    
    with db_transaction():
        feature = Feature.create_feature(
            sprint_id=sprint_id,
            **data
        )
        return feature.to_dict(), 201
```

### Bulk Operations
```python
@sprint_routes.route("/projects/<int:project_id>/sprints/bulk", methods=["POST"])
@login_required
@require_project_access
def bulk_create_sprints(project_id):
    """Create multiple sprints at once"""
    sprints_data = request.json.get('sprints', [])
    
    if not sprints_data:
        return create_error_response("No sprints provided", status_code=400)
    
    created_sprints = []
    errors = []
    
    with db_transaction():
        for index, sprint_data in enumerate(sprints_data):
            try:
                sprint = Sprint.create_sprint(
                    project_id=project_id,
                    **sprint_data
                )
                created_sprints.append(sprint)
            except ValueError as e:
                errors.append({
                    "index": index,
                    "data": sprint_data,
                    "error": str(e)
                })
                
        if errors:
            # Optionally rollback if any errors
            raise APIException("Some sprints failed validation", errors=errors)
            
    return {
        "sprints": [sprint.to_dict() for sprint in created_sprints],
        "count": len(created_sprints)
    }, 201
```

## Caching Strategies
```python
# app/utils/cache.py
from functools import wraps
from flask_caching import Cache

cache = Cache()

def cached_response(timeout=5*60):
    """Cache decorator for route responses"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            cache_key = f"{f.__name__}:{':'.join(str(arg) for arg in args)}"
            response = cache.get(cache_key)
            
            if response is None:
                response = f(*args, **kwargs)
                cache.set(cache_key, response, timeout=timeout)
                
            return response
        return decorated_function
    return decorator

# Usage in routes
@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
@require_project_access
@cached_response(timeout=300)  # Cache for 5 minutes
def get_sprints(project_id):
    sprints = Sprint.get_all_sprints_for_project(project_id).all()
    return [sprint.to_dict() for sprint in sprints]
```
