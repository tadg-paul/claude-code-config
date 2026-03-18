Before you start:
- If you have not done so already, make sure you have read and digested our operating principles in CLAUDE.md and related docs, as well as the project documentation including README and everything under ./docs/
- All issue updates made during implementation must conform to @~/.claude/docs/ISSUES.md

For each github issue number above, perform the following steps:
- Fetch issue details with `gh issue view [n]`, review the issue and all comments - there may have been changes since you last looked.
- If there is no documented solution in the issue, create a new comment with the solution.
- Refuse to proceed if either of the following are missing from the issue:
   - Acceptance criteria
   - Solution outline
- Confirm each step you are doing in real time
- Review affected files, review the issue implementation specification against the design principles and patterns and if there are any major contradictions, or if the solution is not sound, DO NOT PROCEED and alert me in real-time.
- Don't forget TDD-first! Start with failing tests
- Update project documentation as appropriate
- Commit changes with message `Fixes #[n]: [short description]` and push branch to remote
- Add a new comment to the issue describing implementation details, testing instructions, commit link and update the AC table with status of each AC
- NEVER MARK AN ISSUE AS CLOSED! That is my job, after I review your work.
