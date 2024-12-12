## 1. Project Structure
```bash
your-app/
├── frontend/                
│   ├── src/               
│   ├── vite.config.js     # Dev proxy config
│   └── Dockerfile         
├── backend/               
│   ├── app/              
│   │   ├── __init__.py    # Flask app initialization
│   │   ├── models.py      # SQLAlchemy models
│   │   └── routes.py      # Flask routes
│   ├── migrations/        # Alembic migrations
│   ├── requirements.txt   # Python dependencies
│   └── Dockerfile        
└── render.yaml            # Render configuration
```
**Key Point:** Flask-specific structure with migrations

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

  # Flask Backend Service
  - type: web
    name: myapp-backend
    env: docker
    buildCommand: docker build -f backend/Dockerfile -t myapp-backend ./backend
    startCommand: gunicorn --bind 0.0.0.0:$PORT 'app:create_app()'
    envVars:
      - key: FLASK_ENV
        value: production
      - key: DATABASE_URL
        fromDatabase:
          name: myapp-db
          property: connectionString
      - key: SECRET_KEY
        generateValue: true

  # Database
  - type: pgsql
    name: myapp-db
    ipAllowList: []
```
**Key Points:**
- Uses gunicorn for production
- Auto-generates SECRET_KEY
- Postgres URL auto-injected

## 3. Frontend Dockerfile (Same as Express version)
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
CMD ["sh", "-c", "serve -s dist -l $PORT"]
```

## 4. Flask Backend Dockerfile
```dockerfile
FROM python:3.11-alpine

WORKDIR /app

# Install dependencies for psycopg2 and other builds
RUN apk add --no-cache \
    postgresql-client \
    gcc \
    musl-dev \
    postgresql-dev \
    python3-dev

# Install Python packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Render handles the port
CMD ["gunicorn", "--bind", "0.0.0.0:$PORT", "app:create_app()"]
```
**Key Points:**
- Alpine-based for smaller size
- Includes build dependencies
- Uses gunicorn for production

## 5. Flask App (backend/app/__init__.py)
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
    
    # Health check for Render
    @app.route('/health')
    def health():
        return {'status': 'healthy'}, 200
    
    # Register blueprints
    from .routes import api
    app.register_blueprint(api, url_prefix='/api')
    
    return app
```
**Key Points:**
- Uses Render's DATABASE_URL
- CORS enabled
- Health check endpoint
- Blueprint structure

## 6. requirements.txt
```txt
Flask==2.3.3
flask-sqlalchemy==3.1.1
flask-migrate==4.0.5
flask-cors==4.0.0
psycopg2-binary==2.9.9
python-dotenv==1.0.0
gunicorn==21.2.0
```
**Key Points:**
- Production-ready dependencies
- psycopg2-binary for Postgres
- gunicorn for serving

## 7. Database Models Example
```python
# backend/app/models.py
from . import db
from datetime import datetime

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(120), unique=True, nullable=False)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)

    def to_dict(self):
        return {
            'id': self.id,
            'email': self.email,
            'created_at': self.created_at.isoformat()
        }
```

## 8. Deployment Steps
```bash
# 1. Push code to GitHub

# 2. On Render:
- Create New Blueprint
- Connect repository
- Verify environment variables

# 3. First-time database setup
- Render will create database
- Migrations run automatically

# 4. Subsequent deployments:
- Auto-deploy on push to main
- Migrations run automatically
```

## 9. Environment Variables
```bash
# Set in Render dashboard:

# Frontend
NODE_ENV=production

# Backend
FLASK_ENV=production
SECRET_KEY=auto-generated
DATABASE_URL=auto-provided

# Don't need to set:
PORT (provided by Render)
```

## 10. Common Issues & Solutions
```bash
# 1. Database Migrations
- Check migration logs in Render
- Verify alembic version
- Check DATABASE_URL format

# 2. CORS Issues
- Verify CORS configuration
- Check allowed origins
- Check request headers

# 3. Gunicorn Problems
- Check worker count
- Verify timeout settings
- Monitor memory usage
```

## 11. Development vs Production
```bash
# Local Development
flask run  # Backend
npm run dev  # Frontend

# Production (Render)
gunicorn  # Backend
serve  # Frontend

# Key Differences:
- Render handles SSL
- Auto database connection
- Production-grade server
```

## 12. Best Practices
```bash
# 1. Database
- Use migrations for schema changes
- Handle connection pooling
- Implement retry logic

# 2. Application
- Use blueprints for organization
- Implement proper error handling
- Add request logging

# 3. Security
- Enable CORS properly
- Use environment variables
- Implement rate limiting
```

