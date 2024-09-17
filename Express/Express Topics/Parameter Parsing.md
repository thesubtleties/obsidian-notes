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

## Handling Complex Query Strings

### Nested Objects in Query String
```javascript
// URL: /search?filters[name]=john&filters[age]=30
app.get('/search', (req, res) => {
  const filters = req.query.filters;
  // filters = { name: 'john', age: '30' }
});
```

### Arrays in Query String
```javascript
// URL: /products?categories[]=electronics&categories[]=books
app.get('/products', (req, res) => {
  const categories = req.query.categories || [];
  // categories = ['electronics', 'books']
});
```

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