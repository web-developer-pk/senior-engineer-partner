# HANDOFF.md

The living session cursor — a save-game for the work in progress. The first file
the next session (a fresh instance of Claude, after compaction, a crash, or a new
chat) reads to pick up exactly where the last one left off.

Overwrite this constantly: after any decision, before any risky/long operation, and
at the end of every work chunk. A stale HANDOFF is worse than none — keep it true.

This is NOT the backlog (that's `TODO.md`) and NOT long-term knowledge (that's
`DOMAIN.md` / memory files). It is only: where are we right now, and what's next.

---

## State

- **Branch:** <!-- e.g. feat/oauth-migration -->
- **Ahead / behind remote:** <!-- e.g. 3 ahead, 0 behind -->
- **Last verified green:** <!-- commit SHA / "lint+test+build passing as of <time>" -->

## Done this session

<!-- One line each, only what's finished AND verified.
- Added POST /api/reset endpoint + button; 3 tests green.
- Confirmed schema: orders table uses `buyer_id` (migration 20240601), not `customer_id`.
-->

## Pending (in order)

<!--
1. Wire the reset button's loading state.
2. Add cross-tenant 404 test for the reset endpoint.
-->

## THE single next action

<!-- The ONE concrete thing to do first. Not a list.
e.g. "Run `pytest tests/integration/test_reset.py` — it was written but not yet run."
-->

## Live vs. local (not yet committed / not yet pushed)

<!--
- Dev server running on :8000 (uvicorn, reload on).
- Local DB migrated to head; migration NOT pushed.
- `FEATURE_RESET=1` set only in this shell.
- Uncommitted: server.py, test_reset.py.
-->

## Landmines / constraints for the next session

<!--
- INVARIANT: dev sessions never write to the prod DB. Re-check before any write.
- test_session_expiry is flaky (timezone-dependent) — not your change.
- The 91-vs-90 discount test is a known-bad assertion; don't "fix" the function to match it.
-->
