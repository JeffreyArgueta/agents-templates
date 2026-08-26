## 12. Logging & Observability — Frontend

> Read this when adding logging, error tracking, or Web Vitals.

- **Dev logs:** use a tiny `lib/logger.js` that no-ops in production:
  ```js
  export const logger = {
    info: (...a) => { if (import.meta.env.DEV) console.info(...a); },
    warn: (...a) => { if (import.meta.env.DEV) console.warn(...a); },
    error: (...a) => console.error(...a), // always
  };
  ```
  Never use bare `console.log` in `src/` — enforce `no-console` via eslint (allow `warn` and `error` only).

- **Error tracking:** when an error tracker is chosen in §0, initialize it once:
  ```js
  import * as Sentry from '@sentry/react';
  if (env.VITE_SENTRY_DSN) {
    Sentry.init({ dsn: env.VITE_SENTRY_DSN, tracesSampleRate: 0.1, environment: import.meta.env.MODE });
  }
  // ErrorBoundary fallback calls the tracker's capture method
  ```

- **Web Vitals:** optionally report via `web-vitals`:
  ```js
  import { onCLS, onINP, onLCP } from 'web-vitals';
  onCLS(console.log); onINP(console.log); onLCP(console.log);
  // or send to an analytics endpoint
  ```

- **Correlation:** include a `requestId` (from `X-Request-Id` header) in `http()` error logs so frontend errors can be joined with backend traces.
- Never log sensitive or personally identifiable data.

---
