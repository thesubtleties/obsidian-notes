# Factory and Abstract Factory Patterns

#programming #patterns #creational #factory #design-patterns

## What are Factory Patterns?

Factory patterns are creational design patterns that provide ways to create objects without specifying their exact classes. They encapsulate object creation logic, making code more flexible and maintainable.

## Simple Factory Pattern

The simplest form - a method that creates objects based on input parameters.

### Python Example
```python
from abc import ABC, abstractmethod
from typing import Dict, Type

# Product interface
class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass
    
    @abstractmethod
    def move(self) -> str:
        pass

# Concrete products
class Dog(Animal):
    def speak(self) -> str:
        return "Woof!"
    
    def move(self) -> str:
        return "Runs on four legs"

class Cat(Animal):
    def speak(self) -> str:
        return "Meow!"
    
    def move(self) -> str:
        return "Walks gracefully"

class Bird(Animal):
    def speak(self) -> str:
        return "Chirp!"
    
    def move(self) -> str:
        return "Flies through the air"

# Simple Factory
class AnimalFactory:
    """Simple factory for creating animals"""
    
    _animals: Dict[str, Type[Animal]] = {
        'dog': Dog,
        'cat': Cat,
        'bird': Bird
    }
    
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        """Factory method to create animals"""
        animal_class = AnimalFactory._animals.get(animal_type.lower())
        
        if not animal_class:
            raise ValueError(f"Unknown animal type: {animal_type}")
        
        return animal_class()
    
    @staticmethod
    def register_animal(animal_type: str, animal_class: Type[Animal]):
        """Register new animal types dynamically"""
        AnimalFactory._animals[animal_type.lower()] = animal_class

# Usage
dog = AnimalFactory.create_animal('dog')
print(f"Dog says: {dog.speak()}")
print(f"Dog movement: {dog.move()}")

# Extensibility - add new animal without modifying factory
class Fish(Animal):
    def speak(self) -> str:
        return "Blub blub"
    
    def move(self) -> str:
        return "Swims in water"

AnimalFactory.register_animal('fish', Fish)
fish = AnimalFactory.create_animal('fish')
print(f"Fish says: {fish.speak()}")
```

### TypeScript Example
```typescript
// Product interface
interface Animal {
    speak(): string;
    move(): string;
}

// Concrete products
class Dog implements Animal {
    speak(): string {
        return "Woof!";
    }
    
    move(): string {
        return "Runs on four legs";
    }
}

class Cat implements Animal {
    speak(): string {
        return "Meow!";
    }
    
    move(): string {
        return "Walks gracefully";
    }
}

class Bird implements Animal {
    speak(): string {
        return "Chirp!";
    }
    
    move(): string {
        return "Flies through the air";
    }
}

// Simple Factory
class AnimalFactory {
    private static animals: Map<string, new() => Animal> = new Map([
        ['dog', Dog],
        ['cat', Cat],
        ['bird', Bird]
    ]);
    
    static createAnimal(animalType: string): Animal {
        const AnimalClass = this.animals.get(animalType.toLowerCase());
        
        if (!AnimalClass) {
            throw new Error(`Unknown animal type: ${animalType}`);
        }
        
        return new AnimalClass();
    }
    
    static registerAnimal(animalType: string, animalClass: new() => Animal): void {
        this.animals.set(animalType.toLowerCase(), animalClass);
    }
}

// Usage
const dog = AnimalFactory.createAnimal('dog');
console.log(`Dog says: ${dog.speak()}`);
```

## Factory Method Pattern

Defines an interface for creating objects but lets subclasses decide which class to instantiate.

### Python Example
```python
from abc import ABC, abstractmethod

# Product hierarchy
class Button(ABC):
    @abstractmethod
    def render(self) -> str:
        pass
    
    @abstractmethod
    def click(self) -> str:
        pass

class WindowsButton(Button):
    def render(self) -> str:
        return "[ Windows Button ]"
    
    def click(self) -> str:
        return "Windows button clicked!"

class MacButton(Button):
    def render(self) -> str:
        return "( Mac Button )"
    
    def click(self) -> str:
        return "Mac button clicked!"

class LinuxButton(Button):
    def render(self) -> str:
        return "< Linux Button >"
    
    def click(self) -> str:
        return "Linux button clicked!"

# Creator hierarchy
class Dialog(ABC):
    """Abstract creator"""
    
    @abstractmethod
    def create_button(self) -> Button:
        """Factory method"""
        pass
    
    def render_dialog(self) -> str:
        """Template method using factory method"""
        button = self.create_button()
        return f"Dialog with {button.render()}"

class WindowsDialog(Dialog):
    def create_button(self) -> Button:
        return WindowsButton()

class MacDialog(Dialog):
    def create_button(self) -> Button:
        return MacButton()

class LinuxDialog(Dialog):
    def create_button(self) -> Button:
        return LinuxButton()

# Application code
def create_dialog(os_type: str) -> Dialog:
    """Create appropriate dialog for OS"""
    dialogs = {
        'windows': WindowsDialog,
        'mac': MacDialog,
        'linux': LinuxDialog
    }
    
    dialog_class = dialogs.get(os_type.lower())
    if not dialog_class:
        raise ValueError(f"Unsupported OS: {os_type}")
    
    return dialog_class()

# Usage
dialog = create_dialog('windows')
print(dialog.render_dialog())
button = dialog.create_button()
print(button.click())
```

## Abstract Factory Pattern

Provides an interface for creating families of related objects without specifying their concrete classes.

### Python Example
```python
from abc import ABC, abstractmethod

# Abstract products
class Button(ABC):
    @abstractmethod
    def paint(self) -> str:
        pass

class Checkbox(ABC):
    @abstractmethod
    def check(self) -> str:
        pass

class ScrollBar(ABC):
    @abstractmethod
    def scroll(self) -> str:
        pass

# Concrete products - Windows family
class WindowsButton(Button):
    def paint(self) -> str:
        return "Rendering Windows button"

class WindowsCheckbox(Checkbox):
    def check(self) -> str:
        return "Checking Windows checkbox"

class WindowsScrollBar(ScrollBar):
    def scroll(self) -> str:
        return "Scrolling Windows scrollbar"

# Concrete products - Mac family
class MacButton(Button):
    def paint(self) -> str:
        return "Rendering Mac button"

class MacCheckbox(Checkbox):
    def check(self) -> str:
        return "Checking Mac checkbox"

class MacScrollBar(ScrollBar):
    def scroll(self) -> str:
        return "Scrolling Mac scrollbar"

# Abstract factory
class GUIFactory(ABC):
    @abstractmethod
    def create_button(self) -> Button:
        pass
    
    @abstractmethod
    def create_checkbox(self) -> Checkbox:
        pass
    
    @abstractmethod
    def create_scrollbar(self) -> ScrollBar:
        pass

# Concrete factories
class WindowsFactory(GUIFactory):
    def create_button(self) -> Button:
        return WindowsButton()
    
    def create_checkbox(self) -> Checkbox:
        return WindowsCheckbox()
    
    def create_scrollbar(self) -> ScrollBar:
        return WindowsScrollBar()

class MacFactory(GUIFactory):
    def create_button(self) -> Button:
        return MacButton()
    
    def create_checkbox(self) -> Checkbox:
        return MacCheckbox()
    
    def create_scrollbar(self) -> ScrollBar:
        return MacScrollBar()

# Client code
class Application:
    def __init__(self, factory: GUIFactory):
        self.factory = factory
        self.button = None
        self.checkbox = None
        self.scrollbar = None
    
    def create_ui(self):
        """Create UI elements using factory"""
        self.button = self.factory.create_button()
        self.checkbox = self.factory.create_checkbox()
        self.scrollbar = self.factory.create_scrollbar()
    
    def paint(self):
        """Use created elements"""
        print(self.button.paint())
        print(self.checkbox.check())
        print(self.scrollbar.scroll())

# Usage
def create_application(os_type: str) -> Application:
    factories = {
        'windows': WindowsFactory(),
        'mac': MacFactory()
    }
    
    factory = factories.get(os_type.lower())
    if not factory:
        raise ValueError(f"Unsupported OS: {os_type}")
    
    app = Application(factory)
    app.create_ui()
    return app

# Create and use application
app = create_application('windows')
app.paint()
```

### TypeScript Example
```typescript
// Abstract products
interface Button {
    paint(): string;
}

interface Checkbox {
    check(): string;
}

interface ScrollBar {
    scroll(): string;
}

// Concrete products - Material Design family
class MaterialButton implements Button {
    paint(): string {
        return "Rendering Material Design button";
    }
}

class MaterialCheckbox implements Checkbox {
    check(): string {
        return "Checking Material Design checkbox";
    }
}

class MaterialScrollBar implements ScrollBar {
    scroll(): string {
        return "Scrolling Material Design scrollbar";
    }
}

// Concrete products - Bootstrap family
class BootstrapButton implements Button {
    paint(): string {
        return "Rendering Bootstrap button";
    }
}

class BootstrapCheckbox implements Checkbox {
    check(): string {
        return "Checking Bootstrap checkbox";
    }
}

class BootstrapScrollBar implements ScrollBar {
    scroll(): string {
        return "Scrolling Bootstrap scrollbar";
    }
}

// Abstract factory
interface UIFactory {
    createButton(): Button;
    createCheckbox(): Checkbox;
    createScrollBar(): ScrollBar;
}

// Concrete factories
class MaterialUIFactory implements UIFactory {
    createButton(): Button {
        return new MaterialButton();
    }
    
    createCheckbox(): Checkbox {
        return new MaterialCheckbox();
    }
    
    createScrollBar(): ScrollBar {
        return new MaterialScrollBar();
    }
}

class BootstrapUIFactory implements UIFactory {
    createButton(): Button {
        return new BootstrapButton();
    }
    
    createCheckbox(): Checkbox {
        return new BootstrapCheckbox();
    }
    
    createScrollBar(): ScrollBar {
        return new BootstrapScrollBar();
    }
}

// Client code
class Application {
    private button: Button;
    private checkbox: Checkbox;
    private scrollbar: ScrollBar;
    
    constructor(private factory: UIFactory) {}
    
    createUI(): void {
        this.button = this.factory.createButton();
        this.checkbox = this.factory.createCheckbox();
        this.scrollbar = this.factory.createScrollBar();
    }
    
    render(): void {
        console.log(this.button.paint());
        console.log(this.checkbox.check());
        console.log(this.scrollbar.scroll());
    }
}
```

## Real-World Examples

### Database Connection Factory
```python
class DatabaseFactory:
    """Factory for creating database connections"""
    
    @staticmethod
    def create_connection(db_type: str, **config):
        if db_type == 'postgresql':
            import psycopg2
            return psycopg2.connect(
                host=config.get('host', 'localhost'),
                port=config.get('port', 5432),
                database=config.get('database'),
                user=config.get('user'),
                password=config.get('password')
            )
        
        elif db_type == 'mysql':
            import mysql.connector
            return mysql.connector.connect(
                host=config.get('host', 'localhost'),
                port=config.get('port', 3306),
                database=config.get('database'),
                user=config.get('user'),
                password=config.get('password')
            )
        
        elif db_type == 'sqlite':
            import sqlite3
            return sqlite3.connect(config.get('database', ':memory:'))
        
        else:
            raise ValueError(f"Unsupported database type: {db_type}")

# Usage
db_config = {
    'database': 'myapp',
    'user': 'admin',
    'password': 'secret'
}

conn = DatabaseFactory.create_connection('postgresql', **db_config)
```

### Parser Factory
```typescript
// Parser factory for different file formats
interface Parser {
    parse(content: string): any;
}

class JSONParser implements Parser {
    parse(content: string): any {
        return JSON.parse(content);
    }
}

class XMLParser implements Parser {
    parse(content: string): any {
        // XML parsing logic
        return { parsed: "xml" };
    }
}

class YAMLParser implements Parser {
    parse(content: string): any {
        // YAML parsing logic
        return { parsed: "yaml" };
    }
}

class ParserFactory {
    private static parsers = new Map<string, new() => Parser>([
        ['json', JSONParser],
        ['xml', XMLParser],
        ['yaml', YAMLParser],
        ['yml', YAMLParser]
    ]);
    
    static createParser(fileExtension: string): Parser {
        const ParserClass = this.parsers.get(fileExtension.toLowerCase());
        
        if (!ParserClass) {
            throw new Error(`No parser available for .${fileExtension} files`);
        }
        
        return new ParserClass();
    }
    
    static createFromFilename(filename: string): Parser {
        const extension = filename.split('.').pop() || '';
        return this.createParser(extension);
    }
}

// Usage
const parser = ParserFactory.createFromFilename('config.json');
const data = parser.parse('{"name": "app"}');
```

## When to Use Factory Patterns

### Simple Factory
> [!tip] Use When
> - Object creation is straightforward
> - You need a central place for creation logic
> - The types are known at compile time

### Factory Method
> [!tip] Use When
> - Subclasses need to choose the type to create
> - You want to delegate creation to subclasses
> - Following the Open/Closed Principle

### Abstract Factory
> [!tip] Use When
> - Creating families of related objects
> - You need to ensure objects work together
> - Supporting multiple product lines

## Benefits

> [!info] Advantages
> 1. **Encapsulation** - Creation logic in one place
> 2. **Flexibility** - Easy to add new types
> 3. **Decoupling** - Client doesn't know concrete classes
> 4. **Consistency** - Ensures compatible objects
> 5. **Testability** - Easy to mock factories

## Common Pitfalls

> [!warning] Watch Out For
> 1. **Over-engineering** - Don't use factories for simple cases
> 2. **Type explosion** - Too many factory classes
> 3. **Tight coupling** to factory interface
> 4. **Complexity** - Can make code harder to understand