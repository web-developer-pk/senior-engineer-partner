# ARCHITECTURE.md

System design for this project. Read this to understand what talks to what.

## High-level diagram

<!--
ASCII diagram or linked image. Show the major components:
frontend, backend(s), database(s), external services, queues, caches.
-->

```
[ Browser ] ──HTTPS──▶ [ Next.js ] ──HTTP──▶ [ FastAPI ] ──▶ [ Postgres ]
                                                    │
                                                    └──▶ [ Redis ] (cache)
                                                    └──▶ [ Stripe ] (payments)
```

## Components

### Frontend

<!-- Framework, routing model, state management, data fetching, auth handling. -->

### Backend

<!-- Framework, request lifecycle, auth model, persistence layer, background jobs. -->

### Database

<!-- Engine, migrations tool, schema evolution approach. -->

### External integrations

<!-- Third-party services, what they do, how they're authenticated. -->

## Primary data flows

### User registration

<!-- Step-by-step: client → API → DB → response → client state. -->

### <another primary flow>

<!-- ... -->

## Non-functional

- **Performance targets:** <!-- e.g., p95 API latency < 200ms -->
- **Security model:** <!-- e.g., session cookies with SameSite=Lax, CSRF tokens on mutations -->
- **Deployment topology:** <!-- e.g., Vercel (frontend), Fly.io (backend), Supabase (db) -->
- **Observability:** <!-- e.g., logs to Datadog, errors to Sentry -->

## Out of scope

<!-- What's NOT in this codebase and why. Helps future readers know where to look. -->
