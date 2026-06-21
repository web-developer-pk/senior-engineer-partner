# PRODUCT.md

What this product is, who it's for, and the core problems it solves. The tiebreaker document — when requirements conflict, reread this.

Keep this to one screen. If it grows past that, you're probably mixing in feature specs that belong elsewhere.

## What is it

<!--
One or two sentences. The elevator pitch. If you can't say it in two sentences,
you don't understand the product well enough yet.

Example:
"A subscription billing platform for independent SaaS businesses. It handles
plans, upgrades, dunning, and tax calculation so founders don't have to build
billing themselves."
-->

## Who is it for

<!--
The specific users. Not "everyone" — that's never true. Be concrete about
role, company size, technical level, and the moment they'd reach for your
product.

Example:
- Primary: technical founders of SaaS companies, 1-20 employees, between
  $0 and $1M ARR, who have outgrown Stripe Checkout but don't want to build
  a full billing stack.
- Secondary: finance teams at the above companies who need reliable revenue
  reports.
- NOT for: enterprise finance orgs with custom contract requirements. They
  should buy Zuora.
-->

## Core jobs the product does

<!--
The 3-5 things this product is designed to do well. If something isn't on
this list, it's out of scope — or you need to update this list.

Example:
1. Price plans with flexible trial / billing / discount rules
2. Handle upgrades and downgrades with correct proration
3. Retry failed payments and manage dunning communications
4. Calculate and remit sales tax / VAT
5. Produce revenue reports finance teams can trust
-->

## Explicitly out of scope

<!--
What the product deliberately does NOT do. Just as important as what it does,
because this is where feature creep gets caught.

Example:
- Custom enterprise contracts (use a CPQ tool)
- Payment processing itself (we sit on top of Stripe / Adyen)
- Customer support ticketing
-->

## Non-functional commitments

<!--
Product-level promises: uptime target, data durability, geographic
availability, compliance posture. These shape architecture decisions.

Example:
- 99.9% uptime SLA for production workloads
- Data residency: US and EU regions available
- SOC 2 Type II compliance
- No customer data used for ML training
-->

## Success criteria

<!--
How do we know the product is working? Keep this honest — vanity metrics
don't belong here.

Example:
- Monthly active paying customers
- Time-to-first-successful-charge after signup (target: under 15 minutes)
- Involuntary churn rate (target: under 2% monthly)
-->
