## Model Methods for Common Operations
```python
class Project(db.Model):
    __tablename__ = 'projects'

    # Flexible query method with multiple filters
    @classmethod
    def get_filtered_projects(cls, **kwargs):
        """
        Usage: Project.get_filtered_projects(status="active", owner_id=1)
        """
        query = cls.query
        for key, value in kwargs.items():
            if hasattr(cls, key):
                query = query.filter(getattr(cls, key) == value)
        return query.all()

    # Bulk operations with error handling
    @classmethod
    def bulk_create(cls, projects_data):
        """
        Safely create multiple projects at once
        """
        try:
            projects = [cls(**data) for data in projects_data]
            db.session.add_all(projects)
            db.session.commit()
            return projects
        except Exception as e:
            db.session.rollback()
            raise ValueError(f"Bulk create failed: {str(e)}")
```
These utility methods make common operations easier:
- Flexible filtering method can handle any combination of filters
- Bulk operations handle multiple records efficiently
- Error handling ensures database consistency
- Class methods keep related functionality in the model

## Advanced Relationships and Queries
```python
class User(db.Model):
    __tablename__ = 'users'

    # Get all tasks (both created and assigned)
    @property
    def all_tasks(self):
        """Combines tasks created by and assigned to user"""
        created = set(self.created_tasks)
        assigned = set(self.assigned_tasks)
        return list(created | assigned)

    # Complex query method
    @classmethod
    def get_active_project_leads(cls):
        """Gets users who own active projects"""
        return cls.query\
            .join(Project, Project.owner_id == cls.id)\
            .filter(Project.status == 'active')\
            .distinct()\
            .all()
```
Advanced queries and relationship handling:
- Properties can combine related data
- Complex queries using joins and filters
- Use distinct() to avoid duplicates
- Keep complex query logic in model methods

## Mixins and Common Functionality
```python
class TimestampMixin:
    """Add to any model that needs timestamps"""
    created_at = db.Column(
        db.DateTime, 
        default=datetime.utcnow
    )
    updated_at = db.Column(
        db.DateTime,
        default=datetime.utcnow,
        onupdate=datetime.utcnow
    )

class SearchMixin:
    """Add search functionality to any model"""
    @classmethod
    def search(cls, term):
        if hasattr(cls, 'name'):
            return cls.query.filter(cls.name.ilike(f'%{term}%')).all()
        raise NotImplementedError("Model must have 'name' field")

class Project(db.Model, TimestampMixin, SearchMixin):
    __tablename__ = 'projects'
    # Now has timestamps and search automatically
```
Mixins help reduce code duplication:
- Share common functionality across models
- Keep code DRY (Don't Repeat Yourself)
- Easy to add features to multiple models
- Maintain consistency across models

## Error Handling and Validation Patterns
```python
from sqlalchemy.exc import IntegrityError

class Feature(db.Model):
    # ... other model code ...

    def safe_delete(self):
        """Delete with dependency checking"""
        try:
            if self.tasks:
                raise ValueError("Cannot delete feature with existing tasks")
            db.session.delete(self)
            db.session.commit()
            return True
        except IntegrityError:
            db.session.rollback()
            raise ValueError("Database integrity error")

    @classmethod
    def create_with_validation(cls, **kwargs):
        """Create with extra validation"""
        try:
            # Custom validation
            if kwargs.get('priority', 0) > 5:
                raise ValueError("Priority cannot exceed 5")
            
            # Check related objects exist
            project = Project.query.get(kwargs.get('project_id'))
            if not project:
                raise ValueError("Project does not exist")

            feature = cls(**kwargs)
            db.session.add(feature)
            db.session.commit()
            return feature
        except Exception as e:
            db.session.rollback()
            raise ValueError(f"Creation failed: {str(e)}")
```
Robust error handling patterns:
- Check dependencies before deletions
- Validate related objects exist
- Roll back transactions on error
- Provide clear error messages
- Handle database integrity errors

