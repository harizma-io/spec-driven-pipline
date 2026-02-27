# Test Quality Review Guide

Methodology for analyzing quality of existing tests. Detects meaningless, ineffective, or poorly designed tests.

## Table of Contents
- [Core Philosophy](#core-philosophy)
- [Categories of Bad Tests](#categories-of-bad-tests)
- [Severity Levels](#severity-levels)
- [Review Process](#review-process)
- [Status Decision Criteria](#status-decision-criteria)
- [Task Required Decision](#task-required-decision)
- [Litmus Test Methodology](#litmus-test-methodology)
- [Prescriptive Findings](#prescriptive-findings)

## Core Philosophy

Tests exist to:
1. Verify that code does what it should
2. Catch regressions when code changes
3. Document expected behavior

Tests that fail these purposes are worse than no tests - they provide false confidence.

---

## Categories of Bad Tests

### Category 1: Empty/Meaningless Tests

Tests that verify nothing:

```python
# BAD - Tests nothing
def test_should_exist():
    assert True

# BAD - No assertions
def test_renders_component():
    render(MyComponent())  # no assertion

# BAD - Only checks callable
def test_function_defined():
    assert callable(my_function)  # always True
```

### Category 2: Mock-Only Tests

Tests that only verify mock calls without checking results:

```python
# BAD - Only tests mock was called
async def test_calls_api():
    await fetch_user_data(1)
    mock_api.get.assert_called_with('/users/1')
    # No assertion on the actual result!

# BAD - Mocks everything, tests nothing real
def test_processes_data():
    mock_processor = Mock(return_value='result')
    assert mock_processor() == 'result'  # Testing the mock!
```

### Mock-Return Anti-pattern

Agent mocks a dependency to return a value, then asserts the same value:

```python
# Tests mock wiring, not code behavior
mock_user = {"id": 1, "name": "Alice"}
mock_user_service.create.return_value = mock_user
result = await handler(request)
assert result == mock_user
# Litmus test: delete handler implementation → test still passes
```

### Category 3: Missing Coverage

Code without corresponding tests:

- Business logic functions without unit tests
- API endpoints without integration tests
- Decision branches (if/else) not covered
- Error handling paths untested
- Edge cases from spec not tested

### Category 4: Test Pyramid Violations

Wrong test distribution:

```
Expected:
- Many unit tests (fast, isolated)
- Some integration tests (real DB/API)
- Few E2E tests (critical paths only)

Violations:
- All E2E, no unit tests (slow, brittle)
- Only unit tests for UI app (misses real interactions)
- Integration tests for pure logic (overkill)
```

### Category 5: Excessive Mocking

When mocking defeats the purpose:

```python
# BAD - Mocks 3+ dependencies
@patch('src.database.save')
@patch('src.cache.get')
@patch('src.email.send')
@patch('src.logger.info')
def test_user_service(mock_log, mock_email, mock_cache, mock_db):
    # At this point, what are we even testing?
    pass
```

**Rule:** If mocking 3+ dependencies, this should be an integration test.

### Category 6: Test Anti-patterns

- **Implementation testing** - Tests break when refactoring without behavior change
- **Snapshot abuse** - Large snapshots nobody reviews
- **Flaky tests** - Random failures due to timing/order
- **Shared state** - Tests depend on each other
- **Magic values** - Unexplained test data

---

## Severity Levels

### Critical
- No tests at all for business-critical code
- Tests that actively hide bugs (incorrect assertions)
- All tests are empty/meaningless (false coverage)

### High
- Missing tests for error handling
- Tests verify only mock calls (no result checking)
- Key acceptance criteria not tested

### Medium
- Excessive mocking (should be integration test)
- Test pyramid violation (wrong test type used)
- Edge cases from spec not covered

### Low
- Minor best practice violations
- Could be more specific assertions
- Naming improvements needed

---

## Review Process

1. **Identify Test Files**: Find all test files for reviewed code
2. **Map Coverage**: Match implementation files to test files
3. **Analyze Each Test**:
   - Does it have meaningful assertions?
   - Does it test real behavior or just mocks?
   - Does it cover the right scenarios?
4. **Check Pyramid Balance**: Assess unit/integration/E2E distribution
5. **Find Gaps**: Identify untested code paths
6. **Categorize Findings**: Group by category and severity

---

## Status Decision Criteria

### passed
- All tests have meaningful assertions
- Critical business logic is tested
- Test pyramid is reasonably balanced
- Minor suggestions only (low severity)

### needs_improvement
- Some tests need better assertions (medium severity)
- Some coverage gaps exist (non-critical areas)
- Pyramid slightly unbalanced
- No critical issues

### failed
- Tests are meaningless (empty or mock-only)
- Critical business logic untested
- Tests hide bugs (wrong assertions)
- Test pyramid severely inverted
- Multiple high/critical severity issues

**Decision matrix:**
- `critical > 0` → failed
- `high >= 3` → failed
- `high >= 1 AND medium >= 3` → needs_improvement
- `medium >= 5` → needs_improvement
- Only low issues → passed
- No issues → passed

---

## Task Required Decision

Set `taskRequired.needed = true` when:
- status === "failed"
- critical > 0
- high >= 2
- Critical business logic has no tests

Set `taskRequired.needed = false` when:
- status === "passed"
- Only low/medium issues
- Issues can be fixed in current context

---

## Litmus Test Methodology

For every test touching business logic, ask:

> "If I remove the core logic line being tested, does this test still pass?"

**How to apply:**
1. Identify the core logic line (computation, validation, or side effect)
2. Mentally remove it
3. Trace test execution without that line
4. If test still passes → flag as litmus test failure

**Common patterns that fail:**
- Mock returns X, test asserts X (passes with empty function)
- Test only asserts mock.toHaveBeenCalled() (passes with any call)
- Test uses same hardcoded data for input and expected output

---

## Prescriptive Findings

Every finding must include a concrete replacement, not just a problem description.

**Bad finding:**
```json
{ "issue": "Test has no meaningful assertions", "recommendation": "Add assertion that verifies actual behavior" }
```

**Good finding:**
```json
{
  "issue": "Mock returns mockUser, test asserts mockUser — tests mock wiring, not code",
  "litmusTestFailed": true,
  "replacement": {
    "approach": "Call real createUser with test data, assert on actual result",
    "assertions": ["result.id is defined", "result.email === input.email"],
    "mockChange": "Remove mockUserService, use test DB"
  }
}
```
