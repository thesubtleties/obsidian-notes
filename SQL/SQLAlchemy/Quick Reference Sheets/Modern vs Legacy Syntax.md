## Key Changes and Best Practices

### 1. Query Execution
```python
from sqlalchemy import create_engine, text

# OLD (pre-1.4)
result = connection.execute("SELECT * FROM table")

# NEW (1.4+)
result = connection.execute(text("SELECT * FROM table"))

# With parameters - OLD
result = connection.execute("SELECT * FROM table WHERE id = %s", (5,))

# With parameters - NEW
result = connection.execute(
    text("SELECT * FROM table WHERE id = :id"),
    {"id": 5}
)
```

### 2. Session Usage
```python
from sqlalchemy.orm import Session

# OLD
session = Session()
try:
    # work with session
    session.commit()
finally:
    session.close()

# NEW (preferred)
with Session(engine) as session:
    with session.begin():
        # work with session
        # auto-commits if no exceptions
```

### 3. Engine Creation and Connection
```python
# OLD
engine = create_engine('sqlite:///db.sqlite')
connection = engine.connect()
try:
    # work with connection
finally:
    connection.close()

# NEW
engine = create_engine('sqlite:///db.sqlite')
with engine.connect() as connection:
    # work with connection
    # auto-closes when done
```

### 4. ORM Query Syntax
```python
# OLD
session.query(User).filter(User.name == 'John').all()

# NEW (2.0-style)
from sqlalchemy import select
result = session.execute(
    select(User).where(User.name == 'John')
).scalars().all()

# Joining - OLD
session.query(User).join(Address).all()

# Joining - NEW
result = session.execute(
    select(User).join(Address)
).scalars().all()
```

### 5. Insert Operations
```python
# OLD
session.add(User(name='John'))
session.commit()

# NEW (can still use old style, but new style available)
from sqlalchemy import insert
stmt = insert(User).values(name='John')
session.execute(stmt)
session.commit()

# Bulk Insert - NEW
stmt = insert(User).values([
    {"name": "John"},
    {"name": "Jane"}
])
session.execute(stmt)
```

### 6. Update Operations
```python
# OLD
session.query(User).filter(User.name == 'John').update({"name": "Johnny"})

# NEW
from sqlalchemy import update
stmt = update(User).where(User.name == 'John').values(name='Johnny')
session.execute(stmt)
```

### 7. Delete Operations
```python
# OLD
session.query(User).filter(User.name == 'John').delete()

# NEW
from sqlalchemy import delete
stmt = delete(User).where(User.name == 'John')
session.execute(stmt)
```

### 8. Model Definition
```python
# OLD and NEW (both still work, but declarative_base is legacy)
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

# NEW (preferred)
from sqlalchemy.orm import DeclarativeBase
class Base(DeclarativeBase):
    pass

# Model Definition
class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    name = Column(String)
```

### 9. Relationship Loading
```python
# Eager Loading - NEW explicit style
stmt = (
    select(User)
    .options(selectinload(User.addresses))
    .where(User.name == 'John')
)
result = session.execute(stmt).scalars().first()

# Lazy Loading - Still works but warning in 2.0
user = session.query(User).first()
addresses = user.addresses  # triggers separate query
```

### 10. Type Annotations (2.0+)
```python
from typing import Optional
from sqlalchemy.orm import Mapped, mapped_column

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    email: Mapped[Optional[str]]
```

### Key Points to Remember:
- Always use `text()` for raw SQL queries
- Prefer context managers (`with` statements)
- Use `execute()` with `select()`, `insert()`, `update()`, `delete()`
- Session management is more explicit
- Type annotations are supported and encouraged in 2.0+
- `select()` replaces `query()` in modern syntax
- Parameters use named style (`:param`) instead of `%s`

### Common Gotchas:
1. Raw strings need `text()`
2. Result sets may need `.scalars()` for single-column results
3. Parameters must be passed as dictionaries with named parameters
4. Type annotations require SQLAlchemy 2.0+
5. Some older tutorials might show deprecated patterns

