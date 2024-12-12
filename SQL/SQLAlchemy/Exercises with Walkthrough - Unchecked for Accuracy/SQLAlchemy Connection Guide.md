Additional help for: https://appacademy.instructure.com/courses/319/pages/connecting-sqlalchemy-to-sqlite3-2?module_item_id=56615

## 1. Understanding Database Connections

### Connection Components
```python
# URL Format:
# sqlite:///path/to/database.db
#   ^       ^
#   |       |
# scheme  path
```
Understanding database URLs is crucial for proper connection setup. SQLite URLs are simpler than other databases.

## 2. Creating the Engine

### Basic Engine Setup
```python
from sqlalchemy import create_engine

# Local database
engine = create_engine("sqlite:///dev.db")

# Absolute path
engine = create_engine("sqlite:////absolute/path/to/dev.db")
```
The engine is your primary connection manager - create only one per database.

### Engine Lifecycle
```python
# Create engine
engine = create_engine("sqlite:///dev.db")

try:
    # Use engine here
    pass
finally:
    # Clean up
    engine.dispose()
```
Always dispose of your engine when you're done to free up resources.

## 3. Using Connections

### Basic Connection Pattern
```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///dev.db")

with engine.connect() as connection:
    result = connection.execute("""
        SELECT * FROM owners
    """)
    for row in result:
        print(row)
```
The `with` statement ensures proper cleanup of database connections.

### Executing Queries
```python
with engine.connect() as connection:
    # Simple query
    result = connection.execute("SELECT * FROM owners")
    
    # Join query
    result = connection.execute("""
        SELECT o.first_name, p.name
        FROM owners o
        JOIN ponies p ON o.id = p.owner_id
    """)
```
You can execute any valid SQL through the connection.

## 4. Working with Sessions

### Session Setup
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

# Create engine
engine = create_engine("sqlite:///dev.db")

# Create session factory
SessionFactory = sessionmaker(bind=engine)

# Create session
session = SessionFactory()
```
Sessions are used for ORM operations rather than raw SQL.

### Session Usage Pattern
```python
SessionFactory = sessionmaker(bind=engine)

try:
    session = SessionFactory()
    # Use session for database operations
    session.commit()
except:
    session.rollback()
    raise
finally:
    session.close()
```
Always handle session cleanup and transaction management properly.

## 5. Complete Examples

### Raw SQL Example
```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///dev.db")

def get_owner_ponies():
    with engine.connect() as connection:
        result = connection.execute("""
            SELECT o.first_name, o.last_name, p.name
            FROM owners o
            JOIN ponies p ON (o.id = p.owner_id)
        """)
        return [dict(row) for row in result]

try:
    owners_and_ponies = get_owner_ponies()
    for record in owners_and_ponies:
        print(f"{record['first_name']} owns {record['name']}")
finally:
    engine.dispose()
```

### Session Example
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

engine = create_engine("sqlite:///dev.db")
SessionFactory = sessionmaker(bind=engine)

def get_owners():
    session = SessionFactory()
    try:
        # ORM queries go here
        return session.query(Owner).all()
    finally:
        session.close()
```

## 6. Best Practices

### Engine Management
```python
# DO: Create one engine per database
engine = create_engine("sqlite:///dev.db")

# DON'T: Create multiple engines
def bad_practice():
    engine = create_engine("sqlite:///dev.db")  # Don't do this!
```

### Connection Management
```python
# DO: Use context managers
with engine.connect() as conn:
    result = conn.execute("SELECT 1")

# DON'T: Leave connections open
conn = engine.connect()  # Don't forget to close!
```

### Session Management
```python
# DO: Use session in a managed context
def good_practice():
    session = SessionFactory()
    try:
        # Use session
        session.commit()
    except:
        session.rollback()
        raise
    finally:
        session.close()
```

## 7. Quick Reference

### Connection URLs
```python
# SQLite URLs
"sqlite:///relative/path/db.db"    # Relative path
"sqlite:////absolute/path/db.db"   # Absolute path
"sqlite:///:memory:"               # In-memory database
```

### Common Operations
```python
# Create engine
engine = create_engine(url)

# Get connection
with engine.connect() as conn:
    pass

# Create session
Session = sessionmaker(bind=engine)
session = Session()
```

Remember:
- One engine per database
- Always dispose of engines
- Use context managers for connections
- Handle session transactions properly
- Clean up resources
- Use appropriate error handling
- Consider connection pooling
- Monitor connection usage

