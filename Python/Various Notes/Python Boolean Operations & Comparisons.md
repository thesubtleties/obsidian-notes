## Basic Boolean Values
```python
# Boolean literals
True    # Note the capital T
False   # Note the capital F

# Converting to boolean
bool(1)      # True
bool(0)      # False
bool("")     # False
bool("abc")  # True
bool([])     # False
bool([1,2])  # True
```

## Comparison Operators
```python
# Equality
x == y    # Equal to
x != y    # Not equal to
x is y    # Identity comparison (same object in memory)
x is not y # Negative identity comparison

# Numeric Comparisons
x < y     # Less than
x <= y    # Less than or equal to
x > y     # Greater than
x >= y    # Greater than or equal to
```

## Logical Operators (JavaScript vs Python)
```python
# JavaScript  # Python    # Description
&&           and        # Logical AND
||           or         # Logical OR
!            not        # Logical NOT
??           no direct  # Nullish coalescing

# Examples
True and False    # False
True or False     # True
not True          # False

# Nullish coalescing alternatives
x if x is not None else y
y if x is None else x
```

## Truthy and Falsy Values
```python
# Falsy Values
False
None
0
0.0
""    # Empty string
[]    # Empty list
{}    # Empty dict
()    # Empty tuple
set() # Empty set

# Everything else is Truthy
# Examples of Truthy values
1
-1
"hello"
[1, 2]
{'a': 1}
True
```

## Common Operations & Patterns
```python
# Short-circuit evaluation
x = a and b      # If a is False, returns a; otherwise returns b
y = a or b       # If a is True, returns a; otherwise returns b

# Common patterns
value = some_dict.get('key') or default_value
user_input = input('Name: ') or 'Anonymous'

# Default values (similar to JavaScript ??)
name = user_name if user_name is not None else 'Anonymous'  # None check
name = user_name or 'Anonymous'                            # Falsy check

# Multiple comparisons (unique to Python)
if 0 <= x <= 10:    # Equivalent to: if x >= 0 and x <= 10
    print("x is between 0 and 10")
```

## Key JavaScript vs Python Differences
```python
# Python          # JavaScript
and              # &&
or               # ||
not              # !
is              # ===
is not          # !==
None            # null
if x is None    # if x === null
```

## Common Gotchas
```python
# 'is' vs '=='
# Use 'is' for None, True, False
x is None     # Good
x == None     # Bad practice

# Use '==' for value comparisons
a == 1        # Good
a is 1        # Bad practice - may work but unreliable

# List/Dict Comparisons
[1, 2] == [1, 2]     # True
[1, 2] is [1, 2]     # False (different objects)

# 'or' vs None checking
0 or 'default'          # Returns 'default' (checks falsy)
x if x is not None else 'default'  # Only returns 'default' if x is None
```

## Best Practices
1. Use `is` only for `None`, `True`, and `False` comparisons
2. Use explicit comparisons when possible
3. Use `and`, `or`, `not` instead of symbols
4. Avoid comparing directly to `True` or `False`:
   ```python
   # Good
   if is_valid():
       
   # Less Good
   if is_valid() == True:
   ```
5. Use truthiness for existence checks:
   ```python
   # Good
   if some_list:
       # list is non-empty
   
   # Less Good
   if len(some_list) > 0:
   ```

## Type Checking
```python
# Modern type checking (Python 3.x)
isinstance(x, bool)    # Check if x is boolean
isinstance(x, (int, float))  # Check if x is number
type(x) is bool       # Alternative (less flexible)
```

## Remember
* Python uses words instead of symbols:
```python
and  # instead of &&
or   # instead of ||
not  # instead of !
```

* Identity vs Value comparison:
```python
is   # identity comparison (use for None, True, False)
==   # value comparison (use for everything else)
```

* No direct `??` equivalent - use `if/else` or `or` depending on needs
* Python's boolean operations require proper capitalization (`True`/`False`)
* Python allows chained comparisons (`0 <= x <= 10`)
* Always use `is` when comparing with `None`