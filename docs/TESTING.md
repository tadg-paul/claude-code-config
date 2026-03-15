# Testing Standards

## Test-Driven Development

We practice TDD. Per issue:

1. Read the issue, understand the requirements
2. Write failing tests defining the desired behaviour (**issue tests**)
3. Run issue tests, confirm they fail as expected
4. Write minimal code to pass
5. Run issue tests, confirm success
6. Refactor while keeping issue tests green

Each new test becomes part of the **regression test pack** (`make test`).

## Issue Tests vs Regression Tests

- **Issue tests:** the tests written specifically for the issue being worked on. Run these frequently during development.
- **Regression tests:** the full test suite (`make test`). Run these only at batch boundaries (see below), not after every individual issue.

Our projects typically have hundreds of tests. Running the full suite after every small change wastes time. Trust the issue tests during development; the regression pack catches cross-cutting breakage at the end.

## Batch Workflow

When working on multiple issues, follow this sequence:

**Standard batch (small issues):**
```
For each issue in the batch:
    1. Write failing issue tests (TDD step 2-3)
    2. Implement (TDD step 4-6)
    3. Run issue tests only
After ALL issues in the batch are complete:
    4. Run regression test pack (make test)
    5. Fix any regressions
```

**Large issue isolation:** If a batch includes a large or architecturally significant issue (e.g. a major refactor), isolate it into its own batch with its own regression run. Example for 4 small issues + 1 large issue:

```
Option A: [4 small issues] → [regression] → [large issue] → [regression]
Option B: [large issue] → [regression] → [4 small issues] → [regression]
```

Never mix a large refactor into a batch of small issues — a regression failure becomes impossible to attribute.

**Documentation-only changes** (no code modified): no tests required, no regression run required.

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
