# Service Locator Pattern

#programming #patterns #service-locator #anti-pattern #dependency-injection

## What is the Service Locator Pattern?

The **Service Locator Pattern** is a design pattern where a central registry provides access to services throughout an application. Instead of injecting dependencies directly, components request services from the locator when needed.

> [!warning] Often Considered an Anti-Pattern
> Service Locator is controversial - many consider it an anti-pattern because it hides dependencies and makes testing harder. However, it can be useful in specific scenarios.

## Why It's Controversial

Think of Service Locator like a global phone book for your application. While convenient, it creates several problems:

1. **Hidden Dependencies** - You can't tell what a class needs by looking at its constructor
2. **Runtime Errors** - Missing services only fail when requested, not at construction
3. **Harder Testing** - Must configure the entire locator for each test
4. **Global State** - The locator itself becomes a form of global state

## Basic Implementation

### Python Example
```python
from typing import Type, Dict, Any, TypeVar, Generic, Optional
from abc import ABC, abstractmethod

T = TypeVar('T')

class ServiceLocator:
    """Central service registry"""
    
    _instance: Optional['ServiceLocator'] = None
    
    def __init__(self):
        self._services: Dict[Type, Any] = {}
        self._factories: Dict[Type, callable] = {}
    
    @classmethod
    def get_instance(cls) -> 'ServiceLocator':
        """Get singleton instance"""
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance
    
    def register(self, service_type: Type[T], instance: T) -> None:
        """Register a service instance"""
        self._services[service_type] = instance
    
    def register_factory(self, service_type: Type[T], factory: callable) -> None:
        """Register a factory function"""
        self._factories[service_type] = factory
    
    def get(self, service_type: Type[T]) -> T:
        """Get service by type"""
        # Check registered instances
        if service_type in self._services:
            return self._services[service_type]
        
        # Check factories
        if service_type in self._factories:
            instance = self._factories[service_type]()
            self._services[service_type] = instance
            return instance
        
        raise ValueError(f"Service {service_type.__name__} not registered")
    
    def clear(self) -> None:
        """Clear all services (useful for testing)"""
        self._services.clear()
        self._factories.clear()

# Define service interfaces
class IDatabase(ABC):
    @abstractmethod
    def query(self, sql: str) -> list:
        pass

class IEmailService(ABC):
    @abstractmethod
    def send(self, to: str, subject: str, body: str) -> None:
        pass

# Implementations
class PostgresDatabase(IDatabase):
    def query(self, sql: str) -> list:
        print(f"PostgreSQL: {sql}")
        return []

class SmtpEmailService(IEmailService):
    def send(self, to: str, subject: str, body: str) -> None:
        print(f"Sending email to {to}: {subject}")

# Using Service Locator (problematic approach)
class UserService:
    """Service with hidden dependencies"""
    
    def __init__(self):
        # Dependencies are hidden!
        self._locator = ServiceLocator.get_instance()
    
    def create_user(self, email: str, name: str):
        # Get dependencies at runtime
        database = self._locator.get(IDatabase)
        email_service = self._locator.get(IEmailService)
        
        # Use services
        database.query(f"INSERT INTO users (email, name) VALUES ('{email}', '{name}')")
        email_service.send(email, "Welcome!", f"Hello {name}")

# Setup
locator = ServiceLocator.get_instance()
locator.register(IDatabase, PostgresDatabase())
locator.register(IEmailService, SmtpEmailService())

# Usage - no dependencies visible!
user_service = UserService()
user_service.create_user("user@example.com", "John")
```

**What's Wrong Here?**
- 🚫 Can't tell UserService needs IDatabase and IEmailService
- 🚫 If services aren't registered, fails at runtime
- 🚫 Testing requires setting up the entire locator

### TypeScript Example
```typescript
// Service locator implementation
class ServiceLocator {
    private static instance: ServiceLocator;
    private services: Map<string | symbol, any> = new Map();
    private factories: Map<string | symbol, () => any> = new Map();
    
    private constructor() {}
    
    static getInstance(): ServiceLocator {
        if (!ServiceLocator.instance) {
            ServiceLocator.instance = new ServiceLocator();
        }
        return ServiceLocator.instance;
    }
    
    register<T>(token: string | symbol, instance: T): void {
        this.services.set(token, instance);
    }
    
    registerFactory<T>(token: string | symbol, factory: () => T): void {
        this.factories.set(token, factory);
    }
    
    get<T>(token: string | symbol): T {
        if (this.services.has(token)) {
            return this.services.get(token);
        }
        
        if (this.factories.has(token)) {
            const instance = this.factories.get(token)!();
            this.services.set(token, instance);
            return instance;
        }
        
        throw new Error(`Service ${String(token)} not registered`);
    }
    
    clear(): void {
        this.services.clear();
        this.factories.clear();
    }
}

// Service tokens
const DATABASE_TOKEN = Symbol('Database');
const EMAIL_TOKEN = Symbol('EmailService');
const LOGGER_TOKEN = Symbol('Logger');

// Interfaces
interface Database {
    query(sql: string): Promise<any[]>;
}

interface EmailService {
    send(to: string, subject: string, body: string): Promise<void>;
}

interface Logger {
    log(level: string, message: string): void;
}

// Using service locator in a class
class OrderService {
    private locator = ServiceLocator.getInstance();
    
    async processOrder(orderId: string): Promise<void> {
        // Hidden dependencies fetched at runtime
        const db = this.locator.get<Database>(DATABASE_TOKEN);
        const email = this.locator.get<EmailService>(EMAIL_TOKEN);
        const logger = this.locator.get<Logger>(LOGGER_TOKEN);
        
        logger.log('info', `Processing order ${orderId}`);
        
        const order = await db.query(`SELECT * FROM orders WHERE id = '${orderId}'`);
        await email.send(order[0].email, 'Order Confirmed', 'Your order is processing');
    }
}
```

## When Service Locator Might Be Acceptable

### 1. **Legacy System Integration**
```python
# When refactoring legacy code that can't be fully converted
class LegacyServiceAdapter:
    """Adapter for legacy systems that expect global access"""
    
    def __init__(self):
        # Legacy code expects to pull dependencies
        self._locator = ServiceLocator.get_instance()
    
    def legacy_operation(self):
        # Gradual migration path
        old_service = self._locator.get(OldServiceType)
        return old_service.do_legacy_thing()
```

### 2. **Plugin Systems**
```python
class PluginManager:
    """Plugin system where available services are dynamic"""
    
    def __init__(self):
        self._locator = ServiceLocator.get_instance()
        self._plugins = {}
    
    def load_plugin(self, plugin_name: str):
        """Dynamically load plugin that may use various services"""
        plugin_module = import_module(f"plugins.{plugin_name}")
        
        # Plugin discovers available services
        available_services = []
        for service_type in [IDatabase, ICache, ILogger]:
            try:
                service = self._locator.get(service_type)
                available_services.append(service_type)
            except ValueError:
                pass
        
        # Initialize plugin with available services
        plugin = plugin_module.create_plugin(available_services)
        self._plugins[plugin_name] = plugin
```

### 3. **Framework Internals**
```typescript
// Frameworks often use service locator internally
class FrameworkCore {
    private serviceLocator: ServiceLocator;
    
    constructor() {
        this.serviceLocator = new ServiceLocator();
        this.registerCoreServices();
    }
    
    private registerCoreServices(): void {
        // Framework manages its own services
        this.serviceLocator.register('router', new Router());
        this.serviceLocator.register('view', new ViewEngine());
        this.serviceLocator.register('orm', new ORM());
    }
    
    // Expose for extensions
    getService<T>(name: string): T {
        return this.serviceLocator.get<T>(name);
    }
}
```

## Better Alternatives

### 1. **Dependency Injection**
```python
# Better: Explicit dependencies
class UserService:
    def __init__(self, database: IDatabase, email_service: IEmailService):
        self.database = database
        self.email_service = email_service
    
    def create_user(self, email: str, name: str):
        # Same functionality, but dependencies are explicit
        self.database.query(f"INSERT INTO users (email, name) VALUES ('{email}', '{name}')")
        self.email_service.send(email, "Welcome!", f"Hello {name}")

# Clear what's needed
database = PostgresDatabase()
email_service = SmtpEmailService()
user_service = UserService(database, email_service)
```

### 2. **Factory Pattern**
```python
from typing import Protocol

class ServiceFactory:
    """Factory that creates configured services"""
    
    def __init__(self, config: dict):
        self.config = config
    
    def create_user_service(self) -> UserService:
        """Factory method shows dependencies clearly"""
        database = self._create_database()
        email_service = self._create_email_service()
        return UserService(database, email_service)
    
    def _create_database(self) -> IDatabase:
        db_type = self.config.get('database_type', 'postgres')
        if db_type == 'postgres':
            return PostgresDatabase(self.config.get('db_connection'))
        elif db_type == 'mysql':
            return MySQLDatabase(self.config.get('db_connection'))
        else:
            raise ValueError(f"Unknown database type: {db_type}")
    
    def _create_email_service(self) -> IEmailService:
        return SmtpEmailService(self.config.get('smtp_server'))
```

## Mitigating Service Locator Problems

If you must use Service Locator, here's how to make it less problematic:

### 1. **Scoped Locators**
```python
class ScopedServiceLocator:
    """Locator with scoped lifetime management"""
    
    def __init__(self, parent: Optional['ScopedServiceLocator'] = None):
        self.parent = parent
        self._services = {}
        self._scoped_services = {}
    
    def create_scope(self) -> 'ScopedServiceLocator':
        """Create child scope"""
        return ScopedServiceLocator(parent=self)
    
    def register_scoped(self, service_type: Type[T], factory: callable) -> None:
        """Register service that lives only in this scope"""
        self._scoped_services[service_type] = factory
    
    def get(self, service_type: Type[T]) -> T:
        # Check this scope first
        if service_type in self._services:
            return self._services[service_type]
        
        # Create from scoped factory
        if service_type in self._scoped_services:
            instance = self._scoped_services[service_type]()
            self._services[service_type] = instance
            return instance
        
        # Check parent scope
        if self.parent:
            return self.parent.get(service_type)
        
        raise ValueError(f"Service {service_type.__name__} not found")
```

### 2. **Service Validation**
```python
class ValidatingServiceLocator(ServiceLocator):
    """Locator that validates dependencies at startup"""
    
    def __init__(self):
        super().__init__()
        self._required_services: Set[Type] = set()
    
    def require(self, service_type: Type) -> None:
        """Mark service as required"""
        self._required_services.add(service_type)
    
    def validate(self) -> None:
        """Validate all required services are available"""
        missing = []
        for service_type in self._required_services:
            try:
                self.get(service_type)
            except ValueError:
                missing.append(service_type.__name__)
        
        if missing:
            raise RuntimeError(f"Missing required services: {', '.join(missing)}")
    
    def create_validated(self, cls: Type[T]) -> T:
        """Create instance after validation"""
        self.validate()
        return cls()
```

### 3. **Typed Service Locator**
```typescript
// Type-safe service locator
interface ServiceMap {
    database: Database;
    emailService: EmailService;
    logger: Logger;
    cache: CacheService;
}

class TypedServiceLocator {
    private services: Partial<ServiceMap> = {};
    
    register<K extends keyof ServiceMap>(
        key: K, 
        service: ServiceMap[K]
    ): void {
        this.services[key] = service;
    }
    
    get<K extends keyof ServiceMap>(key: K): ServiceMap[K] {
        const service = this.services[key];
        if (!service) {
            throw new Error(`Service '${key}' not registered`);
        }
        return service;
    }
    
    // Type-safe service requirements
    require<K extends keyof ServiceMap>(...keys: K[]): void {
        const missing = keys.filter(key => !this.services[key]);
        if (missing.length > 0) {
            throw new Error(`Missing services: ${missing.join(', ')}`);
        }
    }
}
```

## Testing with Service Locator

### The Challenge
```python
# Testing with service locator is painful
class TestUserService(unittest.TestCase):
    def setUp(self):
        # Must setup entire locator
        self.locator = ServiceLocator.get_instance()
        self.locator.clear()
        
        # Register all dependencies
        self.mock_db = Mock(spec=IDatabase)
        self.mock_email = Mock(spec=IEmailService)
        self.locator.register(IDatabase, self.mock_db)
        self.locator.register(IEmailService, self.mock_email)
    
    def tearDown(self):
        # Clean up global state
        self.locator.clear()
    
    def test_create_user(self):
        # Test is coupled to service locator
        service = UserService()
        service.create_user("test@example.com", "Test User")
        
        self.mock_db.query.assert_called_once()
        self.mock_email.send.assert_called_once()
```

### Better: Dependency Injection Testing
```python
# Testing with DI is cleaner
class TestUserServiceDI(unittest.TestCase):
    def test_create_user(self):
        # Direct dependency injection
        mock_db = Mock(spec=IDatabase)
        mock_email = Mock(spec=IEmailService)
        
        service = UserService(mock_db, mock_email)
        service.create_user("test@example.com", "Test User")
        
        mock_db.query.assert_called_once()
        mock_email.send.assert_called_once()
```

## Key Takeaways

> [!important] Service Locator Pattern
> 1. **Hides dependencies** - Can't see what a class needs from its interface
> 2. **Runtime failures** - Missing services only detected when requested
> 3. **Harder testing** - Requires setting up global state
> 4. **Consider alternatives** - Dependency injection is usually better
> 5. **Limited use cases** - Plugins, legacy code, framework internals
> 6. **If you must use it** - Add validation, scoping, and type safety

## When to Use Each Pattern

| Pattern | Use When |
|---------|----------|
| **Dependency Injection** | You know dependencies at compile time (most cases) |
| **Service Locator** | Dependencies are truly dynamic (plugins, extensions) |
| **Factory Pattern** | You need complex creation logic but known types |
| **Registry Pattern** | You need global registration but not hidden dependencies |