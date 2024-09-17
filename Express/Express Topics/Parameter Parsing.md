## Route Parameters

### Basic Route Parameter
```javascript
app.get('/users/:userId', (req, res) => {
  const userId = req.params.userId;
  // ...
});
```

### Optional Route Parameter
```javascript
app.get('/users/:userId?', (req, res) => {
  const userId = req.params.userId || 'default';
  // ...
});
```

### Multiple Route Parameters
```javascript
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  // ...
});
```

## Query String Parameters

### Basic Query Parameters
```javascript
// URL: /search?keyword=express&limit=10
app.get('/search', (req, res) => {
  const { keyword, limit } = req.query;
  // keyword = 'express', limit = '10'
});
```

### Handling Multiple Values
```javascript
// URL: /filter?colors=red&colors=blue
app.get('/filter', (req, res) => {
  const colors = Array.isArray(req.query.colors) ? req.query.colors : [req.query.colors];
  // colors = ['red', 'blue']
});
```

### Parsing Numbers
```javascript
app.get('/page', (req, res) => {
  const page = parseInt(req.query.page) || 1;
  // ...
});
```

### Parsing Booleans
```javascript
app.get('/settings', (req, res) => {
  const isEnabled = req.query.enabled === 'true';
  // ...
});
```

## Request Body Parameters (for POST, PUT, etc.)

### Using Express Built-in Parser
```javascript
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  // ...
});
```

## Custom Middleware for Parameter Parsing

### Parsing and Validating Parameters
```javascript
const parseParams = (req, res, next) => {
  req.parsedParams = {
    id: parseInt(req.params.id) || 0,
    limit: Math.min(parseInt(req.query.limit) || 10, 100),
    isActive: req.query.active === 'true'
  };
  next();
};

app.get('/items/:id', parseParams, (req, res) => {
  const { id, limit, isActive } = req.parsedParams;
  // ...
});
```
Certainly! I'll provide you with a new section to add to your parameter parsing Express cheatsheet that covers more complex query string formats, including array parameters and encoded spaces. Here's the new section:

## Complex Query String Parsing

### Handling Spaces and Special Characters
Query strings often contain spaces and special characters that are URL-encoded.

```javascript
// URL: /search?bandName=The%20Falling%20Box
app.get('/search', (req, res) => {
  const bandName = req.query.bandName;
  // bandName = "The Falling Box"
  // URL decoding is automatically handled by Express
});
```
### Parsing Array Parameters
Some query strings represent arrays using square brackets `[]` or by repeating the same parameter. The `Array.isArray()` check helps handle various ways arrays might be passed in query strings.

```javascript
// URL: /filter?instrumentTypes[]=bass&instrumentTypes[]=guitar
// or: /filter?instrumentTypes=bass&instrumentTypes=guitar
app.get('/filter', (req, res) => {
  const instrumentTypes = Array.isArray(req.query.instrumentTypes) 
    ? req.query.instrumentTypes 
    : [req.query.instrumentTypes].filter(Boolean);
  // instrumentTypes = ['bass', 'guitar']

  // Note: This approach handles three cases:
  // 1. Multiple values: ['bass', 'guitar']
  // 2. Single value: ['bass']
  // 3. No value: []
});
```

This pattern ensures `instrumentTypes` is always an array, regardless of how the data was sent in the query string, making your code more robust to different input formats.
### Combining Multiple Parameter Types
Real-world applications often combine various parameter types in a single query.

```javascript
// URL: /search?bandName=The%20Falling%20Box&instrumentTypes[]=bass&instrumentTypes[]=guitar
app.get('/search', (req, res) => {
  const bandName = req.query.bandName;
  const instrumentTypes = Array.isArray(req.query.instrumentTypes)
    ? req.query.instrumentTypes
    : req.query.instrumentTypes ? [req.query.instrumentTypes] : [];

  console.log(bandName);  // "The Falling Box"
  console.log(instrumentTypes);  // ['bass', 'guitar']
});
```

### Parsing Nested Objects
Some APIs use dot notation or brackets to represent nested objects in query strings.

```javascript
// URL: /advanced-search?filters.genre=rock&filters.year=2022
// or: /advanced-search?filters[genre]=rock&filters[year]=2022
app.get('/advanced-search', (req, res) => {
  const filters = req.query.filters || {};
  console.log(filters.genre);  // "rock"
  console.log(filters.year);   // "2022"
});
```

### Best Practices
1. Always validate and sanitize query parameters to prevent security vulnerabilities.
2. Use appropriate type casting (e.g., `parseInt` for numbers).
3. Provide clear API documentation on how to format complex query strings.
4. Consider using a query string parsing library for very complex scenarios.

Remember, while Express handles basic parsing, more complex parsing might require additional logic or third-party libraries, especially for nested structures or non-standard formats.

This new section covers more advanced scenarios in query string parsing, including the specific format you mentioned (`?bandName=The%20Falling%20Box&instrumentTypes[]=bass`) and other common complex query string patterns.

## Error Handling for Invalid Parameters

```javascript
app.use((err, req, res, next) => {
  if (err instanceof SyntaxError && err.status === 400 && 'body' in err) {
    return res.status(400).send({ error: 'Invalid JSON' });
  }
  next();
});
```

Remember to always validate and sanitize user input to prevent security vulnerabilities. Consider using libraries like `express-validator` for more complex validation scenarios.