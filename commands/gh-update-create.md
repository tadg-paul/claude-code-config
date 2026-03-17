**DO NOT WRITE ANY CODE. THIS IS A DOCUMENTATION ACTIVITY.**

Before you start:
- If you have not done so already, make sure you have read and digested our operating principles in CLAUDE.md and related docs, as well as the project documentation including README and everything under ./docs/

**Issue content standard**

A well-formed issue must contain:
- A clear problem statement or feature description
- A solution section (see below for rules on creating/updating this)
- If there are multiple options to address the issue, list them all and make a recommendation
- A table of Acceptance Criteria with the following columns:
  - **ID** - to identify the AC
  - **AC** - what must be true
  - **Test** - which test proves it and how
  - **Status** - tracked during implementation

You may create sub-issues as needed to break down complex work. Sub-issues are appropriate where a discrete piece of work can be tracked and tested independently, or where the scope is large enough to warrant it (e.g. a code review or significant refactor).

**Creating a new issue**

- Review affected files
- Create the issue with a body conforming to the issue content standard above

**Updating existing issues**

For each issue number provided:
- Fetch issue details with `gh issue view [n]`, review the issue and all comments
- Review affected files
- We do not want to litter the issue with multiple solutions, and we do not want to leave old solutions that have been superseded
  - If there is no solution documented yet, add a new comment with the solution
  - Otherwise, edit the existing solution text to reflect the updated solution
- Add a comment summarizing the changes made to the solution
