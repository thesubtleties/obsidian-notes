## Security and Environment Configuration

### Environment Configuration
```python
# config.py
import os
from datetime import timedelta

class Config:
    """Base configuration"""
    # Basic Flask config
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-key-please-change'
    
    # Database URLs
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
    if SQLALCHEMY_DATABASE_URI and SQLALCHEMY_DATABASE_URI.startswith('postgres://'):
        SQLALCHEMY_DATABASE_URI = SQLALCHEMY_DATABASE_URI.replace('postgres://', 'postgresql://')
    
    # Security settings
    SESSION_COOKIE_SECURE = True
    SESSION_COOKIE_HTTPONLY = True
    SESSION_COOKIE_SAMESITE = 'Lax'
    
    # CORS settings
    CORS_ORIGINS = os.environ.get('CORS_ORIGINS', '*').split(',')
    CORS_METHODS = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS']

class DevelopmentConfig(Config):
    """Development configuration"""
    DEBUG = True
    SQLALCHEMY_ECHO = True  # Log SQL queries
    SESSION_COOKIE_SECURE = False  # Allow HTTP in development

class ProductionConfig(Config):
    """Production configuration"""
    # Additional security headers
    SECURITY_HEADERS = {
        'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
        'X-Content-Type-Options': 'nosniff',
        'X-Frame-Options': 'SAMEORIGIN',
        'X-XSS-Protection': '1; mode=block'
    }
```

### Security Middleware
```python
# app/middleware/security.py
from functools import wraps
from flask import request, current_app
import re

def rate_limit(max_requests=100, window=timedelta(minutes=1)):
    """Rate limiting decorator"""
    def decorator(f):
        @wraps(f)
        def decorated_function(*args, **kwargs):
            # Get client identifier (IP or user ID)
            client_id = request.headers.get('X-Real-IP') or request.remote_addr
            
            # Check rate limit in Redis/cache
            key = f'rate_limit:{client_id}:{f.__name__}'
            current = cache.get(key) or 0
            
            if current >= max_requests:
                return {
                    'message': 'Too many requests',
                    'retry_after': window.seconds
                }, 429
                
            # Increment counter
            cache.incr(key)
            if not cache.ttl(key):
                cache.expire(key, window.seconds)
                
            return f(*args, **kwargs)
        return decorated_function
    return decorator

def sanitize_input(f):
    """Sanitize request input"""
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if request.is_json:
            # Deep clean JSON data
            request._cached_json = deep_clean(request.get_json())
        return f(*args, **kwargs)
    return decorated_function

def deep_clean(data):
    """Recursively clean data"""
    if isinstance(data, str):
        # Remove potential XSS
        data = re.sub(r'<[^>]*?>', '', data)
        # Remove SQL injection attempts
        data = re.sub(r'(\b(SELECT|INSERT|UPDATE|DELETE|DROP|UNION)\b)', '', data, flags=re.IGNORECASE)
        return data
    elif isinstance(data, dict):
        return {k: deep_clean(v) for k, v in data.items()}
    elif isinstance(data, list):
        return [deep_clean(x) for x in data]
    return data
```

### Production-Ready Route Example
```python
# app/api/routes/sprint_routes.py
@sprint_routes.route("/projects/<int:project_id>/sprints", methods=["POST"])
@login_required
@require_project_access
@rate_limit(max_requests=60, window=timedelta(minutes=1))
@sanitize_input
@log_route_errors
@monitor_performance
def create_sprint(project_id):
    """Production-ready route with all safeguards"""
    try:
        data = request.json
        errors = {}

        # Validation with timeouts
        with timeout(seconds=5):  # Prevent long-running validations
            if not data.get("name"):
                errors["name"] = "Name is required"

        if errors:
            return create_error_response("Validation error", errors), 400

        # Database operations with retry
        for attempt in range(3):
            try:
                with db_transaction():
                    new_sprint = Sprint.create_sprint(
                        project_id=project_id,
                        name=data.get("name"),
                        start_date=data.get("start_date"),
                        end_date=data.get("end_date")
                    )
                break
            except SQLAlchemyError as e:
                if attempt == 2:  # Last attempt
                    logger.error(f"Database error after 3 attempts: {e}")
                    raise
                continue

        # Cache invalidation
        cache.delete(f'project_sprints:{project_id}')
        
        # Async task for notifications
        send_sprint_notification.delay(
            project_id=project_id,
            sprint_id=new_sprint.id
        )

        return new_sprint.to_dict(), 201

    except TimeoutError:
        return create_error_response("Request timeout"), 408
    except Exception as e:
        logger.exception("Unexpected error in create_sprint")
        return create_error_response("Internal server error"), 500
```
