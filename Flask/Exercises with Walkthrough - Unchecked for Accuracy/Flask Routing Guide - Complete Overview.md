Additional help for: https://appacademy.instructure.com/courses/319/pages/routing-in-flask-2?module_item_id=56599
## 1. Understanding Flask Routes

### What is Routing?
```python
# Routes map URLs to Python functions
@app.route('/hello')
def hello():
    return 'Hello World!'

# When user visits /hello, they see: Hello World!
```

### Basic Route Structure
```python
from flask import Flask
app = Flask(__name__)

# Three components:
@app.route('/path')      # 1. Decorator with URL pattern
def function_name():     # 2. Function definition
    return 'Content'     # 3. Return value (response)
```

## 2. Types of Routes

### Static Routes
```python
# Basic routes with fixed paths
@app.route('/')
def home():
    return '<h1>Home Page</h1>'

@app.route('/about')
def about():
    return '<h1>About Page</h1>'

# Multiple URLs for same function
@app.route('/')
@app.route('/home')
def home():
    return '<h1>Home Page</h1>'
```

### Dynamic Routes (URL Parameters)
```python
# Basic parameter
@app.route('/user/<username>')
def show_user(username):
    return f'User: {username}'

# Type-specific parameters
@app.route('/post/<int:post_id>')    # Must be integer
@app.route('/price/<float:amount>')   # Must be float
@app.route('/path/<path:subpath>')    # Can include slashes
```

## 3. Request Lifecycle Hooks

### Before Request
```python
@app.before_request
def before_request_function():
    # Runs before EVERY request
    print("Processing request...")
    # Common uses:
    # - Check if user is logged in
    # - Open database connection
    # - Load user preferences
```

### After Request
```python
@app.after_request
def after_request_function(response):
    # Runs after EVERY request
    print("Request completed")
    # Must return response
    return response
```

## 4. Static Files

### Directory Structure
```plaintext
your_app/
├── static/                # Static files directory
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   └── script.js
│   └── images/
│       └── logo.png
└── app.py
```

### Accessing Static Files
```python
# URL Pattern:
# /static/filename

# Examples:
# /static/css/style.css
# /static/images/logo.png
```

## 5. Practice Exercises

### 1. Basic Routes
```python
# Create these routes:
# - Home page ('/')
# - About page ('/about')
# - Contact page ('/contact')
```

### 2. Dynamic Routes
```python
# Create routes that:
# - Display user profile by username
# - Show product by ID (must be integer)
# - Show category items by category name
```

### 3. Request Hooks
```python
# Create hooks that:
# - Log request URL before processing
# - Log response status code after processing
```

## 6. Common Patterns & Best Practices

### URL Building
```python
from flask import url_for

@app.route('/')
def index():
    # Generate URL for 'user_profile' route
    url = url_for('user_profile', username='john')
    return f'User profile URL: {url}'
```

### Error Handling
```python
@app.errorhandler(404)
def page_not_found(e):
    return 'Page not found', 404

@app.errorhandler(500)
def server_error(e):
    return 'Server error', 500
```

## 7. Quick Reference

### Route Decorators
```python
@app.route('/')                    # Basic route
@app.route('/path/')              # Trailing slash
@app.route('/user/<name>')        # Parameter
@app.route('/id/<int:id>')        # Typed parameter
@app.before_request               # Before hook
@app.after_request               # After hook
```

### Parameter Types
```python
<string:param>    # (default) string
<int:param>       # integer
<float:param>     # floating point
<path:param>      # string including slashes
<uuid:param>      # UUID strings
```

## 8. Testing Your Routes

### Manual Testing
```plaintext
Test these URLs:
✓ http://localhost:5000/
✓ http://localhost:5000/about
✓ http://localhost:5000/user/john
✓ http://localhost:5000/post/123
✗ http://localhost:5000/post/abc (should fail)
```

### Automated Testing
```python
def test_home_page():
    response = app.test_client().get('/')
    assert response.status_code == 200
    assert b'Home Page' in response.data
```

## 9. Troubleshooting

Common Issues:
```plaintext
404 Error:
- Check route spelling
- Check parameter types
- Verify file exists (for static files)

500 Error:
- Check function return values
- Verify parameter handling
```

Remember:
- Use meaningful function names
- Keep routes organized
- Handle errors appropriately
- Use type checking for parameters
- Document your routes
- Test all paths
- Use request hooks wisely
- Structure static files properly

