# Refactoring: Singleton to Static

#refactoring #patterns #singleton #static

## Why Refactor Singleton to Static?

Converting singleton to static pattern makes sense when:
- The singleton has no real state (just methods)
- You want to eliminate the object creation overhead
- The functionality is purely utility-based
- You don't need initialization parameters
- You want simpler, more direct access
- Thread safety of singleton adds unnecessary complexity

> [!info] Less Common Refactoring
> This refactoring is less common because it removes flexibility. Ensure you truly don't need instance features before proceeding.

## Simple Example: Math Constants Service

### Before: Singleton Pattern (Overengineered)
```python
import math
import threading

class MathConstants:
    """Singleton for math constants (unnecessary complexity)"""
    
    _instance = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized'):
            return
            
        # "Initialize" constants (they're already constant!)
        self._pi = math.pi
        self._e = math.e
        self._golden_ratio = (1 + math.sqrt(5)) / 2
        self._initialized = True
    
    @property
    def pi(self):
        return self._pi
    
    @property
    def e(self):
        return self._e
    
    @property
    def golden_ratio(self):
        return self._golden_ratio
    
    def circle_area(self, radius):
        return self.pi * radius ** 2
    
    def circle_circumference(self, radius):
        return 2 * self.pi * radius

# Usage - unnecessary object creation
constants = MathConstants()
area = constants.circle_area(5)
print(f"PI = {constants.pi}")
```

**Problems with singleton here:**
- 🚫 Unnecessary object creation for constants
- 🚫 Thread safety overhead for immutable data
- 🚫 More complex than needed
- 🚫 No real state to manage

### After: Static Pattern
```python
import math

class MathConstants:
    """Static math constants and utilities"""
    
    # Class-level constants
    PI = math.pi
    E = math.e
    GOLDEN_RATIO = (1 + math.sqrt(5)) / 2
    
    # Common mathematical constants
    SQRT_2 = math.sqrt(2)
    SQRT_3 = math.sqrt(3)
    LN_2 = math.log(2)
    LN_10 = math.log(10)
    
    @staticmethod
    def circle_area(radius):
        """Calculate circle area"""
        return MathConstants.PI * radius ** 2
    
    @staticmethod
    def circle_circumference(radius):
        """Calculate circle circumference"""
        return 2 * MathConstants.PI * radius
    
    @staticmethod
    def sphere_volume(radius):
        """Calculate sphere volume"""
        return (4/3) * MathConstants.PI * radius ** 3
    
    @staticmethod
    def golden_rectangle_dimensions(width):
        """Get golden rectangle dimensions"""
        return width, width * MathConstants.GOLDEN_RATIO

# Usage - direct access, no object needed
area = MathConstants.circle_area(5)
print(f"PI = {MathConstants.PI}")
width, height = MathConstants.golden_rectangle_dimensions(100)
```

**Benefits of static approach here:**
- ✅ No unnecessary objects
- ✅ Direct access to constants
- ✅ Clearer intent (utilities and constants)
- ✅ No initialization overhead

## Complex Example: Encoding Utilities

### Before: Singleton Encoder
```python
import base64
import hashlib
import threading
from typing import Optional

class EncodingService:
    """Singleton encoding service"""
    
    _instance: Optional['EncodingService'] = None
    _lock = threading.Lock()
    
    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance
    
    def __init__(self):
        if hasattr(self, '_initialized'):
            return
            
        # Initialize encoding mappings (static data)
        self._hex_chars = '0123456789abcdef'
        self._b64_chars = (
            'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
        )
        self._initialized = True
    
    def to_base64(self, data: bytes) -> str:
        """Encode to base64"""
        return base64.b64encode(data).decode('ascii')
    
    def from_base64(self, encoded: str) -> bytes:
        """Decode from base64"""
        return base64.b64decode(encoded)
    
    def to_hex(self, data: bytes) -> str:
        """Encode to hexadecimal"""
        return data.hex()
    
    def from_hex(self, encoded: str) -> bytes:
        """Decode from hexadecimal"""
        return bytes.fromhex(encoded)
    
    def hash_md5(self, data: bytes) -> str:
        """Get MD5 hash"""
        return hashlib.md5(data).hexdigest()
    
    def hash_sha256(self, data: bytes) -> str:
        """Get SHA256 hash"""
        return hashlib.sha256(data).hexdigest()
    
    def hash_sha512(self, data: bytes) -> str:
        """Get SHA512 hash"""
        return hashlib.sha512(data).hexdigest()

# Usage
encoder = EncodingService()
encoded = encoder.to_base64(b"Hello World")
decoded = encoder.from_base64(encoded)
hash_val = encoder.hash_sha256(b"password")
```

### After: Static Encoder
```python
import base64
import hashlib
from typing import Dict, Callable
import binascii

class EncodingUtils:
    """Static encoding utilities"""
    
    # Encoding/decoding maps (if needed)
    HEX_CHARS = '0123456789abcdef'
    B64_CHARS = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    
    # Hash algorithm registry
    HASH_ALGORITHMS: Dict[str, Callable] = {
        'md5': hashlib.md5,
        'sha1': hashlib.sha1,
        'sha256': hashlib.sha256,
        'sha512': hashlib.sha512,
        'sha3_256': hashlib.sha3_256,
        'sha3_512': hashlib.sha3_512,
    }
    
    # Base64 variants
    @staticmethod
    def to_base64(data: bytes, urlsafe: bool = False) -> str:
        """Encode to base64"""
        if urlsafe:
            return base64.urlsafe_b64encode(data).decode('ascii')
        return base64.b64encode(data).decode('ascii')
    
    @staticmethod
    def from_base64(encoded: str, urlsafe: bool = False) -> bytes:
        """Decode from base64"""
        if urlsafe:
            return base64.urlsafe_b64decode(encoded)
        return base64.b64decode(encoded)
    
    # Hex encoding
    @staticmethod
    def to_hex(data: bytes, separator: str = '') -> str:
        """Encode to hexadecimal with optional separator"""
        hex_str = data.hex()
        if separator:
            # Add separator every 2 characters
            return separator.join(hex_str[i:i+2] for i in range(0, len(hex_str), 2))
        return hex_str
    
    @staticmethod
    def from_hex(encoded: str) -> bytes:
        """Decode from hexadecimal"""
        # Remove common separators
        cleaned = encoded.replace(' ', '').replace(':', '').replace('-', '')
        return bytes.fromhex(cleaned)
    
    # Generic hash function
    @staticmethod
    def hash(data: bytes, algorithm: str = 'sha256') -> str:
        """Hash data with specified algorithm"""
        if algorithm not in EncodingUtils.HASH_ALGORITHMS:
            raise ValueError(f"Unknown algorithm: {algorithm}")
        
        hasher = EncodingUtils.HASH_ALGORITHMS[algorithm]()
        hasher.update(data)
        return hasher.hexdigest()
    
    # Convenience methods for common hashes
    @staticmethod
    def md5(data: bytes) -> str:
        """Get MD5 hash (deprecated, use only for compatibility)"""
        return EncodingUtils.hash(data, 'md5')
    
    @staticmethod
    def sha256(data: bytes) -> str:
        """Get SHA256 hash"""
        return EncodingUtils.hash(data, 'sha256')
    
    @staticmethod
    def sha512(data: bytes) -> str:
        """Get SHA512 hash"""
        return EncodingUtils.hash(data, 'sha512')
    
    # Additional utilities
    @staticmethod
    def constant_time_compare(a: bytes, b: bytes) -> bool:
        """Compare bytes in constant time (for security)"""
        return hmac.compare_digest(a, b)
    
    @staticmethod
    def encode_multibase(data: bytes, encoding: str = 'base64') -> str:
        """Encode with multiple encoding options"""
        encoders = {
            'base64': EncodingUtils.to_base64,
            'hex': EncodingUtils.to_hex,
            'base32': lambda d: base64.b32encode(d).decode('ascii'),
            'base85': lambda d: base64.b85encode(d).decode('ascii'),
        }
        
        if encoding not in encoders:
            raise ValueError(f"Unknown encoding: {encoding}")
        
        return encoders[encoding](data)

# Usage - cleaner and more direct
encoded = EncodingUtils.to_base64(b"Hello World")
decoded = EncodingUtils.from_base64(encoded)

# More flexible
url_safe = EncodingUtils.to_base64(b"user/data", urlsafe=True)
hex_spaced = EncodingUtils.to_hex(b"Hello", separator=' ')  # "48 65 6c 6c 6f"

# Generic hash function
sha3_hash = EncodingUtils.hash(b"password", 'sha3_256')

# Multiple encodings
for encoding in ['base64', 'hex', 'base32']:
    result = EncodingUtils.encode_multibase(b"test", encoding)
    print(f"{encoding}: {result}")
```

## What Changes During Refactoring

### 1. **Remove Singleton Mechanism**
```python
# Remove these:
# - _instance variable
# - _lock for thread safety
# - __new__ override
# - Initialization checks in __init__
```

### 2. **Convert Methods to Static**
```python
# Before
def method(self, param):
    return process(param)

# After
@staticmethod
def method(param):
    return process(param)
```

### 3. **Move Instance State to Class Level**
```python
# Before
def __init__(self):
    self.cache = {}
    self.config = load_config()

# After  
class Service:
    # Class-level state (if needed)
    _cache = {}
    _config = load_config()  # Loaded at import time
```

### 4. **Remove Property Access**
```python
# Before
@property
def value(self):
    return self._value

# After - direct class attribute
VALUE = computed_value
```

## Migration Strategy

### Step 1: Identify Stateless Singletons
Look for singletons that:
- Have no mutable state
- Don't need initialization parameters
- Contain only utility methods
- Don't manage resources

### Step 2: Create Static Version Alongside
```python
class Service:
    _instance = None
    
    # Keep singleton methods
    def process(self, data):
        return transform(data)
    
    # Add static versions
    @staticmethod
    def process_static(data):
        return transform(data)
```

### Step 3: Migrate Callers
```python
# Phase 1: Update imports
# from service import Service
# service = Service()
# result = service.process(data)

# Phase 2: Use static methods
# from service import Service  
# result = Service.process_static(data)

# Phase 3: Remove singleton mechanism
```

## When This Makes Sense

### Good Candidates

#### 1. **Pure Utility Functions**
```python
# Before - singleton
class StringUtils:
    _instance = None
    
    def reverse(self, text):
        return text[::-1]

# After - static
class StringUtils:
    @staticmethod
    def reverse(text):
        return text[::-1]
```

#### 2. **Constant Providers**
```python
# Before - singleton
class Constants:
    _instance = None
    
    def __init__(self):
        self.PI = 3.14159
        self.E = 2.71828

# After - static
class Constants:
    PI = 3.14159
    E = 2.71828
```

#### 3. **Stateless Validators**
```python
# Before - singleton  
class Validator:
    _instance = None
    
    def is_email(self, text):
        return '@' in text

# After - static
class Validator:
    @staticmethod
    def is_email(text):
        return '@' in text
```

## Common Pitfalls

### 1. **Losing Resource Management**
```python
# BAD - singleton managed resources
class FileManager:
    def __init__(self):
        self.open_files = []
    
    def open_file(self, path):
        f = open(path)
        self.open_files.append(f)
        return f
    
    def cleanup(self):
        for f in self.open_files:
            f.close()

# Can't convert to static - needs instance state!
```

### 2. **Breaking Configurability**
```python
# Singleton allows configuration
class Logger:
    def __init__(self, level='INFO'):
        self.level = level

# Static can't be configured at runtime
class Logger:
    LEVEL = 'INFO'  # Fixed at import time
```

### 3. **Losing Testability**
```python
# Singleton can be mocked/reset
def test_with_singleton():
    Service.reset()
    service = Service(test_mode=True)
    
# Static is harder to mock
def test_with_static():
    # Must patch at module level
    with patch.object(Service, 'method'):
        Service.method()
```

## Performance Comparison

```python
import timeit

# Singleton version
class SingletonCalc:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
    
    def add(self, a, b):
        return a + b

# Static version
class StaticCalc:
    @staticmethod
    def add(a, b):
        return a + b

# Benchmark singleton
singleton_time = timeit.timeit("""
calc = SingletonCalc()
calc.add(5, 3)
""", globals=globals(), number=1000000)

# Benchmark static
static_time = timeit.timeit("""
StaticCalc.add(5, 3)
""", globals=globals(), number=1000000)

print(f"Singleton: {singleton_time:.4f}s")
print(f"Static: {static_time:.4f}s")
print(f"Static is {singleton_time/static_time:.2f}x faster")
```

## Key Takeaways

> [!important] When Refactoring Singleton to Static
> 1. **Ensure truly stateless** - no mutable data needed
> 2. **Remove all singleton machinery** - `_instance`, `__new__`, locks
> 3. **Add @staticmethod** to all methods
> 4. **Convert instance attributes** to class attributes
> 5. **Consider import-time costs** - static initialization happens at import
> 6. **Document the change** - explain why singleton wasn't needed
> 7. **Verify thread safety** - static class variables are shared