## Basic Structure
```python
class MyContextManager:
    def __enter__(self):
        # Setup code
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        # Cleanup code
        return True/False  # Controls exception handling
```

## __exit__ Parameters
- `exc_type`: Type of exception raised (if any)
- `exc_value`: Instance of the exception raised
- `traceback`: Traceback object
- All are `None` if no exception occurred

## Return Value Behavior
```python
# Return True: Suppresses exception
def __exit__(self, *args):
    return True  # Exception stops here

# Return False: Propagates exception
def __exit__(self, *args):
    return False  # Exception continues up
```

## Common Usage Patterns

### Basic Resource Management
```python
class FileHandler:
    def __enter__(self):
        self.file = open('file.txt', 'r')
        return self.file

    def __exit__(self, exc_type, exc_value, traceback):
        self.file.close()
        return False  # Let exceptions propagate
```

### Selective Error Handling
```python
class DatabaseConnection:
    def __exit__(self, exc_type, exc_value, traceback):
        self.close()
        if isinstance(exc_value, DatabaseError):
            log_error(exc_value)
            return True  # Handle DB errors
        return False  # Propagate other errors
```

### Error Transformation
```python
class ErrorTransformer:
    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is ValueError:
            raise CustomError("Transformed error") from exc_value
        return False
```

## Detailed Behavior Examples

### Example 1: Suppressing Exceptions (__exit__ returns True)
```python
class MyContextManager:
    def __enter__(self):
        print("Entering context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print(f"Caught error: {exc_type}")
        print("Exiting context")
        return True  # Suppresses the exception

# Using it:
try:
    with MyContextManager():
        print("Inside context")
        raise ValueError("Something went wrong!")
    print("This WILL execute because exception was suppressed")
except ValueError:
    print("This will NOT execute because exception was suppressed")

# Output:
# Entering context
# Inside context
# Caught error: <class 'ValueError'>
# Exiting context
# This WILL execute because exception was suppressed
```

### Example 2: Propagating Exceptions (__exit__ returns False)
```python
class MyContextManager:
    def __enter__(self):
        print("Entering context")
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        print(f"Caught error: {exc_type}")
        print("Exiting context")
        return False  # Lets the exception propagate

# Using it:
try:
    with MyContextManager():
        print("Inside context")
        raise ValueError("Something went wrong!")
    print("This will NOT execute because exception propagated")
except ValueError:
    print("This WILL execute because exception was propagated")

# Output:
# Entering context
# Inside context
# Caught error: <class 'ValueError'>
# Exiting context
# This WILL execute because exception was propagated
```

### Real-World Example: Database Connection Handler
```python
class DatabaseConnection:
    def __enter__(self):
        self.connect()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self.disconnect()
        if isinstance(exc_value, DatabaseError):
            # Handle database-specific errors
            log_error(exc_value)
            return True  # Suppress database errors
        return False  # Let other errors propagate

# Usage
try:
    with DatabaseConnection() as db:
        db.execute_query()
except OtherError:
    # Will only catch non-database errors
    handle_other_error()
```

## Key Points About These Examples
- Example 1 shows complete exception suppression
- Example 2 demonstrates exception propagation
- Real-world example shows practical selective error handling
- All examples ensure cleanup code runs
- Database example shows how to handle specific error types differently

## Common Patterns from Examples
- Error logging before suppression
- Selective error handling based on error type
- Resource cleanup regardless of error state
- Clear separation of setup (__enter__) and teardown (__exit__)
- Explicit error type checking
## Using Context Managers

### Basic Usage
```python
with MyContextManager() as ctx:
    # Protected code block
```

### Try/Except Integration
```python
try:
    with MyContextManager() as ctx:
        # Protected code
except Exception:
    # Only catches if __exit__ returns False
```

### Multiple Contexts
```python
with ContextA() as a, ContextB() as b:
    # Both contexts are active here
```

## Key Points
- `__exit__` always runs, regardless of exceptions
- Return `True` to suppress exceptions
- Return `False` (or None) to propagate exceptions
- Great for resource management (files, DB connections, locks)
- Can handle multiple resources in one `with` statement

## Common Use Cases
```python
# File handling
with open('file.txt') as f:
    data = f.read()

# Database connections
with DatabaseConnection() as db:
    db.execute_query()

# Lock management
with threading.Lock():
    # Thread-safe code
```

## Gotchas
- `__exit__` runs even if `__enter__` fails
- Suppressed exceptions are completely silent
- Multiple contexts are entered left-to-right
- Multiple contexts are exited right-to-left
- Return value from `__exit__` must be boolean (or None)

## Best Practices
- Always close resources in `__exit__`
- Log suppressed exceptions
- Keep context managers focused and single-purpose
- Use meaningful names for context variables
- Consider using `contextlib.contextmanager` for simple cases

## contextlib Alternative
```python
from contextlib import contextmanager

@contextmanager
def simple_context():
    try:
        yield  # Setup code before this
    finally:
        pass  # Cleanup code here
```