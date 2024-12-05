## Numbers
```python
# Integers (unlimited size)
age = 25
big_num = 1_000_000      # Underscores for readability
hex_num = 0xFF           # Hexadecimal
oct_num = 0o755          # Octal
bin_num = 0b1010         # Binary

# Floating Point
pi = 3.14159
scientific = 1.23e-4     # Scientific notation
infinity = float('inf')  # Infinity
not_number = float('nan')# NaN

# Complex Numbers
complex_num = 3 + 4j     # Complex number
magnitude = abs(complex_num)  # Get magnitude
```
Python handles different number types automatically. Unlike JavaScript, integers have unlimited precision, and there's built-in support for complex numbers.

## Strings
```python
# String Creation
single = 'Single quotes'
double = "Double quotes"  # Both work the same
multi = """
    Multiple
    lines
"""

# String Operations
name = "Python"
print(len(name))         # Length: 6
print(name[0])          # First character: 'P'
print(name[-1])         # Last character: 'n'
print(name[1:4])        # Slice: 'yth'
print(name[::-1])       # Reverse: 'nohtyP'

# String Methods
text = "  hello, world  "
print(text.strip())         # Remove whitespace
print(text.upper())         # "  HELLO, WORLD  "
print(text.capitalize())    # "  Hello, world  "
print(text.replace('l', 'L'))  # Replace characters
print(','.join(['a', 'b'])) # Join with delimiter

# String Formatting
name = "Alice"
age = 30
# f-strings (Python 3.6+)
print(f"Name: {name}, Age: {age}")
# format() method
print("Name: {}, Age: {}".format(name, age))
# % operator (older style)
print("Name: %s, Age: %d" % (name, age))
```
Strings in Python are immutable and have many built-in methods for manipulation. F-strings are the modern way to format strings.

## Boolean & None
```python
# Boolean values
is_active = True
is_done = False

# Boolean operations
print(True and False)    # False
print(True or False)     # True
print(not True)          # False

# Truthy and Falsy values
bool("")        # False
bool([])        # False
bool(0)         # False
bool(None)      # False
bool("text")    # True
bool([1, 2])    # True
bool(1)         # True

# None type (similar to null)
value = None
if value is None:        # Proper way to check for None
    print("No value")
```
Python uses `True`/`False` (capitalized) and `None` instead of JavaScript's `true`/`false` and `null`/`undefined`.

## Lists
```python
# List creation
numbers = [1, 2, 3, 4, 5]
mixed = [1, "two", 3.0, [4, 5]]

# List operations
numbers.append(6)        # Add to end
numbers.insert(0, 0)     # Insert at index
numbers.pop()           # Remove and return last item
numbers.remove(3)       # Remove first occurrence of value
del numbers[0]          # Delete at index

# List slicing
print(numbers[1:4])     # Elements 1 through 3
print(numbers[::2])     # Every second element
print(numbers[::-1])    # Reverse list

# List comprehension
squares = [x**2 for x in range(5)]
evens = [x for x in range(10) if x % 2 == 0]

# List methods
nums = [3, 1, 4, 1, 5, 9]
nums.sort()             # Sort in place
nums.reverse()          # Reverse in place
print(nums.count(1))    # Count occurrences
print(nums.index(4))    # Find index of value
```
Lists are mutable sequences, similar to JavaScript arrays but with more built-in functionality.

## Dictionaries
```python
# Dictionary creation
person = {
    "name": "John",
    "age": 30,
    "skills": ["Python", "JavaScript"]
}

# Dictionary operations
person["location"] = "NYC"    # Add/update item
del person["age"]            # Remove item
value = person.get("age", 0) # Get with default

# Dictionary methods
keys = person.keys()         # Get all keys
values = person.values()     # Get all values
items = person.items()       # Get key-value pairs

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}

# Nested dictionaries
complex_dict = {
    "user": {
        "name": "John",
        "settings": {
            "theme": "dark"
        }
    }
}
```
Dictionaries are Python's equivalent of JavaScript objects, with similar key-value pair functionality.

## Sets
```python
# Set creation
unique_nums = {1, 2, 3, 3}   # Duplicates removed
empty_set = set()            # Creating empty set

# Set operations
unique_nums.add(4)           # Add element
unique_nums.remove(2)        # Remove element
unique_nums.discard(5)       # Remove if exists

# Set mathematics
set1 = {1, 2, 3}
set2 = {3, 4, 5}
print(set1 | set2)          # Union
print(set1 & set2)          # Intersection
print(set1 - set2)          # Difference
print(set1 ^ set2)          # Symmetric difference
```
Sets are unordered collections of unique elements, useful for removing duplicates and set operations.

## Tuples
```python
# Tuple creation
coordinates = (1, 2, 3)      # Immutable sequence
single_tuple = (1,)          # Single-item tuple needs comma
nested = (1, (2, 3), 4)     # Nested tuples

# Tuple unpacking
x, y, z = coordinates       # Unpacking values
first, *rest = (1, 2, 3, 4) # Rest operator

# Tuple methods
coords = (5, 1, 5, 9, 5)
print(coords.count(5))      # Count occurrences
print(coords.index(9))      # Find index
```
Tuples are immutable sequences, often used for grouping related values that shouldn't change.

## Type Conversion
```python
# Converting between types
str_num = "123"
num = int(str_num)          # String to int
float_num = float(str_num)  # String to float
text = str(123)             # Number to string
tuple_list = tuple([1,2,3]) # List to tuple
list_tuple = list((1,2,3))  # Tuple to list
set_list = set([1,1,2,2,3]) # List to set (removes duplicates)
```

## Common Gotchas
```python
# List reference vs copy
list1 = [1, 2, 3]
list2 = list1           # Reference, not copy
list2.append(4)         # Modifies both lists

# Proper copying
shallow_copy = list1.copy()  # or list1[:]
import copy
deep_copy = copy.deepcopy(list1)

# Mutable vs Immutable
# Strings, numbers, and tuples are immutable
# Lists, dictionaries, and sets are mutable

# String concatenation
bad = ""
for i in range(1000):
    bad += str(i)       # Inefficient

good = []
for i in range(1000):
    good.append(str(i))
result = "".join(good)  # More efficient
```

This overview covers the main data types in Python. Each type has its own methods and use cases, and understanding their properties (especially mutability) is crucial for effective Python programming.