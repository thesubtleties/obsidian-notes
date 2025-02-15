# Redux Toolkit Complete Guide - Part 1: Store Structure & Design

## 1. Basic Store Configuration

```javascript
// store/index.js

import { configureStore } from '@reduxjs/toolkit';
import authReducer from './features/auth/authSlice';
import eventReducer from './features/events/eventSlice';
import sessionReducer from './features/sessions/sessionSlice';
import organizationReducer from './features/organizations/organizationSlice';
import userReducer from './features/users/userSlice';
import uiReducer from './features/ui/uiSlice';
import { api } from './services/api';

/*
  What: configureStore is RTK's store setup utility
  Why: Automatically sets up:
  - Redux DevTools
  - Middleware (including thunk)
  - Production/development checks
  - State serialization checks
*/
export const store = configureStore({
  reducer: {
    // Domain state - data from your backend
    auth: authReducer,
    events: eventReducer,
    sessions: sessionReducer,
    organizations: organizationReducer,
    users: userReducer,
    
    // UI state - local app state
    ui: uiReducer,
    
    // API state - RTK Query's cache
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});

/*
  Before (Traditional Redux):
  
  const store = createStore(
    combineReducers({
      auth: authReducer,
      events: eventReducer,
      // etc...
    }),
    applyMiddleware(
      thunk,
      logger,
      // manually add each middleware
    )
  );
*/
```

## 2. State Shape Design

```javascript
// Example of complete state shape with explanations

const exampleState = {
  // Auth State - User authentication and session
  auth: {
    user: null,        // Current user info
    isAuthenticated: false,
    status: 'idle',    // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null        // Authentication errors
  },

  // Events State - Using normalized structure
  events: {
    // Normalized entities - Why? Fast lookups by ID
    entities: {
      // Key is the ID, value is the event object
      '1': {
        id: '1',
        title: 'React Conference',
        description: 'Annual React Conference',
        startDate: '2024-06-01',
        // ... other event fields
      },
      '2': { /* another event */ }
    },
    
    // Array of IDs - Why? Maintain order
    ids: ['1', '2'],
    
    // Currently viewed/edited event
    activeEvent: null,
    
    // Request status tracking
    status: 'idle',
    error: null,
    
    // Local UI state specific to events
    filters: {
      dateRange: null,
      eventType: null,
      searchTerm: ''
    }
  },

  // Sessions State - Similar normalized structure
  sessions: {
    entities: {},
    ids: [],
    // Relationship mapping - Why? Quick access to sessions by event
    byEventId: {
      '1': ['session1', 'session2'], // Event 1's session IDs
    },
    status: 'idle',
    error: null
  },

  // Organizations State
  organizations: {
    entities: {},
    ids: [],
    activeOrganization: null,
    status: 'idle',
    error: null
  },

  // Users State
  users: {
    entities: {},
    ids: [],
    // Relationship mapping for speakers
    speakers: {
      'session1': ['user1', 'user2'] // Speakers for session 1
    },
    status: 'idle',
    error: null
  },

  // UI State - Separated from domain data
  ui: {
    currentView: 'events',
    modals: {
      createEvent: false,
      editSession: false
    },
    notifications: [],
    loading: {
      // Track loading state by operation
      'event-create': false,
      'session-update': false
    }
  }
};
```

## 3. Why This Structure?

### A. Normalized State
**What:** Store entities in an object with IDs as keys
**Why:**
- O(1) lookups instead of array searching
- No duplicate data
- Easier updates
- Better performance with large datasets

```javascript
// ❌ Bad (Nested Arrays)
const events = [
  {
    id: 1,
    title: 'Conference',
    sessions: [
      { id: 1, title: 'Session 1' },
      { id: 2, title: 'Session 2' }
    ]
  }
];

// ✅ Good (Normalized)
const state = {
  events: {
    entities: {
      '1': { id: 1, title: 'Conference' }
    },
    ids: ['1']
  },
  sessions: {
    entities: {
      '1': { id: 1, title: 'Session 1', eventId: 1 },
      '2': { id: 2, title: 'Session 2', eventId: 1 }
    },
    ids: ['1', '2'],
    byEventId: {
      '1': ['1', '2']
    }
  }
};
```

### B. Status Management
**What:** Track loading, error, and request states
**Why:**
- Consistent loading indicators
- Error handling
- Prevent duplicate requests
- Support optimistic updates

```javascript
// Example usage in component
function EventList() {
  const status = useSelector(state => state.events.status);
  const error = useSelector(state => state.events.error);

  if (status === 'loading') {
    return <LoadingSpinner />;
  }

  if (status === 'failed') {
    return <ErrorMessage error={error} />;
  }

  // Render events...
}
```

