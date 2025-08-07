# Semgrep/OpenGrep Bible: Pattern-Based Code Analysis for Beginners

#security #code-analysis #static-analysis #semgrep #patterns #automation

## Table of Contents

1. [[#What is Semgrep?]]
2. [[#Core Concepts]]
3. [[#Installation & Setup]]
4. [[#Basic Pattern Syntax]]
5. [[#Writing Your First Rules]]
6. [[#Pattern Operators]]
7. [[#Metavariables]]
8. [[#Advanced Patterns]]
9. [[#Security Scanning]]
10. [[#Custom Rule Development]]
11. [[#CI/CD Integration]]
12. [[#Real-World Examples]]
13. [[#Performance Tips]]
14. [[#Troubleshooting]]
15. [[#Reference & Resources]]

## What is Semgrep?

Semgrep (Semantic Grep) is a fast, open-source, static analysis tool that finds bugs, security vulnerabilities, and anti-patterns in your code. Think of it as grep, but it understands code structure rather than just text.

### Key Differences from Regular Grep

| Feature | Grep | Semgrep |
|---------|------|---------|
| Search Type | Text/Regex | Semantic/AST-based |
| Language Aware | No | Yes (30+ languages) |
| Ignores | Nothing | Comments, whitespace, formatting |
| Pattern Matching | Line by line | Code structure |
| False Positives | High | Low |

### When to Use Semgrep

**Perfect for:**
- Finding security vulnerabilities
- Enforcing coding standards
- Large-scale refactoring
- Code review automation
- Technical debt identification

**Not ideal for:**
- Simple text searches (use grep)
- Runtime analysis (use debuggers)
- Complex data flow analysis (use specialized tools)

## Core Concepts

### 1. Pattern-Based Analysis
Semgrep looks for code patterns, not text. This code:
```python
if user.is_admin() == True:
    do_something()
```

Matches this pattern:
```yaml
pattern: $X == True
```

Even if written as:
```python
if (
    user.is_admin()
    ==
    True
):
    do_something()
```

### 2. Language-Agnostic Patterns
Write one pattern that works across languages:
```yaml
pattern: console.log(...)  # Works in JavaScript
pattern: print(...)        # Works in Python
pattern: System.out.println(...)  # Works in Java
```

### 3. Semantic Understanding
Semgrep understands that these are equivalent:
```python
x = x + 1
x += 1
```

## Installation & Setup

### Quick Install

**macOS/Linux:**
```bash
# Using pip
pip install semgrep

# Using brew (macOS)
brew install semgrep

# Using Docker
docker pull returntocorp/semgrep
```

**Windows:**
```bash
# WSL recommended
pip install semgrep

# Or use Docker
docker run --rm -v "%cd%:/src" returntocorp/semgrep
```

### First Run
```bash
# Test installation
semgrep --version

# Run default rules on current directory
semgrep --config=auto .

# Run specific rule file
semgrep --config=my-rules.yaml .
```

## Basic Pattern Syntax

### Simple Patterns

**1. Exact Match:**
```yaml
rules:
  - id: hardcoded-password
    pattern: password = "..."
    message: Hardcoded password found
    languages: [python]
    severity: ERROR
```

**2. Function Calls:**
```yaml
pattern: eval(...)  # Matches any eval() call
```

**3. Method Calls:**
```yaml
pattern: $OBJ.dangerous_method(...)
```

### Pattern Components

```yaml
rules:
  - id: rule-name           # Unique identifier
    pattern: ...             # What to match
    message: ...             # What to report
    languages: [...]        # Target languages
    severity: ...            # ERROR, WARNING, INFO
```

## Writing Your First Rules

### Example 1: Find Hardcoded Secrets

```yaml
rules:
  - id: hardcoded-api-key
    patterns:
      - pattern: api_key = "..."
    message: |
      Hardcoded API key detected. Use environment variables instead.
    languages: 
      - python
      - javascript
    severity: ERROR
    fix: api_key = os.environ.get('API_KEY')
```

**What this catches:**
```python
# BAD - will be caught
api_key = "sk-1234567890abcdef"
API_KEY = "super-secret-key"

# GOOD - won't be caught
api_key = os.environ.get('API_KEY')
api_key = config.get_api_key()
```

### Example 2: SQL Injection Detection

```yaml
rules:
  - id: sql-injection
    patterns:
      - pattern: |
          $QUERY = "..." + $INPUT
          $CURSOR.execute($QUERY)
      - pattern: |
          $CURSOR.execute("..." + $INPUT)
    message: Potential SQL injection vulnerability
    languages: [python]
    severity: ERROR
```

**What this catches:**
```python
# BAD - will be caught
query = "SELECT * FROM users WHERE id = " + user_input
cursor.execute(query)

# GOOD - parameterized query
cursor.execute("SELECT * FROM users WHERE id = ?", (user_input,))
```

## Pattern Operators

### 1. Ellipsis (...) - Match Anything

```yaml
# Matches any arguments
pattern: dangerous_function(...)

# Matches any function body
pattern: |
  def $FUNC(...):
    ...
```

### 2. Metavariables ($X) - Capture Values

```yaml
# $X captures the compared value
pattern: if $X == None:

# Multiple metavariables
pattern: $DICT[$KEY] = $VALUE
```

### 3. Pattern Combinations

**pattern-either: (OR logic)**
```yaml
patterns:
  - pattern-either:
      - pattern: eval(...)
      - pattern: exec(...)
      - pattern: compile(...)
```

**patterns: (AND logic)**
```yaml
patterns:
  - pattern: $FUNC(...)
  - metavariable-regex:
      metavariable: $FUNC
      regex: (eval|exec|compile)
```

**pattern-not: (Exclusion)**
```yaml
patterns:
  - pattern: requests.get(...)
  - pattern-not: requests.get(..., verify=True)
```

### 4. Pattern Inside

```yaml
patterns:
  - pattern-inside: |
      def $FUNC(...):
        ...
  - pattern: print(...)
```

## Metavariables

### Basic Metavariables

```yaml
# $X matches any expression
pattern: $X == $X  # Matches: a == a, user.id == user.id

# Different metavariables match different things
pattern: $X == $Y  # Matches: a == b, but NOT a == a
```

### Typed Metavariables

```yaml
# Match specific types
pattern: $X: int = ...
pattern: $FUNC(...) -> str:
```

### Metavariable Patterns

```yaml
patterns:
  - pattern: $REDIS.set($KEY, $VALUE)
  - metavariable-pattern:
      metavariable: $VALUE
      pattern: request.data
```

### Metavariable Regex

```yaml
patterns:
  - pattern: $FUNC(...)
  - metavariable-regex:
      metavariable: $FUNC
      regex: ^(system|exec|eval)$
```

## Advanced Patterns

### 1. Taint Analysis

```yaml
rules:
  - id: tainted-sql
    mode: taint
    pattern-sources:
      - pattern: request.$ANYTHING
    pattern-sinks:
      - pattern: $DB.execute(...)
    message: User input flows to SQL query
```

### 2. Control Flow Patterns

```yaml
patterns:
  - pattern: |
      if $COND:
        ...
        return $X
  - pattern-not: |
      if $COND:
        ...
        $LOG(...)
        return $X
```

### 3. Focus Metavariable

```yaml
patterns:
  - pattern: $OBJ.dangerous_method(...)
  - focus-metavariable: $OBJ
message: Object $OBJ uses dangerous method
```

### 4. Regular Expression Integration

```yaml
patterns:
  - pattern: $PASSWORD = "..."
  - metavariable-regex:
      metavariable: $PASSWORD
      regex: (?i)(password|passwd|pwd|secret|token)
```

## Security Scanning

### Common Vulnerability Patterns

#### 1. Command Injection
```yaml
rules:
  - id: command-injection
    patterns:
      - pattern-either:
          - pattern: os.system($INPUT)
          - pattern: subprocess.call($INPUT, shell=True)
          - pattern: subprocess.run($INPUT, shell=True)
      - pattern-not: os.system("...")  # Exclude hardcoded
    message: Potential command injection
    languages: [python]
    severity: ERROR
```

#### 2. Path Traversal
```yaml
rules:
  - id: path-traversal
    patterns:
      - pattern: open($PATH, ...)
      - metavariable-pattern:
          metavariable: $PATH
          patterns:
            - pattern-either:
                - pattern: request.$ANYTHING
                - pattern: "..." + $USER_INPUT
    message: Potential path traversal vulnerability
```

#### 3. XSS Detection
```yaml
rules:
  - id: xss-vulnerability
    patterns:
      - pattern-either:
          - pattern: |
              return HttpResponse($INPUT)
          - pattern: |
              render_template_string($INPUT)
      - pattern-not: |
          return HttpResponse(escape($INPUT))
    message: Potential XSS vulnerability
    languages: [python]
```

### OWASP Top 10 Coverage

```yaml
# Create a comprehensive security ruleset
rules:
  # A01: Broken Access Control
  - id: missing-auth-check
    pattern: |
      @app.route(...)
      def $FUNC(...):
        ...
    pattern-not: |
      @app.route(...)
      @requires_auth
      def $FUNC(...):
        ...
        
  # A02: Cryptographic Failures
  - id: weak-crypto
    pattern-either:
      - pattern: hashlib.md5(...)
      - pattern: hashlib.sha1(...)
    message: Weak cryptographic algorithm
    
  # A03: Injection (covered above)
  
  # Continue for all OWASP categories...
```

## Custom Rule Development

### Rule Development Workflow

```bash
# 1. Create test file with vulnerable code
echo 'password = "secret123"' > test.py

# 2. Write initial rule
cat > rule.yaml << EOF
rules:
  - id: test-rule
    pattern: password = "..."
    message: Found it!
    languages: [python]
EOF

# 3. Test the rule
semgrep --config=rule.yaml test.py

# 4. Refine and iterate
semgrep --config=rule.yaml --validate
```

### Testing Rules

```yaml
# rule_test.yaml
rules:
  - id: my-rule
    pattern: $X == None
    message: Use 'is None' instead
    languages: [python]
    
# test cases in comments
# ruleid: my-rule
if value == None:  # This should match
    pass

# ok: my-rule  
if value is None:  # This should NOT match
    pass
```

### Rule Organization

```
project/
├── .semgrep/
│   ├── security/
│   │   ├── injection.yml
│   │   ├── auth.yml
│   │   └── crypto.yml
│   ├── performance/
│   │   └── db-queries.yml
│   └── style/
│       └── naming.yml
└── .semgrep.yml  # Config file
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/semgrep.yml
name: Semgrep
on:
  pull_request: {}
  push:
    branches: [main, master]
    
jobs:
  semgrep:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            .semgrep/
```

### GitLab CI

```yaml
# .gitlab-ci.yml
semgrep:
  image: returntocorp/semgrep
  script:
    - semgrep --config=auto --json --output=semgrep.json .
  artifacts:
    reports:
      sast: semgrep.json
```

### Pre-commit Hook

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/returntocorp/semgrep
    rev: 'v1.0.0'
    hooks:
      - id: semgrep
        args: ['--config=auto', '--error']
```

### Docker Integration

```dockerfile
# Dockerfile
FROM python:3.9
COPY . /app
WORKDIR /app

# Security scan during build
RUN pip install semgrep && \
    semgrep --config=auto --error .
```

## Real-World Examples

### Example 1: Django Security Audit

```yaml
rules:
  - id: django-debug-true
    pattern: DEBUG = True
    paths:
      include:
        - settings.py
        - "**/settings/*.py"
    message: DEBUG should be False in production
    severity: WARNING
    
  - id: django-csrf-exempt
    pattern: |
      @csrf_exempt
      def $FUNC(...):
        ...
    message: CSRF protection disabled
    severity: ERROR
    
  - id: django-raw-sql
    patterns:
      - pattern: $MODEL.objects.raw(...)
      - pattern-not: $MODEL.objects.raw("...")
    message: Dynamic SQL in raw query
    severity: ERROR
```

### Example 2: React Security Patterns

```yaml
rules:
  - id: react-dangerouslysetinnerhtml
    pattern: dangerouslySetInnerHTML={...}
    message: Potential XSS via dangerouslySetInnerHTML
    languages: [javascript, typescript, jsx, tsx]
    
  - id: react-no-array-index-key
    pattern: |
      $ARRAY.map(($ITEM, $INDEX) => 
        <$COMPONENT key={$INDEX} ... />
      )
    message: Using array index as key can cause rendering issues
```

### Example 3: AWS Security

```yaml
rules:
  - id: aws-hardcoded-credentials
    patterns:
      - pattern-either:
          - pattern: |
              aws_access_key_id = "..."
          - pattern: |
              aws_secret_access_key = "..."
      - metavariable-regex:
          metavariable: $KEY
          regex: ^AKI[A-Z0-9]{16}$
    message: AWS credentials should not be hardcoded
    
  - id: s3-public-read
    pattern: |
      $BUCKET.put_object(..., ACL='public-read', ...)
    message: S3 object is publicly readable
```

### Example 4: API Key Detection

```yaml
rules:
  - id: generic-api-key
    patterns:
      - pattern-either:
          - pattern: $KEY = "..."
          - pattern: $KEY: "..."
      - metavariable-regex:
          metavariable: $KEY
          regex: (?i).*(api|key|token|secret|password).*
      - pattern-not-inside: |
          # ... test ...
          ...
    message: Potential hardcoded secret
```

## Performance Tips

### 1. Optimize Pattern Order
```yaml
# GOOD - specific pattern first
patterns:
  - pattern: dangerous_function(...)
  - pattern-inside: |
      class $CLASS:
        ...

# BAD - broad pattern first (slower)
patterns:
  - pattern-inside: |
      class $CLASS:
        ...
  - pattern: dangerous_function(...)
```

### 2. Use Path Filters
```yaml
rules:
  - id: check-config
    pattern: ...
    paths:
      include:
        - "*.config.js"
        - "config/*.py"
      exclude:
        - "test/**"
        - "node_modules/**"
```

### 3. Limit Language Scope
```yaml
# Specify exact languages
languages: [python]  # Fast

# Avoid generic if possible
languages: [generic]  # Slower
```

### 4. Use Pattern Caching
```bash
# Cache patterns for faster subsequent runs
semgrep --config=rules.yml --use-git-cache .

# Parallel processing
semgrep --jobs 4 --config=rules.yml .
```

## Troubleshooting

### Common Issues and Solutions

#### 1. Pattern Not Matching

**Problem:** Pattern doesn't match expected code

**Debug Steps:**
```bash
# Test with simple pattern
semgrep -e '$X == $X' --lang python test.py

# Show AST to understand structure
semgrep --dump-ast test.py

# Verbose output
semgrep --verbose --config=rule.yml test.py
```

**Solution:** Simplify pattern, use ellipsis
```yaml
# Instead of exact match
pattern: |
  if user.is_authenticated() == True:
    return render_template("dashboard.html")

# Use ellipsis
pattern: |
  if $CHECK == True:
    ...
```

#### 2. Too Many False Positives

**Solution:** Add exclusion patterns
```yaml
patterns:
  - pattern: eval(...)
  - pattern-not: eval("...")  # Exclude string literals
  - pattern-not-inside: |
      def test_$FUNC(...):  # Exclude test functions
        ...
```

#### 3. Performance Issues

**Solution:** Profile and optimize
```bash
# Profile rule performance
semgrep --time --config=rules.yml .

# Limit scope
semgrep --include="*.py" --exclude="*test*" .
```

### Debugging Techniques

```bash
# 1. Test pattern interactively
semgrep --pattern='$X == None' --lang=python

# 2. Validate rule syntax
semgrep --validate --config=rule.yml

# 3. Show matches with context
semgrep --config=rule.yml --verbose

# 4. Output JSON for analysis
semgrep --json --config=rule.yml > results.json
```

## Reference & Resources

### Pattern Cheat Sheet

```yaml
# Basic Patterns
$X              # Any expression
$X = ...        # Assignment
$FUNC(...)      # Function call
...$X           # Deep match

# Operators
pattern-either  # OR
patterns        # AND
pattern-not     # NOT
pattern-inside  # Context

# Metavariables
$X, $Y, $Z      # Capture expressions
$...ARGS        # Capture multiple
$_              # Don't care

# Special
=~              # Comparison operator
<... ...>       # Any statements
```

### Language Support

| Language | File Extensions | Quality |
|----------|----------------|---------|
| Python | .py | Excellent |
| JavaScript/TypeScript | .js, .ts, .jsx, .tsx | Excellent |
| Java | .java | Excellent |
| Go | .go | Excellent |
| Ruby | .rb | Good |
| C/C++ | .c, .cpp | Good |
| PHP | .php | Good |
| C# | .cs | Good |
| Rust | .rs | Beta |
| Kotlin | .kt | Beta |

### Useful Commands

```bash
# Check version
semgrep --version

# Update rules
semgrep --config=p/security-audit --update

# Run with metrics
semgrep --metrics=on --config=auto .

# Generate SARIF output
semgrep --sarif --config=auto . > results.sarif

# Autofix issues
semgrep --config=auto --autofix .

# Test playground
# Visit: https://semgrep.dev/playground
```

### Official Resources

- **Documentation**: https://semgrep.dev/docs/
- **Rule Registry**: https://semgrep.dev/r
- **Playground**: https://semgrep.dev/playground
- **GitHub**: https://github.com/returntocorp/semgrep
- **Community Rules**: https://github.com/returntocorp/semgrep-rules

### Best Practices

1. **Start Simple**: Begin with basic patterns, add complexity gradually
2. **Test Thoroughly**: Use test cases for all rules
3. **Document Rules**: Add clear messages and fix suggestions
4. **Version Control**: Track rules in git
5. **Regular Updates**: Keep Semgrep and rules updated
6. **Monitor Performance**: Profile rules on large codebases
7. **Collaborate**: Share rules with team
8. **Incremental Adoption**: Start with critical security rules
9. **Custom vs Registry**: Use registry rules when possible
10. **Continuous Improvement**: Refine rules based on feedback

---

## Quick Start Template

Save this as `.semgrep.yml` in your project root:

```yaml
rules:
  # Security
  - id: no-eval
    pattern: eval(...)
    message: Avoid eval() for security
    languages: [python, javascript]
    severity: ERROR
    
  # Code Quality
  - id: no-console-log
    pattern: console.log(...)
    message: Remove console.log
    languages: [javascript, typescript]
    severity: WARNING
    fix: ""
    
  # Best Practices
  - id: use-is-none
    pattern: $X == None
    message: Use 'is None' instead
    languages: [python]
    severity: INFO
    fix: $X is None
```

Run with: `semgrep --config=.semgrep.yml .`

---

This guide provides everything you need to start using Semgrep effectively for code analysis, security scanning, and maintaining code quality. Start with simple patterns and gradually build your rule library as you become more comfortable with the syntax and capabilities.