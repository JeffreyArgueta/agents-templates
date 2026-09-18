## Document Types — Class, Tree, Skeleton

> Read this when the task touches `\documentclass`, a new chapter/section/slide, or any `article`/`report`/`book`/`thesis`/`beamer`/`letter`/`cv`/`poster` structure.

Rule: content files hold body only. Never put `\documentclass` or `\usepackage` in `chapters/*` / `sections/*`.

### article (single-file paper, `sections/` + `\input`)

```tex
% main.tex
\documentclass[11pt,a4paper]{article}
\input{preamble/packages}
\input{preamble/macros}
\title{<title>} \author{<author>} \date{\today}
\begin{document}
\maketitle
\begin{abstract} <150 words> \end{abstract}
\input{sections/01-introduction}
\input{sections/02-method}
\printbibliography % only if biblatex
\end{document}
```

```
main.tex  preamble/  sections/01-introduction.tex  sections/02-method.tex  figures/  bib/
```

### report (multi-chapter, `chapters/` + `\include`)

```tex
\documentclass[11pt,a4paper]{report}
% ... same preamble pattern
\begin{document}
\maketitle \tableofcontents
\include{chapters/01-introduction}
\include{chapters/02-background}
\printbibliography
\end{document}
```

### book (`\frontmatter` / `\mainmatter`)

```tex
\documentclass[11pt,a4paper]{book}
\begin{document}
\frontmatter \maketitle \tableofcontents
\mainmatter
\include{chapters/01-beginnings}
\appendix \include{chapters/a-appendix}
\backmatter \printbibliography
\end{document}
```
Tree: `frontmatter/` (optional preface) + `chapters/` + `figures/` + `bib/`.

### thesis (front matter, chapters, appendices, bibliography)

Use `report` (short) or `book` (long). Required skeleton:

```tex
\begin{document}
\include{frontmatter/titlepage}      % \title \author degree/university
\include{frontmatter/abstract}
\include{frontmatter/declaration}    % originality statement
\tableofcontents \listoffigures \listoftables
\include{chapters/01-introduction}   % problem, objectives, outline
\include{chapters/02-related-work}
\include{chapters/03-method}
\include{chapters/04-results}
\include{chapters/05-conclusion}
\appendix \include{appendices/a-data}
\printbibliography
\end{document}
```
Tree: `frontmatter/` + `chapters/NN-*.tex` + `appendices/` + `figures/` + `bib/`.

### beamer (single-file, frames)

```tex
\documentclass{beamer}
\usetheme{Madrid}
\input{preamble/packages} % no biblatex unless citations needed
\title{<title>} \author{<author>} \date{\today}
\begin{document}
\begin{frame}\titlepage\end{frame}
\begin{frame}{Introduction}\begin{itemize}\item Point \end{itemize}\end{frame}
\end{document}
```
No `chapters/`/`sections/` dirs. One `frame` per idea; figures under `figures/`.

### letter

```tex
\documentclass{letter}
\usepackage[english]{babel} % via preamble
\address{<sender address>} \signature{<name>}
\begin{document}
\begin{letter}{<recipient address>}
\opening{Dear Sir or Madam,}
Body paragraph.
\closing{Yours faithfully,}
\end{letter}
\end{document}
```
Single `main.tex`, no preamble split needed beyond `babel`.

### cv / résumé

Default: `article` class, no external CV class (keeps Tectonic/Overleaf-safe):

```tex
\documentclass[11pt,a4paper]{article}
% preamble: geometry margins 2cm, enumitem, hyperref
\begin{document}
\begin{center}{\LARGE <Name>} \\ <email> — <phone> — <site>\end{center}
\section*{Experience} \section*{Education} \section*{Skills}
\end{document}
```

### poster (`tikzposter` default)

```tex
\documentclass{tikzposter}
\title{<title>} \author{<author>} \institute{<lab>}
\begin{document}\maketitle
\block{Problem}{Text + \includegraphics{figures/result}}
\block{Results}{...}
\end{document}
```
Alt: `beamerposter` inside `beamer` if the venue forces it. Single `main.tex` + `figures/`.

### Adding a chapter/section correctly

1. Create `chapters/03-method.tex` (thesis/report/book) or `sections/03-x.tex` (article) in kebab-case, `NN-` prefixed.
2. Start with `\chapter{...}` / `\section{...}` + first `\label{sec:...}` — no preamble lines.
3. Register exactly one line in `main.tex`: `\include{chapters/03-method}` or `\input{sections/03-x}` in numeric order.
4. Rebuild: `latexmk -pdf main.tex`.

### Draft builds with `\includeonly`

For `report`, `book`, and `thesis` documents, temporarily limit a draft build to selected chapters:

```tex
% main.tex, before \begin{document}
\includeonly{chapters/01-introduction,chapters/03-method}
```

Keep every `\include{...}` registration in `main.tex`; `\includeonly` filters the build without changing the document's table of contents or source order. Remove or comment out the line before the final build so all chapters are included. It does not apply to article sections, which use `\input`.
