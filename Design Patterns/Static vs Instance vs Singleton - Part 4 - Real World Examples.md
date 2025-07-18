# Part 4: Real-World Examples

#programming #python #typescript #patterns #real-world #architecture

[[Static vs Instance vs Singleton Patterns - Python & TypeScript Deep Dive|← Back to Main Guide]] | [[Static vs Instance vs Singleton - Part 3 - Dependency Injection and Testing|← Part 3: DI & Testing]]

## Table of Contents

1. [[#Database Service Patterns]]
2. [[#Content Safety and Validation Services]]
3. [[#AI Agent and Chat Services]]
4. [[#Utility and Helper Services]]
5. [[#API Client Patterns]]
6. [[#Caching Services]]
7. [[#Authentication Services]]
8. [[#Real-World Architecture Examples]]

## Database Service Patterns

### Python Database Patterns

#### Pattern 1: Singleton Connection Pool

```python
import psycopg2
from psycopg2 import pool
import threading
from contextlib import contextmanager

class DatabasePool:
    """Singleton pattern for database connection pooling"""
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
                    cls._instance._initialize_pool()
        return cls._instance
    
    def _initialize_pool(self):
        """Initialize the connection pool"""
        self.pool = psycopg2.pool.ThreadedConnectionPool(
            minconn=2,
            maxconn=20,
            host=os.getenv('DB_HOST', 'localhost'),
            database=os.getenv('DB_NAME', 'myapp'),
            user=os.getenv('DB_USER', 'postgres'),
            password=os.getenv('DB_PASSWORD', '')
        )
    
    @contextmanager
    def get_connection(self):
        """Get a connection from the pool"""
        connection = self.pool.getconn()
        try:
            yield connection
            connection.commit()
        except Exception:
            connection.rollback()
            raise
        finally:
            self.pool.putconn(connection)
    
    def close_all_connections(self):
        """Close all connections in the pool"""
        if hasattr(self, 'pool'):
            self.pool.closeall()

# Usage
db_pool = DatabasePool()

def get_user(user_id: int):
    with db_pool.get_connection() as conn:
        with conn.cursor() as cursor:
            cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
            return cursor.fetchone()
```

**What to understand:** The singleton pattern works well for database connection pools because you want exactly one pool managing all connections. The context manager pattern (`@contextmanager`) ensures connections are properly returned to the pool even if errors occur. Thread-safe initialization prevents multiple pools from being created. The key benefit: all parts of your application share the same pool, preventing connection exhaustion. This pattern is ideal when you need global resource management with controlled access.

#### Pattern 2: Instance-Based Repository Pattern

```python
from typing import Optional, List, Dict, Any
from datetime import datetime

class UserRepository:
    """Instance pattern for database operations"""
    
    def __init__(self, connection_pool: DatabasePool):
        self.pool = connection_pool
        
    def find_by_id(self, user_id: int) -> Optional[Dict[str, Any]]:
        """Find user by ID"""
        with self.pool.get_connection() as conn:
            with conn.cursor(cursor_factory=RealDictCursor) as cursor:
                cursor.execute(
                    "SELECT * FROM users WHERE id = %s",
                    (user_id,)
                )
                return cursor.fetchone()
    
    def find_by_email(self, email: str) -> Optional[Dict[str, Any]]:
        """Find user by email"""
        with self.pool.get_connection() as conn:
            with conn.cursor(cursor_factory=RealDictCursor) as cursor:
                cursor.execute(
                    "SELECT * FROM users WHERE email = %s",
                    (email,)
                )
                return cursor.fetchone()
    
    def create(self, user_data: Dict[str, Any]) -> Dict[str, Any]:
        """Create new user"""
        with self.pool.get_connection() as conn:
            with conn.cursor(cursor_factory=RealDictCursor) as cursor:
                cursor.execute(
                    """
                    INSERT INTO users (email, username, created_at)
                    VALUES (%s, %s, %s)
                    RETURNING *
                    """,
                    (user_data['email'], user_data['username'], datetime.now())
                )
                return cursor.fetchone()
    
    def update(self, user_id: int, updates: Dict[str, Any]) -> Optional[Dict[str, Any]]:
        """Update user data"""
        # Build dynamic UPDATE query
        set_clauses = []
        values = []
        for key, value in updates.items():
            set_clauses.append(f"{key} = %s")
            values.append(value)
        
        values.append(user_id)  # For WHERE clause
        
        with self.pool.get_connection() as conn:
            with conn.cursor(cursor_factory=RealDictCursor) as cursor:
                cursor.execute(
                    f"""
                    UPDATE users 
                    SET {', '.join(set_clauses)}, updated_at = %s
                    WHERE id = %s
                    RETURNING *
                    """,
                    values + [datetime.now(), user_id]
                )
                return cursor.fetchone()
```

**What to understand:** The repository pattern with instances provides better testability and flexibility than static methods. Each repository instance can have its own connection pool, enabling different databases or configurations. The methods follow a consistent pattern: get connection, execute query, return results. Dynamic query building (see `update` method) shows how instance methods can build complex queries while maintaining SQL injection safety through parameterization. This pattern scales well with dependency injection.

#### Pattern 3: Static Query Builder

```python
class QueryBuilder:
    """Static methods for building SQL queries"""
    
    @staticmethod
    def select(table: str, columns: List[str] = None, 
               where: Dict[str, Any] = None, 
               order_by: str = None, 
               limit: int = None) -> tuple[str, List[Any]]:
        """Build SELECT query with parameters"""
        # Select clause
        column_list = ', '.join(columns) if columns else '*'
        query = f"SELECT {column_list} FROM {table}"
        params = []
        
        # Where clause
        if where:
            conditions = []
            for key, value in where.items():
                if value is None:
                    conditions.append(f"{key} IS NULL")
                else:
                    conditions.append(f"{key} = %s")
                    params.append(value)
            query += f" WHERE {' AND '.join(conditions)}"
        
        # Order by clause
        if order_by:
            query += f" ORDER BY {order_by}"
        
        # Limit clause
        if limit:
            query += f" LIMIT %s"
            params.append(limit)
        
        return query, params
    
    @staticmethod
    def insert(table: str, data: Dict[str, Any]) -> tuple[str, List[Any]]:
        """Build INSERT query with parameters"""
        columns = list(data.keys())
        values = list(data.values())
        placeholders = ', '.join(['%s'] * len(values))
        
        query = f"""
            INSERT INTO {table} ({', '.join(columns)})
            VALUES ({placeholders})
            RETURNING *
        """
        
        return query, values
    
    @staticmethod
    def bulk_insert(table: str, records: List[Dict[str, Any]]) -> tuple[str, List[Any]]:
        """Build bulk INSERT query"""
        if not records:
            raise ValueError("No records to insert")
        
        # Get columns from first record
        columns = list(records[0].keys())
        
        # Build values placeholders
        value_rows = []
        params = []
        for record in records:
            placeholders = []
            for col in columns:
                placeholders.append('%s')
                params.append(record.get(col))
            value_rows.append(f"({', '.join(placeholders)})")
        
        query = f"""
            INSERT INTO {table} ({', '.join(columns)})
            VALUES {', '.join(value_rows)}
            RETURNING *
        """
        
        return query, params
```

**What to understand:** Static query builders provide reusable SQL generation without state. The key insight: separate query building (pure function) from execution (requires database connection). Parameterized queries prevent SQL injection while the builder handles the complexity of dynamic SQL. The tuple return `(query, params)` maintains separation between SQL structure and data. This pattern is useful for complex queries that need to be built programmatically but don't need instance state.

### TypeScript Database Patterns

#### Pattern 1: Singleton Database Service

```typescript
import { Pool, PoolClient, QueryResult } from 'pg';

interface DatabaseConfig {
    host: string;
    port: number;
    database: string;
    user: string;
    password: string;
    max: number;
}

class DatabaseService {
    private static instance: DatabaseService;
    private pool: Pool;
    private initialized = false;
    
    private constructor(config: DatabaseConfig) {
        this.pool = new Pool(config);
        
        // Handle pool errors
        this.pool.on('error', (err) => {
            console.error('Unexpected error on idle client', err);
        });
    }
    
    static initialize(config: DatabaseConfig): void {
        if (this.instance) {
            throw new Error('Database already initialized');
        }
        this.instance = new DatabaseService(config);
    }
    
    static getInstance(): DatabaseService {
        if (!this.instance) {
            throw new Error('Database not initialized. Call initialize() first.');
        }
        return this.instance;
    }
    
    async query<T = any>(text: string, params?: any[]): Promise<QueryResult<T>> {
        const start = Date.now();
        try {
            const result = await this.pool.query<T>(text, params);
            const duration = Date.now() - start;
            console.log('Executed query', { text, duration, rows: result.rowCount });
            return result;
        } catch (error) {
            console.error('Database query error:', error);
            throw error;
        }
    }
    
    async transaction<T>(
        callback: (client: PoolClient) => Promise<T>
    ): Promise<T> {
        const client = await this.pool.connect();
        try {
            await client.query('BEGIN');
            const result = await callback(client);
            await client.query('COMMIT');
            return result;
        } catch (error) {
            await client.query('ROLLBACK');
            throw error;
        } finally {
            client.release();
        }
    }
    
    async close(): Promise<void> {
        await this.pool.end();
    }
}

// Usage
DatabaseService.initialize({
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT || '5432'),
    database: process.env.DB_NAME || 'myapp',
    user: process.env.DB_USER || 'postgres',
    password: process.env.DB_PASSWORD || '',
    max: 20
});

export const db = DatabaseService.getInstance();
```

**What to understand:** TypeScript's singleton database service shows explicit initialization - you must call `initialize()` before `getInstance()`. This prevents the "magic" of lazy initialization and makes configuration explicit. The transaction method uses PostgreSQL's transaction semantics with proper rollback on errors. The pool event handling (`pool.on('error')`) prevents unhandled errors from crashing the application. Export the instance directly for convenience while maintaining singleton control.

#### Pattern 2: Repository Pattern with Instances

```typescript
interface User {
    id: string;
    email: string;
    username: string;
    createdAt: Date;
    updatedAt: Date;
}

interface Repository<T> {
    findById(id: string): Promise<T | null>;
    findAll(options?: QueryOptions): Promise<T[]>;
    create(data: Partial<T>): Promise<T>;
    update(id: string, data: Partial<T>): Promise<T | null>;
    delete(id: string): Promise<boolean>;
}

interface QueryOptions {
    limit?: number;
    offset?: number;
    orderBy?: string;
    where?: Record<string, any>;
}

class UserRepository implements Repository<User> {
    constructor(private db: DatabaseService) {}
    
    async findById(id: string): Promise<User | null> {
        const result = await this.db.query<User>(
            'SELECT * FROM users WHERE id = $1',
            [id]
        );
        return result.rows[0] || null;
    }
    
    async findByEmail(email: string): Promise<User | null> {
        const result = await this.db.query<User>(
            'SELECT * FROM users WHERE email = $1',
            [email]
        );
        return result.rows[0] || null;
    }
    
    async findAll(options: QueryOptions = {}): Promise<User[]> {
        let query = 'SELECT * FROM users';
        const params: any[] = [];
        let paramIndex = 1;
        
        // Build WHERE clause
        if (options.where && Object.keys(options.where).length > 0) {
            const conditions = Object.entries(options.where).map(([key, value]) => {
                params.push(value);
                return `${key} = $${paramIndex++}`;
            });
            query += ` WHERE ${conditions.join(' AND ')}`;
        }
        
        // Add ORDER BY
        if (options.orderBy) {
            query += ` ORDER BY ${options.orderBy}`;
        }
        
        // Add LIMIT and OFFSET
        if (options.limit) {
            params.push(options.limit);
            query += ` LIMIT $${paramIndex++}`;
        }
        
        if (options.offset) {
            params.push(options.offset);
            query += ` OFFSET $${paramIndex++}`;
        }
        
        const result = await this.db.query<User>(query, params);
        return result.rows;
    }
    
    async create(userData: Partial<User>): Promise<User> {
        const result = await this.db.query<User>(
            `INSERT INTO users (email, username, created_at, updated_at)
             VALUES ($1, $2, NOW(), NOW())
             RETURNING *`,
            [userData.email, userData.username]
        );
        return result.rows[0];
    }
    
    async update(id: string, updates: Partial<User>): Promise<User | null> {
        // Build dynamic UPDATE query
        const updateFields = Object.keys(updates)
            .filter(key => key !== 'id')
            .map((key, index) => `${key} = $${index + 2}`);
        
        if (updateFields.length === 0) {
            return this.findById(id);
        }
        
        const values = [id, ...Object.values(updates).filter((_, i) => 
            Object.keys(updates)[i] !== 'id'
        )];
        
        const result = await this.db.query<User>(
            `UPDATE users 
             SET ${updateFields.join(', ')}, updated_at = NOW()
             WHERE id = $1
             RETURNING *`,
            values
        );
        
        return result.rows[0] || null;
    }
    
    async delete(id: string): Promise<boolean> {
        const result = await this.db.query(
            'DELETE FROM users WHERE id = $1',
            [id]
        );
        return result.rowCount > 0;
    }
    
    // Bulk operations
    async createMany(users: Partial<User>[]): Promise<User[]> {
        return this.db.transaction(async (client) => {
            const results: User[] = [];
            
            for (const userData of users) {
                const result = await client.query<User>(
                    `INSERT INTO users (email, username, created_at, updated_at)
                     VALUES ($1, $2, NOW(), NOW())
                     RETURNING *`,
                    [userData.email, userData.username]
                );
                results.push(result.rows[0]);
            }
            
            return results;
        });
    }
}
```

**What to understand:** The TypeScript repository pattern leverages interfaces (`Repository<T>`) for consistency across different entity types. Generic typing ensures type safety throughout. The query builder in `findAll` shows how to dynamically construct SQL while maintaining type safety and preventing injection. The `createMany` method demonstrates transaction usage - all inserts succeed or all fail. This pattern works well with dependency injection frameworks and makes testing straightforward with mock databases.

#### Pattern 3: Static Query Utilities

```typescript
type WhereCondition = {
    [key: string]: any | { op: string; value: any };
};

class QueryUtils {
    /**
     * Build WHERE clause from conditions object
     */
    static buildWhereClause(
        conditions: WhereCondition,
        startIndex = 1
    ): { clause: string; params: any[]; nextIndex: number } {
        const params: any[] = [];
        const clauses: string[] = [];
        let paramIndex = startIndex;
        
        for (const [field, condition] of Object.entries(conditions)) {
            if (condition === null) {
                clauses.push(`${field} IS NULL`);
            } else if (typeof condition === 'object' && 'op' in condition) {
                // Handle operators like {op: '>', value: 100}
                const { op, value } = condition;
                if (Array.isArray(value) && op.toUpperCase() === 'IN') {
                    const placeholders = value.map(() => `$${paramIndex++}`).join(', ');
                    clauses.push(`${field} IN (${placeholders})`);
                    params.push(...value);
                } else {
                    clauses.push(`${field} ${op} $${paramIndex++}`);
                    params.push(value);
                }
            } else {
                clauses.push(`${field} = $${paramIndex++}`);
                params.push(condition);
            }
        }
        
        return {
            clause: clauses.length > 0 ? `WHERE ${clauses.join(' AND ')}` : '',
            params,
            nextIndex: paramIndex
        };
    }
    
    /**
     * Build INSERT query with RETURNING clause
     */
    static buildInsertQuery(
        table: string,
        data: Record<string, any>,
        returning = '*'
    ): { query: string; params: any[] } {
        const fields = Object.keys(data);
        const values = Object.values(data);
        const placeholders = fields.map((_, i) => `$${i + 1}`).join(', ');
        
        const query = `
            INSERT INTO ${table} (${fields.join(', ')})
            VALUES (${placeholders})
            RETURNING ${returning}
        `;
        
        return { query, params: values };
    }
    
    /**
     * Build UPDATE query with dynamic fields
     */
    static buildUpdateQuery(
        table: string,
        id: string | number,
        updates: Record<string, any>,
        idField = 'id',
        returning = '*'
    ): { query: string; params: any[] } {
        const fields = Object.keys(updates);
        const values = Object.values(updates);
        
        const setClause = fields
            .map((field, i) => `${field} = $${i + 2}`)
            .join(', ');
        
        const query = `
            UPDATE ${table}
            SET ${setClause}
            WHERE ${idField} = $1
            RETURNING ${returning}
        `;
        
        return { query, params: [id, ...values] };
    }
    
    /**
     * Build paginated query
     */
    static paginate(
        query: string,
        page: number,
        pageSize: number,
        params: any[] = []
    ): { query: string; params: any[] } {
        const offset = (page - 1) * pageSize;
        const paginatedQuery = `${query} LIMIT $${params.length + 1} OFFSET $${params.length + 2}`;
        
        return {
            query: paginatedQuery,
            params: [...params, pageSize, offset]
        };
    }
}

// Usage example
const { clause, params } = QueryUtils.buildWhereClause({
    status: 'active',
    age: { op: '>', value: 18 },
    country: { op: 'IN', value: ['US', 'CA', 'UK'] }
});

const query = `SELECT * FROM users ${clause}`;
// Results in: SELECT * FROM users WHERE status = $1 AND age > $2 AND country IN ($3, $4, $5)
```

**What to understand:** TypeScript's static query utilities show type-safe SQL building. The `WhereCondition` type allows both simple equality and complex operators. The parameter index tracking ensures correct placeholder numbering for PostgreSQL. The builder methods return both query and params, maintaining the separation needed for parameterized queries. Static methods work well here because query building is a pure transformation with no state requirements.

## Content Safety and Validation Services

### Python Validation Patterns

```python
import re
from typing import List, Dict, Any, Optional
from functools import lru_cache
import bleach

class ContentValidator:
    """Static methods for content validation"""
    
    # Compile regex patterns at class level for efficiency
    EMAIL_PATTERN = re.compile(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$')
    URL_PATTERN = re.compile(
        r'^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b'
        r'([-a-zA-Z0-9()@:%_\+.~#?&\/\/=]*)$'
    )
    PHONE_PATTERN = re.compile(r'^\+?1?\d{9,15}$')
    
    # Profanity list (simplified example)
    PROFANITY_WORDS = {'badword1', 'badword2', 'badword3'}
    
    @staticmethod
    def validate_email(email: str) -> bool:
        """Validate email format"""
        return bool(ContentValidator.EMAIL_PATTERN.match(email))
    
    @staticmethod
    def validate_url(url: str) -> bool:
        """Validate URL format"""
        return bool(ContentValidator.URL_PATTERN.match(url))
    
    @staticmethod
    def validate_phone(phone: str) -> bool:
        """Validate phone number format"""
        cleaned = re.sub(r'[\s\-\(\)]', '', phone)
        return bool(ContentValidator.PHONE_PATTERN.match(cleaned))
    
    @staticmethod
    @lru_cache(maxsize=1000)
    def contains_profanity(text: str) -> bool:
        """Check if text contains profanity (cached for performance)"""
        words = text.lower().split()
        return any(word in ContentValidator.PROFANITY_WORDS for word in words)
    
    @staticmethod
    def sanitize_html(html: str, allowed_tags: Optional[List[str]] = None) -> str:
        """Sanitize HTML content"""
        if allowed_tags is None:
            allowed_tags = ['p', 'br', 'span', 'div', 'a', 'strong', 'em']
        
        return bleach.clean(
            html,
            tags=allowed_tags,
            attributes={'a': ['href', 'title']},
            strip=True
        )

class ContentModerationService:
    """Instance-based content moderation with ML models"""
    
    def __init__(self, model_path: str, threshold: float = 0.7):
        self.threshold = threshold
        self.model = self._load_model(model_path)
        self.cache = {}
    
    def _load_model(self, path: str):
        """Load ML model for content moderation"""
        # Simplified - in reality would load actual model
        return {"loaded": True, "path": path}
    
    def check_content(self, content: str) -> Dict[str, Any]:
        """Check content for various safety issues"""
        # Check cache first
        if content in self.cache:
            return self.cache[content]
        
        result = {
            'safe': True,
            'issues': [],
            'scores': {}
        }
        
        # Run various checks
        checks = [
            ('profanity', self._check_profanity),
            ('spam', self._check_spam),
            ('toxicity', self._check_toxicity),
            ('pii', self._check_pii)
        ]
        
        for check_name, check_func in checks:
            score = check_func(content)
            result['scores'][check_name] = score
            
            if score > self.threshold:
                result['safe'] = False
                result['issues'].append(check_name)
        
        # Cache result
        self.cache[content] = result
        return result
    
    def _check_profanity(self, content: str) -> float:
        """Check profanity score"""
        # Simplified - would use actual model
        if ContentValidator.contains_profanity(content):
            return 0.9
        return 0.1
    
    def _check_spam(self, content: str) -> float:
        """Check spam score"""
        spam_indicators = ['buy now', 'click here', 'limited offer', '100% free']
        content_lower = content.lower()
        
        score = sum(1 for indicator in spam_indicators if indicator in content_lower)
        return min(score * 0.3, 1.0)
    
    def _check_toxicity(self, content: str) -> float:
        """Check toxicity score using ML model"""
        # Simplified - would use actual model inference
        toxic_patterns = ['hate', 'kill', 'die', 'stupid']
        content_lower = content.lower()
        
        matches = sum(1 for pattern in toxic_patterns if pattern in content_lower)
        return min(matches * 0.25, 1.0)
    
    def _check_pii(self, content: str) -> float:
        """Check for personally identifiable information"""
        score = 0.0
        
        # Check for email
        if re.search(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', content):
            score += 0.3
        
        # Check for phone
        if re.search(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b', content):
            score += 0.3
        
        # Check for SSN pattern
        if re.search(r'\b\d{3}-\d{2}-\d{4}\b', content):
            score += 0.5
        
        return min(score, 1.0)
```

**What to understand:** Python validation shows when to use static vs instance methods. Static methods handle pure validation (email, URL formats) with compiled regex patterns at class level for efficiency. The `@lru_cache` decorator on `contains_profanity` prevents redundant checks. Instance-based `ContentModerationService` maintains state (ML model, cache) and configuration (threshold). The instance pattern enables different moderation policies for different contexts while static validators remain universal.

### TypeScript Validation Patterns

```typescript
interface ValidationResult {
    valid: boolean;
    errors: string[];
}

interface ContentCheckResult {
    safe: boolean;
    issues: string[];
    scores: Record<string, number>;
}

class ContentValidator {
    // Static regex patterns
    private static readonly EMAIL_PATTERN = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    private static readonly URL_PATTERN = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&\/\/=]*)$/;
    private static readonly PHONE_PATTERN = /^\+?1?\d{9,15}$/;
    
    // Cached compiled patterns
    private static profanityRegex: RegExp | null = null;
    
    static validateEmail(email: string): boolean {
        return this.EMAIL_PATTERN.test(email);
    }
    
    static validateUrl(url: string): boolean {
        return this.URL_PATTERN.test(url);
    }
    
    static validatePhone(phone: string): boolean {
        const cleaned = phone.replace(/[\s\-\(\)]/g, '');
        return this.PHONE_PATTERN.test(cleaned);
    }
    
    static validatePassword(password: string): ValidationResult {
        const errors: string[] = [];
        
        if (password.length < 8) {
            errors.push('Password must be at least 8 characters long');
        }
        if (!/[A-Z]/.test(password)) {
            errors.push('Password must contain at least one uppercase letter');
        }
        if (!/[a-z]/.test(password)) {
            errors.push('Password must contain at least one lowercase letter');
        }
        if (!/[0-9]/.test(password)) {
            errors.push('Password must contain at least one number');
        }
        if (!/[^A-Za-z0-9]/.test(password)) {
            errors.push('Password must contain at least one special character');
        }
        
        return {
            valid: errors.length === 0,
            errors
        };
    }
    
    static sanitizeHtml(html: string, allowedTags: string[] = ['p', 'br', 'span', 'div']): string {
        // Simple sanitization - in production use a library like DOMPurify
        const tagPattern = new RegExp(`<(?!\/?(?:${allowedTags.join('|')})\\s*\/?>)[^>]+>`, 'gi');
        return html.replace(tagPattern, '');
    }
    
    private static getProfanityRegex(): RegExp {
        if (!this.profanityRegex) {
            const profanityWords = ['badword1', 'badword2', 'badword3'];
            const pattern = profanityWords.join('|');
            this.profanityRegex = new RegExp(`\\b(${pattern})\\b`, 'gi');
        }
        return this.profanityRegex;
    }
    
    static containsProfanity(text: string): boolean {
        return this.getProfanityRegex().test(text);
    }
}

class ContentModerationService {
    private cache = new Map<string, ContentCheckResult>();
    
    constructor(
        private readonly threshold: number = 0.7,
        private readonly cacheSize: number = 1000
    ) {}
    
    async checkContent(content: string): Promise<ContentCheckResult> {
        // Check cache
        const cached = this.cache.get(content);
        if (cached) {
            return cached;
        }
        
        const result: ContentCheckResult = {
            safe: true,
            issues: [],
            scores: {}
        };
        
        // Run checks in parallel
        const [profanityScore, spamScore, toxicityScore, piiScore] = await Promise.all([
            this.checkProfanity(content),
            this.checkSpam(content),
            this.checkToxicity(content),
            this.checkPII(content)
        ]);
        
        const checks = [
            { name: 'profanity', score: profanityScore },
            { name: 'spam', score: spamScore },
            { name: 'toxicity', score: toxicityScore },
            { name: 'pii', score: piiScore }
        ];
        
        for (const check of checks) {
            result.scores[check.name] = check.score;
            if (check.score > this.threshold) {
                result.safe = false;
                result.issues.push(check.name);
            }
        }
        
        // Manage cache size
        if (this.cache.size >= this.cacheSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
        
        this.cache.set(content, result);
        return result;
    }
    
    private async checkProfanity(content: string): Promise<number> {
        return ContentValidator.containsProfanity(content) ? 0.9 : 0.1;
    }
    
    private async checkSpam(content: string): Promise<number> {
        const spamIndicators = ['buy now', 'click here', 'limited offer', '100% free'];
        const contentLower = content.toLowerCase();
        
        const matches = spamIndicators.filter(indicator => 
            contentLower.includes(indicator)
        ).length;
        
        return Math.min(matches * 0.3, 1.0);
    }
    
    private async checkToxicity(content: string): Promise<number> {
        // In production, this would call an ML model API
        const toxicPatterns = /\b(hate|kill|die|stupid)\b/gi;
        const matches = (content.match(toxicPatterns) || []).length;
        
        return Math.min(matches * 0.25, 1.0);
    }
    
    private async checkPII(content: string): Promise<number> {
        let score = 0;
        
        // Email check
        if (/\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/.test(content)) {
            score += 0.3;
        }
        
        // Phone check
        if (/\b\d{3}[-.]?\d{3}[-.]?\d{4}\b/.test(content)) {
            score += 0.3;
        }
        
        // SSN check
        if (/\b\d{3}-\d{2}-\d{4}\b/.test(content)) {
            score += 0.5;
        }
        
        // Credit card check
        if (/\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b/.test(content)) {
            score += 0.4;
        }
        
        return Math.min(score, 1.0);
    }
}

// Usage
const validator = ContentValidator;
const moderator = new ContentModerationService();

// Static validation
console.log(validator.validateEmail('user@example.com')); // true

// Instance-based moderation
moderator.checkContent('Check out this great offer! Click here now!')
    .then(result => {
        if (!result.safe) {
            console.log('Content flagged for:', result.issues);
        }
    });
```

**What to understand:** TypeScript validation follows similar patterns to Python. Static validators handle stateless checks while the instance-based moderation service manages cache and configuration. The LRU cache implementation (delete first key when full) keeps memory bounded. Running checks in parallel with `Promise.all` improves performance. The separation allows static validators to be tree-shaken in production builds if unused, while instance services maintain necessary state.

## AI Agent and Chat Services

### Python AI Service Patterns

```python
from typing import List, Dict, Any, Optional
from datetime import datetime
import asyncio
from dataclasses import dataclass, field
import openai

@dataclass
class Message:
    role: str  # 'user', 'assistant', 'system'
    content: str
    timestamp: datetime = field(default_factory=datetime.now)
    metadata: Dict[str, Any] = field(default_factory=dict)

class ConversationMemory:
    """Instance-based conversation memory management"""
    
    def __init__(self, max_messages: int = 50, max_tokens: int = 4000):
        self.messages: List[Message] = []
        self.max_messages = max_messages
        self.max_tokens = max_tokens
        self.total_tokens = 0
    
    def add_message(self, message: Message) -> None:
        """Add message to memory with token management"""
        self.messages.append(message)
        
        # Estimate tokens (rough approximation)
        message_tokens = len(message.content.split()) * 1.3
        self.total_tokens += message_tokens
        
        # Trim if needed
        while (len(self.messages) > self.max_messages or 
               self.total_tokens > self.max_tokens) and len(self.messages) > 1:
            removed = self.messages.pop(0)
            self.total_tokens -= len(removed.content.split()) * 1.3
    
    def get_context(self, system_prompt: Optional[str] = None) -> List[Dict[str, str]]:
        """Get conversation context for API"""
        context = []
        
        if system_prompt:
            context.append({"role": "system", "content": system_prompt})
        
        for message in self.messages:
            context.append({
                "role": message.role,
                "content": message.content
            })
        
        return context
    
    def summarize(self) -> str:
        """Create summary of conversation"""
        if not self.messages:
            return "No conversation history."
        
        summary_parts = [
            f"Conversation with {len(self.messages)} messages:",
            f"Started: {self.messages[0].timestamp.strftime('%Y-%m-%d %H:%M')}",
            f"Last message: {self.messages[-1].timestamp.strftime('%Y-%m-%d %H:%M')}"
        ]
        
        # Extract key topics (simplified)
        user_messages = [m for m in self.messages if m.role == 'user']
        if user_messages:
            summary_parts.append(f"User messages: {len(user_messages)}")
        
        return "\n".join(summary_parts)

class AIAgentService:
    """Singleton AI service for managing agents"""
    
    _instance = None
    _lock = asyncio.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance
    
    async def initialize(self, api_key: str, default_model: str = "gpt-4"):
        """Async initialization"""
        if self._initialized:
            return
        
        async with self._lock:
            if self._initialized:
                return
            
            self.api_key = api_key
            self.default_model = default_model
            self.active_conversations: Dict[str, ConversationMemory] = {}
            self.rate_limiter = RateLimiter(requests_per_minute=60)
            
            # Initialize OpenAI client
            openai.api_key = api_key
            
            self._initialized = True
    
    def get_or_create_conversation(self, conversation_id: str) -> ConversationMemory:
        """Get existing or create new conversation"""
        if conversation_id not in self.active_conversations:
            self.active_conversations[conversation_id] = ConversationMemory()
        return self.active_conversations[conversation_id]
    
    async def send_message(
        self,
        conversation_id: str,
        message: str,
        system_prompt: Optional[str] = None,
        model: Optional[str] = None,
        temperature: float = 0.7
    ) -> str:
        """Send message and get response"""
        # Rate limiting
        await self.rate_limiter.acquire()
        
        # Get conversation
        conversation = self.get_or_create_conversation(conversation_id)
        
        # Add user message
        conversation.add_message(Message(role="user", content=message))
        
        # Get context
        context = conversation.get_context(system_prompt)
        
        try:
            # Make API call
            response = await openai.ChatCompletion.acreate(
                model=model or self.default_model,
                messages=context,
                temperature=temperature
            )
            
            # Extract response
            assistant_message = response.choices[0].message.content
            
            # Add to conversation
            conversation.add_message(
                Message(
                    role="assistant",
                    content=assistant_message,
                    metadata={"model": model or self.default_model}
                )
            )
            
            return assistant_message
            
        except Exception as e:
            # Log error and add to conversation
            error_msg = f"Error: {str(e)}"
            conversation.add_message(
                Message(role="system", content=error_msg)
            )
            raise
    
    def clear_conversation(self, conversation_id: str) -> None:
        """Clear a specific conversation"""
        if conversation_id in self.active_conversations:
            del self.active_conversations[conversation_id]
    
    def get_conversation_summary(self, conversation_id: str) -> Optional[str]:
        """Get summary of a conversation"""
        if conversation_id in self.active_conversations:
            return self.active_conversations[conversation_id].summarize()
        return None

class RateLimiter:
    """Rate limiting for API calls"""
    
    def __init__(self, requests_per_minute: int):
        self.requests_per_minute = requests_per_minute
        self.request_times: List[float] = []
    
    async def acquire(self) -> None:
        """Wait if necessary to respect rate limit"""
        now = asyncio.get_event_loop().time()
        
        # Remove old requests
        minute_ago = now - 60
        self.request_times = [t for t in self.request_times if t > minute_ago]
        
        # Check if we need to wait
        if len(self.request_times) >= self.requests_per_minute:
            sleep_time = 60 - (now - self.request_times[0])
            if sleep_time > 0:
                await asyncio.sleep(sleep_time)
        
        self.request_times.append(now)
```

**What to understand:** The Python AI service shows a sophisticated singleton pattern with async initialization. The conversation memory is instance-based, allowing multiple independent conversations. The singleton manages shared resources (API key, rate limiter) while instances handle conversation state. Rate limiting at the service level prevents API quota exhaustion. The async lock ensures thread-safe initialization. This hybrid approach balances global resource management with per-conversation isolation.

### TypeScript AI Service Patterns

```typescript
interface Message {
    role: 'user' | 'assistant' | 'system';
    content: string;
    timestamp: Date;
    metadata?: Record<string, any>;
}

interface ConversationSummary {
    messageCount: number;
    startTime: Date;
    lastMessageTime: Date;
    userMessageCount: number;
    topics?: string[];
}

class ConversationMemory {
    private messages: Message[] = [];
    private totalTokens = 0;
    
    constructor(
        private readonly maxMessages: number = 50,
        private readonly maxTokens: number = 4000
    ) {}
    
    addMessage(message: Omit<Message, 'timestamp'>): void {
        const fullMessage: Message = {
            ...message,
            timestamp: new Date()
        };
        
        this.messages.push(fullMessage);
        
        // Estimate tokens
        const messageTokens = this.estimateTokens(message.content);
        this.totalTokens += messageTokens;
        
        // Trim if needed
        while (
            (this.messages.length > this.maxMessages || 
             this.totalTokens > this.maxTokens) && 
            this.messages.length > 1
        ) {
            const removed = this.messages.shift()!;
            this.totalTokens -= this.estimateTokens(removed.content);
        }
    }
    
    private estimateTokens(text: string): number {
        // Rough estimation: ~4 characters per token
        return Math.ceil(text.length / 4);
    }
    
    getContext(systemPrompt?: string): Array<{role: string; content: string}> {
        const context: Array<{role: string; content: string}> = [];
        
        if (systemPrompt) {
            context.push({ role: 'system', content: systemPrompt });
        }
        
        for (const message of this.messages) {
            context.push({
                role: message.role,
                content: message.content
            });
        }
        
        return context;
    }
    
    getSummary(): ConversationSummary {
        const userMessages = this.messages.filter(m => m.role === 'user');
        
        return {
            messageCount: this.messages.length,
            startTime: this.messages[0]?.timestamp || new Date(),
            lastMessageTime: this.messages[this.messages.length - 1]?.timestamp || new Date(),
            userMessageCount: userMessages.length
        };
    }
    
    clear(): void {
        this.messages = [];
        this.totalTokens = 0;
    }
}

class AIAgentService {
    private static instance: AIAgentService;
    private conversations = new Map<string, ConversationMemory>();
    private rateLimiter: RateLimiter;
    private initialized = false;
    
    private constructor(
        private apiKey: string,
        private defaultModel: string = 'gpt-4'
    ) {
        this.rateLimiter = new RateLimiter(60); // 60 requests per minute
    }
    
    static initialize(apiKey: string, defaultModel?: string): void {
        if (this.instance) {
            throw new Error('AIAgentService already initialized');
        }
        this.instance = new AIAgentService(apiKey, defaultModel);
    }
    
    static getInstance(): AIAgentService {
        if (!this.instance) {
            throw new Error('AIAgentService not initialized');
        }
        return this.instance;
    }
    
    getOrCreateConversation(conversationId: string): ConversationMemory {
        if (!this.conversations.has(conversationId)) {
            this.conversations.set(conversationId, new ConversationMemory());
        }
        return this.conversations.get(conversationId)!;
    }
    
    async sendMessage(
        conversationId: string,
        message: string,
        options: {
            systemPrompt?: string;
            model?: string;
            temperature?: number;
            maxTokens?: number;
        } = {}
    ): Promise<string> {
        // Rate limiting
        await this.rateLimiter.acquire();
        
        // Get conversation
        const conversation = this.getOrCreateConversation(conversationId);
        
        // Add user message
        conversation.addMessage({ role: 'user', content: message });
        
        // Get context
        const context = conversation.getContext(options.systemPrompt);
        
        try {
            // Make API call (using fetch as example)
            const response = await fetch('https://api.openai.com/v1/chat/completions', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${this.apiKey}`
                },
                body: JSON.stringify({
                    model: options.model || this.defaultModel,
                    messages: context,
                    temperature: options.temperature || 0.7,
                    max_tokens: options.maxTokens
                })
            });
            
            if (!response.ok) {
                throw new Error(`API error: ${response.statusText}`);
            }
            
            const data = await response.json();
            const assistantMessage = data.choices[0].message.content;
            
            // Add assistant response
            conversation.addMessage({
                role: 'assistant',
                content: assistantMessage,
                metadata: { model: options.model || this.defaultModel }
            });
            
            return assistantMessage;
            
        } catch (error) {
            // Log error
            const errorMessage = `Error: ${error.message}`;
            conversation.addMessage({
                role: 'system',
                content: errorMessage
            });
            throw error;
        }
    }
    
    async streamMessage(
        conversationId: string,
        message: string,
        onChunk: (chunk: string) => void,
        options: {
            systemPrompt?: string;
            model?: string;
            temperature?: number;
        } = {}
    ): Promise<void> {
        // Implementation for streaming responses
        // Similar to sendMessage but handles SSE or WebSocket streams
    }
    
    clearConversation(conversationId: string): void {
        this.conversations.delete(conversationId);
    }
    
    getConversationSummary(conversationId: string): ConversationSummary | null {
        const conversation = this.conversations.get(conversationId);
        return conversation ? conversation.getSummary() : null;
    }
    
    getAllConversations(): string[] {
        return Array.from(this.conversations.keys());
    }
}

class RateLimiter {
    private requestTimes: number[] = [];
    
    constructor(private requestsPerMinute: number) {}
    
    async acquire(): Promise<void> {
        const now = Date.now();
        const minuteAgo = now - 60000;
        
        // Remove old requests
        this.requestTimes = this.requestTimes.filter(time => time > minuteAgo);
        
        // Check if we need to wait
        if (this.requestTimes.length >= this.requestsPerMinute) {
            const oldestRequest = this.requestTimes[0];
            const waitTime = 60000 - (now - oldestRequest);
            
            if (waitTime > 0) {
                await new Promise(resolve => setTimeout(resolve, waitTime));
            }
        }
        
        this.requestTimes.push(Date.now());
    }
}

// Usage
AIAgentService.initialize('your-api-key');
const ai = AIAgentService.getInstance();

async function chatExample() {
    const conversationId = 'user-123';
    
    const response = await ai.sendMessage(
        conversationId,
        'What is the weather like today?',
        {
            systemPrompt: 'You are a helpful weather assistant.',
            temperature: 0.5
        }
    );
    
    console.log('AI Response:', response);
}
```

**What to understand:** TypeScript's AI service demonstrates explicit initialization and singleton management. Conversation memory instances track state per conversation while the singleton manages API access. The rate limiter uses timestamps for precise control. Streaming support (via `streamMessage`) shows how instance methods can handle complex stateful operations. The type-safe message structure ensures consistency. This pattern works well for services that need both global configuration and per-user state.

## Utility and Helper Services

### Python Utility Patterns

```python
import hashlib
import json
from datetime import datetime, timezone
from typing import Any, Dict, List, Optional, Union
import re

class StringUtils:
    """Static string manipulation utilities"""
    
    @staticmethod
    def slugify(text: str) -> str:
        """Convert text to URL-friendly slug"""
        # Convert to lowercase and replace spaces
        slug = text.lower().strip()
        slug = re.sub(r'[^\w\s-]', '', slug)
        slug = re.sub(r'[-\s]+', '-', slug)
        return slug.strip('-')
    
    @staticmethod
    def truncate(text: str, length: int, suffix: str = '...') -> str:
        """Truncate text to specified length"""
        if len(text) <= length:
            return text
        return text[:length - len(suffix)].rsplit(' ', 1)[0] + suffix
    
    @staticmethod
    def extract_urls(text: str) -> List[str]:
        """Extract all URLs from text"""
        url_pattern = r'https?://(?:[-\w.]|(?:%[\da-fA-F]{2}))+'
        return re.findall(url_pattern, text)
    
    @staticmethod
    def remove_html_tags(html: str) -> str:
        """Remove HTML tags from string"""
        clean = re.compile('<.*?>')
        return re.sub(clean, '', html)
    
    @staticmethod
    def camel_to_snake(name: str) -> str:
        """Convert camelCase to snake_case"""
        s1 = re.sub('(.)([A-Z][a-z]+)', r'\1_\2', name)
        return re.sub('([a-z0-9])([A-Z])', r'\1_\2', s1).lower()

class DateTimeUtils:
    """Static datetime utilities"""
    
    @staticmethod
    def to_iso8601(dt: datetime) -> str:
        """Convert datetime to ISO 8601 string"""
        if dt.tzinfo is None:
            dt = dt.replace(tzinfo=timezone.utc)
        return dt.isoformat()
    
    @staticmethod
    def from_iso8601(iso_string: str) -> datetime:
        """Parse ISO 8601 string to datetime"""
        return datetime.fromisoformat(iso_string.replace('Z', '+00:00'))
    
    @staticmethod
    def human_readable_duration(seconds: float) -> str:
        """Convert seconds to human readable duration"""
        if seconds < 60:
            return f"{seconds:.1f} seconds"
        elif seconds < 3600:
            minutes = seconds / 60
            return f"{minutes:.1f} minutes"
        elif seconds < 86400:
            hours = seconds / 3600
            return f"{hours:.1f} hours"
        else:
            days = seconds / 86400
            return f"{days:.1f} days"
    
    @staticmethod
    def time_ago(dt: datetime) -> str:
        """Get human readable time ago string"""
        now = datetime.now(timezone.utc)
        if dt.tzinfo is None:
            dt = dt.replace(tzinfo=timezone.utc)
        
        diff = now - dt
        seconds = diff.total_seconds()
        
        if seconds < 60:
            return "just now"
        elif seconds < 3600:
            minutes = int(seconds / 60)
            return f"{minutes} minute{'s' if minutes != 1 else ''} ago"
        elif seconds < 86400:
            hours = int(seconds / 3600)
            return f"{hours} hour{'s' if hours != 1 else ''} ago"
        else:
            days = int(seconds / 86400)
            return f"{days} day{'s' if days != 1 else ''} ago"

class CryptoUtils:
    """Static cryptographic utilities"""
    
    @staticmethod
    def hash_password(password: str, salt: Optional[str] = None) -> Dict[str, str]:
        """Hash password with salt"""
        if salt is None:
            salt = hashlib.sha256(os.urandom(60)).hexdigest()
        
        pwdhash = hashlib.pbkdf2_hmac(
            'sha256',
            password.encode('utf-8'),
            salt.encode('utf-8'),
            100000
        )
        
        return {
            'salt': salt,
            'hash': pwdhash.hex()
        }
    
    @staticmethod
    def verify_password(password: str, salt: str, hash_str: str) -> bool:
        """Verify password against hash"""
        pwdhash = hashlib.pbkdf2_hmac(
            'sha256',
            password.encode('utf-8'),
            salt.encode('utf-8'),
            100000
        )
        return pwdhash.hex() == hash_str
    
    @staticmethod
    def generate_token(length: int = 32) -> str:
        """Generate secure random token"""
        return secrets.token_urlsafe(length)
    
    @staticmethod
    def hash_file(filepath: str, algorithm: str = 'sha256') -> str:
        """Calculate file hash"""
        hash_func = getattr(hashlib, algorithm)()
        
        with open(filepath, 'rb') as f:
            for chunk in iter(lambda: f.read(4096), b''):
                hash_func.update(chunk)
        
        return hash_func.hexdigest()

class DataTransformer:
    """Instance-based data transformation service"""
    
    def __init__(self, config: Optional[Dict[str, Any]] = None):
        self.config = config or {}
        self.transformations = []
    
    def add_transformation(self, name: str, func: callable) -> 'DataTransformer':
        """Add a transformation function"""
        self.transformations.append((name, func))
        return self
    
    def transform(self, data: Any) -> Any:
        """Apply all transformations to data"""
        result = data
        for name, func in self.transformations:
            try:
                result = func(result)
            except Exception as e:
                raise TransformationError(f"Transformation '{name}' failed: {e}")
        return result
    
    def flatten_dict(self, d: Dict[str, Any], parent_key: str = '', sep: str = '.') -> Dict[str, Any]:
        """Flatten nested dictionary"""
        items = []
        for k, v in d.items():
            new_key = f"{parent_key}{sep}{k}" if parent_key else k
            if isinstance(v, dict):
                items.extend(self.flatten_dict(v, new_key, sep=sep).items())
            else:
                items.append((new_key, v))
        return dict(items)
    
    def unflatten_dict(self, d: Dict[str, Any], sep: str = '.') -> Dict[str, Any]:
        """Unflatten dictionary"""
        result = {}
        for key, value in d.items():
            parts = key.split(sep)
            current = result
            for part in parts[:-1]:
                if part not in current:
                    current[part] = {}
                current = current[part]
            current[parts[-1]] = value
        return result
```

**What to understand:** Python utilities show the static vs instance decision clearly. Static methods (`StringUtils`, `DateTimeUtils`, `CryptoUtils`) handle pure transformations with no state. These could be plain functions but grouping in classes provides organization. The instance-based `DataTransformer` maintains transformation pipelines, showing when state matters. The builder pattern (`add_transformation`) enables fluent interfaces. Choose static for universal utilities, instances for configurable processors.

### TypeScript Utility Patterns

```typescript
class StringUtils {
    /**
     * Convert text to URL-friendly slug
     */
    static slugify(text: string): string {
        return text
            .toLowerCase()
            .trim()
            .replace(/[^\w\s-]/g, '')
            .replace(/[\s_-]+/g, '-')
            .replace(/^-+|-+$/g, '');
    }
    
    /**
     * Truncate text to specified length
     */
    static truncate(text: string, length: number, suffix = '...'): string {
        if (text.length <= length) {
            return text;
        }
        
        const truncated = text.substring(0, length - suffix.length);
        const lastSpace = truncated.lastIndexOf(' ');
        
        return (lastSpace > 0 ? truncated.substring(0, lastSpace) : truncated) + suffix;
    }
    
    /**
     * Extract all URLs from text
     */
    static extractUrls(text: string): string[] {
        const urlRegex = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g;
        return text.match(urlRegex) || [];
    }
    
    /**
     * Convert camelCase to snake_case
     */
    static camelToSnake(str: string): string {
        return str.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
    }
    
    /**
     * Convert snake_case to camelCase
     */
    static snakeToCamel(str: string): string {
        return str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());
    }
    
    /**
     * Generate random string
     */
    static randomString(length: number, charset = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'): string {
        let result = '';
        for (let i = 0; i < length; i++) {
            result += charset.charAt(Math.floor(Math.random() * charset.length));
        }
        return result;
    }
}

class DateTimeUtils {
    /**
     * Format date to ISO 8601
     */
    static toISO8601(date: Date): string {
        return date.toISOString();
    }
    
    /**
     * Parse ISO 8601 string
     */
    static fromISO8601(isoString: string): Date {
        return new Date(isoString);
    }
    
    /**
     * Get human readable duration
     */
    static humanReadableDuration(milliseconds: number): string {
        const seconds = milliseconds / 1000;
        
        if (seconds < 60) {
            return `${seconds.toFixed(1)} seconds`;
        } else if (seconds < 3600) {
            const minutes = seconds / 60;
            return `${minutes.toFixed(1)} minutes`;
        } else if (seconds < 86400) {
            const hours = seconds / 3600;
            return `${hours.toFixed(1)} hours`;
        } else {
            const days = seconds / 86400;
            return `${days.toFixed(1)} days`;
        }
    }
    
    /**
     * Get time ago string
     */
    static timeAgo(date: Date): string {
        const now = new Date();
        const secondsAgo = Math.floor((now.getTime() - date.getTime()) / 1000);
        
        if (secondsAgo < 60) {
            return 'just now';
        } else if (secondsAgo < 3600) {
            const minutes = Math.floor(secondsAgo / 60);
            return `${minutes} minute${minutes !== 1 ? 's' : ''} ago`;
        } else if (secondsAgo < 86400) {
            const hours = Math.floor(secondsAgo / 3600);
            return `${hours} hour${hours !== 1 ? 's' : ''} ago`;
        } else if (secondsAgo < 2592000) {
            const days = Math.floor(secondsAgo / 86400);
            return `${days} day${days !== 1 ? 's' : ''} ago`;
        } else {
            const months = Math.floor(secondsAgo / 2592000);
            return `${months} month${months !== 1 ? 's' : ''} ago`;
        }
    }
    
    /**
     * Add business days to date
     */
    static addBusinessDays(date: Date, days: number): Date {
        const result = new Date(date);
        let daysAdded = 0;
        
        while (daysAdded < days) {
            result.setDate(result.getDate() + 1);
            
            // Skip weekends
            if (result.getDay() !== 0 && result.getDay() !== 6) {
                daysAdded++;
            }
        }
        
        return result;
    }
}

class ArrayUtils {
    /**
     * Chunk array into smaller arrays
     */
    static chunk<T>(array: T[], size: number): T[][] {
        const chunks: T[][] = [];
        for (let i = 0; i < array.length; i += size) {
            chunks.push(array.slice(i, i + size));
        }
        return chunks;
    }
    
    /**
     * Get unique values from array
     */
    static unique<T>(array: T[]): T[] {
        return [...new Set(array)];
    }
    
    /**
     * Group array by key
     */
    static groupBy<T>(array: T[], key: keyof T): Record<string, T[]> {
        return array.reduce((groups, item) => {
            const groupKey = String(item[key]);
            if (!groups[groupKey]) {
                groups[groupKey] = [];
            }
            groups[groupKey].push(item);
            return groups;
        }, {} as Record<string, T[]>);
    }
    
    /**
     * Shuffle array
     */
    static shuffle<T>(array: T[]): T[] {
        const shuffled = [...array];
        for (let i = shuffled.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
        }
        return shuffled;
    }
}

class ObjectUtils {
    /**
     * Deep clone object
     */
    static deepClone<T>(obj: T): T {
        if (obj === null || typeof obj !== 'object') {
            return obj;
        }
        
        if (obj instanceof Date) {
            return new Date(obj.getTime()) as any;
        }
        
        if (obj instanceof Array) {
            return obj.map(item => this.deepClone(item)) as any;
        }
        
        if (obj instanceof Object) {
            const clonedObj = {} as T;
            for (const key in obj) {
                if (obj.hasOwnProperty(key)) {
                    clonedObj[key] = this.deepClone(obj[key]);
                }
            }
            return clonedObj;
        }
        
        return obj;
    }
    
    /**
     * Deep merge objects
     */
    static deepMerge<T extends object>(target: T, ...sources: Partial<T>[]): T {
        if (!sources.length) return target;
        
        const source = sources.shift();
        
        if (this.isObject(target) && this.isObject(source)) {
            for (const key in source) {
                if (this.isObject(source[key])) {
                    if (!target[key]) Object.assign(target, { [key]: {} });
                    this.deepMerge(target[key] as any, source[key] as any);
                } else {
                    Object.assign(target, { [key]: source[key] });
                }
            }
        }
        
        return this.deepMerge(target, ...sources);
    }
    
    private static isObject(item: any): boolean {
        return item && typeof item === 'object' && !Array.isArray(item);
    }
    
    /**
     * Pick properties from object
     */
    static pick<T, K extends keyof T>(obj: T, keys: K[]): Pick<T, K> {
        const result = {} as Pick<T, K>;
        keys.forEach(key => {
            if (key in obj) {
                result[key] = obj[key];
            }
        });
        return result;
    }
    
    /**
     * Omit properties from object
     */
    static omit<T, K extends keyof T>(obj: T, keys: K[]): Omit<T, K> {
        const result = { ...obj };
        keys.forEach(key => {
            delete result[key];
        });
        return result;
    }
}

// Instance-based transformation service
class DataTransformer<T = any> {
    private transformations: Array<(data: T) => T> = [];
    
    addTransformation(transform: (data: T) => T): this {
        this.transformations.push(transform);
        return this;
    }
    
    transform(data: T): T {
        return this.transformations.reduce((acc, transform) => transform(acc), data);
    }
    
    static flatten(obj: any, prefix = '', separator = '.'): Record<string, any> {
        const flattened: Record<string, any> = {};
        
        for (const key in obj) {
            if (obj.hasOwnProperty(key)) {
                const newKey = prefix ? `${prefix}${separator}${key}` : key;
                
                if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
                    Object.assign(flattened, this.flatten(obj[key], newKey, separator));
                } else {
                    flattened[newKey] = obj[key];
                }
            }
        }
        
        return flattened;
    }
    
    static unflatten(obj: Record<string, any>, separator = '.'): any {
        const unflattened: any = {};
        
        for (const key in obj) {
            const keys = key.split(separator);
            let current = unflattened;
            
            for (let i = 0; i < keys.length - 1; i++) {
                if (!current[keys[i]]) {
                    current[keys[i]] = {};
                }
                current = current[keys[i]];
            }
            
            current[keys[keys.length - 1]] = obj[key];
        }
        
        return unflattened;
    }
}
```

**What to understand:** TypeScript utilities follow functional programming patterns. Static methods provide pure functions organized by domain. The generic `DataTransformer<T>` shows type-safe transformation pipelines. Static methods like `deepClone` and `deepMerge` handle complex operations without state. The instance transformer allows chaining operations, useful for data processing pipelines. TypeScript's type system ensures transformations maintain type safety throughout the chain.

## API Client Patterns

### Python API Client Patterns

```python
import aiohttp
import requests
from typing import Optional, Dict, Any, List
from urllib.parse import urljoin
import backoff
from functools import wraps

class APIClient:
    """Base API client with common functionality"""
    
    def __init__(self, base_url: str, timeout: int = 30):
        self.base_url = base_url.rstrip('/')
        self.timeout = timeout
        self.session = requests.Session()
        self._setup_session()
    
    def _setup_session(self):
        """Configure session with retry logic"""
        from requests.adapters import HTTPAdapter
        from requests.packages.urllib3.util.retry import Retry
        
        retry_strategy = Retry(
            total=3,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
        )
        
        adapter = HTTPAdapter(max_retries=retry_strategy)
        self.session.mount("http://", adapter)
        self.session.mount("https://", adapter)
    
    def _build_url(self, endpoint: str) -> str:
        """Build full URL from endpoint"""
        return urljoin(self.base_url + '/', endpoint.lstrip('/'))
    
    @backoff.on_exception(
        backoff.expo,
        requests.exceptions.RequestException,
        max_tries=3
    )
    def request(
        self,
        method: str,
        endpoint: str,
        **kwargs
    ) -> requests.Response:
        """Make HTTP request with retry logic"""
        url = self._build_url(endpoint)
        
        # Set default timeout
        kwargs.setdefault('timeout', self.timeout)
        
        response = self.session.request(method, url, **kwargs)
        response.raise_for_status()
        
        return response
    
    def get(self, endpoint: str, **kwargs) -> Dict[str, Any]:
        """GET request"""
        return self.request('GET', endpoint, **kwargs).json()
    
    def post(self, endpoint: str, data: Optional[Dict] = None, **kwargs) -> Dict[str, Any]:
        """POST request"""
        return self.request('POST', endpoint, json=data, **kwargs).json()
    
    def put(self, endpoint: str, data: Optional[Dict] = None, **kwargs) -> Dict[str, Any]:
        """PUT request"""
        return self.request('PUT', endpoint, json=data, **kwargs).json()
    
    def delete(self, endpoint: str, **kwargs) -> None:
        """DELETE request"""
        self.request('DELETE', endpoint, **kwargs)
    
    def close(self):
        """Close session"""
        self.session.close()
    
    def __enter__(self):
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.close()

class AsyncAPIClient:
    """Async API client for high-performance requests"""
    
    def __init__(self, base_url: str, timeout: int = 30):
        self.base_url = base_url.rstrip('/')
        self.timeout = aiohttp.ClientTimeout(total=timeout)
        self.session: Optional[aiohttp.ClientSession] = None
    
    async def __aenter__(self):
        self.session = aiohttp.ClientSession(timeout=self.timeout)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()
    
    def _build_url(self, endpoint: str) -> str:
        """Build full URL from endpoint"""
        return urljoin(self.base_url + '/', endpoint.lstrip('/'))
    
    async def request(
        self,
        method: str,
        endpoint: str,
        **kwargs
    ) -> Dict[str, Any]:
        """Make async HTTP request"""
        if not self.session:
            raise RuntimeError("Session not initialized. Use async with statement.")
        
        url = self._build_url(endpoint)
        
        async with self.session.request(method, url, **kwargs) as response:
            response.raise_for_status()
            return await response.json()
    
    async def get(self, endpoint: str, **kwargs) -> Dict[str, Any]:
        """Async GET request"""
        return await self.request('GET', endpoint, **kwargs)
    
    async def post(self, endpoint: str, data: Optional[Dict] = None, **kwargs) -> Dict[str, Any]:
        """Async POST request"""
        return await self.request('POST', endpoint, json=data, **kwargs)
    
    async def get_many(self, endpoints: List[str]) -> List[Dict[str, Any]]:
        """Fetch multiple endpoints concurrently"""
        tasks = [self.get(endpoint) for endpoint in endpoints]
        return await asyncio.gather(*tasks)

# Specific API client example
class GitHubAPIClient(APIClient):
    """GitHub API client with authentication"""
    
    def __init__(self, token: Optional[str] = None):
        super().__init__('https://api.github.com')
        if token:
            self.session.headers['Authorization'] = f'token {token}'
        self.session.headers['Accept'] = 'application/vnd.github.v3+json'
    
    def get_user(self, username: str) -> Dict[str, Any]:
        """Get user information"""
        return self.get(f'/users/{username}')
    
    def get_user_repos(self, username: str, per_page: int = 30) -> List[Dict[str, Any]]:
        """Get user repositories"""
        return self.get(f'/users/{username}/repos', params={'per_page': per_page})
    
    def create_issue(self, owner: str, repo: str, title: str, body: str) -> Dict[str, Any]:
        """Create an issue"""
        return self.post(
            f'/repos/{owner}/{repo}/issues',
            data={'title': title, 'body': body}
        )
    
    def paginate(self, endpoint: str, per_page: int = 100) -> List[Dict[str, Any]]:
        """Handle pagination automatically"""
        results = []
        page = 1
        
        while True:
            response = self.session.get(
                self._build_url(endpoint),
                params={'page': page, 'per_page': per_page}
            )
            response.raise_for_status()
            
            data = response.json()
            if not data:
                break
            
            results.extend(data)
            
            # Check if there are more pages
            if 'next' not in response.links:
                break
            
            page += 1
        
        return results
```

**What to understand:** Python API clients show instance-based patterns for flexibility. The base `APIClient` handles common functionality (retries, sessions) while subclasses add API-specific methods. The `@backoff` decorator provides exponential backoff for transient failures. Session reuse improves performance. The context manager pattern ensures cleanup. Pagination handling in `paginate()` shows how instance methods can maintain state across multiple requests. This pattern enables different configurations per API endpoint.

### TypeScript API Client Patterns

```typescript
interface RequestConfig {
    headers?: Record<string, string>;
    params?: Record<string, any>;
    timeout?: number;
    retries?: number;
}

interface APIResponse<T = any> {
    data: T;
    status: number;
    headers: Headers;
}

class APIError extends Error {
    constructor(
        message: string,
        public status: number,
        public response?: any
    ) {
        super(message);
        this.name = 'APIError';
    }
}

class APIClient {
    private defaultHeaders: Record<string, string> = {
        'Content-Type': 'application/json'
    };
    
    constructor(
        private readonly baseURL: string,
        private readonly defaultTimeout: number = 30000
    ) {
        this.baseURL = baseURL.replace(/\/$/, '');
    }
    
    private buildURL(endpoint: string, params?: Record<string, any>): string {
        const url = new URL(endpoint, this.baseURL);
        
        if (params) {
            Object.entries(params).forEach(([key, value]) => {
                if (value !== undefined && value !== null) {
                    url.searchParams.append(key, String(value));
                }
            });
        }
        
        return url.toString();
    }
    
    private async retry<T>(
        fn: () => Promise<T>,
        retries: number = 3,
        delay: number = 1000
    ): Promise<T> {
        try {
            return await fn();
        } catch (error) {
            if (retries <= 0) throw error;
            
            await new Promise(resolve => setTimeout(resolve, delay));
            return this.retry(fn, retries - 1, delay * 2);
        }
    }
    
    async request<T = any>(
        method: string,
        endpoint: string,
        data?: any,
        config: RequestConfig = {}
    ): Promise<APIResponse<T>> {
        const url = this.buildURL(endpoint, config.params);
        const headers = {
            ...this.defaultHeaders,
            ...config.headers
        };
        
        const requestOptions: RequestInit = {
            method,
            headers,
            signal: AbortSignal.timeout(config.timeout || this.defaultTimeout)
        };
        
        if (data && ['POST', 'PUT', 'PATCH'].includes(method)) {
            requestOptions.body = JSON.stringify(data);
        }
        
        const makeRequest = async (): Promise<Response> => {
            const response = await fetch(url, requestOptions);
            
            if (!response.ok) {
                const errorData = await response.text();
                throw new APIError(
                    `Request failed: ${response.statusText}`,
                    response.status,
                    errorData
                );
            }
            
            return response;
        };
        
        const response = await this.retry(
            makeRequest,
            config.retries || 3
        );
        
        const responseData = await response.json();
        
        return {
            data: responseData,
            status: response.status,
            headers: response.headers
        };
    }
    
    async get<T = any>(endpoint: string, config?: RequestConfig): Promise<T> {
        const response = await this.request<T>('GET', endpoint, null, config);
        return response.data;
    }
    
    async post<T = any>(endpoint: string, data?: any, config?: RequestConfig): Promise<T> {
        const response = await this.request<T>('POST', endpoint, data, config);
        return response.data;
    }
    
    async put<T = any>(endpoint: string, data?: any, config?: RequestConfig): Promise<T> {
        const response = await this.request<T>('PUT', endpoint, data, config);
        return response.data;
    }
    
    async delete<T = any>(endpoint: string, config?: RequestConfig): Promise<T> {
        const response = await this.request<T>('DELETE', endpoint, null, config);
        return response.data;
    }
    
    async batch<T extends readonly unknown[]>(
        requests: {
            [K in keyof T]: () => Promise<T[K]>
        }
    ): Promise<T> {
        return Promise.all(requests) as Promise<T>;
    }
    
    setDefaultHeader(key: string, value: string): void {
        this.defaultHeaders[key] = value;
    }
    
    removeDefaultHeader(key: string): void {
        delete this.defaultHeaders[key];
    }
}

// Specific API implementation
interface GitHubUser {
    login: string;
    id: number;
    avatar_url: string;
    html_url: string;
    name: string;
    bio: string;
}

interface GitHubRepo {
    id: number;
    name: string;
    full_name: string;
    description: string;
    private: boolean;
    html_url: string;
}

class GitHubAPIClient extends APIClient {
    constructor(token?: string) {
        super('https://api.github.com');
        
        this.setDefaultHeader('Accept', 'application/vnd.github.v3+json');
        
        if (token) {
            this.setDefaultHeader('Authorization', `token ${token}`);
        }
    }
    
    async getUser(username: string): Promise<GitHubUser> {
        return this.get<GitHubUser>(`/users/${username}`);
    }
    
    async getUserRepos(username: string, options?: {
        per_page?: number;
        page?: number;
        sort?: 'created' | 'updated' | 'pushed' | 'full_name';
    }): Promise<GitHubRepo[]> {
        return this.get<GitHubRepo[]>(`/users/${username}/repos`, {
            params: options
        });
    }
    
    async createIssue(
        owner: string,
        repo: string,
        issue: {
            title: string;
            body?: string;
            labels?: string[];
            assignees?: string[];
        }
    ): Promise<any> {
        return this.post(`/repos/${owner}/${repo}/issues`, issue);
    }
    
    async *paginate<T>(
        endpoint: string,
        perPage: number = 100
    ): AsyncGenerator<T[], void, unknown> {
        let page = 1;
        
        while (true) {
            const data = await this.get<T[]>(endpoint, {
                params: { page, per_page: perPage }
            });
            
            if (data.length === 0) break;
            
            yield data;
            
            if (data.length < perPage) break;
            
            page++;
        }
    }
    
    async getAllPages<T>(endpoint: string, perPage: number = 100): Promise<T[]> {
        const allData: T[] = [];
        
        for await (const page of this.paginate<T>(endpoint, perPage)) {
            allData.push(...page);
        }
        
        return allData;
    }
}

// Usage with interceptors
class APIClientWithInterceptors extends APIClient {
    private requestInterceptors: Array<(config: any) => any> = [];
    private responseInterceptors: Array<(response: any) => any> = [];
    
    addRequestInterceptor(interceptor: (config: any) => any): void {
        this.requestInterceptors.push(interceptor);
    }
    
    addResponseInterceptor(interceptor: (response: any) => any): void {
        this.responseInterceptors.push(interceptor);
    }
    
    async request<T = any>(
        method: string,
        endpoint: string,
        data?: any,
        config: RequestConfig = {}
    ): Promise<APIResponse<T>> {
        // Apply request interceptors
        let finalConfig = config;
        for (const interceptor of this.requestInterceptors) {
            finalConfig = interceptor(finalConfig);
        }
        
        // Make request
        let response = await super.request<T>(method, endpoint, data, finalConfig);
        
        // Apply response interceptors
        for (const interceptor of this.responseInterceptors) {
            response = interceptor(response);
        }
        
        return response;
    }
}
```

**What to understand:** TypeScript API clients leverage modern JavaScript features. The base class provides common functionality while maintaining type safety. Async generators (`paginate`) enable memory-efficient pagination. The interceptor pattern shows advanced instance-based design - modify requests/responses without changing core logic. AbortSignal.timeout provides native timeout support. Type-safe batch requests ensure compile-time correctness. This pattern supports complex API interactions while maintaining clean separation of concerns.

## Key Real-World Takeaways

> [!important] Real-World Pattern Selection
> 1. **Database Services**: Use singleton for connection pools, instances for repositories
> 2. **Validation**: Static methods for pure validation, instances for stateful checks
> 3. **AI Services**: Singleton for API management, instances for conversation state
> 4. **Utilities**: Static for pure functions, instances for configurable transformers
> 5. **API Clients**: Base class with instances for specific APIs
> 6. **Caching**: Singleton for global cache, instances for scoped caches
> 7. **Authentication**: Singleton for token management, instances for user sessions

[[Static vs Instance vs Singleton - Part 5 - Advanced Patterns and Performance|Next: Part 5 - Advanced Patterns & Performance →]]