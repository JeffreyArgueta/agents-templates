## 19. Useful Commands

> Read this when you need the canonical pnpm commands for db, lint, test, or Docker.

```bash
pnpm run dev              # nodemon src/server.js
pnpm start                # node src/server.js
pnpm run db:create        # create database
pnpm run db:seed          # seed catalogs
pnpm run db:migrate       # run migrations
pnpm run db:migrate:undo  # undo last migration
pnpm run lint             # eslint check
pnpm run lint:fix         # eslint autofix
pnpm run format           # prettier write
pnpm run format:check     # prettier check
pnpm test                 # all tests
pnpm run test:unit        # unit only
pnpm run test:integration # integration only
pnpm audit                # check vulnerabilities (see §10)

# Quick layer verification (no DB needed, ESM)
node -e "import('./src/models/index.js').then(()=>console.log('models OK'))"
node -e "import('./src/repositories/index.js').then(()=>console.log('repositories OK'))"
node -e "import('./src/services/index.js').then(()=>console.log('services OK'))"
node -e "import('./src/controllers/index.js').then(()=>console.log('controllers OK'))"

# Docker
docker compose up --build       # build + run api + db (healthchecks)
docker compose logs -f api
docker compose down -v          # stop + remove volumes

# Syntax check
find src -name '*.js' -exec node --check {} \;
```

---
