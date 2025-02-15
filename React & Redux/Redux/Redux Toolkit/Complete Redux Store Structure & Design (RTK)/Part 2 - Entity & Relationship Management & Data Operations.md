# Redux Toolkit Complete Guide - Part 2: Entity Management & Data Operations

## 1. Entity Adapters
**What:** Built-in utilities for managing normalized state of a specific data type
**Why:** Simplifies CRUD operations and provides standardized selectors

```javascript
// features/events/eventSlice.js

import { 
  createEntityAdapter, 
  createSlice, 
  createSelector 
} from '@reduxjs/toolkit';

// 1. Create Entity Adapter
const eventsAdapter = createEntityAdapter({
  // Custom ID selector if your entities don't use 'id'
  selectId: (event) => event.id,
  
  // Optional: Sort entities (will affect selectAll selector)
  sortComparer: (a, b) => {
    return new Date(b.startDate) - new Date(a.startDate);
  }
});

// 2. Initialize State with Adapter
/*
  The adapter automatically creates:
  {
    ids: [],          // Array of entity IDs
    entities: {},     // Normalized entities object
  }
  We can add additional fields to this base state
*/
const initialState = eventsAdapter.getInitialState({
  activeEventId: null,
  status: 'idle',
  error: null,
  filters: {
    dateRange: null,
    searchTerm: ''
  }
});

// 3. Create Slice with Adapter Methods
const eventSlice = createSlice({
  name: 'events',
  initialState,
  reducers: {
    // Standard CRUD operations using adapter methods
    addEvent: eventsAdapter.addOne,        // Add single entity
    addEvents: eventsAdapter.addMany,      // Add multiple entities
    updateEvent: eventsAdapter.updateOne,   // Update single entity
    updateEvents: eventsAdapter.updateMany, // Update multiple entities
    removeEvent: eventsAdapter.removeOne,   // Remove single entity
    
    // Custom reducers for additional state
    setActiveEvent: (state, action) => {
      state.activeEventId = action.payload;
    },
    
    setEventFilters: (state, action) => {
      state.filters = {
        ...state.filters,
        ...action.payload
      };
    }
  },
  
  // Handle async operations
  extraReducers: (builder) => {
    builder
      .addCase(fetchEvents.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchEvents.fulfilled, (state, action) => {
        state.status = 'succeeded';
        // Use adapter method to set all entities
        eventsAdapter.setAll(state, action.payload);
      })
      .addCase(fetchEvents.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.error.message;
      });
  }
});

// 4. Get Built-in Selectors
const {
  selectAll: selectAllEvents,    // Get all entities as array
  selectById: selectEventById,   // Get entity by ID
  selectIds: selectEventIds,     // Get array of all IDs
  selectTotal: selectEventCount  // Get total count of entities
} = eventsAdapter.getSelectors(state => state.events);

// 5. Create Custom Selectors
export const selectFilteredEvents = createSelector(
  [selectAllEvents, state => state.events.filters],
  (events, filters) => {
    return events.filter(event => {
      // Filter by search term
      if (filters.searchTerm) {
        const searchLower = filters.searchTerm.toLowerCase();
        if (!event.title.toLowerCase().includes(searchLower)) {
          return false;
        }
      }
      
      // Filter by date range
      if (filters.dateRange) {
        const eventDate = new Date(event.startDate);
        if (eventDate < filters.dateRange.start || 
            eventDate > filters.dateRange.end) {
          return false;
        }
      }
      
      return true;
    });
  }
);

// 6. Example Usage in Component
function EventList() {
  const dispatch = useDispatch();
  
  // Use selectors
  const events = useSelector(selectFilteredEvents);
  const status = useSelector(state => state.events.status);
  
  // Example CRUD operations
  const handleAddEvent = (newEvent) => {
    dispatch(addEvent(newEvent));
  };
  
  const handleUpdateEvent = (eventId, changes) => {
    dispatch(updateEvent({ id: eventId, changes }));
  };
  
  const handleRemoveEvent = (eventId) => {
    dispatch(removeEvent(eventId));
  };
  
  // Render list with loading state
  if (status === 'loading') {
    return <LoadingSpinner />;
  }
  
  return (
    <div>
      {events.map(event => (
        <EventCard
          key={event.id}
          event={event}
          onUpdate={handleUpdateEvent}
          onRemove={handleRemoveEvent}
        />
      ))}
    </div>
  );
}

/*
  Before (Traditional Redux):
  
  case 'ADD_EVENT':
    return {
      ...state,
      events: {
        ...state.events,
        [action.payload.id]: action.payload
      },
      ids: [...state.ids, action.payload.id]
    };
  
  case 'UPDATE_EVENT':
    return {
      ...state,
      events: {
        ...state.events,
        [action.payload.id]: {
          ...state.events[action.payload.id],
          ...action.payload.changes
        }
      }
    };
*/
```



# Relationship Management

## 1. Managing Connected Entities
**What:** Handling relationships between different entities (events, sessions, speakers)
**Why:** Maintain data consistency and enable efficient access patterns

```javascript
// features/relationships/relationshipSlice.js

import { createSlice, createEntityAdapter } from '@reduxjs/toolkit';

/*
  Instead of nesting data like:
  event: {
    sessions: [{
      speakers: []
    }]
  }
  
  We'll maintain separate normalized entities with relationship maps
*/

const initialState = {
  // Map event to sessions
  eventSessions: {
    // eventId: sessionIds[]
    '1': ['session1', 'session2']
  },
  
  // Map session to speakers
  sessionSpeakers: {
    // sessionId: speakerIds[]
    'session1': ['speaker1', 'speaker2']
  },
  
  // Map speaker to sessions (inverse relationship)
  speakerSessions: {
    // speakerId: sessionIds[]
    'speaker1': ['session1', 'session3']
  }
};

const relationshipSlice = createSlice({
  name: 'relationships',
  initialState,
  reducers: {
    // Add session to event
    addSessionToEvent: (state, action) => {
      const { eventId, sessionId } = action.payload;
      if (!state.eventSessions[eventId]) {
        state.eventSessions[eventId] = [];
      }
      if (!state.eventSessions[eventId].includes(sessionId)) {
        state.eventSessions[eventId].push(sessionId);
      }
    },

    // Add speaker to session
    addSpeakerToSession: (state, action) => {
      const { sessionId, speakerId } = action.payload;
      
      // Add to session -> speakers mapping
      if (!state.sessionSpeakers[sessionId]) {
        state.sessionSpeakers[sessionId] = [];
      }
      if (!state.sessionSpeakers[sessionId].includes(speakerId)) {
        state.sessionSpeakers[sessionId].push(speakerId);
      }
      
      // Add to speaker -> sessions mapping (maintain inverse relationship)
      if (!state.speakerSessions[speakerId]) {
        state.speakerSessions[speakerId] = [];
      }
      if (!state.speakerSessions[speakerId].includes(sessionId)) {
        state.speakerSessions[speakerId].push(sessionId);
      }
    },

    // Remove relationships (with cleanup)
    removeSessionFromEvent: (state, action) => {
      const { eventId, sessionId } = action.payload;
      if (state.eventSessions[eventId]) {
        state.eventSessions[eventId] = state.eventSessions[eventId]
          .filter(id => id !== sessionId);
        
        // Cleanup empty arrays
        if (state.eventSessions[eventId].length === 0) {
          delete state.eventSessions[eventId];
        }
      }
    }
  }
});

// Selectors for accessing relationships
export const selectEventSessions = createSelector(
  [
    (state) => state.relationships.eventSessions,
    (state) => state.sessions.entities,
    (state, eventId) => eventId
  ],
  (eventSessions, sessions, eventId) => {
    const sessionIds = eventSessions[eventId] || [];
    return sessionIds.map(id => sessions[id]);
  }
);

// Complex selector for full event data
export const selectEventWithDetails = createSelector(
  [
    (state) => state.events.entities,
    (state) => state.sessions.entities,
    (state) => state.users.entities,
    (state) => state.relationships.eventSessions,
    (state) => state.relationships.sessionSpeakers,
    (state, eventId) => eventId
  ],
  (events, sessions, users, eventSessions, sessionSpeakers, eventId) => {
    const event = events[eventId];
    if (!event) return null;

    const sessionIds = eventSessions[eventId] || [];
    const eventSessionsWithSpeakers = sessionIds.map(sessionId => {
      const session = sessions[sessionId];
      const speakerIds = sessionSpeakers[sessionId] || [];
      const speakers = speakerIds.map(speakerId => users[speakerId]);
      
      return {
        ...session,
        speakers
      };
    });

    return {
      ...event,
      sessions: eventSessionsWithSpeakers
    };
  }
);

// Usage Example
function EventDetail({ eventId }) {
  const eventWithDetails = useSelector(state => 
    selectEventWithDetails(state, eventId)
  );
  
  // Add new session to event
  const handleAddSession = (sessionData) => {
    dispatch(addSession(sessionData)); // Add to sessions slice
    dispatch(addSessionToEvent({
      eventId,
      sessionId: sessionData.id
    })); // Update relationship
  };

  return (
    <div>
      <h1>{eventWithDetails.title}</h1>
      <SessionList 
        sessions={eventWithDetails.sessions}
        onAddSession={handleAddSession}
      />
    </div>
  );
}
```

## 2. Handling Updates Across Relationships

```javascript
// Example of a complex update operation
const updateSessionWithSpeakers = createAsyncThunk(
  'sessions/updateWithSpeakers',
  async ({ sessionId, sessionData, speakerIds }, { dispatch, getState }) => {
    try {
      // Update session data
      await dispatch(updateSession({
        id: sessionId,
        changes: sessionData
      }));

      // Get current speakers
      const state = getState();
      const currentSpeakerIds = 
        state.relationships.sessionSpeakers[sessionId] || [];

      // Remove speakers that are no longer included
      const speakersToRemove = currentSpeakerIds
        .filter(id => !speakerIds.includes(id));
      
      speakersToRemove.forEach(speakerId => {
        dispatch(removeSpeakerFromSession({ sessionId, speakerId }));
      });

      // Add new speakers
      const speakersToAdd = speakerIds
        .filter(id => !currentSpeakerIds.includes(id));
      
      speakersToAdd.forEach(speakerId => {
        dispatch(addSpeakerToSession({ sessionId, speakerId }));
      });

      return sessionId;
    } catch (error) {
      throw error;
    }
  }
);
```

