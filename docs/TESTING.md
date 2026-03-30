# Testing Standards

This document defines how to write, structure, and organize tests. The TDD workflow sequence - when to enumerate, when to write tests, when to run regression - is in CLAUDE.md §3.

## TDD

We practice test-driven development:

1. Enumerate test cases for each AC (every distinct condition gets a test)
2. Write failing tests defining the desired behaviour
3. Run them, confirm they fail
4. Write minimal code to pass
5. Run them, confirm they pass
6. Refactor while keeping tests green

Each new test becomes part of the **regression test pack** (`make test`).

## Multi-condition coverage

An AC that implies multiple conditions requires a test for each condition.

Example: the AC *"The CLI rejects invalid, missing, and malformed configuration files"* implies three conditions. You must write at least three tests - one for invalid, one for missing, one for malformed. Writing a single test and marking the AC as passing is a process violation.

## Issue tests vs regression tests

- **Issue tests:** the tests written specifically for the issue being worked on. Run these frequently during development.
- **Regression tests:** the full test suite (`make test`). Run these only at batch boundaries, not after every individual issue.

Trust the issue tests during development; the regression pack catches cross-cutting breakage at the end.

## One-off tests

Most tests belong in regression. **This is the default.** If you are writing a new test and are unsure where it belongs, it belongs in regression.

A **one-off test** is the exception: a test tied to a specific, non-recurring activity - verifying a data migration, reproducing a production incident, checking a one-time environment state.

### Storage

```
tests/
  regression/    ← default; run by `make test`
  one_off/       ← quarantine; never run by `make test`
```

`make test` must only reference `tests/regression/`.

### Marker

All one-off tests must carry a marker with a mandatory `issue` reference, using whichever annotation mechanism the project's test framework provides:

```python
# Python/pytest - illustrative only
@pytest.mark.one_off(issue="#123")
def test_migration_user_records_backfilled():
    ...
```

If the framework has no native marker, encode the issue reference in the test name: `test_migration_user_records_backfilled_OT007`.

A one-off test without an `issue` reference must fail linting. Both directory placement and marker are required.

### Running one-off tests

```bash
make test-one-off          # run all one-off tests
make test-one-off ISSUE=123  # run one-off tests for a specific issue
```

### Decision rule

> **If in doubt, it's a regression test.**

Only place a test in `one_off/` if you can answer: *"After this issue is closed, will this test ever be meaningful to run again?"* If the answer is yes, or you are unsure, it belongs in regression.

### Lifecycle

One-off tests are temporary. They must be reviewed and deleted once their associated issue is closed and verified.

## Test naming

```
test_<unit>_<scenario>_<expected_result>
```

Examples:
- `test_login_with_invalid_password_returns_401`
- `test_cart_add_item_increases_total`
- `test_export_empty_list_returns_empty_file`

The test name should tell you what failed without reading the code.

## Test IDs

All tests carry a unique ID. The prefix indicates type:

| Prefix | Type | Location | Run by |
|--------|------|----------|--------|
| `RT-NNN` | Regression test | `tests/regression/` | `make test` |
| `OT-NNN` | One-off test | `tests/one_off/` | `make test-one-off` |
| `UT-NNN` | User test | documented in issue only | human |

Numbers are zero-padded to three digits. Each prefix has its own sequence.

**User tests (`UT-NNN`)** require a human to perform and verify manually. They have no corresponding test file. Description, steps, and expected outcome are documented in the AC table. Only Taḋg can mark a UT as passing or failing.

### Usage in markers

```python
# Python/pytest - illustrative only
@pytest.mark.regression(test_id="RT-042")
def test_login_with_invalid_password_returns_401():
    ...

@pytest.mark.one_off(issue="#123", test_id="OT-007")
def test_migration_user_records_backfilled():
    ...
```

If the framework has no native marker, encode the ID in the test name as a suffix:
```
test_login_with_invalid_password_returns_401_RT042
```

### Usage in GitHub issues

```
| AC-1 | Login rejects invalid passwords | ✅ RT-042 |
| AC-2 | Migration backfills user records | ✅ OT-007 |
```

### ID allocation

`tests/NEXT_IDS.txt` is the source of truth:

```
RT 043
OT 008
UT 001
```

Read the file, take the current value, increment, write back - in the same commit as the test.

If `tests/NEXT_IDS.txt` does not exist, create it and seed all sequences at `001`.

Test ID allocation is not a code change and may be done during issue preparation.

### Mid-project migration

#### Directory structure

Projects with a flat `tests/` directory must be migrated before any other work:

1. Move all existing tests into `tests/regression/`
2. Create `tests/one_off/.gitkeep`
3. Update `make test` to point at `tests/regression/`
4. Commit: `chore: migrate test suite to regression/one_off layout`
5. Then proceed with the issue

This must be its own commit.

#### Test IDs

1. **Do not retrofit IDs.** Pre-existing tests without IDs are not modified solely to add one.
2. **Migrate on touch.** If a pre-existing test is modified as part of an issue, add an ID then.
3. **New tests always get an ID.** No exceptions.
4. **Do not flag missing IDs unprompted.** Note in the issue comment but do not modify the test.

## Test structure

Follow Arrange-Act-Assert (AAA) regardless of language:

```
test "discount applied to eligible order":
    # Arrange
    order = Order(items=[item_over_threshold])
    # Act
    result = apply_discount(order)
    # Assert
    assert result.discount_percent == 10
```

## Coverage

- **Minimum:** 80% line coverage for new code
- **Critical paths:** 100% for authentication, payment, data persistence
- Coverage is a floor, not a ceiling - high coverage with weak assertions is worthless

## Test boundaries

| Type | Scope | Speed | When to run |
|------|-------|-------|-------------|
| Unit | Single function/class | Fast (<100ms) | Every save |
| Integration | Multiple components | Medium (<5s) | Pre-commit |
| End-to-end | Full system | Slow | CI pipeline |

- Unit tests: no external dependencies (no network, filesystem, database)
- Integration tests: may use local resources (test database, mock servers)
- E2E tests: exercise the real system

## Test data

- Use factories for generating test objects (factory_boy, FactoryBot, fishery, etc.)
- Fixtures for shared setup - minimal and obvious
- No production data - synthetic only
- Deterministic: same result every run (no random without seed)

## The "no mocks" rule

The rule in CODING.md means **do not mock your way out of testing real behaviour**.

**Acceptable:** mocking external HTTP APIs, time/dates, filesystem for unit tests (prefer tmpdir).

**Not acceptable:** mocking the class under test, mocking so much the test verifies nothing real, using mocks instead of fixing design to be testable.

If removing the mock would make the test fail for reasons *other than* external dependencies, you're mocking too much.

## Warnings and errors

- Never introduce new warnings or errors
- If an action generates a warning or error, stop and resolve it
- Build-time errors are equivalent to compile errors - fix immediately

## Policy

- All production code requires unit, integration, and end-to-end tests
- Test coverage must be runnable via `make test`
- Update regression tests; create them if none exist

**Exception:** requires explicit approval - Taḋg must say **"I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME"**

## Makefile test targets

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

`make test` must never reference `tests/one_off/`.
