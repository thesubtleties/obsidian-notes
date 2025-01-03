## 1. Installation

### Using npm:
```bash
npm install redux react-redux redux-thunk
```

### Using yarn:
```bash
yarn add redux react-redux redux-thunk
```

### What each package does:
- `redux`: The core Redux library
- `react-redux`: Official React bindings for Redux
- `redux-thunk`: Middleware for handling asynchronous actions

## 2. Folder Structure

Create the following folder structure in your `src` directory:

```
src/
├── actions/
├── reducers/
├── store/
├── components/
└── App.js
```

## 3. Create Your First Reducer

File: `src/reducers/counterReducer.js`

```javascript
const initialState = {
  count: 0
};

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
};

export default counterReducer;
```

## 4. Combine Reducers (Even if you only have one)

File: `src/reducers/index.js`

```javascript
import { combineReducers } from 'redux';
import counterReducer from './counterReducer';

const rootReducer = combineReducers({
  counter: counterReducer,
  // Add more reducers here as your app grows
});

export default rootReducer;
```

## 5. Create the Redux Store

File: `src/store/configureStore.js`

```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from '../reducers';

const configureStore = () => {
  return createStore(
    rootReducer,
    applyMiddleware(thunk)
  );
};

export default configureStore;
```

## 6. Integrate Redux with React

Update your main `App.js` or `index.js`:

File: `src/index.js`

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import configureStore from './store/configureStore';
import App from './App';

const store = configureStore();

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```

## 7. Create Your First Action Creator

File: `src/actions/counterActions.js`

```javascript
export const increment = () => ({
  type: 'INCREMENT'
});

export const decrement = () => ({
  type: 'DECREMENT'
});

// Thunk example
export const incrementAsync = () => {
  return dispatch => {
    setTimeout(() => {
      dispatch(increment());
    }, 1000);
  };
};
```

## 8. Use Redux in a React Component

File: `src/components/Counter.js`

```javascript
import React from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementAsync } from '../actions/counterActions';

const Counter = () => {
  const count = useSelector(state => state.counter.count);
  const dispatch = useDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
      <button onClick={() => dispatch(incrementAsync())}>Increment Async</button>
    </div>
  );
};

export default Counter;
```

## Key Points to Remember:

1. Always install `redux`, `react-redux`, and typically `redux-thunk` for async actions.
2. Create a clear folder structure to separate concerns (actions, reducers, store, components).
3. Start with a simple reducer and combine it using `combineReducers`, even if you only have one reducer initially.
4. Configure your store with middleware (like thunk) for handling async actions.
5. Wrap your root React component with the Redux `Provider` and pass it the store.
6. Create action creators for each action your app can dispatch.
7. Use `useSelector` to access Redux state and `useDispatch` to dispatch actions in functional components.
8. Remember that reducers must be pure functions and should not mutate state directly.

## Common Gotchas:

- Forgetting to wrap the app with `Provider`
- Mutating state in reducers instead of returning a new state object
- Not applying middleware when async actions are needed
- Overcomplicating the initial setup – start simple and add complexity as needed

This detailed setup provides a solid foundation for building React applications with Redux, allowing for easy state management and scalability as your application grows.