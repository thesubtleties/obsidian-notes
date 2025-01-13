## Table of Contents
1. Basic Schema Structure
2. Schema Types and Use Cases
3. Schema Configuration Options
4. Working with Model Properties
5. Relationships and Nested Data
6. Validation
7. Advanced Features
8. Best Practices
9. Additional Considerations

## 1. Basic Schema Structure

### Basic SQLAlchemy Auto Schema
```python
from api.extensions import ma, db
from api.models import YourModel
from marshmallow import validates, ValidationError

class YourModelSchema(ma.SQLAlchemyAutoSchema):
    """Base schema for basic serialization"""
    class Meta:
        model = YourModel              # SQLAlchemy model to base schema on
        sqla_session = db.session      # Database session
        include_fk = True             # Include foreign keys
        load_instance = True          # Create model instances on load
```

### Common Meta Options
```python
class Meta:
    model = YourModel
    sqla_session = db.session
    include_fk = True
    load_instance = True
    exclude = ("created_at",)         # Fields to exclude
    include = ("computed_field",)     # Additional fields to include
    dump_only = ("id",)              # Read-only fields
    load_only = ("password",)        # Write-only fields
```

## 2. Schema Types and Use Cases

### Base Schema (List Views)
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    """Basic schema for list views"""
    class Meta:
        model = Event
        sqla_session = db.session
        include_fk = True
        exclude = ('created_at',)  # Exclude unnecessary fields for lists
```

### Detail Schema (Single Item Views)
```python
class EventDetailSchema(EventSchema):
    """Enhanced schema for detailed views"""
    # Add relationships and additional data
    organization = ma.Nested(
        'OrganizationSchema',
        only=('id', 'name')
    )
    sessions = ma.Nested(
        'SessionSchema',
        many=True,
        only=('id', 'title', 'start_time')
    )
```

### Create Schema (Validation for Creation)
```python
class EventCreateSchema(ma.Schema):
    """Schema for creating new events"""
    title = ma.String(required=True)
    start_date = ma.DateTime(required=True)
    
    @validates('title')
    def validate_title(self, value):
        if len(value) < 3:
            raise ValidationError("Title must be at least 3 characters")
```

### Update Schema (Partial Updates)
```python
class EventUpdateSchema(ma.Schema):
    """Schema for updates - all fields optional"""
    title = ma.String()
    description = ma.String()
    start_date = ma.DateTime()
```

## 3. Schema Configuration Options

### Field Types
```python
class ExampleSchema(ma.Schema):
    # Basic types
    string_field = ma.String()
    integer_field = ma.Integer()
    float_field = ma.Float()
    boolean_field = ma.Boolean()
    
    # Special types
    email = ma.Email()
    url = ma.URL()
    date = ma.Date()
    datetime = ma.DateTime()
    
    # Complex types
    dict_field = ma.Dict()
    list_field = ma.List(ma.String())
    nested = ma.Nested('OtherSchema')
```

### Field Options
```python
class FieldOptionsSchema(ma.Schema):
    # Common options
    required_field = ma.String(
        required=True,               # Must be included
        allow_none=True,            # Can be null
        load_only=True,             # Only for deserialization
        dump_only=True,             # Only for serialization
        data_key="fieldName"        # Different name in JSON
    )
```

## 4. Working with Model Properties

### Computed Properties
```python
# In your model
class Event(db.Model):
    @property
    def is_upcoming(self):
        return self.start_date > datetime.utcnow()

# In your schema
class EventSchema(ma.SQLAlchemyAutoSchema):
    # Include computed property
    is_upcoming = ma.Boolean(dump_only=True)
```

### Custom Fields
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    # Custom formatted field
    duration = ma.Method("get_duration")
    
    def get_duration(self, obj):
        return f"{obj.duration_minutes} minutes"
```

## 5. Relationships and Nested Data

### Basic Relationship
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    # Simple nested relationship
    organization = ma.Nested(
        'OrganizationSchema',
        only=('id', 'name')
    )
```

### Many Relationship
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    # List of related items
    sessions = ma.Nested(
        'SessionSchema',
        many=True,
        only=('id', 'title', 'start_time')
    )
```

### Accessing Related Model Fields
```python
class SessionSpeakerSchema(ma.SQLAlchemyAutoSchema):
    # Fields from related model
    speaker_name = ma.String(attribute='user.full_name')
    email = ma.String(attribute='user.email')
    company = ma.String(attribute='user.company_name')
```

## 6. Validation

### Basic Validation
```python
class EventCreateSchema(ma.Schema):
    @validates('title')
    def validate_title(self, value):
        if len(value) < 3:
            raise ValidationError("Title must be at least 3 characters")
```

### Multiple Field Validation
```python
class EventCreateSchema(ma.Schema):
    @validates_schema
    def validate_dates(self, data, **kwargs):
        if data['end_date'] <= data['start_date']:
            raise ValidationError("End date must be after start date")
```

## 7. Advanced Features

### Conditional Fields
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    special_info = ma.Method("get_special_info")
    
    def get_special_info(self, obj):
        if obj.status == 'published':
            return "Additional info"
        return None
```

### Dynamic Field Selection
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = Event
        fields = ('id', 'title')  # Base fields
        
    def __init__(self, *args, **kwargs):
        # Get fields to include
        include_fields = kwargs.pop('include_fields', [])
        super().__init__(*args, **kwargs)
        
        if include_fields:
            # Only include specified fields
            self.fields = {
                key: field for key, field in self.fields.items()
                if key in include_fields
            }
```

## 8. Best Practices

### Schema Organization
```python
# Separate schemas by functionality
class UserSchema(ma.SQLAlchemyAutoSchema):
    """Base schema for lists"""
    pass

class UserDetailSchema(UserSchema):
    """Enhanced schema for single item"""
    pass

class UserCreateSchema(ma.Schema):
    """Validation schema for creation"""
    pass
```

### Relationship Loading
```python
# Be specific about what you need
class EventSchema(ma.SQLAlchemyAutoSchema):
    # Don't load unnecessary fields
    organization = ma.Nested(
        'OrganizationSchema',
        only=('id', 'name')  # Only what you need
    )
```

### Documentation
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    """
    Event schema for list views.
    
    Used for:
    - GET /events
    - Basic event information in related resources
    """
    class Meta:
        model = Event
```

## 9. Additional Considerations

### Performance
- Use `only` to limit fields in relationships
- Consider using `select_in` for loading relationships
- Be careful with deep nesting
- Use base schemas for lists

### Security
- Use `dump_only` for sensitive fields
- Validate input data
- Consider field-level permissions
- Be careful with nested relationships

### Maintainability
- Keep schemas focused and single-purpose
- Document schema usage
- Use consistent naming
- Follow relationship patterns

### Common Gotchas
- Remember trailing commas in tuples
- Watch for circular imports
- Be careful with relationship loading
- Consider validation timing

### Testing
- Test serialization
- Test deserialization
- Test validation
- Test relationships

