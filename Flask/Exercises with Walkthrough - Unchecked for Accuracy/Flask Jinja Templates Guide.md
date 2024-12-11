Additional help for: https://appacademy.instructure.com/courses/319/pages/jinja-templates-2?module_item_id=56600

## 1. Basic Setup

### Installation
```bash
pipenv install Jinja2~=3.0.3
```
The version number is specific to ensure compatibility with Flask. Always check your Flask version's recommended Jinja2 version if you run into issues.

### Project Structure
```plaintext
your_app/
├── templates/          # Required folder name
│   ├── index.html
│   ├── nav.html
│   └── copyright.html
└── app.py
```
Flask specifically looks for the 'templates' directory name - it won't work if you name it anything else. This is similar to how React looks for 'src' or how Django looks for 'templates'.

### Initial Flask Setup
```python
from flask import Flask, render_template

app = Flask(__name__)
```
render_template is Flask's built-in function for processing Jinja templates. You'll use this instead of returning raw HTML strings from your routes.

## 2. Basic Template Usage

### Without Templates (The Old Way)
```python
@app.route('/')
def index():
    return '''
        <!doctype html>
        <html>
            <head><title>Home</title></head>
            <body><h1>Home</h1></body>
        </html>
    '''
```
This approach quickly becomes a maintenance nightmare. Imagine having multiple routes, each with complex HTML structures - your Python files would become unreadable.

### With Templates (The Better Way)
```python
# app.py
@app.route('/')
def index():
    return render_template('index.html')
```
Now your route is clean and focused on its primary job: handling the request and deciding what template to render.

```html
<!-- templates/index.html -->
<!doctype html>
<html>
    <head><title>Home</title></head>
    <body><h1>Home</h1></body>
</html>
```
Your HTML lives where it belongs - in a template file. This separation makes both your Python and HTML code easier to maintain and understand.

## 3. Template Variables

### Basic Variable Usage
```python
# app.py
@app.route('/')
def home():
    return render_template(
        'index.html',
        sitename='My Sample',
        page="Home"
    )
```
Variables are passed as keyword arguments to render_template. This is similar to passing props in React - you're sending data from your Python code to your template.

```html
<!-- templates/index.html -->
<title>{{ page }} - {{ sitename }}</title>
<h1>{{ sitename }}</h1>
<h2>{{ page }}</h2>
```
Double curly braces tell Jinja "this is a variable - replace it with its value." The spaces inside the braces are a convention that makes templates more readable.

## 4. Control Structures

### Conditionals
```html
{% if not logged_in %}
    <a href="/login">Log in</a>
{% endif %}
```
The {% %} syntax is for control flow statements. This works just like Python's if statements, but needs to be wrapped in these special tags so Jinja knows it's not regular HTML.

### Loops
```python
# app.py
nav = [
    {'href': 'https://appacademy.io', 'caption': 'App Academy'},
    {'href': 'https://archive.org', 'caption': 'Internet Archive'},
]
return render_template("page.html", navigation=nav)
```
You can pass any Python data structure to your template. Here we're passing a list of dictionaries that will be used to build a navigation menu.

```html
<ul>
    {% for item in navigation %}
        <li>
            <a href="{{ item.href }}">{{ item.caption }}</a>
        </li>
    {% endfor %}
</ul>
```
Jinja's for loops work just like Python's. Notice how we can access dictionary keys using dot notation in the template.

## 5. Template Reuse

### Include Statement
```html
<!-- templates/nav.html -->
<a href="/">Home</a> | <a href="/about">About</a>
```
This is a reusable component that can be included in any other template. Keep these files focused on one specific piece of UI.

```html
<!-- templates/copyright.html -->
&copy; 2020 Me, myself, and I. All rights reserved.
```
Another reusable component - perfect for things like footers, headers, or any repeated UI elements.

```html
<!-- templates/index.html -->
<!doctype html>
<html>
<head>
    <title>{{ page }} - {{ sitename }}</title>
</head>
<body>
    <h1>{{ sitename }}</h1>
    {% include 'nav.html' %}
    <p>Content here...</p>
    {% include 'copyright.html' %}
</body>
</html>
```
The include statement pulls in the content from other template files. This helps keep your templates DRY (Don't Repeat Yourself).

## 6. Complete Example

### Application Structure
```python
# app.py
from flask import Flask, render_template

app = Flask(__name__)

@app.route('/')
def home():
    return render_template(
        'index.html',
        sitename='My Sample',
        page="Home"
    )

@app.route('/about')
def about():
    return render_template(
        'index.html',
        sitename='My Sample',
        page="About"
    )
```
Notice how both routes use the same template but pass different values for 'page'. This is a common pattern for maintaining consistent layout while changing content.

## 7. Quick Reference

### Template Syntax
```html
{{ variable }}              <!-- Output a variable -->
{% if condition %}          <!-- Start a conditional block -->
{% for x in sequence %}     <!-- Start a loop -->
{% include 'file.html' %}   <!-- Include another template -->
```
These are the most common Jinja template tags you'll use. The syntax is designed to be visible and distinct from regular HTML.

### Common Patterns
```python
# Multiple variables
render_template('template.html',
    var1='value1',
    var2='value2'
)

# Dictionary of variables
context = {
    'var1': 'value1',
    'var2': 'value2'
}
render_template('template.html', **context)
```
Using a context dictionary is helpful when you have many variables to pass. The ** operator unpacks the dictionary into keyword arguments.

Remember:
- Templates must be in the 'templates' folder
- Template files should have .html extension
- Variable names in templates are case-sensitive
- Control structures need both opening and closing tags
- Keep templates focused on presentation logic
- Use includes for reusable components
- Template paths are relative to the templates folder

