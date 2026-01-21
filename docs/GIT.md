# Git and Source Control

## Fundamental Rules

- **Never use `--no-verify`** when committing. Ever.
- Refuse to work on code not in a git repository.
- Before moving, changing, or removing files, ensure they're tracked and recoverable.

## Forbidden Flags

These are prohibited:
- `--no-verify`
- `--no-hooks`
- `--no-pre-commit-hook`
- NEVER state "Co-authoried by Claude" or any AI

Before using any unfamiliar git flag: state it, explain why, confirm it's not forbidden, get permission for any bypass.

## Commit Standards

Messages must be:
- Concise and descriptive
- Imperative mood, present tense
- Conventional commit format

```
feat: add user authentication
fix: resolve memory leak in data processor
docs: update API documentation
refactor: simplify validation logic
```

## Workflow

Before starting work:

```bash
git pull
git status
```

Ensure local repo is current with remote.

## Pre-Commit Hook Failures

When hooks fail, follow this sequence exactly:

1. Read complete error output (explain what you see)
2. Identify which tool failed and why
3. Explain the fix and why it addresses root cause
4. Apply fix, re-run hooks
5. Commit only after all hooks pass

Cannot fix? Ask for help. Never bypass.

## Pressure Response

When asked to commit/push with failing hooks:

- Do not rush to bypass quality checks
- State: "Pre-commit hooks are failing, I need to fix those first"
- Work through systematically
- Quality over speed, even when waiting

Pressure is never justification for bypassing checks.

## Accountability

Before any git command, ask:

- Am I bypassing a safety mechanism?
- Would this violate CLAUDE.md?
- Am I choosing convenience over quality?

If any answer is "yes" or "maybe", explain the concern first.

## Branch Strategy

Default: work on master.

If the issue specifies a feature branch:
- Create the branch
- Work only on that branch
- Push to origin

Alternatives available: worktrees, feature branches.

## Exception Process

When a rule genuinely cannot be followed (not "is inconvenient"):

1. **Document the situation** — what rule, why it can't be followed
2. **Get explicit approval** — from the project owner, in writing (issue comment, PR comment)
3. **Record the exception** — in the commit message or PR description
4. **Set an expiry** — exceptions should be temporary; create a follow-up issue to resolve properly

Format for commit messages with exceptions:
```
fix: emergency patch for production outage

EXCEPTION: Skipping integration tests - test DB unavailable
APPROVED BY: @owner in #123
FOLLOW-UP: #124 to restore test coverage
```

Exceptions are not get-out-of-jail-free cards. They create technical debt that must be repaid.
