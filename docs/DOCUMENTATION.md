# DOCUMENTATION STANDARDS

## Versioning

Internal standards documents (like these guidelines) should include a version header:

```markdown
<!-- Version: 1.2 | Last updated: 2026-01-20 -->
```

When making significant changes, increment the version and note what changed in a brief changelog at the end of the document.

## Process

- Before devising any solution, make sure that you have read and digested the documentation for this project which may have changed since the last time you looked.
- When changing any code, update the relevant documentation after (and not before).

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
