## Accessing State with useSelector

```javascript
import { useSelector } from 'react-redux';

function MyComponent() {
  const user = useSelector(state => state.user);
  // ...
}
```
This hook allows you to extract data from the Redux store state.

## Dispatching Actions with useDispatch

```javascript
import { useDispatch } from 'react-redux';

function MyComponent() {
  const dispatch = useDispatch();
  
  const handleClick = () => {
    dispatch({ type: 'SOME_ACTION' });
  };
  // ...
}
```
This hook returns a reference to the dispatch function from the Redux store.

## Combining useSelector and useDispatch

```javascript
function MyComponent() {
  const count = useSelector(state => state.count);
  const dispatch = useDispatch();

  return (
    <button onClick={() => dispatch({ type: 'INCREMENT' })}>
      Count: {count}
    </button>
  );
}
```
This pattern allows you to both read state and dispatch actions in a component.

## Using Selectors for Derived Data

```javascript
import { createSelector } from 'reselect';

const selectVisibleTodos = createSelector(
  state => state.todos,
  state => state.visibilityFilter,
  (todos, filter) => todos.filter(todo => /* ... */)
);

function TodoList() {
  const visibleTodos = useSelector(selectVisibleTodos);
  // ...
}
```
Selectors can compute derived data, allowing Redux to store the minimal possible state.

## Dispatching Thunks for Async Actions

```javascript
function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const user = useSelector(state => state.users[userId]);

  useEffect(() => {
    dispatch(fetchUserById(userId));
  }, [dispatch, userId]);

  // ...
}
```
Thunks allow you to dispatch async actions, like API calls.

## Accessing Multiple State Slices

```javascript
function Dashboard() {
  const { user, posts, comments } = useSelector(state => ({
    user: state.user,
    posts: state.posts,
    comments: state.comments
  }));
  // ...
}
```
You can select multiple slices of state in a single useSelector call.

## Optimizing with Equality Checks

```javascript
import { shallowEqual } from 'react-redux';

function MyComponent() {
  const { a, b } = useSelector(state => ({
    a: state.a,
    b: state.b
  }), shallowEqual);
  // ...
}
```
Using shallowEqual can prevent unnecessary re-renders when selecting object state.

## Custom Hooks for Reusable Redux Logic

```javascript
function useUser(userId) {
  const dispatch = useDispatch();
  const user = useSelector(state => state.users[userId]);

  useEffect(() => {
    if (!user) dispatch(fetchUserById(userId));
  }, [dispatch, user, userId]);

  return user;
}

// Usage
function UserProfile({ userId }) {
  const user = useUser(userId);
  // ...
}
```
Custom hooks can encapsulate Redux logic for reuse across components.

## Conditional Rendering Based on State

```javascript
function MyComponent() {
  const { isLoading, data, error } = useSelector(state => state.myData);

  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return null;

  return <DataDisplay data={data} />;
}
```
Use Redux state to conditionally render different component states.

## Connecting to Store with Provider

```javascript
import { Provider } from 'react-redux';
import store from './store';

function App() {
  return (
    <Provider store={store}>
      {/* Your app components */}
    </Provider>
  );
}
```
Wrap your root component with Provider to make the store available to all components.

Remember:
- Use `useSelector` for reading state, `useDispatch` for dispatching actions.
- Keep your components focused on rendering and handling user interactions.
- Use selectors for computing derived data.
- Leverage custom hooks to reuse Redux logic across components.
- Be mindful of performance - use memoization techniques when necessary.

