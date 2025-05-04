**Overview:** This comprehensive guide introduces encapsulation in Python, one of the four pillars of Object-Oriented Programming. You'll learn how Python implements access control through naming conventions, property decorators, and getter/setter methods. By the end of this guide, you'll understand how to protect your class data, validate inputs, and create robust, maintainable code.

## 📝 Introduction

Encapsulation is like putting your class's data in a protective bubble, controlling how the outside world interacts with it. In traditional OOP languages, this is enforced through keywords like `private` and `protected`. Python takes a more trusting approach with its "we're all consenting adults" philosophy, using naming conventions and properties rather than strict access modifiers.

Why is encapsulation important? It:
- Prevents accidental modification of important data
- Allows you to validate inputs before accepting them
- Helps you change internal implementations without breaking external code
- Makes your code more maintainable and robust

## 🧠 Key Concepts

### Access Levels in Python

Python uses naming conventions to indicate access levels:

1. **Public attributes**: Regular names like `self.name`
   - Accessible from anywhere
   - No restrictions on access or modification

2. **Protected attributes**: Names with a single underscore like `self._age`
   - Indicates "please don't access this directly"
   - Still accessible, but signals developer intent
   - A gentle suggestion, not enforced

3. **Private attributes**: Names with double underscores like `self.__password`
   - Triggers name mangling (becomes `_ClassName__password`)
   - Makes external access more difficult (but not impossible)
   - Helps prevent accidental access and name collisions in inheritance

### Property Decorators

Python's `@property` decorator transforms methods into attribute-like objects, allowing you to:
- Create read-only attributes
- Add validation when setting values
- Customize behavior when getting, setting, or deleting attributes

The three main property decorators are:
- `@property`: Defines a getter method
- `@attribute.setter`: Defines a setter method
- `@attribute.deleter`: Defines a deleter method

## 💻 Implementation Examples

### Basic Encapsulation with Naming Conventions

```python
class BankAccount:
    def __init__(self, owner_name, balance=0):
        self.owner_name = owner_name      # Public attribute
        self._balance = balance           # Protected attribute
        self.__account_number = "12345"   # Private attribute
    
    def deposit(self, amount):
        if amount > 0:
            self._balance += amount
            return True
        return False
    
    def withdraw(self, amount):
        if 0 < amount <= self._balance:
            self._balance -= amount
            return True
        return False
    
    def get_balance(self):
        return self._balance
    
    def get_account_info(self):
        # We can access __account_number inside the class
        return f"Account for {self.owner_name}, Number: {self.__account_number}"


# Creating an account
account = BankAccount("Alice", 1000)

# Accessing attributes
print(account.owner_name)  # Works fine: Alice
print(account.get_balance())  # Works fine: 1000
print(account.get_account_info())  # Works fine

# Trying to access protected/private attributes
print(account._balance)  # Works but not recommended: 1000
# print(account.__account_number)  # Error! AttributeError

# Name mangling allows this (but don't do it in practice):
print(account._BankAccount__account_number)  # Works: 12345
```

### Using Properties for Controlled Access

```python
class Person:
    def __init__(self, name, age=0):
        self.name = name    # Public attribute
        self._age = age     # Protected attribute, controlled via property
    
    @property
    def age(self):
        """Getter method for age"""
        return self._age
    
    @age.setter
    def age(self, value):
        """Setter method for age with validation"""
        if not isinstance(value, int):
            raise TypeError("Age must be an integer")
        if value < 0 or value > 120:
            raise ValueError("Age must be between 0 and 120")
        self._age = value
    
    @property
    def is_adult(self):
        """Read-only property derived from age"""
        return self._age >= 18


# Creating a person
person = Person("Bob", 25)

# Using properties
print(person.age)  # Uses the getter: 25
person.age = 30    # Uses the setter
print(person.age)  # 30

print(person.is_adult)  # True (read-only property)

try:
    person.age = -5  # Raises ValueError due to validation
except ValueError as e:
    print(f"Error: {e}")  # Error: Age must be between 0 and 120

try:
    person.is_adult = True  # Raises AttributeError - can't set read-only property
except AttributeError as e:
    print(f"Error: {e}")  # Error: can't set attribute
```

### Complete Example with Data Validation

```python
class Product:
    def __init__(self, name, price, stock=0):
        self.name = name
        self._price = 0     # Initialize with default
        self._stock = 0     # Initialize with default
        
        # Use setters for validation during initialization
        self.price = price
        self.stock = stock
        
        self.__discount_code = None  # Private attribute
    
    @property
    def price(self):
        return self._price
    
    @price.setter
    def price(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Price must be a number")
        if value < 0:
            raise ValueError("Price cannot be negative")
        self._price = float(value)
    
    @property
    def stock(self):
        return self._stock
    
    @stock.setter
    def stock(self, value):
        if not isinstance(value, int):
            raise TypeError("Stock must be an integer")
        if value < 0:
            raise ValueError("Stock cannot be negative")
        self._stock = value
    
    @property
    def is_available(self):
        """Read-only property to check if product is in stock"""
        return self._stock > 0
    
    def set_discount_code(self, code):
        """Method to set private discount code"""
        if code and len(code) >= 5:
            self.__discount_code = code
            return True
        return False
    
    def apply_discount(self, code):
        """Apply discount if code matches"""
        if code and code == self.__discount_code:
            return self._price * 0.9  # 10% discount
        return self._price


# Using the Product class
laptop = Product("Laptop", 999.99, 10)

print(f"Product: {laptop.name}, Price: ${laptop.price}, In Stock: {laptop.stock}")
print(f"Available: {laptop.is_available}")

# Update with validation
try:
    laptop.price = 899.99
    print(f"New price: ${laptop.price}")
    
    laptop.stock = 5
    print(f"New stock: {laptop.stock}")
    
    # This will fail validation
    laptop.price = -50
except ValueError as e:
    print(f"Error: {e}")

# Working with private attribute via methods
laptop.set_discount_code("SUMMER2023")
print(f"Regular price: ${laptop.price}")
print(f"Discounted price: ${laptop.apply_discount('SUMMER2023')}")
print(f"With wrong code: ${laptop.apply_discount('WRONG')}")
```

## 🚀 Best Practices

1. **Use the right level of protection**:
   - Public attributes for data that can be freely accessed
   - Protected attributes (`_name`) for internal data that might be accessed by subclasses
   - Private attributes (`__name`) for data that should be hidden from subclasses

2. **Prefer properties over direct getter/setter methods**:
   - Properties make your code more Pythonic
   - They allow attribute-like access while maintaining control
   - You can start with public attributes and add properties later without changing the interface

3. **Validate data in setters**:
   - Check types and values before assigning
   - Raise appropriate exceptions with clear messages
   - Initialize protected attributes in `__init__` and then use setters for validation

4. **Document your interfaces**:
   - Use docstrings to explain the purpose of properties
   - Clearly indicate what values are acceptable
   - Specify what exceptions might be raised

5. **Don't circumvent encapsulation**:
   - Avoid accessing `_protected` attributes from outside the class
   - Never access `_ClassName__private` attributes from outside the class

## 🔍 Common Issues and Debugging

### Pitfall 1: Forgetting to Use Properties in `__init__`

```python
# Problematic code
class Person:
    def __init__(self, age):
        self._age = age  # No validation happens here!
    
    @property
    def age(self):
        return self._age
    
    @age.setter
    def age(self, value):
        if 0 <= value <= 120:
            self._age = value
        else:
            raise ValueError("Invalid age")

# Better approach
class Person:
    def __init__(self, age):
        self._age = 0  # Default value
        self.age = age  # Uses the setter with validation
```

### Pitfall 2: Inconsistent Access Patterns

```python
# Inconsistent - confusing for users
class Account:
    def __init__(self, balance):
        self._balance = balance
    
    # Sometimes using direct access
    def add_interest(self):
        self._balance *= 1.05
    
    # Sometimes using getters/setters
    def get_balance(self):
        return self._balance
    
    def set_balance(self, value):
        self._balance = value

# Better - consistent property usage
class Account:
    def __init__(self, balance):
        self._balance = balance
    
    @property
    def balance(self):
        return self._balance
    
    @balance.setter
    def balance(self, value):
        self._balance = value
    
    def add_interest(self):
        self.balance *= 1.05  # Uses the property
```

### Pitfall 3: Name Conflicts with Inheritance

```python
class Parent:
    def __init__(self):
        self.__data = "Parent's private data"

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__data = "Child's private data"  # Different variable than Parent.__data!
    
    def access_data(self):
        try:
            return self.__data  # Only accesses Child's __data
        except AttributeError:
            return None

# Solution: Use protected attributes for data that subclasses need to access
class BetterParent:
    def __init__(self):
        self._data = "Parent's protected data"  # Subclasses can access this

class BetterChild(BetterParent):
    def access_data(self):
        return self._data  # Can access parent's _data
```

## 📚 Further Learning

### Practice Exercises

1. **Basic Encapsulation**:
   Create a `Temperature` class that stores temperature in Celsius internally but provides properties to get/set it in both Celsius and Fahrenheit.

2. **Data Validation**:
   Create an `Email` class that validates email addresses when set and raises appropriate exceptions for invalid formats.

3. **Complete Class with Encapsulation**:
   Design a `CreditCard` class with private attributes for card number and CVV, protected attributes for expiration date, and public methods for making purchases and checking balance.

### Exercise Solutions

#### Exercise 1: Temperature Class

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = 0
        self.celsius = celsius  # Use the setter for validation
    
    @property
    def celsius(self):
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Temperature must be a number")
        self._celsius = float(value)
    
    @property
    def fahrenheit(self):
        return (self._celsius * 9/5) + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        if not isinstance(value, (int, float)):
            raise TypeError("Temperature must be a number")
        self._celsius = (float(value) - 32) * 5/9

# Testing the Temperature class
temp = Temperature(25)
print(f"{temp.celsius}°C = {temp.fahrenheit}°F")  # 25.0°C = 77.0°F

temp.fahrenheit = 68
print(f"{temp.celsius}°C = {temp.fahrenheit}°F")  # 20.0°C = 68.0°F
```

#### Exercise 2: Email Validation

```python
import re

class Email:
    def __init__(self, address=None):
        self._address = None
        if address:
            self.address = address
    
    @property
    def address(self):
        return self._address
    
    @address.setter
    def address(self, value):
        if not isinstance(value, str):
            raise TypeError("Email address must be a string")
        
        # Simple regex for email validation
        pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if not re.match(pattern, value):
            raise ValueError("Invalid email format")
        
        self._address = value
    
    @property
    def domain(self):
        """Get the domain part of the email address"""
        if self._address:
            return self._address.split('@')[1]
        return None

# Testing the Email class
try:
    email = Email("user@example.com")
    print(f"Email: {email.address}")
    print(f"Domain: {email.domain}")
    
    email.address = "new.user@company.org"
    print(f"New email: {email.address}")
    
    # This should fail validation
    email.address = "invalid-email"
except ValueError as e:
    print(f"Error: {e}")
```

#### Exercise 3: CreditCard Class

```python
from datetime import datetime

class CreditCard:
    def __init__(self, card_number, expiry_date, cvv, balance=0):
        # Private attributes
        self.__card_number = None
        self.__cvv = None
        
        # Protected attributes
        self._expiry_date = None
        self._balance = 0
        
        # Set values with validation
        self.set_card_details(card_number, expiry_date, cvv)
        self._balance = balance
    
    def set_card_details(self, card_number, expiry_date, cvv):
        """Set card details with validation"""
        # Validate card number (simplified)
        if not (isinstance(card_number, str) and len(card_number) == 16 and card_number.isdigit()):
            raise ValueError("Card number must be a 16-digit string")
        self.__card_number = card_number
        
        # Validate expiry date (MM/YY format)
        if not (isinstance(expiry_date, str) and len(expiry_date) == 5 and expiry_date[2] == '/'):
            raise ValueError("Expiry date must be in MM/YY format")
        
        # Check if card is expired
        current_date = datetime.now()
        exp_month = int(expiry_date[:2])
        exp_year = int('20' + expiry_date[3:])
        
        if exp_year < current_date.year or (exp_year == current_date.year and exp_month < current_date.month):
            raise ValueError("Card has expired")
        
        self._expiry_date = expiry_date
        
        # Validate CVV
        if not (isinstance(cvv, str) and len(cvv) == 3 and cvv.isdigit()):
            raise ValueError("CVV must be a 3-digit string")
        self.__cvv = cvv
    
    @property
    def balance(self):
        """Get current balance"""
        return self._balance
    
    @property
    def card_last_four(self):
        """Get last four digits of card number"""
        if self.__card_number:
            return self.__card_number[-4:]
        return None
    
    @property
    def expiry_date(self):
        """Get expiry date"""
        return self._expiry_date
    
    def make_purchase(self, amount, provided_cvv=None):
        """Make a purchase with the card"""
        if not isinstance(amount, (int, float)) or amount <= 0:
            raise ValueError("Amount must be a positive number")
        
        # For security, require CVV for purchases
        if provided_cvv != self.__cvv:
            return False, "Invalid CVV"
        
        if amount > self._balance:
            return False, "Insufficient funds"
        
        self._balance -= amount
        return True, f"Purchase successful. New balance: ${self._balance}"
    
    def add_funds(self, amount):
        """Add funds to the card"""
        if not isinstance(amount, (int, float)) or amount <= 0:
            raise ValueError("Amount must be a positive number")
        
        self._balance += amount
        return f"Added ${amount}. New balance: ${self._balance}"
    
    def get_info(self):
        """Get card information"""
        return {
            "last_four": self.card_last_four,
            "expiry": self.expiry_date,
            "balance": self.balance
        }


# Testing the CreditCard class
try:
    # Create a valid card
    card = CreditCard("1234567890123456", "12/25", "123", 1000)
    
    print(f"Card info: {card.get_info()}")
    
    # Make a purchase
    success, message = card.make_purchase(500, "123")
    print(message)  # Purchase successful. New balance: $500
    
    # Try with wrong CVV
    success, message = card.make_purchase(200, "999")
    print(message)  # Invalid CVV
    
    # Add funds
    print(card.add_funds(300))  # Added $300. New balance: $800
    
    # Try to access private attributes
    # print(card.__card_number)  # This would raise an AttributeError
    
    # Create an invalid card
    # invalid_card = CreditCard("123", "01/20", "12")  # This would raise ValueError
    
except ValueError as e:
    print(f"Error: {e}")
```

### Visual Representation of Encapsulation

```
┌─────────────────────────────────────────┐
│                 Class                   │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │        Private Attributes       │    │
│  │     (self.__private_attr)       │    │
│  │                                 │    │
│  │    Only accessible within       │    │
│  │    the class itself             │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │       Protected Attributes      │    │
│  │      (self._protected_attr)     │    │
│  │                                 │    │
│  │    Accessible within class      │    │
│  │    and subclasses               │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │        Public Interface         │    │
│  │                                 │    │
│  │    - Public Attributes          │    │
│  │    - Properties (@property)     │    │
│  │    - Public Methods             │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
            │
            │ Interacts with
            ▼
┌─────────────────────────────────────────┐
│           External Code                 │
│                                         │
│  Only interacts with the public         │
│  interface of the class                 │
└─────────────────────────────────────────┘
```

### Additional Resources

1. **Python Documentation**:
   - [Python Property Decorator](https://docs.python.org/3/library/functions.html#property)
   - [Python Data Model](https://docs.python.org/3/reference/datamodel.html)

2. **Books**:
   - "Fluent Python" by Luciano Ramalho (Chapter on Python Object Model)
   - "Python Cookbook" by David Beazley and Brian K. Jones (Chapter on Classes and Objects)

3. **Online Tutorials**:
   - Real Python: [Object-Oriented Programming in Python 3](https://realpython.com/python3-object-oriented-programming/)
   - Python Course: [Properties vs. Getters and Setters](https://www.python-course.eu/python3_properties.php)

Remember, encapsulation in Python is about communicating intent rather than enforcing strict rules. The language trusts developers to respect these conventions, which is why understanding them thoroughly is so important!