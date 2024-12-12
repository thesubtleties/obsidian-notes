
# 🐳 Full-Stack Deployment Guide: Local to Docker (Simple Approach)
## 1. Project Structure
```bash
your-app/
├── frontend/                # Same React setup
│   ├── src/               
│   ├── vite.config.js     
│   └── Dockerfile         
├── backend/               
│   ├── app/               # Flask application
│   │   ├── __init__.py    # Flask app initialization
│   │   ├── models.py      # SQLAlchemy models
│   │   └── routes.py      # Flask routes
│   ├── migrations/        # Alembic migrations
│   ├── requirements.txt   # Python dependencies
│   └── Dockerfile        
├── docker-compose.yml     
└── .env                   
```

## 2. Frontend Dockerfile (Stays the same)
```dockerfile
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
ENV NODE_ENV=production
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
RUN npm install -g serve
EXPOSE 80
CMD ["serve", "-s", "dist", "-l", "80"]
```

## 3. Backend Dockerfile (Flask Version)
```dockerfile
# Use Python 3.11 Alpine
FROM python:3.11-alpine

WORKDIR /app

# Install PostgreSQL client and build dependencies
RUN apk add --no-cache postgresql-client gcc musl-dev postgresql-dev

# Copy requirements first for better caching
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Wait for DB and run migrations
CMD ["sh", "-c", "\
    echo 'Waiting for database...' && \
    while ! pg_isready -h $DB_HOST -U $DB_USER; do \
        sleep 1; \
    done && \
    echo 'Database ready!' && \
    flask db upgrade && \
    gunicorn --bind 0.0.0.0:5000 'app:create_app()'"]
```
**Key Points:**
- Uses Python Alpine image
- Includes PostgreSQL client for migrations
- Uses gunicorn for production server
- Runs Flask migrations instead of Sequelize

## 4. requirements.txt
```txt
flask==2.3.3
flask-sqlalchemy==3.1.1
flask-migrate==4.0.5
psycopg2-binary==2.9.9
python-dotenv==1.0.0
gunicorn==21.2.0
flask-cors==4.0.0
```
**Key Points:**
- SQLAlchemy for database ORM
- Flask-Migrate for migrations
- psycopg2 for PostgreSQL
- gunicorn for production server

## 5. docker-compose.yml
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
      - FLASK_APP=app
      - FLASK_ENV=production
      - DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@db/${DB_NAME}
      - SECRET_KEY=${SECRET_KEY}
      - DB_HOST=db
      - DB_USER=${DB_USER}
      - DB_PASSWORD=${DB_PASSWORD}
      - DB_NAME=${DB_NAME}
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
**Key Changes:**
- Flask environment variables
- Database URL format for SQLAlchemy
- Python-specific configurations

## 6. Environment Variables (.env)
```bash
# Database
DB_USER=myuser
DB_PASSWORD=securepassword
DB_NAME=myapp

# Flask
SECRET_KEY=your-secret-key
FLASK_ENV=production

# Optional
FLASK_DEBUG=0
GUNICORN_WORKERS=4
```

## 7. Flask App Structure (backend/app/__init__.py)
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate
from flask_cors import CORS
import os

db = SQLAlchemy()
migrate = Migrate()

def create_app():
    app = Flask(__name__)
    
    # Configuration
    app.config['SQLALCHEMY_DATABASE_URI'] = os.getenv('DATABASE_URL')
    app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False
    app.config['SECRET_KEY'] = os.getenv('SECRET_KEY')
    
    # Initialize extensions
    CORS(app)
    db.init_app(app)
    migrate.init_app(app, db)
    
    # Register blueprints
    from .routes import api
    app.register_blueprint(api, url_prefix='/api')
    
    return app
```

## 8. Common Commands (Flask Specific)
```bash
# Database migrations
docker-compose exec backend flask db migrate
docker-compose exec backend flask db upgrade

# Flask shell
docker-compose exec backend flask shell

# View logs
docker-compose logs -f backend

# Check Flask app status
docker-compose exec backend ps aux | grep gunicorn
```

## 9. Development vs Production
```bash
# Development
# frontend/vite.config.js stays the same
server: {
    proxy: {
      '/api': 'http://localhost:5000',
    },
}

# Production
# Uses gunicorn instead of Flask development server
# Runs through reverse proxy
```

## 10. Troubleshooting Flask-Specific Issues
```bash
# Check Flask logs
docker-compose logs backend

# Check gunicorn status
docker-compose exec backend ps aux | grep gunicorn

# Database connection issues
docker-compose exec backend flask shell
>>> from app import db
>>> db.engine.connect()

# Migration issues
docker-compose exec backend flask db current
docker-compose exec backend flask db history
```

The main differences from the Node.js version are:
1. Python dependencies instead of node_modules
2. Flask-specific environment variables
3. Gunicorn for production server
4. Alembic/Flask-Migrate for migrations
5. SQLAlchemy instead of Sequelize

