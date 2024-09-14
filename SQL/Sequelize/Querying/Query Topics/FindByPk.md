Ah, `findByPk` - that's a good one to know. Let's break it down:

`findByPk` is a method provided by Sequelize, one of the most popular Object-Relational Mapping (ORM) libraries for Node.js. It stands for "find by primary key".

Here's the lowdown:

1. Purpose:
   It's used to retrieve a single record from the database based on its primary key.

2. Syntax:
   ```javascript
   Model.findByPk(id)
   ```

3. Returns:
   It returns a Promise that resolves to the found instance or null if nothing was found.

4. Example:
   ```javascript
   const puppy = await Puppy.findByPk(5);
   if (puppy) {
     console.log(puppy.name);
   } else {
     console.log('No puppy found with id 5');
   }
   ```

5. Under the hood:
   It's essentially a shorthand for:
   ```javascript
   Model.findOne({ where: { id: id } })
   ```

Now, watch out for these gotchas:

- It assumes your primary key is named 'id'. If it's not, you might run into issues.
- It only works for single primary keys. If you have a composite primary key, you'll need to use `findOne` instead.
- Remember, it's an async operation, so always use `await` or handle the promise properly.

When to use it:
- When you're fetching a record and you know its ID.
- In RESTful APIs, often in GET routes like `/puppies/:id`.

When not to use it:
- When you need to fetch based on conditions other than the primary key.
- When you need to fetch multiple records.

Here's a real-world example in an Express route:

```javascript
app.get('/puppies/:id', async (req, res, next) => {
  try {
    const puppy = await Puppy.findByPk(req.params.id);
    if (puppy) {
      res.json(puppy);
    } else {
      res.status(404).json({ message: 'Puppy not found' });
    }
  } catch (error) {
    next(error);
  }
});
```

Remember, `findByPk` is just one tool in your Sequelize toolkit. Knowing when to use it (and when not to) is part of mastering database operations in your Node.js applications.