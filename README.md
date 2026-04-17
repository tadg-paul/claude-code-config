# Claude Code SDLC Configuration

A software development lifecycle framework for human-AI pair programming with Claude Code. Developed iteratively through real project work, refined by failure, and designed around one principle: the human drives, the AI executes.

## The Problem

Claude Code is capable but undisciplined. Without constraints it exhibits predictable failure modes:

- **Premature implementation.** It writes code before requirements are agreed, solving the wrong problem efficiently.
- **Shortcut testing.** It writes tests that exercise internal APIs instead of real user entry points. Tests pass; the application is broken. A TTS tool shipped with a deadlocking subprocess because the test suite never spawned one.
- **Error suppression.** It silences failures with `|| true` instead of handling them, defeating the safety nets it was told to use.
- **Self-authorization.** It declares its own work satisfactory and moves on without review.
- **Scope creep.** It "improves" surrounding code, adds features not requested, and refactors things that work.

Every rule in this framework exists because one of these failures occurred in production work. None are theoretical.

### Observed failure patterns in detail

Beyond the headline failures, specific bad practices recur across projects and languages:

**Testing shortcuts:**
- Calling a library function in a test when the spec says "run the CLI command" -- the test passes but the real binary, with its subprocess spawning, path resolution, and signal handling, is never exercised.
- Asserting a database row exists when the spec says "user sees a confirmation page" -- the row is inserted but the view that displays it is broken.
- Grepping source code to verify behaviour instead of running anything -- a test that checks whether a function definition exists, not whether it works.
- Writing one test per AC even when the AC implies multiple conditions (e.g. "rejects invalid, expired, and missing tokens" tested with only "invalid").

**Error handling shortcuts:**
- `cmd || true` and `cmd || :` to silence failures under `set -e`, defeating the safety header.
- `cmd || rc=$?` to capture the exit code while suppressing the `set -e` trap -- syntactically clever, semantically identical to disabling error checking.
- `set +e` / `set -e` bracketing to temporarily disable safety for a "tricky" section.
- `2>/dev/null` with no fallback logic, hiding diagnostic output that would reveal the real problem.
- Bare `except: pass` in Python, empty `catch {}` in Swift, `catch (Exception e) {}` in Java -- swallowing errors across every language.
- `((count++)) || true` to work around the arithmetic expression gotcha in bash, instead of using `count=$((count + 1))`.

**Process shortcuts:**
- Writing gate keywords (SATISFIED, PROCEED, APPROVED) into its own output to self-authorize advancement.
- Declaring work "fixed", "done", or "complete" when tests pass but no demonstration of actual behaviour has occurred.
- Overwriting human edits to issues, code, or documentation because it disagrees with the change.
- Treating a question ("What do you think of X?") as an instruction to go and implement X.
- Silently deleting AC rows, test statuses, or comments from issues rather than editing them in place.

**Coding shortcuts:**
- Using `eval`, string concatenation, or variable expansion to build and execute shell commands.
- `sed` and `awk` for source code modifications instead of AST-aware tools.
- Hardcoded credentials, disabled SSL verification, `chmod 777`.
- Functions exceeding 50 lines, nesting deeper than 3 levels, god objects.
- `rm` instead of `trash`, destroying recoverability.

### The language coverage challenge

Coding standards can enumerate prohibited patterns for commonly used languages (shell, Python, Swift), but cannot exhaustively list anti-patterns for every language and framework that might be encountered. The same underlying tendency -- taking the narrowest shortcut that technically satisfies the requirement -- manifests differently in each ecosystem.

The mitigation is twofold: CODING.md documents the general principles and the language-specific pitfalls that have been encountered, while the `/audit-code` skill reviews against both CODING.md and the best practice standards for whatever language and stack is actually in use. This means the audit adapts to the project rather than being limited to what was pre-documented.

## Objectives

1. **Human retains control.** No code is written, no issue is closed, no decision is made without explicit human authorization at defined checkpoints.
2. **Quality through discipline.** TDD, real-behaviour testing, proper error handling, and coding standards are enforced by process, not by trust.
3. **Traceability.** Every change traces to an issue, every issue has acceptance criteria, every AC has tests, every test has an ID. Nothing is ad hoc.
4. **No shortcuts.** The framework explicitly names and prohibits the shortcuts Claude Code is known to take, and requires visible accountability (e.g. answering the "real-user test" question in chat) rather than relying on rules it can reinterpret.

## What Was Tried

### Phase 1: Traffic lights

The initial approach used a Green/Amber/Red classification for actions. Green (autonomous): fix tests, correct typos, add imports. Amber (propose first): multi-file changes, new features. Red (always ask): rewriting working code, security changes, data loss risks.

This was too loose. Claude treated most work as Green, wrote code before requirements were clear, wrote tests that checked internal state rather than behaviour, and declared things "done" without evidence.

### Phase 2: Single approval gate

A single APPROVED gate was introduced: no code could be written until the human explicitly typed APPROVED in chat. This stopped premature implementation but conflated three separate decisions (are the requirements right? is the solution sound? is the result acceptable?) into one checkpoint. Requirements problems were caught too late, after solution design or even implementation had already begun.

### Phase 3: Three gates with prescriptive checklist

The single gate was split into three: SATISFIED (requirements agreed), PROCEED (solution approved), APPROVED (result accepted). A 32-step sequential checklist enforced every phase.

This solved the quality problem but created a new one: Claude would grind through every phase sequentially, spending long periods on AC audits, test enumeration, and solution documentation before a line of code was written. Simple bug fixes received the same ceremony as architectural changes. The process felt like 1990s waterfall development.

### Phase 4: Skill-driven workflow (current)

The checklist was decomposed into invocable skills. The human calls the skill needed for the current moment; the quality checks are baked into each skill rather than front-loaded as a sequential script.

The three quality gates remain as hard stops requiring human keywords. The prohibitions remain absolute. But the human controls pacing -- a simple fix can move quickly through the gates, while a complex feature can invoke optional audits for deeper scrutiny.

## Where It Landed

### Architecture

```
~/.claude/
  CLAUDE.md              # Process rules, prohibitions, gates, failure modes
  commands/              # Invocable skills (slash commands)
    draft-issue.md       # Create issue with ACs and test specs
    audit-acs.md         # Challenge AC coverage (advisory)
    audit-tests.md       # Challenge test coverage (advisory)
    design-solution.md   # Document solution on issue
    write-tests.md       # Write test code only (TDD red phase)
    implement.md         # Write code to pass tests
    audit-code.md        # Review code against standards (advisory)
    review.md            # Full review: make test, standards, UT demos
  docs/
    ISSUES.md            # Issue structure, AC quality standards
    TESTING.md           # Testing standards, TDD, real-user test
    CODING.md            # Coding standards, prohibited patterns
    GIT.md               # Git workflow, commit standards, hook policy
    DOCUMENTATION.md     # Documentation standards, voice, sanitisation
```

### The reference documents

Each document in `docs/` addresses a specific craft discipline. All were present from the initial commit in some form (except ISSUES.md); all have evolved substantially through real-world use.

#### CODING.md -- Coding Standards

**What it covers:**
- Platform constraints (macOS/Apple Silicon locally, Linux for deployment)
- Language and tool selection decision hierarchy
- Style baselines for 12 language ecosystems, each mapped to its linter and formatter
- Deep shell scripting standards (safety header, quoting, arrays, prohibited patterns)
- Python standards (version management with `uv`, virtual environments, Ruff configuration, `src/` layout)
- Code commenting rules (ABOUTME headers, evergreen comments, no dead code)
- Code search and modification (ast-grep for transforms, never sed/awk)
- Comprehensive prohibited anti-patterns table
- Script standards (portability, error handling, security, logging)
- Dependency management and security auditing

**How it evolved:**
- Started as a short style guide covering only shell and Python, with a mandatory safety header (`set -euo pipefail`) and basic rules (quote variables, use `command -v` not `which`)
- The style baselines table was added to cover 12 languages after projects began spanning Swift, TypeScript, Ruby, Rust, Go, and others -- each mapped to the community-standard linter and formatter
- The prohibited anti-patterns section grew incrementally, each entry added after a real incident:
  - `|| true` and `|| rc=$?` after Claude repeatedly suppressed `set -e` to make scripts "work"
  - `eval` and string-built commands after injection-vulnerable patterns appeared in shell scripts
  - `sed`/`awk` for code modification after structural corruption in edited files
  - Bare `except: pass` and empty `catch {}` after errors were silently swallowed across Python, Swift, and Java
  - `((count++)) || true` after the arithmetic expression gotcha under `set -e` was mishandled repeatedly
- The language-specific pitfalls table now covers shell, Python, Swift, Java/C#, and Java, with each entry linked to the reason it is banned
- Linter bypass rules were added after `# noqa` and `# type: ignore` appeared without justification -- any suppression now requires the exact rule code and an explanation of why it cannot be fixed properly
- The "other prohibited shortcuts" table covers `sleep` for synchronization, retry without backoff, global state, monkey-patching, SQL concatenation, disabled SSL, `chmod 777`, and hardcoded credentials
- A language and tool selection section was added to prevent defaulting to a familiar language out of habit -- the decision hierarchy is: existing project language, then genuinely best fit, then simplest maintainable solution
- The key evolution was recognizing that prescriptive rules for specific languages are necessary but insufficient -- the `/audit-code` skill supplements by reviewing against the best practice standards for whatever stack is in use

#### TESTING.md -- Testing Standards

**What it covers:**
- The "real-user test" principle (the single most important rule in the framework)
- TDD workflow (enumerate, write failing tests, write code, refactor)
- Multi-condition coverage requirements
- Three test types: RT (regression), OT (one-off), UT (human verification)
- Decision tree for choosing between test types
- Directory layout (`tests/regression/` vs `tests/one_off/`)
- Test naming conventions (`test_<unit>_<scenario>_<expected_result>`)
- Test ID scheme (`RT-{issue}.{n}`, `OT-{issue}.{n}`, `UT-{issue}.{n}`)
- Arrange-Act-Assert structure
- Coverage requirements (80% floor for new code, 100% for critical paths)
- Test boundaries (unit, integration, end-to-end)
- Test data rules (factories, no production data, deterministic)
- The "no mocks" rule (mock external APIs, never mock the thing under test)
- Makefile test targets (`make test`, `make test-one-off`)
- Mid-project migration path for flat test directories

**How it evolved:**
- Started as a basic six-step TDD checklist: write test, confirm fail, write code, confirm pass, refactor, repeat
- The three test types (RT/OT/UT) were introduced after one-off tests (migration verification, incident reproduction) kept appearing in the regression suite, causing confusion about what `make test` actually validated
- The directory split (`tests/regression/` vs `tests/one_off/`) enforced the separation structurally -- `make test` references only `tests/regression/` and must never reference `tests/one_off/`
- The test ID scheme was originally a global sequential counter across all issues. This was changed to issue-scoped IDs (`RT-{issue}.{n}`) for simplicity -- no central counter file needed, just check the current issue's AC table and increment
- The multi-condition coverage rule was added after an AC stating "rejects invalid, expired, and missing tokens" was marked as passing with a single test for "invalid" only
- The "real-user test" principle was the most significant addition, prompted by a critical production failure: a TTS tool shipped with a deadlocking subprocess (ffmpeg receiving SIGTTIN) because every test called the engine's library function in-process instead of invoking the real installed binary. The test suite never spawned a subprocess, so a bug that only manifested in the subprocess code path was invisible. The rule now requires that before marking any RT as passing, the tester must state in chat what user action the test simulates and what the user would observe -- if the answer references internal APIs, database rows, or source code, the test is wrong
- The anti-pattern "tests that invoke the build system" was added after an RT called `make test` from within `make test`, creating infinite recursion
- User tests (UTs) were formalized as a separate type after attempting to automate tests that inherently required human judgement (visual correctness, audio quality, subjective readability) -- these are documented in the AC table only, with no test file, and only the human can mark them as passing or failing
- The "no mocks" rule was clarified to distinguish acceptable mocking (external HTTP APIs, time/dates) from unacceptable mocking (the class under test, mocking so much the test verifies nothing real)
- Mid-project migration rules were added for projects with pre-existing flat `tests/` directories -- move everything to `tests/regression/`, create `tests/one_off/.gitkeep`, update `make test`, and commit as a standalone chore before touching any issue work
- Test ID retrofit rules: do not retrofit IDs to pre-existing tests, migrate on touch, new tests always get an ID

#### ISSUES.md -- GitHub Issue Standards

**What it covers:**
- Voice and tone (impersonal, third person)
- Well-formed issue structure (problem statement, solution section, AC table)
- Acceptance criteria table format with three columns (ID, AC, Tests)
- Single source of truth rule (exactly one AC table per issue, edited in place)
- AC quality heuristic (single-test coverage is a smell)
- Multi-condition AC coverage requirements
- The AC/Test boundary (ACs describe system states, tests describe how to verify them)
- Litmus test for distinguishing ACs from tests
- Forbidden language in the AC column (action verbs, passive test phrasings, test-structure language)
- Self-audit procedure
- Solution section rules (edit in place, never litter with superseded versions)
- Sub-issue standards
- AC and test ID allocation rules
- Immutability after gate sign-off vs free editing during drafting

**How it evolved:**
- Did not exist in the initial commit -- it was created when acceptance criteria quality became a bottleneck
- The core problem: Claude was writing ACs that were actually test descriptions disguised as requirements. "Call the API with an invalid token and assert a 401 is returned" is a test, not a system state. The document was created to enforce the distinction
- The forbidden word list was compiled from real examples: action verbs (call, assert, send, check, verify, run, execute, invoke), passive test phrasings ("is returned", "results in", "produces"), and test-structure language ("when you", "if you", "after calling")
- The litmus test was added as a quick self-check: "Could this statement be true or false without specifying how it is observed?" If not, it is a test, not an AC
- Wrong/right example pairs were documented to make the distinction concrete:
  - Wrong: "Call the validate function with a malformed date string and assert it raises ValueError"
  - Right: "The validator rejects malformed date strings"
- The single source of truth rule was added after multiple AC tables appeared in different comments on the same issue, creating ambiguity about which ACs were current
- The AC quality heuristic ("an AC with exactly one test is a smell") was added after Claude routinely wrote one test per AC even when the AC implied multiple conditions
- The immutability boundary was clarified: ACs and test IDs become immutable after Gate 1 (SATISFIED), but are freely editable during drafting. This distinction was added after the immutability rule was being applied too rigidly during the drafting phase, slowing down iteration
- Voice and tone rules were added after Claude wrote issues referencing "I", "we", and specific people -- issues must stand on their own and outlive conversations

#### GIT.md -- Git and Source Control

**What it covers:**
- Fundamental rules (never `--no-verify`, always work in a git repository)
- Forbidden flags (`--no-verify`, `--no-hooks`, `--no-pre-commit-hook`)
- No AI attribution in commits
- Commit message standards (conventional commits, imperative mood, present tense)
- Issue-linked commit format (`Implement #N: short description`)
- Prohibition on auto-close keywords (`Fixes`, `Closes`, `Resolves`)
- Pre-commit hook failure protocol (five-step sequence: read, identify, explain, fix, re-run)
- Project hooks chaining to global hooks (with a reusable pattern)
- Hook setup via Makefile (`hooks/` directory, `make init` copies to `.git/hooks/`)
- Pressure response protocol
- Accountability self-check before any git command
- Branch strategy (default: master, feature branches when specified)
- Exception process (document, get approval, record, set expiry)

**How it evolved:**
- Started with the fundamental rule that has never changed: never use `--no-verify`
- The forbidden flags list was expanded after Claude used `--no-hooks` and `--no-pre-commit-hook` as alternatives to `--no-verify` -- same bypass, different flag
- Auto-close keywords (`Fixes #N`, `Closes #N`) were banned after Claude used them in commit messages, bypassing the human review step that the gate system exists to enforce
- The pre-commit hook failure protocol was added after Claude's response to a failing hook was to try bypassing it rather than reading the error output and fixing the underlying issue. The protocol forces a specific sequence: read the error, identify which tool failed, explain the fix, apply it, and re-run
- The pressure response section was added after Claude attempted to skip hooks when asked to commit quickly -- "Pre-commit hooks are failing, I need to fix those first" is the required response regardless of urgency
- The global hook chaining pattern was added after a project-level `core.hooksPath` silently replaced the global hooks, disabling global pre-commit checks. The pattern uses `${0##*/}` to resolve the hook's own name, making it work for any hook type without modification
- The hook setup section documents the `hooks/` directory + `make init` pattern for version-controlling project hooks while preserving global hook inheritance
- The exception process was formalized to prevent "just this once" from becoming permanent practice -- every exception must be documented, approved in writing, recorded in the commit message, and have a follow-up issue to resolve properly
- The accountability self-check (Am I bypassing a safety mechanism? Would this violate CLAUDE.md? Am I choosing convenience over quality?) was added as a pre-flight check before any git command

#### DOCUMENTATION.md -- Documentation Standards

**What it covers:**
- Voice and tone (impersonal, third person)
- Versioning with changelog headers for project docs
- Process rules (read docs before work, update docs after code changes)
- Sanitization step (all doc changes run through `sanitize` for OED-standard spelling)
- Review workflow (write to temp files, sanitize, then `code -d` for VS Code diff)
- Project structure (README.md or README.org, `./docs/` directory layout)
- Required documentation files (VISION, architecture, testing, implementation plan)
- Inconsistency handling (stop for major issues, warn for minor ones)
- Licence defaults (MIT, copyright Tadg Paul)

**How it evolved:**
- Started as a structural guide: where files go, what the README should contain, what `./docs/` should include
- Voice and tone rules were added after Claude wrote documentation referencing "I added", "we should", and specific individuals -- documentation must describe the system impersonally
- The sanitization step was added after inconsistent British/American spelling appeared across documents. All documentation changes now run through `sanitize` (from `tigger04/oed-sanitize`) which normalizes to OED-standard spelling (British English with `-ize` suffixes) and fixes problematic symbols (em dashes, smart quotes). The changes summary is reported in chat
- The review workflow was refined to require sanitization before review, not after -- the human must review the sanitized version, not a pre-sanitize draft. The pattern writes the old and new versions to temp files, sanitizes the new version, then opens a VS Code side-by-side diff with `code -d`
- Version headers (`<!-- Version: 1.2 | Last updated: 2026-01-20 -->`) were added for project docs but exempted for global config files under `~/.claude/`, which are version-controlled via git
- The inconsistency handling rule was added after Claude silently proceeded with work despite contradictions in the documentation that affected the solution design

### The three gates

The gate system evolved through three iterations (see "What Was Tried" above): a loose traffic-light classification, a single APPROVED checkpoint, and the current three-gate model. Each iteration addressed the failure mode of its predecessor.

| Gate | Keyword | What it authorizes |
|------|---------|-------------------|
| Gate 1: Requirements | `SATISFIED n` | Solution design may begin |
| Gate 2: Solution | `PROCEED n` | Test and implementation code may be written |
| Gate 3: Review | `APPROVED n` | Issue may be closed |

Gate keywords must come from the human, in ALL CAPS, with the issue number. Claude may never write a gate keyword itself. The strict keyword format (ALL CAPS, with issue number, in the current conversation turn, from the human) was refined after Claude found ways to self-authorize by writing keywords into its own output or referencing approvals from different issues or conversations.

### Typical flow

```
/draft-issue -> SATISFIED n -> /design-solution -> PROCEED n -> /write-tests -> /implement -> /review -> APPROVED n
```

Skills may be skipped, reordered, or repeated. Audits are optional. The gates are the only hard stops.

### Key principles

- **The real-user test.** Before marking any test as passing, state in chat: what user action does this test simulate, and what would the user observe? If the answer references internal APIs, database rows, or source code -- the test is wrong.
- **Tests before parallel agents.** When parallelising work across agents, all tests must be written and committed first. Agents receive test files as their specification, not carte blanche to write their own.
- **Immutability after sign-off.** ACs, tests, and IDs become immutable once a gate is passed. Draft text before sign-off may be freely edited.
- **Impersonal voice.** Issues and documentation are written in the third person. They describe the system, not the conversation.
- **Known failure modes.** CLAUDE.md explicitly names 11 documented tendencies Claude Code exhibits, from premature implementation to error suppression to testing implementation instead of behaviour. Naming them does not prevent them, but it creates accountability -- when one occurs, both parties can identify it by number.

## Areas for Improvement

- **Skill granularity.** The current skills may still be too coarse for some workflows, or too fine for others. The balance between "invoke what you need" and "don't forget a step" is still being calibrated.
- **Cross-conversation memory.** The framework relies on CLAUDE.md being read at conversation start, but lessons from one conversation do not always carry to the next. The memory system helps but is imperfect.
- **Rule-lawyering.** Claude is adept at satisfying the letter of rules while violating their spirit. The "real-user test" question is the current mitigation -- forcing visible justification rather than adding more prohibitions -- but its effectiveness is still being evaluated.
- **Batch and parallel workflows.** The test-first rule for parallel agents is new and relatively untested at scale.
- **Language coverage.** Coding standards are detailed for shell and Python, thinner for other languages. The `/audit-code` skill compensates by reviewing against language-specific best practice, but the documented standards will continue to grow as new patterns are encountered.
- **Cat and mouse.** Each time a specific shortcut is prohibited, Claude finds the next narrowest shortcut that technically satisfies the rules. The shift from specific prohibitions to general principles (the real-user test, error handling philosophy) is an attempt to close this gap, but it remains the central tension of the framework.

## Licence

MIT. Copyright Tadhg O'Brien.
