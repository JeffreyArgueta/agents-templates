## Engines & Languages — pdflatex vs xelatex/lualatex, babel/polyglossia

> Read this when the task touches the TeX engine, `fontspec`, unicode fonts, `babel`/`polyglossia`, or non-English text.

Default: `pdflatex` + `babel english`. Change only per §0.

### Engine matrix

| Engine | When | `latexmkrc` delta | Constraint |
|--------|------|-------------------|------------|
| `pdflatex` (default) | plain English docs, max Overleaf/journal compat | `$pdf_mode = 1;` | No `fontspec`, no system OTF fonts |
| `xelatex` | unicode, OpenType, non-Latin scripts | `$pdf_mode = 5; $xelatex = "xelatex -interaction=nonstopmode -halt-on-error -synctex=1 %O %S";` | Use `polyglossia` or `babel`; `fontspec` allowed |
| `lualatex` | like xelatex + Lua scripting, complex typography | `$pdf_mode = 4; $lualatex = "lualatex -interaction=nonstopmode -halt-on-error -synctex=1 %O %S";` | Slower; same `fontspec` rule |

Tectonic is XeTeX-based: treat as `xelatex` row for font decisions.

### Canonical Preamble (`preamble/packages.tex`)

Every standard document (`article`, `report`, `book`, `thesis`) should use this ordered baseline to prevent package collision warnings:

```tex
% 1. Encoding & Language (engine-dependent)
\usepackage[utf8]{inputenc} % pdflatex only; omit under xelatex/lualatex
\usepackage[T1]{fontenc}    % pdflatex only; omit under xelatex/lualatex
\usepackage[english]{babel}
\usepackage{lmodern}

% 2. Typography & Formatting
\usepackage{microtype}      % improved spacing and kerning
\usepackage{csquotes}       % required by biblatex for correct quotes

% 3. Graphics & Tables
\usepackage{graphicx}       % figures
\usepackage{booktabs}       % professional tables (\toprule, \midrule, \bottomrule)

% 4. Bibliography (if §0 bibliography=biblatex+biber)
\usepackage[backend=biber,style=numeric,sorting=nyt]{biblatex}
\addbibresource{bib/references.bib}

% 5. Hyperlinks & References (ALWAYS LAST TWO)
\usepackage[colorlinks=true,allcolors=blue]{hyperref}
\usepackage{cleveref}       % intelligent \cref; MUST be loaded after hyperref
```

Other language (example Spanish):

```tex
\usepackage[spanish]{babel}
```

`xelatex`/`lualatex` + fonts:

```tex
\usepackage{fontspec}
\setmainfont{Latin Modern Roman} % or any installed OpenType
\usepackage[english]{babel}      % or: \usepackage{polyglossia}\setdefaultlanguage{english}
% then typography, graphics, bib, hyperref, cleveref as above
```

### Rules

- Never load `fontspec` under `pdflatex` — it hard-fails. Switching engine means editing `latexmkrc` `$pdf_mode` + preamble together.
- `inputenc` only for `pdflatex`; drop it under `xelatex`/`lualatex` (native UTF-8).
- Record engine + language in §0; CI image must match engine (see `ci.md`).
