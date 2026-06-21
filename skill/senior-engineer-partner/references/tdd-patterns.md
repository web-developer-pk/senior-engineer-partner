# TDD Patterns for Full-Stack Web Apps

Concrete patterns for common scenarios. Adapt to the specific project.

## The red–green–refactor loop, explicitly

For every unit of new behavior:

1. **Red:** Write the test. Run it. Confirm it fails for the expected reason. A test that fails for the wrong reason (import error, syntax error on an unrelated line) is not a valid red.
2. **Green:** Write the minimum code to pass. Run the test. Confirm it passes. Do not write extra code "while you're there."
3. **Refactor:** Clean up. Run the test again.

If you skip the red step, you haven't done TDD — you've written code and hoped the test is meaningful. Red proves the test can fail.

## Pattern: New API endpoint (backend)

1. **First, verify the schema.** If the endpoint touches a table, open the migration or model. Confirm exact column names. Do not extrapolate.
2. Write an integration test that hits the endpoint as a client would (TestClient, not mocks). Assert on status, response shape, and DB side effects.
3. Write unit tests for new business logic — one test per behavior branch.
4. Run all new tests. Confirm they fail for the expected reason.
5. Implement the route handler, validation, and business logic. Run tests after each meaningful step.
6. Add error-path tests: bad input (400/422), missing auth (401), forbidden (403), not found (404), conflict (409), server errors (500).
7. Add authorization tests: cross-tenant access (must return 404, not 403), wrong-role access (403).
8. Run the full suite. Confirm no regressions.

**Avoid:**
- Happy-path-only tests
- Mocking the database (use a test DB)
- Skipping validation tests
- Assuming column names — verify against schema first

## Pattern: New UI component (frontend)

1. Write the rendering test: does it render with required props? Show expected content?
2. Interaction tests: click, type, submit. Use Testing Library's user-event, not direct event firing.
3. State tests: loading, success, error, empty.
4. Run tests, confirm they fail (component doesn't exist).
5. Implement. Run tests after each meaningful step.
6. If it fetches data, test loading and error paths. Mock at the network boundary (MSW), not at the component level.

**Avoid:**
- Implementation-detail tests (internal state, class names)
- Unreviewed snapshot tests
- Forgetting a11y — querying by role gives accessibility pressure for free

## Pattern: Fixing a bug

1. **Reproduce in a test first.** Run it, confirm it fails with the bug's symptom.
2. This test is your proof the bug exists and your proof the fix works.
3. Implement the fix. Run the test. Confirm it passes.
4. Run the full suite. Confirm no regressions.
5. The test stays as a regression test.

If you can't reproduce the bug in a test, you don't understand it well enough to fix it. Keep investigating.

**Bonus:** If the bug was caused by a previously unwritten invariant, add it to `INVARIANTS.md` with this test linked as the enforcer.

## Pattern: Refactor

Refactoring is structural change without behavior change. Tests are the safety net.

1. Run the full suite first. It must be green. If not, fix that first — you can't safely refactor on a broken base.
2. If coverage is sparse in the area, add tests first. You can't safely refactor untested code.
3. Make small changes. Run tests after each. If tests break, behavior changed — revert or explicitly update the test.
4. At the end, green and behavior unchanged.

## Pattern: Integration points

Most "worked in isolation, broke in integration" bugs live here.

- **FE ↔ BE:** at least one test hits the real backend (not a mocked response). E2E tools (Playwright, Cypress) are ideal.
- **Service ↔ service:** contract tests or integration tests that exercise the real wire format. Don't rely on "both sides have unit tests" — both can agree with each other and both be wrong.
- **Service ↔ DB:** real test database, not a mock. ORMs have subtle behaviors only a real DB surfaces.

## Pattern: Tests exist but aren't run

A classic failure mode. Audit explicitly:

- Before claiming done, run the entire suite. Check the test count is what you expect.
- If CI is configured, verify the CI config includes your test file's path.
- If tests are conditionally skipped (`@pytest.mark.skip`, `it.skip`), investigate why — skips are bugs waiting.

## Pattern: Slow tests

Slow tests get skipped; skipped tests miss bugs.

- Identify slow tests and report them to the user. Don't silently stop running them.
- Propose tiering: fast unit tests on every change; slower integration/E2E at phase boundaries or in CI.
- Never `skip` to "solve" slowness without explicit agreement.

## Pattern: Security-adjacent code

When the change touches auth, input handling, data access, or external interfaces:

1. Walk `references/owasp-security.md` for applicable categories.
2. Write tests for each applicable control — not just happy path. Authorization tests across roles and tenants. Negative tests for injection payloads. Rate-limit tests.
3. Verify the control with the test, not just by reading the code.

## Pattern: Characterization tests for legacy code

You've been asked to change code that has no tests and that you didn't write. You cannot safely change behavior you can't reproduce, and you can't trust that the current behavior is "correct" — but it is the behavior real users and callers depend on right now. Pin it first.

1. **Capture current behavior as-is.** Write tests that assert what the code *actually does* today, for a spread of representative inputs — including weird edge cases and outputs that look wrong. You are not asserting correctness; you are photographing the status quo.
2. **Run them green against the unchanged code.** They should pass immediately, because they describe what the code already does. (This is the opposite of normal TDD's red step — here, green-on-current is the proof you've characterized accurately.)
3. **For output-heavy code, use golden-master.** Snapshot the current output for a batch of real inputs and assert future runs match. Approval/golden-master testing is the fast way to pin a large surface you don't fully understand yet.
4. **Now change the code.** The characterization tests are your safety net: any test that flips from pass to fail marks a behavior you changed. Confirm each flip is intended. Unintended flips are regressions — fix them or revert.
5. **As you understand each behavior, upgrade the test.** Replace "this is what it does" with "this is what it should do" once you actually know — and fix the code if those differ, as a separate, deliberate change.

If a behavior you've pinned is genuinely a bug, don't silently "fix" it inside an unrelated change — surface it. Some callers may depend on the bug.

## Pattern: Inherited a sick test suite

Real codebases have test suites that are slow, flaky, partially skipped, or red on arrival. Pretending otherwise leads to the worst outcome: claiming your change is "verified" against a suite that was never trustworthy.

1. **Establish the baseline first, before any change.** Run the full suite on the untouched code. Record exactly what passes, what fails, what's skipped, and how long it takes. This is the reference point for everything you do next.
2. **Never add to the red.** If the suite is already failing, your job is not to make it worse. A new failure introduced by your change must be distinguishable from the pre-existing failures — which is only possible because you captured the baseline in step 1.
3. **Quarantine, don't delete.** A flaky or broken test still encodes intent. Mark it skipped *with a reason and a `TODO.md` entry* rather than deleting it. Deleting tests to make the bar green is the inverse of the skill — it manufactures a fake "done."
4. **Don't "fix" flakiness by loosening assertions** without understanding the flake. A flaky test is usually pointing at a real race, time-dependence, or order-dependence. Investigate before quarantining; record what you found.
5. **Report the suite's health honestly.** If you verified your change against a suite that's 30% skipped, say so: "My new tests pass and the change is green, but the existing suite has 14 pre-existing failures and 22 skips unrelated to this change — verification is partial." Don't let a sick suite masquerade as a clean bill of health.
6. **Treat the suite itself as tech debt worth flagging.** A suite you can't trust undermines every future "done." Propose a dedicated task to restore it — separate from feature work.

## Coverage

Coverage is a weak signal but a useful canary. If you add a feature and coverage drops, you've under-tested. Report coverage numbers when configured.

100% coverage is not the goal. Coverage of every meaningful branch — including error paths and edge cases — is.
