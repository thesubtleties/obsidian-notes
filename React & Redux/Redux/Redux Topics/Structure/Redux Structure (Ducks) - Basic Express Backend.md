1. src/index.js (Entry point)

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import App from './App';
import store from './store';

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
```

2. src/store/index.js (Redux store configuration)

```javascript
import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import userReducer from './userSlice';

const rootReducer = combineReducers({
  user: userReducer,
  // Add other reducers here as needed
});

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

3. src/store/userSlice.js (Ducks-style slice for user-related Redux logic)

```javascript
import { apiCall } from '../utils/api';

// Action Types
const SET_USER = 'user/setUser';
const LOGOUT = 'user/logout';
const SET_LOADING = 'user/setLoading';

// Action Creators
export const setUser = (user) => ({ type: SET_USER, payload: user });
export const logout = () => ({ type: LOGOUT });
export const setLoading = (isLoading) => ({ type: SET_LOADING, payload: isLoading });

// Thunks
export const login = (credentials) => async (dispatch) => {
  dispatch(setLoading(true));
  try {
    const user = await apiCall('/login', 'POST', credentials);
    dispatch(setUser(user));
  } catch (error) {
    console.error('Login failed:', error);
  } finally {
    dispatch(setLoading(false));
  }
};

export const checkAuthStatus = () => async (dispatch) => {
  try {
    const user = await apiCall('/auth/me');
    dispatch(setUser(user));
  } catch (error) {
    console.error('Auth check failed:', error);
  }
};

// Initial State
const initialState = {
  currentUser: null,
  isLoading: false,
};

// Reducer
export default function userReducer(state = initialState, action) {
  switch (action.type) {
    case SET_USER:
      return { ...state, currentUser: action.payload };
    case LOGOUT:
      return { ...state, currentUser: null };
    case SET_LOADING:
      return { ...state, isLoading: action.payload };
    default:
      return state;
  }
}

// Selectors
export const selectUser = (state) => state.user.currentUser;
export const selectIsLoading = (state) => state.user.isLoading;
```

4. src/App.js (Main application component)

```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { Route, Switch } from 'react-router-dom';
import { checkAuthStatus, selectIsLoading } from './store/userSlice';
import Home from './components/Home';
import Login from './components/Login';
import Profile from './components/Profile';
import Navigation from './components/Navigation';

function App() {
  const dispatch = useDispatch();
  const isLoading = useSelector(selectIsLoading);

  useEffect(() => {
    dispatch(checkAuthStatus());
  }, [dispatch]);

  if (isLoading) return <div>Loading...</div>;

  return (
    <div className="App">
      <Navigation />
      <Switch>
        <Route exact path="/" component={Home} />
        <Route path="/login" component={Login} />
        <Route path="/profile" component={Profile} />
      </Switch>
    </div>
  );
}

export default App;
```

5. src/components/Navigation.js

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { Link } from 'react-router-dom';
import { selectUser, logout } from '../store/userSlice';

function Navigation() {
  const user = useSelector(selectUser);
  const dispatch = useDispatch();

  return (
    <nav>
      <ul>
        <li><Link to="/">Home</Link></li>
        {user ? (
          <>
            <li><Link to="/profile">Profile</Link></li>
            <li><button onClick={() => dispatch(logout())}>Logout</button></li>
          </>
        ) : (
          <li><Link to="/login">Login</Link></li>
        )}
      </ul>
    </nav>
  );
}

export default Navigation;
```

6. src/components/Login.js

```javascript
import React, { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { login, selectIsLoading } from '../store/userSlice';

function Login() {
  const dispatch = useDispatch();
  const isLoading = useSelector(selectIsLoading);
  const [credentials, setCredentials] = useState({ username: '', password: '' });

  const handleChange = (e) => {
    setCredentials({ ...credentials, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch(login(credentials));
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="username"
        value={credentials.username}
        onChange={handleChange}
        placeholder="Username"
      />
      <input
        type="password"
        name="password"
        value={credentials.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit" disabled={isLoading}>
        {isLoading ? 'Logging in...' : 'Login'}
      </button>
    </form>
  );
}

export default Login;
```

7. src/utils/api.js

```javascript
const API_BASE_URL = '/api';

export async function apiCall(endpoint, method = 'GET', body = null) {
  const options = {
    method,
    headers: {
      'Content-Type': 'application/json',
    },
  };

  if (body) {
    options.body = JSON.stringify(body);
  }

  const response = await fetch(`${API_BASE_URL}${endpoint}`, options);

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.message || 'An error occurred');
  }

  return response.json();
}
```

This setup provides a complete basic structure for a React-Redux application using the Ducks pattern for the Redux logic. It includes:

- Redux store configuration
- A Ducks-style slice for user-related state management
- Main App component with routing
- Navigation component
- Login component
- API utility for making fetch calls

Remember to create other necessary components (like Home and Profile) and additional slices as your application grows. This structure gives you a solid foundation to build upon, keeping your Redux logic organized and your components connected to the store.