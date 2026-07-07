# Session Continuity — Surviving Compaction, Crashes, and Cold Starts

Agentic environments lose working context in three ways: the conversation gets **compacted** (auto-summarized when the token limit is hit), the session **crashes**, or a **new session** starts cold with no memory of the last one. A behavioral skill cannot give you memory. What it can do is force two disciplines that make memory loss survivable: **externalize state to durable files continuously**, and **re-derive state and safety rules from those files** at the start of every session and before every risky action.

Read this file whenever a task spans more than a few steps, whenever you're resuming work, or whenever you're about to do something irreversible. It is the highest-leverage part of the skill for long or multi-session work.

Two hard truths everything here is built around:

1. **Conversation memory is volatile.** A fact that lives only in the chat — a decision, a constraint, "the column is `buyer_id`", "this project must never write to prod from dev" — does not survive compaction, a crash, or a new session. If it isn't written to a durable file, assume it will be lost.
2. **This skill itself may not be loaded.** Whether the skill triggers in a given session is probabilistic, and a compacted or cold session may never re-trigger it. So the most critical continuity machinery must *also* live where it loads deterministically — the auto-loaded `CLAUDE.md`, and optionally lifecycle hooks — not only inside this skill. A recovery instruction written only in a skill that didn't load is no instruction at all.

## HANDOFF.md — the living session cursor

`HANDOFF.md` at the project root is a single, always-current snapshot of *where the work is right now*. It is the one file a fresh session (a new instance of you, post-compaction or post-crash) reads first to pick up exactly where the last one left off.

It is not documentation and it is not a backlog — it is a **save-game**. Because it's rewritten constantly, it does not rot the way unmaintained docs do: a stale `HANDOFF.md` is immediately, visibly wrong, which is a feature.

**Required contents — keep it short and current:**

- **Branch + ahead/behind count** — the exact git branch and how many commits ahead/behind its remote.
- **Done** — what's finished and verified this work session (one line each).
- **Pending** — what remains, in order.
- **The single next action** — the one concrete thing the next session should do first. Not a list — the *one* next step.
- **Live-vs-local state** — anything true in the running world but not yet committed: uncommitted files, a running dev server on a port, a migration applied to the local DB but not pushed, an env var set only in this shell, a feature flag flipped.
- **Landmines** — anything the next session could get wrong: a decision that looks arbitrary but isn't, a test that's flaky, a constraint that must hold.

**Update cadence — this is the discipline that makes it work:**

- **After any decision or constraint is established.** The moment a fact exists that the next session must know, write it. A safety rule survives a crash only if it was written to a file — generalize that to *every* decision.
- **Before any risky, long, or irreversible operation.** Externalize state *before* the thing that might kill the session or take you past the token limit, not after.
- **At the end of every work chunk.** Not just at the end of the session — compaction and token-death happen *mid-chunk*, without you stopping. A `HANDOFF.md` updated only "before you stop" doesn't survive a death you didn't choose.

**Delineate it from the other files, or they drift:**

- `HANDOFF.md` = the *volatile cursor* (current branch, live state, the single next action). Overwritten constantly.
- `TODO.md` = the *durable backlog* (everything pending someday, known issues, tech debt). Groomed occasionally.
- `MEMORY.md` / memory files (if you use the productivity memory system) = *long-term knowledge* (people, acronyms, project facts). Rarely changes.

When in doubt: if it answers "what do I do in the next 30 seconds," it's HANDOFF; if it answers "what's on the roadmap," it's TODO; if it answers "what does this term mean," it's MEMORY/DOMAIN.

See `assets/HANDOFF.template.md` for a starter.

## The resume-and-reconcile protocol

Cold starts — a new session, a post-compaction continuation, a restart after a crash — are exactly where you act on **stale assumptions**. The fix is a fixed protocol that runs *before the first substantive action* of any session:

1. **Read the durable state.** `HANDOFF.md` first (the cursor), then `CLAUDE.md` (how to work here) and any memory files. Now you have the *claimed* state.
2. **Reconcile the claim against reality.** The docs describe what *should* be true; verify it *is* true before acting on it. This is the verify-don't-infer rule promoted from *names* to *world state*:
   - `git status` and branch/ahead-count — does it match what `HANDOFF.md` claims?
   - Is the dev server actually up? Hit the real endpoint (`curl .../health`), don't assume.
   - Is the database up and migrated to the expected revision?
   - Are the uncommitted files `HANDOFF.md` mentions actually there?
3. **Surface any mismatch before the first action.** If reality and the docs disagree — the branch moved, a migration isn't applied, the "running" server is down — stop and reconcile it *first*. Acting on a stale handoff is how a resumed session does the wrong thing confidently.

This protocol is worth almost nothing if it only lives in this skill (which may not load on the cold start). Put it in `CLAUDE.md` too — see below.

## Re-derive safety rules at the moment of action

The most dangerous moment is an irreversible, outward-facing action — a deploy, a push, a send, a delete, an external write — taken against a **safety rule the model is only *remembering***. Remembered rules are unreliable: compaction may have summarized the rule away, or a new session never had it.

**The rule:** before any hard-to-reverse or outward-facing action, STOP and *re-read the relevant standing constraints from the durable docs at that moment* — `INVARIANTS.md`, `CLAUDE.md`'s security baseline, `HANDOFF.md`'s live-state notes — then require explicit confirmation. Treat conversation memory as an unreliable cache for safety rules; always re-fetch from durable state. This is the safety-rail equivalent of re-reading the schema before writing a query: the thirty seconds it costs is far cheaper than the live-server mistake it prevents. See `references/guardrails.md`.

## Make it deterministic — CLAUDE.md bootstrap and hooks

Everything above still depends on the model *choosing* to do it. For the parts that matter most, remove the choice.

### CLAUDE.md bootstrap (works in Claude Code and Cowork)

`CLAUDE.md` is auto-injected at the start of every session in both Claude Code and Cowork — it loads whether or not this skill triggers. So it is the right home for the bootstrap that pulls everything else in. Put a block like this near the top of the project's `CLAUDE.md`:

```markdown
## Session start — do this first, every session

1. Read `HANDOFF.md` for current state, then run the resume-and-reconcile
   protocol: check it against `git status`, branch/ahead-count, and whether
   the dev server + DB are actually up. Surface any mismatch before acting.
2. Apply the `senior-engineer-partner` skill for all substantive coding work.
   If it did not auto-load, follow its core rules anyway: verify-don't-infer,
   TDD, no stubs, earned "done", and keep `HANDOFF.md` current.
3. Before any deploy / push / delete / external write, re-read the standing
   constraints in `INVARIANTS.md` and this file's security baseline at that
   moment - do not rely on remembered rules.
```

This is the single highest-leverage change for the "it forgot / the skill didn't load" problem: it makes continuity a property of the auto-loaded file, not of a skill that may be absent.

### Claude Code lifecycle hooks (deterministic, not model discretion)

Hooks run as shell commands at fixed points in the session lifecycle — they fire regardless of what the model decides, the same "enforcement over request" logic as the guardrail hooks. Two are directly relevant. *(Details below reflect the current Claude Code hooks reference; hook names and behavior can change between versions, so confirm against `code.claude.com/docs/en/hooks` for your version before relying on them. Hooks are a Claude Code feature; Cowork behavior may differ depending on its backend.)*

**`SessionStart` — re-inject state on every start, resume, AND after compaction.** SessionStart fires with a `source` of `startup`, `resume`, `clear`, or `compact`. Crucially, its stdout is added to the session context (SessionStart is one of the few events whose stdout Claude actually sees). So a command hook that prints `HANDOFF.md` re-loads your cursor automatically after a compaction or on a fresh session — deterministically, without relying on the model to remember to read it. In `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'echo \"=== HANDOFF.md ===\"; cat HANDOFF.md 2>/dev/null; echo; echo \"=== git ===\"; git status -sb 2>/dev/null'"
          }
        ]
      }
    ]
  }
}
```

For richer control you can instead return JSON with `hookSpecificOutput.additionalContext` (a string injected into context, capped at 10,000 characters) and even `reloadSkills: true`, which makes Claude Code re-scan skill directories after the hook so a freshly-synced skill is available in the *same* session — a direct lever on the "skill didn't load" problem.

**`PreCompact` — snapshot before the context is summarized away.** PreCompact fires before compaction (trigger `manual` or `auto`) and can even block it (exit code 2, or JSON `decision: "block"`). A hook can't make the model write `HANDOFF.md`, but it can deterministically archive the current handoff/transcript so nothing is truly lost, and — by blocking auto-compaction — force a clean manual handoff point:

```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'mkdir -p .claude/handoff-archive && cp HANDOFF.md \".claude/handoff-archive/HANDOFF-$(date +%s).md\" 2>/dev/null; exit 0'"
          }
        ]
      }
    ]
  }
}
```

The pairing is the point: `PreCompact` preserves state going into a compaction, and `SessionStart` (source `compact`) re-injects `HANDOFF.md` coming out of it — so a compaction becomes a structural non-event instead of a memory wipe you hope the model prepared for.
