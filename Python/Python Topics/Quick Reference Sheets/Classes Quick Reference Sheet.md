## Basic Class Structure
```python
class ClassName:
    # Class variable (shared by all instances)
    class_variable = "I'm shared"

    def __init__(self, param1, param2):
        # Instance variables (unique to each instance)
        self.param1 = param1
        self.param2 = param2
    
    def method(self):
        # Instance method
        return f"Using {self.param1}"
    
    @classmethod
    def class_method(cls):
        # Class method
        return f"Using {cls.class_variable}"
    
    @staticmethod
    def static_method():
        # Static method (doesn't use class or instance)
        return "I'm static"
```

## Instance vs Class Variables
```python
class Dog:
    # Class variable (shared)
    species = "Canis"
    
    def __init__(self, name):
        # Instance variable (unique)
        self.name = name

# Usage
dog1 = Dog("Rover")
dog2 = Dog("Spot")

print(dog1.name)        # "Rover" (instance)
print(Dog.species)      # "Canis" (class)
print(dog1.species)     # "Canis" (inherited)
```

## Methods Types
```python
class Example:
    def instance_method(self):
        # Regular method, gets self
        return f"Instance method"
    
    @classmethod
    def class_method(cls):
        # Gets class as first arg
        return f"Class method"
    
    @staticmethod
    def static_method():
        # Gets neither class nor self
        return "Static method"
    
    @property
    def prop(self):
        # Used as property
        return "I'm a property"
```

## Inheritance
```python
class Parent:
    def speak(self):
        return "Parent speaking"

class Child(Parent):
    def speak(self):
        # Override parent method
        return "Child speaking"
    
    def both_speak(self):
        # Call parent method
        parent_speech = super().speak()
        return f"{parent_speech} and child too"
```

## Properties and Setters
```python
class Person:
    def __init__(self, name):
        self._name = name
    
    @property
    def name(self):
        # Getter
        return self._name
    
    @name.setter
    def name(self, value):
        # Setter
        if not isinstance(value, str):
            raise ValueError("Name must be string")
        self._name = value
    
    @name.deleter
    def name(self):
        # Deleter
        del self._name
```

## Special Methods
```python
class SpecialExample:
    def __init__(self, value):
        self.value = value
    
    def __str__(self):
        # String representation
        return f"Value: {self.value}"
    
    def __repr__(self):
        # Developer representation
        return f"SpecialExample({self.value})"
    
    def __len__(self):
        # Length
        return len(self.value)
    
    def __eq__(self, other):
        # Equality comparison
        return self.value == other.value
```

## Common Patterns
```python
# Factory Pattern
class Factory:
    @classmethod
    def create(cls, type):
        if type == "A":
            return ClassA()
        return ClassB()

# Singleton Pattern
class Singleton:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Context Manager
class FileHandler:
    def __enter__(self):
        # Setup
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        # Cleanup
        pass
```

## Quick Tips
```python
# Private variables (name mangling)
class Example:
    def __init__(self):
        self.__private = "Can't touch this"
        self._protected = "Please don't touch"
        self.public = "Touch freely"

# Multiple Inheritance
class Child(Parent1, Parent2):
    pass

# Abstract Class
from abc import ABC, abstractmethod

class Abstract(ABC):
    @abstractmethod
    def must_implement(self):
        pass
```

Remember:
- 🔑 Always use `self` for instance methods
- 📌 Class variables are shared
- 🔒 Use properties for controlled access
- ⭐ Inherit with care
- 💡 Use `super()` to call parent methods
- 🎯 Special methods customize behavior

Common Conventions:
- CamelCase for class names
- snake_case for methods/variables
- `_single_leading` for protected
- `__double_leading` for private
- Properties for getter/setter behavior
- Abstract classes for interfaces

Usage Example:
```python
class BankAccount:
    interest_rate = 0.02  # Class variable
    
    def __init__(self, balance=0):
        self._balance = balance  # Protected
    
    @property
    def balance(self):
        return self._balance
    
    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("No negative balance")
        self._balance = value
    
    def deposit(self, amount):
        self.balance += amount
        return self.balance

# Using the class
account = BankAccount(100)
account.deposit(50)
print(account.balance)  # 150
```