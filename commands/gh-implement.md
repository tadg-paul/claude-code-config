
For each github issue number above, perform the following steps:

- Confirm each step you are doing in real time
- Fetch the latest issue details with `gh issue view [n]`, review the issue and all comments
- Review project documentation as this may have changes since you last looked
  - pay special attention to vision, implementation guide, patterns, design principles (if any)
- Review affected files, review the issue implementation specification against the design principles and patterns and if there are any major contradictions, or if the solution is not sound, DO NOT PROCEED and alert me in real-time.
- Update project documentation as appropriate
- Commit changes with message `Fixes #[n]: [short description]` and push branch to remote
- Add a comment to the issue describing implementation details and testing instructions, and commit link
