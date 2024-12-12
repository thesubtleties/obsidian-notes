## 1. Project Structure
```bash
your-app/
├── frontend/                
│   ├── src/               
│   ├── vite.config.js     # Dev proxy config
│   └── Dockerfile         
├── backend/               
│   ├── src/              
│   │   ├── index.js       # Express entry
│   │   ├── models/        # Sequelize models
│   │   └── routes/        # Express routes
│   └── Dockerfile        
└── render.yaml            # Render configuration
```
**Key Point:** Simplified structure for Render deployment

## 2. render.yaml
```yaml
services:
  # Frontend Service
  - type: web
    name: myapp-frontend
    env: docker
    buildCommand: docker build -f frontend/Dockerfile -t myapp-frontend ./frontend
    startCommand: serve -s dist -l $PORT
    envVars:
      - key: NODE_ENV
        value: production

  # Backend Service
  - type: web
    name: myapp-backend
    env: docker
    buildCommand: docker build -f backend/Dockerfile -t myapp-backend ./backend
    startCommand: node src/index.js
    envVars:
      - key: NODE_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: myapp-db
          property: connectionString
      - key: JWT_SECRET
        generateValue: true

  # Database
  - type: pgsql
    name: myapp-db
    ipAllowList: []
```
**Key Points:**
- Each service defined separately
- Database URL auto-injected
- Automatic secret generation
- No port mapping needed

## 3. Frontend Dockerfile
```dockerfile
# Build stage
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV NODE_ENV=production
RUN npm run build

# Serve stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
RUN npm install -g serve
# Render provides PORT
CMD ["sh", "-c", "serve -s dist -l $PORT"]
```
**Key Points:**
- Uses Render's PORT
- Multi-stage build
- Production optimized

## 4. Backend Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install PostgreSQL client
RUN apk add --no-cache postgresql-client

COPY package*.json ./
RUN npm install

COPY . .

# Render handles the port
CMD ["node", "src/index.js"]
```
**Key Points:**
- Simpler than self-hosted version
- No need for wait scripts
- Render handles database connection

## 5. Express Entry Point (backend/src/index.js)
```javascript
const express = require('express');
const { sequelize } = require('./models');
const cors = require('cors');

const app = express();
const PORT = process.env.PORT || 3000;

app.use(cors());
app.use(express.json());

// Health check for Render
app.get('/health', (req, res) => {
  res.send('OK');
});

// API routes
app.use('/api', require('./routes'));

// Render will wait for this
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
  // Connect to database
  sequelize.authenticate()
    .then(() => console.log('Database connected'))
    .catch(err => console.error('Database connection error:', err));
});
```
**Key Points:**
- Uses Render's PORT
- Health check endpoint
- Proper error handling

## 6. Database Config (backend/src/config/database.js)
```javascript
module.exports = {
  production: {
    use_env_variable: 'DATABASE_URL',
    dialect: 'postgres',
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false
      }
    }
  }
};
```
**Key Points:**
- Uses Render's DATABASE_URL
- SSL configuration for Render
- Production settings

## 7. Deployment Steps
```bash
# 1. Push code to GitHub

# 2. On Render:
- Create New Blueprint
- Connect repository
- Verify environment variables

# 3. Auto-deploys will trigger on:
- Push to main branch
- Manual deploy click
```

## 8. Environment Variables
```bash
# Set in Render dashboard:

# Frontend
NODE_ENV=production

# Backend
NODE_ENV=production
JWT_SECRET=auto-generated
DATABASE_URL=auto-provided

# Don't need to set:
PORT (provided by Render)
DATABASE_URL (provided by Render)
```

## 9. Common Issues & Solutions
```bash
# 1. Build Failures
- Check node version in Dockerfile
- Verify all dependencies in package.json
- Check build logs in Render dashboard

# 2. Database Connection
- Check SSL settings
- Verify DATABASE_URL format
- Check sequelize config

# 3. Frontend/Backend Communication
- Verify API URLs
- Check CORS settings
- Verify proxy settings
```

## 10. Development vs Production
```bash
# Local Development
docker-compose up --build

# Production (Render)
git push origin main

# Key Differences:
- Render manages ports
- Automatic SSL
- Managed PostgreSQL
- Auto-scaling
```

## 11. Monitoring & Maintenance
```bash
# Render Dashboard provides:
- Build logs
- Deploy logs
- Resource usage
- Database metrics
- Auto-restart on crash
```

## 12. Best Practices
```bash
# 1. Security
- Use environment variables
- Enable SSL
- Set CORS properly

# 2. Performance
- Optimize Docker images
- Use caching
- Minimize dependencies

# 3. Maintenance
- Regular dependency updates
- Monitor logs
- Set up alerts
```

