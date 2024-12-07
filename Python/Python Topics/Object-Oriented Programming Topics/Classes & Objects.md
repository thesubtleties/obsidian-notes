## Basic Class Structure
```python
class Car:
    # Class variable (shared by all instances)
    total_cars = 0
    
    # Constructor
    def __init__(self, make: str, model: str, year: int):
        # Instance variables (unique to each instance)
        self.make = make
        self.model = model
        self.year = year
        self._mileage = 0  # Protected attribute (by convention)
        self.__vin = None  # Private attribute (name mangling)
        Car.total_cars += 1
    
    # Instance method
    def drive(self, miles: int) -> None:
        self._mileage += miles
        
    # Property getter
    @property
    def mileage(self) -> int:
        return self._mileage
```
Basic class structure showing instance creation, variables, and methods. Class variables are shared among all instances, while instance variables are unique to each object.

## Instance vs Class Methods
```python
class Employee:
    # Class variable
    company = "TechCorp"
    
    def __init__(self, name: str, salary: float):
        self.name = name
        self.salary = salary
    
    # Instance method (has access to instance attributes)
    def get_salary(self) -> float:
        return self.salary
    
    # Class method (has access to class attributes)
    @classmethod
    def set_company(cls, new_company: str) -> None:
        cls.company = new_company
    
    # Static method (no access to instance or class attributes)
    @staticmethod
    def is_valid_salary(salary: float) -> bool:
        return salary > 0
```
Demonstrates different types of methods and their uses. Instance methods can access instance data, class methods can modify class state, and static methods are utility functions.

## Inheritance and Method Override
```python
class Vehicle:
    def __init__(self, brand: str):
        self.brand = brand
    
    def start_engine(self) -> str:
        return "Engine starting..."

class ElectricCar(Vehicle):
    def __init__(self, brand: str, battery_size: int):
        # Call parent constructor
        super().__init__(brand)
        self.battery_size = battery_size
    
    # Override parent method
    def start_engine(self) -> str:
        return "Silent start... Electric motor running"
    
    # Add new method
    def charge(self) -> str:
        return f"Charging {self.battery_size}kWh battery"
```
Shows how to inherit from parent classes, override methods, and extend functionality.

## Properties and Encapsulation
```python
class BankAccount:
    def __init__(self, owner: str, initial_balance: float = 0):
        self._owner = owner
        self._balance = initial_balance
        self._transactions = []
    
    @property
    def balance(self) -> float:
        """Get current balance"""
        return self._balance
    
    @balance.setter
    def balance(self, value: float) -> None:
        """Set balance with validation"""
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._transactions.append(value - self._balance)
        self._balance = value
    
    @property
    def transaction_history(self) -> list:
        """Read-only property"""
        return self._transactions.copy()
```
Demonstrates proper encapsulation using properties, including read-only properties and validation.

## Class Composition
```python
class Engine:
    def start(self) -> str:
        return "Engine running"
    
    def stop(self) -> str:
        return "Engine stopped"

class Transmission:
    def shift(self, gear: int) -> str:
        return f"Shifting to gear {gear}"

class Car:
    def __init__(self):
        # Composition
        self.engine = Engine()
        self.transmission = Transmission()
    
    def start(self) -> str:
        return self.engine.start()
    
    def shift_gear(self, gear: int) -> str:
        return self.transmission.shift(gear)
```
Shows how to compose complex objects from simpler ones, demonstrating the "has-a" relationship.

## Class Factories and Alternative Constructors
```python
class Pizza:
    def __init__(self, size: str, toppings: list):
        self.size = size
        self.toppings = toppings
    
    @classmethod
    def margherita(cls, size: str) -> 'Pizza':
        """Factory method for creating a margherita pizza"""
        return cls(size, ['tomato', 'mozzarella', 'basil'])
    
    @classmethod
    def from_dict(cls, data: dict) -> 'Pizza':
        """Alternative constructor from dictionary"""
        return cls(data['size'], data['toppings'])
    
    def add_topping(self, topping: str) -> None:
        self.toppings.append(topping)
```
Demonstrates factory methods and alternative constructors for creating objects in different ways.

## Private and Protected Members
```python
class SecureData:
    def __init__(self, public_data: str, private_data: str):
        self.public_data = public_data        # Public
        self._protected_data = private_data   # Protected (convention)
        self.__private_data = private_data    # Private (name mangling)
    
    def _internal_method(self) -> str:
        """Protected method"""
        return self._protected_data
    
    def __very_private(self) -> str:
        """Private method"""
        return self.__private_data
    
    def get_private_data(self) -> str:
        """Public interface to private data"""
        return self.__very_private()
```
Shows Python's conventions for data hiding and encapsulation.

## Object Representation
```python
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
    
    def __str__(self) -> str:
        """Informal string representation"""
        return f"{self.name}, {self.age} years old"
    
    def __repr__(self) -> str:
        """Official string representation"""
        return f"Person(name='{self.name}', age={self.age})"
    
    def __format__(self, format_spec: str) -> str:
        """Custom string formatting"""
        if format_spec == "brief":
            return self.name
        return str(self)
```
Shows how to customize string representation of objects for different contexts.

## Class Decorators
```python
def singleton(cls):
    """Decorator to create singleton classes"""
    instances = {}
    
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]
    
    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self, host: str):
        self.host = host
        print(f"Connecting to {host}")
```
Demonstrates how to modify class behavior using decorators.

## Best Practices
```python
class BestPracticesExample:
    """
    Class demonstrating Python best practices.
    
    Attributes:
        name (str): The name of the instance
        value (int): The value associated with the instance
    """
    
    def __init__(self, name: str, value: int):
        self.name = name
        self._value = value
    
    @property
    def value(self) -> int:
        """Get the current value."""
        return self._value
    
    @value.setter
    def value(self, new_value: int) -> None:
        """Set the value with validation."""
        if new_value < 0:
            raise ValueError("Value cannot be negative")
        self._value = new_value
    
    def __str__(self) -> str:
        return f"{self.name}: {self._value}"
```
Demonstrates Python coding conventions, documentation, and type hints.

This overview covers the main aspects of working with classes and objects in Python, from basic structure to advanced features and best practices.