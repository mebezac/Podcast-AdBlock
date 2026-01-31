# Testing & Quality Assurance

## Test Suite Overview

Podly uses **pytest** as the testing framework with tests organized by component.

## Test Structure

```
tests/
├── conftest.py              # Shared fixtures and configuration
├── test_auth.py             # Authentication tests
├── test_config_routes.py    # Configuration API tests
├── test_feeds.py            # Feed management tests
├── test_posts.py            # Episode/post tests
├── test_jobs.py             # Background job tests
├── test_processor.py        # Audio processing tests
├── test_writer.py           # Writer service tests
└── integration/             # Integration tests
    └── test_end_to_end.py
```

## Running Tests

```bash
# Run all tests
pytest

# Run with verbose output
pytest -v

# Run specific test file
pytest tests/test_feeds.py

# Run specific test
pytest tests/test_feeds.py::test_add_feed

# Run with coverage
pytest --cov=src --cov-report=html

# Run CI suite
./scripts/ci.sh
```

## Test Configuration

**conftest.py fixtures:**
```python
@pytest.fixture
def app():
    """Create application for testing."""
    app = create_app(testing=True)
    return app

@pytest.fixture
def client(app):
    """Test client."""
    return app.test_client()

@pytest.fixture
def auth_client(client):
    """Authenticated test client."""
    client.post('/api/auth/login', json={
        'username': 'test',
        'password': 'test'
    })
    return client
```

## Test Categories

### Unit Tests
Test individual functions and classes in isolation.

```python
def test_hash_password():
    from app.auth.passwords import hash_password, verify_password
    
    password = "test123"
    hashed = hash_password(password)
    
    assert verify_password(password, hashed) is True
    assert verify_password("wrong", hashed) is False
```

### API Tests
Test Flask routes and endpoints.

```python
def test_get_feeds(auth_client):
    response = auth_client.get('/api/feeds')
    
    assert response.status_code == 200
    data = response.get_json()
    assert 'feeds' in data
```

### Integration Tests
Test component interactions.

```python
def test_feed_processing_workflow():
    # Add feed
    # Download episode
    # Process episode
    # Verify output
```

### End-to-End Tests
Full workflow testing (marked as slow).

```python
@pytest.mark.slow
def test_full_podcast_processing():
    # Real audio processing test
    # Takes several minutes
```

## Code Quality Tools

### Black (Formatter)
```bash
black src/ tests/
```
- Enforces consistent code style
- Line length: 88 characters
- Configured in `pyproject.toml`

### isort (Import Sorter)
```bash
isort src/ tests/
```
- Organizes imports
- Black-compatible profile
- Floats imports to top

### mypy (Type Checker)
```bash
mypy src/
```
- Static type checking
- Strict mode enabled
- Catches type errors before runtime

### Pylint (Linter)
```bash
pylint src/
```
- Code quality analysis
- Configured in `.pylintrc`
- Disabled rules: docstring requirements, too-many-arguments, etc.

### CI Script
```bash
./scripts/ci.sh
```
Runs all checks in sequence:
1. pytest
2. black --check
3. isort --check
4. mypy
5. pylint

## Writing Tests

### Best Practices

1. **Use fixtures** for common setup
2. **Mock external APIs** (LLM, Whisper)
3. **Test both success and error cases**
4. **Keep tests independent** (no shared state)
5. **Use descriptive names** `test_<what>_<condition>`

### Example Test Pattern

```python
import pytest
from unittest.mock import patch, MagicMock

def test_process_post_success(auth_client, sample_post):
    """Test successful episode processing."""
    # Arrange
    post_id = sample_post.id
    
    # Mock external services
    with patch('app.writer.client.writer_client.action') as mock_action:
        mock_action.return_value = MagicMock(
            success=True,
            data={'job_id': '123'}
        )
        
        # Act
        response = auth_client.post(f'/api/posts/{post_id}/process')
        
        # Assert
        assert response.status_code == 200
        mock_action.assert_called_once()

def test_process_post_not_found(auth_client):
    """Test processing non-existent episode."""
    response = auth_client.post('/api/posts/99999/process')
    
    assert response.status_code == 404
```

### Mocking Database

```python
@pytest.fixture
def mock_db(app):
    """Provide mocked database session."""
    with app.app_context():
        # Rollback after each test
        yield db.session
        db.session.rollback()
```

### Mocking External APIs

```python
@pytest.fixture
def mock_llm():
    """Mock LLM API calls."""
    with patch('podcast_processor.ad_classifier.litellm.completion') as mock:
        mock.return_value = {
            'choices': [{
                'message': {
                    'content': '{"is_ad": true, "confidence": 0.95}'
                }
            }]
        }
        yield mock
```

## Test Data

### Factories
Use factory functions to create test data:

```python
def create_test_feed(title="Test Feed", rss_url=None):
    if rss_url is None:
        rss_url = f"https://example.com/{uuid4()}.rss"
    
    return writer_client.create('Feed', {
        'title': title,
        'rss_url': rss_url
    })
```

### Fixtures
```python
@pytest.fixture
def sample_feed():
    return create_test_feed()

@pytest.fixture
def sample_post(sample_feed):
    return create_test_post(feed_id=sample_feed.data['id'])
```

## Performance Testing

### Benchmarks
```python
@pytest.mark.benchmark
def test_transcription_performance(benchmark):
    result = benchmark(transcribe_audio, test_audio_file)
    assert result is not None
```

### Load Testing
Not implemented, but could use `locust`:
```bash
pip install locust
locust -f locustfile.py
```

## Coverage

**Target Coverage:** 70%+

```bash
# Generate coverage report
pytest --cov=src --cov-report=html --cov-report=term

# View HTML report
open htmlcov/index.html
```

**Coverage Configuration:**
```ini
# pyproject.toml
[tool.coverage.run]
source = ["src"]
omit = ["*/tests/*", "*/migrations/*"]

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError"
]
```

## Continuous Integration

GitHub Actions workflow (`.github/workflows/`):

```yaml
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - run: pip install -r requirements.txt
      - run: ./scripts/ci.sh
```

## Debugging Failed Tests

```bash
# Run with debugger
pytest --pdb

# Stop on first failure
pytest -x

# Show locals on failure
pytest --showlocals

# Full traceback
pytest --tb=long
```

## Test Database

Tests use a separate SQLite database:
- Path: `:memory:` or `/tmp/test.db`
- Created fresh for each test
- Migrations applied automatically
- Rolled back after each test

Configure in `conftest.py`:
```python
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///:memory:'
```
