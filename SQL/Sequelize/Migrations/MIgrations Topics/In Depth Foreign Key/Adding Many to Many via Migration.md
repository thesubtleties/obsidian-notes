
1. Create the junction table migration:
   ```bash
   npx sequelize-cli migration:generate --name create-user-projects
   ```

2. Edit the new migration file:
   ```javascript
   'use strict';

   module.exports = {
     up: async (queryInterface, Sequelize) => {
       await queryInterface.createTable('UserProjects', {
         userId: {
           type: Sequelize.INTEGER,
           references: {
             model: 'Users',
             key: 'id',
           },
           onUpdate: 'CASCADE',
           onDelete: 'CASCADE',
         },
         projectId: {
           type: Sequelize.INTEGER,
           references: {
             model: 'Projects',
             key: 'id',
           },
           onUpdate: 'CASCADE',
           onDelete: 'CASCADE',
         },
         createdAt: {
           allowNull: false,
           type: Sequelize.DATE,
         },
         updatedAt: {
           allowNull: false,
           type: Sequelize.DATE,
         },
       });
       
       await queryInterface.addConstraint('UserProjects', {
         fields: ['userId', 'projectId'],
         type: 'primary key',
         name: 'user_project_pkey',
       });
     },

     down: async (queryInterface, Sequelize) => {
       await queryInterface.dropTable('UserProjects');
     }
   };
   ```

3. Update the User model (models/user.js):
   ```javascript
   'use strict';
   const { Model } = require('sequelize');

   module.exports = (sequelize, DataTypes) => {
     class User extends Model {
       static associate(models) {
         User.belongsToMany(models.Project, {
           through: 'UserProjects',
           foreignKey: 'userId',
           otherKey: 'projectId',
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

4. Update the Project model (models/project.js):
   ```javascript
   'use strict';
   const { Model } = require('sequelize');

   module.exports = (sequelize, DataTypes) => {
     class Project extends Model {
       static associate(models) {
         Project.belongsToMany(models.User, {
           through: 'UserProjects',
           foreignKey: 'projectId',
           otherKey: 'userId',
         });
       }
     }
     Project.init({
       // existing fields...
     }, {
       sequelize,
       modelName: 'Project',
     });
     return Project;
   };
   ```

5. Run the migration:
   ```bash
   npx sequelize-cli db:migrate
   ```

6. Create a model for the junction table:
   ```javascript
   // models/userproject.js
   'use strict';
   const { Model } = require('sequelize');

   module.exports = (sequelize, DataTypes) => {
     class UserProject extends Model {
       static associate(models) {
         // define association here if needed
       }
     }
     UserProject.init({
       userId: DataTypes.INTEGER,
       projectId: DataTypes.INTEGER
     }, {
       sequelize,
       modelName: 'UserProject',
     });
     return UserProject;
   };
   ```

   Note: While technically optional, creating a model for the junction table is highly recommended and considered best practice. It provides several benefits:
   - Direct access to the association data
   - Ability to add methods or validations to the association
   - Easier querying and manipulation of the relationship data
   - Consistency with Sequelize's model-centric approach
   - Future-proofing for potential additions to the relationship

   Only in very simple, read-only scenarios might you consider omitting this step, but it's generally advised to always create the junction model for consistency and flexibility.

Key points to watch out for:

1. The junction table name is typically a combination of both model names (e.g., 'UserProjects').
2. In the migration, we're creating a composite primary key using both foreign keys.
3. In the models, we use `belongsToMany` to define the many-to-many relationship.
4. The `through` option in `belongsToMany` specifies the name of the junction table.
5. `foreignKey` in each model refers to the key in the junction table that references that model.
6. `otherKey` in each model refers to the key in the junction table that references the other model.

After this, you can use the relationship like:

```javascript
const user = await User.findByPk(1, {
  include: Project
});

const project = await Project.findByPk(1, {
  include: User
});
```

This will give you the associated projects for a user and users for a project, respectively.

Remember, migrations are a one-way street. Once you run them, you need to be careful about making changes. If you mess up, it's often easier to revert the migration, fix it, and run it again rather than trying to patch things up with additional migrations.