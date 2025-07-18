# Part 3: Dependency Injection & Testing

#programming #python #typescript #testing #dependency-injection #design-patterns

[[Static vs Instance vs Singleton Patterns - Python & TypeScript Deep Dive|← Back to Main Guide]] | [[Static vs Instance vs Singleton - Part 2 - Implementation Deep Dive|← Part 2: Implementation]]

## Table of Contents

1. [[#Dependency Injection Fundamentals]]
2. [[#DI with Static Methods]]
3. [[#DI with Instance Methods]]
4. [[#DI with Singletons]]
5. [[#Testing Strategies]]
6. [[#Mocking and Stubbing]]
7. [[#Framework-Specific DI]]
8. [[#Advanced Testing Patterns]]

## Dependency Injection Fundamentals

### What is Dependency Injection?

Dependency Injection (DI) is a design pattern where objects receive their dependencies from external sources rather than creating them internally. This promotes:
- **Testability**: Easy to substitute mock objects
- **Flexibility**: Change implementations without modifying code
- **Decoupling**: Reduced dependencies between components

### DI Principles Comparison

| Principle | Python Approach | TypeScript Approach |
|-----------|----------------|-------------------|
| **Constructor Injection** | `__init__` parameters | Constructor parameters |
| **Property Injection** | Direct assignment | Property decorators |
| **Method Injection** | Function parameters | Method parameters |
| **Interface Segregation** | Duck typing / Protocols | Interfaces |
| **IoC Containers** | Manual / Libraries | Built-in (Angular) / Libraries |

## DI with Static Methods

### The Challenge with Static Methods

Static methods present unique challenges for dependency injection:
- No instance to inject into
- Global state issues
- Difficult to mock

### Python Static Method DI Patterns

```python
# Pattern 1: Parameter Injection
class EmailService:
    @staticmethod
    def send_email(recipient: str, subject: str, body: str, 
                   smtp_client=None):
        """Accept dependencies as parameters"""
        client = smtp_client or DefaultSMTPClient()
        return client.send(recipient, subject, body)

# Testing
def test_email_service():
    mock_client = Mock()
    EmailService.send_email("test@example.com", "Test", "Body", 
                          smtp_client=mock_client)
    mock_client.send.assert_called_once()

# Pattern 2: Configuration Injection
class ConfigurableService:
    _config = None
    
    @classmethod
    def configure(cls, config):
        cls._config = config
    
    @staticmethod
    def process_data(data):
        config = ConfigurableService._config or DefaultConfig()
        return data * config.multiplier

# Pattern 3: Factory Pattern
class ServiceFactory:
    @staticmethod
    def create_processor(dependencies=None):
        deps = dependencies or {}
        return DataProcessor(
            validator=deps.get('validator', DefaultValidator()),
            transformer=deps.get('transformer', DefaultTransformer())
        )
```

### TypeScript Static Method DI Patterns

```typescript
// Pattern 1: Parameter Injection with Interfaces
interface EmailClient {
    send(recipient: string, subject: string, body: string): Promise<void>;
}

class EmailService {
    static async sendEmail(
        recipient: string, 
        subject: string, 
        body: string,
        client?: EmailClient
    ): Promise<void> {
        const emailClient = client || new DefaultEmailClient();
        await emailClient.send(recipient, subject, body);
    }
}

// Testing
describe('EmailService', () => {
    it('should send email', async () => {
        const mockClient: EmailClient = {
            send: jest.fn().mockResolvedValue(undefined)
        };
        
        await EmailService.sendEmail('test@example.com', 'Test', 'Body', mockClient);
        expect(mockClient.send).toHaveBeenCalledWith('test@example.com', 'Test', 'Body');
    });
});

// Pattern 2: Static Configuration
interface ServiceConfig {
    apiUrl: string;
    timeout: number;
}

class ConfigurableService {
    private static config: ServiceConfig;
    
    static configure(config: ServiceConfig): void {
        this.config = config;
    }
    
    static async fetchData(endpoint: string): Promise<any> {
        const config = this.config || { apiUrl: 'default', timeout: 5000 };
        // Use config...
    }
}

// Pattern 3: Ambient Context Pattern
namespace DependencyContext {
    let emailClient: EmailClient | null = null;
    
    export function setEmailClient(client: EmailClient): void {
        emailClient = client;
    }
    
    export function getEmailClient(): EmailClient {
        return emailClient || new DefaultEmailClient();
    }
}

class ContextualEmailService {
    static async sendEmail(recipient: string, subject: string, body: string): Promise<void> {
        const client = DependencyContext.getEmailClient();
        await client.send(recipient, subject, body);
    }
}
```

## DI with Instance Methods

### Python Instance DI

```python
# Constructor Injection
class OrderService:
    def __init__(self, 
                 payment_processor,
                 inventory_service,
                 email_service,
                 logger=None):
        self.payment_processor = payment_processor
        self.inventory_service = inventory_service
        self.email_service = email_service
        self.logger = logger or logging.getLogger(__name__)
    
    def process_order(self, order):
        try:
            # Check inventory
            if not self.inventory_service.check_availability(order.items):
                raise OutOfStockError()
            
            # Process payment
            payment_result = self.payment_processor.charge(
                order.customer, 
                order.total
            )
            
            # Update inventory
            self.inventory_service.reserve_items(order.items)
            
            # Send confirmation
            self.email_service.send_confirmation(order)
            
            self.logger.info(f"Order {order.id} processed successfully")
            return payment_result
            
        except Exception as e:
            self.logger.error(f"Order processing failed: {e}")
            raise

# Property Injection with validation
class ServiceContainer:
    def __init__(self):
        self._database = None
        self._cache = None
    
    @property
    def database(self):
        if not self._database:
            raise ValueError("Database not configured")
        return self._database
    
    @database.setter
    def database(self, db):
        if not hasattr(db, 'query'):
            raise TypeError("Database must have query method")
        self._database = db
    
    @property
    def cache(self):
        return self._cache or NoOpCache()
    
    @cache.setter
    def cache(self, cache):
        self._cache = cache

# Method Injection
class DataProcessor:
    def process_with_custom_validator(self, data, validator=None):
        val = validator or DefaultValidator()
        if not val.validate(data):
            raise ValidationError()
        return self._transform(data)
```

### TypeScript Instance DI

```typescript
// Constructor Injection with Interfaces
interface PaymentProcessor {
    charge(customer: Customer, amount: number): Promise<PaymentResult>;
}

interface InventoryService {
    checkAvailability(items: Item[]): Promise<boolean>;
    reserveItems(items: Item[]): Promise<void>;
}

interface EmailService {
    sendConfirmation(order: Order): Promise<void>;
}

interface Logger {
    info(message: string): void;
    error(message: string, error?: Error): void;
}

class OrderService {
    constructor(
        private readonly paymentProcessor: PaymentProcessor,
        private readonly inventoryService: InventoryService,
        private readonly emailService: EmailService,
        private readonly logger: Logger
    ) {}
    
    async processOrder(order: Order): Promise<PaymentResult> {
        try {
            // Check inventory
            const available = await this.inventoryService.checkAvailability(order.items);
            if (!available) {
                throw new OutOfStockError();
            }
            
            // Process payment
            const paymentResult = await this.paymentProcessor.charge(
                order.customer,
                order.total
            );
            
            // Update inventory
            await this.inventoryService.reserveItems(order.items);
            
            // Send confirmation
            await this.emailService.sendConfirmation(order);
            
            this.logger.info(`Order ${order.id} processed successfully`);
            return paymentResult;
            
        } catch (error) {
            this.logger.error(`Order processing failed`, error);
            throw error;
        }
    }
}

// Property Injection with Decorators
function Inject(token: string) {
    return function (target: any, propertyKey: string) {
        // Metadata for DI container
        Reflect.defineMetadata('inject', token, target, propertyKey);
    };
}

class UserService {
    @Inject('DATABASE')
    private database!: Database;
    
    @Inject('CACHE')
    private cache!: Cache;
    
    async getUser(id: string): Promise<User> {
        const cached = await this.cache.get(`user:${id}`);
        if (cached) return cached;
        
        const user = await this.database.query('SELECT * FROM users WHERE id = ?', [id]);
        await this.cache.set(`user:${id}`, user);
        return user;
    }
}
```

## DI with Singletons

### Python Singleton DI

```python
# Configurable Singleton
class ConfigurableSingleton:
    _instance = None
    _dependencies = {}
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialize_with_dependencies()
        return cls._instance
    
    @classmethod
    def configure(cls, **dependencies):
        """Configure dependencies before first use"""
        cls._dependencies.update(dependencies)
    
    def _initialize_with_dependencies(self):
        self.database = self._dependencies.get('database', DefaultDatabase())
        self.cache = self._dependencies.get('cache', DefaultCache())
        self.logger = self._dependencies.get('logger', logging.getLogger())

# Singleton with Dependency Injection Container
class DIContainer:
    _services = {}
    _singletons = {}
    
    @classmethod
    def register(cls, name, factory, singleton=False):
        """Register a service factory"""
        cls._services[name] = {
            'factory': factory,
            'singleton': singleton
        }
    
    @classmethod
    def get(cls, name):
        """Get a service instance"""
        if name not in cls._services:
            raise KeyError(f"Service '{name}' not registered")
        
        service_config = cls._services[name]
        
        if service_config['singleton']:
            if name not in cls._singletons:
                cls._singletons[name] = service_config['factory']()
            return cls._singletons[name]
        
        return service_config['factory']()
    
    @classmethod
    def reset(cls):
        """Reset container for testing"""
        cls._services.clear()
        cls._singletons.clear()

# Usage
DIContainer.register('database', lambda: PostgresDatabase(), singleton=True)
DIContainer.register('email', lambda: EmailService(), singleton=False)

# In application
db = DIContainer.get('database')  # Returns singleton
email1 = DIContainer.get('email')  # New instance
email2 = DIContainer.get('email')  # New instance
```

### TypeScript Singleton DI

```typescript
// Singleton with Constructor Dependencies
class DatabaseConnection {
    private static instance: DatabaseConnection;
    
    private constructor(
        private readonly config: DatabaseConfig,
        private readonly logger: Logger
    ) {
        this.logger.info('Database connection created');
    }
    
    static initialize(config: DatabaseConfig, logger: Logger): void {
        if (this.instance) {
            throw new Error('Database already initialized');
        }
        this.instance = new DatabaseConnection(config, logger);
    }
    
    static getInstance(): DatabaseConnection {
        if (!this.instance) {
            throw new Error('Database not initialized. Call initialize() first.');
        }
        return this.instance;
    }
    
    async query(sql: string): Promise<any> {
        this.logger.debug(`Executing query: ${sql}`);
        // Execute query...
    }
}

// Singleton Registry Pattern
class ServiceRegistry {
    private static instances = new Map<string, any>();
    private static factories = new Map<string, () => any>();
    
    static registerSingleton<T>(token: string, factory: () => T): void {
        this.factories.set(token, factory);
    }
    
    static get<T>(token: string): T {
        if (!this.instances.has(token)) {
            const factory = this.factories.get(token);
            if (!factory) {
                throw new Error(`No factory registered for ${token}`);
            }
            this.instances.set(token, factory());
        }
        return this.instances.get(token);
    }
    
    static reset(): void {
        this.instances.clear();
    }
    
    static resetToken(token: string): void {
        this.instances.delete(token);
    }
}

// Usage
ServiceRegistry.registerSingleton('logger', () => new ConsoleLogger());
ServiceRegistry.registerSingleton('database', () => {
    const logger = ServiceRegistry.get<Logger>('logger');
    return new Database(logger);
});
```

## Testing Strategies

### Testing Static Methods

```python
# Python - Testing Static Methods
import unittest
from unittest.mock import Mock, patch

class TestStaticMethods(unittest.TestCase):
    def test_with_dependency_injection(self):
        """Test by injecting mock dependencies"""
        mock_validator = Mock()
        mock_validator.validate.return_value = True
        
        result = DataValidator.validate_data(
            {"test": "data"},
            validator=mock_validator
        )
        
        mock_validator.validate.assert_called_once()
        self.assertTrue(result)
    
    @patch('mymodule.external_service')
    def test_with_patching(self, mock_service):
        """Test by patching external dependencies"""
        mock_service.fetch.return_value = {"status": "ok"}
        
        result = DataFetcher.fetch_and_process()
        
        mock_service.fetch.assert_called_once()
        self.assertEqual(result["status"], "ok")
    
    def test_with_configuration(self):
        """Test by configuring static state"""
        # Save original config
        original_config = ServiceConfig.get_config()
        
        try:
            # Set test config
            ServiceConfig.set_config({"timeout": 1})
            
            result = ServiceClient.make_request()
            self.assertIsNotNone(result)
            
        finally:
            # Restore original config
            ServiceConfig.set_config(original_config)
```

```typescript
// TypeScript - Testing Static Methods
describe('Static Method Testing', () => {
    describe('DataValidator', () => {
        it('should validate with injected validator', () => {
            const mockValidator = {
                validate: jest.fn().mockReturnValue(true)
            };
            
            const result = DataValidator.validateData(
                { test: 'data' },
                mockValidator
            );
            
            expect(mockValidator.validate).toHaveBeenCalledWith({ test: 'data' });
            expect(result).toBe(true);
        });
    });
    
    describe('ServiceClient', () => {
        let originalFetch: typeof global.fetch;
        
        beforeEach(() => {
            originalFetch = global.fetch;
            global.fetch = jest.fn();
        });
        
        afterEach(() => {
            global.fetch = originalFetch;
        });
        
        it('should handle fetch correctly', async () => {
            (global.fetch as jest.Mock).mockResolvedValue({
                ok: true,
                json: async () => ({ data: 'test' })
            });
            
            const result = await ServiceClient.fetchData();
            
            expect(global.fetch).toHaveBeenCalled();
            expect(result).toEqual({ data: 'test' });
        });
    });
});
```

### Testing Instance Methods

```python
# Python - Testing Instance Methods
class TestOrderService(unittest.TestCase):
    def setUp(self):
        """Create mocks for all dependencies"""
        self.mock_payment = Mock()
        self.mock_inventory = Mock()
        self.mock_email = Mock()
        self.mock_logger = Mock()
        
        self.service = OrderService(
            payment_processor=self.mock_payment,
            inventory_service=self.mock_inventory,
            email_service=self.mock_email,
            logger=self.mock_logger
        )
    
    def test_successful_order(self):
        """Test successful order processing"""
        # Arrange
        order = Order(id="123", items=["item1"], total=100)
        self.mock_inventory.check_availability.return_value = True
        self.mock_payment.charge.return_value = PaymentResult(success=True)
        
        # Act
        result = self.service.process_order(order)
        
        # Assert
        self.assertTrue(result.success)
        self.mock_inventory.check_availability.assert_called_once_with(["item1"])
        self.mock_payment.charge.assert_called_once()
        self.mock_email.send_confirmation.assert_called_once_with(order)
        self.mock_logger.info.assert_called()
    
    def test_out_of_stock(self):
        """Test order with out of stock items"""
        # Arrange
        order = Order(id="124", items=["item2"], total=200)
        self.mock_inventory.check_availability.return_value = False
        
        # Act & Assert
        with self.assertRaises(OutOfStockError):
            self.service.process_order(order)
        
        self.mock_payment.charge.assert_not_called()
        self.mock_email.send_confirmation.assert_not_called()

# Integration test with partial mocking
class TestOrderServiceIntegration(unittest.TestCase):
    def test_with_real_inventory(self):
        """Test with real inventory service but mocked others"""
        real_inventory = InventoryService()  # Real implementation
        mock_payment = Mock()
        mock_email = Mock()
        
        service = OrderService(
            payment_processor=mock_payment,
            inventory_service=real_inventory,
            email_service=mock_email
        )
        
        # Test with real inventory logic
        # ...
```

```typescript
// TypeScript - Testing Instance Methods
describe('OrderService', () => {
    let orderService: OrderService;
    let mockPaymentProcessor: jest.Mocked<PaymentProcessor>;
    let mockInventoryService: jest.Mocked<InventoryService>;
    let mockEmailService: jest.Mocked<EmailService>;
    let mockLogger: jest.Mocked<Logger>;
    
    beforeEach(() => {
        // Create mocks
        mockPaymentProcessor = {
            charge: jest.fn()
        };
        mockInventoryService = {
            checkAvailability: jest.fn(),
            reserveItems: jest.fn()
        };
        mockEmailService = {
            sendConfirmation: jest.fn()
        };
        mockLogger = {
            info: jest.fn(),
            error: jest.fn()
        };
        
        // Create service with mocks
        orderService = new OrderService(
            mockPaymentProcessor,
            mockInventoryService,
            mockEmailService,
            mockLogger
        );
    });
    
    describe('processOrder', () => {
        it('should process order successfully', async () => {
            // Arrange
            const order: Order = {
                id: '123',
                items: [{ id: 'item1', quantity: 1 }],
                total: 100,
                customer: { id: 'cust1' }
            };
            
            mockInventoryService.checkAvailability.mockResolvedValue(true);
            mockPaymentProcessor.charge.mockResolvedValue({ 
                success: true, 
                transactionId: 'tx123' 
            });
            
            // Act
            const result = await orderService.processOrder(order);
            
            // Assert
            expect(result.success).toBe(true);
            expect(mockInventoryService.checkAvailability).toHaveBeenCalledWith(order.items);
            expect(mockPaymentProcessor.charge).toHaveBeenCalledWith(order.customer, order.total);
            expect(mockInventoryService.reserveItems).toHaveBeenCalledWith(order.items);
            expect(mockEmailService.sendConfirmation).toHaveBeenCalledWith(order);
            expect(mockLogger.info).toHaveBeenCalledWith(expect.stringContaining('123'));
        });
        
        it('should throw error when items not available', async () => {
            // Arrange
            const order: Order = {
                id: '124',
                items: [{ id: 'item2', quantity: 1 }],
                total: 200,
                customer: { id: 'cust1' }
            };
            
            mockInventoryService.checkAvailability.mockResolvedValue(false);
            
            // Act & Assert
            await expect(orderService.processOrder(order)).rejects.toThrow(OutOfStockError);
            expect(mockPaymentProcessor.charge).not.toHaveBeenCalled();
            expect(mockEmailService.sendConfirmation).not.toHaveBeenCalled();
            expect(mockLogger.error).toHaveBeenCalled();
        });
    });
});
```

### Testing Singletons

```python
# Python - Testing Singletons
class TestSingletonService(unittest.TestCase):
    def setUp(self):
        """Reset singleton state before each test"""
        SingletonService._instance = None
        SingletonService._dependencies = {}
    
    def test_singleton_with_mocked_dependencies(self):
        """Test singleton with injected mocks"""
        mock_db = Mock()
        mock_cache = Mock()
        
        # Configure before first use
        SingletonService.configure(
            database=mock_db,
            cache=mock_cache
        )
        
        service = SingletonService()
        result = service.get_data("test")
        
        mock_db.query.assert_called_once()
        mock_cache.get.assert_called_once()
    
    def test_singleton_reset(self):
        """Test resetting singleton for isolation"""
        # First test
        service1 = SingletonService()
        service1.set_value("test", "value1")
        
        # Reset for second test
        SingletonService.reset()
        
        service2 = SingletonService()
        self.assertIsNone(service2.get_value("test"))
    
    @patch.object(SingletonService, '_instance', None)
    def test_singleton_initialization_error(self):
        """Test handling initialization errors"""
        mock_db = Mock()
        mock_db.connect.side_effect = ConnectionError("DB Error")
        
        SingletonService.configure(database=mock_db)
        
        with self.assertRaises(ConnectionError):
            SingletonService()
```

```typescript
// TypeScript - Testing Singletons
describe('SingletonService', () => {
    beforeEach(() => {
        // Reset singleton state
        (SingletonService as any).instance = undefined;
        ServiceRegistry.reset();
    });
    
    afterEach(() => {
        // Clean up
        (SingletonService as any).instance = undefined;
        ServiceRegistry.reset();
    });
    
    it('should work with mocked dependencies', async () => {
        const mockDatabase = {
            query: jest.fn().mockResolvedValue([{ id: 1 }])
        };
        const mockLogger = {
            info: jest.fn(),
            error: jest.fn()
        };
        
        // Initialize with mocks
        SingletonService.initialize(mockDatabase, mockLogger);
        
        const service = SingletonService.getInstance();
        const result = await service.getData('test');
        
        expect(mockDatabase.query).toHaveBeenCalled();
        expect(mockLogger.info).toHaveBeenCalled();
        expect(result).toEqual([{ id: 1 }]);
    });
    
    it('should throw error if not initialized', () => {
        expect(() => SingletonService.getInstance()).toThrow('not initialized');
    });
    
    it('should handle initialization errors', () => {
        const mockDatabase = {
            connect: jest.fn().mockImplementation(() => {
                throw new Error('Connection failed');
            })
        };
        
        expect(() => SingletonService.initialize(mockDatabase, {} as Logger))
            .toThrow('Connection failed');
    });
});

// Testing with Service Registry
describe('ServiceRegistry', () => {
    beforeEach(() => {
        ServiceRegistry.reset();
    });
    
    it('should provide isolated instances for tests', () => {
        // Register test doubles
        ServiceRegistry.registerSingleton('database', () => ({
            query: jest.fn().mockResolvedValue([])
        }));
        
        ServiceRegistry.registerSingleton('cache', () => ({
            get: jest.fn(),
            set: jest.fn()
        }));
        
        const db = ServiceRegistry.get('database');
        const cache = ServiceRegistry.get('cache');
        
        expect(db).toBeDefined();
        expect(cache).toBeDefined();
    });
});
```

## Mocking and Stubbing

### Advanced Mocking Patterns

```python
# Python - Advanced Mocking
from unittest.mock import Mock, MagicMock, PropertyMock, AsyncMock

# Mocking properties
class ServiceWithProperties:
    def __init__(self, config):
        self._config = config
    
    @property
    def timeout(self):
        return self._config.get('timeout', 30)
    
    @property
    def is_connected(self):
        return self._check_connection()

# Test with property mocks
mock_service = Mock(spec=ServiceWithProperties)
type(mock_service).timeout = PropertyMock(return_value=10)
type(mock_service).is_connected = PropertyMock(return_value=True)

# Async mocking
class AsyncService:
    async def fetch_data(self):
        async with aiohttp.ClientSession() as session:
            async with session.get('http://api.example.com') as response:
                return await response.json()

# Test async methods
async def test_async_service():
    service = AsyncService()
    service.fetch_data = AsyncMock(return_value={'data': 'test'})
    
    result = await service.fetch_data()
    service.fetch_data.assert_awaited_once()

# Context manager mocking
class DatabaseConnection:
    def __enter__(self):
        self.connect()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

# Test context managers
mock_db = MagicMock(spec=DatabaseConnection)
mock_db.__enter__.return_value = mock_db
mock_db.query.return_value = [{'id': 1}]

with mock_db as db:
    result = db.query('SELECT * FROM users')
    
mock_db.__enter__.assert_called_once()
mock_db.__exit__.assert_called_once()
```

```typescript
// TypeScript - Advanced Mocking
// Mocking modules
jest.mock('./database', () => ({
    Database: jest.fn().mockImplementation(() => ({
        connect: jest.fn().mockResolvedValue(true),
        query: jest.fn().mockResolvedValue([])
    }))
}));

// Mocking with custom implementations
class ComplexService {
    async processData(data: any[]): Promise<ProcessedData> {
        const validated = await this.validate(data);
        const transformed = this.transform(validated);
        return this.save(transformed);
    }
    
    private async validate(data: any[]): Promise<any[]> {
        // Complex validation
        return data;
    }
    
    private transform(data: any[]): any[] {
        // Complex transformation
        return data;
    }
    
    private async save(data: any[]): Promise<ProcessedData> {
        // Save to database
        return { success: true, count: data.length };
    }
}

// Test with partial mocking
describe('ComplexService', () => {
    it('should process data with partial mocks', async () => {
        const service = new ComplexService();
        
        // Mock only specific methods
        jest.spyOn(service as any, 'validate').mockResolvedValue([1, 2, 3]);
        jest.spyOn(service as any, 'save').mockResolvedValue({ 
            success: true, 
            count: 3 
        });
        
        // Let transform run normally
        const transformSpy = jest.spyOn(service as any, 'transform');
        
        const result = await service.processData([]);
        
        expect(result).toEqual({ success: true, count: 3 });
        expect(transformSpy).toHaveBeenCalledWith([1, 2, 3]);
    });
});

// Mocking time-based functionality
describe('TimeBasedService', () => {
    beforeEach(() => {
        jest.useFakeTimers();
    });
    
    afterEach(() => {
        jest.useRealTimers();
    });
    
    it('should handle timeouts correctly', async () => {
        const service = new TimeBasedService();
        const promise = service.delayedOperation(1000);
        
        // Fast-forward time
        jest.advanceTimersByTime(1000);
        
        const result = await promise;
        expect(result).toBe('completed');
    });
});
```

## Framework-Specific DI

### Python Framework DI

```python
# Flask with Flask-Injector
from flask import Flask
from flask_injector import FlaskInjector
from injector import inject, singleton

# Define interfaces (protocols)
class DatabaseProtocol(Protocol):
    def query(self, sql: str) -> List[Any]: ...

class CacheProtocol(Protocol):
    def get(self, key: str) -> Optional[Any]: ...
    def set(self, key: str, value: Any) -> None: ...

# Implementations
class PostgresDatabase:
    def query(self, sql: str) -> List[Any]:
        # Real implementation
        pass

class RedisCache:
    def get(self, key: str) -> Optional[Any]:
        # Real implementation
        pass
    
    def set(self, key: str, value: Any) -> None:
        # Real implementation
        pass

# Service with injected dependencies
class UserService:
    @inject
    def __init__(self, db: DatabaseProtocol, cache: CacheProtocol):
        self.db = db
        self.cache = cache
    
    def get_user(self, user_id: int):
        cached = self.cache.get(f"user:{user_id}")
        if cached:
            return cached
        
        user = self.db.query(f"SELECT * FROM users WHERE id = {user_id}")
        self.cache.set(f"user:{user_id}", user)
        return user

# Configure bindings
def configure(binder):
    binder.bind(DatabaseProtocol, to=PostgresDatabase, scope=singleton)
    binder.bind(CacheProtocol, to=RedisCache, scope=singleton)

# Flask app setup
app = Flask(__name__)

@app.route('/users/<int:user_id>')
@inject
def get_user(user_id: int, user_service: UserService):
    return user_service.get_user(user_id)

# Initialize injector
FlaskInjector(app=app, modules=[configure])

# Django with django-injector
from django_injector import inject
from myapp.services import EmailService, SMSService

class NotificationView(View):
    @inject
    def __init__(self, email_service: EmailService, sms_service: SMSService):
        self.email_service = email_service
        self.sms_service = sms_service
    
    def post(self, request):
        # Use injected services
        self.email_service.send(request.data['email'])
        self.sms_service.send(request.data['phone'])
        return JsonResponse({'status': 'sent'})
```

### TypeScript Framework DI

```typescript
// NestJS Built-in DI
import { Injectable, Module, Inject } from '@nestjs/common';

// Define injection tokens
const DATABASE_TOKEN = 'DATABASE_CONNECTION';
const CACHE_TOKEN = 'CACHE_SERVICE';

// Injectable services
@Injectable()
export class DatabaseService {
    async query(sql: string): Promise<any[]> {
        // Implementation
        return [];
    }
}

@Injectable()
export class CacheService {
    private cache = new Map<string, any>();
    
    get(key: string): any {
        return this.cache.get(key);
    }
    
    set(key: string, value: any): void {
        this.cache.set(key, value);
    }
}

// Service with dependencies
@Injectable()
export class UserService {
    constructor(
        @Inject(DATABASE_TOKEN) private database: DatabaseService,
        @Inject(CACHE_TOKEN) private cache: CacheService
    ) {}
    
    async getUser(id: string): Promise<User> {
        const cached = this.cache.get(`user:${id}`);
        if (cached) return cached;
        
        const users = await this.database.query(`SELECT * FROM users WHERE id = ${id}`);
        if (users.length > 0) {
            this.cache.set(`user:${id}`, users[0]);
            return users[0];
        }
        
        throw new NotFoundException();
    }
}

// Module configuration
@Module({
    providers: [
        {
            provide: DATABASE_TOKEN,
            useClass: DatabaseService
        },
        {
            provide: CACHE_TOKEN,
            useClass: CacheService
        },
        UserService
    ],
    exports: [UserService]
})
export class UserModule {}

// Angular DI
import { Injectable, Inject, InjectionToken } from '@angular/core';

// Define injection tokens
export const API_URL = new InjectionToken<string>('api.url');
export const HTTP_TIMEOUT = new InjectionToken<number>('http.timeout');

// Service with injected configuration
@Injectable({
    providedIn: 'root'
})
export class ApiService {
    constructor(
        @Inject(API_URL) private apiUrl: string,
        @Inject(HTTP_TIMEOUT) private timeout: number,
        private http: HttpClient
    ) {}
    
    async fetchData<T>(endpoint: string): Promise<T> {
        return this.http.get<T>(`${this.apiUrl}/${endpoint}`, {
            timeout: this.timeout
        }).toPromise();
    }
}

// Module configuration
@NgModule({
    providers: [
        { provide: API_URL, useValue: 'https://api.example.com' },
        { provide: HTTP_TIMEOUT, useValue: 5000 }
    ]
})
export class AppModule {}

// InversifyJS for non-framework TypeScript
import { Container, injectable, inject } from 'inversify';
import 'reflect-metadata';

const TYPES = {
    Database: Symbol.for('Database'),
    Cache: Symbol.for('Cache'),
    Logger: Symbol.for('Logger')
};

@injectable()
class PostgresDatabase implements Database {
    async query(sql: string): Promise<any[]> {
        // Implementation
        return [];
    }
}

@injectable()
class UserRepository {
    constructor(
        @inject(TYPES.Database) private database: Database,
        @inject(TYPES.Cache) private cache: Cache,
        @inject(TYPES.Logger) private logger: Logger
    ) {}
    
    async findById(id: string): Promise<User> {
        this.logger.info(`Finding user ${id}`);
        // Implementation using injected dependencies
    }
}

// Container setup
const container = new Container();
container.bind<Database>(TYPES.Database).to(PostgresDatabase).inSingletonScope();
container.bind<Cache>(TYPES.Cache).to(RedisCache).inSingletonScope();
container.bind<Logger>(TYPES.Logger).to(ConsoleLogger).inSingletonScope();
container.bind<UserRepository>(UserRepository).toSelf();

// Usage
const userRepo = container.get<UserRepository>(UserRepository);
```

## Advanced Testing Patterns

### Test Doubles Strategy

```python
# Python - Test Double Patterns
from abc import ABC, abstractmethod

# Define interface for test doubles
class PaymentGateway(ABC):
    @abstractmethod
    def process_payment(self, amount: float) -> dict:
        pass

# Stub - Returns predetermined values
class PaymentStub(PaymentGateway):
    def __init__(self, success=True):
        self.success = success
    
    def process_payment(self, amount: float) -> dict:
        if self.success:
            return {"status": "success", "transaction_id": "stub123"}
        return {"status": "failed", "error": "Insufficient funds"}

# Spy - Records interactions
class PaymentSpy(PaymentGateway):
    def __init__(self):
        self.calls = []
    
    def process_payment(self, amount: float) -> dict:
        self.calls.append({"method": "process_payment", "amount": amount})
        return {"status": "success", "transaction_id": "spy123"}
    
    def assert_called_with(self, amount: float):
        assert any(call["amount"] == amount for call in self.calls)

# Fake - Working implementation
class PaymentFake(PaymentGateway):
    def __init__(self):
        self.balance = 1000.0
        self.transactions = []
    
    def process_payment(self, amount: float) -> dict:
        if amount <= self.balance:
            self.balance -= amount
            transaction = {
                "status": "success",
                "transaction_id": f"fake{len(self.transactions)}",
                "amount": amount
            }
            self.transactions.append(transaction)
            return transaction
        return {"status": "failed", "error": "Insufficient funds"}
```

```typescript
// TypeScript - Test Double Patterns
interface PaymentGateway {
    processPayment(amount: number): Promise<PaymentResult>;
}

// Stub
class PaymentStub implements PaymentGateway {
    constructor(private success = true) {}
    
    async processPayment(amount: number): Promise<PaymentResult> {
        if (this.success) {
            return { status: 'success', transactionId: 'stub123' };
        }
        return { status: 'failed', error: 'Insufficient funds' };
    }
}

// Spy
class PaymentSpy implements PaymentGateway {
    calls: Array<{ method: string; args: any[] }> = [];
    
    async processPayment(amount: number): Promise<PaymentResult> {
        this.calls.push({ method: 'processPayment', args: [amount] });
        return { status: 'success', transactionId: 'spy123' };
    }
    
    assertCalledWith(amount: number): void {
        const called = this.calls.some(
            call => call.method === 'processPayment' && call.args[0] === amount
        );
        if (!called) {
            throw new Error(`processPayment not called with ${amount}`);
        }
    }
}

// Fake
class PaymentFake implements PaymentGateway {
    private balance = 1000;
    private transactions: PaymentResult[] = [];
    
    async processPayment(amount: number): Promise<PaymentResult> {
        if (amount <= this.balance) {
            this.balance -= amount;
            const result: PaymentResult = {
                status: 'success',
                transactionId: `fake${this.transactions.length}`,
                amount
            };
            this.transactions.push(result);
            return result;
        }
        return { status: 'failed', error: 'Insufficient funds' };
    }
    
    getBalance(): number {
        return this.balance;
    }
    
    getTransactions(): PaymentResult[] {
        return [...this.transactions];
    }
}

// Usage in tests
describe('OrderService with Test Doubles', () => {
    it('should use stub for simple testing', async () => {
        const paymentStub = new PaymentStub(true);
        const service = new OrderService(paymentStub);
        
        const result = await service.processOrder({ amount: 100 });
        expect(result.status).toBe('success');
    });
    
    it('should use spy for interaction testing', async () => {
        const paymentSpy = new PaymentSpy();
        const service = new OrderService(paymentSpy);
        
        await service.processOrder({ amount: 150 });
        paymentSpy.assertCalledWith(150);
    });
    
    it('should use fake for complex scenarios', async () => {
        const paymentFake = new PaymentFake();
        const service = new OrderService(paymentFake);
        
        // Process multiple orders
        await service.processOrder({ amount: 300 });
        await service.processOrder({ amount: 500 });
        await service.processOrder({ amount: 300 }); // Should fail
        
        expect(paymentFake.getBalance()).toBe(200);
        expect(paymentFake.getTransactions()).toHaveLength(2);
    });
});
```

## Key Testing & DI Takeaways

> [!important] Testing & DI Best Practices
> 1. **Constructor Injection** is most testable - prefer it over other patterns
> 2. **Use interfaces/protocols** to define contracts between components
> 3. **Avoid global state** in static methods - use parameter injection
> 4. **Reset singleton state** between tests for isolation
> 5. **Choose appropriate test doubles** - mocks, stubs, spies, or fakes
> 6. **Leverage framework DI** when available for cleaner code
> 7. **Test behavior, not implementation** - focus on public interfaces

[[Static vs Instance vs Singleton - Part 4 - Real World Examples|Next: Part 4 - Real World Examples →]]