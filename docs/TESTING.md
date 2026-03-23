# Testing Standards

## Test-Driven Development
We practice TDD. Per issue:

1. **Verify the issue has been explicitly APPROVED by me.** (Do not proceed if unapproved).
2. Read the issue, understand the requirements.
3. Write failing tests defining the desired behaviour (**issue tests**).
4. Run issue tests, confirm they fail as expected.
5. Write minimal code to pass.
6. Run issue tests, confirm success.
7. Refactor while keeping issue tests green.

Each new test becomes part of the **regression test pack** (`make test`).

## Issue Tests vs Regression Tests

- **Issue tests:** the tests written specifically for the issue being worked on. Run these frequently during development.
- **Regression tests:** the full test suite (`make test`). Run these only at batch boundaries (see below), not after every individual issue.

Our projects typically have hundreds of tests. Running the full suite after every small change wastes time. Trust the issue tests during development; the regression pack catches cross-cutting breakage at the end.

## One-Off Tests

Most tests should become part of the regression test pack. **This is the default.** If you are writing a new test and are unsure where it belongs, it belongs in regression.

A **one-off test** is the exception: a test tied to a specific, non-recurring activity — verifying a data migration ran correctly, reproducing a specific production incident, checking a one-time environment state. Once its purpose is served, it has no value running again.

### Storage

Tests live in one of two locations:

```
tests/
  regression/    ← default; all new tests go here; run by `make test`
  one_off/       ← quarantine; never run by `make test`
```

`make test` must only reference `tests/regression/`. `tests/one_off/` is structurally excluded. This is the primary protection against one-off tests polluting CI.

### Marker

All one-off tests must also carry a marker with a mandatory `issue` reference, using whichever annotation or tagging mechanism the project's test framework provides. The following is illustrative only — apply the equivalent for your language and framework:

```python
# Python/pytest — illustrative only
@pytest.mark.one_off(issue="#123")
def test_migration_user_records_backfilled():
    ...
```

If the framework has no native marker mechanism, encode the issue reference in the test name as a suffix: `test_migration_user_records_backfilled_OT007`.

A one-off test without an `issue` reference must fail linting. Both the directory placement and the marker are required — either alone can catch a mistake.

### Running one-off tests manually

Use the test runner appropriate to the project's language and framework. Via the Makefile:

```bash
make test-one-off          # run all one-off tests
make test-one-off ISSUE=123  # run one-off tests for a specific issue
```

### Decision rule

> **If in doubt, it's a regression test.**

Only place a test in `one_off/` if you can answer: *"After this issue is closed and verified, will this test ever be meaningful to run again?"* If the answer is yes, or you are unsure, it belongs in regression.

### Lifecycle

One-off tests are temporary. They must be reviewed and deleted once their associated issue is closed and verified. They are not to accumulate indefinitely.

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

## Test IDs

All tests carry a unique ID encoded in their marker. The prefix indicates type:

| Prefix | Type | Location | Run by |
|--------|------|----------|--------|
| `RT-NNN` | Regression test | `tests/regression/` | `make test` |
| `OT-NNN` | One-off test | `tests/one_off/` | `make test-one-off` |
| `UT-NNN` | User test | documented in issue only | human |

Numbers are zero-padded to three digits. Each prefix has its own sequence.

**User tests (`UT-NNN`)** are tests that cannot be automated or executed by an AI agent — they require a human to perform and verify manually (e.g. validating content is accessible in a specific app or device after a migration). They have no corresponding test file. The test description, steps, and expected outcome are documented in the AC table of the relevant GitHub issue. They must still be assigned an ID and marked as `passing` / `failing` / `skipped` in the AC table by the user after execution.

### Usage in markers

Embed the test ID in whichever marker or annotation mechanism the project's test framework provides. The following Python/pytest examples are illustrative only — apply the equivalent in your language and framework (Jest, RSpec, JUnit, Swift Testing, etc.):

```python
# Python/pytest — illustrative only
@pytest.mark.regression(test_id="RT-042")
def test_login_with_invalid_password_returns_401():
    ...

@pytest.mark.one_off(issue="#123", test_id="OT-007")
def test_migration_user_records_backfilled():
    ...
```

If the framework has no native marker or annotation mechanism, encode the ID in the test name as a suffix:

```
test_login_with_invalid_password_returns_401_RT042
test_migration_user_records_backfilled_OT007
```

### Usage in GitHub issues

Reference test IDs in AC tables:

```
| AC-1 | Login rejects invalid passwords | ✅ RT-042 |
| AC-2 | Migration backfills user records | ✅ OT-007 |
```

### ID allocation

The file `tests/NEXT_IDS.txt` is the source of truth for the next available ID in each sequence:

```
RT 043
OT 008
```

When allocating a new ID: read the file, take the current value, increment it, write it back. This must happen in the same commit as the test itself.

If `tests/NEXT_IDS.txt` does not exist in a project, create it and seed all three sequences at `001` before allocating the first ID.

```
RT 001
OT 001
UT 001
```

**Test ID allocation is not a code change.** Allocating IDs in a GitHub issue's AC table, and creating `tests/NEXT_IDS.txt` if it does not yet exist, may be done at any time as part of issue preparation — no approved issue is required for either activity.

### Mid-project migration policy

This convention was introduced mid-project. The following rules govern legacy tests.

#### Directory structure

Projects created before this convention may have a flat `tests/` directory with no `regression/` or `one_off/` subdirectories.

When beginning work on any issue in such a project, **migrate the full test suite first** as a discrete step before any other work:

1. Move all existing tests from `tests/` into `tests/regression/`
2. Create `tests/one_off/.gitkeep`
3. Update `make test` to point at `tests/regression/`
4. Commit with message: `chore: migrate test suite to regression/one_off layout`
5. Then proceed with the issue

Do not fold this into the issue commit. It must be its own commit so any breakage is immediately attributable.

This layout is intentionally CI-ready. Any future CI pipeline will pick up `tests/regression/` as the test target automatically. CI configuration, if and when introduced, is code and follows the same standards as all other code.

#### Test IDs

1. **Do not retrofit IDs.** Pre-existing tests with no ID must not be modified solely to add one. Retrofitting adds churn with no functional value.

2. **Migrate on touch.** If a pre-existing test is being modified as part of an issue, add an ID at that point.

3. **New tests always get an ID.** No exceptions, regardless of project age.

4. **Do not flag missing IDs unprompted.** If Claude Code notices a legacy test lacks an ID while working on something unrelated, note it in the issue comment but do not modify the test.

## Test Structure

Follow Arrange-Act-Assert (AAA) regardless of language or framework:

```
# Pseudocode — apply in whichever language the project uses
test "discount applied to eligible order":
    # Arrange: set up preconditions
    order = Order(items=[item_over_threshold])

    # Act: perform the action
    result = apply_discount(order)

    # Assert: verify the outcome
    assert result.discount_percent == 10
```

Python/pytest example (illustrative only):

```python
def test_discount_applied_to_eligible_order():
    # Arrange
    order = Order(items=[item_over_threshold])
    # Act
    result = apply_discount(order)
    # Assert
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

- **Use factories** for generating test objects — use the factory/fixture library appropriate to the project's language and framework (e.g. factory_boy for Python, FactoryBot for Ruby, fishery for TypeScript, fixtures for Go)
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

## Makefile Test Targets

Projects must expose these standard targets. Replace `<test-runner>` and `<filter-flag>` with the appropriate command for the project's language and framework (e.g. `pytest` for Python, `jest` for Node, `swift test` for Swift, `go test ./...` for Go):

```makefile
test:
	<test-runner> tests/regression/

test-one-off:
ifdef ISSUE
	<test-runner> tests/one_off/ <filter-flag> "$(ISSUE)"
else
	<test-runner> tests/one_off/
endif
```

`make test` must never reference `tests/one_off/`. If a project does not yet have a `tests/one_off/` directory, create it with a `.gitkeep`.

---

# Mandatory

Every interaction with me must begin with "EHLO!"
