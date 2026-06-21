# Working in an Existing Codebase

The core workflow assumes you can read the ground truth before you write. In a greenfield project that's easy — you wrote it, or there isn't much yet. In a mature codebase (months to years old, written by people who aren't here, with conventions nobody wrote down), "orient before you write" stops being a quick skim and becomes the actual work. This file is the strategy for that.

Read this file when Orient reveals an unfamiliar or mature repo: code you didn't write, a history you weren't part of, conventions that are implicit, or context docs that don't exist yet. The no-extrapolation rule, the no-stubs rule, and TDD still apply unchanged — this file adds the brownfield-specific moves on top.

## First session in a strange codebase — the archaeology workflow

Don't start by reading files at random, and don't start by writing code. Establish ground truth in a fixed order, because each step tells you where to look next:

1. **How do I run it?** Find the entry point(s) and the dev run command. `package.json` scripts, `Makefile`, `Procfile`, `docker-compose.yml`, the README's "getting started." If you can't run it, you can't verify anything you change — say so before going further.
2. **How do I test it?** Find the test command and run it *now*, before touching anything. You need the baseline: does the suite pass, and how long does it take? A red or flaky baseline changes everything downstream (see `tdd-patterns.md` -> "Inherited a sick test suite").
3. **What is the shape?** Map the top-level directories to responsibilities. Don't read every file — read the directory tree, the main config, and one representative file per major area. You're building a mental index, not memorizing.
4. **Where does the data live?** Find the schema source of truth — migrations, ORM models, `schema.prisma`, `schema.sql`. Confirm where it is and that it's current. This is the anchor for every future verify-don't-infer check. Record it in `CLAUDE.md` the moment you find it.
5. **Where are the seams?** Find the boundaries you'll work near: API routes, the service/repository layer, the auth middleware, the module you've been asked to change. Read those files *and the files that call them*.
6. **What does CI enforce?** Read `.github/workflows/` (or equivalent). CI is the authoritative definition of "passing" for this team — match it, don't invent your own gates.

Only after these six do you propose anything. If the task is narrow, scope the archaeology to the relevant slice — but steps 1, 2, and 4 are non-negotiable even for a one-line change, because they're what let you verify it.

## Discovering conventions instead of setting them

In a new project you set conventions. In an existing one you *discover* them, and they may be inconsistent — two eras of code, two authors who disagreed, a half-finished migration from one pattern to another. Match the local neighborhood, don't impose a global ideal.

- **Sample the area you're touching.** Read 3-5 recent files in the same directory or module. Match *their* naming, error handling, test style, and import structure — not the style you'd pick from scratch, and not the style of a different corner of the repo.
- **Local convention beats global convention beats your preference.** When the file you're editing disagrees with the rest of the repo, follow the file. When the repo is internally split, follow the side that's newer or that the task is part of — and note the inconsistency rather than silently picking.
- **When conventions genuinely conflict and it matters, ask.** "This module uses `Result`-style returns; the rest of the app throws. Match the module or the app?" is a real clarifying question. Don't guess.
- **Do not 'fix' conventions you dislike while passing through.** That's collateral damage. If a convention is actively harmful, raise it as a separate proposal — don't fold a style migration into an unrelated task.
- **Record what you learn.** Once you've reverse-engineered the conventions, write them into `CLAUDE.md` so the next session doesn't repeat the archaeology.

## Bootstrapping context docs from cold

The skill wants `CLAUDE.md`, `ARCHITECTURE.md`, etc. to exist. In a mature repo they don't, and writing an accurate `ARCHITECTURE.md` from a cold read is itself a large exploration task — not a side effect of a feature. Trying to produce a complete, authoritative one in a single pass is how you end up writing confident fiction.

Do it incrementally and label confidence honestly:

- **Write a skeleton, not a monument.** Capture what you've actually confirmed this session. Leave explicit `UNKNOWN` / `TODO: verify` markers for what you haven't. A doc that says "auth: not yet mapped" is more useful than one that guesses.
- **Mark verified vs inferred.** If you confirmed the schema source of truth by reading it, state that. If you're recording a guess about why something is the way it is, label it a guess. The whole point of these docs is that the next session can *trust* them — an unlabeled guess poisons that.
- **Grow them as you touch areas.** Each task that explores a new region adds a verified section. Over several sessions the docs become real. This is the only honest way to build them for a codebase you didn't write.
- **For the big architecture doc specifically:** if the task needs a real `ARCHITECTURE.md`, propose a dedicated context-setup session rather than bolting a half-true one onto a feature. Same for `DOMAIN.md` / `INVARIANTS.md` — reverse-engineering the domain language and the invariants of a mature system is its own piece of work.

## Don't break working things

A test gate only catches regressions in behavior that's tested. Established codebases routinely have large untested regions, so the gate is full of holes exactly where the risk is highest. Compensate:

- **Identify blast radius before you change anything.** Grep for callers of the function/endpoint/column you're about to touch. List who depends on it. A change is only as safe as your knowledge of who relies on the current behavior. If the dependents are many or unclear, that's a planning finding, not a detail.
- **Characterize before you change.** Before modifying untested legacy code, pin its current behavior with characterization tests — even behavior that looks wrong. See `tdd-patterns.md` -> "Characterization tests for legacy code." You cannot safely change what you cannot first reproduce.
- **Prefer additive and reversible.** A feature flag, a new code path behind a default-off toggle, or a parallel implementation is safer than mutating a hot path in place. Make risky changes easy to turn off.
- **Smaller blast radius beats cleaner diff.** In legacy code, the change that touches the fewest call sites and preserves the most existing behavior is usually the right one, even if a larger refactor would be tidier. Tidy later, deliberately, with tests.

## Migration and refactor workflow

Touching a 5-year-old module is qualitatively different from writing a new one. Don't rip-and-replace.

- **Strangler fig.** Build the new implementation alongside the old, route a slice of calls to it, expand the slice as confidence grows, then remove the old path once nothing calls it. The old code keeps working the whole time.
- **Parallel run / shadow.** For high-stakes replacements, run new and old in parallel and compare outputs (log divergences, don't act on the new path) before cutting over. Cheap insurance against "looked equivalent, wasn't."
- **Deprecation cycle, not deletion.** Mark the old path deprecated, give callers time/notice to move, *then* delete. Deleting a still-referenced thing because "the new one is better" is collateral damage.
- **One behavior-preserving step at a time.** Refactors are structural change with no behavior change. Make small moves, run tests after each, keep the suite green throughout. If a step needs a behavior change, that's a separate, tested change — not part of the refactor.

## Backward compatibility

When you change something that already exists, other things already depend on the old shape. These are silent failure modes — they don't show up in your local tests:

- **Existing API clients** — mobile apps, third-party integrations, and old web bundles you can't redeploy still call the old contract. Add fields, don't rename or remove them; version the endpoint if the shape must change.
- **In-flight requests and jobs** during deploy — a queue item enqueued by the old code gets processed by the new code. The new code must still understand the old payload.
- **On-disk / serialized formats** — data already written in the old format (DB rows, cached blobs, event logs, files) must still be readable. A migration is part of the change, not a follow-up.
- **Migrations must be safe under rolling deploy** — old and new code run simultaneously during rollout. Add columns nullable/with-default first; do destructive schema changes in a later, separate step after all code stopped using the old shape (expand/contract).
- **Third-party contracts** — webhooks you emit, formats partners consume. Changing these is a coordinated change, not a unilateral one.

Before changing any existing interface, state in the plan who the existing consumers are and how each survives the change. If you can't enumerate the consumers, that's an open question to raise, not an assumption to make.

## Coexisting with other people's work

You are not the only one in this repo. Other developers have in-flight branches, settled debates, and deliberate choices that look arbitrary from the outside.

- **Check the history before reversing a decision.** `git log` / `git blame` on the code you're tempted to "fix." If something looks wrong, it may be load-bearing — written that way on purpose for a reason that isn't visible in the current snapshot. Read the commit message; check `DECISIONS.md`.
- **Don't reopen settled debates.** Tabs-vs-spaces, this-pattern-vs-that — if the codebase has clearly chosen, the choice is made. Re-litigating it via your diff is noise and collateral damage.
- **Assume parallel work exists.** Don't do sweeping cross-cutting renames or reformats that will collide with everyone else's open branches. Keep diffs scoped so they merge cleanly.
- **When you do override a prior choice, do it explicitly.** Say what you're changing, why, and record it as an ADR — don't quietly reverse someone's deliberate decision.

## ADRs in a mature codebase

In a new project, ADRs are forward-looking — you're deciding something now. In a mature one, you're often *reverse-engineering* a decision made implicitly years ago and never written down. That's still worth recording.

- **Document retroactively.** When you finally understand why the system does something non-obvious, write the ADR you wish had existed. Mark it: `Status: Accepted — documented retroactively` with the approximate original date if known.
- **Record the constraint, not a fiction.** If you don't know the real original reasoning, say what you *can* verify ("the code has assumed this since commit X; changing it breaks Y") rather than inventing a clean rationale.
- **Capture decisions at the moment you make a non-obvious choice in legacy code**, too — including the choice to leave something alone. "We kept the old pagination because N clients depend on it" is exactly the kind of thing the next session needs.

## Fix forward vs revert

The skill's default tone is fix-forward, but in an existing system with real users that's not always right. Decide deliberately:

- **Revert when:** a recent change caused the problem, the revert is clean, you're under time pressure, or your confidence in a forward fix is low. Reverting to a known-good state is faster and safer than reasoning forward under stress.
- **Fix forward when:** the broken state has been live long enough that reverting would itself break things built on top of it, the revert isn't clean, or you fully understand the fix and it's small and tested.
- **State which and why.** Don't default into fix-forward because it feels more like "real engineering." Under incident conditions, revert-first is usually correct — see `incident-mode.md`.
