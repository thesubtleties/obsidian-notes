
### Transactions in Sequelize

Transactions in Sequelize allow you to perform a series of database operations as a single unit of work. This ensures data integrity by guaranteeing that either all operations within the transaction succeed, or none of them do. If any operation within the transaction fails, all changes are rolled back, leaving the database in its original state.

Here's a breakdown of how transactions work in Sequelize:

1. Starting a Transaction:
   - Use `sequelize.transaction()` to begin a new transaction.
   - This returns a transaction object (often named `t`) that you'll use for all operations within the transaction.

2. Performing Operations:
   - For each database operation (create, update, delete) within the transaction, pass the transaction object as an option.
   - This ties the operation to the transaction, preventing it from being immediately committed to the database.

3. Committing the Transaction:
   - If all operations are successful, call `await t.commit()` to finalize the changes and write them to the database.

4. Rolling Back on Error:
   - If any error occurs during the transaction, call `await t.rollback()` to undo all operations performed within the transaction.

5. Error Handling:
   - Use a try-catch block to handle potential errors and ensure proper rollback if needed.

### Key Points

- Transactions are particularly useful for operations that depend on each other, where partial completion would lead to data inconsistency.
- They're commonly used in scenarios like transferring money between accounts, where you need to ensure that both the debit and credit operations succeed or fail together.
- Transactions can span multiple models and even raw SQL queries, as long as they're all performed using the same transaction object.
- Remember that transactions lock the affected rows in the database, so they should be kept as short as possible to avoid performance issues in high-concurrency scenarios.

##### Example:

Imagine you're creating a user and immediately assigning them a role. You want to ensure that either both operations succeed or neither does. Here's how you'd use a transaction for this:

```javascript
const t = await sequelize.transaction();

try {
  // Create a user
  const user = await User.create({ 
    name: 'John Doe', 
    email: 'john@example.com' 
  }, { transaction: t });

  // Assign a role to the user
  await user.addRole(roleId, { transaction: t });

  // If we reach here, both operations were successful.
  // Commit the transaction to save the changes:
  await t.commit();

  console.log('User created and role assigned successfully!');
} catch (error) {
  // If any error occurred, rollback the transaction:
  await t.rollback();
  console.error('Error occurred, transaction rolled back:', error);
  // Here you might want to handle the error (e.g., send an error response)
}
```

In this example, if creating the user succeeds but assigning the role fails, the `t.rollback()` call will undo the user creation, maintaining data consistency. Only if both operations succeed will the changes be committed to the database.
