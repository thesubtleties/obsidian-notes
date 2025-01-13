## Basic Relationship Patterns

### One-to-Many Relationship
```python
# Parent Model (One)
class Organization(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String)
    
    # One org has many events
    events = db.relationship("Event", back_populates="organization")

# Child Model (Many)
class Event(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    organization_id = db.Column(db.Integer, db.ForeignKey("organizations.id"))
    
    # Each event belongs to one org
    organization = db.relationship("Organization", back_populates="events")
```
This pattern is used when one record can have many related records, but each related record belongs to only one parent.

### Many-to-Many Relationship
```python
# Junction Table
class EventUser(db.Model):
    __tablename__ = "event_users"
    event_id = db.Column(db.Integer, db.ForeignKey("events.id"), primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey("users.id"), primary_key=True)
    role = db.Column(db.Enum(EventUserRole))

    # Direct relationships to both sides
    event = db.relationship("Event", back_populates="event_users")
    user = db.relationship("User", back_populates="event_users")

# First Model
class Event(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    
    # Secondary relationship (many-to-many)
    users = db.relationship(
        "User",
        secondary="event_users",
        back_populates="events"
    )
    # Direct relationship to junction
    event_users = db.relationship("EventUser", back_populates="event")

# Second Model
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    
    # Secondary relationship (many-to-many)
    events = db.relationship(
        "Event",
        secondary="event_users",
        back_populates="users"
    )
    # Direct relationship to junction
    event_users = db.relationship("EventUser", back_populates="user")
```
This pattern is used when records on both sides can have many relationships to each other.

## Usage Examples

### Accessing Relationships
```python
# One-to-Many
org = Organization.query.first()
org_events = org.events  # List of events

event = Event.query.first()
event_org = event.organization  # Single organization

# Many-to-Many
event = Event.query.first()
event_users = event.users  # List of users
event_user_roles = event.event_users  # List of EventUser (with roles)

user = User.query.first()
user_events = user.events  # List of events
user_event_roles = user.event_users  # List of EventUser (with roles)
```

### Querying Through Relationships
```python
# Find events for an organization
org_events = Event.query.filter_by(organization_id=org.id).all()

# Find users with specific role in event
speakers = (User.query
    .join(EventUser)
    .filter(
        EventUser.event_id == event.id,
        EventUser.role == EventUserRole.SPEAKER
    ).all())

# Find events where user is speaker
speaking_events = (Event.query
    .join(EventUser)
    .filter(
        EventUser.user_id == user.id,
        EventUser.role == EventUserRole.SPEAKER
    ).all())
```

## Common Patterns

### Cascade Delete
```python
class Event(db.Model):
    # Delete sessions when event is deleted
    sessions = db.relationship(
        "Session",
        back_populates="event",
        cascade="all, delete-orphan"
    )
```

### Lazy Loading Options
```python
class Event(db.Model):
    # Default lazy loading
    users = db.relationship("User", lazy="select")
    
    # Eager loading
    users = db.relationship("User", lazy="joined")
    
    # Dynamic loading (query object)
    users = db.relationship("User", lazy="dynamic")
```

### Relationship Properties
```python
class Event(db.Model):
    @property
    def speaker_count(self):
        return (EventUser.query
            .filter_by(
                event_id=self.id,
                role=EventUserRole.SPEAKER
            ).count())

    @property
    def active_speakers(self):
        return [eu.user for eu in self.event_users 
                if eu.role == EventUserRole.SPEAKER]
```

## Best Practices

### 1. Relationship Naming
```python
# Clear, descriptive names
class User(db.Model):
    # Better than just 'events'
    organized_events = db.relationship(
        "Event",
        secondary="event_users",
        primaryjoin="and_(User.id==EventUser.user_id, "
                   "EventUser.role=='organizer')",
        back_populates="organizers"
    )
```

### 2. Relationship Access Methods
```python
class Event(db.Model):
    def add_user(self, user, role: EventUserRole):
        """Safe way to add user with role"""
        if not self.has_user(user):
            event_user = EventUser(
                event_id=self.id,
                user_id=user.id,
                role=role
            )
            db.session.add(event_user)

    def has_user(self, user) -> bool:
        """Check if user is in event"""
        return user in self.users
```

### 3. Validation in Relationships
```python
class EventUser(db.Model):
    def update_role(self, new_role: EventUserRole):
        """Update with validation"""
        if new_role == self.role:
            return False
            
        if (self.role == EventUserRole.ORGANIZER and 
            self._is_last_organizer()):
            raise ValueError("Cannot remove last organizer")
            
        self.role = new_role
        return True
```

## Common Gotchas

### 1. Circular Imports
```python
# Instead of top-level imports
from api.models import Event  # Can cause circular imports

# Import inside method
def get_events(self):
    from api.models import Event  # Safe from circular imports
    return Event.query.filter_by(user_id=self.id).all()
```

### 2. Back-populates vs Backref
```python
# Back-populates (explicit, preferred)
class Event(db.Model):
    users = db.relationship("User", back_populates="events")

class User(db.Model):
    events = db.relationship("Event", back_populates="users")

# Backref (implicit, not recommended)
class Event(db.Model):
    users = db.relationship("User", backref="events")
```

### 3. Loading Related Data
```python
# N+1 Problem
events = Event.query.all()
for event in events:
    print(event.organization.name)  # Causes additional query per event

# Solution: Eager loading
events = Event.query.options(
    db.joinedload(Event.organization)
).all()
```

Remember:
- Always match back_populates on both sides
- Use explicit relationship names
- Consider cascade behavior
- Handle circular imports carefully
- Use eager loading when appropriate
- Add validation where needed

