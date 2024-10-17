
Example structure:
```
src/
  components/
    LoginFormPage/
      LoginForm.jsx
      LoginForm.css
      index.jsx
```

In `LoginFormPage/index.jsx`:
```jsx
import LoginForm from './LoginForm';

export default LoginForm;
```

Now, when importing elsewhere in your app, you can do:
```jsx
import LoginForm from './components/LoginFormPage';
```

Instead of:
```jsx
import LoginForm from './components/LoginFormPage/LoginForm';
```

Additional tips (JSX version):

1. For multiple exports:

   ```jsx
   export { default as LoginForm } from './LoginForm';
   export { default as LoginHeader } from './LoginHeader';
   ```

2. If you have the main component logic in `index.jsx`:

   ```jsx
   import React from 'react';
   import LoginHeader from './LoginHeader';
   import LoginFields from './LoginFields';
   import LoginButton from './LoginButton';

   function LoginForm() {
     return (
       <div className="login-form">
         <LoginHeader />
         <LoginFields />
         <LoginButton />
       </div>
     );
   }

   export default LoginForm;
   ```

3. This pattern works well with feature-based structures:

   ```
   src/
     features/
       auth/
         components/
           LoginForm.jsx
           SignupForm.jsx
         hooks/
           useAuth.js
         index.jsx
   ```

   In `features/auth/index.jsx`:
   ```jsx
   export { default as LoginForm } from './components/LoginForm';
   export { default as SignupForm } from './components/SignupForm';
   export { useAuth } from './hooks/useAuth';
   ```

Remember:
- Consistency is key. If you use `.jsx` for React components, stick to it throughout your project.
- Modern build tools often don't require different extensions for JSX files, but using `.jsx` can improve clarity and IDE support.
- Always import React in files using JSX for versions of React before 17.
- Some teams prefer keeping the main component name the same as the folder (e.g., `LoginFormPage.jsx` instead of `LoginForm.jsx`) for clarity.

This approach with JSX extensions emphasizes that these are React component files, which can be helpful in larger projects or teams where not all JavaScript files contain React code.