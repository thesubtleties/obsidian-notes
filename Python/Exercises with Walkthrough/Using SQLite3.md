 Reformatted version of: https://appacademy.instructure.com/courses/319/pages/using-sqlite3-2?module_item_id=56597
 
 Note: If you exit mid run you will need to create cursor, etc. again:
 ```py
 >>> import sqlite3 
 >>> conn = sqlite3.connect('dev.db') 
 >>> cursor = conn.cursor()
```

## 1. Introduction & Setup
```python
import sqlite3
DB_FILE = "dev.db"
```
First, we'll create our database. In terminal:
```bash
sqlite3 dev.db
```

Copy these SQL commands to set up test data:
```sql
CREATE TABLE owners (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    first_name VARCHAR(255) NOT NULL,
    last_name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);

CREATE TABLE cars (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    manu_year INTEGER NOT NULL,
    make VARCHAR(255),
    model VARCHAR(255),
    owner_id INTEGER NOT NULL,
    FOREIGN KEY (owner_id) REFERENCES owners(id)
);

INSERT INTO owners (first_name, last_name, email) VALUES
('Tim', 'Petrol', 'rotary@fast.com'),
('Ryan', 'Runner', '10sec@jdm.com'),
('Tia', 'Petrol', 'typer@wtec.com');

INSERT INTO cars (manu_year, make, model, owner_id) VALUES
(1993, 'Mazda', 'Rx7', 1),
(1995, 'Mitsubishi', 'Eclipse', 2),
(1994, 'Acura', 'Integra', 3);
```

## 2. First Python [[Connection & Cursor| Connection]]
Create `sqlite3_demo.py`:
```python
with sqlite3.connect(DB_FILE) as conn:
    print(conn)  # Shows: <sqlite3.Connection object at 0x...>
```

💡 Learning Check:
```python
# In Python REPL:
>>> import sqlite3
>>> conn = sqlite3.connect('dev.db')
>>> print(type(conn))  # See it's a Connection object
>>> print(dir(conn))   # Explore available methods
```

## 3. Working with [[Connection & Cursor| Cursors]]
```python
with sqlite3.connect(DB_FILE) as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT 'Hello World!'")
    result = cursor.fetchone()
    print(result)  # Shows: ('Hello World!',)
```

💡 Learning Check:
```python
# In REPL:
>>> cursor = conn.cursor()
>>> for method in dir(cursor):
...     if not method.startswith('_'):
...         print(method)
# Explore cursor capabilities
```

## 4. Understanding Data Types
SQLite3 to Python type conversions:
```python
# In REPL:
>>> cursor.execute("SELECT * FROM cars LIMIT 1")
<sqlite3.Cursor object at 0x7fc727937730>
>>> row = cursor.fetchone()
>>> print(row)  # Let's see the actual data first
(1, 1993, 'Mazda', 'Rx7', 1)
>>> print(type(row[0]))  # id column (INTEGER)
<class 'int'>
>>> print(type(row[2]))  # make column (TEXT)
<class 'str'>
```

Type Mapping Reference:
- NULL → None
- INTEGER → int
- REAL → float
- TEXT → str
- BLOB → bytes

## 5. Basic SELECT Operations
```python
def print_all_cars():
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT manu_year, make, model FROM cars')
        cars = cursor.fetchall()
        for car in cars:
            print(car)

# Try it:
>>> from sqlite3_demo import print_all_cars
>>> print_all_cars()
```

💡 Learning Check:
```python
# In REPL:
>>> cursor.execute("SELECT * FROM cars")
>>> print(cursor.description)  # See column info
>>> one_row = cursor.fetchone()
>>> all_rows = cursor.fetchall()
>>> print(len(all_rows))  # How many rows left?
```

## 6. [[Parameterized Queries]]
```python
def get_owners_cars(owner_id):
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        # Notice the :owner_id syntax
        cursor.execute("""
            SELECT manu_year, make, model FROM cars
            WHERE owner_id = :owner_id
        """, {'owner_id': owner_id})
        return cursor.fetchall()

# Alternative style using ?
def get_owners_cars_alt(owner_id):
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            SELECT manu_year, make, model FROM cars
            WHERE owner_id = ?
        """, (owner_id,))  # Note the comma creating a tuple
        return cursor.fetchall()
```

💡 Why Parameterized Queries?
- Prevents SQL injection
- Handles data type conversion
- Cleaner code

## 7. Complete CRUD Operations
```python
def add_new_car(manu_year, make, model, owner_id):
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            INSERT INTO cars (manu_year, make, model, owner_id)
            VALUES (:manu_year, :make, :model, :owner_id)
        """, locals())  # locals() creates dict from parameters

def change_car_owner(car_id, new_owner_id):
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            UPDATE cars SET owner_id = :new_owner_id
            WHERE id = :car_id
        """, {'car_id': car_id, 'new_owner_id': new_owner_id})

def delete_car(car_id):
    with sqlite3.connect(DB_FILE) as conn:
        cursor = conn.cursor()
        cursor.execute('DELETE FROM cars WHERE id = :car_id',
                      {'car_id': car_id})
```

See: [[locals() in SQLite3 Queries]]
## 8. Error Handling
```python
def safe_car_operation():
    try:
        with sqlite3.connect(DB_FILE) as conn:
            cursor = conn.cursor()
            # Operations here
            conn.commit()  # Only needed for write operations
    except sqlite3.Error as e:
        print(f"Database error: {e}")
        return False
    return True
```

## 9. Testing It All
```python
if __name__ == "__main__":
    print("\nAll cars initially:")
    print_all_cars()
    
    print("\nAdding new car...")
    add_new_car(2000, 'Ford', 'Lightning', 2)
    
    print("\nOwner 1's cars:")
    print(get_owners_cars(1))
    
    print("\nChanging car owner...")
    change_car_owner(1, 2)
    
    print("\nFinal car list:")
    print_all_cars()
```

## 10. Key Takeaways
1. Always use `with` statements for connections
2. Always use parameterized queries
3. Understand the type conversions
4. Use error handling for robustness
5. The cursor is your main interface to the database

## 11. Common Gotchas
- Forgetting to commit changes
- Not closing connections (solved by `with`)
- String formatting instead of parameters
- Not handling errors
- Forgetting tuples need comma: `(value,)`

This guide mixes practical implementation with learning exercises, helping you understand both how and why things work the way they do in Python's SQLite3 implementation.