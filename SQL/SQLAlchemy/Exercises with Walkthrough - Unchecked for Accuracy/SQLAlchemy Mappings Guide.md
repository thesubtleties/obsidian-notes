Additional help for: https://appacademy.instructure.com/courses/319/pages/sqlalchemy-mappings-2?module_item_id=56616

Note: This guide shows raw SQLAlchemy syntax to explain core concepts. In our Flask application, we use Flask-SQLAlchemy which simplifies this setup. While the syntax differs (db.Model instead of Base, db.Column instead of Column), understanding these fundamentals helps explain what's happening behind Flask-SQLAlchemy's convenient abstractions. See: [[Flask - SQLAlchemy Integration Guide]]
## 1. Basic Setup

### Creating Base Class
```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```
This creates the foundation for all your model classes. Think of it as the parent class that provides SQLAlchemy functionality.

### Basic Model Structure
```python
class Owner(Base):
    __tablename__ = 'owners'  # Explicitly declare table name
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
```
SQLAlchemy requires explicit table names - no magical guessing like in other ORMs.

## 2. Column Mappings

### Basic Column Types
```python
from sqlalchemy.types import Integer, String, Boolean, DateTime

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    username = Column(String(50), unique=True)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime)
```
Each column maps directly to a database column with specific types and constraints.

### Column Options
```python
class Product(Base):
    __tablename__ = 'products'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100), nullable=False)  # Required field
    price = Column(Integer, default=0)          # Default value
    sku = Column(String(50), unique=True)       # Unique constraint
```
Column options help enforce data integrity at the database level.

## 3. Relationships

### One-to-Many Relationship
```python
class Owner(Base):
    __tablename__ = 'owners'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    
    # One owner has many ponies
    ponies = relationship("Pony", back_populates="owner")

class Pony(Base):
    __tablename__ = 'ponies'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    owner_id = Column(Integer, ForeignKey('owners.id'))
    
    # Each pony has one owner
    owner = relationship("Owner", back_populates="ponies")
```
`back_populates` keeps both sides of the relationship in sync - when you update one side, the other updates automatically.

### Many-to-Many Relationship
```python
# Association table (no model needed)
pony_handlers = Table(
    'pony_handlers',
    Base.metadata,
    Column('pony_id', ForeignKey('ponies.id'), primary_key=True),
    Column('handler_id', ForeignKey('handlers.id'), primary_key=True)
)

class Pony(Base):
    __tablename__ = 'ponies'
    
    id = Column(Integer, primary_key=True)
    handlers = relationship(
        "Handler",
        secondary=pony_handlers,
        back_populates="ponies"
    )

class Handler(Base):
    __tablename__ = 'handlers'
    
    id = Column(Integer, primary_key=True)
    ponies = relationship(
        "Pony",
        secondary=pony_handlers,
        back_populates="handlers"
    )
```
Many-to-many relationships use an intermediate table to store the connections.

## 4. Working with Relationships

### Adding Related Objects
```python
# Create objects
owner = Owner(name="John Doe")
pony = Pony(name="Silver")

# Method 1: Set relationship directly
pony.owner = owner

# Method 2: Add to collection
owner.ponies.append(pony)
```
Both methods achieve the same result thanks to `back_populates`.

### Querying Related Objects
```python
# Get all ponies for an owner
owner = session.query(Owner).first()
for pony in owner.ponies:
    print(pony.name)

# Get owner of a pony
pony = session.query(Pony).first()
print(pony.owner.name)
```
Relationships can be accessed like normal Python attributes.

## 5. Complete Example

### Full Model Setup
```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship
from sqlalchemy.schema import Column, ForeignKey, Table
from sqlalchemy.types import Integer, String

Base = declarative_base()

# Association table
pony_handlers = Table(
    'pony_handlers', Base.metadata,
    Column('pony_id', ForeignKey('ponies.id'), primary_key=True),
    Column('handler_id', ForeignKey('handlers.id'), primary_key=True)
)

class Owner(Base):
    __tablename__ = 'owners'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    ponies = relationship("Pony", back_populates="owner")

class Pony(Base):
    __tablename__ = 'ponies'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    owner_id = Column(Integer, ForeignKey('owners.id'))
    
    owner = relationship("Owner", back_populates="ponies")
    handlers = relationship(
        "Handler",
        secondary=pony_handlers,
        back_populates="ponies"
    )

class Handler(Base):
    __tablename__ = 'handlers'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(255))
    ponies = relationship(
        "Pony",
        secondary=pony_handlers,
        back_populates="handlers"
    )
```

## 6. Best Practices

### Model Organization
```python
# One model per file
# models/owner.py
class Owner(Base):
    __tablename__ = 'owners'
    # ... columns and relationships

# models/__init__.py
from .owner import Owner
from .pony import Pony
from .handler import Handler
```
Keep models organized and separated for maintainability.

### Relationship Naming
```python
# Clear, descriptive relationship names
class User(Base):
    authored_posts = relationship("Post", back_populates="author")
    received_messages = relationship("Message", back_populates="recipient")
```
Use descriptive names that indicate the relationship type and purpose.

## 7. Quick Reference

### Common Column Types
```python
Column(Integer)           # Whole numbers
Column(String(length))    # Variable-length strings
Column(Boolean)          # True/False values
Column(DateTime)         # Date and time
Column(Float)            # Decimal numbers
Column(JSON)            # JSON data
```

### Relationship Types
```python
# One-to-Many
relationship("Model", back_populates="attribute")

# Many-to-Many
relationship("Model", 
            secondary=association_table,
            back_populates="attribute")

# One-to-One
relationship("Model", 
            back_populates="attribute",
            uselist=False)
```

Remember:
- Always specify `__tablename__`
- Use `back_populates` for relationships
- Create association tables for many-to-many
- Keep models organized
- Use clear naming conventions
- Consider column constraints
- Handle relationships consistently

