# Senior Engineer Partner

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) that makes Claude behave like a rigorous senior software engineer and architect — not an eager junior that produces plausible-looking code and reports "done" when it isn't.

It was built for **Cowork** and other agentic coding environments that have no memory between sessions, but works anywhere Claude supports skills.

## The problem it solves

Agentic coding assistants have a recurring failure class: code that *looks* finished but isn't. Stubbed functions reported as complete. Tests written but never run. References to database columns or API fields that don't exist because they were guessed from a pattern. Happy-path-only implementations with no error handling. Work that passes in isolation but was never integrated.

This skill is the behavioral antidote. It makes Claude think before it types, verify before it assumes, implement completely, and **earn** the word "done" through verifiable evidence.

## Core principles

- **Trust contract** — four ranked failure modes to avoid: lying about completion, inventing names from patterns, working superficially, and silent assumptions.
- **Verify, don't infer** — a hard rule against extrapolating identifiers (DB columns, API fields, env vars, paths, signatures). If you can't see a name, you don't know it exists — read the migration, the schema, the actual file.
- **Size-scaled rigor** — a 30-line button doesn't get the ceremony of a new service, but the non-negotiables (no stubs, no inferred names, tests run, gates pass, honest report) hold at every size.
- **Simplicity first** — the minimum code that fully solves the problem; no speculative abstractions, no gold-plating.
- **TDD by default** — failing test first, confirmed red, then the implementation.
- **OWASP-aware** — Top 10 and API Top 10 walked during planning and verification for security-adjacent changes.
- **Earned "done"** — the word requires every verification gate (lint, typecheck, tests, build, smoke test) to have run and passed, with the output shown.

## Cross-session memory

Because agentic environments forget between sessions, the skill persists context in seven project-root Markdown files, divided into two tiers:

**Engineering tier** — `CLAUDE.md` (how to work here), `ARCHITECTURE.md` (system design), `DECISIONS.md` (an append-only ADR log), `TODO.md` (in-flight and deferred work).

**Product / domain tier** — `PRODUCT.md` (what the product is and who it's for), `DOMAIN.md` (the ubiquitous language, to kill same-word-different-meaning bugs), `INVARIANTS.md` (the tested "never, ever" list).

Starter templates for all seven live in [`skill/senior-engineer-partner/assets/`](skill/senior-engineer-partner/assets).

## What's in the box

```
skill/senior-engineer-partner/
├── SKILL.md                      # the behavioral spec
├── references/
│   ├── stack-detection.md        # stack -> verification commands; DB introspection
│   ├── tdd-patterns.md           # TDD patterns, incl. characterization tests & sick suites
│   ├── owasp-security.md         # OWASP Top 10 + API Top 10 + data-handling/privacy
│   ├── context-docs.md           # guidance for the seven context files
│   ├── existing-codebase.md      # brownfield: archaeology, migration, backward-compat
│   ├── performance.md            # performance as a first-class verification gate
│   ├── incident-mode.md          # fast-patch override for active production incidents
│   └── guardrails.md             # Always/Ask/Never buckets + pre-tool-use hooks
└── assets/                       # starter templates for the 7 context docs
```

## Install

### Cowork
Download [`senior-engineer-partner.skill`](senior-engineer-partner.skill) from this repo (or from a [Release](../../releases)), **rename the `.skill` extension to `.zip`** (Cowork's uploader expects a ZIP), then upload it in Settings → Capabilities.

### Claude Code
Copy the skill into your skills directory:

```bash
git clone https://github.com/web-developer-pk/senior-engineer-partner.git
cp -r senior-engineer-partner/skill/senior-engineer-partner ~/.claude/skills/
```

Claude Code discovers it automatically; it activates when a task matches the skill's description.

### claude.ai
Create a Skill in your project/settings and paste in the contents of `SKILL.md`. The skill is most powerful where Claude can also read the `references/` files, so prefer surfaces that support bundled resources.

## When it triggers

Substantive coding work: building features, fixing bugs, refactoring, adding tests, wiring APIs, scaffolding services, modifying existing codebases.

It deliberately does **not** trigger for pure explanatory questions ("what does this regex do?"), one-off shell commands, single-line snippets, documentation-only requests, or casual throwaway prototyping.

## Credits and influences

This skill is an original synthesis based on my personal experiences, but it also stands on ideas from others:

- **Andrej Karpathy** — the spec / verifier / environment framing and the "you can outsource thinking, but not understanding" principle that anchors the skill.
- **The Claude Code / Cowork community** — the `CLAUDE.md` convention and the broader practice of persisting working context to Markdown.
- **Google Gemini** — an independent review pass that contributed the anti-thrash circuit breaker, headless-E2E graceful degradation, permission batching, and guardrail self-test ideas.
- **Anthropic** — the [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) format and tooling.

Referenced third-party material (a cheatsheet infographic and a video transcript on the Karpathy method) informed the design but is **not** redistributed here — please seek out the original creators' work.

## License

[MIT](LICENSE) © 2026 Muhammad Omar
