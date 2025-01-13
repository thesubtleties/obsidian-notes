## Conceptual Overview
A REST API resource represents an entity or collection in your system. Each resource:
- Maps to a URL endpoint
- Handles HTTP methods (GET, POST, PUT, DELETE)
- Manages data validation and serialization
- Controls access and permissions
- Returns appropriate responses and status codes

## Setup Requirements

1. Register resources in your views.py:
```python
# api/api/views.py
from flask_restful import Api
from api.api.resources import (
    SignupResource,
    LoginResource,
    UserResource,
    OrganizationResource
)

blueprint = Blueprint("api", __name__, url_prefix="/api")
api = Api(blueprint)

# Register routes
api.add_resource(SignupResource, '/auth/signup')
api.add_resource(LoginResource, '/auth/login')
api.add_resource(UserResource, '/users/<int:user_id>')
api.add_resource(OrganizationResource, '/organizations/<int:org_id>')
```

**Explanation:**
- Blueprint creates a URL prefix for all API routes
- Api object manages RESTful resources
- add_resource maps classes to URLs
- URL parameters are typed (<int:user_id>)

2. Configure JWT and Database in extensions:
```python
# api/extensions.py
from flask_sqlalchemy import SQLAlchemy
from flask_jwt_extended import JWTManager
from flask_marshmallow import Marshmallow

db = SQLAlchemy()
jwt = JWTManager()
ma = Marshmallow()
```

**Explanation:**
- Extensions are initialized as singletons
- Used throughout your application
- Provide database, JWT, and serialization functionality

## Basic Resource Structure

```python
from flask import request
from flask_restful import Resource
from flask_jwt_extended import jwt_required, get_jwt_identity
from api.extensions import db
from api.api.schemas import ResourceSchema

class BasicResource(Resource):
    Regular docstring description
    ---
    get:
      tags:
        - resource-name
      summary: Get resource
      responses:
        200:
          content:
            application/json:
              schema: ResourceSchema
    
    @jwt_required()  # Protect endpoint
    def get(self, resource_id):
        # Get current user from token
        current_user_id = int(get_jwt_identity())
        
        # Get requested resource
        resource = Resource.query.get_or_404(resource_id)
        
        # Return serialized data
        return ResourceSchema().dump(resource)
```

**Explanation:**
- Resources inherit from Flask-RESTful's Resource class
- Swagger documentation goes between triple quotes
- @jwt_required() protects endpoints
- get_jwt_identity() retrieves user ID from token
- Schemas handle serialization/deserialization

## Working with Schemas

1. First, create your schemas:
```python
# api/api/schemas/resource.py
from api.extensions import ma
from api.models import YourModel

class BaseSchema(ma.SQLAlchemyAutoSchema):
    """Base schema for simple operations"""
    class Meta:
        model = YourModel
        sqla_session = db.session
        load_instance = True

class DetailSchema(BaseSchema):
    """Detailed schema with relationships"""
    related_items = ma.Nested(
        "RelatedSchema",
        many=True,
        only=("id", "name")
    )
```

**Explanation:**
- Schemas define how data is serialized/deserialized
- Base schemas handle simple operations
- Detail schemas include relationships
- only= specifies which fields to include

## Authentication Flow

1. First, set up JWT configuration:
```python
# api/config.py
JWT_SECRET_KEY = os.getenv("JWT_SECRET_KEY", "jwt-secret-key")
JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=1)
JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)
JWT_BLOCKLIST_ENABLED = True
JWT_BLOCKLIST_TOKEN_CHECKS = ["access", "refresh"]
```

**Explanation:**
- Configures JWT token settings
- Sets token expiration times
- Enables token blocklisting for logout

2. Create Authentication Resources:
```python
class SignupResource(Resource):
    Regular docstring description
    ---
    post:
      tags:
        - auth
      summary: Register new user
      requestBody:
        content:
          application/json:
            schema: SignupSchema
      responses:
        201:
          content:
            application/json:
              schema:
                properties:
                  message:
                    type: string
                  access_token:
                    type: string
                  refresh_token:
                    type: string
    
    def post(self):
        # Validate input
        schema = SignupSchema()
        data = schema.load(request.json)
        
        # Check existing user
        if User.query.filter_by(email=data["email"]).first():
            return {"message": "Email already registered"}, 400
            
        # Create user
        user = User(**data)
        db.session.add(user)
        db.session.commit()
        
        # Generate tokens
        access_token = create_access_token(identity=str(user.id))
        refresh_token = create_refresh_token(identity=str(user.id))
        
        return {
            "message": "User created successfully",
            "access_token": access_token,
            "refresh_token": refresh_token
        }, 201
```

**Explanation:**
- Validates incoming data using schema
- Checks for existing users
- Creates new user and generates tokens
- Returns tokens for immediate authentication

## Protected Resources with Permissions

```python
class OrganizationResource(Resource):
    Regular docstring description
    ---
    get:
      tags:
        - organizations
      summary: Get organization details
      parameters:
        - in: path
          name: org_id
          schema:
            type: integer
          required: true
      responses:
        200:
          content:
            application/json:
              schema: OrganizationDetailSchema
        403:
          description: Not authorized
    
    @jwt_required()
    def get(self, org_id):
        # Get current user from token
        current_user_id = int(get_jwt_identity())
        current_user = User.query.get_or_404(current_user_id)
        
        # Get organization
        org = Organization.query.get_or_404(org_id)
        
        # Check permissions
        if not org.has_user(current_user):
            return {"message": "Not authorized"}, 403
            
        # Return detailed view
        return OrganizationDetailSchema().dump(org)
```

**Explanation:**
- @jwt_required() ensures authentication
- Convert JWT identity to int (stored as string)
- Check permissions before operations
- Return appropriate status codes
- Use detailed schema for response

## Handling Relationships (Many-to-Many)

```python
class OrganizationUserList(Resource):
    Regular docstring description
    ---
    post:
      tags:
        - organization-users
      summary: Add user to organization
      parameters:
        - in: path
          name: org_id
          schema:
            type: integer
          required: true
      requestBody:
        content:
          application/json:
            schema: OrganizationUserCreateSchema
    
    @jwt_required()
    def post(self, org_id):
        current_user_id = int(get_jwt_identity())
        current_user = User.query.get_or_404(current_user_id)
        org = Organization.query.get_or_404(org_id)
        
        # Check admin status
        if not current_user.is_org_admin(org_id):
            return {"message": "Admin access required"}, 403
            
        # Validate input
        schema = OrganizationUserCreateSchema()
        data = schema.load(request.json)
        
        # Get user to add
        new_user = User.query.get_or_404(data['user_id'])
        
        # Check if already member
        if org.has_user(new_user):
            return {"message": "Already a member"}, 400
            
        # Add user with role
        org.add_user(new_user, data['role'])
        db.session.commit()
        
        return OrganizationUserDetailSchema().dump(
            OrganizationUser.query.filter_by(
                organization_id=org_id,
                user_id=new_user.id
            ).first()
        ), 201
```

**Explanation:**
- Handles many-to-many relationships
- Checks permissions on both entities
- Prevents duplicate relationships
- Uses specific schemas for creation/response
- Returns detailed relationship data

## Error Handling and Best Practices

1. Set up error handlers:
```python
@blueprint.errorhandler(ValidationError)
def handle_marshmallow_error(e):
    """Return json error for marshmallow validation errors."""
    return jsonify(e.messages), 400

@jwt.invalid_token_loader
def invalid_token_callback(error):
    return {"message": "Invalid token"}, 401

@jwt.expired_token_loader
def expired_token_callback(jwt_header, jwt_payload):
    return {"message": "Token has expired"}, 401
```

2. Use pagination for lists:
```python
from api.commons.pagination import paginate

class ListResource(Resource):
    @jwt_required()
    def get(self):
        query = Model.query
        
        # Apply filters
        filter_param = request.args.get('filter')
        if filter_param:
            query = query.filter_by(status=filter_param)
            
        return paginate(query, ModelSchema(many=True))
```

**Best Practices:**
1. Always validate input with schemas
2. Convert JWT identity to int for database
3. Use appropriate status codes
4. Check permissions before operations
5. Use different schemas for different operations
6. Handle relationships through model methods
7. Implement proper error handling
8. Document all endpoints
9. Use pagination for lists
10. Keep resource methods focused

## Testing Your API

1. Using Swagger UI:
- Navigate to /swagger-ui
- Try endpoints with "Try it out"
- Use Bearer token authentication
- Check response codes and bodies

2. Using curl or Postman:
```bash
# Login
curl -X POST http://localhost:5000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"password123"}'

# Use protected endpoint
curl -X GET http://localhost:5000/api/organizations/1 \
  -H "Authorization: Bearer your_token_here"
```

