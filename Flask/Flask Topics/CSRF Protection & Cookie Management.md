## What is CSRF?
Cross-Site Request Forgery (CSRF) is an attack that forces authenticated users to submit unwanted requests. Our protection strategy uses a dual-token pattern: one in a cookie and one in a header that must match.

## Core Concepts
```javascript
Frontend (Cookie) -> Backend (Validate) -> Frontend (Header) -> Backend (Compare)
```
This flow ensures requests come from your application, not malicious sites, as other sites can't read your cookies to get the token.

## Backend Setup (Flask)

### 1. Initial Configuration
```python
# __init__.py
from flask_wtf.csrf import CSRFProtect, generate_csrf

def create_app():
    app = Flask(__name__)
    csrf = CSRFProtect(app)  # Enable CSRF protection
```
CSRFProtect automatically checks for and validates CSRF tokens on all non-GET requests. It's your first line of defense.

### 2. CSRF Cookie Setup
```python
@app.after_request
def inject_csrf_token(response):
    response.set_cookie(
        'csrf_token',
        generate_csrf(),
        secure=False,          # True in production
        samesite=None,         # 'Strict' in production
        httponly=False         # Must be False for CSRF token
    )
    return response
```
**Cookie Flags Explained:**
- `secure`: Only send cookie over HTTPS
  - Development: `False` (allows HTTP)
  - Production: `True` (requires HTTPS)

- `httponly`: Prevents JavaScript access
  - CSRF Token: `False` (JS needs to read it)
  - Session Cookies: `True` (protect from XSS)

- `samesite`: Controls cross-site cookie behavior
  - Development: `None` (more permissive)
  - Production: `'Strict'` (more secure)

**When to Use HttpOnly:**
- `True` for:
  - Session cookies
  - Authentication tokens
  - Any sensitive data not needed by JS
- `False` for:
  - CSRF tokens (JS must read them)
  - Client-side preferences
  - Analytics cookies

### 3. CORS Configuration
```python
CORS(app, supports_credentials=True, resources={
    r"/api/*": {
        "origins": [
            "http://localhost:5173",
            "http://127.0.0.1:5173"
        ],
        "methods": ["GET", "POST", "PUT", "DELETE"],
        "allow_headers": ["Content-Type", "Authorization", "X-CSRF-Token"],
        "supports_credentials": True
    }
})
```
CORS (Cross-Origin Resource Sharing) controls which domains can access your API. `supports_credentials` is crucial for cookies to work cross-origin.

### 4. Form Validation
```python
class LoginForm(FlaskForm):  # Inherits CSRF protection
    email = StringField('email', validators=[
        DataRequired(), 
        Email()
    ])
    password = PasswordField('password', validators=[
        DataRequired()
    ])
```
FlaskForm automatically includes CSRF protection. Each form submission must include a valid CSRF token.

### 5. Route Protection
```python
@app.route('/api/users/login', methods=['POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():  # Checks CSRF token automatically
        # Process login
        return jsonify({"message": "Success"})
    return jsonify({"errors": form.errors}), 400
```
`validate_on_submit()` checks both form data and CSRF token. Invalid tokens trigger automatic rejection.

## Frontend Setup (React)

### 1. CSRF Fetch Utility
```javascript
import Cookies from 'js-cookie';

export async function csrfFetch(url, options = {}) {
    options.method = options.method || 'GET';
    options.headers = options.headers || {};

    // Include CSRF token in non-GET requests
    if (options.method.toUpperCase() !== 'GET') {
        options.headers['Content-Type'] = 'application/json';
        const csrfToken = Cookies.get('csrf_token');
        if (csrfToken) {
            options.headers['X-CSRF-Token'] = csrfToken;
        }
    }

    options.credentials = 'include';  // Important for cookies!

    const response = await fetch(url, options);
    if (response.status >= 400) throw response;
    return response;
}
```
This utility wraps fetch to automatically include CSRF tokens. `credentials: 'include'` ensures cookies are sent with cross-origin requests.

### 2. CSRF Restoration
```javascript
export async function restoreCSRF() {
    const response = await fetch('/api/users/csrf/restore', {
        credentials: 'include'
    });
    const data = await response.json();
    if (data.csrf_token) {
        Cookies.set('csrf_token', data.csrf_token);
    }
    return response;
}
```
Gets a fresh CSRF token when starting your app or after token expiration.

### 3. Using in Components/Thunks
```javascript
// Login thunk example
export const thunkLogin = (credentials) => async (dispatch) => {
    try {
        const response = await csrfFetch('/api/users/login', {
            method: 'POST',
            body: JSON.stringify(credentials)
        });
        const data = await response.json();
        dispatch(setUser(data));
        return data;
    } catch (err) {
        const errors = await err.json();
        return errors;
    }
};
```
Use csrfFetch instead of regular fetch for protected routes. It handles token inclusion automatically.

## Security Best Practices

### Cookie Security Matrix
```
Cookie Type  | HttpOnly | Secure | SameSite
-------------|----------|---------|----------
CSRF Token   | False    | True*   | Strict*
Session ID   | True     | True*   | Strict*
Auth Token   | True     | True*   | Strict*
Preferences  | False    | False   | Lax
Analytics    | False    | False   | Lax

* In production
```

### Environment-Specific Settings
```python
# Development
secure=False
samesite=None
httponly=False  # For CSRF token only

# Production
secure=True
samesite='Strict'
httponly=True  # Except CSRF token
```

## Common Issues & Solutions

### 1. Cookie Issues
- Check `httponly=False` for CSRF token
- Verify cookie in DevTools > Application > Cookies
- Ensure `credentials: 'include'` in fetch requests

**Debugging Steps:**
1. Check cookie presence in DevTools
2. Verify cookie flags
3. Confirm JavaScript can read token

### 2. CORS Issues
- Check origins in CORS configuration
- Verify all needed headers are allowed
- Ensure `supports_credentials: true`

**Common CORS Errors:**
```
Access-Control-Allow-Origin missing
Access-Control-Allow-Credentials missing
Preflight request failed
```

### 3. Token Issues
- Verify token in cookie matches header
- Check cookie name matches exactly
- Ensure proper error handling

## Testing in Postman
```
Headers needed:
Content-Type: application/json
X-CSRF-Token: [token from cookie]

Make sure:
- Cookies are enabled
- Send cookies is checked
- Token matches cookie value
```

## DevTools Debugging
```
1. Network Tab:
   - Check request headers
   - Verify cookie presence
   - Look for CSRF token

2. Application > Cookies:
   - Verify csrf_token exists
   - Check HttpOnly flag (should be false)
   - Verify token value
```

## Docker Considerations
```bash
# Rebuild after cookie changes
docker compose down
docker compose up --build

# Check logs
docker logs container_name

# Access shell
docker exec -it container_name /bin/bash
```

Key Points to Remember:
1. CSRF protection requires both cookie and header tokens
2. HttpOnly should be false ONLY for CSRF token
3. Always use HTTPS in production
4. Include credentials for cross-origin requests
5. Regularly rotate CSRF tokens
6. Validate tokens on all state-changing requests
7. Keep security headers up to date

This dual-token approach provides strong protection against CSRF attacks while maintaining usability.