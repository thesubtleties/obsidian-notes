## Core Principles

### User-Centric Methods (What can this user do?)
```python
class User(db.Model):
    # Permission & Access Checks
    def can_edit_event(self, event_id) -> bool:
        """Can THIS user edit THIS event?"""
        return self.get_event_role(event_id) in [EventUserRole.ORGANIZER]

    # Role Queries
    def get_event_role(self, event_id) -> EventUserRole:
        """What role does THIS user have?"""
        return EventUser.query.filter_by(
            user_id=self.id,
            event_id=event_id
        ).first().role

    # User's Content/Data
    def get_upcoming_events(self):
        """What events is THIS user involved in?"""
        return Event.query.join(EventUser).filter(
            EventUser.user_id == self.id,
            Event.start_date > db.func.current_timestamp()
        ).all()
```
These methods focus on user capabilities, permissions, and personal data.

### Event-Centric Methods (What's happening in this event?)
```python
class Event(db.Model):
    # Event Management
    def add_speaker(self, user, role=EventUserRole.SPEAKER):
        """Add speaker to THIS event"""
        event_user = EventUser(
            event_id=self.id,
            user_id=user.id,
            role=role
        )
        db.session.add(event_user)

    # Event Queries
    def get_speakers(self):
        """Who are THIS event's speakers?"""
        return User.query.join(EventUser).filter(
            EventUser.event_id == self.id,
            EventUser.role == EventUserRole.SPEAKER
        ).all()

    # Event State
    def is_active(self) -> bool:
        """Is THIS event currently active?"""
        return (
            self.status == EventStatus.PUBLISHED and
            self.start_date <= datetime.utcnow() <= self.end_date
        )
```
These methods handle event operations, state, and event-wide queries.

## Decision Making Guide

### Put in User Model When:
```python
class User(db.Model):
    # ✅ User Permissions
    def is_org_admin(self, org_id) -> bool:
        """Is user admin of this org?"""
        pass

    # ✅ User's Personal Data
    @property
    def full_name(self) -> str:
        """User's full name"""
        pass

    # ✅ User's Relationships
    def get_managed_events(self):
        """Events user manages"""
        pass
```

### Put in Event Model When:
```python
class Event(db.Model):
    # ✅ Event State Management
    def publish(self):
        """Publish this event"""
        pass

    # ✅ Event-wide Operations
    def add_session(self, session_data):
        """Add session to event"""
        pass

    # ✅ Event Queries
    def get_active_sessions(self):
        """Get event's active sessions"""
        pass
```

## Real-World Examples

### User Operations
```python
class User(db.Model):
    def get_speaking_sessions(self):
        """Get all sessions where user is speaking"""
        return Session.query.join(SessionSpeaker).filter(
            SessionSpeaker.user_id == self.id
        ).all()

    def is_available(self, start_time, end_time):
        """Check if user is available during time period"""
        return not Session.query.join(SessionSpeaker).filter(
            SessionSpeaker.user_id == self.id,
            Session.start_time < end_time,
            Session.end_time > start_time
        ).first()

    def can_access_organization(self, org_id):
        """Check if user has org access"""
        return OrganizationUser.query.filter_by(
            user_id=self.id,
            organization_id=org_id
        ).first() is not None
```

### Event Operations
```python
class Event(db.Model):
    def get_attendee_count(self):
        """Get number of attendees"""
        return EventUser.query.filter_by(
            event_id=self.id,
            role=EventUserRole.ATTENDEE
        ).count()

    def has_conflicts(self, start_time, end_time):
        """Check for scheduling conflicts"""
        return Session.query.filter(
            Session.event_id == self.id,
            Session.start_time < end_time,
            Session.end_time > start_time
        ).first() is not None

    def get_schedule(self):
        """Get event schedule"""
        return Session.query.filter_by(
            event_id=self.id
        ).order_by(Session.start_time).all()
```

## Cross-Model Operations

### When Operation Involves Multiple Models:
```python
class Session(db.Model):
    def add_speaker(self, user, role=SessionSpeakerRole.SPEAKER):
        """Add speaker to session AND update event role"""
        # Add to session
        session_speaker = SessionSpeaker(
            session_id=self.id,
            user_id=user.id,
            role=role
        )
        
        # Update event role if needed
        event_user = EventUser.query.filter_by(
            event_id=self.event_id,
            user_id=user.id
        ).first()
        
        if not event_user:
            event_user = EventUser(
                event_id=self.event_id,
                user_id=user.id,
                role=EventUserRole.SPEAKER
            )
            db.session.add(event_user)
```

## Best Practices

1. **Ask "Who Owns This?"**
```python
# User owns their permissions
user.can_edit_event(event_id)  # ✅
event.can_user_edit(user_id)   # ❌

# Event owns its schedule
event.get_schedule()           # ✅
user.get_event_schedule(id)    # ❌
```

2. **Consider the Primary Actor**
```python
# User is primary actor
user.register_for_event(event)     # ✅
event.register_user(user)          # ❌

# Event is primary actor
event.cancel_session(session)      # ✅
session.cancel_in_event(event)     # ❌
```

3. **Think About Natural Language**
```python
# These read naturally
user.get_upcoming_events()
event.get_speakers()
session.is_full()

# These don't
event.get_user_upcoming()
user.get_event_speakers(event_id)
user.is_session_full(session_id)
```

Remember:
- Methods go where they're most naturally accessed
- Consider who "owns" the operation
- Think about where the data lives
- Consider query efficiency
- Keep related operations together

You got it! I'll start updating with the first section and if that format looks good, I'll continue with the rest. Here's how I'd do it:

## [[Service-Level Operations]]

### When Operation is Complex/Spans Multiple Models
```python
# services/registration_service.py
class RegistrationService:
    """Handle complex registration logic"""
    def register_speaker_for_session(self, user, session):
        # First check if user has any time conflicts
        if not user.is_available(session.start_time, session.end_time):
            raise ConflictError("Speaker has schedule conflict")
            
        # Verify session isn't already full
        if session.is_at_speaker_capacity():
            raise CapacityError("Session full")
            
        # Ensure user has proper event-level role
        event = session.event
        if not user.get_event_role(event.id):
            # Add as speaker if no existing role
            event.add_user(user, EventUserRole.SPEAKER)
            
        # Finally, add to session
        session.add_speaker(user)
```
This service pattern is useful when an operation:
1. Affects multiple models
2. Requires complex validation
3. Needs transaction management
4. Doesn't naturally belong to any single model

## [[State-Based Operations]]

### When Behavior Depends on State
```python
class Session(db.Model):
    def can_modify(self) -> bool:
        """Different states allow different operations"""
        # Completed sessions are locked
        if self.status == SessionStatus.COMPLETED:
            return False
        # Can't modify while session is live
        if self.status == SessionStatus.LIVE:
            return False
        return True

    def transition_to(self, new_status: SessionStatus):
        """Handle state transitions with validation"""
        # Define valid state transitions
        valid_transitions = {
            SessionStatus.SCHEDULED: [SessionStatus.LIVE, SessionStatus.CANCELLED],
            SessionStatus.LIVE: [SessionStatus.COMPLETED],
            SessionStatus.COMPLETED: [],  # Terminal state - no transitions out
        }
        
        # Validate the transition is allowed
        if new_status not in valid_transitions[self.status]:
            raise InvalidTransitionError(
                f"Cannot transition from {self.status} to {new_status}"
            )
            
        self.status = new_status
```
State management is crucial for maintaining data integrity. This pattern:
- Enforces valid state transitions
- Prevents invalid modifications
- Makes state logic explicit and centralized
- Easy to audit and modify state rules

# [[Complex Relationships]]
## Relationship-Based Operations

### When Dealing with Complex Relationships
```python
class Organization(db.Model):
    def transfer_ownership(self, from_user, to_user):
        """Complex ownership transfer with role management"""
        # Verify current ownership
        if not self.is_owner(from_user):
            raise PermissionError("User is not owner")
            
        # Use nested transaction for atomicity
        db.session.begin_nested()
        try:
            # Remove owner role from current owner
            self.remove_user_role(from_user, OrganizationUserRole.OWNER)
            # Assign owner role to new user
            self.add_user_role(to_user, OrganizationUserRole.OWNER)
            # Demote previous owner to admin
            self.add_user_role(from_user, OrganizationUserRole.ADMIN)
            db.session.commit()
        except:
            # Rollback on any error
            db.session.rollback()
            raise

    def get_role_hierarchy(self, user):
        """Get highest role when user might have multiple"""
        # Get all roles for user in this org
        roles = OrganizationUser.query.filter_by(
            organization_id=self.id,
            user_id=user.id
        ).all()
        
        # Define role hierarchy (higher number = higher rank)
        hierarchy = {
            OrganizationUserRole.OWNER: 3,
            OrganizationUserRole.ADMIN: 2,
            OrganizationUserRole.MEMBER: 1
        }
        
        # Return highest ranked role
        return max(roles, key=lambda r: hierarchy[r.role])
```
Complex relationships require careful management. This approach:
- Maintains data consistency
- Handles edge cases
- Uses transactions for safety
- Makes hierarchy explicit

## Aggregate Operations

### When Operation Affects Multiple Records
```python
class Event(db.Model):
    def reschedule(self, new_start_date, new_end_date):
        """Reschedule event and all related sessions"""
        # Calculate duration to ensure it's maintained
        duration = self.end_date - self.start_date
        new_duration = new_end_date - new_start_date
        
        # Validate duration hasn't changed
        if duration != new_duration:
            raise ValueError("New schedule must maintain same duration")
            
        # Calculate time difference for shifting sessions
        time_diff = new_start_date - self.start_date
        
        # Update all session times proportionally
        for session in self.sessions:
            session.start_time += time_diff
            session.end_time += time_diff
            
        # Update event dates
        self.start_date = new_start_date
        self.end_date = new_end_date
```
Aggregate operations maintain consistency across related records. Benefits:
- Atomic updates
- Maintains data relationships
- Ensures business rules
- Single point of modification

## Validation Operations

### When Complex Validation is Needed
```python
class Session(db.Model):
    def validate_speaker_assignment(self, user):
        """Comprehensive speaker validation"""
        # Check for scheduling conflicts
        if not user.is_available(self.start_time, self.end_time):
            raise ValidationError("Speaker has time conflict")
            
        # Enforce daily session limit
        sessions_that_day = user.get_speaking_sessions_for_date(
            self.start_time.date()
        )
        if len(sessions_that_day) >= 3:
            raise ValidationError("Speaker exceeded daily session limit")
            
        # Verify required certifications
        if self.requires_certification:
            if not user.has_certification(self.required_cert_type):
                raise ValidationError("Speaker missing required certification")
```
Complex validation ensures data integrity and business rules. This approach:
- Centralizes validation logic
- Makes rules explicit
- Easy to modify requirements
- Clear error messages

## [[Query Optimization Operations]]

### When Complex Queries Need Encapsulation
```python
class Event(db.Model):
    def get_session_stats(self):
        """Optimized aggregation query for session statistics"""
        return db.session.query(
            Session.status,
            db.func.count(Session.id).label('count'),
            # Convert duration to seconds for averaging
            db.func.avg(
                db.func.extract('epoch', Session.end_time - Session.start_time)
            ).label('avg_duration')
        ).filter(
            Session.event_id == self.id
        ).group_by(
            Session.status
        ).all()

    def get_speakers_with_sessions(self):
        """Efficiently load speakers and their sessions"""
        return User.query.join(
            SessionSpeaker
        ).options(
            # Eager load relationships to prevent N+1 queries
            db.joinedload(User.speaking_sessions)
        ).filter(
            SessionSpeaker.session_id.in_(
                Session.query.filter_by(event_id=self.id).with_entities(Session.id)
            )
        ).all()
```
Query optimization is crucial for performance. These patterns:
- Reduce database queries
- Optimize data loading
- Encapsulate complex queries
- Make performance intentions clear

## Utility Operations - [[Mixins and Shared Functionality]]

### When Operation is Shared Across Models
```python
# utils/model_utils.py
class TimeRangeMixin:
    """Shared functionality for models with time ranges"""
    def overlaps_with(self, start_time, end_time):
        """Check if time range overlaps with given range"""
        return (
            self.start_time < end_time and
            self.end_time > start_time
        )
        
    def contains_time(self, timestamp):
        """Check if timestamp falls within range"""
        return self.start_time <= timestamp <= self.end_time
        
    @property
    def duration_minutes(self):
        """Calculate duration in minutes"""
        return (self.end_time - self.start_time).total_seconds() / 60

# Use in models
class Session(db.Model, TimeRangeMixin):
    pass

class Event(db.Model, TimeRangeMixin):
    pass
```
Mixins provide reusable functionality across models. Benefits:
- DRY (Don't Repeat Yourself)
- Consistent behavior
- Easy to maintain
- Clear inheritance

