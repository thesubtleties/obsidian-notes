# Refactoring: Instance to Singleton

#refactoring #patterns #instance #singleton

## Why Refactor Instance to Singleton?

Converting instance pattern to singleton makes sense when:
- You need exactly one instance throughout the application
- You're managing expensive resources (database connections, thread pools)
- You need global access to shared state
- You want to control resource allocation
- You're implementing caching or configuration services

> [!warning] Think Carefully
> Singletons introduce global state and can make testing harder. Make sure you really need the singleton pattern before refactoring.

## Simple Example: Logger Service

### Before: Instance Pattern
```python
class Logger:
    """Instance-based logger"""
    
    def __init__(self, log_file='app.log', level='INFO'):
        self.log_file = log_file
        self.level = level
        self.file_handle = open(log_file, 'a')
        
    def log(self, message, level='INFO'):
        if self._should_log(level):
            timestamp = datetime.now().isoformat()
            self.file_handle.write(f"[{timestamp}] {level}: {message}\n")
            self.file_handle.flush()
    
    def _should_log(self, level):
        levels = ['DEBUG', 'INFO', 'WARNING', 'ERROR']
        return levels.index(level) >= levels.index(self.level)
    
    def close(self):
        self.file_handle.close()

# Usage - multiple instances problem
# module1.py
logger1 = Logger('app.log')
logger1.log("Starting module 1")

# module2.py  
logger2 = Logger('app.log')  # Another file handle!
logger2.log("Starting module 2")

# Problems: Multiple file handles to same file
# Resource waste and potential conflicts
```

**Problems with multiple instances:**
- 🚫 Multiple file handles to same file
- 🚫 Inconsistent configuration
- 🚫 Resource waste
- 🚫 No centralized control

### After: Singleton Pattern
```python
import threading

class Logger:
    """Singleton logger"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                # Double-check locking pattern
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, log_file='app.log', level='INFO'):
        # Only initialize once
        if hasattr(self, '_initialized'):
            return
            
        self.log_file = log_file
        self.level = level
        self.file_handle = open(log_file, 'a')
        self._initialized = True
        
    def log(self, message, level='INFO'):
        if self._should_log(level):
            timestamp = datetime.now().isoformat()
            with self._lock:  # Thread-safe writing
                self.file_handle.write(f"[{timestamp}] {level}: {message}\n")
                self.file_handle.flush()
    
    def _should_log(self, level):
        levels = ['DEBUG', 'INFO', 'WARNING', 'ERROR']
        return levels.index(level) >= levels.index(self.level)
    
    def close(self):
        if hasattr(self, 'file_handle'):
            self.file_handle.close()
    
    @classmethod
    def reset_instance(cls):
        """Reset singleton for testing"""
        with cls._lock:
            if cls._instance:
                cls._instance.close()
            cls._instance = None

# Usage - always same instance
# module1.py
logger1 = Logger('app.log')
logger1.log("Starting module 1")

# module2.py
logger2 = Logger()  # Same instance!
logger2.log("Starting module 2")

print(logger1 is logger2)  # True
```

**Benefits of singleton here:**
- ✅ Single file handle
- ✅ Consistent configuration
- ✅ Resource efficiency
- ✅ Centralized control

## What Changes During Refactoring

### 1. **Override `__new__` Method**
```python
# Add singleton logic
def __new__(cls, *args, **kwargs):
    if cls._instance is None:
        cls._instance = super().__new__(cls)
    return cls._instance
```

### 2. **Add Thread Safety**
```python
# Add lock for thread safety
_lock = threading.Lock()

def __new__(cls):
    if cls._instance is None:
        with cls._lock:
            if cls._instance is None:
                cls._instance = super().__new__(cls)
    return cls._instance
```

### 3. **Protect Initialization**
```python
def __init__(self, config=None):
    # Prevent re-initialization
    if hasattr(self, '_initialized'):
        return
    
    # Initialize only once
    self.config = config
    self._initialized = True
```

### 4. **Add Reset Method for Testing**
```python
@classmethod
def reset_instance(cls):
    """Reset singleton (mainly for testing)"""
    cls._instance = None
```

## Complex Example: Database Connection Pool

### Before: Instance-Based Pool
```python
class DatabasePool:
    """Instance-based connection pool"""
    
    def __init__(self, host, port, database, max_connections=10):
        self.host = host
        self.port = port
        self.database = database
        self.max_connections = max_connections
        self._connections = []
        self._available = queue.Queue(maxsize=max_connections)
        self._create_initial_connections()
    
    def _create_initial_connections(self):
        for _ in range(self.max_connections):
            conn = self._create_connection()
            self._connections.append(conn)
            self._available.put(conn)
    
    def _create_connection(self):
        return psycopg2.connect(
            host=self.host,
            port=self.port,
            database=self.database
        )
    
    def get_connection(self):
        return self._available.get()
    
    def return_connection(self, conn):
        self._available.put(conn)
    
    def close_all(self):
        for conn in self._connections:
            conn.close()

# Problem: Multiple pools to same database
pool1 = DatabasePool('localhost', 5432, 'mydb')
pool2 = DatabasePool('localhost', 5432, 'mydb')  # Duplicate!
```

### After: Singleton Pool
```python
import threading
import weakref

class DatabasePool:
    """Singleton connection pool with registry"""
    
    _instances = {}  # Registry for different databases
    _lock = threading.Lock()
    
    def __new__(cls, host, port, database, max_connections=10):
        # Create unique key for this database
        key = (host, port, database)
        
        with cls._lock:
            if key not in cls._instances:
                instance = super().__new__(cls)
                cls._instances[key] = instance
            return cls._instances[key]
    
    def __init__(self, host, port, database, max_connections=10):
        # Only initialize once per unique database
        if hasattr(self, '_initialized'):
            return
            
        self.host = host
        self.port = port
        self.database = database
        self.max_connections = max_connections
        self._connections = []
        self._available = queue.Queue(maxsize=max_connections)
        self._in_use = weakref.WeakSet()  # Track connections in use
        self._lock = threading.RLock()
        self._create_initial_connections()
        self._initialized = True
    
    def _create_initial_connections(self):
        for _ in range(self.max_connections):
            conn = self._create_connection()
            self._connections.append(conn)
            self._available.put(conn)
    
    def _create_connection(self):
        return psycopg2.connect(
            host=self.host,
            port=self.port,
            database=self.database
        )
    
    @contextmanager
    def get_connection(self):
        """Context manager for safe connection handling"""
        conn = self._available.get()
        self._in_use.add(conn)
        try:
            yield conn
            conn.commit()
        except Exception:
            conn.rollback()
            raise
        finally:
            self._in_use.discard(conn)
            self._available.put(conn)
    
    def close_all(self):
        """Close all connections"""
        with self._lock:
            # Wait for connections to be returned
            while self._in_use:
                time.sleep(0.1)
            
            # Close all connections
            for conn in self._connections:
                try:
                    conn.close()
                except:
                    pass
            
            self._connections.clear()
            
    @classmethod
    def reset_pool(cls, host, port, database):
        """Reset specific pool"""
        key = (host, port, database)
        with cls._lock:
            if key in cls._instances:
                cls._instances[key].close_all()
                del cls._instances[key]

# Usage - always same pool for same database
pool1 = DatabasePool('localhost', 5432, 'mydb')
pool2 = DatabasePool('localhost', 5432, 'mydb')  # Same instance!
print(pool1 is pool2)  # True

# Different database gets different instance
pool3 = DatabasePool('localhost', 5432, 'testdb')  # Different instance
print(pool1 is pool3)  # False

# Safe connection usage
with pool1.get_connection() as conn:
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM users")
    results = cursor.fetchall()
```

## Migration Strategy

### Step 1: Identify Shared Resource
```python
# Look for resources that should be shared
# - File handles
# - Database connections  
# - Network sockets
# - Thread pools
# - Caches
```

### Step 2: Add Singleton Gradually
```python
class Service:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized'):
            return
        # Existing initialization code
        self._initialized = True
    
    # Keep all existing instance methods unchanged
```

### Step 3: Add Factory Method Option
```python
class Service:
    _instance = None
    
    def __init__(self, config):
        # Still allow normal instantiation
        self.config = config
    
    @classmethod
    def get_singleton(cls, config=None):
        """Get singleton instance"""
        if cls._instance is None:
            cls._instance = cls(config or default_config)
        return cls._instance
    
    @classmethod
    def create_new(cls, config):
        """Create new instance (not singleton)"""
        return object.__new__(cls).__init__(config)
```

## Common Pitfalls to Avoid

### 1. **Initialization Race Conditions**
```python
# BAD - not thread-safe
class BadSingleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:  # Race condition!
            cls._instance = super().__new__(cls)
        return cls._instance

# GOOD - thread-safe
class GoodSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
```

### 2. **Re-initialization Issues**
```python
# BAD - reinitializes on every call
class BadSingleton:
    def __init__(self, value):
        self.value = value  # Overwrites on each call!

# GOOD - initializes only once
class GoodSingleton:
    def __init__(self, value):
        if hasattr(self, '_initialized'):
            return
        self.value = value
        self._initialized = True
```

### 3. **Testing Difficulties**
```python
# Make singleton testable
class TestableSingleton:
    _instance = None
    _test_mode = False
    
    def __new__(cls):
        if cls._test_mode:
            # Allow new instances in test mode
            return super().__new__(cls)
            
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    @classmethod
    def enable_test_mode(cls):
        cls._test_mode = True
        cls._instance = None
```

## Testing Considerations

### Testing Singletons
```python
import unittest

class TestSingleton(unittest.TestCase):
    def setUp(self):
        # Reset singleton before each test
        MySingleton.reset_instance()
    
    def test_singleton_behavior(self):
        instance1 = MySingleton()
        instance2 = MySingleton()
        self.assertIs(instance1, instance2)
    
    def test_with_different_config(self):
        # Test that config doesn't change after first init
        instance1 = MySingleton(config={'debug': True})
        instance2 = MySingleton(config={'debug': False})
        
        # Both should have same config (first one)
        self.assertEqual(instance1.config['debug'], True)
        self.assertEqual(instance2.config['debug'], True)
```

### Mock Singleton for Testing
```python
from unittest.mock import Mock

def test_with_mock_singleton():
    # Replace singleton with mock
    original = Service._instance
    try:
        mock_service = Mock()
        Service._instance = mock_service
        
        # Test code that uses Service()
        result = function_using_service()
        
        # Verify mock was called
        mock_service.process.assert_called_once()
        
    finally:
        # Restore original
        Service._instance = original
```

## Key Takeaways

> [!important] When Refactoring Instance to Singleton
> 1. **Add thread-safe `__new__`** with double-check locking
> 2. **Protect `__init__`** from re-initialization  
> 3. **Consider registry pattern** for multiple unique singletons
> 4. **Add reset method** for testing
> 5. **Document singleton behavior** clearly
> 6. **Handle cleanup** properly (close files, connections)
> 7. **Make testable** with reset/mock capabilities