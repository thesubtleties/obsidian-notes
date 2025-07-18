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
> - ✅ Consider immutability for thread safety
> - ✅ Use weak references for large cached objects
> 
> **Singletons:**
> - ✅ Implement thread-safe initialization
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