## Basic Parameter Types
| Type | Syntax | Description | Example |
|------|--------|-------------|---------|
| Required | `def func(param)` | Must be provided | `func(10)` |
| Optional | `def func(param=None)` | Has default value | `func()` or `func(10)` |
| *args | `def func(*args)` | Collects extra positional | `func(1,2,3)` |
| Keyword-only | `def func(*, param)` or `def func(*args, param)` | Must use keyword | `func(param=10)` |
| **kwargs | `def func(**kwargs)` | Collects extra keywords (must be last!) | `func(x=1, y=2)` |

## Parameter Order Rules
```python
def correct_order(
    required,         # 1. Required parameters first
    optional=None,    # 2. Optional parameters with defaults
    *args,           # 3. Extra positional collector
    kwonly1,         # 4. Keyword-only parameters (after *args)
    kwonly2,         # 4. More keyword-only parameters
    **kwargs         # 5. Extra keyword collector (MUST be last!)
): pass

# Example usage:
correct_order(
    "required_val",       # required
    "optional_val",       # optional
    1, 2, 3,             # collected into args
    kwonly1="must_name", # must use keyword
    kwonly2="also_name", # must use keyword
    extra1="extra",      # collected into kwargs
    extra2="more"        # collected into kwargs
)
```

## Common Patterns
```python
# Collect variable positional args
def sum_all(*numbers):
    return sum(numbers)
sum_all(1, 2, 3)  # numbers = (1, 2, 3)

# Force keyword arguments
def user_info(*, name, age):  # using *,
    return f"{name} is {age}"
user_info(name="Bob", age=25)  # Must use keywords

# Collect extra keywords
def config(host, **options):  # kwargs must be last
    print(f"Host: {host}")
    print(f"Extra options: {options}")
config("localhost", port=8000, debug=True)
```

## Two Ways to Force Keyword-Only
```python
# Method 1: Using *args
def func1(*args, must_be_keyword):
    print(f"args: {args}")
    print(f"keyword: {must_be_keyword}")

# Method 2: Using just *
def func2(*, must_be_keyword):
    print(f"keyword: {must_be_keyword}")

# Both called the same way:
func1(1, 2, must_be_keyword="value")
func2(must_be_keyword="value")
```

## Real World Example
```python
def api_call(
    endpoint,          # Required
    method="GET",      # Optional
    *args,            # Catch any positional args
    headers=None,     # Keyword-only (after *args)
    params=None,      # Keyword-only
    **extra           # Extra keyword arguments (last!)
):
    # Check for unwanted positional args
    if args:
        raise ValueError("Use keyword arguments!")
    
    return {
        "endpoint": endpoint,
        "method": method,
        "headers": headers or {},
        "params": params or {},
        **extra  # Unpack any extra options
    }

# Usage:
api_call(
    "/users",
    headers={"Authorization": "Bearer token"},
    params={"user_id": 123},
    timeout=30  # Goes into extra
)
```

Quick Tips:
- 📌 `*args` collects extra positional args as tuple
- 🔑 `**kwargs` must always be the last parameter
- ⭐ Everything after `*args` is keyword-only
- 🔄 Use `*,` when you don't need to collect positional args
- 💡 Parameter order is strictly enforced by Python

Remember:
- `**kwargs` must be last parameter
- Everything after `*args` requires keywords
- Can't put positional parameters after keyword parameters
- `*args` and `**kwargs` are conventional names but not required
- Use `*,` to force keyword-only without collecting args