## Lists (Mutable Sequences)
```python
# Creation
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]
nested = [1, [2, 3], [4, 5, 6]]

# Common operations
numbers.append(6)        # Add to end
numbers.insert(0, 0)     # Insert at index
numbers.pop()           # Remove and return last item
numbers.remove(3)       # Remove first occurrence of value
del numbers[0]          # Delete at index
length = len(numbers)   # Get length

# Slicing
first_three = numbers[:3]    # First three elements
last_two = numbers[-2:]      # Last two elements
reversed_list = numbers[::-1] # Reverse list
```
Lists are Python's most versatile sequence type, similar to JavaScript arrays.

## Dictionaries (Key-Value Pairs)
```python
# Creation
person = {
    "name": "John",
    "age": 30,
    "skills": ["Python", "JavaScript"]
}

# Common operations
person["city"] = "NYC"           # Add/update item
del person["age"]               # Remove item
age = person.get("age", 25)     # Get with default
keys = person.keys()            # Get all keys
values = person.values()        # Get all values
items = person.items()          # Get key-value pairs

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}
```
Dictionaries are Python's primary mapping type, similar to JavaScript objects.

## Sets (Unordered, Unique Elements)
```python
# Creation
unique_nums = {1, 2, 3, 3}   # Duplicates removed
empty_set = set()            # Creating empty set

# Set operations
unique_nums.add(4)           # Add element
unique_nums.remove(2)        # Remove element
unique_nums.discard(5)       # Remove if exists

# Set mathematics
set1 = {1, 2, 3}
set2 = {3, 4, 5}
union = set1 | set2         # Union
intersection = set1 & set2  # Intersection
difference = set1 - set2    # Difference
```
Sets are optimized for membership testing and eliminating duplicates.

## Tuples (Immutable Sequences)
```python
# Creation
point = (1, 2, 3)
single_item = (1,)          # Note the comma
nested = (1, (2, 3), 4)

# Common operations
x, y, z = point             # Tuple unpacking
first = point[0]            # Indexing
length = len(point)         # Length
contains = 2 in point       # Membership testing

# Named tuples
from collections import namedtuple
Person = namedtuple('Person', ['name', 'age'])
john = Person('John', 30)
```
Tuples are immutable and often used for grouping related values.

## Arrays (Homogeneous Sequences)
```python
# Regular arrays (from array module)
from array import array
numbers = array('i', [1, 2, 3, 4, 5])  # Integer array

# NumPy arrays (for numerical computing)
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
matrix = np.array([[1, 2], [3, 4]])
```
Arrays are more efficient than lists for numerical data.

## Queues and Stacks
```python
# Queue (FIFO)
from collections import deque
queue = deque()
queue.append(1)          # Add to right
queue.appendleft(2)      # Add to left
queue.pop()             # Remove from right
queue.popleft()         # Remove from left

# Stack (LIFO - using list)
stack = []
stack.append(1)         # Push
stack.pop()            # Pop
```
Python provides specialized containers for different use cases.

## Advanced Collections
```python
from collections import Counter, defaultdict, OrderedDict

# Counter
words = ['a', 'b', 'a', 'c', 'b', 'a']
word_counts = Counter(words)

# defaultdict
grouped = defaultdict(list)
grouped['a'].append(1)  # No KeyError if key doesn't exist

# OrderedDict (less relevant in Python 3.7+)
ordered = OrderedDict()
ordered['a'] = 1
ordered['b'] = 2
```
The collections module provides specialized container datatypes.

## Common Operations
```python
# List comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]

# Dictionary comprehension
square_dict = {x: x**2 for x in range(5)}

# Set comprehension
even_squares = {x**2 for x in range(10) if x % 2 == 0}

# Sorting
sorted_list = sorted(numbers)
sorted_dict = sorted(person.items(), key=lambda x: x[1])
```

## Memory and Performance Considerations
```python
# List vs. Generator
numbers = [x for x in range(1000000)]  # Creates list in memory
numbers_gen = (x for x in range(1000000))  # Generator object

# Copy vs. Reference
shallow_copy = list1.copy()  # or list1[:]
import copy
deep_copy = copy.deepcopy(list1)
```

## Best Practices
- Choose the right data structure for your use case:
  - Lists for ordered, mutable sequences
  - Dictionaries for key-value mappings
  - Sets for unique elements
  - Tuples for immutable groups
  - Arrays for numerical data
- Use list comprehensions for clarity
- Consider generators for large sequences
- Use specialized collections when appropriate
- Be aware of mutable vs immutable types
- Consider memory usage for large datasets
- Use built-in methods over manual implementations
- Leverage Python's built-in functions (len, sorted, etc.)

## Common Gotchas
```python
# Mutable default arguments
def bad(items=[]):        # DON'T
    items.append(1)
    return items

def good(items=None):     # DO
    if items is None:
        items = []
    items.append(1)
    return items

# List multiplication
zeros = [0] * 3          # OK: [0, 0, 0]
nested = [[0]] * 3       # NOT OK: Creates references
```

This overview covers the main data structures in Python. Each has its own strengths and use cases, and understanding when to use each one is key to writing efficient Python code.