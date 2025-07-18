# Static vs Instance vs Singleton Patterns: Python & TypeScript Deep Dive

#programming #python #typescript #patterns #architecture #design-patterns

## Table of Contents

1. [[#Executive Summary]]
2. [[#Core Concepts Overview]]
3. [[#Pattern Quick Reference]]
4. [[#Decision Framework]]
5. [[#Detailed Guides]]
6. [[#Cross-Language Comparison]]
7. [[#Further Reading]]

## Executive Summary

This guide provides a comprehensive analysis of Static, Instance, and Singleton patterns in Python and TypeScript, focusing on:
- **Memory and execution models** - What happens behind the scenes
- **Implementation details** - How each pattern works in both languages
- **Performance implications** - Memory usage, speed, and scalability
- **Testing strategies** - Approaches for unit and integration testing
- **Dependency injection** - Enabling extensibility and testability
- **Real-world applications** - When and why to use each pattern

### Key Takeaways

> [!info] Quick Decision Guide
> - **Static Methods**: Utility functions, no state needed, pure computations
> - **Instance Methods**: State management, multiple independent objects, dependency injection
> - **Singleton Pattern**: Global state, resource sharing, configuration management

## Core Concepts Overview

### Pattern Definitions

**Static Methods/Properties**
- Belong to the class itself, not instances
- No access to instance state (`self`/`this`)
- Shared across all instances
- Loaded at import/module time

**Instance Methods/Properties**
- Belong to individual objects
- Access to instance state via `self`/`this`
- Each instance has its own copy
- Created at instantiation time

**Singleton Pattern**
- Only one instance exists globally
- Shared state across application
- Lazy or eager initialization
- Various implementation strategies

## Pattern Quick Reference

### Python Syntax

```python
class Example:
    # Static (class) variable
    static_var = "shared"
    
    def __init__(self):
        # Instance variable
        self.instance_var = "unique"
    
    # Instance method
    def instance_method(self):
        return self.instance_var
    
    # Static method
    @staticmethod
    def static_method():
        return "no self needed"
    
    # Class method (alternative to static)
    @classmethod
    def class_method(cls):
        return cls.static_var
```

### TypeScript Syntax

```typescript
class Example {
    // Static property
    static staticVar = "shared";
    
    // Instance property
    instanceVar: string;
    
    constructor() {
        this.instanceVar = "unique";
    }
    
    // Instance method
    instanceMethod(): string {
        return this.instanceVar;
    }
    
    // Static method
    static staticMethod(): string {
        return "no this needed";
    }
}
```

## Decision Framework

### When to Use Each Pattern

| Pattern | Use When | Avoid When |
|---------|----------|------------|
| **Static** | • Pure utility functions<br>• No state required<br>• Helper/conversion methods<br>• Factory methods | • Need instance data<br>• Require polymorphism<br>• Need dependency injection |
| **Instance** | • Managing object state<br>• Multiple independent objects<br>• Polymorphic behavior<br>• Dependency injection | • No state needed<br>• Single global instance<br>• Pure calculations |
| **Singleton** | • Global configuration<br>• Resource pooling<br>• Caching layer<br>• Service registry | • Multiple instances needed<br>• Testing is priority<br>• Avoiding global state |

### Performance Comparison

| Aspect | Static | Instance | Singleton |
|--------|--------|----------|-----------|
| Memory | Lowest | Per instance | Single instance |
| Creation Time | At import | On instantiation | First access/import |
| Access Speed | Fastest | Fast | Fast (after init) |
| Testing | Easy | Easy | Challenging |
| Thread Safety | Implicit | Per instance | Needs consideration |

## Detailed Guides

### Part 1: Memory & Execution Models
- [[Static vs Instance vs Singleton - Part 1 - Memory and Execution Models|Part 1: Memory & Execution Models]]
  - Python memory model and method resolution
  - TypeScript prototype chain and compilation
  - Import time vs runtime behavior
  - Garbage collection implications

### Part 2: Implementation Deep Dive
- [[Static vs Instance vs Singleton - Part 2 - Implementation Deep Dive|Part 2: Implementation Deep Dive]]
  - Static method internals in both languages
  - Instance method mechanics and binding
  - Singleton pattern implementations
  - Thread safety and concurrency

### Part 3: Dependency Injection & Testing
- [[Static vs Instance vs Singleton - Part 3 - Dependency Injection and Testing|Part 3: Dependency Injection & Testing]]
  - DI patterns in Python vs TypeScript
  - Testing strategies for each pattern
  - Mocking and stubbing techniques
  - Framework integration

### Part 4: Real-World Examples
- [[Static vs Instance vs Singleton - Part 4 - Real World Examples|Part 4: Real-World Examples]]
  - Database service patterns
  - Validation and content safety
  - AI/ML service integration
  - API client patterns

### Part 5: Advanced Patterns & Performance
- [[Static vs Instance vs Singleton - Part 5 - Advanced Patterns and Performance|Part 5: Advanced Patterns & Performance]]
  - Hybrid patterns and factories
  - Performance benchmarks
  - Memory profiling
  - Optimization strategies

## Cross-Language Comparison

### Key Differences

| Feature | Python | TypeScript |
|---------|--------|------------|
| Static Method Declaration | `@staticmethod` | `static` keyword |
| Instance Reference | `self` (explicit) | `this` (implicit) |
| Method Binding | Manual/automatic | Prototype-based |
| Module System | Import-time execution | Compiled modules |
| Type Safety | Runtime + hints | Compile-time |
| Singleton Implementation | Multiple approaches | Class/module-based |

### Universal Principles

> [!important] Language-Agnostic Concepts
> 1. **Separation of Concerns**: Static for utilities, instance for state
> 2. **Single Responsibility**: Each pattern serves a specific purpose
> 3. **Testability**: Consider testing needs when choosing patterns
> 4. **Performance**: Understand memory and execution implications
> 5. **Maintainability**: Choose patterns that match team expertise

## Common Pitfalls

> [!warning] Watch Out For
> - **Global State Abuse**: Singletons can create hidden dependencies
> - **Testing Difficulties**: Static methods and singletons can be hard to mock
> - **Memory Leaks**: Singletons holding references can prevent garbage collection
> - **Thread Safety**: Both languages require careful singleton implementation
> - **Import Side Effects**: Python's import-time execution can cause issues

## Best Practices

### General Guidelines

1. **Prefer Instance Methods** for flexibility and testability
2. **Use Static Methods** for pure functions and utilities
3. **Implement Singletons** carefully with clear lifecycle management
4. **Document Pattern Choice** to help future maintainers
5. **Consider Alternatives** like dependency injection containers

### Testing Recommendations

```python
# Python: Use dependency injection for testability
class Service:
    def __init__(self, database=None):
        self.database = database or DefaultDatabase()
```

```typescript
// TypeScript: Interface-based design
interface IDatabase {
    query(sql: string): Promise<any>;
}

class Service {
    constructor(private database: IDatabase) {}
}
```

## Migration Strategies

> [!tip] Refactoring Between Patterns
> 1. **Static → Instance**: Add state management, update call sites
> 2. **Instance → Static**: Extract pure functions, remove state dependencies
> 3. **Instance → Singleton**: Implement getInstance(), manage lifecycle
> 4. **Singleton → Instance**: Add factory pattern, update dependencies

## Further Reading

### Internal Links
- [[Python Topics/Object-Oriented Programming Topics/Design Patterns in Python - A Beginner's Guide|Python Design Patterns Guide]]
- [[React & Redux/React Topics/React and Modern JavaScript Patterns|JavaScript Patterns in React]]
- [[Full Stack Patterns|Full Stack Architecture Patterns]]

### Topics to Explore
- Dependency Injection Containers
- Factory and Abstract Factory Patterns
- Service Locator Pattern
- Repository Pattern
- Unit of Work Pattern

---

> [!note] About This Guide
> This guide provides in-depth coverage of static, instance, and singleton patterns in Python and TypeScript. Each linked section contains detailed examples, performance benchmarks, and practical applications. The guide emphasizes understanding "what happens behind the scenes" to make informed architectural decisions.