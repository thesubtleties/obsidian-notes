## File Structure

```
src/
├── actions/
│   └── todoActions.js
├── reducers/
│   ├── index.js (root reducer)
│   └── todoReducer.js
├── selectors/
│   └── todoSelectors.js
├── store/
│   └── configureStore.js
├── components/
│   └── TodoList.js
├── api/
│   └── todoApi.js
└── App.js
```

## 1. Actions and Thunks (src/actions/todoActions.js)

```javascript
import * as todoApi from '../api/todoApi';

// Action Types
export const ADD_TODO = 'ADD_TODO';
export const FETCH_TODOS_START = 'FETCH_TODOS_START';
export const FETCH_TODOS_SUCCESS = 'FETCH_TODOS_SUCCESS';
export const FETCH_TODOS_FAILURE = 'FETCH_TODOS_FAILURE';

// Action Creators
export const addTodo = (todo) => ({ type: ADD_TODO, payload: todo });

// Thunks
export const fetchTodos = () => async (dispatch) => {
  dispatch({ type: FETCH_TODOS_START });
  try {
    const todos = await todoApi.fetchTodos();
    dispatch({ type: FETCH_TODOS_SUCCESS, payload: todos });
  } catch (error) {
    dispatch({ type: FETCH_TODOS_FAILURE, payload: error.message });
  }
};

export const addTodoAsync = (text) => async (dispatch) => {
  try {
    const newTodo = await todoApi.addTodo(text);
    dispatch(addTodo(newTodo));
  } catch (error) {
    console.error('Failed to add todo:', error);
  }
};
```

## 2. Reducer (src/reducers/todoReducer.js)

```javascript
import { ADD_TODO, FETCH_TODOS_START, FETCH_TODOS_SUCCESS, FETCH_TODOS_FAILURE } from '../actions/todoActions';

const initialState = { todos: [], loading: false, error: null };

export default function todoReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return { ...state, todos: [...state.todos, action.payload] };
    case FETCH_TODOS_START:
      return { ...state, loading: true };
    case FETCH_TODOS_SUCCESS:
      return { ...state, todos: action.payload, loading: false };
    case FETCH_TODOS_FAILURE:
      return { ...state, error: action.payload, loading: false };
    default:
      return state;
  }
}
```

## 3. Root Reducer (src/reducers/index.js)

```javascript
import { combineReducers } from 'redux';
import todoReducer from './todoReducer';
import userReducer from './userReducer';

export default combineReducers({
  todos: todoReducer,
  user: userReducer,
});
```

## 4. Selectors (src/selectors/todoSelectors.js)

```javascript
export const getTodos = state => state.todos.todos;
export const getLoading = state => state.todos.loading;
export const getError = state => state.todos.error;

export const getVisibleTodos = state => {
  const todos = getTodos(state);
  const filter = state.todos.visibilityFilter;
  // Apply filter logic
  return filteredTodos;
};
```

## 5. Store Configuration (src/store/configureStore.js)

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

const configureStore = () => {
  return createStore(rootReducer, applyMiddleware(thunk));
};

export default configureStore;
```

## 6. API (src/api/todoApi.js)

```javascript
export const fetchTodos = async () => {
  const response = await fetch('/api/todos');
  if (!response.ok) throw new Error('Failed to fetch todos');
  return response.json();
};

export const addTodo = async (text) => {
  const response = await fetch('/api/todos', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ text })
  });
  if (!response.ok) throw new Error('Failed to add todo');
  return response.json();
};
```

## 7. React Component (src/components/TodoList.js)

```javascript
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchTodos, addTodoAsync } from '../actions/todoActions';
import { getVisibleTodos, getLoading, getError } from '../selectors/todoSelectors';

function TodoList() {
  const dispatch = useDispatch();
  const todos = useSelector(getVisibleTodos);
  const loading = useSelector(getLoading);
  const error = useSelector(getError);

  useEffect(() => {
    dispatch(fetchTodos());
  }, [dispatch]);

  const handleAddTodo = (text) => {
    dispatch(addTodoAsync(text));
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
      <button onClick={() => handleAddTodo('New Todo')}>Add Todo</button>
    </div>
  );
}

export default TodoList;
```

## 8. App Setup (src/App.js)

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

Key Points:
- Thunks are defined in the actions file alongside regular action creators.
- The API layer is separated, allowing for easy mocking and testing.
- The store is configured with the thunk middleware.
- Components dispatch both regular actions and thunks using `useDispatch`.

This structure separates concerns while allowing for complex asynchronous operations through thunks.