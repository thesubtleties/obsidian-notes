## Core Concepts

### Why UTC?
```python
# Bad - Using local time
event_start = datetime.now()  # Could be any timezone!

# Good - Using UTC
event_start = datetime.now(timezone.utc)  # Always UTC
```

**Explanation:**
- UTC is the universal time standard
- Eliminates timezone confusion
- Makes data consistent across regions
- Simplifies database queries
- Essential for distributed systems

## Database Setup

### Model Configuration
```python
# In your SQLAlchemy models
class Event(db.Model):
    # Store timezone info, will be UTC
    start_date = db.Column(db.DateTime(timezone=True), nullable=False)
    end_date = db.Column(db.DateTime(timezone=True), nullable=False)
    
    # For tracking records
    created_at = db.Column(
        db.DateTime(timezone=True),
        server_default=db.func.current_timestamp()
    )
    updated_at = db.Column(
        db.DateTime(timezone=True),
        onupdate=db.func.current_timestamp()
    )
```

**Explanation:**
- `timezone=True` preserves timezone information
- `server_default` ensures UTC timestamps
- Consistent datetime handling across all models

## Working with Dates

### Creating Dates
```python
from datetime import datetime, timezone, timedelta

# Current time in UTC
now_utc = datetime.now(timezone.utc)

# Future date in UTC
tomorrow_utc = datetime.now(timezone.utc) + timedelta(days=1)

# Creating specific UTC datetime
specific_date = datetime(2025, 1, 1, tzinfo=timezone.utc)
```

### Validation Patterns
```python
class EventCreateSchema(ma.Schema):
    start_date = ma.DateTime(required=True)
    end_date = ma.DateTime(required=True)

    @validates_schema
    def validate_dates(self, data, **kwargs):
        if "start_date" in data and "end_date" in data:
            now = datetime.now(timezone.utc)
            
            # Ensure start is future
            if data["start_date"] <= now:
                raise ValidationError("Start date must be in the future")
                
            # Ensure end after start
            if data["end_date"] <= data["start_date"]:
                raise ValidationError("End date must be after start date")
```

## Common Patterns

### Model Properties
```python
class Event(db.Model):
    @property
    def is_upcoming(self):
        """Check if event hasn't started"""
        return self.start_date > datetime.now(timezone.utc)

    @property
    def is_ongoing(self):
        """Check if event is currently running"""
        now = datetime.now(timezone.utc)
        return self.start_date <= now <= self.end_date

    @property
    def is_past(self):
        """Check if event has ended"""
        return self.end_date < datetime.now(timezone.utc)
```

### Date Comparisons
```python
def is_date_valid(target_date):
    """Always compare UTC to UTC"""
    now = datetime.now(timezone.utc)
    return target_date > now

def get_date_range(start_date, days):
    """Generate date range in UTC"""
    end_date = start_date + timedelta(days=days)
    return start_date, end_date
```

## API Handling

### Receiving Dates
```python
# Expected format from client
{
    "start_date": "2025-01-13T00:00:00+00:00",  # UTC ISO format
    "end_date": "2025-01-14T00:00:00+00:00"
}

# In your resource
@jwt_required()
def post(self):
    schema = EventCreateSchema()
    data = schema.load(request.json)  # Will parse to UTC datetime
    
    event = Event(**data)
    db.session.add(event)
    db.session.commit()
```

### Sending Dates
```python
class EventSchema(ma.SQLAlchemyAutoSchema):
    """Dates will be serialized in ISO format with UTC timezone"""
    class Meta:
        model = Event
        include_fk = True
```

## Common Pitfalls

### Avoid These Patterns
```python
# BAD - Naive datetime (no timezone)
datetime.now()  

# BAD - Mixing naive and aware datetimes
if naive_date > timezone_aware_date:  # Will raise TypeError

# BAD - Assuming local timezone
datetime.strptime("2025-01-01", "%Y-%m-%d")  # No timezone info
```

### Use These Instead
```python
# GOOD - Always UTC
datetime.now(timezone.utc)

# GOOD - Convert to UTC if needed
naive_date.replace(tzinfo=timezone.utc)

# GOOD - Parse with timezone
datetime.strptime("2025-01-01", "%Y-%m-%d").replace(tzinfo=timezone.utc)
```

## Testing

### DateTime Testing Patterns
```python
def test_event_creation():
    # Create future date for testing
    future_date = datetime.now(timezone.utc) + timedelta(days=1)
    
    event = Event(
        title="Test Event",
        start_date=future_date,
        end_date=future_date + timedelta(hours=2)
    )
    
    assert event.is_upcoming == True
    assert event.is_ongoing == False
```

## Frontend Considerations

```javascript
// Convert UTC to local time for display
const localStartDate = new Date(event.start_date).toLocaleString()

// Send UTC to backend
const utcDate = new Date().toISOString()  // Sends in UTC
```

## Best Practices Summary

1. Database:
   - Always use timezone=True
   - Store everything in UTC

2. Backend Code:
   - Always create dates with timezone.utc
   - Always compare UTC to UTC
   - Validate dates in UTC

3. API:
   - Accept ISO format with timezone
   - Return ISO format in UTC
   - Validate timezone presence

4. Frontend:
   - Convert UTC to local only for display
   - Always send UTC to backend
   - Use ISO string format

