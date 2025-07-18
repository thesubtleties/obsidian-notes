# Hidden Dependencies

#programming #antipattern #dependencies #architecture #testing

## What are Hidden Dependencies?

**Hidden Dependencies** occur when a component (function, class, or module) relies on external state or behavior that isn't explicitly declared in its interface. The dependency is "hidden" because you can't tell from the function signature or class constructor what the code actually needs to work.

## Computer Science Definition

A hidden dependency is a coupling between components where the dependency relationship is not explicitly declared through parameters, constructor arguments, or interfaces. This violates the Principle of Explicit Dependencies and makes code harder to understand, test, and maintain.

## Examples of Hidden Dependencies

### Python Examples
```python
# BAD: Hidden dependencies
import os
import requests
from datetime import datetime

class EmailService:
    def send_email(self, to, subject, body):
        # Hidden dependency on environment variable
        api_key = os.environ['SENDGRID_API_KEY']  # Hidden!
        
        # Hidden dependency on global time
        timestamp = datetime.now()  # Hidden!
        
        # Hidden dependency on external service
        response = requests.post(  # Hidden!
            'https://api.sendgrid.com/v3/mail/send',
            headers={'Authorization': f'Bearer {api_key}'},
            json={
                'to': to,
                'subject': subject,
                'body': body,
                'sent_at': timestamp.isoformat()
            }
        )
        return response.status_code == 200

# GOOD: Explicit dependencies
class EmailService:
    def __init__(self, api_key, http_client, clock):
        self.api_key = api_key  # Explicit!
        self.http_client = http_client  # Explicit!
        self.clock = clock  # Explicit!
    
    def send_email(self, to, subject, body):
        timestamp = self.clock.now()
        
        response = self.http_client.post(
            'https://api.sendgrid.com/v3/mail/send',
            headers={'Authorization': f'Bearer {self.api_key}'},
            json={
                'to': to,
                'subject': subject,
                'body': body,
                'sent_at': timestamp.isoformat()
            }
        )
        return response.status_code == 200
```

### TypeScript Examples
```typescript
// BAD: Hidden dependencies
class OrderProcessor {
    processOrder(order: Order): void {
        // Hidden dependency on global database
        const db = Database.getInstance();  // Hidden!
        
        // Hidden dependency on global logger
        console.log(`Processing order ${order.id}`);  // Hidden!
        
        // Hidden dependency on static configuration
        const tax = order.total * TaxConfig.RATE;  // Hidden!
        
        db.save({...order, tax});
    }
}

// GOOD: Explicit dependencies
class OrderProcessor {
    constructor(
        private db: IDatabase,
        private logger: ILogger,
        private taxCalculator: ITaxCalculator
    ) {}  // All dependencies explicit!
    
    processOrder(order: Order): void {
        this.logger.log(`Processing order ${order.id}`);
        const tax = this.taxCalculator.calculate(order.total);
        this.db.save({...order, tax});
    }
}
```

## Types of Hidden Dependencies

### 1. **Environmental Dependencies**
```python
# Hidden dependency on environment
def get_database_url():
    return os.environ['DATABASE_URL']  # What if it's not set?

# Explicit alternative
def get_database_url(env_vars):
    return env_vars.get('DATABASE_URL', 'sqlite:///default.db')
```

### 2. **Temporal Dependencies**
```python
# Hidden dependency on current time
def create_backup():
    filename = f"backup_{datetime.now().timestamp()}.zip"
    
# Explicit alternative
def create_backup(timestamp):
    filename = f"backup_{timestamp}.zip"
```

### 3. **Static/Global Dependencies**
```python
# Hidden dependency on class state
class Logger:
    level = "INFO"
    
    @staticmethod
    def log(message):
        if Logger.level == "DEBUG":  # Hidden dependency!
            print(f"[DEBUG] {message}")

# Explicit alternative
class Logger:
    def __init__(self, level="INFO"):
        self.level = level
    
    def log(self, message):
        if self.level == "DEBUG":
            print(f"[DEBUG] {message}")
```

### 4. **Framework Dependencies**
```python
# Flask example - hidden dependency on request context
from flask import request

def get_user_id():
    return request.headers.get('X-User-ID')  # Hidden dependency on Flask's request!

# Explicit alternative
def get_user_id(headers):
    return headers.get('X-User-ID')
```

## Why Hidden Dependencies are Problematic

### 1. **Testing Nightmares**
```python
# How do you test this?
def calculate_discount():
    if datetime.now().weekday() == 4:  # Friday
        return 0.2
    return 0.1

# Testable version
def calculate_discount(current_date):
    if current_date.weekday() == 4:
        return 0.2
    return 0.1

# Now you can test easily
assert calculate_discount(datetime(2024, 1, 5)) == 0.2  # Friday
assert calculate_discount(datetime(2024, 1, 4)) == 0.1  # Thursday
```

### 2. **Coupling and Brittleness**
Hidden dependencies create tight coupling between unrelated parts of your code.

### 3. **Unclear Requirements**
You can't tell what a component needs just by looking at its interface.

## Conceptual Understanding & Analogies

### The Recipe Analogy
A hidden dependency is like a recipe that says "bake until golden" but doesn't mention you need an oven:
- **Hidden**: The recipe assumes you have an oven
- **Explicit**: "Preheat oven to 350°F. Bake for 20 minutes."

### The Travel Directions Analogy
Hidden dependencies are like directions that say "turn left at the big tree":
- **Hidden**: Assumes the tree exists and you'll recognize it
- **Explicit**: "Turn left at the intersection of Main St. and Oak Ave."

### The IKEA Furniture Analogy
Hidden dependencies are like furniture instructions that don't list all the tools needed:
- **Hidden**: Step 5 suddenly requires a power drill
- **Explicit**: "Tools needed: screwdriver, hammer, power drill"

## How It Relates to Our Patterns

### Static Methods Often Hide Dependencies
```python
class DataProcessor:
    @staticmethod
    def process():
        # Hidden dependencies on class variables
        data = DataStorage.get_data()  # Where's DataStorage?
        config = Config.get_settings()  # Where's Config?
        return transform(data, config)
```

### Singletons Create Hidden Dependencies
```python
# Anywhere in your code:
logger = Logger.getInstance()  # Hidden singleton dependency
logger.log("Something happened")

# Better: Inject the logger
def process_data(data, logger):
    logger.log("Processing data")
    # ...
```

### Instance Methods Make Dependencies Explicit
```python
class DataProcessor:
    def __init__(self, storage, config):  # Explicit!
        self.storage = storage
        self.config = config
    
    def process(self):
        data = self.storage.get_data()
        return transform(data, self.config)
```

## Detecting Hidden Dependencies

> [!warning] Signs of Hidden Dependencies
> 1. **Import statements** at function level
> 2. **`os.environ`** access inside functions
> 3. **Singleton access** via `getInstance()`
> 4. **Global variable** access
> 5. **Static method calls** to other classes
> 6. **`datetime.now()`** or `time.time()` calls
> 7. **Direct file system** access
> 8. **Network calls** without injected clients

## Fixing Hidden Dependencies

### Before: Hidden Dependencies
```python
class ReportGenerator:
    def generate_report(self, data):
        # Hidden dependencies everywhere!
        template = open('template.html').read()
        
        processed_data = DataProcessor.process(data)
        
        report = template.format(
            date=datetime.now(),
            data=processed_data
        )
        
        EmailSender.send('admin@example.com', report)
        
        return report
```

### After: Explicit Dependencies
```python
class ReportGenerator:
    def __init__(self, template_loader, data_processor, clock, email_sender):
        self.template_loader = template_loader
        self.data_processor = data_processor
        self.clock = clock
        self.email_sender = email_sender
    
    def generate_report(self, data, recipient):
        # All dependencies are explicit!
        template = self.template_loader.load('template.html')
        
        processed_data = self.data_processor.process(data)
        
        report = template.format(
            date=self.clock.now(),
            data=processed_data
        )
        
        self.email_sender.send(recipient, report)
        
        return report
```

## Benefits of Explicit Dependencies

> [!info] Why Make Dependencies Explicit?
> 1. **Testability**: Easy to mock/stub dependencies
> 2. **Clarity**: Interface shows what's needed
> 3. **Flexibility**: Easy to swap implementations
> 4. **Debugging**: Clear where problems originate
> 5. **Refactoring**: Safe to move code around
> 6. **Documentation**: Self-documenting code

## Key Takeaways

> [!important] Remember About Hidden Dependencies
> 1. **If it's not in the parameters, it's hidden**
> 2. **Constructor injection** makes dependencies explicit
> 3. **Avoid global state** access inside functions
> 4. **Pass time as parameter** instead of using `now()`
> 5. **Inject clients** for external services
> 6. **Test difficulty** indicates hidden dependencies
> 7. **"new" is glue"** - avoid creating dependencies inside