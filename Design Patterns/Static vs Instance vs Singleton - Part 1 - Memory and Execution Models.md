# Part 1: Memory & Execution Models

#programming #python #typescript #memory #performance #architecture

[[Static vs Instance vs Singleton Patterns - Python & TypeScript Deep Dive|← Back to Main Guide]]

## Table of Contents

1. [[#Overview]]
2. [[#Python Memory Model]]
3. [[#TypeScript/JavaScript Memory Model]]
4. [[#Import Time vs Runtime Behavior]]
5. [[#Method Resolution and Lookup]]
6. [[#Garbage Collection Implications]]
7. [[#Performance Measurements]]

## Overview

Understanding how Python and TypeScript handle memory allocation and method execution is crucial for making informed decisions about static, instance, and singleton patterns. This section explores what actually happens "behind the scenes" when you use each pattern.

## Python Memory Model

### Object Creation and Memory Layout

```python
import sys
import gc

class Example:
    class_var = "shared"  # Stored in class __dict__
    
    def __init__(self):
        self.instance_var = "unique"  # Stored in instance __dict__

# Memory analysis
example = Example()
print(f"Instance size: {sys.getsizeof(example)} bytes")
print(f"Instance dict: {sys.getsizeof(example.__dict__)} bytes")
print(f"Class dict: {sys.getsizeof(Example.__dict__)} bytes")
```

> [!info] Python Memory Layout
> - **Class objects**: Stored in heap with metadata, methods, and class variables
> - **Instance objects**: Header + __dict__ for attributes + reference to class
> - **Methods**: Stored once in class, bound dynamically to instances

### Static Method Memory

```python
import dis

class MathUtils:
    @staticmethod
    def add(x, y):
        return x + y
    
    def instance_add(self, x, y):
        return x + y

# Bytecode comparison
print("Static method bytecode:")
dis.dis(MathUtils.add)

print("\nInstance method bytecode:")
dis.dis(MathUtils.instance_add)
```

**Key Differences:**
- Static methods have no implicit first parameter
- No method binding overhead
- Stored as regular functions in class namespace

### Instance Method Memory

```python
class Counter:
    def __init__(self):
        self.count = 0
    
    def increment(self):
        self.count += 1

# Method binding demonstration
counter1 = Counter()
counter2 = Counter()

# Methods are the same object
print(Counter.increment)  # <function Counter.increment>

# But bound methods are different
print(counter1.increment)  # <bound method Counter.increment of <__main__.Counter>>
print(counter2.increment)  # <bound method Counter.increment of <__main__.Counter>>

# Memory addresses
print(id(Counter.increment))  # Same for class
print(id(counter1.increment))  # Different for each instance
print(id(counter2.increment))  # Different for each instance
```

### Python Method Resolution Order (MRO)

```python
class A:
    @staticmethod
    def static_method():
        return "A.static"
    
    def instance_method(self):
        return "A.instance"

class B(A):
    @staticmethod
    def static_method():
        return "B.static"

# MRO examination
print(B.__mro__)  # (<class 'B'>, <class 'A'>, <class 'object'>)

# Static methods follow same MRO but no instance binding
B.static_method()  # Direct lookup in B.__dict__
b = B()
b.instance_method()  # Lookup through MRO + binding
```

## TypeScript/JavaScript Memory Model

### Prototype Chain and Memory

```typescript
class Example {
    static classVar = "shared";
    instanceVar: string;
    
    constructor() {
        this.instanceVar = "unique";
    }
    
    static staticMethod() {
        return "static";
    }
    
    instanceMethod() {
        return this.instanceVar;
    }
}

// Compiled JavaScript (simplified)
function Example() {
    this.instanceVar = "unique";
}
Example.classVar = "shared";
Example.staticMethod = function() { return "static"; };
Example.prototype.instanceMethod = function() { return this.instanceVar; };

// Memory layout
console.log(Example.prototype);  // { instanceMethod: [Function] }
console.log(Example);  // { classVar: 'shared', staticMethod: [Function] }
```

### TypeScript Compilation Output

```typescript
// TypeScript source
class Database {
    private static instance: Database;
    private connection: any;
    
    private constructor() {
        this.connection = this.connect();
    }
    
    static getInstance(): Database {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }
    
    private connect() {
        return { connected: true };
    }
}

// Compiled JavaScript (ES6)
class Database {
    constructor() {
        this.connection = this.connect();
    }
    static getInstance() {
        if (!Database.instance) {
            Database.instance = new Database();
        }
        return Database.instance;
    }
    connect() {
        return { connected: true };
    }
}
```

### Memory Allocation Comparison

```typescript
// Memory profiling example
class MemoryTest {
    static staticData = new Array(1000).fill(0);
    instanceData: number[];
    
    constructor() {
        this.instanceData = new Array(1000).fill(0);
    }
    
    static staticMethod() {
        return MemoryTest.staticData.length;
    }
    
    instanceMethod() {
        return this.instanceData.length;
    }
}

// Node.js memory measurement
const used1 = process.memoryUsage().heapUsed;
const instances = Array.from({ length: 100 }, () => new MemoryTest());
const used2 = process.memoryUsage().heapUsed;

console.log(`Memory per instance: ${(used2 - used1) / 100} bytes`);
console.log(`Static data shared: ${MemoryTest.staticData === MemoryTest.staticData}`);
```

## Import Time vs Runtime Behavior

### Python Import Execution

```python
# module.py
print("Module loading...")

class Config:
    print("Class definition executing...")
    
    # Executes at import time
    DEFAULT_TIMEOUT = 30
    _compiled_regex = re.compile(r'\d+')
    
    def __init__(self):
        print("Instance creation...")

# Singleton via module
_instance = None

def get_instance():
    global _instance
    if _instance is None:
        print("Creating singleton...")
        _instance = Config()
    return _instance

# When imported:
# 1. "Module loading..." prints
# 2. "Class definition executing..." prints
# 3. Class and functions defined
# 4. _instance remains None until get_instance() called
```

### TypeScript/Node.js Module Caching

```typescript
// config.ts
console.log("Module loading...");

export class Config {
    static {
        console.log("Static initializer executing...");
    }
    
    // Executes when class is first referenced
    static readonly DEFAULT_TIMEOUT = 30;
    private static _instance: Config;
    
    constructor() {
        console.log("Instance creation...");
    }
    
    static getInstance(): Config {
        if (!Config._instance) {
            console.log("Creating singleton...");
            Config._instance = new Config();
        }
        return Config._instance;
    }
}

// Module cached after first import
// Subsequent imports return cached module
```

### Loading Time Comparison

```python
# Python loading measurement
import time

# measure_import.py
start = time.perf_counter()
import heavy_module  # Contains many static computations
end = time.perf_counter()
print(f"Import time: {(end - start) * 1000:.2f}ms")

# measure_runtime.py
import heavy_module
start = time.perf_counter()
instance = heavy_module.HeavyClass()  # Instance creation
end = time.perf_counter()
print(f"Instance creation: {(end - start) * 1000:.2f}ms")
```

```typescript
// TypeScript loading measurement
// measure_import.ts
console.time('import');
import { HeavyModule } from './heavy-module';
console.timeEnd('import');

// measure_runtime.ts
import { HeavyClass } from './heavy-module';
console.time('instantiation');
const instance = new HeavyClass();
console.timeEnd('instantiation');
```

## Method Resolution and Lookup

### Python Method Resolution

```python
import timeit

class TestClass:
    @staticmethod
    def static_method(x):
        return x * 2
    
    def instance_method(self, x):
        return x * 2

# Lookup performance
instance = TestClass()

# Static method lookup (direct)
static_time = timeit.timeit(
    'TestClass.static_method(5)',
    globals=globals(),
    number=1000000
)

# Instance method lookup (MRO + binding)
instance_time = timeit.timeit(
    'instance.instance_method(5)',
    globals=globals(),
    number=1000000
)

print(f"Static method: {static_time:.4f}s")
print(f"Instance method: {instance_time:.4f}s")
print(f"Overhead: {((instance_time - static_time) / static_time) * 100:.1f}%")
```

### TypeScript Method Resolution

```typescript
class TestClass {
    static staticMethod(x: number): number {
        return x * 2;
    }
    
    instanceMethod(x: number): number {
        return x * 2;
    }
}

// Performance measurement
const instance = new TestClass();
const iterations = 1000000;

console.time('static');
for (let i = 0; i < iterations; i++) {
    TestClass.staticMethod(5);
}
console.timeEnd('static');

console.time('instance');
for (let i = 0; i < iterations; i++) {
    instance.instanceMethod(5);
}
console.timeEnd('instance');

// Prototype chain lookup
console.log(instance.instanceMethod === TestClass.prototype.instanceMethod); // true
console.log(TestClass.staticMethod === TestClass.staticMethod); // true
```

## Garbage Collection Implications

### Python Reference Counting

```python
import gc
import weakref

class Resource:
    instances = []  # Class-level reference
    
    def __init__(self, name):
        self.name = name
        Resource.instances.append(self)  # Prevents GC!
    
    def __del__(self):
        print(f"Deleting {self.name}")

# Singleton with proper cleanup
class SingletonResource:
    _instance = None
    _refs = weakref.WeakSet()  # Weak references
    
    @classmethod
    def getInstance(cls):
        if cls._instance is None:
            cls._instance = cls()
        return cls._instance
    
    def register_user(self, user):
        self._refs.add(user)  # Won't prevent GC

# Memory leak example
r1 = Resource("R1")
r1 = None  # Won't be garbage collected!
gc.collect()
print(f"Resource instances: {len(Resource.instances)}")  # Still 1

# Proper cleanup
Resource.instances.clear()
gc.collect()
```

### TypeScript/JavaScript Garbage Collection

```typescript
// Memory leak with static references
class MemoryLeaker {
    private static cache: Map<string, Buffer> = new Map();
    
    static cacheData(key: string, data: Buffer): void {
        // This will grow indefinitely!
        this.cache.set(key, data);
    }
}

// Proper memory management
class ManagedCache {
    private static cache: Map<string, WeakRef<Buffer>> = new Map();
    private static maxSize = 100;
    
    static cacheData(key: string, data: Buffer): void {
        // Use WeakRef for automatic cleanup
        this.cache.set(key, new WeakRef(data));
        
        // Implement size limits
        if (this.cache.size > this.maxSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
        }
    }
    
    static getData(key: string): Buffer | undefined {
        const ref = this.cache.get(key);
        const data = ref?.deref();
        
        // Clean up dead references
        if (ref && !data) {
            this.cache.delete(key);
        }
        
        return data;
    }
}

// Singleton lifecycle management
class ManagedSingleton {
    private static instance?: ManagedSingleton;
    private resources: any[] = [];
    
    static getInstance(): ManagedSingleton {
        if (!this.instance) {
            this.instance = new ManagedSingleton();
        }
        return this.instance;
    }
    
    static dispose(): void {
        if (this.instance) {
            this.instance.cleanup();
            this.instance = undefined;
        }
    }
    
    private cleanup(): void {
        this.resources.forEach(r => r.dispose?.());
        this.resources = [];
    }
}
```

## Performance Measurements

### Comprehensive Benchmark

```python
# Python benchmark
import time
import tracemalloc
from dataclasses import dataclass

@dataclass
class Metrics:
    time_ms: float
    memory_bytes: int

def measure_performance(func, iterations=100000):
    # Memory measurement
    tracemalloc.start()
    start_mem = tracemalloc.get_traced_memory()[0]
    
    # Time measurement
    start_time = time.perf_counter()
    for _ in range(iterations):
        func()
    end_time = time.perf_counter()
    
    # Calculate metrics
    end_mem = tracemalloc.get_traced_memory()[0]
    tracemalloc.stop()
    
    return Metrics(
        time_ms=(end_time - start_time) * 1000,
        memory_bytes=end_mem - start_mem
    )

# Test implementations
class TestPatterns:
    data = [1, 2, 3, 4, 5]
    
    @staticmethod
    def static_sum():
        return sum(TestPatterns.data)
    
    def __init__(self):
        self.data = [1, 2, 3, 4, 5]
    
    def instance_sum(self):
        return sum(self.data)

# Singleton
class SingletonTest:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.data = [1, 2, 3, 4, 5]
        return cls._instance
    
    def sum(self):
        return sum(self.data)

# Run benchmarks
static_metrics = measure_performance(TestPatterns.static_sum)
instance = TestPatterns()
instance_metrics = measure_performance(instance.instance_sum)
singleton = SingletonTest()
singleton_metrics = measure_performance(singleton.sum)

print(f"Static - Time: {static_metrics.time_ms:.2f}ms, Memory: {static_metrics.memory_bytes} bytes")
print(f"Instance - Time: {instance_metrics.time_ms:.2f}ms, Memory: {instance_metrics.memory_bytes} bytes")
print(f"Singleton - Time: {singleton_metrics.time_ms:.2f}ms, Memory: {singleton_metrics.memory_bytes} bytes")
```

```typescript
// TypeScript benchmark
interface Metrics {
    timeMs: number;
    memoryBytes: number;
}

function measurePerformance(func: () => void, iterations = 100000): Metrics {
    const startMem = process.memoryUsage().heapUsed;
    const startTime = performance.now();
    
    for (let i = 0; i < iterations; i++) {
        func();
    }
    
    const endTime = performance.now();
    const endMem = process.memoryUsage().heapUsed;
    
    return {
        timeMs: endTime - startTime,
        memoryBytes: endMem - startMem
    };
}

// Test implementations
class TestPatterns {
    static data = [1, 2, 3, 4, 5];
    private instanceData = [1, 2, 3, 4, 5];
    
    static staticSum(): number {
        return TestPatterns.data.reduce((a, b) => a + b, 0);
    }
    
    instanceSum(): number {
        return this.instanceData.reduce((a, b) => a + b, 0);
    }
}

// Singleton
class SingletonTest {
    private static instance?: SingletonTest;
    private data = [1, 2, 3, 4, 5];
    
    static getInstance(): SingletonTest {
        if (!SingletonTest.instance) {
            SingletonTest.instance = new SingletonTest();
        }
        return SingletonTest.instance;
    }
    
    sum(): number {
        return this.data.reduce((a, b) => a + b, 0);
    }
}

// Run benchmarks
const staticMetrics = measurePerformance(() => TestPatterns.staticSum());
const instance = new TestPatterns();
const instanceMetrics = measurePerformance(() => instance.instanceSum());
const singleton = SingletonTest.getInstance();
const singletonMetrics = measurePerformance(() => singleton.sum());

console.log(`Static - Time: ${staticMetrics.timeMs.toFixed(2)}ms, Memory: ${staticMetrics.memoryBytes} bytes`);
console.log(`Instance - Time: ${instanceMetrics.timeMs.toFixed(2)}ms, Memory: ${instanceMetrics.memoryBytes} bytes`);
console.log(`Singleton - Time: ${singletonMetrics.timeMs.toFixed(2)}ms, Memory: ${singletonMetrics.memoryBytes} bytes`);
```

## Key Takeaways

> [!important] Memory & Performance Insights
> 1. **Static methods** have lowest memory overhead but can't access instance state
> 2. **Instance methods** require object allocation but provide flexibility
> 3. **Singletons** save memory for shared resources but need careful lifecycle management
> 4. **Python** uses reference counting + cycle detection; watch for circular references
> 5. **TypeScript/JS** uses mark-and-sweep GC; be mindful of closures and event listeners
> 6. **Import time costs** can be significant for static initialization
> 7. **Method lookup** performance differs between languages but is rarely a bottleneck

[[Static vs Instance vs Singleton - Part 2 - Implementation Deep Dive|Next: Part 2 - Implementation Deep Dive →]]