# Testing Standards

## Test-Driven Development

We practice TDD:

1. Write a failing test defining all desired behaviour
2. Run test, confirm it fails as expected
3. Write minimal code to pass
4. Run test, confirm success
5. Refactor while keeping tests green
6. Repeat for each feature/bugfix
7. Each new test must become part of our regression test pack, to be run after every subsequent change
8. If I give you multiple issues to work on, run the regression test pack after all of them are done, not after each one.

## Policy

- All production code requires unit tests, integration tests, AND end-to-end tests.
- Implement with test coverage that can be run via `make test`
- Update regression tests, create regression tests if there are none

**Exception:** Requires explicit approval in commit message:
```
SKIP TESTS: [specific reason]
```

To authorize skipping: I must say **"I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME"**

## Test Naming

Use descriptive names that document what's being tested:

```
test_<unit>_<scenario>_<expected_result>
```

Examples:
- `test_login_with_invalid_password_returns_401`
- `test_cart_add_item_increases_total`
- `test_export_empty_list_returns_empty_file`

The test name should tell you what failed without reading the code.

## Test Structure

Follow Arrange-Act-Assert (AAA):

```python
def test_discount_applied_to_eligible_order():
    # Arrange: set up preconditions
    order = Order(items=[item_over_threshold])

    # Act: perform the action
    result = apply_discount(order)

    # Assert: verify the outcome
    assert result.discount_percent == 10
```

## Coverage

- **Minimum threshold:** 80% line coverage for new code
- **Critical paths:** 100% coverage for authentication, payment, data persistence
- Coverage is a floor, not a ceiling — high coverage with weak assertions is worthless

## Test Boundaries

| Type | Scope | Speed | When to run |
|------|-------|-------|-------------|
| Unit | Single function/class | Fast (<100ms) | Every save |
| Integration | Multiple components | Medium (<5s) | Pre-commit |
| End-to-end | Full system | Slow | CI pipeline |

- Unit tests should have no external dependencies (no network, no filesystem, no database)
- Integration tests may use local resources (test database, mock servers)
- E2E tests exercise the real system

## Test Data

- **Use factories** for generating test objects (factory_boy, faker, etc.)
- **Fixtures** for shared setup — but keep them minimal and obvious
- **No production data** in tests — synthetic data only
- **Deterministic:** tests must produce the same result every run (no random without seed)

## Clarification: "No Mocks" Rule

The "no mocks" rule in CODING.md refers to **not mocking your way out of testing real behaviour**. Specifically:

**Acceptable:**
- Mocking external HTTP APIs to avoid network calls in unit tests
- Mocking time/dates for deterministic tests
- Mocking filesystem for unit tests (but prefer tmpdir fixtures)

**Not acceptable:**
- Mocking the class you're supposed to be testing
- Mocking so much that the test doesn't verify real behaviour
- Using mocks instead of fixing the design to be testable

When in doubt: if removing the mock would make the test fail for reasons *other than* external dependencies, you're mocking too much.

## Requirements

- Tests MUST cover functionality being implemented
- Never ignore test output — logs often contain critical information
- Test output must be pristine to pass
- If logs should contain errors, capture and test them

## Quality

- Clear, maintainable tests
- One behaviour per test
- Descriptive names explaining what's tested
- Include edge cases and error conditions
- Tests are documentation — write for the next developer

## Warnings and Errors

- Never introduce new warnings or errors
- If an action generates a warning or error, stop everything and resolve it
- Build-time errors (e.g., Hugo static site) are equivalent to compile errors — fix immediately

---

# Mandatory

Every interaction with me must begin with "EHLO!"
