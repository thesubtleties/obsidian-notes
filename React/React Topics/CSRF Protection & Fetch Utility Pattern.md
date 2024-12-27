## Overview
The `csrfFetch` utility is a wrapper around the native `fetch` that automatically handles CSRF token inclusion for non-GET requests. This is necessary when your frontend React application needs to make state-changing requests (POST, PUT, DELETE) to your Flask backend, which requires CSRF validation for security.

## Location
```
react-app/
└── src/
    └── utils/
        └── fetch.js    # Put csrfFetch here in React frontend
```

## The Pattern
```javascript
// utils/fetch.js
export async function csrfFetch(url, options = {}) {
  // Set defaults
  options.method = options.method || 'GET';
  options.headers = options.headers || {};

  // Only add CSRF token for non-GET requests
  if (options.method.toUpperCase() !== 'GET') {
    // Set content type if not already set
    options.headers['Content-Type'] = 
      options.headers['Content-Type'] || 'application/json';
    // Get CSRF token from cookie set by Flask
    options.headers['X-CSRF-Token'] = getCookie('csrf_token');
  }

  const res = await fetch(url, options);
  if (res.ok) return res;
  throw res;
}

// Internal helper - don't export
function getCookie(name) {
  const cookies = document.cookie.split(';');
  for (let cookie of cookies) {
    const [cookieName, cookieValue] = cookie.trim().split('=');
    if (cookieName === name) return cookieValue;
  }
  return null;
}
```

## When to Use

### ✅ Use csrfFetch for:
```javascript
// Any state-changing operations:
// POST requests
const response = await csrfFetch('/api/projects', {
  method: 'POST',
  body: JSON.stringify(projectData)
});

// PUT requests
const response = await csrfFetch(`/api/projects/${id}`, {
  method: 'PUT',
  body: JSON.stringify(updates)
});

// DELETE requests
const response = await csrfFetch(`/api/projects/${id}`, {
  method: 'DELETE'
});
```

### ❌ Regular fetch is fine for:
```javascript
// GET requests don't need CSRF
const response = await fetch('/api/projects');
const response = await fetch(`/api/projects/${id}`);
```

## How It Works

1. Flask Backend Setup:
```python
# Flask automatically injects CSRF token after every request
@app.after_request
def inject_csrf_token(response):
    response.set_cookie("csrf_token", generate_csrf())
    return response
```

2. Frontend Flow:
```plaintext
1. Flask sets csrf_token cookie
2. csrfFetch reads cookie when making request
3. Sends token in X-CSRF-Token header
4. Flask validates token matches
5. Request proceeds if valid
```

## Usage in Redux Thunks
```javascript
// redux/projects.js
import { csrfFetch } from '../../utils/fetch';

// Use regular fetch for GET
export const thunkLoadProjects = () => async dispatch => {
  const response = await fetch('/api/projects');
  // ... handle response
};

// Use csrfFetch for POST/PUT/DELETE
export const thunkCreateProject = (project) => async dispatch => {
  const response = await csrfFetch('/api/projects', {
    method: 'POST',
    body: JSON.stringify(project)
  });
  // ... handle response
};
```

## Why This Pattern?

### Security:
- Prevents CSRF attacks
- Validates request origin
- Protects state-changing operations

### Convenience:
- Centralizes CSRF handling
- Consistent across application
- DRY principle

### Maintenance:
- One place to modify fetch behavior
- Easy to add features (like error handling)
- Consistent error handling

## Common Gotchas

1. Missing CSRF Token:
```javascript
// Won't work - missing CSRF token
fetch('/api/projects', {
  method: 'POST',
  body: JSON.stringify(data)
}); 

// Correct - uses csrfFetch
csrfFetch('/api/projects', {
  method: 'POST',
  body: JSON.stringify(data)
});
```

2. Content Type:
```javascript
// Don't need to set Content-Type manually
csrfFetch('/api/projects', {
  method: 'POST',
  body: JSON.stringify(data)
  // Content-Type set automatically
});
```

3. GET Requests:
```javascript
// Don't need csrfFetch for GET
const response = await fetch('/api/projects');
```

## Testing Considerations
```javascript
// Can check if CSRF token is being sent
const response = await csrfFetch('/api/test', {
  method: 'POST'
});

// Check network tab in dev tools:
// - Should see X-CSRF-Token header
// - Should see csrf_token cookie
```

Remember:
- Only needed for React frontend
- Only for non-GET requests
- Works with Flask's CSRF system
- Part of frontend security strategy