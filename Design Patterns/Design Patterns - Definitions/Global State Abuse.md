# Global State Abuse

#programming #antipattern #architecture #state-management

## What is Global State?

**Global State** refers to data that is accessible from anywhere in your program. It exists outside of any specific function or class instance and persists throughout the program's execution.

## Computer Science Definition

Global state is data stored in the global scope that can be read or modified by any part of the program without explicit parameter passing. It violates the principle of encapsulation and creates implicit dependencies between components.

## Examples of Global State

```python
# Python global state examples
DATABASE_CONFIG = {"host": "localhost", "port": 5432}  # Global variable
connected_users = []  # Global list

class AppConfig:
    DEBUG_MODE = True  # Class variable (global to all instances)
    _instance = None   # Singleton instance (global)

# Modifying global state
def add_user(username):
    connected_users.append(username)  # Modifies global list
    
def toggle_debug():
    AppConfig.DEBUG_MODE = not AppConfig.DEBUG_MODE  # Modifies class variable
```

```typescript
// TypeScript global state examples
let globalCounter = 0;  // Global variable
const cache: Map<string, any> = new Map();  // Global cache

class Settings {
    static apiUrl = "https://api.example.com";  // Static property (global)
    private static instance: Settings;  // Singleton (global)
}

// Window/global object pollution
(window as any).myApp = {
    user: null,
    settings: {}
};
```

## Why Global State is Problematic

### 1. **Hidden Dependencies**
Code appears independent but secretly depends on global state:
```python
# Bad: Hidden dependency on global state
def calculate_price(amount):
    # Looks pure but depends on global TAX_RATE
    return amount * (1 + TAX_RATE)  # Where does TAX_RATE come from?

# Good: Explicit dependency
def calculate_price(amount, tax_rate):
    return amount * (1 + tax_rate)  # Clear where tax_rate comes from
```

### 2. **Testing Difficulties**
```python
# Hard to test - depends on global state
class UserService:
    def get_current_user(self):
        return global_user_session  # How do we mock this?

# Easy to test - dependency injection
class UserService:
    def __init__(self, session):
        self.session = session
    
    def get_current_user(self):
        return self.session.current_user  # Easy to inject mock
```

### 3. **Race Conditions & Thread Safety**
```python
# Dangerous with multiple threads
global_counter = 0

def increment():
    global global_counter
    temp = global_counter
    # Another thread might change global_counter here!
    global_counter = temp + 1
```

### 4. **Coupling & Maintenance**
Changes to global state affect entire program unpredictably.

## Conceptual Understanding & Analogies

### The Shared Whiteboard Analogy
Imagine an office with one whiteboard that everyone uses:
- Anyone can write on it anytime
- You never know when someone will erase your work
- Multiple people writing simultaneously creates chaos
- You can't have private information
- Testing changes means affecting everyone

### The Town Square Bulletin Board
Global state is like a town bulletin board:
- Everyone can see it (read access)
- Anyone can modify it (write access)
- No ownership or responsibility
- Changes affect the entire community
- Hard to track who changed what

## How It Relates to Our Patterns

### Static Methods & Global State
Static methods often access class-level (global) state:
```python
class Counter:
    count = 0  # Class variable is global state
    
    @staticmethod
    def increment():
        Counter.count += 1  # Modifying global state
```

### Singletons ARE Global State
```python
class DatabaseConnection:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance  # Global instance

# Anywhere in code:
db = DatabaseConnection()  # Same global instance
```

### Instance Methods Avoid Global State
```python
class Counter:
    def __init__(self):
        self.count = 0  # Instance state, not global
    
    def increment(self):
        self.count += 1  # Modifying local state
```

## Signs of Global State Abuse

> [!warning] Red Flags
> 1. Functions that work differently based on "configuration"
> 2. Tests that must run in specific order
> 3. Tests that need extensive setup/teardown
> 4. "Action at a distance" - changing A breaks unrelated B
> 5. Difficulty running tests in parallel
> 6. `global` keyword in Python, `window` manipulation in JS
> 7. Static variables that change during runtime

## Better Alternatives

### 1. **Dependency Injection**
```python
# Instead of global config
def process_data(data, config):  # Pass config explicitly
    return data * config['multiplier']
```

### 2. **Context Objects**
```python
class AppContext:
    def __init__(self, db, cache, config):
        self.db = db
        self.cache = cache
        self.config = config

# Pass context around instead of using globals
def handle_request(request, context):
    user = context.db.get_user(request.user_id)
    return process_user(user, context)
```

### 3. **Immutable Configuration**
```python
from typing import NamedTuple

class Config(NamedTuple):
    debug: bool
    timeout: int
    api_url: str

# Create once, never modify
CONFIG = Config(debug=True, timeout=30, api_url="...")
```

## When Global State is Acceptable

> [!info] Acceptable Uses
> 1. **True constants** that never change: `PI = 3.14159`
> 2. **Immutable configuration** loaded once at startup
> 3. **Logging** (though even this can be injected)
> 4. **Module-level caches** for expensive computations (with care)

## Key Takeaways

> [!important] Remember About Global State
> 1. **Makes code unpredictable** - behavior depends on hidden state
> 2. **Kills testability** - tests become order-dependent
> 3. **Creates coupling** - everything depends on everything
> 4. **Causes concurrency issues** - race conditions and data races
> 5. **Prefer explicit dependencies** - pass what you need
> 6. **Use sparingly** - constants and immutable config only
> 7. **Singletons are global state** - use with extreme caution