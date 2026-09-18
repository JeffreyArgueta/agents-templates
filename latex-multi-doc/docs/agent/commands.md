## Useful Commands

> Read this when you need the canonical build, watch, clean, lint, or verification commands.

```bash
latexmk -pdf main.tex            # canonical build (latexmk default)
latexmk -pvc -pdf main.tex       # watch mode while writing
latexmk -C                       # remove generated files (clean-build proof)
chktex main.tex                  # lint (must be clean)
chktex -q main.tex               # lint, warnings only

# Draft a report/book/thesis with selected chapters (set \includeonly in main.tex)
latexmk -pdf main.tex

make                             # = latexmk -pdf main.tex (if Makefile kept)
make watch                       # = latexmk -pvc -pdf main.tex
make clean                       # = latexmk -C

# Tectonic profile (only when §0 build=tectonic)
tectonic -o build main.tex
# Manual profile (only when §0 build=manual)
pdflatex main.tex && biber main && pdflatex main.tex && pdflatex main.tex
pdflatex main.tex && pdflatex main.tex   # bib=none variant

# Quick verification (no PDF needed)
chktex -q main.tex && echo OK
ls main.tex latexmkrc .gitignore preamble/  # scaffold sanity
git status --short               # must show no build/ or *.aux tracked
```

Workshop builds via `Ctrl+Alt+B` (recipe `latexmk` from `editor-overleaf.md`); output lands in `build/` with SyncTeX enabled.
