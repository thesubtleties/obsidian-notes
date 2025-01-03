## Standard Redux Structure:

1. Store setup (in store.js):
```javascript
const store = createStore(rootReducer, applyMiddleware(thunk));
```

2. Action creators (in actions/todoActions.js):
```javascript
export const addTodo = (text) => ({ type: 'ADD_TODO', payload: text });
```

3. Thunk action creators (in actions/todoActions.js):
```javascript
export const addTodoAsync = (text) => {
  return (dispatch) => {
    dispatch({ type: 'ADD_TODO_START' });
    setTimeout(() => {
      dispatch({ type: 'ADD_TODO_SUCCESS', payload: text });
    }, 1000);
  };
};
```

4. Reducer (in reducers/todoReducer.js):
```javascript
export default function todoReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO_SUCCESS':
      return [...state, { text: action.payload, completed: false }];
    default:
      return state;
  }
}
```

5. Root reducer (in reducers/index.js):
```javascript
import { combineReducers } from 'redux';
import todoReducer from './todoReducer';

export default combineReducers({
  todos: todoReducer,
  // other reducers...
});
```

6. Component usage (in components/TodoList.js):
```javascript
import { useSelector, useDispatch } from 'react-redux';
import { addTodoAsync } from '../actions/todoActions';

function TodoList() {
  const todos = useSelector(state => state.todos);
  const dispatch = useDispatch();

  const handleAddTodo = (text) => {
    dispatch(addTodoAsync(text));
  };
  // render logic...
}
```

## Ducks-style Redux Structure:

1. Store setup (in store.js) - remains the same:
```javascript
const store = createStore(rootReducer, applyMiddleware(thunk));
```

2. Duck module (in ducks/todoDuck.js) - combines actions, action creators, and reducer:
```javascript
// Action Types
const ADD_TODO = 'myApp/todos/ADD_TODO';
const ADD_TODO_START = 'myApp/todos/ADD_TODO_START';
const ADD_TODO_SUCCESS = 'myApp/todos/ADD_TODO_SUCCESS';

// Action Creators
export const addTodo = (text) => ({ type: ADD_TODO, payload: text });

// Thunk Action Creators
export const addTodoAsync = (text) => {
  return (dispatch) => {
    dispatch({ type: ADD_TODO_START });
    setTimeout(() => {
      dispatch({ type: ADD_TODO_SUCCESS, payload: text });
    }, 1000);
  };
};

// Reducer
const initialState = [];

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO_SUCCESS:
      return [...state, { text: action.payload, completed: false }];
    default:
      return state;
  }
}

// Selectors
export const getTodos = state => state.todos;
```

3. Root reducer (in ducks/index.js):
```javascript
import { combineReducers } from 'redux';
import todoReducer from './todoDuck';

export default combineReducers({
  todos: todoReducer,
  // other reducers...
});
```

4. Component usage (in components/TodoList.js) - similar to standard structure:
```javascript
import { useSelector, useDispatch } from 'react-redux';
import { addTodoAsync, getTodos } from '../ducks/todoDuck';

function TodoList() {
  const todos = useSelector(getTodos);
  const dispatch = useDispatch();

  const handleAddTodo = (text) => {
    dispatch(addTodoAsync(text));
  };
  // render logic...
}
```

The main difference in the Ducks pattern is that all related Redux logic (action types, action creators, reducer, and even selectors) for a specific feature are contained in a single file, typically named after the feature (e.g., todoDuck.js). This approach aims to improve modularity and reduce the need to switch between multiple files when working on a single feature.