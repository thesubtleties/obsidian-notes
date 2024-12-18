NotebookLM podcast style overview of this guide: https://notebooklm.google.com/notebook/aa9ddc58-1f6f-4056-b630-e9ab6c18cedc/audio

## Basic Structure
```
project_root/
├── app/
│   ├── __init__.py         # App initialization
│   ├── config.py           # Configuration settings
│   ├── models.py           # Database models
│   ├── forms.py            # Form definitions
│   ├── routes/
│   │   ├── __init__.py     # Blueprint registration
│   │   └── simple.py       # Route definitions
│   └── templates/
│       ├── main_page.html
│       ├── simple_form.html
│       └── simple_form_data.html
├── migrations/
├── .flaskenv
├── .env
└── requirements.txt
```

## 1. Configuration Setup
```python
# app/config.py
import os
from dotenv import load_dotenv # Not actually needed

# Load environment variables from .env file
load_dotenv()

class Configuration:
    # Use environment variables with fallbacks
    SECRET_KEY = os.environ.get('SECRET_KEY', 'dev-key')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL', 'sqlite:///dev.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

```bash
# .env
SECRET_KEY='your-secret-key-here'
DATABASE_URL='sqlite:///dev.db'
```
**What's happening here:**
- Basic configuration for Flask app
- Setting up database connection
- Disabling SQLAlchemy event system
- Importing required Python modules```
**Real-world considerations:**
```python
# app/config.py
import os
from datetime import timedelta
from typing import Dict, Any  # Type hints for better code documentation
from urllib.parse import urlparse

class BaseConfig:
    """Base configuration with shared settings"""
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-key'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    
    # Rate limiting for API endpoints
    RATELIMIT_DEFAULT = "100/hour"
    
    # Redis cache configuration
    REDIS_URL = os.environ.get('REDIS_URL')
    
    # CDN configuration for static files
    CDN_DOMAIN = os.environ.get('CDN_DOMAIN')
    
    @property
    def REDIS_CONFIG(self) -> Dict[str, Any]:
        """Parse Redis URL into config dict"""
        if not self.REDIS_URL:
            return {}
        
        url = urlparse(self.REDIS_URL)
        return {
            'host': url.hostname,
            'port': url.port,
            'password': url.password,
        }

class DevelopmentConfig(BaseConfig):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'

class ProductionConfig(BaseConfig):
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    SQLALCHEMY_POOL_SIZE = 20
```

**Learn More:**
- [Rate Limiting in Flask](https://flask-limiter.readthedocs.io/en/stable/) - For API rate limiting implementation
- [Redis Caching with Flask](https://www.youtube.com/watch?v=iO0sL8Vb0LA) - Great video on implementing Redis caching

## 2. Models Setup

```python
# app/models.py
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

# Create database instance
db = SQLAlchemy()

class SimplePerson(db.Model):
    __tablename__ = 'simple_people'

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50), nullable=False)
    age = db.Column(db.Integer)
    bio = db.Column(db.String(2000))
    
    def __repr__(self):
        return f'<Person {self.name}>'
```

**What's happening here:**
- Creating SQLAlchemy instance
- Defining basic model with table name
- Setting up columns with types and constraints
- Adding string representation for debugging

**Real-world considerations:**
```python
# app/models.py
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime
from typing import Dict, Any
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy.dialects.postgresql import JSONB
from sqlalchemy.orm import validates
import re

db = SQLAlchemy()

class BaseModel(db.Model):
    """Abstract base model with common fields"""
    __abstract__ = True
    
    id = db.Column(db.Integer, primary_key=True)
    created_at = db.Column(db.DateTime, default=datetime.utcnow)
    updated_at = db.Column(db.DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    is_active = db.Column(db.Boolean, default=True)
    
    def save(self) -> bool:
        """Safe save method with error handling"""
        try:
            db.session.add(self)
            db.session.commit()
            return True
        except Exception as e:
            db.session.rollback()
            print(f"Save error: {str(e)}")  # In real life, use proper logging
            return False

class SimplePerson(BaseModel):
    __tablename__ = 'simple_people'
    
    name = db.Column(db.String(50), nullable=False, index=True)
    age = db.Column(db.Integer)
    bio = db.Column(db.Text)
    metadata = db.Column(JSONB)  # For flexible data storage
    
    @validates('name')
    def validate_name(self, key, name):
        """Validate name field"""
        if not name:
            raise ValueError("Name cannot be empty")
        if not re.match("^[a-zA-Z ]*$", name):
            raise ValueError("Name can only contain letters and spaces")
        return name.strip()
    
    @hybrid_property
    def is_adult(self) -> bool:
        """Can be used in queries and Python code"""
        return self.age >= 18 if self.age else False
    
    def to_dict(self) -> Dict[str, Any]:
        """Serialize model for API responses"""
        return {
            'id': self.id,
            'name': self.name,
            'age': self.age,
            'bio': self.bio,
            'is_adult': self.is_adult,
            'created_at': self.created_at.isoformat(),
            'metadata': self.metadata or {}
        }
    
    def __repr__(self):
        return f'<Person {self.name} ({self.age})>'
```

**Learn More:**
- [SQLAlchemy Hybrid Properties](https://www.youtube.com/watch?v=P7n4OQr5H-A) - Great explanation of hybrid properties and their power
- [PostgreSQL JSONB in SQLAlchemy](https://amercader.net/blog/beware-of-json-fields-in-sqlalchemy/) - Deep dive into JSONB usage

Key real-world additions explained:
1. Base model with common fields
2. Field validation
3. Hybrid properties for computed fields
4. JSON field support
5. Safe database operations
6. Type hints for better IDE support


## 3. Forms Setup

```python
# app/forms.py
from flask_wtf import FlaskForm
from wtforms import StringField, IntegerField, TextAreaField, SubmitField
from wtforms.validators import DataRequired

class SimpleForm(FlaskForm):
    name = StringField('Name', validators=[DataRequired()])
    age = IntegerField('Age')
    bio = TextAreaField('Biography')
    submit = SubmitField('Submit')
```

**What's happening here:**
- Creating a Flask-WTF form class
- Defining form fields with labels
- Adding basic validation (required field)
- Setting up submit button

**Real-world considerations:**
```python
# app/forms.py
from flask_wtf import FlaskForm
from wtforms import StringField, IntegerField, TextAreaField, SubmitField
from wtforms.validators import (
    DataRequired, Length, Optional, NumberRange, 
    ValidationError, Regexp
)
from typing import Optional as OptionalType  # Avoid naming conflict

class SimpleForm(FlaskForm):
    name = StringField('Name', validators=[
        DataRequired(message="Name is required"),
        Length(min=2, max=50, message="Name must be between 2 and 50 characters"),
        Regexp(r'^[a-zA-Z\s]*$', message="Name can only contain letters and spaces")
    ])
    
    age = IntegerField('Age', validators=[
        Optional(),
        NumberRange(min=0, max=150, message="Please enter a valid age")
    ])
    
    bio = TextAreaField('Biography', validators=[
        Optional(),
        Length(max=2000, message="Bio must be less than 2000 characters")
    ])
    
    submit = SubmitField('Submit')
    
    def __init__(self, *args, **kwargs):
        """Initialize form with custom setup"""
        super(SimpleForm, self).__init__(*args, **kwargs)
        self.original_name = None
    
    def validate_name(self, field) -> OptionalType[ValidationError]:
        """Custom validator for name field"""
        # Example: Check for reserved names
        reserved_names = ['admin', 'root', 'system']
        if field.data.lower() in reserved_names:
            raise ValidationError('This name is reserved')
            
    def process_data(self) -> dict:
        """Clean and process form data"""
        return {
            'name': self.name.data.strip(),
            'age': self.age.data if self.age.data else None,
            'bio': self.bio.data.strip() if self.bio.data else None
        }
    
    @property
    def has_bio(self) -> bool:
        """Check if bio is provided"""
        return bool(self.bio.data and self.bio.data.strip())
```

**Learn More:**
- [WTForms Custom Validators](https://wtforms.readthedocs.io/en/3.0.x/validators/#custom-validators) - Deep dive into custom validation
- [Flask-WTF Security Best Practices](https://flask-wtf.readthedocs.io/en/1.0.x/form/#security-features) - Important security considerations

Key real-world additions explained:
1. Comprehensive validation rules
2. Custom validation methods
3. Data processing methods
4. Helper properties
5. Type hints
6. Detailed error messages
7. Form initialization customization

The real-world version includes:
- Better error messages for user experience
- Multiple validators per field
- Custom validation logic
- Data processing methods
- Type safety
- Regular expression validation
- Helper methods for form handling

## 4. Routes Setup

```python
# app/routes/simple.py
from flask import Blueprint, render_template, redirect
from ..forms import SimpleForm
from ..models import db, SimplePerson

simple_bp = Blueprint('simple', __name__)

@simple_bp.route('/')
def index():
    return render_template('main_page.html')

@simple_bp.route('/simple-form', methods=['GET', 'POST'])
def simple_form():
    form = SimpleForm()
    if form.validate_on_submit():
        person = SimplePerson(
            name=form.name.data,
            age=form.age.data,
            bio=form.bio.data
        )
        db.session.add(person)
        db.session.commit()
        return redirect('/')
    return render_template('simple_form.html', form=form)
```

**What's happening here:**
- Creating a Blueprint for route organization
- Defining basic routes with methods
- Handling form submission
- Basic database operations
- Simple redirects

**Real-world considerations:**
```python
# app/routes/simple.py
from flask import (
    Blueprint, render_template, redirect, request, 
    flash, current_app, jsonify, abort
)
from werkzeug.exceptions import HTTPException
from sqlalchemy.exc import SQLAlchemyError
from typing import Union, Tuple
from ..forms import SimpleForm
from ..models import db, SimplePerson
from functools import wraps
import logging

logger = logging.getLogger(__name__)
simple_bp = Blueprint('simple', __name__)

def handle_db_error(f):
    """Decorator for database error handling"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        try:
            return f(*args, **kwargs)
        except SQLAlchemyError as e:
            db.session.rollback()
            logger.error(f"Database error: {str(e)}")
            flash("An error occurred. Please try again.", "error")
            return redirect('/error')
    return decorated_function

@simple_bp.route('/')
def index():
    return render_template('main_page.html')

@simple_bp.route('/simple-form', methods=['GET', 'POST'])
@handle_db_error
def simple_form() -> Union[str, Tuple[str, int]]:
    form = SimpleForm()
    
    if request.method == 'POST':
        if form.validate_on_submit():
            try:
                person = SimplePerson(
                    name=form.name.data.strip(),
                    age=form.age.data,
                    bio=form.bio.data
                )
                db.session.add(person)
                db.session.commit()
                
                flash("Person added successfully!", "success")
                
                # Handle AJAX requests
                if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
                    return jsonify({'status': 'success', 'id': person.id})
                    
                return redirect('/')
            
            except Exception as e:
                logger.error(f"Error creating person: {str(e)}")
                db.session.rollback()
                flash("Error creating record", "error")
                return render_template('simple_form.html', form=form), 400
        else:
            # Form validation failed
            if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
                return jsonify({'status': 'error', 'errors': form.errors}), 400
    
    # GET request or form validation failed
    return render_template('simple_form.html', form=form)

@simple_bp.route('/simple-form-data')
def simple_form_data():
    try:
        # Paginated query
        page = request.args.get('page', 1, type=int)
        per_page = current_app.config.get('ITEMS_PER_PAGE', 10)
        
        query = SimplePerson.query.filter(
            SimplePerson.name.like('M%')
        ).order_by(SimplePerson.created_at.desc())
        
        pagination = query.paginate(page=page, per_page=per_page)
        
        if request.headers.get('X-Requested-With') == 'XMLHttpRequest':
            return jsonify({
                'items': [p.to_dict() for p in pagination.items],
                'total': pagination.total,
                'pages': pagination.pages
            })
            
        return render_template(
            'simple_form_data.html',
            people=pagination.items,
            pagination=pagination
        )
        
    except Exception as e:
        logger.error(f"Error retrieving data: {str(e)}")
        abort(500)

@simple_bp.errorhandler(HTTPException)
def handle_exception(e):
    """Handle HTTP errors"""
    response = e.get_response()
    response.data = jsonify({
        "code": e.code,
        "name": e.name,
        "description": e.description,
    }).data
    response.content_type = "application/json"
    return response
```

**Learn More:**
- [Flask Error Handling Best Practices](https://www.youtube.com/watch?v=P-Z1wXFW4Is) - Comprehensive error handling in Flask
- [RESTful API Design](https://www.youtube.com/watch?v=MJZCCpCYEqk) - Great video on API design principles

Key real-world additions explained:
1. Error handling with decorators
2. AJAX support
3. Pagination
4. Logging
5. Database transaction management
6. Flash messages
7. Content negotiation (HTML/JSON responses)
8. Type hints
9. Custom error handlers

## 5. Application Factory Pattern

```python
# Simple Version (what we used)
# app/__init__.py
from flask import Flask
from .config import Configuration
from .routes import register_blueprints
from .models import db

app = Flask(__name__)
app.config.from_object(Configuration)
db.init_app(app)
register_blueprints(app)
```

**What's happening here:**
- Creating Flask app directly
- Loading config
- Initializing database
- Registering routes
- Single configuration

**Real-world Factory Pattern:**
```python
# app/__init__.py
from flask import Flask
from flask_migrate import Migrate
from flask_cors import CORS
from flask_caching import Cache
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address
from logging.config import dictConfig
from typing import Optional
import os

# Initialize extensions outside of factory
db = SQLAlchemy()
migrate = Migrate()
cache = Cache()
limiter = Limiter(key_func=get_remote_address)
cors = CORS()

def setup_logging():
    """Configure logging for the application"""
    dictConfig({
        'version': 1,
        'formatters': {
            'default': {
                'format': '[%(asctime)s] %(levelname)s in %(module)s: %(message)s',
            }
        },
        'handlers': {
            'wsgi': {
                'class': 'logging.StreamHandler',
                'stream': 'ext://flask.logging.wsgi_errors_stream',
                'formatter': 'default'
            },
            'file': {
                'class': 'logging.handlers.RotatingFileHandler',
                'filename': 'app.log',
                'maxBytes': 1024 * 1024,  # 1MB
                'backupCount': 10,
                'formatter': 'default'
            }
        },
        'root': {
            'level': 'INFO',
            'handlers': ['wsgi', 'file']
        }
    })

def create_app(config_name: Optional[str] = None) -> Flask:
    """Application factory function"""
    
    # Setup logging first
    setup_logging()
    
    # Initialize app
    app = Flask(__name__)
    
    # Load configuration
    if config_name is None:
        config_name = os.environ.get('FLASK_ENV', 'development')
    
    config_map = {
        'development': 'config.DevelopmentConfig',
        'production': 'config.ProductionConfig',
        'testing': 'config.TestingConfig'
    }
    
    app.config.from_object(config_map[config_name])
    
    # Initialize extensions
    db.init_app(app)
    migrate.init_app(app, db)
    cache.init_app(app)
    limiter.init_app(app)
    cors.init_app(app)
    
    # Register blueprints
    from .routes import register_blueprints
    register_blueprints(app)
    
    # Register error handlers
    register_error_handlers(app)
    
    # Register CLI commands
    register_commands(app)
    
    # Configure security headers
    @app.after_request
    def add_security_headers(response):
        response.headers['X-Content-Type-Options'] = 'nosniff'
        response.headers['X-Frame-Options'] = 'SAMEORIGIN'
        response.headers['X-XSS-Protection'] = '1; mode=block'
        return response
    
    return app

def register_error_handlers(app):
    """Register error handlers"""
    @app.errorhandler(404)
    def not_found_error(error):
        if request.accepts('json'):
            return jsonify({'error': 'Not found'}), 404
        return render_template('errors/404.html'), 404

    @app.errorhandler(500)
    def internal_error(error):
        db.session.rollback()  # Reset failed database sessions
        if request.accepts('json'):
            return jsonify({'error': 'Internal server error'}), 500
        return render_template('errors/500.html'), 500

def register_commands(app):
    """Register Flask CLI commands"""
    @app.cli.command("init-db")
    def init_db():
        """Initialize the database."""
        db.create_all()
        click.echo('Initialized the database.')

    @app.cli.command("create-admin")
    @click.argument("username")
    @click.password_option()
    def create_admin(username, password):
        """Create an admin user."""
        # Admin user creation logic here
        click.echo(f'Created admin user: {username}')
```

**Learn More:**
- [Application Factory Deep Dive](https://www.youtube.com/watch?v=WCpNvteLCDI) - Excellent explanation of why and how to use the factory pattern
- [Flask Project Structure](https://exploreflask.com/en/latest/organizing.html) - Different ways to structure larger Flask apps

Key real-world additions explained:
1. Multiple configuration environments
2. Extension initialization
3. Logging setup
4. Error handlers
5. Security headers
6. CLI commands
7. Blueprint registration
8. CORS handling
9. Rate limiting
10. Caching

Benefits of Factory Pattern:
- Multiple instances with different configs
- Easier testing
- Better dependency management
- More modular code
- Environment-specific configuration
- Better extension management

## 6. Testing Setup

```python
# Simple Version
# tests/test_routes.py
import unittest
from app import app

class TestRoutes(unittest.TestCase):
    def setUp(self):
        self.client = app.test_client()
        
    def test_index_route(self):
        response = self.client.get('/')
        self.assertEqual(response.status_code, 200)
        
    def test_form_submission(self):
        response = self.client.post('/simple-form', data={
            'name': 'Test User',
            'age': 25,
            'bio': 'Test bio'
        })
        self.assertEqual(response.status_code, 302)  # Redirect after success
```

**What's happening here:**
- Basic unittest setup
- Simple route testing
- Form submission testing
- Status code checking

**Real-world Testing Setup:**
```python
# tests/conftest.py
import pytest
from app import create_app, db
from app.models import SimplePerson
from typing import Generator, Any

@pytest.fixture
def app():
    """Create application for the tests."""
    app = create_app('testing')
    app.config.update({
        'TESTING': True,
        'SQLALCHEMY_DATABASE_URI': 'sqlite:///:memory:'
    })
    return app

@pytest.fixture
def client(app):
    """Test client for our app."""
    return app.test_client()

@pytest.fixture
def runner(app):
    """Test CLI runner."""
    return app.test_cli_runner()

@pytest.fixture
def init_database(app) -> Generator[Any, Any, None]:
    """Initialize test database."""
    with app.app_context():
        db.create_all()
        yield db  # this is where the testing happens
        db.drop_all()

@pytest.fixture
def sample_person(init_database) -> SimplePerson:
    """Create a sample person for testing."""
    person = SimplePerson(
        name='Test User',
        age=25,
        bio='Test biography'
    )
    db.session.add(person)
    db.session.commit()
    return person

# tests/test_routes.py
from flask import url_for
import pytest
from app.models import SimplePerson

def test_index_route(client):
    """Test index route."""
    response = client.get('/')
    assert response.status_code == 200
    assert b'Python Flask Practice Assessment' in response.data

def test_form_submission(client, init_database):
    """Test form submission."""
    # Test GET request
    response = client.get('/simple-form')
    assert response.status_code == 200
    assert b'Name' in response.data
    
    # Test POST request with valid data
    response = client.post('/simple-form', data={
        'name': 'Test User',
        'age': '25',
        'bio': 'Test bio'
    }, follow_redirects=True)
    assert response.status_code == 200
    
    # Verify database entry
    person = SimplePerson.query.filter_by(name='Test User').first()
    assert person is not None
    assert person.age == 25

def test_invalid_form_submission(client, init_database):
    """Test form submission with invalid data."""
    response = client.post('/simple-form', data={
        'name': '',  # Required field
        'age': 'not a number',
        'bio': 'Test bio'
    })
    assert response.status_code == 400
    assert b'Bad Data' in response.data

# tests/test_models.py
import pytest
from app.models import SimplePerson
from sqlalchemy.exc import IntegrityError

def test_create_person(init_database):
    """Test person creation."""
    person = SimplePerson(name='Test User', age=25)
    db.session.add(person)
    db.session.commit()
    
    assert person.id is not None
    assert person.name == 'Test User'

def test_name_required(init_database):
    """Test that name is required."""
    with pytest.raises(IntegrityError):
        person = SimplePerson(age=25)
        db.session.add(person)
        db.session.commit()

# tests/test_forms.py
from app.forms import SimpleForm

def test_valid_form_data():
    """Test form validation with valid data."""
    form = SimpleForm(data={
        'name': 'Test User',
        'age': '25',
        'bio': 'Test bio'
    })
    assert form.validate() is True

def test_invalid_form_data():
    """Test form validation with invalid data."""
    form = SimpleForm(data={
        'name': '',  # Required field
        'age': 'not a number',
        'bio': 'Test bio'
    })
    assert form.validate() is False
    assert 'name' in form.errors
    assert 'age' in form.errors
```

**Learn More:**
- [Python Testing with pytest](https://www.youtube.com/watch?v=DhUpxWjOhME) - Comprehensive pytest tutorial
- [Flask Testing Best Practices](https://testdriven.io/blog/flask-pytest/) - Detailed guide on Flask testing

Key real-world additions explained:
1. Fixtures for reusable test components
2. Database testing with in-memory SQLite
3. Form validation testing
4. Model testing
5. Error case testing
6. CLI command testing
7. Proper test isolation
8. Type hints in tests
9. Comprehensive test coverage
10. Testing different response types

Testing Best Practices:
- Use fixtures for setup/teardown
- Test both success and failure cases
- Isolate tests from each other
- Use in-memory database for speed
- Mock external services
- Test form validation thoroughly
- Verify database state after operations
- Test API responses
- Check error handling
- Maintain test database isolation

## 7. Deployment Setup

```python
# Simple Version
# wsgi.py
from app import app

if __name__ == '__main__':
    app.run()
```

**What's happening here:**
- Basic WSGI entry point
- Development server setup
- No production configurations

**Real-world Deployment Setup:**

```python
# wsgi.py
import os
from app import create_app
from werkzeug.middleware.proxy_fix import ProxyFix

app = create_app(os.getenv('FLASK_ENV', 'production'))
app.wsgi_app = ProxyFix(app.wsgi_app, x_for=1, x_proto=1)

if __name__ == '__main__':
    app.run()
```

```python
# gunicorn_config.py
import multiprocessing
import os

# Gunicorn config
bind = f"0.0.0.0:{os.getenv('PORT', '8000')}"
workers = multiprocessing.cpu_count() * 2 + 1
threads = 2
worker_class = 'gevent'
worker_connections = 1000
timeout = 30
keepalive = 2

# Logging
accesslog = '-'
errorlog = '-'
loglevel = 'info'

# Security
limit_request_line = 4096
limit_request_fields = 100
limit_request_field_size = 8190
```

```ini
# Dockerfile
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV FLASK_ENV production

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Run gunicorn
CMD ["gunicorn", "--config", "gunicorn_config.py", "wsgi:app"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      - db
      - redis
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:13
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}

  redis:
    image: redis:6-alpine
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://web:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /app/static/;
        expires 30d;
    }

    location /media/ {
        alias /app/media/;
        expires 30d;
    }
}
```

**Learn More:**
- [Docker for Python Developers](https://testdriven.io/blog/docker-best-practices/) - Comprehensive Docker guide
- [Flask Production Deployment](https://www.youtube.com/watch?v=vXn11dJwg9s) - Excellent video on production deployment

Key real-world additions explained:

1. **Production Server Setup**
- Gunicorn as WSGI server
- Worker configuration
- Connection handling
- Timeout settings
- Logging configuration

2. **Docker Configuration**
- Multi-stage builds
- Security considerations
- Environment variables
- System dependencies
- Health checks

3. **Nginx Setup**
- Reverse proxy
- Static file serving
- SSL termination
- Header management
- Caching configuration

4. **Monitoring & Health**
```python
# app/routes/health.py
from flask import Blueprint, jsonify
from app.models import db

health_bp = Blueprint('health', __name__)

@health_bp.route('/health')
def health_check():
    try:
        # Check database connection
        db.session.execute('SELECT 1')
        db_status = True
    except Exception:
        db_status = False

    health_status = {
        'status': 'healthy' if db_status else 'unhealthy',
        'database': 'connected' if db_status else 'disconnected',
        'version': '1.0.0'
    }
    
    return jsonify(health_status)
```

5. **Environment Variables**
```bash
# .env.example
FLASK_ENV=production
SECRET_KEY=your-secret-key
DATABASE_URL=postgresql://user:pass@db:5432/dbname
REDIS_URL=redis://redis:6379/0
ALLOWED_HOSTS=example.com,www.example.com
```

Deployment Considerations:
1. Security
   - Secret management
   - SSL/TLS configuration
   - Header security
   - Rate limiting

2. Performance
   - Worker configuration
   - Database pooling
   - Caching strategy
   - Static file serving

3. Monitoring
   - Health checks
   - Error tracking
   - Performance monitoring
   - Log management

4. Scaling
   - Load balancing
   - Database replication
   - Cache distribution
   - Session handling

## 8. Templates and Jinja Setup

```python
# Simple Version (our practice assessment)
# app/templates/main_page.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <h1>Python Flask Practice Assessment</h1>
</body>
</html>

# app/templates/simple_form.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simple Form</title>
</head>
<body>
    <form method="post" action="/simple-form">
        {{ form.csrf_token }}
        <div>
            {{ form.name.label }}
            {{ form.name }}
        </div>
        <div>
            {{ form.age.label }}
            {{ form.age }}
        </div>
        <div>
            {{ form.bio.label }}
            {{ form.bio }}
        </div>
        {{ form.submit }}
    </form>
</body>
</html>
```

**What's happening here:**
- Basic HTML structure
- Simple form rendering
- CSRF protection
- Form field display

**Real-world Template Setup:**
```python
# app/templates/base.html
<!DOCTYPE html>
<html lang="en">
<head>
    {% block head %}
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Default Title{% endblock %} - My Site</title>
    
    {# CSS includes #}
    <link rel="stylesheet" href="{{ url_for('static', filename='css/main.css') }}">
    {% block extra_css %}{% endblock %}
    
    {# Favicon #}
    <link rel="icon" type="image/x-icon" href="{{ url_for('static', filename='favicon.ico') }}">
    {% endblock %}
</head>
<body>
    {# Flash messages #}
    {% with messages = get_flashed_messages(with_categories=true) %}
        {% if messages %}
            {% for category, message in messages %}
                <div class="alert alert-{{ category }}">{{ message }}</div>
            {% endfor %}
        {% endif %}
    {% endwith %}

    {# Main content #}
    <nav>
        {% block navigation %}
        <a href="{{ url_for('simple.index') }}">Home</a>
        <a href="{{ url_for('simple.simple_form') }}">Add Person</a>
        {% endblock %}
    </nav>

    <main>
        {% block content %}{% endblock %}
    </main>

    <footer>
        {% block footer %}
        &copy; {{ now.year }} My Site
        {% endblock %}
    </footer>

    {# JavaScript includes #}
    <script src="{{ url_for('static', filename='js/main.js') }}"></script>
    {% block extra_js %}{% endblock %}
</body>
</html>

# app/templates/simple_form.html
{% extends "base.html" %}

{% block title %}Add Person{% endblock %}

{% block content %}
<div class="form-container">
    <h1>Add New Person</h1>
    
    <form method="post" action="{{ url_for('simple.simple_form') }}" class="form">
        {{ form.csrf_token }}
        
        {% for field in [form.name, form.age, form.bio] %}
        <div class="form-group {% if field.errors %}has-error{% endif %}">
            {{ field.label(class="form-label") }}
            {{ field(class="form-input") }}
            
            {% if field.errors %}
                <div class="error-messages">
                    {% for error in field.errors %}
                        <span class="error">{{ error }}</span>
                    {% endfor %}
                </div>
            {% endif %}
        </div>
        {% endfor %}

        {{ form.submit(class="submit-button") }}
    </form>
</div>
{% endblock %}

{% block extra_js %}
<script>
document.addEventListener('DOMContentLoaded', function() {
    const form = document.querySelector('form');
    form.addEventListener('submit', function(e) {
        const nameField = document.querySelector('#name');
        if (!nameField.value.trim()) {
            e.preventDefault();
            alert('Name is required!');
        }
    });
});
</script>
{% endblock %}

# app/templates/macros/forms.html
{% macro render_field(field) %}
<div class="form-group {% if field.errors %}has-error{% endif %}">
    {{ field.label(class="form-label") }}
    {{ field(class="form-input")|safe }}
    
    {% if field.errors %}
        <div class="error-messages">
            {% for error in field.errors %}
                <span class="error">{{ error }}</span>
            {% endfor %}
        </div>
    {% endif %}
    
    {% if field.description %}
        <small class="field-help">{{ field.description }}</small>
    {% endif %}
</div>
{% endmacro %}

# app/templates/simple_form_data.html
{% extends "base.html" %}
{% from "macros/forms.html" import render_field %}

{% block title %}People Starting with M{% endblock %}

{% block content %}
<div class="data-container">
    <h1>People Starting with M</h1>
    
    {% if people %}
        {% for person in people %}
        <div class="person-card" data-id="{{ person.id }}">
            <div class="person-name">{{ person.name }}</div>
            <div class="person-age">{{ person.age }}</div>
            <div class="person-bio">{{ person.bio }}</div>
        </div>
        {% endfor %}
        
        {# Pagination #}
        {% if pagination %}
        <div class="pagination">
            {% for page in pagination.iter_pages() %}
                {% if page %}
                    <a href="{{ url_for('simple.simple_form_data', page=page) }}"
                       class="{% if page == pagination.page %}active{% endif %}">
                        {{ page }}
                    </a>
                {% else %}
                    <span class="ellipsis">...</span>
                {% endif %}
            {% endfor %}
        </div>
        {% endif %}
    {% else %}
        <p class="no-results">No people found whose names start with 'M'.</p>
    {% endif %}
</div>
{% endblock %}
```

**Learn More:**
- [Jinja2 Template Engine](https://www.youtube.com/watch?v=bxhXQG1qJPM) - Deep dive into Jinja2
- [Flask Template Best Practices](https://exploreflask.com/en/latest/templates.html)

Key real-world additions explained:
1. Template Inheritance
   - Base template
   - Block overrides
   - Macro usage

2. Error Handling
   - Form validation errors
   - Flash messages
   - Client-side validation

3. Organization
   - Macros for reusability
   - Structured blocks
   - Static file management

4. Features
   - Pagination
   - Dynamic content
   - Conditional rendering
   - URL generation

## 9. Database Migrations with Alembic/Flask-Migrate

```python
# Simple Version (Basic Setup)
# app/__init__.py
from flask import Flask
from flask_migrate import Migrate
from .models import db

app = Flask(__name__)
db.init_app(app)
migrate = Migrate(app, db)
```

Basic Commands:
```bash
# Initialize migrations
flask db init

# Create a migration
flask db migrate -m "Create simple_people table"

# Apply migration
flask db upgrade
```

**Real-world Migrations Setup:**

```python
# app/migrations/env.py
from logging.config import fileConfig
from sqlalchemy import engine_from_config
from alembic import context
from app.models import db

# Load model metadata for autogeneration
target_metadata = db.metadata

# Add custom logic for different environments
def get_url():
    from app.config import Configuration
    return Configuration.SQLALCHEMY_DATABASE_URI

def run_migrations_online():
    """Run migrations in 'online' mode."""
    configuration = context.config
    configuration.set_main_option('sqlalchemy.url', get_url())

    with engine_from_config(
        configuration.get_section(configuration.config_ini_section),
        prefix='sqlalchemy.',
        poolclass=pool.NullPool,
    ) as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            # Enable UUID type
            compare_type=True,
            # Enable checking constraints
            compare_server_default=True,
            # Include schemas
            include_schemas=True
        )

        with context.begin_transaction():
            context.run_migrations()
```

Example Migrations:
```python
# migrations/versions/123456789_create_simple_people.py
"""create simple_people table

Revision ID: 123456789
Revises: 
Create Date: 2024-01-15 10:00:00.000000
"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = '123456789'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Create table
    op.create_table(
        'simple_people',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('name', sa.String(length=50), nullable=False),
        sa.Column('age', sa.Integer(), nullable=True),
        sa.Column('bio', sa.String(length=2000), nullable=True),
        sa.PrimaryKeyConstraint('id')
    )
    # Add index
    op.create_index(
        'ix_simple_people_name',
        'simple_people',
        ['name']
    )

def downgrade():
    # Remove index
    op.drop_index('ix_simple_people_name')
    # Remove table
    op.drop_table('simple_people')
```

More Complex Migration Example:
```python
# migrations/versions/987654321_add_timestamps.py
"""add timestamps and soft delete

Revision ID: 987654321
Revises: 123456789
Create Date: 2024-01-15 11:00:00.000000
"""
from alembic import op
import sqlalchemy as sa
from datetime import datetime

revision = '987654321'
down_revision = '123456789'

def upgrade():
    # Add new columns
    op.add_column('simple_people',
        sa.Column('created_at', sa.DateTime(), nullable=False, 
                  server_default=sa.text('CURRENT_TIMESTAMP'))
    )
    op.add_column('simple_people',
        sa.Column('updated_at', sa.DateTime(), nullable=False,
                  server_default=sa.text('CURRENT_TIMESTAMP'),
                  onupdate=sa.text('CURRENT_TIMESTAMP'))
    )
    op.add_column('simple_people',
        sa.Column('is_deleted', sa.Boolean(), nullable=False,
                  server_default=sa.text('false'))
    )

def downgrade():
    # Remove columns
    op.drop_column('simple_people', 'is_deleted')
    op.drop_column('simple_people', 'updated_at')
    op.drop_column('simple_people', 'created_at')
```

Migration CLI Commands:
```bash
# Create new migration
flask db migrate -m "Add user table"

# Apply migrations
flask db upgrade

# Rollback one migration
flask db downgrade

# Show migration history
flask db history

# Show current migration
flask db current

# Mark migration without running
flask db stamp head

# Generate SQL without running
flask db upgrade --sql
```

Custom Migration Commands:
```python
# app/cli.py
import click
from flask.cli import with_appcontext
from app.models import db

@click.command('verify-migrations')
@with_appcontext
def verify_migrations_command():
    """Verify all migrations are applied."""
    from flask_migrate import get_current_revision
    current = get_current_revision()
    click.echo(f'Current migration: {current}')

@click.command('reset-db')
@with_appcontext
def reset_db_command():
    """Reset database (DANGEROUS)."""
    if click.confirm('Are you sure? This will delete all data!'):
        db.drop_all()
        db.create_all()
        click.echo('Database reset complete.')
```

Best Practices:
1. **Version Control**
   ```bash
   # .gitignore
   migrations/versions/*.py
   !migrations/versions/.gitkeep
   ```

2. **Data Migration**
   ```python
   def upgrade():
       # Get bind for raw SQL execution
       bind = op.get_bind()
       
       # Execute data updates
       bind.execute(
           "UPDATE simple_people SET bio = 'No bio' WHERE bio IS NULL"
       )
   ```

3. **Batch Processing**
   ```python
   def upgrade():
       bind = op.get_bind()
       result_proxy = bind.execute(
           "SELECT id FROM simple_people WHERE bio IS NULL"
       )
       
       # Process in batches
       batch_size = 1000
       while True:
           batch = result_proxy.fetchmany(batch_size)
           if not batch:
               break
           
           ids = [row[0] for row in batch]
           bind.execute(
               "UPDATE simple_people SET bio = 'No bio' "
               f"WHERE id IN ({','.join(map(str, ids))})"
           )
   ```

**Learn More:**
- [Alembic Official Documentation](https://alembic.sqlalchemy.org/en/latest/)
- [Flask-Migrate Tutorial](https://flask-migrate.readthedocs.io/en/latest/)
- [Database Migration Patterns](https://www.youtube.com/watch?v=0z2_h_VTJio)

Key Concepts:
1. Migration Generation
2. Upgrade/Downgrade Paths
3. Data Migration
4. Schema Changes
5. Index Management
6. Constraint Handling
7. Batch Processing
8. Version Control
9. Environment Management
10. Migration Testing

