## Basic Inheritance
```python
class Animal:
    def __init__(self, name: str):
        self.name = name  # Instance variable
        
    def speak(self) -> str:
        return "Some sound"  # Base method that can be overridden
        
    def describe(self) -> str:
        return f"I am {self.name}"

class Dog(Animal):  # Dog inherits from Animal
    def __init__(self, name: str, breed: str):
        super().__init__(name)  # Call parent constructor (like super() in JS)
        self.breed = breed      # Add new instance variable
        
    def speak(self) -> str:     # Override parent method
        return "Woof!"
        
    def fetch(self) -> str:     # Add new method specific to Dog
        return f"{self.name} is fetching"
```
Similar to JavaScript's extends keyword. The main difference is Python's explicit `self` parameter (equivalent to JavaScript's `this`) and the use of `super()` to call parent methods. Type hints (`: str` and `-> str`) are optional but provide better IDE support.

## Multiple Inheritance
```python
# These are like interfaces in JavaScript, but can contain implementation
class Flyable:
    def fly(self) -> str:
        return "Flying..."
        
    def land(self) -> str:
        return "Landing..."

class Swimmable:
    def swim(self) -> str:
        return "Swimming..."
        
    def dive(self) -> str:
        return "Diving..."

# Duck inherits from THREE classes - not possible in JavaScript!
class Duck(Animal, Flyable, Swimmable):
    def speak(self) -> str:
        return "Quack!"
    
    def move(self) -> list[str]:
        return [self.fly(), self.swim()]  # Can use methods from all parent classes
```
Multiple inheritance is a major difference from JavaScript. In JavaScript, a class can only extend one class (though it can implement multiple interfaces). Python allows a class to inherit from multiple parent classes, combining all their methods and attributes.

## Method Resolution Order (MRO)
```python
class A:
    def method(self):
        return "A method"

class B(A):
    def method(self):
        return "B method"

class C(A):
    def method(self):
        return "C method"

class D(B, C):  # Inherits from both B and C
    pass    # Empty class that inherits all parent methods
    
# Check MRO
print(D.__mro__)  # Shows method resolution order
# Output: (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```
The "Diamond Problem" occurs in multiple inheritance when a class inherits from two classes that have the same parent class. Python resolves this using the C3 linearization algorithm, which determines the order in which methods are looked up. This isn't an issue in JavaScript since it doesn't support multiple inheritance.

## Abstract Base Classes
```python
from abc import ABC, abstractmethod

# Similar to abstract classes in TypeScript
class Shape(ABC):  # ABC = Abstract Base Class
    @abstractmethod  # Methods marked with this must be implemented by children
    def area(self) -> float:
        pass
    
    @abstractmethod
    def perimeter(self) -> float:
        pass
    
    # Concrete method that uses abstract methods
    def describe(self) -> str:
        return f"Area: {self.area()}, Perimeter: {self.perimeter()}"

class Rectangle(Shape):
    def __init__(self, width: float, height: float):
        self.width = width
        self.height = height
    
    # Must implement all abstract methods
    def area(self) -> float:
        return self.width * self.height
    
    def perimeter(self) -> float:
        return 2 * (self.width + self.height)
```
Similar to abstract classes in TypeScript. Abstract base classes can't be instantiated directly and force derived classes to implement certain methods. This is like combining JavaScript's abstract classes and interfaces.

## Mixin Classes
```python
# Mixins are reusable classes that add functionality to other classes
class JSONSerializableMixin:
    def to_json(self) -> dict:
        # Convert object attributes to dictionary
        return {
            key: value for key, value in self.__dict__.items()
            if not key.startswith('_')
        }

class LoggerMixin:
    def log(self, message: str) -> None:
        print(f"[{self.__class__.__name__}] {message}")

# Use mixins to add functionality without full inheritance
class User(JSONSerializableMixin, LoggerMixin):
    def __init__(self, name: str, email: str):
        self.name = name
        self.email = email
        self.log(f"Created user {name}")  # Can use mixin methods

# Usage
user = User("John", "john@example.com")
print(user.to_json())  # Uses JSONSerializableMixin
user.log("User logged in")  # Uses LoggerMixin
```
Mixins are a way to add functionality to classes without using inheritance. While JavaScript has mixins through various patterns, Python's multiple inheritance makes them more straightforward. Think of mixins as plug-and-play functionality that you can add to any class.

## Type Checking with Inheritance
```python
from typing import Type, TypeVar

# T is a type variable that must be a subclass of Animal
T = TypeVar('T', bound='Animal')

class AnimalFactory:
    @classmethod
    def create_animal(cls, animal_type: Type[T], name: str) -> T:
        return animal_type(name)

# Usage
dog = AnimalFactory.create_animal(Dog, "Rover")    # Returns Dog instance
cat = AnimalFactory.create_animal(Cat, "Whiskers") # Returns Cat instance
```
This is similar to TypeScript's generics. It provides type safety when working with classes and inheritance. The `TypeVar` creates a type variable that can represent any subclass of `Animal`.

## Best Practices
```python
# Favor composition over inheritance
class Engine:
    def start(self):
        return "Engine started"

class Car:
    def __init__(self):
        # Composition: Car HAS an Engine instead of IS an Engine
        self.engine = Engine()  # This is often better than inheritance
    
    def start(self):
        return self.engine.start()

# Use abstract base classes for interfaces
class Validator(ABC):
    @abstractmethod
    def validate(self, data: any) -> bool:
        pass

# Keep inheritance hierarchies shallow
class Animal:
    pass

class Mammal(Animal):  # One level deep
    pass

class Dog(Mammal):     # Two levels deep
    pass  # Avoid going deeper if possible - can get confusing!
```
These patterns are similar in JavaScript, but Python's multiple inheritance and abstract base classes provide more options for code organization. The principle of favoring composition over inheritance is universal across languages.

The main differences from JavaScript are:
1. Multiple inheritance (Python) vs single inheritance (JavaScript)
2. Explicit self parameter vs implicit this
3. Built-in support for abstract classes and mixins
4. Method Resolution Order (MRO) for handling multiple inheritance
5. More robust type hinting system (similar to TypeScript)
6. Different syntax for class definition and inheritance