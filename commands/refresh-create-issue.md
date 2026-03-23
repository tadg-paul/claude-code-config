**DO NOT WRITE ANY CODE. THIS IS A DOCUMENTATION ACTIVITY.**

Exception: allocating test IDs (RT-NNN/OT-NNN/UT-NNN) in AC tables and creating `tests/NEXT_IDS.txt` if it does not yet exist are permitted — these are issue preparation tasks, not code changes.

Before you start:
- Read and digest CLAUDE.md and related docs if you have not already done so.
- All issues and sub-issues must conform to the standards in @~/.claude/docs/ISSUES.md.

# Creating a new issue

- Review affected files.
- Draft the issue body. 
- **CRITICAL SELF-AUDIT BEFORE POSTING:** Look at the Acceptance Criteria you just drafted. If any AC uses action verbs (call, assert, send, check, verify, should return), you have written a test, not an AC. You MUST rewrite it to describe the required *state of the system* before creating the issue.
- Create the issue with the body conforming to @~/.claude/docs/ISSUES.md.
- Provide me the URL to the issue and wait for me to say "APPROVED" before taking any further action.

# Updating existing issues

For each issue number provided:
- Fetch issue details with `gh issue view [n]`, review the issue and all comments.
- Review affected files.
- Update the solution and AC table per the rules in @~/.claude/docs/ISSUES.md.
- **CRITICAL SELF-AUDIT:** Ensure no ACs are written as tests (no action/assertion verbs).
- Add a comment summarizing the changes made.