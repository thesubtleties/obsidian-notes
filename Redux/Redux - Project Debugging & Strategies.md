# Redux Store Setup & Debugging Guide - Comprehensive Walkthrough

## 1. CSRF & API Routing Setup

**Concept**: Centralized fetch utility to handle CSRF tokens and API endpoints consistently across the application. This prevents repetitive code and ensures consistent error handling.

```javascript
// Centralized CSRF fetch utility
export async function csrfFetch(URL, options = {}) {
  // Handle API prefix automatically - prevents URL construction errors
  const apiUrl = URL.startsWith('/api') ? URL : `/api${URL}`;
  
  // Set default options with proper headers
  options.method = options.method || 'GET';
  options.headers = options.headers || {};

  // Only add CSRF token for non-GET requests (POST, PUT, DELETE)
  if (options.method.toUpperCase() !== 'GET') {
    options.headers['Content-Type'] = 'application/json';
    options.headers['X-CSRF-Token'] = Cookies.get('csrf_token');
  }

  // Standardized response handling
  const res = await fetch(apiUrl, options);
  if (res.ok) return res;
  throw res;  // Throw failed responses to be caught in thunks
}
```

**What We Learned**:
- Always centralize API calls to maintain consistency
- CSRF protection is crucial for non-GET requests
- Error handling should be standardized
- URL construction should be automated to prevent mistakes

**Common Pitfalls Avoided**:
- Inconsistent API endpoint formatting
- Missing CSRF tokens
- Duplicate error handling logic
- Manual URL construction errors

## 2. Error Handling Pattern

**Concept**: Standardized error handling across all thunks ensures consistent user experience and easier debugging.

```javascript
// Base error for fallback
const baseError = { server: 'Something went wrong' };

// Example thunk with complete error handling
export const thunkExample = () => async (dispatch) => {
  // Always start with loading state
  dispatch(setLoading(true));
  
  try {
    const response = await csrfFetch('/endpoint');
    const data = await response.json();
    
    // Success path
    dispatch(successAction(data));
    dispatch(setErrors(null));  // Clear any existing errors
    return data;  // Return for component usage
    
  } catch (err) {
    // Error path
    dispatch(setErrors(err.errors || baseError));
    return null;  // Consistent return on error
    
  } finally {
    // Always clean up loading state
    dispatch(setLoading(false));
  }
};
```

**What We Learned**:
- Loading states should be handled in try/catch/finally blocks
- Always clear errors on success
- Return values help components handle success/failure
- Use a consistent error format

**Best Practices Discovered**:
- Always include loading states
- Clear errors on success
- Use try/catch/finally
- Return meaningful values

## 3. Payload Handling

**Concept**: API responses might come in different formats or be empty. Robust payload handling prevents runtime errors.

```javascript
// Reducer handling different payload formats
[LOAD_DATA]: (state, action) => {
  const newState = { ...state };
  
  // Handle different response formats and empty cases
  const data = action.payload?.data || action.payload || [];
  
  // Only try to iterate if we have an array
  if (Array.isArray(data)) {
    data.forEach(item => {
      newState.allItems[item.id] = item;
    });
  }
  
  // Always return new state
  return newState;
}
```

**What We Learned**:
- Never assume payload format
- Handle null/undefined gracefully
- Check array type before forEach
- Always return valid state

**Common Issues Solved**:
- undefined.forEach errors
- Null pointer exceptions
- State corruption
- Type errors

## 4. Testing Setup

**Concept**: Comprehensive testing utilities make debugging easier and help verify functionality.

```javascript
// Development testing helpers
if (process.env.NODE_ENV !== 'production') {
  // Expose store and actions for console testing
  window.store = store;
  window.sessionActions = sessionActions;
  window.projectActions = projectActions;

  // Add test helpers
  window.testHelpers = {
    // Authentication testing
    login: async () => {
      const result = await store.dispatch(sessionActions.thunkLogin({
        email: 'demo@aa.io',
        password: 'password'
      }));
      console.log('Login result:', result);
    },

    // Data loading testing
    loadProjects: async () => {
      const result = await store.dispatch(projectActions.thunkLoadProjects());
      console.log('Projects:', result);
    },

    // State inspection
    getState: () => console.log('Current state:', store.getState()),
    getUser: () => console.log('Current user:', store.getState().session.user)
  };
}
```

**What We Learned**:
- Console testing is crucial for development
- Test helpers should be comprehensive
- State inspection tools are essential
- Keep testing code out of production

**Testing Best Practices**:
- Create reusable test helpers
- Test both success and failure paths
- Include state inspection tools
- Make testing easy and accessible
## 5. Authentication Flow & State Management

**Concept**: Managing user sessions and authentication state requires careful coordination between frontend and backend.

```javascript
// Session slice example with proper state management
const initialState = {
  user: null,
  isLoading: false,
  errors: null
};

export const thunkAuthenticate = () => async (dispatch) => {
  dispatch(setLoading(true));
  try {
    const response = await csrfFetch('/auth');
    const data = await response.json();
    dispatch(setUser(data));
    dispatch(setErrors(null));
    return data;
  } catch (err) {
    dispatch(setErrors(err.errors || baseError));
    return null;
  } finally {
    dispatch(setLoading(false));
  }
};
```

**What We Learned**:
- Authentication state affects entire application
- Session restoration is crucial
- Error states must be managed carefully
- Loading states prevent race conditions

## 6. Normalized State Shape

**Concept**: Properly structured Redux state prevents duplication and makes updates efficient.

```javascript
// Example of normalized state structure
const initialState = {
  allItems: {}, // Keyed by ID for O(1) lookup
  byId: {
    1: { id: 1, name: 'Item 1' },
    2: { id: 2, name: 'Item 2' }
  },
  byCategory: {
    'category1': [1, 2], // Reference by ID
    'category2': [2]
  },
  currentItem: null,
  isLoading: false,
  errors: null
};

// Reducer maintaining normalization
const handlers = {
  [ADD_ITEM]: (state, action) => ({
    ...state,
    byId: {
      ...state.byId,
      [action.payload.id]: action.payload
    }
  })
};
```

**What We Learned**:
- Normalize data for efficient updates
- Use IDs as references
- Avoid data duplication
- Structure determines performance

## 7. Relationship Handling

**Concept**: Managing related data (like projects, features, and tasks) requires careful state organization.

```javascript
// Example of handling nested relationships
export const thunkLoadProjectWithDetails = (projectId) => async (dispatch) => {
  dispatch(setLoading(true));
  try {
    // Load main resource
    const project = await dispatch(thunkLoadProject(projectId));
    if (!project) throw new Error('Failed to load project');

    // Load related data in parallel
    await Promise.all([
      dispatch(thunkLoadFeatures(projectId)),
      dispatch(thunkLoadSprints(projectId)),
      dispatch(thunkLoadTasks(projectId))
    ]);

    return project;
  } catch (err) {
    dispatch(setErrors(err.errors || baseError));
    return null;
  } finally {
    dispatch(setLoading(false));
  }
};
```

**What We Learned**:
- Handle related data efficiently
- Use parallel loading where possible
- Maintain data consistency
- Consider loading order

## 8. Debugging Strategies

**Concept**: Effective debugging requires proper tools and strategies.

```javascript
// Redux logging middleware
const loggerMiddleware = store => next => action => {
  console.group(action.type);
  console.log('Previous State:', store.getState());
  console.log('Action:', action);
  const result = next(action);
  console.log('Next State:', store.getState());
  console.groupEnd();
  return result;
};

// Debug helpers
const debugHelpers = {
  checkAuth: () => {
    console.log('CSRF Token:', Cookies.get('csrf_token'));
    console.log('Current User:', store.getState().session.user);
    console.log('Auth Headers:', document.cookie);
  },
  
  testEndpoint: async (url) => {
    try {
      const response = await csrfFetch(url);
      console.log('Response:', await response.json());
    } catch (err) {
      console.error('Error:', err);
    }
  }
};
```

**What We Learned**:
- Console logging is essential
- Track state changes
- Monitor network requests
- Create debugging utilities

