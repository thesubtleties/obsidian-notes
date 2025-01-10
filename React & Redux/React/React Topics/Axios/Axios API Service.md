## File Structure
```
src/
└── api/
    ├── index.js
    ├── userApi.js
    └── postApi.js
```

## Basic Setup (src/api/index.js)

```javascript
import axios from 'axios';

const API = axios.create({
  baseURL: process.env.REACT_APP_API_URL || 'http://localhost:5000/api', // '/api' if we have a vite config proxy
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json'
  }
});

export default API;
```

## Authentication Interceptor (src/api/index.js)

```javascript
API.interceptors.request.use((config) => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
}, (error) => Promise.reject(error));
```

## Error Handling Interceptor (src/api/index.js)

```javascript
API.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response.status === 401) {
      // Handle unauthorized access
    }
    return Promise.reject(error);
  }
);
```

## User API Service (src/api/userApi.js)

```javascript
import API from './index';

export const fetchUser = (userId) => API.get(`/users/${userId}`);
export const updateUser = (userId, userData) => API.put(`/users/${userId}`, userData);
export const createUser = (userData) => API.post('/users', userData);
export const deleteUser = (userId) => API.delete(`/users/${userId}`);
```

## Post API Service (src/api/postApi.js)

```javascript
import API from './index';

export const fetchPosts = (userId) => API.get(`/users/${userId}/posts`);
export const createPost = (userId, postData) => API.post(`/users/${userId}/posts`, postData);
export const updatePost = (postId, postData) => API.put(`/posts/${postId}`, postData);
export const deletePost = (postId) => API.delete(`/posts/${postId}`);
```

## Using API in Thunks (src/actions/userActions.js)

```javascript
import * as userApi from '../api/userApi';

export const fetchUser = (userId) => async (dispatch) => {
  dispatch({ type: 'FETCH_USER_REQUEST' });
  try {
    const { data } = await userApi.fetchUser(userId);
    dispatch({ type: 'FETCH_USER_SUCCESS', payload: data });
  } catch (error) {
    dispatch({ type: 'FETCH_USER_FAILURE', payload: error.message });
  }
};
```

## Error Handling in API Calls

```javascript
export const fetchUser = async (userId) => {
  try {
    const response = await API.get(`/users/${userId}`);
    return response.data;
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
};
```

## Cancelling Requests

```javascript
const source = axios.CancelToken.source();

export const fetchUserCancellable = (userId) => {
  return API.get(`/users/${userId}`, {
    cancelToken: source.token
  });
};

// To cancel:
source.cancel('Operation cancelled by the user.');
```

## Handling File Uploads

```javascript
export const uploadUserAvatar = (userId, file) => {
  const formData = new FormData();
  formData.append('avatar', file);
  
  return API.post(`/users/${userId}/avatar`, formData, {
    headers: {
      'Content-Type': 'multipart/form-data'
    }
  });
};
```

## Key Points:

1. Centralize API configuration in one file.
2. Use environment variables for API URL.
3. Implement interceptors for common tasks like authentication and error handling.
4. Organize API calls by feature or entity.
5. Handle errors consistently across all API calls.
6. Use cancellation tokens for long-running or potentially outdated requests.
7. Adapt the API service to handle different types of requests (GET, POST, PUT, DELETE, file uploads, etc.).
8. Keep the API service independent of state management (Redux) for better separation of concerns.

Remember: This API service acts as a bridge between your frontend React/Redux application and your backend Express.js API. It encapsulates all the logic for making HTTP requests, allowing your application logic to remain clean and focused.