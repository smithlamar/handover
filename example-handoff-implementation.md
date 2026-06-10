# Customer Stats API — Design Handoff (2026-06-02)

> First handoff for this branch — nothing to supersede. This cycle settled the
> read API's **contract** (the trait + return type). The implementation behind it
> does not exist yet — building it is the next session's goal.

## Resume here

> The brief for the next session. If you were told to "pick up handover <this
> file>", start here, then read in the order below before acting.

- **Goal:** Implement `CustomerStatsRepository` against the database — including
  designing the implementation, since none exists yet. The one open decision is
  where streaks are computed (SQL window query vs. derived in Scala).
- **Read in order:** `docs/db-schema.md` (the `customers` + `daily_activity`
  tables) → this doc → `repository/CustomerStatsRepository.scala` (the contract
  you're implementing).
- **Starting state:** clean, branch `design/customer-stats-api`, HEAD `c4e1a07`,
  version `0.4.0`. The trait + domain types are committed; no implementation yet.

## Pickup state
- **Repo:** `customer-api`, branch `design/customer-stats-api`, HEAD `c4e1a07`,
  version `0.4.0`.
- The domain types and the repository trait are committed as the **agreed
  contract** (`d4f9b21`). There is **no implementation** behind the trait — the
  method is unimplemented on purpose; wiring it up is the objective below.
- **Orientation entry point:** this handoff, then the two committed files, then the
  DB schema doc (read order below).

## What this cycle accomplished

Settled the read-side contract for fetching per-customer activity stats. The
design lived only in this session's discussion, so it is captured here and in the
two committed files — there is no separate design doc.

`domain/CustomerStats.scala` and `repository/CustomerStatsRepository.scala`:

```scala
final case class CustomerId(value: String) extends AnyVal

final case class CustomerStats(
  activeDaysStreak: Int,
  longestActiveDaysStreak: Int,
  lastActive: Instant,
  accountCreatedAt: Instant
)

trait CustomerStatsRepository {
  /** Stats for the customer, or None if no such customer exists. */
  def getCustomerStats(customerId: CustomerId): Future[Option[CustomerStats]]
}
```

Decisions made, and why:
- **`CustomerId` as a value class over a raw `String`** — type safety at call
  sites with no runtime allocation cost. Cheap to introduce now, expensive to
  retrofit later.
- **`Future[Option[CustomerStats]]` return type** — `Future` because the call hits
  the database; `Option` to model "no such customer" as a value rather than a
  thrown exception. Not-found is an expected outcome, not an error.
- **Read-only, single method** — no create/update in scope. The trait is the
  boundary; the service layer above it is out of scope this cycle.

## Objective for next session: implement `CustomerStatsRepository`

Build the database-backed implementation behind the agreed trait. **The
implementation design does not exist yet — designing it is part of this goal.**

The one real open decision to resolve during implementation:
- **Where are streaks computed?** Two options, pick during impl:
  - (a) a SQL window query over the `daily_activity` table (push computation to the
    DB),
  - (b) fetch raw activity rows and derive the streak in Scala.
  Tradeoff is DB cost / query complexity vs. in-memory work and row transfer. No
  decision was made this cycle — it depends on the `daily_activity` row volume,
  which the next session should check first.

The rest follows from the schema:
- `lastActive` and `accountCreatedAt` come directly from the `customers` and
  `daily_activity` tables — see `docs/db-schema.md` (don't restate it here; read it).
- Return `None` when the `customerId` has no `customers` row.
- Add unit tests against a mocked data source covering: a customer with an active
  streak, a customer with a broken/zero streak, and a non-existent customer.

Read order: `docs/db-schema.md` (the `customers` + `daily_activity` tables) → this
handoff → `repository/CustomerStatsRepository.scala` (the contract you're
implementing).

## Execution guideline (pause vs proceed)
- **Pause and consult before:** changing the trait signature — it is the agreed
  contract, and callers will be written against it; adding DB columns or a
  migration; picking the streak-computation approach **if** it meaningfully changes
  query cost or read latency (that's a design call worth surfacing).
- **Proceed without checking in for:** writing the implementation behind the
  agreed trait; adding unit tests against a mocked data source; local refactors and
  helper extraction within the implementation.

## Deferred / carried over
- **Streak-computation approach** — the one substantive design decision, left to
  the implementation session on purpose (see Objective).
- **Caching** of stats results — out of scope; revisit only if read latency
  becomes a concern.
- **Write path** (recording activity, creating customers) — not in scope here.

## Working-tree state at handoff
```
customer-api : clean, branch design/customer-stats-api, HEAD c4e1a07, version 0.4.0
```
- Committed as the contract: `domain/CustomerStats.scala`,
  `repository/CustomerStatsRepository.scala` (trait method unimplemented by design).
- No implementation file exists yet — the next session creates it.
