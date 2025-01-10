## Backend Setup (Flask)

```python
# api/extensions.py
from flask_wtf.csrf import CSRFProtect

csrf = CSRFProtect()

# api/app.py
from api.extensions import csrf

def create_app():
    app = Flask(__name__)
    app.config['SECRET_KEY'] = 'your-secret-key'
    csrf.init_app(app)
```
Sets up CSRF protection in Flask. The SECRET_KEY is used to generate CSRF tokens.

## Axios Setup

```javascript
// api/axios.js
import axios from 'axios';

const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true,  // Important! Allows cookies to be sent
  headers: {
    'Content-Type': 'application/json',
  }
});

// Get CSRF token from cookie
function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(';').shift();
}

// Add CSRF token to every request
api.interceptors.request.use(config => {
  const token = getCookie('csrf_token');
  if (token) {
    config.headers['X-CSRF-TOKEN'] = token;
  }
  return config;
});

export default api;
```
Sets up axios to handle CSRF tokens automatically. The withCredentials option is crucial for cookie handling.

## Usage in Thunks

```javascript
// store/eventSlice.js
import { createAsyncThunk } from '@reduxjs/toolkit';
import api from '../api/axios';

export const createEvent = createAsyncThunk(
  'events/createEvent',
  async (eventData) => {
    // CSRF token is automatically added by interceptor
    const response = await api.post('/events', eventData);
    return response.data;
  }
);
```
The CSRF token is automatically added to requests through the axios interceptor.

## Error Handling

```javascript
// api/axios.js
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 403) {
      // CSRF token validation failed
      console.error('CSRF validation failed');
      // Maybe refresh the token or redirect to login
    }
    return Promise.reject(error);
  }
);
```
Handles CSRF validation errors specifically.

## Complete Setup Example

```javascript
// api/csrf.js
import axios from 'axios';

export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL,
  withCredentials: true,
  headers: {
    'Content-Type': 'application/json',
  }
});

// CSRF token management
const getCSRFToken = () => getCookie('csrf_token');

api.interceptors.request.use(
  config => {
    // Add CSRF token
    const token = getCSRFToken();
    if (token) {
      config.headers['X-CSRF-TOKEN'] = token;
    }
    
    // Add auth token if exists
    const authToken = localStorage.getItem('token');
    if (authToken) {
      config.headers['Authorization'] = `Bearer ${authToken}`;
    }
    
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

// Handle response errors
api.interceptors.response.use(
  response => response,
  error => {
    if (error.response?.status === 403) {
      // CSRF error
      console.error('CSRF validation failed');
    }
    return Promise.reject(error);
  }
);

export default api;
```

## Usage in Components

```javascript
// components/EventForm.js
function EventForm() {
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      // CSRF token is handled automatically
      const response = await api.post('/events', formData);
      console.log('Success:', response.data);
    } catch (error) {
      if (error.response?.status === 403) {
        console.error('CSRF validation failed');
      }
    }
  };

  return <form onSubmit={handleSubmit}>...</form>;
}
```

## Flask Route Example

```python
# api/routes.py
from flask import jsonify
from flask_wtf.csrf import generate_csrf

@app.route('/api/csrf-token', methods=['GET'])
def get_csrf_token():
    token = generate_csrf()
    response = jsonify({'csrf_token': token})
    response.set_cookie('csrf_token', token)
    return response
```
Endpoint to get a new CSRF token if needed.

## Initial Setup in App

```javascript
// App.jsx or similar
import { useEffect } from 'react';
import api from './api/axios';

function App() {
  useEffect(() => {
    // Get initial CSRF token
    const fetchCSRFToken = async () => {
      try {
        await api.get('/api/csrf-token');
      } catch (error) {
        console.error('Failed to fetch CSRF token:', error);
      }
    };
    
    fetchCSRFToken();
  }, []);

  return <div>...</div>;
}
```

Remember:
- Always use withCredentials: true
- CSRF token is sent in cookies
- Add token to headers via interceptor
- Handle 403 errors appropriately
- Fetch new token when needed

Common Issues:
1. Missing withCredentials
2. Incorrect cookie name
3. Missing SECRET_KEY in Flask
4. Cross-origin issues

