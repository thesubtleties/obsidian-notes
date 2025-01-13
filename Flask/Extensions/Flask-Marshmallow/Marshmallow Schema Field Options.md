## Basic Field Options
```python
class ExampleSchema(ma.Schema):
    # Common Field Options
    field1 = ma.String(
        required=True,          # Field must be included in input
        allow_none=True,        # Field can be None/null
        load_only=True,         # Field only used for deserialization (reading input)
        dump_only=True,         # Field only used for serialization (sending output)
        data_key="fieldOne",    # Different name in JSON than in Python
        validate=Length(min=3)  # Validation rule
    )
```

## Field Types Reference
```python
class FieldTypesSchema(ma.Schema):
    # String Fields
    string_field = ma.String()           # Any string
    email_field = ma.Email()             # Must be valid email
    url_field = ma.URL()                 # Must be valid URL
    uuid_field = ma.UUID()               # Must be valid UUID

    # Number Fields
    integer_field = ma.Integer()         # Whole numbers
    decimal_field = ma.Decimal()         # Precise decimals
    float_field = ma.Float()             # Floating point numbers
    
    # Boolean Fields
    boolean_field = ma.Boolean()         # True/False
    
    # Date/Time Fields
    date_field = ma.Date()              # Date only
    datetime_field = ma.DateTime()       # Date and time
    time_field = ma.Time()              # Time only
    
    # Special Fields
    enum_field = ma.Enum(SomeEnum)      # Must be valid enum value
    list_field = ma.List(ma.String())   # List of strings
    dict_field = ma.Dict()              # Dictionary/object
    nested_field = ma.Nested('OtherSchema')  # Nested object
```

## Common Options Reference
```python
field = ma.String(
    # Validation Options
    required=True,              # Must be included
    allow_none=False,          # Can't be null
    validate=Length(min=3),    # Custom validation
    
    # Serialization Options
    dump_only=True,            # Only for output
    load_only=True,            # Only for input
    data_key="fieldName",      # Different JSON name
    
    # Default Values
    missing=None,              # Default when missing in input
    default=None,              # Default for output
    
    # Nested Options
    many=True,                 # List of items
    only=("id", "name"),       # Include only these fields
    exclude=("password",),     # Exclude these fields
)
```

## Relationship Fields Example
```python
class UserSchema(ma.SQLAlchemyAutoSchema):
    # One-to-Many Relationship
    posts = ma.Nested(
        'PostSchema',
        many=True,                    # List of posts
        only=("id", "title"),         # Only these fields
        dump_only=True               # Only for output
    )

    # Many-to-Many Relationship
    groups = ma.Nested(
        'GroupSchema',
        many=True,
        exclude=("members",),         # Avoid circular nesting
        dump_default=[]              # Default empty list
    )
```

## Validation Examples
```python
class ValidationSchema(ma.Schema):
    # Built-in Validators
    email = ma.Email(error_messages={"invalid": "Not a valid email"})
    age = ma.Integer(validate=Range(min=0, max=120))
    url = ma.URL(require_tld=True)

    # Custom Validation Method
    @validates('password')
    def validate_password(self, value):
        if len(value) < 8:
            raise ValidationError("Password too short")
        
    # Multiple Validators
    username = ma.String(
        validate=[
            Length(min=3, max=30),
            Regexp(r'^[a-zA-Z0-9]+$')
        ]
    )
```

## Meta Options
```python
class MetaOptionsSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        model = User                # SQLAlchemy model
        load_instance = True        # Create/update model instances
        sqla_session = db.session   # Database session
        include_fk = True          # Include foreign keys
        include_relationships = True # Include model relationships
        unknown = EXCLUDE          # Ignore unknown fields
```

## Common Patterns
```python
# Read-only Computed Property
full_name = ma.String(dump_only=True)

# Write-only Field
password = ma.String(load_only=True)

# Optional Field with Default
status = ma.String(missing="active")

# Required Field with Custom Error
email = ma.Email(
    required=True,
    error_messages={"required": "Email is required"}
)

# Nested Relationship with Depth Control
comments = ma.Nested(
    'CommentSchema',
    many=True,
    only=("id", "text", "author.name")
)
```

## Gotchas and Tips:
1. **Circular Nesting**:
   - Use `only` or `exclude` to prevent infinite nesting
   - Consider separate schemas for deep relationships

2. **Performance**:
   - Use `only` to limit fields in relationships
   - Be careful with deep nesting
   - Consider pagination for large lists

3. **Validation**:
   - Combine multiple validators when needed
   - Use custom validation methods for complex rules
   - Add clear error messages

4. **Defaults**:
   - `missing` is for deserialization (input)
   - `default` is for serialization (output)
   - Use callables for dynamic defaults

5. **Relationships**:
   - Always use `many=True` for lists
   - Handle circular references
   - Consider load performance

