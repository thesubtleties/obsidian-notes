## `__repr__` and `__str__` Methods
```python
class Project(db.Model):
    __tablename__ = 'projects'
    
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(100))
    status = db.Column(db.String(50))
    
    def __repr__(self):
        """
        Developer-friendly representation
        Used in debugging, console, logs
        """
        return f'<Project {self.id}: {self.name}>'
    
    def __str__(self):
        """
        User-friendly representation
        Used when str() is called or print()
        """
        return self.name

class Task(db.Model):
    __tablename__ = 'tasks'
    
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    status = db.Column(db.String(50))
    feature_id = db.Column(db.Integer, db.ForeignKey('features.id'))
    
    def __repr__(self):
        """Include relationship info in repr"""
        return f'<Task {self.id}: {self.title} (Feature: {self.feature_id})>'

class User(db.Model):
    __tablename__ = 'users'
    
    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(255))
    
    def __repr__(self):
        """Careful not to expose sensitive data in repr"""
        return f'<User {self.id}: {self.email.split("@")[0]}...>'
```

Usage and Benefits:
```python
# In Flask shell or debugger
>>> project = Project.query.first()
>>> print(project)  # Uses __str__
"Marketing Campaign"

>>> project  # Uses __repr__
<Project 1: Marketing Campaign>

>>> Project.query.all()  # List of reprs
[<Project 1: Marketing Campaign>, <Project 2: Website Redesign>]

# In logging
logging.debug(f"Processing {project}")  # Clean log output

# In development
print(f"Task created: {new_task}")  # Easy debugging
```

Key Points:
1. `__repr__` is for developers:
   - Include useful debugging info
   - Use angle brackets by convention
   - Include ID and key identifiers
   - Don't expose sensitive data

2. `__str__` is for users:
   - More readable format
   - Just the essential info
   - Used by print() and str()
   - Falls back to `__repr__` if not defined

3. Best Practices:
   - Always implement `__repr__`
   - Make it informative but concise
   - Include relationship IDs for debugging
   - Consider security in representations
   - Keep it maintainable

4. Common Patterns:
```python
# Basic pattern
def __repr__(self):
    return f'<{self.__class__.__name__} {self.id}>'

# With multiple fields
def __repr__(self):
    return f'<{self.__class__.__name__} {self.id}: {self.name} ({self.status})>'

# With relationships
def __repr__(self):
    return f'<{self.__class__.__name__} {self.id} owned by User {self.owner_id}>'

# Safe representation
def __repr__(self):
    return f'<{self.__class__.__name__} {self.id}: {self.public_identifier}>'
```

This makes debugging and development much easier without affecting production functionality!