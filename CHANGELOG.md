# Changelog

All notable changes to this skill are documented here.

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
