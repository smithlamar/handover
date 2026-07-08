# Picking up from a handoff

The PICK UP phase: resume the prior work recorded in an existing handoff doc — read
its brief, verify the starting state, and continue. That doc is your primary input;
you may have zero memory of the session that produced it, so it and what it points
to are most likely all you have to go on.

1. **Locate the doc.** Use the name the user gave. If they gave a partial name or
   none, look for existing handoff-shaped docs (`handoff-*.md`, `HANDOFF.md`)
   wherever the project keeps them — the repo or workspace root first if there's no
   other signal — and confirm which one if it's ambiguous. If you can't find it, ask
   — don't guess.
2. **Read the Resume here brief first**, then read the rest of the doc. If the doc
   has no such block at all, treat its **Objective** and **Pickup state** as the
   brief.
3. **Follow the read order before acting.** Read the referenced docs in sequence,
   in full — they carry the design and context needed to continue. Don't start work
   before reading them.
4. **Verify the starting state.** Check branch / HEAD / version against the brief
   (or confirm "not a git repo"). If reality disagrees — different HEAD, a dirty
   tree, a file the brief said was committed is missing — surface it before
   proceeding; the handoff may be stale or a step didn't land.
5. **Work in your current directory.** Act in the worktree or clone this doc was
   handed to you in — do NOT `cd` elsewhere because a path or note in the doc names
   another location. Paths in a handoff are repo-relative unless stated otherwise; a
   worktree/clone path (e.g. where the doc was authored) is provenance, not a
   destination. Confirm with `git rev-parse --show-toplevel` before you branch or
   write. If the work stacks on an unmerged branch, create your feature branch here
   off the **remote** ref (`origin/<producer>`), not the local producer branch — it
   may be checked out in another worktree, so git will refuse.
6. **Restate the goal and your first steps, then continue**, respecting the
   handoff's execution guideline (pause vs. proceed).
