# Stage 1: repository skeleton and tooling

Delivers the Stage 1 row of `docs/07-development-roadmap.md`. No business features, domain rules, database tables or integrations.

## What's included
- **pnpm workspaces:** `apps/api`, `apps/web`, `packages/domain`, `packages/db`, `packages/shared`. Every dependency is pinned exactly in `pnpm-lock.yaml`.
- **API (Fastify 5.12.7):** `GET /healthz` returns 200. `GET /readyz` runs `SELECT 1` against MySQL. It returns 200 when MySQL answers and 503 when it doesn't. The client never sees error detail; that is logged on the server only.
- **Web:** Vite 8 + React 19 + TypeScript shell page.
- **TypeScript 6.0.3** in strict mode everywhere, with `noUncheckedIndexedAccess` and `exactOptionalPropertyTypes`.
- **ESLint 10:** `packages/domain` can't import Node built-ins, `@abt/db`, `mysql2`, Drizzle, Fastify or HTTP clients. It also can't use `process`, `fetch` or timers, call `Date.now()`, `DateTime.now()` or `DateTime.local()`, or create `new Date()` without arguments. Each of these is covered by a test. `dangerouslySetInnerHTML` is banned in the web app. Prettier checks formatting.
- **Vitest 5** runs in every package, plus tests for the tooling itself. **Playwright** has two projects: `desktop-chromium` and `mobile-375` (375×812).
- **`infra/docker-compose.yml`:** MySQL 8.4.11 on an `internal: true` network with no published port, UTC time zone and strict `sql_mode`. The development API container is published on the host loopback only. **`infra/docker-compose.test.yml`:** a disposable MySQL on tmpfs, bound to the loopback.
- **`.env.example`** contains placeholders only. `.env` stays git-ignored.
- **Scripts:** `lint`, `typecheck`, `test:unit`, `test:e2e`, `test:trace`, `audit`, `verify`.
- **CI** on the GitHub Actions free tier: installs, checks the compose files, then runs `pnpm verify`.

## Dependency justification (CLAUDE.md §2)
All dependencies are open source (MIT, Apache-2.0, ISC, BSD, MPL-2.0, BlueOak; one CC-BY-4.0 data package) and actively maintained. None is a paid service.

| Dependency | Why it's needed |
|---|---|
| fastify | The API framework chosen in D-02 |
| mysql2 | MySQL driver for the readiness check, and later for Drizzle |
| react, react-dom, vite, @vitejs/plugin-react | The web stack chosen in D-02 |
| typescript, tsx | Type checking, and running TypeScript in development |
| eslint, @eslint/js, typescript-eslint, eslint-plugin-react-hooks, eslint-config-prettier, globals | Linting, including the domain purity rule |
| prettier | Formatting |
| vitest | Unit tests (`docs/05`) |
| @playwright/test | E2E tests (`docs/05`) |
| @types/* | Type definitions |

## Verification (run locally in the session sandbox)
- `pnpm verify` exits 0. That covers lint, typecheck, unit tests (25 passing in 7 files), the traceability check, `pnpm audit` (no known vulnerabilities) and the E2E tests (4 passing: 2 desktop, 2 at 375 px).
- `docker compose -f infra/docker-compose.yml up -d mysql`: healthy, no host port published.
- `/healthz` → `200 {"status":"ok"}`. `/readyz` → `200 {"status":"ready","checks":{"mysql":"ok"}}`.
- With MySQL stopped, `/readyz` → `503 {"status":"not_ready","checks":{"mysql":"unavailable"}}` and `/healthz` still returns 200. After restarting MySQL, readiness recovers.

## Needs review
- Technical assumptions T-01 to T-07 in `docs/08 §2d`. The most important is T-01: by default the traceability check fails on undefined IDs but only reports requirements that have no tests yet, and `--strict` turns those into failures.
- The CI workflow hasn't run yet. It will run for the first time on this PR.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_011FWuaLDSK1NpSGTBhBRBVM
