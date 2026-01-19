
For each github issue number above, perform the following steps:

- Before you start, read our operating principles in CLAUDE.md. Read all of the docs referenced in CLAUDE.md. Read all of the project documentation (README and everything under ./docs/)
  - pay special attention to vision, implementation guide, patterns, design principles (if any)
- Confirm each step you are doing in real time
- Fetch the latest issue details with `gh issue view [n]`, review the issue and all comments as there may be new comments or edits since you last looked
- Review affected files, review the issue implementation specification against the design principles and patterns and if there are any major contradictions, or if the solution is not sound, DO NOT PROCEED and alert me in real-time.
- Update project documentation as appropriate
- Commit changes with message `Fixes #[n]: [short description]` and push branch to remote
- Add a comment to the issue describing implementation details and testing instructions, and commit link
- NEVER MARK AN ISSUE AS CLOSED! That is my job, after I review your work.
