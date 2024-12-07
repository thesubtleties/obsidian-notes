## Basic Class Structure
```python
class Person:
    # Class variable (shared by all instances)
    species = "Human"
    
    # Constructor
    def __init__(self, name: str, age: int):
        # Instance variables (unique to each instance)
        self.name = name
        self.age = age
        self._private = "Hidden"  # Convention for private
        
    # Instance method
    def greet(self) -> str:
        return f"Hello, I'm {self.name}"
    
    # Class method
    @classmethod
    def create_anonymous(cls) -> 'Person':
        return cls("Anonymous", 0)
        
    # Static method
    @staticmethod
    def is_adult(age: int) -> bool:
        return age >= 18
```
Basic class structure showing instance variables, methods, and class-level attributes. This forms the foundation of OOP in Python.

## Inheritance and Polymorphism
```python
class Animal:
    def __init__(self, name: str):
        self.name = name
    
    def speak(self) -> str:
        raise NotImplementedError("Subclass must implement")

class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says Woof!"

class Cat(Animal):
    def speak(self) -> str:
        return f"{self.name} says Meow!"

# Multiple inheritance
class Pet(Animal, Comparable):
    pass
```
Inheritance allows code reuse and polymorphism enables different classes to be treated uniformly. Python supports multiple inheritance, allowing classes to inherit from multiple parent classes.

## Properties and Decorators
```python
class BankAccount:
    def __init__(self, initial_balance: float = 0):
        self._balance = initial_balance
    
    @property
    def balance(self) -> float:
        """Get the current balance"""
        return self._balance
    
    @balance.setter
    def balance(self, value: float):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value
    
    @balance.deleter
    def balance(self):
        self._balance = 0
```
Properties provide a way to manage attribute access with getter, setter, and deleter methods. They enable data encapsulation and validation.

## Magic Methods
```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x = x
        self.y = y
    
    def __str__(self) -> str:
        return f"Vector({self.x}, {self.y})"
    
    def __add__(self, other: 'Vector') -> 'Vector':
        return Vector(self.x + other.x, self.y + other.y)
    
    def __len__(self) -> int:
        return 2
    
    def __eq__(self, other: object) -> bool:
        if not isinstance(other, Vector):
            return NotImplemented
        return self.x == other.x and self.y == other.y
```
Magic methods (dunder methods) customize class behavior for built-in operations and functions. They allow classes to work with Python's built-in features.

## Abstract Classes and Interfaces
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height
    
    def area(self) -> float:
        return self.width * self.height
    
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)
```
Abstract base classes define interfaces that derived classes must implement. They ensure consistent behavior across related classes.

## Data Classes (Python 3.7+)
```python
from dataclasses import dataclass, field

@dataclass
class Point:
    x: float
    y: float
    label: str = "Default"
    history: list = field(default_factory=list)
    
    def move(self, dx: float, dy: float):
        self.x += dx
        self.y += dy
        self.history.append((self.x, self.y))

@dataclass(frozen=True)  # Immutable dataclass
class Configuration:
    host: str
    port: int = 8080
```
Data classes automatically generate special methods for classes that primarily store data. They reduce boilerplate code for simple classes.

## Class Composition
```python
class Engine:
    def start(self):
        return "Engine starting"

class Car:
    def __init__(self):
        self.engine = Engine()  # Composition
        self.fuel = 100
    
    def start(self):
        if self.fuel > 0:
            return self.engine.start()
        return "Out of fuel"
```
Composition allows building complex classes by combining simpler ones. It's often preferred over inheritance for flexibility.

## Method Types and Decorators
```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius
    
    @property
    def celsius(self) -> float:
        return self._celsius
    
    @celsius.setter
    def celsius(self, value: float):
        self._celsius = value
    
    @property
    def fahrenheit(self) -> float:
        return (self._celsius * 9/5) + 32
    
    @classmethod
    def from_fahrenheit(cls, fahrenheit: float) -> 'Temperature':
        celsius = (fahrenheit - 32) * 5/9
        return cls(celsius)
```
Different method types (instance, class, static) and decorators provide flexibility in how methods are used and accessed.

## Context Managers
```python
class FileManager:
    def __init__(self, filename: str):
        self.filename = filename
        self.file = None
    
    def __enter__(self):
        self.file = open(self.filename, 'r')
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()

# Usage
with FileManager('data.txt') as file:
    content = file.read()
```
Context managers provide a clean way to manage resources, ensuring proper setup and cleanup.

## Metaclasses
```python
class Singleton(type):
    _instances = {}
    
    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    def __init__(self):
        self.connection = "Connected"
```
Metaclasses allow customization of class creation. They're advanced features used for implementing patterns like singletons or registries.

## Best Practices
```python
class BestPractices:
    """
    Class demonstrating Python OOP best practices.
    """
    def __init__(self):
        self._private = None  # Use underscore for private
    
    @property
    def private(self):
        """Getter with docstring."""
        return self._private
    
    def method_with_type_hints(self, param: str) -> bool:
        """Methods should have type hints and docstrings."""
        return bool(param)
    
    @classmethod
    def factory_method(cls) -> 'BestPractices':
        """Use class methods for alternative constructors."""
        return cls()
```
Follow Python's conventions for naming, documentation, and type hints. Use properties for attribute access control and appropriate method types.

This overview covers the main aspects of Object-Oriented Programming in Python, from basic class structure to advanced concepts like metaclasses and context managers.