## Types of Associations

1. One-To-One
2. One-To-Many
3. Many-To-Many

## Defining Associations

### One-To-One

```javascript
// In User model
User.hasOne(Profile);

// In Profile model
Profile.belongsTo(User);
```

### One-To-Many

```javascript
// In User model
User.hasMany(Post);

// In Post model
Post.belongsTo(User);
```

### Many-To-Many

```javascript
// In User model
User.belongsToMany(Role, { through: 'UserRoles' });

// In Role model
Role.belongsToMany(User, { through: 'UserRoles' });
```

## Association Options

```javascript
User.hasMany(Post, {
  foreignKey: 'authorId', // Custom foreign key
  as: 'articles', // Alias for the association
  onDelete: 'CASCADE', // What happens when parent is deleted
  onUpdate: 'CASCADE', // What happens when parent is updated
  hooks: true, // Trigger hooks for this association
  constraints: false // Disable foreign key constraint
});
```

## Customizing the Junction Table (Many-To-Many)

```javascript
User.belongsToMany(Project, {
  through: {
    model: UserProject,
    unique: false
  },
  constraints: false
});
```

## Eager Loading

```javascript
const users = await User.findAll({
  include: [
    {
      model: Post,
      as: 'articles',
      where: { status: 'published' }
    },
    {
      model: Profile
    }
  ]
});
```

## Lazy Loading

```javascript
const user = await User.findOne({ where: { id: 1 } });
const posts = await user.getPosts();
```

## Creating Associated Data

```javascript
const user = await User.create({ name: 'John' });
const post = await user.createPost({ title: 'Hello World' });
```

## [[Association Methods]]
Assuming a User hasMany Posts:

- `user.getPosts()`
- `user.setPosts()`
- `user.addPost()`
- `user.addPosts()`
- `user.removePost()`
- `user.removePosts()`
- `user.hasPost()`
- `user.hasPosts()`
- `user.countPosts()`

## [[Scopes with Associations]]

```javascript
// In Post model
static associate(models) {
  Post.belongsTo(models.User, {
    foreignKey: 'userId',
    as: 'author'
  });
}

// Usage
const posts = await Post.scope('withAuthor').findAll();
```
Scopes with associations in Sequelize allow you to define reusable query conditions that include related models. This approach encapsulates inclusion logic within the model definition, promoting code reusability and cleaner query syntax. Unlike manual includes, scopes can be easily combined with other query conditions and scopes, offering a flexible way to structure complex queries involving associations, though they're less dynamic than specifying includes directly in each query.
## Key Points

1. Associations are defined in the model files, not in migrations.
2. The order of defining associations matters for some types (e.g., `hasOne` and `belongsTo`).
3. Always define both sides of an association for full functionality.
4. Use aliases (`as`) to give meaningful names to associations, especially when you have multiple associations between the same models.
5. Eager loading can significantly reduce the number of queries needed.

## Associations in Models vs. [[Foreign Keys|Migrations]]

- **Model Associations:**
  - Define the relationship between models in JavaScript.
  - Provide methods for interacting with related data (e.g., `getAssociated()`, `setAssociated()`).
  - Allow for easy querying of related data.
  - Are used by Sequelize ORM to understand relationships between tables.

- **Migration Associations:**
  - Define the actual database structure and constraints.
  - Create foreign key columns and constraints in the database.
  - Don't provide model-level methods for interacting with data.
  - Are necessary for maintaining database integrity at the SQL level.

Best practice is to define associations in both places:
- Use migrations to set up the database structure correctly.
- Use model associations to define how Sequelize should interact with these relationships in your application code.

Remember, model associations define how Sequelize interacts with your data, while migration associations define the actual database structure. Both are important for a fully functional and consistent database design.