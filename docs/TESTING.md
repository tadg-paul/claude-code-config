# Testing Standards

## Test-Driven Development

We practice TDD:

1. Write a failing test defining desired behaviour
2. Run test, confirm it fails as expected
3. Write minimal code to pass
4. Run test, confirm success
5. Refactor while keeping tests green
6. Repeat for each feature/bugfix

## Policy

- All production code requires unit tests, integration tests, AND end-to-end tests.
- Implement with test coverage that can be run via `make test`
- Update regression tests, create regression tests if there are none

**Exception:** Requires explicit approval in commit message:
```
SKIP TESTS: [specific reason]
```

To authorize skipping: I must say **"I AUTHORIZE YOU TO SKIP WRITING TESTS THIS TIME"**

## Requirements

- Tests MUST cover functionality being implemented
- Never ignore test output - logs often contain critical information
- Test output must be pristine to pass
- If logs should contain errors, capture and test them

## Quality

- Clear, maintainable tests
- One behaviour per test
- Descriptive names explaining what's tested
- Include edge cases and error conditions
- Tests are documentation - write for the next developer

## Warnings and Errors

- Never introduce new warnings or errors.
- If an action generates a warning or error, stop everything and resolve it.
- Build-time errors (e.g., Hugo static site) are equivalent to compile errors - fix immediately.

# Mandatory

Every interaction with me must begin with "EHLO!"
