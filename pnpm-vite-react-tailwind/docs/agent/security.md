## 10. Security & Hardening (Frontend)

> Read this when handling auth, env secrets, XSS, CSRF, CORS, CSP, or dependency audits.

> Apply every item; check it in PR review. Adapt to the auth choice in §0.

- **No secrets in `VITE_*`**: anything prefixed `VITE_` is bundled into the client and readable by anyone. `VITE_API_URL` is public by design; never put JWT secrets, API keys, or database credentials there.
- **Env validation:** fail fast if a required env var is missing or malformed — see §11.
- **XSS:** React escapes by default — never use `dangerouslySetInnerHTML` and never inject `innerHTML` for markdown or rich text unless sanitized with a library like `DOMPurify`. Sanitize user-provided free-text fields before rendering if they are displayed elsewhere.
- **Auth:** if cookie-based, always use `credentials: 'include'` and let the backend set `httpOnly`, `secure`, `sameSite` — the frontend never reads the cookie value and never logs `headers.Cookie`. If bearer-based, store tokens as decided in §0 (memory or secure cookie, never plain `localStorage` without justification) and inject via the `http` wrapper.
- **CSRF:** with `sameSite: strict` or `lax`, most flows are safe. If the backend uses `sameSite: none`, add a double-submit token: backend sets a non-httpOnly `csrfToken` cookie and the frontend sends an `x-csrf-token` header on mutating requests.
- **CORS:** the backend must whitelist allowed origins; the frontend must document the `credentials` requirement. Do not use `mode: 'no-cors'`.
- **Content injection:** data coming from catalogs or external adapters must be validated (e.g. `z.string()`) before render. React escapes text, but do not bypass it.
- **Headers (via hosting):** set `Content-Security-Policy`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, and `Permissions-Policy` via CDN, reverse proxy, or `<meta http-equiv>` fallback. For static hosting, add these at the edge.
- **Rate limiting and abuse:** sensitive endpoints are rate-limited server-side; the frontend should handle `429` with a user-facing message and respect `Retry-After`, with optional exponential backoff in the `http` wrapper.
- **Dependency audit:** `pnpm audit` and secret scanning (e.g. `gitleaks detect`) run in CI; enable automated dependency updates.
- **Do not log sensitive data:** never log tokens, passwords, or personally identifiable information from payloads. Redact in any logger that mirrors backend observability.

---
