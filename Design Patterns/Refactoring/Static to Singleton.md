# Refactoring: Static to Singleton

#refactoring #patterns #static #singleton

## Why Refactor Static to Singleton?

Converting static pattern to singleton makes sense when:
- You need lazy initialization of expensive resources
- You want controlled lifecycle management
- You need to manage stateful resources (connections, files)
- You want to add initialization parameters
- You need better testing capabilities than pure static
- You want to implement interfaces/protocols

> [!warning] Consider Carefully
> This refactoring adds complexity. Make sure you need singleton features before converting from static.

## Simple Example: Application Logger

### Before: Static Pattern
```python
import logging
import sys

class Logger:
    """Static logger with class-level configuration"""
    
    # Class-level state
    _logger = None
    _initialized = False
    
    @staticmethod
    def _ensure_initialized():
        """Initialize logger if needed"""
        if not Logger._initialized:
            Logger._logger = logging.getLogger('app')
            Logger._logger.setLevel(logging.INFO)
            
            handler = logging.StreamHandler(sys.stdout)
            formatter = logging.Formatter(
                '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
            )
            handler.setFormatter(formatter)
            Logger._logger.addHandler(handler)
            
            Logger._initialized = True
    
    @staticmethod
    def debug(message):
        Logger._ensure_initialized()
        Logger._logger.debug(message)
    
    @staticmethod
    def info(message):
        Logger._ensure_initialized()
        Logger._logger.info(message)
    
    @staticmethod
    def error(message, exc_info=False):
        Logger._ensure_initialized()
        Logger._logger.error(message, exc_info=exc_info)
    
    @staticmethod
    def set_level(level):
        Logger._ensure_initialized()
        Logger._logger.setLevel(level)

# Usage
Logger.info("Application started")
Logger.set_level(logging.DEBUG)
Logger.debug("Debug message")
```

**Problems with static approach:**
- 🚫 No control over initialization timing
- 🚫 Can't pass configuration at runtime
- 🚫 Difficult to mock for testing
- 🚫 No way to reset or reconfigure

### After: Singleton Pattern
```python
import logging
import sys
import threading
from typing import Optional

class Logger:
    """Singleton logger with configurable initialization"""
    
    _instance: Optional['Logger'] = None
    _lock = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self, 
                 name='app',
                 level=logging.INFO,
                 format_string=None,
                 handlers=None):
        """Initialize logger (only runs once)"""
        if hasattr(self, '_initialized'):
            return
            
        self.name = name
        self._logger = logging.getLogger(name)
        self._logger.setLevel(level)
        self._logger.handlers.clear()  # Clear any existing handlers
        
        # Default format
        if format_string is None:
            format_string = '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        
        # Add handlers
        if handlers is None:
            handler = logging.StreamHandler(sys.stdout)
            handler.setFormatter(logging.Formatter(format_string))
            self._logger.addHandler(handler)
        else:
            for handler in handlers:
                self._logger.addHandler(handler)
        
        self._initialized = True
    
    def debug(self, message, **kwargs):
        self._logger.debug(message, **kwargs)
    
    def info(self, message, **kwargs):
        self._logger.info(message, **kwargs)
    
    def error(self, message, exc_info=False, **kwargs):
        self._logger.error(message, exc_info=exc_info, **kwargs)
    
    def set_level(self, level):
        self._logger.setLevel(level)
    
    def add_handler(self, handler):
        """Add additional handler"""
        self._logger.addHandler(handler)
    
    @classmethod
    def configure(cls, **kwargs):
        """Configure logger before first use"""
        if cls._instance is not None:
            raise RuntimeError("Logger already initialized")
        return cls(**kwargs)
    
    @classmethod
    def reset(cls):
        """Reset logger (mainly for testing)"""
        with cls._lock:
            if cls._instance and hasattr(cls._instance, '_logger'):
                cls._instance._logger.handlers.clear()
            cls._instance = None

# Usage
# Configure before first use
Logger.configure(
    name='myapp',
    level=logging.DEBUG,
    format_string='[%(levelname)s] %(message)s'
)

# Use normally
logger = Logger()  # Gets configured instance
logger.info("Application started")
logger.debug("Debug information")

# Add file handler later
from logging.handlers import RotatingFileHandler
file_handler = RotatingFileHandler('app.log', maxBytes=10485760, backupCount=5)
logger.add_handler(file_handler)
```

**Benefits of singleton approach:**
- ✅ Configurable initialization
- ✅ Lazy loading of resources
- ✅ Better lifecycle management
- ✅ Can be reset for testing
- ✅ Supports runtime configuration

## Complex Example: Database Query Builder

### Before: Static Query Builder
```python
class QueryBuilder:
    """Static SQL query builder"""
    
    # Static configuration
    _dialect = 'postgresql'
    _quote_char = '"'
    
    @staticmethod
    def set_dialect(dialect):
        """Set SQL dialect globally"""
        QueryBuilder._dialect = dialect
        QueryBuilder._quote_char = '"' if dialect == 'postgresql' else '`'
    
    @staticmethod
    def quote_identifier(identifier):
        """Quote table/column names"""
        char = QueryBuilder._quote_char
        return f"{char}{identifier}{char}"
    
    @staticmethod
    def select(table, columns=None, where=None, order_by=None):
        """Build SELECT query"""
        if columns is None:
            columns = ['*']
        
        # Quote identifiers based on dialect
        table = QueryBuilder.quote_identifier(table)
        columns = [QueryBuilder.quote_identifier(col) if col != '*' else col 
                  for col in columns]
        
        query = f"SELECT {', '.join(columns)} FROM {table}"
        
        if where:
            conditions = []
            for key, value in where.items():
                col = QueryBuilder.quote_identifier(key)
                if value is None:
                    conditions.append(f"{col} IS NULL")
                else:
                    conditions.append(f"{col} = %s")
            query += f" WHERE {' AND '.join(conditions)}"
        
        if order_by:
            query += f" ORDER BY {QueryBuilder.quote_identifier(order_by)}"
        
        return query
    
    @staticmethod
    def insert(table, data):
        """Build INSERT query"""
        table = QueryBuilder.quote_identifier(table)
        columns = [QueryBuilder.quote_identifier(k) for k in data.keys()]
        placeholders = ['%s'] * len(data)
        
        return (
            f"INSERT INTO {table} ({', '.join(columns)}) "
            f"VALUES ({', '.join(placeholders)})"
        )

# Problem: Global dialect affects all queries
QueryBuilder.set_dialect('mysql')
query1 = QueryBuilder.select('users')  # Uses MySQL syntax

QueryBuilder.set_dialect('postgresql')  
query2 = QueryBuilder.select('products')  # Now PostgreSQL syntax!
```

### After: Singleton Query Builder with Instances
```python
import threading
from typing import Dict, Any, Optional, List
from abc import ABC, abstractmethod

class QueryDialect(ABC):
    """Abstract base for SQL dialects"""
    
    @abstractmethod
    def quote_identifier(self, identifier: str) -> str:
        pass
    
    @abstractmethod
    def placeholder(self, index: int) -> str:
        pass

class PostgreSQLDialect(QueryDialect):
    def quote_identifier(self, identifier: str) -> str:
        return f'"{identifier}"'
    
    def placeholder(self, index: int) -> str:
        return '%s'

class MySQLDialect(QueryDialect):
    def quote_identifier(self, identifier: str) -> str:
        return f'`{identifier}`'
    
    def placeholder(self, index: int) -> str:
        return '%s'

class SQLiteDialect(QueryDialect):
    def quote_identifier(self, identifier: str) -> str:
        return f'"{identifier}"'
    
    def placeholder(self, index: int) -> str:
        return '?'

class QueryBuilder:
    """Singleton query builder with dialect support"""
    
    _instances: Dict[str, 'QueryBuilder'] = {}
    _lock = threading.Lock()
    
    # Available dialects
    _dialects = {
        'postgresql': PostgreSQLDialect(),
        'mysql': MySQLDialect(),
        'sqlite': SQLiteDialect()
    }
    
    def __new__(cls, dialect='postgresql'):
        """Get or create instance for specific dialect"""
        with cls._lock:
            if dialect not in cls._instances:
                instance = super().__new__(cls)
                cls._instances[dialect] = instance
            return cls._instances[dialect]
    
    def __init__(self, dialect='postgresql'):
        """Initialize with specific dialect"""
        if hasattr(self, '_initialized'):
            return
            
        if dialect not in self._dialects:
            raise ValueError(f"Unknown dialect: {dialect}")
            
        self.dialect_name = dialect
        self.dialect = self._dialects[dialect]
        self._query_cache = {}
        self._initialized = True
    
    def quote_identifier(self, identifier: str) -> str:
        """Quote identifier using current dialect"""
        return self.dialect.quote_identifier(identifier)
    
    def select(self, 
               table: str,
               columns: Optional[List[str]] = None,
               where: Optional[Dict[str, Any]] = None,
               order_by: Optional[str] = None,
               limit: Optional[int] = None) -> tuple[str, List[Any]]:
        """Build SELECT query with parameters"""
        
        # Check cache
        cache_key = ('select', table, tuple(columns or []), 
                    tuple(where.items()) if where else None,
                    order_by, limit)
        
        if cache_key in self._query_cache:
            query_template = self._query_cache[cache_key]
        else:
            # Build query
            if columns is None:
                columns = ['*']
            
            table_quoted = self.quote_identifier(table)
            columns_quoted = [
                self.quote_identifier(col) if col != '*' else col 
                for col in columns
            ]
            
            query_parts = [f"SELECT {', '.join(columns_quoted)} FROM {table_quoted}"]
            
            if where:
                conditions = []
                for key in where:
                    col = self.quote_identifier(key)
                    if where[key] is None:
                        conditions.append(f"{col} IS NULL")
                    else:
                        conditions.append(f"{col} = {self.dialect.placeholder(0)}")
                query_parts.append(f"WHERE {' AND '.join(conditions)}")
            
            if order_by:
                query_parts.append(f"ORDER BY {self.quote_identifier(order_by)}")
            
            if limit:
                query_parts.append(f"LIMIT {self.dialect.placeholder(0)}")
            
            query_template = ' '.join(query_parts)
            self._query_cache[cache_key] = query_template
        
        # Build parameters
        params = []
        if where:
            params.extend(v for v in where.values() if v is not None)
        if limit:
            params.append(limit)
        
        return query_template, params
    
    def insert(self, table: str, data: Dict[str, Any]) -> tuple[str, List[Any]]:
        """Build INSERT query with parameters"""
        table_quoted = self.quote_identifier(table)
        columns = [self.quote_identifier(k) for k in data.keys()]
        placeholders = [self.dialect.placeholder(i) for i in range(len(data))]
        
        query = (
            f"INSERT INTO {table_quoted} ({', '.join(columns)}) "
            f"VALUES ({', '.join(placeholders)})"
        )
        
        return query, list(data.values())
    
    @classmethod
    def for_dialect(cls, dialect: str) -> 'QueryBuilder':
        """Get query builder for specific dialect"""
        return cls(dialect)
    
    @classmethod
    def register_dialect(cls, name: str, dialect: QueryDialect):
        """Register custom dialect"""
        cls._dialects[name] = dialect
    
    @classmethod
    def reset_all(cls):
        """Reset all instances (for testing)"""
        with cls._lock:
            cls._instances.clear()

# Usage - different builders for different databases

# PostgreSQL queries
pg_builder = QueryBuilder.for_dialect('postgresql')
query, params = pg_builder.select(
    'users',
    columns=['id', 'name'],
    where={'active': True},
    order_by='created_at'
)
print(query)  # SELECT "id", "name" FROM "users" WHERE "active" = %s ORDER BY "created_at"

# MySQL queries (independent)
mysql_builder = QueryBuilder.for_dialect('mysql')
query, params = mysql_builder.select('products', where={'category': 'electronics'})
print(query)  # SELECT * FROM `products` WHERE `category` = %s

# They maintain separate state
print(pg_builder.dialect_name)    # postgresql
print(mysql_builder.dialect_name)  # mysql
```

## Migration Strategy

### Step 1: Wrap Static in Singleton
```python
class Service:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    # Keep static methods but delegate to instance
    @staticmethod
    def static_method(param):
        # Get singleton and call instance method
        instance = Service()
        return instance.instance_method(param)
    
    def instance_method(self, param):
        # New instance-based implementation
        return process(param)
```

### Step 2: Gradual Migration
```python
# Phase 1: Add instance methods alongside static
class Service:
    @staticmethod
    def process_static(data):  # Keep for compatibility
        return data * 2
    
    def process(self, data):  # New instance method
        return data * 2

# Phase 2: Static methods delegate to singleton
class Service:
    _instance = None
    
    @staticmethod
    def process_static(data):
        """Deprecated: Use instance method"""
        return Service()._process(data)
    
    def _process(self, data):
        return data * 2

# Phase 3: Remove static methods
```

## Key Differences to Implement

### 1. **Add State Management**
```python
# Static has no state
@staticmethod
def process(data):
    return transform(data)

# Singleton can maintain state
def __init__(self):
    self.cache = {}
    self.stats = {'processed': 0}

def process(self, data):
    self.stats['processed'] += 1
    if data in self.cache:
        return self.cache[data]
    result = transform(data)
    self.cache[data] = result
    return result
```

### 2. **Enable Configuration**
```python
# Static - no configuration
@staticmethod
def format_date(date):
    return date.strftime('%Y-%m-%d')

# Singleton - configurable
def __init__(self, date_format='%Y-%m-%d'):
    self.date_format = date_format

def format_date(self, date):
    return date.strftime(self.date_format)
```

### 3. **Add Lifecycle Methods**
```python
class ResourceManager:
    _instance = None
    
    def __init__(self):
        self.resources = []
        self.initialize()
    
    def initialize(self):
        """Setup resources"""
        self.connection = create_connection()
        self.resources.append(self.connection)
    
    def shutdown(self):
        """Cleanup resources"""
        for resource in self.resources:
            resource.close()
        self.resources.clear()
    
    @classmethod
    def reset(cls):
        """Reset singleton"""
        if cls._instance:
            cls._instance.shutdown()
        cls._instance = None
```

## Testing Improvements

### Static Method Testing (Limited)
```python
def test_static_method():
    # Can only test with global state
    result = Service.static_process(data)
    assert result == expected
    # No way to inject dependencies
```

### Singleton Testing (Better)
```python
def test_singleton_method():
    # Reset for clean state
    Service.reset()
    
    # Configure for testing
    service = Service(test_mode=True)
    
    # Can mock dependencies
    service.dependency = Mock()
    
    result = service.process(data)
    assert result == expected
    service.dependency.method.assert_called_once()
```

## Key Takeaways

> [!important] When Refactoring Static to Singleton
> 1. **Add singleton mechanism** - `__new__`, `_instance`, thread safety
> 2. **Convert static methods** to instance methods  
> 3. **Add initialization logic** in `__init__`
> 4. **Enable configuration** through constructor
> 5. **Implement lifecycle methods** - reset, shutdown
> 6. **Maintain compatibility** with wrapper methods
> 7. **Consider registry pattern** for multiple configurations