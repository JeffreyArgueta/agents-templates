## Git Workflow — Commits, Branches, PRs

> Read this when the task touches commits, branches, PRs, or release PDFs.

- Branches: `main` (compiling HEAD) + `draft/<topic>` for edits. Never push a non-compiling `main` — `latexmk -pdf main.tex` from clean must pass.
- Commits: `<area>: <what>` — e.g. `chapters: add method section`, `preamble: switch to authoryear`, `build: fix biber path`. One chapter/idea per commit.
- Before push: `latexmk -C && latexmk -pdf main.tex` + `chktex main.tex` + `git status --short` (no `build/`, `*.aux`, `*.log`, `*.synctex.gz` tracked).
- PRs: state §0 choices touched (doctype/bib/build/engine/language/CI), the exact build command run, and any new package with why + engine compat (TeX Live vs Tectonic).
- Never commit secrets (Overleaf tokens, registry creds) or compiled `main.pdf` unless the release flow versions it — CI artifacts cover review.
- `.gitignore` + `latexmkrc` changes are sensitive: call them out in the PR body like DB migrations in backend templates.
