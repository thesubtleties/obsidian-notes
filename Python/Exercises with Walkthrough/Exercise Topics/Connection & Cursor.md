Super Basic Explanation: [[Understanding Database Connections & Cursors]]
## Basic Setup
```python
import sqlite3

# Method 1 (Preferred): Using 'with' statement
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    # Do database stuff here
    # Connection closes automatically

# Method 2 (Not Preferred): Manual cleanup
conn = sqlite3.connect('database.db')
cursor = conn.cursor()
# Do database stuff here
cursor.close()
conn.close()
```

## What's What?
- `sqlite3`: The imported library
- `connect()`: Creates phone line to database
- `cursor()`: Creates your typing cursor to send commands
- `execute()`: Sends SQL commands through cursor
- `fetchall()/fetchone()`: Gets results back

Think of it like:
```
You (Python) → Connection (Phone line) → Cursor (Typing) → Database
```

## Cursor Operations
```python
# Execute a command
cursor.execute("SELECT * FROM cars")

# Get ALL results
all_rows = cursor.fetchall()      # Returns: [(row1), (row2), ...]

# Get ONE result
one_row = cursor.fetchone()       # Returns: (row1)

# Get SOME results
some_rows = cursor.fetchmany(5)   # Returns: First 5 rows

# Get column info
column_info = cursor.description  # Returns: Column names and types
```

## Common Patterns

### Basic Query
```python
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM cars")
    results = cursor.fetchall()
    for row in results:
        print(row)
```

### Insert Data
```python
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("""
        INSERT INTO cars (make, model) 
        VALUES (?, ?)
    """, ('Toyota', 'Supra'))
    conn.commit()  # Save changes!
```

### Get Single Result
```python
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM cars WHERE id = ?", (1,))
    car = cursor.fetchone()
```

## Error Handling
```python
try:
    with sqlite3.connect('database.db') as conn:
        cursor = conn.cursor()
        cursor.execute("SELECT * FROM cars")
        results = cursor.fetchall()
except sqlite3.Error as e:
    print(f"Database error: {e}")
```

## Best Practices
1. Always use `with` statements (handles cleanup)
2. Always use parameterized queries
3. Commit after changes (INSERT, UPDATE, DELETE)
4. Handle errors appropriately
5. Close connections if not using `with`

## Common Gotchas
```python
# Need to commit changes!
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    cursor.execute("INSERT INTO cars (make) VALUES ('Toyota')")
    conn.commit()  # Don't forget this for changes!

# Cursor is only valid inside 'with' block
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
cursor.execute("SELECT * FROM cars")  # Error! Cursor invalid
```

## Quick Reference
```python
# Full example of typical usage
with sqlite3.connect('database.db') as conn:
    cursor = conn.cursor()
    
    # Execute a query
    cursor.execute("SELECT * FROM cars WHERE make = ?", ('Toyota',))
    
    # Get results
    results = cursor.fetchall()
    
    # Work with results
    for row in results:
        print(f"Car ID: {row[0]}, Make: {row[1]}")
```

Remember:
- Connection is your phone line to the database
- Cursor is your typing cursor to send commands
- `with` handles cleanup automatically
- Always use parameters for values in queries
- Commit after making changes