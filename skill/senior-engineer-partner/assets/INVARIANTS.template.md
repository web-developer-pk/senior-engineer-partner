# INVARIANTS.md

The "never, ever" list. Things that must be true at all times, regardless of what any code is doing, regardless of whether an engineer remembered to enforce them.

Each invariant here is a promise the system makes. Each one deserves an enforcing test. If an invariant is not tested, it is not actually an invariant — it's a hope.

## How to use this file

- Before writing code that touches a critical boundary (auth, tenancy, billing, deletion), reread the relevant invariants. If your change could violate one, stop and rethink.
- When you discover a new invariant (often by fixing a bug caused by its violation), add it here and write a test that enforces it.
- When you write a test that enforces an invariant, link it from the invariant entry so future readers can find the enforcing code.

## Invariant entry format

Each entry follows this shape:

```markdown
### INV-NNNN: <short name>

**Statement:** <the invariant, stated as a must-hold condition>

**Why it matters:** <what breaks if this is violated — data leak, financial loss, compliance breach, user harm>

**Enforced by:** <test file(s), DB constraint(s), middleware, etc.>

**Known risks:** <places in the code where a careless change could break this>
```

## Invariants

<!--
Add entries below. Examples to get started for a multi-tenant SaaS:

### INV-0001: Tenant data isolation

**Statement:** No query, API response, background job, log line, or error
message ever surfaces data belonging to Account A to a User of Account B.

**Why it matters:** Violation is a data breach. Regulatory, reputational,
and contractual consequences are severe.

**Enforced by:**
- Every query that reads multi-tenant tables must filter by `account_id`.
  We enforce this via a SQLAlchemy event listener in `backend/db/tenant_guard.py`
  that rejects any query on tenant tables without an `account_id` filter.
- Integration test suite `tests/integration/test_tenant_isolation.py` seeds
  two accounts and verifies no endpoint can return the other's data.
- Log formatter redacts `account_id` from any log line that doesn't originate
  from an authenticated request matching that account.

**Known risks:**
- Admin / support tooling intentionally reads across tenants. Those endpoints
  are in `backend/admin/` and require a separate permission check. Do not
  colocate admin reads with regular endpoints.
- Background jobs must set `account_id` context at the start of execution,
  not rely on inheriting it.

### INV-0002: Billing events are append-only

**Statement:** Rows in `billing_events` are never updated or deleted after
creation. Corrections are made by inserting reversing events.

**Why it matters:** Financial audit trail. Tax and revenue reports reconstruct
from this log — mutation makes reports non-reproducible.

**Enforced by:**
- Postgres: `billing_events` has no UPDATE or DELETE permission granted to
  the app role. Migration `20260301_billing_events_append_only.sql`.
- Code review: any PR touching `billing_events` writes gets a second reviewer.

**Known risks:**
- Developers sometimes reach for `.update()` to fix a recent event. Don't.
  Insert a reversing event instead.

### INV-0003: Soft-deleted user data is not served

**Statement:** Users with `deleted_at IS NOT NULL` do not appear in any
product surface: no login, no API response, no search result, no admin
listing below the "deleted users" section, no mention in other users'
comment/activity feeds.

**Why it matters:** User expectation and privacy commitment. A deleted
account that still shows up is experienced as a bug at best and a privacy
violation at worst.

**Enforced by:**
- Default query scope on `User` model filters deleted rows.
- Feed / activity endpoints explicitly join through `User` so deleted
  authors are filtered.
- Integration tests under `tests/integration/test_user_deletion.py`.

**Known risks:**
- Raw SQL queries bypass the default scope. Any raw SQL on `users` must
  add `WHERE deleted_at IS NULL` explicitly.
- Denormalized caches (Redis, search index) need invalidation on delete.
-->
