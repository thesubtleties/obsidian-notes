Here's an example page that demonstrates how scopes can be combined with other query conditions and scopes, showcasing their flexibility in structuring complex queries with associations:

```javascript
// models/post.js
module.exports = (sequelize, DataTypes) => {
  const Post = sequelize.define('Post', {
    title: DataTypes.STRING,
    content: DataTypes.TEXT,
    status: DataTypes.ENUM('draft', 'published', 'archived')
  });

  Post.associate = function(models) {
    Post.belongsTo(models.User, { as: 'author' });
    Post.belongsToMany(models.Tag, { through: 'PostTags' });
  };

  Post.addScope('published', {
    where: { status: 'published' }
  });

  Post.addScope('withAuthor', {
    include: [{ model: sequelize.models.User, as: 'author' }]
  });

  Post.addScope('withTags', {
    include: [{ model: sequelize.models.Tag }]
  });

  return Post;
};

// Usage in a route or controller
const { Post, Tag } = require('./models');
const { Op } = require('sequelize');

async function getRecentPostsByAuthor(authorId) {
  const posts = await Post.scope(['published', 'withAuthor', 'withTags']).findAll({
    where: {
      authorId: authorId,
      createdAt: { [Op.gte]: new Date(new Date() - 7 * 24 * 60 * 60 * 1000) } // Last 7 days
    },
    order: [['createdAt', 'DESC']],
    limit: 5
  });

  return posts;
}

// Example usage
app.get('/recent-posts/:authorId', async (req, res) => {
  try {
    const posts = await getRecentPostsByAuthor(req.params.authorId);
    res.json(posts);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});
```

This example demonstrates:
1. Defining multiple scopes on the Post model.
2. Combining scopes (`published`, `withAuthor`, `withTags`) in a single query.
3. Adding additional where conditions and query options alongside scopes.
4. Using scopes to create a reusable, complex query that includes associations and specific conditions.

This approach allows for flexible and readable queries that can be easily maintained and modified as needed.