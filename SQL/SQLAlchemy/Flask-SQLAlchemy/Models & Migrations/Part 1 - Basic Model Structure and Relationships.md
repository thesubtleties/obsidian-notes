## Basic Model Structure
```python
from .db import db  # Your SQLAlchemy instance
from datetime import datetime

class Example(db.Model):
    __tablename__ = 'examples'  # Explicitly declare table name
    
    # Production setup (if using services like Render)
    if environment == "production":
        __table_args__ = {'schema': SCHEMA}

    # Basic Fields
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)  # Required field
    description = db.Column(db.Text)                 # Optional field
    is_active = db.Column(db.Boolean, default=True)  # With default
    
    # Timestamps
    created_at = db.Column(
        db.DateTime, 
        default=datetime.utcnow  # Note: no parentheses - passing function reference
    )
    updated_at = db.Column(
        db.DateTime,
        default=datetime.utcnow,
        onupdate=datetime.utcnow  # Automatically updates on any change
    )

    def to_dict(self):
        """Convert model to dictionary for API responses"""
        return {
            'id': self.id,
            'name': self.name,
            'description': self.description,
            'is_active': self.is_active,
            'created_at': self.created_at.isoformat(),
            'updated_at': self.updated_at.isoformat()
        }
```
This shows the basic structure of a SQLAlchemy model. Key points:
- `__tablename__` explicitly names your table (don't rely on automatic naming)
- Production schema setup is needed for services like Render
- Different column types (Integer, String, Text, Boolean) for different needs
- `nullable=False` makes fields required
- `default` provides fallback values
- Timestamps use function references (not function calls) for defaults
- `to_dict()` method standardizes API responses

## One-to-Many Relationships
```python
# Parent Model (One)
class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    
    # One user has many posts
    posts = db.relationship('Post', back_populates='user')

# Child Model (Many)
class Post(db.Model):
    __tablename__ = 'posts'
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'), nullable=False)
    
    # Each post belongs to one user
    user = db.relationship('User', back_populates='posts')
```
One-to-Many is the most common relationship type. Notice:
- Foreign key goes on the "Many" side (Post has user_id)
- Relationship defined on both sides with `back_populates`
- Parent uses plural name (posts), child uses singular (user)
- `nullable=False` means every post must have a user

## Many-to-Many Relationships
```python
# Association Table (no model needed for simple relationships)
project_users = db.Table('project_users',
    db.Column('project_id', db.Integer, db.ForeignKey('projects.id'), primary_key=True),
    db.Column('user_id', db.Integer, db.ForeignKey('users.id'), primary_key=True)
)

class Project(db.Model):
    __tablename__ = 'projects'
    id = db.Column(db.Integer, primary_key=True)
    
    # Many projects can have many users
    users = db.relationship(
        'User', 
        secondary=project_users,  # References association table
        back_populates='projects'
    )

class User(db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    
    # Many users can have many projects
    projects = db.relationship(
        'Project', 
        secondary=project_users,  # Same association table
        back_populates='users'
    )
```
Many-to-Many relationships need an association table:
- Association table has no model if it's just connecting tables
- Both foreign keys are primary keys in association table
- Both models reference same association table via `secondary`
- Use `back_populates` to create two-way relationship
- If association table needs additional fields, make it a model instead

### Why back_populates vs backref?
```python
# Using back_populates (more explicit)
class User(db.Model):
    posts = db.relationship('Post', back_populates='user')

class Post(db.Model):
    user = db.relationship('User', back_populates='posts')

# Using backref (more magical)
class User(db.Model):
    posts = db.relationship('Post', backref='user')

# Post model doesn't need to define relationship
```
- `back_populates` is more explicit - you see relationships in both models
- `backref` is more concise but hides relationship in other model
- Prefer `back_populates` for clarity and maintainability
- Use `backref` for simple, obvious relationships or prototyping

