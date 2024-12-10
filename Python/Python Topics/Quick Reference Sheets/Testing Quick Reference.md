## Basic pytest Commands
| Command | Description | Example |
|---------|-------------|----------|
| `pytest` | Run all tests | `pytest` |
| `pytest test_file.py` | Run specific test file | `pytest test_file.py` |
| `pytest -v` | Verbose output | `pytest -v` |
| `pytest -k "test_name"` | Run tests matching pattern | `pytest -k "test_user"` |
| `pytest -m marker` | Run marked tests | `pytest -m "slow"` |
| `pytest -s` | Show print statements | `pytest -s` |
| `pytest --pdb` | Debug on failure | `pytest --pdb` |
| `pytest --cov` | Show coverage report | `pytest --cov=myapp` |

## Coverage Commands
| Command | Description | Example |
|---------|-------------|----------|
| `coverage run` | Run coverage | `coverage run -m pytest` |
| `coverage report` | Show coverage report | `coverage report` |
| `coverage html` | Generate HTML report | `coverage html` |
| `coverage xml` | Generate XML report | `coverage xml` |

## unittest Commands
| Command | Description | Example |
|---------|-------------|----------|
| `python -m unittest` | Run all tests | `python -m unittest` |
| `python -m unittest test_file.py` | Run specific file | `python -m unittest test_file.py` |
| `python -m unittest -v` | Verbose output | `python -m unittest -v` |
| `python -m unittest discover` | Auto-discover tests | `python -m unittest discover` |

## Common pytest Flags
| Flag | Description | Example |
|------|-------------|----------|
| `-v` | Verbose output | `pytest -v` |
| `-q` | Quiet output | `pytest -q` |
| `-x` | Stop after first failure | `pytest -x` |
| `--lf` | Run last failed tests | `pytest --lf` |
| `--ff` | Run failed tests first | `pytest --ff` |
| `-n auto` | Parallel testing | `pytest -n auto` |
| `--durations=N` | Show N slowest tests | `pytest --durations=3` |

## Debugging Commands
| Command | Description | Example |
|---------|-------------|----------|
| `breakpoint()` | Set breakpoint in code | `breakpoint()` |
| `pytest --trace` | Enter debugger on start | `pytest --trace` |
| `pytest -vv` | Extra verbose output | `pytest -vv` |
| `pytest --showlocals` | Show local variables | `pytest --showlocals` |

## Common Test Decorators
| Decorator | Description | Example |
|-----------|-------------|----------|
| `@pytest.mark.slow` | Mark test as slow | `@pytest.mark.slow` |
| `@pytest.fixture` | Define test fixture | `@pytest.fixture` |
| `@pytest.mark.skip` | Skip test | `@pytest.mark.skip(reason="...")` |
| `@pytest.mark.parametrize` | Parametrize test | `@pytest.mark.parametrize("input,expected", [...])` |

## Common Test Patterns
```python
# Basic Test
def test_function():
    assert function() == expected

# Fixture Usage
@pytest.fixture
def sample_data():
    return {"key": "value"}

def test_with_fixture(sample_data):
    assert sample_data["key"] == "value"

# Parametrized Test
@pytest.mark.parametrize("input,expected", [
    ("test1", 1),
    ("test2", 2),
])
def test_parametrized(input, expected):
    assert function(input) == expected

# Skip Test
@pytest.mark.skip(reason="not implemented")
def test_future():
    pass

# Expected Exception
def test_exception():
    with pytest.raises(ValueError):
        function()
```

## pytest.ini Example
```ini
[pytest]
markers =
    slow: marks tests as slow
    integration: marks tests as integration
testpaths = tests
python_files = test_*.py
addopts = -v --cov=myapp
```

Quick Tips:
- 💡 Use `conftest.py` for shared fixtures
- 🐞 Use `--pdb` for debugging failed tests
- 📊 Regular coverage checks with `--cov`
- 🏃‍♂️ Use `-n auto` for parallel testing
- ⏱️ Profile slow tests with `--durations`
- 🔄 Use fixtures for test setup/teardown
- 📝 Group related tests in classes
- 🎯 Use meaningful test names

Remember:
- Always run tests in clean environment
- Keep tests independent
- One assertion per test
- Use descriptive test names
- Mock external dependencies
- Test edge cases
- Regular coverage checks