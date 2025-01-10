## API Setup

```javascript
// api/axios.js
import axios from 'axios';

// Create base axios instance with common configuration
export const api = axios.create({
  baseURL: import.meta.env.VITE_API_URL || 'http://localhost:5000/api',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add auth interceptor for JWT
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```
This creates a reusable axios instance with common configuration. The interceptor automatically adds authentication tokens to requests, keeping your auth logic DRY.

## Slice Setup

```javascript
// store/eventSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { api } from '../api/axios';

// Async thunks - handle API calls
export const fetchEvents = createAsyncThunk(
  'events/fetchEvents',
  async (_, { rejectWithValue }) => {
    try {
      const response = await api.get('/events');
      return response.data;  // Schema-formatted data
    } catch (error) {
      return rejectWithValue(error.response.data);
    }
  }
);

export const createEvent = createAsyncThunk(
  'events/createEvent',
  async (eventData, { rejectWithValue }) => {
    try {
      const response = await api.post('/events', eventData);
      return response.data;
    } catch (error) {
      // Handle schema validation errors
      return rejectWithValue(error.response.data.errors);
    }
  }
);

// Slice definition
const eventSlice = createSlice({
  name: 'events',
  initialState: {
    items: [],
    status: 'idle', // idle | loading | succeeded | failed
    error: null,
    currentEvent: null
  },
  reducers: {
    clearErrors: (state) => {
      state.error = null;
    }
  },
  extraReducers: (builder) => {
    builder
      // Fetch events cases
      .addCase(fetchEvents.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchEvents.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchEvents.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload;
      })
      // Create event cases
      .addCase(createEvent.fulfilled, (state, action) => {
        state.items.push(action.payload);
      });
  }
});
```
This slice setup handles async operations with proper loading states and error handling. The thunks use axios for API calls, and the slice manages the state updates.

## Component Integration

```javascript
// components/EventList.js
import { useEffect } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { fetchEvents } from '../store/eventSlice';

function EventList() {
  const dispatch = useDispatch();
  const {
    items: events,
    status,
    error
  } = useSelector(state => state.events);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchEvents());
    }
  }, [status, dispatch]);

  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'failed') return <div>Error: {error}</div>;

  return (
    <div>
      {events.map(event => (
        <div key={event.id}>{event.title}</div>
      ))}
    </div>
  );
}
```
Components dispatch thunks and access state through useSelector. The component handles different states (loading, error, success) appropriately.

## Form Handling with Thunks

```javascript
// components/EventForm.js
import { useState } from 'react';
import { useDispatch, useSelector } from 'react-redux';
import { createEvent } from '../store/eventSlice';

function EventForm() {
  const dispatch = useDispatch();
  const [formData, setFormData] = useState({
    title: '',
    description: '',
    start_date: ''
  });
  
  const { error } = useSelector(state => state.events);

  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      // Dispatch returns a promise
      const resultAction = await dispatch(createEvent(formData));
      if (createEvent.fulfilled.match(resultAction)) {
        // Success! Schema validated and saved
        console.log('Event created:', resultAction.payload);
      }
    } catch (err) {
      // Handle any errors that weren't handled in the thunk
      console.error('Failed to create event:', err);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={formData.title}
        onChange={(e) => setFormData({
          ...formData,
          title: e.target.value
        })}
      />
      {error?.title && <span>{error.title}</span>}
      {/* More fields */}
    </form>
  );
}
```
Forms dispatch thunks and handle both success and error cases. Schema validation errors from the backend are displayed in the UI.

## Error Handling Patterns

```javascript
// store/errorUtils.js
export const createAsyncThunkWithError = (typePrefix, payloadCreator) =>
  createAsyncThunk(
    typePrefix,
    async (arg, thunkAPI) => {
      try {
        const response = await payloadCreator(arg);
        return response.data;
      } catch (error) {
        if (error.response) {
          // Schema validation errors
          if (error.response.status === 400) {
            return thunkAPI.rejectWithValue(error.response.data.errors);
          }
          // Auth errors
          if (error.response.status === 401) {
            // Handle unauthorized
          }
        }
        throw error;
      }
    }
  );

// Usage
export const updateEvent = createAsyncThunkWithError(
  'events/updateEvent',
  (eventData) => api.put(`/events/${eventData.id}`, eventData)
);
```
This pattern provides consistent error handling across thunks, handling both schema validation and other API errors.

## Loading States and Selectors

```javascript
// store/selectors.js
export const selectEventById = (state, eventId) =>
  state.events.items.find(event => event.id === eventId);

export const selectEventsByStatus = (state, status) =>
  state.events.items.filter(event => event.status === status);

// Component usage
function EventDetail({ id }) {
  const event = useSelector(state => selectEventById(state, id));
  const status = useSelector(state => state.events.status);
  
  if (status === 'loading') return <Spinner />;
  if (!event) return <NotFound />;
  
  return <div>{event.title}</div>;
}
```
Selectors help access and filter state data, while components handle different loading states appropriately.

Remember:
- Thunks handle async logic and API calls
- Axios makes the actual HTTP requests
- Schema validates and formats data on backend
- Slice manages state updates
- Components dispatch and consume state

This pattern provides:
- Clean separation of concerns
- Consistent error handling
- Type-safe API interactions
- Predictable state updates
- Reusable API logic

