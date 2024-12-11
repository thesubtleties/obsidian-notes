Additional help for: https://appacademy.instructure.com/courses/319/pages/flask-sessions-2?module_item_id=56626

## 1. Understanding Sessions

### What are Sessions?
```python
# Sessions are like temporary storage for each user:
# - Persists across requests
# - User-specific
# - Expires after browser closes (by default)
# - Stored securely using encryption
```
Think of sessions like a personal locker for each user visiting your site. The locker can store data that persists between page loads.

## 2. Basic Setup

### Configuration
```python
# .flaskenv or .env
SECRET_KEY=your-super-secret-key

# app/__init__.py
from flask import Flask, session

app = Flask(__name__)
app.config['SECRET_KEY'] = os.environ.get('SECRET_KEY')
```
The secret key is crucial - it's used to encrypt session data. Never share or commit your actual secret key.

## 3. Working with Sessions

### Setting Session Data
```python
from flask import session

@app.route('/login')
def login():
    session['user_id'] = 123
    session['username'] = 'john_doe'
    return 'Logged in!'
```
Session data is stored like a dictionary but persists across requests for each user.

### Reading Session Data
```python
@app.route('/profile')
def profile():
    if 'username' in session:
        return f'Hello, {session.get("username")}!'
    return 'Please log in'
```
Always check if data exists in session before trying to use it.

### Updating Session Data
```python
@app.route('/visits')
def count_visits():
    session['visits'] = session.get('visits', 0) + 1
    return f'You have visited {session["visits"]} times'
```
The get method with a default value helps avoid KeyError exceptions.

### Removing Session Data
```python
# Remove specific item
session.pop('user_id', None)  # None is default if key doesn't exist

# Clear entire session
session.clear()
```
Always provide a default value to pop to avoid KeyError.

## 4. Common Use Cases

### User Authentication
```python
@app.route('/login', methods=['POST'])
def login():
    if verify_user(request.form):  # Your verification logic
        session['user_id'] = get_user_id()
        session['logged_in'] = True
        return redirect('/dashboard')
    return 'Login failed'

@app.route('/logout')
def logout():
    session.clear()
    return redirect('/login')
```
Sessions are perfect for maintaining login state.

### Shopping Cart
```python
@app.route('/add-to-cart/<int:product_id>')
def add_to_cart(product_id):
    cart = session.get('cart', {})
    cart[product_id] = cart.get(product_id, 0) + 1
    session['cart'] = cart
    return f'Added to cart'
```
Sessions can store temporary user data like shopping carts.

## 5. Security Considerations

### Session Configuration
```python
app.config.update(
    SESSION_COOKIE_SECURE=True,    # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,  # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax', # CSRF protection
    PERMANENT_SESSION_LIFETIME=timedelta(minutes=30)  # Session timeout
)
```
These settings help protect your sessions from common vulnerabilities.

### Session Expiry
```python
from datetime import timedelta

@app.route('/login')
def login():
    session.permanent = True  # Enable session expiry
    app.permanent_session_lifetime = timedelta(days=7)
    # ... login logic
```
Setting session expiry helps manage server resources and security.

## 6. Complete Example

### User Session Management
```python
from flask import Flask, session, redirect, url_for, request
from functools import wraps

app = Flask(__name__)
app.secret_key = 'your-secret-key'

# Login required decorator
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        session['user_id'] = request.form['username']
        return redirect(url_for('dashboard'))
    return 'Login page'

@app.route('/dashboard')
@login_required
def dashboard():
    return f'Welcome {session["user_id"]}'

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('login'))
```

## 7. Best Practices

### Session Data Management
```python
# Do: Use get() with default value
user_id = session.get('user_id', None)

# Don't: Direct dictionary access
user_id = session['user_id']  # Could raise KeyError

# Do: Check for existence
if 'user_id' in session:
    # do something
```

### Session Size
```python
# Keep session data minimal
session['user_id'] = 123          # Good
session['user'] = user_object     # Bad - too much data
```
Sessions should only store essential data.

## 8. Quick Reference

### Common Operations
```python
# Set
session['key'] = value

# Get
value = session.get('key', default)

# Delete
session.pop('key', None)
session.clear()  # Clear all

# Check
'key' in session

# Modify
session['counter'] = session.get('counter', 0) + 1
```

Remember:
- Always use SECRET_KEY
- Keep session data minimal
- Clear sessions on logout
- Use secure cookie settings
- Handle session expiry
- Check for session existence
- Use get() with default values
- Consider session timeout

Would you like me to expand on any section or add additional content?