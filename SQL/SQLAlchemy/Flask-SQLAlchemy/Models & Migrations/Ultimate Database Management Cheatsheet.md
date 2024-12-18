## 1. Basic Setup and Commands

### Initial Migration Setup
```bash
# One-time initialization
flask db init                              # Creates migrations folder
```

### Migration Commands
```bash
# Creating and Applying Migrations
flask db migrate -m "describe changes"     # Create new migration file
flask db upgrade                           # Apply migrations
flask db downgrade                         # Undo last migration

# Information Commands
flask db current                           # Show current migration
flask db history                           # Show migration history
flask db stamp head                        # Mark current as latest without migrating
```

### Our Automated Approach (init.sh)
```bash
#!/bin/sh
set -e

# 1. Wait for Database
while ! nc -z db 5432; do
  echo "Postgres is unavailable - sleeping"
  sleep 1
done
echo "PostgreSQL started successfully!"

# 2. Run Migrations
flask --app __init__.py db upgrade
echo "Migrations completed!"

# 3. Handle Seeds
flask --app __init__.py seed undo
flask --app __init__.py seed all

# 4. Start Server
exec gunicorn --bind 0.0.0.0:5000 "__init__:app"
```

### Database Connection Setup

#### Development (Docker)
```yaml
# docker-compose.yml
services:
  db:
    image: postgres:14
    environment:
      POSTGRES_USER: your_user
      POSTGRES_PASSWORD: your_password
      POSTGRES_DB: your_db
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  backend:
    build: .
    depends_on:
      db:
        condition: service_healthy
```

#### Production Setup
```python
# config.py
import os

class Config:
    # Handle different database URLs
    DATABASE_URL = os.environ.get('DATABASE_URL')
    
    # Render PostgreSQL fix
    if DATABASE_URL and DATABASE_URL.startswith('postgres://'):
        DATABASE_URL = DATABASE_URL.replace('postgres://', 'postgresql://')
    
    SQLALCHEMY_DATABASE_URI = DATABASE_URL
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

## 2. Model Management and Migrations

### Basic Model Structure
```python
from .db import db, environment, SCHEMA

class Example(db.Model):
    __tablename__ = 'examples'

    # Production Schema Setup
    if environment == "production":
        __table_args__ = {'schema': SCHEMA}

    # Columns
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100), nullable=False)
    
    # Timestamps
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(
        db.DateTime,
        default=datetime.utcnow,
        onupdate=datetime.utcnow
    )
```

### Migration Patterns

#### Adding New Table
```python
# In your model file
class NewFeature(db.Model):
    __tablename__ = 'new_features'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))

# In terminal
$ flask db migrate -m "add new_features table"
# Review migration file in migrations/versions/
$ docker compose up  # init.sh handles upgrade
```

#### Adding Column to Existing Table
```python
# Safe addition to existing table
new_field = db.Column(db.String(50), nullable=True)  # Safe for existing rows

# Adding required field
new_field = db.Column(db.String(50), nullable=False, server_default='default')
```

A required field (nullable=False) can be unsafe when adding to an existing table that already has data, because those existing rows won't have a value for the new column. You have a few options:

1. Safe approach:
```python
# First migration: Add nullable column
new_field = db.Column(db.String(50), nullable=True)

# Second migration: After populating data, make it required
new_field = db.Column(db.String(50), nullable=False)
```

2. With default value (safe):
```python
# Existing rows will get the default value
new_field = db.Column(db.String(50), nullable=False, server_default='default_value')
```

3. Unsafe approach (will fail if table has data):
```python
# Will fail on existing rows because they'll be NULL
new_field = db.Column(db.String(50), nullable=False)
```

## 3. Development vs Production Considerations

### Development Workflow
```python
# 1. Make model changes
class Project(db.Model):
    new_field = db.Column(db.String(50), nullable=True)  # Start nullable

# 2. Create migration
$ flask db migrate -m "add new_field to projects"

# 3. Review migration file in migrations/versions/
"""
def upgrade():
    # Add column
    op.add_column('projects', sa.Column('new_field', sa.String(50), nullable=True))

def downgrade():
    # Remove column
    op.drop_column('projects', 'new_field')
"""

# 4. Let init.sh handle upgrade or do manually
$ flask db upgrade
```

### Production Considerations
```python
# Schema handling for services like Render
if environment == "production":
    __table_args__ = {'schema': SCHEMA}

# Migration file with schema handling
def upgrade():
    if environment == "production":
        op.execute(f"SET search_path TO {SCHEMA}")
    
    op.create_table(...)

# Connection handling
DATABASE_URL = os.environ.get('DATABASE_URL')
if DATABASE_URL.startswith('postgres://'):
    DATABASE_URL = DATABASE_URL.replace('postgres://', 'postgresql://')
```

## 4. Error Handling and Recovery

### Common Issues and Solutions
```python
# Database out of sync
$ flask db stamp head     # Mark current as latest

# Need to reset migrations
$ flask db downgrade base  # Undo all migrations
$ rm -rf migrations/      # Remove migration files
$ flask db init          # Start fresh
$ flask db migrate -m "initial migration"
$ flask db upgrade

# Connection issues
while ! nc -z db 5432; do
    echo "Waiting for database..."
    sleep 1
done
```

### Production Error Recovery
```bash
# If migration fails in production
1. Access production environment
2. $ flask db current  # Check current state
3. $ flask db downgrade  # Roll back if needed
4. Fix migration files
5. Deploy again

# If database totally stuck
1. Backup data
2. $ flask db stamp head  # Reset migration state
3. Make manual fixes if needed
4. Verify with $ flask db current
```

## 5. Best Practices

### Migration Guidelines
```python
# 1. Always make columns nullable or provide defaults when adding to existing tables
new_field = db.Column(db.String(50), nullable=True)
# or
new_field = db.Column(db.String(50), nullable=False, server_default='default')

# 2. Use multiple migrations for complex changes
# Migration 1: Add nullable column
def upgrade():
    op.add_column('table', sa.Column('new_field', sa.String(50), nullable=True))

# Migration 2: After data population, make it required
def upgrade():
    op.alter_column('table', 'new_field', nullable=False)

# 3. Always include downgrade paths
def downgrade():
    op.drop_column('table', 'new_field')
```

### Development Best Practices
1. Always review migration files before applying
2. Keep migrations small and focused
3. Test migrations on development first
4. Back up data before production migrations
5. Use meaningful migration messages
6. Consider data volume when planning changes

### Production Best Practices
1. Always backup before migrations
2. Test migrations on staging environment
3. Have rollback plan ready
4. Monitor logs during migration
5. Consider maintenance window for large migrations
6. Document significant changes

## 6. Platform-Specific Considerations

### Docker (Our Setup)
```bash
# Advantages
- Consistent development environment
- Automated migration application
- Easy database reset
- Simple connection management

# Considerations
- Watch logs for migration errors
- Remember to create migrations after model changes
- Seeds reset on container restart
```

### Render
```python
# Considerations
- Uses PostgreSQL-specific URL format
- Needs schema handling
- Different connection management
- No direct database access

# Configuration
SCHEMA = os.environ.get("SCHEMA")
DATABASE_URL = os.environ.get("DATABASE_URL")
```

### Other Platforms (Heroku, AWS, etc.)
```python
# Common Considerations
- Connection string format
- SSL requirements
- Connection pooling
- Migration strategy
- Backup procedures
```

