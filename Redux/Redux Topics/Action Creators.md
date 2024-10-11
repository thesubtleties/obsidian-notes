## Basic Action Creator

```javascript
const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    payload: text
  };
};
```

## Action Creator with Multiple Arguments

```javascript
const updateUser = (id, name, email) => {
  return {
    type: 'UPDATE_USER',
    payload: { id, name, email }
  };
};
```

## Action Types as Constants

```javascript
const ADD_TODO = 'ADD_TODO';
const REMOVE_TODO = 'REMOVE_TODO';

const addTodo = (text) => ({ type: ADD_TODO, payload: text });
const removeTodo = (id) => ({ type: REMOVE_TODO, payload: id });
```

## Async Action Creator (Thunk)

```javascript
const fetchUser = (userId) => {
  return async (dispatch) => {
    dispatch({ type: 'FETCH_USER_START' });
    try {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      dispatch({ type: 'FETCH_USER_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_USER_FAILURE', payload: error.message });
    }
  };
};
```

## Action Creator with Conditional Logic

```javascript
const incrementIfOdd = (currentValue) => {
  return (dispatch) => {
    if (currentValue % 2 !== 0) {
      dispatch({ type: 'INCREMENT' });
    }
  };
};
```

## Action Creator Returning Multiple Actions

```javascript
const completeAndNotify = (todoId) => {
  return (dispatch) => {
    dispatch({ type: 'COMPLETE_TODO', payload: todoId });
    dispatch({ type: 'SHOW_NOTIFICATION', payload: 'Todo completed!' });
  };
};
```

## Action Creator with Parameters and Getstate

```javascript
const addTodoIfNotExists = (text) => {
  return (dispatch, getState) => {
    const todos = getState().todos;
    if (!todos.find(todo => todo.text === text)) {
      dispatch({ type: 'ADD_TODO', payload: text });
    }
  };
};
```

## Action Creator Factory

```javascript
const createItemAction = (itemType) => {
  return (item) => ({
    type: `ADD_${itemType.toUpperCase()}`,
    payload: item
  });
};

const addBook = createItemAction('book');
const addMovie = createItemAction('movie');
```

## Key Points:

1. Action creators are functions that return action objects.
2. Actions must have a `type` property; `payload` is a common convention for data.
3. Use constants for action types to avoid typos and improve maintainability.
4. Async action creators typically use middleware like Redux Thunk.
5. Thunks allow you to dispatch multiple actions and perform async operations.
6. Action creators can access current state using `getState()` in thunks.
7. Keep action creators pure and predictable for easier testing and debugging.
8. Use factories to create similar action creators and reduce code duplication.

Remember: Action creators help encapsulate the process of creating actions, making your code more maintainable and easier to test.