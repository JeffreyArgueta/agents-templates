# AGENTS.md — Reusable API Template (Layer 1 — Always Loaded)

> Generic architecture for Node.js + Express REST APIs (ESM, pnpm).
> Copy this folder for new projects and edit only section 0. Layer 1 is the only file the agent always reads; thematic detail lives in `docs/agent/*.md` and is opened on demand.

## Contents

- [0. Project Context](#0-project-context-edit-per-project) — fill once per project
- [1. How the Agent Should Reason](#1-how-the-agent-should-reason)
- [2. Folder Structure](#2-folder-structure)
- [3. Naming Conventions](#3-naming-conventions)
- [4. Thematic Docs Index — When to Read What](#4-thematic-docs-index--when-to-read-what)
- [5. Rules for Agents](#5-rules-for-agents) — always enforced

---

## 0. Project Context (EDIT per project)

- **Name:** `<project-name>`
- **Type:** REST API (Node.js + Express)
- **Language:** JavaScript (ESM — `type: module`) or TypeScript
- **Package manager:** pnpm
- **Node version:** latest (e.g. `22.x` LTS, see `.nvmrc`)
- **Database / ORM:** MySQL + Sequelize 6 (`mysql2` driver)
- **Auth:** cookie-based (httpOnly, secure, sameSite) + JWT — see `docs/agent/auth.md`
- **Key commands:**
  - Install: `pnpm install`
  - Development: `pnpm run dev` (nodemon)
  - Production: `pnpm start` (`node server.js`)
  - DB create: `pnpm run db:create`
  - DB seed: `pnpm run db:seed`
  - DB migrate: `pnpm run db:migrate`
  - DB migrate undo: `pnpm run db:migrate:undo`
  - Lint: `pnpm run lint` · Lint fix: `pnpm run lint:fix` · Format: `pnpm run format`
  - Tests: `pnpm test` (unit + integration) · Audit: `pnpm audit`
  - Layer check: `node -e "import('./src/models/index.js').then(()=>console.log('models OK'))"`
  - Syntax check: `find src -name '*.js' -exec node --check {} \;`

---

## 1. How the Agent Should Reason

- Before writing code, **read existing files** in the relevant folder to copy the established style instead of inventing a new one.
- If a task is ambiguous, pick the most reasonable interpretation following this document and state what was assumed.
- Never introduce a new library unless it already exists in `package.json`, unless explicitly requested. If needed, propose it first.
- Do not modify sensitive config files (`.env`, DB migrations, CI/CD, `docker-compose.yml`) without calling it out explicitly.
- Keep changes scoped to the request: do not opportunistically refactor unrelated code.
- For structural changes to generated layers, edit the generator script, not the generated files.
- Prefer ESM `import`/`export` everywhere; never use `require`/`module.exports` (project has `"type": "module"`).

**ESM essentials:** No `__dirname`/`__filename` — derive via `fileURLToPath(import.meta.url)`. All imports use explicit `.js` extension. `sequelize-cli` needs `.sequelizerc` with `database/config.cjs` or ESM `config.js`. `nodemon` + `vitest` work with ESM on Node >=20 with `package.json { "type":"module" }`. See `docs/agent/layers.md` and `docs/agent/data-layer.md` for details.

---

## 2. Folder Structure

```
.
  package.json  .nvmrc  .env.example  eslint.config.js  .prettierrc  .gitleaksignore
  docker-compose.yml  docker-compose.override.yml  Dockerfile  .dockerignore
  src/
    app.js  server.js
    config/{environment.js,database.js,swagger.js,swagger/}
    models/<model>.model.js + index.js
    schemas/<model>.schema.js
    repositories/<model>/  services/<model>/  controllers/<model>/  # 5 files each
    routes/v1/<model>.routes.js + v1/index.js + index.js
    middlewares/  utils/{logger.js,pagination.js,errors.js}
  database/{migrations,seeders,config.cjs}
  tests/{unit/{repositories,services},integration/routes,factories,setup.js}
```

**Transversal modules** (e.g. `statistics`, `reports`): aggregated SQL, no underlying model. Own `repositories/`, `services/`, `controllers/`, `routes/`, `schemas/` but **not** aggregated in top-level `index.js` files — routes import them directly.

**Layer rule:** `routes (v1) → controllers → services → repositories → models` · `schemas ↑ utils ↑ errors ↑`. Controllers never import models/repositories; services never import `req`/`res`; repositories own all data access and contain no business rules.

---

## 3. Naming Conventions

### File Naming

| Element | Convention | Example |
|---------|------------|---------|
| Model files | camelCase `<model>.model.js` | `user.model.js` |
| Schema files | camelCase `<model>.schema.js` | `user.schema.js` |
| Route files | camelCase `<model>.routes.js` | `user.routes.js` (under `routes/v1/`) |
| Repository folder | camelCase (model) | `repositories/user/` |
| Repository file | `<verb><Resource>.repository.js` | `getUsers.repository.js` |
| Service folder | camelCase (model) | `services/user/` |
| Service file | `<verb><Resource>.service.js` | `getUsers.service.js` |
| Controller folder | camelCase (model) | `controllers/user/` |
| Controller file | `<verb><Resource>.controller.js` | `getUsers.controller.js` |
| Middleware | kebab-case `<name>.middleware.js` | `auth.middleware.js` |
| Utils / Tests | camelCase or kebab-case / `<file>.test.js` | `pagination.js`, `user.service.test.js` |
| Endpoints | kebab-case, plural | `/api/v1/user-profiles` |

### Identifiers

| Element | Convention | Example |
|---------|------------|---------|
| Classes / Models | PascalCase | `User`, `UserRole` |
| Functions / variables | camelCase | `getUserById()` |
| Global constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| DB tables / columns | snake_case (`underscored: true` maps to camelCase) | `user_profiles` / `first_name` ↔ `firstName` |

**Operations per model (5 each):** `get<Plural>` (list paginated), `get<Singular>ById`, `create<Singular>`, `update<Singular>`, `delete<Singular>` — same file set in `repositories/`, `services/`, `controllers/` and re-exported via `index.js` keyed by `PascalCase`. Layers are generated via scripts in `/tmp/opencode/` — edit the generator, not generated files.

---

## 4. Thematic Docs Index — When to Read What

The agent **always** reads this file. It **only** opens a file below when the task touches that area. The trigger column is the decision signal — not just a link.

| Task touches… | Read this | Trigger |
|---------------|-----------|---------|
| Models, migrations, DB config, ORM | `docs/agent/data-layer.md` | `database/migrations/*`, `src/models/*`, `src/config/database.js` |
| Joi schemas, validation rules | `docs/agent/validation.md` | `src/schemas/*`, `*.schema.js` |
| Repositories / services / controllers, layering | `docs/agent/layers.md` | `src/repositories/*`, `src/services/*`, `src/controllers/*` |
| API response shape, error classes, error middleware | `docs/agent/response-errors.md` | `src/utils/errors.js`, `src/middlewares/error.middleware.js` |
| Pagination (offset or cursor) | `docs/agent/pagination.md` | `src/utils/pagination.js`, `limit`/`offset`/`cursor` |
| Auth, cookies, JWT, guards | `docs/agent/auth.md` | `src/middlewares/auth.middleware.js`, `src/config/environment.js` (JWT) |
| Security, CORS, helmet, rate limiting, raw SQL | `docs/agent/security.md` | `src/app.js` middleware order, `src/middlewares/*`, `sequelize.query` |
| Env vars, config validation | `docs/agent/config-env.md` | `src/config/environment.js`, `.env.example` |
| Logging, metrics, tracing, health | `docs/agent/observability.md` | `src/utils/logger.js`, `src/utils/metrics.js`, `src/config/tracing.js` |
| Unit / integration tests, factories, setup | `docs/agent/testing.md` | `tests/*`, `tests/factories/*` |
| ESLint, Prettier, Husky | `docs/agent/lint-format.md` | `eslint.config.js`, `.prettierrc` |
| API versioning, deprecation | `docs/agent/versioning.md` | `src/routes/v1/*`, `src/routes/v2/*` |
| Swagger / OpenAPI docs | `docs/agent/documentation.md` | `src/config/swagger.js`, `src/config/swagger/*` |
| Dockerfile, compose, deployment | `docs/agent/docker.md` | `Dockerfile`, `docker-compose.yml`, `.dockerignore` |
| Commits, branches, PRs, versioning | `docs/agent/git-workflow.md` | Commits, PR checklist |
| Canonical `pnpm` commands | `docs/agent/commands.md` | Any `pnpm run db:*`, `lint`, `test`, `docker` task |

**How to use:** If your task is "add a new model", you read this file + `docs/agent/data-layer.md` + `docs/agent/validation.md` + `docs/agent/layers.md`. If it is "fix auth", you add `docs/agent/auth.md` + `docs/agent/security.md`. You still have 100% of the detail — you just pay the context cost only when warranted.

**Tooling note:** Some environments (Claude Code, Cursor, etc.) auto-read `AGENTS.md` but do not auto-open referenced files. The agent must have `read_file` / `view` access and use the relative paths above (`docs/agent/<topic>.md` from the folder that contains this `AGENTS.md`). Verify the tool can open them before assuming.

---

## 5. Rules for Agents

1. **Source of truth:** migrations in `database/migrations/` + `src/models/` for ORM mapping. Verify names before assuming.
2. **Respect layers:** `routes/v1` → `controllers` → `services` → `repositories` → `models`. Never skip layers.
3. **Generated code:** `repositories`, `services` and `controllers` are regenerated from scripts in `/tmp/opencode/` (`generate-repositories.js`, `generate-services.js`, `generate-controllers.js`). Structural changes go in the generator, then re-run it.
4. **English:** messages, logs, function names and comments in English, consistent with existing files.
5. **Errors:** services throw typed errors from `utils/errors.js` (`NotFoundError`, `ConflictError`, `ValidationError`); controllers **throw** and let the centralized `error.middleware` handle them — never duplicate `res.status(error.status||500)`. Joi → `ValidationError` (422).
6. **ESM:** use `import`/`export` with explicit `.js` extensions, `package.json { "type": "module" }`. Handle `__dirname` via `import.meta.url` (§1). Never use `require`/`module.exports`.
7. **Never commit secrets:** `.env` is gitignored; config changes go in `.env.example` and `docker-compose.yml` via `env_file`. Run `pnpm audit` + `gitleaks detect` before push; enable Dependabot alerts.
8. **Auth:** use httpOnly cookies (`authenticate` + `authorize` middlewares). Never store tokens in localStorage, never log cookies. `CORS credentials:true` + `sameSite` policy as `docs/agent/auth.md`. Mutating routes need CSRF token when `sameSite:none`.
9. **Versioning:** additive → `v1`, breaking → new `v2` folder + new Swagger spec. Never break `/api/v1` without deprecation headers.
10. **Audit pattern (if needed):** create snapshots before UPDATE of main entity and after INSERT/UPDATE/DELETE on bridge tables, **within the same transaction**. Snapshot shape must equal the aggregated `GET /:id/complete` response so it can be diffed and round-tripped via `PUT /:id/complete`.
11. **Quality gates:** lint + format + tests + audit must pass before push (husky + CI). Maintain ≥80% coverage. Use factories (`tests/factories/`) for test data.
12. **Security:** body limit `10kb`, `compression` before routes, always `replacements`/`bind` for raw SQL, `helmet`+`cors`+`rate-limit`+`xss-sanitize` in order (`docs/agent/security.md`). For cursor pagination use `docs/agent/pagination.md` helper; for large lists prefer cursor over offset.

---

## Maintenance — Keeping the Two Layers in Sync

- The index table in §4 is the single source of truth for thematic docs — add/remove a row whenever you add/remove a file in `docs/agent/`.
- Each `docs/agent/*.md` keeps its `> Read this when…` line at the top; it is the trigger that tells the agent whether to open it.
- Do not summarize long sections into Layer 1 — keep code examples in Layer 2 and reference them from the index.
- Verify the agent has `read_file`/`view` access and that relative paths `docs/agent/<topic>.md` resolve from this file's directory.
