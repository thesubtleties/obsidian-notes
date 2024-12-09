## Basic Import Syntax
| Import Style | Syntax | Example | Usage |
|-------------|---------|---------|--------|
| Basic import | `import module` | `import math` | `math.sqrt(16)` |
| From import | `from module import item` | `from math import sqrt` | `sqrt(16)` |
| Import as | `import module as alias` | `import numpy as np` | `np.array([1,2,3])` |
| Import multiple | `from module import item1, item2` | `from math import sqrt, pi` | `sqrt(pi)` |
| Import all (not recommended) | `from module import *` | `from math import *` | `sqrt(16)` |

## Common Import Patterns

Standard imports:
```python
import os
import sys
import json
```

Aliased imports:
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
```

Specific imports:
```python
from datetime import datetime, timedelta
from collections import defaultdict, Counter
from typing import List, Dict, Optional
```

Relative imports:
```python
# Same directory
from .module import function
# Parent directory
from ..module import function
# Subpackage
from .subpackage.module import function
```

## Directory Structure
```
project/
│
├── main.py
├── package/
│   ├── __init__.py
│   ├── module1.py
│   └── module2.py
│
└── tests/
    ├── __init__.py
    └── test_module1.py
```

## Import Examples by Category

Let me isolate those sections with different formatting to ensure they display correctly:

## System/OS Imports
```python
# System and OS operations
import os           # Operating system interface
import sys         # System-specific parameters
import platform    # System platform info
import subprocess  # Subprocess management
import shutil      # File operations
```

## Data Processing Imports
```python
# Data format handling
import json        # JSON encoder/decoder
import csv         # CSV file operations
import pickle     # Python object serialization
import xml.etree.ElementTree as ET  # XML processing
import yaml       # YAML parser
```

Date/Time:
```python
from datetime import datetime, date, timedelta
import time
import calendar
```

Data Science:
```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import scipy
import sklearn
```

Web/Network:
```python
import requests
import urllib
import socket
import http
import ssl
```

## Import Best Practices

Order of Imports:
```python
# Standard library imports
import os
import sys

# Third-party imports
import numpy as np
import pandas as pd

# Local application imports
from .module import function
from .utils import helper
```

Using __init__.py:
```python
# In __init__.py
from .module1 import ClassA
from .module2 import function_b

__all__ = ['ClassA', 'function_b']
```

## Common Patterns and Tips

Conditional Imports:
```python
try:
    import numpy as np
except ImportError:
    print("NumPy is not installed")
```

Lazy Imports:
```python
def function_needing_numpy():
    import numpy as np
    return np.array([1, 2, 3])
```

Type Hints Imports:
```python
from typing import (
    List,
    Dict,
    Tuple,
    Optional,
    Union,
    Any
)
```

Quick Tips:
- 📦 Keep imports at the top of the file
- 🔄 Use absolute imports over relative when possible
- 🎯 Import only what you need
- 🚫 Avoid `from module import *`
- 📝 Group imports logically
- ⚡ Use aliases for common libraries

Common Issues:
```python
# Circular imports - DON'T
# file1.py
from file2 import function2
# file2.py
from file1 import function1

# Solution - Move shared code to separate module
# shared.py
def shared_function(): 
    pass
```

Remember:
- Imports are case-sensitive
- Python searches for modules in `sys.path`
- Use `requirements.txt` for dependencies
- Keep `__init__.py` files clean and minimal
- Use virtual environments for project dependencies