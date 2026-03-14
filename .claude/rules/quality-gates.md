---
paths:
  - "Slides/**/*.tex"
  - "Quarto/**/*.qmd"
  - "scripts/**/*.R"
---

# Quality Gates & Scoring Rubrics

## Thresholds

- **80/100 = Commit** -- good enough to save
- **90/100 = PR** -- ready for deployment
- **95/100 = Excellence** -- aspirational

## Quarto Slides (.qmd)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Compilation failure | -100 |
| Critical | Equation overflow | -20 |
| Critical | Broken citation | -15 |
| Critical | Typo in equation | -10 |
| Major | Text overflow | -5 |
| Major | TikZ label overlap | -5 |
| Major | Notation inconsistency | -3 |
| Minor | Font size reduction | -1 per slide |
| Minor | Long lines (>100 chars) | -1 (EXCEPT documented math formulas) |

## R Scripts (.R)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Syntax errors | -100 |
| Critical | Domain-specific bugs | -30 |
| Critical | Hardcoded absolute paths | -20 |
| Major | Missing set.seed() | -10 |
| Major | Missing figure generation | -5 |

## Beamer Slides (.tex)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | XeLaTeX compilation failure | -100 |
| Critical | Undefined citation | -15 |
| Critical | Overfull hbox > 10pt | -10 |

## Enforcement

- **Score < 80:** Block commit. List blocking issues.
- **Score < 90:** Allow commit, warn. List recommendations.
- User can override with justification.

## Quality Reports

Generated **only at merge time**. Use `templates/quality-report.md` for format.
Save to `quality_reports/merges/YYYY-MM-DD_[branch-name].md`.

## STATA Scripts (.do)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Syntax error / abnormal exit | -100 |
| Critical | Hardcoded absolute paths | -20 |
| Critical | Missing `set seed` | -15 |
| Major | Missing log file | -10 |
| Major | Non-reproducible sample selection (undocumented drops) | -10 |
| Major | `esttab` output doesn't match manuscript table | -10 |
| Minor | No `assert` for data integrity checks | -3 |

## MATLAB Scripts (.m)

| Severity | Issue | Deduction |
|----------|-------|-----------|
| Critical | Runtime error | -100 |
| Critical | Non-convergence without diagnostic flag | -20 |
| Critical | Hardcoded absolute paths | -20 |
| Major | Missing `rng()` seed | -15 |
| Major | Missing convergence check or max-iteration guard | -10 |
| Major | No `.mat` save between computation phases | -10 |
| Minor | Column/row vector inconsistency (not caught by runtime) | -5 |

## Tolerance Thresholds (Research)

| Quantity | Tolerance | Rationale |
|----------|-----------|-----------|
| GE convergence (MATLAB) | 1e-8 | Standard for fixed-point iteration |
| STATA point estimates | 1e-6 | Rounding in paper display |
| Standard errors | 1e-4 | Numerical precision |
| Welfare changes (%) | 1e-4 | Economic significance threshold |
