# Refactoring: Instance to Static

#refactoring #patterns #instance #static

## Why Refactor Instance to Static?

Converting instance methods to static methods makes sense when:
- The method doesn't use any instance state (`self`)
- You have pure utility functions
- You want to reduce memory overhead
- The functionality is truly stateless
- You're creating helper/utility classes

> [!warning] Caution
> This refactoring is less common than Static→Instance because it reduces flexibility. Make sure you really don't need instance state before proceeding.

## Simple Example: Math Utilities

### Before: Instance Pattern (Unnecessary)
```python
class MathUtils:
    """Instance-based math utilities (overengineered)"""
    
    def __init__(self):
        # No real instance state needed
        self.operations_count = 0  # Artificial state
    
    def add(self, a, b):
        """Add two numbers"""
        self.operations_count += 1  # Unnecessary tracking
        return a + b
    
    def multiply(self, a, b):
        """Multiply two numbers"""
        self.operations_count += 1
        return a * b
    
    def calculate_area(self, length, width):
        """Calculate rectangle area"""
        self.operations_count += 1
        return length * width

# Usage - creating instances unnecessarily
math = MathUtils()
result = math.add(5, 3)
area = math.calculate_area(10, 20)
```

**Problems with instance approach here:**
- 🚫 Unnecessary object creation
- 🚫 Memory overhead for each instance
- 🚫 No real state to maintain
- 🚫 More complex than needed

### After: Static Pattern
```python
class MathUtils:
    """Static math utilities (simpler)"""
    
    @staticmethod
    def add(a, b):
        """Add two numbers"""
        return a + b
    
    @staticmethod
    def multiply(a, b):
        """Multiply two numbers"""
        return a * b
    
    @staticmethod
    def calculate_area(length, width):
        """Calculate rectangle area"""
        return length * width

# Usage - no instance needed
result = MathUtils.add(5, 3)
area = MathUtils.calculate_area(10, 20)
```

**Benefits of static approach here:**
- ✅ No unnecessary objects
- ✅ Clear that methods are stateless
- ✅ Lower memory usage
- ✅ Simpler to use

## What Changes During Refactoring

### 1. **Remove self Parameter and Add @staticmethod**
```python
# Before
def method_name(self, param1, param2):
    return param1 + param2

# After
@staticmethod
def method_name(param1, param2):  # No self
    return param1 + param2
```

### 2. **Remove Instance State Access**
```python
# Before
def process(self, data):
    return data * self.multiplier  # Uses instance state

# After - pass needed values as parameters
@staticmethod
def process(data, multiplier):
    return data * multiplier  # No instance state
```

### 3. **Update Call Sites**
```python
# Before
instance = MyClass()
result = instance.method(data)

# After
result = MyClass.method(data)
```

## Complex Example: String Formatter

### Before: Instance with Configuration
```python
class StringFormatter:
    """Instance formatter with configuration"""
    
    def __init__(self, 
                 capitalize=False, 
                 strip_whitespace=True,
                 max_length=None):
        self.capitalize = capitalize
        self.strip_whitespace = strip_whitespace
        self.max_length = max_length
    
    def format(self, text):
        """Format text according to configuration"""
        result = text
        
        if self.strip_whitespace:
            result = result.strip()
        
        if self.capitalize:
            result = result.capitalize()
            
        if self.max_length:
            result = result[:self.max_length]
            
        return result
    
    def format_list(self, texts):
        """Format a list of texts"""
        return [self.format(text) for text in texts]

# Usage
formatter = StringFormatter(capitalize=True, max_length=50)
formatted = formatter.format("  hello world  ")
```

### After: Static with Explicit Parameters
```python
class StringFormatter:
    """Static formatter with explicit parameters"""
    
    @staticmethod
    def format(text, 
               capitalize=False, 
               strip_whitespace=True,
               max_length=None):
        """Format text with explicit options"""
        result = text
        
        if strip_whitespace:
            result = result.strip()
        
        if capitalize:
            result = result.capitalize()
            
        if max_length:
            result = result[:max_length]
            
        return result
    
    @staticmethod
    def format_list(texts, **kwargs):
        """Format a list of texts with same options"""
        return [StringFormatter.format(text, **kwargs) for text in texts]
    
    @staticmethod
    def create_formatter(**default_options):
        """Factory for partial application"""
        def formatter(text, **overrides):
            options = {**default_options, **overrides}
            return StringFormatter.format(text, **options)
        return formatter

# Usage - explicit parameters
formatted = StringFormatter.format("  hello world  ", 
                                 capitalize=True, 
                                 max_length=50)

# Or create a configured formatter function
title_formatter = StringFormatter.create_formatter(
    capitalize=True, 
    max_length=50
)
formatted = title_formatter("  hello world  ")
```

## When This Refactoring Makes Sense

### Good Candidates for Instance→Static

#### 1. **Pure Utility Functions**
```python
# Before
class Validator:
    def __init__(self):
        pass  # No state
    
    def is_email(self, text):
        return '@' in text and '.' in text

# After
class Validator:
    @staticmethod
    def is_email(text):
        return '@' in text and '.' in text
```

#### 2. **Calculation Methods**
```python
# Before
class Calculator:
    def __init__(self):
        self.history = []  # Rarely used
    
    def calculate_tax(self, amount, rate):
        result = amount * rate
        self.history.append(result)  # Unnecessary
        return result

# After
class Calculator:
    @staticmethod
    def calculate_tax(amount, rate):
        return amount * rate
```

#### 3. **Data Transformations**
```python
# Before
class DataTransformer:
    def __init__(self):
        pass
    
    def to_uppercase(self, text):
        return text.upper()

# After  
class DataTransformer:
    @staticmethod
    def to_uppercase(text):
        return text.upper()
```

## Migration Strategy

### Step 1: Identify Stateless Methods
```python
class Service:
    def __init__(self, config):
        self.config = config
    
    def process_with_config(self, data):
        # Uses self.config - keep as instance
        return data * self.config['multiplier']
    
    def validate_format(self, data):
        # Doesn't use self - candidate for static
        return isinstance(data, list)
```

### Step 2: Extract State as Parameters
```python
class Service:
    def __init__(self, multiplier):
        self.multiplier = multiplier
    
    # Before - uses instance state
    def calculate(self, value):
        return value * self.multiplier
    
    # After - state as parameter
    @staticmethod
    def calculate_static(value, multiplier):
        return value * multiplier
    
    # Adapter for compatibility
    def calculate(self, value):
        return Service.calculate_static(value, self.multiplier)
```

### Step 3: Gradual Migration
```python
class Service:
    def __init__(self, config=None):
        self.config = config or {}
    
    # New static version
    @staticmethod
    def process_static(data, config):
        return transform(data, config)
    
    # Compatibility wrapper
    def process(self, data):
        """Deprecated: Use process_static instead"""
        return Service.process_static(data, self.config)
```

## Common Pitfalls to Avoid

### 1. **Forcing Static When State is Needed**
```python
# BAD - passing too many parameters
@staticmethod
def process(data, config, cache, logger, validator):
    # Too many parameters! Should be instance

# GOOD - keep as instance when multiple related parameters
def __init__(self, config, cache, logger, validator):
    self.config = config
    self.cache = cache
    self.logger = logger
    self.validator = validator

def process(self, data):
    # Clean interface
```

### 2. **Losing Flexibility**
```python
# Before - flexible
class Processor:
    def __init__(self, strategy):
        self.strategy = strategy
    
    def process(self, data):
        return self.strategy.apply(data)

# After - less flexible
class Processor:
    @staticmethod
    def process(data, strategy):
        return strategy.apply(data)
        
# Now callers must always pass strategy
```

### 3. **Breaking Inheritance**
```python
# Before - can be overridden
class Base:
    def calculate(self, x):
        return x * 2

class Derived(Base):
    def calculate(self, x):
        return x * 3

# After - static methods don't participate in inheritance nicely
class Base:
    @staticmethod
    def calculate(x):
        return x * 2

class Derived(Base):
    @staticmethod
    def calculate(x):  # Doesn't override properly
        return x * 3
```

## Performance Comparison

```python
import timeit

# Instance version
class InstanceMath:
    def add(self, a, b):
        return a + b

# Static version
class StaticMath:
    @staticmethod
    def add(a, b):
        return a + b

# Benchmark instance
instance = InstanceMath()
instance_time = timeit.timeit(
    'instance.add(5, 3)',
    globals=globals(),
    number=1000000
)

# Benchmark static  
static_time = timeit.timeit(
    'StaticMath.add(5, 3)',
    globals=globals(),
    number=1000000
)

print(f"Instance: {instance_time:.4f}s")
print(f"Static: {static_time:.4f}s")
print(f"Static is {instance_time/static_time:.2f}x faster")
```

## Testing Considerations

### Instance Testing (Before)
```python
def test_formatter():
    formatter = StringFormatter(capitalize=True)
    assert formatter.format("hello") == "Hello"
    
    # Can mock easily
    formatter.capitalize = False
    assert formatter.format("hello") == "hello"
```

### Static Testing (After)
```python
def test_formatter():
    # Direct testing
    assert StringFormatter.format("hello", capitalize=True) == "Hello"
    assert StringFormatter.format("hello", capitalize=False) == "hello"
    
    # Less mocking needed but also less flexible
```

## Key Takeaways

> [!important] When Refactoring Instance to Static
> 1. **Ensure no state needed** - method must be pure
> 2. **Add @staticmethod** decorator
> 3. **Remove self parameter** from method signature
> 4. **Pass state as parameters** if needed occasionally
> 5. **Consider flexibility loss** - static is less extensible
> 6. **Update all call sites** to use class name
> 7. **Document why** it's static for future developers