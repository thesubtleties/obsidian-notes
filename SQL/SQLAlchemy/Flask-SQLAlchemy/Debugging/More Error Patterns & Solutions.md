## Frontend-Backend Connection Issues

### 401 UNAUTHORIZED on /api/auth/
**Symptom:**
```
Failed to load resource: the server responded with a status of 401 (UNAUTHORIZED)
```
**Cause:** This is actually EXPECTED for `/api/auth/` when not logged in - it's checking authentication status.

**Solution:**
- Frontend should handle 401 gracefully
- This is normal for initial auth check

### 400 BAD REQUEST on Login
**Symptom:**
```
POST http://127.0.0.1:5173/api/auth/login 400 (BAD REQUEST)
```
**Cause:** Usually CSRF token issues or form validation

**Solution:**
```python
# Backend: Make sure CSRF is set up
@app.after_request
def inject_csrf_token(response):
    response.set_cookie(
        'csrf_token',
        generate_csrf(),
        secure=True if os.environ.get('FLASK_ENV') == 'production' else False,
        samesite='Strict' if os.environ.get('FLASK_ENV') == 'production' else None,
        httponly=True
    )
    return response

# Frontend: Include CSRF token
const response = await fetch('/api/auth/login', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'XSRF-TOKEN': Cookies.get('XSRF-TOKEN')
    },
    credentials: 'include',
    body: JSON.stringify(loginData)
});
```

### Invalid JSON Response
**Symptom:**
```
Uncaught (in promise) SyntaxError: Unexpected token '<', "<!doctype "... is not valid JSON
```
**Cause:** Getting HTML instead of JSON response

**Solution:**
```python
# Make sure route returns JSON
@auth_routes.route('/login', methods=['POST'])
def login():
    try:
        # ... login logic ...
        return jsonify({
            "user": user.to_dict()
        })
    except Exception as e:
        return jsonify({"errors": str(e)}), 400
```

## Database Issues

### "relation does not exist"
**Symptom:**
```
psycopg2.errors.UndefinedTable: relation "users" does not exist
```
**Cause:** Database tables not created/migrations not run

**Solution:**
```bash
# In Docker container
docker exec -it backend sh

# Then run full migration sequence
flask db downgrade base
rm -rf migrations/
flask db init
flask db migrate -m "initial migration"
flask db upgrade
flask seed all
```

### Hot Reload & Development Flow
**Question:** "Do I need to rebuild for API changes?"

**Answer:** No, if volumes are mounted correctly:
```yaml
# docker-compose.yml
volumes:
  - ./app:/app
```

**When you DO need to rebuild:**
1. Changes to requirements.txt
2. Changes to Dockerfile
3. Changes to docker-compose.yml
4. Environment variable changes

```bash
# For those cases:
docker compose down
docker compose up --build
```

### Database Connection Verification
**How to verify database state:**
```python
# In Flask shell
flask shell

# Check tables
from models import db
db.engine.table_names()

# Check relationships
from models import User, Project
user = User.query.first()
print(user.to_dict())
```

### Blueprint Registration Issues
**Symptom:** Routes not found or 404 errors

**Solution:** Make sure blueprints are registered with correct prefixes:
```python
api = Blueprint("api", __name__)

api.register_blueprint(auth_routes, url_prefix="/auth")
api.register_blueprint(project_routes, url_prefix="/projects")
api.register_blueprint(feature_routes, url_prefix="/features")
api.register_blueprint(sprint_routes, url_prefix="/sprints")
api.register_blueprint(task_routes, url_prefix="/tasks")
```

This creates proper route structure:
```
/api/auth/login
/api/projects/
/api/features/
etc...
```

