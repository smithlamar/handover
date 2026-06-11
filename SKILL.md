---
name: handover
description: Produces a session handoff document that leads with a self-contained "Resume here" brief, and resumes work from one. Use to PRODUCE a handoff when the user asks to "prep a handoff", "do a handover", "hand off", or "wrap up the session" — covers cleaning up stale handoff docs, writing the new one (resume brief, pickup state, accomplishments, next objective, execution guardrails, deferred items, working-tree state), persisting it, and handing the user a short pickup line. Use to RESUME when the user says "Pickup handover <file>", "pick up handover <file>", or "/handover <file>" — or points a fresh session at a handoff doc — reads the brief, follows the read order, verifies the starting state, and continues the work. The argument disambiguates: a handoff filename means RESUME; a stated goal (or no argument) means PRODUCE.
---

# Handover

A handoff document lets a successor — a fresh agent instance with zero memory of
this conversation, or a human teammate — resume work without re-deriving what
already happened. The successor's only inputs are the handoff doc and whatever it
points to, so the doc must be specific and self-contained.

This skill covers both ends of a handover: **producing** one (Steps 1–3 below) and
**resuming** from one (see "Resuming from a handoff" at the end). If the user is
pointing a fresh session at an existing handoff doc — e.g. "pick up handover
<file>" — skip to that section.

## The next session's goal is a required input

Almost every handoff is written *toward* a goal for the next session — "implement
the design we just finished", "continue the investigation", "review and merge the
branch". That goal is the most important input: it determines both what to include
and how much detail to carry forward. Treat it as required.

If the request doesn't state a goal, ask for it before writing — a one-line intent
is enough. The goal headlines the doc's **Resume here** brief, drives the
**Objective** section, and calibrates the detail of everything else (see Step 2).

## Carry context forward — don't do the next session's work

Producing a handoff is an act of *preserving* context before it's cleared, not of
spending it. The goal belongs to the **successor**; the current session's job is to
distill what it already knows into the doc, not to start executing the objective.
Doing the next session's substantive work now burns the very context the handoff
exists to conserve — and usually does it worse, against an already-full context
window near the end of its useful life.

Light validation that makes the handoff *trustworthy* is fair game — confirming a
path exists, a command runs, the branch and HEAD are what you claim. But the
analysis, design, or implementation the goal calls for should be reserved for the
successor. When you catch yourself digging into the problem rather than describing
it, stop and package the lead instead: what to do, where to look, and what is already
known. A handoff that hands over a clear objective and the context to act on it has
done its job; it does not need to have started the job.

Run all three steps below in order. Don't stop after one or two and wait for
confirmation unless a step is genuinely blocked (see each step's stop conditions).

Copy this checklist and track progress:

```
Handover progress:
- [ ] Confirm the next session's goal (ask if not stated)
- [ ] Step 1: Clean up stale handoff docs
- [ ] Step 2: Write the new handoff doc (detail calibrated to the goal)
- [ ] Step 3: Persist the docs (commit or leave untracked, per the user) + hand the user the pickup line
```

## Step 1: Clean up stale handoff docs

Remove prior handoff docs that are now fully consumed (the work they tracked has
shipped or been superseded).

- **Find them.** Typical locations: `docs/handoff-*.md`, `docs/v*-handoff.md`, a
  top-level `HANDOFF.md` — but check wherever the project keeps them.
- **Verify staleness before deleting.** Briefly confirm the work each doc tracked
  is done or superseded. Don't delete a doc covering work that is still in flight.
- **Leave active reference docs alone.** Design plans, tracking tables,
  architecture notes, and audit logs are NOT handoffs — they are living
  references. Only retire docs that are specifically session-handoff artifacts.
- **Only delete docs you have first-hand context on** — docs created in the
  current session, or docs explicitly referenced in the prompt that started this
  session. Any other handoff-shaped doc you merely *suspect* is stale: leave it
  and flag it for the user to clear. Don't delete on suspicion alone.

If there are no stale handoffs to remove, say so explicitly rather than skipping
the step silently.

## Step 2: Write the new handoff doc

- **Follow a documented convention if one exists.** If the project records where
  handoffs live and how they're named (e.g. in its `CLAUDE.md`, or a handoffs
  folder's README), follow that directly — don't rediscover it by grepping and
  reading old docs.
- **Otherwise name it** `docs/handoff-<YYYY-MM-DD>.md`, or match the pattern any
  existing handoffs use.
- **The structure below is the format** — you don't need to reverse-engineer it
  from old examples. Glance at the most recent prior handoff only to catch a
  project-specific deviation, then mirror that.
- **Be specific.** Name file paths, line numbers, identifiers, IDs, and commit
  hashes explicitly. Every concrete reference you include saves the successor a
  re-search. Vague summaries force the successor to rediscover what you already
  know.

### Calibrate the detail to the goal

There is no fixed level of detail. Infer it from three things: what was done this
session, what documentation already exists, and what the next session's goal
demands.

- **Match the detail to what the goal needs to act.** "Implement the design we
  finished" needs the design carried forward in usable form — the decided
  approach, the key interfaces or code snippets, the target files, and the
  reasoning behind the choices — so the successor can build without re-deriving it.
  "Continue the investigation" needs the live leads and the ruled-out suspects.
  "Review the change" needs the diff scope and what to scrutinize. A summary that's
  right for one goal is too thin for another.
- **Reference, don't duplicate.** If a design doc, plan, or spec already captures
  the detail, point to it (path + the sections to read, in order) instead of
  restating it. Pull only the load-bearing snippets inline — the ones the successor
  needs to act before opening every reference.
- **Persist decisions that live only in the conversation.** A design, an approach,
  or a rationale that was worked out in chat is lost to the successor unless the
  handoff records it. Either capture it in the handoff doc or write it to a
  durable doc and link that. Don't assume the next session can reconstruct it.

### Label confidence — separate what's proven from what's assumed

A handoff is read as fact. Anything you write without qualification, the successor
takes as established and builds on — so an unlabeled hunch becomes a false premise it
chases down a dead end, burning a whole cycle. The fix is cheap: keep proven and
assumed visibly distinct. What matters most is flagging the load-bearing claims that
are **not** fully verified — the unlabeled assumption is the one that does damage. A
fact that's a given or a settled decision doesn't strictly need a label (though one
never hurts); the priority is that no hunch is ever mistaken for proven.

- **Proven / verified** — you confirmed it this session. State *how* you know — the
  command you ran, the file and line you read, the behavior you reproduced — so the
  successor can re-check in seconds rather than re-derive from scratch.
- **Assumption / hunch / provisional** — plausible but unconfirmed. Carry it (leads
  are valuable), but label it as a lead, not a finding, and where you can, note what
  would confirm or refute it, how much rests on it being true, and — in brief — where
  it came from (the observation or reasoning that prompted it), so the successor can
  weigh the lead instead of inheriting it blind.
- **Ruled out** — record what you eliminated and why, so the successor doesn't
  re-walk a dead end you already closed.

Make the marker explicit and scannable — e.g. `Verified:`, `Assumption:`,
`Provisional —`, `Unconfirmed —`, `Ruled out:`. A one-word label costs nothing; a
successor mistaking a guess for a finding can cost it an entire session. The
investigation example models this — its root-cause lead is marked `provisional` and
its eliminated suspects `ruled out`.

Keep it lean, though: record *how* you know only when that detail is load-bearing for
the next session's work or saves it from repeating a step — not as ceremony. When
concise framing and exhaustive proof-tracking pull against each other, favor concise.

Use this structure (adapt section names to the project's existing handoffs):

```markdown
# <Project> Handoff — <date or version>

> If this replaces a prior handoff, note it here: "Replaces <old doc>, now
> superseded." Briefly say what carried over.

## Resume here

> The brief for the next session. If you were told to "pick up handover <this
> file>", start here, then read in the order below before acting.

- **Goal:** <the next session's objective, in one or two sentences.>
- **Read in order:** <this doc in full, then the external docs the successor needs
  to act — paths in sequence.>
- **Starting state:** <branch / HEAD / version, or "not a git repo — state is files
  on disk".>

## Pickup state
- Repo / branch / HEAD commit / version (or "not a git repo — state is files on disk").
- Anything the successor must know to even start: build version, live environment,
  what to attach to or open first, the orientation entry-point doc.

## What this cycle accomplished
- Concrete, numbered list. What was found, built, fixed, ruled out. Cite sources,
  and label each load-bearing claim verified vs. assumed (see "Label confidence"):
  - `Verified:` <claim> — <how you confirmed it: command run, file:line read, or repro observed>.
  - `Assumption:` <lead> — <what would confirm/refute it, how much rests on it, where it came from>.
  - `Ruled out:` <eliminated suspect> — <why, so the successor doesn't re-walk it>.

## Objective for next session
- The single next goal, stated plainly. What "done" looks like.
- Read order: the docs the successor should read, in sequence, to get oriented.

## Execution guideline
- **Pause and consult the user before:** <consequential or hard-to-reverse actions
  — irreversible changes, judgment calls the user would want a say in, value tuning
  that alters outcomes, spending real resources, or writing a load-bearing fact as
  "confirmed" without proof.>
- **Proceed without pausing for:** <read-only analysis, tactical renames, recording
  findings, reversible edit-tests, re-running known-safe scripts.>

## Deferred / carried over
- Open items not done this cycle, with enough context to pick each up cold.

## Working-tree state at handoff
- Per repo: clean/dirty, branch, HEAD, version. If not a git repo, list the key
  files and their roles.
```

The **Resume here** block at the top is the self-contained brief the next session
reads first — it carries the goal, the read order, and the starting state. Keep it
short; it points into the detailed sections below rather than repeating them. This
is why the user doesn't need to paste a long prompt (Step 3): the brief lives in
the doc, where it's durable and recoverable.

The **execution guideline** section matters: it tells the successor when to act
autonomously and when to check in, so it neither stalls on trivia nor barrels
through consequential calls. There's no universal line — set it for the work at
hand — but lean toward **pause** when an action is hard to undo, commits a
judgment call the user would want a say in, spends real resources, or rests on an
assumption that hasn't been verified; lean toward **proceed** for reversible,
low-stakes, mechanical work. Make both sides concrete for the project.

## Step 3: Persist the docs + hand the user the pickup line

**Whether handoff docs are committed to the repo is the user's call.** Some
projects track them in git; others keep them out of the repo — gitignored, in a
local-only directory, or simply left untracked. Follow the established convention
if there is one (match how existing handoff docs are handled). If there's no
precedent and the user hasn't said, ask which they want before touching git.

- **If committing:** stage the new handoff doc together with the deletions of the
  stale ones into a **single docs-only commit** — e.g.
  `docs: drop N stale handoff(s) + add <new handoff>`. Don't bundle code changes
  into this commit.
- **If not committing (or not a git repo):** leave the docs untracked and note in
  the handoff that its state lives as files on disk, not in version history.

**Hand the user a short pickup line, not a long prompt.** The full brief already
lives in the doc's **Resume here** block, so the user doesn't paste a prompt — they
just point a fresh session at the file:

```
Pickup handover <handoff-file-name>
```

For a guaranteed trigger, the slash form invokes the skill deterministically rather
than relying on phrase-matching (the argument is the handoff filename, which the
skill reads as a RESUME request):

```
/handover <handoff-file-name>
```

Give them one of these. A one-line command is hard to get wrong; a long paste-prompt
invites copy errors and is lost entirely if the user hits the wrong key or forgets
to copy it. The detail belongs in the doc, which is durable — if the user only
remembers the filename, the brief is still right there to recover from.

Resuming from that line is the other half of this skill — see **Resuming from a
handoff** below.

Worked examples (note how each carries a different level of detail, set by its
goal):

- [example-handoff-implementation.md](example-handoff-implementation.md) — goal is
  "implement the design we finished", so the doc carries the agreed contract
  forward inline (code) and persists the decisions made in conversation.
- [example-handoff-investigation.md](example-handoff-investigation.md) — goal is
  "continue the investigation", so the doc carries live leads and ruled-out
  suspects rather than a finished design.

## Resuming from a handoff

This is the consume side. When the user says "Pickup handover <file>", "pick up
handover <file>", "/handover <file>" (or points a fresh session at a handoff doc),
resume the prior work. The argument is the discriminator: a handoff filename means
resume; a stated goal means produce a new handoff (Steps 1–3 above).

1. **Locate the doc.** Use the name the user gave. If they gave a partial name or
   none, look in the usual places (`docs/handoff-*.md`, `docs/v*-handoff.md`,
   `HANDOFF.md`) and confirm which one if it's ambiguous. If you can't find it, ask
   — don't guess.
2. **Read the Resume here brief first**, then read the rest of the doc. If the doc
   has no Resume here block, treat its **Objective** and **Pickup state** as the
   brief.
3. **Follow the read order before acting.** Read the referenced docs in sequence,
   in full — they carry the design and context needed to continue. Don't start work
   before reading them.
4. **Verify the starting state.** Check branch / HEAD / version against the brief
   (or confirm "not a git repo"). If reality disagrees — different HEAD, a dirty
   tree, a file the brief said was committed is missing — surface it before
   proceeding; the handoff may be stale or a step didn't land.
5. **Restate the goal and your first steps, then continue**, respecting the
   handoff's execution guideline (pause vs. proceed).

## Notes

- If a step doesn't apply this cycle (e.g. no stale handoffs, or not a git repo),
  say so explicitly rather than skipping it silently.
- If something blocks a step in a way you can't resolve (e.g. the working tree is
  dirty with unrelated changes that can't be committed cleanly), pause and ask
  before improvising.
