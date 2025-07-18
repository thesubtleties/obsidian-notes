# Dependency Injection Containers

#programming #dependency-injection #architecture #patterns #ioc

## What is a Dependency Injection Container?

A **Dependency Injection (DI) Container** (also called an IoC Container - Inversion of Control) is a framework that manages object creation and dependency resolution automatically. Instead of manually creating objects and passing dependencies, you configure the container with rules about how to create objects, and it handles the rest.

## Core Concepts

### Inversion of Control (IoC)
Instead of your code controlling object creation:
```python
# Traditional approach - you control creation
database = Database('localhost', 5432)
cache = RedisCache('localhost', 6379)
service = UserService(database, cache)
```

The container controls it:
```python
# IoC approach - container controls creation
container = DIContainer()
service = container.get(UserService)  # Dependencies injected automatically
```

### Registration and Resolution
1. **Registration**: Tell the container how to create objects
2. **Resolution**: Ask the container for objects

## Simple Python DI Container

```python
from typing import Type, Dict, Any, Callable
import inspect

class DIContainer:
    """Simple dependency injection container"""
    
    def __init__(self):
        self._services: Dict[Type, Any] = {}
        self._singletons: Dict[Type, Any] = {}
        self._factories: Dict[Type, Callable] = {}
    
    def register(self, 
                 service_type: Type,
                 implementation: Type = None,
                 factory: Callable = None,
                 singleton: bool = False):
        """Register a service with the container"""
        
        if implementation:
            # Register class type
            self._services[service_type] = {
                'implementation': implementation,
                'singleton': singleton
            }
        elif factory:
            # Register factory function
            self._factories[service_type] = factory
        else:
            # Register concrete type as itself
            self._services[service_type] = {
                'implementation': service_type,
                'singleton': singleton
            }
    
    def get(self, service_type: Type):
        """Resolve and return a service instance"""
        
        # Check if singleton already exists
        if service_type in self._singletons:
            return self._singletons[service_type]
        
        # Check if factory exists
        if service_type in self._factories:
            instance = self._factories[service_type]()
            if self._is_singleton(service_type):
                self._singletons[service_type] = instance
            return instance
        
        # Check if service is registered
        if service_type in self._services:
            service_info = self._services[service_type]
            implementation = service_info['implementation']
            instance = self._create_instance(implementation)
            
            if service_info['singleton']:
                self._singletons[service_type] = instance
            
            return instance
        
        # Try to auto-resolve
        return self._create_instance(service_type)
    
    def _create_instance(self, cls: Type):
        """Create instance with dependency injection"""
        
        # Get constructor signature
        sig = inspect.signature(cls.__init__)
        kwargs = {}
        
        # Resolve each parameter
        for param_name, param in sig.parameters.items():
            if param_name == 'self':
                continue
                
            # Get type annotation
            param_type = param.annotation
            
            if param_type != inspect.Parameter.empty:
                # Recursively resolve dependency
                kwargs[param_name] = self.get(param_type)
            elif param.default != inspect.Parameter.empty:
                # Use default value
                kwargs[param_name] = param.default
            else:
                raise ValueError(
                    f"Cannot resolve parameter '{param_name}' for {cls.__name__}"
                )
        
        return cls(**kwargs)
    
    def _is_singleton(self, service_type: Type) -> bool:
        """Check if service is registered as singleton"""
        if service_type in self._services:
            return self._services[service_type].get('singleton', False)
        return False

# Example usage
from abc import ABC, abstractmethod

# Define interfaces
class IDatabase(ABC):
    @abstractmethod
    def query(self, sql: str): pass

class ICache(ABC):
    @abstractmethod
    def get(self, key: str): pass
    @abstractmethod
    def set(self, key: str, value: Any): pass

class ILogger(ABC):
    @abstractmethod
    def log(self, message: str): pass

# Implementations
class PostgresDatabase(IDatabase):
    def __init__(self, connection_string: str = "localhost:5432"):
        self.connection_string = connection_string
        print(f"Connecting to {connection_string}")
    
    def query(self, sql: str):
        print(f"Executing: {sql}")
        return []

class RedisCache(ICache):
    def __init__(self):
        self.cache = {}
        print("Redis cache initialized")
    
    def get(self, key: str):
        return self.cache.get(key)
    
    def set(self, key: str, value: Any):
        self.cache[key] = value

class ConsoleLogger(ILogger):
    def log(self, message: str):
        print(f"[LOG] {message}")

# Service that depends on interfaces
class UserService:
    def __init__(self, database: IDatabase, cache: ICache, logger: ILogger):
        self.database = database
        self.cache = cache
        self.logger = logger
        self.logger.log("UserService initialized")
    
    def get_user(self, user_id: int):
        # Check cache first
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            self.logger.log(f"User {user_id} found in cache")
            return cached
        
        # Query database
        self.logger.log(f"Fetching user {user_id} from database")
        users = self.database.query(f"SELECT * FROM users WHERE id = {user_id}")
        
        if users:
            self.cache.set(f"user:{user_id}", users[0])
            return users[0]
        
        return None

# Configure container
container = DIContainer()

# Register services
container.register(IDatabase, PostgresDatabase, singleton=True)
container.register(ICache, RedisCache, singleton=True)
container.register(ILogger, ConsoleLogger)
container.register(UserService)  # Auto-resolution of dependencies

# Use services
user_service = container.get(UserService)
user = user_service.get_user(123)

# Get same instances for singletons
db1 = container.get(IDatabase)
db2 = container.get(IDatabase)
print(db1 is db2)  # True - singleton

logger1 = container.get(ILogger)
logger2 = container.get(ILogger)
print(logger1 is logger2)  # False - not singleton
```

## Advanced TypeScript DI Container

```typescript
// Decorator-based DI Container
import 'reflect-metadata';

// Types
type ServiceIdentifier<T> = (new (...args: any[]) => T) | symbol | string;
type Factory<T> = () => T;

interface ServiceMetadata {
    factory?: Factory<any>;
    singleton?: boolean;
    dependencies?: ServiceIdentifier<any>[];
}

// Container implementation
class Container {
    private services = new Map<ServiceIdentifier<any>, ServiceMetadata>();
    private instances = new Map<ServiceIdentifier<any>, any>();
    
    register<T>(
        identifier: ServiceIdentifier<T>,
        implementation?: new (...args: any[]) => T,
        options: { singleton?: boolean } = {}
    ): void {
        const metadata: ServiceMetadata = {
            singleton: options.singleton ?? false
        };
        
        if (implementation) {
            // Get constructor parameters from metadata
            const paramTypes = Reflect.getMetadata('design:paramtypes', implementation) || [];
            metadata.dependencies = paramTypes;
            metadata.factory = () => this.createInstance(implementation, paramTypes);
        }
        
        this.services.set(identifier, metadata);
    }
    
    registerFactory<T>(
        identifier: ServiceIdentifier<T>,
        factory: Factory<T>,
        options: { singleton?: boolean } = {}
    ): void {
        this.services.set(identifier, {
            factory,
            singleton: options.singleton ?? false
        });
    }
    
    get<T>(identifier: ServiceIdentifier<T>): T {
        // Check for existing singleton instance
        if (this.instances.has(identifier)) {
            return this.instances.get(identifier);
        }
        
        const metadata = this.services.get(identifier);
        if (!metadata) {
            throw new Error(`Service ${String(identifier)} not registered`);
        }
        
        let instance: T;
        
        if (metadata.factory) {
            instance = metadata.factory();
        } else if (typeof identifier === 'function') {
            // Auto-resolve
            const paramTypes = Reflect.getMetadata('design:paramtypes', identifier) || [];
            instance = this.createInstance(identifier as any, paramTypes);
        } else {
            throw new Error(`Cannot resolve ${String(identifier)}`);
        }
        
        if (metadata.singleton) {
            this.instances.set(identifier, instance);
        }
        
        return instance;
    }
    
    private createInstance<T>(
        ctor: new (...args: any[]) => T,
        paramTypes: any[]
    ): T {
        const dependencies = paramTypes.map(type => this.get(type));
        return new ctor(...dependencies);
    }
}

// Decorators
function Injectable(target: any) {
    // This decorator is used to mark a class as injectable
    // The actual work is done by TypeScript's emitDecoratorMetadata
}

function Inject(token: ServiceIdentifier<any>) {
    return function (target: any, propertyKey: string | symbol, parameterIndex: number) {
        const existingTokens = Reflect.getMetadata('custom:inject-tokens', target) || {};
        existingTokens[parameterIndex] = token;
        Reflect.defineMetadata('custom:inject-tokens', existingTokens, target);
    };
}

// Example usage with decorators
// Interfaces
interface IEmailService {
    send(to: string, subject: string, body: string): Promise<void>;
}

interface IUserRepository {
    findById(id: string): Promise<User | null>;
    save(user: User): Promise<void>;
}

// Tokens for interfaces
const EMAIL_SERVICE = Symbol('EmailService');
const USER_REPOSITORY = Symbol('UserRepository');

// Implementations
@Injectable
class EmailService implements IEmailService {
    async send(to: string, subject: string, body: string): Promise<void> {
        console.log(`Sending email to ${to}: ${subject}`);
    }
}

@Injectable
class UserRepository implements IUserRepository {
    private users = new Map<string, User>();
    
    async findById(id: string): Promise<User | null> {
        return this.users.get(id) || null;
    }
    
    async save(user: User): Promise<void> {
        this.users.set(user.id, user);
    }
}

@Injectable
class UserService {
    constructor(
        @Inject(USER_REPOSITORY) private userRepo: IUserRepository,
        @Inject(EMAIL_SERVICE) private emailService: IEmailService
    ) {}
    
    async createUser(data: CreateUserData): Promise<User> {
        const user = new User(data);
        await this.userRepo.save(user);
        
        await this.emailService.send(
            user.email,
            'Welcome!',
            'Thanks for signing up!'
        );
        
        return user;
    }
}

// Configure container
const container = new Container();

container.register(EMAIL_SERVICE, EmailService, { singleton: true });
container.register(USER_REPOSITORY, UserRepository, { singleton: true });
container.register(UserService);

// Usage
const userService = container.get(UserService);
const user = await userService.createUser({
    name: 'John Doe',
    email: 'john@example.com'
});
```

## Popular DI Containers

### Python
- **dependency-injector**: Full-featured with decorators
- **injector**: Google Guice-inspired
- **punq**: Lightweight and simple

### JavaScript/TypeScript
- **InversifyJS**: Powerful with decorators
- **tsyringe**: Microsoft's lightweight container
- **Angular DI**: Built into Angular framework
- **NestJS DI**: Built into NestJS framework

## Benefits of DI Containers

> [!info] Advantages
> 1. **Automatic dependency resolution** - No manual wiring
> 2. **Centralized configuration** - All bindings in one place
> 3. **Lifetime management** - Singleton, transient, scoped
> 4. **Easier testing** - Swap implementations easily
> 5. **Reduced coupling** - Depend on interfaces, not implementations
> 6. **Better organization** - Clear separation of concerns

## Common Patterns

### 1. **Interface Segregation**
```python
# Define small, focused interfaces
class IReader(ABC):
    @abstractmethod
    def read(self, key: str): pass

class IWriter(ABC):
    @abstractmethod
    def write(self, key: str, value: Any): pass

# Service can depend on only what it needs
class ReadOnlyService:
    def __init__(self, reader: IReader):
        self.reader = reader
```

### 2. **Factory Registration**
```python
# Register factories for complex creation
container.register(
    DatabaseConnection,
    factory=lambda: DatabaseConnection(
        host=os.getenv('DB_HOST'),
        port=int(os.getenv('DB_PORT', 5432))
    ),
    singleton=True
)
```

### 3. **Scoped Lifetime**
```python
# Per-request scope in web applications
class ScopedContainer:
    def __init__(self, parent: DIContainer):
        self.parent = parent
        self.scoped_instances = {}
    
    def get_scoped(self, service_type):
        if service_type not in self.scoped_instances:
            self.scoped_instances[service_type] = self.parent.get(service_type)
        return self.scoped_instances[service_type]
```

## When to Use DI Containers

> [!tip] Use DI Containers When
> - Large applications with many dependencies
> - Need to swap implementations (testing, different environments)
> - Want centralized dependency configuration
> - Complex object graphs with deep dependencies
> - Need lifetime management (singleton, scoped, transient)

> [!warning] Avoid DI Containers When
> - Small applications with few dependencies
> - Simple scripts or utilities
> - Performance is critical (adds overhead)
> - Team is not familiar with DI concepts