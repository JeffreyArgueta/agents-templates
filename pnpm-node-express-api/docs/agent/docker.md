## 17. Docker & Compose — Hardened

> Read this when changing Dockerfile, docker-compose, or deployment — check hosting type first.

### Dockerfile (multi-stage, pnpm, non-root)
```dockerfile
FROM node:22-alpine AS base
RUN corepack enable && corepack prepare pnpm@latest --activate
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . .
# RUN pnpm run build # if TS

FROM node:22-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
WORKDIR /app
COPY --from=base /app/node_modules ./node_modules
COPY --from=base /app/src ./src
COPY --from=base /app/package.json ./
COPY --from=base /app/database ./database
USER appuser
EXPOSE 3000
CMD ["node", "src/server.js"]
```

### docker-compose.yml (healthcheck + depends_on condition)
```yaml
services:
  api:
    build: .
    ports: ["${PORT:-3000}:3000"]
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    volumes: ["./logs:/app/logs"]
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "node", "-e", "fetch('http://localhost:3000/health').then(r=>{if(!r.ok)throw new Error('unhealthy')})"]
      interval: 30s
      timeout: 5s
      retries: 3
  db:
    image: mysql:8.4
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_PASSWORD}
      MYSQL_DATABASE: ${DB_NAME}
      MYSQL_USER: ${DB_USER}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    ports: ["3306:3306"]
    volumes: ["db_data:/var/lib/mysql"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${DB_PASSWORD}"]
      interval: 10s
      timeout: 5s
      retries: 5
volumes: { db_data: {} }
```

### docker-compose.override.yml (dev only, not committed or gitignored exception)
```yaml
services:
  api:
    volumes: [".:/app", "/app/node_modules"]
    command: pnpm run dev
    environment: [NODE_ENV=development]
```

- **Rules:** never bake `.env` into image; use `env_file` or Docker secrets. Run migrations via `command: sh -c "pnpm run db:migrate && node src/server.js"` or an init container. `.dockerignore` must exclude `node_modules`, `logs`, `.git`, `.env`, `coverage`.
- **Override:** `docker compose up` merges `docker-compose.yml` + `docker-compose.override.yml` automatically — keep production compose clean.

---
