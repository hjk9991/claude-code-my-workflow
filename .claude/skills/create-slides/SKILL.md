---
name: create-slides
description: Generate academic presentation slides in LaTeX Beamer from a paper draft or outline. Enforces assertion titles, one-idea-per-slide, equation intuition, and auto-runs /slide-excellence before delivery.
argument-hint: "[path to paper draft, outline, or section description]"
allowed-tools: ["Read", "Write", "Edit", "Bash", "Glob", "Grep", "Agent"]
---

# Create Academic Presentation Slides

Generate a complete LaTeX Beamer slide deck from a paper draft or outline, following strict content and formatting standards. Auto-compiles and runs `/slide-excellence` before delivery.

**Input:** `$ARGUMENTS` — path to a paper draft (`.tex`, `.pdf`), an outline file, or a plain-text description of what the talk should cover.

---

## Slide Format (Non-Negotiable)

Every deck produced by this skill uses the following preamble. Do not deviate.

```latex
\documentclass[aspectratio=169]{beamer}

% Theme: plain minimalist
\usetheme{default}
\usecolortheme{default}
\usefonttheme{professionalfonts}

% No header, no footer decorations — page number only, bottom right
\setbeamertemplate{headline}{}
\setbeamertemplate{navigation symbols}{}
\setbeamertemplate{footline}{%
  \hfill\insertframenumber\hspace{1em}\vspace{0.5em}%
}

% No total page count in footline (just current frame number)
\setbeamertemplate{framenumber}{\insertframenumber}

% Clean background
\setbeamercolor{background canvas}{bg=white}
\setbeamercolor{normal text}{fg=black}
\setbeamercolor{frametitle}{fg=black}

% Tight frametitle
\setbeamertemplate{frametitle}{%
  \vspace{0.4em}\insertframetitle\par%
}
```

---

## Content Standards

### 1. One key idea per slide — strictly enforced
- If a slide has two ideas, split it into two slides.
- The test: can you state the slide's point in one sentence? If not, split.

### 2. Titles must be assertions, not labels
**Wrong:** "Results", "Model", "Data", "Identification"
**Right:** "China shock reduced Korean manufacturing employment by 12%"
**Right:** "Outward MP attenuates import-competition displacement"
**Right:** "Proposition 1 decomposes welfare into three exact channels"

The title should be the conclusion a reader takes away from the slide.

### 3. No bullet points unless absolutely necessary
Prefer:
- A single figure with a clear caption
- A clean table
- Two or three sentences of prose
- A displayed equation with surrounding context

Use bullet points only when listing genuinely parallel items (e.g., data sources, robustness checks). Never use bullets as a substitute for prose argument.

### 4. Motivation before formalism
- Always set up the intuition *before* showing an equation or diagram.
- One or two sentences of economic context must precede any formal result.
- Never open a slide with an equation as the first element.

### 5. Every equation has a preceding intuition sentence
Before any `\begin{equation}` or displayed math, write one sentence in plain English explaining what the equation says and why it matters. This sentence goes on the same slide, above the equation.

### 6. Anticipate pedagogical questions
For every major result, ask: *What would a sharp audience member ask here?* Address the most important objection or confusion preemptively — either on the slide itself or on the immediately following slide.

---

## Workflow

### Step 1: Read the source material

Read the full source document passed in `$ARGUMENTS`. For a paper draft:
- Extract the main argument (one sentence)
- Identify the 3–5 key results worth presenting
- Note any figures or tables that should appear as slides
- Note the paper's Proposition/Theorem statements

For an outline or description: proceed directly to Step 2.

### Step 2: Design the slide structure

Map the talk to a skeleton before writing any LaTeX. Standard academic talk structure:

| Section | Slides | Purpose |
|---------|--------|---------|
| Title | 1 | Paper title, authors, affiliation, date |
| Motivation | 2–3 | Why does this question matter? What is surprising? |
| Preview of findings | 1 | "In this paper, we find..." — one slide with the 3 key takeaways |
| Data / Setting | 1–2 | What variation do we use? Who are the agents? |
| Empirical strategy | 2–3 | Identification: what is the instrument/design? Why valid? |
| Main results | 2–4 | One result per slide; assertion title; figure or table |
| Model | 2–4 | Setup, key equation, Proposition (with intuition) |
| Quantification | 2–3 | Counterfactual results; welfare decomposition |
| Robustness | 1–2 | Key robustness checks; briefly |
| Conclusion | 1 | Three bullets: what we did, what we found, why it matters |

Adjust slide counts based on talk length. For a 60-minute seminar: 35–45 slides. For a 20-minute conference presentation: 18–25 slides.

Write the skeleton as a commented block at the top of the `.tex` file:
```latex
% SLIDE SKELETON
% 1. Title
% 2. Motivation: [assertion]
% 3. ...
```

### Step 3: Draft all slides in LaTeX Beamer

Write every `\begin{frame}...\end{frame}` block. For each slide:
1. Write the assertion title as `\frametitle{...}`
2. Check: does it pass the one-idea test?
3. Check: if there's an equation, is there an intuition sentence above it?
4. Check: are bullet points avoided unless truly necessary?

Use these environments as appropriate:
- `\includegraphics` for figures (width ≤ \textwidth)
- `tabular` or `booktabs` tables
- `\begin{block}{...}...\end{block}` for key definitions or propositions
- Displayed math for equations (not inline)

### Step 4: Compile to PDF

Identify or create `compile.sh` in the slide directory. If it does not exist, create it:

```bash
#!/usr/bin/env bash
set -euo pipefail
DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$DIR"
pdflatex -interaction=nonstopmode slides.tex
pdflatex -interaction=nonstopmode slides.tex
echo "SUCCESS: $DIR/slides.pdf"
```

Two pdflatex passes suffice for Beamer (no bibtex needed unless bibliography slide is included; add bibtex pass if `\bibliography{}` is present).

Run:
```bash
bash compile.sh 2>&1
grep -c "^!" compile.log 2>/dev/null   # should be 0
```

### Step 5: Run /slide-excellence

After a clean compile, invoke the `slide-excellence` skill on the compiled `.tex` file. This runs three review agents in parallel: visual audit, pedagogical review, and proofreading.

Parse the synthesized report for:
- **Blocking issues** (must fix before delivery)
- **Major issues** (should fix)
- **Minor issues** (nice to fix)
- **Quality score** (must be ≥ 80 before delivery)

### Step 6: Fix blocking and major issues

For each Blocking and Major issue:
1. Read the flagged slide
2. Apply the minimal fix (split slide, rewrite title, add intuition sentence, reduce text)
3. Recompile after each batch of fixes
4. Re-run the quality check if score was below 80

Do not fix minor issues unless they take less than 2 minutes each — report them instead.

### Step 7: Final compile and report

Run a final clean compile. Then report:

```
Slide Deck: [filename].pdf
Slides: [N] total
Quality score: [N]/100 ([slide-excellence result])

Structure:
  Title + motivation:   [N] slides
  Empirical:            [N] slides
  Model:                [N] slides
  Quantification:       [N] slides
  Robustness + concl.:  [N] slides

Quality gates:
  [x] Zero LaTeX errors
  [x] No text overflow
  [x] All equations have intuition sentence
  [x] All titles are assertions
  [x] No slide with > 1 key idea
  [x] slide-excellence score ≥ 80

Remaining issues (minor):
  - [list any minor issues not fixed]
```

---

## Quality Gates (Hard Stops)

Do not deliver the slide deck until all of these pass:

| Gate | Check |
|------|-------|
| Compile clean | `grep -c "^!" compile.log` = 0 |
| No overflow | No `Overfull \hbox` wider than 20pt |
| Equation intuition | Every `\begin{equation}` or `\[` preceded by a sentence on the same slide |
| Assertion titles | No frame title is a single noun or label word |
| One idea per slide | Visual scan: each slide makes exactly one point |
| slide-excellence ≥ 80 | Synthesized score from Step 5 |

---

## Beamer Quick Reference

```latex
% Standard slide
\begin{frame}
\frametitle{Assertion title here}
Introductory sentence or figure.
\end{frame}

% Slide with equation (intuition required)
\begin{frame}
\frametitle{Trade costs enter welfare through the domestic expenditure share}
When bilateral trade costs fall, firms can source more cheaply abroad, raising real income.
Formally, the welfare gain for country $i$ in sector $j$ is:
\[
  \hat{W}_i^j = \left(\hat{\lambda}_{ii}^j\right)^{-1/\theta^j}
\]
where $\hat{\lambda}_{ii}^j$ is the change in the domestic expenditure share.
\end{frame}

% Proposition block
\begin{frame}
\frametitle{Proposition 1 decomposes welfare into three exact multiplicative channels}
Each worker's welfare gain separates cleanly into what trade, MP, and migration each contribute.
\begin{block}{Proposition 1 (Welfare Decomposition)}
$\hat{W}_{g} = \prod_{k} \left(\hat{\lambda}_{ii}^{k}\right)^{-\tilde{a}_{i}^{jk}/\theta^k}
               \cdot \left(\hat{\lambda}_{iii}^{k}\right)^{(1-\rho)\tilde{a}_{i}^{jk}/\theta^k}
               \cdot \hat{\mu}_{igg}^{k}$
\end{block}
Trade (first term), MP (second), migration (third). Proposition is exact — not an approximation.
\end{frame}

% Figure slide
\begin{frame}
\frametitle{Export-intensive regions gain most from China's opening}
\begin{center}
  \includegraphics[width=0.85\textwidth]{figures/figure_welfare_map.pdf}
\end{center}
\end{frame}

% Two-column layout
\begin{frame}
\frametitle{MP exposure attenuates the import-competition coefficient}
\begin{columns}[T]
  \begin{column}{0.48\textwidth}
    Without MP control: coefficient on imports is near zero (attenuation bias).
  \end{column}
  \begin{column}{0.48\textwidth}
    Controlling for MP: import coefficient becomes negative and significant.
  \end{column}
\end{columns}
\end{frame}
```

---

## Notes

- Beamer compiles with `pdflatex` (not XeLaTeX) — keep fonts within standard LaTeX font selection
- For talks citing the paper's own tables, copy the relevant rows into a `tabular`; do not `\input{}` from the paper's `tables/` directory
- If source figures are PDF vector graphics, use `\includegraphics` directly — they scale cleanly in Beamer
- The `/slide-excellence` skill targets lecture slides (Quarto/Beamer); it will still run on Beamer `.tex` files
- For a 20-minute talk, aim for 1 slide per 50–60 seconds of speaking time
