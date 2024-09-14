# Sequelize Grouping Cheatsheet

## Basic Grouping

### Simple Group By
```javascript
const result = await User.findAll({
  attributes: ['status', [sequelize.fn('COUNT', sequelize.col('id')), 'userCount']],
  group: ['status']
});
```

### Multiple Columns Grouping
```javascript
const result = await Order.findAll({
  attributes: [
    'status',
    'paymentMethod',
    [sequelize.fn('SUM', sequelize.col('amount')), 'totalAmount']
  ],
  group: ['status', 'paymentMethod']
});
```

## Aggregation Functions

### COUNT
```javascript
const result = await Post.findAll({
  attributes: [
    'userId',
    [sequelize.fn('COUNT', sequelize.col('id')), 'postCount']
  ],
  group: ['userId']
});
```

### SUM
```javascript
const result = await Order.findAll({
  attributes: [
    'userId',
    [sequelize.fn('SUM', sequelize.col('amount')), 'totalSpent']
  ],
  group: ['userId']
});
```

### AVG
```javascript
const result = await Product.findAll({
  attributes: [
    'category',
    [sequelize.fn('AVG', sequelize.col('price')), 'averagePrice']
  ],
  group: ['category']
});
```

### MIN and MAX
```javascript
const result = await Order.findAll({
  attributes: [
    'userId',
    [sequelize.fn('MIN', sequelize.col('amount')), 'minOrder'],
    [sequelize.fn('MAX', sequelize.col('amount')), 'maxOrder']
  ],
  group: ['userId']
});
```

## Advanced Grouping

### Grouping with HAVING Clause
```javascript
const result = await Order.findAll({
  attributes: [
    'userId',
    [sequelize.fn('SUM', sequelize.col('amount')), 'totalSpent']
  ],
  group: ['userId'],
  having: sequelize.where(sequelize.fn('SUM', sequelize.col('amount')), {
    [Op.gt]: 1000
  })
});
```
This query groups orders by userId and calculates the total amount spent for each user. The HAVING clause filters out groups where the total spent is not greater than 1000. `sequelize.fn('SUM', sequelize.col('amount'))` creates an SQL function call to SUM the 'amount' column.

### Grouping with Associations
```javascript
const result = await User.findAll({
  attributes: [
    'id',
    'name',
    [sequelize.fn('COUNT', sequelize.col('posts.id')), 'postCount']
  ],
  include: [{
    model: Post,
    attributes: []
  }],
  group: ['User.id']
});
```
This query retrieves users along with their post count. It joins the User and Post tables, groups by User.id, and counts the number of posts for each user. The `include` with empty attributes ensures the join without fetching post data.

### Grouping with Raw SQL
```javascript
const result = await sequelize.query(`
  SELECT 
    EXTRACT(YEAR FROM "createdAt") as year,
    EXTRACT(MONTH FROM "createdAt") as month,
    COUNT(*) as count
  FROM "Orders"
  GROUP BY 
    EXTRACT(YEAR FROM "createdAt"),
    EXTRACT(MONTH FROM "createdAt")
  ORDER BY year, month
`, { type: QueryTypes.SELECT });
```
This raw SQL query groups orders by year and month of their creation date. It uses database-specific EXTRACT function to pull year and month from the date. This approach is useful when Sequelize's ORM methods are insufficient for complex date manipulations.

## Subqueries with Grouping

### Subquery in WHERE Clause
```javascript
const avgOrderAmount = await Order.findOne({
  attributes: [[sequelize.fn('AVG', sequelize.col('amount')), 'avgAmount']]
});

const result = await User.findAll({
  attributes: [
    'id',
    'name',
    [sequelize.fn('SUM', sequelize.col('Orders.amount')), 'totalSpent']
  ],
  include: [{
    model: Order,
    attributes: []
  }],
  group: ['User.id'],
  having: sequelize.where(
    sequelize.fn('SUM', sequelize.col('Orders.amount')),
    {
      [Op.gt]: avgOrderAmount.getDataValue('avgAmount')
    }
  )
});
```
This complex query first calculates the average order amount across all orders. Then, it finds users whose total order amount is above this average. It demonstrates how to use a subquery result in a HAVING clause. `sequelize.where()` is used to create a condition for the HAVING clause, comparing the sum of each user's orders to the overall average.

These examples showcase how Sequelize can handle complex grouping scenarios, including working with associations, using raw SQL for database-specific operations, and incorporating subqueries into grouped queries. The `sequelize.fn()` method is used to represent SQL functions, while `sequelize.col()` refers to table columns in a database-agnostic way.
## Key Points and Gotchas

1. Always include grouped columns in the `attributes` array.
2. Use `sequelize.fn()` and `sequelize.col()` for aggregate functions and column references.
3. The `HAVING` clause filters groups, while `WHERE` filters rows before grouping.
4. Grouping can significantly impact query performance. Ensure relevant columns are indexed.
5. When grouping with associations, include the association and set its `attributes` to an empty array to avoid fetching unnecessary data.
6. Some databases (like MySQL in ONLY_FULL_GROUP_BY mode) require all selected columns to be in the GROUP BY clause or be part of an aggregate function.
7. Be cautious with floating-point comparisons in HAVING clauses due to potential precision issues.
8. Complex grouping operations might be more efficiently handled using raw SQL queries.
9. When using `COUNT(DISTINCT column)`, you need to use `sequelize.fn('COUNT', sequelize.fn('DISTINCT', sequelize.col('column')))`.
10. Remember that `NULL` values form their own group in SQL GROUP BY operations.

## Database-Specific Considerations

### MySQL
- In strict mode (ONLY_FULL_GROUP_BY), every non-aggregated column in the SELECT list must be in the GROUP BY clause.

### PostgreSQL
- Supports GROUPING SETS, CUBE, and ROLLUP for more complex grouping operations.

### SQLite
- Has limited support for complex GROUP BY operations compared to other databases.

Remember to always test your grouped queries with representative data volumes to ensure they perform as expected, especially when combining grouping with other complex query operations.