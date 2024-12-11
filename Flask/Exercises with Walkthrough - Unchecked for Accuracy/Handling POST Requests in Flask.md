Additional help for: https://appacademy.instructure.com/courses/319/pages/handling-posts-2?module_item_id=56602

## 1. Understanding HTTP Methods

### GET vs POST
```plaintext
GET:
- Data in URL (max 2,048 characters)
- Bookmarkable
- Good for searches
- Visible in browser history

POST:
- Data in request body
- Can handle large amounts of data
- Can send files
- More secure for sensitive data
```
Understanding these differences helps you choose the right method for your forms.

## 2. Basic Setup

### Route Configuration
```python
from flask import Flask, render_template, redirect

@app.route('/form', methods=['GET', 'POST'])
def form():
    form = SampleForm()
    if form.validate_on_submit():
        return redirect('/')
    return render_template('form.html', form=form)
```
The `methods` list tells Flask this route can handle both GET and POST requests. Without it, only GET is allowed.

### Form Template
```html
<form action="" method="post" novalidate>
    {{ form.hidden_tag() }}
    <p>
        {{ form.name.label }}
        {{ form.name(size=32) }}
    </p>
    <p>{{ form.submit() }}</p>
</form>
```
The `hidden_tag()` includes CSRF protection. Without it, your form won't validate for security reasons.

## 3. Form Validation

### Basic Validator Setup
```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired

class SampleForm(FlaskForm):
    name = StringField('Name', validators=[DataRequired()])
    submit = SubmitField('Save')
```
Validators ensure data meets your requirements before processing. `DataRequired()` is just one of many available validators.

### Common Validators
```python
from wtforms.validators import (
    DataRequired,    # Field cannot be empty
    Email,          # Must be valid email format
    Length,         # Control string length
    NumberRange,    # Numeric range validation
    EqualTo        # Must match another field (like password confirmation)
)
```
Choose validators based on your data requirements. Multiple validators can be used on one field.

## 4. Handling Form Submission

### Basic Flow
```python
@app.route('/form', methods=['GET', 'POST'])
def form():
    form = SampleForm()
    if form.validate_on_submit():
        # Form is valid and was submitted via POST
        return redirect('/')
    # Either GET request or invalid form
    return render_template('form.html', form=form)
```
`validate_on_submit()` checks both the HTTP method and validation rules.

### Accessing Form Data
```python
@app.route('/form', methods=['GET', 'POST'])
def form():
    form = SampleForm()
    if form.validate_on_submit():
        name = form.name.data  # Get data from form
        # Do something with the data
        return redirect('/')
    return render_template('form.html', form=form)
```
Form data is available through the `.data` attribute of each field.

## 5. Redirecting After Success

### Basic Redirect
```python
from flask import redirect

@app.route('/form', methods=['GET', 'POST'])
def form():
    if form.validate_on_submit():
        return redirect('/')  # Go to home page
```
Redirecting after successful form submission prevents duplicate submissions.

### Redirect with Flash Messages
```python
from flask import flash, redirect

@app.route('/form', methods=['GET', 'POST'])
def form():
    if form.validate_on_submit():
        flash('Form submitted successfully!')
        return redirect('/')
```
Flash messages provide user feedback across redirects.

## 6. Complete Example

### Full Application
```python
from flask import Flask, render_template, redirect
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField
from wtforms.validators import DataRequired

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-secret-key'

class SampleForm(FlaskForm):
    name = StringField('Name', validators=[DataRequired()])
    submit = SubmitField('Save')

@app.route('/')
def index():
    return '<h1>Simple App</h1><a href="/form">Form</a>'

@app.route('/form', methods=['GET', 'POST'])
def form():
    form = SampleForm()
    if form.validate_on_submit():
        # Process form data here
        return redirect('/')
    return render_template('form.html', form=form)
```
This shows all the pieces working together.

## 7. Common Gotchas

### Form Not Validating
```plaintext
Check for:
1. Missing SECRET_KEY
2. Missing hidden_tag()
3. Incorrect form method
4. Missing CSRF token
```

### Redirect Not Working
```plaintext
Check for:
1. Import redirect
2. Return the redirect
3. Valid route in redirect
```

## 8. Quick Reference

### Route Methods
```python
@app.route('/path')               # GET only
@app.route('/path', methods=['GET'])     # Same as above
@app.route('/path', methods=['POST'])    # POST only
@app.route('/path', methods=['GET', 'POST'])  # Both
```

### Validation Flow
```python
form.validate_on_submit()  # Returns True if:
                          # 1. Request is POST
                          # 2. All validators pass
```

Remember:
- Always use CSRF protection
- Redirect after successful POST
- Validate form data
- Handle both GET and POST methods
- Use appropriate validators
- Provide user feedback
- Consider security implications

