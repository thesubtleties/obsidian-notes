
```javascript
// File: hooks/userHooks.js

const { User } = require('../models');  // Import the User model
const bcrypt = require('bcrypt');  // If you need external libraries

// Define your hooks
const beforeCreateHook = async (user, options) => {
  if (user.password) {
    user.password = await bcrypt.hash(user.password, 10);
  }
};

const afterCreateHook = (user, options) => {
  console.log(`New user created: ${user.email}`);
};

// Attach hooks to the model
const attachHooks = () => {
  User.addHook('beforeCreate', beforeCreateHook);
  User.addHook('afterCreate', afterCreateHook);

  // You can also use the direct method
  User.beforeUpdate(async (user, options) => {
    if (user.changed('password')) {
      user.password = await bcrypt.hash(user.password, 10);
    }
  });
};

module.exports = { attachHooks };
```

Then, in your main application file or where you initialize your models, you would import and call the `attachHooks` function:

```javascript
// File: app.js or models/index.js

const { attachHooks } = require('./hooks/userHooks');

// After initializing your models
attachHooks();
```

Key points:

1. Import the model(s) you need at the top of the hooks file.
2. Define your hook functions.
3. Create a function (like `attachHooks`) that attaches these hooks to the model.
4. Export this function to be called after your models are initialized.

This approach allows you to:
- Keep your hooks organized in separate files.
- Reuse hooks across different parts of your application if needed.
- Easily manage and update hooks without modifying the main model file.

Remember, the exact structure might vary based on your project's needs and how you've set up your Sequelize models. Ensure that you're importing the correct model and that the hooks are attached after the models are fully initialized.