# Thread Safety

#programming #concurrency #multithreading #synchronization #patterns

## What is Thread Safety?

**Thread Safety** means that code functions correctly when accessed by multiple threads simultaneously. Thread-safe code prevents race conditions, data corruption, and other concurrency issues that arise when multiple threads access shared resources.

## Computer Science Definition

A piece of code is thread-safe if it can be safely invoked concurrently by multiple threads without causing data races, maintaining its correctness invariants, and producing deterministic results. This is achieved through synchronization mechanisms, immutability, or avoiding shared state.

## Key Concepts

### Race Condition
When multiple threads access shared data and try to change it at the same time:
```python
# NOT thread-safe - race condition
counter = 0

def increment():
    global counter
    temp = counter      # Thread 1 reads: counter = 0
    # Context switch!   # Thread 2 reads: counter = 0  
    temp = temp + 1     # Thread 1: temp = 1
    # Context switch!   # Thread 2: temp = 1
    counter = temp      # Thread 1 writes: counter = 1
    # Context switch!   # Thread 2 writes: counter = 1
    # Result: counter = 1 instead of 2!
```

### Critical Section
Code that accesses shared resources and must not be executed by more than one thread at a time.

### Synchronization Primitives
- **Mutex (Lock)**: Ensures only one thread accesses a resource
- **Semaphore**: Controls access to a resource with limited capacity
- **Condition Variable**: Allows threads to wait for conditions
- **Atomic Operations**: Operations that complete in a single step

## Thread Safety in Our Patterns

### Static Methods - Usually NOT Thread-Safe
```python
class UnsafeCounter:
    count = 0  # Shared class variable
    
    @staticmethod
    def increment():
        # NOT thread-safe!
        UnsafeCounter.count += 1  # Read-modify-write

# Making it thread-safe
import threading

class SafeCounter:
    count = 0
    _lock = threading.Lock()
    
    @staticmethod
    def increment():
        with SafeCounter._lock:  # Thread-safe!
            SafeCounter.count += 1
```

### Instance Methods - Thread-Safe Per Instance
```python
class InstanceCounter:
    def __init__(self):
        self.count = 0  # Each instance has its own count
        self._lock = threading.Lock()
    
    def increment(self):
        with self._lock:
            self.count += 1  # Thread-safe for this instance
```

### Singleton - Requires Special Care
```python
# NOT thread-safe singleton
class UnsafeSingleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:  # Two threads might pass this check!
            cls._instance = super().__new__(cls)
        return cls._instance

# Thread-safe singleton
class SafeSingleton:
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:  # Only one thread can create
                if cls._instance is None:  # Double-check
                    cls._instance = super().__new__(cls)
        return cls._instance
```

## Thread Safety Strategies

### 1. **Immutability**
Immutable objects are inherently thread-safe:
```python
from typing import NamedTuple

# Thread-safe - immutable
class Point(NamedTuple):
    x: float
    y: float

# Can be shared between threads safely
shared_point = Point(10, 20)
```

### 2. **Thread Confinement**
Each thread works with its own copy:
```python
import threading

# Thread-local storage
thread_local_data = threading.local()

def process():
    # Each thread has its own value
    if not hasattr(thread_local_data, 'counter'):
        thread_local_data.counter = 0
    thread_local_data.counter += 1
```

### 3. **Synchronization**
```python
class ThreadSafeList:
    def __init__(self):
        self._list = []
        self._lock = threading.Lock()
    
    def append(self, item):
        with self._lock:
            self._list.append(item)
    
    def pop(self):
        with self._lock:
            return self._list.pop() if self._list else None
```

### 4. **Atomic Operations**
```python
import queue

# Thread-safe queue
safe_queue = queue.Queue()

# These operations are atomic
safe_queue.put(item)  # Thread-safe
item = safe_queue.get()  # Thread-safe
```

## Conceptual Understanding & Analogies

### The Bathroom Analogy
Thread safety is like a single-occupancy bathroom:
- **Lock (Mutex)**: The door lock - only one person can use it
- **Race Condition**: Two people trying to enter at once
- **Deadlock**: Someone locks themselves in and falls asleep
- **Thread-Safe**: Has a working lock
- **Not Thread-Safe**: No lock on the door

### The Kitchen Analogy
Multiple cooks in a kitchen:
- **Shared Resource**: The stove, cutting board
- **Thread-Safe**: Each cook has assigned stations
- **Race Condition**: Two cooks reaching for the same knife
- **Synchronization**: "Behind you!" callouts
- **Deadlock**: Cook A needs B's pot, B needs A's pan

### The Bank Account Analogy
```python
# Race condition in banking
class UnsafeBankAccount:
    def __init__(self):
        self.balance = 1000
    
    def withdraw(self, amount):
        if self.balance >= amount:  # Check
            # Another thread might withdraw here!
            self.balance -= amount   # Withdraw
            return True
        return False

# Thread-safe version
class SafeBankAccount:
    def __init__(self):
        self.balance = 1000
        self._lock = threading.Lock()
    
    def withdraw(self, amount):
        with self._lock:  # Atomic check-and-withdraw
            if self.balance >= amount:
                self.balance -= amount
                return True
            return False
```

## Common Thread Safety Issues

### 1. **Check-Then-Act**
```python
# BAD: Race condition between check and act
if not os.path.exists(filename):  # Check
    # Another thread might create file here!
    with open(filename, 'w') as f:  # Act
        f.write('data')

# GOOD: Atomic operation
try:
    with open(filename, 'x') as f:  # 'x' = exclusive create
        f.write('data')
except FileExistsError:
    pass  # File already exists
```

### 2. **Read-Modify-Write**
```python
# BAD: Non-atomic increment
counter = counter + 1  # Read, modify, write

# GOOD: Use lock or atomic operation
with lock:
    counter = counter + 1
```

### 3. **Deadlocks**
```python
# Potential deadlock
lock1 = threading.Lock()
lock2 = threading.Lock()

def thread1():
    with lock1:
        with lock2:  # Wants lock2 while holding lock1
            pass

def thread2():
    with lock2:
        with lock1:  # Wants lock1 while holding lock2
            pass
```

## TypeScript/JavaScript Considerations

JavaScript is single-threaded but has concurrent concerns:

```typescript
// Web Workers - actual parallelism
const worker = new Worker('worker.js');
worker.postMessage({cmd: 'process', data: largeArray});

// Shared memory with SharedArrayBuffer
const sab = new SharedArrayBuffer(1024);
const arr = new Int32Array(sab);

// Atomics for thread-safe operations
Atomics.add(arr, 0, 1);  // Atomic increment
Atomics.compareExchange(arr, 0, oldValue, newValue);  // CAS operation

// async/await concurrency issues
let counter = 0;

async function increment() {
    const temp = counter;  // Read
    await someAsyncOperation();  // Another function might run!
    counter = temp + 1;  // Write - race condition!
}
```

## Testing for Thread Safety

```python
import threading
import time

def test_thread_safety():
    counter = Counter()
    threads = []
    
    def increment_many():
        for _ in range(1000):
            counter.increment()
    
    # Start 10 threads
    for _ in range(10):
        t = threading.Thread(target=increment_many)
        threads.append(t)
        t.start()
    
    # Wait for all threads
    for t in threads:
        t.join()
    
    # Should be 10,000 if thread-safe
    assert counter.value == 10000
```

## Best Practices

> [!tip] Thread Safety Guidelines
> 1. **Prefer immutability** - Can't have races on unchangeable data
> 2. **Minimize shared state** - Less sharing = fewer problems
> 3. **Use thread-safe collections** - `queue.Queue`, `concurrent.futures`
> 4. **Lock consistently** - Always acquire locks in same order
> 5. **Keep critical sections small** - Lock only what's necessary
> 6. **Use higher-level abstractions** - ThreadPoolExecutor, asyncio
> 7. **Document thread safety** - State whether code is thread-safe

## Key Takeaways

> [!important] Remember About Thread Safety
> 1. **Not automatic** - You must design for it
> 2. **Immutability is your friend** - Can't race on constants
> 3. **Locks prevent races** - But can cause deadlocks
> 4. **Test with multiple threads** - Single-threaded tests miss issues
> 5. **Static methods often unsafe** - Shared class state
> 6. **Instance methods safer** - But not automatically safe
> 7. **Singletons need special care** - Global shared state