## Core Concepts

1. Store
   - Single source of truth for the entire application state
   - Created using `createStore(reducer, [preloadedState], [enhancer])`

2. Actions
   - Plain JavaScript objects describing what happened
   - Must have a `type` property
   - Example: `{ type: 'ADD_TODO', text: 'Buy milk' }`

3. Reducers
   - Pure functions that specify how state changes
   - `(state, action) => newState`
   - Never mutate state directly; always return a new state object

4. Dispatch
   - Method to send actions to the store
   - `store.dispatch(action)`

5. Selectors
   - Functions to extract specific pieces of state
   - Helps with performance optimization and encapsulation

## Setting Up Redux

```javascript
import { createStore } from 'redux';

const reducer = (state = initialState, action) => {
  switch (action.type) {
    case 'ACTION_TYPE':
      return { ...state, /* updates */ };
    default:
      return state;
  }
};

const store = createStore(reducer);
```

## [[Action Creators]]

```javascript
const addTodo = (text) => ({
  type: 'ADD_TODO',
  payload: { text }
});
```

## Combining Reducers

```javascript
import { combineReducers } from 'redux';

const rootReducer = combineReducers({
  todos: todosReducer,
  user: userReducer
});
```

## Middleware

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';

const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```

## Async Actions with Thunk

```javascript
const fetchTodos = () => {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_TODOS_START' });
    try {
      const response = await fetch('/api/todos');
      const todos = await response.json();
      dispatch({ type: 'FETCH_TODOS_SUCCESS', payload: todos });
    } catch (error) {
      dispatch({ type: 'FETCH_TODOS_FAILURE', error });
    }
  };
};
```

## Connecting to React

```javascript
import { Provider } from 'react-redux';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

## Using in Components

```javascript
import { useSelector, useDispatch } from 'react-redux';

function MyComponent() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch(addTodo('New Todo'))}>
      Add Todo
    </button>
  );
}
```

## [[Ducks Redux Reducer Pattern|Ducks Pattern]]

The Ducks pattern is a methodology for organizing Redux code. It suggests bundling related action types, action creators, and reducers in a single file.

```javascript
// userDuck.js

// Action Types
const LOGIN = 'app/user/LOGIN';
const LOGOUT = 'app/user/LOGOUT';

// Action Creators
export const login = (username) => ({ type: LOGIN, payload: username });
export const logout = () => ({ type: LOGOUT });

// Reducer
const initialState = { username: null, isLoggedIn: false };

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case LOGIN:
      return { ...state, username: action.payload, isLoggedIn: true };
    case LOGOUT:
      return { ...state, username: null, isLoggedIn: false };
    default:
      return state;
  }
}

// Optional: Selectors
export const getUsername = (state) => state.user.username;
export const getLoginStatus = (state) => state.user.isLoggedIn;
```

Key points:
- Combines actions, action creators, and reducer in one file
- Typically named after the state slice they manage
- Promotes modularity and encapsulation
- Often used with Redux Toolkit's `createSlice`
## Redux Toolkit (Modern Redux)

```javascript
import { configureStore, createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    addTodo: (state, action) => {
      state.push(action.payload);
    }
  }
});

const store = configureStore({
  reducer: {
    todos: todosSlice.reducer
  }
});

export const { addTodo } = todosSlice.actions;
```

## Key Principles

1. Single source of truth
2. State is read-only
3. Changes are made with pure functions

## Best Practices

1. Keep state normalized
2. Use selector functions to read state
3. Use Redux Toolkit for boilerplate reduction
4. Avoid deeply nested state
5. Use middleware for side effects (e.g., API calls)

## Debugging

- Use Redux DevTools extension
- Log actions and state changes
- Time-travel debugging

Remember: Redux is a predictable state container designed to help you write JavaScript apps that behave consistently across client, server, and native environments.