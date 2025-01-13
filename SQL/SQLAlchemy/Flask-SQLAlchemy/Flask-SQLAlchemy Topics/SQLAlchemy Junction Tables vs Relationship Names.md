## Basic Structure

### Junction Table (Actual Model)
```python
# This is a real table in the database
class EventUser(db.Model):
    __tablename__ = "event_users"
    
    # Primary key columns linking both sides
    event_id = db.Column(db.BigInteger, db.ForeignKey("events.id"), primary_key=True)
    user_id = db.Column(db.BigInteger, db.ForeignKey("users.id"), primary_key=True)
    
    # Additional data about the relationship
    role = db.Column(db.Enum(EventUserRole))
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    # Direct relationships to both sides
    event = db.relationship("Event", back_populates="event_users")
    user = db.relationship("User", back_populates="event_users")
```

### Relationship Names (Convenient Access)
```python
class User(db.Model):
    # Easy access to events (through junction table)
    events = db.relationship(
        "Event",
        secondary="event_users",  # Specifies junction table
        back_populates="users"    # Matches name in Event model
    )
    # Direct access to junction records
    event_users = db.relationship(
        "EventUser",
        back_populates="user"
    )

class Event(db.Model):
    # Easy access to users (through junction table)
    users = db.relationship(
        "User",
        secondary="event_users",
        back_populates="events"
    )
    # Direct access to junction records
    event_users = db.relationship(
        "EventUser",
        back_populates="event"
    )
```

## Usage Examples

### Using Junction Table Directly
```python
# Create relationship with additional data
event_user = EventUser(
    event_id=1,
    user_id=1,
    role=EventUserRole.SPEAKER
)
db.session.add(event_user)

# Query relationship with conditions
speakers = EventUser.query.filter_by(
    event_id=1,
    role=EventUserRole.SPEAKER
).all()

# Update relationship data
event_user = EventUser.query.filter_by(
    event_id=1,
    user_id=1
).first()
event_user.role = EventUserRole.MODERATOR
```

### Using Relationship Names
```python
# Get related records easily
user = User.query.first()
user_events = user.events  # List of events

event = Event.query.first()
event_users = event.users  # List of users

# Access junction data when needed
for event_user in user.event_users:
    print(f"Role in event {event_user.event.id}: {event_user.role}")
```

## Common Patterns

### Many-to-Many with Extra Data
```python
# Junction Table with additional fields
class SessionSpeaker(db.Model):
    session_id = db.Column(db.BigInteger, db.ForeignKey("sessions.id"), primary_key=True)
    user_id = db.Column(db.BigInteger, db.ForeignKey("users.id"), primary_key=True)
    role = db.Column(db.Enum(SessionSpeakerRole))
    order = db.Column(db.Integer)  # Speaking order
    bio = db.Column(db.Text)       # Speaker bio for this session

# Accessing the extra data
session = Session.query.first()
for speaker in session.session_speakers:  # Use junction relationship
    print(f"Speaker: {speaker.user.name}")
    print(f"Speaking order: {speaker.order}")
    print(f"Bio: {speaker.bio}")
```

### Helper Methods
```python
class Event(db.Model):
    def add_user(self, user, role: EventUserRole):
        """Add user with role"""
        if not self.has_user(user):
            event_user = EventUser(
                event_id=self.id,
                user_id=user.id,
                role=role
            )
            db.session.add(event_user)

    def get_users_by_role(self, role: EventUserRole):
        """Get users with specific role"""
        return [eu.user for eu in self.event_users if eu.role == role]

class User(db.Model):
    def get_role_in_event(self, event_id) -> EventUserRole:
        """Get user's role in specific event"""
        eu = next((eu for eu in self.event_users 
                  if eu.event_id == event_id), None)
        return eu.role if eu else None
```

## Querying Examples

### Basic Queries
```python
# Through junction table
speakers = EventUser.query.filter_by(
    event_id=1,
    role=EventUserRole.SPEAKER
).all()

# Through relationship
event = Event.query.get(1)
event_speakers = [eu.user for eu in event.event_users 
                 if eu.role == EventUserRole.SPEAKER]
```

### Complex Queries
```python
# Find users who are speakers in any event
speakers = User.query.join(EventUser).filter(
    EventUser.role == EventUserRole.SPEAKER
).distinct().all()

# Find events where user is organizer
user_events = Event.query.join(EventUser).filter(
    EventUser.user_id == user_id,
    EventUser.role == EventUserRole.ORGANIZER
).all()
```

## Best Practices

### 1. Naming Conventions
```python
# Clear, descriptive names for relationships
class User(db.Model):
    # Generic relationship name
    events = db.relationship(...)
    
    # More specific relationship names
    organized_events = db.relationship(
        "Event",
        secondary="event_users",
        primaryjoin="and_(User.id==EventUser.user_id, "
                   "EventUser.role=='organizer')"
    )
```

### 2. Data Access Patterns
```python
# Bad: Accessing junction data through secondary relationship
for event in user.events:
    role = EventUser.query.filter_by(
        event_id=event.id, 
        user_id=user.id
    ).first().role

# Good: Using junction relationship directly
for event_user in user.event_users:
    role = event_user.role
```

### 3. Validation in Junction Model
```python
class EventUser(db.Model):
    def update_role(self, new_role: EventUserRole):
        """Update with validation"""
        if self.role == EventUserRole.ORGANIZER:
            # Check if last organizer
            other_organizers = EventUser.query.filter(
                EventUser.event_id == self.event_id,
                EventUser.role == EventUserRole.ORGANIZER,
                EventUser.user_id != self.user_id
            ).count()
            if other_organizers == 0:
                raise ValueError("Cannot remove last organizer")
        self.role = new_role
```

Remember:
- Junction tables store the actual relationship data
- Relationship names provide convenient access
- Use junction table for detailed relationship data
- Use relationship names for simple access
- Add helper methods for common operations
- Keep validation in junction model

