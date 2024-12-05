## Tuple Creation
```python
# Basic tuple creation
point = (1, 2, 3)
single_item = (1,)           # Note the comma!
empty = ()
mixed = (1, "hello", 3.14)

# Tuple packing
coordinates = 1, 2, 3        # Parentheses optional
```
The comma is what makes a tuple, not the parentheses. Single-item tuples need a trailing comma to distinguish them from parenthesized expressions.

## Basic Operations
```python
# Accessing elements
x = point[0]                # First element
y = point[-1]               # Last element
subset = point[1:3]         # Slicing

# Length and membership
length = len(point)         # Get length
exists = 2 in point         # Check membership

# Concatenation and repetition
combined = (1, 2) + (3, 4)  # (1, 2, 3, 4)
repeated = (1, 2) * 3       # (1, 2, 1, 2, 1, 2)
```
Unlike lists, tuples are immutable - you can't modify elements after creation.

## Tuple Unpacking
```python
# Basic unpacking
x, y, z = point             # Unpack to variables

# Extended unpacking (Python 3+)
first, *rest = (1, 2, 3, 4)  # first=1, rest=[2, 3, 4]
first, *middle, last = (1, 2, 3, 4)  # middle=[2, 3]

# Swapping values
a, b = b, a                 # Tuple unpacking for swap

# Ignoring values
x, _, z = point             # Ignore middle value
```
Tuple unpacking is a powerful feature for working with multiple values simultaneously.

## Named Tuples
```python
from collections import namedtuple

# Creating named tuple class
Point = namedtuple('Point', ['x', 'y', 'z'])
Person = namedtuple('Person', 'name age city')  # Space-separated string

# Creating instances
p = Point(1, 2, 3)
person = Person('John', 30, 'NYC')

# Accessing values
print(p.x)                  # Using attribute name
print(person[1])            # Using index

# Converting to dictionary
person_dict = person._asdict()
```
Named tuples add semantic meaning to tuple elements while maintaining immutability.

## Tuple Methods
```python
numbers = (1, 2, 2, 3, 2)

# Count occurrences
count = numbers.count(2)    # Returns 3

# Find index
index = numbers.index(2)    # Returns 1 (first occurrence)
index = numbers.index(2, 2) # Start search from index 2
```
Tuples have only two methods due to their immutable nature.

## Type Hints with Tuples
```python
from typing import Tuple

# Basic type hints
coordinates: Tuple[int, int, int] = (1, 2, 3)

# Mixed types
record: Tuple[str, int, float] = ("John", 30, 175.5)

# Variable-length tuples
numbers: Tuple[int, ...] = (1, 2, 3, 4, 5)
```
Type hints help document expected tuple structures and enable better IDE support.

## Common Use Cases
```python
# Return multiple values from function
def get_coordinates():
    return (3, 4)

# Dictionary keys (tuples are immutable)
locations = {(0, 0): 'origin', (1, 1): 'point'}

# Data records
records = [
    ('John', 30, 'NYC'),
    ('Alice', 25, 'LA'),
]

# Fixed data structures
DAYS = ('Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun')
```
Tuples are great for representing fixed collections and returning multiple values.

## Performance Considerations
```python
# Memory usage
import sys
list_size = sys.getsizeof([1, 2, 3])
tuple_size = sys.getsizeof((1, 2, 3))  # Typically smaller

# Creation speed
from timeit import timeit
list_time = timeit(lambda: [1, 2, 3], number=1000000)
tuple_time = timeit(lambda: (1, 2, 3), number=1000000)
```
Tuples generally use less memory and are faster to create than lists.

## Converting Between Types
```python
# List to tuple
my_list = [1, 2, 3]
my_tuple = tuple(my_list)

# String to tuple
chars = tuple("hello")      # ('h', 'e', 'l', 'l', 'o')

# Nested conversion
nested_list = [[1, 2], [3, 4]]
nested_tuple = tuple(tuple(x) for x in nested_list)
```
Conversion maintains the sequence order but changes mutability.

## Error Handling
```python
# Handling unpacking errors
try:
    x, y = (1, 2, 3)  # Too many values
except ValueError as e:
    print("Unpacking error:", e)

# Safe tuple access
def safe_get(tup, index, default=None):
    try:
        return tup[index]
    except IndexError:
        return default
```
Always handle potential errors when unpacking or accessing tuple elements.

## Best Practices
```python
# Use tuples for heterogeneous data
person = ('John', 30, 'NYC')  # Different types

# Use lists for homogeneous data
numbers = [1, 2, 3, 4]      # Same type

# Use named tuples for clarity
User = namedtuple('User', ['name', 'age', 'city'])
user = User('John', 30, 'NYC')

# Document tuple structure
def process_point(point: Tuple[float, float, float]):
    """Process a 3D point tuple (x, y, z)."""
    x, y, z = point
    return x + y + z
```
Choose tuples when you want immutability and have a fixed number of elements.

## Common Gotchas
```python
# Single item tuple
single_number = (1)         # NOT a tuple - just an int
single_tuple = (1,)        # Correct - note the comma

# Nested mutability
point = (1, [2, 3], 4)
point[1].append(5)         # Works! List inside is mutable

# Tuple unpacking length
def process_pair(pair):
    x, y = pair            # Assumes exactly 2 elements
    return x + y

# Better version
def safe_process_pair(pair):
    if len(pair) != 2:
        raise ValueError("Expected pair of values")
    x, y = pair
    return x + y
```
Be careful with single-item tuples, nested mutability, and unpacking assumptions.

This covers the main aspects of working with tuples in Python, from basic operations to advanced usage patterns and best practices.