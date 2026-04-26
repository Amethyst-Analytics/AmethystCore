# AmethystCore — Agent Rules

> Repo-specific rules for Claude Code and Gemini CLI. Read `../CLAUDE.md` and `../GEMINI.md` first — this file extends those, it does not replace them.

---

## Purpose

AmethystCore is the **shared Python library** used by every other service. It contains only code that is genuinely shared — database connectors, config loading, base classes, and logging. Nothing service-specific lives here.

---

## Stack

| Item | Detail |
|---|---|
| Language | Python 3.11.x |
| Package manager | Poetry 1.7.x |
| Package name | `amethyst_core` |
| Install method | Local path dependency in each consuming service's `pyproject.toml` |

---

## What Belongs Here

| Module | Purpose |
|---|---|
| `amethyst_core.connectors.postgres` | `asyncpg` connection pool factory |
| `amethyst_core.connectors.redis` | `aioredis` connection factory |
| `amethyst_core.config` | `AmethystBaseConfig` (Pydantic BaseSettings) |
| `amethyst_core.logging` | Structured JSON logger setup |
| `amethyst_core.base_service` | `BaseService` — SIGTERM/SIGINT graceful shutdown |
| `amethyst_core.market_calendar` | UTC market hours, trading day logic |

---

## What Does NOT Belong Here

- Business logic specific to any single service
- Route handlers, FastAPI apps
- Tick processing, instrument sync, OAuth flows
- Any import from `market_monitor`, `amethyst_server`, or `amethyst_analytics`

**If you are unsure**: ask before adding. A wrong addition here forces a version bump and re-install across all services.

---

## Naming Conventions

- Module names: `snake_case`
- Classes: `PascalCase`
- Constants: `SCREAMING_SNAKE_CASE`
- Private helpers: `_leading_underscore`
- Config fields: `snake_case` (Pydantic will map to env vars as `UPPER_SNAKE_CASE`)

---

## Forbidden Patterns

- No `from amethyst_server import ...` / `from market_monitor import ...` — circular dependency
- No hardcoded connection strings, secrets, or hostnames — all come from `AmethystBaseConfig`
- No `os.environ` inline — use the config object
- No `print()` — use the structured logger
- No wildcard imports

---

## Testing

- Unit tests must mock all I/O (DB, Redis).
- Integration tests use `testcontainers` with real PostgreSQL 16 + TimescaleDB and real Redis 7.2.
- Test file: `tests/<module>/test_<file>.py` mirroring `src/amethyst_core/<module>/<file>.py`.
- Coverage minimum: 80% (CI hard fails below this).

---

## Service Boundaries

AmethystCore has no runtime of its own. It is imported as a library. It must never open network connections at import time — connections are created on demand via factory functions.
