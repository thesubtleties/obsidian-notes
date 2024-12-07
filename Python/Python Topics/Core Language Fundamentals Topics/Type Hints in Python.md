## Basic Type Hints
```python
# Basic variable type hints
name: str = "John"
age: int = 30
height: float = 1.85
is_active: bool = True

# Function type hints
def greet(name: str) -> str:
    return f"Hello, {name}"

def calculate_total(price: float, tax_rate: float = 0.1) -> float:
    return price * (1 + tax_rate)

# Multiple parameters
def process_user(name: str, age: int, scores: list[int]) -> dict:
    return {
        "name": name,
        "age": age,
        "average_score": sum(scores) / len(scores)
    }
```
Type hints are optional but help with code clarity, IDE support, and documentation. They don't enforce types at runtime.

## Collection Type Hints
```python
from typing import List, Dict, Set, Tuple

# List type hints
numbers: List[int] = [1, 2, 3]
names: List[str] = ["Alice", "Bob"]

# Dictionary type hints
user: Dict[str, str] = {"name": "John", "email": "john@example.com"}
scores: Dict[str, int] = {"Alice": 95, "Bob": 85}

# Set type hints
unique_numbers: Set[int] = {1, 2, 3}

# Tuple type hints
point: Tuple[int, int] = (10, 20)
person: Tuple[str, int, float] = ("John", 30, 1.85)
```
Use the `typing` module for complex collection types.

## Optional and Union Types
```python
from typing import Optional, Union

# Optional type (can be None)
def get_user(user_id: int) -> Optional[str]:
    if user_id > 0:
        return "John"
    return None

# Union type (multiple possible types)
def process_id(user_id: Union[int, str]) -> str:
    if isinstance(user_id, int):
        return f"ID: {user_id}"
    return f"ID: {int(user_id)}"

# Python 3.10+ union operator
def newer_process_id(user_id: int | str) -> str:
    return f"ID: {user_id}"
```
Use `Optional` when a value might be None, and `Union` when multiple types are possible.

## Custom Type Hints
```python
from typing import TypeVar, NewType

# Custom type
UserId = NewType('UserId', int)
user_id: UserId = UserId(123)

# Generic type variable
T = TypeVar('T')
def first_element(l: List[T]) -> T:
    return l[0]

# Type aliases
Vector = List[float]
Matrix = List[Vector]

def scale_vector(v: Vector, scalar: float) -> Vector:
    return [x * scalar for x in v]
```
Create custom types for more specific type hints.

## Class Type Hints
```python
class User:
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age
    
    # Method return type hints
    def get_summary(self) -> str:
        return f"{self.name} is {self.age} years old"
    
    # Class methods
    @classmethod
    def from_dict(cls, data: Dict[str, any]) -> 'User':
        return cls(data['name'], data['age'])
```
Use string literals for forward references to the class itself.

## Type Hints with *args and **kwargs
```python
from typing import Any

def print_all(*args: Any) -> None:
    for arg in args:
        print(arg)

def process_options(**kwargs: str) -> None:
    for key, value in kwargs.items():
        print(f"{key}: {value}")

# More specific version
from typing import TypeVar, Callable
T = TypeVar('T')
def map_values(func: Callable[[T], T], *values: T) -> List[T]:
    return [func(x) for x in values]
```
Use `Any` for completely flexible types, or be more specific with TypeVars.

## Common Patterns and Best Practices
```python
# Pattern: Type aliases for clarity
from typing import Dict, List

JsonDict = Dict[str, Any]
UserList = List[Dict[str, Any]]

def process_data(data: JsonDict) -> UserList:
    return [{"name": "John"}]

# Pattern: Union for multiple return types
def divide(a: float, b: float) -> Union[float, str]:
    if b == 0:
        return "Cannot divide by zero"
    return a / b

# Pattern: Optional for nullable values
def find_user(user_id: int) -> Optional[Dict[str, any]]:
    if user_id < 0:
        return None
    return {"id": user_id, "name": "John"}
```

## Type Checking Tools
```python
# Use mypy for static type checking
# Terminal: mypy your_file.py

# Example with type issues
def greet(name: str) -> str:
    if name == "":
        return None  # mypy will flag this error
    return f"Hello, {name}"

# Type checking decorators (runtime)
from typing import runtime_checkable, Protocol

@runtime_checkable
class Printable(Protocol):
    def print(self) -> str: ...
```

## Gotchas and Notes
1. Type hints are for documentation and tools, not runtime enforcement
2. Forward references need string literals or `from __future__ import annotations`
3. Type hints can impact startup time (but not runtime performance)
4. IDEs like PyCharm and VSCode use type hints for better code completion
5. Type hints were introduced in Python 3.5 and improved in later versions

## Best Practices
1. Use type hints in public APIs
2. Be as specific as possible with types
3. Use `Optional` instead of Union with None
4. Use type aliases for complex types
5. Consider using type checking tools like mypy
6. Don't overuse `Any` - be specific when possible

This covers the main aspects of type hints in Python, focusing on practical usage and common patterns.