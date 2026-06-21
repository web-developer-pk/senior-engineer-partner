# CLAUDE.md

Operating guide for Claude working in this project. Read this first.

## Project summary

<!-- One paragraph: what this project is and what it does. -->

## Stack

- Language(s): <!-- e.g., TypeScript 5.x + Python 3.12 -->
- Framework(s): <!-- e.g., Next.js 14, FastAPI 0.110 -->
- Database: <!-- e.g., PostgreSQL via Prisma -->
- Key libraries: <!-- e.g., React Query, Zod, pytest -->
- Package managers: <!-- e.g., pnpm (frontend), uv (backend) -->

## Directory layout

<!-- Major directories and what each one owns. -->

## Schema source of truth

<!--
Where the database schema is authoritative. Future sessions look here before
writing any DB code. Examples:
- "Migrations in `backend/migrations/`. ORM models in `backend/app/models/`
  reflect the current migration state. When uncertain, run
  `alembic upgrade head --sql` to see the canonical schema."
- "Prisma schema at `prisma/schema.prisma`. Generated types in
  `prisma/generated/`. Do not edit the generated files."
-->

## Conventions

- Naming: <!-- file casing, identifier casing per language -->
- Error handling: <!-- e.g., never swallow exceptions; use AppError class -->
- Logging: <!-- structured logging, fields included, what level for what -->
- Validation: <!-- where validation lives, what library -->
- Tests: TDD; unit colocated; integration in `tests/integration/`

## Verification

Run all before claiming done:

- <!-- lint -->
- <!-- typecheck -->
- <!-- unit tests -->
- <!-- integration tests -->
- <!-- build -->
- <!-- smoke / E2E -->

## Security baseline

<!--
Project-specific security rules. Examples:
- All endpoints under `/api/admin/*` require the `admin` role (enforced by middleware in `auth/admin_guard.ts`).
- All multi-tenant queries must filter by `account_id` (enforced by `tenant_guard` in `db/`).
- Secrets come from `.env`; never commit them. Pre-commit hook blocks common patterns.
-->

## Gotchas

<!-- Project-specific footguns. Add as encountered. -->

## Workflow

- Branch from `main`; PRs require all gates green.
- See `ARCHITECTURE.md` for system design.
- See `DECISIONS.md` for ADRs.
- See `TODO.md` for in-flight work.
- See `DOMAIN.md` for term definitions (if present).
- See `INVARIANTS.md` for rules that must always hold (if present).
