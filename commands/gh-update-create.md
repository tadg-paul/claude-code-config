**DO NOT WRITE ANY CODE. THIS IS A DOCUMENTATION ACTIVITY.**

Before you start:
- If you have not done so already, make sure you have read and digested our operating principles in CLAUDE.md and related docs, as well as the project documentation including README and everything under ./docs/

# Issue content standard

A well-formed issue must contain:
- A clear problem statement or feature description
- A solution section (see below for rules on creating/updating this)
- If there are multiple options to address the issue, list them all and make a recommendation
- A table of Acceptance Criteria with the following columns:

| Column | Purpose |
|---|---|
| **ID** | Format: `AC{issue}.{n}` — e.g. `AC12.1` |
| **AC** | A single, falsifiable *state of the system* — not a test, not a user action, not an implementation step. Write it as: *"Given [context], [system] [does/returns/stores/rejects] [X]."* If it describes something a test *does*, rewrite it as what the system *must be true of*. |
| **Test** | How to verify the AC is met. Name the test(s) and briefly describe the stimulus and expected observable output. Multiple tests per AC are allowed. |
| **Status** | `pending` / `passing` / `failing` / `skipped` |

## The AC/Test boundary (the most common mistake)

> An **AC** describes *what must be true about the system*.  
> A **Test** describes *how you confirm it is true*.

- **Wrong AC:** *"Call the API with an invalid token and assert a 401 is returned."*  
- **Right AC:** *"The API rejects requests with invalid tokens."*  
- **Right Test:** *"Send request with malformed JWT → assert HTTP 401 and `error: unauthorized` body."*

If your AC contains words like *call*, *assert*, *send*, *check*, *verify*, or *should return*, it is probably a test description. Rewrite it.

# Creating a new issue

- Review affected files
- Create the issue with a body conforming to the issue content standard above

## Complex issues and sub-issues
You may create sub-issues as needed to break down complex work. Sub-issues are appropriate where a discrete piece of work can be tracked and tested independently, or where the scope is large enough to warrant it (e.g. a code review or significant refactor).

# Updating existing issues

For each issue number provided:
- Fetch issue details with `gh issue view [n]`, review the issue and all comments
- Review affected files
- We do not want to litter the issue with multiple solutions, and we do not want to leave old solutions that have been superseded
  - If there is no solution documented yet, add a new comment with the solution
  - Otherwise, edit the existing solution text to reflect the updated solution
- Add a comment summarizing the changes made to the solution
