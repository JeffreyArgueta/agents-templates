## 17. Docker & Compose — Hardened (Static) — Conditional

> Read this when adding Docker/nginx or deploying to Vercel/Netlify/Cloudflare Pages — check hosting type first.

> **Use only if the hosting is a self-managed container (own VPS, own Docker host, ECS, etc.).**
> If the project is deployed to **Vercel / Netlify / Cloudflare Pages** (or any static hosting that builds from `pnpm build` and serves `dist/` natively), **skip this entire section** and use the provider's native config (`vercel.json` / `netlify.toml` / `_headers` & `_redirects` / `wrangler.toml`). Do not add a `Dockerfile`, `nginx.conf`, or `docker-compose.yml` in that case — they will confuse the agent and bloat the repo.
>
> **Decision rule for the agent:** Check for an existing `Dockerfile` / `docker-compose.yml` or an explicit hosting decision in §0 / `README.md`. If none exists and the hosting is Vercel/Netlify/Cloudflare Pages, do not create Docker files. If hosting is self-managed, apply the hardened static setup below.

The frontend is a static build served via `nginx:alpine` (or `caddy`). Do not use `node` in production for static assets. This subsection applies only to the self-hosted container case.

### `Dockerfile` (multi-stage, pnpm, non-root)

```dockerfile
FROM node:22-alpine AS build
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# Vite needs env at build time — pass via --build-arg or env_file
ARG VITE_API_URL
ENV VITE_API_URL=$VITE_API_URL
RUN pnpm build

FROM nginx:1.27-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
USER appuser
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### `nginx.conf` (SPA fallback + security headers)

```nginx
server {
  listen 80;
  root /usr/share/nginx/html;
  index index.html;
  add_header X-Frame-Options "DENY" always;
  add_header X-Content-Type-Options "nosniff" always;
  add_header Referrer-Policy "strict-origin-when-cross-origin" always;
  add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self'; connect-src 'self' https:;" always;
  location / { try_files $uri /index.html; }
  location /assets/ { expires 1y; add_header Cache-Control "public, immutable"; }
}
```

### `docker-compose.yml`

```yaml
services:
  web:
    build:
      context: .
      args:
        VITE_API_URL: ${VITE_API_URL}
    ports: ["${PORT:-3000}:80"]
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-qO-", "http://localhost/"]
      interval: 30s
      timeout: 5s
      retries: 3
volumes: {}
```

Rules (self-hosted only): never bake `.env` into the image beyond `VITE_` build args; use compose `args` or CI secrets. `.dockerignore` must exclude `node_modules`, `dist`, `.git`, `.env`, `coverage`. If runtime env without rebuilding is needed, inject `window.__ENV__` via an entrypoint script that templates `env.js`.

### When Deployed to Vercel / Netlify / Cloudflare Pages — Skip Docker

Use the provider's native config instead:

- **Vercel:** `vercel.json` for rewrites/headers (`{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }`) and env vars in the dashboard. No `Dockerfile`.
- **Netlify:** `netlify.toml` (`[[redirects]] from="/*" to="/index.html" status=200`) + `_headers` for `Content-Security-Policy` / `X-Frame-Options`. Set `VITE_*` in Netlify env UI.
- **Cloudflare Pages:** `_headers` and `_redirects` (`/* /index.html 200`) or `wrangler.toml` / Functions for headers. Set `VITE_*` in Pages env vars.

In all three cases: `pnpm build` outputs `dist/` and the platform serves it with SPA fallback — no `nginx.conf` needed. Replicate the security headers from the `nginx.conf` example above using the provider's header mechanism.

---
