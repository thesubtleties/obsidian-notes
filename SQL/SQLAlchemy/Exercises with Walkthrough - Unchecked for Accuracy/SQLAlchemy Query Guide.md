Additional help for: https://appacademy.instructure.com/courses/319/pages/querying-data-with-sqlalchemy-2?module_item_id=56619

## 1. Understanding Queries

### The Query Object
```python
# Create a basic query
query = session.query(Pony)
print(query)  # Shows the SQL it will execute
```
The Query object doesn't hit the database until you tell it to - it just builds the SQL statement.

### Basic Query Structure
```python
# Three main parts:
session.query(Model)           # What to select
       .filter(Model.attr)     # What conditions to apply
       .all()                  # How to return results
```
Think of it like building blocks - each part adds to your query.

## 2. Basic Queries

### Query All Records
```python
# Get all ponies
ponies = session.query(Pony).all()

# Get specific columns
names = session.query(Pony.name, Pony.breed).all()
```
Use `all()` when you want every matching record returned as a list.

### Query by Primary Key
```python
# Get single record by ID
pony = session.query(Pony).get(4)  # Returns None if not found

# Equivalent to:
pony = session.query(Pony).filter(Pony.id == 4).first()
```
`get()` is a shortcut for primary key lookups - use it when you know the ID.

### Select Specific Columns
```python
# Select just what you need
owners = session.query(
    Owner.first_name,
    Owner.last_name
).all()

for first_name, last_name in owners:  # Returns tuples
    print(f"{first_name} {last_name}")
```
Selecting specific columns is more efficient than getting entire records.

## 3. Filtering Results

### Basic Filters
```python
# Equality
ponies = session.query(Pony).filter(Pony.breed == 'Appaloosa')

# Multiple conditions
ponies = session.query(Pony)\
    .filter(Pony.breed == 'Appaloosa')\
    .filter(Pony.birth_year > 2015)
```
Each `filter()` adds an AND condition to your query.

### Common Filter Operations
```python
# Equals
.filter(Pony.name == "Lucky")

# Not equals
.filter(Pony.name != "Lucky")

# Like (case sensitive)
.filter(Pony.name.like("%lucky%"))

# ILike (case insensitive)
.filter(Pony.name.ilike("%lucky%"))

# Greater/Less than
.filter(Pony.birth_year > 2015)

# In list
.filter(Pony.breed.in_(['Appaloosa', 'Arabian']))

# Is null
.filter(Pony.breed.is_(None))

# Is not null
.filter(Pony.breed.isnot_(None))
```
Choose the right operator for your needs - they translate directly to SQL operations.

## 4. Ordering Results

### Basic Ordering
```python
# Single column ascending
query = session.query(Pony).order_by(Pony.name)

# Single column descending
query = session.query(Pony).order_by(Pony.birth_year.desc())

# Multiple columns
query = session.query(Pony).order_by(
    Pony.breed,
    Pony.birth_year.desc()
)
```
Order matters - results are sorted by each column in order.

## 5. Getting Results

### Result Methods
```python
# Get all results
ponies = query.all()  # Returns list

# Get first result
pony = query.first()  # Returns object or None

# Get exactly one result
pony = query.one()    # Raises error if not exactly one result

# Get one or none
pony = query.one_or_none()  # Returns None if no results
```
Choose the right method based on how many results you expect.

### Counting Results
```python
# Count all matches
count = session.query(Pony).count()

# Count with filter
count = session.query(Pony)\
    .filter(Pony.breed == 'Appaloosa')\
    .count()
```
`count()` is optimized to return just the count, not the actual records.

## 6. Complete Examples

### Complex Query
```python
# Find all young Appaloosas with names containing 'lucky'
ponies = session.query(Pony)\
    .filter(Pony.breed == 'Appaloosa')\
    .filter(Pony.birth_year > 2015)\
    .filter(Pony.name.ilike('%lucky%'))\
    .order_by(Pony.name)\
    .all()

for pony in ponies:
    print(f"{pony.name} ({pony.birth_year})")
```

### Error Handling
```python
try:
    pony = session.query(Pony)\
        .filter(Pony.name == 'Unique')\
        .one()
except NoResultFound:
    print("No pony found!")
except MultipleResultsFound:
    print("Multiple ponies found!")
```

## 7. Best Practices

### Query Efficiency
```python
# DO: Select specific columns when possible
session.query(Pony.name, Pony.breed)

# DON'T: Select everything if you don't need it
session.query(Pony)
```

### Query Organization
```python
# Build queries incrementally
query = session.query(Pony)
if breed:
    query = query.filter(Pony.breed == breed)
if min_year:
    query = query.filter(Pony.birth_year >= min_year)
results = query.all()
```

## 8. Quick Reference

### Common Operations
```python
# Basic query
.query(Model)

# Filtering
.filter(condition)

# Ordering
.order_by(Model.column)

# Results
.all()          # List of all results
.first()        # First result or None
.one()          # Exactly one or error
.one_or_none()  # One result or None
.count()        # Count of results
```

Remember:
- Queries don't execute until you call a result method
- Use specific column selection for efficiency
- Choose appropriate result methods
- Handle potential errors
- Build queries incrementally
- Use proper filtering methods
- Consider query performance
