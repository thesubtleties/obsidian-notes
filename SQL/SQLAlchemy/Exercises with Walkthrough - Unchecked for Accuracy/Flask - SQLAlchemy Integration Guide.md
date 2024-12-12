Additional help for: https://appacademy.instructure.com/courses/319/pages/using-sqlalchemy-with-flask-2?module_item_id=56621

## 1. Initial Setup

### Installation
```bash
# Basic installation
pipenv install Flask-SQLAlchemy

# Complete setup with all dependencies
pipenv install Flask SQLAlchemy Flask-SQLAlchemy
```
Flask-SQLAlchemy combines Flask and SQLAlchemy, making database operations more Flask-friendly.

### Project Structure
```plaintext
your_project/
├── app/
│   ├── __init__.py      # Flask app and DB setup
│   ├── models.py        # Database models
│   ├── routes.py        # View functions
│   └── templates/       # Jinja templates
└── config.py           # Configuration
```
This structure separates concerns and follows Flask conventions.

## 2. Basic Configuration

### Initial Setup (__init__.py)
```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
from config import Config

# Create Flask app
app = Flask(__name__)

# Load config
app.config.from_object(Config)

# Create DB object
db = SQLAlchemy(app)
```
The `db` object becomes your interface to SQLAlchemy - it includes all the tools you need.

### Configuration (config.py)
```python
class Config:
    # Required
    SQLALCHEMY_DATABASE_URI = 'sqlite:///dev.db'
    
    # Optional but recommended
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_ECHO = True  # Log SQL queries
```
Configuration settings control database behavior and performance options.

## 3. Defining Models

### Basic Model Structure
```python
from app import db

class User(db.Model):
    __tablename__ = 'users'  # Optional: Flask-SQLAlchemy can guess
    
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True)
    email = db.Column(db.String(120))
```
Models inherit from `db.Model` instead of declarative_base() - Flask-SQLAlchemy handles this for you.

### Relationships
```python
class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    user_id = db.Column(db.Integer, db.ForeignKey('users.id'))
    
    # Relationship with backref
    author = db.relationship('User', backref='posts')
```
Relationships work the same as SQLAlchemy but use the `db` object.

## 4. Database Operations

### Query Interface
```python
# Flask-SQLAlchemy adds .query to models
@app.route('/users')
def user_list():
    # Get all users
    users = User.query.all()
    
    # Get specific user
    user = User.query.get(1)
    
    # Filter users
    admins = User.query.filter_by(role='admin').all()
```
The `.query` attribute is a Flask-SQLAlchemy convenience - no session needed.

### Common Operations
```python
# Create
new_user = User(username='john')
db.session.add(new_user)
db.session.commit()

# Update
user = User.query.get(1)
user.username = 'john_doe'
db.session.commit()

# Delete
db.session.delete(user)
db.session.commit()
```
Use `db.session` instead of creating your own Session.

## 5. Convenience Features

### Error Handling
```python
# Instead of:
user = User.query.get(1)
if not user:
    abort(404)

# Use:
user = User.query.get_or_404(1)

# Or:
user = User.query.first_or_404()
```
Flask-SQLAlchemy provides shortcuts for common patterns.

### Pagination
```python
@app.route('/users')
def list_users():
    page = request.args.get('page', 1, type=int)
    users = User.query.paginate(
        page=page, 
        per_page=20,
        error_out=False
    )
    return render_template('users.html', users=users)
```
Built-in pagination support for large datasets.

## 6. Complete Example

### Application Factory Pattern
```python
# app/__init__.py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    app.config.from_object('config.Config')
    
    db.init_app(app)
    
    with app.app_context():
        from . import routes
        db.create_all()
        
    return app
```

### Models
```python
# app/models.py
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80))
    posts = db.relationship('Post', backref='author')

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(120))
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))
```

### Routes
```python
# app/routes.py
from flask import render_template
from app import db
from app.models import User, Post

@app.route('/user/<int:id>')
def user_detail(id):
    user = User.query.get_or_404(id)
    return render_template('user.html', user=user)

@app.route('/post/create', methods=['POST'])
def create_post():
    post = Post(**request.form)
    db.session.add(post)
    db.session.commit()
    return redirect(url_for('post_detail', id=post.id))
```

## 7. Best Practices

### Session Management
```python
# Use context managers for safety
def create_user(data):
    try:
        user = User(**data)
        db.session.add(user)
        db.session.commit()
        return user
    except:
        db.session.rollback()
        raise
```

### Query Organization
```python
# Add model methods for common queries
class User(db.Model):
    @classmethod
    def get_active_users(cls):
        return cls.query.filter_by(active=True)
```

Remember:
- Use Flask-SQLAlchemy's conveniences
- Handle database errors appropriately
- Use proper configuration
- Follow Flask patterns
- Keep models organized
- Use relationships effectively
- Consider performance implications
