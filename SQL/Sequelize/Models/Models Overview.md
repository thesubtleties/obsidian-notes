## Model Definition

```javascript
// File: models/user.js
const { Model, DataTypes } = require('sequelize');

class User extends Model {
  static init(sequelize) {
    super.init({
      // Attributes
      username: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true
      },
      email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: {
          isEmail: true
        }
      },
      password: {
        type: DataTypes.STRING,
        allowNull: false
      },
      isAdmin: {
        type: DataTypes.BOOLEAN,
        defaultValue: false
      }
    }, {
      sequelize,
      modelName: 'User',
      tableName: 'users', // Optional: explicitly set table name
      timestamps: true,   // Adds createdAt and updatedAt
      paranoid: false,    // Soft deletes
      underscored: true   // Use snake_case for auto-generated fields
    });
  }

  // Define associations
  static associate(models) {
    User.hasMany(models.Post, { foreignKey: 'userId' });
    User.belongsToMany(models.Role, { through: 'UserRoles' });
  }
}

module.exports = User;
```

## Data Types

Common Sequelize data types:
- `DataTypes.STRING`
- `DataTypes.TEXT`
- `DataTypes.INTEGER`
- `DataTypes.FLOAT`
- `DataTypes.BOOLEAN`
- `DataTypes.DATE`
- `DataTypes.ENUM('value1', 'value2')`
- `DataTypes.JSON`
- `DataTypes.ARRAY(DataTypes.INTEGER)`

## [[Model creation, constraints, validations, hooks|Validations]]

```javascript
email: {
  type: DataTypes.STRING,
  validate: {
    isEmail: true,
    notEmpty: true,
    len: [3, 255]
  }
}
```

## [[Getters and Setters|Getters and Setters]]

```javascript
fullName: {
  type: DataTypes.VIRTUAL,
  get() {
    return `${this.firstName} ${this.lastName}`;
  },
  set(value) {
    const [firstName, lastName] = value.split(' ');
    this.setDataValue('firstName', firstName);
    this.setDataValue('lastName', lastName);
  }
}
```

## [[Hooks (Lifecycle Events)]]

```javascript
// In model definition
hooks: {
  beforeCreate: async (user) => {
    user.password = await bcrypt.hash(user.password, 10);
  }
}

// Or after model definition
User.beforeCreate(async (user) => {
  user.password = await bcrypt.hash(user.password, 10);
});
```

## [[Associations]]

```javascript
// One-to-Many
User.hasMany(Post, { foreignKey: 'userId' });
Post.belongsTo(User, { foreignKey: 'userId' });

// Many-to-Many
User.belongsToMany(Role, { through: 'UserRoles' });
Role.belongsToMany(User, { through: 'UserRoles' });

// One-to-One
User.hasOne(Profile);
Profile.belongsTo(User);
```

## [[Scope]]

```javascript
// In model definition
scopes: {
  active: {
    where: { isActive: true }
  },
  adminOnly: {
    where: { isAdmin: true }
  }
}

// Usage
const activeUsers = await User.scope('active').findAll();
```

## Instance Methods

```javascript
class User extends Model {
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

## Class Methods

```javascript
class User extends Model {
  static async findByEmail(email) {
    return this.findOne({ where: { email } });
  }
}
```

## Model Options

- `timestamps`: Adds `createdAt` and `updatedAt` fields
- `paranoid`: Enables soft deletes (adds `deletedAt`)
- `underscored`: Uses snake_case for auto-generated fields
- `freezeTableName`: Prevents Sequelize from pluralizing table name
- `tableName`: Explicitly sets the table name
- `indexes`: Defines database indexes

## Importing and Using Models

```javascript
// File: models/index.js
const User = require('./user');
const Post = require('./post');

// Initialize models
User.init(sequelize);
Post.init(sequelize);

// Run associations
User.associate({ Post });
Post.associate({ User });

module.exports = { User, Post };

// Usage in other files
const { User, Post } = require('./models');
```

Remember, this cheatsheet provides an overview of key concepts in Sequelize models. The exact implementation may vary based on your project structure and requirements.