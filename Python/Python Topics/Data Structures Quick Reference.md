## Core Data Types Overview

| `Primary Use`                               | `Ordered` | `Data Type` | `Mutable` | `Creation`                    | `Common Methods`                                                                    | `Special Notes`                                                               |
| ------------------------------------------- | --------- | ----------- | --------- | ----------------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| `Ordered sequences, mutable collections`    | `✅`       | `List`      | `✅`       | `[] list() [1,2,3]`           | `.append() .extend() .pop() .remove() .sort() .index() .count() .insert() .clear()` | `• Allows duplicates • Indexed access • Slicing • Dynamic size`               |
| `Fixed sequences, return values, dict keys` | `✅`       | `Tuple`     | `❌`       | `() tuple() (1,2,3) 1,2,3`    | `.count() .index()`                                                                 | `• Immutable • Can be dict keys • Memory efficient • Faster than lists`       |
| `Mappings, lookups, caches`                 | `✅*`      | `Dict`      | `✅`       | `{} dict() {'a':1} dict(a=1)` | `.keys() .values() .items() .get() .pop() .update() .setdefault()`                  | `• Key-value pairs • Keys must be immutable • Ordered (3.7+) • No duplicates` |
| `Unique collections, membership tests`      | `❌`       | `Set`       | `✅`       | `set() {1,2,3}`               | `.add() .remove() .discard() .union() .intersection() .difference() .update()`      | `• No duplicates • Fast lookup • Math operations • Unordered`                 |
| `Immutable sets, dict keys`                 | `❌`       | `FrozenSet` | `❌`       | `frozenset([1,2,3])`          | `.union() .intersection() .difference()`                                            | `• Immutable • Can be dict key • No add/remove`                               |
| `Text processing`                           | `✅`       | `String`    | `❌`       | `'' "" str() ''''''`          | `.split() .join() .strip() .replace() .upper() .lower() .format()`                  | `• Immutable • Unicode • Slicing • Multi-line with '''`                       |

*Ordered since Python 3.7

## Time Complexity

| Operation | List  | Dict | Set  | String |
| --------- | ----- | ---- | ---- | ------ |
| Access    | O(1)  | O(1) | N/A  | O(1)   |
| Search    | O(n)  | O(1) | O(1) | O(n)   |
| Insert    | O(1)* | O(1) | O(1) | N/A    |
| Delete    | O(n)  | O(1) | O(1) | N/A    |
| Length    | O(1)  | O(1) | O(1) | O(1)   |
| Copy      | O(n)  | O(n) | O(n) | O(n)   |

*O(n) if resizing needed

```python
### Common Gotchas and Patterns ###

# List Gotchas
bad_matrix = [[0] * 3] * 3    # DON'T: Creates references
good_matrix = [[0 for _ in range(3)] for _ in range(3)]  # DO

# Default Arguments
def bad(lst=[]):              # DON'T: Mutable default
def good(lst=None):           # DO
    lst = lst if lst is not None else []

# Dict Access
d['key']                      # DON'T: Can raise KeyError
d.get('key')                  # DO: Returns None
d.get('key', default)         # DO: Returns default

### Quick Operations ###

# List/Tuple Slicing
sequence[start:end:step]      # [inclusive:exclusive:step]
sequence[::-1]               # Reverse
sequence[::2]                # Every second item

# Dict Merging
{**dict1, **dict2}           # Python 3.5+
dict1 | dict2                # Python 3.9+

# Set Operations
s1 | s2                      # Union
s1 & s2                      # Intersection
s1 - s2                      # Difference
s1 ^ s2                      # Symmetric difference

### Type Conversions ###
list(tuple_or_set_or_dict)   # To list
tuple(list_or_set_or_dict)   # To tuple
set(list_or_tuple_or_dict)   # To set
dict(list_of_pairs)          # To dict

### Collection Utilities ###
from collections import Counter, defaultdict, deque

# Counter
word_counts = Counter(['a', 'b', 'a'])

# DefaultDict
d = defaultdict(list)        # Never raises KeyError
d = defaultdict(int)         # Starts count at 0

# Double-ended queue
d = deque(['a', 'b', 'c'])
d.appendleft('x')
d.popleft()

### Memory Tips ###
# Use tuples for small, fixed collections
# Use sets for membership testing
# Use lists for ordered, mutable sequences
# Use dicts for key-value relationships
```

