---
description: Close a discovery session non-destructively. Either promote the sketches into a real issue, or rule the direction out. Sketch commits remain in history either way. See ISSUES.md §"Discovery issues".
---

End the current discovery session. Read and follow @~/.claude/docs/ISSUES.md §"Discovery issues".

**This skill is user-invoked only.** Per CLAUDE.md §1, you may never invoke `/end-discovery` on your own initiative. The user decides when discovery ends.

The user must indicate one of two outcomes:

Discovery is non-destructive: `wip(discovery):` commits remain in history regardless of outcome. They are the record of what was tried. The only state change at end of discovery is closing the issue.

## Path A: Promote

The discovery produced enough clarity to draft a real issue.

1. Summarize what was learned during the session in one short paragraph. Cite specific sketches/commits by hash.
2. Close the discovery issue: `gh issue close N` with a closing comment containing the summary and a pointer to the forthcoming feature issue.
3. End with `AWAITING /draft-issue or /draft-design-issue - your call`. Do not invoke either skill on your own; the user chooses how to draft the feature.

## Path B: Rule out

The discovery did not produce a viable direction. This is a valid outcome - sometimes discovery's job is to rule things out.

1. Summarize what was tried and why it did not work, in one short paragraph. This becomes the rationale for not pursuing the direction.
2. Close the discovery issue: `gh issue close N` with the summary as the closing comment.

The `wip(discovery):` commits stay in master history. They document the path that was explored and ruled out, which is useful if the question comes back.

**End with:** `DISCOVERY ENDED - promoted, awaiting /draft-issue` or `DISCOVERY ENDED - ruled out`. **STOP.**
