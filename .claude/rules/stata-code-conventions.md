---
paths:
  - "**/*.do"
  - "scripts/Stata/**"
---

# STATA Code Standards

**Standard:** Reproducible, publication-ready STATA code for empirical economics research.

---

## 1. Reproducibility

- `set seed YYYYMMDD` called ONCE at the top of each do-file
- All paths relative — use a `global root` macro pointing to the project root:
  ```stata
  global root "path/to/MP, Trade, and Migration"
  global data  "$root/data"
  global out   "$root/output"
  global log   "$root/logs"
  ```
- Log files opened at the start of every do-file:
  ```stata
  log using "$log/scriptname.log", replace
  ```
- Close log at the end: `log close`

---

## 2. Data Integrity

- Use `assert` statements after any merge or reshape to verify expectations:
  ```stata
  merge 1:1 region year using "$data/employment.dta"
  assert _merge == 3  // all observations must match
  drop _merge
  ```
- Document sample drops with comments explaining the rationale
- `preserve` / `restore` for temporary transformations; never overwrite the master dataset in-place

---

## 3. Estimation & Output

- Prefer `reghdfe` for fixed-effects regressions; `ivreghdfe` for IV with FE
- Cluster standard errors at the appropriate level (industry, region, or both):
  ```stata
  reghdfe lemployment china_shock controls, absorb(region year) cluster(industry region)
  ```
- Export tables to LaTeX using `esttab`:
  ```stata
  esttab m1 m2 m3 using "$out/tables/table2.tex", replace ///
    booktabs label se star(* 0.10 ** 0.05 *** 0.01) ///
    title("Main Results") nomtitle
  ```
- Every `esttab` table must be verified against the manuscript table (coefficients, SEs, N, significance stars)

---

## 4. Shift-Share Analysis

- Follow Goldsmith-Pinkham, Sorkin, Swift (2020) exactly:
  - **Exposure:** `Z_i = Σ_j β_ij * g_j` where `β_ij` are base-period industry shares and `g_j` are national growth rates
  - **Base period:** clearly defined and documented (not the same as the outcome period)
  - **Exogeneity:** pre-period industry shares must be argued as valid instruments
- Compute Rotemberg weights for sensitivity analysis
- Test for pre-trends using placebo regressions with pre-period outcomes

---

## 5. File Naming & Saving

- Do-files: `XX_descriptive_name.do` (two-digit prefix for ordering)
- Intermediate datasets: `descriptive_name_vYYYYMMDD.dta` (versioned)
- Final datasets: `final_descriptive_name.dta`
- Tables: `tableN_short_description.tex`
- Figures: `figureN_short_description.pdf`

---

## 6. Common Pitfalls

| Pitfall | Impact | Prevention |
|---------|--------|------------|
| `cluster()` df adjustment varies by command | SE differences across estimators | Check `reghdfe` vs. `ivreg2` df adjustment explicitly |
| `quietly` suppresses warnings | Silent errors in merges | Use `quietly` only for output suppression, not data operations |
| `collapse` overwrites current dataset | Data loss | Always `preserve` before `collapse` or work on a temp copy |
| `merge` without checking `_merge` | Silently wrong sample | Always `assert _merge == 3` (or document partial merges) |
| Missing `, replace` in `log using` | Fails on rerun | Always include `, replace` |

---

## 7. Code Quality Checklist

```
[ ] set seed at top
[ ] global root macro defined
[ ] log file opened and closed
[ ] All paths use $root/$data/$out macros
[ ] assert after every merge
[ ] Sample drops commented with rationale
[ ] esttab output verified against manuscript
[ ] File saved with descriptive versioned name
[ ] Comments explain WHY not WHAT
```
