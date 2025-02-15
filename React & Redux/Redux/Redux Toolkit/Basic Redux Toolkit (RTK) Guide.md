## 1. Basic Setup

### Store Configuration
```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './features/counterSlice';
import userReducer from './features/userSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    user: userReducer,
  },
});
```
**What:** Sets up your Redux store with all reducers
**Before (Redux):** Required manual store creation, middleware setup, and DevTools configuration (15+ lines)
**Why RTK:** One function handles store creation, DevTools, thunk middleware, and development checks
**Key Benefit:** Reduces boilerplate by ~70% and prevents common setup mistakes

### Provider Setup
```javascript
// index.js
import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
);
```
**What:** Makes Redux store available to React components
**Note:** This part is identical to regular Redux - RTK doesn't change the Provider pattern

## 2. Creating and Using Slices

### Basic Slice
```javascript
// features/counterSlice.js
import { createSlice } from '@reduxjs/toolkit';

const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0,
    status: 'idle'
  },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
  },
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;
```
**What:** Creates a slice containing state and reducers for one feature
**Before (Redux):** Required separate action creators, action types, and reducer with switch statement (40+ lines)
**Why RTK:** Combines actions and reducers in one place, auto-generates action creators
**Key Benefit:** Eliminates action type constants and reduces code by ~60%

### Using the Slice in Components
```javascript
// components/Counter.js
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount } from './counterSlice';

function Counter() {
  // Select specific state to avoid unnecessary rerenders
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>Add 5</button>
    </div>
  );
}
```
**What:** Connects React components to Redux store
**Before (Redux):** Required mapStateToProps, mapDispatchToProps, and connect HOC
**Why RTK:** Hooks provide simpler, more intuitive access to store
**Key Benefit:** More readable code and better TypeScript support

## 3. Handling Mutations (The Right Way)

### Immutable Updates with RTK
```javascript
// features/todosSlice.js
import { createSlice } from '@reduxjs/toolkit';

const todosSlice = createSlice({
  name: 'todos',
  initialState: [],
  reducers: {
    // These "mutations" are safe inside createSlice
    addTodo: (state, action) => {
      state.push(action.payload);
    },
    updateTodo: (state, action) => {
      const todo = state.find(todo => todo.id === action.payload.id);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    removeTodo: (state, action) => {
      const index = state.findIndex(todo => todo.id === action.payload);
      state.splice(index, 1);
    }
  }
});
```
**What:** Write simpler reducer logic that appears to mutate state
**Before (Redux):**
```javascript
// Old Redux way - explicit immutability
const todoReducer = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    case 'UPDATE_TODO':
      return state.map(todo => 
        todo.id === action.payload.id 
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.payload);
    default:
      return state;
  }
};
```
**Why RTK:** Immer handles immutability behind the scenes
**Key Benefit:** More intuitive code that's less prone to mistakes

## 4. Async Operations with createAsyncThunk

### Basic Async Thunk
```javascript
// features/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUserById = createAsyncThunk(
  'users/fetchById',
  async (userId, thunkAPI) => {
    try {
      const response = await fetch(`https://api.example.com/users/${userId}`);
      return await response.json();
    } catch (error) {
      return thunkAPI.rejectWithValue(error.message);
    }
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState: {
    data: null,
    status: 'idle', // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUserById.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUserById.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchUserById.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload || action.error.message;
      });
  },
});
```
**What:** Handles async operations with automatic loading, success, and error states
**Before (Redux):** Required manual action creators for each state, thunk middleware setup, and complex reducer logic
**Why RTK:** Automatically generates pending/fulfilled/rejected actions and provides type safety
**Key Benefit:** Standardized approach to async operations with much less code

### Using Async Thunk in Components
```javascript
// components/UserProfile.js
import { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchUserById } from './userSlice';

function UserProfile({ userId }) {
  const dispatch = useDispatch();
  const { data: user, status, error } = useSelector((state) => state.user);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchUserById(userId));
    }
  }, [userId, status, dispatch]);

  if (status === 'loading') return <div>Loading...</div>;
  if (status === 'failed') return <div>Error: {error}</div>;
  if (!user) return null;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```
**What:** Implements loading, error, and success states for async operations
**Before (Redux):** Required manual tracking of loading states and error handling
**Why RTK:** Standardized pattern for handling async state
**Key Benefit:** Consistent, predictable async state management

# Redux Toolkit Cheatsheet (Continued)

## 5. RTK Query

### Basic API Setup
```javascript
// services/api.js
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: 'https://api.example.com' }),
  tagTypes: ['Post', 'User'],
  endpoints: (builder) => ({
    getPosts: builder.query({
      query: () => 'posts',
      providesTags: ['Post'],
    }),
    getPost: builder.query({
      query: (id) => `posts/${id}`,
      providesTags: (result, error, id) => [{ type: 'Post', id }],
    }),
    addPost: builder.mutation({
      query: (post) => ({
        url: 'posts',
        method: 'POST',
        body: post,
      }),
      invalidatesTags: ['Post'],
    }),
  }),
});

export const { 
  useGetPostsQuery, 
  useGetPostQuery, 
  useAddPostMutation 
} = api;
```
**What:** Creates a complete API client with caching, polling, and cache invalidation
**Before (Redux):** Required manual thunks, loading states, and cache management for each endpoint
**Why RTK Query:** Eliminates need for manual data fetching and caching logic
**Key Benefit:** Automatic cache management and real-time updates

### Using RTK Query in Components
```javascript
// components/Posts.js
import { useGetPostsQuery, useAddPostMutation } from '../services/api';

function Posts() {
  const { data: posts, isLoading, error } = useGetPostsQuery();
  const [addPost, { isLoading: isAdding }] = useAddPostMutation();

  if (isLoading) return <div>Loading posts...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <button 
        onClick={() => addPost({ title: 'New Post', body: 'Content' })}
        disabled={isAdding}
      >
        {isAdding ? 'Adding...' : 'Add Post'}
      </button>
      
      {posts?.map(post => (
        <div key={post.id}>
          <h2>{post.title}</h2>
          <p>{post.body}</p>
        </div>
      ))}
    </div>
  );
}
```
**What:** Implements data fetching with automatic loading, error states, and cache updates
**Before (Redux):** Required useEffect, dispatch calls, and manual state management
**Why RTK Query:** Handles all API state management automatically
**Key Benefit:** Dramatically reduces boilerplate and prevents common data-fetching bugs

## 6. Advanced Slice Patterns

### Normalized State
```javascript
// features/postsSlice.js
import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

const postsAdapter = createEntityAdapter({
  // Assume posts have `id` field
  sortComparer: (a, b) => b.date.localeCompare(a.date),
});

const postsSlice = createSlice({
  name: 'posts',
  initialState: postsAdapter.getInitialState({
    loading: 'idle',
    error: null,
  }),
  reducers: {
    // Built-in CRUD operations
    addOne: postsAdapter.addOne,
    addMany: postsAdapter.addMany,
    updateOne: postsAdapter.updateOne,
    removeOne: postsAdapter.removeOne,
    
    // Custom reducers
    setAllPosts: (state, action) => {
      postsAdapter.setAll(state, action.payload);
      state.loading = 'succeeded';
    },
  },
});

// Selectors
export const {
  selectAll: selectAllPosts,
  selectById: selectPostById,
  selectIds: selectPostIds,
} = postsAdapter.getSelectors((state) => state.posts);
```
**What:** Manages normalized state with efficient CRUD operations
**Before (Redux):** Required manual normalization and denormalization
**Why RTK:** Provides optimized operations for normalized data
**Key Benefit:** Better performance with large datasets

### Using Normalized State
```javascript
// components/PostsList.js
import { useSelector } from 'react-redux';
import { selectAllPosts, selectPostIds } from './postsSlice';

function PostsList() {
  // Get sorted IDs array
  const postIds = useSelector(selectPostIds);
  
  return (
    <div>
      {postIds.map(id => (
        <PostListItem key={id} id={id} />
      ))}
    </div>
  );
}

// Efficient single post component
function PostListItem({ id }) {
  // Only re-renders when this specific post changes
  const post = useSelector(state => selectPostById(state, id));
  
  return (
    <div>
      <h3>{post.title}</h3>
      <p>{post.content}</p>
    </div>
  );
}
```
**What:** Efficiently renders lists of items from normalized state
**Before (Redux):** Required manual optimization and memoization
**Why RTK:** Provides optimized selectors and update patterns
**Key Benefit:** Better performance and simpler code

## 7. Performance Optimization

### Memoized Selectors
```javascript
// features/selectors.js
import { createSelector } from '@reduxjs/toolkit';

// Basic selector
const selectPosts = state => state.posts.entities;
const selectSearchTerm = state => state.filters.searchTerm;
const selectCategory = state => state.filters.category;

// Complex memoized selector
export const selectFilteredPosts = createSelector(
  [selectPosts, selectSearchTerm, selectCategory],
  (posts, searchTerm, category) => {
    return Object.values(posts)
      .filter(post => 
        post.title.toLowerCase().includes(searchTerm.toLowerCase()) &&
        (category === 'all' || post.category === category)
      )
      .sort((a, b) => b.date.localeCompare(a.date));
  }
);
```
**What:** Creates memoized selectors for complex state derivations
**Before (Redux):** Required manual memoization or recomputation on every render
**Why RTK:** Built-in memoization with createSelector
**Key Benefit:** Better performance for expensive computations

### Optimized Component Updates
```javascript
// components/PostList.js
import { useSelector, shallowEqual } from 'react-redux';

function PostList() {
  // Only re-render if array references change
  const posts = useSelector(selectFilteredPosts, shallowEqual);
  
  // Memoize expensive calculations
  const statistics = useMemo(() => ({
    total: posts.length,
    published: posts.filter(p => p.published).length,
    draft: posts.filter(p => !p.published).length,
  }), [posts]);

  return (
    <div>
      <PostStatistics stats={statistics} />
      {posts.map(post => (
        <PostItem 
          key={post.id}
          post={post}
        />
      ))}
    </div>
  );
}

// Memoized child component
const PostItem = memo(function PostItem({ post }) {
  return (
    <div>
      <h3>{post.title}</h3>
      <p>{post.excerpt}</p>
    </div>
  );
});
```
**What:** Implements performance optimizations for React components
**Before (Redux):** Required complex manual optimizations
**Why RTK:** Works well with React's optimization features
**Key Benefit:** Prevents unnecessary rerenders

# Redux Toolkit Cheatsheet (Continued)

## 8. Error Handling Patterns

### Global Error Handling
```javascript
// features/errorSlice.js
import { createSlice } from '@reduxjs/toolkit';

const errorSlice = createSlice({
  name: 'error',
  initialState: {
    errors: [],
    lastError: null,
  },
  reducers: {
    addError: (state, action) => {
      state.errors.push({
        id: Date.now(),
        message: action.payload.message,
        type: action.payload.type,
        timestamp: new Date().toISOString(),
      });
      state.lastError = action.payload;
    },
    clearError: (state, action) => {
      state.errors = state.errors.filter(error => error.id !== action.payload);
    },
    clearAllErrors: (state) => {
      state.errors = [];
      state.lastError = null;
    },
  },
});

export const { addError, clearError, clearAllErrors } = errorSlice.actions;
```
**What:** Centralizes error handling across the application
**Why:** Provides consistent error tracking and display
**Key Benefit:** Easier error management and debugging

### Error Middleware
```javascript
// middleware/errorMiddleware.js
import { isRejectedWithValue } from '@reduxjs/toolkit';
import { addError } from '../features/errorSlice';

export const errorMiddleware = (store) => (next) => (action) => {
  // Handle async thunk rejections
  if (isRejectedWithValue(action)) {
    store.dispatch(addError({
      message: action.payload,
      type: 'API_ERROR',
    }));
    
    // Optional: Log to error tracking service
    logErrorToService({
      error: action.payload,
      action: action.type,
      state: store.getState(),
    });
  }

  // Handle specific error actions
  if (action.type?.endsWith('/error')) {
    store.dispatch(addError({
      message: action.payload,
      type: 'APP_ERROR',
    }));
  }

  return next(action);
};

// Usage in store configuration
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(errorMiddleware)
});
```
**What:** Intercepts and processes errors across the application
**Before (Redux):** Required manual error handling in each thunk/reducer
**Why RTK:** Centralizes error handling logic
**Key Benefit:** Consistent error processing and logging

### Error Boundary Integration
```javascript
// components/ErrorBoundary.js
import { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { clearError } from '../features/errorSlice';

function ErrorDisplay() {
  const dispatch = useDispatch();
  const { errors, lastError } = useSelector(state => state.error);

  useEffect(() => {
    if (lastError) {
      // Auto-clear non-critical errors after 5 seconds
      const timer = setTimeout(() => {
        dispatch(clearError(lastError.id));
      }, 5000);
      
      return () => clearTimeout(timer);
    }
  }, [lastError, dispatch]);

  if (!errors.length) return null;

  return (
    <div className="error-container">
      {errors.map(error => (
        <div key={error.id} className={`error-alert ${error.type}`}>
          <p>{error.message}</p>
          <button onClick={() => dispatch(clearError(error.id))}>
            Dismiss
          </button>
        </div>
      ))}
    </div>
  );
}
```
**What:** Displays and manages errors in the UI
**Why:** Provides user feedback for errors
**Key Benefit:** Better user experience with error handling

## 9. Advanced Middleware Configuration

### Custom Middleware Setup
```javascript
// store/middleware.js
import { createListenerMiddleware, isAnyOf } from '@reduxjs/toolkit';
import { persistState } from '../utils/localStorage';

// Listener Middleware for Side Effects
export const listenerMiddleware = createListenerMiddleware();

// Add listeners for specific actions
listenerMiddleware.startListening({
  matcher: isAnyOf(
    action => action.type.endsWith('/fulfilled'),
    action => action.type.endsWith('/rejected')
  ),
  effect: async (action, listenerApi) => {
    // Log all fulfilled/rejected actions
    console.log(`Action completed: ${action.type}`, action);
  }
});

// Custom Analytics Middleware
export const analyticsMiddleware = store => next => action => {
  // Track specific actions
  if (action.type?.startsWith('user/')) {
    trackUserAction(action);
  }
  return next(action);
};

// State Persistence Middleware
export const persistenceMiddleware = store => next => action => {
  const result = next(action);
  const state = store.getState();
  
  // Persist specific slices of state
  persistState({
    user: state.user,
    settings: state.settings
  });
  
  return result;
};

// Store Configuration
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // Ignore specific action types
        ignoredActions: ['persist/PERSIST'],
        // Ignore specific paths in state
        ignoredPaths: ['entities.files'],
      },
      thunk: {
        extraArgument: {
          api: apiClient,
          logger: customLogger,
        },
      },
    })
      .prepend(listenerMiddleware.middleware)
      .concat([
        analyticsMiddleware,
        persistenceMiddleware,
        // Add conditional middleware
        ...(process.env.NODE_ENV === 'development' ? [debugMiddleware] : []),
      ])
});
```
**What:** Configures custom middleware for specific needs
**Before (Redux):** Required complex middleware setup and manual configuration
**Why RTK:** Provides powerful middleware customization options
**Key Benefit:** Better control over side effects and state management

### Advanced Middleware Patterns
```javascript
// middleware/batchActionsMiddleware.js
import { createAction } from '@reduxjs/toolkit';

// Action batching
export const batchActions = createAction('BATCH_ACTIONS');

export const batchMiddleware = store => next => action => {
  if (action.type === batchActions.type) {
    action.payload.forEach(batchedAction => {
      store.dispatch(batchedAction);
    });
    return;
  }
  return next(action);
};

// Usage example
dispatch(batchActions([
  increment(),
  updateUser({ name: 'John' }),
  fetchData()
]));

// Conditional Middleware
export const conditionalMiddleware = (condition) => store => next => action => {
  if (!condition(store.getState())) {
    return next(action);
  }
  
  // Custom handling when condition is met
  console.log('Condition met for action:', action);
  return next(action);
};

// Usage in store
const store = configureStore({
  reducer: rootReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware()
      .concat(
        conditionalMiddleware(
          state => state.user.isAdmin
        )
      )
});
```
**What:** Implements advanced middleware patterns for specific use cases
**Why:** Handles complex scenarios like action batching and conditional processing
**Key Benefit:** More flexible and powerful state management

