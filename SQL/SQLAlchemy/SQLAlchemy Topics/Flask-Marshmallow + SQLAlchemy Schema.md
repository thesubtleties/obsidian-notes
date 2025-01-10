

## Basic Schema Setup

### Auto Schema (Recommended)
```python
from api.extensions import ma
from api.models import User

class UserSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User                # Tell schema which model to use
        load_instance = True        # Deserialize directly to model instances
        include_fk = True          # Include foreign key fields
        exclude = ('_password',)    # Fields to exclude from serialization
        
        # Other common Meta options:
        # include_relationships = True  # Include model relationships
        # load_only = ('password',)    # Fields only for deserialization
        # dump_only = ('id',)          # Fields only for serialization
```
AutoSchema automatically generates schema fields based on your SQLAlchemy model. This reduces boilerplate and keeps your serialization in sync with your models. The Meta class configures how the schema behaves.

### Manual Schema
```python
class UserSchema(ma.Schema):
    # Define each field manually
    id = ma.Integer(dump_only=True)         # Read only field
    email = ma.Email(required=True)         # Field with validation
    password = ma.String(load_only=True)    # Write only field
    
    # Common field options:
    # required=True          - Field must be present
    # allow_none=True       - Field can be None
    # validate=some_func    - Custom validation
```
Manual schemas give you complete control over field definitions. This is useful when you need different serialization than what your models define, or when you're not using SQLAlchemy.

## Usage Patterns

### Basic Usage
```python
# Initialize schemas - define how we'll serialize
user_schema = UserSchema()              # For single object
users_schema = UserSchema(many=True)    # For lists of objects

# Serialize (Python → JSON)
user_data = user_schema.dump(user)      # Convert single user to JSON
users_data = users_schema.dump(users)   # Convert list of users to JSON

# Deserialize (JSON → Python)
# This validates and converts incoming JSON to User instance
user = user_schema.load(request.json)
```
The basic pattern is to create schema instances for both single and multiple objects. `dump()` converts to JSON for sending responses, while `load()` validates incoming JSON and converts to Python objects.

### In Routes
```python
@app.route('/api/users/<id>')
def get_user(id):
    # Get user or return 404
    user = User.query.get_or_404(id)
    # Serialize user to JSON
    return user_schema.dump(user)

@app.route('/api/users', methods=['POST'])
def create_user():
    # Validate and convert JSON to User instance
    user = user_schema.load(request.json)
    # Save to database
    db.session.add(user)
    db.session.commit()
    # Return serialized user and 201 Created status
    return user_schema.dump(user), 201
```
In routes, schemas handle both input validation and output formatting. They ensure your API has consistent data formats and validation.

## Customizing Fields

### Adding Computed Fields
```python
class UserSchema(ma.SQLAlchemyAutoSchema):
    # Add fields that don't exist in model
    full_name = ma.String(dump_only=True)       # Computed property
    age = ma.Method("calculate_age")            # Method field
    events_count = ma.Function(                 # Function field
        lambda obj: len(obj.events)
    )

    def calculate_age(self, obj):
        # Custom calculation method
        return calculate_age(obj.birth_date)

    class Meta:
        model = User
```
Computed fields let you add data to your API responses that isn't directly stored in your database. They're great for derived data or formatted values.

### Field Options
```python
class UserSchema(ma.SQLAlchemyAutoSchema):
    # Field with validation
    email = ma.Email(
        required=True,                  # Must be included
        error_messages={                # Custom error messages
            'required': 'Email required',
            'invalid': 'Invalid email'
        }
    )
    
    # Write-only field (for passwords etc)
    password = ma.String(
        load_only=True,                # Only used when loading data
        required=True
    )
    
    # Read-only field (for computed values)
    id = ma.Integer(dump_only=True)    # Only included in output
    
    # Optional field with default
    role = ma.String(
        missing='user',                # Default if not provided
        validate=validate.OneOf(['user', 'admin'])
    )
    
    class Meta:
        model = User
```
Field options give you fine-grained control over how each field behaves during serialization and deserialization. They're crucial for proper API design and data validation.

## Handling Relationships

### Basic Relationships
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Event
        include_fk = True              # Include foreign keys

class UserSchema(ma.SQLAlchemyAutoSchema):
    # Nested relationship - includes full event data
    events = ma.Nested(
        EventSchema,
        many=True,                     # One-to-many relationship
        dump_only=True                 # Read-only relationship
    )
    
    # You can also nest just specific fields
    organization = ma.Nested(
        'OrganizationSchema',
        only=('id', 'name')            # Only include these fields
    )
    
    class Meta:
        model = User
        include_relationships = True    # Include all relationships
```
Nested relationships allow you to include related data in your API responses. This is powerful for reducing API calls but be careful of circular references and data size.

