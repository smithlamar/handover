# Cleaning up a finished handover

The CLEAN UP phase: when the work a handover tracked is **totally done** — no
successor session follows — delete the handover artifacts and close the loop.
Unlike the produce phase, no new handoff doc is written: there is nothing left to
hand over. This differs from the produce phase's Step 1 (which retires *stale*
handoffs while producing a new one); cleanup is the terminal step after the final
session of the workstream.

Copy this checklist and track progress:

```
Cleanup progress:
- [ ] Find the handover artifacts in jurisdiction
- [ ] Verify the work is complete (enumerate outstanding items + confirm if not)
- [ ] Delete + persist (match how the docs were tracked)
- [ ] Report what was removed and what was left alone
```

1. **Find the artifacts.** The handoff doc that started or steered this session
   (the pickup source) is the primary target. Then check wherever the project keeps
   handoffs for any others — the repo or workspace root first if there's no other
   signal. Jurisdiction is the produce phase's Step 1 rule: only docs you have
   first-hand context on — created in this session, or referenced in the prompt
   that started it. Everything else — handoff-shaped docs outside jurisdiction,
   anything you're unsure about — stays put; flag it for the user to decide.
   Living reference docs (design plans, tracking tables, architecture notes,
   audit logs) are off-limits, as always.
2. **Verify the work is actually complete.** Check each doc's objective and
   deferred items against reality before deleting. If anything looks unfinished —
   open objectives, deferred items never picked up, incomplete tasks in the
   session — don't delete yet: enumerate each outstanding item with a concise
   brief (enough for the user to judge it or pick it up later), and ask for
   explicit confirmation that they want to clean up anyway. Deferred work may be
   intentionally abandoned — but that's the user's call to make, not yours.
3. **Delete and persist.** Match how the docs were tracked: if they're committed,
   stage the deletions as a single docs-only commit (e.g.
   `docs: remove consumed handoff(s)`); if untracked or gitignored, just delete
   the files.
4. **Report.** List what was deleted, what was left alone (and why), and any
   outstanding items the user chose to drop.
