## Lint & Format — chktex, .chktexrc, .gitignore

> Read this when the task touches `chktex`, `.chktexrc`, `.gitignore`, or committed aux/log files.

### chktex (lint gate)

```bash
chktex main.tex                 # lint entrypoint (non-verbose, must be clean)
chktex -q main.tex              # quiet: warnings only
```

`.chktexrc` — commit it (sane defaults, silence false-positive `:` in `\ref` ranges):

```
# Warn on common faults, allow double spaces after periods
CmdLine { --verbosity 2 }
Warn { STDERR }
```

Fix order: errors (unmatched braces, missing `\end`) → warnings (`\label` after `\caption`, interword spacing) → style. Justify any suppressed warning inline with `% chktex 13` + comment.

No auto-formatter is canonical for LaTeX (unlike prettier). Indent 2 spaces, one sentence per line for clean diffs, UTF-8, LF, no trailing whitespace.

### .editorconfig (copy exactly — always scaffolded)

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.{tex,bib}]
max_line_length = off

[*.md]
trim_trailing_whitespace = false
```

### .gitattributes (copy exactly — always scaffolded)

```gitattributes
# Consistent line endings
* text=auto eol=lf

# LaTeX text sources
*.tex text eol=lf
*.bib text eol=lf
*.sty text eol=lf
*.cls text eol=lf
latexmkrc text eol=lf
Makefile text eol=lf

# Binary media
*.pdf binary
*.png binary
*.jpg binary
*.jpeg binary

# Treat figure PDFs as binary (no merge conflicts on diagram rebuilds)
figures/*.pdf -diff
```

### .gitignore (copy exactly — always scaffolded)

```gitignore
# LaTeX build artifacts (latexmk -C proves clean without them)
build/
main.pdf
*.aux *.bbl *.bcf *.blg *.fdb_latexmk *.fls
*.log *.out *.toc *.lot *.lof *.idx *.ind *.ilg
*.synctex.gz *.run.xml *.nav *.snm *.vrb
*.dvi *.ps

# Tectonic
*.pdf.log _tectonic/

# Editors / OS
.vscode/*.log .DS_Store Thumbs.db
```

### What NOT to commit

`build/` and every `*.aux *.bbl *.bcf *.blg *.log *.out *.toc *.synctex.gz` — plus the compiled `main.pdf` unless the release process explicitly versions it. If an aux file is already tracked: `git rm --cached <file>`, keep `.gitignore`, rebuild clean.

Overleaf note: it regenerates aux server-side; committing them only causes sync conflicts.
