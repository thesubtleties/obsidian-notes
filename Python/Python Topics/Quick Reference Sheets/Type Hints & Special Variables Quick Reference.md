## Type Hints with Generics
| Type | Usage | Example |
|------|--------|---------|
| `Dict[K, V]` | Dictionary | `Dict[str, int]` |
| `List[T]` | List | `List[str]` |
| `Set[T]` | Set | `Set[int]` |
| `Tuple[T, ...]` | Tuple | `Tuple[int, str, int]` |
| `Optional[T]` | Maybe None | `Optional[str]` |
| `Union[T, U]` | Multiple Types | `Union[str, int]` |
| `Any` | Any Type | `Any` |

```python
from typing import Dict, List, Set, Tuple, Optional, Union, Any

# Function with type hints
def process_data(
    items: List[str],                    # List of strings
    data: Dict[str, int],                # Dict with str keys, int values
    config: Optional[Dict[str, Any]] = None,  # Optional dict with any values
    flags: Set[int] = set(),             # Set of integers
    points: Tuple[int, int] = (0, 0)     # Tuple of two integers
) -> Dict[str, List[int]]:               # Returns dict of string to list of ints
    pass
```

## Common Built-in Types (No brackets needed)
| Type | Usage | Example |
|------|--------|---------|
| `str` | String | `x: str = ""` |
| `int` | Integer | `x: int = 0` |
| `float` | Float | `x: float = 0.0` |
| `bool` | Boolean | `x: bool = True` |

## Protected/Private Naming
```python
class Example:
    def __init__(self):
        self.public = "Public"           # Public
        self._protected = "Protected"     # Protected (convention)
        self.__private = "Private"        # Private (name mangling)
```

## Special Methods (Dunder)
```python
class Example:
    def __init__(self) -> None:        # Constructor
    def __str__(self) -> str:          # String representation
    def __len__(self) -> int:          # Length
    def __call__(self) -> Any:         # Make callable
```

## Common Type Hint Patterns
```python
# Basic container types
numbers: List[int]
pairs: Dict[str, int]
unique: Set[str]
point: Tuple[int, int]

# Nested structures
matrix: List[List[int]]
tree: Dict[str, Dict[str, Any]]
grid: Tuple[Tuple[int, ...], ...]

# Optional and Union
maybe_str: Optional[str]  # Same as Union[str, None]
id_type: Union[int, str]  # Can be int or str

# Functions with type hints
def get_user(
    user_id: Union[int, str]
) -> Optional[Dict[str, Any]]:
    pass

# Classes with type hints
class DataProcessor:
    def __init__(self, data: List[Dict[str, Any]]) -> None:
        self.data = data

    def process(self) -> Dict[str, List[int]]:
        pass
```

## Special Variables (No type hints needed)
```python
__name__: str        # Module name
__file__: str        # File path
__doc__: str         # Documentation string
```

Quick Tips:
- 🔑 Generic types need `[]` with type parameters
- 📝 Basic types (`str`, `int`, etc.) don't need brackets
- 💡 `Optional[T]` is shorthand for `Union[T, None]`
- ⚡ Use `Any` when type is truly unknown
- 🎯 Be as specific as possible with container types
- 🔄 Import complex types from `typing`

Remember:
- Generic containers need type parameters in `[]`
- Basic types don't use brackets
- `None` is used instead of `null`
- Type hints are optional but helpful
- Modern IDEs use type hints for better suggestions
- Python 3.9+ allows `list[str]` instead of `List[str]`

```python
# Python 3.9+ built-in generics
def modern_hints(
    items: list[str],
    data: dict[str, int],
    points: tuple[int, int]
) -> set[str]:
    pass
```