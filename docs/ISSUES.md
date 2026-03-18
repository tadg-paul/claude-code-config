# GitHub Issue Standards

This document is always in context. All issue creation and updates must conform to these standards, whether triggered by a slash command, plan mode, or any other activity.

## Well-formed issue

A well-formed issue must contain:

- A clear problem statement or feature description
- A solution section (see rules below)
- If there are multiple options, list them all and make a recommendation
- A table of Acceptance Criteria (see below)

## Acceptance Criteria table

| Column | Purpose |
|---|---|
| **ID** | Format: `AC{issue}.{n}` — e.g. `AC12.1` |
| **AC** | A single, falsifiable *state of the system* — not a test, not a user action, not an implementation step. Write it as: *"Given [context], [system] [does/returns/stores/rejects] [X]."* If it describes something a test *does*, rewrite it as what the system *must be true of*. |
| **Test** | How to verify the AC is met. Name the test(s) with their RT-NNN/OT-NNN IDs and briefly describe the stimulus and expected observable output. Multiple tests per AC are allowed. |
| **Status** | `pending` / `passing` / `failing` / `skipped` |

## The AC/Test boundary (the most common mistake)

> An **AC** describes *what must be true about the system*.
> A **Test** describes *how you confirm it is true*.

- **Wrong AC:** *"Call the API with an invalid token and assert a 401 is returned."*
- **Right AC:** *"The API rejects requests with invalid tokens."*
- **Right Test:** *"RT-042: Send request with malformed JWT → assert HTTP 401 and `error: unauthorized` body."*

If your AC contains words like *call*, *assert*, *send*, *check*, *verify*, or *should return*, it is probably a test description. Rewrite it.

## Solution section rules

- If no solution is documented yet, add a new comment with the solution
- Do not litter the issue with multiple superseded solutions
- If a solution already exists, edit it in place to reflect the updated approach
- Add a comment summarising what changed and why

## Sub-issues

Sub-issues may be created as needed to break down complex work. They are appropriate where:
- A discrete piece of work can be tracked and tested independently
- The scope is large enough to warrant it (e.g. a major refactor, a code review)

Every sub-issue must also conform to this standard — a well-formed issue with AC table, solution outline, and test IDs. Sub-issues are not lightweight stubs.

## Test ID allocation in issues

Allocating test IDs (RT-NNN/OT-NNN) in AC tables and creating `tests/NEXT_IDS.txt` if it does not yet exist are **not code changes**. They may be done at any time as part of issue preparation without an approved issue.
