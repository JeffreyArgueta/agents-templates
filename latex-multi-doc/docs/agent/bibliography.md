## Bibliography — biblatex+biber (default) vs none

> Read this when the task touches `bib/*`, `\cite`, `\printbibliography`, or any biber/bibtex error.

Default is `biblatex+biber`. Never mix `bibtex` + `biblatex` in one document.

### A. biblatex+biber (default)

`preamble/packages.tex`:

```tex
\usepackage{csquotes} % required by biblatex for contextual quotes
\usepackage[backend=biber,style=numeric,sorting=nyt]{biblatex}
\addbibresource{bib/references.bib} % path relative to main.tex
```

> **Note on order:** Always load `csquotes` *before* `biblatex`. If using `hyperref` + `cleveref`, load `biblatex` before `hyperref`, and `cleveref` last (see canonical preamble in `engines-languages.md`).

End of `main.tex` (before `\end{document}`):

```tex
\printbibliography
```

`bib/references.bib` (always ship one compiling entry):

```bibtex
@book{knuth:1984:texbook,
  author    = {Donald E. Knuth},
  title     = {The TeXbook},
  year      = {1984},
  publisher = {Addison-Wesley},
}
```

Use in body: `According to \textcite{knuth:1984:texbook}.` Key format `author:year:keyword`.

Build: `latexmk -pdf main.tex` runs `biber` automatically when `latexmkrc` sets `$biber`. Manual fallback: `pdflatex main && biber main && pdflatex main && pdflatex main`. Tectonic's Biber support depends on the selected interface and installed helper; verify that `biber` or `tectonic-biber` is available before choosing the Tectonic profile.

Styles: `numeric` default; `authoryear` for humanities/thesis if §0 says so. Record non-default style in §0.

### B. none

When §0 bibliography=`none` (beamer/letter/cv/poster often):

- Delete `bib/` entirely.
- No `\usepackage{biblatex}`, no `\addbibresource`, no `\printbibliography`, no `\cite` — plain text instead.
- `latexmkrc` still works (biber line is a no-op); do not add bib-specific config.

### Switching later

`none` → `biblatex`: create `bib/references.bib`, add the two preamble lines + `\printbibliography`, confirm `latexmkrc` has `$biber`. Reverse: delete all three + the dir. Rebuild clean (`latexmk -C && latexmk -pdf main.tex`).
