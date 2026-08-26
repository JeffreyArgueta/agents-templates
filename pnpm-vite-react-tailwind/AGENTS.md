# AGENTS.md — Reusable Frontend Template (Layer 1 — Always Loaded)

> Generic architecture for Vite + React + Tailwind CSS SPAs (ESM, pnpm).
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
- **Type:** SPA (Vite + React + Tailwind CSS)
- **Language:** JavaScript (ESM — `type: module`) or TypeScript (`strict` recommended)
- **Package manager:** pnpm (`pnpm@10` via `corepack`; do not mix with npm/yarn)
- **Node version:** latest LTS (e.g. `22.x`, see `.nvmrc` + `engines`)
- **Build:** Vite 6+ (`@vitejs/plugin-react`)
- **Styling:** Tailwind CSS (pinned per project) + `postcss` + `autoprefixer`
- **Stack decisions — fill before coding (only `pnpm` is fixed):**

  | Area | Options (pick one) | Chosen |
  |------|--------------------|--------|
  | Routing | `react-router-dom` / `tanstack-router` / none | `<choose>` |
  | Server state | `@tanstack/react-query` / `swr` / custom fetch | `<choose>` |
  | Client state | `useState`/`useReducer` / `zustand` / `jotai` / `redux-toolkit` | `<choose>` |
  | Forms | `react-hook-form` / `formik` / native | `<choose>` |
  | Validation | `zod` / `yup` / `valibot` / `joi` | `<choose>` |
  | Auth | cookie (`httpOnly` + `credentials: include`) / bearer + refresh / none | `<choose>` |
  | Error tracking | `sentry` / none | `<choose>` |
  | Component docs | `storybook` / none | `<choose>` |

- **Key commands:** `pnpm install` · `pnpm dev` · `pnpm build` · `pnpm preview` · `pnpm lint` / `lint:fix` · `pnpm format` / `format:check` · `pnpm test` · `pnpm audit` · `pnpm typecheck` (if TS)

> Record the choice in the table and follow it consistently. Do not mix two options for the same area without an explicit decision.

---

## 1. How the Agent Should Reason

- Before writing code, **read existing files** in the relevant folder (`components/ui/*`, `services/*`, `tailwind.config.js`, `src/index.css`) to copy the established style.
- If a task is ambiguous, pick the most reasonable interpretation and state the assumption in the PR description.
- Never introduce a new library unless it already exists in `package.json` — propose first (bundle, a11y, license).
- Do not modify sensitive config (`.env`, `vite.config.js` proxy, `docker-compose.yml`, CI) without calling it out.
- Keep changes scoped; do not refactor unrelated code.
- Prefer composition over prop drilling: extract `hooks/use-*.js` before adding global state.
- When a project already has a different convention, follow the project and note the divergence.

**Vite + ESM essentials:** `type: module` → `import`/`export` everywhere. Alias `@` → `src/` is canonical (never `../../../`). `import.meta.env` is the only way to read env in `src/`; all browser vars are `VITE_*` and validated in `config/env.js` (see `docs/agent/config-env.md`).

---

## 2. Folder Structure

```
.
  package.json  .nvmrc  .env.example  vite.config.js  tailwind.config.js  postcss.config.js
  jsconfig.json  index.html  eslint.config.js  .prettierrc  .husky/
  public/  src/
    main.jsx  app.jsx  index.css
    config/env.js  routes/index.jsx  routes/guards.jsx
    features/<feature>/{components,hooks,services,schemas,index.js}
    components/{ui,layout}  hooks/  services/{http.js,api.js}  stores/  lib/cn.js  utils/  data/  test/{setup.js,factories,mocks}
  e2e/  docs/agent/
```

**Layer rule:** `routes → features/components → components/ui + hooks → services/http → external API` · `schemas ↑ stores ↑ lib/cn ↑ utils ↑`. No feature imports another feature's private `components/`; `services/` owns all network I/O; `lib/cn` is the only class-name joiner. Pick `features/` or flat `components/`+`hooks`+`services` and stay consistent. See `docs/agent/` files for full trees.

---

## 3. Naming Conventions

**Single file pattern for the entire codebase: kebab-case for all files.**

| Element | Convention | Example |
|---------|------------|---------|
| Components | kebab-case `*.jsx` | `text-field.jsx`, `progress-bar.jsx` |
| Hooks | kebab-case `use-*.js` | `use-wizard.js`, `use-debounce.js` |
| Services / Schemas / Stores / Utils | kebab-case | `http.js`, `user.schema.js`, `auth.store.js`, `cn.js` |
| Routes / Config / Tests / Styles | kebab-case | `routes/index.jsx`, `config/env.js`, `button.test.jsx`, `index.css` |

| Identifiers | Convention | Example |
|-------------|------------|---------|
| Components / Classes | PascalCase | `TextField`, `UserCard` |
| Hooks / Functions / Vars | camelCase | `useAuth`, `getUserById()` |
| Constants / Env | UPPER_SNAKE / `VITE_*` | `MAX_RETRIES`, `VITE_API_URL` |
| CSS tokens | kebab-case | `brand-primary`, `surface-muted` |

Primitives accept `className` + `...props`, forward `ref`, use `cn()` for classes, and expose `displayName`. Every input has `label[htmlFor]` + `aria-invalid` + `aria-describedby`.

---

## 4. Thematic Docs Index — When to Read What

The agent **always** reads this file. It **only** opens a file below when the task touches that area. The trigger column is the decision signal — not just a link.

| Task touches… | Read this | Trigger |
|---------------|-----------|---------|
| Forms, wizards, server-state fetching, cascading selects | `docs/agent/state-forms.md` | Any form, `useReducer`/`useForm`, query/mutation, dependent selects |
| Validation schemas, field rules, cross-field logic | `docs/agent/validation.md` | New/changed `*.schema.js`, `zod`/`yup`/`valibot` rules |
| UI primitives, layout, feature components | `docs/agent/components.md` | `components/ui/*`, `components/layout/*`, `features/*` |
| API client, `http` wrapper, domain adapters | `docs/agent/services.md` | `services/http.js`, `services/api.js`, external adapters |
| Tailwind config, tokens, global CSS | `docs/agent/styling.md` | `tailwind.config.js`, `src/index.css`, design tokens |
| Routes, guards, navigation, code-splitting | `docs/agent/routing.md` | `routes/*`, `createBrowserRouter`, `lazy`/`Suspense` |
| Auth, secrets, XSS/CSRF/CORS/CSP, audits | `docs/agent/security.md` | `VITE_*`, auth, `dangerouslySetInnerHTML`, headers |
| Env vars, config validation, build vs runtime env | `docs/agent/config-env.md` | `config/env.js`, `.env.example` |
| Logging, error tracking, Web Vitals | `docs/agent/observability.md` | `lib/logger.js`, Sentry, `web-vitals` |
| Unit / integration / e2e tests, factories, mocks | `docs/agent/testing.md` | `*.test.*`, `test/setup.js`, `mocks/`, `e2e/` |
| ESLint, Prettier, Husky, lint-staged | `docs/agent/lint-format.md` | `eslint.config.js`, `.prettierrc` |
| Bundle, rendering, keyboard/screen-reader a11y | `docs/agent/performance-a11y.md` | Perf budgets, `axe`, `prefers-reduced-motion` |
| README, Storybook, ADRs | `docs/agent/documentation.md` | `README.md`, Storybook, `docs/adr/` |
| Docker/nginx **or** Vercel/Netlify/Cloudflare | `docs/agent/docker.md` | `Dockerfile`, `nginx.conf`, `vercel.json`, `netlify.toml` — **conditional** (see file) |
| Commits, branches, PRs, versioning | `docs/agent/git-workflow.md` | Commits, PR checklist |
| Canonical `pnpm` commands | `docs/agent/commands.md` | Any `pnpm dev/build/test/lint` task |

**How to use:** If your task is "fix a button", you read this file + `docs/agent/components.md` only. If it is "configure CI", you add `docs/agent/lint-format.md` + `docs/agent/testing.md`. You still have 100% of the detail — you just pay the context cost only when warranted.

**Tooling note:** Some environments (Claude Code, Cursor, etc.) auto-read `AGENTS.md` but do not auto-open referenced files. The agent must have `read_file` / `view` access and use the relative paths above (`docs/agent/<topic>.md` from the folder that contains this `AGENTS.md`). Verify the tool can open them before assuming.

---

## 5. Rules for Agents

1. **Source of truth:** `tailwind.config.js` for tokens, `src/index.css` for globals, `src/config/env.js` for env. Verify `@` alias resolves before assuming a path.
2. **Respect layers:** `routes → features → components/ui + hooks → services/http`. Never call `fetch` from a component; never import another feature's private `components/`.
3. **No monoliths:** keep feature containers focused — extract `useWizard`, `useSubmit`, `useDependentData`, `useScrollToFirstError` when a file exceeds ~150 lines. Extend config/schemas instead of copy-pasting blocks.
4. **English:** code, logs, function names, and comments in English. Follow kebab-case for all files; `PascalCase` for components, `camelCase` for hooks/functions.
5. **Errors:** `services/http.js` throws typed `HttpError` (`status`/`code`); components render a user-facing message with `aria-live="polite"`. Do not duplicate status mapping or log sensitive data.
6. **ESM + alias:** `type: module` + `import`/`export` + `@/...` imports. Never use `require`/`module.exports`.
7. **Never commit secrets:** `.env` is gitignored; env changes go in `.env.example` + compose `args` or provider env UI. Run `pnpm audit` and enable secret scanning.
8. **Auth:** follow the choice in §0; do not mix modes. Never store tokens in plain `localStorage` without an explicit decision; never log auth headers/cookies; forward `signal` for cancellable requests.
9. **A11y + perf are gates:** every input has `label` + `aria-invalid` + `aria-describedby`; routes are `lazy()` when splitting is enabled; overlays are keyboard-navigable. `axe` and tests must pass.
10. **Quality gates:** `lint` + `format:check` + `test -- --coverage` (threshold per project) + `build` must pass before push (Husky + CI). Use `test/factories/` and mock handlers.
11. **Styling:** no new hex values — use `theme.extend` tokens; use `cn()` (`clsx` + `tailwind-merge`) for conditional classes; respect `prefers-reduced-motion`.
12. **Config drift:** if `tailwind.config.js` `content` changes, update `postcss.config.js` + `vite.config.js`; if a `VITE_` var is added, update `config/env.js`, `.env.example`, `README.md`, and Docker/provider config together.

---

## Maintenance — Keeping the Two Layers in Sync

- The index table in §4 is the single source of truth for thematic docs — add/remove a row whenever you add/remove a file in `docs/agent/`.
- Each `docs/agent/*.md` keeps its `> Read this when…` line at the top; it is the trigger that tells the agent whether to open it.
- Do not summarize long sections into Layer 1 — keep code examples in Layer 2 and reference them from the index.
- Verify the agent has `read_file`/`view` access and that relative paths `docs/agent/<topic>.md` resolve from this file's directory.
