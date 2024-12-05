## Variable Declaration & Assignment
```python
# Basic variable assignment
name = "John"           # String
age = 25               # Integer
height = 1.85          # Float
is_active = True       # Boolean
has_value = None       # None type

# Multiple assignment
x, y, z = 1, 2, 3      # Assigns values in order
a = b = c = 0          # Same value to multiple variables

# Variable naming conventions
first_name = "John"     # Snake case (Python convention)
MAX_VALUE = 100         # Constants (by convention)
__private = "hidden"    # Name mangling for class attributes
```
Unlike JavaScript, Python has no `var`, `let`, or `const`. Variables are created when first assigned. By convention, constants are in UPPERCASE but aren't truly immutable.

## Type Hints (Python 3.6+)
```python
# Basic type hints
name: str = "John"
age: int = 25
height: float = 1.85
is_active: bool = True

# Complex type hints
from typing import List, Dict, Optional, Union

numbers: List[int] = [1, 2, 3]
user: Dict[str, str] = {"name": "John", "role": "admin"}
maybe_number: Optional[int] = None  # Can be int or None
mixed: Union[str, int] = "hello"    # Can be str or int
```
Type hints are optional but help with code clarity and IDE support. They don't enforce types at runtime by default.

## Basic Operators
```python
# Arithmetic operators
sum = 5 + 3            # Addition
diff = 5 - 3           # Subtraction
product = 5 * 3        # Multiplication
quotient = 5 / 3       # Division (returns float)
floor_div = 5 // 3     # Floor division
remainder = 5 % 3      # Modulus
power = 5 ** 3         # Exponentiation

# Assignment operators
x = 5
x += 3                 # Same as: x = x + 3
x -= 3                 # Same as: x = x - 3
x *= 3                 # Same as: x = x * 3
x /= 3                 # Same as: x = x / 3

# Comparison operators
x == y                 # Equal to
x != y                 # Not equal to
x > y                  # Greater than
x < y                  # Less than
x >= y                 # Greater than or equal to
x <= y                 # Less than or equal to
x is y                 # Identity comparison
x is not y             # Negative identity
```

Python uses 'is' for identity comparison (like '\=\=\=' in JavaScript) and '\=\=' for value comparison.

## String Basics
```python
# String creation
single = 'Single quotes'
double = "Double quotes"     # Both work the same
multi = """
    Multiple
    lines of text
    are possible
"""

# String formatting
name = "John"
age = 25
# f-strings (modern, preferred)
message = f"Name: {name}, Age: {age}"
# .format() method
message = "Name: {}, Age: {}".format(name, age)
# % operator (old style)
message = "Name: %s, Age: %d" % (name, age)
```
F-strings (introduced in Python 3.6) are the modern way to format strings, similar to template literals in JavaScript.

## Comments and Documentation
```python
# Single line comment

"""
Multi-line comment
or docstring when used
at the start of a function/class
"""

def greet(name: str) -> str:
    """
    Greet a person by name.
    
    Args:
        name (str): The person's name
        
    Returns:
        str: A greeting message
    """
    return f"Hello, {name}!"
```
Python uses `#` for single-line comments and triple quotes for multi-line comments or documentation strings.

## Indentation and Blocks
```python
# Python uses indentation for code blocks
def example_function():
    if True:
        print("Indented block")
        for i in range(3):
            print("Another level")
            if i == 2:
                print("Deep indentation")
    print("Back to function level")

# Common indentation errors
def bad_function():
    print("This is fine")
   print("Wrong indentation")  # IndentationError
```
Indentation is crucial in Python - it defines code blocks instead of curly braces like in JavaScript.

## Truth Values
```python
# Falsy values
bool(False)    # False
bool(None)     # False
bool(0)        # False
bool("")       # False
bool([])       # False
bool({})       # False
bool(set())    # False

# Truthy values
bool(True)     # True
bool(1)        # True
bool(-1)       # True
bool("text")   # True
bool([0])      # True
bool({"key": "value"})  # True
```
Python has similar truthy/falsy concepts to JavaScript, but with some differences (e.g., empty strings are falsy).

## Common Gotchas
```python
# Variable scope
x = 1
def function():
    # x = x + 1  # UnboundLocalError
    global x     # Need to declare global to modify
    x = x + 1

# Identity vs Equality
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)    # True (same value)
print(a is b)    # False (different objects)

# Integer caching
x = 256
y = 256
print(x is y)    # True (cached)
x = 257
y = 257
print(x is y)    # False (not cached)
```

## Best Practices
- Use descriptive variable names
- Follow PEP 8 naming conventions (snake_case for variables and functions)
- Use type hints for better code clarity
- Use meaningful constants instead of magic numbers
- Be consistent with string quotes (single or double)
- Keep lines under 79 characters (PEP 8 recommendation)
- Use spaces around operators and after commas
