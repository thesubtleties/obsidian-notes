Additional help for: https://appacademy.instructure.com/courses/319/pages/serving-html-from-flask-2?module_item_id=56601

## 1. Initial Setup

### Installation
```bash
pipenv install Flask~=1.1
pipenv install Jinja2~=2.11
pipenv install python-dotenv~=0.13
pipenv install Flask-WTF~=0.14
```
These packages work together to create a secure form system. Flask-WTF is a wrapper around WTForms that makes it play nicely with Flask.

### Project Structure
```plaintext
your_project/
├── app/
│   ├── __init__.py         # Flask application setup
│   ├── config.py           # Configuration settings
│   ├── sample_form.py      # Form classes
│   └── templates/
│       └── form.html       # Jinja template
└── .flaskenv               # Environment variables
```
This structure separates concerns: configuration, application logic, forms, and templates each have their own space.

## 2. Configuration Setup

### Environment Variables (.flaskenv)
```plaintext
FLASK_APP=app
FLASK_ENV=development
SECRET_KEY=super-secret-stuff
```
The SECRET_KEY is crucial for CSRF protection - think of it like a password that helps prevent form tampering.

### Configuration Class (app/config.py)
```python
import os

class Config(object):
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'default-key-for-devs'
```
This pattern allows for different keys in development and production. Never use the default key in production!

## 3. Basic Form Setup

### Form Class (app/sample_form.py)
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField

class SampleForm(FlaskForm):
    name = StringField('Name')
    submit = SubmitField('Save')
```
FlaskForm handles all the CSRF protection behind the scenes. Each field type corresponds to a different HTML input type.

### Application Setup (app/__init__.py)
```python
from flask import Flask, render_template
from app.config import Config
from app.sample_form import SampleForm

app = Flask(__name__)
app.config.from_object(Config)

@app.route('/')
def index():
    return '<h1>Simple App</h1><a href="/form">Form</a>'

@app.route('/form')
def form():
    form = SampleForm()
    return render_template('form.html', form=form)
```
The form instance is passed to the template, similar to passing props in React. The template can then access all form fields and helpers.

### Template Setup (app/templates/form.html)
```html
<!doctype html>
<html>
<head>
    <title>Sample Form</title>
</head>
<body>
    <h1>Sample Form</h1>
    <form action="" method="post" novalidate>
        {{ form.csrf_token }}
        <p>
            {{ form.name.label }}
            {{ form.name(size=32) }}
        </p>
        <p>{{ form.submit() }}</p>
    </form>
</body>
</html>
```
The csrf_token is automatically generated. The size=32 parameter sets the input field width - you can adjust this number as needed.

## 4. Available Form Fields

### Basic Fields
```python
from wtforms import (
    StringField,      # Text input
    PasswordField,    # Password input
    TextAreaField,    # Multi-line text input
    SubmitField      # Submit button
)
```
These are the most common field types you'll use. Each creates a different type of HTML input element.

### Numeric Fields
```python
from wtforms import (
    IntegerField,     # Whole numbers
    DecimalField,     # Precise decimal numbers
    FloatField        # Floating point numbers
)
```
These fields include built-in validation for their respective number types.

### Selection Fields
```python
from wtforms import (
    SelectField,           # Dropdown menu
    SelectMultipleField,   # Multi-select dropdown
    RadioField            # Radio buttons
)
```
These fields are great for when users need to choose from predefined options.

### Special Fields
```python
from wtforms import (
    FileField,            # File upload
    MultipleFileField,    # Multiple file upload
    DateField,            # Date picker
    DateTimeField,        # Date and time picker
    BooleanField          # Checkbox
)
```
These fields handle special input types like files, dates, and boolean values.

## 5. Form Usage Examples

### Basic String Input
```python
class UserForm(FlaskForm):
    username = StringField('Username', render_kw={"placeholder": "Enter username"})
```
The render_kw parameter lets you add HTML attributes to the input field.

### Selection Field with Choices
```python
class ColorForm(FlaskForm):
    color = SelectField('Favorite Color', 
                       choices=[('red', 'Red'), 
                               ('blue', 'Blue'), 
                               ('green', 'Green')])
```
For SelectField, each choice is a tuple of (value, label) where value is what's stored and label is what's displayed.

## 6. Common Patterns

### Form with Multiple Fields
```python
class RegistrationForm(FlaskForm):
    username = StringField('Username')
    password = PasswordField('Password')
    bio = TextAreaField('About Me')
    submit = SubmitField('Register')
```
Group related fields together in a single form class. This keeps your form logic organized.

### Custom Field Sizes
```html
{{ form.username(size=32) }}         <!-- Width of 32 characters -->
{{ form.bio(rows=4, cols=50) }}      <!-- 4 rows, 50 columns -->
```
Customize input sizes to match your design needs. Different field types support different attributes.

## 7. Quick Reference

### Field Types
```python
# Text inputs
StringField()        # Single line text
TextAreaField()      # Multi-line text
PasswordField()      # Password field

# Numbers
IntegerField()       # Whole numbers
DecimalField()       # Decimal numbers
FloatField()         # Floating point

# Selections
SelectField()        # Dropdown
RadioField()         # Radio buttons
BooleanField()       # Checkbox

# Special
FileField()         # File upload
DateField()         # Date picker
```

Remember:
- Always include CSRF token in forms
- Keep SECRET_KEY secure
- Use appropriate field types for data
- Consider validation needs
- Group related fields together
- Use meaningful field labels
- Consider user experience when choosing field types
