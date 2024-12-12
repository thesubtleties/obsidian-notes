Additional help for: https://appacademy.instructure.com/courses/319/pages/set-up-for-sqlalchemy-query-articles-2?module_item_id=56618

## 1. Project Structure

### Basic Setup
```plaintext
your_project/
├── dev.db              # SQLite database
├── pony_owners.py      # Main Python file
└── Pipfile             # Dependencies
```
This structure keeps everything organized for learning SQLAlchemy queries.

## 2. Database Connection Setup

### Engine and Session Configuration
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Database URL
db_url = "sqlite:///dev.db"

# Create engine
engine = create_engine(db_url)

# Create session factory
SessionFactory = sessionmaker(bind=engine)

# Create session
session = SessionFactory()
```
This creates your database connection and session - you'll need these for all queries.

## 3. Model Definitions

### Base Class Setup
```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```
The Base class provides the foundation for all your models.

### Association Table
```python
pony_handlers = Table(
    "pony_handlers",
    Base.metadata,
    Column("pony_id", ForeignKey("ponies.id"), primary_key=True),
    Column("handler_id", ForeignKey("handlers.id"), primary_key=True)
)
```
This table enables many-to-many relationships between ponies and handlers.

### Model Classes
```python
class Owner(Base):
    __tablename__ = 'owners'
    
    id = Column(Integer, primary_key=True)
    first_name = Column(String(255))
    last_name = Column(String(255))
    email = Column(String(255))
    
    # Relationship to ponies
    ponies = relationship("Pony", back_populates="owner")
```
Each model maps to a database table and defines its relationships.

```python
class Pony(Base):
    __tablename__ = "ponies"
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    birth_year = Column(Integer)
    breed = Column(String(255))
    owner_id = Column(Integer, ForeignKey("owners.id"))
    
    # Relationships
    owner = relationship("Owner", back_populates="ponies")
    handlers = relationship("Handler",
                          secondary=pony_handlers,
                          back_populates="ponies")
```
The Pony model shows both one-to-many and many-to-many relationships.

```python
class Handler(Base):
    __tablename__ = "handlers"
    
    id = Column(Integer, primary_key=True)
    first_name = Column(String(50))
    last_name = Column(String(50))
    employee_id = Column(String(12))
    
    # Relationship to ponies
    ponies = relationship("Pony",
                         secondary=pony_handlers,
                         back_populates="handlers")
```
Handlers demonstrate the other side of the many-to-many relationship.

## 4. Complete Setup File

### Full Code (pony_owners.py)
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship, sessionmaker
from sqlalchemy.schema import Column, ForeignKey, Table
from sqlalchemy.types import Integer, String

# Base class
Base = declarative_base()

# Association table
pony_handlers = Table(
    "pony_handlers",
    Base.metadata,
    Column("pony_id", ForeignKey("ponies.id"), primary_key=True),
    Column("handler_id", ForeignKey("handlers.id"), primary_key=True)
)

# Models
[Previous model definitions here]

# Database connection
db_url = "sqlite:///dev.db"
engine = create_engine(db_url)
SessionFactory = sessionmaker(bind=engine)
session = SessionFactory()

# Your query code will go here

# Cleanup
session.close()
engine.dispose()
```

## 5. Verification Steps

### Testing Setup
```python
# Add to bottom of file to test setup
if __name__ == "__main__":
    # Test database connection
    try:
        result = session.query(Owner).first()
        print("Database connection successful!")
    except Exception as e:
        print(f"Error: {e}")
```

## 6. Common Issues and Solutions

### Connection Issues
```python
# Check database file exists
import os
if not os.path.exists('dev.db'):
    print("Database file not found!")

# Verify table existence
from sqlalchemy import inspect
inspector = inspect(engine)
print(inspector.get_table_names())
```

### Import Issues
```python
# Make sure all imports are at top
from sqlalchemy import (
    create_engine, Column, ForeignKey, 
    Integer, String, Table
)
from sqlalchemy.orm import relationship, sessionmaker
from sqlalchemy.ext.declarative import declarative_base
```

Remember:
- Keep virtual environment activated
- Check database file location
- Verify all imports
- Close session when done
- Dispose of engine
- Use proper indentation
- Follow model naming conventions

This setup provides the foundation for:
- Writing queries
- Filtering results
- Using relationships
- Performing joins
- Handling query results

