## 19. Useful Commands

> Read this when you need the canonical pnpm commands for dev, build, test, or Docker.

```bash
pnpm install                 # install (frozen lockfile in CI)
pnpm dev                     # vite --host (HMR, LAN preview)
pnpm build                   # vite build (production)
pnpm preview                 # vite preview dist/
pnpm lint                    # eslint check
pnpm lint:fix                # eslint autofix
pnpm format                  # prettier write
pnpm format:check            # prettier check (CI)
pnpm test                    # run tests (vitest or chosen runner)
pnpm test:watch              # watch mode
pnpm test:coverage           # coverage + threshold
pnpm test:e2e                # e2e (playwright or chosen runner)
pnpm audit                   # vulnerability audit
pnpm typecheck               # tsc --noEmit (if TS)
pnpm storybook               # storybook dev (if installed)
pnpm build-storybook         # storybook static build

# Quick layer verification (no server needed, ESM)
node --check src/services/http.js
node --check src/config/env.js
node -e "import('./src/lib/cn.js').then(m=>console.log('cn:', typeof m.cn))"

# Docker
docker compose up --build
docker compose logs -f web
docker compose down

# Bundle analysis (if configured)
pnpm build -- --analyze
```

---
