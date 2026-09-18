## Figures & Tables — Formats, Floats, TikZ

> Read this when the task touches `figures/*`, `\includegraphics`, tables, `tikz`/`pgfplots`, or float placement.

### Formats (prefer vector)

- Vector first: `figures/architecture.pdf` for diagrams/plots. Raster only for photos/screenshots: `figures/result.png` (lossless) — avoid `.jpg` unless photo-size forces it.
- Never commit SVGs as include targets; export to `.pdf` first. Keep the `.svg` source out of the repo or beside it only if §0 says so.
- Kebab-case, no spaces: `figures/nn-accuracy-curve.pdf`. Reference: `\includegraphics[width=\linewidth]{figures/nn-accuracy-curve}` (no extension — engine picks `.pdf`/`.png`).

### Figure pattern

```tex
\begin{figure}[htbp]
  \centering
  \includegraphics[width=0.8\linewidth]{figures/architecture}
  \caption{System overview.} \label{fig:arch}
\end{figure}
```
`\label` always after `\caption`. Reference as `Figure~\ref{fig:arch}`.

### Table pattern

```tex
\begin{table}[htbp]
  \centering
  \caption{Results.} \label{tab:results}
  \begin{tabular}{lcc} \hline Method & Acc & F1 \\ \hline Ours & 91.2 & 0.90 \\ \hline \end{tabular}
\end{table}
```
Use `booktabs` (`\toprule`/`\midrule`/`\bottomrule`) when available. `\label` after `\caption`.

### TikZ / pgfplots

```tex
\usepackage{tikz, pgfplots} \pgfplotsset{compat=1.18}
```
Inline small diagrams; externalize large ones only if build time forces it (needs `-shell-escape` — call it out, never silent). Poster (`tikzposter`) blocks take `\includegraphics` directly.

### Placement

Default `[htbp]`; never `[H]` without `float` package + justification. One float per idea; `width=\linewidth` max; thesis/report lists come free from `\listoffigures`/`\listoftables`.
