Additional help for: https://appacademy.instructure.com/courses/319/pages/create-update-and-delete-with-sqlalchemy-2?module_item_id=56617

## 1. Understanding CRUD in SQLAlchemy

### Overview of Operations
```python
# The four main operations:
# Create - Add new records
# Read   - Query existing records (covered in query guide)
# Update - Modify existing records
# Delete - Remove records
```
Unlike raw SQL, SQLAlchemy lets you work with Python objects instead of writing SQL statements directly.

## 2. Creating Records

### Basic Object Creation
```python
from sqlalchemy.orm import Session
from models import Owner, Pony

# Create new owner
owner = Owner(
    first_name="John",
    last_name="Doe",
    email="john@example.com"
)

session.add(owner)      # Stage the change
session.commit()        # Save to database

print(owner.id)  # Now has a value!
```
Notice how the `id` is automatically set after commit - SQLAlchemy handles the database-generated values for you.

### Creating Related Objects
```python
# Create owner with pony
owner = Owner(first_name="John", last_name="Doe")
pony = Pony(
    name="Silver",
    birth_year=2020,
    breed="Appaloosa",
    owner=owner    # Relationship is handled automatically
)

# Only need to add one!
session.add(owner)  # SQLAlchemy tracks related objects
session.commit()

print(owner.ponies)  # Shows [<Pony 'Silver'>]
print(pony.owner)    # Shows <Owner 'John Doe'>
```
SQLAlchemy's relationship management is much simpler than other ORMs - just assign objects naturally.

## 3. Updating Records

### Simple Updates
```python
# Find and update
pony = session.query(Pony).first()
pony.name = "New Silver"
pony.birth_year = 2019
session.commit()
```
Changes aren't saved until you commit - this lets you make multiple changes and save them all at once.

### Updating Relationships
```python
# Change pony's owner
new_owner = Owner(first_name="Jane", last_name="Doe")
pony.owner = new_owner

# Add pony to owner's collection
owner.ponies.append(new_pony)

session.commit()  # Save all changes at once
```
Relationships can be modified in both directions thanks to `back_populates`.

## 4. Deleting Records

### Basic Deletion
```python
# Delete single record
session.delete(pony)
session.commit()
```
Simple when there are no relationships to consider.

### Cascade Deletion
```python
class Owner(Base):
    __tablename__ = 'owners'
    
    ponies = relationship(
        "Pony",
        back_populates="owner",
        cascade="all, delete-orphan"  # Key for handling related records
    )

# Now when you delete an owner:
session.delete(owner)  # Also deletes their ponies
session.commit()
```
Cascade options determine what happens to related records - very important for maintaining data integrity.

## 5. Transaction Management

### Using Rollback
```python
# Make some changes
pony.name = "Mistake"
pony.birth_year = 1800

# Oops! Let's undo those changes
session.rollback()
print(pony.name)  # Back to original name
```
Rollback is your "undo" button - it reverts objects to their last committed state.

### Error Handling Pattern
```python
try:
    # Make changes
    owner.name = "New Name"
    pony.owner = owner
    session.commit()
except SQLAlchemyError as e:
    session.rollback()
    print(f"An error occurred: {e}")
    raise
```
Always handle database operations with proper error management.

## 6. Complete Example

### Full CRUD Workflow
```python
# Setup
from sqlalchemy.orm import Session
from models import Owner, Pony

# Create
new_owner = Owner(first_name="John", last_name="Doe")
new_pony = Pony(
    name="Silver",
    birth_year=2020,
    owner=new_owner
)
session.add(new_owner)
session.commit()

# Update
new_pony.name = "Golden Silver"
new_pony.birth_year = 2019
session.commit()

# Delete
session.delete(new_owner)  # Cascades to pony
session.commit()
```

## 7. Best Practices

### Session Management
```python
# Always use try/finally or context managers
try:
    session.add(owner)
    session.commit()
except SQLAlchemyError:
    session.rollback()
    raise
finally:
    session.close()
```

### Relationship Handling
```python
# Do: Use relationship attributes
pony.owner = new_owner

# Don't: Manually set IDs
pony.owner_id = new_owner.id  # Avoid this
```

Remember:
- Always commit or rollback changes
- Use proper cascade settings
- Handle errors appropriately
- Close sessions when done
- Use relationship attributes
- Keep transactions focused
- Test cascade behaviors

