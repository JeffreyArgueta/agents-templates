## 11. Configuration & Environment

> Read this when adding or changing env vars, config validation, or runtime vs build-time env.

`src/config/env.js` is the **only** place that reads `import.meta.env`. It validates on import and throws early.

```js
// src/config/env.js
import { z } from 'zod';

const envSchema = z.object({
  VITE_API_URL: z.string().url('VITE_API_URL must be a valid URL').or(z.literal('')),
  VITE_USE_MOCK: z.enum(['true', 'false']).optional().default('false'),
  VITE_SENTRY_DSN: z.string().optional(),
  VITE_APP_NAME: z.string().optional(),
});

export const env = envSchema.parse({
  VITE_API_URL: import.meta.env.VITE_API_URL,
  VITE_USE_MOCK: import.meta.env.VITE_USE_MOCK,
  VITE_SENTRY_DSN: import.meta.env.VITE_SENTRY_DSN,
  VITE_APP_NAME: import.meta.env.VITE_APP_NAME,
});

export function assertEnv() {
  if (!env.VITE_API_URL && env.VITE_USE_MOCK !== 'true') {
    throw new Error('Set VITE_API_URL or enable VITE_USE_MOCK=true for development without a backend');
  }
}
assertEnv();
```

- Adjust the schema to match the env vars actually needed per project — keep the single-file validation principle.
- If a different validation library is chosen in §0, use it here instead of `zod` with the same fail-fast behavior.
- `.env` is gitignored; `.env.example` documents required vars with example values.
- In Docker or preview, env is injected at **build time** for Vite (static). For runtime switching without rebuilding, inject `window.__ENV__` from the server and read it in `env.js` — document the chosen strategy in `README.md`.
- Never commit `.env`; CI injects from secrets.

---
