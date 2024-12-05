## Counter
```python
from collections import Counter

# Creating Counters
word_counts = Counter(['a', 'b', 'a', 'c', 'b', 'a'])
text_counts = Counter("mississippi")
dict_counts = Counter({'a': 3, 'b': 2})

# Common operations
print(word_counts['a'])          # Count of 'a'
print(word_counts.most_common(2)) # Two most common elements
print(list(word_counts.elements())) # Iterator over elements
```
Counter provides a dict subclass for counting hashable objects, perfect for counting occurrences.

## defaultdict
```python
from collections import defaultdict

# Different default factories
int_dict = defaultdict(int)      # Default value: 0
list_dict = defaultdict(list)    # Default value: []
set_dict = defaultdict(set)      # Default value: set()

# Usage examples
# Group items by key
groups = defaultdict(list)
for name, group in [("A", 1), ("B", 2), ("A", 3)]:
    groups[name].append(group)

# Count occurrences
counts = defaultdict(int)
for word in ["apple", "banana", "apple"]:
    counts[word] += 1
```
defaultdict automatically initializes new keys with a default value, eliminating key error checks.

## OrderedDict
```python
from collections import OrderedDict

# Create ordered dictionary
ordered = OrderedDict()
ordered['a'] = 1
ordered['b'] = 2
ordered['c'] = 3

# Compare dictionaries
dict1 = OrderedDict({'a': 1, 'b': 2})
dict2 = OrderedDict({'b': 2, 'a': 1})
print(dict1 == dict2)  # False (order matters)

# Move to end
ordered.move_to_end('a')  # Move 'a' to end
ordered.move_to_end('b', last=False)  # Move 'b' to start
```
OrderedDict remembers the order of keys added (less important since Python 3.7 where regular dicts maintain order).

## ChainMap
```python
from collections import ChainMap

# Create chain of mappings
defaults = {'color': 'red', 'size': 'medium'}
user_prefs = {'color': 'blue'}
combined = ChainMap(user_prefs, defaults)

# Lookup precedence
print(combined['color'])  # 'blue' (from user_prefs)
print(combined['size'])   # 'medium' (from defaults)

# Modify mappings
combined['color'] = 'green'  # Only modifies first mapping
```
ChainMap combines multiple dictionaries or mappings into a single view.

## namedtuple
```python
from collections import namedtuple

# Define named tuple
Point = namedtuple('Point', ['x', 'y'])
Person = namedtuple('Person', 'name age city')

# Create instances
p = Point(11, y=22)
john = Person('John', 30, 'NYC')

# Access values
print(p.x, p.y)          # Using attributes
print(john[0])           # Using index

# Convert to dictionary
person_dict = john._asdict()

# Create new instance with changes
jane = john._replace(name='Jane')
```
namedtuple creates tuple subclasses with named fields, providing a lightweight alternative to classes.

## deque (Double-Ended Queue)
```python
from collections import deque

# Create deque
d = deque(maxlen=3)  # Fixed-size deque
d = deque([1, 2, 3])  # From iterable

# Operations on both ends
d.append(4)          # Add to right
d.appendleft(0)      # Add to left
d.pop()              # Remove from right
d.popleft()          # Remove from left

# Rotation
d.rotate(1)          # Rotate right
d.rotate(-1)         # Rotate left

# Extend operations
d.extend([4, 5, 6])  # Extend right
d.extendleft([0, -1, -2])  # Extend left
```
deque provides fast O(1) append/pop operations from both ends.

## UserDict, UserList, and UserString
```python
from collections import UserDict, UserList, UserString

# Custom dictionary
class CustomDict(UserDict):
    def __setitem__(self, key, value):
        super().__setitem__(key.lower(), value)

# Custom list
class CustomList(UserList):
    def append(self, item):
        super().append(str(item))

# Custom string
class CustomString(UserString):
    def reverse(self):
        return self.data[::-1]
```
These classes are wrappers around built-in types, making it easier to create custom versions.

## Advanced Usage Patterns
```python
# Counting with conditions
def count_with_condition(items, condition):
    return Counter(x for x in items if condition(x))

# Multi-level defaultdict
def nested_dict():
    return defaultdict(nested_dict)

tree = nested_dict()
tree[1][2][3] = "value"

# Combining multiple configurations
def merge_configs(*configs):
    return ChainMap(*configs)

# Priority queue with counter
class PriorityCounter:
    def __init__(self):
        self.counter = Counter()
        self.order = deque()

    def add(self, item):
        self.counter[item] += 1
        if item not in self.order:
            self.order.append(item)
```
Advanced collections can be combined for more complex data structures.

## Performance Considerations
```python
# Counter vs manual counting
def count_performance(items):
    # Using Counter (faster)
    counter = Counter(items)
    
    # Manual counting (slower)
    manual = {}
    for item in items:
        manual[item] = manual.get(item, 0) + 1

# defaultdict vs dict.setdefault
def grouping_performance(items):
    # Using defaultdict (faster)
    groups = defaultdict(list)
    for k, v in items:
        groups[k].append(v)
    
    # Using setdefault (slower)
    manual = {}
    for k, v in items:
        manual.setdefault(k, []).append(v)
```
Choose the right collection type for better performance.

## Error Handling
```python
def safe_counter_operations(items):
    try:
        counts = Counter(items)
        return counts.most_common()
    except TypeError:
        return None

def safe_chain_lookup(chain, key, default=None):
    try:
        return chain[key]
    except KeyError:
        return default

def safe_deque_ops(d, maxlen=None):
    try:
        return deque(d, maxlen=maxlen)
    except (TypeError, ValueError):
        return deque(maxlen=maxlen)
```
Always handle potential errors when working with collections.

## Best Practices
```python
# Use Counter for counting
def word_frequency(text):
    return Counter(text.lower().split())

# Use defaultdict for grouping
def group_by_category(items):
    groups = defaultdict(list)
    for item, category in items:
        groups[category].append(item)
    return dict(groups)

# Use ChainMap for configuration
def create_config(user_config, default_config):
    return ChainMap(user_config, default_config)

# Use namedtuple for structured data
def create_data_structure():
    DataPoint = namedtuple('DataPoint', ['x', 'y', 'label'])
    return DataPoint
```
Choose the appropriate collection type based on your specific needs.

This covers the main aspects of Python's advanced collections, including their creation, usage patterns, and best practices.