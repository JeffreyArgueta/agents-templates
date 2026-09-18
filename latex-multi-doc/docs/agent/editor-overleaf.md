## Editor & Overleaf — Workshop + Sync-Safe Layout

> Read this when the task touches `.vscode/settings.json`, Workshop recipes, or Overleaf import/sync.

### VS Code LaTeX Workshop (recommended local)

`.vscode/settings.json` — commit it:

```json
{
  "latex-workshop.latex.recipe.default": "latexmk",
  "latex-workshop.latex.recipes": [{ "name": "latexmk", "tools": ["latexmk"] }],
  "latex-workshop.latex.tools": [
    {
      "name": "latexmk",
      "command": "latexmk",
      "args": ["-synctex=1", "-interaction=nonstopmode", "-file-line-error", "-pdf", "-outdir=%OUTDIR%", "%DOC%"]
    }
  ],
  "latex-workshop.latex.outDir": "build",
  "latex-workshop.latex.rootFile": "main.tex",
  "latex-workshop.synctex.afterBuild.enabled": true
}
```

Tectonic variant: add a second tool `{"name": "tectonic", "command": "tectonic", "args": ["-o", "build", "%DOC%"]}` and set it default only when §0 build=`tectonic`. XeLaTeX/LuaLaTeX: add `-xelatex` / `-lualatex` flag variant matching `$pdf_mode`.

Root-file rule: `main.tex` (with `\documentclass`) lives at repo root. Workshop auto-detects it; `latex-workshop.latex.rootFile` pins it for subfile builds. `$do_cd = 1;` in `latexmkrc` keeps `\input` paths root-relative.

### Overleaf-compatible layout (must hold for every doctype)

- `main.tex` at root, all `\input`/`\includegraphics`/`\addbibresource` paths relative (`chapters/01-x`, `figures/y`, `bib/references.bib`). No absolute paths, no `../`.
- Commit `latexmkrc` — Overleaf runs `latexmk` and reads it (engine, `$biber`, `$out_dir` ignored server-side but harmless).
- `Tectonic.toml` is ignored by Overleaf — keep the `latexmkrc` fallback even for Tectonic docs.
- Zip-upload safe: no `build/` contents, no symlinks, UTF-8 `.tex`/`.bib`.
- Sync check: after `latexmk -C && latexmk -pdf main.tex` passes locally, the same tree uploads and compiles on Overleaf without edits.
