## 18. Git Workflow — Conventional Commits

> Read this when committing, branching, opening a PR, or versioning.

- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/) — same as this repository. Format: `<type>(<scope>): <subject>`
  - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
  - Examples: `feat(auth): add cookie-based login`, `fix(pagination): handle cursor edge case`, `docs(api): update v1 deprecation note`
- **Branches:** `main` (protected), `feat/<name>`, `fix/<name>`, `chore/<name>`. No direct push to `main` — PR required.
- **PR checklist:** lint + format + tests pass, migrations reviewed, Swagger updated, no secrets, `pnpm audit` clean.
- **Versioning:** `feat` → minor, `fix` → patch, `BREAKING CHANGE:` footer or `feat!:` → major (bump `package.json` + Swagger version + `src/routes/v2` if needed).

---
