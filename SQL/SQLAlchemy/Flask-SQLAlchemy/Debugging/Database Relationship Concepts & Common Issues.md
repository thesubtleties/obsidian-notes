## Model Relationships & Naming
### One-to-Many Linkage Issues
**Symptoms:**
- Error: "Mapper has no property 'projects'"
- Relationships return None unexpectedly
- Can't access related items through relationship

**Common Mistakes:**
```python
# Wrong: Mismatched plurality
class Project(db.Model):
    feature = db.relationship("Feature", back_populates="projects")  # Singular when should be plural

# Wrong: Inconsistent back_populates
class Feature(db.Model):
    project = db.relationship("Project", back_populates="feature")  # Doesn't match other side
```

**Correct Pattern:**
```python
class Project(db.Model):
    features = db.relationship("Feature", back_populates="project")  # One has many (plural)

class Feature(db.Model):
    project_id = db.Column(db.Integer, db.ForeignKey("projects.id"))
    project = db.relationship("Project", back_populates="features")  # Many belongs to one (singular)
```

**Prevention:**
- One side uses plural (features)
- Many side uses singular (project)
- back_populates names match exactly
- Foreign key on "Many" side

### Many-to-Many Association Issues
**Symptoms:**
- Error: "Table 'project_users' doesn't exist"
- Can't add users to project.users
- Relationship returns empty list unexpectedly

**Common Mistakes:**
```python
# Wrong: Association table defined after models
class Project(db.Model):
    users = db.relationship('User', secondary=project_users)  # project_users not defined yet!

# Then defining association table
project_users = db.Table(...)  # Too late!
```

**Correct Pattern:**
```python
# Define association table first
project_users = db.Table('project_users',
    db.Column('project_id', db.Integer, db.ForeignKey('projects.id')),
    db.Column('user_id', db.Integer, db.ForeignKey('users.id'))
)

# Then use in models
class Project(db.Model):
    users = db.relationship('User', secondary=project_users, back_populates='projects')
```

# Date/Time & Property Handling

## DateTime Issues
**Symptoms:**
- All records have same timestamp
- Timestamps not updating
- TypeError about datetime callable

**Common Mistakes:**
```python
# Wrong: Function called at model definition
created_at = db.Column(db.DateTime, default=datetime.utcnow())  # () calls it immediately!

# Wrong: Updated_at not updating
updated_at = db.Column(db.DateTime, default=datetime.utcnow)  # Missing onupdate
```

**Correct Pattern:**
```python
class Model(db.Model):
    created_at = db.Column(db.DateTime, default=datetime.utcnow)  # No ()
    updated_at = db.Column(
        db.DateTime, 
        default=datetime.utcnow,
        onupdate=datetime.utcnow  # Updates on any model change
    )
```

## Protected Properties (_private fields)
**Symptoms:**
- Validation not running
- Dates storing incorrectly
- Can't access _private fields directly

**Common Mistakes:**
```python
# Wrong: Accessing _private field directly
class Sprint(db.Model):
    _start_date = db.Column(db.DateTime)
    
    def set_dates(self):
        self._start_date = some_date  # Bypasses validation!
```

**Correct Pattern:**
```python
class Sprint(db.Model):
    _start_date = db.Column('start_date', db.DateTime)  # Note DB column name

    @property
    def start_date(self):
        return self._start_date

    @start_date.setter
    def start_date(self, value):
        # Validation happens here
        if self._end_date and value > self._end_date:
            raise ValueError("Start date can't be after end date")
        self._start_date = value
```

# Validation & Data Integrity

## Status/Enum Validation
**Symptoms:**
- Invalid statuses in database
- No error when setting invalid status
- Error: "validates is not defined"

**Common Mistakes:**
```python
# Wrong: Missing validator import
@validates("status")  # ImportError: validates not defined
def validate_status(self, key, status):
    pass

# Wrong: Validation bypass
feature.status = "Invalid Status"  # Should not allow this!
```

**Correct Pattern:**
```python
from sqlalchemy.orm import validates

class Feature(db.Model):
    VALID_STATUSES = ["Not Started", "In Progress", "Completed"]
    
    @validates("status")
    def validate_status(self, key, status):
        if status not in self.VALID_STATUSES:
            raise ValueError(f"Status must be one of: {', '.join(self.VALID_STATUSES)}")
        return status
```

# Database Session & Transaction Management

## Session State Issues
**Symptoms:**
- "Object already attached to session"
- "Instance is not bound to a Session"
- Changes not saving to database

**Common Mistakes:**
```python
# Wrong: Not handling transaction errors
def update_something(self):
    self.name = new_name
    # No commit! Changes lost
    return self

# Wrong: No rollback on error
def create_thing():
    thing = Thing()
    db.session.add(thing)
    # Error occurs, no rollback, session left in bad state
```

**Correct Pattern:**
```python
def update_feature(self, **kwargs):
    try:
        for key, value in kwargs.items():
            setattr(self, key, value)
        db.session.commit()
        return self
    except Exception as e:
        db.session.rollback()
        raise e  # Re-raise for route handler
```

# Seeding & Data Dependencies

## Dependency Order Issues
**Symptoms:**
- Error: "'NoneType' object has no attribute 'extend'"
- Error: "Cannot add or update a child row: a foreign key constraint fails"
- Relationships empty when they shouldn't be

**Common Mistakes:**
```python
# Wrong: Adding related data before parent exists
def seed_projects():
    website = Project(owner_id=1)  # User 1 doesn't exist yet!
    db.session.add(website)

# Wrong: Trying to establish relationships with non-existent data
def seed_features():
    feature.sprint = Sprint.query.get(1)  # Sprint might not exist
```

**Correct Pattern:**
```python
# In seeds/__init__.py
@seed_commands.command("all")
def seed():
    # Clear in reverse dependency order
    if environment == "production":
        undo_tasks()      # Most dependent first
        undo_features()
        undo_sprints()
        undo_projects()
        undo_users()      # Least dependent last

    # Seed in dependency order
    seed_users()          # Base tables first
    seed_projects()       # Then tables that need users
    seed_sprints()        # Then tables that need projects
    seed_features()       # Then tables that need sprints
    seed_tasks()          # Most dependent last
```

## Relationship Verification in Seeds
**Symptoms:**
- Silent failures in relationships
- Missing data in related tables
- Integrity errors in production

**Common Mistakes:**
```python
# Wrong: Assuming data exists
def seed_projects():
    demo = User.query.first()  # What if no users exist?
    project = Project(owner_id=demo.id)

# Wrong: Not verifying relationship data
website_redesign.users.extend([demo, sarah])  # demo or sarah might be None
```

**Correct Pattern:**
```python
def seed_projects():
    # Verify required data exists
    demo = User.query.filter_by(username="Demo").first()
    sarah = User.query.filter_by(username="sarah").first()
    
    if not all([demo, sarah]):
        raise Exception("Required users not found. Run seed_users first.")

    try:
        website_redesign = Project(
            name="Website Redesign",
            owner_id=demo.id
        )
        # Only extend if both users exist
        website_redesign.users.extend([demo, sarah])
        
        db.session.add(website_redesign)
        db.session.commit()
    except Exception as e:
        db.session.rollback()
        raise e
```

## Circular Dependencies in Seeds
**Symptoms:**
- ImportError: cannot import name
- Circular import detected
- Models not found when seeding

**Common Mistakes:**
```python
# Wrong: Circular imports at top level
# seeds/projects.py
from .users import seed_users  # Circular if users.py imports from projects

# Wrong: Importing models in wrong order
from .feature import Feature  # Might depend on Project
from .project import Project  # Might depend on Feature
```

**Correct Pattern:**
```python
# Import models where needed
def seed_projects():
    from app.models import User, Project  # Import inside function
    
    # Now use models...

# Or use dependency injection
def seed_projects(User=None, Project=None):
    if User is None:
        from app.models import User
    if Project is None:
        from app.models import Project
```

## Data Integrity in Seeds
**Symptoms:**
- Inconsistent test data
- Missing required relationships
- Validation errors in seeded data

**Common Mistakes:**
```python
# Wrong: Not handling required fields
def seed_tasks():
    task = Task(
        name="Design Homepage"
        # Missing required feature_id!
    )

# Wrong: Inconsistent data across related models
sprint = Sprint(
    start_date=future_date,
    end_date=past_date  # Violates date validation
)
```

**Correct Pattern:**
```python
def seed_tasks():
    # Prepare all required data first
    feature = Feature.query.first()
    if not feature:
        raise Exception("No features exist for task assignment")

    # Use constants or utilities for consistent data
    from datetime import datetime, timedelta
    start = datetime.now()
    end = start + timedelta(days=14)

    task = Task(
        feature_id=feature.id,  # Required relationship
        name="Design Homepage",
        start_date=start,
        due_date=end,          # Maintains date validation
        status="Not Started"   # Valid status
    )

    try:
        db.session.add(task)
        db.session.commit()
    except Exception as e:
        db.session.rollback()
        print(f"Failed to seed task: {str(e)}")
        raise e
```

# Docker & Environment Setup Issues

## Container Access & Command Issues
**Symptoms:**
- "bash: command not found"
- Can't access container
- Commands not working as expected

**Common Mistakes:**
```bash
# Wrong: Using bash with Alpine
docker exec -it backend bash  # Error: bash not found

# Wrong: Running Flask commands outside container
flask db migrate  # Won't work on host machine
```

**Correct Pattern:**
```bash
# Use sh for Alpine-based containers
docker exec -it backend sh

# Then inside container
flask db downgrade base
rm -rf migrations/
flask db init
flask db migrate -m "initial migration"
flask db upgrade
```

## Database Connection & Migration Issues
**Symptoms:**
- "relation does not exist"
- Can't connect to database
- Migrations not applying
- Seeds failing because tables don't exist

**Common Mistakes:**
```python
# Wrong: Not waiting for database
# In init.sh
flask db upgrade  # Database might not be ready!

# Wrong: Running migrations before database is up
flask seed all  # Tables don't exist yet!
```

**Correct Pattern:**
```bash
# In init.sh
#!/bin/sh
set -e

echo "Waiting for postgres..."
while ! nc -z db 5432; do
  echo "Postgres is unavailable - sleeping"
  sleep 1
done
echo "PostgreSQL started successfully!"

# Now safe to run migrations
flask db upgrade
flask seed all
```

## Environment & Path Issues
**Symptoms:**
- ModuleNotFoundError: No module named 'app'
- Import errors in Docker
- Can't find modules

**Common Mistakes:**
```dockerfile
# Wrong: Missing PYTHONPATH
WORKDIR /app
# No PYTHONPATH set

# Wrong: Incorrect import paths
from app.models import db  # When PYTHONPATH is /app
```

**Correct Pattern:**
```dockerfile
# Dockerfile
WORKDIR /app
ENV PYTHONPATH=/app

# docker-compose.yml
services:
  backend:
    build:
      context: ./app
    environment:
      - PYTHONPATH=/app
    volumes:
      - ./app:/app

# In Python files
from models import db  # Correct when PYTHONPATH is /app
```

## Migration & Seeding in Docker Context
**Symptoms:**
- Changes not persisting
- Database resets unexpectedly
- Seeds running multiple times

**Common Mistakes:**
```yaml
# Wrong: Not persisting database
volumes:
  - ./app:/app
  # Missing database volume

# Wrong: Order of operations in init.sh
flask seed all
flask db upgrade  # Too late, seeds already tried to run
```

**Correct Pattern:**
```yaml
# docker-compose.yml
services:
  db:
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=${POSTGRES_DB}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}

volumes:
  postgres_data:  # Persists database between restarts
```

```bash
# Proper reset sequence
docker compose down -v  # Remove volumes for fresh start
docker compose up --build  # Rebuild with new migrations

# In container
flask db downgrade base
rm -rf migrations/
flask db init
flask db migrate -m "initial migration"
flask db upgrade
flask seed all
```

## Common Debugging Steps in Docker Context
**Symptoms:**
- Unclear where error is occurring
- Can't tell if migrations ran
- Database state unclear

**Solutions:**
```bash
# Check container logs
docker compose logs backend
docker compose logs db

# Get into container and check state
docker exec -it backend sh
flask db current  # Check migration state
flask db history  # See migration history

# Check database directly
docker exec -it db psql -U postgres
\dt  # List tables
\d tablename  # Describe table

# Reset everything
docker compose down -v
docker compose up --build
```

# Development Workflow & Testing Issues

## Development vs Production Setup
**Symptoms:**
- Works locally but fails on Render
- Schema errors in production
- Different behavior between environments

**Common Mistakes:**
```python
# Wrong: Missing production schema handling
class Feature(db.Model):
    __tablename__ = 'features'
    # Missing production schema setup

# Wrong: Hardcoded database URLs
SQLALCHEMY_DATABASE_URI = 'postgresql://localhost/myapp'
```

**Correct Pattern:**
```python
# In models
class Feature(db.Model):
    __tablename__ = 'features'
    
    if environment == "production":
        __table_args__ = {'schema': SCHEMA}

# In config
class Config:
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    if SQLALCHEMY_DATABASE_URI.startswith('postgres://'):
        SQLALCHEMY_DATABASE_URI = SQLALCHEMY_DATABASE_URI.replace('postgres://', 'postgresql://')
```

## Testing Database Operations
**Symptoms:**
- Unclear why relationships aren't working
- Can't verify data state
- Unsure if seeds worked

**Common Mistakes:**
```python
# Wrong: Not verifying relationships
project = Project.query.first()
project.features  # Are they actually there?

# Wrong: Assuming seeds worked
flask seed all  # Did it actually succeed?
```

**Correct Pattern:**
```python
# In Flask shell (docker exec -it backend sh, then flask shell)
>>> from models import db, User, Project, Feature

# Check relationships
>>> project = Project.query.first()
>>> print(f"Owner: {project.owner.username}")
>>> print(f"Features: {[f.name for f in project.features]}")
>>> print(f"Members: {[u.username for u in project.users]}")

# Verify seeds
>>> User.query.all()  # Check users exist
>>> Project.query.filter_by(owner_id=1).all()  # Check relationships
```

## Debugging Database State
**Symptoms:**
- Inconsistent data
- Missing relationships
- Validation errors

**Common Mistakes:**
```python
# Wrong: Not checking data before operations
feature.sprint_id = sprint_id  # Is sprint_id valid?

# Wrong: Not verifying cascade deletes
db.session.delete(project)  # What happens to related features?
```

**Correct Pattern:**
```python
# Verify before operations
def assign_to_sprint(self, sprint_id):
    if sprint_id:
        sprint = Sprint.query.get(sprint_id)
        if not sprint:
            raise ValueError("Sprint not found")
        if sprint.project_id != self.project_id:
            raise ValueError("Sprint must belong to same project")
    self.sprint_id = sprint_id
    db.session.commit()

# Check cascade effects
def test_cascade():
    project = Project.query.first()
    feature_ids = [f.id for f in project.features]
    
    db.session.delete(project)
    db.session.commit()
    
    # Verify features were deleted
    remaining = Feature.query.filter(Feature.id.in_(feature_ids)).all()
    print(f"Remaining features: {remaining}")
```

## Error Handling Strategies
**Symptoms:**
- Unclear error messages
- Database left in bad state
- Silent failures

**Common Mistakes:**
```python
# Wrong: No error handling
def update_feature(self, **kwargs):
    for key, value in kwargs.items():
        setattr(self, key, value)
    db.session.commit()  # What if this fails?

# Wrong: Swallowing errors
try:
    do_something()
except Exception:
    pass  # Error lost!
```

**Correct Pattern:**
```python
from sqlalchemy.exc import IntegrityError, DataError

def update_feature(self, **kwargs):
    try:
        for key, value in kwargs.items():
            setattr(self, key, value)
        db.session.commit()
        return self
    except IntegrityError:
        db.session.rollback()
        raise ValueError("Update violates database constraints")
    except DataError as e:
        db.session.rollback()
        raise ValueError(f"Invalid data format: {str(e)}")
    except Exception as e:
        db.session.rollback()
        raise Exception(f"Update failed: {str(e)}")

# In routes
@feature_routes.route('/<int:id>', methods=['PUT'])
@login_required
def update_feature(id):
    try:
        feature = Feature.query.get(id)
        if not feature:
            return {"errors": "Feature not found"}, 404
            
        updated = feature.update_feature(**request.json)
        return updated.to_dict()
    except ValueError as e:
        return {"errors": str(e)}, 400
    except Exception as e:
        return {"errors": "An error occurred"}, 500
```

