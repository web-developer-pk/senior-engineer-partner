# OWASP Security Reference

A working checklist for building and verifying secure web applications. Covers OWASP Top 10 (2021) for web app risks and OWASP API Security Top 10 (2023) for API-specific risks.

Use this file in two places in the workflow:

1. **During Plan** — for any task that touches auth, input handling, data access, or external integrations, scan the relevant section here and call out which risks apply to the change. Make controls part of the plan, not an afterthought.
2. **During Verify** — before claiming a security-adjacent task is done, walk the relevant checklists and confirm each control is in place.

This file is not a substitute for a formal security review on high-risk systems. For systems handling PII, payments, healthcare, or other regulated data, supplement with OWASP ASVS (Application Security Verification Standard) — see end of file.

---

## OWASP Top 10 (2021) — web app risks

### A01 — Broken Access Control

Most common and most damaging. Every authorization check missed is a vulnerability.

**Controls:**

- Enforce authorization on **every** non-public endpoint, server-side. Client-side checks are UX, not security.
- Default-deny: a missing authorization decision means deny, not allow.
- Authorize at the **resource** level, not just the route level: "can this user read this specific record" — not just "can this user reach this endpoint."
- For multi-tenant systems: every query that reads tenant-scoped tables must filter by tenant ID. This belongs in `INVARIANTS.md` with an enforcing test.
- Direct object references (`/users/123`) must verify the requester is authorized for that specific id — not just that the route requires auth.
- Forbid privilege escalation paths: a regular user must not be able to assign themselves admin role via API parameters, profile fields, or any other request shape.
- Log authorization failures, not just successes. Repeated failures from one principal warrant alerting.

**Tests to write:**

- For each role, a test confirming forbidden actions return 403.
- For each tenant-scoped resource, a test confirming cross-tenant access returns 404 (not 403 — never confirm existence to an unauthorized party).
- Tests for the IDOR (insecure direct object reference) class: user A creates resource X, user B tries every CRUD verb against X.

### A02 — Cryptographic Failures

**Controls:**

- HTTPS everywhere. HTTP redirects only at the edge, never inside the app.
- Passwords: hashed with a memory-hard algorithm (argon2id preferred, bcrypt acceptable). Never MD5, SHA-1, SHA-256, or any unsalted hash.
- Secrets (API keys, DB passwords, signing keys) come from environment or a secret manager, never from source. Pre-commit hooks or scanning to prevent commits.
- Encryption at rest for sensitive data — application-level for highly sensitive fields (PII, financial), disk-level for the rest.
- TLS configuration: TLS 1.2 minimum (1.3 preferred), strong ciphers only, HSTS header set, secure cookies.
- Tokens (session IDs, password reset links, API tokens) generated from a CSPRNG, sufficient entropy (128+ bits), short-lived where possible.
- Random number generation: use the language's CSPRNG (`secrets` in Python, `crypto.randomBytes` in Node). Never `Math.random()` for anything security-related.

**Tests / checks:**

- Static analysis: no hardcoded secrets in any committed file. Use a tool (gitleaks, trufflehog) in CI.
- Verify TLS configuration with a scanner (testssl.sh, SSL Labs) at deploy time.

### A03 — Injection

SQL, NoSQL, OS command, LDAP, ORM injection — all the same root cause: untrusted input concatenated into a command.

**Controls:**

- Parameterized queries everywhere. Never construct SQL via string concatenation, format strings, f-strings, or template literals. Use the ORM's parameter binding or the driver's prepared statements.
- This applies to **identifiers too**: if you must interpolate a table or column name (which you almost never should), validate against an allowlist — not regex, not escape.
- Shell commands: prefer language-native APIs (`os.path`, `pathlib`) over shelling out. When shelling out is unavoidable, use array-form invocation that doesn't go through a shell — never `shell=True`, never string concatenation into `system()`.
- Server-side template injection: never pass user input as a template; pass it as template data.
- Output encoding for HTML (prevents stored XSS at render time even if input validation missed it).

**Tests:**

- Negative tests for each input field: feed it SQL meta-characters (`' OR 1=1 --`), shell meta-characters (`; rm -rf`), template syntax (`{{ 7*7 }}`) — confirm the response is a normal validation error, not an executed payload.

### A04 — Insecure Design

Categories of design flaw that no amount of patching can fix.

**Controls:**

- Threat-model significant new features before building. What's the worst that happens if input X is malicious? If user Y impersonates Z?
- Rate limit by user, by IP, and by endpoint where appropriate. Especially: login, password reset, signup, anything that costs money or sends email/SMS.
- Defense in depth: do not rely on a single layer. Auth + authz + tenant filter + DB constraint is better than any one of those alone.
- For business logic flows (checkout, transfer, refund), enumerate the abuse cases and add controls for each. Negative quantities, replay, race conditions, order-of-operations attacks.

This category often surfaces as an `INVARIANTS.md` entry — design-level rules that the rest of the code must respect.

### A05 — Security Misconfiguration

**Controls:**

- No default credentials, ever. No admin/admin, no example accounts in production.
- Disable verbose error messages in production. Stack traces, SQL errors, framework debug pages — none should reach the client.
- Disable unused features, modules, and ports.
- Security headers: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, `Permissions-Policy`. Generate via middleware, not per-route.
- CORS: explicit allowlist of origins. Never `Access-Control-Allow-Origin: *` for endpoints that use credentials.
- Cookies: `Secure`, `HttpOnly`, `SameSite` (Lax or Strict depending on flow). Path and Domain scoped tightly.
- Patch promptly. CVE feeds for dependencies; `npm audit`, `pip-audit`, `cargo audit` in CI.

### A06 — Vulnerable and Outdated Components

**Controls:**

- Lock dependency versions (lockfiles). Renovate or Dependabot to surface updates.
- CI runs dependency scanning (`npm audit`, `pip-audit`, `cargo audit`, GitHub Dependabot alerts) — high/critical findings block merge.
- Track which versions you're running in `CLAUDE.md` or a dedicated `DEPENDENCIES.md` so audits are quick.
- Periodic SBOM (software bill of materials) generation for compliance-heavy contexts.

### A07 — Identification and Authentication Failures

**Controls:**

- Reject weak passwords (length, breach corpus check via Have I Been Pwned API or local copy).
- Multi-factor authentication available for all accounts; required for admin and high-risk roles.
- Account lockout / progressive delays after failed login attempts (per account and per IP).
- Session tokens: rotate on login, invalidate on logout, expire after inactivity.
- Password reset flows: single-use tokens, short expiry, require re-authentication for sensitive changes.
- No username enumeration: identical responses for "user does not exist" and "wrong password."
- No session fixation: issue a new session ID after authentication, never accept session IDs from URL parameters.

### A08 — Software and Data Integrity Failures

**Controls:**

- Verify integrity of dependencies (lockfiles with hashes, signed releases where available).
- Don't deserialize untrusted data. If you must, use formats that don't execute code on deserialize (JSON over pickle, JSON over PHP serialize, JSON over Java native serialization).
- CI/CD pipelines themselves are a target: protect signing keys, restrict who can modify pipeline definitions, log all production deploys.
- Auto-update from untrusted sources is dangerous. Pin versions and review changelogs.

### A09 — Security Logging and Monitoring Failures

**Controls:**

- Log auth events (success and failure), authorization denials, input validation failures, high-value operations (delete, transfer, admin actions).
- Logs include: timestamp, principal (user id), source IP, action, resource, outcome.
- Logs do **not** include: passwords, tokens, full credit card numbers, full SSNs, raw request bodies for sensitive endpoints.
- Logs are tamper-resistant (append-only, shipped off-host promptly).
- Alerting on suspicious patterns: many auth failures, sudden privilege escalation, unusual data volume access.

### A10 — Server-Side Request Forgery (SSRF)

**Controls:**

- If user input determines a URL the server fetches, validate against an allowlist of hosts or schemes.
- Block requests to private IP ranges (RFC1918, link-local, loopback) by default unless explicitly required.
- Block redirects through DNS rebinding by resolving the host and pinning the connection.
- Cloud metadata endpoints (`169.254.169.254`) must be blocked at the application level, not just network — application-level SSRF can reach metadata even when network rules allow it.

---

## OWASP API Security Top 10 (2023)

If the project exposes an API (REST, GraphQL, etc.), these apply on top of the web Top 10.

### API1 — Broken Object Level Authorization (BOLA)

The single most common API vulnerability. Same root cause as A01 — authorization checked at the endpoint level but not at the object level.

**Rule:** Every endpoint that accepts a resource ID must verify the requester is authorized for that specific id, in addition to being authenticated.

### API2 — Broken Authentication

Same controls as A07, with extra attention to: API keys (rotation, scoping, transport security), JWT handling (signature verification, algorithm pinning, audience and expiry checks).

### API3 — Broken Object Property Level Authorization

Includes mass assignment (`PATCH /users/me` accepting `{role: "admin"}`) and excessive data exposure (returning fields the requester shouldn't see).

**Controls:**

- Allowlist writable fields per endpoint, per role. Reject unexpected fields rather than silently dropping them — silent dropping masks attack attempts.
- Per-role response schemas. The endpoint that returns a user object returns different fields for self vs admin vs public.

### API4 — Unrestricted Resource Consumption

**Controls:**

- Pagination is mandatory on any endpoint that can return more than one row. Maximum page size, not just default.
- Query complexity limits for GraphQL (depth, breadth, total node count).
- Request body size limits.
- Per-endpoint timeouts.
- Rate limits per IP, per user, per API key.

### API5 — Broken Function Level Authorization

Same as A01 applied to function-level access (admin-only endpoints reachable by regular users).

**Controls:**

- Group endpoints by required role; enforce role at the group level via middleware, not per-handler (per-handler is too easy to forget).
- Negative tests: for each admin endpoint, test that a regular user gets 403.

### API6 — Unrestricted Access to Sensitive Business Flows

Some flows are sensitive even when functionally authorized: bulk signup, password reset, refund, item purchase. Abuse looks like normal use, just at scale.

**Controls:**

- Per-flow rate limits beyond standard rate limiting.
- CAPTCHA or proof-of-work for high-abuse flows.
- Anomaly detection on volume and pattern.

### API7 — Server-Side Request Forgery

Same as A10.

### API8 — Security Misconfiguration

Same as A05, with API-specific items: API documentation should not be publicly accessible in production unless intended; OPTIONS responses should not leak unintended endpoints.

### API9 — Improper Inventory Management

**Controls:**

- Maintain an authoritative inventory of all API endpoints, versions, and their status (active, deprecated, sunset).
- Old API versions are attack surface. Sunset them on a schedule.
- Non-production environments (staging, dev) reachable from the internet are common breach paths — protect or remove.

### API10 — Unsafe Consumption of APIs

**Controls:**

- Treat responses from third-party APIs as untrusted input. Validate before using.
- TLS verification on all outbound calls — never disable certificate validation.
- Timeouts and retries with circuit breakers — a slow upstream shouldn't take you down.

---

## Data handling and privacy (beyond the Top 10)

The OWASP Top 10 is about keeping attackers out. This section is about handling the data you legitimately hold responsibly — privacy, retention, and accidental exposure. These failures don't look like a breach; they look like a normal feature that quietly mishandles personal data. They carry real legal and trust consequences (GDPR, CCPA, HIPAA, and similar), and they're easy to introduce without noticing.

For a security-adjacent change that touches user or customer data, walk this list during Plan and Verify alongside the relevant Top 10 categories.

**Know what data you're touching.** Classify it before you write code. Is any of it PII (name, email, IP, device ID), sensitive PII (government ID, financial, health), or regulated (PHI under HIPAA, cardholder data under PCI)? The classification sets the bar — sensitive and regulated data warrant stricter controls and often ASVS-level rigor. If you can't tell whether a field is personal data, treat it as if it is.

**Minimize.** Collect, store, and return the least data that does the job. Don't add a column "because we might want it." Don't return fields the client doesn't use (this overlaps API3 — excessive data exposure). Less data held is less data to leak, log, or have to delete later.

**Respect retention and deletion.**

- Personal data has a lifespan. If the project has a retention policy, honor it; if it doesn't and you're adding a new store of personal data, raise the question — "how long do we keep this, and what deletes it?" is a Plan-time concern, not an afterthought.
- Distinguish **delete from purge** (see `DOMAIN.md` if present): a GDPR/CCPA "delete my data" request usually means *purge* — gone from the database, backups handled per policy — not a soft-delete flag. Implementing "delete my account" as `deleted_at = now()` while the rows live on forever is a compliance gap dressed as a feature.
- A deletion request must reach the denormalized copies too: caches, search indexes, analytics stores, exports, logs. Personal data that survives in a search index after "deletion" is still a violation.

**Don't leak data into places it doesn't belong.**

- **Logs and error messages.** Never log raw request bodies for endpoints that carry credentials or PII, full card numbers, government IDs, tokens, or passwords. This overlaps A09 but is broader than security logging — it includes accidental `print`/`console.log` debugging left in, and verbose error messages that echo personal data back. Scrub before you log.
- **Analytics and third-party tools.** Sending PII to an analytics provider, error tracker, or session-replay tool is a disclosure to a third party. Confirm there's a basis for it; redact what doesn't need to go.
- **LLM prompts and external APIs.** If the system sends data to an LLM or any third-party API, personal or regulated data going into that prompt/payload is a disclosure to that vendor. Don't send it without a lawful basis and the user's awareness — and never send secrets. Treat the boundary to any external service as a place where data minimization applies.
- **Test fixtures and seed data.** Don't seed dev/test environments with real production personal data. Use synthetic data. Copying a prod dump into staging is one of the most common quiet PII leaks.

**Surface the lawful-basis question for regulated contexts.** If the project handles health, financial, children's, or other specially-regulated data, the *legal* basis and consent model is an architectural constraint, not a detail. Raise it during Clarify — "what's our basis for processing this, and is consent recorded?" — the same way you'd raise ASVS escalation. Building the feature first and retrofitting compliance later is how the expensive mistakes happen.

**Recording in INVARIANTS.md.** Hard data rules belong with the other invariants, each with an enforcing test: "PII fields are never written to application logs," "account deletion purges within 30 days across all stores," "no real production data in non-prod environments." A privacy rule without a test is a hope, like any other invariant.

## How to apply this file in the workflow

### During Plan

For a security-adjacent change, name the OWASP categories that apply and the specific controls the plan includes. Example planning snippet:

> "This change adds a public endpoint for password reset. Applicable categories: A01 (no enumeration), A02 (token from CSPRNG, 256-bit, 15-minute expiry), A04 (rate limit per email and per IP), A07 (single-use, re-auth required for password change), A09 (log all reset attempts). Tests will cover each."

### During Verify

Before claiming done, walk the relevant categories. For each, confirm the control is in place AND has a test. A category without a test is a category that will regress.

### Recording in INVARIANTS.md

Security rules that must always hold belong in `INVARIANTS.md`, not just in this file. This file is the reference; `INVARIANTS.md` is the project-specific enforcement. Example: "Every endpoint under `/admin/` requires the `admin` role" is an invariant — write it down, write a test, link the test from the invariant entry.

### When to escalate to ASVS

For systems with regulated data (PHI, PCI, financial) or high-value targets, the Top 10 is a starting floor, not a sufficient bar. Move to OWASP ASVS Level 2 or 3 and treat it as a structured checklist. If the user's project falls in this category and ASVS is not being used, raise that as a question during Clarify — it's an architectural concern that affects how the system must be built.
