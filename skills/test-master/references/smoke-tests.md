# Smoke Testing Guide

**For:** Infrastructure setup, verifying basic project functionality

## Table of Contents
- [Purpose](#purpose)
- [When to Write Smoke Tests](#when-to-write-smoke-tests)
- [What to Test](#what-to-test)
- [Example Smoke Tests](#example-smoke-tests)
- [Characteristics](#characteristics)
- [CI/CD Integration](#cicd-integration)
- [Key Principles](#key-principles)
- [Common Mistakes](#common-mistakes)
- [Checklist](#checklist)

## Purpose

Smoke tests verify minimal system functionality - "is the system alive?"

**Not for:**
- Testing business logic (use unit tests)
- Testing API endpoints (use integration tests)
- Testing user flows (use E2E tests)

**For:**
- Verifying project setup works
- Ensuring test infrastructure is functional
- Providing CI/CD with basic health check
- Confirming dependencies are installed

---

## When to Write Smoke Tests

### During Infrastructure Setup (Step 7)
- Part of testing infrastructure setup
- Created once per project
- Verifies test framework configuration
- Checks environment is set up

### Add to CI Pipeline
- Run smoke tests first (before unit/integration/E2E)
- If smoke test fails → don't run other tests (fail fast)
- Saves CI time by catching infrastructure problems early

---

## What to Test

### ✅ Write smoke tests for:
- Key modules/packages can be imported
- **App can start** (renders, server starts, CLI runs)
- Environment variables accessible (`process.env.NODE_ENV`)

**Good smoke test:**
```python
def test_main_module_import():
    """Verify main application module can be imported."""
    import src.main  # should not raise ImportError
```

**Bad smoke test - tests nothing:**
```python
# ❌ USELESS - delete such tests
def test_always_passes():
    assert True
```

### ❌ Don't write smoke tests for:
- Business logic (use unit tests)
- API endpoints (use integration tests)
- User workflows (use E2E tests)
- Data processing (use unit tests)

---

## Example Smoke Tests

### Python (primary)



**File:** `tests/test_smoke.py`

```python
"""
Smoke tests - verify basic project setup
"""

import os


def test_environment_configured():
    """Verify environment variables can be accessed."""
    # This should pass even if ENVIRONMENT is not set
    # (allows test to pass in minimal CI environments)
    env = os.getenv('ENVIRONMENT', 'test')
    assert env is not None


def test_main_module_import():
    """Verify main application module can be imported."""
    try:
        import src.main  # Adjust based on your structure
        assert True
    except ImportError as e:
        assert False, f"Failed to import main module: {e}"
```

---

## Characteristics

### Speed
- **Target:** Milliseconds
- **Requirement:** <1 second total
- Fastest tests in the pyramid

### Scope
- **Minimal:** 1-2 tests are sufficient
- Don't test everything, just basics
- If more than 5 smoke tests → probably testing too much

### When They Run
- **First** in test suite (before all others)
- **Every CI run** (fail fast if infrastructure broken)
- **Locally** when setting up project

### What They Don't Do
- ❌ Don't test business logic
- ❌ Don't make database queries
- ❌ Don't make API calls
- ❌ Don't test user interactions

---

## CI/CD Integration

### Run Smoke Tests First

```yaml
# .github/workflows/ci.yml
jobs:
  smoke-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
        with:
          python-version: "3.13"
      - run: uv sync --frozen
      - name: Run smoke tests
        run: uv run pytest tests/test_smoke.py -v

  unit-tests:
    needs: smoke-test  # Only run if smoke passes
    runs-on: ubuntu-latest
    steps:
      - name: Run unit tests
        run: uv run pytest tests/unit/ -v
```

### Fail Fast Strategy

If smoke test fails:
- Stop CI pipeline immediately
- Don't run slower tests
- Save CI time and costs
- Infrastructure problem is obvious

---

## Key Principles

1. **Minimal** - Only 1-2 tests, not comprehensive
2. **Fast** - Must run in milliseconds
3. **Infrastructure-focused** - Tests setup, not logic
4. **Always pass** - If smoke test fails, stop everything and fix setup
5. **Run first** - Before all other test types

---

## Common Mistakes

### ❌ Too Many Smoke Tests
```python
# Bad: Testing too much in smoke tests
def test_database_connection(): ...
def test_api_endpoint(): ...
def test_user_creation(): ...
def test_authentication(): ...
# ... 20 more tests
```

**Fix:** Keep minimal (1-2 tests). Move others to appropriate test type.

### ❌ Slow Smoke Tests
```python
# Bad: Smoke test that makes real connections
async def test_database_alive():
    await db.connect()           # Slow!
    await db.execute("SELECT 1") # Not a smoke test!
```

**Fix:** Smoke tests shouldn't make real connections. Just test imports work.

### ❌ Business Logic in Smoke Tests
```python
# Bad: Testing business logic in smoke tests
def test_discount_calculation():
    assert calculate_discount(100, 0.2) == 80.0
```

**Fix:** Move to unit tests.

---

## Checklist

Before completing smoke test setup:

- [ ] 1-2 smoke tests created
- [ ] Tests verify test framework works
- [ ] Tests verify environment configured
- [ ] Tests verify main module imports
- [ ] Tests run in <1 second
- [ ] Tests always pass (if fail → fix infrastructure)
- [ ] Tests added to CI/CD as first job
- [ ] CI configured to fail fast if smoke tests fail

