## Introduction: Why Docker Matters

Docker allows you to package your application and all its dependencies into a standardized unit (a container) that runs consistently across any environment. For self-hosting applications on your private server, this means:

- Consistent deployment from development to production
- Isolation between applications
- Easier management of dependencies
- Simplified scaling and updates

When using Docker Compose, you're orchestrating multiple containers that work together (like your web app and database), defining their relationships in a single YAML file.

## Prerequisites

- Basic command line familiarity
- Docker and Docker Compose installed on your system
- Understanding of your application's basic requirements

## 1. Dockerfile Fundamentals

### What is a Dockerfile?

A Dockerfile is a text document containing commands that Docker uses to automatically build an image. Think of it as a recipe for creating the environment your application needs to run.

### Basic Structure of a Dockerfile

```dockerfile
# Base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy dependency files
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy application code
COPY . .

# Build the application
RUN npm run build

# Command to run when container starts
CMD ["npm", "start"]
```

### Essential Instructions Explained

- **FROM**: The foundation of your image. Always start here.
  - *Why*: You build on existing images rather than from scratch to leverage pre-configured environments
  - *Example*: `FROM node:18-alpine` uses a lightweight Node.js environment

- **WORKDIR**: Sets the working directory for subsequent instructions
  - *Why*: Creates and organizes a dedicated directory for your app, preventing file location confusion
  - *Example*: `WORKDIR /app` creates and switches to /app directory

- **COPY**: Adds files from your local machine to the container
  - *Why*: Transfers your application code into the image
  - *Example*: `COPY . .` copies all files from current directory to WORKDIR

- **RUN**: Executes commands and creates a new layer in the image
  - *Why*: Used for installing dependencies, building assets, etc.
  - *Example*: `RUN npm install` installs Node.js dependencies

- **EXPOSE**: Documents which ports the container will listen on
  - *Why*: Makes your intention clear about networking, but doesn't actually open ports
  - *Example*: `EXPOSE 3000` indicates the app listens on port 3000

- **CMD/ENTRYPOINT**: Defines the default command to run when the container starts
  - *Why*: Specifies how your application starts
  - *Example*: `CMD ["npm", "start"]` runs the start script

## 2. Creating Dockerfiles for Specific Applications

### Python/Flask Application

```dockerfile
# Start with Python image
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies (if needed)
RUN apt-get update && apt-get install -y \
    gcc \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Run the application
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:app"]
```

**Key considerations for Python apps:**
- Use `requirements.txt` for dependencies
- Consider using a production-ready WSGI server like Gunicorn
- Install any system-level dependencies needed for Python packages
- Set environment variables for Flask (`FLASK_ENV`, `FLASK_APP`)

### React/Vite/Next.js Application (Multi-stage build)

```dockerfile
# Build stage
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine

# Copy built assets from build stage
COPY --from=build /app/dist /usr/share/nginx/html

# Copy nginx config if needed
# COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Key considerations for JavaScript frameworks:**
- Multi-stage builds reduce final image size
- For Next.js, adjust the build commands and container
- Optimize build by copying package.json first (better caching)
- Use nginx for serving static files in production

### Database Containers

Typically for databases, you'll rarely write custom Dockerfiles and instead use official images with configuration:

**In your docker-compose.yml for PostgreSQL:**

```yaml
services:
  postgres:
    image: postgres:14
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

**Key database considerations:**
- Always use volumes for persistence
- Set credentials via environment variables
- Consider initialization scripts for setup

## 3. Development vs. Production Environments

### Development Dockerfile

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

# In development, we mount the code as a volume
# So we don't need to copy it here

EXPOSE 3000

CMD ["npm", "run", "dev"]
```

### Production Dockerfile

```dockerfile
# Build stage
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

# Production stage - smaller image
FROM node:18-alpine

WORKDIR /app

COPY --from=builder /app/package*.json ./
RUN npm install --production

COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public

EXPOSE 3000

CMD ["npm", "run", "start"]
```

### Key Differences Explained

1. **Dependencies**
   - Dev: Full development dependencies
   - Prod: Only production dependencies (`npm install --production`)

2. **Build Process**
   - Dev: Often uses hot-reloading
   - Prod: Pre-builds all assets for performance

3. **Security**
   - Dev: May include debugging tools
   - Prod: Minimizes attack surface, no dev tools

4. **Image Size**
   - Dev: Can be larger
   - Prod: Optimized with multi-stage builds

5. **Configuration**
   - Dev: More verbose output, development-specific environment variables
   - Prod: Performance-optimized configurations

## 4. Docker Compose for Multi-Container Applications

Example docker-compose.yml for a full-stack application:

```yaml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev  # Use Dockerfile.prod for production
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app  # For development
      - /app/node_modules  # Don't override node_modules with host
    environment:
      - REACT_APP_API_URL=http://backend:5000
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
    ports:
      - "5000:5000"
    volumes:
      - ./backend:/app  # For development
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

## 5. Advanced Dockerfile Techniques

### Multi-stage Builds

```dockerfile
# Build stage
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Why use multi-stage builds?**
- Dramatically reduces final image size
- Keeps build tools out of production
- Separates build-time from runtime dependencies

### Optimizing Layer Caching

```dockerfile
# Place commands that change less frequently earlier
FROM node:18-alpine

WORKDIR /app

# Dependencies change less often than code
COPY package*.json ./
RUN npm install

# Application code changes frequently
COPY . .

# Build step
RUN npm run build

CMD ["npm", "start"]
```

**Why this order matters:**
- Docker caches layers
- When a layer changes, all subsequent layers must be rebuilt
- By separating dependency installation from code, you avoid reinstalling dependencies when only code changes

### Using .dockerignore

Create a `.dockerignore` file:
```
node_modules
.git
.env
*.log
```

**Why use .dockerignore?**
- Prevents unnecessary files from being sent to the build context
- Makes builds faster and images smaller
- Prevents sensitive information from being included

## 6. Best Practices for Production Dockerfiles

### Security Best Practices

1. **Run as non-root user:**
```dockerfile
FROM node:18-alpine

# Create app directory
WORKDIR /app

# Create a user and set permissions
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
RUN chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Continue with your Dockerfile...
```

2. **Use specific tags** for base images rather than `:latest`

3. **Scan images** for vulnerabilities with tools like Docker Scout

4. **Don't store secrets** in Dockerfiles; use environment variables or secrets management

### Health Checks

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .
RUN npm install && npm run build

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/ || exit 1

CMD ["npm", "start"]
```

**Why add health checks?**
- Allows Docker to detect when applications are unhealthy
- Enables automatic restarts
- Essential for production reliability

### Optimizing Image Size

1. **Use lightweight base images**:
   - Alpine-based (`node:18-alpine` vs `node:18`)
   - Distroless images for even smaller footprints

2. **Clean up in the same layer**:
```dockerfile
RUN apt-get update && apt-get install -y \
    package1 \
    package2 \
    && rm -rf /var/lib/apt/lists/*
```

3. **Combine RUN commands** with `&&` to reduce layers

## 7. Common Pitfalls and Troubleshooting

### Permission Issues

**Problem**: Permission denied errors when accessing files
**Solution**: 
- Set correct ownership in Dockerfile with `chown`
- Properly manage volumes permissions
- Use non-root users with appropriate access

### Container Startup Failures

**Problem**: Container exits immediately after starting
**Solutions**:
- Check CMD/ENTRYPOINT is correct
- Verify all dependencies are installed
- Check for port conflicts
- View logs with `docker logs <container_name>`

### Database Connection Issues

**Problem**: Application can't connect to database
**Solutions**:
- Use service name (e.g., `db`) not localhost in connection strings
- Ensure correct ports are exposed
- Add appropriate `depends_on` in docker-compose.yml
- Use wait scripts for dependent services

## 8. Real-World Example: Full-Stack Application

### Project Structure
```
my-app/
├── docker-compose.yml
├── docker-compose.prod.yml
├── frontend/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   └── ...
├── backend/
│   ├── Dockerfile
│   └── ...
└── nginx/
    └── nginx.conf
```

### Practical docker-compose.prod.yml
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf
      - ./certbot/conf:/etc/letsencrypt
      - ./certbot/www:/var/www/certbot
    depends_on:
      - frontend
      - backend

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.prod
    environment:
      - NODE_ENV=production
      - API_URL=/api

  backend:
    build:
      context: ./backend
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgres://user:password@db:5432/myapp
    depends_on:
      - db

  db:
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=myapp
    restart: unless-stopped

volumes:
  postgres_data:
```

## Conclusion

Creating effective Dockerfiles is both an art and a science. Start simple, focusing on getting your application running correctly, then optimize for production use. Remember:

1. Start with a solid base image
2. Structure your Dockerfile for optimal caching
3. Use multi-stage builds for production
4. Keep security in mind from the beginning
5. Use different strategies for development and production
6. Docker Compose is your friend for multi-container applications
7. Always test in an environment similar to production

As you grow more comfortable with Docker, you'll develop intuition for what works best for your specific applications. The examples in this guide provide a strong foundation for self-hosting various web applications and databases on your private server.
