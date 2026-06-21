# Incident Mode — Fast-Patch Override

The normal workflow — orient, clarify, plan, TDD, verify every gate, document — is built for getting things *right*. When production is broken right now and users are affected, that cadence is too slow. Incident mode is the explicit, narrow override for that situation: stop the bleeding first, restore the rigor immediately after, and log every shortcut as debt.

This is the only place in the skill where steps get skipped. It is a loan against your normal standards, not forgiveness from them.

## When this applies — and when it doesn't

**Applies:** production is actively broken — an outage, data loss in progress, a security incident, a payment/auth failure hitting real users, a runaway job. The cost of waiting is measured in minutes and real harm.

**Does not apply:** an "urgent" feature, a deadline, a demo tomorrow, a stakeholder who wants it now. Time pressure is not an incident. Those still get the normal workflow — see the size-scaling guidance; urgency changes priority, not rigor. If someone is invoking urgency to skip verification on non-broken code, that's the "just slap it in" pressure the skill exists to resist. Push back.

When you switch into incident mode, say so explicitly: "Treating this as an active incident — switching to fast-patch mode. I'll stop the bleeding, then write up the debt and the proper fix." This makes the override visible and intentional, not a silent lowering of standards.

## The fast-patch cadence

1. **Understand the blast radius before touching anything.** What's broken, for whom, since when. A change made blind during an incident usually makes it worse. Thirty seconds of "what changed" beats a panicked guess.
2. **Read what changed.** Incidents are overwhelmingly caused by a recent change. Read the recent deploy diff / last few commits *before* theorizing. Do not assume what changed — confirm it. (The no-extrapolation rule does not relax under pressure; guessing at a column or a config value while firefighting is how you cause a second incident.)
3. **Smallest safe change to stop the bleeding.** Prefer the minimal intervention that restores service: revert, feature-flag-off, config rollback, scale up, disable the broken path. Not a redesign. The goal is "users are okay again," not "the underlying problem is elegantly solved."
4. **Revert-first bias.** If a recent deploy caused it, reverting to the last known-good state is almost always faster and safer than reasoning out a forward fix under stress. Forward-fix only when revert isn't clean or the bad state is already depended upon (see `existing-codebase.md` -> "Fix forward vs revert").
5. **Verify the specific symptom is gone.** You're not running the full gate suite, but you must confirm *the actual thing that was broken* now works — with a real signal (the failing request now succeeds, the error rate dropped, the job completes). "I reverted it" is not verification; "I reverted it and the 500s stopped" is.
6. **Communicate status honestly throughout.** Say what you did, what you know, and what you don't. "Mitigated by reverting deploy X; root cause not yet understood" is the correct shape of an incident update.

## What does NOT relax, even here

Incident mode skips ceremony, not integrity:

- **No inferred names.** Verify identifiers against source even while firefighting — more so, because the cost of a wrong guess is now another outage.
- **No silent assumptions about cause.** Read the diff; don't theorize blind.
- **No dishonest status.** Never report an incident "resolved" when you've only mitigated it, or when you don't actually know the symptom cleared.
- **No new destructive action without care.** "Fixing" an incident by dropping a table or mass-deleting rows is how a small incident becomes an unrecoverable one. Mitigations that destroy data still warrant a stop-and-confirm.

## The mandatory follow-up — incident mode is a loan

Every shortcut taken in incident mode is debt that must be recorded *the moment the bleeding stops*, before you move on:

- **Log every shortcut to `TODO.md` immediately.** The reverted change that still needs a real fix, the test that should have caught this and didn't, the characterization test you skipped, the monitoring gap that let it run. Write it down now — "later" without a written record means never.
- **The proper fix gets the full workflow.** The real fix — TDD, gates, documentation — is a follow-up task, not part of the incident. Schedule it, don't skip it.
- **Write the invariant.** An incident almost always reveals a "this should never happen" that wasn't enforced. Add it to `INVARIANTS.md` with an enforcing test, so the next session can't recreate the same outage. This is the single highest-value thing to come out of an incident.
- **Postmortem, blameless.** What broke, the timeline, why it wasn't caught, and the concrete prevention items (with owners). Record it — a `DECISIONS.md` entry or a dedicated postmortem doc. An incident you don't learn from, you repeat.

The measure of incident mode done well is not just "service restored" — it's "service restored, debt written down, and the system now can't break the same way again."
