## Core Concepts (Framework-Independent)

1. Store
   ```javascript
   import { createStore } from 'redux';
   const store = createStore(reducer);
   ```

2. Actions
   ```javascript
   const action = { type: 'INCREMENT', payload: 1 };
   ```

3. Reducers
   ```javascript
   function reducer(state = initialState, action) {
     switch (action.type) {
       case 'INCREMENT':
         return { ...state, count: state.count + action.payload };
       default:
         return state;
     }
   }
   ```

4. Dispatch
   ```javascript
   store.dispatch({ type: 'INCREMENT', payload: 1 });
   ```

5. Subscribe
   ```javascript
   const unsubscribe = store.subscribe(() => console.log(store.getState()));
   ```

## Framework-Agnostic Usage

### Vanilla JavaScript
```javascript
const store = createStore(reducer);

document.getElementById('incrementBtn').addEventListener('click', () => {
  store.dispatch({ type: 'INCREMENT' });
});

store.subscribe(() => {
  document.getElementById('count').textContent = store.getState().count;
});
```

### React (with react-redux)
```jsx
import { Provider, useSelector, useDispatch } from 'react-redux';

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}

function Counter() {
  const count = useSelector(state => state.count);
  const dispatch = useDispatch();
  return (
    <button onClick={() => dispatch({ type: 'INCREMENT' })}>
      Count: {count}
    </button>
  );
}
```

### Vue (basic integration)
```javascript
import { createStore } from 'redux';

const store = createStore(reducer);

new Vue({
  el: '#app',
  data: {
    state: store.getState()
  },
  created() {
    store.subscribe(() => {
      this.state = store.getState();
    });
  },
  methods: {
    increment() {
      store.dispatch({ type: 'INCREMENT' });
    }
  }
});
```

## Middleware (Framework-Independent)

```javascript
const logger = store => next => action => {
  console.log('dispatching', action);
  let result = next(action);
  console.log('next state', store.getState());
  return result;
};

const store = createStore(reducer, applyMiddleware(logger));
```

## Async Actions with Thunk (Framework-Independent)

```javascript
const thunkMiddleware = ({ dispatch, getState }) => next => action => {
  if (typeof action === 'function') {
    return action(dispatch, getState);
  }
  return next(action);
};

const fetchData = () => async dispatch => {
  dispatch({ type: 'FETCH_START' });
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    dispatch({ type: 'FETCH_SUCCESS', payload: data });
  } catch (error) {
    dispatch({ type: 'FETCH_ERROR', error });
  }
};
```

## Framework-Specific Bindings

1. React: react-redux
2. Vue: vuex (inspired by Redux)
3. Angular: @ngrx/store (inspired by Redux)

## Testing (Framework-Independent)

```javascript
import reducer from './reducer';

test('reducer increments count', () => {
  const initialState = { count: 0 };
  const action = { type: 'INCREMENT' };
  const nextState = reducer(initialState, action);
  expect(nextState.count).toBe(1);
});
```

## Key Principles of Redux's Agnostic Design

1. Single Source of Truth: The store is framework-independent.
2. State is Read-Only: Actions are plain objects, dispatched the same way regardless of framework.
3. Changes are Made with Pure Functions: Reducers are pure JavaScript functions.
4. Unidirectional Data Flow: This concept is not tied to any specific UI library.

## Benefits of Framework-Agnostic Design

1. Portability: Core logic can be moved between different framework implementations.
2. Testability: Business logic can be tested independently of UI.
3. Separation of Concerns: State management is decoupled from UI rendering.
4. Ecosystem Compatibility: Middleware and tools often work across different frameworks.

Remember, while Redux is framework-agnostic, it's most commonly used with React. The framework-agnostic design allows for flexibility and broad applicability, but framework-specific bindings (like react-redux) often provide the most ergonomic developer experience for a given framework.