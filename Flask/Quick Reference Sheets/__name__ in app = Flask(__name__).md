# Python `__name__` in Flask

## Basic Usage
```python
from flask import Flask
app = Flask(__name__)
```

## What `__name__` Returns

| Scenario | Value of `__name__` | When This Happens |
|----------|-------------------|-------------------|
| Direct Run | `'__main__'` | `python app.py` |
| Imported | `'app'` | `from app import app` |

## Common Pattern
```python
# app.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello'

if __name__ == '__main__':    # Only runs if direct
    app.run(debug=True)       # Won't run if imported
```

## Why Use `__name__`?
- ✅ Helps Flask locate your application root
- ✅ Enables automatic resource finding (templates/static)
- ✅ Standard Python pattern for module/script detection
- ✅ Prevents code from running when imported

## Testing It
```python
# Print __name__ to see what it is
print(f"__name__ is: {__name__}")

# Running directly:     → '__main__'
# Importing elsewhere:  → 'app' (or your filename)
```

## Common Gotchas
- 🚫 Don't hardcode the app name (`Flask('app')`)
- 🚫 Don't skip the `if __name__ == '__main__'` check
- ✅ Always use `__name__` for Flask initialization
- ✅ Keep main run block at bottom of file

## Quick Reference
```python
# ✅ Do this:
app = Flask(__name__)

# 🚫 Not this:
app = Flask('my_app')
```

## Key Points
- Acts as application identifier
- Changes based on how code is run
- Essential for Flask's resource location
- Standard Python best practice