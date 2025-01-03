## Key Concepts

1. Bundling
   - Combines actions, action creators, and reducers in one file
   - Each module (duck) manages a specific domain of your app

2. Naming Convention
   - Use `UPPER_SNAKE_CASE` for action types
   - Prefix action types with the module name to avoid conflicts

3. Structure
   - Export reducer as default
   - Export action creators as named exports
   - Define action types as constants

## Basic Duck Structure

```javascript
// Action Types
const ADD_TODO = 'myApp/todos/ADD_TODO';
const TOGGLE_TODO = 'myApp/todos/TOGGLE_TODO';

// Action Creators
export const addTodo = (text) => ({
  type: ADD_TODO,
  payload: { text }
});

export const toggleTodo = (id) => ({
  type: TOGGLE_TODO,
  payload: { id }
});

// Reducer
const initialState = [];

export default function reducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return [...state, { text: action.payload.text, completed: false }];
    case TOGGLE_TODO:
      return state.map(todo =>
        todo.id === action.payload.id ? { ...todo, completed: !todo.completed } : todo
      );
    default:
      return state;
  }
}
```

## Key Points

- Each duck should correspond to a specific feature or domain
- Action types should be unique across the entire app
- The reducer is the default export
- Action creators are named exports

## Best Practices

1. Use a consistent file naming convention (e.g., `featureName.js`)
2. Keep each duck focused on a single part of your state
3. Use selectors for accessing state in components

## Integration with Root Reducer

```javascript
import { combineReducers } from 'redux';
import todosReducer from './ducks/todos';
import usersReducer from './ducks/users';

const rootReducer = combineReducers({
  todos: todosReducer,
  users: usersReducer,
});
```

## Advantages

- Keeps related code together
- Easier to understand and maintain
- Reduces boilerplate
- Simplifies importing in components

Remember: The Ducks pattern is not an official Redux pattern, but it's widely used in the community for its simplicity and organization benefits. It works well for small to medium-sized applications but may need to be adapted for larger, more complex projects.