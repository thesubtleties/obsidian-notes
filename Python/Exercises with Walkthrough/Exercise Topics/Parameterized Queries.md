## The Bad Way (NEVER DO THIS)
```python
# BAD - SQL Injection Risk!
user_id = "1 OR 1=1"  # Malicious input
cursor.execute(f"SELECT * FROM cars WHERE owner_id = {user_id}")

# BAD - String Formatting
year = 1995
cursor.execute(f"SELECT * FROM cars WHERE year > {year}")
```

## The Good Way: Two Styles

### Style 1: Question Marks (?)
```python
# Single Parameter
cursor.execute("SELECT * FROM cars WHERE year > ?", (1995,))

# Multiple Parameters
cursor.execute("""
    SELECT * FROM cars 
    WHERE make = ? AND year > ?
""", ('Toyota', 1995))
```
- `?` is a placeholder in the SQL
- Values go in a tuple
- Order matters! First value matches first ?, second matches second ?, etc.

### Style 2: Named Parameters (:name)
```python
# Single Parameter
cursor.execute("""
    SELECT * FROM cars 
    WHERE year > :year
""", {'year': 1995})

# Multiple Parameters
cursor.execute("""
    SELECT * FROM cars 
    WHERE make = :make AND year > :year
""", {
    'make': 'Toyota',
    'year': 1995
})
```
- `:year` is a placeholder in the SQL
- Values go in a dictionary
- Names must match between SQL and dictionary
- Order doesn't matter because names match up

## What's Actually Happening
1. You write SQL with placeholders (? or :name)
2. You provide values separately
3. SQLite safely combines them
4. Prevents SQL injection

## Quick Examples
```python
# Style 1 (Question Marks)
owner_id = 1
cursor.execute("SELECT * FROM cars WHERE owner_id = ?", (owner_id,))

# Style 2 (Named Parameters)
owner_id = 1
cursor.execute("SELECT * FROM cars WHERE owner_id = :id", {'id': owner_id})

# They do the same thing!
```

The `:name` in the SQL is just a placeholder that matches the key in your dictionary. It's part of SQLite's syntax for parameterized queries.