## File Structure
```
src/
├── actions/
│   └── userActions.js
├── reducers/
│   ├── index.js
│   └── userReducer.js
├── store/
│   └── configureStore.js
├── api/
│   └── index.js
└── components/
    └── UserComponent.js
```

## API Service (src/api/index.js)

```javascript
const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

export const fetchUser = async (userId) => {
  const response = await fetch(`${API_URL}/users/${userId}`);
  if (!response.ok) throw new Error('Network response was not ok');
  return response.json();
};

export const createUser = async (userData) => {
  const response = await fetch(`${API_URL}/users`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(userData),
  });
  if (!response.ok) throw new Error('Network response was not ok');
  return response.json();
};
```

## Action Types (src/actions/types.js)

```javascript
export const FETCH_USER_REQUEST = 'FETCH_USER_REQUEST';
export const FETCH_USER_SUCCESS = 'FETCH_USER_SUCCESS';
export const FETCH_USER_FAILURE = 'FETCH_USER_FAILURE';
```

## Action Creators (src/actions/userActions.js)

```javascript
import * as api from '../api';
import * as types from './types';

export const fetchUser = (userId) => async (dispatch) => {
  dispatch({ type: types.FETCH_USER_REQUEST });
  try {
    const data = await api.fetchUser(userId);
    dispatch({ type: types.FETCH_USER_SUCCESS, payload: data });
  } catch (error) {
    dispatch({ type: types.FETCH_USER_FAILURE, payload: error.message });
  }
};

export const createUser = (userData) => async (dispatch) => {
  dispatch({ type: 'CREATE_USER_REQUEST' });
  try {
    const data = await api.createUser(userData);
    dispatch({ type: 'CREATE_USER_SUCCESS', payload: data });
  } catch (error) {
    dispatch({ type: 'CREATE_USER_FAILURE', payload: error.message });
  }
};
```

## Reducer (src/reducers/userReducer.js)

```javascript
import * as types from '../actions/types';

const initialState = {
  data: null,
  loading: false,
  error: null,
};

const userReducer = (state = initialState, action) => {
  switch (action.type) {
    case types.FETCH_USER_REQUEST:
      return { ...state, loading: true, error: null };
    case types.FETCH_USER_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case types.FETCH_USER_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

export default userReducer;
```

## Root Reducer (src/reducers/index.js)

```javascript
import { combineReducers } from 'redux';
import userReducer from './userReducer';

const rootReducer = combineReducers({
  user: userReducer,
  // other reducers...
});

export default rootReducer;
```

## Store Configuration (src/store/configureStore.js)

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

const configureStore = () => {
  return createStore(rootReducer, applyMiddleware(thunk));
};

export default configureStore;
```

## Using in React Component (src/components/UserComponent.js)

```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUser } from '../actions/userActions';

const UserComponent = ({ userId }) => {
  const dispatch = useDispatch();
  const { data: user, loading, error } = useSelector(state => state.user);

  useEffect(() => {
    dispatch(fetchUser(userId));
  }, [dispatch, userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      {/* Rest of your component */}
    </div>
  );
};

export default UserComponent;
```

## Key Points:

1. The API service uses Fetch API to communicate with the backend.
2. Action creators are async functions that dispatch actions.
3. We use Redux Thunk middleware to handle async action creators.
4. The reducer handles the state updates based on dispatched actions.
5. Components use `useDispatch` to dispatch actions and `useSelector` to access state.
6. Error handling is implemented at multiple levels (API, action creators, component).

Remember to wrap your root component with `Provider` from react-redux and pass the store to it:

```javascript
import React from 'react';
import { Provider } from 'react-redux';
import configureStore from './store/configureStore';
import App from './App';

const store = configureStore();

const Root = () => (
  <Provider store={store}>
    <App />
  </Provider>
);

export default Root;
```

This setup integrates Redux with your React application, handling API calls using the Fetch API, and managing state updates through Redux actions and reducers.