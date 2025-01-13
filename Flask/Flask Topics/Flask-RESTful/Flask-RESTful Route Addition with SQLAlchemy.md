## Step 1: Create Schema (api/api/schemas/your_feature.py)
```python
from api.extensions import ma
from api.models import YourModel  # Your SQLAlchemy model

# Base Schema - For basic operations
class YourModelSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = YourModel
        sqla_session = db.session
        include_fk = True  # Include if you need foreign keys

# Detailed Schema - For GET operations with relationships
class YourModelDetailSchema(YourModelSchema):
    # Add related fields
    related_items = ma.Nested('RelatedSchema', many=True)
    computed_field = ma.String(dump_only=True)  # Read-only field

# Create Schema - For POST operations with validation
class YourModelCreateSchema(ma.Schema):
    required_field = ma.String(required=True)
    optional_field = ma.String()
    
    @validates('required_field')
    def validate_field(self, value):
        if len(value) < 3:
            raise ValidationError("Field must be at least 3 characters")
```
**Note**: Choose schema types based on your needs. Create separate schemas for different operations if they have different requirements.

## Step 2: Create Resource (api/api/resources/your_feature.py)
```python
from flask import request
from flask_restful import Resource
from flask_jwt_extended import jwt_required, get_jwt_identity
from api.models import YourModel
from api.extensions import db
from api.api.schemas import (
    YourModelSchema, 
    YourModelDetailSchema, 
    YourModelCreateSchema
)

class YourModelResource(Resource):
    """
    Single item operations
    ---
    get:
      tags:
        - your-feature
      summary: Get single item
      parameters:
        - in: path
          name: item_id
          schema:
            type: integer
      responses:
        200:
          content:
            application/json:
              schema: YourModelDetailSchema
    """
    method_decorators = [jwt_required()]  # Protect all methods

    def get(self, item_id):
        item = YourModel.query.get_or_404(item_id)
        return YourModelDetailSchema().dump(item)

    def put(self, item_id):
        item = YourModel.query.get_or_404(item_id)
        item = YourModelSchema().load(request.json, instance=item)
        db.session.commit()
        return YourModelDetailSchema().dump(item)

class YourModelList(Resource):
    """List operations"""
    method_decorators = [jwt_required()]

    def get(self):
        schema = YourModelSchema(many=True)
        query = YourModel.query
        return schema.dump(query.all())

    def post(self):
        schema = YourModelCreateSchema()
        item = schema.load(request.json)
        db.session.add(item)
        db.session.commit()
        return YourModelDetailSchema().dump(item), 201
```
**Note**: The docstring format is important for Swagger documentation. Include all parameters and responses.

## Step 3: Update Schema Init (api/api/schemas/__init__.py)
```python
from api.api.schemas.your_feature import (
    YourModelSchema,
    YourModelDetailSchema,
    YourModelCreateSchema
)

__all__ = [
    # ... existing schemas ...
    "YourModelSchema",
    "YourModelDetailSchema",
    "YourModelCreateSchema",
]
```
**Note**: This makes your schemas importable from api.api.schemas

## Step 4: Update Resource Init (api/api/resources/__init__.py)
```python
from api.api.resources.your_feature import YourModelResource, YourModelList

__all__ = [
    # ... existing resources ...
    "YourModelResource",
    "YourModelList",
]
```
**Note**: This makes your resources importable from api.api.resources

## Step 5: Register in Views (api/api/views.py)
```python
# Add imports
from api.api.resources import YourModelResource, YourModelList
from api.api.schemas import (
    YourModelSchema,
    YourModelDetailSchema,
    YourModelCreateSchema,
)

# Register routes
api.add_resource(YourModelResource, "/your-features/<int:item_id>")
api.add_resource(YourModelList, "/your-features")

@blueprint.before_app_request
def register_views():
    # Add schemas to registration
    schemas = {
        # ... existing schemas ...
        "YourModelSchema": YourModelSchema,
        "YourModelDetailSchema": YourModelDetailSchema,
        "YourModelCreateSchema": YourModelCreateSchema,
    }

    # Register paths for Swagger
    for resource in [
        # ... existing resources ...
        YourModelResource,
        YourModelList,
    ]:
        apispec.spec.path(view=resource, app=current_app)
```

## Common Gotchas and Tips:

1. **Circular Imports**:
   - Keep model imports inside methods if needed
   - Use proper __init__.py files
   - Don't import models in schemas/__init__.py

2. **Schema Registration**:
   - Register ALL schemas used in responses
   - Include nested schemas
   - Check schema names are unique

3. **JWT Protection**:
   - Don't forget method_decorators for protection
   - Handle current_user_id properly
   - Add proper permission checks

4. **Swagger Documentation**:
   - Follow docstring format exactly
   - Include all possible responses
   - Document error cases

5. **Relationship Loading**:
   - Be careful with lazy loading in schemas
   - Use joinedload for performance
   - Consider separate schemas for deep relationships

6. **Validation**:
   - Add validation in Create/Update schemas
   - Handle validation errors properly
   - Use proper error messages

7. **Database Sessions**:
   - Commit after all changes
   - Handle rollbacks in error cases
   - Use atomic operations

8. **Testing**:
   - Test each endpoint individually
   - Test with and without authentication
   - Test validation cases

## Quick Checklist:
- [ ] Created schemas
- [ ] Created resources
- [ ] Updated __init__.py files
- [ ] Registered routes
- [ ] Registered schemas
- [ ] Added Swagger docs
- [ ] Added authentication
- [ ] Added validation
- [ ] Tested endpoints

Want me to:
1. Add more specific examples?
2. Show more validation patterns?
3. Add more testing examples?
4. Show more complex routing patterns?
Further reading:

### 1. Schemas
```python
# Your SQLAlchemy Model
class Event(db.Model):
    id = db.Column(db.BigInteger, primary_key=True)
    title = db.Column(db.Text, nullable=False)
    start_date = db.Column(db.DateTime)

# Your Schema
class EventSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Event  # Links to your model
        load_instance = True  # Converts JSON directly to model instance
        sqla_session = db.session
```
- Yes, schemas align your DB and routes - they handle serialization (DB → JSON) and deserialization (JSON → DB)
- Field names should match for clarity, but can be different if needed
- SQLAlchemy types are automatically converted to appropriate schema types:
  ```python
  db.BigInteger → ma.Integer
  db.Text → ma.String
  db.DateTime → ma.DateTime
  ```

### 2. Resources
```python
class EventResource(Resource):
    # Yes, method_decorators is standard Flask-RESTful
    method_decorators = [jwt_required()]  # Applies to all methods
    
    # Custom decorator would go in api/auth/decorators.py
    @admin_required  # Your custom decorator
    def put(self, id):  # 'put' is built into Flask-RESTful
        pass

# Custom decorator example (api/auth/decorators.py)
from functools import wraps
from flask_jwt_extended import get_jwt_identity

def admin_required():
    def wrapper(fn):
        @wraps(fn)
        def decorator(*args, **kwargs):
            current_user_id = get_jwt_identity()
            user = User.query.get(current_user_id)
            if not user.is_admin:
                return {"message": "Admin required"}, 403
            return fn(*args, **kwargs)
        return decorator
    return wrapper
```
- Yes, Resource is your route handler
- HTTP methods (get, post, put, delete) are built into Flask-RESTful
- They automatically route to the correct method based on HTTP method

### 3. __init__.py as Index
```python
# api/api/resources/__init__.py
from api.api.resources.event import EventResource, EventList
from api.api.resources.user import UserResource, UserList

__all__ = [
    "EventResource", "EventList",
    "UserResource", "UserList"
]
```
Yes, it acts as an index and makes imports cleaner:
```python
# Instead of:
from api.api.resources.event import EventResource

# You can do:
from api.api.resources import EventResource
```

### DB Methods and Clean Routes
```python
class EventResource(Resource):
    def get(self, id):
        # Use model methods to keep routes clean
        event = Event.query.get_or_404(id)
        return EventSchema().dump(event)

    def put(self, id):
        event = Event.query.get_or_404(id)
        if not event.can_user_edit(get_jwt_identity()):  # Model method
            return {"message": "Not authorized"}, 403
```
Yes, keep business logic in models, keep routes clean!

### get_or_404() Lookups
```python
# Instead of:
event = Event.query.get(id)
if not event:
    return {"message": "Not found"}, 404

# Use this:
event = Event.query.get_or_404(id)  # Automatically returns 404 if not found
```
get_or_404() is a convenience method that returns 404 if nothing found

### partial=True and PATCH
```python
class EventResource(Resource):
    def put(self, id):  # Full update
        event = Event.query.get_or_404(id)
        # Must provide all fields
        event = EventSchema().load(request.json, instance=event)
        
    def patch(self, id):  # Partial update
        event = Event.query.get_or_404(id)
        # Can provide just some fields
        event = EventSchema().load(
            request.json, 
            instance=event, 
            partial=True
        )
```
partial=True means fields are optional - perfect for PATCH requests

### Docstrings and Swagger
```python
class EventResource(Resource):
    """
    Event operations
    ---
    get:
      summary: Get an event
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
      responses:
        200:
          content:
            application/json:
              schema: EventSchema
        404:
          description: Event not found
    """
    def get(self, id):
        event = Event.query.get_or_404(id)
        return EventSchema().dump(event)
```
Yes! The docstring (especially after `---`) is used by Swagger to:
- Document endpoints
- Show expected parameters
- Show response formats
- Generate interactive documentation

Quick Examples:
```python
# Schema with relationships
class EventSchema(ma.SQLAlchemyAutoSchema):
    organizer = ma.Nested('UserSchema', only=('id', 'name'))
    attendees = ma.Nested('UserSchema', many=True, exclude=('password',))

# Resource with custom decorator
class EventResource(Resource):
    method_decorators = {
        'get': [jwt_required()],
        'put': [jwt_required(), admin_required()],
        'delete': [jwt_required(), admin_required()]
    }

# Model method for clean routes
class Event(db.Model):
    def can_user_edit(self, user_id):
        return EventUser.query.filter_by(
            event_id=self.id,
            user_id=user_id,
            role='organizer'
        ).first() is not None

# Route using model method
class EventResource(Resource):
    def put(self, id):
        event = Event.query.get_or_404(id)
        if not event.can_user_edit(get_jwt_identity()):
            return {"message": "Not authorized"}, 403
```

