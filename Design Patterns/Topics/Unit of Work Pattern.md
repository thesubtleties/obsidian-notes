# Unit of Work Pattern

#programming #patterns #unit-of-work #transactions #data-access

## What is the Unit of Work Pattern?

The **Unit of Work Pattern** maintains a list of objects affected by a business transaction and coordinates writing out changes and resolving concurrency problems. It keeps track of everything you do during a business transaction that can affect the database, and when you're done, it figures out everything that needs to be done to alter the database as a result of your work.

Think of it like a shopping cart at a store: you add items (changes) to your cart throughout your shopping trip, but nothing is finalized until you go to checkout (commit). If you decide not to buy, you can abandon the cart (rollback) without any changes.

## Core Concepts

### Why Use Unit of Work?

1. **Transaction Management** - Ensures all changes succeed or fail together
2. **Change Tracking** - Automatically tracks what needs to be saved
3. **Performance** - Batches database operations for efficiency
4. **Consistency** - Maintains data integrity across related changes
5. **Memory Efficiency** - Prevents duplicate object instances

## Basic Implementation

### Python Example
```python
from typing import Dict, List, Set, Any, Optional
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
import sqlite3
from datetime import datetime

# Domain models
@dataclass
class Entity(ABC):
    """Base entity with ID"""
    id: Optional[int] = None
    
    def __hash__(self):
        return hash(self.id) if self.id else id(self)
    
    def __eq__(self, other):
        if not isinstance(other, Entity):
            return False
        return self.id == other.id if self.id else self is other

@dataclass
class User(Entity):
    username: str = ""
    email: str = ""
    created_at: Optional[datetime] = None

@dataclass
class Order(Entity):
    user_id: int = 0
    total: float = 0.0
    status: str = "pending"
    created_at: Optional[datetime] = None

@dataclass
class OrderItem(Entity):
    order_id: int = 0
    product_id: int = 0
    quantity: int = 0
    price: float = 0.0

# Unit of Work implementation
class UnitOfWork:
    """Tracks changes and coordinates database updates"""
    
    def __init__(self, connection: sqlite3.Connection):
        self.connection = connection
        self._new: Set[Entity] = set()
        self._dirty: Set[Entity] = set()
        self._removed: Set[Entity] = set()
        self._identity_map: Dict[tuple, Entity] = {}
        self._in_transaction = False
    
    def register_new(self, entity: Entity):
        """Register a new entity to be inserted"""
        if entity in self._dirty:
            raise ValueError("Entity is already registered as dirty")
        if entity in self._removed:
            raise ValueError("Entity is already registered for removal")
        if entity.id is not None:
            raise ValueError("New entity should not have an ID")
        
        self._new.add(entity)
    
    def register_dirty(self, entity: Entity):
        """Register an existing entity that has been modified"""
        if entity.id is None:
            raise ValueError("Entity must have an ID to be marked dirty")
        if entity in self._removed:
            raise ValueError("Removed entity cannot be marked dirty")
        
        # Only register if not new (new entities are saved anyway)
        if entity not in self._new:
            self._dirty.add(entity)
    
    def register_removed(self, entity: Entity):
        """Register an entity to be deleted"""
        if entity.id is None:
            # If new entity without ID, just remove from new
            self._new.discard(entity)
            return
        
        # Remove from other sets if present
        self._new.discard(entity)
        self._dirty.discard(entity)
        
        self._removed.add(entity)
    
    def register_clean(self, entity: Entity):
        """Register an entity as clean (no changes)"""
        # Remove from all tracking sets
        self._new.discard(entity)
        self._dirty.discard(entity)
        self._removed.discard(entity)
    
    def commit(self):
        """Commit all changes to database"""
        if self._in_transaction:
            raise RuntimeError("Already in a transaction")
        
        self._in_transaction = True
        cursor = self.connection.cursor()
        
        try:
            # Start transaction
            cursor.execute("BEGIN")
            
            # Insert new entities
            for entity in self._new:
                self._insert_entity(cursor, entity)
            
            # Update dirty entities
            for entity in self._dirty:
                self._update_entity(cursor, entity)
            
            # Delete removed entities
            for entity in self._removed:
                self._delete_entity(cursor, entity)
            
            # Commit transaction
            self.connection.commit()
            
            # Clear tracking sets
            self._clear()
            
        except Exception as e:
            # Rollback on error
            self.connection.rollback()
            raise
        finally:
            self._in_transaction = False
    
    def rollback(self):
        """Discard all pending changes"""
        self._clear()
        
        # Reload dirty entities from database
        for entity in self._dirty:
            self._reload_entity(entity)
    
    def _insert_entity(self, cursor, entity: Entity):
        """Insert entity into database"""
        if isinstance(entity, User):
            cursor.execute(
                "INSERT INTO users (username, email, created_at) VALUES (?, ?, ?)",
                (entity.username, entity.email, datetime.now().isoformat())
            )
            entity.id = cursor.lastrowid
            entity.created_at = datetime.now()
            
        elif isinstance(entity, Order):
            cursor.execute(
                "INSERT INTO orders (user_id, total, status, created_at) VALUES (?, ?, ?, ?)",
                (entity.user_id, entity.total, entity.status, datetime.now().isoformat())
            )
            entity.id = cursor.lastrowid
            entity.created_at = datetime.now()
            
        elif isinstance(entity, OrderItem):
            cursor.execute(
                "INSERT INTO order_items (order_id, product_id, quantity, price) VALUES (?, ?, ?, ?)",
                (entity.order_id, entity.product_id, entity.quantity, entity.price)
            )
            entity.id = cursor.lastrowid
    
    def _update_entity(self, cursor, entity: Entity):
        """Update entity in database"""
        if isinstance(entity, User):
            cursor.execute(
                "UPDATE users SET username = ?, email = ? WHERE id = ?",
                (entity.username, entity.email, entity.id)
            )
            
        elif isinstance(entity, Order):
            cursor.execute(
                "UPDATE orders SET user_id = ?, total = ?, status = ? WHERE id = ?",
                (entity.user_id, entity.total, entity.status, entity.id)
            )
            
        elif isinstance(entity, OrderItem):
            cursor.execute(
                "UPDATE order_items SET order_id = ?, product_id = ?, quantity = ?, price = ? WHERE id = ?",
                (entity.order_id, entity.product_id, entity.quantity, entity.price, entity.id)
            )
    
    def _delete_entity(self, cursor, entity: Entity):
        """Delete entity from database"""
        table_name = self._get_table_name(entity)
        cursor.execute(f"DELETE FROM {table_name} WHERE id = ?", (entity.id,))
    
    def _get_table_name(self, entity: Entity) -> str:
        """Get table name for entity type"""
        if isinstance(entity, User):
            return "users"
        elif isinstance(entity, Order):
            return "orders"
        elif isinstance(entity, OrderItem):
            return "order_items"
        else:
            raise ValueError(f"Unknown entity type: {type(entity)}")
    
    def _reload_entity(self, entity: Entity):
        """Reload entity from database"""
        # Implementation would fetch from DB and update entity attributes
        pass
    
    def _clear(self):
        """Clear all tracking sets"""
        self._new.clear()
        self._dirty.clear()
        self._removed.clear()

# Repository using Unit of Work
class Repository(ABC):
    """Base repository that uses Unit of Work"""
    
    def __init__(self, unit_of_work: UnitOfWork):
        self.uow = unit_of_work
    
    @abstractmethod
    def _get_table_name(self) -> str:
        pass
    
    @abstractmethod
    def _map_row(self, row) -> Entity:
        pass

class UserRepository(Repository):
    def _get_table_name(self) -> str:
        return "users"
    
    def _map_row(self, row) -> User:
        return User(
            id=row[0],
            username=row[1],
            email=row[2],
            created_at=datetime.fromisoformat(row[3]) if row[3] else None
        )
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        cursor = self.uow.connection.cursor()
        cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
        row = cursor.fetchone()
        return self._map_row(row) if row else None
    
    def add(self, user: User):
        self.uow.register_new(user)
    
    def update(self, user: User):
        self.uow.register_dirty(user)
    
    def remove(self, user: User):
        self.uow.register_removed(user)

# Service layer using Unit of Work
class OrderService:
    """Business logic that coordinates multiple changes"""
    
    def __init__(self, unit_of_work: UnitOfWork):
        self.uow = unit_of_work
        self.user_repo = UserRepository(unit_of_work)
    
    def create_order_with_items(self, user_id: int, items: List[Dict[str, Any]]) -> Order:
        """Create order with multiple items in one transaction"""
        
        # Verify user exists
        user = self.user_repo.find_by_id(user_id)
        if not user:
            raise ValueError(f"User {user_id} not found")
        
        # Create order
        order = Order(
            user_id=user_id,
            total=0.0,
            status="pending"
        )
        self.uow.register_new(order)
        
        # Create order items and calculate total
        total = 0.0
        for item_data in items:
            order_item = OrderItem(
                order_id=0,  # Will be set after order is saved
                product_id=item_data['product_id'],
                quantity=item_data['quantity'],
                price=item_data['price']
            )
            total += order_item.quantity * order_item.price
            self.uow.register_new(order_item)
        
        # Update order total
        order.total = total
        
        # Commit all changes together
        self.uow.commit()
        
        # Update order items with order ID
        for item in items:
            item['order_id'] = order.id
        
        return order

# Usage example
def main():
    # Setup database
    conn = sqlite3.connect(':memory:')
    conn.execute('''
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            username TEXT NOT NULL,
            email TEXT NOT NULL,
            created_at TEXT
        )
    ''')
    conn.execute('''
        CREATE TABLE orders (
            id INTEGER PRIMARY KEY,
            user_id INTEGER NOT NULL,
            total REAL NOT NULL,
            status TEXT NOT NULL,
            created_at TEXT
        )
    ''')
    conn.execute('''
        CREATE TABLE order_items (
            id INTEGER PRIMARY KEY,
            order_id INTEGER NOT NULL,
            product_id INTEGER NOT NULL,
            quantity INTEGER NOT NULL,
            price REAL NOT NULL
        )
    ''')
    
    # Create unit of work
    uow = UnitOfWork(conn)
    
    # Create user
    user = User(username="john", email="john@example.com")
    uow.register_new(user)
    uow.commit()
    
    # Create order with items
    service = OrderService(uow)
    order = service.create_order_with_items(
        user_id=user.id,
        items=[
            {'product_id': 1, 'quantity': 2, 'price': 10.99},
            {'product_id': 2, 'quantity': 1, 'price': 25.50}
        ]
    )
    
    print(f"Created order {order.id} with total ${order.total}")
```

**What's happening here?**
- 🎯 Unit of Work tracks all changes (new, modified, deleted)
- 🎯 Changes are applied together in a transaction
- 🎯 Rollback discards all pending changes
- 🎯 Identity map prevents duplicate objects

### TypeScript Example
```typescript
// Base classes
abstract class Entity {
    id?: number;
    
    equals(other: Entity): boolean {
        if (!this.id || !other.id) return this === other;
        return this.id === other.id && this.constructor === other.constructor;
    }
}

interface IUnitOfWork {
    registerNew(entity: Entity): void;
    registerDirty(entity: Entity): void;
    registerRemoved(entity: Entity): void;
    commit(): Promise<void>;
    rollback(): void;
}

// Domain models
class User extends Entity {
    constructor(
        public username: string,
        public email: string,
        public createdAt?: Date
    ) {
        super();
    }
}

class Product extends Entity {
    constructor(
        public name: string,
        public price: number,
        public stock: number
    ) {
        super();
    }
}

// Unit of Work implementation
class UnitOfWork implements IUnitOfWork {
    private newObjects = new Set<Entity>();
    private dirtyObjects = new Set<Entity>();
    private removedObjects = new Set<Entity>();
    private identityMap = new Map<string, Entity>();
    
    constructor(private db: any) {} // Database connection
    
    registerNew(entity: Entity): void {
        if (entity.id) {
            throw new Error("New entity should not have an ID");
        }
        if (this.dirtyObjects.has(entity) || this.removedObjects.has(entity)) {
            throw new Error("Entity is already registered");
        }
        
        this.newObjects.add(entity);
    }
    
    registerDirty(entity: Entity): void {
        if (!entity.id) {
            throw new Error("Entity must have an ID to be marked dirty");
        }
        if (this.removedObjects.has(entity)) {
            throw new Error("Removed entity cannot be marked dirty");
        }
        
        if (!this.newObjects.has(entity)) {
            this.dirtyObjects.add(entity);
        }
    }
    
    registerRemoved(entity: Entity): void {
        if (!entity.id) {
            this.newObjects.delete(entity);
            return;
        }
        
        this.newObjects.delete(entity);
        this.dirtyObjects.delete(entity);
        this.removedObjects.add(entity);
    }
    
    async commit(): Promise<void> {
        const transaction = await this.db.beginTransaction();
        
        try {
            // Insert new entities
            for (const entity of this.newObjects) {
                await this.insertEntity(entity, transaction);
            }
            
            // Update dirty entities
            for (const entity of this.dirtyObjects) {
                await this.updateEntity(entity, transaction);
            }
            
            // Delete removed entities
            for (const entity of this.removedObjects) {
                await this.deleteEntity(entity, transaction);
            }
            
            await transaction.commit();
            this.clear();
            
        } catch (error) {
            await transaction.rollback();
            throw error;
        }
    }
    
    rollback(): void {
        this.clear();
    }
    
    private async insertEntity(entity: Entity, transaction: any): Promise<void> {
        if (entity instanceof User) {
            const result = await transaction.query(
                'INSERT INTO users (username, email, created_at) VALUES ($1, $2, $3) RETURNING id',
                [entity.username, entity.email, new Date()]
            );
            entity.id = result.rows[0].id;
            entity.createdAt = new Date();
        } else if (entity instanceof Product) {
            const result = await transaction.query(
                'INSERT INTO products (name, price, stock) VALUES ($1, $2, $3) RETURNING id',
                [entity.name, entity.price, entity.stock]
            );
            entity.id = result.rows[0].id;
        }
    }
    
    private async updateEntity(entity: Entity, transaction: any): Promise<void> {
        if (entity instanceof User) {
            await transaction.query(
                'UPDATE users SET username = $1, email = $2 WHERE id = $3',
                [entity.username, entity.email, entity.id]
            );
        } else if (entity instanceof Product) {
            await transaction.query(
                'UPDATE products SET name = $1, price = $2, stock = $3 WHERE id = $4',
                [entity.name, entity.price, entity.stock, entity.id]
            );
        }
    }
    
    private async deleteEntity(entity: Entity, transaction: any): Promise<void> {
        const table = this.getTableName(entity);
        await transaction.query(`DELETE FROM ${table} WHERE id = $1`, [entity.id]);
    }
    
    private getTableName(entity: Entity): string {
        if (entity instanceof User) return 'users';
        if (entity instanceof Product) return 'products';
        throw new Error(`Unknown entity type: ${entity.constructor.name}`);
    }
    
    private clear(): void {
        this.newObjects.clear();
        this.dirtyObjects.clear();
        this.removedObjects.clear();
    }
    
    // Identity Map functionality
    getFromIdentityMap<T extends Entity>(type: new() => T, id: number): T | undefined {
        const key = `${type.name}:${id}`;
        return this.identityMap.get(key) as T;
    }
    
    addToIdentityMap(entity: Entity): void {
        if (entity.id) {
            const key = `${entity.constructor.name}:${entity.id}`;
            this.identityMap.set(key, entity);
        }
    }
}

// Repository pattern with UoW
class Repository<T extends Entity> {
    constructor(
        protected uow: UnitOfWork,
        protected entityType: new(...args: any[]) => T
    ) {}
    
    add(entity: T): void {
        this.uow.registerNew(entity);
    }
    
    update(entity: T): void {
        this.uow.registerDirty(entity);
    }
    
    remove(entity: T): void {
        this.uow.registerRemoved(entity);
    }
}

// Specialized repositories
class UserRepository extends Repository<User> {
    constructor(uow: UnitOfWork) {
        super(uow, User);
    }
    
    async findByEmail(email: string): Promise<User | null> {
        // Check identity map first
        // ... implementation
        return null;
    }
}

// Service using UoW
class InventoryService {
    constructor(
        private uow: UnitOfWork,
        private productRepo: Repository<Product>
    ) {}
    
    async processOrder(items: Array<{productId: number, quantity: number}>): Promise<void> {
        const products: Product[] = [];
        
        // Load all products and check stock
        for (const item of items) {
            const product = await this.loadProduct(item.productId);
            
            if (product.stock < item.quantity) {
                throw new Error(`Insufficient stock for ${product.name}`);
            }
            
            // Update stock
            product.stock -= item.quantity;
            this.uow.registerDirty(product);
            products.push(product);
        }
        
        // All checks passed, commit changes
        await this.uow.commit();
    }
    
    private async loadProduct(id: number): Promise<Product> {
        // Implementation to load product
        throw new Error("Not implemented");
    }
}
```

## Advanced Patterns

### 1. **Nested Unit of Work**
```python
class NestedUnitOfWork(UnitOfWork):
    """Supports nested transactions"""
    
    def __init__(self, connection):
        super().__init__(connection)
        self._savepoints: List[str] = []
        self._parent: Optional[NestedUnitOfWork] = None
    
    def begin_nested(self) -> 'NestedUnitOfWork':
        """Start a nested unit of work"""
        nested = NestedUnitOfWork(self.connection)
        nested._parent = self
        
        # Create savepoint
        savepoint_name = f"sp_{len(self._savepoints)}"
        self.connection.execute(f"SAVEPOINT {savepoint_name}")
        self._savepoints.append(savepoint_name)
        
        return nested
    
    def commit(self):
        if self._parent:
            # Nested commit - transfer changes to parent
            self._parent._new.update(self._new)
            self._parent._dirty.update(self._dirty)
            self._parent._removed.update(self._removed)
            self._clear()
        else:
            # Root commit - execute changes
            super().commit()
    
    def rollback(self):
        if self._parent and self._savepoints:
            # Rollback to savepoint
            savepoint = self._savepoints.pop()
            self.connection.execute(f"ROLLBACK TO SAVEPOINT {savepoint}")
        else:
            # Full rollback
            super().rollback()

# Usage
uow = NestedUnitOfWork(connection)

# Start main transaction
user = User(username="john", email="john@example.com")
uow.register_new(user)

# Nested transaction
nested_uow = uow.begin_nested()
try:
    order = Order(user_id=1, total=100.0)
    nested_uow.register_new(order)
    # Something goes wrong
    raise Exception("Order validation failed")
except:
    nested_uow.rollback()  # Only rollback the order

# User is still pending
uow.commit()  # Commits the user
```

### 2. **Event Sourcing Integration**
```python
@dataclass
class DomainEvent:
    """Base domain event"""
    entity_id: int
    entity_type: str
    timestamp: datetime = field(default_factory=datetime.now)

@dataclass 
class EntityCreated(DomainEvent):
    initial_data: Dict[str, Any] = field(default_factory=dict)

@dataclass
class EntityUpdated(DomainEvent):
    changes: Dict[str, Any] = field(default_factory=dict)

@dataclass
class EntityDeleted(DomainEvent):
    pass

class EventSourcingUnitOfWork(UnitOfWork):
    """UoW that captures domain events"""
    
    def __init__(self, connection, event_store):
        super().__init__(connection)
        self.event_store = event_store
        self._events: List[DomainEvent] = []
    
    def register_new(self, entity: Entity):
        super().register_new(entity)
        
        # Capture creation event
        event = EntityCreated(
            entity_id=id(entity),  # Temporary ID
            entity_type=type(entity).__name__,
            initial_data=self._entity_to_dict(entity)
        )
        self._events.append(event)
    
    def register_dirty(self, entity: Entity):
        # Capture changes before registering
        original = self._load_original(entity)
        changes = self._compute_changes(original, entity)
        
        if changes:
            super().register_dirty(entity)
            
            event = EntityUpdated(
                entity_id=entity.id,
                entity_type=type(entity).__name__,
                changes=changes
            )
            self._events.append(event)
    
    def commit(self):
        # Save events first
        for event in self._events:
            self.event_store.append(event)
        
        # Then commit changes
        super().commit()
        
        # Clear events
        self._events.clear()
    
    def _compute_changes(self, original: Entity, current: Entity) -> Dict[str, Any]:
        """Compute what changed between original and current"""
        changes = {}
        for attr in vars(current):
            if not attr.startswith('_'):
                original_value = getattr(original, attr, None)
                current_value = getattr(current, attr)
                if original_value != current_value:
                    changes[attr] = {
                        'old': original_value,
                        'new': current_value
                    }
        return changes
```

### 3. **Optimistic Locking**
```typescript
interface VersionedEntity extends Entity {
    version: number;
}

class OptimisticLockingUnitOfWork extends UnitOfWork {
    private originalVersions = new Map<Entity, number>();
    
    registerDirty(entity: Entity): void {
        super.registerDirty(entity);
        
        // Store original version for concurrency check
        if (this.isVersioned(entity) && !this.originalVersions.has(entity)) {
            this.originalVersions.set(entity, (entity as VersionedEntity).version);
        }
    }
    
    protected async updateEntity(entity: Entity, transaction: any): Promise<void> {
        if (this.isVersioned(entity)) {
            const versioned = entity as VersionedEntity;
            const originalVersion = this.originalVersions.get(entity);
            
            // Update with version check
            const result = await this.updateWithVersionCheck(
                versioned, 
                originalVersion!, 
                transaction
            );
            
            if (result.rowCount === 0) {
                throw new Error(
                    `Optimistic lock failure: Entity ${entity.constructor.name} ` +
                    `with ID ${entity.id} was modified by another transaction`
                );
            }
            
            // Increment version
            versioned.version++;
        } else {
            await super.updateEntity(entity, transaction);
        }
    }
    
    private async updateWithVersionCheck(
        entity: VersionedEntity,
        expectedVersion: number,
        transaction: any
    ): Promise<any> {
        const table = this.getTableName(entity);
        const setClause = this.buildUpdateSetClause(entity);
        
        return await transaction.query(
            `UPDATE ${table} SET ${setClause}, version = version + 1 
             WHERE id = $1 AND version = $2`,
            [entity.id, expectedVersion]
        );
    }
    
    private isVersioned(entity: Entity): entity is VersionedEntity {
        return 'version' in entity;
    }
}
```

## Integration with Repository Pattern

```python
class UnitOfWorkManager:
    """Manages UoW lifecycle and repository access"""
    
    def __init__(self, connection_factory):
        self.connection_factory = connection_factory
        self._current: Optional[UnitOfWork] = None
    
    def __enter__(self) -> 'UnitOfWorkManager':
        self.begin()
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if exc_type:
            self.rollback()
        else:
            self.commit()
    
    def begin(self):
        """Start new unit of work"""
        if self._current:
            raise RuntimeError("Unit of work already active")
        
        connection = self.connection_factory()
        self._current = UnitOfWork(connection)
    
    def commit(self):
        """Commit current unit of work"""
        if not self._current:
            raise RuntimeError("No active unit of work")
        
        self._current.commit()
        self._current = None
    
    def rollback(self):
        """Rollback current unit of work"""
        if not self._current:
            raise RuntimeError("No active unit of work")
        
        self._current.rollback()
        self._current = None
    
    @property
    def users(self) -> UserRepository:
        """Get user repository for current UoW"""
        if not self._current:
            raise RuntimeError("No active unit of work")
        return UserRepository(self._current)
    
    @property
    def orders(self) -> OrderRepository:
        """Get order repository for current UoW"""
        if not self._current:
            raise RuntimeError("No active unit of work")
        return OrderRepository(self._current)

# Clean usage
uow_manager = UnitOfWorkManager(lambda: sqlite3.connect('app.db'))

with uow_manager as uow:
    # Create user
    user = User(username="john", email="john@example.com")
    uow.users.add(user)
    
    # Create order
    order = Order(user_id=user.id, total=100.0)
    uow.orders.add(order)
    
    # Automatically commits if no exception
```

## Testing with Unit of Work

```python
import unittest
from unittest.mock import Mock, MagicMock

class TestOrderService(unittest.TestCase):
    def setUp(self):
        # Create in-memory database
        self.conn = sqlite3.connect(':memory:')
        self.create_schema()
        
        # Create unit of work
        self.uow = UnitOfWork(self.conn)
        self.service = OrderService(self.uow)
    
    def test_create_order_success(self):
        # Create user first
        user = User(username="test", email="test@example.com")
        self.uow.register_new(user)
        self.uow.commit()
        
        # Create order
        order = self.service.create_order_with_items(
            user_id=user.id,
            items=[
                {'product_id': 1, 'quantity': 2, 'price': 10.0},
                {'product_id': 2, 'quantity': 1, 'price': 20.0}
            ]
        )
        
        # Verify order was created
        self.assertIsNotNone(order.id)
        self.assertEqual(order.total, 40.0)
        self.assertEqual(order.status, "pending")
    
    def test_rollback_on_error(self):
        # Mock connection to simulate error
        mock_conn = Mock()
        mock_cursor = Mock()
        mock_conn.cursor.return_value = mock_cursor
        mock_cursor.execute.side_effect = Exception("Database error")
        
        uow = UnitOfWork(mock_conn)
        
        # Try to commit changes
        user = User(username="test", email="test@example.com")
        uow.register_new(user)
        
        with self.assertRaises(Exception):
            uow.commit()
        
        # Verify rollback was called
        mock_conn.rollback.assert_called_once()
    
    def create_schema(self):
        """Create test database schema"""
        self.conn.executescript('''
            CREATE TABLE users (
                id INTEGER PRIMARY KEY,
                username TEXT NOT NULL,
                email TEXT NOT NULL,
                created_at TEXT
            );
            
            CREATE TABLE orders (
                id INTEGER PRIMARY KEY,
                user_id INTEGER NOT NULL,
                total REAL NOT NULL,
                status TEXT NOT NULL,
                created_at TEXT
            );
        ''')
```

## Common Pitfalls and Solutions

### 1. **Long-Running Transactions**
```python
# BAD: Holding transaction open too long
def process_large_batch(uow: UnitOfWork, items: List[Any]):
    for item in items:  # Could be thousands
        # Process item...
        entity = create_entity(item)
        uow.register_new(entity)
    
    # Transaction held open for entire loop
    uow.commit()

# GOOD: Batch commits
def process_large_batch_better(uow_factory, items: List[Any], batch_size=100):
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        uow = uow_factory()
        
        for item in batch:
            entity = create_entity(item)
            uow.register_new(entity)
        
        uow.commit()  # Commit each batch
```

### 2. **Memory Leaks**
```python
class ImprovedUnitOfWork(UnitOfWork):
    """UoW with memory management"""
    
    def __init__(self, connection, max_tracked_entities=10000):
        super().__init__(connection)
        self.max_tracked_entities = max_tracked_entities
    
    def _check_memory_usage(self):
        """Prevent tracking too many entities"""
        total_tracked = len(self._new) + len(self._dirty) + len(self._removed)
        
        if total_tracked > self.max_tracked_entities:
            raise MemoryError(
                f"Too many entities tracked: {total_tracked}. "
                f"Consider committing in smaller batches."
            )
    
    def register_new(self, entity: Entity):
        self._check_memory_usage()
        super().register_new(entity)
```

## Key Takeaways

> [!important] Unit of Work Pattern Best Practices
> 1. **Keep transactions short** - Don't hold database locks longer than necessary
> 2. **One UoW per request** - In web apps, create one per HTTP request
> 3. **Clear tracking after commit** - Prevent memory leaks
> 4. **Handle concurrency** - Use optimistic locking for concurrent updates
> 5. **Test transaction boundaries** - Ensure rollback works correctly
> 6. **Combine with Repository** - UoW manages transactions, Repository handles queries
> 7. **Consider event sourcing** - Capture changes as events for audit trails

## When to Use Unit of Work

### Good Fit
- ✅ Complex business transactions
- ✅ Multiple related changes
- ✅ Need for rollback capability
- ✅ Domain-driven design
- ✅ Consistency requirements

### Poor Fit
- ❌ Simple CRUD operations
- ❌ Read-heavy applications
- ❌ Single entity updates
- ❌ Using ORM with built-in UoW (Entity Framework, SQLAlchemy)