## 14. Lint & Format (ESM)

> Read this when changing ESLint, Prettier, Husky, or lint-staged config.

- **ESLint** (flat config `eslint.config.js`, ESM) + **Prettier**. Config enforces ESM (`import`/`export`), `camelCase`, no `require`, `no-console` (use `logger`), `no-implied-eval`.
- **Scripts:** `pnpm run lint` (check), `pnpm run lint:fix` (autofix), `pnpm run format` (Prettier), `pnpm run format:check`.
- **Husky + lint-staged:** pre-commit runs `eslint --fix` + `prettier --write` on staged `*.js`; pre-push runs `pnpm test`.
- **CI:** GitHub Actions (or similar) fails on `lint` or `test` failure.

`package.json` must have `"type": "module"` — all imports use explicit `.js` extension: `import { logger } from './utils/logger.js'`.

---
