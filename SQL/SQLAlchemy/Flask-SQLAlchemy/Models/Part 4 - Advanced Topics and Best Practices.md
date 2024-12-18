## Custom Model Events and Hooks
```python
from sqlalchemy import event

class Project(db.Model):
    __tablename__ = 'projects'
    status = db.Column(db.String, default='active')
    
    # Lifecycle hooks using SQLAlchemy events
    @staticmethod
    def before_update(mapper, connection, target):
        """Runs before any update"""
        target.updated_at = datetime.utcnow()

    @staticmethod
    def after_update(mapper, connection, target):
        """Runs after any update"""
        if target.status == 'completed':
            # Could trigger notifications, updates, etc.
            pass

# Register the event listeners
event.listen(Project, 'before_update', Project.before_update)
event.listen(Project, 'after_update', Project.after_update)
```
SQLAlchemy events let you:
- Automatically update fields
- Trigger side effects
- Implement business logic
- Handle model lifecycle events
- Keep logic close to the model

## Advanced Query Methods and Filters
```python
class Task(db.Model):
    __tablename__ = 'tasks'
    status = db.Column(db.String)
    due_date = db.Column(db.DateTime)

    # Query property for common filters
    @property
    def is_overdue(self):
        return self.due_date < datetime.utcnow() and self.status != 'completed'

    # Class-level query methods
    @classmethod
    def get_overdue(cls):
        return cls.query\
            .filter(cls.due_date < datetime.utcnow())\
            .filter(cls.status != 'completed')\
            .order_by(cls.due_date)\
            .all()

    @classmethod
    def get_dashboard_stats(cls):
        """Complex aggregation query"""
        from sqlalchemy import func
        return db.session.query(
            cls.status,
            func.count(cls.id).label('count'),
            func.avg(cls.priority).label('avg_priority')
        ).group_by(cls.status).all()
```
Advanced queries help with:
- Complex filtering logic
- Aggregations and statistics
- Common query patterns
- Performance optimization
- Business intelligence

## Hybrid Properties and Methods
```python
from sqlalchemy.ext.hybrid import hybrid_property, hybrid_method

class User(db.Model):
    __tablename__ = 'users'
    first_name = db.Column(db.String)
    last_name = db.Column(db.String)

    @hybrid_property
    def full_name(self):
        """Works both on instance and in queries"""
        return f"{self.first_name} {self.last_name}"

    @hybrid_method
    def has_access_level(self, level):
        """Can be used in queries"""
        return self.access_level >= level

    # Usage in queries:
    # User.query.filter(User.full_name.like('%John%'))
    # User.query.filter(User.has_access_level(5))
```
Hybrid properties/methods:
- Work both on instances and in queries
- Allow complex conditions in filters
- Combine multiple fields
- Keep logic consistent
- Improve query flexibility

## Performance Optimization Patterns
```python
class Feature(db.Model):
    __tablename__ = 'features'

    # Add indexes for frequently queried fields
    __table_args__ = (
        db.Index('idx_feature_status', 'status'),
        db.Index('idx_feature_project', 'project_id'),
        {'schema': SCHEMA} if environment == "production" else None
    )

    # Lazy loading options for relationships
    tasks = db.relationship(
        'Task',
        back_populates='feature',
        lazy='dynamic'  # Returns query object instead of loading all
    )

    # Query optimization methods
    @classmethod
    def get_with_related(cls, feature_id):
        """Optimized query loading related data"""
        return cls.query\
            .options(
                db.joinedload(cls.project),
                db.joinedload(cls.tasks)
            )\
            .get(feature_id)
```
Performance considerations:
- Add indexes for searched/filtered fields
- Use appropriate lazy loading strategies
- Optimize queries with joins
- Avoid N+1 query problems
- Consider data access patterns

## Model Versioning and History
```python
class VersionMixin:
    """Track changes to model"""
    version = db.Column(db.Integer, default=1)
    
    def create_version(self):
        """Save current state as history"""
        history = self.to_dict()
        history['version'] = self.version
        history['main_id'] = self.id
        
        # Save to history table
        version_obj = self.HistoryModel(**history)
        db.session.add(version_obj)
        
        # Update version
        self.version += 1
        db.session.commit()

class Project(db.Model, VersionMixin):
    # Define corresponding history model
    class HistoryModel(db.Model):
        __tablename__ = 'project_history'
        id = db.Column(db.Integer, primary_key=True)
        main_id = db.Column(db.Integer)
        version = db.Column(db.Integer)
        # ... other fields ...
```
Versioning helps:
- Track changes over time
- Maintain audit trails
- Support undo/redo
- Meet compliance requirements
- Debug issues
