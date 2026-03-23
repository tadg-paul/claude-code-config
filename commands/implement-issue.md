# Implementation Workflow

**CRITICAL GATE: Do not begin these steps unless I have explicitly given you the "APPROVED" signal for this specific issue.**

Before you start:
- If you have not done so already, make sure you have read and digested our operating principles in CLAUDE.md and related docs, as well as the project documentation including README and everything under ./docs/
- All issue updates made during implementation must conform to @~/.claude/docs/ISSUES.md

For each github issue number provided, perform the following steps IN STRICT ORDER:

1. Fetch issue details with `gh issue view [n]`, review the issue and all comments - there may have been changes since you last looked.
2. **Check for Solution & Approval:**
   - If there is no documented solution in the issue, create a new comment with the proposed solution. **THEN STOP. Do not proceed to code. Ask me for approval and wait for me to say "APPROVED".**
3. **Audit the Acceptance Criteria:**
   - Check the issue for an Acceptance Criteria (AC) table.
   - **Refuse to proceed** if the ACs are missing.
   - **Refuse to proceed** if the ACs are written as tests. If any AC uses words like *call*, *assert*, *send*, *check*, *verify*, or *should return*, it is a test, not an AC. Alert me that the ACs violate the AC/Test boundary rules in `@~/.claude/docs/ISSUES.md` and stop.
4. **Refuse to proceed** if ANY of the following are missing from the issue:
   - Solution outline
   - Explicit approval from me to implement the solution
5. Confirm each step you are doing with me in real time.
6. Review affected files, review the issue implementation specification against the design principles and patterns. If there are any major contradictions, or if the solution is not sound, DO NOT PROCEED and alert me in real-time.
7. Don't forget TDD-first! Start with failing tests.
8. Update project documentation as appropriate.
9. Commit changes with message `Fixes #[n]: [short description]` and push branch to remote.
10. Add a new comment to the issue describing implementation details, testing instructions, commit link and update the AC table with status of each AC.
11. NEVER MARK AN ISSUE AS CLOSED! That is my job, after I review your work.