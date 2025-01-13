## 1. Basic Structure
```python
api/
├── api/
│   ├── resources/      # Route handlers
│   ├── schemas/        # Marshmallow schemas
│   ├── commons/        # Shared utilities
│   └── auth/          # Auth-related code
├── models/            # SQLAlchemy models
└── extensions.py      # Flask extensions
```

## 2. Standard Response Format
```python
# api/commons/responses.py
def api_response(data=None, message=None, status=200, meta=None):
    response = {
        "success": 200 <= status < 300,
        "message": message,
        "data": data
    }
    if meta:
        response["meta"] = meta
    return response, status

# Usage in resources
def get(self, id):
    event = Event.query.get_or_404(id)
    return api_response(
        data=EventSchema().dump(event),
        message="Event retrieved successfully"
    )
```

## 3. Error Handling
```python
# api/commons/errors.py
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

# Register globally
@app.errorhandler(APIError)
def handle_api_error(error):
    return api_response(
        message=error.message,
        status=error.status_code
    )

# Usage
if not user.can_edit(event):
    raise APIError("Not authorized", status_code=403)
```

## 4. Schema Patterns
```python
# Base Schema
class BaseSchema(ma.SQLAlchemyAutoSchema):
    class Meta:
        load_instance = True
        include_fk = True

# Model Schema
class EventSchema(BaseSchema):
    class Meta(BaseSchema.Meta):
        model = Event
    
    # Computed fields
    duration = ma.Function(lambda obj: obj.end_date - obj.start_date)
    
    # Nested relationships
    organizer = ma.Nested('UserSchema', only=('id', 'name'))
    
    # Field validation
    @validates('end_date')
    def validate_end_date(self, value):
        if value <= self.context.get('start_date'):
            raise ValidationError("End date must be after start date")
```

## 5. Resource Patterns
```python
class EventResource(Resource):
    """
    Single event operations
    ---
    get:
      summary: Get event details
      responses:
        200:
          content:
            application/json:
              schema: EventSchema
    """
    method_decorators = {
        'get': [jwt_required()],
        'put': [jwt_required(), admin_required()],
        'delete': [jwt_required(), admin_required()]
    }

    def get(self, id):
        event = Event.query.options(
            joinedload(Event.attendees)  # Eager loading
        ).get_or_404(id)
        return api_response(EventSchema().dump(event))

    @validate_args({  # Request validation
        'title': fields.Str(required=True),
        'date': fields.Date(required=True)
    })
    def put(self, id, args):
        event = Event.query.get_or_404(id)
        event = EventSchema().load(args, instance=event)
        db.session.commit()
        return api_response(EventSchema().dump(event))
```

## 6. Pagination
```python
# api/commons/pagination.py
def paginate(query, schema, **kwargs):
    page = request.args.get('page', 1, type=int)
    per_page = min(
        request.args.get('per_page', 10, type=int),
        100  # Max limit
    )
    
    paginated = query.paginate(page=page, per_page=per_page)
    
    return api_response(
        data=schema.dump(paginated.items),
        meta={
            "total": paginated.total,
            "pages": paginated.pages,
            "page": page,
            "per_page": per_page
        }
    )

# Usage in resource
def get(self):
    query = Event.query.order_by(Event.created_at.desc())
    return paginate(query, EventSchema(many=True))
```

## 7. Query Optimization
```python
# Eager loading relationships
events = (Event.query
    .options(
        joinedload(Event.organizer),
        joinedload(Event.attendees)
    )
    .filter(Event.status == 'active')
    .all())

# Complex filters
def get(self):
    query = Event.query
    
    # Apply filters
    if 'type' in request.args:
        query = query.filter(Event.type == request.args['type'])
    
    # Apply search
    if 'q' in request.args:
        search = f"%{request.args['q']}%"
        query = query.filter(Event.title.ilike(search))
    
    # Apply sorting
    sort = request.args.get('sort', 'created_at')
    direction = request.args.get('direction', 'desc')
    if direction == 'desc':
        query = query.order_by(desc(sort))
    else:
        query = query.order_by(asc(sort))
```

## 8. Common Decorators
```python
# api/auth/decorators.py
def admin_required():
    def wrapper(fn):
        @wraps(fn)
        def decorator(*args, **kwargs):
            user_id = get_jwt_identity()
            user = User.query.get(user_id)
            if not user.is_admin:
                raise APIError("Admin required", 403)
            return fn(*args, **kwargs)
        return decorator
    return wrapper

def validate_args(schema):
    def wrapper(fn):
        @wraps(fn)
        def decorator(*args, **kwargs):
            try:
                kwargs['args'] = schema.load(request.json)
            except ValidationError as err:
                raise APIError(str(err), 400)
            return fn(*args, **kwargs)
        return decorator
    return wrapper
```

## Common Gotchas & Tips:

1. **Circular Imports**
   - Import models inside functions when needed
   - Use lazy relationships in SQLAlchemy

2. **Performance**
   - Always use pagination for lists
   - Eager load relationships you know you'll need
   - Index frequently queried columns

3. **Security**
   - Never trust client input
   - Always validate request data
   - Use proper error handling
   - Implement rate limiting for sensitive endpoints

4. **Maintenance**
   - Keep responses consistent
   - Document all endpoints
   - Use clear naming conventions
   - Write tests for all endpoints

5. **Common Mistakes**
   - Not handling errors properly
   - N+1 query problems
   - Missing input validation
   - Inconsistent response formats
   - Not using transactions when needed

