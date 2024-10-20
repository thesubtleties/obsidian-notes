## Basic Thunk Structure

```javascript
const basicThunk = (arg1, arg2) => (dispatch, getState) => {
  // Thunk logic here
};
```
This is the basic structure of a thunk. It's a function that returns another function, which receives `dispatch` and `getState` as arguments. This allows you to perform complex logic and dispatch multiple actions if needed.

## Async Thunk Structure

```javascript
const asyncThunk = (arg1, arg2) => async (dispatch, getState) => {
  // Async thunk logic here
};
```
For asynchronous operations, we use async/await syntax. This makes it easier to handle promises and perform sequential asynchronous operations.

## Using getState in Thunks

```javascript
const thunkWithGetState = () => (dispatch, getState) => {
  const currentState = getState();
  console.log('Current state:', currentState);
  // Use currentState in your logic
};
```
`getState()` allows you to access the current Redux state within your thunk. This is useful when you need to make decisions based on the current state or use current state values in your logic.

## Dispatching Actions in Thunks

```javascript
const dispatchingThunk = () => (dispatch) => {
  dispatch({ type: 'SOME_ACTION', payload: 'data' });
};
```
Thunks can dispatch actions using the `dispatch` function. This allows you to update the Redux state as part of your thunk's logic.

## Combining Thunks and Selectors

```javascript
// Selector
const selectUserData = (state, userId) => state.users[userId];

// Thunk using selector
const fetchAndSelectUserThunk = (userId) => async (dispatch, getState) => {
  await dispatch(fetchUserDataThunk(userId));
  return selectUserData(getState(), userId);
};
```
This pattern combines a thunk with a selector. It first dispatches another thunk to fetch data, then uses a selector to extract specific data from the updated state. This is powerful for fetching and then immediately selecting relevant data.

## Chaining Thunks

```javascript
const chainedThunk = () => async (dispatch) => {
  await dispatch(thunk1());
  await dispatch(thunk2());
  return dispatch(thunk3());
};
```
Thunks can be chained to perform a series of asynchronous operations. This is useful for complex workflows where multiple actions need to be dispatched in a specific order.

## Error Handling in Thunks

```javascript
const errorHandlingThunk = () => async (dispatch) => {
  try {
    const response = await apiCall();
    dispatch(successAction(response));
  } catch (error) {
    dispatch(errorAction(error));
  }
};
```
It's important to handle errors in thunks, especially for async operations. This pattern allows you to dispatch different actions based on the success or failure of an operation.

## Conditional Dispatching

```javascript
const conditionalThunk = () => (dispatch, getState) => {
  const state = getState();
  if (state.someCondition) {
    dispatch(actionA());
  } else {
    dispatch(actionB());
  }
};
```
Thunks can use the current state to make decisions about which actions to dispatch. This is useful for implementing complex business logic that depends on the current application state.

## Thunks with Parameters

```javascript
const parameterizedThunk = (id, data) => async (dispatch) => {
  const response = await apiCall(id, data);
  dispatch(successAction(response));
};
```
Thunks can accept parameters, allowing you to create more flexible and reusable thunks. This is commonly used for API calls where you need to pass specific data.

## Using Thunks in Components

```javascript
import { useDispatch, useSelector } from 'react-redux';

function MyComponent() {
  const dispatch = useDispatch();
  const data = useSelector(selectData);

  useEffect(() => {
    dispatch(myThunk());
  }, [dispatch]);

  // Component logic
}
```
This shows how to use thunks in a React component. The `useDispatch` hook is used to get the dispatch function, and `useEffect` is used to dispatch the thunk when the component mounts.

## Advanced: Thunk with Multiple Selectors

```javascript
const complexSelector = createSelector(
  [selectA, selectB],
  (a, b) => ({ combined: a + b })
);

const advancedThunk = () => async (dispatch, getState) => {
  await dispatch(fetchDataThunk());
  const result = complexSelector(getState());
  return result;
};
```
This advanced pattern uses a memoized selector created with `createSelector` from the Reselect library. The thunk fetches data and then uses this complex selector to compute derived data from the updated state. This is efficient for complex state derivations.