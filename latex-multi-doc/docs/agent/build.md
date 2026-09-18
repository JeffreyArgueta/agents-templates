## Build — latexmk (default) vs tectonic vs manual

> Read this when the task touches `latexmkrc`, `Makefile`, `Tectonic.toml`, `-outdir`, or any build failure.

One build driver per document (§0). `latexmk` is default; keep the others dormant/deleted.

### A. latexmk (default, TeX Live + Overleaf-safe)

`latexmkrc` — single build entrypoint, commit it:

```perl
$pdf_mode = 1;              # pdflatex -> pdf (use 5 for xelatex, see engines-languages.md)
$out_dir = "build";
$aux_dir = "build";
$biber = "biber %O %S";
$pdflatex = "pdflatex -interaction=nonstopmode -halt-on-error -synctex=1 %O %S";
$do_cd = 1;                 # Workshop builds from subfiles correctly
push @generated_exts, "synctex.gz", "run.xml";
```

Exact invocations:

```bash
latexmk -pdf main.tex          # clean/CI build
latexmk -pvc -pdf main.tex     # watch mode while writing
latexmk -C                     # remove generated files, proving no stale dependence
```

`Makefile` (thin wrapper, optional but recommended):

```make
.PHONY: all watch clean
all: ; latexmk -pdf main.tex
watch: ; latexmk -pvc -pdf main.tex
clean: ; latexmk -C
```

### B. tectonic (self-contained alt)

When §0 build=`tectonic` (small Docker/CI, auto-fetch packages, XeTeX-based):

`Tectonic.toml` (the capitalized filename is required by Tectonic's V2 interface):

```toml
[doc]
name = "<document-name>"
bundle = "https://data1.fullyjustified.net/tle-2024.tar"
[[output]]
name = "default"
type = "pdf"
```

```bash
tectonic -o build main.tex            # single invocation, reruns internally
```

Notes: no intermediate aux in repo; keep a minimal `latexmkrc` alongside for Overleaf fallback (Overleaf ignores `Tectonic.toml`). `fontspec` works (XeTeX base). For `biblatex`, make `biber` (or a compatible `tectonic-biber` helper for the V2 interface) available and verify it with the selected Tectonic version; `tectonic -o build main.tex` is not a substitute for an installed Biber executable. Delete `Makefile` latexmk targets or retarget `all: ; tectonic -o build main.tex`.

### C. manual (no rc/Makefile, journal-forced only)

No `latexmkrc`, no `Makefile`. Document the chain in `commands.md`:

```bash
pdflatex main.tex && biber main && pdflatex main.tex && pdflatex main.tex  # with bib
pdflatex main.tex && pdflatex main.tex                                      # without bib
```

Prefer `latexmk` unless the venue forbids it — manual chains miss reruns and break Workshop/CI uniformity.

### Build failures checklist

1. Wrong dir? Build from root where `main.tex` + `latexmkrc` live.
2. Biber stale? `latexmk -C`, rebuild; check `\addbibresource` path.
3. Engine mismatch? `$pdf_mode` vs `\usepackage{fontspec}` (see `engines-languages.md`).
4. Missing `-outdir`? Workshop must pass `-outdir=%OUTDIR%` or outputs pollute root.
