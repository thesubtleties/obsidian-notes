## Docker Configuration

### Docker Setup
```dockerfile
# Dockerfile
FROM python:3.9-slim

# Security: Run as non-root user
RUN useradd -m myapp
WORKDIR /home/myapp

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application
COPY . .
RUN chown -R myapp:myapp ./

# Switch to non-root user
USER myapp

# Run script
COPY init.sh .
RUN chmod +x init.sh
CMD ["./init.sh"]
```

### Initialization Script
```bash
# init.sh
#!/bin/sh
set -e

echo "Waiting for postgres..."
while ! nc -z db 5432; do
  sleep 1
done
echo "PostgreSQL started"

# Apply migrations
flask db upgrade

# In production, don't automatically seed
if [ "$FLASK_ENV" = "development" ]; then
  echo "Running seeds..."
  flask seed undo
  flask seed all
fi

# Start application
if [ "$FLASK_ENV" = "development" ]; then
  flask run --host=0.0.0.0
else
  gunicorn --bind 0.0.0.0:5000 \
    --workers=4 \
    --threads=2 \
    --timeout=60 \
    --access-logfile=- \
    --error-logfile=- \
    "app:create_app()"
fi
```

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    environment:
      - FLASK_APP=app
      - FLASK_ENV=development
      - DATABASE_URL=postgresql://user:password@db:5432/dbname
      - SECRET_KEY=dev-secret-change-in-production
    depends_on:
      db:
        condition: service_healthy
    ports:
      - "5000:5000"

  db:
    image: postgres:14
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=dbname
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d dbname"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_USER: test_user
          POSTGRES_PASSWORD: test_password
          POSTGRES_DB: test_db
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v2
    
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.9'
        
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
        pip install pytest pytest-cov
        
    - name: Run tests
      env:
        DATABASE_URL: postgresql://test_user:test_password@localhost:5432/test_db
        FLASK_ENV: testing
      run: |
        pytest --cov=app tests/
        
    - name: Upload coverage
      uses: codecov/codecov-action@v2

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    
    steps:
    - name: Deploy to production
      env:
        RENDER_API_KEY: ${{ secrets.RENDER_API_KEY }}
      run: |
        curl -X POST "https://api.render.com/v1/services/$SERVICE_ID/deploys" \
          -H "Authorization: Bearer $RENDER_API_KEY"
```

## Production Database Management

### Migration Script
```python
# migrations/env.py
import os
import sys
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

# Add application to path
sys.path.append(os.path.dirname(os.path.dirname(__file__)))

# Import models
from app.models import *
from app.models.db import SCHEMA

# Get database URL
config = context.config
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

def run_migrations_online():
    """Run migrations in 'online' mode."""
    
    def process_revision_directives(context, revision, directives):
        if getattr(config.cmd_opts, 'autogenerate', False):
            script = directives[0]
            if script.upgrade_ops.is_empty():
                directives[:] = []

    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix='sqlalchemy.',
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        if os.environ.get("FLASK_ENV") == "production":
            connection.execute(f"CREATE SCHEMA IF NOT EXISTS {SCHEMA}")
            
        context.configure(
            connection=connection,
            target_metadata=Base.metadata,
            process_revision_directives=process_revision_directives,
            include_schemas=True,
            version_table_schema=SCHEMA if os.environ.get("FLASK_ENV") == "production" else None
        )

        with context.begin_transaction():
            if os.environ.get("FLASK_ENV") == "production":
                context.execute(f"SET search_path TO {SCHEMA}")
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

## Monitoring and Logging

### Logging Configuration
```python
# app/utils/logging_config.py
import logging
import json
from datetime import datetime
from pythonjsonlogger import jsonlogger

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    def add_fields(self, log_record, record, message_dict):
        super(CustomJsonFormatter, self).add_fields(log_record, record, message_dict)
        log_record['timestamp'] = datetime.utcnow().isoformat()
        log_record['level'] = record.levelname
        log_record['environment'] = os.getenv('FLASK_ENV', 'development')

def setup_logging():
    """Configure application logging"""
    logger = logging.getLogger()
    handler = logging.StreamHandler()
    
    formatter = CustomJsonFormatter(
        '%(timestamp)s %(level)s %(name)s %(message)s'
    )
    handler.setFormatter(formatter)
    
    logger.addHandler(handler)
    logger.setLevel(logging.INFO if os.getenv('FLASK_ENV') == 'production' else logging.DEBUG)

# Usage in routes
logger = logging.getLogger(__name__)

@sprint_routes.route("/projects/<int:project_id>/sprints")
@login_required
def get_sprints(project_id):
    logger.info('Fetching sprints', extra={
        'project_id': project_id,
        'user_id': current_user.id,
        'endpoint': request.endpoint
    })
    # ... route logic ...
```

[Would you like me to continue with Part 8: Performance Optimization and Scaling?]

This will cover:
- Database optimization
- Caching strategies
- Load balancing
- Async tasks
- Memory management
- Error recovery

