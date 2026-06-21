# DOMAIN.md

The ubiquitous language. Every term used in the code, the UI, and conversations about the product means the same thing here. When you catch yourself or a teammate using two words for the same thing — or the same word for two things — update this file.

This file prevents a specific, high-cost class of bug: the bug where two people (or two Claude sessions) use the same word to mean different things, and the code quietly does the wrong thing because each part was written against a different mental model.

## How to use this file

- Before writing code, check the terms you're about to use. If they're defined here, match the definition exactly. If they're not, define them here before writing the code.
- When you're tempted to introduce a synonym, don't. Use the existing term or change the existing term — but don't let both exist.
- Definitions include lifecycle states where relevant. A term isn't fully defined until you know what states it can be in and how it transitions between them.

## Core entities

<!--
One subsection per entity. Keep definitions tight.

Example:
### Account
An organization that has signed up for the product. An account is the billing
entity and the top-level tenant boundary — data in one account is never visible
from another account.

States: `trial`, `active`, `past_due`, `canceled`, `suspended`.
- `trial`: in the initial 14-day trial period, no payment method on file
- `active`: paying, in good standing
- `past_due`: latest invoice failed, in dunning
- `canceled`: user-initiated cancellation, data retained 90 days then purged
- `suspended`: we disabled it (fraud, ToS violation) — data retained indefinitely

Transitions: trial → active (when card added), active → past_due (failed invoice),
past_due → active (successful retry), past_due → canceled (after 3 failed retries),
any → suspended (admin action only).

### User
An individual human with login credentials, belonging to exactly one Account.
Users have Roles (see USER_ROLES.md if present). A User is never shared across
Accounts — the same human using two products has two distinct User records.

### Subscription
A contract between an Account and a Plan over a time period. An Account can have
at most one active Subscription at a time. Canceled subscriptions are retained
for audit / reactivation.
-->

## Distinctions that matter

<!--
Pairs of terms that sound similar but mean different things. This section
pays for itself the first time it prevents a bug.

Example:
- **Cancel vs Suspend**: Cancel is user-initiated, reversible for 90 days.
  Suspend is admin-initiated, reversible only by admin action.
- **User vs Account Owner**: Every Account has exactly one Owner (a User
  with the Owner role). "The user" in conversation almost always means a
  regular User, not the Owner specifically — say "Owner" when you mean it.
- **Plan vs Subscription**: Plan is the template (Pro Monthly, $29/mo).
  Subscription is the instance (Account X is on Plan Y until date Z).
- **Deleted vs Purged**: Deleted means soft-deleted (row retained with
  `deleted_at` timestamp). Purged means gone from the database entirely.
  GDPR "delete my data" requests trigger purge, not delete.
-->

## Forbidden terms

<!--
Words that sound useful but are banned because they're ambiguous. If you're
tempted to use one, pick a precise alternative.

Example:
- "Customer" — ambiguous between Account and User. Say which.
- "Admin" — ambiguous between our admins and an account's Owner role. Use
  "Anthropic admin" or "Account Owner" specifically.
- "Premium" — we have Starter, Pro, and Enterprise plans. None is called
  Premium. Don't use it in code, UI, or docs.
-->
