## N+1 Query Problem
```python
class Project(db.Model):
    # BAD: Will cause N+1 queries
    def bad_get_all_with_features(cls):
        projects = Project.query.all()
        return [{
            **project.to_dict(),
            'features': [f.to_dict() for f in project.features]  # One query per project!
        } for project in projects]

    # GOOD: Uses join to avoid N+1
    @classmethod
    def get_all_with_features(cls):
        """Efficient loading of related data"""
        return cls.query\
            .options(db.joinedload(cls.features))\
            .all()

    # BETTER: Selective loading with filtering
    @classmethod
    def get_filtered_with_features(cls, **filters):
        """Efficient filtered query with relationships"""
        query = cls.query.options(db.joinedload(cls.features))
        
        # Add any filters
        for key, value in filters.items():
            if hasattr(cls, key):
                query = query.filter(getattr(cls, key) == value)
        
        return query.all()
```
N+1 solutions:
- Use joinedload for relationships
- Batch load related data
- Plan queries efficiently
- Monitor query counts
- Use proper indexing

## Circular Dependencies
```python
# BAD: Direct circular import
# feature.py
from .task import Task  # Circular!

# GOOD: Use strings for relationships
class Feature(db.Model):
    tasks = db.relationship('Task', back_populates='feature')

# BETTER: Resolve circular dependencies with events
from sqlalchemy import event

class Feature(db.Model):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._tasks = []

    @property
    def tasks(self):
        return self._tasks

# Set up relationship after all models are defined
def setup_relationships():
    from .task import Task
    Feature.tasks = db.relationship('Task', back_populates='feature')

event.listen(db.Model.metadata, 'after_configure', setup_relationships)
```
Handling circular dependencies:
- Use string references
- Lazy imports
- Event-based setup
- Proper module organization
- Clear dependency hierarchy

## Memory Management
```python
class Project(db.Model):
    @classmethod
    def process_large_dataset(cls):
        """Handle large datasets efficiently"""
        # BAD: Loads everything into memory
        # projects = cls.query.all()
        
        # GOOD: Use yield to process in chunks
        def process_in_chunks(chunk_size=1000):
            query = cls.query
            for offset in range(0, query.count(), chunk_size):
                chunk = query.limit(chunk_size).offset(offset).all()
                yield from chunk
                db.session.expunge_all()  # Clear session

        # BETTER: Use native database features
        def stream_results():
            return cls.query\
                .yield_per(1000)\
                .enable_eagerloads(False)\
                .stream()

    @classmethod
    def bulk_update_status(cls, project_ids, new_status):
        """Efficient bulk updates"""
        cls.query\
            .filter(cls.id.in_(project_ids))\
            .update(
                {cls.status: new_status},
                synchronize_session=False
            )
        db.session.commit()
```
Memory optimization techniques:
- Process in chunks
- Use generators
- Stream results
- Clear session regularly
- Bulk operations

## Transaction Handling
```python
from contextlib import contextmanager
from sqlalchemy.exc import IntegrityError

@contextmanager
def transaction_scope():
    """Transaction context manager"""
    try:
        yield
        db.session.commit()
    except Exception:
        db.session.rollback()
        raise

class Feature(db.Model):
    @classmethod
    def safe_bulk_create(cls, features_data):
        """Safe bulk creation with transaction"""
        with transaction_scope():
            features = []
            for data in features_data:
                feature = cls(**data)
                features.append(feature)
                db.session.add(feature)
            return features

    def update_with_tasks(self, feature_data, tasks_data):
        """Complex update involving multiple models"""
        try:
            # Start transaction
            self.name = feature_data['name']
            self.description = feature_data['description']
            
            # Create associated tasks
            from .task import Task
            for task_data in tasks_data:
                task = Task(feature_id=self.id, **task_data)
                db.session.add(task)
            
            db.session.commit()
            return True
            
        except IntegrityError:
            db.session.rollback()
            raise ValueError("Database integrity error")
            
        except Exception as e:
            db.session.rollback()
            raise ValueError(f"Update failed: {str(e)}")
```
Transaction best practices:
- Use context managers
- Handle rollbacks properly
- Catch specific exceptions
- Maintain data integrity
- Clear error messages

## SQLAlchemy Session Management
```python
class SessionMixin:
    """Mixin for session management helpers"""
    
    @classmethod
    def bulk_save(cls, objects):
        """Safe bulk save with session management"""
        try:
            db.session.bulk_save_objects(objects)
            db.session.commit()
        except Exception:
            db.session.rollback()
            raise

    @classmethod
    def refresh_object(cls, obj):
        """Refresh object from database"""
        db.session.refresh(obj)
        return obj

    @classmethod
    def merge_or_create(cls, **kwargs):
        """Merge existing or create new"""
        instance = db.session.merge(cls(**kwargs))
        db.session.commit()
        return instance

class Project(db.Model, SessionMixin):
    @classmethod
    def get_or_create(cls, **kwargs):
        """Get existing or create new"""
        instance = cls.query.filter_by(**kwargs).first()
        if not instance:
            instance = cls(**kwargs)
            db.session.add(instance)
            db.session.commit()
        return instance
```
Session management considerations:
- Proper session lifecycle
- Efficient bulk operations
- Object state management
- Merge strategies
- Refresh policies

