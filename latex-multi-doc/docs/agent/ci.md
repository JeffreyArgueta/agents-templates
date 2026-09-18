## CI — PDF on Push (conditional)

> Read this when the task touches `.github/workflows/*` or CI build failures. Only scaffold when §0 CI=`yes`.

Delete `.github/workflows/build-pdf.yml` entirely when §0 CI=`no`.

### TeX Live + latexmk (default)

`.github/workflows/build-pdf.yml`:

```yaml
name: build-pdf
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint-and-build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2

      - name: Run ChkTeX linter
        run: |
          sudo apt-get update && sudo apt-get install -y chktex
          chktex -q -v0 -n24 -n13 main.tex

      - name: Build LaTeX PDF
        uses: xu-cheng/latex-action@1717208d197607736b44747db551528646f90382 # v3.2.0
        with:
          root_file: main.tex
          compiler: latexmk
          args: -pdf -interaction=nonstopmode -halt-on-error -outdir=build

      - name: Upload PDF Artifact
        uses: actions/upload-artifact@4cec3d8aa04e39d1a68397de0c4cd6fb99736d90 # v4.6.1
        with:
          name: main-pdf
          path: build/main.pdf
          retention-days: 7
```

### Tectonic variant (when §0 build=`tectonic`)

```yaml
      - name: Build PDF via Tectonic
        uses: wtfjoke/tectonic-docker-action@2732ce6bf5c2826ab562c1451f211516e8b2b93e # v3.1.0
        with:
          root_file: main.tex
          args: "-o build"
```

### Rules

- **Pin actions by SHA:** Never float major tags (`@v3`, `@v4`) without SHA pinning to protect from compromised upstream actions.
- **Security:** Never add `-shell-escape` to CI args. Actions must run sandboxed without arbitrary command execution.
- **Lint as gate:** ChkTeX must pass before compilation starts; fix syntax/formatting errors in PR before merging.
- **Clean build semantics:** CI runs from a fresh container (implicit `latexmk -C`). If it fails on CI but works locally, local build is hiding stale aux artifacts.
- **Engine must match §0:** `pdflatex` default image works; `xelatex`/`lualatex` need `texlive-xetex`/`texlive-luatex` packages or the Tectonic action.
- **Artifact:** Upload only `build/main.pdf` with `retention-days: 7` — never upload auxiliary files or build caches.
