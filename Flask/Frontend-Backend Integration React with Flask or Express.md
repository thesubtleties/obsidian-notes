## Basic Fetch Examples
```javascript
// React Component
const [data, setData] = useState(null);

// Works the same for both Flask/Express backends
const fetchData = async () => {
    try {
        const response = await fetch('/api/data');
        const json = await response.json();
        setData(json);
    } catch (error) {
        console.error('Error:', error);
    }
};
```

## CRUD Operations with Both Backends

### GET Request
```javascript
// React
const getData = async () => {
    const response = await fetch('/api/items');
    const data = await response.json();
    
    // Flask returns: {"items": [...]}
    // Express returns: {items: [...]}
    // (Minor JSON formatting differences)
};
```

### POST Request
```javascript
// React
const createItem = async (item) => {
    const response = await fetch('/api/items', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(item)
    });
    const data = await response.json();
};

// Flask backend expects:
@app.route('/api/items', methods=['POST'])
def create_item():
    data = request.json
    return jsonify(data), 201

// Express backend expects:
app.post('/api/items', (req, res) => {
    const data = req.body;
    res.status(201).json(data);
});
```

### File Upload
```javascript
// React
const uploadFile = async (file) => {
    const formData = new FormData();
    formData.append('file', file);

    const response = await fetch('/api/upload', {
        method: 'POST',
        body: formData,
        // Don't set Content-Type header - browser will set it
    });
    const data = await response.json();
};

// Flask backend:
@app.route('/api/upload', methods=['POST'])
def upload_file():
    file = request.files['file']
    return jsonify({"filename": file.filename})

// Express backend:
app.post('/api/upload', upload.single('file'), (req, res) => {
    res.json({ filename: req.file.filename });
});
```

## Authentication

### JWT with Both Backends
```javascript
// React - Login
const login = async (credentials) => {
    const response = await fetch('/api/login', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(credentials)
    });
    const data = await response.json();
    // Store token
    localStorage.setItem('token', data.token);
};

// Protected Request
const getProtectedData = async () => {
    const token = localStorage.getItem('token');
    const response = await fetch('/api/protected', {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    });
    const data = await response.json();
};

// Flask backend:
@app.route('/api/protected')
@auth_required  # Custom decorator
def protected():
    return jsonify({"data": "protected"})

// Express backend:
app.get('/api/protected', authMiddleware, (req, res) => {
    res.json({ data: "protected" });
});
```

## Error Handling
```javascript
// React - Comprehensive Error Handling
const apiRequest = async (url, options = {}) => {
    try {
        const response = await fetch(url, options);
        
        // Handle both Flask and Express error responses
        if (!response.ok) {
            // Flask typically sends: {"error": "message"}
            // Express typically sends: {error: "message"}
            const error = await response.json();
            throw new Error(error.error || 'Something went wrong');
        }
        
        return await response.json();
    } catch (error) {
        // Handle network errors or JSON parsing errors
        console.error('API Error:', error);
        throw error;
    }
};

// Usage with Error Boundary or try-catch
try {
    const data = await apiRequest('/api/items');
    setData(data);
} catch (error) {
    setError(error.message);
}
```

## Development Setup
```javascript
// React with Flask (package.json)
{
    "proxy": "http://localhost:5000"
}

// React with Express (package.json)
{
    "proxy": "http://localhost:3000"
}

// This allows you to use relative URLs in fetch
// Instead of: fetch('http://localhost:5000/api/data')
// You can use: fetch('/api/data')
```

## Common Gotchas

### CORS Issues
```javascript
// React - If CORS is needed (development)
fetch('http://localhost:5000/api/data', {
    credentials: 'include',  // For cookies
    headers: {
        'Access-Control-Allow-Origin': '*'
    }
});

// Flask Solution:
from flask_cors import CORS
CORS(app, supports_credentials=True)

// Express Solution:
app.use(cors({
    origin: 'http://localhost:3000',
    credentials: true
}));
```

### Content Type
```javascript
// React - Sending JSON
fetch('/api/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify(data)
});

// Both backends will parse this correctly:
// Flask: request.json
// Express: req.body (with express.json() middleware)
```

Remember:
- Use proxy in development
- Handle errors consistently
- Check CORS settings
- Use appropriate Content-Type headers
- Handle loading and error states in React
- Consider using axios or other fetch wrappers
- Be consistent with API response formats
- Use environment variables for URLs in production