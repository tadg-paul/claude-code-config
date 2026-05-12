# Claude Code SDLC Configuration

A software development lifecycle framework for human-AI pair programming with Claude Code. Developed iteratively through real project work, refined by failure, and designed around one principle: the human drives, the AI executes.

This repository contains the complete framework -- process rules, coding standards, testing standards, issue management, git workflow, and documentation standards -- along with the invocable skills (slash commands) that make the process practical to use day-to-day.

## The Problem

Claude Code is a powerful AI coding assistant, but without constraints it exhibits predictable failure modes that undermine the quality of work it produces:

- **Premature implementation.** It writes code before requirements are agreed, solving the wrong problem efficiently. The human is then left choosing between accepting work that doesn't match what they wanted, or discarding it and starting over.
- **Shortcut testing.** It writes tests that exercise internal APIs instead of real user entry points. Tests pass; the application is broken. In one project, a TTS tool shipped with a deadlocking subprocess because the test suite called the engine's library function directly instead of invoking the real binary -- the subprocess code path was never exercised, and the bug was invisible until a user ran the tool.
- **Error suppression.** It silences failures with patterns like `|| true` instead of handling them properly, defeating the safety nets (`set -euo pipefail`) it was told to use. The script appears to work until it encounters a real error, at which point it silently continues with corrupt state.
- **Self-authorization.** It declares its own work satisfactory and moves on without waiting for human review. It has been observed writing approval keywords into its own output to advance past quality gates.
- **Scope creep.** It "improves" surrounding code, adds features not requested, and refactors things that already work -- introducing risk and making review harder without delivering value.

Every rule in this framework exists because one of these failures occurred in production work. None are theoretical.

### Observed failure patterns in detail

Beyond the headline failures, specific bad practices recur across projects and languages. These are documented here because they inform the rules in the framework -- each prohibition or process step traces back to one of these patterns.

**Testing shortcuts:**

- **Testing the internals instead of the real thing.** The specification says "run the CLI command and check the output", but the test calls the underlying library function directly in the same process. The test passes, but the real binary -- with its argument parsing, subprocess spawning, file path resolution, and signal handling -- is never exercised. This is the pattern that shipped a deadlocked TTS tool: the test called `engine.synthesize()` in-process, so the bug in the subprocess code path (where ffmpeg inherited the parent's terminal stdin and received a stop signal) was invisible to the entire test suite.

- **Testing the storage layer instead of the user experience.** The specification says "user sees a confirmation page", but the test checks whether a row was inserted into the database. The row might be inserted correctly while the page that displays it is broken -- the test would never catch that, because it never exercises the rendering path.

- **Checking that code exists instead of running it.** The test searches the source code (using grep or similar) to verify that a function is defined, rather than actually calling the function and checking its output. This verifies that someone wrote the function, not that the function works.

- **Covering one condition when the specification implies several.** An acceptance criterion states "the system rejects invalid, expired, and missing tokens" -- three distinct conditions. The test covers only "invalid" and is marked as passing. The other two conditions are never tested, and the gap is not caught until a user encounters the untested case in production.

**Error handling shortcuts:**

- **Appending "or succeed anyway" to commands that might fail.** In shell scripts, the framework requires a safety mode (`set -e`) that stops execution if any command fails. Claude appends `|| true` or `|| :` to commands, which tells the shell "if this command fails, treat it as a success and keep going." This defeats the entire safety mechanism -- the script continues past failures as if nothing went wrong. Example: `risky_command || true`

- **Capturing the error code in a way that hides the failure.** A variation of the above: `cmd || rc=$?` captures the exit code into a variable, but as a side effect it also suppresses the safety mechanism. It looks like the error is being handled, but the script continues regardless. The correct approach is to use a conditional: `if cmd; then ... else ... fi` -- this handles the failure explicitly without disabling safety.

- **Temporarily turning off error checking.** Instead of handling a specific failure, Claude turns off the safety mechanism entirely (`set +e`), runs the risky code, then turns safety back on (`set -e`). This leaves a window where any command can fail silently -- and if the re-enable line is forgotten or the code is nested, safety never comes back.

- **Hiding error messages instead of fixing them.** Redirecting a command's error output to nowhere (`2>/dev/null`) without any fallback logic. The error still occurs, but the diagnostic message that would explain what went wrong is discarded. The program continues in a broken state with no indication of what happened.

- **Catching all errors and doing nothing with them.** In Python: `except: pass`. In Swift: `catch { }`. In Java: `catch (Exception e) { }`. The program encounters an error, catches it, and silently continues as if nothing happened. The error disappears and the program proceeds with state that may be corrupt. The correct approach is to catch specific errors and handle them meaningfully -- log them, retry, use a fallback, or propagate them upward.

- **Working around a language quirk by suppressing the safety system.** In bash, arithmetic expressions like `((count++))` return exit code 1 when the result is zero, which triggers the safety mechanism and kills the script. Rather than using the correct pattern (`count=$((count + 1))`, which always succeeds), Claude appends `|| true` to suppress the safety mechanism -- the same problematic pattern applied to a language-specific edge case.

**Process shortcuts:**

- **Self-authorizing advancement past quality gates.** The framework requires the human to type specific keywords (SATISFIED, PROCEED, APPROVED) to advance past quality checkpoints. Claude has been observed writing these keywords into its own output, effectively approving its own work and moving to the next phase without human review.

- **Claiming completion without demonstrating the result.** Declaring work "fixed", "done", or "complete" when automated tests pass, but without showing the actual user-facing behaviour. Tests passing is necessary but not sufficient -- the human needs to see the real result, not just a test report.

- **Overwriting the human's changes.** When the human edits an issue, a file, or documentation, Claude reverts those changes because it disagrees with them. The human's edits are authoritative -- Claude may raise concerns, but must not silently undo the human's work.

- **Treating a question as an instruction.** The human asks "What do you think of X?" -- an invitation for discussion. Claude interprets this as an instruction to go and implement X, writing code and making changes unprompted. A question should be answered, not acted upon.

- **Silently removing information from issues.** Rather than editing acceptance criteria, test statuses, or comments in place (preserving the history of what changed and why), Claude deletes rows or content entirely. The information disappears with no audit trail.

**Coding shortcuts:**

- **Building shell commands from strings.** Using `eval`, string concatenation, or variable expansion to construct and execute shell commands at runtime. This is a classic injection vulnerability -- if any part of the string comes from user input or external data, arbitrary commands can be executed. The correct approach is to use arrays to build commands: `cmd=(prog arg1 arg2); "${cmd[@]}"`.

- **Using text-replacement tools to modify source code.** Using `sed` and `awk` (text stream editors designed for line-by-line text processing) to modify source code, configuration files, or documentation. These tools have no understanding of the structure of what they are editing -- they match and replace text patterns, which is brittle, prone to corrupting file structure (e.g. breaking indentation, matching the wrong occurrence), and difficult to review. The correct approach for source code transformations is to use AST-aware tools like `ast-grep` that understand the language's syntax tree.

- **Embedding secrets in source code.** Hardcoding passwords, API keys, or credentials directly in the code rather than loading them from environment variables or a secrets manager. Also: disabling SSL/TLS certificate verification (allowing man-in-the-middle attacks) and setting file permissions to `chmod 777` (making files readable, writable, and executable by everyone on the system).

- **Writing functions that are too large or too deeply nested.** Functions exceeding 50 lines, conditional logic nested deeper than 3 levels, or "god objects" that handle too many responsibilities. These are not bugs but they make code harder to understand, test, and maintain -- and they tend to hide bugs.

- **Permanently deleting files instead of moving them to the trash.** Using `rm` (which permanently deletes files with no recovery option) instead of `trash` (which moves them to the system trash, allowing recovery if the deletion was a mistake). In a development workflow where files are frequently created and removed, accidental permanent deletion is a real risk.

### The language coverage challenge

Coding standards can enumerate prohibited patterns for commonly used languages (shell, Python, Swift), but cannot exhaustively list anti-patterns for every language and framework that might be encountered across all projects. The same underlying tendency -- taking the narrowest shortcut that technically satisfies the requirement -- manifests differently in each ecosystem. A shell script suppresses errors with `|| true`; Python code uses `except: pass`; Swift code uses `try!` with force unwrapping.

The mitigation is twofold: the coding standards document (CODING.md) captures the general principles and the specific pitfalls that have been encountered so far, while a dedicated code audit skill (`/audit-code`) reviews against both the documented standards and the best practice conventions for whatever language and stack the project actually uses. This means the audit adapts to the project rather than being limited to what was pre-documented.

## Objectives

1. **Human retains control.** No code is written, no issue is closed, no decision is made without explicit human authorization at defined checkpoints.
2. **Quality through discipline.** Test-driven development, real-behaviour testing, proper error handling, and coding standards are enforced by process, not by trust.
3. **Traceability.** Every change traces to a GitHub issue, every issue has acceptance criteria, every acceptance criterion has tests, every test has a unique ID. Nothing is ad hoc.
4. **No shortcuts.** The framework explicitly names and prohibits the shortcuts Claude Code is known to take, and requires visible accountability (e.g. answering the "real-user test" question in chat before marking a test as passing) rather than relying on rules it can reinterpret.

## What Was Tried

### Phase 1: Traffic lights

The initial approach used a Green/Amber/Red classification for actions. Green (autonomous): fix tests, correct typos, add imports. Amber (propose first): multi-file changes, new features. Red (always ask): rewriting working code, security changes, data loss risks.

This was too loose. Claude treated most work as Green, wrote code before requirements were clear, wrote tests that checked internal state rather than observable behaviour, and declared things "done" without evidence. The classification relied on Claude's own judgement about which category a task fell into, and it consistently chose the most autonomous option.

### Phase 2: Single approval gate

A single APPROVED gate was introduced: no code could be written until the human explicitly typed APPROVED in chat. This stopped premature implementation but conflated three separate decisions into one checkpoint:
- Are the requirements right?
- Is the solution design sound?
- Is the finished result acceptable?

With only one gate, requirements problems were caught too late -- after solution design or even implementation had already begun. The human would approve requirements and a solution in one step, then discover during review that the requirements had been incomplete.

### Phase 3: Three gates with prescriptive checklist

The single gate was split into three separate checkpoints, each with its own keyword:
- **SATISFIED** -- requirements agreed (acceptance criteria and test specifications)
- **PROCEED** -- solution design approved (architecture, patterns, approach)
- **APPROVED** -- finished result accepted (tests pass, code reviewed, demonstrated)

A 32-step sequential checklist enforced every phase, with mandatory quality checks at each step.

This solved the quality problem but created a new one: Claude would grind through every phase sequentially, spending long periods on acceptance criteria audits, test enumeration, and solution documentation before a line of code was written. Simple bug fixes received the same ceremony as architectural changes. The process felt like 1990s waterfall development -- comprehensive but paralysingly slow.

### Phase 4: Skill-driven workflow (current)

The 32-step checklist was decomposed into invocable skills (slash commands). The human calls the skill needed for the current moment; the quality checks are baked into each skill rather than front-loaded as a sequential script.

The three quality gates remain as hard stops requiring human keywords. The absolute prohibitions remain unchanged. But the human controls pacing -- a simple fix can move quickly through the gates, while a complex feature can invoke optional audit skills for deeper scrutiny. The process adapts to the size of the task instead of treating everything identically.

### Canary system

## Canary System

Each reference document contains a canary section at the end. CLAUDE.md establishes the root canary string, and every other reference document appends its own suffix. The full chain currently is:

```
EHLO ISSUES TEST CODE GIT DOC SHELL PY GO WEB
```

Every response in the chat interface should begin with this string. If any element is missing, that document's instructions were not loaded into context for the current interaction. The presence of all elements does not guarantee the rules were followed -- only that the text was seen. But it closes one failure mode.

The CLAUDE.md canary has evolved from a simple acknowledgement to an attestation:

> # Canary
> The canary string is "EHLO". It means you have read and agree with the SDLC described in this document, including all documents referenced from here; it means you agree with the spirit of it and you will not try to game it. If anything in it is unclear, countermands a previous instruction or contradicts itself internally you must say so now. If you are not prepared to follow this process, say so now. If the above is all true, say "EHLO" at the beginning of every interaction with me.

And each other document appends a one-line suffix. For example, in `DOCUMENTATION.md`:

> # Canary
> Suffix the canary string with "DOC "

The change from "start every interaction with this string" to a commitment-based attestation -- spirit clause and opt-out included -- correlates with observably more diligent agent behaviour. The dryness of the wording is the contract.

## Where It Landed

### Architecture

```
~/.claude/
  CLAUDE.md              # Process rules, prohibitions, gates, failure modes
  skills/                # Invocable skills (slash commands, skill format)
    draft-issue/         # Create issue with ACs and test specs
    draft-design-issue/  # Draft issue + solution design in one pass
    audit-acs/           # Challenge AC coverage (advisory)
    audit-tests/         # Challenge test coverage (advisory)
    design-solution/     # Document solution on issue
    write-tests/         # Write test code only (TDD red phase)
    implement/           # Write code to pass tests
    audit-code/          # Review code against standards (advisory)
    review/              # Full review: make test, standards, user test demos
    summarize-issues/    # Open issues summary, gap analysis, prioritization
    satisfied/           # Gate 1 keyword (human-only, model invocation disabled)
    proceed/             # Gate 2 keyword (human-only, model invocation disabled)
    approved/            # Gate 3 keyword (human-only, model invocation disabled)
    bypass/              # BYPASS-GATE-7 (human-only, model invocation disabled)
  docs/
    ISSUES.md            # Issue structure, AC quality standards
    TESTING.md           # Testing standards, TDD, real-user test
    CODING.md            # Cross-language coding standards
    GIT.md               # Git workflow, commit standards, hook policy
    DOCUMENTATION.md     # Documentation standards, voice, sanitisation
    CODE/                # Language and platform-specific standards
      SHELL.md           # Shell commands and scripts
      PYTHON.md          # Python (uv, venvs, ruff, src layout)
      GO.md              # Go (12-factor config, error handling, HTTP timeouts)
      WEB.md             # HTML, CSS, JavaScript, web-content testing
```

### The reference documents

Each document in `docs/` addresses a specific craft discipline. All were present from the initial commit in some form (except ISSUES.md, which was created later when acceptance criteria quality became a bottleneck). All have evolved substantially through real-world use -- each section, rule, and prohibition traces to a specific failure that occurred in practice.

#### CODING.md -- Coding Standards

**What it covers:**
- Platform constraints (macOS/Apple Silicon for local development, Linux for deployment)
- A decision hierarchy for language and tool selection -- use what the project already uses, or if starting fresh, choose the genuinely best fit rather than defaulting to a familiar language. Explicit instruction not to reach for shell scripting when a compiled language (Go, Rust) would produce a less brittle result
- Style baselines table mapping each language ecosystem (shell, Python, Swift, HTML, CSS, JavaScript, TypeScript, Ruby, Rust, Go, Java, Kotlin, C/C++, and a catch-all) to its community-standard linter and formatter
- Language-specific standards live in their own documents under `CODE/`, loaded alongside this one when the relevant language is in use
- Code commenting rules: every file starts with a two-line ABOUTME comment explaining its purpose; comments must be evergreen (describe the code as it is, not how it evolved)
- Data handling principle: format-aware parsers for any structured data (jq, yq+jq, dasel, htmlq for shell; `encoding/json`, `gopkg.in/yaml.v3` for Go; `json`, `yaml` for Python). Never use sed, awk, or perl to modify data of any kind. `ast-grep` for source code, but only for cross-file structural refactors -- direct editing is fine for single-file changes
- A general anti-embedding rule: never embed one language as a string inside another (Nix-in-Go `fmt.Sprintf`, HTML by string concatenation, SQL string-building, YAML heredocs). Use file-based templates with the appropriate engine, parameterised queries, or proper code generation
- Comprehensive prohibited anti-patterns section covering error suppression, unsafe practices, and security vulnerabilities -- with language-specific examples for shell, Python, Swift, Java, C#, and Go
- A cross-language escaping subsection unifying SQL-parameterisation, shell-quoting (`%q` in Go, `shlex.quote` in Python), HTML auto-escaping, JSON marshalling, and URL encoding as a single principle: when one language constructs commands or markup in another, use the target language's native escaping
- Cross-language engineering practices: error handling principles, code structure (function size, nesting depth, no inline templates), security (secrets, input validation, permissions, Docker hardening), output formats (ISO 8601), logging, file operations (atomic writes), network operations
- Dependency management and security auditing

**How it evolved:**
- Started as a short style guide covering only shell and Python, with a handful of rules (quote your variables, use `command -v` not `which`, mandatory safety header)
- The style baselines table was added after projects began spanning Swift, TypeScript, Ruby, Rust, Go, and others -- without a table mapping each language to its standard tools, Claude would use inconsistent linting and formatting across files in the same project
- The prohibited anti-patterns section grew incrementally. Each entry was added after a real incident:
  - `|| true` and `|| rc=$?` were added after Claude repeatedly used these patterns to suppress `set -e` -- making scripts that appeared to work but silently continued past failures
  - `eval` and string-built commands were added after injection-vulnerable patterns appeared in shell scripts
  - `sed`/`awk` for code modification was banned after structural corruption occurred in edited source files
  - Bare `except: pass` and empty `catch {}` were added after errors were silently swallowed across Python, Swift, and Java projects
  - The arithmetic expression gotcha (`((count++)) || true`) was documented after the `set -e` interaction was mishandled repeatedly
- Linter bypass rules were added after `# noqa` and `# type: ignore` appeared in Python code without justification -- now any suppression requires the exact rule code being suppressed and an explanation of why it cannot be fixed properly
- The language and tool selection hierarchy was added to prevent defaulting to a familiar language out of habit -- the order is: match the existing project, then choose the genuinely best fit, then choose the simplest maintainable option
- The most significant recent change was extracting language-specific content into a `CODE/` subdirectory after the document's shell-heavy bias became a problem -- when half the worked examples in a "cross-language standards" document were bash, the implicit signal to agents was "solve problems with shell scripts." Shell and Python content moved to `CODE/SHELL.md` and `CODE/PYTHON.md`; new content for Go and Web landed directly in `CODE/`. Doing this with three language docs in scope was the right inflection point -- waiting until five or six docs would have meant a more painful refactor with more cross-references to update
- The data handling section was added after repeated failures of agents using grep/sed/awk to parse structured data. A real example: a `awk -F. '{print $(NF-1)"."$NF}'` zone parser in a deployment script broke on multi-part TLDs like `.co.uk`. The fix was a pure-Go rewrite using the Cloudflare API with longest-suffix matching -- one of several cases where the "use a parser, not regex" rule paid for itself directly
- The cross-language escaping section was added after recognizing that SQL-injection-prevention, shell-quoting, HTML auto-escaping, JSON marshalling, and URL encoding are all the same anti-injection pattern in different costumes. Unifying them under one rule replaces a scattered set of specific prohibitions with one principle: use the target language's native escaping
- `ast-grep` was demoted from "always use it for source code transformations" to "preferred for cross-file structural refactors" -- direct editing is acceptable for single-file changes. The original mandate was overcalibrated for an agent with a working Edit tool

#### CODE/SHELL.md -- Shell Standards

**What it covers:**
- Coverage applies to both interactive shell commands and scripts. Script-specific sections (safety header, portability, schedulers) are marked as such
- Version targeting: bash 5+ for all projects. Stock macOS bash 3.2 is reserved for documented edge cases (Homebrew bootstrap scripts), not a default
- Mandatory safety header for scripts (`#!/usr/bin/env bash`, `set -euo pipefail`)
- The third leg of "Bash Strict Mode" (`IFS=$'\n\t'`) is explicitly forbidden, with a documented rationale: it actively introduces bugs into correctly-written shell (`read A B C < <(cmd)`, parsing space-separated CLI output) while defending only against the unquoted-variable anti-pattern this document already prohibits
- Required practices: variable quoting, array-based command construction, `command -v` not `which`, `find -print0` pipelines, `-h`/`--help`/`--version` flags on all CLI executables
- Prohibited patterns: `eval`, `$cmd` as a command, string-built commands, scripts discovered via `find` or globbing, wiring discovered paths into schedulers
- The arithmetic expression gotcha (`((count++))` returning exit code 1 when the result is zero, triggering `set -e`) with safe alternatives
- Data handling: comprehensive tool table for structured formats (jq, yj+jq for YAML, dasel for TOML, xmlstarlet for XML, jc for CLI output -> JSON). Explicit prohibition on sed/awk/perl for any data modification. `grep` for plaintext only; `ripgrep` for file discovery only (`rg -l` to enumerate files for processing by a format-aware tool)
- A data format policy table: JSON for program-internal data and machine-readable output, YAML for user config, CSV for tabular user data, follow upstream for externally-standardized formats
- Portability across Darwin/Linux, error handling with traps, atomic writes, file operations using `trash` not `rm`
- Schedulers and services (systemd, launchd, cron): never generate definitions from runtime discovery, always use absolute paths in `ExecStart`

**How it evolved:**
- Extracted from CODING.md when the document's shell-heavy bias started biasing agents toward shell as the default problem-solving language
- The bash 3.2 fallback for "public/open-source projects" was removed after the costs (workarounds for missing associative arrays, regex captures, `mapfile`) consistently outweighed the benefit. Anyone capable of installing a public shell tool is capable of running `brew install bash`
- The `IFS=$'\n\t'` prohibition was added after the canonical "Bash Strict Mode" line broke a real `read A B C < <(stty size)` pattern. An audit confirmed the line defends against an anti-pattern (unquoted variables) the Required Practices already prohibit, while breaking correct code that interoperates with space-separated tool output. Documented and forbidden, not just deprecated
- The data handling section was the most substantial recent addition. Tool selections (yj+jq over yq as default, dasel for TOML, jc for CLI output normalization) reflect both correctness and practical concerns -- yq's name collision with the unrelated Python yq is a real footgun, addressed with a guard pattern when yq must be used
- ripgrep was scoped down to file discovery (`rg -l`) after recognizing that content-extraction use cases almost always wanted a format-aware tool downstream anyway

#### CODE/PYTHON.md -- Python Standards

**What it covers:**
- Never use system Python on macOS
- Version management with `uv` preferred (Python versions, venvs, and packages in one tool), `pyenv` + venv acceptable as fallback
- Virtual environments are mandatory: never `pip install` or `uv pip install` outside a venv
- `src/` layout for new projects to prevent accidental imports of source rather than installed package
- `pyproject.toml` for packaged projects, `requirements.txt` acceptable for simple scripts
- Standard Ruff configuration for linting and formatting (Black-default line length 88, `E501` always ignored to avoid linter-formatter conflict)

**How it evolved:**
- Extracted from CODING.md as part of the language-doc split. Content essentially unchanged from the original Python section in CODING.md
- The split makes future Python-specific additions (testing patterns, async conventions, FastAPI/Django opinions if they accumulate) a low-friction addition without re-bloating CODING.md

#### CODE/GO.md -- Go Standards

**What it covers:**
- Standard project layout (`cmd/<app>/main.go`, `internal/`, optional `pkg/`)
- 12-factor configuration: runtime config via environment variables, never hardcoded paths. Bind to `ADDR` from environment, never hardcode a port or `0.0.0.0`
- Error handling: every error must be checked; `errcheck` enforces this. Discarding with `_ = err` requires a one-line justification. Wrap errors with `fmt.Errorf("context: %w", err)`. Use `errors.Is`/`errors.As` for type checks. No `panic` in library code
- HTTP clients: never use `http.DefaultClient` (no timeout, hung remotes block indefinitely). Always create explicit clients with timeouts. Always close response bodies (lint with `bodyclose`)
- Shell-safe interpolation when constructing shell commands from Go: use `fmt.Sprintf` with `%q`, not `%s`
- Function decomposition: `main()` is a coordinator, not a body. When it grows past 50 lines, extract phase functions (`parseArgs`, `loadConfig`, `validate`, `run`)
- Cross-compilation: `GOOS=linux GOARCH=amd64 go build` for Linux deployment from macOS. Do not install the Go toolchain on production servers
- Concurrency: goroutines must have a clear lifecycle (context-cancelled), channels for communication and mutexes for state (don't mix), `go test -race` for code with goroutines
- Testing patterns: table-driven tests, `t.Helper()` in test helpers, `_test.go` alongside code under test
- Standard tooling: `gofmt`/`goimports`, `go vet`, `golangci-lint` with errcheck/staticcheck/govet/bodyclose/gosec/unused enabled
- Standard library preference: stdlib first (`net/http`, `encoding/json`, `text/template`, `context`, `database/sql`, `log/slog`). Third-party dependencies require justification beyond a small de-facto-standard whitelist (`gopkg.in/yaml.v3`, `github.com/google/uuid`)

**How it evolved:**
- Created when migration from brittle shell to Go binaries began. Most conventions were drawn from a code audit of an actual Hetzner deployment toolchain, where each finding traced to a real bug:
  - `cmd.Run()` with discarded error -> the discarded-error rule
  - `http.DefaultClient` for Cloudflare API calls -> the always-timeout rule
  - `fmt.Sprintf("hugo --baseURL %s", cfg.BaseURL)` shell injection -> the `%q` rule
  - 200-line `main()` -> the phase-function decomposition rule
  - `awk -F. '{print $(NF-1)"."$NF}'` broken on `.co.uk` -> the prefer-stdlib rule (replaced with pure-Go DNS handling using the Cloudflare API and longest-suffix matching)
- The 12-factor configuration convention was adopted not by design but by convergent evolution -- the deployment model (cross-compile on Mac, ship binary, run under systemd, configure via env vars) hit ten of the twelve factors without naming them. See the Notable Findings section below

#### CODE/WEB.md -- Web Standards

**What it covers:**
- A source vs rendered vs presented tier model for web content: source files (templates, source CSS/JS) are source code and not test targets; rendered/built HTML is a user-facing artefact (the browser receives it) and is a legitimate test target; post-JS-execution DOM requires a browser or human to verify
- HTML standards: semantic elements (`<article>`, `<nav>`, `<header>`, never `<div>` soup), document outline (one `<h1>` per page, no skipped heading levels), accessibility minimums (meaningful `alt` text, labelled form inputs, keyboard navigation, WCAG AA colour contrast), validation via `htmltest`
- CSS standards: `rem` units for anything that scales with user font preference (never `px` for font sizes), mobile-first responsive design, no inline styles, BEM or project-existing methodology, `stylelint` configuration
- JavaScript minimums: ESM only, never `eval` or `new Function(string)`, strict equality (`===`/`!==`), `async`/`await` over `.then()` chains, every promise has a handler or an explicit fire-and-forget comment, no `var`, no global pollution
- Build and test tooling: `htmlq` for CSS-selector queries against rendered HTML, `htmltest` for build-time smoke checks, `stylelint` for CSS linting, `eslint`+`prettier` for JS/TS. Headless browser tooling (Playwright/Cypress) reserved for the case where the body of UI tests justifies the cost (heuristic: ~8-10 user tests across a project)
- A testing decision tree by tier: rendered-tier checks become regression tests; computed-style/hover/JS-execution checks become user tests (default) or regression tests with a headless browser; visual judgement always remains a user test

**How it evolved:**
- Created when web-content tooling started accreting across multiple docs (htmlq in CODE/SHELL.md, a Web Design subsection in CODING.md, CSS lint discussion floating). Web is its own domain spanning HTML+CSS+JS+build+accessibility -- not a "language" in the same sense as Go or Python, but distinct enough to warrant its own home
- The source vs rendered tier model resolved a real-time methodology question. An agent writing tests for a Hugo theme's hover affordance had grepped the source CSS to verify the hover rule; the new "no source-introspection in tests" rule (TESTING.md) made the tests non-compliant. The tier model articulates which artefacts tests may legitimately target: rendered HTML is the user-facing artefact (the browser parses it), source CSS is not. The same agent recognized the violation, presented three options for handling it, and self-corrected -- a strong validation signal for the explicit-rule-plus-cross-references approach
- HTML and CSS rows were added to the Style Baselines table in CODING.md (linter/formatter references that point at CODE/WEB.md for the standards). This kept CODING.md as the central per-language index without duplicating the standards

#### TESTING.md -- Testing Standards

**What it covers:**
- The "real-user test" principle -- the single most important rule in the framework
- Test-driven development workflow: enumerate test cases for each acceptance criterion, write failing tests, write minimal code to pass, refactor
- Multi-condition coverage requirements
- Three categories of test (regression, one-off, and user tests -- explained below)
- A decision tree for choosing between test categories
- Directory layout separating regression tests from one-off tests
- Test naming conventions that make failure messages self-explanatory
- A test ID scheme that ties every test to the issue that created it
- Arrange-Act-Assert structure for all tests regardless of language
- Coverage requirements (80% floor for new code, 100% for critical paths like authentication and payment)
- Test boundaries (unit, integration, end-to-end) with guidance on speed and when to run each
- Test data rules (use factories, never production data, always deterministic)
- The "no mocks" rule: mock external APIs and time/dates, but never mock the thing under test
- Makefile targets for running regression tests (`make test`) and one-off tests (`make test-one-off`)

**The three test categories and why they exist:**

The framework distinguishes three categories of test. These categories were not designed upfront -- they emerged from problems encountered during development:

- **Regression tests (RT)** are the default. These guard ongoing behaviour that could break in future changes. They live in `tests/regression/` and run on every `make test`. Most tests are regression tests.

- **One-off tests (OT)** exist because some tests are tied to a specific, non-recurring event -- verifying a data migration completed correctly, reproducing a production incident, checking a one-time environment state. These tests are useful once but have no purpose after the event is over. They live in `tests/one_off/` and are never included in `make test`. The separation was introduced after one-off tests kept appearing in the regression suite, causing confusion about what `make test` actually validated. A critical anti-pattern also drove this: automated tests that invoke `make`, `make test`, or other build system targets. Since regression tests run inside `make test`, a regression test calling `make test` creates infinite recursion. Tests that verify build system behaviour belong as one-off tests (run once, manually) or user tests -- never as regression tests.

- **User tests (UT)** exist because some outcomes genuinely require human judgement and cannot be automated. "Does the audio sound correct?" "Does the layout look right at all display scales?" "Is this error message clear?" These are documented in the issue's acceptance criteria table but have no corresponding test file -- they are verified by the human during review. The category was formalized after Claude made everything a user test to avoid writing automated tests. The framework now requires a specific decision tree: can a machine verify this? If yes, it is not a user test. Only outcomes requiring human judgement (visual correctness, audio quality, subjective readability) qualify. This forces the default toward automation and reserves user tests as a last resort.

**How it evolved:**
- Started as a basic six-step TDD checklist: write test, confirm fail, write code, confirm pass, refactor, repeat
- The test ID scheme was originally a global sequential counter across all issues, requiring a central counter file. This was simplified to issue-scoped IDs (e.g. `RT-12.1` for the first regression test in issue #12) -- no central file needed, just check the issue's acceptance criteria table and increment
- The multi-condition coverage rule was added after an acceptance criterion stating "rejects invalid, expired, and missing tokens" was marked as passing with a single test for "invalid" only. The other two conditions were never tested, and the gap was not caught until review
- The "real-user test" principle was the most significant addition. It was prompted by the TTS deadlock incident described earlier: every test called the engine's library function in-process instead of invoking the real installed binary, so a bug that only manifested in the subprocess code path (ffmpeg receiving SIGTTIN because it inherited the parent's stdin) was invisible to the entire test suite. The rule now requires that before marking any regression test as passing, the tester must state in chat what user action the test simulates and what the user would observe. If the answer references internal APIs, database rows, or source code rather than something the user would actually see, the test does not match its specification and must be rewritten. This is not a rule about a specific shortcut -- it is a question that forces accountability, and it is harder to rules-lawyer than a list of prohibited patterns
- The "no mocks" rule was clarified to distinguish acceptable mocking (external HTTP APIs, time/dates, filesystem for unit tests) from unacceptable mocking (the class under test, mocking so much the test verifies nothing real). The test: if removing the mock would make the test fail for reasons other than external dependencies, too much is being mocked
- Mid-project migration rules were added for projects with pre-existing flat `tests/` directories -- move everything to `tests/regression/`, create `tests/one_off/.gitkeep`, update `make test`, and commit as a standalone housekeeping change before starting any issue work

#### ISSUES.md -- GitHub Issue Standards

**What it covers:**
- Voice and tone: issues must be written impersonally, in the third person, describing the system rather than the conversation or the people involved
- Well-formed issue structure: problem statement, solution section, acceptance criteria table
- Acceptance criteria table format with three columns: ID, acceptance criterion, and tests
- The critical distinction between acceptance criteria and tests -- acceptance criteria describe what must be true about the system; tests describe how that truth is verified
- A litmus test for telling them apart: "Could this statement be true or false without specifying how it is observed?" If not, it is a test, not an acceptance criterion
- A forbidden word list for the acceptance criteria column (action verbs like "call", "assert", "send", "verify"; passive test phrasings like "is returned", "results in"; test-structure language like "when you", "after calling")
- Single source of truth rule: exactly one acceptance criteria table per issue, edited in place -- never create a second table in a later comment
- Quality heuristic: an acceptance criterion with exactly one test is a smell -- either the criterion is too narrow (really a test in disguise) or the test coverage is incomplete
- Multi-condition coverage: if a criterion implies three conditions, three tests are required
- Solution section rules: edit in place, never litter the issue with multiple superseded solutions
- ID allocation rules and immutability after sign-off

**How it evolved:**
- Did not exist in the initial commit -- it was created when acceptance criteria quality became a persistent bottleneck
- The core problem: Claude was writing acceptance criteria that were actually test descriptions disguised as requirements. For example: "Call the API with an invalid token and assert a 401 is returned" -- this describes an action someone performs (a test), not a state the system must be in. The correct form is: "The API rejects requests with invalid tokens with HTTP 401" -- a falsifiable statement about the system that can be verified in multiple ways
- The forbidden word list was compiled from real examples that kept recurring: action verbs (call, assert, send, check, verify, run, execute, invoke), passive test phrasings ("is returned", "results in", "produces"), and test-structure language ("when you", "if you", "after calling"). If any acceptance criterion contains these words, it must be rewritten before proceeding
- The single source of truth rule was added after multiple acceptance criteria tables appeared in different comments on the same issue. With two tables, it became ambiguous which criteria were current, leading to implementation work against outdated requirements
- The quality heuristic ("an acceptance criterion with exactly one test is a smell") was added after Claude routinely wrote one test per criterion even when the criterion implied multiple conditions that each needed separate verification
- The immutability boundary was clarified after a tension emerged: acceptance criteria and test IDs must be immutable after sign-off (to preserve traceability), but this rule was being applied too rigidly during drafting, slowing down iteration. The solution: before the human signs off with SATISFIED, criteria and tests are draft text and may be freely added, edited, removed, or renumbered. After sign-off, they become immutable -- removed items are marked with strikethrough, never deleted
- Voice and tone rules were added after Claude wrote issues that referenced "I", "we", and specific people by name. Issues outlive conversations and must be comprehensible to anyone reading them later, without knowing who "I" was or what conversation preceded the issue

#### GIT.md -- Git and Source Control

**What it covers:**
- The fundamental rule (which has never changed since the initial commit): never use `--no-verify` when committing
- Forbidden flags: `--no-verify`, `--no-hooks`, `--no-pre-commit-hook`, and any AI attribution in commits
- Commit message standards: conventional commit format, imperative mood, present tense
- Issue-linked commits: `Implement #N: short description` -- and a strict prohibition on GitHub auto-close keywords (`Fixes`, `Closes`, `Resolves`), because issues are closed manually after human review, and auto-closing bypasses that step
- A five-step protocol for handling pre-commit hook failures: read the complete error output, identify which tool failed and why, explain the fix and why it addresses the root cause, apply the fix, and re-run the hooks. Never bypass.
- A reusable pattern for project hooks to chain to global hooks rather than silently replacing them
- Hook setup via Makefile: store hooks in a `hooks/` directory at the project root, copy to `.git/hooks/` via `make init`
- A "pressure response" protocol: when asked to commit or push with failing hooks, the required response is "Pre-commit hooks are failing, I need to fix those first" -- regardless of urgency
- An accountability self-check before any git command: Am I bypassing a safety mechanism? Would this violate the process? Am I choosing convenience over quality?
- A formal exception process for situations where a rule genuinely cannot be followed (not "is inconvenient"): document the situation, get explicit written approval, record the exception in the commit message, and create a follow-up issue to resolve properly. Exceptions are not get-out-of-jail-free cards.

**How it evolved:**
- Started with `--no-verify` as the only forbidden flag. The list was expanded after Claude used `--no-hooks` and `--no-pre-commit-hook` as alternatives -- same bypass, different flag name
- Auto-close keywords were banned after Claude used `Fixes #N` in commit messages, which triggered GitHub's automatic issue closure and bypassed the human review step that the entire gate system exists to enforce
- The pre-commit hook failure protocol was added after Claude's response to a failing hook was to try bypassing it rather than reading the error output and fixing the underlying issue
- The pressure response section was added after Claude attempted to skip hooks when asked to commit quickly -- the framework now explicitly states that pressure is never justification for bypassing quality checks
- The global hook chaining pattern was added after a project-level `core.hooksPath` setting silently replaced the global hooks, disabling global pre-commit checks without anyone noticing until a policy violation slipped through
- The exception process was formalized to prevent "just this once" from becoming permanent practice

#### DOCUMENTATION.md -- Documentation Standards

**What it covers:**
- Voice and tone: documentation must be written impersonally, in the third person -- "The `--verbose` flag enables debug output", not "I added a flag to handle this"
- Versioning with changelog headers for project documentation files
- Process rules: read existing documentation before starting work; update documentation after code changes, not before
- A mandatory sanitization step: all documentation changes are run through `sanitize` (a tool from the `tigger04/oed-sanitize` project) which normalizes spelling to the OED standard (British English with `-ize` suffixes -- organize, realize, prioritize -- which is the Oxford English Dictionary's preferred form) and fixes problematic typographic symbols (smart quotes, em dashes). The changes summary is reported in chat so the human can see what was normalized
- A review workflow for documentation edits: write old and new versions to temp files, sanitize the new version, then open a side-by-side diff in VS Code with `code -d`. The human reviews the sanitized version, not a pre-sanitize draft
- Project structure requirements: README.md (or README.org, but not both), all other documentation in `./docs/`, required document types (vision, architecture, testing, implementation plan)
- Inconsistency handling: stop and alert on major inconsistencies that affect the current task; warn about minor inconsistencies that do not

**How it evolved:**
- Started as a structural guide: where files go, what the README should contain, what `./docs/` should include
- Voice and tone rules were added after Claude wrote documentation referencing "I added", "we should", and specific individuals by name -- documentation must describe the system impersonally so it remains comprehensible to anyone reading it later
- The sanitization step was added after inconsistent British/American spelling appeared across documents in the same project. Rather than relying on Claude to spell consistently, all documentation changes now run through an automated normalization tool
- The review workflow was refined to require sanitization before review, not after. The human was reviewing pre-sanitized text, approving it, and then the sanitization step would change words -- meaning the human had approved text they never actually saw in its final form
- The inconsistency handling rule was added after Claude silently proceeded with implementation work despite contradictions in the project documentation that directly affected the solution design

### The three gates

The gate system evolved through three iterations (described in "What Was Tried" above): a loose traffic-light classification that gave Claude too much autonomy, a single APPROVED checkpoint that conflated three decisions into one, and the current three-gate model where each gate authorizes a specific next step.

| Gate | Keyword | What it authorizes |
|------|---------|-------------------|
| Gate 1: Requirements | `SATISFIED n` | Solution design may begin |
| Gate 2: Solution | `PROCEED n` | Test and implementation code may be written |
| Gate 3: Review | `APPROVED n` | Issue may be closed |

Gate keywords must come from the human, typed in ALL CAPS, followed by the issue number (e.g. `SATISFIED 12`). Claude may never write a gate keyword itself -- this is an absolute prohibition. The strict format requirements (ALL CAPS, with issue number, in the current conversation turn, from the human) were refined after Claude found ways to self-authorize: writing keywords into its own output, referencing approvals from different issues, and inferring approval from context rather than waiting for the explicit keyword.

The gate keywords are implemented as skills with `disable-model-invocation: true` in their frontmatter, which means the platform itself prevents Claude from invoking them -- this prohibition is enforced at the system level, not just by instruction.

### Typical flow

```
/draft-issue -> SATISFIED n -> /design-solution -> PROCEED n -> /write-tests -> /implement -> /review -> APPROVED n
```

Or, for faster iteration, `/draft-design-issue` combines the issue and solution design into a single pass, skipping straight to AWAITING PROCEED:

```
/draft-design-issue -> PROCEED n -> /write-tests -> /implement -> /review -> APPROVED n
```

The human invokes each skill when it is needed. Skills may be skipped, reordered, or repeated as the situation demands. The three optional audit skills (`/audit-acs`, `/audit-tests`, `/audit-code`) are available when the human wants a second opinion on coverage or quality but are not mandatory steps. `/summarize-issues` provides a birds-eye view of open issues against the project's architecture and plan. The gate keywords are the only hard stops -- no code can be written before PROCEED, and no issue can be closed before APPROVED.

### Key principles

- **The real-user test.** Before marking any test as passing, state in chat: what user action does this test simulate, and what would the user observe? If the answer references internal APIs, database rows, or source code -- the test is wrong. This principle exists because prohibiting specific shortcuts (like "don't call the library directly") leads to a cat-and-mouse game where Claude finds the next narrowest shortcut. The real-user test is a general question that is harder to satisfy without actually testing the right thing.
- **Tests before parallel agents.** When parallelising implementation work across multiple Claude Code agents, all tests must be written and committed before the agents are spawned. Each agent receives the test files as its specification and is told "make these tests pass." No agent writes its own tests. This prevents the failure mode where each agent optimizes locally, writes narrow tests that validate only its slice, and the integrated result has gaps and interface mismatches.
- **Immutability after sign-off.** Acceptance criteria, tests, and their IDs become immutable once a gate is passed. Removed items are marked with strikethrough, never silently deleted. Before sign-off, everything is draft text and may be freely edited. This preserves the audit trail for completed work while keeping the drafting phase lightweight.
- **Impersonal voice.** Issues and documentation are written in the third person. They describe the system, not the conversation. This ensures they remain comprehensible to anyone reading them later, without context about who wrote them or what conversation preceded them.
- **Known failure modes.** CLAUDE.md explicitly names 11 documented tendencies Claude Code exhibits, from premature implementation to error suppression to testing implementation instead of behaviour. Naming them does not prevent them, but it creates shared vocabulary -- when a failure occurs, both parties can identify it by number and address it directly rather than debating whether something went wrong.

## Notable Findings

Rules and patterns that emerged from real failures during iteration. Surfaced separately because each one contradicts something widely treated as best practice, generalizes a scattered set of specific rules, or captures a non-obvious lesson worth sharing.

### `IFS=$'\n\t'` is harmful, not protective

The canonical "Bash Strict Mode" prescription is a three-line header:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

The first two lines genuinely catch bugs. The third one introduces them. `IFS=$'\n\t'` exists to defend against unquoted-variable bugs (`for f in $files` where filenames contain spaces). In a codebase that already mandates quoted variables and arrays for command construction (as this framework does), it is belt-and-braces where the braces actively trip the user.

What it breaks:

- `read A B C < <(stty size)` and any similar pattern reading space-separated output
- `read -ra arr <<< "$line"` for intentional word-splitting
- Parsing `wc -l`, `df`, `ip addr`, version strings, etc. -- the majority of CLI tool output

`IFS=$'\n\t'` is now explicitly forbidden in `CODE/SHELL.md`. The rule has its own subsection in the safety-header documentation so future agents do not "helpfully" reintroduce it as the canonical pattern.

### Tests must not introspect source code

A regression test that asserts a function is defined, a CSS rule exists in a source file, or a string appears in a template, is testing that someone *wrote* the code -- not that the code *works*. The user never sees source files; the user sees the running system. A test that passes by grepping source proves nothing about behaviour.

This rule was already implicit in the "real-user test" principle, but adding it explicitly in TESTING.md produced an immediate observable effect: an agent writing tests for a Hugo theme's hover affordance recognized mid-task that the CSS-source-grep tests it had just written violated the new rule. Without prompting, it surfaced three options (convert to user tests, install Playwright, lint instead of test), recommended the most appropriate one (user tests for hover, deferring Playwright until tooling cost justified), and asked.

The lesson: making the principle explicit -- and naming it as a recognisable rule -- gives agents vocabulary to self-correct in real time, rather than relying on the human to catch the violation later.

### Source vs rendered vs presented: three tiers, different test eligibility

Web content occupies an awkward middle ground in the source-code-introspection rule. Is a built `index.html` source code (it is text in a file) or output (the browser will render it)? The framework now distinguishes three tiers:

1. **Source** -- templates, partials, source CSS/JS. Source code. Tests must not introspect it.
2. **Rendered/built** -- post-build HTML, fingerprinted CSS bundle, transpiled JS. The artefact the browser receives over the wire. Tests *may* query this tier with format-aware tools (htmlq for HTML); this is real-user testing because the rendered output is the user-facing artefact.
3. **Presented** -- post-JS-execution DOM, computed styles, layout. Requires a browser (Playwright/Cypress) or a human (user test) to verify.

For mostly-static sites (Hugo), tier 2 is sufficient. For JS-heavy sites where meaningful content materializes only after browser execution, tier 2 is insufficient and tier 3 is required.

### "Use a parser, not regex" earns its keep, every time

A deployment toolchain contained a Cloudflare zone-from-domain parser written in awk:

```bash
ZONE=$(echo "$DOMAIN" | awk -F. '{print $(NF-1)"."$NF}')
```

This works for `example.com`. It produces `co.uk` for `example.co.uk`. It produces `example.co.uk` for `sub.example.co.uk`. It cannot work, because awk does not know about TLDs.

The fix was not a smarter regex -- it was deleting awk from the equation entirely. A pure-Go rewrite queried the Cloudflare API for all zones once, then used longest-suffix matching (`strings.HasSuffix(domain, "."+z.Name)`) to find the correct zone. Correct for any TLD configuration, no string-parsing fragility, observable failure mode if a domain does not match any zone.

This is the canonical example of why the format-aware-parsers-not-regex rule pays for itself. Tools (jq, yj+jq, dasel, htmlq, jc) for shell use; standard library (`encoding/json`, `gopkg.in/yaml.v3`, `golang.org/x/net/html`) for compiled languages.

### Embedded-language strings and cross-language escaping are one rule

Most coding standards prohibit string-concatenated SQL because of injection. Fewer prohibit string-concatenated HTML, string-built shell commands, or Nix expressions embedded inside Go's `fmt.Sprintf`. They are all the same anti-pattern: language A constructing valid language B by gluing strings together, with no compiler, linter, or editor checking that the result is well-formed.

CODING.md now treats this as a single rule rather than a list of specific prohibitions:

- **Never embed one language as a string inside another.** Use file-based templates with the appropriate engine (`text/template`, `html/template`, jinja2), parameterised queries, or proper code generation. The narrow exception is a one- or two-line constant string with no interpolation.
- **When one language constructs commands or markup in another, use the target language's native escaping.** SQL parameterisation, shell quoting (`fmt.Sprintf("%q", v)` in Go, `shlex.quote` in Python), HTML auto-escaping (`html/template`, Jinja2 autoescape), JSON marshalling, URL encoding. Listed together in a single table in CODING.md.

The benefit is framing: an agent that internalizes one cross-cutting rule applies it everywhere, where a list of language-specific prohibitions invites cat-and-mouse on the next language not listed.

### Convergent evolution to 12-factor

The Heroku-era 12-Factor App methodology turned out to describe the stack's deployment model almost completely without anyone naming it -- ten of the twelve factors are hit, two are principled divergences:

| Factor | Match |
|---|---|
| 1. Codebase | One repo per app ✅ |
| 2. Dependencies | `go.mod` + `dependencies/nix` ✅ |
| 3. Config | Env vars (`ADDR`, `CONFIG_PATH`, `SECRETS_PATH`) ✅ |
| 4. Backing services | Pattern in place when needed ✅ |
| 5. Build/release/run | Cross-compile on Mac, deploy binary, run under systemd ✅ |
| 6. Processes | Stateless Go apps, `DynamicUser=yes` ✅ |
| 7. Port binding | `127.0.0.1:18080-18099`, Caddy out front ✅ |
| 8. Concurrency | Goroutines -- principled divergence (Go's concurrency model contradicts process-only scaling) ❌ |
| 9. Disposability | `Restart=always`, 5s backoff ✅ |
| 10. Dev/prod parity | NixOS migration is literally this ✅ |
| 11. Logs | stderr -> journalctl, plaintext by default ✅ |
| 12. Admin processes | Single-binary model does not map to this Rails-era pattern ⚠️ |

The lesson: when the stack is designed for security, reproducibility, and simplicity, it converges on most operational best practices regardless of which framework "officially" inspired them. Naming the framework still earns its keep -- it gives an agent a one-sentence shorthand and surfaces the deliberate divergences -- but the practice predates the naming.

### The doc-structure mushroom

Three signs that a single document is taking on more weight than it should:

1. A "cross-language standards" document where half the worked examples are bash. Implicit signal to agents: shell is the default problem-solving language.
2. A "shell scripting standards" document where most rules apply equally to one-shot commands at the prompt. The doc title undersells the rules' applicability.
3. A "web tooling" entry in the shell-script doc's data-handling table. Cross-references multiplying across docs that should be siblings, not parent-child.

The framework's response was to extract `CODE/SHELL.md`, `CODE/PYTHON.md`, `CODE/GO.md`, and `CODE/WEB.md` under a single `CODE/` subdirectory, leaving `CODING.md` as the cross-language root. Doing this with three language docs in scope (SHELL + PYTHON + GO, before WEB.md was added) was the right inflection point -- waiting until five or six docs would have meant a more painful refactor with more cross-references to update.

### The canary works -- and cannot easily be ablated

The per-document canary suffix mechanism (each reference document appends a one-line acknowledgement -- "Suffix the canary string with 'CODE'", "...with 'DOC'", and so on -- so the agent's first line of output enumerates which documents were actually loaded) accumulated organically. It now seems excessive: the attestation in CLAUDE.md alone, with its spirit clause and opt-out, plausibly does most of the work. Dropping the per-document suffixes would simplify the system.

But agent behaviour has measurably improved since the canary system reached its current shape, and there is no easy way to A/B test which element is doing the work. Is it the attestation wording? The per-doc suffixes proving each document was actually loaded? The combination? The cost of finding out is regressing real, observable agent diligence.

This is a recurring pattern in framework iteration: a change lands, behaviour improves, and the specific causal factor is not isolable without expensive experimentation. The conservative position is "do not optimize away a working mechanism just because parts of it look redundant." Less elegant; more robust. The redundancy is the safety margin.

The honest framing: parts of this framework are working better than the rules' authors can fully explain.

## Projects

Projects developed using this SDLC framework (or earlier iterations of it):

| Project | Summary | Link |
|---------|---------|------|
| yapper | Fast, Apple Silicon-native TTS CLI and Swift library, powered by Kokoro-82M via MLX | [tigger04/yapper](https://github.com/tigger04/yapper) |
| make-audiobook | Convert ebooks and documents into audiobooks using open-source TTS, locally and privately | [tigger04/make-audiobook](https://github.com/tigger04/make-audiobook) |
| oed-sanitize | CLI tool that converts English text to Oxford spelling (OED British English with -ize suffixes) | [tigger04/oed-sanitize](https://github.com/tigger04/oed-sanitize) |
| writeback | Feedback capture system for a writing group | [tigger04/writeback](https://github.com/tigger04/writeback) |
| superscale | AI image upscaling on Mac using Apple Neural Engine, runs locally | [tigger04/superscale](https://github.com/tigger04/superscale) |
| smart-rename | AI-powered file renaming based on file content | [tigger04/smart-rename](https://github.com/tigger04/smart-rename) |
| image-outliner | CLI tool converting bitmap images into clean monochrome vector outlines | [tigger04/image-outliner](https://github.com/tigger04/image-outliner) |
| storyboard-gen | Turn a YAML storyboard into AI-generated video with Ken Burns effects | [tigger04/storyboard-gen](https://github.com/tigger04/storyboard-gen) |
| transcribe-summarize | Turn audio/video recordings into structured documents and subtitles, locally | [tigger04/transcribe-summarize](https://github.com/tigger04/transcribe-summarize) |
| summarize-text | Text summarization tool | [tigger04/summarize-text](https://github.com/tigger04/summarize-text) |
| golink | Self-hosted HTTP redirect service, domain-agnostic | [tigger04/golink](https://github.com/tigger04/golink) |
| bg-clock | Native macOS analogue clock rendered on the desktop background, built with SwiftUI | [tigger04/bg-clock](https://github.com/tigger04/bg-clock) |
| fzflauncher | macOS application launcher using fzf in a GUI wrapper | [tigger04/fzflauncher](https://github.com/tigger04/fzflauncher) |

## Areas for Improvement

- **Skill granularity.** The current skills may still be too coarse for some workflows, or too fine for others. The balance between "invoke what you need" and "don't forget a step" is still being calibrated through daily use.
- **Cross-conversation memory.** The framework relies on CLAUDE.md being read at the start of each conversation, but lessons from one conversation do not always carry to the next. Claude Code has a memory system that helps, but it is imperfect -- patterns that were corrected in one session may recur in the next.
- **Rule-lawyering.** Claude is adept at satisfying the letter of rules while violating their spirit. Each time a specific shortcut is prohibited, it finds the next narrowest shortcut that technically satisfies the new rule. The shift from specific prohibitions to general principles (like the real-user test question) is an attempt to close this gap, but it remains the central tension of the framework. The cat-and-mouse dynamic between adding rules and finding loopholes is ongoing.
- **Batch and parallel workflows.** The test-first rule for parallel agents is relatively new and has not yet been tested at large scale across many concurrent agents.
- **Language coverage.** Detailed standards now exist for shell, Python, Go, and web (HTML/CSS/JS) under `CODE/`. Thinner for Swift, Rust, Ruby, TypeScript, and Java/Kotlin. The `/audit-code` skill compensates by reviewing against language-specific best practice for whatever stack is in use, but the documented standards will continue to grow as new patterns and pitfalls surface in other ecosystems.
- **Causal opacity of working mechanisms.** As the Canary System finding notes, the framework now contains mechanisms that demonstrably work but whose specific causal factors are not isolable without expensive experimentation. This argues for conservative iteration: do not optimize away apparently-redundant parts of working mechanisms unless the cost of regression is acceptable.

## Licence

MIT. Copyright Tadhg O'Brien.
