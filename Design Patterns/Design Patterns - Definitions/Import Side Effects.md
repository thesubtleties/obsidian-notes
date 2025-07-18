# Import Side Effects

#programming #python #javascript #modules #antipattern

## What are Import Side Effects?

**Import Side Effects** occur when importing a module executes code that changes the program state, performs I/O operations, or has other observable effects beyond just defining functions, classes, and variables. The module "does things" when imported rather than just providing definitions.

## Computer Science Definition

An import side effect is any observable change to the system state that occurs during the module loading/importing process. This violates the principle of referential transparency and makes module loading non-idempotent (importing twice may have different effects than importing once).

## Examples of Import Side Effects

### Python Import Side Effects
```python
# bad_module.py - Full of side effects!

print("Loading module...")  # Side effect: console output

# Side effect: Modifying global state
import sys
sys.path.append('/custom/path')

# Side effect: Network request on import
import requests
API_DATA = requests.get('https://api.example.com/config').json()

# Side effect: Database connection
import psycopg2
DB_CONN = psycopg2.connect("dbname=test user=postgres")  # Connected on import!

# Side effect: File I/O
with open('config.json') as f:
    CONFIG = json.load(f)

# Side effect: Starting threads
import threading
background_thread = threading.Thread(target=some_function)
background_thread.start()  # Thread starts on import!

# Side effect: Registering handlers
import atexit
atexit.register(cleanup_function)  # Registered on import

# Side effect: Class registration
class MyClass:
    pass

REGISTRY.append(MyClass)  # Modifies global registry
```

### JavaScript/TypeScript Import Side Effects
```typescript
// bad-module.ts - Side effects everywhere!

// Side effect: Console output
console.log('Module loading...');

// Side effect: DOM manipulation
if (typeof window !== 'undefined') {
    document.body.style.backgroundColor = 'red';
}

// Side effect: Global modification
(window as any).myGlobal = { loaded: true };

// Side effect: Network request
fetch('/api/config')
    .then(res => res.json())
    .then(config => {
        globalConfig = config;  // Setting global state
    });

// Side effect: Event listener registration
window.addEventListener('resize', handleResize);

// Side effect: Timer started
setInterval(() => {
    console.log('Heartbeat');
}, 1000);

// Side effect: Local storage access
const savedData = localStorage.getItem('userData');

export class MyClass {
    // Class definition is fine
}
```

## Why Import Side Effects are Problematic

### 1. **Order Dependencies**
```python
# The order of imports matters!
import module_a  # Sets up logging
import module_b  # Expects logging to be configured
# Swap these and the program breaks!
```

### 2. **Testing Difficulties**
```python
# How do you test this without making network calls?
# bad_weather.py
import requests
CURRENT_WEATHER = requests.get('http://api.weather.com/current').json()

def get_temperature():
    return CURRENT_WEATHER['temp']

# test_weather.py
# Just importing the module makes a network request!
import bad_weather  # Network call happens here
```

### 3. **Circular Import Issues**
```python
# module_a.py
import module_b
print(f"A: B's value is {module_b.VALUE}")  # Side effect
VALUE = 10

# module_b.py
import module_a  # Circular import!
print(f"B: A's value is {module_a.VALUE}")  # May fail!
VALUE = 20
```

### 4. **Performance Impact**
```python
# Slow import due to side effects
import heavy_module  # Takes 5 seconds due to data loading!
# Even if you only need one function
```

## Conceptual Understanding & Analogies

### The Library Analogy
Import side effects are like a library that:
- **Without side effects**: You get a library card, books stay on shelves until you check them out
- **With side effects**: Walking in automatically checks out random books, starts a book club, and orders pizza

### The Tool Shed Analogy
- **Without side effects**: Opening the shed shows you available tools
- **With side effects**: Opening the shed starts all power tools, opens paint cans, and begins construction

### The Recipe Book Analogy
- **Without side effects**: Opening cookbook shows recipes
- **With side effects**: Opening cookbook starts cooking, preheats oven, and orders ingredients

## How It Relates to Our Patterns

### Static Methods and Import Side Effects
```python
# BAD: Side effect on import
class DataProcessor:
    # This runs when module is imported!
    DEFAULT_CONFIG = load_config_from_file('config.json')  # Side effect!
    
    @staticmethod
    def process(data):
        return transform(data, DataProcessor.DEFAULT_CONFIG)

# GOOD: Lazy loading
class DataProcessor:
    _config = None
    
    @classmethod
    def get_config(cls):
        if cls._config is None:
            cls._config = load_config_from_file('config.json')
        return cls._config
    
    @staticmethod
    def process(data):
        return transform(data, DataProcessor.get_config())
```

### Singleton Pattern Issues
```python
# BAD: Singleton created on import
class DatabaseConnection:
    pass

# This runs on import!
DB_CONNECTION = DatabaseConnection()  # Side effect!
DB_CONNECTION.connect()  # Another side effect!

# GOOD: Lazy singleton
class DatabaseConnection:
    _instance = None
    
    @classmethod
    def get_instance(cls):
        if cls._instance is None:
            cls._instance = cls()
            cls._instance.connect()
        return cls._instance
```

## Detecting Import Side Effects

> [!warning] Signs of Import Side Effects
> 1. **Print statements** at module level
> 2. **Network calls** outside functions
> 3. **File I/O** at module level
> 4. **Global variable modification**
> 5. **Thread/process creation**
> 6. **Event handler registration**
> 7. **Database connections**
> 8. **Heavy computations** at module level

## Fixing Import Side Effects

### Before: With Side Effects
```python
# config.py
import json
import os

# Side effects on import
print("Loading configuration...")
CONFIG_PATH = os.environ['CONFIG_PATH']  # May fail on import!

with open(CONFIG_PATH) as f:  # File I/O on import
    CONFIG = json.load(f)

# Validate on import
if 'database' not in CONFIG:
    raise ValueError("Invalid config")  # May crash on import!

def get_database_url():
    return CONFIG['database']['url']
```

### After: No Side Effects
```python
# config.py
import json
import os
from functools import lru_cache

# No side effects - just definitions
_config = None

@lru_cache(maxsize=1)
def load_config():
    """Load configuration lazily when needed"""
    config_path = os.environ.get('CONFIG_PATH', 'default_config.json')
    
    try:
        with open(config_path) as f:
            config = json.load(f)
    except FileNotFoundError:
        config = get_default_config()
    
    validate_config(config)
    return config

def get_database_url():
    """Get database URL from config"""
    config = load_config()  # Loaded on first call
    return config['database']['url']

def validate_config(config):
    """Validate configuration structure"""
    if 'database' not in config:
        raise ValueError("Invalid config: missing database section")

def get_default_config():
    """Provide default configuration"""
    return {
        'database': {
            'url': 'sqlite:///default.db'
        }
    }
```

## Best Practices

### 1. **Initialization Functions**
```python
# Instead of side effects on import
def initialize_app():
    """Call this explicitly to set up the application"""
    setup_logging()
    connect_database()
    load_configuration()
    
# User explicitly calls when ready
if __name__ == '__main__':
    initialize_app()
    run_app()
```

### 2. **Factory Functions**
```python
# Instead of creating objects on import
def create_database_connection(config):
    """Factory function to create connections"""
    conn = DatabaseConnection(config)
    conn.connect()
    return conn

# User creates when needed
db = create_database_connection(config)
```

### 3. **Lazy Loading**
```python
class LazyResource:
    def __init__(self):
        self._resource = None
    
    @property
    def resource(self):
        if self._resource is None:
            self._resource = self._load_resource()
        return self._resource
    
    def _load_resource(self):
        # Expensive operation happens on first access
        return load_heavy_resource()
```

## Module Design Principles

> [!tip] Good Module Design
> 1. **Define, don't execute** - Modules should define things
> 2. **Explicit initialization** - Require explicit setup calls
> 3. **Lazy loading** - Defer expensive operations
> 4. **Pure functions** - Side effects only when called
> 5. **Configuration injection** - Pass config, don't load it
> 6. **Idempotent imports** - Import twice = import once

## When Side Effects are Acceptable

> [!info] Acceptable Import Side Effects
> 1. **Defining constants** from literals (not I/O)
> 2. **Registering codecs/handlers** (if necessary)
> 3. **Python path manipulation** (sparingly)
> 4. **Debug logging** of module loading (development only)

## Key Takeaways

> [!important] Remember About Import Side Effects
> 1. **Imports should be fast** - No slow operations
> 2. **Imports should be safe** - No exceptions from imports
> 3. **Imports should be pure** - No state changes
> 4. **Order shouldn't matter** - Except for true dependencies
> 5. **Testing requires import** - Side effects break tests
> 6. **Use initialization functions** - Explicit > implicit
> 7. **Lazy load resources** - Defer until needed