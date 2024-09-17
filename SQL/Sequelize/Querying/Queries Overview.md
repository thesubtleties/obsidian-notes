# Sequelize Query Cheatsheet

## [[Importing Modules]]

```javascript
const { Sequelize, Model, DataTypes, Op } = require('sequelize');
```

## Basic CRUD Operations

### Create

#### Create a single record
```javascript
const user = await User.create({ name: 'John Doe', email: 'john@example.com' });
```

#### Create multiple records
```javascript
const users = await User.bulkCreate([
  { name: 'Jane Doe', email: 'jane@example.com' },
  { name: 'Jim Beam', email: 'jim@example.com' }
]);
```

### Read

#### Find all records
```javascript
const allUsers = await User.findAll();
```

#### Find one record
```javascript
const user = await User.findOne({ where: { id: 1 } });
```

#### [[FindByPk|Find by primary key]]
```javascript
const user = await User.findByPk(1);
```

#### Get specific attributes
```javascript
const users = await User.findAll({
  attributes: ['name', 'email']
});
```

#### Exclude specific attributes
```javascript
const users = await User.findAll({
  attributes: { exclude: ['createdAt', 'updatedAt'] }
});
```

#### Count
```javascript
const userCount = await User.count();
```

### [[Update]]

#### Update a single record
```javascript
await User.update({ name: 'Johnny Doe' }, { where: { id: 1 } });
```

#### Update multiple records
```javascript
await User.update(
  { status: 'inactive' }, 
  { where: { lastLoginDate: { [Op.lt]: new Date() } } }
);
```

### Delete

#### Delete a single record
```javascript
await User.destroy({ where: { id: 1 } });
```

#### Delete multiple records
```javascript
await User.destroy({ where: { status: 'inactive' } });
```

## Advanced Querying

### Where Clauses

#### Basic where
```javascript
const users = await User.findAll({
  where: { status: 'active' }
});
```

#### Multiple conditions (AND)
```javascript
const users = await User.findAll({
  where: {
    status: 'active',
    age: { [Op.gte]: 18 }
  }
});
```

#### OR condition
```javascript
const users = await User.findAll({
  where: {
    [Op.or]: [
      { status: 'active' },
      { age: { [Op.gte]: 18 } }
    ]
  }
});
```

### [[Common Sequelize Operators]]
Follow above link for more.

```javascript
// Equal
{ status: 'active' }

// Not equal
{ status: { [Op.ne]: 'inactive' } }

// Greater than
{ age: { [Op.gt]: 18 } }

// Greater than or equal
{ age: { [Op.gte]: 18 } }

// Less than
{ age: { [Op.lt]: 65 } }

// Less than or equal
{ age: { [Op.lte]: 65 } }

// BETWEEN
{ age: { [Op.between]: [18, 65] } }

// IN
{ status: { [Op.in]: ['active', 'pending'] } }

// NOT IN
{ status: { [Op.notIn]: ['banned', 'deleted'] } }

// LIKE
{ name: { [Op.like]: 'John%' } }

// NOT LIKE
{ name: { [Op.notLike]: 'John%' } }

// IS NULL
{ deletedAt: { [Op.is]: null } }

// IS NOT NULL
{ deletedAt: { [Op.not]: null } }
```

### [[Order]] and Limit

#### Order
```javascript
const users = await User.findAll({
  order: [
    ['name', 'ASC'],
    ['age', 'DESC']
  ]
});
```

#### Limit and Offset ([[Pagination]])
```javascript
const users = await User.findAll({
  limit: 10,
  offset: 0  // Skip 0 records (first page)
});
```

### [[Grouping]]
```javascript
const result = await User.findAll({
  attributes: ['status', [Sequelize.fn('COUNT', Sequelize.col('id')), 'count']],
  group: ['status']
});
```

## Associations

Associations define relationships between models, allowing you to work with related data more easily.

### Types of Associations:
1. One-to-One
2. One-to-Many
3. Many-to-Many

### When to use:
- When you need to represent relationships between different entities in your database.
- To perform joins and eager loading of related data efficiently.

### Example:
```javascript
// One-to-Many association
User.hasMany(Post);
Post.belongsTo(User);

// Usage
const users = await User.findAll({
  include: [{ model: Post }]
});
```

This example sets up a one-to-many relationship between User and Post models, where one user can have many posts. The `User.hasMany(Post)` indicates that the User model has multiple associated Post records, while `Post.belongsTo(User)` specifies that each Post belongs to a single User. In the usage part, `User.findAll()` retrieves all users and includes their associated posts in a single query. This is known as eager loading, where related data (posts) is loaded alongside the main data (users) in one database operation, reducing the need for separate queries.
### Nested Associations:
```javascript
const users = await User.findAll({
  include: [{
    model: Post,
    include: [{ model: Comment }]
  }]
});
```

This example demonstrates nested associations, where we're not only including posts for each user but also comments for each post. It sets up a chain of relationships: User -> Post -> Comment. The query will fetch all users, along with their associated posts and the comments on those posts, all in a single database operation. This nested include allows for efficient retrieval of deeply related data, enabling you to access a user's posts and the comments on those posts without additional queries. It's particularly useful when you need to display complex, hierarchical data structures in your application.
### [[Transactions]]

Transactions ensure that a series of database operations are executed as a single unit of work, maintaining data integrity.

### When to use:
- When multiple database operations need to succeed or fail together.
- To prevent partial updates in case of errors.

### Example:
```javascript
const t = await sequelize.transaction();

try {
  const user = await User.create({ name: 'John' }, { transaction: t });
  await user.addRole(roleId, { transaction: t });
  await t.commit();
} catch (error) {
  await t.rollback();
  console.error('Transaction failed:', error);
}
```

## Raw Queries

Raw queries allow you to execute custom SQL when Sequelize's query builder doesn't meet your needs.

### When to use:
- For complex queries that are difficult to express using Sequelize's ORM methods.
- When you need to optimize performance for specific database operations.

### Example:
```javascript
const [results, metadata] = await sequelize.query(
  "SELECT * FROM users WHERE status = :status",
  {
    replacements: { status: 'active' },
    type: QueryTypes.SELECT
  }
);
```

## [[Scope]]

Scopes in Sequelize are powerful tools for creating reusable query logic, particularly useful when you frequently need to apply the same conditions or includes across multiple queries. They're ideal for common filtering scenarios (like 'active' users), standard includes (like 'with associated posts'), or complex query conditions that you want to encapsulate. However, overuse of scopes can lead to reduced code clarity and potential performance issues. Avoid using scopes for one-off queries or highly specific conditions that aren't reused. Also, be cautious with default scopes, as they can lead to unexpected query results if forgotten. Instead of complex, nested scopes, consider using separate query methods or service functions for more maintainable and explicit code.

### When to use:
- To encapsulate query logic for frequently used filters or conditions.
- To maintain consistent query conditions across different parts of your application.

### Example:
```javascript
// In model definition
class User extends Model {}
User.init({
  // ... attributes
}, {
  defaultScope: {
    where: { active: true }
  },
  scopes: {
    admin: {
      where: { role: 'admin' }
    },
    withPosts: {
      include: [{ model: Post }]
    }
  },
  sequelize
});

// Usage
const activeUsers = await User.findAll();  // Uses default scope
const adminUsers = await User.scope('admin').findAll();
const usersWithPosts = await User.scope('withPosts').findAll();
```

## Hooks (Lifecycle Events)

Hooks are functions that are called before or after specific events in Sequelize's lifecycle, allowing you to automate certain actions.

### When to use:
- To perform automatic data transformations (e.g., hashing passwords before save).
- To trigger side effects (e.g., sending notifications after record creation).
- For validation or data normalization before database operations.

### Example:
```javascript
User.beforeCreate(async (user, options) => {
  user.password = await bcrypt.hash(user.password, 10);
});

User.afterCreate(async (user, options) => {
  await sendWelcomeEmail(user.email);
});
```

## Attributes and Virtual Fields

Attributes define the structure of your model, while virtual fields allow you to define properties that don't exist in your database but can be computed from other fields.

### When to use Virtual Fields:
- To compute values on the fly without storing them in the database.
- To provide a convenient way to access derived data.

### Example:
```javascript
class User extends Model {}
User.init({
  firstName: DataTypes.STRING,
  lastName: DataTypes.STRING,
  fullName: {
    type: DataTypes.VIRTUAL,
    get() {
      return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
      throw new Error('Do not try to set the `fullName` value!');
    }
  }
}, { sequelize });

// Usage
const user = await User.create({ firstName: 'John', lastName: 'Doe' });
console.log(user.fullName);  // 'John Doe'
```


Certainly! Here's a section to add to your Queries Overview on SQL/Express Efficiency:

## SQL/Express Efficiency

Optimizing database queries and Express.js operations is crucial for application performance. Key strategies include:

1. Refactoring expensive calculations to use SQL over JavaScript
2. Eliminating N+1 queries through proper eager loading
3. Creating appropriate indexes on frequently queried columns
4. Using query caching for repetitive requests
5. Implementing pagination for large datasets
6. Optimizing JOIN operations and subqueries
7. Utilizing database-specific features for complex operations

Regularly profiling and analyzing query performance helps identify bottlenecks and areas for improvement.
## Key Points

1. Associations help model relationships between entities and optimize data retrieval.
2. Transactions ensure data integrity for multiple related operations.
3. Raw queries provide flexibility for complex or performance-critical operations.
4. Scopes allow for reusable, consistent query conditions across your application.
5. Hooks automate actions at specific points in the model lifecycle.
6. Virtual fields compute values on-the-fly, reducing database storage needs.
7. Always use `await` with asynchronous Sequelize operations.
8. Use `Op` for complex where clauses and comparisons.

Remember: Proper use of these features can significantly improve your application's data management, performance, and maintainability. Always consider the specific needs of your application when deciding which features to use.