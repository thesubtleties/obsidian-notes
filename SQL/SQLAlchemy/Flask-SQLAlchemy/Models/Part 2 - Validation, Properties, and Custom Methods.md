
## Property Decorators and Validation
```python
class Sprint(db.Model):
    __tablename__ = "sprints"
    
    # Use underscore for "private" fields that will have properties
    _start_date = db.Column('start_date', db.Date)
    _end_date = db.Column('end_date', db.Date)

    # Simple property (getter)
    @property
    def start_date(self):
        return self._start_date

    # Property with setter and validation
    @start_date.setter
    def start_date(self, value):
        if self._end_date and value and value > self._end_date:
            raise ValueError("Start date can't be after end date")
        self._start_date = value

    # Computed property (no database column needed)
    @property
    def duration(self):
        if self._start_date and self._end_date:
            difference = self._end_date - self._start_date
            return difference.days
        return None
```
Properties allow us to add logic when getting or setting values. The underscore prefix (_start_date) is a Python convention indicating these are "private" variables that should be accessed through properties. This lets us:
- Validate data before saving (like ensuring start_date is before end_date)
- Compute values on the fly (like duration)
- Control how data is accessed and modified
- Add business logic without changing the database schema

## Field Validation Using @validates
```python
from sqlalchemy.orm import validates

class Feature(db.Model):
    __tablename__ = "features"
    
    status = db.Column(db.String, default="Not Started")
    priority = db.Column(db.Integer, default=0)

    # Define valid options
    VALID_STATUSES = ["Not Started", "In Progress", "Completed"]

    # Validate status field
    @validates("status")
    def validate_status(self, key, status):
        if status not in self.VALID_STATUSES:
            raise ValueError(f"Status must be one of: {', '.join(self.VALID_STATUSES)}")
        return status

    # Validate multiple fields
    @validates("priority")
    def validate_priority(self, key, value):
        if not isinstance(value, int) or value < 0:
            raise ValueError("Priority must be a non-negative integer")
        return value
```
The @validates decorator provides automatic validation whenever a field is set. This happens:
- During object creation
- During updates
- Any time the field is modified
This ensures data consistency and moves validation logic into the model where it belongs, rather than spreading it across different routes.

## CRUD Class Methods and Instance Methods
```python
class Project(db.Model):
    __tablename__ = "projects"

    # Class method for creation
    @classmethod
    def create_project(cls, **kwargs):
        """Use class method when creating new instances"""
        project = cls(**kwargs)
        db.session.add(project)
        db.session.commit()
        return project

    # Instance method for updates
    def update_project(self, **kwargs):
        """Use instance method when working with existing instance"""
        try:
            for key, value in kwargs.items():
                setattr(self, key, value)
            db.session.commit()
            return self
        except Exception as e:
            db.session.rollback()
            raise e

    # Class method for queries
    @classmethod
    def get_by_owner(cls, owner_id):
        """Class methods for common queries"""
        return cls.query.filter_by(owner_id=owner_id).all()

    # Instance method for specific actions
    def add_member(self, user):
        """Instance methods for specific business logic"""
        if user not in self.users:
            self.users.append(user)
            db.session.commit()
```
This shows the difference between class methods (@classmethod) and instance methods:
- Class methods (create_project, get_by_owner) operate on the class itself
  - Good for creating new instances
  - Good for database queries
  - Don't have access to instance attributes
- Instance methods (update_project, add_member) operate on specific instances
  - Good for updating existing records
  - Can access instance attributes (self.users)
  - Handle specific business logic

The distinction helps organize code and makes it clear whether you're working with the class as a whole or a specific instance.

