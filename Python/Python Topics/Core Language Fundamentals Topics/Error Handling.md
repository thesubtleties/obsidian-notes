## Basic Try/Except
```python
# Basic error handling
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero!")

# Capturing the error object
try:
    num = int("not a number")
except ValueError as e:
    print(f"Error: {e}")

# Multiple except blocks
try:
    value = my_dict["key"]
    result = 10 / value
except KeyError:
    print("Key not found in dictionary")
except ZeroDivisionError:
    print("Value cannot be zero")
except Exception as e:
    print(f"Some other error occurred: {e}")
```
Python's error handling is more specific than JavaScript's try/catch, allowing you to catch specific error types.

## Try/Except/Else/Finally
```python
# Complete error handling structure
try:
    file = open("data.txt")
    data = file.read()
except FileNotFoundError:
    print("File not found")
else:
    # Runs if no exception occurs
    print(f"File contents: {data}")
finally:
    # Always runs
    file.close()

# Practical example
def get_user_data(user_id):
    try:
        user = database.fetch(user_id)
    except DatabaseError:
        return None
    else:
        return process_user_data(user)
    finally:
        database.close_connection()
```
The `else` clause runs if no exception occurs, and `finally` always runs regardless of exceptions.

## Raising Exceptions
```python
# Raising basic exceptions
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Custom exceptions
class CustomError(Exception):
    """Custom error class"""
    def __init__(self, message, error_code):
        self.message = message
        self.error_code = error_code
        super().__init__(self.message)

# Using custom exceptions
def process_data(data):
    if not data:
        raise CustomError("Data is empty", 100)
    if not isinstance(data, list):
        raise CustomError("Data must be a list", 101)
```
Python allows raising built-in exceptions and creating custom exception classes.

## Exception Chaining
```python
# Re-raising exceptions
try:
    process_data()
except CustomError as e:
    raise RuntimeError("Processing failed") from e

# Suppressing chaining
try:
    process_data()
except CustomError:
    raise RuntimeError("Processing failed") from None
```
Use `raise from` to chain exceptions, showing the cause of the error.

## Context Managers (with)
```python
# File handling with context manager
with open("file.txt") as file:
    content = file.read()
    # File automatically closes after block

# Custom context manager
from contextlib import contextmanager

@contextmanager
def temporary_state():
    # Setup
    previous_state = save_state()
    try:
        yield
    finally:
        # Cleanup
        restore_state(previous_state)
```
The `with` statement ensures proper resource cleanup, similar to JavaScript's `try/finally`.

## Common Built-in Exceptions
```python
# TypeError
def add(a, b):
    if not (isinstance(a, (int, float)) and isinstance(b, (int, float))):
        raise TypeError("Arguments must be numbers")
    return a + b

# ValueError
def process_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age

# KeyError
def get_config(key):
    try:
        return config_dict[key]
    except KeyError:
        return default_config[key]

# AttributeError
try:
    value.nonexistent_method()
except AttributeError:
    print("Method does not exist")
```
Python provides many built-in exceptions for specific error cases.

## Best Practices and Patterns
```python
# EAFP (Easier to Ask for Forgiveness than Permission)
# Python Style:
try:
    value = my_dict["key"]
except KeyError:
    value = default_value

# vs LBYL (Look Before You Leap)
# Less Pythonic:
if "key" in my_dict:
    value = my_dict["key"]
else:
    value = default_value

# Cleanup pattern
def process_file(filename):
    file = None
    try:
        file = open(filename)
        return file.read()
    finally:
        if file:
            file.close()
```

## Exception Hierarchy
```python
# Common exception hierarchy
BaseException
 ├── SystemExit
 ├── KeyboardInterrupt
 ├── GeneratorExit
 └── Exception
      ├── StopIteration
      ├── ArithmeticError
      │    ├── FloatingPointError
      │    ├── OverflowError
      │    └── ZeroDivisionError
      ├── AssertionError
      ├── AttributeError
      ├── EOFError
      ├── ImportError
      │    └── ModuleNotFoundError
      ├── LookupError
      │    ├── IndexError
      │    └── KeyError
      ├── NameError
      └── ValueError
```

## Common Gotchas
```python
# Bare except clause (avoid)
try:
    process_data()
except:  # DON'T DO THIS
    pass

# Better version
try:
    process_data()
except Exception as e:  # DO THIS
    logger.error(f"Error processing data: {e}")

# Not cleaning up resources
file = open("data.txt")
try:
    data = file.read()
except FileNotFoundError:
    print("File not found")
# file never closed! Use 'with' instead
```

## Best Practices
- Be specific with exception types
- Use `try`/`except` blocks only around code that may raise exceptions
- Prefer `with` statements for resource management
- Follow EAFP (Easier to Ask for Forgiveness than Permission)
- Create custom exceptions for your application's specific errors
- Always clean up resources in `finally` blocks or use context managers
- Use exception chaining when re-raising exceptions
- Avoid bare `except` clauses
- Document exceptions in function docstrings

This covers the main aspects of error handling in Python. The system is more structured than JavaScript's, with specific exception types and additional control flow options like `else` and `finally`.