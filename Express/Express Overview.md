# Express Overview

Express is a minimal and flexible Node.js web application framework that provides a robust set of features for web and mobile applications. It's designed to make the process of building web applications and APIs easier and more intuitive.

## Key Concepts

1. **Middleware**: Functions that have access to the request object (req), the response object (res), and the next middleware function in the application's request-response cycle, usually denoted by a variable named `next`.

2. **Routing**: Refers to how an application's endpoints (URIs) respond to client requests. You define routing using methods of the Express app object that correspond to HTTP methods.

3. **Request & Response Objects**: Express enhances Node's req and res objects with additional properties and methods.

4. **Static File Serving**: Express can serve static files and assets directly.

5. **Template Engines**: Express can be used with various template engines to dynamically generate HTML.

## Basic Usage

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## [[Express Setup]]

Setting up an Express application involves several steps:

1. Initialize a new Node.js project
2. Install Express
3. Create the main application file
4. Set up basic middleware
5. Define routes
6. Start the server

## Middleware

Middleware functions can:
- Execute any code
- Make changes to the request and response objects
- End the request-response cycle
- Call the next middleware in the stack

Example:
```javascript
app.use((req, res, next) => {
  console.log('Time:', Date.now());
  next();
});
```

## [[Express Routes|Routing]]

Express supports all HTTP methods:

```javascript
app.get('/users', (req, res) => {
  // Handle GET request
});

app.post('/users', (req, res) => {
  // Handle POST request
});
```

## Request & Response

```javascript
app.get('/api/users/:id', (req, res) => {
  const userId = req.params.id;
  const userAgent = req.get('User-Agent');
  res.json({ userId, userAgent });
});
```
## [[Parameter Parsing|Parameter Parsing in Express]]

Express provides several ways to access parameters from client requests:

1. **Route Parameters**: Access parts of the URL path
   ```javascript
   app.get('/users/:userId', (req, res) => {
     const userId = req.params.userId;
   });
   ```

2. **Query String Parameters**: Parse data from URL query strings
   ```javascript
   // URL: /search?keyword=express
   app.get('/search', (req, res) => {
     const keyword = req.query.keyword;
   });
   ```

3. **Request Body**: Parse data sent in the request body (requires middleware)
   ```javascript
   app.use(express.json());
   app.post('/users', (req, res) => {
     const { name, email } = req.body;
   });
   ```

Parameter parsing allows you to extract and utilize data sent by the client, enabling dynamic and interactive server responses. Always validate and sanitize parameters to ensure application security.
## Error Handling

```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```
Certainly! I'll provide you with a small section on Auth for your Express overview cheatsheet, and then a more comprehensive cheatsheet on Auth in Express, including JWT.

For your Express Overview Cheatsheet:

## [[Auth in Express|Authentication in Express]]

Authentication is crucial for securing your Express application. Common approaches include:

1. **Session-based Authentication**: Uses server-side sessions to track logged-in users.
2. **Token-based Authentication (e.g., JWT)**: Stateless authentication using encrypted tokens.
3. **OAuth**: For third-party authentication providers.

Basic JWT implementation:

```javascript
const jwt = require('jsonwebtoken');

// Login route
app.post('/login', (req, res) => {
  // Verify user credentials
  const token = jwt.sign({ userId: user.id }, 'secret_key', { expiresIn: '1h' });
  res.json({ token });
});

// Middleware to verify JWT
const verifyToken = (req, res, next) => {
  const token = req.headers['authorization'];
  if (!token) return res.status(403).send('No token provided');
  
  jwt.verify(token, 'secret_key', (err, decoded) => {
    if (err) return res.status(401).send('Invalid token');
    req.userId = decoded.userId;
    next();
  });
};

// Protected route
app.get('/protected', verifyToken, (req, res) => {
  res.send('Access granted');
});
```
## Best Practices

1. Use environment variables for configuration
2. Implement proper error handling
3. Use middleware for common functionalities
4. Structure your application with separation of concerns
5. Use async/await for asynchronous operations
6. Implement proper logging
7. Use security best practices (e.g., helmet middleware)

## Popular Middleware

- `body-parser`: Parse incoming request bodies
- `morgan`: HTTP request logger
- `cors`: Enable CORS with various options
- `helmet`: Help secure Express apps with various HTTP headers
- `express-validator`: Validate and sanitize user input

Remember, Express is highly customizable and can be extended with numerous third-party middleware and libraries to suit your specific needs.