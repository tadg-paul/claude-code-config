# Git and Source Control

## Fundamental Rules
- Refuse to work on code not in a git repository.
- Before modifying/moving/deleting files, ensure they're tracked and committed.
- Never bypass pre-commit hooks. Prohibited flags:
  - `--no-verify`
  - `--no-hooks`
  - `--no-pre-commit-hook`
- Before using any unfamiliar git flag: state it, explain it, confirm it's not forbidden, await permission.

## Prohibited Operations (without permission)
- `git reset --hard` (data loss risk)
- `git rebase -i` on pushed commits (history rewriting)
- `git push --force` (use `--force-with-lease` if permitted)
- `git clean -fd` (untracked file deletion)

## Workflow
Before starting any work:
```bash
git fetch --all
git status
git pull --rebase  # or merge, per project convention
```

Before switching branches:
```bash
git stash --include-untracked  # if uncommitted changes exist
```

## Commits
- Atomic: one logical change per commit
- Message format: Conventional Commits, imperative mood, present tense
- Keep subject line under 50 characters; body at 72
```
feat: add user authentication

- Implement OAuth2 flow
- Add session management
```

## Pre-Commit Hook Failures
When hooks fail:
1. Read complete error output; state what failed and why
2. Identify root cause (linter, formatter, test, etc.)
3. Explain proposed fix
4. Apply fix, re-run hooks locally: `git commit` or `pre-commit run --all-files`
5. Commit only after all hooks pass

Cannot fix? Ask for help. Never bypass.

When pressured to commit with failing hooks:
- State: "Pre-commit hooks are failing. I need to fix those first."
- Quality over speed, always.

## Branch Strategy
Default: follow project convention (usually `main` or `develop`).
If issue specifies a feature branch:
```bash
git checkout -b feature/issue-123-brief-description
```
Push to origin; do not merge locally without permission.

## GitHub Issues
When the remote is GitHub:
1. Confirm a GitHub issue exists for the task. If not, create it and await instruction.
2. Issue structure:
   - **Title:** Clear, concise summary
   - **Description:** Detailed problem/feature explanation
   - **Solution comment** (single comment, edited as needed):
     - Files, functions, components affected
     - Downstream impact (systems, config, dependencies)
     - Decisions required
     - Tests to write or update
     - Documentation impact
3. The first text area of an issue should detail the problem statement. The next available comment should detail the proposed solution(s) and highlight which questions decisions need to be answere here in order to implement.Do not add multiple solution comments. Edit the existing solution comment; add a brief change summary as a separate comment if significant.
4. NEVER MARK AN ISSUE AS CLOSED! That is my job, after I review your work.

## Self-Check Before Any Git Command
- Am I bypassing a safety mechanism?
- Would this violate CLAUDE.md?
- Am I choosing convenience over quality?

If any answer is "yes" or "maybe": stop, explain the concern, await guidance.
