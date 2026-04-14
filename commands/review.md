Perform a full review of the current issue's implementation.

1. Run `make test` and paste the output (includes lint).
2. Verify: zero errors, no new warnings. These are hard blocks -- no exceptions.
3. List each coding standard section you checked against, by name and section number.
4. For each UT: launch the application/tool, show Tadg what's on screen, and ask "Does this pass UT-{issue}.{n}?" as a yes/no question. Never give Tadg instructions to run something himself.
5. Update the AC table **in place**. Update automated test statuses. Leave UTs as pending until Tadg answers.
6. Update project documentation as appropriate.
7. Commit with message `Implement #[n]: [short description]` and push.
8. Add a comment to the issue: implementation details, testing instructions, commit link.

**End with:** `READY FOR REVIEW - issue #NNN` and the issue link. **STOP.**
