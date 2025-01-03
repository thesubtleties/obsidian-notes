## 1. Action Types

```javascript
const FETCH_DATA_REQUEST = 'FETCH_DATA_REQUEST';
const FETCH_DATA_SUCCESS = 'FETCH_DATA_SUCCESS';
const FETCH_DATA_FAILURE = 'FETCH_DATA_FAILURE';
```

Concept: Define separate action types for each stage of the async operation.

## 2. Action Creators

```javascript
const fetchDataRequest = () => ({ type: FETCH_DATA_REQUEST });
const fetchDataSuccess = (data) => ({ type: FETCH_DATA_SUCCESS, payload: data });
const fetchDataFailure = (error) => ({ type: FETCH_DATA_FAILURE, payload: error });

// Thunk action creator
const fetchData = () => {
  return async (dispatch) => {
    dispatch(fetchDataRequest());
    try {
      const response = await api.fetchData();
      dispatch(fetchDataSuccess(response.data));
    } catch (error) {
      dispatch(fetchDataFailure(error.message));
    }
  };
};
```

Concept: Create action creators for each stage and a thunk for the entire async operation.

## 3. Reducer

```javascript
const initialState = {
  data: null,
  loading: false,
  error: null
};

const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true, error: null };
    case FETCH_DATA_SUCCESS:
      return { ...state, loading: false, data: action.payload };
    case FETCH_DATA_FAILURE:
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};
```

Concept: Update the state to reflect each stage of the async operation.

## 4. Using in Components

```javascript
import React, { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchData } from './actions';

const DataComponent = () => {
  const dispatch = useDispatch();
  const { data, loading, error } = useSelector(state => state.data);

  useEffect(() => {
    dispatch(fetchData());
  }, [dispatch]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data) return null;

  return <div>{/* Render data */}</div>;
};
```

Concept: Dispatch the async action and handle different states in the component.

## 5. Cancelling Requests

```javascript
let cancelRequest = false;

const fetchDataSafe = () => async (dispatch) => {
  dispatch(fetchDataRequest());
  try {
    const response = await api.fetchData();
    if (!cancelRequest) {
      dispatch(fetchDataSuccess(response.data));
    }
  } catch (error) {
    if (!cancelRequest) {
      dispatch(fetchDataFailure(error.message));
    }
  }
};

// In component:
useEffect(() => {
  dispatch(fetchDataSafe());
  return () => { cancelRequest = true; };
}, [dispatch]);
```

Concept: Implement a cancellation mechanism to handle component unmounting during async operations.

## 6. Handling Multiple Requests

```javascript
const dataReducer = (state = initialState, action) => {
  switch (action.type) {
    case FETCH_DATA_REQUEST:
      return { ...state, loading: true, error: null };
    case FETCH_DATA_SUCCESS:
      return {
        ...state,
        loading: false,
        data: [...state.data, ...action.payload],
        page: state.page + 1
      };
    // ... other cases
  }
};
```

Concept: Adapt the reducer to handle pagination or multiple data fetches.

## 7. Debouncing Requests

```javascript
import debounce from 'lodash/debounce';

const debouncedFetchData = debounce((dispatch) => {
  dispatch(fetchData());
}, 300);

// In component:
const handleSearch = (query) => {
  debouncedFetchData(dispatch);
};
```

Concept: Use debouncing to prevent excessive API calls, especially for search inputs.

Key Points:
- Use separate action types for request, success, and failure states.
- Implement loading and error states in your reducer.
- Use thunks or middleware for async operations.
- Handle different states (loading, error, success) in your components.
- Consider cancellation for long-running requests.
- Adapt your reducer for pagination or multiple requests if needed.
- Use techniques like debouncing to optimize API calls.

Remember: Proper handling of async states improves user experience and helps manage complex data flows in your application.