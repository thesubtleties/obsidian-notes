## List Comprehension
```python
# Basic list comprehension
squares = [x**2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]

# Nested comprehension
matrix = [[i+j for j in range(3)] for i in range(3)]

# Multiple conditions
filtered = [x for x in range(100) 
           if x % 2 == 0 
           if x % 3 == 0]

# With if-else
values = [x if x > 0 else 0 for x in [-1, 0, 1, 2]]
```
List comprehensions provide a concise way to create lists based on existing sequences. They're more readable and often faster than equivalent for loops. Use them when transforming or filtering data, but avoid when the logic becomes too complex. Perfect for data processing, mathematical operations, and filtering collections.

## Dictionary Comprehension
```python
# Basic dictionary comprehension
square_dict = {x: x**2 for x in range(5)}

# Filtering with conditions
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}

# Transform existing dictionary
prices = {'apple': 0.40, 'banana': 0.50}
doubled = {k: v*2 for k, v in prices.items()}

# Combining two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
combined = {k: v for k, v in zip(keys, values)}
```
Dictionary comprehensions are ideal for creating mappings from existing data. They're particularly useful for data transformation, creating lookup tables, and restructuring data. Common in data processing, configuration management, and mapping relationships between values.

## Sorting
```python
# Basic sorting
sorted_list = sorted([3, 1, 4, 1, 5, 9])
reverse_sorted = sorted([1, 2, 3], reverse=True)

# Sort with key function
words = ['banana', 'pie', 'Washington', 'book']
sorted_by_len = sorted(words, key=len)
sorted_ignore_case = sorted(words, key=str.lower)

# Sort objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [Person('John', 30), Person('Alice', 25)]
sorted_people = sorted(people, key=lambda p: p.age)

# Multiple criteria sorting
data = [(1, 'B'), (2, 'A'), (1, 'A')]
sorted_data = sorted(data, key=lambda x: (x[0], x[1]))
```
Sorting is fundamental in data organization and processing. Python's sorting is stable (preserves order of equal elements) and highly customizable through key functions. Essential for data analysis, displaying information, and maintaining ordered collections.

## Filtering and Mapping
```python
numbers = [1, 2, 3, 4, 5, 6]

# Using filter
evens = list(filter(lambda x: x % 2 == 0, numbers))

# Using map
squares = list(map(lambda x: x**2, numbers))

# Combining map and filter
even_squares = list(map(lambda x: x**2, 
                       filter(lambda x: x % 2 == 0, numbers)))

# With multiple iterables
a = [1, 2, 3]
b = [10, 20, 30]
sums = list(map(lambda x, y: x + y, a, b))
```
Filter and map operations are fundamental functional programming concepts. Filter selects elements based on a condition, while map transforms each element. They're memory-efficient as they return iterators, making them suitable for large datasets.

## Zip and Enumerate
```python
# Basic zip
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")

# Enumerate with start index
for i, name in enumerate(names, start=1):
    print(f"{i}. {name}")

# Unzipping
pairs = [(1, 'a'), (2, 'b'), (3, 'c')]
numbers, letters = zip(*pairs)

# Creating dictionaries
dict(zip(names, ages))  # {'Alice': 25, 'Bob': 30, 'Charlie': 35}
```
Zip combines multiple iterables in parallel, while enumerate adds counting to iteration. These are essential for parallel iteration, creating dictionaries from parallel lists, and maintaining indexed collections. Common in data processing and file parsing.

## String Operations
```python
# String methods
text = "  Hello, World!  "
cleaned = text.strip()
upper = text.upper()
lower = text.lower()
replaced = text.replace('Hello', 'Hi')

# Splitting and joining
words = text.split(',')
lines = text.splitlines()
joined = ', '.join(['apple', 'banana', 'cherry'])

# String formatting
name = "Alice"
age = 30
# f-strings (Python 3.6+)
message = f"{name} is {age} years old"
# format() method
message = "{} is {} years old".format(name, age)
# % operator (older style)
message = "%s is %d years old" % (name, age)
```
String operations are crucial for text processing and formatting. Python provides rich string manipulation methods. Essential for data cleaning, text analysis, file processing, and generating formatted output.

## Set Operations
```python
# Set operations
set1 = {1, 2, 3, 4}
set2 = {3, 4, 5, 6}

union = set1 | set2          # All elements from both sets
intersection = set1 & set2   # Common elements
difference = set1 - set2     # Elements in set1 but not in set2
sym_diff = set1 ^ set2      # Elements in either set but not both

# Set comprehension
evens = {x for x in range(10) if x % 2 == 0}

# Membership testing
if 1 in set1:
    print("Found")
```
Set operations are perfect for dealing with unique collections and mathematical set operations. They're highly efficient for membership testing and removing duplicates. Common in data deduplication, finding common elements, and mathematical operations.

## Dictionary Operations
```python
# Dictionary methods
data = {'a': 1, 'b': 2}
keys = data.keys()
values = data.values()
items = data.items()

# Safe get with default
value = data.get('c', 0)  # Returns 0 if key not found

# Dictionary update
data.update({'c': 3, 'd': 4})

# Dictionary merge (Python 3.9+)
dict1 = {'a': 1, 'b': 2}
dict2 = {'b': 3, 'c': 4}
merged = dict1 | dict2  # {'a': 1, 'b': 3, 'c': 4}

# Dictionary views
for key, value in data.items():
    print(f"{key}: {value}")
```
Dictionary operations are fundamental for key-value pair manipulation. They're essential for lookup tables, caching, counting occurrences, and maintaining relationships between data. Common in configuration management, data organization, and caching systems.

## Generator Expressions
```python
# Generator expression
gen = (x**2 for x in range(1000000))

# Memory efficient iteration
for value in gen:
    process(value)

# Pipeline of generators
numbers = (x for x in range(100))
evens = (x for x in numbers if x % 2 == 0)
squares = (x**2 for x in evens)
```
Generator expressions provide memory-efficient iteration for large datasets. They're lazy-evaluated, meaning values are generated only when needed. Perfect for processing large files, data streams, and memory-constrained environments.

This overview covers the most common operations in Python, each with its specific use cases and advantages. Understanding these operations is crucial for writing efficient and pythonic code.