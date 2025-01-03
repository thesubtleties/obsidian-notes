## File Structure:
```
src/
├── actions/
│   ├── types.js
│   ├── userActions.js
│   └── postActions.js
├── reducers/
│   ├── index.js
│   ├── userReducer.js
│   └── postReducer.js
├── store/
│   └── configureStore.js
├── api/
│   └── index.js
└── components/
    └── UserProfile.js
```

## 1. Action Types (src/actions/types.js)

```javascript
export const FETCH_USER_REQUEST = 'FETCH_USER_REQUEST';
export const FETCH_USER_SUCCESS = 'FETCH_USER_SUCCESS';
export const FETCH_USER_FAILURE = 'FETCH_USER_FAILURE';

export const FETCH_POSTS_REQUEST = 'FETCH_POSTS_REQUEST';
export const FETCH_POSTS_SUCCESS = 'FETCH_POSTS_SUCCESS';
export const FETCH_POSTS_FAILURE = 'FETCH_POSTS_FAILURE';
```

## 2. API Service (src/api/index.js)

```javascript
import axios from 'axios';

const API = axios.create({ baseURL: 'https://api.example.com' });

export const fetchUser = (userId) => API.get(`/users/${userId}`);
export const fetchPosts = (userId) => API.get(`/users/${userId}/posts`);
```

## 3. Thunk Action Creators (src/actions/userActions.js)

```javascript
import * as api from '../api';
import { 
  FETCH_USER_REQUEST, 
  FETCH_USER_SUCCESS, 
  FETCH_USER_FAILURE 
} from './types';

export const fetchUser = (userId) => async (dispatch) => {
  dispatch({ type: FETCH_USER_REQUEST });
  try {
    const { data } = await api.fetchUser(userId);
    dispatch({ type: FETCH_USER_SUCCESS, payload: data });
  } catch (error) {
    dispatch({ type: FETCH_USER_FAILURE, payload: error.message });
  }
};

// Thunk with parameters and getState
export const updateUserIfChanged = (userId, userData) => async (dispatch, getState) => {
  const currentUser = getState().user.data;
  if (JSON.stringify(currentUser) !== JSON.stringify(userData)) {
    // Perform update logic here
  }
};
```

## 4. Chaining Thunks (src/actions/postActions.js)

```javascript
import * as api from '../api';
import { fetchUser } from './userActions';
import {
  FETCH_POSTS_REQUEST,
  FETCH_POSTS_SUCCESS,
  FETCH_POSTS_FAILURE
} from './types';

export const fetchPosts = (userId) => async (dispatch) => {
  dispatch({ type: FETCH_POSTS_REQUEST });
  try {
    const { data } = await api.fetchPosts(userId);
    dispatch({ type: FETCH_POSTS_SUCCESS, payload: data });
  } catch (error) {
    dispatch({ type: FETCH_POSTS_FAILURE, payload: error.message });
  }
};

// Chaining thunks
export const fetchUserAndPosts = (userId) => async (dispatch) => {
  await dispatch(fetchUser(userId));
  dispatch(fetchPosts(userId));
};
```

## 5. Reducer (src/reducers/userReducer.js)

```javascript
import {
  FETCH_USER_REQUEST,
  FETCH_USER_SUCCESS,
  FETCH_USER_FAILURE
} from '../actions/types';

const initialState = {
  data: null,
  loading: false,
  error: null
};

export default function userReducer(state = initialState, action) {
  switch (action.type) {
    case FETCH_USER_REQUEST:
      return { ...state, loading: true, error: null };
    case FETCH_USER_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_USER_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
}
```

## 6. Root Reducer (src/reducers/index.js)

```javascript
import { combineReducers } from 'redux';
import userReducer from './userReducer';
import postReducer from './postReducer';

export default combineReducers({
  user: userReducer,
  posts: postReducer
});
```

## 7. Store Configuration (src/store/configureStore.js)

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

const configureStore = () => {
  return createStore(rootReducer, applyMiddleware(thunk));
};

export default configureStore;
```

## 8. Using Thunks in Components (src/components/UserProfile.js)

```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchUserAndPosts } from '../actions/postActions';

const UserProfile = ({ userId }) => {
  const dispatch = useDispatch();
  const { user, posts, loading, error } = useSelector(state => ({
    user: state.user.data,
    posts: state.posts.data,
    loading: state.user.loading || state.posts.loading,
    error: state.user.error || state.posts.error
  }));

  useEffect(() => {
    dispatch(fetchUserAndPosts(userId));
  }, [dispatch, userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <h2>Posts:</h2>
      <ul>
        {posts.map(post => (
          <li key={post.id}>{post.title}</li>
        ))}
      </ul>
    </div>
  );
};

export default UserProfile;
```

Key Points:
1. Thunks are defined in action creator files.
2. They can interact with APIs, dispatch multiple actions, and access the current state.
3. Reducers handle the actions dispatched by thunks to update the state.
4. The store needs to be configured with the thunk middleware.
5. Components can dispatch thunks and access the resulting state using hooks like `useDispatch` and `useSelector`.

This structure provides a clear separation of concerns and makes it easy to manage complex asynchronous flows in a Redux application.