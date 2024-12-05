## Basic Syntax & Variables
```python
# Variable assignment (no declaration keywords needed)
name = "John"              # String
age = 30                   # Integer
height = 1.85             # Float
is_active = True          # Boolean
nothing = None            # None type (similar to null)

# Multiple assignment
x, y, z = 1, 2, 3

# Type hints (optional but recommended)
user_name: str = "Alice"
count: int = 0

# Constants (by convention, no true constants)
MAX_USERS = 100
PI = 3.14159
```
Python uses dynamic typing but supports type hints for better code clarity and tooling support. Unlike JavaScript, there's no `var`, `let`, or `const`.

## Data Types
```python
# Numbers
integer = 42
float_num = 3.14
complex_num = 3 + 4j

# String operations
text = "Hello"
multiline = """
    Multiple
    lines
    of text
"""
formatted = f"Name: {name}, Age: {age}"  # f-strings
joined = " ".join(["Hello", "World"])    # "Hello World"

# Type conversion
str_num = "123"
num = int(str_num)        # String to integer
text = str(num)           # Integer to string
```
Python's string handling is powerful with many built-in methods. F-strings (like JS template literals) are the modern way to format strings.

## Control Flow
```python
# If statements
if age < 18:
    print("Minor")
elif age < 65:
    print("Adult")
else:
    print("Senior")

# For loops
for i in range(5):                # 0 to 4
    print(i)

for item in ["a", "b", "c"]:      # Iterating over sequence
    print(item)

# While loops
count = 0
while count < 5:
    count += 1

# Break and Continue
for i in range(10):
    if i == 3:
        continue    # Skip this iteration
    if i == 8:
        break       # Exit loop
```
Python's loops are more similar to JavaScript's `for...of` than traditional C-style loops. `range()` is commonly used for numeric iterations.

## Functions & Lambda Expressions
```python
# Basic function
def greet(name: str) -> str:
    return f"Hello, {name}"

# Default parameters
def create_user(name="Anonymous", age=0):
    return {"name": name, "age": age}

# *args and **kwargs for flexible arguments
def print_all(*args, **kwargs):
    for arg in args:
        print(arg)
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_all(1, 2, 3, name="John", age=30)

# Lambda functions (similar to arrow functions)
square = lambda x: x**2
double = lambda x: x * 2
```
Python functions can use type hints, default values, and flexible argument handling. Lambda functions are more limited than JavaScript arrow functions.

## Error Handling
```python
# Basic try/except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")
except Exception as e:
    print(f"Other error: {e}")
else:
    print("No error occurred")
finally:
    print("This always runs")

# Raising exceptions
def validate_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age

# Custom exceptions
class CustomError(Exception):
    pass
```
Python's error handling is more structured than JavaScript's try/catch, with specific exception types and the ability to create custom exceptions.

## Key Concepts to Remember
- Indentation is crucial for code blocks (no curly braces)
- Everything is an object
- Dynamic typing but with optional type hints
- No variable declarations (unlike `let`/`const` in JS)
- More built-in data types than JavaScript
- Functions are first-class objects
- Exception handling is type-specific

## Common Gotchas
```python
# Mutable default arguments
def bad(items=[]):        # DON'T: list is shared between calls
    items.append(1)
    return items

def good(items=None):     # DO: Create new list each time
    items = items or []
    items.append(1)
    return items

# Variable scope
x = 1
def scope_test():
    # x = x + 1  # UnboundLocalError
    global x     # Need to declare global to modify
    x = x + 1
```

