## Basic Property Usage
```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius    # Protected variable convention
    
    # Basic property getter
    @property
    def celsius(self) -> float:
        return self._celsius
    
    # Property setter
    @celsius.setter
    def celsius(self, value: float) -> None:
        if value < -273.15:    # Absolute zero validation
            raise ValueError("Temperature cannot be below absolute zero!")
        self._celsius = value
    
    # Property that performs calculation
    @property
    def fahrenheit(self) -> float:
        return (self._celsius * 9/5) + 32

# Usage
temp = Temperature(25)
print(temp.celsius)     # 25
temp.celsius = 30       # Uses setter
print(temp.fahrenheit)  # 86
```
Properties provide a way to use methods like attributes. They're similar to JavaScript getters/setters but more explicit and commonly used in Python.

## Property Patterns
```python
class Circle:
    def __init__(self, radius: float):
        self._radius = radius
        self._area = None      # Cache for computed property
    
    @property
    def radius(self) -> float:
        return self._radius
    
    @radius.setter
    def radius(self, value: float) -> None:
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value
        self._area = None      # Invalidate cache
    
    @property
    def area(self) -> float:
        # Cached property pattern
        if self._area is None:
            self._area = 3.14159 * self._radius ** 2
        return self._area

# Usage
circle = Circle(5)
print(circle.area)      # Calculated first time
print(circle.area)      # Returns cached value
circle.radius = 10      # Cache invalidated
```
Common patterns include validation, computed properties, and caching results.

## Class and Static Methods
```python
class BankAccount:
    interest_rate = 0.02    # Class variable
    
    def __init__(self, balance: float):
        self.balance = balance
    
    # Instance method (has access to self)
    def add_interest(self) -> None:
        self.balance += self.balance * self.interest_rate
    
    # Class method (has access to class)
    @classmethod
    def set_interest_rate(cls, rate: float) -> None:
        if not 0 <= rate <= 0.2:
            raise ValueError("Invalid interest rate")
        cls.interest_rate = rate
    
    # Static method (no access to instance or class)
    @staticmethod
    def validate_amount(amount: float) -> bool:
        return amount > 0

# Usage
account = BankAccount(1000)
BankAccount.set_interest_rate(0.03)    # Class method
account.add_interest()                 # Instance method
BankAccount.validate_amount(500)       # Static method
```
Different types of methods serve different purposes. Class methods often used as alternative constructors.

## Property with All Decorators
```python
class User:
    def __init__(self, username: str):
        self._username = username
    
    @property
    def username(self) -> str:
        """Get the username."""
        return self._username
    
    @username.setter
    def username(self, value: str) -> None:
        """Set the username with validation."""
        if not value.isalnum():
            raise ValueError("Username must be alphanumeric")
        self._username = value
    
    @username.deleter
    def username(self) -> None:
        """Handle deletion of username."""
        self._username = None

# Usage
user = User("john123")
print(user.username)    # Property getter
user.username = "jane123"  # Property setter
del user.username      # Property deleter
```
Full property implementation includes getter, setter, and deleter methods.

## Factory Methods with Class Decorators
```python
class Date:
    def __init__(self, year: int, month: int, day: int):
        self.year = year
        self.month = month
        self.day = day
    
    @classmethod
    def from_string(cls, date_str: str) -> 'Date':
        """Alternative constructor from string."""
        year, month, day = map(int, date_str.split('-'))
        return cls(year, month, day)
    
    @classmethod
    def today(cls) -> 'Date':
        """Factory method for current date."""
        import datetime
        today = datetime.date.today()
        return cls(today.year, today.month, today.day)

# Usage
date1 = Date(2023, 12, 25)           # Regular constructor
date2 = Date.from_string("2023-12-25")  # Class method
date3 = Date.today()                    # Factory method
```
Class methods are commonly used to provide alternative ways to create objects.

## Common Patterns and Gotchas
```python
class CommonPatterns:
    def __init__(self):
        self._value = None
        self._cached_result = None
    
    # Read-only property
    @property
    def value(self):
        return self._value
    
    # Property with validation
    @property
    def validated_value(self):
        return self._value
    
    @validated_value.setter
    def validated_value(self, value):
        if value < 0:
            raise ValueError("Must be positive")
        self._value = value
    
    # Cached property
    @property
    def expensive_calculation(self):
        if self._cached_result is None:
            # Perform expensive calculation
            self._cached_result = sum(range(10000))
        return self._cached_result
```

## Best Practices
1. Use properties instead of getter/setter methods
2. Use properties for computed values
3. Use class methods for alternative constructors
4. Use static methods for utility functions
5. Cache expensive computations
6. Clear cache when dependent values change

## Gotchas
```python
class Gotchas:
    # BAD: Infinite recursion
    @property
    def bad_property(self):
        return self.bad_property  # Calls itself!
    
    # GOOD: Use different name
    @property
    def good_property(self):
        return self._good_property
    
    # BAD: Mutable default in class method
    @classmethod
    def bad_factory(cls, items=[]):  # Mutable default!
        return cls()
    
    # GOOD: Use None as default
    @classmethod
    def good_factory(cls, items=None):
        items = items or []
        return cls()
```

This covers the main aspects of properties and decorators in Python, focusing on practical usage and common patterns.