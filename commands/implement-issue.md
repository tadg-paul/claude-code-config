# Implementation Workflow

**CRITICAL GATE: Do not begin these steps unless I have explicitly given you the "APPROVED" signal for this specific issue.** If you have not seen APPROVED in my most recent message, stop. Re-read section 1 of CLAUDE.md.

Before you start:
- If you have not done so already, make sure you have read and digested our operating principles in CLAUDE.md and related docs, as well as the project documentation including README and everything under ./docs/
- All issue updates made during implementation must conform to @~/.claude/docs/ISSUES.md

For each github issue number provided, perform the following steps IN STRICT ORDER:

1. Fetch issue details with `gh issue view [n]`, review the issue and all comments — there may have been changes since you last looked.
2. **Check for Solution & Approval:**
   - If there is no documented solution in the issue, create a new comment with the proposed solution. **THEN STOP. Do not proceed to code. Respond with `AWAITING APPROVAL — issue #NNN` and produce no further output.**
3. **Audit the Acceptance Criteria:**
   - Check the issue for an Acceptance Criteria (AC) table.
   - **Refuse to proceed** if the ACs are missing.
   - **Refuse to proceed** if any AC violates the AC/Test boundary. Run the self-audit procedure defined in `@~/.claude/docs/ISSUES.md` on every AC row. If any AC uses forbidden language (action verbs, passive test phrasings, test-structure language as listed in ISSUES.md), alert me and stop.
4. **Refuse to proceed** if ANY of the following are missing from the issue:
   - Solution outline
   - Explicit approval from me to implement the solution
5. Confirm each step you are doing with me in real time.
6. Review affected files, review the issue implementation specification against the design principles and patterns. If there are any major contradictions, or if the solution is not sound, DO NOT PROCEED and alert me in real-time.
7. **Enumerate test cases before writing tests.**
   - For each AC, list every distinct condition it implies.
   - Post this enumeration as a comment on the issue.
   - Each condition must map to at least one test. An AC with three conditions requires at least three tests — not one.
   - Then proceed with TDD: write failing tests for all enumerated conditions.
8. TDD cycle: run issue tests, confirm they fail. Write minimal code to pass. Run issue tests, confirm success. Refactor while keeping issue tests green.
9. **Demonstrate the fix or feature.**
   - After implementation, show the specific scenario described in the issue working correctly. Running tests is necessary but not sufficient — show me the actual output.
   - Never state that an issue is fixed. State that the tests pass and show the evidence. I will determine whether it is fixed.
10. Update project documentation as appropriate.
11. Commit changes with message `Fixes #[n]: [short description]` and push branch to remote.
12. Add a new comment to the issue describing implementation details, testing instructions, commit link and update the AC table with status of each AC.
13. NEVER MARK AN ISSUE AS CLOSED! That is my job, after I review your work.
