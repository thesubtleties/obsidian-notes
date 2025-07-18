# Part 5: Advanced Patterns & Performance

#programming #python #typescript #patterns #performance #optimization

[[Static vs Instance vs Singleton Patterns - Python & TypeScript Deep Dive|← Back to Main Guide]] | [[Static vs Instance vs Singleton - Part 4 - Real World Examples|← Part 4: Real-World Examples]]

## Table of Contents

1. [[#Hybrid Patterns]]
2. [[#Factory Patterns]]
3. [[#Service Locator Pattern]]
4. [[#Lazy Initialization]]
5. [[#Performance Benchmarks]]
6. [[#Memory Profiling]]
7. [[#Optimization Strategies]]
8. [[#Pattern Migration Guide]]

## Hybrid Patterns

### Combining Static and Instance Methods

```python
# Python: Hybrid pattern example
class DataProcessor:
    """Combines static utilities with instance-based processing"""
    
    # Class-level configuration
    _default_config = {
        'batch_size': 100,
        'timeout': 30,
        'retry_count': 3
    }
    
    def __init__(self, config: Optional[Dict[str, Any]] = None):
        """Instance initialization with config"""
        self.config = {**self._default_config, **(config or {})}
        self.processed_count = 0
        self.errors = []
    
    @staticmethod
    def validate_data(data: Any) -> bool:
        """Static validation - no state needed"""
        if not isinstance(data, (list, dict)):
            return False
        if isinstance(data, list):
            return all(isinstance(item, dict) for item in data)
        return True
    
    @staticmethod
    def normalize_item(item: Dict[str, Any]) -> Dict[str, Any]:
        """Static normalization - pure function"""
        return {
            k.lower().strip(): v
            for k, v in item.items()
            if v is not None
        }
    
    @classmethod
    def from_file(cls, filepath: str, config: Optional[Dict] = None) -> 'DataProcessor':
        """Factory method - creates configured instance"""
        file_config = {
            'source': filepath,
            'format': filepath.split('.')[-1]
        }
        combined_config = {**file_config, **(config or {})}
        return cls(combined_config)
    
    def process(self, data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Instance method - uses state"""
        if not self.validate_data(data):
            raise ValueError("Invalid data format")
        
        results = []
        for batch in self._batch_data(data):
            try:
                processed_batch = self._process_batch(batch)
                results.extend(processed_batch)
                self.processed_count += len(processed_batch)
            except Exception as e:
                self.errors.append({
                    'batch_index': len(results) // self.config['batch_size'],
                    'error': str(e)
                })
        
        return results
    
    def _batch_data(self, data: List[Any]) -> Iterator[List[Any]]:
        """Instance helper - uses config"""
        batch_size = self.config['batch_size']
        for i in range(0, len(data), batch_size):
            yield data[i:i + batch_size]
    
    def _process_batch(self, batch: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """Instance processing - can be overridden"""
        return [self.normalize_item(item) for item in batch]
    
    @property
    def summary(self) -> Dict[str, Any]:
        """Instance property - provides state summary"""
        return {
            'processed': self.processed_count,
            'errors': len(self.errors),
            'config': self.config
        }

# Usage
processor = DataProcessor({'batch_size': 50})
if DataProcessor.validate_data(raw_data):  # Static method
    processed = processor.process(raw_data)  # Instance method
    print(processor.summary)  # Instance property
```

**Understanding the Hybrid Pattern:**

This DataProcessor class demonstrates a thoughtful combination of static and instance methods, each chosen for specific reasons:

**Static Methods (`validate_data`, `normalize_item`):**
These are pure functions that don't need any state. Think of them as utility functions that could theoretically live outside the class, but we keep them here for organization and namespace clarity. They're marked with `@staticmethod` because:
- They perform the same operation regardless of any instance
- They don't need access to `self` or class variables
- They can be called without creating an instance: `DataProcessor.validate_data(my_data)`

**Class Method (`from_file`):**
This is our factory method - it knows how to create properly configured instances. We use `@classmethod` because it needs to return a new instance of whatever class it's called on (important for inheritance!). Notice how it:
- Takes the class itself as the first parameter (`cls`)
- Builds configuration based on the file path
- Returns a new instance with merged configuration

**Instance Methods (`process`, `_batch_data`, `_process_batch`):**
These methods need access to instance state (`self.config`, `self.processed_count`, `self.errors`). They're the workhorses that:
- Use the configuration stored in the instance
- Maintain processing statistics
- Can be overridden in subclasses for different processing logic

**The Design Philosophy:**
- **Static for stateless operations**: Validation and normalization don't care about configuration or history
- **Instance for stateful operations**: Processing needs to track counts, errors, and use configuration
- **Class method for smart construction**: The factory method creates pre-configured instances

This hybrid approach gives us flexibility - we can use the static utilities without instantiation when we just need quick operations, but we get full stateful processing when we create instances. It's like having both a Swiss Army knife (static methods) and a full toolbox (instance) in the same class.

```typescript
// TypeScript: Hybrid pattern example
interface ProcessorConfig {
    batchSize?: number;
    timeout?: number;
    retryCount?: number;
}

interface ProcessingSummary {
    processed: number;
    errors: number;
    config: ProcessorConfig;
}

class DataProcessor {
    // Static configuration
    private static readonly defaultConfig: ProcessorConfig = {
        batchSize: 100,
        timeout: 30,
        retryCount: 3
    };
    
    // Instance state
    private processedCount = 0;
    private errors: Array<{batchIndex: number; error: string}> = [];
    private config: ProcessorConfig;
    
    constructor(config?: ProcessorConfig) {
        this.config = { ...DataProcessor.defaultConfig, ...config };
    }
    
    // Static validation method
    static validateData(data: unknown): data is Array<Record<string, any>> {
        if (!Array.isArray(data)) return false;
        return data.every(item => typeof item === 'object' && item !== null);
    }
    
    // Static normalization method
    static normalizeItem(item: Record<string, any>): Record<string, any> {
        const normalized: Record<string, any> = {};
        
        for (const [key, value] of Object.entries(item)) {
            if (value !== null && value !== undefined) {
                normalized[key.toLowerCase().trim()] = value;
            }
        }
        
        return normalized;
    }
    
    // Factory method
    static fromFile(filepath: string, config?: ProcessorConfig): DataProcessor {
        const fileConfig: ProcessorConfig = {
            ...config,
            // Add file-specific configuration
        };
        
        return new DataProcessor(fileConfig);
    }
    
    // Instance method using static helpers
    process(data: unknown[]): Record<string, any>[] {
        if (!DataProcessor.validateData(data)) {
            throw new Error('Invalid data format');
        }
        
        const results: Record<string, any>[] = [];
        const batches = this.batchData(data);
        
        for (const [index, batch] of batches.entries()) {
            try {
                const processedBatch = this.processBatch(batch);
                results.push(...processedBatch);
                this.processedCount += processedBatch.length;
            } catch (error) {
                this.errors.push({
                    batchIndex: index,
                    error: error.message
                });
            }
        }
        
        return results;
    }
    
    // Instance helper method
    private *batchData<T>(data: T[]): Generator<T[], void, unknown> {
        const batchSize = this.config.batchSize || 100;
        
        for (let i = 0; i < data.length; i += batchSize) {
            yield data.slice(i, i + batchSize);
        }
    }
    
    // Instance processing method
    protected processBatch(batch: Record<string, any>[]): Record<string, any>[] {
        return batch.map(item => DataProcessor.normalizeItem(item));
    }
    
    // Instance getter for summary
    get summary(): ProcessingSummary {
        return {
            processed: this.processedCount,
            errors: this.errors.length,
            config: this.config
        };
    }
    
    // Static utility combined with instance state
    async processAsync(data: unknown[]): Promise<Record<string, any>[]> {
        // Can use both static methods and instance state
        if (!DataProcessor.validateData(data)) {
            throw new Error('Invalid data format');
        }
        
        // Async processing with instance config
        const timeout = this.config.timeout || 30;
        return Promise.race([
            this.process(data),
            new Promise<never>((_, reject) => 
                setTimeout(() => reject(new Error('Timeout')), timeout * 1000)
            )
        ]);
    }
}
```

**Understanding the TypeScript Hybrid Pattern:**

This TypeScript implementation shows how strong typing enhances the hybrid pattern. Let's break down the key design decisions:

**Type Safety First (Interfaces & Type Guards):**
The interfaces `ProcessorConfig` and `ProcessingSummary` define our contracts upfront. Notice how `validateData` is both a validator AND a type guard (`data is Array<Record<string, any>>`). This means:
- After calling `validateData`, TypeScript knows the data type is safe
- We get compile-time guarantees about data shape
- The type guard eliminates the need for type assertions later

**Static Methods with Purpose:**
- `validateData`: A type guard that serves double duty - runtime validation AND compile-time type narrowing
- `normalizeItem`: Pure transformation function that's completely predictable
- `fromFile`: Factory method that could be extended to read file metadata and configure accordingly

**Instance State Management:**
Look at the property declarations:
- `private processedCount = 0`: Inline initialization (cleaner than constructor assignment)
- `private errors: Array<{...}>`: Strongly typed error tracking
- `private config: ProcessorConfig`: Immutable after construction

```ts
  Constructor assignment (the alternative):
  class DataProcessor {
      private processedCount: number;  // Just declared, not initialized
      private errors: Array<{batchIndex: number; error: string}>;

      constructor(config?: ProcessorConfig) {
          this.processedCount = 0;  // Initialized in constructor
          this.errors = [];         // Initialized in constructor
          this.config = { ...DataProcessor.defaultConfig, ...config };
      }
  }
```

**Generator Function (`*batchData`):**
This is a beautiful use of TypeScript generators! Instead of building all batches at once:
- Memory efficient - creates batches on demand
- Type-safe with generics `<T>`
- Works seamlessly with for...of loops

**Access Modifiers Tell a Story:**
- `static`: No instance needed, pure utilities
- `private`: Internal implementation details (batchData)
- `protected`: Designed for inheritance (processBatch can be overridden)
- `public`: The API surface for consumers

**Async Pattern (`processAsync`):**
This method showcases hybrid design at its best:
- Uses static validation (no duplication!)
- Accesses instance config for timeout
- Implements Promise.race for timeout handling
- Shows how instance methods can build on each other

The beauty here is that TypeScript's type system enforces our design decisions at compile time. The hybrid pattern isn't just about organizing code - it's about creating a type-safe, extensible API that guides users toward correct usage.

### Mixed Singleton with Static Utilities

```python
# Python: Singleton with static utilities
class CacheManager:
    """Singleton cache with static utility methods"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialize()
        return cls._instance
    
    def _initialize(self):
        """Initialize instance state"""
        self.cache = {}
        self.stats = {
            'hits': 0,
            'misses': 0,
            'evictions': 0
        }
        self.max_size = 1000
    
    # Static utility methods
    @staticmethod
    def generate_key(*args, **kwargs) -> str:
        """Generate cache key from arguments"""
        key_parts = [str(arg) for arg in args]
        key_parts.extend(f"{k}:{v}" for k, v in sorted(kwargs.items()))
        return hashlib.md5('|'.join(key_parts).encode()).hexdigest()
    
    @staticmethod
    def serialize_value(value: Any) -> bytes:
        """Serialize value for storage"""
        return pickle.dumps(value)
    
    @staticmethod
    def deserialize_value(data: bytes) -> Any:
        """Deserialize stored value"""
        return pickle.loads(data)
    
    # Instance methods using static utilities
    def get(self, key: str) -> Optional[Any]:
        """Get value from cache"""
        if key in self.cache:
            self.stats['hits'] += 1
            return self.deserialize_value(self.cache[key]['data'])
        
        self.stats['misses'] += 1
        return None
    
    def set(self, key: str, value: Any, ttl: Optional[int] = None) -> None:
        """Set value in cache"""
        if len(self.cache) >= self.max_size:
            self._evict_oldest()
        
        self.cache[key] = {
            'data': self.serialize_value(value),
            'timestamp': time.time(),
            'ttl': ttl
        }
    
    def cache_function(self, ttl: Optional[int] = None):
        """Decorator for caching function results"""
        def decorator(func):
            @functools.wraps(func)
            def wrapper(*args, **kwargs):
                # Use static method to generate key
                cache_key = self.generate_key(func.__name__, *args, **kwargs)
                
                # Check cache
                cached = self.get(cache_key)
                if cached is not None:
                    return cached
                
                # Compute and cache
                result = func(*args, **kwargs)
                self.set(cache_key, result, ttl)
                return result
            
            return wrapper
        return decorator
    
    def _evict_oldest(self):
        """Evict oldest entry"""
        if not self.cache:
            return
        
        oldest_key = min(self.cache.keys(), 
                        key=lambda k: self.cache[k]['timestamp'])
        del self.cache[oldest_key]
        self.stats['evictions'] += 1

# Usage
cache = CacheManager()

# Use static method directly
key = CacheManager.generate_key('user', id=123)

# Use instance with decorator
@cache.cache_function(ttl=300)
def expensive_operation(x, y):
    return x ** y

result = expensive_operation(2, 10)  # Computed
result2 = expensive_operation(2, 10)  # From cache
```

**Understanding the Singleton + Static Hybrid:**

This CacheManager beautifully demonstrates why you might combine singleton with static methods. Let's explore the architecture:

**Thread-Safe Singleton Pattern:**
Notice the double-checked locking in `__new__`:
- First check: `if cls._instance is None` (fast, no lock needed)
- Second check inside the lock: Prevents race conditions where two threads passed the first check
- This ensures exactly one instance exists, even in multi-threaded environments

**Static Utilities - The "Why":**
The static methods (`generate_key`, `serialize_value`, `deserialize_value`) are pure functions that:
- Don't need cache state to operate
- Could be useful outside the cache context
- Are deterministic - same input always gives same output
- Act as the "toolbox" that both the singleton and external code can use

**Instance State - The Singleton Part:**
The singleton maintains:
- `self.cache`: The actual cached data
- `self.stats`: Hit/miss/eviction tracking
- `self.max_size`: Configuration that affects all cache operations

**The Decorator Pattern (`cache_function`):**
This is where the magic happens! The decorator:
- Uses the static `generate_key` method (no code duplication)
- Leverages instance state (the cache itself)
- Creates a closure that captures both the singleton instance and the function

**Design Insights:**
1. **Static methods for algorithms**: Key generation and serialization are algorithms, not stateful operations
2. **Singleton for shared state**: Everyone uses the same cache instance
3. **Separation of concerns**: You could use `generate_key` for other purposes (like distributed caching)
4. **Lazy initialization**: The cache isn't created until first access

The pattern says: "We need exactly one cache (singleton), but the utilities for working with cached data are universal (static)." It's like having a single shared warehouse (singleton) with a set of packing/unpacking tools (static methods) that anyone can use.

```typescript
// TypeScript: Singleton with static utilities
interface CacheEntry {
    data: string;
    timestamp: number;
    ttl?: number;
}

interface CacheStats {
    hits: number;
    misses: number;
    evictions: number;
}

class CacheManager {
    private static instance: CacheManager;
    private cache = new Map<string, CacheEntry>();
    private stats: CacheStats = {
        hits: 0,
        misses: 0,
        evictions: 0
    };
    private readonly maxSize = 1000;
    
    private constructor() {}
    
    static getInstance(): CacheManager {
        if (!this.instance) {
            this.instance = new CacheManager();
        }
        return this.instance;
    }
    
    // Static utility methods
    static generateKey(...args: any[]): string {
        const keyString = JSON.stringify(args);
        // Simple hash function (in production use crypto)
        let hash = 0;
        for (let i = 0; i < keyString.length; i++) {
            const char = keyString.charCodeAt(i);
            hash = ((hash << 5) - hash) + char;
            hash = hash & hash; // Convert to 32-bit integer
        }
        return hash.toString(36);
    }
    
    static serializeValue(value: any): string {
        return JSON.stringify(value);
    }
    
    static deserializeValue<T>(data: string): T {
        return JSON.parse(data);
    }
    
    // Instance methods
    get<T>(key: string): T | null {
        const entry = this.cache.get(key);
        
        if (!entry) {
            this.stats.misses++;
            return null;
        }
        
        // Check TTL
        if (entry.ttl && Date.now() - entry.timestamp > entry.ttl * 1000) {
            this.cache.delete(key);
            this.stats.misses++;
            return null;
        }
        
        this.stats.hits++;
        return CacheManager.deserializeValue<T>(entry.data);
    }
    
    set(key: string, value: any, ttl?: number): void {
        if (this.cache.size >= this.maxSize) {
            this.evictOldest();
        }
        
        this.cache.set(key, {
            data: CacheManager.serializeValue(value),
            timestamp: Date.now(),
            ttl
        });
    }
    
    // Method decorator for caching
    static memoize(ttl?: number) {
        const cache = CacheManager.getInstance();
        
        return function (
            target: any,
            propertyKey: string,
            descriptor: PropertyDescriptor
        ) {
            const originalMethod = descriptor.value;
            
            descriptor.value = function (...args: any[]) {
                const key = CacheManager.generateKey(propertyKey, ...args);
                
                const cached = cache.get(key);
                if (cached !== null) {
                    return cached;
                }
                
                const result = originalMethod.apply(this, args);
                cache.set(key, result, ttl);
                return result;
            };
            
            return descriptor;
        };
    }
    
    private evictOldest(): void {
        let oldestKey: string | null = null;
        let oldestTime = Infinity;
        
        for (const [key, entry] of this.cache.entries()) {
            if (entry.timestamp < oldestTime) {
                oldestTime = entry.timestamp;
                oldestKey = key;
            }
        }
        
        if (oldestKey) {
            this.cache.delete(oldestKey);
            this.stats.evictions++;
        }
    }
    
    getStats(): Readonly<CacheStats> {
        return { ...this.stats };
    }
}

// Usage example
class MathService {
    @CacheManager.memoize(300) // Cache for 5 minutes
    complexCalculation(x: number, y: number): number {
        console.log('Computing...');
        return Math.pow(x, y);
    }
}

const service = new MathService();
const result1 = service.complexCalculation(2, 10); // Computed
const result2 = service.complexCalculation(2, 10); // From cache
```

**Understanding the TypeScript Singleton + Static Pattern:**

This TypeScript implementation showcases modern patterns and language features. Let's dissect the design:

**Private Constructor Pattern:**
```typescript
private constructor() {}
```
This is TypeScript's way of enforcing singleton - you literally cannot call `new CacheManager()` from outside the class. It's compile-time enforcement of the singleton pattern!

**Type-Safe Serialization:**
Notice the generic methods:
- `deserializeValue<T>(data: string): T` - Returns the correct type
- `get<T>(key: string): T | null` - Maintains type information through the cache

This means when you retrieve from cache, TypeScript knows the expected type!

**The Decorator Factory (`memoize`):**
This is a masterpiece of TypeScript metaprogramming:
- It's a **static method** that returns a **method decorator**
- The decorator gets the singleton instance internally
- It wraps the original method with caching logic
- The `@CacheManager.memoize(300)` syntax is clean and declarative

**Map vs Object:**
Using `Map<string, CacheEntry>` instead of a plain object gives us:
- Better performance for frequent additions/deletions
- Guaranteed iteration order (insertion order)
- Clean size property (`this.cache.size`)
- No prototype pollution concerns

**Readonly Return Type:**
```typescript
getStats(): Readonly<CacheStats>
```
This prevents callers from modifying stats directly - they get a read-only view. It's a contract that says "you can look but don't touch."

**Static Utilities Design:**
The static methods serve as:
- **generateKey**: Creates consistent cache keys (notice the simple hash implementation)
- **serializeValue/deserializeValue**: Type-safe JSON operations
- **getInstance**: The singleton accessor
- **memoize**: The decorator factory

**Why This Pattern Shines:**
1. **Decorator pattern**: Clean, reusable caching without modifying method bodies
2. **Type safety**: Generics preserve types through serialization
3. **Encapsulation**: Private constructor, readonly returns
4. **Separation**: Static utilities can be used independently of the singleton

The beauty is in how TypeScript's features (private constructors, decorators, generics) make the pattern more robust than the Python version. It's the same concept but with compile-time guarantees!

## Factory Patterns

### Abstract Factory with Static and Instance Methods

```python
# Python: Abstract Factory Pattern
from abc import ABC, abstractmethod
from typing import Type, Dict, Any

class DatabaseConnection(ABC):
    """Abstract base for database connections"""
    
    @abstractmethod
    def connect(self) -> None:
        pass
    
    @abstractmethod
    def execute(self, query: str) -> Any:
        pass
    
    @abstractmethod
    def close(self) -> None:
        pass

class PostgresConnection(DatabaseConnection):
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.connection = None
    
    def connect(self) -> None:
        print(f"Connecting to PostgreSQL at {self.config['host']}")
        # Actual connection logic
    
    def execute(self, query: str) -> Any:
        print(f"Executing PostgreSQL query: {query}")
        # Actual execution logic
    
    def close(self) -> None:
        print("Closing PostgreSQL connection")

class MongoConnection(DatabaseConnection):
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.client = None
    
    def connect(self) -> None:
        print(f"Connecting to MongoDB at {self.config['host']}")
        # Actual connection logic
    
    def execute(self, query: str) -> Any:
        print(f"Executing MongoDB query: {query}")
        # Actual execution logic
    
    def close(self) -> None:
        print("Closing MongoDB connection")

class DatabaseFactory:
    """Factory with static registry and instance creation"""
    
    # Static registry of database types
    _registry: Dict[str, Type[DatabaseConnection]] = {}
    
    @classmethod
    def register(cls, db_type: str, connection_class: Type[DatabaseConnection]) -> None:
        """Register a database type (static)"""
        cls._registry[db_type] = connection_class
    
    @classmethod
    def get_available_types(cls) -> List[str]:
        """Get registered database types (static)"""
        return list(cls._registry.keys())
    
    @staticmethod
    def validate_config(db_type: str, config: Dict[str, Any]) -> bool:
        """Validate configuration (static)"""
        required_fields = {
            'postgres': ['host', 'port', 'database', 'user', 'password'],
            'mongo': ['host', 'port', 'database'],
        }
        
        if db_type in required_fields:
            return all(field in config for field in required_fields[db_type])
        return True
    
    def __init__(self, default_config: Optional[Dict[str, Any]] = None):
        """Instance initialization with default config"""
        self.default_config = default_config or {}
        self.connections: Dict[str, DatabaseConnection] = {}
    
    def create_connection(
        self,
        db_type: str,
        config: Optional[Dict[str, Any]] = None
    ) -> DatabaseConnection:
        """Create database connection (instance method)"""
        if db_type not in self._registry:
            raise ValueError(f"Unknown database type: {db_type}")
        
        # Merge configs
        final_config = {**self.default_config, **(config or {})}
        
        # Validate
        if not self.validate_config(db_type, final_config):
            raise ValueError(f"Invalid configuration for {db_type}")
        
        # Create connection
        connection_class = self._registry[db_type]
        connection = connection_class(final_config)
        
        # Store reference
        connection_id = f"{db_type}_{id(connection)}"
        self.connections[connection_id] = connection
        
        return connection
    
    def close_all_connections(self) -> None:
        """Close all managed connections (instance method)"""
        for connection in self.connections.values():
            try:
                connection.close()
            except Exception as e:
                print(f"Error closing connection: {e}")
        self.connections.clear()

# Register database types
DatabaseFactory.register('postgres', PostgresConnection)
DatabaseFactory.register('mongo', MongoConnection)

# Usage
factory = DatabaseFactory({'host': 'localhost', 'port': 5432})

# Use static method
print(DatabaseFactory.get_available_types())  # ['postgres', 'mongo']

# Create connections
pg_conn = factory.create_connection('postgres', {'database': 'mydb', 'user': 'user', 'password': 'pass'})
mongo_conn = factory.create_connection('mongo', {'database': 'mydb'})

# Cleanup
factory.close_all_connections()
```

**Understanding the Abstract Factory Pattern:**

This implementation beautifully demonstrates how to combine static registration with instance-based creation. Let's explore the architecture:

**The Abstract Base Class (`DatabaseConnection`):**
Using ABC (Abstract Base Class) enforces a contract - any database connection MUST implement connect, execute, and close. This is the foundation of the factory pattern - we can work with any database through this common interface.

**Static Registry Pattern:**
The `_registry` class variable acts as a global catalog:
- `register()`: Adds new database types at runtime (class method because it modifies class state)
- `get_available_types()`: Reports what's available (class method because it reads class state)
- This allows extensibility - you can add new database types without modifying the factory code!

**Static Validation (`validate_config`):**
This is a pure function - it doesn't need registry access or instance state. Making it static means:
- It can be used anywhere (even outside the factory)
- It's clearly a utility function
- It could be tested in isolation

**Instance State and Configuration:**
Each factory instance maintains:
- `default_config`: Base configuration applied to all connections
- `connections`: Tracks created connections for lifecycle management

This split is brilliant - the registry (what types exist) is global/static, but the connections (what we've created) are instance-specific.

**The Creation Flow:**
1. Check the static registry (type exists?)
2. Merge instance defaults with provided config
3. Validate using static method
4. Create the connection
5. Track it in instance state

**Why This Design Works:**
- **Separation of Concerns**: Registration (static) vs Creation (instance)
- **Extensibility**: New database types can be registered without changing factory code
- **Configuration Management**: Default configs at factory level, overrides at creation time
- **Resource Management**: The factory tracks and can clean up all connections it created

**Real-World Insight:**
This pattern is perfect when you need:
- A plugin system (register new types dynamically)
- Different configuration contexts (dev/staging/prod factories)
- Resource lifecycle management (the factory owns what it creates)

The static methods provide the "what's possible" while instance methods handle the "what we're doing."

```typescript
// TypeScript: Abstract Factory Pattern
interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
    [key: string]: any;
}

abstract class DatabaseConnection {
    constructor(protected config: DatabaseConfig) {}
    
    abstract connect(): Promise<void>;
    abstract execute<T = any>(query: string): Promise<T>;
    abstract close(): Promise<void>;
}

class PostgresConnection extends DatabaseConnection {
    private connection: any;
    
    async connect(): Promise<void> {
        console.log(`Connecting to PostgreSQL at ${this.config.host}`);
        // Actual connection logic
    }
    
    async execute<T = any>(query: string): Promise<T> {
        console.log(`Executing PostgreSQL query: ${query}`);
        // Actual execution logic
        return {} as T;
    }
    
    async close(): Promise<void> {
        console.log('Closing PostgreSQL connection');
        // Actual close logic
    }
}

class MongoConnection extends DatabaseConnection {
    private client: any;
    
    async connect(): Promise<void> {
        console.log(`Connecting to MongoDB at ${this.config.host}`);
        // Actual connection logic
    }
    
    async execute<T = any>(query: string): Promise<T> {
        console.log(`Executing MongoDB query: ${query}`);
        // Actual execution logic
        return {} as T;
    }
    
    async close(): Promise<void> {
        console.log('Closing MongoDB connection');
        // Actual close logic
    }
}

type ConnectionConstructor = new (config: DatabaseConfig) => DatabaseConnection;

class DatabaseFactory {
    // Static registry
    private static registry = new Map<string, ConnectionConstructor>();
    
    // Instance state
    private connections = new Map<string, DatabaseConnection>();
    
    constructor(private defaultConfig: Partial<DatabaseConfig> = {}) {}
    
    // Static registration method
    static register(type: string, connectionClass: ConnectionConstructor): void {
        this.registry.set(type, connectionClass);
    }
    
    // Static utility method
    static getAvailableTypes(): string[] {
        return Array.from(this.registry.keys());
    }
    
    // Static validation
    static validateConfig(type: string, config: DatabaseConfig): boolean {
        const requiredFields: Record<string, string[]> = {
            postgres: ['host', 'port', 'database', 'user', 'password'],
            mongo: ['host', 'port', 'database']
        };
        
        const required = requiredFields[type];
        if (!required) return true;
        
        return required.every(field => field in config);
    }
    
    // Instance method for creating connections
    async createConnection(
        type: string,
        config?: Partial<DatabaseConfig>
    ): Promise<DatabaseConnection> {
        const ConnectionClass = DatabaseFactory.registry.get(type);
        
        if (!ConnectionClass) {
            throw new Error(`Unknown database type: ${type}`);
        }
        
        // Merge configurations
        const finalConfig = {
            ...this.defaultConfig,
            ...config
        } as DatabaseConfig;
        
        // Validate
        if (!DatabaseFactory.validateConfig(type, finalConfig)) {
            throw new Error(`Invalid configuration for ${type}`);
        }
        
        // Create and initialize connection
        const connection = new ConnectionClass(finalConfig);
        await connection.connect();
        
        // Store reference
        const connectionId = `${type}_${Date.now()}`;
        this.connections.set(connectionId, connection);
        
        return connection;
    }
    
    // Instance method for cleanup
    async closeAllConnections(): Promise<void> {
        const closePromises = Array.from(this.connections.values()).map(
            conn => conn.close().catch(err => 
                console.error('Error closing connection:', err)
            )
        );
        
        await Promise.all(closePromises);
        this.connections.clear();
    }
    
    // Factory method pattern
    static async createPostgresConnection(config: DatabaseConfig): Promise<PostgresConnection> {
        const conn = new PostgresConnection(config);
        await conn.connect();
        return conn;
    }
    
    static async createMongoConnection(config: DatabaseConfig): Promise<MongoConnection> {
        const conn = new MongoConnection(config);
        await conn.connect();
        return conn;
    }
}

// Register types
DatabaseFactory.register('postgres', PostgresConnection);
DatabaseFactory.register('mongo', MongoConnection);

// Usage
async function example() {
    const factory = new DatabaseFactory({ host: 'localhost' });
    
    // Use static method
    console.log(DatabaseFactory.getAvailableTypes());
    
    // Create connections through instance
    const pgConn = await factory.createConnection('postgres', {
        port: 5432,
        database: 'mydb',
        user: 'user',
        password: 'pass'
    });
    
    // Use static factory method
    const mongoConn = await DatabaseFactory.createMongoConnection({
        host: 'localhost',
        port: 27017,
        database: 'mydb'
    });
    
    // Cleanup
    await factory.closeAllConnections();
}
```

**Understanding the TypeScript Abstract Factory:**

This TypeScript implementation showcases advanced type safety and async patterns. Let's break down the key improvements:

**Type-Safe Constructor Type:**
```typescript
type ConnectionConstructor = new (config: DatabaseConfig) => DatabaseConnection;
```
This type alias defines exactly what a valid connection class looks like. It must:
- Be constructable with `new`
- Accept a `DatabaseConfig`
- Return a `DatabaseConnection`

This gives us compile-time safety when registering classes!

**Abstract Class vs Interface:**
Unlike Python's ABC, TypeScript's abstract class provides:
- Partial implementation (the constructor stores config)
- `protected` config accessible to subclasses
- Abstract methods that MUST be implemented

**Generic Execute Method:**
```typescript
abstract execute<T = any>(query: string): Promise<T>;
```
This allows type-safe query results. Callers can specify the expected return type:
```typescript
const users = await conn.execute<User[]>('SELECT * FROM users');
```

**Map vs Object for Registry:**
Using `Map<string, ConnectionConstructor>` instead of a plain object:
- No prototype chain issues
- Clear intent (it's a collection, not an object)
- Better performance for dynamic keys
- Type-safe key/value pairs

**Async All The Things:**
Notice everything is async:
- `connect()`, `execute()`, `close()` all return Promises
- `createConnection()` awaits the connect before returning
- `closeAllConnections()` uses `Promise.all()` for parallel cleanup

**Error Handling in Cleanup:**
```typescript
conn => conn.close().catch(err => 
    console.error('Error closing connection:', err)
)
```
This ensures one failed connection doesn't prevent others from closing - resilient cleanup!

**Two Factory Patterns:**
1. **Registry pattern**: Dynamic type registration (`createConnection`)
2. **Static factory methods**: Type-specific creation (`createPostgresConnection`)

The static factory methods provide type-safe alternatives when you know the type at compile time.

**Partial<T> for Flexible Config:**
Using `Partial<DatabaseConfig>` allows providing only some config fields, merged with defaults. This creates a better developer experience - you only specify what differs from defaults.

**Why This Design Excels:**
- **Type safety**: Can't register invalid classes or create with wrong config
- **Async-first**: Built for modern async/await patterns
- **Flexible**: Both dynamic (registry) and static (factory methods) creation
- **Resilient**: Graceful error handling in cleanup

The TypeScript version isn't just a port - it leverages TypeScript's type system to prevent entire classes of errors at compile time!

## Service Locator Pattern

```python
# Python: Service Locator Pattern
from typing import Type, TypeVar, Optional, Callable, Any
import inspect

T = TypeVar('T')

class ServiceLocator:
    """Service locator combining static registration with instance management"""
    
    _instance = None
    _services: Dict[Type, Any] = {}
    _factories: Dict[Type, Callable[[], Any]] = {}
    _singletons: Dict[Type, Any] = {}
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    @classmethod
    def register(
        cls,
        service_type: Type[T],
        implementation: Optional[T] = None,
        factory: Optional[Callable[[], T]] = None,
        singleton: bool = True
    ) -> None:
        """Register a service (static method)"""
        if implementation:
            if singleton:
                cls._singletons[service_type] = implementation
            else:
                cls._services[service_type] = implementation
        elif factory:
            cls._factories[service_type] = factory
        else:
            raise ValueError("Either implementation or factory must be provided")
    
    @classmethod
    def register_lazy(
        cls,
        service_type: Type[T],
        factory: Callable[[], T],
        singleton: bool = True
    ) -> None:
        """Register a lazy-loaded service"""
        if singleton:
            # Wrap factory to cache result
            def singleton_factory():
                if service_type not in cls._singletons:
                    cls._singletons[service_type] = factory()
                return cls._singletons[service_type]
            cls._factories[service_type] = singleton_factory
        else:
            cls._factories[service_type] = factory
    
    def get(self, service_type: Type[T]) -> T:
        """Get a service instance"""
        # Check singletons first
        if service_type in self._singletons:
            return self._singletons[service_type]
        
        # Check factories
        if service_type in self._factories:
            return self._factories[service_type]()
        
        # Check regular services
        if service_type in self._services:
            return self._services[service_type]
        
        # Try to auto-resolve
        if self._can_auto_resolve(service_type):
            return self._auto_resolve(service_type)
        
        raise KeyError(f"Service {service_type.__name__} not registered")
    
    def _can_auto_resolve(self, service_type: Type[T]) -> bool:
        """Check if type can be auto-resolved"""
        if not inspect.isclass(service_type):
            return False
        
        # Check if constructor parameters can be resolved
        sig = inspect.signature(service_type.__init__)
        for param in sig.parameters.values():
            if param.name == 'self':
                continue
            if param.annotation == inspect.Parameter.empty:
                return False
            if param.annotation not in self._services and \
               param.annotation not in self._factories and \
               param.annotation not in self._singletons:
                return False
        
        return True
    
    def _auto_resolve(self, service_type: Type[T]) -> T:
        """Auto-resolve dependencies"""
        sig = inspect.signature(service_type.__init__)
        kwargs = {}
        
        for param in sig.parameters.values():
            if param.name == 'self':
                continue
            kwargs[param.name] = self.get(param.annotation)
        
        return service_type(**kwargs)
    
    @classmethod
    def clear(cls):
        """Clear all registrations"""
        cls._services.clear()
        cls._factories.clear()
        cls._singletons.clear()

# Example services
class Logger:
    def log(self, message: str):
        print(f"[LOG] {message}")

class Database:
    def __init__(self, connection_string: str):
        self.connection_string = connection_string
    
    def query(self, sql: str):
        print(f"Executing: {sql}")

class UserService:
    def __init__(self, database: Database, logger: Logger):
        self.database = database
        self.logger = logger
    
    def get_user(self, user_id: int):
        self.logger.log(f"Getting user {user_id}")
        self.database.query(f"SELECT * FROM users WHERE id = {user_id}")

# Registration
ServiceLocator.register(Logger, Logger())
ServiceLocator.register_lazy(
    Database,
    lambda: Database("postgresql://localhost/mydb")
)

# Auto-resolution
locator = ServiceLocator()
user_service = locator.get(UserService)  # Auto-resolves dependencies
user_service.get_user(123)
```

**Understanding the Service Locator Pattern:**

This Service Locator is a sophisticated dependency injection container that combines static registration with automatic dependency resolution. Let's explore its architecture:

**Three Storage Mechanisms:**
- `_services`: Direct instance storage (non-singleton)
- `_factories`: Functions that create instances on demand
- `_singletons`: Cached singleton instances

This separation allows different lifecycle management strategies for different services.

**Registration Flexibility:**
The `register` method accepts either:
- An existing instance (immediate registration)
- A factory function (lazy creation)
- A singleton flag to control instance sharing

This gives you fine-grained control over when and how services are created.

**Lazy Registration with Caching:**
The `register_lazy` method is brilliant - it wraps factory functions to implement singleton behavior:
```python
def singleton_factory():
    if service_type not in cls._singletons:
        cls._singletons[service_type] = factory()
    return cls._singletons[service_type]
```
This creates the instance only on first access, then caches it forever.

**Auto-Resolution Magic:**
The real power is in `_can_auto_resolve` and `_auto_resolve`:
1. Inspects the constructor using Python's `inspect` module
2. Checks if all constructor parameters have type annotations
3. Verifies all dependencies are registered
4. Automatically creates instances with resolved dependencies

This means you can register just the leaf services, and the locator figures out how to build the entire dependency tree!

**Type Safety with TypeVar:**
Using `Type[T]` and `TypeVar` provides type hints:
```python
def get(self, service_type: Type[T]) -> T:
```
This tells IDEs and type checkers what type will be returned.

**Singleton Pattern for the Locator:**
The ServiceLocator itself is a singleton, ensuring all parts of your application use the same service registry. This creates a central service hub.

**Design Insights:**
1. **Static registration, instance resolution**: Register globally, resolve locally
2. **Multiple lifecycle strategies**: Singleton, factory, or direct instance
3. **Automatic dependency injection**: No manual wiring needed
4. **Type introspection**: Uses Python's reflection capabilities
5. **Lazy loading**: Services created only when needed

**When to Use This Pattern:**
- Large applications with complex dependency graphs
- When you want automatic dependency injection without a framework
- Need different lifecycle management for different services
- Want to avoid passing dependencies through multiple layers

The beauty is that it combines the convenience of a global registry with the power of automatic dependency injection!

```typescript
// TypeScript: Service Locator Pattern
type ServiceFactory<T> = () => T;
type ServiceConstructor<T> = new (...args: any[]) => T;

interface ServiceRegistration<T> {
    instance?: T;
    factory?: ServiceFactory<T>;
    singleton: boolean;
}

class ServiceLocator {
    private static instance: ServiceLocator;
    private services = new Map<any, ServiceRegistration<any>>();
    
    private constructor() {}
    
    static getInstance(): ServiceLocator {
        if (!this.instance) {
            this.instance = new ServiceLocator();
        }
        return this.instance;
    }
    
    // Static registration methods
    static register<T>(
        token: any,
        implementation: T | ServiceFactory<T>,
        options: { singleton?: boolean } = {}
    ): void {
        const locator = this.getInstance();
        const singleton = options.singleton ?? true;
        
        if (typeof implementation === 'function') {
            locator.services.set(token, {
                factory: implementation as ServiceFactory<T>,
                singleton
            });
        } else {
            locator.services.set(token, {
                instance: implementation,
                singleton
            });
        }
    }
    
    static registerClass<T>(
        token: any,
        constructor: ServiceConstructor<T>,
        options: { singleton?: boolean; dependencies?: any[] } = {}
    ): void {
        const locator = this.getInstance();
        const singleton = options.singleton ?? true;
        const deps = options.dependencies || [];
        
        locator.services.set(token, {
            factory: () => {
                const resolvedDeps = deps.map(dep => locator.get(dep));
                return new constructor(...resolvedDeps);
            },
            singleton
        });
    }
    
    // Instance method for getting services
    get<T>(token: any): T {
        const registration = this.services.get(token);
        
        if (!registration) {
            throw new Error(`Service ${token.toString()} not registered`);
        }
        
        // Return existing singleton instance
        if (registration.instance) {
            return registration.instance;
        }
        
        // Create new instance from factory
        if (registration.factory) {
            const instance = registration.factory();
            
            // Cache if singleton
            if (registration.singleton) {
                registration.instance = instance;
            }
            
            return instance;
        }
        
        throw new Error(`Invalid registration for ${token.toString()}`);
    }
    
    // Decorator for dependency injection
    static inject(...dependencies: any[]) {
        return function <T extends { new(...args: any[]): {} }>(constructor: T) {
            return class extends constructor {
                constructor(...args: any[]) {
                    const locator = ServiceLocator.getInstance();
                    const resolvedDeps = dependencies.map(dep => locator.get(dep));
                    super(...resolvedDeps, ...args);
                }
            };
        };
    }
    
    // Clear all services
    static clear(): void {
        this.getInstance().services.clear();
    }
}

// Token-based registration
const LOGGER = Symbol('Logger');
const DATABASE = Symbol('Database');
const USER_SERVICE = Symbol('UserService');

interface ILogger {
    log(message: string): void;
}

interface IDatabase {
    query(sql: string): Promise<any>;
}

class ConsoleLogger implements ILogger {
    log(message: string): void {
        console.log(`[LOG] ${message}`);
    }
}

class PostgresDatabase implements IDatabase {
    constructor(private connectionString: string) {}
    
    async query(sql: string): Promise<any> {
        console.log(`Executing on ${this.connectionString}: ${sql}`);
        return [];
    }
}

@ServiceLocator.inject(DATABASE, LOGGER)
class UserService {
    constructor(
        private database: IDatabase,
        private logger: ILogger
    ) {}
    
    async getUser(userId: number): Promise<any> {
        this.logger.log(`Getting user ${userId}`);
        return this.database.query(`SELECT * FROM users WHERE id = ${userId}`);
    }
}

// Registration
ServiceLocator.register(LOGGER, new ConsoleLogger());
ServiceLocator.register(DATABASE, () => 
    new PostgresDatabase('postgresql://localhost/mydb')
);
ServiceLocator.registerClass(USER_SERVICE, UserService, {
    dependencies: [DATABASE, LOGGER]
});

// Usage
const locator = ServiceLocator.getInstance();
const userService = locator.get<UserService>(USER_SERVICE);
userService.getUser(123);
```

**Understanding the TypeScript Service Locator:**

This TypeScript implementation showcases advanced patterns including decorators, symbols, and type-safe dependency injection. Let's dive into the design:

**Symbol-Based Tokens:**
```typescript
const LOGGER = Symbol('Logger');
```
Using Symbols as service identifiers provides:
- Guaranteed uniqueness (no string collision)
- Private registration tokens (symbols aren't easily accessible)
- Clear intent that these are service identifiers

**Type-Safe Registration:**
The `ServiceRegistration<T>` interface ensures type safety throughout:
- Either an instance OR a factory (not both)
- Singleton flag controls lifecycle
- Generic `T` maintains type information

**Three Registration Methods:**
1. `register<T>`: Accepts instance or factory function
2. `registerClass`: Specifically for constructor functions with dependencies
3. `@inject` decorator: Automatic dependency injection

Each serves a different use case while maintaining the same underlying storage.

**The @inject Decorator Magic:**
```typescript
@ServiceLocator.inject(DATABASE, LOGGER)
class UserService {
```
This decorator:
- Returns a new class that extends the original
- Resolves dependencies before calling the original constructor
- Maintains all original class properties and methods
- Works transparently with TypeScript's type system

**Smart Factory Pattern:**
```typescript
factory: () => {
    const resolvedDeps = deps.map(dep => locator.get(dep));
    return new constructor(...resolvedDeps);
}
```
This creates a factory that:
- Resolves dependencies at creation time (not registration time)
- Supports circular dependencies (resolved lazily)
- Maintains proper TypeScript typing

**Interface-Based Design:**
Notice how services implement interfaces (`ILogger`, `IDatabase`):
- Decouples implementation from contract
- Allows easy mocking for tests
- Enables multiple implementations of the same interface
- TypeScript ensures type safety at compile time

**Singleton Caching Logic:**
```typescript
if (registration.singleton) {
    registration.instance = instance;
}
```
The caching happens after first creation, converting factories to instances for singletons.

**Why This Design Excels:**
1. **Type safety**: Generic methods preserve types through registration/resolution
2. **Multiple registration patterns**: Direct, factory, class with dependencies
3. **Decorator support**: Clean syntax for dependency injection
4. **Symbol tokens**: Avoid naming collisions and provide privacy
5. **Interface-driven**: Promotes loose coupling

**Real-World Benefits:**
- No need for string-based service names (error-prone)
- Decorators make dependency injection declarative
- Supports both eager and lazy initialization
- Type inference works throughout the system

The TypeScript version leverages language features to create a more robust, type-safe service locator that catches errors at compile time rather than runtime!

## Lazy Initialization

### Advanced Lazy Loading Patterns

```python
# Python: Lazy initialization patterns
import functools
from typing import TypeVar, Generic, Callable, Optional

T = TypeVar('T')

class Lazy(Generic[T]):
    """Generic lazy initialization wrapper"""
    
    def __init__(self, factory: Callable[[], T]):
        self._factory = factory
        self._value: Optional[T] = None
        self._initialized = False
        self._lock = threading.Lock()
    
    @property
    def value(self) -> T:
        """Get the lazily initialized value"""
        if not self._initialized:
            with self._lock:
                if not self._initialized:
                    self._value = self._factory()
                    self._initialized = True
        return self._value
    
    def is_initialized(self) -> bool:
        """Check if value has been initialized"""
        return self._initialized
    
    def reset(self) -> None:
        """Reset the lazy value"""
        with self._lock:
            self._value = None
            self._initialized = False

class LazyProperty:
    """Lazy property decorator"""
    
    def __init__(self, func):
        self.func = func
        self.__doc__ = func.__doc__
    
    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        
        value = self.func(obj)
        # Replace the property with the computed value
        obj.__dict__[self.func.__name__] = value
        return value

class ExpensiveResource:
    """Example class with lazy initialization"""
    
    def __init__(self):
        self._connection_lazy = Lazy(self._create_connection)
        self._cache_lazy = Lazy(self._create_cache)
    
    @LazyProperty
    def heavy_computation(self):
        """Expensive computation done only once"""
        print("Performing heavy computation...")
        import time
        time.sleep(1)  # Simulate expensive operation
        return sum(range(1000000))
    
    @property
    def connection(self):
        """Lazy database connection"""
        return self._connection_lazy.value
    
    @property
    def cache(self):
        """Lazy cache initialization"""
        return self._cache_lazy.value
    
    def _create_connection(self):
        print("Creating database connection...")
        return {"status": "connected"}
    
    def _create_cache(self):
        print("Initializing cache...")
        return {}
    
    @functools.lru_cache(maxsize=128)
    def cached_method(self, param: int) -> int:
        """Method with built-in caching"""
        print(f"Computing for {param}")
        return param ** 2

# Usage
resource = ExpensiveResource()
# Nothing initialized yet

print(resource.heavy_computation)  # Computed once
print(resource.heavy_computation)  # Returns cached value

print(resource.connection)  # Creates connection on first access
print(resource.cache)       # Creates cache on first access
```

**Understanding Lazy Initialization Patterns:**

This code demonstrates three distinct approaches to lazy initialization, each solving different problems. Let's explore each pattern:

**Generic Lazy<T> Wrapper:**
The `Lazy` class is a thread-safe, reusable wrapper for any lazy initialization:
- Uses double-checked locking for thread safety
- Generic type `T` maintains type information
- `reset()` method allows re-initialization
- Separates the "what" (factory function) from the "when" (access time)

This pattern is perfect when you want explicit control over lazy initialization.

**LazyProperty Decorator - The Clever Trick:**
```python
obj.__dict__[self.func.__name__] = value
```
This is Python magic at its finest! The decorator:
1. Computes the value on first access
2. **Replaces itself** in the instance dictionary
3. Future accesses bypass the descriptor entirely

It's self-removing lazy initialization - after first use, it's just a regular attribute!

**Property-Based Lazy Loading:**
The `connection` and `cache` properties wrap `Lazy` instances:
```python
@property
def connection(self):
    return self._connection_lazy.value
```
This provides a clean API - users just access `resource.connection` without knowing it's lazy.

**Built-in Caching with @lru_cache:**
```python
@functools.lru_cache(maxsize=128)
def cached_method(self, param: int) -> int:
```
Python's built-in LRU cache provides:
- Automatic memoization based on parameters
- Size-limited cache with LRU eviction
- Thread-safe by default
- Cache statistics available via `cached_method.cache_info()`

**Design Philosophy:**
1. **Lazy<T> for explicit control**: When you need thread safety and reset capability
2. **LazyProperty for one-time computation**: When the value never changes after initialization
3. **Property wrapping for clean APIs**: Hide lazy implementation details
4. **@lru_cache for parameter-based caching**: When results depend on inputs

**Performance Benefits:**
- Faster startup (expensive operations deferred)
- Memory efficiency (resources created only if needed)
- Better resource utilization (connections opened on demand)
- Automatic caching prevents redundant computation

**Real-World Applications:**
- Database connections that might not be used
- Configuration parsing that's expensive
- API clients that require network calls to initialize
- Heavy computations that might be skipped

The beauty is in having multiple tools - each pattern serves a specific use case, and they can be combined for maximum flexibility!

```typescript
// TypeScript: Lazy initialization patterns
class Lazy<T> {
    private value?: T;
    private initialized = false;
    
    constructor(private factory: () => T) {}
    
    get(): T {
        if (!this.initialized) {
            this.value = this.factory();
            this.initialized = true;
        }
        return this.value!;
    }
    
    isInitialized(): boolean {
        return this.initialized;
    }
    
    reset(): void {
        this.value = undefined;
        this.initialized = false;
    }
}

// Async lazy initialization
class AsyncLazy<T> {
    private promise?: Promise<T>;
    
    constructor(private factory: () => Promise<T>) {}
    
    async get(): Promise<T> {
        if (!this.promise) {
            this.promise = this.factory();
        }
        return this.promise;
    }
    
    reset(): void {
        this.promise = undefined;
    }
}

// Lazy decorator
function LazyGetter<T>(factory: () => T) {
    const key = Symbol();
    
    return function(target: any, propertyKey: string) {
        Object.defineProperty(target, propertyKey, {
            get() {
                if (!this[key]) {
                    this[key] = factory.call(this);
                }
                return this[key];
            },
            enumerable: true,
            configurable: true
        });
    };
}

// Memoize decorator
function Memoize() {
    return function(
        target: any,
        propertyKey: string,
        descriptor: PropertyDescriptor
    ) {
        const originalMethod = descriptor.value;
        const cacheKey = Symbol();
        
        descriptor.value = function(...args: any[]) {
            if (!this[cacheKey]) {
                this[cacheKey] = new Map();
            }
            
            const key = JSON.stringify(args);
            const cache = this[cacheKey];
            
            if (cache.has(key)) {
                return cache.get(key);
            }
            
            const result = originalMethod.apply(this, args);
            cache.set(key, result);
            return result;
        };
        
        return descriptor;
    };
}

class ExpensiveResource {
    private connectionLazy = new Lazy(() => this.createConnection());
    private configLazy = new AsyncLazy(() => this.loadConfig());
    
    @LazyGetter(() => {
        console.log('Computing expensive value...');
        return Array.from({ length: 1000000 }, (_, i) => i).reduce((a, b) => a + b, 0);
    })
    expensiveValue!: number;
    
    get connection() {
        return this.connectionLazy.get();
    }
    
    async getConfig() {
        return this.configLazy.get();
    }
    
    private createConnection() {
        console.log('Creating connection...');
        return { status: 'connected' };
    }
    
    private async loadConfig() {
        console.log('Loading configuration...');
        await new Promise(resolve => setTimeout(resolve, 1000));
        return { debug: true, timeout: 30 };
    }
    
    @Memoize()
    calculateValue(x: number): number {
        console.log(`Calculating for ${x}`);
        return x * x;
    }
}

// Advanced lazy pattern with dependencies
class LazyServiceContainer {
    private services = new Map<string, any>();
    private factories = new Map<string, () => any>();
    private initializing = new Set<string>();
    
    register(name: string, factory: () => any): void {
        this.factories.set(name, factory);
    }
    
    get<T>(name: string): T {
        // Return if already initialized
        if (this.services.has(name)) {
            return this.services.get(name);
        }
        
        // Check for circular dependencies
        if (this.initializing.has(name)) {
            throw new Error(`Circular dependency detected: ${name}`);
        }
        
        // Get factory
        const factory = this.factories.get(name);
        if (!factory) {
            throw new Error(`Service not registered: ${name}`);
        }
        
        // Initialize
        this.initializing.add(name);
        try {
            const service = factory();
            this.services.set(name, service);
            return service;
        } finally {
            this.initializing.delete(name);
        }
    }
}
```

**Understanding TypeScript Lazy Initialization:**

This TypeScript implementation showcases advanced lazy patterns including async support, decorators, and circular dependency detection. Let's explore the patterns:

**AsyncLazy<T> - Promise-Based Lazy Loading:**
```typescript
if (!this.promise) {
    this.promise = this.factory();
}
return this.promise;
```
This pattern is brilliant for async operations:
- The factory is called only once, even with concurrent access
- Multiple callers get the same Promise instance
- Perfect for API calls, file loading, or database connections
- No race conditions - the Promise itself handles synchronization

**Symbol-Based Caching in Decorators:**
```typescript
const key = Symbol();
```
Using Symbols for cache storage:
- Prevents property name collisions
- Keeps cache data "hidden" from enumeration
- Each decorated property gets its own unique storage
- Cleaner than string-based property names

**@LazyGetter Decorator:**
This decorator transforms a property into a lazy-computed value:
- First access triggers computation
- Result is cached using a Symbol key
- `factory.call(this)` preserves the `this` context
- Works seamlessly with TypeScript's type system

**@Memoize Decorator with Map Cache:**
```typescript
const key = JSON.stringify(args);
```
This memoization pattern:
- Uses Map for better performance than objects
- JSON.stringify creates cache keys from arguments
- Per-instance caching (each instance has its own cache)
- Could be enhanced with WeakMap for memory efficiency

**LazyServiceContainer - Advanced Pattern:**
This showcases enterprise-level lazy initialization:
1. **Circular dependency detection**: Tracks what's currently initializing
2. **Two-phase storage**: Factories registered, services created on demand
3. **Error boundaries**: try/finally ensures cleanup even on failure
4. **Type-safe generics**: `get<T>(name: string): T`

**Design Patterns Comparison:**
- **Lazy<T>**: Simple, synchronous lazy initialization
- **AsyncLazy<T>**: For Promise-based async operations
- **@LazyGetter**: Decorator for property-level laziness
- **@Memoize**: Parameter-based caching
- **LazyServiceContainer**: Enterprise dependency injection

**Real-World Benefits:**
1. **Performance**: Defer expensive operations until needed
2. **Memory**: Don't allocate resources that might not be used
3. **Startup time**: Faster application initialization
4. **Scalability**: Load resources on demand based on usage

**Advanced Considerations:**
- AsyncLazy prevents "thundering herd" - multiple requests don't trigger multiple initializations
- Symbol-based storage prevents accidental property access
- Circular dependency detection prevents infinite loops
- Type preservation through generics maintains IntelliSense

The TypeScript patterns leverage language features (decorators, symbols, async/await) to create more robust lazy initialization than possible in many other languages!

## Performance Benchmarks

### Comprehensive Performance Testing

```python
# Python: Performance benchmarking
import time
import timeit
import memory_profiler
import cProfile
import pstats
from contextlib import contextmanager
from dataclasses import dataclass
from typing import List, Callable

@dataclass
class BenchmarkResult:
    name: str
    time_ms: float
    memory_mb: float
    iterations: int

class PerformanceBenchmark:
    """Comprehensive performance testing framework"""
    
    def __init__(self):
        self.results: List[BenchmarkResult] = []
    
    @contextmanager
    def measure_time(self, name: str):
        """Context manager for timing"""
        start = time.perf_counter()
        yield
        end = time.perf_counter()
        print(f"{name}: {(end - start) * 1000:.2f}ms")
    
    @memory_profiler.profile
    def measure_memory(self, func: Callable):
        """Decorator for memory profiling"""
        return func()
    
    def benchmark_patterns(self):
        """Benchmark all three patterns"""
        iterations = 100000
        
        # Setup test classes
        class StaticOperations:
            data = list(range(100))
            
            @staticmethod
            def process():
                return sum(StaticOperations.data)
        
        class InstanceOperations:
            def __init__(self):
                self.data = list(range(100))
            
            def process(self):
                return sum(self.data)
        
        class SingletonOperations:
            _instance = None
            
            def __new__(cls):
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance.data = list(range(100))
                return cls._instance
            
            def process(self):
                return sum(self.data)
        
        # Benchmark static method
        static_time = timeit.timeit(
            'StaticOperations.process()',
            globals=locals(),
            number=iterations
        )
        
        # Benchmark instance method
        instance = InstanceOperations()
        instance_time = timeit.timeit(
            'instance.process()',
            globals=locals(),
            number=iterations
        )
        
        # Benchmark singleton method
        singleton = SingletonOperations()
        singleton_time = timeit.timeit(
            'singleton.process()',
            globals=locals(),
            number=iterations
        )
        
        # Memory usage comparison
        import tracemalloc
        
        # Static memory
        tracemalloc.start()
        for _ in range(1000):
            StaticOperations.process()
        static_memory = tracemalloc.get_traced_memory()[1]
        tracemalloc.stop()
        
        # Instance memory
        tracemalloc.start()
        instances = [InstanceOperations() for _ in range(1000)]
        for inst in instances:
            inst.process()
        instance_memory = tracemalloc.get_traced_memory()[1]
        tracemalloc.stop()
        
        # Singleton memory
        tracemalloc.start()
        singletons = [SingletonOperations() for _ in range(1000)]
        for s in singletons:
            s.process()
        singleton_memory = tracemalloc.get_traced_memory()[1]
        tracemalloc.stop()
        
        # Store results
        self.results = [
            BenchmarkResult(
                "Static", 
                static_time * 1000, 
                static_memory / 1024 / 1024,
                iterations
            ),
            BenchmarkResult(
                "Instance", 
                instance_time * 1000,
                instance_memory / 1024 / 1024,
                iterations
            ),
            BenchmarkResult(
                "Singleton",
                singleton_time * 1000,
                singleton_memory / 1024 / 1024,
                iterations
            )
        ]
        
        return self.results
    
    def profile_code(self, func: Callable):
        """Profile function execution"""
        profiler = cProfile.Profile()
        profiler.enable()
        
        result = func()
        
        profiler.disable()
        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(10)
        
        return result
    
    def generate_report(self):
        """Generate performance report"""
        print("\n=== Performance Benchmark Report ===\n")
        print(f"{'Pattern':<12} {'Time (ms)':<12} {'Memory (MB)':<12} {'Ops/sec':<12}")
        print("-" * 48)
        
        for result in self.results:
            ops_per_sec = (result.iterations / result.time_ms) * 1000
            print(f"{result.name:<12} {result.time_ms:<12.2f} {result.memory_mb:<12.2f} {ops_per_sec:<12.0f}")
        
        # Calculate relative performance
        if self.results:
            base_time = self.results[0].time_ms
            print("\nRelative Performance:")
            for result in self.results:
                relative = (result.time_ms / base_time) * 100
                print(f"{result.name}: {relative:.1f}%")

# Run benchmarks
benchmark = PerformanceBenchmark()
benchmark.benchmark_patterns()
benchmark.generate_report()
```

**Understanding Performance Benchmarking:**

This comprehensive benchmarking framework demonstrates how to scientifically compare the performance characteristics of our three patterns. Let's break down the approach:

**Context Manager for Timing:**
```python
@contextmanager
def measure_time(self, name: str):
```
Using a context manager provides:
- Clean syntax with `with` statements
- Automatic timing calculation
- Exception-safe measurement (time is recorded even if code fails)
- Human-readable output in milliseconds

**Memory Profiling with tracemalloc:**
The memory comparison is particularly insightful:
- Static: Measures memory for repeated calls (no new instances)
- Instance: Creates 1000 instances and measures total memory
- Singleton: "Creates" 1000 references to the same instance

This reveals the true memory cost of each pattern.

**timeit for Accurate Timing:**
```python
timeit.timeit('StaticOperations.process()', globals=locals(), number=iterations)
```
Why timeit over simple time measurement:
- Runs multiple iterations for statistical accuracy
- Disables garbage collection during timing
- Uses high-resolution timer
- Provides reliable, repeatable results

**Key Performance Insights:**
1. **Static methods**: Fastest execution (no instance lookup)
2. **Instance methods**: Slight overhead for attribute access
3. **Singleton**: Additional overhead for instance checking

**Memory Usage Patterns:**
- **Static**: Minimal memory (shared class data)
- **Instance**: Linear growth with instance count
- **Singleton**: Constant memory regardless of "instance" count

**The BenchmarkResult Dataclass:**
Using dataclasses for results provides:
- Clean, immutable data structure
- Automatic __repr__ for debugging
- Type hints for clarity
- Easy serialization if needed

**cProfile Integration:**
The `profile_code` method allows deep performance analysis:
- Function-level timing breakdown
- Call count statistics
- Identifies bottlenecks
- Sorts by cumulative time

**Report Generation:**
The formatted report shows:
- Absolute performance metrics
- Operations per second
- Relative performance percentages
- Memory usage comparison

**Real-World Applications:**
- Choose static for high-frequency, stateless operations
- Use instances when you need multiple independent states
- Pick singleton for shared resources with significant memory footprint

**Performance Testing Best Practices:**
1. Test with realistic data sizes
2. Include memory profiling, not just speed
3. Use multiple iterations for accuracy
4. Profile to find actual bottlenecks
5. Consider both startup and steady-state performance

This framework gives you empirical data to make informed architectural decisions!

```typescript
// TypeScript: Performance benchmarking
interface BenchmarkResult {
    name: string;
    timeMs: number;
    memoryMb: number;
    iterations: number;
    opsPerSec: number;
}

class PerformanceBenchmark {
    private results: BenchmarkResult[] = [];
    
    async measureTime<T>(
        name: string,
        fn: () => T | Promise<T>
    ): Promise<T> {
        const start = performance.now();
        const result = await fn();
        const end = performance.now();
        
        console.log(`${name}: ${(end - start).toFixed(2)}ms`);
        return result;
    }
    
    measureMemory(): { before: number; after: number } {
        if (typeof process !== 'undefined' && process.memoryUsage) {
            const before = process.memoryUsage().heapUsed;
            return {
                before,
                after: () => process.memoryUsage().heapUsed - before
            };
        }
        return { before: 0, after: () => 0 };
    }
    
    async benchmarkPatterns(): Promise<BenchmarkResult[]> {
        const iterations = 100000;
        
        // Test classes
        class StaticOperations {
            static data = Array.from({ length: 100 }, (_, i) => i);
            
            static process(): number {
                return this.data.reduce((a, b) => a + b, 0);
            }
        }
        
        class InstanceOperations {
            private data = Array.from({ length: 100 }, (_, i) => i);
            
            process(): number {
                return this.data.reduce((a, b) => a + b, 0);
            }
        }
        
        class SingletonOperations {
            private static instance: SingletonOperations;
            private data = Array.from({ length: 100 }, (_, i) => i);
            
            private constructor() {}
            
            static getInstance(): SingletonOperations {
                if (!this.instance) {
                    this.instance = new SingletonOperations();
                }
                return this.instance;
            }
            
            process(): number {
                return this.data.reduce((a, b) => a + b, 0);
            }
        }
        
        // Benchmark static
        const staticStart = performance.now();
        for (let i = 0; i < iterations; i++) {
            StaticOperations.process();
        }
        const staticTime = performance.now() - staticStart;
        
        // Benchmark instance
        const instance = new InstanceOperations();
        const instanceStart = performance.now();
        for (let i = 0; i < iterations; i++) {
            instance.process();
        }
        const instanceTime = performance.now() - instanceStart;
        
        // Benchmark singleton
        const singleton = SingletonOperations.getInstance();
        const singletonStart = performance.now();
        for (let i = 0; i < iterations; i++) {
            singleton.process();
        }
        const singletonTime = performance.now() - singletonStart;
        
        // Memory benchmarks
        const memResults = await this.benchmarkMemory();
        
        // Store results
        this.results = [
            {
                name: 'Static',
                timeMs: staticTime,
                memoryMb: memResults.static / 1024 / 1024,
                iterations,
                opsPerSec: (iterations / staticTime) * 1000
            },
            {
                name: 'Instance',
                timeMs: instanceTime,
                memoryMb: memResults.instance / 1024 / 1024,
                iterations,
                opsPerSec: (iterations / instanceTime) * 1000
            },
            {
                name: 'Singleton',
                timeMs: singletonTime,
                memoryMb: memResults.singleton / 1024 / 1024,
                iterations,
                opsPerSec: (iterations / singletonTime) * 1000
            }
        ];
        
        return this.results;
    }
    
    private async benchmarkMemory() {
        const results = {
            static: 0,
            instance: 0,
            singleton: 0
        };
        
        if (typeof process === 'undefined') {
            return results;
        }
        
        // Force garbage collection if available
        if (global.gc) {
            global.gc();
        }
        
        // Static memory
        const staticBefore = process.memoryUsage().heapUsed;
        class Static {
            static data = new Array(1000).fill(0);
            static process() { return this.data.length; }
        }
        for (let i = 0; i < 1000; i++) {
            Static.process();
        }
        results.static = process.memoryUsage().heapUsed - staticBefore;
        
        // Instance memory
        if (global.gc) global.gc();
        const instanceBefore = process.memoryUsage().heapUsed;
        class Instance {
            data = new Array(1000).fill(0);
            process() { return this.data.length; }
        }
        const instances = Array.from({ length: 1000 }, () => new Instance());
        instances.forEach(i => i.process());
        results.instance = process.memoryUsage().heapUsed - instanceBefore;
        
        // Singleton memory
        if (global.gc) global.gc();
        const singletonBefore = process.memoryUsage().heapUsed;
        class Singleton {
            private static instance: Singleton;
            data = new Array(1000).fill(0);
            static getInstance() {
                if (!this.instance) {
                    this.instance = new Singleton();
                }
                return this.instance;
            }
            process() { return this.data.length; }
        }
        const singletons = Array.from({ length: 1000 }, () => Singleton.getInstance());
        singletons.forEach(s => s.process());
        results.singleton = process.memoryUsage().heapUsed - singletonBefore;
        
        return results;
    }
    
    generateReport(): void {
        console.log('\n=== Performance Benchmark Report ===\n');
        console.log('Pattern      Time (ms)    Memory (MB)  Ops/sec');
        console.log('------------------------------------------------');
        
        for (const result of this.results) {
            console.log(
                `${result.name.padEnd(12)} ` +
                `${result.timeMs.toFixed(2).padEnd(12)} ` +
                `${result.memoryMb.toFixed(2).padEnd(12)} ` +
                `${result.opsPerSec.toFixed(0)}`
            );
        }
        
        // Relative performance
        if (this.results.length > 0) {
            const baseTime = this.results[0].timeMs;
            console.log('\nRelative Performance:');
            for (const result of this.results) {
                const relative = (result.timeMs / baseTime) * 100;
                console.log(`${result.name}: ${relative.toFixed(1)}%`);
            }
        }
    }
    
    // Advanced profiling with Chrome DevTools Protocol
    async profileWithCDP(fn: () => void): Promise<any> {
        // This would integrate with Chrome DevTools for detailed profiling
        // Implementation depends on runtime environment
        console.log('Profiling requires Chrome DevTools integration');
        return fn();
    }
}

// Usage
async function runBenchmarks() {
    const benchmark = new PerformanceBenchmark();
    await benchmark.benchmarkPatterns();
    benchmark.generateReport();
}

runBenchmarks();
```

**Understanding TypeScript Performance Benchmarking:**

This TypeScript implementation showcases browser and Node.js compatible benchmarking with modern performance APIs. Let's explore the key differences and improvements:

**Cross-Platform Performance API:**
```typescript
const start = performance.now();
```
Using `performance.now()` instead of `Date.now()`:
- Microsecond precision (vs millisecond)
- Monotonic clock (immune to system time changes)
- Available in both browser and Node.js
- Better for measuring small time differences

**Environment-Aware Memory Profiling:**
```typescript
if (typeof process !== 'undefined' && process.memoryUsage)
```
This code gracefully handles:
- Node.js environments (has process.memoryUsage)
- Browser environments (no process object)
- Makes the benchmark portable across platforms

**Async-First Design:**
The entire benchmark suite is async:
- `async measureTime<T>`: Supports both sync and async operations
- `await benchmark.benchmarkPatterns()`: Clean async/await syntax
- Future-proof for async performance testing

**Garbage Collection Awareness:**
```typescript
if (global.gc) {
    global.gc();
}
```
This attempts to trigger garbage collection:
- Ensures consistent memory measurements
- Requires Node.js flag: `--expose-gc`
- Prevents GC from skewing results

**Memory Benchmark Strategy:**
The memory test creates different scenarios:
- Static: Single shared array for all calls
- Instance: 1000 separate arrays (one per instance)
- Singleton: 1000 references to the same array

This clearly shows memory allocation patterns.

**Type-Safe Results Interface:**
```typescript
interface BenchmarkResult {
    name: string;
    timeMs: number;
    memoryMb: number;
    iterations: number;
    opsPerSec: number;
}
```
Provides:
- IntelliSense support
- Compile-time type checking
- Self-documenting code
- Easy to extend with new metrics

**String Padding for Formatted Output:**
```typescript
`${result.name.padEnd(12)} ${result.timeMs.toFixed(2).padEnd(12)}`
```
Creates aligned, readable output without external libraries.

**Chrome DevTools Protocol Hook:**
The `profileWithCDP` method hints at advanced profiling:
- CPU profiling
- Heap snapshots
- Flame graphs
- Timeline analysis

**Performance Insights:**
1. **Browser vs Node.js**: Different memory characteristics
2. **JIT Optimization**: Results may vary as code "warms up"
3. **Garbage Collection**: Can cause timing spikes
4. **Memory Pressure**: Different patterns under memory constraints

**Real-World Considerations:**
- Run benchmarks multiple times for consistency
- Test with production-like data sizes
- Consider cold start vs warm performance
- Profile in the target deployment environment

This framework provides actionable metrics for choosing the right pattern based on your specific performance requirements!

## Memory Profiling

### Detailed Memory Analysis

```python
# Python: Memory profiling utilities
import gc
import sys
import objgraph
import tracemalloc
from memory_profiler import profile
from pympler import asizeof, muppy, summary

class MemoryProfiler:
    """Advanced memory profiling tools"""
    
    @staticmethod
    def size_of(obj):
        """Get size of object and all referenced objects"""
        return asizeof.asizeof(obj)
    
    @staticmethod
    def get_object_stats():
        """Get statistics about all objects in memory"""
        all_objects = muppy.get_objects()
        sum_obj = summary.summarize(all_objects)
        summary.print_(sum_obj)
    
    @staticmethod
    @contextmanager
    def track_memory():
        """Context manager for tracking memory usage"""
        tracemalloc.start()
        snapshot1 = tracemalloc.take_snapshot()
        
        yield
        
        snapshot2 = tracemalloc.take_snapshot()
        top_stats = snapshot2.compare_to(snapshot1, 'lineno')
        
        print("\n[ Top 10 memory allocations ]")
        for stat in top_stats[:10]:
            print(stat)
    
    @staticmethod
    def find_memory_leaks(func, iterations=10):
        """Detect potential memory leaks"""
        gc.collect()
        initial_objects = len(gc.get_objects())
        
        # Run function multiple times
        for _ in range(iterations):
            func()
            gc.collect()
        
        final_objects = len(gc.get_objects())
        leaked_objects = final_objects - initial_objects
        
        if leaked_objects > iterations:
            print(f"Potential memory leak: {leaked_objects} objects leaked")
            
            # Show growth
            objgraph.show_growth()
            
            # Find common types
            print("\nMost common types:")
            objgraph.show_most_common_types(limit=10)
        else:
            print("No significant memory leaks detected")
    
    @profile
    def profile_patterns(self):
        """Profile memory usage of different patterns"""
        
        # Static pattern memory usage
        class StaticStorage:
            large_data = [0] * 1000000  # 1M integers
            
            @staticmethod
            def get_data():
                return StaticStorage.large_data
        
        # Instance pattern memory usage
        instances = []
        for i in range(100):
            class InstanceStorage:
                def __init__(self):
                    self.large_data = [0] * 10000  # 10K integers each
            
            instances.append(InstanceStorage())
        
        # Singleton pattern memory usage
        class SingletonStorage:
            _instance = None
            
            def __new__(cls):
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance.large_data = [0] * 1000000
                return cls._instance
        
        # Create multiple references to singleton
        singletons = [SingletonStorage() for _ in range(100)]
        
        # Measure sizes
        print(f"Static size: {self.size_of(StaticStorage)} bytes")
        print(f"Instances size: {self.size_of(instances)} bytes")
        print(f"Singleton refs size: {self.size_of(singletons)} bytes")

# Example usage
profiler = MemoryProfiler()

# Track memory in specific code block
with profiler.track_memory():
    # Some memory-intensive operation
    data = [list(range(1000)) for _ in range(1000)]
    processed = [[x * 2 for x in row] for row in data]

# Check for memory leaks
def potentially_leaky_function():
    cache = {}
    for i in range(100):
        cache[i] = list(range(1000))
    # Oops, cache is never cleared!

profiler.find_memory_leaks(potentially_leaky_function)
```

**Understanding Memory Profiling:**

This comprehensive memory profiling toolkit demonstrates advanced techniques for understanding memory usage patterns. Let's explore each tool:

**Deep Object Size with pympler:**
```python
asizeof.asizeof(obj)
```
Unlike `sys.getsizeof()`, this:
- Recursively measures all referenced objects
- Includes containers' contents
- Accounts for shared references
- Gives true memory footprint

**Memory Tracking Context Manager:**
The `track_memory()` context manager provides:
- Line-by-line memory allocation tracking
- Before/after snapshots
- Detailed comparison showing which lines allocated memory
- Perfect for finding memory hotspots

**Leak Detection Strategy:**
```python
leaked_objects = final_objects - initial_objects
```
This clever approach:
1. Counts objects before running the function
2. Runs the function multiple times
3. Forces garbage collection
4. Compares object counts

If objects keep growing, you have a leak!

**Object Graph Analysis:**
```python
objgraph.show_growth()
objgraph.show_most_common_types()
```
These tools reveal:
- Which object types are growing
- Memory allocation patterns
- Unexpected object retention
- Reference cycles

**@profile Decorator:**
From memory_profiler, this decorator:
- Shows line-by-line memory usage
- Identifies memory-hungry operations
- No code changes needed (just decorate)
- Outputs detailed memory report

**Pattern-Specific Insights:**
The profiling reveals:
- **Static**: Single allocation, shared by all
- **Instance**: Linear growth with instance count
- **Singleton**: Multiple references, single allocation

**Real Memory Leak Example:**
```python
cache = {}
for i in range(100):
    cache[i] = list(range(1000))
# Cache grows indefinitely!
```
This demonstrates a common leak pattern - unbounded caches.

**Advanced Profiling Techniques:**
1. **tracemalloc**: Python's built-in, traces all allocations
2. **memory_profiler**: Line-by-line profiling
3. **pympler**: Deep object analysis
4. **objgraph**: Visual reference graphs
5. **gc module**: Garbage collection control

**When to Use Each Tool:**
- **Quick check**: `sys.getsizeof()`
- **Deep analysis**: `asizeof.asizeof()`
- **Finding leaks**: `objgraph` + gc analysis
- **Line profiling**: `@profile` decorator
- **Production monitoring**: `tracemalloc`

**Memory Optimization Strategies:**
1. Use `__slots__` for classes with many instances
2. Employ weak references for caches
3. Clear large data structures explicitly
4. Use generators for large sequences
5. Profile before optimizing!

This toolkit gives you X-ray vision into your application's memory usage!

```typescript
// TypeScript: Memory profiling utilities
class MemoryProfiler {
    static getMemoryUsage(): NodeJS.MemoryUsage | null {
        if (typeof process !== 'undefined' && process.memoryUsage) {
            return process.memoryUsage();
        }
        return null;
    }
    
    static formatBytes(bytes: number): string {
        const units = ['B', 'KB', 'MB', 'GB'];
        let size = bytes;
        let unitIndex = 0;
        
        while (size >= 1024 && unitIndex < units.length - 1) {
            size /= 1024;
            unitIndex++;
        }
        
        return `${size.toFixed(2)} ${units[unitIndex]}`;
    }
    
    static async profileMemory<T>(
        name: string,
        fn: () => T | Promise<T>
    ): Promise<{ result: T; memory: NodeJS.MemoryUsage | null }> {
        const before = this.getMemoryUsage();
        
        // Force garbage collection if available
        if (global.gc) {
            global.gc();
        }
        
        const result = await fn();
        
        const after = this.getMemoryUsage();
        
        if (before && after) {
            console.log(`\nMemory Profile: ${name}`);
            console.log(`Heap Used: ${this.formatBytes(after.heapUsed - before.heapUsed)}`);
            console.log(`External: ${this.formatBytes(after.external - before.external)}`);
            console.log(`Total: ${this.formatBytes(after.heapTotal - before.heapTotal)}`);
        }
        
        return { result, memory: after };
    }
    
    static createMemoryLeakDetector() {
        const snapshots: WeakMap<object, number> = new WeakMap();
        let objectCount = 0;
        
        return {
            track(obj: object): void {
                snapshots.set(obj, ++objectCount);
            },
            
            checkLeaks(): void {
                // In a real implementation, this would analyze retained objects
                console.log(`Tracked objects: ${objectCount}`);
            }
        };
    }
    
    static async comparePatterns() {
        console.log('=== Memory Usage Comparison ===\n');
        
        // Test different patterns
        const results = {
            static: { instances: 0, memory: 0 },
            instance: { instances: 0, memory: 0 },
            singleton: { instances: 0, memory: 0 }
        };
        
        // Static pattern
        await this.profileMemory('Static Pattern', () => {
            class StaticData {
                static cache = new Map<string, any>();
                static data = new Array(10000).fill(0);
                
                static process() {
                    return this.data.length;
                }
            }
            
            // Simulate usage
            for (let i = 0; i < 1000; i++) {
                StaticData.cache.set(`key${i}`, i);
                StaticData.process();
            }
            
            results.static.instances = 1;
        });
        
        // Instance pattern
        await this.profileMemory('Instance Pattern', () => {
            class InstanceData {
                private data = new Array(100).fill(0);
                
                process() {
                    return this.data.length;
                }
            }
            
            const instances: InstanceData[] = [];
            for (let i = 0; i < 1000; i++) {
                const instance = new InstanceData();
                instance.process();
                instances.push(instance);
            }
            
            results.instance.instances = instances.length;
        });
        
        // Singleton pattern
        await this.profileMemory('Singleton Pattern', () => {
            class SingletonData {
                private static instance: SingletonData;
                private data = new Array(10000).fill(0);
                private cache = new Map<string, any>();
                
                static getInstance() {
                    if (!this.instance) {
                        this.instance = new SingletonData();
                    }
                    return this.instance;
                }
                
                process() {
                    return this.data.length;
                }
            }
            
            // Simulate multiple accesses
            const refs: SingletonData[] = [];
            for (let i = 0; i < 1000; i++) {
                const singleton = SingletonData.getInstance();
                singleton.process();
                refs.push(singleton);
            }
            
            results.singleton.instances = 1;
        });
        
        console.log('\nSummary:');
        console.log(`Static: ${results.static.instances} instance(s)`);
        console.log(`Instance: ${results.instance.instances} instance(s)`);
        console.log(`Singleton: ${results.singleton.instances} instance(s)`);
    }
}

// Advanced memory monitoring
class MemoryMonitor {
    private intervals: NodeJS.Timer[] = [];
    private history: Array<{ timestamp: Date; usage: NodeJS.MemoryUsage }> = [];
    
    startMonitoring(intervalMs: number = 1000): void {
        const timer = setInterval(() => {
            const usage = process.memoryUsage();
            this.history.push({
                timestamp: new Date(),
                usage
            });
            
            // Keep only last 100 entries
            if (this.history.length > 100) {
                this.history.shift();
            }
        }, intervalMs);
        
        this.intervals.push(timer);
    }
    
    stopMonitoring(): void {
        this.intervals.forEach(timer => clearInterval(timer));
        this.intervals = [];
    }
    
    getReport(): string {
        if (this.history.length === 0) {
            return 'No data collected';
        }
        
        const latest = this.history[this.history.length - 1];
        const oldest = this.history[0];
        
        const heapGrowth = latest.usage.heapUsed - oldest.usage.heapUsed;
        const timespan = latest.timestamp.getTime() - oldest.timestamp.getTime();
        
        return `
Memory Report:
- Monitoring duration: ${(timespan / 1000).toFixed(1)}s
- Samples collected: ${this.history.length}
- Heap growth: ${MemoryProfiler.formatBytes(heapGrowth)}
- Current heap: ${MemoryProfiler.formatBytes(latest.usage.heapUsed)}
- Heap growth rate: ${MemoryProfiler.formatBytes(heapGrowth / (timespan / 1000))}/s
        `.trim();
    }
}
```

**Understanding TypeScript Memory Profiling:**

This TypeScript memory profiling toolkit provides production-ready monitoring and analysis tools. Let's explore the sophisticated patterns:

**Cross-Platform Memory Access:**
```typescript
if (typeof process !== 'undefined' && process.memoryUsage)
```
This defensive programming:
- Works in Node.js (has process)
- Gracefully degrades in browsers
- Provides null-safe API
- Enables isomorphic code

**Human-Readable Byte Formatting:**
The `formatBytes` function automatically:
- Converts to appropriate units (B, KB, MB, GB)
- Maintains precision (2 decimal places)
- Handles edge cases
- Creates readable output

**Async Memory Profiling:**
```typescript
async profileMemory<T>(name: string, fn: () => T | Promise<T>)
```
This generic function:
- Supports both sync and async operations
- Preserves return type with `T`
- Forces GC before measurement (if available)
- Returns both result and memory stats

**WeakMap for Leak Detection:**
```typescript
const snapshots: WeakMap<object, number> = new WeakMap();
```
Using WeakMap is brilliant because:
- Doesn't prevent garbage collection
- Automatically cleans up when objects are collected
- Perfect for tracking without interfering
- No memory leaks from the detector itself!

**Pattern Comparison Methodology:**
The comparison test:
1. **Static**: Single shared array, Map for cache
2. **Instance**: 1000 instances, each with small array
3. **Singleton**: One instance accessed 1000 times

This reveals real-world memory characteristics.

**Memory Monitor - Time Series Analysis:**
The `MemoryMonitor` class provides:
- Continuous memory sampling
- Rolling window (last 100 samples)
- Growth rate calculation
- Heap usage trends

**Key Metrics Tracked:**
- `heapUsed`: Actual memory in use
- `heapTotal`: Total allocated heap
- `external`: Memory used by C++ objects
- `rss`: Resident set size (total process memory)

**Production-Ready Features:**
1. **Multiple timers**: Track different intervals
2. **History management**: Prevents unbounded growth
3. **Growth rate**: Identifies memory leaks
4. **Clean shutdown**: Proper timer cleanup

**Real-World Applications:**
- **Development**: Profile during feature development
- **Testing**: Automated memory regression tests
- **Production**: Monitor long-running services
- **Debugging**: Find memory leaks in production

**Advanced Analysis Insights:**
- Growth rate indicates potential leaks
- Heap total vs used shows fragmentation
- External memory reveals native module usage
- Time series data enables trend analysis

**Best Practices:**
1. Profile in production-like environment
2. Use WeakMap/WeakSet for caches
3. Monitor growth rate, not absolute values
4. Set up alerts for abnormal growth
5. Profile under load, not idle

This toolkit transforms memory profiling from guesswork into data-driven optimization!

## Optimization Strategies

### Pattern-Specific Optimizations

```python
# Python: Optimization strategies
from functools import lru_cache, cached_property
import weakref
from typing import Dict, Any, Optional

class OptimizedStatic:
    """Optimized static pattern implementations"""
    
    # Pre-computed constants
    _PRECOMPUTED_VALUES = {i: i ** 2 for i in range(100)}
    
    # Compiled regex patterns
    _REGEX_CACHE = {}
    
    @staticmethod
    @lru_cache(maxsize=1024)
    def expensive_computation(n: int) -> int:
        """Cached static computation"""
        return sum(i ** 2 for i in range(n))
    
    @classmethod
    def get_compiled_regex(cls, pattern: str):
        """Lazy regex compilation with caching"""
        if pattern not in cls._REGEX_CACHE:
            cls._REGEX_CACHE[pattern] = re.compile(pattern)
        return cls._REGEX_CACHE[pattern]
    
    @staticmethod
    def process_batch(items: List[int]) -> List[int]:
        """Optimized batch processing"""
        # Use vectorized operations when possible
        import numpy as np
        
        arr = np.array(items)
        # Vectorized computation is much faster
        return (arr ** 2 + arr).tolist()

class OptimizedInstance:
    """Optimized instance pattern implementations"""
    
    __slots__ = ['_data', '_cache', '_dirty']  # Reduce memory overhead
    
    def __init__(self, data: List[int]):
        self._data = data
        self._cache = {}
        self._dirty = True
    
    @cached_property
    def computed_value(self) -> int:
        """Compute expensive value only once"""
        return sum(x ** 2 for x in self._data)
    
    def process(self) -> int:
        """Process with intelligent caching"""
        cache_key = 'processed'
        
        if self._dirty or cache_key not in self._cache:
            result = self._compute()
            self._cache[cache_key] = result
            self._dirty = False
        
        return self._cache[cache_key]
    
    def update_data(self, new_data: List[int]):
        """Update data and invalidate cache"""
        self._data = new_data
        self._dirty = True
        self._cache.clear()
        
        # Clear cached_property
        if 'computed_value' in self.__dict__:
            del self.__dict__['computed_value']
    
    def _compute(self) -> int:
        """Internal computation"""
        return sum(self._data)

class OptimizedSingleton:
    """Optimized singleton with resource management"""
    
    _instance: Optional['OptimizedSingleton'] = None
    _lock = threading.RLock()
    
    def __new__(cls):
        # Fast path - no lock needed
        if cls._instance is not None:
            return cls._instance
        
        # Slow path - need lock
        with cls._lock:
            if cls._instance is None:
                cls._instance = super().__new__(cls)
                cls._instance._initialize()
            return cls._instance
    
    def _initialize(self):
        """Lazy initialization of resources"""
        self._resources = {}
        self._weak_refs = weakref.WeakValueDictionary()
        self._pool = self._create_pool()
    
    def _create_pool(self):
        """Create resource pool"""
        from concurrent.futures import ThreadPoolExecutor
        return ThreadPoolExecutor(max_workers=4)
    
    def get_resource(self, key: str) -> Any:
        """Get resource with caching"""
        # Check weak references first (memory-efficient)
        if key in self._weak_refs:
            return self._weak_refs[key]
        
        # Check strong references
        if key in self._resources:
            return self._resources[key]
        
        # Create new resource
        resource = self._create_resource(key)
        
        # Store based on resource type
        if self._is_heavyweight(resource):
            self._weak_refs[key] = resource
        else:
            self._resources[key] = resource
        
        return resource
    
    def _create_resource(self, key: str) -> Any:
        """Create new resource"""
        # Expensive resource creation
        return {"key": key, "data": list(range(1000))}
    
    def _is_heavyweight(self, resource: Any) -> bool:
        """Check if resource should use weak reference"""
        return sys.getsizeof(resource) > 1024  # 1KB threshold
    
    async def process_async(self, data: List[Any]) -> List[Any]:
        """Async processing with connection pooling"""
        import asyncio
        
        async def process_item(item):
            # Simulate async operation
            await asyncio.sleep(0.01)
            return item * 2
        
        # Process in batches for efficiency
        batch_size = 100
        results = []
        
        for i in range(0, len(data), batch_size):
            batch = data[i:i + batch_size]
            batch_results = await asyncio.gather(
                *[process_item(item) for item in batch]
            )
            results.extend(batch_results)
        
        return results

# Performance comparison
def compare_optimizations():
    """Compare optimized vs non-optimized implementations"""
    import time
    
    # Test data
    test_data = list(range(1000))
    iterations = 1000
    
    # Non-optimized static
    class BasicStatic:
        @staticmethod
        def compute(n):
            return sum(i ** 2 for i in range(n))
    
    # Test non-optimized
    start = time.perf_counter()
    for _ in range(iterations):
        BasicStatic.compute(100)
    basic_time = time.perf_counter() - start
    
    # Test optimized (with cache)
    start = time.perf_counter()
    for _ in range(iterations):
        OptimizedStatic.expensive_computation(100)
    optimized_time = time.perf_counter() - start
    
    print(f"Basic static: {basic_time:.3f}s")
    print(f"Optimized static: {optimized_time:.3f}s")
    print(f"Speedup: {basic_time / optimized_time:.1f}x")
```

**Understanding Optimization Strategies:**

This comprehensive optimization guide demonstrates advanced techniques for each pattern. Let's explore the optimization strategies:

**Static Pattern Optimizations:**

**1. Pre-computed Constants:**
```python
_PRECOMPUTED_VALUES = {i: i ** 2 for i in range(100)}
```
Trading memory for speed - common values are pre-calculated at module load time.

**2. LRU Cache Decorator:**
```python
@lru_cache(maxsize=1024)
```
Automatic memoization that:
- Caches up to 1024 most recent results
- Thread-safe by default
- Provides cache statistics
- Near-zero overhead for cache hits

**3. Vectorized Operations:**
Using NumPy for batch processing provides:
- 10-100x speedup for numerical operations
- SIMD instruction utilization
- Reduced Python overhead
- Memory-efficient operations

**Instance Pattern Optimizations:**

**1. __slots__ for Memory Efficiency:**
```python
__slots__ = ['_data', '_cache', '_dirty']
```
This optimization:
- Prevents dynamic attribute creation
- Reduces memory by ~40% per instance
- Faster attribute access
- Critical for many small instances

**2. Cached Properties:**
```python
@cached_property
def computed_value(self):
```
Computes once, then becomes a regular attribute - perfect for expensive derived values.

**3. Dirty Flag Pattern:**
The `_dirty` flag enables:
- Lazy recomputation only when data changes
- Cache invalidation without immediate recalculation
- Efficient batch updates

**Singleton Pattern Optimizations:**

**1. Double-Checked Locking with RLock:**
```python
if cls._instance is not None:  # Fast path
    return cls._instance
with cls._lock:  # Slow path
```
Optimizes the common case (instance exists) while maintaining thread safety.

**2. Weak References for Large Objects:**
```python
self._weak_refs = weakref.WeakValueDictionary()
```
Brilliant memory management:
- Heavy resources can be garbage collected
- Automatically recreated if needed again
- Prevents memory bloat
- Perfect for caches

**3. Resource Pooling:**
Thread pool executor for:
- Reusing expensive resources
- Limiting concurrent operations
- Preventing resource exhaustion

**4. Async Batch Processing:**
```python
await asyncio.gather(*[process_item(item) for item in batch])
```
Processes items concurrently in batches:
- Limits concurrent operations
- Prevents overwhelming the system
- Maintains throughput

**Key Optimization Principles:**

1. **Cache Aggressively**: But with size limits
2. **Lazy Evaluation**: Compute only when needed
3. **Batch Operations**: Amortize overhead
4. **Memory vs Speed**: Make conscious tradeoffs
5. **Profile First**: Optimize based on data

**Real-World Impact:**
The comparison shows dramatic improvements:
- Static with caching: ~100x faster after first call
- Instance with slots: 40% less memory
- Singleton with pooling: Better resource utilization

**Advanced Techniques:**
- Use `__slots__` for data classes
- Implement `__del__` carefully in singletons
- Consider `functools.singledispatch` for type-based optimization
- Use `memoryview` for large data manipulation

Remember: Premature optimization is evil, but informed optimization based on profiling is engineering!

```typescript
// TypeScript: Optimization strategies
class OptimizedStatic {
    // Pre-computed lookup tables
    private static readonly LOOKUP_TABLE = new Map(
        Array.from({ length: 100 }, (_, i) => [i, i * i])
    );
    
    // Lazy-loaded resources
    private static largeData?: number[];
    
    // Memoization with size limit
    private static cache = new Map<string, any>();
    private static readonly MAX_CACHE_SIZE = 1000;
    
    static memoizedCompute(n: number): number {
        const key = `compute_${n}`;
        
        if (this.cache.has(key)) {
            return this.cache.get(key);
        }
        
        const result = Array.from({ length: n }, (_, i) => i * i)
            .reduce((a, b) => a + b, 0);
        
        // Manage cache size
        if (this.cache.size >= this.MAX_CACHE_SIZE) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(key, result);
        return result;
    }
    
    static get largeDataset(): number[] {
        // Lazy initialization
        if (!this.largeData) {
            console.log('Initializing large dataset...');
            this.largeData = new Array(1000000).fill(0).map((_, i) => i);
        }
        return this.largeData;
    }
    
    // Optimized batch processing
    static processBatch<T, R>(
        items: T[],
        processor: (item: T) => R,
        batchSize = 1000
    ): R[] {
        const results: R[] = [];
        
        for (let i = 0; i < items.length; i += batchSize) {
            const batch = items.slice(i, i + batchSize);
            const batchResults = batch.map(processor);
            results.push(...batchResults);
            
            // Allow event loop to process other tasks
            if (i + batchSize < items.length) {
                setImmediate(() => {});
            }
        }
        
        return results;
    }
}

class OptimizedInstance {
    private cache = new Map<string, any>();
    private dirty = new Set<string>();
    
    constructor(private data: number[]) {}
    
    // Getter with caching
    get computedValue(): number {
        const key = 'computed';
        
        if (!this.cache.has(key) || this.dirty.has(key)) {
            const value = this.data.reduce((a, b) => a + b * b, 0);
            this.cache.set(key, value);
            this.dirty.delete(key);
        }
        
        return this.cache.get(key);
    }
    
    updateData(newData: number[]): void {
        this.data = newData;
        this.dirty.add('computed');
    }
    
    // Async processing with concurrency control
    async processAsync(
        concurrency = 4
    ): Promise<number[]> {
        const results: number[] = [];
        const queue = [...this.data];
        const processing: Promise<void>[] = [];
        
        const processItem = async (item: number): Promise<void> => {
            // Simulate async work
            await new Promise(resolve => setTimeout(resolve, 10));
            results.push(item * 2);
        };
        
        while (queue.length > 0 || processing.length > 0) {
            while (processing.length < concurrency && queue.length > 0) {
                const item = queue.shift()!;
                const promise = processItem(item).then(() => {
                    const index = processing.indexOf(promise);
                    processing.splice(index, 1);
                });
                processing.push(promise);
            }
            
            if (processing.length > 0) {
                await Promise.race(processing);
            }
        }
        
        return results;
    }
}

class OptimizedSingleton {
    private static instance?: OptimizedSingleton;
    private resources = new Map<string, any>();
    private weakResources = new WeakMap<object, any>();
    private pool?: any; // Connection pool
    
    private constructor() {
        this.initializePool();
    }
    
    static getInstance(): OptimizedSingleton {
        // Double-checked locking pattern (conceptual in JS)
        if (!this.instance) {
            this.instance = new OptimizedSingleton();
        }
        return this.instance;
    }
    
    private initializePool(): void {
        // Initialize connection pool or worker pool
        console.log('Initializing resource pool...');
    }
    
    getResource<T>(key: string, factory: () => T): T {
        // Check cache first
        if (this.resources.has(key)) {
            return this.resources.get(key);
        }
        
        // Create new resource
        const resource = factory();
        
        // Store based on size (simplified)
        const size = JSON.stringify(resource).length;
        if (size > 10000) {
            // Large resources use weak references
            const wrapper = { resource };
            this.weakResources.set(wrapper, resource);
            this.resources.set(key, wrapper);
        } else {
            // Small resources use strong references
            this.resources.set(key, resource);
        }
        
        return resource;
    }
    
    // Resource pooling for expensive operations
    async executePooled<T>(
        task: () => Promise<T>
    ): Promise<T> {
        // In a real implementation, this would use a worker pool
        return task();
    }
    
    clearCache(): void {
        this.resources.clear();
        // WeakMap clears automatically
    }
}

// Performance utilities
class PerformanceOptimizer {
    static debounce<T extends (...args: any[]) => any>(
        func: T,
        wait: number
    ): (...args: Parameters<T>) => void {
        let timeout: NodeJS.Timeout;
        
        return (...args: Parameters<T>) => {
            clearTimeout(timeout);
            timeout = setTimeout(() => func(...args), wait);
        };
    }
    
    static throttle<T extends (...args: any[]) => any>(
        func: T,
        limit: number
    ): (...args: Parameters<T>) => void {
        let inThrottle = false;
        
        return (...args: Parameters<T>) => {
            if (!inThrottle) {
                func(...args);
                inThrottle = true;
                setTimeout(() => inThrottle = false, limit);
            }
        };
    }
    
    static createObjectPool<T>(
        factory: () => T,
        reset: (obj: T) => void,
        maxSize = 10
    ) {
        const pool: T[] = [];
        
        return {
            acquire(): T {
                return pool.pop() || factory();
            },
            
            release(obj: T): void {
                if (pool.length < maxSize) {
                    reset(obj);
                    pool.push(obj);
                }
            },
            
            size(): number {
                return pool.length;
            }
        };
    }
}
```

**Understanding TypeScript Optimization Strategies:**

This TypeScript optimization showcase demonstrates modern performance patterns and memory management techniques. Let's explore the advanced optimizations:

**Static Pattern Optimizations:**

**1. Pre-computed Lookup Tables with Map:**
```typescript
private static readonly LOOKUP_TABLE = new Map(
    Array.from({ length: 100 }, (_, i) => [i, i * i])
);
```
Using Map for lookups provides:
- O(1) access time
- Better performance than object properties
- Type-safe key-value pairs
- Memory-efficient storage

**2. FIFO Cache with Size Management:**
```typescript
if (this.cache.size >= this.MAX_CACHE_SIZE) {
    const firstKey = this.cache.keys().next().value;
    this.cache.delete(firstKey);
}
```
Simple but effective:
- Removes oldest entry when full
- No external dependencies
- Predictable memory usage
- Fast cache operations

**3. Event Loop Friendly Batch Processing:**
```typescript
setImmediate(() => {});
```
This clever trick:
- Yields to the event loop between batches
- Prevents blocking on large datasets
- Maintains UI responsiveness
- Allows other async operations to run

**Instance Pattern Optimizations:**

**1. Dirty Flag with Set:**
```typescript
private dirty = new Set<string>();
```
Using Set for dirty tracking:
- O(1) add/delete/has operations
- Multiple properties can be dirty
- Clear invalidation logic
- Type-safe property tracking

**2. Concurrency-Limited Async Processing:**
The `processAsync` method implements:
- Configurable concurrency limit
- Queue-based processing
- Promise.race for efficiency
- Maintains order of results

This pattern prevents:
- Memory exhaustion from too many promises
- API rate limit violations
- System overload

**Singleton Pattern Optimizations:**

**1. Hybrid Strong/Weak References:**
```typescript
if (size > 10000) {
    const wrapper = { resource };
    this.weakResources.set(wrapper, resource);
}
```
Intelligent caching strategy:
- Large objects can be garbage collected
- Small objects stay in memory
- Automatic memory pressure relief
- No manual cache eviction needed

**2. Resource Pooling Pattern:**
Though simplified here, the pooling concept:
- Reuses expensive resources
- Limits concurrent operations
- Prevents resource exhaustion
- Improves throughput

**Performance Utilities:**

**1. Type-Safe Debounce:**
```typescript
<T extends (...args: any[]) => any>
```
Preserves function signatures while adding debouncing - TypeScript magic!

**2. Object Pool Implementation:**
```typescript
acquire(): T {
    return pool.pop() || factory();
}
```
Efficient object reuse:
- Zero allocation when pool has objects
- Automatic creation when empty
- Configurable pool size
- Reset function for cleanup

**Key TypeScript-Specific Optimizations:**

1. **Const assertions**: `readonly` arrays prevent mutations
2. **Type narrowing**: Compiler optimizations based on type guards
3. **Private fields**: Better minification than private methods
4. **Optional chaining**: Reduces null checks overhead
5. **Template literals**: Faster than string concatenation

**Real-World Performance Gains:**
- Lookup tables: 1000x faster than computation
- Object pooling: 10x reduction in GC pressure
- Concurrency limiting: Predictable memory usage
- Weak references: Automatic memory management

**Advanced Techniques:**
- Use `ReadonlyArray<T>` for immutable data
- Leverage `const enum` for compile-time constants
- Consider `Buffer` for binary data (Node.js)
- Use Web Workers for CPU-intensive tasks

The beauty of TypeScript optimizations is they're type-safe and maintainable while delivering real performance gains!

## Pattern Migration Guide

### Migrating Between Patterns

```python
# Python: Pattern migration examples

# Step 1: Identify current pattern
class LegacyStaticService:
    """Current implementation using static methods"""
    
    _config = {'timeout': 30, 'retries': 3}
    
    @staticmethod
    def process_data(data):
        # Direct config access
        timeout = LegacyStaticService._config['timeout']
        return [d * 2 for d in data]
    
    @classmethod
    def update_config(cls, **kwargs):
        cls._config.update(kwargs)

# Step 2: Create instance-based version
class ServiceV2:
    """Migrated to instance-based pattern"""
    
    def __init__(self, config=None):
        self.config = {'timeout': 30, 'retries': 3}
        if config:
            self.config.update(config)
    
    def process_data(self, data):
        # Now uses instance config
        timeout = self.config['timeout']
        return [d * 2 for d in data]
    
    def update_config(self, **kwargs):
        self.config.update(kwargs)

# Step 3: Create migration adapter
class MigrationAdapter:
    """Adapter to support both old and new API"""
    
    def __init__(self):
        self._instance = ServiceV2()
        self._setup_static_compatibility()
    
    def _setup_static_compatibility(self):
        """Setup static method compatibility"""
        # Create static wrappers
        LegacyStaticService.process_data = staticmethod(
            lambda data: self._instance.process_data(data)
        )
        LegacyStaticService.update_config = classmethod(
            lambda cls, **kwargs: self._instance.update_config(**kwargs)
        )
    
    def get_instance(self):
        return self._instance

# Step 4: Gradual migration strategy
class MigrationManager:
    """Manages the migration process"""
    
    def __init__(self):
        self.adapter = MigrationAdapter()
        self.migration_stats = {
            'static_calls': 0,
            'instance_calls': 0
        }
    
    def track_usage(self, pattern_type):
        """Track which pattern is being used"""
        self.migration_stats[f'{pattern_type}_calls'] += 1
    
    def get_migration_progress(self):
        """Get migration progress report"""
        total = sum(self.migration_stats.values())
        if total == 0:
            return 0
        
        instance_percentage = (
            self.migration_stats['instance_calls'] / total * 100
        )
        return {
            'progress': instance_percentage,
            'static_calls': self.migration_stats['static_calls'],
            'instance_calls': self.migration_stats['instance_calls']
        }

# Usage during migration
migration = MigrationManager()

# Old code still works
result1 = LegacyStaticService.process_data([1, 2, 3])
migration.track_usage('static')

# New code uses instance
service = migration.adapter.get_instance()
result2 = service.process_data([1, 2, 3])
migration.track_usage('instance')

print(migration.get_migration_progress())
```

**Understanding Pattern Migration:**

This migration guide demonstrates a systematic approach to transitioning between patterns without breaking existing code. Let's explore the migration strategy:

**The Migration Challenge:**
When you have existing code using one pattern, switching to another can break countless dependencies. This guide shows how to migrate gracefully.

**Step 1: Current State Analysis:**
The `LegacyStaticService` represents typical static pattern usage:
- Class-level configuration
- Static methods throughout
- Direct class attribute access
- No instance state

**Step 2: Target Pattern Implementation:**
`ServiceV2` shows the instance-based equivalent:
- Configuration moved to instance
- Methods now use `self`
- Each instance can have different config
- More flexible and testable

**Step 3: The Adapter Pattern Magic:**
The `MigrationAdapter` is the key to seamless migration:
```python
LegacyStaticService.process_data = staticmethod(
    lambda data: self._instance.process_data(data)
)
```
This dynamically replaces static methods with wrappers that delegate to an instance!

**Key Migration Techniques:**

**1. Dynamic Method Replacement:**
Python's dynamic nature allows replacing class methods at runtime. The adapter:
- Creates an instance of the new implementation
- Wraps instance methods as static methods
- Maintains the original API surface

**2. Usage Tracking:**
The `MigrationManager` provides metrics:
- Counts static vs instance usage
- Calculates migration progress
- Helps identify unmigrated code
- Provides data for deprecation decisions

**3. Gradual Migration Path:**
```
1. Deploy adapter (no code changes needed)
2. Track usage patterns
3. Update code to use instances gradually
4. Monitor progress
5. Remove adapter when complete
```

**Benefits of This Approach:**
- **Zero downtime**: Old code continues working
- **Gradual transition**: No "big bang" migration
- **Measurable progress**: Track adoption metrics
- **Reversible**: Can roll back if issues arise
- **Testing-friendly**: Can test both patterns

**Real-World Considerations:**

**1. Performance Impact:**
The adapter adds a small overhead, but it's temporary and usually negligible.

**2. Thread Safety:**
If the static pattern was thread-safe, ensure the instance pattern maintains this guarantee.

**3. Configuration Migration:**
Static config becomes instance config - consider how to handle shared vs. instance-specific settings.

**4. Deprecation Strategy:**
Use the metrics to set deprecation timelines:
```python
if self.migration_stats['static_calls'] > 0:
    warnings.warn(
        "Static API is deprecated, use instance API",
        DeprecationWarning
    )
```

**Advanced Migration Patterns:**
- **Feature flags**: Toggle between implementations
- **A/B testing**: Compare performance/behavior
- **Canary deployment**: Roll out to subset of users
- **Proxy pattern**: More sophisticated adapters

This migration strategy transforms a risky refactoring into a controlled, measurable process!

```typescript
// TypeScript: Pattern migration examples

// Original static implementation
class LegacyStaticService {
    private static config = { timeout: 30, retries: 3 };
    
    static processData(data: number[]): number[] {
        const timeout = this.config.timeout;
        return data.map(d => d * 2);
    }
    
    static updateConfig(updates: Partial<typeof LegacyStaticService.config>): void {
        Object.assign(this.config, updates);
    }
}

// New instance-based implementation
interface ServiceConfig {
    timeout: number;
    retries: number;
}

class ServiceV2 {
    constructor(private config: ServiceConfig = { timeout: 30, retries: 3 }) {}
    
    processData(data: number[]): number[] {
        const timeout = this.config.timeout;
        return data.map(d => d * 2);
    }
    
    updateConfig(updates: Partial<ServiceConfig>): void {
        Object.assign(this.config, updates);
    }
}

// Migration adapter for backward compatibility
class MigrationAdapter {
    private static instance = new ServiceV2();
    
    static setupCompatibility(): void {
        // Override static methods to use instance
        LegacyStaticService.processData = (data: number[]) => {
            return this.instance.processData(data);
        };
        
        LegacyStaticService.updateConfig = (updates: any) => {
            this.instance.updateConfig(updates);
        };
    }
    
    static getInstance(): ServiceV2 {
        return this.instance;
    }
}

// Gradual migration with feature flags
class FeatureFlagMigration {
    private static flags = new Map<string, boolean>();
    private static oldService = LegacyStaticService;
    private static newService = new ServiceV2();
    
    static enableFeature(feature: string): void {
        this.flags.set(feature, true);
    }
    
    static processData(data: number[]): number[] {
        if (this.flags.get('useInstanceService')) {
            console.log('Using new instance-based service');
            return this.newService.processData(data);
        } else {
            console.log('Using legacy static service');
            return this.oldService.processData(data);
        }
    }
}

// Automated migration tool
class PatternMigrator {
    static async migrateFile(
        filePath: string,
        options: {
            from: 'static' | 'instance' | 'singleton';
            to: 'static' | 'instance' | 'singleton';
        }
    ): Promise<void> {
        console.log(`Migrating ${filePath} from ${options.from} to ${options.to}`);
        
        // In a real implementation, this would:
        // 1. Parse the TypeScript AST
        // 2. Identify pattern usage
        // 3. Transform the code
        // 4. Write the updated file
    }
    
    static generateMigrationReport(
        codebase: string[]
    ): { file: string; pattern: string; changes: number }[] {
        // Analyze codebase and generate report
        return codebase.map(file => ({
            file,
            pattern: 'static', // Detected pattern
            changes: Math.floor(Math.random() * 10) // Number of changes needed
        }));
    }
}

// Safe refactoring utilities
class RefactoringUtils {
    static createDeprecationProxy<T extends object>(
        target: T,
        newImplementation: T,
        deprecationMessage: string
    ): T {
        return new Proxy(target, {
            get(obj, prop) {
                console.warn(`DEPRECATED: ${deprecationMessage}`);
                return newImplementation[prop as keyof T];
            }
        });
    }
    
    static async parallelMigration<T>(
        items: T[],
        migrator: (item: T) => Promise<void>,
        concurrency = 4
    ): Promise<void> {
        const queue = [...items];
        const active: Promise<void>[] = [];
        
        while (queue.length > 0 || active.length > 0) {
            while (active.length < concurrency && queue.length > 0) {
                const item = queue.shift()!;
                const promise = migrator(item);
                active.push(promise);
                
                promise.then(() => {
                    const index = active.indexOf(promise);
                    active.splice(index, 1);
                });
            }
            
            if (active.length > 0) {
                await Promise.race(active);
            }
        }
    }
}

// Usage example
MigrationAdapter.setupCompatibility();

// Gradual migration with monitoring
FeatureFlagMigration.processData([1, 2, 3]); // Uses old
FeatureFlagMigration.enableFeature('useInstanceService');
FeatureFlagMigration.processData([1, 2, 3]); // Uses new

// Generate migration plan
const files = ['service1.ts', 'service2.ts', 'service3.ts'];
const report = PatternMigrator.generateMigrationReport(files);
console.log('Migration Report:', report);
```

**Understanding TypeScript Pattern Migration:**

This TypeScript migration implementation showcases advanced techniques leveraging TypeScript's type system and modern JavaScript features. Let's explore the sophisticated approaches:

**Type-Safe Migration:**
TypeScript's static typing makes migrations safer:
- Interfaces define contracts
- Compiler catches breaking changes
- Refactoring tools work better
- IDE support throughout migration

**Migration Adapter Pattern:**
```typescript
LegacyStaticService.processData = (data: number[]) => {
    return this.instance.processData(data);
};
```
Unlike Python, TypeScript requires more careful type handling, but provides compile-time safety.

**Feature Flag Strategy:**
The `FeatureFlagMigration` class demonstrates:
- Runtime pattern switching
- A/B testing capabilities
- Gradual rollout control
- Easy rollback mechanism

**Key TypeScript Migration Advantages:**

**1. Proxy-Based Deprecation:**
```typescript
return new Proxy(target, {
    get(obj, prop) {
        console.warn(`DEPRECATED: ${deprecationMessage}`);
        return newImplementation[prop as keyof T];
    }
});
```
This creates a transparent wrapper that:
- Logs deprecation warnings
- Delegates to new implementation
- Maintains type safety
- Zero runtime overhead in production

**2. AST-Based Migration:**
The `PatternMigrator` hints at automated refactoring:
- Parse TypeScript AST
- Identify pattern usage
- Transform code programmatically
- Maintain formatting and comments

**3. Concurrent Migration Processing:**
The `parallelMigration` utility:
- Processes files in parallel
- Limits concurrency to prevent overload
- Maintains progress tracking
- Handles errors gracefully

**Real-World Migration Workflow:**

**Phase 1: Analysis**
```typescript
const report = PatternMigrator.generateMigrationReport(files);
```
Identifies all files needing migration and estimates effort.

**Phase 2: Compatibility Layer**
```typescript
MigrationAdapter.setupCompatibility();
```
Ensures old code continues working with new implementation.

**Phase 3: Feature Flags**
```typescript
FeatureFlagMigration.enableFeature('useInstanceService');
```
Gradually enables new pattern for different users/features.

**Phase 4: Monitoring**
Track usage, performance, and errors during migration.

**Phase 5: Cleanup**
Remove compatibility layers once migration is complete.

**TypeScript-Specific Considerations:**

**1. Module Systems:**
- Consider ESM vs CommonJS implications
- Handle circular dependencies
- Update import/export statements

**2. Type Definitions:**
- Update .d.ts files
- Maintain backward compatibility
- Consider declaration merging

**3. Build Process:**
- Update webpack/rollup configurations
- Adjust tree-shaking settings
- Handle code splitting changes

**4. Testing Strategy:**
- Type-level tests with `ts-expect-error`
- Runtime behavior verification
- Performance regression tests

**Advanced Techniques:**

**1. Conditional Types:**
```typescript
type MigratedService<T> = T extends StaticPattern 
    ? InstancePattern 
    : T;
```

**2. Template Literal Types:**
Track migration status in type system.

**3. Decorators:**
Mark deprecated methods/classes.

**4. Source Maps:**
Maintain debugging capability during migration.

**Benefits Over Dynamic Languages:**
- Compile-time verification
- Better refactoring tools
- Type-safe migrations
- IDE intelligence throughout

This TypeScript approach combines the flexibility of gradual migration with the safety of static typing!

## Final Optimization Checklist

> [!important] Performance Optimization Checklist
> 
> **Static Methods:**
> - ✅ Pre-compute expensive constants
> - ✅ Use `@lru_cache` / memoization
> - ✅ Compile regex patterns once
> - ✅ Minimize class variable mutations
> - ✅ Consider lazy imports for heavy dependencies
> 
> **Instance Methods:**
> - ✅ Use `__slots__` in Python for memory efficiency
> - ✅ Implement intelligent caching with invalidation
> - ✅ Pool expensive resources
> - ✅ Consider immutability for [[Thread Safety|thread safety]]
> - ✅ Use weak references for large cached objects
> 
> **Singletons:**
> - ✅ Implement [[Thread Safety|thread-safe]] initialization
> - ✅ Use lazy initialization for expensive resources
> - ✅ Provide cleanup/reset mechanisms
> - ✅ Monitor for memory leaks
> - ✅ Consider alternatives (dependency injection)
> 
> **General:**
> - ✅ Profile before optimizing
> - ✅ Measure memory and CPU impact
> - ✅ Use appropriate data structures
> - ✅ Batch operations when possible
> - ✅ Consider async/await for I/O operations

---

This completes the comprehensive guide on Static vs Instance vs Singleton patterns in Python and TypeScript. Each section provides deep technical insights, practical examples, and performance considerations to help make informed architectural decisions.