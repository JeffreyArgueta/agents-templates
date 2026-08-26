## 16. Documentation

> Read this when writing README, Storybook, or ADRs.

- **Component docs (optional):** when enabled in §0, use Storybook — `pnpm dlx storybook@latest init` — with a story per `components/ui/*` primitive showing variants, states (default, disabled, error), and accessibility notes. Serve in development and build statically for hosting.
- **README.md** should contain:
  - Features and requirements (Node version, `pnpm`, quick start).
  - Folder map with `features/` vs `components/ui/`.
  - Env vars table (`VITE_API_URL` and any project-specific vars).
  - Tailwind tokens table (colors, radii, fonts) from `tailwind.config.js`.
  - API contracts (endpoints, payloads, error shapes).
  - Integration or migration notes (how to embed the app or a feature into another host).
- **ADRs (optional):** one markdown per major decision (`docs/adr/001-alias.md`, `002-form-library.md`) when working cross-team.

---
