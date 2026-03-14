---
paths:
  - "**/*.m"
  - "scripts/MATLAB/**"
---

# MATLAB Code Standards

**Standard:** Reproducible, publication-ready MATLAB code for GE model quantification.

---

## 1. Reproducibility

- `rng(seed, 'twister')` called ONCE at the top of each script (seed = YYYYMMDD):
  ```matlab
  rng(20230101, 'twister');
  ```
- All paths use `fullfile()` — never hardcode separators or absolute paths:
  ```matlab
  root_dir = fileparts(mfilename('fullpath'));
  data_dir = fullfile(root_dir, '..', '..', 'data');
  out_dir  = fullfile(root_dir, '..', '..', 'output');
  ```
- Run scripts as functions or with a clear top-level entry point — no global state side effects

---

## 2. Fixed-Point Iteration

- Always include a convergence check with a tolerance and max-iteration guard:
  ```matlab
  tol     = 1e-8;
  max_iter = 10000;
  diff    = Inf;
  iter    = 0;

  while diff > tol && iter < max_iter
      % ... update step ...
      diff = max(abs(P_new - P_old));
      P_old = P_new;
      iter  = iter + 1;
  end

  if iter == max_iter
      warning('Fixed-point did not converge after %d iterations (diff = %.2e)', max_iter, diff);
  end
  ```
- Convergence criterion: 1e-8 (default for GE price and wage systems)
- Report final `diff` and `iter` count in output

---

## 3. Save Checkpoints

- Save `.mat` files at the end of each computation phase (calibration, counterfactual, welfare):
  ```matlab
  save(fullfile(out_dir, 'calibration_YYYYMMDD.mat'), 'params', 'shares', 'wages');
  ```
- Use descriptive variable-scoped saves — do not `save('all')` (creates reproducibility issues)
- Versioned filenames (`_vYYYYMMDD`) for interim results; `_final` only for publication

---

## 4. Matrix Dimensions

- Document array dimensions explicitly in comments:
  ```matlab
  % T: (N_countries x N_countries x J_industries) trade share matrix
  % W: (N_regions x 1) regional wage vector
  ```
- Use consistent dimension ordering throughout: (origin, destination, industry)
- Check dimensions before matrix operations; use `assert(size(X,1)==N, 'dimension mismatch: X')`

---

## 5. Counterfactual Analysis

- Separate baseline calibration from counterfactual simulation in distinct scripts/functions
- Counterfactual changes reported in "hat algebra" notation — comment which variables are changes vs. levels
- Welfare decomposition: trade gains, MP gains, and migration gains must sum to total; verify numerically
- Save both baseline and counterfactual equilibrium objects for comparison

---

## 6. File Naming & Saving

- Scripts: `XX_descriptive_name.m` (two-digit prefix for ordering)
- Calibration output: `calibration_vYYYYMMDD.mat`
- Counterfactual output: `counterfactual_[scenario]_vYYYYMMDD.mat`
- Final results tables: `tableN_short_description.tex`
- Figures: `figureN_short_description.pdf`

---

## 7. Figure Output

- AEA style — PDF vector output, no gridlines, no in-figure title
- Fonts ≥ 8pt; figure width ≤ 6.5in (full-page) or 3.5in (half-page)
  ```matlab
  set(gcf, 'Units', 'inches', 'Position', [0 0 6.5 4]);
  exportgraphics(gcf, fullfile(out_dir, 'figures', 'figure1_welfare.pdf'), 'ContentType', 'vector');
  ```
- Caption and title go in LaTeX, not in the figure

---

## 8. Common Pitfalls

| Pitfall | Impact | Prevention |
|---------|--------|------------|
| Column vs. row vector mismatch | Silent wrong results in matrix products | Explicitly transpose and document `(N x 1)` vs. `(1 x N)` |
| `save` without variable list | Saves workspace junk, reproducibility breaks | Always list variables: `save(file, 'var1', 'var2')` |
| Non-convergence not flagged | Silent wrong equilibrium | Always check `iter == max_iter` and `warning()` |
| Absolute paths with `cd` | Breaks on other machines | Use `fullfile()` and `mfilename('fullpath')` |
| Implicit broadcasting pre-R2016b | Wrong results silently | Use explicit `repmat()` for older compatibility, or check MATLAB version |

---

## 9. Code Quality Checklist

```
[ ] rng() seed at top (YYYYMMDD)
[ ] All paths via fullfile() — no hardcoded separators
[ ] Fixed-point: tolerance 1e-8, max_iter guard, convergence warning
[ ] .mat checkpoint saved at end of each computation phase
[ ] Array dimensions documented in comments
[ ] Counterfactual separated from calibration
[ ] Welfare components verified to sum correctly
[ ] Figures: PDF vector, AEA style, no in-figure title
[ ] Comments explain WHY not WHAT
```
