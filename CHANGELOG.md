# Changelog

All notable changes to this skill are documented here.

## [1.1.0] — 2026-07-08

Session-continuity release — surviving compaction, crashes, and cold starts.

### Added
- `HANDOFF.md` — a living session cursor (current branch/ahead-count, done,
  pending, the single next action, live-vs-local state, landmines) as the
  eighth context document, with a starter template in `assets/`.
- `references/session-continuity.md` — the resume-and-reconcile protocol
  (verify durable-doc claims against `git status` / health / DB before acting),
  the `CLAUDE.md` bootstrap (so continuity survives even when the skill does
  not load), and verified Claude Code `SessionStart` / `PreCompact` hook
  configs for re-injecting and archiving state around compaction.

### Changed
- Orient step now runs the resume-and-reconcile protocol on cold/resumed starts;
  Document step now mandates keeping `HANDOFF.md` current at every chunk boundary.
- `guardrails.md` adds "re-derive the rule at the moment of action" — re-read
  standing constraints from durable docs before any irreversible action rather
  than trusting recalled rules.
- `CLAUDE.md` template gains a session-start bootstrap block.
- Skill description tuned to trigger more reliably and to prompt reloading after
  compaction or resume.

## [1.0.0] — 2026-06-21

Initial public release.

### Core
- Trust contract (four ranked failure modes) and the verify-don't-infer
  no-extrapolation rule.
- Size-scaled workflow (Small / Medium / Large) over a seven-step core
  loop: Orient, Clarify, Plan, Execute, Verify, Definition-of-done, Document.
- TDD-first execution discipline; earned "done" gated on real verification.
- Simplicity-first principle (no over-building) alongside the no-stubs rule.
- Seven project-root context documents as a cross-session memory system:
  CLAUDE, ARCHITECTURE, DECISIONS, TODO, PRODUCT, DOMAIN, INVARIANTS.

### References
- `stack-detection.md`, `tdd-patterns.md`, `owasp-security.md`,
  `context-docs.md`.
- `existing-codebase.md` — brownfield strategy (archaeology, convention
  discovery, blast-radius, migration, backward compatibility, ADR-archaeology,
  fix-forward-vs-revert).
- `performance.md` — performance as a first-class verification gate.
- `incident-mode.md` — fast-patch override for active production incidents.
- `guardrails.md` — Always/Ask/Never bucketing and pre-tool-use hooks.

### Behavioral hardening
- Anti-thrash circuit breaker in Verify (hypothesis-based, not a blind retry
  counter).
- Headless E2E graceful-degradation guidance.
- Chunk-don't-truncate rule for output-limit-bound implementations.
- Selective context-doc reading to limit attention degradation.
- Permission batching and a guardrail self-test ("verify the cage holds").
