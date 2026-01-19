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
