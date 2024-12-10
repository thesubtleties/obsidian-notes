## Basic Setup Comparison
```javascript
// Express
const express = require('express');
const app = express();
app.use(express.json());

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```
```python
# Flask
from flask import Flask, request, jsonify
app = Flask(__name__)

if __name__ == '__main__':
    app.run(port=3000)
```

## Route Handlers
```javascript
// Express
app.get('/api/users', (req, res) => {
    res.json({ users: [] });
});

app.post('/api/users', (req, res) => {
    const data = req.body;
    res.status(201).json(data);
});
```
```python
# Flask
@app.route('/api/users', methods=['GET'])
def get_users():
    return jsonify({"users": []})

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.json
    return jsonify(data), 201
```

## Middleware
```javascript
// Express
app.use((req, res, next) => {
    console.log(`${req.method} ${req.path}`);
    next();
});

const auth = (req, res, next) => {
    if (!req.headers.authorization) {
        return res.status(401).json({ error: 'Unauthorized' });
    }
    next();
};
```
```python
# Flask
from functools import wraps

@app.before_request
def logging_middleware():
    print(f"{request.method} {request.path}")

def auth_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        if not request.headers.get('Authorization'):
            return jsonify({"error": "Unauthorized"}), 401
        return f(*args, **kwargs)
    return decorated
```

## Request/Response Handling
```javascript
// Express
app.get('/api/items', (req, res) => {
    const query = req.query.search;  // URL params
    const headers = req.headers;     // Headers
    const body = req.body;           // Body
    const params = req.params.id;    // Route params
});
```
```python
# Flask
@app.route('/api/items')
def get_items():
    query = request.args.get('search')  # URL params
    headers = request.headers          # Headers
    body = request.json               # Body
    # Route params come as function parameters
```

## Error Handling
```javascript
// Express
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ error: 'Something broke!' });
});

// Custom error
app.get('/error', (req, res) => {
    throw new Error('Custom error');
});
```
```python
# Flask
@app.errorhandler(500)
def handle_500(error):
    return jsonify({'error': 'Something broke!'}), 500

@app.route('/error')
def cause_error():
    raise Exception('Custom error')
```

## Route Parameters
```javascript
// Express
app.get('/users/:id', (req, res) => {
    const id = req.params.id;
});
```
```python
# Flask
@app.route('/users/<int:id>')
def get_user(id):
    return jsonify({"id": id})
```

## Database Integration Example
```javascript
// Express with Sequelize
const User = require('./models/user');

app.get('/users', async (req, res) => {
    const users = await User.findAll();
    res.json(users);
});
```
```python
# Flask with SQLAlchemy
from models import User, db

@app.route('/users')
def get_users():
    users = User.query.all()
    return jsonify([user.to_dict() for user in users])
```

## File Upload
```javascript
// Express with multer
const multer = require('multer');
const upload = multer({ dest: 'uploads/' });

app.post('/upload', upload.single('file'), (req, res) => {
    res.json({ file: req.file });
});
```
```python
# Flask
@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({"error": "No file"}), 400
    file = request.files['file']
    file.save('uploads/' + file.filename)
    return jsonify({"file": file.filename})
```

## Environment Variables
```javascript
// Express
require('dotenv').config();
const port = process.env.PORT;
```
```python
# Flask
from dotenv import load_dotenv
import os

load_dotenv()
port = os.environ.get('PORT')
```

## CORS
```javascript
// Express
const cors = require('cors');
app.use(cors());
```
```python
# Flask
from flask_cors import CORS
CORS(app)
```

## Key Differences to Remember:
1. Decorators vs Middleware
   - Flask uses decorators (@app.route)
   - Express uses middleware chaining

2. Request Handling
   - Flask: request is a global object
   - Express: req is passed to each route handler

3. Response Format
   - Flask: return values are converted to responses
   - Express: explicitly use res.send() or res.json()

4. Route Parameters
   - Flask: defined in route with <param>
   - Express: defined with :param

5. Error Handling
   - Flask: uses decorators for error handlers
   - Express: uses middleware for error handling

Remember:
- Flask is more "magical" (implicit)
- Express is more explicit
- Flask uses Python conventions
- Express uses JavaScript/Node conventions
- Both can accomplish the same things
- Choose based on your team's expertise