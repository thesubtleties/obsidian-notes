# locals() in SQLite3 Queries Cheatsheet

## Basic Usage
```python
# Works: Variable names match SQL exactly
def add_car(make, model, year):
    cursor.execute("""
        INSERT INTO cars (make, model, year)
        VALUES (:make, :model, :year)
    """, locals())
```

## Common Patterns

### ✅ Good - Names Match
```python
def find_car(make, year):
    cursor.execute("""
        SELECT * FROM cars 
        WHERE make = :make 
        AND year = :year
    """, locals())
```

### ❌ Bad - Names Don't Match
```python
def find_car(car_make, car_year):  # Won't work!
    cursor.execute("""
        SELECT * FROM cars 
        WHERE make = :make    # No variable named 'make'
        AND year = :year      # No variable named 'year'
    """, locals())
```

## How to Fix Mismatched Names

### Option 1: Match SQL to Variables
```python
def find_car(car_make, car_year):
    cursor.execute("""
        SELECT * FROM cars 
        WHERE make = :car_make 
        AND year = :car_year
    """, locals())
```

### Option 2: Manual Dictionary
```python
def find_car(car_make, car_year):
    cursor.execute("""
        SELECT * FROM cars 
        WHERE make = :make 
        AND year = :year
    """, {
        'make': car_make,
        'year': car_year
    })
```

### Option 3: Use ? Style Instead
```python
def find_car(car_make, car_year):
    cursor.execute("""
        SELECT * FROM cars 
        WHERE make = ? 
        AND year = ?
    """, (car_make, car_year))
```

## Quick Tips
- locals() includes ALL local variables
- Names must match EXACTLY
- Use ? style if names don't match
- Better for simple scripts
- Manual dictionaries clearer for complex queries

## When to Use What
```python
# Use locals() when names match and it's simple
def simple_query(make, model):
    cursor.execute("SELECT * FROM cars WHERE make = :make AND model = :model", locals())

# Use dictionary when names don't match
def complex_query(car_make, car_model):
    cursor.execute("SELECT * FROM cars WHERE make = :make AND model = :model", 
        {'make': car_make, 'model': car_model})

# Use ? when it's straightforward and order is clear
def simple_positional(make, model):
    cursor.execute("SELECT * FROM cars WHERE make = ? AND model = ?", 
        (make, model))
```