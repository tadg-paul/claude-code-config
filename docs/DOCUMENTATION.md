# DOCUMENTATION STANDARDS

## Voice and tone

Documentation must be written impersonally, in the third person. Describe the system, the behaviour, and the rationale -- not who did what or who will decide. Documentation outlives conversations and must stand on its own.

**Wrong:**
- "I added a new flag to handle this case."
- "We should run the linter before committing."
- "Tadg decides which voice to use."

**Right:**
- "The `--verbose` flag enables debug output."
- "The linter must pass before a commit is accepted."
- "Voice selection is configured via the `--voice` flag."

## Versioning

Project documentation files (under `./docs/` in a project) should include a version header:

```markdown
<!-- Version: 1.2 | Last updated: 2026-01-20 -->
```

When making significant changes, increment the version and note what changed in a brief changelog at the end of the document.

Global config files in `~/.claude/` are version-controlled via git and do not require version headers.

## Process

- Before devising any solution, make sure that you have read and digested the documentation for this project which may have changed since the last time you looked.
- When changing any code, update the relevant documentation after (and not before).
- After writing or editing any documentation file, run it through `sanitize` (from `tigger04/oed-sanitize`, installed at `~/.local/bin/sanitize`) to normalize spelling (OED standard) and fix problematic symbols. Sanitized content comes on stdout, a changes summary on stderr. Report the stderr summary in chat.
  ```bash
  sanitize -q < path/to/file.md > /tmp/sanitized.md
  mv /tmp/sanitized.md path/to/file.md
  ```

## Reviewing doc changes

When proposing documentation edits - whether for approval or as evidence of completion - write the before and after versions to temp files, run `sanitize` on the new version, then open a VS Code diff. I must review the sanitized version, not a pre-sanitize draft.

```bash
cp path/to/file.md /tmp/file_old.md
cp path/to/file.md /tmp/file_new.md
# edit /tmp/file_new.md
sanitize -q < /tmp/file_new.md > /tmp/file_sanitized.md
mv /tmp/file_sanitized.md /tmp/file_new.md
code -d /tmp/file_old.md /tmp/file_new.md
```

Apply to the real file only after approval or confirmation.

## Structure
- Each project should have README.md or README.org (but not both)
- Documentation should be in .md or .org format
- All other documentation should reside in ./docs/
- README should contain:
  - A high-level overview of the project
  - A Quickstart section
  - Document any project dependencies
  - A list of the important files in the project and their purpose
  - A brief overview of each doc file and a jumping off point to it
- ./docs/ should contain
  - VISION: A document outlining the overall vision and goals of the project.
  - architecture: A document describing the high-level architecture of the project.
  - testing: A document detailing the testing strategy and procedures.
  - implementation_plan (if relevant)
  - a separate doc for each significant feature or area

## Problems
- Stop and alert me if there are any major inconsistencies in the documentation that impact the task at hand
- Warn me if there are any other inconsistencies in the documentation that do not impact the task at hand

## License

License is MIT with Copyright Taḋg Paul unless otherwise specified

# Canary
Suffix the canary string with "DOC "
