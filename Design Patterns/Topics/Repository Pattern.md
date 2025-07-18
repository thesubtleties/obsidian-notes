# Repository Pattern

#programming #patterns #repository #data-access #architecture

## What is the Repository Pattern?

The **Repository Pattern** is a design pattern that creates an abstraction layer between your business logic and data access logic. It encapsulates the logic needed to access data sources, providing a more object-oriented view of the persistence layer.

Think of it like a librarian: instead of going directly to the shelves (database), you ask the librarian (repository) for a book. The librarian knows exactly where to find it and how to retrieve it.

## Core Concepts

### Why Use Repository Pattern?

1. **Separation of Concerns** - Business logic doesn't know about database details
2. **Testability** - Easy to mock repositories for unit testing
3. **Flexibility** - Can switch data sources without changing business logic
4. **Consistency** - Standardized way to access data across the application
5. **Query Centralization** - All database queries in one place

## Basic Implementation

### Python Example
```python
from abc import ABC, abstractmethod
from typing import List, Optional, Dict, Any
from dataclasses import dataclass
from datetime import datetime
import sqlite3
import json

# Domain model
@dataclass
class User:
    id: Optional[int] = None
    username: str = ""
    email: str = ""
    created_at: Optional[datetime] = None
    is_active: bool = True

# Repository interface
class IUserRepository(ABC):
    """Abstract repository interface"""
    
    @abstractmethod
    def find_by_id(self, user_id: int) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass
    
    @abstractmethod
    def find_all(self) -> List[User]:
        pass
    
    @abstractmethod
    def save(self, user: User) -> User:
        pass
    
    @abstractmethod
    def delete(self, user_id: int) -> bool:
        pass
    
    @abstractmethod
    def find_active_users(self) -> List[User]:
        pass

# Concrete implementation - SQL
class SqlUserRepository(IUserRepository):
    """SQL implementation of user repository"""
    
    def __init__(self, connection: sqlite3.Connection):
        self.connection = connection
        self._ensure_table()
    
    def _ensure_table(self):
        """Create table if not exists"""
        self.connection.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                username TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                created_at TEXT NOT NULL,
                is_active INTEGER NOT NULL DEFAULT 1
            )
        ''')
        self.connection.commit()
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        cursor = self.connection.execute(
            "SELECT * FROM users WHERE id = ?", (user_id,)
        )
        row = cursor.fetchone()
        return self._row_to_user(row) if row else None
    
    def find_by_email(self, email: str) -> Optional[User]:
        cursor = self.connection.execute(
            "SELECT * FROM users WHERE email = ?", (email,)
        )
        row = cursor.fetchone()
        return self._row_to_user(row) if row else None
    
    def find_all(self) -> List[User]:
        cursor = self.connection.execute("SELECT * FROM users")
        return [self._row_to_user(row) for row in cursor.fetchall()]
    
    def save(self, user: User) -> User:
        if user.id is None:
            # Insert new user
            cursor = self.connection.execute(
                '''INSERT INTO users (username, email, created_at, is_active) 
                   VALUES (?, ?, ?, ?)''',
                (user.username, user.email, 
                 user.created_at or datetime.now().isoformat(),
                 int(user.is_active))
            )
            user.id = cursor.lastrowid
        else:
            # Update existing user
            self.connection.execute(
                '''UPDATE users 
                   SET username = ?, email = ?, is_active = ?
                   WHERE id = ?''',
                (user.username, user.email, int(user.is_active), user.id)
            )
        
        self.connection.commit()
        return user
    
    def delete(self, user_id: int) -> bool:
        cursor = self.connection.execute(
            "DELETE FROM users WHERE id = ?", (user_id,)
        )
        self.connection.commit()
        return cursor.rowcount > 0
    
    def find_active_users(self) -> List[User]:
        cursor = self.connection.execute(
            "SELECT * FROM users WHERE is_active = 1"
        )
        return [self._row_to_user(row) for row in cursor.fetchall()]
    
    def _row_to_user(self, row) -> User:
        """Convert database row to User object"""
        return User(
            id=row[0],
            username=row[1],
            email=row[2],
            created_at=datetime.fromisoformat(row[3]),
            is_active=bool(row[4])
        )

# In-memory implementation for testing
class InMemoryUserRepository(IUserRepository):
    """In-memory implementation for testing"""
    
    def __init__(self):
        self._users: Dict[int, User] = {}
        self._next_id = 1
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        return self._users.get(user_id)
    
    def find_by_email(self, email: str) -> Optional[User]:
        for user in self._users.values():
            if user.email == email:
                return user
        return None
    
    def find_all(self) -> List[User]:
        return list(self._users.values())
    
    def save(self, user: User) -> User:
        if user.id is None:
            user.id = self._next_id
            self._next_id += 1
            user.created_at = user.created_at or datetime.now()
        
        self._users[user.id] = user
        return user
    
    def delete(self, user_id: int) -> bool:
        if user_id in self._users:
            del self._users[user_id]
            return True
        return False
    
    def find_active_users(self) -> List[User]:
        return [u for u in self._users.values() if u.is_active]

# Service layer using repository
class UserService:
    """Business logic layer"""
    
    def __init__(self, user_repository: IUserRepository):
        self.user_repository = user_repository
    
    def register_user(self, username: str, email: str) -> User:
        """Register new user with validation"""
        # Check if email already exists
        existing = self.user_repository.find_by_email(email)
        if existing:
            raise ValueError(f"Email {email} already registered")
        
        # Create and save new user
        user = User(
            username=username,
            email=email,
            created_at=datetime.now()
        )
        return self.user_repository.save(user)
    
    def deactivate_user(self, user_id: int) -> bool:
        """Deactivate user instead of deleting"""
        user = self.user_repository.find_by_id(user_id)
        if not user:
            return False
        
        user.is_active = False
        self.user_repository.save(user)
        return True
    
    def get_active_users_count(self) -> int:
        """Get count of active users"""
        return len(self.user_repository.find_active_users())

# Usage example
# Production
connection = sqlite3.connect('app.db')
user_repo = SqlUserRepository(connection)
user_service = UserService(user_repo)

# Testing
test_repo = InMemoryUserRepository()
test_service = UserService(test_repo)
```

**What's happening here?**
- 🎯 Repository interface defines data access contract
- 🎯 Concrete implementations handle specific data sources
- 🎯 Service layer uses repository without knowing implementation
- 🎯 Easy to swap between SQL and in-memory for testing

### TypeScript Example
```typescript
// Domain model
interface User {
    id?: number;
    username: string;
    email: string;
    createdAt?: Date;
    isActive: boolean;
}

// Repository interface
interface IUserRepository {
    findById(id: number): Promise<User | null>;
    findByEmail(email: string): Promise<User | null>;
    findAll(): Promise<User[]>;
    save(user: User): Promise<User>;
    delete(id: number): Promise<boolean>;
    findActiveUsers(): Promise<User[]>;
}

// Base repository with common functionality
abstract class BaseRepository<T> {
    protected items: T[] = [];
    
    abstract getIdField(): keyof T;
    
    protected generateId(): number {
        return Math.max(0, ...this.items.map(item => 
            (item[this.getIdField()] as any) || 0
        )) + 1;
    }
}

// SQL implementation
class SqlUserRepository implements IUserRepository {
    constructor(private db: any) {} // db would be your database connection
    
    async findById(id: number): Promise<User | null> {
        const result = await this.db.query(
            'SELECT * FROM users WHERE id = $1', [id]
        );
        return result.rows[0] ? this.mapRowToUser(result.rows[0]) : null;
    }
    
    async findByEmail(email: string): Promise<User | null> {
        const result = await this.db.query(
            'SELECT * FROM users WHERE email = $1', [email]
        );
        return result.rows[0] ? this.mapRowToUser(result.rows[0]) : null;
    }
    
    async findAll(): Promise<User[]> {
        const result = await this.db.query('SELECT * FROM users');
        return result.rows.map(this.mapRowToUser);
    }
    
    async save(user: User): Promise<User> {
        if (user.id) {
            // Update
            await this.db.query(
                `UPDATE users SET username = $1, email = $2, is_active = $3 
                 WHERE id = $4`,
                [user.username, user.email, user.isActive, user.id]
            );
            return user;
        } else {
            // Insert
            const result = await this.db.query(
                `INSERT INTO users (username, email, created_at, is_active) 
                 VALUES ($1, $2, $3, $4) RETURNING id`,
                [user.username, user.email, new Date(), user.isActive]
            );
            return { ...user, id: result.rows[0].id };
        }
    }
    
    async delete(id: number): Promise<boolean> {
        const result = await this.db.query(
            'DELETE FROM users WHERE id = $1', [id]
        );
        return result.rowCount > 0;
    }
    
    async findActiveUsers(): Promise<User[]> {
        const result = await this.db.query(
            'SELECT * FROM users WHERE is_active = true'
        );
        return result.rows.map(this.mapRowToUser);
    }
    
    private mapRowToUser(row: any): User {
        return {
            id: row.id,
            username: row.username,
            email: row.email,
            createdAt: new Date(row.created_at),
            isActive: row.is_active
        };
    }
}

// MongoDB implementation
class MongoUserRepository implements IUserRepository {
    constructor(private collection: any) {} // MongoDB collection
    
    async findById(id: number): Promise<User | null> {
        const doc = await this.collection.findOne({ _id: id });
        return doc ? this.mapDocumentToUser(doc) : null;
    }
    
    async findByEmail(email: string): Promise<User | null> {
        const doc = await this.collection.findOne({ email });
        return doc ? this.mapDocumentToUser(doc) : null;
    }
    
    async findAll(): Promise<User[]> {
        const docs = await this.collection.find({}).toArray();
        return docs.map(this.mapDocumentToUser);
    }
    
    async save(user: User): Promise<User> {
        if (user.id) {
            await this.collection.updateOne(
                { _id: user.id },
                { $set: { 
                    username: user.username,
                    email: user.email,
                    isActive: user.isActive
                }}
            );
            return user;
        } else {
            const result = await this.collection.insertOne({
                username: user.username,
                email: user.email,
                createdAt: new Date(),
                isActive: user.isActive
            });
            return { ...user, id: result.insertedId };
        }
    }
    
    async delete(id: number): Promise<boolean> {
        const result = await this.collection.deleteOne({ _id: id });
        return result.deletedCount > 0;
    }
    
    async findActiveUsers(): Promise<User[]> {
        const docs = await this.collection.find({ isActive: true }).toArray();
        return docs.map(this.mapDocumentToUser);
    }
    
    private mapDocumentToUser(doc: any): User {
        return {
            id: doc._id,
            username: doc.username,
            email: doc.email,
            createdAt: doc.createdAt,
            isActive: doc.isActive
        };
    }
}
```

## Advanced Repository Patterns

### 1. **Generic Repository**
```python
from typing import TypeVar, Generic, Type
from abc import ABC, abstractmethod

T = TypeVar('T')

class IRepository(Generic[T], ABC):
    """Generic repository interface"""
    
    @abstractmethod
    def find_by_id(self, id: Any) -> Optional[T]:
        pass
    
    @abstractmethod
    def find_all(self) -> List[T]:
        pass
    
    @abstractmethod
    def save(self, entity: T) -> T:
        pass
    
    @abstractmethod
    def delete(self, id: Any) -> bool:
        pass

class SqlRepository(IRepository[T]):
    """Generic SQL repository"""
    
    def __init__(self, connection, entity_type: Type[T], table_name: str):
        self.connection = connection
        self.entity_type = entity_type
        self.table_name = table_name
    
    def find_by_id(self, id: Any) -> Optional[T]:
        cursor = self.connection.execute(
            f"SELECT * FROM {self.table_name} WHERE id = ?", (id,)
        )
        row = cursor.fetchone()
        return self._map_row_to_entity(row) if row else None
    
    def find_all(self) -> List[T]:
        cursor = self.connection.execute(f"SELECT * FROM {self.table_name}")
        return [self._map_row_to_entity(row) for row in cursor.fetchall()]
    
    # ... other methods
```

### 2. **Specification Pattern with Repository**
```python
class Specification(ABC):
    """Base specification for queries"""
    
    @abstractmethod
    def is_satisfied_by(self, candidate: Any) -> bool:
        pass
    
    @abstractmethod
    def to_sql(self) -> tuple[str, list]:
        """Convert to SQL WHERE clause"""
        pass

class ActiveUserSpecification(Specification):
    def is_satisfied_by(self, user: User) -> bool:
        return user.is_active
    
    def to_sql(self) -> tuple[str, list]:
        return "is_active = ?", [1]

class EmailDomainSpecification(Specification):
    def __init__(self, domain: str):
        self.domain = domain
    
    def is_satisfied_by(self, user: User) -> bool:
        return user.email.endswith(f"@{self.domain}")
    
    def to_sql(self) -> tuple[str, list]:
        return "email LIKE ?", [f"%@{self.domain}"]

class AndSpecification(Specification):
    def __init__(self, *specs: Specification):
        self.specs = specs
    
    def is_satisfied_by(self, candidate: Any) -> bool:
        return all(spec.is_satisfied_by(candidate) for spec in self.specs)
    
    def to_sql(self) -> tuple[str, list]:
        clauses = []
        params = []
        for spec in self.specs:
            clause, spec_params = spec.to_sql()
            clauses.append(f"({clause})")
            params.extend(spec_params)
        return " AND ".join(clauses), params

# Enhanced repository with specifications
class SpecificationUserRepository(SqlUserRepository):
    def find_by_specification(self, spec: Specification) -> List[User]:
        """Find users matching specification"""
        where_clause, params = spec.to_sql()
        cursor = self.connection.execute(
            f"SELECT * FROM users WHERE {where_clause}", params
        )
        return [self._row_to_user(row) for row in cursor.fetchall()]

# Usage
repo = SpecificationUserRepository(connection)

# Find active Gmail users
gmail_active_spec = AndSpecification(
    ActiveUserSpecification(),
    EmailDomainSpecification("gmail.com")
)
gmail_users = repo.find_by_specification(gmail_active_spec)
```

### 3. **Unit of Work Integration**
```python
class UnitOfWork:
    """Tracks changes and coordinates saving"""
    
    def __init__(self, connection):
        self.connection = connection
        self._new = []
        self._dirty = []
        self._removed = []
    
    def register_new(self, entity):
        self._new.append(entity)
    
    def register_dirty(self, entity):
        if entity not in self._dirty:
            self._dirty.append(entity)
    
    def register_removed(self, entity):
        self._removed.append(entity)
    
    def commit(self):
        """Save all changes in a transaction"""
        try:
            # Process all changes
            for entity in self._new:
                self._insert(entity)
            
            for entity in self._dirty:
                self._update(entity)
            
            for entity in self._removed:
                self._delete(entity)
            
            self.connection.commit()
            self._clear()
        except Exception:
            self.connection.rollback()
            raise
    
    def _clear(self):
        self._new.clear()
        self._dirty.clear()
        self._removed.clear()

# Repository with UoW awareness
class UowUserRepository(IUserRepository):
    def __init__(self, unit_of_work: UnitOfWork):
        self.uow = unit_of_work
        self._identity_map = {}
    
    def save(self, user: User) -> User:
        if user.id is None:
            self.uow.register_new(user)
        else:
            self.uow.register_dirty(user)
        return user
    
    def delete(self, user_id: int) -> bool:
        user = self.find_by_id(user_id)
        if user:
            self.uow.register_removed(user)
            return True
        return False
```

## Common Repository Patterns

### 1. **Query Object Pattern**
```typescript
class UserQuery {
    private conditions: string[] = [];
    private parameters: any[] = [];
    private orderByClause?: string;
    private limitValue?: number;
    
    whereActive(): UserQuery {
        this.conditions.push('is_active = ?');
        this.parameters.push(true);
        return this;
    }
    
    whereEmailLike(pattern: string): UserQuery {
        this.conditions.push('email LIKE ?');
        this.parameters.push(pattern);
        return this;
    }
    
    whereCreatedAfter(date: Date): UserQuery {
        this.conditions.push('created_at > ?');
        this.parameters.push(date);
        return this;
    }
    
    orderBy(field: string, direction: 'ASC' | 'DESC' = 'ASC'): UserQuery {
        this.orderByClause = `${field} ${direction}`;
        return this;
    }
    
    limit(count: number): UserQuery {
        this.limitValue = count;
        return this;
    }
    
    build(): { sql: string; params: any[] } {
        let sql = 'SELECT * FROM users';
        
        if (this.conditions.length > 0) {
            sql += ' WHERE ' + this.conditions.join(' AND ');
        }
        
        if (this.orderByClause) {
            sql += ' ORDER BY ' + this.orderByClause;
        }
        
        if (this.limitValue) {
            sql += ' LIMIT ' + this.limitValue;
        }
        
        return { sql, params: this.parameters };
    }
}

// Repository with query object
class QueryableUserRepository extends SqlUserRepository {
    async findByQuery(query: UserQuery): Promise<User[]> {
        const { sql, params } = query.build();
        const result = await this.db.query(sql, params);
        return result.rows.map(this.mapRowToUser);
    }
}

// Usage
const recentActiveUsers = await repo.findByQuery(
    new UserQuery()
        .whereActive()
        .whereCreatedAfter(new Date('2023-01-01'))
        .orderBy('created_at', 'DESC')
        .limit(10)
);
```

### 2. **Caching Repository**
```python
from functools import lru_cache
import time

class CachingUserRepository(IUserRepository):
    """Repository with caching layer"""
    
    def __init__(self, inner_repository: IUserRepository, cache_ttl: int = 300):
        self.inner = inner_repository
        self.cache_ttl = cache_ttl
        self._cache = {}
        self._cache_times = {}
    
    def find_by_id(self, user_id: int) -> Optional[User]:
        cache_key = f"user:{user_id}"
        
        # Check cache
        if self._is_cache_valid(cache_key):
            return self._cache[cache_key]
        
        # Load from inner repository
        user = self.inner.find_by_id(user_id)
        self._set_cache(cache_key, user)
        return user
    
    def save(self, user: User) -> User:
        # Save through to inner repository
        saved_user = self.inner.save(user)
        
        # Update cache
        if saved_user.id:
            cache_key = f"user:{saved_user.id}"
            self._set_cache(cache_key, saved_user)
        
        # Invalidate list caches
        self._invalidate_pattern("users:*")
        
        return saved_user
    
    def _is_cache_valid(self, key: str) -> bool:
        if key not in self._cache:
            return False
        
        cache_time = self._cache_times.get(key, 0)
        return (time.time() - cache_time) < self.cache_ttl
    
    def _set_cache(self, key: str, value: Any):
        self._cache[key] = value
        self._cache_times[key] = time.time()
    
    def _invalidate_pattern(self, pattern: str):
        """Invalidate cache entries matching pattern"""
        import fnmatch
        keys_to_remove = [
            key for key in self._cache.keys()
            if fnmatch.fnmatch(key, pattern)
        ]
        for key in keys_to_remove:
            del self._cache[key]
            del self._cache_times[key]
```

## Testing with Repository Pattern

### Easy Unit Testing
```python
import unittest
from unittest.mock import Mock

class TestUserService(unittest.TestCase):
    def setUp(self):
        # Use in-memory repository for tests
        self.repository = InMemoryUserRepository()
        self.service = UserService(self.repository)
    
    def test_register_user(self):
        # Register new user
        user = self.service.register_user("john", "john@example.com")
        
        # Verify user was saved
        self.assertIsNotNone(user.id)
        self.assertEqual(user.username, "john")
        
        # Verify can retrieve user
        found = self.repository.find_by_id(user.id)
        self.assertEqual(found.email, "john@example.com")
    
    def test_duplicate_email_rejected(self):
        # Register first user
        self.service.register_user("john", "john@example.com")
        
        # Try to register with same email
        with self.assertRaises(ValueError):
            self.service.register_user("jane", "john@example.com")
    
    def test_deactivate_user(self):
        # Create active user
        user = self.service.register_user("john", "john@example.com")
        self.assertTrue(user.is_active)
        
        # Deactivate
        success = self.service.deactivate_user(user.id)
        self.assertTrue(success)
        
        # Verify user is inactive
        updated = self.repository.find_by_id(user.id)
        self.assertFalse(updated.is_active)

# Integration testing with real database
class TestSqlUserRepository(unittest.TestCase):
    def setUp(self):
        # Use in-memory SQLite for tests
        self.connection = sqlite3.connect(':memory:')
        self.repository = SqlUserRepository(self.connection)
    
    def tearDown(self):
        self.connection.close()
    
    def test_persistence(self):
        # Save user
        user = User(username="test", email="test@example.com")
        saved = self.repository.save(user)
        
        # Retrieve from database
        found = self.repository.find_by_id(saved.id)
        self.assertEqual(found.username, "test")
        self.assertEqual(found.email, "test@example.com")
```

## Key Takeaways

> [!important] Repository Pattern Best Practices
> 1. **Keep repositories focused** - One repository per aggregate root
> 2. **Use interfaces** - Define contracts for easy testing and swapping
> 3. **No business logic** - Repositories only handle data access
> 4. **Return domain objects** - Not database rows or documents
> 5. **Consider specifications** - For complex query requirements
> 6. **Add caching carefully** - Can complicate consistency
> 7. **Test with in-memory versions** - Fast and isolated

## When to Use Repository Pattern

### Good Fit
- ✅ Complex domain models
- ✅ Multiple data sources
- ✅ Need for testability
- ✅ Rich query requirements
- ✅ Domain-driven design

### Poor Fit
- ❌ Simple CRUD applications
- ❌ Direct database access is sufficient
- ❌ Using an ORM that already provides abstraction
- ❌ Performance-critical queries need optimization