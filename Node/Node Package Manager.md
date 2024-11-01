## 🚀 Installation & Setup
```bash
# Install Node.js (includes npm)
node -v    # Check Node version
npm -v     # Check npm version

# Update npm itself
npm install -g npm@latest
```

## 📦 Package Management
```bash
# Initialize a new project
npm init           # Interactive
npm init -y        # Accept defaults

# Installing packages
npm install package-name          # Local install
npm i package-name               # Shorthand
npm install -g package-name      # Global install
npm install package-name@1.2.3   # Specific version
npm install package-name --save-dev  # Dev dependency

# Uninstalling packages
npm uninstall package-name
npm remove package-name          # Alternative
npm uninstall -g package-name    # Global uninstall

# Update packages
npm update                # Update all packages
npm update package-name   # Update specific package
```

## 📝 package.json Management
```json
{
  "dependencies": {
    "package": "^1.0.0",     // Major.Minor.Patch
    "package": "~1.0.0",     // Only patch updates
    "package": "1.0.0",      // Exact version
    "package": "*"           // Latest version
  }
}
```

## 🛠️ Scripts
```json
{
  "scripts": {
    "start": "node index.js",
    "test": "jest",
    "custom": "echo 'custom script'"
  }
}
```
```bash
npm run script-name    # Run custom script
npm start             # Run start script
npm test              # Run test script
```

## 📊 Dependency Management
```bash
# List installed packages
npm list              # Local packages
npm list -g           # Global packages
npm list --depth=0    # Direct dependencies only

# Check outdated packages
npm outdated

# Check for security vulnerabilities
npm audit
npm audit fix         # Auto-fix vulnerabilities
```

## 🔄 Version Control
```bash
# Lock dependencies
npm shrinkwrap              # Create npm-shrinkwrap.json

# Clean install (from package.json)
npm ci                      # Faster, stricter than npm install
```

## 💻 Common Commands
```bash
npm search keyword          # Search packages
npm info package-name      # View package info
npm repo package-name      # Open package repository
npm docs package-name      # Open package documentation
```

## ⚠️ Key Points & Gotchas
- `package-lock.json` tracks exact versions - don't ignore it!
- `npm install` vs `npm ci`:
  - `install` updates package-lock.json
  - `ci` requires package-lock.json and is stricter
- Dependencies vs DevDependencies:
  - `dependencies`: Required in production
  - `devDependencies`: Only for development/testing
- Version numbers:
  - `^`: Updates minor and patch
  - `~`: Updates only patch
  - Exact version: No updates

## 🔒 Security Best Practices
```bash
# Regular security checks
npm audit
npm audit fix
npm audit fix --force    # Warning: May break things!

# Update all packages to latest secure versions
npm update
```

## 📁 Cache Management
```bash
npm cache verify     # Verify cache
npm cache clean     # Clean cache
npm cache clean -f  # Force clean
```

## 🌟 Pro Tips
- Use `npm ci` in CI/CD pipelines for consistent builds
- Keep `package-lock.json` in version control
- Use `--save-exact` for critical dependencies
- Run `npm audit` regularly
- Use `.npmrc` for team-wide configurations
- Consider using `npm workspaces` for monorepos

Remember: Watch out for the gotcha where `npm install` might update your package-lock.json unexpectedly. Use `npm ci` for reproducible builds!