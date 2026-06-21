# DECISIONS.md

Architecture decision records (ADRs). Append-only log of non-obvious decisions and their rationale. New entries at the bottom.

## ADR format

```markdown
## ADR-NNNN: <short title>

**Date:** YYYY-MM-DD
**Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX

### Context

Why this decision needs to be made. The forcing function.

### Decision

What we're doing. Specific enough to act on.

### Consequences

- Pro: ...
- Con: ...

### Alternatives considered

What else we looked at and why we rejected it.
```

## When to write an ADR

- Choosing between real alternatives
- Adopting a new dependency that shapes the codebase
- Deviating from existing convention
- A decision you'd justify to a skeptical reviewer

## When NOT

- Obvious choices with no real alternative
- Implementation details
- Temporary workarounds (use `TODO.md`)

---

<!-- ADRs below. -->
