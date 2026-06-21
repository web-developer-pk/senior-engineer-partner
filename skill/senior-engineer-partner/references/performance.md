# Performance Reference

Performance regressions are a top "looked done, wasn't" failure: the feature works, the tests pass, the demo is fine on ten rows — and it falls over on ten million. Like security, performance is not something you sprinkle on at the end; it's a property you design for and *measure*, not guess at. This file is the working checklist for performance-adjacent work.

Read this file when a change touches: a loop over data of unknown size, a new or modified database query, a hot path (request handlers, render paths, anything per-item in a large set), a new external call in a request, or a payload that grows with data. Use it in two places — name the relevant risks during Plan, and *measure* the relevant ones during Verify.

## The "looked done, wasn't" performance failures

These are the ones that pass a shallow test and bite in production:

- **N+1 queries.** A loop that issues one query per item. Works on 3 items in a test, issues 5,000 queries in production. The single most common backend performance bug. Fix: eager-load / join / batch.
- **Missing index.** A query that filters or joins on an unindexed column does a full table scan. Fine at 1,000 rows, catastrophic at 10 million. Verify with the query planner, not by eye.
- **Unbounded result sets.** An endpoint or query with no `LIMIT` and no pagination returns everything. Grows silently with the data until it times out or exhausts memory. Pagination is mandatory on anything that can return more than one row (also OWASP API4).
- **Synchronous work in the request path.** Sending email, calling a third-party API, resizing an image, or doing heavy computation inline in a request handler. Each one adds its full latency to every request and couples your uptime to theirs. Move it to a background job.
- **Accidental quadratic.** A nested loop, or a `.includes()` / `in` check inside a loop over the same growing collection — O(n^2) hiding in readable code. Fine in tests, quadratic in production. Use a set/map for membership.
- **Oversized payloads.** Returning every field of every row (including large blobs) when the client needs three columns. Bloats serialization, transfer, and client parsing. Select what you need (also OWASP API3, excessive data exposure).
- **Unbounded growth structures.** In-memory caches, lists, or maps that only ever grow. Fine for a day, an OOM after a week. Bound them.

## During Plan

For a performance-adjacent change, state the expected performance characteristics, the same way you state applicable OWASP categories:

- **What's the hot path?** Which part runs often or per-item, and how big can the input get? "This endpoint is called once per page load; the loop is over a user's orders, which can be thousands for power users."
- **What's the expected complexity and query count?** "One query to fetch, one batched update — constant query count regardless of order count," not "a query per order."
- **What scale does it need to hold?** Tie it to a number from reality, not a vibe. If you don't know the realistic scale, that's a Clarify-time question.
- **Where's the budget recorded?** If the project has performance budgets (p95 latency, max query count per endpoint), name the relevant one. If it doesn't and this path is hot, propose recording one in `CLAUDE.md` / `ARCHITECTURE.md` non-functional notes.

## During Verify — measure, don't guess

"It seems fast enough" is the performance equivalent of "this should work." Get a real signal:

- **Database queries: read the plan.** `EXPLAIN ANALYZE` the actual query against a realistically-sized dataset (not three seeded rows). Confirm index usage; confirm no unexpected sequential scan on a large table. A query that's instant on an empty test DB tells you nothing.
- **Query count: assert it in a test.** For endpoints, the strongest regression guard is a test that counts queries and asserts the expected number (`assertNumQueries` in Django, query-count helpers in most ORMs, or a query-logging fixture). This catches N+1 the moment it's reintroduced — far better than noticing latency in production.
- **Hot-path timing: benchmark the realistic case.** Time the operation over a realistic input size, not the test fixture. Even a crude `time`/timer around the call on representative data beats guessing. For algorithmic changes, test at two sizes and confirm the scaling is what you claimed.
- **Payload size: check it.** Log or assert the response size for a realistic record. A list endpoint that returns 2 MB per page has a problem regardless of latency.
- **Concurrency, where it matters.** If the path will see real concurrent load, a single-request timing is not enough — note that load testing is needed before high-traffic deploy, even if you can't run it here.

When you genuinely can't measure in the current environment (no realistic dataset, no load tool), say so explicitly and state what *should* be measured before this ships — exactly as you would for an unrunnable security check. Don't silently assume it's fine.

## Tests as performance regression guards

The failures above come back the moment someone refactors. Lock them down:

- **Query-count assertions** on endpoints that were vulnerable to N+1.
- **A bound check** on anything that should be paginated (assert the response never exceeds max page size).
- **A scaling test** for algorithmic-complexity-sensitive code: assert it completes within budget at the high end of realistic input.

A performance property without a test will regress, the same way a security control without a test regresses.

## Recording in the project docs

When you establish a performance budget or discover a performance-critical path, record it so the next session inherits it:

- Latency / query-count budgets for hot endpoints -> `CLAUDE.md` (verification section) or `ARCHITECTURE.md` non-functional notes.
- "This path is performance-sensitive because X" -> a note near the code and in `ARCHITECTURE.md`.
- A hard performance rule that must always hold ("`/api/export` must stream, never buffer the full result set") -> `INVARIANTS.md`, with the enforcing test.
