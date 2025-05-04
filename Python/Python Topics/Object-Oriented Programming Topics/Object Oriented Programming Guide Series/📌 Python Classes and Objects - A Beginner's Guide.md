
**Overview:** This guide introduces the fundamental concepts of Object-Oriented Programming (OOP) in Python, focusing on classes and objects. You'll learn how to define classes, create objects, understand the `self` parameter, work with attributes and methods, and apply these concepts to solve real-world problems. By the end of this guide, you'll have a solid foundation in Python OOP principles with practical examples to reinforce your learning.

## 📝 Introduction

Object-Oriented Programming is a programming paradigm that uses "objects" to model real-world entities. Python, being a multi-paradigm language, provides excellent support for OOP. Understanding classes and objects is essential for writing organized, reusable, and maintainable code.

Think of OOP as a way to create blueprints (classes) for creating specific things (objects). Just as an architect creates blueprints before building a house, programmers define classes before creating objects. This approach helps manage complexity in larger programs and promotes code reuse.

## 🧠 Key Concepts

### What is a Class?

A class is a blueprint or template that defines the structure and behavior of objects. It encapsulates data (attributes) and functions (methods) that operate on that data.

### What is an Object?

An object (or instance) is a concrete entity created from a class. If a class is like a cookie cutter, an object is like a cookie made from that cutter.

### Class vs Object Relationship

```
Class (Blueprint) → Object (Instance)
```

![Class and Object Relationship](https://i.imgur.com/JZzTwgL.png)

### The `self` Parameter

In Python, the `self` parameter refers to the instance of the class. It's used to access variables and methods associated with that specific instance.

### Attributes and Methods

- **Attributes**: Variables that belong to a class or object
- **Methods**: Functions that belong to a class or object

## 💻 Implementation Examples

### Basic Class Definition

```python
class Dog:
    # Class attribute (shared by all instances)
    species = "Canis familiaris"
    
    # Initializer / Constructor
    def __init__(self, name, age):
        # Instance attributes (unique to each instance)
        self.name = name
        self.age = age
    
    # Instance method
    def bark(self):
        return f"{self.name} says Woof!"
    
    # Instance method with additional logic
    def dog_info(self):
        return f"{self.name} is {self.age} years old."
```

### Creating Objects (Instances)

```python
# Creating instances of the Dog class
buddy = Dog("Buddy", 5)
max = Dog("Max", 3)

# Accessing attributes
print(buddy.name)  # Output: Buddy
print(max.age)     # Output: 3

# Accessing class attribute
print(buddy.species)  # Output: Canis familiaris
print(max.species)    # Output: Canis familiaris

# Calling methods
print(buddy.bark())      # Output: Buddy says Woof!
print(max.dog_info())    # Output: Max is 3 years old.
```

### A More Practical Example: Bank Account

```python
class BankAccount:
    def __init__(self, account_number, owner_name, balance=0):
        self.account_number = account_number
        self.owner_name = owner_name
        self.balance = balance
    
    def deposit(self, amount):
        if amount > 0:
            self.balance += amount
            return f"Deposited ${amount}. New balance: ${self.balance}"
        return "Deposit amount must be positive"
    
    def withdraw(self, amount):
        if amount > 0:
            if amount <= self.balance:
                self.balance -= amount
                return f"Withdrew ${amount}. New balance: ${self.balance}"
            return "Insufficient funds"
        return "Withdrawal amount must be positive"
    
    def get_balance(self):
        return f"Current balance: ${self.balance}"

# Creating a bank account
account1 = BankAccount("12345", "John Doe", 1000)

# Using the account
print(account1.get_balance())      # Output: Current balance: $1000
print(account1.deposit(500))       # Output: Deposited $500. New balance: $1500
print(account1.withdraw(200))      # Output: Withdrew $200. New balance: $1300
print(account1.withdraw(2000))     # Output: Insufficient funds
```

## 🚀 Best Practices

1. **Use CamelCase for class names**: `BankAccount` instead of `bank_account`
2. **Use descriptive names** for classes, methods, and attributes
3. **Initialize all instance attributes in `__init__`** for clarity
4. **Keep classes focused on a single responsibility**
5. **Document your classes** with docstrings
6. **Validate data** in methods to ensure object integrity

```python
class Student:
    """A class representing a student in a school system."""
    
    def __init__(self, name, student_id, gpa=0.0):
        """
        Initialize a new Student.
        
        Args:
            name (str): The student's full name
            student_id (str): The student's ID number
            gpa (float, optional): Grade point average. Defaults to 0.0
        """
        self.name = name
        self.student_id = student_id
        self.set_gpa(gpa)  # Using method for validation
    
    def set_gpa(self, gpa):
        """Set the student's GPA with validation."""
        if 0.0 <= gpa <= 4.0:
            self.gpa = gpa
        else:
            raise ValueError("GPA must be between 0.0 and 4.0")
```

## 🔍 Common Issues and Debugging

### Forgetting `self` Parameter

```python
# Incorrect
class Car:
    def __init__(model, year):  # Missing self!
        model = model
        year = year

# Correct
class Car:
    def __init__(self, model, year):
        self.model = model
        self.year = year
```

### Confusing Class and Instance Attributes

```python
class Counter:
    count = 0  # Class attribute
    
    def __init__(self):
        self.count = 0  # Instance attribute with same name
    
    def increment(self):
        self.count += 1

# This can lead to confusion
c1 = Counter()
c2 = Counter()
c1.increment()
print(c1.count)  # 1
print(c2.count)  # 0
print(Counter.count)  # 0 (class attribute unchanged)
```

### Modifying Mutable Class Attributes

```python
class Team:
    members = []  # Class attribute (shared by all instances!)
    
    def add_member(self, name):
        self.members.append(name)

# This leads to unexpected behavior
team1 = Team()
team2 = Team()
team1.add_member("Alice")
print(team2.members)  # ['Alice'] - Surprise!

# Correct approach
class Team:
    def __init__(self):
        self.members = []  # Instance attribute
    
    def add_member(self, name):
        self.members.append(name)
```

## 📚 Practice Exercises

### Exercise 1: Create a Rectangle Class
Create a `Rectangle` class with attributes for width and height. Include methods to calculate area and perimeter.

<details>
<summary>Solution</summary>

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# Test
rect = Rectangle(5, 10)
print(f"Area: {rect.area()}")  # Area: 50
print(f"Perimeter: {rect.perimeter()}")  # Perimeter: 30
```
</details>

### Exercise 2: Create a Book Class
Create a `Book` class with attributes for title, author, and pages. Include methods to display book information and to check if the book is long (>300 pages).

<details>
<summary>Solution</summary>

```python
class Book:
    def __init__(self, title, author, pages):
        self.title = title
        self.author = author
        self.pages = pages
    
    def book_info(self):
        return f"'{self.title}' by {self.author}, {self.pages} pages"
    
    def is_long(self):
        return self.pages > 300

# Test
book1 = Book("Python Crash Course", "Eric Matthes", 544)
book2 = Book("Automate the Boring Stuff", "Al Sweigart", 280)

print(book1.book_info())  # 'Python Crash Course' by Eric Matthes, 544 pages
print(f"Is book1 long? {book1.is_long()}")  # Is book1 long? True
print(f"Is book2 long? {book2.is_long()}")  # Is book2 long? False
```
</details>

### Exercise 3: Create a Shopping Cart Class
Create a `ShoppingCart` class that can add items, remove items, and calculate the total price.

<details>
<summary>Solution</summary>

```python
class ShoppingCart:
    def __init__(self):
        self.items = {}  # Dictionary to store item: price pairs
    
    def add_item(self, item, price):
        self.items[item] = price
        return f"Added {item} to cart"
    
    def remove_item(self, item):
        if item in self.items:
            del self.items[item]
            return f"Removed {item} from cart"
        return f"{item} not in cart"
    
    def calculate_total(self):
        return sum(self.items.values())
    
    def show_cart(self):
        if not self.items:
            return "Cart is empty"
        cart_contents = "\n".join([f"{item}: ${price}" for item, price in self.items.items()])
        return f"Cart contents:\n{cart_contents}\nTotal: ${self.calculate_total()}"

# Test
cart = ShoppingCart()
print(cart.add_item("Laptop", 1200))
print(cart.add_item("Headphones", 100))
print(cart.add_item("Mouse", 25))
print(cart.show_cart())
print(cart.remove_item("Headphones"))
print(cart.show_cart())
```
</details>

## 🚀 Further Learning

To deepen your understanding of OOP in Python, explore these topics next:

1. **Inheritance**: Creating new classes from existing ones
2. **Encapsulation**: Restricting access to methods and variables
3. **Polymorphism**: Using a single interface with different underlying forms
4. **Magic methods**: Special methods like `__str__`, `__repr__`, `__eq__`
5. **Property decorators**: Using `@property` for getter/setter methods

### Recommended Resources

- [Python Documentation on Classes](https://docs.python.org/3/tutorial/classes.html)
- Book: "Python Crash Course" by Eric Matthes (Chapter on Classes)
- Book: "Fluent Python" by Luciano Ramalho (for more advanced OOP concepts)
- [Real Python's OOP Articles](https://realpython.com/python3-object-oriented-programming/)

Remember, the best way to learn is by doing! Create your own classes to model real-world problems and continue to practice these concepts.