---
description: Close a discovery session by either promoting the sketches into a real issue or discarding them. See ISSUES.md §"Discovery issues".
---

End the current discovery session. Read and follow @~/.claude/docs/ISSUES.md §"Discovery issues".

**This skill is user-invoked only.** Per CLAUDE.md §1, you may never invoke `/end-discovery` on your own initiative. The user decides when discovery ends.

The user must indicate one of two outcomes:

## Path A: Promote

The discovery produced enough clarity to draft a real issue.

1. Summarize what was learned during the session in one short paragraph. Cite specific sketches/commits.
2. Invoke `/draft-issue` (or `/draft-design-issue` if the user prefers a one-pass draft) to create the proper feature issue with ACs.
3. Link the new feature issue from the discovery issue, e.g. *"Promoted to #NNN"*.
4. Decide the sketch fate with the user:
   - Discard the `wip(discovery):` commits (preferred for a clean history; the new feature issue starts fresh).
   - Cherry-pick specific sketches into the feature implementation later, then discard the rest.
5. Close the discovery issue: `gh issue close N` with a comment pointing to the promoted issue.

## Path B: Discard

The discovery did not produce a viable direction. This is a valid outcome - sometimes discovery's job is to rule things out.

1. Summarize what was tried and why it did not work, in one short paragraph. This becomes the rationale for not pursuing the direction.
2. Trash the `wip(discovery):` commits (use `git reset` or `git branch -D` as appropriate - confirm the destructive action with the user first per CLAUDE.md "Executing actions with care").
3. Close the discovery issue: `gh issue close N` with the summary as the closing comment.

**End with:** `DISCOVERY ENDED - promoted to #NNN` or `DISCOVERY ENDED - discarded`. **STOP.**
