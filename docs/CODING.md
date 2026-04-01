# Coding Standards

## Platform

- **Local development:** macOS/Apple Silicon — this is a constraint of the hardware in use, not a technology preference
- **Deployment:** Linux servers (remote)
- **Approach:** CLI-first, scriptable, FOSS preferred
- **Required tools:** GNU coreutils, ast-grep (`sg`), ShellCheck, shfmt — plus the linter/formatter appropriate to the project language (see Style Baselines)

## Reference Standards

These external standards inform our practices:

- **Shell:** [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html), [Bash Pitfalls](https://mywiki.wooledge.org/BashPitfalls), [ShellCheck Wiki](https://github.com/koalaman/shellcheck/wiki)
- **Python:** [PEP 8](https://peps.python.org/pep-0008/), [PEP 20](https://peps.python.org/pep-0020/) (Zen of Python), [CERT Python](https://wiki.sei.cmu.edu/confluence/display/python)
- **Swift:** [Swift API Design Guidelines](https://www.swift.org/documentation/api-design-guidelines/)
- **JavaScript/TypeScript:** [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript), [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/)
- **Ruby:** [Ruby Style Guide](https://rubystyle.guide/)
- **Rust:** [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/), [Rust Book](https://doc.rust-lang.org/book/)
- **Go:** [Effective Go](https://go.dev/doc/effective_go), [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- **Java/Kotlin:** [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html), [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html)
- **C/C++:** [Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)
- **Security:** [OWASP Top 10](https://owasp.org/www-project-top-ten/), [CERT Secure Coding](https://wiki.sei.cmu.edu/confluence/display/seccode)

## Language and Tool Selection

Use the best tool for the job. Do not bias towards any particular language, framework, or technology out of habit or personal preference.

Decision hierarchy:
1. **Existing project language/framework** — follow what is already in use unless there is a clear, documented reason not to.
2. **Genuinely best fit** — if starting fresh or extending significantly, choose the technology that best fits the problem, deployment context, and maintainability requirements.
3. **Simplest maintainable solution** — prefer the option that is easiest to read, test, and hand over.

Do not introduce new languages, frameworks, or dependencies for marginal convenience. Do not default to a language because it is familiar — justify the choice on merit.

## Style Baselines

These are the languages most commonly used across our projects. For any language not listed, apply that language's dominant community standard and the general principles in this document.

| Language | Style Standard | Linter | Formatter |
|----------|---------------|--------|-----------|
| Shell/Bash | Google Shell Style Guide + safety rules below | ShellCheck | shfmt |
| Python | PEP 8, PEP 20 | Ruff | Ruff |
| Swift | Swift API Design Guidelines | SwiftLint | swift-format |
| JavaScript | Airbnb or project-existing standard | ESLint | Prettier |
| TypeScript | Airbnb or project-existing standard | ESLint + typescript-eslint | Prettier |
| Ruby | Ruby Style Guide | RuboCop | RuboCop |
| Rust | Rust API Guidelines | Clippy | rustfmt |
| Go | Effective Go | golangci-lint | gofmt |
| Java | Google Java Style Guide | Checkstyle / SpotBugs | google-java-format |
| Kotlin | Kotlin Coding Conventions | ktlint | ktlint |
| C/C++ | Google C++ Style Guide | clang-tidy | clang-format |
| Any other | Dominant community standard for that language | Community standard linter | Community standard formatter |

All linting must pass for changed files regardless of language.

## Shell

### Version Targeting

- **Private projects:** bash 5+ (associative arrays, regex matching, etc.) — assume Homebrew locally, standard on Linux
- **Public/open-source:** bash 3.2 (macOS default compatibility)

### Mandatory Safety Header

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'
```

Omit only with documented reason.

### Required Practices

- Quote variables: `"$var"`
- Terminate options: `trash -- "$file"`
- Use arrays for command construction, not strings
- Use `command -v`, not `which`
- Never parse `ls`
- `find` pipelines: `-print0` with `read -r -d ''` or `xargs -0`
- Executables provide `-h`/`--help`,`--version` and `--dry-run` where appropriate
- If releasing on homebrew, always `make release` then install locally via `brew install` or `upgrade` - never try and live patch an existing package install from brew or any other package manager.

### Prohibited Shell Patterns

Never execute code from strings or discovery:

- `$cmd` / `"$cmd"` / `${cmd}` as commands
- `eval` (any use)
- String concatenation to build and run commands
- Running scripts found via `find` or globbing
- Wiring discovered paths into schedulers/services

### Safe Alternatives

- Functions or `case` dispatch for behaviour selection
- Bash array for dynamic commands: `cmd=(git status --porcelain); "${cmd[@]}"`
- Script selection: allowlist by name, validate against `^[A-Za-z0-9._-]+$`, verify exists/executable/permissions

### Arithmetic Expression Gotcha

The `(( ))` construct returns exit code 1 when the expression evaluates to 0, triggering `set -e`. The `let` builtin has the same behaviour. Never suppress this with `|| true`.

```bash
# BAD: suppresses the exit code problem
((count++)) || true
((count++)) || :
let count++ || true

# GOOD: arithmetic expansion in assignment (always succeeds)
count=$((count + 1))

# GOOD: null command consumes the expression (always succeeds)
: $((count++))

# GOOD: declare as integer, then use +=
declare -i count=0
count+=1
```

## Python

**Never use system Python on macOS.**

### Environment

Target Python 3.12.x (or current stable). Respect `.python-version`. Assume pyenv installed.

```bash
[ -d ".venv" ] && source .venv/bin/activate || {
    pyenv local 3.12.x
    python3 -m venv .venv
    source .venv/bin/activate
}
```

- Never `pip install` outside a venv
- Update `requirements.txt` when adding dependencies

## Code Commenting

- **Comments are sacred.** Never remove unless provably false. Every code file starts with a 2-line ABOUTME comment explaining its purpose.
  - The exception to this is "dead code" i.e. commented-out code
- **Evergreen comments.** Describe code as it is, not how it evolved. No temporal context.
- **Evergreen naming.** Never name things 'improved', 'new', 'enhanced', etc.

## Code Search and Modification

**Use ast-grep (`sg`) for source code transformations** — refactoring, renaming, structural changes. AST-aware queries enable safe, language-aware modifications.

**grep/ripgrep are acceptable for searching** any file type — log files, config files, plain text, source code.

**Never use sed/awk to modify files** — source code, documentation, configuration, or otherwise. Use `sg` for source code transformations, and appropriate tools (or direct editing) for other file types. sed/awk are brittle, prone to corrupting structure, and difficult to review.

---

## Prohibited Anti-Patterns

"Errors should never pass silently." — PEP 20

### Error Suppression — Forbidden Patterns (All Languages)

Never hide errors instead of handling them. The following pseudocode patterns are **strictly forbidden** regardless of language:

```
# FORBIDDEN: suppress all errors silently
risky_operation() || ignore

# FORBIDDEN: disable error checking temporarily
disable_error_checking()
risky_operation()
enable_error_checking()

# FORBIDDEN: discard error output with no fallback
risky_operation() 2> /dev/null

# FORBIDDEN: convert failure to success
risky_operation() || exit_success

# FORBIDDEN: catch everything and do nothing
try:
    risky_operation()
catch all:
    pass
```

### Error Suppression — Correct Alternatives

```
# GOOD: check precondition before acting
if precondition_met:
    perform_operation()

# GOOD: when failure is expected, log and continue
if not try_operation():
    log("Expected condition: operation did not apply")

# GOOD: capture failure context for diagnostics
result, err = try_operation()
if err:
    log_warning("Operation failed: " + err)
    handle_gracefully()

# GOOD: catch specific errors with meaningful recovery
try:
    config = load_config(path)
catch FileNotFoundError:
    config = create_default_config(path)
```

### Language-Specific Pitfalls

These are real-language examples where the forbidden patterns commonly appear:

| Pattern | Language | Why It's Banned |
|---------|----------|-----------------|
| `\|\| true` / `\|\| :` | Shell | Silences failures; defeats `set -e` |
| `set +e` (temporary) | Shell | Disables safety; handle specific commands instead |
| `2>/dev/null` (unguarded) | Shell | Hides diagnostics; must have fallback logic |
| `cmd \|\| exit 0` | Shell | Converts failure to success; masks real errors |
| `((expr)) \|\| true` | Shell | Hides arithmetic exit code; use assignment instead |
| `try: ... except: pass` | Python | Swallows all errors silently |
| `except Exception:` (bare) | Python | Too broad; catch specific exceptions |
| `# type: ignore` (bare) | Python | Must specify error code and justification |
| `# noqa` (bare) | Python | Must specify rule and justification |
| `catch { }` (empty) | Swift | Swallows errors; handle or propagate |
| `try!` / `force unwrap` | Swift | Use `guard`, `if let`, or `try?` with handling |
| `catch (Exception e) {}` | Java/C# | Empty catch blocks hide bugs |
| `@SuppressWarnings("all")` | Java | Suppress only specific, documented warnings |

### Linter/Check Bypass Rules

When suppressing a linter warning is genuinely necessary:

```python
# GOOD: specific rule, clear justification
# noqa: E501 - URL cannot be split across lines
long_url = "https://example.com/very/long/path/that/exceeds/line/limit"

# GOOD: type ignore with reason
result = external_lib.untyped_function()  # type: ignore[no-untyped-call] - upstream lacks stubs
```

```bash
# GOOD: shellcheck directive with justification
# shellcheck disable=SC2086 # Intentional word splitting for flags
$command $flags
```

**Requirements for any linter suppression:**
1. Specify the exact rule/code being suppressed
2. Explain why suppression is necessary (not just convenient)
3. Confirm the underlying issue cannot be fixed properly

### Other Prohibited Shortcuts

| Pattern | Why It's Banned | Do Instead |
|---------|-----------------|------------|
| `sleep N` for synchronisation | Race condition; service may not be ready | Poll with timeout, use readiness checks |
| Retry without backoff | Can overwhelm failing service | Exponential backoff with jitter |
| Retry without limit | Infinite loops on persistent failures | Max attempts with circuit breaker |
| Global variables for state | Hidden dependencies, testing nightmare | Pass parameters explicitly |
| Monkey-patching in production | Fragile, version-sensitive | Dependency injection, proper abstraction |
| Reflection to bypass access | Breaks encapsulation, security risk | Fix the API or use public interface |
| String concatenation for SQL | Injection vulnerability | Parameterised queries always |
| String building shell commands | Injection vulnerability | Use arrays: `cmd=(prog arg1 arg2); "${cmd[@]}"` |
| Disabling SSL verification | MITM vulnerability | Fix certificates properly |
| `chmod 777` | Security disaster | Use minimum required permissions |
| Hardcoded credentials | Security breach waiting to happen | Environment variables, secrets manager |

---

## Script Standards

### Portability

- Cross-platform Darwin/Linux where feasible
- `#!/usr/bin/env` shebangs
- Handle macOS paths (`~/Library`, etc.)
- Consider ARM64 quirks
- Prefer Unix built-ins (exceptions: coreutils, ast-grep)

### Error Handling

Core principle: **fail fast, fail loud, fail safe.**

- Exit codes: 0=success, 1=error, 2=misuse
- Errors to stderr, output to stdout
- Validate inputs early; reject bad data at the boundary
- Never convert errors to success silently
- Log sufficient context to diagnose failures
- Clean up resources on failure (use traps in shell)

```bash
# Shell cleanup pattern
cleanup() {
    # remove temp files, close connections, etc.
}
trap cleanup EXIT
```

### Web design
- Use responsive design unless there is a very good reason not to
- Responsiveness should be defined in terms of rems never px, as this affects browser zoom/resizing etc.

### Structure

- One function per task, <50 lines
- Separate configuration from logic
- No inline templates. File content intended for output (scaffolding, generated configs, boilerplate) belongs in package resource files, not string constants
- Prefer pure functions
- Maximum 3 levels of nesting; refactor deeper logic into functions

### Security

#### Secrets Management

Never hardcode credentials, API keys, or tokens. Use:

- **Local development:** Environment variables via `.env` files (must be in `.gitignore`)
- **CI/CD:** Platform secrets (GitHub Actions secrets, etc.)
- **Production:** Secrets manager (1Password CLI, AWS Secrets Manager, etc.)

Pre-commit hook recommended: `detect-secrets` or `gitleaks` to catch accidental commits.

#### Input Validation

Validate all external input at system boundaries:

- **Prefer allowlists** over denylists — explicitly permit known-good values
- **Set maximum lengths** — unbounded input enables DoS
- **Validate encoding** — reject or sanitise non-UTF-8 where unexpected
- **Prevent path traversal** — reject `../` in user-supplied filenames
- **Parameterised queries always** — never concatenate SQL

#### Permissions

- Minimum required permissions for files and processes
- Never `chmod 777` — use the least permissive mode that works
- Run services as non-root where possible

#### Docker

When using containers:

- Run as non-root user (`USER` directive)
- Never use `--privileged` without explicit justification
- Pin base image versions (not `:latest`)
- Scan images for vulnerabilities (`docker scout`, `trivy`)

### Documentation

- Usage examples in headers/docstrings
- Document assumptions and limitations
- Inline comments for non-obvious edge cases only

### Output

- ISO 8601 for dates/times
- Clear, parseable formats

### Logging

- **Levels:** DEBUG (development detail), INFO (normal operation), WARN (recoverable issues), ERROR (failures requiring attention)
- **Structured logging** for production — JSON format enables parsing
- **Never log:** credentials, tokens, PII, full credit card numbers
- **Always log:** request IDs/correlation IDs for tracing, operation context, error details
- **Log to stderr** for CLI tools; use proper logging framework for services

## File Operations

- Use `trash` instead of `rm` to allow recovery
- **Exception:** temporary files and directories created for the sole purpose of a single operation may be deleted directly in cleanup/teardown logic rather than trashed.
- Atomic writes: write to temp, then `mv`
- Use `flock` when multiple processes may write

## Network Operations

- Set explicit timeouts (connect and read separately)
- Retry with exponential backoff for transient failures
- Maximum retry attempts with circuit breaker pattern
- Log request/response metadata at debug level

## Schedulers and Services

For systemd, launchd, cron:

- Never generate definitions from runtime discovery (unless job-dedicated directory with validated scripts)
- Use build/install-time path resolution
- `ExecStart=` must use absolute, fixed paths
- Avoid `sh -c`/`bash -c` in units; pass arguments directly
- Logs must go somewhere predictable; failures must be visible

## Dependencies

- Prefer standard library
- Document required versions
- Avoid deprecated APIs

### Dependency Security

- **Audit regularly:** `npm audit`, `pip-audit`, `cargo audit`, `bundler-audit`
- **Pin versions** in production; use ranges only in libraries
- **Review new dependencies** before adding — check maintenance status, known vulnerabilities
- **Update promptly** when security patches are released

## Model References

When citing LLM models (OpenAI, Anthropic), verify via search if uncertain — knowledge cutoff may cause errors about recent releases.

## System Configuration

Never modify system config (`.bashrc`, etc.) without explicit permission.

## Linting

See the Style Baselines table above for the linter and formatter for each language. All linting must pass for changed files. If a project has a pre-existing linter configuration, follow it — do not override it with personal preference.

---

## Code Review Red Flags

Stop and investigate if you see any of these:

### Immediate Blockers

- Any pattern from the Prohibited Anti-Patterns table
- Credentials, API keys, or secrets in code
- SQL or shell commands built from strings
- SSL/TLS verification disabled
- `chmod 777` or overly permissive settings

### Requires Justification

- Any linter suppression comment
- Commented-out code (why not deleted?)
- TODO/FIXME without issue reference
- "Temporary" fixes without expiry ticket
- Functions exceeding 50 lines
- Nesting deeper than 3 levels
- Tests with no assertions or asserting `True`
- Magic numbers without named constants
- Bare `except:` or empty `catch` blocks

### Smells Worth Discussing

- Multiple return statements in complex functions
- Boolean parameters (consider separate functions or enums)
- God objects or functions doing too many things
- Circular dependencies between modules
- Duplicated code across files (DRY violation)

# Canary
Suffix the canary string with "CODE "
