Assuming we have these models:
- `User` has many `Post`
- `Post` belongs to `User`
- `User` belongs to many `Role` through `UserRole`

## Get Associated

### getAssociation

```javascript
// Get all posts for a user
const user = await User.findByPk(1);
const posts = await user.getPosts();

// With options
const publishedPosts = await user.getPosts({ where: { status: 'published' } });
```

### getAssociations (for Many-to-Many)

```javascript
// Get all roles for a user
const user = await User.findByPk(1);
const roles = await user.getRoles();
```

## Set Associated

### setAssociation

```javascript
// Set a new user for a post
const post = await Post.findByPk(1);
const newUser = await User.findByPk(2);
await post.setUser(newUser);
```

### setAssociations

```javascript
// Set new roles for a user
const user = await User.findByPk(1);
const newRoles = await Role.findAll({ where: { name: ['admin', 'editor'] } });
await user.setRoles(newRoles);
```

## Add Associated

### addAssociation

```javascript
// Add a new post to a user
const user = await User.findByPk(1);
const newPost = await Post.create({ title: 'New Post' });
await user.addPost(newPost);
```

### addAssociations

```javascript
// Add multiple roles to a user
const user = await User.findByPk(1);
const newRoles = await Role.findAll({ where: { name: ['moderator', 'contributor'] } });
await user.addRoles(newRoles);
```

## Remove Associated

### removeAssociation

```javascript
// Remove a specific post from a user
const user = await User.findByPk(1);
const postToRemove = await Post.findByPk(5);
await user.removePost(postToRemove);
```

### removeAssociations

```javascript
// Remove multiple roles from a user
const user = await User.findByPk(1);
const rolesToRemove = await Role.findAll({ where: { name: ['editor', 'contributor'] } });
await user.removeRoles(rolesToRemove);
```

## Create Associated

### createAssociation

```javascript
// Create a new post for a user
const user = await User.findByPk(1);
const newPost = await user.createPost({ title: 'Brand New Post' });
```

## Check Association

### hasAssociation

```javascript
// Check if a user has a specific post
const user = await User.findByPk(1);
const post = await Post.findByPk(5);
const hasPost = await user.hasPost(post);
```

### hasAssociations

```javascript
// Check if a user has specific roles
const user = await User.findByPk(1);
const roles = await Role.findAll({ where: { name: ['admin', 'editor'] } });
const hasRoles = await user.hasRoles(roles);
```

## Count Associated

### countAssociations

```javascript
// Count posts for a user
const user = await User.findByPk(1);
const postCount = await user.countPosts();

// With conditions
const publishedPostCount = await user.countPosts({ where: { status: 'published' } });
```

## Key Points

1. These methods are automatically added by Sequelize based on your association definitions.
2. Method names are generated using the model name (e.g., `getPosts`, `setRoles`).
3. Singular and plural forms are used appropriately (e.g., `addPost` vs `addPosts`).
4. Many methods accept options like `where` clauses for more specific operations.
5. These methods handle the complexities of managing relationships, including junction tables for many-to-many associations.
6. Always use `await` with these methods as they return promises.

Remember, the exact method names will depend on how you've defined your associations and model names. Sequelize generates these methods dynamically based on your model and association setup.