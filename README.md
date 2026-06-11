# handover

A [Claude Code Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
that turns "wrap up and hand off this session" into a repeatable workflow: clean up
stale handoff docs, write a structured new one that leads with a self-contained
brief, persist it, and hand you a short one-line command to resume in a fresh
session. It also handles the other end of the baton — resuming work from a handoff
doc — and the final lap: cleaning up the handover artifacts once the work is done.

## Contents

- [Usage](#usage)
- [Why this exists](#why-this-exists)
- [Prepping a handoff](#prepping-a-handoff)
- [Resuming](#resuming)
- [Cleaning up](#cleaning-up)
- [Files](#files)
- [Installing](#installing)
- [Verify](#verify)
- [License](#license)

## Usage

One command, three flavors — the argument picks the phase:

- `/handover <describe next session goal>` — prep a handoff doc for that goal and hand
  back a one-line pickup command. The goal is required — invoked without one,
  the skill asks for it before writing.
- `/handover <file>` — resume in a fresh session after `/clear` (which destroys
  the old session's context and starts a new one): read the doc's brief, verify
  the starting state, continue the work.
- `/handover cleanup` — verify the tracked work is completely done, then delete
  the handover artifacts; use when no successor session follows.

Natural language works too: `prep a handoff`, `hand off`, or `wrap up the
session` all produce a new handoff doc and its pickup line; `Pickup handover
<file>` resumes; `clean up the handover docs` cleans up. The slash forms are just the deterministic spelling — a
filename argument means resume, the literal `cleanup` means clean up, and
anything else (or nothing) means prep.

## Why this exists

A long session's context window keeps growing, and you pay for that prefix on every
turn. Eventually you compact or clear it — but native compaction just shrinks the
context in place; it leaves no artifact you can re-read, hand to a teammate, or pick
up days later. A handoff is, in effect, a **durable, curated compaction**: it
captures the scope, decisions, and state into a file *before* you clear context, so
the next session — a fresh agent, you tomorrow, or a colleague — resumes from a
deliberate brief instead of a lossy auto-summary. The file is the artifact native
compaction never leaves you.

That packaging is *all* the producing session does — it distills what it already
knows for the successor to act on. It deliberately doesn't start solving the next
goal: spending the current context doing the next session's work defeats the point of
preserving it.

## Prepping a handoff

Prepping runs three steps:

1. **Clean up stale handoff docs** — retire ones whose work has shipped, leave
   living reference docs (and in-flight handoffs) alone.
2. **Write the new handoff doc** — a consistent structure that leads with a
   **Resume here** brief (goal, read order, starting state), then pickup state,
   what-this-cycle-accomplished, next objective, an execution guideline (when to
   pause vs. proceed), deferred items, and working-tree state.
3. **Persist + hand you the pickup line** — leave the docs untracked by default
   (they're session ephemera; committed only if you say so or the project
   already tracks them), then give you the short pickup line to paste into a
   fresh session.

The point of a handoff is that the successor has *zero memory* of the prior
session, so the doc must be specific and self-contained — every assumption spelled
out or reachable via a path it cites. It also marks confidence as it goes — what's
**verified** (and how it was confirmed) versus what's an **assumption or lead** — so
the successor doesn't mistake a hunch for a fact and chase it down a dead end. The
brief lives **in the doc**, not in a prompt you have to copy, so you can't lose it by
forgetting to copy a prompt.

## Resuming

In the fresh session, the same skill reads the doc's **Resume here** brief,
follows the read order, verifies the starting state against reality, and resumes
the work.

## Cleaning up

The skill finds the handover artifacts it has first-hand context on, verifies the
work is actually complete — if anything looks unfinished, it enumerates each
outstanding item and asks you to confirm before deleting — then removes the docs
and reports what went and what it left alone. No new handoff is written; cleanup
closes the loop.

## Files

- `SKILL.md` — the skill itself (metadata + workflow). This is what Claude loads.
- `README.md` — this file.
- `example-handoff-implementation.md` — worked example: an "implement the design we
  finished" handoff that carries the agreed contract forward inline.
- `example-handoff-investigation.md` — worked example: a "continue the
  investigation" handoff that carries live leads and ruled-out suspects.

## Installing

Claude Code discovers skills from two locations:

- **Personal** — `~/.claude/skills/` (available in all your projects)
- **Project** — `<your-project>/.claude/skills/` (scoped to one repo, shareable
  with collaborators)

This repository's root *is* the skill (it contains `SKILL.md`), so install it by
copying the repo's contents to `<skills-dir>/handover/` — however you prefer to
copy files. The skill loads at the start of the next session. To pick up a newer
version, re-copy over the same location.

## Verify

Start a fresh session and ask it to `prep a handoff` — it should run the workflow.
To test the other direction, point a fresh session at a handoff doc with
`Pickup handover <file>` and confirm it reads the brief and resumes. To test
cleanup, say `/handover cleanup` in a session whose work is done and confirm it
verifies completion before deleting anything.

## License

MIT No Attribution (MIT-0). See [license.txt](license.txt).
