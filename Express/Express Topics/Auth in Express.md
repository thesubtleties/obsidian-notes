## JWT (JSON Web Tokens)

### Installation
```bash
npm install jsonwebtoken
```

### Generating a Token
```javascript
const jwt = require('jsonwebtoken');

const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '1h' }
);
```

### Verifying a Token
```javascript
const verifyToken = (req, res, next) => {
  const token = req.headers['authorization']?.split(' ')[1];
  
  if (!token) {
    return res.status(403).json({ message: 'No token provided' });
  }
  
  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) {
      return res.status(401).json({ message: 'Invalid token' });
    }
    req.userId = decoded.userId;
    next();
  });
};
```

### Using JWT Middleware
```javascript
app.get('/api/protected', verifyToken, (req, res) => {
  res.json({ message: 'Access granted', userId: req.userId });
});
```

## Session-based Authentication

### Installation
```bash
npm install express-session
```

### Setup
```javascript
const session = require('express-session');

app.use(session({
  secret: 'your_secret_key',
  resave: false,
  saveUninitialized: true,
  cookie: { secure: true, maxAge: 60000 }
}));
```

### Login
```javascript
app.post('/login', (req, res) => {
  // Verify credentials
  req.session.userId = user.id;
  res.json({ message: 'Logged in successfully' });
});
```

### Logout
```javascript
app.get('/logout', (req, res) => {
  req.session.destroy(err => {
    if (err) {
      return res.status(500).json({ message: 'Could not log out' });
    }
    res.json({ message: 'Logged out successfully' });
  });
});
```

## OAuth 2.0 (with Passport)

### Installation
```bash
npm install passport passport-google-oauth20
```

### Setup
```javascript
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

passport.use(new GoogleStrategy({
    clientID: GOOGLE_CLIENT_ID,
    clientSecret: GOOGLE_CLIENT_SECRET,
    callbackURL: "http://www.example.com/auth/google/callback"
  },
  (accessToken, refreshToken, profile, cb) => {
    // Handle user data
    cb(null, profile);
  }
));

app.use(passport.initialize());
```

### Routes
```javascript
app.get('/auth/google',
  passport.authenticate('google', { scope: ['profile', 'email'] }));

app.get('/auth/google/callback', 
  passport.authenticate('google', { failureRedirect: '/login' }),
  (req, res) => {
    res.redirect('/dashboard');
  });
```

## Password Hashing (with bcrypt)

### Installation
```bash
npm install bcrypt
```

### Hashing a Password
```javascript
const bcrypt = require('bcrypt');

const saltRounds = 10;
const hashedPassword = await bcrypt.hash(password, saltRounds);
```

### Verifying a Password
```javascript
const isMatch = await bcrypt.compare(password, hashedPassword);
```

## Best Practices

1. Always use HTTPS in production.
2. Store secrets and keys in environment variables.
3. Implement rate limiting to prevent brute-force attacks.
4. Use secure HTTP-only cookies for storing session IDs.
5. Implement proper error handling and logging.
6. Regularly update dependencies to patch security vulnerabilities.
7. Implement multi-factor authentication for enhanced security.
8. Use CSRF protection for session-based authentication.

## Additional Security Measures

1. Implement password strength requirements.
2. Use secure password reset mechanisms.
3. Implement account lockout after multiple failed attempts.
4. Provide options for session management (view active sessions, logout from all devices).
5. Use security headers (Helmet middleware for Express).

Remember, authentication is a critical component of web application security. Always stay updated with the latest security best practices and consider regular security audits.