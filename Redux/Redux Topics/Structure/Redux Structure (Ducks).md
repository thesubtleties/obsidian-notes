## File Structure

```
src/
├── ducks/
│   ├── index.js (root reducer)
│   └── todos.js
├── store/
│   └── configureStore.js
├── components/
│   └── TodoList.js
└── App.js
```

## 1. Duck Module (src/ducks/todos.js)

```javascript
// Action Types
const ADD_TODO = 'myApp/todos/ADD_TODO';
const FETCH_TODOS_START = 'myApp/todos/FETCH_TODOS_START';
const FETCH_TODOS_SUCCESS = 'myApp/todos/FETCH_TODOS_SUCCESS';
const FETCH_TODOS_FAILURE = 'myApp/todos/FETCH_TODOS_FAILURE';

// Action Creators
export const addTodo = (text) => ({ type: ADD_TODO, payload: text });

export const fetchTodos = () => async (dispatch) => {
  dispatch({ type: FETCH_TODOS_START });
  try {
    const todos = await api.getTodos();
    dispatch({ type: FETCH_TODOS_SUCCESS, payload: todos });
  } catch (error) {
    dispatch({ type: FETCH_TODOS_FAILURE, payload: error.message });
  }
};

// Reducer
const initialState = { todos: [], loading: false, error: null };

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return { ...state, todos: [...state.todos, action.payload] };
    case FETCH_TODOS_SUCCESS:
      return { ...state, todos: action.payload, loading: false };
    // ... other cases
    default:
      return state;
  }
}

// Selectors
export const getTodos = state => state.todos.todos;
export const getVisibleTodos = state => {
  const todos = getTodos(state);
  const filter = state.todos.visibilityFilter;
  // Apply filter logic
  return filteredTodos;
};
```

## 2. Root Reducer (src/ducks/index.js)

```javascript
import { combineReducers } from 'redux';
import todosReducer from './todos';
import userReducer from './user';

export default combineReducers({
  todos: todosReducer,
  user: userReducer,
});
```

## 3. Store Configuration (src/store/configureStore.js)

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../ducks';

const configureStore = () => {
  return createStore(rootReducer, applyMiddleware(thunk));
};

export default configureStore;
```

## 4. React Component (src/components/TodoList.js)

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, fetchTodos, getVisibleTodos } from '../ducks/todos';

function TodoList() {
  const dispatch = useDispatch();
  const todos = useSelector(getVisibleTodos);

  React.useEffect(() => {
    dispatch(fetchTodos());
  }, [dispatch]);

  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
      <button onClick={() => dispatch(addTodo('New Todo'))}>Add Todo</button>
    </ul>
  );
}
```

## 5. App Setup (src/App.js)

```javascript
import React from 'react';
import { Provider } from 'react-redux';
import configureStore from './store/configureStore';
import TodoList from './components/TodoList';

const store = configureStore();

function App() {
  return (
    <Provider store={store}>
      <TodoList />
    </Provider>
  );
}

export default App;
```

## Key Differences in Ducks Pattern

1. All Redux-related logic for a feature is in one file (action types, action creators, reducer, and selectors).
2. Action types are typically prefixed with the module name to avoid conflicts.
3. The reducer is the default export, while action creators and selectors are named exports.
4. Components import everything they need from a single duck module.

Remember:
- Each duck module represents a discrete domain of your application.
- This pattern reduces the need to switch between multiple files when working on a feature.
- It can make it easier to understand and maintain each feature in isolation.
- Be cautious of file size as features grow; consider splitting large ducks into sub-ducks if necessary.

This structure promotes modularity and encapsulation of feature-specific Redux logic.