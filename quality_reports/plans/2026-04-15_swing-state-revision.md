# Plan: Swing State Strategy — Phase F Revision (Post-Review Fixes)

**Project folder:** `~/Library/CloudStorage/OneDrive-개인/Research/Swing State Strategy/`
**Scope:** Address critical/major findings from three-agent holistic review (domain, STATA, proofreader).
**Target:** Conference-ready manuscript + defensible empirical pipeline.

---

## Context

Phase F shipped §3.3 (Regression Evidence) and Appendix B (Methodology) plus a
`feenstra1994new` bib entry. Three independent reviewers surfaced convergent
problems: the Feenstra estimator is **mis-implemented** (median-demeaning
rather than reference-variety GMM), the paradox coefficient's positive sign
is a **collinearity artifact** in 03 (not a substantive failure), and §3.3
**overstates** the leakage triple-diff (β₁ = −0.482 is p = 0.118).

We fix the empirics first (so the manuscript numbers reflect a correct run),
then rewrite the prose, add the missing formal Proposition 5, and swap the
σ<1 citation block to the correct supporting papers.

---

## Phase 1 — STATA fixes (re-run all three specifications)

### 1A. `04_leakage.do` — event-study base year
- Omit `tau2017` from generation loop (line 74–77):
  ```
  forvalues y = 2012/2024 {
      if `y' != 2017 gen tau`y' = (year==`y') * china_crit
  }
  ```
- Expected effect: coefficients now interpretable as deviations from pre-treatment
  2017. Headline range for §3.3 will change.
- **Add pre-trend joint F-test** after `eststo t2`:
  ```
  test tau2012 tau2013 tau2014 tau2015 tau2016
  estadd scalar F_pretrend = r(F)
  estadd scalar p_pretrend = r(p)
  ```
- **Two-way cluster** for 54-cluster concern: switch `cluster(panel_id)` →
  `vce(cluster partner_id year)` for t1 headline; retain panel cluster for t2.

### 1B. `03_paradox_test.do` — resolve collinearity + add post-2018 split
- **Remove year FE** (line 70–72) to identify `lag_el` level effect; keep
  sector FE and cluster at panel. Current dual absorption made `lag_el` a
  pure time series, sweeping the aggregate.
- **Add post-2018 subsample regression** (Domain C2):
  ```
  preserve
  keep if partner_iso3 == "CHN" & reporter_iso3 == "USA" & year >= 2018
  eststo m3: reghdfe ln_m lag_el, absorb(sector_id) cluster(panel_id)
  eststo m4: reghdfe ln_m c.lag_el##i.sector_id, absorb(sector_id) cluster(panel_id)
  restore
  ```
  Update `esttab` to report m1–m4.
- **Zero-drop accounting:** add `count if value_usd<=0 | missing(value_usd)`
  with `di "Dropped: " r(N)` before the drop.

### 1C. `02_sigma_estimation.do` — downgrade framing (do not refit)
- Full Feenstra GMM reimplementation is out of scope for conference. Instead,
  **rename estimator** in comments and log as "robust reduced-form unit-value
  difference" (not "Feenstra-Broda-Weinstein"). Manuscript (Appendix B) will
  cite Atalay (2017) and Boehm et al. (2019) for the σ<1 claim and describe
  our estimate as directional-only.

### 1D. Re-run all three do-files, regenerate:
- `output/tables/leakage_triplediff.tex`
- `output/tables/paradox_main.tex` (will now have 4 columns)
- `output/figures/leakage_event_study.pdf`

---

## Phase 2 — Manuscript numerical/prose fixes

### 2A. §3.3 (lines ~154–183 of `main.tex`) — rewrite around new numbers
- Update the event-study coefficient range to **reflect the new (post-fix)
  output** from `04_leakage.do`; do NOT hardcode `−1.1 to −1.5`. After
  Phase 1A the ranges will shift because the base year changed.
- Rewrite the β₁ = −0.482 sentence honestly: "directionally consistent with
  Proposition 5 but statistically imprecise (p = 0.118); the event-study
  decomposition in Figure X shows the cumulative pattern more clearly."
- Rewrite the paradox-positive-slope paragraph using m3/m4 (post-2018
  subsample) as the **headline** result; demote pooled m1/m2 to "pooled
  benchmark" with explicit caveat.
- Fix L.\mathrm{EL} → `EL^{\text{CHN}}_{t-1}`.

### 2B. Appendix B — correct estimator description and citation
- Replace "Feenstra/Broda-Weinstein double-difference estimator" with
  "robust unit-value difference estimator in the spirit of Feenstra (1994),
  though without the full exporter-variety GMM identification."
- Fix "cross-sectional variation in the error covariance across HS codes"
  → "heteroscedasticity across exporter varieties within an HS code
  (which our median-demeaning specification does not exploit)."
- Tense fix: "We implemented" → "We estimate."
- State explicitly: the σ<1 claim used in the model is **sourced from
  Atalay (2017) and Boehm et al. (2019)**; our own estimate is reported
  as directional.

### 2C. Add **Proposition 5 (Leakage Asymmetry)** to main.tex
- Insert formal environment near §5, `\label{prop:leakage}`:
  statement that under σ_critical < 1 < σ_commodity, the post-coercion
  substitution from China to swing states is **incomplete in critical
  sectors** and near-complete in commodity sectors.
- Proof sketch: one paragraph, or "See Appendix A.5" if pushed to appendix.
- Update Appendix B reference from "Prop 5" (currently dangling) to
  `\Cref{prop:leakage}`.

### 2D. Event-study reinterpretation (Domain M2)
- Rewrite the "post-2021 narrowing" sentence: the narrowing is consistent
  with **swing-state leakage attenuating the decoupling** (consistent with
  Prop 5, not contradicting Prop 2), because by 2021+ the swing-state
  indirect-China channel is operating.

---

## Phase 3 — Citation hygiene

### 3A. Add to `ref.bib` (web-verify before commit per MEMORY.md feedback):
- `atalay2017import` — Enes Atalay, "How Important Are Sectoral Shocks?",
  AEJ: Macroeconomics 9(4), 2017, 254–280.
- Confirm `boehm2019input` entry already present and correct (REStat
  101(1), 60–75, 2019).

### 3B. Replace `broda2006variety` in Appendix B §σ with `atalay2017import`
+ `boehm2019input` as primary support. Retain BW (2006) only for
variety-level discussion, not sector-level σ.

---

## Phase 4 — Formal Proposition 5 placement decision

Two options:
- **A (preferred):** Insert Prop 5 at end of §5 (main model), immediately
  after Prop 4. Clean narrative flow.
- **B (fallback):** Push to Appendix A.5 and cite `\Cref{prop:leakage}`
  from §3.3.

Default: A. Switch to B only if §5 already overflows.

---

## Phase 5 — Verification

1. Re-run `02_sigma_estimation.do`, `03_paradox_test.do`, `04_leakage.do`
   sequentially; confirm each `log close` writes cleanly.
2. Confirm `output/tables/leakage_triplediff.tex`, `paradox_main.tex`,
   `leakage_event_study.pdf` regenerate.
3. Full compile cycle on `main.tex`:
   `pdflatex → bibtex → pdflatex → pdflatex → pdflatex`
   Target: **0 undefined references, 0 undefined citations, ~49–51 pp.**
4. Grep sanity checks:
   - `\label{prop:leakage}` exists once, cited from §3.3 and Appendix B.
   - No literal `L\.` inside `$...$` math.
   - No `\cite{broda2006variety}` inside the new σ paragraph.
5. Diff the new event-study coefficient range against §3.3 quoted range;
   must match to one decimal.
6. Read §3.3 end-to-end; confirm no claim is stronger than the
   p-value warrants.

---

## Files to Modify

**STATA (re-run):**
- `empirical/scripts/02_sigma_estimation.do` — comment/framing only
- `empirical/scripts/03_paradox_test.do` — collinearity fix + post-2018 split
- `empirical/scripts/04_leakage.do` — base year + pre-trend F + 2-way cluster

**LaTeX:**
- `main.tex` — §3.3 rewrite, new Prop 5, Appendix B corrections
- `ref.bib` — add `atalay2017import`

**Regenerated (by STATA, do not hand-edit):**
- `tables/empirical/leakage_triplediff.tex`
- `empirical/output/tables/paradox_main.tex`
- `figures/empirical/leakage_event_study.pdf`

---

## Non-goals (explicit exclusions)

- Full Feenstra (1994) GMM implementation — out of scope; paper will rely on
  published σ values.
- `ppmlhdfe` migration for zero-retention (STATA M8) — flagged for appendix
  robustness in a follow-up session.
- Nested-CES robustness and bifurcation-locus figure (from original Phase 1
  theory-revision plan) — already covered by current §6.9.
- GTA / ICIO data — still blocked on manual user action.

---

## Phased Execution Order

**Phase A (30–45 min):** STATA re-runs (1A, 1B, 1C, 1D).
**Phase B (30 min):** Manuscript prose + numbers (2A, 2B, 2D) using the new output.
**Phase C (20 min):** Prop 5 insertion (2C) + citation swap (3A, 3B).
**Phase D (10 min):** Verify compile + grep checks (Phase 5).

Single session, sequential. Total: ~1.5–2 hours.
