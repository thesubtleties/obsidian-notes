# Method Resolution Order (MRO)

#programming #python #inheritance #oop #patterns

## What is MRO?

**Method Resolution Order (MRO)** is the order in which Python looks for a method or attribute in a hierarchy of classes. When you call a method on an object, Python needs to know which implementation to use if that method exists in multiple places in the inheritance chain.

## Computer Science Definition

MRO is a linearization algorithm that creates a deterministic ordering of classes in an inheritance hierarchy, ensuring that:
1. Each class appears before its parents
2. The order respects the inheritance relationships
3. The same class doesn't appear twice
4. The algorithm is monotonic (preserves order)

Python uses the **C3 Linearization Algorithm** (also called C3 superclass linearization) to compute MRO.

## How It Relates to Static vs Instance Patterns

In the context of our patterns:
- **Instance methods** follow MRO when inherited
- **Static methods** also follow MRO but don't have access to the instance
- **Class methods** follow MRO and receive the actual class as first argument

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
    
    # Inherits instance_method from A

# MRO: B -> A -> object
print(B.__mro__)  # (<class 'B'>, <class 'A'>, <class 'object'>)

# Both static and instance methods follow MRO
b = B()
print(B.static_method())     # "B.static" - Found in B first
print(b.instance_method())   # "A.instance" - Not in B, found in A
```

## Conceptual Understanding & Analogies

### The Family Tree Analogy
Think of MRO like looking for a family trait:
1. You first check yourself (the instance's class)
2. Then your immediate parents (left to right)
3. Then grandparents (following the same pattern)
4. Until you find who has that trait

### The Corporate Hierarchy Analogy
Imagine asking for approval in a company:
```
        CEO (object)
         |
    Vice President (Base Class A)
         |
    Manager (Your Class B)
         |
    You (Instance)
```

When you need something, you ask your manager first. If they don't have it, they ask the VP, and so on.

## Why MRO Matters

### 1. **Multiple Inheritance Complexity**
```python
class A:
    def method(self):
        return "A"

class B:
    def method(self):
        return "B"

class C(A, B):  # Order matters!
    pass

# C's MRO: C -> A -> B -> object
# C().method() returns "A" because A comes before B
```

### 2. **Diamond Problem Resolution**
```python
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    def speak(self):
        return "Woof"

class Cat(Animal):
    def speak(self):
        return "Meow"

class Chimera(Dog, Cat):  # Mythical dog-cat hybrid
    pass

# MRO: Chimera -> Dog -> Cat -> Animal -> object
# Chimera().speak() returns "Woof" (from Dog)
```

## Practical Implications

### For Static Methods
- Static methods are found using MRO but don't participate in polymorphism the same way
- They're essentially namespaced functions

### For Instance Methods  
- Critical for understanding which method gets called
- Affects `super()` behavior
- Determines attribute lookup order

### For Singleton Pattern
- MRO affects how singleton classes inherit behavior
- Important when creating singleton hierarchies

## Debugging MRO Issues

```python
# Always check MRO when debugging inheritance
print(YourClass.__mro__)
print(YourClass.mro())  # Alternative method

# Use help() to see all methods and their sources
help(YourClass)

# Check where a method comes from
print(YourClass.method_name)  # Shows which class defines it
```

## Key Takeaways

> [!important] Remember About MRO
> 1. **Order matters** in multiple inheritance: `class C(A, B)` is different from `class C(B, A)`
> 2. **Left-to-right, depth-first** with duplicates removed
> 3. **Use `super()`** to respect MRO instead of hardcoding parent calls
> 4. **Check `__mro__`** when debugging inheritance issues
> 5. **Affects all methods** - static, class, and instance methods