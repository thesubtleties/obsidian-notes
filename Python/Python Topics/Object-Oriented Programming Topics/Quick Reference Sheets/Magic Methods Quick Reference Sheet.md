## Quick Start - Most Common Needs
```python
"I want to..."                              "Use this..."
Initialize an object                     -> __init__
Make my object printable                 -> __str__
Make my object comparable (==, <, etc.)  -> __eq__, __lt__
Make my object work with len()           -> __len__
Make my object iterable (for loops)      -> __iter__
Make my object subscriptable (obj[key])  -> __getitem__
```

## Essential Magic Methods
### Constructor and Representation
```python
Method      Trigger         Returns     Common Use
__init__    obj = MyClass() None       Initialize object
__str__     str(obj)       str         Human-readable string
__repr__    repr(obj)      str         Debugging/development string

# Example
class Person:
    def __init__(self, name):
        self.name = name
    
    def __str__(self):
        return self.name  # Simple string for users
    
    def __repr__(self):
        return f"Person(name='{self.name}')"  # Detailed for debugging
```
Note: Always implement `__repr__`. Python uses `__str__` if available, falls back to `__repr__`.

### Basic Comparison Methods
```python
Method      Trigger     Returns     Notes
__eq__      obj == x    bool        Define this first
__lt__      obj < x     bool        Python derives other comparisons
__gt__      obj > x     bool        Optional if __lt__ defined
__le__      obj <= x    bool        Optional if __lt__ defined
__ge__      obj >= x    bool        Optional if __lt__ defined

# Example with minimum implementation
class SoccerScore:
    def __init__(self, team: str, goals: int):
        self.team = team
        self.goals = goals
    
    def __eq__(self, other):
        if not isinstance(other, SoccerScore):
            return NotImplemented
        return self.goals == other.goals
    
    def __lt__(self, other):
        if not isinstance(other, SoccerScore):
            return NotImplemented
        return self.goals < other.goals

# Usage
manchester = SoccerScore("Manchester United", 3)
chelsea = SoccerScore("Chelsea", 1)
arsenal = SoccerScore("Arsenal", 3)

print(chelsea < manchester)     # True (1 < 3 goals)
print(manchester == arsenal)    # True (both 3 goals)
print(manchester > chelsea)     # True (Python derives this from __lt__)

# Works with sorting and min/max
scores = [manchester, chelsea, arsenal]
highest_score = max(scores)     # Manchester or Arsenal (both 3 goals)
sorted_scores = sorted(scores)  # [chelsea, manchester, arsenal]
```
Gotcha: Always check `isinstance` and return `NotImplemented` for incompatible types.

### Container Methods
```python
Method        Trigger       Returns     Common Use
__len__      len(obj)      int         Size of container
__getitem__  obj[key]      any         Access items
__setitem__  obj[key] = x  None        Set items
__delitem__  del obj[key]  None        Delete items
__iter__     iter(obj)     iterator    Make iterable

# Example of minimal container
class SimpleList:
    def __init__(self):
        self._items = []
    
    def __len__(self):
        return len(self._items)
    
    def __getitem__(self, index):
        return self._items[index]
    
    def __iter__(self):
        return iter(self._items)
```
Pattern: These usually go together for container-like objects.

## Arithmetic Methods
```python
Method      Trigger     Returns         Notes
__add__     obj + x    new object      Addition
__sub__     obj - x    new object      Subtraction
__mul__     obj * x    new object      Multiplication
__truediv__ obj / x    new object      Division
__floordiv__ obj // x  new object      Floor division

# Example
class Vector:
    def __init__(self, x, y):
        self.x, self.y = x, y
    
    def __add__(self, other):
        if not isinstance(other, Vector):
            return NotImplemented
        return Vector(self.x + other.x, self.y + other.y)
```
Pattern: Always return a new object, don't modify self.

## Context Manager Methods
```python
Method      Trigger         Notes
__enter__   with obj:      Setup
__exit__    end of with    Cleanup

# Example
class FileHandler:
    def __init__(self, filename):
        self.filename = filename
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename)
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
```
Common Use: Resource management (files, connections, locks)

## Advanced Methods (Less Common)
```python
Method        Trigger           Use Case
__call__     obj()             Make object callable like function
__getattr__  obj.undefined     Handle missing attributes
__setattr__  obj.x = y         Customize attribute setting
__slots__    class variable    Memory optimization
```

## Methods Requiring Imports
```python
from abc import ABC, abstractmethod
class MyABC(ABC):
    @abstractmethod
    def my_method(self):
        pass

from dataclasses import dataclass
@dataclass
class Point:
    x: float
    y: float
```

## Common Patterns and Gotchas

### Pattern: Basic Class Setup
```python
class MyClass:
    def __init__(self):
        pass
    
    def __str__(self):
        return "Human readable"
    
    def __repr__(self):
        return f"{self.__class__.__name__}()"
```

### Pattern: Comparable Object
```python
class ComparableObject:
    def __eq__(self, other):
        if not isinstance(other, type(self)):
            return NotImplemented
        return self.value == other.value
    
    def __lt__(self, other):
        if not isinstance(other, type(self)):
            return NotImplemented
        return self.value < other.value
```

### Major Gotchas
1. Not checking types in comparison methods
2. Modifying self in arithmetic operations
3. Forgetting to return NotImplemented
4. Complex computations in __len__
5. Missing __repr__ implementation

### Best Practices
1. Always implement __repr__
2. Make arithmetic operations return new objects
3. Keep __str__ simple and human-readable
4. Use @property instead of getattr when possible
5. Implement __iter__ for container-like objects

