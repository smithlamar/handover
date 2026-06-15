---
name: handover
description: "Produce, pick up, or clean up a session handoff doc. Slash-only: /handover <goal> produces one, /handover <file> picks one up, /handover cleanup closes out."
argument-hint: [next-session goal | handoff-filename | cleanup]
disable-model-invocation: true
---

# Handover

A handoff document lets a successor — a fresh agent instance with zero memory of
this conversation, or a human teammate — resume work without re-deriving what
already happened. The successor's only inputs are the handoff doc and whatever it
points to, so the doc must be specific and self-contained.

This skill has three phases, each documented in its own file. Determine the phase
from the argument passed to `/handover`, then read and follow that phase's file:

| Argument                                         | Phase                                                                       | Read & follow            |
|--------------------------------------------------|-----------------------------------------------------------------------------|--------------------------|
| A stated goal for the next session, or no arg    | **Produce** a new handoff or update a related existing doc                  | [handoff.md](handoff.md) |
| A handoff filename (e.g. `handoff-cache-ttl.md`) | **Pick up** the work the doc tracks                                         | [pickup.md](pickup.md)   |
| The literal `cleanup`                            | **Clean up** related handover docs once the work they tracked is fully done | [cleanup.md](cleanup.md) |

For produce, the next session's goal is a required input — if none is stated, ask
for a one-line intent before writing.

Don't act from this router alone; the phase file carries the actual workflow,
checklists, and guardrails. Read it before doing anything.

## Notes

- If a step doesn't apply this cycle (e.g. no stale handovers, or not a git repo),
  say so explicitly rather than skipping it silently.
- If something blocks a step in a way you can't resolve (e.g. the working tree is
  dirty with unrelated changes that can't be committed cleanly), pause and ask
  before improvising.
