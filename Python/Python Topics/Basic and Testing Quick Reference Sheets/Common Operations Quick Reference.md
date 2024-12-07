## Basic Operations
| Command/Operation | Description | Example |
|------------------|-------------|----------|
| `len()` | Get length | `len([1,2,3])` |
| `type()` | Get type | `type(variable)` |
| `print()` | Print output | `print(f"{var=}")` |
| `range()` | Create sequence | `range(0, 10, 2)` |
| `enumerate()` | Index iteration | `for i, val in enumerate(list)` |
| `zip()` | Combine iterables | `for x, y in zip(a, b)` |

## String Operations
| Operation | Description | Example |
|-----------|-------------|----------|
| `str.split()` | Split string | `"a,b,c".split(',')` |
| `str.join()` | Join strings | `",".join(['a','b'])` |
| `str.strip()` | Remove whitespace | `" text ".strip()` |
| `str.replace()` | Replace text | `"hi!".replace('!','?')` |
| `str.format()` | Format string | `"{} {}".format(1,2)` |
| `f""` | f-string | `f"Value: {x}"` |

## List Operations
| Operation | Description | Example |
|-----------|-------------|----------|
| `list.append()` | Add item | `lst.append(item)` |
| `list.extend()` | Add iterable | `lst.extend([1,2])` |
| `list.pop()` | Remove & return | `lst.pop()` |
| `list.sort()` | Sort in place | `lst.sort()` |
| `sorted()` | Return sorted | `sorted(lst)` |
| `list.reverse()` | Reverse in place | `lst.reverse()` |

## Dictionary Operations
| Operation | Description | Example |
|-----------|-------------|----------|
| `dict.get()` | Safe get | `d.get('key', default)` |
| `dict.keys()` | Get keys | `d.keys()` |
| `dict.values()` | Get values | `d.values()` |
| `dict.items()` | Get pairs | `d.items()` |
| `dict.update()` | Update dict | `d.update({'a': 1})` |
| `dict.pop()` | Remove & return | `d.pop('key')` |

## File Operations
| Operation | Description | Example |
|-----------|-------------|----------|
| `open()` | Open file | `with open('f.txt') as f:` |
| `read()` | Read all | `f.read()` |
| `readline()` | Read line | `f.readline()` |
| `write()` | Write to file | `f.write('text')` |
| `close()` | Close file | `f.close()` |

## Error Handling
| Operation | Description | Example |
|-----------|-------------|----------|
| `try/except` | Basic handling | `try: ... except Error:` |
| `raise` | Raise error | `raise ValueError()` |
| `finally` | Cleanup | `try: ... finally:` |
| `assert` | Assertion | `assert x > 0` |

## Common Built-ins
| Function | Description | Example |
|----------|-------------|----------|
| `map()` | Map function | `map(fn, iterable)` |
| `filter()` | Filter items | `filter(fn, iterable)` |
| `lambda` | Anonymous function | `lambda x: x*2` |
| `any()` | Any true | `any([True, False])` |
| `all()` | All true | `all([True, True])` |

## List Comprehensions
| Type | Example |
|------|----------|
| Basic | `[x for x in range(10)]` |
| Filtered | `[x for x in range(10) if x%2==0]` |
| Nested | `[(x,y) for x in a for y in b]` |
| Dict | `{x: x**2 for x in range(5)}` |

## Common Modules
| Import | Use Case | Example |
|--------|----------|----------|
| `datetime` | Date/Time | `from datetime import datetime` |
| `json` | JSON handling | `import json` |
| `os` | OS operations | `import os` |
| `sys` | System specific | `import sys` |
| `random` | Random numbers | `import random` |
| `re` | Regular expressions | `import re` |

Quick Tips:
- 🔄 Use `with` for file operations
- 💡 f-strings for string formatting
- 🎯 List comprehensions for clarity
- 🔑 `dict.get()` for safe access
- 🐍 Type hints for better code
- 🎨 Use `enumerate()` for counters

Common Patterns:
```python
# Safe dict access
value = data.get('key', default_value)

# File reading
with open('file.txt', 'r') as f:
    content = f.read()

# Error handling
try:
    result = risky_operation()
except Exception as e:
    logging.error(f"Failed: {e}")

# List comprehension with condition
evens = [x for x in range(10) if x % 2 == 0]

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}
```