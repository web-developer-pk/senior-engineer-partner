# Tool-Level Guardrails

A rule written in a prompt is a request. A rule enforced at the tool level is a wall. The difference matters because you are a statistical system, not a deterministic one: "never edit files under `/config/prod`" in `CLAUDE.md` reduces the probability of a mistake but does not make it zero. For anything where a mistake is expensive or irreversible, lower the probability to zero by enforcing it where tool calls actually execute — not where they're merely described.

This is the same philosophy as `INVARIANTS.md`: an unenforced rule is a hope. There, the enforcement is a test. Here, it's a hook. Read this file when setting up a project's safety rails, or when a task involves destructive or protected operations and the user wants real protection rather than a polite request.

## Bucket every operation: Always / Ask / Never

Before working in a project, sort its operations into three groups and record the sort in `CLAUDE.md`. The classification drives behavior; the hooks (below) enforce the hard edges.

- **Always-do** — safe, reversible, routine. Read files, run the test suite, run the linter, type-check, build, create a branch, run read-only queries. No need to stop and ask; just do them.
- **Ask-first** — consequential or hard to undo, but legitimate. Database migrations, deleting files, force-pushing, installing new dependencies, editing CI/CD config, anything touching auth/billing/tenancy code, anything writing to a shared or production resource. Stop, state what you're about to do and why, and get explicit sign-off before proceeding. **Batch homogeneous requests:** when several Ask-first actions are of the same low-ambiguity kind (e.g. deleting 15 clearly-dead files), ask for them in one approval rather than once per item — per-item nagging causes confirmation fatigue, and a fatigued human rubber-stamps everything, which defeats the check. But a genuinely destructive or irreversible step (a data migration, a production write, a mass delete with any ambiguity) gets its own explicit call-out — never folded into a batch to be waved through.
- **Never-do** — out of bounds, full stop. Writing to production data stores from a dev session, committing secrets, editing a protected directory, running destructive commands against prod, disabling security controls. These shouldn't depend on your judgment in the moment — they should be blocked at the tool level so they *can't* happen.

The point of writing the buckets down is that the next session inherits the same boundaries instead of re-deriving them under pressure.

## Enforce the hard edges with pre-tool-use hooks

A pre-tool-use hook runs before a tool call executes and can block it. This is how "Never-do" becomes real rather than aspirational. In Claude Code / Cowork, hooks are configured in settings (e.g. `.claude/settings.json`) with a matcher on the tool and a command that inspects the call and exits non-zero to deny it.

The shape of the protection:

- **Match the dangerous tool** (the file-write/edit tool, or the shell tool).
- **Inspect the target** (the path being written, or the command being run).
- **Deny when it hits a protected pattern** — exit non-zero with a message explaining why. The tool call never runs.

Concrete examples to adapt:

**Block writes to a protected directory.** A hook on the write/edit tool that rejects any path matching a protected prefix (e.g. `important-dont-edit/`, `infra/prod/`, `*.pem`, `.env`):

```bash
#!/usr/bin/env bash
# pre-tool-use hook for the file write/edit tool.
# Reads the tool input (path) from the hook payload and denies protected targets.
target="$1"   # wire to your runner's payload (e.g. jq -r '.tool_input.file_path')
case "$target" in
  *important-dont-edit/*|*/infra/prod/*|*.pem|*/.env)
    echo "BLOCKED: $target is a protected path. Edit requires a human." >&2
    exit 1 ;;
esac
exit 0
```

**Block destructive shell commands.** A hook on the shell tool that denies obvious foot-guns regardless of how the prompt framed them:

```bash
#!/usr/bin/env bash
cmd="$1"   # the command string from the hook payload
case "$cmd" in
  *"rm -rf /"*|*"DROP TABLE"*|*"git push --force"*|*"TRUNCATE"*)
    echo "BLOCKED: destructive command requires a human to run it." >&2
    exit 1 ;;
esac
exit 0
```

These are starting patterns, not a finished policy. Tailor the protected paths and command patterns to the project, and wire the argument extraction to how your specific hook runner passes the tool payload (consult the current hooks documentation for the exact payload shape and settings keys — don't assume the field names; verify them, per the no-extrapolation rule).

## Verify the cage actually holds

A hook you set up but never tested is back to being a hope. When you install or first rely on a guardrail for a sensitive operation, confirm it actually fires — once — with a *harmless canary*, not the real dangerous command. Attempt a no-op write to a protected sentinel path (e.g. `touch important-dont-edit/.canary`) and confirm the hook blocks it. Never test a guardrail by attempting the actual destructive action: if the hook is misconfigured, you've just executed the thing you were trying to prevent. Do this once at setup time, not as a per-session ritual.

## How this interacts with the rest of the skill

- The **Ask-first** bucket is the structured version of the skill's existing "push back on destructive actions" posture. Bucketing makes it systematic instead of case-by-case.
- The **Never-do** bucket plus hooks is the enforcement layer for security and data `INVARIANTS.md` entries — e.g. an invariant "dev sessions never write to the prod database" is backed by a hook, not just a sentence.
- Hooks protect against *your own* mistakes too. Setting them up is not an admission of distrust; it's the same reason you write tests — to make a class of error impossible rather than merely unlikely.

When a project handles regulated or high-value data and has no tool-level guardrails, raise that during Clarify. "There's nothing stopping a write to prod here" is an architectural gap worth surfacing before, not after, the first accident.
