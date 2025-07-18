# Refactoring: Static to Instance

#refactoring #patterns #static #instance

## Why Refactor Static to Instance?

Converting static methods to instance methods is common when:
- You need to support multiple configurations
- You want to enable dependency injection
- You need to maintain different states
- You want better testability
- You're removing global state dependencies

## Simple Example: Configuration Service

### Before: Static Pattern
```python
class ConfigService:
    """Static configuration service with global state"""
    
    # Class-level (global) state
    _config = {
        'api_url': 'https://api.example.com',
        'timeout': 30,
        'retries': 3
    }
    
    @staticmethod
    def get(key):
        """Get configuration value"""
        return ConfigService._config.get(key)
    
    @staticmethod
    def set(key, value):
        """Set configuration value"""
        ConfigService._config[key] = value
    
    @staticmethod
    def load_from_file(filepath):
        """Load configuration from file"""
        import json
        with open(filepath) as f:
            ConfigService._config.update(json.load(f))

# Usage - all callers share same config
ConfigService.set('api_url', 'https://api.prod.com')
url = ConfigService.get('api_url')
```

**Problems with this approach:**
- 🚫 Global state - all parts of app share same config
- 🚫 Hard to test - tests affect each other
- 🚫 Can't have multiple configurations
- 🚫 Thread safety issues with concurrent access

### After: Instance Pattern
```python
class ConfigService:
    """Instance-based configuration service"""
    
    def __init__(self, initial_config=None):
        """Initialize with optional config"""
        # Instance-level state (not global!)
        self._config = {
            'api_url': 'https://api.example.com',
            'timeout': 30,
            'retries': 3
        }
        if initial_config:
            self._config.update(initial_config)
    
    def get(self, key):
        """Get configuration value"""
        return self._config.get(key)
    
    def set(self, key, value):
        """Set configuration value"""
        self._config[key] = value
    
    def load_from_file(self, filepath):
        """Load configuration from file"""
        import json
        with open(filepath) as f:
            self._config.update(json.load(f))
    
    def clone(self):
        """Create a copy with same config"""
        return ConfigService(self._config.copy())

# Usage - each instance has its own config
prod_config = ConfigService()
prod_config.set('api_url', 'https://api.prod.com')

test_config = ConfigService({'api_url': 'https://api.test.com'})

# They're independent!
print(prod_config.get('api_url'))  # https://api.prod.com
print(test_config.get('api_url'))  # https://api.test.com
```

**Benefits of instance approach:**
- ✅ No global state - each instance is independent
- ✅ Easy to test - create fresh instances
- ✅ Multiple configurations supported
- ✅ Thread safe with proper instance handling

## What Changes During Refactoring

### 1. **Remove @staticmethod Decorators**
```python
# Before
@staticmethod
def method_name():
    pass

# After
def method_name(self):  # Add self parameter
    pass
```

### 2. **Convert Class Variables to Instance Variables**
```python
# Before
class MyClass:
    shared_data = []  # Class variable

# After  
class MyClass:
    def __init__(self):
        self.shared_data = []  # Instance variable
```

### 3. **Update Method Calls**
```python
# Before
ClassName.static_method()

# After
instance = ClassName()
instance.method()
```

### 4. **Handle State Access**
```python
# Before - accessing class state
@staticmethod
def process():
    return MyClass.data * 2  # Class attribute

# After - accessing instance state
def process(self):
    return self.data * 2  # Instance attribute
```

## Complex Example: Data Processor

### Before: Static with Shared State
```python
class DataProcessor:
    """Static processor with global cache"""
    
    # Shared across all usage
    _cache = {}
    _processed_count = 0
    
    @staticmethod
    def process(data):
        # Check cache (global)
        cache_key = str(data)
        if cache_key in DataProcessor._cache:
            return DataProcessor._cache[cache_key]
        
        # Process data
        result = [x * 2 for x in data]
        
        # Update global state
        DataProcessor._cache[cache_key] = result
        DataProcessor._processed_count += 1
        
        return result
    
    @staticmethod
    def get_stats():
        return {
            'processed': DataProcessor._processed_count,
            'cache_size': len(DataProcessor._cache)
        }
    
    @staticmethod
    def clear_cache():
        DataProcessor._cache.clear()
```

### After: Instance with Dependency Injection
```python
class DataProcessor:
    """Instance processor with injected cache"""
    
    def __init__(self, cache=None, stats_collector=None):
        # Instance state
        self._cache = cache or {}
        self._stats = stats_collector or {'processed': 0}
        
    def process(self, data):
        # Check instance cache
        cache_key = str(data)
        if cache_key in self._cache:
            return self._cache[cache_key]
        
        # Process data
        result = [x * 2 for x in data]
        
        # Update instance state
        self._cache[cache_key] = result
        self._stats['processed'] = self._stats.get('processed', 0) + 1
        
        return result
    
    def get_stats(self):
        return {
            'processed': self._stats.get('processed', 0),
            'cache_size': len(self._cache)
        }
    
    def clear_cache(self):
        self._cache.clear()

# Usage with different configurations
# Production with persistent cache
from redis import Redis
redis_cache = Redis()
prod_processor = DataProcessor(cache=redis_cache)

# Testing with in-memory cache
test_processor = DataProcessor()  # Default dict cache

# Development with no cache
dev_processor = DataProcessor(cache=type('NoCache', (), {
    '__getitem__': lambda s, k: KeyError(),
    '__setitem__': lambda s, k, v: None,
    '__contains__': lambda s, k: False
})())
```

## Migration Strategy

### Step 1: Create Instance Version Alongside Static
```python
class Service:
    # Keep static version temporarily
    @staticmethod
    def static_process(data):
        return data * 2
    
    # Add instance version
    def process(self, data):
        return data * 2
```

### Step 2: Adapter Pattern for Compatibility
```python
class Service:
    def __init__(self):
        self.multiplier = 2
    
    def process(self, data):
        return data * self.multiplier
    
    # Compatibility layer
    _default_instance = None
    
    @classmethod
    def static_process(cls, data):
        """Deprecated: Use instance method instead"""
        if cls._default_instance is None:
            cls._default_instance = cls()
        return cls._default_instance.process(data)
```

### Step 3: Update Call Sites Gradually
```python
# Phase 1: Create instance but use familiar API
service = Service()

# Phase 2: Switch to instance methods
# Old: Service.static_process(data)
# New: service.process(data)

# Phase 3: Remove static methods
```

## Common Pitfalls to Avoid

### 1. **Forgetting to Initialize Instance State**
```python
# BAD - forgot to initialize
class Service:
    def process(self, data):
        return self.cache[data]  # Error! No cache attribute

# GOOD - proper initialization
class Service:
    def __init__(self):
        self.cache = {}
    
    def process(self, data):
        return self.cache.get(data)
```

### 2. **Shared Mutable Default Arguments**
```python
# BAD - mutable default
class Service:
    def __init__(self, cache={}):  # Shared dict!
        self.cache = cache

# GOOD - None default
class Service:
    def __init__(self, cache=None):
        self.cache = cache or {}
```

### 3. **Not Updating All Call Sites**
```python
# Don't forget to update imports and calls
# from service import Service
# Service.method()  # Old static call

# to
# from service import Service  
# service = Service()
# service.method()  # New instance call
```

## Testing Benefits

### Before: Difficult Testing
```python
def test_processor():
    # Static version affects other tests
    DataProcessor.clear_cache()  # Must reset global state
    result = DataProcessor.process([1, 2, 3])
    assert result == [2, 4, 6]
    # Cache is now polluted for other tests
```

### After: Easy Testing
```python
def test_processor():
    # Instance version is isolated
    processor = DataProcessor()  # Fresh instance
    result = processor.process([1, 2, 3])
    assert result == [2, 4, 6]
    # No effect on other tests
    
def test_with_mock_cache():
    # Easy to inject test doubles
    mock_cache = {'[1, 2, 3]': [10, 20, 30]}
    processor = DataProcessor(cache=mock_cache)
    result = processor.process([1, 2, 3])
    assert result == [10, 20, 30]  # Used mock cache
```

## Key Takeaways

> [!important] When Refactoring Static to Instance
> 1. **Add self parameter** to all methods
> 2. **Move class variables** to `__init__`
> 3. **Create instances** before calling methods
> 4. **Enable dependency injection** in constructor
> 5. **Test in isolation** with fresh instances
> 6. **Consider compatibility layer** for gradual migration
> 7. **Update all call sites** throughout codebase