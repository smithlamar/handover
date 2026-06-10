# Orders Service — Latency Investigation Handoff (2026-05-27)

> Replaces `handoff-2026-05-25.md`, now superseded. That doc's objective ("find
> the source of the p99 spike") was the live hunt; **this cycle isolated the slow
> path to the order-enrichment cache and found the unbounded growth**, but did not
> finish mapping which call site fails to evict. This doc carries that forward.

## Resume here

> The brief for the next session. If you were told to "pick up handover <this
> file>", start here, then read in the order below before acting.

- **Goal:** Finish mapping the TTL-less cache inserts — find every call site that
  inserts into `EnrichmentCache` without an expiry, then confirm the fix shape and
  verify p99 stays flat under the load profile.
- **Read in order:** `docs/investigation-notes.md` (the "Enrichment cache —
  findings (2026-05-27)" block + the "Repro setup" section) → this doc →
  `internal/enrich/cache.go` (the `put` / `putRaw` / `sweepExpired` trio).
- **Starting state:** clean, branch `investigate/p99-latency`, HEAD `a1b9f3c`,
  version `2.14.0`. Reproduce with `make load-test PROFILE=enrichment`.

## Pickup state
- **Repo:** `orders-service`, branch `investigate/p99-latency`, HEAD `a1b9f3c`,
  version `2.14.0` (unchanged this cycle — no deploy; everything below reproduces
  against the current build).
- **Reproduction is set up and reliable.** `make load-test PROFILE=enrichment`
  drives the p99 spike within ~90s. The heap snapshot tooling is wired
  (`scripts/heap-diff.sh`) — attach the profiler on pickup.
- ⚠️ **Uncertain whether the new tracing spans added to `EnrichmentCache.get`
  were committed before the session ended** — re-add them from this doc if missing
  (see step 1 below). The baseline profile capture `profiles/baseline.pprof` is the
  clean reference, untouched.
- **Orientation entry point:** `docs/investigation-notes.md` → the new
  **"Enrichment cache — findings (2026-05-27)"** block holds the full call-site
  map; this handoff is the narrative + the next move.

## What this cycle accomplished
1. **Re-validated the repro** on `2.14.0` (p99 climbs from 40ms to >800ms under the
   enrichment load profile) — no code change required to reproduce.
2. **Isolated the slow path** via heap diffing across the load test: snapshot at
   calm baseline, drive load, snapshot again, diff. Allocation growth concentrated
   in a single map held by `EnrichmentCache`.
3. **Found the unbounded map** — `EnrichmentCache.entries` grows monotonically and
   is never trimmed under the load profile. Provisional root cause, BUT the cache
   *does* have a TTL sweep (`sweepExpired`, runs every 60s) → so the leak is
   probably a path that inserts without setting an expiry, not a missing sweep.
   Label provisional pending a call-site tie-in.
4. **Ruled out `RequestContext.metadata`** — it grows during a request but is
   released on response; not the leak (removed from suspects).
5. **`grep` + call-graph mapped the insert sites:** `EnrichmentCache.put` (sets
   TTL, looks correct) and a second path `EnrichmentCache.putRaw`
   (`internal/enrich/cache.go:212`) that inserts **without** a TTL — the likely
   culprit. Callers of `putRaw`: the batch-prefetch warmup
   (`prefetch.go:88`) and the fallback hydrate (`hydrate.go:140`).
6. **Captured a candidate trace** showing `putRaw` from the prefetch warmup, but
   ⚠️ **the change-test was inconclusive** — disabling prefetch did not fully flatten
   the growth, so there is likely a **second** TTL-less insert still in play.
   Mapping every `putRaw` caller is the unfinished objective.

## Objective for next session: FINISH MAPPING THE TTL-LESS INSERTS
1. **Definitive locate (do this FIRST):** add a temporary assertion/log in `putRaw`
   that records the caller (stack frame) on every insert, run the load profile, and
   collect the full set of call sites that hit it. ⚠️ **Remove the temporary log
   before any commit** — it is hot-path and noisy.
   - Fallback if the log is too noisy: sample 1-in-100 inserts, or set a conditional
     breakpoint on `putRaw` filtered to entries with no expiry.
2. **Confirm the fix shape:** decide whether each `putRaw` caller should switch to
   `put` (with TTL) or whether `putRaw` itself should default to the cache's TTL.
   Tie the decision to whether any caller legitimately needs a permanent entry.
3. **Verify against the repro:** apply the fix, re-run `make load-test
   PROFILE=enrichment`, confirm p99 stays flat and the heap diff shows bounded
   growth.
4. **Decide on a regression guard:** a test that asserts the cache size stays
   bounded under sustained inserts, so this can't silently regress.

Read in order: `docs/investigation-notes.md` (the "Enrichment cache — findings
(2026-05-27)" block + the "Repro setup" section) → `internal/enrich/cache.go`
(the `put` / `putRaw` / `sweepExpired` trio) → this handoff.

## Execution guideline (pause vs proceed)
- **Pause and consult before:** shipping any fix to a shared branch or deploying;
  changing cache TTL/eviction behavior in a way that alters production memory or
  latency characteristics; marking a call site as "the confirmed root cause"
  without a behavioral repro proving it; refactoring the cache API beyond the
  minimal fix.
  - ⚠️ Do **not** lower the global TTL as a blunt fix — it would mask the leak and
    change hit-rate for every caller. **Find the specific TTL-less insert first.**
- **Proceed without checking in for:** read-only profiling and heap diffs; adding
  and removing temporary instrumentation (revert before commit); recording
  confirmed findings in the notes doc; re-running the load test; ruling candidates
  in or out.

## Side notes / deferred
- **No standing dashboard for cache size** — the hunt is profiler- and
  snapshot-driven. A size gauge metric would make future regressions obvious;
  deferred as a nice-to-have.
- **The load profile is reliable but synthetic** — production traffic mix may hit a
  different TTL-less path. Worth a prod heap snapshot to cross-check once the
  synthetic leak is fixed.
- **`sweepExpired` interval (60s) was briefly suspected** but ruled out — entries
  with no expiry are never eligible for the sweep regardless of interval.
- Stale artifacts from the 05-23 session (`profiles/scan-*.pprof`) — keep as
  reference only; not part of the current trail.

## Working-tree state at handoff
```
orders-service : clean, branch investigate/p99-latency, HEAD a1b9f3c, version 2.14.0
```
- ⚠️ Verify the `EnrichmentCache.get` tracing spans are present; re-add from step 1
  if the commit didn't land.
- Key files: `internal/enrich/cache.go` (the cache under investigation),
  `docs/investigation-notes.md` (source of truth, updated this session),
  `scripts/heap-diff.sh` + `profiles/baseline.pprof` (repro tooling),
  `handoff-2026-05-27.md` (this file, replaced `handoff-2026-05-25.md`).
