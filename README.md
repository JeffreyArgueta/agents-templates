# agents-templates

Reusable project templates optimized for **AI coding agents** — opinionated, layered, and ready to copy-paste for new projects.

This repository is a collection of starter templates where the **agent instructions are the architecture**. Each template ships with a two-layer documentation system so agents get full context at minimal token cost, while humans get consistent conventions and quality gates out of the box.

> Copy a folder → fill in Project Context → start coding. No setup wizard, no generator CLI.

---

## Templates

| Template | Stack | Description |
|----------|-------|-------------|
| [`pnpm-node-express-api`](./pnpm-node-express-api/) | Node.js + Express + Sequelize (MySQL) · ESM · pnpm | REST API with layered architecture (`routes → controllers → services → repositories → models`), JWT cookie auth, Joi validation, Swagger, Docker |
| [`pnpm-vite-react-tailwind`](./pnpm-vite-react-tailwind/) | Vite 6 + React + Tailwind CSS · ESM · pnpm | SPA with feature-based structure, token-driven styling, `cn()` utilities, and composable hooks |

Both templates share the same principles:

- **ESM-only** (`"type": "module"`), `pnpm@10` via `corepack`, Node 22 LTS
- **English** code / logs / comments, strict naming conventions
- **Quality gates**: ESLint + Prettier + tests + audit must pass before push (Husky + CI)
- **No secrets in repo**: `.env` gitignored, `.env.example` is the contract
- **Security & a11y as gates**, not afterthoughts

---

## The Two-Layer Agent System

Traditional templates dump everything into one large `README` or `CLAUDE.md`. Agents either miss details or burn context on irrelevant sections.

These templates use a **layered loading** strategy:

```
Layer 1 (always loaded)          Layer 2 (on demand)
┌─────────────────────┐          ┌──────────────────────────┐
│ AGENTS.md           │   ───►   │ docs/agent/<topic>.md   │
│ - Project Context   │          │ - auth.md               │
│ - How to reason     │          │ - validation.md         │
│ - Folder structure  │          │ - security.md           │
│ - Naming conventions│          │ - testing.md            │
│ - Thematic index    │          │ - docker.md             │
│ - Rules (enforced)  │          │ - ... (16 files)        │
└─────────────────────┘          └──────────────────────────┘
         150 lines                   Full detail, loaded
         ~always in context          only when task touches it
```

**How it works:**

1. The agent **always** reads `AGENTS.md` (Layer 1) — ~150 lines with project context, folder structure, naming conventions, and a thematic index.
2. When a task touches a specific area (e.g., "fix auth" or "add a form"), the agent opens **only** the relevant `docs/agent/*.md` file(s) via the trigger table in §4.
3. Result: 100% of detail available, but context cost is paid only when warranted.

> **Tooling note:** Works with any agent that can read files — OpenCode, Claude Code, Cursor, etc. The agent needs `read_file`/`view` access and must resolve `docs/agent/<topic>.md` relative to the `AGENTS.md` location.

---

## Usage

### 1. Copy the template

```bash
# API example
cp -r pnpm-node-express-api/ ~/my-new-api
cd ~/my-new-api

# Frontend example
cp -r pnpm-vite-react-tailwind/ ~/my-new-app
cd ~/my-new-app
```

### 2. Fill in Project Context

Open `AGENTS.md` §0 and fill the table — this is the **only** file you must edit per project:

**API (`pnpm-node-express-api`):**
- Project name, DB/ORM choice, auth strategy, Node version

**Frontend (`pnpm-vite-react-tailwind`):**
- Project name, routing / state / forms / validation / auth / error tracking choices
- Example: `react-router-dom` | `@tanstack/react-query` | `zustand` | `react-hook-form` | `zod`

Record the choice and follow it consistently. Do not mix two options for the same area without an explicit decision.

### 3. Install & run

```bash
pnpm install

# API
pnpm run dev        # nodemon
pnpm run db:create && pnpm run db:migrate && pnpm run db:seed
pnpm test           # unit + integration
pnpm run lint && pnpm run format

# Frontend
pnpm dev            # vite --host (HMR)
pnpm build && pnpm preview
pnpm lint && pnpm format:check && pnpm test -- --coverage
```

### 4. Let the agent work

Point your agent at `AGENTS.md`. It will follow the layer rules, naming conventions, and quality gates automatically.

---

## Repository Structure

```
agents-templates/
├── README.md                         # this file
├── pnpm-node-express-api/
│   ├── AGENTS.md                     # Layer 1 — always loaded
│   └── docs/agent/                   # Layer 2 — thematic docs (16 files)
│       ├── auth.md                   # cookie + JWT, authenticate/authorize
│       ├── data-layer.md             # models, migrations, Sequelize
│       ├── layers.md                 # repositories/services/controllers
│       ├── validation.md             # Joi schemas
│       ├── response-errors.md        # error classes & middleware
│       ├── pagination.md             # offset / cursor helpers
│       ├── security.md               # helmet, cors, rate-limit, xss
│       ├── config-env.md             # env validation
│       ├── observability.md          # logger, metrics, tracing
│       ├── testing.md                # factories, unit/integration
│       ├── docker.md                 # Dockerfile & compose
│       └── ...
└── pnpm-vite-react-tailwind/
    ├── AGENTS.md                     # Layer 1 — always loaded
    └── docs/agent/                   # Layer 2 — thematic docs (17 files)
        ├── components.md             # ui primitives, layout, features
        ├── state-forms.md            # forms, wizards, server state
        ├── services.md               # http wrapper, api client
        ├── styling.md                # Tailwind tokens, cn()
        ├── routing.md                # guards, lazy, code-splitting
        ├── security.md               # XSS/CSRF/CORS/CSP
        ├── performance-a11y.md       # bundle, axe, reduced-motion
        └── ...
```

---

## Design Principles

| Principle | What it means |
|-----------|---------------|
| **Layer rule is law** | API: `routes → controllers → services → repositories → models`. Frontend: `routes → features → components/ui + hooks → services/http`. Never skip layers, never call `fetch` from a component, never import another feature's private code. |
| **Conventions over configuration** | One file naming pattern per template (kebab-case frontend, camelCase `*.model.js` backend), one class-name joiner (`cn()`), one env validator (`config/env.js`). Copy existing style before inventing. |
| **Generated, not hand-edited** | API repositories/services/controllers are generated from scripts in `/tmp/opencode/` — edit the generator, then re-run. Prevents drift across 5-file CRUD sets. |
| **Security by default** | `helmet`+`cors`+`rate-limit`+`xss-sanitize` in order, `10kb` body limit, `replacements`/`bind` for raw SQL, httpOnly cookies (never `localStorage`), `pnpm audit` + `gitleaks` in CI. |
| **Scoped changes** | Agents keep changes scoped to the request. No opportunistic refactors, no new libraries without proposal (bundle/a11y/license check). |

---

## Keeping Templates in Sync

Each `AGENTS.md` §4 index table is the single source of truth for its `docs/agent/` folder:

- Add/remove a row when you add/remove a thematic doc
- Each `docs/agent/*.md` keeps its `> Read this when…` trigger line at the top
- Keep code examples in Layer 2 — reference them from Layer 1, don't summarize

---

## Contributing

1. Edit `AGENTS.md` or the relevant `docs/agent/<topic>.md` in the template folder
2. Keep Layer 1 lean (~150 lines) and Layer 2 detailed
3. Verify the agent can resolve `docs/agent/<topic>.md` relative to `AGENTS.md`
4. Ensure `pnpm lint` + `pnpm format:check` + `pnpm test` pass

---

## License

Licensed under the [Apache License 2.0](LICENSE) — use freely for new projects. Copy, modify, ship.
