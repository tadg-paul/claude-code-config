# GitHub Issue Standards

This document defines issue structure and AC quality standards. The process workflow - when to create issues, when to stop for approval, when to audit - is in CLAUDE.md §3.

---

## Well-formed issue

A well-formed issue must contain:

- A clear problem statement or feature description
- A solution section (see rules below)
- If there are multiple options, list them all and make a recommendation
- A table of Acceptance Criteria (see below)

---

## Acceptance Criteria table

| Column | Purpose |
|---|---|
| **ID** | Format: `AC{issue}.{n}` - e.g. `AC12.1` |
| **AC** | A single, falsifiable *state of the system* - not a test, not a user action, not an implementation step. Write it as: *"Given [context], [system] [does/returns/stores/rejects] [X]."* If it describes something a test *does*, rewrite it as what the system *must be true of*. |
| **Test** | How to verify the AC is met. Name tests with their issue-scoped IDs (RT-{issue}.{n} for regression, OT-{issue}.{n} for one-off, UT-{issue}.{n} for user tests). Briefly describe stimulus and expected observable output for each. **Multiple tests per AC are expected and the norm.** Each test on a new line. |
| **Status** | `⏳ pending` / `⚠️ blocked` / `✅ passing` / `❌ failing` / `🚫 removed` |

### Single source of truth

Each issue has exactly one AC table - in the issue body, or the first comment if GitHub's interface required the solution and ACs to be posted there.

- Never create a second AC table in a later comment, even if ACs have changed.
- If ACs need to change, edit the existing table in place.
- Add a comment summarizing what changed and why.
- Multiple AC tables in an issue is a known failure mode: it creates ambiguity about which ACs are current.

### AC quality heuristic

A well-defined AC describes a single, falsifiable system state - and nearly always requires more than one test to verify. An AC with exactly one test is a smell: either the AC is too narrow (and is really a test in disguise), or the test coverage is incomplete.

Before accepting single-test coverage for any AC, ask: *"What other inputs, edge cases, or boundary conditions could falsify this?"* If the answer is none, the AC may need rewriting. If the answer is several, the tests are missing.

### Multi-condition ACs require multi-condition coverage

If an AC implies multiple distinct conditions, it requires a test for each condition. An AC like *"The API rejects requests with invalid, expired, or missing tokens"* implies three conditions: invalid, expired, and missing. Each condition requires its own test. Writing a single test for one of the three conditions and marking the AC as passing is a process violation.

Before writing tests, enumerate the conditions each AC implies. Post this enumeration as a comment on the issue. Every condition must map to at least one test.

---

## The AC/Test boundary

> An **AC** describes *what must be true about the system*.
> A **Test** describes *how you confirm it is true*.

### The litmus test

Ask: *"Could this statement be true or false without specifying how it is observed?"* If not - if it describes an action someone performs to check something - it is a test, not an AC.

### Forbidden language in the AC column

The following words and phrasings are **strictly forbidden** in the AC column. If any AC contains them, rewrite it before proceeding.

**Action verbs:** call, assert, send, check, verify, run, execute, invoke, trigger, submit, click, request, query, fetch, post

**Passive test phrasings:** "is returned", "results in", "produces", "yields", "should return", "should produce", "responds with", "outputs"

**Test-structure language:** "when you", "if you", "given that we send", "after calling"

### Examples

| ❌ Wrong (test description) | ✅ Right (system state) |
|---|---|
| Call the API with an invalid token and assert a 401 is returned. | The API rejects requests with invalid tokens with HTTP 401. |
| Send a POST to /users with a duplicate email and check that a 409 is returned. | The system prevents duplicate user registration. |
| Run the export command with an empty dataset and verify the output file is empty. | Exporting an empty dataset produces an empty file. |
| Check that the config file is created in ~/.config/app/ after installation. | Installation creates a configuration file at ~/.config/app/. |
| Call the validate function with a malformed date string and assert it raises ValueError. | The validator rejects malformed date strings. |
| Verify that when the --verbose flag is passed, debug output is printed to stderr. | The --verbose flag enables debug output on stderr. |

### Self-audit procedure

After drafting ACs, perform this audit on every row of the AC table:

1. Read the AC column aloud. Does it describe an action someone performs, or a state the system is in?
2. Does it contain any word from the forbidden list above?
3. Apply the litmus test: could this be true or false without specifying how to observe it?

If any AC fails any of these checks, rewrite it before creating or updating the issue.

---

## Solution section rules

- If no solution is documented yet, add a new comment with the solution
- Do not litter the issue with multiple superseded solutions
- If a solution already exists, edit it in place to reflect the updated approach
- Add a comment summarizing what changed and why

---

## Sub-issues

Sub-issues may be created as needed to break down complex work. They are appropriate where:
- A discrete piece of work can be tracked and tested independently
- The scope is large enough to warrant it (e.g. a major refactor, a code review)

Every sub-issue must also conform to this standard - a well-formed issue with AC table, solution outline, and test IDs. Sub-issues are not lightweight stubs.

---

## AC and test ID allocation

- ACs follow the pattern `AC{issue}.{n}` — e.g. the first AC in issue #12 is `AC12.1`, then `AC12.2`, and so on.
- Test IDs follow the same issue-scoped pattern: `RT-{issue}.{n}`, `OT-{issue}.{n}`, `UT-{issue}.{n}` — e.g. the first regression test for issue #12 is `RT-12.1`, the second is `RT-12.2`. Each prefix has its own sequence within the issue.
- Once an ID has been allocated for an AC or test it is immutable. Never renumber, reuse, or delete IDs. If an AC is removed, mark it as `🚫 removed` in the table but do not delete the row or its ID. If a test is removed, mark it as removed in the test suite but do not delete the test or its ID.
