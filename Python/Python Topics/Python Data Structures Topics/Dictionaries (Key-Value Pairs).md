## Dictionary Creation
```python
# Basic dictionary creation
person = {
    "name": "John",
    "age": 30,
    "city": "New York"
}

# Alternative creation methods
dict_from_tuples = dict([("a", 1), ("b", 2)])
dict_from_keys = dict.fromkeys(["a", "b", "c"], 0)
empty_dict = {}  # or dict()

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}
```

## Basic Operations
```python
# Accessing values
name = person["name"]              # Direct access
age = person.get("age")           # Safe access
age = person.get("age", 25)       # With default value

# Adding/Updating items
person["email"] = "john@email.com" # Add new key-value
person["age"] = 31                # Update existing
person.update({"phone": "123", "age": 32})  # Update multiple

# Removing items
del person["age"]                 # Remove key-value
phone = person.pop("phone")       # Remove and return value
last_item = person.popitem()      # Remove and return last item
person.clear()                    # Remove all items
```

## Dictionary Methods
```python
# Common methods
keys = person.keys()              # Get all keys
values = person.values()          # Get all values
items = person.items()            # Get all key-value pairs

# Checking keys
exists = "name" in person         # Check if key exists
value = person.setdefault("age", 25)  # Get or set default

# Merging dictionaries
dict1 = {"a": 1, "b": 2}
dict2 = {"b": 3, "c": 4}
merged = {**dict1, **dict2}       # Python 3.5+
merged = dict1 | dict2            # Python 3.9+
```

## Nested Dictionaries
```python
# Creating nested structures
user = {
    "name": "John",
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "zip": "10001"
    },
    "phones": {
        "home": "555-1234",
        "work": "555-5678"
    }
}

# Accessing nested values
city = user["address"]["city"]
# Safe nested access
work_phone = user.get("phones", {}).get("work")
```

## Dictionary Comprehensions
```python
# Basic comprehension
squares = {x: x**2 for x in range(5)}

# Conditional comprehension
even_squares = {x: x**2 for x in range(10) if x % 2 == 0}

# Transform existing dictionary
prices = {"apple": 0.5, "banana": 0.25}
doubled = {k: v*2 for k, v in prices.items()}

# Filtering dictionary
filtered = {k: v for k, v in prices.items() if v > 0.3}
```

## Advanced Dictionary Operations
```python
# Dictionary views
prices = {"apple": 0.5, "banana": 0.25}
keys_view = prices.keys()         # Dynamic view of keys
values_view = prices.values()     # Dynamic view of values
items_view = prices.items()       # Dynamic view of items

# View operations
all_keys = list(keys_view)        # Convert view to list
key_exists = "apple" in keys_view # Membership testing

# Dictionary union (Python 3.9+)
dict1 = {"a": 1, "b": 2}
dict2 = {"c": 3, "d": 4}
combined = dict1 | dict2          # Union operator
```

## Common Patterns
```python
# Grouping data
from collections import defaultdict

# Group by first letter
words = ["apple", "banana", "avocado", "berry"]
grouped = defaultdict(list)
for word in words:
    grouped[word[0]].append(word)

# Counting occurrences
from collections import Counter
word_counts = Counter(words)

# Sorting dictionary
# By keys
sorted_by_key = dict(sorted(person.items()))
# By values
sorted_by_value = dict(sorted(person.items(), key=lambda x: x[1]))
```

## Dictionary Copying
```python
# Shallow copy
original = {"name": "John", "data": [1, 2, 3]}
shallow = original.copy()

# Deep copy
import copy
deep = copy.deepcopy(original)
```

## Error Handling
```python
# Safe key access
try:
    value = person["nonexistent"]
except KeyError:
    value = None

# Better way using get()
value = person.get("nonexistent")
value = person.get("nonexistent", "default")

# Multiple keys
required = ["name", "age", "email"]
try:
    values = [person[k] for k in required]
except KeyError as e:
    print(f"Missing required key: {e}")
```

## Best Practices
```python
# Use get() for safe access
name = person.get("name", "Unknown")

# Dictionary unpacking
defaults = {"port": 8000, "host": "localhost"}
config = {**defaults, **user_config}  # User config overrides defaults

# Type hints (Python 3.5+)
from typing import Dict, List
prices: Dict[str, float] = {"apple": 0.5, "banana": 0.25}
```

## Common Gotchas
```python
# Mutable default values
def bad_defaults(data={}):    # DON'T
    data["key"] = "value"
    return data

def good_defaults(data=None): # DO
    if data is None:
        data = {}
    data["key"] = "value"
    return data

# Key type restrictions
valid = {
    "string": "value",
    (1, 2): "tuple is ok",
    # [1, 2]: "list is NOT ok"  # TypeError: unhashable type
}
```

## Performance Tips
```python
# Membership testing
# Dictionary (O(1))
data = {"key1": 1, "key2": 2}
"key1" in data  # Fast

# Avoid repetitive key lookups
# Bad
if "key" in data:
    value = data["key"]
    # use value

# Good
value = data.get("key")
if value is not None:
    # use value
```

This covers the main aspects of working with dictionaries in Python, from basic operations to advanced usage patterns and best practices.