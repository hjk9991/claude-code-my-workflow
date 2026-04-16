# Session Log: 2026-04-16 -- S=1/S=2 Main-Appendix Split

**Status:** COMPLETED

## Objective

Reorganize the Trade/MP/Migration manuscript so that the main text uses the
homogeneous-labor (S=1) model as the baseline for all headline numbers and
comparisons (including the a^j sensitivity), and move the skill-stratified
(S=2) extension — Proposition 1', skill welfare table/figure, and S=2 a^j
sensitivity — into a dedicated appendix. Also regenerate the main-text
a^j table from saved MATLAB output rather than hand-restored values.

## Changes Made

| File | Change | Reason | Quality Score |
|------|--------|--------|---|
| `LaTeX/main.tex` (abstract) | Removed skill-stratified sentence | S=2 moved to appendix | 92/100 |
| `LaTeX/main.tex` (model, l.630-643) | Removed Prop 1' block + intro paragraph | Same | 92/100 |
| `LaTeX/main.tex` (a^j paragraph, l.1006-1016) | Reverted from 5-variant S=2 to 4-variant S=1 narrative | Main text = S=1 | 92/100 |
| `LaTeX/main.tex` (skill-welfare subsection) | Removed subsection + Table ~ref{tab:skill_welfare} + Figure | Relocated to appendix | 92/100 |
| `LaTeX/main.tex` (conclusion) | Removed within-region skill-inequality sentence | Same | 92/100 |
| `LaTeX/main.tex` (new appendix `app:skill_extension`) | Added full skill-stratified extension: model, Prop 1', skill welfare table/figure, S=2 a^j sensitivity | Consistency directive | 92/100 |
| `LaTeX/tables/table_robustness_a_j.tex` | Regenerated S=1 4-variant (from `robustness_a_j.mat`) | Script-of-truth provenance | 95/100 |
| `LaTeX/tables/table_robustness_a_j_skill.tex` | New file: S=2 5-variant a^j (copy of prior S=2 output) | Appendix uses it | 95/100 |
| `Model quantification/write_table_robustness_a_j_s1.m` | NEW: re-emits main-text a^j table from saved .mat | Replaces hand-written table | 95/100 |

## Design Decisions

| Decision | Alternatives Considered | Rationale |
|----------|------------------------|-----------|
| Main text = S=1; appendix = S=2 (full separation) | (A) Keep S=2 in main with footnote; (B) revert to S=1 everywhere | User-requested: consistency of headline numbers matters more than elegance of a single Prop statement |
| Regenerate S=1 table from saved `.mat` via helper script | Rerun full calibration cycle at S=1 (~20 min) | Saved output from earlier S=1 run already authoritative; recalibration adds no information |
| Keep S=2 5-variant a^j table in appendix (not re-cut to 4 variants) | Drop 3x/4x/5x to match main-text column count | Appendix benefits from showing the concave-saturation evidence at larger a^j; readers see the main-text numbers nested within the appendix range |
| Use `\citep{caliendo2019trade}` for kappa_s values rather than plain-text | Plain (Caliendo-Dvorkin-Parro 2019) citation | Entry already in ref.bib; consistent with rest of manuscript |

## Incremental Work Log

**01:00 local:** User directed: main text entirely S=1, appendix entirely S=2 (for headline-number consistency). Task #29 opened.

**01:05:** Overwrote `table_robustness_a_j.tex` with hand-restored S=1 values (+2.12%, +1.84%, +2.02%, +2.73%).

**01:10:** Removed abstract skill sentence; removed Prop 1' block (l.630-643); reverted a^j paragraph and table caption to S=1 narrative.

**01:18:** Removed skill-welfare subsection (paragraph + table + figure) from main Counterfactuals section; reverted conclusion skill sentence.

**01:22:** Added new `\section{Skill-Stratified Extension}\label{app:skill_extension}` before bibliography: model subsection, Prop 1', parameterization, skill-welfare table/figure subsection, S=2 a^j sensitivity subsection (using `table_robustness_a_j_skill.tex`).

**01:28:** Compile clean, 96 pages, 0 undefined refs. Task #29 completed.

**01:35:** User requested (1) regenerate table from script, (2) write session log.

**01:42:** Wrote `write_table_robustness_a_j_s1.m` to load `robustness_a_j.mat` (Apr 16 14:18, pre-S=2-work) and re-emit the table. Regenerated values match hand-restored within rounding (+2.11% vs +2.12%, +2.74% vs +2.73%); main-text paragraph updated to match.

**01:48:** Final compile clean, 96 pages.

## Learnings & Corrections

- [LEARN:workflow] When exploring a structural extension (here S=2), keep the S=1 calibration `.mat` file under a distinct name so it can be reloaded for main-text tables without a full rerun. In this project, `robustness_a_j.mat` (S=1) coexists with `robustness_a_j_s2.mat` (S=2); the helper script `write_table_robustness_a_j_s1.m` pulls from the former and emits the main-text fragment. Avoids 20-minute recalibration cycles when reorganizing the manuscript.

- [LEARN:econ-modeling] The Cobb-Douglas skill aggregator has a clean analytical property: prices/trade/MP blocks depend only on the composite wage, so Proposition 1' decomposes cleanly into a skill-common (trade+MP) piece and a skill-specific (migration) piece. The first-order residual in log-welfare from the Leontief coupling was <2.5e-3 in our calibration.

- [LEARN:latex] Tables using `\quad Seoul/Cap` instead of `Seoul/Capital` cause label inconsistency across tables in the same manuscript. When a MATLAB helper script emits region labels, always use the canonical spelling used elsewhere in the tables folder.

## Verification Results

| Check | Result | Status |
|-------|--------|--------|
| Compile produces PDF | 96 pages, 0 fatal errors | PASS |
| No undefined references | `grep undefined main.log` returns 0 | PASS |
| `app:skill_extension` label resolves | Appendix forward reference from main-text a^j paragraph compiles | PASS |
| `prop:welfare_skill`, `tab:skill_welfare`, `fig:welfare_by_skill` only referenced from within the appendix itself | grep confirms | PASS |
| Main-text a^j table (S=1) regenerated from saved `.mat` | matches hand-restored values within rounding | PASS |
| S=2 appendix a^j table unchanged | byte-for-byte identical to pre-split version | PASS |

## Open Questions / Blockers

- [ ] Pitt-talk slides (Phase 6F): if the conference deck is still live, it needs the same aggregate-first / skill-in-appendix reframing. User declined to take this on in this session.

## Next Steps

- [ ] Commit the changes (user decision).
- [ ] At next opportunity, consider whether the skill-stratified appendix needs its own standalone slide set for discussant Q&A.
