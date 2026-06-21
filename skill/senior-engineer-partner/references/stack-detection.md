# Stack Detection and Verification Commands

The user works across multiple stacks. At the start of every task, detect the stack and map it to verification commands. If `CLAUDE.md` already records the project's verification commands, use those — they are the source of truth. Otherwise detect, propose, confirm with the user, and record in `CLAUDE.md`.

## Detection cues

Look for these in the project root:

| File / directory | Stack signal |
|---|---|
| `package.json` | Node.js / JavaScript / TypeScript |
| `tsconfig.json` | TypeScript |
| `pyproject.toml` | Python (modern) |
| `requirements.txt`, `setup.py` | Python (older) |
| `Pipfile` | Python (pipenv) |
| `poetry.lock` | Python (poetry) |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` | Java (Maven) |
| `build.gradle` / `build.gradle.kts` | Java / Kotlin (Gradle) |
| `Gemfile` | Ruby |
| `composer.json` | PHP |
| `mix.exs` | Elixir |
| `next.config.js` / `.mjs` | Next.js |
| `vite.config.*` | Vite |
| `remix.config.*` | Remix |
| `nuxt.config.*` | Nuxt |
| `svelte.config.*` | SvelteKit / Svelte |
| `astro.config.*` | Astro |
| `prisma/schema.prisma` | Prisma ORM |
| `alembic.ini` / `migrations/` | Alembic / Django migrations |
| `docker-compose.yml` | Containerized dev env |
| `.github/workflows/` | CI config — read for canonical gates |

Always read `.github/workflows/` (or `.gitlab-ci.yml`, `Jenkinsfile`) if present — CI defines the authoritative gates.

Also read `package.json` scripts, `Makefile`, `justfile`, `Taskfile.yml` — these define the exact commands the team uses.

## Default verification commands by stack

Starting points. Always confirm with the user and record the final set in `CLAUDE.md`.

### Node.js / TypeScript

```bash
npm run lint         # or: eslint . / biome check .
npm run typecheck    # or: tsc --noEmit
npm test             # or: vitest run / jest
npm run build        # or: tsc / next build / vite build
```

If `package.json` defines a custom `check` or `ci` script, prefer that.

### Python

```bash
ruff check .                # or: flake8
ruff format --check .       # or: black --check .
mypy .                      # or: pyright
pytest                      # with coverage: pytest --cov
```

If `pyproject.toml` defines commands or the project uses `tox` / `nox`, prefer those.

### Go

```bash
go vet ./...
go build ./...
go test ./...
golangci-lint run   # if configured
```

### Rust

```bash
cargo fmt --check
cargo clippy -- -D warnings
cargo test
cargo build
```

### Full-stack web apps

Frontend and backend usually have separate gate sets. Run both, plus an E2E or smoke test:

```bash
# Frontend
cd frontend && npm run lint && npm run typecheck && npm test && npm run build

# Backend
cd backend && ruff check . && mypy . && pytest

# E2E
npm run test:e2e   # or equivalent
```

## DB schema introspection commands

When the source of truth for a schema is the database itself (not the migrations or models), use these to verify column names before writing queries. **Never assume column names — always confirm.**

```bash
# Postgres
psql -d DBNAME -c "\d TABLENAME"
psql -d DBNAME -c "SELECT column_name, data_type FROM information_schema.columns WHERE table_name = 'TABLENAME';"

# SQLite
sqlite3 PATH.db "PRAGMA table_info(TABLENAME);"

# MySQL
mysql -u USER -p DBNAME -e "DESCRIBE TABLENAME;"
```

Prefer reading migrations or ORM models first — they're version-controlled and authoritative. Fall back to live introspection when migrations are unavailable or when you suspect the live schema has drifted.

## Smoke test guidance

For web apps, a passing unit test suite is not enough. Verify:

- The server actually starts and responds to a real request (`curl` or equivalent)
- The frontend actually renders and fetches from the real backend (not a mock)
- A primary user flow works end-to-end

If the project has Playwright, Cypress, or similar, run at least one E2E test. Otherwise propose adding one, or do a manual smoke test and record the result.

**If browser-based E2E can't run in your environment** — no display server, no browser binaries, a sandbox that blocks them — do not thrash trying to force it, and do not silently claim E2E coverage you never ran. Fall back to the strongest verification you *can* execute (an API-level integration test, an in-process request against the real handler) and record the gap explicitly in `TODO.md`: what couldn't be run, why, and what should run it (e.g. "Playwright smoke test — needs Chromium in CI"). Honest partial verification beats a fake green.

## Recording in CLAUDE.md

Once verified, record commands under a `## Verification` section:

```markdown
## Verification

- `npm run lint`
- `npm run typecheck`
- `npm test`
- `npm run build`
- `npm run test:e2e`
- Backend: `cd api && ruff check . && mypy . && pytest`
```
