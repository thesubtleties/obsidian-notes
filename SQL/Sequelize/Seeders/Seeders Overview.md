## Data Manipulation in Seeders: Model Methods vs QueryInterface

### [[Adding (Model vs QueryInterface)|Inserting Data]]

#### Model.bulkCreate
- Uses Sequelize models
- Runs validations and hooks
- Slower but ensures data integrity

```javascript
await User.bulkCreate([
  { username: 'john_doe', email: 'john@example.com' },
  { username: 'jane_doe', email: 'jane@example.com' }
], { validate: true });
```

#### queryInterface.bulkInsert
- Operates directly on database
- Bypasses model logic
- Faster for large datasets

```javascript
await queryInterface.bulkInsert('Users', [
  { username: 'john_doe', email: 'john@example.com', createdAt: new Date(), updatedAt: new Date() },
  { username: 'jane_doe', email: 'jane@example.com', createdAt: new Date(), updatedAt: new Date() }
]);
```

### [[Deleting (Model vs QueryInterface)|Deleting Data]]

#### Model.destroy
- Triggers hooks and respects associations
- Performs soft deletes on paranoid models

```javascript
await User.destroy({
  where: { username: ['john_doe', 'jane_doe'] }
});
```

#### queryInterface.bulkDelete
- Direct database operation
- Always performs hard deletes
- Bypasses model logic

```javascript
await queryInterface.bulkDelete('Users', {
  username: ['john_doe', 'jane_doe']
});
```

Choose method based on need for model integration vs. raw performance. Consider data volume, validation requirements, and consistency with application logic when deciding.

## Basic Structure of a Seeder File

```javascript
// seeders/YYYYMMDDHHMMSS-seed-users.js
'use strict';

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert('Users', [
      {
        name: 'John Doe',
        email: 'john@example.com',
        createdAt: new Date(),
        updatedAt: new Date()
      },
      // More records...
    ], {});
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete('Users', null, {});
  }
};
```

## Creating a Seeder

```bash
npx sequelize-cli seed:generate --name seed-users
```

## Running Seeders

```bash
# Run all seeders
npx sequelize-cli db:seed:all

# Run a specific seeder
npx sequelize-cli db:seed --seed YYYYMMDDHHMMSS-seed-users.js

# Undo the most recent seeder
npx sequelize-cli db:seed:undo

# Undo a specific seeder
npx sequelize-cli db:seed:undo --seed YYYYMMDDHHMMSS-seed-users.js

# Undo all seeders
npx sequelize-cli db:seed:undo:all
```

## Using Faker for Realistic Data

```javascript
const faker = require('faker');

module.exports = {
  up: async (queryInterface, Sequelize) => {
    const users = [...Array(100)].map(() => ({
      name: faker.name.findName(),
      email: faker.internet.email(),
      createdAt: new Date(),
      updatedAt: new Date()
    }));

    await queryInterface.bulkInsert('Users', users, {});
  },
  // ...
};
```

## Seeding with Associations

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    const users = await queryInterface.sequelize.query(
      `SELECT id from Users;`
    );

    const userRows = users[0];

    await queryInterface.bulkInsert('Posts', [
      {
        title: 'First Post',
        content: 'Hello World',
        userId: userRows[0].id,
        createdAt: new Date(),
        updatedAt: new Date()
      },
      // More posts...
    ]);
  },
  // ...
};
```
Certainly! Here's a section to add to your Seeders Overview about seeding join tables:

## [[Seeding Join Tables]]

When seeding join tables, consider these key points:

1. Ensure referenced entities exist before seeding the join table.
2. Use `bulkInsert` for efficiency when adding multiple records.
3. Include `createdAt` and `updatedAt` if your model uses timestamps.
4. Consider [[Seeding by Reference]] (e.g., names) instead of IDs for better readability and maintainability.
5. Use transactions to maintain data integrity across related tables.

Example:
```javascript
await queryInterface.bulkInsert('UserRoles', [
  { userId: 1, roleId: 2, createdAt: new Date(), updatedAt: new Date() },
  { userId: 2, roleId: 1, createdAt: new Date(), updatedAt: new Date() },
], {});
```


## Using Model Files in Seeders

```javascript
const { User } = require('../models');

module.exports = {
  up: async (queryInterface, Sequelize) => {
    await User.bulkCreate([
      { name: 'John', email: 'john@example.com' },
      { name: 'Jane', email: 'jane@example.com' }
    ]);
  },
  // ...
};
```

## Conditional Seeding

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    const [results, metadata] = await queryInterface.sequelize.query(
      "SELECT * FROM Users WHERE email = 'admin@example.com';"
    );

    if (results.length === 0) {
      await queryInterface.bulkInsert('Users', [{
        name: 'Admin',
        email: 'admin@example.com',
        createdAt: new Date(),
        updatedAt: new Date()
      }]);
    }
  },
  // ...
};
```

## Seeding Large Datasets

```javascript
const chunkSize = 1000;
const totalRecords = 1000000;

module.exports = {
  up: async (queryInterface, Sequelize) => {
    for (let i = 0; i < totalRecords; i += chunkSize) {
      const users = [...Array(chunkSize)].map(() => ({
        name: faker.name.findName(),
        email: faker.internet.email(),
        createdAt: new Date(),
        updatedAt: new Date()
      }));

      await queryInterface.bulkInsert('Users', users, {});
    }
  },
  // ...
};
```

## Key Points

1. Seeders are used to populate your database with initial or test data.
2. The `up` function defines what should be inserted into the database.
3. The `down` function should undo what `up` did (usually by deleting the inserted data).
4. Use `bulkInsert` for efficient insertion of multiple records.
5. Seeders are run in alphabetical order of their filenames.
6. Be cautious with production databases; seeders can potentially overwrite existing data.
7. Use transactions in seeders to ensure data integrity.
8. Consider using libraries like Faker.js for generating realistic test data.
9. For large datasets, seed in chunks to avoid memory issues.
10. Seeders can use raw SQL queries for complex operations not easily done with Sequelize methods.

Remember, while seeders are great for development and testing, be cautious when using them in production environments. Always backup your data before running seeders on production databases.