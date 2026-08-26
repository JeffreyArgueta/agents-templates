## 11. Configuration & Environment

> Read this when adding or changing env vars or config validation.

`src/config/environment.js` (ESM) is the **only** place that reads `process.env`. It:

- Loads `dotenv` from project root `.env`.
- Declares `requiredEnvVars` (e.g. `ALLOWED_ORIGINS`, `DB_URI`, `DB_DIALECT`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_NAME`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `COOKIE_SECRET`, `PORT`, `NODE_ENV`).
- Throws on startup if any required var is missing.
- Exports typed config: `NODE_ENV`, `PORT` (parsed int), `ALLOWED_ORIGINS`, `DB_*`, `JWT_*`, `COOKIE_*`.

`.env` is gitignored; `.env.example` documents required vars. Never commit secrets. In Docker, env comes from `docker-compose.yml` / compose secrets. See §13 and §18 for scanning.

---
