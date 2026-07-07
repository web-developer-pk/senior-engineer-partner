# Context Documentation Files

These files live at the project root and persist context across sessions. Because Cowork has no memory between sessions, they are the mechanism by which the next session (possibly a fresh instance of you) understands the project and picks up where this one left off.

Keep them short, accurate, and current. Stale docs are worse than missing docs — they lie to the next session.

## The two tiers

**Engineering context (every project needs these):**

- `CLAUDE.md` — how to work in this project
- `ARCHITECTURE.md` — system design
- `DECISIONS.md` — ADR log
- `TODO.md` — in-flight and deferred work
- `HANDOFF.md` — the living session cursor (current state + the single next action), for surviving compaction, crashes, and cold starts

**Product/domain context (strongly recommended for SaaS / multi-tenant):**

- `PRODUCT.md` — what the product is and who it's for
- `DOMAIN.md` — the ubiquitous language
- `INVARIANTS.md` — the "never, ever" list

For greenfield SaaS, create the product/domain tier at kickoff. For existing projects that lack them, propose a dedicated context-setup task rather than bolting them onto unrelated work.

## CLAUDE.md

**Purpose:** How to work in this project. Conventions, commands, gotchas. The first file any new session reads.

**Audience:** The next instance of Claude (and the human).

**Length:** 1–3 screens. If it grows longer, split into sub-files and link.

**Required sections:** project summary, stack, directory layout, schema source of truth, conventions, verification commands, security baseline, gotchas, workflow pointers.

**Verification commands must be exact** — paste the actual commands, not "the usual tests." Future sessions will run these literally.

**Schema notes belong here too.** If the project has a database, record where the schema source-of-truth lives (migration directory, ORM model file, generated `schema.sql`) so future sessions know where to look when verifying column names. This directly supports the verify-don't-infer rule.

**Filled-in example:**

```markdown
# CLAUDE.md

Operating guide for Claude working in this project. Read this first.

## Project summary

One paragraph. What this project is and what it does.

## Stack

- Language(s): e.g., TypeScript 5.x + Python 3.12
- Framework(s): e.g., Next.js 14, FastAPI 0.110
- Database: e.g., PostgreSQL via Prisma
- Key libraries: e.g., React Query, Zod, pytest
- Package managers: e.g., pnpm (frontend), uv (backend)

## Directory layout

- `frontend/` — Next.js app, App Router
- `backend/` — FastAPI service
- `shared/` — TypeScript types shared between FE and BE
- `infra/` — Docker, migrations, deploy config
- `docs/` — ADRs and architecture notes

## Schema source of truth

Migrations in `backend/migrations/` (Alembic). ORM models in `backend/app/models/`
reflect the current migration state. When uncertain, run
`alembic upgrade head --sql` to see the canonical schema. Do NOT infer column
names from related tables — always read the migration or run the introspection.

## Conventions

- Naming: kebab-case file names, PascalCase components, snake_case Python
- Error handling: never swallow exceptions; use the `AppError` class in `backend/errors.py`
- Logging: structured logs via `pino` (FE) / `structlog` (BE); include request IDs
- Validation: Zod on the edges (FE), Pydantic on the edges (BE)
- Tests: TDD; unit tests colocated; integration tests in `tests/integration/`

## Verification

Run before claiming a task done:

- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm build`
- `cd backend && ruff check . && mypy . && pytest`
- `pnpm test:e2e` (smoke)

## Security baseline

- All endpoints under `/api/admin/*` require the `admin` role (enforced by middleware in `backend/auth/admin_guard.py`).
- All multi-tenant queries must filter by `account_id` (enforced by `tenant_guard` SQLAlchemy listener).
- Secrets come from `.env`; never commit them. Pre-commit hook (`gitleaks`) blocks common patterns.

## Gotchas

- The migration in `20240601_add_users.sql` is not idempotent — don't re-run it locally.
- `NEXT_PUBLIC_API_URL` must be set in `.env.local` or client fetches silently fail.
- Tests that hit the DB require `docker compose up db` first.

## Workflow

- Branch from `main`; PRs require all verification gates green.
- See `ARCHITECTURE.md` for system design.
- See `DECISIONS.md` for why things are the way they are.
- See `TODO.md` for in-flight work.
- See `DOMAIN.md` for term definitions.
- See `INVARIANTS.md` for rules that must always hold.
```

## ARCHITECTURE.md

**Purpose:** System design. What talks to what, data flow, module boundaries, key tradeoffs.

**Audience:** A new engineer (human or AI) onboarding to the codebase.

**Length:** 2–5 screens. Diagrams welcome (ASCII or linked images).

**What to include:**

- High-level component diagram (frontend, backend, database, external services, queues)
- Data flow for the primary user flows
- Module boundaries — what each major directory owns and does not own
- Integration points — external APIs, auth providers, payment processors, etc.
- Non-functional notes — performance targets, security model, deployment topology
- What's *not* in this codebase and why (e.g., "notifications live in a separate service")

**What to leave out:**

- Line-level implementation details — those belong in code comments or docstrings
- Anything likely to change daily — those belong in `TODO.md`

Update `ARCHITECTURE.md` any time a component boundary changes, a new service is added, or a key integration changes. Do not update it for normal feature work that fits within existing boundaries.

## DECISIONS.md

**Purpose:** Append-only log of non-obvious architectural decisions. The project's "why."

**Audience:** Future maintainers who will wonder why something was done a particular way.

**Format:** Lightweight ADRs (architecture decision records). One entry per decision.

**Filled-in example:**

```markdown
## ADR-0012: Use cursor-based pagination for the search endpoint

**Date:** 2026-04-20
**Status:** Accepted

### Context

The search endpoint originally used offset-based pagination. Performance degraded
past page ~50 due to the cost of counting and skipping rows in large tables.

### Decision

Switch to cursor-based pagination using the primary-sort key as the cursor.

### Consequences

- Pro: Constant-time pagination regardless of depth.
- Pro: Avoids skip-based issues when results mutate between requests.
- Con: Cannot jump to an arbitrary page; clients get "next/prev" only.
- Con: Existing clients must migrate; see MIGRATION.md for the shim.

### Alternatives considered

- Keyset pagination with composite keys — rejected due to added complexity for the small performance gain over simple cursor.
- Caching offset results — rejected; doesn't solve the fundamental scan cost.
```

**When to write an ADR:**

- Choosing between two or more real alternatives
- Adopting a new dependency that will shape the codebase
- Deviating from an existing convention in the project
- Any decision you would want to justify to a skeptical reviewer

**When NOT to write an ADR:**

- Obvious choices with no real alternative
- Implementation details that don't shape the architecture
- Temporary workarounds — those go in `TODO.md`

## TODO.md

**Purpose:** What's in progress, what's partially done, what's known-broken, what's next.

**Audience:** The next session (and the user).

**Length:** As long as it needs to be, but groom regularly. Close items that are done. Move stale items to an archive section if they're no longer relevant.

**Filled-in example:**

```markdown
# TODO.md

## In progress

- [ ] User profile editing — form is wired, validation pending. See `frontend/app/profile/edit/`.
  - Next step: hook up Zod schema, wire submit handler to `PATCH /api/users/me`.
  - Owner: current session.

## Blocked / needs input

- [ ] Payment webhook retry logic — waiting on product decision about idempotency key format.

## Known issues

- [ ] `GET /api/search` returns 500 on empty query string instead of 400. Low-pri.
- [ ] Flaky test: `tests/integration/test_auth.py::test_session_expiry`. Seems timezone-dependent.

## Tech debt

- [ ] Replace hand-rolled rate limiter in `backend/middleware/rate_limit.py` with `slowapi` once we bump FastAPI.

## Follow-ups from recent work

- [ ] Added `users.last_login_at` column but haven't added an index. Add one if query usage grows.
```

**Update rules:**

- At the end of every task, update `TODO.md` with anything deferred, anything partial, or anything you noticed that needs attention later.
- Never claim a task is "done" while leaving a stub in the code — record it in `TODO.md` instead and tell the user.
- When you complete a `TODO.md` item, check it off and then delete it (or move it to a `## Done` / archive section if the team prefers history).

## Creating engineering-tier files when missing

If any of the four engineering-tier files (`CLAUDE.md`, `ARCHITECTURE.md`, `DECISIONS.md`, `TODO.md`) are missing from the project root, create them as part of your current task. Use the templates in `assets/` as starting points but fill them in with real project info — do not leave placeholder text in place. A half-filled template is worse than no template, because it lies to the next reader.

For a brand-new project, it's fine for `ARCHITECTURE.md` and `DECISIONS.md` to start small — they'll grow as the project does. `CLAUDE.md` should be complete and accurate from day one. `TODO.md` can be empty (or list "Nothing in flight") but should exist.

## HANDOFF.md

**Purpose:** The living session cursor — a save-game of *where the work is right now*, so a fresh session (after compaction, a crash, or a new chat) resumes exactly where the last left off.

**Audience:** The very next session, which may be a cold start that never saw this conversation.

**Length:** Short and always current. It is overwritten constantly, not appended to.

**What it holds:** current branch + ahead/behind count; what's done-and-verified; what's pending; **the single next action**; live-vs-local state (uncommitted files, running servers, local-only migrations, shell-only env vars); and landmines/constraints the next session could get wrong.

**What makes it different from `TODO.md`:** `TODO.md` is the durable backlog (someday work, known issues, debt). `HANDOFF.md` is the volatile cursor (the next 30 seconds). Keep them separate or they drift and both become untrustworthy.

**When to update:** after any decision or constraint is established; before any risky/long/irreversible operation; and at the end of every work chunk — *not* only when you stop, because compaction and token-death happen mid-chunk. A stale `HANDOFF.md` is worse than none.

**Make it survive deterministically:** the resume-and-reconcile protocol that reads it, and the CLAUDE.md bootstrap plus optional Claude Code `SessionStart` / `PreCompact` hooks that re-inject and archive it, live in `references/session-continuity.md`. Because a skill may not load on a cold start, the bootstrap belongs in `CLAUDE.md` (auto-loaded), not only here.

See `assets/HANDOFF.template.md` for a starter.

---

# Product / Domain Context Tier

These three files encode *what the product is*, not *how the code is organized*. They are the highest-leverage prevention for a specific class of bug: the "plausible but wrong" bug, where code works but does the wrong thing because it was built against a different mental model than the rest of the system.

For greenfield SaaS or multi-tenant projects, create all three at kickoff. For existing projects that lack them, propose a dedicated context-setup task — do not bolt them onto an unrelated feature.

## PRODUCT.md

**Purpose:** What the product does, who it's for, what's in and out of scope. The tiebreaker when requirements conflict.

**Audience:** Anyone making scope decisions, including Claude during Clarify and Plan steps.

**Length:** One screen. If it grows past that, you're probably mixing feature specs in.

**What to include:**

- One-or-two-sentence elevator pitch
- Specific target users (role, company size, the moment they'd reach for this)
- 3–5 core jobs the product does
- What's explicitly out of scope
- Non-functional commitments (SLA, data residency, compliance)
- Success criteria (honest, not vanity metrics)

**When to update:** When product scope shifts, when a new user segment is added or dropped, when a commitment (like uptime or compliance) changes. Do not update it for feature-level additions within existing scope.

See `assets/PRODUCT.template.md` for the starter template with full worked examples.

## DOMAIN.md

**Purpose:** The ubiquitous language. Every term used in the code, the UI, and conversations about the product means the same thing here.

**Audience:** Every engineer and every Claude session that writes code or talks about the domain.

**Length:** As long as the domain is complex. Start small and grow as terms accumulate.

**What to include:**

- Core entities with precise definitions, lifecycle states, and state transitions
- Distinctions that matter (pairs of terms that sound similar but mean different things)
- Forbidden terms (ambiguous words banned in favor of precise alternatives)

**Why this file has outsized value:** A huge fraction of SaaS bugs come from two people (or two Claude sessions) using the same word to mean different things. "User," "account," "customer," "admin," "tenant" — these get conflated constantly. When the code matches `DOMAIN.md` exactly, that class of bug disappears.

**When to update:** Whenever a new domain term is introduced, an existing term is refined, or a term that was being used one way is found to have been used differently elsewhere. Update as part of the same task that touches the term — do not defer.

**Enforcement tip:** When reviewing your own code or a PR, scan for domain terms and check each against `DOMAIN.md`. Mismatches are bugs waiting to happen.

See `assets/DOMAIN.template.md` for the starter template with full worked examples.

## INVARIANTS.md

**Purpose:** The "never, ever" list. Rules that must hold at all times regardless of what code is doing.

**Audience:** Claude (and any engineer) before touching code near a critical boundary — auth, tenancy, billing, deletion, audit logs.

**Length:** As long as it needs to be. Each entry is short; the list grows as invariants are discovered.

**What each entry needs:**

- A precise statement of the invariant
- Why it matters (what breaks if violated — data leak, financial loss, compliance, user harm)
- What enforces it (tests, DB constraints, middleware, code review policy)
- Known risks — places in the code where a careless change could break it

**The test requirement:** Every invariant in this file should have an enforcing test. If it doesn't, it's not an invariant — it's a hope. Discovering an invariant without writing a test for it is one of the most common ways invariants get silently violated later.

**When to update:**

- When you discover a new invariant (often by fixing a bug caused by its violation — write it down *now* so it doesn't recur)
- When a new enforcement mechanism is added for an existing invariant
- When the set of "known risks" for an invariant changes

**Before writing code near a critical boundary:** Reread the relevant invariants. If your change could violate one, stop and rethink. Explicitly state in your plan how the change preserves the invariants it's adjacent to.

Security-specific invariants (from `references/owasp-security.md`) often belong here too — e.g., "Every endpoint under `/admin/` requires the `admin` role" is a project-specific invariant backed by general OWASP guidance.

See `assets/INVARIANTS.template.md` for the starter template with three full worked examples (tenant isolation, append-only billing events, soft-delete visibility).
