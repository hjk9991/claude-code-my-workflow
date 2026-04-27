# CLAUDE.MD -- Trade, MP, and Regional Inequality

**Project:** Trade, Multinational Production, and Regional Inequality
**Institution:** Korea Development Institute
**Branch:** main

## Core Principles

- **Plan first** -- enter plan mode before non-trivial tasks; save plans to `quality_reports/plans/`
- **Verify after** -- confirm outputs compile and match expectations at the end of every task
- **Scripts are the source of truth** -- tables and figures are ephemeral; always re-generate from `.do` / `.m` scripts
- **Quality gates** -- 80 (commit) / 90 (PR) / 95 (excellence); full rubrics in `.claude/rules/quality-gates.md`
- **[LEARN] tags** -- when corrected, save `[LEARN:category] wrong → right` to MEMORY.md

## Workflow Model

**Contractor mode:** You direct, Claude orchestrates (PLAN → EXECUTE → REPORT).

**Claude asks when:** Design forks, code ambiguity, replication edge cases, scope questions.
**Claude executes when:** Obvious bug fixes, verification, documentation, plotting, deployment.

## Project Code (external)

All project code lives on OneDrive — not in this repo:

```
~/Library/CloudStorage/OneDrive-개인/Research/MP, Trade, and Migration/
├── LaTeX_Overleaf/          # Overleaf-synced (git); DO NOT restructure
│   ├── main.tex             # Master manuscript
│   ├── ref.bib              # Bibliography
│   ├── tables/              # Written by STATA scripts
│   └── figures/             # Written by STATA/MATLAB scripts
├── Shift share analysis/
│   ├── scripts/             # 00_master.do + 01–33 numbered scripts
│   ├── raw data/            # Source datasets
│   ├── processed data/      # Generated .dta files
│   └── output/              # estimates/, diagnostics/
├── code_Mira/               # Coauthor code — do not modify
└── code_v0625/              # Legacy code — do not modify
```

## Commands

```bash
# Compile manuscript (pdflatex + bibtex, 3 passes)
bash "~/Library/CloudStorage/OneDrive-개인/Research/MP, Trade, and Migration/LaTeX/compile.sh"

# Run STATA do-file (from Shift share analysis/ root)
stata -b do scripts/scriptname.do

# Run MATLAB script
matlab -nodisplay -nosplash -r "run('scripts/MATLAB/scriptname.m'); exit"
```

## Coding Standards

**STATA:** `set seed YYYYMMDD` · relative paths via `global root` · log files always · `esttab` for LaTeX tables
**MATLAB:** `rng(seed, 'twister')` · `fullfile()` for paths · convergence 1e-8 · save `.mat` at each phase
**Figures:** AEA style — PDF vector, no gridlines, no in-figure title, fonts ≥ 8pt, width ≤ 6.5in

## Editor / Local Tools

- **VS Code + LaTeX Workshop** is installed. Custom build recipe runs `bash compile.sh` (matches the OneDrive scripts above). PDF auto-refreshes on disk write — no manual reload after recompile from terminal or skill.
- SyncTeX bindings: `Cmd+Opt+J` (forward, `.tex` → PDF location), `Cmd+click` in PDF (inverse, → source).
- Korean OneDrive path (`개인`) occasionally trips LaTeX Workshop; check the "LaTeX Compiler" output panel if a build fails silently.

## Code ↔ Paper Sync (hard rule)

When a `.m`, `.do`, or `.R` file implements an algorithm that the manuscript describes verbatim (pseudocode block, equation, "we use Gauss-Seidel iteration"), update both **in the same session**. Drift between code and prose is a recurring critical-issue source (see Migration §3.7, Static §4 GS→Jacobi).

Practical check before commit: if the diff touches a numbered iteration loop, fixed-point operator, tolerance constant, or convergence criterion, grep the manuscript for the algorithm's name/tolerance and verify the description still matches.

---

## Research Workflow Patterns

The estimation cycle is NOT "run code → check output." It is:

1. **Estimate/solve** -- run the regression, solve the equilibrium, calibrate the model
2. **Diagnose** -- did it converge? Are coefficients economically plausible? Do signs match theory?
3. **Identify bottleneck** -- wrong objective? Algorithm oscillating? Bad initial guess? Implausible parameter regime?
4. **Surgical fix** -- change ONE thing blocking progress (damping, initialization, specification), not everything
5. **Verify with minimal rerun** -- warm-start from prior solution; rerun only what changed

When diagnostics reveal unexpected results:
- **First:** verify it's not a code bug (review, alternative spec, robustness)
- **Then:** reframe as feature or new hypothesis if it survives robustness
- **Decide:** main text, appendix, or separate paper? Prioritize by effort/insight ratio

When to write a new diagnostic script vs. modify the solver:
- **New script** if you need to *interpret* results in ways not anticipated upfront
- **Modify solver** only if the algorithm itself is wrong, not the post-processing
- Keep diagnostics decoupled: load saved `.mat`/`.dta`, compute decomposition, save separately

---

## Code → Table → Manuscript Pipeline

The pipeline is non-linear -- expect 2-5 backtracking cycles per revision:

```
Estimate → Export table → Compile → [Referee feedback / New insight]
    ↑                                          ↓
    └──── Modify specification ← Re-evaluate narrative
```

**Table fragment convention:**
- Fragments are HEADLESS -- no `\begin{table}`, `\toprule`, `\bottomrule`
- Framing (caption, notes, borders) goes in `main.tex` wrapper
- `esttab` math in footnotes breaks easily -- watch for stray `$` before punctuation, `\_` escapes

**Compilation error triage (fix in this order):**
1. Fatal TeX errors -- missing `}`, undefined control sequence
2. Undefined references (`??` in PDF) -- broken `\ref{}` or `\cite{}`; search source for the broken key
3. Overfull hbox -- cosmetic; address after content is stable

**Rule:** Always compile after STRUCTURE changes (section moves, table reorganization) BEFORE content edits. Structural errors cascade; content errors stay local.

---

## Algorithm Diagnosis Checklist

**Silent failures in numerical solvers:**
- `NaN > tol` evaluates to FALSE in MATLAB -- loops exit silently on divergence. Always guard: `if ~isfinite(diff), warning(...); break; end`
- Zero-valued denominators (zero labor demand, zero trade flow) → NaN propagation. Pre-fill missing observations with explicit defaults (e.g., tariff = 1, not 0/0)
- Column vs. row vector mismatch → silent wrong results in matrix products. Document all dimensions in comments: `% T: (N x N x J)`

**When convergence is slow (100+ iterations):**
- Diagnose BEFORE increasing `max_iter` -- is spectral radius > 1? Is damping too aggressive/too weak?
- Try warm-starting from a nearby solution (safe if within ~10% of expected answer)
- Compare Gauss-Seidel (sequential) vs. Jacobi (simultaneous) -- they should agree, but damping behavior differs

**Multiple equilibria:**
- Try 3+ starting points (supremum, infimum, midpoint) before declaring uniqueness
- If solutions differ: check iteration method and damping first (artifact?), then parameters (regime?)
- If multiplicity is real: document it, test if welfare conclusions are robust to equilibrium choice

**Tolerance-speed trade-offs:**
- Any relaxation from default (1e-8) must be justified in comments
- After any speedup optimization, compare output to pre-optimized baseline

---

## Economic Plausibility Checks

Apply these sanity checks after every solve/estimation -- catch bugs before they propagate:

- **Welfare decomposition** must sum to total; residual > 1e-6 → something is wrong in the accounting
- **Responses should be monotonic** in policy intensity -- if a larger shock produces a smaller effect, investigate (boundary case? discrete jump? bug?)
- **GE amplification** vs. PE: ratios of 2-5x are typical; >10x is likely a bug or extreme parameterization
- **Spillovers** to untreated units should decrease with economic distance (trade intensity, geography)
- **Zero observations** in trade/production data: assign explicit defaults before any weighted-average computation; never let 0/0 → NaN propagate silently
- **Coefficient stability**: if adding a control variable flips signs, the original specification likely has omitted variable bias -- don't just report the "better" result

---

## Collaboration Skills

| Skill | When | What it does |
|-------|------|-------------|
| `/collab-setup` | Once per coauthor | Create branches, set rules, update registry |
| `/collab-start` | Every session start | Pull, check coauthor activity, report conflicts |
| `/collab-end` | Every session end | Commit, push, write handoff note |
| `/collab-merge` | When ready to integrate | Merge dev branches into main, tag release |
| `/collab-status` | Anytime | Show branch state, conflicts, handoff notes |

**Project registry:** `.claude/projects/*.md` -- never push directly to `main`

**Carve-outs (direct push allowed):**
- **Overleaf-git remotes** (`git.overleaf.com/*`) — `master` is the only branch; no PR mechanism exists. Direct push to `master` is the standard sync workflow.
- **Solo personal repos** with no coauthors — the rule exists to protect shared review state; if there is no second pair of eyes, the gate is pure friction.

The rule still applies to: GitHub multi-author repos, shared research repos with active review, anything with CI gates that should run before merge.
