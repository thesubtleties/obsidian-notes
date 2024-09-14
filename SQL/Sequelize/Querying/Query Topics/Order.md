
# Sequelize Order By Cheatsheet

## Basic Ordering

### Single Column Ascending
```javascript
const users = await User.findAll({
  order: [['name', 'ASC']]
});
```

### Single Column Descending
```javascript
const users = await User.findAll({
  order: [['createdAt', 'DESC']]
});
```

### Multiple Columns
```javascript
const users = await User.findAll({
  order: [
    ['lastName', 'ASC'],
    ['firstName', 'ASC']
  ]
});
```

## Advanced Ordering

### Ordering by Association
```javascript
const users = await User.findAll({
  include: [{ model: Post, as: 'posts' }],
  order: [
    ['name', 'ASC'],
    [{ model: Post, as: 'posts' }, 'title', 'DESC']
  ]
});
```

### Ordering with Functions
```javascript
const users = await User.findAll({
  order: [
    [sequelize.fn('LOWER', sequelize.col('name')), 'ASC']
  ]
});
```

### Ordering with Literal SQL
```javascript
const users = await User.findAll({
  order: [
    [sequelize.literal('CHAR_LENGTH(name)'), 'DESC']
  ]
});
```

## Conditional Ordering

### Dynamic Ordering
```javascript
const orderColumn = req.query.orderBy || 'createdAt';
const orderDirection = req.query.direction === 'ASC' ? 'ASC' : 'DESC';

const users = await User.findAll({
  order: [[orderColumn, orderDirection]]
});
```

### Ordering with Case Statement
```javascript
const users = await User.findAll({
  order: [
    [
      sequelize.literal(`
        CASE
          WHEN status = 'active' THEN 1
          WHEN status = 'pending' THEN 2
          ELSE 3
        END
      `),
      'ASC'
    ],
    ['name', 'ASC']
  ]
});
```

## Ordering in Associations

### Eager Loading with Ordered Association
```javascript
const users = await User.findAll({
  include: [{
    model: Post,
    as: 'posts',
    separate: true,
    order: [['createdAt', 'DESC']]
  }],
  order: [['name', 'ASC']]
});
```

### Ordering in Many-to-Many Relationships
```javascript
const users = await User.findAll({
  include: [{
    model: Project,
    through: { attributes: [] }
  }],
  order: [
    ['name', 'ASC'],
    [Project, 'name', 'DESC']
  ]
});
```

## Pagination with Ordering

### Offset-based Pagination
```javascript
const { count, rows } = await User.findAndCountAll({
  order: [['createdAt', 'DESC']],
  limit: 10,
  offset: 0
});
```

### Cursor-based Pagination
```javascript
const users = await User.findAll({
  where: {
    createdAt: { [Op.lt]: lastCreatedAt }
  },
  order: [['createdAt', 'DESC']],
  limit: 10
});
```

## Key Points and Gotchas

1. Always use an array of arrays for `order`, even for single column ordering.
2. Column names are usually automatically escaped, but use `sequelize.literal()` for raw SQL.
3. When ordering by an associated model's column, make sure the association is included in the query.
4. Ordering can significantly impact query performance, especially on large datasets. Ensure relevant columns are indexed.
5. Be cautious when allowing user input for ordering columns to prevent SQL injection.
6. Some complex orderings might not be possible using Sequelize's ORM methods and may require raw queries.
7. When using `limit` and `offset` with ordering, ensure consistent ordering for reliable pagination.
8. Ordering by computed columns or function results may prevent the use of indexes, impacting performance.
9. In MySQL, ordering by a TEXT or BLOB column requires creating a function-based index or using a generated column.
10. PostgreSQL allows `NULLS FIRST` or `NULLS LAST` in ordering, which can be achieved in Sequelize using raw SQL.

Remember to test your queries with representative data volumes to ensure performance meets your requirements, especially when combining complex orderings with other query conditions and associations.