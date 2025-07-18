# Refactoring: Singleton to Instance

#refactoring #patterns #singleton #instance

## Why Refactor Singleton to Instance?

Converting singleton to instance pattern is important when:
- You need multiple independent instances
- You want to improve testability
- You need different configurations for different parts of the app
- You want to eliminate global state
- You're experiencing concurrency issues
- You need better isolation between components

> [!tip] Common Scenario
> This is one of the most common refactorings as applications grow and singleton limitations become apparent.

## Simple Example: Cache Service

### Before: Singleton Pattern
```python
class CacheService:
    """Singleton cache service"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized'):
            return
        
        self._cache = {}
        self._max_size = 1000
        self._initialized = True
    
    def get(self, key):
        return self._cache.get(key)
    
    def set(self, key, value):
        if len(self._cache) >= self._max_size:
            # Remove oldest (simplified)
            first_key = next(iter(self._cache))
            del self._cache[first_key]
        
        self._cache[key] = value
    
    def clear(self):
        self._cache.clear()

# Usage - global shared cache
# module1.py
cache = CacheService()
cache.set('user:123', user_data)

# module2.py
cache = CacheService()  # Same instance
user = cache.get('user:123')  # Sees module1's data
```

**Problems with singleton:**
- 🚫 All modules share same cache
- 🚫 Can't have separate caches for different purposes
- 🚫 Testing requires clearing global state
- 🚫 No isolation between features

### After: Instance Pattern
```python
class CacheService:
    """Instance-based cache service"""
    
    def __init__(self, max_size=1000, name=None):
        """Initialize independent cache instance"""
        self._cache = {}
        self._max_size = max_size
        self.name = name or f"cache_{id(self)}"
        self._stats = {
            'hits': 0,
            'misses': 0,
            'evictions': 0
        }
    
    def get(self, key):
        if key in self._cache:
            self._stats['hits'] += 1
            return self._cache[key]
        else:
            self._stats['misses'] += 1
            return None
    
    def set(self, key, value):
        if len(self._cache) >= self._max_size:
            # Remove oldest (simplified)
            first_key = next(iter(self._cache))
            del self._cache[first_key]
            self._stats['evictions'] += 1
        
        self._cache[key] = value
    
    def clear(self):
        self._cache.clear()
    
    def get_stats(self):
        return {
            'name': self.name,
            'size': len(self._cache),
            'max_size': self._max_size,
            **self._stats
        }

# Usage - independent caches
# user_service.py
user_cache = CacheService(max_size=500, name='users')
user_cache.set('user:123', user_data)

# product_service.py  
product_cache = CacheService(max_size=2000, name='products')
product_cache.set('product:456', product_data)

# They're independent
print(user_cache.get('product:456'))  # None - different cache
print(product_cache.get_stats())  # Separate statistics
```

**Benefits of instance pattern:**
- ✅ Independent caches for different purposes
- ✅ Configurable per instance
- ✅ Easy testing with fresh instances
- ✅ Better separation of concerns

## What Changes During Refactoring

### 1. **Remove Singleton Mechanism**
```python
# Remove these singleton elements:
# - _instance class variable
# - __new__ override  
# - _lock for thread safety
# - Initialization guard in __init__
```

### 2. **Convert to Standard Constructor**
```python
# Before
def __init__(self):
    if hasattr(self, '_initialized'):
        return
    # ... initialization
    self._initialized = True

# After
def __init__(self, config=None):
    # Direct initialization, no guards
    self.config = config or {}
    # ... rest of initialization
```

### 3. **Add Dependency Injection**
```python
# Enable configuration through constructor
def __init__(self, 
             database=None,
             cache=None,
             logger=None):
    self.database = database or DefaultDatabase()
    self.cache = cache or {}
    self.logger = logger or get_logger()
```

### 4. **Update Client Code**
```python
# Before - implicit singleton
service = ServiceClass()

# After - explicit instance creation
service = ServiceClass(config={...})
```

## Complex Example: Configuration Manager

### Before: Singleton Configuration
```python
import json
import os

class ConfigManager:
    """Singleton configuration manager"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized'):
            return
            
        self._config = {}
        self._load_config()
        self._initialized = True
    
    def _load_config(self):
        """Load from multiple sources"""
        # Default config
        self._config.update({
            'debug': False,
            'timeout': 30
        })
        
        # Environment variables
        for key, value in os.environ.items():
            if key.startswith('APP_'):
                config_key = key[4:].lower()
                self._config[config_key] = value
        
        # Config file
        if os.path.exists('config.json'):
            with open('config.json') as f:
                self._config.update(json.load(f))
    
    def get(self, key, default=None):
        return self._config.get(key, default)
    
    def set(self, key, value):
        self._config[key] = value

# Problem: All parts of app share same config
config = ConfigManager()
config.set('debug', True)  # Affects entire application!
```

### After: Instance-Based Configuration
```python
import json
import os
from typing import Dict, Any, Optional

class ConfigManager:
    """Instance-based configuration manager"""
    
    def __init__(self, 
                 sources: Optional[List[str]] = None,
                 defaults: Optional[Dict[str, Any]] = None,
                 env_prefix: str = 'APP_',
                 mutable: bool = True):
        """
        Initialize configuration manager
        
        Args:
            sources: List of config file paths
            defaults: Default configuration values
            env_prefix: Prefix for environment variables
            mutable: Whether config can be modified after loading
        """
        self._config = {}
        self._defaults = defaults or {}
        self._sources = sources or []
        self._env_prefix = env_prefix
        self._mutable = mutable
        self._frozen = False
        self._load_config()
    
    def _load_config(self):
        """Load configuration from all sources"""
        # Start with defaults
        self._config.update(self._defaults)
        
        # Load from files
        for source in self._sources:
            if os.path.exists(source):
                with open(source) as f:
                    if source.endswith('.json'):
                        self._config.update(json.load(f))
                    elif source.endswith('.yaml'):
                        import yaml
                        self._config.update(yaml.safe_load(f))
        
        # Override with environment variables
        if self._env_prefix:
            for key, value in os.environ.items():
                if key.startswith(self._env_prefix):
                    config_key = key[len(self._env_prefix):].lower()
                    self._config[config_key] = self._parse_env_value(value)
    
    def _parse_env_value(self, value: str) -> Any:
        """Parse environment variable value"""
        # Try to parse as JSON for complex types
        try:
            return json.loads(value)
        except:
            # Check for boolean
            if value.lower() in ('true', 'false'):
                return value.lower() == 'true'
            # Check for number
            try:
                return int(value)
            except ValueError:
                try:
                    return float(value)
                except ValueError:
                    return value
    
    def get(self, key: str, default: Any = None) -> Any:
        """Get configuration value"""
        return self._config.get(key, default)
    
    def set(self, key: str, value: Any) -> None:
        """Set configuration value (if mutable)"""
        if self._frozen or not self._mutable:
            raise RuntimeError("Configuration is immutable")
        self._config[key] = value
    
    def freeze(self) -> None:
        """Make configuration immutable"""
        self._frozen = True
    
    def clone(self) -> 'ConfigManager':
        """Create a copy of this configuration"""
        cloned = ConfigManager(defaults=self._config.copy(), 
                             mutable=self._mutable)
        return cloned
    
    def get_namespace(self, prefix: str) -> Dict[str, Any]:
        """Get all config values with given prefix"""
        return {
            key[len(prefix):]: value
            for key, value in self._config.items()
            if key.startswith(prefix)
        }

# Usage - different configs for different components

# Application config
app_config = ConfigManager(
    sources=['config/app.json'],
    defaults={'debug': False, 'port': 8000},
    env_prefix='APP_'
)

# Database config  
db_config = ConfigManager(
    sources=['config/database.json'],
    defaults={'host': 'localhost', 'port': 5432},
    env_prefix='DB_'
)

# Test config (immutable)
test_config = ConfigManager(
    defaults={'debug': True, 'use_mocks': True},
    mutable=False
)
test_config.freeze()

# Feature-specific config
feature_config = app_config.get_namespace('feature_')
```

## Migration Strategy

### Step 1: Create Factory Function
```python
# Add factory alongside singleton
class Service:
    _instance = None
    
    def __new__(cls, *args, **kwargs):
        # Keep singleton for backward compatibility
        if cls._instance is None:
            cls._instance = object.__new__(cls)
        return cls._instance
    
    @classmethod
    def create_instance(cls, *args, **kwargs):
        """Factory method for new instances"""
        instance = object.__new__(cls)
        instance.__init__(*args, **kwargs)
        return instance
```

### Step 2: Dependency Injection Container
```python
class ServiceContainer:
    """Manage service instances"""
    
    def __init__(self):
        self._services = {}
        self._factories = {}
    
    def register_singleton(self, name, service):
        """Register existing singleton"""
        self._services[name] = service
    
    def register_factory(self, name, factory):
        """Register factory for instances"""
        self._factories[name] = factory
    
    def get(self, name):
        """Get service by name"""
        if name in self._services:
            return self._services[name]
        elif name in self._factories:
            return self._factories[name]()
        else:
            raise KeyError(f"Service '{name}' not found")

# Migration usage
container = ServiceContainer()

# Register existing singleton
container.register_singleton('legacy_cache', CacheService())

# Register factory for new instances  
container.register_factory('cache', lambda: CacheService.create_instance())

# Gradual migration
legacy = container.get('legacy_cache')  # Singleton
new = container.get('cache')  # New instance
```

### Step 3: Update Client Code Gradually
```python
# Phase 1: Identify singleton usage
# Search for: ClassName() patterns

# Phase 2: Add explicit instance creation
# Old
cache = CacheService()

# New  
cache = CacheService.create_instance()  # Or
cache = container.get('cache')

# Phase 3: Remove singleton mechanism
# Once all clients updated, remove __new__ override
```

## Common Pitfalls to Avoid

### 1. **Shared State Dependencies**
```python
# Code may depend on singleton's shared state
# BAD - depends on global state
def process_user(user_id):
    cache = CacheService()  # Expects singleton
    return cache.get(f'user:{user_id}')

# GOOD - accepts cache as parameter
def process_user(user_id, cache):
    return cache.get(f'user:{user_id}')
```

### 2. **Initialization Order Issues**
```python
# Singletons often hide initialization order dependencies
# May need to make explicit

# Before - hidden dependency
class ServiceA:
    def __init__(self):
        self.b = ServiceB()  # Assumes B is ready

# After - explicit dependency
class ServiceA:
    def __init__(self, service_b):
        self.b = service_b  # Explicit dependency
```

### 3. **Resource Cleanup**
```python
# Singletons often lack proper cleanup
# Add proper lifecycle management

class ResourceManager:
    def __init__(self):
        self.resources = []
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.cleanup()
    
    def cleanup(self):
        for resource in self.resources:
            resource.close()
        self.resources.clear()
```

## Testing Benefits

### Before: Difficult Singleton Testing
```python
class TestWithSingleton(unittest.TestCase):
    def setUp(self):
        # Must reset global state
        CacheService._instance = None
        
    def tearDown(self):
        # Must clean up global state
        if CacheService._instance:
            CacheService._instance.clear()
    
    def test_caching(self):
        cache = CacheService()
        cache.set('key', 'value')
        # Other tests might see this data!
```

### After: Easy Instance Testing
```python
class TestWithInstance(unittest.TestCase):
    def test_caching(self):
        # Fresh instance for each test
        cache = CacheService()
        cache.set('key', 'value')
        self.assertEqual(cache.get('key'), 'value')
        # No cleanup needed - instance is garbage collected
    
    def test_independent_caches(self):
        cache1 = CacheService(max_size=10)
        cache2 = CacheService(max_size=20)
        
        cache1.set('key', 'value1')
        cache2.set('key', 'value2')
        
        self.assertEqual(cache1.get('key'), 'value1')
        self.assertEqual(cache2.get('key'), 'value2')
```

## Key Takeaways

> [!important] When Refactoring Singleton to Instance
> 1. **Remove singleton mechanism** - `__new__`, `_instance`, locks
> 2. **Add configuration parameters** to constructor
> 3. **Enable dependency injection** for flexibility
> 4. **Create factory methods** for gradual migration
> 5. **Update client code** to create instances explicitly
> 6. **Add lifecycle management** for resource cleanup
> 7. **Improve testability** with independent instances