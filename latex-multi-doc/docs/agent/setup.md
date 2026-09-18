## Setup — Guided Scaffold

> Read this when §0 is blank, the user starts a new document, or any scaffold/doctype question comes up.

Ask in this order. Defaults apply if the user says "default" or skips. Ask only what materially changes structure.

### 1. Questions (ask all 6, compact)

1. **Doctype?** `article` / `report` / `book` / `thesis` / `beamer` / `letter` / `cv` / `poster`
2. **Bibliography?** `biblatex+biber` (default) / `none`
3. **Build?** `latexmk` (default) / `tectonic` / `manual`
4. **Engine?** `pdflatex` (default) / `xelatex` / `lualatex` — if `xelatex`/`lualatex` needed (unicode/OpenType fonts), note it now
5. **Language?** `english` (default, `babel`) / other (give `babel` code, e.g. `spanish`, `french`)
6. **CI?** `yes` (PDF on push) / `no`

Record answers in `AGENTS.md` §0 table before writing any `.tex`.

### 2. Scaffold matrix

| Doctype | Entry | Content dirs | Preamble delta (see `document-types.md`) |
|---------|-------|--------------|------------------------------------------|
| `article` | `main.tex` | `sections/`, `figures/`, `bib/` | `article` class, `biblatex` if bib |
| `report` | `main.tex` | `chapters/`, `figures/`, `bib/` | `report` class, `\include` chapters |
| `book` | `main.tex` | `chapters/`, `frontmatter/`, `figures/`, `bib/` | `book` class, `\frontmatter`/`\mainmatter` |
| `thesis` | `main.tex` | `frontmatter/`, `chapters/`, `appendices/`, `figures/`, `bib/` | `report` or `book` + title/abstract/declaration |
| `beamer` | `main.tex` (single-file) | `figures/` | `beamer` class, frames; no `chapters/` |
| `letter` | `main.tex` (single-file) | — | `letter` class, `\address`/`\signature` |
| `cv` | `main.tex` (single-file) | — | `article` + `moderncv`-style sectioning (no external class dependency by default) |
| `poster` | `main.tex` (single-file) | `figures/` | `tikzposter` default; `beamerposter` alt |

Content-file rule: `article` → `sections/NN-name.tex` with `\input`; `report`/`book`/`thesis` → `chapters/NN-name.tex` with `\include`; others → body inline in `main.tex`.

### 3. Write order

1. `AGENTS.md` §0 (record choices).
2. `.gitignore` + `.gitattributes` + `.editorconfig` + `.chktexrc` (from `lint-format.md`) — always.
3. `latexmkrc` + `Makefile` if build=`latexmk`; `Tectonic.toml` (+ minimal `latexmkrc` for Overleaf fallback) if `tectonic`; nothing extra if `manual` (see `build.md`).
4. `preamble/packages.tex` + `preamble/macros.tex` (engine + language + bib per `engines-languages.md` / `bibliography.md`).
5. `main.tex` + content files (per `document-types.md`) with placeholder title/author/section/chapter + one sample `\cite` if bib enabled.
6. `bib/references.bib` if bib, else no `bib/` dir and no citation commands.
7. `.vscode/settings.json` (from `editor-overleaf.md`) — always.
8. `.github/workflows/build-pdf.yml` if CI=`yes`, else ensure it does not exist (see `ci.md`).

### 4. Prune rule

The finished repo holds **only** the chosen type's files. Delete: unchosen `chapters/` vs `sections/`, `Tectonic.toml` when build≠`tectonic`, workflow when CI=`no`, `bib/` when bib=`none`.

### 5. Verify (must pass before done)

```bash
latexmk -C && latexmk -pdf main.tex   # clean build proves no stale aux dependence
chktex main.tex
```

If build=`tectonic`: `tectonic -o build main.tex`. If `manual`: `pdflatex main.tex && biber main` (if bib) `&& pdflatex main.tex && pdflatex main.tex`.
