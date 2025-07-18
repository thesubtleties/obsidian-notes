# Part 2: Implementation Deep Dive

#programming #python #typescript #patterns #implementation #design-patterns

[[Static vs Instance vs Singleton Patterns - Python & TypeScript Deep Dive|← Back to Main Guide]] | [[Static vs Instance vs Singleton - Part 1 - Memory and Execution Models|← Part 1: Memory & Execution]]

## Table of Contents

1. [[#Static Methods - Deep Dive]]
2. [[#Instance Methods - Deep Dive]]
3. [[#Singleton Pattern - Deep Dive]]
4. [[#Thread Safety|Thread Safety and Concurrency]]
5. [[#Framework Integration]]
6. [[#Error Handling and Edge Cases]]

## Static Methods - Deep Dive

### Python Static Method Internals

```python
# Understanding @staticmethod decorator
class DeepDive:
    # What @staticmethod actually does
    def regular_function(x, y):
        return x + y
    
    # Equivalent to:
    static_method = staticmethod(regular_function)

# Behind the scenes
import inspect

class Analyzer:
    @staticmethod
    def static_method(x):
        return x * 2
    
    def instance_method(self, x):
        return x * 2

# Inspect the methods
print(type(Analyzer.__dict__['static_method']))  # <class 'staticmethod'>
print(type(Analyzer.__dict__['instance_method']))  # <class 'function'>

# Descriptor protocol
print(hasattr(Analyzer.static_method, '__get__'))  # True - it's a descriptor!
```

**What to understand:** The `@staticmethod` decorator doesn't just mark a method as static - it wraps the function in a descriptor object. When you access `Analyzer.static_method`, Python calls the descriptor's `__get__` method, which returns the original function unchanged. Regular methods are also descriptors, but their `__get__` creates bound method objects. This shows why static methods have less overhead - they skip the binding step entirely.

#### Static Method Descriptor Implementation

```python
# How staticmethod works internally (simplified)
class MyStaticMethod:
    def __init__(self, func):
        self.func = func
    
    def __get__(self, instance, owner):
        # Returns the original function unchanged
        return self.func
    
    def __repr__(self):
        return f"<mystaticmethod {self.func.__name__}>"

# Usage example
class Example:
    def raw_func(x, y):
        return x + y
    
    # These are equivalent:
    method1 = staticmethod(raw_func)
    method2 = MyStaticMethod(raw_func)

# Both work the same way
print(Example.method1(2, 3))  # 5
print(Example.method2(2, 3))  # 5
```

**What to understand:** This demonstrates how `staticmethod` is just a descriptor class. The key insight is in the `__get__` method - it returns `self.func` unchanged, regardless of whether it's accessed via class or instance. This is the mechanism that prevents `self` from being passed to static methods. Understanding descriptors helps you create your own method types and decorators.

### TypeScript Static Method Compilation

```typescript
// TypeScript source with static methods
class MathOperations {
    private static PI_CACHE = Math.PI;
    
    static circleArea(radius: number): number {
        return MathOperations.PI_CACHE * radius * radius;
    }
    
    static rectangleArea(width: number, height: number): number {
        return width * height;
    }
}

// Compiled to ES6
class MathOperations {
    static circleArea(radius) {
        return MathOperations.PI_CACHE * radius * radius;
    }
    static rectangleArea(width, height) {
        return width * height;
    }
}
MathOperations.PI_CACHE = Math.PI;

// Compiled to ES5
var MathOperations = /** @class */ (function () {
    function MathOperations() {
    }
    MathOperations.circleArea = function (radius) {
        return MathOperations.PI_CACHE * radius * radius;
    };
    MathOperations.rectangleArea = function (width, height) {
        return width * height;
    };
    MathOperations.PI_CACHE = Math.PI;
    return MathOperations;
}());
```

**What to understand:** TypeScript static methods compile differently based on the target. In ES6, they're just properties on the class. In ES5, they're properties on the constructor function. The key point: static data like `PI_CACHE` is initialized once and shared across all calls. This shows why static methods are memory-efficient for utilities but can't access instance data - they exist on the class/constructor, not on prototypes or instances.

### Advanced Static Method Patterns

```python
# Python: Class method vs static method
class Advanced:
    _registry = {}
    
    @classmethod
    def class_factory(cls, name):
        # Has access to class (cls)
        instance = cls()
        cls._registry[name] = instance
        return instance
    
    @staticmethod
    def static_utility(data):
        # No access to class or instance
        return len(data) > 0

# Practical difference
class Child(Advanced):
    pass

# Class method uses actual class
obj1 = Advanced.class_factory("adv")  # Creates Advanced instance
obj2 = Child.class_factory("child")   # Creates Child instance

print(type(obj1))  # <class 'Advanced'>
print(type(obj2))  # <class 'Child'>
```

**What to understand:** `@classmethod` vs `@staticmethod` is crucial. Class methods receive the actual class as their first argument (`cls`), which enables polymorphic behavior - `Child.class_factory()` creates a `Child` instance, not an `Advanced` instance. Static methods can't do this because they don't know which class they're called on. Use class methods for alternative constructors and factory methods that should respect inheritance.

```typescript
// TypeScript: Generic static methods
class DataProcessor<T> {
    // Static methods can be generic
    static transform<U, V>(data: U, fn: (item: U) => V): V {
        return fn(data);
    }
    
    // Static factory with type constraints
    static create<T extends { id: number }>(data: T): DataProcessor<T> {
        return new DataProcessor<T>();
    }
}

// Usage with type safety
const result = DataProcessor.transform("hello", s => s.length); // number
const processor = DataProcessor.create({ id: 1, name: "test" }); // DataProcessor<{id: number, name: string}>
```

**What to understand:** TypeScript static methods can be generic independently of the class generics. This enables powerful patterns like static factory methods that infer types from their arguments. The static `create` method uses a constraint (`T extends { id: number }`) to ensure type safety. This pattern is common in libraries where static methods provide typed factory functions or utilities.

## Instance Methods - Deep Dive

### Python Instance Method Mechanics

```python
# Method binding internals
class MethodBinding:
    def __init__(self, value):
        self.value = value
    
    def get_value(self):
        return self.value

# Create instance
obj = MethodBinding(42)

# Method binding process
unbound = MethodBinding.get_value  # Function object
bound = obj.get_value  # Bound method object

print(type(unbound))  # <class 'function'>
print(type(bound))    # <class 'method'>

# Manual binding
import types
manually_bound = types.MethodType(unbound, obj)
print(manually_bound())  # 42

# Method objects store both function and instance
print(bound.__func__ is unbound)  # True
print(bound.__self__ is obj)      # True
```

**What to understand:** Python's bound methods are objects that store both the original function (`__func__`) and the instance (`__self__`). When you call a bound method, Python essentially does `__func__(__self__, *args)`. This binding happens every time you access an instance method (like `obj.get_value`), which is why storing method references can impact performance in tight loops. The binding is what enables the method to access instance data via `self`.

#### Context and Binding Behavior

```python
# Dynamic method binding
class Dynamic:
    def __init__(self, name):
        self.name = name
    
    def greet(self):
        return f"Hello, {self.name}"

# Methods can be reassigned
obj1 = Dynamic("Alice")
obj2 = Dynamic("Bob")

# Store method reference
greet_alice = obj1.greet
greet_bob = obj2.greet

# Change object state
obj1.name = "Charlie"

# Bound methods maintain their context
print(greet_alice())  # "Hello, Charlie" - follows object state
print(greet_bob())    # "Hello, Bob"

# Rebinding methods
obj1.greet = types.MethodType(lambda self: f"Goodbye, {self.name}", obj1)
print(obj1.greet())   # "Goodbye, Charlie"
print(greet_alice())  # Still "Hello, Charlie" - original binding preserved
```

**What to understand:** Bound methods maintain a reference to their instance, not a snapshot of its state. When `obj1.name` changes, `greet_alice()` reflects the new value because it's still bound to `obj1`. However, the stored bound method (`greet_alice`) is independent of later modifications to `obj1.greet`. This shows why bound methods can lead to unexpected behavior if you're not careful about when binding occurs and what state changes afterward.

### TypeScript Instance Method Implementation

```typescript
// Understanding 'this' binding
class ThisBinding {
    name: string;
    
    constructor(name: string) {
        this.name = name;
    }
    
    // Regular method - 'this' is dynamic
    greet() {
        return `Hello, ${this.name}`;
    }
    
    // Arrow function property - 'this' is lexically bound
    greetArrow = () => {
        return `Hello, ${this.name}`;
    }
    
    // Explicitly bound method
    greetBound = this.greet.bind(this);
}

// Binding differences
const obj = new ThisBinding("Alice");
const { greet, greetArrow, greetBound } = obj;

// Regular method loses context
try {
    console.log(greet());  // Error: Cannot read property 'name' of undefined
} catch (e) {
    console.log("Regular method failed without context");
}

// Arrow function maintains context
console.log(greetArrow());  // "Hello, Alice"

// Bound method maintains context
console.log(greetBound());  // "Hello, Alice"

// Memory implications
console.log(obj.greet === new ThisBinding("Bob").greet);  // true - shared prototype
console.log(obj.greetArrow === new ThisBinding("Bob").greetArrow);  // false - per instance
```

**What to understand:** JavaScript handles `this` differently than Python handles `self`. Regular methods lose their context when destructured because `this` is determined at call time. Arrow functions capture `this` lexically (where they're defined), making them reliable but memory-intensive (each instance gets its own function). The `.bind()` approach creates a new function with fixed `this`. Choose based on your needs: prototype methods for memory efficiency, arrow functions for reliability, or explicit binding for specific cases.

### Advanced Instance Patterns

```python
# Python: Property decorators and descriptors
class Advanced:
    def __init__(self):
        self._value = 0
        self._access_count = 0
    
    @property
    def value(self):
        self._access_count += 1
        return self._value
    
    @value.setter
    def value(self, val):
        print(f"Setting value to {val}")
        self._value = val
    
    @property
    def access_count(self):
        return self._access_count
    
    # Cached property
    @functools.lru_cache(maxsize=1)
    def expensive_computation(self):
        print("Computing...")
        return sum(range(1000000))

# Usage
obj = Advanced()
print(obj.value)  # Increments access count
obj.value = 42    # Triggers setter
print(f"Accessed {obj.access_count} times")

# Cached method calls
result1 = obj.expensive_computation()  # Prints "Computing..."
result2 = obj.expensive_computation()  # Returns cached result
```

**What to understand:** Python's `@property` decorator creates computed attributes that look like simple attributes but execute code. The setter allows validation or side effects. `@lru_cache` on methods provides automatic memoization - but be careful, it caches per instance and can cause memory leaks if instances are long-lived. These patterns show how instance methods can maintain sophisticated per-object behavior that static methods cannot achieve.

```typescript
// TypeScript: Decorators and method modification
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    
    descriptor.value = function(...args: any[]) {
        console.log(`Calling ${propertyKey} with args:`, args);
        const result = original.apply(this, args);
        console.log(`Result:`, result);
        return result;
    };
    
    return descriptor;
}

function memoize(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const original = descriptor.value;
    const cache = new Map();
    
    descriptor.value = function(...args: any[]) {
        const key = JSON.stringify(args);
        if (cache.has(key)) {
            return cache.get(key);
        }
        const result = original.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

class Calculator {
    private multiplier: number;
    
    constructor(multiplier: number) {
        this.multiplier = multiplier;
    }
    
    @log
    multiply(x: number): number {
        return x * this.multiplier;
    }
    
    @memoize
    fibonacci(n: number): number {
        if (n <= 1) return n;
        return this.fibonacci(n - 1) + this.fibonacci(n - 2);
    }
}
```

**What to understand:** TypeScript decorators modify methods at definition time. The `@log` decorator wraps the method to add logging without changing the method's code. The `@memoize` decorator adds caching, dramatically improving performance for recursive functions like Fibonacci. Note that decorators maintain the original `this` context using `apply(this, args)`. This pattern enables aspect-oriented programming - adding cross-cutting concerns (logging, caching, validation) without cluttering business logic.

## Singleton Pattern - Deep Dive

### Python Singleton Implementations

#### Method 1: Metaclass Approach

```python
class SingletonMeta(type):
    _instances = {}
    _lock = threading.Lock()
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            with cls._lock:
                # Double-checked locking
                if cls not in cls._instances:
                    instance = super().__call__(*args, **kwargs)
                    cls._instances[cls] = instance
        return cls._instances[cls]

class Database(metaclass=SingletonMeta):
    def __init__(self):
        print("Creating database connection...")
        self.connection = self._connect()
    
    def _connect(self):
        return {"host": "localhost", "port": 5432}

# Thread-safe singleton
db1 = Database()  # Creates instance
db2 = Database()  # Returns same instance
print(db1 is db2)  # True
```

**What to understand:** The metaclass approach is the most Pythonic singleton implementation. The metaclass intercepts class instantiation via `__call__`, implementing double-checked locking for [[Thread Safety|thread safety]]. The outer check avoids lock acquisition when the instance exists, while the inner check prevents race conditions. This pattern ensures exactly one instance even under concurrent access, but adds complexity - consider if you really need a singleton versus a simple module-level instance.

#### Method 2: Decorator Approach

```python
def singleton(cls):
    instances = {}
    lock = threading.Lock()
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            with lock:
                if cls not in instances:
                    instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class Configuration:
    def __init__(self):
        print("Loading configuration...")
        self.settings = self._load_settings()
    
    def _load_settings(self):
        return {"debug": True, "port": 8080}

# Usage
config1 = Configuration()  # Creates instance
config2 = Configuration()  # Returns same instance
```

**What to understand:** The decorator approach turns the entire class into a factory function. When you call `Configuration()`, you're actually calling `get_instance()`, not the original class constructor. This is simpler than metaclasses but has a gotcha: `isinstance(config1, Configuration)` returns `False` because `Configuration` is now a function, not a class. Use this pattern when you want singleton behavior without the complexity of metaclasses.

#### Method 3: Module-Level Singleton

```python
# config.py
class _Configuration:
    def __init__(self):
        self.settings = {}
        self.load()
    
    def load(self):
        print("Loading configuration...")
        self.settings = {"debug": True}
    
    def get(self, key, default=None):
        return self.settings.get(key, default)

# Module-level instance
_instance = _Configuration()

# Public API
def get_config():
    return _instance

def reload_config():
    _instance.load()

# Usage in other modules
# from config import get_config
# config = get_config()  # Always same instance
```

**What to understand:** Python modules are singletons by nature - they're imported once and cached. This "module singleton" pattern leverages that behavior. The underscore prefix (`_Configuration`, `_instance`) indicates these are private implementation details. Exposing only functions (`get_config`, `reload_config`) gives you control over access and enables operations like reloading. This is often the simplest and most Pythonic singleton approach - [[Module Caching|module caching]] gives you singleton behavior for free.

### TypeScript Singleton Implementations

#### Method 1: Classic Singleton

```typescript
class DatabaseConnection {
    private static instance: DatabaseConnection;
    private connection: any;
    private constructor() {
        console.log("Creating database connection...");
        this.connection = this.connect();
    }
    
    private connect() {
        return { host: "localhost", port: 5432 };
    }
    
    public static getInstance(): DatabaseConnection {
        if (!DatabaseConnection.instance) {
            DatabaseConnection.instance = new DatabaseConnection();
        }
        return DatabaseConnection.instance;
    }
    
    public query(sql: string): any {
        console.log(`Executing: ${sql}`);
        return [];
    }
}

// Usage
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();
console.log(db1 === db2);  // true
```

**What to understand:** The classic TypeScript singleton uses a private constructor to prevent direct instantiation and a static method to control access. The lazy initialization (checking `if (!DatabaseConnection.instance)`) delays creation until first use. However, this isn't thread-safe in environments with true parallelism (like Workers). In typical Node.js/browser environments, JavaScript's single-threaded nature makes this safe, but be aware of async initialization issues.

#### Method 2: Namespace Singleton

```typescript
namespace ConfigurationManager {
    interface Settings {
        debug: boolean;
        port: number;
        apiUrl: string;
    }
    
    let settings: Settings | null = null;
    let initialized = false;
    
    export function initialize(overrides?: Partial<Settings>): void {
        if (initialized) {
            console.warn("Configuration already initialized");
            return;
        }
        
        settings = {
            debug: false,
            port: 3000,
            apiUrl: "https://api.example.com",
            ...overrides
        };
        initialized = true;
    }
    
    export function get<K extends keyof Settings>(key: K): Settings[K] {
        if (!settings) {
            throw new Error("Configuration not initialized");
        }
        return settings[key];
    }
    
    export function getAll(): Readonly<Settings> {
        if (!settings) {
            throw new Error("Configuration not initialized");
        }
        return Object.freeze({ ...settings });
    }
}

// Usage
ConfigurationManager.initialize({ debug: true });
const port = ConfigurationManager.get("port");
```

**What to understand:** TypeScript namespaces provide a singleton-like pattern without classes. The namespace encapsulates private state (`settings`, `initialized`) and exposes only public functions. This pattern is useful for configuration or service objects that don't need inheritance. The initialization guard prevents re-initialization, and the frozen return value in `getAll()` prevents external mutation. Namespaces are particularly useful for organizing related functionality without the overhead of classes.

#### Method 3: Module Singleton with Class

```typescript
// logger.ts
class Logger {
    private logs: string[] = [];
    
    constructor(private readonly prefix: string) {
        console.log(`Logger initialized with prefix: ${prefix}`);
    }
    
    log(message: string): void {
        const timestamp = new Date().toISOString();
        const logEntry = `[${timestamp}] ${this.prefix}: ${message}`;
        this.logs.push(logEntry);
        console.log(logEntry);
    }
    
    getLogs(): readonly string[] {
        return [...this.logs];
    }
}

// Export singleton instance
export const logger = new Logger("APP");

// Additional named exports for testing
export function createLogger(prefix: string): Logger {
    return new Logger(prefix);
}

// Usage in other modules
// import { logger } from './logger';
// logger.log("Application started");
```

**What to understand:** ES6 modules provide natural singleton behavior - the module is evaluated once and cached. Exporting an instance (`export const logger = new Logger("APP")`) creates a singleton that's shared across all imports. The additional `createLogger` export enables testing by allowing creation of independent instances. This pattern is idiomatic in modern JavaScript/TypeScript and leverages the module system's built-in caching behavior.

### Advanced Singleton Patterns

```python
# Python: Lazy initialization with parameters
class ParameterizedSingleton:
    _instances = {}
    _lock = threading.Lock()
    
    def __new__(cls, *args, **kwargs):
        # Create key from parameters
        key = (cls, args, tuple(sorted(kwargs.items())))
        
        if key not in cls._instances:
            with cls._lock:
                if key not in cls._instances:
                    instance = super().__new__(cls)
                    cls._instances[key] = instance
        return cls._instances[key]
    
    def __init__(self, config_file):
        # Only initialize once
        if hasattr(self, '_initialized'):
            return
        
        self.config_file = config_file
        self.settings = self._load_config()
        self._initialized = True
    
    def _load_config(self):
        print(f"Loading config from {self.config_file}")
        return {"file": self.config_file}

# Different parameters = different instances
config1 = ParameterizedSingleton("app.conf")
config2 = ParameterizedSingleton("app.conf")  # Same instance
config3 = ParameterizedSingleton("test.conf")  # Different instance

print(config1 is config2)  # True
print(config1 is config3)  # False
```

**What to understand:** This parameterized singleton creates different instances for different parameters - it's really a multiton pattern. The key is creating a hashable tuple from the parameters. This is useful for connection pools or configuration objects where you want one instance per configuration. Be careful with mutable parameters - they can't be hashed. Also watch for memory leaks as the `_instances` dictionary grows with each unique parameter combination.

```typescript
// TypeScript: Registry pattern for multiple singletons
class ServiceRegistry {
    private static services = new Map<string, any>();
    
    static register<T>(name: string, factory: () => T): void {
        if (this.services.has(name)) {
            throw new Error(`Service ${name} already registered`);
        }
        
        // Lazy initialization wrapper
        let instance: T | null = null;
        const lazyFactory = () => {
            if (!instance) {
                instance = factory();
            }
            return instance;
        };
        
        this.services.set(name, lazyFactory);
    }
    
    static get<T>(name: string): T {
        const factory = this.services.get(name);
        if (!factory) {
            throw new Error(`Service ${name} not found`);
        }
        return factory();
    }
    
    static reset(name?: string): void {
        if (name) {
            this.services.delete(name);
        } else {
            this.services.clear();
        }
    }
}

// Usage
ServiceRegistry.register('database', () => new DatabaseConnection());
ServiceRegistry.register('cache', () => new CacheService());

const db = ServiceRegistry.get<DatabaseConnection>('database');
const cache = ServiceRegistry.get<CacheService>('cache');
```

**What to understand:** The service registry pattern provides controlled access to multiple singletons. Each service is lazily initialized on first access via the wrapper function. This pattern is common in dependency injection containers. The type parameter on `get<T>` provides TypeScript type safety. The `reset` method enables testing by clearing services. This pattern scales better than individual singleton classes when you have many services to manage.

## Thread Safety and Concurrency

### Python Thread Safety

```python
import threading
import time
from concurrent.futures import ThreadPoolExecutor

# Thread-unsafe singleton
class UnsafeSingleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            time.sleep(0.001)  # Simulate initialization time
            cls._instance = super().__new__(cls)
            cls._instance.value = 0
        return cls._instance

# Thread-safe singleton with lock
class SafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    time.sleep(0.001)
                    cls._instance = super().__new__(cls)
                    cls._instance.value = 0
        return cls._instance

# Test thread safety
def create_instance(singleton_class, results, index):
    instance = singleton_class()
    results[index] = id(instance)

# Test unsafe
results = [None] * 10
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(create_instance, UnsafeSingleton, results, i) 
               for i in range(10)]
    for future in futures:
        future.result()

unsafe_unique = len(set(results))
print(f"Unsafe singleton created {unsafe_unique} unique instances")

# Test safe
results = [None] * 10
with ThreadPoolExecutor(max_workers=10) as executor:
    futures = [executor.submit(create_instance, SafeSingleton, results, i) 
               for i in range(10)]
    for future in futures:
        future.result()

safe_unique = len(set(results))
print(f"Safe singleton created {safe_unique} unique instances")
```

**What to understand:** This demonstrates why [[Thread Safety|thread safety]] matters for singletons. The unsafe version can create multiple instances when threads simultaneously check `_instance is None`. The safe version uses double-checked locking: the outer check avoids unnecessary lock acquisition for performance, while the inner check with lock held prevents race conditions. In Python, the GIL (Global Interpreter Lock) provides some protection, but explicit locking is still needed for true thread safety in singleton initialization.

### TypeScript/Node.js Considerations

```typescript
// Node.js is single-threaded, but async operations can cause issues
class AsyncSingleton {
    private static instance: AsyncSingleton;
    private static initializationPromise: Promise<AsyncSingleton>;
    
    private constructor(private data: any) {}
    
    private static async initialize(): Promise<AsyncSingleton> {
        // Simulate async initialization
        await new Promise(resolve => setTimeout(resolve, 100));
        const data = await this.loadData();
        return new AsyncSingleton(data);
    }
    
    private static async loadData(): Promise<any> {
        // Simulate async data loading
        return { config: "loaded" };
    }
    
    public static async getInstance(): Promise<AsyncSingleton> {
        if (!this.instance) {
            // Ensure only one initialization happens
            if (!this.initializationPromise) {
                this.initializationPromise = this.initialize();
            }
            this.instance = await this.initializationPromise;
        }
        return this.instance;
    }
}

// Safe concurrent access
async function testConcurrency() {
    const promises = Array.from({ length: 10 }, () => 
        AsyncSingleton.getInstance()
    );
    
    const instances = await Promise.all(promises);
    const uniqueInstances = new Set(instances).size;
    console.log(`Created ${uniqueInstances} unique instances`);  // Always 1
}
```

**What to understand:** JavaScript's single-threaded event loop doesn't protect against initialization races with async operations. Multiple calls to `getInstance()` before initialization completes could trigger multiple initializations. The solution: store the initialization promise, not just the instance. All callers await the same promise, ensuring only one initialization occurs. This pattern is crucial for singletons that require async setup (database connections, config loading, etc.).

## Framework Integration

### Python Framework Integration

```python
# Flask Integration
from flask import Flask, g
import sqlite3

class DatabaseManager:
    def __init__(self, db_path):
        self.db_path = db_path
    
    def get_connection(self):
        return sqlite3.connect(self.db_path)

# Singleton instance
db_manager = DatabaseManager('app.db')

app = Flask(__name__)

@app.before_request
def before_request():
    # Per-request connection
    g.db = db_manager.get_connection()

@app.teardown_request
def teardown_request(exception):
    db = getattr(g, 'db', None)
    if db is not None:
        db.close()

# Django Integration
from django.conf import settings

class CacheService:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.initialize()
        return cls._instance
    
    def initialize(self):
        self.redis_client = self._create_client()
    
    def _create_client(self):
        # Use Django settings
        return redis.Redis(
            host=settings.REDIS_HOST,
            port=settings.REDIS_PORT
        )

# Usage in views
cache_service = CacheService()
```

**What to understand:** Framework integration often requires adapting patterns to framework conventions. Flask uses `g` for request-scoped data, so the singleton `db_manager` provides connections while Flask manages their lifecycle. Django's settings integration shows how singletons can use framework configuration. The key insight: singletons should provide resources but let frameworks manage request/response lifecycles. Never store request-specific data in singletons - that would break concurrent request handling.

### TypeScript Framework Integration

```typescript
// Express.js Integration
import express from 'express';

class DatabaseService {
    private static instance: DatabaseService;
    private pool: any;
    
    private constructor() {
        this.pool = this.createPool();
    }
    
    private createPool() {
        // Create connection pool
        return { query: (sql: string) => Promise.resolve([]) };
    }
    
    static getInstance(): DatabaseService {
        if (!this.instance) {
            this.instance = new DatabaseService();
        }
        return this.instance;
    }
    
    async query(sql: string): Promise<any> {
        return this.pool.query(sql);
    }
}

// Middleware for dependency injection
const app = express();

app.use((req, res, next) => {
    req.db = DatabaseService.getInstance();
    next();
});

// NestJS Integration
import { Injectable, Module } from '@nestjs/common';

@Injectable()
export class ConfigService {
    private readonly config: Map<string, any>;
    
    constructor() {
        this.config = new Map();
        this.loadConfiguration();
    }
    
    private loadConfiguration() {
        // Load from environment
        this.config.set('port', process.env.PORT || 3000);
    }
    
    get(key: string): any {
        return this.config.get(key);
    }
}

@Module({
    providers: [
        {
            provide: ConfigService,
            useFactory: () => {
                // Ensures singleton
                return new ConfigService();
            }
        }
    ],
    exports: [ConfigService]
})
export class ConfigModule {}
```

**What to understand:** Modern frameworks often have built-in dependency injection that makes manual singleton patterns unnecessary. Express middleware can inject services into requests. NestJS's `@Injectable` decorator and module system automatically manage singleton services - the `useFactory` ensures a single instance. The lesson: before implementing singleton patterns, check if your framework already provides singleton behavior through its DI container. Framework-managed singletons are usually better tested and integrated.

## Error Handling and Edge Cases

### Singleton Error Handling

```python
# Python: Robust singleton with error handling
class RobustSingleton:
    _instance = None
    _lock = threading.Lock()
    _initialization_error = None
    
    def __new__(cls):
        if cls._initialization_error:
            raise cls._initialization_error
        
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    try:
                        instance = super().__new__(cls)
                        instance._initialize()
                        cls._instance = instance
                    except Exception as e:
                        cls._initialization_error = e
                        raise
        
        return cls._instance
    
    def _initialize(self):
        # Initialization that might fail
        self.connection = self._connect_to_database()
    
    def _connect_to_database(self):
        # Simulate potential failure
        import random
        if random.random() < 0.1:
            raise ConnectionError("Failed to connect to database")
        return {"connected": True}
    
    @classmethod
    def reset(cls):
        """Reset singleton for retry after error"""
        with cls._lock:
            cls._instance = None
            cls._initialization_error = None
```

**What to understand:** Singleton initialization can fail (database unreachable, config missing, etc.). This robust pattern caches initialization errors to provide consistent behavior - subsequent instantiation attempts get the same error. The `reset` class method allows recovery after fixing the underlying issue. Without this pattern, a transient initialization failure could leave your singleton in an inconsistent state. Always consider: what happens if singleton initialization fails, and how can users recover?

```typescript
// TypeScript: Singleton with lifecycle management
class ManagedSingleton {
    private static instance?: ManagedSingleton;
    private static isInitializing = false;
    private resources: Array<{ dispose: () => void }> = [];
    
    private constructor() {}
    
    static async getInstance(): Promise<ManagedSingleton> {
        if (!this.instance) {
            if (this.isInitializing) {
                throw new Error("Singleton is already being initialized");
            }
            
            this.isInitializing = true;
            try {
                const instance = new ManagedSingleton();
                await instance.initialize();
                this.instance = instance;
            } catch (error) {
                this.isInitializing = false;
                throw new Error(`Failed to initialize singleton: ${error.message}`);
            }
            this.isInitializing = false;
        }
        
        return this.instance;
    }
    
    private async initialize(): Promise<void> {
        // Initialize resources
        const db = await this.connectDatabase();
        this.resources.push(db);
        
        const cache = await this.connectCache();
        this.resources.push(cache);
    }
    
    static async dispose(): Promise<void> {
        if (this.instance) {
            await this.instance.cleanup();
            this.instance = undefined;
        }
    }
    
    private async cleanup(): Promise<void> {
        // Cleanup in reverse order
        for (const resource of this.resources.reverse()) {
            try {
                await resource.dispose();
            } catch (error) {
                console.error("Error disposing resource:", error);
            }
        }
        this.resources = [];
    }
    
    private async connectDatabase() {
        return {
            dispose: async () => console.log("Closing database connection")
        };
    }
    
    private async connectCache() {
        return {
            dispose: async () => console.log("Closing cache connection")
        };
    }
}

// Graceful shutdown
process.on('SIGTERM', async () => {
    await ManagedSingleton.dispose();
    process.exit(0);
});
```

**What to understand:** Singletons often manage resources (database connections, file handles, timers) that need explicit cleanup. This pattern tracks resources and ensures they're disposed in reverse order of acquisition (important for dependencies). The `isInitializing` flag prevents concurrent initialization attempts. The process event handler ensures cleanup on shutdown. Key principle: any singleton that acquires resources must provide a way to release them, especially important for graceful shutdown in production systems.

## Key Implementation Takeaways

> [!important] Implementation Best Practices
> 1. **Static Methods**: Use descriptor protocol in Python, understand prototype in JS
> 2. **Instance Methods**: Be aware of binding behavior and memory implications
> 3. **Singletons**: Always consider thread/async safety and initialization errors
> 4. **Error Handling**: Plan for initialization failures and cleanup
> 5. **Framework Integration**: Leverage framework-specific patterns when available
> 6. **Lifecycle Management**: Implement proper cleanup for singletons
> 7. **Testing**: Design with testability in mind from the start

[[Static vs Instance vs Singleton - Part 3 - Dependency Injection and Testing|Next: Part 3 - Dependency Injection & Testing →]]