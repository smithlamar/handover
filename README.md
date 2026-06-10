# handover

A [Claude Code Agent Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
that turns "wrap up and hand off this session" into a repeatable workflow: clean up
stale handoff docs, write a structured new one that leads with a self-contained
brief, persist it, and hand you a short one-line command to resume in a fresh
session. It also handles the other end of the baton — resuming work from a handoff
doc.

## Why this exists

A long session's context window keeps growing, and you pay for that prefix on every
turn. Eventually you compact or clear it — but native compaction just shrinks the
context in place; it leaves no artifact you can re-read, hand to a teammate, or pick
up days later. A handoff is, in effect, a **durable, curated compaction**: it
captures the scope, decisions, and state into a file *before* you clear context, so
the next session — a fresh agent, you tomorrow, or a colleague — resumes from a
deliberate brief instead of a lossy auto-summary. The file is the artifact native
compaction never leaves you.

## What it does

When you say "prep a handoff" / "do a handover" / "hand off" / "wrap up the
session", the skill runs three steps:

1. **Clean up stale handoff docs** — retire ones whose work has shipped, leave
   living reference docs (and in-flight handoffs) alone.
2. **Write the new handoff doc** — a consistent structure that leads with a
   **Resume here** brief (goal, read order, starting state), then pickup state,
   what-this-cycle-accomplished, next objective, an execution guideline (when to
   pause vs. proceed), deferred items, and working-tree state.
3. **Persist + hand you the pickup line** — commit the docs (or leave them
   untracked, your call), then give you a short line to paste into a fresh session:
   `Pickup handover <file>` (or the deterministic slash form, `/handover <file>`).

The point of a handoff is that the successor has *zero memory* of the prior
session, so the doc must be specific and self-contained — every assumption spelled
out or reachable via a path it cites. The brief lives **in the doc**, not in a
prompt you have to copy, so you can't lose it by forgetting to copy a prompt.

## Resuming

When you paste `Pickup handover <file>` — or the slash form `/handover <file>`, which
triggers the skill deterministically — into a fresh session, the same skill reads
the doc's **Resume here** brief, follows the read order, verifies the starting
state against reality, and resumes the work.

## Files

- `SKILL.md` — the skill itself (metadata + workflow). This is what Claude loads.
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
placing the repo's contents at `<skills-dir>/handover/`. From inside your clone:

```sh
# Copy it in (simplest)
cp -r . ~/.claude/skills/handover

# Or symlink your clone, so edits stay live and version-controlled in place
ln -s "$(pwd)" ~/.claude/skills/handover
```

On Windows, use a directory junction instead of a symlink (no admin required), run
from the repo root:

```bat
cmd /c mklink /J "%USERPROFILE%\.claude\skills\handover" "%CD%"
```

The skill loads at the start of the next session.

## Verify

Start a fresh session and ask it to "prep a handoff" — it should run the workflow.
To test the other direction, point a fresh session at a handoff doc with
`Pickup handover <file>` and confirm it reads the brief and resumes.

## License

MIT No Attribution (MIT-0). See [license.txt](license.txt).
