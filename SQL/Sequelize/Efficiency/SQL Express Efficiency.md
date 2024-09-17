## Refactoring Expensive Calculations to SQL

### JavaScript Calculation (Inefficient)
```javascript
const users = await User.findAll();
const activeUsers = users.filter(user => user.lastLogin > someDate);
```

### SQL Calculation (Efficient)
```javascript
const activeUsers = await User.findAll({
  where: {
    lastLogin: { [Op.gt]: someDate }
  }
});
```

## Eliminating N+1 Queries

### N+1 Problem (Inefficient)
```javascript
const users = await User.findAll();
for (let user of users) {
  user.posts = await user.getPosts();
}
```

### [[Lazy vs Eager Loading|Eager Loading Solution]] (Efficient)
```javascript
const users = await User.findAll({
  include: [{ model: Post }]
});
```

## [[Index|Creating an Index]]

### Adding an Index
```javascript
// In a migration file
await queryInterface.addIndex('Users', ['email']);
```

### Composite Index
```javascript
await queryInterface.addIndex('Posts', ['userId', 'createdAt']);
```

## Query Caching

### Using Redis for Caching
```javascript
const redis = require('redis');
const client = redis.createClient();

app.get('/users', async (req, res) => {
  const cacheKey = 'all_users';
  const cachedUsers = await client.get(cacheKey);
  
  if (cachedUsers) {
    return res.json(JSON.parse(cachedUsers));
  }

  const users = await User.findAll();
  await client.set(cacheKey, JSON.stringify(users), 'EX', 3600); // Cache for 1 hour
  res.json(users);
});
```

## [[Pagination|Implementing Pagination]]

### Offset-based Pagination
```javascript
const { page = 1, limit = 20 } = req.query;
const offset = (page - 1) * limit;

const users = await User.findAndCountAll({
  limit: parseInt(limit),
  offset: offset
});
```

### Cursor-based Pagination
```javascript
const { cursor, limit = 20 } = req.query;

const users = await User.findAll({
  where: {
    id: { [Op.gt]: cursor }
  },
  limit: parseInt(limit),
  order: [['id', 'ASC']]
});
```

## Optimizing JOIN Operations

### Efficient JOIN
```javascript
const result = await User.findAll({
  include: [{
    model: Post,
    required: true, // INNER JOIN
    where: { status: 'published' }
  }],
  limit: 10
});
```

## Using Database-Specific Features

### PostgreSQL Array Operations
```javascript
// Assuming tags is an ARRAY column
const posts = await Post.findAll({
  where: {
    tags: { [Op.overlap]: ['javascript', 'nodejs'] }
  }
});
```

## Query Optimization Techniques

1. Use `SELECT` with specific columns instead of `SELECT *`
2. Utilize `EXPLAIN` to analyze query execution plans
3. Consider denormalization for read-heavy operations
4. Use appropriate data types (e.g., UUID vs. INT for IDs)
5. Implement database partitioning for very large tables

## Express.js Optimization

1. Use compression middleware
```javascript
const compression = require('compression');
app.use(compression());
```

2. Implement proper error handling
```javascript
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

3. Use asynchronous operations
```javascript
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (error) {
    next(error);
  }
});
```

Remember, optimization should be done based on profiling and actual performance bottlenecks in your specific application. Always measure the impact of your optimizations to ensure they're providing the expected benefits.
## Benchmarking Performance

Benchmarking is crucial for understanding and improving your application's performance. Here are key points and methods for effective benchmarking:

### Tools and Techniques

1. **Built-in Node.js Profiler:**
   ```bash
   node --prof app.js
   ```
   Analyze with:
   ```bash
   node --prof-process isolate-0xnnnnnnnnnnnn-v8.log > processed.txt
   ```

2. **Console Time:**
   ```javascript
   console.time('operationName');
   // ... code to benchmark
   console.timeEnd('operationName');
   ```

3. **Apache Bench (ab) for HTTP requests:**
   ```bash
   ab -n 1000 -c 100 http://localhost:3000/api/endpoint
   ```

4. **Siege for load testing:**
   ```bash
   siege -c 100 -t 1M http://localhost:3000/api/endpoint
   ```

### Database Query Performance

1. **Sequelize Logging:**
   ```javascript
   const sequelize = new Sequelize('database', 'username', 'password', {
     logging: console.log // or use a custom logger
   });
   ```

2. **Explain Queries:**
   ```sql
   EXPLAIN ANALYZE SELECT * FROM Users WHERE email = 'user@example.com';
   ```

### Best Practices

1. Benchmark in an environment similar to production.
2. Run multiple iterations and average the results.
3. Test with varying loads and data sizes.
4. Compare before and after optimization changes.
5. Monitor both response times and resource utilization (CPU, memory, I/O).

### Code-level Profiling

Use Node.js built-in `perf_hooks`:

```javascript
const { performance, PerformanceObserver } = require('perf_hooks');

const obs = new PerformanceObserver((items) => {
  console.log(items.getEntries()[0].duration);
  performance.clearMarks();
});
obs.observe({ entryTypes: ['measure'] });

performance.mark('A');
// Code to benchmark
performance.mark('B');
performance.measure('A to B', 'A', 'B');
```

### Continuous Monitoring

Implement continuous performance monitoring in your CI/CD pipeline to catch performance regressions early.

Remember, benchmarking should be an ongoing process. Regularly benchmark your application to ensure performance improvements are maintained and to identify new optimization opportunities as your application evolves.