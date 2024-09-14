## Basic Route Structure

```javascript
app.METHOD(PATH, HANDLER)
```

Where:
- `app` is an instance of express
- `METHOD` is an HTTP request method (GET, POST, PUT, DELETE, etc.)
- `PATH` is the path on the server
- `HANDLER` is the function executed when the route is matched

## HTTP Methods

```javascript
app.get('/', (req, res) => {
  res.send('GET request to homepage');
});

app.post('/', (req, res) => {
  res.send('POST request to homepage');
});

app.put('/', (req, res) => {
  res.send('PUT request to homepage');
});

app.delete('/', (req, res) => {
  res.send('DELETE request to homepage');
});

app.all('/secret', (req, res) => {
  res.send('Accessing the secret section ...');
});
```

## Route Parameters

```javascript
app.get('/users/:userId/books/:bookId', (req, res) => {
  res.send(req.params);
});
```

## Route Handlers

Single callback:
```javascript
app.get('/example', (req, res) => {
  res.send('Hello World!');
});
```

Multiple callbacks:
```javascript
app.get('/example', (req, res, next) => {
  console.log('First callback');
  next();
}, (req, res) => {
  res.send('Second callback');
});
```

## Response Methods

```javascript
res.json({ user: 'tobi' });  // Send a JSON response
res.status(404).send('Sorry, we cannot find that!');  // Set status and send response
res.sendFile('/path/to/file.txt');  // Send a file
res.render('index', { title: 'Hey', message: 'Hello there!' });  // Render a view template
```

## Chainable Route Handlers

```javascript
app.route('/book')
  .get((req, res) => {
    res.send('Get a random book');
  })
  .post((req, res) => {
    res.send('Add a book');
  })
  .put((req, res) => {
    res.send('Update the book');
  });
```

## Express Router

```javascript
// users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.send('Users home page');
});

router.get('/:id', (req, res) => {
  res.send(`User ${req.params.id}`);
});

module.exports = router;

// app.js
const usersRouter = require('./users');
app.use('/users', usersRouter);
```

## Middleware in Routes

```javascript
const myMiddleware = (req, res, next) => {
  console.log('Middleware executed');
  next();
};

app.get('/example', myMiddleware, (req, res) => {
  res.send('Hello World!');
});
```

## Error Handling in Routes

```javascript
app.get('/error', (req, res, next) => {
  const err = new Error('Something went wrong');
  err.status = 500;
  next(err);
});

// Error handling middleware
app.use((err, req, res, next) => {
  res.status(err.status || 500);
  res.json({
    error: {
      message: err.message
    }
  });
});
```

## Async Route Handlers

```javascript
app.get('/async', async (req, res, next) => {
  try {
    const result = await someAsyncOperation();
    res.json(result);
  } catch (error) {
    next(error);
  }
});
```

## Regular Expressions in Routes

```javascript
app.get(/.*fly$/, (req, res) => {
  res.send('The URL path ends with "fly"');
});
```

Remember to organize your routes logically, use descriptive names, and leverage Express Router for modular route handling in larger applications.