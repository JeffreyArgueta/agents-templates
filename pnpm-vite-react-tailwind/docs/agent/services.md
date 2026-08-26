## 7. Services — API Client & Domain Contracts

> Read this when touching the API client, HTTP wrapper, domain contracts, or external adapters.

### `services/http.js` — single fetch wrapper (no duplication)

```js
// services/http.js
import { env } from '@/config/env.js';

export class HttpError extends Error {
  constructor(message, { status, code, details } = {}) {
    super(message);
    this.name = 'HttpError';
    this.status = status;
    this.code = code;
    this.details = details;
  }
}

export async function http(path, { method = 'GET', body, headers, signal, credentials = 'include' } = {}) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 15000);
  const res = await fetch(`${env.VITE_API_URL}${path}`, {
    method,
    headers: { 'Content-Type': 'application/json', ...headers },
    body: body ? JSON.stringify(body) : undefined,
    signal: signal ?? controller.signal,
    credentials,
  }).finally(() => clearTimeout(timeout));

  const data = await res.json().catch(() => ({}));
  if (!res.ok || data.ok === false) {
    throw new HttpError(data.error || data.message || 'Request failed', {
      status: res.status,
      code: data.code,
      details: data,
    });
  }
  return data.data ?? data;
}
```

- Always `encodeURIComponent` path params.
- Always forward `signal` from the server-state library or an `AbortController` so navigation cancels in-flight requests.
- Do not expose `fetch` directly — every domain function goes through `http()` so `credentials`, `baseUrl`, and error shape are consistent.
- Adjust `credentials`, headers, and timeout per project auth choice in §0. If bearer auth is used, inject the token via an interceptor or wrapper rather than per call.

### Domain Contracts (`services/api.js`)

- Keep an **adapter** layer between API payloads and UI values. Map between backend field names and form field names in one place and document the contract version in a JSDoc header.
- Validate env before any call (e.g. missing `VITE_API_URL` throws a typed error) — do not silently fall through.
- Post-write verification (e.g. confirming saved selections) belongs in the service, not the component — components only handle `try/catch` and surface a user-facing message.
- For local development without a backend, gate mocks via an explicit env flag (e.g. `VITE_USE_MOCK=true`) or mock handlers, not via `if (!apiUrl)` branching scattered in components.

### External Adapters (`services/external.js` — optional)

- Wrap third-party calls with retry, `staleTime` (via the chosen server-state library), in-memory caching where appropriate, and a local fallback when the external service is unavailable.
- Forward `AbortSignal` and use `cache: 'force-cache'` for rarely changing data.
- When the backend eventually provides the same data, the adapter should become a thin switch between the external source and the backend endpoint.

---
