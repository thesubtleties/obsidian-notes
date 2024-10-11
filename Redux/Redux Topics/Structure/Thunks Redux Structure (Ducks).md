## File Structure (unchanged)

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

## Duck Module (src/ducks/todos.js)

```javascript
import api from '../api'; // Assume this is your API service

// Action Types
const ADD_TODO = 'myApp/todos/ADD_TODO';
const FETCH_TODOS_START = 'myApp/todos/FETCH_TODOS_START';
const FETCH_TODOS_SUCCESS = 'myApp/todos/FETCH_TODOS_SUCCESS';
const FETCH_TODOS_FAILURE = 'myApp/todos/FETCH_TODOS_FAILURE';

// Action Creators
export const addTodo = (text) => ({ type: ADD_TODO, payload: text });

// Thunks
export const fetchTodos = () => async (dispatch) => {
  dispatch({ type: FETCH_TODOS_START });
  try {
    const todos = await api.getTodos();
    dispatch({ type: FETCH_TODOS_SUCCESS, payload: todos });
  } catch (error) {
    dispatch({ type: FETCH_TODOS_FAILURE, payload: error.message });
  }
};

export const addTodoAsync = (text) => async (dispatch) => {
  try {
    const newTodo = await api.addTodo(text);
    dispatch(addTodo(newTodo));
  } catch (error) {
    console.error('Failed to add todo:', error);
  }
};

// Reducer
const initialState = { todos: [], loading: false, error: null };

export default function reducer(state = initialState, action) {
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

// Selectors
export const getTodos = state => state.todos.todos;
export const getLoading = state => state.todos.loading;
export const getError = state => state.todos.error;
```

## Key Points About Thunks in Ducks

1. Thunks are defined alongside other action creators in the duck file.
2. They are typically exported like regular action creators.
3. Thunks can dispatch multiple actions, including other action creators defined in the same file.
4. They can interact with external services (like API calls) before dispatching actions.

## Using Thunks in Components

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodoAsync, fetchTodos, getTodos, getLoading, getError } from '../ducks/todos';

function TodoList() {
  const dispatch = useDispatch();
  const todos = useSelector(getTodos);
  const loading = useSelector(getLoading);
  const error = useSelector(getError);

  React.useEffect(() => {
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
```

Remember:
- Thunks allow you to write action creators that return a function instead of an action object.
- They are useful for async logic, complex synchronous logic, or any logic that needs to dispatch multiple actions.
- In the Ducks pattern, thunks are part of the duck module, keeping all related logic together.
- The thunk middleware must be applied to the store for thunks to work (this is done in the store configuration file).

This structure keeps all todo-related logic, including asynchronous operations, in one place, making it easier to manage and understand the feature as a whole.