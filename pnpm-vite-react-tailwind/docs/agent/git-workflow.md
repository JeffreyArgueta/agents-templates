## 18. Git Workflow — Conventional Commits

> Read this when committing, branching, opening a PR, or versioning.

- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/) — `type(scope): subject`
  - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
  - Examples: `feat(auth): add login route`, `fix(a11y): add aria-describedby to text field`, `docs(readme): update env table`
- **Branches:** `main` (protected), `feat/<name>`, `fix/<name>`, `chore/<name>`. No direct push to `main` — PR required.
- **PR checklist:** lint, format, tests, and build pass; no secrets; `pnpm audit` clean; automated a11y passes on changed routes; screenshots for UI changes.
- **Versioning:** `feat` → minor, `fix` → patch, `BREAKING CHANGE:` or `feat!:` → major (bump `package.json` and `CHANGELOG.md` if maintained).

---
