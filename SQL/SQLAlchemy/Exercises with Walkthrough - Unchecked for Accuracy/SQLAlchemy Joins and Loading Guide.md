Additional help for: https://appacademy.instructure.com/courses/319/pages/querying-across-joins-in-sqlalchemy-2?module_item_id=56620

## 1. Understanding Joins

### Basic Join Query
```python
from sqlalchemy.orm import joinedload

# Find owners with specific pony breeds
owners = session.query(Owner)\
    .join(Pony)\
    .filter(Pony.breed == "Hirzai")\
    .all()
```
Joins let you filter records based on related data - like finding owners based on their ponies' attributes.

## 2. Loading Strategies

### Lazy Loading (Default)
```python
# Lazy loading example
for owner in session.query(Owner):
    print(owner.first_name)
    # New query happens here!
    for pony in owner.ponies:  
        print(f"\t{pony.name}")
```
Lazy loading makes separate queries as needed. Simple but can lead to the "N+1 query problem".

### Eager Loading
```python
# Eager loading with joinedload
owners = session.query(Owner)\
    .options(joinedload(Owner.ponies))\
    .all()

for owner in owners:  # No additional queries!
    print(owner.first_name)
    for pony in owner.ponies:
        print(f"\t{pony.name}")
```
Eager loading gets all data in one query - more efficient when you know you'll need related data.

## 3. Query Debugging

### Enable SQL Logging
```python
import logging

# Setup logging
logging.basicConfig()
logging.getLogger('sqlalchemy.engine').setLevel(logging.INFO)
```
Logging helps you see exactly what SQL queries are being executed.

### Understanding Query Output
```python
# Lazy loading generates multiple queries:
INFO: SELECT * FROM owners
INFO: SELECT * FROM ponies WHERE owner_id = 1
INFO: SELECT * FROM ponies WHERE owner_id = 2
INFO: SELECT * FROM ponies WHERE owner_id = 3

# Eager loading generates one query:
INFO: SELECT * FROM owners JOIN ponies ON owners.id = ponies.owner_id
```

## 4. Complex Queries

### Combining Joins and Eager Loading
```python
# Filter by related data AND eager load
query = session.query(Owner)\
    .join(Pony)\
    .filter(Pony.breed == "Hirzai")\
    .options(joinedload(Owner.ponies))

for owner in query:
    print(f"{owner.first_name}'s ponies:")
    for pony in owner.ponies:  # Already loaded!
        print(f"\t{pony.name}")
```
Use `join()` for filtering and `joinedload()` for loading strategy.

### Multiple Relationships
```python
# Loading multiple relationships
query = session.query(Pony)\
    .options(
        joinedload(Pony.owner),
        joinedload(Pony.handlers)
    )
```
You can eager load multiple relationships at once.

## 5. Best Practices

### Choosing Loading Strategies
```python
# Use lazy loading when:
# - You might not need related data
# - Working with large datasets
owners = session.query(Owner).all()

# Use eager loading when:
# - You know you'll need related data
# - Working with smaller datasets
owners = session.query(Owner)\
    .options(joinedload(Owner.ponies))\
    .all()
```

### Performance Optimization
```python
# Limit results for pagination
page_size = 10
page = 1
query = session.query(Owner)\
    .options(joinedload(Owner.ponies))\
    .limit(page_size)\
    .offset((page - 1) * page_size)
```

## 6. Common Patterns

### Filtering on Multiple Relations
```python
# Find owners with specific pony and handler criteria
query = session.query(Owner)\
    .join(Pony)\
    .join(Handler)\
    .filter(Pony.breed == "Hirzai")\
    .filter(Handler.employee_id.startswith("A"))\
    .options(joinedload(Owner.ponies))
```

### Conditional Eager Loading
```python
from sqlalchemy.orm import contains_eager

query = session.query(Owner)\
    .join(Pony)\
    .filter(Pony.breed == "Hirzai")\
    .options(contains_eager(Owner.ponies))
```

## 7. Quick Reference

### Loading Types
```python
# Lazy Loading
query = session.query(Owner)

# Eager Loading
query = session.query(Owner).options(joinedload(Owner.ponies))

# Selective Loading
query = session.query(Owner).options(
    joinedload(Owner.ponies).load_only("name")
)
```

Remember:
- Use joins for filtering
- Use joinedload for loading strategy
- Consider data size when choosing strategy
- Monitor query performance
- Use logging for debugging
- Consider pagination for large datasets
- Keep queries readable and maintainable

