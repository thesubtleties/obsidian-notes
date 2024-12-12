# 🐳 Full-Stack Deployment Guide: Local to Docker (Simple Approach)

## 1. Project Structure
```bash
your-app/
├── frontend/                
│   ├── src/               
│   ├── vite.config.js     # Dev proxy config
│   └── Dockerfile         # Uses 'serve' instead of nginx
├── backend/               
│   ├── src/              
│   └── Dockerfile        
├── docker-compose.yml     # Orchestrates all containers
└── .env                   # Environment variables
```
**Key Point:** Simple structure, each part does one thing well

## 2. Frontend Dockerfile (frontend/Dockerfile)
```dockerfile
# Build stage - compile the React/Vite app
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV NODE_ENV=production
RUN npm run build

# Serve stage - lightweight static server
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
RUN npm install -g serve
EXPOSE 80
CMD ["serve", "-s", "dist", "-l", "80"]
```
**Why This Works:**
- Multi-stage build keeps final image small
- `serve` handles SPA routing automatically
- No complex nginx configuration needed
- Production-ready and lightweight

## 3. Backend Dockerfile (backend/Dockerfile)
```dockerfile
FROM node:18-alpine

WORKDIR /app

# Database tools for migrations/seeds
RUN apk add --no-cache postgresql-client

COPY package*.json ./
RUN npm install

COPY . .

# Wait for DB, run migrations, start server
CMD ["sh", "-c", "\
    until pg_isready -h ${DB_HOST} -U ${DB_USER}; do \
        echo 'Waiting for database...' && \
        sleep 2; \
    done && \
    npm run migrate && \
    npm start"]
```
**Key Points:**
- Includes DB tools for management
- Waits for database before starting
- Simple and focused on backend tasks

## 4. docker-compose.yml
```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: myapp-frontend
    restart: unless-stopped
    networks:
      - shared_portainer_network
    depends_on:
      - backend
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost:80"]
      interval: 30s
      timeout: 10s
      retries: 3

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: myapp-backend
    restart: unless-stopped
    environment:
      - NODE_ENV=production
      - PORT=5000
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
      - DB_HOST=db
      - JWT_SECRET=${JWT_SECRET}
    networks:
      - shared_portainer_network
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    container_name: myapp-db
    restart: unless-stopped
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - shared_portainer_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  shared_portainer_network:
    external: true

volumes:
  postgres_data:
```
**Key Points:**
- Uses external network for your reverse proxy
- Health checks for all services
- Persistent database volume
- Environment variables from .env

## 5. Development Proxy (frontend/vite.config.js)
```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': 'http://localhost:5000',
    },
  },
});
```
**Why This Matters:**
- Development-only configuration
- Allows same API calls in dev and prod
- No need to change URLs between environments

## 6. Environment Variables (.env)
```bash
# Database
DB_USER=myuser
DB_PASSWORD=securepassword
DB_NAME=myapp

# JWT (if using authentication)
JWT_SECRET=your-secret-key
JWT_EXPIRES_IN=7d

# Node
NODE_ENV=production
```

## 7. Deployment Steps
```bash
# 1. Clone and setup
git clone your-repository
cd your-app
cp .env.example .env  # Create and edit .env

# 2. Build and start
docker-compose up --build -d

# 3. Watch logs
docker-compose logs -f

# 4. Verify all services
docker-compose ps

# 5. Check each container's health
docker-compose ps
```

## 8. Common Commands
```bash
# Start everything
docker-compose up -d

# Stop everything
docker-compose down

# Rebuild specific service
docker-compose up -d --build frontend

# View logs
docker-compose logs -f frontend
docker-compose logs -f backend

# Reset database
docker-compose down -v  # Removes volumes
docker-compose up -d   # Recreates everything

# Shell into containers
docker-compose exec backend sh
docker-compose exec db psql -U myuser -d myapp
```

## 9. Development vs Production
```bash
# Development
npm run dev    # Runs Vite dev server
# Uses vite.config.js proxy

# Production
docker-compose up -d  # Runs containerized version
# Uses your reverse proxy (like Traefik/nginx)
```

## 10. Troubleshooting
```bash
# 1. Frontend not accessible
docker-compose logs frontend
# Check if serve started correctly

# 2. Backend connection issues
docker-compose logs backend
# Check if waiting for database

# 3. Database issues
docker-compose logs db
# Check if postgres started properly

# 4. Network issues
docker network ls
docker network inspect shared_portainer_network
```

