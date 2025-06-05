# Redis Beginner's Bible: Caching & WebSockets

## 📚 Table of Contents
1. [[#Redis Fundamentals|Redis Fundamentals]]
2. [[#Installation & Setup|Installation & Setup]]
3. [[#Basic Commands|Basic Commands]]
4. [[#Redis for Caching|Redis for Caching]]
5. [[#Redis for WebSockets|Redis for WebSockets]]
6. [[#Data Types & Use Cases|Data Types & Use Cases]]
7. [[#Best Practices|Best Practices]]
8. [[#Common Patterns|Common Patterns]]
9. [[#Troubleshooting|Troubleshooting]]

---

## 🎯 Redis Fundamentals

### What is Redis?
- **RE**mote **DI**ctionary **S**erver
- In-memory data structure store
- Key-value database with advanced data types
- Blazingly fast (sub-millisecond latency)
- Persistent storage options available

### Why Redis for Caching & WebSockets?
```
✅ Speed: All data in memory
✅ Pub/Sub: Built-in messaging system
✅ Expiration: Automatic cache invalidation
✅ Atomic Operations: Thread-safe operations
✅ Clustering: Horizontal scaling support
```

---

## 🛠️ Installation & Setup

### Docker (Recommended for Development)
```bash
# Pull and run Redis
docker run -d --name redis-server -p 6379:6379 redis:7-alpine

# Connect to Redis CLI
docker exec -it redis-server redis-cli
```

### Local Installation
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install redis-server

# macOS
brew install redis

# Start Redis server
redis-server

# Connect to Redis CLI
redis-cli
```

### Configuration Basics
```bash
# redis.conf key settings
maxmemory 256mb
maxmemory-policy allkeys-lru
save 900 1  # Persistence settings
```

---

## 🔧 Basic Commands

### Essential Commands Cheat Sheet
```bash
# Connection & Info
PING                    # Test connection
INFO                    # Server information
SELECT 0               # Switch database (0-15)

# Key Operations
SET key value          # Set string value
GET key               # Get string value
EXISTS key            # Check if key exists
DEL key               # Delete key
KEYS pattern          # Find keys (avoid in production!)
EXPIRE key seconds    # Set expiration
TTL key              # Time to live

# Data Types
HSET hash field value # Hash set
HGET hash field      # Hash get
LPUSH list value     # List push left
RPOP list           # List pop right
SADD set member     # Set add
ZADD zset score member # Sorted set add
```

---

## 🚀 Redis for Caching

### Basic Caching Pattern
```python
import redis
import json
from datetime import timedelta

# Connection
r = redis.Redis(host='localhost', port=6379, db=0, decode_responses=True)

class CacheManager:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def get(self, key):
        """Get cached value"""
        try:
            value = self.redis.get(key)
            return json.loads(value) if value else None
        except Exception as e:
            print(f"Cache get error: {e}")
            return None
    
    def set(self, key, value, expire_seconds=3600):
        """Set cached value with expiration"""
        try:
            serialized = json.dumps(value)
            return self.redis.setex(key, expire_seconds, serialized)
        except Exception as e:
            print(f"Cache set error: {e}")
            return False
    
    def delete(self, key):
        """Delete cached value"""
        return self.redis.delete(key)
    
    def exists(self, key):
        """Check if key exists"""
        return self.redis.exists(key)

# Usage Example
cache = CacheManager(r)

# Cache user data
user_data = {"id": 123, "name": "John", "email": "john@example.com"}
cache.set("user:123", user_data, expire_seconds=1800)  # 30 minutes

# Retrieve cached data
cached_user = cache.get("user:123")
```

### Cache-Aside Pattern (Most Common)
```python
def get_user_profile(user_id):
    cache_key = f"user_profile:{user_id}"
    
    # Try cache first
    cached_data = cache.get(cache_key)
    if cached_data:
        return cached_data
    
    # Cache miss - fetch from database
    user_data = database.get_user(user_id)
    
    # Store in cache for next time
    cache.set(cache_key, user_data, expire_seconds=1800)
    
    return user_data

def update_user_profile(user_id, new_data):
    # Update database
    database.update_user(user_id, new_data)
    
    # Invalidate cache
    cache_key = f"user_profile:{user_id}"
    cache.delete(cache_key)
```

### Advanced Caching Strategies
```python
class AdvancedCache:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def get_or_set(self, key, fetch_function, expire_seconds=3600):
        """Get from cache or set if missing"""
        cached = self.get(key)
        if cached is not None:
            return cached
        
        # Fetch fresh data
        fresh_data = fetch_function()
        self.set(key, fresh_data, expire_seconds)
        return fresh_data
    
    def multi_get(self, keys):
        """Get multiple keys at once"""
        pipeline = self.redis.pipeline()
        for key in keys:
            pipeline.get(key)
        
        results = pipeline.execute()
        return {
            key: json.loads(value) if value else None 
            for key, value in zip(keys, results)
        }
    
    def invalidate_pattern(self, pattern):
        """Invalidate keys matching pattern (use carefully!)"""
        keys = self.redis.keys(pattern)
        if keys:
            self.redis.delete(*keys)
```

---

## 🌐 Redis for WebSockets

### Pub/Sub Fundamentals
```python
import redis
import json
import asyncio
from typing import Dict, Set

class RedisPubSub:
    def __init__(self, redis_url="redis://localhost:6379"):
        self.redis = redis.from_url(redis_url, decode_responses=True)
        self.pubsub = self.redis.pubsub()
    
    def publish(self, channel, message):
        """Publish message to channel"""
        if isinstance(message, dict):
            message = json.dumps(message)
        return self.redis.publish(channel, message)
    
    def subscribe(self, channels):
        """Subscribe to channels"""
        if isinstance(channels, str):
            channels = [channels]
        self.pubsub.subscribe(*channels)
    
    def listen(self):
        """Listen for messages"""
        for message in self.pubsub.listen():
            if message['type'] == 'message':
                try:
                    data = json.loads(message['data'])
                    yield {
                        'channel': message['channel'],
                        'data': data
                    }
                except json.JSONDecodeError:
                    yield {
                        'channel': message['channel'],
                        'data': message['data']
                    }
```

### WebSocket + Redis Integration
```python
import asyncio
import websockets
import redis.asyncio as redis
import json
from typing import Dict, Set

class WebSocketManager:
    def __init__(self):
        self.connections: Dict[str, Set[websockets.WebSocketServerProtocol]] = {}
        self.redis = None
        self.pubsub = None
    
    async def connect_redis(self):
        """Initialize Redis connection"""
        self.redis = redis.from_url("redis://localhost:6379", decode_responses=True)
        self.pubsub = self.redis.pubsub()
    
    async def add_connection(self, websocket, room_id):
        """Add WebSocket connection to room"""
        if room_id not in self.connections:
            self.connections[room_id] = set()
        self.connections[room_id].add(websocket)
        
        # Subscribe to Redis channel for this room
        await self.pubsub.subscribe(f"room:{room_id}")
    
    async def remove_connection(self, websocket, room_id):
        """Remove WebSocket connection from room"""
        if room_id in self.connections:
            self.connections[room_id].discard(websocket)
            if not self.connections[room_id]:
                del self.connections[room_id]
                await self.pubsub.unsubscribe(f"room:{room_id}")
    
    async def broadcast_to_room(self, room_id, message):
        """Broadcast message to all connections in room"""
        # Publish to Redis (for other server instances)
        await self.redis.publish(f"room:{room_id}", json.dumps(message))
        
        # Send to local connections
        if room_id in self.connections:
            disconnected = set()
            for websocket in self.connections[room_id]:
                try:
                    await websocket.send(json.dumps(message))
                except websockets.exceptions.ConnectionClosed:
                    disconnected.add(websocket)
            
            # Clean up disconnected clients
            self.connections[room_id] -= disconnected
    
    async def listen_redis_messages(self):
        """Listen for Redis pub/sub messages"""
        async for message in self.pubsub.listen():
            if message['type'] == 'message':
                channel = message['channel']
                if channel.startswith('room:'):
                    room_id = channel.split(':', 1)[1]
                    data = json.loads(message['data'])
                    
                    # Broadcast to local WebSocket connections
                    if room_id in self.connections:
                        disconnected = set()
                        for websocket in self.connections[room_id]:
                            try:
                                await websocket.send(json.dumps(data))
                            except websockets.exceptions.ConnectionClosed:
                                disconnected.add(websocket)
                        
                        self.connections[room_id] -= disconnected

# WebSocket handler
manager = WebSocketManager()

async def websocket_handler(websocket, path):
    room_id = path.strip('/')  # Extract room ID from path
    
    try:
        await manager.add_connection(websocket, room_id)
        
        async for message in websocket:
            data = json.loads(message)
            
            # Broadcast message to all clients in room
            await manager.broadcast_to_room(room_id, {
                'type': 'message',
                'data': data,
                'timestamp': asyncio.get_event_loop().time()
            })
    
    except websockets.exceptions.ConnectionClosed:
        pass
    finally:
        await manager.remove_connection(websocket, room_id)

# Start server
async def main():
    await manager.connect_redis()
    
    # Start Redis message listener
    asyncio.create_task(manager.listen_redis_messages())
    
    # Start WebSocket server
    server = await websockets.serve(websocket_handler, "localhost", 8765)
    await server.wait_closed()

if __name__ == "__main__":
    asyncio.run(main())
```

---

## 📊 Data Types & Use Cases

### String (Most Common for Caching)
```python
# Simple caching
r.set("session:abc123", "user_data", ex=3600)
r.get("session:abc123")

# Counters
r.incr("page_views:homepage")
r.incrby("downloads:file123", 5)
```

### Hash (Perfect for Objects)
```python
# User profile caching
r.hset("user:123", mapping={
    "name": "John Doe",
    "email": "john@example.com",
    "last_login": "2024-01-15"
})

r.hget("user:123", "name")
r.hgetall("user:123")  # Get entire hash
```

### List (Message Queues, Recent Items)
```python
# Recent activity feed
r.lpush("activity:user123", "logged_in")
r.lpush("activity:user123", "viewed_profile")
r.lrange("activity:user123", 0, 9)  # Get last 10 items

# Simple job queue
r.lpush("job_queue", json.dumps({"task": "send_email", "user_id": 123}))
job = r.brpop("job_queue", timeout=5)  # Blocking pop
```

### Set (Unique Collections)
```python
# Online users
r.sadd("online_users", "user123", "user456")
r.sismember("online_users", "user123")  # Check if online
r.smembers("online_users")  # Get all online users

# Tags
r.sadd("article:123:tags", "python", "redis", "caching")
```

### Sorted Set (Leaderboards, Rankings)
```python
# Leaderboard
r.zadd("leaderboard", {"player1": 1000, "player2": 1500, "player3": 800})
r.zrevrange("leaderboard", 0, 9, withscores=True)  # Top 10

# Time-based data
import time
r.zadd("user_activity", {f"user:{user_id}": time.time()})
```

---

## ✅ Best Practices

### 1. Key Naming Conventions
```python
# Good naming patterns
"user:123:profile"
"session:abc123def456"
"cache:product:456"
"queue:email_notifications"
"counter:page_views:homepage"

# Use consistent separators (:)
# Include data type hints
# Make keys self-documenting
```

### 2. Memory Management
```python
# Always set expiration for cache keys
r.setex("temp_data", 3600, value)  # 1 hour

# Use appropriate data types
# Hash for objects (more memory efficient than JSON strings)
# Sets for unique collections
# Sorted sets for rankings

# Monitor memory usage
info = r.info('memory')
print(f"Used memory: {info['used_memory_human']}")
```

### 3. Connection Management
```python
import redis
from redis.connection import ConnectionPool

# Use connection pooling
pool = ConnectionPool(
    host='localhost',
    port=6379,
    db=0,
    max_connections=20,
    decode_responses=True
)
r = redis.Redis(connection_pool=pool)

# For async applications
import redis.asyncio as redis
r = redis.from_url("redis://localhost:6379", max_connections=20)
```

### 4. Error Handling
```python
def safe_cache_operation(func):
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except redis.ConnectionError:
            print("Redis connection failed - falling back to database")
            return None
        except redis.TimeoutError:
            print("Redis timeout - operation took too long")
            return None
        except Exception as e:
            print(f"Unexpected Redis error: {e}")
            return None
    return wrapper

@safe_cache_operation
def get_cached_data(key):
    return r.get(key)
```

---

## 🔄 Common Patterns

### 1. Cache-Aside with Fallback
```python
async def get_user_data(user_id):
    cache_key = f"user:{user_id}"
    
    # Try cache first
    try:
        cached = await redis_client.get(cache_key)
        if cached:
            return json.loads(cached)
    except Exception as e:
        logger.warning(f"Cache read failed: {e}")
    
    # Fallback to database
    user_data = await database.get_user(user_id)
    
    # Try to cache for next time (fire and forget)
    try:
        await redis_client.setex(
            cache_key, 
            1800,  # 30 minutes
            json.dumps(user_data)
        )
    except Exception as e:
        logger.warning(f"Cache write failed: {e}")
    
    return user_data
```

### 2. Distributed Locking
```python
import time
import uuid

class RedisLock:
    def __init__(self, redis_client, key, timeout=10):
        self.redis = redis_client
        self.key = f"lock:{key}"
        self.timeout = timeout
        self.identifier = str(uuid.uuid4())
    
    def acquire(self):
        """Acquire distributed lock"""
        end = time.time() + self.timeout
        while time.time() < end:
            if self.redis.set(self.key, self.identifier, nx=True, ex=self.timeout):
                return True
            time.sleep(0.001)
        return False
    
    def release(self):
        """Release distributed lock"""
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        return self.redis.eval(lua_script, 1, self.key, self.identifier)

# Usage
lock = RedisLock(r, "critical_section")
if lock.acquire():
    try:
        # Critical section code
        pass
    finally:
        lock.release()
```

### 3. Rate Limiting
```python
def rate_limit(key, limit=100, window=3600):
    """Simple rate limiting using Redis"""
    current = r.get(key)
    
    if current is None:
        # First request
        r.setex(key, window, 1)
        return True
    
    if int(current) >= limit:
        return False
    
    r.incr(key)
    return True

# Usage
user_key = f"rate_limit:user:{user_id}"
if not rate_limit(user_key, limit=1000, window=3600):
    return {"error": "Rate limit exceeded"}
```

---

## 🐛 Troubleshooting

### Common Issues & Solutions

#### 1. Memory Issues
```bash
# Check memory usage
redis-cli INFO memory

# Common solutions:
# - Set maxmemory policy
# - Use appropriate data types
# - Set expiration on keys
# - Monitor key patterns
```

#### 2. Connection Problems
```python
# Test connection
try:
    r.ping()
    print("Redis connected successfully")
except redis.ConnectionError:
    print("Cannot connect to Redis")

# Check Redis server status
# redis-cli ping
```

#### 3. Performance Issues
```bash
# Monitor slow queries
redis-cli --latency-history

# Check connected clients
redis-cli CLIENT LIST

# Monitor commands
redis-cli MONITOR
```

#### 4. Debugging Commands
```bash
# Check key information
redis-cli TYPE keyname
redis-cli TTL keyname
redis-cli OBJECT ENCODING keyname

# Memory usage of specific key
redis-cli MEMORY USAGE keyname

# Find large keys
redis-cli --bigkeys
```

---

## 🎯 Quick Reference Card

### Essential Commands
```bash
# Basic Operations
SET key value EX 3600    # Set with expiration
GET key                  # Get value
DEL key                  # Delete key
EXISTS key              # Check existence
EXPIRE key 3600         # Set expiration
TTL key                 # Check time to live

# Pub/Sub
PUBLISH channel message  # Publish message
SUBSCRIBE channel       # Subscribe to channel
PSUBSCRIBE pattern*     # Pattern subscribe

# Data Types
HSET hash field value   # Hash operations
LPUSH list value       # List operations
SADD set member        # Set operations
ZADD zset score member # Sorted set operations
```

### Connection Strings
```python
# Basic connection
redis://localhost:6379/0

# With password
redis://:password@localhost:6379/0

# Redis Sentinel
redis-sentinel://localhost:26379/mymaster

# Redis Cluster
redis://localhost:7000,localhost:7001,localhost:7002
```

---

> [!tip] Pro Tips
> - Always handle Redis failures gracefully with fallback mechanisms
> - Monitor memory usage regularly in production environments
> - Use connection pooling for better performance
> - Set appropriate expiration times for all cache keys
> - Consider Redis Cluster for high availability and scaling

> [!warning] Production Considerations
> - Never use `KEYS *` in production (use `SCAN` instead)
> - Always set `maxmemory` and `maxmemory-policy`
> - Monitor Redis metrics (memory, connections, operations/sec)
> - Implement proper error handling and circuit breakers
> - Use Redis Sentinel or Cluster for high availability

> [!note] Learning Path
> 1. Start with basic caching patterns ([[#Redis for Caching|Redis for Caching]])
> 2. Learn pub/sub for real-time features ([[#Redis for WebSockets|Redis for WebSockets]])
> 3. Master different data types ([[#Data Types & Use Cases|Data Types & Use Cases]])
> 4. Implement advanced patterns ([[#Common Patterns|Common Patterns]])
> 5. Focus on production best practices ([[#Best Practices|Best Practices]])

This Redis beginner's bible covers everything you need for caching and WebSocket applications. The Obsidian-formatted links will help you navigate between sections easily, and the callout boxes highlight important information for quick reference!