## Basic Count

### Count All Records
```javascript
const count = await User.count();
```

### Count with Where Clause
```javascript
const count = await User.count({
  where: {
    status: 'active'
  }
});
```

## Advanced Counting

### Distinct Count
```javascript
const count = await User.count({
  distinct: true,
  col: 'id'
});
```

### Count with Group By
```javascript
const counts = await User.count({
  attributes: ['status'],
  group: 'status'
});
```

### Count with Include
```javascript
const count = await User.count({
  include: [{
    model: Post,
    where: { status: 'published' }
  }]
});
```

## Aggregations

### Sum
```javascript
const total = await Order.sum('amount');
```

### Average
```javascript
const avgAge = await User.avg('age');
```

### Min and Max
```javascript
const oldest = await User.max('age');
const youngest = await User.min('age');
```

## Complex Counting

### Subquery Count
```javascript
const { Op, literal } = require('sequelize');

const usersWithPosts = await User.findAll({
  attributes: {
    include: [
      [
        literal(`(
          SELECT COUNT(*)
          FROM Posts AS post
          WHERE post.userId = User.id
        )`),
        'postCount'
      ]
    ]
  },
  where: {
    '$postCount$': { [Op.gt]: 0 }
  }
});
```

### Count with Raw Query
```javascript
const [results, metadata] = await sequelize.query(
  "SELECT COUNT(*) as count FROM Users WHERE age > :age",
  {
    replacements: { age: 18 },
    type: QueryTypes.SELECT
  }
);
const count = results[0].count;
```

## Performance Considerations

1. Use `count({ col: 'id' })` instead of `count()` for better performance on large tables.
2. Be cautious with `include` in count queries, as they can significantly impact performance.
3. For large datasets, consider using raw SQL queries for complex counts.
4. Utilize database indexes on columns frequently used in count operations.

## Best Practices

1. Always handle the case where count might return `null` or `undefined`.
2. Use transactions for counts that are part of critical operations.
3. Consider caching count results for frequently accessed, slowly changing data.
4. Be aware of the differences in count behavior with `paranoid` models.

Remember, the efficiency of count operations can vary depending on your database system and table size. Always test and optimize for your specific use case.