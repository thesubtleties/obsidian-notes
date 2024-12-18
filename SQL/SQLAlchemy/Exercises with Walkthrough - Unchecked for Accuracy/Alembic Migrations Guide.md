Additional help for: https://appacademy.instructure.com/courses/319/pages/migrations-with-alembic-2?module_item_id=56624

## 1. Initial Setup

### Installation
```bash
# Create virtual environment and install Alembic
pipenv install alembic
```
Alembic is SQLAlchemy's migration tool - think of it like version control for your database schema.

### Project Structure
```bash
# Initialize Alembic
pipenv run alembic init migrations
```
This creates:
```plaintext
your_project/
├── migrations/           # Migration directory
│   ├── env.py           # Environment configuration
│   ├── script.py.mako   # Migration template
│   └── versions/        # Migration files
└── alembic.ini          # Alembic configuration
```

## 2. Configuration

### Database URL (alembic.ini)
```ini
# Don't hardcode database URLs
sqlalchemy.url = driver://user:pass@localhost/dbname
```
Instead, use environment variables:

```python
# env.py
import os
config.set_main_option('sqlalchemy.url', os.environ.get('DATABASE_URL'))
```

### File Naming (alembic.ini)
```ini
# Better file naming with timestamps
file_template = %%(year)d%%(month).2d%%(day).2d_%%(hour).2d%%(minute).2d%%(second).2d_%%(slug)s
```
This creates readable filenames like `20230817_143022_create_users.py` instead of random hashes.

## 3. Creating Migrations

### Generate Migration
```bash
# Create new migration
pipenv run alembic revision -m "create users table"
```
This creates a new file with upgrade and downgrade methods.

### Basic Migration Example
```python
"""create users table

Revision ID: abc123def456
Revises: 
Create Date: 2023-08-17 14:30:22
"""
from alembic import op
import sqlalchemy as sa

def upgrade():
    op.create_table(
        'users',
        sa.Column('id', sa.Integer, primary_key=True),
        sa.Column('username', sa.String(50), nullable=False),
        sa.Column('email', sa.String(255), unique=True)
    )

def downgrade():
    op.drop_table('users')
```
Notice how similar this is to SQLAlchemy models - you can use the same column types and constraints.

## 4. Running Migrations

### Apply Migrations
```bash
# Apply all pending migrations
pipenv run alembic upgrade head

# Apply next migration only
pipenv run alembic upgrade +1

# Apply to specific revision
pipenv run alembic upgrade abc123def456
```
The `head` refers to the latest migration version.

### Rollback Migrations
```bash
# Rollback all migrations
pipenv run alembic downgrade base

# Rollback one migration
pipenv run alembic downgrade -1

# Rollback to specific revision
pipenv run alembic downgrade abc123def456
```
Always test your downgrades to ensure they work correctly.

## 5. Handling Multiple Branches

### Branch Situation
```plaintext
                     -- abc123 (Team A's changes)
                    /
<-- base revision <
                    \
                     -- def456 (Team B's changes)
```
This happens when multiple developers create migrations from the same base.

### Merging Branches
```bash
# Merge two branches
pipenv run alembic merge -m "merge team changes" abc123 def456
```
Creates a new merge migration that combines both branches.

## 6. Best Practices

### Migration Structure
```python
def upgrade():
    # Always order operations logically
    # 1. Create tables
    # 2. Add indexes
    # 3. Add foreign keys
    op.create_table(...)
    op.create_index(...)
    op.create_foreign_key(...)

def downgrade():
    # Reverse order of upgrade
    op.drop_foreign_key(...)
    op.drop_index(...)
    op.drop_table(...)
```

### Environment Variables
```bash
# Store database URL in environment
export DATABASE_URL="postgresql://user:pass@localhost/dbname"
```
Never commit sensitive database credentials.

## 7. Common Operations

### Table Operations
```python
# Create table
op.create_table(
    'users',
    sa.Column('id', sa.Integer, primary_key=True),
    sa.Column('name', sa.String(50))
)

# Alter table
op.alter_column('users', 'name',
    existing_type=sa.String(50),
    type_=sa.String(100))

# Drop table
op.drop_table('users')
```

### Index Operations
```python
# Create index
op.create_index('idx_user_email', 'users', ['email'])

# Drop index
op.drop_index('idx_user_email')
```

Remember:
- Always test migrations before applying to production
- Keep migrations small and focused
- Use meaningful migration names
- Handle branches carefully
- Test both upgrade and downgrade paths
- Use environment variables for configuration
- Keep track of migration history

