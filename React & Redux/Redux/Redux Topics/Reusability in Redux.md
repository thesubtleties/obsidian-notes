## 1. Action Creators

```javascript
const createAction = (type) => (payload) => ({ type, payload });

const increment = createAction('INCREMENT');
const setUser = createAction('SET_USER');

// Usage
dispatch(increment(5));
dispatch(setUser({ id: 1, name: 'John' }));
```

Explanation: This pattern creates a reusable function for generating action creators. It reduces boilerplate and ensures consistency in action structure across your application. The `createAction` function takes a type and returns a function that accepts a payload, creating a standardized action object.

## 2. Reusable Reducer Logic

```javascript
const createReducer = (initialState, handlers) => 
  (state = initialState, action) => 
    handlers.hasOwnProperty(action.type) 
      ? handlers[action.type](state, action) 
      : state;

const counterReducer = createReducer(0, {
  INCREMENT: (state, action) => state + action.payload,
  DECREMENT: (state, action) => state - action.payload
});
```

Explanation: This pattern creates a function that generates reducers based on an object of handlers. It eliminates the need for switch statements in reducers, making them more readable and easier to maintain. Each key in the handlers object corresponds to an action type, and the value is the function to handle that action.

## 3. Higher-Order Reducers

```javascript
const withLoading = (reducer) => (state, action) => {
  switch (action.type) {
    case 'FETCH_START':
      return { ...state, loading: true };
    case 'FETCH_END':
      return { ...state, loading: false };
    default:
      return reducer(state, action);
  }
};

const userReducer = withLoading((state, action) => {
  // User-specific reducer logic
});
```

Explanation: Higher-order reducers allow you to wrap existing reducers with additional functionality. In this example, `withLoading` adds loading state management to any reducer it wraps. This pattern is useful for adding cross-cutting concerns to multiple reducers without duplicating code.

## 4. Reusable Selectors

```javascript
const createSelector = (...funcs) => (state) => {
  const last = funcs.pop();
  return last(...funcs.map(f => f(state)));
};

const selectUsers = state => state.users;
const selectActiveUserId = state => state.activeUserId;
const selectActiveUser = createSelector(
  selectUsers,
  selectActiveUserId,
  (users, activeUserId) => users.find(user => user.id === activeUserId)
);
```

Explanation: This pattern creates reusable and composable selectors. The `createSelector` function allows you to combine multiple selectors and memoize the result for performance. It's particularly useful for derived data that depends on multiple parts of the state.

## 5. Reusable Thunks

```javascript
const createFetchThunk = (endpoint, actionTypes) => () => async dispatch => {
  dispatch({ type: actionTypes.REQUEST });
  try {
    const response = await fetch(endpoint);
    const data = await response.json();
    dispatch({ type: actionTypes.SUCCESS, payload: data });
  } catch (error) {
    dispatch({ type: actionTypes.FAILURE, error });
  }
};

const fetchUsers = createFetchThunk('/api/users', {
  REQUEST: 'FETCH_USERS_REQUEST',
  SUCCESS: 'FETCH_USERS_SUCCESS',
  FAILURE: 'FETCH_USERS_FAILURE'
});
```

Explanation: This pattern creates a reusable thunk for fetching data. It encapsulates the common logic of making an API request, handling success and failure cases, and dispatching appropriate actions. This reduces duplication in async action creators and makes it easy to add new API endpoints with consistent behavior.

## 6. Reusable Middleware

```javascript
const createLogger = (getState = state => state) => store => next => action => {
  console.log('dispatching', action);
  const result = next(action);
  console.log('next state', getState(store.getState()));
  return result;
};

const logger = createLogger(state => state.user);
```

Explanation: This pattern creates a reusable middleware factory. In this example, it creates a customizable logger middleware. The `createLogger` function allows you to specify which part of the state to log, making it adaptable to different use cases while maintaining a consistent logging structure.

## 7. Reusable Store Enhancers

```javascript
const monitorReducerEnhancer = createStore => (
  reducer,
  initialState,
  enhancer
) => {
  const monitoredReducer = (state, action) => {
    const start = performance.now();
    const newState = reducer(state, action);
    const end = performance.now();
    const diff = round(end - start);
    console.log('reducer process time:', diff);
    return newState;
  };

  return createStore(monitoredReducer, initialState, enhancer);
};
```

Explanation: This pattern creates a reusable store enhancer. Store enhancers allow you to add functionality to the Redux store creation process. In this example, `monitorReducerEnhancer` adds performance monitoring to all reducers. This pattern is useful for adding global behaviors or instrumentation to your Redux store.

## 8. Reusable Action Type Creators

```javascript
const createRequestTypes = (base) => ({
  REQUEST: `${base}_REQUEST`,
  SUCCESS: `${base}_SUCCESS`,
  FAILURE: `${base}_FAILURE`,
});

const USER = createRequestTypes('USER');
// USER.REQUEST, USER.SUCCESS, USER.FAILURE
```

Explanation: This pattern creates a function that generates a set of related action types. It's particularly useful for async operations that typically have request, success, and failure states. This ensures consistency in naming conventions and reduces the chance of typos in action types.

## 9. Reusable Normalized State Structure

```javascript
import { normalize, schema } from 'normalizr';

const userSchema = new schema.Entity('users');
const normalizeUsers = (data) => normalize(data, [userSchema]);

// In reducer
case 'FETCH_USERS_SUCCESS':
  const normalizedData = normalizeUsers(action.payload);
  return {
    ...state,
    entities: {
      ...state.entities,
      users: {
        ...state.entities.users,
        ...normalizedData.entities.users
      }
    },
    ids: [...new Set([...state.ids, ...normalizedData.result])]
  };
```

Explanation: This pattern uses the normalizr library to create a reusable normalized state structure. Normalizing state makes it easier to update and retrieve data, especially for complex, nested data structures. This approach helps maintain a flat state structure, which is more efficient for updates and lookups in Redux.