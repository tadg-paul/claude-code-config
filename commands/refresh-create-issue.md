**DO NOT WRITE ANY CODE. THIS IS A DOCUMENTATION ACTIVITY.**

Exception: allocating test IDs (RT-NNN/OT-NNN/UT-NNN) in AC tables and creating `tests/NEXT_IDS.txt` if it does not yet exist are permitted — these are issue preparation tasks, not code changes.

Before you start:
- Read and digest CLAUDE.md and related docs if you have not already done so.
- All issues and sub-issues must conform to the standards in @~/.claude/docs/ISSUES.md.

# Creating a new issue

- Review affected files.
- Draft the issue body.
- **CRITICAL SELF-AUDIT BEFORE POSTING:** Run the self-audit procedure defined in @~/.claude/docs/ISSUES.md on every AC row. Check for forbidden language (action verbs, passive test phrasings, test-structure language). Apply the litmus test. If any AC fails, rewrite it.
- For each AC with multiple implied conditions, ensure the Test column accounts for every condition — not just one.
- Create the issue with the body conforming to @~/.claude/docs/ISSUES.md.
- Provide me the URL to the issue, then respond with `AWAITING APPROVAL — issue #NNN` and produce no further output until I say APPROVED.

# Updating existing issues

For each issue number provided:
- Fetch issue details with `gh issue view [n]`, review the issue and all comments.
- Review affected files.
- Update the solution and AC table per the rules in @~/.claude/docs/ISSUES.md.
- **CRITICAL SELF-AUDIT:** Run the full self-audit procedure on every AC row.
- Add a comment summarizing the changes made.
