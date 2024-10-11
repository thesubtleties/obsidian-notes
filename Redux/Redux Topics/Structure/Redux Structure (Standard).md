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
│   └── store.js
├── components/
│   └── TodoList.js
└── App.js
```

## 1. Actions (src/actions/todoActions.js)

```javascript
export const addTodo = (text) => ({ type: 'ADD_TODO', payload: text });

export const fetchTodos = () => async (dispatch) => {
  dispatch({ type: 'FETCH_TODOS_START' });
  try {
    const todos = await api.getTodos();
    dispatch({ type: 'FETCH_TODOS_SUCCESS', payload: todos });
  } catch (error) {
    dispatch({ type: 'FETCH_TODOS_FAILURE', payload: error.message });
  }
};
```

## 2. Reducer (src/reducers/todoReducer.js)

```javascript
const initialState = { todos: [], loading: false, error: null };

export default function todoReducer(state = initialState, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return { ...state, todos: [...state.todos, action.payload] };
    case 'FETCH_TODOS_SUCCESS':
      return { ...state, todos: action.payload, loading: false };
    // ... other cases
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
export const getVisibleTodos = state => {
  const todos = getTodos(state);
  const filter = state.todos.visibilityFilter;
  // Apply filter logic
  return filteredTodos;
};
```

## 5. Store Configuration (src/store/store.js)

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

## 6. React Component (src/components/TodoList.js)

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { addTodo, fetchTodos } from '../actions/todoActions';
import { getVisibleTodos } from '../selectors/todoSelectors';

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

## 7. App Setup (src/App.js)

```javascript
import React from 'react';
import { Provider } from 'react-redux';
import store from './store/store';
import TodoList from './components/TodoList';

function App() {
  return (
    <Provider store={store}>
      <TodoList />
    </Provider>
  );
}

export default App;
```

Remember:
- Actions define what happened
- Reducers specify how state changes in response to actions
- Selectors extract and compute derived data from the state
- Store holds the state tree of the application
- Components use useSelector to access state and useDispatch to dispatch actions

This structure separates concerns and makes the application more maintainable and scalable.