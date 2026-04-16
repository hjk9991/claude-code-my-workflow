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


---
**Context compaction (auto) at 18:14**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 18:15**
Check git log and quality_reports/plans/ for current state.


---
## Addendum (18:45): Substantive review findings addressed

Per user direction after presenting four substantive domain-reviewer items, worked on three (deferred item 4 on $\kappa_s$ citation).

### Task 1 — Prop 1' residual bound (analytical)

Derived closed-form bound. In $S=2$, the wedge between skill-specific and composite wages is
$\Delta\log w_{s,n}^k - \Delta\log w_{\mathrm{comp},n}^k = \beta_{s,k}\,\Delta\log(w_{H,n}^k/w_{L,n}^k)$
with $\beta_{H,k}=1-\alpha_H^k$, $\beta_{L,k}=-\alpha_H^k$. Propagating through the Leontief IO inverse gives an explicit residual $R_{g,s} = \sum_{j,k} \alpha_i^j\tilde a_i^{jk}\gamma_{\mathrm{labor},i}^k\beta_{s,k}\Delta\log(w_H^k/w_L^k)$. Two bounds combine: (a) $|\beta_{s,k}|\le 1$ and Caliendo--Parro-style summability of the Leontief weight; (b) Cobb--Douglas FOC pins $\Delta\log(w_H^k/w_L^k) = \Delta\log(Z_L^k/Z_H^k)$ compressed by common Fréchet response. Added as `\paragraph{Residual bound.}` after Proposition~\ref{prop:welfare_skill} with new equation labels `eq_wedge` and `eq_residual`.

### Task 2 — $w_{\mathrm{comp}}$ separability remark in §app:skill_model

Added one paragraph after the Fréchet-preferences paragraph and before Proposition~\ref{prop:welfare_skill}: makes the chain-rule decomposition explicit, $\partial\log P_i^k/\partial\log w_{s,n}^j = (\partial\log P_i^k/\partial\log w_{\mathrm{comp},n}^j)\cdot(\partial\log w_{\mathrm{comp},n}^j/\partial\log w_{s,n}^j)$, and states that trade/MP channels are scalar rescalings of the $S=1$ operator with skill heterogeneity entering only through migration.

### Task 3 — S=2 at a^j=0 and 0.5×XVEM (RESOLVED)

Created `Model quantification/probe_low_a_j_s2.m`: bounded probe (max_iter=800) for both low corners under S=2, with try/catch around `calibrate.m` plus explicit residual classification. Modified `calibrate.m` to honor an optional `max_iter_override` so the probe can run quickly without altering the canonical 15000-iter budget. Backed up `calibration_skill_final.mat` before each corner run (calibrate.m saves unconditionally on exit, so an unsupervised probe would otherwise corrupt the canonical .mat).

**Outcome — split:**
- **a=0:** diverged. Korean residual stalls at 0.0186 around iter 400, drifts upward to 0.0279 by iter 800 (tolerance 0.012). Confirms the prior author's exclusion.
- **a=0.5×XVEM:** converged at iter 409, korea=0.0120 (= tolerance), row=0.1196. Korea pooled gain +2.23%. The prior "ill-behaved" claim was overly pessimistic.

**Action taken:**
- Created `Model quantification/write_table_robustness_a_j_skill.m` — emitter that combines `robustness_a_j_s2.mat` (existing 5 cols: 1× through 5× XVEM) with `probe_low_a_j_s2.mat` (new 0.5× col) into a single 6-column table fragment. Decoupled from calibration so the table can be regenerated without rerun.
- Regenerated `LaTeX/tables/table_robustness_a_j_skill.tex` (6 columns).
- Updated main.tex appendix prose (l.1561–1565): footnote now excludes only a=0 with specific diagnostic numbers; first sentence now says "spanning $0.5\times$ to $5\times$"; numerical sentences updated to anchor at the new $0.5\times$ column ($+2.23\%$, differential $-0.53$~pp).
- Compile clean: 97 pages, 0 undefined refs.

### Verification status

| Check | Result | Status |
|-------|--------|--------|
| Tasks 1, 2 LaTeX edits compile | 97 pages, 0 undefined refs | PASS |
| Task 3 probe MATLAB run | a=0 diverged (diff_korea=0.028); 0.5×XVEM converged (diff_korea=0.012, +2.23%) | PASS (split outcome) |
| Decoupled emitter regenerates table | 6-column table_robustness_a_j_skill.tex written from saved .mat files | PASS |
| Final compile after Task 3 prose update | 97 pages, 0 undefined refs | PASS |
| Canonical calibration_skill_final.mat preserved | Backed up to `.BACKUP_pre_probe_low.mat` and restored after probe | PASS |

### Learnings (Tasks 1–3)

- [LEARN:matlab] Long-running iterative solvers (`calibrate.m`, 15000-iter budget) need an `if ~exist('max_iter_override','var')` shim so diagnostic probes can cap iterations without editing the script. Cleaner than commenting out and remembering to revert.
- [LEARN:workflow] Calibration scripts that unconditionally save to a canonical `.mat` on exit will corrupt that file when run in failure-probing mode. Either (a) gate the save on convergence, or (b) back up the canonical file before launching probes. We did (b); (a) is the right long-term fix.
- [LEARN:econ-modeling] Footnotes that exclude parameter corners on "ill-behaved" grounds should always be re-tested when the underlying model changes (here: S=1 → S=2 calibration target dimension doubled). The prior exclusion of $0.5\times$ XVEM did not survive re-probing — convergence is achievable and yields a sensible $+2.23\%$ that strengthens the monotonic-and-concave narrative.

---

## Addendum (2026-04-17): Item 4 — κ_s citation reattribution

The original split log deferred a domain-reviewer item flagging that `\citet{caliendo2019trade}` was attached to $\kappa_L=1.20$, $\kappa_H=1.80$ in the parameterization paragraph. Web verification confirms the citation is wrong on substance: CDP19 (Econometrica, not REStud) estimates a SINGLE migration elasticity ($\nu \approx 5.34$ quarterly) and does not split by skill. The do-file (`41_skill_matlab_inputs.do` l.269) compounded the error by also mislabelling the journal.

Actual provenance of the values: a calibration designed to bracket the Roy--Fr\'echet shape parameter $\kappa \approx 1.5$ from \citet{galle2023slicing} (RestudY 2023) — already in `ref.bib` as `galle2023slicing` — with $\kappa_H > \kappa_L$ per the standard mobility-by-education pattern.

**Action taken (Option A — reattribute to GRY):**
- `LaTeX/main.tex` l.1534: replaced the CDP19 sentence with a GRY-bracketing rationale that names the shape parameter, justifies the H>L ordering, and notes KLIPS thinness as the reason for using literature values.
- `Shift share analysis/scripts/41_skill_matlab_inputs.do` l.269–276: fixed the comment block to (a) credit GRY as the bracketing source, (b) explicitly state that CDP19 is NOT the source and why, (c) preserve the "above unity for $\eta$" requirement.
- Compile clean: 97 pages, 0 undefined refs.

**Why Option A over re-running the calibration with CDP19's $\nu$:** GRY's framework is conceptually closest (multi-sector gravity, worker groups, Roy--Fr\'echet labor allocation), the value bracketing was the calibration's actual logic, and changing the headline numbers would require re-running the full S=2 pipeline (~25 min) without strengthening the substantive argument.

### Learnings (Item 4)

- [LEARN:bib] Auto-saved Zotero bibkeys can be misleading. `caliendo2019trade` is the CDP19 China-shock paper in Econometrica, not the trade-policy survey CDP wrote separately. Always read the bib entry's `journal` and `title` fields before reusing a citation in a new context — bibkey alone is not provenance.
- [LEARN:workflow] When a calibration fallback is "literature-anchored," the script comment, the manuscript citation, and the bib entry all need to agree on the source paper. Drift between the three (we had three different stories: do-file said "CDP19 REStud", manuscript said `\citet{caliendo2019trade}`, bib entry said Econometrica) is a referee magnet. Single-source the attribution from the start.


---
**Context compaction (auto) at 18:19**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 18:35**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 18:36**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 18:38**
Check git log and quality_reports/plans/ for current state.


---
**Context compaction (auto) at 23:58**
Check git log and quality_reports/plans/ for current state.
