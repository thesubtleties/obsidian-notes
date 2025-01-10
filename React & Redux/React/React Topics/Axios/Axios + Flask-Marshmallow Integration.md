## Basic CRUD Operations

### GET Requests
```javascript
// Single item
const getEvent = async (id) => {
  try {
    const response = await axios.get(`/api/events/${id}`);
    // Schema automatically formats response.data
    return response.data;
  } catch (error) {
    console.error('Error fetching event:', error.response?.data);
  }
}

// List of items
const getEvents = async () => {
  const response = await axios.get('/api/events');
  // Schema handles array with many=True
  return response.data;
}
```

### POST Requests
```javascript
// Create new item
const createEvent = async (eventData) => {
  try {
    const response = await axios.post('/api/events', {
      title: eventData.title,
      start_date: eventData.startDate,
      // Schema will validate these fields
    });
    return response.data;
  } catch (error) {
    // Schema validation errors
    if (error.response?.data?.errors) {
      throw error.response.data.errors;
    }
  }
}
```

### PUT/PATCH Requests
```javascript
// Full update (PUT)
const updateEvent = async (id, eventData) => {
  const response = await axios.put(`/api/events/${id}`, eventData);
  return response.data;
}

// Partial update (PATCH)
const updateEventPartial = async (id, changes) => {
  // Schema(partial=True) handles partial updates
  const response = await axios.patch(`/api/events/${id}`, changes);
  return response.data;
}
```

## Error Handling

### Standard Error Pattern
```javascript
const apiCall = async () => {
  try {
    const response = await axios.post('/api/events', data);
    return response.data;
  } catch (error) {
    if (error.response) {
      // Schema validation errors
      if (error.response.status === 400) {
        console.log('Validation failed:', error.response.data.errors);
      }
      // Authentication errors
      if (error.response.status === 401) {
        console.log('Please login');
      }
    }
    throw error;
  }
}
```

## Working with Relationships

### Nested Data
```javascript
// Getting event with related data
const getEventDetails = async (id) => {
  const response = await axios.get(`/api/events/${id}`);
  const event = response.data;
  
  console.log(event.sessions);  // Nested by SessionSchema
  console.log(event.organizer); // Nested by UserSchema
}

// Creating with relationships
const createEventWithSessions = async (eventData) => {
  const response = await axios.post('/api/events', {
    title: eventData.title,
    sessions: [
      { title: 'Session 1', start_time: '2025-02-01T09:00' },
      { title: 'Session 2', start_time: '2025-02-01T10:00' }
    ]
  });
  // Schema handles nested creation
}
```

## Axios Instance Setup

### Base Configuration
```javascript
// api.js
const api = axios.create({
  baseURL: 'http://localhost:5000/api',
  headers: {
    'Content-Type': 'application/json'
  }
});

// Add auth token
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});
```

## Common Patterns

### Form Submission
```javascript
const handleSubmit = async (event) => {
  event.preventDefault();
  try {
    const response = await api.post('/events', formData);
    // Schema has validated and transformed data
    console.log('Created:', response.data);
  } catch (error) {
    // Handle validation errors from schema
    setErrors(error.response?.data?.errors || {});
  }
}
```

### Data Loading States
```javascript
const EventComponent = () => {
  const [event, setEvent] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const loadEvent = async () => {
      try {
        const response = await api.get(`/events/${id}`);
        // Schema formatted data
        setEvent(response.data);
      } catch (error) {
        setError(error.response?.data);
      } finally {
        setLoading(false);
      }
    };
    loadEvent();
  }, [id]);
}
```

### File Uploads
```javascript
const uploadEventImage = async (eventId, file) => {
  const formData = new FormData();
  formData.append('image', file);
  
  const response = await api.post(
    `/events/${eventId}/image`, 
    formData,
    {
      headers: {
        'Content-Type': 'multipart/form-data'
      }
    }
  );
  // Schema handles response formatting
  return response.data;
}
```

## Type Safety (if using TypeScript)
```typescript
// Types from schema structure
interface Event {
  id: number;
  title: string;
  start_date: string;
  sessions?: Session[];
}

// Type-safe requests
const getEvent = async (id: number): Promise<Event> => {
  const response = await api.get<Event>(`/events/${id}`);
  return response.data;
}
```

Remember:
- Schema handles validation on backend
- Axios handles HTTP communication
- Together they provide clean data flow
- Error handling becomes predictable
- Relationships are handled automatically

