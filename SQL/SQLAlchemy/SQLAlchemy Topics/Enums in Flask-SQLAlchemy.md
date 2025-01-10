## Basic Setup and Usage
```python
# enums.py
from enum import Enum

class EventType(str, Enum):
    CONFERENCE = "conference"
    SINGLE_SESSION = "single_session"

# models.py
class Event(db.Model):
    event_type = db.Column(db.Enum(EventType), nullable=False)
```
Using str, Enum makes your enums work seamlessly with both Python and your database. The values are stored as strings but have all enum validation benefits.

## Validation & Comparison
```python
# Direct string comparison
if event.event_type == EventType.CONFERENCE:
    print("It's a conference!")

# Validation in routes
@app.route('/events', methods=['POST'])
def create_event():
    if request.json['event_type'] not in EventType.__members__:
        return {"error": "Invalid event type"}, 400
    
    event = Event(event_type=EventType(request.json['event_type']))
```
Enums provide type safety and validation out of the box. Invalid values will raise errors.

## Database Queries
```python
# Filter by enum value
conferences = Event.query.filter_by(
    event_type=EventType.CONFERENCE
).all()

# Multiple enum values
draft_or_published = Event.query.filter(
    Event.status.in_([EventStatus.DRAFT, EventStatus.PUBLISHED])
).all()
```
Enums work seamlessly with SQLAlchemy queries and filters.

## Schema Integration
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    event_type = ma.Enum(EventType)
    status = ma.Enum(EventStatus)

    class Meta:
        model = Event
```
Marshmallow will automatically validate enum values during serialization/deserialization.

## Getting Enum Values
```python
# Get all possible values
all_types = list(EventType.__members__.keys())
# ['CONFERENCE', 'SINGLE_SESSION']

# Get string value
event_type_str = EventType.CONFERENCE.value
# 'conference'

# Convert string to enum
event_type = EventType('conference')
# EventType.CONFERENCE
```

## Role-Based Access Control
```python
def check_permission(user, required_role: OrganizationUserRole):
    if user.org_role == required_role:
        return True
    return False

@app.route('/admin-only')
@jwt_required()
def admin_endpoint():
    if not check_permission(current_user, OrganizationUserRole.ADMIN):
        return {"error": "Admin access required"}, 403
```

## Error Handling
```python
try:
    # Will raise ValueError if invalid
    event_type = EventType(some_string)
except ValueError:
    print("Invalid event type")

# Safe conversion
if hasattr(EventType, some_string.upper()):
    event_type = EventType[some_string.upper()]
```

## Benefits

1. **Type Safety**
```python
# This will raise an error - prevents bugs
event.status = "draft"  # Error!
event.status = EventStatus.DRAFT  # Correct
```

2. **Documentation**
```python
class EventStatus(str, Enum):
    DRAFT = "draft"      # Event not visible to public
    PUBLISHED = "published"  # Event is live
    ARCHIVED = "archived"    # Event is past
```

3. **Maintainability**
```python
# Change once, updates everywhere
class EventType(str, Enum):
    CONFERENCE = "conference"
    SINGLE_SESSION = "single_session"
    WORKSHOP = "workshop"  # Add new type here only
```

4. **API Consistency**
```python
# In your API docs
{
    "event_type": ["conference", "single_session"],
    "status": ["draft", "published", "archived"]
}
```

5. **Database Integrity**
```python
# Database will enforce valid values
event_type = db.Column(db.Enum(EventType), nullable=False)
```

## Common Patterns

1. **Default Values**
```python
status = db.Column(
    db.Enum(EventStatus),
    nullable=False,
    default=EventStatus.DRAFT
)
```

2. **Enum Groups**
```python
def is_terminal_status(status: EventStatus) -> bool:
    return status in [EventStatus.ARCHIVED, EventStatus.CANCELLED]
```

3. **Permission Checking**
```python
def can_edit_event(user_role: EventUserRole) -> bool:
    return user_role in [
        EventUserRole.ORGANIZER,
        EventUserRole.MODERATOR
    ]
```

Remember:
- Use str, Enum for database compatibility
- Keep enum definitions centralized
- Use descriptive names
- Document enum values
- Consider adding helper methods

