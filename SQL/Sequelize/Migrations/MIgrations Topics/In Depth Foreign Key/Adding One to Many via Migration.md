Certainly! Here's a step-by-step cheat sheet for creating a one-to-many relationship in Sequelize. Let's assume we're creating a relationship between `User` (one) and `Post` (many).

1. Create migration for the 'many' side (if it doesn't exist):
   ```bash
   npx sequelize-cli migration:generate --name create-posts
   ```

2. Edit the migration file:
   ```javascript
   'use strict';

   module.exports = {
     up: async (queryInterface, Sequelize) => {
       await queryInterface.createTable('Posts', {
         id: {
           allowNull: false,
           autoIncrement: true,
           primaryKey: true,
           type: Sequelize.INTEGER
         },
         title: {
           type: Sequelize.STRING,
           allowNull: false
         },
         content: {
           type: Sequelize.TEXT,
           allowNull: false
         },
         userId: {
           type: Sequelize.INTEGER,
           references: {
             model: 'Users',
             key: 'id',
           },
           onUpdate: 'CASCADE',
           onDelete: 'SET NULL',
         },
         createdAt: {
           allowNull: false,
           type: Sequelize.DATE
         },
         updatedAt: {
           allowNull: false,
           type: Sequelize.DATE
         }
       });
     },

     down: async (queryInterface, Sequelize) => {
       await queryInterface.dropTable('Posts');
     }
   };
   ```

3. Update the 'one' side model (models/user.js):
   ```javascript
   'use strict';
   const { Model } = require('sequelize');

   module.exports = (sequelize, DataTypes) => {
     class User extends Model {
       static associate(models) {
         User.hasMany(models.Post, {
           foreignKey: 'userId',
           as: 'posts'
         });
       }
     }
     User.init({
       // existing fields...
     }, {
       sequelize,
       modelName: 'User',
     });
     return User;
   };
   ```

4. Create or update the 'many' side model (models/post.js):
   ```javascript
   'use strict';
   const { Model } = require('sequelize');

   module.exports = (sequelize, DataTypes) => {
     class Post extends Model {
       static associate(models) {
         Post.belongsTo(models.User, {
           foreignKey: 'userId',
           as: 'author'
         });
       }
     }
     Post.init({
       title: DataTypes.STRING,
       content: DataTypes.TEXT,
       userId: DataTypes.INTEGER
     }, {
       sequelize,
       modelName: 'Post',
     });
     return Post;
   };
   ```

5. Run the migration:
   ```bash
   npx sequelize-cli db:migrate
   ```

Key points to watch out for:

1. Foreign Key: In the 'many' side (Posts), we add a foreign key (`userId`) that references the 'one' side (Users).

2. In the migration, we define the foreign key relationship:
   ```javascript
   userId: {
     type: Sequelize.INTEGER,
     references: {
       model: 'Users',
       key: 'id',
     },
     onUpdate: 'CASCADE',
     onDelete: 'SET NULL',
   },
   ```
   Note: `onDelete: 'SET NULL'` means if a User is deleted, their Posts will remain but with `userId` set to NULL. Adjust this based on your requirements.

3. In the User model, we use `hasMany`:
   ```javascript
   User.hasMany(models.Post, {
     foreignKey: 'userId',
     as: 'posts'
   });
   ```

4. In the Post model, we use `belongsTo`:
   ```javascript
   Post.belongsTo(models.User, {
     foreignKey: 'userId',
     as: 'author'
   });
   ```

5. The `as` option in both models allows you to give a name to the association, which can be used when including related models in queries.

After setting this up, you can use the relationship like this:

```javascript
// Get a user with their posts
const user = await User.findByPk(1, {
  include: 'posts'
});

// Get a post with its author
const post = await Post.findByPk(1, {
  include: 'author'
});

// Create a post for a user
const user = await User.findByPk(1);
const newPost = await user.createPost({
  title: 'New Post',
  content: 'This is the content'
});
```

Remember:
- Always consider the appropriate `onDelete` behavior for your use case.
- The foreign key goes in the 'many' side of the relationship.
- Consistency in naming (e.g., `userId` in both the migration and model) is crucial.
- After creating this relationship, you might need to update your seeders or existing data to maintain referential integrity.

This cheat sheet covers the basics of setting up a one-to-many relationship in Sequelize. As always, adjust the specifics based on your project's needs and existing structure.