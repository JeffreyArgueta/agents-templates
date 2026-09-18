# AGENTS.md — Reusable LaTeX Template (Layer 1 — Always Loaded)

> Generic architecture for multi-type LaTeX documents (latexmk default, TeX Live or Tectonic).
> Copy this folder for new documents and edit only section 0. Layer 1 is the only file the agent always reads; thematic detail lives in `docs/agent/*.md` and is opened on demand.

## Contents

- [0. Project Context](#0-project-context-edit-per-project) — fill once per document
- [1. How the Agent Should Reason](#1-how-the-agent-should-reason)
- [2. Folder Structure](#2-folder-structure)
- [3. Naming Conventions](#3-naming-conventions)
- [4. Thematic Docs Index — When to Read What](#4-thematic-docs-index--when-to-read-what)
- [5. Rules for Agents](#5-rules-for-agents) — always enforced

---

## 0. Project Context (EDIT per project)

- **Name:** `<document-name>`
- **Type:** LaTeX (multi-type: article / report / book / thesis / beamer / letter / cv / poster)
- **Engine:** `pdflatex` (default) or `xelatex` / `lualatex` — see `docs/agent/engines-languages.md`
- **Language:** `english` via `babel` (default) or other — see `docs/agent/engines-languages.md`
- **Bibliography:** `biblatex+biber` (default) or `none` — see `docs/agent/bibliography.md`
- **Build:** `latexmk` (default) or `tectonic` / `manual` — see `docs/agent/build.md`
- **CI:** `yes` (PDF on push) / `no` — see `docs/agent/ci.md`
- **Stack decisions — fill before writing (only `latexmk` + `biblatex+biber` + `english` + `pdflatex` are defaults):**

  | Area | Options (pick one) | Chosen |
  |------|--------------------|--------|
  | Doctype | `article` / `report` / `book` / `thesis` / `beamer` / `letter` / `cv` / `poster` | `<choose>` |
  | Bibliography | `biblatex+biber` / `none` | `<choose>` |
  | Build | `latexmk` / `tectonic` / `manual` | `<choose>` |
  | Engine | `pdflatex` / `xelatex` / `lualatex` | `<choose>` |
  | Language | `english` / other `babel` code | `<choose>` |
  | CI | `yes` / `no` | `<choose>` |

- **Key commands:** `latexmk -pdf main.tex` · `latexmk -pvc -pdf main.tex` · `latexmk -C` · `chktex main.tex` · `make` / `make watch` / `make clean` (if Makefile kept)

> Record the choice in the table and follow it consistently. Do not mix two build engines for the same document without an explicit decision. If §0 is blank, run `docs/agent/setup.md` first.

---

## 1. How the Agent Should Reason

- Before writing, **read the relevant Layer 2 file(s)** per §4 and copy the snippet style — never invent a preamble from memory when a snippet exists.
- If §0 is unfilled, run the guided setup in `docs/agent/setup.md`: ask the 6 questions, then scaffold **only** the chosen doctype's files.
- If a task is ambiguous, pick the most reasonable interpretation and state the assumption.
- Never introduce a new package/class without checking it exists in the chosen engine (TeX Live vs Tectonic bundle) — propose first.
- Do not modify build config (`latexmkrc`, `Tectonic.toml`, CI) without calling it out.
- Keep changes scoped; do not reformat unrelated chapters or switch bibliography engines opportunistically.
- When a document already has a different convention, follow the document and note the divergence.

**LaTeX essentials:** `main.tex` (with `\documentclass`) at repo root is the single entrypoint — Overleaf, Workshop, and CI all use it. Multi-file splits use `\input{}` (sections) / `\include{}` (chapters, page-break). Build outputs go to `build/` via `-outdir`; never commit them.

---

## 2. Folder Structure

Template repo (before setup):

```
.
  AGENTS.md  docs/agent/
```

Scaffolded document (after setup — only chosen type's files):

```
.
  main.tex  latexmkrc  Makefile  Tectonic.toml*  .chktexrc  .gitignore
  .vscode/settings.json  .github/workflows/build-pdf.yml*
  preamble/{packages.tex,macros.tex}  sections/|chapters/  figures/  bib/references.bib  build/
```

`*` conditional: `Tectonic.toml` only if build=`tectonic`; workflow only if CI=`yes`. See `docs/agent/setup.md` for the per-doctype tree (`thesis` uses `chapters/`, `article` uses `sections/`, `beamer` is single-file + `figures/`).

**Layer rule:** `main.tex → preamble/* → content/* → bib/figures → build/` · `preamble ↑ bib ↑ figures`. Content files never set `\documentclass` or load packages; `preamble/` owns all `\usepackage`; `bib/` owns all citation data. No content file imports another content file's internals.

---

## 3. Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Entry point | `main.tex` at root (fixed) | `main.tex` |
| Preamble | `preamble/packages.tex`, `preamble/macros.tex` | `\input{preamble/packages}` |
| Chapters | kebab-case `NN-name.tex` under `chapters/` | `chapters/01-introduction.tex` |
| Sections | kebab-case under `sections/` | `sections/01-intro.tex` |
| Figures | kebab-case, vector first | `figures/architecture.pdf`, `figures/result.png` |
| Bibliography | single `bib/references.bib` | `\addbibresource{bib/references.bib}` |
| Build dir | `build/` (gitignored) | `latexmk -outdir=build` |
| Labels | `prefix:key` | `fig:arch`, `tab:results`, `sec:intro`, `eq:emc2` |

Primitives: one `\section`/`\chapter` per file, `\label` after `\caption`/`\section`, UTF-8 only, no absolute paths. Citation keys are `author:year:keyword` (e.g. `knuth:1984:texbook`).

---

## 4. Thematic Docs Index — When to Read What

The agent **always** reads this file. It **only** opens a file below when the task touches that area. The trigger column is the decision signal — not just a link.

| Task touches… | Read this | Trigger |
|---------------|-----------|---------|
| First run, blank §0, new document | `docs/agent/setup.md` | Any scaffold, doctype question, unfilled §0 |
| Doctype structure, class, per-type tree | `docs/agent/document-types.md` | `article`/`report`/`book`/`thesis`/`beamer`/`letter`/`cv`/`poster`, `\documentclass`, new chapter/slide |
| Citations, `.bib`, biber vs none | `docs/agent/bibliography.md` | `bib/*`, `\cite`, `\printbibliography`, biber errors |
| Build driver, `latexmkrc`, Makefile, Tectonic, manual chain | `docs/agent/build.md` | `latexmkrc`, `Makefile`, `Tectonic.toml`, build failure, `-outdir` |
| Engine (`pdflatex`/`xelatex`/`lualatex`), `babel`/`polyglossia`, fonts | `docs/agent/engines-languages.md` | `\usepackage[english]{babel}`, `fontspec`, unicode, engine switch |
| VS Code Workshop, Overleaf sync | `docs/agent/editor-overleaf.md` | `.vscode/settings.json`, Overleaf import, root-file detection |
| Figures, tables, TikZ, image formats | `docs/agent/figures-tables.md` | `figures/*`, `\includegraphics`, `tikz`, float placement |
| `chktex`, `.chktexrc`, `.gitignore`, aux files | `docs/agent/lint-format.md` | `chktex`, `*.aux` committed, lint warnings |
| PDF-on-push workflow, TeX Live vs Tectonic CI | `docs/agent/ci.md` | `.github/workflows/*`, CI build failure |
| Commits, branches, PRs | `docs/agent/git-workflow.md` | Commits, PR checklist |
| Canonical `latexmk`/`make` commands | `docs/agent/commands.md` | Any build/clean/watch task |

**How to use:** If your task is "add a chapter", you read this file + `docs/agent/document-types.md` only. If it is "fix citations", you add `docs/agent/bibliography.md` + `docs/agent/build.md`. You still have 100% of the detail — you just pay the context cost only when warranted.

**Tooling note:** Some environments auto-read `AGENTS.md` but do not auto-open referenced files. The agent must have `read_file` / `view` access and use the relative paths above (`docs/agent/<topic>.md` from the folder that contains this `AGENTS.md`). Verify the tool can open them before assuming.

---

## 5. Rules for Agents

1. **Source of truth:** `main.tex` for class, `preamble/` for packages, `latexmkrc` for build. Verify names before assuming.
2. **Guided setup first:** never scaffold blindly — ask per `docs/agent/setup.md`, then write only the chosen type's files and delete the conditionals you did not pick.
3. **Respect layers:** content files contain body only — no `\documentclass`, no `\usepackage` outside `preamble/`.
4. **English default:** body, labels, keys, comments in English unless §0 language says otherwise. File names are kebab-case, ASCII only.
5. **Build gate:** `latexmk -pdf main.tex` from a clean tree (`latexmk -C` first) must pass before push; `chktex main.tex` must be warning-clean or warnings justified.
6. **Never commit build outputs:** `build/`, `*.aux`, `*.bbl`, `*.bcf`, `*.blg`, `*.log`, `*.out`, `*.toc`, `*.synctex.gz` are gitignored — see `docs/agent/lint-format.md`.
7. **Bibliography:** follow §0 — `biblatex+biber` (`\addbibresource`, `\printbibliography`, `latexmk` runs biber) or `none` (no `bib/` dir, no citation commands). Never mix `bibtex` + `biblatex`.
8. **One engine:** follow the §0 engine; `fontspec`/`polyglossia` only under `xelatex`/`lualatex`. Never add `-shell-escape` without calling it out.
9. **Overleaf-safe:** `main.tex` at root, relative paths only, commit `latexmkrc`, no absolute `-outdir` outside `build/`.
10. **CI:** if §0 CI=`yes`, keep `.github/workflows/build-pdf.yml` green; if `no`, the file must not exist.
11. **Scoped changes:** one chapter/section per edit; extract `\newcommand` to `preamble/macros.tex` before duplicating blocks longer than ~10 lines.

---

## Maintenance — Keeping the Two Layers in Sync

- The index table in §4 is the single source of truth for thematic docs — add/remove a row whenever you add/remove a file in `docs/agent/`.
- Each `docs/agent/*.md` keeps its `> Read this when…` line at the top; it is the trigger that tells the agent whether to open it.
- Do not summarize long sections into Layer 1 — keep code examples in Layer 2 and reference them from the index.
- Verify the agent has `read_file`/`view` access and that relative paths `docs/agent/<topic>.md` resolve from this file's directory.
