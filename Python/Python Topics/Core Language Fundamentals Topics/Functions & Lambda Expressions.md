## Basic Function Definition
```python
# Basic function
def greet(name: str) -> str:
    """
    Simple greeting function.
    Args:
        name: Person's name
    Returns:
        Greeting message
    """
    return f"Hello, {name}!"

# Function with multiple parameters
def calculate_total(price: float, tax_rate: float = 0.1) -> float:
    return price * (1 + tax_rate)

# Function with type hints
def process_items(items: list[str], count: int) -> list[str]:
    return items[:count]
```
Python functions use `def`, can include type hints, and support docstrings for documentation.

## Parameters and Arguments
```python
# Default parameters
def create_user(name: str = "Anonymous", age: int = 0):
    return {"name": name, "age": age}

# Keyword arguments
create_user(age=25, name="John")  # Order doesn't matter

# Required parameters first
def greet(name, greeting="Hello"):  # Required param before default
    return f"{greeting}, {name}!"

# Mixing positional and keyword args
greet("John", greeting="Hi")
```
Unlike JavaScript, Python has robust support for keyword arguments and default parameters.

## *args and **kwargs
```python
# Variable positional arguments
def sum_all(*args):
    return sum(args)

print(sum_all(1, 2, 3, 4))  # 10

# Variable keyword arguments
def print_info(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="John", age=30, city="NYC")

# Combining both
def combined(*args, **kwargs):
    print(f"Args: {args}")
    print(f"Kwargs: {kwargs}")

combined(1, 2, name="John", age=30)
```
`*args` and `**kwargs` allow functions to accept any number of arguments.

## Lambda Functions
```python
# Basic lambda
square = lambda x: x**2

# Lambda with multiple parameters
add = lambda x, y: x + y

# Lambda in built-in functions
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
evens = list(filter(lambda x: x % 2 == 0, numbers))

# Lambda in sorting
pairs = [(1, 'one'), (2, 'two'), (3, 'three')]
sorted_pairs = sorted(pairs, key=lambda x: x[1])  # Sort by string
```
Lambda functions are similar to JavaScript arrow functions but more limited - one expression only.

## Return Values
```python
# Multiple return values (actually returns tuple)
def get_coordinates():
    return 3, 4  # Returns tuple (3, 4)

x, y = get_coordinates()  # Tuple unpacking

# Return different types
def process_number(num: int) -> Union[str, int]:
    if num < 0:
        return "Negative"
    return num * 2

# Early returns
def check_value(value: int) -> str:
    if value < 0:
        return "Negative"
    if value == 0:
        return "Zero"
    return "Positive"
```
Python functions can return multiple values via tuples and support multiple return types.

## Decorators
```python
# Basic decorator
def log_function(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@log_function
def add(a, b):
    return a + b

# Decorator with parameters
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello {name}")
```
Decorators modify or enhance functions, similar to higher-order functions in JavaScript.

## Scope and Closure
```python
# Global and local scope
global_var = "I'm global"

def outer_function():
    outer_var = "I'm from outer"
    
    def inner_function():
        nonlocal outer_var  # Access outer function's variable
        global global_var   # Access global variable
        print(outer_var, global_var)
    
    return inner_function

# Closure example
def counter():
    count = 0
    def increment():
        nonlocal count
        count += 1
        return count
    return increment
```
Python uses `global` and `nonlocal` keywords to modify variables in outer scopes.

## Common Gotchas
```python
# Mutable default arguments
def bad_append(item, items=[]):  # DON'T DO THIS
    items.append(item)
    return items

def good_append(item, items=None):  # DO THIS
    if items is None:
        items = []
    items.append(item)
    return items

# Late binding closures
def create_multipliers():
    return [lambda x: i * x for i in range(4)]  # Unexpected!

def create_multipliers_fixed():
    return [lambda x, i=i: i * x for i in range(4)]  # Fixed
```

## Best Practices
- Use type hints for better code clarity
- Write clear docstrings for functions
- Use keyword arguments for better readability
- Avoid mutable default arguments
- Keep functions focused and single-purpose
- Use meaningful parameter names
- Return early to avoid deep nesting
- Use lambda functions sparingly
- Consider using dataclasses for complex return types

This covers the main aspects of functions in Python. Functions are first-class objects and can be used in many powerful ways, though some patterns differ significantly from JavaScript.